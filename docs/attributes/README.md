# Attributes

UdonSharp ships four custom attributes plus two supporting enums. Source: `Runtime/UdonSharpAttributes.cs`. **[Source]**

| Page | Symbol |
|---|---|
| [UdonSynced](udon-synced.md) | `[UdonSynced(UdonSyncMode)]` — mark a field for network sync. |
| [BehaviourSyncMode](behaviour-sync-mode.md) | `[UdonBehaviourSyncMode(BehaviourSyncMode)]` — class-level sync mode enforcement. |
| [RecursiveMethod](recursive-method.md) | `[RecursiveMethod]` — allow a method to recurse safely. |
| [FieldChangeCallback](field-change-callback.md) | `[FieldChangeCallback(string)]` — route writes through a property setter. |

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
