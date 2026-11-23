# VRCPlacementZone

`VRCPlacementZone` controls where players can place user content in your world: **portals, stickers, sharing pedestals and world-attached items**. It shipped in **Worlds SDK 3.10.6** (Oct 30, 2026). **[Docs]**

Sources: <https://creators.vrchat.com/worlds/components/vrc_placementzone/> · release notes <https://creators.vrchat.com/releases/release-3-10-6>. **[Docs]**

A zone is a box. For each content type it can block placement inside the box, allow placement *only* inside the box, or leave that type alone. With gizmos on, the SDK draws the box in yellow in the Scene view. **[Docs]**

## Inspector properties

**[Docs]**

| Property | Meaning |
|---|---|
| Center | Box centre in the GameObject's local space. |
| Rotation | Box rotation in the GameObject's local space. |
| Size | Width, height and depth in local space. |
| Portals | Rule for user portals. |
| Stickers | Rule for stickers. |
| Sharing Pedestals | Rule for sharing pedestals. |
| Attached Items | Rule for items attached to surfaces. Items that spawn without attaching to a surface, such as pickups, aren't affected. |

## Permission options

Each content type takes one of three values. **[Docs]**

| Inspector label | Enum value | Effect |
|---|---|---|
| Unaffected | `Unaffected` | The zone ignores this content type. |
| Block Here | `BlockHere` | No placement inside the zone. Placement elsewhere is unaffected. |
| Only Allow Here | `OnlyAllowHere` | Placement only inside this zone, or inside another *Only Allow Here* zone. It's blocked everywhere else. |

The enum value names come from `VRCSDK3.dll` in the 3.10.6 Worlds package. The docs give only the inspector labels. **[Source]**

## Combining zones

**[Docs]**

- **Two blocking zones:** content can still go anywhere outside both of them.
- **Two allow-only zones:** content can only go inside one of them (or the overlap). Nothing can be placed outside both.
- **An allow-only zone overlapping a blocking zone:** the overlap **allows** placement. A blocking zone fully inside an allow-only zone does nothing.

In short, *Only Allow Here* beats *Block Here* where they overlap. If you need a no-go area inside an allowed area, shape the allowed zones around it instead.

## Practical rules

- **Sink the zone slightly into the surface you're protecting.** Stickers in particular can go on any surface, not just floors, so a box that stops exactly at a wall may miss it. **[Docs]**
- **Zones prevent placement; they don't remove anything.** Turning on a zone that blocks stickers leaves the stickers already inside it. **[Docs]**
- **Keep zones the same on every client.** If you move a zone or change its rules at runtime, sync that change yourself (synced variables or a network event). Clients that disagree about zones can behave unexpectedly. **[Docs]** See [Networking](../networking/README.md).

## Udon access

The official page documents no Udon API. The 3.10.6 Udon extern table does expose the component, along with these members: **[Source]** `Runtime/Udon/External/VRC.Udon.Wrapper.dll` (com.vrchat.worlds 3.10.6)

| Member | Type | Access |
|---|---|---|
| `Center` | `Vector3` | get / set |
| `Size` | `Vector3` | get / set |
| `Rotation` | `Quaternion` | set (a matching getter signature also exists) |
| `Portals` | `VRCPlacementZone.PlacementPermission` | get / set |
| `Stickers` | `VRCPlacementZone.PlacementPermission` | get / set |
| `SharingPedestals` | `VRCPlacementZone.PlacementPermission` | get / set |
| `AttachedItems` | `VRCPlacementZone.PlacementPermission` | get / set |

The extern table is a flat list of signatures. The member names above are the ones that first appeared in 3.10.6 alongside the `VRCSDK3ComponentsVRCPlacementZone` type, and they match the inspector. The full type is `VRC.SDK3.Components.VRCPlacementZone`. **[Source]**

Because none of this is documented, treat it as usable but unsupported. VRChat could change it without notice. **[Unverified]**

**The enum type.** `PlacementPermission` has no extern module of its own, only the property signatures that use it. `VRC_Pickup.PickupHand` is in the same situation and works fine in U#, so declaring a `VRCPlacementZone.PlacementPermission` field or local probably works too. This hasn't been tested. **[Unverified]**

**Turning a zone on and off.** `VRCSDK3.dll` contains `RegisterPlacementZone` / `UnregisterPlacementZone` on an internal `VRCPlacementZoneHandler`. That suggests zones register when enabled and unregister when disabled, so toggling the component or GameObject should switch a zone on and off. **[Unverified]**

### Example (U#)

A host-controlled switch that closes a stage to portals and stickers during an event. The state is synced so every client applies the same rule.

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDKBase;

[UdonBehaviourSyncMode(BehaviourSyncMode.Manual)]
public class StageLock : UdonSharpBehaviour
{
    public GameObject stageZone;   // holds a VRCPlacementZone with Portals/Stickers set to Block Here

    [UdonSynced, FieldChangeCallback(nameof(Locked))]
    private bool _locked = true;

    public bool Locked
    {
        get => _locked;
        set
        {
            _locked = value;
            stageZone.SetActive(value);   // see the toggle caveat above
        }
    }

    public override void Interact()
    {
        Networking.SetOwner(Networking.LocalPlayer, gameObject);
        Locked = !Locked;
        RequestSerialization();
    }
}
```

Toggling a dedicated GameObject only uses APIs that are certain to compile, and it keeps the enum type out of U#. To switch individual content types instead, reference the `VRCPlacementZone` and assign `zone.Stickers = ...` and similar. That relies on the [Unverified] enum note above.

## See also

- [World Components overview](README.md)
- [Pedestals & Portals](pedestals-and-portals.md): portals and avatar pedestals that *you* place in the world (placement zones are about content players place).
- [Networking → Sync modes](../networking/sync-modes.md) and [Attributes → FieldChangeCallback](../attributes/field-change-callback.md): the sync pattern used above.
