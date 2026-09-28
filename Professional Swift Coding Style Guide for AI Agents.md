# Swift Coding Style Guide

Write Swift as an experienced professional Swift developer maintaining a real production codebase.

The goal is clean, idiomatic, maintainable Swift—not code that looks generated, over-engineered, excessively protocol-oriented, or mechanically translated from Objective-C, Java, C#, Kotlin, or TypeScript.

The most important rule:

> Do not optimize for demonstrating Swift features, protocol-oriented programming, concurrency, or architecture. Optimize for producing the smallest idiomatic production-quality change that an experienced Swift maintainer would reasonably write.

## General Principles

Prefer:

- Existing project conventions over personal preferences.
- Value semantics where they naturally fit.
- Concrete types until abstraction is justified.
- Small focused types.
- Enums for meaningful state.
- Optionals for legitimate absence.
- Strong typing.
- Explicit ownership and lifecycle.
- `async`/`await` for asynchronous workflows.
- Structured concurrency.
- Composition over inheritance.
- Simple protocols at actual abstraction boundaries.
- Standard-library and Apple framework APIs.
- Clear error propagation.
- Small focused changes.
- Direct implementation over speculative extensibility.

Avoid:

- Protocols for every implementation.
- `Manager`, `Service`, `Provider`, `Repository`, and `Factory` layers without real responsibilities.
- Giant protocol hierarchies.
- `!` used to avoid proper optional handling.
- `try!` in production paths.
- `as!` used to silence type problems.
- Classes when value semantics are more appropriate.
- Reference semantics by default.
- Singletons as dependency injection.
- Massive ObservableObjects.
- Unstructured `Task` creation.
- Global actors applied indiscriminately.
- Excessive Combine pipelines.
- Callbacks where async/await is clearer.
- Excessive property wrappers.
- Giant SwiftUI `body` implementations.
- Tiny view extraction with no semantic value.
- Unnecessary UIKit abstraction wrappers.
- Clever language features that make simple behavior difficult to read.

Do not refactor unrelated code unless required by the task.

---

## Match the Existing Codebase

Before writing Swift, inspect nearby files and follow the repository's established conventions for:

- Swift version.
- Deployment targets.
- SwiftUI vs UIKit/AppKit.
- Architecture.
- State management.
- Dependency injection.
- Concurrency.
- Error handling.
- Networking.
- Persistence.
- Navigation.
- Naming.
- Formatting.
- Linting.
- Testing.
- Package management.
- Generated code.
- Objective-C interoperability.

Check relevant project files such as:

```text
Package.swift
Package.resolved
project.pbxproj
.xcconfig
.swiftlint.yml
.swiftformat
Tuist/
Project.swift
Package.swift
Podfile
Cartfile
```

Also inspect:

```text
Sources/
Tests/
App/
Features/
Frameworks/
```

and nearby implementation files.

Consistency with the existing repository is more important than introducing a preferred architecture.

Do not migrate:

- UIKit to SwiftUI.
- Combine to async/await.
- MVC to MVVM.
- MVVM to TCA.
- CocoaPods to Swift Package Manager.
- Closures to actors.
- Classes to structs.

as unrelated cleanup.

---

## Swift Version

Determine the project's supported Swift language version before using newer syntax or concurrency behavior.

Check:

- Xcode version.
- Swift package tools version.
- CI configuration.
- Project build settings.
- `swift-tools-version`.

Do not assume the latest Swift version.

Features and semantics around:

- Strict concurrency.
- `Sendable`.
- Typed throws.
- Observation.
- Macros.
- Existentials.
- Parameter packs.
- Ownership.

vary by Swift version and compiler mode.

Use only what the project supports.

---

## Deployment Targets

Check supported:

```text
iOS
macOS
watchOS
tvOS
visionOS
```

versions before using newer APIs.

Do not assume a framework API exists merely because the current SDK contains it.

Follow the project's availability strategy.

---

## Availability

Use:

```swift
if #available(iOS 18, *) {
    // ...
}
```

when runtime availability genuinely differs.

Use:

```swift
@available(...)
```

where API availability should be declared.

Do not scatter availability checks unnecessarily when the deployment target already guarantees support.

---

## Naming

Follow Swift API Design Guidelines and project conventions.

Use:

- `UpperCamelCase` for types.
- `lowerCamelCase` for methods, properties, variables, and cases.
- Descriptive argument labels.
- Boolean names that read naturally as assertions.

Good:

```swift
weaponDefinition
assetPath
packageName
loadResult

loadPackage()
resolveAsset()
parseWeaponData()
```

Avoid vague AI-style names:

```swift
dataManager
processHandler
utilityHelper
genericService
resultProcessor
enhancedProcessor
operationManager
```

Prefer domain terminology.

Bad:

```swift
let data = getData()
let result = processData(data)
```

Better:

```swift
let weapon = loadWeaponDefinition()
let stats = parseWeaponStats(weapon)
```

---

## Avoid Verbose Names

Swift APIs should read naturally.

Avoid:

```swift
let successfullyParsedWeaponConfigurationResult =
    parseWeaponConfiguration()
```

Prefer:

```swift
let weaponConfig = parseWeaponConfiguration()
```

Names should provide useful context without restating the entire type or operation.

---

## Argument Labels

Use labels so calls read naturally.

Good:

```swift
loadWeapon(at: path)
moveWeapon(from: source, to: destination)
```

Avoid:

```swift
loadWeapon(path: path)
```

if:

```swift
loadWeapon(at: path)
```

better expresses the relationship.

Follow existing project style.

---

## Boolean Naming

Prefer names that read naturally:

```swift
isLoaded
hasMetadata
canRetry
shouldRefresh
```

Avoid:

```swift
loadedFlag
checkEnabled
statusBool
```

A boolean should usually answer a question.

---

## Functions First

Use free functions or static helpers for operations that do not require object identity or stored state.

Good:

```swift
func parseWeaponStats(
    from data: WeaponData
) -> WeaponStats {
    WeaponStats(
        damage: data.damage,
        fireRate: data.fireRate
    )
}
```

Do not automatically create:

```swift
final class WeaponStatsParser {
    func parse(
        _ data: WeaponData
    ) -> WeaponStats {
        // ...
    }
}
```

unless the parser owns actual:

- State.
- Dependencies.
- Configuration.
- Lifecycle.
- Related behavior.

---

## Structs vs Classes

Prefer `struct` when the type represents a value.

Good candidates include:

- Models.
- Configuration.
- Coordinates.
- Parsed data.
- Request/response values.
- View state.
- Immutable domain values.

Use `class` when:

- Identity matters.
- Shared mutable state matters.
- Framework inheritance requires it.
- Objective-C interoperability requires it.
- Lifetime/deinitialization behavior matters.

Do not use classes merely because the type contains methods.

---

## Value Semantics

Swift's value semantics are a major strength.

Prefer:

```swift
struct Weapon {
    let id: WeaponID
    let name: String
}
```

when independent copies naturally represent independent values.

Do not turn every model into:

```swift
final class Weapon
```

without a reason.

---

## Reference Semantics

Use classes deliberately when multiple parts of the system should observe or mutate the same identity.

Examples may include:

- Controllers.
- Shared stores.
- Coordinators.
- Framework objects.
- Resource owners.

Do not reach for classes solely because mutation seems easier.

---

## `final`

Mark classes `final` when subclassing is not intended and project conventions support it.

Good:

```swift
final class PackageLoader {
}
```

Do not mark everything `final` mechanically if extension/subclassing is part of the API.

---

## Inheritance

Prefer composition over inheritance.

Use inheritance when:

- UIKit/AppKit requires it.
- The domain genuinely models substitutable subclasses.
- Existing framework architecture requires it.

Do not create abstract-style class trees merely to share a few helper methods.

---

## NSObject

Do not inherit from:

```swift
NSObject
```

unless required for:

- Objective-C interoperability.
- KVO.
- Selectors.
- Cocoa framework APIs.
- Runtime mechanisms requiring it.

Pure Swift types usually do not need NSObject.

---

## Models

Keep simple models simple.

Good:

```swift
struct WeaponStats: Equatable {
    let damage: Double
    let fireRate: Double
    let magazineSize: Int
}
```

Do not automatically add:

```text
Identifiable
Hashable
Codable
Sendable
Comparable
CustomStringConvertible
```

unless the type actually needs them.

Every conformance becomes part of the type's behavior.

---

## Memberwise Initialization

Use synthesized memberwise initializers where appropriate.

Do not write:

