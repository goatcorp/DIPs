- Feature Name: `modules`
- Start Date: 2026-01-18
- DIP PR: [goatcorp/DIPs#0000](https://github.com/goatcorp/DIPs/pull/0000)
- Repo-Relevant Issue: [goatcorp/dalamud#0000](https://github.com/goatcorp/dalamud/issues/0000)

# Summary

[summary]: #summary

As Dalamud is growing in scale and complexity, its core is becoming heavier and harder to bring up after patches.
We want to implement infrastructure to transform it into a modular framework, where not all code has to run at the same time and plugins can dynamically link to other modules.

# Motivation

[motivation]: #motivation

Dalamud's core is getting quite large. Every feature we add makes the framework heavier, which means longer patch-day recovery times and more complexity for plugin developers. Right now, if you're writing a plugin, you're pulling in the entire Dalamud runtime whether you need all of it or not. We want to break this monolithic structure apart into smaller, composable pieces that can be loaded on-demand.

At the same time, plugin developers are already sharing code between their projects, but they're doing it through git submodules, copy-paste, or custom IPC systems because we don't give them a better option. A proper module system solves both problems: it lets us slim down core by making features opt-in, and it gives developers a standard way to build and share reusable functionality. The end goal is a lighter Dalamud that's easier to maintain and a cleaner development experience for everyone building on top of it.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

## Module Plugins

Dalamud can load __module plugins__ that expose services to other plugins, but don't offer functionality to users directly.

## Obtaining Services from Module Plugins

Plugins can obtain services from module plugins through dependency injection in the same way that Dalamud services are obtained right now.

## Core Dalamud as Modules

Core Dalamud services are slowly moved into module plugins, with the Dalamud shell being implemented as a plugin. Dalamud services and module plugins provided by community developers use the same technology and infrastructure.

## What should be a module plugin

Libraries that share data or processing, and need to make it available to multiple plugins, are ideal to be converted to module plugins. If your library only provides types or functionality that does not benefit from being shared across multiple plugins, it's generally easier to continue using NuGet packages or submodules.

Examples that make sense to be modules: 
* Library using a large amount of hooks to provide functionality to subscribers
* Library processing a large amount of data that should not be duplicated
* Library facilitating communication between plugins

Example that do not make sense to be modules:
* Library containing helper functions
* Library providing functionality that is largely standalone and can support being loaded in multiple plugins

## Automatic Dependency Management

The plugin manager automatically resolves and installs dependency modules, **users will not choose or install module plugins by hand**.

Module plugins do not list in the installer together with regular plugins, they receive their own section that lists what depends on them and offers updates.

Module plugins cannot be enabled, disabled or uninstalled from the installer, the plugin manager automatically disables and uninstalls them once no plugin is enabled or installed that depends on the module plugin.

## Developer Experience

The developer experience for module plugins is similar to regular plugins, they show up as dev-plugins and override instances of the module plugin that would come from the repo.

When module plugins are hot reloaded, all plugins that depend on them must reload as well, as services cannot be reinstantiated while dependent plugins are loaded.

Plugins themselves still cannot depend on other plugins.

## Referencing Module Plugins

Module Plugins are referenced through a Dalamud.NET.Sdk MSBuild item that converts them into NuGet package references, to obtain reference assemblies, and instructs DalamudPackager to generate a list of required module plugins that is added to the plugin manifest and can be used by clients to resolve and install module plugins.

The supported syntax for version ranges mirrors [NuGet syntax for version ranges](https://learn.microsoft.com/en-us/nuget/concepts/package-versioning?tabs=semver20sort#version-ranges) exactly.

```xml
<ItemGroup>
  <DalamudModuleReference Include="Foo" Version="1.*" />
</ItemGroup>
```

## Creating and Distributing Module Plugins

Developers can offer module plugins that implement required functionality that must or can be shared between multiple plugins.

Third-party repositories can offer module plugins, but if a module plugin from a third-party repository is to be used, the repository that contains the module plugin must first be added by the user.

Reference assemblies are generated when a module plugin is released, and published through NuGet. A module plugin is referenced like a NuGet package with a version range through your csproj, which automatically adds it as a dependency to the manifest by DalamudPackager and informs Dalamud that the module plugin is required to instantiate a plugin.

CI infrastructure (GitHub Actions) to release module plugins to NuGet is provided by us.

Module plugins on the main repository must be released through Plogon, D17 will be amended appropriately (TODO - separate folder, provide nuget setup for trusted publishing on dev account, etc.).

## Versioning for Module Plugins

Module plugins must follow semver conventions for versioning. CI infrastructure provided by us will fail or warn when making releases that break API without increasing the major version.

It is encouraged to only increase the major version during an API bump, but compatibility can also be broken during a release cycle, with conflicts being handled by the resolver as seen in the next section. If all plugins that depend on the module plugin are already updated and compatible, the major is effectively invisible to users.

### Conflict Resolution

The plugin installer notifies users about conflicts when updates cannot be installed cleanly.

The resolvers' goal is to always update as many plugins as possible to the latest version, at the cost of disabling plugins incompatible with the version of the module that was resolved. The plugin that supports the highest version of the module plugin always wins - plugins that do not support the highest version of the module in the resolution are disabled and marked as outdated. Users are shown a message akin to "This plugin is disabled because Plugin A and Plugin B depend on a newer version of Module X. Disable them to re-enable this plugin."

This approach has been chosen for now to reduce complexity in the resolver, and to reduce the complexity users are confronted with when compatibility breaks.

### Keeping compatibility

When expecting to iterate fast on APIs, while wanting to keep compatibility for consumers, it's possible to introduce a layer of indirection into the module dependency chain which wraps a "core module" that implements functionality and shims its API.

```
FooCore v5.0.0
  -> FooApi v5.0.0 (directly shims to FooCore v5 APIs)

FooCore v6.0.0
  -> FooApi v5.0.1 (updated to reference FooCore v6 and translates API)
  -> FooApi V6.0.0 (directly shims to FooCore v6 APIs)

=> Plugins that reference FooApi v5.* keep working
```

> [!NOTE]  
> Goat: This wouldn't work as is with how the spec is written right now, since an instance of a module can only exist once. If we want to support this, we would have to establish a way to remove that restriction or find another solution.

## Module Plugin Dependencies

Module plugins can depend on other module plugins in the same manner plugins can depend on module plugins.

Module plugins can break API without a Dalamud API bump, but it is generally not recommended as it creates friction for users - all plugins that depend on the library must be updated to function at the same time.

## Banned Module Plugins

Module plugins can be banned like any other plugin. Plugins that depend on a banned module plugin do not load.

If a version of the module plugin that is not banned resolves, a message is shown for applicable plugins and choosing to update said plugins (or all plugins) will install and load the resolved version of the module plugin. The user experience is akin to updating the plugin directly, with slightly different messaging.

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

## Service Lifecycle and Scoping

Services exposed to plugins are required to be scoped to the plugin and must manage their lifecycle carefully.

## Repository Pinning and Isolation

The source repository is "pinned" cryptographically, a module plugin cannot be resolved from a different repository than the one it was built toward.

If multiple plugins depend on different instances of a module plugin (e.g. Plugin A from Repo A depends on module plugin A from repo A, but plugin B from repo B depends on module plugin A from repo B due to forking), both instances of the module plugin are loaded, no matter if it uses the same internal name. The installer will list the module plugin as "Module (from repo \<name\>)" and the modules are effectively treated as if they were distinct.

## Version Constraints

It is not possible to load multiple versions of the same module plugin at the same time. The resolver must choose the best version.

# Drawbacks

[drawbacks]: #drawbacks

- We need to handle dependency resolution, conflict detection, version matching, and transitive dependency graphs. The installer needs to figure out which modules to pull in, handle cases where plugins want different versions of the same module, and gracefully fail when things don't resolve

- We will have to take care of significant CI and infrastructure overhead. Every module plugin needs automated builds, reference assembly generation, NuGet publishing, and versioning workflows. We need to set up trusted publishing for the main repo, create GitHub Actions templates for third-party devs, and integrate all of this into Plogon

- There will be a large administrative burden on core devs and PAC. Someone has to bump versions, coordinate releases between dependent modules, publish reference assemblies. A lot of this needs to be automated or it becomes a bottleneck. We'll also need clear policies on versioning, deprecation timelines, and breaking changes that might differ from what we have right now for plugins

- Users will face friction when modules break API. When a module plugin introduces breaking changes, every single plugin that depends on it needs to be updated before users can use any of them. This creates "all or nothing" update scenarios where users might be stuck without their favorite plugins until devs catch up. Unlike a Dalamud API bump where there's usually coordination and notice, module breaks can happen more frequently and affect smaller subsets of plugins, making it harder to communicate and plan around

- There is a large potential for dependency hell. Different repos hosting different versions of the "same" module means we can end up loading multiple copies of functionally identical code. This wastes memory, can cause subtle bugs if plugins expect to share state through a module, and makes debugging harder when you have to figure out which instance of a module is actually being used (though arguably this is already the case now)

- We will likely face a steeper learning curve for plugin developers. New devs now need to understand NuGet package references, version ranges and module plugin architecture on top of the existing Dalamud API. The "simple plugin" that just does one thing gets more complex to set up

- The ability to update "core" Dalamud module plugins one by one may not actually be useful to us, as the range of core modules required by plugins might be large enough to render this effectively useless. We may need to implement an analyzer that notifies you if you request a module that you are not actively using.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

The main alternative is to keep doing what we're doing now: developers share dependencies manually through submodules or NuGet packages, and coordinate at runtime using IPC or data share. This works okay for small-scale sharing, but it doesn't solve the core problem that Dalamud itself is getting too heavy. We can't modularize Dalamud's own services without a proper module system, and the current patch-day bring-up time is only going to get worse as we add more features to core.

We could try a halfway approach where we only use modules for Dalamud's internal refactoring and don't expose them to third-party developers. This would reduce the complexity around dependency resolution and third-party repos, but it misses an opportunity. Plugin developers are already trying to share code between their projects, and they're doing it in awkward ways because we don't provide infrastructure for it. If we're building a module system anyway, making it available to the ecosystem gives everyone a standard way to share functionality instead of the current mix of git submodules, copy-paste, and IPC hacks.

The status quo has a shelf life. Every patch cycle, core gets a little heavier, bring-up takes a little longer, and developers reinvent a little more shared code. This proposal is betting that the upfront cost of building proper module infrastructure is worth it to make Dalamud sustainable long-term, both for us maintaining core and for developers building on top of it. We just need to make sure that we have the manpower for this upfront, because **doing it halfway and ending up with broken garbage that just makes our life harder would be fatal**.

# Prior art

[prior-art]: #prior-art

- Factorio mods have a built-in dependency system where mods can declare dependencies on other mods, including version requirements and optional dependencies. The game's mod portal handles automatic dependency resolution and downloads. This is pretty close to what we're proposing and their experience shows that automated dependency management practically a requirement for a modding ecosystem, but also that dependency conflicts can still frustrate users when mods aren't kept up to date. Another example of this would be Kerbal Space Program mods (CKAN)
- Skyrim/Fallout modding (SKSE/F4SE) has plugin dependencies but no formal dependency management. The community relies on manual installation, load order management, and prayer. This works because the modding community is experienced and willing to troubleshoot, but it's a barrier to entry and causes endless support headaches. Our experience is better than this right now, just because our ecosystem is smaller/younger and devs are willing to coordinate (as there is a certain degree of centralization around the main repo and large plugins in the modding sphere), but it shows what happens when you punt on dependency infrastructure entirely
- Package managers for programming languages, like NuGet or npm, spent a long time solving dependency resolution. It kinda shows that even with sophisticated tooling, you still get dependency hell, breaking changes causing ecosystem-wide pain, and complex scenarios like diamond dependencies. They have lockfiles, semver conventions, and massive communities ironing out best practices. We're building a smaller-scale version of this, so we'll hit similar problems but with less tooling to help

Dependency management seems to be really valuable when it works, and really painful when it doesn't. The ecosystems that invested in proper infrastructure early (Factorio) have smoother developer and user experiences than those that didn't (Skyrim). But all of them deal with some version of dependency hell, it's just a question of who has to deal with it, devs or users.

# Unresolved questions

[unresolved-questions]: #unresolved-questions

- Does this work out in practice? Before committing, we **need** a proof of concept
- Do we want manifest/repo v2 before implementing this to make things smoother?
- Are we actually prepared for the admin effort of people moving things into modules? A lot of tooling code will need to be written to make this smooth, in CI, Plogon, etc.
- Should this DIP be split up into modules for Dalamud Core functionality, and modules for external developers?

# Future possibilities

[future-possibilities]: #future-possibilities

- Optional plugin-to-plugin dependencies
- "API layers" that can shim functionality, which would allow us to iterate more on API design without breaking compatibility with plugins and maybe even allow us to forego Dalamud API levels altogether for API versioning
- **Game itself (and CS) could be represented as a module that you have to declare a dependency on if you use interop or CS, so that you don't have to update your plugin if nothing you use actually broke**
  - ...but we could already do this now by introducing an "interop version" 
