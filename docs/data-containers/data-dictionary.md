# DataDictionary

`VRC.SDK3.Data.DataDictionary` — a key-value collection where both keys and values are `DataToken`s. Behaves like `Dictionary<DataToken, DataToken>`. **[Docs]**

## Constructors

```csharp
new DataDictionary();
new DataDictionary(int capacity);
```

## Properties

| Property | Get/Set | Notes |
|---|---|---|
| `Count` | get | Number of key-value pairs. |

There is no public `Capacity` exposed.

## Methods

| Method | Returns | Notes |
|---|---|---|
| `Add(DataToken key, DataToken value)` | void | **Throws** if key already exists. |
| `SetValue(DataToken key, DataToken value)` | void | Upsert — creates or overwrites. Prefer this over `Add` for "set." |
| `TryGetValue(DataToken key, out DataToken output)` | bool | `output` is an error token if missing. |
| `TryGetValue(DataToken key, TokenType expected, out DataToken output)` | bool | Type-checked. |
| `Remove(DataToken key)` | bool | True iff removed. |
| `Remove(DataToken key, out DataToken value)` | bool | Removes and outputs the removed value. |
| `ContainsKey(DataToken key)` | bool | |
| `ContainsValue(DataToken value)` | bool | `O(n)`. |
| `Clear()` | void | |
| `GetKeys()` | `DataList` | All keys, in insertion order (or implementation-defined order). |
| `GetValues()` | `DataList` | All values. |
| `ShallowClone()` | `DataDictionary` | Top-level copy; nested lists/dicts are *shared* references. |
| `DeepClone()` | `DataDictionary` | Recursive copy of nested DataList/DataDictionary structure. Object references (Reference tokens) are still shared. |

## Iteration

```csharp
DataList keys = dict.GetKeys();
for (int i = 0; i < keys.Count; i++) {
    if (keys.TryGetValue(i, out DataToken key)) {
        dict.TryGetValue(key, out DataToken value);
        Debug.Log($"{key} = {value}");
    }
}
```

Each `GetKeys` / `GetValues` call returns a new `DataList`. Don't call inside a tight inner loop.

## Examples

```csharp
DataDictionary settings = new DataDictionary();
settings.SetValue("volume", 0.8f);
settings.SetValue("playerName", "Alice");
settings.SetValue("score", 100);

if (settings.TryGetValue("score", TokenType.Int, out DataToken scoreTok))
    Debug.Log($"Score: {scoreTok.Int}");
```

## Nested structures

`DataDictionary` values can be lists or other dictionaries:

```csharp
DataDictionary players = new DataDictionary();

DataDictionary alice = new DataDictionary();
alice.SetValue("score", 100);
alice.SetValue("alive", true);
players.SetValue("Alice", alice);

DataDictionary bob = new DataDictionary();
bob.SetValue("score", 75);
bob.SetValue("alive", false);
players.SetValue("Bob", bob);
```

The structure round-trips through `VRCJson` directly.

## Caveats

- **`Add` throws on duplicate key.** Always use `SetValue` if you don't strictly need the duplicate-key check.
- **Key equality is structural** — two `DataToken`s with the same TokenType and value compare equal as keys.
- **No documented size limit** — large dictionaries impact memory and iteration cost.
- **`ContainsValue` is O(n)** — fine for small dictionaries, expensive at scale.
- **`ShallowClone` shares nested references** — if you mutate a nested `DataList` in the clone, the original sees the change. `DeepClone` avoids that.

## Known community gotchas

- **`DeepClone` is shallow on `Reference` tokens.** Nested `DataList`/`DataDictionary` get recursively cloned, but `TokenType.Reference` tokens (UnityEngine objects, raw arrays) keep the original reference. Modify-by-reference will surprise you. **[Community]** <https://creators.vrchat.com/worlds/udon/data-containers/data-dictionaries/>
- **`TokenType` confusion via `.Number`.** Reading any numeric token through `.Number` returns `double`. For `long` / `ulong` values above 2^53 you lose precision silently. Use `.Long` / `.ULong` explicitly when the type matters.
- **Unity does NOT serialize `DataDictionary`.** Declare instance fields `private` or `[NonSerialized]`. Public serialized fields are wiped on play.

## See also

- [DataToken](data-token.md)
- [DataList](data-list.md)
- [VRCJson](vrc-json.md) — `DataDictionary` serializes as a JSON object.
