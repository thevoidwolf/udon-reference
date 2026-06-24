# Networking Events

The subset of [Lifecycle Events](lifecycle-events.md) that interact with the network. Collected here for easy reference. Signatures from `Runtime/UdonSharpBehaviour.cs`. **[Source]**

---

## Synchronization

```csharp
public virtual void OnPreSerialization()
public virtual void OnDeserialization()
public virtual void OnDeserialization(VRC.Udon.Common.DeserializationResult result)
public virtual void OnPostSerialization(VRC.Udon.Common.SerializationResult result)
```

| Event | Fires on | When |
|---|---|---|
| `OnPreSerialization` | Owner | Just before a packet is built from `[UdonSynced]` fields. |
| `OnDeserialization()` | Non-owner | Just after a packet has been applied to local `[UdonSynced]` fields. |
| `OnDeserialization(DeserializationResult)` | Non-owner | As above with timing metadata (`sendTime`, `receiveTime`). |
| `OnPostSerialization` | Owner | After a serialization attempt completes. `result.success` is `false` if the packet exceeded the byte budget. |

**Pattern: derived state.** Compute synced fields from gameplay state in `OnPreSerialization`, then apply them to visuals in `OnDeserialization`:

```csharp
[UdonSynced] int _heldItemId;
public GameObject[] itemPrefabs;

public override void OnPreSerialization() {
    _heldItemId = currentItemId;
}

public override void OnDeserialization() {
    ShowItem(_heldItemId);   // runs on remote clients
}
```

`OnDeserialization` does **not** fire on the owner. If you want the same code path on owner and non-owner, call your apply method directly after modifying the synced field.

---

## Ownership

```csharp
public virtual bool OnOwnershipRequest(VRCPlayerApi requestingPlayer, VRCPlayerApi requestedOwner) => true
public virtual void OnOwnershipTransferred(VRCPlayerApi player)
```

`OnOwnershipRequest` is your gate against unwanted transfers. Return `false` to veto. Both `requestingPlayer` and `requestedOwner` are provided because `Networking.SetOwner(p, obj)` lets a third party push ownership onto someone else.

```csharp
public override bool OnOwnershipRequest(VRCPlayerApi requestingPlayer, VRCPlayerApi requestedOwner) {
    return requestedOwner.isMaster;  // only master may own this
}
```

`OnOwnershipTransferred` fires on every client *after* a transfer commits.

---

## Player presence

```csharp
public virtual void OnPlayerJoined(VRCPlayerApi player)
public virtual void OnPlayerLeft(VRCPlayerApi player)
```

These technically aren't "sync" events, but they are network-driven. See [Player events](player-events.md) for the full discussion (especially the "no persistence saves in OnPlayerLeft" rule).

---

## Network settled

Not an event, but the corresponding check:

```csharp
if (Networking.IsNetworkSettled) { /* safe to use synced state */ }
```

`IsNetworkSettled` flips to `true` once the client has finished initial state reconciliation with the instance. Before that, `[UdonSynced]` fields may still be at their compile-time defaults rather than the current authoritative value. Useful at world load. **[Docs]**

## See also

- [Networking → sync modes](../networking/sync-modes.md) — Continuous vs Manual and what each one does to event timing.
- [Networking → ownership](../networking/ownership.md) — full ownership transfer model.
- [Attributes → UdonSynced](../attributes/udon-synced.md) — what is and isn't syncable.
