# Sync Modes

A behaviour replicates state in one of two modes: **Continuous** or **Manual**. The mode is configured per behaviour either via the inspector dropdown (when `BehaviourSyncMode.Any`) or pinned by `[UdonBehaviourSyncMode(...)]` on the class.

## Continuous

The runtime automatically broadcasts the behaviour's `[UdonSynced]` fields at a few-times-per-second cadence (exact rate not documented; community-observed at ~5 Hz). Non-owners receive incremental updates.

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.Continuous)]
public class SpinningCube : UdonSharpBehaviour {
    [UdonSynced(UdonSyncMode.Smooth)] float rotation;

    void Update() {
        if (Networking.IsOwner(gameObject))
            rotation += 90f * Time.deltaTime;
        transform.rotation = Quaternion.Euler(0, rotation, 0);
    }
}
```

**Use Continuous when:**
- Values change every frame or every few frames.
- A smooth visual is more important than a precise one.
- The state is "lossy-tolerant" — missing one packet just means a slightly stale value.

**Trade-offs:**
- Each behaviour eats a slice of the instance's bandwidth budget. Worlds with hundreds of continuous behaviours degrade quickly.
- Per-serialization packet cap is **~200 bytes** — even a moderate-length synced string blows it. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-details/>
- `RequestSerialization()` is a no-op on Continuous behaviours.
- `OnPostSerialization` still fires; you can use it to monitor packet success.

## Manual

The owner calls `RequestSerialization()` when state has changed. The runtime emits a packet at the next available slot. No automatic re-broadcasts — if a remote client missed the packet, they won't know to ask for it again (the runtime *does* re-send if the network is congested, but as an author you treat it as fire-once).

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.Manual)]
public class Door : UdonSharpBehaviour {
    [UdonSynced] bool isOpen;

    public override void Interact() {
        if (!Networking.IsOwner(gameObject))
            Networking.SetOwner(Networking.LocalPlayer, gameObject);
        isOpen = !isOpen;
        ApplyVisuals();
        RequestSerialization();
    }

    public override void OnDeserialization() => ApplyVisuals();
    void ApplyVisuals() => transform.rotation = isOpen ? Quaternion.Euler(0,90,0) : Quaternion.identity;
}
```

**Use Manual when:**
- State changes are discrete events (toggles, picks, points scored).
- Correctness matters more than smoothness.
- You're transmitting larger payloads (a `string` over a few hundred bytes).

**Trade-offs:**
- Late joiners get the *most recently broadcast* state when they enter the instance. **[Docs]**
- `Linear`/`Smooth` sync modes still work on numeric fields — but interpolation only kicks in across the gap between `RequestSerialization()` calls, which can be long. Usually you set `UdonSyncMode.None` on manual behaviours.
- Per-serialization budget **~280 KB** (much larger than continuous). **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-details/>

## Mixed-mode behaviours don't exist

A single behaviour has a single mode. If you have both "every-frame smooth rotation" state and "important discrete state" state to sync, split them across two behaviours on the same GameObject.

## Event timing differences

| Event | Continuous | Manual |
|---|---|---|
| `OnPreSerialization` | Fires before each auto-broadcast | Fires when `RequestSerialization()` is called |
| `OnPostSerialization` | After each broadcast | After each manual broadcast |
| `OnDeserialization` | After every received packet | After every received packet |

## See also

- [Attributes → BehaviourSyncMode](../attributes/behaviour-sync-mode.md)
- [Serialization](serialization.md)
- [UdonSharpBehaviour → networking events](../udonsharp-behaviour/networking-events.md)
