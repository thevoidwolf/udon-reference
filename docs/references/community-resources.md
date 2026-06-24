# Community Resources

A curated index of community-built libraries, tools, simulators, and documentation that supplement the official VRChat / UdonSharp ecosystem. Each entry has a source URL. **Status flags** mark active vs. deprecated projects as of the research date.

> All entries on this page are **[Community]** unless otherwise noted. Verify currency before adopting — the ecosystem moves quickly and some flagship projects are archived after VRChat ships a built-in equivalent.

---

## Networking libraries

| Project | What it does | Source | Status |
|---|---|---|---|
| **VRCLockstep** (JanSharp) | Deterministic lockstep networking: broadcasts ordered input actions; every client runs the same ticks in the same order. Eliminates ownership race conditions; supports large mutating state via byte-array snapshots for late joiners. | <https://github.com/JanSharp/VRCLockstep> | Active (v1.1.1) |
| **UNet** (Xytabich) | Reliable binary RPC layer over Udon. `ByteBufferWriter`/`Reader`, sequenced or unordered modes, per-player targeting, message-ID acks. Predates SDK 3.8.1's `[NetworkCallable]`. | <https://github.com/Xytabich/UNet> | Archived (Apr 2026) |
| **UdonNet** (poi-vrc) | WIP P2P Udon networking. Less mature than UNet. | <https://github.com/poi-vrc/UdonNet> | Experimental |
| **SimpleNetwork** (tutinoco) | Wrapper simplifying typed networking. | <https://github.com/tutinoco/SimpleNetwork> | — |
| **Smart Object Sync** (JanSharp) | Underlying sync layer used by JanSharp's grip prefabs (Cannon/Movement/Rotation Grip). | <https://jansharp.github.io/> | Active |
| **UdonUtils** (Guribo) | `TlpNetworkTime`, `TlpAccurateSyncBehaviour`, state machines, rigidbody recording/prediction, `TlpLogger`. Requires World SDK 3.10.3+. | <https://github.com/Guribo/UdonUtils> | Active (v14.0.4) |

## Per-player state

| Project | What it does | Source | Status |
|---|---|---|---|
| **CyanPlayerObjectPool** | Master-authoritative per-player GameObject pool. Was *the* canonical pattern for per-player state pre-PlayerObject. Master-migration safe. | <https://github.com/CyanLaser/CyanPlayerObjectPool> | **Archived Oct 2024** — superseded by stock `VRCPlayerObject`. Still in many older worlds. |
| **LSNPlayerObjectPool** | Fork of CyanPlayerObjectPool. | <https://github.com/LastationVRChat/LSNPlayerObjectPool> | — |
| **NUSaveState** (Nestorboy) | Pre-PlayerData persistence hack using avatar parameters as a storage substrate (read back via bone rotations). The *only* practical cross-world persistence solution. | <https://github.com/Nestorboy/NUSaveState> | Active (v1.6.0, Dec 2024) |

## Video players

VRChat officially endorses USharpVideo, VideoTXL, and ProTV in the same paragraph: <https://creators.vrchat.com/worlds/udon/video-players/>.

| Project | What's distinctive | Source | Status |
|---|---|---|---|
| **USharpVideo** (Merlin) | Network-time-anchored sync model. Manual sync for late joiners. AVPro + Unity backends. | <https://github.com/MerlinVR/USharpVideo> | Maintenance mode (last release Oct 2023) |
| **USharpVideoQueue** (sam-ln) | Synced playlist queue add-on for USharpVideo. | <https://github.com/sam-ln/USharpVideoQueue> | — |
| **VideoTXL** | Three-manager architecture (Video / Audio / Screen). Ships Sync **and** Local prefabs. AudioLink + VRSL integration. | <https://github.com/vrctxl/VideoTXL>, <https://vrctxl.github.io/Docs/> | Active |
| **ProTV** (ArchiTech) | Event-driven plugin architecture. Multi-config swap. LTCGI adapters. | <https://gitlab.com/techanon/protv>, <https://protv.dev/faq> | Active |
| **SynMediaPlayer** | Gapless playback, automatic retry on failure, low-latency streams. | <https://github.com/synergiance/SynMediaPlayer> | Active (Mar 2024) |
| **VVMW / VizVid** (JLChnToZ) | Per-platform URL pairs (PC vs Quest), RTSP/RTMP, request debouncing. | <https://github.com/JLChnToZ/VVMW> | Active |
| **RiskiPlayer** | Modified ProTV with on-screen UI. | <https://github.com/RiskiVR/RiskiPlayer> | — |
| **USharpVideoModernUI** (DrBlackRat) | UI replacement for USharpVideo. | <https://drblackrat.github.io/vpm-listing/> | Active |

