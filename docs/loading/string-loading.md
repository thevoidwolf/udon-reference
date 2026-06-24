# String Loading

`VRC.SDK3.StringLoading.VRCStringDownloader` — load text from a URL. Source: <https://creators.vrchat.com/worlds/udon/string-loading/>. **[Docs]**

## API

```csharp
static IVRCStringDownload LoadUrl(VRCUrl url, IUdonEventReceiver receiver);
```

The receiver is typically `(IUdonEventReceiver)this` from inside an `UdonSharpBehaviour`.

## Callbacks

```csharp
public override void OnStringLoadSuccess(IVRCStringDownload result) {
    string content = result.Result;
    // also accessible: result.ResultBytes for raw bytes
}

public override void OnStringLoadError(IVRCStringDownload result) {
    Debug.LogError($"failed: {result.Error}: {result.ErrorMessage}");
}
```

## `IVRCStringDownload`

- `string Result` — the downloaded text (decoded as UTF-8 by default).
- `byte[] ResultBytes` — raw bytes for custom decoding.
- `VRCStringDownloadError Error` — error enum.
- `string ErrorMessage` — human-readable detail.
- `VRCUrl Url` — the URL that was requested.

Exact field names may vary across SDK versions. **[Unverified]**

## Limits

| Limit | Value |
|---|---|
| Max content size | 100 MB per string |
| Queue capacity | 1000 items |
| Rate limit | 1 string every 5 seconds |

**[Docs]**

## Trusted hosts

Default allowlist: GitHub, Pastebin, VRCDN, DisBridge, GitHub Gist. See [Trusted URLs](trusted-urls.md) for the full list and untrusted-URL behaviour.

## Patterns

### Load a JSON config at world load

```csharp
public VRCUrl configUrl;
DataDictionary _config;

void Start() {
    VRCStringDownloader.LoadUrl(configUrl, (IUdonEventReceiver)this);
}

public override void OnStringLoadSuccess(IVRCStringDownload result) {
    if (VRCJson.TryDeserializeFromJson(result.Result, out DataToken tok)
        && tok.TokenType == TokenType.Dictionary) {
        _config = tok.DataDictionary;
        ApplyConfig();
    }
}

public override void OnStringLoadError(IVRCStringDownload result) {
    Debug.LogError($"config load failed: {result.ErrorMessage}");
}
```

### Periodic refresh

```csharp
void Start() {
    SendCustomEventDelayedSeconds(nameof(_Refresh), 5f);
}
public void _Refresh() {
    VRCStringDownloader.LoadUrl(url, (IUdonEventReceiver)this);
    SendCustomEventDelayedSeconds(nameof(_Refresh), 60f);   // every minute
}
```

(Mind the 5-second rate limit shared across all string downloads on this client.)

## Caveats

- **Rate limit is per-user, not per-URL.** Two `LoadUrl` calls within 5 s will queue.
- **Queue at 1000 items.** Don't burst-load thousands of URLs.
- **No partial downloads, no streaming.** You get the whole string in one callback.
- **Network failures are not exceptions.** Use the error callback.

## Community patterns

- **Config from GitHub Gist.** Publish a JSON gist via `gist.githubusercontent.com/<user>/<id>/raw`; world fetches on join, parses with `VRCJson.TryDeserializeFromJson`. The dominant pattern. **[Community]**
- **News tickers / leaderboards via Pastebin or GitHub Pages.** For dynamic data, back a `*.github.io` page with CI builds, or use a Cloudflare Worker behind a `*.vrcdn.cloud` or `disbridge.com` proxy.
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
