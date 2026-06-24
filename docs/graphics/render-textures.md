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
    var cam = Camera.main;
    Vector3 reflectedPos = MirrorPosition(cam.transform.position, mirrorPlane);
    mirrorCam.transform.position = reflectedPos;
    // (oblique projection logic omitted)
}
```

For Udon-style mirrors VRChat ships a built-in `VRC_MirrorReflection` component you should usually use instead — it handles per-eye rendering, layer culling, and Quest constraints.

## Community patterns

- **"Fake compute" via fragment-shader-into-RT.** See [VRCGraphics → fake compute](vrc-graphics.md#the-fake-compute-pattern). `MerlinVR/Blit-Component` is ~10× cheaper than Unity's `Camera` for this. **[Community]** <https://github.com/MerlinVR/Blit-Component>
- **RT pixel readback via PNG-sized Texture2D placeholder.** Udon has no direct RenderTexture read API; the workaround uses a placeholder `Texture2D` baked to PNG and matching RT size. **[Community]** <https://github.com/llealloo/vrc-udon-texture-sampler>
- **RT size rule of thumb (shadertrixx):** make all RenderTexture edges divisible by 16 for maximum platform support. **[Community]** <https://github.com/cnlohr/shadertrixx>
- **Mirror gotcha**: depth-buffer access inside a mirror requires the special helper `GetLinearZFromZDepth_WorksWithMirrors()` because the standard `LinearEyeDepth` macro fails under oblique projection. **[Community]** <https://github.com/cnlohr/shadertrixx>

## Notable community projects

- **CCTV / camera systems** — `rhaamo/CameraSystem` (ordered cameras + auth hook). **[Community]** <https://github.com/rhaamo/CameraSystem>
- **Slideshow / image worlds** — official `vrchat-community/examples-image-loading` plus community `PachyPictureFrame`, `DeacsImageLoader`. **[Community]** <https://github.com/vrchat-community/examples-image-loading>
- **VRC Mirror improvements** — `acertainbluecat/VRCPlayersOnlyMirror` (players-only no-background). **[Community]** <https://github.com/acertainbluecat/VRCPlayersOnlyMirror>

## See also

- [VRCGraphics](vrc-graphics.md) — Blit into RTs.
- [Textures](textures.md) — ReadPixels from RT into Texture2D.
- [Materials](materials.md)
- [References → Community Resources](../references/community-resources.md)
