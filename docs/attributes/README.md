# Attributes

UdonSharp ships four custom attributes plus two supporting enums. Source: `Runtime/UdonSharpAttributes.cs`. **[Source]** The VRChat Worlds SDK adds `[NetworkCallable]` (SDK 3.8.1+, namespace `VRC.SDK3.UdonNetworkCalling`). **[Docs]**

Official reference: <https://creators.vrchat.com/worlds/udon/udonsharp/attributes/>. It covers `UdonSynced`, `UdonSyncMode`, `UdonBehaviourSyncMode` and `BehaviourSyncMode`, and recommends Varneon's UdonSharp Development Practices for an overview. **[Docs]** (The old `udonsharp.docs.vrchat.com/attributes` URL now returns 404.)

| Page | Symbol |
|---|---|
| [UdonSynced](udon-synced.md) | `[UdonSynced(UdonSyncMode)]` — mark a field for network sync. |
| [BehaviourSyncMode](behaviour-sync-mode.md) | `[UdonBehaviourSyncMode(BehaviourSyncMode)]` — class-level sync mode enforcement. |
| [RecursiveMethod](recursive-method.md) | `[RecursiveMethod]` — allow a method to recurse safely. |
| [FieldChangeCallback](field-change-callback.md) | `[FieldChangeCallback(string)]` — route writes through a property setter. |
| [NetworkCallable](network-callable.md) | `[NetworkCallable]` / `[NetworkCallable(maxEventsPerSecond: X)]` — make a method callable over the network, with up to 8 parameters and a rate limit. **(SDK 3.8.1+)** |

`UdonSharpAttributes.cs` also declares `[CompileInit]`, marked `[Obsolete("This attribute currently does nothing")]`. Don't use it. **[Source]**

Standard Unity / .NET attributes generally also work on U# fields and classes:

| Attribute | Use |
|---|---|
| `[SerializeField]` | Serialize a non-public field. |
| `[Header("...")]` `[Tooltip("...")]` `[Space]` | Inspector cosmetics. |
| `[HideInInspector]` | Don't show this field in the inspector. |
| `[Range(min, max)]` | Slider in inspector. |
| `[TextArea]` | Multi-line string field. |
| `[ColorUsage(...)]` | Color picker config. |
| `[AddComponentMenu("Path/Name")]` | Where this script appears in the Add Component menu. |
| `[DefaultExecutionOrder(int)]` | Unity execution order. |
| `[FormerlySerializedAs("oldName")]` | Migration: pull data from a previously-named field. |
| `[JetBrains.Annotations.PublicAPI]` | Suppress "unused" warnings; no runtime effect. |

`[Obsolete]` is supported but only as an editor warning — the runtime doesn't enforce it.

## Not supported

- `[ContextMenu]` does nothing on U# (no MonoBehaviour `Editor` to wire it through at runtime).
- Custom attributes whose constructors run user code at attribute-load time. The Udon VM cannot execute the constructor — the attribute is preserved on the proxy but is invisible at runtime.
- `[RuntimeInitializeOnLoadMethod]` and other engine-bootstrap attributes — Udon programs are loaded on a per-behaviour basis, not a static class basis.
