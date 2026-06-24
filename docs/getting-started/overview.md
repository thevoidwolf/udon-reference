# Overview

## The pieces

When you build a VRChat world that has scripted behaviour, four things are involved:

```
   .cs source       →     U# compiler     →   Udon Program Asset  →   Udon VM
   (your code)            (build-time)         (.asset on disk)        (runtime)
        │
        └────── attached to ───→  UdonSharpBehaviour (proxy, MonoBehaviour)
                                          ↕  paired
                                 UdonBehaviour       (backing, hidden)
                                          │
                                          └──── component on a GameObject in scene
```

### UdonBehaviour
A Unity `MonoBehaviour` shipped by the VRChat SDK. It holds a reference to an **Udon Program** (the bytecode), an array of **public variables**, and a list of subscribed events. At runtime, the VRChat client iterates these and dispatches the appropriate events into the VM. **[Docs]** <https://creators.vrchat.com/worlds/udon/>

### Udon Program Asset
A serialized asset on disk that contains compiled Udon Assembly bytecode plus type metadata. When you write a U# script, the compiler generates one of these next to your `.cs` file (an `.asset` paired by GUID).

### Udon VM
A register-based virtual machine inside the VRChat client. It executes Udon Assembly opcodes. The VM's API surface is restricted to an internal **allowlist** of methods on .NET / Unity / VRChat SDK types. Calls outside the allowlist either fail to compile (UdonSharp catches them at build time) or throw at runtime.

### UdonSharpBehaviour (the proxy)
Your C# class derives from `UdonSharp.UdonSharpBehaviour`. It is itself a `MonoBehaviour` so Unity can serialize its fields, draw an inspector, and survive prefab operations. It also holds a hidden reference to its **backing** `UdonBehaviour` — the runtime component on the same GameObject that actually executes the compiled program.

At runtime, only the backing exists. Inspector edits flow from proxy → backing via `UdonSharpEditorUtility.CopyProxyToUdon`.

## Lifecycle in one sentence

> An UdonBehaviour is, conceptually, a MonoBehaviour whose code is bytecode interpreted at runtime instead of native CIL, restricted to a security allowlist, and shipped with a built-in networking model.

That sentence captures three out of four interesting points. The fourth is that Udon has *no* compile-time per-script type system across UdonBehaviours — they look at each other through `UdonBehaviour` references (or `UdonSharpBehaviour` references in U# which the compiler resolves), with method calls dispatched by string name (`SendCustomEvent`). This is why U#'s networking and event APIs are string-based rather than delegate-based.

## What UdonSharp does for you on top of that

- Translates ordinary-looking C# into the same Udon Assembly that the graph editor produces, so existing world projects mix freely.
- Generates **strongly-typed cross-behaviour calls** — calling a method on another `MyScript` reference gets compiled to the right `SendCustomEvent` plumbing.
- Enforces the runtime allowlist **at compile time**, so you find out about unsupported APIs in your IDE rather than at upload.
- Adds inspector-quality serialization (via the bundled Odin Serializer in editor) so you can drag references onto behaviours.

## What it does *not* do

- It is **not** a Unity sandbox. You're still in the regular Unity Editor and can write arbitrary `#if UNITY_EDITOR` code for editor tooling that runs outside Udon.
- It is **not** a general C# subset. Familiar features (`async`/`await`, generics in user code, LINQ, reflection, threads) are absent. See [Compiler constraints](../compiler/README.md).
- It is **not** managed by VRChat's allowlist directly — the U# compiler tries to track it, but the runtime is authoritative.

## Two layers of API

When working in U#, the API surface you call comes from two distinct sources:

1. **UdonSharp's own API** — the `UdonSharp.UdonSharpBehaviour` base class, the U#-specific attributes, and the editor utilities. Documented in the [UdonSharpBehaviour](../udonsharp-behaviour/README.md) and [Attributes](../attributes/README.md) and [Editor API](../editor-api/README.md) sections of this reference.
2. **The VRChat SDK** — `VRC.SDKBase.*`, `VRC.SDK3.*`, and the various network components. Most of the practical surface lives here: `VRCPlayerApi`, `Networking`, `VRCObjectSync`, `VRCObjectPool`, `VRCStringDownloader`, `VRCImageDownloader`, `VRCJson`, the video players, etc. This reference documents these where they intersect Udon — for some you'll occasionally want to consult <https://creators.vrchat.com/worlds/udon/> directly.

UdonSharp is the bridge that lets you call SDK methods *as if* they were ordinary .NET methods. The compiler maps them to opcodes the VM understands.

## Where to go next

- New to U# entirely → [Installation](installation.md) → [First script](first-script.md).
- Already have a project, want the lifecycle / event surface → [UdonSharpBehaviour](../udonsharp-behaviour/README.md).
- Trying to figure out what's allowed vs forbidden → [Compiler constraints](../compiler/README.md).
