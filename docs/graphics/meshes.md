# Meshes

Procedural mesh construction at runtime. Officially undocumented but widely used by the community. **[Community-known]**

## Construction

```csharp
Mesh m = new Mesh();
```

The default constructor creates an empty mesh.

## Writing data

```csharp
m.vertices  = new Vector3[] { /* ... */ };
m.normals   = new Vector3[] { /* ... */ };
m.tangents  = new Vector4[] { /* ... */ };
m.uv        = new Vector2[] { /* ... */ };
m.uv2       = new Vector2[] { /* ... */ };
m.colors    = new Color[]   { /* ... */ };
m.colors32  = new Color32[] { /* ... */ };
m.triangles = new int[]     { /* ... */ };
m.boneWeights = ...;       // for skinned meshes
m.bindposes   = ...;
```

The `Set*` variants also work and are equivalent:

```csharp
m.SetVertices(List<Vector3> vertices);
m.SetTriangles(int[] triangles, int submesh);
m.SetNormals(...);
m.SetUVs(int channel, ...);
m.SetColors(...);
```

**[Community-known]** for all of the above.

## Recalculation helpers

```csharp
m.RecalculateNormals();
m.RecalculateBounds();
m.RecalculateTangents();
m.Clear();              // wipe the mesh
```

`RecalculateNormals` infers per-vertex normals from triangle topology. `RecalculateBounds` updates the AABB used for culling. Call after every set of changes.

## Reading data

```csharp
Vector3[] verts = m.vertices;     // returns a copy
int       n     = m.vertexCount;
int[]     tris  = m.triangles;
Bounds    b     = m.bounds;
```

Reading `vertices`/`triangles` allocates a fresh array each time — cache locally if you're using both.

## Assigning to a renderer

```csharp
MeshFilter mf = GetComponent<MeshFilter>();
mf.mesh = newMesh;            // per-instance copy
mf.sharedMesh = newMesh;      // shared
```

`mf.mesh` follows the same per-instance-copy semantics as `Renderer.material` — first access forks.

For skinned meshes:

```csharp
SkinnedMeshRenderer smr = GetComponent<SkinnedMeshRenderer>();
smr.sharedMesh = newMesh;
```

## Canonical pattern: build a quad in Start

```csharp
public MeshFilter targetFilter;   // pre-attached

void Start() {
    Mesh m = new Mesh();
    m.vertices = new Vector3[] {
        new Vector3(0, 0, 0),
        new Vector3(1, 0, 0),
        new Vector3(1, 1, 0),
        new Vector3(0, 1, 0),
    };
    m.uv = new Vector2[] {
        new Vector2(0, 0),
        new Vector2(1, 0),
        new Vector2(1, 1),
        new Vector2(0, 1),
    };
    m.triangles = new int[] { 0, 1, 2,  0, 2, 3 };
    m.RecalculateNormals();
    m.RecalculateBounds();
    targetFilter.mesh = m;
}
```

Note: you cannot `AddComponent<MeshFilter>()`. The MeshFilter / MeshRenderer must already be on the GameObject in the scene authoring.

## Submeshes

```csharp
m.subMeshCount = 2;
m.SetTriangles(trianglesForMaterial0, 0);
m.SetTriangles(trianglesForMaterial1, 1);
```

A submesh renders with a different material on the same `MeshRenderer`. Assign materials in the same order on the renderer (size the `materials` array to `subMeshCount`).

## Index format

For meshes with more than 65535 vertices, set:

```csharp
m.indexFormat = UnityEngine.Rendering.IndexFormat.UInt32;
```

**[Unverified]** — community evidence is mixed; some versions of Udon don't expose `IndexFormat`. Test in your project.

## Cleanup

`Mesh` is a `UnityEngine.Object`. If you create many meshes over time and don't reassign them, they accumulate. `Destroy(meshAsset)` works on runtime-created meshes. Don't destroy meshes assigned via `sharedMesh` to authored scene assets.

## Performance notes

- Each `vertices = ...` assignment uploads to the GPU. Batch all writes, *then* `RecalculateNormals`/`RecalculateBounds`, then assign to the filter.
- For meshes that change every frame: pre-allocate the arrays, mutate in place, and use `mesh.MarkDynamic()` (community-known) before any writes to hint the GPU driver.
- For thousands of identical meshes, prefer `VRCGraphics.DrawMeshInstanced` over many `MeshFilter`s.

## GPU-side alternatives (community)

Most heavy procedural geometry in production VRChat worlds happens **in shaders**, not in Udon-side `Mesh` writes. cnlohr's `shadertrixx` reference notebook documents the common tricks: **[Community]** <https://github.com/cnlohr/shadertrixx>

- **GPU particles via geometry shaders** — expand points into triangles in the shader. Mind that **mesh bounds must be enlarged** when vertices move in-shader, or frustum culling will eject the mesh.
- **`REDSIM/GPUParticleVolumes`** — ultra-fast GPU particles with VRC Light Volumes support. **[Community]** <https://github.com/REDSIM/GPUParticleVolumes>
- **`SCRN-VRC/Raymarched-GPU-Particles-with-Screenspace-Physics`** — raymarched particles using GrabPass for screen-space physics. **[Community]** <https://github.com/SCRN-VRC/Raymarched-GPU-Particles-with-Screenspace-Physics>
- **Wireframe overlays** — `lereldarion/unity-shaders`, `mika-archived/Unity-WireframeShader`, `netri/Neitri-Unity-Shaders`. **[Community]**

**Quest perf:** tessellation is slower than pre-subdivided geometry on VR hardware; geometry shaders perform poorly on Quest. Both push you toward `DrawMeshInstanced` for many copies of the same pre-subdivided mesh.

> Honest gap: as of research no flagship community marching-cubes or voxel-renderer project for Udon worlds surfaced — discoverable Unity implementations (Scrawk, Javier-Garzo) are not Udon-targeted. The GPU-particle approach has filled this niche for most use cases.

## See also

- [VRCGraphics](vrc-graphics.md) — `DrawMeshInstanced`.
- [Materials](materials.md)
- [GameObject Lifecycle](../gameobject-lifecycle/README.md) — why MeshFilter must be pre-attached.
- [References → Community Resources](../references/community-resources.md)
