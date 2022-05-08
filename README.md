# Dalamud Improvement Proposals

[Dalamud DIPs]: #dalamud-dips

_with many thanks to the [Rust RFC process](https://github.com/rust-lang/rfcs),
which this is based off_

Many changes, including bug fixes and documentation improvements can be
implemented and reviewed via the normal GitHub pull request workflow.

Some changes though are "substantial", and we ask that these be put through a
bit of a design process and produce a consensus among the developer community.

The DIP process is intended to provide a consistent and controlled path for
new changes to enter the goatcorp ecosystem, so that all stakeholders can be
confident about the direction Dalamud et al are evolving in.

## Table of Contents

[table of contents]: #table-of-contents

- [Opening](#dalamud-dips)
- [Table of Contents]
- [When you need to follow this process]
- [Before creating a DIP]
- [What the process is]
- [The DIP life-cycle]
- [Reviewing DIPs]
- [Implementing a DIP]
- [DIP Postponement]
- [Help this is all too informal!]
- [Contributions]

## When you need to follow this process

[when you need to follow this process]: #when-you-need-to-follow-this-process

You need to follow this process if you intend to make "substantial" changes to
Dalamud, XIVLauncher, the surrounding infrastructure, or the DIP process itself.
What constitutes a "substantial" change is evolving based on community norms and
varies depending on what part of the ecosystem you are proposing to change, but
may include the following.

- Any semantic or syntactic change to Dalamud that is not a bugfix.
- Removing Dalamud features, including those that are feature-gated.
- Changes to the interface between the game and Dalamud.
- Additions to the `Dalamud` namespace.

Some changes do not require a DIP:

- Rephrasing, reorganizing, refactoring, or otherwise "changing shape does
  not change meaning".
- Additions that strictly improve objective, numerical quality criteria
  (warning removal, speedup, better platform coverage, more parallelism, trap
  more errors, etc.)

If you submit a pull request to implement a new feature without going through
the DIP process, it may be closed with a polite request to submit a DIP first.

## Before creating a DIP

[before creating a DIP]: #before-creating-a-dip

A hastily-proposed DIP can hurt its chances of acceptance. Low quality
proposals, proposals for previously-rejected features, or those that don't fit
into the near-term roadmap, may be quickly rejected, which can be demotivating
for the unprepared contributor. Laying some groundwork ahead of the DIP can
make the process smoother.

Although there is no single way to prepare for submitting a DIP, it is
generally a good idea to pursue feedback from other project developers
beforehand, to ascertain that the DIP may be desirable; having a consistent
impact on the project requires concerted effort toward consensus-building.

The most common preparation for writing and submitting a DIP is talking
the idea over on the [Goat Place Discord server](https://discord.gg/3NMcUV5).
You may file issues on this repo for discussion, but these are not actively
looked at by the teams.

As a rule of thumb, receiving encouraging feedback from long-standing project
developers, and particularly members of the relevant sub-team is a good
indication that the DIP is worth pursuing.

## What the process is

[what the process is]: #what-the-process-is

In short, to get a major change made for Dalamud, one must first get the DIP
merged into the DIP repository as a markdown file. At that point the DIP is
"active" and may be implemented with the goal of eventual inclusion into Dalamud.

- Fork the DIP repo.
- Copy `0000-template.md` to `text/0000-my-feature.md` (where "my-feature" is
  descriptive). Don't assign a DIP number yet; This is going to be the PR
  number and we'll rename the file accordingly if the DIP is accepted.
- Fill in the DIP. Put care into the details: DIPs that do not present
  convincing motivation, demonstrate lack of understanding of the design's
  impact, or are disingenuous about the drawbacks or alternatives tend to
  be poorly-received.
- Submit a pull request. As a pull request the DIP will receive design
  feedback from the larger community, and the author should be prepared to
  revise it in response.
- Now that your DIP has an open pull request, use the issue number of the PR
  to update your `0000-` prefix to that number.
- Each pull request will be labeled with the most relevant sub-team, which
  will lead to its being triaged by that team in a future meeting and assigned
  to a member of the subteam.
- Build consensus and integrate feedback. DIPs that have broad support are
  much more likely to make progress than those that don't receive any
  comments. Feel free to reach out to the DIP assignee in particular to get
  help identifying stakeholders and obstacles.
- The sub-team will discuss the DIP pull request, as much as possible in the
  comment thread of the pull request itself. Offline discussion will be
  summarized on the pull request comment thread.
- DIPs rarely go through this process unchanged, especially as alternatives
  and drawbacks are shown. You can make edits, big and small, to the DIP to
  clarify or change the design, but make changes as new commits to the pull
  request, and leave a comment on the pull request explaining your changes.
  Specifically, do not squash or rebase commits after they are visible on the
  pull request.
- Once discussion has settled and a rough consensus has been reached, the DIP
  will be merged by a member of the relevant subteam, and work can begin.

## The DIP life-cycle

[the dip life-cycle]: #the-dip-life-cycle

Once a DIP becomes "active" then authors may implement it and submit the
feature as a pull request to the relevant repo. Being "active" is not a rubber
stamp, and in particular still does not mean the feature will ultimately be
merged; it does mean that in principle all the major stakeholders have agreed
to the feature and are amenable to merging it.

Furthermore, the fact that a given DIP has been accepted and is "active"
implies nothing about what priority is assigned to its implementation, nor does
it imply anything about whether a developer has been assigned the task of
implementing the feature. While it is not _necessary_ that the author of the
DIP also write the implementation, it is by far the most effective way to see
a DIP through to completion: authors should not expect that other project
developers will take on responsibility for implementing their accepted feature.

Modifications to "active" DIPs can be done in follow-up pull requests. We
strive to write each DIP in a manner that it will reflect the final design of
the feature; but the nature of the process means that we cannot expect every
merged DIP to actually reflect what the end result will be at the time of the
next major release.

In general, once accepted, DIPs should not be substantially changed. Only very
minor changes should be submitted as amendments. More substantial changes
should be new DIPs, with a note added to the original DIP.

## Reviewing DIPs

[reviewing dips]: #reviewing-dips

A sub-team makes final decisions about DIPs after the benefits and drawbacks
are well understood. These decisions can be made at any time, but the sub-team
will regularly issue decisions. When a decision is made, the DIP pull request
will either be merged or closed. In either case, if the reasoning is not clear
from the discussion in thread, the sub-team will add a comment describing the
rationale for the decision.

## Implementing a DIP

[implementing a DIP]: #implementing-a-dip

Some accepted DIPs represent vital features that need to be implemented right
away. Other accepted DIPs can represent features that can wait until some
arbitrary developer feels like doing the work. Every accepted DIP has an
associated issue tracking its implementation in the relevant repository.

The author of a DIP is not obligated to implement it. Of course, the DIP
author (like any other developer) is welcome to post an implementation for
review after the DIP has been accepted.

If you are interested in working on the implementation for an "active" DIP, but
cannot determine if someone else is already working on it, feel free to ask
(e.g. by leaving a comment on the associated issue).

## DIP Postponement

[dip postponement]: #dip-postponement

Some DIP pull requests are tagged with the "postponed" label when they are
closed (as part of the rejection process). a DIP closed with "postponed" is
marked as such because we want neither to think about evaluating the proposal
nor about implementing the described feature until some time in the future, and
we believe that we can afford to wait until then to do so. Historically,
"postponed" was used to postpone features until after 1.0. Postponed pull
requests may be re-opened when the time is right. We don't have any formal
process for that, you should ask members of the relevant sub-team.

Usually a DIP pull request marked as "postponed" has already passed an
informal first round of evaluation, namely the round of "do we think we would
ever possibly consider making this change, as outlined in the DIP pull request,
or some semi-obvious variation of it." (When the answer to the latter question
is "no", then the appropriate response is to close the DIP, not postpone it.)

### Help this is all too informal!

[help this is all too informal!]: #help-this-is-all-too-informal

The process is intended to be as lightweight as reasonable for the present
circumstances. As usual, we are trying to let the process be driven by
consensus and community norms, not impose more structure than necessary.

### Contributions

[contributions]: #contributions

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in the work by you shall be licensed as MIT without
any additional terms or conditions. Additionally, any contribution submitted
for inclusion in works affected by a DIP shall be licensed with the same
license as that of the target work (e.g. contributions made to Dalamud under
the aegis of a DIP will be licensed as AGPL, as Dalamud is AGPL at time of
writing.)
