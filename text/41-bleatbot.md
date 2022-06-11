- Feature Name: (fill me in with a unique ident, `my_awesome_feature`)
- Start Date: (fill me in with today's date, YYYY-MM-DD)
- DIP PR: [goatcorp/DIPs#41](https://github.com/goatcorp/DIPs/pull/41)
- Repo-Relevant Issue: [goatcorp/dalamud#0000](https://github.com/goatcorp/dalamud/issues/0000)

# Summary

[summary]: #summary

Introducing Bleatbot, a GitHub bot designed to help manage issues and pull requests for various repositories (support, plugin submissions, DIPs, etc.).

# Motivation

[motivation]: #motivation

There are many problems with GitHub issues:

- It is difficult to provide support via GitHub issues. Franzbot (the FAQ bot in the Goat Place Discord server) does not exist outside of Discord, so the support team does not have an easy way to provide common troubleshooting steps.
- Issues and pull requests often are forgotten about and become stale, left open for extended periods of time. While Caprine Operator exists to help plugin PRs, there is no functionality to help clean up issues.
- The DIP process is manual, done by humans, even including the merge process.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

In the comments of a issue/pull request on a goatcorp GitHub repository, members of the goatcorp GitHub group can execute any of the following commands:

- `@bleatbot faq <entry>` - Post an FAQ entry.
- `@bleatbot autoClose <when>` - Automatically close an issue after the specified time (e.g `24h`).
- `@bleatbot autoMerge <when>` - Automatically merge a pull request after the specified time (e.g. `24h`).
- `@bleatbot cancelAuto` - Cancel any automatic tasks.

TODO

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

Bleatbot is written in TypeScript. Its source code is available in the [NotNite/bleatbot](https://github.com/NotNite/bleatbot) GitHub repository.

It uses SQLite as its database (through Prisma), Octokit to use the GitHub API, and Koa to receive events through a webhook.

Each GitHub repository has a webhook added to it, which points to an HTTP server the Bleatbot codebase runs. Upon an event, the webhook is triggered and validated, and the command is parsed and ran.

# Drawbacks

[drawbacks]: #drawbacks

- Data storage of any kind is a liability.
- This adds complexity to the core Dalamud ecosystem, both as a result of being an additional tool and through its terminology/commands.
- Server resources are required to host Bleatbot.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

- As a member of the support team myself, it is annoying to either type out a long-winded sentence or redirect users to our Discord server, because we do not have the proper functionality desired.
- I chose TypeScript because I like TypeScript. This would work as well in other languages, it's just what I'm comfortable in. I have no excuses.
- I chose the libraries I used because they either fit TypeScript well, I'm comfortable with them, or they were suggested to me by ackwell.

# Prior art

[prior-art]: #prior-art

Franzbot is prior art regarding the FAQ system. It has proven to be massively helpful in the Discord server, where we receive a lot more users needing support.

Many GitHub bots with command functionality & auto merging exist for communities, but there are far too many to list.

# Unresolved questions

[unresolved-questions]: #unresolved-questions

- What other features does Bleatbot need?
- Does any command need to be renamed?

# Future possibilities

[future-possibilities]: #future-possibilities

- More FAQ entries
- Integration with the Caprine Operator
