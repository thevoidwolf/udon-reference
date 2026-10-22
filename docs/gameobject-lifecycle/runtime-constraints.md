# Runtime Constraints

Things Udon does **not** let you do at runtime, in one place, reviewed for SDK 3.10.5 (Sep 2026). These follow from the Udon VM allowlist; VRChat does widen the allowlist over time (see [What changed recently](#what-changed-recently)), so confirm edge cases in **VRChat SDK → Udon Sharp → Class Exposure Tree** (red = not exposed, green = exposed). **[Docs]**

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
| `Coroutine` (`IEnumerator`-yielding methods) | Coroutines are not in the allowlist. Use `SendCustomEventDelayedSeconds`/`Frames`, or `VRCTween.DelayedCall` (SDK 3.10.4+), to schedule deferred work. |
| `Job System` / `Burst` | Blocked. |

## Cannot use reflection

| Call | Reason |
|---|---|
| Dynamic type lookup | `typeof()` is supported **[Docs]**, but there is no runtime type lookup by name. Note `.GetType()` may not return what you expect, since U# abstracts some types (e.g. any jagged array reports `object[]`). **[Docs]** |
| `System.Reflection.*` | All reflection APIs are blocked. Use `GetProgramVariable`/`SetProgramVariable` or `SendCustomEvent`. |
| `Activator.CreateInstance` | Blocked. |

## Cannot use generic collections

| Call | Reason |
|---|---|
| `List<T>`, `Dictionary<K,V>`, other generic collections | "Udon and UdonSharp currently only support array `[]` collections and data containers." Use arrays or [`DataList` / `DataDictionary`](../data-containers/README.md). **[Docs]** |

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
| Schedule deferred work | `SendCustomEventDelayedSeconds(eventName, seconds)` / `SendCustomEventDelayedFrames`; optional `EventTiming` now includes `FixedUpdate` and `PostLateUpdate` (SDK 3.10.2+). **[Docs]** |
| Cancelable timer | `VRCTween.DelayedCall(this, nameof(Handler), seconds)` returns a `VRCTweenHandle` you can `Kill()` (SDK 3.10.4+). **[Docs]** |
| Animate a transform / value over time without an `Update` loop | VRCTween, e.g. `obj.TweenPosition(target, 2f, VRCTweenEase.OutQuad)` (SDK 3.10.4+; a DOTween subset). **[Docs]** <https://creators.vrchat.com/worlds/udon/vrctween/> |
| Secondary motion, touch/proximity triggers, constraints | PhysBones, Contacts and VRC Constraints are supported in worlds with Udon APIs/events (SDK 3.10.0+) — they must still be authored on the template, not added at runtime. **[Docs]** |
| Toggle a component | `enabled` / `isActiveAndEnabled` on any `Behaviour` (SDK 3.10.2+). **[Docs]** |
| Load a texture | `VRCImageDownloader.DownloadImage`. |
| Load text from a URL | `VRCStringDownloader.LoadUrl`. |
| Persist data across sessions | `PlayerData` / Player Objects (SDK 3.7.4+). |
| Run reflection-style work | `GetProgramVariable` / `SetProgramVariable`. |
| Run "background" tasks | Frame-spread your work using `SendCustomEventDelayedFrames`. |

## What changed recently

Things older guides list as impossible that are now supported (all **[Docs]**, release notes):

| SDK | Now possible |
|---|---|
| 3.7.4 | Cross-session persistence (`PlayerData`, `VRCPlayerObject`) — no need for `PlayerPrefs`. |
| 3.7.5 | `GetComponent` on all TextMeshPro components. |
| 3.10.0 | PhysBones, Contacts, VRC Constraints in worlds, with Udon APIs and events. |
| 3.10.2 | `enabled` / `isActiveAndEnabled` on all `Behaviour` derivatives; `EventTiming.FixedUpdate` / `PostLateUpdate` for delayed events. |
| 3.10.4 | VRCTween (tweens, virtual tweens, cancelable `DelayedCall`, `DelayedSetActive`). `UdonSharpBehaviour` implements `IUdonEventReceiver`. |

Still **not** possible as of 3.10.5: `AddComponent`, `new GameObject()`, coroutines, generic collections, reflection, threading, file/raw network I/O.

## VRChat's position on `AddComponent`

`Instantiate` is documented as not networked, with "the only workaround is to use object-pooling." **[Docs]** <https://udonsharp.docs.vrchat.com/networking-tips-&-tricks/> There is **no public commitment** from VRChat to ever expose runtime `AddComponent`. Treat as not coming. **[Community-known]**

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
