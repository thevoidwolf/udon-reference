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
static float SimulationTime(VRCPlayerApi player);
static float SimulationTime(GameObject obj);        // networked objects
```

Both overloads are listed on the official Networking page and in the extern table. **[Docs]** / **[Source]** Each returns how far back in time a player or networked object is being simulated (used internally by `VRCObjectSync` and players). `Time.realtimeSinceStartup - Networking.SimulationTime(player)` gives that player's current replication delay in seconds. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-components>

## Server time

```csharp
static double GetServerTimeInSeconds();
static int    GetServerTimeInMilliseconds();     // int, not long
static double CalculateServerDeltaTime(double timeInSeconds, double previousTimeInSeconds);
static System.DateTime GetNetworkDateTime();
```

Signatures from the official Networking methods table (Oct 2026), and they match the extern table (`__GetServerTimeInMilliseconds__SystemInt32`, `__CalculateServerDeltaTime__SystemDouble_SystemDouble__SystemDouble`). **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-components/#networking-methods> / **[Source]**

- Every client sees about the same server time, so use it rather than `Time.time` for timestamps you compare across players. **[Docs]**
- **Server time wraps around and can be negative in VRChat.** Don't compare or subtract two timestamps directly. Use `CalculateServerDeltaTime(now, earlier)`, which corrects for the wrap. **[Docs]**
- **ClientSim starts server time near zero and never wraps**, so wrap bugs won't show in the editor. Test them in VRChat. **[Docs]**

(Earlier revisions of this page gave `GetServerTimeInMilliseconds` as returning `long` and `CalculateServerDeltaTime` as taking one argument and returning `float`. Both were wrong.)

## Other `Networking` methods

| Method | Returns | Notes |
|---|---|---|
| `IsObjectReady(GameObject obj)` | `bool` | Whether the object has finished network setup and is safe to use. **[Docs]** |
| `GetUniqueName(GameObject obj)` | `string` | A network-unique name for the object. **[Docs]** |
| `Destroy(GameObject obj)` | `void` | Destroys a networked object. For runtime spawn/despawn, VRChat points you to `VRCObjectPool`. **[Docs]** |

These were all in the extern table before Oct 2026. They were added to the official methods table then. **[Docs]** / **[Source]**

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

Sync only the start time and have every client work out the elapsed time itself:

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.Manual)]
public class SharedCountdown : UdonSharpBehaviour
{
    [UdonSynced] private double _startTime;

    public override void Interact()
    {
        Networking.SetOwner(Networking.LocalPlayer, gameObject);
        _startTime = Networking.GetServerTimeInSeconds();
        RequestSerialization();
    }

    void Update()
    {
        double elapsed = Networking.CalculateServerDeltaTime(
            Networking.GetServerTimeInSeconds(), _startTime);
        // Every client computes the same value from the same synced start time.
    }
}
```

Adapted from the official example. **[Docs]** `CalculateServerDeltaTime` handles wrap-around, which subtracting raw timestamps doesn't.

## See also

- [Networking](../networking/README.md)
- [Networking → Debugging & stats](../networking/debugging-and-stats.md) — `IsClogged`, network stats.
- [VRCPlayerApi](vrc-player-api.md)
- [Persistence → Player Objects](../persistence/player-objects.md)
