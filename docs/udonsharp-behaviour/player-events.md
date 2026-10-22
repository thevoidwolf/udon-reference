# Player Events

Events that fire when a player physically interacts with — or just exists relative to — the GameObject this `UdonSharpBehaviour` is attached to. All take a `VRCPlayerApi` argument identifying the relevant player. Override only what you need.

Source: `Runtime/UdonSharpBehaviour.cs`. **[Source]**

> **Validity reminder:** `VRCPlayerApi` references can become invalid if the player leaves between the moment the event fired and the moment you actually use the reference. Always early-return on `!Utilities.IsValid(player)` if you stash the reference.

---

## Trigger callbacks

The GameObject must have a `Collider` set to **Is Trigger** for these to fire. A player's capsule collider triggers them on entering / exiting / staying inside the volume.

```csharp
public virtual void OnPlayerTriggerEnter(VRCPlayerApi player)
public virtual void OnPlayerTriggerExit(VRCPlayerApi player)
public virtual void OnPlayerTriggerStay(VRCPlayerApi player)
```

All three fire for the capsule collider of **any** player in the instance, not just the local one. **[Docs]** The official docs describe `OnPlayerTriggerStay` as firing "every frame" while the player is inside; since it's backed by Unity's physics callbacks it is likely tied to the physics step like regular `OnTriggerStay`. **[Unverified]**

---

## Collision callbacks

For non-trigger colliders that contact a player:

```csharp
public virtual void OnPlayerCollisionEnter(VRCPlayerApi player)
public virtual void OnPlayerCollisionExit(VRCPlayerApi player)
public virtual void OnPlayerCollisionStay(VRCPlayerApi player)
public virtual void OnPlayerParticleCollision(VRCPlayerApi player)
public virtual void OnControllerColliderHitPlayer(VRC.SDK3.ControllerColliderPlayerHit hit)
```

- `OnPlayerParticleCollision` requires the GameObject to be a particle system with **Collision** and **Send Collision Messages** enabled. **[Docs]**
- `OnControllerColliderHitPlayer` is the player equivalent of Unity's `CharacterController.OnControllerColliderHit`. The `ControllerColliderPlayerHit` struct carries the player, collider, point, normal, etc. **[Source]**

---

## Join / leave

These fire on **every** `UdonSharpBehaviour` in the scene that overrides them — not just the one a player is "near."

```csharp
public virtual void OnPlayerJoined(VRCPlayerApi player)
public virtual void OnPlayerLeft(VRCPlayerApi player)
```

- When you join, `OnPlayerJoined` fires for **every player already in the instance, including yourself**; afterwards only for each newcomer. **[Docs]** Combine with `Networking.IsNetworkSettled` for a "world ready" signal.
- `OnPlayerLeft` fires before the player object is invalidated, so `player.displayName` is still readable. **[Community-known]** Still guard stored references with `Utilities.IsValid`, as the official example does. **[Docs]**
- **The local player's persistent data cannot be saved in their own `OnPlayerLeft`.** **[Docs]**
- For persistent data, wait for `OnPlayerRestored(VRCPlayerApi)` (SDK 3.7.4+) rather than `OnPlayerJoined` — PlayerData may arrive later and overwrite early writes. **[Docs]** See [Lifecycle → persistence events](lifecycle-events.md#persistence-events).

### Player state (SDK 3.7.0+)

```csharp
public override void OnPlayerSuspendChanged(VRCPlayerApi player)
public override void OnMasterTransferred(VRCPlayerApi newMaster)
```

`OnPlayerSuspendChanged` fires when any player's device sleeps or switches app (and on wake); read `player.isSuspended`. `OnMasterTransferred` fires for everyone when the master leaves and a new one is picked. **[Docs]** Details in [Lifecycle events](lifecycle-events.md#player-state-platform-and-client-settings).

---

## Spawn / respawn

```csharp
public virtual void OnPlayerRespawn(VRCPlayerApi player)
```

Per the official docs, `OnPlayerRespawn` fires when the **local** player respawns by clicking "Respawn" in their menu. **[Docs]** Behaviour for `VRCPlayerApi.Respawn()` calls and remote players is undocumented. **[Unverified]**

`OnSpawn()` is **not** a player event — it's a deprecated object-pool event (use `OnEnable`). **[Docs]**

---

## Avatar

```csharp
public virtual void OnAvatarChanged(VRCPlayerApi player)
public virtual void OnAvatarEyeHeightChanged(VRCPlayerApi player, float prevEyeHeightAsMeters)
```

`OnAvatarChanged` fires on every client when `player`'s avatar finishes loading (initial load and on every subsequent change). `OnAvatarEyeHeightChanged` fires whenever an avatar swap or runtime scaling changes the height; the parameter is the *previous* height (may be `0` on first load; for remote players it can fire several times and before `OnAvatarChanged`). **[Source]** **[Docs]**

---

## Station

```csharp
public virtual void OnStationEntered(VRCPlayerApi player)
public virtual void OnStationExited(VRCPlayerApi player)
```

A `VRCStation` component must be on the same GameObject. The events fire when `player` sits / stands — for **both local and remote** players, so check `player.isLocal` if you only care about one. **[Docs]** Parameterless legacy forms are `[Obsolete(error: true)]`. **[Source]**

---

## Pickup events

Covered in [Interaction events](interaction-events.md) — they don't take a `VRCPlayerApi` argument because the pickup is always interacted with by the local player.

PhysBone grabs and Contact touches by players (SDK 3.10.0+) are in [Dynamics events](dynamics-events.md).

---

## Example: floor that ejects whoever touches it

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.None)]
public class EjectFloor : UdonSharpBehaviour {
    public Vector3 ejectVelocity = new Vector3(0, 8f, 0);

    public override void OnPlayerCollisionEnter(VRCPlayerApi player) {
        if (player == null || !Utilities.IsValid(player) || !player.isLocal) return;
        player.SetVelocity(ejectVelocity);
    }
}
```

Note the `player.isLocal` guard — `OnPlayerCollisionEnter` fires on **every** client when **any** player collides, but only the local player can act on velocity changes meaningfully. Without that guard, every client would try to push every other player, and only the colliding player's local sim matters.

## See also

- [Players → VRCPlayerApi](../players/vrc-player-api.md) — what you can read off the player reference.
- [Interaction events](interaction-events.md) — `Interact()`, pickup events.
- [Lifecycle events](lifecycle-events.md) — the network sync subset, plus player-state / settings / persistence events.
- [Dynamics events](dynamics-events.md) — PhysBone grabs, Contact enter/exit.
