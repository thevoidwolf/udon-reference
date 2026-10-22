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
| `Sources` | `VRCConstraintSourceKeyableList` | Source list. **[Docs]** |
| `TargetTransform` | `Transform` | `null` means the constraint's own transform. **Udon can't change it at runtime**, because it's cached. Toggle between several constraint components instead. **[Docs]** |
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

```csharp
for (int i = 0; i < constraint.Sources.Count; i++)
{
    VRCConstraintSource source = constraint.Sources[i];
    source.Weight = UnityEngine.Random.value;
    constraint.Sources[i] = source;
}
```

```csharp
VRCConstraintSource source = new VRCConstraintSource(transform, 1.0f);
constraint.Sources.Add(source);
constraint.Sources.Remove(source);   // or
constraint.Sources.RemoveAt(i);
```

The snippets above are verbatim from the docs. **[Docs]** Avoid changing source transform references at runtime where you can, because the constraint may have to re-evaluate. Changing source *weights*, or moving the source transforms themselves, is fine. **[Docs]**

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

## See also

- [Dynamics overview](README.md)
- [PhysBones](physbones.md)
- [Contacts](contacts.md)
- [UdonSharpBehaviour](../udonsharp-behaviour/README.md)
- [GameObject Lifecycle → Transform parenting](../gameobject-lifecycle/transform-parenting.md): an alternative to parent constraints.
- [Persistence → Player Objects](../persistence/player-objects.md)
