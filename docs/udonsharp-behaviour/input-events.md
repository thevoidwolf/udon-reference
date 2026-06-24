# Input Events

Override these on an `UdonSharpBehaviour` to receive raw input from the local player. They are dispatched only on the **local** client — there's no networking attached. **[Docs]**

Source for the C# signatures: `Runtime/UdonSharpBehaviour.cs`. The bindings table is from <https://creators.vrchat.com/worlds/udon/input-events/>.

> **Menu gotcha:** Udon cannot detect input while a VRChat menu is open (main menu, quick menu, text input). Opening a menu releases held inputs; closing it triggers held *Unity* inputs but **not** Udon button events. **[Docs]**

---

## Button events

All have signature `void Input<Name>(bool value, UdonInputEventArgs args)`. `value` is `true` on press, `false` on release.

| Event | Desktop | VR | Notes |
|---|---|---|---|
| `InputJump` | Spacebar | Face button | |
| `InputUse` | Left-click | Trigger | Same trigger that fires `Interact()` |
| `InputGrab` | (n/a desktop) | Grip | |
| `InputDrop` | Right-click | Grip release | |

```csharp
public override void InputJump(bool value, UdonInputEventArgs args) {
    if (!value) return;             // only react to press, not release
    Debug.Log("Jumped");
}
```

---

## Axis events

Float events fire (potentially) every frame the axis is non-zero. Values are clamped to `[-1, 1]`.

| Event | Desktop | VR | Notes |
|---|---|---|---|
| `InputMoveHorizontal` | A/D | Left stick X | Strafing |
| `InputMoveVertical` | W/S | Left stick Y | Forward/back |
| `InputLookHorizontal` | Mouse X | Right stick X | Look/turn |
| `InputLookVertical` | Mouse Y | Right stick Y | Look |

```csharp
public override void InputLookHorizontal(float value, UdonInputEventArgs args) {
    if (Mathf.Abs(value) < 0.1f) return;  // ignore stick deadzone yourself
    transform.Rotate(0, value * Time.deltaTime * 90f, 0);
}
```

The documentation does not formally specify whether axis events fire every frame or only on value change; community-known behaviour is **every frame** while the axis is non-default. **[Community-known]**

---

## `UdonInputEventArgs`

`VRC.Udon.Common.UdonInputEventArgs` — passed to every input event.

| Member | Type | Meaning |
|---|---|---|
| `eventType` | `UdonInputEventType` enum | `BUTTON` or `AXIS` |
| `boolValue` | `bool` | The button state (button events only; `false` for axis events) |
| `floatValue` | `float` | The axis value (axis events only; `0f` for button events) |
| `handType` | `HandType` enum | `LEFT` or `RIGHT` — the controller hand for VR input; ignored for desktop/keyboard input |

**[Docs]**

---

## `OnInputMethodChanged`

```csharp
public virtual void OnInputMethodChanged(VRCInputMethod method)
```

Fires when the local player switches input source (e.g. picks up a controller after using a keyboard).

`VRCInputMethod` enum values (from official docs): `Keyboard`, `Mouse`, `Controller`, `Vive`, `ViveXr`, `Touchscreen`. **[Docs]**

```csharp
public override void OnInputMethodChanged(VRCInputMethod method) {
    Debug.Log("Now using " + method);
}
```

---

## Patterns

### Detect press vs hold

```csharp
bool _holding;
public override void InputUse(bool value, UdonInputEventArgs args) {
    _holding = value;
}
void Update() {
    if (_holding) /* per-frame work while held */;
}
```

### Use deltaTime correctly for axis events

The axis event fires more often than `Update`, but the actual physics step still uses `Time.deltaTime` on the receiving frame. Do not multiply axis values by `Time.deltaTime` inside the event itself; cache the value and consume it in `Update`/`LateUpdate`:

```csharp
float _moveH;
public override void InputMoveHorizontal(float value, UdonInputEventArgs args) {
    _moveH = value;
}
void Update() {
    transform.position += Vector3.right * _moveH * Time.deltaTime;
}
```

### Per-hand reaction in VR

```csharp
public override void InputGrab(bool value, UdonInputEventArgs args) {
    if (!value) return;
    if (args.handType == HandType.LEFT)  DoLeft();
    else                                  DoRight();
}
```

## Reference: Phasedragon's Input Table

The community-canonical reference for VRChat controller bindings — which `Input.GetButton` / `GetAxis` keys map to which controller actions, per device — is **Phasedragon's Input Table** (Google Sheet) cited in the official UdonSharp community-resources page. Indispensable when implementing custom controller logic that goes beyond the `Input*` events. **[Community]** <https://udonsharp.docs.vrchat.com/community-resources/>

## Custom gesture detection

There is no first-party API for detecting controller gestures. The community pattern: read controller velocity via `Networking.LocalPlayer.GetTrackingData(RightHand)` over a sliding window of recent samples; threshold on speed/direction to detect flick, throw, punch. Only works on the **local** player (remote tracking is not synced at high fidelity). **[Community]**

## See also

- [Interaction events](interaction-events.md) — `Interact()` is its own event, fired only when aimed at this GameObject.
- [Players → tracking data](../players/tracking-data.md) — for reading hand position/rotation in VR.
- [Input & UI](../input-and-ui/README.md) — for UI-driven button input.
