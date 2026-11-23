# SDK 3.11 Beta Preview (Unity 6)

> **Beta, not for production.** Everything on this page comes from **Worlds SDK 3.11.0-beta.1** (GitHub release Nov 19, 2026; release notes dated Nov 18, 2026). Beta APIs can change before release. The rest of this reference still describes the stable **3.10.6** SDK on Unity **2022.3.22f1**. When 3.11.0 ships stable, this page's content moves into the topic pages.

Sources: beta release notes <https://vrc-beta-docs.netlify.app/releases/release-3-11-0>, beta docs <https://vrc-beta-docs.netlify.app/>, and the `com.vrchat.worlds` / `com.vrchat.base` **3.11.0-beta.1** packages from <https://github.com/vrchat/packages/releases/tag/3.11.0-beta.1> (SHA-256 matched the VPM listing). Package contents were diffed against 3.10.6. **[Docs]** (beta) / **[Source]**

The beta notes' summary line says "SDK Version `3.10.11`". The package, the release title and the page heading all say **3.11.0**, so the summary line is a typo. **[Docs]** / **[Source]**

## Unity 6 migration

- 3.11.0-beta.1 targets **Unity 6000.0.67f1**. The package manifest's minimum Unity version is `"unity": "6000.0"`. **[Docs]** (beta) / **[Source]** (VPM manifest)
- Migration is **one-way**. Back up first, test on Unity 2022 before migrating, and know that once a world is uploaded from Unity 6 it **can't be uploaded from 2022 again**. Update the VCC first, because outdated versions can't upgrade projects. **[Docs]** (beta) <https://vrc-beta-docs.netlify.app/sdk/upgrade/unity-6000/>
- The SDK adds a **Unity 6 Upgrade Wizard** (VRChat SDK > Utilities > Upgrade Wizard). It opens automatically on first load. **[Docs]** (beta)

### Changes that affect Udon / U# code

