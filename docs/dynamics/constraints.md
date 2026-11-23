# VRC Constraints

VRChat's own constraint components move, rotate and scale a transform relative to other transforms. They mirror Unity's six built-in constraints. In worlds you can use VRC constraints, Unity constraints, or both. VRChat warns that too many Unity constraints can hurt performance and says VRC constraints may perform better. **[Docs]**

Sources: <https://creators.vrchat.com/common-components/constraints> and <https://creators.vrchat.com/common-components/constraints/constraints-api>. **[Docs]**

> Only use the members documented on the API page. VRChat says undocumented ones "might be removed from the public interface at any time." **[Docs]**

## Types

All in `VRC.SDK3.Dynamics.Constraint.Components`. **[Docs]**

| Type | Behaviour |
|---|---|
| `VRCAimConstraint` | Rotates the target to aim at the sources. The forward axis is configurable. |
| `VRCLookAtConstraint` | A simplified aim that keeps +Z facing the sources. |
| `VRCParentConstraint` | Moves and rotates as if parented to the sources. |
| `VRCPositionConstraint` | Matches the sources' position. |
| `VRCRotationConstraint` | Matches the sources' rotation. |
| `VRCScaleConstraint` | Matches the sources' scale. |

These supporting types are in `VRC.Dynamics`: `VRCConstraintSource` (`SourceTransform`, `Weight`) and `VRCConstraintSourceKeyableList`. **[Docs]**

## Common properties

