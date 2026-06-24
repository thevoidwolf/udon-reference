# Networking (static)

`VRC.SDKBase.Networking` — instance-wide helpers. Distinct from the per-behaviour [Networking](../networking/README.md) topic; this page is about the static class itself.

## Player access

```csharp
static VRCPlayerApi LocalPlayer { get; }   // null in editor before init
static bool IsMaster { get; }              // true if local player is master
```

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
static bool IsNetworkSettled { get; }   // true after initial state reconciliation
static bool IsClogged { get; }          // true if the connection is backed up
```

`IsNetworkSettled` is the right "world has settled" check at startup. **[Docs]**

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
- [VRCPlayerApi](vrc-player-api.md)
- [Persistence → Player Objects](../persistence/player-objects.md)
