# VRCBillboard

`VRCBillboard` turns a transform to face the local player's viewpoint every frame: the screen camera on desktop, the head in VR. It shipped in **Worlds SDK 3.10.6** (Oct 30, 2026). **[Docs]**

Sources: <https://creators.vrchat.com/worlds/udon/vrc-billboard/> · release notes <https://creators.vrchat.com/releases/release-3-10-6>. **[Docs]**

Registered transforms are rotated by one central pass in the client. That is cheaper than turning each object yourself in an Udon `Update`, and the gap grows with the number of objects. **[Docs]**

A billboard can only aim at (or copy the rotation of) the **local player's viewpoint**. To point an object at anything else, use a `VRCAimConstraint` or `VRCLookAtConstraint`. See [Constraints](../dynamics/constraints.md). **[Docs]**

## Two ways to use it

### The component

Add **Add Component > VRChat > Worlds > VRC Billboard**. It registers its own transform in `OnEnable` and unregisters in `OnDisable`, so toggling the GameObject does the right thing. **Up From** and **Match Viewpoint Rotation** sit under a collapsed **Overrides** foldout. **[Docs]**

To change a setting at runtime, assign the matching property on the component: `Facing`, `AllowTilt`, `SmoothingFactor`, `UpFrom`, `MatchViewpointRotation`. Each assignment re-registers the component, so the change applies on the next billboarding pass. **[Docs]** Getters and setters for all five appear in the 3.10.6 Udon extern table. **[Source]** `Runtime/Udon/External/VRC.Udon.Wrapper.dll` (com.vrchat.worlds 3.10.6)

### The static API

`VRCBillboard.Register` billboards any `Transform`, with or without the component. Everything after the first parameter is optional:

```csharp
VRCBillboard.Register(
    Transform transform,
    VRCBillboardFacing facing = VRCBillboardFacing.Back,
    bool allowTilt = false,
    float smoothingFactor = 0f,
    Transform upFrom = null,
    bool matchViewpointRotation = false);

VRCBillboard.Unregister(Transform transform);
```

Signatures and defaults as documented. **[Docs]** In Udon both are single externs: `Register` takes all six arguments (`__Register__UnityEngineTransform_VRCSDK3ComponentsVRCBillboardFacing_SystemBoolean_SystemSingle_UnityEngineTransform_SystemBoolean__SystemVoid`) and `Unregister` takes one. U# fills in the defaults at compile time. **[Source]** `VRC.Udon.Wrapper.dll` (3.10.6)

The type is `VRC.SDK3.Components.VRCBillboard`. The docs' example imports `VRC.SDK3.Components`, and the extern type name is `VRCSDK3ComponentsVRCBillboard`. **[Docs]** / **[Source]**

Use named arguments to set only what you need, for example `VRCBillboard.Register(transform, allowTilt: true, smoothingFactor: 0.25f)`. **[Docs]** Named arguments that skip optional parameters compile correctly only from SDK 3.10.6, which is the same release that added `VRCBillboard`. Earlier compilers dropped the named argument without a warning. See [Compiler](../compiler/supported-features.md). **[Docs]** (3.10.6 release notes)

## Options

The component fields and the `Register` parameters are the same five options. **[Docs]**

| Inspector / parameter | Default | Effect |
|---|---|---|
| **Facing** / `facing` | `VRCBillboardFacing.Back` | Which of the object's own local axes is aimed at the viewpoint. See [Facing](#facing). |
| **Allow Tilt** / `allowTilt` | `false` | Off: the object only turns around its up axis and stays upright. On: it also pitches to aim straight at the viewpoint. Use off for text and signs, on for sprites and icons. |
| **Smoothing Factor** / `smoothingFactor` | `0` | Eases the turn. `0` or less snaps every frame. The object covers about 63% of the remaining turn every `smoothingFactor` seconds, independent of framerate. |
| **Up From** / `upFrom` | `null` (world up) | Another transform whose up direction the billboard treats as "up". It controls roll and the plane that `allowTilt: false` flattens against. |
| **Match Viewpoint Rotation** / `matchViewpointRotation` | `false` | Off: aim at the viewpoint's *position*. On: copy the viewpoint's *rotation*, so every such object ends up parallel to the screen instead of converging on the viewer. |

The inspector shows Smoothing Factor as a 0-1 slider (since 3.10.6-beta.3). Scripts can pass larger values for a slower turn. **[Docs]**

Setting **Up From** to the billboarded object itself counts as empty, because an object can't be its own up reference. **[Docs]**

### Facing

`VRCBillboardFacing` names the local axis that should point at the viewer. The values match the `Vector3` constants of the same name. **[Docs]**

