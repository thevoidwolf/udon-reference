# Dynamics Events

**(SDK 3.10.0+)** VRChat Dynamics — PhysBones, Contacts and VRC Constraints — are supported in world projects, with Udon APIs and events. **[Docs]** This page covers the events an `UdonSharpBehaviour` can override. Component properties and methods (`ApplyConfigurationChanges()`, `CalculateProximity()`, `ReleaseGrabs()`, …) are documented on the official pages linked below.

Signatures are from the official docs (<https://creators.vrchat.com/common-components/physbones/>, <https://creators.vrchat.com/common-components/contacts/>). These events are not in the public UdonSharp repo, only in the copy bundled with the Worlds SDK. **[Docs]**

> **Placement rule:** the behaviour must be on the **same GameObject** as the `VRCPhysBone` or `VRCContactReceiver` component that raises the event. **[Docs]**

---

## PhysBone events

`VRCPhysBone` lives in `VRC.SDK3.Dynamics.PhysBone.Components`. **[Docs]**

```csharp
public override void OnPhysBoneGrabbed(PhysBoneGrabbedInfo physBoneInfo)
public override void OnPhysBoneReleased(PhysBoneReleasedInfo physBoneInfo)
public override void OnPhysBonePosed(PhysBonePosedInfo physBoneInfo)
public override void OnPhysBoneUnPosed(PhysBoneUnPosedInfo physBoneInfo)
```

| Event | Fires when |
|---|---|
| `OnPhysBoneGrabbed` | A player grabs the PhysBone. |
| `OnPhysBoneReleased` | A player's grab is released. |
| `OnPhysBonePosed` | A player locks the PhysBone into a pose. |
| `OnPhysBoneUnPosed` | The pose is released. |

**[Docs]**

- The fields of the `PhysBone…Info` structs and their namespace are not documented. The grab info carries the grabbing player: SDK 3.10.0 fixed ClientSim "not receiving the player that initiated the grab". **[Unverified]** for exact member names. Check them in your IDE.
- To poll state instead, read `IsGrabbed` / `IsPosed`, and `Angle` / `Squish` / `Stretch`. **[Docs]**
- `ReleaseGrabs()` / `ReleasePoses()` act on the **local client only**. To release for everyone, call them from a network event. **[Docs]**
- PhysBones created with `Object.Instantiate()` simulate, but their grabs and poses are **not synchronized** between clients. **[Docs]** (3.10.0 release notes)

---

## Contact events

`VRCContactSender` / `VRCContactReceiver` live in `VRC.SDK3.Dynamics.Contact.Components`. `ContactEnterInfo` / `ContactExitInfo` live in `VRC.Dynamics`. **[Docs]**

```csharp
using VRC.Dynamics;

public override void OnContactEnter(ContactEnterInfo contactInfo)
public override void OnContactExit(ContactExitInfo contactInfo)
```

| Member | On | Type | Meaning |
|---|---|---|---|
| `contactSender` | Enter, Exit | `ContactSenderProxy` | The sender involved. |
| `contactReceiver` | Enter, Exit | `ContactReceiverProxy` | The receiver involved. |
| `enterVelocity` | Enter | `Vector3` | Relative velocity of sender vs. receiver at contact. |
| `contactPoint` | Enter | `Vector3` | Estimated contact point on the receiver's surface, in world space. |
| `matchingTags` | Enter; Exit **(SDK 3.10.2+)** | `string[]` | Collision tags the two have in common. |

**[Docs]**

The proxy objects expose `isValid` (check it first), `player` (`VRCPlayerApi` owning the contact, or `null` for world contacts), `usage` (`DynamicsUsage`, e.g. `World` / `Avatar`), and `position`, `rotation`, `scale`. Compare a proxy against your own `VRCContactSender` / `VRCContactReceiver` with `==` / `!=`. **[Docs]** Since SDK 3.10.2, `Utilities.IsValid(proxy)` also reports whether the proxy is still valid. **[Docs]**

On a world receiver, the "Allow Self" / "Allow Others" options only apply to senders on avatars. Use **Content Types** to choose whether world, avatar and/or item senders trigger the receiver. **[Docs]**

### Example: react to a player's hand

```csharp
using UdonSharp;
using UnityEngine;
using VRC.Dynamics;
using VRC.SDKBase;

[UdonBehaviourSyncMode(BehaviourSyncMode.None)]
public class TouchButton : UdonSharpBehaviour {
    public override void OnContactEnter(ContactEnterInfo contactInfo) {
        var sender = contactInfo.contactSender;
        if (!sender.isValid) return;
        // world-owned senders have player == null
        if (!Utilities.IsValid(sender.player) || !sender.player.isLocal) return;
        Debug.Log($"Pressed with velocity {contactInfo.enterVelocity.magnitude:F2} m/s");
    }
}
```

Contacts are simulated locally, so expect the event on **every client** that sees the contact. Guard with `player.isLocal` (as above) before doing anything networked. **[Unverified]**

---

## Notes

- Dynamics components work on `VRCPlayerObject`s (fixed in SDK 3.10.0). **[Docs]**
- In SDK 3.10.4+ you can change `VRCPhysBoneCollider` properties at runtime, then call `ApplyConfigurationChanges()`. Config changes are expensive, so batch them. **[Docs]**
- Since SDK 3.10.5, PhysBones with "Reset When Disabled" also reset Udon state when disabled. **[Docs]**

## See also

- [Player events](player-events.md): collider and trigger callbacks for player capsules.
- [Lifecycle events](lifecycle-events.md): the full event index.
- Official: <https://creators.vrchat.com/common-components/physbones/#physbone-udon-access-in-worlds>, <https://creators.vrchat.com/common-components/contacts/#udon-access-in-worlds>
