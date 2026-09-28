# C# Style Guide

Write C# as an experienced professional developer maintaining a real production codebase.

The goal is clean, idiomatic, maintainable code—not code that looks generated, over-engineered, or written as a tutorial.

## General Principles

Prefer:

- Simple solutions over clever abstractions.
- Existing project conventions over personal preferences.
- Idiomatic modern C#.
- Small, focused changes.
- Clear code over explanatory comments.
- Direct implementation over speculative extensibility.
- Explicit domain terminology over generic abstraction names.

Do not refactor unrelated code unless required by the task.

Do not introduce new patterns, interfaces, helper classes, or abstraction layers unless they solve an actual problem.

## Match the Existing Codebase

Before writing code, inspect nearby files and follow their established conventions for:

- Naming
- Namespace style
- File organization
- Dependency injection
- Logging
- Exception handling
- Nullability
- Async code
- LINQ usage
- Constructor style
- Testing
- Formatting

Consistency with the repository is more important than imposing a preferred style.

## Naming

Use normal .NET naming conventions:

- `PascalCase` for classes, records, methods, properties, enums, and public members.
- `camelCase` for parameters and local variables.
- `_camelCase` for private instance fields if the project uses that convention.
- Interfaces use the `I` prefix where appropriate.
- Async methods end in `Async` when they actually perform asynchronous work.

Choose names based on the domain.

Good:

```csharp
weaponDefinition
assetPath
loadResult
ResolvePackageAsync()
ExportTextureAsync()
```

Avoid vague AI-style names such as:

```csharp
dataManager
processHandler
utilityHelper
resultProcessor
genericService
enhancedProcessor
```

Do not make names excessively descriptive when the surrounding context already makes their purpose obvious.

## Methods

Keep methods focused.

Prefer:

```csharp
var package = await provider.LoadPackageAsync(path, cancellationToken);
return mapper.Map(package);
```

over splitting trivial operations into unnecessary helpers.

Do not create a private method merely to avoid three or four readable lines of code.

Extract methods when they:

- Represent a meaningful operation.
- Are reused.
- Simplify genuinely complex logic.
- Improve testability.

Avoid methods named:

```csharp
ProcessData()
HandleStuff()
PerformOperation()
ExecuteLogic()
```

Use terminology that describes the actual domain operation.

## Comments

Do not narrate the code.

Bad:

```csharp
// Check if the package is null
if (package is null)
{
    // Return null because package was not found
    return null;
}
```

Good:

```csharp
if (package is null)
    return null;
```

Write comments only when they explain something the code itself cannot make obvious, such as:

- A non-obvious engine limitation.
- A protocol requirement.
- A workaround.
- An unusual performance decision.
- Why seemingly unnecessary behavior exists.

Prefer explaining **why**, not **what**.

Avoid comments such as:

```csharp
// Initialize the list
// Iterate through the items
// Return the result
// Handle errors
// Create a new instance
```

Do not add XML documentation to every member automatically.

Use XML documentation for public APIs when the project expects it or when behavior is not obvious from the API itself.

## Control Flow

Prefer guard clauses over deeply nested conditions.

Good:

```csharp
if (asset is null)
    return null;

if (!asset.IsLoaded)
    await asset.LoadAsync(cancellationToken);

return asset;
```

Avoid:

```csharp
if (asset is not null)
{
    if (!asset.IsLoaded)
    {
        await asset.LoadAsync(cancellationToken);
    }

    return asset;
}

return null;
```

Avoid unnecessary `else` blocks after `return`, `throw`, `continue`, or `break`.

## Types and `var`

Use `var` when the type is obvious from the right-hand side or improves readability.

Good:

```csharp
var package = new PackageInfo();
var assets = provider.GetAssets();
```

Use the explicit type when it communicates useful information:

```csharp
IReadOnlyList<WeaponDefinition> weapons = parser.ParseWeapons(data);
```

Do not mechanically use either style everywhere.

## Collections

Use collection expressions when supported by the project:

```csharp
private static readonly string[] SupportedExtensions =
[
    ".uasset",
    ".umap"
];
```

Prefer collection interfaces that communicate intent:

```csharp
IReadOnlyList<AssetInfo>
IEnumerable<PackageEntry>
IReadOnlyDictionary<string, WeaponDefinition>
```

Do not expose mutable collections without a reason.

## LINQ

Use LINQ when it makes an operation clearer.

Good:

```csharp
var weapons = assets
    .Where(asset => asset.Type == AssetType.Weapon)
    .Select(ParseWeapon)
    .ToArray();
```

Do not build long LINQ pipelines when straightforward loops are easier to understand.

Avoid LINQ purely to make code shorter.

## Null Handling

Respect nullable reference types when enabled.

Prefer explicit handling:

```csharp
var asset = provider.FindAsset(path);

if (asset is null)
    return AssetResult.NotFound(path);
```

Do not scatter null-forgiving operators throughout the code:

```csharp
asset!.Package!.Exports!.First()!
```

Use `!` only when the invariant is genuinely guaranteed and cannot reasonably be expressed to the compiler.

Do not add redundant null checks for values whose contracts guarantee non-null values.

## Exceptions

Use exceptions for exceptional conditions, not normal control flow.

Catch exceptions only when you can:

- Add useful context.
- Recover.
- Translate the exception into a meaningful domain error.
- Perform required cleanup.

Avoid:

```csharp
try
{
    ...
}
catch (Exception ex)
{
    Console.WriteLine(ex);
}
```

Do not silently swallow exceptions.

Do not wrap every method in `try/catch`.

Preserve the original exception when wrapping:

```csharp
throw new PackageLoadException(path, ex);
```

## Async Code

Use async only for genuinely asynchronous work.

Do not write:

```csharp
public async Task<int> GetCountAsync()
{
    return await Task.FromResult(_items.Count);
}
```

Prefer:

```csharp
public int GetCount() => _items.Count;
```

Propagate `CancellationToken` through asynchronous operations when the surrounding API supports cancellation.

Do not add `Task.Run` around naturally asynchronous I/O.

Avoid `.Result`, `.Wait()`, and `.GetAwaiter().GetResult()` unless synchronous bridging is unavoidable.

## Dependencies

Prefer constructor injection for actual dependencies.

Do not create interfaces solely because a class exists.

Avoid this:

```csharp
IWeaponParser
WeaponParser
WeaponParserFactory
WeaponParserProvider
WeaponParserService
```

when this is sufficient:

```csharp
WeaponParser
```

Introduce interfaces when there are multiple implementations, a meaningful architectural boundary, or a testing requirement that justifies one.

## Classes

Prefer cohesive classes with clear responsibilities.

Do not turn every concept into a class.

Use:

- `record` for immutable data-oriented models where appropriate.
- `record struct` or `readonly struct` for small value types when justified.
- `sealed` when inheritance is not intended and the codebase commonly uses it.

Avoid inheritance unless the domain genuinely benefits from it.

Favor composition.

## Constructors

Use primary constructors when they improve readability and match the project's target C# version and conventions.

For example:

```csharp
internal sealed class PackageReader(IFileProvider provider)
{
    public async Task<Package> ReadAsync(
        string path,
        CancellationToken cancellationToken = default)
    {
        return await provider.ReadPackageAsync(path, cancellationToken);
    }
}
```

Do not convert existing constructor styles solely to use newer syntax.

## Properties

Use expression-bodied members for genuinely simple members:

```csharp
public bool IsLoaded => _package is not null;
```

Do not force complex logic into expression-bodied members.

## Constants and Magic Values

Give meaningful names to values when their meaning is not obvious.

Good:

```csharp
private const int PackageMagic = unchecked((int)0x9E2A83C1);
```

Do not create constants for trivial values simply to avoid literals:

```csharp
private const int Zero = 0;
```

## Logging

Use the project's logging abstraction.

Log meaningful events and failures.

Avoid logging every method entry, branch, and successful operation.

Bad:

```csharp
_logger.LogInformation("Starting ProcessWeapon");
_logger.LogInformation("Weapon processed successfully");
```

Better:

```csharp
_logger.LogWarning(
    "Unable to resolve weapon asset {AssetPath}",
    assetPath);
```

Do not log the same exception at multiple layers unless each layer adds meaningful context.

## Validation

Validate data at system boundaries.

Examples:

- API input
- Configuration
- External files
- User-provided paths
- Network responses
- Plugin/MCP inputs

Do not repeatedly validate the same invariant throughout internal code.

## Error Messages

Write actionable error messages.

Good:

```text
Unable to load package '/Game/Weapons/Rifle_A'. The package was not found in the mounted archives.
```

Bad:

```text
An error occurred while processing data.
```

Include useful identifiers such as paths, asset names, IDs, or operation names when appropriate.

## Formatting

Use the repository's `.editorconfig` and formatter.

Do not manually introduce formatting conventions that conflict with the project.

Keep code visually compact without sacrificing readability.

Avoid excessive blank lines.

Do not place comments between every logical operation.

## Avoid AI-Looking Structure

Do not automatically structure every implementation as:

```text
Manager
Service
Provider
Handler
Factory
Helper
Utility
Processor
Coordinator
Context
Result
```

Use these names only when they accurately describe an established architectural role.

Do not produce symmetrical architecture just because it looks clean on paper.

Real production code should reflect the actual problem.

## Avoid Premature Abstraction

Do not design for hypothetical future requirements.

If one implementation exists, implement it directly unless there is evidence another implementation is needed.

Do not add:

- Generic repositories without a need.
- Factory patterns for one implementation.
- Strategy patterns for one algorithm.
- Interfaces for every class.
- Builders for simple constructors.
- Custom result wrappers when the codebase already has an error-handling convention.
- Extension methods used only once.
- Utility classes containing unrelated operations.

## Do Not Rewrite Working Code Unnecessarily

When modifying an existing feature:

1. Identify the smallest responsible area.
2. Preserve unrelated behavior.
3. Match the existing architecture.
4. Change only what the task requires.
5. Add or update tests for behavior that changed.

Avoid large cleanup refactors unless specifically requested.

## Tests

Tests should verify behavior, not implementation details.

Use descriptive test names consistent with the repository.

Example:

```csharp
[Fact]
public void ParseWeapon_ReturnsNull_WhenAssetHasNoWeaponDefinition()
```

or, if the repository uses another convention:

```csharp
[Fact]
public void ParseWeapon_WithMissingDefinition_ReturnsNull()
```

Avoid excessive setup abstractions for simple tests.

Do not mock everything automatically.

Prefer real lightweight objects when practical.

## Modern C#

Use modern C# features when they genuinely improve readability and the project's target framework supports them:

- Pattern matching
- Switch expressions
- Records
- Collection expressions
- `required`
- File-scoped namespaces
- `init`
- Nullable reference types
- Primary constructors
- `await foreach`
- `IAsyncEnumerable<T>`

Do not use new language features merely to make the code look sophisticated.

## Before Finishing

Review the change and remove:

- Redundant comments.
- Unnecessary abstractions.
- Unused helpers.
- Duplicate validation.
- Generic AI-style names.
- Excessive logging.
- Unnecessary `try/catch`.
- Redundant null checks.
- Speculative extensibility.
- Unrelated refactors.

Run the repository's formatter, compiler, analyzers, and relevant tests.

The final code should look like it naturally belongs in the repository rather than like a standalone generated solution.