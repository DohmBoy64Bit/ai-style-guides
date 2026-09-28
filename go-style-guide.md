# Go Style Guide

Write Go as an experienced professional Go developer maintaining a real production codebase.

The goal is clean, idiomatic, maintainable Go—not code that looks generated, over-engineered, excessively abstract, or translated mechanically from Java, C#, TypeScript, or Rust.

The most important rule:

> Do not optimize for demonstrating architecture, patterns, or language features. Optimize for producing the smallest idiomatic production-quality change that an experienced Go maintainer would reasonably write.

## General Principles

Prefer:

- Simple code over clever code.
- Existing project conventions over personal preferences.
- Concrete types until abstraction is necessary.
- Small interfaces defined by consumers.
- Clear ownership of data and goroutines.
- Straightforward control flow.
- Explicit error handling.
- The standard library where it is sufficient.
- Composition over inheritance-like patterns.
- Small cohesive packages.
- Meaningful zero values.
- Value semantics where practical.
- Pointers only when semantics require them.
- Goroutines only when concurrency provides real value.
- Direct implementation over speculative extensibility.

Avoid:

- Enterprise layering without need.
- Interfaces for one implementation.
- Factories that construct one concrete type.
- Dependency-injection containers.
- Manager/service/provider abstractions with no real behavior.
- Goroutines added merely to make code concurrent.
- Channels used where a normal function call works.
- Context passed through everything without purpose.
- Pointer-heavy APIs by default.
- Generic helper packages.
- Exception-style error abstractions.
- Clever reflection or metaprogramming.
- Reimplementing standard-library functionality.

Go should feel boring in a good way.

---

## Match the Existing Codebase

Before writing code, inspect nearby files and follow the repository's established conventions for:

- Go version.
- Package layout.
- Naming.
- Error handling.
- Context usage.
- Logging.
- Concurrency.
- Testing.
- Dependency injection.
- Interfaces.
- Configuration.
- HTTP handling.
- Database access.
- Code generation.
- Formatting.
- Linting.
- Build tags.

Check relevant files such as:

```text
go.mod
go.sum
go.work
Makefile
Taskfile.yml
.golangci.yml
.golangci.yaml
Dockerfile
```

Consistency with the repository is more important than imposing a preferred architecture.

Do not reorganize packages, rename APIs, introduce interfaces, or rewrite error handling as unrelated cleanup.

---

## Go Version

Determine the project's supported Go version before using newer language or standard-library features.

Check:

```text
go.mod
CI configuration
Dockerfile
toolchain directives
```

For example:

```go
go 1.24
```

Do not assume the project supports the latest Go release.

Avoid introducing APIs or syntax unavailable to the declared toolchain.

---

## Naming

Follow Go naming conventions.

Use:

- `camelCase` for unexported identifiers.
- `PascalCase` for exported identifiers.
- Short names for small local scopes.
- Descriptive names for broader scopes.
- Acronyms consistently.

Good:

```go
weapon
assetPath
packageName
loadResult

loadPackage
resolveAsset
parseWeaponData
```

Avoid generic AI-style names:

```go
dataManager
processHandler
utilityHelper
genericService
resultProcessor
operationManager
```

Prefer domain language.

Bad:

```go
data := getData()
result := processData(data)
```

Better:

```go
weapon := loadWeaponDefinition()
stats := parseWeaponStats(weapon)
```

Do not make names excessively verbose.

Avoid:

```go
successfullyParsedWeaponConfigurationResult
```

when:

```go
weaponConfig
```

is clear.

---

## Short Names

Go often favors short names in tight scope.

Good:

```go
for _, weapon := range weapons {
    if weapon.Enabled {
        active = append(active, weapon)
    }
}
```

Also good in a very small loop:

```go
for _, w := range weapons {
    total += w.Damage
}
```

Do not use cryptic single-letter names across large functions.

Scope should determine naming length.

---

## Acronyms

Follow common Go acronym conventions.

Prefer:

```go
ID
URL
HTTP
JSON
API
UUID
```

Examples:

```go
weaponID
HTTPClient
parseJSON
apiURL
```

Avoid:

```go
weaponId
HttpClient
JsonParser
ApiUrl
```

unless the existing codebase consistently uses that style.

---

## Package Names

Package names should be:

- Short.
- Lowercase.
- Singular when practical.
- Descriptive without repeating imported identifiers.

Good:

```go
package weapon
package cache
package manifest
```

Avoid:

```go
package weaponmanager
package weaponutils
package commonhelpers
```

Do not use names like:

```text
util
utils
common
misc
helpers
```

unless the package genuinely represents a stable cohesive concept.

---

## Avoid Package Name Repetition

Avoid APIs like:

```go
weaponmanager.NewWeaponManager()
```

Prefer:

```go
weapon.NewManager()
```

if a manager is genuinely needed.

Better still, ask whether `Manager` is needed at all.

Package context should carry part of the meaning.

---

## Exported Names

Export only what callers need.

Use:

```go
type Loader struct
```

only if callers outside the package need the type.

Otherwise keep it unexported:

```go
type loader struct
```

A small public API is easier to maintain.

Do not export helpers "just in case."

---

## Comments on Exported Identifiers

Follow Go documentation conventions when exported APIs require comments.

Good:

```go
// Loader loads weapon packages from disk.
type Loader struct {
    // ...
}
```

Do not generate verbose documentation for obvious internal code.

Comments should begin with the exported identifier when required by tooling and style.

Avoid:

```go
// This struct is responsible for loading weapon packages.
type Loader struct {
}
```

Prefer concise technical wording.

---

## Functions First

Use functions for stateless operations.

Good:

```go
func parseWeaponStats(data WeaponData) WeaponStats {
    return WeaponStats{
        Damage:   data.Damage,
        FireRate: data.FireRate,
    }
}
```

Do not automatically create:

```go
type WeaponStatsParser struct{}

func (p WeaponStatsParser) Parse(data WeaponData) WeaponStats {
    // ...
}
```

unless the parser owns actual state or dependencies.

Go packages already provide namespacing.

---

## Structs

Use structs for cohesive data and behavior.

Good:

```go
type PackageCache struct {
    packages map[string]Package
}
```

Do not create a struct solely to hold one stateless method.

Avoid:

```go
type StringUtils struct{}
```

with methods like:

```go
func (StringUtils) Normalize(...)
```

Prefer normal package functions.

---

## Zero Values

Design useful zero values when practical.

Good:

```go
var buf bytes.Buffer
```

works immediately.

Where sensible, prefer types that do not require ceremonial initialization.

Do not force zero-value support when the type cannot be valid without required configuration.

Use constructors when invariants genuinely require them.

---

## Constructors

Go constructors are ordinary functions, usually named:

```go
New
NewClient
NewLoader
```

Use constructors when initialization:

- Establishes invariants.
- Requires dependencies.
- Sets non-obvious defaults.
- Allocates required internal structures.

Do not create constructors for trivial structs that can be initialized clearly with a literal.

This may be enough:

```go
weapon := Weapon{
    ID:   id,
    Name: name,
}
```

rather than:

```go
weapon := NewWeapon(id, name)
```

if `NewWeapon` adds no behavior.

---

## `New`

Within a focused package:

```go
weapon.New(...)
```

is often preferable to:

```go
weapon.NewWeapon(...)
```

because the package already supplies context.

Follow existing project style.

---

## Value vs Pointer Semantics

Use values when:

- The type is small.
- Copying is natural.
- Identity does not matter.
- Mutation is not required.

Use pointers when:

- Mutation is intended.
- The value is large enough that copying matters.
- Identity matters.
- Nil is meaningful.
- Internal synchronization should not be copied.

Do not use pointers simply because the value is a struct.

---

## Pointer Parameters

Avoid:

```go
func ProcessWeapon(weapon *Weapon)
```

when the function only reads a small value and nil is not meaningful.

Prefer:

```go
func ProcessWeapon(weapon Weapon)
```

or:

```go
func ProcessWeapon(weapon *Weapon)
```

only when pointer semantics are actually needed.

Do not blindly apply either rule.

---

## Pointer Returns

Do not return pointers merely to avoid copying tiny structs.

For small immutable-like results:

```go
func Stats() WeaponStats
```

may be clearer than:

```go
func Stats() *WeaponStats
```

Use pointers when shared identity or mutation matters.

---

## Nil

Nil is useful for:

- Optional pointers.
- Maps.
- Slices.
- Channels.
- Functions.
- Interfaces.

Do not use nil as an ambiguous catch-all state.

Be especially careful with nil interfaces.

---

## Nil Interfaces

Understand this distinction:

```go
var err error = nil
```

is nil.

But:

```go
var ptr *MyError
var err error = ptr
```

is not a nil interface even if `ptr == nil`.

Do not return typed nil pointers through interfaces accidentally.

This is a real correctness issue.

---

## Methods

Define methods when behavior naturally belongs to a type.

Good:

```go
func (w Weapon) Enabled() bool {
    return w.Status == StatusEnabled
}
```

Use a free function when the behavior is broader orchestration or transformation.

Do not attach every helper to a type simply because methods look organized.

---

## Receiver Names

Use short, consistent receiver names.

Good:

```go
func (w Weapon) Name() string
func (c *Cache) Get(key string)
```

Avoid:

```go
func (weaponInstance Weapon) Name() string
```

Receiver names should generally be one or two letters and consistent across methods.

Do not use `this` or `self`.

---

## Pointer vs Value Receivers

Use pointer receivers when methods:

- Mutate the receiver.
- Should not copy the receiver.
- Operate on a type containing synchronization primitives.
- Need consistent method-set semantics.

Use value receivers for small immutable-like values.

Do not mix pointer and value receivers arbitrarily on the same type.

Prefer consistency unless semantics clearly differ.

---

## Do Not Copy Mutexes

