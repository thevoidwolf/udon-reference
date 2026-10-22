# VRCJson

`VRC.SDK3.Data.VRCJson` — bidirectional conversion between `DataList`/`DataDictionary` and JSON strings. **[Docs]**

## Methods

```csharp
static bool TrySerializeToJson(DataToken input, JsonExportType exportType, out DataToken result);
static bool TryDeserializeFromJson(string input, out DataToken result);
```

**Both out parameters are `DataToken`** — on serialization success the JSON is in `result.String`, not in an `out string`. **[Docs]** <https://creators.vrchat.com/worlds/udon/data-containers/vrcjson/>

Both return a `bool` success indicator with error details delivered through the out token:
- On serialization failure, `result` is an error token (`DataError.TypeUnsupported` / `ValueUnsupported`); `result.ToString()` gives the reason.
- On deserialization failure, `result` is an error token with `TokenType == TokenType.Error`; `result.ToString()` explains what went wrong.
- Since **SDK 3.10.2**, `TryDeserializeFromJson` always returns `false` on undeserializable input instead of potentially throwing. **[Docs]** (release notes)

In Udon Graph the `VRC` prefix is dropped — search for `Json`. Namespace in U#: `VRC.SDK3.Data`. **[Docs]**

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

### Parser fixes by SDK version

| SDK | Change **[Docs]** (release notes) |
|---|---|
| 3.7.4 | Braces inside strings no longer break deserialization; empty arrays with inner whitespace (`[ ]`) parse; a leading BOM (`\uFEFF`) is ignored. |
| 3.10.2 | `TryDeserializeFromJson` returns `false` instead of throwing on bad input. `Minify` output no longer contains redundant spaces. |
| 3.10.5 | Stricter, more robust parsing: mismatched closers like `[}` are now **rejected**; collection characters inside strings (`["["]`, `["{"]`) and nested braces in strings (`{ "x": { "y": "{" } }`) now parse correctly. |

If you relied on the old lenient behaviour (malformed JSON from a hand-edited file that used to "work"), validate your sources after upgrading to 3.10.5.

## Examples

### Round-trip

```csharp
DataDictionary state = new DataDictionary();
state.SetValue("score", 100);
state.SetValue("playerName", "Alice");

string json = null;
if (VRCJson.TrySerializeToJson(state, JsonExportType.Minify, out DataToken jsonTok)) {
    json = jsonTok.String;
    Debug.Log(json);   // e.g. {"score":100,"playerName":"Alice"} (key order not guaranteed)
} else {
    Debug.LogError(jsonTok.ToString());
}

if (VRCJson.TryDeserializeFromJson(json, out DataToken result) && result.TokenType == TokenType.DataDictionary) {
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
    if (VRCJson.TrySerializeToJson(data, JsonExportType.Minify, out DataToken tok)) {
        _payload = tok.String;
        RequestSerialization();
    }
}

public override void OnDeserialization() {
    if (VRCJson.TryDeserializeFromJson(_payload, out DataToken result)
        && result.TokenType == TokenType.DataDictionary)
        ApplyState(result.DataDictionary);
}
```

Watch the serialization packet budget — JSON has overhead vs. raw binary.

### Use with persistence

`PlayerData.SetString(key, jsonString)` to persist; `PlayerData.GetString` to read; deserialize on load. The cap is 100 KB of PlayerData (plus 100 KB of PlayerObject data) per player per world, measured compressed. **[Docs]** <https://creators.vrchat.com/worlds/udon/persistence/>

## Limits (not officially documented)

- **JSON string size:** undocumented. Sync budgets implicitly cap network-bound JSON: roughly **280,496 bytes** per serialization for Manual sync and **~200 bytes** for Continuous, and Udon scripts can send out only about **11 KB per second** overall. **[Docs]** <https://creators.vrchat.com/worlds/udon/networking/network-details/>
- **Token count:** undocumented.
- **Nesting depth:** undocumented. Deeply nested structures may be slow to serialize.
- **Rate limits:** undocumented.

## Community workarounds for VRCJson limitations

- **Numeric type loss.** All numbers come back as `Double` after a round-trip. If the original type matters, store it explicitly:
  ```csharp
  {"hp": 100, "hp_type": "int"}
  ```
  Cast on read. Or accept lossy round-trip with `(int)token.Double` — `token.Int` on a `Double` token throws. **[Community]**
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
    if (VRCJson.TrySerializeToJson(_state, JsonExportType.Minify, out DataToken tok))
        _json = tok.String;
    else
        Debug.LogError(tok.ToString());
}

public override void OnDeserialization() {
    if (VRCJson.TryDeserializeFromJson(_json, out DataToken tok)
        && tok.TokenType == TokenType.DataDictionary) {
        _state = tok.DataDictionary;
    }
}
```

This pattern is the de-facto community standard and is documented in official examples (Data Lists / Data Dictionaries pages, "Syncing … over the network"). **[Docs]** <https://creators.vrchat.com/worlds/udon/data-containers/data-dictionaries/>

## See also

- [DataDictionary](data-dictionary.md)
- [DataList](data-list.md)
- [Persistence → PlayerData](../persistence/player-data.md)
- [Networking → serialization](../networking/serialization.md)
