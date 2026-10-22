# Video

VRChat ships two video player components, both deriving from `BaseVRCVideoPlayer`. Both are scene components. Drag the prefab you need from `Packages/VRChat SDK - Worlds/Samples/UdonExampleScene/Prefabs/VideoPlayers` into your scene. **[Docs]** The prefabs' sample sync graphs turn looping off so that sync works. To loop, enable *Loop* and remove the UdonBehaviour. **[Docs]**

Source: <https://creators.vrchat.com/worlds/udon/video-players/>. **[Docs]**

## Pages

| Page | Topic |
|---|---|
| [VRCAVProVideoPlayer](av-pro.md) | The AVPro-backed player. Live stream support. |
| [VRCUnityVideoPlayer](unity-video.md) | The Unity VideoPlayer-backed player. Editor preview support. |

## When to use which

| Feature | AVPro | Unity |
|---|---|---|
| Live streams (YouTube Live, Twitch) | **Yes** | No |
| Editor Play Mode preview | No (needs Build & Test) | **Yes**, for direct `.mp4` / `.webm` links only. YouTube, Vimeo and other hosted services work only in the client. |
| Linked file URLs | Yes | Yes |
| Audio support | 6-channel (5.1), EAC3 7.1 on PCVR | Standard stereo |
| Android / Quest | Yes. The URL resolver now runs on Android too. | Yes |

On Android, VRChat doesn't play video from hosts that don't use **HTTPS**. **[Docs]** <https://creators.vrchat.com/worlds/udon/video-players/www-whitelist/>

Most production worlds use AVPro. Use Unity Video when you want quick iteration on a non-stream URL in the editor.

## Rate limit (both players)

> *"A given user is only permitted to handle a new video player URL once every five seconds."* **[Docs]**

This is per **user** and global across all video players. It covers default URLs as well as `LoadURL` / `PlayURL`. A late joiner in a world with two playing video players has to send two requests, and if both go out at once one fails, so stagger them. **[Docs]**

## Untrusted URLs in public instances (SDK 3.7.5+)

Since SDK 3.7.5, video players **won't play untrusted URLs in Public and Group Public instances** unless the domain is in the world's own **Video Player Allowed Domains** list. **[Docs]** <https://creators.vrchat.com/releases/release-3-7-5/>

| Instance type | Allowlisted service (YouTube, Twitch, VRCDN, …) | Other domain |
|---|---|---|
| Public / Group Public | Plays for everyone | Plays only if the domain is in the world's allowed domains **and** the viewer enabled *Allow Untrusted URLs* |
| All other instance types | Plays for everyone | Plays if the viewer enabled *Allow Untrusted URLs* |

The rule applies to hardcoded URLs and to URLs users enter at runtime alike. To add domains, open vrchat.com → *My Worlds* → your world → **Edit World Info** → **Video Player Allowed Domains**, and use full domain names such as `example.com`. **[Docs]** <https://creators.vrchat.com/worlds/udon/video-players/www-whitelist/>

Short links may fail because the *resolved* URL has to match an allowlisted domain. The sample sync player also doesn't handle an instance master who has *Allow Untrusted URLs* off. The docs suggest giving sync ownership to the user who requests the video. **[Docs]**

## Events on UdonSharpBehaviour

Place the listener UdonSharpBehaviour on the same GameObject as the video player. Events covered on [UdonSharpBehaviour → video events](../udonsharp-behaviour/video-events.md):

- `OnVideoReady`
- `OnVideoStart` / `OnVideoPlay` / `OnVideoPause` / `OnVideoEnd` / `OnVideoLoop`
- `OnVideoError(VideoError)`

## VideoError

The `VRC.SDK3.Components.Video.VideoError` enum carries the failure reason. Values include `Unknown`, `InvalidURL`, `AccessDenied`, `PlayerError`, `RateLimited`. Exact list may vary; check the SDK source. **[Unverified]**

## Community-built players

