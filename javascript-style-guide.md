# JavaScript Style Guide

Write JavaScript as an experienced professional developer maintaining a real production codebase.

The goal is clean, idiomatic, maintainable JavaScript—not code that looks generated, over-engineered, excessively defensive, or written as a tutorial.

The most important rule:

> Do not optimize for demonstrating programming practices. Optimize for producing the smallest idiomatic production-quality change that an experienced maintainer would reasonably write.

## General Principles

Prefer:

- Simple solutions over clever abstractions.
- Existing project conventions over personal preferences.
- Idiomatic modern JavaScript.
- Small, focused changes.
- Clear code over explanatory comments.
- Functions and modules over unnecessary classes.
- Built-in language and platform APIs over custom abstractions.
- Direct implementation over speculative extensibility.
- Domain terminology over generic architectural names.
- Explicit data flow over hidden magic.
- Runtime validation at real trust boundaries, not everywhere.

Do not refactor unrelated code unless required by the task.

Do not introduce classes, factories, wrappers, managers, services, helpers, or abstraction layers unless they solve an actual problem.

Do not write JavaScript as if it were Java, C#, or TypeScript.

---

## Match the Existing Codebase

Before writing code, inspect nearby files and follow the repository's established conventions for:

- Naming.
- Module structure.
- File organization.
- Import and export style.
- Semicolon usage.
- Quote style.
- Error handling.
- Logging.
- Async patterns.
- Validation.
- State management.
- Framework conventions.
- Testing.
- ESLint.
- Prettier.
- Package manager.
- Supported Node or browser versions.

Check relevant files such as:

```text
package.json
eslint.config.js
.eslintrc
.prettierrc
vite.config.js
webpack.config.js
rollup.config.js
babel.config.js
jsconfig.json
```

Consistency with the repository is more important than imposing a preferred style.

Do not perform style migrations as part of unrelated work.

---

## Runtime Compatibility

Determine the project's supported runtime before using newer syntax or APIs.

Check:

```json
{
  "engines": {
    "node": ">=20"
  }
}
```

or the project's browser targets, deployment platform, bundler, or CI configuration.

Do not assume the newest Node.js or browser features are available.

Avoid introducing APIs unsupported by the declared runtime.

---

## Naming

Use normal JavaScript naming conventions:

- `camelCase` for variables, functions, parameters, and properties.
- `PascalCase` for classes and components.
- `UPPER_SNAKE_CASE` for genuine constants where appropriate.
- Boolean variables should usually describe a state or condition.

Good:

```js
assetPath
weaponDefinition
packageName
isLoaded
hasPermission
canRetry

loadPackage()
resolveAsset()
parseWeaponData()
```

Avoid vague AI-style names:

```js
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

```js
const data = getData();
const result = processData(data);
```

Better:

```js
const weapon = getWeaponDefinition();
const stats = parseWeaponStats(weapon);
```

Do not make names excessively verbose.

Avoid:

```js
const successfullyParsedWeaponConfigurationResult =
  parseWeaponConfiguration();
```

when:

```js
const weaponConfig = parseWeaponConfiguration();
```

is clear.

---

## Functions First

Prefer functions for stateless behavior.

Good:

```js
export function parseWeaponStats(data) {
  return {
    damage: data.damage,
    fireRate: data.fireRate,
  };
}
```

Do not automatically create:

```js
export class WeaponStatsParser {
  parse(data) {
    // ...
  }
}
```

unless the object genuinely requires:

- State.
- Multiple coordinated methods.
- Lifecycle behavior.
- Dependency ownership.
- Encapsulation.
- Polymorphism.

JavaScript is naturally module-oriented.

Use modules and functions when they are sufficient.

---

## Function Size

Keep functions focused.

Prefer:

```js
const pkg = await provider.loadPackage(path);
return mapper.map(pkg);
```

over:

```js
const pkg = await loadPackageFromProvider(provider, path);
const mappedPackage =
  mapLoadedPackageUsingMapper(mapper, pkg);

return mappedPackage;
```

Do not extract helpers solely to reduce a function's line count.

Extract functions when they:

- Represent a meaningful operation.
- Are reused.
- Remove genuinely complex logic.
- Improve testability.
- Clarify difficult control flow.

Avoid names such as:

```js
processData()
handleStuff()
performOperation()
executeLogic()
doWork()
```

Use names that describe the actual domain behavior.

---

## Avoid Excessive Tiny Helpers

Do not hide obvious property access behind unnecessary functions.

Avoid:

```js
function getWeaponName(weapon) {
  return weapon.name;
}

function getWeaponDamage(weapon) {
  return weapon.damage;
}
```

when:

```js
weapon.name
weapon.damage
```

is clearer.

Every helper should provide actual semantic value.

---

## Prefer Plain Objects for Data

Use plain objects for simple structured data.

Good:

```js
const weapon = {
  id,
  name,
  damage,
  fireRate,
};
```

Do not automatically create classes for data containers.

Avoid:

```js
class WeaponDto {
  constructor(id, name, damage, fireRate) {
    this.id = id;
    this.name = name;
    this.damage = damage;
    this.fireRate = fireRate;
  }
}
```

when a plain object communicates the same thing more simply.

Use classes when behavior and state belong together.

---

## Classes

Use classes when object identity, state, lifecycle, or encapsulated behavior makes them useful.

Good:

```js
class PackageCache {
  #packages = new Map();

  get(path) {
    return this.#packages.get(path);
  }

  set(path, pkg) {
    this.#packages.set(path, pkg);
  }
}
```

Do not create classes merely because a concept is a noun.

Avoid architecture like:

```text
WeaponManager
WeaponService
WeaponProvider
WeaponHandler
WeaponProcessor
WeaponCoordinator
```

unless those represent distinct real responsibilities.

---

## Avoid Java-Style Architecture

Do not emulate enterprise Java or C# automatically.

Avoid:

```js
class WeaponParserInterface {
  parse(data) {
    throw new Error("Not implemented");
  }
}

