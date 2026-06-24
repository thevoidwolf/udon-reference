# Unsupported Features

C# and Unity features the UdonSharp compiler rejects. Most of these have a [workaround](workarounds.md).

## Language features

### Generics in user code
- **No generic methods.** `T DoThing<T>(T x)` is rejected.
- **No generic classes.** `class Bag<T> : UdonSharpBehaviour` is rejected.
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
- Pattern matching with `is T t`.
- Tuples (`(int, string)`, `ValueTuple`).
- Record types.
- `init` accessors.
- `with` expressions.
- Top-level statements.
- Local functions.
- Range / index operators (`a[1..^1]`).

### Lambdas / delegates
- No anonymous methods or lambda expressions.
- No `Action`, `Func`, `Predicate` types.
- No `event` declarations on `UdonSharpBehaviour`s.

### Other
- `ref` / `out` parameters in user-defined methods, except where required by API contracts (e.g. `DataDictionary.TryGetValue`).
- Custom indexers (`this[int i]`).
- Operator overloading on user types.
- Conversion operators (implicit/explicit) on user types.
- Static constructors.
- `try`/`catch`/`finally`. **No exception handling at all.**
- `throw` statements.
- Constructors with parameters on `UdonSharpBehaviour` subclasses.
- `goto label` (not `goto case`).
- `checked` / `unchecked` blocks (overflow is undefined; do your own checks).

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
- `MaterialPropertyBlock` — unverified, generally believed blocked.

### Editor-only Unity APIs
- `UnityEditor.*` namespace — these only exist in the editor and are stripped at build time anyway. Guarded `#if UNITY_EDITOR` editor code is fine; runtime references error.

## Why these are blocked

Most of the blocks fall into three categories:

1. **Security model.** I/O, native code, threads, reflection.
2. **VM limitations.** Generics, lambdas, async, exceptions.
3. **VRChat-specific safety.** `AddComponent` (worlds can't grow at runtime), `Resources.Load` (no arbitrary asset loading), `SceneManager` (worlds are fixed).

The U# team has gradually expanded what's allowed across versions. Always check current docs if community advice says "X doesn't work."

## See also

- [Workarounds](workarounds.md)
- [GameObject lifecycle → runtime constraints](../gameobject-lifecycle/runtime-constraints.md)
