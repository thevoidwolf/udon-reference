# UI Event Whitelist

The complete list of component methods / properties that VRChat allows to be called or set from a **UI event** (`Button.onClick`, `Slider.onValueChanged`, etc.). Source: <https://creators.vrchat.com/worlds/udon/ui-events/> (checked Oct 2026). **[Docs]**

**Animation events use a different, shorter list** of 13 function names — see [Animation Events](animation-events.md).

This is *separate* from what you can call from C# code in your U# script. From script you have access to everything in the Udon allowlist (see the SDK's Class Exposure Tree). The whitelist below only applies to **inspector-configured UI events**. Components not listed (e.g. `Transform`, TextMeshPro components) can't be targeted.

Capitalised entries are methods; lower-case entries are properties.

## `UdonBehaviour`

`RunProgram`, `SendCustomEvent`, `Interact`

## `Animator`

Methods: `Play`, `PlayInFixedTime`, `Rebind`, `SetBool`, `SetFloat`, `SetInteger`, `SetTrigger`, `ResetTrigger`.
Properties: `speed`.

## `AudioSource`

Methods: `Pause`, `Play`, `PlayDelayed`, `PlayOneShot`, `Stop`, `UnPause`.
Properties: `bypassEffects`, `bypassListenerEffects`, `bypassReverbZones`, `dopplerLevel`, `enabled`, `loop`, `maxDistance`, `rolloffMode`, `minDistance`, `mute`, `pitch`, `playOnAwake`, `priority`, `spatialize`, `spread`, `time`, `volume`.

## `AudioDistortionFilter`, `AudioEchoFilter`

Properties (the docs list the same set for both): `decayRatio`, `delay`, `dryMix`, `enabled`, `wetMix`.

## `AudioHighPassFilter`

Properties: `cutoffFrequency`, `enabled`, `highpassResonanceQ`.

## `AudioLowPassFilter`

Properties: `cutoffFrequency`, `enabled`, `lowpassResonanceQ`.

## `AudioReverbFilter`

Properties: `decayHFRatio`, `decayTime`, `density`, `diffusion`, `dryLevel`, `enabled`, `hfReference`, `reflectionsDelay`, `reflectionsLevel`, `reverbDelay`, `reverbLevel`, `room`, `roomHF`, `roomLF`.

## `AudioReverbZone`

Properties: `decayHFRatio`, `decayTime`, `density`, `diffusion`, `enabled`, `HFReference`, `LFReference`, `maxDistance`, `minDistance`, `reflections`, `reflectionsDelay`, `room`, `roomHF`, `roomLF`.

## `Button`, `Selectable`

Properties: `enabled`, `interactable`, `targetGraphic`.

## `Toggle`

Properties: `enabled`, `group`, `interactable`, `isOn`, `targetGraphic`.

## `ToggleGroup`

Properties: `allowSwitchOff`, `enabled`.

## `Slider`

Properties: `enabled`, `fillRect`, `handleRect`, `interactable`, `maxValue`, `minValue`, `normalizedValue`, `targetGraphic`, `value`, `wholeNumbers`.

## `Scrollbar`

Properties: `enabled`, `handleRect`, `interactable`, `numberOfSteps`, `size`, `targetGraphic`, `value`.

## `Dropdown`

Properties: `captionText`, `enabled`, `interactable`, `itemText`, `targetGraphic`, `template`, `value`.

## `InputField`

Methods: `ForceLabelUpdate`.
Properties: `caretBlinkRate`, `caretPosition`, `caretWidth`, `characterLimit`, `customCaretColor`, `enabled`, `interactable`, `readOnly`, `selectionAnchorPosition`, `selectionFocusPosition`, `text`, `textComponent`.
**Input fields are limited to 16,000 characters** — the maximum a (legacy) text component can render.

## `ScrollRect`