## Graphics / shaders

| Project | What it does | Source |
|---|---|---|
| **AudioLink** (llealloo + collaborators) | World-audio-to-shader bridge. Broadcasts `_AudioTexture` (the *only* non-`_Udon` prefix VRChat allowlists for `VRCShader.SetGlobal*`). Cross-platform (VRC/CVR/Poligon). | <https://github.com/llealloo/audiolink> |
| **LTCGI** (PiMaker) | Real-time area lighting via Linearly Transformed Cosines. UdonSharp integration; ProTV adapter. Quest caps at 2 screens + FAST_SAMPLING. | <https://github.com/PiMaker/ltcgi>, <https://ltcgi.dev/> |
| **shadertrixx** (cnlohr) | Reference notes on VRChat shader tricks: mirror depth, GPU particles, RT loop techniques, mesh-bounds gotchas. | <https://github.com/cnlohr/shadertrixx> |
| **Poiyomi Toon** | Most popular feature-rich toon shader. | <https://github.com/poiyomi/PoiyomiToonShader> |
| **Mochie's Unity Shaders** | Standard, water, post-processing variants. | <https://github.com/MochiesCode/Mochies-Unity-Shaders> |
| **z3y / Okabintaro shaders** | Standard/Surface replacement; first to ship Quest box-projected AreaLit + LTCGI. | <https://github.com/z3y/shaders>, <https://github.com/Okabintaro/z3y_shaders> |
| **Silent's Cel Shading** | Lightweight cel shader respecting Unity lighting. | <https://ask.vrchat.com/t/silents-cel-shading-shader/3234> |
| **Neitri-Unity-Shaders** | Wireframe overlays and utilities. | <https://github.com/netri/Neitri-Unity-Shaders> |
| **Blit-Component** (Merlin) | ~10× cheaper than `Camera`-component for RT-loop rendering. Community proposal, not officially merged. | <https://github.com/MerlinVR/Blit-Component> |
| **GPUParticleVolumes** (REDSIM) | Ultra-fast GPU particles with VRC Light Volumes support. | <https://github.com/REDSIM/GPUParticleVolumes> |
| **Raymarched GPU Particles** (SCRN-VRC) | Raymarched particles using GrabPass for screen-space physics. | <https://github.com/SCRN-VRC/Raymarched-GPU-Particles-with-Screenspace-Physics> |
| **vrc-udon-texture-sampler** (llealloo) | Workaround for reading RenderTexture pixel data back into Udon (no native RT-read API). | <https://github.com/llealloo/vrc-udon-texture-sampler> |
| **VRCPlayersOnlyMirror** (acertainbluecat) | Players-only no-background mirror prefab. | <https://github.com/acertainbluecat/VRCPlayersOnlyMirror> |

## Custom inspectors / editor tooling

