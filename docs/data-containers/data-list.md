# DataList

`VRC.SDK3.Data.DataList` — a sequential, dynamically-sized collection of `DataToken` values. Behaves like `List<DataToken>` from regular .NET. **[Docs]**

## Constructors

```csharp
new DataList();
new DataList(int capacity);                // SDK 3.10.4+ — initial capacity, like List<T>(int)
new DataList(params DataToken[] tokens);   // create pre-populated
```

The capacity constructor shipped in SDK 3.10.4 ("`VRCDataList` and `VRCDataDictionary` can now be constructed with a custom capacity"). **[Docs]** <https://creators.vrchat.com/worlds/udon/data-containers/data-lists/>

Requires `using VRC.SDK3.Data;` in UdonSharp. Most functions are thin wrappers over the underlying C# `List<T>`, so .NET semantics apply. **[Docs]**

## Properties

| Property | Get/Set | Notes |
|---|---|---|
| `Count` | get | Number of elements. |
| `Capacity` | get/set | Internal storage size, as `List<T>.Capacity`. Setting it below `Count` is invalid in .NET (`List<T>` throws) — use `TrimExcess()` to shrink. |

## Methods

Full set per the official docs. **[Docs]**

| Method | Returns | Notes |
|---|---|---|
| `Add(DataToken token)` | void | Append to end. |
| `AddRange(DataList other)` | void | Append all values of another list. |
| `BinarySearch(DataToken value)` | int | List **must be sorted**. Compares containers by count, so unsuitable for finding a specific nested list/dict. |
| `BinarySearch(DataToken value, int startIndex, int count)` | int | Ranged variant. |
| `Clear()` | void | Removes all. Doesn't shrink Capacity. |
| `Contains(DataToken value)` | bool | |
| `DeepClone()` | `DataList` | Recursively copies nested `DataList`/`DataDictionary`; arrays and other references stay shared. |
| `GetRange(int index, int count)` | `DataList` | Copies a segment. |
| `IndexOf(DataToken item)` | int | `-1` if not found. Also `(item, startIndex)` and `(item, startIndex, count)` overloads. |
| `Insert(int index, DataToken token)` | bool | Shifts subsequent elements right. Returns false if index out of range. |
| `InsertRange(int index, DataList input)` | void | Insert another list's values. |
| `LastIndexOf(DataToken item)` | int | `-1` if not found. Also `(item, startIndex)` and `(item, startIndex, count)` overloads. |
| `Remove(DataToken value)` | bool | Removes first occurrence. |
| `RemoveAll(DataToken value)` | bool | Removes every occurrence; true if any were found. |
| `RemoveAt(int index)` | void | Out-of-range index faults like `List<T>.RemoveAt` — check `Count` first. |
| `RemoveRange(int index, int count)` | void | |
| `Reverse()` / `Reverse(int index, int count)` | void | |
| `SetValue(int index, DataToken input)` | void | Overwrite at index. |
| `ShallowClone()` | `DataList` | Nested containers stay shared references. |
| `Sort()` / `Sort(int index, int count)` | void | Same-type → native comparison; mixed numbers → numeric; mixed non-numeric → `Null, Number, String, DataList, DataDictionary, Reference`. |
| `ToArray()` | `DataToken[]` | No typed `ToArray` variants (Udon has no generics). |
| `TrimExcess()` | void | Shrinks Capacity to Count (below a threshold, as `List<T>`). |
| `TryGetValue(int index, out DataToken output)` | bool | `output` is an error token on failure. |
| `TryGetValue(int index, TokenType expected, out DataToken output)` | bool | Type-checked variant; `DataError.TypeMismatch` on wrong type. |

Indexer `list[i]` (get/set) also works in U# (`DataList Get Item` in Graph) but halts the behaviour on a bad index — only use it on data you control. **[Docs]**

Calling whole-list functions (`Contains`, `IndexOf`, `LastIndexOf`) on a list produced by `VRCJson` forces parsing of all not-yet-parsed top-level values — expensive the first time, cheaper after. **[Docs]**

## Iteration

```csharp
DataList items = new DataList();
items.Add("a"); items.Add("b"); items.Add("c");

for (int i = 0; i < items.Count; i++) {
    if (items.TryGetValue(i, TokenType.String, out DataToken t))
        Debug.Log(t.String);
}
```

`foreach` over a `DataList` is not supported — use index iteration. **[Community-known]** (The open-source U# 1.1.x compiler only lowers `foreach` for arrays, `string`, and `Transform`. **[Source]**)

## Examples

```csharp
// Collection initializers only work on field declarations (private / [NonSerialized]), not inside methods. [Docs]
private DataList groceries = new DataList() { "Bananas", "Grapes", "Milk" };

void Start() {
    groceries.Add("Eggs");
    groceries.Insert(0, "Apples");
    Debug.Log(groceries.Count);                   // 5

    if (groceries.IndexOf("Milk") >= 0) {
        Debug.Log("Got milk.");
    }
}
```

## Caveats

- **No documented hard size limit.** Performance degrades with very large lists.
- `RemoveAt` throws on bad index; use `TryGetValue` first if you're unsure.
- Iteration cost is `O(n)` for `Contains` / `IndexOf` / `Remove(value)`.
- Mixed-type contents are legal: a `DataList` can hold a string, an int, and a nested `DataDictionary` simultaneously.
- **Unity does not serialize `DataList`.** Use it only on `private` or `[NonSerialized] public` fields. **[Docs]**
- **Not syncable directly** — serialize with [VRCJson](vrc-json.md) and sync the string. Arrays are cheaper for both performance and bandwidth when they fit. **[Docs]**

## See also

- [DataToken](data-token.md)
- [DataDictionary](data-dictionary.md)
- [VRCJson](vrc-json.md) — `DataList` can serialize as a JSON array.
