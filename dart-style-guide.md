# Dart Style Guide

Write Dart as an experienced professional Dart developer maintaining a real production codebase.

The goal is clean, idiomatic, maintainable Dart—not code that looks generated, over-engineered, excessively abstract, or mechanically translated from Java, C#, TypeScript, or Kotlin.

The most important rule:

> Do not optimize for demonstrating Dart features, design patterns, or architecture. Optimize for producing the smallest idiomatic production-quality change that an experienced Dart maintainer would reasonably write.

## General Principles

Prefer:

- Existing project conventions over personal preferences.
- Simple concrete implementations.
- Strong static typing.
- Null safety used correctly rather than bypassed.
- Small focused functions.
- Small cohesive classes.
- Immutable data where it naturally fits.
- Composition over inheritance.
- Direct data flow.
- Clear async behavior.
- Standard Dart APIs before custom abstractions.
- Framework-native patterns when using Flutter.
- Small public APIs.
- Explicit state ownership.
- Direct implementation over speculative extensibility.

Avoid:

- Enterprise-style layering without need.
- Interfaces for a single implementation.
- Factory classes that construct one concrete type.
- Manager/service/provider/repository classes with no real behavior.
- `dynamic` used as a type escape hatch.
- Excessive `late`.
- Excessive non-null assertions.
- Null checks repeated everywhere.
- Deep inheritance hierarchies.
- Huge widget build methods.
- Tiny extracted widgets with no semantic value.
- State-management frameworks introduced for trivial local state.
- Async methods that do not perform async work.
- Premature generic abstractions.
- Unnecessary extension methods.
- Reflection-like dynamic patterns where normal typed code works.

Dart should remain readable and unsurprising.

---

## Match the Existing Codebase

Before writing Dart, inspect nearby files and follow the repository's established conventions for:

- Dart SDK version.
- Flutter version.
- Package organization.
- Naming.
- File naming.
- State management.
- Error handling.
- Dependency injection.
- Serialization.
- Testing.
- Routing.
- Networking.
- Persistence.
- Logging.
- Linting.
- Formatting.
- Widget composition.
- Model architecture.

Check relevant files such as:

```text
pubspec.yaml
pubspec.lock
analysis_options.yaml
dart_test.yaml
build.yaml
lib/
test/
integration_test/
```

For Flutter projects, also inspect:

```text
android/
ios/
web/
macos/
windows/
linux/
```

when platform behavior matters.

Consistency with the repository is more important than imposing a preferred architecture.

Do not migrate state-management systems, rename entire module hierarchies, introduce new layers, or reorganize folders during an unrelated change.

---

## Dart SDK Version

Determine the supported Dart version before using newer features.

Check:

```yaml
environment:
  sdk: ^3.5.0
```

or the project's current constraint.

Do not assume the newest SDK.

Language features such as:

- Records.
- Patterns.
- Sealed classes.
- Class modifiers.
- Extension types.
- Enhanced enums.

must match the declared SDK.

Do not modernize unrelated files solely to use a newer feature.

---

## Flutter Version

If the project uses Flutter, verify the supported Flutter version before relying on newer widgets, Material APIs, or framework behaviors.

Do not assume:

- Material 3 is enabled.
- A recently added widget exists.
- A new platform API is available.
- Deprecated APIs can be removed safely.

Match the project's Flutter version and design system.

---

## Naming

Use normal Dart naming conventions:

- `lowerCamelCase` for variables, functions, parameters, fields, and methods.
- `UpperCamelCase` for classes, enums, typedefs, extensions, and mixins.
- `lowercase_with_underscores.dart` for file names.
- Private identifiers begin with `_`.
- Constants generally use `lowerCamelCase`, not screaming snake case.

Good:

```dart
weaponDefinition
assetPath
packageName
loadResult

loadPackage()
resolveAsset()
parseWeaponData()
```

Avoid generic AI-style names:

```dart
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

```dart
final data = getData();
final result = processData(data);
```

Better:

```dart
final weapon = loadWeaponDefinition();
final stats = parseWeaponStats(weapon);
```

Do not make names excessively verbose.

Avoid:

```dart
final successfullyParsedWeaponConfigurationResult =
    parseWeaponConfiguration();
```

when:

```dart
final weaponConfig = parseWeaponConfiguration();
```

is clear.

---

## File Names

Use:

```text
weapon_parser.dart
asset_loader.dart
package_cache.dart
```

Do not use:

```text
WeaponParser.dart
weaponParser.dart
weapon-parser.dart
```

unless generated tooling or legacy code requires it.

Match existing file organization.

---

## Private Identifiers

Use `_` for library-private members:

```dart
final _cache = <String, Weapon>{};
```

Do not prefix everything with `_` merely to make implementation look encapsulated.

Only hide members that should remain private.

---

## Functions First

Use top-level or local functions for stateless behavior when appropriate.

Good:

```dart
WeaponStats parseWeaponStats(WeaponData data) {
  return WeaponStats(
    damage: data.damage,
    fireRate: data.fireRate,
  );
}
```

Do not automatically create:

```dart
class WeaponStatsParser {
  WeaponStats parse(WeaponData data) {
    // ...
  }
}
```

unless the parser actually owns:

- State.
- Dependencies.
- Configuration.
- Multiple related operations.
- Lifecycle.

Dart files and libraries already provide organization.

---

## Classes

Use classes when state and behavior belong together.

Good:

```dart
class PackageCache {
  final Map<String, Package> _packages = {};

  Package? get(String path) => _packages[path];

  void put(String path, Package package) {
    _packages[path] = package;
  }
}
```

Do not create classes merely because a concept is a noun.

Avoid architecture such as:

```text
WeaponManager
WeaponService
WeaponProvider
WeaponHandler
WeaponProcessor
WeaponCoordinator
```

unless these represent genuinely separate responsibilities.

---

## Data Classes

Use plain classes for structured data.

Example:

```dart
class WeaponStats {
  const WeaponStats({
    required this.damage,
    required this.fireRate,
    required this.magazineSize,
  });

  final double damage;
  final double fireRate;
  final int magazineSize;
}
```

Do not create getters that simply expose final fields.

Bad:

```dart
double getDamage() => _damage;
```

when:

```dart
final double damage;
```

already communicates the API clearly.

---

## Immutable Data

Prefer immutable data where the model naturally represents a value.

Good:

```dart
class Weapon {
  const Weapon({
    required this.id,
    required this.name,
  });

  final String id;
  final String name;
}
```

Do not force immutability onto stateful objects where mutation is the natural design.

Use the model that best reflects semantics.

---

## `const`

Use `const` constructors and values when they are genuinely constant.

Good:

```dart
const EdgeInsets.all(16)
```

or:

```dart
const WeaponCategory.rifle()
```

where applicable.

Do not add `const` mechanically where it makes code noisier without value.

In Flutter, `const` can reduce rebuild work and communicate immutability, but it should still follow readability and project conventions.

---

## Constructors

Use constructors to establish a valid object.

Good:

```dart
class Weapon {
  const Weapon({
    required this.id,
    required this.name,
  });

  final String id;
  final String name;
}
```

Do not perform substantial hidden work in constructors such as:

- Network requests.
- File reads.
- Starting streams.
- Registering global listeners.
- Launching async work.

Use factories or explicit initialization methods when construction requires complex work.

---

## Named Constructors

Use named constructors when they communicate meaningful construction variants.

Example:

```dart
Weapon.fromJson(Map<String, Object?> json)
```

or:

```dart
Weapon.empty()
```

if an empty weapon is a legitimate domain state.

Do not create multiple named constructors merely to demonstrate the feature.

---

## Factory Constructors

Use factory constructors when:

- Returning cached instances.
- Choosing among implementations.
- Redirecting construction.
- Parsing into an existing type.
- Construction logic cannot be expressed by a normal generative constructor.

Do not use `factory` automatically.

A normal constructor is usually simpler.

---

## Private Constructors

Use private constructors when they support a real pattern such as:

- Static-only utility namespace.
- Controlled construction.
- Singleton-like immutable instance.
- Generated serialization pattern.

Do not create private constructors merely to make a class look architecturally controlled.

---

## Static Utility Classes

Avoid Java-style utility classes.

Bad:

```dart
class WeaponUtils {
  static String normalizeName(String name) {
    return name.trim().toLowerCase();
  }
}
```

Prefer:

```dart
String normalizeWeaponName(String name) {
  return name.trim().toLowerCase();
}
```

or an extension when it naturally belongs to a widely reusable type and project conventions support it.

---

## Extensions

Use extension methods when behavior naturally augments an existing type.

Example:

```dart
extension AssetPathParsing on String {
  bool get isGameAssetPath => startsWith('/Game/');
}
```

Do not create extension methods for one local use.

Avoid giant extension libraries that add dozens of unrelated methods to:

```text
String
Iterable
BuildContext
Widget
Object
```

Extensions affect discoverability and can create naming collisions.

Use them sparingly.

---

## Avoid `BuildContext` Extension Explosion

Flutter code often accumulates extensions like:

```dart
context.theme
context.colors
context.textTheme
context.navigator
context.mediaQuery
context.padding
```

These can be useful in an established design system.

Do not create them automatically.

Native APIs such as:

```dart
Theme.of(context)
Navigator.of(context)
MediaQuery.of(context)
```

are often perfectly clear.

Follow the codebase.

---

## Interfaces

Dart supports implicit interfaces.

Every class defines an interface automatically.

Do not create:

```dart
abstract class WeaponParser {
  Weapon parse(String input);
}

class DefaultWeaponParser implements WeaponParser {
  // ...
}
```

when there is only one implementation and no consumer needs substitution.

Start concrete.

Introduce abstraction when real usage requires it.

---

## Abstract Classes

Use abstract classes when they define a meaningful shared contract or partial implementation.

Do not create abstract base classes solely to mimic Java/C# interfaces.

A normal concrete class is often enough.

---

## Consumer-Owned Abstractions

When a consumer genuinely needs a small abstraction, keep it focused.

Example:

```dart
abstract interface class WeaponStore {
  Future<Weapon?> find(String id);
}
```

may be appropriate if the consumer only needs lookup behavior.

Avoid enormous interfaces with unrelated CRUD, caching, searching, and lifecycle methods.

---

## Class Modifiers

Use Dart class modifiers intentionally when supported:

```text
abstract
base
interface
final
sealed
mixin
```

Do not add them merely to show advanced language knowledge.

They should express actual library or inheritance constraints.

---

## `final class`

Use `final class` when inheritance outside the class's library should be prohibited for a real design reason.

Do not mark every class final automatically unless that is the project's convention.

---

## `sealed`

Use sealed classes for closed hierarchies where exhaustive handling provides value.

Example:

```dart
sealed class LoadState {}

final class Loading extends LoadState {}

