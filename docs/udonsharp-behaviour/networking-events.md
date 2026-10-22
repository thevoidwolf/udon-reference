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
| `OnDeserialization(DeserializationResult)` | Non-owner | As above with `sendTime`, `receiveTime` (local `realtimeSinceStartup` clock) and `isFromStorage` (data restored from persistence). **[Docs]** |
| `OnPostSerialization` | Owner | After a serialization attempt completes. `result.success` is `false` if the packet exceeded the byte budget; `result.byteCount` is the bytes sent. **[Docs]** |

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

`OnOwnershipRequest` is your gate against unwanted transfers. Return `false` to veto. Both `requestingPlayer` and `requestedOwner` are provided because `Networking.SetOwner(p, obj)` lets a third party push ownership onto someone else. It runs locally on **both the requester and the current owner**, so base the decision only on state both sides agree on (synced data, player properties) — otherwise you get a desync where the owner silently rejects. **[Docs]**

```csharp
public override bool OnOwnershipRequest(VRCPlayerApi requestingPlayer, VRCPlayerApi requestedOwner) {
    return requestedOwner.isMaster;  // only master may own this
}
```

The official docs advise against using *master* status to gate features (the master is just whoever is left holding unowned objects); for access control prefer the instance owner (`Networking.InstanceOwner`, SDK 3.7.0+) or your own allowlist. **[Docs]**

`OnOwnershipTransferred` fires on every client *after* a transfer commits.

---

## Instance master

```csharp
public override void OnMasterTransferred(VRCPlayerApi newMaster)   // SDK 3.7.0+
```

Fires for everyone when the master changes because the previous master left. `newMaster` is always valid. For the first player in a new instance it fires after `OnPlayerJoined` (transfer from "nobody"). The new master is chosen before `OnPlayerLeft` runs for the old one. Read the current master any time with `Networking.Master` (SDK 3.7.0+). **[Docs]**

---

## Player presence

```csharp
public virtual void OnPlayerJoined(VRCPlayerApi player)
public virtual void OnPlayerLeft(VRCPlayerApi player)
```

These technically aren't "sync" events, but they are network-driven. See [Player events](player-events.md) for the full discussion (especially the "no persistence saves in OnPlayerLeft" rule). `OnPlayerSuspendChanged(VRCPlayerApi)` (SDK 3.7.0+) is the cue to move ownership away from a player whose device went to sleep. **[Docs]**

---

## Receiving custom network events

Not overrides, but the receiving side of `SendCustomNetworkEvent` (see [API methods](api-methods.md#sendcustomnetworkevent)):

- **(SDK 3.8.1+)** Methods meant to be called over the network should carry [`[NetworkCallable]`](../attributes/network-callable.md) (namespace `VRC.SDK3.UdonNetworkCalling`), may take up to 8 syncable parameters, and must be `public`, non-`static`/`virtual`/`override`, non-overloaded, with no return value. The receiving behaviour's sync mode must not be `None`. **[Docs]**
- Inside the handler, `NetworkCalling.CallingPlayer` is the `VRCPlayerApi` that sent the event (`null` outside a network call), and `NetworkCalling.InNetworkCall` tells you whether the current code is running as part of one (it stays `true` for nested calls until the entry function returns). **[Docs]**

```csharp
using VRC.SDK3.UdonNetworkCalling;

[NetworkCallable]
public void RequestReset() {
    VRCPlayerApi sender = NetworkCalling.CallingPlayer;
    if (!Utilities.IsValid(sender) || !sender.isInstanceOwner) return;   // reject everyone else
    DoReset();
}
```

`isInstanceOwner` is always `false` in Group and Public instances and in Build & Test, so real worlds usually check against their own moderator list instead. **[Docs]**

Full model (rate limits, parameter sizes, legacy events): [Networking → custom network events](../networking/custom-network-events.md) and <https://creators.vrchat.com/worlds/udon/networking/events/>.

---

## Network settled

Not an event, but the corresponding check:

```csharp
if (Networking.IsNetworkSettled) { /* safe to use synced state */ }
```

`IsNetworkSettled` returns `true` once all the data in the instance has been deserialized, applied, and is ready for use. **[Docs]** Before that, `[UdonSynced]` fields may still be at their compile-time defaults rather than the current authoritative value. Useful at world load. For persistent data use `OnPlayerRestored` instead.

## See also

- [Networking → sync modes](../networking/sync-modes.md) — Continuous vs Manual and what each one does to event timing.
- [Networking → ownership](../networking/ownership.md) — full ownership transfer model.
- [Attributes → UdonSynced](../attributes/udon-synced.md) — what is and isn't syncable.
