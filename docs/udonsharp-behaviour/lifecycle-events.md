# Lifecycle Events

The methods listed here are `public virtual` on `UdonSharpBehaviour` (source: `Runtime/UdonSharpBehaviour.cs`). Override the ones you need. The VRChat runtime dispatches each at the point described.

> **Source caveat:** **[Source]** tags refer to the public UdonSharp repo (standalone 1.1.9). Events added in later SDKs are only present in the copy bundled with the Worlds SDK and are tagged **[Docs]** from the official event list at <https://creators.vrchat.com/worlds/udon/graph/event-nodes/>. U# override signatures for those are written from the documented outputs; parameter *names* may differ from the SDK's, which doesn't matter for overriding.

Methods inherited from `MonoBehaviour` (`Awake`, `Start`, `OnEnable`, `OnDisable`, `Update`, `LateUpdate`, `FixedUpdate`, `OnDestroy`, `OnTriggerEnter/Exit/Stay`, `OnCollisionEnter/Exit/Stay`, etc.) work as in regular Unity. **[Source]**

This page focuses on the Udon-specific virtuals. For player-trigger / collision callbacks, see [Player Events](player-events.md).

---

## Frame events

### `PostLateUpdate`
```csharp
public virtual void PostLateUpdate()
```
Fires after every `LateUpdate` callback in the frame, before rendering. Use this for camera follow or IK that must observe the *final* state of all transforms. **[Source]**

---

## Network sync events

These integrate with `[UdonSynced]` and `RequestSerialization`. See [Networking → serialization](../networking/serialization.md) for the byte-level story.

### `OnPreSerialization`
```csharp
public virtual void OnPreSerialization()
```
Called on the **owner** just before this behaviour's synced state is packed into a network packet. Update derived synced fields here. **[Source]**

### `OnDeserialization`
```csharp
public virtual void OnDeserialization()
public virtual void OnDeserialization(VRC.Udon.Common.DeserializationResult result)
```
Called on **non-owners** immediately after they receive a packet and apply it to local synced fields. Two overloads exist; override whichever signature you need — both are stubbed. **[Source]**

`DeserializationResult` has three properties: `sendTime` and `receiveTime` (seconds, on the *local* `Time.realtimeSinceStartup` clock — `sendTime` can be negative), and `isFromStorage` (`true` when the data was restored from persistence rather than received from a live client). **[Docs]**

### `OnPostSerialization`
```csharp
public virtual void OnPostSerialization(VRC.Udon.Common.SerializationResult result)
```
Fires on the owner after a `RequestSerialization()` attempt completes. Inspect `result.success` (bool) and `result.byteCount` (int) to detect packets that were dropped because they exceeded the size budget. **[Source]** **[Docs]**

---

## Ownership

### `OnOwnershipRequest`
```csharp
public virtual bool OnOwnershipRequest(VRCPlayerApi requestingPlayer, VRCPlayerApi requestedOwner) => true
```
Called **before** ownership is transferred; return `false` to reject. The default implementation returns `true`. **[Source]** The logic runs locally on **both the requester and the current owner** — if the two disagree (e.g. your check depends on unsynced local state) you get a desync, which typically shows up as the owner unexpectedly rejecting the transfer. **[Docs]**

`requestedOwner` is the *target* of the transfer (usually equal to `requestingPlayer`, but `Networking.SetOwner` allows transferring ownership to a third party). The official docs name this parameter `newOwner`.

### `OnOwnershipTransferred`
```csharp
public virtual void OnOwnershipTransferred(VRCPlayerApi player)
```
Called on every client after ownership of this GameObject changes to `player`. **[Source]** **[Docs]**

The deprecated `OnOwnershipTransferred()` (no parameter) form is `[Obsolete(... error: true)]` — it won't compile. **[Source]**

---

## Interaction

`Interact`, `OnPickup`, `OnDrop`, `OnPickupUseDown`, `OnPickupUseUp` are covered in [Interaction Events](interaction-events.md).

`OnStationEntered(VRCPlayerApi)` / `OnStationExited(VRCPlayerApi)` fire when a player sits in / exits a `VRCStation` component attached to the GameObject this behaviour is on. The legacy parameterless forms are `[Obsolete(error: true)]`. **[Source]**

---

## Spawn / respawn

### `OnSpawn`
```csharp
public virtual void OnSpawn()
```
Stubbed on the base class **[Source]**, but **deprecated**: it is an object-pool "spawned" event, *not* a player-spawn event. The official docs say to use `OnEnable` instead to react to an object being spawned from a `VRCObjectPool`. **[Docs]**

