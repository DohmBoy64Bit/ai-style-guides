# Ruby Style Guide

Write Ruby as an experienced professional Ruby developer maintaining a real production codebase.

The goal is clean, idiomatic, maintainable Ruby—not code that looks generated, over-engineered, excessively defensive, or translated mechanically from Java, C#, Python, or TypeScript.

The most important rule:

> Do not optimize for demonstrating Ruby features or design patterns. Optimize for producing the smallest idiomatic production-quality change that an experienced Ruby maintainer would reasonably write.

## General Principles

Prefer:

- Existing project conventions over personal preferences.
- Simple objects and methods.
- Clear domain terminology.
- Small focused changes.
- Ruby's standard library and core abstractions.
- Enumerable methods when they improve readability.
- Straightforward loops when they are clearer.
- Explicit control flow over hidden magic.
- Duck typing where appropriate.
- Objects that have real responsibilities.
- Value-oriented objects for meaningful domain values.
- Composition over inheritance.
- Guard clauses over deep nesting.
- Clear errors over silent fallback.
- Direct implementation over speculative extensibility.

Avoid:

- Java-style class hierarchies.
- Interfaces simulated through abstract base classes.
- Manager/service/provider/factory architecture without need.
- Excessive modules.
- Excessive `Concern` usage.
- Metaprogramming for ordinary application logic.
- Monkey-patching external classes without a strong reason.
- `send` and dynamic dispatch where direct calls suffice.
- Callbacks that hide major business logic.
- Repeated defensive checks for internal invariants.
- Clever one-liners that make code harder to understand.

Do not refactor unrelated code unless the task requires it.

---

## Match the Existing Codebase

Before writing Ruby, inspect nearby code and follow the repository's established conventions for:

- Ruby version.
- Framework version.
- Naming.
- Directory structure.
- Service objects.
- Query objects.
- Form objects.
- Jobs.
- Models.
- Modules.
- Error handling.
- Logging.
- Testing.
- Dependency management.
- Formatting.
- Linting.
- Type checking if present.
- Rails conventions if applicable.

Check relevant files such as:

```text
.ruby-version
Gemfile
Gemfile.lock
.gemspec
.rubocop.yml
.rspec
sorbet/
steep/
Rakefile
config/
```

Consistency with the repository is more important than imposing a different Ruby style.

Do not introduce a new architectural pattern merely because it is popular elsewhere.

---

## Ruby Version

Determine the supported Ruby version before using newer syntax.

Check:

```text
.ruby-version
Gemfile
CI configuration
gemspec
```

Do not assume the newest Ruby release.

Features such as:

- Pattern matching.
- Endless methods.
- Numbered block parameters.
- Data classes.
- Argument forwarding.
- New standard-library APIs.

must match the supported runtime.

Do not modernize unrelated files solely to use newer syntax.

---

## Naming

Use conventional Ruby naming:

- `snake_case` for variables and methods.
- `PascalCase` for classes and modules.
- `SCREAMING_SNAKE_CASE` for true constants.
- Predicate methods end in `?`.
- Dangerous or surprising variants may end in `!`.
- Assignment-like methods may end in `=`.

Good:

```ruby
weapon_definition
asset_path
package_name

load_package
resolve_asset
parse_weapon_data
weapon_loaded?
```

Avoid generic AI-style names:

```ruby
data_manager
process_handler
utility_helper
generic_service
result_processor
enhanced_processor
operation_manager
```

Prefer domain language.

Bad:

```ruby
data = get_data
result = process_data(data)
```

Better:

```ruby
weapon = load_weapon_definition
stats = parse_weapon_stats(weapon)
```

Do not make names excessively verbose.

Avoid:

```ruby
successfully_parsed_weapon_configuration_result
```

when:

```ruby
weapon_config
```

is clear.

---

## Method Names

Methods should describe behavior.

Prefer:

```ruby
load_weapon
resolve_asset
normalize_path
build_manifest
```

Avoid:

```ruby
process
handle
execute
perform
manage
run_logic
```

unless the surrounding type already provides enough context.

For example:

```ruby
WeaponImporter#call
```

may be perfectly clear if the object has one obvious operation.

Context matters.

---

## Predicate Methods

Use `?` for methods that answer a question.

Good:

```ruby
weapon.loaded?
config.valid?
user.admin?
```

Do not write:

```ruby
is_loaded
has_permission
```

unless matching an external API or established project convention.

Prefer:

```ruby
loaded?
permission?
```

or a domain-specific predicate.

---

## Bang Methods

Use `!` meaningfully.

A bang method should generally indicate a more dangerous, mutating, exception-raising, or surprising version of another operation.

For example:

```ruby
save
save!
```

Do not add `!` simply because a method mutates.

Many idiomatic Ruby methods mutate without `!` when no non-bang counterpart exists.

The distinction should be meaningful.

---

## Methods

Keep methods focused.

Good:

```ruby
package = provider.load_package(path)
mapper.map(package)
```

Avoid splitting obvious operations into unnecessary helpers.

Do not create:

```ruby
def load_package_from_provider(provider, path)
  provider.load_package(path)
end
```

unless the helper introduces meaningful behavior or a real abstraction boundary.

---

## Method Length

Do not enforce arbitrary line-count rules mechanically.

A 20-line cohesive method can be better than six tiny methods requiring constant jumping around.

Extract methods when they:

- Represent a meaningful domain operation.
- Are reused.
- Clarify genuinely complex behavior.
- Isolate side effects.
- Improve testability.

Do not extract solely because a linter reports a number without considering readability.

---

## Functions vs Objects

Ruby has no standalone functions in the same sense as some languages, but module functions and simple methods are often enough.

Do not turn every operation into a class.

Avoid:

```ruby
class WeaponNameNormalizer
  def call(name)
    name.strip.downcase
  end
end
```

when:

```ruby
def normalize_weapon_name(name)
  name.strip.downcase
end
```

or an appropriate module function is enough.

Use an object when it owns meaningful state, dependencies, or cohesive behavior.

---

## Classes

Use classes when they model actual objects with responsibilities.

Good:

```ruby
class PackageCache
  def initialize
    @packages = {}
  end

  def fetch(path)
    @packages[path]
  end

  def store(path, package)
    @packages[path] = package
  end
end
```

Do not create classes merely because a concept has a noun.

Avoid architecture such as:

```text
WeaponManager
WeaponService
WeaponProvider
WeaponHandler
WeaponProcessor
WeaponCoordinator
```

unless those types represent genuinely distinct responsibilities.

---

## Plain Old Ruby Objects

Simple Ruby objects are often enough.

Example:

```ruby
class WeaponImporter
  def initialize(repository:)
    @repository = repository
  end

  def call(path)
    weapon = parse(path)
    repository.save(weapon)
  end

  private

  attr_reader :repository
end
```

Do not wrap this in:

```text
WeaponImporterInterface
WeaponImporterFactory
WeaponImporterManager
```

without an actual need.

---

## Avoid Java-Style Architecture

Do not write Ruby as if it were Java or C#.

Avoid:

```ruby
class AbstractWeaponProcessor
  def process
    raise NotImplementedError
  end
end

class DefaultWeaponProcessor < AbstractWeaponProcessor
  # ...
end
```

when a normal object or callable is sufficient.

Ruby already supports duck typing.

Do not simulate interface hierarchies unnecessarily.

---

## Duck Typing

Prefer behavioral contracts over strict inheritance when appropriate.

If a collaborator needs:

```ruby
provider.load(path)
```

then any object responding appropriately may work.

Do not require all implementations to inherit from:

```ruby
BaseProvider
```

merely to prove compatibility.

Tests and clear APIs often provide enough confidence.

---

## Modules

Use modules for:

- Namespacing.
- Shared behavior.
- Mixins with genuine semantic meaning.
- Module functions.
- Framework extension points.

Do not create modules merely to avoid having another file-level function.

Do not treat modules as dumping grounds.

---

## Avoid Generic Utility Modules

Avoid:

```text
Utils
Helpers
Common
Misc
General
```

containing unrelated behavior.

Prefer domain-specific modules:

```text
WeaponPaths
ManifestParsing
AssetNaming
```

Small local helpers can remain private methods near their usage.

---

## Mixins

Use mixins when several classes genuinely share a coherent capability.

Bad:

```ruby
module CommonMethods
  def log
    # ...
  end

  def validate
    # ...
  end

  def normalize
    # ...
  end
end
```

A mixin should represent a meaningful behavior.

Avoid using mixins solely to avoid composition.

---

## Rails Concerns

If the project uses Rails concerns, keep them cohesive.

Good concern names describe behavior:

```text
Searchable
Publishable
Archivable
```

Avoid:

```text
UserHelpers
SharedModelMethods
CommonBehavior
```

Do not move methods into a concern merely to make a model shorter.

A shorter file is not automatically a better design.

---

## Avoid Concern Explosion

A model with:

```text
Trackable
Searchable
Serializable
Cacheable
Filterable
Sortable
Exportable
Auditable
```

may have become harder to understand rather than more modular.

Use concerns where shared behavior is genuinely reusable and cohesive.

---

## Inheritance

Prefer composition unless the relationship is naturally hierarchical.

Inheritance can be appropriate for:

- Framework conventions.
- STI.
- Shared base infrastructure.
- Specialized domain hierarchies.

Do not create inheritance merely to reuse a few methods.

---

## ActiveRecord Inheritance

If working with Rails STI, use it intentionally.

Do not choose STI merely because several models have similar fields.

STI can introduce coupling and awkward query behavior.

Follow existing application architecture.

---

## `Struct`

Use `Struct` for simple data objects when appropriate.

Example:

```ruby
WeaponStats = Struct.new(
  :damage,
  :fire_rate,
  :magazine_size,
  keyword_init: true
)
```

Do not automatically use `Struct` for every data shape.

A normal class may be clearer when behavior or validation exists.

---

## `Data`

If supported by the Ruby version, `Data` can be useful for immutable value-like objects.

Example:

```ruby
WeaponStats = Data.define(
  :damage,
  :fire_rate,
  :magazine_size
)
```

Use it where immutability and value semantics match the domain.

Do not introduce it into an older-version project or replace existing classes as unrelated cleanup.

---

## Value Objects

Use value objects when they prevent meaningful mistakes or encapsulate domain invariants.

Example:

