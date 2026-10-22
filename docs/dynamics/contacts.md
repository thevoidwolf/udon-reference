# Contacts

Contacts are VRChat's own collision-detection volumes and are separate from Unity colliders. A **Sender** exists only to be detected. A **Receiver** detects Senders. On avatars a Receiver drives an animator parameter. **In worlds it raises Udon events** on the behaviours attached to the same GameObject. **[Docs]**

Source: <https://creators.vrchat.com/common-components/contacts>. **[Docs]**

## Components

| Component | Namespace |
|---|---|
| `VRCContactSender` | `VRC.SDK3.Dynamics.Contact.Components` **[Docs]** |
| `VRCContactReceiver` | `VRC.SDK3.Dynamics.Contact.Components` **[Docs]** |

### Shape

| Setting | Notes |
|---|---|
| `Shape Type` | Sphere, Capsule or **Box**. Box was added in 3.10.4 for avatars and worlds. **[Docs]** |
| `Radius` | Sphere and Capsule. **[Docs]** |
| `Height` | Capsule. Includes the end caps. **[Docs]** |
| `Size` | Box. Each axis is independent. **[Docs]** |
| `Position` / `Rotation` | Offset from the root transform. **[Docs]** |

Limits: 3 m maximum radius and 6 m maximum width/height/depth, applied after scaling. **[Docs]**

Box Receivers have a **`Use Face Proximity`** option. It measures proximity to the box's **positive-Z face** instead of its centre. In worlds, this is the only Receiver setting you can configure. **[Docs]**

### Filtering

- **`Content Types`** (world contacts only) sets whether the contact interacts with **world**, **avatar** and/or **item** contacts. **[Docs]**
- **`Collision Tags`** are case-sensitive strings, with at most 16 per contact. A contact fires when the Sender and Receiver share at least one tag. VRChat recommends PascalCase for custom tags. **[Docs]**
- `Allow Self` / `Allow Others` / `Local Only` are avatar-only. Since 3.10.0, a Receiver's Allow Self/Others apply only to **avatar** Senders and are ignored for world Senders. **[Docs]**

### Built-in tags

The **Body Parts** tags are the only ones with special handling in the client. When a humanoid avatar loads, Senders tagged `Head`, `Torso`, `Hand`, `HandL`, `HandR`, `Foot`, `FootL`, `FootR`, `Finger`, … are generated on its bones automatically if the author didn't define them. A world Receiver with those tags detects any player's body parts, which suits push buttons and head-pats. **[Docs]** For the full list, see <https://creators.vrchat.com/common-components/contacts/built-in-contact-tags>.

## Events

Attach the behaviour to the **same GameObject as the `VRCContactReceiver`**. **[Docs]**

```csharp
OnContactEnter(ContactEnterInfo contactInfo)  // a Sender starts contacting the Receiver
OnContactExit(ContactExitInfo contactInfo)    // a Sender stops contacting the Receiver
```

The docs signatures are quoted above. `ContactEnterInfo` and `ContactExitInfo` are in `VRC.Dynamics`. **[Docs]** The U# form is `public override void ...`. **[Unverified]** (see [PhysBones → Events](physbones.md#events)).

### ContactEnterInfo

| Field | Type | Meaning |
|---|---|---|
| `contactSender` | `ContactSenderProxy` | The Sender involved. **[Docs]** |
| `contactReceiver` | `ContactReceiverProxy` | The Receiver involved. **[Docs]** |
| `enterVelocity` | `Vector3` | Sender velocity relative to the Receiver at contact. **[Docs]** |
| `contactPoint` | `Vector3` | Estimated contact point on the Receiver's surface, in world space. **[Docs]** |
| `matchingTags` | `string[]` | Tags the Sender and Receiver share. **[Docs]** |

### ContactExitInfo

| Field | Type | Meaning |
|---|---|---|
| `contactSender` | `ContactSenderProxy` | **[Docs]** |
| `contactReceiver` | `ContactReceiverProxy` | **[Docs]** |
| `matchingTags` | `string[]` | **Added in 3.10.2.** Before that it was only available on enter. **[Docs]** |

### Contact proxies

`contactSender` and `contactReceiver` are **proxy objects**, not the components themselves. To check whether a proxy refers to one of your own `VRCContactSender`/`VRCContactReceiver` components, compare it with `==` / `!=`. **[Docs]**

