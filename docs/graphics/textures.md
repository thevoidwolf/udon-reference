# Textures

VRChat's *officially documented* runtime path for bringing textures into a world is `VRCImageDownloader` (see [Image Downloader](image-downloader.md)). Lower-level `Texture2D` manipulation is widely used by the community but not formally documented. **[Community-known]**

## Officially documented path

```csharp
VRCImageDownloader dl = new VRCImageDownloader();
dl.DownloadImage(url, targetMaterial, this);
// Texture lands via OnImageLoadSuccess
```

See [Image Downloader](image-downloader.md) for full coverage. That page includes per-download `IVRCImageDownload.Dispose()` (SDK 3.8.0+) for freeing textures you've replaced.

## Texture2D construction (community-known)

```csharp
Texture2D tex = new Texture2D(width, height);
Texture2D tex = new Texture2D(width, height, TextureFormat.RGBA32, mipChain: false);
```

Both forms are **[Community-known]** to work at runtime.

## Pixel-level access

```csharp
tex.SetPixel(int x, int y, Color c);
tex.SetPixels(Color[] colors);
tex.SetPixels32(Color32[] colors);
tex.Apply();                              // upload to GPU
```

**[Community-known]** as functional.

```csharp
Color  c   = tex.GetPixel(x, y);
Color[]    px  = tex.GetPixels();
Color32[]  px32 = tex.GetPixels32();
```

Also **[Community-known]**.

```csharp
tex.ReadPixels(Rect source, int dstX, int dstY);
```

Reads from `RenderTexture.active`. The standard "render-target-to-Texture2D" path. **[Community-known]** It's a synchronous GPU→CPU stall. If you only need the pixel data in Udon, `VRCAsyncGPUReadback` (SDK 3.2.0+) is the officially documented, non-blocking option. See [Render Textures](render-textures.md#reading-back-to-the-cpu-vrcasyncgpureadback). **[Docs]**

## What is *not* available

| Call | Status |
|---|---|
| `Texture2D.LoadImage(byte[])` | **Not** in Udon. Use `VRCImageDownloader` for runtime image loading from data. **[Community-known]** |
| `Texture2D.EncodeToPNG/JPG` | **Not** available — there's no I/O surface to write to in Udon. |
| `Texture2D.LoadRawTextureData` | **Unverified** — community evidence is mixed. |
| `Texture2D.Compress` | **Unverified** — likely blocked. |

## Assigning to a material

```csharp
material.mainTexture = tex;
material.SetTexture("_MyMap", tex);
```

Both work.

## Patterns

### Procedural pattern

```csharp
public MeshRenderer target;
Texture2D _tex;

void Start() {
    _tex = new Texture2D(128, 128, TextureFormat.RGBA32, false);
    Color[] pixels = new Color[128 * 128];
    for (int y = 0; y < 128; y++)
    for (int x = 0; x < 128; x++)
        pixels[y * 128 + x] = ((x ^ y) & 1) == 0 ? Color.white : Color.black;
    _tex.SetPixels(pixels);
    _tex.Apply();
    target.material.mainTexture = _tex;
}
```

### Render-to-texture readback

To capture from a `RenderTexture` into a `Texture2D`:

```csharp
public RenderTexture rt;
Texture2D _tex;

void Capture() {
    var prev = RenderTexture.active;
    RenderTexture.active = rt;
    if (_tex == null) _tex = new Texture2D(rt.width, rt.height, TextureFormat.RGBA32, false);
    _tex.ReadPixels(new Rect(0, 0, rt.width, rt.height), 0, 0);
    _tex.Apply();
    RenderTexture.active = prev;
}
```

`RenderTexture.active` getter/setter is **[Community-known]**.

## Performance notes

- `Apply()` is expensive (full GPU upload). Don't call it every frame on a 2048×2048 texture.
- `GetPixels`/`SetPixels` allocate the full pixel array. Prefer `Color32` versions for 4-byte pixels.
- For real-time procedural visuals, use `VRCGraphics.Blit` with a shader writing into a `RenderTexture` — it's almost always faster than CPU-side `SetPixels`.

## See also

- [Image Downloader](image-downloader.md) — the official path for external textures.
- [Render Textures](render-textures.md)
- [VRCGraphics](vrc-graphics.md) — `Blit` for shader-driven texture work.