```ruby
class WeaponId
  attr_reader :value

  def initialize(value)
    raise ArgumentError, "weapon ID is required" if value.empty?

    @value = value.freeze
  end

  def ==(other)
    other.is_a?(self.class) && other.value == value
  end
end
```

Do not wrap every string and number in a class.

A value object should earn its existence.

---

## Attribute Readers

Use:

```ruby
attr_reader :name
```

instead of:

```ruby
def name
  @name
end
```

when behavior is identical.

Likewise use:

```ruby
attr_accessor
attr_writer
```

when direct access is truly intended.

Do not generate explicit getters/setters just to resemble another language.

---

## Avoid Getters and Setters Everywhere

Bad:

```ruby
def get_name
  @name
end

def set_name(name)
  @name = name
end
```

Prefer idiomatic Ruby:

```ruby
attr_accessor :name
```

or, better, expose only the operations the domain actually needs.

---

## Encapsulation

Keep implementation details private.

Use `private` for helpers callers should not depend on.

Do not make methods public merely for tests.

Test behavior through public APIs where practical.

---

## `private`

Remember Ruby's private method semantics.

Private methods cannot normally be called with an explicit receiver.

Use this intentionally.

Do not change visibility simply to avoid understanding call semantics.

---

## Protected Methods

Use `protected` rarely.

It is useful when peer instances of the same hierarchy need access.

Do not use it merely because it exists between public and private.

---

## Initialization

Construct objects in a valid state where practical.

Good:

```ruby
class Weapon
  def initialize(id:, name:)
    @id = id
    @name = name
  end
end
```

Do not create partially initialized objects requiring callers to set several fields after construction unless framework conventions require it.

---

## Keyword Arguments

Use keyword arguments when multiple arguments would be ambiguous.

Good:

```ruby
Weapon.new(
  id: id,
  name: name,
  damage: damage
)
```

Avoid:

```ruby
Weapon.new(id, name, damage, fire_rate, capacity, category)
```

when the meaning becomes difficult to track.

Do not use keyword arguments for every trivial two-argument method.

---

## Default Arguments

Use defaults when they represent meaningful normal behavior.

Example:

```ruby
def load_weapon(path, retries: 3)
  # ...
end
```

Do not expose dozens of options merely to make a method "flexible."

Configuration is complexity.

---

## Keyword Rest Arguments

Avoid:

```ruby
def process(**options)
```

when the accepted options are known.

Explicit keywords improve:

- Documentation.
- Error detection.
- Editor support.
- Refactoring.

Use `**options` when forwarding or genuinely accepting extensible options.

---

## Argument Forwarding

Use Ruby's argument forwarding when a method truly forwards its arguments.

Example:

```ruby
def initialize(...)
  super
end
```

or:

```ruby
def call(...)
  target.call(...)
end
```

if supported by the project's Ruby version.

Do not use forwarding when explicit arguments better communicate the API.

---

## Blocks

Use blocks for operations naturally expressed as behavior passed to another method.

Good:

```ruby
weapons.each do |weapon|
  render(weapon)
end
```

Do not create callback-heavy APIs merely to seem idiomatic.

Blocks should simplify control flow, not hide it.

---

## Enumerable

Use Enumerable methods when they clearly express intent.

Good:

```ruby
enabled_weapons = weapons.select(&:enabled?)
```

and:

```ruby
names = weapons.map(&:name)
```

Do not force every loop into a chain.

A normal loop may be clearer when:

- Several side effects occur.
- Multiple branches exist.
- Early exit matters.
- State changes across iterations.

---

## Avoid Enumerable Gymnastics

Bad:

```ruby
result = weapons
  .select { |weapon| weapon.enabled? }
  .group_by(&:category)
  .transform_values do |group|
    group
      .sort_by(&:name)
      .map { |weapon| serialize(weapon) }
  end
```

may be perfectly reasonable if the transformation is genuinely this pipeline.

But if the chain becomes difficult to inspect or debug, use intermediate variables or a loop.

Do not optimize for the longest elegant-looking method chain.

---

## `map`

Use `map` when transforming every item.

Do not use `map` merely for side effects.

Bad:

```ruby
weapons.map do |weapon|
  save(weapon)
end
```

when the returned array is ignored.

Prefer:

```ruby
weapons.each do |weapon|
  save(weapon)
end
```

---

## `each`

Use `each` for iteration with side effects.

Do not collect a useless result.

Choose methods based on their semantics.

---

## `select` and `reject`

Use:

```ruby
select
reject
```

for filtering.

Do not manually push values into another array if a simple filter expresses the operation clearly.

---

## `filter_map`

When supported by the Ruby version, use `filter_map` when mapping and discarding `nil` values is naturally one operation.

Example:

```ruby
weapons.filter_map do |asset|
  parse_weapon(asset)
end
```

Do not introduce it if project support does not include the required Ruby version.

---

## `find`

Use:

```ruby
weapons.find { |weapon| weapon.id == id }
```

for first-match lookup.

Do not use:

```ruby
weapons.select { ... }.first
```

when `find` communicates intent directly.

---

## `any?`, `all?`, and `none?`

Use predicate Enumerable methods.

Good:

```ruby
weapons.any?(&:enabled?)
```

Avoid manual flags:

```ruby
enabled = false

weapons.each do |weapon|
  enabled = true if weapon.enabled?
end
```

unless more complex control flow is involved.

---

## `each_with_object`

Use it when accumulating a structure clearly.

Example:

```ruby
weapons_by_id = weapons.each_with_object({}) do |weapon, result|
  result[weapon.id] = weapon
end
```

But prefer built-ins like:

```ruby
weapons.to_h { |weapon| [weapon.id, weapon] }
```

when supported and clearer.

---

## `reduce`

Use `reduce` for real reductions.

Good:

```ruby
total_damage = weapons.sum(&:damage)
```

may be better than:

```ruby
weapons.reduce(0) do |total, weapon|
  total + weapon.damage
end
```

when `sum` exists.

Do not use `reduce` to show functional-programming cleverness.

---

## Prefer Specific Enumerable APIs

Use:

```ruby
sum
tally
group_by
index_by
index_with
```

when available and appropriate.

If using Rails/ActiveSupport helpers, do not assume they exist in plain Ruby.

Know whether an API comes from:

- Ruby core.
- Standard library.
- ActiveSupport.
- Another gem.

---

## ActiveSupport Awareness

In Rails projects, methods such as:

```ruby
blank?
present?
presence
index_by
deep_symbolize_keys
```

may be available.

Do not use ActiveSupport APIs in plain Ruby libraries unless ActiveSupport is an explicit dependency.

Do not accidentally make library code depend on Rails globally.

---

## Guard Clauses

Prefer guard clauses over deep nesting.

Good:

```ruby
def load_weapon(id)
  return unless id
  return unless enabled?

  repository.find(id)
end
```

Avoid:

```ruby
def load_weapon(id)
  if id
    if enabled?
      repository.find(id)
    end
  end
end
```

Keep the happy path visible.

---

## Avoid Excessive Guard Clauses

Do not turn every method into ten early returns.

If several conditions represent one conceptual validation, grouping them may be clearer.

Readable flow matters more than rigid adherence to a rule.

---

## `unless`

Use `unless` for simple negative conditions.

Good:

```ruby
return unless weapon
```

Avoid:

```ruby
unless weapon.nil? == false
```

Do not use `unless` with an `else` if it makes logic difficult to read.

This:

```ruby
if enabled
  ...
else
  ...
end
```

is usually clearer than:

```ruby
unless disabled
  ...
else
  ...
end
```

---

## Negation

Avoid double negatives.

Bad:

```ruby
unless !weapon.disabled?
```

Prefer:

```ruby
if weapon.disabled?
```

or an appropriately named positive predicate.

---

## Ternaries

Use ternaries for simple expressions.

Good:

```ruby
label = enabled? ? "Enabled" : "Disabled"
```

Avoid nested ternaries.

Use `if`/`case` for substantial branching.

---

## `case`

Use `case` when branching on one conceptual value.

Good:

```ruby
case state
when :ready
  start
when :loading
  wait
when :failed
  report_error
end
```

Do not replace simple two-way conditionals with `case` without a reason.

---

## Pattern Matching

Use pattern matching where it materially clarifies structured data or variants.

Example:

```ruby
case result
in { status: :ok, weapon: }
  weapon
in { status: :not_found }
  nil
end
```

Do not use pattern matching merely because modern Ruby supports it.

Simple hashes and conditionals may be easier for many cases.

---

## Symbols vs Strings

Use symbols for internal identifiers and keys where appropriate.

Use strings for user-facing or external textual data.

Do not convert external string keys to symbols without understanding:

- Source format.
- Memory behavior.
- API expectations.

Follow project convention.

---

## Hash Keys

Use consistent key types.

Avoid mixing:

```ruby
{
  "name" => "Rifle",
  damage: 42
}
```

without a reason.

At external boundaries, preserve the expected shape or normalize it once.

---

## Hash Syntax

Prefer modern symbol-key syntax:

```ruby
{
  name: "Rifle",
  damage: 42
}
```

over:

```ruby
{
  :name => "Rifle",
  :damage => 42
}
```

unless maintaining established style or needing non-symbol keys.

---

## Keyword Shorthand

When supported by the Ruby version and clear:

```ruby
{
  name:,
  damage:
}
```

can reduce repetition.

Do not introduce it if the project targets an older Ruby version or consistently avoids it.

---

## Hash Access

Use:

```ruby
hash.fetch(:key)
```

when absence is a bug or must be distinguished from `nil`.

Use:

```ruby
hash[:key]
```

when missing keys naturally return `nil`.

Do not use one mechanically everywhere.

---

## `fetch`

Good:

```ruby
timeout = config.fetch(:timeout, 30)
```

when the default applies only to missing keys.

This differs from:

```ruby
timeout = config[:timeout] || 30
```

when `false` or `nil` may have meaning.

Use semantics intentionally.

---

## Nil

Use `nil` for expected absence where idiomatic.

Do not wrap every nullable value in a custom Maybe/Option abstraction.

Ruby already has established nil handling.

Use stronger domain objects when absence has complex meaning.

---

## Safe Navigation

Use:

```ruby
weapon&.metadata&.name
```

when missing values are genuinely acceptable.

Do not create long safe-navigation chains to hide broken invariants.

Bad:

```ruby
package&.data&.weapon&.stats&.damage
```

if all of those should exist after validation.

Validate or normalize once.

---