```swift
init(
    damage: Double,
    fireRate: Double
) {
    self.damage = damage
    self.fireRate = fireRate
}
```

if synthesis already provides exactly the required behavior.

Write custom initializers when they establish invariants or improve API design.

---

## Initializers

Initializers should establish a valid object.

Avoid hidden side effects such as:

- Network requests.
- File I/O.
- Starting tasks.
- Registering global observers.

inside ordinary model initializers.

Use explicit asynchronous factories or methods where initialization requires asynchronous work.

---

## Failable Initializers

Use:

```swift
init?(...)
```

when invalid input naturally means construction can fail without detailed error information.

Use throwing initializers when callers need failure details.

Do not choose one mechanically.

---

## Throwing Initializers

Example:

```swift
init(data: Data) throws {
    // ...
}
```

is appropriate when parsing failure has meaningful error information.

Do not replace straightforward optional construction with elaborate error hierarchies when no caller needs details.

---

## Static Factory Methods

Use a static factory when creation:

- Is asynchronous.
- Chooses a concrete implementation.
- Requires substantial validation.
- Produces cached values.

Do not create:

```swift
WeaponFactory.createWeapon(...)
```

merely to call a normal initializer.

---

## Protocols

Protocols are powerful but frequently overused in generated Swift.

Create a protocol when:

- Multiple implementations genuinely exist.
- A consumer benefits from substitution.
- Framework architecture requires it.
- A reusable behavioral contract exists.
- Testing and production architecture both benefit from the boundary.

Do not create a protocol for every concrete type.

---

## Avoid Protocol-for-Every-Class Architecture

Bad:

```swift
protocol WeaponParserProtocol {
    func parse(_ data: Data) throws -> Weapon
}

final class WeaponParser: WeaponParserProtocol {
}
```

when there is one parser and no consumer requires abstraction.

Prefer:

```swift
struct WeaponParser {
    func parse(_ data: Data) throws -> Weapon {
        // ...
    }
}
```

or a function.

---

## Protocol Names

Protocols describing capabilities should usually read naturally:

```swift
Sequence
Identifiable
Cacheable
Persisting
WeaponLoading
```

Avoid mechanically suffixing everything with:

```text
Protocol
Interface
Abstract
```

unless the repository already follows that style.

---

## Small Protocols

Prefer narrow interfaces.

Good:

```swift
protocol WeaponLoading {
    func loadWeapon(
        id: WeaponID
    ) async throws -> Weapon?
}
```

Avoid giant protocols:

```swift
protocol WeaponRepository {
    func create(...)
    func update(...)
    func delete(...)
    func find(...)
    func findAll(...)
    func search(...)
    func cache(...)
    func clearCache(...)
    func sync(...)
}
```

unless every conformer and consumer genuinely shares that contract.

---

## Define Protocols Where They Matter

Prefer protocols designed around consumer needs rather than implementation taxonomy.

Do not create giant implementation-owned abstractions simply because dependency injection might someday need them.

Start concrete.

Extract the smallest useful protocol when a real boundary appears.

---

## Protocol Composition

Use:

```swift
SomeProtocol & AnotherProtocol
```

when the combined capability genuinely matters.

Do not create giant composition chains that make signatures difficult to understand.

---

## Protocol Extensions

Use protocol extensions for genuinely shared default behavior.

Do not hide large portions of business logic in default protocol implementations.

This can make dynamic dispatch and conformer behavior difficult to understand.

---

## Avoid Faux Inheritance Through Protocol Extensions

Do not turn:

```swift
protocol BaseService {}
extension BaseService {
    // dozens of default methods
}
```

into a replacement for a base class.

If many types require a shared implementation, composition may be clearer.

---

## Existentials

Use `any Protocol` when runtime existential storage is required.

Example:

```swift
let loader: any WeaponLoading
```

Do not replace concrete types with `any Protocol` everywhere for abstractness.

Existentials introduce dynamic dispatch and erase concrete type information.

---

## Opaque Types

Use:

```swift
some Protocol
```

when callers should know there is one concrete type while the implementation remains hidden.

SwiftUI's:

```swift
some View
```

is the obvious example.

Do not use opaque result types merely to appear modern.

---

## `some` vs `any`

Understand the difference.

Broadly:

```swift
some P
```

preserves one hidden concrete conforming type.

```swift
any P
```

stores any conforming type at runtime.

Do not substitute one for the other without understanding the API semantics.

---

## Generics

Use generics when behavior truly applies across multiple types.

Good:

```swift
func firstMatch<T>(
    in values: [T],
    where predicate: (T) -> Bool
) -> T? {
    values.first(where: predicate)
}
```

Do not genericize domain-specific code solely to avoid repetition.

---

## Generic Constraints

Keep generic constraints readable.

Avoid signatures with several:

```text
where
associatedtype
same-type constraints
conditional conformances
```

for a function used with one concrete domain type.

Genericity should simplify real reuse.

---

## Associated Types

Use associated types where the protocol genuinely has type-dependent behavior.

Do not create PAT-heavy architectures when a concrete generic type or closure would be easier.

---

## Type Erasure

Avoid custom:

```text
AnySomething
```

type-erasure wrappers unless existential limitations or API requirements genuinely require them.

Modern Swift often supports simpler existential use than older versions did.

Do not carry old type-erasure patterns forward without checking whether they remain necessary.

---

## Enums

Use enums aggressively for real finite state.

Good:

```swift
enum LoadState {
    case idle
    case loading
    case loaded(Weapon)
    case failed(Error)
}
```

This is often better than:

```swift
struct LoadState {
    var isLoading: Bool
    var weapon: Weapon?
    var error: Error?
}
```

because the latter can represent contradictory states.

---

## Associated Values

Use associated values when state variants naturally carry different data.

Example:

```swift
enum AuthenticationState {
    case signedOut
    case signingIn
    case signedIn(User)
    case failed(AuthenticationError)
}
```

Do not create a class hierarchy for a small closed set of states.

---

## Raw-Value Enums

Use raw values for serialization/interoperability when appropriate.

Example:

```swift
enum WeaponCategory: String, Codable {
    case rifle
    case shotgun
    case sniper
}
```

Do not use raw values merely because enums support them.

---

## Avoid Stringly Typed State

Bad:

```swift
if status == "loading" {
}
```

when the set is known.

Prefer:

```swift
if status == .loading {
}
```

Enums provide stronger guarantees.

---

## Optionals

Use optionals when absence is a legitimate state.

Good:

```swift
func weapon(
    id: WeaponID
) -> Weapon?
```

if missing is expected.

Do not use magic values such as:

```text
""
-1
0
```

to represent absence when `Optional` expresses the contract better.

---

## Avoid Unnecessary Optionals

Do not make values optional simply to simplify initialization.

Bad:

```swift
var repository: Repository!
var configuration: Configuration?
```

when the object cannot function without them.

Require dependencies in the initializer.

---

## Optional Binding

Prefer:

```swift
if let weapon {
    // ...
}
```

or:

```swift
guard let weapon else {
    return
}
```

when supported and consistent with the project's Swift version.

Do not force older verbose syntax if the codebase uses modern shorthand.

---

## Guard Statements

Use `guard` for required preconditions and early exits.

Good:

```swift
guard let weapon else {
    return
}

render(weapon)
```

Avoid deep nesting.

---

## Avoid Guard Chains Everywhere

A method containing fifteen guards may be difficult to follow.

If several checks represent one validation concept, consider centralizing the validation.

Use guard when it clarifies control flow, not mechanically.

---

## Force Unwrapping

Avoid:

```swift
value!
```

unless failure would truly indicate a programmer invariant and the invariant is obvious.

Acceptable places may include:

- Certain test setup.
- Static resources guaranteed by application packaging.
- Framework outlets under established lifecycle guarantees.

Even there, follow repository conventions.

Do not use `!` merely to silence the compiler.

---

## Implicitly Unwrapped Optionals

Use:

```swift
Type!
```

only where lifecycle/framework requirements genuinely justify it.

Classic UIKit outlets may use them.

Do not model ordinary dependencies or state as IUOs.

---

## `try!`

Avoid:

```swift
try!
```

in production paths.

Use only when failure is provably impossible and that invariant is clear.

Examples may include static regex/resource creation where invalidity would be a programmer bug.

Prefer an explicit invariant over casual force-trying.

---

## `as!`

Avoid forced casts.

Bad:

```swift
let weapon = value as! Weapon
```

if runtime type may vary.

Prefer:

```swift
guard let weapon = value as? Weapon else {
    // handle mismatch
}
```

