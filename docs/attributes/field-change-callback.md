# `[FieldChangeCallback]`

Routes external writes to a field through a property setter. Defined in `Runtime/UdonSharpAttributes.cs`. **[Source]**

## Signature

```csharp
[AttributeUsage(AttributeTargets.Field, AllowMultiple = false)]
public class FieldChangeCallbackAttribute : Attribute
{
    public string CallbackPropertyName { get; private set; }
    public FieldChangeCallbackAttribute(string targetPropertyName);
}
```

`targetPropertyName` is the name of a public or non-public property on the same class. That property's setter will be invoked whenever the field is written through:

- Network deserialization (i.e. when a `[UdonSynced]` field updates from a remote owner).
- `SetProgramVariable(name, value)` from another script.

Plain assignment in your own code (`_score = 42;`) does **not** trigger the callback — only the two indirect write paths do.

## Pattern

```csharp
[UdonSynced, FieldChangeCallback(nameof(Score))]
private int _score;

public int Score {
    get => _score;
    set {
        _score = value;            // setter is responsible for writing the field
        scoreText.text = value.ToString();
        OnScoreChanged();
    }
}

void Award() {
    if (!Networking.IsOwner(gameObject))
        Networking.SetOwner(Networking.LocalPlayer, gameObject);

    Score = _score + 1;             // direct setter invocation — runs both lines
    RequestSerialization();
}
```

When the next sync packet lands on another client:
1. The runtime decodes the value and is about to write `_score`.
2. It sees `[FieldChangeCallback("Score")]` and instead invokes `Score = value;`.
3. The setter updates the field and the UI text in one go.

## Why this is useful

Without `[FieldChangeCallback]`, the standard pattern is to override `OnDeserialization` and re-apply every visual that depends on synced state. That gets repetitive and easy to forget. The callback gives you one location — the setter — that runs in every write path.

## Reflection (`SetProgramVariable`)

```csharp
otherBehaviour.SetProgramVariable("_score", 99);
```

This call inside `UdonSharpBehaviour.SetProgramVariable` checks for `[FieldChangeCallback]` and routes accordingly:

```csharp
// Runtime/UdonSharpBehaviour.cs (simplified)
FieldChangeCallbackAttribute fieldChangeCallback =
    variableField.GetCustomAttribute<FieldChangeCallbackAttribute>();
if (fieldChangeCallback != null) {
    PropertyInfo p = type.GetProperty(fieldChangeCallback.CallbackPropertyName, ...);
    p.GetSetMethod(true).Invoke(this, new object[] { value });
} else {
    variableField.SetValue(this, value);
}
```

**[Source]**

## Gotchas

- The setter is **expected** to write the field. If you forget, the field will not be updated and the next sync packet will revert to the old value.
- The property must be on the same class as the field (the source code does `variableField.DeclaringType.GetProperty(...)`).
- If the property is missing or has no setter, `SetProgramVariable` silently no-ops on that field. **[Source]**

## See also

- [`[UdonSynced]`](udon-synced.md)
- [UdonSharpBehaviour → API methods](../udonsharp-behaviour/api-methods.md) — `SetProgramVariable`.
- [UdonSharpBehaviour → networking events](../udonsharp-behaviour/networking-events.md) — `OnDeserialization` (an alternative path).
