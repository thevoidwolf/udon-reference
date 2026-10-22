# Pickups & Stations

## Pickups

`VRC.SDK3.Components.VRCPickup` is the component on any GameObject that can be grabbed by a player. It needs a **Rigidbody** and a **Collider** on the same GameObject. **[Docs]** <https://creators.vrchat.com/worlds/components/vrc_pickup>

Adding a `VRCPickup` automatically moves the GameObject to the **`Pickup`** layer (13) — SDK 3.10.4 notes say this happens "again", i.e. it had regressed in earlier 3.x SDKs. The `Pickup` layer does not collide with players. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-4>, <https://creators.vrchat.com/worlds/layers>

### Player-side controls

```csharp
player.EnablePickups(bool enabled);            // default true
VRCPickup p = player.GetPickupInHand(VRC_Pickup.PickupHand hand);
```

`EnablePickups` toggles the player's ability to pick up and use pickups at all (on by default). `GetPickupInHand` returns the pickup currently held in `hand` (`Left` / `Right` / `None`), or `null` — local-player only. **[Docs]**

### `VRCPickup` inspector fields

**[Docs]** <https://creators.vrchat.com/worlds/components/vrc_pickup>

| Field | Notes |
|---|---|
| Auto Hold | Whether the pickup stays in hand after Grab is released — see [Auto Hold](#auto-hold). |
| Use Text | Prompt shown while held. Requires Auto Hold. |
| Interaction Text | Hover label (VR, or when looked at on desktop). Falls back to a default label if empty. |
| Proximity | Maximum pickup distance in metres (raycast from hand in VR / head on desktop; VR hover sphere is 0.4 m × avatar scale). |
| Orientation | `Any` (default) / `Gun` / `Grip`. |
| Exact Gun / Exact Grip | Transforms defining the hold pose for Gun / Grip orientation (use child transforms). |
| Pickupable | Whether the object can currently be picked up. |
| Disallow Theft | Blocks pickup while a remote player is holding it. |
| Allow Manipulation When Equipped | Lets keyboard / controller users rotate the held pickup. |
| Momentum Transfer Method | How collision force is applied via `AddForceAtPosition` (other object needs `AllowCollisionTransfer`). |
| Throw Velocity Boost Min Speed / Scale | Release-speed threshold (negative disables) and throw multiplier (1 = none). |
| Outline Renderers | (SDK 3.10.5+) Which renderers get the hover outline. Only `MeshRenderer` / `SkinnedMeshRenderer`; empty or invalid entries fall back to auto-detection. |

### `VRCPickup` script surface

Commonly used members on the component (accessed via `GetComponent`). The official component page documents only inspector fields, so these are **[Community-known]**:

| Member | Type | Notes |
|---|---|---|
| `currentPlayer` | `VRCPlayerApi` | The player currently holding it, or `null`. |
| `currentHand` | `VRC_Pickup.PickupHand` | `None` / `Left` / `Right`. |
| `IsHeld` | `bool` | Convenience. |
| `pickupable` | `bool` | Allow / disallow picking up. |
| `proximity` | `float` | Pickup range. |
| `InteractionText` / `UseText` | `string` | The pickup's own hover / use labels (not the UdonBehaviour's interact text). |
| `Drop()` | `void` | Force the current holder to drop. |

(An earlier revision listed a `ForceDrop()` method; it could not be verified and has been removed.)

UdonBehaviours on the same GameObject as a `VRCPickup` receive these events (see [Interaction events](../udonsharp-behaviour/interaction-events.md)):

- `OnPickup`, `OnDrop` — fired when the **local** player picks up / drops this object. **[Docs]**
- `OnPickupUseDown`, `OnPickupUseUp` — Use button pressed / released while held. Require Auto Hold on desktop. **[Docs]**

All local-only.

### Auto Hold

Auto Hold decides whether the pickup stays in the hand after the initial Grab. It has nothing to do with ownership. **[Docs]**

| Component version | Auto Hold |
|---|---|
| 1.0 (legacy) | Dropdown: `Yes` / `No` / `Sometimes` / `AutoDetect`. `Sometimes` and `AutoDetect` are legacy and should generally be avoided. |
| 1.1 (SDK 3.9.0+) | Checkbox. Checked: first Grab attaches, a second Grab (or mapped Drop action) releases. Unchecked: drops when Grab is released. Works on **all input devices**. |

Existing pickups stay on 1.0 until you press **Upgrade** on the component and re-upload; un-upgraded worlds are unaffected. (Known issue: the Upgrade button may not appear when several pickups are multi-selected.) VRChat recommends the latest version. **[Docs]** <https://creators.vrchat.com/releases/release-3-9-0>

Ownership: picking up a networked pickup transfers ownership of its GameObject to the picker automatically. **[Community-known]** If other synced UdonBehaviours elsewhere need to follow, call `Networking.SetOwner(Networking.LocalPlayer, ...)` on them in `OnPickup`.

### Networking pickup state

Pickup position/rotation can be synced via `VRCObjectSync` (see [Networking → ObjectSync](../networking/object-sync.md)). Without it, the pickup is local-only — other players see it sitting where it was. **[Docs]**

```
GameObject "Sword"
├── VRCPickup
├── VRCObjectSync
├── Rigidbody
├── BoxCollider
└── (your UdonSharpBehaviour for behaviour logic)
```

## Stations

A `VRC.SDK3.Components.VRCStation` is the "sit here" component. The GameObject needs a collider (usually **Is Trigger**) so players can interact with it. The player is moved to the station's enter location and can stand up by moving / jumping unless exit is disabled. **[Docs]** <https://creators.vrchat.com/worlds/components/vrc_station>

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

Both fire on every client (for local and remote players). Use the parameter to act on the right player. **[Docs]**

To force a player out — e.g. when `disableStationExit` is set — use the station's `ExitStation` node (`station.ExitStation(player)` in U#). **[Docs]** for the node; U# form **[Community-known]**

### Common configuration

| Inspector field | Use |
|---|---|
| `PlayerMobility` | `Mobile` (can move while seated), `Immobilize` (locked at the enter location), `ImmobilizeForVehicle` (like Immobilize, tuned for moving stations). |
| `canUseStationFromStation` | Allow switching station-to-station while seated. |
| `disableStationExit` | Player cannot leave by moving / jumping. Provide another way out (trigger or `ExitStation`). |
| `seated` | Whether the player sits or stands. |
| `stationEnterPlayerLocation` | Transform the player is moved to. |
| `stationExitPlayerLocation` | Transform where the player ends up after exit. |
| `animatorController` | Optional controller replacing the default sitting animations. |
| `controlsObject` | For stations that control an object, such as a vehicle. |

Field meanings **[Docs]**; C# field casing **[Community-known]**.

## See also

- [VRCPlayerApi](vrc-player-api.md)
- [Interaction events](../udonsharp-behaviour/interaction-events.md)
- [Networking → ObjectSync](../networking/object-sync.md)
