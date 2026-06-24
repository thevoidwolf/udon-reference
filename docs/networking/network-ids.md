# Network IDs

VRChat assigns each networked GameObject a stable **Network ID** so clients can refer to the same object across the wire. Most authors never interact with this directly — the SDK assigns IDs automatically at upload — but it's worth knowing the system exists.

## What gets a Network ID

Any GameObject in the scene that:
- Has an `UdonBehaviour` (or `UdonSharpBehaviour` proxy) component, or
- Has a `VRCObjectSync` component, or
- Has a `VRCObjectPool` component, or
- Is otherwise marked as networked by an SDK3 component.

## When IDs are assigned

The SDK assigns network IDs **at build/upload time** based on a deterministic scan of the scene. The IDs are baked into the scene asset. Pre-placed scene objects therefore have stable IDs across clients.

Runtime `Instantiate` clones do **not** receive a network ID — they are local-only. To spawn networked GameObjects you must use `VRCObjectPool` (whose pre-placed children already have IDs) or a Player Object.

## Why you might care

In practice you don't read or write Network IDs from U#. The main consequence to be aware of:

- **Adding or removing networked components from your scene shifts every later ID.** If you have synced state and you reorder networked GameObjects between uploads, clients on the old version of the world may end up applying state to the wrong objects when they connect to the new one. Worlds in the wild manage this by avoiding reordering after launch.

## Manual network ID assignment

Newer versions of the SDK (3+) offer a manual network-ID assignment dialog in the SDK control panel. Use it when you need to reorder networked objects without breaking persistent state for existing players. Details: <https://creators.vrchat.com/worlds/udon/networking/network-ids> — note the URL may 404 if VRChat has moved it; the relevant tool is under **VRChat SDK → Show Control Panel → Builder → Network IDs**. **[Unverified]** as docs path.

## See also

- [VRCObjectPool](object-pool.md) — for shared spawning, pools provide stable IDs for pool members.
- [Persistence](../persistence/README.md) — Player Objects also have stable IDs.
