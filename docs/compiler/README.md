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
| **`esnya/UdonRabbit.Analyzer`** | Older Roslyn analyzer (55 rules `URA0001`–`URA0055`). **[Community]** <https://github.com/esnya/UdonRabbit.Analyzer> | **Deprecated** — no U# 1.0+ support |
| **`refiaa/vscode-udonsharp-linter`** | VS Code linter extension. **[Community]** <https://marketplace.visualstudio.com/items?itemName=refiaa.vscode-udonsharp-linter> | — |
| **`BlueAmulet/UdonSharpOptimizer`** | Post-compile bytecode optimizer (COPY removal, tail-call optimisation, temp-var reuse). Reduces instructions and improves runtime. **[Community]** <https://github.com/BlueAmulet/UdonSharpOptimizer> | Active |

The U# 1.2 beta1 added compiler-level "fake generics" — `List<>`, `Dictionary<>`, `HashSet<>`, `Queue<>`, `Stack<>` work for concrete custom types with custom `GetHashCode`/`Equals`, **but not** when stored as `object` or generic-typed references. Structs and inheritance on non-behaviour types remain unsupported. **[Community]** <https://github.com/MerlinVR/UdonSharp/releases/tag/v1.2.0-b1>

## See also

- [GameObject lifecycle → runtime constraints](../gameobject-lifecycle/runtime-constraints.md) — the same constraints from a "what can I do at runtime" angle.
- [Attributes](../attributes/README.md) — language additions specific to U#.
- [References → Community Resources](../references/community-resources.md) — full analyzer/optimizer listing.
