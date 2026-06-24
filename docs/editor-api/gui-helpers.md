# UdonSharpGUI

Static class `UdonSharpEditor.UdonSharpGUI`. Inspector-style drawing helpers used by U#'s default inspectors and available to custom inspectors. Source: `Editor/Editors/UdonSharpGUI.cs`. **[Source]**

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
            serializedObject.ApplyModifiedProperties();
            t.ApplyProxyModifications(ProxySerializationPolicy.All);
        }

        UdonSharpGUI.DrawUILine();
        UdonSharpGUI.DrawUtilities(target);   // optional, gives users a "compile" button
    }
}
#endif
```

## `ApplyProxyModifications`

An extension method on `UdonSharpBehaviour` that wraps `UdonSharpEditorUtility.CopyProxyToUdon`:

```csharp
behaviour.ApplyProxyModifications(ProxySerializationPolicy.All);
```

Use after any code that modifies proxy fields outside `SerializedProperty`-based edits.

## See also

- [Editor Utility](editor-utility.md)
- [Proxy ↔ Backing](proxy-backing.md)
- [Undo](undo.md)