| Project | What it does | Source | Status |
|---|---|---|---|
| **UdonToolkit** (orels1) | Attribute-driven custom inspectors for U#. Property-Drawer/Attribute UI framework. | <https://github.com/orels1/UdonToolkit>, <https://ut.orels.sh/> | Stale (v1.2.1 Nov 2022) — predates U# 1.0 |
| **CyanTrigger** | In-scene Udon compiler with SDK2-style UI. | <https://github.com/CyanLaser/CyanTrigger> | — |
| **EasyEventEditor** (Merlin) | Better UnityEvent editor (general; helpful in U# worlds). | <https://github.com/MerlinVR> | — |
| **VRWorld Toolkit** (oneVR, curated) | 90+ scene validators, Build Report, Mass Texture Importer, `DisableOnBuild` automation. | <https://github.com/oneVR/VRWorldToolkit> | Active (v3.4.1 Mar 2026) |

## Analyzers / linters

| Project | What it does | Source | Status |
|---|---|---|---|
| **UdonRabbit.Analyzer** (esnya) | Roslyn analyzer with 55 rules (URA0001–URA0055) for U# patterns. | <https://github.com/esnya/UdonRabbit.Analyzer> | **Deprecated** — no U# 1.0+ support |
| **udon-analyzer** (mika-f) | Successor-in-spirit. Detects unsupported APIs, perf concerns, `SendCustomEvent` ↔ handler mismatches. | <https://github.com/mika-f/udon-analyzer> | Active |
| **vscode-udonsharp-linter** (refiaa) | VS Code quick linter. | <https://marketplace.visualstudio.com/items?itemName=refiaa.vscode-udonsharp-linter> | — |

## Compiler / optimization

| Project | What it does | Source | Status |
|---|---|---|---|
| **UdonSharpOptimizer** (BlueAmulet) | Post-compile bytecode optimizer: COPY removal, tail-call optimisation, temp-var reuse. | <https://github.com/BlueAmulet/UdonSharpOptimizer> | Active (v1.0.12, Mar 2025) |

## Data containers / utilities

| Project | What it does | Source | Status |
|---|---|---|---|
| **generic-data-container** (koyashiro) | Type-safe `DataList<T>`, `DataDictionary<K,V>` wrappers via VPM. | <https://github.com/koyashiro/generic-data-container> | Active |
| **udon-list, udon-dictionary** (koyashiro) | Pre-`DataList` standalone implementations. | <https://github.com/koyashiro/udon-list>, <https://github.com/koyashiro/udon-dictionary> | — |
| **UArrayCollections** (hoke946) | Treat plain arrays like List/Dictionary. | <https://github.com/hoke946/UArrayCollections> | — |
| **Kemocade VRC Data Extensions** | Extension methods on `DataList`/`DataDictionary`. | <https://vpm-catalog.vercel.app/packages/com.kemocade.vrc.data.extensions/> | — |

## Parsers / file formats

| Project | What it does | Source | Status |
|---|---|---|---|
| **UdonZip** (Foorack) | ZIP archive parser in pure Udon. Read-only. Useful for accepting compressed save files through player text input. API: `Extract(byte[])`, `GetFileNames(obj)`, `GetFile(obj, string)`, `GetFileData(obj)`. | <https://github.com/Foorack/UdonZip> | Minimally maintained (5 commits) |
| **UdonXML** (Foorack) | XML parser/writer in pure Udon. API: `LoadXml`, `GetChildNode`, `GetNodeValue`, `SetAttribute`, `CreateChildNode`. Lenient validation. | <https://github.com/Foorack/UdonXML> | Minimally maintained (last release Apr 2020) |

Both **[Community]**. Together they cover the practical "parse a structured blob a player paste-inputs" use case before VRChat shipped Persistence — still useful for share-codes, save imports, and external config without going through `VRCStringDownloader`.

## Voice / audio

| Project | What it does | Source | Status |
|---|---|---|---|
| **UdonVoiceUtils** (Guribo) | Per-player voice management: private channels, audio occlusion through walls, reverb zones, voice directionality, push-to-talk / toggle mic activation, automatic voice-range adjustment by avatar height. Targets World SDK 3.10.3+ / Unity 2022.3. Main classes: `PlayerAudioController`, `PlayerAudioOverride`, `PlayerAudioConfigurationModel`, `VoiceRangeVisualizer`, `MicActivation`. | <https://github.com/Guribo/UdonVoiceUtils> | Active (v5.1.4 May 2026) |

**[Community]**. Effectively *the* community library for non-trivial player-audio worlds — privacy zones, immersive bar/restaurant ambiance, role-based voice routing.

## Coroutines / scheduling

| Project | What it does | Source |
|---|---|---|
| **Advanced-local-events** (MyroG) | Adds cancellation, pause, delay-after-queue to `SendCustomEventDelayed*`. ID-based handles. | <https://github.com/MyroG/Advanced-local-events> |
| **UdonCoroutines** (Caeden117) | `BaseUdonCoroutine` base class with `WaitForSeconds`, `WaitForSequential`, etc. Early stage. | <https://github.com/Caeden117/UdonCoroutines> |

## Object pools / spawners

| Project | What it does | Source |
|---|---|---|
| **EXUR_UdonObjectPool** (naqtn) | Alternative pool with explicit acquire/release semantics. | <https://github.com/naqtn/EXUR_UdonObjectPool> |
| **VRC-Object-Spawner** (Pokeyi) | Pre-built network-synced spawner wrapping pool internals. | <https://github.com/Pokeyi/VRC-Object-Spawner> |

## String / image loading

| Project | What it does | Source |
|---|---|---|
| **VRC-Simple-String-Loader** (DrBlackRat) | Drop-in announcement board with auto-reload + error states. | <https://github.com/DrBlackRat/VRC-Simple-String-Loader> |
| **Discord2VRC** (acertainbluecat) | Redirector endpoint to serve Discord-hosted images (handles signed-URL expiry). | <https://github.com/acertainbluecat/Discord2VRC> |
| **PachyPictureFrame** (kdingo) | Slideshow world template. | <https://github.com/kdingo/PachyPictureFrame> |
| **DeacsImageLoader** (DeaconDP) | Dynamic image loader prefab. | <https://github.com/DeaconDP/DeacsImageLoader> |
| **vrchat-remote-world-list-loader** (vr-voyage) | Downloads a TSV from pastebin, opens portals dynamically. | <https://github.com/vr-voyage/vrchat-remote-world-list-loader> |
| **UdonSharp-TranslateManager** (Sonic853) | PO-file-based translation system; loads remote PO via `VRCStringDownloader`. | <https://github.com/Sonic853/UdonSharp-TranslateManager> |
| **examples-image-loading** (official) | Slideshow + image-array reference. | <https://github.com/vrchat-community/examples-image-loading> |

## Debug / logging

| Project | What it does | Source | Status |
|---|---|---|---|
| **VRCNetworkDebugger** (Centauri2442) | Drag-and-drop world-side overlay: top-20 recent transmissions, per-object bandwidth, serialization timing. | <https://github.com/Centauri2442/VRCNetworkDebugger> | Stale (Feb 2024) |
| **VUdon-Logger** (Varneon) | In-world console window for runtime log capture in VR. | <https://github.com/Varneon/VUdon> | Active |
| **TlpLogger** (Guribo) | Structured logging library. | <https://github.com/Guribo/UdonUtils> | Active |

## Notable author profiles (worth browsing)

| Author | What they ship |
|---|---|
| **CyanLaser** (now VRChat staff) | CyanTrigger, CyanEmu, CyanPlayerObjectPool, WallWalkingExamples. <https://github.com/CyanLaser> |
| **JanSharp** | VRCLockstep, Smart Object Sync, grip prefabs, Chalkboard. <https://jansharp.github.io/> |
| **MerlinVR** (now VRChat staff) | UdonSharp itself, USharpVideo, EasyEventEditor, VRC-Bakery-Adapter, Blit-Component. <https://github.com/MerlinVR> |
| **Varneon** | VUdon ecosystem: Playerlist, Noclip, Logger, Music Player, Quick Menu, Array Extensions, Event Dispatcher, Udonity. <https://github.com/Varneon/VUdon> |
| **Guribo** | UdonUtils (TLP networking + scheduling + logging). <https://github.com/Guribo/UdonUtils> |
| **BlueAmulet** | UdonSharpOptimizer. <https://github.com/BlueAmulet/UdonSharpOptimizer> |
| **orels1** | UdonToolkit, ORL shaders, Audio-related prefabs. <https://github.com/orels1> |
| **cnlohr** | shadertrixx (the reference notebook for VRChat shader insights). <https://github.com/cnlohr/shadertrixx> |
| **Nestorboy** | NUSaveState. <https://github.com/Nestorboy/NUSaveState> |

## Awesome lists

- **awesome-vrchat** (madjin) — <https://github.com/madjin/awesome-vrchat> — General / Tutorials / Avatars / Worlds / Tools / Udon / OSC / Shaders / Resources.
- **UdonVR/Udon** — <https://github.com/UdonVR/Udon> — older but stable compilation of scripts/prefabs (Basics, Games, Toys).
- **VPM Catalog** (kurotu) — <https://vpm-catalog.vercel.app/> + <https://github.com/kurotu/vpm-catalog> — browsable index of 500+ community repos.

## Curated VPM listings

VRChat maintains a curated subset of community VPM packages: <https://vcc.docs.vrchat.com/vpm/curated-community-packages/>. Currently includes AudioLink, Avatars 3.0 Manager (VRLabs), EasyQuestSwitch, Gesture Manager (BlackStartx), VRWorld Toolkit.

Independent VPM publishers worth knowing:

| Publisher | Listing |
|---|---|
| CyanLaser | <https://cyanlaser.github.io/CyanTrigger/> |
| JanSharp | <https://jansharp.github.io/> |
| BlueAmulet | <https://blueamulet.github.io/UdonSharpOptimizer/> |
| DrBlackRat | <https://drblackrat.github.io/vpm-listing/> |
| enitimeago | <https://enitimeago.github.io/vpm-repos/> |
| deltaneverused | <https://deltaneverused.github.io/VRChatPackages/> |

How to publish: fork VRChat's Package Listing Template → edit `source.json` → enable GitHub Pages with "GitHub Actions" → push triggers the "Build Repo Listing" action. Each package needs a full `package.json`. Never delete old versions (breaks consumers' lockfiles). Guide: <https://vcc.docs.vrchat.com/guides/create-listing/>.

