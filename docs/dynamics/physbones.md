# PhysBones

PhysBones add secondary motion to a chain of transforms (hair, tails, plants, wires, cloth substitutes). Players can grab them, and can also pose them, which freezes them in a position. Worlds have supported them since SDK 3.10.0, and in worlds you script them with Udon instead of animator parameters. **[Docs]**

Source: <https://creators.vrchat.com/common-components/physbones>. **[Docs]**

## Components

| Component | Namespace | Purpose |
|---|---|---|
| `VRCPhysBone` | `VRC.SDK3.Dynamics.PhysBone.Components` | Defines and simulates a bone chain. **[Docs]** |
| `VRCPhysBoneCollider` | `VRC.SDK3.Dynamics.PhysBone.Components` | Sphere, Capsule or Plane collider that affects PhysBones. **[Docs]** |
| `VRCPhysBoneRoot` | not documented | **World-only, optional.** Sets the movement root and the update `Timing` for the child PhysBones. **[Docs]** |

`VRCPhysBoneRoot.Timing` has three modes. `Automatic` is the default and switches between fixed and real time to reduce jitter. `Fixed Time` uses the `FixedUpdate` step and `Real Time` uses the `Update` step. The docs don't say whether this is exposed to Udon. **[Docs]** / Udon access **[Unverified]**

## Events

Add an Udon Graph or U# behaviour **to the same GameObject as the `VRCPhysBone`**. **[Docs]**

```csharp
OnPhysBoneGrabbed(PhysBoneGrabbedInfo physBoneInfo)     // grabbed by a player
OnPhysBoneReleased(PhysBoneReleasedInfo physBoneInfo)   // released from a player's grab
OnPhysBonePosed(PhysBonePosedInfo physBoneInfo)         // locked to a pose by a player
OnPhysBoneUnPosed(PhysBoneUnPosedInfo physBoneInfo)     // pose released
```

Signatures are verbatim from the docs. **[Docs]** In U#, write them as `public override void ...` like other built-in events. This form wasn't checked against the bundled `UdonSharpBehaviour` source during this refresh. **[Unverified]**

**The docs don't list the members of the `PhysBone*Info` structs.** According to the 3.10.0 release notes, `OnPhysBoneGrabbed()` "now receives the initiating player" in ClientSim, so the grabbed info carries a player reference of some kind. The field name is not documented. **[Unverified]**

## Read-only state

| Member | Notes |
|---|---|
| `IsGrabbed` | Currently grabbed. **[Docs]** |
| `IsPosed` | Currently posed. **[Docs]** |
| `Angle` | Current angle value. **[Docs]** |
| `Squish` | Current squish value. **[Docs]** |
| `Stretch` | Current stretch value. **[Docs]** |

Don't confuse `Stretch` (capital S, the live value you read) with `stretch` (the configuration field below). **[Docs]**

## Configuration fields

"You can both read and set all the properties of a PhysBone via the appropriately named fields (`pull`, `stretch`, etc.), as well as the curves those properties can use (`pullCurve`, `stretchCurve`, etc.)." **[Docs]**

The docs give only those four names as examples. The rest presumably follow the inspector names (Spring, Stiffness, Gravity, Immobile, Radius…) in camelCase, but they aren't listed. Check the Udon Class Exposure Tree before you rely on one. **[Unverified]**

After any write, call:

```csharp
physBone.ApplyConfigurationChanges();
```

If you don't, the changes aren't applied. Batch your writes and apply once. Config changes are expensive, so don't make them every frame. **[Docs]**

## Methods

| Method | Effect |
|---|---|
| `ApplyConfigurationChanges()` | Commits field changes to the simulation. **[Docs]** |
| `ReleaseGrabs()` | Forces the **local** player to let go if they're grabbing it. **[Docs]** |
| `ReleasePoses()` | Releases any frozen poses so the chain returns to its natural shape. Runs **locally**. **[Docs]** |

Both release methods run on the local client only. To release for everyone, call them from a [network event](../networking/custom-network-events.md) that all clients receive. **[Docs]**

## VRCPhysBoneCollider (3.10.4+)

