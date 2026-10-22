# DataToken

`VRC.SDK3.Data.DataToken` — a tagged union that wraps any value storable inside a `DataList` or `DataDictionary`. **[Docs]**

## Construction

```csharp
DataToken t1 = new DataToken(42);              // explicit
DataToken t2 = 42;                              // implicit conversion (U# supports it for primitives)
DataToken t3 = "hello";
DataToken t4 = true;
DataToken t5 = 3.14f;
DataToken nullTok = new DataToken();           // TokenType.Null
DataToken refTok = new DataToken(gameObject);  // reference to a UnityEngine.Object
```

In Udon Graph: use the `DataToken Implicit` or `DataToken Constructor` nodes.

Don't use `nameof()` to implicitly create DataTokens in UdonSharp — the docs warn it can cause errors. **[Docs]**

## `TokenType` enum

```csharp
public enum TokenType {
    Null,
    Boolean,
    SByte, Byte,
    Short, UShort,
    Int, UInt,
    Long, ULong,
    Float, Double,
    String,
    DataList,
    DataDictionary,
    Reference,
    Error,
}
```

Member names as used in the official examples (`TokenType.DataList`, `TokenType.DataDictionary`, `TokenType.String`, `TokenType.ULong`, …) — note it is `DataList` / `DataDictionary`, **not** `List` / `Dictionary`. **[Docs]** <https://creators.vrchat.com/worlds/udon/data-containers/data-tokens/>

`Reference` covers any `UnityEngine.Object`, `UdonBehaviour`, or other boxed reference (including arrays). Reference tokens **cannot be serialized to JSON**.

## Properties

Value accessors accept **widening** conversions from smaller numeric types; anything else throws and halts the UdonBehaviour. **[Docs]**

| Property | Type | Notes |
|---|---|---|
| `TokenType` | `TokenType` | Always safe to read. |
| `IsNull` | `bool` | True if the value is null *in any form*: `TokenType.Null` is always null; numbers/bools never are; strings check for `null` (not emptiness); references use `Utilities.IsValid` internally (so left players / destroyed objects count as null). |
| `IsNumber` | `bool` | True for any numeric tokenType (SByte..Double). |
| `Boolean` | `bool` | Throws if `TokenType != Boolean`. |
| `Number` | `double` | Returns the value as `double` for any numeric token; throws otherwise. Lossy for `long`/`ulong`. |
| `SByte` / `Byte` | `sbyte` / `byte` | Exact type only. |
| `Short` | `short` | Accepts `short`, `sbyte`, `byte`. |
| `UShort` | `ushort` | Accepts `ushort`, `byte`. |
| `Int` | `int` | Accepts `int`, `sbyte`, `byte`, `short`, `ushort`. Throws on `Double` (e.g. JSON-deserialized numbers). |
| `UInt` | `uint` | Accepts `uint`, `byte`, `ushort`. |
| `Long` | `long` | Accepts `long`, `sbyte`, `byte`, `short`, `ushort`, `uint` — and `int` as of **SDK 3.10.5** (fix: "a DataToken storing an `int` could not be converted into a `long`"). The docs table still omits `int`. **[Docs]** |
| `ULong` | `ulong` | Accepts `ulong`, `byte`, `ushort`, `uint`. |
| `Float` | `float` | Accepts `float` and all integer types. |
| `Double` | `double` | Accepts `double` or any other numeric type. |
| `String` | `string` | Throws if not `String`. On an **error token**, returns the error's detail message. |
| `DataList` | `DataList` | Throws if not `DataList`. |
| `DataDictionary` | `DataDictionary` | Throws if not `DataDictionary`. |
| `Reference` | `object` | Throws if not `Reference`. |
| `Error` | `DataError` | Returns `DataError.None` if the token isn't an error. Never throws. |

## Methods

| Method | Returns |
|---|---|
| `Bitcast(TokenType destinationType)` | `DataToken` — reinterprets the bit pattern (like `reinterpret_cast` / `BitConverter`). Useful e.g. `double ↔ ulong`. Truncates if the destination is smaller, zero-extends if larger. Primitive types only; returns a copy. **[Docs]** (added SDK 3.5.2) |
| `ToString()` | `string` — content as text. Never throws. Numbers use `ToString("G", CultureInfo.InvariantCulture)`; on an error token it combines the `DataError` and its message. **[Docs]** |
| `GetHashCode()` | `int` — for dictionary keying. |
| `CompareTo(DataToken other)` | `int` — -1 / 0 / 1. Containers (lists/dictionaries) compare **by count**; different non-numeric types compare by `TokenType` enum order. **[Docs]** |

See also the official [Byte and Bit Operations](https://creators.vrchat.com/worlds/udon/data-containers/byte-and-bit-operations/) page (`Bitcast`, `BitConverter`, `Buffer`). **[Docs]**

## Safe access pattern

Throwing on type mismatch halts your Udon program. Always discriminate first:

```csharp
if (dict.TryGetValue("name", TokenType.String, out DataToken result)) {
    Debug.Log(result.String);
}
```

`TryGetValue(key, TokenType expected, out)` is the safest API — it returns `false` for both "missing" and "wrong type." For optional reads where the type is dynamic:

```csharp
if (dict.TryGetValue("score", out DataToken t)) {
    switch (t.TokenType) {
        case TokenType.Int:    int s = t.Int;       break;
        case TokenType.Float:  float sf = t.Float;  break;
        case TokenType.Double: double sd = t.Double; break;   // e.g. came from VRCJson
    }
} else {
    // t is an error token here (e.g. DataError.KeyDoesNotExist)
}
```

## `DataError` enum

| Value | Cause |
|---|---|
| `None` | No error. |
| `KeyDoesNotExist` | Accessed a missing key in a DataDictionary. |
| `IndexOutOfRange` | Accessed `< 0` or `>= Count` in a DataList. |
| `TypeMismatch` | `TryGetValue(key, TokenType.Int, out)` but the stored value is something else. |
| `TypeUnsupported` | Used a reference token where JSON cannot serialize it. |
| `ValueUnsupported` | NaN/Infinity float during JSON serialization. |
| `UnableToParse` | Malformed JSON; surfaces lazily on nested element access. |

Error tokens carry both the enum (`token.Error`) and an optional detail string (`token.String`); `token.ToString()` / `Debug.Log(token)` prints both. **[Docs]**

## Caveats

- `Number` always returns `double` — for `long`/`ulong` you lose precision above 2^53.
- JSON-deserialized numbers are always `Double`; `.Int` / `.Long` on them throws. Use `.Number` / `.Double` and cast, or check `TokenType` first.
- `Reference` tokens are not JSON-serializable.
- `Bitcast` is an "unsafe" operation; it truncates / zero-extends when widths differ.
- `CompareTo` returning `0` does **not** mean two containers are equal — lists/dictionaries are compared by count only.

## See also

- [DataList](data-list.md)
- [DataDictionary](data-dictionary.md)
- [VRCJson](vrc-json.md)
