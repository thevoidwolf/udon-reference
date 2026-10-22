# Build Pipeline

UdonSharp integrates with the VRChat SDK build pipeline through two `IVRCSDKBuildRequestedCallback`s. Both are `internal` to the package, but reading them clarifies what runs when.

Source: `Editor/BuildUtilities/` (in the Worlds SDK: `Packages/com.vrchat.worlds/Integrations/UdonSharp/Editor/BuildUtilities/`). **[Source]** — checked against the copy in SDK 3.10.3.

The interface itself (`VRC.SDKBase.Editor.BuildPipeline.IVRCSDKBuildRequestedCallback`: `int callbackOrder`, `bool OnBuildRequested(VRCSDKRequestedBuildType requestedBuildType)`; return `false` to abort; `VRCSDKRequestedBuildType` is `Avatar` or `Scene`) is documented at **[Docs]** <https://creators.vrchat.com/sdk/build-pipeline-callbacks-and-interfaces/>. Callbacks run in ascending `callbackOrder`.

## `UdonSharpBuildChecks` — `callbackOrder => -1`

Runs **first**. Forces an `AssetDatabase.Refresh` so Unity picks up script changes, then aborts the build ("Scripts are in the process of compiling, please retry build after scripts have compiled.") if Unity is still compiling. With **Force compile on upload** enabled in *Project Settings → Udon Sharp*, it instead does a synchronous refresh/compile (marked experimental in source). The comment in source explains why: uploading while Unity's own scripts are uncompiled can corrupt assemblies and wipe UdonBehaviour variables. **[Source]**

## `UdonSharpBuildCompile` — `callbackOrder => 100`

Runs after the checks. For world (`Scene`) builds it:

```csharp
// Simplified from Editor/BuildUtilities/UdonSharpBuildCompile.cs
public bool OnBuildRequested(VRCSDKRequestedBuildType requestedBuildType) {
    if (requestedBuildType == VRCSDKRequestedBuildType.Avatar) return true;
    if (UdonSharpSettings.GetSettings().disableUploadCompile) return true;   // "Disable compile on upload"

    UdonSharpCompilerV1.CompileSync(new UdonSharpCompileOptions() { IsEditorBuild = false });
    UdonSharpEditorCache.SaveAllCache();

    if (UdonSharpProgramAsset.AnyUdonSharpScriptHasError()) return false;    // logs + editor notification
    if (UdonSharpEditorManager.RunAllUpgrades()) return false;               // asks you to retry after upgrade
    return true;
}
```

So every upload gets a fresh, non-editor (`IsEditorBuild = false`) compile of all U# scripts, and the build is aborted on any compile error. **[Source]** Authors typically don't interact with this directly.

## Adding your own pre-build hook

There is no `[PublicAPI]` extension hook on UdonSharp's pipeline. Add your own `IVRCSDKBuildRequestedCallback` and pick `callbackOrder` relative to U#'s two callbacks:

| Your `callbackOrder` | Runs… |
|---|---|
| `< -1` | before U#'s asset refresh / compiling check. |
| `0`–`99` | after the refresh, **before** the U# compile — program assets may still be stale. |
| `> 100` | after the U# compile; `UdonSharpProgramAsset.AnyUdonSharpScriptHasError()` now reflects this build. |

```csharp
#if UNITY_EDITOR
using UnityEditor;
using VRC.SDKBase.Editor.BuildPipeline;

public class MyPreBuild : IVRCSDKBuildRequestedCallback {
    public int callbackOrder => -100;   // run before U#'s checks and compile

    public bool OnBuildRequested(VRCSDKRequestedBuildType type) {
        if (type != VRCSDKRequestedBuildType.Scene) return true;
        // Validate, sanitize, set version numbers, etc.
        if (somethingWrong) {
            EditorUtility.DisplayDialog("Build blocked", "...", "OK");
            return false;
        }
        return true;
    }
}
#endif
```

For per-component work, the SDK also offers `IPreprocessCallbackBehaviour` (`OnPreprocess()`, `PreprocessOrder`) and the `IEditorOnly` marker. **[Docs]** For event-style hooks (`VRCSdkControlPanel.OnSdkPanelEnable` → `TryGetBuilder<…>`, `OnSdkBuildStart`, build/upload success & error events) see the Public SDK API; the page's examples cover the avatar builder, so check the world-builder interface in the SDK's `Editor/VRCSDK/SDK3/Public SDK API` folder for exact names. **[Docs]** <https://creators.vrchat.com/sdk/public-sdk-api/>

## Order of operations during build