Types containing:

```go
sync.Mutex
sync.RWMutex
sync.Once
atomic types
```

should generally use pointer receivers and should not be copied after use.

Pay attention to `go vet` copylock warnings.

---

## Interfaces

One of the most important Go rules:

> Define interfaces where they are consumed, not where implementations are created.

Prefer small consumer-owned interfaces.

Good:

```go
type packageLoader interface {
    Load(ctx context.Context, path string) (Package, error)
}
```

inside the package that needs loading behavior.

Avoid exporting giant provider interfaces from implementation packages.

---

## Interfaces Should Be Small

Good:

```go
type Reader interface {
    Read([]byte) (int, error)
}
```

Go's standard library demonstrates the value of tiny behavioral interfaces.

Avoid:

```go
type WeaponRepository interface {
    Create(...)
    Update(...)
    Delete(...)
    Find(...)
    FindAll(...)
    FindByCategory(...)
    Search(...)
    Count(...)
    Exists(...)
    Transaction(...)
}
```

unless every consumer truly requires that entire contract.

Split interfaces by consumer need.

---

## Do Not Create Interfaces Prematurely

Bad:

```go
type WeaponParser interface {
    Parse(data []byte) (*Weapon, error)
}

type DefaultWeaponParser struct{}
```

when there is exactly one implementation and no caller requires abstraction.

Prefer:

```go
type Parser struct{}
```

or simply:

```go
func ParseWeapon(data []byte) (Weapon, error)
```

Add the interface when a consumer actually needs one.

---

## Accept Interfaces, Return Concrete Types

This Go guideline is often useful:

> Accept interfaces when abstraction benefits the caller; return concrete types when callers benefit from knowing what they receive.

Do not interpret this as an absolute law.

Avoid returning an interface purely to hide a concrete type without a real reason.

---

## Empty Interface

Modern Go uses:

```go
any
```

instead of:

```go
interface{}
```

when arbitrary values are genuinely required.

Avoid `any` when a useful concrete type or generic constraint exists.

Dynamic data should stay near actual dynamic boundaries.

---

## Type Assertions

Use type assertions when working with genuinely dynamic interfaces.

Prefer the safe form:

```go
value, ok := input.(Weapon)
if !ok {
    // handle mismatch
}
```

Do not scatter type assertions throughout typed application code.

If everything requires assertions, the type model is probably too weak.

---

## Type Switches

Use type switches for intentionally heterogeneous interface values.

Do not build object-oriented class hierarchies around `any` and type switches when normal typed APIs would be clearer.

---

## Generics

Use generics when code genuinely works across multiple types and the abstraction remains simple.

Good examples may include:

- Generic containers.
- Reusable algorithms.
- Common transformation helpers.

Do not genericize domain-specific logic.

Bad:

```go
func ParseEntity[TInput any, TOutput any](input TInput) TOutput
```

for one weapon parser.

Concrete code is often better Go.

---

## Generic Constraints

Keep constraints simple.

Do not build elaborate type-set hierarchies for ordinary application code.

If the generic signature takes longer to understand than several concrete functions would, reconsider the abstraction.

---

## Avoid Generic Utility Packages

Do not create a giant package of generic helpers such as:

```text
Map
Filter
Reduce
Contains
Transform
Convert
```

merely to make Go resemble another language.

Use generics when they genuinely improve the project, not to recreate a functional standard library.

---

## Composition

Go favors composition.

Embed or hold collaborators rather than trying to simulate inheritance.

Good:

```go
type Loader struct {
    fs FileSystem
}
```

Do not build base-type hierarchies.

---

## Embedding

Use embedding when promoted methods or structural composition genuinely improve the API.

Example:

```go
type Server struct {
    *http.Server
}
```

may be appropriate in some designs.

Do not embed merely to avoid writing field names.

Embedding changes the public method set and can expose more API than intended.

---

## Avoid Fake Inheritance

Do not use embedding to recreate Java-style inheritance.

Bad conceptual design:

```go
type BaseProcessor struct {
    // ...
}

type WeaponProcessor struct {
    BaseProcessor
}
```

solely to inherit helper behavior.

Use composition or plain functions.

---

## Errors

Return errors as normal values.

Use the standard pattern:

```go
value, err := loadWeapon(path)
if err != nil {
    return Weapon{}, err
}
```

Do not invent exception-style mechanisms.

Explicit error handling is a feature of Go.

---

## Error Context

Wrap errors when adding meaningful context.

Good:

```go
data, err := os.ReadFile(path)
if err != nil {
    return nil, fmt.Errorf("read weapon file %q: %w", path, err)
}
```

Avoid vague layers:

```go
return nil, fmt.Errorf("failed to process data: %w", err)
```

Prefer domain-specific context.

---

## `%w`

Use `%w` when callers may need to inspect the underlying error.

Example:

```go
fmt.Errorf("load package %q: %w", path, err)
```

Do not use `%v` when wrapping semantics are intended.

---

## Error Strings

Follow Go convention:

- Lowercase.
- No trailing punctuation in most composable errors.

Good:

```go
errors.New("weapon ID is required")
```

Avoid:

```go
errors.New("Weapon ID is required.")
```

unless producing a final user-facing message rather than a composable error.

---

## Sentinel Errors

Use sentinel errors when callers genuinely need identity-based matching.

Example:

```go
var ErrNotFound = errors.New("weapon not found")
```

Then:

```go
if errors.Is(err, ErrNotFound) {
    // ...
}
```

Do not create sentinel errors for every possible message.

---

## Custom Error Types

Use custom error types when callers need structured error data.

Example:

```go
type ParseError struct {
    Path string
    Line int
    Err  error
}

func (e *ParseError) Error() string {
    return fmt.Sprintf("parse %q at line %d: %v", e.Path, e.Line, e.Err)
}

func (e *ParseError) Unwrap() error {
    return e.Err
}
```

Do not create a massive error hierarchy.

Go errors are intentionally lightweight.

---

## `errors.Is`

Use:

```go
errors.Is(err, target)
```

for wrapped sentinel errors.

Do not compare wrapped errors directly:

```go
if err == ErrNotFound
```

unless no wrapping can occur and that is intentional.

---

## `errors.As`

Use:

```go
errors.As
```

when callers need a specific error type.

Do not parse error strings.

Bad:

```go
if strings.Contains(err.Error(), "not found") {
```

Structured errors are safer.

---

## Do Not Return Error Strings as Data

Avoid:

```go
return "error: package missing", nil
```

when the operation actually failed.

Use the error return.

Do not overload success values to encode failure.

---

## Avoid Custom Result Wrappers

Do not create:

```go
type Result[T any] struct {
    Value   T
    Success bool
    Err     error
}
```

for ordinary Go APIs.

Go already has:

```go
value, err
```

Use result structs only when there are several meaningful non-error states requiring structured data.

---

## Error Handling Boundaries

Handle errors where you can:

- Recover.
- Add meaningful context.
- Convert them to protocol responses.
- Log them at the application boundary.

Do not log and return the same error at every layer.

---

## Logging Errors

Avoid:

```go
log.Printf("load failed: %v", err)
return err
```

at every internal layer.

This creates duplicate logs.

Usually, internal code should add context and return.

A boundary such as:

- HTTP handler.
- CLI command.
- Worker loop.

can decide how to log.

---

## Panic

Use panic for:

- Programmer errors.
- Impossible initialization conditions.
- Truly unrecoverable internal invariants.

Do not panic for ordinary runtime failures.

Bad:

```go
if err != nil {
    panic(err)
}
```

inside a reusable library operation.

Return the error.

---

## `Must` Helpers

A `MustX` helper can be appropriate for package initialization or tests where failure truly means the program cannot proceed.

Example:

```go
var pattern = regexp.MustCompile(`...`)
```

Do not create `Must` versions of ordinary runtime operations merely for convenience.

---

## Recover

Use `recover` only at carefully chosen boundaries.

Examples:

- Server middleware protecting the process.
- Plugin boundaries.
- Framework internals.

Do not use `recover` as normal error handling.

Do not silently swallow panics.

---

## Defer

Use `defer` for cleanup tied to scope.

Good:

```go
file, err := os.Open(path)
if err != nil {
    return err
}
defer file.Close()
```

Defer makes resource ownership clear.

---

## Check Cleanup Errors When They Matter

Do not always ignore errors from:

```go
file.Close()
writer.Flush()
response.Body.Close()
```

If finalization can affect correctness, handle it.

For read-only cleanup, ignoring `Close` may be reasonable depending on the resource.

Use domain judgment.

---

## Defer in Loops

Be careful with:

```go
for _, path := range paths {
    file, _ := os.Open(path)
    defer file.Close()
}
```

because defers execute when the surrounding function returns, not at the end of each iteration.

For many resources, use a helper function or explicit close per iteration.

---

## Context

Use `context.Context` for request-scoped:

- Cancellation.
- Deadlines.
- Timeouts.
- Cross-API request metadata where conventionally appropriate.

Do not use context as a generic bag of dependencies or configuration.

---

## Context First

By convention, functions accepting a context usually take it first:

```go
func Load(ctx context.Context, path string) (Package, error)
```

Use the name:

```go
ctx
```

Do not store context permanently in structs in most application designs.

Pass it through call chains where request lifetime matters.

---

## Do Not Pass Nil Context

Use:

```go
context.Background()
```

or:

```go
context.TODO()
```

when an explicit root is necessary.

Do not pass nil contexts.

---

## Do Not Put Dependencies in Context

Bad:

```go
ctx = context.WithValue(ctx, repositoryKey, repository)
```

to avoid passing dependencies.

Context is not a service locator.

Pass repositories, clients, loggers, or config explicitly unless a framework has a documented context mechanism.

---

## Context Values

Use context values only for request-scoped metadata that legitimately crosses API boundaries.

