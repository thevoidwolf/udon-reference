# Debugging & Network Stats

Tools for seeing what your world's networking is actually doing: in-client debug views, and the `VRC.SDK3.Network.Stats` API that lets Udon read the same numbers (SDK 3.8.1+).

## Network Stats API (SDK 3.8.1+)

SDK 3.8.1 exposed network debug information to Udon through the static class `VRC.SDK3.Network.Stats`. VRChat describes it as meant for advanced networking creators. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-stats/> · <https://creators.vrchat.com/releases/release-3-8-1/>

### Global statistics

All items in this section: **[Docs]**.

| Name | Type | Description |
|---|---|---|
| `ThroughputPercentage` | `float` | Running average of the allowed outgoing throughput currently in use. |
| `RoundTripVariance` | `float` | Statistical variance of round-trip time to VRChat's servers. |
| `BytesInMax` / `BytesOutMax` | `float` | Max bytes received / sent in one second. |
| `BytesInAverage` / `BytesOutAverage` | `float` | Running average of bytes received / sent per second. |
| `HitchesPerNetworkTick` | `float` | Running average of recorded missing samples. |
| `Suffering` | `float` | Measure of how many outbound messages are queued. Also shown in debug view 4. |
| `TimeInRoom` | `float` | Time spent in the current instance. |

### Per-GameObject / per-player statistics

All items in this section: **[Docs]**.

These return the default value for objects that aren't networked.

| Name | Type | Description |
|---|---|---|
| `UpdateInterval` | `float` | Running average time between sends for this object. |
| `ReceiveInterval` | `float` | Running average time between receipts for this object. |
| `FinalDelay` | `float` | Sync time adjustment for the object. |
| `Group` | `int` | Internal proximity group the object is batched with. |
| `GroupDelay` | `float` | Running average sync adjustment for the object's group. |
| `Sleeping` | `bool` | `true` if the object is at rest and not sending or receiving. |
| `Size` | `int` | Bytes in the most recent message for this object. |
| `BytesPerSecondAverage` | `float` | Running average throughput for the object. |
| `TotalBytes` | `int` | Total data used by the object. |
| `ReliableEventsInOutboundQueue` | `int` | Manual-sync and other reliable events queued for the object. |
| `LastSendTime` / `LastReceiveTime` | `float` | Last time a message was sent / received for the object. |

The official page lists names and types only. It doesn't give the call shape for per-object stats, such as whether they take a `GameObject` or `VRCPlayerApi` argument. Check the Udon class exposure tree or your IDE before relying on a specific overload. **[Unverified]** call signatures.

## Other runtime signals

| API | Use |
|---|---|
| `Networking.IsClogged` | `true` when there is too much outgoing data. Back off. **[Docs]** |
| `Networking.IsNetworkSettled` | `true` once all instance data has been deserialized and applied. **[Docs]** |
| `Networking.SimulationTime(...)` | How far back an object or player is simulated. `Time.realtimeSinceStartup - SimulationTime(player)` approximates that player's current latency. **[Docs]** |
| `NetworkCalling.GetQueuedEvents` / `GetAllQueuedEvents` | Outgoing network-event queue depth (SDK 3.8.1+). See [Custom Network Events](custom-network-events.md#congestion-monitoring-sdk-381). **[Docs]** |
| `OnPostSerialization(SerializationResult)` | `success` and `byteCount` for each send. See [Serialization](serialization.md). **[Docs]** |

## In-client debug views

Open the debug menu with the "Toggle Debug UI" button at the bottom of the Quick Menu Settings page, or with `Right Shift` + `~` + `<number>`. **[Docs]** <https://creators.vrchat.com/worlds/udon/world-debug-views/>

| View | Shows |
|---|---|
| **4 (Players)** | Per-player master flag, group, send interval (`Intrvl`) and target delay (`Fnl D`), plus global network state at the top, such as **Suffering** (whether Udon is sending too much). |
| **6 (Net Objects)** | Every networked object: owner `playerId`, group, sleeping, delay, **Size** (bytes per serialization), **Bps**, time since last send, interval. |
| **8 (Network Object Info overlay)** | Per-object stat panel drawn on top of each synced object. |
| **9 (Player Info overlay)** | The same kind of panel at each player's feet. |

By default only the world creator can open view 6, the Audio Sources page and the world overlays. Other users can use them if **World Debugging** is enabled in the world's settings on the VRChat website; they have to rejoin after you enable it. **[Docs]** View 6 can slow the client down in worlds with many objects. **[Docs]**

## Common issues (official checklist)

All items in this section: **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/debugging/>.

- **Object sync not working**: missing `VRCObjectSync`, ownership not transferred, or a manual behaviour that never calls `RequestSerialization()`.
- **Late joiners see the wrong state**: state was sent through events instead of synced variables.
- **Ownership conflicts**: several scripts transfer ownership at once, or `OnOwnershipRequest` doesn't check anything.

## See also

- [Serialization](serialization.md) — byte budgets and `SerializationResult`.
- [Custom Network Events](custom-network-events.md) — event rate limits and queues.
- [Ownership](ownership.md)