## `dig`

Use `dig` for genuinely nested optional hash/array lookup.

Example:

```ruby
damage = data.dig(:weapon, :stats, :damage)
```

Do not use it to silently absorb missing required fields.

If absence is invalid, fail explicitly.

---

## `nil?`

Use `nil?` when distinguishing nil from other falsey values matters.

Remember Ruby has only two falsey values:

```text
nil
false
```

Do not write JavaScript-style truthiness assumptions involving `0` or `""`.

Both are truthy in Ruby.

---

## `||`

Use `||` for fallback only when `false` should also trigger the fallback.

Be careful:

```ruby
enabled = config[:enabled] || true
```

will turn explicit `false` into `true`.

Use key checks or `fetch` when false is valid.

---

## `||=`

Use memoization idioms carefully.

This:

```ruby
@value ||= calculate
```

does not memoize `false` or `nil`.

If those are valid results, use explicit initialization tracking.

Do not apply `||=` mechanically.

---

## Memoization

Memoize when:

- Computation is meaningfully expensive.
- Result should remain stable.
- Object lifetime semantics make caching correct.

Do not memoize every method automatically.

Caching adds state.

---

## `defined?`

When nil/false are valid memoized values, an explicit check may be appropriate:

```ruby
return @value if defined?(@value)

@value = calculate
```

Use this only when needed.

Do not complicate ordinary memoization unnecessarily.

---

## Arrays

Use arrays for ordered collections.

Do not create custom collection classes without meaningful domain behavior.

Example:

```ruby
weapons = []
```

is perfectly professional.

---

## Sets

Use `Set` when uniqueness or repeated membership checks matter.

Remember to:

```ruby
require "set"
```

in plain Ruby environments where necessary.

Do not use a Set for tiny lists without a reason.

---

## Hashes

Use hashes for key-value lookup and flexible structured data.

Do not use hashes as unvalidated anonymous domain objects throughout a large system when stable shape matters.

A class, `Struct`, `Data`, or typed object may be clearer once the shape becomes important.

---

## Arrays of Hashes

Arrays of hashes are fine at transport and serialization boundaries.

Do not automatically convert every small payload into a class.

Conversely, do not let unstructured hashes spread throughout domain logic if their required keys are important.

---

## Strings

Use normal strings for text.

Prefer interpolation:

```ruby
"Unable to load package #{path}"
```

over:

```ruby
"Unable to load package " + path
```

when interpolation is clearer.

---

## Single vs Double Quotes

Follow project style.

Use single quotes when interpolation and escapes are unnecessary if the project prefers that style.

Use double quotes when interpolation is required.

Do not reformat entire files merely to change quote style.

---

## Frozen Strings

Respect the project's `frozen_string_literal` convention.

If files use:

```ruby
# frozen_string_literal: true
```

follow it.

Do not add or remove the magic comment across unrelated files as style cleanup.

---

## Mutating Strings

Know whether a string may be frozen.

Avoid mutating literals or shared constants unexpectedly.

Use:

```ruby
dup
```

only when independent mutable storage is genuinely needed.

Do not `dup` everything defensively.

---

## String Building

For a small number of values, interpolation is clear.

For large repeated concatenation, use an appropriate mutable buffer.

Do not optimize string construction without evidence.

---

## Heredocs

Use heredocs for multi-line text.

Prefer squiggly heredocs for indentation when supported:

```ruby
message = <<~TEXT
  Unable to load weapon.
  Check the asset path and try again.
TEXT
```

Do not create awkward concatenated string fragments where a heredoc is clearer.

---

## Regular Expressions

Use regex when it expresses the problem clearly.

Example:

```ruby
ASSET_PATH_PATTERN = %r{\A/Game/[A-Za-z0-9_/]+\z}
```

Do not use giant regexes when straightforward parsing is easier.

Name complex expressions.

---

## Anchors

For whole-string validation, understand the difference between:

```ruby
^
$
```

and:

```ruby
\A
\z
```

In Ruby, `\A` and `\z` are often safer for validating the entire string.

Do not copy regex habits blindly from other languages.

---

## Match APIs

Use modern clear APIs such as:

```ruby
pattern.match?(value)
```

when only a boolean is needed.

Avoid creating a MatchData object unnecessarily.

Do not use regex at all when a simple string operation works.

---

## Exceptions

Use exceptions for exceptional failures.

Do not use exceptions for ordinary expected branching when a normal value communicates the state better.

Examples of reasonable exceptions:

- Invalid configuration.
- I/O failure.
- External service failure.
- Parsing failure where caller expects valid input.

Routine lookup absence may simply return `nil`.

---

## Rescue Specific Exceptions

Avoid:

```ruby
rescue StandardError
```

throughout ordinary application logic.

Prefer specific failures:

```ruby
rescue Errno::ENOENT
```

when that is the expected error.

Broad rescue can hide programming bugs.

---

## Never Rescue `Exception`

Do not write:

```ruby
rescue Exception
```

in normal application code.

`Exception` includes system-level conditions such as:

- `SystemExit`
- `Interrupt`

that should generally propagate.

Use `StandardError` only at true boundaries when broad handling is appropriate.

---

## Bare Rescue

Remember:

```ruby
rescue
```

rescues `StandardError`.

Even so, avoid bare rescue when specific expected exceptions are known.

Explicitness improves maintenance.

---

## Do Not Swallow Exceptions

Bad:

```ruby
begin
  load_weapon
rescue
  nil
end
```

unless converting failure to absence is deliberately part of the API.

Silent failure makes debugging difficult.

---

## Re-Raising

Use:

```ruby
raise
```

to re-raise the current exception.

Do not write:

```ruby
raise error
```

without understanding traceback behavior.

Preserve original failure context.

---

## Wrapping Exceptions

When translating errors, retain useful context.

Example:

```ruby
rescue Errno::ENOENT => error
  raise PackageLoadError,
        "Unable to load #{path}: #{error.message}"
end
```

If the project uses `cause`, custom error hierarchy, or framework error handling, follow that convention.

---

## Custom Errors

Create domain-specific errors when callers genuinely need to distinguish them.

Example:

```ruby
class PackageLoadError < StandardError; end
```

Do not create:

```text
BaseWeaponError
WeaponProcessingError
WeaponValidationProcessingError
WeaponValidationFieldError
```

unless the hierarchy provides real value.

---

## Errors vs Results

Ruby code often uses exceptions rather than explicit Result wrappers.

Do not introduce a custom:

```ruby
Success(...)
Failure(...)
```

result framework unless the project already uses one or the domain benefits from it.

Likewise, do not force exceptions where established code uses result objects.

Match the codebase.

---

## Logging

Use the project's established logger.

In Rails:

```ruby
Rails.logger
```

may be appropriate.

In plain Ruby:

```ruby
Logger
```

or an existing logging gem may be used.

Do not use:

```ruby
puts
p
pp
```

for production diagnostics unless output is intentionally user-facing.

---

## Logging Context

Log meaningful identifiers.

Good:

```ruby
logger.warn(
  "Unable to resolve weapon asset",
  asset_path: asset_path
)
```

depending on logger capabilities.

Avoid:

```ruby
logger.info "Starting process"
logger.info "Processing"
logger.info "Finished process"
```

without operational value.

---

## Duplicate Logging

Do not log an exception at every level and then re-raise it.

Choose the layer that has meaningful context or owns observability.

Duplicate stack traces create noise.

---

## `puts`

Use `puts` for:

- CLI output.
- Small scripts.
- Intentional stdout output.

Do not use it as application logging when a logger exists.

Remove debugging `puts` before finishing.

---

## `p` and `pp`

Use them while debugging.

Do not leave them in production code.

---

## Comments

Do not narrate the code.

Bad:

```ruby
# Check if the weapon exists
return nil if weapon.nil?

# Return the weapon name
weapon.name
```

Good:

```ruby
return unless weapon

weapon.name
```

Write comments when they explain:

- Why a workaround exists.
- A framework limitation.
- A performance tradeoff.
- A strange domain rule.
- External compatibility.
- Non-obvious behavior.

Prefer **why**, not **what**.

---

## Avoid AI-Looking Comments

Avoid phrases like:

```text
This method is responsible for...
This ensures that...
The following logic handles...
In order to...
It is important to note that...
This provides a robust and flexible solution...
```

Bad:

```ruby
# This method is responsible for ensuring that the weapon
# data is properly validated before processing continues.
```

Better:

```ruby
# Legacy manifests may omit fields present in current exports.
```

---

## Documentation

Use YARD or project-standard documentation for public library APIs when useful.

Do not document every obvious private method.

Bad:

```ruby
# Returns the weapon name.
#
# @return [String] the weapon name
def name
  @name
end
```

The method already explains itself.

---

## Avoid Decorative Comments

Do not add:

```ruby
# ========================================
# INITIALIZATION
# ========================================
```

unless the project deliberately uses this style.

Good code structure should usually make these unnecessary.

---

## Metaprogramming

Ruby's metaprogramming is powerful.

Use it when it removes real repetitive behavior or supports framework-level DSLs.

Do not use it for ordinary application logic.

Avoid:

```ruby
define_method(method_name) do
  # ...
end
```

when three explicit methods would be easier to understand.

---

## `send`

Prefer direct calls:

```ruby
weapon.name
```

over:

```ruby
weapon.send(:name)
```

when the method is known.

Use `public_send` instead of `send` when dynamic public dispatch is genuinely required.

`send` bypasses visibility and should not be a casual convenience.

---

## `public_send`

Use dynamic dispatch only when method names are truly data.

Example:

```ruby
record.public_send(field_name)
```

may be appropriate for a controlled field list.

Do not use user-controlled method names without validation.

---

## `method_missing`

Avoid `method_missing` unless implementing a genuinely dynamic proxy or DSL.

When used, implement:

```ruby
respond_to_missing?
```

correctly.

Do not use `method_missing` simply to avoid writing explicit methods.

It makes:

- Debugging harder.
- Static analysis harder.
- IDE support worse.
- Typos harder to detect.

---

## `define_method`

Use when method generation meaningfully removes repetitive boilerplate.

Do not hide important business behavior behind loops that dynamically define dozens of methods.

Explicit code is often easier to search and maintain.

---

## `class_eval` and `instance_eval`

Use sparingly.

These are useful for:

- DSLs.
- Framework internals.
- Carefully controlled metaprogramming.

