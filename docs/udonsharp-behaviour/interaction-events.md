# Interaction Events

VRChat exposes a built-in "Use" interaction (E-key on desktop, trigger on VR) targeted at whatever the local player is aiming at. This page covers the interaction event itself and the pickup-event family.

Source: `Runtime/UdonSharpBehaviour.cs`. **[Source]**

---

## `Interact`

```csharp
public virtual void Interact()
```

Fires on the local client only when the local player presses Use while aimed at this GameObject. The GameObject needs both a `Collider` and an `UdonBehaviour`. **[Docs]** For 2D UI, use a `VRC Ui Shape` + `Button` instead of `Interact`. **[Docs]**

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

Text shown in the interact tooltip when a player aims at this object. Default `"Use"`. **[Source]**

### Interact inspector options

When a U# class implements `Interact()`, its inspector shows options that control how the interaction is displayed. They work the same way as the matching [VRCPickup](../players/pickups-stations.md#vrcpickup-inspector-fields-and-udon-properties) fields. **[Docs]** <https://creators.vrchat.com/worlds/udon/graph/event-nodes#interact>

| Inspector field | Notes |
|---|---|
| Interaction Text | Tooltip text (`InteractionText` above). **[Docs]** |
| Proximity | Maximum interaction distance. **[Docs]** |
| Outline Renderers | **SDK 3.10.6+.** Which child renderers get the hover outline. Useful when the object contains "hidden" meshes that shouldn't glow. **[Docs]** The U# inspector writes the list to the backing `UdonBehaviour.OutlineRenderers`. **[Source]** (`Editor/Editors/UdonSharpGUI.cs`) |

From client **2026.4.1**, an interactable or pickup with an **empty** Outline Renderers list no longer auto-outlines `SkinnedMeshRenderer`s. List them explicitly if they should glow. ClientSim in 3.10.6 behaves the same way. **[Docs]** <https://creators.vrchat.com/releases/release-3-10-6> / ClientSim **[Source]**

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

On Desktop, the two Use events require the pickup's **Auto Hold** option. **[Docs]** SDK 3.9.0 added a new Auto Hold mode that works on all input devices; existing pickups must be upgraded in the inspector to use it. **[Docs]**

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

Related `VRCPickup` changes: adding the component auto-moves the GameObject to the `Pickup` layer again (SDK 3.10.4+). An **Outline Renderers** property selects which renderers get the hover highlight (SDK 3.10.5+; plain interactables got it in 3.10.6). The official pickup page now documents `Drop(VRCPlayerApi)`, `GenerateHapticEvent(...)` and `PlayHaptics()` too. See [Pickups & Stations](../players/pickups-stations.md#vrcpickup-runtime-api). **[Docs]**

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
