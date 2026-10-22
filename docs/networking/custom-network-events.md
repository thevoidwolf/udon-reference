# Custom Network Events

`SendCustomNetworkEvent` invokes a public method on remote clients. It is the "RPC" of the Udon model.

```csharp
public void SendCustomNetworkEvent(NetworkEventTarget target, string eventName);
// SDK 3.8.1+: up to 8 trailing parameters (see below)
SendCustomNetworkEvent(NetworkEventTarget.All, nameof(PrintMessage), "VRCat", 11);

// SDK 3.8.1+: equivalent static form, can target any UdonBehaviour
NetworkCalling.SendCustomNetworkEvent(behaviour, NetworkEventTarget.All, nameof(PrintMessage), "VRCat", 11);
```

The instance method is inherited from `UdonSharpBehaviour`. Source: `Runtime/UdonSharpBehaviour.cs`. **[Source]** The two forms are identical; `NetworkCalling` lives in `VRC.SDK3.UdonNetworkCalling`. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/events/>

The targeted `UdonBehaviour` can be a different one from the sender, and it may be **disabled**. **[Docs]**

## Target enum

`VRC.Udon.Common.Interfaces.NetworkEventTarget`:

| Value | Effect |
|---|---|
| `All` | Every player in the instance — *including the sender*. **[Docs]** |
| `Owner` | The current owner of the GameObject only. **[Docs]** |
| `Others` | Every player **except** the sender. **(SDK 3.8.1+)** **[Docs]** |
| `Self` | "Loopback": only the sender receives it. Never goes over the network, so it **bypasses all rate limiting** — useful for invoking a parameterized event locally. **(SDK 3.8.1+)** **[Docs]** |

If the sender is among the recipients (`All`, `Self`, or `Owner` when the sender owns the object), the event runs locally **immediately**, like a normal method call, before `SendCustomNetworkEvent` returns. Remote copies are sent asynchronously. **[Docs]**

There is no single-player target. Pass the target's `playerId` as a parameter, send to `All`, and have receivers ignore it unless it matches `Networking.LocalPlayer.playerId`. **[Docs]**

To exclude the sender on pre-3.8.1 SDKs, guard inside the receiver:

```csharp
public void OnGlobalFire() {
    if (Networking.LocalPlayer == _firingPlayer) return;
    PlaySound();
}
```

(`_firingPlayer` would be a synced field.)

## Rules