Better still, avoid untyped values where possible.

---

## Nil-Coalescing

Use:

```swift
value ?? defaultValue
```

when absence legitimately maps to the default.

Do not use defaults to hide missing required configuration or malformed external data.

---

## Optional Chaining

Use:

```swift
weapon?.metadata?.name
```

when missing intermediate values are valid.

Do not build long optional chains to hide invalid internal state.

Normalize or validate required data once.

---

## `map` on Optional

Optional transformations can be elegant:

```swift
let name = weapon.map(\.name)
```

or:

```swift
let title = name.map(normalize)
```

when clear.

Do not turn straightforward optional logic into dense functional chains for style points.

---

## Error Handling

Use Swift's throwing model deliberately.

Good:

```swift
func loadWeapon(
    at url: URL
) throws -> Weapon
```

when failure is meaningful.

Do not convert every possible missing value into an exception.

Use optionals for expected absence.

---

## Error Types

Use domain-specific error enums where callers benefit from structured failure handling.

Example:

```swift
enum PackageLoadError: Error {
    case notFound(URL)
    case invalidFormat(URL)
    case unreadable(URL, underlying: Error)
}
```

Do not create huge application-wide error enums containing every possible subsystem error.

Keep errors close to meaningful boundaries.

---

## Error Context

Preserve underlying errors when useful.

Do not flatten everything into:

```swift
throw AppError(
    message: error.localizedDescription
)
```

if structured error information is useful.

---

## Localized Errors

Use `LocalizedError` when errors genuinely need user-facing descriptions.

Do not make every internal error conform merely to provide a string.

Keep technical errors separate from presentation when appropriate.

---

## Catch Specific Errors

Prefer:

```swift
do {
    // ...
} catch PackageLoadError.notFound {
    // ...
}
```

or typed handling where appropriate.

Do not catch everything merely to return nil.

---

## Avoid Empty Catch

Bad:

```swift
do {
    try save()
} catch {
}
```

Never silently swallow failures unless ignoring them is an explicit, understood contract.

---

## `try?`

Use:

```swift
try?
```

when converting any failure into nil is genuinely correct.

Do not use it simply to avoid handling errors.

Bad:

```swift
let weapon = try? importantOperation()
```

if callers need to know why the operation failed.

---

## `defer`

Use `defer` for cleanup that should happen when leaving scope.

Example:

```swift
lock.lock()
defer { lock.unlock() }
```

Do not use it for ordinary sequential work if it makes execution order harder to understand.

---

## Async/Await

Prefer async/await for modern asynchronous workflows where supported by the project.

Good:

```swift
func loadWeapon(
    id: WeaponID
) async throws -> Weapon {
    try await client.weapon(id: id)
}
```

Do not wrap synchronous work in async functions unnecessarily.

---

## Avoid Async for Synchronous Work

Bad:

```swift
func weaponCount(
    _ weapons: [Weapon]
) async -> Int {
    weapons.count
}
```

Prefer:

```swift
func weaponCount(
    _ weapons: [Weapon]
) -> Int {
    weapons.count
}
```

---

## Structured Concurrency

Prefer structured concurrency where task lifetime follows lexical scope.

Use:

```swift
async let
withTaskGroup
withThrowingTaskGroup
```

when concurrent child operations belong to the current operation.

Avoid detached/unstructured work unless independence is intentional.

---

## `Task`

Use `Task` when bridging synchronous code into asynchronous work or creating UI-owned asynchronous operations.

Before writing:

```swift
Task {
    await work()
}
```

ask:

- Who owns this task?
- Should it be cancelled?
- What happens when the view disappears?
- Where do errors go?
- Does the caller need completion?

Do not use Task as a generic substitute for calling an async function correctly.

---

## Detached Tasks

Avoid:

```swift
Task.detached
```

unless the operation genuinely should not inherit actor context, task-local values, priority, or cancellation.

Detached tasks are advanced tools.

They should be rare in application code.

---

## Cancellation

Cooperative cancellation matters.

For long-running operations, check:

```swift
Task.isCancelled
```

or:

```swift
try Task.checkCancellation()
```

where appropriate.

Do not add cancellation checks to tiny operations that immediately complete.

---

## Task Handles

Retain task handles when lifecycle ownership requires cancellation.

Example:

```swift
private var loadTask: Task<Void, Never>?
```

may be appropriate for a controller or view model.

Cancel superseded work when necessary.

Do not store every Task automatically.

---

## Async Let

Use:

```swift
async let weapon = loadWeapon()
async let metadata = loadMetadata()

return try await (weapon, metadata)
```

when operations are independent.

Do not parallelize work if:

- One operation depends on another.
- Ordering matters.
- Resource usage must be bounded.
- A service cannot handle concurrency.

---

## Task Groups

Use task groups for dynamic collections of concurrent child work.

Do not create a task group to process three cheap synchronous values.

Concurrency has overhead and semantics.

---

## Actors

Use actors for mutable state that genuinely requires concurrent isolation.

Example:

```swift
actor PackageCache {
    private var packages: [URL: Package] = [:]

    func package(for url: URL) -> Package? {
        packages[url]
    }
}
```

Do not convert every class into an actor merely because concurrency exists somewhere in the application.

---

## Actor Isolation

Understand what actor isolation protects.

Do not assume actors automatically make returned mutable reference objects thread-safe.

Isolation applies to actor-isolated state and accesses.

---

## `@MainActor`

Use `@MainActor` for code that truly belongs to UI/main-actor state.

Typical examples:

- View models directly driving UI.
- UI controllers.
- UI mutation APIs.

Do not annotate entire networking or domain layers `@MainActor` simply to silence concurrency diagnostics.

---

## Avoid MainActor as a Compiler Fix

Bad:

```swift
@MainActor
final class NetworkClient {
    // ...
}
```

merely because Sendable errors were inconvenient.

Fix isolation and ownership correctly.

---

## Main Actor Hopping

Use:

```swift
await MainActor.run {
    // update UI state
}
```

when an operation legitimately needs to update main-actor state from elsewhere.

Do not hop to the main actor unnecessarily.

---

## Sendable

Conform to `Sendable` when values genuinely cross concurrency domains safely.

Simple immutable structs often fit naturally.

Do not add:

```swift
@unchecked Sendable
```

merely to silence the compiler.

That transfers correctness responsibility to the developer.

---

## `@unchecked Sendable`

Use only when thread safety is manually guaranteed and understood.

Document non-obvious synchronization invariants.

If you cannot explain why it is safe, do not use it.

---

## Global Actors

Define custom global actors only when the application has a genuine process-wide isolation domain.

Do not create:

```text
DatabaseActor
NetworkActor
CacheActor
EverythingActor
```

merely to make concurrency compile.

---

## Locks

Locks remain useful for small synchronous critical sections.

Do not replace every lock with an actor automatically.

Likewise, do not use locks when actor isolation already owns the state.

Choose based on API and execution model.

---

## `NSLock`

When used:

```swift
lock.lock()
defer { lock.unlock() }
```

can provide clear cleanup.

Keep critical sections short.

Do not hold locks while performing slow async work.

---

## Continuations

Use checked continuations to bridge callback APIs into async/await.

Prefer:

```swift
withCheckedContinuation
withCheckedThrowingContinuation
```

during development and normal application code.

Do not use unsafe continuations unless overhead has been measured and correctness is guaranteed.

---

## Resume Exactly Once

A continuation must be resumed exactly once.

Do not create bridging code where multiple callback branches can accidentally resume twice or never resume.

This is a correctness invariant.

---

## Combine

Use Combine when:

- The project already uses it.
- Multiple asynchronous streams benefit from operators.
- Reactive composition genuinely improves the workflow.

Do not introduce Combine for a single one-shot request that async/await handles more clearly.

---

## Avoid Combine for Everything

Bad conceptual pattern:

```text
Just
map
flatMap
catch
eraseToAnyPublisher
sink
```

around ordinary one-time operations.

Use the simplest asynchronous abstraction.

---

## Publisher Types

Avoid `eraseToAnyPublisher()` automatically.

Type erasure can be useful for public API stability, but it also hides implementation and adds abstraction.

Use it where it solves an actual API problem.

---

## AnyCancellable

Store subscriptions according to ownership.

Do not create global cancellable bags.

Subscriptions should usually belong to the object whose lifecycle controls them.

---

## Retain Cycles in Closures

Be mindful of closure capture semantics.

This can create a cycle:

```swift
service.onUpdate = {
    self.refresh()
}
```

