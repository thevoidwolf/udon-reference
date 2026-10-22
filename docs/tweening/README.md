# Tweening (VRCTween)

`VRCTween` is a built-in tweening system that exposes "select functionality from the powerful DOTween library to Udon". It shipped in **Worlds SDK 3.10.4** (Jun 17, 2026) and works in both UdonSharp and Udon Graph. **[Docs]**

Source: <https://creators.vrchat.com/worlds/udon/vrctween/> · release notes <https://creators.vrchat.com/releases/release-3-10-4>. **[Docs]**

## Pages

| Page | Covers |
|---|---|
| [Tween types](tween-types.md) | Built-in tweens for Transform, Path, UI, Sprite, Renderer, Light and Audio, including `TweenPitch`. |
| [Virtual tweens & timers](virtual-tweens.md) | `TweenFloat` / `TweenInt` / `TweenColor` / `TweenVector3`, `DelayedCall`, `DelayedSetActive`. |
| [Handle, settings & easing](handle.md) | `VRCTweenHandle` control, query and config methods, reuse, `Goto`, all enums. |

## Shape of the API

- **Creation:** extension methods on the target (`cube.TweenPosition(...)`) or static methods on `VRCTween`. Every creation method returns a `VRCTweenHandle`. **[Docs]** The docs only give signatures for the *static* forms of the virtual tweens and timers. They don't show static equivalents of the extension methods. **[Unverified]**
- **Configuration and control:** instance methods on `VRCTweenHandle`. Config methods return the handle, so you can chain them. **[Docs]**
- **Callbacks:** `.OnComplete(this, nameof(Method))` / `.OnRewind(...)`. The target method **must be public**. **[Docs]**
- **Namespace:** the official examples import `using VRC.SDK3.Components;`. **[Docs]**

```csharp
cube.TweenPosition(new Vector3(0, 5, 0), 2f, VRCTweenEase.OutQuad)
    .From()
    .SetDelay(0.5f)
    .SetLoops(2, VRCTweenLoopType.Yoyo)
    .OnComplete(this, nameof(OnDone));
```

The snippet above is verbatim from the docs. **[Docs]**

## Example (U#)

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDK3.Components;

public class PopButton : UdonSharpBehaviour
{
    public GameObject button;
    public GameObject confirmPanel;
    private VRCTweenHandle _scale;
    private VRCTweenHandle _autoHide;

    public override void Interact()
    {
        _scale.Kill();                                   // a default handle is a no-op target
        _scale = button.TweenScale(Vector3.one * 1.2f, 0.15f, VRCTweenEase.OutQuad)
            .SetLoops(2, VRCTweenLoopType.Yoyo);

        confirmPanel.SetActive(true);
        _autoHide.Kill();                                // cancel the previous timer
        _autoHide = VRCTween.DelayedSetActive(confirmPanel, false, 3f);
    }

    void OnDestroy()
    {
        button.KillAllTweens();
        _autoHide.Kill();
    }
}
```

Every call above is documented. **[Docs]** The docs don't say what calling `Kill()` on a `default` (invalid) handle does. They do say `IsValid` is false for default handles, and that invalid input generally fails silently rather than throwing. Guard with `if (_scale.IsValid)` if you want to be strict. **[Unverified]**

## DelayedCall vs `SendCustomEventDelayedSeconds`

| | `VRCTween.DelayedCall(this, nameof(X), 3f)` | `SendCustomEventDelayedSeconds(nameof(X), 3f)` |
|---|---|---|
| Returns | `VRCTweenHandle` | `void` |
| Cancel | `handle.Kill()` **[Docs]** | Not possible. The usual workaround is a guard flag or counter. **[Community-known]** |
| Frame phase | Not documented | `EventTiming` parameter. See [API methods](../udonsharp-behaviour/api-methods.md). |
| Availability | SDK 3.10.4+ | All SDK3 versions |

VRChat describes `DelayedCall` as working "like `SendCustomEventDelayedSeconds`, but returns a handle you can `Kill()` to cancel". **[Docs]** `DelayedSetActive(go, bool, seconds)` is the documented shorthand for a `DelayedCall` whose callback only calls `SetActive`. If the target is destroyed first, the call is skipped silently. **[Docs]**

## Limitations

- **Local only.** Tweens don't sync. For networked motion, sync a start time and seek with `Goto` (see [Handle → Seeking](handle.md#seeking-with-goto)). **[Docs]**
- Durations below **0.01 s** may not animate smoothly. **[Docs]**
- Tween handles are unique per scene instance. **[Docs]**
- **No exceptions on bad input.** A null target, or a duration that is negative, NaN or Infinity, returns an *invalid handle* and creates no tween. Check `handle.IsValid`. **[Docs]**
- Position, scale and path values with NaN, Infinity or magnitudes above roughly **520,000** units are rejected. Rotation only needs finite values. **[Docs]**
- DOTween's `RotateMode` and `PathMode` aren't exposed. There's a community feedback request, and virtual tweens are the workaround. **[Community]** <https://feedback.vrchat.com/udon/p/export-pathmode-and-rotatemode-for-vrctween>

## Performance

- Up to about 50 interaction-triggered tweens, simply creating new tweens is fine, because DOTween pools internally. **[Docs]**
- Don't start hundreds of tweens in one frame. Stagger them, for example with `SendCustomEventDelayedSeconds`. **[Docs]**
- For tweens that are redirected constantly, reuse one handle with `ChangeEndValue` + `SetDuration` + `SetEase` + `Restart()`. VRChat measured **46× less memory and 10× faster** than killing and recreating, with 500 tweens over 300 frames. **[Docs]** See [Handle → Tween reuse](handle.md#tween-reuse).
- Clean up infinite or long tweens in `OnDestroy` with `gameObject.KillAllTweens()` or `VRCTween.KillAll()`. Finished tweens clean themselves up. **[Docs]**

## Udon Graph

Search for the `VRCTween` and `VRCTweenHandle` nodes (for example "VRCTween TweenPosition"). For callbacks, use the `VRCTweenHandle OnComplete` node with the behaviour (usually `this`) and a custom event name. **[Docs]**

## See also

- [UdonSharpBehaviour → API methods](../udonsharp-behaviour/api-methods.md): `SendCustomEventDelayedSeconds` / `Frames`.
- [UdonSharpBehaviour](../udonsharp-behaviour/README.md)
- [Networking](../networking/README.md): tweens are local, so sync a start time.
- [Compiler → Workarounds](../compiler/workarounds.md): "run this after N seconds" patterns.
- [Graphics](../graphics/README.md): material and `MaterialPropertyBlock` context for renderer tweens.
- [Input & UI → UI events](../input-and-ui/ui-events.md)
- [Dynamics](../dynamics/README.md): another 3.10.x feature.
