# UdonSharpEditorUtility

Static class `UdonSharpEditor.UdonSharpEditorUtility`. The main editor-side surface for proxy ↔ backing lookups and data sync. Source: `Editor/UdonSharpEditorUtility.cs`. **[Source]**

## Lookup

```csharp
// Get the backing UdonBehaviour for a proxy
public static UdonBehaviour GetBackingUdonBehaviour(UdonSharpBehaviour behaviour);

// Get the proxy for a backing UdonBehaviour (null if not U#-sourced)
public static UdonSharpBehaviour GetProxyBehaviour(UdonBehaviour udonBehaviour);

// Predicates
public static bool IsProxyBehaviour(UdonSharpBehaviour behaviour);
public static bool IsUdonSharpBehaviour(UdonBehaviour udonBehaviour);

// Get the C# Type a backing UdonBehaviour represents
public static Type GetUdonSharpBehaviourType(UdonBehaviour udonBehaviour);
```

## Program asset lookup (three overloads)

```csharp
public static UdonSharpProgramAsset GetUdonSharpProgramAsset(UdonSharpBehaviour behaviour);
public static UdonSharpProgramAsset GetUdonSharpProgramAsset(Type type);
public static UdonSharpProgramAsset GetUdonSharpProgramAsset(UdonBehaviour udonBehaviour);
```

Each returns the `UdonSharpProgramAsset` that compiled the script. Use whichever overload matches what you have a reference to.

## Construction

```csharp
public static UdonBehaviour CreateBehaviourForProxy(UdonSharpBehaviour proxy);
```

Creates and wires the backing `UdonBehaviour` if missing, copies the proxy's state to it, and returns the backing.

Use `UdonSharpUndo.AddComponent<T>(gameObject)` to create a proxy+backing pair with Undo support — see [Undo](undo.md).

## Destruction

```csharp
public static void DestroyImmediate(UdonSharpBehaviour behaviour);
```

Destroys both the proxy and its backing. Calls Unity's `DestroyImmediate` internally. **Use this** rather than the raw Unity equivalent — direct destruction leaves dangling backing components.

For Undo-tracked destruction: `UdonSharpUndo.DestroyImmediate`.

## Data sync between proxy and backing

```csharp
public static void CopyProxyToUdon(UdonSharpBehaviour proxy);
public static void CopyProxyToUdon(UdonSharpBehaviour proxy, ProxySerializationPolicy policy);

public static void CopyUdonToProxy(UdonSharpBehaviour proxy);
public static void CopyUdonToProxy(UdonSharpBehaviour proxy, ProxySerializationPolicy policy);
```

`CopyProxyToUdon` flushes the proxy's serialized values into the backing's public-variable table — used after inspector edits.

`CopyUdonToProxy` reads from the backing into the proxy — used during Play mode to surface live runtime state.

Both can throw if the program asset is not currently compiled or its version doesn't match the proxy. Always verify program asset state first if you're calling these from a script that touches many behaviours at once.

## `ProxySerializationPolicy`

Defined in `Editor/Serialization/ProxySerializationPolicy.cs`:

| Constant | Behaviour |
|---|---|
| `ProxySerializationPolicy.Default` | Equivalent to `RootOnly`. |
| `ProxySerializationPolicy.RootOnly` | Copy direct fields only. |
| `ProxySerializationPolicy.All` | Recursively walk and copy referenced `UdonSharpBehaviour`s. |
| `ProxySerializationPolicy.NoSerialization` | Skip copy entirely. |

Properties:
- `int MaxSerializationDepth { get; }` — recursion depth.
- `bool CollectDependencies { get; }` — internal flag (mostly used by build pipeline).
- `bool IsPreBuildSerialize { get; }` — internal flag (mostly used by build pipeline).

## Patterns

### Replace a backing program asset

```csharp
public void Reassign(UdonSharpBehaviour proxy, UdonSharpProgramAsset newAsset) {
    var backing = UdonSharpEditorUtility.GetBackingUdonBehaviour(proxy);
    backing.programSource = newAsset;
    UdonSharpEditorUtility.CopyProxyToUdon(proxy, ProxySerializationPolicy.All);
}
```

### Iterate every U#-sourced behaviour in the scene

```csharp
foreach (var ub in GameObject.FindObjectsOfType<UdonBehaviour>()) {
    if (!UdonSharpEditorUtility.IsUdonSharpBehaviour(ub)) continue;
    var proxy = UdonSharpEditorUtility.GetProxyBehaviour(ub);
    // ... do something with proxy ...
}
```

## See also

- [Proxy ↔ Backing](proxy-backing.md)
- [Program Asset](program-asset.md)
- [Undo](undo.md)
