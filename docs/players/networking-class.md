# Networking (static)

`VRC.SDKBase.Networking` — instance-wide helpers. Distinct from the per-behaviour [Networking](../networking/README.md) topic; this page is about the static class itself.

## Player access

```csharp
static VRCPlayerApi LocalPlayer { get; }     // null in editor before init
static bool IsMaster { get; }                // true if local player is master
static VRCPlayerApi Master { get; }          // SDK 3.7.0+ — current instance master; always valid
static VRCPlayerApi InstanceOwner { get; }   // SDK 3.7.0+ — instance owner, or null if they aren't in the instance
static bool IsInstanceOwner { get; }         // true if the local player is the instance owner
```

`Master` and `InstanceOwner` signatures are verbatim from the 3.7.0 release notes; descriptions from the Network Components page. **[Docs]** <https://creators.vrchat.com/releases/release-3-7-0>, <https://creators.vrchat.com/worlds/udon/networking/network-components>

- **Master** is the default owner of networked objects. It changes when the current master leaves (chosen server-side — don't predict who). React with `public override void OnMasterTransferred(VRCPlayerApi newMaster)` (SDK 3.7.0+). Don't use master status for security gating. **[Docs]**
- **Instance owner** is the creator of an Invite / Invite+ / Friends / Friends+ instance and never changes. `IsInstanceOwner` is always `false` in Group, Public, and Build & Test instances. Prefer this over master for "host" privileges. **[Docs]**

## Ownership

```csharp
static bool IsOwner(VRCPlayerApi player, GameObject gameObject);
static bool IsOwner(GameObject gameObject);                          // shortcut for local player
static VRCPlayerApi GetOwner(GameObject gameObject);
static void SetOwner(VRCPlayerApi player, GameObject gameObject);
```

See [Networking → ownership](../networking/ownership.md) for the full transfer rules.

## Network state

```csharp
static bool IsNetworkSettled { get; }   // true once all instance data has been deserialized and applied
static bool IsClogged { get; }          // true when too much data is trying to leave the client
```

`IsNetworkSettled` is the right "world has settled" check at startup. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-components>

### Simulation time

```csharp
static float SimulationTime(VRCPlayerApi player);   // float [Docs]; docs say it also works for networked objects — that overload (GameObject?) [Unverified]
```

Returns how far back in time a player or networked object is being simulated (used internally by `VRCObjectSync` and players). `Time.realtimeSinceStartup - Networking.SimulationTime(player)` gives that player's current replication delay in seconds. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-components>

## Server time

```csharp
static long GetServerTimeInMilliseconds();
static double GetServerTimeInSeconds();           // [Community-known]
static System.DateTime GetNetworkDateTime();      // [Community-known]
static float CalculateServerDeltaTime(double timeStamp);
```

Server time is the same across all clients (within network drift). Use this rather than `Time.time` for time-stamped events you want to compare across players.

## Player object lookups

```csharp
static GameObject[] GetPlayerObjects(VRCPlayerApi player);
static Component FindComponentInPlayerObjects(VRCPlayerApi player, Component referenceComponent);
```

See [Persistence → Player Objects](../persistence/player-objects.md).

## Persistence storage information

The docs place the persistence storage-usage methods in `VRC.SDKBase.Networking` too **[Docs]**:

```csharp
static int  GetPlayerDataStorageLimit();                         // SDK 3.10.0+, bytes
static int  GetPlayerDataStorageUsage(VRCPlayerApi target);      // SDK 3.10.0+, last computed usage
static int  GetPlayerObjectStorageLimit();                       // bytes
static int  GetPlayerObjectStorageUsage(VRCPlayerApi target);    // last computed usage
static void RequestStorageUsageUpdate();                         // SDK 3.10.0+, local player; result via OnPersistenceUsageUpdated
```

See [Persistence → PlayerData](../persistence/player-data.md#storage-information-sdk-3100).

## Instantiation

```csharp
// Mentioned in docs but not officially recommended:
// static GameObject Instantiate(...)
```

The standard advice is **don't** call `Networking.Instantiate`. Use a `VRCObjectPool` for networked spawning. **[Docs]**

## Patterns

### "Wait for the world to be ready"

```csharp
void OnEnable() {
    SendCustomEventDelayedFrames(nameof(_TryStart), 1);
}

public void _TryStart() {
    if (!Networking.IsNetworkSettled) {
        SendCustomEventDelayedFrames(nameof(_TryStart), 5);
        return;
    }
    BeginRealLogic();
}
```

### Time-stamped events

```csharp
[UdonSynced] long _eventStartMs;

public void StartEvent() {
    if (!Networking.IsOwner(gameObject)) return;
    _eventStartMs = Networking.GetServerTimeInMilliseconds();
    RequestSerialization();
}

public override void OnDeserialization() {
    long now = Networking.GetServerTimeInMilliseconds();
    double secondsSinceStart = (now - _eventStartMs) / 1000.0;
    // every client agrees on this value within network drift
}
```

## See also

- [Networking](../networking/README.md)
- [Networking → Debugging & stats](../networking/debugging-and-stats.md) — `IsClogged`, network stats.
- [VRCPlayerApi](vrc-player-api.md)
- [Persistence → Player Objects](../persistence/player-objects.md)