The official video-players page points readers to **VideoTXL**, **ProTV** and **USharpVideo** as community prefabs. **[Docs]** <https://creators.vrchat.com/worlds/udon/video-players/> Its supported-hosts section also carries a disclaimer that "listings… do not constitute partnerships or endorsements". Each is open-source and a useful read for understanding network-sync video patterns.

| Project | Distinctive feature | Source |
|---|---|---|
| **USharpVideo** (Merlin) | Canonical synced player. Network-time-anchored sync; manual sync for late joiners. | <https://github.com/MerlinVR/USharpVideo> |
| **USharpVideoQueue** (sam-ln) | Synced playlist queue add-on. | <https://github.com/sam-ln/USharpVideoQueue> |
| **VideoTXL** | Three-manager architecture (Video / Audio / Screen). Ships **Sync** and **Local** prefabs. AudioLink + VRSL integration. | <https://github.com/vrctxl/VideoTXL>, <https://vrctxl.github.io/Docs/> |
| **ProTV** (ArchiTech) | Event-driven plugin architecture. Multi-config swap. LTCGI adapters. | <https://gitlab.com/techanon/protv>, <https://protv.dev/faq> |
| **SynMediaPlayer** | Gapless playback, automatic retry-on-failure, low-latency stream selection. | <https://github.com/synergiance/SynMediaPlayer> |
| **VVMW / VizVid** (JLChnToZ) | Per-platform URL pairs (PC vs Quest fallback), RTSP/RTMP, request debouncing. | <https://github.com/JLChnToZ/VVMW> |
| **USharpVideoModernUI** (DrBlackRat) | Drop-in UI replacement for USharpVideo. | <https://drblackrat.github.io/vpm-listing/> |
| **RiskiPlayer** | Modified ProTV with on-screen UI. | <https://github.com/RiskiVR/RiskiPlayer> |

All **[Community]**.

## Common community patterns

Across the ecosystem, recurring designs:

- **Network-time anchor model** — instead of streaming the current playhead, broadcast a single `_videoStartNetworkTime` and have each client compute `offset = (NetworkTime − startTime)`. USharpVideo's approach. **[Community]** <https://github.com/MerlinVR/USharpVideo>
- **Drift correction with threshold** — periodically re-evaluate local offset vs. computed offset and `SetTime` only if drift exceeds ~0.85 s. Prevents per-frame jumpiness from network-time jitter.
- **Per-platform URL pairs** — store both a PC URL and a Quest URL on each video request; pick at runtime based on platform. Solves YouTube's region/codec divergence. **[Community]** (VVMW's headline feature)
- **Auto-retry on failure** — `OnVideoError` schedules a delayed reload while respecting the 5 s URL cooldown. **[Community]** (SynMediaPlayer)
- **Local-vs-sync prefab variants** — VideoTXL ships both; gives creators a clean "I want a video that doesn't pay the sync cost" opt-out.

## Known issues (community-tracked)

- **`Networking.GetServerTimeInMilliseconds()` returns negative values for some players.** The sign persists per-session. Any time-anchored sync algorithm must use sign-aware deltas. **[Community]** <https://phi16.hatenablog.com/entry/2025/01/08/023655>
- **5-second URL cooldown is per-user, not per-player.** Multiple players joining during a URL change can hit the rate limit simultaneously; queue and stagger.
- **`VRCUrl` instances can only be constructed at world build-time or via player text input.** There is no runtime URL string construction. ProTV's FAQ documents this. **[Community]** <https://protv.dev/faq>

## See also

- [UdonSharpBehaviour → video events](../udonsharp-behaviour/video-events.md)
- [Loading → trusted URLs](../loading/trusted-urls.md)
- Official [Video Player Allowlist](https://creators.vrchat.com/worlds/udon/video-players/www-whitelist/), the full list of trusted video domains.
- [References → Community Resources](../references/community-resources.md) — complete video-player index.
