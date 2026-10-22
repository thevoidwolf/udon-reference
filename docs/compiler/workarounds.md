# Workarounds

Substitutions for missing C# features.

## `List<T>` / `Dictionary<TKey, TValue>`

| Need | Substitute |
|---|---|
| Variable-size collection of one type | Manually-managed array + `int _count`, grow with `System.Array.Resize` or pre-allocate. |
| Heterogeneous collection | `VRC.SDK3.Data.DataList` of `DataToken`. |
| Key-value lookup | `VRC.SDK3.Data.DataDictionary`. |

See [Data Containers](../data-containers/README.md).

## Generic methods

You can't declare `T Pick<T>(T[] items)` on a `UdonSharpBehaviour`. Options:
- Move it into a separate `static class` as a generic static (or extension) method — the bundled compiler accepts that. **[Code-confirmed]** `Tests~/TestScripts/Core/GenericsTest.cs`
- Inline the logic at each call site.
- Use `object[]` and cast.
- Specialize per type: `int PickInt(int[])`, `string PickString(string[])`.

## `async` / `await`

| Need | Substitute |
|---|---|
| "Run this after N seconds" | `SendCustomEventDelayedSeconds(eventName, seconds)`. |
| "Run this next frame" | `SendCustomEventDelayedFrames(eventName, 1)`. |
| "Frame-spread work over several frames" | Schedule chained delayed events processing chunks. |
| "Wait for a network event" | Override `OnPlayerJoined` / `OnDeserialization` / etc. |
| "Wait for a download" | `OnImageLoadSuccess`, `OnStringLoadSuccess`, etc. |

## Coroutines

Same answer as async: chain `SendCustomEventDelayedFrames` calls.

```csharp
public void StartTween() {
    _t = 0f;
    _DoTweenStep();
}
public void _DoTweenStep() {
    _t += Time.deltaTime;
    transform.position = Vector3.Lerp(start, end, _t);
    if (_t < 1f) SendCustomEventDelayedFrames(nameof(_DoTweenStep), 1);
}
```

(Or simpler: just put it in `Update()`.)

## Try / catch

Udon doesn't throw exceptions. Most ops that *would* throw in regular C# either return a success flag (`TryGetValue`) or halt the program. Defensive coding:

```csharp
// Instead of: try { x = arr[i]; } catch { x = -1; }
int x = (i >= 0 && i < arr.Length) ? arr[i] : -1;
```

For the few APIs that do throw (e.g. `DataToken.Int` on a non-Int token), test `TokenType` first.

## Reflection

| Need | Substitute |
|---|---|
| Read a field by name on another behaviour | `behaviour.GetProgramVariable("name")`. |
| Write a field by name | `behaviour.SetProgramVariable("name", value)`. |
| Invoke a method by name | `behaviour.SendCustomEvent("methodName")`. |
| Type checks across UdonBehaviours | `behaviour.GetUdonTypeID() == GetUdonTypeID<MyScript>()`. |

See [UdonSharpBehaviour → API methods](../udonsharp-behaviour/api-methods.md).

## LINQ

Rewrite as explicit loops. The common patterns:

```csharp
// Where
int count = 0;
for (int i = 0; i < src.Length; i++) if (src[i] > 5) count++;
int[] result = new int[count];
int j = 0;
for (int i = 0; i < src.Length; i++) if (src[i] > 5) result[j++] = src[i];

// Sum
int sum = 0;
for (int i = 0; i < src.Length; i++) sum += src[i];

// Any
bool any = false;
for (int i = 0; i < src.Length; i++) if (src[i] > 5) { any = true; break; }
```

## Tuples

C#:
```csharp
(int, string) Get() => (1, "hello");
```

U#:
```csharp
// Use individual `out` params (supported on user methods [Docs]), or pack into a DataList / parallel arrays.
void Get(out int a, out string b) { a = 1; b = "hello"; }
```

## Custom exceptions / error codes

Return a `bool` plus an `out` parameter, mirroring the `Try*` pattern that the SDK uses:

```csharp
bool TryDoThing(string input, out int result) {
    if (string.IsNullOrEmpty(input)) { result = 0; return false; }
    result = input.Length;
    return true;
}
```

## `enum` and Bit flags

Enums with `[Flags]` are supported. Bitwise ops on `int`-backed enums work normally. Declare your enums at namespace level, not nested inside the behaviour ("Nested type declarations are not currently supported by U#"). **[Source]** If an enum `switch` or comparison misbehaves on an older SDK, update — an enum miscompile tied to specific member counts was fixed in **SDK 3.10.4**. **[Docs]**

## Static fields / shared state

Static fields on your own types are rejected ("Static fields are not yet supported on user defined types"). **[Source]** Use `const` for compile-time constants; for state shared between behaviours, put it in instance fields on one "manager" behaviour and give others a reference to it.

