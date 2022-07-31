- Feature Name: Chat API Improvements
- Start Date: 2022-06-03
- DIP PR: [DIPs/39](https://github.com/goatcorp/DIPs/pull/39)
- Repo-Relevant Issue: TBD

# Summary

[summary]: #summary

Add functionality to Dalamud's `ChatGui` API that lets callers and subscribers make better decisions with the messages they are handling, and fix naming/typing mistakes in the current API.

# Motivation

[motivation]: #motivation

Currently, users cannot easily know where chat messages that are received in raised events originated.  This can cause issues where messages printed to the chat log from plugins are handled in an undesirable way.  Furthermore, the `senderId` in current API functions and events is not actually related to the sender, creating confusion.  Within Dalamud's API itself (i.e., not custom hooks), here is currently no way to discern the sender of a message (or set the sender for an API user's message).

The time spent implementing this DIP is also a good opportunity to make minor improvements to the chat typing system.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

### Current Behavior:
- When a user wants to be notified of chat messages and potentially intercept or alter them, they subscribe to one of the events defined by Dalamud's `ChatGui` API (`ChatMessage`, `CheckMessageHandled`, `ChatMessageHandled`, and `ChatMessageUnhandled`).
- These events will be raised when an appropriate message is written to the game's chat log (or handled by another subscriber, as appropriate).  The subscriber receives, depending on the event type, some combination of the chat channel, `ref`s to the message sender's name and the message itself, the message's timestamp[^1], and a `ref` flag to indicate whether Dalamud should propagate the message any further.
- The subscriber can, as appropriate to the event type, alter these parameters, prevent the message from propagating, and execute their own logic based on what they are given.
- When using the `ChatGui` API, a user can also send their own messages to be printed to the chat log (`PrintChat`, `Print`, and `PrintError`).  These can, essentially, include the same parameters as those found in the events (sender name, message, and timestamp[^1]).
- Values of type `XivChatType` currently can contain source/target info in the upper bits.  This can be confusing when processing chat messages.

