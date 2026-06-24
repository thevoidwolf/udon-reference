# Trusted URLs

VRChat gates external URL access through a per-subsystem allowlist. The user's setting "Allow Untrusted URLs" bypasses the prompt; trusted hosts skip it entirely.

## Per-subsystem allowlists

Each loader has its own host list. Below is the union of what's documented as of writing — verify with current VRChat docs for any specific subsystem. **[Docs]** **[Unverified]** as a unified list.

### Strings (`VRCStringDownloader`)

Trusted host patterns by default (per <https://wiki.vrchat.com/wiki/Trusted_URLs>) **[Community]**:

- `*.disbridge.com`
- `*.github.io`
- `gist.githubusercontent.com`
- `pastebin.com`
- `*.vrcdn.cloud`

**Google Sheets is NOT trusted.** Common workaround: deploy a Google Apps Script as a Web App and front it through GitHub Pages (`*.github.io`) or a VRCDN-hosted redirector.

Source: <https://creators.vrchat.com/worlds/udon/string-loading/>.

### Images (`VRCImageDownloader`)

Trusted by default: **DisBridge**, **Dropbox**, **GitHub**, **ImageBam**, **ImgBB**, **imgbox**, **Imgur**, **Postimages**, **Reddit**, **Twitter**, **VRCDN**, **VRChat**, **Ytimg**.

Source: <https://creators.vrchat.com/worlds/udon/image-loading/>.

### Videos (`VRCAVProVideoPlayer` / `VRCUnityVideoPlayer`)

Trusted by default: YouTube and related (the AVPro player decodes streams via the YouTube ABR pipeline), plus VRCDN and similar. The video allowlist is generally broader because most video sites are recognized stream sources.

Source: <https://creators.vrchat.com/worlds/udon/video-players/>.

## What "untrusted" means

If a user tries to view a world that requests a non-trusted URL **and** they have not enabled "Allow Untrusted URLs" in their VRChat settings, the load fails. The error callback fires with a "denied" type error.

To enable: **VRChat Settings → Stream and Streaming Audio → Allow Untrusted URLs** (path may vary by client version). **[Docs]**

## The user-facing prompt

Currently, untrusted URLs do *not* trigger an in-game accept/deny prompt — they either silently fail (default), or silently succeed (if the user enabled untrusted URLs). World authors cannot prompt the user.

## Implications for world design

- **Plan around the default-trusted hosts.** Self-hosted assets on `https://example.com/foo.png` will fail for most users.
- **VRCDN exists for content creators** who want to host video/audio on a trusted CDN without setting up their own allowlisted hosting.
- **Test with "Allow Untrusted URLs" off** to verify your world works for users with default settings.

## See also

- [String Loading](string-loading.md)
- [Graphics → Image Downloader](../graphics/image-downloader.md)
- [Video](../video/README.md)