## Object initializers / `?.` / `is`

| C# | U# substitute |
|---|---|
| `new Foo { X = 1 }` | Construct, then assign fields on separate lines. Array initializers `{ 1, 2 }` are fine. |
| `a?.b` | `a != null ? a.b : default` (or `Utilities.IsValid(a)` for Unity objects / players). |
| `x is Foo f` / `x as Foo` | Keep a correctly-typed field, or compare U# type IDs — see [Type identity](../udonsharp-behaviour/type-identity.md). |

All **[Source]** (no binder support in the bundled compiler).

## `nameof(...)`

Supported. Use it everywhere instead of magic strings:

```csharp
SendCustomEvent(nameof(DoTheThing));
SendCustomNetworkEvent(NetworkEventTarget.All, nameof(Fire));
```

This is important because string-typed event names won't be caught by IDE rename refactors otherwise.

## Threading

There is no threading in Udon. If you need to spread work across frames, see "async / await" above.

## File I/O

Persistence: [`PlayerData`](../persistence/player-data.md). External text: [`VRCStringDownloader`](../loading/string-loading.md). External images: [`VRCImageDownloader`](../graphics/image-downloader.md).

## Community libraries that fill specific gaps

| Need | Library | Source |
|---|---|---|
| Type-safe generic data containers | `koyashiro/generic-data-container` (stale since Feb 2025) | <https://github.com/koyashiro/generic-data-container> |
| Cancellable delayed events | `MyroG/Advanced-local-events` | <https://github.com/MyroG/Advanced-local-events> |
| Coroutine-style scheduling | `Caeden117/UdonCoroutines` | <https://github.com/Caeden117/UdonCoroutines> |
| Event-hub / pub-sub between behaviours | `orels1/UdonToolkit` (EventDispatcher; stale, predates U# 1.0) | <https://ut.orels.sh/attributes/attributes-list> |
| Reliable typed RPC | `Xytabich/UNet` (archived) | <https://github.com/Xytabich/UNet> |
| Deterministic shared-state networking | `JanSharp/VRCLockstep` | <https://github.com/JanSharp/VRCLockstep> |
| Post-compile bytecode optimisation | `BlueAmulet/UdonSharpOptimizer` | <https://github.com/BlueAmulet/UdonSharpOptimizer> |

All **[Community]**.

## Common compile-error → fix translation

| Error | Cause | Fix |
|---|---|---|
| "X is not exposed to Udon" | Method/field not whitelisted (e.g. `Math.PI`, certain operators) | Use the `Mathf` equivalent or a VRChat-exposed API. |
| "Cannot use generic method X" | Generic dispatch unsupported except for whitelisted Unity types | Use the non-generic overload: `(UdonBehaviour)GetComponent(typeof(UdonBehaviour))`. |
| "Awake unsupported, use Start" | `Awake` not exposed | Use `Start`. `OnEnable` fires before `Start`. |
| "Try/Catch/Finally is not supported by UdonSharp…" **[Source]** | No exception handling | Use guard clauses / early returns. |
| "Static fields are not yet supported on user defined types" **[Source]** | `static` / `static readonly` field | Use `const` or an instance field on a shared manager behaviour. |
| "Cannot use typeof on user-defined types" **[Source]** | `typeof(MyScript)` | Use `GetUdonTypeID<MyScript>()` / `GetComponent<MyScript>()`. |
| "U# Does not yet support initializer lists" **[Source]** | `new Foo { … }` | Assign members after construction. |
| "Local method declarations are not currently supported…" **[Source]** | Local function | Promote to a private method. |
| "UdonSharpBehaviour classes must have the same name as their containing .cs file" **[Source]** | Class/file name mismatch | Rename one to match. |
| "UdonSharp does not currently support node type '…'" **[Source]** | Unsupported syntax (`?.`, `is`/`as`, `goto`, lambdas, `switch` expressions, …) | See [Unsupported features](unsupported-features.md). |
| Recursion stack overflow | Method called itself without `[RecursiveMethod]` | Add the attribute. |
| Synced var set, event arrives before var update on remote | Event-before-var race | Defer event a frame, or detect change in `OnDeserialization`. |

**[Community]** — error-message table originally cross-referenced with the (archived) UdonRabbit.Analyzer rule list (URA0001–URA0055); rows tagged **[Source]** quote the bundled compiler's messages verbatim. Note `ref`/`out` parameters on user methods **are** supported. **[Docs]**

## See also

- [Supported features](supported-features.md)
- [Unsupported features](unsupported-features.md)
- [Data Containers](../data-containers/README.md)
- [Persistence](../persistence/README.md)
- [References → Community Resources](../references/community-resources.md)
