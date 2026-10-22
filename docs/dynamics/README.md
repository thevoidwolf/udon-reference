# Dynamics (PhysBones, Contacts, Constraints)

Since **Worlds SDK 3.10.0** (Oct 24, 2025), the three VRChat Dynamics systems that used to be avatar-only ship with world projects: **PhysBones**, **Contacts** and **VRC Constraints**. Each has Udon APIs and, for PhysBones and Contacts, Udon events. **[Docs]**

Source: <https://creators.vrchat.com/common-components/> · release notes <https://creators.vrchat.com/releases/release-3-10-0>. **[Docs]**

## Pages

| Page | Covers |
|---|---|
| [PhysBones](physbones.md) | `VRCPhysBone`, `VRCPhysBoneCollider`, `VRCPhysBoneRoot`, grab/pose events, runtime config, global colliders. |
| [Contacts](contacts.md) | `VRCContactSender`, `VRCContactReceiver`, `OnContactEnter`/`OnContactExit`, contact proxies, tags, box shapes. |
| [Constraints](constraints.md) | The six `VRC*Constraint` types, `Sources`, freeze-to-world, at-rest/offset values. |

## Namespaces

| Types | Namespace |
|---|---|
| `VRCPhysBone`, `VRCPhysBoneCollider` | `VRC.SDK3.Dynamics.PhysBone.Components` **[Docs]** |
| `VRCContactSender`, `VRCContactReceiver` | `VRC.SDK3.Dynamics.Contact.Components` **[Docs]** |
| `ContactEnterInfo`, `ContactExitInfo` | `VRC.Dynamics` **[Docs]** |
| `VRCAimConstraint`, `VRCLookAtConstraint`, `VRCParentConstraint`, `VRCPositionConstraint`, `VRCRotationConstraint`, `VRCScaleConstraint` | `VRC.SDK3.Dynamics.Constraint.Components` **[Docs]** |
| `VRCConstraintSource`, `VRCConstraintSourceKeyableList`, `VRCConstraintBase.WorldUpType` | `VRC.Dynamics` **[Docs]** |

The namespaces of `PhysBoneGrabbedInfo` (and the other PhysBone info structs), `ContactSenderProxy` / `ContactReceiverProxy`, `DynamicsUsage` and `DynamicsUsageFlags` are not stated in the docs. **[Unverified]**

## Rules that apply to all three

- **Change config via Udon, not animators.** In worlds, VRChat tells you to drive these components from Udon scripts rather than Animation Clips. **[Docs]**
- **Call `ApplyConfigurationChanges()` after writing properties.** PhysBones, PhysBone colliders, Contacts and Constraints all need it. Until you call it, the changes don't take effect. Batch your writes and call it once. **[Docs]**
- **Configuration changes are expensive.** The docs warn repeatedly against changing config often. Don't do it every frame. **[Docs]**
- **`name` is not exposed.** SDK 3.10.0 removed `name` from the Udon symbols for PhysBones, Contacts and VRC Constraints. Use `.gameObject.name` instead. **[Docs]**
- **They work on `VRCPlayerObject`.** Since 3.10.0, all three behave properly on a [Player Object](../persistence/player-objects.md). **[Docs]**
- **Simulation is local.** Events and simulation run on each client. A method like `ReleaseGrabs()` affects only the local client, so use a [network event](../networking/custom-network-events.md) to apply it on every client. **[Docs]**
- **Instantiated PhysBones simulate but don't sync.** Since 3.10.0, PhysBones created with `Object.Instantiate()` simulate instead of staying rigid. Their grabs and poses do not sync between clients, the same way Udon behaviours on instantiated objects are unsynced. **[Docs]** See [Instantiate](../gameobject-lifecycle/instantiate.md).
- **Don't put a Constraint and a PhysBone on the same GameObject.** This causes execution-order issues. Put the constraint on the parent instead. **[Docs]**

## Version history

| SDK | Change |
|---|---|
| 3.10.0 (Oct 2025) | PhysBones, Contacts and VRC Constraints arrive in worlds with Udon APIs and events. `VRCContactReceiver.UpdateContentTypes()` "take[s] an array of strings instead of an `IEnumerable` of strings", which fixes Udon Graph compile errors. This breaks U# code that passed anything other than a `string[]` (see the [discrepancy note](contacts.md#updating-content-types-and-tags)). `PermissionFilter` gains new constructors, including a parameterless default. `name` is removed from the Udon symbols. Instantiated PhysBones now simulate. The components work on `VRCPlayerObject`. In ClientSim, `OnPhysBoneGrabbed` now receives the initiating player. **[Docs]** |
| 3.10.2 (Feb 2026) | `matchingTags` added to `ContactExitInfo`. `Utilities.IsValid()` now returns accurate results for Contact Sender/Receiver proxies. PhysBone roots can be multi-edited. Fixed jitter on bones resting on colliders. Fixed exceptions when you destroy a PhysBone that parents VRC Constraints. **[Docs]** |
| 3.10.4 (Jun 2026) | `VRCPhysBoneCollider` exposed to Udon: settable properties plus `ApplyConfigurationChanges()`. Contacts can be **box-shaped**. Avatars can mark extra PhysBone colliders as global, and those can collide with world PhysBones. **[Docs]** |

## Performance and limits

- The Quest hard limit on avatar PhysBones **does not apply to worlds**. You should still keep the count down if your world is on mobile. **[Docs]**
- One PhysBone component can affect **at most 256 transforms**, root included. Consider splitting chains above about 128 transforms. PhysBones are multi-threaded per component. **[Docs]**
- Contact shapes are capped at a **3 m radius** and **6 m** width/height/depth, measured after scaling. A contact can have at most **16 collision tags**. **[Docs]**
- Worlds have **no limit** on global PhysBone colliders. Avatars have a limit (see [PhysBones](physbones.md#global-colliders)). **[Docs]**
- Worlds have no performance ranking, so the constraint performance section on the official page (count, depth) is avatar-only. VRChat says VRC Constraints *may* perform better than Unity constraints in a world, and warns against overusing Unity constraints. **[Docs]**

## Testing in the editor

In Play mode, use the left and right mouse buttons in the Game view to grab and pose PhysBones. **[Docs]**

## See also

- [UdonSharpBehaviour](../udonsharp-behaviour/README.md): where event overrides live.
- [UdonSharpBehaviour → Dynamics events](../udonsharp-behaviour/dynamics-events.md): the event overrides in one place.
- [Persistence → Player Objects](../persistence/player-objects.md): dynamics work on `VRCPlayerObject` since 3.10.0.
- [Networking](../networking/README.md) and [Custom network events](../networking/custom-network-events.md): dynamics state is local.
- [GameObject Lifecycle → Instantiate](../gameobject-lifecycle/instantiate.md): instantiated PhysBones don't sync.
- [Players](../players/README.md): `VRCPlayerApi`, returned by contact proxies.
- [Tweening](../tweening/README.md): `VRCTween`, another 3.10.x feature.
