# API Methods

Concrete (non-virtual) methods and properties declared on `UdonSharpBehaviour`. Source: `UdonSharp/Runtime/UdonSharpBehaviour.cs` (bundled in the Worlds SDK since 3.4.0; see [README → Source](README.md#source)).

---

## Custom-event dispatch

### `SendCustomEvent`

```csharp
public void SendCustomEvent(string eventName)
```

Invokes a public no-arg method on this behaviour by name. Return values, if any, are discarded.

**Behaviour:**
- The target method must be `public` and take **zero parameters**.
- If no method with that name exists, the call is a silent no-op.
- In the Editor, a stack-frame check guards against animation events on proxies — this prevents Unity's animation system from calling proxy methods erroneously. **[Source]**

**Example:**
```csharp
public void DoTheThing() { /* ... */ }

void Start() {
    SendCustomEvent(nameof(DoTheThing));   // same as calling DoTheThing() directly
}
```

Prefer direct method calls in U#. `SendCustomEvent` is most useful for:
- UI events configured in the inspector ("On Click" → method name).
- Animation events that need to land on an UdonBehaviour.
- Cross-behaviour calls when you only have an `UdonBehaviour` reference (not a typed reference).

### `SendCustomNetworkEvent`

```csharp
public void SendCustomNetworkEvent(NetworkEventTarget target, string eventName)
```

Networked variant. The target method is invoked on every client identified by `target`. **[Source]**

**(SDK 3.8.1+) Parameters.** Up to **8** arguments of any syncable type can follow `eventName`; the receiver must be marked `[NetworkCallable]`. The official docs show both call forms: **[Docs]**

```csharp
using VRC.SDK3.UdonNetworkCalling;
using VRC.Udon.Common.Interfaces;

NetworkCalling.SendCustomNetworkEvent(this, NetworkEventTarget.All, nameof(PrintMessage), "VRCat", 11);
this.SendCustomNetworkEvent(NetworkEventTarget.All, nameof(PrintMessage), "VRCat", 11);

[NetworkCallable]
public void PrintMessage(string name, int age) { /* ... */ }
```

The `this.` prefix in the docs suggests the parameterised overloads are extension methods rather than members of `UdonSharpBehaviour`; their exact declared signatures aren't documented. **[Unverified]** Argument types must match the declared parameters exactly or sending fails; `null` arrives as `default(T)`. **[Docs]**

**Rules (receiving method):** **[Docs]**
- Must be `public`; not `static`, `virtual` or `override`; not overloaded; no return value. With parameters: max 8, syncable types only, no `params`, no default values.
- Should carry [`[NetworkCallable]`](../attributes/network-callable.md) (optionally `[NetworkCallable(maxEventsPerSecond: X)]`, 1–100, **default 5/s**). Parameterless public methods without it still work as **legacy events**, which use GameObject-wide (`SendMessage`-like) targeting instead of per-component targeting.
- **Methods whose name begins with `_` are not callable over the network** — *unless* they carry `[NetworkCallable]`, which overrides the underscore rule.
- The **receiving behaviour's sync mode must not be `None`**.
- Events over the rate limit are **queued** locally (not dropped), and order is preserved per sender, except that an event held back by its own rate limit can be overtaken by other events. The server may drop events only in edge cases (e.g. mixed world versions with stricter limits). Total parameter data per event is capped at 16 KB.

`NetworkEventTarget` enum (`VRC.Udon.Common.Interfaces.NetworkEventTarget`): **[Docs]**
| Value | Effect |
|---|---|
| `All` | All players in the instance (the sender runs it immediately, locally). |
| `Others` | All players except the sender. **(SDK 3.8.1+)** |
| `Owner` | The current owner of this GameObject. |
| `Self` | Loopback — only the sender; bypasses rate limiting. **(SDK 3.8.1+)** |

Inside the receiver, `NetworkCalling.CallingPlayer` identifies the sender — see [Networking events](networking-events.md#receiving-custom-network-events).

**[Source]** stubs the call locally (which is enough for the U# editor; the real VRChat runtime injects the network dispatch). Full model: [Networking → custom network events](../networking/custom-network-events.md).

### `SendCustomEventDelayedSeconds`

```csharp
public void SendCustomEventDelayedSeconds(
    string eventName,
    float delaySeconds,
    VRC.Udon.Common.Enums.EventTiming eventTiming = VRC.Udon.Common.Enums.EventTiming.Update)
```

Schedules `eventName` to fire after at least `delaySeconds` of real time.

- `delaySeconds = 0` fires in the same frame *or* the next frame, depending on when you call it (see the frame-count note below). **[Docs]**
- `eventTiming` selects which frame phase to fire on: `Update` (default), `LateUpdate`, and **(SDK 3.10.2+)** `FixedUpdate` and `PostLateUpdate`. **[Docs]**

**[Source]** — the body is a stub; the runtime fills it in.

There is no way to cancel a pending delayed event. **(SDK 3.10.4+)** `VRCTween.DelayedCall` behaves like `SendCustomEventDelayedSeconds` but returns a handle you can `Kill()`. **[Docs]** <https://creators.vrchat.com/worlds/udon/vrctween/virtual-tweens/#delayedcall>

### `SendCustomEventDelayedFrames`

```csharp
public void SendCustomEventDelayedFrames(
    string eventName,
    int delayFrames,
    VRC.Udon.Common.Enums.EventTiming eventTiming = VRC.Udon.Common.Enums.EventTiming.Update)
```

Same shape but frame-based. Minimum delay is 1 frame; `delayFrames` of 0 and 1 behave the same. **[Source]** **[Docs]** Unity's frame count advances at `Update`, so if you call this *before* `Update` (e.g. from `Start` or an input event) the delay can be one frame shorter than expected. **[Docs]**

---

## Network serialization

### `RequestSerialization`

```csharp
public void RequestSerialization()
```

Requests that this behaviour's synced fields be packed and broadcast.

**Constraints:**
- Only effective on behaviours whose `BehaviourSyncMode` is `Manual`. **[Docs]** **[Source]**
- The caller must be the owner of the GameObject (`Networking.IsOwner(gameObject)`). **[Source]**
- After the broadcast attempt, `OnPostSerialization(SerializationResult)` fires locally — check `result.success` to detect oversize packets.

**[Source]** stub.

See [Networking → serialization](../networking/serialization.md) for the packet model.

---

## Variable reflection (workaround for absent C# reflection)

Udon does not allow `System.Reflection` at runtime. These two methods give you the equivalent of `obj.GetType().GetField(name).GetValue(obj)` for the common case.

### `GetProgramVariable`

```csharp
public object GetProgramVariable(string name)
```

Returns the value of the field named `name` on this behaviour, or `null` if no such field exists. Searches public **and non-public** instance fields. **[Source]**

### `SetProgramVariable`

```csharp
public void SetProgramVariable(string name, object value)
```

Sets the field. If the field has `[FieldChangeCallback("PropertyName")]`, the property's setter is invoked **instead of** writing the field directly — the setter is responsible for writing the underlying field. See [`[FieldChangeCallback]`](../attributes/field-change-callback.md). **[Source]**

**Important gotchas:**
- Silently no-ops if `name` doesn't match a field. No exception.
- Performs no type validation. Writing a `string` to an `int` field will crash the Udon VM. Verify the type before calling.

```csharp
otherBehaviour.SetProgramVariable("counter", 42);
int? counter = (int?)otherBehaviour.GetProgramVariable("counter");
```

---

## Interaction properties

### `DisableInteractive`

```csharp
public bool DisableInteractive { get; set; }
```

When `true`, the `Interact()` event will not fire on this behaviour and the on-hover outline (Quest / desktop) is suppressed. Useful for temporarily disabling an interaction without removing the component. **[Source]**

### `InteractionText`

```csharp
public string InteractionText { get; set; }
```

The text shown in the interact tooltip when a player aims at this object. Default is "Use". **[Source]**

---

## Component state (SDK 3.10.2+)

```csharp
public bool enabled { get; set; }            // inherited from UnityEngine.Behaviour
public bool isActiveAndEnabled { get; }
```

Allowlisted in U# and Graph for everything deriving from `UnityEngine.Behaviour`, including U# behaviours. Does **not** work through a field typed as `Behaviour` that points at an `UdonSharpBehaviour` — type the field as your U# class. **[Docs]** (3.10.2 release notes)

---

## Obsolete / removed members

```csharp
[Obsolete("This method is obsolete, use Object.Instantiate(gameObject) instead")]
protected static GameObject VRCInstantiate(GameObject original)
```

Calls `Object.Instantiate` internally. Use `Instantiate` directly. **[Source]**

```csharp
[Obsolete(error: true)] public virtual void OnStationEntered()
[Obsolete(error: true)] public virtual void OnStationExited()
[Obsolete(error: true)] public virtual void OnOwnershipTransferred()
```

These parameterless legacy event signatures are compile errors. Use the `VRCPlayerApi`-parameter variants. **[Source]**

## See also

- [Type identity](type-identity.md) — `GetUdonTypeID` / `GetUdonTypeName`.
- [Networking → custom network events](../networking/custom-network-events.md) — the full SendCustomNetworkEvent model.
- [Networking events](networking-events.md) — `[NetworkCallable]` receivers and `NetworkCalling.CallingPlayer`.
- [Attributes → FieldChangeCallback](../attributes/field-change-callback.md) — how SetProgramVariable interacts with property setters.
