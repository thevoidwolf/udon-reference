# String Loading

`VRC.SDK3.StringLoading.VRCStringDownloader` — load text from a URL. Source: <https://creators.vrchat.com/worlds/udon/string-loading/>. **[Docs]**

## API

```csharp
// static method on VRC.SDK3.StringLoading.VRCStringDownloader
LoadUrl(VRCUrl url, IUdonEventReceiver udonBehaviour);
```

Parameters per the docs: **Url** (the `VRCUrl` to load) and **UdonBehaviour** (the behaviour that receives the events; in Udon Graph it defaults to the current one). **[Docs]** The receiver is typed `IUdonEventReceiver` in VRChat's official image-loading example (`_udonEventReceiver = (IUdonEventReceiver)this;`). **[Community]** <https://github.com/vrchat-community/examples-image-loading> The docs don't document a return value (older revisions of this page claimed `IVRCStringDownload`) — **[Unverified]**; read results from the callbacks.

From U#, pass `this`. Since **SDK 3.10.4** `UdonSharpBehaviour` implements `IUdonEventReceiver`, so no `(IUdonEventReceiver)this` cast is needed (the official example code predating 3.10.4 uses the cast). **[Docs]** (release notes)

The SDK also ships a ready-made `DownloadString` Udon program: add an `UdonBehaviour`, pick `DownloadString` as program source, set the URL and target text component. **[Docs]**

## Callbacks

```csharp
public override void OnStringLoadSuccess(IVRCStringDownload result) {
    string content = result.Result;
    // also accessible: result.ResultBytes for raw bytes
}

public override void OnStringLoadError(IVRCStringDownload result) {
    Debug.LogError($"failed: {result.ErrorCode} - {result.Error}");
}
```

## `IVRCStringDownload`

| Property | Type | Meaning |
|---|---|---|
| `Result` | `string` | The downloaded text, decoded as UTF-8. |
| `ResultBytes` | `byte[]` | Raw bytes for custom decoding (e.g. `System.Text.Encoding.ASCII.GetString`). Returns a **copy** each access. Added SDK 3.5.2. |
| `Error` | `string` | Error message (for `OnStringLoadError`). |
| `ErrorCode` | `int` | HTTP error code (for `OnStringLoadError`). |
| `UdonBehaviour` | `UdonBehaviour` | The behaviour events are sent to. |
| `Url` | `VRCUrl` | The URL the download was attempted from. |

**[Docs]** <https://creators.vrchat.com/worlds/udon/string-loading/> — note there is no `ErrorMessage` property and `Error` is a `string`, not an enum (unlike `IVRCImageDownload`).

## Limits

| Limit | Value |
|---|---|
| Max content size | 100 MB per string |
| Queue capacity | 1000 items |
| Rate limit | 1 string every 5 seconds; excess requests are queued and downloaded **in random order** |
| Redirects | Not followed — the URL must serve the content directly (see below) |

**[Docs]** (size, queue, rate limit, random order). Redirect behaviour: **[Community]** — VRChat staff told a security reporter "redirects will be blocked on VRChat side" (<https://ask.vrchat.com/t/developer-update-19-january-2023/15936?page=3>); a 2024 feature request to allow redirects for string/image loading is still open (<https://feedback.vrchat.com/udon/p/enable-url-redirect-for-stringloading-image-loading>). Image loading officially rejects redirects. **[Docs]**

## Trusted hosts

Default allowlist: DisBridge (`*.disbridge.com`), GitHub Pages (`*.github.io`), GitHub Gist (`gist.githubusercontent.com`), Pastebin (`pastebin.com`), VRCDN (`*.vrcdn.cloud`). Anything else loads only if the user enabled "Allow Untrusted URLs". **[Docs]** See [Trusted URLs](trusted-urls.md).

## Patterns

### Load a JSON config at world load

```csharp
public VRCUrl configUrl;
DataDictionary _config;

void Start() {
    VRCStringDownloader.LoadUrl(configUrl, this);   // pre-3.10.4: cast (IUdonEventReceiver)this if the compiler rejects `this`
}

public override void OnStringLoadSuccess(IVRCStringDownload result) {
    if (VRCJson.TryDeserializeFromJson(result.Result, out DataToken tok)
        && tok.TokenType == TokenType.DataDictionary) {
        _config = tok.DataDictionary;
        ApplyConfig();
    }
}

public override void OnStringLoadError(IVRCStringDownload result) {
    Debug.LogError($"config load failed: {result.ErrorCode} - {result.Error}");
}
```

### Periodic refresh

```csharp
void Start() {
    SendCustomEventDelayedSeconds(nameof(_Refresh), 5f);
}
public void _Refresh() {
    VRCStringDownloader.LoadUrl(url, this);
    SendCustomEventDelayedSeconds(nameof(_Refresh), 60f);   // every minute
}
```

(Mind the 5-second rate limit shared across all string downloads.)

## Caveats

- **Rate limit is global, not per-URL.** Two `LoadUrl` calls within 5 s will queue. **[Docs]** (That it is per-client rather than per-instance is **[Community-known]**; the image-loading page says its equivalent limit "applies to your entire scene".)
- **Queued requests complete in random order.** Don't assume FIFO when firing several loads. **[Docs]**
- **Queue at 1000 items.** Don't burst-load thousands of URLs.
- **No partial downloads, no streaming.** You get the whole string in one callback.
- **Network failures are not exceptions.** Use the error callback.

## Community patterns

- **Config from GitHub Gist.** Publish a JSON gist via `gist.githubusercontent.com/<user>/<id>/raw`; world fetches on join, parses with `VRCJson.TryDeserializeFromJson`. The dominant pattern. **[Community]**
- **News tickers / leaderboards via Pastebin or GitHub Pages.** For dynamic data, back a `*.github.io` page with CI builds that regenerate the file. **[Community-known]**
- **The "redirect via VRCDN" trick.** Since `*.vrcdn.cloud` is on the trusted list, some creators host a small CDN endpoint there that redirects to the real source (e.g. a Google Sheets CSV export). Lets you fetch dynamic data without prompting users for "Allow Untrusted URLs". Same family as `Discord2VRC`'s redirect approach. **[Community]** <https://github.com/acertainbluecat/Discord2VRC>
- **Bundle on the server.** Instead of N separate URLs, emit one JSON blob containing all configs/translations/leaderboards — one download, parse client-side. Cuts past the 5 s rate limit.
- **Server-driven cadence.** Include `nextRefreshSeconds` in the blob so the server controls polling intervals.
- **ETag-style cache key.** Include a `version` field; only update UI if version increased.
- **Don't poll for "live" data.** Worlds in production target eventually-consistent — 60 s refresh is generally enough.

## Community helpers

- **`VRC-Simple-String-Loader`** (DrBlackRat) — drop-in announcement-board prefab with auto-reload + error states. **[Community]** <https://github.com/DrBlackRat/VRC-Simple-String-Loader>
- **`vr-voyage/vrchat-remote-world-list-loader`** — downloads a TSV from pastebin, opens portals dynamically. **[Community]** <https://github.com/vr-voyage/vrchat-remote-world-list-loader>
- **`UdonSharp-TranslateManager`** (Sonic853) — PO-file-based translation system loading remote PO via `VRCStringDownloader`. Archived but illustrative. **[Community]** <https://github.com/Sonic853/UdonSharp-TranslateManager>

## See also

- [Trusted URLs](trusted-urls.md)
- [Data Containers → VRCJson](../data-containers/vrc-json.md) — parsing the result.
- [References → Community Resources](../references/community-resources.md)
