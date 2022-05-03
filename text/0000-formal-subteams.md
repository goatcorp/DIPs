- Feature Name: Formal Subteams
- Start Date: 2022-05-03
- DIP PR: [goatcorp/DIPs#18](https://github.com/goatcorp/DIPs/pull/18)
- Repo-Relevant Issue: [goatcorp/dalamud#0000](https://github.com/goatcorp/dalamud/issues/0000)

# Summary

[summary]: #summary

Introducing formal subteams for each area of the goatcorp ecosystem to make it easier to find the responsible parties.

# Motivation

[motivation]: #motivation

It is currently difficult to know who to ask for help, support or guidance. goat is the primary point of contact for all operations, which causes problems when he is otherwise indisposed. Introducing subteams will help delegate responsibility, and make it easier to keep track of ongoing initiatives.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

Six subteams will be created for each area of the ecosystem:

- **Launcher**: development and maintenance of XIVLauncher
- **Dalamud**: development and maintenance of Dalamud
- **Linux**: ensures that XIVLauncher and Dalamud work well and/or conveniently on Linux, including the Steam Deck
- **Documentation**: creation, maintenance and improvement of goatcorp-related documentation, including the FAQs, support guides, and developer documentation.
  - RFC: DIPs too?
  - RFC: SamplePlugin et al?
- **Community**: community relations and support, as well as bringing community feedback to the other teams
- **Plugin Approval**: reviewing and approving plugins, and taking the necessary steps to ensure that they are safe

Members can belong to multiple subteams, but they are encouraged to specialise and delegate where possible to ensure continuity of operations.

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

The initial members of each subteam will be as follows:

- **Launcher**: goat, kizer, ???
- **Dalamud**: goat, daemitus, ???
- **Linux**: ???
- **Documentation**: Franz, Philpax, kal, ???
- **Community**: Franz, kal, ???
- **Plugin Approval**: goat, Franz, Caraxi, ???

A GitHub subteam under goatcorp will be created for each subteam, and each member will be added to it. Each subteam will be granted contributor privileges (RFC: too much?) to the repositories.

Adding or removing members to a subteam is done by informal consensus. This may be specified further at a later date, but we don't know how well it'll work until we try it.

The GitHub subteam list will remain as the authoritative source for team membership at this time, but it may be replicated to the FAQ or the Discord (e.g. for roles).

# Drawbacks

[drawbacks]: #drawbacks

This will induce some degree of organisational burden amongst goatcorp (specifically goat) while everything is being set up.

Aspects of the goatcorp ecosystem will be entrusted to people who aren't goat, which could potentially mean the wrong call is made with goatcorp permissions. Hopefully we pick the right people!

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

These are relatively straightforward teams that formalise community dynamics that we already have. It is relatively conservative in this regard, and should only seek to give names to what we already have.

This RFC does not propose anything further as goatcorp will need to explore this before it attempts anything more.

If we do not do this, the majority of the governance will remain with goat, which is inherently unscalable, especially as the number of plugins, and devs, grows well into the dozens. Other people are already technically empowered to act on goat's behalf, but generally avoid doing so as they have not been given explicit permission.

# Prior art

[prior-art]: #prior-art

This is patterned on the [Rust governance model](https://forge.rust-lang.org/governance/index.html), which seems to work pretty well for them. I believe this is not too dissimilar from other FOSS projects.

# Unresolved questions

[unresolved-questions]: #unresolved-questions

- Are these the only subteams we want?
- Do we want formal leaders for each subteam?
- What comes under the purview of Documentation?
- What level of permission do we want to entrust to a subteam member?

# Future possibilities

[future-possibilities]: #future-possibilities

We could potentially develop policies around how to join a subteam, so that people interested in participating know how to start.