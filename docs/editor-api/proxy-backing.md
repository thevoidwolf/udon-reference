# Proxy ↔ Backing Model

The architectural pattern behind UdonSharp's editor integration.

## What lives where

Every `UdonSharpBehaviour` component (the C# "proxy") is paired with an `UdonBehaviour` component (the "backing") on the **same GameObject**. The pairing is stored in a hidden serialized field on the proxy:

```csharp
[SerializeField, HideInInspector, UsedImplicitly]
private VRC.Udon.UdonBehaviour _udonSharpBackingUdonBehaviour;
```

`Runtime/UdonSharpBehaviour.cs:263`. **[Source]**

| Component | Visible? | Role |
|---|---|---|
| `UdonSharpBehaviour` subclass | Yes (in inspector) | Inspector / serialization surface. Holds the *editor copy* of fields. |
| `UdonBehaviour` (backing) | Hidden by default | Holds the compiled Udon Program Asset and the *runtime copy* of public variables. The VM reads/writes this. |

## Why two

The Udon VM doesn't know about C# classes. It works in terms of:
- A program asset (bytecode + variable table).
- A flat `UdonBehaviour` component whose serialized public-variable table feeds the VM.

Unity, on the other hand, wants `MonoBehaviour` subclasses with strongly-typed fields so the inspector, prefab system, animation events, and serialization all work.

UdonSharp solves both by maintaining a proxy `MonoBehaviour` and a backing `UdonBehaviour`. The proxy gets all the Unity-quality editor experience; the backing actually runs.

## Data flow

```
Inspector edit
     │
     ▼
proxy fields (the C# instance variables you wrote)
     │
     │  UdonSharpEditorUtility.CopyProxyToUdon(proxy, ProxySerializationPolicy)
     ▼
backing UdonBehaviour's public-variable table
     │
     ▼
At play / build, the Udon VM reads from the backing.
```

The reverse direction (`CopyUdonToProxy`) is used during Play mode to surface the running variable state in the inspector.

## When sync happens automatically

UdonSharp's import hooks and inspector code call `CopyProxyToUdon` at various points:
- On script reload (when fields change shape).
- On entering Play mode.
- During the SDK build process.

You explicitly call it from a custom inspector when you've programmatically modified fields and need the change reflected on the backing immediately:

```csharp
public override void OnInspectorGUI() {
    if (UdonSharpGUI.DrawDefaultUdonSharpBehaviourHeader(target)) return;
    MyScript t = (MyScript)target;

    // ... edit fields on t ...

    t.ApplyProxyModifications(ProxySerializationPolicy.All);
}
```

`ApplyProxyModifications` is the extension wrapper around `UdonSharpEditorUtility.CopyProxyToUdon`.

## What gets copied

Default policy (`ProxySerializationPolicy.RootOnly`): only this behaviour's direct serialized fields.

`ProxySerializationPolicy.All`: recursively walks `UdonSharpBehaviour` references and copies their data too. Use this when you edit fields that themselves reference other proxies whose state must also be flushed.

`ProxySerializationPolicy.NoSerialization`: skip the copy entirely.

## Lifecycle utility methods

```csharp
UdonSharpEditorUtility.GetBackingUdonBehaviour(proxy);
UdonSharpEditorUtility.GetProxyBehaviour(udonBehaviour);
UdonSharpEditorUtility.IsProxyBehaviour(proxy);
UdonSharpEditorUtility.IsUdonSharpBehaviour(udonBehaviour);
UdonSharpEditorUtility.GetUdonSharpBehaviourType(udonBehaviour);
UdonSharpEditorUtility.CreateBehaviourForProxy(proxy);
UdonSharpEditorUtility.DestroyImmediate(proxy);    // destroys both proxy and backing
```

`DestroyImmediate` is the safe way to remove a `UdonSharpBehaviour` — direct `Object.DestroyImmediate(proxy)` would orphan the backing.

## Gotchas

- **Mutating the proxy in code without calling `ApplyProxyModifications` leaves the backing stale.** Especially in custom inspectors that bypass `SerializedProperty`.
- **Two-way sync isn't free.** Custom inspectors should batch field edits, then call `ApplyProxyModifications` once.
- **At runtime there is no proxy.** The proxy exists for editor purposes. Don't write runtime code that relies on it.

## See also

- [Editor Utility](editor-utility.md)
- [Program Asset](program-asset.md)
- [GUI Helpers](gui-helpers.md)