Do not use them in ordinary business logic when normal method calls suffice.

---

## `eval`

Avoid `eval`.

Never evaluate untrusted input.

Use normal Ruby objects, parsing, and dispatch instead.

Dynamic code execution should be exceptionally rare.

---

## Monkey Patching

Do not monkey-patch core or third-party classes casually.

Bad:

```ruby
class String
  def weaponize
    # ...
  end
end
```

for application-specific behavior.

Prefer:

```ruby
WeaponName.normalize(value)
```

or a domain object.

Monkey patches can cause:

- Global behavior changes.
- Dependency conflicts.
- Difficult debugging.
- Version upgrade issues.

---

## Refinements

Refinements can limit monkey-patch scope.

Use them only when they genuinely improve a specialized integration.

Do not introduce refinements for normal domain helpers where explicit functions are clearer.

---

## Open Classes

Ruby permits reopening classes.

Do not rely on this casually across unrelated files.

Framework extensions should follow established project conventions.

Keep ownership of behavior understandable.

---

## Constants

Use constants for genuine shared fixed values.

Good:

```ruby
MAX_RETRY_ATTEMPTS = 3
DEFAULT_TIMEOUT = 30
```

Do not define constants for trivial values:

```ruby
ZERO = 0
EMPTY_STRING = ""
```

A constant name should communicate domain meaning.

---

## Mutable Constants

Ruby constants can reference mutable objects.

If a constant should not change:

```ruby
SUPPORTED_EXTENSIONS = %w[.uasset .umap].freeze
```

may be appropriate.

Do not freeze everything mechanically if mutation is intentional.

---

## Deep Freezing

Ruby's `freeze` is generally shallow.

Do not assume:

```ruby
CONFIG.freeze
```

freezes every nested object.

If deep immutability matters, use an intentional mechanism.

Do not build deep-freeze machinery without an actual need.

---

## Global Variables

Avoid:

```ruby
$global_state
```

in normal production code.

Global variables create hidden dependencies and difficult tests.

Prefer explicit ownership.

---

## Class Variables

Avoid:

```ruby
@@value
```

unless inheritance semantics are deliberately understood.

Class variables are shared through inheritance in ways that often surprise maintainers.

Class instance variables are frequently clearer.

---

## Class Instance Variables

Use class instance variables for state owned specifically by the class object when necessary.

Do not turn classes into global mutable registries without need.

---

## Singleton Methods

Class-level methods are appropriate when behavior belongs to the class rather than instances.

Example:

```ruby
class Weapon
  def self.parse(data)
    # ...
  end
end
```

Do not create large static-style utility classes filled entirely with class methods.

A module may be clearer for stateless namespacing.

---

## `class << self`

Use if the project prefers it for several class methods.

Do not switch between:

```ruby
def self.foo
```

and:

```ruby
class << self
```

as unrelated style cleanup.

Follow local convention.

---

## State

Keep mutable state narrow.

Avoid objects with many instance variables changing from unrelated methods.

If state transitions become difficult to understand, reconsider the object's responsibility.

Do not make everything immutable if mutation is natural.

---

## Side Effects

Keep side effects obvious.

Examples:

- Database writes.
- File writes.
- Network requests.
- Logging.
- Email.
- Jobs.

Do not hide major side effects inside innocent-looking accessors.

Bad:

```ruby
def status
  sync_remote_state
  @status
end
```

A reader expects `status` to read, not perform remote synchronization.

---

## Query vs Command Methods

Where practical, distinguish:

- Methods that return information.
- Methods that change state.

Ruby does not require strict CQRS-style separation, but surprising side effects should be avoided.

---

## Mutating Methods

When a method changes its receiver significantly, naming should make intent clear.

Do not overuse `!` just because state changes.

Follow Ruby ecosystem conventions.

---

## File Organization

Keep files cohesive.

Do not enforce one-method-per-file.

A Ruby class or module usually gets its own file in larger applications, especially Rails, but small related structures may reasonably live together.

Follow project conventions.

---

## File Names

Use `snake_case.rb`.

Type/file relationships generally follow:

```text
WeaponImporter
weapon_importer.rb
```

especially with Zeitwerk/Rails autoloading.

Do not violate autoloader conventions.

---

## Namespacing

Use modules to reflect meaningful domain/package structure.

Example:

```ruby
module Weapons
  class Importer
  end
end
```

Do not create deep namespaces like:

```text
Application::Services::Managers::Processors::Weapons::Importer
```

without a real architectural need.

---

## Rails Autoloading

When using Rails/Zeitwerk, file names and constants must correspond correctly.

Do not create clever constant layouts that fight autoloading.

Follow conventional paths.

---

## Requires

In plain Ruby, use `require` intentionally.

Do not rely on incidental load order.

In Rails applications, follow autoloading conventions instead of manually requiring every application file.

---

## `require_relative`

Use for local files in plain Ruby projects where appropriate.

Do not use it throughout Rails autoloaded code unless the project explicitly requires it.

---

## Gems

Use the project's dependency management.

Do not add a gem before checking whether:

- Ruby already provides the functionality.
- Rails/ActiveSupport already provides it.
- Another project dependency already solves it.
- The feature is simple enough to implement directly.
- The gem is maintained.

Do not install a gem for five straightforward lines of code.

---

## Bundler

Use Bundler normally.

Do not manually modify `Gemfile.lock`.

Use:

```text
bundle install
bundle update <specific gem>
```

according to project workflow.

Avoid broad dependency updates during unrelated feature work.

---

## Gem Versions

Do not casually loosen or tighten gem constraints without understanding compatibility.

Dependency changes can affect the whole application.

---

## Rails

When working in Rails, follow Rails conventions before generic Ruby preferences.

Prefer:

- Conventional controllers.
- ActiveRecord patterns.
- ActiveJob.
- ActiveSupport where already available.
- RESTful routes.
- Standard validations.
- Framework lifecycle hooks.

Do not wrap Rails APIs unnecessarily.

---

## Rails Models

Keep domain behavior near the model when it naturally belongs there.

Do not automatically move every method into:

```text
app/services
```

merely because "models should be skinny."

A model with meaningful domain behavior can be healthy.

Avoid both giant god models and anemic models surrounded by dozens of trivial services.

---

## Fat Model / Skinny Controller

Treat this as a guideline, not a commandment.

Controllers should generally coordinate HTTP concerns.

Domain behavior belongs in:

- Models.
- Domain objects.
- Purpose-built objects.

depending on complexity.

Do not create service objects just to reduce line count.

---

## Controllers

Controllers should generally:

- Read request data.
- Authorize.
- Call domain/application behavior.
- Choose response.

Avoid major business logic in controllers.

But do not create three abstraction layers for a simple action.

---

## Service Objects

Use service objects when an operation:

- Coordinates several domain objects.
- Has several side effects.
- Represents an application-level use case.
- Does not naturally belong to one model.

Good:

```ruby
class ImportWeapon
  def initialize(repository:, asset_reader:)
    @repository = repository
    @asset_reader = asset_reader
  end

  def call(path)
    # meaningful orchestration
  end
end
```

Do not create one service class per model method automatically.

---

## Avoid Service Object Explosion

Bad architecture:

```text
CreateWeaponService
UpdateWeaponService
DeleteWeaponService
FindWeaponService
ListWeaponsService
ValidateWeaponService
```

when ActiveRecord or simple model methods already express these operations cleanly.

Service objects should capture real workflows.

---

## `call`

A single `call` method is idiomatic for operation objects.

Use it when the class has one obvious action.

Do not force every object to expose `call`.

Objects with several meaningful behaviors can have appropriately named methods.

---

## Query Objects

Use query objects when database queries are:

- Complex.
- Reused.
- Difficult to compose.
- Important enough to test independently.

Do not create a query object for:

```ruby
Weapon.where(active: true)
```

unless project architecture explicitly requires it.

---

## Scopes

Use ActiveRecord scopes for reusable composable query fragments.

Good:

```ruby
scope :active, -> { where(active: true) }
```

Avoid scopes containing large business workflows or side effects.

Scopes should remain query-oriented.

---

## ActiveRecord Callbacks

Use callbacks carefully.

Good uses may include:

- Maintaining simple local invariants.
- Normalizing directly owned values.
- Framework lifecycle concerns.

Avoid hiding major business workflows inside:

```ruby
after_save
after_commit
before_validation
```

when explicit orchestration would be clearer.

---

## Callback Chains

Too many callbacks make execution order difficult to understand.

If saving one model unexpectedly:

- Sends email.
- Creates jobs.
- Updates unrelated models.
- Calls external services.

the design may be too implicit.

Prefer explicit application-level workflows for major side effects.

---

## Validations

Use model validations for invariants appropriate to the model.

Example:

```ruby
validates :name, presence: true
```

Do not duplicate the exact same validation manually in multiple layers without a reason.

Remember database constraints may still be necessary for integrity.

---

## Database Constraints

Rails validations alone do not protect against all concurrency or direct database writes.

Use database constraints for critical integrity where appropriate.

Do not duplicate every application validation as a database check if the constraint does not belong there.

---

## ActiveRecord Bang Methods

Understand:

```ruby
save
save!
update
update!
create
create!
```

Use the version matching failure semantics.

Do not call `save!` blindly everywhere.

Use it when failure should raise.

---

## `find` vs `find_by`

Use:

```ruby
find(id)
```

when absence should raise `ActiveRecord::RecordNotFound`.

Use:

```ruby
find_by(...)
```

when absence should return `nil`.

Do not rescue `RecordNotFound` immediately just to turn it into nil when `find_by` already expresses the desired behavior.

---

## N+1 Queries

Be aware of N+1 queries.

Use:

```ruby
includes
preload
eager_load
```

according to actual query behavior.

Do not eager-load every association automatically.

Load what the operation needs.

---

## `pluck`

Use `pluck` when only database column values are needed.

Example:

```ruby
Weapon.where(active: true).pluck(:id)
```

can be cheaper than loading full models.

Do not replace model loading with `pluck` when model behavior is needed.

---

## `select`

Remember ActiveRecord's `select` differs from Enumerable's `select` depending on context.

Be explicit enough that readers understand whether filtering occurs:

- In SQL.
- In Ruby.

Avoid accidentally loading huge datasets for Ruby-side filtering.

---

## `exists?`

Use database existence checks:

```ruby
scope.exists?
```

instead of loading records merely to check whether any exist.

Avoid:

