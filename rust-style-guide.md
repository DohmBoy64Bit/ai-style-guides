# Rust Style Guide

Write Rust as an experienced professional Rust developer maintaining a real production codebase.

The goal is clean, idiomatic, maintainable Rust—not code that looks generated, over-engineered, excessively generic, or written as a tutorial.

The most important rule:

> Do not optimize for demonstrating Rust features. Optimize for producing the smallest idiomatic production-quality change that an experienced maintainer would reasonably write.

## General Principles

Prefer:

- Simple solutions over clever abstractions.
- Existing project conventions over personal preferences.
- Idiomatic ownership and borrowing.
- Explicit data flow.
- Small, focused changes.
- Enums for meaningful state.
- Concrete types until abstraction is justified.
- `Result` for recoverable errors.
- Clear code over explanatory comments.
- Iterators when they improve readability.
- Loops when they are clearer than iterator chains.
- Standard library features before custom abstractions.
- Direct implementation over speculative extensibility.

Do not refactor unrelated code unless required by the task.

Do not introduce traits, generics, macros, async runtimes, synchronization primitives, wrappers, or architectural layers unless they solve an actual problem.

Do not write Rust as if it were Java, C#, or TypeScript.

---

## Match the Existing Codebase

Before writing code, inspect nearby files and follow the repository's established conventions for:

- Module organization.
- Naming.
- Error handling.
- `Result` types.
- Logging.
- Async runtime.
- Trait usage.
- Serialization.
- Testing.
- Feature flags.
- Dependency injection patterns.
- Public API style.
- Clippy configuration.
- Rustfmt configuration.
- MSRV.
- Edition.
- Workspace structure.

Check relevant project files such as:

```text
Cargo.toml
Cargo.lock
rust-toolchain.toml
rustfmt.toml
clippy.toml
deny.toml
```

Consistency with the repository is more important than imposing a preferred style.

Do not modernize the edition, MSRV, dependency structure, error system, or async runtime as part of unrelated work.

---

## Rust Version and MSRV

Determine the project's minimum supported Rust version before using newer syntax or APIs.

Do not assume the newest stable Rust version.

Check:

```toml
rust-version = "1.82"
```

or repository documentation and CI configuration.

Use syntax and standard-library APIs supported by the declared MSRV.

Do not introduce a newer feature just because it is available locally.

---

## Naming

Follow standard Rust naming conventions:

- `snake_case` for variables, functions, methods, and modules.
- `PascalCase` for structs, enums, traits, and variants.
- `SCREAMING_SNAKE_CASE` for constants and statics.
- Lifetimes should normally be short and conventional.

Good:

```rust
weapon_definition
asset_path
package_name
load_result

load_package()
resolve_asset()
parse_weapon_data()
```

Avoid vague AI-style names:

```rust
data_manager
process_handler
utility_helper
generic_service
result_processor
enhanced_processor
operation_manager
```

Prefer domain terminology.

Bad:

```rust
let data = get_data();
let result = process_data(data);
```

Better:

```rust
let weapon = load_weapon_definition();
let stats = parse_weapon_stats(&weapon);
```

Do not make names excessively verbose.

Avoid:

```rust
let successfully_parsed_weapon_configuration_result =
    parse_weapon_configuration();
```

when:

```rust
let weapon_config = parse_weapon_configuration();
```

is obvious.

---

## Concrete Types First

Prefer concrete types until multiple implementations or true generic reuse exists.

Good:

```rust
fn parse_weapon(data: &WeaponData) -> Weapon {
    // ...
}
```

Avoid prematurely writing:

```rust
fn parse_entity<T, U, P>(data: T, parser: P) -> U
where
    P: Parser<T, Output = U>,
{
    // ...
}
```

when the function parses one concrete domain type.

Generic Rust is powerful, but unnecessary generics create:

- Harder compiler errors.
- Larger APIs.
- More trait bounds.
- More monomorphization.
- More cognitive overhead.

Abstractions should earn their complexity.

---

## Functions First

Prefer functions for stateless operations.

Good:

```rust
fn parse_weapon_stats(data: &WeaponData) -> WeaponStats {
    WeaponStats {
        damage: data.damage,
        fire_rate: data.fire_rate,
    }
}
```

Do not automatically create:

```rust
struct WeaponStatsParser;

impl WeaponStatsParser {
    fn parse(&self, data: &WeaponData) -> WeaponStats {
        // ...
    }
}
```

unless the parser actually owns state, configuration, dependencies, or behavior that belongs together.

Rust modules already provide namespacing.

---

## Structs

Use structs for data with meaningful named fields and behavior.

Example:

```rust
struct WeaponStats {
    damage: f32,
    fire_rate: f32,
    magazine_size: u32,
}
```

Do not create wrapper structs around every primitive or collection automatically.

Use newtypes when they provide real value such as:

- Type safety.
- Unit distinction.
- Validation.
- API boundaries.
- Trait implementations.

Do not create newtypes merely to make the domain look more modeled.

---

## Newtypes

A newtype is useful when two values have the same underlying representation but distinct meaning.

Good:

```rust
struct WeaponId(String);
struct PlayerId(String);
```

This prevents accidental interchange.

Do not write:

```rust
struct WeaponName(String);
struct WeaponDescription(String);
struct WeaponCategoryName(String);
```

unless those types enforce different behavior or invariants.

---

## Enums

Use enums for meaningful alternatives and state.

Good:

```rust
enum LoadResult {
    Loaded(Weapon),
    NotFound,
    InvalidFormat(ParseError),
}
```

Prefer enums over boolean flags representing multiple states.

Bad:

```rust
struct LoadState {
    loaded: bool,
    not_found: bool,
    failed: bool,
}
```

Enums make invalid states harder to represent.

---

## Avoid Boolean State Explosions

If multiple booleans encode one conceptual state, use an enum.

Bad:

```rust
struct ConnectionState {
    connected: bool,
    reconnecting: bool,
    failed: bool,
}
```

Better:

```rust
enum ConnectionState {
    Connected,
    Reconnecting,
    Failed,
}
```

Do not replace every boolean with an enum, though.

A genuine yes/no property is fine:

```rust
enabled: bool
```

---

## Option

Use `Option<T>` when absence is a normal and expected state.

Good:

```rust
fn find_weapon(id: &str) -> Option<&Weapon> {
    // ...
}
```

Do not use sentinel values such as:

```rust
-1
""
0
```

to represent absence when `Option` expresses the contract better.

---

## Result

Use `Result<T, E>` for operations that may fail in meaningful ways.

Example:

```rust
fn load_weapon(path: &Path) -> Result<Weapon, LoadError> {
    // ...
}
```

Do not return:

```rust
Option<Weapon>
```

when callers need to distinguish "not found" from parsing or I/O failure.

Likewise, do not create an elaborate error type when simple absence is normal.

---

## `Option<Result<T, E>>` and `Result<Option<T>, E>`

Choose the structure that matches the semantics.

Usually:

```rust
Result<Option<T>, E>
```

means:

- The operation itself can fail.
- A missing value is not an error.

Example:

```rust
fn find_weapon(id: &str) -> Result<Option<Weapon>, DatabaseError>
```

Do not choose nested types mechanically.

Make the states meaningful.

---

## Avoid `unwrap()`

Do not use `unwrap()` in production paths unless failure truly represents an invariant violation that cannot reasonably occur.

Bad:

```rust
let weapon = weapons.get(id).unwrap();
```

when the ID may be missing.

Prefer:

```rust
let weapon = weapons
    .get(id)
    .ok_or_else(|| WeaponError::NotFound(id.to_owned()))?;
```

or return `Option` if absence is expected.

`unwrap()` is often acceptable in:

- Tests.
- Small internal setup code with proven invariants.
- Static initialization where failure would be a programming error.

Use judgment.

---

## Avoid `expect()` as Fake Error Handling

`expect()` is slightly more informative than `unwrap()`, but it is still a panic.

Bad:

```rust
let config = read_config().expect("failed to read config");
```

in library code where the caller could handle the error.

Use `expect()` only when panic is genuinely the correct behavior and the message explains the invariant.

Good invariant-style message:

```rust
let current = current.expect(
    "current weapon must exist after successful selection"
);
```

Do not use generic messages such as:

```rust
expect("error")
```

---

## Panic

Use `panic!` for programmer bugs and impossible internal states.

Do not use panic for ordinary runtime failures such as:

- Missing files.
- Invalid user input.
- Network failures.
- Missing configuration.
- Parse errors.

Libraries should generally return errors instead of terminating the process.

Application entry points may decide how to handle unrecoverable startup failures.

---

## Error Handling

Catch and translate errors only where doing so adds value.

Use `?` to propagate errors naturally.

Good:

```rust
let text = fs::read_to_string(path)?;
let weapon = parse_weapon(&text)?;
```

Do not manually match every error just to re-return it.

Bad:

```rust
let text = match fs::read_to_string(path) {
    Ok(text) => text,
    Err(error) => return Err(error.into()),
};
```

Prefer:

```rust
let text = fs::read_to_string(path)?;
```

when no extra behavior is required.

---

## The `?` Operator

Use `?` freely when it makes error flow clear.

Do not avoid it in favor of verbose `match` expressions unless you need to:

- Recover.
- Add context.
- Transform the error.
- Handle one variant differently.

---

## Error Context

Add context at meaningful boundaries.

With `anyhow`, for example:

```rust
let text = fs::read_to_string(path)
    .with_context(|| format!("failed to read {}", path.display()))?;
```

Do not attach repetitive context at every layer.

Avoid messages like:

```text
failed to process data
failed to execute operation
operation failed
```

Use domain-specific context.

---

## `anyhow`

Use `anyhow` primarily in application code where callers do not need to match on structured error types.

Good uses:

- CLI applications.
- Binaries.
- Top-level orchestration.
- Internal tools.

