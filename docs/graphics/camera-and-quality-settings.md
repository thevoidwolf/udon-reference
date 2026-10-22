# Camera & Quality Settings

Two VRChat wrapper classes in `VRC.SDK3.Rendering` let Udon read, and partly control, the user's own cameras and Unity's global quality settings. They were added in **SDK 3.8.1** and extended in 3.9.0 and 3.10.5. **[Docs]** Sources: <https://creators.vrchat.com/worlds/udon/vrc-graphics/vrc-camera-settings/>, <https://creators.vrchat.com/worlds/udon/vrc-graphics/vrc-quality-settings/>, <https://creators.vrchat.com/releases/release-3-8-1/>.

This page also covers the **Camera Dolly** components (SDK 3.9.0+), which drive the user's handheld camera from world-authored paths.

## Version summary

| SDK | Addition |
|---|---|
| 3.8.1 | `VRCCameraSettings` (`ScreenCamera`, `PhotoCamera`), `VRCQualitySettings`, `SetShadowDistance` / `ResetShadowDistance`, events `OnVRCCameraSettingsChanged` / `OnVRCQualitySettingsChanged`. Reserved layers can't have their `LayerCullDistances` changed. **[Docs]** |
| 3.9.0 | `VRCCameraSettings.CullingMask`, `VRCCameraSettings.GetCurrentCamera`, `DepthTextureMode` writable on `PhotoCamera`, writable shadow-cascade splits, Camera Dolly components + `VRCCameraDollyAnimation.Import()`. Setters usable from `Start` in ClientSim. **[Docs]** <https://creators.vrchat.com/releases/release-3-9-0/> |
| 3.10.5 | `VRCQualitySettings.RealtimeReflectionProbes`, `VRCQualitySettings.ShadowmaskMode` (Distance Shadowmask on PC). **[Docs]** <https://creators.vrchat.com/releases/release-3-10-5/> |

From 3.10.5 on, U# code can gate on the SDK version with Assembly Version Defines (for example `#if SDK_310_OR_NEWER`). You must declare the define in your Assembly Definition file. **[Docs]**

## VRCCameraSettings (SDK 3.8.1+)

You reach the class through two static instances. You never get the raw `Camera` components. **[Docs]**

| Accessor | What it is |
|---|---|
| `VRCCameraSettings.ScreenCamera` | The camera rendering the user's view. It is a stereo camera in VR. Changes also affect the mobile Focus View camera. |
| `VRCCameraSettings.PhotoCamera` | The handheld camera's **preview** camera. Its properties update only while `Active` is true. Changes also reach the Stream Camera and every Dolly Multi Cam camera; for multi-cam, reads return the first camera's values. **`null` in ClientSim**, never `null` in the client. |

```csharp
using VRC.SDK3.Rendering;

VRCCameraSettings screen = VRCCameraSettings.ScreenCamera;
Vector3 camPos = screen.Position;
```

### Read-only properties **[Docs]**

| Member | Notes |
|---|---|
| `Vector3 Position`, `Quaternion Rotation` | World space. Both are zero while `Active` is false. |
| `Vector3 Forward`, `Up`, `Right` | Convenience vectors. While inactive they return `Vector3.forward` / `up` / `right`. |
| `int PixelWidth`, `int PixelHeight` | Render-target size, which VR super-sampling can change. |
| `float Aspect` | Render-target aspect ratio. |
| `float FieldOfView` | *Vertical* FOV. It may be inaccurate for VR users. |
| `bool Active` | Always `true` on `ScreenCamera`. On `PhotoCamera` it detects the handheld camera, and it is also `true` while Spout streaming. |
| `bool StereoEnabled` | `true` on `ScreenCamera` in VR. Prefer the Player API (`IsUserInVR`) for VR detection. |
| `CameraMode` | `ScreenCamera`: `Screen`, `FocusView`. `PhotoCamera`: `PhotoOrVideo`, `Print`, `DroneHandheld`, `DroneFPV`, `Unknown` (when inactive). The enum's type name isn't given in the docs. |

### Read-write properties **[Docs]**

