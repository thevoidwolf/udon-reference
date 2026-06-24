# Video Events

UdonBehaviour callbacks dispatched by `BaseVRCVideoPlayer` subclasses (`VRCAVProVideoPlayer`, `VRCUnityVideoPlayer`). The events fire on the **same GameObject as the video player component**.

Source for the signatures: `Runtime/UdonSharpBehaviour.cs`. **[Source]**
Behavioural detail: <https://creators.vrchat.com/worlds/udon/video-players/>. **[Docs]**

## Events

```csharp
public virtual void OnVideoStart()
public virtual void OnVideoPlay()
public virtual void OnVideoPause()
public virtual void OnVideoEnd()
public virtual void OnVideoLoop()
public virtual void OnVideoReady()
public virtual void OnVideoError(VRC.SDK3.Components.Video.VideoError videoError)
```

| Event | Fires when |
|---|---|
| `OnVideoReady` | A URL has finished loading and the player can `Play()`. |
| `OnVideoStart` | A new playback session begins (initial `Play` after load). |
| `OnVideoPlay` | Playback transitions from paused → playing. |
| `OnVideoPause` | Playback transitions from playing → paused. |
| `OnVideoEnd` | Stream finishes (a non-looping clip reaches the end). |
| `OnVideoLoop` | A looping stream restarts (does not fire when `Loop` is off). |
| `OnVideoError` | The player failed to load or play. Inspect `videoError`. |

The `VideoError` enum's exact values are not enumerated in the official docs. **[Unverified]** Inspect at the call site.

## Minimal player skeleton

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDK3.Components.Video;
using VRC.SDK3.Video.Components;        // VRCUnityVideoPlayer
// using VRC.SDK3.Video.Components.AVPro;  // for VRCAVProVideoPlayer
using VRC.SDKBase;

[UdonBehaviourSyncMode(BehaviourSyncMode.None)]
public class SimplePlayer : UdonSharpBehaviour {

    public VRCUnityVideoPlayer videoPlayer;
    public VRCUrl url;

    void Start() {
        if (Utilities.IsValid(url))
            videoPlayer.LoadURL(url);
    }

    public override void OnVideoReady()  => videoPlayer.Play();
    public override void OnVideoEnd()    => Debug.Log("done");
    public override void OnVideoError(VideoError err) => Debug.LogError($"video err: {err}");
}
```

## See also

- [Video → VRCAVProVideoPlayer](../video/av-pro.md) — methods, live streams, format support.
- [Video → VRCUnityVideoPlayer](../video/unity-video.md) — editor-friendly variant.
- [Loading → trusted URLs](../loading/trusted-urls.md) — video URLs share the same trusted-host system.
