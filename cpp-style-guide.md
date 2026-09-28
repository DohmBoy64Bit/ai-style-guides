# C++ Style Guide

Write C++ as an experienced professional C++ developer maintaining a real production codebase.

The goal is clean, idiomatic, maintainable, safe C++—not code that looks generated, over-engineered, excessively generic, or written as a tutorial.

The most important rule:

> Do not optimize for demonstrating C++ features or design patterns. Optimize for producing the smallest idiomatic production-quality change that an experienced maintainer would reasonably write.

## General Principles

Prefer:

- Existing project conventions over personal preferences.
- RAII over manual resource management.
- Value semantics where practical.
- Clear ownership.
- Standard-library facilities over custom equivalents.
- Concrete types until abstraction is justified.
- Composition over inheritance.
- Small focused changes.
- Explicit domain terminology.
- Simple control flow.
- `const` correctness.
- Strong types when they prevent real mistakes.
- Modern C++ features when they improve clarity.
- Compile-time guarantees where they reduce runtime complexity.
- Direct implementation over speculative extensibility.

Avoid:

- Manual `new` and `delete`.
- Raw owning pointers.
- Excessive inheritance.
- Interfaces for one implementation.
- Unnecessary factories.
- Singleton-heavy architecture.
- Template metaprogramming without a real need.
- Macro-heavy code.
- Shared ownership by default.
- Exception swallowing.
- Undefined-behavior-prone shortcuts.
- C-style code in modern C++ projects.

Do not refactor unrelated code unless required by the task.

---

## Match the Existing Codebase

Before writing code, inspect nearby files and follow the repository's established conventions for:

- C++ standard version.
- Naming.
- Header/source organization.
- Namespace usage.
- Error handling.
- Exceptions.
- Smart pointer conventions.
- Ownership semantics.
- Logging.
- Build system.
- Testing.
- Formatting.
- Static analysis.
- Platform abstractions.
- Compiler support.
- Warning policy.
- Dependency style.

Check relevant files such as:

```text
CMakeLists.txt
CMakePresets.json
meson.build
BUILD
WORKSPACE
.vcxproj
.clang-format
.clang-tidy
conanfile.py
conanfile.txt
vcpkg.json
```

Consistency with the repository is more important than imposing another C++ style.

Do not migrate:

- C++ standards.
- Build systems.
- Error handling.
- Smart pointer conventions.
- Naming conventions.
- Header structure.

as part of an unrelated task.

---

## C++ Standard

Determine the project's supported C++ standard before using newer features.

Possible targets include:

```text
C++17
C++20
C++23
```

Do not assume the project supports the newest standard.

Check build configuration for settings such as:

```cmake
set(CMAKE_CXX_STANDARD 20)
```

or:

```cmake
target_compile_features(target PRIVATE cxx_std_20)
```

Use features supported by the declared toolchain.

Do not introduce `std::expected`, ranges, concepts, or other newer APIs without confirming availability.

---

## Naming

Follow the project's existing naming convention first.

Common conventions may use:

- `PascalCase` for types.
- `camelCase` or `snake_case` for functions and variables.
- `_member` or `member_` for private fields.
- `kConstantName` or `UPPER_SNAKE_CASE` for constants.

Do not introduce a second naming convention.

Choose domain-specific names.

Good:

```cpp
weaponDefinition
assetPath
packageName
loadResult

loadPackage()
resolveAsset()
parseWeaponData()
```

Avoid vague AI-style names:

```cpp
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

```cpp
auto data = getData();
auto result = processData(data);
```

Better:

```cpp
auto weapon = loadWeaponDefinition();
auto stats = parseWeaponStats(weapon);
```

Do not make names unnecessarily verbose.

Avoid:

```cpp
successfullyParsedWeaponConfigurationResult
```

when:

```cpp
weaponConfig
```

is clear.

---

## Functions First

Prefer free functions for stateless operations when they naturally fit.

Good:

```cpp
WeaponStats parseWeaponStats(const WeaponData& data)
{
    return {
        .damage = data.damage,
        .fireRate = data.fireRate,
    };
}
```

Do not automatically create:

```cpp
class WeaponStatsParser
{
public:
    WeaponStats parse(const WeaponData& data) const;
};
```

unless the parser actually owns:

- State.
- Configuration.
- Dependencies.
- Lifecycle.
- Multiple related operations.

Namespaces already provide organization.

---

## Classes

Use classes when state and behavior genuinely belong together.

Good:

```cpp
class PackageCache
{
public:
    const Package* find(std::string_view path) const;
    void insert(std::string path, Package package);

private:
    std::unordered_map<std::string, Package> packages_;
};
```

Do not create classes merely because a domain concept is a noun.

Avoid structures like:

```text
WeaponManager
WeaponService
WeaponProvider
WeaponHandler
WeaponProcessor
WeaponCoordinator
```

unless they represent genuinely distinct responsibilities.

---

## Struct vs Class

Use `struct` for simple data-oriented types where public fields are appropriate.

Example:

```cpp
struct WeaponStats
{
    float damage;
    float fireRate;
    std::uint32_t magazineSize;
};
```

Use `class` when invariants, encapsulation, or non-trivial behavior justify private state.

Do not write getters and setters around every data field just because the type is a class.

---

## Value Semantics

Prefer values when ownership is simple and copying or moving is appropriate.

Good:

```cpp
Weapon loadWeapon(const std::filesystem::path& path);
```

rather than returning heap ownership unnecessarily.

Modern C++ is good at moving values efficiently.

Do not heap-allocate objects by default.

---

## RAII

Use RAII for resource management.

Resources include:

- Memory.
- Files.
- Locks.
- Sockets.
- Handles.
- Database transactions.
- Graphics resources.
- Temporary state.

Good:

```cpp
std::ifstream file(path);

if (!file)
    throw FileError(path);
```

The file closes automatically.

Do not write manual cleanup paths unless RAII cannot express the resource lifetime.

---

## Avoid Manual `new` and `delete`

Do not write:

```cpp
Weapon* weapon = new Weapon();
// ...
delete weapon;
```

Prefer:

```cpp
Weapon weapon;
```

or, when dynamic lifetime is actually needed:

```cpp
auto weapon = std::make_unique<Weapon>();
```

Manual `new`/`delete` should be rare in modern production C++.

---

## Ownership

Make ownership obvious.

Prefer:

- Values for owned local objects.
- `std::unique_ptr` for exclusive heap ownership.
- `std::shared_ptr` only for genuine shared lifetime.
- Raw pointers and references for non-owning access.

Do not use one pointer type for every relationship.

---

## `std::unique_ptr`

Use `std::unique_ptr<T>` when:

- Heap allocation is necessary.
- Exactly one owner exists.
- Ownership transfers explicitly.

Example:

```cpp
std::unique_ptr<Renderer> createRenderer();
```

Do not use `unique_ptr` when a normal value member works equally well.

Bad:

```cpp
class Weapon
{
    std::unique_ptr<std::string> name_;
};
```

when:

```cpp
std::string name_;
```

is sufficient.

---

## `std::shared_ptr`

Do not use `shared_ptr` by default.

Use it only when several objects genuinely share ownership of the same lifetime.

Bad:

```cpp
std::shared_ptr<Weapon> weapon =
    std::make_shared<Weapon>();
```

merely because pointers are convenient.

Ask:

- Who owns this object?
- Can ownership remain unique?
- Can callers borrow it?
- Can it be stored by value?

Shared ownership introduces:

- Reference counting.
- Hidden lifetime coupling.
- Atomic overhead in many implementations.
- Cycles.
- More difficult reasoning.

---

## `std::weak_ptr`

Use `weak_ptr` to observe an object managed by `shared_ptr` without extending its lifetime.

Most often this is useful for breaking ownership cycles.

Do not introduce `weak_ptr` unless shared ownership already exists for a good reason.

---

## Raw Pointers

Raw pointers are appropriate for non-owning nullable access.

Example:

```cpp
const Weapon* findWeapon(WeaponId id);
```

if absence is represented by `nullptr`.

Do not use raw pointers to represent ownership.

If a raw pointer owns memory, the ownership model is unclear.

---

## References

Use references when an argument:

- Must exist.
- Is non-owning.
- Should not be copied.

Example:

```cpp
void processWeapon(const Weapon& weapon);
```

Prefer pointers when nullability is meaningful.

Do not use pointers simply out of habit.

---

## `const`

Use `const` to communicate intent.

Good:

```cpp
void render(const Weapon& weapon);

