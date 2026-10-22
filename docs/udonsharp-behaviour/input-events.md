# Input Events

Override these on an `UdonSharpBehaviour` to receive raw input from the local player. They are dispatched only on the **local** client — there's no networking attached. **[Docs]**

Source for the C# signatures: `Runtime/UdonSharpBehaviour.cs` **[Source]** (except `OnInputMethodChanged`, which is only in the SDK-bundled copy). The bindings table is from <https://creators.vrchat.com/worlds/udon/input-events/>. These events respect the player's remapped controls. **[Docs]**

> **Menu gotcha:** Udon cannot detect input while a VRChat menu is open (main menu, quick menu, text input). Opening a menu releases held inputs; closing it triggers held *Unity* inputs but **not** Udon button events. **[Docs]**

---

## Button events

All have signature `void Input<Name>(bool value, UdonInputEventArgs args)`. `value` is `true` on press, `false` on release.

| Event | Desktop | VR | Notes |
|---|---|---|---|
| `InputJump` | Spacebar | Face button (typically) | |
| `InputUse` | Left-click | Trigger (typically) | Same trigger that fires `Interact()` |
| `InputGrab` | Left-click | Grip | |
| `InputDrop` | Right-click | Grip **press** on Vive wands and some WMR controllers, grip **release** on others | |

**[Docs]**

```csharp
public override void InputJump(bool value, UdonInputEventArgs args) {
    if (!value) return;             // only react to press, not release
    Debug.Log("Jumped");
}
```

---

## Axis events

Values typically range over `[-1, 1]`. **[Docs]**

| Event | Desktop | VR | Notes |
|---|---|---|---|
| `InputMoveHorizontal` | A/D | Left stick X | Strafing |
| `InputMoveVertical` | W/S | Left stick Y | Forward/back |
| `InputLookHorizontal` | Mouse X | Right stick X | Turn (VR: only without Comfort Turning) |
| `InputLookVertical` | Mouse Y | Right stick Y | Look |

**[Docs]**

```csharp
float _lookH;
public override void InputLookHorizontal(float value, UdonInputEventArgs args) {
    _lookH = Mathf.Abs(value) < 0.1f ? 0f : value;  // apply your own deadzone
}
void Update() {
    transform.Rotate(0, _lookH * Time.deltaTime * 90f, 0);
}
```

Axis events fire **on value change**, not every frame: an analog stick produces a new event per step (0 → 0.1 → 0.2 …), and Desktop keys produce whole numbers (-1, 0, 1). **[Docs]** Holding a key or a stick still therefore produces no further events — cache the last value (see the pattern below).

---

## `UdonInputEventArgs`

`VRC.Udon.Common.UdonInputEventArgs` — passed to every input event.

| Member | Type | Meaning |
|---|---|---|
| `eventType` | `UdonInputEventType` enum | `BUTTON` or `AXIS` |
| `boolValue` | `bool` | The button state (button events only; `false` for axis events) |
| `floatValue` | `float` | The axis value (axis events only; `0f` for button events) |
| `handType` | `HandType` enum | `LEFT` or `RIGHT`. Also set for Desktop: mouse reports `RIGHT`, keyboard reports `LEFT`. |

**[Docs]**

---

## `OnInputMethodChanged`

```csharp
public override void OnInputMethodChanged(VRCInputMethod inputMethod)
```

Fires when the local player switches input method (e.g. keyboard → mouse, controller, touchscreen). **[Docs]**

`VRC.SDKBase.VRCInputMethod` values (index in parentheses): `Keyboard` (0), `Mouse` (1), `Controller` (2), `Gaze` (3), `Vive` (5), `Oculus` (6), `ViveXR` (7), `Index` (10), `HPMotionController` (11), `Osc` (12), `QuestHands` (13), `Generic` (14), `Touch` (15), `OpenXRGeneric` (16), `Pico` (17), `SteamVR2` (18), `Embodied` (19). **[Docs]** (<https://creators.vrchat.com/worlds/udon/graph/type-nodes/#vrcsdkbasevrcinputmethod>)

- Most PC VR controllers report `SteamVR2`. Switching to Quest/Pico hand tracking reports `Embodied` (finger tracking of any kind). **[Docs]**
- `Vive` = Vive controller through SteamVR; `ViveXR` = Vive XR Elite controller via OpenXR (the prose docs spell it `ViveXr`; the type table spells it `ViveXR` — check in your IDE). **[Docs]**
- Touchscreen input is `Touch`; there is no `Touchscreen` member. **[Docs]**

```csharp
public override void OnInputMethodChanged(VRCInputMethod inputMethod) {
    Debug.Log("Now using " + inputMethod);
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

Axis events only fire when the value *changes*, so a stick held steady produces no events. Do not move things inside the event itself; cache the value and consume it every frame in `Update`/`LateUpdate`:

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
- [Input & UI](../input-and-ui/README.md) — for UI-driven button input, and the allowlisted `UnityEngine.Input` members (`GetKey`, `GetAxis`, `inputString`, …).