| Proxy property | Type | Notes |
|---|---|---|
| `isValid` | `bool` | Check this before you read anything else. **[Docs]** |
| `player` | `VRCPlayerApi` | The owning player. **`null` for world contacts.** **[Docs]** |
| `usage` | `DynamicsUsage` | `World` for your world's contacts, `Avatar` for avatar contacts. **[Docs]** |
| `position` | `Vector3` | World-space position of the root transform. **[Docs]** |
| `rotation` | `Quaternion` | World-space rotation of the root transform. **[Docs]** |
| `scale` | `Vector3` | Lossy scale of the root transform. **[Docs]** |

Since **3.10.2**, `Utilities.IsValid(proxy)` gives accurate results for Sender and Receiver proxies. Earlier SDKs could report them wrongly. **[Docs]**

The docs don't list the other `DynamicsUsage` values. Items are a documented content type, so an item value probably exists, but it isn't named. **[Unverified]**

## Runtime configuration

| Member | Notes |
|---|---|
| `radius`, `height`, `size`, `position`, `rotation` | Read and write on Sender and Receiver. **[Docs]** |
| `ApplyConfigurationChanges()` | Must be called after writing the fields above. Expensive, so batch writes. **[Docs]** |
| `CalculateProximity()` | On the **Receiver**. Returns a float in `0.0–1.0` for how close the Sender is to the Receiver's centre, or to its +Z face for a box with `Use Face Proximity`. The docs give the name with empty parentheses and don't say how the Sender is chosen or whether it takes a parameter. **[Docs]** / parameters **[Unverified]** |
| `UpdateContentTypes(DynamicsUsageFlags newContentTypes)` | Changes the allowed content types. **[Docs]** (see note below) |
| `UpdateCollisionTags(string[] newCollisionTags)` | Replaces the tag list. Only the first 16 apply. **[Docs]** |

### Updating content types and tags

> **Discrepancy:** The 3.10.0 release notes say: "Changed `VRCContactReceiver.UpdateContentTypes()` to take an array of strings instead of an `IEnumerable` of strings." They add that U# code passing anything other than a `string[]` must be updated. However, the **current** Contacts page lists `UpdateContentTypes(DynamicsUsageFlags newContentTypes)`, and the `string[]` signature belongs to **`UpdateCollisionTags`**. The docs don't reconcile the two. It may have been a later change, or the release note may have named the wrong method. To be safe, pass a real `string[]` to the tag method (not a `DataList` or other enumerable), and check the content-types overload in the Udon Class Exposure Tree for your SDK version. **[Docs]** (both statements) / resolution **[Unverified]**

## Example: a hand-pressed button

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDKBase;
using VRC.Dynamics;

// Put this on the same GameObject as a VRCContactReceiver whose
// Collision Tags include "Hand" or "Finger" and whose Content Types include Avatar.
public class TouchButton : UdonSharpBehaviour
{
    public override void OnContactEnter(ContactEnterInfo contactInfo)
    {
        if (!contactInfo.contactSender.isValid) return;
        if (contactInfo.contactSender.usage != DynamicsUsage.Avatar) return;

        VRCPlayerApi presser = contactInfo.contactSender.player;
        if (!Utilities.IsValid(presser) || !presser.isLocal) return;   // react to our own hand only

        Debug.Log($"Pressed by {presser.displayName} at {contactInfo.contactPoint}");
        // Broadcast with SendCustomNetworkEvent if everyone should see the result.
    }
}
```

The `isLocal` filter matters because every client sees every avatar's hand Senders, so without it each client's Receiver would fire for a press by anyone. This is an inference from contacts being simulated per client, not a documented statement. **[Unverified]** The example also assumes `DynamicsUsage` resolves from `VRC.Dynamics`, but its namespace isn't documented. **[Unverified]**

## See also

- [Dynamics overview](README.md)
- [PhysBones](physbones.md): avatar hand and finger colliders double as global PhysBone colliders.
- [Constraints](constraints.md)
- [UdonSharpBehaviour](../udonsharp-behaviour/README.md)
- [Players → VRCPlayerApi](../players/vrc-player-api.md): the proxy's `player`.
- [Networking](../networking/README.md) and [Custom network events](../networking/custom-network-events.md)
- [Persistence → Player Objects](../persistence/player-objects.md)
