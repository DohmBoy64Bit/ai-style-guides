# Python Style Guide

Write Python as an experienced professional developer maintaining a real production codebase.

The goal is clean, idiomatic, maintainable Python—not code that looks generated, over-engineered, excessively defensive, or written as a tutorial.

The most important rule:

> Do not optimize for demonstrating programming practices. Optimize for producing the smallest idiomatic production-quality change that an experienced Python maintainer would reasonably write.

## General Principles

Prefer:

- Simple solutions over clever abstractions.
- Existing project conventions over personal preferences.
- Idiomatic modern Python.
- Small, focused changes.
- Clear code over explanatory comments.
- Functions and modules over unnecessary classes.
- Built-in language features over custom abstractions.
- Standard-library solutions when they are sufficient.
- Explicit domain terminology over generic architectural names.
- Type hints that improve clarity without overwhelming the implementation.
- Direct implementation over speculative extensibility.

Do not refactor unrelated code unless required by the task.

Do not introduce classes, protocols, factories, decorators, abstractions, helpers, or dependency injection layers unless they solve an actual problem.

Do not write Python as if it were Java, C#, or TypeScript.

---

## Match the Existing Codebase

Before writing code, inspect nearby files and follow the repository's established conventions for:

- Naming
- Package structure
- Module organization
- Import style
- Type hints
- Docstrings
- Logging
- Exception handling
- Async code
- Configuration
- Testing
- Dependency management
- Formatting
- Linting
- Supported Python version

Check relevant project files such as:

```text
pyproject.toml
setup.cfg
setup.py
requirements.txt
requirements-dev.txt
tox.ini
pytest.ini
ruff.toml
mypy.ini
.pylintrc
```

Consistency with the existing repository is more important than imposing a preferred style.

Do not perform formatting, typing, or architecture migrations as part of unrelated work.

---

## Python Version

Determine the project's supported Python version before using newer syntax.

Do not assume the project supports the newest Python release.

Check:

```toml
requires-python = ">=3.11"
```

or equivalent configuration.

Use syntax compatible with the declared runtime.

For example, do not introduce:

```python
type WeaponId = str
```

into a project that must support Python 3.11.

Likewise, do not use newer standard-library APIs without verifying availability.

---

## Naming

Follow PEP 8 and the repository's existing conventions.

Use:

- `snake_case` for functions, variables, parameters, and modules.
- `PascalCase` for classes.
- `UPPER_SNAKE_CASE` for true constants.
- A leading underscore for internal implementation details where appropriate.

Good:

```python
weapon_definition
asset_path
package_name
load_result

load_package()
resolve_asset()
parse_weapon_data()
```

Avoid vague AI-style names:

```python
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

```python
data = get_data()
result = process_data(data)
```

Better:

```python
weapon = load_weapon_definition()
stats = parse_weapon_stats(weapon)
```

Do not make names excessively descriptive.

Avoid:

```python
successfully_parsed_weapon_configuration_result
```

when:

```python
weapon_config
```

is clear.

---

## Functions First

Prefer functions for stateless behavior.

Good:

```python
def parse_weapon_stats(data: WeaponData) -> WeaponStats:
    return WeaponStats(
        damage=data.damage,
        fire_rate=data.fire_rate,
    )
```

Do not automatically create:

```python
class WeaponStatsParser:
    def parse(self, data: WeaponData) -> WeaponStats:
        ...
```

unless the object genuinely needs:

- State.
- Dependencies.
- Multiple coordinated methods.
- Lifecycle behavior.
- Polymorphism.
- Encapsulation that materially improves the design.

Python is naturally module-oriented.

Use modules and functions when they are enough.

---

## Function Size

Keep functions focused, but do not split them mechanically.

Prefer:

```python
package = await provider.load_package(path)
return mapper.map(package)
```

over:

```python
package = await load_package_from_provider(provider, path)
mapped_package = map_loaded_package_using_mapper(mapper, package)
return mapped_package
```

Do not create helpers merely because a function exceeds an arbitrary line count.

Extract a helper when it:

- Represents a meaningful operation.
- Is reused.
- Removes genuinely complex logic.
- Clarifies a difficult section.
- Improves testability.

Avoid meaningless helper names:

```python
process_data()
handle_stuff()
execute_logic()
perform_operation()
do_work()
```

---

## Avoid Excessive Tiny Functions

Python code can become harder to follow when every two or three lines are hidden behind helpers.

Avoid:

```python
def get_weapon_name(weapon):
    return weapon.name


def get_weapon_damage(weapon):
    return weapon.damage


def get_weapon_category(weapon):
    return weapon.category
```

when direct attribute access is clearer:

```python
weapon.name
weapon.damage
weapon.category
```

Every function should provide useful semantic value.

---

## Classes

Use classes when they model an actual object with meaningful state or behavior.

Good:

```python
class PackageCache:
    def __init__(self) -> None:
        self._packages: dict[str, Package] = {}

    def get(self, path: str) -> Package | None:
        return self._packages.get(path)

    def set(self, path: str, package: Package) -> None:
        self._packages[path] = package
```

Do not create classes merely because a concept has a noun.

Avoid structures like:

```text
WeaponManager
WeaponService
WeaponProvider
WeaponHandler
WeaponProcessor
WeaponCoordinator
```

unless those names describe real responsibilities.

---

## Avoid Java-Style Architecture

Do not recreate Java or C# patterns automatically.

Avoid this:

```python
class IWeaponParser(Protocol):
    def parse(self, data: WeaponData) -> Weapon:
        ...


class DefaultWeaponParser:
    def parse(self, data: WeaponData) -> Weapon:
        ...
```

when this is sufficient:

```python
def parse_weapon(data: WeaponData) -> Weapon:
    ...
```

Do not add:

- Interface-like protocols for one implementation.
- Abstract base classes for one subclass.
- Factories that construct one type.
- Service classes containing only static methods.
- Getter and setter methods for normal attributes.
- Enterprise repository layers around straightforward libraries.

Python code should feel like Python.

---

## Dataclasses

Use `dataclass` for data-oriented objects when it improves clarity.

Example:

```python
from dataclasses import dataclass


@dataclass(slots=True)
class WeaponStats:
    damage: float
    fire_rate: float
    magazine_size: int
```

Do not automatically turn every dictionary into a dataclass.

Use dataclasses when:

- The object has a stable schema.
- Attribute access improves readability.
- Equality or representation behavior is useful.
- The object is passed through multiple parts of the program.

A simple temporary dictionary may be completely appropriate.

---

## `slots=True`

Use `slots=True` when it matches the project's style or provides actual value.

Do not add it to every dataclass mechanically.

Its benefits include:

- Preventing accidental attributes.
- Potential memory savings.
- Making the intended field set explicit.

Its tradeoffs can matter in inheritance, introspection, or dynamic code.

Use it deliberately.

---

## NamedTuple

Use `NamedTuple` when tuple semantics are actually useful.

Do not use it merely because the value has several fields.

For most domain data, a dataclass or normal class is often clearer.

---

## Enums

Use `Enum` when values have meaningful runtime identity or behavior.

Example:

```python
from enum import StrEnum


class WeaponCategory(StrEnum):
    RIFLE = "rifle"
    SMG = "smg"
    SHOTGUN = "shotgun"
```

For lightweight type checking, a literal may be simpler:

```python
from typing import Literal

WeaponCategory = Literal[
    "rifle",
    "smg",
    "shotgun",
]
```

Follow the project's existing convention.

Do not introduce enums for every small closed set automatically.

---

## Type Hints

Use type hints to improve understanding and tooling.

Good:

```python
def find_weapon(weapon_id: str) -> Weapon | None:
    ...
```

Do not let type hints dominate the implementation.

Avoid unnecessarily complicated signatures:

```python
def process_entity(
    entity: TEntity,
    processor: Callable[
        [TEntity],
        Awaitable[Sequence[MappedResult[TOutput]]],
    ],
) -> Mapping[str, Sequence[TOutput]]:
    ...
