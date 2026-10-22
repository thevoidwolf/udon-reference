# MIDI Events

Three callbacks on `UdonSharpBehaviour` receive MIDI messages, from two sources: **[Docs]**

- **Realtime** — a MIDI device plugged into the local user's computer, routed through a `VRC Midi Listener` component.
- **Playback** — a `.mid` file played in sync with an `AudioSource` by a `VRCMidiPlayer` component (Note On / Note Off only).

Local-only — there is no built-in networking.

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

(Your device may not output the full 0–127 range.) **[Docs]**

### `MidiNoteOn`
A Note On message: a key pressed on the device, or a note starting in MIDI playback. `velocity` indicates how hard. **[Docs]**

Since SDK 3.2.1, a Note On with velocity `0` is **no longer converted** into a Note Off (that conversion broke MIDI file parsing). **[Docs]** (3.2.1 release notes) Many devices send "Note On, velocity 0" to mean note release, so treat `velocity == 0` in `MidiNoteOn` as a release.

### `MidiNoteOff`
Note released. `velocity` is typically 0, but may vary by device. **[Docs]**

### `MidiControlChange`
Knob, slider, or wheel motion. For continuous controls, `value` sweeps 0–127 smoothly. For *infinite-rotation* encoders, devices may emit just `0` and `1` (or some other range) to indicate decrement / increment — you must accumulate. **[Docs]**

## Setup

**Realtime input:** add a **VRC Midi Listener** component, tick the **Active Events** you want (none are selected by default), and set its **Behaviour** to the UdonBehaviour that should receive them (same or another GameObject). **[Docs]** Do not add the `VRCMidiHandler` object yourself; VRChat creates it at runtime. **[Docs]**

**Device selection:** in-game, VRChat opens the **first MIDI device it finds**. Users with several devices pick one with the launch option `--midi=<part of device name>` (case-insensitive partial match). In the editor, choose a test device through the VRChat SDK MIDI utility window (saved in Editor preferences). There is no in-world API to select or list devices. **[Docs]**

**Playback:** a `VRCMidiPlayer` sends Note On / Note Off to every UdonBehaviour in its **Target Behaviours** array. See [MIDI → playback](../midi/README.md#playback-vrcmidiplayer). **[Docs]**

## Example

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.None)]
public class MidiVisualizer : UdonSharpBehaviour {
    public GameObject[] noteCubes;  // 128 cubes, one per note number

    public override void MidiNoteOn(int channel, int number, int velocity) {
        if (number < 0 || number >= noteCubes.Length) return;
        if (velocity == 0) { MidiNoteOff(channel, number, 0); return; }  // many devices send Note On vel 0 as a release
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
- Only Note On, Note Off and Control Change are exposed. No SysEx, pitch bend, aftertouch or program change events are documented. **[Docs]**
- No high-resolution / 14-bit CC support is documented. Standard MIDI sends 14-bit CC as two 7-bit messages (MSB on controller *n*, LSB on *n*+32), so they should arrive as two separate `MidiControlChange` calls that you combine yourself. **[Unverified]**
- Events fire on the local client only; if you want everyone to see a reaction, manually `SendCustomNetworkEvent`.

## See also

- [MIDI](../midi/README.md) — setup, `VRCMidiPlayer` playback, `MidiData` classes.
- [Networking → custom network events](../networking/custom-network-events.md) — broadcasting MIDI-driven reactions.
- [Input events](input-events.md) — for non-MIDI controller / keyboard input.
