# Tracking Data

Position and rotation of a player's head, hands, root, and avatar root.

## API

```csharp
VRCPlayerApi.TrackingData GetTrackingData(VRCPlayerApi.TrackingDataType target);
```

### `TrackingDataType` enum

| Value | Source |
|---|---|
| `Head` | Headset / camera point of view. |
| `LeftHand` | Left controller / animated hand. |
| `RightHand` | Right controller / animated hand. |
| `Origin` | The player's local play-space origin (room-scale). |
| `AvatarRoot` | The avatar's hip / root bone. |

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

Returns `Vector3.zero` / `Quaternion.identity` if the avatar lacks that bone (e.g. a generic-rig avatar with no hand bones).

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

- **Local player only for high fidelity.** Tracking and bone data for remote players is replicated at lower fidelity. Hand rotation in particular is unreliable on remote players. **[Community]** <https://feedback.vrchat.com/udon/p/remote-playerapi-gettrackingdata-and-getbonepositionrotationtransform-return-wro>
- **`Origin` is room-scale.** In VR, the player can be standing far from `Origin` if they've walked physically. Don't conflate it with `Head` or with avatar root.
- **Bone data is only meaningful for humanoid avatars.** Generic avatars return defaults.

## See also

- [VRCPlayerApi](vrc-player-api.md)
- [Voice & Audio](voice-audio.md)
