# Lifecycle Events

The methods listed here are `public virtual` on `UdonSharpBehaviour` (source: `Runtime/UdonSharpBehaviour.cs`). Override the ones you need. The VRChat runtime dispatches each at the point described.

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
Called on **non-owners** immediately after they receive a packet and apply it to local synced fields. Two overloads exist; the second carries timing metadata (`result.sendTime`, `result.receiveTime`). Override whichever signature you need — both are stubbed. **[Source]**

### `OnPostSerialization`
```csharp
public virtual void OnPostSerialization(VRC.Udon.Common.SerializationResult result)
```
Fires on the owner after a `RequestSerialization()` attempt completes. Inspect `result.success` (bool) and `result.byteCount` (int) to detect packets that were dropped because they exceeded the size budget. **[Source]**

---

## Ownership

### `OnOwnershipRequest`
```csharp
public virtual bool OnOwnershipRequest(VRCPlayerApi requestingPlayer, VRCPlayerApi requestedOwner) => true
```
Called before a remote-initiated ownership transfer is committed. Return `false` to veto the transfer. The default implementation returns `true`. **[Source]**

`requestedOwner` is the *target* of the transfer (usually equal to `requestingPlayer`, but `Networking.SetOwner` allows transferring ownership to a third party).

### `OnOwnershipTransferred`
```csharp
public virtual void OnOwnershipTransferred(VRCPlayerApi player)
```
Called on every client after ownership of this GameObject changes to `player`. **[Source]**

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
Fires on the local player when they spawn at one of the world's `VRC_SceneDescriptor` spawn points. **[Source]**

### `OnPlayerRespawn`
```csharp
public virtual void OnPlayerRespawn(VRCPlayerApi player)
```
Fires on every client when any player respawns. **[Docs]**

---

## Avatar

### `OnAvatarChanged`
```csharp
public virtual void OnAvatarChanged(VRCPlayerApi player)
```
Fires when `player`'s avatar finishes loading (initial load and any subsequent change). **[Source]** **[Docs]**

### `OnAvatarEyeHeightChanged`
```csharp
public virtual void OnAvatarEyeHeightChanged(VRCPlayerApi player, float prevEyeHeightAsMeters)
```
Fires when an avatar change or runtime scaling shifts `player`'s eye height. `prevEyeHeightAsMeters` is the value *before* the change. Use `player.GetAvatarEyeHeightAsMeters()` for the current value. **[Source]**

---

## Player join / leave

Both are usually treated as networking events but the runtime dispatches them on every `UdonSharpBehaviour` in the scene that overrides them.

```csharp
public virtual void OnPlayerJoined(VRCPlayerApi player)
public virtual void OnPlayerLeft(VRCPlayerApi player)
```

Notes:

- `OnPlayerJoined` fires for the **local** player too, once at world load. Use this as your "world has settled" signal.
- `OnPlayerLeft` is the last event you'll see for a leaving player. **You cannot save persistence data from inside `OnPlayerLeft`** — that write is rejected. Save earlier. **[Docs]**

See [Players → Networking class](../players/networking-class.md) for `Networking.IsNetworkSettled`.

---

## Event execution order

The full ordering of Unity's MonoBehaviour events + Udon's additions is documented at <https://creators.vrchat.com/worlds/udon/event-execution-order/>. Per the official docs: *"Unity and VRChat updates may change the event execution order ... not all events are listed, and some events may be executed in a different order depending on circumstances (being an object's owner, joining a world late, etc.)"* **[Docs]**

If event ordering matters for a feature, do not assume — guard with explicit state (`Networking.IsNetworkSettled`, a `bool _initialized`, etc.).

## See also

- [Networking events](networking-events.md) — only the sync subset, in one place.
- [Player events](player-events.md) — trigger / collision / spawn callbacks.
- [API methods](api-methods.md) — `SendCustomEventDelayedFrames`, `SendCustomEventDelayedSeconds` for scheduling.
