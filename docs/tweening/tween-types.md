# Built-in Tween Types

Extension methods on the target component, each returning a `VRCTweenHandle`. **[Docs]**

Source: <https://creators.vrchat.com/worlds/udon/vrctween/tween-types>. **[Docs]**

> The docs show each method **through example calls**, not formal declarations. The "call shape" column below copies the argument order from those examples. Parameter *names* are given only for path tweens, so the other names here are descriptive. **[Docs]** (order) / names **[Unverified]**

## Transform

You can call these on either a `GameObject` or a `Transform`. **[Docs]**

| Method | Call shape | Notes |
|---|---|---|
| `TweenPosition` | `(Vector3 target, float duration, VRCTweenEase ease)` | World position. |
| `TweenLocalPosition` | `(Vector3 target, float duration, VRCTweenEase ease)` | Local position. |
| `TweenRotation` | `(Vector3 eulerAngles, float duration, VRCTweenEase ease)` | World Euler angles. |
| `TweenLocalRotation` | `(Vector3 eulerAngles, float duration, VRCTweenEase ease)` | Local Euler angles. |
| `TweenScale` | `(Vector3 target, float duration, VRCTweenEase ease)` | `localScale`. There's no world-scale variant, because world scale is read-only. |

```csharp
door.TweenPosition(new Vector3(0, 3, 0), 1f, VRCTweenEase.OutQuad);
lever.TweenRotation(new Vector3(-45, 0, 0), 0.5f, VRCTweenEase.OutBack);
```

## Path

Also on `GameObject` or `Transform`. `TweenPath` uses world space and `TweenLocalPath` uses local space. **[Docs]**

```csharp
platform.TweenPath(route, 8f, VRCTweenPathType.Linear, true, 10, VRCTweenEase.Linear)
    .SetLoops(-1, VRCTweenLoopType.Restart);
```

| Parameter | Notes |
|---|---|
| `waypoints` | `Vector3[]`, minimum 2. NaN, Infinity or extreme values are rejected. **[Docs]** |
| (duration) | Seconds, or units per second with `SetSpeedBased()`. **[Docs]** |
| `pathType` | `VRCTweenPathType.Linear` or `VRCTweenPathType.CatmullRom`. **[Docs]** |
| `closePath` | `true` closes the path from the last waypoint back to the first. **[Docs]** |
| `resolution` | CatmullRom smoothness. Ignored for Linear. Clamped silently to 1–50. The docs call 10 the default. **[Docs]** |
| (ease) | `VRCTweenEase`. **[Docs]** |

For a seamless patrol loop, use `closePath = true` with `.SetLoops(-1, VRCTweenLoopType.Restart)`. **[Docs]**

## UI

| Target | Method | Call shape |
|---|---|---|
| `Graphic` (Image, Text, RawImage…) | `TweenColor` | `(Color target, float duration, VRCTweenEase ease)` |
| `Graphic` | `TweenFade` | `(float alpha, float duration, VRCTweenEase ease)` |
| `CanvasGroup` | `TweenFade` | `(float alpha, float duration, VRCTweenEase ease)` |
| `Slider` | `TweenValue` | `(float value, float duration, VRCTweenEase ease)` |
| `RectTransform` | `TweenAnchorPos` | `(Vector2 target, float duration, VRCTweenEase ease)` |
| `RectTransform` | `TweenSizeDelta` | `(Vector2 target, float duration, VRCTweenEase ease)` |

All **[Docs]**. Use `TweenAnchorPos`, not `TweenPosition`, to move elements inside a Canvas layout. **[Docs]**

## Sprite

| Target | Method | Call shape |
|---|---|---|
| `SpriteRenderer` | `TweenColor` | `(Color target, float duration, VRCTweenEase ease)` |
| `SpriteRenderer` | `TweenFade` | `(float alpha, float duration, VRCTweenEase ease)` |

## Renderer

These use a **`MaterialPropertyBlock`**, so they create no material instances and stay GPU-instancing-friendly. **[Docs]**

| Method | Call shape | Notes |
|---|---|---|
| `TweenColor` | `(Color target, float duration, VRCTweenEase ease)` | Defaults to `_Color`. |
| `TweenColor` | `(string propertyName, Color target, float duration, VRCTweenEase ease)` | e.g. `"_EmissionColor"`. |
| `TweenFloat` | `(string propertyName, float target, float duration, VRCTweenEase ease)` | e.g. `"_DissolveAmount"`, `"_Metallic"`. |

If nothing visible changes, the shader probably uses a different property name. Check it in the material inspector. **[Docs]** `ChangeEndValue` is **not** supported on renderer tweens. **[Docs]**

## Light

| Method | Call shape |
|---|---|
| `TweenIntensity` | `(float target, float duration, VRCTweenEase ease)` |
| `TweenColor` | `(Color target, float duration, VRCTweenEase ease)` |

`ChangeEndValue` is not supported on light tweens. **[Docs]**

## Audio (`AudioSource`)

| Method | Call shape | Notes |
|---|---|---|
| `TweenVolume` | `(float target, float duration, VRCTweenEase ease)` | |
| `TweenPitch` | `(float target, float duration, VRCTweenEase ease)` | **Added in 3.10.4-beta.3.** Typically −3 to 3, but any finite value is accepted. |

```csharp
engineSource.TweenPitch(2f, 1f, VRCTweenEase.OutQuad);
```

All **[Docs]**.

## See also

- [Tweening overview](README.md)
- [Virtual tweens & timers](virtual-tweens.md): for values not covered here, such as camera FOV or animator parameters.
- [Handle, settings & easing](handle.md)
- [Graphics → Materials](../graphics/materials.md)
- [Input & UI → UI events](../input-and-ui/ui-events.md)
- [Players → Voice & audio](../players/voice-audio.md)