```

when the function handles one concrete domain concept.

Prefer specific types when the code is domain-specific.

---

## Do Not Over-Generalize Types

Avoid generics for functions that only serve one real use case.

Bad:

```python
TInput = TypeVar("TInput")
TOutput = TypeVar("TOutput")


def parse_entity(data: TInput) -> TOutput:
    ...
```

when the function specifically parses a weapon.

Prefer:

```python
def parse_weapon(data: WeaponData) -> Weapon:
    ...
```

Generic code should actually be reusable.

---

## Modern Type Syntax

When the supported Python version allows it, prefer modern built-in generic syntax:

```python
list[str]
dict[str, Weapon]
tuple[int, int]
set[str]
```

over:

```python
List[str]
Dict[str, Weapon]
Tuple[int, int]
Set[str]
```

Prefer:

```python
Weapon | None
```

over:

```python
Optional[Weapon]
```

when compatible with the project.

Do not modernize unrelated files solely for style.

---

## `Any`

Avoid `Any` when a useful type is known.

Bad:

```python
def parse(data: Any) -> Any:
    ...
```

Use a specific type when possible.

For genuinely unknown external data, use an appropriate runtime shape or validator.

Do not invent massive type hierarchies merely to eliminate one legitimate `Any`.

---

## `object`

Use `object` when a function truly accepts any Python object but must not perform unchecked operations on it.

This can be safer than `Any`.

However, do not use `object` merely to avoid thinking about the real type.

---

## Type Assertions

Avoid `cast()` solely to silence the type checker.

Bad:

```python
weapon = cast(Weapon, data)
```

when nothing proves that `data` is a `Weapon`.

Prefer narrowing or validation.

Use `cast()` when the runtime invariant is genuinely known but cannot reasonably be inferred by the checker.

---

## Protocols

Use `Protocol` for meaningful structural interfaces when multiple implementations or pluggable behavior actually exists.

Example:

```python
class PackageProvider(Protocol):
    async def load_package(self, path: str) -> Package:
        ...
```

Do not create protocols for every collaborator automatically.

If only one concrete implementation exists and no boundary requires substitution, direct use may be clearer.

---

## Abstract Base Classes

Use abstract base classes only when inheritance is genuinely part of the model.

Do not use ABCs merely to emulate interfaces.

Prefer protocols or composition where appropriate.

Avoid:

```python
class AbstractWeaponProcessor(ABC):
    ...
```

with one implementation and no realistic need for inheritance.

---

## Duck Typing

Use Python's structural nature when appropriate.

Do not add runtime inheritance purely to prove an object supports a small operation.

If an object only needs:

```python
read()
```

it may not need to inherit from a large custom hierarchy.

Balance duck typing with type hints where helpful.

---

## Comments

Do not narrate the code.

Bad:

```python
# Check if the weapon exists
if weapon is None:
    # Return None if the weapon does not exist
    return None
```

Good:

```python
if weapon is None:
    return None
```

Write comments when they explain something the code cannot make obvious, such as:

- A third-party library limitation.
- A protocol requirement.
- A compatibility workaround.
- A performance tradeoff.
- A strange domain constraint.
- Why apparently unnecessary behavior exists.

Prefer explaining **why**, not **what**.

Avoid:

```python
# Initialize the list
# Loop through the items
# Check the condition
# Return the result
# Handle the error
```

---

## Docstrings

Use docstrings where they add useful information.

Good candidates:

- Public APIs.
- Public classes.
- Non-obvious behavior.
- Important side effects.
- Units and formats.
- Exceptions callers need to understand.
- Complex algorithms.
- Library APIs.

Avoid pointless docstrings:

```python
def get_weapon_name(weapon: Weapon) -> str:
    """Get the weapon name."""
    return weapon.name
```

The function name and type already explain it.

Do not generate long Google-style or NumPy-style docstrings for every private helper unless the project requires them.

---

## Avoid AI-Looking Docstrings

Avoid:

```python
"""
This function is responsible for processing weapon data and
ensuring that all required fields are properly validated before
returning the processed result.
"""
```

Prefer concise documentation:

```python
"""Parse weapon metadata from an exported asset record."""
```

or no docstring when the function is already obvious.

---

## Control Flow

Prefer guard clauses over deep nesting.

Good:

```python
if weapon is None:
    return None

if not weapon.loaded:
    await weapon.load()

return weapon
```

Avoid:

```python
if weapon is not None:
    if not weapon.loaded:
        await weapon.load()

    return weapon

return None
```

Keep the normal path easy to follow.

---

## Avoid Unnecessary `else`

After `return`, `raise`, `break`, or `continue`, an `else` is often unnecessary.

Avoid:

```python
if weapon is None:
    return None
else:
    return weapon.name
```

Prefer:

```python
if weapon is None:
    return None

return weapon.name
```

Use `else` when it genuinely improves readability.

---

## Truthiness

Use Python truthiness when it correctly represents the intent.

Good:

```python
if not weapons:
    return []
```

Do not mechanically write:

```python
if len(weapons) == 0:
    return []
```

unless an exact length comparison is meaningful.

However, distinguish between:

```python
if value is None:
```

and:

```python
if not value:
```

when falsy values such as `0`, `False`, or `""` are valid.

---

## `None`

Use:

```python
if value is None:
```

and:

```python
if value is not None:
```

Do not use:

```python
if value == None:
```

Do not mix `None` with unrelated sentinel meanings without a clear contract.

---

## Sentinel Values

Use a dedicated sentinel when `None` is a valid value.

Example:

```python
_MISSING = object()
```

Then:

```python
value = config.get("timeout", _MISSING)

if value is _MISSING:
    ...
```

Do not invent custom sentinel classes unless a plain object is insufficient.

---

## Comparisons

Use chained comparisons where natural:

```python
if 0 <= index < len(items):
    ...
```

rather than:

```python
if index >= 0 and index < len(items):
    ...
```

Do not sacrifice clarity for cleverness.

---

## Membership

Prefer:

```python
if category in allowed_categories:
```

over long repeated comparisons.

Avoid:

```python
if category == "rifle" or category == "smg" or category == "shotgun":
```

when a set or tuple communicates the intent better.

---

## Early `continue`

Use `continue` to reduce nesting in loops.

Good:

```python
for asset in assets:
    if asset.type != "weapon":
        continue

    weapon = parse_weapon(asset)

    if weapon is None:
        continue

    weapons.append(weapon)
```

This is often clearer than nesting each condition.

---

## Comprehensions

Use comprehensions when they remain readable.

Good:

```python
weapon_names = [
    weapon.name
    for weapon in weapons
    if weapon.enabled
]
```

Avoid dense comprehensions containing:

- Several nested loops.
- Multiple conditions.
- Assignment expressions.
- Complex function calls.
- Non-trivial side effects.

A normal loop is often clearer.

---

## Avoid Clever Nested Comprehensions

Bad:

```python
result = {
    category: [
        transform(item)
        for group in groups
        for item in group.items
        if item.category == category and is_valid(item)
    ]
    for category in categories
}
```

This may be compact but difficult to maintain.

Prefer explicit code if the transformation is non-trivial.

---

## Generator Expressions

Use generator expressions when lazy iteration is useful.

Example:

```python
total_damage = sum(
    weapon.damage
    for weapon in weapons
)
```

Do not convert lists into generators mechanically when the data is small and reused.

Use the structure that best communicates intent.

---

## `map()` and `filter()`

Comprehensions are often more idiomatic for simple transformations.

Prefer:

```python
names = [weapon.name for weapon in weapons]
```

over:

```python
names = list(map(lambda weapon: weapon.name, weapons))
```

However, normal functions with `map()` can be perfectly reasonable.

Do not treat either approach as universally correct.

---

## Lambda Functions

Use lambdas for small local expressions.

Good:

```python
weapons.sort(key=lambda weapon: weapon.name)
```

Avoid complicated lambdas:

```python
key=lambda weapon: (
    weapon.category.lower()
    if weapon.category
    else calculate_fallback_category(weapon)
)
```

If the logic needs explanation or multiple operations, use a named function.

---

## Assignment Expressions

Use the walrus operator when it genuinely improves flow.

Example:

```python
if match := pattern.search(text):
    return match.group(1)
