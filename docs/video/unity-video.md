# VRCUnityVideoPlayer

`VRC.SDK3.Video.Components.VRCUnityVideoPlayer` — wraps Unity's built-in `VideoPlayer`. Doesn't handle live streams but **does** preview in the Unity Editor's Play Mode, which makes it the better choice when you're iterating on URL-based video. **[Docs]** The editor preview only works with links that point straight at supported files (`.mp4`, `.webm`). Hosted services such as YouTube and Vimeo resolve only in the VRChat client. **[Docs]**

## Same surface as AVPro

The API matches `VRCAVProVideoPlayer` for `LoadURL`, `PlayURL`, `Play`, `Pause`, `Stop`, `GetTime`, `SetTime`, `GetDuration`, `Loop`, `IsPlaying`, `IsReady`, `EnableAutomaticResync`, plus `VideoWidth` / `VideoHeight` (SDK 3.6.1+, **[Docs]**). Both inherit these from `BaseVRCVideoPlayer`. **[Community-known]** See [VRCAVProVideoPlayer](av-pro.md) for the call shapes.

The 5-second URL rate limit and the untrusted-URL rules for public instances (SDK 3.7.5+) apply here too. See [Video](README.md#untrusted-urls-in-public-instances-sdk-375). **[Docs]**

## What's different

| | AVPro | Unity |
|---|---|---|
| Live streams | Supported via `PlayURL` | **Not supported** |
| Editor Play Mode preview | No | Yes (direct file links only) |
| Codec coverage | Broad (AVPro's strength) | Whatever Unity's VideoPlayer supports |
| `PlayURL` distinction | Required for streams | No semantic difference vs `LoadURL` + `Play` |

## When to use

- Prototyping a video feature without doing Build & Test cycles.
- Worlds that play pre-rendered, file-linked videos (not streams).
- Embedded video on Quest where AVPro's stream features aren't needed.

## See also

- [VRCAVProVideoPlayer](av-pro.md)
- [UdonSharpBehaviour → video events](../udonsharp-behaviour/video-events.md)
