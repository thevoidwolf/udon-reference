# UdonSharpBehaviour

`UdonSharp.UdonSharpBehaviour` is the abstract base class every U# script inherits from. It is itself a `UnityEngine.MonoBehaviour` so Unity treats it as a regular component for inspector / prefab / serialization purposes; the U# compiler additionally emits runtime code that the Udon VM executes via a paired backing `UdonBehaviour`.

This section enumerates every member of the base class and every overridable lifecycle event the VRChat runtime can dispatch.

## Pages

| Page | What's in it |
|---|---|
| [Lifecycle Events](lifecycle-events.md) | Unity-style and Udon-specific virtual events (Start, Update, OnPlayerJoined, OnPreSerialization, etc.) |
| [Input Events](input-events.md) | `InputJump`, `InputUse`, axis events, `UdonInputEventArgs`. |
| [Player Events](player-events.md) | Player-trigger / player-collision / spawn / respawn / avatar callbacks. |
| [Interaction Events](interaction-events.md) | `Interact()`, `DisableInteractive`, `InteractionText`, pickup events. |
| [Video Events](video-events.md) | `OnVideoStart`, `OnVideoEnd`, etc. |
| [MIDI Events](midi-events.md) | `MidiNoteOn`, `MidiNoteOff`, `MidiControlChange`. |
| [Networking Events](networking-events.md) | `OnPreSerialization`, `OnDeserialization`, ownership callbacks. |
| [API Methods](api-methods.md) | `SendCustomEvent`, `SendCustomNetworkEvent`, `RequestSerialization`, `GetProgramVariable`, `SetProgramVariable`, delayed-event helpers. |
| [Type Identity](type-identity.md) | `GetUdonTypeID`, `GetUdonTypeName` — runtime type checks for U# classes. |

## Inheritance

```
UnityEngine.MonoBehaviour
    └── UdonSharp.UdonSharpBehaviour    (abstract)
            └── YourScript              (concrete)
```

Two interfaces are also implemented for serialization:

- `UnityEngine.ISerializationCallbackReceiver` — drives the proxy ↔ backing data flow through `OnBeforeSerialize` / `OnAfterDeserialize`.
- `VRC.Udon.Serialization.OdinSerializer.ISupportsPrefabSerialization` — lets U# fields serialize types Unity doesn't natively (jagged arrays, nested collections, polymorphic refs).

Both are normally invisible to your code.

## Skeleton

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDKBase;

[UdonBehaviourSyncMode(BehaviourSyncMode.None)]
public class MyScript : UdonSharpBehaviour
{
    // Inspector field
    public Transform target;

    // Synced field — see [UdonSynced]
    // [UdonSynced] public int counter;

    // Unity-style lifecycle
    void Start() { }
    void Update() { }

    // Udon-specific lifecycle
    public override void Interact() { }
    public override void OnPlayerJoined(VRCPlayerApi player) { }
}
```

## Compile-time vs runtime

- **At edit time** your `MyScript` is a real `MonoBehaviour`. Editor scripts can reflect over it normally.
- **At build time** the U# compiler walks the class and emits an Udon Program Asset (`.asset`).
- **At runtime** the VM executes the program; your C# code as written is *not* invoked directly. Instead the compiler-generated assembly is.

This indirection has two consequences worth internalising:

1. **Constructors don't run.** `UdonSharpBehaviour` subclasses cannot have parameterised constructors; field initializers are evaluated at compile time and baked into the asset.
2. **Reflection doesn't work at runtime.** Use `GetProgramVariable` / `SetProgramVariable` / `SendCustomEvent` instead.

See [Compiler constraints](../compiler/README.md) for the full list of write-time rules.

## Source

`Packages/com.vrchat.UdonSharp/Runtime/UdonSharpBehaviour.cs` defines every member documented here.

## See also

- [Attributes](../attributes/README.md) — `[UdonSynced]`, `[UdonBehaviourSyncMode]`, etc.
- [Networking](../networking/README.md) — how `SendCustomNetworkEvent` / `RequestSerialization` actually move bytes.
- [Editor API](../editor-api/README.md) — proxy/backing utilities for writing custom inspectors.
