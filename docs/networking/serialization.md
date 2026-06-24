# Serialization

The packet model behind `[UdonSynced]` fields.

## Lifecycle

For continuous sync, the runtime:
1. Calls `OnPreSerialization()` on the owner.
2. Reads all `[UdonSynced]` fields, packs them into a binary packet.
3. Sends to remote clients.
4. On the owner, calls `OnPostSerialization(result)`.
5. On each remote client, applies the bytes to local fields, calls `OnDeserialization()`.

For manual sync the same flow happens, gated on the owner explicitly calling `RequestSerialization()`.

## `SerializationResult` (owner side)

`VRC.Udon.Common.SerializationResult` is passed to `OnPostSerialization`.

| Member | Type | Meaning |
|---|---|---|
| `success` | `bool` | `false` if the packet was too large to fit in the budget and was dropped. |
| `byteCount` | `int` | Number of bytes written. |

```csharp
public override void OnPostSerialization(SerializationResult result) {
    if (!result.success) Debug.LogWarning($"sync dropped: {result.byteCount} bytes");
}
```

## `DeserializationResult` (remote side)

`VRC.Udon.Common.DeserializationResult` is passed to the two-argument `OnDeserialization`:

| Member | Type | Meaning |
|---|---|---|
| `sendTime`    | `float` | Network time at which the owner sent the packet. |
| `receiveTime` | `float` | Network time at which this client applied it. |

Useful for compensating latency in time-sensitive visuals.

## Byte budget

Per VRChat's official Network Details page (<https://creators.vrchat.com/worlds/udon/networking/network-details/>) **[Docs]**:

| Mode | Per-serialization packet limit | Notes |
|---|---|---|
| **Manual** | **~280,496 bytes** | Soft cap; events >1024 B split internally. |
| **Continuous** | **~200 bytes per serialization** | Small per-tick budget, broadcasts frequently. |
| Bandwidth ceiling | ~11 KB/s per script (documented), ~4 KB/s sustained (community-observed) | **[Docs]** + **[Community]** <https://ask.vrchat.com/t/question-about-udon-bandwidth/24903> |

Rate-limiting scales with payload — bigger sends extend the cooldown before the next `RequestSerialization()` actually goes out (`OnPreSerialization` is deferred until the budget allows).

When the connection is "clogged":
- **Continuous** behaviours **drop** the serialization event and log errors.
- **Manual** behaviours **queue and retry**.

`bool` syncs as ≥1 byte; per-packet overhead is documented as small but unspecified. Community estimates ~20 bytes per packet. **[Community]** <https://ask.vrchat.com/t/question-about-udon-bandwidth/24903>

If your packet exceeds the budget, `OnPostSerialization(result)` reports `success == false` and the packet is dropped — non-owners do not get an update.

> **Long synced strings** — A single `[UdonSynced] string` over a few hundred bytes will already blow the **continuous** sync budget. This is why USharpVideo and most modern video players use manual sync for URL state. **[Community]** <https://github.com/MerlinVR/USharpVideo>

## What counts toward the budget

Each `[UdonSynced]` field contributes:

| Type | Approx bytes |
|---|---|
| `bool`, `byte`, `sbyte` | 1 |
| `short`, `ushort`, `char` | 2 |
| `int`, `uint`, `float` | 4 |
| `long`, `ulong`, `double` | 8 |
| `Vector2` | 8 |
| `Vector3` | 12 |
| `Vector4`, `Quaternion`, `Color` | 16 |
| `Color32` | 4 |
| `string` | length + a small overhead |
| Array | total of all elements + overhead |

Plus per-packet overhead (small).

## Reducing packet size

- Split state across multiple manual-sync behaviours.
- Encode multiple booleans as a bitmask (`int`).
- Quantize floats: a `Vector3` at 1 mm precision can be a `Vector3Int` of mm * 1000 (12 bytes either way, but you can pack each component into a `short` for 6 bytes if your range is small).
- Avoid sending strings whose content didn't change.

## Bandwidth-aware patterns

### Dirty-only sync

```csharp
[UdonSynced] int _dataA;
[UdonSynced] int _dataB;

public void SetA(int a) {
    if (a == _dataA) return;
    _dataA = a;
    if (Networking.IsOwner(gameObject)) RequestSerialization();
}
```

### Coalescing writes

```csharp
bool _serialPending;

public void MarkDirty() {
    if (_serialPending) return;
    _serialPending = true;
    SendCustomEventDelayedFrames(nameof(_DoSerial), 1);
}
public void _DoSerial() {
    _serialPending = false;
    if (Networking.IsOwner(gameObject)) RequestSerialization();
}
```

## What does *not* go through the serialization pipeline

- **GameObject Transforms** — use `VRCObjectSync` if you want them networked.
- **Animator states** — use `[UdonSynced]` parameters mapped to animator parameters in `OnDeserialization`.
- **Particle systems, audio** — local-only. To synchronize, fire `SendCustomNetworkEvent` on trigger.

## Known gotchas (community-sourced)

- **Event-before-variable race.** `SendCustomNetworkEvent` packets arrive faster than serialized variable updates. Setting a synced var, then firing an event, leaves remote receivers running the event handler with the old value. Workarounds: defer the event by a frame, or detect the change in `OnDeserialization` / `OnVariableChanged` instead. **[Community]** <https://udonsharp.docs.vrchat.com/networking-tips-&-tricks/>
- **`OnVariableChanged` can fire before `Start()`** on late joiners when the owner calls `RequestSerialization()` inside `OnPlayerJoined`. Tracked since 2022 and still reported in late 2024. **[Community]** <https://feedback.vrchat.com/udon/p/1259-synced-variables-can-be-changed-from-owners-requestserialization-before-lat>
- **`Networking.GetServerTimeInMilliseconds()` sign asymmetry.** Returns positive for some players and negative for others; the sign persists per-session. Time-anchored sync algorithms must use unchecked-wrap subtraction or sign-aware deltas. **[Community]** <https://phi16.hatenablog.com/entry/2025/01/08/023655>
- **Bigger objects beat many small objects.** Per-serialization overhead is amortized; one packet with a packed array of values is cheaper than many packets with individual values. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/performance/>
- **Bit-pack booleans.** A `bool[]` costs ~1 byte per element. Pack 8 bools into a `byte` (or 32 into an `int`) for 8–32× savings. **[Docs]** <https://creators.vrchat.com/worlds/udon/data-containers/byte-and-bit-operations/>

## See also

- [Sync modes](sync-modes.md)
- [Attributes → UdonSynced](../attributes/udon-synced.md)
- [UdonSharpBehaviour → networking events](../udonsharp-behaviour/networking-events.md)
- [References → Community Resources](../references/community-resources.md) — `VRCLockstep`, `UNet` and other RPC/networking libraries.