Do not automatically use `anyhow::Error` for reusable library APIs if callers need meaningful error variants.

Follow existing project conventions.

---

## `thiserror`

Use `thiserror` when defining structured library or domain errors provides value.

Example:

```rust
#[derive(Debug, thiserror::Error)]
enum LoadError {
    #[error("weapon file not found: {0}")]
    NotFound(PathBuf),

    #[error("invalid weapon data")]
    InvalidData(#[from] ParseError),

    #[error("failed to read weapon file")]
    Io(#[from] std::io::Error),
}
```

Do not create dozens of variants for failures callers will never distinguish.

---

## Avoid Giant Error Enums

Do not create one global error enum covering the entire application unless the architecture truly benefits from it.

Bad conceptual pattern:

```text
AppError
  Io
  Parse
  Config
  Network
  Database
  Rendering
  Cache
  Authentication
  Validation
  Plugin
  ...
```

This often creates tight coupling.

Prefer errors scoped to meaningful boundaries.

---

## Source Errors

Preserve underlying errors where useful.

Do not convert every error into a string.

Bad:

```rust
Err(MyError::Other(error.to_string()))
```

when the source error can be retained.

Preserving structured source errors improves debugging and error chains.

---

## Avoid Stringly Typed Errors

Avoid APIs returning:

```rust
Result<T, String>
```

for substantial production code when a structured error type would be more useful.

For small internal helpers or prototypes, `String` may be acceptable.

Do not create a custom enum for a one-line script solely to avoid `String`.

---

## Borrow Instead of Clone

Prefer borrowing when ownership transfer is unnecessary.

Good:

```rust
fn parse_weapon(data: &WeaponData) -> Weapon {
    // ...
}
```

Do not default to:

```rust
fn parse_weapon(data: WeaponData) -> Weapon
```

if the caller still needs `data`.

Avoid cloning just to silence borrow checker errors.

Bad:

```rust
let name = weapon.name.clone();
```

when a borrow is enough:

```rust
let name = &weapon.name;
```

---

## Do Not Fight the Borrow Checker with Clones

Repeated `.clone()` calls often indicate unclear ownership.

Before cloning, ask:

- Does this function really need ownership?
- Can it borrow instead?
- Can ownership move later?
- Can data be reorganized to avoid overlapping borrows?

Cloning is not inherently bad.

Clone when:

- Independent ownership is actually required.
- The type is cheap.
- Simpler ownership is worth the small cost.
- The codebase intentionally prefers it.

Do not treat "zero clones" as a goal.

---

## `Copy`

Use `Copy` for small plain-value types where implicit duplication is natural.

Examples:

```rust
u32
bool
f32
small coordinate structs
```

Do not derive `Copy` on large or semantically ownership-heavy types merely for convenience.

---

## References

Prefer borrowed parameters when ownership is not needed:

```rust
fn normalize_name(name: &str) -> String
```

rather than:

```rust
fn normalize_name(name: String) -> String
```

if the function does not need to consume the input.

Likewise prefer slices:

```rust
fn total_damage(weapons: &[Weapon]) -> u32
```

instead of:

```rust
fn total_damage(weapons: &Vec<Weapon>) -> u32
```

when only slice behavior is required.

---

## Use `&str` for Borrowed String Input

Prefer:

```rust
fn find_weapon(name: &str)
```

over:

```rust
fn find_weapon(name: &String)
```

unless an API specifically requires `String`.

`&str` is more flexible and idiomatic.

---

## Use Slices for Borrowed Collections

Prefer:

```rust
fn process_weapons(weapons: &[Weapon])
```

over:

```rust
fn process_weapons(weapons: &Vec<Weapon>)
```

when vector-specific behavior is unnecessary.

Similarly:

```rust
&[u8]
```

instead of:

```rust
&Vec<u8>
```

for byte input.

---

## Owned Types at Boundaries

Owned types are often appropriate when:

- Storing data.
- Returning newly created values.
- Crossing task boundaries.
- Moving data into long-lived state.
- Spawning threads or tasks.

Do not borrow aggressively when ownership makes the design cleaner.

Ownership should match lifetime semantics, not ideology.

---

## Lifetimes

Let the compiler infer lifetimes when possible.

Good:

```rust
fn find_weapon<'a>(
    weapons: &'a [Weapon],
    id: &str,
) -> Option<&'a Weapon> {
    // ...
}
```

when explicit lifetime relationships are required.

Do not add named lifetimes mechanically to simple functions where elision works:

```rust
fn weapon_name(weapon: &Weapon) -> &str {
    &weapon.name
}
```

is preferable to unnecessarily explicit lifetime syntax.

---

## Lifetime Names

Use conventional short names unless domain-specific naming genuinely improves understanding:

```rust
'a
'b
```

Do not use verbose lifetime names such as:

```rust
'weapon_data_lifetime
```

unless the relationship is unusually complex and the name actually helps.

---

## Traits

Use traits for real shared behavior, abstraction boundaries, or generic capabilities.

Good:

```rust
trait PackageProvider {
    fn load(&self, path: &Path) -> Result<Package, LoadError>;
}
```

when multiple providers exist or callers genuinely depend on the abstraction.

Do not create a trait for every struct.

Avoid:

```rust
trait WeaponParser {
    fn parse(&self, data: &WeaponData) -> Weapon;
}

struct DefaultWeaponParser;
```

when there will only ever be one parser and direct functions are simpler.

---

## Trait Bounds

Keep trait bounds as simple as possible.

Avoid signatures like:

```rust
fn process<T, U, V, E>(...)
where
    T: IntoIterator<Item = U> + Send + Sync + 'static,
    U: Borrow<V> + Clone,
    V: SomeTrait + AnotherTrait + ?Sized,
    E: Error + Send + Sync + 'static,
{
    // ...
}
```

unless the reusable API genuinely requires that flexibility.

Concrete code is easier to understand, compile, and debug.

---

## `impl Trait`

Use `impl Trait` when it makes APIs simpler.

Example:

```rust
fn weapon_names(
    weapons: &[Weapon],
) -> impl Iterator<Item = &str> {
    weapons.iter().map(|weapon| weapon.name.as_str())
}
```

Do not use opaque return types merely to show off advanced Rust.

Return concrete types when they are simpler and part of the useful API.

---

## Trait Objects

Use:

```rust
dyn Trait
```

when runtime polymorphism is actually required.

Example:

```rust
Vec<Box<dyn Plugin>>
```

may make sense for dynamically heterogeneous plugins.

Do not reach for trait objects when an enum can represent a small known set of variants more clearly.

---

## Enum vs Trait Object

Prefer enums when:

- The implementations are known and closed.
- Pattern matching is useful.
- Allocation can be avoided.
- Behavior is simple.

Prefer trait objects when:

- Implementations are open-ended.
- Plugins or external implementations are expected.
- Runtime polymorphism is genuinely required.

Do not use dynamic dispatch by default.

---

## Associated Types

Use associated types when a trait logically has one associated output type.

Do not introduce associated types where a normal generic parameter is clearer.

Choose the type-system tool that makes the API simplest for callers.

---

## Default Trait Methods

Default methods are useful for genuinely shared behavior.

Do not turn traits into inheritance hierarchies.

If a trait has many default methods and one required method solely to simulate a base class, reconsider the design.

---

## Generics

Use generics when code truly applies to multiple types.

Good:

```rust
fn first<T>(items: &[T]) -> Option<&T> {
    items.first()
}
```

Do not genericize domain-specific logic.

Bad:

```rust
fn parse_entity<TInput, TOutput>(
    input: TInput,
) -> TOutput {
    // ...
}
```

for one weapon parser.

---

## Const Generics

Use const generics when compile-time sizes or constants genuinely matter.

Do not introduce them for ordinary runtime configuration.

They are powerful but increase API complexity.

---

## Macros

Use macros when they eliminate real repetitive syntax or enable functionality normal functions cannot express.

Good examples include:

- Declarative DSLs.
- Repeated trait implementations.
- Compile-time code generation.
- Derive macros.

Do not create a macro for a few repeated lines.

Bad:

```rust
macro_rules! create_weapon {
    ($name:expr, $damage:expr) => {
        Weapon::new($name, $damage)
    };
}
```

when a function already does the job.

---

## Declarative Macros

Keep `macro_rules!` macros small and understandable.

Avoid complex token-tree gymnastics when ordinary Rust would be clearer.

Macros should reduce maintenance burden, not move complexity into a harder-to-debug layer.

---

## Procedural Macros

Procedural macros are a major maintenance commitment.

Do not introduce one unless code generation or syntax transformation genuinely justifies it.

Prefer derive crates or existing ecosystem tools where appropriate.

---

## Iterators

Use iterators when they clearly express a transformation.

Good:

```rust
let names: Vec<_> = weapons
    .iter()
    .filter(|weapon| weapon.enabled)
    .map(|weapon| weapon.name.as_str())
    .collect();
```

Do not create long iterator chains that require repeated rereading.

Use a loop when:

- Multiple side effects occur.
- Error handling differs per item.
- Early exit matters.
- Intermediate state is meaningful.
- The chain becomes difficult to understand.

---

## Avoid Iterator Gymnastics

Bad:

```rust
let result = items
    .iter()
    .filter_map(...)
    .flat_map(...)
    .scan(...)
    .filter(...)
    .map(...)
    .fold(...);
```

if a simple loop would be clearer.

Rust does not award points for maximum iterator density.

---

## `map` vs `and_then`

Use combinators where they express the logic naturally.

Good:

```rust
let name = weapon
    .metadata
    .as_ref()
    .map(|metadata| metadata.display_name.as_str());
```

Do not chain combinators so deeply that a `match` or `if let` is easier to read.

---

## `if let`

Use `if let` when handling one relevant pattern.

Good:

```rust
if let Some(weapon) = weapon {
    process_weapon(weapon);
}
```