```

Do not use assignment expressions merely to make code shorter.

Avoid deeply nested expressions with `:=`.

---

## Pattern Matching

Use `match` when it makes multi-branch structural logic clearer.

Good:

```python
match result.status:
    case "success":
        return result.weapon
    case "not_found":
        return None
    case "error":
        raise result.error
```

Do not replace every `if/elif` chain with `match`.

Use it when the structure naturally fits.

---

## Dictionaries

Use dictionaries for dynamic key-value data.

Good:

```python
weapons_by_id = {
    weapon.id: weapon
    for weapon in weapons
}
```

Do not introduce custom mapping classes without a reason.

Use `.get()` when missing keys are normal:

```python
weapon = weapons_by_id.get(weapon_id)
```

Use direct indexing when the key must exist:

```python
weapon = weapons_by_id[weapon_id]
```

Do not use `.get()` everywhere if a missing key represents a bug.

---

## Sets

Use sets when uniqueness or membership checking is the point.

Good:

```python
processed_paths: set[str] = set()
```

Do not use lists for repeated membership checks when the collection is large or uniqueness matters.

Do not use sets when stable ordering is required.

---

## Tuples

Use tuples for small immutable groupings or return values where positional meaning is obvious.

Example:

```python
return width, height
```

If a tuple grows or fields become unclear, use a dataclass or named structure.

Avoid returning tuples such as:

```python
return weapon, stats, metadata, error, source, cached
```

when the positions are difficult to remember.

---

## Unpacking

Use unpacking when it improves readability.

Good:

```python
width, height = dimensions
```

Avoid clever extended unpacking if it obscures the data structure.

---

## String Formatting

Prefer f-strings for straightforward formatting:

```python
message = f"Unable to load package: {path}"
```

Do not use:

```python
"Unable to load package: {}".format(path)
```

unless the existing project or specific formatting requirement justifies it.

Avoid unnecessary f-strings:

```python
f"weapon"
```

Use:

```python
"weapon"
```

---

## Long Strings

Use parentheses for readable multi-line strings where appropriate.

Example:

```python
message = (
    f"Unable to load package {path!r}: "
    "the package was not found in the mounted archives."
)
```

Do not concatenate many fragments unnecessarily when a clearer structure exists.

---

## Paths

Use `pathlib` for normal filesystem path operations when appropriate.

Prefer:

```python
from pathlib import Path

config_path = root / "config" / "settings.json"
```

over:

```python
config_path = os.path.join(root, "config", "settings.json")
```

when the project's Python version and existing style support `pathlib`.

Do not refactor an entire mature `os.path` codebase solely to modernize it.

---

## File I/O

Use context managers.

Good:

```python
with path.open("r", encoding="utf-8") as file:
    data = json.load(file)
```

Do not manually manage file closure unless necessary.

Specify text encodings where relevant.

Do not rely unnecessarily on platform-default encoding.

---

## JSON

Do not assume decoded JSON has a trusted shape.

External JSON may require validation.

Avoid:

```python
weapon = Weapon(**json.loads(text))
```

when input is untrusted and arbitrary fields may be present.

Validate at the appropriate boundary.

Do not repeatedly validate already-trusted internal objects.

---

## Exceptions

Use exceptions for exceptional situations.

Catch only when you can:

- Recover.
- Add meaningful context.
- Translate into a domain-specific error.
- Perform required cleanup.
- Continue safely.

Avoid:

```python
try:
    ...
except Exception:
    pass
```

Never silently swallow errors without a deliberate reason.

---

## Do Not Catch Everything Automatically

Bad:

```python
try:
    return load_weapon(path)
except Exception as exc:
    logger.error("An error occurred: %s", exc)
    return None
```

This may hide programming errors.

Prefer catching specific expected exceptions:

```python
try:
    return load_weapon(path)
except FileNotFoundError:
    return None
```

Let unexpected failures propagate unless the application boundary requires otherwise.

---

## Broad Exception Handling

A broad:

```python
except Exception:
```

can be appropriate at true application boundaries such as:

- CLI entry points.
- Job runners.
- Worker loops.
- HTTP middleware.
- Plugin boundaries.

Even there, preserve useful context.

Do not use broad exception handling throughout normal application logic.

---

## Preserve Exception Context

When translating exceptions, preserve the original cause.

Good:

```python
try:
    package = load_package(path)
except OSError as exc:
    raise PackageLoadError(path) from exc
```

Avoid:

```python
raise PackageLoadError(path)
```

when doing so destroys useful context.

Use:

```python
raise
```

to re-raise the current exception.

Do not write:

```python
raise exc
```

unless you intentionally want different traceback behavior.

---

## Custom Exceptions

Create custom exceptions when callers genuinely benefit from distinguishing a domain-specific failure.

Good:

```python
class PackageLoadError(RuntimeError):
    pass
```

Do not create an exception subclass for every individual failure.

Avoid hierarchies like:

```text
BaseWeaponException
WeaponProcessingException
WeaponProcessingValidationException
WeaponProcessingValidationFieldException
```

unless the application actually needs that granularity.

---

## Error Messages

Write actionable error messages.

Good:

```text
Unable to load package '/Game/Weapons/Rifle_A': file was not found.
```

Bad:

```text
An error occurred.
```

Include relevant information such as:

- Paths.
- IDs.
- Names.
- Operations.

Do not include secrets or sensitive values.

---

## `assert`

Use `assert` for programmer invariants, not user-input validation.

Good:

```python
assert current_package is not None
```

when the preceding logic guarantees that invariant and a failure represents a programming bug.

Do not write:

```python
assert user_age >= 18
```

for runtime validation that must remain enabled.

Assertions can be disabled with optimization flags.

---

## Logging

Use the project's logging framework.

Usually:

```python
import logging

logger = logging.getLogger(__name__)
```

Log meaningful events and failures.

Avoid logging every method entry, exit, branch, and successful operation.

Bad:

```python
logger.info("Starting weapon processing")
logger.info("Processing weapon")
logger.info("Weapon processed successfully")
```

Better:

```python
logger.warning(
    "Unable to resolve weapon asset %s",
    asset_path,
)
```

---

## Lazy Logging Formatting

Prefer logging's argument interpolation:

```python
logger.debug("Loaded package %s", package_name)
```

instead of:

```python
logger.debug(f"Loaded package {package_name}")
```

when using standard logging.

This avoids formatting work when the log level is disabled.

Follow the project's logger if it uses another convention such as structlog.

---

## Avoid Duplicate Logging

Do not log an exception at every layer.

If a lower layer raises meaningful context, let the application boundary decide whether to log it.

Repeated logging produces noisy duplicate stack traces.

---

## `print`

Use `print()` for:

- CLI output.
- User-facing command output.
- Small scripts.
- Debugging during development.

Do not leave debugging prints inside application or library code.

Use logging when diagnostic output belongs to the application's observability system.

---

## Async Code

Use `async` for genuinely asynchronous operations.

Do not make functions async merely because surrounding code is async.

Bad:

```python
async def get_count(items: list[Weapon]) -> int:
    return len(items)
```

Prefer:

```python
def get_count(items: list[Weapon]) -> int:
    return len(items)
```

Do not use async as decoration.

---

## Never Block the Event Loop Accidentally

Inside async code, avoid blocking operations such as:

```python
time.sleep(1)
```

Use:

```python
await asyncio.sleep(1)
```

for asynchronous waiting.

For blocking I/O or CPU-heavy work, use the project's established strategy.

Do not automatically move everything into threads.

---

## Concurrent Async Work

Run independent async operations concurrently when appropriate.

Example:

```python
weapons, attachments = await asyncio.gather(
    load_weapons(),
    load_attachments(),
)
```

Do not parallelize operations that:

- Depend on each other.
- Must occur in order.
- Could overload a remote service.
- Mutate shared state unsafely.

Concurrency is not automatically faster or better.

---

## Task Groups

When supported by the project's Python version, `asyncio.TaskGroup` can provide clearer structured concurrency:

```python
async with asyncio.TaskGroup() as group:
    weapon_task = group.create_task(load_weapons())
    attachment_task = group.create_task(load_attachments())
