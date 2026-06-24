# Player Objects

A **Player Object** is a GameObject template that the VRChat runtime auto-instantiates once per joining player. Combined with `VRCEnablePersistence`, an UdonBehaviour on the Player Object can have `[UdonSynced]` fields that persist across sessions. **[Docs]**

## Setup

1. Create a GameObject in your scene that represents a single player's persistent state (e.g. their inventory holder).
2. Add a **`VRCPlayerObject`** component to it.
3. Add an `UdonSharpBehaviour` (or `UdonBehaviour`) with `[UdonSynced]` fields for the state to persist.
4. Add a **`VRCEnablePersistence`** component to *the same GameObject* as the UdonBehaviour with persistent fields.
5. The original template GameObject is disabled by VRChat at runtime. Per-player instances are created automatically.

```
PersistenceRoot
├── _Template (VRCPlayerObject; disabled at runtime by VRChat)
│    ├── UdonBehaviour with [UdonSynced] fields
│    └── VRCEnablePersistence
```

## How `[UdonSynced]` interacts with persistence

Fields marked `[UdonSynced]` on a Player Object's UdonBehaviour are:
- Replicated to all clients (normal sync behaviour).
- **Additionally** stored in VRChat's server-side persistence layer because of `VRCEnablePersistence`.
- Restored on join: when a returning player enters a world instance, the server provides their saved values.

Crucially: **do not read or write these fields before `OnPlayerRestored` fires for that player.** Their initial values are placeholder; the server's stored value lands shortly after join.

## Lifecycle

1. Player joins.
2. VRChat instantiates the Player Object for that player.
3. `OnPlayerJoined(player)` fires on all behaviours.
4. Server delivers the persistent data → fields are written.
5. `PlayerData.OnPlayerRestored(player)` fires.
6. The Player Object's `OnDeserialization` may fire if other clients sync further changes.

## Ownership

Each player **owns** their own Player Object. Ownership cannot be transferred. Player A cannot directly write to Player B's Player Object even with `Networking.SetOwner` — the runtime rejects the transfer.

That makes Player Objects ideal for "this player's state that I want to read but not mutate from anywhere else."

## Discovering instances

`VRC.SDKBase.Networking` exposes lookup methods:

```csharp
GameObject[] objects = Networking.GetPlayerObjects(player);
// — or via the instance method:
GameObject[] objects = player.GetPlayerObjects();
```

To map a *template component reference* to the matching component on a specific player's instance:

```csharp
public UdonBehaviour templateBehavior;  // reference to the template UdonBehaviour

void Use(VRCPlayerApi player) {
    UdonBehaviour theirInstance =
        (UdonBehaviour)Networking.FindComponentInPlayerObjects(player, templateBehavior);
    // now read/write fields on theirInstance using GetProgramVariable / SendCustomEvent
}
```

## Storage inspection

```csharp
int GetPlayerObjectStorageLimit();                       // 102400 (100 KB)
int GetPlayerObjectStorageUsage(VRCPlayerApi target);    // current compressed bytes; -1 if not loaded
```

Same warning / exceeded events fire as for PlayerData but for the Player Object subsystem:

| Event | Fires |
|---|---|
| `OnPlayerObjectStorageWarning` | ~90 KB used. |
| `OnPlayerObjectStorageExceeded` | Exceeded 100 KB; writes not saved. |

## Example: per-player inventory

```csharp
// On the player object template's UdonBehaviour:
[UdonBehaviourSyncMode(BehaviourSyncMode.Manual)]
public class PlayerInventory : UdonSharpBehaviour {
    [UdonSynced] public string serializedItems = "";

    public void AddItem(string item) {
        if (!Networking.IsOwner(gameObject)) return;
        // ...mutate inventory...
        SerializeAndPush();
    }

    void SerializeAndPush() {
        // serialize state into serializedItems
        RequestSerialization();
    }

    public override void OnDeserialization() {
        DeserializeFrom(serializedItems);
    }
}
```

The owner is the player whose Player Object this is. Other players can *read* the synced state but `Networking.SetOwner` is rejected.

## Caveats

- Cannot share data **between** Player Objects of different players except through the normal sync model.
- Cannot transfer ownership.
- Cannot delete keys (just like PlayerData).
- Adding `VRCEnablePersistence` *without* `VRCPlayerObject` does nothing — both must be present.
- Like PlayerData, **persistence cannot be saved from `OnPlayerLeft`** — schedule earlier.

## Community history

`VRC_PlayerObject` shipped in SDK 3.7.4 (Oct 2024) and replaced the canonical community pattern:

- **CyanPlayerObjectPool** (CyanLaser) was *the* per-player state mechanism for years. Archived the same day VRChat shipped `VRC_PlayerObject`. New worlds should use the stock component; older worlds may still rely on it. **[Community]** <https://github.com/CyanLaser/CyanPlayerObjectPool>

Known bug being tracked: in some cases the player does not own their `VRC_PlayerObject` at `Start` / `OnPlayerRestored`. **[Community]** <https://feedback.vrchat.com/persistence/p/player-does-not-own-their-vrc-playerobject-item-at-start-or-onplayerrestored>

## See also

- [PlayerData](player-data.md)
- [Players → VRCPlayerApi](../players/vrc-player-api.md)
- [Networking → VRCObjectPool](../networking/object-pool.md) — a different per-player-ish system, but for ephemeral spawning.
- [References → Community Resources](../references/community-resources.md)