when `self` owns `service`.

Use:

```swift
[weak self]
```

where the ownership relationship requires it.

Do not use weak self mechanically in every closure.

---

## Weak Self

Use weak captures when the closure should not extend the object's lifetime.

Do not automatically write:

```swift
[weak self]
guard let self else { return }
```

in short-lived non-escaping closures or Task contexts without understanding ownership.

Sometimes strong capture is correct.

---

## Unowned

Use:

```swift
unowned
```

only when the referenced object is guaranteed to outlive the reference.

If that invariant can fail, the application crashes.

Prefer weak references when lifetime is uncertain.

---

## Escaping Closures

Use `@escaping` only when the closure is stored or executed after the function returns.

Do not add it unless required.

---

## Closures

Use closures for local behavior and callbacks.

Do not move substantial business logic into giant inline closures.

Extract meaningful named operations when the closure becomes difficult to understand.

---

## Trailing Closures

Use trailing closure syntax when it improves readability.

Do not create ambiguous multiple-trailing-closure APIs merely because Swift supports them.

API calls should remain easy to understand.

---

## Capture Lists

Use capture lists to express ownership deliberately.

Do not add:

```swift
[weak self]
```

because a linter or habit says every closure needs it.

Understand whether the closure escapes and who retains it.

---

## Collections

Use standard collections:

```text
Array
Dictionary
Set
```

according to semantics.

Do not wrap them in custom collection types without domain behavior or invariants.

---

## Arrays

Use arrays for ordered collections.

Do not use linked structures or custom collections without a measured need.

Swift arrays are efficient value types with copy-on-write behavior.

---

## Copy-on-Write

Understand that standard Swift collections use copy-on-write.

Do not manually clone arrays/dictionaries to achieve independent value semantics.

Assignment already behaves as value semantics.

---

## Dictionary Lookup

Dictionary subscripting returns an optional.

Use:

```swift
guard let weapon = weapons[id] else {
    return nil
}
```

when presence matters.

Do not immediately force unwrap:

```swift
weapons[id]!
```

unless the invariant is truly proven.

---

## Dictionary Defaults

Use:

```swift
counts[key, default: 0] += 1
```

where appropriate.

This is clearer than repeated optional lookup for simple accumulation.

---

## Sets

Use sets for uniqueness and membership.

Do not use them for tiny collections solely because lookup is theoretically faster.

Choose semantics first.

---

## Sequence APIs

Use:

```text
map
compactMap
filter
reduce
first(where:)
contains(where:)
```

when they clearly express transformations.

Do not force every loop into a functional chain.

---

## `compactMap`

Use when transforming and removing nil results are one natural operation.

Do not use it merely to hide unexpected nils.

---

## `reduce`

Use for real reductions.

Avoid clever reduce-based dictionary/list construction if a loop is easier to read.

Swift code does not need maximum functional density.

---

## Loops

Plain loops are perfectly idiomatic.

Good:

```swift
var weapons: [Weapon] = []

for asset in assets {
    guard asset.kind == .weapon else {
        continue
    }

    weapons.append(
        try parseWeapon(asset)
    )
}
```

Do not contort clear stateful logic into chained sequence operations.

---

## `forEach`

Be cautious with:

```swift
values.forEach { ... }
```

when you need:

- `break`.
- `continue`.
- Early return semantics.

A normal `for` loop is often clearer.

---

## Key Paths

Key paths can improve standard operations:

```swift
weapons.map(\.name)
```

Use them where simple.

Do not design elaborate key-path-driven abstractions for ordinary domain logic.

---

## Strings

Swift Strings are Unicode-correct collections, not byte arrays.

Do not assume integer indexing.

Use proper String indices when manipulating user-visible text.

---

## String Indexing

Avoid complicated manual String index manipulation when standard APIs such as:

```swift
split
prefix
suffix
range(of:)
```

solve the problem.

String indexing is intentionally not integer-based.

---

## Substring

Remember:

```swift
Substring
```

may retain the original String's storage.

For long-lived stored values, convert to:

```swift
String(substring)
```

where appropriate.

Do not convert every temporary Substring automatically.

---

## Data

Use `Data` for binary data.

Do not pass binary buffers around as String unless encoding is genuinely part of the protocol.

---

## URL

Use:

```swift
URL
```

for URLs and file URLs.

Do not represent URLs as Strings throughout internal APIs when URL semantics matter.

---

## File Paths

For file operations, prefer Foundation's URL-based APIs where project conventions do so.

Avoid manual path concatenation:

```swift
root + "/" + filename
```

Prefer:

```swift
root.appendingPathComponent(filename)
```

or modern URL path APIs supported by the project.

---

## Codable

Use Codable where it cleanly models serialization.

Good:

```swift
struct Weapon: Codable {
    let id: String
    let name: String
}
```

Do not conform every domain model to Codable merely because data eventually crosses a boundary.

Separate transport and domain types when their responsibilities materially differ.

---

## Coding Keys

Use `CodingKeys` only when wire names differ or custom serialization behavior is needed.

Do not declare them when synthesized names already match.

---

## Custom Decoding

Use custom `init(from:)` when actual normalization or complex compatibility requires it.

Do not hand-write decoding for simple models where synthesis works.

Generated boilerplate adds bugs.

---

## Decoding External Data

Do not assume successful Codable decoding guarantees all domain invariants.

If a numeric/string combination is syntactically valid but semantically invalid, validate it at the appropriate boundary.

---

## DTOs vs Domain Models

Do not automatically create:

```text
WeaponDTO
WeaponResponse
WeaponEntity
WeaponModel
WeaponViewModel
WeaponDomain
```

when they all contain identical fields.

Separate representations when:

- External naming differs.
- Optionality differs.
- Persistence concerns differ.
- Domain invariants differ.
- UI state differs.

Every mapping layer should represent a real transformation.

---

## JSONDecoder

Configure decoders centrally where shared policies exist.

Examples:

```swift
keyDecodingStrategy
dateDecodingStrategy
dataDecodingStrategy
```

Do not duplicate decoder configuration throughout the application.

Do not create a decoder framework for one request.

---

## Date

Use `Date` for points in time.

Do not store formatted user-facing strings as the canonical date representation.

Format at the presentation boundary.

---

## DateFormatter

DateFormatter creation can be relatively expensive.

Reuse formatters where formatting is frequent and appropriate.

Modern formatting APIs may provide cleaner alternatives depending on deployment targets.

Do not build a global formatting manager automatically.

---

## Calendar

Use `Calendar` for calendar-aware date operations.

Do not perform date arithmetic by manually adding seconds when calendar semantics matter.

---

## Localization

Use the project's localization strategy.

Do not hard-code user-visible strings throughout UI code in an app that supports localization.

Do not introduce localization infrastructure into a project that explicitly does not require it.

---

## String Catalogs

If the project uses modern String Catalogs, follow that workflow.

Do not create parallel `.strings` systems without reason.

---

## Networking

Use the project's existing networking stack.

Possible approaches include:

- `URLSession`.
- Alamofire.
- Custom typed client.
- Generated API client.

Do not add a second networking abstraction casually.

---

## URLSession

For straightforward networking, URLSession is often enough.

Example:

```swift
let (data, response) =
    try await session.data(for: request)
```

Do not wrap every URLSession method in several generic service layers without a reason.

---

## HTTP Status

A successful URLSession request does not guarantee an application-level successful HTTP response.

Validate status where appropriate.

Example:

```swift
guard let response =
    response as? HTTPURLResponse
else {
    throw NetworkError.invalidResponse
}

guard 200..<300 ~= response.statusCode else {
    throw NetworkError.httpStatus(
        response.statusCode
    )
}
```

---

## HTTP Clients

Reuse URLSession/client configuration appropriately.

Do not create a new session per request without a reason.

Session lifecycle affects:

- Connection pooling.
- Caching.
- Cookies.
- Authentication.
- Performance.

---

## Network Models

Normalize remote data into typed models near the network boundary.

Do not pass:

```text
[String: Any]
Any
NSDictionary
```

through modern Swift domain code unless interoperability requires it.

---

## `Any`

Avoid `Any` in ordinary Swift APIs.

Use it only when values are intentionally heterogeneous or crossing legacy/dynamic boundaries.

Strong typing is one of Swift's major benefits.

---

## Objective-C Collections

Do not use:

```text
NSArray
NSDictionary
NSString
NSNumber
```

in pure Swift code unless Objective-C interoperability requires them.

Prefer native Swift types.

---

## Objective-C Interoperability

