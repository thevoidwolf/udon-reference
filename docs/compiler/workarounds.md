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

You can't write `T Pick<T>(T[] items)`. Options:
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
// Use individual `out` params, or pack into a small class — but classes have their own caveats.
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

Enums with `[Flags]` are supported. Bitwise ops on `int`-backed enums work normally.

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
| Type-safe generic data containers | `koyashiro/generic-data-container` | <https://github.com/koyashiro/generic-data-container> |
| Cancellable delayed events | `MyroG/Advanced-local-events` | <https://github.com/MyroG/Advanced-local-events> |
| Coroutine-style scheduling | `Caeden117/UdonCoroutines` | <https://github.com/Caeden117/UdonCoroutines> |
| Event-hub / pub-sub between behaviours | `orels1/UdonToolkit` (EventDispatcher) | <https://ut.orels.sh/attributes/attributes-list> |
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
| "try/catch not supported" | No exception handling | Use guard clauses / early returns. |
| "out/in/ref parameters not supported" | Not allowed on user methods | Return arrays / structs, or use fields. |
| "is/as not supported" | Type-pattern checks not allowed | Restructure with explicit type fields, or upgrade to U# 1.x. |
| Recursion stack overflow | Method called itself without `[RecursiveMethod]` | Add the attribute. |
| Synced var set, event arrives before var update on remote | Event-before-var race | Defer event a frame, or detect change in `OnDeserialization`. |

**[Community]** — error-message table cross-referenced with UdonRabbit.Analyzer's rule list (URA0001–URA0055).

## See also

- [Supported features](supported-features.md)
- [Unsupported features](unsupported-features.md)
- [Data Containers](../data-containers/README.md)
- [Persistence](../persistence/README.md)
- [References → Community Resources](../references/community-resources.md)
