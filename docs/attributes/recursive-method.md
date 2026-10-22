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

Udon has no call stack for locals. Each method's parameters and locals live in fixed heap slots of the program, so a method calling itself would clobber its own variables. In a method marked `[RecursiveMethod]`, U# emits code around **every call to a user method**: it pushes the method's parameters and live locals onto a per-behaviour `object[]` stack before the call and pops them back afterwards. The stack doubles in size when it fills. **[Source]** (`Editor/Compiler/Binder/BoundNodes/BoundUserMethodInvocation.cs`, `BoundInvocationExpression.cs`)

This is not on by default because it has measurable cost: the marked method "perform[s] slower and usually generate[s] more garbage". Apply the attribute only to methods that **actually recurse** (directly or through a chain). **[Source]** (attribute doc comment)

## Direct vs. indirect recursion

The attribute goes on the methods **being called recursively**. Methods that merely call a recursive method don't need it. **[Source]** Because the save/restore code is emitted in the *marked* method around its outgoing calls, in an indirect cycle (`A` → `B` → `A`) **every method in the cycle** whose locals or parameters are used after the call needs the attribute, `B` as well as `A`. **[Source]** (inferred from the emit code)

The U# source has no diagnostic that detects unmarked recursion. Forgetting the attribute compiles fine and silently corrupts locals at runtime. **[Source]**

```csharp
[RecursiveMethod]
public void Walk(GameObject node) {
    foreach (Transform child in node.transform)
        Walk(child.gameObject);
}
```

## Cost

`[RecursiveMethod]` methods perform:
- A push/pop of the method's parameters and live locals for every user-method call they make. Value types are boxed into `object[]`, which is where the extra garbage comes from. **[Source]**
- An occasional resize (doubling) of the shared recursion stack when it fills. **[Source]**

**Limitation:** a `[RecursiveMethod]` method can't call user methods that have `ref`/`out` parameters. The compiler error is *"U# does not yet support calling user methods with ref/out parameters from methods marked with RecursiveMethod"*. **[Source]**

For deeply recursive workloads (a tree walk of thousands of nodes), prefer rewriting as iteration with an explicit stack/queue (using a `DataList` or array) when you can. **[Community-known]**

## Not for cross-behaviour reentrancy

If `A.Foo` calls `B.SendCustomEvent("Foo")` and `B.Foo` calls back into `A.SendCustomEvent("Foo")`, you do not need this attribute — those are independent stacks across different Udon programs.

## See also

- [Compiler constraints](../compiler/README.md) — for the general "what's allowed" picture.