```

Use it when its failure semantics fit the task.

Do not rewrite existing `asyncio.gather()` code without a reason.

---

## Async Iteration

Use:

```python
async for item in source:
    ...
```

for asynchronous iterators.

Do not materialize large async streams into lists unless necessary.

---

## Context Managers

Use context managers for resources with clear lifetime boundaries:

- Files.
- Locks.
- Database transactions.
- Temporary directories.
- Network sessions.
- Timers where appropriate.

Good:

```python
with database.transaction():
    ...
```

or:

```python
async with session.get(url) as response:
    ...
```

Do not manually duplicate cleanup logic when a context manager already exists.

---

## Custom Context Managers

Do not create a custom context manager for trivial setup and cleanup that happens once.

Use one when it creates a reusable, meaningful resource boundary.

---

## Decorators

Use decorators when they express cross-cutting behavior cleanly.

Good examples may include:

- Framework route registration.
- Caching.
- Authentication.
- Retry behavior.
- Test fixtures.
- Registration mechanisms.

Do not create custom decorators merely to make code look elegant.

Avoid hiding important control flow behind multiple stacked custom decorators.

---

## Dependency Injection

Python often does not need a DI framework.

Simple dependency passing is usually enough:

```python
def load_weapon(
    path: str,
    provider: PackageProvider,
) -> Weapon:
    package = provider.load_package(path)
    return parse_weapon(package)
```

Do not introduce a container solely for testability.

Avoid:

```text
ServiceContainer
DependencyRegistry
ProviderFactory
DependencyResolver
```

unless the application genuinely benefits from them.

---

## Configuration

Load and validate configuration near the application's boundary.

Avoid accessing environment variables throughout the codebase.

Bad:

```python
timeout = int(os.environ["TIMEOUT"])
```

scattered across many modules.

Prefer central configuration:

```python
@dataclass(frozen=True)
class Config:
    timeout: int
    api_url: str
```

Then pass or import configuration according to the project's architecture.

Do not build a giant configuration framework for a small script.

---

## Environment Variables

Treat environment variables as strings until parsed.

Validate required values clearly.

Avoid:

```python
debug = bool(os.getenv("DEBUG"))
```

because:

```text
DEBUG=false
```

still produces a truthy string.

Parse intentionally.

---

## Imports

Follow PEP 8 and the repository's formatter.

Generally group:

1. Standard library.
2. Third-party packages.
3. Local application imports.

Example:

```python
import json
from pathlib import Path

import httpx

from project.models import Weapon
```

Do not manually rearrange imports if the project uses Ruff or isort.

---

## Avoid Wildcard Imports

Do not use:

```python
from module import *
```

except in rare contexts where the project intentionally exposes a controlled namespace.

Explicit imports are easier to understand and analyze.

---

## Avoid Import-Time Side Effects

Modules should generally not perform significant work merely by being imported.

Avoid:

```python
client = connect_to_remote_service()
load_all_assets()
start_background_thread()
```

at module import time unless the application architecture explicitly requires it.

Import side effects make testing and startup behavior harder to reason about.

---

## Circular Imports

Do not solve circular imports by scattering local imports everywhere without understanding the architecture.

A local import can be appropriate, but persistent circular dependencies may signal an incorrect module boundary.

Fix the architecture when practical.

Do not perform a large module refactor unless required.

---

## Package `__init__.py`

Do not re-export everything automatically.

Use `__init__.py` exports intentionally when they define a meaningful public package API.

Large re-export surfaces can create:

- Circular imports.
- Hidden coupling.
- Slower imports.
- Difficult navigation.

---

## Modules

Keep modules cohesive.

Do not impose arbitrary "one class per file" rules.

Python modules can naturally contain several closely related functions and data types.

Avoid creating:

```text
weapon_helper.py
weapon_utils.py
weapon_manager.py
weapon_service.py
weapon_processor.py
```

unless these truly represent distinct responsibilities.

Prefer domain-based organization.

---

## Avoid Generic Utility Modules

Do not dump unrelated code into:

```text
utils.py
helpers.py
common.py
misc.py
```

when a clearer domain-specific module exists.

Better:

```text
weapon_paths.py
asset_format.py
package_validation.py
```

Small local helpers can remain next to the code that uses them.

---

## Private APIs

Use leading underscores to signal module-internal or class-internal APIs:

```python
def _normalize_asset_path(path: str) -> str:
    ...
```

Do not create accessor methods simply to hide normal internal attributes.

Python privacy is conventional, not absolute.

---

## Properties

Use `@property` when attribute-like access makes sense.

Good:

```python
@property
def loaded(self) -> bool:
    return self._package is not None
```

Do not write Java-style getters and setters by default:

```python
def get_name(self) -> str:
    return self._name

def set_name(self, value: str) -> None:
    self._name = value
```

Prefer:

```python
weapon.name
```

unless behavior is required.

---

## Property Side Effects

Properties should generally be inexpensive and unsurprising.

Avoid a property that:

- Performs network I/O.
- Writes files.
- Mutates global state.
- Performs expensive computation unexpectedly.

Use a method when an operation has meaningful work or side effects.

---

## Magic Methods

Implement magic methods when they provide natural Python behavior.

Examples:

```python
__repr__
__len__
__iter__
__contains__
__enter__
__exit__
```

Do not implement magic methods simply because they exist.

Avoid surprising operator overloading.

---

## `__repr__`

Use a useful `__repr__` for objects developers may inspect.

Prefer concise identifying information.

Do not dump massive object state or secrets.

Dataclasses may already provide a sufficient representation.

---

## `__str__`

Use `__str__` for meaningful user-facing representations.

Do not add it automatically.

---

## Equality

Use dataclass-generated equality or explicit equality when value semantics matter.

Do not override `__eq__` for objects whose identity is more meaningful.

---

## Constants

Use constants for values with domain meaning.

Good:

```python
MAX_RETRY_ATTEMPTS = 3
DEFAULT_TIMEOUT_SECONDS = 30
```

Do not create:

```python
ZERO = 0
ONE = 1
EMPTY_STRING = ""
```

A constant should communicate meaning.

---

## Module-Level State

Avoid mutable global state.

Bad:

```python
current_weapon = None
loaded_packages = {}
```

unless the application architecture deliberately owns global state.

Prefer passing dependencies or encapsulating state where appropriate.

Do not turn every global into a singleton class; that often adds more complexity rather than solving the problem.

---

## Singletons

Avoid custom singleton patterns unless there is a real reason.

Python modules already provide a natural shared namespace.

Do not implement:

```python
class SingletonMeta(type):
    ...
```

just to hold application configuration or a logger.

---

## Caching

Do not add caching without a demonstrated need.

Caching introduces:

- Invalidation.
- Stale state.
- Memory usage.
- Concurrency concerns.
- Harder tests.

When caching is appropriate, built-ins may be sufficient:

```python
from functools import cache
```

or:

```python
from functools import lru_cache
```

Do not memoize functions automatically.

---

## Performance

Prefer readable code until performance matters.

Do not prematurely:

- Micro-optimize loops.
- Avoid every allocation.
- Introduce multiprocessing.
- Introduce threads.
- Cache everything.
- Replace clear Python with obscure tricks.
- Move code to NumPy purely because it "might be faster."

Measure actual bottlenecks first.

---

## Iteration

Iterate directly over objects.

Prefer:

```python
for weapon in weapons:
    ...
```

over:

```python
for index in range(len(weapons)):
    weapon = weapons[index]
```

unless the index is genuinely needed.

When the index is needed:

```python
for index, weapon in enumerate(weapons):
    ...
