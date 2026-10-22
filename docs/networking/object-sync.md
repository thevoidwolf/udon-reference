# VRCObjectSync

`VRC.SDK3.Components.VRCObjectSync` automatically replicates a GameObject's `Transform` and `Rigidbody` physics, including the gravity and kinematic flags it manages. **[Docs]** <https://creators.vrchat.com/worlds/components/vrc_objectsync/> · <https://creators.vrchat.com/worlds/udon/networking/network-components/#vrc-object-sync>

The two official pages differ on scale: the networking page says "position, rotation scale", and the component page lists only position and rotation (plus kinematic/gravity). Don't rely on scale being synced. **[Docs]**

## When to use it

- Physics objects that must behave consistently for everyone (a ball, a stack of crates).
- A pickup whose post-drop trajectory should match across clients.
- A moving platform whose timing should be authoritative on the owner.

## When *not* to use it

- If the object's state is fully derivable from `[UdonSynced]` fields (e.g. a switch's open/closed state with derived rotation), `VRCObjectSync` duplicates that work and wastes bandwidth.
- For visual-only follow behaviour. Use a regular `UdonSharpBehaviour` that reads `Networking.LocalPlayer.GetPosition()` and updates locally.

## Setup

1. Add a `VRCObjectSync` component to the GameObject.
2. Add a `Rigidbody` if you want physics replication. Without one, only the `Transform` is synced.
3. Make sure ownership is sensible. Grabbing a `VRCPickup` takes ownership **[Community-known]**; otherwise transfer manually via `Networking.SetOwner`.

## Inspector properties

All items in this section: **[Docs]**.

| Property | Effect |
|---|---|
| **Allow Collision Ownership Transfer** | If checked, ownership transfers when the object collides with an object owned by another player. |
| **Force Kinematic On Remote** *(SDK 3.8.0+)* | If checked, the Rigidbody is forced kinematic for all **non-owners**. Use it for objects whose physics should run *only* on the owner. |

"Force Kinematic On Remote" was added in SDK 3.8.0 (Apr 3, 2025). **[Docs]** <https://creators.vrchat.com/releases/release-3-8-0/>

## Methods

| Method | Behaviour |
|---|---|
| `void FlagDiscontinuity()` | Mark this frame as a teleport: changes made this frame are applied without smoothing, so remote clients snap instead of lerping. Call it in the same frame as any sudden position/rotation change. **[Docs]** |
| `void SetGravity(bool value)` | Set the gravity flag. With `VRCObjectSync`, the component controls it, not the Rigidbody. Behaves like a synced variable, so **only the owner** can set it. **[Docs]** |
| `bool GetGravity()` | Read the current gravity flag. **[Docs]** ("Set/Get Gravity") |
| `void SetKinematic(bool value)` | Set the kinematic flag; same rules as gravity, owner-only. **[Docs]** |
| `bool GetKinematic()` | Read the current kinematic flag. **[Docs]** ("Set/Get Kinematic") |
| `void Respawn()` | Teleport back to the starting position and rotation and zero the velocities. Sets the discontinuity hint, so the move is instant. **[Docs]** Owner-only. **[Community-known]** |
| `void TeleportTo(Transform targetLocation)` | Move the object to the given `Transform`'s location. Note the parameter is a **`Transform`**, not `(Vector3, Quaternion)`; that overload belongs to `VRCPlayerApi.TeleportTo`. **[Docs]** <https://creators.vrchat.com/worlds/components/vrc_objectsync/> The docs don't say whether it flags a discontinuity or needs ownership. Assume owner-only. **[Unverified]** |

## Patterns

### Teleport without lerp ghosts

```csharp
VRCObjectSync sync;

void Start() => sync = (VRCObjectSync)GetComponent(typeof(VRCObjectSync));

public void Teleport(Vector3 dest) {
    if (!Networking.IsOwner(gameObject)) return;
    transform.position = dest;
    sync.FlagDiscontinuity();
}
```

### Hold then drop with physics

A `VRCPickup` and a `VRCObjectSync` on the same GameObject already do this. Picking up takes ownership, and dropping leaves you as the owner, so the post-drop trajectory comes from your physics sim. No code needed. **[Community-known]** (`Auto Hold` doesn't matter here; it only changes how grabbing works.)

### Owner-only physics

Enable **Force Kinematic On Remote** (SDK 3.8.0+) when non-owners shouldn't simulate the Rigidbody themselves. Remote clients then just follow the owner's synced transform instead of running their own physics that might diverge. **[Docs]**

### Make a Rigidbody act as a moving platform

Use `SetKinematic(true)` so the Rigidbody isn't affected by physics, then move it via Transform in `Update` on the owner. `VRCObjectSync` will replicate the position to other clients.

## Limits / gotchas

- `VRCObjectSync` and `[UdonSynced]` on the same behaviour are both allowed, but they each cost packet budget. Don't double-sync the same data.
- Each client runs its own physics simulation against the *owner*'s authoritative state. Brief desync during high-velocity collisions is normal.
- Calls like `SetGravity` / `SetKinematic` only take effect for the owner; always guard on ownership. **[Docs]**
- Set gravity/kinematic through `VRCObjectSync`, not directly on the `Rigidbody`. The component owns those flags. **[Docs]**
- Objects with `VRCObjectSync` can **sleep** when at rest and stop sending data. Debug view 6 shows this. **[Docs]** See [Debugging & network stats](debugging-and-stats.md).
- `Networking.SimulationTime(gameObject)` shows how far behind real time the object is being simulated. Useful for building your own `VRCObjectSync`-like system. **[Docs]**

## See also

- [Ownership](ownership.md)
- [VRCObjectPool](object-pool.md)
- [Players → VRCPlayerApi](../players/vrc-player-api.md) — `Networking.SetOwner` etc.
