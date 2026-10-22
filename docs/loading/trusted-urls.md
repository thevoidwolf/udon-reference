# Trusted URLs

VRChat gates external URL access through a per-subsystem allowlist. Trusted hosts load for everyone; any other host loads only if the viewer enabled **"Allow Untrusted URLs"** in their settings. **[Docs]** <https://creators.vrchat.com/worlds/udon/external-urls/>

All URLs must be wrapped in a `VRCUrl`. `new VRCUrl(string)` can only be called at **editor time** — at runtime, URLs come from fields baked into the world or from a `VRCUrlInputField` the user types into. The allowlist applies the same way to both sources. **[Docs]**

| | On allowlist | Not on allowlist |
|---|---|---|
| User enters `VRCUrl` into input field | Allowed | Requires "Allow Untrusted URLs" |
| Udon declares the `VRCUrl` before runtime | Allowed | Requires "Allow Untrusted URLs" |

`VRCUrl` API: `VRCUrl.Empty`, `url.Get()`, `VRCUrl.IsNullOrEmpty(VRCUrl)`. **[Docs]**

## Per-subsystem allowlists

Each loader has its own host list. Lists below are copied from the current official pages (checked Oct 2026); VRChat notes they "may change at any time without notice." **[Docs]**

### Strings (`VRCStringDownloader`)

- DisBridge — `*.disbridge.com`
- GitHub — `*.github.io`
- GitHub Gist — `gist.githubusercontent.com`
- Pastebin — `pastebin.com`
- VRCDN — `*.vrcdn.cloud`

**Google Sheets is NOT trusted.** Because string loading doesn't follow redirects (see [String Loading → Limits](string-loading.md#limits)), a redirector won't help — publish the exported data to a trusted host (e.g. a scheduled GitHub Action that writes the CSV/JSON to GitHub Pages). **[Community-known]**

Source: <https://creators.vrchat.com/worlds/udon/string-loading/>.

### Images (`VRCImageDownloader`)

- DisBridge — `*.disbridge.com`
- Dropbox — `dl.dropbox.com`, `dl.dropboxusercontent.com`
- GitHub — `*.github.io`
- ImageBam — `images4.imagebam.com`
- ImgBB — `i.ibb.co`
- imgbox — `images2.imgbox.com`
- Imgur — `i.imgur.com`
- Postimages — `i.postimg.cc`
- Reddit — `i.redd.it`
- Twitter — `pbs.twimg.com`
- VRCDN — `*.vrcdn.cloud`
- VRChat — `assets.vrchat.com`
- Ytimg — `i.ytimg.com`

The URL must point **directly** at the image file — redirection is not allowed and results in an error. **[Docs]**

Source: <https://creators.vrchat.com/worlds/udon/image-loading/>.

### Videos (`VRCAVProVideoPlayer` / `VRCUnityVideoPlayer`)

| Service | Domains |
|---|---|
| Akamai CDN | `vod-progressive.akamaized.net` |
| Facebook Video | `*.facebook.com`, `*.fbcdn.net` |
| Google Video | `*.googlevideo.com` |
| Hyperbeam | `*.hyperbeam.com`, `*.hyperbeam.dev` |
| Mixcloud | `*.mixcloud.com` |
| NicoNico | `*.nicovideo.jp` |
| Soundcloud | `soundcloud.com`, `*.sndcdn.com` |
| Topaz Chat | `*.topaz.chat` |
| Twitch.TV | `*.twitch.tv`, `*.ttvnw.net`, `*.twitchcdn.net` |
| VRCDN | `*.vrcdn.live`, `*.vrcdn.video`, `*.vrcdn.cloud` |
| Vimeo | `*.vimeo.com` |
| Youku | `*.youku.com` |
| YouTube | `*.youtube.com`, `youtu.be` |

Unlike strings/images, video players **do** follow redirects, but the *resolved* URL must match an allowlisted domain (short-links may fail). VRChat on Android won't play non-HTTPS video. **[Docs]**

**Public instances (SDK 3.7.5+ behaviour):** in Public and Group Public instances, untrusted video URLs play only if the domain is in the world's **"Video Player Allowed Domains"** (set on vrchat.com → My Worlds → Edit World Info) *and* the viewer enabled "Allow Untrusted URLs". In other instance types any untrusted domain plays if the viewer allows untrusted URLs. Applies to hardcoded and user-entered URLs alike. **[Docs]**

Source: <https://creators.vrchat.com/worlds/udon/video-players/www-whitelist/>.

## What "untrusted" means

If a world requests a non-trusted URL **and** the viewer has not enabled "Allow Untrusted URLs", the load fails. **[Docs]** It surfaces through the subsystem's normal error callback (`OnStringLoadError`, `OnImageLoadError`, `OnVideoError`) **[Community-known]**; the exact error value reported (e.g. `VRCImageDownloadError.AccessDenied`) is **[Unverified]**.

The toggle is in the VRChat client settings; the exact menu path varies by client version and isn't given in the creator docs. **[Unverified]**

## The user-facing prompt

Untrusted URLs do *not* trigger an in-game accept/deny prompt — they either fail (default) or succeed (if the user enabled untrusted URLs). World authors cannot prompt the user or toggle the setting. **[Community-known]**

## Implications for world design

- **Plan around the default-trusted hosts.** Self-hosted assets on `https://example.com/foo.png` will fail for most users.
- **VRCDN exists for content creators** who want to host video/audio on a trusted CDN without setting up their own allowlisted hosting.
- **Video in public instances:** add every non-allowlisted video domain your world uses to "Video Player Allowed Domains" on the website, or it won't play in Public / Group Public instances even for users who allow untrusted URLs. **[Docs]**
- **Test with "Allow Untrusted URLs" off** to verify your world works for users with default settings.

## See also

- [String Loading](string-loading.md)
- [Graphics → Image Downloader](../graphics/image-downloader.md)
- [Video](../video/README.md)
- Official: [External URLs](https://creators.vrchat.com/worlds/udon/external-urls/), [Video Player Allowlist](https://creators.vrchat.com/worlds/udon/video-players/www-whitelist/)