std::string_view name() const;
```

Do not add `const` mechanically where it adds no useful information.

For local values that never change, `const` can improve clarity:

```cpp
const auto count = weapons.size();
```

Follow project convention.

---

## Const Correctness

A `const` member function should not modify observable object state.

Do not use:

```cpp
mutable
```

merely to work around poor API design.

Reasonable uses of `mutable` include:

- Caches.
- Lazy synchronization primitives.
- Internal instrumentation.

Use it deliberately.

---

## Pass by Value vs Reference

Use:

```cpp
const T&
```

for large read-only objects when borrowing is appropriate.

Use values for:

- Small trivial types.
- Objects being consumed.
- APIs that benefit from move semantics.

Example:

```cpp
class Weapon
{
public:
    explicit Weapon(std::string name)
        : name_(std::move(name))
    {
    }

private:
    std::string name_;
};
```

This "take by value, then move" pattern can be useful when the function needs ownership.

Do not use it mechanically for every parameter.

---

## Small Types

Pass small trivially copied values by value.

Good:

```cpp
void setDamage(float damage);
void selectWeapon(WeaponId id);
```

Avoid:

```cpp
void setDamage(const float& damage);
```

References to tiny primitive types add no benefit.

---

## Move Semantics

Use move semantics when transferring ownership.

Example:

```cpp
weapons_.push_back(std::move(weapon));
```

when the original value will no longer be used.

Do not sprinkle `std::move` everywhere.

An unnecessary move can:

- Prevent copy elision.
- Leave objects in unspecified states unnecessarily.
- Obscure intent.

---

## Return Values

Return values normally.

Do not write:

```cpp
Weapon result;
populateWeapon(result);
return result;
```

when:

```cpp
return parseWeapon(data);
```

or direct construction is clearer.

Modern compilers optimize return values effectively.

---

## Do Not `std::move` Local Return Values

Avoid:

```cpp
return std::move(result);
```

for ordinary local variables.

This can interfere with NRVO.

Prefer:

```cpp
return result;
```

unless there is a specific reason.

---

## Copying

Avoid unnecessary copies, but do not turn the code into unreadable reference gymnastics to eliminate trivial copies.

Ask whether a copy is:

- Expensive.
- Repeated.
- On a hot path.
- Semantically unnecessary.

Optimize meaningful costs.

---

## `std::string`

Use `std::string` for owned text.

Do not manage C strings manually in normal modern C++ code unless interfacing with C APIs.

Prefer:

```cpp
std::string name;
```

over:

```cpp
char* name;
```

for owned dynamic strings.

---

## `std::string_view`

Use `std::string_view` for non-owning string input when lifetime requirements are straightforward.

Example:

```cpp
bool isWeaponCategory(std::string_view value);
```

Do not store a `string_view` unless the referenced storage is guaranteed to outlive it.

This is dangerous:

```cpp
struct Weapon
{
    std::string_view name;
};
```

if the view may outlive its source.

Use owned `std::string` for persistent state unless borrowing lifetime is explicit and safe.

---

## Avoid Dangling `string_view`

Bad:

```cpp
std::string_view makeName()
{
    std::string name = "Rifle";
    return name;
}
```

The returned view dangles.

Do not use views merely to reduce allocations without understanding lifetime.

---

## C Strings

When interacting with APIs requiring null-terminated strings:

```cpp
string.c_str()
```

is appropriate.

Do not retain the pointer after the string may reallocate or be destroyed.

---

## `std::span`

Use `std::span` for non-owning contiguous sequences when supported by the project's C++ version.

Good:

```cpp
void processBytes(std::span<const std::byte> data);
```

This can be more flexible than:

```cpp
void processBytes(const std::vector<std::byte>& data);
```

when the function does not care about the owning container.

Do not use `span` if the project targets pre-C++20 without an established equivalent.

---

## Arrays

Prefer:

```cpp
std::array<T, N>
```

for fixed-size owned arrays.

Prefer:

```cpp
std::vector<T>
```

for dynamically sized arrays.

Avoid raw arrays unless needed for interoperability or low-level code.

---

## `std::vector`

Use `std::vector` as the default dynamic sequence.

Do not reach for:

```cpp
std::list
```

merely because insertions occur.

`std::vector` often performs better due to locality even when theoretical complexity suggests otherwise.

Choose containers based on actual usage.

---

## `std::deque`

Use `deque` when efficient growth at both ends or stable storage characteristics matter.

Do not use it as a generic replacement for vector.

---

## `std::list`

Use linked lists only when their specific characteristics are genuinely useful.

They often have poor cache locality and allocation overhead.

Do not choose `std::list` because "insertion is O(1)" without considering traversal cost.

---

## Maps

Choose:

```cpp
std::unordered_map
```

for hash lookup when ordering is unnecessary.

Use:

```cpp
std::map
```

when sorted ordering or stable tree semantics matter.

Do not choose one based solely on theoretical complexity.

Consider:

- Dataset size.
- Iteration behavior.
- Memory.
- Ordering.
- Key type.

---

## Sets

Use sets when uniqueness or membership is the purpose.

Do not use them where a vector plus occasional search is simpler and adequate for a tiny dataset.

---

## Reserve Capacity

Use:

```cpp
vector.reserve(count);
```

when size is known or repeated reallocations are meaningful.

Do not reserve speculative capacities everywhere.

Avoid premature micro-optimization.

---

## `emplace_back`

Do not prefer `emplace_back` automatically.

This is clear:

```cpp
weapons.push_back(Weapon{id, name});
```

and can be as good as:

```cpp
weapons.emplace_back(id, name);
```

Use whichever best communicates construction.

Do not treat `emplace` as universally superior.

---

## Range-Based For Loops

Prefer:

```cpp
for (const auto& weapon : weapons)
{
    // ...
}
```

for ordinary iteration.

Use:

```cpp
auto&
```

when modifying elements.

Use values when copying is intended.

Avoid index-based loops unless the index is genuinely needed.

---

## Index Loops

Bad:

```cpp
for (std::size_t i = 0; i < weapons.size(); ++i)
{
    process(weapons[i]);
}
```

when:

```cpp
for (const auto& weapon : weapons)
{
    process(weapon);
}
```

is enough.

Use indices when:

- Position matters.
- Multiple containers are indexed together.
- Random access is integral to the algorithm.

---

## Iterators

Use iterators when library algorithms or container APIs naturally require them.

Do not expose raw iterator machinery when range-based code is clearer.

---

## Algorithms

Use standard algorithms when they improve intent.

Good:

```cpp
const auto it = std::find_if(
    weapons.begin(),
    weapons.end(),
    [&](const Weapon& weapon)
    {
        return weapon.id == id;
    });
```

But do not force every loop into `<algorithm>`.

A simple loop may be clearer when:

- Multiple operations happen.
- Early exits and side effects matter.
- State is updated.
- Error handling differs.

---

## Ranges

If the project uses C++20/23 ranges, use them where they genuinely simplify code.

Do not convert ordinary readable loops into deeply nested range pipelines merely because ranges are modern.

Avoid code that requires extensive knowledge of views to understand a straightforward transformation.

---

## Range Pipelines

Bad when overdone:

```cpp
auto result =
    values
    | std::views::filter(...)
    | std::views::transform(...)
    | std::views::drop(...)
    | std::views::take(...)
    | std::views::reverse;
```

if a few explicit statements would be easier to maintain.

Readable intent matters more than density.

---

## Lambdas

Use lambdas for small local behavior.

Good:

```cpp
std::sort(
    weapons.begin(),
    weapons.end(),
    [](const Weapon& a, const Weapon& b)
    {
        return a.name < b.name;
    });
```

Do not place large business logic inside giant lambdas.

Extract a named function when the operation becomes substantial.

---

## Lambda Captures

Use the narrowest reasonable capture.

Prefer:

```cpp
[&weaponId]
```

or:

```cpp
[this]
```

over:

```cpp
[&]
```

or:

```cpp
[=]
```

when broad capture obscures dependencies.

Follow local project style.

---

## Avoid Accidental Lifetime Captures

Be especially careful when lambdas outlive their scope.

Do not capture local references into:

- Threads.
- Callbacks.
- Async operations.
- Deferred tasks.

unless lifetime is guaranteed.

Prefer value captures when the lambda must own data.

---

## `auto`

Use `auto` when:

- The type is obvious.
- The exact iterator type is noisy.
- Generic code benefits from deduction.
- Structured bindings improve readability.

Good:

```cpp
auto weapon = loadWeapon(path);
auto it = weapons.find(id);
```

Use explicit types when they communicate useful domain information.

Bad:

```cpp
auto value = calculate();
```

if the result type is important but not obvious.

Do not mechanically use or avoid `auto`.

---

## `decltype`

Use `decltype` when type relationships genuinely require it.

Do not use it for ordinary variables merely to demonstrate template knowledge.

---

## Type Aliases

Use:

```cpp
using WeaponId = std::uint64_t;
```

when the alias improves readability.

Be aware that this does not create a distinct type.

If accidental interchange matters, use a strong wrapper type.

Do not alias obvious types without semantic benefit.

---

## Strong Types

Use strong types when similar primitive values are easy to confuse.

Example:

```cpp
struct WeaponId
{
    std::uint64_t value;
};

struct PlayerId
{
    std::uint64_t value;
};
```

Do not wrap every number and string automatically.

The extra type should prevent an actual class of mistake.

---

## Enums

Use enums for closed sets of meaningful values.

Prefer:

```cpp
enum class WeaponCategory
{
    Rifle,
    Smg,
    Shotgun,
    Sniper
};
```

over unscoped enums in modern C++.

`enum class` avoids implicit integer conversions and namespace pollution.

---

## Avoid Boolean State Machines

Bad:

```cpp
struct State
{
    bool loading;
    bool ready;
    bool failed;
};
```

Better:

```cpp
enum class State
{
    Loading,
    Ready,
    Failed
};
```

Use booleans for genuine yes/no properties.

---

## `std::optional`

Use `std::optional<T>` when absence is expected and not itself an error.

Good:

```cpp
std::optional<Weapon> findWeapon(WeaponId id);
```

Do not use magic sentinel values such as:

```cpp
-1
0
""
```

when optional expresses the contract better.

---

## Returning Optional References

`std::optional<T&>` is not supported by standard `std::optional`.

For optional non-owning access, common choices include:

```cpp
const Weapon*
```

or:

```cpp
std::optional<std::reference_wrapper<const Weapon>>
```

Use the simplest established project pattern.

Do not build complicated wrappers unnecessarily.

---

## `std::variant`

Use `std::variant` when a value may be one of several closed types.

Example:

```cpp
using LoadResult =
    std::variant<Weapon, NotFound, ParseError>;
```

But do not replace every class hierarchy with a variant mechanically.

Use variants when:

- The alternatives are known.
- Exhaustive handling is useful.
- Value semantics are beneficial.

---

## `std::visit`

Use `std::visit` for variant handling when appropriate.

Do not build elaborate overload helpers unless they improve repeated variant code.

For one simple variant operation, straightforward code may be clearer.

---

## `std::expected`

If the project uses C++23 or an established expected implementation, use it for value-or-error APIs when it fits the architecture.

Do not introduce it into an exception-based codebase casually.

Consistency in error handling matters.

---

## Exceptions

Follow the project's error-handling model.

If exceptions are used, use them for exceptional failures.

Do not use exceptions for ordinary branching.

Do not catch exceptions simply to log and rethrow without adding value.

Bad:

```cpp
try
{
    return loadPackage(path);
}
catch (const std::exception& e)
{
    logger.error(e.what());
    throw;
}
```

if a higher layer already logs failures.

---

## Catch by Reference

Catch exceptions by reference:

```cpp
catch (const std::exception& error)
```

not by value:

```cpp
catch (std::exception error)
```

Catching by value can slice derived exceptions.

---

## Preserve Exception Information

When wrapping errors, preserve useful context.

If the project uses nested exceptions:

```cpp
std::throw_with_nested(
    PackageLoadError(path));
