# `[NetworkCallable]` (SDK 3.8.1+)

Marks a U# method as callable over the network with `SendCustomNetworkEvent`, optionally with up to 8 parameters and a per-event rate limit. Added in Worlds SDK 3.8.1 (May 22, 2025). **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/events/>

Unlike the other attributes in this section, `[NetworkCallable]` is not defined in `UdonSharpAttributes.cs`. It ships with the VRChat Worlds SDK in the `VRC.SDK3.UdonNetworkCalling` namespace (the official examples import it from there). **[Docs]** It is absent from the public UdonSharp repo, which predates it. **[Source]**

## Usage

```csharp
using VRC.SDK3.UdonNetworkCalling;
using VRC.Udon.Common.Interfaces;

[NetworkCallable]                              // default: 5 events/s
public void PrintMessage(string name, int age) { }

[NetworkCallable(maxEventsPerSecond: 1)]       // any integer 1..100
public void ResetGame() { }
```

The only documented argument is `maxEventsPerSecond`. **[Docs]** The full attribute class declaration is not published, so it isn't reproduced here.

## Requirements on the method

All items in this section: **[Docs]**.

- `public`. Not `static`, `virtual` or `override`.
- Returns `void`.
- No member overloading. No other method on the class may share the name, including ones without the attribute.
- At most **8** parameters, each of a [synced-variable type](https://creators.vrchat.com/worlds/udon/networking/network-details#synced-variables). No `params`, no default values.
- The receiving behaviour must not use `BehaviourSyncMode.None`.

## Effects

- **Allows network calls.** Without the attribute, U# methods can only be called as parameterless *legacy* events. **[Docs]**
- **Overrides the underscore rule.** A method named `_Foo` is normally never network-callable. With `[NetworkCallable]` it *is*. Leave the attribute off methods you want to keep local-only. **[Docs]**
- **Rate limit.** Per-event limit, default **5/s**, max **100/s**. Calls above the limit are queued on the sender. Calls with more than 1024 bytes of parameters count as several events. Official guidance is to set the limit as low as you can. **[Docs]**
- **Component targeting.** A call reaches only the targeted `UdonBehaviour`, not every `UdonBehaviour` on the GameObject (legacy events reach all of them). **[Docs]**

## Udon Graph equivalent

The `Custom Event` node has a parameter-count dropdown and a `MaxEventsPerSecond` field. Setting `MaxEventsPerSecond` to `0` on a parameterless node makes it a legacy event. **[Docs]**

## See also

- [Networking → Custom Network Events](../networking/custom-network-events.md) — targets, size limits, `NetworkCalling.CallingPlayer`, queue monitoring.
- [`[UdonBehaviourSyncMode]`](behaviour-sync-mode.md) — `None` blocks receiving events.
