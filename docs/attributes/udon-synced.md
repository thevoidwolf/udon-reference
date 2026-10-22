# `[UdonSynced]` and `UdonSyncMode`

Marks a field on an `UdonSharpBehaviour` as network-replicated. Defined in `Runtime/UdonSharpAttributes.cs`. **[Source]**

## Signature

```csharp
[AttributeUsage(AttributeTargets.Field, AllowMultiple = false, Inherited = false)]
public class UdonSyncedAttribute : Attribute
{
    public UdonSyncMode NetworkSyncType { get; }
    public UdonSyncedAttribute(UdonSyncMode networkSyncTypeIn = UdonSyncMode.None);
}
```

## Usage

```csharp
[UdonSynced] public int score;
[UdonSynced(UdonSyncMode.Linear)]  public float lerpedFloat;
[UdonSynced(UdonSyncMode.Smooth)]  public Vector3 smoothedPos;
```

The field's value is included in the next serialization packet whenever the **owner** runs `RequestSerialization()` (manual sync) or the runtime auto-broadcasts (continuous sync). Non-owners receive the value and, after the apply, get an `OnDeserialization` callback.

## `UdonSyncMode` values

```csharp
public enum UdonSyncMode
{
    NotSynced,
    None,
    Linear,
    Smooth,
}
```

| Value | Behaviour |
|---|---|
| `NotSynced` | Equivalent to omitting the attribute. Its `[Obsolete]` marker is commented out in source, so it's effectively soft-deprecated. **[Source]** |
| `None` | Snap to the new value. No interpolation. The default. |
| `Linear` | Lerp toward the new value over the sync interval. **Numeric types only.** |
| `Smooth` | Smoothed interpolation. **Numeric types only.** |

`Linear` / `Smooth` apply only to fields whose type is a scalar number or `Vector2`/`3`/`4`, `Quaternion`, `Color`. The compiler errors on non-numeric types with these modes.

## What's syncable

The Udon allowlist accepts these field types for sync:

- Primitives: `bool`, `byte`, `sbyte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `float`, `double`.
- `string` (max length capped by packet budget — see [serialization](../networking/serialization.md)).
- `char`.
- Unity value types: `Vector2`, `Vector3`, `Vector4`, `Quaternion`, `Color`, `Color32`.
- `VRCUrl` (~2 bytes per char). **[Docs]**
- Arrays of any of the above. **Always initialize synced arrays** (at least to an empty array): if any synced array is `null`, the behaviour won't sync at all. **[Docs]**

Official list: <https://creators.vrchat.com/worlds/udon/networking/network-details/#synced-variables>. The same set is valid for `[NetworkCallable]` parameters (SDK 3.8.1+). **[Docs]**

**Not syncable:**
- `DataList` / `DataDictionary` / `DataToken` — these are reference types and not in the syncable list. Use a string of serialized JSON via `VRCJson` and sync the string.
- Custom structs / classes — only the listed Unity value types.
- `UnityEngine.Object` references (GameObjects, components, materials). Cross-network object references don't have a stable identity to sync against. Use indices into pre-placed arrays.
- Multi-dimensional arrays. Jagged arrays via `[UdonSynced]` are not supported.

## Sync mode enforcement

Whether continuous or manual sync applies is set per-behaviour, either by:

1. The `Synchronization` dropdown in the `UdonBehaviour` inspector (when `BehaviourSyncMode.Any`), or
2. A `[UdonBehaviourSyncMode(...)]` attribute on the class.

Manual sync requires `RequestSerialization()` calls from the owner; continuous sync is automatic. See [`[UdonBehaviourSyncMode]`](behaviour-sync-mode.md).

## Combining with `[FieldChangeCallback]`

```csharp
[UdonSynced, FieldChangeCallback(nameof(Score))]
private int _score;

public int Score {
    get => _score;
    set {
        _score = value;
        scoreText.text = value.ToString();
    }
}
```

When the deserialized value lands, the setter runs instead of writing the field directly. Lets you keep visual state in sync with field state without overriding `OnDeserialization`. See [`[FieldChangeCallback]`](field-change-callback.md).

## Packet budget

Per VRChat's official Network Details page: **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-details/>

- **Continuous sync:** ~200 bytes per serialization. Long strings or arrays blow this immediately.
- **Manual sync:** ~280 KB per serialization. Much higher headroom.
- Bandwidth ceiling: "about 11 kilobytes per second" for Udon scripts. VRChat notes all specs are subject to change.

If your packet exceeds the budget, `OnPostSerialization(result)` reports `result.success == false`. Split state across multiple behaviours or use string-encoded chunks. See [Serialization](../networking/serialization.md) for size-reduction techniques.

## See also

- [Networking → sync modes](../networking/sync-modes.md)
- [Networking → serialization](../networking/serialization.md)
- [`[FieldChangeCallback]`](field-change-callback.md)
