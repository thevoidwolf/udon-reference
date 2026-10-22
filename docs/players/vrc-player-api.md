# VRCPlayerApi

`VRC.SDKBase.VRCPlayerApi` represents one player in the instance. Returned by `Networking.LocalPlayer`, passed to player events, and obtainable via `VRCPlayerApi.GetPlayers(...)` / `VRCPlayerApi.GetPlayerById(int)`.

Source: official docs at <https://creators.vrchat.com/worlds/udon/players/> plus community-known surface. **[Docs]** / **[Community-known]** where noted.

---

## Identity

| Member | Type | Notes |
|---|---|---|
| `playerId` | `int` | Unique within the instance. **[Docs]** |
| `displayName` | `string` | The displayed name; differs from login username for some accounts. **[Docs]** |
| `isLocal` | `bool` | `true` for the local client. **[Docs]** |
| `isMaster` | `bool` | `true` if this player is currently instance master. **[Docs]** |
| `isInstanceOwner` | `bool` | `true` if this player is the current instance owner. Only the creator of Invite / Invite+ / Friends / Friends+ instances counts; nobody is instance owner in Group, Public, or Build & Test instances. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-components> |
| `isVRCPlus` | `bool` | Has an active VRC+ subscription. (SDK 3.10.3+) **[Docs]** |
| `isSuspended` | `bool` | Device suspended (sleep/background). Always `false` for the local player. Pair with the `OnPlayerSuspendChanged(VRCPlayerApi)` event. (SDK 3.7.0+) **[Docs]** |
| `IsUserInVR()` | `bool` | VR headset active. **[Docs]** |
| `GetDrone()` | `VRCDroneApi` | The player's VRChat+ Drone handle. (SDK 3.8.0+) See [Drones](drones.md). **[Docs]** |

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
| `SetVelocity(Vector3)` | `void` | Only meaningful for local player. Sets the local player's grounded state to `false`. **[Docs]** |
| `GetVelocity()` | `Vector3` | World-space velocity. **[Docs]** |
| `IsPlayerGrounded()` | `bool` | Whether the player is grounded (grounding is what enables Jump). **[Docs]** |
| `Respawn()` | `void` | Send the local player to a spawn point. With a scene-descriptor **Spawn Radius** (SDK 3.8.2+) the landing point is randomised within that radius. **[Docs]** |
| `TeleportTo(Vector3 pos, Quaternion rot)` | `void` | Local player only. **[Docs]** |
| `TeleportTo(Vector3 pos, Quaternion rot, VRC_SceneDescriptor.SpawnOrientation orientation)` | `void` | |
| `TeleportTo(Vector3 pos, Quaternion rot, VRC_SceneDescriptor.SpawnOrientation orientation, bool lerpOnRemote)` | `void` | `lerpOnRemote` (default `false`) controls whether remote clients see a smooth move. |

`TeleportTo` notes **[Docs]** <https://creators.vrchat.com/worlds/udon/players/player-positions>:
- Only the **local** player can be teleported — use a network event to make remote players teleport themselves. Stations can prevent teleporting.
- `lerpOnRemote = false`: instant for remotes (more bandwidth). `true`: treated as normal movement, smooth for remotes, no animation/IK reset.
- Teleports over 1 m or 45° reset animation and IK unless `lerpOnRemote` is `true`.
- Don't teleport during network callbacks such as `OnDeserialization` — delay one frame with `SendCustomEventDelayedFrames`.
- Not the same as `VRCObjectSync.TeleportTo(Transform targetLocation)`, which moves a synced *object* to a Transform. **[Docs]** See [Networking → ObjectSync](../networking/object-sync.md).

## Locomotion

All local-player only — they can't be used on remote players. **[Docs]** <https://creators.vrchat.com/worlds/udon/players/player-forces>