Examples may include:

- Request IDs.
- Authentication claims.
- Trace information.

Use private key types.

Do not use raw string keys.

---

## Cancellation

If a function accepts context and performs cancellable work, propagate it.

Bad:

```go
func Load(ctx context.Context, url string) error {
    req, _ := http.NewRequest(http.MethodGet, url, nil)
    // ctx ignored
}
```

Better:

```go
req, err := http.NewRequestWithContext(
    ctx,
    http.MethodGet,
    url,
    nil,
)
```

Do not accept context purely for appearance if it cannot affect the operation.

---

## Timeouts

Prefer caller-owned timeouts where possible.

A caller may do:

```go
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()
```

Libraries may also define transport-level safety timeouts where necessary.

Do not stack arbitrary timeout layers without understanding which boundary owns policy.

---

## Goroutines

Do not add goroutines simply because Go makes concurrency easy.

Before writing:

```go
go doWork()
```

ask:

- Who owns this goroutine?
- When does it stop?
- How are errors reported?
- Can it leak?
- Does the caller need completion?
- Is concurrency actually beneficial?

Every goroutine needs a lifecycle story.

---

## Fire-and-Forget Goroutines

Avoid untracked fire-and-forget goroutines in servers and libraries.

Bad:

```go
go saveAnalytics(data)
```

if failure, shutdown, or lifecycle matters.

Use a worker system, errgroup, queue, or explicit ownership when necessary.

---

## Goroutine Leaks

Common leak causes include:

- Blocked channel sends.
- Blocked receives.
- Forgotten cancellation.
- Infinite loops.
- Network calls without timeout.
- Workers with no shutdown path.

Design termination before starting the goroutine.

---

## Channels

Use channels for communication and synchronization between goroutines.

Do not use channels as a replacement for:

- Normal function calls.
- Simple return values.
- Ordinary mutex-protected state.

Channels are not automatically more idiomatic than locks.

---

## Channel Direction

Use directional channel types when they clarify APIs:

```go
func produce(out chan<- Item)
func consume(in <-chan Item)
```

Do not add direction annotations where they make internal code noisier without benefit.

---

## Channel Ownership

The sender that creates/owns production is usually responsible for closing the channel.

Receivers generally should not close channels they did not create.

Do not close channels merely because a consumer is done.

---

## Never Send on Closed Channels

Design ownership clearly enough that channel closure is not ambiguous.

Avoid recover-based handling of send-on-closed-channel panics.

Fix the lifecycle.

---

## Nil Channels

Remember nil channels block forever on send and receive.

This can be useful inside select-based state machines, but it is also a common bug.

Use intentionally.

---

## Buffered Channels

Use buffering when it matches actual producer/consumer behavior.

Do not add arbitrary buffer sizes like:

```go
make(chan Item, 100)
```

just to avoid blocking.

Choose capacity based on real semantics or leave it unbuffered.

---

## Select

Use `select` for coordinating multiple channel operations or cancellation.

Good:

```go
select {
case item := <-items:
    return item, nil
case <-ctx.Done():
    return Item{}, ctx.Err()
}
```

Do not use `select` where a direct receive is sufficient.

---

## Default Case

Be cautious with:

```go
select {
case ...
default:
}
```

A default makes the select non-blocking and can create busy loops.

Do not add default branches mechanically.

---

## Worker Pools

Use worker pools when:

- Work can run concurrently.
- Concurrency must be bounded.
- Input may be large.

Do not build a worker pool for ten trivial operations.

Prefer simpler concurrency when enough.

---

## `errgroup`

When the project uses `golang.org/x/sync/errgroup`, it can simplify groups of related goroutines with cancellation/error propagation.

Do not add the dependency for a trivial pair of goroutines if plain code is clearer.

Follow existing dependency conventions.

---

## WaitGroup

Use `sync.WaitGroup` when waiting for a known group of goroutines.

Ensure:

- `Add` occurs before the goroutine can call `Done`.
- Every started task eventually calls `Done`.

Prefer:

```go
defer wg.Done()
```

inside simple workers.

---

## Mutex

Use `sync.Mutex` for shared mutable state when it is the clearest solution.

Do not avoid locks merely because "share memory by communicating" is a slogan.

A mutex around a map can be simpler than an elaborate channel-owned state machine.

---

## RWMutex

Use `sync.RWMutex` only when the workload benefits from concurrent readers.

Do not assume it is automatically better than `Mutex`.

For short critical sections, normal `Mutex` may be simpler and faster.

---

## Keep Locks Small

Do not hold locks during:

- Network calls.
- Disk I/O.
- Expensive computation.
- User callbacks.

unless the invariant requires it.

Copy needed state and release the lock where possible.

---

## Atomic Operations

Use `sync/atomic` for simple atomic state when appropriate.

Do not implement complex lock-free algorithms casually.

Locks are often clearer and sufficiently fast.

---

## `sync.Once`

Use `sync.Once` for genuinely once-only concurrent initialization.

Do not use it merely to implement hidden global mutable state.

---

## Concurrency Ownership

Prefer designs where one component clearly owns mutable state.

The easiest race to prevent is one that cannot exist because state ownership is obvious.

Do not expose mutable internal maps/slices across goroutines without synchronization.

---

## Race Detector

For concurrent changes, run the race detector when practical:

```text
go test -race ./...
```

Do not assume successful normal tests prove concurrency safety.

---

## Slices

Use slices as the default sequence type.

Understand that slices are descriptors over shared backing arrays.

Passing a slice does not copy its elements.

This matters for mutation.

---

## Nil vs Empty Slices

Both can be valid:

```go
var weapons []Weapon
```

and:

```go
weapons := []Weapon{}
```

They behave similarly for most operations but differ in cases such as JSON output.

Follow API expectations and project conventions.

Do not normalize one to the other without a reason.

---

## Append

Use:

```go
items = append(items, value)
```

normally.

Do not manually manage capacity unless performance or expected size makes it useful.

---

## Preallocation

When size is known, this may be appropriate:

```go
items := make([]Weapon, 0, len(records))
```

Do not estimate capacities everywhere.

Avoid premature micro-optimization.

---

## Slice Aliasing

Be aware that subslices may retain large backing arrays.

This:

```go
small := large[:10]
```

may keep the entire large allocation alive.

Copy only when retention matters.

Do not copy every subslice defensively.

---

## Returning Internal Slices

If callers must not mutate internal state, consider:

- Returning a copy.
- Returning an iterator-like operation.
- Documenting ownership clearly.

Do not expose mutable internals accidentally.

Do not copy automatically when mutation is intentionally allowed.

---

## Maps

Use maps for key-value lookup.

Remember:

- Reading a nil map is safe.
- Writing to a nil map panics.
- Map iteration order is unspecified.

Initialize before writing.

---

## Comma-Ok Idiom

Use:

```go
weapon, ok := weapons[id]
if !ok {
    // missing
}
```

when absence matters.

Do not compare against zero values if the zero value can be a valid stored value.

---

## Map Iteration Order

Never rely on map iteration order.

If deterministic output matters:

1. Collect keys.
2. Sort them.
3. Iterate in sorted order.

This is especially important for tests, serialization, and generated output.

---

## Sets

Go has no built-in set type.

Use:

```go
map[string]struct{}
```

when a set is actually useful.

Do not create a generic custom set package for one usage unless the project already has one or generics make repeated use worthwhile.

---

## `struct{}`

Use:

```go
map[string]struct{}
```

for sets when value storage is unnecessary.

A `map[string]bool` is also sometimes clearer if boolean presence semantics are useful.

Follow project style.

---

## Strings

Use strings for UTF-8 text.

Remember indexing a string returns a byte:

```go
s[0]
```

not a rune.

Do not assume byte indexing equals character indexing.

---

## Runes

Use `rune` when operating on Unicode code points.

Example:

```go
for _, r := range text {
    // r is a rune
}
```

Do not use rune-based processing when the input is deliberately raw bytes.

---

## Grapheme Clusters

A rune is not necessarily a user-perceived character.

For complex user-visible text operations, a Unicode segmentation library may be necessary.

Do not introduce grapheme handling for ASCII-only protocols.

---

## `strings.Builder`

Use `strings.Builder` for repeated string construction when appropriate.

Do not use it for two concatenations where:

```go
a + b
```

is clearer.

---

## `bytes.Buffer`

Use `bytes.Buffer` for accumulated bytes/string-like output, especially where APIs already operate on bytes.

Choose the simplest tool.

---

## String Formatting

Use:

```go
fmt.Sprintf
```

when formatting is genuinely needed.

Do not use `fmt.Sprintf` for simple concatenation if direct operations are clearer.

For errors, `fmt.Errorf` is usually appropriate.

---

## String Comparisons

Use direct equality for exact string comparison.

For case-insensitive Unicode-aware comparisons, `strings.EqualFold` is usually preferable to lowercasing both strings.

Avoid unnecessary allocations.

---

## `[]byte` Conversion

Do not repeatedly convert between:

```go
string
[]byte
```

without a reason.

Conversions allocate in normal cases.

In actual hot paths, choose APIs that minimize unnecessary conversions.

Do not optimize trivial paths prematurely.

---

## Struct Fields

Export fields only when callers need direct access.

Use unexported fields to protect invariants.

Do not create getters for every field if direct access is appropriate.

---

## Getters

Go convention generally avoids `Get` prefixes.

Prefer:

```go
func (w Weapon) Name() string
```

over:

```go
func (w Weapon) GetName() string
```

unless matching an external API or project convention.

---

## Setters

Use setters only when mutation is genuinely part of the API.

Prefer meaningful domain operations where appropriate.

Instead of:

```go
func (w *Weapon) SetStatus(status Status)
```

a method like:

```go
func (w *Weapon) Disable()
```

may better express an invariant.

Do not create setters automatically.

---