### Proposed changes:
- The API would retain the same broad functionality as currently exists; users would be able to subscribe to the same message events, and be able to send their own chat log messages.  The differences are primarily in the information exposed to and controlled by the user:
	- The timestamp parameter[^1] would be renamed to properly indicate its purpose.  It would also be made a `ref` in the events so that event handlers could alter it (most won't, but I see little reason to restrict this when it may be useful in rare cases).
	- A parameter would be added to the chat events that indicates the source of the message (game, Dalamud, or plugin), with which subscribers can inform their decisions about handling the message.  A supplemental parameter would be added with the `InternalName` of the plugin for plugin-originating messages.
- When a user uses `PrintChat`, a callback can be provided that receives the message index[^2][^3].  This index does not matter to most API users, but is relevant when working with certain game chat functions.
- XivChatType would have methods to mask the source/target/channel, possibly implicitly, to make use by uninformed devs less prone to error.[^5]


### Examples:

This event subscriber as currently exists

```csharp
private void OnChatMessage( XivChatType type, UInt32 senderId, ref SeString sender, ref SeString message, ref bool isHandled )
{
    //  The next line is ugly, and there's no indication to the uninformed dev that this may be how they have to check the chat channel.
    if( (XivChatType)((Uint16)type & 0x7F) != XivChatType.GatheringSystemMessage ) return;
    
    //  Change the message contents.  This catches all messages of this type, even ones we might not want to alter.
}
```

could become[^6]

```csharp
private void OnChatMessage( XivChatType type, ref UInt32 timestamp, ref SeString sender, ref SeString message, XivChatSource source, string sourceName, ref bool isHandled )
{
    if( source == XivChatSource.Plugin && sourceName == "SuperImportantPlugin_DoNotChangeMessages" ) return;
    if( type.Channel != XivChatType.GatheringSystemMessage ) return;

    //  Change the message contents.  We can now exclude messages from sources that should not be altered.
}
```

Sending a chat message as currently exists

```csharp
var chatEntry = new XivChatEntry
{
    Type = XivChatType.Say,
    Name = "Bob Loblaw",	//  Player link payloads omitted for clarity.
    SenderId = bobsObjectID,	//  See footnote 1.
    Message = MyAwesomeTranslationFunction( "Check out my cool law blog!" ),
};

mChatGui.PrintChat( chatEntry );
```

could become[^6]

```csharp
var chatEntry = new XivChatEntry
{
    Type = XivChatType.Say,
    Name = "Bob Loblaw",	//  Player link payloads omitted for clarity.
    Timestamp = 0,		//  A value of zero makes the timestamp automatic.
    Message = MyAwesomeTranslationFunction( "Check out my cool law blog!" ),
};

//  The optional callback gives us an opportunity to do things that depend on the message index, such as setting sender information to make some context menu items functional.
UInt32 messageIndex = mChatGui.PrintChat( chatEntry, msgIndex =>
{
    RaptureLogModuleSetSenderInfo( bobsContentID, msgIndex, bobsServerID, XivChatType.Say );
} );
```

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

- All occurrances of sender ID fields and parameters are renamed to reflect that they are timestamps.
- An enumeration for chat message source is created (`Game`, `Dalamud`, and `Plugin`).
- The majority of the logic in `ChatGui.HandlePrintMessageDetour` is split into a new function, with parameters for the chat message source enum and plugin name.
- The hook that redirected to that function will now redirect to a small function that calls the new `HandlePrintMessageDetour` with a message source of `Game`.
- The other `ChatGui` message printing functions will place messages into the queue with with the appropriate message source parameter(s), which will call the new `HandlePrintMessageDetour` function with those parameters.  There will be both internal and public versions of the message queueing functions so that plugins cannot impersonate or otherwise abuse the message source.
- When queueing a message to be printed, the user can provide a callback that receives the message index once printed (since it is returned by the game function that prints to log).
- Add a note to the documentation for `PrintChat` that describes how available context menu entries are determined.  Those that require a content ID to be set will not be available from within the Dalamud API, requiring either Client Structs or manual invocation, depending on whether Client Structs even wants that function.
- Fill in missing chat channel enums in `XivChatType`.
- Add values to the enum for source/target flags, enabling combining enums for special case uses that may need it.[^5]
- Implement extension methods for `XivChatType` that do the following:
	- Mask the type to provide sender, target, and actual channel.
	- Provide the localized name of a given channel.

# Drawbacks

[drawbacks]: #drawbacks

- This would require either an API level change, which would breaking existing plugins, or adding bloat to Dalamud by implementing parallel API types, events, and methods to retain backward compatibility.  Timing these changes with the next Dalamud API level change is probably the better choice.
- Changes to `XivChatType` have the potential to break existing plugins in more undesirable ways.  They also have the potential to confuse users selecting a chat channel in Dalamud/plugin settings, letting them select a channel that doesn't exist.  Devs will have to be mindful of this, depending on the changes made implementing this facet of the DIP.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

- The current design of the API makes some chat log-related plugin design more difficult; for example, filtering out messages from the game, but letting messages from Dalamud or plugins pass through.
- The current API misnames a parameter and (accidentally) misrepresents how to set the sender of a given message (from the game's point of view).
- This is a relatively narrow change to Dalamud overall, and has minimal impact on users outside of a potential API level change, which would only require a few parameter changes to maintain existing functionality[^6].  Anyone currently misusing senderID[^1] would simply have to rename their parameters/properties, and could retain current behavior.

# Prior art

[prior-art]: #prior-art

As this is a change to an extant Dalamud API, and it is an API that's somewhat specific to FFXIV's implementation of the chat log, there is probably not significant prior art for the changes.  

# Unresolved questions

[unresolved-questions]: #unresolved-questions

- Is asking the caller to supply a callback for receiving the message index the best way to handle the consumer getting that information?
- Should the `parameter` parameter of the game's print to chat log function be RE'd in more detail as part of the implementation of this DIP?
- Should chat type in the chat events be made a ref?  Could be useful, but changing that also has the potential to cause some fuckery for certain types of messages.  So does changing the message sender if it's a player payload, though, and that's already a ref.
- Is providing a string parameter with the plugin name along with the message source (for plugin-sourced messages) the best idea?  Is there a better solution that still allows proper source discrimination?  Can we easily get the calling plugin's name inside of Dalamud (with reasonable performance) so that we don't have to trust plugins to set their name?  The most obvious solution is `Assembly.GetCallingAssembly`.  How reliable is that, and how slow is it?  Are there better alternatives?
- Is anyone from the the more experienced RE community available to help confirm game function behavior prior to implementation of changes to such a frequently-used API?
- Would adding source/target values to the `XivChatType` enum cause issues for existing users?  I personally enumerate the enum values and put them in a dropdown, where having these values could potentially let a user make a bad decision.  I believe that Dalamud settings does this as well.  It is an easily resolvable problem if you know about it, but it could be a pitfall for uninformed plugin developers.
- The other potential option for improving `XivChatType` is making it a struct with const values for the different channels, and methods/operators to handle the rest.  My gut feeling is that doing this could create unexpected surprises in existing plugins, and that extension methods on the existing enum are the better way forward (the downside being lack of implicit conversions and operators).  What is the best way to handle chat type masking, while avoiding pitfalls for newer developers?
- Is implementing client language localization for `XivChatType` values a good idea?  I do personally localize in places where I use these, and it would be nice to not have to write/copy code to do it each time.  This would require loading sheets in the chat API code, and it is not obvious how to name a few chat channels, such as both incoming and outgoing tells occupying the same row in the `LogFilter` sheet.

# Future possibilities

[future-possibilities]: #future-possibilities

As this is a fairly narrowly scoped DIP, the intent is to ensure that the changes above resolve the root issues and update the API to provide what users need at the implementation of this DIP.  Stakeholders are encouraged to bring up any additions/conflicts that they envision within the scope of this DIP during its review process.


[^1]: This timestamp is currently called senderId in the API, leading to some confusion.  See [Dalamud PR #865](https://github.com/goatcorp/Dalamud/pull/865).
[^2]: This is the `UInt32` index of the message in the game's log, and is used for some log-related game functions.  See the Reference-level explanation for more information.
[^3]: It is unclear whether providing a message index to the caller is desirable for `Print` and `PrintError`, since those are simpler functions.  I don't personally see a drawback to allowing the caller to get that information if they want it, though.
[^4]: REMOVED
[^5]: This may cause issues with existing plugins and parts of Dalamud.  See Drawbacks and Unresolved questions.
[^6]: It is intended that current plugin code continue to function as it exists, with only new parameters added in event handlers and the `SenderId` property name changed.  This conflicts somewhat with the `XivChatType` question in Unresolved questions.
