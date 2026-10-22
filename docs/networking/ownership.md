# Ownership

Every networked GameObject in an Udon world has exactly one **owner** — a player whose writes to `[UdonSynced]` fields are authoritative. **[Docs]**

The owner is *not* the same as the master, though the first player to enter an instance starts out as owner of everything by default. More precisely, the master owns every object whose ownership was never set or transferred. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/ownership/>

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

After `SetOwner` runs, the local player is treated as the new owner, so the idiom above can write and serialize in the same frame. **[Docs]** Before the transfer is confirmed, `OnOwnershipRequest` runs locally on **both the requester and the current owner** (not on every client). If either returns `false`, the transfer is rejected. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-components/>

## `OnOwnershipRequest` — gating transfers

```csharp
public virtual bool OnOwnershipRequest(VRCPlayerApi requestingPlayer, VRCPlayerApi requestedOwner) => true
```

Return `false` to deny. The official signature names the second parameter `newOwner`; the name doesn't matter for overriding. **[Docs]**

> **Keep the logic deterministic.** The requester and the owner each evaluate it. If they disagree, for example because it reads unsynced local state, ownership desyncs. This usually shows up as the owner unexpectedly rejecting the transfer. **[Docs]**

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

Fires on every client after a transfer commits, with the **new** owner. **[Docs]** Useful for "highlight whoever currently holds this" or for re-initializing per-owner state.

## Ownership inheritance

A GameObject's ownership is **not** automatically related to its parent's ownership. Two children of the same parent can have different owners.

Grabbing a networked `VRCPickup` (for example one with `VRCObjectSync`) transfers ownership of that GameObject to the grabbing player. **[Community-known]** `Auto Hold` has nothing to do with ownership. It only controls whether the pickup stays in the hand after the grab action is released. **[Docs]** <https://creators.vrchat.com/worlds/components/vrc_pickup/> Other behaviours on *child* or *parent* objects keep their own owners, so call `Networking.SetOwner` on those yourself.

## Master vs Owner

| Concept | What it means | API |
|---|---|---|
| **Master** | The instance master. Default owner of networked objects. Changes when the master leaves. | `VRCPlayerApi.isMaster`, `Networking.IsMaster`, `Networking.Master` *(3.7.0+)*, `OnMasterTransferred` *(3.7.0+)* |
| **Instance owner** | The player who created the instance. Has moderation permissions. **Never changes.** | `VRCPlayerApi.isInstanceOwner`, `Networking.IsInstanceOwner`, `Networking.InstanceOwner` *(3.7.0+)* |
| **Owner** | Authority over a specific GameObject's synced state. | `Networking.GetOwner(go)`, `Networking.IsOwner(go)`, `Networking.SetOwner(p, go)` |

### Master and instance-owner APIs (SDK 3.7.0+)

All items in this section: **[Docs]**.

```csharp
VRCPlayerApi Networking.Master { get; }         // current master; always valid
VRCPlayerApi Networking.InstanceOwner { get; }  // null while the instance owner is not in the instance
public virtual void OnMasterTransferred(VRCPlayerApi newMaster)
```

- `OnMasterTransferred` fires for everyone when the master changes because the previous master left. `newMaster` is always valid. For the first player in a new instance, it fires *after* `OnPlayerJoined`, as a transfer from "nobody". **[Docs]**
- `IsInstanceOwner` is `true` for the instance creator in Invite, Invite+, Friends and Friends+ instances. It is **always `false`** in Group instances, Public instances, and Build & Test. `InstanceOwner` returns the creator again if they rejoin. **[Docs]**

Source: <https://creators.vrchat.com/worlds/udon/networking/network-components/#networking-properties>. Introduced in SDK 3.7.0 (Aug 16, 2024; first in the 3.6.2 beta). **[Docs]**

### Master selection rules

All items in this section: **[Docs]**.

These are the only guarantees VRChat makes:

- There is always a valid master.
- The first player to enter an empty instance becomes master.
- The master changes when the current master leaves. It can also change if an Android master keeps VRChat in the background too long.
- When the master leaves, the new master is chosen **before** `OnPlayerLeft` fires.
- Don't rely on any particular player becoming master. The server picks based on platform, network conditions, and other criteria.

The master concept is fragile. A master can become unresponsive for a while, and events during that time may not run. The official recommendation is to use ownership checks for networking logic. Don't use master status to gate access to features; use the instance owner or your own moderation system for that. **[Docs]**

## Late-joiner ownership

When a player joins, they receive the current owner of every networked object. They do not become owner of anything by default. **[Docs]** If an owner leaves, VRChat automatically assigns a new owner. **[Docs]** In practice that is usually the master. **[Community-known]**

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

Fine for load-balancing work, such as "someone has to run the game loop". **Not** for access control; see [Master vs Owner](#master-vs-owner).

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

- **"Can't write synced vars right after `SetOwner`" is outdated.** The UdonSharp tips page (written in 2020, before the Udon networking update) says the old owner must acknowledge the transfer first. **[Community]** <https://udonsharp.docs.vrchat.com/networking-tips-&-tricks/> Current official docs and examples call `SetOwner`, write, then `RequestSerialization()` in the same frame. **[Docs]** A transfer can still be *rejected* by `OnOwnershipRequest` on the current owner, so don't assume it stuck. Check `Networking.IsOwner` later or react in `OnOwnershipTransferred`.
- **`OnOwnershipTransferred` vs `OnPlayerLeft` ordering is non-deterministic.** It used to be transfer-before-left; reported as inconsistent since Nov 2024. The Canny post is marked "tracked", with no fix announced. **[Community]** <https://feedback.vrchat.com/udon/p/order-of-onownershiptransferred-and-onplayerleft-when-another-player-leaves-the> (The *master* is re-chosen before `OnPlayerLeft`. **[Docs]**)
- **Historical: passive ownership transfer skipping `OnOwnershipTransferred`.** When an owner left without an explicit transfer, the callback could be skipped. VRChat marked this fixed in client 2023.2.1 (build 1295). Older workarounds that set a one-frame flag in `OnPlayerLeft` are no longer needed. **[Community]** <https://feedback.vrchat.com/udon/p/1258-onownershiptransferred-does-not-fire-at-onplayerleft-if-last-owner-is-passi>
- **`Networking.SetOwner` to a "third-party" target requires that target to accept the transfer.** A vetoed transfer fires `OnOwnershipRequest` but does *not* fire `OnOwnershipTransferred`.

## See also

- [Sync modes](sync-modes.md)
- [VRCObjectSync](object-sync.md) — automatic Transform/Rigidbody sync rides on the same ownership model.
- [Players → Networking class](../players/networking-class.md)
- [Debugging & network stats](debugging-and-stats.md) — debug view 6 shows each object's owner.
- [References → Community Resources](../references/community-resources.md) — `VRCLockstep` sidesteps ownership entirely.