1. VRChat SDK collects build-requested callbacks.
2. They run in `callbackOrder` (lowest to highest); any `false` aborts.
3. `UdonSharpBuildChecks` (`-1`) refreshes assets / blocks if Unity is compiling.
4. `UdonSharpBuildCompile` (`100`) compiles all U# programs and blocks on errors.
5. SDK proceeds to scene processing — UdonSharp copies every proxy's data into its backing `UdonBehaviour` here (`OnSceneBuild` → `CopyProxyToUdon(…, ProxySerializationPolicy.All)`). **[Source]** `Editor/UdonSharpEditorManager.cs`
6. The world is packed, then tested locally or uploaded.

Related SDK changes:

- **Build panel overhaul (SDK 3.7.6+).** The Builder panel was reworked: multi-platform *Build & Publish* runs each selected platform in sequence (PC first), content warnings became a dropdown, tags moved to a popup, thumbnail capture improved. **[Docs]** <https://creators.vrchat.com/releases/release-3-7-6/>
- **Build & Test** is available without a trust-rank requirement (**SDK 3.9.0+**), on Quest (**3.7.5+**), Android devices via ADB, and iOS (**3.8.1+**). **[Docs]** See [First Script → Attach it](../getting-started/first-script.md#attach-it).
- **Platform switch recompile (SDK 3.9.0+).** U# scripts are recompiled when you switch build platform, so platform `#if`s take effect without a manual recompile. **[Docs]** <https://creators.vrchat.com/releases/release-3-9-0/>
- **Build & Reload** (Build & Test with 0 clients) is treated as a Test build in `VRC_SdkBuilder.ActiveBuildType` (**SDK 3.10.5+**). **[Docs]** <https://creators.vrchat.com/releases/release-3-10-5/>
- **Known issue (listed in several 3.8.x–3.10.x release notes):** a rare "All pipe instances are busy" build error is a Unity issue; restart the editor. **[Docs]**

## Debugging in-game (community-known)

VRChat ships in-game debug views accessible via **`RShift + ` + N`** (where N selects the page):

| Page | Content |
|---|---|
| 1 | AssetBundle / memory info |
| 2 | Version & info |
| 3 | Log viewer — `Debug.Log`, UdonBehaviour crashes, Unity errors |
| 4 | Players (incl. "Suffering" field flagging excessive per-player Udon data) |
| 6 | Net Objects — ownership, group, bandwidth, sync interval |
| 7–0 | Overlays: PhysBones, network objects, players, UI shapes |

World creator gate applies to view 6, Audio Sources, and overlays unless "World Debugging" is enabled. **[Docs]** <https://creators.vrchat.com/worlds/udon/world-debug-views/>

VRChat launch flags for verbose Udon logging: `--enable-udon-debug-logging`, `--enable-debug-gui` and `--enable-sdk-log-levels` (plus `--no-vr` for desktop testing). Set via VRC Quick Launcher in VCC's Tools tab, a batch file, or Steam launch options. **[Docs]** <https://creators.vrchat.com/worlds/udon/debugging-udon-projects/>

Log file location (PC): `C:\Users\<user>\AppData\LocalLow\VRChat\VRChat\output_log_<timestamp>.txt`. Contains UdonBehaviour exceptions with script + line, all `Debug.Log` output, asset-bundle load events, network warnings. **[Docs]** <https://help.vrchat.com/hc/en-us/articles/9521522810899-Where-do-I-find-my-Output-Logs-and-Crash-Dumps>

Getting those logs back into Unity:

- **Runtime log watcher.** With *Listen for client exceptions* enabled (default), UdonSharp tails the client log during Build & Test and maps Udon exceptions to script + line in the editor console; *Output log watch mode* can forward other messages (optionally only those with given prefixes). **[Docs]** <https://creators.vrchat.com/worlds/udon/debugging-udon-projects/> / **[Source]** `Editor/UdonSharpRuntimeLogWatcher.cs`. Updated for VRChat's new log file format in **SDK 3.7.6** ("Fixed UdonSharp's log watcher by updating it to VRChat's new log file format"); on older SDKs, expect the watcher to miss exceptions from current clients. **[Docs]**
- **`VRChat SDK → Udon Sharp → Parse Logs from File`** (**SDK 3.10.2+**) — pick a saved `output_log_*.txt` (e.g. pulled from a Quest or a tester's PC) and run the same parsing over it. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-2/>

## Build-validator community tools

- **VRWorld Toolkit** (oneVR) — 90+ scene-validator checks (lighting, post-processing, SDK config, optimization), pre-build Mass Texture Importer, the `DisableOnBuild` GameObject tag. **[Community]** <https://github.com/oneVR/VRWorldToolkit>
- **EasyQuestSwitch** (Jordo, VRChat-curated) — pre-build pipeline component swapping for PC ↔ Android cross-platform builds. **[Community]**

## See also

- [Program Asset](program-asset.md) — methods you might want from a build hook.
- [Editor Utility](editor-utility.md)
- [References → Community Resources](../references/community-resources.md)
