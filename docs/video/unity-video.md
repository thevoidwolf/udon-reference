# VRCUnityVideoPlayer

`VRC.SDK3.Video.Components.VRCUnityVideoPlayer` — wraps Unity's built-in `VideoPlayer`. Doesn't handle live streams but **does** preview in the Unity Editor's Play Mode, which makes it the better choice when you're iterating on URL-based video. **[Docs]**

## Same surface as AVPro

The API matches `VRCAVProVideoPlayer` for `LoadURL`, `Play`, `Pause`, `Stop`, `GetTime`, `SetTime`, `GetDuration`, `Loop`, `Volume`, `IsPlaying`, `IsReady`, `EnableAutomaticResync`. See [VRCAVProVideoPlayer](av-pro.md) for the call shapes.

## What's different

| | AVPro | Unity |
|---|---|---|
| Live streams | Supported via `PlayURL` | **Not supported** |
| Editor Play Mode preview | No | Yes |
| Codec coverage | Broad (AVPro's strength) | Whatever Unity's VideoPlayer supports |
| `PlayURL` distinction | Required for streams | No semantic difference vs `LoadURL` + `Play` |

## When to use

- Prototyping a video feature without doing Build & Test cycles.
- Worlds that play pre-rendered, file-linked videos (not streams).
- Embedded video on Quest where AVPro's stream features aren't needed.

## See also

- [VRCAVProVideoPlayer](av-pro.md)
- [UdonSharpBehaviour → video events](../udonsharp-behaviour/video-events.md)
