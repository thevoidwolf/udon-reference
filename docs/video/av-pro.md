# VRCAVProVideoPlayer

`VRC.SDK3.Video.Components.AVPro.VRCAVProVideoPlayer` — the AVPro-backed video player. Source for the documented methods: <https://creators.vrchat.com/worlds/udon/video-players/>. **[Docs]** Where the docs are silent, members are flagged **[Community-known]**.

## Setup

1. Drop the `AVProVideoPlayer` prefab into the scene from `VRChat SDK - Worlds/Samples/UdonExampleScene/Prefabs/VideoPlayers/`.
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

`PlayURL` differs from `LoadURL + Play` when the source is a live stream — only `PlayURL` handles the live-stream protocol negotiation correctly.

## Properties

| Member | Type | Notes |
|---|---|---|
| `IsPlaying` | `bool` | |
| `IsReady` | `bool` | True once the URL has loaded and the player can begin. |
| `Loop` | `bool` (settable) | Loop the current source. |
| `Volume` | `float` (settable, 0..1) | |
| `EnableAutomaticResync` | `bool` (settable) | Periodic resync to compensate for clock drift on remote clients. |

## Time methods

```csharp
float GetTime();          // current playhead, seconds
void  SetTime(float t);   // seek
float GetDuration();      // total length; -1 or 0 for live streams
```

## Live streams

- Use `PlayURL` (not `LoadURL`).
- `GetDuration` typically returns `-1` for streams.
- Loop has no effect on streams.
- Supported sources: YouTube Live, Twitch, and other compatible streaming services. **[Docs]**

## Editor preview limitation

AVPro does not preview inside Unity Play Mode. Use Build & Test to verify behaviour. **[Docs]**

## Quest

Works on Quest. URL resolution is handled by the VRChat client on Android (no client-side proxy needed in modern SDKs).

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

(Simplified — production worlds usually adjust for network latency more carefully.)

## See also

- [UdonSharpBehaviour → video events](../udonsharp-behaviour/video-events.md)
- [Loading → trusted URLs](../loading/trusted-urls.md)