```ruby
scope.to_a.any?
```

when a database query can answer directly.

---

## `count`, `size`, and `length`

Understand ActiveRecord differences.

Do not switch between them blindly.

Depending on loaded state:

- `count` may query.
- `size` may use loaded data or count.
- `length` loads records.

Use the one matching actual needs.

---

## Transactions

Use transactions for related database writes that must succeed or fail together.

Keep transaction scope small.

Do not hold transactions open while performing slow external network calls unless semantics require it.

---

## Locks

Use pessimistic/optimistic locking when actual concurrency problems justify it.

Do not add locks preemptively.

Concurrency complexity should address a real race.

---

## Jobs

Use background jobs for work that:

- Should not block the request.
- Is slow.
- Can be retried.
- Can run asynchronously.

Do not enqueue trivial operations merely because ActiveJob exists.

---

## Job Arguments

Pass stable identifiers rather than large mutable objects when the job system serializes arguments.

For Rails/GlobalID-aware models, follow project conventions.

Do not assume in-memory state survives until job execution.

---

## Job Idempotency

Design retryable jobs so repeated execution does not create incorrect duplicate side effects where practical.

Do not add complex distributed-lock machinery unless duplicate execution is an actual risk.

---

## Mailers

Keep mailer code focused on email rendering and delivery concerns.

Do not bury unrelated business logic inside mailer methods.

---

## Routes

Prefer conventional RESTful routes when they fit.

Do not create custom action names for normal CRUD operations without reason.

Use explicit custom routes for real domain actions.

---

## Views

Keep templates focused on presentation.

Do not perform database queries from views.

Avoid substantial business logic in ERB.

A simple condition is fine.

A multi-step domain calculation probably belongs elsewhere.

---

## Helpers

Use Rails helpers for presentation-specific reusable logic.

Do not create gigantic helper modules containing unrelated domain behavior.

---

## Partials

Use partials for meaningful reusable or cohesive view fragments.

Do not extract every five lines of markup solely to reduce file length.

Too many tiny partials make templates hard to follow.

---

## View Components

If the project uses ViewComponent or similar abstractions, follow that system.

Do not introduce component libraries for one small fragment unless the project already uses them.

---

## Strong Parameters

Use Rails strong parameters correctly.

Do not permit:

```ruby
params.permit!
```

unless the security model deliberately allows all fields.

Permit only expected input.

---

## Mass Assignment

Do not pass untrusted hashes directly into models without strong parameter or validation boundaries.

Be explicit about allowed fields.

---

## Serialization

Use the project's established serializer approach.

Possible options include:

- `as_json`.
- ActiveModel serializers.
- Blueprinter.
- Jbuilder.
- JSONAPI serializers.
- Plain hashes.

Do not introduce a second serializer framework casually.

---

## JSON

Ruby's JSON parser returns string keys by default.

Do not assume symbol keys unless explicitly configured or transformed.

Validate untrusted external JSON before relying on required fields.

---

## YAML

Use safe YAML loading for untrusted input.

Avoid unsafe deserialization APIs with user-controlled YAML.

Ruby object deserialization can execute dangerous behavior.

---

## Marshal

Never load untrusted `Marshal` data.

`Marshal` is Ruby-object serialization, not a safe interchange format.

Use safer formats for untrusted boundaries.

---

## Security

Do not weaken security for convenience.

Avoid:

- `eval` on untrusted data.
- Unsafe YAML/Marshal loading.
- Shell interpolation with untrusted values.
- SQL interpolation.
- HTML injection.
- Mass assignment of uncontrolled fields.
- Path traversal.
- Secret logging.

Use established framework protections.

---

## SQL

Use parameterized queries.

Bad:

```ruby
Weapon.where(
  "name = '#{params[:name]}'"
)
```

Good:

```ruby
Weapon.where(name: params[:name])
```

or:

```ruby
Weapon.where(
  "name = ?",
  params[:name]
)
```

Do not bypass ActiveRecord parameterization.

---

## Shell Commands

Avoid interpolating untrusted values into shell strings.

Bad:

```ruby
system("tool --file #{path}")
```

Prefer argument separation:

```ruby
system("tool", "--file", path)
```

when appropriate.

Understand whether the API invokes a shell.

---

## Open3

Use `Open3` when stdout/stderr/status handling matters.

Do not use it for every trivial command if `system` or project abstraction is enough.

---

## Backticks

Avoid backticks for substantial production subprocess handling.

They hide exit status unless explicitly checked and encourage shell interpolation.

Use a clearer subprocess API when behavior matters.

---

## File Paths

Use:

```ruby
File
FileUtils
Pathname
```

according to project style.

Do not manually concatenate path separators.

Good:

```ruby
File.join(root, "config", "settings.yml")
```

Do not assume `/` or `\` manually in portable code.

---

## Pathname

`Pathname` can make path operations expressive.

Do not introduce it into one function if the entire project consistently uses strings and `File` helpers unless it meaningfully improves the code.

---

## File I/O

Use blocks so resources close automatically.

Good:

```ruby
File.open(path, "r") do |file|
  parse(file)
end
```

Do not manually open and forget to close.

---

## Convenience File APIs

For small files:

```ruby
File.read(path)
File.write(path, content)
```

may be simpler.

Do not create stream-heavy code for tiny files without need.

---

## Encoding

Be aware of Ruby string encodings.

Do not assume every external byte sequence is valid UTF-8.

For text protocols and files, set or validate encoding appropriately.

Do not add encoding conversions without understanding the data source.

---

## Binary Data

Use binary mode where necessary:

```ruby
File.binread(path)
```

Do not process arbitrary binary data through text assumptions.

---

## Time

Use the framework's time abstractions when appropriate.

In Rails:

```ruby
Time.current
```

is often preferable to:

```ruby
Time.now
```

because it respects configured application time zones.

Do not use Rails helpers in plain Ruby code.

---

## Dates

Use:

```ruby
Date
Time
DateTime
```

according to project conventions.

Do not use `DateTime` automatically when `Time` is the established type.

---

## Time Zones

Avoid manual timezone math.

Use Rails time zone helpers or established libraries.

Do not parse and offset timestamps by hand unless implementing a protocol.

---

## Randomness

Use:

```ruby
rand
```

for ordinary non-security-sensitive randomness.

Use:

```ruby
SecureRandom
```

for security-sensitive IDs, tokens, or secrets.

Do not use predictable randomness for authentication values.

---

## UUIDs

Use established UUID facilities or `SecureRandom.uuid`.

Do not build custom random identifiers without a domain need.

---

## Threading

Do not add threads unless the workload benefits and the application architecture supports them.

Ruby implementation details matter.

Concurrency behavior differs between:

- MRI.
- JRuby.
- TruffleRuby.

Do not assume CPU-bound threads provide parallel execution everywhere.

---

## Mutexes

Use `Mutex` when real shared mutable state requires synchronization.

Do not add mutexes around state that can simply have one owner.

Keep synchronization narrow.

---

## Fibers

Use fibers when the framework/runtime intentionally uses them.

Do not introduce custom fiber scheduling for ordinary application code.

---

## Ractors

Use Ractors only when the supported Ruby version and workload justify them.

They impose strict object-sharing semantics and are not a general-purpose replacement for threads.

Do not add Ractors merely because they offer parallelism.

---

## Async Ruby

If the project uses an async framework, follow its conventions.

Do not introduce an async runtime into a synchronous Rails or Ruby project for one feature.

Concurrency architecture should remain consistent.

---

## External HTTP

Use the project's existing HTTP client.

Possible choices include:

```text
Net::HTTP
Faraday
HTTP.rb
HTTParty
RestClient
```

Do not add a second HTTP stack casually.

---

## Timeouts

External requests should generally have sensible connection/read timeouts.

Do not allow indefinite network hangs unless deliberately required.

Do not choose tiny arbitrary timeouts.

---

## Retries

Retry only plausibly transient failures.

Appropriate examples:

- Timeouts.
- Temporary connection failures.
- 429 responses.
- Some 5xx responses.

Do not retry:

- Invalid input.
- Authentication errors needing new credentials.
- Deterministic parsing failures.

Keep retries bounded.

---

## JSON API Responses

Do not blindly trust remote response shape.

Validate the fields required by the operation.

Do not create heavy schema machinery when a few explicit checks are enough.

Use established validation libraries if the project already has them.

---

## Memoization and Thread Safety

Remember instance-variable memoization may behave differently under concurrent access.

Do not add synchronization merely for a theoretically duplicated calculation unless correctness requires it.

But do not assume memoized mutable shared objects are safe under all concurrency models.

---

## Caching

Do not add caching without a real need.

Caching creates:

- Stale data.
- Invalidation rules.
- More state.
- Harder tests.

When using Rails cache or another cache, define:

- Key semantics.
- Expiration.
- Invalidation.
- Failure behavior.

Do not cache everything because a query looks expensive.

Measure first.

---

## Performance

Prefer readable code until performance matters.

Do not prematurely:

- Memoize every method.
- Add caching.
- Build custom object pools.
- Replace clear Enumerable code with obscure loops.
- Freeze everything.
- Micro-optimize allocations.
- Move code to C extensions.

Measure actual bottlenecks.

---

## Allocation Awareness

Ruby allocates frequently by design.

In true hot paths, avoid needless:

- Temporary strings.
- Arrays.
- Hashes.
- Regex MatchData.
- Symbol/string conversions.

Do not sacrifice ordinary readability for tiny allocation savings outside measured hot paths.

---

## Freeze

Use `freeze` when immutability is meaningful.

Do not add `.freeze` after every literal mechanically.

In codebases with `frozen_string_literal: true`, additional string freezing is often unnecessary.

---

## Lazy Enumerators

Use:

```ruby
lazy
```

for genuinely large or streaming pipelines.

Do not use lazy enumerators for a ten-element array.

They add complexity and overhead.

---

## Enumerators

Use enumerators when laziness, iteration abstraction, or external iteration provides real value.

Do not expose Enumerator APIs merely because they are idiomatic.

---

## Database Performance

For Rails, inspect query behavior before optimizing Ruby loops.

Often the biggest cost is:

- Query count.
- N+1 behavior.
- Loading unnecessary columns.
- Missing indexes.

Do not micro-optimize Ruby while issuing hundreds of unnecessary SQL queries.

---

## Indexes

When query performance matters, consider database indexes as part of the solution.

Do not add indexes blindly; understand:

- Selectivity.
- Write cost.
- Existing indexes.
- Query plans.

---

## Testing

Use the project's existing framework.

Common options include:

```text
RSpec
Minitest
```

Do not migrate test frameworks during unrelated work.

Test behavior rather than internal implementation.

---

## RSpec

Use descriptive examples.

Good:

```ruby
it "returns nil when the weapon does not exist" do
  # ...
