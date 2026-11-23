# Attributes

UdonSharp ships four custom attributes plus two supporting enums. Source: `Runtime/UdonSharpAttributes.cs`. **[Source]** The VRChat Worlds SDK adds `[NetworkCallable]` (SDK 3.8.1+, namespace `VRC.SDK3.UdonNetworkCalling`). **[Docs]**

Official reference: <https://creators.vrchat.com/worlds/udon/udonsharp/attributes/>. Since the Oct 30, 2026 docs update it covers all four U# attributes (`UdonSynced`, `UdonBehaviourSyncMode`, `RecursiveMethod`, `FieldChangeCallback`), `NetworkCallable`, `DefaultExecutionOrder`, and the Unity serialization and inspector attributes. It replaced the old pointer to Varneon's UdonSharp Development Practices. **[Docs]** (The old `udonsharp.docs.vrchat.com/attributes` URL now returns 404.)

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
| `[DefaultExecutionOrder(int)]` | Order of this behaviour's update events. See [below](#defaultexecutionorder). |
| `[NonSerialized]` | Stop a public field from being serialized. **[Docs]** |
| `[SerializeReference]` | Serialize a field by reference. Listed as supported. **[Docs]** |
| `[OdinSerialize]` | Serialize types Unity can't (e.g. jagged arrays). Takes priority over `[NonSerialized]`. Requires Odin. **[Docs]** |
| `[Multiline]` `[GradientUsage(...)]` | More inspector cosmetics, listed as supported. **[Docs]** |
| `[FormerlySerializedAs("oldName")]` | Migration: pull data from a previously-named field. |
| `[JetBrains.Annotations.PublicAPI]` | Suppress "unused" warnings; no runtime effect. |

### DefaultExecutionOrder

`[DefaultExecutionOrder(int)]` on a U# class sets when its `Update`, `LateUpdate`, `FixedUpdate` and `PostLateUpdate` run relative to other Udon behaviours. Everything defaults to `0`. Lower runs earlier, and negatives are allowed. **[Docs]** <https://creators.vrchat.com/worlds/udon/udonsharp/attributes/#defaultexecutionorder>

- It affects **only those four update events**. `Start`, `Interact`, network events and every other event keep their normal order. **[Docs]**
- It is **not inherited**. Put it on the concrete class you attach, not on a base class. **[Docs]**

```csharp
[DefaultExecutionOrder(-10)]   // Update runs before default-order behaviours
public class InputPoller : UdonSharpBehaviour { }
```

`[Obsolete]` is supported but only as an editor warning — the runtime doesn't enforce it.

## Not supported

- `[ContextMenu]` does nothing on U# (no MonoBehaviour `Editor` to wire it through at runtime).
- Custom attributes whose constructors run user code at attribute-load time. The Udon VM cannot execute the constructor — the attribute is preserved on the proxy but is invisible at runtime.
- `[RuntimeInitializeOnLoadMethod]` and other engine-bootstrap attributes — Udon programs are loaded on a per-behaviour basis, not a static class basis.
