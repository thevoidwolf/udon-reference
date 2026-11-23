# Avatar Pedestals & Portal Markers

Two long-standing world components with a small Udon surface. Neither is new. The official pages have listed their **Udon property names and methods only since the October 2026 docs sync**, which shipped alongside SDK 3.10.6. Before that you had to find the names in the Class Exposure Tree. **[Docs]**

Sources: <https://creators.vrchat.com/worlds/components/vrc_avatarpedestal/>, <https://creators.vrchat.com/worlds/components/vrc_portalmarker/>. **[Docs]**

## VRCAvatarPedestal

`VRC.SDK3.Components.VRCAvatarPedestal` shows an avatar, and players can interact with it to switch into that avatar. The SDK's example scene has an `AvatarPedestal` prefab that calls `SetAvatarUse` on interact. **[Docs]**

### Properties

| Inspector | Udon property | Type | Notes |
|---|---|---|---|
| Blueprint Id | `blueprintId` | `string` | Avatar ID to show, like `avtr_...`. |
| Placement (optional) | `Placement` | `Transform` | Where the avatar is displayed. |
| Change Avatar On Use | `ChangeAvatarsOnUse` | `bool` | Must be `true` for `SetAvatarUse` to work. Note the plural *Avatars* in the property name. |
| Scale | `scale` | `float` | Display size of the pedestal avatar only. |

Names and meanings from the docs. **[Docs]** All four have getters and setters in the 3.10.6 extern table. **[Source]** `Runtime/Udon/External/VRC.Udon.Wrapper.dll` (com.vrchat.worlds 3.10.6)

### Methods

| Method | Effect |
|---|---|
| `SwitchAvatar(string id)` | Changes which avatar the pedestal **displays**. It does not change anyone's avatar. |
| `SetAvatarUse(VRCPlayerApi instigator)` | Puts the player into the displayed avatar. Only works when `instigator` is the **local player** and Change Avatar On Use is enabled. |

**[Docs]**

So "show a different avatar" is `SwitchAvatar`, and "wear it" is `SetAvatarUse(Networking.LocalPlayer)`. You can't put a remote player into an avatar. Each player triggers their own switch. **[Docs]**

What other players see depends on the avatar's release status: **[Docs]**

- **Public avatar:** everyone can see the pedestal and use it.
- **Private avatar:** only the uploader can see and use it. Everyone else sees an error.
- **Marketplace avatar:** everyone can see it. Players who don't own the avatar get its details page instead of switching.

### Example (U#)

Cycles a pedestal through a list of avatar IDs each time a player interacts. A second interact target lets the player wear the one on display.

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDKBase;
using VRC.SDK3.Components;

public class PedestalCycler : UdonSharpBehaviour
{
    public VRCAvatarPedestal pedestal;
    public string[] avatarIds;
    private int _index;

    public override void Interact()
    {
        if (avatarIds.Length == 0) return;
        _index = (_index + 1) % avatarIds.Length;
        pedestal.SwitchAvatar(avatarIds[_index]);   // display only, local to this client
    }

    // Hook this to a "Wear" button (UI event or a separate interactable).
    public void Wear()
    {
        pedestal.SetAvatarUse(Networking.LocalPlayer);  // needs Change Avatar On Use
    }
}
```

`SwitchAvatar` runs only on the client that calls it. Like any component write in Udon, it doesn't sync. If everyone should see the same avatar on display, sync the index and call `SwitchAvatar` from a `FieldChangeCallback` or `OnDeserialization`. **[Community-known]** See [Networking](../networking/README.md).

## VRCPortalMarker

`VRC.SDK3.Components.VRCPortalMarker` places a portal to another world, the player's home world, or the Hub. **[Docs]**

| Inspector | Udon property | Notes |
|---|---|---|
| World ID | `roomId` (`string`) | Destination world ID, like `wrld_...`. |
| Custom Portal Name | none | Overrides the world name shown above the portal. Not exposed to Udon. |
| World | none | `None` (use World ID), `Home` or `Hub`. Not exposed to Udon. |

| Method | Effect |
|---|---|
| `RefreshPortal()` | Rebuilds the portal. Call it after changing `roomId` at runtime, or the portal keeps pointing at the old world. |

**[Docs]** `roomId` get/set and `RefreshPortal()` are in the 3.10.6 extern table. **[Source]**

```csharp
public VRCPortalMarker portal;

public void PointAt(string worldId)
{
    portal.roomId = worldId;
    portal.RefreshPortal();
}
```

As with the pedestal, setting `roomId` only changes the portal on the client that runs it. Sync the ID if every player should see the same destination. **[Community-known]**

`roomId` is a plain `string`, unlike URLs, which need a `VRCUrl`. The docs don't say whether the client validates or restricts destinations set at runtime, for example private worlds or malformed IDs. **[Unverified]**

## Related: placement zones

Player-placed portals, stickers and sharing pedestals are a separate system. Control where players can put them with [VRCPlacementZone](placement-zone.md) (3.10.6). **[Docs]**

## See also

- [World Components overview](README.md)
- [Players → Pickups & Stations](../players/pickups-stations.md): the other world components with an Udon API (`VRCPickup`, `VRCStation`).
- [Players → VRCPlayerApi](../players/vrc-player-api.md): `Networking.LocalPlayer` and player checks.
- [Attributes → FieldChangeCallback](../attributes/field-change-callback.md): reacting to a synced avatar index or world ID.