end
```

Avoid:

```ruby
it "works" do
  # ...
end
```

Keep descriptions focused on behavior.

---

## RSpec `let`

Use `let` for setup that benefits from lazy reusable definition.

Do not create dozens of layered `let` values that require scrolling to understand the test.

Sometimes a local variable inside the example is clearer.

---

## `let!`

Use `let!` only when eager creation is necessary.

Do not replace every fixture setup with `let!`.

Hidden eager database writes can make tests slower and harder to follow.

---

## Subjects

Use implicit `subject` sparingly.

Bad:

```ruby
subject { described_class.new(foo, bar, baz) }
```

with many expectations far from construction can obscure what is under test.

Explicit objects are often clearer.

---

## Shared Examples

Use shared examples when several implementations genuinely must satisfy the same contract.

Do not use them merely to eliminate three repeated expectations.

Overusing shared examples can hide test behavior.

---

## Contexts

Use `context` for meaningful conditions.

Good:

```ruby
context "when the weapon is missing" do
```

Avoid deeply nested context trees.

Tests should remain easy to scan.

---

## Mocks

Do not mock everything.

Prefer real lightweight objects where practical.

Mock:

- External HTTP.
- Email.
- Payment providers.
- Expensive external systems.
- Time when necessary.

Do not mock plain hashes or simple domain objects.

---

## Message Expectations

Use interaction assertions when the interaction itself matters.

Avoid testing every internal call:

```ruby
expect(parser).to receive(:parse)
expect(mapper).to receive(:map)
expect(repository).to receive(:save)
```

when asserting the final result would better test behavior.

---

## `allow_any_instance_of`

Avoid:

```ruby
allow_any_instance_of(...)
```

or:

```ruby
expect_any_instance_of(...)
```

when possible.

These often indicate unclear object ownership and make tests broad.

Inject or construct the relevant collaborator instead.

---

## Stubbing Constants

Use:

```ruby
stub_const
```

carefully.

Do not restructure production code solely to make constant stubbing convenient.

---

## Time Tests

Use project-provided time helpers.

In Rails:

```ruby
travel_to
freeze_time
```

may be appropriate.

Do not depend on the real current clock in tests where timing affects behavior.

---

## Random Tests

Seed randomness or use deterministic inputs.

Do not allow tests to fail intermittently because random values happen to collide.

---

## Database Tests

Create only the records a test actually needs.

Avoid massive fixture graphs.

Use factories deliberately if the project uses FactoryBot.

---

## FactoryBot

Factories should produce valid useful defaults.

Do not build giant factories with dozens of callbacks and associations automatically.

Use traits for meaningful variants.

Avoid generating entire object graphs when one record is enough.

---

## Fixtures

If the project uses fixtures, follow that convention.

Do not introduce FactoryBot solely because it is popular.

Use the existing test data strategy.

---

## Integration Tests

Use integration/request/system tests for behavior spanning meaningful boundaries.

Do not test every small method through a browser.

Choose the lowest test level that confidently covers the behavior.

---

## System Tests

Use browser/system tests for:

- Navigation.
- JavaScript behavior.
- Forms.
- Full user workflows.

Do not use them to validate a pure string formatter.

---

## RuboCop

Respect the project's RuboCop configuration.

Do not disable cops merely to make generated code pass.

Avoid:

```ruby
# rubocop:disable all
```

or broad file-wide disables.

Use narrowly scoped exceptions only when justified.

---

## Metrics Cops

Do not mechanically extract methods/classes solely to satisfy:

```text
Metrics/MethodLength
Metrics/AbcSize
Metrics/ClassLength
```

if doing so makes the design worse.

The lint is a signal, not architecture.

Refactor where readability actually improves.

---

## Style Cops

Follow project conventions.

Do not rewrite unrelated code simply because your preferred RuboCop defaults differ.

---

## Auto-Correct

Be cautious with broad:

```text
rubocop -A
```

runs.

Keep changes scoped.

Avoid generating enormous unrelated style diffs.

---

## Formatting

Follow the project's formatter/linter.

Do not manually align assignment operators or hashes in a way auto-formatting will undo.

Keep source readable and compact.

---

## Parentheses

Follow project style.

Ruby often omits parentheses for simple DSL-like calls:

```ruby
validates :name, presence: true
```

But method calls with complex arguments may be clearer with parentheses.

Do not remove or add parentheses mechanically.

---

## Method Calls

Prefer clarity over clever omission.

This:

```ruby
load_weapon(path)
```

is often easier to distinguish from variable access than:

```ruby
load_weapon path
```

in ordinary application code.

DSLs are an exception.

Follow project style.

---

## Semicolons

Do not use semicolons to place multiple statements on one line.

Ruby permits it, but normal production Ruby rarely benefits.

---

## One-Line Methods

Short methods can sometimes use endless method syntax if supported and consistent:

```ruby
def loaded? = !@package.nil?
```

Do not convert every tiny method to endless syntax in a codebase that does not use it.

Readability and consistency matter.

---

## One-Line Conditionals

This is idiomatic:

```ruby
return unless weapon
```

Do not compress significant logic into postfix conditionals.

Bad:

```ruby
save_weapon(weapon) if weapon && weapon.valid? && repository.ready? && !dry_run?
```

Use a normal conditional when the condition becomes substantial.

---

## Modifier Conditionals

Use for short obvious statements.

Good:

```ruby
return nil unless weapon
```

Avoid:

```ruby
perform_complex_operation_with_many_side_effects if complicated_condition
```

when important logic becomes easy to miss.

---

## Endless Chaining

Ruby's syntax makes chains attractive.

Do not create enormous:

```ruby
foo
  .bar
  .baz
  .then
  .yield_self
  .transform_values
  .filter_map
```

pipelines simply because they read elegantly in isolation.

Use intermediate names when they clarify domain meaning.

---

## `then` / `yield_self`

Use when a transformation pipeline genuinely becomes clearer.

Do not use them for ordinary sequential code merely to appear functional.

---

## `tap`

Use `tap` for configuring or observing an object while returning it.

Good:

```ruby
config = Config.new.tap do |value|
  value.timeout = 30
end
```

Avoid using `tap` for unrelated side effects that surprise readers.

---

## `then`

Useful for transformation:

```ruby
path
  .expand_path
  .then { |value| load_file(value) }
```

But a local variable may be clearer.

Use judgment.

---

## `with_object`

Use where it makes accumulation clear.

Do not introduce it when a simple literal or helper is easier.

---

## Destructuring

Use parallel assignment and pattern matching when the shape is obvious.

Example:

```ruby
name, value = line.split(":", 2)
```

Do not destructure large positional structures that should have named fields.

---

## Multiple Return Values

Ruby effectively returns arrays for multiple values.

This is fine:

```ruby
width, height = dimensions
```

Avoid returning six positional values whose meaning is difficult to remember.

Use an object or hash with named fields.

---

## Constants and Magic Values

Give names to meaningful values.

Good:

```ruby
MAX_RETRY_ATTEMPTS = 3
DEFAULT_TIMEOUT_SECONDS = 30
```

Do not create constants merely to avoid literals.

---

## Boolean Parameters

Avoid:

```ruby
process_weapon(weapon, true, false, true)
```

Prefer keywords:

```ruby
process_weapon(
  weapon,
  validate: true,
  normalize: false,
  include_metadata: true
)
```

if those options genuinely need caller control.

Do not create configurable flags for fixed behavior.

---

## Symbolic Modes

Where a boolean substantially changes behavior, a symbolic mode can be clearer.

Instead of:

```ruby
load_package(path, true)
```

consider:

```ruby
load_package(path, mode: :cached)
```

when there are meaningful modes.

Do not replace obvious yes/no arguments unnecessarily.

---

## Configuration

Centralize configuration appropriately.

In Rails, use the application's established config or credentials systems.

Do not read environment variables throughout random model and service methods.

Validate configuration near startup or integration boundaries where practical.

---

## Environment Variables

Remember:

```ruby
ENV["DEBUG"]
```

is a string or nil.

Do not write:

```ruby
debug = !!ENV["DEBUG"]
```

because:

```text
DEBUG=false
```

is still truthy.

Parse intentionally.

---

## Secrets

Use Rails credentials, environment variables, or the project's secrets system.

Do not hard-code secrets.

Do not log them.

Do not expose them in exception messages.

---

## Credentials

Do not casually move secrets from a secure config mechanism into ordinary YAML or source code.

Follow deployment conventions.

---

## Dependency Injection

Ruby usually does not need a dependency injection container.

Constructor injection is often enough:

```ruby
class WeaponLoader
  def initialize(repository:)
    @repository = repository
  end
end
```

Do not create:

```text
DependencyContainer
ServiceRegistry
Resolver
ProviderFactory
```

for straightforward objects.

---

## Testability

Do not introduce abstractions solely "for testability" without considering simpler alternatives.

Ruby's dynamic nature already makes substitution easy.

You usually do not need an interface class just to inject a fake.

---

## Factories

Use factory objects when construction is genuinely complex or selects among implementations.

Do not create:

```ruby
WeaponFactory.build(...)
```

when:

```ruby
Weapon.new(...)
```

is sufficient.

---

## Builder Pattern

Ruby's keyword arguments and blocks already make construction expressive.

Do not create elaborate builders for simple objects.

Bad:

```ruby
WeaponBuilder.new
  .with_name(name)
  .with_damage(damage)
  .build
