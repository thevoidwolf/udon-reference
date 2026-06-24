# Loading

Bringing external data into a world at runtime. Three subsystems, all with rate limits and trusted-host gates:

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
3. Receive `On*LoadSuccess(IVRC*Download)` and `On*LoadError(IVRC*Download)` callbacks on the receiver.
4. Read the result off the success-payload object.

There is no synchronous "load now" path. Everything is asynchronous.

## See also

- [Trusted URLs](trusted-urls.md)
- [Graphics → Image Downloader](../graphics/image-downloader.md)
- [Video](../video/README.md)
