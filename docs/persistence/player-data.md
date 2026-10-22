# PlayerData

Static class `VRC.SDK3.Persistence.PlayerData` (SDK 3.7.4+, Nov 2024). Key-value storage scoped per player per world. **[Docs]** <https://creators.vrchat.com/worlds/udon/persistence/player-data>

All UdonBehaviours in a world share the same PlayerData. Writes sync automatically (independent of any behaviour's sync mode); remote users receive only the final value of rapid changes, and **any** change re-sends all of the local player's PlayerData — roughly the bandwidth of one Manual-sync behaviour. **[Docs]**

## Setters (local player only)

All have signature `void Set<Type>(string key, <Type> value)`. There is no player parameter: they always write the **local** player's data; remote players' data cannot be set. You can overwrite a key with a value of a different type. **[Docs]**

```csharp
PlayerData.SetString(string key, string value);
PlayerData.SetBool  (string key, bool value);
PlayerData.SetInt   (string key, int value);
PlayerData.SetUInt  (string key, uint value);
PlayerData.SetLong  (string key, long value);
PlayerData.SetULong (string key, ulong value);
PlayerData.SetFloat (string key, float value);
PlayerData.SetDouble(string key, double value);
PlayerData.SetSByte (string key, sbyte value);
PlayerData.SetByte  (string key, byte value);
PlayerData.SetShort (string key, short value);
PlayerData.SetUShort(string key, ushort value);
PlayerData.SetBytes (string key, byte[] value);
PlayerData.SetVector2   (string key, Vector2 value);
PlayerData.SetVector3   (string key, Vector3 value);
PlayerData.SetVector4   (string key, Vector4 value);
PlayerData.SetQuaternion(string key, Quaternion value);
PlayerData.SetColor     (string key, Color value);
PlayerData.SetColor32   (string key, Color32 value);
```

No success indicator. **Cannot save from the local player's `OnPlayerLeft`.** **[Docs]**

## Getters (any player)

```csharp
PlayerData.GetString    (VRCPlayerApi player, string key) → string
PlayerData.GetBool      (VRCPlayerApi player, string key) → bool
PlayerData.GetInt       (VRCPlayerApi player, string key) → int
PlayerData.GetUInt      (VRCPlayerApi player, string key) → uint
PlayerData.GetLong      (VRCPlayerApi player, string key) → long
PlayerData.GetULong     (VRCPlayerApi player, string key) → ulong
PlayerData.GetFloat     (VRCPlayerApi player, string key) → float
PlayerData.GetDouble    (VRCPlayerApi player, string key) → double
PlayerData.GetSByte     (VRCPlayerApi player, string key) → sbyte
PlayerData.GetByte      (VRCPlayerApi player, string key) → byte
PlayerData.GetShort     (VRCPlayerApi player, string key) → short
PlayerData.GetUShort    (VRCPlayerApi player, string key) → ushort
PlayerData.GetBytes     (VRCPlayerApi player, string key) → byte[]
PlayerData.GetVector2   (VRCPlayerApi player, string key) → Vector2
PlayerData.GetVector3   (VRCPlayerApi player, string key) → Vector3
PlayerData.GetVector4   (VRCPlayerApi player, string key) → Vector4
PlayerData.GetQuaternion(VRCPlayerApi player, string key) → Quaternion
PlayerData.GetColor     (VRCPlayerApi player, string key) → Color
PlayerData.GetColor32   (VRCPlayerApi player, string key) → Color32
```

A missing key returns the type's **default value** (e.g. `GetInt` → `0`); so does reading a key with the wrong type. Use `TryGet*` or `HasKey` / `TryGetType` when you need to tell "missing" from "default". **[Docs]** (Earlier revisions of this page claimed getters throw — they don't.)

## TryGetters

```csharp
bool PlayerData.TryGetString(VRCPlayerApi player, string key, out string value)
bool PlayerData.TryGetInt   (VRCPlayerApi player, string key, out int    value)
// ... same pattern for every type
```

The docs list the outputs as `<type> value, bool success` (Udon Graph form); in U# `value` is the `out` parameter and `success` is the return value — `true` if the key existed and the type matched. **[Docs]**

```csharp
if (PlayerData.TryGetInt(player, "score", out int score)) {
    Debug.Log($"score: {score}");
}
```