```

when:

```ruby
Weapon.new(
  name: name,
  damage: damage
)
```

is clearer.

---

## Decorators

Use decorators/presenters when they cleanly add presentation behavior without polluting domain models.

Do not wrap every model in a decorator automatically.

Use them when there is real presentation-specific behavior.

---

## Delegation

Ruby's:

```ruby
Forwardable
delegate
```

can reduce repetitive pass-through methods.

Use delegation when the API genuinely wants to expose another object's behavior.

Do not build long delegation chains that obscure ownership.

---

## `delegate`

In Rails:

```ruby
delegate :name, to: :weapon
```

can be appropriate.

Do not delegate dozens of methods until one object becomes a disguised proxy for another.

---

## Null Object Pattern

A null object can simplify code when absence has repeated behavior.

Do not introduce one simply to avoid an occasional `nil` check.

Use the pattern when it materially clarifies the API.

---

## Service Locator

Avoid global registries that let objects fetch dependencies implicitly.

This:

```ruby
Services.fetch(:repository)
```

throughout the codebase creates hidden coupling.

Prefer explicit dependencies.

---

## Singletons

Ruby provides the `Singleton` module, but use it sparingly.

Most application services do not need singleton semantics.

Global singleton state hurts:

- Tests.
- Isolation.
- Concurrency.
- Lifecycle clarity.

---

## DSLs

Ruby is excellent for DSLs.

Use one only when the domain benefits from declarative syntax.

Do not create a DSL for a handful of configuration values.

A plain hash or object may be clearer.

---

## DSL Implementation

Keep DSL magic discoverable.

Avoid combining:

```text
method_missing
instance_eval
dynamic constants
implicit global state
```

unless building framework-level infrastructure.

Application code should remain debuggable.

---

## Refining APIs

Prefer clear explicit APIs over syntactic magic.

Professional Ruby is expressive because it removes ceremony, not because it maximizes cleverness.

---

## Avoid Premature Abstraction

Do not design for hypothetical future requirements.

If one implementation exists, direct code may be best.

Do not add:

- Abstract base classes.
- Service registries.
- Factories for one class.
- Strategies for one algorithm.
- Dynamic dispatch tables for one case.
- Generic result wrappers.
- Plugin systems without plugins.
- Hooks without consumers.

Build what the current requirement needs.

---

## Avoid Premature DRY

Do not extract two similar blocks merely because they look alike.

Duplication can be cheaper than the wrong abstraction.

Extract common behavior when:

- The concepts are genuinely the same.
- They change for the same reasons.
- The abstraction has a clear domain name.

Do not create a generic helper with five flags to eliminate ten repeated lines.

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

These names are not inherently wrong.

Use them only when they describe actual responsibilities.

Do not generate architecture by vocabulary.

---

## Avoid Pass-Through Classes

Bad:

```ruby
class WeaponService
  def initialize(repository:)
    @repository = repository
  end

  def find(id)
    @repository.find(id)
  end
end
```

if the class adds no behavior.

Every layer should earn its existence.

---

## Avoid Excessive Service Objects

Not every operation requires:

```ruby
SomethingService.new(...).call
```

A model method, module function, or plain object may be more natural.

Service classes are useful when they represent real application workflows.

---

## Avoid Excessive Concerns

Do not use `ActiveSupport::Concern` as a file-size management tool.

If behavior belongs to one model, leaving it there may be clearer than hiding it in a concern.

---

## Avoid Excessive Metaprogramming

Before adding:

```text
method_missing
define_method
class_eval
instance_eval
const_get
send
```

ask whether explicit Ruby code would be easier to understand.

Usually it will be.

---

## Avoid Dynamic Constant Lookup

Avoid:

```ruby
Object.const_get(type_name)
```

with uncontrolled input.

Prefer explicit mappings:

```ruby
TYPES = {
  "rifle" => Rifle,
  "shotgun" => Shotgun
}.freeze
```

when dynamic selection is genuinely needed.

Explicit mappings are safer and easier to audit.

---

## Avoid String-Based Dispatch

Bad:

```ruby
send("#{action}_weapon")
```

when actions are fixed.

Prefer:

```ruby
case action
when :create
  create_weapon
when :delete
  delete_weapon
end
```

or an explicit map of callables.

Dynamic dispatch should be genuinely dynamic.

---

## Avoid Clever Symbol-to-Proc Chains

This is clear:

```ruby
weapons.map(&:name)
```

This may not be:

```ruby
records
  .map(&:metadata)
  .compact
  .map(&:stats)
  .map(&:damage)
```

if intermediate domain meaning matters.

Use explicit blocks or local variables where necessary.

---

## Avoid Overusing `&.`

Repeated safe navigation can hide a poor data contract.

Before writing:

```ruby
weapon&.metadata&.stats&.damage
```

ask whether a missing metadata/stats object is actually valid.

If not, fail earlier.

---

## Avoid `rescue nil`

Do not write:

```ruby
value = dangerous_operation rescue nil
```

in production code.

It hides all `StandardError` failures and makes debugging extremely difficult.

Handle expected failures explicitly.

---

## Avoid Broad Inline Rescue

Likewise avoid:

```ruby
Integer(value) rescue 0
```

when invalid data needs deliberate handling.

Use:

```ruby
Integer(value, exception: false)
```

if supported and appropriate, or an explicit rescue around the narrow operation.

---

## Avoid Exceptions for Normal Parsing

Ruby APIs often provide non-raising variants.

Use them where appropriate instead of rescuing exceptions in tight loops.

Do not sacrifice clarity merely to avoid exceptions.

---

## Avoid Reopening Core Classes

Do not add project-specific methods to:

```text
String
Array
Hash
Integer
Object
```

globally without a strong architectural reason.

This creates surprising behavior everywhere.

---

## Avoid Generic Helper Files

Do not accumulate:

```text
helpers.rb
utils.rb
common.rb
misc.rb
```

containing unrelated functions.

Create domain-focused files or leave private helpers where they are used.

---

## Avoid Global Method Definitions

Do not define arbitrary methods on `Object` via top-level Ruby code in application libraries.

Use modules/classes to keep namespaces clear.

Small executable scripts are an exception.

---

## Avoid Magic Callbacks

Do not make ordinary method invocation trigger large hidden cascades of unrelated behavior.

This applies especially to Rails model callbacks.

Explicit orchestration is often easier to debug.

---

## Avoid `after_commit` Everything

`after_commit` is useful for behavior that must occur only after a transaction succeeds.

Do not use it as a general event bus.

If several workflows depend on a save, consider explicit application logic or events with clear ownership.

---

## Avoid Fake Robustness

Do not check every internal value defensively.

Bad:

```ruby
def process_weapon(weapon)
  return nil unless weapon
  return nil unless weapon.respond_to?(:name)
  return nil unless weapon.name
  return nil unless weapon.name.is_a?(String)

  # ...
end
```

when callers guarantee a Weapon object.

Validate at trust boundaries.

Trust established internal contracts.

---

## `respond_to?`

Use `respond_to?` when working with intentionally polymorphic/dynamic collaborators.

Do not use it to compensate for unknown internal object types everywhere.

Clear object contracts are better.

---

## Type Checking

If the project uses Sorbet or Steep, follow its conventions.

Do not add a second typing system.

Do not introduce type signatures into an untyped codebase during unrelated work unless requested.

---

## Sorbet

When Sorbet is used:

- Keep signatures accurate.
- Do not use `T.untyped` as the default escape hatch.
- Avoid excessive `T.must`.
- Avoid casts merely to silence errors.
- Respect strictness levels.

Do not make runtime code awkward solely to satisfy a poorly chosen signature.

Fix the type model where practical.

---

## `T.must`

Frequent:

```ruby
T.must(value)
```

may indicate nullable types or control flow are modeled poorly.

Use it when the invariant is genuinely established but Sorbet cannot infer it.

Do not sprinkle it everywhere.

---

## `T.cast`

Do not use `T.cast` merely to make type errors disappear.

A cast does not validate runtime data.

Fix the actual boundary or model where possible.

---

## Steep and RBS

If the project uses RBS/Steep, keep signatures aligned with actual behavior.

Do not create extremely generic signatures merely to eliminate checker warnings.

Use domain-specific types where helpful.

---

## Type Signatures vs Ruby Style

Static typing should support Ruby, not turn Ruby into Java.

Avoid giant layers of interfaces and abstract types solely because the checker can express them.

Keep runtime architecture simple.

---

## Rake

Use Rake tasks for real project automation.

Do not put substantial application logic inside tasks.

Call normal application objects from the task.

This keeps behavior testable and reusable.

---

## CLI Applications

Use the project's CLI framework if one exists.

Possible options include:

```text
OptionParser
Thor
Dry::CLI
GLI
```

Do not add a framework for a tiny script with two options.

---

## Exit Codes

CLI boundaries can use:

```ruby
exit 1
```

or return statuses according to the architecture.

Do not call `exit` deep inside reusable library code.

Raise or return an error and let the CLI decide.

---

## STDOUT vs STDERR

Use:

- STDOUT for normal command output.
- STDERR for errors/diagnostics.

Do not send all logging and data output through the same stream when command-line consumers depend on output.

---

## Libraries vs Applications

Reusable library code should generally avoid:

- Calling `exit`.
- Configuring global logging.
- Reading global environment state everywhere.
- Performing work at require time.

Applications can own these process-level decisions.

---

## Require-Time Side Effects

Avoid:

```ruby
connect_to_database
start_worker
load_all_assets
```

merely because a file was required.

Keep loading definitions separate from starting behavior.

This improves:

- Tests.
- Boot predictability.
- Reuse.

---

## Initializers

In Rails, initializers are appropriate for application startup configuration.

Do not perform heavy external work in initializers without a strong reason.

Startup should remain predictable.

---

## Zeitwerk and Initializers

Do not reference reloadable application constants incorrectly during initialization.

Follow Rails autoload/reloading conventions.

Avoid clever load-order dependencies.

---

## Observability

Use existing logging, metrics, and tracing infrastructure.

Do not add ad-hoc instrumentation libraries for one feature.

Instrument meaningful boundaries, not every helper method.

---

## Metrics

Metrics should represent useful system behavior.

Avoid metrics like:

```text
weapon_method_called_total
helper_execution_total
```

for every method.

Measure outcomes, latency, failures, queue depth, or other operationally useful signals.

---

## Instrumentation Blocks

Framework instrumentation APIs can be appropriate around meaningful operations.

Do not wrap every trivial method simply because tracing exists.

---

## Security-Sensitive Comparisons

For secrets/tokens, use secure constant-time comparison helpers where the framework provides them.

Do not use plain string equality for cryptographic verification when timing resistance matters.

---

## Passwords

Use established password hashing libraries/framework facilities such as bcrypt integrations.

Never implement password hashing yourself.

---

## Tokens

Use secure random generation and established token mechanisms.

Do not create predictable token schemes.

---

## HTML Safety

In Rails views, understand:

```ruby
html_safe
raw
sanitize
```

Do not mark untrusted strings as safe merely to fix escaping.

Escaping is a security boundary.

---

## `html_safe`

Avoid:

```ruby
user_input.html_safe
```

This can create XSS vulnerabilities.

Only mark strings safe when their contents are fully controlled or correctly sanitized.

---

## Mass Assignment

Never broadly permit untrusted params merely to avoid validation work.

Keep input boundaries explicit.

---

## Path Handling

When using user-controlled file paths, consider path traversal where confinement matters.

Do not add arbitrary path restrictions to developer tools that intentionally permit full filesystem access.

Apply the constraint based on actual threat model.

---

## Performance in Rails

Before optimizing Ruby code, inspect:

- Database query count.
- N+1 behavior.
- Cache behavior.
- Serialization.
- View rendering.
- External HTTP.
- Job throughput.

The hottest Ruby loop may not be the real bottleneck.

---

## Eager Loading

Use includes/preload deliberately.

Do not preload massive associations "just in case."

Excessive eager loading can consume more memory than the N+1 it was meant to solve.

---

## Batch Processing

Use:

```ruby
find_each
find_in_batches
```

for large ActiveRecord datasets where loading everything at once would be excessive.

Do not use batching for tiny collections.

---

## Transactions and External Calls

Avoid:

```ruby
ApplicationRecord.transaction do
  update_records
  call_remote_api
