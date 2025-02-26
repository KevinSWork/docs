---
title: Nullable reference types
description: This article provides an overview of nullable reference types, added in C# 8.0. You'll learn how the feature provides safety against null reference exceptions, for new and existing projects.
ms.technology: csharp-null-safety
ms.date: 04/21/2020
---
# Nullable reference types

C# 8.0 introduces **nullable reference types** and **non-nullable reference types** that enable you to make important statements about the properties for reference type variables:

- **A reference isn't supposed to be null**. When variables aren't supposed to be null, the compiler enforces rules that ensure it's safe to dereference these variables without first checking that it isn't null:
  - The variable must be initialized to a non-null value.
  - The variable can never be assigned the value `null`.
- **A reference may be null**. When variables may be null, the compiler enforces different rules to ensure that you've correctly checked for a null reference:
  - The variable may only be dereferenced when the compiler can guarantee that the value isn't null.
  - These variables may be initialized with the default `null` value and may be assigned the value `null` in other code.

This new feature provides significant benefits over the handling of reference variables in earlier versions of C# where the design intent can't be determined from the variable declaration. The compiler didn't provide safety against null reference exceptions for reference types:

- **A reference can be null**. The compiler doesn't issue warnings when a reference-type variable is initialized to `null`, or later assigned `null`. The compiler issues warnings when these variables are dereferenced without null checks.
- **A reference is assumed to be not null**. The compiler doesn't issue any warnings when reference types are dereferenced. The compiler issues warnings if a variable is set to an expression that may be null.

These warnings are emitted at compile time. The compiler doesn't add any null checks or other runtime constructs in a nullable context. At runtime, a nullable reference and a non-nullable reference are equivalent.

With the addition of nullable reference types, you can declare your intent more clearly. The `null` value is the correct way to represent that a variable doesn't refer to a value. Don't use this feature to remove all `null` values from your code. Rather, you should declare your intent to the compiler and other developers that read your code. By declaring your intent, the compiler informs you when you write code that is inconsistent with that intent.

A **nullable reference type** is noted using the same syntax as [nullable value types](language-reference/builtin-types/nullable-value-types.md): a `?` is appended to the type of the variable. For example, the following variable declaration represents a nullable string variable, `name`:

```csharp
string? name;
```

Any variable where the `?` isn't appended to the type name is a **non-nullable reference type**. That includes all reference type variables in existing code when you've enabled this feature.

The compiler uses static analysis to determine if a nullable reference is known to be non-null. The compiler warns you if you dereference a nullable reference when it may be null. You can override this behavior by using the [null-forgiving operator](language-reference/operators/null-forgiving.md) `!` following a variable name. For example, if you know the `name` variable isn't null but the compiler issues a warning, you can write the following code to override the compiler's analysis:

```csharp
name!.Length;
```

## Nullability of types

Any reference type can have one of four *nullabilities*, which describes when warnings are generated:

- *Nonnullable*: Null can't be assigned to variables of this type. Variables of this type don't need to be null-checked before dereferencing.
- *Nullable*: Null can be assigned to variables of this type. Dereferencing variables of this type without first checking for `null` causes a warning.
- *Oblivious*: Oblivious is the pre-C# 8.0 state. Variables of this type can be dereferenced or assigned without warnings.
- *Unknown*: Unknown is generally for type parameters where constraints don't tell the compiler that the type must be *nullable* or *nonnullable*.

The nullability of a type in a variable declaration is controlled by the *nullable context* in which the variable is declared.

## Nullable contexts

Nullable contexts enable fine-grained control for how the compiler interprets reference type variables. The **nullable annotation context** of any given source line is either enabled or disabled. You can think of the pre-C# 8.0 compiler as compiling all your code in a disabled nullable context: any reference type may be null. The **nullable warnings context** may also be enabled or disabled. The nullable warnings context specifies the warnings generated by the compiler using its flow analysis.

The nullable annotation context and nullable warning context can be set for a project using the `Nullable` element in your *.csproj* file. This element configures how the compiler interprets the nullability of types and what warnings are generated. Valid settings are:

- `enable`: The nullable annotation context is **enabled**. The nullable warning context is **enabled**.
  - Variables of a reference type, `string` for example, are non-nullable.  All nullability warnings are enabled.
- `warnings`: The nullable annotation context is **disabled**. The nullable warning context is **enabled**.
  - Variables of a reference type are oblivious. All nullability warnings are enabled.
- `annotations`: The nullable annotation context is **enabled**. The nullable warning context is **disabled**.
  - Variables of a reference type, string for example, are non-nullable. All nullability warnings are disabled.
- `disable`: The nullable annotation context is **disabled**. The nullable warning context is **disabled**.
  - Variables of a reference type are oblivious, just like earlier versions of C#. All nullability warnings are disabled.

**Example**:

```xml
<Nullable>enable</Nullable>
```

You can also use directives to set these same contexts anywhere in your project:

- `#nullable enable`: Sets the nullable annotation context and nullable warning context to **enabled**.
- `#nullable disable`: Sets the nullable annotation context and nullable warning context to **disabled**.
- `#nullable restore`: Restores the nullable annotation context and nullable warning context to the project settings.
- `#nullable disable warnings`: Set the nullable warning context to **disabled**.
- `#nullable enable warnings`: Set the nullable warning context to **enabled**.
- `#nullable restore warnings`: Restores the nullable warning context to the project settings.
- `#nullable disable annotations`: Set the nullable annotation context to **disabled**.
- `#nullable enable annotations`: Set the nullable annotation context to **enabled**.
- `#nullable restore annotations`: Restores the annotation warning context to the project settings.

