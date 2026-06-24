# PlayerData

Static class `VRC.SDK3.Persistence.PlayerData`. Key-value storage scoped per player per world. **[Docs]**

## Setters (local player only)

All have signature `void Set<Type>(string key, <Type> value)`. They write the **local** player's data — calls referencing remote players' data silently fail.

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

No success indicator. **Cannot be called from `OnPlayerLeft`.**

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

These **throw** if the key is missing. Always pair with `HasKey` or use `TryGet*`.

## TryGetters

```csharp
PlayerData.TryGetString(VRCPlayerApi player, string key, out string value, out bool success) → bool
PlayerData.TryGetInt   (VRCPlayerApi player, string key, out int    value, out bool success) → bool
// ... same pattern for every type
```

The two out params:
- `value` — the read value (default-initialised on failure).
- `success` — `true` if the key existed and the type matched.

The method's return value matches `success` for compactness in `if` statements:

```csharp
if (PlayerData.TryGetInt(player, "score", out int score, out _)) {
    Debug.Log($"score: {score}");
}
```

## Query methods

```csharp
bool HasKey(VRCPlayerApi player, string key);
System.Type GetType(VRCPlayerApi player, string key);
bool TryGetType(VRCPlayerApi player, string key, out System.Type t);
```

`GetType` returns the runtime type token currently stored under the key. Throws if missing — use `TryGetType` for safe access.

## Storage inspection

```csharp
int GetPlayerDataStorageLimit();                    // returns 102400 (100 KB)
int GetPlayerDataStorageUsage(VRCPlayerApi target); // current compressed bytes; -1 if not yet loaded
void RequestStorageUsageUpdate();                   // ask the runtime to recompute usage
```

After `RequestStorageUsageUpdate`, `OnPersistenceUsageUpdated` fires when the new value is ready.

## Events

All hang off `VRC.SDK3.Persistence.PlayerData` as static `event System.Action<...>` :

| Event | Signature | Fires |
|---|---|---|
| `OnPlayerRestored` | `Action<VRCPlayerApi>` | Once per player after their persistent data has loaded from VRChat servers. **Wait for this before reading.** |
| `OnPlayerDataUpdated` | `Action<VRCPlayerApi, PlayerData.Info[]>` | At end-of-frame whenever any player's PlayerData changed (local writes or remote updates). `Info` lists the changed keys. |
| `OnPersistenceUsageUpdated` | `Action<VRCPlayerApi>` | After `RequestStorageUsageUpdate` completes — `GetPlayerDataStorageUsage` is now accurate. |
| `OnPlayerDataStorageWarning` | `Action<VRCPlayerApi>` | Crossed ~90 KB. |
| `OnPlayerDataStorageExceeded` | `Action<VRCPlayerApi>` | Attempted to write beyond 100 KB. The write is *not saved*. |

**[Docs]** for all of the above.

## What is *not* available

- `RemoveKey(player, key)` — does not exist.
- `ClearAll(player)` — does not exist.
- Atomic batch writes — no documented API.

Players can wipe their own data through the VRChat web panel. World authors cannot delete keys programmatically.

## Patterns

### Initialize on player join

```csharp
void Start() {
    PlayerData.OnPlayerRestored += HandleRestored;
}

void HandleRestored(VRCPlayerApi player) {
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
void Start() {
    PlayerData.OnPlayerDataStorageWarning += HandleWarning;
}

void HandleWarning(VRCPlayerApi player) {
    if (player.isLocal) Debug.LogWarning("Persistent storage almost full");
}
```

### Save a small structure

For a few related values, prefix the keys consistently:

```csharp
PlayerData.SetInt   ("inv_count", inventory.Length);
PlayerData.SetString("inv_state", VRCJson.TrySerializeToJson(inventory, JsonExportType.Minify, out var s) ? s : "");
```

### Saves around player leave

```csharp
public override void OnPlayerLeft(VRCPlayerApi player) {
    // ❌ DON'T:  PlayerData.SetInt("score", currentScore);    // rejected
    // Save earlier (on score changes) instead.
}
```

## Caveats

- Data is **compressed server-side**; the 100 KB limit refers to compressed bytes, so you may have more or less effective space depending on entropy.
- Max key length, max string/byte[] length, and rate limits are **not documented**. Test before shipping.
- Writes are deferred — there's no synchronous confirmation that data made it to the server.

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
public override void OnPlayerDataStorageWarning(VRCPlayerApi player) {
    // crossed ~90 KB; warn the player or trigger compaction
}

public override void OnPlayerDataStorageExceeded(VRCPlayerApi player) {
    // exceeded 100 KB; the offending write was NOT saved
}
```

## See also

- [Player Objects](player-objects.md)
- [Players → VRCPlayerApi](../players/vrc-player-api.md)
- [Data Containers → VRCJson](../data-containers/vrc-json.md) — for serializing complex values into a single string.
- [References → Community Resources](../references/community-resources.md) — NUSaveState for cross-world.
