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
| [Render Textures](render-textures.md) | Authoring-time vs runtime RenderTexture usage. |
| [Image Downloader](image-downloader.md) | `VRCImageDownloader` — official runtime texture loading. |

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

## Quest gotchas

Many "PCVR-only" graphics features fall over on Quest in non-obvious ways:

- `VRCGraphics.Blit` requires `ZTest Always` in the shader, *or* the destination RenderTexture must have depth disabled, otherwise the call silently does nothing on Quest. **[Docs]**
- Procedural meshes with thousands of vertices stress the mobile GPU; budget tightly.
- `VRCImageDownloader` works on Quest, but texture sizes still cap at 2048×2048. **[Docs]**

Always test on Quest before shipping graphics-heavy features.

## Mirrors

VRChat's built-in `VRCMirrorReflection` component handles per-eye rendering, layer culling, and Quest constraints. Key facts:

- **Reflection roughly doubles total render cost** because occlusion culling does not function inside mirrors — a mirror facing a wall still renders everything behind it. **[Docs]** <https://creators.vrchat.com/worlds/components/vrc_mirrorreflection/>
- **Default mirrors OFF.** Toggle via Udon (an interactable button) or by `OnPlayerTriggerEnter` proximity volume. **[Community]** <https://wiki.vrchat.com/wiki/Mirrors>
- **Use the 32-bit `m_ReflectLayers` LayerMask** to skip environment / post-processing layers in the reflection pass. Tradable from Udon at runtime to swap between "selfie-quality" and "low-perf" modes. **[Community]** <https://www.ucodelibrary.com/concepts/vrc-custom-mirrors>
- **Lower the resolution.** Mirror quality degrades visibly above 2048px on most headsets; below that, perf wins outweigh the visual hit.
- **Mirror-aware shaders** need `_VRChatMirrorMode` and cnlohr's `GetLinearZFromZDepth_WorksWithMirrors()` helper (standard `LinearEyeDepth` fails under oblique projection). **[Community]** <https://github.com/cnlohr/shadertrixx>
- **Players-only no-background mirror** — `acertainbluecat/VRCPlayersOnlyMirror`. Common dance-world choice. **[Community]** <https://github.com/acertainbluecat/VRCPlayersOnlyMirror>

## Community-recommended shaders

- **Poiyomi Toon** — most popular feature-rich toon shader. **[Community]** <https://github.com/poiyomi/PoiyomiToonShader>
- **Mochie's Unity Shaders** — Standard, water, post-processing. **[Community]** <https://github.com/MochiesCode/Mochies-Unity-Shaders>
- **z3y / Okabintaro shaders** — Standard/Surface replacement; first to ship Quest box-projected AreaLit + LTCGI. **[Community]** <https://github.com/z3y/shaders>
- **Silent's Cel Shading** — lightweight, respects Unity lighting. **[Community]** <https://ask.vrchat.com/t/silents-cel-shading-shader/3234>

## See also

- [GameObject Lifecycle → Runtime constraints](../gameobject-lifecycle/runtime-constraints.md) — why you can't `AddComponent<MeshRenderer>` at runtime.
- [References → Community Resources](../references/community-resources.md) — full graphics / shaders index.
