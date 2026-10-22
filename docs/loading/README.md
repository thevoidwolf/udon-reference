# Loading

Bringing external data into a world at runtime. Three subsystems, each with its own trusted-host allowlist and a **one-request-per-5-seconds** rate limit (strings and images queue excess requests in random order; video players allow one new URL per user every 5 s across all players). **[Docs]**

| Subsystem | Page |
|---|---|
| Text/JSON | [String Loading](string-loading.md) |
| Image | [Graphics → Image Downloader](../graphics/image-downloader.md) |
| Video | [Video](../video/README.md) |

And the cross-cutting [Trusted URLs](trusted-urls.md) policy.

## Common shape

Every loader follows the same pattern:

1. Construct a downloader object (or call a static method on one).
2. Call `Download*` / `Load*` passing a `VRCUrl`, a receiver (an `UdonBehaviour` or `IUdonEventReceiver`), and any per-call options.
3. Receive `On*LoadSuccess(IVRC*Download)` and `On*LoadError(IVRC*Download)` callbacks on the receiver (strings: `OnStringLoadSuccess`/`OnStringLoadError`; images: `OnImageLoadSuccess`/`OnImageLoadError`). Video players use their own event set (`OnVideoReady`, `OnVideoError`, …).
4. Read the result off the success-payload object.

Since SDK 3.10.4 `UdonSharpBehaviour` implements `IUdonEventReceiver`, so `this` can be passed as the receiver without a cast. **[Docs]** (release notes)

String and image downloaders **don't follow HTTP redirects** (image loading: **[Docs]**; strings: **[Community]**, see [String Loading → Limits](string-loading.md#limits)). Video players resolve redirects, but the final URL must be allowlisted. **[Docs]**

There is no synchronous "load now" path. Everything is asynchronous.

## See also

- [Trusted URLs](trusted-urls.md)
- [Graphics → Image Downloader](../graphics/image-downloader.md)
- [Video](../video/README.md)
- Official: [String Loading](https://creators.vrchat.com/worlds/udon/string-loading/), [Image Loading](https://creators.vrchat.com/worlds/udon/image-loading/), [External URLs](https://creators.vrchat.com/worlds/udon/external-urls/)
