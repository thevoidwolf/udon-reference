# Render Textures

## Authoring-time setup

The least surprising way to use a `RenderTexture` is to author it at edit time:

1. Right-click in Project → Create → Render Texture.
2. Configure resolution, depth, format.
3. Drag it onto a `Camera`'s `Target Texture` field.
4. Drag it onto a `Material`'s texture slot to show what the camera sees.

No code required. This works perfectly in Udon worlds — the render target is set up before runtime.

## Runtime construction

`new RenderTexture(width, height, depth, RenderTextureFormat)` is **[Community-known]** to work at runtime.

```csharp
RenderTexture rt = new RenderTexture(512, 512, 24, RenderTextureFormat.ARGB32);
rt.Create();
```

Don't forget `Create()` if you're using the RT immediately — Unity defers GPU allocation otherwise.

Properties:

```csharp
rt.width;
rt.height;
rt.depth;
rt.format;
rt.useMipMap;
rt.autoGenerateMips;
rt.filterMode;
rt.wrapMode;
rt.anisoLevel;
rt.antiAliasing;
```

Most are settable before `Create()`. Some require destroying and re-creating the RT.

## Methods

```csharp
rt.Create();
rt.Release();
rt.GenerateMips();
rt.DiscardContents();
```

All **[Community-known]**.

## Assigning to a Camera

```csharp
Camera cam = GetComponent<Camera>();
cam.targetTexture = rt;
```

The camera renders into `rt` instead of the screen.

## Assigning to a Material

```csharp
material.mainTexture = rt;
material.SetTexture("_BaseMap", rt);
```

## `RenderTexture.active`

```csharp
RenderTexture prev = RenderTexture.active;
RenderTexture.active = rt;
// ... Graphics.Blit / Texture2D.ReadPixels ...
RenderTexture.active = prev;
```

`RenderTexture.active` is the global "where draws go" target. Save and restore it around any blits — leaving it set to a non-screen target breaks subsequent normal rendering. **[Community-known]**

## Reading back to the CPU: `VRCAsyncGPUReadback`

Since **SDK 3.2.0**, Udon can read texture data from the GPU without stalling the main thread. VRChat wraps Unity's `AsyncGPUReadback`. **[Docs]** <https://creators.vrchat.com/worlds/udon/vrc-graphics/asyncgpureadback/>

It differs from Unity's API in three ways: **[Docs]**
- You call `VRCAsyncGPUReadback.Request(...)` instead of `AsyncGPUReadback.Request(...)`.
- You pass an UdonBehaviour, not an `Action<AsyncGPUReadbackRequest>` callback. That behaviour receives `OnAsyncGpuReadbackComplete`.
- You read results with `TryGetData(...)`, not `GetData`.

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDK3.Rendering;

public class ReadbackExample : UdonSharpBehaviour
{
    public Texture texture;   // e.g. a RenderTexture written by a camera or VRCGraphics.Blit

    void Start() {
        VRCAsyncGPUReadback.Request(texture, 0, this);   // mip 0, this behaviour receives the event
    }

    public void OnAsyncGpuReadbackComplete(VRCAsyncGPUReadbackRequest request) {
        if (request.hasError) { Debug.LogError("GPU readback error!"); return; }
        var px = new Color32[texture.width * texture.height];
        if (request.TryGetData(px)) Debug.Log(px[0]);
    }
}
```

Adapted from the official example. **[Docs]** The docs point to Unity's `AsyncGPUReadback.Request` for the other overloads and don't list which ones Udon exposes. **[Unverified]**

## Quest gotcha for `Blit`

If you use `VRCGraphics.Blit` to write into a RenderTexture, on Quest:

- The shader must have `ZTest Always`, **or**
- The destination `RenderTexture` must have depth disabled (`depth = 0` at construction).

Otherwise the operation silently fails on Quest. **[Docs]**

## Cleanup

`RenderTexture` is a `UnityEngine.Object`. Use `Destroy(rt)` to free GPU memory. Don't `Destroy` a RT asset placed in the scene at edit time — VRChat treats those as authored.

## Patterns

### Mirror

A "mirror" world is typically a `Camera` whose position is reflected across the mirror plane each frame, rendering into a `RenderTexture` that's bound to the mirror surface's material:

```csharp
public Camera mirrorCam;
public Transform mirrorPlane;

void LateUpdate() {
    // The player's camera isn't a Camera you can reach. Read its pose through VRCCameraSettings (SDK 3.8.1+).
    Vector3 camPos = VRCCameraSettings.ScreenCamera.Position;
    Vector3 reflectedPos = MirrorPosition(camPos, mirrorPlane);
    mirrorCam.transform.position = reflectedPos;
    // (oblique projection logic omitted)
}
```

For mirrors, use VRChat's built-in `VRCMirrorReflection` component instead in most cases. It handles per-eye rendering, layer culling and Quest constraints. See [Graphics → Mirrors](README.md#mirrors). Since SDK 3.10.3 it renders in `Camera.onPreCull`. **[Docs]** For per-eye positions in a hand-rolled version, use `VRCCameraSettings.GetEyePosition(Camera.StereoscopicEye)`. See [Camera & Quality Settings](camera-and-quality-settings.md). **[Docs]**

## Community patterns

- **"Fake compute" via fragment-shader-into-RT.** See [VRCGraphics → fake compute](vrc-graphics.md#the-fake-compute-pattern). `MerlinVR/Blit-Component` is ~10× cheaper than Unity's `Camera` for this. **[Community]** <https://github.com/MerlinVR/Blit-Component>
- **RT pixel readback via PNG-sized Texture2D placeholder.** This is a pre-SDK-3.2 workaround. It uses a placeholder `Texture2D` baked to PNG at the RT's size, read with `ReadPixels`. **[Community]** <https://github.com/llealloo/vrc-udon-texture-sampler> For new work, prefer [`VRCAsyncGPUReadback`](#reading-back-to-the-cpu-vrcasyncgpureadback), which doesn't block the main thread. **[Docs]**
- **RT size rule of thumb (shadertrixx):** make all RenderTexture edges divisible by 16 for maximum platform support. **[Community]** <https://github.com/cnlohr/shadertrixx>
- **Mirror gotcha**: depth-buffer access inside a mirror requires the special helper `GetLinearZFromZDepth_WorksWithMirrors()` because the standard `LinearEyeDepth` macro fails under oblique projection. **[Community]** <https://github.com/cnlohr/shadertrixx>

## Notable community projects

- **CCTV / camera systems** — `rhaamo/CameraSystem` (ordered cameras + auth hook). **[Community]** <https://github.com/rhaamo/CameraSystem>
- **Slideshow / image worlds** — official `vrchat-community/examples-image-loading` plus community `PachyPictureFrame`, `DeacsImageLoader`. **[Community]** <https://github.com/vrchat-community/examples-image-loading>
- **VRC Mirror improvements** — `acertainbluecat/VRCPlayersOnlyMirror` (players-only no-background). **[Community]** <https://github.com/acertainbluecat/VRCPlayersOnlyMirror>

## See also

- [VRCGraphics](vrc-graphics.md) — Blit into RTs.
- [Textures](textures.md) — ReadPixels from RT into Texture2D.
- [Camera & Quality Settings](camera-and-quality-settings.md) — `VRCCameraSettings` for the player's own camera.
- [Materials](materials.md)
- [References → Community Resources](../references/community-resources.md)