class DefaultWeaponParser extends WeaponParserInterface {
  parse(data) {
    // ...
  }
}
```

when this is enough:

```js
function parseWeapon(data) {
  // ...
}
```

Do not create:

- Interface-like base classes for one implementation.
- Abstract classes for one subclass.
- Factories that construct one type.
- Static utility classes.
- Getter/setter boilerplate.
- Repository layers around straightforward APIs.
- Dependency containers for small applications.

JavaScript should feel like JavaScript.

---

## Avoid Static Utility Classes

Bad:

```js
class WeaponUtils {
  static normalizeName(name) {
    return name.trim().toLowerCase();
  }
}
```

Prefer:

```js
export function normalizeWeaponName(name) {
  return name.trim().toLowerCase();
}
```

Modules already provide namespaces.

---

## Private Class Fields

Use private fields when actual encapsulation helps:

```js
class Cache {
  #items = new Map();
}
```

Do not convert every property into a private field mechanically.

Sometimes a normal property is entirely appropriate.

Follow project conventions.

---

## Constructors

Keep constructors simple.

Avoid constructors that:

- Perform network requests.
- Read files.
- Start background work.
- Register global listeners unexpectedly.
- Perform expensive computation.

Prefer explicit initialization methods or factory functions when construction requires substantial work.

---

## Factory Functions

Factory functions can be useful when:

- Creating closures.
- Hiding internal state.
- Wiring dependencies.
- Returning related operations.

Example:

```js
export function createWeaponLoader(provider) {
  return async function loadWeapon(path) {
    const pkg = await provider.loadPackage(path);
    return parseWeapon(pkg);
  };
}
```

Do not create a factory when direct construction is clearer.

Avoid:

```js
createWeaponParserFactory()
```

that only returns:

```js
new WeaponParser()
```

---

## Objects

Use object shorthand when clear:

```js
const weapon = {
  id,
  name,
  damage,
};
```

rather than:

```js
const weapon = {
  id: id,
  name: name,
  damage: damage,
};
```

Do not use shorthand if it makes the code harder to understand.

---

## Destructuring

Use destructuring when it improves readability.

Good:

```js
const { id, name, stats } = weapon;
```

Do not destructure huge objects solely because the syntax is available.

Sometimes:

```js
weapon.stats.damage
```

is clearer than introducing many local names.

Use context.

---

## Nested Destructuring

Avoid deeply nested destructuring when it hides structure.

Bad:

```js
const {
  metadata: {
    stats: {
      damage: {
        base,
      },
    },
  },
} = weapon;
```

Prefer:

```js
const baseDamage = weapon.metadata.stats.damage.base;
```

when that is easier to scan.

---

## Object Spread

Use object spread for straightforward copying or updates.

Good:

```js
const updatedWeapon = {
  ...weapon,
  damage: newDamage,
};
```

Avoid long spread chains with unclear precedence:

```js
const result = {
  ...defaults,
  ...config,
  ...environment,
  ...overrides,
  ...runtime,
};
```

unless the precedence is intentional and obvious.

Explicit assignments may be clearer.

---

## Avoid Excessive Cloning

Do not spread objects merely to avoid all mutation.

Bad:

```js
const updated = {
  ...object,
  nested: {
    ...object.nested,
    options: {
      ...object.nested.options,
      enabled: true,
    },
  },
};
```

when local mutation is safe and expected.

Use the state model appropriate to the framework and codebase.

Immutability is useful, not sacred.

---

## Arrays

Use array methods when they make intent clear.

Good:

```js
const weapons = assets
  .filter((asset) => asset.type === "weapon")
  .map(parseWeapon);
```

Use loops when:

- Multiple operations happen per item.
- Early exit matters.
- Error handling differs by item.
- Mutation is intentional.
- The chain becomes hard to read.

Do not force all iteration into method chains.

---

## Avoid Clever `reduce`

Do not use `reduce()` when a loop or dedicated API communicates the operation more clearly.

Bad:

```js
const weaponsById = weapons.reduce(
  (result, weapon) => ({
    ...result,
    [weapon.id]: weapon,
  }),
  {},
);
```

Better:

```js
const weaponsById = new Map();

for (const weapon of weapons) {
  weaponsById.set(weapon.id, weapon);
}
```

or:

```js
const weaponsById = Object.fromEntries(
  weapons.map((weapon) => [weapon.id, weapon]),
);
```

Choose the clearest representation.

---

## `Map`

Use `Map` when:

- Keys are not necessarily strings.
- Insertion order matters.
- Frequent insertion/deletion is expected.
- Map semantics are clearer than object semantics.

Example:

```js
const weaponsById = new Map();
```

Do not replace plain objects with `Map` automatically when objects better match the data or serialization format.

---

## `Set`

Use `Set` when uniqueness or membership is the point.

```js
const processedPaths = new Set();
```

Do not use arrays for large repeated membership checks when a set is more appropriate.

Do not use sets where duplicates or indexing matter.

---

## Loops

Use the loop that best communicates intent.

Good:

```js
for (const weapon of weapons) {
  processWeapon(weapon);
}
```

Avoid:

```js
for (let i = 0; i < weapons.length; i++) {
  processWeapon(weapons[i]);
}
```

unless the index is actually needed.

Use normal loops freely.

Readable imperative code is not inferior to array chaining.

---

## `for...in`

Use `for...in` for enumerable object keys when appropriate.

Do not use it for arrays.

Prefer:

```js
for (const item of items) {
  // ...
}
```

instead of:

```js
for (const index in items) {
  // ...
}
```

---

## Control Flow

Prefer guard clauses over deep nesting.

Good:

```js
if (!weapon) {
  return null;
}

if (!weapon.loaded) {
  await weapon.load();
}

return weapon;
```

Avoid:

```js
if (weapon) {
  if (!weapon.loaded) {
    await weapon.load();
  }

  return weapon;
}

return null;
```

Keep the normal path easy to follow.

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

Bad:

```js
if (!weapon) {
  return null;
} else {
  return weapon.name;
}
```

Prefer:

```js
if (!weapon) {
  return null;
}

return weapon.name;
```

Use `else` when it actually improves readability.

---

## Ternaries

Use ternaries for simple value selection.

Good:

```js
const label = enabled ? "Enabled" : "Disabled";
```

Avoid nested ternaries for substantial control flow.

Bad:

```js
const label = active
  ? selected
    ? loading
      ? "Loading"
      : "Selected"
    : "Active"
  : "Inactive";
```

Use normal branching.

---

## Boolean Logic

Write conditions so intent is obvious.

Prefer:

```js
if (!weapon) {
  return;
}
```

when all falsy values are invalid.

Use explicit checks when falsy values are valid:

```js
if (value == null) {
  return;
}
```

or:

```js
if (value === undefined) {
  return;
}
```

depending on the contract.

Do not use clever coercion just to save characters.

---

## `==` vs `===`

Prefer strict equality:

```js
value === 0
value !== null
```

Use loose equality only when its coercion behavior is deliberate and well understood.

One legitimate example can be:

```js
value == null
```

when the intent is explicitly to match both `null` and `undefined`.

Do not introduce loose comparisons casually.

Follow lint rules.

---

## Truthiness

Use truthiness when it matches the domain.

Good:

```js
if (!items.length) {
  return;
}
```

Be careful when valid values may include:

```text
0
false
""
```

Do not replace precise checks with broad truthiness when semantics differ.

---

## Optional Chaining

Use optional chaining when missing values are expected.

Good:

```js
const displayName = weapon.metadata?.displayName;
```

Do not hide required invariants behind long chains:

```js
const damage =
  package?.data?.weapon?.stats?.base?.damage;
