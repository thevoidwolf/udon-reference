# Instantiate

UdonSharp aliases a restricted set of `Object.Instantiate` overloads to a VRChat extern via the compiler shim `UdonSharp.Lib.Internal.InstantiationShim`. Source: `Packages/com.vrchat.UdonSharp/Runtime/Libraries/CompilerInternal/InstantiationShim.cs`. **[Source]**

## Supported overloads

```csharp
GameObject Instantiate(GameObject original);
GameObject Instantiate(GameObject original, Vector3 position, Quaternion rotation);
GameObject Instantiate(GameObject original, Transform parent);
GameObject Instantiate(GameObject original, Transform parent, bool worldPositionStays);
GameObject Instantiate(GameObject original, Vector3 position, Quaternion rotation, Transform parent);
```

Each clone is **local** to the calling client. Other clients see nothing.

## Constraints

| Rule | Detail |
|---|---|
| **GameObject only** | No generic `Instantiate<T>(T original)`. Clone the GameObject and `GetComponent` on the result. |
| **Source must be in the scene** | Project-asset prefab references cannot be resolved at runtime. Place a disabled "template" GameObject in your scene and `Instantiate` from there. |
| **No `AddComponent` afterward** | Whatever components the template has are what the clone has. |
| **Not networked** | The new GameObject exists only on this client. For networked spawning use [`VRCObjectPool`](../networking/object-pool.md). |
| **References survive** | If the template's `UdonBehaviour` has serialized references to other in-scene objects, the clone's components keep those references. |

## Canonical pattern

Author template GameObjects beneath a disabled root so they don't render or update:

```
Scene root
├── _Templates (active = false)
│    └── Bullet                  ← active=true, has all the components we'll need
└── Game
     └── Spawner
```

Then:

```csharp
public GameObject bulletTemplate;
public Transform muzzle;

void Fire() {
    GameObject b = Instantiate(bulletTemplate, muzzle.position, muzzle.rotation);
    b.SetActive(true);
    // Now configure: set fields on its UdonBehaviour, push velocity, etc.
}
```

## Limits & gotchas

- **Hierarchy preserved.** The clone retains the template's children. If the template has 50 child GameObjects, all 50 are cloned.
- **Component initial state preserved.** UdonBehaviours come up with the template's serialized field values (not their compiled defaults — they preserve inspector-set values).
- **No notification.** There is no `OnInstantiated` event on the clone. Use `Start()` (it will fire on the next frame) or call a setup method explicitly after spawning.
- **Garbage collection.** Clones not actively cleaned up via `Destroy()` accumulate. Long-running worlds that spawn frequently must pair every `Instantiate` with an eventual `Destroy`.

## What you *cannot* do

```csharp
GameObject go = new GameObject();                                  // blocked
gameObject.AddComponent<MeshFilter>();                             // blocked
var clone = Instantiate(prefabFromProjectAssets);                  // resolves to null/error
var c = Instantiate(originalComponent);                            // blocked (GameObject only)
```

See [Runtime constraints](runtime-constraints.md).

## Community libraries for spawning

- **`Pokeyi/VRC-Object-Spawner`** — drop-in network-synced spawner wrapping `VRCObjectPool` internals. Saves you the "set owner → spawn → place" boilerplate. **[Community]** <https://github.com/Pokeyi/VRC-Object-Spawner>
- **`naqtn/EXUR_UdonObjectPool`** — alternative pool with explicit acquire/release semantics. **[Community]** <https://github.com/naqtn/EXUR_UdonObjectPool>
- **`CyanLaser/CyanPlayerObjectPool`** — historical per-player object pool, archived after `VRCPlayerObject` shipped. Still in many older worlds. **[Community]** <https://github.com/CyanLaser/CyanPlayerObjectPool>

## See also

- [Destroy](destroy.md)
- [Runtime constraints](runtime-constraints.md)
- [Networking → VRCObjectPool](../networking/object-pool.md) — networked spawning.
- [Graphics → Meshes](../graphics/meshes.md) — swapping the mesh data on a clone.
- [References → Community Resources](../references/community-resources.md)
