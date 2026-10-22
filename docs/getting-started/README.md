# Getting Started

This section is for readers who have not written an UdonSharp world before. If you've shipped an UdonBehaviour you can skip ahead to [UdonSharpBehaviour](../udonsharp-behaviour/README.md).

## Pages

| Page | What's in it |
|---|---|
| [Overview](overview.md) | What Udon, U#, and the VM actually are. |
| [Installation](installation.md) | Project setup via the VRChat Creator Companion (or ALCOM), Unity version, where UdonSharp lives inside the Worlds SDK. |
| [First Script](first-script.md) | A minimal, working U# script with explanations. |

## TL;DR for impatient readers

1. Install Unity **2022.3.22f1** — the exact version VRChat uses as of SDK 3.10.5. Other Unity versions (including newer 2022.3 patches that Unity Hub may recommend) are not supported. **[Docs]** <https://creators.vrchat.com/sdk/upgrade/current-unity-version/>
2. Install the **VRChat Creator Companion** (VCC) from <https://vrchat.com/home/download>. It installs the correct Unity version for you. (Windows only; the community-made [ALCOM](installation.md#alternative-alcom-community) also runs on macOS/Linux.)
3. Create a new **World** project. UdonSharp has been **bundled inside the Worlds SDK** (`com.vrchat.worlds`) since SDK 3.4.0, so every World project already has it — there is no separate UdonSharp package to add. **[Docs]** <https://creators.vrchat.com/releases/release-3-4-0/>
4. In the Project window, right-click → `Create` → `U# Script`, then pick a file name in the save dialog. This creates both a `.cs` file and a paired Udon Program Asset (`.asset`). **[Source]**
5. Drag the `.cs` onto a GameObject (or use **Add Component**). UdonSharp adds your class as a component and creates the hidden backing `UdonBehaviour` for you — see [Proxy ↔ Backing](../editor-api/proxy-backing.md).
6. Write your code by overriding methods on `UdonSharpBehaviour` — see [First Script](first-script.md).

## What this section will *not* teach

- Unity fundamentals (GameObjects, components, prefabs, the Editor). Use Unity's official Learn portal.
- C# fundamentals. The Microsoft C# guide is the canonical reference.
- World design, optimisation, or publishing flow. See VRChat's creator docs at <https://creators.vrchat.com/worlds/>.

This reference assumes you know "what a `Vector3` is" and want to look up "what `VRCObjectPool.TryToSpawn` returns."

## Where the community lives

- **Official VRChat Discord** — `discord.gg/vrchat`. Channels: `#world-development`, `#udon-general`, `#udon-networking`, `#udon-persistence`. Fastest answer surface for U# questions. **[Community]**
- **VRChat Ask Forum** — <https://ask.vrchat.com/c/creator-hub/udon/66>. Long-form, searchable. **[Community]**
- **VRChat Wiki** — <https://wiki.vrchat.com/>. Community-editable. **[Community]**
- **VRCLibrary** — <https://vrclibrary.com/wiki/>. Hosts *Varneon's UdonSharp Development Practices* and other long-form guides. **[Community]**
- **`awesome-vrchat`** — <https://github.com/madjin/awesome-vrchat>. Curated index of tools, tutorials, shaders. **[Community]**
- See [References → Community Resources](../../docs/references/community-resources.md) for the full list.
