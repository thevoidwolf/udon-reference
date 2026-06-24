# Voice & Audio

Per-player voice and avatar-audio configuration. Each setter affects only what the caller's client hears — the change is **not** networked. To make everyone hear a player at a different volume, every client must call the setter for that player. The standard pattern is `OnPlayerJoined` plus a `WorldAudioSettings` behaviour. **[Docs]**

## Voice (talk audio)

```csharp
player.SetVoiceGain(float dB);                 // 0..24, default 15
player.SetVoiceDistanceNear(float meters);     // default 0
player.SetVoiceDistanceFar(float meters);      // default 25
player.SetVoiceVolumetricRadius(float meters); // default 0 — keep at 0 unless you know what you're doing
player.SetVoiceLowpass(bool enabled);          // distance low-pass filter; default true
```

| Setting | What it does |
|---|---|
| `VoiceGain` | Master volume of voice in dB. |
| `VoiceDistanceNear` | Distance at which the voice is at full volume. |
| `VoiceDistanceFar` | Distance at which the voice fades to silence. |
| `VoiceVolumetricRadius` | The "radius around the player" treated as full-volume. Default 0 = point source. |
| `VoiceLowpass` | Apply a low-pass filter as distance increases. |

## Avatar audio

```csharp
player.SetAvatarAudioGain(float dB);
player.SetAvatarAudioFarRadius(float meters);
player.SetAvatarAudioNearRadius(float meters);
player.SetAvatarAudioVolumetricRadius(float meters);
player.SetAvatarAudioForceSpatial(bool force);
player.SetAvatarAudioCustomCurve(bool useCustomCurves);
```

| Setting | What it does |
|---|---|
| `AvatarAudioGain` | Maximum gain allowed on avatar AudioSources. |
| `AvatarAudioFarRadius` | Maximum allowed far-distance for avatar audio. 0 = mute. |
| `AvatarAudioNearRadius` | Maximum allowed near-distance. |
| `AvatarAudioVolumetricRadius` | Maximum allowed volumetric radius. |
| `AvatarAudioForceSpatial` | Force avatars to use spatial audio (override per-source setting). |
| `AvatarAudioCustomCurve` | Whether to allow custom distance curves on avatar sources. |

## Canonical pattern (from `Samples~/Utilities/WorldAudioSettings.cs`)

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.NoVariableSync)]
public class WorldAudioSettings : UdonSharpBehaviour {
    public float voiceGain = 15f;
    public float voiceFar = 25f;
    public float avatarMaxAudioGain = 10f;
    // ...

    public override void OnPlayerJoined(VRCPlayerApi player) {
        if (player.isLocal) return;     // don't reconfigure ourselves
        player.SetVoiceGain(voiceGain);
        player.SetVoiceDistanceFar(voiceFar);
        player.SetAvatarAudioGain(avatarMaxAudioGain);
        // ... etc.
    }
}
```

Drop one of these in your scene and every player you encounter gets your world's audio policy applied.

## Caveats

- **Settings do not network.** Every client must apply them to every player it sees. Use `OnPlayerJoined` to catch incoming players.
- **Mid-session changes** require iterating over all players and re-applying.
- **Avatar audio caps** are *maximums* — they don't force audio to a specific level, they cap what avatars can request. Avatars with low source volumes stay quiet.
- Some defaults may differ across VRChat client versions; check current values in the docs at <https://creators.vrchat.com/worlds/udon/players/> (page may have moved). **[Docs]**

## Community library

- **UdonVoiceUtils** (Guribo) — substantially deeper than the stock API: private channels, audio occlusion through walls, reverb zones, voice directionality based on facing direction, push-to-talk / toggle mic activation, automatic voice-range adjustment by avatar height. Targets World SDK 3.10.3+ / Unity 2022.3. **[Community]** <https://github.com/Guribo/UdonVoiceUtils>

If your world needs anything more nuanced than "everyone hears everyone at the same volume," start here before writing your own.

## See also

- [VRCPlayerApi](vrc-player-api.md)
- [UdonSharpBehaviour → player events](../udonsharp-behaviour/player-events.md) — `OnPlayerJoined`.
- [References → Community Resources](../references/community-resources.md)
