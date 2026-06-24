# VRCJson

`VRC.SDK3.Data.VRCJson` — bidirectional conversion between `DataList`/`DataDictionary` and JSON strings. **[Docs]**

## Methods

```csharp
static bool TrySerializeToJson(DataToken input, JsonExportType type, out string output);
static bool TryDeserializeFromJson(string input, out DataToken result);
```

Both return a `bool` success indicator with error details delivered through the out parameter:
- On serialization failure, `output` carries a structured error message.
- On deserialization failure, `result` is an error token with `TokenType == TokenType.Error`.

## `JsonExportType`

```csharp
public enum JsonExportType {
    Beautify,   // newlines + tab-per-depth indent (human-readable)
    Minify,     // single-line, minimal whitespace
}
```

## Serialization rules

- The **root** `DataToken` must be a `DataList` or `DataDictionary`. Raw primitives at the root fail with `DataError.TypeUnsupported`.
- Dictionary keys must be **strings**. Non-string keys → `DataError.TypeUnsupported`.
- `Reference` tokens cannot be serialized → `DataError.TypeUnsupported`.
- `float`/`double` values that are `NaN` or `±Infinity` fail with `DataError.ValueUnsupported`.
- Round-tripping loses numeric type detail: deserialized numbers come back as `Double`.

## Deserialization rules

- Top-level JSON validity does not guarantee nested validity. Errors surface **lazily** when you read a malformed nested element via `TryGetValue` (`DataError.UnableToParse`).
- The result is always a `DataList` (top-level array) or `DataDictionary` (top-level object). Check `TokenType` after deserializing.

## Examples

### Round-trip

```csharp
DataDictionary state = new DataDictionary();
state.SetValue("score", 100);
state.SetValue("playerName", "Alice");

if (VRCJson.TrySerializeToJson(state, JsonExportType.Minify, out string json)) {
    Debug.Log(json);   // {"score":100,"playerName":"Alice"}
}

if (VRCJson.TryDeserializeFromJson(json, out DataToken result) && result.TokenType == TokenType.Dictionary) {
    DataDictionary d = result.DataDictionary;
    if (d.TryGetValue("playerName", TokenType.String, out DataToken name))
        Debug.Log(name.String);
}
```

### Use with networking

Since `DataList` / `DataDictionary` are not directly syncable, serialize and sync the string:

```csharp
[UdonSynced] string _payload;

void Push(DataDictionary data) {
    if (VRCJson.TrySerializeToJson(data, JsonExportType.Minify, out _payload))
        RequestSerialization();
}

public override void OnDeserialization() {
    if (VRCJson.TryDeserializeFromJson(_payload, out DataToken result)
        && result.TokenType == TokenType.Dictionary)
        ApplyState(result.DataDictionary);
}
```

Watch the serialization packet budget — JSON has overhead vs. raw binary.

### Use with persistence

`PlayerData.SetString(key, jsonString)` to persist; `PlayerData.GetString` to read; deserialize on load. The 100 KB-per-player persistence cap applies.

## Limits (not officially documented)

- **JSON string size:** undocumented. The byte budgets of `[UdonSynced]` (≈7 KB / ≈49 KB) implicitly cap network-bound JSON.
- **Token count:** undocumented.
- **Nesting depth:** undocumented. Deeply nested structures may be slow to serialize.
- **Rate limits:** undocumented.

## Community workarounds for VRCJson limitations

- **Numeric type loss.** All numbers come back as `Double` after a round-trip. If the original type matters, store it explicitly:
  ```csharp
  {"hp": 100, "hp_type": "int"}
  ```
  Cast on read. Or accept lossy round-trip with `(int)token.Double`. **[Community]**
- **Non-string keys.** Stringify (`"42"` instead of `42`) and parse back on read.
- **Object references.** Serialize a stable lookup key (network ID, name, index into a pre-placed array) and rehydrate on read. References cannot be serialized natively.
- **NaN/Infinity.** Sanitize floats before serializing — `DataError.ValueUnsupported` will fail the whole call.
- **Always check the boolean return** of `TrySerializeToJson` / `TryDeserializeFromJson`. The error-token / error-string in the out parameter is your only signal.

## Canonical sync pattern

The recommended pattern when networking a `DataDictionary` or `DataList`:

```csharp
[UdonSynced] private string _json;
private DataDictionary _state = new DataDictionary();

public override void OnPreSerialization() {
    VRCJson.TrySerializeToJson(_state, JsonExportType.Minify, out _json);
}

public override void OnDeserialization() {
    if (VRCJson.TryDeserializeFromJson(_json, out DataToken tok)
        && tok.TokenType == TokenType.Dictionary) {
        _state = tok.DataDictionary;
    }
}
```

This pattern is the de-facto community standard and is documented in official examples. **[Docs]** <https://creators.vrchat.com/worlds/udon/data-containers/>

## See also

- [DataDictionary](data-dictionary.md)
- [DataList](data-list.md)
- [Persistence → PlayerData](../persistence/player-data.md)
- [Networking → serialization](../networking/serialization.md)