| Change | What to do |
|---|---|
| `Rigidbody.velocity` renamed to `linearVelocity`. (Unity 6 also renames `drag` → `linearDamping` and `angularDrag` → `angularDamping`. VRChat's guide only calls out velocity.) | **U#**: Unity's API updater rewrites your `.cs` files. **Udon Graph**: run the Upgrade Wizard. **Hand-written Udon Assembly**: replace `UnityEngineRigidbody.__get_velocity` / `__set_velocity` with `__get_linearVelocity` / `__set_linearVelocity` yourself. **[Docs]** (beta) The 3.11 extern table adds the `linearVelocity`, `linearDamping` and `angularDamping` signatures. **[Source]** |
| Forces/torques with `ForceMode.Acceleration` or `ForceMode.VelocityChange` behave differently (`AddForce`, `AddTorque`, `AddRelativeForce`, `AddRelativeTorque`, `AddForceAtPosition`, `AddExplosionForce`). | Switch to `ForceMode.Force` and multiply by the rigidbody's mass. **Not auto-fixed.** The wizard's "Force and Torque Changes" step only *selects* affected U# scripts and graphs (U#'s new `Editor/UpgradeWizard/ForceUpgradeStep.cs` regex-scans the `.cs` source). **[Docs]** (beta) / **[Source]** |
| TextMeshPro uses linear colour (text looks darker) and has updated shaders. | Import the updated TMP package and convert each migrated scene's TMP colours (wizard, or VRChat SDK > Utilities > Convert Current Scene TextMeshPro Colors > To Gamma). **[Docs]** (beta) |
| Particle ring buffer "Loop Until Replaced" no longer exceeds Max Particles. | Raise Max Particles if emission stops. **[Docs]** (beta) |
| `NavMeshObstacle` on a `NavMeshAgent` behaves differently. | Remove the obstacle (wizard / SDK auto-fix) and use agent Priority for avoidance. **[Docs]** (beta) See [AI Navigation](../ai-navigation/README.md). |
| Legacy `OffMeshLink` members are gone from the Udon extern table (`autoUpdatePositions`, `biDirectional`, `costOverride`, `UpdatePositions`, `NavMeshAgent.offMeshLink`). New `NavMeshLinkInstance` helpers appear (`IsLinkValid`, `IsLinkActive`, `SetLinkActive`, `IsLinkOccupied`, `GetLinkOwner`, `SetLinkOwner`). | Move off-mesh-link logic to `NavMeshLink`. **[Source]** (extern diff; not in the beta notes) |

Other Unity 6 surface that newly appears in the extern table includes: `PhysicsMaterial` (the old `PhysicMaterial` type is still listed), Material variant APIs (`parent`, `IsChildOf`, `IsPropertyLocked`, `SetPropertyLock`, `RevertPropertyOverride`, …), more `Rigidbody2D` / `Collider2D` queries, and gamepad-speaker members. Whether each is usable and safe in the client isn't documented. **[Source]** / usability **[Unverified]**

## New Udon APIs

### Attaching transforms to bones

```csharp
player.AttachTransformToBone(Transform transform, HumanBodyBones bone);
player.DetachTransformFromBone(Transform transform);
```

- Keeps a world transform on a humanoid bone by updating its position and rotation every frame, **without reparenting** it. **[Docs]** (beta) <https://vrc-beta-docs.netlify.app/worlds/udon/players/player-positions/#attachtransformtobone>
- Runs every frame **right after PhysBones and Constraints**. **[Docs]** (beta)
- Attaching to a new bone on the same avatar detaches it from the old one. While the avatar lacks that bone (or isn't humanoid), the transform stops moving. Detaching a transform that isn't attached does nothing. **[Docs]** (beta)
- **Don't attach one transform to bones of different players at the same time.** The behaviour is undefined. **[Docs]** (beta)
- Extern signatures: `__AttachTransformToBone__UnityEngineTransform_UnityEngineHumanBodyBones__SystemVoid`, `__DetachTransformFromBone__UnityEngineTransform__SystemVoid`. **[Source]**

On 3.10.6, keep using the `GetBonePosition` / `GetBoneRotation` follower in `PostLateUpdate`. See [Tracking Data](../players/tracking-data.md).

### Platform detection

New `VRCPlayerApi` methods: `IsUserOnPC()`, `IsUserOnAndroid()`, `IsUserOnIOS()`, `IsUserOnMobile()`. **[Docs]** (beta) / **[Source]**

| Device | `IsUserInVR` | `IsUserOnPC` | `IsUserOnAndroid` | `IsUserOnIOS` | `IsUserOnMobile` |
|---|---|---|---|---|---|
| Desktop PC | false | true | false | false | false |
| PC VR headset | true | true | false | false | false |
| Standalone VR headset | true | false | true | false | false |
| Android phone / tablet | false | false | true | false | true |
| iPhone / iPad | false | false | false | true | true |

Table from the beta players page. **[Docs]** (beta) `IsUserOnMobile` means a handheld phone or tablet, not a Quest.

### Social Hooks

A new static class, `VRC.SDK3.SocialHooks` (extern type `VRCSDK3SocialHooks`). **[Docs]** (beta) <https://vrc-beta-docs.netlify.app/worlds/udon/social-hooks/> / **[Source]**

| Member | Returns | Notes |
|---|---|---|
| `GetLocalPlayerFriendBucket()` | `FriendBucket` | How many of the local player's friends are in the instance, as a bucket: `None` (0–1), `Few` (2–4), `Some` (5–9), `Many` (10+). Exact counts are never exposed. |
| `HasLocalPlayerFavoritedWorld()` | `bool` | Whether the world is in any of the local player's favourite lists. |
| `GetInstanceGroupId()` | `string` | Owning group ID in Group / Group+ / Group Public instances. Empty elsewhere. |
| `IsLocalPlayerInInstanceGroup()` | `bool` | `false` outside group instances. |
| `GetInstanceGroupInfo()` | `SocialHooks.GroupInfo` | `Name`, `MemberCount`, `BannerTexture`, `ShortCode`, `Description`, `Rules`. Returns `GroupInfo.Empty` outside group instances or while loading. |

New events on `UdonSharpBehaviour`:

```csharp
public override void OnLocalPlayerFriendBucketChanged(FriendBucket previous, FriendBucket current) { }
public override void OnLocalPlayerFriendRequestSent() { }      // doesn't say to whom
public override void OnLocalPlayerFriendRequestAccepted() { }  // doesn't say whose
public override void OnLocalPlayerFavoritedWorld() { }         // at most once per session
```

- **Privacy:** players can turn off "Allow Worlds To Count Friends In Instance". Worlds can't tell that apart from having no friends present. Bucket changes are delayed and coalesced, so you can't match them to one player joining. **[Docs]** (beta)
- In U# 3.11.0-beta.1 these four stubs sit behind `#if VRC_ENABLE_SOCIAL_HOOKS` in `Runtime/UdonSharpBehaviour.cs`. That define is referenced from the SDK's `VRCCore-Editor.dll`, which suggests the SDK sets it for you. **[Source]** / auto-set **[Unverified]**
- ClientSim's settings window gains **Has Favorited World** and **Friend Bucket**, both changeable during Play mode to fire the events. **[Source]** (`Integrations/ClientSim/Editor/Windows/ClientSimSettingsWindow.cs`)

## Other changes

- **PhysBone colliders:** `VRCPhysBoneCollider` gains **Global Ignored Phys Bones** (field `globalIgnoredPhysBones`). A global collider skips the listed PhysBones, unless that PhysBone also lists the collider directly. **[Docs]** (beta) No Udon getter/setter for it appears in the beta extern table. **[Source]**
- **Contacts:** world-project Receivers now show the Material Contacts ("write to materials") options in the inspector. Worlds already supported this. Faked player IDs are now assigned to contacts incrementally. A rare case where relative velocity read as zero is fixed. **[Docs]** (beta) See [Contacts](../dynamics/contacts.md).
- **UdonSharp:** relational operators on **Unity/VRChat enums** are fixed. Through 3.10.6 they compiled as `!=` (see [Supported features → Types](../compiler/supported-features.md#types)). The U# menu items get explicit priorities, Harmony patches are updated for Unity 6 editor APIs, and `SendCustomEventDelayedSeconds/Frames` doc comments now state that `0` means the next frame. **[Source]** (3.10.6 vs 3.11.0-beta.1 diff)
- **SDK:** Package Maker moves into the base SDK. Harmony 2.4.3 (Apple Silicon). The help buttons on PhysBones, Contacts and Constraints link to the docs. The SDK says when uploads are restricted by account standing. **[Docs]** (beta)

## See also

- [Getting Started → Installation](../getting-started/installation.md#requirements): Unity version guidance.
- [Official Docs](official-docs.md): beta docs and package links.
- [Source Map](source-map.md): how to diff SDK packages yourself.