Do not use a full `match` when one branch matters and the rest is ignored.

---

## `let else`

Use `let ... else` for early exits when supported by the MSRV.

Good:

```rust
let Some(weapon) = weapons.get(id) else {
    return Ok(None);
};
```

This can be clearer than nested `if let`.

Do not use it automatically if a normal `match` better communicates several cases.

---

## `match`

Use `match` when multiple meaningful variants need handling.

Good:

```rust
match state {
    State::Ready => start(),
    State::Loading => wait(),
    State::Failed(error) => report(error),
}
```

Do not replace simple boolean checks with `match` merely to look Rust-like.

---

## Exhaustive Matching

Take advantage of exhaustive matching for enums.

Do not immediately add:

```rust
_ => {}
```

to every match.

A wildcard can hide newly added variants.

Prefer explicit variants when API evolution should force review.

Use `_` when other variants truly share one behavior or future variants should intentionally be ignored.

---

## `matches!`

Use `matches!` for simple pattern checks:

```rust
if matches!(state, State::Ready | State::Cached) {
    // ...
}
```

Do not use it where extracting values is necessary.

---

## Loops

Use straightforward loops freely.

Good:

```rust
for asset in assets {
    if asset.kind != AssetKind::Weapon {
        continue;
    }

    let Some(weapon) = parse_weapon(asset)? else {
        continue;
    };

    weapons.push(weapon);
}
```

This can be clearer than forcing the same logic into iterator combinators.

---

## `for` vs Manual Iteration

Prefer:

```rust
for weapon in weapons {
    // ...
}
```

over manually calling `.next()` unless iterator control itself matters.

Use the simplest construct.

---

## Collections

Choose collections based on actual access patterns.

Use:

- `Vec<T>` for ordered sequences.
- `HashMap<K, V>` for key-value lookup.
- `HashSet<T>` for uniqueness and membership.
- `BTreeMap` or `BTreeSet` when ordering matters.
- `VecDeque` for efficient front/back queue operations.

Do not use a more complex collection merely because it seems more efficient theoretically.

---

## `Vec`

Use `Vec` as the default growable sequence.

Do not wrap every vector in a custom collection type unless domain behavior justifies it.

---

## HashMap

Use `HashMap` for normal lookup tables.

Example:

```rust
let mut weapons_by_id = HashMap::new();

for weapon in weapons {
    weapons_by_id.insert(weapon.id.clone(), weapon);
}
```

Do not use a map if linear scan is perfectly adequate for a tiny fixed collection.

---

## Entry API

Use the `entry` API when it actually simplifies insert/update logic.

Good:

```rust
counts
    .entry(category)
    .and_modify(|count| *count += 1)
    .or_insert(1);
```

Do not use `entry` for simple unconditional inserts.

---

## Strings

Use `String` for owned mutable or stored UTF-8 text.

Use `&str` for borrowed string input.

Do not convert repeatedly between `String` and `&str` without need.

Avoid:

```rust
name.to_string().as_str()
```

patterns caused by unclear ownership.

---

## String Construction

Use:

```rust
format!()
```

when formatting is required.

Do not call `format!` for simple static strings.

For repeated concatenation in a loop, consider `String::with_capacity` only when profiling or obvious scale justifies it.

Do not prematurely optimize tiny strings.

---

## `to_string()` vs `to_owned()`

Use the clearest method for the context.

For `&str` to `String`, both are valid:

```rust
name.to_owned()
name.to_string()
```

Follow project conventions.

Do not churn code between them as style cleanup.

---

## Paths

Use `Path` and `PathBuf` for filesystem paths.

Prefer:

```rust
fn load_package(path: &Path)
```

over:

```rust
fn load_package(path: &str)
```

when the value is genuinely a filesystem path.

Use `PathBuf` for owned stored paths.

Do not convert paths to UTF-8 strings unless required.

---

## Avoid `to_str().unwrap()`

Filesystem paths are not guaranteed to be valid UTF-8.

Bad:

```rust
let path = path.to_str().unwrap();
```

Prefer APIs that accept `Path`.

When display text is needed:

```rust
path.display()
```

is often sufficient.

---

## Filesystem I/O

Use standard library helpers when they fit:

```rust
fs::read_to_string(path)
fs::read(path)
fs::write(path, data)
```

Do not manually open and buffer files if the whole-file helper already matches the use case.

Use buffered I/O for large or streaming workloads.

---

## Buffered I/O

Use `BufReader` and `BufWriter` when repeated or streaming I/O benefits from buffering.

Do not wrap every one-shot small file read in buffering ceremony.

---

## Serialization

Use established crates such as `serde` when the project already uses them and serialization is non-trivial.

Do not hand-write JSON parsing for structured data without a reason.

Likewise, do not introduce `serde` to parse one tiny fixed format if simple code is clearer and the dependency cost matters.

---

## Serde Derives

Derive only what is needed.

Avoid mechanically adding:

```rust
Serialize
Deserialize
Clone
Debug
Default
PartialEq
Eq
Hash
```

to every type.

Each derive is part of the type's contract.

For example, deriving `Clone` should reflect intentional cloneability, not convenience.

---

## `Default`

Derive or implement `Default` when there is a meaningful default value.

Do not invent arbitrary defaults to make construction easier.

A weapon ID may not have a meaningful default.

If construction requires data, require that data.

---

## Builders

Use builder patterns when construction genuinely has many optional parameters or staged configuration.

Do not create:

```rust
WeaponBuilder
```

for a struct with three obvious required fields.

Prefer direct construction:

```rust
Weapon {
    id,
    name,
    damage,
}
```

when it is clear.

---

## Constructors

Rust constructors are conventionally associated functions like:

```rust
impl Weapon {
    fn new(id: WeaponId, name: String) -> Self {
        Self { id, name }
    }
}
```

Use `new()` when it provides useful invariants or convenience.

Do not write trivial constructors solely to hide obvious struct initialization unless the project's API style prefers them.

---

## Smart Constructors

Use constructors that validate invariants when the type should never exist in an invalid state.

Example:

```rust
impl WeaponName {
    fn new(value: String) -> Result<Self, NameError> {
        if value.trim().is_empty() {
            return Err(NameError::Empty);
        }

        Ok(Self(value))
    }
}
```

Do not validate trivial trusted internal values repeatedly.

---

## Typestate

Typestate can provide strong compile-time guarantees.

Do not use typestate for ordinary application flows unless the state machine and safety benefits clearly justify the added generic complexity.

It is easy to make APIs harder to use than the problem requires.

---

## Ownership in APIs

Design APIs around ownership intentionally.

Ask:

- Does the function need to store the value?
- Can it borrow?
- Should it consume the value?
- Is returning ownership useful?
- Is cloning unavoidable or simply convenient?

Do not design signatures solely to satisfy the first implementation.

---

## Return References Carefully

Returning borrowed references can be efficient but couples lifetimes.

Do not return references where returning an owned small value creates a much simpler API.

Balance ergonomics and allocation.

---

## Cow

Use `Cow<'a, str>` or other `Cow` types when callers sometimes borrow and sometimes need owned data.

Do not introduce `Cow` for ordinary string parameters just to avoid hypothetical allocations.

It adds complexity and lifetime coupling.

---

## Interior Mutability

Use:

```rust
Cell
RefCell
Mutex
RwLock
```

only when mutation through shared ownership is genuinely required.

Do not reach for interior mutability merely to avoid redesigning ownership.

---

## RefCell

`RefCell` is appropriate for single-threaded runtime-checked interior mutability.

Do not use it as a general workaround for borrow checker problems.

Repeated borrow panics indicate the state model may be wrong.

---

## Mutex

Use `Mutex` for shared mutable state across threads or async tasks when mutual exclusion is needed.

Do not wrap everything in:

```rust
Arc<Mutex<T>>
```

by default.

Ask whether:

- Ownership can be transferred instead.
- Message passing is simpler.
- State can be partitioned.
- A read-only `Arc<T>` is enough.
- The value even needs sharing.

---

## RwLock

Use `RwLock` when many readers and relatively few writers make it beneficial.

Do not assume it is automatically faster than `Mutex`.

Contention behavior depends on the workload.

Prefer simplicity unless measurements justify otherwise.

---

## Arc

Use `Arc<T>` for shared ownership across threads/tasks when needed.

Do not use `Arc` merely because async code exists.

Many async values can be owned directly or borrowed within a task.

---

## Rc

Use `Rc<T>` for shared ownership in single-threaded code.

Do not use `Rc<RefCell<T>>` as a default object graph pattern.

It can be useful, but often indicates ownership relationships should be reconsidered.

---

## Weak

Use `Weak` references to break ownership cycles when using `Rc` or `Arc`.

Do not introduce weak references without a real cyclic ownership problem.

---

## Atomics

Use atomics only for small shared state where atomic semantics are appropriate.

Do not replace mutexes with atomics merely for perceived performance.

Memory ordering is subtle.

Prefer:

```rust
Ordering::Relaxed
```

only when its semantics are actually sufficient.

Do not guess about ordering.

---

## Unsafe

Avoid `unsafe` unless it is genuinely required.

Before using unsafe, ask whether:

- A safe standard-library API exists.
- An established crate solves the problem.
- The performance benefit is measured.
- FFI requires it.

Keep unsafe blocks as small as possible.

Document the invariants that make unsafe code sound.

---

## Unsafe Comments

For non-trivial unsafe code, explain the safety invariant.

Good:

```rust
// SAFETY: `ptr` points to `len` initialized bytes owned by `buffer`
// and remains valid for the duration of this slice.
unsafe {
    std::slice::from_raw_parts(ptr, len)
}
```

Do not write vague comments like:

```rust
// SAFETY: this is safe
```

---

## FFI

Keep unsafe FFI boundaries narrow.

Validate inputs before crossing the boundary where practical.

Translate raw pointers and error codes into safe Rust types promptly.