## Struct Literals

Use keyed fields for important structs:

```go
weapon := Weapon{
    ID:     id,
    Name:   name,
    Damage: damage,
}
```

Avoid positional struct literals across package boundaries.

Keyed fields are more robust to field reordering and clearer to readers.

---

## Unexported Positional Literals

Within tiny internal structs, positional literals can be acceptable if they remain obvious.

Follow project style.

Do not use positional literals merely to save characters.

---

## Embedding Interfaces

Avoid embedding large interfaces into new interfaces without understanding the resulting contract.

Small interface composition can be useful.

Do not build giant "god interfaces."

---

## Standard Library Interfaces

Reuse standard interfaces where appropriate:

```go
io.Reader
io.Writer
io.Closer
fmt.Stringer
http.Handler
```

Do not create:

```go
type CustomReader interface {
    Read([]byte) (int, error)
}
```

when `io.Reader` already describes the behavior.

---

## `io.Reader`

Prefer stream-oriented interfaces when data may be large or already arrives as a stream.

Do not convert everything to `[]byte` merely for convenience if streaming matters.

For small files or payloads, whole-buffer APIs may still be simpler.

---

## `io.Writer`

Accepting `io.Writer` can make output APIs composable.

Example:

```go
func Encode(w io.Writer, weapon Weapon) error
```

Do not introduce writer-based APIs if all callers naturally need a returned byte slice and data is tiny.

---

## File I/O

Use standard library functions where appropriate.

For small files:

```go
data, err := os.ReadFile(path)
```

may be clearer than manually opening and buffering.

For large/streamed data, use `os.Open` and readers.

Choose based on actual workload.

---

## File Permissions

When creating files, use permissions deliberately.

Do not cargo-cult:

```go
0644
0755
```

without understanding whether the file should be executable or private.

Sensitive files may require stricter permissions.

---

## Paths

Use:

```go
path/filepath
```

for filesystem paths.

Use:

```go
path
```

for slash-separated logical paths such as URLs.

Do not confuse them.

Good:

```go
filepath.Join(root, "config", "settings.json")
```

for local filesystem paths.

---

## Clean vs Join

Use path-cleaning functions when normalization is actually needed.

Be careful with untrusted paths confined to a root; simple `filepath.Clean` does not automatically prevent traversal outside the root.

Apply security based on actual threat model.

---

## Temporary Files

Use:

```go
os.CreateTemp
os.MkdirTemp
```

rather than inventing predictable temporary names.

Remember to clean up when appropriate.

---

## JSON

Use `encoding/json` unless the project has an established alternative or performance requirement.

Do not add another JSON dependency casually.

Use struct tags deliberately:

```go
type Weapon struct {
    ID   string `json:"id"`
    Name string `json:"name"`
}
```

---

## JSON Struct Tags

Do not add tags that simply repeat defaults unless they are part of an external contract.

Use:

```go
json:"weapon_id"
```

when external naming differs.

Be cautious with:

```go
omitempty
```

because omission semantics can differ from zero-value semantics.

---

## `omitempty`

Do not use it mechanically on every field.

Ask whether consumers need to distinguish:

- Missing.
- Empty.
- Zero.
- False.

API compatibility matters.

---

## Unknown JSON Fields

By default, `encoding/json` ignores unknown object fields.

For strict configuration or protocol inputs, you may use:

```go
decoder.DisallowUnknownFields()
```

when appropriate.

Do not make all inputs strict automatically; forward compatibility may matter.

---

## JSON Numbers

Remember decoding arbitrary JSON into:

```go
map[string]any
```

typically yields `float64` for numbers unless configured otherwise.

Prefer typed structs where schema is known.

Do not spread loosely typed JSON maps throughout domain logic.

---

## YAML

Use the project's established YAML library.

Be cautious about loose decoding and configuration defaults.

Normalize and validate config at the boundary.

Do not maintain several config representations without need.

---

## Configuration

Load configuration near startup or application boundaries.

Avoid scattered:

```go
os.Getenv(...)
```

throughout domain packages.

Prefer a validated config struct passed to components that need it.

Do not build a huge configuration framework for three values.

---

## Environment Variables

Parse explicitly.

Bad conceptual behavior:

```go
debug := os.Getenv("DEBUG") != ""
```

because:

```text
DEBUG=false
```

would still enable it.

Use:

```go
strconv.ParseBool
```

or the project's config library.

---

## Defaults

Defaults should be intentional.

Do not silently default malformed configuration values.

Missing and invalid are not always the same.

Validate required configuration clearly.

---

## HTTP Clients

Reuse HTTP clients.

Bad:

```go
func fetch(url string) {
    client := &http.Client{}
    // ...
}
```

for every request.

Clients are safe for concurrent use and benefit from connection reuse.

Configure timeouts appropriately.

---

## `http.DefaultClient`

Be cautious with the default client because it has no overall timeout.

In long-running services, use an intentionally configured client where hanging requests would be harmful.

Do not create arbitrary tiny timeouts.

---

## HTTP Requests

Use context-aware requests:

```go
req, err := http.NewRequestWithContext(
    ctx,
    http.MethodGet,
    url,
    nil,
)
```

when request cancellation matters.

Always handle request-construction errors.

---

## HTTP Responses

Always close response bodies:

```go
resp, err := client.Do(req)
if err != nil {
    return err
}
defer resp.Body.Close()
```

Check status codes deliberately.

Do not decode a 500 response as if it were successful data.

---

## HTTP Status Codes

Handle expected status codes explicitly.

Example:

```go
if resp.StatusCode == http.StatusNotFound {
    return ErrNotFound
}

if resp.StatusCode < 200 || resp.StatusCode >= 300 {
    return fmt.Errorf("request failed: %s", resp.Status)
}
```

Do not assume every non-network-error request succeeded.

---

## HTTP Servers

Prefer standard:

```go
http.Handler
http.HandlerFunc
```

and project-established routers.

Do not build custom routing abstractions without need.

---

## Handlers

HTTP handlers should generally:

- Parse request input.
- Validate boundary data.
- Authorize.
- Call application/domain logic.
- Write response.

Avoid burying large business workflows directly in handlers.

But do not create five layers for a simple endpoint.

---

## Response Errors

Centralized error-to-HTTP mapping can be useful.

Do not invent giant internal response wrapper objects merely because the API returns JSON.

Keep domain errors separate from HTTP representation where useful.

---

## Middleware

Use middleware for true cross-cutting HTTP concerns:

- Authentication.
- Logging.
- Tracing.
- Recovery.
- Request IDs.
- CORS.

Do not put endpoint-specific business logic in generic middleware.

---

## Database Code

Use the project's existing database layer.

Possible approaches include:

- `database/sql`.
- sqlc.
- GORM.
- Ent.
- Bun.
- Custom query packages.

Do not introduce a repository abstraction automatically over every ORM/query library.

---

## `database/sql`

Understand that `sql.DB` is a connection pool, not a single connection.

Reuse it.

Do not open a new database handle per request.

---

## Context in Queries

For request-scoped operations, use context-aware methods:

```go
QueryContext
ExecContext
BeginTx
```

when cancellation/deadlines matter.

Do not accept context if the database operation ignores it.

---

## Transactions

Keep transactions explicit and small.

Typical pattern:

```go
tx, err := db.BeginTx(ctx, nil)
if err != nil {
    return err
}

defer tx.Rollback()

// operations

if err := tx.Commit(); err != nil {
    return err
}
```

A deferred rollback after successful commit is typically harmless and simplifies cleanup.

Follow project conventions.

---

## Do Not Hold Transactions Across Network Calls

Avoid long database transactions while waiting on unrelated remote systems.

This can increase lock contention and failure complexity.

Use an intentional workflow if cross-system consistency is required.

---

## SQL Injection

Use query parameters.

Never concatenate untrusted values directly into SQL.

Do not build table/column identifiers from user input without strict allowlists because placeholders do not parameterize identifiers.

---

## Repositories

Repository abstractions can be useful when they represent a meaningful persistence boundary.

Do not create:

```go
type WeaponRepository interface {
    Create(...)
    Update(...)
    Delete(...)
    Find(...)
}
```

automatically merely because the application touches a database.

Let actual consumers define the methods they need.

---

## Dependency Injection

Go usually needs nothing more than constructors and fields.

Good:

```go
type Loader struct {
    repo Repository
}

func NewLoader(repo Repository) *Loader {
    return &Loader{repo: repo}
}
```

Do not add a DI container for ordinary projects.

Explicit wiring is a strength.

---

## Main Wiring

It is normal for `main` or startup code to wire dependencies explicitly.

Do not hide all construction behind reflection or service locators merely to make startup look smaller.

Boring startup code is often healthy Go.

---

## Functional Options

Use the functional-options pattern when:

- Construction has several optional settings.
- Defaults are meaningful.
- Backward-compatible extension is useful.

Example:

```go
client := NewClient(
    WithTimeout(5*time.Second),
    WithLogger(logger),
)
```

Do not use functional options for a constructor with one or two required values.

They add indirection.

---

## Option Explosion

Avoid dozens of:

```go
WithFoo
WithBar
WithBaz
```

for internal types.

If configuration is naturally data, a config struct may be clearer.

---

## Config Structs

Use config structs when multiple related configuration values belong together.

Example:

```go
type Config struct {
    Timeout time.Duration
    Retries int
}
```

Do not create a config object for one string parameter.

---

## Factories

Use factory functions when construction selects among genuinely different implementations.

Do not create:

```go
WeaponFactory
```

as a type solely to call `NewWeapon`.

Go constructor functions are usually enough.

---

## Registries

Use registries for truly extensible plugin/handler systems.

Do not create a registry containing one implementation.

A switch or map may be clearer for a small fixed set.

---

## Reflection

Avoid `reflect` unless the problem is genuinely dynamic.

