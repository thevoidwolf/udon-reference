# Materials

Mutating a `UnityEngine.Material`'s shader inputs at runtime works in Udon. The compiler ships a test scene that exercises this surface: `Tests~/MiscTests/EditorClientSetColor.cs:13-19`. **[Code-confirmed]**

```csharp
Material m = GetComponent<MeshRenderer>().material;
m.SetColor("_Color", Color.blue);
```

## Whitelisted setters

```csharp
material.SetColor(string name, Color value);
material.SetFloat(string name, float value);
material.SetInt(string name, int value);
material.SetVector(string name, Vector4 value);
material.SetMatrix(string name, Matrix4x4 value);
material.SetTexture(string name, Texture value);

material.color = ...;                  // shorthand for _Color
material.mainTexture = ...;             // shorthand for _MainTex
material.mainTextureOffset = ...;
material.mainTextureScale = ...;

material.EnableKeyword(string keyword);
material.DisableKeyword(string keyword);
material.renderQueue = ...;
```

**[Community-known]** as a complete list — the official docs do not enumerate it. The setters above are widely used in production worlds.

## `Renderer.material` vs `Renderer.sharedMaterial`

```csharp
Renderer r = GetComponent<Renderer>();

r.material        // a per-instance copy. First access *allocates*. Mutations affect only this Renderer.
r.sharedMaterial  // the asset reference. Mutations affect every Renderer using this material.
```

Practical consequences:

- If 200 objects share `sharedMaterial` and one of them reads `r.material`, that one detaches and now has its own copy. The other 199 still share.
- Use `material` when you need per-object visual state.
- Use `sharedMaterial` *to read* properties without forking. To write through `sharedMaterial`, do so only if you intend the change to propagate to every Renderer using it.
- Repeated `r.material` access returns the same allocated copy — cache the reference for clarity, not for performance.

## ID-based setters (faster)

```csharp
int colorId = Shader.PropertyToID("_Color");
material.SetColor(colorId, Color.red);
```

Cache the ID at `Start()`. The string-based setter has a name-to-ID lookup cost on every call.

Note: this is `UnityEngine.Shader.PropertyToID`, **not** `VRCShader.PropertyToID`. The Unity version is fine for per-material uniforms because it doesn't go through the global pipeline. The `_Udon` / `_AudioTexture` prefix rule only applies to `VRCShader.SetGlobal*`.

## Material construction

**Not officially documented as supported.** Community Udon code uses `new Material(otherMaterial)` for copy-construction extensively, and it works in practice. **[Community-known]**

Safer pattern: pre-author one material per visual variant and swap references rather than constructing materials at runtime.

```csharp
public Material[] palette;     // pre-placed
public Renderer target;

public void SetColor(int index) {
    target.material = palette[index];   // swap whole material
}
```

## What doesn't work

- `Material.CopyPropertiesFromMaterial` — not whitelisted. **[Unverified]**
- `Material.Lerp` — not whitelisted. **[Unverified]**
- Procedural material asset creation (e.g. from raw bytes) — blocked.

## Patterns

### Animate a color over time

```csharp
Material _m;
float _t;

void Start() { _m = GetComponent<Renderer>().material; }

void Update() {
    _t += Time.deltaTime;
    _m.color = Color.Lerp(Color.red, Color.blue, Mathf.PingPong(_t, 1f));
}
```

### Cache property IDs

```csharp
static readonly int kColorId = Shader.PropertyToID("_Color");
static readonly int kGlowId  = Shader.PropertyToID("_GlowAmount");

// ❌ U# does not allow `static readonly` in user code in all cases — verify in your version.
// Safer fallback: instance fields initialized in Start.
int _colorId;
void Start() { _colorId = Shader.PropertyToID("_Color"); }
```

### Material PropertyBlocks

`MaterialPropertyBlock` with `Renderer.SetPropertyBlock` / `GetPropertyBlock` is used from UdonSharp in community worlds. A VRChat bug report about MPB colour changes from U# shows that the API compiles and runs. **[Community]** <https://feedback.vrchat.com/open-beta/p/1354-the-color-change-using-materialpropertyblock-from-udonsharp-does-not-seem-t> No official page lists it, so check it in your SDK version before you build around it. **[Unverified]** for the full member set.

```csharp
MaterialPropertyBlock _mpb;
void Start() { _mpb = new MaterialPropertyBlock(); }

public void Tint(Renderer r, Color c) {
    r.GetPropertyBlock(_mpb);
    _mpb.SetColor("_Color", c);
    r.SetPropertyBlock(_mpb);   // per-renderer override, no material fork
}
```

If MPBs don't fit your case, use pre-authored material variants, or `VRCShader.SetGlobal*` with a `_UdonPerObjectIndex` uniform.

## See also

- [VRCShader](vrc-shader.md)
- [Textures](textures.md)
- [Meshes](meshes.md)
