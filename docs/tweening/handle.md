# VRCTweenHandle, Settings & Easing

Every creation method returns a `VRCTweenHandle`. You use it to control the tween (play, pause, kill, seek), query its state, and configure it with chainable setters. **[Docs]**

Source: <https://creators.vrchat.com/worlds/udon/vrctween/settings>. **[Docs]**

## Control methods

| Method | Effect |
|---|---|
| `Play()` | Resume a paused tween. |
| `Pause()` | Pause. Returns the handle, so you can chain it. |
| `Kill()` | Stop and remove the tween. |
| `Complete()` | Jump to the end values immediately. |
| `Restart()` | Restart from the beginning. |
| `Flip()` | Reverse the direction. **Doesn't start playback** if the tween is paused. |
| `PlayBackwards()` | Set the direction to backwards **and** play. |
| `PlayForwards()` | Set the direction to forwards **and** play. |
| `Goto(float to, bool andPlay)` | Seek to `to` seconds. Plays afterwards if `andPlay` is true, otherwise pauses. NaN/Infinity are ignored and other values are clamped to the duration. |

| Static / extension | Effect |
|---|---|
| `VRCTween.KillAll()` | Kill every active tween. |
| `gameObject.KillAllTweens()` | Kill every tween on that GameObject. |

All **[Docs]**.

## Query properties

| Property | Type | Meaning |
|---|---|---|
| `IsValid` | `bool` | The handle refers to a tween. `false` for `default` handles and failed creations. |
| `IsPlaying` | `bool` | The tween is playing right now. |
| `IsActive` | `bool` | The tween exists and hasn't been killed. |
| `Elapsed` | `float` | Elapsed seconds. |
| `Duration` | `float` | Total seconds. |
| `IsBackwards` | `bool` | The tween is playing backwards. |

All **[Docs]**.

## Config methods

All return the handle. **Apply `OnComplete`, `SetLoops`, `SetDelay`, `SetUpdate`, `From` and `SetSpeedBased` immediately after creation.** Anything applied after the tween starts playing (after the current frame) may be ignored. You can call `SetDuration`, `SetEase` and `ChangeEndValue` on a tween that is already running. **[Docs]**

| Method | Notes |
|---|---|
| `OnComplete(callback, eventName)` | `callback` is the behaviour (`this`). The event must be **public**. |
| `OnRewind(callback, eventName)` | Fires when backwards playback reaches position 0. It **doesn't fire on `Restart()`**. |
| `SetLoops(loops, loopType)` | `-1` means infinite. `VRCTweenLoopType`. |
| `SetDelay(delay)` | Seconds. Negative, NaN or Infinity is ignored. |
| `SetUpdate(updateType)` | `VRCTweenUpdateType`. |
| `From()` | Animate *from* the given value *to* the current value, for "appear into place". **Not available for virtual tweens.** |
| `SetSpeedBased()` | `duration` becomes units per second. Must be called immediately. |
| `SetEase(easeType)` / `SetEase(curve)` | A `VRCTweenEase` preset or a Unity `AnimationCurve`. The curve's time axis (0–1) is progress and its value axis is the interpolation factor. |
| `SetDuration(duration)` | Seconds. Zero is allowed. Negative, NaN or Infinity is ignored. |
| `ChangeEndValue(newEndValue, snapStartValue)` | The new target must match the tween's type: `Vector3` for transform tweens, `float` for fades, `Color` for colour tweens, `Vector2` for RectTransform tweens. With `snapStartValue = true`, the tween starts from the current value. Unsafe values are ignored. **Not supported** for virtual Color/Vector3, renderer, light or path tweens. |

All **[Docs]**. The docs list these with parameter names only (no types beyond those described above).

## Tween reuse

For tweens you redirect often (UI following a target, input-driven motion), create the tween once and then reconfigure it. VRChat's benchmark found this used 46× less memory and ran 10× faster than killing and recreating. For one-shot animations, plain create calls are simpler and fine. **[Docs]**

