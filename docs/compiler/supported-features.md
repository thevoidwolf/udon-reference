# Supported Features

The C# subset and Unity API surface that the U# compiler accepts. The official list is the "C# features supported" section of <https://udonsharp.docs.vrchat.com/> **[Docs]**; items tagged **[Source]** were checked against the compiler bundled in SDK 3.10.3 (`Packages/com.vrchat.worlds/Integrations/UdonSharp/Editor/Compiler/`). Anything the binder has no handler for fails with "UdonSharp does not currently support node type '…'".

## Language features

### Types
- `bool`, `byte`, `sbyte`, `char`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `float`, `double`, `decimal`.
- `string`.
- Single-dimensional arrays of any supported element type. Jagged arrays of supported types. **[Docs]** Arrays are the only collection type. **[Docs]** **[Community-known]** the compiler may struggle with deeply nested jagged arrays — keep them shallow. Note `GetType()` on a jagged array reports `object[]`. **[Docs]**
- User-defined `enum`s, including `switch` over them. **[Code-confirmed]** `Tests~/TestScripts/FlowControl/SwitchTest.cs`. An enum miscompile bug affecting enums with a specific member count was fixed in **SDK 3.10.4**. **[Docs]**
- Unity value types: `Vector2`, `Vector3`, `Vector4`, `Quaternion`, `Color`, `Color32`, `Bounds`, `Rect`, `Matrix4x4`, `Plane`, `Ray`, `RaycastHit`, `Touch`.
- `System.DateTime`, `System.TimeSpan`.

### Control flow
- `if`/`else`, `while`, `do`/`while`, `for`, `foreach` over arrays **[Docs]**; also over a `Transform` (its children) and a `string` (its chars) **[Source]**.
- `switch`/`case` statements with constant labels (not `goto case` — no binder support for any `goto`). **[Source]**
- `break`, `continue`, `return`.
- `?:` ternary, `??` null-coalescing, `&&`/`||` short-circuiting. **[Docs]** / **[Source]**

### Methods
- Static and instance methods, with parameters and return values. **[Docs]**
- **`out` / `ref` parameters on user-defined methods**, and calling extern methods with `out`/`ref` (e.g. `Quaternion.ToAngleAxis(out a, out b)`). **[Docs]**
- Extension methods. **[Docs]**
- Generic **static** methods (including generic extension methods) declared in a non-`UdonSharpBehaviour` static class, e.g. `static T[] AddElement<T>(this T[] a, T item)`. **[Code-confirmed]** `Tests~/TestScripts/Core/GenericsTest.cs`. Generic methods *on* a `UdonSharpBehaviour` are rejected. **[Source]**
- Expression-bodied members (`int Count() => 4;`). **[Code-confirmed]**
- Recursion via `[RecursiveMethod]`. **[Docs]**
- Up to ~15 parameters (compiler-internal limit). **[Unverified]**
- Default parameter values.
- Method overloading by parameter list.
- `params` arrays. **[Docs]**
- Optional / named arguments. **[Source]**

### Operators
- All arithmetic, comparison, logical, bitwise.
- Compound assignment (`+=`, `*=`, etc.).
- Implicit and explicit conversions; numeric casts are **overflow-checked** due to Udon VM limitations. **[Docs]**
- String interpolation `$"{x}"`. **[Docs]**
- `typeof(T)` for Unity/.NET/VRChat (extern) types — **not** for your own types ("Cannot use typeof on user-defined types"). **[Source]**
- `nameof(...)`. **[Code-confirmed]**
- **Not** supported despite older claims: `is` / `as` type tests and the `?.` null-conditional operator — the bundled compiler has no binder support for them. Use `!= null` / `Utilities.IsValid` and [Type identity](../udonsharp-behaviour/type-identity.md) instead. **[Source]**

### Classes & inheritance
- Public/private/protected fields and methods.
- User-defined properties, including auto-properties. **[Docs]**
- Static **methods**. **[Docs]** Static **fields** are rejected ("Static fields are not yet supported on user defined types") — this includes `static readonly`. **[Source]**
- `const` fields and locals. **[Source]**
- Field initializers (evaluated at compile time). **[Docs]**
- Inheritance through `UdonSharpBehaviour` subclasses only (you cannot derive from `MonoBehaviour` directly). Abstract base behaviours are fine but cannot have their own program asset. **[Docs]** / **[Source]**
- `virtual` / `override` methods. **[Docs]** A derived class may not redeclare (hide) a base field. **[Source]**
- Referencing other `UdonSharpBehaviour` types, accessing their fields and calling their methods. **[Docs]**
- The class name must match its `.cs` file name. **[Source]**

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

- `Behaviour.enabled` and `Behaviour.isActiveAndEnabled` on anything deriving from `UnityEngine.Behaviour` (most built-in and custom components) — **SDK 3.10.2+**. Known limitation: does not work when the field is typed as `Behaviour` and the target is a `UdonSharpBehaviour`. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-2/>

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
- `UdonSharpBehaviour` implements `IUdonEventReceiver` (**SDK 3.10.4+**), so it can be passed where the SDK expects an event receiver without casting. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-4/>

## Preprocessor

- `COMPILER_UDONSHARP` is defined while U# compiles a script — use `#if !COMPILER_UDONSHARP` to hide editor-only code in the same file. **[Docs]** <https://udonsharp.docs.vrchat.com/editor-scripting>
- Assembly Version Defines from the script's asmdef, e.g. `#if SDK_310_OR_NEWER` — **SDK 3.10.5+**. **[Docs]** See [Assembly Definition](../editor-api/assembly-definition.md#assembly-version-defines-sdk-3105).

## See also

- [Unsupported features](unsupported-features.md) — the inverse list.
- [Workarounds](workarounds.md) — what to substitute.
- [Attributes](../attributes/README.md) — U#-specific attributes.