Use:

```swift
@objc
dynamic
NSObject
```

only when runtime/Objective-C behavior requires it.

Do not mark methods `@objc` merely to make them look framework-compatible.

---

## `dynamic`

Avoid Swift's `dynamic` dispatch modifier unless Objective-C runtime dispatch or KVO-style behavior requires it.

Do not confuse it with dynamic typing in other languages.

---

## Selectors

Use `#selector` only where target-action or Objective-C runtime APIs require it.

Modern SwiftUI and closure-based APIs often provide alternatives.

Follow framework architecture.

---

## SwiftUI

When using SwiftUI, favor simple declarative views.

Keep:

- State ownership clear.
- Side effects outside `body`.
- Business logic out of views.
- Navigation deliberate.
- Dependencies explicit.

Do not treat SwiftUI as a reason to introduce a huge MVVM framework.

---

## SwiftUI `body`

A `body` may contain a moderately sized coherent hierarchy.

Do not extract every `HStack`, `Text`, or `Spacer` into a separate type.

Extract when a component:

- Has semantic meaning.
- Is reused.
- Owns behavior.
- Owns local state.
- Clarifies a genuinely large parent.

---

## Avoid Giant SwiftUI Views

If one view manages:

- Networking.
- Navigation.
- Filtering.
- Persistence.
- Search.
- Animation.
- Error handling.
- Presentation.

consider meaningful separation.

Do not split solely by line count.

---

## View Extraction

Good:

```text
WeaponCard
SearchBar
EpisodeList
PlayerControls
```

when these are real UI concepts.

Avoid:

```text
WeaponCardOuterContainer
WeaponCardInnerStack
WeaponCardTitleText
WeaponCardSpacer
```

for trivial fragments.

---

## State

Use the appropriate SwiftUI state wrapper for actual ownership.

Understand the distinction among:

```text
@State
@Binding
@StateObject
@ObservedObject
@Environment
@EnvironmentObject
@Observable
@Bindable
```

depending on project/toolchain.

Do not choose property wrappers by trial and error.

---

## `@State`

Use for value state owned by the view.

Example:

```swift
@State private var isExpanded = false
```

Do not use it to store external reference objects whose lifecycle needs explicit ownership unless the framework's current observation model supports that pattern.

---

## Bindings

Use `@Binding` when a child edits state owned elsewhere.

Do not pass bindings deep through the hierarchy when a more appropriate feature/state boundary exists.

Likewise, do not introduce a global store merely to avoid two levels of binding.

---

## Observable State

Use the observation system already adopted by the project.

Do not mix:

```text
ObservableObject
@Observable
Combine publishers
custom observation
```

without a clear reason.

---

## `ObservableObject`

In projects using it, keep observable objects focused.

Avoid a giant:

```text
AppViewModel
AppStateManager
GlobalStore
```

that owns every feature.

State should live near the feature that owns it.

---

## `@Published`

Do not mark every property published.

Only publish state the UI or observers actually depend on.

Derived state can often remain computed.

---

## Derived State

Avoid storing values that can be cheaply derived.

Bad:

```swift
@Published var firstName: String
@Published var lastName: String
@Published var fullName: String
```

if:

```swift
var fullName: String {
    "\(firstName) \(lastName)"
}
```

is sufficient.

Duplicated state introduces synchronization bugs.

---

## Environment

Use environment values for dependencies truly appropriate to the view hierarchy.

Do not turn:

```swift
@Environment
@EnvironmentObject
```

into a service locator for every dependency.

Hidden dependency chains make views difficult to reason about.

---

## Navigation

Use the project's navigation approach.

Possible architectures include:

- SwiftUI NavigationStack.
- UIKit coordinators.
- Router objects.
- TCA navigation.
- Custom feature navigation.

Do not introduce a coordinator/router layer just because navigation exists.

---

## NavigationStack

For SwiftUI, use typed navigation where appropriate.

Do not store arbitrary `AnyHashable` destinations merely to avoid modeling routes.

Enums can often express finite app routes cleanly.

---

## Route Enums

A route enum can be useful:

```swift
enum Route: Hashable {
    case weapon(WeaponID)
    case settings
}
```

Do not create enormous universal route enums if features already own independent navigation.

---

## UIKit

When using UIKit, follow UIKit lifecycle and ownership conventions.

Do not wrap every UIKit type merely to make it look SwiftUI-like.

UIViewController remains a legitimate controller abstraction.

---

## View Controllers

Keep controllers focused on presentation and interaction.

Do not turn every controller into a pure forwarding shell surrounded by five additional classes.

Likewise, avoid dumping all domain behavior directly into a controller.

Use proportional architecture.

---

## Storyboards and XIBs

If the project uses Interface Builder, follow its conventions.

Do not rewrite storyboard views programmatically during unrelated work.

Likewise, do not introduce storyboards into a fully programmatic UI project.

---

## IBOutlets

Implicitly unwrapped outlets may be appropriate under established UIKit lifecycle guarantees:

```swift
@IBOutlet private weak var titleLabel: UILabel!
```

Do not treat this as permission to use IUOs throughout non-UI code.

---

## Weak Outlets

Follow framework/project conventions regarding weak vs strong outlets.

Do not change ownership qualifiers casually.

---

## Auto Layout

Use established layout mechanisms:

- Constraints.
- UIStackView.
- Layout anchors.
- SwiftUI layout.

Do not manually calculate frames unless the UI genuinely requires custom layout or performance dictates it.

---

## SnapKit and Similar Libraries

If the project already uses a layout DSL, follow it.

Do not introduce a layout dependency solely to avoid a few native constraints.

---

## UITableView / UICollectionView

Use modern APIs where project targets support them and architecture already uses them.

Do not rewrite existing delegate/data-source code into diffable/compositional APIs merely because newer APIs exist.

---

## Diffable Data Sources

Use when they simplify dynamic collection updates.

Do not introduce them for a tiny static table unless the project already standardizes on them.

---

## Delegates

Delegate protocols are idiomatic in Cocoa where a one-to-one behavioral relationship exists.

Do not invent delegates for ordinary internal communication when:

- Closures.
- Direct calls.
- Async APIs.

would be simpler.

---

## Closures vs Delegates

Use closures for small, focused callbacks.

Use delegates when:

- Multiple callbacks belong together.
- A long-lived relationship exists.
- Framework conventions favor them.

Do not create a protocol for one callback.

---

## Notifications

Use NotificationCenter for truly broadcast process-level events where sender and receivers should remain decoupled.

Do not use it as a generic application event bus.

Typed direct state flow is usually easier to reason about.

---

## Notification Tokens

Manage observer lifetimes correctly for APIs requiring explicit removal.

Do not leak observer tokens.

Modern block/selector behavior varies by API and platform.

Follow established project practices.

---

## Dependency Injection

Constructor injection is usually sufficient.

Good:

```swift
final class WeaponLoader {
    private let client: APIClient

    init(client: APIClient) {
        self.client = client
    }
}
```

Do not introduce a DI container for ordinary code.

Explicit wiring is often clearer.

---

## Avoid Service Locators

Bad:

```swift
let repository =
    AppContainer.shared.weaponRepository
```

throughout arbitrary domain objects.

Service locators hide dependencies.

If the project already uses one, follow its established boundaries rather than expanding it casually.

---

## Singletons

Use singletons only when process-wide identity genuinely makes sense.

Examples may include certain:

- System wrappers.
- Caches.
- Framework adapters.

Do not make every service:

```swift
static let shared = ...
```

merely to avoid dependency injection.

Singletons create:

- Hidden dependencies.
- Shared mutable state.
- Test coupling.
- Lifecycle ambiguity.

---

## Managers

Before creating:

```text
NetworkManager
DatabaseManager
CacheManager
WeaponManager
```

ask what the object actually does.

Prefer specific names such as:

```text
APIClient
WeaponStore
PackageCache
AssetLoader
```

when those better communicate responsibility.

---

## Service Types

`Service` is acceptable when it reflects a real application/domain service.

Do not create one service class per noun or endpoint by template.

---

## Repository Pattern

Use repositories where persistence abstraction genuinely matters.

Do not automatically define:

```swift
protocol WeaponRepository {
    // CRUD
}
```

because the application has storage.

Direct database/API clients can be appropriate for simple features.

---

## Factories

Use a factory when construction truly chooses or hides implementations.

Do not create:

```swift
final class WeaponFactory {
    func makeWeapon(...) -> Weapon
}
```

when a normal initializer suffices.

---

## Coordinator Pattern

