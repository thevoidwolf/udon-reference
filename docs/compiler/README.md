# Compiler

UdonSharp accepts a *subset* of C#. This section enumerates what's supported, what isn't, and how to work around the absences.

## Pages

| Page | Topic |
|---|---|
| [Supported features](supported-features.md) | Language features and APIs that work. |
| [Unsupported features](unsupported-features.md) | What the compiler rejects and why. |
| [Workarounds](workarounds.md) | Common substitutions for missing features. |

## Two distinct allowlists

When U# rejects something, it's because of one of two allowlists:

1. **The U# compiler's own language allowlist.** Things like `async`/`await`, generic user code, ref-struct types. The compiler errors at build time.
2. **The Udon VM's runtime allowlist.** Things like `Resources.Load`, `File.WriteAllText`, `Reflection.Emit`. The compiler also catches most of these via static analysis, but ultimately the VRChat runtime is authoritative.

In practice you don't think of them separately — both surface as compile errors. But knowing the distinction helps when you read older community advice that talks about "extern allowlist" updates: the runtime allowlist evolves over VRChat client versions.

## How to think about it

> *"Most C# works. Most Unity API surface works. The exceptions are: generics in user code, anything I/O-related, reflection, async, threading, native interop, AddComponent."*

Memorize that and you'll be right ~95% of the time.

## Community tooling

| Tool | What it does | Status |
|---|---|---|
| **`mika-f/udon-analyzer`** | Roslyn analyzer: catches unsupported APIs, perf issues, `SendCustomEvent` ↔ handler mismatches the U# compiler misses. **[Community]** <https://github.com/mika-f/udon-analyzer> | Active |
| **`esnya/UdonRabbit.Analyzer`** | Older Roslyn analyzer (55 rules `URA0001`–`URA0055`). **[Community]** <https://github.com/esnya/UdonRabbit.Analyzer> | **Archived** — no U# 1.0+ support |
| **`refiaa/vscode-udonsharp-linter`** | VS Code linter extension. **[Community]** <https://marketplace.visualstudio.com/items?itemName=refiaa.vscode-udonsharp-linter> | — |
| **`BlueAmulet/UdonSharpOptimizer`** | Post-compile bytecode optimizer (COPY removal, tail-call optimisation, temp-var reuse). Reduces instructions and improves runtime. **[Community]** <https://github.com/BlueAmulet/UdonSharpOptimizer> | Active |

MerlinVR's **U# 1.2.0-b1** is a *separate, unofficial pre-release*, not the U# bundled with the Worlds SDK: its install steps tell you to delete the bundled `Packages/com.vrchat.worlds/Integrations/UdonSharp` folder and drop it in instead. It adds compiler-level "fake generics" — `List<>`, `Dictionary<>`, `HashSet<>`, `Queue<>`, `Stack<>` work for concrete custom types with custom `GetHashCode`/`Equals`, **but not** when stored as `object` or generic-typed references. Structs and inheritance on non-behaviour types remain unsupported. Replacing a file inside a VPM-managed package is overwritten on SDK updates. **[Community]** <https://github.com/MerlinVR/UdonSharp/releases/tag/v1.2.0-b1> Everything else in this section describes the **bundled** U# (verified against the SDK 3.10.3 copy, with 3.10.4 to 3.10.6 changes diffed from the packages; release notes through 3.10.6).

## SDK releases that changed the compiler

UdonSharp ships with the Worlds SDK, so compiler changes arrive in SDK releases. **[Docs]** <https://creators.vrchat.com/releases/>

| SDK | Change |
|---|---|
| 3.4.0 (Oct 2023) | UdonSharp (and ClientSim) merged into `com.vrchat.worlds` → `Integrations/UdonSharp/`. |
| 3.7.6 (Mar 2025) | Runtime log watcher updated for VRChat's new log file format. |
| 3.8.1 (May 2025) | `[NetworkCallable]` attribute for network events with parameters. See [Attributes](../attributes/README.md). |
| 3.9.0 (Oct 2025) | U# scripts recompile when you switch build platform. |
| 3.10.2 (Feb 2026) | `enabled` / `isActiveAndEnabled` allowlisted on everything deriving from `UnityEngine.Behaviour` (see [Supported features](supported-features.md#unity-api-surface-broad-strokes)). `VRChat SDK/Udon Sharp/Parse Logs from File` tool. |
| 3.10.4 (Jun 2026) | Fixed enum usage miscompiling for enums with a specific number of members. `UdonSharpBehaviour` implements `IUdonEventReceiver`. |
| 3.10.5 (Sep 2026) | Assembly Version Defines usable in U# code (e.g. `#if SDK_310_OR_NEWER`) — see [Assembly Definition](../editor-api/assembly-definition.md#assembly-version-defines-sdk-3105). U# scripts can be created under `Packages/` again. Asset importing paused during U# updates (collapses reimports). |
| 3.10.6 (Oct 30, 2026) | Relational operators (`<`, `<=`, `>`, `>=`) on **user-defined** enums no longer compile as `!=`. Named arguments that skip optional parameters (`M(1, c: 3)`) no longer drop the argument. U# inspectors for classes with `Interact()` get an **Outline Renderers** list. See [Supported features](supported-features.md#types). |
| 3.11.0-beta.1 (Nov 2026, **beta**, Unity 6) | Relational operators on **extern** (Unity/VRChat) enums fixed too. Social Hooks event stubs added to `UdonSharpBehaviour` (behind `VRC_ENABLE_SOCIAL_HOOKS`). Harmony patches updated for Unity 6 APIs. New `Editor/UpgradeWizard/ForceUpgradeStep.cs` adds a "Force and Torque Changes" step to the SDK's Unity 6 Upgrade Wizard. It finds U# scripts (by regex over the `.cs` source) and Udon Graphs under `Assets/` that call `AddForce`/`AddTorque`/`AddRelative*`/`AddForceAtPosition`/`AddExplosionForce`, so you can fix them by hand. It does not rewrite them. **[Source]** See [SDK 3.11 beta preview](../references/sdk-3-11-beta.md). |

## See also

- [GameObject lifecycle → runtime constraints](../gameobject-lifecycle/runtime-constraints.md) — the same constraints from a "what can I do at runtime" angle.
- [Attributes](../attributes/README.md) — language additions specific to U#.
- [References → Community Resources](../references/community-resources.md) — full analyzer/optimizer listing.
