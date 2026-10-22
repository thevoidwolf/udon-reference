# Input Events

See [UdonSharpBehaviour → input events](../udonsharp-behaviour/input-events.md) for the per-event signatures and patterns. This page is a quick cross-reference of the bindings table and `VRCInputMethod`.

Source: <https://creators.vrchat.com/worlds/udon/input-events/>. **[Docs]** These events respect the player's remapped controls.

## Button events

`bool` value is `true` on press, `false` on release. **[Docs]**

| Event | Desktop | Controller | Notes |
|---|---|---|---|
| `InputJump(bool, args)` | Spacebar | Typically a face button | |
| `InputUse(bool, args)` | Left-click | Typically the trigger | Same trigger that fires `Interact()` when aimed at an interactable. |
| `InputGrab(bool, args)` | Left-click | Typically the grip (VR) | |
| `InputDrop(bool, args)` | Right-click | Grip **press** on Vive wands and some WMR controllers; grip **release** on others | |

## Axis events

`float` value, typically `[-1, 1]`. Axis events fire **when the value changes** (an analog stick produces a new event per step, e.g. 0 → 0.1 → 0.2 …), not every frame. Desktop keys/mouse output whole numbers (-1, 0, 1). **[Docs]**

| Event | Desktop | Controller |
|---|---|---|
| `InputMoveHorizontal(float, args)` | A / D | Left stick/pad X |
| `InputMoveVertical(float, args)` | W / S | Left stick/pad Y |
| `InputLookHorizontal(float, args)` | Mouse X | Right stick X (VR: only without Comfort Turning) |
| `InputLookVertical(float, args)` | Mouse Y | Right stick Y |

## `UdonInputEventArgs`

`VRC.Udon.Common.UdonInputEventArgs`. Properties: **[Docs]**

| Property | Type | Meaning |
|---|---|---|
| `eventType` | `UdonInputEventType` | `BUTTON` or `AXIS`. |
| `boolValue` | `bool` | Button state; `false` for axis events. |
| `floatValue` | `float` | Axis value (-1..1); `0` for button events. |
| `handType` | `HandType` | `LEFT` / `RIGHT`. Also set for desktop: **mouse is `RIGHT`, keyboard is `LEFT`**. |

## `OnInputMethodChanged`

```csharp
public override void OnInputMethodChanged(VRCInputMethod inputMethod)
```

Fires when the local player switches input methods (e.g. Keyboard → Mouse, Controller, or Touch). Added SDK 3.5.2. **[Docs]**

`VRC.SDKBase.VRCInputMethod` values (17, with gaps in the numbering) **[Docs]** <https://creators.vrchat.com/worlds/udon/graph/type-nodes/#vrcsdkbasevrcinputmethod>:

| Value | Name | | Value | Name |
|---|---|---|---|---|
| 0 | `Keyboard` | | 12 | `Osc` |
| 1 | `Mouse` | | 13 | `QuestHands` |
| 2 | `Controller` | | 14 | `Generic` |
| 3 | `Gaze` | | 15 | `Touch` |
| 5 | `Vive` | | 16 | `OpenXRGeneric` |
| 6 | `Oculus` | | 17 | `Pico` |
| 7 | `ViveXR` | | 18 | `SteamVR2` |
| 10 | `Index` | | 19 | `Embodied` |
| 11 | `HPMotionController` | | | |

- Most PC VR controllers report as `SteamVR2`. Switching to Quest/Pico hand tracking (or other finger tracking) reports `Embodied`. **[Docs]**
- `Vive` = Vive controller via SteamVR; `ViveXR` = Vive XR Elite controller via OpenXR. The docs spell it `ViveXR` in the enum table and `ViveXr` in prose — check casing against your SDK's autocomplete. **[Unverified]**

## Unity `Input` API

Udon also exposes part of `UnityEngine.Input`: `Input.anyKey`, `anyKeyDown`, `inputString`, `imeIsSelected`, `GetAxis`, `GetAxisRaw`, `GetButton`/`Down`/`Up`, `GetMouseButton`/`Down`/`Up`, `GetJoystickNames`, `GetKey`/`Down`/`Up`. **[Docs]** Prefer the Udon input events for cross-platform input — the docs only guarantee remapping support for those.

## Menu gotcha

Udon does not receive input while the **Main menu**, **Quick menu** (Desktop/mobile only) or **text input popup** is open. On menu-open, all held inputs are released (`InputJump(false)` fires; `Input.GetButtonUp()` returns true). On menu-close, held *Unity* inputs are pressed again (`GetButtonDown()` true), but **Udon button events do not refire** (no `InputJump(true)`). Design state machines defensively. **[Docs]**

## See also

- [UdonSharpBehaviour → input events](../udonsharp-behaviour/input-events.md) — full discussion + examples.
- [UdonSharpBehaviour → interaction events](../udonsharp-behaviour/interaction-events.md) — `Interact()`, `OnPickup*`.
