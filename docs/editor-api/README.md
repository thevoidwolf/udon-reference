# Editor API

UdonSharp ships a substantial editor-side API for writing custom tooling: inspectors, batch-editing scripts, build-time validators. None of this runs in Udon — it's Unity Editor code that orchestrates the proxy ↔ backing-UdonBehaviour duality.

All symbols in this section live under `UdonSharpEditor` (Editor assembly). Wrap calls in `#if !COMPILER_UDONSHARP && UNITY_EDITOR`.

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

A `UdonSharpBehaviour` is a regular `MonoBehaviour`. At edit time, Unity's serializer reads its public fields and stores them in the scene/prefab. But the compiled program that actually runs at runtime is an Udon Program Asset attached to a sibling `UdonBehaviour` component on the same GameObject (`_udonSharpBackingUdonBehaviour` private field; see `Runtime/UdonSharpBehaviour.cs:263`). **[Source]**

The two components must stay in lockstep: change a value in the C# inspector, that value needs to land on the backing UdonBehaviour's public-variable table (which is what the runtime reads). The Editor API exists to manage this synchronization.

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
            t.targetPosition = newPos;
            t.ApplyProxyModifications(ProxySerializationPolicy.All);  // push to backing
        }
    }
}
#endif
```

## Community editor tools

| Tool | What it does | Status |
|---|---|---|
| **ClientSim** (VRChat staff, CyanLaser) | The editor-time client simulator. Bundled inside the World SDK now (no separate install). | Active <https://github.com/vrchat-community/ClientSim> |
| **CyanEmu** | Predecessor to ClientSim; SDK2 + SDK3 support. **Archived.** | <https://github.com/CyanLaser/CyanEmu> |
| **UdonToolkit** (orels1) | Attribute system for custom inspectors without raw IMGUI. **Stale (v1.2.1 Nov 2022); predates U# 1.0.** | <https://github.com/orels1/UdonToolkit>, <https://ut.orels.sh/> |
| **VRWorld Toolkit** (oneVR, curated) | 90+ scene validators, Build Report, Mass Texture Importer, `DisableOnBuild`. | Active <https://github.com/oneVR/VRWorldToolkit> |
| **CyanTrigger** | In-scene Udon compiler with SDK2-style UI. | <https://github.com/CyanLaser/CyanTrigger> |
| **EasyEventEditor** (Merlin) | Better UnityEvent editor. | <https://github.com/MerlinVR> |
| **VRCNetworkDebugger** (Centauri2442) | Drag-in world-side overlay: recent transmissions, per-object bandwidth. | <https://github.com/Centauri2442/VRCNetworkDebugger> |
| **VUdon-Logger** (Varneon) | In-world console window for VR runtime log capture. | <https://github.com/Varneon/VUdon> |
| **udonity** (Varneon) | In-runtime editor inside VRChat. | <https://github.com/Varneon/VUdon> |

All **[Community]** unless otherwise noted.

## ClientSim known gaps

Multi-player networking is **not** simulated — ClientSim runs single-client only. Tracked open issues that frequently bite developers:

- `Input.GetKey` broken with SDK 3.6.1 (#117). **[Community]** <https://github.com/vrchat-community/ClientSim/issues/117>
- `SetVelocity` + `GetVelocity` in the same frame fails (#129).
- `VRCPlayerApi.GetTrackingData` returns world origin instead of avatar transforms for remote players (#115).
- `OnBecameVisible` / `OnBecameInvisible` don't fire correctly (#130).
- No automated test harness (#133).

## See also

- [UdonSharpBehaviour](../udonsharp-behaviour/README.md)
- [Compiler constraints](../compiler/README.md) — runtime versus editor.
- [References → Community Resources](../references/community-resources.md) — full editor / debug tool catalogue.
