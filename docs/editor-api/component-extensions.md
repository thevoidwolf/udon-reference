# Component Extensions

Convenience extension methods on `GameObject` and `Component` for working with `UdonSharpBehaviour`s without going through Unity's regular `AddComponent` / `GetComponent` flow.

Source: `Editor/Editors/UdonSharpComponentExtensions.cs`. **[Source]**

## Adding components

```csharp
public static T AddUdonSharpComponent<T>(this GameObject gameObject) where T : UdonSharpBehaviour;
```

Adds a U# component **without** Undo registration. Use [`UdonSharpUndo.AddComponent`](undo.md) when Undo is desired.

```csharp
GameObject go = new GameObject("Bullet");
var b = go.AddUdonSharpComponent<Bullet>();
b.velocity = Vector3.forward * 10f;
b.ApplyProxyModifications(ProxySerializationPolicy.All);
```

## `GetUdonSharpComponent*` (deprecated)

Various `GetUdonSharpComponent<T>` / `GetUdonSharpComponents<T>` / `InChildren` / `InParent` extensions exist but are marked `[Obsolete]` in current versions. Use the standard `GetComponent<T>()` / `GetComponents<T>()` family — they work for `UdonSharpBehaviour` subclasses just like regular `MonoBehaviour`. **[Source]**

## When to use these

- **Editor-time scripts** that build prefabs or scene content procedurally.
- **Custom menu items** that add behaviours to selected objects.
- **Build-pipeline preprocessors** that inject helper components before upload.

Not for runtime — these are all in the Editor assembly.

## See also

- [Editor Utility](editor-utility.md)
- [Undo](undo.md)
