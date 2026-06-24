# UI / Animation Event Whitelist

The complete list of component methods / properties that VRChat allows to be called or set from a UI event (`Button.onClick` etc.) or Animation Event. Source: <https://creators.vrchat.com/worlds/udon/ui-events/>. **[Docs]**

This is *separate* from what you can call from C# code in your U# script. From script you have full access to most Unity APIs. The whitelist below only applies to **inspector-configured events**.

## `UdonBehaviour`

| Method |
|---|
| `RunProgram(string)` |
| `SendCustomEvent(string)` |
| `Interact()` |

## `Animator`

Methods: `Play`, `PlayInFixedTime`, `Rebind`, `SetBool`, `SetFloat`, `SetInteger`, `SetTrigger`, `ResetTrigger`.
Properties: `speed`.

## `AudioSource`

Methods: `Pause`, `Play`, `PlayDelayed`, `PlayOneShot`, `Stop`, `UnPause`.
Properties: `bypassEffects`, `bypassListenerEffects`, `bypassReverbZones`, `dopplerLevel`, `enabled`, `loop`, `maxDistance`, `rolloffMode`, `minDistance`, `mute`, `pitch`, `playOnAwake`, `priority`, `spatialize`, `spread`, `time`, `volume`.

## `AudioDistortionFilter`, `AudioEchoFilter`

Properties: `decayRatio`, `delay`, `dryMix`, `enabled`, `wetMix`.

## `AudioHighPassFilter`, `AudioLowPassFilter`

Properties: `cutoffFrequency`, `enabled`, `resonanceQ`.

## `AudioReverbFilter`

Properties: `decayHFRatio`, `decayTime`, `density`, `diffusion`, `dryLevel`, `enabled`, `hfReference`, `reflectionsDelay`, `reflectionsLevel`, `reverbDelay`, `reverbLevel`, `room`, `roomHF`, `roomLF`.

## `AudioReverbZone`

Properties: `decayHFRatio`, `decayTime`, `density`, `diffusion`, `dryLevel`, `enabled`, `hfReference`, `maxDistance`, `minDistance`, `reflectionsDelay`, `reflectionsLevel`, `reverbDelay`, `reverbLevel`, `room`, `roomHF`, `roomLF`.

## `Button` (and `Selectable`, `Toggle`)

Properties: `enabled`, `interactable`, `targetGraphic`.

## `Toggle`

Plus: `isOn`.

## `Slider`, `Scrollbar`

Properties: `enabled`, `value`, `minValue`, `maxValue`, `interactable`, `targetGraphic`, `handleRect`.

## `Dropdown`

Properties: `captionText`, `enabled`, `interactable`, `itemText`, `targetGraphic`, `template`, `value`.

## `InputField`

Methods: `ForceLabelUpdate`.
Properties: `caretBlinkRate`, `caretPosition`, `caretWidth`, `characterLimit`, `customCaretColor`, `enabled`, `interactable`, `readOnly`, `text`, `textComponent`. **String length capped at 16,000 characters per `text` write.**

## `ScrollRect`

Properties: `content`, `decelerationRate`, `elasticity`, `enabled`, `horizontal`, `verticalNormalizedPosition`, `horizontalNormalizedPosition`, `scrollbars`.

## `Text`, `Image`, `RawImage`

Properties: `enabled`, `text` (Text only), `fontSize` (Text only), `fillAmount` (Image only), `maskable`, `raycastTarget`.

## `Light`

Methods: `Reset`.
Properties: `bounceIntensity`, `colorTemperature`, `cookie`, `enabled`, `intensity`, `range`, `shadow*` properties, `spotAngle`.

## `MeshRenderer`, `SkinnedMeshRenderer`

Properties: `shadowCastingMode`, `enabled`, `probeAnchor`, `receiveShadows`, `lightProbeUsage`.

## `LineRenderer`, `TrailRenderer`

Methods: `Clear`.
Properties: `enabled`, `startWidth`, `endWidth`, `loop`, `motionVectorGenerationMode`.

## `ParticleSystem`

Methods: `Clear`, `Emit`, `Pause`, `Play`, `Simulate`, `Stop`, `TriggerSubEmitter`.
Properties: `time`, `useAutoRandomSeed`.

## `Collider`

Properties: `enabled`, `isTrigger`.

## `GameObject`

Methods: `SetActive`.

## `Transform`

Not exposed via UI events. Mutate transform from C# inside your U# script.

## Why this exists

UI events live in scene serialization data, not in Udon code. The Udon VM can't sandbox arbitrary `UnityEvent` invocations at runtime. The whitelist limits what scene authors (and animation authors) can wire up directly. Anything more complex routes through `UdonBehaviour.SendCustomEvent`, where the U# code is itself sandboxed at compile time.

## See also

- [UI Events](ui-events.md)
- [Animation Events](animation-events.md)
