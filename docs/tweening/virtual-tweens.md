# Virtual Tweens & Timers

A virtual tween animates a value instead of a component. Each frame it writes the interpolated value into a **public field** on your behaviour and then calls your callback, which applies the value however you like. VRChat suggests them for camera FOV, score counters and animator parameters. The same page also covers the two timer helpers. **[Docs]**

Source: <https://creators.vrchat.com/worlds/udon/vrctween/virtual-tweens>. **[Docs]**

## Methods

All are static on `VRCTween` and return a `VRCTweenHandle`. **[Docs]**

| Method | Documented example |
|---|---|
| `TweenFloat` | `VRCTween.TweenFloat(0f, 100f, 2f, this, nameof(myValue), nameof(OnUpdate), VRCTweenEase.Linear)` |
| `TweenInt` | `VRCTween.TweenInt(0, 100, 5f, this, nameof(myValue), nameof(OnUpdate), VRCTweenEase.Linear)` |
| `TweenColor` | `VRCTween.TweenColor(Color.red, Color.blue, 2f, this, nameof(myValue), nameof(OnUpdate), VRCTweenEase.Linear)` |
| `TweenVector3` | `VRCTween.TweenVector3(Vector3.zero, Vector3.one, 2f, this, nameof(myValue), nameof(OnUpdate), VRCTweenEase.Linear)` |
| `DelayedCall` | `VRCTween.DelayedCall(this, nameof(OnTimer), 5f)` |
| `DelayedSetActive` | `VRCTween.DelayedSetActive(myObject, false, 2f)` |

The examples are verbatim. The argument order is **from, to, duration, behaviour, variable name, callback name, ease**. **[Docs]**

## Field and callback rules

- The target variable **must be `public`**. Mark it `[System.NonSerialized]` so the transient value isn't saved into the scene. **[Docs]**
- Use correctly typed fields (`float`, `int`, `Color`, `Vector3`). To run several tweens at once, give each its own field. **[Docs]**
- The callback is called every frame and takes no parameters. Read the field inside it. **[Docs]**
- `TweenVector3` applies easing to the whole vector, not to each axis separately. **[Docs]**
- `From()` doesn't work on virtual tweens. **[Docs]**
- `ChangeEndValue` works on virtual **float/int** tweens but **not** on virtual Color or Vector3 tweens. **[Docs]**

```csharp
[System.NonSerialized] public float fovValue;

VRCTweenHandle tweenHandle = VRCTween.TweenFloat(60f, 90f, 2f, this, nameof(fovValue), nameof(OnFovUpdate), VRCTweenEase.OutQuad);

public void OnFovUpdate()
{
    myCamera.fieldOfView = fovValue;  // smoothly widens the camera's field of view
}
```

The example above is verbatim. **[Docs]**

## DelayedCall

A cancelable delayed event. It's VRChat's suggested alternative to `SendCustomEventDelayedSeconds` when you need to cancel. **[Docs]**

```csharp
VRCTweenHandle timerHandle = VRCTween.DelayedCall(this, nameof(OnTimerFinished), 5.0f);

// Cancel the timer anytime.
timerHandle.Kill();

public void OnTimerFinished()
{
    Debug.Log("5 seconds elapsed!");
}
```

The example above is verbatim. The callback must be public. **[Docs]** For a side-by-side comparison, see the [overview](README.md#delayedcall-vs-sendcustomeventdelayedseconds).

### Pattern: restartable timeout

```csharp
private VRCTweenHandle _idle;

public void Poke()                      // call on any activity
{
    _idle.Kill();
    _idle = VRCTween.DelayedCall(this, nameof(OnIdle), 30f);
}

public void OnIdle() { /* reset the puzzle, dim the lights, ... */ }
```

This is built only from documented calls. With `SendCustomEventDelayedSeconds`, you would need a counter to ignore stale callbacks. **[Docs]** (calls) / pattern **[Community-known]**

## DelayedSetActive

Added in 3.10.4-beta.3. It sets a `GameObject` active or inactive after a delay, and is shorthand for a `DelayedCall` whose callback only calls `SetActive`. If the target is destroyed before the delay ends, the action is skipped silently. **[Docs]**

```csharp
// Disable an object after 3 seconds.
VRCTweenHandle handle = VRCTween.DelayedSetActive(myObject, false, 3f);

// Cancel it if needed.
handle.Kill();
```

The example above is verbatim. **[Docs]**

## See also

- [Tweening overview](README.md)
- [Tween types](tween-types.md)
- [Handle, settings & easing](handle.md)
- [UdonSharpBehaviour → API methods](../udonsharp-behaviour/api-methods.md): `SendCustomEventDelayedSeconds` / `Frames`.
- [Compiler → Workarounds](../compiler/workarounds.md)