```

---

## Parallel Lists

Avoid maintaining related data in separate synchronized lists.

Bad:

```python
weapon_names = []
weapon_damage = []
weapon_categories = []
```

when each index represents one weapon.

Use a data structure representing a weapon.

---

## Sorting

Use the `key` parameter.

Good:

```python
weapons.sort(key=lambda weapon: weapon.name)
```

Do not manually implement comparison loops.

Use `operator.attrgetter` when it improves clarity, but do not introduce it merely to avoid a simple lambda.

---

## `sorted()` vs `.sort()`

Use:

```python
sorted(items)
```

when you need a new list.

Use:

```python
items.sort()
```

when in-place mutation is intended.

Do not copy collections unnecessarily just to avoid mutation if the local mutation is obvious and harmless.

---

## Copying

Understand shallow versus deep copies.

Do not call:

```python
copy.deepcopy()
```

automatically to avoid mutation concerns.

Deep copying can be expensive and may copy objects incorrectly.

Prefer deliberate reconstruction or shallow copying when appropriate.

---

## Immutability

Use immutable structures where they naturally improve correctness.

Do not force functional programming onto ordinary Python.

This is perfectly readable:

```python
weapons = []

for asset in assets:
    weapon = parse_weapon(asset)

    if weapon is not None:
        weapons.append(weapon)
```

Do not rewrite it into a dense expression merely to avoid mutation.

---

## Frozen Dataclasses

Use:

```python
@dataclass(frozen=True)
```

when immutability is an actual invariant.

Do not make every model frozen automatically.

Some domain objects genuinely change state.

---

## Pydantic and Validation Libraries

Use Pydantic, attrs, Marshmallow, msgspec, or another validation library when the project already uses it or runtime validation provides real value.

Good uses include:

- API requests.
- API responses.
- Configuration.
- External manifests.
- Plugin inputs.
- Untrusted serialized data.

Do not introduce Pydantic to represent simple internal data that a dataclass handles perfectly well.

---

## Validate at Boundaries

Validate external data once near the trust boundary.

Examples:

- HTTP input.
- JSON files.
- Environment variables.
- CLI arguments.
- Database payloads when schemas are not guaranteed.
- Plugin/MCP inputs.

After validation, use trusted internal types.

Do not repeatedly validate the same values throughout internal code.

---

## Avoid Fake Robustness

Do not add checks for impossible states merely to look defensive.

Bad:

```python
def process_weapon(weapon: Weapon) -> WeaponStats:
    if weapon is None:
        raise ValueError("weapon cannot be None")
```

when the type and calling convention guarantee a `Weapon`.

Validate where untrusted input enters the system.

Trust established internal invariants.

---

## Avoid Defensive `hasattr` Everywhere

Do not write:

```python
if hasattr(weapon, "name"):
    ...
```

for typed internal objects whose contract guarantees `.name`.

This weakens the code's assumptions rather than improving robustness.

Use runtime introspection at genuinely dynamic boundaries.

---

## Reflection

Use `getattr`, `setattr`, `hasattr`, and introspection when the problem is genuinely dynamic.

Do not replace straightforward attribute access with reflection.

Bad:

```python
name = getattr(weapon, "name", None)
```

when `weapon` is known to be a `Weapon`.

Prefer:

```python
name = weapon.name
```

---

## Dynamic Imports

Avoid dynamic imports unless plugin loading or optional dependencies genuinely require them.

Normal imports are easier for:

- IDEs.
- Static analysis.
- Packaging.
- Refactoring.
- Readers.

Do not use `importlib` merely to make a system appear extensible.

---

## Metaclasses

Do not use metaclasses unless the problem genuinely requires class creation customization.

Most application code does not need them.

If a simpler decorator, class method, registry, or normal function works, prefer it.

Metaclasses should be rare.

---

## Descriptors

Use descriptors only when their behavior provides real reusable value.

Do not use descriptors for ordinary validation or attribute access when properties or dataclasses are sufficient.

---

## Decorator Registries

Registries can be useful for plugin-style systems.

Example:

```python
handlers: dict[str, Handler] = {}


def register_handler(name: str):
    def decorator(handler: Handler) -> Handler:
        handlers[name] = handler
        return handler

    return decorator
```

Do not create a registry for a fixed set of two functions that never changes dynamically.

---

## Dependency Management

Use the repository's existing package manager and workflow.

Possible tools include:

```text
pip
uv
Poetry
PDM
pip-tools
Hatch
```

Do not switch dependency managers as part of unrelated work.

Do not edit lockfiles manually.

Use the project's established commands.

---

## Adding Dependencies

Before adding a dependency, ask:

- Does the standard library already solve this well?
- Does the project already have a dependency for it?
- Is the dependency maintained?
- Is the functionality complex enough to justify a package?
- Does it materially increase deployment size or startup cost?

Do not install a package to replace five obvious lines of code.

Also do not reimplement security-sensitive or standards-heavy functionality purely to avoid dependencies.

---

## Standard Library

Know and use the standard library where appropriate.

Common useful modules include:

```text
pathlib
dataclasses
collections
itertools
functools
contextlib
enum
json
csv
logging
asyncio
subprocess
tempfile
shutil
statistics
datetime
zoneinfo
```

Do not reinvent functionality these modules already provide.

Do not force standard-library solutions when a project dependency already provides a clearer established approach.

---

## Dates and Time

Use timezone-aware datetimes when representing real-world timestamps.

Avoid:

```python
datetime.now()
```

when the timestamp represents an absolute point in time across systems.

Prefer an explicit timezone:

```python
from datetime import datetime, timezone

now = datetime.now(timezone.utc)
```

Follow project conventions.

Use `zoneinfo` for IANA time zones on supported Python versions.

Do not hand-roll timezone calculations.

---

## Regular Expressions

Use regex for problems regex expresses clearly.

Compile reused or complex patterns where appropriate:

```python
ASSET_PATH_RE = re.compile(r"^/Game/[A-Za-z0-9_/]+$")
```

Do not write massive unreadable regexes when explicit parsing is clearer.

Use verbose mode for complex expressions if the project accepts it.

---

## Subprocesses

Prefer `subprocess.run()` for straightforward command execution.

Example:

```python
subprocess.run(
    ["git", "status", "--short"],
    check=True,
)
```

Avoid:

```python
os.system(...)
```

for modern production code.

Do not pass untrusted strings through a shell unnecessarily.

Prefer argument lists and `shell=False`.

---

## Shell Commands

Avoid:

```python
subprocess.run(
    f"tool --file {filename}",
    shell=True,
)
```

when `filename` can be passed directly:

```python
subprocess.run(
    ["tool", "--file", filename],
    check=True,
)
```

Use shell execution only when shell semantics are actually required.

---

## Temporary Files

Use `tempfile`.

Do not invent random temporary paths manually.

Prefer:

```python
with tempfile.TemporaryDirectory() as temp_dir:
    ...
```

or:

```python
with tempfile.NamedTemporaryFile() as file:
    ...
```

when appropriate.

---

## Resource Cleanup

Use:

- Context managers.
- `try/finally`.
- Framework lifecycle hooks.

Do not depend on garbage collection to promptly close important resources.

---

## CLI Code

For small CLIs, `argparse` may be sufficient.

Use Typer, Click, or another framework when the project already uses it or when features justify it.

Do not add a CLI framework for a script with two obvious arguments.

Keep parsing separate from core logic where practical.

---

## Entry Points

Use:

```python
def main() -> int:
    ...

if __name__ == "__main__":
    raise SystemExit(main())
```

for normal executable modules where appropriate.

Returning an exit code makes testing easier.

Do not force this pattern into framework-managed applications.

---

## Library vs Application Code

Library code should usually:

- Avoid calling `sys.exit`.
- Avoid configuring global logging.
- Avoid reading process-wide configuration implicitly.
- Raise meaningful exceptions.

Application entry points can translate failures into:

- Exit codes.
- User messages.
- Logs.

Keep these boundaries clear.

---

## Testing

Use the project's existing test framework.

Usually this means `pytest`, but do not assume.

Tests should verify behavior, not implementation details.

Good:

```python
def test_parse_weapon_returns_none_without_definition():
    ...
```

Avoid:

```python
def test_weapon_parser():
    ...
