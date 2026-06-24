# Players

`VRC.SDKBase.VRCPlayerApi` is the runtime handle for any player in the instance (local or remote). The companion static class `VRC.SDKBase.Networking` provides instance-wide helpers — local player, master status, ownership.

## Pages

| Page | Topic |
|---|---|
| [VRCPlayerApi](vrc-player-api.md) | Properties, position, locomotion, audio, voice, haptics, language. |
| [Tracking Data](tracking-data.md) | `GetTrackingData`, head/hands position & rotation, bones. |
| [Voice & Audio](voice-audio.md) | Per-player voice and avatar audio configuration. |
| [Pickups & Stations](pickups-stations.md) | `EnablePickups`, `GetPickupInHand`, station entry. |
| [Player Tags](player-tags.md) | `SetPlayerTag` / `GetPlayerTag` / `GetPlayersWithTag`. |
| [Networking class](networking-class.md) | `Networking.LocalPlayer`, `IsMaster`, ownership, server time. |

## Quick facts

- The local player reference: `Networking.LocalPlayer`. May be `null` in the editor's Play mode before the SDK initializes. Always null-check.
- Player references can become invalid mid-frame if the player leaves. Use `Utilities.IsValid(player)` or `VRCPlayerApi.IsValid(player)` before dereferencing a stored reference.
- `player.playerId` is unique within the instance; **not** globally unique across worlds.
- The master changes when the current master leaves. Don't bake master-status assumptions into long-running logic.

## See also

- [UdonSharpBehaviour → player events](../udonsharp-behaviour/player-events.md)
- [Networking](../networking/README.md)
