# Interaction Events

VRChat exposes a built-in "Use" interaction (E-key on desktop, trigger on VR) targeted at whatever the local player is aiming at. This page covers the interaction event itself and the pickup-event family.

Source: `Runtime/UdonSharpBehaviour.cs`. **[Source]**

---

## `Interact`

```csharp
public virtual void Interact()
```

Fires on the local client only when the local player presses Use while aimed at this GameObject. The GameObject must have a `Collider` (non-trigger) for the raycast to hit.

```csharp
public override void Interact() {
    transform.Rotate(0, 90f, 0);
}
```

### `DisableInteractive`

```csharp
public bool DisableInteractive { get; set; }
```

When `true`, `Interact` does not fire and the on-hover outline disappears. Useful for temporarily disabling interaction without removing the component. **[Source]**

### `InteractionText`

```csharp
public string InteractionText { get; set; }
```

Tooltip shown to other players when they aim at this object. Default `"Use"`. **[Source]**

You can also configure the interaction text and proximity in the inspector via the `UdonBehaviour` component fields ("Interaction Text", "Proximity").

---

## Pickup events

A pickup is a GameObject with a `VRC.SDK3.Components.VRCPickup` component. Pickups generate four events on UdonBehaviours attached to the same GameObject:

```csharp
public virtual void OnPickup()
public virtual void OnDrop()
public virtual void OnPickupUseDown()
public virtual void OnPickupUseUp()
```

| Event | Fires |
|---|---|
| `OnPickup` | Local player picks up this pickup |
| `OnDrop` | Local player drops it (intentional release) |
| `OnPickupUseDown` | "Use" pressed while holding |
| `OnPickupUseUp` | "Use" released while holding |

All four are **local-only**. Other clients see only what gets networked through `VRCObjectSync` or your own `[UdonSynced]` state. **[Source]**

### Reading the held hand

The `VRCPickup` component has a `currentHand` property (`VRC_Pickup.PickupHand` enum: `None`, `Left`, `Right`) and a `currentPlayer` (`VRCPlayerApi`) you can read inside the event.

```csharp
VRCPickup _pickup;

void Start() { _pickup = (VRCPickup)GetComponent(typeof(VRCPickup)); }

public override void OnPickupUseDown() {
    if (_pickup.currentHand == VRC_Pickup.PickupHand.Right) {
        FireRight();
    }
}
```

### Networking a pickup-triggered action

To make a pickup action visible to other players, combine the local event with `SendCustomNetworkEvent`:

```csharp
public override void OnPickupUseDown() {
    SendCustomNetworkEvent(NetworkEventTarget.All, nameof(Fire));
}
public void Fire() { /* fires for everyone */ }
```

For shared pickup *physics*, attach `VRCObjectSync` to the same GameObject — see [Networking → ObjectSync](../networking/object-sync.md).

---

## Pattern: interactable that requires ownership

```csharp
public override void Interact() {
    if (!Networking.IsOwner(gameObject))
        Networking.SetOwner(Networking.LocalPlayer, gameObject);

    // Now safe to write to [UdonSynced] fields and call RequestSerialization.
}
```

Without ownership, your synced writes will not propagate. See [Networking → ownership](../networking/ownership.md).

## See also

- [Player events](player-events.md) — trigger / collision callbacks.
- [Input events](input-events.md) — raw button / axis input regardless of where the player is aiming.
- [Networking → object sync](../networking/object-sync.md) — networking pickup physics.
