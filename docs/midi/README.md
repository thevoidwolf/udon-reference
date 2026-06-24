# MIDI

Udon receives MIDI input messages from the local user's configured MIDI device. Input-only — there's no MIDI output API.

Source: <https://creators.vrchat.com/worlds/udon/midi/>. **[Docs]**

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
| `velocity` / `value` | 0–127 | Pressure / controller value. |

See [UdonSharpBehaviour → MIDI events](../udonsharp-behaviour/midi-events.md) for examples and patterns.

## User setup

A user must explicitly choose a MIDI device under:

**VRChat Settings → Controls & Camera → MIDI**

Until they do, no MIDI events fire. The choice is per-player, per-world session — there's no in-world API to select or list devices.

## Limitations

- **Input only.** No way to send MIDI from Udon.
- **No SysEx support.**
- **No 14-bit / high-resolution CC documented.** Devices that send 14-bit CC pairs will surface only the upper 7 bits.
- **Local only.** Events fire on the local client. To broadcast to others, use `SendCustomNetworkEvent`.
- **Infinite-rotation encoders** typically send 0 or 1 to indicate decrement / increment. You must accumulate the position yourself.

## See also

- [UdonSharpBehaviour → MIDI events](../udonsharp-behaviour/midi-events.md)
- [Input & UI → Input Events](../input-and-ui/input-events.md)