> [!IMPORTANT]
> The global nullable context does not apply for generated code files. Under either strategy, the nullable context is *disabled* for any source file marked as generated. This means any APIs in generated files are not annotated. There are four ways a file is marked as generated:
>
> 1. In the .editorconfig, specify `generated_code = true` in a section that applies to that file.
> 1. Put `<auto-generated>` or `<auto-generated/>` in a comment at the top of the file. It can be on any line in that comment, but the comment block must be the first element in the file.
> 1. Start the file name with *TemporaryGeneratedFile_*
> 1. End the file name with *.designer.cs*, *.generated.cs*, *.g.cs*, or *.g.i.cs*.
>
> Generators can opt-in using the [`#nullable`](language-reference/preprocessor-directives/preprocessor-nullable.md) preprocessor directive.

By default, nullable annotation and warning contexts are **disabled**, including new projects. That means that your existing code compiles without changes and without generating any new warnings.

These options provide two distinct strategies to [update an existing codebase](nullable-migration-strategies.md) to use nullable reference types.

## Nullable annotation context

The compiler uses the following rules in a disabled nullable annotation context:

- You can't declare nullable references in a disabled context.
- All reference variables may be assigned a value of null.
- No warnings are generated when a variable of a reference type is dereferenced.
- The null-forgiving operator may not be used in a disabled context.

The behavior is the same as previous versions of C#.

The compiler uses the following rules in an enabled nullable annotation context:

- Any variable of a reference type is a **non-nullable reference**.
- Any non-nullable reference may be dereferenced safely.
- Any nullable reference type (noted by `?` after the type in the variable declaration) may be null. Static analysis determines if the value is known to be non-null when it's dereferenced. If not, the compiler warns you.
- You can use the null-forgiving operator to declare that a nullable reference isn't null.

In an enabled nullable annotation context, the `?` character appended to a reference type declares a **nullable reference type**. The **null-forgiving operator** `!` may be appended to an expression to declare that the expression isn't null.

## Nullable warning context

The nullable warning context is distinct from the nullable annotation context. Warnings can be enabled even when the new annotations are disabled. The compiler uses static flow analysis to determine the **null state** of any reference. The null state is either **not null** or **maybe null** when the *nullable warning context* isn't **disabled**. If you dereference a reference when the compiler has determined it's **maybe null**, the compiler warns you. The state of a reference is **maybe null** unless the compiler can determine one of two conditions:

1. The variable has been definitely assigned a non-null value.
1. The variable or expression has been checked against null before de-referencing it.

The compiler generates warnings when you dereference a variable or expression that is **maybe null** in a nullable warning context. Furthermore, the compiler generates warnings when a nonnullable reference-type variable is assigned a **maybe null** variable or expression in an enabled nullable annotation context.

## Attributes describe APIs

You add attributes to APIs that provide the compiler more information about when arguments or return values can or can't be null. You can learn more about these attributes in our article in the language reference covering the [nullable attributes](language-reference/attributes/nullable-analysis.md). These attributes are being added to .NET libraries over current and upcoming releases. The most commonly used APIs are being updated first.

## Known pitfalls

Arrays and structs that contain reference types are known pitfalls in nullable reference types feature.

### Structs

A struct that contains non-nullable reference types allows assigning `default` for it without any warnings. Consider the following example:

```csharp
using System;

#nullable enable

public struct Student
{
    public string FirstName;
    public string? MiddleName;
    public string LastName;
}

public static class Program
{
    public static void PrintStudent(Student student)
    {
        Console.WriteLine($"First name: {student.FirstName.ToUpper()}");
        Console.WriteLine($"Middle name: {student.MiddleName.ToUpper()}");
        Console.WriteLine($"Last name: {student.LastName.ToUpper()}");
    }

    public static void Main() => PrintStudent(default);
}
```

In the preceding example, there is no warning in `PrintStudent(default)` while the non-nullable reference types `FirstName` and `LastName` are null.

Another more common case is when you deal with generic structs. Consider the following example:

```csharp
#nullable enable

public struct Foo<T>
{
    public T Bar { get; set; }
}

public static class Program
{
    public static void Main()
    {
        string s = default(Foo<string>).Bar;
    }
}
```

In the preceding example, the property `Bar` is going to be `null` at runtime, and it's assigned to non-nullable string without any warnings.

### Arrays

Arrays are also a known pitfall in nullable reference types. Consider the following example which doesn't produce any warnings:

```csharp
using System;

#nullable enable

public static class Program
{
    public static void Main()
    {
        string[] values = new string[10];
        string s = values[0];
        Console.WriteLine(s.ToUpper());
    }
}
```

In the preceding example, the declaration of the array shows it holds non-nullable strings, while its elements are all initialized to null. Then, the variable `s` is assigned a null value (the first element of the array). Finally, the variable `s` is dereferenced causing a runtime exception.

## See also

- [Draft nullable reference types specification](~/_csharplang/proposals/csharp-9.0/nullable-reference-types-specification.md)
- [Intro to nullable references tutorial](whats-new/tutorials/nullable-reference-types.md)
- [Migrate an existing codebase to nullable references](whats-new/tutorials/upgrade-to-nullable-references.md)
- [**Nullable** (C# Compiler option)](language-reference/compiler-options/language.md#nullable)