| Value | Local axis | Typical content |
|---|---|---|
| `Back` (default) | -Z | `Canvas`, TextMeshPro, `Quad`, `Plane`, `SpriteRenderer` |
| `Forward` | +Z | Imported models with a front face along +Z |
| `Up` | +Y | Content drawn on the top of an object |
| `Down` | -Y | Content drawn on the underside |
| `Left` | -X | Content drawn on the left side |
| `Right` | +X | Content drawn on the right side |

Unity's flat content reads correctly from its local -Z side, which is why `Back` is the default. Quick check: select the object and look at the blue Z handle in the Scene view. If the side players should see is the side the arrow comes out of, use `Forward`. If the arrow comes out of the back, use `Back`. **[Docs]**

### Up direction

By default a billboard stays upright against world up and does not roll when the player tilts their head. **[Docs]**

Set `upFrom` when the player's own sense of up has changed, for example when they sit in a station on a wall or in a vehicle that banks. Pass the station or vehicle transform. The client reads it every frame, so content stays upright for the rider while the vehicle moves. **[Docs]**

## Registration rules

- **Keyed by `Transform`, last call wins.** Each transform has one set of options. Registering it again replaces the previous options, whichever script made the earlier call. **[Docs]**
- **`Unregister` is global.** It stops billboarding the transform no matter which script registered it. If another script still needs it, that script must register it again. **[Docs]**
- **Destroyed objects drop out on their own.** You don't need to unregister before destroying a GameObject. Unregister when a live object should stop turning. **[Docs]**
- **The component and the static API share the registry.** The component calls the same registration on enable. So a script that registers or unregisters the component's own transform overrides it until the component is toggled again. This follows from the rules above. **[Unverified]**

## Execution order

The billboarding pass runs **after** `PostLateUpdate`, so it is the last thing to set a registered transform's rotation in a frame. **[Docs]** Reading that rotation from `Update`, `LateUpdate` or `PostLateUpdate` gives you the previous frame's value. **[Docs]**

If another object must match a billboard, don't copy the rotation over in script. Register the second object with its own `VRCBillboard.Register` call, so both resolve in the same pass. **[Docs]**

ClientSim mirrors this with `ClientSimBillboardHelper` (new in 3.10.6). It uses `[DefaultExecutionOrder(31002)]`, which puts it after the PostLateUpdate driver (31000) and after `VRCVelocityTracker` (31001). It uses `VRCCameraSettings.ScreenCamera` as the viewpoint and shares the client's solver, so Play mode in the editor shows the same result. **[Source]** `Packages/com.vrchat.worlds/Integrations/ClientSim/Runtime/Helpers/ClientSimBillboardHelper.cs`

## Udon Graph caveat

Use the `VRCBillboard` → `Register` node. Unconnected ports take their default value, with one exception. An unconnected `Transform` port resolves to **the object the graph is on**, not to `null`. **[Docs]**

- If the target `transform` is the graph's own object, `Register` sees `upFrom` pointing at itself and treats it as world up. Nothing to do. **[Docs]**
- If the target is a **different** object, connect `upFrom` explicitly: a `Const null` for world up, or the transform you want. Otherwise the graph's own object becomes the up reference. **[Docs]**

U# doesn't have this problem, because omitted optional parameters compile to their real defaults.

## Example (U#)

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDK3.Components;

public class NameTag : UdonSharpBehaviour
{
    public Transform vehicle;   // optional: keep upright for riders of a banking vehicle

    void OnEnable()
    {
        // Upright, yaw-only: good for text.
        VRCBillboard.Register(transform, upFrom: vehicle, smoothingFactor: 0.15f);
    }

    void OnDisable()
    {
        VRCBillboard.Unregister(transform);
    }
}
```

Pairing `OnEnable` with `OnDisable` keeps billboarding correct when the object is toggled. **[Docs]** If `vehicle` is left empty, `upFrom` is `null`, which means world up. **[Docs]**

For a model whose front faces +Z, pass `VRCBillboardFacing.Forward` as the second argument. For a sprite or icon that should face the viewer head-on, add `allowTilt: true`. **[Docs]**

## See also

- [World Components overview](README.md)
- [Velocity Tracker](velocity-tracker.md): the other 3.10.6 component that saves you an Udon update loop.
- [Dynamics → Constraints](../dynamics/constraints.md): aim at things other than the viewer. Constraints also gained a `SmoothingFactor` in 3.10.6.
- [UdonSharpBehaviour → Lifecycle events](../udonsharp-behaviour/lifecycle-events.md): where `PostLateUpdate` sits in the frame.
- [Graphics → Camera & quality settings](../graphics/camera-and-quality-settings.md): `VRCCameraSettings`, the viewpoint ClientSim uses.