end
```

unless holding the DB transaction during the remote call is genuinely required.

Long transactions increase lock contention.

---

## Background Work and Transactions

Ensure jobs depending on committed database state are enqueued at the appropriate point.

Follow Rails/framework conventions.

Do not invent sleep/retry hacks for transaction timing problems.

---

## Avoid `sleep` as Synchronization

Do not use:

```ruby
sleep 1
```

to "wait for" async state in production or tests.

Use proper synchronization, polling with bounds, or framework helpers.

---

## Avoid Retry Loops Without Limits

Never write unbounded:

```ruby
begin
  perform
rescue
  retry
end
```

Use bounded retry behavior with understood failure conditions.

---

## Data Transformation

Keep transformations explicit.

If several stages have domain meaning, name them:

```ruby
records = load_records
valid_records = validate_records(records)
weapons = records_to_weapons(valid_records)
```

Do not compress everything into one chain merely to reduce lines.

---

## Serialization Boundaries

Normalize external data once.

For example:

```ruby
payload = normalize_payload(JSON.parse(response.body))
```

Then internal code should not repeatedly check both string and symbol keys or alternate shapes.

---

## Avoid Mixed Data Shapes

Do not allow internal methods to accept:

```text
Hash
Weapon
JSON string
nil
Array
```

all interchangeably unless the API is intentionally polymorphic.

Normalize at the boundary.

---

## Avoid Boolean Explosion

If a method receives many flags:

```ruby
process(
  weapon,
  validate: true,
  persist: false,
  publish: true,
  notify: false
)
```

consider whether it represents several operations.

Do not automatically create an options object; Ruby keyword args already are one.

But reconsider the responsibility if the flags create many behavioral combinations.

---

## Avoid Configurable Everything

Generated code often turns fixed behavior into options.

Do not make callers configure things the domain always requires.

Configuration multiplies states and tests.

---

## Avoid Generic Result Hashes

Bad:

```ruby
{
  success: true,
  data: weapon,
  error: nil,
  message: nil
}
```

for every operation.

Prefer natural Ruby contracts:

- Return the value.
- Return `nil` for expected absence.
- Raise for failure.
- Use a specific result type when several meaningful states exist.

Do not invent a universal response envelope internally.

---

## Avoid Custom Result Classes Everywhere

A result object is useful when the operation genuinely has several non-exception outcomes.

Do not create:

```ruby
OperationResult.new(
  success: true,
  value: ...
)
```

for trivial methods.

---

## Avoid Overusing OpenStruct

Avoid:

```ruby
OpenStruct
```

as a default dynamic data model.

It is slower and less explicit than normal hashes, Struct, Data, or classes.

Use it only where dynamic attributes are genuinely useful.

---

## Avoid `HashWithIndifferentAccess` Everywhere

In Rails, indifferent access can be useful at request/config boundaries.

Do not spread it throughout domain logic.

Choosing a consistent key type is usually clearer.

---

## Avoid `deep_symbolize_keys` Blindly

Symbolizing arbitrary untrusted keys can be wasteful and may create unnecessary symbols depending on runtime behavior and input size.

Normalize only the keys your application actually understands when practical.

---

## Avoid Excessive Safe Defaults

Generated code often returns:

```ruby
[]
{}
""
0
nil
```

on every error.

This can hide real failures.

Use defaults only when they represent valid domain behavior.

---

## Avoid `rescue StandardError => e; nil`

This pattern is especially damaging.

Do not turn unknown programming failures into missing values.

Handle the expected exception or let it propagate.

---

## Avoid Generic Exception Messages

Bad:

```ruby
raise "Something went wrong"
```

Better:

```ruby
raise PackageLoadError,
      "Unable to load package #{path}: file not found"
```

Provide useful context.

---

## Avoid Overusing Symbols as Mini Enums

Symbols are convenient:

```ruby
:pending
:ready
:failed
```

Use them when the set is simple and local.

For complex state behavior, a richer object or framework enum may provide more structure.

Do not create a class for every three-symbol state set.

---

## Rails Enums

Use ActiveRecord enums when the field naturally has a closed database-backed state set.

Understand generated methods and scopes.

Do not use enums simply to avoid defining constants.

---

## Avoid Hidden Generated Methods

Framework macros can generate many methods.

Know what:

```ruby
enum
delegate
has_many
belongs_to
attribute
```

create.

Do not define conflicting method names accidentally.

---

## Avoid Clever ActiveRecord Scopes

Scopes should remain predictable queries.

Avoid dynamic scope DSLs so abstract that reading the model no longer reveals SQL behavior.

---

## Avoid Callback-Based State Machines

State transitions should be explicit when business rules are important.

Do not hide them across several callbacks.

A state-machine gem may be appropriate if the project already uses one and the domain is genuinely stateful.

---

## Avoid Overusing Gems

Ruby's ecosystem makes it easy to add dependencies.

Before adding one, ask:

- Does Ruby already support this?
- Does Rails already support this?
- Does the project already have a gem for it?
- Is the functionality complex enough to justify another dependency?
- Is the gem actively maintained?
- Is it compatible with the project's Ruby/Rails versions?

---

## Avoid Framework Leakage

Library/domain code should not depend on Rails helpers accidentally unless that dependency is intentional.

This can happen through:

```ruby
blank?
present?
days
hours
with_indifferent_access
```

Do not assume every Ruby environment includes ActiveSupport.

---

## Avoid Excessive Callbacks in Plain Ruby

Observer hooks, before/after hooks, and callback registries can be useful.

Do not turn ordinary object calls into a framework of lifecycle events unless extension is genuinely required.

---

## Avoid Magic Configuration DSLs

A simple configuration object may be clearer than:

```ruby
MyLibrary.configure do |config|
  config.magic do
    option :foo
  end
end
```

when there are only two values.

DSLs should make a domain easier to express, not merely more Ruby-looking.

---

## Avoid Clever Operator Overloading

Ruby lets you define operators.

Use them only when semantics are natural.

Do not make:

```ruby
weapon + attachment
```

mean something surprising.

Readable named methods are often better.

---

## Equality

Implement:

```ruby
==
eql?
hash
```

carefully for value objects used in hashes/sets.

Do not override equality for mutable identity-oriented entities without understanding semantics.

---

## `Comparable`

Include `Comparable` when the type has a meaningful ordering and defines `<=>`.

Do not invent arbitrary ordering merely to gain sorting methods.

---

## Enumerable Custom Types

Include `Enumerable` when the object genuinely represents a collection and defines `each`.

Do not make every container-like domain object enumerable automatically.

Expose domain operations if raw iteration should not be part of its API.

---

## `to_s`

Implement `to_s` for meaningful human-readable representation.

Do not overload it with serialization behavior.

Use dedicated serialization methods for machine formats.

---

## `inspect`

Use meaningful inspection for debugging if necessary.

Do not include secrets.

Framework objects often already provide useful `inspect`.

---

## `method(:name)`

Method objects are useful when passing existing behavior as a callable.

Do not use them instead of straightforward block syntax when it reduces clarity.

---

## Procs and Lambdas

Use lambdas/procs for actual callable behavior.

Remember:

- Lambdas enforce arity more strictly.
- `return` behaves differently in lambdas vs non-lambda Procs.

Do not use them casually without understanding control-flow semantics.

---

## Closures

Closures are useful.

Be mindful of retained state and object lifetimes, especially in long-lived callbacks.

Do not capture large objects unnecessarily in global or long-lived procs.

---

## Before Finishing

Review the change and remove or correct:

- Unnecessary classes.
- Unnecessary modules.
- Unnecessary concerns.
- Service-object boilerplate.
- Generic AI-style names.
- Pass-through layers.
- Abstract base classes with one implementation.
- Unnecessary factories.
- Excessive callbacks.
- Excessive metaprogramming.
- `method_missing` without real need.
- `send` where direct calls work.
- Monkey patches.
- Broad rescue clauses.
- Silent `rescue nil`.
- Defensive checks for already-trusted internal objects.
- Long safe-navigation chains hiding broken invariants.
- Debugging `puts`, `p`, or `pp`.
- Unnecessary memoization.
- Speculative configuration.
- Placeholder TODOs.
- Dead code.
- Unused requires.
- Unrelated refactors.

Then run the project's established checks where available.

Typical Ruby checks may include:

```bash
bundle exec rubocop
bundle exec rspec
```

or:

```bash
bundle exec ruby -Itest ...
bundle exec rake test
```

For Rails projects:

```bash
bin/rails test
bin/rails test:system
bin/rails zeitwerk:check
```

may be relevant.

Type-checked projects may additionally use:

```text
srb tc
steep check
```

Do not assume these exact commands exist.

Inspect:

```text
Gemfile
Rakefile
bin/
.rubocop.yml
CI configuration
project documentation
```

and use the repository's established workflow.

The final code should look like it naturally belongs in the repository rather than like a generic AI-generated Ruby solution.

It should feel like Ruby written by an experienced maintainer: expressive without being clever, dynamic without being magical, object-oriented without being ceremonial, and concise without sacrificing clarity.