Do not let raw pointers leak through large parts of the application.

---

## Async

Use async only for operations that benefit from asynchronous I/O or concurrency.

Do not make a function async merely because its caller is async.

Bad:

```rust
async fn weapon_count(weapons: &[Weapon]) -> usize {
    weapons.len()
}
```

Prefer:

```rust
fn weapon_count(weapons: &[Weapon]) -> usize {
    weapons.len()
}
```

---

## Async Runtime

Use the runtime already chosen by the project.

Common examples:

```text
Tokio
async-std
smol
```

Do not add a second async runtime casually.

Do not migrate runtimes as part of unrelated work.

---

## Avoid Blocking in Async Code

Do not perform blocking work directly on an async executor thread when it can meaningfully block progress.

Examples include:

- Large filesystem operations.
- CPU-heavy parsing.
- Blocking network libraries.
- Long synchronous subprocess waits.

Use the runtime's blocking facilities where appropriate.

Do not move trivial work into `spawn_blocking` automatically.

---

## Tokio

When using Tokio, follow established patterns.

Use:

```rust
tokio::spawn
tokio::time
tokio::sync
```

when the task actually needs them.

Do not wrap every operation in a spawned task.

A spawned task adds:

- Lifetime constraints.
- `Send` requirements.
- Error-handling complexity.
- Cancellation behavior.

Call async functions directly when concurrency is unnecessary.

---

## `tokio::spawn`

Use task spawning when work should execute concurrently and independently enough to justify a separate task.

Do not spawn just to "make it async."

If you immediately await a task handle, spawning may provide no value.

---

## Join

Use concurrency primitives that match failure semantics.

For a fixed number of independent futures, tools such as:

```rust
tokio::join!
try_join!
```

may be appropriate.

Do not introduce concurrency where ordering or resource limits matter.

---

## Async Streams

Use streams when data arrives asynchronously over time.

Do not convert ordinary collections into async streams merely to make APIs appear scalable.

---

## Cancellation

Understand cancellation behavior.

Dropping a future can cancel it.

Do not assume cleanup always runs after cancellation unless the code guarantees it.

For long-lived tasks, define ownership and shutdown semantics explicitly.

---

## Channels

Use channels for message passing when they simplify ownership and concurrency.

Do not introduce channels merely to avoid shared borrowing in single-threaded code.

Choose:

- `mpsc`.
- `oneshot`.
- `broadcast`.
- `watch`.

based on actual communication semantics.

---

## Backpressure

For producer-consumer systems, consider bounded channels where unbounded growth could exhaust memory.

Do not use unbounded channels by default for potentially unlimited workloads.

---

## Locks Across Await

Avoid holding synchronous locks across `.await`.

This can cause deadlocks or block executor threads.

Even async locks should not be held longer than necessary.

Prefer:

1. Lock.
2. Copy or extract the needed state.
3. Drop the guard.
4. Await.

when semantics allow it.

---

## Drop Guards Explicitly When Needed

If lock lifetime is unclear, scope it explicitly:

```rust
let value = {
    let state = state.lock().await;
    state.current.clone()
};

do_async_work(value).await;
```

Do not rely on complicated implicit drop timing in concurrency-sensitive code.

---

## Threading

Use threads for real parallel or blocking workloads.

Do not introduce threads into a simple application merely because Rust makes them safe.

Prefer simpler synchronous code when concurrency does not provide value.

---

## Rayon

Use Rayon for CPU-parallel data processing when:

- Work is substantial.
- Data parallelism is natural.
- The project already uses it or the dependency is justified.

Do not parallelize tiny loops automatically.

Parallelism has overhead.

---

## Logging and Tracing

Use the project's established observability stack.

Common crates include:

```text
tracing
log
env_logger
tracing-subscriber
```

Do not mix logging ecosystems casually.

---

## Logging

Log meaningful events and failures.

Avoid:

```rust
info!("starting function");
info!("processing item");
info!("function completed");
```

unless those events matter operationally.

Prefer contextual logs:

```rust
warn!(
    asset_path = %path.display(),
    "weapon asset could not be resolved"
);
```

Do not log the same error at every propagation layer.

---

## Tracing Spans

Use spans for meaningful request/task boundaries.

Do not create a span for every tiny helper function.

Observability should help diagnosis, not flood traces.

---

## Structured Logging

When using `tracing`, prefer structured fields over manually formatted messages when practical.

Good:

```rust
tracing::warn!(
    weapon_id = %weapon_id,
    path = %path.display(),
    "failed to load weapon"
);
```

This produces better searchable telemetry.

---

## Debug Output

Do not leave:

```rust
dbg!(...)
println!("here")
println!("test")
```

in production library or application code unless output is intentional.

Use logging or remove debugging statements before finishing.

---

## `println!`

`println!` is fine for:

- CLI user output.
- Simple scripts.
- Intentional stdout protocols.

Do not use it as application logging when a logging system exists.

---

## Modules

Keep modules cohesive.

Do not create deep module hierarchies without need.

Avoid:

```text
weapon/
  services/
    managers/
      processors/
        handlers/
```

for a small project.

Prefer domain-based organization.

---

## Avoid One-Type-Per-File Dogma

Rust modules can contain multiple closely related types and functions.

Do not create separate files for every:

```text
Weapon
WeaponError
WeaponConfig
WeaponParser
WeaponBuilder
```

unless that improves actual navigation and ownership.

---

## `mod.rs`

Follow the existing project structure.

Do not convert between:

```text
foo.rs
foo/bar.rs
```

and:

```text
foo/mod.rs
```

as unrelated cleanup.

Both styles are valid.

---

## Visibility

Keep APIs as private as possible.

Prefer:

```rust
fn helper()
pub(crate) fn internal_api()
pub fn public_api()
```

based on actual consumers.

Do not mark everything `pub` for convenience.

A smaller public surface is easier to maintain.

---

## `pub(crate)`

Use `pub(crate)` for APIs that genuinely need crate-wide access but should not be public externally.

Do not use it automatically when private module restructuring would be cleaner.

---

## Re-exports

Use `pub use` intentionally to define a clear public API.

Do not re-export every internal item through the crate root.

Large re-export surfaces hide module boundaries and increase coupling.

---

## Prelude Modules

Avoid custom preludes unless a large project genuinely benefits from a stable commonly imported set of items.

They can obscure where names come from.

Do not create a `prelude` for five imports.

---

## Dependencies

Use the project's existing dependencies where practical.

Before adding a crate, ask:

- Can the standard library handle this?
- Does the project already have an equivalent dependency?
- Is the crate maintained?
- Is the dependency weight justified?
- Does it affect MSRV?
- Does it bring significant transitive dependencies?

Do not add a crate for five lines of simple code.

Do not hand-roll security-sensitive or standards-heavy functionality merely to avoid a dependency.

---

## Cargo Features

Use feature flags for meaningful optional functionality.

Do not feature-gate ordinary code merely to make the crate appear configurable.

Feature combinations increase testing complexity.

Every feature adds maintenance cost.

---

## Default Features

Be deliberate about default features on dependencies.

Do not write:

```toml
default-features = false
```

automatically.

Only disable defaults when there is a concrete reason such as:

- Binary size.
- Platform compatibility.
- Avoiding unwanted runtime integrations.

---

## Workspace Dependencies

Follow workspace dependency conventions where present.

Do not duplicate versions across crates if the workspace intentionally centralizes them.

Likewise, do not restructure dependency management without need.

---

## Semver

For public crates, treat public types, trait implementations, and feature behavior as API.

Avoid unnecessary public API churn.

Do not expose internal dependency types unless that coupling is intentional.

---

## Feature Creep

Do not add:

- Plugin systems.
- Dynamic loading.
- Serialization formats.
- Async support.
- Trait abstractions.
- Parallelism.
- Configuration knobs.

unless the task actually requires them.

Rust makes sophisticated architecture possible; that does not mean it should always be used.

---

## Derives

Derive only traits that make semantic sense.

Good:

```rust
#[derive(Debug, Clone, PartialEq)]
struct Weapon {
    // ...
}
```

when all three are genuinely useful.

Do not mechanically derive:

```rust
Default
Eq
Hash
Ord
Serialize
Deserialize
Clone
Copy
```

for every type.

---

## `Debug`

Deriving `Debug` is usually useful for internal structs and errors.

Be careful with types containing secrets or sensitive information.

Do not log `Debug` representations of credentials or tokens.

---

## `Clone`

Derive `Clone` when callers legitimately need independent ownership.

Do not derive it merely because the borrow checker became inconvenient.

The ability to clone can encourage accidental allocation and hide ownership problems.

---

## `PartialEq` and `Eq`

Derive equality when value comparison makes semantic sense.

Do not derive it just to make tests easier if equality is not meaningful for the type.

---

## Hash

Derive `Hash` when values are legitimate hash map/set keys.

Do not derive it automatically.

---

## Ordering

Derive or implement ordering only when the domain has a meaningful total or partial order.

Do not define arbitrary ordering merely so `.sort()` compiles.

Use explicit sort keys where appropriate.

---

## Default Values

Do not invent meaningless defaults.

Bad conceptual example:

```rust
impl Default for WeaponId {
    fn default() -> Self {
        Self(String::new())
    }
}
```

if an empty weapon ID is invalid.

Require valid construction instead.

---

## `From` and `Into`

Use `From` for natural, infallible conversions.

Example:

```rust
impl From<String> for WeaponName {
    fn from(value: String) -> Self {
        Self(value)
    }
}
```

only if every `String` is valid.

Do not implement `From` when conversion can fail.

Use `TryFrom` instead.

---

## `TryFrom`

Use `TryFrom` for validation or fallible conversion.

Example:

```rust
impl TryFrom<String> for WeaponName {
    type Error = NameError;

    fn try_from(value: String) -> Result<Self, Self::Error> {
        // ...
    }
}
```

This makes invalid states explicit.