| Setter / getter | Default | Working range | Notes |
|---|---|---|---|
| `SetWalkSpeed` / `GetWalkSpeed` | 2 | ~0–5 | Keep below run speed. |
| `SetRunSpeed` / `GetRunSpeed` | 4 | ~0–10 | Keep above walk speed. |
| `SetStrafeSpeed` / `GetStrafeSpeed` | 2 | ~0–5 | Not affected by running; match walk speed. |
| `SetJumpImpulse` / `GetJumpImpulse` | 0 | ~0–10 | Jump is **disabled by default**; the stock VRCWorld prefab sets 3. |
| `SetGravityStrength` / `GetGravityStrength` | 1 | ~0–10 | Multiplier on Earth gravity. Use this instead of changing `Physics.gravity`. |

`Immobilize(bool)` keeps the player in place and disables locomotion; VR players can still look around but their avatar stays put. No getter. **[Docs]**

`UseLegacyLocomotion()` reverts to SDK2 movement until the player leaves the world — **not recommended, do not use in new worlds**. **[Docs]**

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
string lang = VRCPlayerApi.GetCurrentLanguage();      // local user's selected language, RFC 5646: "en", "ja", "zh-CN", ...
string[] langs = VRCPlayerApi.GetAvailableLanguages(); // every language selectable in VRChat settings
```

Both report the **local** user's settings. The docs list them without a `VRCPlayerApi` input, hence the static call shown — static-ness is inferred, **[Unverified]**. React to changes with the `OnLanguageChanged` event (SDK 3.7.0+; also fires on join). **[Docs]**

## Avatar height

`GetAvatarEyeHeightAsMeters()` — configured eye height of the target player's avatar; works on local **and** remote players. Useful in combination with `OnAvatarEyeHeightChanged`. **[Docs]**

The setters are local-player only and have no effect if avatar scaling is disabled on the website: `SetAvatarEyeHeightByMeters(float)`, `SetAvatarEyeHeightByMultiplier(float)`, `SetAvatarEyeHeightMinimumByMeters(float)` (≥ 0.2 m), `SetAvatarEyeHeightMaximumByMeters(float)` (≤ 5 m), `SetManualAvatarScalingAllowed(bool)`, plus the matching `Get*` accessors. Udon-set heights may range 0.01–10000 m, but values outside 0.1–100 m are unsupported. **[Docs]** <https://creators.vrchat.com/worlds/udon/players/player-avatar-scaling>

## Iterating all players

Simple, allocating form (SDK 3.10.2+) **[Docs]**:

```csharp
VRCPlayerApi[] players = VRCPlayerApi.GetPlayers();   // new array every call
```

Non-allocating form — reuse a buffer, grow it when needed (the official example starts at 10 and doubles) **[Docs]** <https://creators.vrchat.com/worlds/udon/players/getting-players>:

```csharp
int n = VRCPlayerApi.GetPlayerCount();
VRCPlayerApi[] buffer = new VRCPlayerApi[n];
VRCPlayerApi.GetPlayers(buffer);
for (int i = 0; i < n; i++) {
    if (!Utilities.IsValid(buffer[i])) continue;
    // use buffer[i]
}
```

Both `GetPlayerCount` and `GetPlayers` are static. Allocate the array up front and reuse; if the array is smaller than the player count, the excess players are silently skipped. **[Docs]**

## Patterns

### "Do something to whichever player is master right now"

```csharp
VRCPlayerApi master = Networking.Master;   // SDK 3.7.0+; always valid [Docs]
```

On older SDKs, iterate `GetPlayers` and test `isMaster`.

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
- **`Store.DoesPlayerOwnProduct`** in `Start` can race with the purchase data not yet loaded — wait for `OnPurchasesLoaded` instead. **[Docs]** <https://creators.vrchat.com/economy/sdk/udon-documentation>

## See also

- [Player events on UdonSharpBehaviour](../udonsharp-behaviour/player-events.md)
- [Networking → ownership](../networking/ownership.md)
- [Networking class](networking-class.md)
- [Networking → ObjectSync](../networking/object-sync.md)
- [Drones](drones.md)
- [References → Community Resources](../references/community-resources.md)
