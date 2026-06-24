# GameObject Lifecycle

How to create, destroy, parent, and otherwise manipulate GameObjects from Udon. Udon's main constraint: **you cannot synthesize a GameObject from raw nothing.** Everything you spawn must already exist as a GameObject in the scene at load time.

## Pages

| Page | Topic |
|---|---|
| [Instantiate](instantiate.md) | The supported `Instantiate` overloads and their constraints. |
| [Destroy](destroy.md) | Destroying GameObjects and components at runtime. |
| [Transform & parenting](transform-parenting.md) | Position, rotation, scale, parenting, layers, tags. |
| [Runtime constraints](runtime-constraints.md) | What you cannot do (AddComponent, new GameObject, etc.) and why. |

## Decision matrix

| Goal | Primitive |
|---|---|
| Local-only effect | `Instantiate(prefab)` from a scene template |
| Pickup that everyone sees | Pre-place + `VRCObjectSync` |
| Spawnable that anyone can request and everyone sees | [VRCObjectPool](../networking/object-pool.md) |
| Per-player automatic instance | `VRCPlayerObject` + `VRCEnablePersistence` |
| Mass procedural geometry without GameObjects | `VRCGraphics.DrawMeshInstanced` (see [Graphics](../graphics/README.md)) |

## The "no AddComponent at runtime" rule

Udon does not let you call `gameObject.AddComponent<T>()` for *any* type — not `MeshFilter`, not `Collider`, not `Rigidbody`, not `UdonBehaviour`. Whatever components your spawned GameObject needs **must already be on the template** in the scene.

This is the single most surprising constraint coming from regular Unity development. It shapes how you author worlds: instead of building objects out of parts, you build "the most general version of the thing" once, place it disabled in the scene, and clone it at runtime.

See [Runtime constraints](runtime-constraints.md) for the full list.

## See also

- [Graphics → Meshes](../graphics/meshes.md) — you *can* swap a clone's mesh data at runtime.
- [Networking → VRCObjectSync](../networking/object-sync.md)
- [Networking → VRCObjectPool](../networking/object-pool.md)