final class Loaded extends LoadState {
  Loaded(this.weapon);

  final Weapon weapon;
}

final class Failed extends LoadState {
  Failed(this.error);

  final Object error;
}
```

Do not create sealed hierarchies for trivial two-state booleans.

---

## Enums

Use enums for closed sets of values.

Good:

```dart
enum WeaponCategory {
  rifle,
  smg,
  shotgun,
  sniper,
}
```

Do not use raw strings when the state is truly closed and internal.

For external APIs, map wire values deliberately.

---

## Enhanced Enums

Use enhanced enums when variants naturally own related data or behavior.

Do not turn every enum into a mini-class.

Simple enums should remain simple.

---

## Boolean State

Use `bool` for actual yes/no state.

Use enums or sealed classes when several mutually exclusive states exist.

Bad:

```dart
class LoadState {
  bool isLoading;
  bool isLoaded;
  bool hasFailed;
}
```

Better:

```dart
enum LoadStatus {
  loading,
  loaded,
  failed,
}
```

or a sealed state hierarchy if associated data matters.

---

## Records

Use records when a small temporary grouping of values is clear.

Example:

```dart
(String name, int damage) parseSummary(String input) {
  // ...
}
```

Named fields can help:

```dart
({String name, int damage}) parseSummary(String input) {
  // ...
}
```

Do not return large anonymous records where a domain class would improve readability and API stability.

---

## Patterns

Use patterns when they simplify destructuring or state handling.

Do not convert straightforward property access into pattern-heavy code merely because Dart supports it.

Use advanced syntax where it reduces complexity.

---

## Switch Expressions

Use switch expressions for concise value selection.

Good:

```dart
final label = switch (status) {
  Status.ready => 'Ready',
  Status.loading => 'Loading',
  Status.failed => 'Failed',
};
```

Do not force substantial side-effect-heavy control flow into expressions.

Use a normal `switch` when clearer.

---

## Exhaustive Switching

Take advantage of exhaustive switching with enums and sealed types.

Avoid default branches when explicit variants help the compiler catch future changes.

Use `_` when future variants should intentionally share behavior.

---

## Null Safety

Use Dart's null-safety type system correctly.

Prefer:

```dart
Weapon?
```

only when absence is legitimate.

Use:

```dart
Weapon
```

when the value must exist.

Do not make fields nullable merely because initialization is inconvenient.

Design types around actual semantics.

---

## Avoid Excessive `!`

The non-null assertion:

```dart
value!
```

should mean:

> The program has already established that this value cannot be null here.

Do not use `!` to silence analyzer errors.

Bad:

```dart
return config!.server!.host!;
```

If these values are required, validate them once and make the internal types non-nullable.

---

## Avoid `late` as an Escape Hatch

Use `late` when initialization genuinely happens after construction but before first use.

Good examples may include:

- Framework-managed lifecycle initialization.
- Expensive lazy values.
- Dependency initialization with a guaranteed lifecycle.

Do not use:

```dart
late String name;
late Client client;
late Repository repository;
```

merely to avoid constructor parameters or nullable types.

Prefer explicit initialization.

---

## `late final`

Use:

```dart
late final
```

when delayed one-time initialization matches the object's lifecycle.

This can be safer than mutable `late`.

Do not use it if the value can simply be initialized directly.

---

## Nullable Fields

Do not make required fields nullable and then assert everywhere.

Bad:

```dart
Weapon? _weapon;

void render() {
  print(_weapon!.name);
}
```

if the object's valid operating state always requires a weapon.

Consider modeling state explicitly.

---

## Safe Navigation

Use:

```dart
weapon?.name
```

when absence is expected.

Do not use long chains such as:

```dart
package?.data?.weapon?.stats?.damage
```

to hide broken invariants.

Validate boundary data and normalize it into stronger internal models.

---

## `??`

Use null-coalescing when null specifically means "use default."

Good:

```dart
final retries = config.retries ?? 3;
```

Do not use defaults that hide missing required configuration.

---

## `??=`

Use for simple lazy/default assignment:

```dart
cache ??= <String, Weapon>{};
```

Do not use it where explicit initialization would be clearer.

---

## `dynamic`

Avoid `dynamic` in typed internal code.

Prefer:

```dart
Object?
```

for values of unknown type that must be checked.

`dynamic` disables static checking for operations on the value.

Use it only where an API genuinely requires dynamic behavior.

---

## `Object?`

At untrusted boundaries, prefer:

```dart
Object?
```

over `dynamic`.

Then narrow explicitly:

```dart
if (value is! String) {
  throw FormatException('expected string');
}
```

This preserves analyzer help.

---

## Type Checks

Use:

```dart
is
is!
```

when runtime narrowing is necessary.

Do not scatter type checks throughout domain code if data can be parsed into typed models once at the boundary.

---

## Casts

Avoid:

```dart
value as String
```

merely to silence typing issues.

A cast does not validate external data safely unless failure is an acceptable exception.

For untrusted values, check types deliberately.

---

## Generic Types

Use generic types where real reuse exists.

Good:

```dart
class Cache<K, V> {
  // ...
}
```

Do not genericize domain-specific code.

Avoid:

```dart
TOutput processEntity<TInput, TOutput>(TInput input)
```

for one concrete operation.

---

## Generic Constraints

Keep generic constraints understandable.

Do not construct deep generic hierarchies where a concrete type or small interface would be easier to maintain.

---

## Collections

Use:

```text
List
Set
Map
```

according to semantics.

Do not create custom collection wrappers unless they provide meaningful domain behavior.

---

## Lists

Use `List<T>` for ordered collections.

Prefer typed lists:

```dart
final weapons = <Weapon>[];
```

Do not use:

```dart
List<dynamic>
```

unless the data is genuinely dynamic.

---

## `final` Collections

Remember:

```dart
final items = <Weapon>[];
```

prevents reassignment of `items`, not mutation of the list.

The following is still valid:

```dart
items.add(weapon);
```

Do not mistake `final` for deep immutability.

---

## Unmodifiable Collections

Use unmodifiable collections when callers must not mutate exposed state.

Example:

```dart
List<Weapon> get weapons =>
    List.unmodifiable(_weapons);
```

Do not allocate defensive copies on every getter unless the API actually needs isolation.

Consider immutable storage or exposing an `Iterable` depending on semantics.

---

## Iterable

Accept or return `Iterable<T>` when callers only need iteration and a concrete list is unnecessary.

Do not use `Iterable` purely to look abstract.

If random access or materialization is part of the contract, `List<T>` is clearer.

---

## Collection Literals

Use Dart collection literals:

```dart
final weapons = <Weapon>[
  rifle,
  shotgun,
];
```

Prefer them over verbose constructors where appropriate.

---

## Collection `if`

Use collection conditionals when they keep UI/data construction clear.

Example:

```dart
final actions = [
  saveAction,
  if (canDelete) deleteAction,
];
```

Do not create complex nested conditional logic inside collection literals.

Extract logic when readability suffers.

---

## Collection `for`

Use collection-for when it clearly maps values into a literal.

Example:

```dart
final widgets = [
  for (final weapon in weapons)
    WeaponTile(weapon: weapon),
];
```

Do not force complicated branching/transformation pipelines into collection syntax.

---

## Spread Operators

Use:

```dart
...items
...?optionalItems
```

when merging collections clearly.

Do not create long chains of spreads whose precedence or provenance is hard to understand.

---

## `map`

Use `map` for transformation.

Good:

```dart
final names = weapons
    .map((weapon) => weapon.name)
    .toList();
```

Do not use `map` for side effects.

Bad:

```dart
weapons.map((weapon) {
  saveWeapon(weapon);
});
```

Use:

```dart
for (final weapon in weapons) {
  saveWeapon(weapon);
}
```

---

## `where`

Use `where` for filtering.

Good:

```dart
final active = weapons.where(
  (weapon) => weapon.enabled,
);
```

Do not construct custom filter utilities for ordinary collection behavior.

---

## Iterator Chains

Avoid giant chains when intermediate concepts matter.

Bad when excessive:

```dart
final result = weapons
    .where(...)
    .map(...)
    .expand(...)
    .where(...)
    .fold(...);
```

Use intermediate variables or loops when clearer.

Readable Dart matters more than functional density.

---

## `fold`

Use `fold` for actual accumulation when it reads naturally.

Do not force every reduction into `fold`.

A loop may be easier to understand.

---

## `firstWhere`

Use carefully.

This:

```dart
weapons.firstWhere((weapon) => weapon.id == id)
```

throws if nothing matches.

If absence is expected, use the project's established approach, a helper, or package utilities already present.

Do not hide exceptions with a catch when a nullable lookup abstraction would be clearer.

---

## Maps

Use typed maps:

```dart
final weaponsById = <String, Weapon>{};
```

Do not use:

```dart
Map<String, dynamic>
```

throughout domain code when a model class can represent the shape.

---

## Map Lookup

Remember:

```dart
map[key]
```

returns a nullable value.

If the map guarantees presence due to an established invariant, handle that invariant deliberately rather than scattering `!`.

---

## Sets

Use sets when uniqueness or membership is the point.

Do not use lists for repeated membership checks on large collections when a set better expresses intent.

Do not convert tiny lists to sets without need.

---

## Equality

Dart's default object equality is identity unless overridden.

For value types, implement equality when semantic value comparison matters.

Do not implement equality on every class automatically.

---

## `hashCode`

If overriding `==`, implement compatible `hashCode`.

Do not use mutable fields in hash identity carelessly when objects may be stored in sets/maps.

---

## Equality Libraries

If the project already uses:

```text
equatable
freezed
built_value
```

follow that architecture.

Do not add a package just to avoid writing equality for one tiny type.

---

## Freezed

Freezed can be useful for:

- Immutable models.
- Union/sealed states.
- Copying.
- Generated equality.
- Serialization integration.

Do not introduce Freezed automatically.

For a small model:

```dart
class Weapon {
  const Weapon({
    required this.id,
    required this.name,
  });

  final String id;
  final String name;
}
```

may be entirely sufficient.

---

## Generated Data Models

Use code generation when the project already relies on it or the scale justifies it.

Do not create generated-model infrastructure for a few simple classes.

Generated code adds:

- Build steps.
- Dependency coupling.
- Regeneration requirements.

---

## `copyWith`

Use `copyWith` for immutable models where partial updates occur regularly.

Do not manually add `copyWith` to every class.

Do not add an entire generation package merely for one `copyWith`.

---

## Exceptions

Use exceptions for failures that cannot be represented as normal expected outcomes.

Do not throw for ordinary control flow.

For lookup absence:

```dart
Weapon?
```

may be better than throwing if missing values are expected.

---

## Exception Types

Throw specific exception types where callers benefit.

Example:

```dart
throw FormatException(
  'missing weapon ID',
);
```

Do not throw generic strings.

Bad:

```dart
throw 'weapon failed';
```

Always throw objects implementing appropriate exception semantics.

---

## Custom Exceptions

Create custom exceptions when a meaningful domain boundary benefits from them.

Example:

```dart
class PackageLoadException implements Exception {
  PackageLoadException(this.path, this.cause);

