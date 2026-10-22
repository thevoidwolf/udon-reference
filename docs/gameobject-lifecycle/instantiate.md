# Instantiate

UdonSharp aliases a restricted set of `Object.Instantiate` overloads to a VRChat extern (`VRCInstantiate`) via the compiler shim `UdonSharp.Lib.Internal.InstantiationShim`. Source: `Packages/com.vrchat.UdonSharp/Runtime/Libraries/CompilerInternal/InstantiationShim.cs` (re-checked against <https://github.com/vrchat-community/UdonSharp>, U# 1.1.9). **[Source]** The Udon VM docs confirm `VRCInstantiate` is the one extern whose Udon type name is "falsified". **[Docs]** <https://creators.vrchat.com/worlds/udon/vm-and-assembly/>

## Supported overloads

```csharp
GameObject Instantiate(GameObject original);
GameObject Instantiate(GameObject original, Vector3 position, Quaternion rotation);
GameObject Instantiate(GameObject original, Transform parent);
GameObject Instantiate(GameObject original, Transform parent, bool worldPositionStays);
GameObject Instantiate(GameObject original, Vector3 position, Quaternion rotation, Transform parent);
```

Only the single-argument form is a real extern; the others are U# helpers that clone and then call `SetPositionAndRotation` / `SetParent`. Note `(original, parent)` parents with `worldPositionStays: false`, while `(original, position, rotation, parent)` sets the world pose first and then parents with `worldPositionStays: true`. **[Source]**

Each clone is **local** to the calling client. Other clients see nothing. Instantiated objects "cannot be correctly synchronized … the only workaround is to use object-pooling." **[Docs]** <https://udonsharp.docs.vrchat.com/networking-tips-&-tricks/>

## Constraints

| Rule | Detail |
|---|---|
| **GameObject only** | No generic `Instantiate<T>(T original)`. Clone the GameObject and `GetComponent` on the result. |
| **Source must be referenced** | There is no `Resources.Load`, so the original must come from a serialized field. Placing a disabled "template" GameObject in the scene is the conventional, always-works approach. Whether a *project-asset prefab* assigned to a field instantiates correctly in-client isn't covered by the official docs — test it in Build & Test before relying on it. **[Unverified]** |
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
- **No notification.** There is no `OnInstantiated` event on the clone. Use `Start()` or call a setup method explicitly after spawning. (ClientSim was changed in SDK 3.6.1 so Udon behaviours instantiated by other scripts "initialize right away and have variables and functions ready", matching the client. **[Docs]** (release notes))
- **Dynamics on clones (SDK 3.10.0+).** PhysBones on objects created with `Object.Instantiate()` simulate properly (before 3.10.0 they stayed rigid), but their grabs and poses do **not** sync between clients — same as Udon behaviours on instantiated objects. **[Docs]** (release notes)
- **`GetComponent` on clones.** Works on standard components, U# behaviours, and (since SDK 3.7.5) all TextMeshPro components. **[Docs]** (release notes)
- **Garbage collection.** Clones not actively cleaned up via `Destroy()` accumulate. Long-running worlds that spawn frequently must pair every `Instantiate` with an eventual `Destroy`.

## What you *cannot* do

```csharp
GameObject go = new GameObject();                                  // blocked
gameObject.AddComponent<MeshFilter>();                             // blocked
var o = Resources.Load<GameObject>("Bullet");                      // blocked (no Resources API)
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