Reflection can be appropriate for:

- Serialization frameworks.
- Generic infrastructure.
- Dependency tooling.
- Schema inspection.

Do not use reflection to avoid writing straightforward typed code.

---

## Unsafe

Avoid `unsafe` in ordinary application code.

Use it only when:

- Interacting with low-level APIs.
- Performance is measured and significant.
- Standard APIs cannot solve the problem.

Keep unsafe code narrow and documented.

---

## CGO

Use cgo when integration with C libraries genuinely requires it.

Be aware of:

- Build complexity.
- Cross-compilation impact.
- Pointer rules.
- Scheduler transitions.
- Deployment dependencies.

Do not use cgo for functionality the standard library or pure-Go libraries handle well.

---

## Defer and Mutexes

This can be clear:

```go
mu.Lock()
defer mu.Unlock()
```

for short functions.

In hot paths or large functions, explicit unlock may make lock scope clearer.

Do not sacrifice understandable lock lifetime for micro-optimization without evidence.

---

## `sync.Pool`

Use `sync.Pool` only for measured allocation-heavy temporary objects.

Do not treat it as a general object cache.

Pool contents may disappear at any time.

---

## Object Pools

Avoid manual object pooling unless allocation pressure is proven to matter.

Go's garbage collector is designed for normal allocation patterns.

Measure first.

---

## Memory Ownership

Go has garbage collection, but ownership still matters conceptually.

Clarify which component:

- Mutates data.
- Shares data.
- Retains references.
- Owns goroutine lifetimes.

Do not rely on GC to compensate for unclear architecture.

---

## Escape Analysis

Do not manually contort APIs solely to keep values on the stack unless profiling shows allocation matters.

Let the compiler optimize normal code.

Use:

```text
go build -gcflags
```

or profiling tools only when performance work justifies it.

---

## Performance

Prefer clarity first.

Do not prematurely:

- Pool objects.
- Add unsafe conversions.
- Add goroutines.
- Replace interfaces.
- Micro-optimize allocations.
- Rewrite code to avoid every bounds check.
- Manually inline functions.

Measure with real benchmarks and profiles.

---

## Benchmarking

Use Go benchmarks:

```go
func BenchmarkParseWeapon(b *testing.B) {
    // ...
}
```

when performance matters.

Do not benchmark setup work accidentally.

Use:

```go
b.ResetTimer()
```

or setup outside the benchmark loop where appropriate.

---

## Profiling

Use real profiling tools such as:

```text
pprof
go tool pprof
runtime/trace
```

rather than guessing.

Optimize actual hot paths.

---

## Compiler Optimizations

Do not add obscure code solely to encourage inlining or bounds-check elimination without evidence.

Readable Go often optimizes well.

---

## Logging

Use the project's existing logger.

Possible choices include:

```text
log
log/slog
zap
zerolog
logrus
```

Do not add another logging ecosystem casually.

---

## Structured Logging

If the project uses structured logging, use structured fields.

Example with `slog`:

```go
logger.Warn(
    "weapon asset could not be resolved",
    "asset_path", assetPath,
)
```

Do not manually concatenate metadata into long strings if structured fields are supported.

---

## Log Levels

Use levels deliberately.

Do not log routine success at `Info` for every method.

Operational logs should be useful.

Avoid:

```text
starting function
processing item
finished function
```

noise.

---

## Secrets in Logs

Never log:

- Passwords.
- API keys.
- Auth tokens.
- Private keys.
- Sensitive payloads.

Be careful when logging entire structs or request bodies.

---

## `fmt.Println`

Use it for:

- CLI output.
- Tiny scripts.
- Debugging during development.

Do not leave debug prints in production services when a logging system exists.

---

## Testing

Use the standard `testing` package unless the project has established additions.

Go's built-in test tooling is usually enough.

Do not introduce a test framework merely for assertion syntax unless the project already uses one.

---

## Table-Driven Tests

Use table-driven tests when several cases share the same structure.

Good:

```go
tests := []struct {
    name string
    in   string
    want bool
}{
    {
        name: "valid weapon path",
        in:   "/Game/Weapons/Rifle",
        want: true,
    },
    {
        name: "empty path",
        in:   "",
        want: false,
    },
}
```

Do not force every single test into a table.

A standalone test is clearer when there is only one meaningful case.

---

## Subtests

Use:

```go
t.Run(...)
```

for related cases.

Do not create five layers of nested subtests.

Keep failure output understandable.

---

## Test Names

Use descriptive names.

Good:

```go
func TestParseWeaponRejectsMissingID(t *testing.T)
```

Avoid:

```go
func TestParseWeapon1(t *testing.T)
```

---

## Test Helpers

Use:

```go
t.Helper()
```

for assertion/setup helpers so failure locations point to the caller.

Do not build a large custom assertion framework.

---

## Test Behavior

Test observable behavior rather than internal call structure.

Do not introduce interfaces and mocks just so every dependency call can be asserted.

Prefer real lightweight collaborators when practical.

---

## Fakes

Small in-memory fakes are often more maintainable than mock frameworks.

Example:

```go
type fakeRepository struct {
    weapon Weapon
    err    error
}
```

Use what makes the test easiest to understand.

Do not create elaborate mocking architecture.

---

## External Tests

Package tests can use:

```go
package weapon
```

to access internals when appropriate.

Use:

```go
package weapon_test
```

when testing from the consumer's perspective is valuable.

Follow project conventions.

Do not move all tests externally merely for purity.

---

## Test Data

Keep test data minimal.

Do not construct enormous object graphs when the test needs three fields.

Use helper constructors only after repetition becomes real.

---

## Golden Tests

Golden files are useful for large serialized/rendered output.

Do not use them for small values that are easier to assert directly.

When updating goldens, review the diff rather than blindly accepting it.

---

## Fuzz Testing

Use Go's built-in fuzzing for parsers and input-heavy code where malformed input matters.

Example:

```go
func FuzzParseWeapon(f *testing.F) {
    // ...
}
```

Do not add fuzz tests for trivial getters.

---

## Race Tests

For concurrency-sensitive packages:

```text
go test -race ./...
```

is valuable.

Do not assume race-free design merely because mutexes appear in the code.

---

## Example Tests

Use `Example` tests when examples serve both documentation and executable verification.

Do not create examples for every trivial method.

---

## Benchmarks

Keep benchmarks deterministic enough to compare.

Avoid network, disk, or unrelated setup unless those are what the benchmark intends to measure.

---

## Test Parallelism

Use:

```go
t.Parallel()
```

when tests are independent and benefit from parallel execution.

Do not mark tests parallel if they share:

- Global state.
- Environment variables.
- Ports.
- Database state.
- Files.

unless isolation is handled properly.

---

## Environment Variables in Tests

Use:

```go
t.Setenv(...)
```

where supported.

Do not manually set global env state without cleanup.

---

## Temporary Directories

Use:

```go
t.TempDir()
```

instead of manual temp directory management.

---

## Cleanup

Use:

```go
t.Cleanup(...)
```

for test resources whose cleanup should occur after the test.

Do not duplicate manual cleanup code.

---

## Formatting

Always use `gofmt`.

Go formatting is intentionally standardized.

Do not hand-format code against `gofmt`.

Do not argue about indentation styles.

Run:

```text
gofmt
```

or project-provided tooling.

---

## `goimports`

If the project uses `goimports`, use it for imports and formatting.

Do not manually maintain large import groups unnecessarily.

---

## Imports

Keep imports minimal.

Do not use blank imports unless they intentionally trigger registration or initialization.

Example legitimate use:

```go
import _ "modernc.org/sqlite"
```

may be required by a driver architecture.

Comment unusual blank imports when their purpose is not obvious.

---

## Dot Imports

Avoid:

```go
import . "package"
```

in production code.

They obscure where identifiers come from.

Tests or generated DSL-like packages may occasionally use them, but they should be rare.

---

## Import Aliases

Use aliases when:

- Names conflict.
- Generated package names are awkward.
- The alias materially improves clarity.

Do not alias imports merely to make names shorter.

---

## Internal Packages

Use:

```text
internal/
```

when Go's enforced import boundary genuinely matches the project architecture.

Do not place everything under `internal` mechanically.

Use the mechanism deliberately.

---

## `cmd`

For repositories with multiple executables, conventional layouts may use:

```text
cmd/toolname/
```

Follow existing architecture.

Do not introduce the standard project-layout template blindly into a small repository.

Go does not require a universal directory hierarchy.

---

## Avoid Cargo-Cult Project Layouts

Do not automatically create:

```text
cmd/
internal/
pkg/
api/
configs/
scripts/
build/
deployments/
```

for every Go project.

Use only directories the project needs.

A small application can be:

```text
main.go
config.go
server.go
```

and be perfectly professional.

---

## `pkg` Directory

A top-level `pkg/` directory is not required by Go.

Use it only when the project intentionally follows that convention.

Do not create it merely because some repositories do.

---

## Small Packages

Prefer packages with cohesive responsibility.

Do not split code into dozens of microscopic packages solely to enforce architecture.

Every package creates:

- An API boundary.
- Import relationships.
- Naming overhead.
- Potential cycles.

Package boundaries should earn their existence.

---

## Import Cycles

Go forbids import cycles.

Do not solve cycles by creating a generic `common` package and moving everything there.

Instead, examine dependency direction.

Often:

- Interfaces belong with consumers.
- Shared domain types need a lower-level package.
- Responsibilities should be separated differently.

---

## Avoid `common`

A `common` package often becomes a dumping ground.

Prefer explicit packages with real concepts.

Bad:

```text
common.StringUtils
common.Errors
common.Models
common.Helpers
```

---

## Internal Helpers

Small helpers used by one file/package do not require their own package.

Keep them close to usage.

Do not create an import boundary for two tiny functions.

---