---

## `AsRef`

Use `AsRef` sparingly for ergonomic APIs that naturally accept several borrowed forms.

Do not make every path/string parameter generic over `AsRef` automatically.

This:

```rust
fn load(path: &Path)
```

is often simpler than:

```rust
fn load<P: AsRef<Path>>(path: P)
```

especially for internal functions.

Use the generic form when caller ergonomics genuinely benefit.

---

## `Into`

Avoid taking `impl Into<String>` on every string parameter.

It can be convenient for public constructors, but it also:

- Hides allocations.
- Complicates inference.
- Makes signatures more generic than necessary.

For internal code, `String` or `&str` is often clearer.

---

## `Borrow`

Use `Borrow` only when collection-key semantics or generic borrowing genuinely require it.

Do not use it simply to make APIs accept more input forms.

---

## Functional Style

Rust supports functional patterns well, but do not force them.

This is fine:

```rust
let mut weapons = Vec::new();

for asset in assets {
    if let Some(weapon) = parse_weapon(asset)? {
        weapons.push(weapon);
    }
}
```

Do not rewrite it into a dense chain solely to avoid mutation.

---

## Mutation

Local mutation is normal and idiomatic.

Use `mut` when state naturally changes.

Do not contort code to avoid all mutation.

At the same time, keep mutable scope narrow.

Good:

```rust
let mut total = 0;

for weapon in weapons {
    total += weapon.damage;
}
```

---

## Narrow Mutable Scope

Declare values immutable by default.

Use `mut` only where needed.

Do not mark large structures mutable "just in case."

Narrow mutation makes ownership easier to understand.

---

## Shadowing

Rust shadowing can be useful for transformations.

Good:

```rust
let input = input.trim();
let input = input.parse::<u32>()?;
```

Do not overuse shadowing when values represent meaningfully different concepts.

Use clearer names when necessary.

---

## Tuple Returns

Use tuples for small obvious groupings.

Example:

```rust
fn dimensions() -> (u32, u32)
```

Avoid returning large tuples like:

```rust
(Weapon, Stats, Metadata, PathBuf, bool, ErrorState)
```

when named fields would be clearer.

Use a struct.

---

## Tuple Structs

Use tuple structs when the wrapped value has one clear semantic identity.

Example:

```rust
struct WeaponId(u64);
```

Do not use them for several unrelated fields where names improve clarity.

---

## Public Struct Fields

Expose fields publicly only when direct access is part of the intended API.

Private fields with methods can preserve invariants.

Do not automatically hide every field behind getters either.

For simple data types, public fields may be entirely appropriate.

---

## Getters

Do not write Java-style getters for every field.

Bad:

```rust
impl Weapon {
    fn get_name(&self) -> &str {
        &self.name
    }
}
```

If the field is intentionally public:

```rust
weapon.name
```

is enough.

If encapsulation matters, use idiomatic accessor naming:

```rust
fn name(&self) -> &str
```

not `get_name()`.

---

## Setters

Avoid generic setters when mutation should preserve invariants.

Bad:

```rust
fn set_name(&mut self, name: String) {
    self.name = name;
}
```

when names require validation.

Prefer domain operations:

```rust
fn rename(&mut self, name: WeaponName)
```

if the behavior has meaning.

---

## Methods vs Free Functions

Use methods when behavior clearly belongs to the type.

Example:

```rust
weapon.is_loaded()
```

Use free functions when behavior is external transformation or coordination.

Do not turn every utility function into an inherent method merely to look object-oriented.

---

## Associated Functions

Use associated functions for constructors or operations closely tied to a type but not requiring `self`.

Example:

```rust
Weapon::from_manifest(...)
```

Do not use associated functions as static utility buckets.

---

## Drop

Implement `Drop` only when deterministic cleanup is truly required.

Be cautious because custom `Drop` affects move semantics and can make ownership harder.

Prefer RAII types from the standard library or dependencies where possible.

---

## RAII

Use RAII naturally.

Resources such as:

- Files.
- Locks.
- Temporary state.
- Connections.

should be released when their owning guards drop.

Do not manually emulate cleanup APIs when the type system already provides them.

---

## Lock Guards

Keep lock guard scopes small.

Do not hold guards across unrelated work.

This reduces contention and makes ownership clearer.

---

## Testing

Use the project's existing testing style.

Unit tests commonly live near the code:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn returns_none_for_missing_weapon() {
        // ...
    }
}
```

Integration tests may live under:

```text
tests/
```

Follow the repository's organization.

---

## Test Behavior, Not Implementation

Test observable behavior.

Good:

```rust
#[test]
fn parse_weapon_rejects_missing_id() {
    // ...
}
```

Avoid tests whose only purpose is to confirm internal helper calls or private structure.

Rust makes refactoring easier when tests focus on contracts.

---

## Avoid Overbuilt Test Fixtures

Do not build:

```text
WeaponFixtureFactory
MockWeaponBuilder
TestContextManager
```

for a few simple tests.

Use direct values where possible.

Extract helpers only when repetition becomes genuinely burdensome.

---

## Mocking

Rust often needs less mocking than OOP-heavy languages.

Prefer:

- Real lightweight implementations.
- In-memory stores.
- Temporary directories.
- Small test doubles.
- Trait abstractions only when production architecture already benefits from them.

Do not introduce a trait solely so one test can mock a function.

---

## Temporary Files

Use established crates or standard patterns for temporary resources.

If the project already uses `tempfile`, use it.

Do not invent random temp paths manually.

---

## Property Testing

Use tools like `proptest` or `quickcheck` when the domain benefits from broad generated inputs.

Good candidates:

- Parsers.
- Serializers.
- Numeric algorithms.
- State machines.

Do not introduce property testing for trivial fixed behavior.

---

## Snapshot Testing

Use snapshot tests when large structured output is genuinely easier to review as a snapshot.

Do not snapshot everything.

Snapshots can hide meaningful assertions and create noisy updates.

---

## Async Tests

Use the runtime's normal test macros when appropriate.

For Tokio:

```rust
#[tokio::test]
async fn loads_weapon() {
    // ...
}
```

Do not make synchronous tests async unnecessarily.

---

## Benchmarks

Add benchmarks when performance is important and measurable.

Do not add Criterion or custom benchmark harnesses for trivial code.

Benchmark actual hot paths.

---

## Clippy

Treat Clippy as a useful static analysis tool, not an absolute design authority.

Run the project's configured lint level.

Common command:

```bash
cargo clippy --all-targets --all-features
```

Do not assume that exact command fits every workspace.

Follow CI and project scripts.

---

## Clippy Allows

Do not add:

```rust
#[allow(clippy::...)]
```

merely to silence generated code.

Use targeted allows only when:

- The lint genuinely does not fit.
- The reason is understood.
- A clearer implementation would be worse.

Avoid crate-wide allowances casually.

---

## Rustfmt

Use rustfmt.

Do not manually fight formatting.

Typical command:

```bash
cargo fmt --check
```

Do not hand-align fields or arguments in ways rustfmt will undo.

---

## Cargo Check

Use:

```bash
cargo check
```

for fast compile validation where appropriate.

For changes affecting tests or features, run the relevant broader commands too.

---

## Tests and Build

Typical checks may include:

```bash
cargo fmt --check
cargo check
cargo clippy
cargo test
```

or workspace-specific variants.

Do not assume every repository uses the same commands.

Inspect CI and `Cargo.toml`.

---

## Warnings

Treat warnings seriously.

Do not suppress warnings globally to make generated code compile.

Unused imports, dead code, and unnecessary mutability often reveal incomplete work.

Fix the cause.

---

## Dead Code

Do not leave:

```rust
#[allow(dead_code)]
```

around unused generated abstractions.

Remove code that is not needed.

Use dead-code allowances only for legitimate platform, feature-gated, or externally invoked cases.

---

## Feature-Gated Code

When code is conditional, ensure all supported feature combinations compile when practical.

Do not create impossible or untested feature matrices.

Keep feature interactions simple.

---

## Platform-Specific Code

Use `cfg` attributes deliberately.

Example:

```rust
#[cfg(windows)]
fn platform_path() {
    // ...
}
```

Do not scatter platform checks throughout generic logic when a small module boundary is clearer.

---

## Conditional Compilation

Avoid complex nested `cfg` expressions unless necessary.

Too many compile-time branches make code hard to reason about and test.

Use runtime branching when compile-time exclusion provides no real benefit.

---

## Build Scripts

Keep `build.rs` small and deterministic.

Do not put application logic in build scripts.

Build scripts can complicate reproducibility and incremental compilation.

Use them only for genuine build-time tasks.

---

## Procedural Build Logic

Prefer Cargo features, standard configuration, and existing tooling before adding custom build orchestration.

Do not make `build.rs` a general-purpose setup script.

---

## Unsafe Dependencies

Be cautious with low-level crates.

Do not reject a dependency merely because it contains unsafe internally; much of the Rust ecosystem uses carefully audited unsafe code.

Focus on:

- Maintenance quality.
- Security posture.
- Ecosystem trust.
- Whether the dependency is appropriate.

---

## Security

Do not weaken safety for convenience.

Never casually:

- Disable TLS validation.
- Use unchecked deserialization.
- Execute shell strings built from untrusted input.
- Bypass path validation where confinement matters.
- Use unsafe to avoid borrow-checker work.
- Store secrets in logs.
- Ignore integer overflow where it matters.

Use established safe APIs.

---

## Integer Conversions

Be deliberate about integer conversions.

Avoid unchecked:

```rust
value as usize
```

when the value may be negative or exceed the target range.

Use:

```rust
usize::try_from(value)?
```

when correctness requires bounds checking.

Do not replace all casts mechanically if the range is trivially guaranteed.

---

## `as` Casts

Use `as` when the conversion is understood and safe in context.

Be cautious with:

- Narrowing.
- Signed/unsigned conversions.
- Float/integer conversion.
- Pointer casts.

Do not introduce complex conversion wrappers for obviously safe literal or widening cases.

---

## Overflow

Understand release vs debug overflow behavior.

Use:

```rust
checked_add
saturating_add
wrapping_add
overflowing_add
```

when the domain requires explicit overflow semantics.

Do not add checked arithmetic everywhere without need.

---

## Indexing

Direct indexing can panic:

```rust
items[index]
```

Use it when the index is guaranteed valid by the logic.

Use:

```rust
items.get(index)
```

when out-of-range is a legitimate runtime possibility.

Do not replace every index operation with `.get()` mechanically.

---

## UTF-8 Strings

Do not index strings by byte position as if they were character arrays.

Rust intentionally prevents this.

Use:

```rust
chars()
char_indices()
bytes()
```

depending on whether the operation is about Unicode scalar values or bytes.

Do not implement naive slicing based on assumptions about ASCII unless the input is guaranteed ASCII.

---

## Unicode

Be clear about what "character" means.

Possible interpretations include:

- Bytes.
- Unicode scalar values.
- Grapheme clusters.

For user-visible text, grapheme semantics may require a dedicated crate.

Do not overcomplicate ASCII-only protocols with Unicode machinery.

---

## FFI Strings

When interfacing with C, use appropriate `CString` and `CStr` APIs.

Do not pass Rust string pointers directly without respecting null termination and lifetime requirements.

Keep conversion boundaries explicit.

---

## Networking

Use the project's established networking stack.

Do not add a new HTTP client if one already exists.

Common crates include:

```text
reqwest
hyper
ureq
```

Choose based on existing runtime and needs.

---

## HTTP Errors

Check HTTP status explicitly when required.

With `reqwest`, for example:

```rust
let response = client
    .get(url)
    .send()
    .await?
    .error_for_status()?;
