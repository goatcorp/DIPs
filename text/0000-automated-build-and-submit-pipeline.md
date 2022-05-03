- Feature Name: Automated Build and Submit Pipeline
- Start Date: 2022-05-03
- DIP PR: [goatcorp/DIPs#17](https://github.com/goatcorp/DIPs/pull/17)
- Repo-Relevant Issue: [goatcorp/dalamud#0000](https://github.com/goatcorp/dalamud/issues/0000)

# Summary

[summary]: #summary

Introduce a new automated build and submit pipeline to simplify plugin submission, validation and updates.

# Motivation

[motivation]: #motivation

The existing method of submitting and updating a plugin is tedious and labour-intensive. It requires the developer to compile their own code, package it up, and create a matching PR to the DalamudPlugins repo, while ensuring that all package metadata remains consistent.

Once a plugin is submitted for the first time, it must be reviewed by the goatcorp plugin review team. Because we have no guarantee that the code for the plugin matches the binary that was submitted, we must manually decompile and cross-reference the code.

Plugin updates also suffer from this process, but experience less oversight. This means that a developer could potentially deploy malicious code within an update, and it is unlikely to be caught.

To fix this, I would like to propose a design for an automated build and submission pipeline, integrated with GitHub Actions, that can ease this process and help us scale up to accommodate more developers.

Upon completion, this should allow for developers to easily submit new plugins for review, check on the status of their plugin, and release updates, without having to build and package their own plugins. On the goatcorp side, it should ease the review process for initial plugin submission, reduce the overhead of reviewing new code, and remove the need to decompile incoming plugins (as what we receive is what we build is what we publish).

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

A new GitHub repository will be created. In this repository, each file corresponds to a submitted plugin and contains a JSON/TOML/YAML structure that looks something like this:

```json
{
    "repository": "https://github.com/philpax/plogonscript.git",
    "commit": "7ba9de4049066218f67e260e2c0f8ec0651ffed7"
}
```

The `repository` refers to a Git repository (not necessarily hosted on Git), and the `commit` refers to a specific commit to build and deploy. For this to work, the repository must be structured to be amenable to automatic builds and packaging. Details of this will be covered in [reference-level-explanation].

When submitting a plugin for the first time, the developer creates the file with the relevant details filled in, and creates a PR. A GitHub Action will retrieve the repository at the specified commit, attempt to build it, and produce an artifact that can be downloaded and loaded into Dalamud for testing by a goatcorp plugin reviewer. If accepted, the PR is merged, and the deployment process follows.

When a new version of a metadata file is merged, a GitHub Action will build the repository, as with initial submission, and the resulting artifact will be deployed to DalamudPlugins automatically. The developer does not need to do anything after the metadata update has been merged.

When updating a plugin, the developer merely needs to update the `commit` for the metadata file, and make a PR. The previously mentioned steps will take place.

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

With the initial submission, proposed updates or final deployment, the GitHub Action will attempt to build the repository at the specified commit. This code is then automatically packaged with DalamudPackager, using the root-level package metadata, and this is used to produce an artifact. This build process could be later extended to feature additional verification steps (see [future-possibilities]).

The plugin directory structure should look like this:

```
to be specified
```

The `csproj` file should describe how to build the plugin, but should not feature any extraneous steps (RFC: to be defined). It should not feature the DalamudPackager step, as that will be done by the GitHub Action (RFC: is this the right call?)

The primary role of the action is to consume a (repository, commit) and produce an artifact that can be loaded by Dalamud, or be included within DalamudPlugins. DalamudPlugins will only be updated on deployment (e.g. committing of the metadata file to `main` in the repository); it will not be updated otherwise.

(RFC: I feel like I'm forgetting something here...)

# Drawbacks

[drawbacks]: #drawbacks

This will require all first-party plugin developers to abide by a common project structure that is amenable to being built by CI, and for the code of their plugins to be open-source. The latter is already true, but the former may be a difficult requirement, especially with some of the complex build arrangements that existing plugins use.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

This design has a few key benefits over competing designs:

- We don't pay for compute
- It is fully declarative (e.g. what's specified in the metadata repo is what's deployed)
- It does not require us to trust plugin developers' build systems
- It is not a significant change from our existing system

Initial discussion proposed a web portal, likely in addition to existing goatcorp services, where developers could submit their plugins to be reviewed. While this would be nice to have, it's not really immediately necessary.

An alternative design was proposed by [Ay](https://github.com/ayyaruq) where plugin developers, upon main repo approval, would be given keys exclusive to the plugin that could be used to sign future updates. This would allow new versions of plugins to be deployed without explicit goatcorp approval. However, this solution was put aside due to its relative complexity for limited payoff.

If we do not implement a solution like this, plugin submission and review will remain tedious, which will cause an increasing amount of frustration as we continue to attract new plugin developers and thus more plugins and updates. Each step in the packaging process is a potential failure point, and supporting a developer through a failure point takes valuable time. By removing this friction, we can make the experience better for both developers and goatcorp.

# Prior art

[prior-art]: #prior-art

RFC: Got some cool prior art? Ay mentioned working on something similar in the past.

# Unresolved questions

[unresolved-questions]: #unresolved-questions

- What should the directory/project structure look like for a plugin under this system? Should we even be enforcing it?
- What should be allowed in a plugin `csproj`?
- Should the developer specify the packaging step within their `csproj`, or should we do it for them?
- Do we want to automatically approve updates to already-submitted plugins?

# Future possibilities

[future-possibilities]: #future-possibilities

This would form the backbone of any future CI we do around plugin submission. Ideas for improvement include:

- Implementing a web frontend for the plugin submission/review/update process. Instead of manually PRing up changes to the metadata repo, a plugin developer could pop over to the frontend and specify which commit they'd like deployed, then hit the button.
- As we're building the code, we have an opportunity to analyse it and ensure that it isn't doing anything outright naughty. We would likely use a solution similar to [S&box](https://wiki.facepunch.com/sbox/AccessList)'s. This could be combined with other steps, like asking the plugin to declare the namespaces it will use.
- This RFC does not currently describe a way to disable a plugin. The metadata structure could be extended with an `enabled: bool` that will govern whether or not the plugin is blacklisted, thus allowing both goatcorp and plugin developers to blacklist plugins with ease.