  final String path;
  final Object cause;

  @override
  String toString() =>
      'Unable to load package "$path": $cause';
}
```

Do not create a large exception hierarchy for every failure.

---

## Catch Specific Failures

Prefer specific handling:

```dart
try {
  // ...
} on FormatException catch (error) {
  // ...
}
```

over broad:

```dart
catch (_) {
  // ...
}
```

when a known failure is expected.

---

## Catching `Object`

Dart can throw non-`Exception` objects, though normal code should not.

When a true boundary must catch all failures:

```dart
catch (error, stackTrace)
```

preserve useful debugging context.

Do not broadly catch and ignore errors.

---

## Stack Traces

When logging or translating unexpected errors, preserve stack traces.

Avoid:

```dart
catch (error) {
  throw MyException(error.toString());
}
```

if doing so destroys useful original context.

Use:

```dart
Error.throwWithStackTrace
```

or project-established error wrapping when appropriate.

---

## Rethrow

Use:

```dart
rethrow;
```

when the current catch block adds context/logging but the original exception should continue.

Do not write:

```dart
throw error;
```

without understanding its stack-trace behavior.

---

## Avoid Swallowing Errors

Bad:

```dart
try {
  await loadWeapon();
} catch (_) {
  return null;
}
```

unless converting failure into absence is an intentional API contract.

Do not hide programming bugs or network failures.

---

## Async Functions

Use `async` only for actual asynchronous work.

Bad:

```dart
Future<int> weaponCount(
  List<Weapon> weapons,
) async {
  return weapons.length;
}
```

Prefer:

```dart
int weaponCount(List<Weapon> weapons) {
  return weapons.length;
}
```

Do not make functions async merely because their callers are async.

---

## Futures

Use `Future<T>` for one asynchronous result.

Do not wrap already available values in unnecessary futures.

Avoid:

```dart
return Future.value(value);
```

inside an `async` function where:

```dart
return value;
```

is enough.

---

## Avoid Unnecessary `await`

This:

```dart
return loadWeapon();
```

can be enough when the function simply forwards a future.

Use:

```dart
return await loadWeapon();
```

when local try/catch/finally behavior or stack semantics justify it.

Do not add `await` mechanically.

---

## Parallel Futures

Use:

```dart
Future.wait(...)
```

when operations are independent and concurrency is appropriate.

Do not parallelize when:

- Order matters.
- One result feeds another.
- A service has rate limits.
- Resource usage would spike.
- Failure handling requires sequential behavior.

---

## Streams

Use `Stream<T>` for sequences of asynchronous events over time.

Do not use streams for a single result.

Use `Future<T>` instead.

---

## Stream Ownership

When listening to a stream, decide:

- Who owns the subscription?
- When is it cancelled?
- Can multiple listeners exist?
- Is the stream broadcast?

Do not leak stream subscriptions.

---

## StreamSubscription

Store and cancel long-lived subscriptions appropriately.

Example:

```dart
late final StreamSubscription<Event>
    _subscription;
```

may be valid if lifecycle is guaranteed.

Do not use `late` here unless initialization and disposal are truly reliable.

---

## Broadcast Streams

Use broadcast streams only when multiple listeners legitimately need the same event stream.

Do not convert every stream to broadcast to suppress listener errors.

Understand the ownership model.

---

## Stream Controllers

Avoid custom `StreamController` infrastructure when:

- A `ValueNotifier`.
- Existing state-management primitive.
- Direct callback.
- Existing stream.

would suffice.

Stream controllers require lifecycle management.

---

## Completers

Use `Completer` only when adapting callback-based behavior into a Future or manually controlling completion is genuinely necessary.

Do not use `Completer` inside an ordinary async function that can simply `await`.

Bad:

```dart
Future<Weapon> loadWeapon() {
  final completer = Completer<Weapon>();

  doAsyncWork().then(
    completer.complete,
    onError: completer.completeError,
  );

  return completer.future;
}
```

Prefer:

```dart
Future<Weapon> loadWeapon() async {
  return doAsyncWork();
}
```

---

## Isolates

Use isolates for CPU-bound work that genuinely needs parallel execution.

Do not use isolates for normal asynchronous I/O.

Network/file APIs already operate asynchronously where appropriate.

---

## `compute` in Flutter

Use `compute` or isolate helpers for meaningful CPU-heavy work.

Do not move trivial JSON parsing or small list transformations into isolates without evidence.

Serialization and isolate startup have overhead.

---

## Cancellation

Dart Futures do not provide universal built-in cancellation.

Use project/framework-specific cancellation mechanisms where needed.

Do not invent a complex cancellation framework for trivial work.

---

## Timers

Use `Timer` for actual delayed or periodic work.

Always consider cancellation and widget/object lifecycle.

Do not use arbitrary delays to fix race conditions.

Bad:

```dart
await Future.delayed(
  const Duration(milliseconds: 500),
);
```

to "wait for the UI" or "wait for state."

Fix the lifecycle or synchronization problem.

---

## Duration

Use `Duration` rather than raw integer time units.

Good:

```dart
const timeout = Duration(seconds: 5);
```

Avoid:

```dart
const timeout = 5000;
```

where the unit is implicit.

---

## Dependency Injection

Dart usually does not require a DI container for ordinary code.

Constructor injection is often enough:

```dart
class WeaponLoader {
  WeaponLoader(this.repository);

  final WeaponRepository repository;
}
```

Do not introduce:

```text
ServiceLocator
DependencyContainer
Resolver
ProviderFactory
```

merely to avoid passing dependencies explicitly.

---

## Service Locators

Global service locator patterns can hide dependencies.

Example:

```dart
final repository = getIt<WeaponRepository>();
```

may be valid in a codebase already built around GetIt.

Do not introduce GetIt or another service locator automatically.

Match existing architecture.

---

## Factories

Use factory functions or constructors when construction genuinely chooses among implementations or performs meaningful setup.

Do not create:

```dart
class WeaponFactory {
  Weapon create(...) => Weapon(...);
}
```

for simple object construction.

---

## Repository Pattern

Repositories can be useful for meaningful persistence/domain boundaries.

Do not create one for every entity automatically.

If a feature simply calls one API client method, another wrapper layer may add nothing.

---

## Service Classes

A service can be appropriate when it coordinates several dependencies or represents a meaningful application operation.

Do not create:

```text
CreateWeaponService
UpdateWeaponService
DeleteWeaponService
FindWeaponService
```

by template when direct methods or repository calls already express the behavior.

---

## Managers

`Manager` often hides an unclear responsibility.

Prefer specific names such as:

```text
Cache
Loader
Scheduler
Store
Resolver
Controller
Coordinator
```

only when those are the actual roles.

Do not use `Manager` as a default suffix.

---

## Providers

In Flutter, `Provider` may refer to a specific package or architecture.

Do not call arbitrary classes `Provider` unless:

- They provide actual data/resources.
- The framework convention uses the term.
- The role is clear.

Avoid naming collisions with state-management packages.

---

## State Management

Use the simplest state mechanism that fits the feature.

Possible choices include:

- Local `State`.
- `ValueNotifier`.
- `ChangeNotifier`.
- Provider.
- Riverpod.
- Bloc/Cubit.
- Signals.
- Redux.
- MobX.

Follow the project's existing architecture.

Do not introduce a new state-management library for one screen.

---

## Local State

For simple widget-local state, `StatefulWidget` is often enough.

Bad architecture:

```text
CounterBloc
CounterRepository
CounterUseCase
CounterState
CounterEvent
CounterProvider
```

for a single toggle or counter.

Use architecture proportional to complexity.

---

## `setState`

`setState` is not inherently bad.

Use it for small local UI state.

Do not avoid it solely because a state-management package exists.

Conversely, do not manage large shared business state through deeply nested `setState`.

Use the project's existing boundaries.

---

## ChangeNotifier

Use `ChangeNotifier` when its mutable observable model fits the application.

Do not build giant ChangeNotifier objects containing unrelated screen state, networking, persistence, and navigation.

Keep responsibilities cohesive.

---

## Riverpod

If the project uses Riverpod:

- Follow its provider conventions.
- Keep providers focused.
- Avoid provider proliferation.
- Do not move every pure function into a provider.
- Keep business logic testable outside widgets.

Do not introduce Riverpod into another state architecture without explicit reason.

---

## Bloc

If the project uses Bloc/Cubit:

- Keep events/states meaningful.
- Avoid one event per setter where simpler state works.
- Keep business logic outside presentation where appropriate.
- Do not create massive event/state hierarchies for tiny features.

Do not introduce Bloc merely because the application is Flutter.

---

## Provider

If the project uses Provider:

- Keep models focused.
- Avoid deeply nested provider trees.
- Avoid exposing giant mutable objects globally.
- Use `Consumer`, `Selector`, or context methods according to existing conventions.

Do not optimize rebuilds prematurely with selectors everywhere.

---

## Global State

Avoid global mutable state.

Do not place application state in top-level variables merely for convenience.

Use explicit ownership through:

- Widget tree.
- Application object.
- State-management architecture.
- Dependency injection.

depending on the project.

---

## Flutter Widgets

Prefer small meaningful widgets.

Extract a widget when it:

- Represents a real UI concept.
- Is reused.
- Owns behavior/state.
- Makes the parent substantially clearer.
- Has independent testing value.

Do not extract every `Row`, `Padding`, or `Container` into its own class.

---

## Avoid Widget Fragmentation

Bad conceptual structure:

```text
WeaponCard
WeaponCardContainer
WeaponCardInner
WeaponCardHeader
WeaponCardTitle
WeaponCardTitleText
```

for a simple card.

A widget hierarchy should reflect real UI concepts, not every DOM-like node.

---

## Build Methods

Keep build methods understandable.

A build method can be moderately long if it represents one coherent UI tree.

Do not extract widgets solely to satisfy a line-count preference.

Extract where structure or reuse benefits.

---

## Widget Methods

Avoid large methods like:

```dart
Widget _buildHeader()
Widget _buildBody()
Widget _buildFooter()
```

solely to split a build method.

Private widget-returning methods can be fine, but actual widget classes may be better when:

- Keys matter.
- Independent rebuilds matter.
- Reuse exists.
- Behavior is owned.

Use whichever makes the structure clearer.

---

## `Container`

Do not use `Container` for everything.

Use more specific widgets when they communicate intent:

```text
Padding
SizedBox
DecoratedBox
ColoredBox
Align
Center
ConstrainedBox
```

Do not replace every `Container` mechanically either.

`Container` is useful when several layout/decorative concerns genuinely combine.

---

## Avoid Nested Layout Noise

Generated Flutter often creates trees like:

```text
Container
  Padding
    Container
      Center
        Row
          Container