```

Do not treat every successful transport as a successful application response.

---

## Timeouts

Set meaningful timeouts for external network operations where indefinite waits would be harmful.

Do not add tiny arbitrary timeouts without understanding real latency.

---

## Retries

Retry only transient failures.

Do not retry deterministic parse failures or invalid credentials.

Use bounded retries with sensible backoff.

Do not create custom retry frameworks if the project already uses one.

---

## Database Code

Use the established database library directly when its API is already appropriate.

Do not automatically add:

```text
Repository
Service
Manager
Provider
Adapter
```

around simple queries.

A direct SQLx or Diesel query can be perfectly maintainable.

---

## Transactions

Use transactions for logically atomic groups of operations.

Do not create transaction abstractions around isolated reads.

Keep transaction scope as small as practical.

Avoid holding a database transaction across slow unrelated network calls unless the semantics require it.

---

## SQL

Use parameterized queries.

Do not interpolate untrusted values into SQL strings.

Use the database crate's placeholder and binding APIs.

---

## CLI Code

Use the project's existing CLI crate if one exists.

Commonly:

```text
clap
argh
lexopt
```

Do not add a full CLI framework for a trivial two-argument internal tool unless it provides real value.

---

## Library vs Binary Boundaries

Library code should generally:

- Return structured errors.
- Avoid exiting the process.
- Avoid global logging setup.
- Avoid reading process-wide configuration implicitly.

Binary entry points can:

- Parse CLI args.
- Configure logging.
- Read environment variables.
- Decide exit codes.
- Print user-facing errors.

Keep boundaries clear.

---

## `std::process::exit`

Avoid calling `process::exit` deep inside library or domain code.

Return an error and let the application boundary decide.

Use explicit exits at the real process boundary when necessary.

---

## Environment Variables

Read and validate environment variables near startup.

Do not scatter:

```rust
std::env::var(...)
```

throughout unrelated modules.

Prefer a validated configuration object where the application benefits from one.

---

## Configuration

Do not create a massive config struct for three constants.

Likewise, do not hard-code deployment-specific values throughout the codebase.

Use the simplest structure that matches actual configurability.

---

## Global State

Avoid mutable global state.

Rust makes this intentionally difficult.

Do not work around that difficulty with:

```rust
static mut
```

or global mutexes unless there is a real process-wide shared resource.

Prefer explicit ownership.

---

## `OnceLock` and `LazyLock`

Use once-initialized globals for genuinely process-wide immutable or safely shared state.

Examples:

- Compiled regexes.
- Static configuration after startup.
- Expensive immutable tables.

Do not put arbitrary application state into globals merely for convenience.

---

## Regex

Compile reused regexes once when appropriate.

Do not add a regex crate for a problem that simple string operations solve more clearly.

Avoid enormous unreadable regexes.

---

## Parsing

Prefer explicit parsers for non-trivial formats.

Do not implement complex protocols with fragile split chains such as:

```rust
line.split(':').nth(3).unwrap()
```

when malformed input is possible.

Make failure behavior explicit.

---

## Avoid Chained `unwrap()` Parsing

Bad:

```rust
let damage = line
    .split(':')
    .nth(1)
    .unwrap()
    .trim()
    .parse::<u32>()
    .unwrap();
```

Prefer:

```rust
let (_, value) = line
    .split_once(':')
    .ok_or(ParseError::MissingSeparator)?;

let damage = value
    .trim()
    .parse::<u32>()?;
```

This is clearer and preserves failure information.

---

## `split_once`

Use helpers such as:

```rust
split_once
strip_prefix
strip_suffix
trim
```

when they express the parsing operation directly.

Do not manually calculate string indices unnecessarily.

---

## Closures

Use closures for small local behavior.

Good:

```rust
weapons.sort_by_key(|weapon| weapon.name.clone());
```

But be mindful that cloning in sort keys may be wasteful.

Prefer borrowing forms where supported.

Do not move complex business logic into enormous closures.

Extract named functions when the logic becomes substantial.

---

## Sorting

Use appropriate sorting APIs.

Examples:

```rust
sort()
sort_by()
sort_by_key()
sort_unstable()
```

Choose stable vs unstable ordering based on requirements.

Do not use `sort_unstable` purely for theoretical speed when preserving equal-element order matters.

---

## Performance

Prefer clear code until performance actually matters.

Do not prematurely:

- Add `unsafe`.
- Add arenas.
- Add custom allocators.
- Add SIMD.
- Parallelize loops.
- Preallocate every collection.
- Replace strings with interners.
- Use lock-free data structures.

Measure first.

Rust is already efficient in straightforward code.

---

## Capacity Preallocation

Use:

```rust
Vec::with_capacity(...)
String::with_capacity(...)
```

when the required size is known or a large repeated allocation is obvious.

Do not estimate capacities everywhere without evidence that it matters.

---

## Allocation

Avoid gratuitous allocation when borrowing is simple.

But do not make APIs unreadable to eliminate a tiny allocation that is irrelevant to the workload.

Performance and clarity both matter.

---

## Zero-Copy

Zero-copy designs can be valuable in parsers and high-performance systems.

Do not force lifetime-heavy zero-copy APIs onto ordinary application code where owned values are simpler and the allocation cost is negligible.

---

## Boxing

Use `Box<T>` when:

- Recursive types require indirection.
- Large enum variants need size reduction.
- Trait objects require it.
- Ownership transfer benefits from heap allocation.

Do not box values merely to reduce stack usage without evidence.

---

## Large Enums

Be aware that enum size is determined by the largest variant.

If one variant is much larger than others and the enum is heavily replicated, boxing may help.

Do not optimize this without measuring or at least inspecting actual size impact.

---

## Arena Allocation

Use arenas when object lifetimes and allocation volume clearly benefit from them.

Do not introduce arenas into normal CRUD or application code.

They increase lifetime coupling and architectural complexity.

---

## Pin

Use `Pin` only where pinning semantics are actually required.

Most application code should not manipulate `Pin` directly.

Do not introduce self-referential designs merely because Rust makes them technically possible.

---

## Futures

Avoid hand-writing `Future` implementations unless necessary.

Use `async fn` and async blocks for ordinary async code.

Manual futures are low-level infrastructure.

---

## Streams and Sinks

Use `Stream`/`Sink` traits when the application genuinely operates on asynchronous sequences or channels.

Do not expose them in simple APIs that naturally return one result.

---

## Dynamic Dispatch

Do not default to:

```rust
Box<dyn Trait>
Arc<dyn Trait>
```

for flexibility.

Use them when runtime heterogeneity or plugin boundaries justify them.

Static dispatch and enums are often simpler.

---

## Monomorphization

Be mindful that generic-heavy APIs can increase compile time and binary size.

Do not genericize every helper solely for reuse.

Concrete internal functions are often better.

---

## Compile Times

Avoid unnecessary macro-heavy or generic-heavy designs in large codebases.

Compile time is part of developer experience.

Do not introduce heavyweight dependencies for trivial convenience.

---

## Public API Ergonomics

For public crates, design APIs for callers, not just internal implementation.

Consider:

- Ownership.
- Borrowing.
- Error types.
- Feature flags.
- Trait bounds.
- Semver stability.

Do not leak internal complexity through public signatures unnecessarily.

---

## Sealed Traits

Use sealed traits when external implementations would break invariants or future compatibility.

Do not seal traits by default.

Open implementation can be valuable when that is part of the crate's purpose.

---

## Marker Traits

Use marker traits only when they represent meaningful compile-time capability.

Do not invent marker traits merely to categorize internal types.

Enums or normal methods may be clearer.

---

## PhantomData

Use `PhantomData` only when the compiler needs to understand ownership, variance, or type association not represented by stored fields.

Do not add it as a generic design ornament.

If you are reaching for `PhantomData`, make sure the type-level design actually requires it.

---

## Zero-Sized Types

ZSTs can be useful for marker states, stateless strategies, or type-level APIs.

Do not create them where a free function or enum variant would be clearer.

---

## Typing Invalid States Away

Use the type system to prevent meaningful invalid states.

Good examples include:

- Distinct ID types.
- Enums instead of incompatible booleans.
- Validated newtypes.
- Non-empty domain types when critical.

Do not encode every business rule into the type system if runtime validation is simpler and clearer.

The goal is safer design, not maximum type sophistication.

---

## Avoid Type-Level Programming for Its Own Sake

Rust allows sophisticated compile-time modeling.

Do not turn ordinary application logic into:

```text
phantom state markers
nested generic states
associated type chains
marker traits
const-generic state machines
```

unless the guarantees materially improve correctness.

Complex types create complex compiler errors and onboarding cost.

---

## Documentation Comments

Use `///` and `//!` for public API documentation when useful.

