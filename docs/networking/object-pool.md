# VRCObjectPool

`VRC.SDK3.Components.VRCObjectPool` manages a fixed-size array of pre-placed GameObjects and synchronizes their active/inactive state across the network. It is the SDK's answer to *"how do I spawn things for everyone?"*. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-components/#vrc-object-pool>

## Setup

1. Create a parent GameObject; add `VRCObjectPool` to it.
2. Pre-place child GameObjects under the parent, each carrying whatever components you want on a "spawned thing" — meshes, colliders, rigidbodies, UdonBehaviours, `VRCObjectSync` if needed.
3. Drag each child into the `Pool` array on the `VRCObjectPool` inspector. Children start **inactive**.
4. The pool's owner (typically the first player) can call `TryToSpawn` / `Return`.

The pool size is **fixed at scene authoring time** — you cannot grow it at runtime.

## Methods

| Method | Behaviour |
|---|---|
| `GameObject TryToSpawn()` | Activates one pooled GameObject and returns it, or `null` if none are available. The active state replicates to all clients. **[Docs]** |
| `void Return(GameObject obj)` | Returns `obj` to the pool and disables it automatically. **[Docs]** |

Only the **owner** of the pool spawns or returns. **[Docs]** What happens when a non-owner calls these isn't documented; in practice nothing replicates. **[Community-known]** Use `Networking.SetOwner(Networking.LocalPlayer, poolGameObject)` if you want a local player to be able to spawn.

The pool's internal `StartPositions` / `StartRotations` fields are not part of the Udon API. SDK 3.8.2 fixed them wrongly showing as exposed in the class exposure tree. **[Docs]** <https://creators.vrchat.com/releases/release-3-8-2/>

## Lifecycle event

Use Unity's **`OnEnable()`** on UdonBehaviours of the pooled object as the "I've just been spawned" hook. Spawning activates the GameObject on every client. **[Docs]**

```csharp
void OnEnable() { /* reset state, position, etc. */ }
```

> **`OnSpawn` is deprecated.** `UdonSharpBehaviour` still declares `public virtual void OnSpawn()` **[Source]**, but the official docs say: *"This event is deprecated - use the typical OnEnabled event if you want to do something when an object is 'Spawned' from the pool."* **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-components/#onspawn> The official Udon Example Scene does the same: each pooled box restores its start position in `OnEnable`. **[Docs]**

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
// On the pooled object's UdonBehaviour:
public VRCObjectPool pool;

void OnEnable() {                       // runs on every client when spawned
    rb.velocity = transform.forward * 10f;
    SendCustomEventDelayedSeconds(nameof(_SelfReturn), 5f);
}
public void _SelfReturn() {
    // The timer runs on every client, but only the pool's owner may return.
    if (Networking.IsOwner(pool.gameObject)) pool.Return(gameObject);
}
```

(Don't use `NetworkEventTarget.Owner` from the pooled object for this. It targets the owner of the *pooled object*, who isn't necessarily the owner of the *pool*.)

### Player-driven spawn (everyone shares one pool, SDK 3.8.1+)

```csharp
// On an UdonSharpBehaviour on the SAME GameObject as the VRCObjectPool,
// so NetworkEventTarget.Owner == the pool's owner.
public void RequestSpawnAt(Vector3 where) {
    SendCustomNetworkEvent(NetworkEventTarget.Owner, nameof(SpawnAt), where);
}

[NetworkCallable(maxEventsPerSecond: 2)]
public void SpawnAt(Vector3 where) {
    if (!Networking.IsOwner(pool.gameObject)) return;   // ownership may have moved
    GameObject obj = pool.TryToSpawn();
    if (obj) obj.transform.position = where;
}
```

Before 3.8.1 this needed a synced `_spawnAt` field plus an event, which raced (the event usually arrives before the variable). `[NetworkCallable]` parameters remove the race. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/events/>

Remote clients only see the new position if the pooled object syncs its transform, for example with `VRCObjectSync` plus `FlagDiscontinuity()`. The pool syncs only the active state. **[Docs]**

## Gotchas

- The pool itself has an owner (the parent GameObject). Don't conflate it with the ownership of spawned children — each pooled GameObject is its own networked entity with its own ownership.
- Calling `TryToSpawn` from a non-owner should be treated as unsupported. Guard with `Networking.IsOwner(pool.gameObject)`. **[Community-known]**
- The pool syncs **only active state**. It doesn't reset or sync position. The previous use's transform stays until you overwrite it (the official example scene resets it in `OnEnable`). **[Docs]**

## Community alternatives

- **CyanPlayerObjectPool** (CyanLaser) — historically *the* "per-player object" pattern. Master-authoritative pool that assigns one unique GameObject per player on join. **Archived** (last push May 2024). Superseded by native Player Objects (SDK 3.7.4, Nov 2024). Widely used in older worlds, and its documented design is still worth reading. **[Community]** <https://github.com/CyanLaser/CyanPlayerObjectPool>
- **EXUR_UdonObjectPool** (naqtn) — alternative pool with explicit acquire/release semantics. **[Community]** <https://github.com/naqtn/EXUR_UdonObjectPool>
- **VRC-Object-Spawner** (Pokeyi) — drop-in spawner that wraps `VRCObjectPool` for "spawn-and-forget" ergonomics. **[Community]** <https://github.com/Pokeyi/VRC-Object-Spawner>
- **LSNPlayerObjectPool** — fork of CyanPlayerObjectPool. **[Community]** <https://github.com/LastationVRChat/LSNPlayerObjectPool>

## See also

- [Ownership](ownership.md)
- [VRCObjectSync](object-sync.md) — usually paired with pooled physics objects.
- [GameObject Lifecycle](../gameobject-lifecycle/README.md) — `Instantiate` vs Pool.
- [Persistence → Player Objects](../persistence/player-objects.md) — native replacement for CyanPlayerObjectPool.
