# Input & UI

Player input (controller / keyboard / VR), Unity UI hookups, and the animation-event allowlist.

## Pages

| Page | Topic |
|---|---|
| [Input Events](input-events.md) | Raw player input on `UdonSharpBehaviour`. |
| [UI Events](ui-events.md) | Wiring Unity UI components (Button onClick, etc.) to U# methods. |
| [Animation Events](animation-events.md) | Methods callable from Unity's Animation Window. |
| [UI Whitelist](ui-whitelist.md) | The complete list of methods/properties exposed to UI / animation events. |

## Concept summary

Three distinct dispatch surfaces feed into your U# methods:

1. **Input events** — `InputJump`, `InputUse`, axis events. Dispatched by the VRChat client every frame the input is active. Local-only.
2. **UI events** — `Button.onClick`, `Slider.onValueChanged`, `Toggle.onValueChanged`, etc. Configured in the inspector to call a public no-arg method on an UdonBehaviour. Equivalent to a local `SendCustomEvent`.
3. **Animation events** — Unity's Animation Window timeline events. Trigger at specified timestamps during animation playback. Limited to a whitelisted set of methods.

All three eventually route through `SendCustomEvent` semantically. The whitelists for UI / animation events exist because the configured event happens *outside* of U# code (in scene data, in animation clip data), so VRChat caps what they can call.

## See also

- [UdonSharpBehaviour → input events](../udonsharp-behaviour/input-events.md)
- [UdonSharpBehaviour → interaction events](../udonsharp-behaviour/interaction-events.md)