```

if all of those properties should exist after validation.

Validate or establish the structure once, then use it normally.

---

## Nullish Coalescing

Use `??` when defaults should only apply to `null` or `undefined`.

Good:

```js
const retries = config.retries ?? 3;
```

Avoid:

```js
const retries = config.retries || 3;
```

when `0` is a valid value.

---

## Logical Assignment

Use operators such as:

```js
||=
&&=
??=
```

when they make the intent clearer.

Example:

```js
config.timeout ??= DEFAULT_TIMEOUT;
```

Do not use them where the assignment semantics are easy to misread.

---

## Comments

Do not narrate the code.

Bad:

```js
// Check if the weapon exists
if (!weapon) {
  // Return null if it does not exist
  return null;
}
```

Good:

```js
if (!weapon) {
  return null;
}
```

Write comments only when they explain something the code itself cannot make obvious, such as:

- A browser limitation.
- A third-party bug.
- A compatibility workaround.
- A protocol requirement.
- A performance tradeoff.
- A non-obvious domain constraint.
- Why strange-looking behavior exists.

Prefer explaining **why**, not **what**.

Avoid:

```js
// Initialize array
// Loop through items
// Check value
// Return result
// Handle error
```

---

## Avoid AI-Looking Comments

Avoid phrases like:

```text
This function is responsible for...
This ensures that...
The following code handles...
In order to...
It is important to note that...
This provides a robust and flexible solution...
```

Bad:

```js
// This function is responsible for ensuring that all weapon
// data is properly validated before processing can continue.
```

Better:

```js
// External weapon manifests are not guaranteed to match our schema.
```

---

## JSDoc

Use JSDoc when it adds meaningful information.

Useful cases include:

- Public library APIs.
- JavaScript projects using `checkJs`.
- Complex object shapes.
- External contracts.
- Important side effects.
- Units and formats.
- Deprecation notices.
- APIs consumed by editors or tooling.

Do not add JSDoc to every obvious function.

Avoid:

```js
/**
 * Gets the weapon name.
 *
 * @param {Object} weapon The weapon.
 * @returns {string} The weapon name.
 */
function getWeaponName(weapon) {
  return weapon.name;
}
```

The implementation already explains itself.

---

## JSDoc Types

In JavaScript projects that use JSDoc for static analysis, use it intentionally.

Example:

```js
/**
 * @typedef {Object} Weapon
 * @property {string} id
 * @property {string} name
 * @property {number} damage
 */
```

Do not recreate TypeScript's entire type system through enormous JSDoc definitions.

If type documentation becomes more complicated than the code, reconsider the design or whether TypeScript would be more appropriate for that project.

Do not convert a JavaScript project to TypeScript unless explicitly requested.

---

## Runtime Validation

JavaScript has no compile-time type enforcement.

Validate data at trust boundaries when correctness matters.

Examples:

- API responses.
- User input.
- Local storage.
- Configuration files.
- Environment variables.
- Plugin messages.
- MCP inputs.
- WebSocket payloads.
- PostMessage events.
- Parsed JSON.

Do not repeatedly validate values that have already crossed a trusted boundary.

---

## Avoid Fake Runtime Safety

Do not add checks for every internal property merely because JavaScript is dynamic.

Bad:

```js
if (
  weapon &&
  typeof weapon === "object" &&
  weapon !== null &&
  "name" in weapon &&
  typeof weapon.name === "string"
) {
  // ...
}
```

inside code where `weapon` was already validated earlier.

Validate once at the boundary.

Trust internal contracts afterward.

---

## Schema Validation

Use the project's existing validation library if one exists.

Examples may include:

```text
Zod
Valibot
Joi
Ajv
Yup
ArkType
```

Do not add another validation library without a reason.

Do not introduce a schema package when a couple of straightforward checks are sufficient.

---

## `typeof`

Remember JavaScript quirks.

For example:

```js
typeof null === "object"
```

So object validation should account for `null` when necessary.

Use:

```js
if (
  typeof value !== "object" ||
  value === null
) {
  // ...
}
```

when validating unknown object values.

---

## `Array.isArray`

Use:

```js
Array.isArray(value)
```

for array checks.

Do not rely on:

```js
typeof value === "object"
```

to distinguish arrays.

---

## `instanceof`

Use `instanceof` when prototype identity is meaningful and reliable.

Example:

```js
if (error instanceof Error) {
  // ...
}
```

Be cautious across:

- iframes.
- realms.
- multiple package copies.
- serialization boundaries.

Do not use `instanceof` as universal data validation.

---

## Errors

Use errors for exceptional failures.

Do not use exceptions for routine control flow when a normal return value is clearer.

Good:

```js
function findWeapon(id) {
  return weapons.get(id);
}
```

when missing values are normal.

Use an exception when the operation itself failed.

---

## Error Handling

Catch errors only when you can:

- Recover.
- Add useful context.
- Translate the failure.
- Perform cleanup.
- Return an expected application-level result.

Avoid:

```js
try {
  return await loadPackage(path);
} catch (error) {
  console.error(error);
  throw error;
}
```

This adds no value.

Let the error propagate.

---

## Preserve Error Causes

When wrapping errors, preserve the original cause:

```js
throw new Error(
  `Failed to load package: ${path}`,
  { cause: error },
);
```

when supported by the runtime.

Do not throw unrelated new errors that erase useful context.

---

## Error Messages

Write actionable messages.

Good:

```text
Unable to load package "/Game/Weapons/Rifle_A": package was not found.
```

Bad:

```text
Something went wrong.
```

Include relevant identifiers when appropriate:

- Paths.
- IDs.
- Asset names.
- URLs.
- Operation names.

Do not expose secrets.

---

## Custom Error Classes

Create custom error classes when callers genuinely benefit from distinguishing failure categories.

Example:

```js
class PackageLoadError extends Error {
  constructor(path, options) {
    super(`Unable to load package: ${path}`, options);
    this.name = "PackageLoadError";
    this.path = path;
  }
}
```

Do not create a custom error subclass for every possible failure.

Avoid deep error hierarchies without a practical reason.

---

## Async Code

Use `async` only for genuinely asynchronous work.

Bad:

```js
async function getCount(items) {
  return items.length;
}
```

Prefer:

```js
function getCount(items) {
  return items.length;
}
```

Do not make functions async simply because their callers are async.

---

## Avoid Unnecessary `await`

Prefer:

```js
return loadWeapon();
```

over:

```js
return await loadWeapon();
```

when no local error handling, cleanup, or stack behavior requires the `await`.

Use `await` when it improves actual control flow.

---

## Parallel Async Work

Run independent asynchronous work concurrently when appropriate.

Good:

```js
const [weapons, attachments] =
  await Promise.all([
    loadWeapons(),
    loadAttachments(),
  ]);