Document:

- Non-obvious behavior.
- Panics.
- Errors.
- Safety requirements.
- Side effects.
- Examples where they help.

Do not narrate obvious signatures.

Bad:

```rust
/// Gets the weapon name.
fn name(&self) -> &str {
    &self.name
}
```

when the API is self-explanatory.

---

## Public API Docs

For reusable libraries, public items should generally have useful docs if the project expects them.

Avoid auto-generated prose such as:

```text
This function is responsible for...
This method provides functionality to...
```

Write concise technical documentation.

---

## Safety Docs

For `unsafe fn`, document the caller's safety obligations clearly.

Use a `# Safety` section where appropriate.

Example:

```rust
/// # Safety
///
/// `ptr` must point to `len` initialized bytes that remain valid
/// for the returned slice's lifetime.
```

Do not leave unsafe contracts implicit.

---

## Panic Docs

Document panics in public APIs when panic conditions are non-obvious and meaningful to callers.

Do not add a `# Panics` section to every trivial method mechanically.

---

## Error Docs

Document significant error conditions where callers need to understand them.

Do not restate every error variant line-by-line if the type already communicates them clearly.

---

## Examples

Use documentation examples when they materially help users understand a public API.

Keep examples:

- Small.
- Compilable when possible.
- Focused on the main use case.

Do not add giant tutorial examples to every item.

---

## Comments

Do not narrate the code.

Bad:

```rust
// Check if the weapon exists
if weapon.is_none() {
    // Return None if not found
    return None;
}
```

Good:

```rust
let weapon = weapon?;
```

or:

```rust
let Some(weapon) = weapon else {
    return None;
};
```

Write comments only when they explain something the code cannot.

Prefer **why**, not **what**.

---

## Avoid AI-Looking Comments

Avoid phrases like:

```text
This function is responsible for...
This method ensures...
The following implementation...
In order to...
This provides a robust and flexible solution...
```

Bad:

```rust
// This function is responsible for ensuring that the weapon
// data is properly validated before it is processed.
```

Better:

```rust
// Imported manifests may omit fields present in runtime assets.
```

---

## Avoid Decorative Section Comments

Do not add:

```rust
// ========================================
// INITIALIZATION
// ========================================
```

unless the repository already uses that convention.

Normal module structure should make these unnecessary.

---

## Avoid Placeholder Code

Do not leave:

```rust
todo!()
unimplemented!()
```

in finished production work unless the task explicitly calls for a stub or an unreachable future integration point.

Do not leave speculative TODO comments for unrequested features.

---

## `todo!()`

`todo!()` is useful during development.

Before finishing, replace it unless the intentionally incomplete behavior is explicitly documented and accepted.

---

## `unreachable!()`

Use `unreachable!()` only when the branch is truly impossible due to a proven invariant.

Do not use it to silence incomplete matching.

Prefer making the state impossible through types or handling it properly.

---

## `unreachable_unchecked`

Avoid `unreachable_unchecked`.

It invokes undefined behavior if the invariant is wrong.

Use only in highly performance-sensitive unsafe code with a rigorously proven invariant.

---

## Assertions

Use:

```rust
assert!
assert_eq!
debug_assert!
```

for programmer invariants and tests.

Do not use assertions for recoverable user input or environmental failures.

---

## Debug Assertions

Use `debug_assert!` for expensive or development-only invariant checks where release builds do not need them.

Do not rely on them for correctness-critical validation.

---

## API Validation

Validate untrusted or external input at boundaries.

Examples:

- CLI input.
- Network responses.
- Config files.
- Plugin inputs.
- Deserialized data.
- FFI boundaries.

Do not repeatedly validate trusted internal values.

---

## Serde Validation

Deserialization only validates shape and field types.

Domain invariants may still require explicit validation.

Do not assume a successfully deserialized struct is automatically semantically valid.

Consider validated constructors or post-deserialization checks where necessary.

---

## Public Field Deserialization

Be cautious if invalid values can be deserialized directly into types that are supposed to enforce invariants.

Sometimes a separate raw DTO and validated domain type is appropriate.

Do not duplicate models without a real boundary.

---

## Raw vs Domain Models

Use separate transport/config/domain types when they actually differ.

Do not automatically create:

```text
WeaponDto
WeaponRaw
WeaponModel
WeaponEntity
WeaponDomain
WeaponView
```

for one identical data shape.

Mappings should represent real transformations.

---

## Database Models

Likewise, do not create a separate domain struct just because ORM rows exist if the representations are truly identical and stable.

Separate them when coupling would otherwise harm the design.

---

## Avoid OOP Layering

Do not automatically structure Rust applications as:

```text
Controller
Service
Repository
Manager
Provider
Factory
```

unless those boundaries genuinely correspond to responsibilities.

Rust often benefits from:

- Plain functions.
- Modules.
- Data types.
- Traits at real boundaries.
- Explicit orchestration.

Do not import enterprise architecture mechanically.

---

## Avoid AI-Looking Architecture

Do not automatically introduce names such as:

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
Registry
Strategy
Adapter
Repository
Facade
Orchestrator
```

These names can be legitimate.

Use them only when they describe real architectural roles.

---

## Avoid Pass-Through Layers

Bad:

```rust
struct WeaponService {
    repository: WeaponRepository,
}

