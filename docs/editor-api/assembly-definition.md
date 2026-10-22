# UdonSharpAssemblyDefinition

A `ScriptableObject` you place next to a Unity `AssemblyDefinitionAsset` to mark that assembly as containing UdonSharp code.

Source: `Editor/UdonSharpAssemblyDefinition.cs` (bundled at `Packages/com.vrchat.worlds/Integrations/UdonSharp/`). **[Source]**

## Why

A Unity assembly definition (`*.asmdef`) groups scripts into a separate DLL with explicit dependencies. UdonSharp needs to know which of those assemblies contain U# scripts so its compiler scans the right places. The marker is the `UdonSharpAssemblyDefinition` `ScriptableObject`:

```csharp
public class UdonSharpAssemblyDefinition : ScriptableObject {
    public AssemblyDefinitionAsset sourceAssembly;
}
```

## How to use it

1. Create or open an assembly definition asset for your U# scripts (e.g. `MyWorld.UdonScripts.asmdef`).
2. Select the `.asmdef` and right-click → **Create → U# Assembly Definition** (`Assets/Create/U# Assembly Definition`). With an `.asmdef` selected, the new asset is pre-filled and named after it; otherwise it is created as `AsmDef.asset`. **[Source]**
3. If needed, drag your `.asmdef` onto the `sourceAssembly` field of the new asset (changing it resets U#'s assembly caches).
4. UdonSharp's compiler now scans the assembly for U# scripts.

The asset itself has no runtime effect — it's pure compile-time metadata. **[Source]**

## Assembly Version Defines (SDK 3.10.5+)

U# scripts now honour Unity's **Version Defines** from their assembly definition, so you can gate code on the installed SDK version at compile time. The release-notes example: use `3.10` as the *Version Define Expression* in your asmdef, then guard code with `#if SDK_310_OR_NEWER`. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-5/>

In Unity's asmdef inspector → *Version Defines*, that means roughly: **Resource** = the Worlds SDK package (`com.vrchat.worlds`), **Define** = `SDK_310_OR_NEWER` (you choose the symbol name), **Expression** = `3.10`. The exact resource name used by the release-notes example isn't stated. **[Unverified]**

```csharp
#if SDK_310_OR_NEWER
    // code using 3.10-only APIs
#else
    // fallback for older SDKs
#endif
```

The release notes present this as new ("can now use"), so don't rely on version defines inside U# scripts for projects that must also build on SDKs older than 3.10.5. For editor-side SDK detection see also <https://creators.vrchat.com/sdk/detecting-vrcsdk/>. **[Docs]**

## When you need it

- You've moved U# scripts into a Unity assembly definition for incremental compile speed.
- You're authoring a UPM package that ships U# scripts (creating U# scripts under `Packages/` is supported again in **SDK 3.10.5+** **[Docs]**).
- You want [Assembly Version Defines](#assembly-version-defines-sdk-3105).

The default UdonSharp install scans `Assembly-CSharp` (Unity's default no-asmdef assembly). If all your U# code lives there, you don't need this.

## See also

- Unity manual on [Assembly Definitions](https://docs.unity3d.com/Manual/ScriptCompilationAssemblyDefinitionFiles.html).
- [Program Asset](program-asset.md)
