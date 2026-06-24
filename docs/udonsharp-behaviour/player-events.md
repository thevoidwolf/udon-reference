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

`OnPlayerTriggerStay` fires *per FixedUpdate while the player is inside*. Treat it like the regular `OnTriggerStay`. **[Source]**

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

- `OnPlayerParticleCollision` requires the GameObject to be a particle system whose particles collide with the player.
- `OnControllerColliderHitPlayer` is the player equivalent of Unity's `CharacterController.OnControllerColliderHit`. The `ControllerColliderPlayerHit` struct carries the player, collider, point, normal, etc. **[Source]**

---

## Join / leave

These fire on **every** `UdonSharpBehaviour` in the scene that overrides them — not just the one a player is "near."

```csharp
public virtual void OnPlayerJoined(VRCPlayerApi player)
public virtual void OnPlayerLeft(VRCPlayerApi player)
```

- `OnPlayerJoined` also fires for the **local** player exactly once at world load. Useful as a "world ready" signal in conjunction with `Networking.IsNetworkSettled`.
- `OnPlayerLeft` fires before the player object is invalidated, so `player.displayName` is still readable.
- **Persistence cannot be saved from `OnPlayerLeft`** — the write is rejected by the runtime. **[Docs]**

---

## Spawn / respawn

```csharp
public virtual void OnSpawn()                              // local player spawned
public virtual void OnPlayerRespawn(VRCPlayerApi player)   // any player respawned
```

`OnSpawn` is local-only. `OnPlayerRespawn` fires on every client when *any* player calls `Respawn()` or triggers a respawn volume.

---

## Avatar

```csharp
public virtual void OnAvatarChanged(VRCPlayerApi player)
public virtual void OnAvatarEyeHeightChanged(VRCPlayerApi player, float prevEyeHeightAsMeters)
```

`OnAvatarChanged` fires on every client when `player`'s avatar finishes loading (initial load and on every subsequent change). `OnAvatarEyeHeightChanged` fires whenever an avatar swap or runtime scaling changes the height; the parameter is the *previous* height. **[Source]** **[Docs]**

---

## Station

```csharp
public virtual void OnStationEntered(VRCPlayerApi player)
public virtual void OnStationExited(VRCPlayerApi player)
```

A `VRCStation` component must be on the same GameObject. The events fire when `player` sits / stands. Parameterless legacy forms are `[Obsolete(error: true)]`. **[Source]**

---

## Pickup events

Covered in [Interaction events](interaction-events.md) — they don't take a `VRCPlayerApi` argument because the pickup is always interacted with by the local player.

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
- [Lifecycle events](lifecycle-events.md) — the network sync subset.