| Property | Type | Notes |
|---|---|---|
| `IsActive` | `bool` | Whether the constraint is being evaluated. Same as Unity's `constraintActive`. **[Docs]** |
| `GlobalWeight` | `float` | Applied on top of each source's weight. Same as Unity's `weight`. **[Docs]** |
| `Locked` | `bool` | Always treated as locked in Play mode. **[Docs]** |
| `Sources` | `VRCConstraintSourceKeyableList` | Raw source list. Still exposed to Udon, but **use the source methods below instead** (since 3.10.6). **[Docs]** |
| `SourceCount` | `int` | Number of sources (3.10.6+). **[Docs]** |
| `TargetTransform` | `Transform` | `null` means the constraint's own transform. **Udon can't change it at runtime**, because it's cached. Toggle between several constraint components instead. **[Docs]** Since 3.10.6, constraints in an SDK project *do* react to `TargetTransform` changes made from editor C# while in Play mode. That is an editor convenience, not a runtime Udon feature. **[Docs]** |
| `SmoothingFactor` | `float` | Motion smoothing, in seconds (3.10.6+). `0` (the default) snaps to the target. See [Smoothing](#smoothing-sdk-3106). **[Docs]** |
| `SolveInLocalSpace` | `bool` | Match local values instead of world values. **[Docs]** |
| `FreezeToWorld` | `bool` | Ignore the sources and hold the current world pose. The pose is captured on the false→true transition. It only affects axes marked as frozen. **[Docs]** |
| `RebakeOffsetsWhenUnfrozen` | `bool` | Recompute the offsets when `FreezeToWorld` goes true→false. **[Docs]** |

## Common methods

| Method | Effect |
|---|---|
| `ApplyConfigurationChanges()` | **Required** after you change properties from script. Call it once after a batch. **[Docs]** |
| `ActivateConstraint()` | Activate and lock, keeping the current offsets. Same as the inspector's Activate button. **[Docs]** |
| `ZeroConstraint()` | Activate and lock, resetting the offsets. Same as the inspector's Zero button. **[Docs]** |

## Sources

`VRCConstraintSource` is a struct, so **read it, modify it, then write it back**. **[Docs]**

### Source methods (SDK 3.10.6+)

SDK 3.10.6 added helper methods for reading and changing sources. They work like Unity's own constraint API. In Udon, use these instead of editing the `Sources` list. VRChat found that changes made directly to the list (a struct-typed collection) don't always apply reliably from Udon. The list stays exposed only so older worlds keep compiling. **[Docs]** <https://creators.vrchat.com/common-components/constraints/constraints-api#source-access> · <https://creators.vrchat.com/releases/release-3-10-6>

These signatures first appear in the Udon extern table in 3.10.6 (`__GetSource__SystemInt32__VRCDynamicsVRCConstraintSource`, `__AddSource__VRCDynamicsVRCConstraintSource__SystemVoid`, `__ClearSources__SystemVoid`, and so on). **[Source]** (`Runtime/Udon/External/VRC.Udon.Wrapper.dll`)

| Member | Returns | Notes |
|---|---|---|
| `SourceCount` | `int` | Number of sources. **[Docs]** |
| `GetSource(int index)` | `VRCConstraintSource` | Out of range: logs an error and returns an empty source. **[Docs]** |
| `SetSource(int index, VRCConstraintSource source)` | `void` | Overwrites the source at `index`. Out of range: logs an error. **[Docs]** |
| `AddSource(VRCConstraintSource source)` | `void` | Appends to the end. **[Docs]** |
| `RemoveSource(VRCConstraintSource source)` | `bool` | `true` if a matching source was found and removed. **[Docs]** |
| `RemoveSource(int index)` | `void` | Removes by index. **[Docs]** |
| `GetSources(VRCConstraintSource[] sources)` | `int` | Copies sources into your array from index 0 and returns how many it copied. Stops early if the array is too small. **[Docs]** |
| `SetSources(VRCConstraintSource[] sources)` | `void` | Replaces every source with the array. `null` or empty clears the list. **[Docs]** |
| `ClearSources()` | `void` | Removes all sources. **[Docs]** |

```csharp
// Randomise every source weight (3.10.6+).
for (int i = 0; i < constraint.SourceCount; i++)
{
    VRCConstraintSource source = constraint.GetSource(i);
    source.Weight = UnityEngine.Random.value;
    constraint.SetSource(i, source);
}

// Add, then remove, a source.
VRCConstraintSource extra = new VRCConstraintSource(transform, 1.0f);
constraint.AddSource(extra);
bool removed = constraint.RemoveSource(extra);
```

Adapted from the official examples. **[Docs]** Bounds-check `index` against `SourceCount` before calling `GetSource` / `SetSource`; out-of-range calls log an error rather than throw. **[Docs]**

### The raw `Sources` list (pre-3.10.6 code)

Code written before 3.10.6 edits the list directly:

```csharp
VRCConstraintSource source = constraint.Sources[i];
source.Weight = UnityEngine.Random.value;
constraint.Sources[i] = source;

constraint.Sources.Add(source);
constraint.Sources.Remove(source);   // or
constraint.Sources.RemoveAt(i);
```

This still compiles, but VRChat "strongly recommends" moving Udon code to the methods above. The list is still fine in animation clips and in editor/SDK tooling written in C#. Only the first **16** entries can be targeted by animators. **[Docs]**

Avoid changing source transform references at runtime where you can, because the constraint may have to re-evaluate. Changing source *weights*, or moving the source transforms themselves, is fine. **[Docs]**

## Smoothing (SDK 3.10.6+)

`SmoothingFactor` (inspector: **Smoothing Factor**) eases the constraint towards its target instead of snapping. **[Docs]**

- `0` (default) means no smoothing. Above zero, the constraint covers about 63% of the remaining distance every `SmoothingFactor` seconds. **[Docs]**
- Each frame it applies `result = lerp(current, target, 1 - exp(-deltaTime / smoothingFactor))`, so the easing doesn't depend on frame rate. **[Docs]**
- Values between 0 and 1 usually work best. Go above 1 for very slow motion. **[Docs]**
- In SDK projects, smoothing only runs in Play mode. **[Docs]**
- It's a configuration property, so call `ApplyConfigurationChanges()` after setting it from Udon. **[Docs]** (general rule for constraint properties)

Example use: an aim constraint that turns towards a moving target with some lag. **[Docs]**

## Per-type values

- **At-rest / offset:** each type has its own appropriately named pair. Position constraints use `PositionAtRest` / `PositionOffset` (a distance) and rotation constraints use `RotationAtRest` / `RotationOffset` (Euler angles). **[Docs]** The docs don't spell out the names for aim, look-at and scale. **[Unverified]**
- **Parent:** offsets are per source. Set `ParentPositionOffset` / `ParentRotationOffset` on each `VRCConstraintSource` in `Sources`. **[Docs]**
- **Aim:** `AimAxis` (`Vector3`), `UpAxis` (`Vector3`), `WorldUpTransform` (`Transform`), `WorldUpVector` (`Vector3`), `WorldUp` (`VRCConstraintBase.WorldUpType`: `SceneUp`, `ObjectUp`, `ObjectRotationUp`, `Vector`, `None`). **[Docs]**
- **Look-at:** `Roll` (`float`, degrees, used when `UseUpTransform` is false), `WorldUpTransform` (`Transform`, used when `UseUpTransform` is true), `UseUpTransform` (`bool`). **[Docs]**

## Example: drop an object in place

```csharp
using UdonSharp;
using VRC.SDK3.Dynamics.Constraint.Components;

public class FreezeToggle : UdonSharpBehaviour
{
    public VRCParentConstraint follow;   // all axes frozen in the inspector

    public override void Interact()
    {
        follow.FreezeToWorld = !follow.FreezeToWorld;
        follow.ApplyConfigurationChanges();
    }
}
```

This is the world equivalent of the documented avatar pattern of animating `Freeze To World`: "you would use an Udon script to change the value of `FreezeToWorld`". **[Docs]**

## Gotchas

- If a constraint seems dead, check that `Is Active` is enabled, the component and GameObject are enabled, and `Lock` is enabled. **[Docs]**
- Freezing to world is **not** the same as disabling the component. A frozen constraint actively counter-moves the transform in local space. **[Docs]**
- Don't put a constraint on the same GameObject as a PhysBone. **[Docs]**
- Since 3.10.2, destroying a PhysBone that parents VRC Constraints no longer throws internal exceptions. **[Docs]**
- `name` isn't exposed to Udon on constraints (3.10.0). Use `.gameObject.name`. **[Docs]**
- Udon code that edits `Sources` directly may not apply its changes reliably. Use the 3.10.6 source methods. **[Docs]**

## See also

- [Dynamics overview](README.md)
- [PhysBones](physbones.md)
- [Contacts](contacts.md)
- [UdonSharpBehaviour](../udonsharp-behaviour/README.md)
- [GameObject Lifecycle → Transform parenting](../gameobject-lifecycle/transform-parenting.md): an alternative to parent constraints.
- [Persistence → Player Objects](../persistence/player-objects.md)