```csharp
VRCTweenHandle _moveHandle;

void Start()
{
    // Create the tween once with infinite loops so it stays alive after completing.
    _moveHandle = gameObject.TweenPosition(Vector3.zero, 1f, VRCTweenEase.OutQuad)
        .SetLoops(-1, VRCTweenLoopType.Restart)
        .Pause();
}

public void MoveTo(Vector3 target, float duration)
{
    // Reconfigure and restart without allocating.
    _moveHandle.ChangeEndValue(target, true)
        .SetDuration(duration)
        .SetEase(VRCTweenEase.OutCubic);
    _moveHandle.Restart();
}
```

The example above is verbatim. **[Docs]**

**Paused, zero-duration tweens:** since 3.10.4-beta.3, "you can now create paused, zero-duration tweens for reuse". Creation accepts a duration of `0` as a documented exception "for creating reusable tweens". That means you can create the handle with `0f` and `.Pause()`, then give it a real duration later with `SetDuration`. **[Docs]** The docs' own reuse example still uses `1f` plus infinite loops. They don't explain how a zero-duration tween behaves without loops after `Restart()`, so test that combination. **[Unverified]**

## Direction control

`PlayForwards()` / `PlayBackwards()` reverse the tween mid-animation and keep the easing curve, so there's no restart pop. A tween that plays backwards to the start is **not** killed automatically. Kill it yourself, for example in `OnDestroy`. **[Docs]**

```csharp
_fadeHandle = audioSource.TweenVolume(1f, 2f, VRCTweenEase.InSine)
    .OnRewind(this, nameof(OnFadeOut))
    .Pause();
// OnZoneEnter → _fadeHandle.PlayForwards();   OnZoneExit → _fadeHandle.PlayBackwards();
```

## Seeking with Goto

Tweens don't sync. For long-running shared motion, sync a start time and seek on each client. **[Docs]**

```csharp
[UdonSynced] float _tweenStartTime;
VRCTweenHandle _syncedTween;

public override void OnDeserialization()
{
    float elapsed = (float)(Networking.GetServerTimeInSeconds() - _tweenStartTime);
    _syncedTween.Goto(elapsed, true);
}
```

The example above is abridged from the docs. **[Docs]**

## Enums

| Enum | Values |
|---|---|
| `VRCTweenLoopType` | `Restart` (0), `Yoyo` (1), `Incremental` (2, which adds each loop to the previous end value). **[Docs]** |
| `VRCTweenUpdateType` | `Update` (0), `LateUpdate` (1), `FixedUpdate` (2), `PostLateUpdate` (3, after all LateUpdate calls, for overriding camera/IK-driven transforms). **[Docs]** |
| `VRCTweenPathType` | `Linear`, `CatmullRom`. **[Docs]** |

### VRCTweenEase (31 values)

`Linear` · `InSine` `OutSine` `InOutSine` · `InQuad` `OutQuad` `InOutQuad` · `InCubic` `OutCubic` `InOutCubic` · `InQuart` `OutQuart` `InOutQuart` · `InQuint` `OutQuint` `InOutQuint` · `InExpo` `OutExpo` `InOutExpo` · `InCirc` `OutCirc` `InOutCirc` · `InElastic` `OutElastic` `InOutElastic` · `InBack` `OutBack` `InOutBack` · `InBounce` `OutBounce` `InOutBounce` **[Docs]**

VRChat suggests `OutQuad` for UI, `OutElastic` for a playful bounce and `InOutSine` for natural motion. You can preview the curves at <https://easings.net/>. **[Docs]**

## See also

- [Tweening overview](README.md)
- [Tween types](tween-types.md)
- [Virtual tweens & timers](virtual-tweens.md)
- [Networking → Serialization](../networking/serialization.md): syncing the start time for `Goto`.
- [Players → Networking class](../players/networking-class.md): `Networking.GetServerTimeInSeconds()`.
- [UdonSharpBehaviour → Lifecycle events](../udonsharp-behaviour/lifecycle-events.md): `OnDestroy` cleanup.
