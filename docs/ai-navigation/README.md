# AI Navigation

Udon supports Unity's AI Navigation package: `NavMesh`, `NavMeshAgent`, `NavMeshObstacle`, runtime baking via `NavMeshSurface`.

Source: <https://creators.vrchat.com/worlds/udon/ai-navigation/>. **[Docs]**

## What works

- **`NavMeshAgent`** as a component on a GameObject — `SetDestination`, `velocity`, `speed`, `angularSpeed`, `stoppingDistance`, `Warp`, etc. **[Docs]**
- **`NavMesh`** queries — `Raycast`, `SamplePosition`, `CalculatePath`. **[Docs]**
- **`NavMeshObstacle`** for dynamic obstacles.
- **`NavMeshSurface`** for runtime baking. **[Docs]**
- **Off-mesh links** for door / jump / climb traversals.
- **Multiple NavMesh Surfaces** per world for granular control.

## Known limitations

- **Custom Agent Types are not supported.** Use the built-in agent type definitions.
- **`NavMeshSurface.CollectObjects` does not function correctly** in Udon. **[Docs]**
- **`NavMeshSurface.UpdateNavMesh()`** does not function correctly. **[Docs]**
- VRChat does not enumerate the full method allowlist — most other `NavMeshSurface` methods (`BuildNavMesh`, etc.) and the standard `NavMeshAgent` surface are believed to work. **[Unverified]**

For the full method surface, consult Unity's official AI Navigation package documentation; VRChat docs point readers there.

## Skeleton

```csharp
using UdonSharp;
using UnityEngine;
using UnityEngine.AI;

[UdonBehaviourSyncMode(BehaviourSyncMode.None)]
public class Wanderer : UdonSharpBehaviour {
    public Transform[] waypoints;
    NavMeshAgent _agent;
    int _i;

    void Start() {
        _agent = (NavMeshAgent)GetComponent(typeof(NavMeshAgent));
        Next();
    }

    void Update() {
        if (!_agent.pathPending && _agent.remainingDistance < 0.5f) Next();
    }

    void Next() {
        _agent.SetDestination(waypoints[_i].position);
        _i = (_i + 1) % waypoints.Length;
    }
}
```

## Runtime baking

`NavMeshSurface.BuildNavMesh()` works in Udon and is the standard way to bake a NavMesh whose obstacles only exist at runtime (e.g. instantiated scenery).

```csharp
public NavMeshSurface surface;

void Start() {
    surface.BuildNavMesh();
}
```

Don't call this every frame — it's an expensive operation. Bake once on load, then again only when the world's static geometry actually changes.

## Networking

`NavMeshAgent`'s motion is **not** automatically synced. If you want NPC positions to agree across clients, either:

- Run the agent only on the owner and sync the position via `[UdonSynced]` (or `VRCObjectSync`), or
- Run an identical agent on every client and trust deterministic playback (rarely deterministic in practice).

The owner-only approach is standard.

## See also

- Unity's [AI Navigation package documentation](https://docs.unity3d.com/Packages/com.unity.ai.navigation@latest).
- [GameObject Lifecycle](../gameobject-lifecycle/README.md) — for spawning navigatable scenery.
- [Networking](../networking/README.md).
