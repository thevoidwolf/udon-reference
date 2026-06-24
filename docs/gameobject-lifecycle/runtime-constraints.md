# Runtime Constraints

Things Udon does **not** let you do at runtime, in one place. Every entry here is a hard rule of the Udon VM allowlist as of writing — not a temporary limitation.

## Cannot create

| Call | Reason |
|---|---|
| `new GameObject(...)` | No raw GameObject construction. Always `Instantiate` from a template. |
| `gameObject.AddComponent<T>()` | The allowlist forbids `AddComponent` for *any* type. Template-time is the only time to attach components. |
| `new UdonBehaviour()` | Same — UdonBehaviours cannot be created at runtime. |
| `new Camera(...)`, `new Light(...)`, etc. | All `UnityEngine.Component` subtype constructors are blocked at runtime. |

## Cannot dynamically load

| Call | Reason |
|---|---|
| `Resources.Load<T>(string)` | The Resources system is not exposed. Reference assets directly through inspector fields. |
| `AssetBundle.LoadFromFile` etc. | File / asset-bundle I/O is blocked. |
| `SceneManager.LoadScene` | World-internal scene loading is not allowed; VRChat manages world boundaries. |

## Cannot do filesystem / network I/O

| Call | Reason |
|---|---|
| `System.IO.File.*` | Filesystem access not allowed. |
| `WWW`, `UnityWebRequest`, `HttpClient`, raw sockets | Use `VRCStringDownloader` / `VRCImageDownloader` instead, which go through trusted URL gates. |
| `System.Net.*` | Blocked. |
| `PlayerPrefs.*` | Use [`PlayerData`](../persistence/player-data.md) for cross-session storage. |

## Cannot use threading / async

| Call | Reason |
|---|---|
| `Task`, `async`/`await` | Udon is single-threaded. |
| `Thread`, `Mutex`, `Semaphore` | Same. |
| `Coroutine` (`IEnumerator`-yielding methods) | Coroutines are not in the allowlist. Use `SendCustomEventDelayedSeconds`/`Frames` to schedule deferred work. |
| `Job System` / `Burst` | Blocked. |

## Cannot use reflection

| Call | Reason |
|---|---|
| `typeof(T)` is supported at compile time only | The compiler resolves it to a constant. Dynamic type lookup is not. |
| `System.Reflection.*` | All reflection APIs are blocked. Use `GetProgramVariable`/`SetProgramVariable` or `SendCustomEvent`. |
| `Activator.CreateInstance` | Blocked. |

## Cannot use `unsafe` / native interop

| Call | Reason |
|---|---|
| `unsafe`, pointer types, `stackalloc` | Not in the compiler allowlist. |
| `DllImport` / native function calls | Blocked. |
| `Span<T>`, `Memory<T>`, `ref struct` | Blocked. |

## Workarounds

| Want | Do this instead |
|---|---|
| Create a GameObject at runtime | Place a disabled template in the scene → `Instantiate`. |
| Attach behaviour at runtime | Author the behaviour onto the template *before* upload. |
| Schedule deferred work | `SendCustomEventDelayedSeconds(eventName, seconds)` / `SendCustomEventDelayedFrames`. |
| Load a texture | `VRCImageDownloader.DownloadImage`. |
| Load text from a URL | `VRCStringDownloader.LoadUrl`. |
| Persist data across sessions | `PlayerData` / Player Objects. |
| Run reflection-style work | `GetProgramVariable` / `SetProgramVariable`. |
| Run "background" tasks | Frame-spread your work using `SendCustomEventDelayedFrames`. |

## VRChat's position on `AddComponent`

`Instantiate` is documented as not networked, with "the only workaround is object pooling." There is **no public commitment** from VRChat to ever expose runtime `AddComponent`. Treat as permanently not coming. **[Community]** <https://udonsharp.docs.vrchat.com/networking-tips-&-tricks/>

The community-canonical "dynamic loadout" pattern, in lieu of `AddComponent`:

1. Place every possible loadout/item as a disabled child under a holder GameObject.
2. On equip, `SetActive(true)` the chosen ones, `SetActive(false)` the others.
3. Sync the chosen index via `[UdonSynced]`, apply state in `OnDeserialization`.

Documented in Varneon's *UdonSharp Development Practices*. **[Community]** <https://vrclibrary.com/wiki/books/varneons-udonsharp-development-practices/export/html>

## See also

- [Instantiate](instantiate.md)
- [Compiler constraints](../compiler/README.md)
- [Loading](../loading/README.md)
- [Persistence](../persistence/README.md)
- [References → Community Resources](../references/community-resources.md)
