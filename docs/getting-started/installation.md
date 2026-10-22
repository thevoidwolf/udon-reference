# Installation

## Requirements

| Requirement | Version |
|---|---|
| Unity | **2022.3.22f1** (exact). This is the version VRChat uses as of SDK 3.10.5; upgrading to a different Unity version may stop uploaded content from loading. **[Docs]** <https://creators.vrchat.com/sdk/upgrade/current-unity-version/> |
| Unity modules | *Android Build Support* and/or *iOS Build Support* if you target Quest/Android or iOS. **[Docs]** |
| VRChat Creator Companion (VCC) | latest (2.4.5 stable, Dec 2025; 2.5.0-beta.2, Jun 2026). Windows 10/11 64-bit. **[Docs]** <https://vcc.docs.vrchat.com/news> |
| VRChat Worlds SDK (`com.vrchat.worlds`) | latest (3.10.5 as of Sep 2026) |
| UdonSharp | **Bundled with the Worlds SDK** since 3.4.0 — nothing separate to install. **[Docs]** <https://creators.vrchat.com/releases/release-3-4-0/> |

> **Legacy note.** Before SDK 3.4.0 (Oct 2023), UdonSharp and ClientSim were separate VCC packages (`com.vrchat.udonsharp`, `com.vrchat.clientsim`). SDK 3.4.0 merged both into the Worlds package's `Integrations` folder and lists the old IDs as `legacyPackages`, so VCC / the Resolver remove the stand-alone copies when you upgrade. Update to VCC 2.1.6+ before migrating such a project. **[Docs]** <https://creators.vrchat.com/releases/release-3-4-0/>
>
> The UdonSharp docs site's [Setup](https://udonsharp.docs.vrchat.com/setup) page still describes the pre-3.4.0 flow (Unity 2019.4.31f1, adding UdonSharp from the Curated repo). Treat it as historical.

## Recommended path: VRChat Creator Companion

1. Download VCC from <https://vrchat.com/home/download>.
2. Install it and complete the initial setup. It will install Unity Hub and the required Unity version if missing. **[Docs]** <https://creators.vrchat.com/sdk/>
3. Click **Create New Project** and choose the **World** template. VCC's template list is *Avatar*, *World* and *UdonSharp*; the UdonSharp template "includes everything in the World Template", and since SDK 3.4.0 the World template already contains UdonSharp, so either works. **[Docs]** <https://vcc.docs.vrchat.com/vpm/templates>
4. Open the project from VCC. Unity will resolve packages on first launch.
5. Check that the Unity title bar ends with `PC, Mac & Linux Standalone <DX11>` (switch under `File → Build Settings…` if not), then sign in via `VRChat SDK → Show Control Panel → Authentication`. **[Docs]** <https://creators.vrchat.com/sdk/>

### Alternative: ALCOM (community)

**ALCOM** (by anatawa12, built on the `vrc-get` CLI) is a fast, open-source, **unofficial** alternative to VCC that runs on Windows, macOS and Linux (v1.1.8 at time of writing). It reads the same VPM package listings and templates. Not affiliated with VRChat. **[Community]** <https://vrc-get.anatawa12.com/en/alcom/>

The official VPM CLI also works on non-Windows platforms as of VCC 2.5.0 (no longer requires Unity / Unity Hub to be configured). **[Docs]** <https://vcc.docs.vrchat.com/news/release-2.5.0>

## Manual path (existing Unity project)

