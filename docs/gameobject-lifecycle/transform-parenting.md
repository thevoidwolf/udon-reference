# Transform & Parenting

Standard Unity Transform operations work in Udon. This page collects the ones that come up frequently.

## Reading position

```csharp
Vector3 worldPos      = transform.position;
Vector3 localPos      = transform.localPosition;
Quaternion worldRot   = transform.rotation;
Quaternion localRot   = transform.localRotation;
Vector3 worldEulers   = transform.eulerAngles;
Vector3 localScale    = transform.localScale;
Vector3 lossyScale    = transform.lossyScale;   // read-only world-space scale
```

`transform.right`, `.up`, `.forward` give the unit basis vectors in world space.

## Writing position

```csharp
transform.position = new Vector3(0, 1, 0);
transform.SetPositionAndRotation(pos, rot);     // atomic, avoids double recalc
transform.localPosition = Vector3.zero;
transform.eulerAngles = new Vector3(0, 90, 0);
transform.localScale = Vector3.one * 2f;
```

`transform.Translate(...)`, `Rotate(...)`, `RotateAround(...)`, `LookAt(...)` all work.

## Parenting

```csharp
transform.SetParent(otherTransform, worldPositionStays: false);
transform.SetParent(otherTransform);            // worldPositionStays defaults to true
transform.parent = otherTransform;              // equivalent to SetParent(other, true)
transform.SetParent(null);                      // unparent
```

The `worldPositionStays` flag controls whether the child's world transform is preserved when reparenting. Default `true` re-derives local position so world position stays put.

## Iterating children

```csharp
for (int i = 0; i < transform.childCount; i++) {
    Transform child = transform.GetChild(i);
    // ...
}
```

`foreach (Transform child in transform)` also works — `Transform`'s `IEnumerator` is whitelisted. **[Community-known]**

## Layers and tags

```csharp
gameObject.layer = LayerMask.NameToLayer("Player");
gameObject.tag = "Untagged";   // tag string must already exist in the project
gameObject.name = "Cube_42";
gameObject.SetActive(true);
bool active = gameObject.activeSelf;
bool activeHierarchy = gameObject.activeInHierarchy;
```

Setting `tag` to a string the project hasn't declared throws. Add the tag in **Project Settings → Tags & Layers** first.

## `GameObject.Find` and `Transform.Find`

```csharp
GameObject obj = GameObject.Find("MyObject");                  // global by name
Transform   c  = transform.Find("Child/Grandchild");          // path under this
GameObject found = GameObject.FindWithTag("Player");
GameObject[] all = GameObject.FindGameObjectsWithTag("Loot");
```

All work but are slow — they scan the active scene. Prefer caching references in inspector fields where you can. The exact runtime cost is the same as in regular Unity.

## `GetComponent`

Works for all standard component types, plus `UdonBehaviour` and `UdonSharpBehaviour` subclasses:

```csharp
Rigidbody rb = GetComponent<Rigidbody>();
MyScript ms  = GetComponent<MyScript>();
Component[] all = GetComponents<Collider>();
```

For UdonSharpBehaviours specifically there is a compiler shim (`GetComponentShim.cs`) that handles the cross-program lookup; see [Compiler](../compiler/README.md) if you need the details.

`GetComponentInChildren`, `GetComponentInParent`, and their `s` variants are all supported.

## Bounds

```csharp
Renderer r = GetComponent<Renderer>();
Bounds b = r.bounds;            // world-space AABB
```

`Bounds`, `Plane`, `Ray`, `RaycastHit`, and other geometry value types are whitelisted.

## Raycasting

```csharp
RaycastHit hit;
if (Physics.Raycast(transform.position, transform.forward, out hit, 10f)) {
    // hit.collider, hit.point, hit.normal, hit.distance
}
```

Layer-mask overloads and `Physics.RaycastAll`, `SphereCast`, `OverlapBox`, `OverlapSphere` all work.

## See also

- [Players → VRCPlayerApi](../players/vrc-player-api.md) — player-relative positions.
- [Graphics](../graphics/README.md) — rendering primitives.
