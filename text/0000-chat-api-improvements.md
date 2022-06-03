- Feature Name: Chat API Improvements
- Start Date: 2022-06-03
- DIP PR: [DIPs/39](https://github.com/goatcorp/DIPs/pull/39)
- Repo-Relevant Issue: TBD

# Summary

[summary]: #summary

Add functionality to Dalamud's `ChatGui` API that lets callers and subscribers make better decisions with the messages they are handling, and fix naming/typing mistakes in the current API.

# Motivation

[motivation]: #motivation

Currently, users cannot easily know from where chat messages that are received in raised events originate.  This can cause issues where messages printed to the chat log from plugins are handled in an undesirable way.  Furthermore, the `senderId` in current API functions and events is not actually related to the sender, creating confusion.  Within Dalamud's API itself (i.e., not custom hooks), here is currently no way to discern the sender of a message (or set the sender for an API user's message).

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

### Current Behavior:
- When a user wants to be notified of chat messages and potentially intercept or alter them, they subscribe to one of the events defined by Dalamud's `ChatGui` API (`ChatMessage`, `CheckMessageHandled`, `ChatMessageHandled`, and `ChatMessageUnhandled`).
- These events will be raised when an appropriate message is written to the game's chat log (or handled by another subscriber, as appropriate).  The subscriber receives, depending on the event type, some combination of the chat channel, message sender's name, the message itself, the message's timestamp[^1], and a `ref` flag to indicate whether Dalamud should propagate the message any further.
- The subscriber can, as appropriate to the event type, alter these parameters, prevent the message from propagating, and execute their own logic based on what they are given.
- When using the `ChatGui` API, a user can also send their own messages to be printed to the chat log (`PrintChat`, `Print`, and `PrintError`).  These can, essentially, include the same parameters as those found in the events (sender name, message, and timestamp[^1]).