Coordinators can be valuable in UIKit applications with complex navigation.

Do not introduce one for a screen with one push transition.

Architecture should match navigation complexity.

---

## MVVM

MVVM can be useful.

Do not assume every SwiftUI view requires a ViewModel.

A simple view with:

```swift
@State
```

and injected domain data may need no additional object.

Use ViewModels where state and behavior meaningfully outgrow the view.

---

## Avoid ViewModel for Every View

Do not create:

```text
TitleViewModel
ButtonViewModel
CardViewModel
ToolbarViewModel
```

for static presentation components.

View models should own meaningful presentation state or behavior.

---

## Massive ViewModels

A ViewModel handling:

- Networking.
- Persistence.
- Navigation.
- Analytics.
- Formatting.
- Authentication.
- Every child screen.

has become a god object.

Split by actual responsibilities.

Do not split for arbitrary size alone.

---

## TCA / Redux-Style Architectures

If the project uses The Composable Architecture or another unidirectional system, follow its established conventions.

Do not introduce TCA to manage one toggle or simple form.

Do not bypass an existing TCA architecture with ad-hoc mutable state without understanding the project.

---

## Core Data

Follow the project's Core Data architecture.

Do not wrap every NSManagedObject operation in generic repositories automatically.

Pay attention to:

- Managed object contexts.
- Thread/concurrency isolation.
- Object IDs.
- Save boundaries.

---

## SwiftData

Use SwiftData where the deployment targets/project architecture support it.

Do not migrate Core Data code merely because SwiftData is newer.

---

## Managed Objects

Do not pass context-bound managed objects across concurrency boundaries carelessly.

Use appropriate identifiers/value models where required.

Follow the persistence framework's concurrency model.

---

## UserDefaults

Use UserDefaults for small preference-like values.

Do not use it as a general database.

Do not scatter string keys throughout the application if an existing settings abstraction centralizes them.

---

## Keychain

Use Keychain or established secure storage for secrets/credentials.

Do not place secrets in:

- UserDefaults.
- Plain files.
- Source code.
- Logs.

---

## FileManager

Use FileManager and URL APIs for filesystem work.

Do not create generic FileSystemManager wrappers unless abstraction or testing genuinely needs one.

---

## Security

Do not:

- Disable TLS validation.
- Hard-code secrets.
- Use unsafe cryptography.
- Trust server data blindly.
- Log authentication credentials.
- Store secrets insecurely.
- Force-cast untrusted dynamic values.
- Ignore keychain errors.

Use Apple security APIs or established audited libraries.

---

## CryptoKit

Use CryptoKit where it satisfies cryptographic requirements.

Do not implement custom cryptographic algorithms.

---

## Randomness

Use secure system randomness for security-sensitive values.

Do not build authentication tokens from:

```swift
Int.random(...)
```

without understanding the security requirements.

---

## Keychain Errors

Do not silently ignore Keychain failures.

Security-sensitive persistence should produce meaningful handling or propagation.

---

## Logging

Use the project's logging system.

Possible APIs include:

```text
Logger
OSLog
swift-log
custom structured logger
```

Do not leave:

```swift
print("here")
debugPrint(value)
```

in production paths unless output is intentional.

---

## Structured Logging

Prefer meaningful categories and privacy annotations when using OSLog.

Do not interpolate secrets into public logs.

Be deliberate about:

```text
privacy: .private
privacy: .public
```

where supported.

---

## Avoid Logging Every Method

Do not log:

```text
starting load
entered parser
finished parser
leaving method
```

unless operationally valuable.

Logs should help diagnose real behavior.

---

## Duplicate Logging

Do not log and rethrow the same error at every layer.

Usually the boundary with enough context should log.

Internal layers should enrich and propagate.

---

## Testing

Use the project's established test framework and style.

Modern projects may use:

- XCTest.
- Swift Testing.
- Snapshot testing.
- UI tests.

Do not migrate frameworks during unrelated work.

---

## Test Behavior

Test observable behavior.

Avoid tests that merely assert internal helper calls.

Do not introduce protocols solely so everything can be mocked.

---

## XCTest

Use descriptive test names according to project style.

Example:

```swift
func testLoadWeaponReturnsNilWhenMissing()
```

or the repository's preferred naming structure.

Avoid:

```swift
func testWeapon1()
```

---

## Swift Testing

If the project uses modern Swift Testing, follow its idioms.

Do not mix test frameworks randomly inside one feature unless the migration is intentional.

---

## Mocks

Mock external boundaries where interaction matters.

Prefer:

- Fakes.
- In-memory implementations.
- Stub clients.

when these are easier to understand.

Do not create a mocking protocol for every concrete object.

---

## Fakes

A small fake can often be clearer:

```swift
struct FakeWeaponStore: WeaponLoading {
    var weapon: Weapon?

    func loadWeapon(
        id: WeaponID
    ) async throws -> Weapon? {
        weapon
    }
}
```

Avoid giant reusable test infrastructure for a few tests.

---

## Test Builders

Do not create:

```text
WeaponTestDataBuilder
MockFactory
FixtureManager
TestingUtility
```

for three small tests.

Direct construction is often easier to read.

---

## Async Tests

Use async test support directly where available.

Avoid expectations/semaphores for APIs already using async/await.

Use legacy async testing primitives only where needed.

---

## MainActor Tests

Annotate tests with `@MainActor` only when the behavior genuinely requires main-actor isolation.

Do not use it merely to silence concurrency errors.

---

## Snapshot Tests

Use snapshots when visual/large structured output genuinely benefits.

Do not snapshot trivial labels or every view.

Snapshot maintenance has a cost.

---

## UI Tests

Use UI tests for important end-to-end user workflows.

Do not use XCUI tests to validate pure parsing or model logic.

Use the lowest useful testing layer.

---

## Test Determinism

Avoid tests dependent on:

- Current time.
- Random ordering.
- Real network.
- Shared global state.
- Arbitrary sleeps.
- Animation timing.

Inject or control these dependencies where necessary.

---

## Avoid Sleep in Tests

Do not use:

```swift
sleep(1)
```

to wait for asynchronous behavior.

Use:

- Async/await.
- Expectations.
- Clocks.
- Test schedulers.
- Framework synchronization.

according to the architecture.

---

## Time

Use `Date`, `Duration`, `Clock`, or project abstractions according to semantics and supported versions.

Do not use raw numeric seconds throughout APIs when typed duration constructs fit.

---

## Clocks

Modern Swift clock APIs can make timeout/testing logic clearer.

Do not introduce them into a project targeting unsupported Swift/platform versions.

---

## Formatting

Use the repository's formatting conventions.

Possible tools include:

```text
swift-format
SwiftFormat
Xcode formatting
```

Do not reformat unrelated files.

Avoid formatting churn.

---

## SwiftLint

Respect the project's SwiftLint configuration.

Do not disable rules merely to make generated code pass.

Avoid broad:

```swift
// swiftlint:disable all
```

or file-wide suppression without strong justification.

---

## Lint Suppression

If a suppression is necessary:

- Keep it narrow.
- Understand the rule.
- Explain unusual exceptions when useful.

Do not suppress architecture problems.

---

## Compiler Warnings

Treat warnings seriously.

Do not silence warnings using:

```text
!
as!
@unchecked Sendable
@MainActor
unused captures
```

without understanding the root cause.

---

## Access Control

Use the narrowest useful access level.

Commonly:

```text
private
fileprivate
internal
public
open
```

Do not mark everything public.

A smaller public API is easier to evolve.

---

## `private`

Use for implementation details within the enclosing declaration/scope.

Do not expose methods only so tests can call them.

Test through behavior where possible.

---

## `fileprivate`

Use only when file-level collaboration genuinely requires it.

Do not use `fileprivate` reflexively.

---

## `public`

Use public for APIs consumed outside the module.

Do not expose implementation details unnecessarily.

---

## `open`

Use `open` only when external subclassing or overriding is deliberately part of the public API.

Do not use it as a more permissive `public`.

---

## Extensions

Use extensions to organize real conceptual conformances or related functionality.

Good:

```swift
extension Weapon: Codable {
}
```

or:

```swift
extension Weapon {
    var displayName: String {
        // ...
    }
}
```

Do not split a class into ten extensions merely to create decorative MARK sections.

---

## Conformance Extensions

Putting protocol conformances in focused extensions can improve organization.

Follow repository style.

Do not move trivial members between extensions merely to satisfy arbitrary file structure.

---

## `MARK`

Use:

```swift
// MARK: - Networking
```

