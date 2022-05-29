- Feature Name: Formal Subteams
- Start Date: 2022-05-03
- DIP PR: [goatcorp/DIPs#18](https://github.com/goatcorp/DIPs/pull/18)

# Summary

[summary]: #summary

Introducing formal subteams for each area of the goatcorp ecosystem to make it easier to find the responsible parties.

# Motivation

[motivation]: #motivation

It is currently difficult to know who to ask for help, support or guidance. goat is the primary point of contact for all operations, which causes problems when he is otherwise indisposed. Introducing subteams will help delegate responsibility, and make it easier to keep track of ongoing initiatives.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

Eight subteams will be created for each area of the ecosystem:

- **Launcher**: development, maintenance and code documentation of XIVLauncher
- **Dalamud**: development, maintenance and code documentation of Dalamud
- **Unix**: ensures that XIVLauncher and Dalamud work well and/or conveniently on Linux and macOS, including the Steam Deck
- **Documentation**: creation, maintenance and improvement of goatcorp-related documentation, including the FAQs, support guides, DIPs and developer documentation (for now, including SamplePlugin). Does not handle documentation within goatcorp code, but may document code intended for other people (e.g. plugin developers.)
- **Moderation**: interacting with users and ensuring that they respect the sanctity of the Goat Place
- **Community**: community relations and support, triaging and pruning issues, as well as bringing community feedback to the other teams. explicitly _not_ moderation.
- **Plugin Approval**: reviewing and approving plugins, and taking the necessary steps to ensure that they are safe
- **Reverse Engineering**: people who are particularly good at reversing the game and can be consulted for assistance or guidance

Members can belong to multiple subteams, but they are encouraged to specialise and delegate where possible to ensure continuity of operations.

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

The initial members of each subteam will be as follows:

- **Launcher**: goat, kizer, Marzent
- **Dalamud**: goat, daemitus, Aireil, Caraxi
- **Unix**: ashkitten, Caliel666, Centzilius, Marzent, Dormanil, Helios747, Zips
- **Documentation**: Franz, Philpax, kal
- **Moderation**: Aida Enna, Arc/Dis, Caraxi, Dale, AmenneHolelane
- **Community**: Franz, kal, NotNite, goat
- **Plugin Approval**: goat, Franz, Caraxi, karashiiro
- **Reverse Engineering**: aers, Adam, Pohky, Caraxi

A GitHub subteam under goatcorp will be created for each subteam, and each member will be added to it. Each subteam will be granted contributor privileges to relevant repositories.

Adding or removing members to a subteam is done by informal consensus. This may be specified further at a later date, but we don't know how well it'll work until we try it.

The GitHub subteam list will remain as the authoritative source for team membership at this time, but it may be replicated to the FAQ or the Discord (e.g. for roles).

# Drawbacks

[drawbacks]: #drawbacks

This will induce some degree of organisational burden amongst goatcorp (specifically goat) while everything is being set up.

Aspects of the goatcorp ecosystem will be entrusted to people who aren't goat, which could potentially mean the wrong call is made with goatcorp permissions. Hopefully we pick the right people!

There also exists the risk of the community becoming insular, with new people feeling disinclined to join. We have generally done a good job of managing this so far, but it's something we need to keep in mind. We need to make sure that these teams bring people together, not keep them out.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

These are relatively straightforward teams that formalise community dynamics that we already have. It is relatively conservative in this regard, and should only seek to give names to what we already have.

This DIP does not propose anything further as goatcorp will need to explore this before it attempts anything more.

If we do not do this, the majority of the governance will remain with goat, which is inherently unscalable, especially as the number of plugins, and devs, grows well into the dozens. Other people are already technically empowered to act on goat's behalf, but generally avoid doing so as they have not been given explicit permission.

# Prior art

[prior-art]: #prior-art

This is patterned on the [Rust governance model](https://forge.rust-lang.org/governance/index.html), which seems to work pretty well for them. I believe this is not too dissimilar from other FOSS projects.

# Unresolved questions

[unresolved-questions]: #unresolved-questions

- Are these the only subteams we want?:
  - Initial consensus: yes, for now.
- Do we want formal leaders for each subteam?
  - Initial consensus: no, that's too much administrative overhead. We'll try without leaders for now and see how we go.
- What comes under the purview of Documentation?
  - Initial consensus: some developer relations-related responsibilities as well, but code documentation remains with the people writing the code
- What level of permission do we want to entrust to a subteam member?

# Future possibilities

[future-possibilities]: #future-possibilities

- We could potentially develop policies around how to join a subteam, so that people interested in participating know how to start.
  - A Discord bot could be used for this to vet people's applications, [as suggested by ArcaneDisgea](https://github.com/goatcorp/DIPs/pull/18#discussion_r880978776).