```

Do not serialize independent work unnecessarily:

```js
const weapons = await loadWeapons();
const attachments = await loadAttachments();
```

However, do not use `Promise.all` when:

- Operations depend on each other.
- Side effects must be ordered.
- Concurrency would overload a service.
- Failure handling needs to be isolated.

---

## Avoid Async `forEach`

Do not write:

```js
items.forEach(async (item) => {
  await processItem(item);
});
```

The promises are not awaited by `forEach`.

Use:

```js
for (const item of items) {
  await processItem(item);
}
```

for sequential execution.

Or:

```js
await Promise.all(
  items.map(processItem),
);
```

when concurrent execution is appropriate.

---

## `Promise.allSettled`

Use `Promise.allSettled()` when all operations should finish even if some fail.

Do not use it automatically where failure should stop the operation.

Choose based on actual semantics.

---

## Abort Signals

Use `AbortController` and `AbortSignal` when cancellation matters.

Example:

```js
async function loadPackage(path, { signal } = {}) {
  // ...
}
```

Propagate existing signals through fetches and long-running operations.

Do not create cancellation infrastructure for trivial synchronous code.

---

## Fetch

Check HTTP status when appropriate.

Bad:

```js
const response = await fetch(url);
return response.json();
```

Better:

```js
const response = await fetch(url);

if (!response.ok) {
  throw new Error(
    `Request failed with ${response.status} ${response.statusText}`,
  );
}

return response.json();
```

Do not assume `fetch()` rejects on HTTP 404 or 500.

It normally rejects only on network-level failure.

---

## HTTP Timeouts

Where supported by the runtime or project libraries, use bounded request timeouts for remote services.

Do not allow external calls to hang indefinitely in long-running server or CLI workflows.

Do not build custom timeout wrappers if the platform already supports them.

---

## Retries

Retry only failures that are plausibly temporary.

Examples:

- Network failures.
- 429 rate limits.
- Temporary service outages.

Do not retry:

- Invalid input.
- Authentication failures requiring changed credentials.
- Deterministic parsing errors.
- Programming errors.

Use bounded retries.

Do not create a retry framework unless needed.

---

## Modules

Follow the repository's module system.

Do not mix CommonJS and ESM casually.

ESM:

```js
import { readFile } from "node:fs/promises";

export function parseWeapon() {}
```

CommonJS:

```js
const fs = require("node:fs");

module.exports = {
  parseWeapon,
};
```

Respect the environment.

---

## ESM

Check `package.json`:

```json
{
  "type": "module"
}
```

before changing module syntax.

Do not perform an ESM migration as part of unrelated work.

---

## CommonJS

Do not convert working CommonJS code to ESM simply because ESM is newer.

Likewise, do not introduce CommonJS into an established ESM project.

Consistency matters more than novelty.

---

## Import Paths

Respect runtime and bundler requirements.

Node ESM may require file extensions:

```js
import { parseWeapon } from "./weapon.js";
```

Do not blindly remove `.js` because the source file is conceptually related to another language or build system.

Follow the actual runtime.

---

## Imports

Keep imports intentional.

Do not import entire libraries when a smaller API is sufficient if the library supports granular imports.

Avoid import churn unrelated to the task.

Use established aliases where configured.

---

## Exports

Export only what callers need.

Do not expose every helper "just in case."

Keep implementation details private to the module.

Avoid giant barrel modules that re-export the entire project unless the architecture intentionally uses them.

---

## Default vs Named Exports

Follow repository conventions.

Do not switch between default and named exports as an unrelated cleanup.

Named exports can be useful for refactoring:

```js
export function parseWeapon() {}
```

Framework-required or conventionally default exports are also fine.

---

## File Organization

Keep files cohesive.

Do not enforce "one function per file."

Several closely related functions can reasonably live together.

Avoid:

```text
weapon-helper.js
weapon-utils.js
weapon-manager.js
weapon-service.js
weapon-processor.js
```

unless each file genuinely owns a different responsibility.

Prefer domain-oriented modules.

---

## Avoid Generic Utility Files

Do not dump unrelated functions into:

```text
utils.js
helpers.js
common.js
misc.js
```

when a domain-specific name is clearer.

Better:

```text
weapon-paths.js
asset-format.js
package-validation.js
```

Small local helpers can remain near the code that uses them.

---

## Dependency Injection

JavaScript usually does not need a full DI framework.

Simple dependency passing is often enough:

```js
export function createWeaponLoader(provider) {
  return async function loadWeapon(path) {
    const pkg = await provider.loadPackage(path);
    return parseWeapon(pkg);
  };
}
```

Do not add:

```text
DependencyContainer
ServiceRegistry
ProviderResolver
InjectionFactory
```

for a small or straightforward application.

---

## Avoid Pass-Through Layers

Bad:

```js
class WeaponService {
  constructor(repository) {
    this.repository = repository;
  }

  getWeapon(id) {
    return this.repository.getWeapon(id);
  }
}
```

if the service adds no behavior.

Every layer should earn its existence.

---

## Configuration

Centralize meaningful configuration when the project benefits from it.

Do not make every literal configurable.

Avoid:

```js
const options = {
  enableParsing: true,
  enableValidation: true,
  enableNormalization: true,
  enableTransformation: true,
};
```

when those operations are mandatory.

Configuration adds states and complexity.

Expose it only when callers need control.

---

## Environment Variables

Parse environment variables deliberately.

Remember they are strings.

Bad:

```js
const debug = Boolean(process.env.DEBUG);
```

because:

```text
DEBUG=false
```

still becomes `true`.

Prefer intentional parsing.

Example:

```js
const debug = process.env.DEBUG === "true";
```

Validate required environment variables near application startup.

Do not scatter raw `process.env` access across the codebase without a reason.

---

## Constants

Name values with real domain meaning.

Good:

```js
const MAX_RETRY_ATTEMPTS = 3;
const DEFAULT_TIMEOUT_MS = 10_000;
```

Avoid:

```js
const ZERO = 0;
const ONE = 1;
const TRUE = true;
```

A constant should communicate meaning.

---

## Magic Numbers

Do not extract every literal merely because it appears in code.

This is fine:

```js
if (items.length === 0) {
  return;
}
```

A domain-specific timeout or retry limit is more likely to deserve a name.

---

## String Formatting

Prefer template literals for interpolation:

```js
const message =
  `Unable to load package: ${path}`;
