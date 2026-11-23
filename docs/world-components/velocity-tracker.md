# VRCVelocityTracker

`VRCVelocityTracker` measures how fast a GameObject is moving and turning, and lets Udon read the result. It shipped in **Worlds SDK 3.10.6** (Oct 30, 2026). **[Docs]**

Sources: <https://creators.vrchat.com/worlds/components/vrc_velocitytracker/> · release notes <https://creators.vrchat.com/releases/release-3-10-6>. **[Docs]**

## Why use it

The usual way to get the speed of an object without a `Rigidbody` is to store its position every frame and divide the change by `Time.deltaTime`. That needs an Udon `Update` running on every tracked object. The tracker does the measuring natively and runs no update loop of its own. Your script only pays when it reads a value. **[Docs]**

This matters for objects moved by animation, constraints, `VRCBillboard`, player bones or script. Those have no physics velocity to read.

## Setup

Add the **VRC Velocity Tracker** component to the GameObject you want to measure. Then reference it from a script and call one of the getters. **[Docs]**

The type is `VRC.SDK3.Components.VRCVelocityTracker`. The official page doesn't give the namespace. It follows from the extern type name `VRCSDK3ComponentsVRCVelocityTracker`. **[Source]** `Runtime/Udon/External/VRC.Udon.Wrapper.dll` (com.vrchat.worlds 3.10.6)

## Methods

All four take no arguments and return a `Vector3`. **[Docs]** / **[Source]** (extern table)

| Method | Returns | Measured over |
|---|---|---|
| `GetLinearVelocity()` | Velocity in metres per second | The last rendered frame |
| `GetAngularVelocity()` | Angular velocity in radians per second | The last rendered frame |
| `GetFixedLinearVelocity()` | Velocity in metres per second | The last physics step |
| `GetFixedAngularVelocity()` | Angular velocity in radians per second | The last physics step |

All four return `Vector3.zero` while the component is disabled. **[Docs]**

Use the frame-based pair for visuals and anything read in `Update`, `LateUpdate` or an input event. Use the `Fixed` pair inside `FixedUpdate` or when you feed the value into physics. That pairing is a rule of thumb from how the two values are sampled. VRChat doesn't state it.

The docs list no settings and no reset method. The only tracker-specific externs that 3.10.6 added are these four getters, plus the usual inherited `Component` members. **[Docs]** / **[Source]** (3.10.6 extern table)

## Things the docs don't say

- **Coordinate space.** The page doesn't say whether the vectors are in world or local space. World space is the likely reading, because the values are described as plain m/s and rad/s with no reference frame. Test before relying on direction. **[Unverified]**
- **Timing.** A comment in ClientSim's `ClientSimBillboardHelper.cs` says the tracker runs at execution order **31001**, after the `PostLateUpdate` driver (31000). **[Source]** `Packages/com.vrchat.worlds/Integrations/ClientSim/Runtime/Helpers/ClientSimBillboardHelper.cs` If that holds in the client too, a value read during `Update` describes movement up to the end of the *previous* frame. That matches "measured over the last rendered frame". **[Unverified]**
- **Networking.** The tracker measures what the local client sees. For a remote player's object driven by `VRCObjectSync`, you get the smoothed, interpolated motion on your screen, not the owner's raw input. **[Unverified]**

## Example (U#)

A drum that plays louder the faster the stick hits it. The stick can be held, animated or constraint-driven. Nothing runs per frame; the velocity is read only when a hit happens.

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDK3.Components;

public class Drum : UdonSharpBehaviour
{
    public VRCVelocityTracker stickTracker;  // on the drumstick
    public Collider stickTip;
    public AudioSource hitSound;
    public float fullVolumeSpeed = 5f;       // m/s for a full-volume hit

    void OnTriggerEnter(Collider other)
    {
        if (other != stickTip) return;

        float speed = stickTracker.GetLinearVelocity().magnitude;
        hitSound.volume = Mathf.Clamp01(speed / fullVolumeSpeed);
        hitSound.Play();
    }
}
```

`OnTriggerEnter` needs a `Rigidbody` on one of the two colliders. A `VRCPickup` drumstick already has one. Players and drones never fire this Unity event. See [Lifecycle events](../udonsharp-behaviour/lifecycle-events.md). **[Docs]**

## See also

- [World Components overview](README.md)
- [Billboard](billboard.md): the other 3.10.6 component that replaces an Udon update loop.
- [Players → Pickups & Stations](../players/pickups-stations.md): pickups and their throw settings.
- [Networking → Object sync](../networking/object-sync.md): what remote motion looks like locally.