| Member | Notes |
|---|---|
| `float NearClipPlane` | Clamped to `0.001`–`0.05`. The user's "Forced Camera Near Distance" setting can override it, so read the value back after you set it. |
| `float FarClipPlane` | Clamped to at least `NearClipPlane + 0.1`. Setting it can move `NearClipPlane` when the user's near distance is "Dynamic". |
| `bool AllowHDR` | Whether the camera submits HDR values. |
| `DepthTextureMode DepthTextureMode` | Turns on the camera depth texture without a shadow-casting realtime light. A depth light in the scene does **not** show up here. `PhotoCamera` always has `Depth`; from SDK 3.9.0 you can add more flags to it, but you can't remove `Depth`. |
| `bool UseOcclusionCulling` | Default `true`. It only matters if the world has baked occlusion data. |
| `bool AllowMSAA` | Set it to `false` to force MSAA off whatever the user's settings. |
| `LayerMask CullingMask` | **SDK 3.9.0+.** The setter **throws on anything except `ScreenCamera`**. You can't change reserved layers, `MirrorReflection` or `InternalUI`. `InternalUI` can read 0 while visible because of camera stacking. |
| `CameraClearFlags ClearFlags`, `Color BackgroundColor` | Background clear mode. `BackgroundColor` applies when `ClearFlags` is `SolidColor`. |
| `float[] LayerCullDistances` | Needs exactly 32 entries. `0` means "use `FarClipPlane`" and `null` resets every layer. Values are clamped to at least `NearClipPlane + 0.1`. Reserved layers and `InternalUI` can't change and always read `0`. |
| `bool LayerCullSpherical` | **Disabled.** Setting it does nothing because it caused UI culling issues. |

The current docs don't say which SDK release added `CameraMode`, `ClearFlags`/`BackgroundColor`, `AllowMSAA` and `UseOcclusionCulling`. Check them against your installed SDK. **[Unverified]**

### Static functions **[Docs]**

```csharp
Vector3    VRCCameraSettings.GetEyePosition(Camera.StereoscopicEye eye);
Quaternion VRCCameraSettings.GetEyeRotation(Camera.StereoscopicEye eye);

// SDK 3.9.0+. Use this instead of Camera.current, which is only set during render events.
void VRCCameraSettings.GetCurrentCamera(out VRCCameraSettings internalComponent, out Camera externalComponent);
```

`GetCurrentCamera` results: **[Docs]**
- An internal camera is rendering: `internalComponent` is `ScreenCamera` or `PhotoCamera`, and `externalComponent` is `null`.
- A camera you placed in the world is rendering: `internalComponent` is `null`, and `externalComponent` holds that `Camera`.
- **Both can be `null` at any time, even during render events.** This happens when `Camera.current` is null, for avatar cameras, and for VRChat's internal render steps such as menus. Always handle this case.

### Events

```csharp
public override void OnVRCCameraSettingsChanged(VRCCameraSettings camera) { }   // SDK 3.8.1+
```

The event fires when the user changes graphics options such as "Near Clip Override" or "Field of View". Your own writes through `VRCCameraSettings` do **not** fire it, and neither does camera movement. It can fire **every frame**, or more than once per frame, while the photo-camera zoom slider moves or a Dolly path changes zoom. Keep the handler cheap. **[Docs]** <https://creators.vrchat.com/worlds/udon/graph/event-nodes/#onvrccamerasettingschanged>

### Example (from the official page) **[Docs]**

```csharp
using TMPro;
using UdonSharp;
using UnityEngine;
using VRC.SDK3.Rendering;

public class CameraInfoDisplay : UdonSharpBehaviour
{
    [SerializeField] private TextMeshProUGUI info;

    void Start()
    {
        OnVRCCameraSettingsChanged(VRCCameraSettings.ScreenCamera);   // initialize once
    }

    public override void OnVRCCameraSettingsChanged(VRCCameraSettings camera)
    {
        if (camera != VRCCameraSettings.ScreenCamera) return;         // ignore handheld camera
        info.text = $"{camera.PixelWidth}x{camera.PixelHeight} fov={camera.FieldOfView}";
    }
}
```

## VRCQualitySettings (SDK 3.8.1+)

A thin layer over `UnityEngine.QualitySettings`. The docs call its properties with the class name (`VRCQualitySettings.SetShadowDistance()`), so treat it as static. **[Docs]**

### Exposed read-only properties **[Docs]**

| Member | Notes |
|---|---|
| `int AntiAliasing` | |
| `int PixelLightCount` | |
| `float LODBias` | |
| `int MaximumLODLevel` | |
| `ShadowResolution ShadowResolution` | |
| `float ShadowDistance` | Set it with `SetShadowDistance`, below. |
| `int ShadowCascades` | |
| `int vSyncCount` | Lower-case `v`, as the docs spell it. |
| `bool RealtimeReflectionProbes` | **SDK 3.10.5+.** |
| `ShadowmaskMode ShadowmaskMode` | **SDK 3.10.5+.** Distance Shadowmask is supported on PC only. |

