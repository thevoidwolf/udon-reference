# Drones

(SDK 3.8.0+) Udon can read and move the **VRChat+ Drone** of players in your world through `VRCDroneApi`, obtained from the player that owns the drone. **[Docs]** <https://creators.vrchat.com/worlds/udon/players/drones/>

Official pages: [Drone API](https://creators.vrchat.com/worlds/udon/players/drones/) · [Getting Drones](https://creators.vrchat.com/worlds/udon/players/drones/getting-drones) · [Drone Information](https://creators.vrchat.com/worlds/udon/players/drones/drone-information).

The docs list inputs/outputs in Udon Graph form, without C# parameter names; the signatures below use illustrative names.

## Getting a drone

```csharp
VRCDroneApi drone = player.GetDrone();     // the drone belonging to this player
VRCPlayerApi owner = drone.GetPlayer();    // the player who owns this drone
```

**[Docs]**

The docs don't say what `GetDrone` returns for a player who has no drone, so null-check with `Utilities.IsValid(drone)` and test `IsDeployed()` before acting. **[Unverified]**

## Drone information

| Member | Returns | Notes |
|---|---|---|
| `GetPlayer()` | `VRCPlayerApi` | Owning player. |
| `IsDeployed()` | `bool` | Whether the drone is currently deployed in the world. |
| `GetPosition()` | `Vector3` | World-space position. |
| `TryGetPosition(out Vector3 position)` | `bool` | Success flag + world-space position. |
| `GetRotation()` | `Quaternion` | World-space rotation. |
| `TryGetRotation(out Quaternion rotation)` | `bool` | Success flag + world-space rotation. |
| `GetVelocity()` | `Vector3` | World-space velocity. |
| `TryGetVelocity(out Vector3 velocity)` | `bool` | Success flag + world-space velocity. |
| `TeleportTo(Vector3 position, Quaternion rotation)` | `void` | World-space target. |
| `TeleportTo(Vector3 position, Quaternion rotation, bool lerpOnRemote)` | `void` | `false` / omitted: remote users see the drone snap. `true`: remote users see it lerp smoothly. |
| `SetVelocity(Vector3 velocity)` | `void` | World-space velocity. |

**[Docs]** for names, types and behaviour. The `lerpOnRemote` parameter name mirrors `VRCPlayerApi.TeleportTo` and is **[Unverified]**.

The docs don't state who may call the setters. By analogy with `VRCPlayerApi.TeleportTo` (local player only), expect them to act on the **local** player's drone and route requests for other players' drones through a network event. **[Unverified]**

## Events

Fired when a drone interacts with a trigger collider in your world:

```csharp
public override void OnDroneTriggerEnter(VRCDroneApi drone) { }   // drone entered the trigger
public override void OnDroneTriggerStay(VRCDroneApi drone)  { }   // drone stays inside the trigger
public override void OnDroneTriggerExit(VRCDroneApi drone)  { }   // drone exited, or despawned inside the trigger
```

**[Docs]** (each event outputs a `VRCDroneApi`). Note `OnDroneTriggerExit` also fires when the drone despawns inside the trigger — handy for cleaning up "drone is in zone" state.

## Networking note

SDK 3.8.0 also added **Force Kinematic On Remote** to `VRCObjectSync`; the VRChat Drone uses it so only the owner simulates physics and collisions. **[Docs]** <https://creators.vrchat.com/releases/release-3-8-0>

## Patterns

### Keep drones out of an area

```csharp
public Transform ejectPoint;

public override void OnDroneTriggerEnter(VRCDroneApi drone) {
    if (!Utilities.IsValid(drone)) return;
    VRCPlayerApi owner = drone.GetPlayer();
    if (!Utilities.IsValid(owner) || !owner.isLocal) return;   // only move our own drone [Unverified: setter scope]
    drone.TeleportTo(ejectPoint.position, ejectPoint.rotation, true);
}
```

## See also

- [VRCPlayerApi](vrc-player-api.md)
- [UdonSharpBehaviour → player events](../udonsharp-behaviour/player-events.md)
- [Networking → ObjectSync](../networking/object-sync.md)