```

or another established mechanism, follow it.

Do not convert every exception into a vague generic message.

---

## Do Not Catch Everything

Avoid:

```cpp
catch (...)
{
}
```

except at true system boundaries where recovery or controlled shutdown is required.

Never silently swallow exceptions without a deliberate reason.

---

## Exception Safety

When modifying state across operations that may throw, consider exception safety.

Prefer changes that naturally provide:

- Basic guarantee.
- Strong guarantee.
- No-throw behavior where appropriate.

RAII and value construction often make exception safety easier.

Do not manually roll back state when simple construction-before-commit can solve it.

---

## `noexcept`

Use `noexcept` when the function is genuinely guaranteed not to throw and the guarantee is meaningful.

Good candidates often include:

- Destructors.
- Move operations.
- Simple accessors.

Do not mark functions `noexcept` merely for performance.

If an exception escapes a `noexcept` function, the process terminates.

---

## Destructors

Destructors should not throw.

Use RAII so cleanup occurs naturally.

Do not place substantial business logic in destructors.

Destruction should primarily release resources.

---

## Rule of Zero

Prefer the Rule of Zero.

If members manage themselves correctly, do not manually implement:

```cpp
destructor
copy constructor
copy assignment
move constructor
move assignment
```

Example:

```cpp
class Weapon
{
    std::string name_;
    std::vector<Attachment> attachments_;
};
```

usually needs none of those manually.

---

## Rule of Five

If a type directly manages a resource and must define special member functions, consider the full ownership semantics.

However, prefer wrapping that resource in an RAII type so Rule of Zero becomes possible.

Do not write custom copy/move operations casually.

---

## Deleted Operations

Delete copy or move operations when the type semantically cannot support them.

Example:

```cpp
class Socket
{
public:
    Socket(const Socket&) = delete;
    Socket& operator=(const Socket&) = delete;
};
```

Do not delete operations merely to look restrictive.

Let value types remain naturally copyable when copying makes sense.

---

## Constructors

Keep constructors focused on establishing a valid object.

Avoid constructors that:

- Perform network calls.
- Start threads unexpectedly.
- Read large files.
- Register global state.
- Perform expensive unrelated operations.

Use explicit factories or initialization operations if creation can fail substantially.

---

## Constructor Validation

A constructor should establish type invariants.

Do not allow objects to exist in invalid states if that can be prevented cleanly.

However, do not put every possible business rule inside constructors when validation belongs at an external boundary.

---

## Member Initialization Lists

Use initialization lists:

```cpp
Weapon::Weapon(std::string name)
    : name_(std::move(name))
{
}
```

Do not default-construct members only to assign them inside the constructor body.

---

## Initialization Order

Remember members initialize in declaration order, not initializer-list order.

Keep initializer lists aligned with member declaration order.

Do not rely on textual initializer order to create dependencies.

---

## Default Member Initializers

Use them for natural defaults:

```cpp
bool enabled_ = true;
std::uint32_t retries_ = 3;
```

Do not duplicate the same initialization across every constructor.

---

## Delegating Constructors

Use delegating constructors when several constructors genuinely share initialization.

Do not create many overloads merely to demonstrate constructor delegation.

---

## `explicit`

Use `explicit` on single-argument constructors unless implicit conversion is intentionally useful.

Good:

```cpp
explicit WeaponId(std::uint64_t value);
```

This prevents surprising conversions.

Do not mark every multi-argument constructor `explicit` unless the supported standard and API semantics warrant it.

---

## Conversion Operators

Avoid implicit conversion operators unless conversion is unsurprising and safe.

Prefer explicit conversions for domain types.

Implicit conversions can create ambiguous overloads and subtle bugs.

---

## Getters

Do not write Java-style getters automatically.

Avoid:

```cpp
const std::string& getName() const;
```

if the project convention prefers:

```cpp
std::string_view name() const;
```

or direct public fields for simple structs.

Follow repository style.

---

## Setters

Do not create generic setters merely to mutate fields.

Bad:

```cpp
void setName(std::string name);
void setDamage(float damage);
void setCategory(Category category);
```

when domain-specific operations better express behavior.

A setter is appropriate when direct mutation is truly the intended API.

---

## Encapsulation

Encapsulation is useful for protecting invariants.

Do not hide every field behind boilerplate accessors when the type is simply data.

Use the least ceremony appropriate to the type.

---

## Inheritance

Prefer composition over inheritance.

Use inheritance when:

- There is a genuine substitutable hierarchy.
- Runtime polymorphism is needed.
- The base abstraction has semantic meaning.

Do not introduce inheritance merely to share a few lines of code.

---

## Interfaces

In C++, an interface is typically an abstract base class.

Use one when there are:

- Multiple implementations.
- A stable architectural boundary.
- Plugin implementations.
- Runtime polymorphism.

Avoid:

```cpp
class IWeaponParser
{
public:
    virtual ~IWeaponParser() = default;
    virtual Weapon parse(const WeaponData&) = 0;
};
```

with exactly one implementation and no realistic need for substitution.

A normal function may be better.

---

## Virtual Destructors

Base classes intended for polymorphic deletion need virtual destructors.

Example:

```cpp
class Plugin
{
public:
    virtual ~Plugin() = default;
};
```

Do not add virtual destructors to classes that are not polymorphic bases without a reason.

---

## `override`

Use `override` on overridden virtual methods.

Good:

```cpp
void update() override;
```

Do not redundantly repeat `virtual` if project style does not require it:

```cpp
virtual void update() override;
```

is legal but often unnecessary.

---

## `final`

Use `final` when inheritance or overriding should intentionally stop.

Do not mark every class `final` mechanically.

Sometimes extensibility is part of the type's design.

---

## Multiple Inheritance

Avoid complex multiple implementation inheritance.

Reasonable cases include:

- Interface-like abstract bases.
- Mixins with clear semantics.

Do not build deep multiple-inheritance hierarchies.

Composition is usually easier to reason about.

---

## Dynamic Casts

Frequent `dynamic_cast` can indicate poor hierarchy design.

Use it when runtime type inspection is genuinely required.

Do not design an API around repeated downcasting.

Consider virtual methods, variants, or better ownership boundaries.

---

## RTTI

Do not disable or depend heavily on RTTI without understanding project requirements.

Follow the existing toolchain configuration.

---

## Templates

Use templates when code genuinely applies to multiple types.

Good:

```cpp
template <typename T>
const T* findById(
    const std::vector<T>& items,
    typename T::Id id);
```

only if this abstraction is truly shared.

Do not genericize domain-specific logic solely to avoid duplication.

---

## Avoid Template Overengineering

Do not create complicated template frameworks for one or two concrete cases.

Templates increase:

- Compile times.
- Error complexity.
- Header exposure.
- Binary size through instantiation.
- API complexity.

Use concrete code until genericity proves useful.

---

## Concepts

Use concepts in C++20+ when they materially clarify template requirements.

Good:

```cpp
template <typename T>
concept WeaponLike =
    requires(const T& value)
    {
        value.id();
        value.name();
    };
```

Do not define concepts for trivial single-use templates.

Do not create concept hierarchies merely to make APIs look sophisticated.

---

## `requires`

Use `requires` clauses when constraints improve diagnostics or API semantics.

Do not add complex constraint expressions where a concrete function would be clearer.

---

## SFINAE

Prefer concepts and modern constraints over complex `enable_if` machinery when the project's C++ standard supports them.

Do not rewrite working SFINAE code merely for style unless the task is modernization.

---

## Template Metaprogramming

Use template metaprogramming only when compile-time behavior genuinely requires it.

Avoid elaborate:

```text
type traits
recursive templates
detection idioms
tag dispatch
constexpr metaprograms
```

for ordinary application logic.

Modern C++ should not resemble a compile-time puzzle without a strong reason.

---

## `constexpr`

Use `constexpr` when values or functions genuinely benefit from compile-time evaluation.

Good:

```cpp
constexpr std::size_t MaxWeaponSlots = 8;
```

Do not mark every function `constexpr` merely because it can technically be evaluated at compile time.

---

## `consteval`

Use `consteval` only when evaluation must happen at compile time.

It significantly constrains callers.

Do not use it as a fashionable replacement for `constexpr`.

---

## `constinit`

Use `constinit` when static initialization order and compile-time initialization guarantees matter.

Do not introduce it without understanding the static initialization problem being solved.

---

## Macros

Avoid macros when language features can solve the problem.

Bad:

```cpp
#define MAX_WEAPONS 100
```

Prefer:

```cpp
constexpr std::size_t MaxWeapons = 100;
```

Avoid function-like macros when:

- Functions.
- Templates.
- `constexpr`.
- Lambdas.

can express the behavior safely.

---

## Valid Macro Uses

Macros can be appropriate for:

- Conditional compilation.
- Compiler-specific attributes.
- Generated code hooks.
- Logging macros.
- Test frameworks.
- Repetitive declarations impossible to express otherwise.

Keep them small and well-scoped.

---

## Macro Safety

When macros are necessary:

- Parenthesize arguments where appropriate.
- Avoid evaluating arguments multiple times.
- Use distinctive names.
- Avoid hidden control flow.
- Prefer statement-safe patterns if required.

Do not create macro DSLs casually.

---

## Preprocessor Conditionals

Use:

```cpp
#ifdef _WIN32
```

or feature macros when platform-specific code is necessary.

Avoid scattering platform checks throughout generic logic.

Prefer platform-specific implementation files or small abstraction boundaries.

---

## Headers

Headers should expose what callers need.

Avoid including heavy headers unnecessarily.

Prefer forward declarations where they safely reduce coupling and compile time.

Do not obsess over forward declarations when they complicate code or conflict with templates/value members.

---

## Include What You Use

Do not rely on transitive includes.

If a file directly uses:

```cpp
std::vector
```

include:

```cpp
#include <vector>
```

Do not assume another header happens to include it.

This improves build stability.

---

## Header Guards

Use the project's convention:

```cpp
#pragma once
```

or classic include guards.

Do not convert between them as unrelated cleanup.

---

## Header-Only Code

Keep implementation in headers only when required by:

- Templates.
- Inline APIs.
- Header-only library architecture.

Do not place large non-template implementation bodies in headers without a reason.

This can increase compile times and coupling.

---

## Forward Declarations

Use forward declarations for pointer/reference dependencies when they meaningfully reduce includes.

Do not forward-declare standard library types.

Do not create fragile declarations simply to avoid a harmless include.

---

## Namespace Usage

Place code in meaningful namespaces.

Avoid:

```cpp
using namespace std;
```

especially in headers.

In source files, broad namespace imports are still usually unnecessary.

Prefer explicit qualification or narrow `using` declarations.

---

## Anonymous Namespaces

Use anonymous namespaces in `.cpp` files for internal linkage.

Example:

```cpp
namespace
{

constexpr int DefaultRetryCount = 3;

}
```

Do not put anonymous namespaces in headers.

---

## Static Functions

File-local `static` functions are valid, but anonymous namespaces are often clearer in modern C++.

Follow project conventions.

---

## Global State

Avoid mutable global state.

Do not solve shared state with:

```cpp
static GlobalManager instance;
```

unless process-wide lifetime is genuinely required.

Prefer explicit ownership and dependency flow.

---

## Singletons

Do not introduce singleton patterns by default.

Bad:

```cpp
class AssetManager
{
public:
    static AssetManager& instance();
};
```

solely to avoid passing dependencies.

Singletons create:

- Hidden dependencies.
- Global lifetime coupling.
- Test difficulties.
- Initialization order concerns.

Use them only when truly process-global semantics are justified.

---

## Function-Local Statics

Function-local statics are useful for immutable or lazily initialized process-wide objects.

Example:

```cpp
const Regex& assetPathRegex()
{
    static const Regex regex(...);
    return regex;
}
```

Do not use them as hidden mutable service locators.

---

## Static Initialization Order

Avoid cross-translation-unit initialization dependencies.

Prefer:

- Function-local statics.
- Explicit initialization.
- Constant initialization.

Do not rely on global constructor order.

---

## Error Codes

If the project uses error codes instead of exceptions, follow that architecture.

Do not mix styles casually.

Use structured error enums or existing status types where appropriate.

Avoid raw integer error codes if stronger types already exist.

---

## Return Booleans

Do not return `bool` when callers need to know why an operation failed.

Bad:

```cpp
bool loadPackage(...);
```

if errors such as:

- Not found.
- Invalid format.
- Permission denied.

matter.

Use the established error model.

A boolean is fine for genuine yes/no operations.

---

## Output Parameters

Avoid output parameters when returning a value is clearer.

Bad:

```cpp
bool loadWeapon(
    const Path& path,
    Weapon& outWeapon);