### Proposed changes:
- The API would retain the same broad functionality as currently exists; users would be able to subscribe to the same message events, and be able to send their own chat log messages.  The differences are primarily in the information exposed to and controlled by the user:
	- The timestamp parameter[^1] would be renamed to properly indicate its purpose.  It would also be made a `ref` in the events so that event handlers could alter it (most won't, but I see little reason to restrict this when it may be useful in rare cases).
	- A parameter would be added to the chat events that indicates the source of the message (game, Dalamud, or plugin), with which subscribers can inform their decisions about handling the message.  A supplemental parameter would be added with the `InternalName` of the plugin for plugin-originating messages.
- When a user uses `PrintChat`, a callback can be provided that receives the message index[^2][^3].
- A new function would be added to set the content ID and world of a specific message's sender.  This is required for some context menu items to work when clicking on a message sender in the game's chat log.[^4].


### Examples:

This event subscriber as currently exists

```csharp
private void OnChatMessage( XivChatType type, UInt32 senderId, ref SeString sender, ref SeString message, ref bool isHandled )
{
    //  Change the message contents.  This catches all messages, even ones we might not want to alter.
}
```

would become

```csharp
private void OnChatMessage( XivChatType type, ref UInt32 timestamp, ref SeString sender, ref SeString message, XivChatSource source, string sourceName, ref bool isHandled )
{
    if( source == XivChatSource.Plugin && sourceName == "SuperImportantPlugin_DoNotChangeMessages" ) return;
	
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
    Message = Translate( "Check out my cool law blog!" ),
};

mChatGui.PrintChat( chatEntry );
```

could become

```csharp
var chatEntry = new XivChatEntry
{
    Type = XivChatType.Say,
    Name = "Bob Loblaw",	//  Player link payloads omitted for clarity.
    Timestamp = 0,		//  A value of zero makes the timestamp automatic.
    Message = Translate( "Check out my cool law blog!" ),
};

//  The callback makes all of the context menu items work on Bob's name in the message we just printed.
UInt32 messageIndex = mChatGui.PrintChat( chatEntry, msgIndex =>
{
    SetContentIDForMessage( bobsContentID, msgIndex, bobsServerID, XivChatType.Say );
} );
```

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

**Implementation details are pending discussion of features, but are outlined as follows:**

- All occurrances of sender ID fields and paramters are renamed to reflect that they are timestamps.
- The majority of the logic in `ChatGui.HandlePrintMessageDetour` is split into a new function, with parameters for the chat message source.
- The hook that redirected to that function will now redirect to a small function that calls the new `HandlePrintMessageDetour` with a message source of "Game".
- The other `ChatGui` message printing functions will place messages into the queue with with the appropriate message source parameter(s), which will call the new `HandlePrintMessageDetour` function with those parameters.  Most likely, there will be both internal and external versions of the message queueing functions so that plugins cannot impersonate or otherwise abuse the message source.
- When queueing a message to be printed, the user can provide a callback that receives the message index once printed (since it is returned by the game function that prints to log).

# Drawbacks

[drawbacks]: #drawbacks

- This would either require an API level change, breaking existing plugins, or add bloat to Dalamud by adding parallel API types, events, and methods to retain backward compatibility.
- The function to set the content ID of a message sender may have the potential to be dangerous if misused.  It may not belong in Dalamud proper.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

- The current design of the API makes some chat log-related plugin design more difficult; for example, filtering out messages from the game, but letting messages from Dalamud or plugins pass through.
- The current API misnames a parameter and (accidentally) misrepresents how to set the sender of a given message (from the game's point of view).
- This is a relatively narrow change to Dalamud overall, and has minimal impact on users outside of a potential API level change, which would only require a few parameter changes to maintain existing functionality.  Anyone currently misusing senderID[^1] would simply have to rename their parameters/properties, and could retain current behavior.

# Prior art

[prior-art]: #prior-art

As this is a change to an extant Dalamud API, and it is an API that's somewhat specific to FFXIV's implementation of the chat log, there is probably not significant prior art for the changes.  

# Unresolved questions

[unresolved-questions]: #unresolved-questions

- Should setting the content ID of a message be allowed through Dalamud, instead of through Client Structs or even requiring the user to hook themselves?  How dangerous is it if a user mis-sets this data that is then used by the game to possibly contact the server (i.e., request the adventurer plate of a player from the chat window context menu)?  Should this feature be removed from the DIP?
- Is it better to send the Content ID desired to be set as part of the message struct instead of having a callback when printing the message?  It probably depends on what we decide for the above.  Regardless, the callback is more general-purpose, and the user can store it for other use not envisioned here, so it may be the more future-resistant implementation.
- Should the `parameter` parameter of the game's print to chat log function be RE'd in more detail as part of the implementation of this DIP?
- Is providing a string parameter of the plugin name along with the message source (for plugin-sourced messages) the best idea?  Is there a better solution that still allows proper source discrimination?  Can we easily get the calling plugin's name inside of Dalamud (wih reasonable performance)?
- Is anyone from the the more experienced RE community available to help confirm game function behavior prior to implementation of changes to such a widely-used API?

# Future possibilities

[future-possibilities]: #future-possibilities

As this is a fairly narrowly scoped DIP, the intent is to ensure that the changes above resolve the root issues and update the API to provide what users need at the implementation of this DIP.


[^1]: This timestamp is currently called senderId in the API, leading to some confusion.  See [Dalamud PR #865](https://github.com/goatcorp/Dalamud/pull/865).
[^2]: This is the `UInt32` index of the message in the game's log, and is used for some log-related game functions.  See the Reference-level explanation for more information.
[^3]: It is unclear whether providing a message index to the caller is desirable for `Print` and `PrintError`, since those are simpler functions.  I don't personally see a drawback to allowing the caller to get that information if they want it, though.
[^4]: This may have the potential to be dangerous if misused.  See Drawbacks and Unresolved questions.