1. Open your existing project in VCC's **Projects** tab. If it doesn't appear, click **Add** and point at the project folder. (VCC may offer to migrate a legacy project first.)
2. Add **VRChat SDK - Worlds**. That's all — UdonSharp and ClientSim come with it.
3. If the project still lists `com.vrchat.udonsharp` or `com.vrchat.clientsim`, let VCC/Resolver remove them (they are `legacyPackages` of the Worlds SDK). **[Docs]**
4. VCC records the dependency in the VPM manifest inside `Packages/` (alongside Unity's `manifest.json`) and installs the packages into `Packages/`. **[Docs]** <https://vcc.docs.vrchat.com/vpm/source-control>

After install, the project structure relevant to U# looks like:

```
YourProject/
├── Assets/
│   ├── UdonSharp/                         # Auto-created data folder: UdonSharpDataLocator.asset,
│   │                                      #   Settings/UdonSharpSettings.asset, PrefabBuild/ intermediates
│   └── (your scripts and scenes)
├── Packages/
│   ├── com.vrchat.worlds/                 # VRChat Worlds SDK ("VRChat SDK - Worlds" in the Project window)
│   │   └── Integrations/
│   │       ├── UdonSharp/                 # UdonSharp: Editor/ (compiler, editors, build hooks), Runtime/, UdonSharpLocator.asset
│   │       └── ClientSim/                 # Editor-time client simulator
│   └── com.vrchat.base/                   # Shared SDK base
└── ProjectSettings/
```

The UdonSharp root is resolved through `UdonSharpLocator.asset` (`Packages/com.vrchat.worlds/Integrations/UdonSharp/UdonSharpLocator.asset`); the per-project data folder defaults to `Assets/UdonSharp/`. Don't move or delete either locator asset. **[Source]** `Runtime/UdonSharpLocator.cs`, `Runtime/UdonSharpDataLocator.cs`

## Verify the install

In Unity:

1. Open `VRChat SDK → Show Control Panel`. The **Builder** tab should report no errors.
2. Right-click in the Project view → `Create → U# Script`. After you choose a file name, a `.cs` file and matching `.asset` program file should appear. **[Source]**
3. The `VRChat SDK → Udon Sharp` menu should exist (Refresh All UdonSharp Assets, Class Exposure Tree, Parse Logs from File, …). U# settings live under **Edit → Project Settings → Udon Sharp**. **[Docs]** <https://udonsharp.docs.vrchat.com/configuration>
4. The Unity console should be free of red errors.

## Where scripts may live

- **Assets/** — always supported.
- **Packages/** (embedded/local packages) — supported again as of **SDK 3.10.5+**. Creating scripts entirely outside the project is still unsupported and now fails with a descriptive error. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-5/>
- Scripts inside a custom assembly definition need a [U# Assembly Definition](../editor-api/assembly-definition.md).

## Common install issues

| Symptom | Cause / Fix |
|---|---|
| "Multiple UdonSharp data locators found" | You duplicated the `Assets/UdonSharp` folder. Remove duplicates. |
| Right-click → Create has no `U# Script` entry | Worlds SDK didn't import cleanly (or the project is an Avatar project). Reimport **Packages → VRChat SDK - Worlds**, or re-resolve packages in VCC. |
| Errors about duplicate UdonSharp / ClientSim types after an upgrade | A legacy stand-alone `com.vrchat.udonsharp` / `com.vrchat.clientsim` (or an old `Assets/UdonSharp` 0.x install) is still present alongside the bundled copy. Remove it via VCC; for 0.x, follow the [migration guide](https://udonsharp.docs.vrchat.com/migration). **[Docs]** |
| Editor errors about missing `VRC.Udon` namespace | The VRChat Worlds SDK is not installed. Add via VCC. |
| Inspector won't display `UdonSharpBehaviour` subclasses | Compilation errors elsewhere are blocking. Check the Unity console first. |
| Unity Hub warns about a security issue / suggests a newer 2022.3 patch | Ignore for VRChat projects — stay on 2022.3.22f1. **[Docs]** |

## VS Code / Rider / IDE setup

UdonSharp does not require a special IDE configuration — it uses Unity's regular C# project files. For autocomplete on VRChat SDK types:

- **Rider**: works out of the box.
- **VS Code**: install the Unity extension, then in Unity go to **Edit → Preferences → External Tools** and click **Regenerate project files**.
- **Visual Studio**: works out of the box.

The reference video at <https://www.youtube.com/watch?v=ihVAKiJdd40> covers VS Code specifically. **[Unverified]** — predates Unity 2022 / the bundled-U# layout.

## Next

→ [First script](first-script.md)