```

Prefer:

```cpp
Weapon loadWeapon(const Path& path);
```

or an error-aware return type.

Output parameters may still be appropriate for:

- Performance-sensitive APIs.
- Multi-value legacy interfaces.
- C interoperability.

---

## Multiple Return Values

Use a small struct when returned values have meaning.

Prefer:

```cpp
struct ParseResult
{
    Weapon weapon;
    Metadata metadata;
};
```

over:

```cpp
std::tuple<Weapon, Metadata, bool, int>
```

for several unrelated values.

Tuples are fine for obvious small pairings.

---

## Structured Bindings

Use structured bindings when they improve readability.

Good:

```cpp
const auto [it, inserted] =
    weapons.emplace(id, weapon);
```

Do not destructure objects merely to shorten names when contextual access is clearer.

---

## Pair and Tuple

Use `std::pair` when there are naturally two associated values.

Use named structs once the meaning becomes non-obvious.

Avoid:

```cpp
std::tuple<int, int, bool, std::string>
```

as a domain API.

---

## Nullability

Use references for required objects.

Use pointers or optional-like types when absence is allowed.

Do not check every reference parameter for null—references cannot be null in valid C++.

Bad conceptual behavior:

```cpp
void processWeapon(const Weapon& weapon)
{
    // redundant imaginary null check
}
```

Trust the type system.

---

## Defensive Checks

Validate at system boundaries:

- User input.
- Network input.
- File formats.
- External APIs.
- C interfaces.
- Plugin boundaries.

Do not repeatedly validate internal invariants after they have already been established.

Excessive defensive code makes intent less clear.

---

## Assertions

Use assertions for programmer invariants.

Example:

```cpp
assert(index < weapons.size());
```

Do not use assertions for user-facing validation or recoverable runtime errors.

Assertions may disappear in release builds.

---

## Static Assertions

Use:

```cpp
static_assert(...)
```

for compile-time invariants.

Good examples include:

- Type size assumptions.
- Template requirements.
- ABI assumptions.

Do not add compile-time assertions simply to prove obvious language guarantees.

---

## Bounds Checking

Use `.at()` when out-of-range access is a legitimate runtime possibility requiring an exception.

Use `operator[]` when the index has already been validated and performance or convention favors it.

Do not replace all indexing with `.at()` mechanically.

---

## Iterator Invalidation

Understand container invalidation rules.

Do not keep iterators, references, or pointers across operations that may invalidate them.

Common invalidating operations include:

- Vector reallocation.
- Erase.
- Insert.
- Rehash.

Be deliberate about lifetime.

---

## Lifetime Safety

Watch for dangling:

- References.
- Pointers.
- `string_view`.
- `span`.
- Iterators.
- Lambda captures.

Modern C++ reduces manual memory management but does not eliminate lifetime bugs.

Do not return references to local variables.

---

## Temporary Lifetime

Be careful with expressions such as:

```cpp
std::string_view view = makeString();
```

if `makeString()` returns a temporary `std::string`.

The view dangles immediately.

Prefer owning the result when necessary.

---

## Undefined Behavior

Never rely on undefined behavior.

Common sources include:

- Out-of-bounds access.
- Use-after-free.
- Dangling references.
- Signed integer overflow.
- Invalid shifts.
- Misaligned casts.
- Uninitialized reads.
- Data races.

Do not trade correctness for cleverness.

---

## Uninitialized Variables

Initialize variables.

Bad:

```cpp
int count;
```

unless it is guaranteed to be assigned before any possible use and that pattern matches project conventions.

Prefer:

```cpp
int count = 0;
```

or direct initialization from the real value.

---

## Initialization

Prefer braces or direct initialization consistently with the project.

Example:

```cpp
Weapon weapon{id, name};
```

Be aware of initializer-list overload selection.

Do not blindly convert all construction to braces if it changes semantics.

---

## Narrowing Conversions

Brace initialization helps prevent some narrowing conversions.

Still be deliberate with conversions between:

- Signed and unsigned.
- Wider and narrower integers.
- Floating and integer.
- Pointer-sized types.

Do not use casts merely to silence warnings.

---

## Casts

Prefer C++ casts:

```cpp
static_cast
dynamic_cast
const_cast
reinterpret_cast
```

over C-style casts.

C++ casts communicate intent.

Avoid:

```cpp
(int)value
```

unless maintaining existing C-style code.

---

## `static_cast`

Use for explicit well-defined conversions.

Do not use it to silence a suspicious conversion without checking range or semantics.

---

## `reinterpret_cast`

Use only for low-level operations where representation reinterpretation is genuinely required.

It should be rare in application-level code.

Document assumptions when they are non-obvious.

---

## `const_cast`

Avoid unless interfacing with a legacy API or implementing a carefully justified const abstraction.

If `const_cast` appears frequently, the API design likely needs review.

---

## C APIs

At C boundaries:

- Keep raw pointers localized.
- Convert to C++ types promptly.
- Wrap handles in RAII types.
- Validate error codes.
- Preserve ownership semantics.

Do not let C-style lifetime management spread through the C++ codebase.

---

## RAII Wrappers for Handles

For resources like:

```text
FILE*
HANDLE
SOCKET
OpenGL object IDs
library handles
```

use RAII wrappers when ownership is non-trivial.

Do not manually repeat cleanup calls across multiple exit paths.

---

## Custom Deleters

Use smart pointer custom deleters when a C API requires non-`delete` cleanup.

Example:

```cpp
using FilePtr =
    std::unique_ptr<FILE, decltype(&std::fclose)>;
```

Keep such wrappers near the integration boundary.

---

## Virtual Dispatch

Use virtual dispatch only when runtime polymorphism is needed.

Do not add a base class merely to unit-test one implementation.

Dependency substitution can often be achieved with:

- Templates.
- Functions.
- Small explicit seams.

Choose the simplest architecture.

---

## Composition

Prefer:

```cpp
class WeaponLoader
{
    FileSystem& fileSystem_;
};
```

over inheriting from `FileSystem` merely to access behavior.

Inheritance should model an "is-a" relationship, not code reuse.

---

## Dependency Injection

C++ usually does not need a DI container.

Constructor injection is often enough:

```cpp
class WeaponLoader
{
public:
    explicit WeaponLoader(FileSystem& fileSystem)
        : fileSystem_(fileSystem)
    {
    }

private:
    FileSystem& fileSystem_;
};
```

Do not build:

```text
ServiceContainer
DependencyRegistry
ProviderFactory
Resolver
```

for straightforward dependency wiring.

---

## Factories

Use factories when construction:

- Can fail.
- Chooses among implementations.
- Requires hidden concrete types.
- Encapsulates complex resource creation.

Do not create:

```cpp
WeaponFactory
```

merely to call:

```cpp
return Weapon(...);
```

Direct construction is clearer.

---

## Builder Pattern

Use builders when:

- Construction has many optional parameters.
- Staged configuration improves clarity.
- Fluent APIs are part of the public interface.

Do not use a builder for:

```cpp
Weapon{id, name, damage};
```

when ordinary construction is simple.

---

## Fluent APIs

Fluent interfaces can be readable.

Do not turn every configuration API into:

```cpp
builder
    .withFoo(...)
    .withBar(...)
    .withBaz(...)
    .build();
```

unless the style genuinely improves usage.

---

## Registry Patterns

Use registries for real dynamically extensible systems.

Do not create a registry containing one or two fixed handlers.

A switch or map of functions may be simpler.

---

## Visitor Pattern

Use the classic visitor pattern only when the domain and architecture benefit from double dispatch.

In modern C++, `std::variant` plus `std::visit` may be simpler for closed sets.

Do not mechanically apply GoF patterns.

---

## Observer Pattern

Use observers/events when decoupled notification is genuinely required.

Do not introduce a global event bus for simple direct relationships.

Explicit calls are easier to understand.

---

## Callbacks

Use `std::function` when type-erased callable storage is required.

Do not use `std::function` for every callback parameter automatically.

Templates or function pointers may avoid allocation/type erasure where appropriate.

But do not over-generalize for tiny performance gains.

---

## `std::function`

Be aware it may allocate and uses type erasure.

Use it for ergonomic runtime callback storage.

Do not replace every `std::function` with custom template wrappers unless measured performance requires it.

---

## Function Pointers

Function pointers are appropriate for simple non-capturing callback APIs.

Do not use them when captured state or richer callable semantics are needed.

---

## Concurrency

Introduce concurrency only when the workload benefits from it.

Do not make straightforward code concurrent merely because C++ offers threads.

Concurrency adds:

- Data races.
- Deadlocks.
- Lifetime complexity.
- Ordering issues.
- Testing difficulty.

Keep code synchronous unless concurrency has a clear purpose.

---

## Threads

Use `std::thread` or the project's threading abstraction deliberately.

Prefer higher-level task systems already present in the codebase when appropriate.

Do not spawn threads for trivial work.

---

## `std::jthread`

In C++20+, `std::jthread` can simplify thread lifetime and cancellation.

Use it when the project supports it and the semantics fit.

Do not migrate existing threading code without reason.

---

## Mutexes

Use mutexes for actual shared mutable state.

Do not protect data with a mutex that could simply have one owner.

Keep lock scope small.

---

## Lock Guards

Use RAII locking:

```cpp
std::lock_guard lock(mutex_);
```

or:

```cpp
std::scoped_lock lock(mutexA, mutexB);
```

Do not manually call:

```cpp
mutex.lock();
...
mutex.unlock();
```

unless highly specialized behavior requires it.

RAII prevents forgotten unlocks.

---

## Avoid Holding Locks Too Long

Do not keep locks while performing:

- Slow I/O.
- Network calls.
- Long computation.
- User callbacks.

unless the protected invariant requires it.

Copy or move needed state out when practical.

---

## `std::shared_mutex`

Use shared mutexes only when many readers and few writers make the complexity worthwhile.

Do not assume reader/writer locks are always faster.

Measure contentious code before optimizing.

---

## Atomics

Use atomics for simple lock-free shared values when their memory semantics are well understood.

Do not replace mutexes with atomics casually.

Memory ordering is subtle.

Prefer default sequential consistency unless weaker ordering is deliberately justified.

---

## Memory Ordering

Do not use:

```cpp
std::memory_order_relaxed
```

because it looks faster.

Use weaker memory ordering only when you can explain why it preserves correctness.

Concurrency bugs are expensive.

---

## Data Races

Data races are undefined behavior.

Do not assume "it probably works" because accesses are small or naturally aligned.

Synchronize shared mutable state properly.

---

## Condition Variables

Use condition variables for coordination when appropriate.

Always wait using a predicate:

```cpp
condition.wait(lock, [&]
{
    return ready_;
});
```

This handles spurious wakeups.

Do not assume one wake means the condition is true.

---

## Futures and Async

Use the project's task/concurrency system.

Be cautious with:

```cpp
std::async
```

because launch behavior can be surprising unless policy is specified.

Do not create asynchronous APIs without a real concurrency requirement.

---

## Coroutines

If the project uses C++20 coroutines, follow its established framework.

Do not introduce raw coroutine machinery for ordinary synchronous code.

Coroutines require supporting types and lifetime discipline.

---

## Thread Pools

Use an existing thread pool rather than creating another.

Do not implement a custom thread pool unless the task genuinely requires one and existing dependencies do not provide it.

Thread pools are easy to get subtly wrong.

---

## Lock-Free Programming

Avoid lock-free structures unless:

- Performance requirements justify them.
- Correctness can be rigorously established.
- Experienced maintainers can support them.

A mutex is often the professional solution.

---

## Filesystem

Use:

```cpp
std::filesystem::path
```

for filesystem paths.

Do not represent paths as ordinary strings throughout the codebase if filesystem semantics matter.

Use:

```cpp
path / "config" / "settings.json"
```

instead of manually concatenating separators.

---

## Filesystem Encoding

Be aware that filesystem path encoding differs by platform.

Do not assume every path is UTF-8 internally.

Avoid unnecessary conversion to narrow strings.

Use `std::filesystem::path` APIs directly.

---

## File I/O

Use standard streams or project abstractions appropriately.

Always check failures when they matter.

Example:

```cpp
std::ifstream file(path, std::ios::binary);