```

Review whether each widget is necessary.

Flutter composition is normal, but unnecessary nesting reduces readability and can complicate layout.

---

## `SizedBox`

Use `SizedBox` for intentional spacing or fixed constraints.

Do not sprinkle arbitrary:

```dart
const SizedBox(height: 13)
```

throughout the app if a design spacing system exists.

Use project tokens or theme spacing conventions.

---

## Theme

Use the project's theme and design system.

Prefer:

```dart
Theme.of(context).colorScheme
Theme.of(context).textTheme
```

or established custom theme extensions.

Avoid hard-coded colors and font sizes if theme tokens already exist.

---

## Theme Extensions

Use `ThemeExtension` when the project genuinely needs custom theme tokens beyond built-in Material theme fields.

Do not create theme extensions for three one-off values.

---

## BuildContext

Do not store `BuildContext` in long-lived objects.

Contexts belong to the widget tree and can become invalid.

Pass context only to code that actually requires framework lookup/navigation/UI operations.

Business logic should usually not depend on it.

---

## Async `BuildContext`

Be careful using context after `await`.

For Flutter code:

```dart
await save();

if (!context.mounted) {
  return;
}

Navigator.of(context).pop();
```

or equivalent `mounted` checks may be required.

Do not add mounted checks mechanically when no asynchronous gap exists.

---

## Navigation

Use the project's routing approach.

Possible systems include:

```text
Navigator
go_router
auto_route
Beamer
Router API
```

Do not add another router for one feature.

Keep route knowledge out of domain code.

---

## UI Side Effects

Navigation, snack bars, dialogs, and focus changes are presentation concerns.

Do not hide them inside repositories or domain models.

Keep boundaries clear.

---

## Dialogs

Use framework dialog APIs according to project conventions.

Do not create global static dialog services merely to avoid passing context unless the existing architecture intentionally does so.

---

## Keys

Use keys when widget identity matters.

Valid cases include:

- Reorderable lists.
- Stateful list children.
- Tests.
- Preserving identity across moves.

Do not add `ValueKey` to every widget.

Keys are not free architectural decoration.

---

## GlobalKey

Use `GlobalKey` sparingly.

It is powerful and can bypass normal tree ownership.

Valid uses include:

- Form state.
- Scaffold access in specific architectures.
- Measuring/controlling a unique widget where no simpler mechanism exists.

Do not use GlobalKeys as a general communication mechanism.

---

## Forms

Use Flutter's form APIs where appropriate.

Do not create giant custom validation frameworks around simple inputs.

Keep field validation close to the actual input/domain constraints.

---

## Text Controllers

Dispose controllers owned by a State object:

```dart
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}
```

Do not create a new controller inside `build`.

Ownership and lifecycle matter.

---

## Focus Nodes

Dispose owned `FocusNode` instances.

Do not create focus infrastructure unless interaction requires it.

---

## Animation Controllers

Dispose controllers.

Prefer implicit animations when they solve the interaction simply.

Do not create `AnimationController` for effects already handled by:

```text
AnimatedContainer
AnimatedOpacity
AnimatedSwitcher
TweenAnimationBuilder
```

---

## Animations

Use motion intentionally.

Do not animate everything.

Respect platform accessibility settings where relevant.

Avoid expensive perpetual animations without a real purpose.

---

## Layout

Use Flutter layout primitives correctly.

Prefer:

```text
Row
Column
Flex
Expanded
Flexible
Wrap
Stack
Align
Padding
ConstrainedBox
LayoutBuilder
```

based on actual layout semantics.

Do not fix layout bugs with arbitrary nested `SizedBox` and `Transform.translate` values before understanding constraints.

---

## Constraints

Understand Flutter's rule:

> Constraints go down, sizes go up, parents set positions.

Many layout bugs come from misunderstanding constraints.

Do not solve overflow by randomly adding:

```text
Expanded
Flexible
SingleChildScrollView
```

until the real constraint problem is understood.

---

## Expanded

Use `Expanded` when a flex child should consume remaining space.

Do not wrap everything in Expanded.

Inside non-flex parents it is invalid.

Understand the parent layout.

---

## Flexible

Use when a flex child may use available space without being forced to fill it.

Do not choose between `Expanded` and `Flexible` randomly.

---

## SingleChildScrollView

Use for reasonably small content that needs scrolling.

Do not wrap huge dynamic lists in:

```dart
SingleChildScrollView(
  child: Column(
    children: thousandsOfItems,
  ),
)
```

Use lazy list widgets for large collections.

---

## ListView

Use `ListView.builder` for large or dynamic lists.

Do not use a builder for three fixed children if a literal list is clearer.

---

## GridView

Use lazy builder variants when the dataset is large.

Do not create complicated grid delegates where a simple responsive grid already fits.

---

## ShrinkWrap

Be cautious with:

```dart
shrinkWrap: true
```

on large scrollable lists.

It can force expensive layout.

Do not use it as a generic fix for nested-scroll errors.

Understand scroll ownership.

---

## Nested Scrolling

Avoid unnecessary nested scroll views.

Use proper slivers or coordinated scrolling when the UI genuinely needs complex scrolling.

Do not stack several scrollable widgets hoping Flutter resolves it automatically.

---

## Slivers

Use slivers when the screen genuinely benefits from coordinated lazy scrolling.

Do not convert ordinary pages to CustomScrollView/slivers simply to appear advanced.

---

## MediaQuery

Use `MediaQuery` for viewport/platform accessibility information.

Do not make every widget manually calculate breakpoints if the project has a responsive layout abstraction.

Likewise, do not add a responsive framework for one screen.

---

## LayoutBuilder

Use when layout should depend on available parent constraints rather than global screen size.

Do not nest LayoutBuilders everywhere.

Use them where component-level responsiveness actually matters.

---

## Responsive Design

Prefer flexible layouts before adding many breakpoints.

Do not build separate widget trees for every screen size unless the interaction truly differs.

Reuse semantics and state.

---

## Platform Checks

Use platform-specific behavior only when requirements differ.

Do not scatter:

```dart
Platform.isWindows
Platform.isAndroid
Platform.isIOS
```

through unrelated UI code.

Isolate platform-specific implementations where practical.

---

## `kIsWeb`

Use where behavior genuinely differs on web.

Do not use it as a generic workaround for every failing API.

Understand whether a package already abstracts the platform difference.

---

## Material vs Cupertino

Follow the application's design system.

Do not mix Material and Cupertino components randomly.

Use platform-adaptive widgets only when the product actually intends platform-specific UI behavior.

---

## Material 3

Use Material 3 conventions only if the project enables and follows them.

Do not migrate components to Material 3 during unrelated feature work.

---

## Theme Colors

Prefer semantic theme colors over hard-coded:

```dart
Colors.blue
Colors.red
Colors.grey
```

when the app has a custom color system.

Hard-coded framework colors are fine for prototypes or intentional design choices.

---

## Text Styles

Use existing text theme styles.

Avoid:

```dart
const TextStyle(
  fontSize: 17,
  fontWeight: FontWeight.w600,
)
```

everywhere if the design system already defines typography.

---

## Spacing

Use established spacing constants or tokens when the project has them.

Do not invent:

```text
13
17
19
23
```

pixel spacing values throughout unrelated widgets.

Do not build a spacing framework if the project intentionally uses direct constants.

---

## Assets

Use the project's asset management conventions.

Declare assets correctly in `pubspec.yaml`.

Do not hard-code duplicated path strings throughout the app if stable shared constants are already used.

Do not create an asset registry class solely to hold three strings unless convention requires it.

---

## Images

Use appropriate Flutter image widgets and caching behavior.

Consider:

- Fit.
- Dimensions.
- Loading state.
- Error state.
- Memory use.

Do not load huge full-resolution images into tiny thumbnails without considering resizing/caching when performance matters.

---

## Network Images

Use the project's image-loading package if one exists.

Do not add another caching package for one image.

Handle failures where they meaningfully affect the UI.

---

## Accessibility

Use Flutter semantics appropriately.

Prefer standard widgets because they already provide much of the expected accessibility behavior.

Do not replace buttons with `GestureDetector` on plain containers when a semantic button widget fits.

---

## Buttons

Prefer:

```text
ElevatedButton
FilledButton
TextButton
OutlinedButton
IconButton
```

where they fit.

Do not create clickable `Container` widgets with manual semantics unless a custom interaction truly requires it.

---

## GestureDetector

Use it for gestures not naturally represented by a standard interactive widget.

Do not use GestureDetector as the default for every tap target.

Standard controls provide better:

- Semantics.
- Focus.
- Keyboard behavior.
- Hover states.
- Disabled handling.

---

## InkWell

Use InkWell/InkResponse when Material ripple interaction is appropriate.

Do not wrap everything in InkWell merely for taps.

Use the actual semantic control where possible.

---

## Semantics

Add explicit `Semantics` when native widget semantics are insufficient.

Do not decorate every widget with redundant semantics.

Incorrect semantics can make accessibility worse.

---

## Tooltip

Use tooltips where icons or controls need additional explanation.

Do not use tooltips as a replacement for clear visible labels in critical workflows.

---

## Focus and Keyboard

Desktop/web Flutter apps should account for keyboard navigation where required.

Do not design interaction around touch only when the platform supports keyboard/mouse use.

Follow project accessibility targets.

---

## MouseRegion

Use only when hover/pointer behavior is actually needed.

Do not wrap standard buttons merely to detect hover they already expose through styling APIs.

---

## Input Formatters

Use Flutter input formatters for appropriate client-side input constraints.

Do not treat them as the only validation layer for external or security-sensitive data.

---

## Business Validation

Validation that defines the domain should generally live outside widget code.

Do not duplicate the same business rule in:

- Text field validator.
- Repository.
- API client.
- Model.

without a reason.

Validate at appropriate boundaries.

---

## Networking

Use the project's existing HTTP client.

Possible approaches include:

```text
dart:io HttpClient
http
dio
chopper
retrofit
```

Do not add a second HTTP stack casually.

---

## HTTP Client Ownership

Reuse client instances where practical.

Do not create and discard a new client for every request unless the package's usage model requires it.

Close clients you own when appropriate.

---

## Timeouts

Set meaningful request timeouts where indefinite waits would be harmful.

Do not add arbitrary tiny timeout values.

Follow the network stack's existing configuration.

---

## Request Models

Use typed request/response models when API structure is stable.

Do not spread:

```dart
Map<String, dynamic>
```

through the entire application.

Dynamic maps are appropriate near JSON boundaries.

Normalize into typed models.

---

## JSON

Dart's JSON decoding returns dynamic object structures.

Example:

```dart
final decoded = jsonDecode(body);
```

Do not immediately assume shape.

At the boundary, validate/narrow before constructing domain objects.

---

## `Map<String, dynamic>`

It is common for JSON adapters.

Keep it close to serialization code.

Avoid passing it through business logic where typed objects are clearer.

---

## Serialization

Follow the project's approach:

- Manual `fromJson`.
- `json_serializable`.
- Freezed.
- Built value.
- Another existing generator.

Do not introduce a second serialization system.

---

## Manual `fromJson`

For small models, explicit parsing can be clearer.

Example:

```dart
factory Weapon.fromJson(
  Map<String, Object?> json,
) {
  final id = json['id'];
  final name = json['name'];

  if (id is! String || name is! String) {
    throw const FormatException(
      'invalid weapon payload',
    );
  }

  return Weapon(
    id: id,
    name: name,
  );
}
```

Do not create generic reflection-like mappers for a handful of models.

---

## Generated Serialization

Use when model volume or consistency justifies it.

Do not manually edit generated files.

Modify source annotations and regenerate.

---

## Database and Persistence

Use the project's established persistence library.

Possible choices include:

```text
sqflite
drift
isar
hive
shared_preferences
realm
```

Do not add another persistence system for one feature.

---

## SharedPreferences

Use for small preference-like key/value state.

Do not use it as a general database.

Large structured datasets belong elsewhere.

---

## Local Database

Keep database queries close to the data layer if the project has one.

Do not create repository/service/domain layers automatically around simple persistence.

Add boundaries where real complexity exists.

---

## Transactions

Use database transactions for related writes that must succeed or fail together.

Do not keep transactions open during unrelated network calls unless semantics require it.

---

## Logging

Use the project's existing logger.

Do not leave:

```dart
print('here');
print(data);
debugPrint('test');
```

in finished production code unless output is intentional.

Use structured/contextual logging where supported.

---

## `print`

Fine for:

- Tiny scripts.
- Quick local debugging.

Not a production logging strategy for serious applications.

Remove debug output before finishing.

---

## Flutter `debugPrint`

Useful during development.

Do not leave excessive debugPrint calls scattered through release application logic.

Use the established logger.

---

## Error Logging

Do not log the same failure at every layer.

Internal layers should usually add useful context and propagate.

The app boundary can decide what to report.

---

## Routing Errors

Presentation layers should translate application failures into:

- Error screens.
- Snack bars.
- Retry states.

Do not make repository code display dialogs or navigate.

---

## Testing

Use the project's established testing stack.

Typical Dart tools include:

```text
package:test
flutter_test
integration_test
mocktail
mockito
```

Do not add a mocking package merely because generated tests are easier with one.

---

## Unit Tests

Test behavior.

Good:

```dart
test(
  'returns null when the weapon does not exist',
  () {
    // ...
  },
);
```

Avoid vague:

```dart
test('works', () {
  // ...
});
```

---

## Flutter Widget Tests

Use widget tests for:

- Rendering behavior.
- Interaction.
- State changes.
- Accessibility semantics.
- Navigation behavior where appropriate.

Do not test pure parsing logic through widgets.

---

## `pump`

Understand what:

```dart
tester.pump()
tester.pumpAndSettle()
```

do.

Do not use `pumpAndSettle` everywhere blindly.

Some animations or repeating timers never settle.

Pump only as much as the behavior requires.

---

## Golden Tests

Use golden tests when visual regression coverage adds real value.

Do not snapshot every widget.

Golden tests can become expensive and brittle if overused.

---

## Integration Tests

Use integration tests for real application workflows.

Do not use them to validate small pure functions.

---

## Fakes vs Mocks

Prefer simple fakes when they make behavior clearer.

Do not define interfaces solely so a mocking framework can generate mocks.

Architecture should serve production code first.

---

## Mocktail / Mockito

Use the project's existing choice.

Do not mix both.

Avoid interaction-heavy tests where asserting final behavior is clearer.

---

## Verify Behavior

Bad:

```dart
verify(
  repository.findWeapon(any()),
).called(1);
```

when the meaningful assertion is the returned state or UI.

Use interaction verification when the interaction itself is part of the contract.

---

## Test Setup

Keep test fixtures small.

Do not build elaborate:

```text
TestDataFactory
MockManager
FixtureBuilder
TestingContext
```

for a few tests.

Extract setup only when repetition becomes real.

---

## Fake Async

Use fake async/time-control utilities when timers and delays are central to behavior.

Do not insert real delays into unit tests.

Avoid:

```dart
await Future.delayed(
  const Duration(seconds: 1),
);
```

to "wait" for code.

---

## Linting

Use the project's analyzer and lint configuration.

Common starting points include:

```yaml
include: package:flutter_lints/flutter.yaml
```

or custom packages such as:

```text
very_good_analysis
lint
custom_lint
```

Do not replace the lint configuration during unrelated work.

---

## Analyzer Warnings

Do not silence analyzer errors using:

```dart
dynamic
as
!
ignore comments
```

unless the code is actually safe and the reason is understood.

Fix the type model where practical.

---

## Ignore Comments

Avoid broad:

```dart
// ignore_for_file: ...
```

unless generated/integration code genuinely requires it.

Prefer narrow suppression with a real justification.

Do not suppress warnings to make generated code pass.

---

## Formatting

Always use Dart's formatter.

Typical command:

```text
dart format .
```

or scoped changed files.

Do not manually align code against formatter behavior.

---

## Static Analysis

Typical commands:

```text
dart analyze
```

or:

```text
flutter analyze
```

depending on the project.

Do not assume one if the repository defines its own scripts.

---

## Tests

Typical commands may include:

```text
dart test
flutter test
flutter test integration_test
```

Use repository-specific scripts where present.

Do not blindly run platform tests that require unavailable tooling.

---

## Code Generation

If the project uses build_runner:

```text
dart run build_runner build
```

may be appropriate.

Do not regenerate the entire project without understanding whether generated output will create unrelated diffs.

Use project scripts when available.

---

## Dependencies

Before adding a package, ask:

- Does Dart or Flutter already provide this?
- Does the project already have an equivalent?
- Is the package maintained?
- Does it support the project's Dart/Flutter version?
- Does it support required platforms?
- Is the dependency worth the complexity?

Do not add a package for five straightforward lines.

Do not reimplement complex standards/security-sensitive behavior merely to avoid a dependency.

---

## Pubspec

Do not manually edit lockfile contents.

Use normal package tooling.

Be deliberate about dependency category:

```text
dependencies
dev_dependencies
dependency_overrides
```

Do not use overrides casually.

---

## Dependency Overrides

`dependency_overrides` should usually be temporary and intentional.

Do not add one merely to silence version resolution problems without understanding compatibility.

---

## Packages vs Internal Code

Do not extract internal code into a separate Dart package solely for architectural purity.

A package boundary adds:

- Versioning.
- Imports.
- Dependency management.
- Public API concerns.

Create packages when independent reuse or isolation genuinely matters.

---

## Folder Structure

Do not impose a universal Flutter structure.

Avoid automatically creating:

```text
core/
common/
shared/
data/
domain/
presentation/
application/
infrastructure/
```

for every project.

Use architecture proportional to the app.

A small feature may simply be:

```text
features/weapons/
  weapon_page.dart
  weapon_model.dart
  weapon_repository.dart
