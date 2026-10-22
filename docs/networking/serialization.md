# Serialization

The packet model behind `[UdonSynced]` fields.

## Lifecycle

For continuous sync, the runtime:
1. Calls `OnPreSerialization()` on the owner.
2. Reads all `[UdonSynced]` fields, packs them into a binary packet.
3. Sends to remote clients.
4. On the owner, calls `OnPostSerialization(result)`.
5. On each remote client, applies the bytes to local fields, calls `OnDeserialization()`.

For manual sync the same flow happens, gated on the owner explicitly calling `RequestSerialization()`. Udon may delay `OnPreSerialization` until the object's rate limit allows the send. **[Docs]**

## `SerializationResult` (owner side)

`VRC.Udon.Common.SerializationResult` is passed to `OnPostSerialization`.

| Member | Type | Meaning |
|---|---|---|
| `success` | `bool` | Whether the send attempt succeeded. `false` e.g. if the packet was too large for the budget, or a synced array was left `null`. |
| `byteCount` | `int` | Number of bytes sent. Use it to measure the real serialized size of your data. **[Docs]** |

```csharp
public override void OnPostSerialization(SerializationResult result) {
    if (!result.success) Debug.LogWarning($"sync dropped: {result.byteCount} bytes");
}
```

## `DeserializationResult` (remote side)

`VRC.Udon.Common.DeserializationResult` is passed to the two-argument `OnDeserialization`:

| Member | Type | Meaning |
|---|---|---|
| `sendTime`    | `float` | When the message was sent, in seconds, on **your local** `Time.realtimeSinceStartup` clock. |
| `receiveTime` | `float` | When the message was received, on the same local clock. |
| `isFromStorage` | `bool` | `true` if the data was restored from **persistence storage** rather than received from a live client. |

All three are **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-components/#deserializationresult>

- `Time.realtimeSinceStartup - result.sendTime` = how many seconds ago the data was sent. Useful for latency compensation.
- These are *local* times, not shared network time. Every player's `sendTime` for the same packet differs. To share a timestamp, sync an **offset**: subtract your `realtimeSinceStartup` before sending, and have receivers add back theirs. **[Docs]**
- `sendTime` can be **negative** if the data was sent before you launched VRChat. **[Docs]**
- Check `isFromStorage` to tell persisted-state restores (Player Objects / `VRCEnablePersistence`) from live updates. See [Persistence](../persistence/README.md).

## Byte budget

Per VRChat's official Network Details page (<https://creators.vrchat.com/worlds/udon/networking/network-details/>) **[Docs]**:

| Mode | Per-serialization packet limit | Notes |
|---|---|---|
| **Manual** | **~280,496 bytes** | Rate-limited as a factor of data size. |
| **Continuous** | **~200 bytes per serialization** | Small per-tick budget, broadcasts frequently. |
| Bandwidth ceiling | "about **11 kilobytes** per second" for Udon scripts (documented); ~4 KB/s sustained (community-observed) | **[Docs]** + **[Community]** <https://ask.vrchat.com/t/question-about-udon-bandwidth/24903> |

VRChat notes that all specs are subject to change. **[Docs]**

Rate-limiting scales with payload. Bigger sends extend the cooldown before the next `RequestSerialization()` actually goes out, because `OnPreSerialization` is deferred until the budget allows. **[Docs]**

Network *events* have separate limits (SDK 3.8.1+): 16 KB of parameters per event, split into internal events above 1024 B, and a total outgoing cap of ~18 KB/s (practically 8–10 KB/s). See [Custom Network Events](custom-network-events.md#size-limits-and-event-splitting). **[Docs]**

When the connection is "clogged" (`Networking.IsClogged == true`) **[Docs]**:
- **Continuous** behaviours **drop** the serialization event and log errors.
- **Manual** behaviours **queue and retry**.
- In both cases the behaviour's logic keeps running; only the data isn't sent or received.

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
| `string` | ~2 bytes per char (approximate in-memory size) + overhead |
| `VRCUrl` | ~2 bytes per char, like `string` |
| Array | total of all elements + overhead |

Plus per-packet overhead (small). These are the official **approximate in-memory** sizes. The serialized size may be larger (a `bool` is at least 1 byte, not 1 bit). Use `OnPostSerialization`'s `byteCount` for real numbers. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-details/#synced-variables>

For network **event parameters**, strings are measured as UTF-8 bytes instead. **[Docs]**

## Reducing packet size

- Split state across multiple manual-sync behaviours.
- Encode multiple booleans as a bitmask (`int`).
- Quantize floats: a `Vector3` at 1 mm precision can be a `Vector3Int` of mm * 1000 (12 bytes either way, but you can pack each component into a `short` for 6 bytes if your range is small).
- Avoid sending strings whose content didn't change.

## Other official sync rules

All items in this section: **[Docs]**.

- **Initialize synced arrays.** If any `[UdonSynced]` array on a behaviour is `null`, the **whole behaviour won't sync**. Initialize to an empty array at minimum, and check `OnPostSerialization`.
- **Visible objects get priority.** Udon periodically checks the visibility of mesh renderers under synced objects and prioritizes visible ones when load-balancing.
- **Mixed sync modes on one GameObject collapse to the most restrictive.** A Manual and a Continuous behaviour on the same object both act as Manual. See [Sync Modes](sync-modes.md#multiple-behaviours-on-one-gameobject).

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

- **Event-before-variable race.** `SendCustomNetworkEvent` packets arrive faster than serialized variable updates. Setting a synced var, then firing an event, leaves remote receivers running the event handler with the old value. Workarounds: defer the event by a frame, detect the change in `OnDeserialization` / `OnVariableChanged` instead, or (SDK 3.8.1+) send the value as a `[NetworkCallable]` parameter. **[Docs]** **[Community]** <https://udonsharp.docs.vrchat.com/networking-tips-&-tricks/>
- **`OnVariableChanged` can fire before `Start()`** on late joiners when the owner calls `RequestSerialization()` inside `OnPlayerJoined`. Tracked since 2022 and still reported in late 2024. **[Community]** <https://feedback.vrchat.com/udon/p/1259-synced-variables-can-be-changed-from-owners-requestserialization-before-lat>
- **Measure, don't guess.** Debug view 6 (Net Objects) shows each object's bytes per serialization and bytes/s. `VRC.SDK3.Network.Stats` (SDK 3.8.1+) exposes the same data to Udon. **[Docs]** See [Debugging & network stats](debugging-and-stats.md).
- **`Networking.GetServerTimeInMilliseconds()` sign asymmetry.** Returns positive for some players and negative for others; the sign persists per-session. Time-anchored sync algorithms must use unchecked-wrap subtraction or sign-aware deltas. **[Community]** <https://phi16.hatenablog.com/entry/2025/01/08/023655>
- **Bigger objects beat many small objects.** Per-serialization overhead is amortized; one packet with a packed array of values is cheaper than many packets with individual values. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/performance/>
- **Bit-pack booleans.** A `bool[]` costs ~1 byte per element. Pack 8 bools into a `byte` (or 32 into an `int`) for 8–32× savings. **[Docs]** <https://creators.vrchat.com/worlds/udon/data-containers/byte-and-bit-operations/>

## See also

- [Sync modes](sync-modes.md)
- [Attributes → UdonSynced](../attributes/udon-synced.md)
- [UdonSharpBehaviour → networking events](../udonsharp-behaviour/networking-events.md)
- [Debugging & network stats](debugging-and-stats.md)
- [References → Community Resources](../references/community-resources.md) — `VRCLockstep`, `UNet` and other RPC/networking libraries.
