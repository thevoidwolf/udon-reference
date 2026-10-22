# Networking

Udon ships its own network model. As an author you do not write socket code or pick replication strategies; instead you declare *what is synced* via `[UdonSynced]` and *who owns it* via the ownership API, then call `RequestSerialization()` (manual) or let the runtime broadcast (continuous).

## Pages

| Page | What's in it |
|---|---|
| [Sync Modes](sync-modes.md) | Continuous vs Manual: when each fires, when to use which. |
| [Ownership](ownership.md) | Per-GameObject owner concept, transfer, `OnOwnershipRequest`, master vs instance owner (`Networking.Master`, `InstanceOwner`, `OnMasterTransferred`). |
| [Custom Network Events](custom-network-events.md) | `SendCustomNetworkEvent`, `NetworkEventTarget`, `[NetworkCallable]` parameters, rate limits, `NetworkCalling`. |
| [Serialization](serialization.md) | The packet model: what fits, what gets dropped, how `OnPostSerialization` reports it. |
| [Network IDs](network-ids.md) | Network ID assignment, world-version compatibility, Network ID Import/Export Utility. |
| [Debugging & Network Stats](debugging-and-stats.md) | `VRC.SDK3.Network.Stats` (SDK 3.8.1+), debug views 4/6/8/9, official troubleshooting checklist. |
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
| One-off action that carries data | `[NetworkCallable]` method + `SendCustomNetworkEvent(target, name, args…)` (SDK 3.8.1+) |
| Per-player saved state across sessions | Player Object + `VRCEnablePersistence` |

## What the network *doesn't* do for you

- **No automatic conflict resolution.** Two clients both writing to the same `[UdonSynced]` field race; whoever owns it last wins.
- **No anti-cheat.** Any client can change ownership and lie. Worlds that need authority (scoreboards, locked doors) must veto ownership in `OnOwnershipRequest` and check who sent events with `NetworkCalling.CallingPlayer` (SDK 3.8.1+). Gate privileged actions on the **instance owner** or your own moderator list, not the master: VRChat says master status should not be used for access control. **[Docs]**
- **No persistence for `SendCustomNetworkEvent`.** Events reach only the players present at send time and are never replayed for late joiners. **[Docs]** Events above the rate limit are queued, not dropped, but a client can still leave before receiving one. Replicate important state transitions by syncing the state itself.

## See also

- [Attributes → UdonSynced](../attributes/udon-synced.md)
- [Attributes → BehaviourSyncMode](../attributes/behaviour-sync-mode.md)
- [UdonSharpBehaviour → networking events](../udonsharp-behaviour/networking-events.md)
- [Players → Networking class](../players/networking-class.md) — the `VRC.SDKBase.Networking` static.
- [Attributes → NetworkCallable](../attributes/network-callable.md)
- Official: <https://creators.vrchat.com/worlds/udon/networking/>