The user's graphics settings can affect some of these values.

**Can you write `RealtimeReflectionProbes` / `ShadowmaskMode`?** The 3.10.5 release notes say you "can now enable/disable Realtime Reflection Probes on all platforms" and "enable Distance Shadowmask mode on PC… Use `VRCQualitySettings.ShadowmaskMode` from Udon". The property page still lists both under "read-only". Neither source shows a setter signature. Confirm against the SDK before you rely on writing them. **[Unverified]** For edit-time setup, the **World Quality Settings** script on the `VRCWorld` prefab configures Shadow Distance, Shadowmask Lighting Mode and Realtime Probes. **[Docs]**

### Shadow distance override **[Docs]**

```csharp
VRCQualitySettings.SetShadowDistance(float low, float medium, float high, float mobile);
VRCQualitySettings.SetShadowDistance(float distance);   // same value for all four tiers
VRCQualitySettings.ResetShadowDistance();               // go back to the user's own setting
```

- `low` / `medium` / `high` map to the user's "Shadow Quality" setting. `mobile` applies to **every non-PC platform**, whatever its settings.
- Values are clamped to `0.1`–`10000`.
- Setting an override replaces the user's Shadow Quality distance and shows them a warning in graphics settings. Large values cost a lot of performance.
- The single-argument overload's parameter name isn't documented. The name `distance` above is a placeholder.

### Shadow cascade splits (SDK 3.9.0+, read-write) **[Docs]**

```csharp
float   shadowCascade2Split;   // casing as on the docs page
Vector3 shadowCascade4Split;
```

The two sources disagree on casing. The property page uses lower-case `shadowCascade2Split` / `shadowCascade4Split`, while the 3.9.0 release notes write `ShadowCascade2Split` (twice, which is a typo for the 4-split one). Use the editor's autocomplete. **[Unverified]** casing.

All VRCQualitySettings properties reset on world load. **[Docs]**

### Event

```csharp
public override void OnVRCQualitySettingsChanged() { }   // SDK 3.8.1+
```

It fires when the user changes a graphics setting that affects an exposed value. It can fire often, so keep the handler light. **[Docs]** The event takes no parameters, per the graph event-node page. **[Docs]**

## Camera Dolly (SDK 3.9.0+)

Worlds can author camera-dolly animations and push them to the **local player's** handheld VRChat camera. **[Docs]** <https://creators.vrchat.com/worlds/components/vrc_cameradolly/>

There are three components, nested Animation → Path(s) → Point(s):

| Component | Role |
|---|---|
| `VRCCameraDollyAnimation` | The top-level object. It holds Is Relative To Player, Is Speed Based, Is Using Look At Me, Is Using Greenscreen, Is Using Multi Stream, Path Type (interpolation), Loop Type, Capture Type, Focus Mode, Anchor Mode and Paths. |
| `VRCCameraDollyPath` | One path, holding a list of Points. The number of points per path and per animation is limited. |
| `VRCCameraDollyPoint` | One keyframe: Zoom, Duration (time-based) or Speed (speed-based), Focal Distance, Aperture, greenscreen Hue / Saturation / Lightness, and Look At Me X/Y offsets. |

Workflow: **[Docs]**
1. Build the hierarchy, select the top-level object and click **Collect Paths & Points**. Do this again every time the hierarchy changes.
2. Call `Import()` on the `VRCCameraDollyAnimation` to apply its settings and animation to the local user's camera.
3. ClientSim can't preview dolly animations. Editor gizmos show the path, and you need **Build & Test** to see the result.

```csharp
public VRCCameraDollyAnimation dollyAnimation;   // assigned in the inspector

public override void Interact() {
    dollyAnimation.Import();
}
```

The docs name the function `VRCCameraDollyAnimation.Import()` but give neither its return type nor its namespace. The instance-call shape above is inferred. **[Unverified]**

## Related: shader-side camera info

Shaders can read the active camera state without Udon through VRChat's shader globals: `_VRChatCameraMode`, `_VRChatCameraMask`, `_VRChatScreenCameraPos/Rot` and `_VRChatPhotoCameraPos/Rot`. See [VRCShader → VRChat shader globals](vrc-shader.md#vrchat-shader-globals). **[Docs]**

## See also

- [VRCGraphics](vrc-graphics.md)
- [Graphics → Mirrors](README.md#mirrors)
- [Render Textures](render-textures.md)
