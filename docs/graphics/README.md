# Graphics

Runtime rendering APIs available to Udon worlds. VRChat does not publish a complete extern allowlist, so every page in this section flags claims with provenance tags (see [main README](../../README.md#provenance-tags)).

## Pages

| Page | Topic |
|---|---|
| [VRCGraphics](vrc-graphics.md) | `Blit`, `DrawMeshInstanced`. |
| [VRCShader](vrc-shader.md) | Global shader uniforms, `PropertyToID`, the `_Udon` prefix rule. |
| [Materials](materials.md) | Mutating material properties, `Renderer.material` vs `sharedMaterial`. |
| [Textures](textures.md) | Texture2D creation and mutation, the official `VRCImageDownloader` path. |
| [Meshes](meshes.md) | Procedural mesh creation, vertices / triangles / UVs. |
| [Render Textures](render-textures.md) | Authoring-time vs runtime RenderTexture usage, `VRCAsyncGPUReadback`. |
| [Image Downloader](image-downloader.md) | `VRCImageDownloader` — official runtime texture loading, per-download `Dispose()`. |
| [Camera & Quality Settings](camera-and-quality-settings.md) | `VRCCameraSettings` (screen / photo camera), `VRCQualitySettings`, Camera Dolly (SDK 3.8.1+ / 3.9.0+). |

## Authoring rule of thumb

| You want | Use |
|---|---|
| Change a color or shader value | [Materials](materials.md) |
| Push a value to *all* shaders in the world | [VRCShader](vrc-shader.md) (global uniforms) |
| Render a quad with a shader once per frame | `VRCGraphics.Blit` |
| Render thousands of identical meshes cheaply | `VRCGraphics.DrawMeshInstanced` |
| Load an image from the internet | `VRCImageDownloader.DownloadImage` |
| Build a mesh from procedurally-generated vertices | `new Mesh()`, write to `meshFilter.mesh` — see [Meshes](meshes.md) |
| Make a "camera that renders to a texture" | Author a `RenderTexture` + `Camera` in the scene; reference both at edit time |
| Read GPU pixels back into Udon | `VRCAsyncGPUReadback.Request` — see [Render Textures](render-textures.md#reading-back-to-the-cpu-vrcasyncgpureadback) |
| Read / tweak the player's own camera (clip planes, culling mask, FOV) | `VRCCameraSettings.ScreenCamera` — see [Camera & Quality Settings](camera-and-quality-settings.md) |
| Override shadow distance / cascades | `VRCQualitySettings` — see [Camera & Quality Settings](camera-and-quality-settings.md#vrcqualitysettings-sdk-381) |

## Quest gotchas

Many "PCVR-only" graphics features fall over on Quest in non-obvious ways:

- `VRCGraphics.Blit` requires `ZTest Always` in the shader, *or* the destination RenderTexture must have depth disabled, otherwise the call silently does nothing on Quest. **[Docs]**
- Procedural meshes with thousands of vertices stress the mobile GPU; budget tightly.
- `VRCImageDownloader` works on Quest; the 2048×2048 cap applies on every platform. **[Docs]**
- `VRCQualitySettings.SetShadowDistance`'s `mobile` argument applies to all non-PC platforms regardless of the user's settings. **[Docs]**

Always test on Quest before shipping graphics-heavy features.

## Mirrors

VRChat's built-in `VRCMirrorReflection` component handles per-eye rendering, layer culling, and Quest constraints. Key facts:

- **Mirrors can drastically reduce framerate** — each active mirror re-renders the scene (per eye in VR). Official advice: keep mirrors off by default, don't reflect every layer, reduce resolution if needed. **[Docs]** <https://creators.vrchat.com/worlds/components/vrc_mirrorreflection/>
- **Occlusion culling is used in mirrors by default**; the *Turn Off Mirror Occlusion* option disables it if objects flicker in the reflection. Objects on the **Water** layer are never rendered in mirrors. **[Docs]**
- **Rendering hook changed in SDK 3.10.3:** mirrors now render in `Camera.onPreCull` instead of `OnWillRenderObject` (matching client 2026.1.3). Custom mirror-aware scripts/shaders that relied on the old ordering should be retested. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-3/>
- **SDK 3.10.2:** `VRCMirrorReflection` instantiates its own non-editable material instead of modifying the original — it no longer alters Unity's built-in default material when added to a primitive. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-2/>
- **Default mirrors OFF.** Toggle via Udon (an interactable button) or by `OnPlayerTriggerEnter` proximity volume. **[Community]** <https://wiki.vrchat.com/wiki/Mirrors>
- **Use the 32-bit `m_ReflectLayers` LayerMask** to skip environment / post-processing layers in the reflection pass. Settable from Udon at runtime to swap between "selfie-quality" and "low-perf" modes. **[Community]** <https://www.ucodelibrary.com/concepts/vrc-custom-mirrors>
- **Resolution.** *Mirror Resolution* is per eye in VR; *Auto* matches the user's HMD/monitor up to a 2048×2048 maximum, so displays above 2048px see reduced mirror quality unless the user picks *Unlimited* in VRChat's settings (costs performance). Lower it if users still have perf issues. **[Docs]**
- **Mirror-aware shaders** need `_VRChatMirrorMode` and cnlohr's `GetLinearZFromZDepth_WorksWithMirrors()` helper (standard `LinearEyeDepth` fails under oblique projection). **[Community]** <https://github.com/cnlohr/shadertrixx>
- **Players-only no-background mirror** — `acertainbluecat/VRCPlayersOnlyMirror`. Common dance-world choice. **[Community]** <https://github.com/acertainbluecat/VRCPlayersOnlyMirror>

## Community-recommended shaders

- **Poiyomi Toon** — most popular feature-rich toon shader. **[Community]** <https://github.com/poiyomi/PoiyomiToonShader>
- **Mochie's Unity Shaders** — Standard, water, post-processing. **[Community]** <https://github.com/MochiesCode/Mochies-Unity-Shaders>
- **z3y / Okabintaro shaders** — Standard/Surface replacement; first to ship Quest box-projected AreaLit + LTCGI. **[Community]** <https://github.com/z3y/shaders>
- **Silent's Cel Shading** — lightweight, respects Unity lighting. **[Community]** <https://ask.vrchat.com/t/silents-cel-shading-shader/3234>

## See also

- [Camera & Quality Settings](camera-and-quality-settings.md) — `VRCCameraSettings`, `VRCQualitySettings`, Camera Dolly.

- [GameObject Lifecycle → Runtime constraints](../gameobject-lifecycle/runtime-constraints.md) — why you can't `AddComponent<MeshRenderer>` at runtime.
- [References → Community Resources](../references/community-resources.md) — full graphics / shaders index.