if (!file)
{
    throw FileError(path);
}
```

Do not assume file operations succeed.

---

## Streams

Use iostreams when they fit the project.

For performance-sensitive parsing, alternative libraries may be appropriate.

Do not rewrite ordinary file code into low-level C APIs solely for perceived speed.

Measure first.

---

## Formatting

If the project uses C++20 `std::format`, `{fmt}`, or another formatting library, follow that convention.

Prefer type-safe formatting over fragile printf-style format strings when available.

Do not add a formatting dependency just to replace one simple stream expression.

---

## Logging

Use the project's established logger.

Log meaningful events and failures.

Avoid:

```cpp
logger.info("Entering function");
logger.info("Processing weapon");
logger.info("Finished processing weapon");
```

unless those events matter operationally.

Prefer contextual information:

```cpp
logger.warn(
    "Unable to resolve weapon asset '{}'",
    path.string());
```

Do not log the same exception/error at every layer.

---

## `std::cout`

Use `std::cout` for:

- CLI output.
- Simple tools.
- Intentional stdout protocol.

Do not use it as production application logging when a logging framework exists.

Remove debugging output before finishing.

---

## Debug Logging

Do not leave:

```cpp
std::cout << "here\n";
std::cerr << "test\n";
```

inside finished code.

Use the project logger or remove the output.

---

## Preconditions

Express clear preconditions through:

- Types.
- Assertions.
- Documentation.
- Validation at boundaries.

Do not silently assume untrusted input satisfies internal invariants.

---

## Postconditions

When an operation must guarantee a state, structure code so the invariant naturally holds.

Avoid scattered comments promising conditions the type system or code does not enforce.

---

## Validation

Validate external data at trust boundaries.

Examples:

- Files.
- Network messages.
- User input.
- Plugins.
- IPC.
- Deserialized data.
- C APIs.

After validation, trust internal types.

Do not repeat identical validation throughout the call chain.

---

## Parsing

Write parsers that fail clearly.

Avoid fragile code like:

```cpp
const auto value =
    std::stoi(line.substr(line.find(':') + 1));
```

when malformed input is possible.

Check boundaries and return meaningful errors.

Do not build enormous parser frameworks for simple formats.

---

## Numeric Parsing

Prefer modern parsing APIs such as:

```cpp
std::from_chars
```

when performance and error handling matter and project support permits it.

Do not replace simple established parsing code merely because `from_chars` is newer.

---

## Numeric Conversions

Be careful with signed/unsigned conversions.

Do not silence warnings using:

```cpp
static_cast<int>(size)
```

without considering range.

Use appropriate types from the start.

---

## `size_t`

Use `std::size_t` for container sizes and indices where appropriate.

Do not force it into domain values that are semantically signed or bounded differently.

---

## Signed vs Unsigned

Avoid unnecessary mixed signed/unsigned comparisons.

Do not respond to warnings by adding casts everywhere.

Fix type choices where practical.

Modern C++20 helpers such as:

```cpp
std::cmp_less
std::cmp_equal
```

may be useful if supported.

---

## Integer Overflow

Signed integer overflow is undefined behavior.

Use appropriate checked or wider arithmetic when input can exceed limits.

Do not add overflow wrappers everywhere when ranges are inherently safe.

---

## Floating Point

Do not compare floating-point values with approximate tolerance mechanically.

Use tolerance when the domain requires approximate equality.

Exact comparisons are valid for certain values and state checks.

Define tolerance according to domain scale, not a random epsilon.

---

## `NaN`

Remember NaN has unusual comparison behavior.

Do not sort floating-point values containing NaN without defining intended semantics.

---

## Time

Use:

```cpp
std::chrono
```

for durations and clocks.

Do not represent durations as bare integers without units when confusion is possible.

Good:

```cpp
std::chrono::milliseconds timeout;
```

Better than:

```cpp
int timeout;
```

when units matter.

---

## `std::chrono`

Prefer typed durations:

```cpp
using namespace std::chrono_literals;

auto timeout = 500ms;
```

if the project accepts chrono literals.

Do not use literals if they conflict with local style.

---

## Clock Choice

Use:

```cpp
std::chrono::steady_clock
```

for elapsed-time measurement.

Use system clock for wall-clock timestamps.

Do not measure durations using wall-clock time.

---

## Randomness

Use standard or established library random facilities.

Do not use:

```cpp
rand()
```

for modern security-sensitive or statistically meaningful random behavior.

Do not implement custom PRNGs without a specific need.

---

## Security-Sensitive Randomness

Use a cryptographically secure source when generating:

- Tokens.
- Keys.
- Nonces.
- Password reset values.

The normal C++ random library is not automatically appropriate for cryptographic security.

Follow project/security library conventions.

---

## Serialization

Use the project's established serialization library.

Do not hand-roll:

- JSON.
- XML.
- Binary protocol parsing.

for complex formats when mature libraries are already in use.

Validate untrusted serialized data.

---

## JSON

Popular choices may include:

```text
nlohmann/json
RapidJSON
simdjson
Boost.JSON
```

Follow the existing dependency.

Do not add a second JSON library without a strong reason.

---

## Network Code

Use the project's networking stack.

Do not write raw socket code if a higher-level established abstraction already exists and suits the requirement.

Likewise, do not add a heavy HTTP library for a tiny project without considering dependency cost.

---

## Socket Ownership

Wrap sockets and OS handles in RAII.

Do not manually close them across multiple return paths.

---

## Timeouts

Network operations should have meaningful timeout behavior.

Do not allow remote operations to hang indefinitely unless explicitly intended.

Do not invent tiny arbitrary timeouts either.

---

## Retries

Retry only transient failures.

Do not retry:

- Invalid requests.
- Authentication failures requiring new credentials.
- Deterministic parse errors.
- Programming errors.

Use bounded retries and existing project mechanisms.

---

## Database Code

Use the established database abstraction directly when appropriate.

Do not automatically layer:

```text
Controller
Service
Repository
Manager
Provider
```

over straightforward database operations.

Add boundaries where they provide actual value.

---

## Transactions

Use transactions for operations that must succeed or fail atomically.

Keep transaction scope small.

Do not hold database transactions open across unrelated slow external operations without a reason.

---

## SQL

Use parameterized queries.

Never concatenate untrusted values into SQL strings.

Follow the database library's binding API.

---

## API Boundaries

Keep public APIs simpler than internal implementation complexity.

Avoid exposing:

- Internal container choices.
- Implementation-only dependencies.
- Raw ownership details.
- Internal synchronization primitives.

Do not hide every type behind an interface either.

Expose what callers actually need.

---

## Pimpl

Use the Pimpl idiom when it provides real benefits such as:

- ABI stability.
- Reducing header dependencies.
- Hiding implementation detail in public libraries.

Do not use Pimpl for every internal class.

It adds:

- Allocation.
- Indirection.
- Boilerplate.
- More complex move/copy behavior.

Use it where ABI or compile-time concerns justify it.

---

## ABI Stability

For public binary libraries, consider ABI carefully.

Avoid changing:

- Public class layout.
- Virtual tables.
- Exported signatures.

casually when ABI compatibility matters.

Do not impose ABI-preserving architecture on ordinary internal applications unnecessarily.

---

## DLL Boundaries

On Windows, be deliberate with exported classes, allocators, STL types, and runtime boundaries.

Follow the project's established export macros and ABI conventions.

Do not invent new export systems.

---

## Platform Abstraction

Keep platform-specific code isolated when practical.

Prefer:

```text
platform/windows/
platform/linux/
```

or implementation files over repeated `#ifdef` branches throughout domain logic.

Do not create a giant abstraction layer for one small platform difference.

---

## Windows APIs

Use RAII wrappers around Windows handles.

Do not leak raw `HANDLE` cleanup requirements across the application.

Use `CloseHandle` through a wrapper with correct ownership.

---

## POSIX APIs

Likewise, wrap:

- File descriptors.
- Sockets.
- Mappings.

in RAII where ownership is non-trivial.

Do not manually close resources across many control-flow branches.

---

## Memory Allocation

Do not optimize allocator behavior without evidence.

Avoid custom allocators, arenas, pools, and PMR unless:

- Allocation is measured as a bottleneck.
- Object lifetime patterns clearly benefit.
- The project already uses them.

Standard containers are usually sufficient.

---

## `std::pmr`

Polymorphic allocators can be valuable in allocation-heavy systems.

Do not introduce PMR to ordinary application code merely because it is modern C++.

It adds allocator lifetime and ownership complexity.

---

## Arena Allocation

Use arenas when many objects share lifetime and allocation overhead matters.

Do not use arenas in simple CRUD-style or tool code without evidence.

---

## Cache Locality

When performance matters, consider data layout.

But do not rewrite clear structures into SoA/packed formats based on speculation.

Profile before large architectural changes.

---

## Alignment

Use alignment controls only when required by:

- SIMD.
- Hardware interfaces.
- ABI.
- Cacheline separation.

Do not add `alignas` without a concrete need.

---

## SIMD

Use SIMD only for measured performance-sensitive workloads.

Prefer compiler auto-vectorization or established libraries before custom intrinsics.

Custom SIMD code raises maintenance and platform complexity.

---

## `volatile`

Do not use `volatile` for thread synchronization.

It does not provide atomicity or inter-thread ordering.

Valid uses may include:

- Memory-mapped hardware.
- Certain signal interactions.
- Platform-specific low-level code.

Use atomics for concurrency.

---

## Memory Ownership in APIs

Document non-obvious ownership rules.

Avoid APIs like:

```cpp
Widget* createWidget();
```

without making ownership obvious.

Prefer:

```cpp
std::unique_ptr<Widget> createWidget();
```

if ownership transfers.

---

## Borrowed Results

