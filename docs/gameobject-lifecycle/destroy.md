# Destroy

Removing GameObjects and components at runtime.

## Supported calls

```csharp
Destroy(gameObject);                                  // destroy at end of frame
Destroy(gameObject, 3.0f);                            // delayed by 3 seconds
Destroy(GetComponent<AudioSource>());                 // remove a component
```

`UnityEngine.Object.Destroy` works for any `GameObject` or `Component` you have a reference to.

## `DestroyImmediate` is editor-only

Udon does **not** expose `DestroyImmediate` at runtime. Use `Destroy` exclusively in U# code paths that run in the VRChat client.

## What can you destroy

| Target | Allowed? |
|---|---|
| GameObjects you spawned with `Instantiate` | Yes |
| GameObjects authored in the scene | Generally yes, but be careful — they cannot be re-created |
| Components on a GameObject | Yes |
| Yourself (the calling `UdonSharpBehaviour`) | Yes — but the calling code won't finish executing safely. Destroy then `return`. |

## Component-level destruction

You can remove most components, including the UdonBehaviour you're calling from:

```csharp
Destroy(GetComponent<Rigidbody>());
Destroy(this);                  // destroys this UdonBehaviour but leaves the GameObject
```

Removing the only `UdonBehaviour` on a GameObject is valid. The GameObject itself stays alive.

## Patterns

### Auto-cleanup

```csharp
void Start() {
    Destroy(gameObject, 5f);     // disappear in 5 s no matter what
}
```

### Pool members never destroy

If a pooled GameObject self-destructs, the pool loses a slot forever. Always `Return` to the pool instead:

```csharp
// On a pool member:
void Cleanup() {
    pool.Return(gameObject);   // not Destroy(gameObject)
}
```

### Defensive null check

After scheduling `Destroy`, the object is still alive *for the rest of the current frame*. The reference becomes `null`-equivalent on the next frame. Always check before dereferencing:

```csharp
if (Utilities.IsValid(target)) target.SomeMethod();
```

(For Unity objects, `target == null` is the standard check; the override that compares against a destroyed object correctly returns true.)

## Gotchas

- **`Destroy` does not fire `OnDestroy` on Udon programs in time for cleanup**. If you need to release a resource (e.g. `VRCImageDownloader.Dispose()`), do it explicitly before calling `Destroy`.
- A destroyed GameObject still exists for the current frame's late phases. References captured before `Destroy` and used later in the same frame may behave unpredictably.
- Destroying authored scene objects breaks deterministic Network ID assignment. Don't do it if you have synced state that depends on cross-client identity.

## See also

- [Instantiate](instantiate.md)
- [Runtime constraints](runtime-constraints.md)
- [VRCObjectPool](../networking/object-pool.md) — preferred over Destroy for networked spawns.
