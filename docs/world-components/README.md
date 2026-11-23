# World Components

VRChat world components that Udon can drive and that don't fit another section. **Worlds SDK 3.10.6** (Oct 30, 2026) added three: **`VRCBillboard`**, **`VRCVelocityTracker`** and **`VRCPlacementZone`**. The same docs sync also documented the Udon surface of two older components, **`VRCAvatarPedestal`** and **`VRCPortalMarker`**. **[Docs]**

Sources: <https://creators.vrchat.com/worlds/components/> · release notes <https://creators.vrchat.com/releases/release-3-10-6>. **[Docs]**

Components covered elsewhere:

- `VRCPickup`, `VRCStation`: [Players → Pickups & Stations](../players/pickups-stations.md)
- VRC Constraints, PhysBones, Contacts: [Dynamics](../dynamics/README.md)
- `VRCObjectSync`, `VRCObjectPool`: [Networking](../networking/README.md)
- Video players: [Video](../video/README.md)
- `VRCUrlInputField`: [Loading → Trusted URLs](../loading/trusted-urls.md)

## Pages

| Page | Covers |
|---|---|
| [Billboard](billboard.md) | `VRCBillboard` component and `VRCBillboard.Register` / `Unregister`: facing axes, tilt, smoothing, up reference, execution order, the Udon Graph port caveat. |
| [Velocity Tracker](velocity-tracker.md) | `VRCVelocityTracker`: frame and physics-step linear/angular velocity without an Udon update loop. |
| [Placement Zone](placement-zone.md) | `VRCPlacementZone`: block or confine portals, stickers, sharing pedestals and attached items; how zones combine; the undocumented Udon properties. |
| [Pedestals & Portals](pedestals-and-portals.md) | `VRCAvatarPedestal` (`SwitchAvatar`, `SetAvatarUse`) and `VRCPortalMarker` (`roomId`, `RefreshPortal`). |

## Namespace

All five types are in `VRC.SDK3.Components`. **[Docs]** (examples import it) / **[Source]** (extern type names `VRCSDK3Components...` in `Runtime/Udon/External/VRC.Udon.Wrapper.dll`, com.vrchat.worlds 3.10.6)

## Rules that apply to all of them

- **Component writes from Udon are local.** Changing a pedestal's avatar, a portal's destination or a zone's rules affects only the client that ran the code. Sync the state yourself if every player should agree. For placement zones VRChat calls this out explicitly. **[Docs]** (placement zones) / **[Community-known]** (general)
- **Billboards and velocity trackers are per-viewer by nature.** A billboard faces the *local* player's viewpoint, and a tracker measures what the local client sees. Neither has anything to sync. **[Docs]**
- **Both 3.10.6 performance components replace an Udon `Update` loop.** `VRCBillboard` rotates every registered transform in one native pass, and `VRCVelocityTracker` measures without per-frame Udon code. Use them instead of hand-rolled `Update` scripts when you have many objects. **[Docs]**
- **They run late in the frame.** Billboarding runs after `PostLateUpdate`. ClientSim's mirror puts the velocity tracker at execution order 31001, between the `PostLateUpdate` driver (31000) and billboarding (31002). **[Docs]** / **[Source]** `Integrations/ClientSim/Runtime/Helpers/ClientSimBillboardHelper.cs`
- **ClientSim supports billboards.** 3.10.6 added `ClientSimBillboardHelper`, which shares the client's solver, so Play mode in the editor matches VRChat. **[Source]**
- **Undocumented members are unsupported.** `VRCPlacementZone` has an Udon API in the extern table but none in the docs. Treat it as subject to change. **[Unverified]**

## Version history

| SDK / date | Change |
|---|---|
| 3.10.6 (Oct 30, 2026) | `VRCBillboard` (component + static `Register`/`Unregister`), `VRCVelocityTracker` and `VRCPlacementZone` added for worlds. `VRCBillboard` and `VRCVelocityTracker` were added to the allowlisted world components list. **[Docs]** |
| 3.10.6-beta.3 | `VRCBillboard`'s smoothing factor field became a 0-1 slider in the inspector. **[Docs]** |
| Oct 2026 docs sync | `VRCAvatarPedestal`, `VRCPortalMarker`, `VRCPickup` and `VRCStation` pages gained "Udon property" columns and Udon API sections. These are existing APIs, documented for the first time. **[Docs]** |

## See also

- [Players → Pickups & Stations](../players/pickups-stations.md)
- [Dynamics](../dynamics/README.md): Constraints gained a `SmoothingFactor` in 3.10.6 with the same easing: about 63% of the way to the target per `SmoothingFactor` seconds.
- [UdonSharpBehaviour → Lifecycle events](../udonsharp-behaviour/lifecycle-events.md): `PostLateUpdate` and frame order.
- [Networking](../networking/README.md): syncing component state that Udon changes locally.
- [Tweening](../tweening/README.md): `VRCTween`, another built-in way to animate transforms without per-frame Udon code.
