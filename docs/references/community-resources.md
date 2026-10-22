# Community Resources

A curated index of community-built libraries, tools, simulators, and documentation that supplement the official VRChat / UdonSharp ecosystem. Each entry has a source URL. **Status flags** were last checked on **2026-10-22** (Worlds SDK 3.10.5):

- **Active** — commits within the last ~12 months.
- **Stale** — repo still up, but no commits for more than ~12 months.
- **Maintenance** — author has explicitly stopped feature work.
- **Archived** — repository archived (read-only) on GitHub.

Versions are the latest stable git tag/release and its date; "last commit" is the default branch's most recent commit.

> All entries on this page are **[Community]** unless otherwise noted. Verify currency before adopting — the ecosystem moves quickly and some flagship projects are archived after VRChat ships a built-in equivalent.

---

## Networking libraries

| Project | What it does | Source | Status |
|---|---|---|---|
| **VRCLockstep** (JanSharp) | Deterministic lockstep networking: broadcasts ordered input actions; every client runs the same ticks in the same order. Eliminates ownership race conditions; supports large mutating state via byte-array snapshots for late joiners. | <https://github.com/JanSharp/VRCLockstep> | Active (v1.1.3, Oct 2026) |
| **UNet** (Xytabich) | Reliable binary RPC layer over Udon. `ByteBufferWriter`/`Reader`, sequenced or unordered modes, per-player targeting, message-ID acks. Predates SDK 3.8.1's `[NetworkCallable]`. | <https://github.com/Xytabich/UNet> | Archived (last commit Jun 2023, u2.1.0) |
| **UdonNet** (poi-vrc) | WIP P2P Udon networking. Less mature than UNet. | <https://github.com/poi-vrc/UdonNet> | Stale — never left WIP (last commit May 2021) |
| **SimpleNetwork** (tutinoco) | Wrapper simplifying typed networking. | <https://github.com/tutinoco/SimpleNetwork> | Stale (v1.8.1, Oct 2023) |
| **Smart Object Sync** (MMMaellon) | Drop-in `VRCObjectSync` replacement with better pickup/physics sync. Used by JanSharp's Cannon Grip (a VPM dependency). The author now points new projects to **LightSync** (<https://github.com/MMMaellon/LightSync>, active, 1.6.x betas Oct 2026). | <https://github.com/MMMaellon/SmartObjectSync> | Maintenance (v3.10.16, Mar 2025; "stable, but deprecated") |
| **UdonUtils** (Guribo) | `TlpNetworkTime`, `TlpAccurateSyncBehaviour`, state machines, rigidbody recording/prediction, `TlpLogger`. Requires World SDK 3.10.3+. | <https://github.com/Guribo/UdonUtils> | Active (v15.0.3, Sep 2026) |

## Per-player state

| Project | What it does | Source | Status |
|---|---|---|---|
| **CyanPlayerObjectPool** | Master-authoritative per-player GameObject pool. Was *the* canonical pattern for per-player state pre-PlayerObject. Master-migration safe. | <https://github.com/CyanLaser/CyanPlayerObjectPool> | **Archived** (Oct 2024; last release v1.1.2, May 2023) — superseded by stock `VRCPlayerObject` (SDK 3.7.4). Still in many older worlds. |
| **LSNPlayerObjectPool** | Fork of CyanPlayerObjectPool. | <https://github.com/LastationVRChat/LSNPlayerObjectPool> | Stale — fork with no commits beyond upstream's May 2023 state |
| **NUSaveState** (Nestorboy) | Pre-PlayerData persistence hack using avatar parameters as a storage substrate (read back via bone rotations). Still the only practical *cross-world* persistence option — stock PlayerData/PlayerObject persistence is per-world. | <https://github.com/Nestorboy/NUSaveState> | Stale (v1.6.0, Jan 2024) |

## Video players

VRChat's video-player page points to VideoTXL, ProTV, and USharpVideo as its suggested community prefabs (still true Oct 2026): <https://creators.vrchat.com/worlds/udon/video-players/>.

