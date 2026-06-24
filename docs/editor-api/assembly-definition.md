# UdonSharpAssemblyDefinition

A `ScriptableObject` you place next to a Unity `AssemblyDefinitionAsset` to mark that assembly as containing UdonSharp code.

Source: `Editor/UdonSharpAssemblyDefinition.cs`. **[Source]**

## Why

A Unity assembly definition (`*.asmdef`) groups scripts into a separate DLL with explicit dependencies. UdonSharp needs to know which of those assemblies contain U# scripts so its compiler scans the right places. The marker is the `UdonSharpAssemblyDefinition` `ScriptableObject`:

```csharp
public class UdonSharpAssemblyDefinition : ScriptableObject {
    public AssemblyDefinitionAsset sourceAssembly;
}
```

## How to use it

1. Create or open an assembly definition asset for your U# scripts (e.g. `MyWorld.UdonScripts.asmdef`).
2. Right-click in the Project window → Create → U# Assembly Definition.
3. Drag your `.asmdef` onto the `sourceAssembly` field of the new asset.
4. UdonSharp's compiler now scans the assembly for U# scripts.

The asset itself has no runtime effect — it's pure compile-time metadata. **[Source]**

## When you need it

- You've moved U# scripts into a Unity assembly definition for incremental compile speed.
- You're authoring a UPM package that ships U# scripts.

The default UdonSharp install scans `Assembly-CSharp` (Unity's default no-asmdef assembly). If all your U# code lives there, you don't need this.

## See also

- Unity manual on [Assembly Definitions](https://docs.unity3d.com/Manual/ScriptCompilationAssemblyDefinitionFiles.html).
- [Program Asset](program-asset.md)
