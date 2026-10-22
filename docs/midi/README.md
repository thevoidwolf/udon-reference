# MIDI

Udon receives MIDI messages from two sources: **realtime** input from a device plugged into the local user's computer, and **playback** of a `.mid` file synced to an audio clip. Input-only — there's no MIDI output API.

Source: <https://creators.vrchat.com/worlds/udon/midi/> (with sub-pages [Realtime MIDI](https://creators.vrchat.com/worlds/udon/midi/realtime-midi/) and [MIDI Playback](https://creators.vrchat.com/worlds/udon/midi/midi-playback/)). **[Docs]**

## Events on UdonSharpBehaviour

```csharp
public virtual void MidiNoteOn(int channel, int number, int velocity)
public virtual void MidiNoteOff(int channel, int number, int velocity)
public virtual void MidiControlChange(int channel, int number, int value)
```

| Param | Range | Notes |
|---|---|---|
| `channel` | 0–15 | MIDI standard is 1–16; this is 0-indexed. |
| `number` | 0–127 | Note number / controller number. |
| `velocity` / `value` | 0–127 | Velocity / controller value. |

**[Docs]**

See [UdonSharpBehaviour → MIDI events](../udonsharp-behaviour/midi-events.md) for examples and patterns.

## Realtime input setup

**In the scene:** add a **VRC Midi Listener** component (`VRCMidiListener`) to a GameObject. **[Docs]**
- Tick the **Active Events** you want to receive. None are selected by default, so nothing fires until you enable them.
- Set **Behaviour** to the UdonBehaviour that receives the events (same GameObject or another).
- A `VRCMidiHandler` GameObject appears automatically at runtime and owns the device connection. **Don't add it yourself.**

**Device selection (in-game):** VRChat opens the **first MIDI device it finds** when you enter a world that uses MIDI events. To pick a specific device, use the launch option `--midi=<name>`, for example `--midi=midikeysmasher`. It matches part of the device name and ignores case. There is no in-game settings menu for this, and no Udon API to select or list devices. **[Docs]** Launch options: <https://docs.vrchat.com/docs/launch-options>

**Device selection (editor):** pick a test device in the VRChat SDK's MIDI utility window. The choice is saved in your Editor preferences across projects. **[Docs]**

Example world: [Udon Midi Test](https://vrchat.com/home/world/wrld_f8bc6485-dcdf-4646-89d8-14e4772561ee). **[Docs]**

## Playback: `VRCMidiPlayer`

Plays a MIDI file in sync with an `AudioSource` and sends **Note On / Note Off** events to every UdonBehaviour in its **Target Behaviours** array. It works much like an Audio Source, but takes a MIDI asset. **[Docs]**

**Assets:** a `.mid` file is imported as a MIDI asset. Select it and assign its matching **AudioClip** (any Unity-supported audio format). The BPM must be correct for notes to line up with the audio. Fix it in the file, or toggle **Override Bpm** on the asset. **[Docs]**

| Inspector field | Meaning |
|---|---|
| Midi File | The SMF-format MIDI asset to play. |
| Audio Source | `AudioSource` holding the matching clip. |
| Target Behaviours | `UdonBehaviour[]` that receive `MidiNoteOn` / `MidiNoteOff`. |
| Display Debug Blocks | Editor-only note visualisation in the Scene view while selected. |

| Member | Meaning |
|---|---|
| `Play()` | Start MIDI events and the Audio Source. |
| `Stop()` | Stop both. |
| `float time` | Get/set the current time of both MIDI and audio. |
| `MidiData midiData` | All data about the current track. Readable before playback, for setup. |

**[Docs]** `GetComponent` works on `VRCMidiPlayer` since SDK 3.7.5. **[Docs]** For a working example, open **VRChat SDK → Samples → MidiPlayback** in Example Central. **[Docs]**

### `MidiData` → `MidiTrack` → `MidiBlock`

| Class | Member | Type | Meaning |
|---|---|---|---|
| `MidiData` | `Tracks` | `MidiTrack[]` | Every track in the file. |
| | `Bpm` | `byte` | Track BPM. |
| `MidiTrack` | `Blocks` | `MidiBlock[]` | Every note in the track. |
| | `minNote` / `maxNote` | `byte` | Lowest / highest note played. |
| | `minVelocity` / `maxVelocity` | `byte` | Lowest (excluding 0) / highest velocity. |
| `MidiBlock` | `Note`, `Velocity` | `byte` | 0–127. |
| | `Channel` | `byte` | **1–16** (note: the events use **0–15**). |
| | `StartTimeMs`, `EndTimeMs` | `float` | Note On / Note Off time in milliseconds. |
| | `StartTimeSec`, `EndTimeSec`, `LengthSec` | `float` | The same in seconds, plus duration. |

**[Docs]** Member casing above is copied from the official tables, which mix PascalCase and camelCase. Confirm it in your IDE. **[Unverified]**

## Limitations

- **Input only.** No way to send MIDI from Udon.
- **Only Note On / Note Off / Control Change.** No SysEx, pitch bend, aftertouch or program change events are documented. Playback sends only Note On / Note Off. **[Docs]**
- **No 14-bit / high-resolution CC documented.** Standard 14-bit CC arrives as two 7-bit messages (MSB on controller *n*, LSB on *n*+32), so expect two separate `MidiControlChange` calls to combine yourself. **[Unverified]**
- **Local only.** Events fire on the local client. To broadcast to others, use `SendCustomNetworkEvent`.
- **Infinite-rotation encoders** may send just 0 and 1, or another range, to mean decrement / increment. You must accumulate the position yourself. **[Docs]**
- **Note On with velocity 0** reaches `MidiNoteOn` as-is (SDK 3.2.1+ no longer converts it to Note Off). Treat it as a release. **[Docs]**

## See also

- [UdonSharpBehaviour → MIDI events](../udonsharp-behaviour/midi-events.md)
- [Input & UI → Input Events](../input-and-ui/input-events.md)