```

or even fewer files.

---

## Clean Architecture

Do not apply Clean Architecture mechanically.

Layers such as:

```text
data
domain
presentation
use_cases
repositories
entities
```

can be useful in a large app with real boundaries.

They are harmful when they produce one-to-one pass-through classes.

Every layer should earn its existence.

---

## Use Cases

Do not create:

```text
GetWeaponUseCase
CreateWeaponUseCase
UpdateWeaponUseCase
DeleteWeaponUseCase
```

automatically.

A use-case object is useful when it encapsulates a meaningful workflow.

Direct repository or service calls are fine when the operation is simple.

---

## Entities vs Models

Do not create separate:

```text
WeaponEntity
WeaponModel
WeaponDto
WeaponResponse
WeaponViewModel
```

when their fields are identical and no boundary requires separation.

Split representations when they genuinely differ.

---

## View Models

Use view models/controllers where UI logic actually benefits from them.

Do not create one per widget by template.

Simple widgets can consume state directly from the existing architecture.

---

## Controllers

Name controller types according to actual framework/domain conventions.

Do not create generic `Controller` classes merely to hold methods.

---

## ViewModel Explosion

Avoid:

```text
HomeViewModel
HomeState
HomeController
HomeService
HomeRepository
```

for a screen that simply displays one API response.

Architecture should match complexity.

---

## Avoid Java/C# Architecture

Do not translate:

```text
IWeaponService
WeaponServiceImpl
WeaponRepository
WeaponRepositoryImpl
WeaponFactory
WeaponManager
```

into Dart simply because those patterns exist elsewhere.

Dart is expressive enough to remain much simpler.

---

## Avoid Interfaces for Testing

Do not create an abstract class solely because a mock generator prefers it.

Use fakes, existing abstractions, or direct dependency injection.

Add an interface when production consumers need one.

---

## Avoid `dynamic` for JSON Everywhere

Keep dynamic parsing at the network/file boundary.

Convert to typed data early.

Do not let maps of dynamic values leak into widgets or business logic.

---

## Avoid Non-Null Assertion Chains

Bad:

```dart
user!.profile!.settings!.theme!
```

This usually signals poor state modeling.

Prefer:

- Required types.
- Boundary validation.
- Explicit nullable state handling.

---

## Avoid `late` Everywhere

If nearly every field is:

```dart
late
```

the lifecycle or constructor design likely needs review.

Use required constructor parameters or nullable state where appropriate.

---

## Avoid State Frameworks for Local UI

Do not use Riverpod/Bloc/Redux merely to manage:

```text
selected tab
checkbox state
expanded panel
text visibility
```

when simple local state is enough and consistent with the project.

---

## Avoid Business Logic in Widgets

Do not put large:

- Parsing.
- Networking.
- Persistence.
- Domain calculations.

inside `build()`.

Widget code should focus on presentation and interaction.

Small transformation for rendering is fine.

---

## Avoid Repository Logic in Build

Never trigger network/database operations every time `build()` runs unless the framework abstraction intentionally memoizes them.

Bad:

```dart
Widget build(BuildContext context) {
  final future = repository.loadWeapons();
  return FutureBuilder(
    future: future,
    ...
  );
}
```

if every rebuild starts a new request.

Own async state appropriately.

---

## FutureBuilder

Use `FutureBuilder` when UI genuinely follows one future's lifecycle.

Do not create a new future in every build unintentionally.

Store or obtain stable futures through the project's state architecture when necessary.

---

## StreamBuilder

Use when UI follows a stream.

Do not create a stream inside build unless the stream's lifecycle is intentionally tied to that build.

---

## ValueListenableBuilder

Useful for lightweight observable state.

Do not replace the project's main state architecture with ad-hoc notifiers without reason.

---

## AnimatedBuilder

Use for animation/listenable-driven rebuilds when appropriate.

Do not wrap huge subtrees if only a small area depends on the listenable.

Keep rebuild boundaries sensible.

---

## Rebuild Optimization

Do not prematurely optimize every rebuild.

Flutter is designed around rebuilding widgets.

Optimize only when:

- Profiling shows a problem.
- Expensive work occurs in build.
- Very large subtrees rebuild unnecessarily.

Do not litter the app with memoization-like abstractions.

---

## Avoid Expensive Work in Build

Do not perform:

- File I/O.
- JSON parsing of large payloads.
- Network calls.
- Heavy sorting.
- Large image processing.

inside `build()`.

Prepare data outside or cache appropriately.

Small list transforms are fine.

---

## Const Widgets

Use const where natural.

Do not contort widget APIs solely to make more constructor calls const.

Readability and architecture come first.

---

## RepaintBoundary

Use for measured repaint problems.

Do not wrap arbitrary widgets in RepaintBoundary "for performance."

Profiling should drive this.

---

## Keys for Performance

Keys do not inherently make widgets faster.

Use them for identity.

Do not add keys as generic optimization.

---

## Caching

Do not add caching without a real need.

When caching exists, define:

- Ownership.
- Lifetime.
- Invalidation.
- Error behavior.
- Capacity.

Do not cache every API response merely because it might be reused.

---

## Image Cache

Flutter already provides image caching behavior.

Do not build a second cache layer without understanding existing framework/package behavior.

---

## Performance

Prefer correct, clear code first.

Do not prematurely:

- Introduce isolates.
- Memoize everything.
- Add repaint boundaries.
- Add custom render objects.
- Replace widgets with painters.
- Cache all computed values.
- Micro-optimize allocations.

Profile actual problems.

---

## DevTools

Use Flutter/Dart DevTools for:

- CPU profiling.
- Memory.
- Rebuild analysis.
- Network inspection.
- Rendering performance.

Do not optimize based solely on intuition.

---

## CustomPainter

Use when drawing custom visual content is actually needed.

Do not implement ordinary UI controls manually in Canvas.

Widgets provide semantics and layout for free.

---

## RenderObject

Custom RenderObjects are advanced infrastructure.

Do not introduce one unless widget/composition/layout APIs cannot reasonably solve the requirement.

They increase complexity substantially.

---

## Platform Channels

Use platform channels when native APIs genuinely require them.

Do not create platform channels for functionality already provided by Flutter/Dart or a maintained plugin.

Keep platform-specific code narrow.

---

## Plugins

Before adding a Flutter plugin, check:

- Platform support.
- Maintenance.
- API quality.
- License.
- Compatibility with current Flutter.
- Whether existing dependencies already solve it.

Do not add several overlapping plugins.

---

## FFI

Use Dart FFI for native libraries when needed.

Be deliberate about:

- Memory ownership.
- Pointer lifetimes.
- Struct layouts.
- Native cleanup.
- Threading.

Do not use FFI to optimize ordinary Dart code without evidence.

---

## Web

When targeting Flutter Web or Dart Web, remember browser APIs and platform limitations differ from mobile/desktop.

Do not assume:

```text
dart:io
filesystem paths
native sockets
process execution
```

are available.

Isolate platform-specific implementations cleanly.

---

## Conditional Imports

Use conditional imports/exports when platform-specific implementations genuinely share one API.

Do not create elaborate platform abstraction layers for one tiny difference.

---

## Desktop

For Windows/macOS/Linux apps, account for:

- Window resizing.
- Keyboard.
- Mouse.
- Hover.
- File system behavior.
- Native menu/window conventions.

Do not design desktop UI as a stretched phone interface by default.

---

## Mobile

For mobile, consider:

- Safe areas.
- Keyboard insets.
- Back navigation.
- Touch targets.
- Lifecycle.
- Orientation.

Do not hard-code one device size.

---

## SafeArea

Use SafeArea where UI could conflict with system insets.

Do not wrap every screen automatically if Scaffold/layout already handles the relevant inset behavior.

Understand actual layout.

---

## Keyboard Insets

Use `MediaQuery.viewInsets` or established form/layout behavior when the on-screen keyboard affects content.

Do not add arbitrary bottom padding based on guessed keyboard heights.

---

## Localization

Use the project's localization system.

Do not hard-code user-facing strings throughout widgets if localization is part of the application architecture.

Do not introduce localization infrastructure to an app that explicitly does not require it.

---

## Generated Localization

If using Flutter localization generation, follow generated keys and ARB conventions.

Do not manually edit generated localization classes.

---

## Text Direction

Avoid assuming left-to-right layout.

Use direction-aware APIs where relevant:

```text
EdgeInsetsDirectional
AlignmentDirectional
start/end
```

especially in localized applications.

Do not convert all layout values mechanically if the project does not support RTL.

---

## Dates and Time

Use `DateTime` deliberately.

Be clear about:

- UTC.
- Local time.
- Serialization format.
- Time zones.

Do not hand-roll timezone conversions.

Use established packages when full timezone data is required.

---

## Duration

Use `Duration` for elapsed time.

Do not use `DateTime` subtraction manually where Duration semantics are clearer.

---

## Formatting Dates

Use the project's localization/date formatting package.

Do not manually concatenate month/day/year for user-facing dates in a localized app.

---

## Regex

Use RegExp when it clearly expresses validation/parsing.

Do not write enormous expressions when explicit parsing is easier.

Anchor validation properly.

Do not use regex to parse structured formats such as full JSON/XML/HTML.

---

## File I/O

For Dart VM/Flutter desktop/mobile:

```dart
File
Directory
RandomAccessFile
```

may be appropriate.

Use async file APIs when operations may block.

Do not use synchronous file I/O on UI-critical paths without understanding the impact.

---

## `readAsStringSync`

Avoid synchronous I/O in Flutter UI paths.

It can block the main isolate.

For short startup/config operations in a CLI, synchronous I/O may be perfectly reasonable.

Context matters.

---

## CLI Dart

For command-line Dart tools, do not apply Flutter architecture.

A simple CLI can use:

```text
main
ArgParser
File
stdout
stderr
```

without service layers and state-management frameworks.

Use architecture proportional to the program.

---

## Exit Codes

At CLI boundaries, use meaningful process exit codes.

Do not call:

```dart
exit(...)
```

deep inside reusable libraries.

Return/throw and let `main` decide process behavior.

---

## stdout vs stderr

Use:

```dart
stdout
```

for normal machine/user output.

Use:

```dart
stderr
```

for errors and diagnostics.

This matters for piping and scripting.

---

## Package Libraries

Reusable packages should generally avoid:

- Reading process environment globally.
- Calling exit.
- Configuring application logging.
- Performing work on import.
- Assuming Flutter exists unless the package is Flutter-specific.

Keep library boundaries clean.

---

## Top-Level Side Effects

Avoid significant work at import/load time.

Top-level initialization should remain lightweight and deterministic.

Do not start background operations from static/top-level initialization.

---

## Singletons

Do not introduce singleton patterns by default.

Global singleton state creates:

- Hidden dependencies.
- Test coupling.
- Lifecycle issues.

Use explicit ownership.

An immutable process-wide shared object may be acceptable when the semantics genuinely require it.

---

## Static Mutable State

Avoid mutable static fields for application state.

Prefer instance ownership through the actual app architecture.

---

## Mixins

Use mixins when multiple types genuinely share behavior that is natural to compose.

Do not use mixins merely to split a large class into multiple files.

If one class alone uses the behavior, private helpers may be clearer.

---

## Avoid Mixin Explosion

A class with:

```text
LoggingMixin
ValidationMixin
CachingMixin
ParsingMixin
NetworkingMixin
SerializationMixin
```

may be harder to understand than a few explicit collaborators.

Mixins should represent meaningful reusable capabilities.

---

## Typedefs

Use typedefs when they clarify complex function types or public APIs.

Good:

```dart
typedef WeaponFilter =
    bool Function(Weapon weapon);
