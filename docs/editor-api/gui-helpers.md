# UdonSharpGUI

Static class `UdonSharpEditor.UdonSharpGUI`. Inspector-style drawing helpers used by U#'s default inspectors and available to custom inspectors. Source: `Editor/Editors/UdonSharpGUI.cs` (bundled at `Packages/com.vrchat.worlds/Integrations/UdonSharp/`). **[Source]**

Wrap every call in `#if !COMPILER_UDONSHARP && UNITY_EDITOR`.

## Headers

```csharp
public static bool DrawDefaultUdonSharpBehaviourHeader(
    UnityEngine.Object target,
    bool skipLine = false,
    bool drawScript = true);

public static bool DrawDefaultUdonSharpBehaviourHeader(
    UnityEngine.Object[] targets,
    bool skipLine = false,
    bool drawScript = true);
```

The one-stop header for a custom inspector. Draws:
- Program-source selector (with type validation).
- Sync mode dropdown.
- Interact settings.
- Optionally the script field (`drawScript`).
- An optional separator line below (`skipLine` to suppress).

Returns `true` if the user just changed the program source — return early from your inspector when this happens.

## Standalone draws

```csharp
public static void DrawCompileErrorTextArea();
```

Red-bordered error text area. Appears when the current program asset has a compile error.

```csharp
public static void DrawUtilities(UnityEngine.Object target);
public static void DrawUtilities(UnityEngine.Object[] targets);
```

Compile button + "Send Custom Event" UI for testing without entering Play mode.

```csharp
public static void DrawVariables(UnityEngine.Object behaviour);
public static void DrawVariables(UnityEngine.Object[] behaviours);
```

The default field inspector for a behaviour's `[UdonSynced]` and serialized fields.

```csharp
public static bool DrawProgramSource(UnityEngine.Object target, bool drawScript = true);
public static bool DrawProgramSource(UnityEngine.Object[] targets, bool drawScript = true);
```

Just the program-source selector. Returns `true` on change.

```csharp
public static void DrawSyncSettings(UnityEngine.Object target);
public static void DrawSyncSettings(UnityEngine.Object[] targets);
```

The sync mode UI by itself.

```csharp
public static void DrawInteractSettings(UnityEngine.Object target);
public static void DrawInteractSettings(UnityEngine.Object[] targets);
public static void DrawInteractSettings(UdonBehaviour[] behaviours);
```

The interact-event UI.

```csharp
public static void DrawUILine();
public static void DrawUILine(Color color, int thickness = 2, int padding = 4);
```

Separator line.

```csharp
public static void DrawPublicVariables(UdonBehaviour behaviour, UdonSharpProgramAsset programAsset, ref bool dirty);
```

Draws a backing `UdonBehaviour`'s public-variable table directly. **[Source]**

`DrawConvertToUdonBehaviourButton(...)` still exists for compatibility but always returns `false` (conversion is gone in U# 1.x). **[Source]**

## Custom inspector skeleton

```csharp
#if !COMPILER_UDONSHARP && UNITY_EDITOR
using UnityEditor;
using UdonSharpEditor;

[CustomEditor(typeof(MyScript))]
public class MyScriptEditor : Editor {
    SerializedProperty _someProp;

    void OnEnable() {
        _someProp = serializedObject.FindProperty("someField");
    }

    public override void OnInspectorGUI() {
        if (UdonSharpGUI.DrawDefaultUdonSharpBehaviourHeader(target)) return;

        MyScript t = (MyScript)target;

        EditorGUI.BeginChangeCheck();
        EditorGUILayout.PropertyField(_someProp);
        if (EditorGUI.EndChangeCheck()) {
            serializedObject.ApplyModifiedProperties();   // that's all U# 1.x needs
        }

        UdonSharpGUI.DrawUILine();
        UdonSharpGUI.DrawUtilities(target);   // optional, gives users a "compile" button
    }
}
#endif
```

## `ApplyProxyModifications` / `UpdateProxy` (obsolete)

These `UdonSharpBehaviour` extension methods are `[Obsolete]` and have **empty bodies** in U# 1.x: "ApplyProxyModifications is no longer needed for editor scripts to modify the state of U# behaviours. If you need editor scripts to update behaviours in play mode, consider using CopyProxyToUdon". **[Source]** `Editor/Editors/UdonSharpComponentExtensions.cs`

```csharp
// 0.x-era code — compiles with a warning, does nothing:
behaviour.ApplyProxyModifications();
// 1.x: edit fields directly; in Play mode, push to the running backing explicitly:
UdonSharpEditorUtility.CopyProxyToUdon(behaviour, ProxySerializationPolicy.All);
```

Inside a custom inspector drawn through UdonSharp's editor wrapper, the Play-mode copy in both directions is already done for you. **[Source]** The official [editor-scripting page](https://udonsharp.docs.vrchat.com/editor-scripting) still shows the 0.x calls.

## See also

- [Editor Utility](editor-utility.md)
- [Proxy ↔ Backing](proxy-backing.md)
- [Undo](undo.md)