```

Do not use template literals when no interpolation or multiline formatting is needed:

```js
const type = "weapon";
```

not:

```js
const type = `weapon`;
```

---

## Template Literals

Avoid building huge HTML strings manually when the project already uses a templating or UI framework.

For small DOM operations, template literals can be fine.

Be careful with untrusted values.

---

## DOM Manipulation

Use standard DOM APIs directly when they are sufficient.

Good:

```js
const button = document.querySelector(
  "[data-action='load']",
);

button?.addEventListener(
  "click",
  handleLoad,
);
```

Do not create custom wrapper classes around trivial DOM operations unless repetition or complexity justifies them.

---

## DOM Queries

Cache DOM references when reused frequently.

Do not cache every element automatically.

Prefer selectors tied to stable semantics such as:

```text
data-* attributes
IDs
well-defined classes
```

rather than brittle deeply nested CSS selectors.

---

## `innerHTML`

Avoid assigning untrusted content to `innerHTML`.

Bad:

```js
element.innerHTML = userInput;
```

Prefer:

```js
element.textContent = userInput;
```

when plain text is intended.

Use a proven sanitizer when HTML from an untrusted source must be rendered.

---

## Event Listeners

Name event handlers clearly.

Good:

```js
function handleSearchInput(event) {
  // ...
}
```

Do not create handler wrappers that only forward arguments unless they improve readability.

This is fine:

```js
button.addEventListener(
  "click",
  () => removeWeapon(id),
);
```

You do not need:

```js
function handleRemoveWeaponClick() {
  removeWeapon(id);
}
```

unless the handler has real logic or is reused.

---

## Event Delegation

Use event delegation when many dynamic children share behavior.

Example:

```js
list.addEventListener("click", (event) => {
  const button = event.target.closest(
    "[data-remove-id]",
  );

  if (!button) {
    return;
  }

  removeWeapon(button.dataset.removeId);
});
```

Do not introduce delegation for one static button.

---

## Browser Storage

Treat data from:

```text
localStorage
sessionStorage
IndexedDB
```

as external serialized data.

Validate or parse carefully.

Do not assume storage contents always match the latest code.

---

## URL APIs

Use standard URL APIs instead of manual string concatenation.

Prefer:

```js
const url = new URL(
  "/api/weapons",
  window.location.origin,
);

url.searchParams.set("category", category);
```

over manually building query strings.

---

## Node.js

Use modern Node APIs supported by the project's version.

Prefer:

```js
import { readFile } from "node:fs/promises";
```

for promise-based file I/O when appropriate.

Do not wrap callback APIs manually if a promise API already exists.

---

## File Paths

Use `node:path` for platform-safe paths.

Example:

```js
import path from "node:path";

const configPath = path.join(
  root,
  "config",
  "settings.json",
);
```

Do not build filesystem paths with hard-coded `/` or `\` unless the path is intentionally URL-like.

---

## File I/O

Use the simplest appropriate API.

Example:

```js
const text = await readFile(
  configPath,
  "utf8",
);
```

Do not introduce streams for tiny files.

Use streams when data size or incremental processing justifies them.

---

## Streams

Use Node streams for large or continuous data.

Do not use streams merely to demonstrate advanced Node.js.

For small files:

```js
await readFile(...)
```

is often clearer.

---

## Subprocesses

Use `spawn`, `execFile`, or established project helpers deliberately.

Prefer argument arrays over shell command construction.

Avoid:

```js
exec(
  `tool --file "${filename}"`,
);
```

when:

```js
execFile(
  "tool",
  ["--file", filename],
);
```

is sufficient.

Avoid `shell: true` unless shell semantics are actually required.

---

## Security

Never:

- Hard-code secrets.
- Use `eval()` on untrusted content.
- Use `new Function()` on untrusted content.
- Build SQL with raw string concatenation.
- Insert untrusted HTML directly.
- Build shell commands from untrusted strings.
- Disable TLS verification casually.
- Trust external object shapes blindly.

Use established project mechanisms.

---

## `eval`

Avoid:

```js
eval(...)
new Function(...)
```

except in highly specialized controlled environments.

Do not use dynamic code execution to avoid writing straightforward parsing or dispatch logic.

---

## Prototype Mutation

Avoid modifying built-in prototypes:

```js
Array.prototype.foo = ...
```

This can conflict with libraries and future platform APIs.

Prefer normal functions or local abstractions.

---

## Monkey Patching

Do not monkey patch third-party libraries unless there is a strong compatibility or testing reason.

Prefer supported extension points.

If unavoidable, isolate the patch and document why it exists.

---

## Global State

Avoid unnecessary mutable globals.

Bad:

```js
let currentWeapon = null;
const loadedPackages = {};
```

when state can be scoped to a module, component, request, or object.

Do not replace every global with a singleton class.

Choose the narrowest reasonable ownership.

---

## Caching

Do not add caching without a reason.

Caching introduces:

- Invalidations.
- Stale data.
- Memory growth.
- More states.
- Harder debugging.

When caching is required, define:

- What is cached.
- For how long.
- How it is invalidated.
- Whether size must be bounded.

Do not memoize everything automatically.

---

## Performance

Prefer clear code until performance matters.

Do not prematurely:

- Memoize functions.
- Pool small objects.
- Replace arrays with complicated structures.
- Introduce workers.
- Parallelize trivial operations.
- Cache every API response.
- Rewrite readable code into micro-optimized loops.

Measure actual bottlenecks first.

---

## Dates and Time

Use `Date` where it is sufficient.

Be explicit about time zones and serialization.

Prefer ISO strings for transport:

```js
new Date().toISOString();
```

Do not manually implement timezone calculations.

Use platform APIs or established libraries when calendar/timezone behavior is complex.

---

## Regular Expressions

Use regex when it expresses the problem clearly.

Example:

```js
const assetPathPattern =
  /^\/Game\/[A-Za-z0-9_/]+$/;
```

Do not write enormous unreadable regexes when explicit parsing would be clearer.

Name non-trivial expressions.

Comment unusual constraints, not obvious regex syntax.

---

## JSON

Remember that JSON parsing can fail.

At external boundaries:

```js
let data;

