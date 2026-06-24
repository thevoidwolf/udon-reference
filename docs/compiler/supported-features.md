# Supported Features

The C# subset and Unity API surface that the U# compiler accepts.

## Language features

### Types
- `bool`, `byte`, `sbyte`, `char`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `float`, `double`, `decimal`.
- `string`.
- Single-dimensional arrays of any supported element type. Jagged arrays of supported types. **[Community-known]** the compiler may struggle with deeply nested jagged arrays — keep them shallow.
- Unity value types: `Vector2`, `Vector3`, `Vector4`, `Quaternion`, `Color`, `Color32`, `Bounds`, `Rect`, `Matrix4x4`, `Plane`, `Ray`, `RaycastHit`, `Touch`.
- `System.DateTime`, `System.TimeSpan`.

### Control flow
- `if`/`else`, `while`, `do`/`while`, `for`, `foreach` (over arrays).
- `switch`/`case` with `goto case`.
- `break`, `continue`, `return`.
- `?:` ternary, `??` null-coalescing (for reference types).

### Methods
- Static and instance methods.
- Up to ~15 parameters (compiler-internal limit).
- Default parameter values.
- Method overloading by parameter list.
- `params` arrays.
- Optional / named arguments.

### Operators
- All arithmetic, comparison, logical, bitwise.
- Compound assignment (`+=`, `*=`, etc.).
- `is` and `as` for **builtin** type checks. User `UdonSharpBehaviour` subclass checks work via compiler magic — but with caveats; see [Type identity](../udonsharp-behaviour/type-identity.md).
- `?.` null-conditional access on `MonoBehaviour`-derived references and `string`.
- String interpolation `$"{x}"`.

### Classes & inheritance
- Public/private/protected fields and methods.
- Properties with backing fields, including auto-properties.
- Static fields and methods.
- `const` and `static readonly` (with constant initializers).
- Inheritance through `UdonSharpBehaviour` subclasses only (you cannot derive from `MonoBehaviour` directly).
- `virtual` / `override` work for lifecycle events.

### Lambdas / delegates
**Not supported.** No anonymous methods, no `Action<T>`, no `Func<...>`. Substitute with `SendCustomEvent` or direct method calls.

### Strings
- `string` concatenation, interpolation, `string.Format`.
- `string.Length`, `[i]` indexing, `Substring`, `Replace`, `Trim`, `ToUpper`, `ToLower`, `IndexOf`, `Contains`, `StartsWith`, `EndsWith`, `Split`, `Join`.

## Unity API surface (broad strokes)

Generally available:

- `Transform` — position/rotation/scale, parenting, hierarchy traversal.
- `GameObject` — `Find`, `FindWithTag`, `SetActive`, `tag`, `name`, `layer`.
- `Component` — `GetComponent<T>`, `GetComponentInChildren`, `GetComponents`, etc.
- `Mathf` — all helpers.
- `Random` — value, range, rotation.
- `Time` — `time`, `deltaTime`, `unscaledTime`, etc.
- `Physics` — `Raycast`, `OverlapSphere`, `CapsuleCast`, layer masks.
- `Animator` — `SetBool`, `SetFloat`, `SetInteger`, `SetTrigger`, `Play`, `speed`.
- `Animation` — basic playback.
- `Renderer`, `MeshFilter`, `MeshRenderer`, `SkinnedMeshRenderer` — read/write `material`/`mesh`/`sharedMaterial`/`sharedMesh`, bounds.
- `Material` — `SetColor`, `SetFloat`, `SetTexture`, `SetVector`, `color`, `mainTexture`. (See [Materials](../graphics/materials.md).)
- `Texture2D` — construction and pixel ops. **[Community-known]**
- `Mesh` — construction, vertices/triangles/uvs/normals/colors, recalc helpers. **[Community-known]**
- `AudioSource` — Play, PlayOneShot, volume, pitch, clip, etc.
- `Collider`, `Rigidbody` — full physics surface.
- `ParticleSystem` — Play, Stop, Emit.
- `Camera` — fieldOfView, transform, targetTexture (with caveats).
- UI: `Text`, `Image`, `RawImage`, `Button`, `Slider`, `Toggle`, `Dropdown`, `InputField`, `ScrollRect`, `Canvas`, `CanvasGroup`, `RectTransform`.

`Debug.Log`, `Debug.LogWarning`, `Debug.LogError` all work.

## VRChat SDK surface

- `VRC.SDKBase.Networking` static class.
- `VRC.SDKBase.VRCPlayerApi`.
- `VRC.SDK3.Components.VRCPickup` / `VRCStation` / `VRCObjectSync` / `VRCObjectPool` / `VRCEnablePersistence` / `VRCPlayerObject`.
- `VRC.SDK3.Image.VRCImageDownloader` and related types.
- `VRC.SDK3.StringLoading.VRCStringDownloader` and related types.
- `VRC.SDK3.Data.DataToken` / `DataList` / `DataDictionary` / `VRCJson`.
- `VRC.SDK3.Persistence.PlayerData`.
- `VRC.SDKBase.VRCUrl`.
- `VRC.SDK3.Video.Components.*` (video players).
- `VRC.SDKBase.VRCGraphics`, `VRC.SDKBase.VRCShader`.

## See also

- [Unsupported features](unsupported-features.md) — the inverse list.
- [Workarounds](workarounds.md) — what to substitute.
- [Attributes](../attributes/README.md) — U#-specific attributes.
