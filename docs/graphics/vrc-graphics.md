# VRCGraphics

Static class `VRC.SDKBase.VRCGraphics`. Wraps a tiny subset of `UnityEngine.Graphics`. Source: <https://creators.vrchat.com/worlds/udon/vrc-graphics/>. **[Docs]**

## Methods

### `Blit`

```csharp
void Blit(Texture source, RenderTexture dest, Material mat);
void Blit(Texture source, RenderTexture dest, Material mat, int pass);
```

Copies (and optionally transforms) `source` into `dest` using `mat`'s shader. The optional `pass` selects which shader pass to use; `-1` runs all passes.

**Rules:**
- **`dest` must not be `null`** — Udon does not allow blits without an explicit destination. (`Graphics.Blit(src, null, mat)` in standard Unity is therefore not equivalent here.) **[Docs]**
- On Quest, the shader must declare `ZTest Always` **or** the target RenderTexture must have its depth disabled, otherwise the operation silently fails. **[Docs]**

**Use cases:**
- Post-process a `RenderTexture` between frames.
- Implement a paint-by-shader effect: draw into a `RenderTexture` and use it as a material's main texture.
- Reproject / warp / blur images.

```csharp
public RenderTexture rt;
public Material blurMaterial;

void Update() {
    VRCGraphics.Blit(rt, rt, blurMaterial);
}
```

### `DrawMeshInstanced`

```csharp
void DrawMeshInstanced(Mesh mesh, int submeshIndex, Material material, Matrix4x4[] matrices, int count = -1);
```

GPU-instanced draw. The same `mesh` is rendered once per matrix in `matrices`. `count = -1` uses the full array.

**Rules:**
- Up to 1023 matrices per call (Unity's standard limit). For more, split into multiple calls.
- The `material`'s shader must be GPU-instancing-aware (`#pragma multi_compile_instancing`).
- Bypasses the scene graph: there's no GameObject, no shadow caster bookkeeping by default. Set `material.enableInstancing = true` in the inspector.

**Use cases:**
- Grass / foliage.
- Bullet trails / particles with custom shaders.
- Debris and crowd visuals.

```csharp
public Mesh grassMesh;
public Material grassMaterial;
Matrix4x4[] _matrices;

void Start() {
    _matrices = new Matrix4x4[1023];
    for (int i = 0; i < _matrices.Length; i++) {
        Vector3 pos = new Vector3(Random.Range(-50f, 50f), 0, Random.Range(-50f, 50f));
        _matrices[i] = Matrix4x4.TRS(pos, Quaternion.identity, Vector3.one);
    }
}

void Update() {
    VRCGraphics.DrawMeshInstanced(grassMesh, 0, grassMaterial, _matrices);
}
```

## What's *not* on `VRCGraphics`

- `Graphics.DrawMesh` (non-instanced) — not exposed.
- `Graphics.DrawProcedural*` — not exposed.
- `Graphics.SetRenderTarget` — not exposed.
- `Graphics.CopyTexture` — not exposed.
- `Graphics.SetRandomWriteTarget` — not exposed; tracked feature request still open. **[Community]** <https://feedback.vrchat.com/udon/p/expose-graphicssetrandomwritetarget>
- **Compute shader dispatch is not available from Udon.** VRChat internally ships a compute shader for SDK-side DPID mipmap downscaling, but world authors cannot invoke `ComputeShader.Dispatch` at runtime. The standard workaround is the "fake compute" pattern (camera → quad → RT → re-sample next frame). **[Community]** <https://github.com/cnlohr/shadertrixx>

For these you usually substitute `Blit` + a custom shader, or attach a Camera with `targetTexture` at edit time.

## The "fake compute" pattern

The canonical persistent-state-without-compute pattern from cnlohr's notes:

1. Set up a quad with a material that runs an off-screen fragment shader.
2. Point a `Camera` at it whose `targetTexture` is your working RT.
3. The fragment shader reads the previous-frame RT and writes the next-frame RT.
4. Any per-pixel state you want is now persistent across frames.

`MerlinVR/Blit-Component` measured ~10× CPU speedup over a `Camera`-based approach (0.3 ms → 0.03 ms). It's a community-proposed alternative — not merged into the SDK. **[Community]** <https://github.com/MerlinVR/Blit-Component>

cnlohr's reference notebook on every VRChat-shader trick worth knowing: **[Community]** <https://github.com/cnlohr/shadertrixx>

## See also

- [VRCShader](vrc-shader.md)
- [Materials](materials.md)
- [Render Textures](render-textures.md)
- [References → Community Resources](../references/community-resources.md) — AudioLink, LTCGI, shader projects.
