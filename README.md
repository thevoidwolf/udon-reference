# Udon Reference

A community-maintained API reference for **Udon** and **UdonSharp (U#)** — the scripting layer used to build interactive VRChat worlds.

This reference exists because the official VRChat documentation, while improving, is not exhaustive: many runtime APIs are undocumented, several method signatures are scattered across multiple pages, and a large portion of practical knowledge lives in community forums, Discord channels, and decompiled sources. This repo consolidates that information into one navigable specification.

Each section pulls from three sources:
1. The open-source UdonSharp runtime and editor packages.
2. The official VRChat creator documentation.
3. **Community projects** — open-source libraries, prefabs, analyzers, and shared knowledge from GitHub, the VRChat Wiki, VRCLibrary, the Ask forum, and Discord. See [Community Resources](docs/references/community-resources.md) for the central index.

> **Status:** Work in progress. Every claim is tagged for provenance — see [Provenance tags](#provenance-tags) below.

---

## Quick navigation

| Section | What's in it |
|---|---|
| [Getting Started](docs/getting-started/README.md) | Concepts, installation, first U# script. |
| [UdonSharpBehaviour](docs/udonsharp-behaviour/README.md) | Base class, lifecycle events, runtime methods, type identity. |
| [Attributes](docs/attributes/README.md) | `[UdonSynced]`, `[UdonBehaviourSyncMode]`, `[RecursiveMethod]`, `[FieldChangeCallback]`. |
| [Networking](docs/networking/README.md) | Sync modes, ownership, custom network events, `VRCObjectSync`, `VRCObjectPool`. |
| [Players](docs/players/README.md) | `VRCPlayerApi`, tracking data, voice/audio, pickups, player tags, `Networking` static class. |
| [GameObject Lifecycle](docs/gameobject-lifecycle/README.md) | `Instantiate`, `Destroy`, parenting, runtime constraints. |
| [Graphics](docs/graphics/README.md) | `VRCGraphics`, `VRCShader`, materials, textures, meshes, render textures, `VRCImageDownloader`. |
| [Data Containers](docs/data-containers/README.md) | `DataToken`, `DataList`, `DataDictionary`, `VRCJson`. |
| [Persistence](docs/persistence/README.md) | `PlayerData`, Player Objects, persistence events. |
| [Loading](docs/loading/README.md) | `VRCStringDownloader`, image loading, trusted URLs. |
| [Video](docs/video/README.md) | `VRCAVProVideoPlayer`, `VRCUnityVideoPlayer`, video events. |
| [Input & UI](docs/input-and-ui/README.md) | Input events, animation events, UI events, whitelists. |
| [MIDI](docs/midi/README.md) | MIDI input events, channel/note ranges. |
| [AI Navigation](docs/ai-navigation/README.md) | NavMesh integration and constraints. |
| [Editor API](docs/editor-api/README.md) | `UdonSharpEditorUtility`, `UdonSharpProgramAsset`, `UdonSharpGUI`, build hooks. |
| [Compiler](docs/compiler/README.md) | Supported and unsupported C# features in U#, workarounds. |
| [References](docs/references/README.md) | Source map, official docs, community resources. |

---

## What is Udon?

**Udon** is VRChat's sandboxed scripting runtime for worlds. It is a register-based virtual machine that executes a restricted bytecode (called **Udon Assembly**) at runtime inside the VRChat client. Udon programs are attached to GameObjects via the `UdonBehaviour` component.

You don't normally write Udon Assembly by hand. Instead you author programs in one of two front-ends:

- **Udon Graph** — visual, node-based; ships with the SDK.
- **UdonSharp (U#)** — a C# subset compiled to Udon Assembly. This reference focuses on U#.

The Udon VM enforces an **allowlist** of permitted .NET / Unity APIs. Anything outside the allowlist is rejected at compile time (UdonSharp) or refused by the runtime. This is what makes the security model work: VRChat can host arbitrary user code without letting that code read files, open sockets, or call into native plug-ins.

## What is UdonSharp?

UdonSharp is a Roslyn-based compiler that translates a strict subset of C# into Udon Assembly. From the developer's perspective:

- You write classes deriving from `UdonSharpBehaviour` (which itself derives from `MonoBehaviour`).
- The compiler emits a serialized **Udon Program Asset** alongside your `.cs` file.
- At runtime, a hidden **backing `UdonBehaviour`** on the same GameObject executes the compiled program.
- The C# class itself acts as an **editor-side proxy** so Unity's inspector, prefab system, and serialization work as expected.

The proxy ↔ backing duality is the source of most of the editor-tooling API (see [Editor API](docs/editor-api/README.md)). At runtime, only the backing exists — the proxy is purely an editor artifact.

---

## How to use this reference

- **Looking up a specific API?** Use the [Quick navigation](#quick-navigation) table or search the repo on GitHub (press `/`).
- **New to Udon?** Start with [Getting Started](docs/getting-started/README.md) then [UdonSharpBehaviour](docs/udonsharp-behaviour/README.md).
- **Building a feature?** Find the closest topic page and cross-link from there. Most pages end with a "See also" section.
- **Need to know if something is officially supported?** Every claim is tagged — see below.

## Provenance tags

Because Udon has no published extern allowlist, this reference distinguishes the *strength of evidence* for every non-obvious claim:

- **[Docs]** — Officially documented at <https://creators.vrchat.com/worlds/udon/> or <https://udonsharp.docs.vrchat.com/>.
- **[Source]** — Confirmed by reading the open-source UdonSharp runtime / editor under `Packages/com.vrchat.UdonSharp/`.
- **[Code-confirmed]** — Confirmed by sample or test code shipped in the UdonSharp package.
- **[Community]** — Sourced from a named community project (GitHub repo, blog post, wiki page) with an explicit URL citation. Higher trust than `[Community-known]`.
- **[Community-known]** — Widely used in production Udon worlds but not formally documented; not tied to a single citable source.
- **[Unverified]** — Mentioned in community sources but I have not personally validated. Trust with care.

If a section makes a claim without a tag, it's a standard C#/Unity fact that doesn't require Udon-specific verification (e.g. "`int` is a 32-bit signed integer").

---

## Contributing

This reference is sourced from:

1. The open-source UdonSharp package at <https://github.com/vrchat-community/UdonSharp>.
2. Official VRChat creator documentation at <https://creators.vrchat.com/worlds/udon/>.
3. UdonSharp dedicated documentation at <https://udonsharp.docs.vrchat.com/>.
4. Community resources catalogued in [docs/references/community-resources.md](docs/references/community-resources.md).

When contributing:
- Cite your source with a provenance tag.
- Quote method signatures verbatim when sourcing from official docs.
- Prefer adding to existing pages over creating new ones.
- Note explicitly when something is *not* documented — gaps in coverage are valuable signal.

## License

This reference is independent of VRChat Inc. and the UdonSharp project. Documentation is offered as-is for educational and developer-reference purposes.

---

## Accuracy & AI-Assisted Creation

Content is provided **"AS IS" without warranty.** Errors, outdated information, or incomplete content may exist. Always verify against official VRChat documentation at <https://creators.vrchat.com/worlds/udon/> and <https://udonsharp.docs.vrchat.com/> before relying on anything here for production work.

This reference was created and maintained with AI tool assistance. All content has been reviewed and every non-obvious claim is tagged for provenance (see [Provenance tags](#provenance-tags)), but AI-generated portions may contain subtle errors — misremembered method signatures, inverted defaults, or stale claims that were accurate at training time but no longer reflect the current SDK. Use at your own risk.

The author and contributors assume no liability for issues caused by use of this repository, including but not limited to: build errors, upload rejections, unexpected world behavior at runtime, sync desynchronization, or persistent-data corruption.

The VRChat development community has a range of views on AI-assisted contributions. If you find any inaccuracy, please open an issue or submit a correction — accuracy is a community effort, and the goal of this reference is to be more useful than the sum of its parts, not to replace the human knowledge in VRChat Discord servers, the Ask forum, or the source code itself.

Creativity is and always shall be a human endeavor. Use this reference as a starting point, not a final authority.