if the project uses MARK comments and the file genuinely benefits from navigation sections.

Do not add dozens of sections to a small file.

---

## Avoid Decorative Comments

Do not use:

```swift
// =================================
// NETWORKING
// =================================
```

unless the repository intentionally uses this style.

---

## Comments

Do not narrate obvious code.

Bad:

```swift
// Check if the weapon exists.
guard let weapon else {
    // Return if weapon doesn't exist.
    return
}
```

Better:

```swift
guard let weapon else {
    return
}
```

Write comments for:

- Non-obvious framework behavior.
- Workarounds.
- Protocol quirks.
- Thread-safety invariants.
- Performance tradeoffs.
- Domain-specific reasons.

Prefer **why**, not **what**.

---

## Avoid AI-Looking Comments

Avoid phrases such as:

```text
This method is responsible for...
This ensures that...
The following implementation...
In order to...
It is important to note...
This provides a robust and scalable solution...
```

Bad:

```swift
// This function is responsible for validating
// the weapon configuration before processing.
```

Better:

```swift
// Legacy manifests omit this field before format version 3.
```

---

## Documentation Comments

Use:

```swift
///
```

for public API documentation where useful.

Document:

- Non-obvious semantics.
- Errors.
- Concurrency guarantees.
- Ownership.
- Side effects.
- Units.
- Important invariants.

Do not restate obvious declarations.

---

## Avoid Generic Base Classes

Do not create:

```text
BaseService
BaseRepository
BaseViewModel
BaseCoordinator
BaseManager
```

merely to share trivial functionality.

Swift's protocol extensions and composition are available, but they also should not be overused.

Extract actual shared behavior.

---

## Avoid Protocol Explosion

Before adding a protocol, ask:

- Are there multiple implementations?
- Does a consumer actually need substitution?
- Is this a stable boundary?
- Would a closure be enough?
- Would the concrete type be simpler?

Do not use protocol-oriented programming as a dogma.

---

## Avoid Existential Everywhere

Do not turn every stored dependency into:

```swift
any SomethingProtocol
```

just to appear decoupled.

Concrete dependencies are often appropriate.

Abstract only where architecture benefits.

---

## Avoid Dependency Containers

Do not create:

```text
AppDependencies
ServiceContainer
DependencyRegistry
Resolver
EnvironmentContainer
```

for a small application merely because dependencies exist.

Explicit initializer injection is usually enough.

---

## Avoid Global Shared Services

Generated Swift frequently creates:

```swift
NetworkManager.shared
DatabaseManager.shared
CacheManager.shared
AnalyticsManager.shared
```

This is easy but creates hidden global dependencies.

Use explicit ownership unless process-wide singleton identity is truly required.

---

## Avoid Manager/Service/Provider Vocabulary

Before using:

```text
Manager
Service
Provider
Handler
Factory
Processor
Coordinator
Context
Registry
Strategy
Adapter
Repository
Facade
Orchestrator
```

ask whether the name accurately describes a real responsibility.

Do not generate architecture by suffix.

---

## Avoid Pass-Through Layers

Bad:

```swift
final class WeaponService {
    private let repository: WeaponRepository

    init(repository: WeaponRepository) {
        self.repository = repository
    }

    func weapon(
        id: WeaponID
    ) async throws -> Weapon? {
        try await repository.weapon(id: id)
    }
}
```

if the service adds no behavior.

Every layer should earn its existence.

---

## Avoid Repository + Service + UseCase Chains

Do not create:

```text
View
→ ViewModel
→ UseCase
→ Service
→ Repository
→ APIClient
```

for one simple request.

Each layer should perform meaningful:

- Policy.
- Transformation.
- Coordination.
- Isolation.

Otherwise simplify.

---

## Avoid Clean Architecture by Template

Clean Architecture can be useful in sufficiently complex applications.

Do not automatically create:

```text
Domain
Data
Presentation
UseCases
Repositories
Entities
DTOs
Mappers
```

for every feature.

The architecture must match the application's complexity.

---

## Avoid UseCase Per Action

Do not automatically create:

```text
GetWeaponUseCase
CreateWeaponUseCase
UpdateWeaponUseCase
DeleteWeaponUseCase
```

when direct domain/store calls already express the operations cleanly.

Use a use-case type for a real multi-step workflow.

---

## Avoid Mapper Explosion

Do not create:

```text
WeaponDTOMapper
WeaponEntityMapper
WeaponViewModelMapper
```

when the transformation is four obvious field assignments used once.

Direct mapping is often easier to follow.

---

## Avoid Builder Pattern

Swift's labeled initializers already provide readable construction.

Avoid:

```swift
WeaponBuilder()
    .setName(name)
    .setDamage(damage)
    .build()
```

when:

```swift
Weapon(
    name: name,
    damage: damage
)
```

is clearer.

---

## Avoid Fluent APIs by Default

Method chaining can be expressive.

Do not turn ordinary domain operations into long fluent chains merely to look elegant.

Use APIs that communicate intent.

---

## Avoid Class-Based Utility Namespaces

Bad:

```swift
enum StringUtils {
    static func normalize(...) {}
}
```

can sometimes be an intentional namespace, but a free function or focused extension may be better.

Do not mechanically recreate static utility classes from Java/C#.

---

## Namespace Enums

A caseless enum can be used as a namespace:

```swift
enum AssetPaths {
    static let root = ...
}
```

Use sparingly.

Do not group unrelated helpers under empty enums merely to avoid top-level declarations.

---

## Avoid `Any` and `[String: Any]`

In modern Swift, dynamic dictionaries should normally stay at interoperability boundaries.

Do not pass them through:

- Domain code.
- SwiftUI state.
- Persistence logic.

Convert to typed models promptly.

---

## Avoid Force Operations

Before finishing, search for:

```text
!
try!
as!
```

and verify every occurrence.

Each one should correspond to a clear invariant.

Generated Swift often uses them to bypass compiler guidance.

Do not.

---

## Avoid Excessive Optionality

If a model contains:

```swift
var id: String?
var name: String?
var stats: WeaponStats?
var category: Category?
```

but none may be missing after construction, the type is poorly modeled.

Require valid values during initialization.

---

## Avoid Excessive Guard Boilerplate

Do not repeatedly unwrap the same required state in every method.

Model that state at the type/lifecycle level where practical.

---

## Avoid Unnecessary Reference Types

If a class contains only immutable value fields and no shared identity, consider a struct.

Do not convert to struct mechanically if framework/reference semantics are intentional.

---

## Avoid Excessive `@MainActor`

If almost every class is `@MainActor`, isolation has likely become a compiler workaround instead of an architecture.

Keep domain/network/persistence work off the UI actor unless it genuinely belongs there.

---

## Avoid `Task` Everywhere

Do not wrap every async call:

```swift
Task {
    try await service.load()
}
```

when the caller is already asynchronous and can simply await.

Tasks create lifecycle and error-handling concerns.

---

## Avoid Detached Tasks

Do not use detached tasks for performance or convenience without understanding:

- Actor isolation.
- Priority.
- Cancellation.
- Task-local values.

Structured concurrency is preferable.

---

## Avoid Excessive Actors

An actor for every service is not automatically safer architecture.

Actors serialize access and change API semantics.

Use them where mutable concurrent state exists.

---

## Avoid `@unchecked Sendable`

Treat it as a serious manual safety promise.

Do not add it simply because Swift's concurrency checker complains.

---

## Avoid Concurrency Workarounds

Do not fix concurrency errors by randomly adding:

```text
@MainActor
nonisolated
Task.detached
@unchecked Sendable
```

Understand which actor owns the data and why it crosses isolation.

---

## Avoid Combine When Async/Await Fits Better

If one request produces one result:

```swift
async throws -> Weapon
```

is often simpler than a Publisher pipeline.

Follow existing architecture, though—do not rewrite mature Combine code casually.

---

## Avoid Async/Await Migration as Cleanup

Do not replace working callbacks/Combine code during unrelated work solely because async/await is newer.

Migrations affect API semantics and testing.

---

## Avoid NotificationCenter as State Management

Do not broadcast internal state changes globally because direct dependency flow seems inconvenient.

Use NotificationCenter for actual broadcast-style events.

---

## Avoid Singleton Navigation

Do not create:

```swift
NavigationManager.shared.navigate(...)
```

to bypass the actual UI ownership structure.

Navigation should usually be controlled by UI/navigation architecture.

---

## Avoid Global Mutable State

Global variables and static mutable storage make ownership difficult.

Prefer explicit state owners.

