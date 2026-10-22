# Unsupported Features

C# and Unity features the UdonSharp compiler rejects. Most of these have a [workaround](workarounds.md). Quoted error messages come from the compiler bundled in SDK 3.10.3 (`Integrations/UdonSharp/Editor/`). **[Source]** Syntax the binder has no handler for fails with "UdonSharp does not currently support node type '…'".

## Language features

### Generics in user code
- **No generic methods on behaviours.** `T DoThing<T>(T x)` declared in a `UdonSharpBehaviour` is rejected: "Generic method declarations on UdonSharpBehaviours are not currently supported, consider using a non-UdonSharpBehaviour class". Generic *static* helpers in a separate static class do compile — see [Supported features](supported-features.md#methods). **[Source]** / **[Code-confirmed]**
- **No generic behaviours.** `class Bag<T> : UdonSharpBehaviour` cannot have a program asset ("Generic U# behaviours cannot have an associated U# program asset"). **[Source]**
- **Most generic standard-library types are blocked:** `List<T>`, `Dictionary<TKey, TValue>`, `HashSet<T>`, `Queue<T>`, `Stack<T>`, `IEnumerable<T>`, `IEnumerator<T>`, `Lazy<T>`, `Nullable<T>`.

A few specific generics on Unity API are allowed because they have non-generic alternatives the compiler can map (e.g. `GetComponent<T>()`).

### Async, concurrency, threading
- `async` / `await` keywords.
- `Task`, `Task<T>`, `ValueTask`.
- `Thread`, `Mutex`, `Semaphore`, `lock` statement.
- Job System, Burst.
- Coroutines (`IEnumerator`-yielding methods registered with `StartCoroutine`).

### LINQ
- No `using System.Linq;` extensions: `Where`, `Select`, `OrderBy`, `Aggregate`, `Sum`, `Count`, etc.

### Reflection
- `System.Reflection.*` — `Type.GetField`, `Activator.CreateInstance`, `MethodInfo.Invoke`, etc.
- `typeof(MyClass)` works at compile time only — the compiler replaces it with a constant.

### Unsafe / native
- `unsafe` blocks.
- Pointer types (`int*`).
- `stackalloc`.
- `Span<T>`, `Memory<T>`, `ReadOnlySpan<T>`.
- `ref struct`, `ref` returns.
- `DllImport`.

### Patterns / modern C#
- `switch` patterns (`x switch { ... }`).
- Pattern matching with `is T t`, `is null`, property patterns.
- Plain `is` / `as` type tests (`x is Foo`, `x as Foo`) — no binder support. **[Source]**
- `?.` null-conditional access (`a?.b`). **[Source]**
- Object and collection initializers (`new Foo { X = 1 }`) — "U# Does not yet support initializer lists". Array initializers (`int[] a = { 1, 2 };`) are fine. **[Source]**
- Tuples (`(int, string)`, `ValueTuple`).
- Record types.
- `init` accessors.
- `with` expressions.
- Top-level statements.
- Local functions — "Local method declarations are not currently supported by UdonSharp". **[Source]**
- Nested type declarations (a `class`/`enum` declared inside your behaviour) — "Nested type declarations are not currently supported by U#". Declare enums at namespace level. **[Source]**
- Partial methods. **[Source]**
- Range / index operators (`a[1..^1]`).

### Lambdas / delegates
- No anonymous methods or lambda expressions.
- No `Action`, `Func`, `Predicate` types.
- No `event` declarations on `UdonSharpBehaviour`s.

### Other
- **Static fields** on your own types (incl. `static readonly`) — "Static fields are not yet supported on user defined types". `const` is fine. **[Source]**
- Hiding a base-class field in a derived behaviour — "U# does not yet support hiding base fields". **[Source]**
- `typeof(MyBehaviour)` on user-defined types — "Cannot use typeof on user-defined types". **[Source]**
- Custom indexers (`this[int i]`).
- Operator overloading on user types.
- Conversion operators (implicit/explicit) on user types.
- Static constructors.
- `try`/`catch`/`finally` — "Try/Catch/Finally is not supported by UdonSharp since Udon does not have a way to handle exceptions". **No exception handling at all.** **[Source]**
- `throw` statements and expressions. **[Source]**
- **Any** constructor on `UdonSharpBehaviour` subclasses — "Constructors are not currently supported on UdonSharpBehaviours". **[Source]**
- `goto` in any form, including `goto case`. **[Source]**
- `checked` / `unchecked` blocks. Note numeric *casts* are always overflow-checked by U# due to Udon VM limitations. **[Docs]**

(`ref` / `out` parameters on user methods **are** supported — the official feature list includes them. **[Docs]**)

## Unity / .NET API surface

### I/O
- All `System.IO.*`.
- `PlayerPrefs.*` — use [`PlayerData`](../persistence/player-data.md).
- `Application.persistentDataPath`, `Application.dataPath`.

### Networking (non-VRChat)
- `WWW`, `UnityWebRequest`, raw sockets, `HttpClient`.
- Use [`VRCStringDownloader`](../loading/string-loading.md) / [`VRCImageDownloader`](../graphics/image-downloader.md) instead.

### Resources
- `Resources.Load<T>(...)`, `Resources.UnloadAsset`. Reference assets directly via inspector fields.
- `AssetBundle.*`.

### Scene management
- `SceneManager.LoadScene`, `SceneManager.UnloadSceneAsync`. VRChat manages world boundaries.

### Component creation
- **`gameObject.AddComponent<T>()` for any T.**
- `new GameObject(...)`.
- Constructors on `UnityEngine.Component` subtypes generally.

### Rendering
- `Graphics.DrawMesh` (non-instanced) — use `DrawMeshInstanced`.
- `Graphics.DrawProcedural*`.
- `Graphics.SetRenderTarget` — use `RenderTexture.active`.
- `MaterialPropertyBlock` — generally believed blocked. **[Unverified]**

### Editor-only Unity APIs
- `UnityEditor.*` namespace — these only exist in the editor and are stripped at build time anyway. Guarded `#if UNITY_EDITOR` editor code is fine; runtime references error.

## Why these are blocked

Most of the blocks fall into three categories:

1. **Security model.** I/O, native code, threads, reflection.
2. **VM limitations.** Generics, lambdas, async, exceptions.
3. **VRChat-specific safety.** `AddComponent` (worlds can't grow at runtime), `Resources.Load` (no arbitrary asset loading), `SceneManager` (worlds are fixed).

The allowlist has gradually expanded across SDK versions — e.g. `enabled` / `isActiveAndEnabled` on `Behaviour` derivatives became available in **SDK 3.10.2**. **[Docs]** Always check current release notes if community advice says "X doesn't work." See [Compiler → SDK releases that changed the compiler](README.md#sdk-releases-that-changed-the-compiler).

## See also

- [Workarounds](workarounds.md)
- [GameObject lifecycle → runtime constraints](../gameobject-lifecycle/runtime-constraints.md)