| Project | What's distinctive | Source | Status |
|---|---|---|---|
| **USharpVideo** (Merlin) | Network-time-anchored sync model. Manual sync for late joiners. AVPro + Unity backends. | <https://github.com/MerlinVR/USharpVideo> | Stale (v1.0.1 Oct 2023; last commit May 2024) |
| **USharpVideoQueue** (sam-ln) | Synced playlist queue add-on for USharpVideo. | <https://github.com/sam-ln/USharpVideoQueue> | Active (v0.10.0, Aug 2026) |
| **VideoTXL** | Three-manager architecture (Video / Audio / Screen). Ships Sync **and** Local prefabs. AudioLink + VRSL integration. | <https://github.com/vrctxl/VideoTXL>, <https://vrctxl.github.io/Docs/> | Active (v2.5.1 Jul 2026; 2.6.0 betas Sep 2026) |
| **ProTV** (ArchiTech) | Event-driven plugin architecture. Multi-config swap. LTCGI adapters. | <https://gitlab.com/techanon/protv>, <https://protv.dev/faq> | Active (v3.0.0 betas, Oct 2026; last stable tag v2.3.15) |
| **SynMediaPlayer** | Gapless playback, automatic retry on failure, low-latency streams. | <https://github.com/synergiance/SynMediaPlayer> | Stale (v1.2.5, Mar 2024) |
| **VVMW / VizVid** (JLChnToZ) | Per-platform URL pairs (PC vs Quest), RTSP/RTMP, request debouncing. | <https://github.com/JLChnToZ/VVMW> | Active (v1.8.2, Sep 2026) |
| **RiskiPlayer** | Modified ProTV with on-screen UI. | <https://github.com/RiskiVR/RiskiPlayer> | Stale (v3.1 Nov 2023; last commit Aug 2024) |
| **USharpVideoModernUI** (DrBlackRat) | UI replacement for USharpVideo. | <https://github.com/DrBlackRat/USharpVideoModernUI>, <https://vpm.drblackrat.xyz/> | Stale (v1.1.2, Nov 2024) |
| **YamaPlayer** (koorimizuw) | Synced video player with in-screen modern UI, play queue, history, playlists (incl. dynamic), speed control, karaoke mode (beta), LTCGI support. Japanese-first docs with English/Spanish READMEs. | <https://github.com/koorimizuw/YamaPlayer> | Active (v1.5.19 Oct 2026; 2.0 betas) |

## Graphics / shaders

