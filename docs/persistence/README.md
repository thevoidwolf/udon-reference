# Persistence

VRChat worlds can save per-player data that survives session boundaries: scores, inventories, preferences, last position, currency, unlocks. Two systems exist:

- **PlayerData** — flat key-value store per player per world.
- **Player Objects** — GameObjects automatically instantiated per player whose `[UdonSynced]` fields can persist if `VRCEnablePersistence` is added.

Source: <https://creators.vrchat.com/worlds/udon/persistence/>. **[Docs]** Introduced in SDK 3.7.4 (Nov 21, 2024).

## Pages

| Page | Topic |
|---|---|
| [PlayerData](player-data.md) | Static API for key-value storage, type-safe accessors. |
| [Player Objects](player-objects.md) | Per-player GameObject instantiation with persistent state. |

## Hard rules

- Data is **scoped per player per world**. Cannot be shared between players, instances of different worlds, or worlds that aren't the same world ID.
- Storage limit: **100 KB compressed** per player per world for each subsystem (so up to 200 KB total if you use both). Highly compressible data can exceed 300 KB raw. Saves that would exceed the limit fail and VRChat logs an error. **[Docs]**
- Query usage at runtime with `Networking.GetPlayerDataStorageUsage` / `GetPlayerObjectStorageUsage`, `Get*StorageLimit` and `RequestStorageUsageUpdate` → `OnPersistenceUsageUpdated`; react to `OnPlayerDataStorageWarning` / `Exceeded` and `OnPlayerObjectStorageWarning` / `Exceeded`. (PlayerData storage API: SDK 3.10.0+.) **[Docs]** See [PlayerData → Storage information](player-data.md#storage-information-sdk-3100) and [Player Objects → Storage information](player-objects.md#storage-information).
- **You cannot save persistence from inside the local player's `OnPlayerLeft`.** Save before they leave. **[Docs]**
- **You cannot delete individual keys.** No `RemoveKey` / `ClearAll` API. Players can reset their own data (per world or all worlds) in-client via Settings → Debug → User Data, or on vrchat.com under the Persistent Data section of their profile settings — irreversibly; world authors cannot. **[Docs]**
- All persistence reads and writes should wait until the `OnPlayerRestored` event (a U# override, not a `PlayerData` member) fires for that player — otherwise you may read stale data or have your writes overwritten when the server's copy arrives. It fires even when the player has no save data. **[Docs]**
- **ClientSim** (SDK 3.10.5+) no longer tries to save simulated changes to player persistence. **[Docs]**

## Choosing between PlayerData and Player Objects

| Need | Choice |
|---|---|
| A few primitive values per player (score, currency, settings flags) | PlayerData |
| Rich per-player state (inventory list, multiple linked values, position history) | Player Object with `[UdonSynced]` fields |
| Need to mutate per-player state in response to interaction with other players' state | Player Object — gives you a per-player UdonBehaviour to host the logic |
| Need to share read access to one player's data with another player | PlayerData — `GetX(player, key)` accepts any `VRCPlayerApi`, not just the local one |

VRChat's official recommendation: **prefer Player Objects** for most use cases. PlayerData re-sends *all* of a player's data on any single change, so a chatty key/value workload can strain bandwidth. **[Docs]**

## Cross-world persistence (no native support)

PlayerData and Player Objects are **per-world**. The community-maintained workaround is **NUSaveState** (Nestorboy) — encodes save data into avatar parameters that the player's account carries between worlds; reads it back via finger-bone rotations. The only practical "shared profile across worlds" solution. **[Community]** <https://github.com/Nestorboy/NUSaveState>

| Method | Status |
|---|---|
| `NUSaveState` data-avatar hack | Last release v1.6.0 (tagged Jan 2024), no commits since — dormant but still the only real cross-world solution. **[Community]** |
| String-loaded "profile" via `VRCStringDownloader` + backend (Google Apps Script / Cloudflare Worker behind a `*.github.io` or `*.vrcdn.cloud` redirector) | Read-only, bandwidth-limited. Used for global leaderboards. **[Community]** |
| Manual export codes (base64 string the player copies between worlds) | Used by some game-jam worlds. Painful UX. **[Community]** |

## Historical context

Before SDK 3.7.4 (Nov 21, 2024), VRChat had **no** native persistence. The community filled the gap with:

- **NUSaveState** (above) — the dominant pre-PlayerData solution.
- **CyanPlayerObjectPool** — provided per-player object identity; archived Oct 11, 2024, just ahead of `VRCPlayerObject` shipping in 3.7.4. **[Community]** <https://github.com/CyanLaser/CyanPlayerObjectPool>
- **SDK2 `vrc_panorama` URL tricks** — fully obsolete.
- **Avatar-parameter signaling** — adjacent to NUSaveState's mechanism; some worlds detected VIP status via finger pose.

## See also

- [Networking](../networking/README.md) — persistence rides on top of the normal sync model.
- [Loading](../loading/README.md) — external data loading is a different system.
- [References → Community Resources](../references/community-resources.md) — NUSaveState, persistence libraries.
