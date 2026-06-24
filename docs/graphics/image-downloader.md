# VRCImageDownloader

The officially supported way to load images at runtime. Source: <https://creators.vrchat.com/worlds/udon/image-loading/>. **[Docs]**

Namespace: `VRC.SDK3.Image`.

## Construction and basic usage

```csharp
using VRC.SDK3.Image;
using VRC.Udon.Common.Interfaces;
using VRC.SDKBase;

VRCImageDownloader downloader = new VRCImageDownloader();
IVRCImageDownload handle = downloader.DownloadImage(url, targetMaterial, this);
```

Hold on to `downloader` (the same instance can issue multiple downloads) and call `Dispose()` when you're permanently done.

## `DownloadImage` overloads

```csharp
IVRCImageDownload DownloadImage(
    VRCUrl url,
    Material targetMaterial,        // optional; if non-null, auto-applied to material
    IUdonEventReceiver receiver,    // recipient of OnImageLoadSuccess/Error callbacks
    TextureInfo info);              // optional; texture parameters
```

There are simpler overloads with fewer parameters; the signature shown is the full one.

`IUdonEventReceiver` is satisfied by any `UdonSharpBehaviour` (cast as needed). Most U# code passes `(VRC.Udon.Common.Interfaces.IUdonEventReceiver)this`.

## `TextureInfo`

Optional configuration for the downloaded texture:

```csharp
new TextureInfo {
    MaterialProperty = "_MainTex",
    GenerateMipMaps = true,
    WrapModeU = TextureWrapMode.Clamp,
    WrapModeV = TextureWrapMode.Clamp,
    FilterMode = FilterMode.Bilinear,
    AnisoLevel = 9,
};
```

`MaterialProperty` controls which texture slot on `targetMaterial` receives the result (default `_MainTex`).

## Callbacks

Override on the receiver:

```csharp
public override void OnImageLoadSuccess(IVRCImageDownload result) {
    Texture2D tex = result.Result;
    // tex is also already assigned to targetMaterial if you passed one
}

public override void OnImageLoadError(IVRCImageDownload result) {
    Debug.LogError($"image load failed: {result.Error}: {result.ErrorMessage}");
}
```

## `IVRCImageDownload` shape

- `Texture2D Result` — the loaded texture.
- `VRCImageDownloadError Error` — error enum.
- `string ErrorMessage` — human-readable detail.
- `VRCUrl Url` — the URL that was requested.
- `VRCImageDownloadState State` — `Pending` / `Complete` / `Error`.
- `int SizeInMemoryBytes` — memory footprint of the result.

Exact field names may vary slightly between SDK versions. **[Unverified]**

## Limits

Per <https://creators.vrchat.com/worlds/udon/image-loading/>: **[Docs]**

| Limit | Value |
|---|---|
| Max image resolution | 2048 × 2048 |
| Max buffer (input + output) | 32 MB |
| Queue capacity | 1000 items |
| Rate limit | 1 image every 5 seconds |

## Trusted hosts

Default allowlist (no user prompt): DisBridge, Dropbox, GitHub, ImageBam, ImgBB, imgbox, Imgur, Postimages, Reddit, Twitter, VRCDN, VRChat, Ytimg.

Other hosts trigger an "Allow Untrusted URLs" prompt unless the user has enabled untrusted URLs globally in their VRChat settings.

## Formats

The runtime decodes:
- PNG (with or without alpha)
- JPEG
- The result is interpreted as RGBA, RGB, or RG depending on source.

```csharp
// Image with alpha: tex is RGBA32
// Opaque image:     tex is RGB24
```

## `Dispose`

```csharp
downloader.Dispose();
```

Frees the textures the downloader created. **Critical:** every `Texture2D` produced by a downloader lives until the downloader is disposed (or until world unload). Don't `new VRCImageDownloader()` per call — reuse one and dispose when done.

## Patterns

### Load → assign → forget

```csharp
public VRCUrl url;
public Renderer target;

VRCImageDownloader _dl;

void Start() {
    _dl = new VRCImageDownloader();
    _dl.DownloadImage(url, target.material, this);
}

public override void OnImageLoadError(IVRCImageDownload r) {
    Debug.LogError($"Image load failed: {r.ErrorMessage}");
}

void OnDestroy() {
    if (_dl != null) _dl.Dispose();
}
```

### Multiple images, shared downloader

```csharp
VRCImageDownloader _dl;
void Start() { _dl = new VRCImageDownloader(); }

public void LoadOne(VRCUrl url, Material into) {
    _dl.DownloadImage(url, into, this);
}
```

One downloader manages many downloads.

## Community projects using `VRCImageDownloader`

- **Official `examples-image-loading`** — the canonical slideshow + image-array reference. Reuses a single downloader for the lifetime of the world. **[Docs]** <https://github.com/vrchat-community/examples-image-loading>
- **PachyPictureFrame** (kdingo). **[Community]** <https://github.com/kdingo/PachyPictureFrame>
- **DeacsImageLoader** (DeaconDP). **[Community]** <https://github.com/DeaconDP/DeacsImageLoader>
- **Discord2VRC** (acertainbluecat) — redirector pattern for Discord-hosted images whose signed URLs expire. **[Community]** <https://github.com/acertainbluecat/Discord2VRC>

## Common pitfalls (community)

- **Crash via leaked textures.** Forgetting to `Dispose()` a downloader (or its produced textures) accumulates GPU memory for the world's lifetime. With a busy gallery world, **visitors will crash**. Reuse one downloader; dispose at world unload. **[Community]**
- **Don't construct a new `VRCImageDownloader` per call** — same leak path.
- **Pass `Material` for auto-apply.** The `DownloadImage(url, material, receiver)` overload auto-assigns to `_MainTex` (or the `TextureInfo.MaterialProperty` you specify) on success. Saves you the manual `material.mainTexture = result.Result` boilerplate.

## See also

- [Loading → trusted URLs](../loading/trusted-urls.md)
- [Textures](textures.md)
- [UdonSharpBehaviour → video events](../udonsharp-behaviour/video-events.md) — the equivalent callback pattern for video.
- [References → Community Resources](../references/community-resources.md)
