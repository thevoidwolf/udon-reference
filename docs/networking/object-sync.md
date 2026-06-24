# VRCObjectSync

`VRC.SDK3.Components.VRCObjectSync` automatically replicates a GameObject's `Transform` (position, rotation, scale) and `Rigidbody` (linear/angular velocity, gravity, kinematic). Source: VRC SDK3. **[Docs]**

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
3. Make sure ownership is sensible (often `VRCPickup`'s `Auto Hold` is enough; otherwise transfer manually via `Networking.SetOwner`).

## Methods

| Method | Behaviour |
|---|---|
| `void FlagDiscontinuity()` | Mark this frame as a teleport so remote clients do not lerp into the new position — they snap. Call right after any sudden position/rotation change. **[Docs]** |
| `void SetGravity(bool)` | Toggle gravity on the underlying Rigidbody. Owner-only. **[Docs]** |
| `bool GetGravity()` | Read the current gravity flag. |
| `void SetKinematic(bool)` | Toggle Rigidbody.isKinematic. Owner-only. **[Docs]** |
| `bool GetKinematic()` | Read the current kinematic flag. |
| `void Respawn()` | Teleport back to the position/rotation the GameObject had at scene load, zeroing velocities. Owner-only. **[Docs]** |
| `void TeleportTo(Vector3 position, Quaternion rotation)` | Move to a new transform, implicitly calling `FlagDiscontinuity`. **[Community-known]** for the convenience method; verify the exact name in your SDK version. |

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

A `VRCPickup` with `Auto Hold` and a `VRCObjectSync` on the same GameObject already does this — picking up takes ownership, dropping leaves you as the owner so the post-drop trajectory comes from your physics sim. No code needed.

### Make a Rigidbody act as a moving platform

Use `SetKinematic(true)` so the Rigidbody isn't affected by physics, then move it via Transform in `Update` on the owner. `VRCObjectSync` will replicate the position to other clients.

## Limits / gotchas

- `VRCObjectSync` and `[UdonSynced]` on the same behaviour are both allowed, but they each cost packet budget. Don't double-sync the same data.
- Each client runs its own physics simulation against the *owner*'s authoritative state. Brief desync during high-velocity collisions is normal.
- Calls like `SetGravity` are no-ops from non-owners; always guard on ownership.

## See also

- [Ownership](ownership.md)
- [VRCObjectPool](object-pool.md)
- [Players → VRCPlayerApi](../players/vrc-player-api.md) — `Networking.SetOwner` etc.
