# Player Objects

A **Player Object** is a GameObject template that the VRChat runtime auto-instantiates once per joining player. Combined with `VRCEnablePersistence`, an UdonBehaviour on the Player Object can have `[UdonSynced]` fields that persist across sessions. Shipped in SDK 3.7.4 (Nov 21, 2024). **[Docs]** <https://creators.vrchat.com/worlds/udon/persistence/player-object>

## Setup

1. Create a GameObject in your scene that represents a single player's persistent state (e.g. their inventory holder).
2. Add a **`VRCPlayerObject`** component to it — this makes it a PlayerObject **template**.
3. Add an `UdonSharpBehaviour` (or `UdonBehaviour`) with `[UdonSynced]` fields for the state to persist — on the template or one of its children.
4. Add a **`VRCEnablePersistence`** component to *the same GameObject* as the UdonBehaviour with persistent fields.
5. The original template GameObject is disabled by VRChat at runtime. Per-player copies (all components and children, same position/rotation/scale/parent as the template) are created automatically.

All steps **[Docs]**. Template rules **[Docs]**:
- Templates may reference their own / child components and other scene components.
- Scene components can **not** reference templates — use direct references to the spawned PlayerObjects (see [Discovering instances](#discovering-instances)).
- Don't modify, destroy, or edit a template after VRChat disables it.

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

Crucially: **do not read or write these fields before `OnPlayerRestored` fires for that player.** Their initial values are placeholder; the server's stored value lands shortly after join and may overwrite your changes. **[Docs]**

### Telling restored data from live sync: `DeserializationResult.isFromStorage`

Override the `OnDeserialization(DeserializationResult result)` overload to see where a deserialization came from:

```csharp
public override void OnDeserialization(DeserializationResult result) {
    if (result.isFromStorage) {
        // values were restored from persistent storage, not sent by a realtime client
    }
    // result.sendTime / result.receiveTime: seconds since VRChat started, from *your* clock
}
```

`isFromStorage` — "If true, then the included data was restored from storage rather than received from other realtime clients." **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-components> The `DeserializationResult` overload itself dates from SDK 3.1.x; the release that added `isFromStorage` isn't stated in the release notes (presumably 3.7.4 with persistence). **[Unverified]**

## Lifecycle

1. Player joins.
2. VRChat instantiates the Player Object for that player.
3. `OnPlayerJoined(player)` fires on all behaviours.
4. Server delivers the persistent data → fields are written (`OnDeserialization` with `isFromStorage == true`).
5. `OnPlayerRestored(player)` fires — on every client, once per player in the instance (including the owner of the PlayerObject). **[Docs]**
6. The Player Object's `OnDeserialization` may fire again as the owner syncs further changes.

In `Start` and `OnDeserialization`, ownership of the PlayerObject is guaranteed to be correct. **[Docs]**

## Ownership

Each player **owns** their own Player Object, plus every GameObject with a synced UdonBehaviour inside it. Ownership cannot be transferred — so tools or pickups inside it can't be "stolen". Player A cannot directly write to Player B's Player Object even with `Networking.SetOwner` — the runtime rejects the transfer. **[Docs]**

That makes Player Objects ideal for "this player's state that I want to read but not mutate from anywhere else."

## Discovering instances

`VRC.SDKBase.Networking` exposes lookup methods:

```csharp
GameObject[] objects = Networking.GetPlayerObjects(player);
// — or via the VRCPlayerApi utility wrapper:
GameObject[] objects = player.GetPlayerObjects();
```

**[Docs]**. Entries can be invalid — the official example skips them with `Utilities.IsValid(objects[i])` before calling `GetComponentInChildren<T>()`.

To map a *template component reference* (which must be on a template or one of its children) to the matching component on a specific player's instance **[Docs]**:

```csharp
public UdonBehaviour templateBehavior;  // reference to the template UdonBehaviour

void Use(VRCPlayerApi player) {
    UdonBehaviour theirInstance =
        (UdonBehaviour)Networking.FindComponentInPlayerObjects(player, templateBehavior);
    // now read/write fields on theirInstance using GetProgramVariable / SendCustomEvent
}
```

## Storage information

Like the PlayerData equivalents, these live on `VRC.SDKBase.Networking`. **[Docs]**

```csharp
int  Networking.GetPlayerObjectStorageLimit();                    // limit in bytes (100 KB per the persistence overview)
int  Networking.GetPlayerObjectStorageUsage(VRCPlayerApi target); // last *computed* usage for target
void Networking.RequestStorageUsageUpdate();                      // SDK 3.10.0+; recomputes local PlayerData + PlayerObject usage
```

Usage can go stale; avoid calling `RequestStorageUsageUpdate` frequently. The exact returned limit value and pre-load behaviour are undocumented. **[Unverified]**

Events (U# overrides, each takes the affected player) **[Docs]**:

| Event | Fires |
|---|---|
| `OnPersistenceUsageUpdated(VRCPlayerApi player)` | (SDK 3.10.0+) The player's persistence usage was updated. |
| `OnPlayerObjectStorageWarning(VRCPlayerApi player)` | Player Object usage is **nearing** the limit (threshold undocumented). |
| `OnPlayerObjectStorageExceeded(VRCPlayerApi player)` | Player Object usage exceeded the limit; saves that would exceed it are not saved. |

The 3.10.0 release notes only list the PlayerData storage functions/events and `RequestStorageUsageUpdate` / `OnPersistenceUsageUpdated` as new; the SDK version that introduced `GetPlayerObjectStorage*` and `OnPlayerObjectStorage*` isn't stated. **[Unverified]**

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
- Like PlayerData, **persistence cannot be saved from the local player's `OnPlayerLeft`** — schedule earlier. **[Docs]**
- **VRChat Dynamics** — PhysBones, Contacts and VRC Constraints behave properly on a `VRCPlayerObject` since SDK 3.10.0 (they were unreliable before). **[Docs]** <https://creators.vrchat.com/releases/release-3-10-0>

## Community history

`VRCPlayerObject` shipped in SDK 3.7.4 (Nov 21, 2024) and replaced the canonical community pattern:

- **CyanPlayerObjectPool** (CyanLaser) was *the* per-player state mechanism for years. The repo was archived on Oct 11, 2024, shortly before 3.7.4's public release. New worlds should use the stock component; older worlds may still rely on it. **[Community]** <https://github.com/CyanLaser/CyanPlayerObjectPool>

Previously reported bug: in some cases the player did not own their PlayerObject at `Start` / `OnPlayerRestored`. **[Community]** <https://feedback.vrchat.com/persistence/p/player-does-not-own-their-vrc-playerobject-item-at-start-or-onplayerrestored> The current docs state ownership is guaranteed correct in `Start` and `OnDeserialization` **[Docs]** — if you still hit this, check ownership in `OnDeserialization` too.

## See also

- [PlayerData](player-data.md)
- [Players → VRCPlayerApi](../players/vrc-player-api.md)
- [Networking → VRCObjectPool](../networking/object-pool.md) — a different per-player-ish system, but for ephemeral spawning.
- [References → Community Resources](../references/community-resources.md)