```

Describe the condition and expected behavior.

---

## Pytest

When using pytest, prefer simple test functions unless classes provide real grouping value.

Good:

```python
def test_find_weapon_returns_none_for_missing_id():
    ...
```

Do not wrap every test in:

```python
class TestWeaponService:
```

unless that structure helps the suite.

---

## Fixtures

Use fixtures for genuinely shared setup.

Avoid converting every local test object into a fixture.

Bad:

```python
@pytest.fixture
def weapon_name():
    return "Rifle"
```

if used once.

Inline data is often easier to understand.

---

## Fixture Scope

Choose fixture scope deliberately.

Do not make fixtures session-scoped solely for speed without understanding shared-state implications.

Prefer isolation unless expensive setup justifies broader scope.

---

## Mocking

Do not mock everything.

Prefer real lightweight collaborators when practical.

Mock:

- Network calls.
- External services.
- Databases when integration isn't being tested.
- Time when necessary.
- Nondeterministic operations.
- Expensive dependencies.

Do not mock:

- Simple dataclasses.
- Pure functions.
- Basic containers.
- Internal logic merely to isolate every function.

Over-mocking creates brittle tests.

---

## `unittest.mock`

Patch where the dependency is **looked up**, not where it originally came from.

Do not scatter patches across internal implementation details.

Prefer dependency injection or a simple seam when that makes tests clearer.

---

## Test Behavior, Not Calls

Avoid tests whose only purpose is to assert internal implementation sequence.

Bad:

```python
mock_parser.parse.assert_called_once()
mock_mapper.map.assert_called_once()
```

when the actual output behavior can be tested directly.

Interaction assertions are appropriate when the interaction itself is the behavior.

---

## Test Helpers

Do not create a testing framework inside the project for a handful of tests.

Avoid unnecessary:

```text
FixtureFactory
TestDataBuilder
MockManager
TestingService
```

Use simple helpers when repetition actually becomes a problem.

---

## Temporary Test Files

Use pytest's `tmp_path` or standard temporary-file utilities.

Do not hard-code test files into random system locations.

---

## Deterministic Tests

Tests should not depend unnecessarily on:

- Real clock time.
- Network availability.
- Test ordering.
- Random values.
- Existing filesystem state.
- Environment-specific paths.

Control nondeterminism where it affects correctness.

---

## Property-Based Testing

Use Hypothesis when the domain benefits from testing broad input spaces.

Do not introduce property-based testing for trivial fixed examples.

It is especially useful for:

- Parsers.
- Serializers.
- Validators.
- Numeric algorithms.
- State machines.

Use it where it provides real coverage.

---

## Formatting

Use the repository's formatter.

Common tools include:

```text
Ruff
Black
YAPF
autopep8
```

Do not manually fight automated formatting.

Avoid formatting unrelated files.

Do not reformat an entire repository as part of a small feature change.

---

## Ruff

If the project uses Ruff, respect its configuration.

Do not add blanket ignores merely to make generated code pass.

Avoid:

```python
# noqa
```

without understanding the violation.

Use targeted ignores only when justified.

Example:

```python
# noqa: E501
```

if a generated string or URL genuinely should remain intact and the project permits it.

---

## Black

If the project uses Black, let Black choose formatting.

Do not manually align arguments or dictionary values in ways Black will undo.

Do not use:

```python
# fmt: off
```

unless there is a strong reason.

---

## Type Checking

Use the project's configured checker.

Common tools include:

```text
mypy
pyright
basedpyright
Pyre
```

Do not weaken global type settings to solve a local problem.

Avoid:

```toml
ignore_errors = true
```

or blanket `# type: ignore`.

Fix the actual type issue when practical.

---

## `type: ignore`

Use targeted ignores when interfacing with broken or incomplete third-party types.

Prefer:

```python
value = library_call()  # type: ignore[assignment]
```

over:

```python
# type: ignore
```

when the checker supports error codes.

Do not use ignores to bypass errors you can fix cleanly.

---

## Linting

Use the repository's linting tools and scripts.

Common commands may include:

```bash
ruff check .
ruff format --check .
mypy .
pytest
```

But do not assume these exact commands exist.

Inspect the repository configuration first.

---

## Avoid AI-Looking Architecture

Do not automatically introduce names such as:

```text
Manager
Service
Provider
Handler
Factory
Processor
Coordinator
Registry
Strategy
Adapter
Repository
Facade
Orchestrator
Context
Utility
Helper
```

These concepts can be legitimate.

Use them only when they describe an actual architectural role.

Do not produce symmetrical layers merely because they look organized.

---

## Avoid Premature Abstraction

Do not design for hypothetical requirements.

If one implementation exists, implement it directly unless another implementation is realistically required.

Do not add:

- Abstract base classes for one subclass.
- Protocols for every dependency.
- Factories for one concrete class.
- Registries for fixed behavior.
- Builders for straightforward construction.
- Custom result wrappers without a need.
- Dependency injection containers for small projects.
- Plugin architectures for one plugin.
- Generic repositories over straightforward APIs.
- Utility classes filled with static methods.

Build what the current task requires.

---

## Avoid Pass-Through Layers

Do not create layers that merely forward calls.

Bad:

```python
class WeaponService:
    def __init__(self, repository: WeaponRepository):
        self._repository = repository

    def get_weapon(self, weapon_id: str) -> Weapon | None:
        return self._repository.get_weapon(weapon_id)
```

if the service adds no behavior and the architecture does not require the boundary.

Every layer should earn its existence.

---

## Avoid Static Utility Classes

Do not write:

```python
class WeaponUtils:
    @staticmethod
    def normalize_name(name: str) -> str:
        ...
```

Prefer:

```python
def normalize_weapon_name(name: str) -> str:
    ...
```

at module scope.

Python modules already provide namespaces.

---

## Avoid Getter/Setter Boilerplate

Do not write:

```python
class Weapon:
    def get_name(self):
        return self._name

    def set_name(self, name):
        self._name = name
```

Prefer:

```python
weapon.name
```

Use properties only when behavior or validation is needed.

---

## Avoid Custom Collection Wrappers

Do not create a custom class around a list or dictionary unless it adds meaningful domain behavior.

Bad:

```python
class WeaponCollection:
    def __init__(self):
        self._weapons = []

    def add(self, weapon):
        self._weapons.append(weapon)

    def all(self):
        return self._weapons
```

when a normal list is enough.

---

## Avoid Excessive Result Types

Do not return a custom result object for every simple operation.

Avoid:

```python
@dataclass
class FindWeaponResult:
    success: bool
    weapon: Weapon | None
    error: str | None
```

when:

```python
def find_weapon(...) -> Weapon | None:
```

is sufficient.

Use structured result types when multiple meaningful non-exception states truly exist.

---

## Avoid Tuple Booleans

Avoid APIs like:

```python
success, weapon = load_weapon(path)
```

when the object itself or an exception provides enough information.

Return contracts should be clear.

---

## Avoid Boolean Parameter Piles

Bad:

```python
process_weapon(
    weapon,
    True,
    False,
    True,
)
```

Prefer keyword arguments:

```python
process_weapon(
    weapon,
    validate=True,
    normalize=False,
    include_metadata=True,
)
```

For many related options, a configuration object may become appropriate.

Do not create one for a simple two-argument function.

---

## Keyword-Only Arguments

Use keyword-only parameters when several arguments would otherwise be ambiguous.

Example:

```python
def export_weapon(
    weapon: Weapon,
    *,
    include_model: bool = True,
    include_thumbnail: bool = True,
) -> Path:
    ...
```

Do not make every parameter keyword-only without a reason.

---

## Default Arguments

Never use mutable objects as default values.

Bad:

```python
def add_weapon(
    weapon: Weapon,
    items: list[Weapon] = [],
):
    ...
```

Prefer:

```python
def add_weapon(
    weapon: Weapon,
    items: list[Weapon] | None = None,
):
    if items is None:
        items = []
```

This is a real Python correctness issue, not merely a style preference.

---

## Sentinel Defaults

If `None` is a valid caller value, use a sentinel rather than overloading it as "not supplied."

Do not complicate functions with sentinels unless the distinction matters.

