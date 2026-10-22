# Official Docs

VRChat-published reference pages cited throughout this repo. Every URL below was checked against <https://creators.vrchat.com/> on **2026-10-22** (current Worlds SDK **3.10.5**, Unity **2022.3.22f1**). VRChat occasionally restructures the docs site — see [Note on links](#note-on-links).

## Top-level

- **Worlds → Udon overview**: <https://creators.vrchat.com/worlds/udon/>
- **UdonSharp dedicated docs**: <https://udonsharp.docs.vrchat.com/>
- **UdonSharp section on the creators site** (states U# "is included in the VRChat worlds SDK"): <https://creators.vrchat.com/worlds/udon/udonsharp/>
  - Attributes: <https://creators.vrchat.com/worlds/udon/udonsharp/attributes>
  - Editor scripting: <https://creators.vrchat.com/worlds/udon/udonsharp/editorscripting>
  - Performance tips: <https://creators.vrchat.com/worlds/udon/udonsharp/performance-tips>
- **Udon Graph**: <https://creators.vrchat.com/worlds/udon/graph/>
- **VM and Assembly**: <https://creators.vrchat.com/worlds/udon/vm-and-assembly/>

## SDK, releases & Unity version

- **Release notes (all SDK versions)**: <https://creators.vrchat.com/releases/>
  - Current release, 3.10.5: <https://creators.vrchat.com/releases/release-3-10-5>
  - 3.4.0 (UdonSharp and ClientSim merged into the Worlds package): <https://creators.vrchat.com/releases/release-3-4-0>
- **Current Unity version** (2022.3.22f1 as of this check): <https://creators.vrchat.com/sdk/upgrade/current-unity-version/>
- **Updating the SDK**: <https://creators.vrchat.com/sdk/updating-the-sdk>
- **Build pipeline callbacks and interfaces**: <https://creators.vrchat.com/sdk/build-pipeline-callbacks-and-interfaces>
- **Public SDK API**: <https://creators.vrchat.com/sdk/public-sdk-api>

## Concepts

- **Event execution order**: <https://creators.vrchat.com/worlds/udon/event-execution-order/>
- **Debugging Udon projects**: <https://creators.vrchat.com/worlds/udon/debugging-udon-projects>
- **World debug views**: <https://creators.vrchat.com/worlds/udon/world-debug-views>
- **Using Build & Test**: <https://creators.vrchat.com/worlds/udon/using-build-test>
- **Layers** (incl. the `Item` layer, formerly `reserved3`): <https://creators.vrchat.com/worlds/layers>
- **ClientSim**: <https://creators.vrchat.com/worlds/clientsim/>

## Players

- **Players overview / VRCPlayerApi**: <https://creators.vrchat.com/worlds/udon/players/>
- **Getting players** (`GetPlayers`, `GetPlayerById`, …): <https://creators.vrchat.com/worlds/udon/players/getting-players>
- **Player tags**: <https://creators.vrchat.com/worlds/udon/players/getting-players#player-tag-system> (the old `/players/player-tags` page is gone; content now lives on *Getting Players*)
- **Player positions**: <https://creators.vrchat.com/worlds/udon/players/player-positions>
- **Player audio**: <https://creators.vrchat.com/worlds/udon/players/player-audio>
- **Player forces**: <https://creators.vrchat.com/worlds/udon/players/player-forces>
- **Player collisions**: <https://creators.vrchat.com/worlds/udon/players/player-collisions>
- **Player avatar scaling**: <https://creators.vrchat.com/worlds/udon/players/player-avatar-scaling>
- **Drones** (SDK 3.8.0+): <https://creators.vrchat.com/worlds/udon/players/drones/>

## Networking

- **Networking overview**: <https://creators.vrchat.com/worlds/udon/networking/>
- **Network events** (incl. parameterised events and `[NetworkCallable]`, SDK 3.8.1+): <https://creators.vrchat.com/worlds/udon/networking/events/>
- **Network variables**: <https://creators.vrchat.com/worlds/udon/networking/variables>
- **Ownership**: <https://creators.vrchat.com/worlds/udon/networking/ownership/>
- **Late joiners**: <https://creators.vrchat.com/worlds/udon/networking/late-joiners>
- **Network components (ObjectSync, ObjectPool)**: <https://creators.vrchat.com/worlds/udon/networking/network-components/>
- **Network ID utility** (replaces the old `/networking/network-ids` page): <https://creators.vrchat.com/worlds/udon/networking/network-id-utility/>
- **Network details**: <https://creators.vrchat.com/worlds/udon/networking/network-details>
- **Network stats**: <https://creators.vrchat.com/worlds/udon/networking/network-stats>
- **Networking performance**: <https://creators.vrchat.com/worlds/udon/networking/performance>
- **Networking debugging**: <https://creators.vrchat.com/worlds/udon/networking/debugging>
- **Compatibility (mismatched world versions)**: <https://creators.vrchat.com/worlds/udon/networking/compatibility>

## Persistence

- **Persistence overview**: <https://creators.vrchat.com/worlds/udon/persistence/>
  - Storage limits (100 KB PlayerData + 100 KB PlayerObject data per player per world): <https://creators.vrchat.com/worlds/udon/persistence/#limitations>
- **PlayerData**: <https://creators.vrchat.com/worlds/udon/persistence/player-data/> (was `/persistence/playerdata`)
  - Storage usage API (`GetPlayerDataStorageUsage`, `GetPlayerDataStorageLimit`, `RequestStorageUsageUpdate`; SDK 3.10.0+): <https://creators.vrchat.com/worlds/udon/persistence/player-data/#storage-information>
- **Player Objects**: <https://creators.vrchat.com/worlds/udon/persistence/player-object/> (was `/persistence/player-objects`)
- **Persistence examples**: <https://creators.vrchat.com/worlds/examples/persistence/>

## Data containers

- **Data containers overview**: <https://creators.vrchat.com/worlds/udon/data-containers/>
- **DataToken**: <https://creators.vrchat.com/worlds/udon/data-containers/data-tokens>
- **DataList**: <https://creators.vrchat.com/worlds/udon/data-containers/data-lists>
- **DataDictionary**: <https://creators.vrchat.com/worlds/udon/data-containers/data-dictionaries>
- **VRCJson**: <https://creators.vrchat.com/worlds/udon/data-containers/vrcjson>
- **Byte and bit operations**: <https://creators.vrchat.com/worlds/udon/data-containers/byte-and-bit-operations>

## External content

- **String loading**: <https://creators.vrchat.com/worlds/udon/string-loading>
- **Image loading**: <https://creators.vrchat.com/worlds/udon/image-loading>
- **External URLs**: <https://creators.vrchat.com/worlds/udon/external-urls>

## Multimedia

- **Video players**: <https://creators.vrchat.com/worlds/udon/video-players/>
- **Video URL allowlist**: <https://creators.vrchat.com/worlds/udon/video-players/www-whitelist>
- **MIDI**: <https://creators.vrchat.com/worlds/udon/midi/>
  - MIDI playback: <https://creators.vrchat.com/worlds/udon/midi/midi-playback>
  - Realtime MIDI: <https://creators.vrchat.com/worlds/udon/midi/realtime-midi>

## Graphics

- **VRCGraphics**: <https://creators.vrchat.com/worlds/udon/vrc-graphics/>
- **VRCCameraSettings** (Screen + Photo camera; SDK 3.8.1+): <https://creators.vrchat.com/worlds/udon/vrc-graphics/vrc-camera-settings/>
- **VRCQualitySettings** (SDK 3.8.1+): <https://creators.vrchat.com/worlds/udon/vrc-graphics/vrc-quality-settings/>
- **AsyncGPUReadback**: <https://creators.vrchat.com/worlds/udon/vrc-graphics/asyncgpureadback>
- **VRChat shader globals**: <https://creators.vrchat.com/worlds/udon/vrc-graphics/vrchat-shader-globals>
- **Camera Dolly** (`VRC_CameraDolly`; SDK 3.9.0+): <https://creators.vrchat.com/worlds/components/vrc_cameradolly/>

## VRChat Dynamics in worlds (SDK 3.10.0+)

PhysBones, Contacts and VRC Constraints are documented under *Common Components* ("work both on avatars and in worlds"):

- **Common Components overview**: <https://creators.vrchat.com/common-components/>
- **PhysBones**: <https://creators.vrchat.com/common-components/physbones/> — see *PhysBone Udon Access in Worlds* (`#physbone-udon-access-in-worlds`) and *Collider Udon Access in Worlds* (`#collider-udon-access-in-worlds`).
- **Contacts**: <https://creators.vrchat.com/common-components/contacts/> — see *Receivers in Worlds* and *Udon Access in Worlds* (`#udon-access-in-worlds`).
- **Constraints**: <https://creators.vrchat.com/common-components/constraints/>
  - Constraints API: <https://creators.vrchat.com/common-components/constraints/constraints-api>

## Components

- **VRC_Pickup**: <https://creators.vrchat.com/worlds/components/vrc_pickup>
- **VRC_ObjectSync**: <https://creators.vrchat.com/worlds/components/vrc_objectsync>

## UI / Input / Animation

- **Input events**: <https://creators.vrchat.com/worlds/udon/input-events>
- **UI events**: <https://creators.vrchat.com/worlds/udon/ui-events>
- **Animation events**: <https://creators.vrchat.com/worlds/udon/animation-events>
- **Avatar events**: <https://creators.vrchat.com/worlds/udon/avatar-events>

## Tweening

- **VRCTween** (SDK 3.10.4+): <https://creators.vrchat.com/worlds/udon/vrctween/>
  - Tween types: <https://creators.vrchat.com/worlds/udon/vrctween/tween-types>
  - Virtual tweens: <https://creators.vrchat.com/worlds/udon/vrctween/virtual-tweens>
  - Settings: <https://creators.vrchat.com/worlds/udon/vrctween/settings>

## Other

- **AI Navigation**: <https://creators.vrchat.com/worlds/udon/ai-navigation>
- **Udon moderation tool guidelines**: <https://creators.vrchat.com/worlds/udon/udon-moderation-tool-guidelines>
- **Creator Economy — Udon documentation**: <https://creators.vrchat.com/economy/sdk/udon-documentation>

## Note on links

VRChat moves pages periodically. If a link 404s, the content has usually moved to a similar URL — search at <https://creators.vrchat.com/>, check the docs sidebar, or look the path up in <https://creators.vrchat.com/sitemap.xml>.

Pages moved since the previous revision of this list (old URLs now 404):

| Old URL | Current URL |
|---|---|
| `/worlds/udon/players/player-tags` | `/worlds/udon/players/getting-players#player-tag-system` |
| `/worlds/udon/networking/network-ids` | `/worlds/udon/networking/network-id-utility/` |
| `/worlds/udon/persistence/playerdata` | `/worlds/udon/persistence/player-data/` |
| `/worlds/udon/persistence/player-objects` | `/worlds/udon/persistence/player-object/` |

## See also

- [Source Map](source-map.md)
- [Community Resources](community-resources.md)
