# AI Navigation

Udon supports Unity 2022's **AI Navigation package** (`com.unity.ai.navigation` 1.1, which the VRChat docs link to): `NavMesh`, `NavMeshAgent`, `NavMeshObstacle`, NavMesh Links, and runtime baking through `NavMeshSurface`. It was integrated in **SDK 3.7.0**, after first appearing in the 3.6.2 beta. **[Docs]** <https://creators.vrchat.com/releases/release-3-7-0/>

Sources: <https://creators.vrchat.com/worlds/udon/ai-navigation/>, plus the official [AI Navigation Example](https://creators.vrchat.com/worlds/examples/ai-navigation/), which has NPCs re-pathing after `NavMeshSurface.BuildNavMesh` runs when a block is dropped. **[Docs]**

## What works

- **`NavMeshAgent`** as a component on a GameObject, with `destination` / `SetDestination` (used in the official example **[Docs]**). Other common members such as `velocity`, `speed`, `angularSpeed`, `stoppingDistance` and `Warp` work too. **[Community-known]**
- **`NavMesh`** static queries (`Raycast`, `SamplePosition`, `CalculatePath`). The docs don't list these, so test before you rely on them. **[Unverified]**
- **`NavMeshObstacle`**. Obstacles can now be dynamic. **[Docs]**
- **Runtime generation.** You can create and update NavMeshes in the VRChat client with `NavMeshSurface`. **[Docs]**
- **NavMesh Links and Off-Mesh Links** for doors, jumps and drops. **[Docs]**
- **Multiple NavMesh Surfaces** per world, plus **Modifiers** and **Modifier Volumes** for finer control. **[Docs]**
- **Area costs** are exported into the world data and restored when someone joins. **[Docs]**

## Known limitations

Per the VRChat docs: **[Docs]**

- **Custom Agent Types aren't supported.** They're bundled into world data, but VRChat has no way to retrieve and apply them yet, so use the default agent type. The feature is "being considered".
- **`NavMeshSurface.agentTypeID`** and **`NavMeshLink.agentTypeID`** aren't useful, because runtime baking doesn't work with custom agents.
- **`NavMeshSurface.CollectObjects`** is unusable because enum arrays aren't available in Udon.
- **`NavMeshSurface.UpdateNavMesh()`** returns an `AsyncOperation`, which Udon can't use. *The method itself should still work*, but you can't await or poll its completion.

VRChat doesn't list the full method allowlist. `BuildNavMesh` is shown working in the official example **[Docs]**, and most of the standard `NavMeshAgent` surface is believed to work. **[Unverified]**

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
using Unity.AI.Navigation;   // NavMeshSurface lives here, not in UnityEngine.AI

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

- Unity's [AI Navigation 1.1 package documentation](https://docs.unity3d.com/Packages/com.unity.ai.navigation@1.1/manual/index.html), the version the VRChat docs link to. `@latest` may describe a newer package than the one shipped with Unity 2022.3.
- Official [AI Navigation Example](https://creators.vrchat.com/worlds/examples/ai-navigation/) and its [example world](https://vrchat.com/home/world/wrld_b7b99484-d92f-403d-ac10-4da7e5a9ce14). **[Docs]**
- [GameObject Lifecycle](../gameobject-lifecycle/README.md) — for spawning navigatable scenery.
- [Networking](../networking/README.md).
