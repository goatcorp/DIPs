- Feature Name: `lockdown-mode`
- Start Date: 2023-01-02
- DIP PR: [goatcorp/DIPs#59](https://github.com/goatcorp/DIPs/pull/59)
- Repo-Relvant Issue: [goatcorp/DIPs#58](https://github.com/goatcorp/DIPs/issues/58)

# Summary
Introduce "Lockdown Mode" to Dalamud, a sandbox feature for Dalamud.

# Motivation
It's simply infeasible to manually review plugins won't do any harms to your system and currently one and only one slip is  enough to do harmful things on your system.

By launching FFXIV inside an app container and limiting what processes inside the container can do, we can trust that even malicious plugins won't do anything bad to user's system.

# Reference-level explanation
On launch time:
1. [Create a new app container](https://learn.microsoft.com/en-us/windows/win32/api/userenv/nf-userenv-createappcontainerprofile) with the id of `Dalamud.Container` and [derive the assigned SID](https://learn.microsoft.com/en-us/windows/win32/api/userenv/nf-userenv-deriveappcontainersidfromappcontainername) from it.
2. Set [capabilities](https://learn.microsoft.com/en-us/windows/uwp/packaging/app-capability-declarations) and [DACL](https://learn.microsoft.com/en-us/windows/win32/secauthz/dacls-and-aces) on files appropriately. (See below)
3. Launch `ffxiv_dx11.exe` using the SID derived from (1) to create the process it inside the app container.

This process will be implemented on `Dalamud.Injector` as this is the process responsible for creating `ffxiv_dx11.exe`.

## Capabilities
Capabilities granted on the app container is as follows.

| Capability | Reason |
|------------|----------|
| SECURITY_CAPABILITY_INTERNET_CLIENT_SERVER | To communicate with the game server and some plugins might act as a local server (not game related fyi) as well. |
| SECURITY_CAPABILITY_PRIVATE_NETWORK_CLIENT_SERVER | Some plugins might communicate with devices attached to home network. |

## File System Access
DACLs we set on the file system to support the feature is as follows:

> [!note]
> As a reminder, `deny` takes precedence over `allow` policy. (e.g. If `user1` is granted rw access on `/foo/bar` but also denied write access on `/foo/bar` then `user1` won't be able to write on `/foo/bar`)

| Path | Access Type | Outcome | Reason |
|------|-------|-------------|---------|
| $base_game | r-x | allow | This is self-explaintory. |
| $game_config | rw- | allow | This is self-explaintory. |
| $game_config/downloads | -w- | deny | This directory is where patch files are stored by the retail launcher and thus should deny write access to prevent sandbox escaping by dropping malicious .patch files. |
| $xl | rwx | allow | Contains plugin modules(.dll), plugin configs, dalamud logs, etc. |
| $xl/{addon, runtime, patches} | -w- | deny | Mostly same reason as $game_config/downloads except it's XL this time. |
| $dalamud | r-x | allow | Normally this directory's policy is automatically inherited (CONTAINER_INHERIT_ACE) from $xl but it's possible that's not the case when people are building Dalamud manually. |

Anything not listed here follows regular access check rules for appcontainers.

# Drawbacks
- Some plugins may not be compatible without changes.
- Current implementation is incomplete. (See [[##Unresolved questions]])

# Prior art
- There have been attempts to limit the scope of what plugins could do. (See [#53](https://github.com/goatcorp/DIPs/issues/53))
	- However, I don't believe hooking .NET APIs is the right approach as APIs you need to secure is astronomical and is extremely hard (if ever possible) to get it right.
	- Not to mention .NET BCL already tried a similar thing with `AppDomain` then later it got abandoned and is replaced wih `AssemblyLoadContext`.

# Unresolved questions
Semi-working prototype("Proof of Concept") is available for testing on [here](https://github.com/goatcorp/Dalamud/pull/1048). Use [this plugin](https://github.com/Minoost/Dalamud.TotallyLegitPlugin) to launch a terminal and test whatever hypothesis you might have from there.

Mind you it has few caveats. Notably:
- `$base_game` directory is usually owned by an administrator with no ways to change the permssion as a normal user.
	- This will be addressed eventually however is out of scope for this DIP. Just take the ownership of this folder for now.
- A process created inside the container has low integrity level (low IL from now) by default.
	- This can be seen as an additional layer of defense as low IL process won't be able to read from (if NO_READ_UP is set) or write to (if NO_WRITE_UP is set) files with higher IL.
	- However, this also means that sandboxed ffxiv won't be able to write configs as files have NO_WRITE_UP by default.
- What should we do about the screenshot directory?
	- We had an idea where the launcher would parse `ScreenShotDir` then give write access to it but discarded as malicious abuse it to gain write access to any directory. (Malicious plugins can simply write into ffxiv config to get write access as they wish)
- You can't connect to `localhost` from AppContainer. This means some plugins **will be incompatible** without some changes.
	- This CAN be worked around using [NetworkIsolationSetAppContainerConfig](https://learn.microsoft.com/en-us/windows/win32/api/netfw/nf-netfw-networkisolationsetappcontainerconfig) function. (See [this blog post](https://googleprojectzero.blogspot.com/2021/08/understanding-network-access-windows-app.html) for more details)
	- However, using this to allow loopback is out of scope for a MVP.

To side step these issues this feature will only be available if `DALAMUD_EXPERIMENT_APP_CONTAINER` is set to `1`.

# Future possibilities
- Some issues (like screenshots, localhost mentioned above) might need help of special process (also known as a "broker") to properly solve it.
	- A process running inside the container would ask the broker (through some kind of IPC channel) to do something sandboxed process can't do by itself.
	- This is what modern web browsers do and is proven effective against a compromised process if it's already sandboxed.
	- However, going with this approach is also no small amount of work. Does it really worth it? Why don't just tell user to use default location or PrtScr? (or your favorite screen capturing tools, really...)
- Windows let you create another AppContainer inside an app container. (DinD, except this time it's called "Nested AppContainer")
	- Can we leverage this to provide per-plugin isolations?

# Helpful Resources
- [Windows Internals, Part 1, 7th Ed.](https://learn.microsoft.com/en-us/sysinternals/resources/windows-internals)
- https://learn.microsoft.com/en-us/windows/win32/secauthz/appcontainer-isolation
- https://chromium.googlesource.com/chromium/src/+/HEAD/docs/design/sandbox.md#App-Container-low-box-token
- https://wiki.mozilla.org/Security/Sandbox#File_System_Restrictions
- https://googleprojectzero.blogspot.com/2021/08/understanding-network-access-windows-app.html