## Query methods

```csharp
bool HasKey(VRCPlayerApi player, string key);
System.Type GetType(VRCPlayerApi player, string key);
bool TryGetType(VRCPlayerApi player, string key, out System.Type t);
```

`GetType` returns the type currently stored under the key. `TryGetType` returns `false` if the key doesn't exist. **[Docs]**

## Storage information (SDK 3.10.0+)

The docs place these on `VRC.SDKBase.Networking`, alongside the Player Object storage methods — **not** on `PlayerData`. **[Docs]**

```csharp
int  Networking.GetPlayerDataStorageLimit();                    // limit in bytes (100 KB per the persistence overview)
int  Networking.GetPlayerDataStorageUsage(VRCPlayerApi target); // last *computed* usage for target
void Networking.RequestStorageUsageUpdate();                    // recompute local player's PlayerData AND PlayerObject usage
```

Results of `RequestStorageUsageUpdate` arrive via `OnPersistenceUsageUpdated`. Usage values can go stale; the docs ask you **not** to call `RequestStorageUsageUpdate` frequently. **[Docs]** The exact limit value returned (e.g. `102400`) and behaviour before data has loaded are not documented. **[Unverified]**

## Events

These are ordinary Udon events — in U# you `override` them on your `UdonSharpBehaviour`. (Earlier revisions of this page described them as static C# `event`s on `PlayerData`; U# can't subscribe to C# events.)