## Code Generation

Use code generation when it provides clear value:

- Protocols.
- Database queries.
- Mocks where project convention uses them.
- Repetitive serialization.
- API clients.

Do not generate code that is simpler to write and maintain directly.

Generated code should have an obvious source and regeneration workflow.

---

## `go generate`

If `go generate` is used, keep directives clear.

Do not rely on generated files whose generation process is undocumented.

Follow repository conventions.

---

## Generated Files

Do not manually edit generated files unless explicitly intended.

Modify the generator or source specification.

Generated files should usually contain a generated-code marker when tooling expects it.

---

## Build Tags

Use build tags for genuine platform, integration, or build-mode differences.

Do not use them for ordinary runtime configuration.

Keep tag combinations manageable.

---

## Platform-Specific Files

Go supports filename-based platform selection:

```text
file_windows.go
file_linux.go
```

Use this when it keeps platform-specific code clear.

Do not scatter runtime OS checks throughout generic logic when build-specific implementations are cleaner.

---

## Cross-Platform Paths

Use `filepath` rather than hard-coded separators.

Do not assume Unix filesystem behavior in Windows-compatible tools.

---

## Signals

Use `os/signal` and appropriate context cancellation for graceful shutdown.

Do not build complex signal frameworks for simple processes.

---

## Graceful Shutdown

Servers and workers should have an intentional shutdown path when long-lived resources/goroutines exist.

Typical concerns:

- Stop accepting work.
- Cancel contexts.
- Wait for workers.
- Close resources.

Do not add graceful-shutdown machinery to a short one-shot CLI that does not need it.

---

## Main

Keep `main` focused on process-level concerns:

- Parse flags.
- Load configuration.
- Wire dependencies.
- Start the application.
- Translate errors to exit status.

Do not place the entire application inside `main`.

But explicit dependency wiring in `main` is fine.

---

## `os.Exit`

Use `os.Exit` at the true process boundary.

Do not call it from reusable packages.

Note that deferred functions do not run after `os.Exit`.

This matters for cleanup.

---

## CLI Flags

Use the standard `flag` package when it meets requirements.

Use Cobra, urfave/cli, or another CLI framework when the project already uses it or complexity justifies it.

Do not add a large CLI framework for three flags.

---

## STDOUT and STDERR

Use stdout for normal command output.

Use stderr for errors and diagnostics.

This matters for scripting and pipelines.

Do not mix structured machine output with logging on stdout unless explicitly intended.

---

## Exit Errors

Make CLI errors concise and actionable.

Internal errors may contain richer wrapping context.

Do not print the same wrapped error at several levels.

---

## Time

Use `time.Duration` for durations.

Good:

```go
timeout time.Duration
```

not:

```go
timeout int
```

when units matter.

Parse configuration explicitly.

---

## Duration Constants

Use:

```go
5 * time.Second
```

rather than magic integers like:

```go
5000
```

representing milliseconds.

The type system should communicate units.

---

## Time Measurement

Use `time.Since(start)` for elapsed durations.

Do not use wall-clock timestamp subtraction manually when a duration API suffices.

---

## Timers and Tickers

Always consider cleanup:

```go
ticker := time.NewTicker(interval)
defer ticker.Stop()
```

Do not leak tickers in long-running code.

---

## `time.After`

Repeated `time.After` in loops may allocate timers repeatedly.

In hot or long-running loops, reusable timers may be more appropriate.

Do not optimize occasional usage prematurely.

---

## Randomness

Use:

```go
math/rand
```

or current project-approved APIs for non-security randomness.

Use:

```go
crypto/rand
```

for secrets, tokens, keys, and security-sensitive randomness.

Do not use pseudo-random generators for authentication tokens.

---

## Security

Do not weaken security for convenience.

Avoid:

- `InsecureSkipVerify: true`.
- Shell construction from untrusted strings.
- SQL string interpolation.
- Unsafe deserialization.
- Path traversal.
- Secret logging.
- Predictable token generation.
- Ignoring cryptographic errors.

Use established secure libraries.

---

## TLS

Do not disable TLS verification merely to fix certificate errors.

If custom trust roots are needed, configure them deliberately.

`InsecureSkipVerify` should be exceptional and clearly justified.

---

## Subprocesses

Use:

```go
exec.CommandContext
```

when cancellation matters.

Pass arguments separately:

```go
exec.Command("tool", "--file", path)
```

rather than constructing a shell command string.

Do not invoke a shell unless shell syntax is genuinely required.

---

## Command Output

Use:

```go
cmd.Output()
cmd.CombinedOutput()
cmd.Run()
```

according to whether output is needed.

Include stderr context where it helps diagnose failure.

Do not return only:

```text
command failed
```

when command output explains why.

---

## Shell Injection

Never build:

```go
exec.Command("sh", "-c", "tool "+userInput)
```

with untrusted values.

Separate executable arguments.

---

## File Permissions and Secrets

Files containing:

- Tokens.
- Credentials.
- Private keys.

should use restrictive permissions where appropriate.

Do not blindly use general-purpose modes for sensitive data.

---

## Crypto

Use standard-library cryptography or established audited libraries.

Never invent:

- Encryption algorithms.
- Password hashing.
- MAC schemes.
- Signature formats.

Do not use general hash functions as password hashing.

---

## Passwords

Use an established password hashing algorithm/library appropriate to the application, such as bcrypt or Argon2 integrations.

Do not implement password hashing yourself.

---

## Constant-Time Comparisons

For authentication tokens or MAC values where timing matters, use appropriate constant-time comparison APIs.

Do not rely on ordinary equality where cryptographic verification requires timing resistance.

---

## HTTP Security

Use standard server protections and project middleware.

Be deliberate about:

- Request size limits.
- Timeouts.
- Header limits.
- CORS.
- Authentication.

Do not add a large custom security layer when standard server configuration handles the concern.

---

## Request Body Limits

For untrusted input, consider bounds.

Do not read arbitrarily large bodies using:

```go
io.ReadAll(r.Body)
```

without a limit when attackers can control input size.

Use:

```go
http.MaxBytesReader
io.LimitReader
```

where appropriate.

---

## Parsing

Parsers should validate bounds and return clear errors.

Avoid panics caused by malformed input.

Bad:

```go
parts := strings.Split(line, ":")
value := parts[1]
```

when the separator may be missing.

Prefer:

```go
name, value, ok := strings.Cut(line, ":")
if !ok {
    return fmt.Errorf("invalid line %q", line)
}
```

Use direct standard-library helpers.

---

## `strings.Cut`

Prefer `strings.Cut` when splitting once.

Do not use:

```go
strings.Split(value, ":")[1]
```

for malformed external input.

---

## Numeric Parsing

Use:

```go
strconv.Atoi
strconv.ParseInt
strconv.ParseFloat
```

and handle errors.

Do not silently convert invalid numbers to zero unless zero is explicitly the intended fallback.

---

## Bounds

Validate indexes derived from external input before indexing slices.

Do not recover from out-of-range panics as input validation.

---

## Overflow

Integer overflow behavior depends on integer type and operation.

Use types appropriate to expected ranges.

For security-sensitive arithmetic, validate bounds deliberately.

Do not add arbitrary overflow checks to obviously bounded values.

---

## `int` vs Sized Integers

Use `int` for normal indexing/counting within a Go process.

Use sized types such as:

```go
int64
uint32
```

when:

- Wire formats require them.
- Database schema requires them.
- Exact range matters.

Do not use `uint` simply because a value cannot be negative.

Signed integers often interact more naturally with Go APIs.

---

## Avoid Unsigned for Ordinary Counts

Do not use `uint` by default for counts, indexes, or lengths.

Negative values may be invalid, but unsigned arithmetic can create awkward underflow and conversion problems.

Use `int` unless a specific representation requires otherwise.

---

## Reflection-Free Serialization

Prefer ordinary structs and tags to hand-written reflection infrastructure.

Do not build generic mappers around `reflect` unless multiple dynamic schemas truly require them.

---

## `init`

Use `init()` sparingly.

Valid uses include:

- Generated registration.
- Package-level initialization with no meaningful error path.

Avoid major work in `init()`:

- Network calls.
- File loading.
- Hidden registration chains.
- Startup side effects.

Explicit initialization is easier to reason about.

---

## Global Variables

Avoid mutable global state.

Package-level immutable constants and stateless values are fine.

Use dependency wiring rather than global clients or repositories when lifecycle/testing matters.

---

## Package-Level Clients

A package-global immutable or concurrency-safe client may be acceptable for simple applications.

Do not turn this into a rule.

Consider whether explicit injection improves:

- Lifecycle.
- Configuration.
- Tests.

Avoid global mutable configuration.

---

## Singletons

Go does not need singleton patterns for ordinary services.

Package state already behaves globally.

That is not an argument to use global state more.

Prefer explicit dependencies.

---

## Functional Style

Go supports functions as values but is not designed around dense functional pipelines.

Do not recreate:

```text
map/filter/reduce chains
monads
result combinators
```

for ordinary application code unless the project deliberately uses such abstractions.

Simple loops are idiomatic.

---

## Loops

Go has one loop keyword:

```go
for
```

Use it directly.

Good:

```go
for _, weapon := range weapons {
    if !weapon.Enabled {
        continue
    }

    active = append(active, weapon)
}
```

Do not build generic filtering helpers merely to avoid four lines.

---

## Range

Use range naturally.

Be mindful of whether you need:

- Index.
- Value.
- Pointer/reference-like access.

Do not copy large values unnecessarily if taking addresses or mutations matter.

---

## Range Variables and Addresses

Be aware of loop-variable semantics for the project's supported Go version.

When code must support older versions, taking addresses of range variables can produce surprising results.

Even on newer Go versions with improved loop-variable semantics, write clear code and respect the declared toolchain.

---

