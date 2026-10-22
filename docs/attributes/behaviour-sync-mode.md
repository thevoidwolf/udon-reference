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
| `None` | No synced variables. Sync dropdown is hidden. The behaviour can't **receive** network events (`SendCustomNetworkEvent` / `NetworkCalling`). **[Docs]** |
| `NoVariableSync` | No synced variables, dropdown hidden, network events still work. Can sit on GameObjects whose other behaviours use Manual or Continuous. **[Docs]** |
| `Continuous` | Forces continuous sync. Replicated automatically at the runtime's tick rate. Suitable for frequently-changing values. **[Docs]** |
| `Manual` | Forces manual sync. Owner calls `RequestSerialization()` to broadcast. Updates are reliable when requested. **[Docs]** |

Official table: <https://creators.vrchat.com/worlds/udon/udonsharp/attributes/#behavioursyncmode>

## Choosing a mode

- **`Continuous`** when the value naturally changes every frame or two: smoothly moving objects, animation parameters, rotating elements. The runtime sends incremental updates at a few times per second; combined with `Linear`/`Smooth` interpolation on the field, this gives smooth visual motion.
- **`Manual`** when the value is a discrete, important state change: a score increment, an inventory pick-up, a toggled switch. Owner commits a packet by calling `RequestSerialization()`. Late joiners receive the most recently broadcast state.
- **`NoVariableSync`** for behaviours that need to fire networked events but have no state to replicate (e.g. a "press to ring bell" trigger that only invokes a remote method).
- **`None`** for truly local behaviour: HUD, local-only animation, debug tools. Slightly lighter at edit time. Remember it also blocks *incoming* network events, including `[NetworkCallable]` ones.
- **`Any`** when you genuinely want both options available per-instance. Rare in practice.

## Mixing on one GameObject

Ownership is per-GameObject, so all behaviours on it share an owner. Sync modes don't truly mix, though. If behaviours on one GameObject declare different modes, **all of them use the most restrictive one**: Manual + Continuous on the same object both act as Manual. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-details/#multiple-udonbehaviours> `NoVariableSync` is the mode meant to sit alongside either. Put continuous state on a separate (child) GameObject.

## Errors the compiler emits

The compiler flags these mismatches (the official docs say only that the attribute "enables additional validations to ensure that your synced variables are suitable for the chosen sync mode" **[Docs]**):
- `[UdonSynced]` field on a class declared `BehaviourSyncMode.None` or `NoVariableSync`.
- `[UdonSynced(UdonSyncMode.Linear)]` / `Smooth` on a non-numeric type.
- Calling `RequestSerialization()` from a class declared `BehaviourSyncMode.Continuous` (the call is unnecessary, since continuous sends automatically). **[Unverified]** that this is a compiler diagnostic rather than just a no-op.

## See also

- [`[UdonSynced]`](udon-synced.md)
- [Networking → sync modes](../networking/sync-modes.md)
- [UdonSharpBehaviour → API methods](../udonsharp-behaviour/api-methods.md) — `RequestSerialization`, `SendCustomNetworkEvent`.
