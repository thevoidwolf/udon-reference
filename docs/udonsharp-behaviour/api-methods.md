# API Methods

Concrete (non-virtual) methods and properties declared on `UdonSharpBehaviour`. Source: `Packages/com.vrchat.UdonSharp/Runtime/UdonSharpBehaviour.cs`.

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

Networked variant. The target method is invoked on every client identified by `target`.

**Rules:**
- The target method must be `public` and no-arg.
- **Methods whose name begins with `_` are non-networkable** and will not be invoked over the network. (This is the standard pattern for "private" helpers you don't want exposed via network.)
- Rate-limited by the VRChat client; spamming will silently drop events.

`NetworkEventTarget` enum (`VRC.Udon.Common.Interfaces.NetworkEventTarget`):
| Value | Effect |
|---|---|
| `All` | All players in the instance. |
| `Owner` | The current owner of this GameObject. |

**[Source]** stubs the call locally (which is enough for the U# editor; the real VRChat runtime injects the network dispatch).

### `SendCustomEventDelayedSeconds`

```csharp
public void SendCustomEventDelayedSeconds(
    string eventName,
    float delaySeconds,
    VRC.Udon.Common.Enums.EventTiming eventTiming = VRC.Udon.Common.Enums.EventTiming.Update)
```

Schedules `eventName` to fire after at least `delaySeconds` of real time.

- `delaySeconds = 0` fires **next frame**, not this frame.
- `eventTiming` selects which frame phase to fire on. Values: `Update` (default), `LateUpdate`.

**[Source]** — the body is a stub; the runtime fills it in.

### `SendCustomEventDelayedFrames`

```csharp
public void SendCustomEventDelayedFrames(
    string eventName,
    int delayFrames,
    VRC.Udon.Common.Enums.EventTiming eventTiming = VRC.Udon.Common.Enums.EventTiming.Update)
```

Same shape but frame-based. `delayFrames` of 0 and 1 both fire next frame.

---

## Network serialization

### `RequestSerialization`

```csharp
public void RequestSerialization()
```

Requests that this behaviour's synced fields be packed and broadcast.

**Constraints:**
- Only effective on behaviours whose `BehaviourSyncMode` is `Manual`. **[Docs]**
- The caller must be the owner of the GameObject (`Networking.IsOwner(gameObject)`).
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

Tooltip text displayed to other players when they aim at this object. Default is "Use". **[Source]**

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
- [Attributes → FieldChangeCallback](../attributes/field-change-callback.md) — how SetProgramVariable interacts with property setters.
