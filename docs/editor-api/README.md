# Editor API

UdonSharp ships a substantial editor-side API for writing custom tooling: inspectors, batch-editing scripts, build-time validators. None of this runs in Udon — it's Unity Editor code that orchestrates the proxy ↔ backing-UdonBehaviour duality.

All symbols in this section live under `UdonSharpEditor` (Editor assembly). Wrap calls in `#if !COMPILER_UDONSHARP && UNITY_EDITOR`.

**Source location.** Since SDK 3.4.0 UdonSharp is bundled in the Worlds SDK, so the source files cited as `Editor/...` / `Runtime/...` in this section live under `Packages/com.vrchat.worlds/Integrations/UdonSharp/` (Project window: *Packages → VRChat SDK - Worlds → Integrations → UdonSharp*). The stand-alone repo <https://github.com/vrchat-community/UdonSharp> still exists, but its code stops at U# 1.1.9 (Aug 2023) and keeps the older `Packages/com.vrchat.UdonSharp/` layout — read the bundled copy for current behaviour. Paths below were checked against the copy shipped in SDK 3.10.3. **[Docs]** <https://creators.vrchat.com/releases/release-3-4-0/> / **[Source]**

> **U# 1.x vs the official editor-scripting page.** <https://udonsharp.docs.vrchat.com/editor-scripting> still documents the 0.x workflow (`UpdateProxy()` / `ApplyProxyModifications()`, disabled proxies). In the bundled 1.x source those extension methods are `[Obsolete]` **no-ops** — editor code can read and write proxy fields directly, and only Play-mode edits need an explicit `CopyProxyToUdon`. **[Source]** `Editor/Editors/UdonSharpComponentExtensions.cs`

## Pages

| Page | Topic |
|---|---|
| [Editor Utility](editor-utility.md) | `UdonSharpEditorUtility` — proxy/backing lookups, data sync. |
| [Proxy ↔ Backing model](proxy-backing.md) | The architectural pattern. |
| [Program Asset](program-asset.md) | `UdonSharpProgramAsset` API. |
| [GUI Helpers](gui-helpers.md) | `UdonSharpGUI` — inspector-style draws. |
| [Undo](undo.md) | `UdonSharpUndo` — Undo-safe component ops. |
| [Component Extensions](component-extensions.md) | `AddUdonSharpComponent<T>` and friends. |
| [Assembly Definition](assembly-definition.md) | `UdonSharpAssemblyDefinition`. |
| [Build Pipeline](build-pipeline.md) | Build-time hooks. |

## Why "proxy and backing"

A `UdonSharpBehaviour` is a regular `MonoBehaviour`. At edit time, Unity's serializer reads its public fields and stores them in the scene/prefab. But the compiled program that actually runs at runtime is an Udon Program Asset attached to a sibling `UdonBehaviour` component on the same GameObject (`_udonSharpBackingUdonBehaviour` private field in `Runtime/UdonSharpBehaviour.cs`, ~line 422 in SDK 3.10.3). **[Source]**

The two components must stay in lockstep: change a value in the C# inspector, that value needs to land on the backing UdonBehaviour's public-variable table (which is what the runtime reads). In U# 1.x UdonSharp does this itself when the scene is processed for Play mode or a build; the Editor API exposes the same synchronization for tooling (and for Play-mode edits). **[Source]**

## Skeleton custom editor

```csharp
#if !COMPILER_UDONSHARP && UNITY_EDITOR
using UnityEditor;
using UdonSharpEditor;

[CustomEditor(typeof(MyScript))]
public class MyScriptEditor : Editor {
    public override void OnInspectorGUI() {
        if (UdonSharpGUI.DrawDefaultUdonSharpBehaviourHeader(target)) return;

        MyScript t = (MyScript)target;

        EditorGUI.BeginChangeCheck();
        Vector3 newPos = EditorGUILayout.Vector3Field("Target", t.targetPosition);
        if (EditorGUI.EndChangeCheck()) {
            Undo.RecordObject(t, "Move target");
            t.targetPosition = newPos;   // editing the proxy is enough in U# 1.x
        }
    }
}
#endif
```

## Community editor tools

