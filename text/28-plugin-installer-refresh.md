- Feature Name: Refresh Plugin Installer
- Start Date: 2022-05-07
- DIP PR: [goatcorp/DIPs#0000](https://github.com/goatcorp/DIPs/pull/28)
- Repo-Relevant Issue: [goatcorp/dalamud#0000](https://github.com/goatcorp/dalamud/issues/0000)

# Summary

[summary]: #summary

Refactor the plugin installer to make it easier to maintain and improve ease of use.

# Motivation

[motivation]: #motivation

The plugin installer code base is known to be messy due to significant enhancements being layered on overtime. This has likely introduced reluctance to touch the code and make further improvements. There are also some QOL features that could be introduced to make the plugin installer a better experience for our users and reduce the related support efforts.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

The following code improvements will be made:
- Simplify code (many redundant methods, classes, etc.)
- Abstract out logic from plugin installer window back into plugin manager and service classes.

The following new features will be added:
- Add toggles for plugins.
- Allow alternative plugin installer views.
- Add condensed plugin view.
- Provide built-in plugin config backup and restore.
- Provide more info on plugin behavior.
- Reorganize existing layout.
- Allow setting individual plugins to use testing versions (rather than all or nothing).
- Allow disabling without uninstalling.
- Ensure is controller friendly.
- Add plugin manifest localization.

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

This may require coordination with:
https://github.com/goatcorp/DIPs/issues/4
https://github.com/goatcorp/DIPs/issues/10

Feature: Add toggles for plugins
- Toggle on/off all plugins.
- Toggle on/off all 3PP plugins.
- Toggle on/off custom sets of plugins.

Feature: Provide more info on plugin behavior
- Start with manually reporting for now.
- Show if plugin modifies the native ui (for streamers).
- This can be enhanced to check imports and more.

Feature: Reorganize existing layout
- Dalamud and plugin change logs could be separated.
- Some plugin installer settings could be easier to access from the installer window.

Feature: Localize Plugin Meta Data
- Show punchline, description and other in dalamud language if provided in manifest.

# Drawbacks

[drawbacks]: #drawbacks

This will be a little tedious.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

If we don't do this, the plugin installer will slowly get worse and harder to maintain overtime.

# Prior art

[prior-art]: #prior-art

N/A

# Unresolved questions

[unresolved-questions]: #unresolved-questions

- Do we want to keep plugin categories? They aren't used by many plugins now and introduce a lot of code / UI space. If we want to keep them, we could review the current categories and then make them mandatory for plugins.
- Are there any other features we want to add or remove from the installer?

# Future possibilities

[future-possibilities]: #future-possibilities

N/A
