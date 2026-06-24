# Custom Network Events

`SendCustomNetworkEvent` invokes a public method on remote clients. It is the "RPC" of the Udon model.

```csharp
public void SendCustomNetworkEvent(NetworkEventTarget target, string eventName);
// SDK 3.8.1+: parameterized overload (see below)
```

Inherited from `UdonSharpBehaviour`. Source: `Runtime/UdonSharpBehaviour.cs`. **[Source]**

## Target enum

`VRC.Udon.Common.Interfaces.NetworkEventTarget`:

| Value | Effect |
|---|---|
| `All` | Every player in the instance — *including the sender*. |
| `Owner` | The current owner of the GameObject only. |
| `Others` | Every player **except** the sender. **(SDK 3.8.1+)** **[Docs]** |
| `Self` | Local-only invocation. Mostly useful for symmetry in shared code. **(SDK 3.8.1+)** **[Docs]** |

To exclude the sender on older SDK versions, guard inside the receiver:

```csharp
public void OnGlobalFire() {
    if (Networking.LocalPlayer == _firingPlayer) return;
    PlaySound();
}
```

(`_firingPlayer` would be a synced field.)

## Rules

- **Method must be `public`.** Return values are discarded.
- **Method names beginning with `_` are non-networkable** — calls to them are silently dropped. This is the standard convention for "local-only" helpers.
- **Best-effort delivery.** The packet may be dropped under network congestion. Use synced state for things that must replicate.
- **Rate-limited.** Sending events rapid-fire can cause the runtime to drop them. Throttle your callers.

## Parameterized network events — `[NetworkCallable]` (SDK 3.8.1+)

VRChat shipped first-class parameterized network events in SDK 3.8.1 (2024). **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/events/>

```csharp
[NetworkCallable]
public void Score(int playerIndex, int delta) {
    scores[playerIndex] += delta;
}

// Caller:
SendCustomNetworkEvent(NetworkEventTarget.All, nameof(Score), 2, 10);
```

**Rules:**
- Receiver method must be marked **`[NetworkCallable]`**.
- Must be `public`, non-static. No `virtual` / `override`. No `params`. No default parameter values.
- **Max 8 parameters.**
- Parameter types must be Udon-syncable primitives (`int`, `string`, `float`, `bool`, etc., plus the Unity value types — same set as `[UdonSynced]`).
- **Max total event size 16 KB.** Events larger than 1024 bytes are split internally and count against rate-limit budget more heavily.
- All four `NetworkEventTarget` values supported.

Pre-3.8.1 worlds had to encode parameters via synced fields or string-encoded payloads — see "Legacy patterns" below.

### Legacy patterns (pre-3.8.1)

Older worlds you'll see in the wild use these patterns; they still work but `[NetworkCallable]` is now preferred:

- **Synced-var-then-event** — set an `[UdonSynced]` field, call `RequestSerialization()`, then `SendCustomNetworkEvent`. Receivers read the field. Fragile due to the event-before-var race. **[Community]** <https://udonsharp.docs.vrchat.com/networking-tips-&-tricks/>
- **`OnDeserialization` dispatcher** — set state, serialize, and let `OnDeserialization` invoke the right method based on which fields changed. Cleaner for state-coupled actions.
- **Enumerated handlers** — `SetXTo0`, `SetXTo1`, ... called by name. Crude but parameterless.
- **Binary-buffer RPC** — community libraries `UNet` (Xytabich) provided typed serialization, ack/retry, and per-player targeting before `[NetworkCallable]` existed. Still useful for >8-arg payloads or guaranteed-ordered streams. **[Community]** <https://github.com/Xytabich/UNet> (archived Apr 2026)

## Patterns

### "Tell everyone to react"

```csharp
public override void Interact() {
    if (!Networking.IsOwner(gameObject))
        Networking.SetOwner(Networking.LocalPlayer, gameObject);
    SendCustomNetworkEvent(NetworkEventTarget.All, nameof(OnBuzzer));
}
public void OnBuzzer() {
    audioSource.PlayOneShot(buzzer);  // fires on every client including sender
}
```

### "Tell the owner to do something"

```csharp
public void RequestSpawn() {
    SendCustomNetworkEvent(NetworkEventTarget.Owner, nameof(DoSpawn));
}
public void DoSpawn() {
    if (!Networking.IsOwner(gameObject)) return;   // defensive
    GameObject obj = pool.TryToSpawn();
}
```

### Replicating parameters via synced fields

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

### "Run this *on me*, but go through the network for ordering"

```csharp
SendCustomNetworkEvent(NetworkEventTarget.All, nameof(Step));
```

Sometimes useful for animations: all clients (including sender) see the event at the same network frame, which keeps ordering consistent.

## Deferred local events

For purely local timers, use the delayed-event helpers (not network):

```csharp
SendCustomEventDelayedSeconds(nameof(Cleanup), 5f);
SendCustomEventDelayedFrames(nameof(Settle), 2);
```

See [UdonSharpBehaviour → API methods](../udonsharp-behaviour/api-methods.md).

## Gotchas

- **`BehaviourSyncMode.None` cannot send networked events.** Use `NoVariableSync` if you need events but no synced fields.
- A networked event sent right before a player leaves may not be observed by others. Don't rely on "last-words" packets.
- Event name typos compile fine (you pass a string). Use `nameof(...)` to catch renames at compile time.

## Community libraries

- **`VRCLockstep`** (JanSharp) — Deterministic input-action layer that sidesteps RPC entirely: every client runs the same ticks in the same order from a shared action stream. Late joiners receive a byte-array state snapshot. Eliminates ownership race conditions for "large mutating state" worlds. **[Community]** <https://github.com/JanSharp/VRCLockstep>
- **`UNet`** (Xytabich) — Reliable binary RPC layer with `ByteBufferWriter`/`Reader`, ack/retry, sequencing, per-player targeting. Predates `[NetworkCallable]`; useful when you need >8 args. **[Community]** <https://github.com/Xytabich/UNet> (archived Apr 2026)
- **`SimpleNetwork`** (tutinoco) — Typed-networking wrapper. **[Community]** <https://github.com/tutinoco/SimpleNetwork>

## See also

- [UdonSharpBehaviour → API methods](../udonsharp-behaviour/api-methods.md)
- [Sync modes](sync-modes.md) — `BehaviourSyncMode.NoVariableSync` rules.
- [Ownership](ownership.md) — `Owner` target depends on it.
- [References → Community Resources](../references/community-resources.md) — full networking-libraries index.