```

Do not alias obvious simple types without semantic value.

---

## Callback APIs

Use callbacks for real event/callback relationships.

Do not pass callbacks through five layers merely to avoid a direct dependency.

Keep ownership obvious.

---

## VoidCallbacks

Flutter's:

```dart
VoidCallback
ValueChanged<T>
```

are appropriate for UI callbacks.

Do not invent duplicate callback typedefs when framework types already exist.

---

## `Function`

Avoid bare:

```dart
Function
```

when a precise callable type can be declared.

Prefer:

```dart
void Function()
Future<void> Function(String id)
```

Static typing should communicate call shape.

---

## Async Callbacks

Be deliberate whether a callback returns:

```dart
void
Future<void>
```

A caller may need to await completion.

Do not discard futures unintentionally.

---

## `unawaited`

Use an explicit unawaited mechanism only when fire-and-forget behavior is deliberate.

Do not simply ignore futures.

Ask:

- Who handles errors?
- Does lifecycle matter?
- Should completion be awaited?

---

## Fire-and-Forget Work

Avoid silent:

```dart
saveData();
```

when `saveData` returns a Future and failure matters.

Await it or intentionally route errors.

---

## Lifecycle

For Flutter StatefulWidgets, own and dispose resources consistently:

```text
AnimationController
TextEditingController
FocusNode
StreamSubscription
ChangeNotifier
Timer
```

Do not allocate them without a cleanup plan.

---

## `initState`

Use for one-time widget state initialization.

Do not perform work requiring inherited widgets before the lifecycle allows it.

Know when:

```text
initState
didChangeDependencies
didUpdateWidget
dispose
```

are appropriate.

---

## `didChangeDependencies`

Use when initialization depends on inherited widgets and must respond to dependency changes.

Do not put arbitrary repeated work there without understanding how often it can run.

---

## `didUpdateWidget`

Use when state must respond to updated widget configuration.

Do not manually compare every field without a real state transition need.

---

## Dispose

Call `super.dispose()` according to normal Flutter lifecycle convention after disposing owned resources unless framework/project guidance dictates otherwise.

Do not forget cleanup.

---

## Mounted

Check `mounted` after asynchronous gaps before updating widget state.

Example:

```dart
await load();

if (!mounted) {
  return;
}

setState(() {
  // ...
});
```

Do not add mounted checks to synchronous code.

---

## `setState`

Keep the callback itself synchronous.

Do not write:

```dart
setState(() async {
  await save();
});
```

Perform async work outside, then update state synchronously.

---

## Widget State Models

Do not duplicate derived state.

Bad:

```dart
String firstName;
String lastName;
String fullName;
```

when:

```dart
String get fullName =>
    '$firstName $lastName';