---

## Avoid Star Arguments Without Purpose

Do not write:

```python
def process(*args, **kwargs):
    ...
```

when the function has a known signature.

Explicit parameters improve:

- Readability.
- IDE support.
- Type checking.
- Refactoring.

Use variadic arguments when the API genuinely requires them.

---

## Avoid Dynamic Attribute Bags

Do not use arbitrary objects as untyped property containers.

Bad:

```python
class Data:
    pass

data = Data()
data.weapon_name = "Rifle"
data.damage = 35
```

Use a dictionary, dataclass, or explicit model depending on the domain.

---

## Avoid Monkey Patching

Do not monkey patch third-party code unless there is a strong compatibility or testing reason.

Prefer supported extension points.

If monkey patching is unavoidable, isolate it and explain why.

---

## Security

Do not weaken security for convenience.

Never:

- Hard-code secrets.
- Use `eval()` on untrusted content.
- Use `exec()` on untrusted content.
- Deserialize untrusted pickle data.
- Disable TLS validation casually.
- Build SQL through string concatenation.
- Build shell commands from untrusted strings.
- Trust external file paths without considering traversal where relevant.

Use established libraries and project mechanisms.

---

## Pickle

Treat pickle as code execution, not a safe data format.

Never load untrusted pickle data.

Use JSON or another safer format when data crosses trust boundaries.

---

## YAML

Be aware of parser safety.

Use safe loading APIs for untrusted YAML.

Do not use unsafe deserialization merely for convenience.

---

## SQL

Use parameterized queries.

Bad:

```python
cursor.execute(
    f"SELECT * FROM weapons WHERE id = '{weapon_id}'"
)
```

Good:

```python
cursor.execute(
    "SELECT * FROM weapons WHERE id = ?",
    (weapon_id,),
)
```

Use the parameter style expected by the database library.

---

## URLs and HTTP

Use an established HTTP client already present in the project.

Examples may include:

```text
httpx
requests
aiohttp
urllib.request
```

Do not add another HTTP library without a reason.

Set reasonable timeouts for external requests.

Do not assume remote responses are valid merely because the request succeeded.

---

## HTTP Status Handling

Check response status as appropriate.

With `httpx`, for example:

```python
response = client.get(url)
response.raise_for_status()
```

Do not blindly decode failure responses as successful data.

---

## Retries

Add retries only for failures where retrying makes sense.

Appropriate examples may include:

- Temporary network failures.
- Rate limits.
- Service unavailability.

Do not retry:

- Invalid input.
- Authentication failures that require new credentials.
- Deterministic parsing errors.
- Programming errors.

Use bounded retries and sensible backoff.

Do not build a custom retry framework if the project already has one.

---

## Database Code

Follow the database or ORM's established conventions.

Do not add repository layers on top of straightforward ORM calls merely because enterprise architecture often does.

If this is clear:

```python
weapon = session.get(Weapon, weapon_id)
```

do not hide it behind three pass-through classes without a real benefit.

---

## Transactions

Use explicit transaction boundaries for related writes.

Do not commit after every individual operation when those operations must succeed together.

Follow the ORM or database driver's established patterns.

---

## Framework Code

Follow the framework instead of fighting it.

For frameworks such as:

- FastAPI.
- Django.
- Flask.
- SQLAlchemy.
- Pydantic.
- Typer.
- Click.
- Celery.
- asyncio-based systems.

Use their normal patterns.

Do not wrap simple framework APIs in custom layers without a reason.

---

## Django

When working in Django:

- Use the ORM unless raw SQL is justified.
- Use forms or serializers where the project already does.
- Keep business logic out of giant views when it grows complex.
- Avoid signal-based behavior when an explicit call is clearer.
- Use migrations for model changes.
- Respect query counts and avoid accidental N+1 queries.

Do not introduce separate repository layers automatically.

---

## FastAPI

When working in FastAPI:

- Use dependency injection where it naturally fits the framework.
- Keep route functions reasonably thin.
- Validate external models at the boundary.
- Do not duplicate Pydantic validation manually.
- Avoid turning every endpoint into six architectural layers.

A route calling one domain function can be perfectly professional.

---

## Pydantic

Use Pydantic models for runtime validation where appropriate.

Do not duplicate every Pydantic model with:

```text
WeaponInputDTO
WeaponInternalModel
WeaponDomainModel
WeaponOutputDTO
```

unless those representations genuinely differ.

Mapping layers should correspond to real boundaries.

---

## Data Processing

For small datasets, normal Python loops are often ideal.

Do not introduce pandas solely to transform a small list of records.

Use pandas when:

- Tabular operations are substantial.
- Aggregation is complex.
- Vectorized data processing is useful.
- The project already uses it.

Likewise, do not force plain loops onto workloads naturally suited to NumPy or pandas.

---

## NumPy

Use NumPy for numeric workloads where vectorization matters.

Do not convert ordinary lists into NumPy arrays merely to look scientific.

Avoid unnecessary conversions back and forth.

---

## Multiprocessing

Use multiprocessing for CPU-bound workloads only when the overhead and serialization model make sense.

Do not automatically parallelize loops.

Consider:

- Startup overhead.
- Pickling cost.
- Memory duplication.
- Platform behavior.
- Process count.
- Error handling.

Measure before adding complexity.

---

## Threads

Threads can be useful for blocking I/O.

Do not use them to solve CPU-bound Python work unless the workload releases the GIL or the runtime behavior supports it.

Do not introduce thread pools casually.

---

## File Organization

Organize by domain or cohesive responsibility.

Do not create deep package hierarchies without a real need.

Avoid:

```text
src/
    services/
        managers/
            processors/
                handlers/
                    weapon_handler.py
```

for a small project.

Prefer the simplest structure that remains understandable.

---

## `__all__`

Use `__all__` when the public module API benefits from being explicit.

Do not add it to every module mechanically.

---

## Circular Architectural Layers

Avoid flows like:

```text
controller
  -> service
    -> manager
      -> provider
        -> repository
          -> adapter
```

when several layers only forward calls.

A production Python system can be well-structured without enterprise layering.

---

## Do Not Rewrite Working Code Unnecessarily

When modifying an existing feature:

1. Identify the smallest responsible area.
2. Understand existing behavior.
3. Preserve unrelated behavior.
4. Match the current architecture.
5. Change only what the task requires.
6. Update tests for changed behavior.
7. Avoid unrelated cleanup.

Do not rewrite a working module because you prefer another style.

---

## Refactoring

When explicitly asked to refactor:

- Preserve observable behavior unless a change is requested.
- Keep public APIs stable where practical.
- Separate structural changes from behavior changes.
- Run tests before and after when possible.
- Avoid style migrations at the same time.
- Remove real duplication, not merely visually similar code.

A refactor should make maintenance easier.

Different is not automatically better.

---

## Avoid Premature DRY

Do not extract two similar pieces of code solely because they look alike.

Duplication can be cheaper than the wrong abstraction.

Extract common logic when:

- The behavior is genuinely the same.
- Changes should logically occur together.
- The abstraction has a clear name.

Do not force unrelated concepts through a generic helper simply to eliminate repeated lines.

---

## Avoid Fake Extensibility

Do not add plugin hooks, registries, abstract factories, or protocol layers for hypothetical future features.

Build extension points when an actual second implementation or external integration requires them.

---

## Avoid Placeholder Architecture

Do not leave speculative TODO comments such as:

```python
# TODO: Add advanced validation
# TODO: Add caching
# TODO: Add retry support
# TODO: Add monitoring
```

unless the user explicitly asked for scaffolding.

Complete the requested behavior.

Do not advertise hypothetical work inside production code.

---

## Avoid AI-Generated Prose

Comments, docstrings, log messages, and errors should sound like normal engineering text.

Avoid phrases such as:

```text
This function is responsible for...
This method ensures that...
The following code handles...
In order to...
It is important to note that...
This provides a robust and flexible solution...
```

Bad:

```python
# This function is responsible for ensuring that the weapon
# data is properly validated before processing can continue.
```

Better:

