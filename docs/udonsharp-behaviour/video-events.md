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
| `OnVideoReady` | The player has loaded a new video and can `Play()`. |
| `OnVideoStart` | Playback starts from a **stopped** state. |
| `OnVideoPlay` | Playback starts — from a stopped state, the next video in a queue, unpausing, or player interaction. |
| `OnVideoPause` | The player is paused. |
| `OnVideoEnd` | Playback finishes — the end of the video *or* stopped via player interaction. |
| `OnVideoLoop` | Looping is enabled and the player finishes a loop. |
| `OnVideoError` | The player hit an error loading the video. Inspect `videoError`. |

**[Docs]** (<https://creators.vrchat.com/worlds/udon/graph/event-nodes/>)

The `VideoError` enum's values are not listed in the official docs. The SDK enum is commonly seen as `Unknown`, `InvalidURL`, `AccessDenied`, `PlayerError`, `RateLimited`. **[Unverified]** Check in your IDE.

Common causes of `OnVideoError`: **[Docs]**
- **Rate limit** — each user may load a new video URL only once every **5 seconds**, globally across all video players (default URLs included).
- **Untrusted URL** — domains outside the allowlist need "Allow Untrusted URLs". In Public / Group Public instances, untrusted domains are blocked unless they're on the world's own allowlist (set on the VRChat website; announced with SDK 3.7.5). See [Loading → trusted URLs](../loading/trusted-urls.md).

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
