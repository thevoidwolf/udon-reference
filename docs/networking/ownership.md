# Ownership

Every networked GameObject in an Udon world has exactly one **owner** — a player whose writes to `[UdonSynced]` fields are authoritative. **[Docs]**

The owner is *not* the same as the master, though the first player to enter an instance starts out as owner of everything by default. **[Docs]**

## Reading ownership

```csharp
VRCPlayerApi owner = Networking.GetOwner(gameObject);
bool isMine     = Networking.IsOwner(gameObject);
bool isTheirs   = Networking.IsOwner(someOtherPlayer, gameObject);
```

`Networking.IsOwner(GameObject)` is the local-player version; the two-arg form lets you check any player.

## Transferring ownership

```csharp
Networking.SetOwner(VRCPlayerApi newOwner, GameObject obj);
```

Common idiom — "take ownership before writing synced state":

```csharp
public override void Interact() {
    if (!Networking.IsOwner(gameObject))
        Networking.SetOwner(Networking.LocalPlayer, gameObject);

    isOpen = !isOpen;
    RequestSerialization();
}
```

`SetOwner` is committed on every client; before commit, each client's `UdonSharpBehaviour`s on the GameObject receive `OnOwnershipRequest`. If *any* returns `false`, the transfer is vetoed.

## `OnOwnershipRequest` — gating transfers

```csharp
public virtual bool OnOwnershipRequest(VRCPlayerApi requestingPlayer, VRCPlayerApi requestedOwner) => true
```

Return `false` to deny.

```csharp
public override bool OnOwnershipRequest(VRCPlayerApi requestingPlayer, VRCPlayerApi requestedOwner) {
    return requestedOwner.isMaster;  // only the master can become owner
}
```

`requestingPlayer` is the one that called `SetOwner`. `requestedOwner` is who they wanted to make owner. Most of the time these are equal (a player taking ownership for themselves), but `SetOwner(p, obj)` lets a third party push ownership onto someone else.

## `OnOwnershipTransferred` — reacting to transfer

```csharp
public virtual void OnOwnershipTransferred(VRCPlayerApi player)
```

Fires on every client after a transfer commits. Useful for "highlight whoever currently holds this" or for re-initializing per-owner state.

## Ownership inheritance

A GameObject's ownership is **not** automatically related to its parent's ownership. Two children of the same parent can have different owners.

A pickup whose `Auto Hold` is off won't automatically transfer ownership on pickup — you have to call `Networking.SetOwner` yourself. A pickup whose `Auto Hold` is on transfers ownership to the player who grabs it.

## Master vs Owner

| Concept | What it means | API |
|---|---|---|
| **Master** | The instance master, usually but not always the first player. May change when the master leaves. | `VRCPlayerApi.isMaster`, `Networking.IsMaster` |
| **Owner** | Authority over a specific GameObject's synced state. | `Networking.GetOwner(go)`, `Networking.IsOwner(go)`, `Networking.SetOwner(p, go)` |

The master concept is fragile (masters change when they leave). The official recommendation: don't rely on master status for game logic; use explicit ownership checks for authority. **[Docs]**

## Late-joiner ownership

When a player joins, they receive the current owner of every networked object. They do not become owner of anything by default. If the previous owner left the instance, ownership of their objects passes to the next-master at that point (or to the new master if there is one).

## Patterns

### "Anyone can toggle, everyone sees"

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.Manual)]
public class GlobalToggle : UdonSharpBehaviour {
    public GameObject toggleObject;
    [UdonSynced] bool isOn;

    public override void Interact() {
        if (!Networking.IsOwner(gameObject))
            Networking.SetOwner(Networking.LocalPlayer, gameObject);
        isOn = !isOn;
        ApplyVisuals();
        RequestSerialization();
    }

    public override void OnDeserialization() => ApplyVisuals();
    void ApplyVisuals() => toggleObject.SetActive(isOn);
}
```

### "Master only"

```csharp
public override void Interact() {
    if (!Networking.IsMaster) return;     // bail
    DoMasterAction();
}

public override bool OnOwnershipRequest(VRCPlayerApi req, VRCPlayerApi target) {
    return target.isMaster;
}
```

## Known race conditions (community-tracked)

- **No synced writes immediately after `SetOwner`.** Writes between `Networking.SetOwner` and the network acknowledging the transfer are silently dropped. Always serialize-after-transfer, not transfer-then-serialize-in-same-frame. **[Community]** <https://udonsharp.docs.vrchat.com/networking-tips-&-tricks/>
- **`OnOwnershipTransferred` vs `OnPlayerLeft` ordering is non-deterministic.** Previously transfer-fired-before-left; now random. **[Community]** <https://feedback.vrchat.com/udon/p/order-of-onownershiptransferred-and-onplayerleft-when-another-player-leaves-the>
- **Passive ownership transfer may not fire `OnOwnershipTransferred`.** When the owner leaves without an explicit transfer, the implicit re-assignment sometimes skips the callback. Community workaround: set a one-frame bool flag during `OnPlayerLeft` handling and inspect it when claiming. **[Community]** <https://feedback.vrchat.com/udon/p/1258-onownershiptransferred-does-not-fire-at-onplayerleft-if-last-owner-is-passive>
- **`Networking.SetOwner` to a "third-party" target requires that target to accept the transfer.** A vetoed transfer fires `OnOwnershipRequest` but does *not* fire `OnOwnershipTransferred`.

## See also

- [Sync modes](sync-modes.md)
- [VRCObjectSync](object-sync.md) — automatic Transform/Rigidbody sync rides on the same ownership model.
- [Players → Networking class](../players/networking-class.md)
- [References → Community Resources](../references/community-resources.md) — `VRCLockstep` sidesteps ownership entirely.