impl WeaponService {
    fn get_weapon(
        &self,
        id: WeaponId,
    ) -> Result<Option<Weapon>, Error> {
        self.repository.get_weapon(id)
    }
}
```

if the service adds no behavior.

Every layer should earn its existence.

---

## Avoid Helper Modules

Do not create generic:

```text
utils.rs
helpers.rs
common.rs
misc.rs
```

for unrelated functions.

Prefer cohesive domain-specific modules:

```text
weapon_paths.rs
manifest.rs
package_reader.rs
```

Small helpers can stay near their call sites.

---

## Avoid Premature Abstraction

Do not design for hypothetical future implementations.

If there is one parser, one backend, and one storage type, direct code may be best.

Do not add:

- Traits for one implementation.
- Generic factories.
- Registries with one entry.
- Strategy enums with one variant.
- Dynamic dispatch without consumers.
- Plugin systems without plugins.
- Configuration flags for fixed behavior.

Build what exists now.

---

## Avoid Premature DRY

Two similar blocks do not automatically require abstraction.

Rust abstractions can introduce:

- Lifetimes.
- Generics.
- Trait bounds.
- Type inference complexity.

Duplicate small code when the concepts are not truly the same.

Extract only when the shared abstraction is clear and stable.

---

## Avoid Fake Extensibility

Do not create an extensible framework around one feature.

Bad conceptual design:

```text
Parser trait
ParserRegistry
ParserFactory
DefaultParser
ParserContext
ParserStrategy
```

for one parser.

Prefer direct code until extension is real.

---

## Avoid Fake Zero-Cost Abstractions

"Zero-cost" in Rust does not mean "zero complexity."

Even if runtime overhead is optimized away, compile-time and maintenance overhead still matter.

Do not justify complex generics solely because they are zero-cost at runtime.

---

## Avoid Clone-Heavy Generated Code

Before accepting a change, scan for repeated:

```rust
.clone()
.to_owned()
.to_string()
```

Ask whether each allocation or copy is needed.

AI-generated Rust often clones excessively to avoid reasoning about ownership.

Remove unnecessary clones.

---

## Avoid `Arc<Mutex<_>>` Everywhere

Before accepting:

```rust
Arc<Mutex<T>>
```

ask:

- Does this value really need shared ownership?
- Does it really need mutation?
- Does it really cross threads/tasks?
- Could ownership move instead?
- Could message passing simplify this?

Do not use synchronization as an ownership shortcut.

---

## Avoid Overuse of Traits

Before adding a trait, ask:

- Are there multiple implementations?
- Is this a meaningful external boundary?
- Does generic reuse justify it?
- Does testing actually need substitution?
- Would an enum or function be simpler?

Do not create traits solely for "clean architecture."

---

## Avoid Overuse of Lifetimes

If lifetime annotations are spreading through many layers, reconsider ownership boundaries.

Sometimes returning owned values or moving ownership simplifies the entire design.

Do not fight for maximum borrowing at the cost of API usability.

---

## Avoid Overuse of `Box`

Do not box values simply to make type errors disappear.

Understand why indirection is needed.

Common valid reasons include:

- Recursive types.
- Trait objects.
- Large enum variants.
- Heap ownership boundaries.

---

## Avoid Overuse of `dyn Trait`

Dynamic dispatch is not a default architecture.

If variants are known, an enum may be simpler and more optimizable.

Use trait objects for actual open-ended runtime polymorphism.

---

## Avoid Overuse of `impl Trait`

`impl Trait` can simplify APIs, but hiding concrete types everywhere can make debugging and composition harder.

Use it where it improves ergonomics.

Do not use it reflexively.

---

## Avoid Overuse of Async

Do not convert a synchronous pipeline to async unless it needs async I/O or concurrency.

Async Rust increases:

- Trait complexity.
- Lifetime complexity.
- `Send` requirements.
- Runtime dependency.
- Testing complexity.

Keep synchronous code synchronous when practical.

---

## Avoid Overusing Channels

Channels are not a universal architecture pattern.

A direct function call is often better.

Use message passing where decoupling or concurrency actually benefits the system.

---

## Avoid Excessive Feature Flags

Do not create feature flags for every optional behavior.

Every feature combination creates another build and test state.

Prefer runtime configuration when compile-time exclusion is unnecessary.

---

## Avoid Build-Time Cleverness

Do not solve ordinary runtime configuration with:

- Build scripts.
- Environment macros.
- Compile-time generated modules.
- Heavy code generation.

unless it meaningfully improves the system.

---

## Avoid Excessive Type Aliases

Type aliases can clarify complex types.

Good:

```rust
type WeaponMap = HashMap<WeaponId, Weapon>;
```

when used widely.

Do not alias obvious types:

```rust
type WeaponNameString = String;
```

without semantic benefit.

---

## Avoid Giant Function Signatures

If a function has many related parameters, consider a struct.

Bad:

```rust
fn create_weapon(
    id: WeaponId,
    name: String,
    damage: u32,
    fire_rate: u32,
    magazine_size: u32,
    category: Category,
    description: String,
) -> Weapon
```

A construction struct may improve clarity.

Do not create option structs for every two-argument function.

---

## Borrowed Parameter Structs

Be cautious with parameter structs that contain many references.

They can complicate lifetimes.

Sometimes an owned config object is simpler.

Choose based on how long the data lives and whether ownership transfer matters.

---

## Public API Stability

For libraries, avoid exposing implementation details through public types.

Examples:

- Concrete async runtime types.
- Internal error crates.
- Internal collection types.
- Private dependency structs.

Do not hide everything behind abstractions either.

Expose what callers genuinely need.

---

## Versioned Data Formats

When reading persisted or networked formats, consider compatibility explicitly.

Do not silently change serialized field names or enum representations in mature systems.

Use serde attributes intentionally.

---

## Serde Attributes

Use:

```rust
#[serde(rename = "...")]
#[serde(default)]
#[serde(skip_serializing_if = "...")]
```

when format compatibility requires them.

Do not decorate every field with serde attributes unnecessarily.

---

## `serde(default)`

Do not use defaults to silently accept malformed required input.

A default should represent legitimate absence or compatibility behavior.

Otherwise require the field.

---

## Unknown Fields

Decide deliberately whether unknown fields should be accepted or rejected.

Strict config formats may benefit from:

```rust
#[serde(deny_unknown_fields)]
```

Flexible forward-compatible formats may not.

Do not choose one mechanically.

---

## Secrets

Do not derive `Debug` or serialize secrets accidentally.

Consider redacted wrappers or custom formatting for:

- API keys.
- Tokens.
- Passwords.
- Private keys.

Do not expose them in errors or logs.

---

## Tests for Errors

Test important error behavior.

Do not assert huge formatted error strings unless wording itself is part of the contract.

Prefer matching variants or key context.

---

## Golden Tests

Use golden/snapshot data for large parser or renderer outputs when appropriate.

Keep fixtures understandable.

Do not make tests depend on enormous opaque generated files unnecessarily.

---

## Fuzzing

Use fuzzing for parsers, binary formats, protocol handling, and unsafe boundaries where malformed input matters.

Do not add fuzzing infrastructure for trivial business logic.

---

## Miri

Use Miri when unsafe code, aliasing, or undefined behavior concerns justify it.

Do not make Miri mandatory for an ordinary safe-only application unless the project already uses it.

---

## Sanitizers

Use sanitizers for low-level FFI, unsafe code, or memory-sensitive systems where appropriate.

Do not add complex sanitizer CI for a simple safe Rust CLI without a reason.

---

## Cross-Platform Code

Respect platform differences in:

- Paths.
- Filesystem semantics.
- Signals.
- Process handling.
- Line endings.
- Sockets.

Do not hard-code Unix assumptions into cross-platform tools unless the project is Unix-only.

---

## Windows Paths

Do not assume path separators are `/`.

Use `Path` and `PathBuf`.

Do not manually concatenate filesystem paths with strings.

---

## Signals

Follow platform support.

Unix signal code should usually be isolated behind appropriate `cfg` boundaries.

Do not pretend Windows and Unix process control behave identically.

---

## Process Execution

Prefer:

```rust
Command::new("tool")
    .arg("--file")
    .arg(path)
```

over building one shell command string.

This avoids quoting and injection problems.

Use a shell only when shell syntax is actually required.

---

## Command Errors

Check exit status.

Do not treat process launch success as command success.

Example:

```rust
let status = Command::new("git")
    .arg("status")
    .status()?;

if !status.success() {
    return Err(CommandError::GitStatusFailed);
}
```

---

## Output Parsing

When consuming subprocess output, preserve stderr or exit status context when helpful.

Do not return:

```text
command failed
```

without indicating which command or why.

---

## Temporary Files

Use a mature temporary-file crate if the project already has one or cleanup guarantees matter.

Do not generate predictable filenames in shared temp directories.

---

## Resource Limits

For untrusted inputs, consider bounded allocation and parsing.

Rust prevents memory unsafety but not:

- Memory exhaustion.
- CPU exhaustion.
- Infinite loops.
- Huge decompression.
- Recursive stack exhaustion.

Do not equate memory safety with complete security.

---

## Recursive Algorithms

Be mindful of stack depth for attacker-controlled or deeply nested input.

Use iterative algorithms where recursion depth is unbounded.

Do not rewrite small bounded tree traversals iteratively without a reason.

---

## HashDoS and Collections

Standard `HashMap` has randomized hashing suitable for general untrusted keys.

Do not replace it with faster non-cryptographic hashers in exposed systems without considering denial-of-service risk.

Performance choices should reflect threat model.

---

## Determinism

For reproducible output, use deterministic ordering explicitly.

Do not rely on `HashMap` iteration order.

Use sorting or ordered collections where output stability matters.

---

## Floating Point

Do not derive `Eq` for floating-point-containing types.

Be cautious with direct equality when domain semantics require tolerance.

Do not introduce approximate comparison everywhere if exact bitwise/value equality is actually required.

---

## NaN

Remember floating-point NaN breaks total ordering assumptions.

Use wrappers such as ordered float types only when ordered floating-point keys are actually needed.

---

## Parsing Numbers

Use checked parsing:

```rust
let damage: u32 = value.parse()?;
```

Handle range errors naturally.

Do not parse into a larger type and blindly cast down.

---

## Units

Use clear names or newtypes when units are easy to confuse.

Good:

```rust
timeout_ms
distance_meters
```

or domain newtypes where safety matters.

Do not wrap every numeric value in a newtype without a practical risk of confusion.

---

## Time

Use established time crates already present in the project.

Common choices include:

```text
std::time
time
chrono
```

Do not add multiple time libraries without a reason.

Use `Instant` for durations and elapsed time.

Use wall-clock datetime types for actual timestamps.

---

## `Instant`

Use:

```rust
Instant::now()
```

for measuring elapsed time.

Do not use wall-clock time for duration measurement when clock adjustments could matter.

---

## Randomness

Use an established randomness crate when needed.

For security-sensitive randomness, use a cryptographically secure source.

Do not implement custom PRNGs for secrets.

---

## IDs

Use domain-specific ID newtypes when accidental interchange would be dangerous or APIs benefit from stronger typing.

Do not wrap every integer ID automatically in small internal scripts.

---

## Formatting

Use rustfmt.

Prefer code that reads naturally after standard formatting.

Do not manually introduce exotic layout to make types look aligned or symmetrical.

---

## Line Length

Let rustfmt handle normal wrapping.

Do not contort expressions solely to remain under an arbitrary line length when the formatter and repository conventions permit otherwise.

---

## Imports

Keep imports clean.

Let rustfmt and the project's conventions guide grouping.

Remove unused imports.

Do not create broad glob imports in production modules unless the module is explicitly intended as a prelude.

---

## Glob Imports

Avoid:

```rust
use module::*;
```

in normal production code when it obscures origin.

Glob imports are often fine in:

- Tests.
- Prelude modules.
- Closely scoped enum variants.

Use judgment.

---

## Enum Variant Imports

This can be fine:

```rust
use State::{Failed, Loading, Ready};
```

when it makes a local state machine easier to read.

Do not import variants globally if names become ambiguous.

---

## Aliased Imports

Use aliases when names conflict or the alternate name improves clarity.

Do not rename imports purely to make them shorter.

---

## Before Finishing

Review the change and remove:

- Unnecessary traits.
- Unnecessary generics.
- Unnecessary lifetimes.
- Unnecessary macros.
- Unnecessary async.
- Excessive cloning.
- Excessive boxing.
- Excessive `Arc`.
- Excessive locking.
- `unwrap()` in recoverable paths.
- `expect()` used as fake error handling.
- Generic AI-style names.
- Pass-through service layers.
- Fake interfaces.
- Giant error enums.
- Duplicate validation.
- Excessive logging.
- Debug `println!()` and `dbg!()`.
- Placeholder `todo!()`.
- Unnecessary derives.
- Dead code.
- Unused imports.
- Speculative feature flags.
- Speculative extensibility.
- Unrelated refactors.

Then run the repository's existing checks where available.

Typical Rust checks may include:

```bash
cargo fmt --check
cargo check
cargo clippy --all-targets --all-features
cargo test
```

For workspaces, the repository may instead use:

```bash
cargo test --workspace
cargo clippy --workspace --all-targets
```

Do not assume these exact commands exist or that all features can be enabled together.

Inspect:

```text
Cargo.toml
CI configuration
justfile
Makefile
task scripts
repository documentation
```

and use the project's established workflow.

The final code should look like it naturally belongs in the repository rather than like a standalone AI-generated Rust solution.

It should feel like Rust written by an experienced maintainer: explicit about ownership, restrained with abstraction, deliberate about errors, comfortable with enums and borrowing, and willing to use straightforward code instead of forcing every problem through traits, generics, async, or synchronization.