If returning a pointer/reference into internal storage, make invalidation rules clear when not obvious.

Do not return internal references that become invalid immediately after routine operations unless the API clearly establishes that constraint.

---

## API Lifetimes

Be wary of callbacks retaining pointers to short-lived objects.

Prefer ownership-aware types and explicit unregister semantics.

Do not rely on informal lifetime assumptions.

---

## `enable_shared_from_this`

Use it only when an object managed by `shared_ptr` genuinely needs to hand out shared ownership to itself.

Do not use it as an excuse to make everything shared ownership.

Misuse can lead to `bad_weak_ptr` and tangled lifetimes.

---

## Cyclic Ownership

Avoid `shared_ptr` cycles.

Use:

```cpp
weak_ptr
```

for back-references when shared ownership architecture genuinely requires it.

Better yet, reconsider whether shared ownership is necessary.

---

## Exception vs Error-Return Consistency

Do not mix:

```text
exceptions
bool returns
optional
error codes
expected
```

randomly for similar operations.

Follow the project's established conventions.

Consistency is more important than theoretical preference.

---

## Logging vs Error Handling

Logging an error does not handle it.

Do not:

```cpp
logger.error(...);
return {};
```

unless returning an empty result is the established recovery behavior.

Preserve failure information when callers need it.

---

## Error Messages

Write actionable errors.

Good:

```text
Unable to load package '/Game/Weapons/Rifle_A': file was not found.
```

Bad:

```text
Operation failed.
```

Include useful identifiers such as:

- Paths.
- IDs.
- Operation names.
- Resource names.

Do not include secrets.

---

## Comments

Do not narrate the code.

Bad:

```cpp
// Check if weapon is null
if (weapon == nullptr)
{
    // Return if weapon doesn't exist
    return;
}
```

Good:

```cpp
if (weapon == nullptr)
    return;
```

Write comments when they explain:

- Why an unusual technique exists.
- A platform limitation.
- A third-party bug.
- A protocol requirement.
- A performance tradeoff.
- A lifetime invariant.
- An unsafe interoperability requirement.

Prefer explaining **why**, not **what**.

---

## Avoid AI-Looking Comments

Avoid:

```text
This function is responsible for...
This method ensures...
The following implementation...
In order to...
It is important to note...
This provides a robust and flexible solution...
```

Bad:

```cpp
// This function is responsible for validating the weapon data
// before it is processed by the system.
```

Better:

```cpp
// Imported manifests may omit fields present in runtime assets.
```

---

## Doxygen

Use Doxygen-style comments when the project expects them or public API documentation genuinely benefits.

Do not generate verbose documentation for obvious private functions.

Avoid:

```cpp
/**
 * @brief Gets the weapon name.
 * @return The weapon name.
 */
std::string_view name() const;
```

when the API is self-explanatory.

---

## Documentation

Document public APIs when behavior is not obvious.

Useful topics include:

- Ownership.
- Lifetime.
- Thread safety.
- Exceptions.
- Preconditions.
- Invalidated references.
- Units.
- Performance characteristics.

Do not document obvious syntax.

---

## Thread Safety Documentation

If a public type has important concurrency guarantees, state them.

Examples:

```text
Thread-safe for concurrent reads.
External synchronization required for mutation.
Not thread-safe.
```

Do not claim thread safety merely because a mutex exists somewhere.

---

## Avoid Decorative Comments

Do not add:

```cpp
// ========================================
// INITIALIZATION
// ========================================
```

unless the repository intentionally uses this style.

Code organization should normally make these unnecessary.

---

## Avoid Placeholder Code

Do not leave:

```cpp
// TODO: implement later
throw std::logic_error("Not implemented");
```

in finished production work unless the user explicitly requested a scaffold.

Complete the requested behavior.

---

## Avoid Premature Abstraction

Do not design for hypothetical future requirements.

If one implementation exists, direct code may be best.

Do not add:

- Interfaces for one type.
- Factories for one constructor.
- Registries with one entry.
- Template frameworks for one data type.
- Plugin systems with one plugin.
- Builders for simple objects.
- Custom allocators without performance need.
- Shared ownership without actual sharing.

Build what the current code needs.

---

## Avoid AI-Looking Architecture

Do not automatically introduce:

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

Use them only when they describe an actual architectural role.

Generated code often creates symmetric architecture where none is needed.

---

## Avoid Pass-Through Layers

Bad:

```cpp
class WeaponService
{
public:
    Weapon getWeapon(WeaponId id)
    {
        return repository_.getWeapon(id);
    }

private:
    WeaponRepository& repository_;
};
```

if the service adds no behavior.

Every layer should earn its existence.

---

## Avoid Utility Classes

Bad:

```cpp
class WeaponUtils
{
public:
    static std::string normalizeName(
        std::string_view name);
};
```

Prefer:

```cpp
std::string normalizeWeaponName(
    std::string_view name);
```

inside an appropriate namespace.

Namespaces already provide grouping.

---

## Avoid Getters and Setters Everywhere

Do not automatically generate:

```cpp
getName()
setName()
getDamage()
setDamage()
```

for every field.

Use simple structs for data or meaningful domain operations for behavior.

Boilerplate is not encapsulation.

---

## Avoid Shared Ownership as Dependency Injection

Do not inject every dependency as:

```cpp
std::shared_ptr<IService>
```

by default.

If the dependency must outlive the consumer and ownership is external:

```cpp
Service&
```

or:

```cpp
Service*
```

may communicate the relationship more accurately.

Ownership and polymorphism are separate concerns.

---

## Avoid Template-Heavy Dependency Injection

Do not replace straightforward constructor injection with compile-time dependency graph frameworks unless the project already uses them.

Compile-time cleverness can increase build times and onboarding cost.

---

## Avoid Excessive `std::function`

Do not type-erase callables unless runtime polymorphism is required.

For internal generic utilities, a template parameter may avoid overhead.

But do not template entire application layers merely to eliminate one `std::function`.

Balance clarity and cost.

---

## Avoid Excessive `shared_ptr`

Before adding one, ask:

- Who owns this?
- Who destroys it?
- Does ownership truly need to be shared?
- Would a reference express non-ownership?
- Would `unique_ptr` be enough?

AI-generated C++ frequently uses `shared_ptr` as an ownership escape hatch.

Do not.

---

## Avoid Excessive Heap Allocation

Do not heap-allocate values solely because they are objects.

Prefer stack/value storage when lifetime and size make sense.

Heap allocation should correspond to:

- Dynamic lifetime.
- Polymorphism.
- Stable addresses.
- Large object concerns.
- Ownership transfer.

not habit.

---

## Avoid Excessive Abstraction Around STL

Do not wrap:

```cpp
std::vector
std::unordered_map
std::filesystem
std::optional
```

in custom types merely to hide the standard library.

Wrap them when domain invariants or APIs genuinely benefit.

---

## Avoid Homegrown STL Replacements

Do not create custom:

```text
DynamicArray
HashMap
Optional
SmartPointer
String
```

unless the project has specialized platform/performance requirements.

Use proven standard or established project containers.

---

## Avoid Macro Constants

Prefer:

```cpp
constexpr
```

or:

```cpp
enum class
```

instead of macros for typed constants.

Macros bypass normal language scoping and type checking.

---

## Avoid C-Style Arrays

Prefer:

```cpp
std::array
std::vector
std::span
```

where appropriate.

Raw arrays are mainly useful for:

- C interoperability.
- Embedded/low-level contexts.
- Certain compile-time data.

---

## Avoid `memset` on C++ Objects

Do not initialize complex C++ types using:

```cpp
std::memset(&object, 0, sizeof(object));
```

unless the type is specifically safe for byte-wise initialization.

Use constructors and value initialization.

---

## Avoid `memcpy` for Object Copying

Use normal copy/move semantics.

`memcpy` is only appropriate for trivially copyable representations when byte-level copying is intentional.

Do not assume object bytes can be copied safely.

---

## Avoid `malloc` and `free`

Use C++ allocation/lifetime abstractions unless interfacing with a C API or specialized allocator.

Do not mix:

```cpp
new/delete
malloc/free
```

for the same resource.

---

## Avoid Raw Ownership Arrays

Bad:

```cpp
Weapon* weapons = new Weapon[count];
```

Prefer:

```cpp
std::vector<Weapon> weapons(count);
```

unless a specialized low-level context demands otherwise.

---

## Avoid Unnecessary `std::endl`

Prefer:

```cpp
'\n'
```

when flushing is not needed.

`std::endl` inserts a newline and flushes the stream.

Do not force a flush on every line accidentally.

---

## Avoid Repeated `.size()` Conversions

Choose compatible types for indices and sizes.

Do not scatter casts merely to silence warnings.

Use:

```cpp
std::size(container)
```

where appropriate and supported.

---

## Avoid Magic Numbers

Give meaningful names to non-obvious values.

Good:

```cpp
constexpr auto MaxRetryAttempts = 3;
constexpr auto HeaderSize = 32U;
```

Do not create constants for trivial values such as:

```cpp
constexpr int Zero = 0;
```

A name should communicate meaning.

---

## Units

Encode units clearly.

Good:

```cpp
std::chrono::milliseconds timeout;
float distanceMeters;
```

Do not use generic:

```cpp
int timeout;
float distance;
```

when unit confusion is realistic.

Strong unit types may be justified in safety-critical or calculation-heavy systems.

---

## Bit Flags

Use scoped enums and bit operations carefully.

Do not use unexplained integer masks throughout the code.

Give flags meaningful names.

If the project has an established flag utility, use it.

---

## Bitfields

Use C++ bitfields only when layout or hardware representation makes them appropriate.

Do not rely on implementation-specific bitfield layout for portable serialized formats.

---

## Binary Formats

When parsing binary data:

- Validate lengths.
- Handle endianness explicitly.
- Avoid unaligned casts.
- Avoid pointer arithmetic without bounds.
- Do not rely on struct packing matching disk format.

Use deliberate decoding.

---

## `reinterpret_cast` Parsing

Avoid:

```cpp
auto* header =
    reinterpret_cast<const Header*>(bytes.data());
```

unless alignment, lifetime, representation, and endianness are all guaranteed.

Safer explicit decoding is often preferable.

---

## Packing

Avoid compiler-specific packing pragmas unless ABI or binary format requirements demand them.

Packed structs can cause unaligned accesses.

Document their purpose.

---

## Endianness

Do not assume host endianness when parsing portable binary/network formats.

Use explicit conversion where needed.

C++20's `<bit>` facilities or project helpers may assist.

---

## Network Byte Order

Use appropriate conversion utilities.

Do not manually reverse bytes unless necessary.

Keep protocol handling explicit.

---

## Serialization Compatibility

Do not silently change persisted binary layouts by reordering struct fields.

Serialized formats should not depend on raw in-memory object representation unless the format explicitly defines that ABI.

---

## Security

Do not weaken safety for convenience.

Avoid:

- Buffer overflows.
- Unchecked pointer arithmetic.
- Format-string vulnerabilities.
- Shell command construction from untrusted input.
- Unsafe deserialization.
- Path traversal.
- Integer overflow.
- Data races.
- Lifetime violations.

Use safe library facilities.

---

## String Formatting Security

Do not pass untrusted strings as format strings to printf-style APIs.

Bad:

```cpp
printf(userInput);
```

Use:

```cpp
printf("%s", userInput);
```

or a type-safe formatting API.

---

## Subprocesses

Prefer APIs that separate executable and arguments.

Avoid building shell commands from untrusted strings.

Use a shell only when shell semantics are genuinely required.

---

## Path Traversal

When handling untrusted paths inside a restricted root, validate that resolved paths remain within the allowed root.

Do not add sandbox restrictions to ordinary developer tools where unrestricted paths are expected.

Apply security at the correct boundary.

---

## Temporary Files

Use secure temporary-file facilities or established project utilities.

Do not generate predictable names in shared temp directories when attackers may influence the environment.

---

## Compiler Warnings

Compile with the project's warning policy.

Common warnings may include:

```text
-Wall
-Wextra
-Wpedantic
/W4
```

Do not blindly add every possible warning flag.

Respect project/compiler portability.

Treat meaningful warnings as real problems.

---

## Do Not Silence Warnings with Casts

Bad:

```cpp
static_cast<int>(value)
```

added only because the compiler complained.

Understand whether the conversion is safe.

Fix the type mismatch if possible.

---

## Warning Suppression

Use targeted suppressions only when:

- The warning is understood.
- The code is correct.
- A better expression is unavailable.
- Third-party/platform code requires it.

Do not globally suppress warnings to make generated code compile.

---

## `clang-tidy`

If the project uses clang-tidy, respect its configuration.

Do not rewrite the entire codebase to satisfy checks unrelated to the task.

Do not disable checks simply to avoid addressing real issues.

---

## Static Analysis

Use existing tools such as:

```text
clang-tidy
clang-analyzer
MSVC Code Analysis
Cppcheck
PVS-Studio
```

when configured.

Do not add several overlapping analyzers without project agreement.

---

## Sanitizers

For relevant code, sanitizers can catch serious bugs.

Common choices:

```text
AddressSanitizer
UndefinedBehaviorSanitizer
ThreadSanitizer
MemorySanitizer
```

Use the project's existing configuration.

Do not enable incompatible sanitizers simultaneously without understanding their requirements.

---

## AddressSanitizer

Useful for:

- Use-after-free.
- Buffer overflows.
- Invalid memory access.

Do not consider a clean ASan run proof of complete memory correctness.

---

## UndefinedBehaviorSanitizer

Useful for detecting many UB cases.

Do not rely solely on sanitizers to justify questionable low-level code.

Write safe code first.

---

## ThreadSanitizer

Use when concurrency behavior matters and the platform/toolchain supports it.

Do not assume locking is correct because ordinary tests pass.

---

## Testing

Use the project's established framework.

Common choices include:

```text
GoogleTest
Catch2
doctest
Boost.Test
CTest
```

Do not replace the test framework without a reason.

Test behavior, not internal implementation details.

---

## Test Naming

Use descriptive names consistent with the project.

Example:

```cpp
TEST(WeaponParser, RejectsMissingWeaponId)
```

or:

```cpp
TEST_CASE("parseWeapon rejects missing weapon id")
```

Avoid vague names:

```cpp
TEST(WeaponParser, Test1)
```

---

## Avoid Over-Mocking

Do not introduce interfaces solely to mock everything.

Prefer:

- Real lightweight collaborators.
- In-memory implementations.
- Temporary files.
- Small fakes.

Mock external boundaries when appropriate.

Over-mocking creates brittle tests tied to implementation.

---

## Test Fixtures

Use fixtures for shared setup when they reduce real repetition.

Do not create a fixture hierarchy for three simple tests.

Local setup is often easier to read.

---

## Test Helpers

Avoid:

```text
MockFactory
FixtureManager
TestDataBuilder
TestingUtility
```

unless the test suite genuinely benefits.

Use straightforward helper functions when enough.

---

## Integration Tests

Use integration tests where behavior spans:

- Filesystem.
- Network boundaries.
- Database.
- Process invocation.
- Multiple major components.

Do not use integration tests for every pure helper.

---

## Deterministic Tests

Avoid tests depending on:

- Current wall-clock time.
- Real network access.
- Random ordering.
- Existing filesystem state.
- Undefined container iteration order.
- Thread timing.

Control nondeterminism where practical.

---

## Randomized Tests

If fuzzing or property-based testing is appropriate, use established tooling.

Do not build custom random-test loops when a mature framework exists.

---

## Fuzzing

C++ parsers and binary decoders often benefit from fuzzing.

Tools may include:

```text
libFuzzer
AFL++
Honggfuzz
```

Do not add fuzz infrastructure to trivial application logic without need.

---

## Benchmarks

Benchmark before optimizing.

Use the project's benchmark framework.

Do not write informal timing loops and treat them as reliable performance measurements.

---

## Profiling

Use real profilers for performance work.

Do not optimize based on intuition alone.

Possible tools include:

```text
Visual Studio Profiler
perf
Tracy
VTune
Instruments
```

Follow platform/project tooling.

---

## Performance

Prefer clear, correct code first.

Do not prematurely:

- Custom-allocate everything.
- Add SIMD.
- Add lock-free structures.
- Replace virtual calls.
- Rewrite vectors.
- Inline every function.
- Add cache layers.
- Use bit hacks.

Measure actual bottlenecks.

---

## Inline

Use:

```cpp
inline
```

primarily for ODR/header semantics.

Do not add it as a performance request to the compiler.

Modern compilers decide inlining independently.

---

## `[[nodiscard]]`

Use `[[nodiscard]]` when ignoring a result is likely a bug.

Good candidates:

```cpp
[[nodiscard]]
Result loadPackage(...);
```

Do not annotate every trivial getter.

Use it where the API benefits.

---

## Attributes

Use standard attributes when they communicate real semantics:

```cpp
[[nodiscard]]
[[maybe_unused]]
[[fallthrough]]
[[deprecated]]
```

Do not add attributes simply to silence diagnostics without understanding the issue.

---

## `[[maybe_unused]]`

Use for intentionally unused entities such as feature/platform-specific variables.

Do not use it to hide dead generated code.

Remove truly unused code.

---

## Fallthrough

Use:

```cpp
[[fallthrough]];
```

when switch fallthrough is intentional.

Do not rely on comments alone if the compiler supports the standard attribute.

---

## Switch Statements

Handle enums deliberately.

Avoid default branches when exhaustive handling is useful and the compiler can warn about missing variants.

Use `default` when unknown/future values should share behavior.

---

## Fallthrough Bugs

Do not rely on accidental fallthrough.

Explicitly mark intentional fallthrough.

---

## Conditionals

Prefer guard clauses over deep nesting.

Good:

```cpp
if (!weapon)
    return {};

if (!weapon->isLoaded())
    weapon->load();

return weapon;
```

Avoid:

```cpp
if (weapon)
{
    if (!weapon->isLoaded())
    {
        weapon->load();
    }

    return weapon;
}

return {};
```

Keep the normal path readable.

---

## Avoid Unnecessary `else`

After:

```text
return
throw
continue
break
```

an `else` is often unnecessary.

Use it only when it improves structure.

---

## Ternaries

Use ternaries for simple expression selection.

Good:

```cpp
const auto label =
    enabled ? "Enabled" : "Disabled";
```

Avoid nested ternaries for multi-step logic.

---

## Boolean Parameters

Avoid APIs like:

```cpp
processWeapon(weapon, true, false, true);
```

Prefer:

- Named enum options.
- Configuration structs.
- Separate functions.

Example:

```cpp
struct ProcessOptions
{
    bool validate = true;
    bool includeMetadata = false;
};
```

Use an options struct only when several related options actually exist.

---

## Enums Over Boolean Modes

If a boolean changes behavior meaningfully, an enum may be clearer.

Instead of:

```cpp
loadPackage(path, true);
```

consider:

```cpp
loadPackage(
    path,
    LoadMode::Cached);
```

Do not replace obvious yes/no properties with enums unnecessarily.

---

## Configuration Structs

Use configuration structs when many related options belong together.

Do not create a config type for a function with two obvious parameters.

---

## Designated Initializers

In C++20, designated initializers can improve aggregate clarity.

Example:

```cpp
WeaponStats stats{
    .damage = 42.0f,
    .fireRate = 700.0f,
};
```

Use them if the project supports and accepts them.

Do not introduce them in pre-C++20 projects.

---

## Defaults

Provide meaningful defaults.

Do not invent defaults that create invalid objects.

A default-constructed object should make semantic sense if a public default constructor exists.

Delete the default constructor if no valid default state exists.

---

## `= default`

Use:

```cpp
Weapon() = default;
```

when explicitly defaulting clarifies API intent.

Do not write trivial empty constructors manually.

---

## `= delete`

Delete unwanted operations explicitly when they must not exist.

Example:

```cpp
FileHandle(const FileHandle&) = delete;
```

Do not delete ordinary operations without a semantic reason.

---

## Private Helpers

Create private helpers when they encapsulate meaningful repeated or complex behavior.

Do not split every few lines into another function.

Excessive tiny helpers make control flow harder to follow.

---

## File Organization

Keep files cohesive.

Do not enforce one-class-per-file dogmatically.

Small related types may belong together.

Avoid project structures such as:

```text
weapon/
  managers/
    services/
      processors/
        handlers/
```

unless each layer is truly distinct.

Prefer domain-oriented organization.

---

## Generic Utility Files

Avoid giant:

```text
Utils.h
Helpers.h
Common.h
Misc.h
```

collections of unrelated functions.

Prefer cohesive names:

```text
WeaponParsing.h
PackagePaths.h
AssetValidation.h
```

Keep local helpers close to their usage.

---

## Header Dependencies

Avoid huge umbrella headers in internal code unless the project intentionally uses them.

Do not include:

```cpp
Everything.h
```

for one type if granular headers are the project norm.

---

## Precompiled Headers

If the project uses PCH, follow its conventions.

Do not add/remove headers from PCH casually; this can affect build performance and dependency assumptions.

---

## Build Systems

Use the existing build system.

Do not migrate:

```text
CMake
Meson
Bazel
Visual Studio projects
Premake
```

as part of unrelated coding work.

---

## CMake

When using CMake:

- Prefer target-based commands.
- Keep dependencies scoped.
- Avoid global flags where target properties suffice.
- Follow existing minimum CMake version.

Do not modernize the entire CMake project during a small feature change.

---

## Target-Based CMake

Prefer:

```cmake
target_link_libraries(...)
target_include_directories(...)
target_compile_features(...)
```

over globally modifying compiler settings when the project already uses modern CMake.

---

## Dependencies

Use the project's existing package/dependency system.

Possible tools include:

```text
vcpkg
Conan
FetchContent
CPM
system packages
vendored libraries
```

Do not add another dependency manager casually.

---

## Adding Dependencies