try {
  data = JSON.parse(text);
} catch (error) {
  throw new Error(
    "Invalid weapon manifest JSON",
    { cause: error },
  );
}
```

Do not wrap every internal `JSON.parse()` in a custom error if the caller can already handle the native exception appropriately.

---

## Serialization

Do not serialize internal implementation state accidentally.

Prefer explicit object shapes for public APIs.

Avoid:

```js
JSON.stringify(instance)
```

when that leaks fields callers should not depend on.

Use an explicit representation if the boundary matters.

---

## Frontend Frameworks

Follow the framework's established patterns.

For React, Vue, Svelte, Angular, Solid, or other frameworks:

- Use framework-native state and lifecycle APIs.
- Avoid custom wrappers around simple framework features.
- Keep components focused.
- Avoid introducing a second state architecture casually.
- Follow the repository's component conventions.

Do not fight the framework to make architecture resemble another ecosystem.

---

## React

When using React:

- Prefer function components unless the project uses classes.
- Keep state local where practical.
- Do not put every value in state.
- Avoid unnecessary effects.
- Avoid automatic memoization.
- Do not introduce global state for a few local values.
- Keep components focused without splitting everything into tiny pieces.

---

## Derived State

Do not store values that can be calculated from existing state.

Bad:

```js
const [firstName, setFirstName] =
  useState("");

const [lastName, setLastName] =
  useState("");

const [fullName, setFullName] =
  useState("");

useEffect(() => {
  setFullName(
    `${firstName} ${lastName}`,
  );
}, [firstName, lastName]);
```

Prefer:

```js
const fullName =
  `${firstName} ${lastName}`;
```

Avoid synchronized duplicate state.

---

## React Effects

Use effects for synchronizing with external systems.

Do not use `useEffect` for ordinary calculations that can happen during render.

Avoid effects that merely copy one piece of state into another.

---

## React Memoization

Do not add `useMemo` or `useCallback` automatically.

Bad:

```js
const processedWeapons =
  useMemo(
    () => weapons.map(processWeapon),
    [weapons],
  );
```

unless memoization is actually useful.

Prefer:

```js
const processedWeapons =
  weapons.map(processWeapon);
```

until performance or referential stability requirements justify memoization.

---

## React Components

Do not extract a component merely because a JSX block is ten lines long.

Extract when:

- It represents a meaningful UI concept.
- It is reused.
- It owns behavior.
- It substantially clarifies the parent.

Avoid component fragmentation.

---

## Vanilla JavaScript UI

For small sites, direct DOM code can be perfectly professional.

Do not introduce React, Vue, or another framework just because the UI has state.

Likewise, do not build a custom mini-framework when the project already uses one.

Use the simplest appropriate architecture.

---

## CSS and Styling

Follow the project's styling system.

If it uses Tailwind:

- Use existing design tokens.
- Avoid arbitrary values unless necessary.
- Do not extract every class string into constants.
- Do not create wrapper components solely to reuse class lists.

If it uses CSS modules, vanilla CSS, Sass, styled-components, or another system, follow that convention.

Do not introduce a second styling system casually.

---

## APIs

Keep API code direct.

Do not create five layers around:

```js
fetch("/api/weapons")
```

unless those layers provide actual behavior such as:

- Authentication.
- Error normalization.
- Caching.
- Retry policy.
- Request tracing.
- Shared base URL handling.

A wrapper should earn its existence.

---

## Database Code

Use the database library or ORM directly when its API is already appropriate.

Do not automatically add:

```text
Repository
Service
Manager
DataProvider
```

on top of straightforward database calls.

A query can remain a query.

---

## Transactions

Use transactions when related writes must succeed or fail together.

Do not create transaction wrappers for isolated read operations.

Follow the database client's established patterns.

---

## Logging

Use the project's logging mechanism.

Log meaningful events and failures.

Avoid:

```js
logger.info("Starting process");
logger.info("Processing data");
logger.info("Process complete");
```

Better:

```js
logger.warn(
  { assetPath },
  "Weapon asset could not be resolved",
);
```

Do not log the same error at every layer.

---

## Console Usage

`console.log()` is fine for:

- Small scripts.
- Development tooling.
- CLI output when appropriate.

Do not leave debugging logs in production application code.

Avoid:

```js
console.log("here");
console.log("test");
console.log(data);
```

in finished work.

Use the existing logger where one exists.

---

## Tests

Use the repository's existing testing framework.

Possible tools include:

```text
Vitest
Jest
Mocha
Node test runner
Playwright
Cypress
```

Do not replace the test stack unless explicitly requested.

Tests should verify behavior, not implementation details.

Good:

```js
it(
  "returns undefined when the weapon does not exist",
  () => {
    // ...
  },
);
```

Avoid:

```js
it("tests weapon parser", () => {
  // ...
});
```

---

## Test Real Behavior

Prefer testing actual outputs and side effects.

Do not mock every dependency.

Use real lightweight objects where practical.

Mock:

- Network requests.
- Databases.
- External services.
- Time when needed.
- Expensive or nondeterministic systems.

Do not mock:

- Plain objects.
- Simple helper functions.
- Basic arrays or maps.
- Internal implementation details without need.

---

## Avoid Over-Mocking

Bad:

```js
expect(parser.parse)
  .toHaveBeenCalledTimes(1);

expect(mapper.map)
  .toHaveBeenCalledTimes(1);