| Event (U# override) | Fires |
|---|---|
| `OnPlayerRestored(VRCPlayerApi player)` | Once per player after all their persistent data (PlayerData **and** PlayerObjects) has loaded — even if they have no save data. **Wait for this before reading or writing.** |
| `OnPlayerDataUpdated(VRCPlayerApi player, PlayerData.Info[] infos)` | End of frame whenever any player's PlayerData changed or was received. `infos` covers all of that player's keys with their state. |
| `OnPersistenceUsageUpdated(VRCPlayerApi player)` | (SDK 3.10.0+) The player's storage usage was updated (e.g. after `RequestStorageUsageUpdate`). |
| `OnPlayerDataStorageWarning(VRCPlayerApi player)` | (SDK 3.10.0+) Usage is **nearing** the limit. No threshold is documented. |
| `OnPlayerDataStorageExceeded(VRCPlayerApi player)` | (SDK 3.10.0+) Usage exceeded the limit. Saves that would exceed the limit are not saved (VRChat logs an error). |

**[Docs]** for all of the above. <https://creators.vrchat.com/worlds/udon/graph/event-nodes>

### `PlayerData.Info`

| Field | Type | Notes |
|---|---|---|
| `Key` | `string` | The key name. |
| `State` | enum | State of the key as of this update. |

`State` values: `Unchanged` (0), `Added` (1), `Removed` (2 — shown once then disappears; key removal isn't currently possible), `Changed` (3), `Restored` (4 — restored from persistent records, only when rejoining an instance you were previously in). **[Docs]**

## What is *not* available

- `RemoveKey(player, key)` — does not exist.
- `ClearAll(player)` — does not exist.
- Atomic batch writes — no documented API.

Players can wipe their own data per world or for all worlds — in-client via **Settings → Debug → User Data**, or on vrchat.com under the **Persistent Data** section of profile settings. This is irreversible. World authors cannot delete keys programmatically. **[Docs]** <https://creators.vrchat.com/worlds/udon/persistence/>

## Patterns

### Initialize on player join

```csharp
public override void OnPlayerRestored(VRCPlayerApi player) {
    if (!player.isLocal) return;
    if (PlayerData.HasKey(player, "score")) {
        currentScore = PlayerData.GetInt(player, "score");
    } else {
        currentScore = 0;
        PlayerData.SetInt("score", 0);
    }
}
```

### Watch for storage approaching the cap

```csharp
// SDK 3.10.0+
public override void OnPlayerDataStorageWarning(VRCPlayerApi player) {
    if (player.isLocal) Debug.LogWarning("Persistent storage almost full");
}
```

### Save a small structure

For a few related values, prefix the keys consistently:

```csharp
// inventory is a DataList
PlayerData.SetInt   ("inv_count", inventory.Count);
PlayerData.SetString("inv_state", VRCJson.TrySerializeToJson(inventory, JsonExportType.Minify, out DataToken json) ? json.String : "");
```

### Saves around player leave

```csharp
public override void OnPlayerLeft(VRCPlayerApi player) {
    // ❌ DON'T:  PlayerData.SetInt("score", currentScore);    // rejected
    // Save earlier (on score changes) instead.
}
```

## Caveats

- Data is **stored compressed**; the 100 KB limit refers to compressed bytes, so effective space depends on entropy — highly compressible data can exceed 300 KB raw. **[Docs]**
- Max key length, max string/byte[] length, and rate limits are **not documented**. Test before shipping.
- Writes are deferred — there's no synchronous confirmation that data made it to the server.
- **ClientSim:** since SDK 3.10.5, ClientSim no longer tries to save simulated changes to player persistence, and won't start a new simulated save while one is still running. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-5>

## Community patterns

### Key-naming convention

Official guidance and community convention: prefix every key with a **unique author/project identifier** to avoid collisions when worlds combine multiple persistent prefabs:

```
Momo-PPP-jumps          // Author-Project-Field (3-segment)
_internal-schemaVersion // underscore-prefixed for hidden bookkeeping keys
```

**[Docs]** <https://creators.vrchat.com/worlds/udon/persistence/player-data/>

### Migration without RemoveKey

Since you cannot delete keys, evolve the schema with an explicit version flag:

```csharp
const int kCurrentSchema = 3;

public override void OnPlayerRestored(VRCPlayerApi player) {
    if (!player.isLocal) return;
    int version = PlayerData.HasKey(player, "_internal-schemaVersion")
        ? PlayerData.GetInt(player, "_internal-schemaVersion")
        : 0;

    if (version < 1) { /* copy 'score' → 'gameScore' */ }
    if (version < 2) { /* split 'inventory' into 'inv_weapons' / 'inv_consumables' */ }
    if (version < 3) { /* normalize unit conversions */ }

    PlayerData.SetInt("_internal-schemaVersion", kCurrentSchema);
}
```

Old keys remain in storage but are unread. Overwriting them with the smallest possible value (e.g. `SetByte(oldKey, 0)`) minimizes their compressed footprint. Type re-use is allowed — you can change the type stored under a key by writing the new type. Canny request for actual deletion: <https://vrchat.canny.io/persistence/p/add-a-way-of-clearing-deleting-existing-playerdata-keys> **[Community]**

### Deferred saves around the "no save in OnPlayerLeft" rule

- **Autosave timer.** Set a dirty flag on writes; periodically (every 10–30 s) the timer commits if dirty. **[Community]**
- **Continuous-sync PlayerObject.** Park frequently-changing fields on a `VRCPlayerObject` in Continuous sync; VRChat auto-saves at the runtime's cadence. **[Community]**
- **Save-on-action.** Write inside the gameplay event that mutated the value, not at exit. The reference *Persistent Idle Game* example uses this pattern. **[Docs]** <https://creators.vrchat.com/worlds/examples/persistence/persistent-idle-game/>
- **"Are you sure?" dismount prompt.** Soft warning when state is dirty before the player closes the client. UX pattern only, no API support.

### Compressed-size monitoring

```csharp
// SDK 3.10.0+
public override void OnPlayerDataStorageWarning(VRCPlayerApi player) {
    // nearing the limit (threshold undocumented); warn the player or trigger compaction
}

public override void OnPlayerDataStorageExceeded(VRCPlayerApi player) {
    // exceeded the limit; saves that would exceed it are NOT saved
}

public void CheckUsage() {
    Networking.RequestStorageUsageUpdate();   // sparingly
}

public override void OnPersistenceUsageUpdated(VRCPlayerApi player) {
    if (!player.isLocal) return;
    int used  = Networking.GetPlayerDataStorageUsage(player);
    int limit = Networking.GetPlayerDataStorageLimit();
    Debug.Log($"PlayerData: {used}/{limit} bytes");
}
```

## See also

- [Player Objects](player-objects.md)
- [Players → VRCPlayerApi](../players/vrc-player-api.md)
- [Data Containers → VRCJson](../data-containers/vrc-json.md) — for serializing complex values into a single string.
- [References → Community Resources](../references/community-resources.md) — NUSaveState for cross-world.
