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

**Store `downloader` in a field** — the docs call this *required*: otherwise the downloader (and every texture it produced) can be garbage-collected at a random time. The same instance can issue many downloads. Dispose individual downloads with `IVRCImageDownload.Dispose()` (SDK 3.8.0+), or the whole downloader with `VRCImageDownloader.Dispose()` when you're permanently done. **[Docs]**

The SDK also ships a ready-made `ImageDownload` Udon program (add an UdonBehaviour, pick `ImageDownload` as program source, assign a Material). **[Docs]**

## `DownloadImage` overloads

```csharp
IVRCImageDownload DownloadImage(
    VRCUrl url,
    Material targetMaterial,        // optional; if non-null, downloaded image becomes its main texture
    IUdonEventReceiver receiver,    // optional; recipient of OnImageLoadSuccess/Error callbacks
    TextureInfo info);              // optional; texture parameters
```

The docs list `Material`, `UdonBehaviour` and `TextureInfo` as optional parameters; the signature shown is the full one. Parameter names above are descriptive, not verbatim. **[Docs]**

- **UdonSharp receives no events unless you pass a receiver.** (In Udon Graph an empty receiver defaults to the current behaviour; U# has no such default.) **[Docs]**
- Calling `DownloadImage` from U# with **no** receiver threw an exception before SDK 3.8.2; fixed in 3.8.2. **[Docs]** <https://creators.vrchat.com/releases/release-3-8-2/>

`IUdonEventReceiver` is satisfied by any `UdonSharpBehaviour` (cast as needed). Most U# code passes `(VRC.Udon.Common.Interfaces.IUdonEventReceiver)this`.

## `TextureInfo`

Optional configuration for the downloaded texture. Assign fields one by one (U# object-initializer support is not something to rely on here):

```csharp
TextureInfo info = new TextureInfo();
info.MaterialProperty = "_MainTex";
info.GenerateMipMaps = true;
info.WrapModeU = TextureWrapMode.Clamp;
info.WrapModeV = TextureWrapMode.Clamp;
info.FilterMode = FilterMode.Bilinear;
info.AnisoLevel = 9;
```

| Field | Default | Notes |
|---|---|---|
| `GenerateMipMaps` | `false` | |
| `FilterMode` | `Trilinear` | |
| `WrapModeU` / `WrapModeV` / `WrapModeW` | `Repeat` | `W` only matters for 3D textures. |
| `AnisoLevel` | `9` | VRChat forces anisotropic filtering: 1–9 becomes 9; >9 is clamped to 9–16; 0 disables. |
| `MaterialProperty` | `_MainTex` | Which texture slot on `targetMaterial` receives the result. |

**[Docs]**

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

Returned by `DownloadImage` and passed to both callbacks. Many fields are invalid until the download completes or fails. **[Docs]**

- `Texture2D Result` — the loaded texture.
- `VRCImageDownloadError Error` — `InvalidURL`, `AccessDenied`, `InvalidImage`, `DownloadError`, `Unknown`.
- `string ErrorMessage` — human-readable detail.
- `VRCUrl Url` — the URL that was requested.
- `VRCImageDownloadState State` — `Pending`, `Error`, `Complete`, `Unloaded` (after `Dispose()`), `Unknown`.
- `float Progress` — 0..1, for custom loading bars.
- `int SizeInMemoryBytes` — memory footprint of the result.
- `Material Material`, `TextureInfo TextureInfo`, `UdonBehaviour` — echo the arguments passed to `DownloadImage`.
- `void Dispose()` — **SDK 3.8.0+**; see [Dispose](#dispose).

Member names above follow the docs' graph-node names (`Get Errormessage`, `Get URL`, …) mapped to C# casing; double-check exact casing (e.g. `Url` vs `URL`) in your IDE. **[Unverified]** for casing.

## Limits

Per <https://creators.vrchat.com/worlds/udon/image-loading/>: **[Docs]**

| Limit | Value |
|---|---|
| Max image resolution | 2048 × 2048 |
| Max buffer (input + output) | 32 MB |
| Queue capacity | 1000 items |
| Rate limit | 1 image every 5 seconds, **for the whole scene** (not per downloader). Excess requests are queued and downloaded in **random order**. |
| Redirects | Not allowed — the URL must point directly at the image file. |

## Trusted hosts

Default allowlist: **[Docs]**

| Service | Domain(s) |
|---|---|
| DisBridge | `*.disbridge.com` |
| Dropbox | `dl.dropbox.com`, `dl.dropboxusercontent.com` |
| GitHub | `*.github.io` |
| ImageBam | `images4.imagebam.com` |
| ImgBB | `i.ibb.co` |
| imgbox | `images2.imgbox.com` |
| Imgur | `i.imgur.com` |
| Postimages | `i.postimg.cc` |
| Reddit | `i.redd.it` |
| Twitter | `pbs.twimg.com` |
| VRCDN | `*.vrcdn.cloud` |
| VRChat | `assets.vrchat.com` |
| Ytimg | `i.ytimg.com` |

Images from any other domain **do not download** unless the user has enabled **Allow Untrusted URLs** in their own settings — there is no per-image prompt. **[Docs]** (The per-world "allowed domains" list on the VRChat website applies to video players, not images — see [Video](../video/README.md#untrusted-urls-in-public-instances-sdk-375).)

## Formats

PNG and JPEG are the formats in common use. The docs do not list supported file formats. **[Community-known]**

VRChat picks the `TextureFormat` automatically: **[Docs]**

- Images with alpha → RGBA32, RGB64, etc.
- Images without alpha → RGB24, RGB48, etc.
- Greyscale → R8, R16, etc. (a greyscale image *with* alpha is interpreted as RG).

## `Dispose`

Two levels of disposal: **[Docs]**

```csharp
download.Dispose();     // IVRCImageDownload — SDK 3.8.0+: frees just this texture; State becomes Unloaded
downloader.Dispose();   // VRCImageDownloader — frees ALL its downloads + textures and invalidates the downloader
```

- `IVRCImageDownload.Dispose()` was allowlisted in **SDK 3.8.0**. It leaves other downloads intact — the intended tool for rotating carousels: dispose the old image when its replacement arrives. <https://creators.vrchat.com/releases/release-3-8-0/>
- `VRCImageDownloader.Dispose()` invalidates the downloader: it can't download new images afterwards.
- Failing to dispose downloads you no longer need grows memory over time until visitors **crash**. Don't `new VRCImageDownloader()` per call — reuse one.

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

### Rotating carousel (SDK 3.8.0+)

```csharp
VRCImageDownloader _dl;
IVRCImageDownload _current;
public Material frame;

void Start() { _dl = new VRCImageDownloader(); }

public void Show(VRCUrl url) {
    _dl.DownloadImage(url, null, this);
}

public override void OnImageLoadSuccess(IVRCImageDownload result) {
    frame.mainTexture = result.Result;
    if (_current != null) _current.Dispose();   // free the previous image only
    _current = result;
}
```

Free the old download only after the new one has arrived, so the frame never shows a destroyed texture.

## Community projects using `VRCImageDownloader`

- **Official `examples-image-loading`** — the canonical slideshow + image-array reference. Reuses a single downloader for the lifetime of the world. **[Docs]** <https://github.com/vrchat-community/examples-image-loading>
- **PachyPictureFrame** (kdingo). **[Community]** <https://github.com/kdingo/PachyPictureFrame>
- **DeacsImageLoader** (DeaconDP). **[Community]** <https://github.com/DeaconDP/DeacsImageLoader>
- **Discord2VRC** (acertainbluecat) — redirector pattern for Discord-hosted images whose signed URLs expire. Note that image loading does not follow redirects (see the table above), so verify it still works before relying on it. **[Community]** <https://github.com/acertainbluecat/Discord2VRC>

## Common pitfalls (community)

- **Crash via leaked textures.** Forgetting to `Dispose()` downloads you've replaced accumulates memory for the world's lifetime; in a busy gallery world, **visitors will crash**. Reuse one downloader and dispose replaced downloads individually (SDK 3.8.0+). **[Docs]**
- **Don't construct a new `VRCImageDownloader` per call** — same leak path.
- **Not storing the downloader in a field.** A downloader held only in a local can be garbage-collected, taking its textures with it. **[Docs]**
- **Pass `Material` for auto-apply.** The `DownloadImage(url, material, receiver)` overload auto-assigns to `_MainTex` (or the `TextureInfo.MaterialProperty` you specify) on success. Saves you the manual `material.mainTexture = result.Result` boilerplate.

## See also

- [Loading → trusted URLs](../loading/trusted-urls.md)
- [Textures](textures.md)
- [UdonSharpBehaviour → video events](../udonsharp-behaviour/video-events.md) — the equivalent callback pattern for video.
- [References → Community Resources](../references/community-resources.md)