```

when the actual resulting weapon can be asserted directly.

Test interactions when the interaction itself is the behavior.

---

## Test Helpers

Do not build a testing framework for a handful of tests.

Avoid unnecessary:

```text
FixtureFactory
MockManager
TestDataBuilder
TestingUtilityService
```

Inline simple setup.

Extract helpers when real duplication appears.

---

## Browser Tests

Use browser automation for actual browser behavior:

- Navigation.
- Focus.
- Keyboard input.
- Responsive layout.
- History.
- Real form behavior.
- Accessibility interactions.

Do not use end-to-end tests for every tiny pure function.

Keep the test level appropriate to the behavior.

---

## ESLint

Respect the project's ESLint configuration.

Do not disable rules merely to make generated code pass.

Avoid:

```js
// eslint-disable-next-line
```

unless:

- The rule genuinely does not apply.
- The reason is understood.
- A cleaner implementation is not practical.

Do not add file-wide disables casually.

---

## Prettier

Let Prettier format code when the project uses it.

Do not manually align values.

Avoid:

```js
const short      = 1;
const longerName = 2;
```

Do not add `prettier-ignore` without a concrete reason.

---

## Lint Fixes

Do not blindly run automatic fixes across unrelated files.

Keep changes scoped to the task.

Avoid producing large formatting diffs for a one-line bug fix.

---

## Package Management

Use the project's existing package manager.

Possible choices include:

```text
npm
pnpm
Yarn
Bun
```

Do not switch package managers as part of unrelated work.

Do not manually edit generated lockfiles.

Use the package manager.

---

## Adding Dependencies

Before adding a package, ask:

- Does JavaScript or the platform already provide this?
- Does the project already depend on something equivalent?
- Is the package maintained?
- Is the dependency worth its runtime or bundle cost?
- Is the implementation simple enough to write directly?

Do not install a dependency to replace five straightforward lines.

Do not reimplement security-sensitive or standards-heavy functionality merely to avoid dependencies.

---

## Avoid Dependency Duplication

Do not add:

```text
axios
```

to a project already using:

```text
fetch
```

unless axios provides functionality the project actually needs.

Do not add:

```text
lodash
```

for one operation available as:

```js
Object.groupBy()
Array.prototype.flatMap()
structuredClone()
```

when runtime support exists.

Conversely, do not force native APIs when the existing dependency is established and consistent.

---

## Avoid Premature Abstraction

Do not design for hypothetical future requirements.

Do not add:

- Factories for one implementation.
- Strategy patterns for one algorithm.
- Base classes for one subclass.
- Registries for fixed behavior.
- Generic repositories around straightforward APIs.
- Service containers.
- Complex DI systems.
- Custom event buses for local state.
- Builders for simple object construction.
- Adapter layers that do not adapt anything.
- Extension hooks with no consumers.

Build what the current task actually needs.

---

## Avoid AI-Looking Architecture

Do not automatically create structures named:

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

Use them only when they represent real architectural roles.

Do not create symmetrical layers merely because generated code tends to look organized that way.

---

## Avoid Excessive Wrapper Functions

Do not write:

```js
function fetchWeaponData(id) {
  return fetchWeapon(id);
}
```

unless the wrapper adds behavior or creates a meaningful boundary.

Every abstraction should earn its existence.

---

## Avoid Fake Flexibility

Do not create configuration or extension points for imaginary requirements.

Avoid:

```js
function processWeapon(
  weapon,
  {
    enableValidation = true,
    enableNormalization = true,
    enableTransformation = true,
  } = {},
) {
  // ...
}
```

when those operations are always required.

Do not add switches merely to make a function "flexible."

---

## Avoid Boolean Parameter Piles

Bad:

```js
processWeapon(
  weapon,
  true,
  false,
  true,
);
```

Prefer:

```js
processWeapon(weapon, {
  validate: true,
  normalize: false,
  includeMetadata: true,
});
```

when multiple independent behaviors genuinely need configuration.

Do not create an options object for a simple one-argument operation.

---

## Avoid Placeholder Architecture

Do not leave speculative TODOs such as:

```js
// TODO: Add caching
// TODO: Add advanced validation
// TODO: Add retry support
// TODO: Add telemetry
```

unless the user explicitly requested scaffolding.

Implement the requested feature fully.

Do not leave commentary about hypothetical future work in production code.

---

## Avoid Decorative Section Comments

Do not add:

```js
// ========================================
// INITIALIZATION
// ========================================
```

unless the codebase already uses that style.

Good organization should normally make those unnecessary.

---

## Avoid Clever One-Liners

Do not compress complex behavior purely to reduce line count.

Bad:

```js
return items?.find((x) => valid(x))
  ?.value ?? fallback();
```

if the operation involves meaningful branching or debugging concerns.

Use explicit statements when they improve readability.

---

## Avoid Overusing Chaining

Bad:

```js
const result = data
  .filter(...)
  .map(...)
  .flatMap(...)
  .filter(...)
  .reduce(...)
  .map(...);
```

when the chain requires substantial mental reconstruction.

Break complex transformations into named stages or use a loop.

---

## Avoid Mutating Function Arguments Unexpectedly

Do not mutate caller-owned objects unless the API clearly expects mutation.

Surprising:

```js
function normalizeWeapon(weapon) {
  weapon.name =
    weapon.name.trim();

  return weapon;
}
```

Safer when mutation is not expected:

```js
function normalizeWeapon(weapon) {
  return {
    ...weapon,
    name: weapon.name.trim(),
  };
}
```

However, do not copy objects mechanically when in-place mutation is part of the surrounding design.

Make the behavior clear.

---

## Immutability

Use immutable updates where they fit the framework or data flow.

Do not treat mutation as inherently bad.

This is perfectly reasonable:

```js
const results = [];

for (const asset of assets) {
  const weapon = parseWeapon(asset);

  if (weapon) {
    results.push(weapon);
  }
}
```

Readable local mutation is often better than convoluted functional expressions.

---

## `const` vs `let`

Prefer `const` when the variable binding does not change.

Use `let` when reassignment is required.

Avoid `var` in modern code unless maintaining legacy style that depends on it.

Do not convert all existing `var` declarations across unrelated files as part of a small task.

---

## Avoid Reassignment Without Need

Bad:

```js
let weapon = loadWeapon();

weapon = normalizeWeapon(weapon);

weapon = enrichWeapon(weapon);
```

This may be fine, but when intermediate meanings differ, clearer names can help:

```js
const weapon =
  loadWeapon();

const normalizedWeapon =
  normalizeWeapon(weapon);

const enrichedWeapon =
  enrichWeapon(normalizedWeapon);
