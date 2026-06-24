# Installation

## Requirements

| Requirement | Version |
|---|---|
| Unity | **2019.4.31f1** (exact — newer versions are not supported by the World SDK as of writing) |
| VRChat Creator Companion (VCC) | latest |
| VRChat World SDK 3 | latest |
| UdonSharp | latest |

## Recommended path: VRChat Creator Companion

1. Download VCC from <https://vrchat.com/home/download>.
2. Install it, sign in with your VRChat account, and complete the initial setup. It will install Unity Hub and the required Unity version if missing.
3. Click **Projects → New → UdonSharp**. VCC creates a project with `com.vrchat.worlds` and `com.vrchat.UdonSharp` already wired up.
4. Open the project from VCC. Unity will resolve packages on first launch.

## Manual path (existing Unity project)

1. Open VCC → **Settings → Packages → Show Pre-release Packages** (if you need pre-release builds).
2. Open your existing project in VCC's **Projects** tab. If it doesn't appear, click **Add** and point at the project folder.
3. Add packages: **VRChat Worlds SDK**, then **UdonSharp**.
4. VCC writes to `Packages/manifest.json` and copies the assets into `Packages/`.

After install, the project structure relevant to U# looks like:

```
YourProject/
├── Assets/
│   ├── UdonSharp/                         # Auto-created on first import. Holds UdonSharpDataLocator and utility scripts.
│   └── (your scripts and scenes)
├── Packages/
│   ├── com.vrchat.UdonSharp/              # UdonSharp package (compiler + runtime + editor)
│   ├── com.vrchat.worlds/                 # VRChat World SDK 3
│   └── com.vrchat.base/                   # Shared SDK base
└── ProjectSettings/
```

## Verify the install

In Unity:

1. Open `VRChat SDK → Show Control Panel`. The **Builder** tab should report no errors.
2. Right-click in the Project view → `Create → U# script`. A `.cs` file and matching `.asset` program file should appear.
3. The Unity console should be free of red errors. Yellow warnings about `_skipEvents` or backing UdonBehaviours are normal.

## Common install issues

| Symptom | Cause / Fix |
|---|---|
| "Multiple UdonSharp data locators found" | You duplicated the `Assets/UdonSharp` folder. Remove duplicates. |
| Right-click → Create has no `U# script` entry | UdonSharp didn't import cleanly. Reimport `Packages/com.vrchat.UdonSharp/`. |
| Editor errors about missing `VRC.Udon` namespace | The VRChat Worlds SDK is not installed alongside UdonSharp. Add via VCC. |
| Inspector won't display `UdonSharpBehaviour` subclasses | Compilation errors elsewhere are blocking. Check the Unity console first. |

## VS Code / Rider / IDE setup

UdonSharp does not require a special IDE configuration — it uses Unity's regular C# project files. For autocomplete on VRChat SDK types:

- **Rider**: works out of the box.
- **VS Code**: install the Unity extension, then in Unity go to **Edit → Preferences → External Tools** and click **Regenerate project files**.
- **Visual Studio**: works out of the box.

The reference video at <https://www.youtube.com/watch?v=ihVAKiJdd40> covers VS Code specifically.

## Next

→ [First script](first-script.md)
