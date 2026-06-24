# MIDI Events

Three callbacks on `UdonSharpBehaviour` receive MIDI messages from a device the local user has configured in VRChat's settings. Local-only — there is no built-in networking.

Source for the C# signatures: `Runtime/UdonSharpBehaviour.cs`. **[Source]**
Behavioural docs: <https://creators.vrchat.com/worlds/udon/midi/>. **[Docs]**

## Events

```csharp
public virtual void MidiNoteOn(int channel, int number, int velocity)
public virtual void MidiNoteOff(int channel, int number, int velocity)
public virtual void MidiControlChange(int channel, int number, int value)
```

All parameters:

| Param | Range | Meaning |
|---|---|---|
| `channel` | 0–15 | MIDI channel (standard MIDI is 1–16; here it's 0-indexed). |
| `number` | 0–127 | Note number (NoteOn/Off) or controller number (ControlChange). |
| `velocity` / `value` | 0–127 | Velocity/pressure (NoteOn/Off) or controller value (ControlChange). |

### `MidiNoteOn`
A Note On message — usually a key pressed or pad triggered on the device. `velocity` indicates how hard.

### `MidiNoteOff`
Note released. On many devices `velocity` is 0; on velocity-sensitive devices it represents release speed.

### `MidiControlChange`
Knob, slider, or wheel motion. For continuous controls, `value` sweeps 0–127 smoothly. For *infinite-rotation* encoders, devices typically emit only `0` or `1` to indicate decrement / increment — you must accumulate. **[Docs]**

## Setup

A user must explicitly choose a MIDI device under **VRChat Settings → Controls & Camera → MIDI**. Until they do, no MIDI events fire. The choice is per-player, per-world session — there's no in-world API to select a device.

## Example

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.None)]
public class MidiVisualizer : UdonSharpBehaviour {
    public GameObject[] noteCubes;  // 128 cubes, one per note number

    public override void MidiNoteOn(int channel, int number, int velocity) {
        if (number < 0 || number >= noteCubes.Length) return;
        var cube = noteCubes[number];
        cube.SetActive(true);
        cube.transform.localScale = Vector3.one * (velocity / 127f);
    }

    public override void MidiNoteOff(int channel, int number, int velocity) {
        if (number < 0 || number >= noteCubes.Length) return;
        noteCubes[number].SetActive(false);
    }
}
```

## Limitations

- No way to send MIDI **out**; input-only.
- No high-resolution / 14-bit CC support documented.
- No SysEx events.
- Events fire on the local client only; if you want everyone to see a reaction, manually `SendCustomNetworkEvent`.

## See also

- [Networking → custom network events](../networking/custom-network-events.md) — broadcasting MIDI-driven reactions.
- [Input events](input-events.md) — for non-MIDI controller / keyboard input.
