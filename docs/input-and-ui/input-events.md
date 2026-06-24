# Input Events

See [UdonSharpBehaviour → input events](../udonsharp-behaviour/input-events.md) for the per-event signatures and patterns. This page is a quick cross-reference of the bindings table and `VRCInputMethod`.

## Button events

| Event | Desktop | VR controller | Notes |
|---|---|---|---|
| `InputJump(bool, args)` | Spacebar | Face button | |
| `InputUse(bool, args)` | Left-click | Trigger | Same trigger that fires `Interact()` when aimed at an interactable. |
| `InputGrab(bool, args)` | (no desktop bind) | Grip | |
| `InputDrop(bool, args)` | Right-click | Grip release | |

## Axis events

Range `[-1, 1]`. Fires every frame the axis is non-zero.

| Event | Desktop | VR controller |
|---|---|---|
| `InputMoveHorizontal(float, args)` | A / D | Left stick X |
| `InputMoveVertical(float, args)` | W / S | Left stick Y |
| `InputLookHorizontal(float, args)` | Mouse X | Right stick X |
| `InputLookVertical(float, args)` | Mouse Y | Right stick Y |

## `UdonInputEventArgs`

`VRC.Udon.Common.UdonInputEventArgs`. Properties:

| Property | Type | Meaning |
|---|---|---|
| `eventType` | `UdonInputEventType` | `BUTTON` or `AXIS`. |
| `boolValue` | `bool` | Button state; `false` for axis events. |
| `floatValue` | `float` | Axis value; `0` for button events. |
| `handType` | `HandType` | `LEFT` / `RIGHT`. Meaningful for VR; ignored for keyboard/mouse. |

## `OnInputMethodChanged`

```csharp
public override void OnInputMethodChanged(VRCInputMethod method)
```

Fires when the local player switches input devices.

`VRCInputMethod` values (per docs): `Keyboard`, `Mouse`, `Controller`, `Vive`, `ViveXr`, `Touchscreen`. **[Docs]**

## Menu gotcha

Udon does not receive input events while a VRChat menu is open. Held buttons are released on menu-open. On menu-close, held *Unity* inputs trigger but **Udon button events do not refire** for buttons that were held during the menu. Design state machines defensively.

## See also

- [UdonSharpBehaviour → input events](../udonsharp-behaviour/input-events.md) — full discussion + examples.
- [UdonSharpBehaviour → interaction events](../udonsharp-behaviour/interaction-events.md) — `Interact()`, `OnPickup*`.
