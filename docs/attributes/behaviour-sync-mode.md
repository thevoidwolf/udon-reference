# `[UdonBehaviourSyncMode]` and `BehaviourSyncMode`

Locks a single sync configuration onto a U# behaviour at the class level. Defined in `Runtime/UdonSharpAttributes.cs`. **[Source]**

## Signature

```csharp
[AttributeUsage(AttributeTargets.Class)]
public class UdonBehaviourSyncModeAttribute : Attribute
{
    public BehaviourSyncMode behaviourSyncMode;
    public UdonBehaviourSyncModeAttribute(BehaviourSyncMode behaviourSyncMode);
}
```

## Usage

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.Manual)]
public class Scoreboard : UdonSharpBehaviour { }
```

The compiler enforces `[UdonSynced]` type-mode compatibility and hides irrelevant editor UI based on the value.

## `BehaviourSyncMode` values

```csharp
public enum BehaviourSyncMode {
    Any,
    None,
    NoVariableSync,
    Continuous,
    Manual,
}
```

| Value | Effect |
|---|---|
| `Any` | No enforcement (default if attribute omitted). User picks Continuous/Manual in the inspector. |
| `None` | No synced variables. `SendCustomNetworkEvent` will **not** work. Sync dropdown is hidden. |
| `NoVariableSync` | No synced variables, but `SendCustomNetworkEvent` still works. |
| `Continuous` | Forces continuous sync. Replicated automatically at the runtime's tick rate. Suitable for frequently-changing values. |
| `Manual` | Forces manual sync. Owner calls `RequestSerialization()` to broadcast. Required for `RequestSerialization()` to do anything. |

## Choosing a mode

- **`Continuous`** when the value naturally changes every frame or two: smoothly moving objects, animation parameters, rotating elements. The runtime sends incremental updates at a few times per second; combined with `Linear`/`Smooth` interpolation on the field, this gives smooth visual motion.
- **`Manual`** when the value is a discrete, important state change: a score increment, an inventory pick-up, a toggled switch. Owner commits a packet by calling `RequestSerialization()`. Late joiners receive the most recently broadcast state.
- **`NoVariableSync`** for behaviours that need to fire networked events but have no state to replicate (e.g. a "press to ring bell" trigger that only invokes a remote method).
- **`None`** for truly local behaviour: HUD, local-only animation, debug tools. Slightly lighter at edit time.
- **`Any`** when you genuinely want both options available per-instance. Rare in practice.

## Mixing on one GameObject

Different `UdonSharpBehaviour` components on the **same GameObject** can use different sync modes. Ownership is per-GameObject, so all behaviours on it share an owner, but each behaviour has its own packet stream.

## Errors the compiler emits

The compiler flags these mismatches:
- `[UdonSynced]` field on a class declared `BehaviourSyncMode.None` or `NoVariableSync`.
- `[UdonSynced(UdonSyncMode.Linear)]` / `Smooth` on a non-numeric type.
- Calling `RequestSerialization()` from a class declared `BehaviourSyncMode.Continuous` (the call is a no-op).

## See also

- [`[UdonSynced]`](udon-synced.md)
- [Networking → sync modes](../networking/sync-modes.md)
- [UdonSharpBehaviour → API methods](../udonsharp-behaviour/api-methods.md) — `RequestSerialization`, `SendCustomNetworkEvent`.