Since SDK 3.10.4, PhysBone colliders in worlds are exposed to Udon. **[Docs]**

| Settable property | Notes |
|---|---|
| `shapeType` | Sphere, Capsule or Plane. Plane colliders don't support Global Collision. **[Docs]** |
| `radius` | **[Docs]** |
| `height` | Capsule only. Includes the end caps. **[Docs]** |
| `position` | Offset from the root transform. **[Docs]** |
| `rotation` | Offset from the root transform. **[Docs]** |

Then call `collider.ApplyConfigurationChanges()`. The same cost warning applies. **[Docs]**

Inspector-only settings that the docs don't list as Udon-settable: `Root Transform`, `Global Collision`, `Inside Bounds`, `Bones As Sphere`. **[Docs]**

## Collision filtering

- **Allow Collision** (on the PhysBone) can be True (global colliders plus the listed ones), False (only the `Colliders` list) or Other. For a world PhysBone, Other lets you choose whether it responds to colliders in the **world**, on **avatars** and/or on **items**. **[Docs]**
- **Global Collision** (on the collider) uses the same True/False/Other options. A world collider set to Other can target world, avatar and/or item PhysBones. The PhysBone's own Allow Collision rules still apply. **[Docs]**
- The 3.10.0 notes mention "compatibility changes related to the `PermissionFilter` struct used by PhysBones". They added new constructors, including a parameterless default, and these "set the `contentTypes` property to `Everything`". `PermissionFilter`'s other members, and which PhysBone fields use it, aren't documented. **[Docs]** / members **[Unverified]**

### Global colliders

- **Worlds:** no limit on global colliders. **[Docs]**
- **Avatars:** every avatar's hand and finger standard colliders already act as global PhysBone colliders that can touch **world** PhysBones. **[Docs]** Since 3.10.4, avatar authors can mark up to four more PhysBone colliders as global. These collide with PhysBones "in worlds or other avatars, if those also allow avatar collision". **[Docs]** For a world PhysBone to react to them, its Allow Collision must accept avatar colliders.

## Limits and gotchas

- Maximum 256 transforms per component, root included. Consider splitting above about 128. **[Docs]**
- The bounding box is capped at 10×10×10 m. Bones outside it still simulate but may be impossible to touch or grab. **[Docs]**
- Don't put a Constraint on the same GameObject as a PhysBone. **[Docs]**
- Dynamic Bones in worlds are **not** auto-converted to PhysBones. VRChat recommends PhysBones. **[Docs]**
- Instantiated PhysBones simulate (3.10.0+), but their grabs and poses don't sync. **[Docs]**

## Example

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDK3.Dynamics.PhysBone.Components;

public class BellRope : UdonSharpBehaviour
{
    public VRCPhysBone rope;   // the VRCPhysBone on this same GameObject
    public AudioSource bell;

    public override void OnPhysBoneGrabbed(PhysBoneGrabbedInfo physBoneInfo)
    {
        bell.Play();
    }

    // Stiffen the rope once, e.g. from a settings toggle. Not per-frame.
    public void MakeStiff()
    {
        rope.pull = 0.8f;
        rope.ApplyConfigurationChanges();
    }

    // Local only. Call via SendCustomNetworkEvent to release for everyone.
    public void ReleaseAll()
    {
        rope.ReleaseGrabs();
        rope.ReleasePoses();
    }
}
```

Every member used above is documented. The `override` form, and the `using` lines that the info-struct types need, are **[Unverified]** (see Events).

## See also

- [Dynamics overview](README.md)
- [UdonSharpBehaviour → Dynamics events](../udonsharp-behaviour/dynamics-events.md)
- [Contacts](contacts.md): the hand and finger standard colliders also act as contact senders.
- [Constraints](constraints.md)
- [UdonSharpBehaviour](../udonsharp-behaviour/README.md)
- [Networking → Custom network events](../networking/custom-network-events.md): broadcast `ReleaseGrabs()` / `ReleasePoses()`.
- [GameObject Lifecycle → Instantiate](../gameobject-lifecycle/instantiate.md)
- [Persistence → Player Objects](../persistence/player-objects.md)