## Wikis / long-form documentation

- **VRChat Wiki** — <https://wiki.vrchat.com/> — community-editable. Pages on Udon, AudioLink, VPM, debug menu, trusted URLs, mirrors.
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

- **ClientSim** — official VRChat editor simulator (evolved from CyanEmu). Now bundled inside the World SDK rather than installed separately. <https://github.com/vrchat-community/ClientSim>. Known to lack remote-player full simulation; multi-player networking is not simulated. Tracked issues: <https://github.com/vrchat-community/ClientSim/issues>.
- **CyanEmu** — predecessor, archived. <https://github.com/CyanLaser/CyanEmu>.

## AI / agent tooling

| Project | What it does | Source | Status |
|---|---|---|---|
| **agent-skills-vrc-udon** (niaka3dayo) | Knowledge base for AI coding agents (Claude / Gemini / Codex) targeting UdonSharp: compile-constraint rules, validation hooks, project templates. Useful both as an input to AI tools *and* as a cross-reference for any human writing U# who wants to read what the AI is being told. | <https://github.com/niaka3dayo/agent-skills-vrc-udon> | Active |

**[Community]**. The author of that project flags that AI-assisted output can contain subtle errors and recommends verification against official VRChat documentation — the same caveat applies to this reference (see the disclaimer at the bottom of the main README).

## Test infrastructure

**No first-class U# unit-test runner exists.** Unity Test Runner can't execute Udon-compiled assemblies, and ClientSim doesn't expose an automated test harness (issue #133 still open). Practical patterns:

- UdonUtils ships test controller templates and gizmos — closest to a community testing convention. <https://github.com/Guribo/UdonUtils>
- ClientSim playmode + scripted player input for integration smoke tests.
- GameCI's `unity-test-runner` for the wrapping C# / Unity layer (not U# itself).
- Manual multi-client testing remains the de-facto QA approach.

This is a recognised ecosystem gap.

## See also

- [Source Map](source-map.md) — pointers into the UdonSharp source.
- [Official Docs](official-docs.md) — first-party VRChat pages cited throughout this reference.
