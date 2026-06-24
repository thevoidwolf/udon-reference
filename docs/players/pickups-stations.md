# Pickups & Stations

## Pickups

`VRC.SDK3.Components.VRCPickup` is the component on any GameObject that can be grabbed by a player.

### Player-side controls

```csharp
player.EnablePickups(bool enabled);            // default true
VRCPickup p = player.GetPickupInHand(VRC_Pickup.PickupHand hand);
```

`EnablePickups` toggles the player's ability to grab pickups at all. `GetPickupInHand` returns the pickup currently held in `hand` (`Left` / `Right` / `None`) — local-player only.

### `VRCPickup` component

Typical inspector / API surface (on the pickup itself, accessed via `GetComponent`):

| Member | Type | Notes |
|---|---|---|
| `currentPlayer` | `VRCPlayerApi` | The player currently holding it, or `null`. |
| `currentHand` | `VRC_Pickup.PickupHand` | `None` / `Left` / `Right`. |
| `IsHeld` | `bool` | Convenience. |
| `pickupable` | `bool` | Globally allow / disallow picking up. |
| `proximity` | `float` | Hover-range. |
| `InteractionText` | `string` | Inherited from `UdonBehaviour`'s tooltip. |
| `Drop()` | `void` | Force the current holder to drop. |
| `ForceDrop()` | `void` | Synonymous with `Drop` in most SDK versions. |

UdonBehaviours on the same GameObject as a `VRCPickup` receive these events (see [Interaction events](../udonsharp-behaviour/interaction-events.md)):

- `OnPickup`, `OnDrop`
- `OnPickupUseDown`, `OnPickupUseUp`

All local-only.

### Auto Hold

The `VRCPickup` inspector has an "Auto Hold" mode (`Yes` / `No` / `AutoDetect`). When `Yes`, picking up the pickup automatically transfers ownership of the GameObject to the picker. Otherwise you must call `Networking.SetOwner(Networking.LocalPlayer, gameObject)` in `OnPickup`.

### Networking pickup state

Pickup position/rotation can be synced via `VRCObjectSync` (see [Networking → ObjectSync](../networking/object-sync.md)). Without it, the pickup is local-only — other players see it sitting where it was.

```
GameObject "Sword"
├── VRCPickup
├── VRCObjectSync
├── Rigidbody
├── BoxCollider
└── (your UdonSharpBehaviour for behaviour logic)
```

## Stations

A `VRC.SDK3.Components.VRCStation` is the "sit here" component. Add it to a GameObject with an animator's `Anchor` configured. The player teleports to the station and can stand up by pressing Jump.

### Programmatic entry

```csharp
public override void Interact() {
    Networking.LocalPlayer.UseAttachedStation();
}
```

`UseAttachedStation` works when called from an UdonBehaviour on the **same GameObject** as the station. The player enters that station.

### Events

UdonBehaviours on the same GameObject as the station receive:

```csharp
public override void OnStationEntered(VRCPlayerApi player) { }
public override void OnStationExited(VRCPlayerApi player) { }
```

Both fire on every client. Use the parameter to act on the right player.

### Common configuration

| Inspector field | Use |
|---|---|
| `PlayerMobility` | `Static`, `Mobile`, `Immobilize` — how the player can move while seated. |
| `canUseStationFromStation` | Allow chaining station-to-station. |
| `disableStationExit` | Player cannot leave via Jump. (Make sure you provide another way out.) |
| `stationEnterPlayerLocation` | Transform the player teleports to. |
| `stationExitPlayerLocation` | Transform where the player ends up after exit. |
| `animatorController` | An override animator applied to the player while seated. |

## See also

- [VRCPlayerApi](vrc-player-api.md)
- [Interaction events](../udonsharp-behaviour/interaction-events.md)
- [Networking → ObjectSync](../networking/object-sync.md)