## Mutating Slice Elements

If modifying elements in place, use the index:

```go
for i := range weapons {
    weapons[i].Enabled = true
}
```

Do not mutate a copied range value and expect the slice to change:

```go
for _, weapon := range weapons {
    weapon.Enabled = true
}
```

if `Weapon` is a struct value.

---

## Break and Continue

Use early `continue` to reduce nesting.

Good:

```go
for _, asset := range assets {
    if asset.Kind != WeaponAsset {
        continue
    }

    weapon, err := parseWeapon(asset)
    if err != nil {
        return nil, err
    }

    weapons = append(weapons, weapon)
}
```

Straightforward control flow is idiomatic Go.

---

## Switch

Use switch for multi-branch logic.

Go switches do not fall through by default.

Do not add:

```go
fallthrough
```

unless the semantics genuinely require it.

Often explicit multiple cases are clearer.

---

## Type Switch

Use type switches for genuine interface-based variation.

Do not design everything around runtime type inspection.

Prefer static types.

---

## Empty Switch

An empty-condition switch:

```go
switch {
case x < 0:
case x == 0:
default:
}
```

can be clear for multi-condition branching.

Do not use it when normal `if`/`else` is simpler.

---

## Error Scope

Use compact initialization:

```go
if err := saveWeapon(weapon); err != nil {
    return err
}
```

when the value is needed only for the condition.

Do not force everything into inline initializers if the variable has later meaning.

---

## Shadowing

Be careful with short declarations:

```go
:=
```

which can shadow variables.

Classic bug:

```go
var err error

if condition {
    value, err := load()
    // inner err shadows outer err
}
```

Use tooling and scope carefully.

Do not avoid `:=`; it is idiomatic. Just understand scope.

---

## Named Return Values

Use named return values when they materially improve documentation or are needed for deferred result modification.

Avoid them in long functions where they make mutation and naked returns difficult to follow.

---

## Naked Returns

Avoid naked `return` in non-trivial functions.

Bad:

```go
func load() (value Weapon, err error) {
    // 40 lines
    return
}
```

Explicit returns are easier to read:

```go
return value, err
```

Small functions may use named returns where project style allows.

---

## Multiple Return Values

Use Go's multiple returns naturally.

Good:

```go
weapon, found := weapons[id]
```

or:

```go
value, err := parse(...)
```

Do not create temporary result structs for ordinary two-value contracts.

Use a struct when several returned values form one meaningful object.

---

## Boolean Returns

Use booleans for real yes/no outcomes.

For lookup APIs:

```go
value, ok := m[key]
```

is idiomatic.

Do not return a bool plus an error if the two states overlap confusingly.

Design return values around meaningful states.

---

## Time to Introduce a Struct

If a function returns:

```go
value, metadata, source, cached, err
```

consider a result struct.

Too many positional returns are hard to understand.

---

## Constants

Use constants for meaningful fixed values.

Good:

```go
const maxRetryAttempts = 3
```

Do not create constants for:

```go
const zero = 0
```

A constant should add meaning.

---

## `iota`

Use `iota` for simple related enum-like constants.

Example:

```go
type Status int

const (
    StatusUnknown Status = iota
    StatusReady
    StatusFailed
)
```

Do not use complex `iota` expressions that require mental arithmetic to understand.

---

## String Enums

For external-facing values, typed strings can be useful:

```go
type Category string

const (
    CategoryRifle   Category = "rifle"
    CategoryShotgun Category = "shotgun"
)
```

This works well for JSON/API representation.

Do not build elaborate enum frameworks.

---

## Validation

Validate at boundaries:

- HTTP requests.
- Config.
- CLI input.
- External files.
- Network responses.
- Database-derived untrusted data.
- Plugin interfaces.

Do not repeatedly validate internal objects whose invariants are already established.

---

## Validation Libraries

Use the project's existing validation library if one exists.

Do not add a validation dependency for three obvious conditions.

Plain Go checks are often the clearest:

```go
if req.Name == "" {
    return errors.New("name is required")
}
```

---

## Tags

Struct tags are part of integration contracts.

Do not overload one domain struct with tags for every system:

```go
json
yaml
db
form
validate
xml
```

without considering whether transport and domain types have diverged too far.

One type can serve multiple boundaries when shapes genuinely match.

Do not duplicate types automatically either.

---

## DTO Explosion

Avoid automatically creating:

```text
CreateWeaponRequest
CreateWeaponDTO
WeaponDomain
WeaponEntity
WeaponModel
WeaponResponse
WeaponView
```

when several shapes are identical.

Separate models only when boundaries truly differ.

---

## Transport vs Domain Types

Separate them when:

- Validation differs.
- Field visibility differs.
- External contracts should not dictate domain structure.
- Persistence fields differ materially.

Do not map identical structs back and forth solely for architecture.

---

## Copying Structs

Explicit field mapping can be fine.

Do not introduce reflection-based auto-mappers merely to avoid a handful of assignments.

Explicit mappings are easy to inspect.

---

## Avoid Builder Pattern by Default

Go struct literals and constructors are usually enough.

Avoid:

```go
NewWeaponBuilder().
    WithName(name).
    WithDamage(damage).
    Build()
```

for straightforward data construction.

Use builders only for genuinely complex staged construction.

---

## Avoid Fluent APIs

Go generally favors direct calls over long fluent chains.

Do not force method chaining into APIs simply because it looks elegant in other languages.

---

## Avoid OOP Layering

Do not automatically structure an application as:

```text
Controller
Service
Repository
Manager
Provider
Factory
```

for every feature.

A Go application can cleanly be:

```text
handler -> domain function -> database query
```

where that is enough.

Architecture should reflect complexity, not ceremony.

---

## Avoid Service Objects Everywhere

A `Service` type can be perfectly reasonable when it coordinates several collaborators.

Do not create one per entity merely by template.

Ask what responsibility the type actually owns.

---

## Avoid Repository Interfaces Everywhere

Do not define generic CRUD repositories merely because the system uses a database.

Consumers should depend on the operations they actually need.

A package can call SQL directly when that is the clearest design.

---

## Avoid Manager Types

`Manager` often means the responsibility is unclear.

Prefer a more specific name:

```text
Cache
Loader
Scheduler
Registry
Store
Resolver
```

if that is what the type actually does.

Use `Manager` only when it genuinely describes the domain.

---

## Avoid Helper Types

Do not create:

```go
type StringHelper struct{}
```

or:

```go
type FileUtils struct{}
```

Go packages already provide namespacing.

Use package functions.

---

## Avoid Generic `util` Packages

Before placing a function in `util`, ask where the concept actually belongs.

Generic utility packages often become dependency magnets.

Prefer cohesive packages.

---

## Avoid Premature Interfaces

This is one of the clearest AI-generated Go smells.

Do not start a feature by defining:

```go
type Service interface
type Repository interface
type Factory interface
```

before concrete code exists.

Start concrete.

Extract interfaces where consumers need substitution.

---

## Avoid Mock-Driven Architecture

Do not create interfaces solely because a mocking library wants them.

Go tests can use:

- Fakes.
- Test servers.
- In-memory implementations.
- Direct dependencies.

Architecture should serve production design first.

---

## Avoid Dependency Injection Frameworks

Do not add reflection-based DI containers unless the existing application already uses one and benefits from it.

Explicit constructor wiring is idiomatic and easy to debug.

---

## Avoid Goroutine-Driven Architecture

Concurrency is not architecture.

Do not split every operation into goroutines and channels to make the program feel more Go-like.

Sequential code is often the right answer.

---

## Avoid Channels for Every Event

Bad:

```go
requestCh <- request
result := <-resultCh
```

when:

```go
result := process(request)
```

does exactly what is needed.

Channels are for concurrent coordination, not indirect function calls.

---

## Avoid Context Everywhere

Do not add:

```go
ctx context.Context
```

to pure functions such as:

```go
func NormalizeName(ctx context.Context, name string) string
```

if the function has nothing cancellable or request-scoped.

Context should have a reason.

---

## Avoid Pointer Everywhere APIs

Do not make every struct:

```go
*Weapon
*Config
*Stats
```

by default.

Pointers should communicate mutation, identity, nil, or copy cost.

Values are idiomatic and often simpler.

---

## Avoid Excessive Defensive Nil Checks

Bad:

```go
func ProcessWeapon(w *Weapon) error {
    if w == nil {
        return errors.New("weapon is nil")
    }

    if w.Name == "" {
        // ...
    }
}
```

if the function is private and all callers establish a valid weapon.

Validate at real boundaries.

Do not clutter trusted internal paths.

---

## Avoid Panics as Validation

Do not replace ordinary validation with:

```go
panic("invalid weapon")
```

Return errors.

Panics should indicate programming failures or unrecoverable initialization.

---

## Avoid Error Boilerplate Without Context

This:

```go
if err != nil {
    return err
}
```

is perfectly fine when no context is needed.

Do not wrap every error just to create text.

Bad:

```go
return fmt.Errorf("error occurred while calling load: %w", err)
```

if the caller already knows exactly what operation failed.

Wrap where it adds meaningful context.

---

## Avoid Over-Wrapping Errors

Error chains like:

```text
failed to execute operation:
failed to process weapon:
failed to load data:
failed to read:
open file: no such file
```

are noisy.

Each layer should add only information unavailable below it.

---

## Avoid Logging and Returning

Unless the current layer owns logging, avoid both logging and returning the same error.

This is one of the most common causes of duplicate production logs.

---

## Avoid Custom Error Codes Without Need

Do not build:

```go
type ErrorCode int
```

and giant code tables when normal errors plus `errors.Is/As` suffice.

Protocol layers can translate domain errors to status codes.

---

## Avoid `any` as an Escape Hatch

Do not replace a difficult type design with:

```go
map[string]any
[]any
```

throughout the system.