Before adding a library, ask:

- Does the standard library already solve this?
- Does the project already have something equivalent?
- Is the dependency maintained?
- Is ABI compatibility relevant?
- Does it support project platforms?
- Does it support the project's compiler/MSVC/GCC/Clang range?
- Is the dependency weight justified?

Do not add a library for five lines of straightforward code.

Do not reimplement cryptography, parsers, or complex standards solely to avoid dependencies.

---

## Boost

Use Boost when the project already depends on it or a Boost component clearly solves the problem.

Do not add all of Boost merely for one trivial utility if a standard library equivalent exists.

Likewise, do not rewrite established Boost-based code merely because newer standard equivalents exist unless modernization is requested.

---

## Third-Party APIs

Wrap third-party APIs only when the wrapper provides real value such as:

- Ownership translation.
- Error translation.
- Platform isolation.
- Stable domain API.

Do not add a pass-through wrapper around every external method.

---

## ABI and Compiler Boundaries

Be cautious when passing STL containers, exceptions, or allocation ownership across DLL/compiler runtime boundaries.

Follow the project's established ABI rules.

Do not invent ABI-safe wrappers without understanding deployment requirements.

---

## Cross-Platform Code

Use platform-neutral standard APIs where possible.

Examples:

```cpp
std::filesystem
std::thread
std::chrono
```

when they satisfy requirements.

Use platform-specific APIs when necessary.

Do not create custom portability layers for functionality the standard library already handles adequately.

---

## Embedded C++

In embedded or resource-constrained projects, normal desktop C++ assumptions may not apply.

Check whether the codebase avoids:

- Exceptions.
- RTTI.
- Dynamic allocation.
- Standard iostreams.

Follow project constraints.

Do not force general-purpose modern C++ practices where platform requirements prohibit them.

---

## Game/Engine Code

When working inside engines such as:

- Unreal Engine.
- Custom game engines.
- Embedded engines.

follow engine conventions over generic C++ advice.

For example, an engine may provide its own:

- Containers.
- Smart pointers.
- Reflection.
- Object ownership.
- Strings.
- Logging.
- Memory allocators.

Do not mix STL and engine types casually if the engine has explicit conventions.

---

## Unreal Engine

If working in Unreal, follow Unreal's C++ style and object model.

Do not replace:

```text
TArray
TMap
FString
TSharedPtr
UObject ownership
UPROPERTY
UFUNCTION
```

with STL equivalents unless the code is specifically outside those systems and project conventions permit it.

Generic C++ style yields to framework requirements.

---

## API Performance Contracts

If a function is performance-sensitive, document meaningful constraints.

Do not litter every function with Big-O comments.

Document complexity when callers genuinely need to choose behavior based on it.

---

## Hot Paths

In hot loops:

- Avoid unnecessary allocations.
- Avoid repeated virtual dispatch if measured significant.
- Avoid repeated string conversion.
- Avoid locking unnecessarily.

But optimize only after identifying actual hot paths.

---

## Branch Prediction Hints

Use:

```cpp
[[likely]]
[[unlikely]]
```

sparingly and only where branch probabilities are well understood.

Do not add them based on guesswork.

Compilers and CPUs already predict well.

---

## Cache-Friendly Design

Prefer contiguous data when performance matters.

But do not convert every object model into data-oriented architecture without evidence.

Use profiling and workload knowledge.

---

## Exception Cost

Do not avoid exceptions purely because "exceptions are slow."

The performance model depends on:

- Compiler.
- Platform.
- Whether exceptions are thrown.
- Binary size.
- Codebase architecture.

Follow project policy.

---

## Virtual Function Cost

Do not eliminate virtual calls unless profiling shows they matter or static architecture is naturally clearer.

A virtual call is often insignificant compared with I/O, allocation, and cache misses.

---

## Compile-Time Cost

Template-heavy designs can dramatically increase build times.

Developer build performance is part of software quality.

Prefer concrete internal implementations where genericity adds little.

---

## Avoid Header Bloat

Moving large implementation details into headers can multiply compile time.

Keep non-template implementations in source files unless inlining/header-only architecture is intentional.

---

## Avoid Excessive Includes

But do not engage in fragile include micro-optimization that makes files depend on transitive includes.

Correct dependencies first.

Optimize compile times systematically.

---

## Modules

If the project uses C++20 modules, follow its established toolchain.

Do not introduce modules into an ordinary header-based project during unrelated work.

Compiler/build support varies.

---

## Source Formatting

Use the project's formatter, usually clang-format.

Do not manually align large blocks if clang-format will undo the alignment.

Avoid whitespace-only churn.

---

## `clang-format`

Run the repository's configured format command.

Do not replace `.clang-format` with your personal style.

---

## Brace Style

Follow the existing project.

Common styles include:

```cpp
if (condition) {
}
```

or:

```cpp
if (condition)
{
}
```

Do not restyle unrelated code.

---

## Includes Ordering

Follow project or formatter conventions.

Do not reorder includes across entire files unless formatting tools do so intentionally.

---

## Comments and Formatting

Do not insert blank lines between every statement.

Generated C++ often becomes artificially spacious.

Keep logically related operations together.

---

## Avoid AI-Looking Architecture

Before adding any:

```text
Abstract
Base
Interface
Manager
Service
Provider
Handler
Factory
Builder
Processor
Strategy
Adapter
Repository
Facade
Coordinator
Registry
Context
Orchestrator
```

ask whether the type has a genuine responsibility requiring that abstraction.

Do not produce architecture by vocabulary.

---

## Avoid Enterprise OOP by Default

Do not transform:

```cpp
Weapon parseWeapon(const Data&);
```

into:

```text
IWeaponParser
WeaponParser
WeaponParserFactory
WeaponParserService
WeaponParserManager
```

without an actual architectural reason.

C++ supports abstraction; it does not require ceremony.

---

## Avoid Smart-Pointer Everywhere Style

Smart pointers are ownership tools, not generic pointer replacements.

Use:

```cpp
T
T&
const T&
T*
unique_ptr<T>
shared_ptr<T>
```

according to the actual ownership relationship.

---

## Avoid Modern C++ Feature Bingo

Do not add:

```text
concepts
ranges
coroutines
constexpr metaprogramming
variants
structured bindings
span
string_view
PMR
```

merely to make the code look modern.

Use a feature because it improves the specific code.

---

## Avoid Excessive Generic Programming

If a function serves one type, write it for that type.

Do not design internal application code as if it were a public generic library.

Genericity should be earned through real reuse.

---

## Avoid Manual Memory Management

If a generated implementation contains several:

```cpp
new
delete
malloc
free
```

review the design carefully.

In modern production C++, most ordinary application code should rely on RAII and value semantics.

---

## Avoid Premature Performance Tricks

Do not use:

```text
custom allocators
object pools
intrinsics
manual prefetching
branch hints
lock-free algorithms
bit hacks
placement new
```

without measured need.

Professional C++ is not defined by cleverness.

---

## Avoid Fake Safety Layers

Do not add redundant checks like:

```cpp
if (&weapon == nullptr)
```

for references.

Do not null-check objects whose types guarantee existence.

Let types express invariants.

---

## Avoid Redundant Copies to Simplify Ownership

AI-generated C++ often copies everything to avoid lifetime reasoning.

Before copying:

```cpp
auto local = expensiveObject;
```

ask whether:

```cpp
const auto& local = expensiveObject;
```

would express the correct lifetime.

Do not eliminate useful copies merely for purity.

---

## Avoid Returning Raw Owning Pointers

Bad:

```cpp
Weapon* createWeapon();
```

if caller owns deletion.

Prefer:

```cpp
std::unique_ptr<Weapon> createWeapon();
```

or simply:

```cpp
Weapon createWeapon();
```

depending on semantics.

---

## Avoid Hidden Ownership

APIs should make resource lifetime understandable without reading implementation details.

Do not rely on comments such as:

```cpp
// Caller must delete this.
```

when the type system can communicate ownership.

---

## Avoid Object Slicing

Do not pass/store polymorphic derived classes by value through base types if derived behavior must be preserved.

Use references/pointers or redesign the hierarchy.

---

## Avoid Base-Class Data Unless Appropriate

Polymorphic bases are often clearer when focused on interface/behavior rather than storing broad mutable state used by all subclasses.

Do not create "god base classes."

---

## Avoid God Objects

Classes should not own:

- Configuration.
- Networking.
- Filesystem.
- Logging.
- Rendering.
- Parsing.
- Caching.

all at once.

Split by meaningful responsibilities when complexity actually warrants it.

Do not split simple classes prematurely either.

---

## Avoid Friend Abuse

Use `friend` when a close relationship genuinely requires internal access.

Do not use `friend` to bypass poor encapsulation throughout the codebase.

Frequent friends can indicate boundaries are wrong.

---

## Avoid Public Mutable Globals

Do not expose writable global variables across translation units.

Use explicit ownership or narrow accessors when process-wide state is genuinely required.

---

## Avoid Preprocessor Configuration Where Runtime Works Better

Compile-time switches are useful for:

- Platforms.
- Optional dependencies.
- Build variants.

Do not use macros for ordinary user-configurable behavior.

Runtime configuration is often easier to test and maintain.

---

## Avoid Boolean Macros

Prefer:

```cpp
constexpr bool
```

inside C++ where compile-time language semantics suffice.

Use preprocessor macros only when compilation itself must change.

---

## Before Finishing

Review the change and remove or correct:

- Manual `new`/`delete`.
- Raw owning pointers.
- Unnecessary `shared_ptr`.
- Unnecessary heap allocation.
- Unnecessary copies.
- Unnecessary `std::move`.
- Dangling `string_view` or `span`.
- Redundant null checks.
- Excessive inheritance.
- Interfaces with one implementation.
- Unnecessary factories.
- Singleton patterns without justification.
- Excessive templates.
- Excessive concepts.
- Macro-based logic that normal C++ could express.
- Pass-through service layers.
- Generic AI-style names.
- Getter/setter boilerplate.
- Silent exception swallowing.
- C-style casts.
- Unchecked narrowing conversions.
- Debug output.
- Dead code.
- Unused includes.
- Placeholder TODOs.
- Speculative extensibility.
- Unrelated refactors.

Then run the project's established checks where available.

Typical C++ checks may include:

```bash
cmake --build build
ctest --test-dir build
```

along with tools such as:

```text
clang-format
clang-tidy
compiler warnings
AddressSanitizer
UndefinedBehaviorSanitizer
ThreadSanitizer
```

or project-specific scripts.

Do not assume these exact commands exist.

Inspect:

```text
CMakeLists.txt
CMakePresets.json
build scripts
CI configuration
.clang-format
.clang-tidy
repository documentation
```

and use the project's established workflow.

The final code should look like it naturally belongs in the repository rather than like a standalone AI-generated C++ solution.

It should feel like C++ written by an experienced maintainer: ownership is obvious, resources manage themselves, abstractions are earned, value semantics are preferred, lifetime is deliberate, modern features are used selectively, and cleverness never outranks clarity.