```

Do not overdo intermediate names for trivial pipelines.

---

## Public APIs

Be deliberate with exported APIs.

For public functions:

- Keep parameters clear.
- Avoid leaking implementation details.
- Preserve compatibility where practical.
- Document non-obvious behavior.
- Do not add options callers do not need.
- Keep error behavior predictable.

Internal helpers can remain simpler.

---

## Function Parameters

Avoid large positional argument lists.

Bad:

```js
createWeapon(
  id,
  name,
  damage,
  fireRate,
  magazineSize,
  category,
  description,
);
```

Prefer:

```js
createWeapon({
  id,
  name,
  damage,
  fireRate,
  magazineSize,
  category,
  description,
});
```

when the values naturally belong together.

Do not wrap every two-argument call in an object automatically.

---

## Default Parameters

Use default parameters for simple defaults:

```js
function loadWeapon(
  id,
  retries = 3,
) {
  // ...
}
```

Be careful with defaults that allocate objects:

```js
function loadWeapon(
  id,
  options = {},
) {
  // ...
}
```

This is safe in JavaScript because the expression is evaluated per call.

Do not introduce complicated default expressions with side effects.

---

## Rest Parameters

Use rest parameters when the API genuinely accepts variable arguments.

Good:

```js
function logWeapons(...weapons) {
  // ...
}
```

Do not use:

```js
function process(...args) {
  // ...
}
```

when the function has a known stable signature.

Explicit parameters are easier to maintain.

---

## Spread Arguments

Use spread when it clearly fits the operation.

Avoid using spread solely for cleverness or when it can create large argument lists.

For very large arrays, avoid patterns such as:

```js
Math.max(...largeArray);
```

if argument limits could become an issue.

---

## Dynamic Properties

Use bracket access when keys are actually dynamic:

```js
record[fieldName]
```

Use dot notation when the property is fixed:

```js
record.name
```

Do not write:

```js
record["name"]
```

without a reason.

---

## `Object.hasOwn`

Use modern own-property checks where supported:

```js
Object.hasOwn(
  object,
  key,
);
```

Avoid unsafe direct calls like:

```js
object.hasOwnProperty(key);
```

on arbitrary objects.

Respect runtime compatibility.

---

## Deep Cloning

Do not use JSON serialization as a generic deep-clone mechanism:

```js
JSON.parse(
  JSON.stringify(value),
);
```

It loses or changes values such as:

- Dates.
- Maps.
- Sets.
- `undefined`.
- Functions.
- BigInts.

Use:

```js
structuredClone(value)
```

when supported and appropriate.

Do not deep clone objects without a real need.

---

## Equality of Objects

Remember:

```js
{} === {}
```

is false.

Do not write ad hoc deep equality logic for complex structures.

Use the project's existing test or comparison utilities where appropriate.

---

## Browser Globals

Do not assume browser globals exist in Node or SSR environments.

Be careful with:

```js
window
document
localStorage
navigator
```

inside code that may execute server-side.

Use framework lifecycle boundaries where necessary.

---

## SSR

When working in server-rendered frameworks:

- Keep browser-only code out of server execution.
- Avoid accessing `window` at module initialization.
- Follow framework conventions for client-only behavior.

Do not solve SSR problems with scattered `typeof window !== "undefined"` checks if the framework provides a cleaner boundary.

---

## Accessibility

Do not recreate native controls with generic `<div>` elements unless necessary.

Prefer semantic HTML:

```html
<button>
<input>
<label>
<nav>
main
```

over JavaScript-driven imitations.

Use native browser behavior where possible.

---

## Forms

Prefer standard form behavior.

Use submit events rather than attaching click handlers only to submit buttons.

Good:

```js
form.addEventListener(
  "submit",
  handleSubmit,
);
```

This preserves keyboard and accessibility behavior.

---

## State

Keep state as small and local as practical.

Avoid synchronized copies of the same information.

Derive values when possible.

Do not introduce a global store because multiple functions need access to one small value.

---

## Event Buses

Do not introduce custom event buses for ordinary local communication.

Use:

- Direct function calls.
- Framework state.
- DOM events.
- Existing project messaging systems.

A global event bus can obscure control flow.

Use one only when decoupled event distribution is genuinely required.

---

## Pub/Sub

When pub/sub is appropriate, define:

- Event ownership.
- Subscription lifetime.
- Cleanup behavior.
- Payload shape.

Do not create a generic event system for two callbacks.

---

## Web Workers

Use workers for genuinely heavy CPU work that would block the UI.

Do not move trivial transformations into a worker merely to improve perceived architecture.

Account for messaging and serialization overhead.

---

## Service Workers

Use service workers for real offline, caching, or network-interception requirements.

Do not add one casually.

They add lifecycle and caching complexity that can create difficult bugs.

---

## Do Not Rewrite Working Code Unnecessarily

When modifying an existing feature:

1. Identify the smallest responsible area.
2. Understand current behavior.
3. Preserve unrelated behavior.
4. Match the existing architecture.
5. Change only what the task requires.
6. Add or update relevant tests.
7. Avoid unrelated cleanup.

Do not rewrite working code merely because you would have structured it differently.

---

## Refactoring

When explicitly asked to refactor:

- Preserve observable behavior unless a change is requested.
- Keep public APIs stable where practical.
- Separate structural changes from behavioral changes.
- Avoid simultaneous style migrations.
- Run existing tests before and after when possible.
- Remove real duplication, not merely visually similar code.

Different is not automatically better.

---

## Avoid Premature DRY

Do not extract two similar blocks just because they look alike.

Duplication can be cheaper than the wrong abstraction.

Extract common logic when:

- The behavior is genuinely the same.
- Changes should logically occur together.
- The abstraction has a clear name.

Do not force unrelated concepts through a generic helper solely to eliminate repeated lines.

---

## Avoid Fake Robustness

Do not add checks for impossible internal states simply to look defensive.

Bad:

```js
function processWeapon(weapon) {
  if (!weapon) {
    throw new Error(
      "Weapon cannot be null",
    );
  }

  // ...
}
```

when the function is only ever called after validated selection.

Validate at real system boundaries.

Keep internal code direct.

---

## Avoid Fake Extensibility

Do not add:

```js
const strategies = {
  default: processWeapon,
};
```

for one behavior.

Add extension mechanisms when an actual second implementation or external consumer requires them.

---

## Avoid AI-Generated Prose

Comments, errors, and documentation should sound like normal engineering text.

Avoid:

```text
This function is responsible for...
This method ensures...
The following logic...
In order to...
It is important to note...
This provides a robust and scalable solution...
```

Write concise technical text.

---

## Modern JavaScript

Use modern language features when they improve clarity and are supported by the runtime.

Useful features may include:

- `const` and `let`.
- Optional chaining.
- Nullish coalescing.
- Object and array destructuring.
- Template literals.
- Async/await.
- `Promise.all`.
- `Map`.
- `Set`.
- Private fields.
- Logical assignment.
- `Object.fromEntries`.
- `Object.entries`.
- `Array.from`.
- `structuredClone`.
- Top-level `await` where supported.

Do not use features merely to make the code look sophisticated.

---

## Before Finishing

Review the change and remove:

- Redundant comments.
- Tutorial-style prose.
- Unnecessary classes.
- Unnecessary factories.
- Unnecessary wrappers.
- Generic AI-style names.
- Pass-through service layers.
- Utility classes.
- Duplicate validation.
- Defensive checks for already-trusted data.
- Excessive logging.
- Debug `console.log()` calls.
- Nested ternaries.
- Overly clever array chains.
- Unnecessary object copying.
- Placeholder TODOs.
- Speculative configuration.
- Speculative extensibility.
- Dead code.
- Unused imports.
- Unrelated refactors.

Then run the project's existing checks where available.

Typical JavaScript checks may include:

```bash
npm run lint
npm test
npm run build
```

or tools such as:

```bash
eslint .
prettier --check .
node --test
```

Do not assume these exact commands exist.

Inspect `package.json` and repository configuration first.

The final code should look like it naturally belongs in the repository rather than like a standalone AI-generated solution.

It should feel like JavaScript written by an experienced maintainer: direct, readable, pragmatic, appropriately defensive at real boundaries, and free of unnecessary architectural ceremony.