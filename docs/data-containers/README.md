# Data Containers

`VRC.SDK3.Data` provides type-erased dynamic containers that Udon does support — substitutes for `Dictionary<TKey, TValue>` and `List<T>` (which are not in the Udon allowlist because they're generic).

Source: <https://creators.vrchat.com/worlds/udon/data-containers/>. **[Docs]**

## Pages

| Page | Type |
|---|---|
| [DataToken](data-token.md) | The variant wrapper used as element type. |
| [DataList](data-list.md) | Sequential collection of `DataToken`s. |
| [DataDictionary](data-dictionary.md) | Key-value collection of `DataToken`s. |
| [VRCJson](vrc-json.md) | Bidirectional conversion between containers and JSON strings. |

## Why these exist

Udon's allowlist forbids most .NET generics in user code (see [Compiler constraints](../compiler/README.md)). `List<int>` and `Dictionary<string, MyClass>` would require runtime type information Udon doesn't carry. Instead VRChat ships:

- `DataToken` — a tagged union that can hold any primitive, string, reference, list, or dictionary.
- `DataList` / `DataDictionary` — homogeneous containers of `DataToken`s.

Pay-per-use: you reach for these when arrays aren't enough.

## When to use what

| You want | Use |
|---|---|
| Fixed-size, homogeneous, primitive elements | Plain C# array |
| Variable-size, homogeneous, primitive elements | `DataList` (or expand an array manually) |
| Variable-size, mixed types | `DataList` of `DataToken` |
| Key-value lookup | `DataDictionary` |
| Persist / transmit a complex structure | Serialize with `VRCJson` |

Plain arrays remain faster for hot loops over primitives. `DataList`/`DataDictionary` are heap-backed and slower per operation.

## Networking

`DataList` / `DataDictionary` / `DataToken` are **not** in the `[UdonSynced]` allowlist. If you need to replicate one, serialize to a JSON string and sync the string:

```csharp
[UdonSynced] string _serializedState;

void Push() {
    VRCJson.TrySerializeToJson(state, JsonExportType.Minify, out _serializedState);
    RequestSerialization();
}

public override void OnDeserialization() {
    VRCJson.TryDeserializeFromJson(_serializedState, out DataToken result);
    if (result.TokenType == TokenType.Dictionary) {
        state = result.DataDictionary;
    }
}
```

Remember the sync packet budget — large containers serialize to large strings.

## Community helpers

- **`koyashiro/generic-data-container`** — type-safe wrappers giving you compile-time `DataList<int>`, `DataDictionary<string, int>` via VPM. **[Community]** <https://github.com/koyashiro/generic-data-container>
- **Kemocade VRC Data Extensions** — extension methods over `DataList` / `DataDictionary` for ergonomic operations. **[Community]** <https://vpm-catalog.vercel.app/packages/com.kemocade.vrc.data.extensions/>
- **`koyashiro/udon-list` and `koyashiro/udon-dictionary`** — pre-`DataList` standalone implementations. **[Community]** <https://github.com/koyashiro/udon-list>
- **`hoke946/UArrayCollections`** — utility helpers treating plain arrays like List/Dictionary. Lower overhead than `DataList` for fixed-type uniform data. **[Community]** <https://github.com/hoke946/UArrayCollections>

## Structured-blob parsers

When you need to accept structured data through a player text-input or `VRCStringDownloader`:

- **JSON** — use [`VRCJson`](vrc-json.md), the canonical built-in path.
- **XML** — **`Foorack/UdonXML`** parses and writes XML in pure Udon. API: `LoadXml`, `GetChildNode`, `GetNodeValue`, `SetAttribute`, `CreateChildNode`. Lenient validation; minimally maintained but functional. **[Community]** <https://github.com/Foorack/UdonXML>
- **ZIP** — **`Foorack/UdonZip`** parses ZIP archives. Read-only. Useful for compressed save-import flows. API: `Extract`, `GetFileNames`, `GetFile`, `GetFileData`. **[Community]** <https://github.com/Foorack/UdonZip>

## Performance lore

Community consensus: prefer **plain arrays** when the size and element type are fixed and you don't need VRCJson interop or mixed types. `DataList` / `DataDictionary` are heap-backed and slower per operation. **[Community]** <https://note.com/2aka/n/n50c8d863e58c?hl=en>

Reference benchmarks (loops, recursion, GetComponent caching, etc.) are catalogued at vrclibrary.com but do not include direct `DataList` vs array micro-benchmarks. **[Community]** <https://vrclibrary.com/wiki/books/udon-benchmarking-and-performance-tests/export/html>

## See also

- [Compiler constraints](../compiler/README.md) — why generics are blocked.
- [Persistence](../persistence/README.md) — `PlayerData` accepts byte arrays which can carry a serialized container.
- [References → Community Resources](../references/community-resources.md)