| Tool | What it does | Status |
|---|---|---|
| **ClientSim** (VRChat staff, CyanLaser) | The editor-time client simulator. Bundled in the Worlds SDK (`Packages/com.vrchat.worlds/Integrations/ClientSim/`) since SDK 3.4.0 — no separate install. **[Docs]** The GitHub repo still hosts the source and an issue tracker. | Active — docs <https://creators.vrchat.com/worlds/clientsim/>, repo <https://github.com/vrchat-community/ClientSim> |
| **CyanEmu** | Predecessor to ClientSim; SDK2 + SDK3 support. **Archived.** | <https://github.com/CyanLaser/CyanEmu> |
| **UdonToolkit** (orels1) | Attribute system for custom inspectors without raw IMGUI. **Stale (v1.2.1 Nov 2022); predates U# 1.0.** | <https://github.com/orels1/UdonToolkit>, <https://ut.orels.sh/> |
| **VRWorld Toolkit** (oneVR, curated) | 90+ scene validators, Build Report, Mass Texture Importer, `DisableOnBuild`. | Active <https://github.com/oneVR/VRWorldToolkit> |
| **CyanTrigger** | In-scene Udon compiler with SDK2-style UI. | <https://github.com/CyanLaser/CyanTrigger> |
| **EasyEventEditor** (Merlin) | Better UnityEvent editor. A copy ships inside UdonSharp itself (`Editor/Editors/EasyEventEditor.cs`, settings under `Edit → Easy Event Editor Settings`). **[Source]** | Stale (standalone repo, Dec 2020) <https://github.com/MerlinVR/EasyEventEditor> |
| **VRCNetworkDebugger** (Centauri2442) | Drag-in world-side overlay: recent transmissions, per-object bandwidth. | Stale (Feb 2024) <https://github.com/Centauri2442/VRCNetworkDebugger> |
| **VUdon-Logger** (Varneon) | In-world console window for VR runtime log capture. | <https://github.com/Varneon/VUdon-Logger> |
| **Udonity** (Varneon) | In-runtime editor inside VRChat. Listed as a package of the VUdon ecosystem. | <https://github.com/Varneon/VUdon> |

All **[Community]** unless otherwise noted.

## Built-in editor menu items

Under **`VRChat SDK → Udon Sharp`** (all **[Source]**, `MenuItem` attributes in the bundled package):

| Menu item | What it does |
|---|---|
| Refresh All UdonSharp Assets | Re-scans the AssetDatabase for `UdonSharpProgramAsset`s, tries to recover ones Unity's type search missed, and rebuilds the program-asset cache. |
| Force Upgrade | Queues every program asset (and the scene/prefab pass) for the U# data upgrader again. |
| Class Exposure Tree | Opens the "Udon Type Exposure Tree" window for browsing which types/members are exposed to Udon. |
| Node Definition Grabber | Small window whose button copies all Udon node definition names to the clipboard. |
| Parse Logs from File | **SDK 3.10.2+.** Opens a saved VRChat client log and runs the runtime log watcher over it, so Udon exceptions are matched to scripts in the editor console. Also callable as `UdonSharpEditor.LogFileParseUtility.ParseLogsFromFile(string path)` (`[PublicAPI]`). **[Docs]** <https://creators.vrchat.com/releases/release-3-10-2/> |

Also: `Assets → Create → U# Script` and `Assets → Create → U# Assembly Definition`; settings under **Edit → Project Settings → Udon Sharp**.

## ClientSim known gaps

ClientSim "only simulates the local VRChat player, not remote players", and serialization results are simulated (e.g. `SerializationResult.success` is always true). **[Docs]** <https://creators.vrchat.com/worlds/clientsim/> Issues on the ClientSim tracker that frequently bite developers (status checked Oct 2026; all **[Community]** unless noted):

- `GetTrackingData(...AvatarRoot)` returns the world origin (#115, open). <https://github.com/vrchat-community/ClientSim/issues/115>
- Other velocity changes not applied on the same frame `SetVelocity` is called (#129, open). <https://github.com/vrchat-community/ClientSim/issues/129>
- `OnBecameVisible` / `OnBecameInvisible` don't fire correctly (#130, open). <https://github.com/vrchat-community/ClientSim/issues/130>
- The old automated-testing feature appears unavailable in the bundled ClientSim (#133, open question). <https://github.com/vrchat-community/ClientSim/issues/133>
- *Historical:* `Input.GetKey` broken in SDK 3.6.1 (#117) — fixed in SDK 3.6.2 ("Fixed issue with broken input events in ClientSim"), **[Docs]** <https://creators.vrchat.com/releases/release-3-6-2/>, although the GitHub issue was never closed. <https://github.com/vrchat-community/ClientSim/issues/117>

Recent ClientSim fixes shipped with the SDK: multiple `UdonBehaviour`s on one GameObject work, new *Hide Menu on Launch* setting (3.7.6); persistence no longer re-saves while a save is in progress, no Unity events such as `OnDisable()` during the Play→Edit transition, no NRE with domain reload disabled (3.10.5). **[Docs]**

## See also

- [UdonSharpBehaviour](../udonsharp-behaviour/README.md)
- [Compiler constraints](../compiler/README.md) — runtime versus editor.
- [References → Community Resources](../references/community-resources.md) — full editor / debug tool catalogue.
