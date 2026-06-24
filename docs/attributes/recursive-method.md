# `[RecursiveMethod]`

Allows a method to safely call itself (directly or indirectly) without the Udon VM stomping its locals. Defined in `Runtime/UdonSharpAttributes.cs`. **[Source]**

## Signature

```csharp
[AttributeUsage(AttributeTargets.Method, AllowMultiple = false, Inherited = false)]
public class RecursiveMethodAttribute : Attribute { }
```

## Usage

```csharp
[RecursiveMethod]
public int Fibonacci(int n) {
    if (n < 2) return n;
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}
```

## Why it's needed

Udon's calling convention reuses a per-method stack region for locals — fast and small, but it means a method calling itself would clobber its own variables. The runtime workaround is to allocate a heap-backed stack frame and copy locals in/out on every recursive call.

This is not on by default because it has measurable cost: a recursive method runs notably slower than a non-recursive one and produces some heap traffic. Apply the attribute only to methods that **actually recurse** (directly or through a chain).

## Direct vs. indirect recursion

The attribute applies to the **recursing method**, not to its callers. If `A` calls `B` and `B` calls back into `A`, only `A` needs the attribute. The compiler will report an error if it detects a recursive call into a method that isn't marked.

```csharp
[RecursiveMethod]
public void Walk(GameObject node) {
    foreach (Transform child in node.transform)
        Walk(child.gameObject);
}
```

## Cost

`[RecursiveMethod]` methods perform:
- An additional heap allocation per call (the saved stack frame).
- A copy-in / copy-out of all the method's locals.

For deeply recursive workloads (a tree walk of thousands of nodes), prefer rewriting as iteration with an explicit stack/queue (using a `DataList` or array) when you can. **[Community-known]**

## Not for cross-behaviour reentrancy

If `A.Foo` calls `B.SendCustomEvent("Foo")` and `B.Foo` calls back into `A.SendCustomEvent("Foo")`, you do not need this attribute — those are independent stacks across different Udon programs.

## See also

- [Compiler constraints](../compiler/README.md) — for the general "what's allowed" picture.