| Project | What it does | Source | Status |
|---|---|---|---|
| **AudioLink** (llealloo + collaborators) | World-audio-to-shader bridge. Broadcasts `_AudioTexture` (the *only* non-`_Udon` prefix VRChat allowlists for `VRCShader.SetGlobal*`). Cross-platform (VRC/CVR/Poligon). | <https://github.com/llealloo/audiolink> | Active (3.1.2 Feb 2026; commits Oct 2026) |
| **LTCGI** (PiMaker) | Real-time area lighting via Linearly Transformed Cosines. UdonSharp integration; ProTV adapter. Quest caps at 2 screens + FAST_SAMPLING. | <https://github.com/PiMaker/ltcgi>, <https://ltcgi.dev/> | Active (v1.7.4, Oct 2026) |
| **shadertrixx** (cnlohr) | Reference notes on VRChat shader tricks: mirror depth, GPU particles, RT loop techniques, mesh-bounds gotchas. | <https://github.com/cnlohr/shadertrixx> | Active (last commit Mar 2026) |
| **Poiyomi Toon** | Most popular feature-rich toon shader. | <https://github.com/poiyomi/PoiyomiToonShader> | Active (v10.0.27, Oct 2026) |
| **Mochie's Unity Shaders** | Standard, water, post-processing variants. | <https://github.com/MochiesCode/Mochies-Unity-Shaders> | Active (tag v.167.2 Nov 2025; commits Oct 2026) |
| **z3y / Okabintaro shaders** | Standard/Surface replacement; first to ship Quest box-projected AreaLit + LTCGI. | <https://github.com/z3y/shaders>, <https://github.com/Okabintaro/z3y_shaders> | **z3y/shaders archived** — successor is Graphlit (<https://github.com/z3y/Graphlit>, Active, v2.7.4 Sep 2026). Okabintaro fork Stale (v3.1.8, Jun 2023). |
| **Silent's Cel Shading** | Lightweight cel shader respecting Unity lighting. | <https://ask.vrchat.com/t/silents-cel-shading-shader/3234> | — |
| **Neitri-Unity-Shaders** | Wireframe overlays and utilities. | <https://github.com/netri/Neitri-Unity-Shaders> | Stale (last commit Feb 2023) |
| **Blit-Component** (Merlin) | ~10× cheaper than `Camera`-component for RT-loop rendering. Community proposal, not officially merged. | <https://github.com/MerlinVR/Blit-Component> | Stale (last commit Apr 2021) |
| **GPUParticleVolumes** (REDSIM) | Ultra-fast GPU particles with VRC Light Volumes support. | <https://github.com/REDSIM/GPUParticleVolumes> | Active (v1.1.0, Oct 2026) |
| **Raymarched GPU Particles** (SCRN-VRC) | Raymarched particles using GrabPass for screen-space physics. | <https://github.com/SCRN-VRC/Raymarched-GPU-Particles-with-Screenspace-Physics> | Archived (last commit Jul 2019) |
| **vrc-udon-texture-sampler** (llealloo) | Pre-`VRCAsyncGPUReadback` workaround for reading RenderTexture pixel data back into Udon. The SDK now ships `VRCAsyncGPUReadback` (<https://creators.vrchat.com/worlds/udon/vrc-graphics/asyncgpureadback>). | <https://github.com/llealloo/vrc-udon-texture-sampler> | Stale (v1.0, Mar 2021) — superseded by `VRCAsyncGPUReadback` |
| **VRCPlayersOnlyMirror** (acertainbluecat) | Players-only no-background mirror prefab. | <https://github.com/acertainbluecat/VRCPlayersOnlyMirror> | Stale (v0.1.5, Sep 2022) |
| **VRC Light Volumes** (REDSIM) | Voxel-based + analytic lighting that also lights avatars and dynamic props; Udon scripting API for runtime light/volume control. | <https://github.com/REDSIM/VRCLightVolumes> | Active (v3.0.1, Oct 2026) |

## Custom inspectors / editor tooling

| Project | What it does | Source | Status |
|---|---|---|---|
| **UdonToolkit** (orels1) | Attribute-driven custom inspectors for U#. Property-Drawer/Attribute UI framework. | <https://github.com/orels1/UdonToolkit>, <https://ut.orels.sh/> | Stale (v1.2.1 Nov 2022; v2.0.0-dev.2 pre-release Apr 2024) |
| **CyanTrigger** | In-scene Udon compiler with SDK2-style UI. | <https://github.com/CyanLaser/CyanTrigger> | Active (4.4.8, Oct 2026) |
| **EasyEventEditor** (Merlin) | Better UnityEvent editor (general; helpful in U# worlds). | <https://github.com/MerlinVR/EasyEventEditor> | Stale (v1.0.4, Dec 2020) |
| **VRWorld Toolkit** (oneVR, curated) | 90+ scene validators, Build Report, Mass Texture Importer, `DisableOnBuild` automation. | <https://github.com/oneVR/VRWorldToolkit> | Active (v3.5.1, Oct 2026) |

## Analyzers / linters

| Project | What it does | Source | Status |
|---|---|---|---|
| **UdonRabbit.Analyzer** (esnya) | Roslyn analyzer with 55 rules (URA0001–URA0055) for U# patterns. | <https://github.com/esnya/UdonRabbit.Analyzer> | **Archived** — deprecated, no U# 1.0+ support |
| **udon-analyzer** (mika-f) | Successor-in-spirit. Detects unsupported APIs, perf concerns, `SendCustomEvent` ↔ handler mismatches. | <https://github.com/mika-f/udon-analyzer> | Active (v1.0.0-beta.11 Nov 2025; last commit Feb 2026) |
| **vscode-udonsharp-linter** (refiaa / Project-Sakanah) | VS Code static-analysis linter for U# scripts. | <https://marketplace.visualstudio.com/items?itemName=refiaa.vscode-udonsharp-linter>, <https://github.com/Project-Sakanah/vscode-udonsharp-linter> | Active (last commit Sep 2026) |

## Compiler / optimization

| Project | What it does | Source | Status |
|---|---|---|---|
| **UdonSharpOptimizer** (BlueAmulet) | Post-compile bytecode optimizer: COPY removal, tail-call optimisation, temp-var reuse. | <https://github.com/BlueAmulet/UdonSharpOptimizer> | Active (v1.1.1, Aug 2026) |

## Data containers / utilities

| Project | What it does | Source | Status |
|---|---|---|---|
| **generic-data-container** (koyashiro) | Type-safe `DataList<T>`, `DataDictionary<K,V>` wrappers via VPM. | <https://github.com/koyashiro/generic-data-container> | Stale (v0.2.2, Feb 2025) |
| **udon-list, udon-dictionary** (koyashiro) | Pre-`DataList` standalone implementations. | <https://github.com/koyashiro/udon-list>, <https://github.com/koyashiro/udon-dictionary> | **Archived** — superseded by `DataList` / `DataDictionary` |
| **UArrayCollections** (hoke946) | Treat plain arrays like List/Dictionary. | <https://github.com/hoke946/UArrayCollections> | Stale (0.0.2, Apr 2023) |
| **Kemocade VRC Data Extensions** | Extension methods on `DataList`/`DataDictionary`. | <https://vpm-catalog.vercel.app/packages/com.kemocade.vrc.data.extensions/>, <https://github.com/kemocade/Kemocade.Vrc.Data.Extensions> | Stale (1.0.2, Aug 2023) |

## Parsers / file formats

| Project | What it does | Source | Status |
|---|---|---|---|
| **UdonZip** (Foorack) | ZIP archive parser in pure Udon. Read-only. Useful for accepting compressed save files through player text input. API: `Extract(byte[])`, `GetFileNames(obj)`, `GetFile(obj, string)`, `GetFileData(obj)`. | <https://github.com/Foorack/UdonZip> | Stale (v0.1.0 May 2020; last commit Jan 2022) |
| **UdonXML** (Foorack) | XML parser/writer in pure Udon. API: `LoadXml`, `GetChildNode`, `GetNodeValue`, `SetAttribute`, `CreateChildNode`. Lenient validation. | <https://github.com/Foorack/UdonXML> | Stale (v0.2.0 Apr 2020; last commit Apr 2021) |

Both **[Community]**. Together they cover the practical "parse a structured blob a player paste-inputs" use case before VRChat shipped Persistence — still useful for share-codes, save imports, and external config without going through `VRCStringDownloader`.

## Voice / audio

| Project | What it does | Source | Status |
|---|---|---|---|
| **UdonVoiceUtils** (Guribo) | Per-player voice management: private channels, audio occlusion through walls, reverb zones, voice directionality, push-to-talk / toggle mic activation, automatic voice-range adjustment by avatar height. Targets World SDK 3.10.3+ (via its `tlp.udonutils` ^15.0.3 dependency) / Unity 2022.3. Main classes: `PlayerAudioController`, `PlayerAudioOverride`, `PlayerAudioConfigurationModel`, `VoiceRangeVisualizer`, `MicActivation`. | <https://github.com/Guribo/UdonVoiceUtils> | Active (v7.0.0, Sep 2026) |

**[Community]**. Effectively *the* community library for non-trivial player-audio worlds — privacy zones, immersive bar/restaurant ambiance, role-based voice routing.

## Coroutines / scheduling

Built-in alternatives now cover part of this space: `EventTiming.PostLateUpdate` / `EventTiming.FixedUpdate` for delayed events (SDK 3.10.2) and VRCTween's killable `DelayedCall` (SDK 3.10.4, <https://creators.vrchat.com/worlds/udon/vrctween/>). **[Docs]**

| Project | What it does | Source | Status |
|---|---|---|---|
| **Advanced-local-events** (MyroG) | Adds cancellation, pause, delay-after-queue to `SendCustomEventDelayed*`. ID-based handles. | <https://github.com/MyroG/Advanced-local-events> | Stale (v1.0.0 Oct 2023; last commit Feb 2024) |
| **UdonCoroutines** (Caeden117) | `BaseUdonCoroutine` base class with `WaitForSeconds`, `WaitForSequential`, etc. Early stage. | <https://github.com/Caeden117/UdonCoroutines> | Early stage (tag 0.0.1; last commit Feb 2026) |

## Object pools / spawners

| Project | What it does | Source | Status |
|---|---|---|---|
| **EXUR_UdonObjectPool** (naqtn) | Alternative pool with explicit acquire/release semantics. | <https://github.com/naqtn/EXUR_UdonObjectPool> | Stale (last commit May 2020) |
| **VRC-Object-Spawner** (Pokeyi) | Pre-built network-synced spawner wrapping pool internals. | <https://github.com/Pokeyi/VRC-Object-Spawner> | Stale (v1.0.0 Dec 2021; last commit Jul 2023) |

## String / image loading

| Project | What it does | Source | Status |
|---|---|---|---|
| **VRC-Simple-String-Loader** (DrBlackRat) | Drop-in announcement board with auto-reload + error states. | <https://github.com/DrBlackRat/VRC-Simple-String-Loader> | Stale (v1.0.1 Apr 2024; last commit Feb 2025) |
| **Discord2VRC** (acertainbluecat) | Redirector endpoint to serve Discord-hosted images (handles signed-URL expiry). | <https://github.com/acertainbluecat/Discord2VRC> | Stale (last commit Jul 2022) |
| **PachyPictureFrame** (kdingo) | Slideshow world template. | <https://github.com/kdingo/PachyPictureFrame> | Stale (last commit Nov 2021) |
| **DeacsImageLoader** (DeaconDP) | Dynamic image loader prefab. | <https://github.com/DeaconDP/DeacsImageLoader> | Stale (last commit May 2023) |
| **vrchat-remote-world-list-loader** (vr-voyage) | Downloads a TSV from pastebin, opens portals dynamically. | <https://github.com/vr-voyage/vrchat-remote-world-list-loader> | Stale (v0.20230224, Feb 2023) |
| **UdonSharp-TranslateManager** (Sonic853) | PO-file-based translation system; loads remote PO via `VRCStringDownloader`. | <https://github.com/Sonic853/UdonSharp-TranslateManager> | **Archived** (v0.1.4, May 2022) |
| **examples-image-loading** (official) | Slideshow + image-array reference. | <https://github.com/vrchat-community/examples-image-loading> | Stale (0.2.0, Aug 2023) |

## Debug / logging

| Project | What it does | Source | Status |
|---|---|---|---|
| **VRCNetworkDebugger** (Centauri2442) | Drag-and-drop world-side overlay: top-20 recent transmissions, per-object bandwidth, serialization timing. | <https://github.com/Centauri2442/VRCNetworkDebugger> | Stale (v1.1.1 Feb 2024; last commit Apr 2025). SDK 3.8.1+ exposes network debug info natively — see <https://creators.vrchat.com/worlds/udon/networking/debugging>. |
| **VUdon-Logger** (Varneon) | In-world console window for runtime log capture in VR. | <https://github.com/Varneon/VUdon-Logger> | Active (0.5.0, Oct 2026) |
| **TlpLogger** (Guribo) | Structured logging library (part of UdonUtils). | <https://github.com/Guribo/UdonUtils> | Active (UdonUtils v15.0.3, Sep 2026) |

## Notable author profiles (worth browsing)

| Author | What they ship |
|---|---|
| **CyanLaser** (now VRChat staff) | CyanTrigger (active), CyanEmu and CyanPlayerObjectPool (both archived), WallWalkingExamples. <https://github.com/CyanLaser> |
| **JanSharp** | VRCLockstep, JanSharp Common, grip prefabs (Cannon/Movement/Rotation Grip), Chalkboard, Music Control, Player Zones. VCC listing: <https://jansharp.github.io/vrc/vcclisting.xhtml>. <https://github.com/JanSharp> |
| **MMMaellon** | Smart Object Sync (maintenance), LightSync. <https://github.com/MMMaellon> |
| **MerlinVR** (now VRChat staff) | UdonSharp itself, USharpVideo, EasyEventEditor, VRC-Bakery-Adapter, Blit-Component. <https://github.com/MerlinVR> |
| **Varneon** | VUdon ecosystem (index repo; each component is its own `Varneon/VUdon-*` repo): Playerlist, Noclip, Logger, Music Player, Quick Menu, Array Extensions, Event Dispatcher, Udonity. <https://github.com/Varneon/VUdon> |
| **Guribo** | UdonUtils (TLP networking + scheduling + logging). <https://github.com/Guribo/UdonUtils> |
| **BlueAmulet** | UdonSharpOptimizer. <https://github.com/BlueAmulet/UdonSharpOptimizer> |
| **orels1** | UdonToolkit (stale), ORL shaders (<https://github.com/orels1/orels-Unity-Shaders>, active), Audio-related prefabs. <https://github.com/orels1> |
| **cnlohr** | shadertrixx (the reference notebook for VRChat shader insights). <https://github.com/cnlohr/shadertrixx> |
| **Nestorboy** | NUSaveState. <https://github.com/Nestorboy/NUSaveState> |
| **REDSIM** | VRC Light Volumes, GPUParticleVolumes. <https://github.com/REDSIM> |

## Awesome lists

- **awesome-vrchat** (madjin) — <https://github.com/madjin/awesome-vrchat> — General / Tutorials / Avatars / Worlds / Tools / Udon / OSC / Shaders / Resources. Last commit Dec 2024.
- **UdonVR/Udon** — <https://github.com/UdonVR/Udon> — older compilation of scripts/prefabs (Basics, Games, Toys). Stale (last commit Dec 2022).
- **VPM Catalog** (kurotu) — <https://vpm-catalog.vercel.app/> + <https://github.com/kurotu/vpm-catalog> — browsable index of 500+ community repos. Active (updated daily as of Oct 2026).

## Curated VPM listings

VRChat maintains a curated subset of community VPM packages: <https://vcc.docs.vrchat.com/vpm/curated-community-packages/>. As of Oct 2026 it includes AudioLink, Avatars 3.0 Manager (VRLabs), EasyQuestSwitch, Gesture Manager (BlackStartx), VRWorld Toolkit.

Independent VPM publishers worth knowing:

| Publisher | Listing |
|---|---|
| CyanLaser | <https://cyanlaser.github.io/CyanTrigger/> |
| JanSharp | <https://jansharp.github.io/vrc/vcclisting.xhtml> |
| BlueAmulet | <https://blueamulet.github.io/UdonSharpOptimizer/> |
| DrBlackRat | <https://vpm.drblackrat.xyz/> (old `drblackrat.github.io/vpm-listing/` redirects here) |
| enitimeago | <https://enitimeago.github.io/vpm-repos/> |
| deltaneverused | <https://deltaneverused.github.io/VRChatPackages/> |
| MMMaellon | <https://mmmaellon.github.io/MMMaellonVCCListing/> |

How to publish: fork VRChat's Package Listing Template → edit `source.json` → enable GitHub Pages with "GitHub Actions" → push triggers the "Build Repo Listing" action. Each package needs a full `package.json`. Never delete old versions (breaks consumers' lockfiles). Guide: <https://vcc.docs.vrchat.com/guides/create-listing/>.

## Wikis / long-form documentation

- **VRChat Wiki** — <https://wiki.vrchat.com/> — community-editable. Pages on Udon, AudioLink, VPM, debug menu, trusted URLs, mirrors. (Behind a Cloudflare bot challenge, so it could not be machine-checked on 2026-10-22.)
- **VRCLibrary wiki** — <https://vrclibrary.com/wiki/> — hosts *Varneon's UdonSharp Development Practices*, *Fax's Retrospectives and Prefabs*, *The VRChat Creation Compendium*, Udon benchmarks.
- **VRChat Ask Forum** — <https://ask.vrchat.com/c/creator-hub/udon/66> — official, async, searchable.
- **Phasedragon Input Table** — Google Sheet cited in official U# community-resources page; canonical VR-input binding reference.
- **U# community-resources page** — <https://udonsharp.docs.vrchat.com/community-resources/> — curated tutorial / tool / reference index.
- **Hai's "Nochat"** — <https://docs.hai-vr.dev/docs/research/nochat> — runs U# as plain C# in Editor VR for prefab dev.

## Discord / forums

- **Official VRChat Discord** — `discord.gg/vrchat`. Creator-relevant channels: `#world-development`, `#world-optimization`, `#world-lighting`, `#world-ui-design`, `#world-quest`, `#udon-general`, `#udon-info`, `#udon-networking`, `#udon-persistence`, `#udon-showoff`.
- **VRCreators** — <https://vrcreators.net/discord> — practical community Discord for worlds / avatars / Udon / Blender.
- **VRChat Ask Forum** (above).

(There is no longer a separate "Merlin's Discord" — Merlin moved to VRChat staff and discussion consolidated into the official server's Udon channels.)

## Editor / simulation

- **ClientSim** — official VRChat editor simulator (evolved from CyanEmu). Bundled inside the World SDK since 3.4.0 (`Packages/com.vrchat.worlds/Integrations/ClientSim/`) rather than installed separately; the standalone repo's last tag is 1.2.7 (Aug 2023) but it still receives commits (Jul 2026). <https://github.com/vrchat-community/ClientSim>, docs: <https://creators.vrchat.com/worlds/clientsim/>. Known to lack remote-player full simulation; multi-player networking is not simulated. Tracked issues: <https://github.com/vrchat-community/ClientSim/issues>.
- **CyanEmu** — predecessor, archived (last commit Mar 2022). <https://github.com/CyanLaser/CyanEmu>.

## AI / agent tooling

| Project | What it does | Source | Status |
|---|---|---|---|
| **agent-skills-vrc-udon** (niaka3dayo) | Knowledge base for AI coding agents (Claude / Gemini / Codex) targeting UdonSharp: compile-constraint rules, validation hooks, project templates. Useful both as an input to AI tools *and* as a cross-reference for any human writing U# who wants to read what the AI is being told. | <https://github.com/niaka3dayo/agent-skills-vrc-udon> | Active (v4.1.1, Sep 2026) |

**[Community]**. The author of that project flags that AI-assisted output can contain subtle errors and recommends verification against official VRChat documentation — the same caveat applies to this reference (see the disclaimer at the bottom of the main README).

## Test infrastructure

**No first-class U# unit-test runner exists.** Unity Test Runner can't execute Udon-compiled assemblies, and ClientSim doesn't expose an automated test harness (issue #133 still open as of Oct 2026). Practical patterns:

- UdonUtils ships test controller templates and gizmos — closest to a community testing convention. <https://github.com/Guribo/UdonUtils>
- ClientSim playmode + scripted player input for integration smoke tests.
- GameCI's `unity-test-runner` for the wrapping C# / Unity layer (not U# itself).
- Manual multi-client testing remains the de-facto QA approach.

This is a recognised ecosystem gap.

## See also

- [Source Map](source-map.md) — pointers into the UdonSharp source.
- [Official Docs](official-docs.md) — first-party VRChat pages cited throughout this reference.
