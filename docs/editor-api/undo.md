# UdonSharpUndo

Static class `UdonSharpEditor.UdonSharpUndo`. Component add/remove operations that register with Unity's Undo system. Source: `Editor/Editors/UdonSharpUndo.cs`. **[Source]**

## Methods

```csharp
public static UdonSharpBehaviour AddComponent(GameObject gameObject, Type type);
public static T AddComponent<T>(GameObject gameObject) where T : UdonSharpBehaviour;
public static void DestroyImmediate(UdonSharpBehaviour behaviour);
```

## Why these wrappers exist

Adding a `UdonSharpBehaviour` to a GameObject in code involves:

1. Adding the C# proxy `MonoBehaviour`.
2. Adding the backing `UdonBehaviour`.
3. Linking proxy → backing through the hidden field.
4. Initializing the backing's program-source reference and variable table.

If any of those steps fails or isn't tracked by Unity's Undo, you end up with a corrupt state that Undo can't repair. `UdonSharpUndo` does all four atomically with full Undo recording.

Same logic for destruction — both halves must die together.

## Examples

```csharp
// Add a component to a newly created GameObject
GameObject go = new GameObject("Hub");
Undo.RegisterCreatedObjectUndo(go, "Create hub");
MyHub hub = UdonSharpUndo.AddComponent<MyHub>(go);
hub.target = someTransform;
hub.ApplyProxyModifications(ProxySerializationPolicy.All);
```

```csharp
// Remove
UdonSharpUndo.DestroyImmediate(hub);
```

## Without Undo

If you don't need Undo support (e.g. a one-shot menu item that builds from scratch), the extension method `GameObject.AddUdonSharpComponent<T>()` is available — see [Component Extensions](component-extensions.md). Prefer the Undo-aware version when the action is initiated by user input in the inspector / scene view.

## See also

- [Editor Utility](editor-utility.md)
- [Component Extensions](component-extensions.md)