Properties: `content`, `decelerationRate`, `elasticity`, `enabled`, `horizontal`, `horizontalNormalizedPosition`, `horizontalScrollbar`, `horizontalScrollbarSpacing`, `inertia`, `scrollSensitivity`, `vertical`, `verticalNormalizedPosition`, `verticalScrollbar`, `verticalScrollbarSpacing`, `viewport`.

## `Text`

Properties: `alignByGeometry`, `enabled`, `fontSize`, `lineSpacing`, `maskable`, `raycastTarget`, `resizeTextForBestFit`, `resizeTextMaxSize`, `resizeTextMinSize`, `supportRichText`, `text`.

## `Image`

Properties: `alphaHitTestMinimumThreshold`, `enabled`, `fillAmount`, `fillCenter`, `fillClockwise`, `fillOrigin`, `maskable`, `preserveAspect`, `raycastTarget`, `useSpriteMesh`.

## `RawImage`

Properties: `enabled`, `maskable`, `raycastTarget`.

## `Mask`, `RectMask2D`

`Mask`: `enabled`, `showMaskGraphic`. `RectMask2D`: `enabled`.

## `Light`

Methods: `Reset`.
Properties: `bounceIntensity`, `colorTemperature`, `cookie`, `enabled`, `intensity`, `range`, `shadowBias`, `shadowNearPlane`, `shadowNormalBias`, `shadowStrength`, `spotAngle`.

## `Projector`

Properties: `aspectRatio`, `enabled`, `nearClipPlane`, `farClipPlane`, `fieldOfView`, `orthographic`, `orthographicSize`.

## `MeshRenderer`

Properties: `shadowCastingMode`, `enabled`, `probeAnchor`, `receiveShadows`, `lightProbeUsage`.

## `SkinnedMeshRenderer`

Properties: `allowOcclusionWhenDynamic`, `shadowCastingMode`, `enabled`, `lightProbeProxyVolumeOverride`, `motionVectorGenerationMode`, `probeAnchor`, `receiveShadows`, `rootBone`, `skinnedMotionVectors`, `updateWhenOffscreen`, `lightProbeUsage`.

## `LineRenderer`

Properties: `allowOcclusionWhenDynamic`, `shadowCastingMode`, `enabled`, `endWidth`, `loop`, `motionVectorGenerationMode`, `numCapVertices`, `numCornerVertices`, `probeAnchor`, `receiveShadows`, `shadowBias`, `startWidth`, `lightProbeUsage`, `useWorldSpace`, `widthMultiplier`. (No `Clear` — that's `TrailRenderer` only.)

## `TrailRenderer`

Methods: `Clear`.
Properties: `allowOcclusionWhenDynamic`, `autodestruct`, `shadowCastingMode`, `enabled`, `emitting`, `endWidth`, `motionVectorGenerationMode`, `numCapVertices`, `numCornerVertices`, `probeAnchor`, `receiveShadows`, `shadowBias`, `startWidth`, `lightProbeUsage`, `widthMultiplier`.

## `ParticleSystem`

Methods: `Clear`, `Emit`, `Pause`, `Play`, `Simulate`, `Stop`, `TriggerSubEmitter`.
Properties: `time`, `useAutoRandomSeed`.

## `ParticleSystemForceField`

Properties: `endRange`, `gravityFocus`, `length`, `multiplyDragByParticleSize`, `multiplyDragByParticleVelocity`, `startRange`.

## `Collider`

Properties: `enabled`, `isTrigger`.

## `GameObject`

Methods: `SetActive`.

## `Transform`

Not exposed via UI events. Mutate transform from C# inside your U# script.

## Why this exists

UI events live in scene serialization data, not in Udon code. The Udon VM can't sandbox arbitrary `UnityEvent` invocations at runtime. The whitelists (this one and the [animation-event list](animation-events.md)) limit what scene and animation authors can wire up directly. Anything more complex routes through `UdonBehaviour.SendCustomEvent`, where the U# code is itself sandboxed at compile time.

## See also

- [UI Events](ui-events.md)
- [Animation Events](animation-events.md)
