# Network IDs

VRChat assigns each networked GameObject a **Network ID**, a number that lets clients refer to the same object across the wire. Most authors never touch it directly because the SDK assigns IDs automatically. It matters mainly for **cross-platform worlds built from different scenes or projects**, where every version must give the same object the same ID. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-id-utility/>

## What gets a Network ID

Any GameObject in the scene that:
- Has an `UdonBehaviour` (or `UdonSharpBehaviour` proxy) component, or
- Has a `VRCObjectSync` component, or
- Has a `VRCObjectPool` component, or
- Is otherwise marked as networked by an SDK3 component.

## When IDs are assigned

The SDK assigns network IDs in the editor and stores them with the scene. The `VRC Scene Descriptor` inspector displays them (a display bug was fixed in SDK 3.7.0). **[Docs]** <https://creators.vrchat.com/releases/release-3-7-0/> Pre-placed scene objects therefore have stable IDs across clients. The exact assignment algorithm isn't documented. **[Unverified]**

Runtime `Instantiate` clones do **not** receive a network ID — they are local-only. To spawn networked GameObjects you must use `VRCObjectPool` (whose pre-placed children already have IDs) or a Player Object.

## Why you might care

In practice you don't read or write Network IDs from U#. The main consequences:

- **Cross-platform scenes must agree.** If your PC and Android/iOS builds come from different scenes or projects, mismatched IDs mean players on different platforms apply data to the wrong objects ("you kick a beach ball, they see you kick an ice cream"). **[Docs]**
- **World-update compatibility.** An object that exists in both the old and new world version is *incompatible* if its component types, count **or** order differ, or its synced variable types, count **or** order differ. Players who join an *active* instance launched on the old version are notified and removed. Once an instance empties and goes inactive, rejoining uses the new version without issue. Adding a component or a synced variable to a synced object will likely make the next upload incompatible. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/compatibility/>
- **Component order matters at runtime too.** `[NetworkCallable]` events target components by index, so don't `Destroy` components on GameObjects with networked `UdonBehaviour`s. **[Docs]** See [Custom Network Events](custom-network-events.md#legacy-events-and-security).

## Network ID Import and Export Utility

Menu: **`VRChat SDK/Utilities/Network ID Import and Export Utility`**. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-id-utility/>

VRChat says you should only need it for cross-platform worlds whose versions live in different scenes or projects. **[Docs]**

- Lists every network ID in the scene. **Regenerate Scene IDs** creates them if the scene has none yet.
- **Export** writes a JSON file mapping ID → hierarchy path, e.g. `{"10":"/CubePickup", "11":"/Prefabs/SyncedPen"}`. **Import** applies a file to another scene. Networked objects must have the **same hierarchy paths** in both scenes. Non-networked objects can differ.
- If everything matches, a single **Accept All** block appears.
- **Conflict resolution**: *Scan For Conflicts* (or *Auto Scan*) finds paths that are missing from the scene and IDs already used by another object. Resolve each with **Ignore** / **Ignore All**, or **Select** a different object. When copying IDs between scenes, clear the existing IDs before importing to avoid most conflicts.
- **Don't use `/` in names of networked GameObjects.** Paths use `/` as the delimiter. Since SDK 3.9.0 the tool warns about this on export. **[Docs]**

All of the above is **[Docs]** from the utility page unless noted.

### Utility changelog

All items in this section: **[Docs]**.

| SDK | Change |
|---|---|
| 3.6.1 | Fixed the utility getting confused by duplicate GameObject paths. Shared hierarchy paths for networked objects are still "not recommended or fully supported". |
| 3.7.0 | Network IDs display correctly in the Scene Descriptor inspector again. |
| 3.9.0 (Oct 6, 2025) | Faster UI in very large projects. Fixed an exception that could make imports fail. Warns on export about `/` in GameObject names. |

## See also

- [VRCObjectPool](object-pool.md) — for shared spawning, pools provide stable IDs for pool members.
- [Persistence](../persistence/README.md) — Player Objects are instantiated per player by the SDK.
- [Debugging & network stats](debugging-and-stats.md) — debug view 6 lists networked objects.
