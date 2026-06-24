# VRCObjectPool

`VRC.SDK3.Components.VRCObjectPool` manages a fixed-size array of pre-placed GameObjects and synchronizes their active/inactive state across the network. It is the SDK's answer to *"how do I spawn things for everyone?"*. **[Docs]**

## Setup

1. Create a parent GameObject; add `VRCObjectPool` to it.
2. Pre-place child GameObjects under the parent, each carrying whatever components you want on a "spawned thing" — meshes, colliders, rigidbodies, UdonBehaviours, `VRCObjectSync` if needed.
3. Drag each child into the `Pool` array on the `VRCObjectPool` inspector. Children start **inactive**.
4. The pool's owner (typically the first player) can call `TryToSpawn` / `Return`.

The pool size is **fixed at scene authoring time** — you cannot grow it at runtime.

## Methods

| Method | Behaviour |
|---|---|
| `GameObject TryToSpawn()` | Activates one pooled GameObject and returns its reference. Returns `null` if the pool is empty. The active state replicates to all clients. **[Docs]** |
| `void Return(GameObject obj)` | Deactivates `obj` and puts it back into the available pool. **[Docs]** |

Only the **owner** of the pool can spawn or return; calls from non-owners are no-ops. Use `Networking.SetOwner(Networking.LocalPlayer, poolGameObject)` if you want a local player to be able to spawn.

## Lifecycle event

UdonBehaviours **on the spawned object itself** receive:

```csharp
public virtual void OnSpawn()
```

`OnSpawn` fires on every client when the pool activates the object. Use this as your "I'm now alive, set me up" hook.

## Late joiners

When a player joins an instance, the pool automatically replicates the current active/inactive state of every member. They see the correct set of spawned objects on join. **[Docs]**

## Patterns

### Owner-side spawn loop

```csharp
public VRCObjectPool pool;
public Transform spawnPoint;

public void Spawn() {
    if (!Networking.IsOwner(pool.gameObject))
        Networking.SetOwner(Networking.LocalPlayer, pool.gameObject);

    GameObject obj = pool.TryToSpawn();
    if (obj == null) return;
    obj.transform.position = spawnPoint.position;
    obj.transform.rotation = spawnPoint.rotation;
}
```

### Per-object init on spawn

```csharp
// On the pooled prefab's UdonBehaviour:
public override void OnSpawn() {
    rb.velocity = transform.forward * 10f;
    SendCustomEventDelayedSeconds(nameof(SelfReturn), 5f);
}
public void SelfReturn() {
    // Only the pool owner can actually return; ask via custom network event.
    SendCustomNetworkEvent(NetworkEventTarget.Owner, nameof(ReturnMeToPool));
}
public void ReturnMeToPool() {
    pool.Return(gameObject);
}
```

### Player-driven spawn (everyone shares one pool)

```csharp
public override void Interact() {
    SendCustomNetworkEvent(NetworkEventTarget.Owner, nameof(SpawnAtMe));
    // synced field to tell the owner where to spawn
    _spawnAt = transform.position;
}
[UdonSynced] Vector3 _spawnAt;

public void SpawnAtMe() {
    if (!Networking.IsOwner(pool.gameObject)) return;
    GameObject obj = pool.TryToSpawn();
    if (obj) obj.transform.position = _spawnAt;
}
```

(A more robust version uses `RequestSerialization` plus `OnDeserialization` to ensure `_spawnAt` arrives before `SpawnAtMe`, but this illustrates the pattern.)

## Gotchas

- The pool itself has an owner (the parent GameObject). Don't conflate it with the ownership of spawned children — each pooled GameObject is its own networked entity with its own ownership.
- Calling `TryToSpawn` from a non-owner returns `null` silently.
- The pool does not initialize the GameObject's position. The previous user's transform is retained until you overwrite it.

## Community alternatives

- **CyanPlayerObjectPool** (CyanLaser) — historically *the* "per-player object" pattern. Master-authoritative pool that assigns one unique GameObject per player on join. **Archived Oct 2024** after VRChat shipped native `VRCPlayerObject`, but extensively used in older worlds; documented design is still worth reading. **[Community]** <https://github.com/CyanLaser/CyanPlayerObjectPool>
- **EXUR_UdonObjectPool** (naqtn) — alternative pool with explicit acquire/release semantics. **[Community]** <https://github.com/naqtn/EXUR_UdonObjectPool>
- **VRC-Object-Spawner** (Pokeyi) — drop-in spawner that wraps `VRCObjectPool` for "spawn-and-forget" ergonomics. **[Community]** <https://github.com/Pokeyi/VRC-Object-Spawner>
- **LSNPlayerObjectPool** — fork of CyanPlayerObjectPool. **[Community]** <https://github.com/LastationVRChat/LSNPlayerObjectPool>

## See also

- [Ownership](ownership.md)
- [VRCObjectSync](object-sync.md) — usually paired with pooled physics objects.
- [GameObject Lifecycle](../gameobject-lifecycle/README.md) — `Instantiate` vs Pool.
- [Persistence → Player Objects](../persistence/player-objects.md) — native replacement for CyanPlayerObjectPool.