```

is sufficient.

Derived values should usually remain derived.

---

## Async State

Model loading/error/data states explicitly when they matter.

Do not use several loosely related booleans like:

```dart
bool loading;
bool failed;
bool loaded;
```

that can enter contradictory combinations.

Use an enum, sealed class, or framework-native async-state type where appropriate.

---

## Avoid Generic Loading Wrappers Everywhere

Do not build a universal:

```text
LoadingState<T>
Result<T>
AsyncResult<T>
OperationState<T>
```

for the whole application unless the project benefits from one consistent abstraction.

Frameworks like Riverpod may already provide one.

---

## Error UI

Show errors that users can act on.

Do not expose raw:

```text
SocketException
FormatException
stack traces
```

directly in production UI.

Translate at the presentation boundary while preserving diagnostic details for logging.

---

## Retry UI

Retry behavior should correspond to actual recoverable operations.

Do not add retry buttons mechanically to deterministic validation failures.

---

## Null UI State

Do not interpret every null as "loading."

Null should have one clear meaning.

Use explicit state when multiple meanings exist.

---

## Theming

Keep design tokens centralized.

Do not make every widget define unique:

```text
padding
radius
colors
text styles
shadows
```

unless the design actually differs.

Consistency beats one-off polish.

---

## Avoid AI-Looking Flutter Styling

Generated Flutter often defaults to:

- Huge gradients.
- Glassmorphism.
- Excessive shadows.
- Pill-shaped everything.
- Nested containers.
- Arbitrary spacing.
- Large rounded corners.
- Animated scale on every tap.
- Custom cards for every section.

Do not introduce these by default.

Match the actual product design.

---

## Avoid Hard-Coded UI Values

If the project has design tokens, use them.

Do not hard-code:

```dart
const Color(0xFF171717)
const EdgeInsets.all(17)
BorderRadius.circular(13)
```

throughout unrelated widgets.

Arbitrary values are acceptable when the design genuinely calls for them.

---

## Avoid Overusing Helper Methods in Widgets

A build method full of:

```dart
_buildHeader()
_buildBody()
_buildActions()
_buildFooter()
_buildSpacer()
```

is not automatically cleaner.

Use helpers where they represent meaningful chunks.

Otherwise a coherent widget tree may be easier to read.

---

## Avoid Private Widget Methods with State Dependencies Everywhere

If a UI fragment becomes substantial and independently meaningful, consider a real widget rather than a private method that closes over many parent fields.

But do not extract tiny fragments purely for style.

---

## Avoid Massive StatelessWidgets

A widget handling:

- Network loading.
- Filtering.
- Selection.
- Persistence.
- Animation.
- Navigation.
- Rendering.

may have too many responsibilities.

Split where real behavioral boundaries exist.

Do not split solely by line count.

---

## Avoid Rebuild-Driven Side Effects

Never put:

```dart
showDialog(...)
Navigator.push(...)
repository.save(...)
```

directly in build based solely on current state without a controlled effect mechanism.

Build should remain declarative.

---

## Avoid Calling `setState` After Dispose

Own async lifecycles properly.

Mounted checks are a safety mechanism, not a substitute for cancellation where cancellation is available.

---

## Avoid Async Work in Constructors

Flutter/Dart constructors cannot be async.

Do not fake it by starting hidden futures and leaving objects partially initialized.

Use:

```dart
static Future<Foo> create(...)
```

or explicit initialization when async construction is genuinely required.

---

## Avoid `Future<void>` Everywhere

Use it only when a function actually performs async work.

Keep synchronous logic synchronous.

---

## Avoid `dynamic` State

Do not store application state as:

```dart
dynamic state;
```

because several unrelated states exist.

Model the states properly.

---

## Avoid Map-Based Models

Bad for stable domain data:

```dart
Map<String, dynamic> weapon;
```

throughout widgets.

Prefer:

```dart
Weapon weapon;
```

after boundary parsing.

---

## Avoid Magic String Keys

If internal code repeatedly uses:

```dart
map['weapon_name']
map['damage']
map['category']
```

the data likely deserves a typed model.

---

## Avoid Huge Constructors

If a constructor has many related optional parameters, consider a config/model object or whether responsibilities need splitting.

Do not automatically introduce a builder pattern.

Named parameters already provide clarity.

---

## Required Parameters

Use:

```dart
required
```

for genuinely required named parameters.

Do not make fields optional just to reduce constructor friction.

Types should communicate requirements.

---

## Defaults

Provide defaults when they are real domain/UI defaults.

Do not silently default invalid or missing external data.

Boundary parsing should distinguish:

- Missing.
- Malformed.
- Intentionally optional.

---

## Covariance

Do not use `covariant` merely to force an override to compile.

It weakens some static guarantees and can introduce runtime checks.

Use only when the API genuinely requires narrowed parameter types.

---

## Operators

Only overload operators when semantics are natural.

Do not make:

```dart
weapon + attachment
```

mean something surprising.

Named methods are often clearer.

---

## Call Operator

Callable classes can be useful for focused operation objects.

Do not make every service callable through:

```dart
call()
```

merely because it looks elegant.

Use a descriptive method when the class has several operations or the action is not obvious.

---

## `toString`

Use for meaningful debugging/human representation.

Do not use `toString` as the application's serialization format.

Use explicit serialization APIs.

---

## Debug Output

Do not leak sensitive fields into generated `toString` implementations.

Be careful with tokens, credentials, and personal data.

---

## Equality Generators

If a package generates equality, use it consistently.

Do not combine manual and generated equality in confusing ways.

---

## Final Fields

Prefer `final` for fields that should not be reassigned.

Do not use mutable fields when state does not actually change.

`final` communicates ownership and simplifies reasoning.

---

## Local `final`

Use `final` for local variables where reassignment is not needed if that matches project style.

Do not add it so aggressively that simple code becomes visually noisy.

Both:

```dart
final weapon = loadWeapon();
```

and:

```dart
var weapon = loadWeapon();
```

can be idiomatic depending on whether reassignment occurs and project preference.

---

## `var`

Use `var` when the inferred type is obvious.

Good:

```dart
var count = 0;
```

or:

```dart
final weapon = parseWeapon(data);
```

Use explicit types when they communicate important API or domain meaning.

Do not annotate every local variable redundantly.

---

## `Object` vs `Object?`

Remember `Object` is non-nullable.

Use `Object?` for truly unknown values that may also be null.

Avoid dynamic when Object? plus narrowing works.

---

## `Never`

Use `Never` where the API genuinely never returns.

Do not introduce it into ordinary code merely because it is a precise bottom type.

---

## `void`

Use `void` for operations whose result callers should ignore.

Do not return meaningless booleans or null solely to indicate completion.

---

## Static Methods

Use static methods when behavior belongs conceptually to a type but not an instance.

Do not use classes as namespaces for unrelated static methods.

Top-level functions are idiomatic Dart.

---

## Libraries

Avoid old-style library partitioning unless the codebase uses it.

Modern Dart generally favors normal files and imports.

Do not introduce:

```text
part
part of
```

without a framework/generation reason.

---

## Imports

Keep imports specific and organized.

Do not import large implementation files merely for one unrelated helper.

Use package imports according to project convention.

---

## Relative vs Package Imports

Follow the project's established style.

Do not convert all:

```dart
import '../models/weapon.dart';
```

to:

```dart
import 'package:app/models/weapon.dart';
```

or vice versa during unrelated work.

Consistency matters.

---

## Import Prefixes

Use:

```dart
import 'foo.dart' as foo;
```

when names conflict or a namespace improves clarity.

Do not prefix every import mechanically.

---

## Show/Hide

Use:

```dart
show
hide
```

sparingly when controlling public imported names genuinely improves clarity.

Do not over-manage imports to make dependencies look smaller.

---

## Barrel Files

Avoid huge:

```text
exports.dart
index.dart
all.dart
```

barrels that re-export the whole application.

They can:

- Hide dependencies.
- Increase coupling.
- Create cycles.
- Make navigation harder.

Use focused public package entry points where appropriate.

---

## Package Public APIs

Reusable packages should expose deliberate entry points.

Do not export internal implementation files "just in case."

Use `lib/src/` conventions when the project follows them.

---

## Circular Dependencies

If package/module boundaries are producing circular dependencies, do not solve the issue by moving everything into `common`.

Reconsider responsibility and dependency direction.

---

## Generic `core` Package

Avoid turning:

```text
core/
```

into a dumping ground for:

```text
utils
services
constants
extensions
helpers
base classes
```

Use domain-focused organization.

---

## Constants Files

Do not create one giant:

```text
constants.dart
```

containing unrelated values from the entire app.

Keep constants near the feature or design system that owns them.

---

## Utils Files

Avoid:

```text
utils.dart
helpers.dart
common.dart
```

for unrelated functions.

Prefer focused modules:

```text
asset_paths.dart
weapon_parsing.dart
date_formatting.dart
```

or keep private helpers near usage.

---

## Error Handling in Flutter

Do not catch everything in widgets and display generic:

```text
Something went wrong
```

without preserving useful error information.

Keep user messaging and technical diagnostics separate.

---

## Flutter Error Boundaries

Use framework error handling mechanisms appropriately for top-level failures.

Do not install global handlers merely to suppress exceptions.

Unexpected errors should still be visible during development.

---

## Zone Error Handling

Use zones only when application-level error handling genuinely requires them.

Do not use `runZonedGuarded` as a replacement for normal error handling.

---

## Assertions

Use `assert` for developer/programmer invariants that apply in debug mode.

Do not use assertions for user input or runtime validation because asserts may be disabled in production.

---

## Debug-Only Behavior

Keep debug-only instrumentation behind framework-supported mechanisms where necessary.

Do not make production correctness depend on debug asserts.

---

## `kDebugMode`

Use for actual debug-only behavior.

Do not scatter debug-mode branches through domain logic.

---

## Security

Do not:

- Hard-code secrets.
- Disable TLS verification casually.
- Store sensitive credentials in source assets.
- Trust remote JSON shape blindly.
- Construct SQL unsafely.
- Expose tokens in logs.
- Use insecure random generation for secrets.

Use platform and library security features correctly.

---

## Secure Storage

Use an established secure-storage mechanism for credentials when the platform requires it.

Do not assume SharedPreferences is secure storage.

---

## Randomness

Use secure randomness for:

- Tokens.
- Keys.
- Nonces.
- Security identifiers.

Do not use normal pseudo-random APIs for security-sensitive values.

---

## Web Security

For Flutter Web, remember client-side code and embedded configuration can be inspected by users.

Do not embed private API secrets in frontend code.

---

## Environment Configuration

Compile-time variables, dotenv-style packages, or config files do not automatically make client-side secrets private.

Anything shipped to the client should be treated as visible.

---

## Performance

Before optimizing Flutter/Dart, profile:

- Frame rendering.
- CPU.
- Network.
- Image loading.
- Large lists.
- Rebuilds.
- Serialization.
- Database queries.

Do not optimize based on style folklore.

---

## Avoid Premature `const` Obsession

`const` is useful, but don't redesign APIs solely to maximize const constructors.

Use it where natural.

---

## Avoid Premature Rebuild Optimization

Do not extract fifty widgets because "smaller widgets rebuild less."

Widget rebuilds are cheap in many cases.

Measure real frame/build costs.

---

## Avoid Premature Isolates

Do not move every parser to an isolate.

Use isolates for work that measurably blocks the main isolate.

---

## Avoid Premature Memoization

Do not cache every derived value.

If a calculation is cheap, recomputing may be simpler than invalidation logic.

---

## Avoid Excessive Abstraction

Before adding:

```text
Manager
Service
Provider
Repository
Factory
Controller
UseCase
Interactor
Coordinator
Handler
Processor
Facade
Registry
Adapter
```

ask whether the type represents a real responsibility.

Do not generate architecture by suffix.

---

## Avoid Pass-Through Layers

Bad:

```dart
class WeaponService {
  WeaponService(this.repository);

