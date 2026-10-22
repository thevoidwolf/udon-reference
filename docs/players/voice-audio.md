# Voice & Audio

Per-player voice and avatar-audio configuration. Each setter affects only what the caller's client hears — the change is **not** networked. To make everyone hear a player at a different volume, every client must call the setter for that player. The standard pattern is `OnPlayerJoined` plus a `WorldAudioSettings` behaviour. The docs frame these APIs as changing how the local player hears *other* players **[Docs]**; the per-client application pattern is **[Community-known]**.

Ranges and defaults below are from <https://creators.vrchat.com/worlds/udon/players/player-audio>. **[Docs]**

## Voice (talk audio)

```csharp
player.SetVoiceGain(float dB);                 // 0..24, default 15
player.SetVoiceDistanceNear(float meters);     // 0..1,000,000; docs strongly recommend leaving at 0
player.SetVoiceDistanceFar(float meters);      // 0..1,000,000, default 25; 0 effectively mutes
player.SetVoiceVolumetricRadius(float meters); // 0..1000, default 0 — keep at 0 unless you know what you're doing; keep below DistanceFar
player.SetVoiceLowpass(bool enabled);          // distance low-pass filter; turn off for e.g. DJ audio (default on [Community-known])
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
player.SetAvatarAudioGain(float dB);                  // 0..10, default 10
player.SetAvatarAudioFarRadius(float meters);         // default 40; 0 effectively mutes
player.SetAvatarAudioNearRadius(float meters);        // default 0
player.SetAvatarAudioVolumetricRadius(float meters);  // default 0; keep below FarRadius
player.SetAvatarAudioForceSpatial(bool force);        // default off; on = spatialize, spatialBlend 1
player.SetAvatarAudioCustomCurve(bool useCustomCurves); // default off
```

| Setting | What it does |
|---|---|
| `AvatarAudioGain` | Maximum gain allowed on avatar AudioSources. |
| `AvatarAudioFarRadius` | Maximum allowed far-distance for avatar audio; compared with each source's `maxDistance` and the smaller wins. 0 = mute. |
| `AvatarAudioNearRadius` | Maximum allowed near-distance; also compared with `maxDistance`, smaller wins. |
| `AvatarAudioVolumetricRadius` | Maximum allowed volumetric radius. |
| `AvatarAudioForceSpatial` | Force avatars to use spatial audio (override per-source setting). |
| `AvatarAudioCustomCurve` | Whether avatar sources use their pre-configured custom curve. |

No getters are documented for any of these. **[Docs]**

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
- Some defaults may differ across VRChat client versions; check current values at <https://creators.vrchat.com/worlds/udon/players/player-audio>. **[Docs]**

## Community library

- **UdonVoiceUtils** (Guribo) — substantially deeper than the stock API: private channels, audio occlusion through walls, reverb zones, voice directionality based on facing direction, push-to-talk / toggle mic activation, automatic voice-range adjustment by avatar height. Actively maintained — v7.0.0 (Sep 2026, breaking: blocks legacy network-event abuse, custom avatar-audio curves on by default); Unity 2022.3. Check its README for the minimum Worlds SDK. **[Community]** <https://github.com/Guribo/UdonVoiceUtils>

If your world needs anything more nuanced than "everyone hears everyone at the same volume," start here before writing your own.

## See also

- [VRCPlayerApi](vrc-player-api.md)
- [UdonSharpBehaviour → player events](../udonsharp-behaviour/player-events.md) — `OnPlayerJoined`.
- [References → Community Resources](../references/community-resources.md)
