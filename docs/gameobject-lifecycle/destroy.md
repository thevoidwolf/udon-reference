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

Use `Destroy` exclusively in U# code paths that run in the VRChat client. Whether `Object.DestroyImmediate` is exposed to Udon at all is not documented — check the SDK's Class Exposure Tree (VRChat SDK → Udon Sharp → Class Exposure Tree) for your version. **[Unverified]** In *editor* scripts, use `UdonSharpEditorUtility.DestroyImmediate()` to remove a U# behaviour together with its backing UdonBehaviour. **[Docs]**

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

### Disable instead of destroy

Often you only need the component to stop running. Since **SDK 3.10.2**, `enabled` and `isActiveAndEnabled` are allowlisted on everything deriving from `UnityEngine.Behaviour` (most built-in and custom components), so `someBehaviour.enabled = false` is reversible where `Destroy` is not. Known limitation: it doesn't work through a field typed as plain `Behaviour` that points at an `UdonSharpBehaviour` — type the field as your U# class instead. **[Docs]** (release notes, U# docs "Differences from Unity/C#")

## Patterns

### Auto-cleanup

```csharp
void Start() {
    Destroy(gameObject, 5f);     // disappear in 5 s no matter what
}
```

### Cancelable timed disable (SDK 3.10.4+)

`Destroy(obj, t)` can't be cancelled. If you just want an object gone after a delay *unless* something happens, `VRCTween.DelayedSetActive(obj, false, t)` returns a `VRCTweenHandle` you can `Kill()`; if the target is destroyed first, the action is silently skipped. **[Docs]** <https://creators.vrchat.com/worlds/udon/vrctween/virtual-tweens/>

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

- **Don't rely on `OnDestroy` for cleanup.** Release resources explicitly before calling `Destroy` — e.g. `VRCImageDownloader.Dispose()`, or `IVRCImageDownload.Dispose()` for a single texture (allowlisted SDK 3.8.0). Undisposed downloads leak memory for the world's lifetime. **[Docs]** (Whether `OnDestroy` fires on U# behaviours at all is **[Unverified]**.)
- A destroyed GameObject still exists for the current frame's late phases. References captured before `Destroy` and used later in the same frame may behave unpredictably.
- Destroying authored *synced* scene objects locally desyncs them from other clients (they still exist elsewhere). Don't do it if you have synced state that depends on cross-client identity. **[Community-known]**

## See also

- [Instantiate](instantiate.md)
- [Runtime constraints](runtime-constraints.md)
- [VRCObjectPool](../networking/object-pool.md) — preferred over Destroy for networked spawns.