  final WeaponRepository repository;

  Future<Weapon?> find(String id) {
    return repository.find(id);
  }
}
```

if the service adds no behavior.

Every layer should earn its existence.

---

## Avoid Repository + Service + UseCase Chains

Bad architecture:

```text
Widget
  -> ViewModel
    -> UseCase
      -> Service
        -> Repository
          -> ApiClient
```

for one simple API call.

Layers are justified when each one owns meaningful transformation, policy, or coordination.

Do not create them to satisfy an architecture diagram.

---

## Avoid Interface + Impl Naming

Avoid Java-style:

```text
WeaponRepository
WeaponRepositoryImpl
WeaponService
WeaponServiceImpl
```

unless multiple implementations genuinely exist and the project already follows that naming.

Concrete implementations should often have meaningful names:

```text
HttpWeaponRepository
CachedWeaponRepository
SqliteWeaponStore
```

when differences actually matter.

---

## Avoid Abstract Base Widgets

Do not create custom abstract widget classes merely to share a few methods.

Use composition, helper functions, or reusable widgets.

---

## Avoid Global Event Buses

Do not add a global event bus for simple communication.

Prefer:

- Direct callbacks.
- Existing state management.
- Notifiers.
- Streams where actual event streams exist.

Global event buses obscure data flow.

---

## Avoid Stream-Driven Everything

A stream is not required for every changing value.

For a single current value, a notifier/state object may be clearer.

Use streams when temporal event sequences actually matter.

---

## Avoid FutureBuilder Everywhere

FutureBuilder is useful, but not a state-management architecture.

If the same data is shared, refreshed, cached, paginated, or mutated, use the project's appropriate state layer.

---

## Avoid Extension Method Abuse

Do not put application logic into:

```dart
extension WeaponContextExtensions on BuildContext
```

or generic Object/String extensions merely to shorten calls.

Explicit code is often easier to search and understand.

---

## Avoid Generic `Result` Types by Default

A result type can be useful if the project has adopted one.

Do not introduce:

```dart
Result<T>
Either<L, R>
Option<T>
```

for ordinary Dart code merely to mimic functional languages.

Dart already supports:

- Nullable values.
- Futures.
- Exceptions.
- Sealed classes.

Use the model that best fits the codebase.

---

## Avoid Functional Programming for Its Own Sake

Do not convert straightforward logic into:

```text
Either
TaskEither
Option
Reader
monadic chains
```

unless the project deliberately uses that ecosystem and it materially clarifies behavior.

Direct Dart is often easier to maintain.

---

## Avoid Deep Generic Wrappers

Bad conceptual API:

```dart
Future<Result<Page<List<WeaponDto>>>>
```

unless each wrapper represents a real useful abstraction.

Simplify return contracts where possible.

---

## Avoid Overly Defensive JSON Parsing Everywhere

Boundary parsing should be robust.

Do not then repeat the same type/null checks in every internal consumer.

Normalize once.

---

## Avoid `try/catch` Around Every Await

Bad:

```dart
try {
  return await repository.load();
} catch (error) {
  rethrow;
}
```

This adds no value.

Catch only to:

- Recover.
- Add context.
- Translate.
- Log at an appropriate boundary.
- Cleanup.

---

## Avoid Empty Catch Blocks

Never silently swallow exceptions:

```dart
try {
  await save();
} catch (_) {}
```

unless ignored failure is explicitly intended and documented.

---

## Avoid `Future.delayed` Fixes

Do not use arbitrary delays to fix:

- Widget lifecycle.
- Navigation ordering.
- Animation state.
- Race conditions.
- Test timing.

Use actual framework lifecycle or synchronization.

---

## Avoid Huge Model Objects

Do not create one AppState class containing every screen's state.

Keep state ownership close to the feature that owns it.

Global state should truly be global.

---

## Avoid Static Global Services

Do not create:

```dart
class Api {
  static final client = Dio();
}
```

throughout the codebase merely for easy access.

Explicit ownership makes configuration/testing/lifecycle clearer.

---

## Avoid Magic Singletons

Singletons should be rare.

If the project already uses one, respect its architecture.

Do not introduce singleton state because passing one dependency feels inconvenient.

---

## Avoid Mutation Through Shared Models

If several widgets/services share the same mutable object reference, state changes can become difficult to reason about.

Use the project's intended state model.

Do not defensively clone everything either.

---

## Avoid Arbitrary Copying

Copy immutable models when the architecture requires state updates.

Do not duplicate large models repeatedly without a reason.

Generated `copyWith` is useful only when copy-style updates are actually part of the state architecture.

---

## Avoid Excessive `copyWith`

If code becomes:

```dart
state = state.copyWith(
  user: state.user.copyWith(
    profile: state.user.profile.copyWith(
      settings: ...
    ),
  ),
);
```

the state model may be too deeply nested.

Reconsider ownership/boundaries rather than automatically generating more helpers.

---

## Avoid Giant BuildContext-Based Services

Business services should not require BuildContext just to access:

- Theme.
- Localization.
- Navigator.
- Providers.

Pass actual domain inputs instead.

Keep UI/framework concerns at UI boundaries.

---

## Avoid Excessive Dependency Providers

If using a provider framework, do not wrap every pure constant or helper in a provider.

Providers should represent state/dependencies with meaningful lifecycle or substitution needs.

---

## Avoid Architecture for "Testability"

Do not create layers solely because AI-generated architecture says they make code testable.

Simple concrete code with injected collaborators is already testable.

Testability should emerge from clear boundaries, not ceremonial interfaces.

---

## Avoid Generic Base Classes

Do not create:

```text
BaseRepository
BaseService
BaseController
BaseViewModel
BaseState
```

to share trivial behavior.

Inheritance couples features and often hides responsibilities.

Use composition.

---

## Avoid Deep Folder Trees

Do not create:

```text
features/
  weapon/
    data/
      datasource/
        remote/
        local/
      models/
      repositories/
    domain/
      entities/
      repositories/
      usecases/
    presentation/
      controllers/
      pages/
      widgets/
```

for a small feature unless the project already uses this structure and the feature complexity warrants it.

File organization should make navigation easier, not impressive.

---

## Comments

Do not narrate obvious code.

Bad:

```dart
// Check if the weapon is null.
if (weapon == null) {
  // Return if the weapon does not exist.
  return;
}
```

Better:

```dart
if (weapon == null) {
  return;
}
```

Write comments when explaining:

- Why a workaround exists.
- Flutter lifecycle nuance.
- Platform limitation.
- Protocol quirk.
- Performance tradeoff.
- Non-obvious domain behavior.

Prefer **why**, not **what**.

---

## Avoid AI-Looking Comments

Avoid phrases like:

```text
This method is responsible for...
This ensures that...
The following code...
In order to...
It is important to note that...
This provides a robust and scalable solution...
```

Bad:

```dart
// This method is responsible for ensuring that all
// weapon data is properly validated before use.
```

Better:

```dart
// Legacy manifests may omit fields added after version 3.
```

---

## Documentation Comments

Use:

```dart
///
```

for public APIs where documentation adds value.

Do not generate verbose docs for obvious private members.

Bad:

```dart
/// Gets the weapon name.
String get name => _name;
```

when the API is self-explanatory.

---

## Avoid Decorative Comments

Do not add:

```dart
// =====================================
// WIDGET BUILDING
// =====================================
```

unless the repository explicitly uses that style.

Normal organization should usually make this unnecessary.

---

## Avoid Placeholder Code

Do not leave:

```dart
throw UnimplementedError();
```

or:

```dart
// TODO: add validation
```

in finished code unless scaffolding was explicitly requested.

Implement the requested functionality.

---

## TODOs

Legitimate TODOs should explain a real unresolved dependency or issue.

Do not leave speculative future enhancements in production code.

---

## Before Finishing

Review the change and remove or correct:

- Unnecessary classes.
- Unnecessary abstract interfaces.
- Pass-through services.
- Repository/service/use-case chains with no value.
- Generic AI-style names.
- Excessive `late`.
- Excessive `!`.
- `dynamic` used to silence types.
- Repeated nullable checks after validation.
- Unnecessary casts.
- Premature generics.
- Extension-method clutter.
- Async methods without async work.
- Unnecessary Completers.
- Fire-and-forget Futures.
- Leaked subscriptions/controllers/timers.
- State-management machinery for trivial local state.
- Widget fragmentation.
- Deep Container/Padding/SizedBox nesting.
- Hard-coded design values that should use the theme.
- Debug `print` / `debugPrint`.
- Empty catch blocks.
- Redundant try/catch.
- Huge `Map<String, dynamic>` usage outside boundaries.
- Global mutable state.
- Singleton services without justification.
- Placeholder TODOs.
- Dead code.
- Unused imports.
- Speculative configuration.
- Speculative extensibility.
- Unrelated refactors.

Then run the project's established checks where available.

Typical Dart checks may include:

```text
dart format .
dart analyze
dart test
```

For Flutter projects:

```text
dart format .
flutter analyze
flutter test
```

Projects using code generation may additionally require an established command such as:

```text
dart run build_runner build
```

or a project-specific script.

Do not assume these exact commands exist.

Inspect:

```text
pubspec.yaml
analysis_options.yaml
build.yaml
Makefile
Taskfile
CI configuration
repository documentation
```

and use the repository's established workflow.

The final code should look like it naturally belongs in the repository rather than like a generic AI-generated Dart or Flutter solution.

It should feel like Dart written by an experienced maintainer: strongly typed without being ceremonial, null-safe without escape hatches, asynchronous only where necessary, simple in its abstractions, disciplined about state ownership, and comfortable using straightforward classes, functions, and widgets instead of layering patterns for their own sake.