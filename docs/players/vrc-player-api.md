# VRCPlayerApi

`VRC.SDKBase.VRCPlayerApi` represents one player in the instance. Returned by `Networking.LocalPlayer`, passed to player events, and obtainable by iterating with `VRCPlayerApi.GetPlayers(...)`.

Source: official docs at <https://creators.vrchat.com/worlds/udon/players/> plus community-known surface. **[Docs]** / **[Community-known]** where noted.

---

## Identity

| Member | Type | Notes |
|---|---|---|
| `playerId` | `int` | Unique within the instance. **[Docs]** |
| `displayName` | `string` | The displayed name; differs from login username for some accounts. **[Docs]** |
| `isLocal` | `bool` | `true` for the local client. **[Docs]** |
| `isMaster` | `bool` | `true` if this player is currently instance master. **[Docs]** |
| `isInstanceOwner` | `bool` | `true` if this player owns the instance (the creator). **[Docs]** |
| `isVRCPlus` | `bool` | Has active VRC+ subscription. **[Docs]** |
| `isSuspended` | `bool` | Device suspended (sleep/background). Always `false` for the local player. **[Docs]** |
| `IsUserInVR()` | `bool` | VR headset active. **[Docs]** |

## Validity

```csharp
static bool IsValid(VRCPlayerApi player);   // safe even if player has left
// also exposed as Utilities.IsValid(player)
```

Stash a `VRCPlayerApi` reference and the player can disappear before you use it. Always:

```csharp
if (!Utilities.IsValid(_storedPlayer)) return;
```

## Position, rotation, velocity

| Member | Signature | Notes |
|---|---|---|
| `GetPosition()` | `Vector3` | World position of player root. |
| `GetRotation()` | `Quaternion` | |
| `SetVelocity(Vector3)` | `void` | Only meaningful for local player. |
| `GetVelocity()` | `Vector3` | |
| `Respawn()` | `void` | Send local player to closest spawn point. |
| `TeleportTo(Vector3 pos, Quaternion rot)` | `void` | |
| `TeleportTo(Vector3 pos, Quaternion rot, VRC_SceneDescriptor.SpawnOrientation orientation)` | `void` | |
| `TeleportTo(Vector3 pos, Quaternion rot, VRC_SceneDescriptor.SpawnOrientation orientation, bool lerpOnRemote)` | `void` | `lerpOnRemote` controls whether remote clients see a smooth move. |

## Locomotion

`SetWalkSpeed`, `SetRunSpeed`, `SetStrafeSpeed`, `SetJumpImpulse`, and corresponding `Get*` accessors. `SetGravityStrength` / `GetGravityStrength`.

`Immobilize(bool)` freezes the player in place. **[Community-known]**

`UseLegacyLocomotion()` reverts to SDK2 movement — **deprecated, do not use in new worlds**. **[Docs]**

```csharp
var p = Networking.LocalPlayer;
p.SetWalkSpeed(4f);
p.SetRunSpeed(8f);
p.SetJumpImpulse(5f);
p.SetGravityStrength(1f);
```

## Tracking & bones

See [Tracking Data](tracking-data.md).

## Voice & avatar audio

See [Voice & Audio](voice-audio.md).

## Pickups & stations

See [Pickups & Stations](pickups-stations.md).

## Player tags

See [Player Tags](player-tags.md).

## Haptics

```csharp
PlayHapticEventInHand(VRC_Pickup.PickupHand hand, float duration, float amplitude, float frequency);
```

All float params in 0..1. `hand` is `Left` or `Right`. **[Docs]**

## Language

```csharp
string lang = player.GetCurrentLanguage();         // RFC 5646: "en", "ja", "zh-CN", ...
string[] langs = player.GetAvailableLanguages();
```

**[Docs]**

## Avatar height

`GetAvatarEyeHeightAsMeters()` — current eye height. Useful in combination with `OnAvatarEyeHeightChanged`. **[Community-known]**

## Iterating all players

```csharp
int n = VRCPlayerApi.GetPlayerCount();
VRCPlayerApi[] buffer = new VRCPlayerApi[n];
VRCPlayerApi.GetPlayers(buffer);
for (int i = 0; i < n; i++) {
    if (!Utilities.IsValid(buffer[i])) continue;
    // use buffer[i]
}
```

Both `GetPlayerCount` and `GetPlayers` are static. Allocate the array up front and reuse.

## Patterns

### "Do something to whichever player is master right now"

```csharp
VRCPlayerApi master = null;
int n = VRCPlayerApi.GetPlayerCount();
var buf = new VRCPlayerApi[n];
VRCPlayerApi.GetPlayers(buf);
for (int i = 0; i < n; i++) {
    if (Utilities.IsValid(buf[i]) && buf[i].isMaster) { master = buf[i]; break; }
}
```

### "Position-based check"

```csharp
public Transform safeZone;
public override void OnPlayerTriggerEnter(VRCPlayerApi player) {
    if (!player.isLocal) return;
    Vector3 d = player.GetPosition() - safeZone.position;
    if (d.magnitude > 5f) player.Respawn();
}
```

## Known gotchas (community-tracked)

- **`Utilities.IsValid(player)` is mandatory** before any property read on stashed references. A player that left will fail validity. **[Docs]** <https://creators.vrchat.com/worlds/udon/players/>
- **`playerId == -1` inside `OnPlayerLeft`** in some cases — even when `IsValid` returns true. Reading `playerId` of a player you've never seen in `OnPlayerJoined` can crash the game. Track joined players in a `DataDictionary`. **[Community]** <https://feedback.vrchat.com/udon/p/vrcplayerapiplayerid-may-returns-1-in-onplayerleft>
- **`GetPosition` / `GetRotation` can null-ref** shortly after a player leaves, even with prior valid reference. **[Community]** <https://vrchat.canny.io/udon/p/vrcplayerapi-getposition-and-getrotation-can-throw-due-to-a-null-ref-if-checked/voters>
- **`SetVelocity` when grounded historically did nothing.** Lift the player slightly first; reportedly fixed in recent client versions. **[Community]** <https://x.com/naqtn/status/1236825253591937025>
- **`SetVelocity(GetVelocity() + impulse)` diverges in ClientSim.** Real client adds correctly; ClientSim drifts the player into the sky. Issue closed "not planned." **[Community]** <https://github.com/vrchat-community/ClientSim/issues/68>
- **Remote-player `GetTrackingData` falls back to bone data.** Tracking is not networked; hand rotation reliable only on local player. **[Community]** <https://feedback.vrchat.com/udon/p/remote-playerapi-gettrackingdata-and-getbonepositionrotationtransform-return-wro>
- **`Networking.LocalPlayer` is null in editor before SDK init.** Always null-check. The community pattern: `isInEditor = (Networking.LocalPlayer == null)` and branch.
- **`OnPlayerJoined` fires for every player already in the instance when you join, including yourself.** Instantiating prefabs inside `OnPlayerJoined` can null-exception; defer via `SendCustomEventDelayedSeconds`. **[Community]** <https://feedback.vrchat.com/open-beta/p/instantiate-on-join-failed>
- **`Store.DoesPlayerOwnProduct`** in `Start` can race with the purchase data not yet loaded — wait for `OnPurchasesLoaded` instead. **[Docs]** <https://creators.vrchat.com/worlds/udon/players/>

## See also

- [Player events on UdonSharpBehaviour](../udonsharp-behaviour/player-events.md)
- [Networking → ownership](../networking/ownership.md)
- [Networking class](networking-class.md)
- [References → Community Resources](../references/community-resources.md)