Dynamic types should stay near dynamic boundaries.

Normalize into typed structures.

---

## Avoid Reflection-Based Mapping

Do not write generic reflection code to copy fields between similar structs unless scale genuinely justifies it.

Explicit assignment is often:

- Faster.
- Safer.
- Easier to search.
- Easier to debug.

---

## Avoid Clever Generics

If three concrete functions are easier to understand than one generic abstraction with several constraints, use the concrete functions.

Genericity should reduce real duplication, not demonstrate Go's type parameters.

---

## Avoid Excessive Functional Options

Do not turn every constructor into:

```go
New(
    WithFoo(...),
    WithBar(...),
    WithBaz(...),
)
```

when a simple config struct communicates the configuration better.

---

## Avoid Magic Registrations

Be cautious with hidden registration in:

```go
init()
```

such as:

```go
Register("weapon", newWeaponHandler)
```

Plugin systems may need this.

Ordinary application wiring is usually clearer when explicit.

---

## Avoid Hidden Side Effects

Do not make importing a package:

- Start goroutines.
- Connect to services.
- Read files.
- Mutate global state.

Package import should generally define behavior, not launch the application.

---

## Avoid Excessive `init`

Multiple `init` functions across packages can make startup order difficult to understand.

Prefer explicit initialization whenever errors or dependencies matter.

---

## Avoid Boolean Parameter Piles

Bad:

```go
processWeapon(weapon, true, false, true)
```

Prefer an options struct or named specialized operations when the flags genuinely exist.

Example:

```go
type ProcessOptions struct {
    Validate        bool
    Normalize       bool
    IncludeMetadata bool
}
```

But do not create options for behavior that is always fixed.

---

## Avoid Fake Flexibility

Bad:

```go
type ProcessorOptions struct {
    EnableValidation  bool
    EnableParsing     bool
    EnableMapping     bool
    EnablePersistence bool
}
```

when every execution always requires those stages.

Do not turn required behavior into configuration.

---

## Avoid Premature Caching

Do not add caches without a real need.

Caching introduces:

- Invalidations.
- Memory use.
- Synchronization.
- Stale data.
- More tests.

Measure before adding complexity.

---

## Avoid Premature Concurrency

Do not parallelize code before determining:

- Work is independent.
- Work is expensive enough.
- Downstream systems tolerate concurrency.
- Ordering is irrelevant or handled.
- Error cancellation semantics are understood.

Concurrency has a cost.

---

## Avoid Premature Optimization

Do not:

- Pool buffers everywhere.
- Use unsafe string/byte conversion.
- Hand-inline functions.
- Replace interfaces based on speculation.
- Reuse objects globally.
- Introduce atomics.
- Build lock-free queues.

Measure first.

---

## Avoid Artificial Abstraction

Bad:

```go
type Processor interface {
    Process(context.Context, Input) (Output, error)
}

type DefaultProcessor struct {
    handler Handler
    mapper  Mapper
}
```

when one function would suffice:

```go
func Process(ctx context.Context, input Input) (Output, error)
```

Do not design an internal framework for one workflow.

---

## Avoid Enterprise Names

Before adding:

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

ask whether the name describes a real responsibility.

Do not create architecture by vocabulary.

---

## Avoid AI-Looking Comments

Do not narrate obvious code.

Bad:

```go
// Check if the weapon exists.
if weapon == nil {
    // Return an error if the weapon does not exist.
    return errors.New("weapon not found")
}
```

Better:

```go
if weapon == nil {
    return errors.New("weapon not found")
}
```

Write comments when explaining:

- Why a workaround exists.
- Concurrency invariants.
- Protocol requirements.
- Platform limitations.
- Non-obvious performance behavior.

Prefer **why**, not **what**.

---

## Avoid Tutorial Prose

Avoid comments such as:

```text
This function is responsible for...
This ensures that...
The following logic...
In order to...
This provides a robust and scalable solution...
```

Write normal engineering comments.

---

## Avoid Decorative Comments

Do not add:

```go
// ========================================
// INITIALIZATION
// ========================================
```

unless the repository deliberately uses this format.

Go files should generally remain simple enough not to need decorative separators.

---

## Avoid Placeholder TODOs

Do not leave:

```go
// TODO: add caching
// TODO: add retries
// TODO: add advanced validation
```

in finished work unless the user explicitly requested scaffolding.

Implement the requested behavior.

---

## TODO Comments

Legitimate TODOs should ideally explain:

- What remains.
- Why it cannot be addressed now.
- Relevant issue/reference if project convention uses one.

Do not use TODOs to advertise speculative features.

---

## Documentation

Use Go doc comments for exported packages/types/functions when required or useful.

Good:

```go
// Loader reads weapon assets from the configured source.
type Loader struct {
    // ...
}
```

Avoid:

```go
// Loader is a struct that is responsible for loading weapons
// and provides robust loading functionality.
```

Keep documentation concise and factual.

---

## Package Comments

Public library packages may benefit from a package comment explaining their purpose.

Do not write a miniature tutorial at the top of every internal package.

---

## Examples

Use example tests/docs when they materially help public API users.

Do not add examples for obvious internal helpers.

---

## Linting

Use the project's configured tooling.

Common tools include:

```text
go vet
staticcheck
golangci-lint
```

Do not add a giant lint suite during unrelated work.

Do not disable warnings merely because generated code triggers them.

---

## `go vet`

Pay attention to warnings such as:

- Copying mutexes.
- Incorrect printf formats.
- Unreachable code.
- Misused struct tags.

Do not suppress them casually.

---

## Staticcheck

If used, treat findings as meaningful signals.

Do not rewrite code mechanically when a warning does not apply to the project's design.

Understand the issue first.

---

## `golangci-lint`

Respect the project's configuration.

Avoid broad:

```text
nolint
```

comments.

When suppression is necessary, target the exact linter and explain why when project style expects it.

---

## `nolint`

Bad:

```go
//nolint
```

Better when genuinely necessary:

```go
//nolint:gosec // Input is an internal constant, not user-controlled.
```

Do not suppress real problems.

---

## Formatting

Always format changed code.

Typical tools:

```text
gofmt
goimports
```

Do not manually align fields or comments in ways formatting tools will undo.

---

## Build

Use:

```text
go build ./...
```

or the project's narrower build commands when appropriate.

Do not assume every package is intended to build under every optional tag/platform.

Inspect CI.

---

## Tests

Typical checks may include:

```text
go test ./...
go test -race ./...
go vet ./...
```

along with project linting.

Do not blindly run expensive all-package race tests if project workflow intentionally scopes them.

Use repository scripts where available.

---

## Modules

Use Go modules.

Do not manually edit `go.sum`.

Use:

```text
go get
go mod tidy
```

according to project workflow.

Avoid broad dependency updates during unrelated work.

---

## `go mod tidy`

Be careful: it may change module files beyond the immediate dependency.

Review the diff.

Do not run it blindly if the project has special build-tag/platform dependencies you have not considered.

---

## Dependencies

Before adding a dependency, ask:

- Does the standard library already solve this?
- Does the project already have an equivalent?
- Is the module maintained?
- Is its dependency tree reasonable?
- Is the API stable?
- Is the extra functionality worth it?

Go's standard library is strong.

Use it when it fits.

---

## Avoid Dependency for Tiny Helpers

Do not add a module solely for:

- String containment.
- Slice filtering.
- UUID formatting if already available in project.
- Retry loops with trivial semantics.
- Simple config parsing.

Do not reimplement complex security/protocol functionality merely to remain dependency-free.

---

## Module Boundaries

Avoid exporting third-party dependency types from public APIs unless the coupling is intentional.

For internal applications, this concern may be less important.

Do not wrap dependencies merely to avoid ever exposing them.

Balance real stability needs against pointless adapter layers.

---

## Versioning

For public modules, respect compatibility.

Avoid breaking exported names or semantics casually.

For internal applications, do not over-engineer every package as if it were a public SDK.

---

## API Design

A good Go API should usually be:

- Small.
- Predictable.
- Explicit.
- Easy to use correctly.
- Hard to misuse.
- Light on abstraction.

Do not make callers understand an internal architecture to perform a simple operation.

---

## Before Finishing

Review the change and remove or correct:

- Unnecessary interfaces.
- Interfaces defined beside their implementations without consumer need.
- Unnecessary manager/service/provider layers.
- Factory types with one implementation.
- Dependency-injection containers.
- Generic utility packages.
- Excessive pointers.
- Redundant nil checks.
- Goroutines without clear ownership.
- Channels used as indirect function calls.
- Context parameters with no real purpose.
- Overly broad `any` values.
- Reflection where static code would work.
- Excessive generics.
- Over-wrapped errors.
- Duplicate logging.
- Panic-based runtime error handling.
- Fire-and-forget goroutines.
- Arbitrary channel buffers.
- Mutable global state.
- Heavy `init()` side effects.
- Debug `fmt.Println`.
- Placeholder TODOs.
- Dead code.
- Unused exports.
- Speculative configuration.
- Speculative extensibility.
- Unrelated refactors.

Then run the repository's established checks where available.

Typical Go checks may include:

```text
gofmt
go test ./...
go vet ./...
go build ./...
```

Concurrency-sensitive projects may additionally use:

```text
go test -race ./...
```

Projects may also use:

```text
staticcheck
golangci-lint
goimports
```

Do not assume these exact commands or tools exist.

Inspect:

```text
go.mod
Makefile
Taskfile.yml
CI configuration
golangci-lint configuration
repository documentation
```

and follow the project's established workflow.

The final code should look like it naturally belongs in the repository rather than like a generic AI-generated Go solution.

It should feel like Go written by an experienced maintainer: concrete before abstract, explicit about errors, restrained with interfaces, conservative with concurrency, small in API surface, and deliberately boring where boring code is the clearest code.