```python
# External weapon manifests are not guaranteed to match our schema.
```

---

## Avoid Decorative Comments

Do not add section banners like:

```python
# ========================================
# INITIALIZATION
# ========================================
```

unless the repository already uses that convention.

Good module structure should usually make them unnecessary.

---

## Avoid Excessive Blank Lines

Use PEP 8 or the project's formatter.

Do not space every logical statement apart.

Bad:

```python
weapon = load_weapon(path)


stats = parse_stats(weapon)


return stats
```

Readable Python is compact.

---

## Avoid Unnecessary Parentheses

Do not add parentheses where Python syntax does not need them and readability does not improve.

Bad:

```python
if (weapon is None):
    return None
```

Prefer:

```python
if weapon is None:
    return None
```

Use parentheses naturally for multiline expressions.

---

## Avoid Explicit `return None`

At the end of a function, this:

```python
return None
```

is sometimes useful for clarity but often unnecessary.

Do not remove it mechanically either.

Use it when the return contract benefits from being explicit.

---

## Avoid `pass` Placeholders

Do not leave:

```python
def load_weapon():
    pass
```

in finished work unless an abstract or stub API intentionally requires it.

Use:

```python
raise NotImplementedError
```

only for APIs genuinely meant to be implemented later.

---

## Avoid Unnecessary `else` on Loops

Python's `for/else` and `while/else` can be useful but unfamiliar.

Use them when they clearly express "loop completed without breaking."

Do not use them merely to showcase Python features.

---

## Avoid Clever One-Liners

Do not compress important logic into dense one-liners.

Bad:

```python
return next((x for x in items if valid(x)), None) if items else None
```

when a straightforward loop would be easier to maintain.

Compact code is not automatically better code.

---

## Avoid Chained Side Effects

Do not hide multiple operations inside expressions.

Prefer explicit statements when work has meaningful side effects.

Readable sequencing is valuable.

---

## Avoid Mutable Shared Defaults in Dataclasses

Do not write:

```python
@dataclass
class Inventory:
    weapons: list[Weapon] = []
```

Use:

```python
from dataclasses import field


@dataclass
class Inventory:
    weapons: list[Weapon] = field(default_factory=list)
```

This prevents shared mutable state across instances.

---

## `default_factory`

Use `default_factory` for mutable defaults such as:

```python
list
dict
set
```

Do not use a lambda when the type itself works:

```python
field(default_factory=list)
```

rather than:

```python
field(default_factory=lambda: [])
```

---

## Frozen Data vs Mutable Data

Choose deliberately.

A configuration model may reasonably be frozen:

```python
@dataclass(frozen=True)
class Config:
    api_url: str
    timeout: float
```

A cache or session object naturally mutates.

Do not force one style across all models.

---

## Public APIs

Be deliberate with exported library APIs.

For public functions:

- Use clear parameters.
- Use useful type hints.
- Keep backward compatibility in mind.
- Avoid leaking implementation-specific objects.
- Document non-obvious behavior.
- Do not add parameters callers do not need.

Internal functions can stay simpler.

---

## Return Types

Use explicit return types for public or important functions.

Example:

```python
def parse_weapon(data: WeaponData) -> Weapon:
    ...
```

Small private helpers may rely on inference if that matches project style and tooling.

Do not annotate everything merely to increase annotation density.

---

## Positional vs Keyword Arguments

Use positional arguments when their meaning is obvious:

```python
load_weapon(path)
```

Use keywords when values might otherwise be ambiguous:

```python
export_weapon(
    weapon,
    include_model=True,
    include_thumbnail=False,
)
```

Do not force callers to spell out obvious one-argument calls.

---

## Booleans as Arguments

Prefer named keyword arguments for booleans.

Bad:

```python
export_weapon(weapon, True, False)
```

Better:

```python
export_weapon(
    weapon,
    include_model=True,
    include_thumbnail=False,
)
```

This avoids guessing what each boolean means.

---

## Public Mutable Collections

Do not expose mutable internal collections if callers should not modify them.

Depending on the design, return:

- A tuple.
- An iterator.
- A copy.
- A read-only interface by convention.

Do not copy everything defensively if callers are allowed to mutate it.

---

## Iterators

Return iterators when lazy consumption provides actual value.

Example:

```python
def iter_weapons() -> Iterator[Weapon]:
    ...
```

Do not return generators solely because they seem more Pythonic.

If callers need indexing, length, or repeated traversal, a concrete collection may be better.

---

## `yield`

Use generators for streaming or lazy sequences.

Do not write a generator for a three-item static collection unless it simplifies the design.

---

## Context-Specific Code

Prefer explicit code tailored to the actual application.

Do not turn application code into a reusable framework unless that is the project goal.

A weapon parser can simply be a weapon parser.

It does not need to become a generic:

```text
EntityProcessingFramework
```

for hypothetical future entities.

---

## Security-Sensitive Operations

Prefer mature libraries and standard mechanisms for:

- Cryptography.
- Authentication.
- Password hashing.
- TLS.
- Token verification.
- XML security.
- Archive extraction.
- Untrusted serialization.

Do not hand-roll security-sensitive algorithms.

---

## Archive Extraction

When extracting user-controlled archives, consider path traversal such as `../`.

Do not blindly extract untrusted archives into arbitrary directories.

Use appropriate safety checks or trusted library behavior.

---

## File Paths from Users

Be deliberate when user-controlled paths can access arbitrary files.

Normalize or constrain paths where the application requires a sandbox or root directory.

Do not add path restrictions to normal developer tooling if unrestricted paths are intentionally supported.

---

## Encoding

Use UTF-8 explicitly for normal text files:

```python
path.read_text(encoding="utf-8")
```

and:

```python
path.write_text(text, encoding="utf-8")
```

when appropriate.

Do not assume platform-default encoding for portable applications.

---

## Line Endings

Do not manually normalize line endings unless required.

Use text mode and the project's formatter or version-control settings.

Avoid rewriting entire files due solely to newline differences.

---

## Randomness

Use:

```python
random
```

for non-security-sensitive randomness.

Use:

```python
secrets
```

for tokens, passwords, authentication codes, and other security-sensitive values.

Do not use `random` for secrets.

---

## UUIDs

Use `uuid.uuid4()` for normal random UUID generation when suitable.

Do not build custom identifier formats unless the domain requires them.

---

## Serialization

Prefer explicit serialization formats.

Avoid relying on object internals or `__dict__` for stable APIs.

Good:

```python
{
    "id": weapon.id,
    "name": weapon.name,
}
```

or use the established serialization library.

Do not expose implementation details accidentally.

---

## Public Error Contracts

If a library function raises certain exceptions intentionally, keep those exceptions predictable.

Do not translate every exception into one generic error if callers need meaningful distinctions.

Likewise, do not expose every low-level dependency exception if doing so couples callers unnecessarily.

Choose the boundary deliberately.

---

## Before Finishing

Review the change and remove:

- Redundant comments.
- Tutorial-style prose.
- Unnecessary classes.
- Unnecessary dataclasses.
- Unnecessary protocols.
- Unnecessary abstract base classes.
- Excessive type machinery.
- Over-generalized generics.
- Generic AI-style names.
- Pass-through service layers.
- Utility classes.
- Factories with one implementation.
- Excessive validation.
- Defensive checks for impossible internal states.
- Broad exception handlers.
- Duplicate logging.
- Debugging `print()` statements.
- Mutable default arguments.
- Unused helpers.
- Unused imports.
- Placeholder TODOs.
- Speculative configuration.
- Speculative extensibility.
- Unrelated refactors.
- Dead code.

Then run the project's existing checks where available.

Typical Python checks might include:

```bash
ruff check .
ruff format --check .
pytest
mypy .
```

or:

```bash
pyright
```

But do not assume these exact tools or commands exist.

Inspect the repository first and use its configured workflow.

The final code should look like it naturally belongs in the repository rather than like a standalone AI-generated solution.

It should feel like Python written by an experienced maintainer: direct, readable, restrained, and appropriately typed without turning ordinary application code into a framework.