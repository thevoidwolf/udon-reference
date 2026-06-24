# Networking

Udon ships its own network model. As an author you do not write socket code or pick replication strategies; instead you declare *what is synced* via `[UdonSynced]` and *who owns it* via the ownership API, then call `RequestSerialization()` (manual) or let the runtime broadcast (continuous).

## Pages

| Page | What's in it |
|---|---|
| [Sync Modes](sync-modes.md) | Continuous vs Manual: when each fires, when to use which. |
| [Ownership](ownership.md) | Per-GameObject owner concept, transfer, `OnOwnershipRequest`. |
| [Custom Network Events](custom-network-events.md) | `SendCustomNetworkEvent`, `NetworkEventTarget`, naming rules. |
| [Serialization](serialization.md) | The packet model: what fits, what gets dropped, how `OnPostSerialization` reports it. |
| [Network IDs](network-ids.md) | Pre-build network ID assignment for scene objects. |
| [VRCObjectSync](object-sync.md) | Built-in Transform + Rigidbody sync component. |
| [VRCObjectPool](object-pool.md) | Built-in object pool for shared spawning. |

## Mental model

1. Every networked GameObject has exactly one **owner** at any time. **[Docs]**
2. Only the owner's writes to `[UdonSynced]` fields propagate.
3. The runtime decides when to push a packet (continuous) or you tell it to (manual).
4. Other clients receive the packet and apply it to their local copy of the synced fields, then fire `OnDeserialization`.
5. Custom events fly through a separate channel (`SendCustomNetworkEvent`) — they don't carry persistent state; they're fire-and-forget RPCs.

There is no concept of "rollback" or "client prediction." The owner is authoritative; non-owners see what the owner sent.

## Choice of sync primitive

| Need | Primitive |
|---|---|
| Make a value visible to all players, snap on change | `[UdonSynced]` field, sync mode `None` |
| Animate a value smoothly across the network | `[UdonSynced(UdonSyncMode.Linear)]` (numeric only) |
| Pickup that physically settles for everyone | `VRCObjectSync` component |
| Spawn an object visible to everyone | `VRCObjectPool` |
| Trigger a one-off action on remote clients | `SendCustomNetworkEvent(target, name)` |
| Per-player saved state across sessions | Player Object + `VRCEnablePersistence` |

## What the network *doesn't* do for you

- **No automatic conflict resolution.** Two clients both writing to the same `[UdonSynced]` field race; whoever owns it last wins.
- **No anti-cheat.** Any client can change ownership and lie. Worlds that need authority (scoreboards, locked doors) must explicitly trust only the master or veto ownership in `OnOwnershipRequest`.
- **No reliability guarantee on `SendCustomNetworkEvent`.** It is best-effort. Important state transitions should be replicated by syncing the state itself, not by sending an event and hoping it arrives.

## See also

- [Attributes → UdonSynced](../attributes/udon-synced.md)
- [Attributes → BehaviourSyncMode](../attributes/behaviour-sync-mode.md)
- [UdonSharpBehaviour → networking events](../udonsharp-behaviour/networking-events.md)
- [Players → Networking class](../players/networking-class.md) — the `VRC.SDKBase.Networking` static.
