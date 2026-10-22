# Players

`VRC.SDKBase.VRCPlayerApi` is the runtime handle for any player in the instance (local or remote). The companion static class `VRC.SDKBase.Networking` provides instance-wide helpers — local player, master status, ownership.

## Pages

| Page | Topic |
|---|---|
| [VRCPlayerApi](vrc-player-api.md) | Properties, position, locomotion, audio, voice, haptics, language. |
| [Tracking Data](tracking-data.md) | `GetTrackingData`, head/hands position & rotation, bones. |
| [Voice & Audio](voice-audio.md) | Per-player voice and avatar audio configuration. |
| [Pickups & Stations](pickups-stations.md) | `EnablePickups`, `GetPickupInHand`, station entry. |
| [Player Tags](player-tags.md) | `SetPlayerTag` / `GetPlayerTag` / `ClearPlayerTags` (and the broken `GetPlayersWithTag`). |
| [Drones](drones.md) | `VRCDroneApi`, `GetDrone`, drone trigger events (SDK 3.8.0+). |
| [Networking class](networking-class.md) | `Networking.LocalPlayer`, `IsMaster`, `Master` / `InstanceOwner`, ownership, server time. |

## Quick facts

- The local player reference: `Networking.LocalPlayer`. May be `null` in the editor's Play mode before the SDK initializes. Always null-check.
- Player references can become invalid mid-frame if the player leaves. Use `Utilities.IsValid(player)` or `VRCPlayerApi.IsValid(player)` before dereferencing a stored reference.
- `player.playerId` is unique within the instance; **not** globally unique across worlds.
- The master changes when the current master leaves. Don't bake master-status assumptions into long-running logic. Use `Networking.Master` (SDK 3.7.0+) to get the current master directly and `OnMasterTransferred` to react to changes; gate privileged features on the **instance owner** (`Networking.InstanceOwner`), not the master. **[Docs]**
- `VRCPlayerApi.GetPlayers()` with no argument (SDK 3.10.2+) returns a freshly allocated array of all players — convenient, but use the `GetPlayers(VRCPlayerApi[])` overload for per-frame work. **[Docs]**
- Spawning: the VRC Scene Descriptor has a **Spawn Radius** (SDK 3.8.2+) — players spawn at a random point within that radius of the spawn point. New worlds from the default template use `0.2` m; `0` keeps the old exact-point behaviour. **[Docs]** <https://creators.vrchat.com/releases/release-3-8-2>
- Layer 3 is now named **`Item`** (formerly `reserved3`, renamed in SDK 3.8.2). User-spawned VRChat items live there and Udon cannot reference them (they appear as `null`); exclude `Item` from raycast `LayerMask`s and `Utilities.IsValid` anything returned from Physics queries. **[Docs]** <https://creators.vrchat.com/worlds/items>

## See also

- [UdonSharpBehaviour → player events](../udonsharp-behaviour/player-events.md)
- [Networking](../networking/README.md)
- Official: <https://creators.vrchat.com/worlds/udon/players/> and subpages (Getting Players, Player Positions, Player Forces, Player Collisions, Player Audio, Player Avatar Scaling, Drone API).
