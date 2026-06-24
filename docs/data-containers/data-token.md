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
    Reference,
    List,
    Dictionary,
    Error,
}
```

`Reference` covers any `UnityEngine.Object`, `UdonBehaviour`, or other boxed reference. Reference tokens **cannot be serialized to JSON**.

## Properties

| Property | Type | Notes |
|---|---|---|
| `TokenType` | `TokenType` | Always safe to read. |
| `IsNull` | `bool` | True iff `TokenType == Null`. |
| `IsNumber` | `bool` | True for any numeric tokenType (Byte..Double). |
| `Boolean` | `bool` | Throws if `TokenType != Boolean`. |
| `Number` | `double` | Returns the value as `double` for any numeric token. Lossy for `long`/`ulong`. |
| `Int` | `int` | Throws if not exactly `Int`. |
| `Float` | `float` | Throws if not exactly `Float`. |
| `Double` | `double` | Throws if not exactly `Double`. |
| `String` | `string` | Throws if not `String`. |
| `DataList` | `DataList` | Throws if not `List`. |
| `DataDictionary` | `DataDictionary` | Throws if not `Dictionary`. |
| `Reference` | `object` | Throws if not `Reference`. |
| `Error` | `DataError` | Returns `DataError.None` if the token isn't an error. Never throws. |

## Methods

| Method | Returns |
|---|---|
| `Bitcast(TokenType destinationType)` | `DataToken` — reinterprets the bit pattern. Useful e.g. `double ↔ ulong`. Only meaningful between same-width types. |
| `ToString()` | `string` — content as text. Never throws. |
| `GetHashCode()` | `int` — for dictionary keying. |
| `CompareTo(DataToken other)` | `int` — -1 / 0 / 1. |

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
        case TokenType.Error:  /* missing */         break;
    }
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

## Caveats

- `Number` always returns `double` — for `long`/`ulong` you lose precision above 2^53.
- `Reference` tokens are not JSON-serializable.
- `Bitcast` is an "unsafe" operation; only use between compatible bit widths.
- Two tokens compare equal via `CompareTo` iff their token types and values match.

## See also

- [DataList](data-list.md)
- [DataDictionary](data-dictionary.md)
- [VRCJson](vrc-json.md)