Use global immutable constants freely where appropriate.

---

## Avoid State Duplication

Do not store the same information in several representations unless synchronization is intentional.

For example:

```swift
var weapons: [Weapon]
var filteredWeapons: [Weapon]
var weaponCount: Int
```

may be redundant if the latter two are cheaply derived.

---

## Avoid Premature Caching

Do not add caches without real need.

Caching requires decisions about:

- Lifetime.
- Capacity.
- Invalidation.
- Concurrency.
- Staleness.

Measure or identify the actual requirement first.

---

## Avoid Premature Performance Work

Do not automatically:

- Convert classes to structs for speed.
- Replace protocols with generics.
- Add unsafe buffers.
- Add actors.
- Add caches.
- Add custom collections.
- Use `withUnsafeBytes`.
- Rewrite Foundation APIs.

Profile before optimizing.

---

## Performance

Use Instruments and project tooling for real performance work.

Possible concerns include:

- Allocations.
- Main-thread work.
- Rendering.
- Networking.
- Database queries.
- Image decoding.
- SwiftUI body recomputation.

Do not optimize based on folklore.

---

## Copy-on-Write Types

Swift collections already provide efficient value semantics.

Do not manually introduce reference wrappers solely to avoid hypothetical copies.

Measure actual copy behavior when performance matters.

---

## Unsafe Swift

Avoid:

```text
UnsafePointer
UnsafeMutablePointer
withUnsafeBytes
assumingMemoryBound
unsafeBitCast
```

unless low-level work genuinely requires them.

Keep unsafe operations narrowly scoped.

Document memory/lifetime assumptions when non-obvious.

---

## `unsafeBitCast`

Use only when representation compatibility is guaranteed and no safer conversion exists.

It is not a general-purpose casting mechanism.

---

## Memory Layout

Do not rely on Swift struct memory layout for serialized formats or external ABI unless explicitly guaranteed.

Swift layout can change.

Use explicit serialization formats.

---

## C Interoperability

At C/Objective-C boundaries:

- Keep unsafe pointers localized.
- Convert promptly into Swift types.
- Respect ownership rules.
- Respect nullability.
- Use unmanaged APIs only when necessary.

Do not let unsafe interoperability patterns spread through the application.

---

## `Unmanaged`

Use only where Core Foundation/Objective-C bridging requires explicit ownership control.

Understand:

```text
takeRetainedValue
takeUnretainedValue
passRetained
passUnretained
```

before using them.

Incorrect choices create leaks or crashes.

---

## Memory Cycles

Review ownership relationships involving:

- Delegates.
- Closures.
- Timers.
- Combine subscriptions.
- Tasks.
- Parent/child controllers.

Use weak/unowned references where lifecycle semantics require them.

Do not add weak references blindly.

---

## Timers

Be careful when a Timer retains its closure/target and the owner retains the timer.

Invalidate timers according to lifecycle.

Modern concurrency clocks/tasks may be simpler for some use cases, but do not rewrite existing timer architecture without reason.

---

## CADisplayLink

Treat display links as owned resources.

Invalidate appropriately.

Do not use continuous frame callbacks where normal animation APIs suffice.

---

## DispatchQueue

Use Grand Central Dispatch where existing architecture or low-level queue semantics require it.

For modern async application code, structured concurrency may be clearer.

Do not mix GCD and async/await unnecessarily.

---

## `DispatchQueue.main.async`

Do not scatter:

```swift
DispatchQueue.main.async {
}
```

through async/await code merely to update UI.

Use actor isolation correctly.

---

## Semaphores

Avoid blocking semaphores to turn asynchronous APIs synchronous.

Bad:

```swift
let semaphore = DispatchSemaphore(value: 0)
// async work
semaphore.wait()
```

This can deadlock and blocks threads.

Use async APIs end-to-end.

---

## Locks vs Queues

Do not create private serial queues solely to simulate a lock unless the queue semantics provide real value.

Use the simplest synchronization primitive appropriate to the system.

---

## Package Organization

Organize code around actual modules/features.

Avoid generic dumping grounds:

```text
Helpers
Utils
Common
Shared
Managers
Services
```

when more specific ownership exists.

---

## Feature Organization

Feature-oriented grouping can be useful:

```text
Weapons/
  WeaponListView.swift
  WeaponDetailView.swift
  WeaponStore.swift
```

Do not impose feature folders if the project uses layer-oriented organization.

Follow repository structure.

---

## Avoid Deep Folder Trees

Do not create:

```text
Features/
  Weapons/
    Domain/
      Entities/
      UseCases/
      Repositories/
    Data/
      DTOs/
      Mappers/
      Sources/
    Presentation/
      ViewModels/
      Views/
```

for a tiny feature unless the existing architecture requires it.

Navigation cost is also complexity.

---

## File Size

Do not split types merely to hit arbitrary file-length targets.

A cohesive file can contain several related small types/extensions.

Split where it improves ownership and navigation.

---

## One Type Per File

Swift projects often use one major type per file, but this is a convention rather than an absolute rule.

Small supporting enums or private types can stay nearby.

Do not create eight files for eight three-line supporting types.

---

## Generated Code

Do not manually edit generated:

- SwiftGen.
- Sourcery.
- API clients.
- Core Data generated files.
- Macro output.

Modify the source/generator.

Follow the repository's regeneration workflow.

---

## Dependencies

Before adding a package, ask:

- Does Apple/Foundation already solve this?
- Does the project already have a dependency for it?
- Is the package actively maintained?
- Does it support required platforms?
- Is its API stable?
- Is the dependency worth the maintenance cost?

Do not add a package for five straightforward lines of code.

Do not reimplement complex security/network/protocol libraries solely to avoid dependencies.

---

## Swift Package Manager

Use the project's package-management system.

Do not migrate CocoaPods/Carthage dependencies to SPM during unrelated work.

For SPM projects, avoid changing unrelated package versions.

Review `Package.resolved` changes.

---

## Dependency Products

Import only the actual modules needed.

Do not expose third-party types through public APIs unless that coupling is intentional.

Do not wrap every dependency merely to hide it either.

---

## Formatting

Follow the project's preferred layout.

Do not manually align assignments or parameters if the formatter will change them.

Avoid whitespace-only churn.

---

## Closure Formatting

Use trailing closures and indentation consistently with repository style.

Avoid deeply nested closure pyramids.

If closures become difficult to follow, extract meaningful operations.

---

## Line Length

Follow formatting/lint rules.

Do not distort APIs solely to satisfy an arbitrary width if the formatter/project convention handles wrapping differently.

---

## Before Finishing

Review the change and remove or correct:

- Unnecessary protocols.
- Protocols with one implementation and no consumer need.
- Generic `Manager`/`Service`/`Repository` layers.
- Pass-through use cases.
- Dependency containers.
- Unnecessary singletons.
- Unnecessary classes where values fit better.
- Excessive inheritance.
- Excessive optionals.
- Force unwraps without proven invariants.
- `try!`.
- `as!`.
- Excessive `Any`.
- Over-generic APIs.
- Unnecessary type erasure.
- Unnecessary actors.
- Misused `@MainActor`.
- `@unchecked Sendable` used as a compiler escape hatch.
- Detached or unowned tasks.
- Combine pipelines where existing architecture does not require them.
- Retain cycles.
- Giant observable/view-model objects.
- SwiftUI widget fragmentation.
- Business logic in views.
- Duplicate derived state.
- Debug `print` calls.
- Empty catch blocks.
- Redundant error wrapping/logging.
- Placeholder TODOs.
- Dead code.
- Unused imports.
- Speculative extensibility.
- Unrelated refactors.

Then run the repository's established checks where available.

Typical Swift checks may include:

```text
swift build
swift test
```

for Swift packages.

Xcode projects may use:

```text
xcodebuild
```

or project-specific scripts.

Linting and formatting may include:

```text
SwiftLint
swift-format
SwiftFormat
```

Do not assume these exact tools or commands exist.

Inspect:

```text
Package.swift
project settings
Makefile
Fastlane
CI configuration
.swiftlint.yml
.swiftformat
repository documentation
```

and follow the project's established workflow.

The final code should look like it naturally belongs in the repository rather than like a generic AI-generated Swift solution.

It should feel like Swift written by an experienced maintainer: strongly typed without needless ceremony, value-oriented where appropriate, protocol-oriented only where useful, explicit about optionals and ownership, disciplined about actor isolation, and willing to use straightforward structs, enums, functions, and framework APIs instead of layering abstractions for their own sake.