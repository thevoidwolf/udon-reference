# Tracking Data

Position and rotation of a player's head, hands, root, and avatar root.

Source: <https://creators.vrchat.com/worlds/udon/players/player-positions>. **[Docs]** unless noted. `GetTrackingData` is the officially suggested way to get head and hand position/rotation.

## API

```csharp
VRCPlayerApi.TrackingData GetTrackingData(VRCPlayerApi.TrackingDataType target);
```

### `TrackingDataType` enum

| Value | Source |
|---|---|
| `Head` | Local player: headset / camera tracking data. Remote players: the avatar's Head bone. |
| `LeftHand` | Local player: left controller tracking. Remote players: the LeftHand bone. |
| `RightHand` | Local player: right controller tracking. Remote players: the RightHand bone. |
| `Origin` | Centre of the **local VR** user's playspace. For local desktop users and **all remote** players it returns the player's position. |
| `AvatarRoot` | The avatar's root transform — the same transform the player capsule is attached to (not the hip bone). Does not rotate with head facing for full-body-tracking users. |

### `TrackingData` struct

```csharp
struct TrackingData {
    Vector3    position;
    Quaternion rotation;
}
```

## Bones (humanoid avatars)

```csharp
Vector3    GetBonePosition(HumanBodyBones bone);
Quaternion GetBoneRotation(HumanBodyBones bone);
```

`HumanBodyBones` is the Unity enum: `Head`, `LeftHand`, `RightHand`, `LeftFoot`, `RightFoot`, every finger joint, spine bones, etc.

Returns `Vector3.zero` / `Quaternion.identity` if the avatar lacks that bone (e.g. a generic-rig avatar with no hand bones). Avatars may not have bones where you expect — don't infer height or pose from bone data (use `GetAvatarEyeHeightAsMeters` for height). **[Docs]**

```csharp
Vector3 head = player.GetBonePosition(HumanBodyBones.Head);
Vector3 leftHand = player.GetBonePosition(HumanBodyBones.LeftHand);
```

## Patterns

### Object that follows a hand

```csharp
public VRCPlayerApi.TrackingDataType target = VRCPlayerApi.TrackingDataType.RightHand;
VRCPlayerApi _p;

void Start() { _p = Networking.LocalPlayer; }

void LateUpdate() {
    if (_p == null) return;                   // editor case
    var td = _p.GetTrackingData(target);
    transform.SetPositionAndRotation(td.position, td.rotation);
}
```

(From `Samples~/Utilities/TrackingDataFollower.cs` in the UdonSharp package.)

### Bone follower

```csharp
public HumanBodyBones bone = HumanBodyBones.Hips;
VRCPlayerApi _p;

void Start() { _p = Networking.LocalPlayer; }

void Update() {
    if (_p == null) return;
    transform.SetPositionAndRotation(_p.GetBonePosition(bone), _p.GetBoneRotation(bone));
}
```

(From `Samples~/Utilities/BoneFollower.cs`.)

## Caveats

- **Local player only for high fidelity.** Remote players' `Head`/`LeftHand`/`RightHand` tracking data comes from their avatar bones **[Docs]**, which are replicated at lower fidelity. Hand rotation in particular is unreliable on remote players. **[Community]** <https://feedback.vrchat.com/udon/p/remote-playerapi-gettrackingdata-and-getbonepositionrotationtransform-return-wro>
- **`Origin` is room-scale.** In VR, the player can be standing far from `Origin` if they've walked physically. Don't conflate it with `Head` or with avatar root.
- **Forward direction:** for a general "which way is the player facing", the docs suggest `GetRotation()` rather than `AvatarRoot`, which doesn't follow head facing for FBT users. **[Docs]**
- **Bone data is only meaningful for humanoid avatars.** Generic avatars return defaults.

## See also

- [VRCPlayerApi](vrc-player-api.md)
- [Voice & Audio](voice-audio.md)
