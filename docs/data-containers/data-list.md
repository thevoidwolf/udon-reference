# DataList

`VRC.SDK3.Data.DataList` — a sequential, dynamically-sized collection of `DataToken` values. Behaves like `List<DataToken>` from regular .NET. **[Docs]**

## Constructors

```csharp
new DataList();
new DataList(int capacity);
new DataList(params DataToken[] tokens);   // create pre-populated
```

## Properties

| Property | Get/Set | Notes |
|---|---|---|
| `Count` | get | Number of elements. |
| `Capacity` | get/set | Internal storage size. Settable, can shrink below `Count`. |

## Methods

| Method | Returns | Notes |
|---|---|---|
| `Add(DataToken token)` | void | Append to end. |
| `Insert(int index, DataToken token)` | bool | Shifts subsequent elements right. Returns success. |
| `Remove(DataToken value)` | bool | Removes first occurrence. |
| `RemoveAt(int index)` | void | Throws on out-of-range. |
| `Contains(DataToken value)` | bool | |
| `IndexOf(DataToken item)` | int | `-1` if not found. |
| `Clear()` | void | Removes all. Doesn't shrink Capacity. |
| `TryGetValue(int index, out DataToken output)` | bool | `output` is an error token on failure. |
| `TryGetValue(int index, TokenType expected, out DataToken output)` | bool | Type-checked variant. |

## Iteration

```csharp
DataList items = new DataList();
items.Add("a"); items.Add("b"); items.Add("c");

for (int i = 0; i < items.Count; i++) {
    if (items.TryGetValue(i, TokenType.String, out DataToken t))
        Debug.Log(t.String);
}
```

`foreach` over a `DataList` is **not** in the Udon allowlist (no whitelisted enumerator). Use index iteration. **[Community-known]**

## Examples

```csharp
DataList groceries = new DataList() { "Bananas", "Grapes", "Milk" };
groceries.Add("Eggs");
groceries.Insert(0, "Apples");
Debug.Log(groceries.Count);                   // 5

if (groceries.IndexOf("Milk") >= 0) {
    Debug.Log("Got milk.");
}
```

## Caveats

- **No documented hard size limit.** Performance degrades with very large lists.
- `RemoveAt` throws on bad index; use `TryGetValue` first if you're unsure.
- Iteration cost is `O(n)` for `Contains` / `IndexOf` / `Remove(value)`.
- Mixed-type contents are legal: a `DataList` can hold a string, an int, and a nested `DataDictionary` simultaneously.

## See also

- [DataToken](data-token.md)
- [DataDictionary](data-dictionary.md)
- [VRCJson](vrc-json.md) — `DataList` can serialize as a JSON array.
