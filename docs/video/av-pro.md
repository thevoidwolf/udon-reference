# VRCAVProVideoPlayer

`VRC.SDK3.Video.Components.AVPro.VRCAVProVideoPlayer` is the AVPro-backed video player. The official page <https://creators.vrchat.com/worlds/udon/video-players/> covers when to pick AVPro, live streams, rate limits and hosts. **[Docs]** It does **not** list the player's Udon members. Members below are **[Community-known]** (from the shared `BaseVRCVideoPlayer` surface used by every community player) unless tagged otherwise.

## Setup

1. Drop the `AVProVideoPlayer` prefab into the scene from `Packages/VRChat SDK - Worlds/Samples/UdonExampleScene/Prefabs/VideoPlayers`. **[Docs]**
2. Configure the prefab's `Material` and `Screen` references.
3. Add your own `UdonSharpBehaviour` that holds a `VRCAVProVideoPlayer` reference and listens to events.

## Methods

```csharp
void LoadURL(VRCUrl url);    // load but don't auto-play
void PlayURL(VRCUrl url);    // load and start; **required for live streams**
void Play();
void Pause();
void Stop();
```

The docs say live streams need a graph that calls `PlayURL`. **[Docs]** In practice `LoadURL` + `Play` behaves differently on streams. **[Community-known]**

## Properties

| Member | Type | Notes |
|---|---|---|
| `IsPlaying` | `bool` | |
| `IsReady` | `bool` | True once the URL has loaded and the player can begin. |
| `Loop` | `bool` (settable) | Loop the current source. |
| `EnableAutomaticResync` | `bool` (settable) | Periodic resync to compensate for clock drift on remote clients. |
| `VideoWidth`, `VideoHeight` | `int` (read-only) | **SDK 3.6.1+**, on `BaseVRCVideoPlayer` and both subclasses. **[Docs]** <https://creators.vrchat.com/releases/release-3-6-1/> |
| `Volume` | — | Probably **not** a player member. Volume usually goes through the `AudioSource`(s) wired to the `VRCAVProVideoSpeaker` component. **[Unverified]** |

## Time methods

```csharp
float GetTime();          // current playhead, seconds
void  SetTime(float t);   // seek
float GetDuration();      // total length; not meaningful for live streams
```

## Live streams

- Use `PlayURL` (not `LoadURL`).
- `GetDuration` returns no usable length for streams. Reports vary between `Infinity`, `0` and negative values, so check with `float.IsInfinity(d) || d <= 0`. **[Unverified]**
- Loop has no effect on streams.
- Supported sources: YouTube Live, Twitch, and some other streaming services. **[Docs]**
- Audio: AVPro's speaker component suggests 8-channel support, but it plays only 6 channels (5.1). EAC3 7.1 works on PCVR only. **[Docs]**

## Editor preview limitation

AVPro does not preview inside Unity Play Mode. Use Build & Test to verify behaviour. **[Docs]**

## Quest

Works on Quest. The URL resolver VRChat uses is available on Android now, so the old workarounds aren't needed. **[Docs]** Android doesn't play video from non-HTTPS hosts. **[Docs]**

## Untrusted URLs

Since SDK 3.7.5, public and group-public instances block non-allowlisted domains unless you add them to the world's *Video Player Allowed Domains*. See [Video → Untrusted URLs](README.md#untrusted-urls-in-public-instances-sdk-375). **[Docs]**

## Patterns

### Auto-play on world load

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.None)]
public class AutoPlay : UdonSharpBehaviour {
    public VRCAVProVideoPlayer videoPlayer;
    public VRCUrl url;

    void Start() {
        if (Utilities.IsValid(url)) videoPlayer.LoadURL(url);
    }

    public override void OnVideoReady() => videoPlayer.Play();
    public override void OnVideoError(VRC.SDK3.Components.Video.VideoError err) {
        Debug.LogError($"video error: {err}");
    }
}
```

### Networked playback

To keep playback synchronized across players, sync the playhead and play/pause state on the owner; non-owners follow:

```csharp
[UdonBehaviourSyncMode(BehaviourSyncMode.Manual)]
public class SyncedPlayer : UdonSharpBehaviour {
    public VRCAVProVideoPlayer videoPlayer;
    [UdonSynced] VRCUrl _url;
    [UdonSynced] float _startTime;
    [UdonSynced] bool _playing;

    public void Set(VRCUrl url) {
        if (!Networking.IsOwner(gameObject))
            Networking.SetOwner(Networking.LocalPlayer, gameObject);
        _url = url;
        _playing = true;
        _startTime = (float)((Networking.GetServerTimeInMilliseconds() % 86400000) / 1000.0);
        videoPlayer.PlayURL(_url);
        RequestSerialization();
    }

    public override void OnDeserialization() {
        if (!Networking.IsOwner(gameObject) && Utilities.IsValid(_url)) {
            videoPlayer.PlayURL(_url);
        }
    }
}
```

(Simplified. Production worlds usually adjust for network latency more carefully. Sending a raw synced `VRCUrl` like this also runs into the untrusted-URL rules above when the requester isn't the owner.)

## See also

- [UdonSharpBehaviour → video events](../udonsharp-behaviour/video-events.md)
- [Loading → trusted URLs](../loading/trusted-urls.md)
- [Video Player Allowlist](https://creators.vrchat.com/worlds/udon/video-players/www-whitelist/) **[Docs]**