### `OnPlayerRespawn`
```csharp
public virtual void OnPlayerRespawn(VRCPlayerApi player)
```
Per the official docs: fired when the **local** player respawns by clicking "Respawn" in their menu. **[Docs]** Whether it also fires for `VRCPlayerApi.Respawn()` calls or for remote players is not documented. **[Unverified]**

---

## Avatar

### `OnAvatarChanged`
```csharp
public virtual void OnAvatarChanged(VRCPlayerApi player)
```
Fires when `player`'s avatar finishes loading (initial load and any subsequent change). **[Source]** **[Docs]** For the local player, eye height read during this event is still the *prefab* height; a persisted scale arrives afterwards via `OnAvatarEyeHeightChanged`. **[Docs]**

### `OnAvatarEyeHeightChanged`
```csharp
public virtual void OnAvatarEyeHeightChanged(VRCPlayerApi player, float prevEyeHeightAsMeters)
```
Fires when an avatar change or runtime scaling shifts `player`'s eye height. `prevEyeHeightAsMeters` is the value *before* the change. Use `player.GetAvatarEyeHeightAsMeters()` for the current value. **[Source]**

On first load the previous value may be `0`. For **remote** players it can fire more than once per avatar change and may arrive *before* `OnAvatarChanged`. **[Docs]** (<https://creators.vrchat.com/worlds/udon/avatar-events/>)

---

## Player join / leave

Both are usually treated as networking events but the runtime dispatches them on every `UdonSharpBehaviour` in the scene that overrides them.

```csharp
public virtual void OnPlayerJoined(VRCPlayerApi player)
public virtual void OnPlayerLeft(VRCPlayerApi player)
```

Notes:

- When **you** join, `OnPlayerJoined` fires once for **every player already in the instance, including yourself**; after that it fires only for each newly joining player. **[Docs]** It is *not* a reliable "synced state is ready" signal — check `Networking.IsNetworkSettled`, and for persistent data wait for `OnPlayerRestored` (below).
- `OnPlayerLeft` is the last event you'll see for a leaving player. If the master leaves, the new master is chosen **before** `OnPlayerLeft` is called. **[Docs]**
- **The local player's persistent data cannot be saved in their own `OnPlayerLeft`** — save earlier. **[Docs]**

See [Players → Networking class](../players/networking-class.md) for `Networking.IsNetworkSettled`.

---

## Instance master

### `OnMasterTransferred` (SDK 3.7.0+)
```csharp
public override void OnMasterTransferred(VRCPlayerApi newMaster)
```
Fires for everyone when the instance master changes because the previous master left. `newMaster` is always valid. For the first player in a new instance it fires after `OnPlayerJoined` (master transferred from "nobody"). **[Docs]** See [Networking events](networking-events.md#instance-master).

---

## Player state, platform and client settings

All signatures below are from the official event list; the version is the SDK that added them. **[Docs]**

```csharp
public override void OnPlayerSuspendChanged(VRCPlayerApi player)              // SDK 3.7.0+
public override void OnLanguageChanged(string language)                       // SDK 3.7.0+
public override void OnInputMethodChanged(VRCInputMethod inputMethod)         // see Input events
public override void OnScreenUpdate(VRC.SDK3.Platform.ScreenUpdateData data)  // SDK 3.5.0+, mobile
public override void OnVRCPlusMassGift(VRCPlayerApi gifter, int numGifts)     // SDK 3.7.5+
public override void OnVRCCameraSettingsChanged(VRCCameraSettings camera)     // SDK 3.8.1+
public override void OnVRCQualitySettingsChanged()                            // SDK 3.8.1+
```

| Event | Fires |
|---|---|
| `OnPlayerSuspendChanged` | Any player's device enters sleep / switches app (fires on wake-up for the suspended player). Check `player.isSuspended` to tell suspend from resume. Suspended devices run no Udon and process no network events — consider moving ownership of important objects to a non-suspended player. PC players currently never suspend, but don't assume it. |
| `OnLanguageChanged` | On join and whenever the local player changes their display language. |
| `OnInputMethodChanged` | Local player switches input method. See [Input events](input-events.md#oninputmethodchanged). |
| `OnScreenUpdate` | Local player first enters on a mobile device, and on each orientation change. `data.type` (`ScreenUpdateType`, currently only `OrientationChanged`), `data.orientation` (`VRCOrientation`: `Landscape` / `Portrait`), `data.resolution` (`Vector2`). |
| `OnVRCPlusMassGift` | Any player in the instance drops a VRC+ gift bomb. |
| `OnVRCCameraSettingsChanged` | User changes certain graphics options (e.g. FOV, near clip) for `VRCCameraSettings.ScreenCamera` or `PhotoCamera`. **Not** raised by your own writes, nor by camera position/rotation. Can fire every frame (photo-camera zoom, dolly zoom, window resize) and multiple times per frame — keep it cheap. |
| `OnVRCQualitySettingsChanged` | User changes graphics settings affecting `VRCQualitySettings`. May fire often — keep it cheap. |

See [Graphics](../graphics/README.md) for `VRCCameraSettings` / `VRCQualitySettings`.

---

## Persistence events

Added with Persistence in SDK 3.7.4; storage-usage events in SDK 3.10.0. **[Docs]**

```csharp
public override void OnPlayerRestored(VRCPlayerApi player)                                  // SDK 3.7.4+
public override void OnPlayerDataUpdated(VRCPlayerApi player, PlayerData.Info[] infos)      // SDK 3.7.4+
public override void OnPersistenceUsageUpdated(VRCPlayerApi player)                         // SDK 3.10.0+
public override void OnPlayerDataStorageWarning(VRCPlayerApi player)                        // SDK 3.10.0+
public override void OnPlayerDataStorageExceeded(VRCPlayerApi player)                       // SDK 3.10.0+
public override void OnPlayerObjectStorageWarning(VRCPlayerApi player)
public override void OnPlayerObjectStorageExceeded(VRCPlayerApi player)
```

- `OnPlayerRestored` — all of a player's PlayerData **and** PlayerObjects have loaded (fires even if they have no saved data). On join you get it for every player including yourself, then once per newcomer. **Don't read or write persistent data before it.**
- `OnPlayerDataUpdated` — end of frame after any player's PlayerData changed or was received; `infos` lists every key with its `State` (`Unchanged`, `Added`, `Removed`, `Changed`, `Restored`).
- `OnPersistenceUsageUpdated` — results of `Networking.RequestStorageUsageUpdate()` are ready.
- `…StorageWarning` / `…StorageExceeded` — the player is nearing / over the PlayerData or PlayerObject storage limit.

The SDK version that added the two `OnPlayerObjectStorage…` events is not stated in the release notes. **[Unverified]** Full model: [Persistence](../persistence/README.md).

---

## Loading and GPU readback

```csharp
public virtual void OnStringLoadSuccess(VRC.SDK3.StringLoading.IVRCStringDownload result)
public virtual void OnStringLoadError(VRC.SDK3.StringLoading.IVRCStringDownload result)
public virtual void OnImageLoadSuccess(VRC.SDK3.Image.IVRCImageDownload result)
public virtual void OnImageLoadError(VRC.SDK3.Image.IVRCImageDownload result)
public virtual void OnAsyncGpuReadbackComplete(VRC.SDK3.Rendering.VRCAsyncGPUReadbackRequest request)
```

Callbacks for `VRCStringDownloader`, `VRCImageDownloader` and `VRCAsyncGPUReadback.Request`. **[Source]** See [Loading](../loading/README.md) and [Graphics → image downloader](../graphics/image-downloader.md).

---

## Dynamics (PhysBones, Contacts)

`OnPhysBoneGrabbed` / `Released` / `Posed` / `UnPosed` and `OnContactEnter` / `OnContactExit` (SDK 3.10.0+) are covered in [Dynamics events](dynamics-events.md).

---

## Event execution order

The full ordering of Unity's MonoBehaviour events + Udon's additions is documented at <https://creators.vrchat.com/worlds/udon/event-execution-order/>. Per the official docs: *"Unity and VRChat updates may change the event execution order ... not all events are listed, and some events may be executed in a different order depending on circumstances (being an object's owner, joining a world late, etc.)"* **[Docs]**

If event ordering matters for a feature, do not assume — guard with explicit state (`Networking.IsNetworkSettled`, a `bool _initialized`, etc.).

## See also

- [Networking events](networking-events.md) — only the sync subset, in one place.
- [Player events](player-events.md) — trigger / collision / spawn callbacks.
- [API methods](api-methods.md) — `SendCustomEventDelayedFrames`, `SendCustomEventDelayedSeconds` for scheduling.
- [Dynamics events](dynamics-events.md) — PhysBone and Contact callbacks.
