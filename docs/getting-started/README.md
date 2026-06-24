# Getting Started

This section is for readers who have not written an UdonSharp world before. If you've shipped an UdonBehaviour you can skip ahead to [UdonSharpBehaviour](../udonsharp-behaviour/README.md).

## Pages

| Page | What's in it |
|---|---|
| [Overview](overview.md) | What Udon, U#, and the VM actually are. |
| [Installation](installation.md) | Project setup via the VRChat Creator Companion, Unity version, package layout. |
| [First Script](first-script.md) | A minimal, working U# script with explanations. |

## TL;DR for impatient readers

1. Install Unity **2019.4.31f1** (exact version — newer Unity is not supported by the VRChat World SDK as of writing).
2. Install the **VRChat Creator Companion** from <https://vrchat.com/home/download>.
3. Create a new **World** project with the UdonSharp package included.
4. In the project, right-click in the Project window → `Create` → `U# script`. This creates both a `.cs` file and a paired Udon Program Asset.
5. Add an `UdonBehaviour` component to a GameObject in your scene and assign the Program Asset (or drag the `.cs` directly).
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
