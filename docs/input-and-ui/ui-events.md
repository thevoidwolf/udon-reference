# UI Events

Wire Unity UI components (`Button`, `Slider`, `Toggle`, etc.) to U# methods — or directly to a whitelisted set of component methods/properties for simple interactions. Source: <https://creators.vrchat.com/worlds/udon/ui-events/>. **[Docs]**

## Setup

1. Add a Canvas + Button in your scene as usual.
2. On the Button's `OnClick ()` event, click `+` to add a handler.
3. Drag a GameObject that has an `UdonBehaviour` (or its proxy) into the object slot.
4. Choose `UdonBehaviour → SendCustomEvent (string)`.
5. Type the name of the public no-arg method to call.

The UdonBehaviour will receive a `SendCustomEvent("YourMethodName")` when the button is clicked.

## Method requirements

- `public`
- **Zero parameters**
- Return value is discarded

```csharp
public void OnPlayButton() {
    audioSource.Play();
}
```

## Property events

Components with `onValueChanged` (Slider, Toggle, Dropdown, InputField, ScrollRect) work the same way. They expose a "string callback" version that bypasses the dynamic-typed parameter — pick that one and pass the U# method name.

For passing the value into your method, read it back from the component yourself:

```csharp
public Slider volumeSlider;

public void OnVolumeChanged() {
    audioSource.volume = volumeSlider.value;
}
```

This is more verbose than wiring `slider.onValueChanged.Float` to a parameterized method, but VRChat's UI event allowlist doesn't permit the parameterized form on UdonBehaviour.

## The UI whitelist

On `UdonBehaviour`, UI events may call only `SendCustomEvent`, `RunProgram`, and `Interact`; `SendCustomEvent` is the one you want for U# methods. To call other Unity component methods directly from the inspector — toggling a GameObject, setting an Animator trigger, etc. — targets are limited to the allowlist documented at [UI Whitelist](ui-whitelist.md). **[Docs]**

TextMeshPro components (`TMP_InputField`, `TMP_Dropdown`, `TextMeshProUGUI`) are usable from Udon code but are **not** UI-event targets; route their events through `SendCustomEvent` on an UdonBehaviour and read/write them from U#. **[Docs]** (absent from the UI-event list) `GetComponent` works on all TextMeshPro component types since SDK 3.7.5. **[Docs]** (release notes)

## Patterns

### Multi-button → single behaviour

```csharp
public void OnButton_Spawn() { /* ... */ }
public void OnButton_Reset() { /* ... */ }
public void OnButton_Quit()  { /* ... */ }
```

Wire each Button's `OnClick` to the appropriate method by name. Naming convention helps readability.

### State sync via networking

UI is local. Mutating UdonBehaviour state from a UI button affects only the local client. To make a UI-triggered action global, the local handler should `SendCustomNetworkEvent` (no ownership needed to send an event; take ownership only if you also change synced variables):

```csharp
public void OnButton_FireBell() {
    SendCustomNetworkEvent(NetworkEventTarget.All, nameof(RingBell));
}

public void RingBell() {
    audioSource.PlayOneShot(bellSound);
}
```

## See also

- [Input Events](input-events.md)
- [UI Whitelist](ui-whitelist.md)
- [UdonSharpBehaviour → API methods](../udonsharp-behaviour/api-methods.md)