- **Method must be `public`.** For `[NetworkCallable]` methods it must also return `void`; data can't be returned over the network. **[Docs]**
- **Method names beginning with `_` can't be called over the network** unless the method is explicitly marked `[NetworkCallable]`. The underscore is the official way to keep a public method local-only. **[Docs]**
- **Not replayed for late joiners.** An event runs only for players in the instance at send time. Use synced variables for state. **[Docs]**
- **Queued, not dropped, when rate-limited.** The sending client queues events above the rate limit and sends them when it can. The queue has no size limit, so sending too fast indefinitely clogs *all* Udon networking in the world. **[Docs]** See [Rate limiting](#rate-limiting-sdk-381).
- **Ordering.** Events from one sender arrive in send order across all behaviours, as long as no `[NetworkCallable]` rate limit is hit. There is no ordering guarantee between different senders. A rate-limited event lets other events "skip the queue": with a 3/s limit on `A`, sending `A1 A2 A3 A4 B1` arrives as `A1 A2 A3 B1 A4`. **[Docs]**
- **Rare server-side drops.** If players in one instance are on world versions with different rate limits, the server may silently drop events that exceed the receiver's expected limit. This is the only documented non-malicious drop case. **[Docs]**

## Parameterized network events — `[NetworkCallable]` (SDK 3.8.1+)

VRChat shipped first-class parameterized network events in SDK 3.8.1 (May 22, 2025). **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/events/> · <https://creators.vrchat.com/releases/release-3-8-1/>

```csharp
using VRC.SDK3.UdonNetworkCalling;   // [NetworkCallable], NetworkCalling
using VRC.Udon.Common.Interfaces;    // NetworkEventTarget

[NetworkCallable]                       // default rate limit: 5 events/s
public void Score(int playerIndex, int delta) {
    scores[playerIndex] += delta;
}

[NetworkCallable(maxEventsPerSecond: 20)]  // 1..100
public void Ping() { }

// Caller:
SendCustomNetworkEvent(NetworkEventTarget.All, nameof(Score), 2, 10);
```

Attribute details: [Attributes → NetworkCallable](../attributes/network-callable.md).

**Rules** **[Docs]**:
- In U#, the receiver must be marked **`[NetworkCallable]`**. Parameterless legacy events are the exception (see [Legacy events](#legacy-events-and-security)).
- Must be `public`. Not `static`, `virtual` or `override`. Must return `void`.
- **No member overloading**: no other method with the same name, including ones without `[NetworkCallable]`.
- **Max 8 parameters.** No `params`. No default values.
- Parameter types: any [synced-variable type](https://creators.vrchat.com/worlds/udon/networking/network-details#synced-variables), the same set as `[UdonSynced]` (including arrays and `VRCUrl`).
- Argument types passed to `SendCustomNetworkEvent` **must match** the declared parameter types, or the send fails.
- Passing `null` gives the receiver `default(T)` for the declared type (e.g. `0` for `int`, `null` for `string`).
- The **receiving** behaviour must not use `BehaviourSyncMode.None`.
- All four `NetworkEventTarget` values are supported.

### Size limits and event splitting

All items in this section: **[Docs]**.

- **Max 16 KB** of parameter data per event, all parameters combined. A single parameter has no limit beyond that.
- Events with more than **1024 bytes** of parameter data are split into several internal events. The receiver reassembles them and gets one call, but each piece counts against the rate limit and shows up in `GetQueuedEvents`. For example, a 2048-byte call under a 2/s limit gives an effective 1 call/s.
- Total outgoing data is hard-capped at **~18 KB/s**, including all overhead. In practice expect at most ~8–10 KB/s.
- Sizes are the encoded parameter bytes, without headers: `int` 4, `Vector3` 12, `string` UTF-8 bytes, `char[]` 2 per char (UTF-16), `string[]` UTF-8 bytes + 4 per entry.

### Rate limiting (SDK 3.8.1+)

- Default: **5 events/second per event**. Configurable with `[NetworkCallable(maxEventsPerSecond: X)]`, where `X` is 1–100. **[Docs]**
- Official guidance is to set this **as low as you can**, so malicious clients can't abuse your events. **[Docs]**
- The limit applies on the sending client (as queuing) and on the server (protection only). Events that run locally for the sender are never rate-limited, so remote copies may still be queued after the local copy has run. **[Docs]**
- There is also a global, non-configurable limit of about **100 events/s** across the whole world. It is dynamic and may change. **[Docs]**
- All rate limiting is best-effort and may not match the configured values exactly. **[Docs]**

### Congestion monitoring (SDK 3.8.1+)

All items in this section: **[Docs]**.

| Member | Description |
|---|---|
| `int NetworkCalling.GetQueuedEvents(udonBehaviour, eventName)` | Number of events of that name currently queued for sending. Normally between 0 and your rate limit; above it means you are sending too fast. |
| `int NetworkCalling.GetAllQueuedEvents()` | Number of events queued across the whole world. A non-zero value alone doesn't mean the network is bad. |
| `Networking.IsClogged` | Overall "too much outgoing data" flag. Excessive event sending also affects it. |

```csharp
queueStatus.text = $"Queue: {NetworkCalling.GetAllQueuedEvents()}"
                 + $"\nThis event: {NetworkCalling.GetQueuedEvents(this, nameof(Score))}"
                 + $"\nClogged: {Networking.IsClogged}";
```

### Identifying the sender (SDK 3.8.1+)

All items in this section: **[Docs]**.

| Property | Description |
|---|---|
| `VRCPlayerApi NetworkCalling.CallingPlayer { get; }` | The player who sent the network call currently running. `null` when you are not inside a network call. Works for legacy events too. |
| `bool NetworkCalling.InNetworkCall { get; }` | `true` while running as part of a network call. Shortcut for `Utilities.IsValid(NetworkCalling.CallingPlayer)`. It stays set until the entry method returns, including in methods or other scripts called from it. |

```csharp
[NetworkCallable]
public void RequestReset() {
    if (NetworkCalling.CallingPlayer != Networking.InstanceOwner) return; // trust check
    ResetGame();
}
```

`CallingPlayer` replaces the old pattern of syncing a "who fired this" field alongside the event.

### Legacy events and security

Before 3.8.1, any `public` method not starting with `_` could be called over the network. For backwards compatibility, **parameterless** public methods without `[NetworkCallable]` (and not starting with `_`) can still be called this way, but this is not recommended. In Udon Graph, a custom-event node is "legacy" when `MaxEventsPerSecond` is `0`. **[Docs]**

**Component vs GameObject targeting.** A `[NetworkCallable]` call targets the *specific* `UdonBehaviour` component. A legacy call (no parameters, no `[NetworkCallable]`) uses GameObject semantics, like `GameObject.SendMessage`: it runs the method on **every** `UdonBehaviour` on that GameObject. Component targeting relies on component order, so don't `Destroy` components on GameObjects with networked `UdonBehaviour`s; the result is undefined. **[Docs]**

### Legacy patterns (pre-3.8.1)

Pre-3.8.1 worlds had to encode parameters in synced fields or string payloads.

Older worlds you'll see in the wild use these patterns. They still work, but `[NetworkCallable]` parameters and `NetworkCalling.CallingPlayer` are now preferred:

- **Synced-var-then-event** — set an `[UdonSynced]` field, call `RequestSerialization()`, then `SendCustomNetworkEvent`. Receivers read the field. Fragile due to the event-before-var race. **[Community]** <https://udonsharp.docs.vrchat.com/networking-tips-&-tricks/>
- **`OnDeserialization` dispatcher** — set state, serialize, and let `OnDeserialization` invoke the right method based on which fields changed. Cleaner for state-coupled actions.
- **Enumerated handlers** — `SetXTo0`, `SetXTo1`, ... called by name. Crude but parameterless.
- **Binary-buffer RPC** — community libraries `UNet` (Xytabich) provided typed serialization, ack/retry, and per-player targeting before `[NetworkCallable]` existed. Still useful for >8-arg payloads or guaranteed-ordered streams. **[Community]** <https://github.com/Xytabich/UNet> (archived; last push Jun 2023)

## Patterns

### "Tell everyone to react"

```csharp
public override void Interact() {
    if (!Networking.IsOwner(gameObject))
        Networking.SetOwner(Networking.LocalPlayer, gameObject);
    SendCustomNetworkEvent(NetworkEventTarget.All, nameof(OnBuzzer));
}
[NetworkCallable]
public void OnBuzzer() {
    audioSource.PlayOneShot(buzzer);  // fires on every client including sender
}
```

### "Tell the owner to do something"

```csharp
public void RequestSpawn() {
    SendCustomNetworkEvent(NetworkEventTarget.Owner, nameof(DoSpawn));
}
[NetworkCallable]
public void DoSpawn() {
    if (!Networking.IsOwner(gameObject)) return;   // defensive
    GameObject obj = pool.TryToSpawn();
}
```

### Sending a value with the event (SDK 3.8.1+)

```csharp
public void Broadcast(string message) {
    SendCustomNetworkEvent(NetworkEventTarget.All, nameof(OnMessage), message);
}
[NetworkCallable]
public void OnMessage(string message) {
    Debug.Log(message);
}
```

No ownership transfer or synced field is needed. Late joiners still won't see past messages.

### Replicating parameters via synced fields (legacy)

```csharp
[UdonSynced] string _pendingMessage;

public void Broadcast(string message) {
    if (!Networking.IsOwner(gameObject))
        Networking.SetOwner(Networking.LocalPlayer, gameObject);
    _pendingMessage = message;
    RequestSerialization();
    SendCustomNetworkEvent(NetworkEventTarget.All, nameof(OnMessage));
}
public void OnMessage() {
    Debug.Log(_pendingMessage);
}
```

(Note: this is a brittle pattern because the event can race the sync packet. Prefer driving the visible action from `OnDeserialization` of the synced field whenever possible.)

### "Run this everywhere with one call"

```csharp
SendCustomNetworkEvent(NetworkEventTarget.All, nameof(Step));
```

The sender runs `Step` immediately, before the call returns. Remote clients run it when the packet arrives. **[Docs]** This is *not* a synchronized "same network frame" for everyone. If remote timing matters, send a timestamp parameter (e.g. from `Networking.GetServerTimeInMilliseconds()`) and schedule locally.

## Deferred local events

For purely local timers, use the delayed-event helpers (not network):

```csharp
SendCustomEventDelayedSeconds(nameof(Cleanup), 5f);
SendCustomEventDelayedFrames(nameof(Settle), 2);
```

See [UdonSharpBehaviour → API methods](../udonsharp-behaviour/api-methods.md).

## Gotchas

- **A behaviour with `BehaviourSyncMode.None` can't *receive* network events.** **[Docs]** Use `NoVariableSync` if you need events but no synced fields.
- A networked event sent right before a player leaves may not be observed by others. Don't rely on "last-words" packets.
- Event name typos compile fine (you pass a string). Use `nameof(...)` to catch renames at compile time. Event names are case-sensitive. **[Docs]**
- A parameter type mismatch between caller and `[NetworkCallable]` signature makes the send fail. **[Docs]**

## Community libraries

- **`VRCLockstep`** (JanSharp) — Deterministic input-action layer that sidesteps RPC entirely: every client runs the same ticks in the same order from a shared action stream. Late joiners receive a byte-array state snapshot. Eliminates ownership race conditions for "large mutating state" worlds. **[Community]** <https://github.com/JanSharp/VRCLockstep>
- **`UNet`** (Xytabich) — Reliable binary RPC layer with `ByteBufferWriter`/`Reader`, ack/retry, sequencing, per-player targeting. Predates `[NetworkCallable]`; useful when you need >8 args. **[Community]** <https://github.com/Xytabich/UNet> (archived; last push Jun 2023)
- **`SimpleNetwork`** (tutinoco) — Typed-networking wrapper. **[Community]** <https://github.com/tutinoco/SimpleNetwork>

## See also

- [UdonSharpBehaviour → API methods](../udonsharp-behaviour/api-methods.md)
- [Attributes → NetworkCallable](../attributes/network-callable.md)
- [Sync modes](sync-modes.md) — `BehaviourSyncMode.NoVariableSync` rules.
- [Debugging & network stats](debugging-and-stats.md) — `VRC.SDK3.Network.Stats`, debug views.
- [Ownership](ownership.md) — `Owner` target depends on it.
- [References → Community Resources](../references/community-resources.md) — full networking-libraries index.
