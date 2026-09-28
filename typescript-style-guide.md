# TypeScript Style Guide

Write TypeScript as an experienced professional developer maintaining a real production codebase.

The goal is clean, idiomatic, maintainable code—not code that looks generated, over-engineered, excessively defensive, or written as a tutorial.

The most important rule:

> Do not optimize for demonstrating programming practices. Optimize for producing the smallest idiomatic production-quality change that an experienced maintainer would reasonably write.

## General Principles

Prefer:

- Simple solutions over clever abstractions.
- Existing project conventions over personal preferences.
- Idiomatic modern TypeScript.
- Small, focused changes.
- Strong types without unnecessary type gymnastics.
- Clear code over explanatory comments.
- Direct implementation over speculative extensibility.
- Domain terminology over generic abstraction names.
- Compiler-enforced correctness over runtime checks for internal invariants.

Do not refactor unrelated code unless required by the task.

Do not introduce new patterns, classes, modules, interfaces, factories, wrappers, or abstraction layers unless they solve an actual problem.

Do not make a simple JavaScript problem complicated merely because TypeScript supports advanced types.

---

## Match the Existing Codebase

Before writing code, inspect nearby files and follow the repository's established conventions for:

- Naming
- Module structure
- File organization
- Import style
- Export style
- Semicolon usage
- Quote style
- Type vs interface usage
- Error handling
- Logging
- Async patterns
- Dependency injection
- Testing
- Validation
- State management
- Framework conventions
- ESLint configuration
- Prettier configuration
- `tsconfig.json`

Consistency with the repository is more important than imposing a preferred style.

Do not perform style migrations as part of unrelated work.

If the project uses:

```ts
interface User
```

do not convert everything to:

```ts
type User =
```

without a reason.

Likewise, do not change default exports to named exports, or vice versa, merely because you prefer one style.

---

## Naming

Use normal TypeScript and JavaScript naming conventions:

- `camelCase` for variables, functions, parameters, and object properties.
- `PascalCase` for classes, interfaces, types, enums, and components.
- `UPPER_SNAKE_CASE` only for genuine constants where that convention is appropriate.
- Boolean names should usually describe a state or condition.
- Event handlers should follow the framework or repository convention.

Good:

```ts
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

```ts
dataManager
processHandler
utilityHelper
genericService
resultProcessor
enhancedProcessor
operationManager
dataUtility
```

Avoid meaningless generic names where domain terminology is available.

Bad:

```ts
const data = getData();
const result = processData(data);
```

Better:

```ts
const weapon = getWeaponDefinition();
const stats = parseWeaponStats(weapon);
```

Do not make names unnecessarily verbose.

Avoid:

```ts
const successfullyParsedWeaponConfigurationResult =
  parseWeaponConfiguration();
```

when:

```ts
const weaponConfig = parseWeaponConfiguration();
```

is obvious.

---

## Functions

Keep functions focused.

Prefer direct readable code:

```ts
const pkg = await provider.loadPackage(path);
return mapper.map(pkg);
```

over unnecessary wrappers:

```ts
const pkg = await loadPackageFromProvider(provider, path);
const mappedPackage = mapLoadedPackageUsingMapper(mapper, pkg);

return mappedPackage;
```

Do not create helper functions just to reduce a readable function by three or four lines.

Extract functions when they:

- Represent a meaningful domain operation.
- Are reused.
- Remove genuinely complex logic.
- Improve testability.
- Provide a useful abstraction boundary.

Avoid generic names such as:

```ts
processData()
handleStuff()
executeLogic()
performOperation()
doWork()
```

Use names that describe what the function actually does.

---

## Prefer Functions Unless a Class Is Justified

Do not automatically create classes.

Prefer simple functions and modules when stateful objects are unnecessary.

Good:

```ts
export function parseWeaponStats(data: WeaponData): WeaponStats {
  return {
    damage: data.damage,
    fireRate: data.fireRate,
  };
}
```

Do not turn this into:

```ts
export class WeaponStatsParser {
  parse(data: WeaponData): WeaponStats {
    // ...
  }
}
```

unless the parser genuinely requires state, dependencies, multiple implementations, lifecycle management, or some other architectural reason.

JavaScript and TypeScript are naturally module-oriented.

Use that strength.

---

## Types

Use types to describe actual data and behavior.

Do not create types merely to make the code appear strongly architected.

Good:

```ts
interface WeaponStats {
  damage: number;
  fireRate: number;
  magazineSize: number;
}
```

Avoid unnecessary layering:

```ts
interface BaseWeaponStatsData {
  damage: number;
}

interface ExtendedWeaponStatsData extends BaseWeaponStatsData {
  fireRate: number;
}

interface CompleteWeaponStatsData extends ExtendedWeaponStatsData {
  magazineSize: number;
}
```

when a single type is sufficient.

---

## `type` vs `interface`

Follow the existing project convention first.

As a general guideline:

Use `interface` for object contracts that may reasonably be extended:

```ts
interface Weapon {
  id: string;
  name: string;
}
```

Use `type` for unions, aliases, tuples, mapped types, or compositions:

```ts
type WeaponCategory =
  | "rifle"
  | "smg"
  | "shotgun"
  | "sniper";

type WeaponId = string;

type Point = [x: number, y: number];
```

Do not waste time converting between `interface` and `type` when either representation is perfectly reasonable.

---

## Avoid Type Gymnastics

Advanced TypeScript is useful when it solves an actual type problem.

Do not introduce complicated conditional types, mapped types, recursive types, or generic hierarchies simply because they are possible.

Avoid:

```ts
type DeeplyMappedConditionalEntityTransformer<
  T extends Record<string, unknown>,
  K extends keyof T = keyof T,
> = {
  [P in K]: T[P] extends object
    ? DeeplyMappedConditionalEntityTransformer<T[P]>
    : T[P];
};
```

when a straightforward explicit type would be easier to understand and maintain.

Types should reduce complexity, not move complexity into the type system.

---

## Avoid Excessive Generics

Use generics when they preserve useful type information across genuinely reusable code.

Good:

```ts
function first<T>(items: readonly T[]): T | undefined {
  return items[0];
}
```

Avoid adding generic parameters to functions used for exactly one concrete type.

Bad:

```ts
function parseEntity<TInput, TOutput>(
  input: TInput,
): TOutput {
  // ...
}
```

when the function specifically parses a weapon.

Prefer:

```ts
function parseWeapon(input: WeaponData): Weapon {
  // ...
}
```

Domain-specific code does not need to pretend to be a framework.

---

## Avoid `any`

Do not use `any` as an escape hatch unless interacting with an API where no reasonable type information exists.

Prefer:

```ts
unknown
```

for values whose type has not yet been established.

Bad:

```ts
function parse(data: any) {
  return data.weapon.name;
}
```

Better:

```ts
function parse(data: unknown): Weapon {
  const weapon = weaponSchema.parse(data);
  return weapon;
}
```

or, when runtime validation is unnecessary:

```ts
function parse(data: WeaponData): Weapon {
  return {
    id: data.id,
    name: data.name,
  };
}
```

Do not replace every `any` with elaborate generic machinery simply to eliminate the lint warning.

Fix the actual boundary.

---

## `unknown`

Use `unknown` for untrusted or externally supplied values.

Examples:

- Parsed JSON.
- API responses.
- Plugin inputs.
- MCP inputs.
- `catch` values.
- External messages.
- Configuration loaded from arbitrary sources.

Narrow the value before use.

```ts
function isWeapon(value: unknown): value is Weapon {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value
  );
}
```

Do not perform repeated runtime type checking once data has already crossed a validated boundary.

---

## Type Assertions

Avoid using `as` merely to silence the compiler.

Bad:

```ts
const weapon = data as Weapon;
```

when `data` has not actually been verified.

A type assertion does not perform validation.

Use assertions when the invariant is genuinely known but cannot reasonably be inferred by TypeScript.

Prefer improving the type flow when possible.

Avoid chains like:

```ts
value as unknown as Weapon
```

unless interfacing with unavoidable third-party typing limitations and the reason is documented.

---

## Non-Null Assertions

Avoid excessive use of `!`.

Bad:

```ts
const name = weapon!.metadata!.displayName!;
```

Prefer handling the actual possibility:

```ts
if (!weapon?.metadata?.displayName) {
  return null;
}

return weapon.metadata.displayName;
```

Do not add unnecessary null checks when the type contract guarantees the value exists.

Fix incorrect types instead of defending against impossible states everywhere.

---

## Strict TypeScript

Prefer projects with strict TypeScript settings.

Typical useful options include:

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

However, do not enable stricter compiler options in an established project as an unrelated change.

Respect the repository's current configuration.

Do not scatter type assertions across the code merely to satisfy strict mode.

Address the underlying type uncertainty.

---

## Null and Undefined

Follow the project's convention.

Avoid mixing `null` and `undefined` arbitrarily.

Prefer one meaning consistently.

For example:

```ts
function findWeapon(id: string): Weapon | undefined
```

is usually natural for lookup functions.

Use `null` when it has a meaningful domain or API meaning.

Do not return:

```ts
undefined | null | false | ""
```

from the same function to represent failure.

Make the contract clear.

---

## Optional Chaining

Use optional chaining when missing values are expected and no action is required.

Good:

```ts
const displayName = weapon.metadata?.displayName;
```

Do not hide required invariants behind long optional chains.

Bad:

```ts
const damage =
  package?.data?.weapon?.stats?.base?.damage;
```

if all of those values should exist after parsing.

Validate the structure once and use a stronger type afterward.

---

## Nullish Coalescing

Use `??` when defaults should only apply to `null` or `undefined`.

```ts
const retries = config.retries ?? 3;
```

Do not use `||` if valid falsy values such as `0`, `false`, or `""` must be preserved.

Bad:

```ts
const retries = config.retries || 3;
```

if `0` is valid.

---

## Discriminated Unions

Use discriminated unions when they make state or variants explicit.

Good:

```ts
type LoadResult =
  | {
      status: "success";
      weapon: Weapon;
    }
  | {
      status: "not-found";
      path: string;
    }
  | {
      status: "error";
      error: Error;
    };
```

This is often clearer than several optional properties:

```ts
interface LoadResult {
  success: boolean;
  weapon?: Weapon;
  path?: string;
  error?: Error;
}
```

Do not use discriminated unions for every tiny operation.

Use them when multiple meaningful states genuinely exist.

---

## Enums

Follow the project's convention.

Prefer string unions for small closed sets when no runtime enum object is required:

```ts
type WeaponCategory =
  | "rifle"
  | "smg"
  | "shotgun"
  | "sniper";
```

Use enums when runtime semantics, interoperability, or existing architecture justify them.

Do not convert an established enum-based codebase to string unions as part of unrelated work.

---

## Constants

Give meaningful names to values whose purpose is not obvious.

Good:

```ts
const MAX_RETRY_ATTEMPTS = 3;
```

Do not create constants for trivial literals merely to eliminate every "magic number."

Bad:

```ts
const ZERO = 0;
const ONE = 1;
```

A value deserves a name when the name communicates domain meaning.

---

## Object Shapes

Prefer simple object literals for data.

Good:

```ts
const weapon: Weapon = {
  id,
  name,
  damage,
};
```

Do not build unnecessary builder patterns:

```ts
new WeaponBuilder()
  .withId(id)
  .withName(name)
  .withDamage(damage)
  .build();
```

unless the construction process is genuinely complex.

---

## Destructuring

Use destructuring when it improves readability.

Good:

```ts
const { id, name, stats } = weapon;
```

Avoid destructuring large objects when it removes useful context.

Sometimes:

```ts
weapon.name
weapon.stats.damage
```

is clearer than introducing many local variables.

Do not destructure automatically.

---

## Object Spread

Use object spread for straightforward immutable updates:

```ts
const updatedWeapon = {
  ...weapon,
  damage: newDamage,
};
```

Do not build long spread chains that obscure which values win.

Avoid:

```ts
const result = {
  ...defaults,
  ...config,
  ...environment,
  ...overrides,
  ...runtime,
};
```

unless the precedence is intentional and clear.

Explicit assignment may be easier to maintain.

---

## Arrays

Use appropriate array methods when they make intent clear.

Good:

```ts
const weapons = assets
  .filter((asset) => asset.type === "weapon")
  .map(parseWeapon);
```

A loop is often better when:

- Multiple operations happen per item.
- Early exit matters.
- Error handling differs per item.
- Mutation is intentional.
- The chain becomes difficult to read.

Do not force everything into:

```ts
.filter()
.map()
.reduce()
.flatMap()
```

just to appear functional.

---

## Avoid Clever `reduce`

Do not use `reduce` when a loop or dedicated helper communicates the operation more clearly.

Bad:

```ts
const weaponsById = weapons.reduce(
  (acc, weapon) => ({
    ...acc,
    [weapon.id]: weapon,
  }),
  {},
);
```

Better:

```ts
const weaponsById = new Map<string, Weapon>();

for (const weapon of weapons) {
  weaponsById.set(weapon.id, weapon);
}
```

or:

```ts
const weaponsById = Object.fromEntries(
  weapons.map((weapon) => [weapon.id, weapon]),
);
```

Choose whichever is clearest for the actual use case.

---

## Maps and Sets

Use `Map` and `Set` when their semantics are useful.

Examples:

```ts
const weaponsById = new Map<string, Weapon>();
const processedPaths = new Set<string>();
```

Do not use plain objects as maps automatically.

Likewise, do not replace simple objects with `Map` when serialization or direct property access is more natural.

---

## Control Flow

Prefer guard clauses over deep nesting.

Good:

```ts
if (!asset) {
  return null;
}

if (!asset.loaded) {
  await asset.load();
}

return asset;
```

Avoid:

```ts
if (asset) {
  if (!asset.loaded) {
    await asset.load();
  }

  return asset;
}

return null;
```

Avoid unnecessary `else` blocks after:

```ts
return
throw
continue
break
```

Keep the happy path easy to follow.

---

## Ternaries

Use ternaries for simple value selection.

Good:

```ts
const label = enabled ? "Enabled" : "Disabled";
```

Do not nest ternaries for complex branching.

Bad:

```ts
const label = active
  ? selected
    ? loading
      ? "Loading"
      : "Selected"
    : "Active"
  : "Inactive";
```

Use normal control flow instead.

---

## Boolean Logic

Write conditions so the intent is obvious.

Prefer:

```ts
if (!weapon) {
  return;
}
```

over:

```ts
if (weapon === undefined || weapon === null) {
  return;
}
```

when both forms mean the same thing in context.

Do not use clever coercion solely for brevity.

Avoid:

```ts
const isValid = !!value;
```

when a more meaningful check is appropriate.

---

## Comments

Do not narrate the code.

Bad:

```ts
// Check if the weapon exists
if (!weapon) {
  // Return null if the weapon doesn't exist
  return null;
}
```

Good:

```ts
if (!weapon) {
  return null;
}
```

Write comments only when they explain something the code itself cannot make obvious, such as:

- A browser limitation.
- A protocol requirement.
- A third-party library bug.
- A compatibility workaround.
- A performance tradeoff.
- A non-obvious domain constraint.
- Why seemingly unnecessary behavior exists.

Prefer explaining **why**, not **what**.

Avoid comments like:

```ts
// Initialize array
// Loop over items
// Check condition
// Return result
// Handle error
// Create object
```

Do not add JSDoc to every function automatically.

---

## JSDoc

Use JSDoc when it adds information that TypeScript cannot express clearly.

Useful examples include:

- Public library APIs.
- Non-obvious behavioral contracts.
- Important side effects.
- Units or formats.
- External protocol requirements.
- Deprecation notices.

Avoid:

```ts
/**
 * Gets the weapon name.
 *
 * @param weapon The weapon.
 * @returns The weapon name.
 */
function getWeaponName(weapon: Weapon): string {
  return weapon.name;
}
```

The function already explains itself.

---

## Error Handling

Do not wrap every async call in `try/catch`.

Catch errors only when you can:

- Recover.
- Add useful context.
- Translate the error into a domain-specific failure.
- Perform necessary cleanup.
- Return an expected result type.

Avoid:

```ts
try {
  return await loadPackage(path);
} catch (error) {
  console.error(error);
  throw error;
}
```

This adds no value.

Prefer letting the error propagate.

---

## Catch Variables

Treat caught errors as unknown unless the codebase or compiler configuration guarantees otherwise.

```ts
try {
  await loadPackage(path);
} catch (error) {
  if (error instanceof Error) {
    logger.error(error.message);
  }
}
```

Do not assume every thrown value is an `Error`.

External JavaScript code can technically throw anything.

---

## Preserve Error Causes

When wrapping errors, preserve the cause when supported.

```ts
throw new Error(`Failed to load package: ${path}`, {
  cause: error,
});
```

Do not throw a new unrelated error that destroys the original failure context.

---

## Error Messages

Write actionable error messages.

Good:

```text
Unable to load package "/Game/Weapons/Rifle_A": package was not found.
```

Bad:

```text
Something went wrong.
```

Include useful identifiers when appropriate:

- Paths
- IDs
- Asset names
- URLs
- Operation names

Do not include secrets or sensitive values in logs or errors.

---

## Expected Failures

Do not use exceptions for routine states when a clearer return contract exists.

For example:

```ts
function findWeapon(id: string): Weapon | undefined
```

is preferable to throwing when "not found" is normal.

Use exceptions when the operation genuinely failed.

---

## Async Code

Use `async` only for asynchronous work.

Do not write:

```ts
async function getCount(): Promise<number> {
  return items.length;
}
```

Prefer:

```ts
function getCount(): number {
  return items.length;
}
```

Avoid unnecessary `await`:

```ts
return loadWeapon();
```

instead of:

```ts
return await loadWeapon();
```

unless `await` is needed for local error handling, stack behavior, or cleanup.

---

## Parallel Async Work

Run independent asynchronous operations concurrently when appropriate.

Good:

```ts
const [weapons, attachments] = await Promise.all([
  loadWeapons(),
  loadAttachments(),
]);
```

Do not serialize independent I/O unnecessarily:

```ts
const weapons = await loadWeapons();
const attachments = await loadAttachments();
```

However, do not use `Promise.all` when:

- Operations depend on one another.
- Concurrency would overwhelm an external service.
- Ordered side effects matter.
- Error behavior needs individual handling.

---

## Avoid Async `forEach`

Do not write:

```ts
items.forEach(async (item) => {
  await processItem(item);
});
```

The returned promises are ignored.

Use:

```ts
for (const item of items) {
  await processItem(item);
}
```

for sequential work.

Or:

```ts
await Promise.all(items.map(processItem));
```

for appropriate concurrent work.

---

## Cancellation

Use `AbortSignal` when the surrounding environment supports cancellation.

Example:

```ts
async function loadPackage(
  path: string,
  signal?: AbortSignal,
): Promise<Package> {
  // ...
}
```

Propagate existing cancellation signals through network or long-running operations.

Do not invent cancellation infrastructure for trivial operations.

---

## Fetch

Check the HTTP status when required.

Bad:

```ts
const response = await fetch(url);
return response.json();
```

Better:

```ts
const response = await fetch(url);

if (!response.ok) {
  throw new Error(
    `Request failed with ${response.status} ${response.statusText}`,
  );
}

return response.json();
```

Validate external response data when correctness depends on it.

Do not blindly cast responses:

```ts
const data = (await response.json()) as Weapon[];
```

if the endpoint is untrusted or unstable.

---

## Runtime Validation

Validate data at trust boundaries.

Examples:

- API responses.
- User input.
- Environment variables.
- Configuration files.
- Database payloads when schemas are not guaranteed.
- Plugin or extension messages.
- MCP inputs.
- WebSocket messages.

Libraries such as Zod, Valibot, ArkType, or an existing project validator can be appropriate.

Do not add a validation library when the project already has one.

Do not repeatedly validate data that has already been validated and converted into trusted internal types.

---

## Environment Variables

Do not access environment variables throughout the application.

Prefer validating configuration once near startup.

For example:

```ts
interface Config {
  apiUrl: string;
  port: number;
}
```

Then pass or import validated configuration according to the project's architecture.

Avoid:

```ts
process.env.API_URL!
```

scattered throughout the codebase.

---

## Imports

Follow the repository's existing import conventions.

Group imports only if the formatter or lint configuration expects it.

Avoid unnecessary import churn.

Use `import type` where it improves emitted JavaScript or matches project conventions:

```ts
import type { Weapon } from "./types";
```

Do not mechanically convert every type import unless the project requires it.

---

## Import Paths

Prefer existing aliases if configured.

For example:

```ts
import { Weapon } from "@/domain/weapon";
```

instead of:

```ts
import { Weapon } from "../../../../domain/weapon";
```

when the project has an established alias.

Do not introduce a path alias merely to fix one ugly import.

---

## Exports

Prefer intentional exports.

Do not export everything "just in case."

Keep implementation details private to the module when they are not part of its API.

Avoid giant barrel files that re-export an entire project unless the architecture intentionally uses them.

Barrel files can introduce:

- Circular dependencies.
- Hidden coupling.
- Poor tree shaking.
- Difficult navigation.

Use them where they provide an actual package boundary.

---

## Default vs Named Exports

Follow the project convention.

Do not refactor between default and named exports without a practical reason.

For reusable library modules, named exports often make refactoring easier:

```ts
export function parseWeapon() {}
```

For framework files where default exports are idiomatic or required, use them.

---

## Modules and Files

Keep files cohesive.

Do not enforce arbitrary rules such as "one function per file."

A small set of closely related functions can belong together.

Do not create files like:

```text
weapon-helper.ts
weapon-utils.ts
weapon-manager.ts
weapon-service.ts
weapon-processor.ts
```

unless those files actually represent separate responsibilities.

Prefer domain-based organization.

---

## Avoid Generic `utils`

Do not dump unrelated functions into:

```text
utils.ts
helpers.ts
common.ts
misc.ts
```

If a helper has a clear domain, name the module after that domain.

Better:

```text
weapon-paths.ts
asset-format.ts
package-validation.ts
```

Small local helpers do not need their own module.

---

## Classes

Use classes when object identity, state, lifecycle, inheritance, or encapsulated behavior makes them appropriate.

Do not use a class merely because the concept has a noun.

Good class use:

```ts
class PackageCache {
  private readonly packages = new Map<string, Package>();

  get(path: string): Package | undefined {
    return this.packages.get(path);
  }

  set(path: string, pkg: Package): void {
    this.packages.set(path, pkg);
  }
}
```

A stateless formatter probably does not need a class.

---

## Inheritance

Favor composition over inheritance.

Do not introduce base classes just to share a few lines of behavior.

Avoid structures like:

```text
BaseProcessor
AbstractProcessor
DefaultProcessor
AdvancedProcessor
CustomProcessor
```

unless the domain genuinely models that hierarchy.

---

## Dependency Injection

Use the dependency pattern already present in the project.

TypeScript does not require a dependency injection container to achieve testability.

Often this is enough:

```ts
interface Dependencies {
  loadPackage(path: string): Promise<Package>;
}

export function createWeaponLoader(deps: Dependencies) {
  return async function loadWeapon(path: string) {
    const pkg = await deps.loadPackage(path);
    return parseWeapon(pkg);
  };
}
```

Do not add a DI framework merely to avoid importing a dependency.

---

## Avoid Unnecessary Interfaces

Do not create an interface for every implementation.

Avoid:

```ts
interface IWeaponParser {
  parse(data: WeaponData): Weapon;
}

class WeaponParser implements IWeaponParser {
  // ...
}
```

when there is only one implementation and no meaningful architectural boundary.

Prefer:

```ts
class WeaponParser {
  // ...
}
```

or simply:

```ts
function parseWeapon(data: WeaponData): Weapon {
  // ...
}
```

Also avoid C#-style `I` prefixes unless the repository explicitly uses them.

Idiomatic TypeScript normally uses:

```ts
interface WeaponParser
```

rather than:

```ts
interface IWeaponParser
```

---

## Framework Code

Follow the framework rather than fighting it.

For React, Vue, Angular, Svelte, Node.js, Next.js, Express, Fastify, Vite, Electron, or other frameworks:

- Follow the framework's established lifecycle.
- Use its standard conventions.
- Avoid custom wrappers around already-simple framework APIs.
- Do not rebuild functionality the framework already provides.

Do not introduce architectural layers imported from another ecosystem unless they solve a real problem.

---

## React

When working in React:

- Prefer functional components unless the project uses classes.
- Keep components focused.
- Keep state as local as practical.
- Do not put every value in state.
- Do not add `useMemo` or `useCallback` automatically.
- Do not add `useEffect` for values that can be calculated during render.
- Avoid prop drilling only when it becomes an actual problem.
- Do not introduce global state management for a few local values.
- Follow the project's React version and conventions.

Avoid:

```ts
const processedWeapons = useMemo(
  () => weapons.map(processWeapon),
  [weapons],
);
```

unless memoization has a reason.

Prefer:

```ts
const processedWeapons = weapons.map(processWeapon);
```

until performance evidence or component behavior justifies memoization.

---

## State

Keep state minimal.

Derived values usually should not become separate state.

Bad:

```ts
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");
const [fullName, setFullName] = useState("");

useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);
```

Prefer:

```ts
const fullName = `${firstName} ${lastName}`;
```

Avoid synchronized duplicate state.

---

## DOM Code

Prefer standard browser APIs where they are sufficient.

Use event delegation when appropriate.

Do not create wrappers around basic DOM operations unless they solve repeated complexity.

Use semantic HTML before adding JavaScript to reproduce native behavior.

---

## Node.js

Use modern Node APIs supported by the project's declared version.

Prefer:

```ts
import { readFile } from "node:fs/promises";
```

over callback wrappers when promises are appropriate.

Do not assume the latest Node version if the project's runtime target is older.

Check:

```text
package.json
engines
tsconfig.json
CI configuration
deployment target
```

before relying on new APIs.

---

## ESM and CommonJS

Respect the project's module system.

Do not mix:

```ts
require()
module.exports
```

with:

```ts
import
export
```

without understanding the build environment.

Check:

```json
"type": "module"
```

and the TypeScript module settings before changing imports.

Do not perform an ESM migration as part of unrelated work.

---

## File Extensions

Follow the runtime and bundler requirements.

Do not blindly remove or add `.js`, `.ts`, or `.mjs` extensions from imports.

Native Node ESM projects may require:

```ts
import { parseWeapon } from "./weapon.js";
```

even though the source file is `weapon.ts`.

Respect the actual toolchain.

---

## Logging

Use the project's logging abstraction.

Log meaningful events.

Do not log every:

- Function entry.
- Function exit.
- Successful operation.
- Branch.
- Variable value.

Bad:

```ts
logger.info("Starting weapon processing");
logger.info("Weapon processing completed");
```

Better:

```ts
logger.warn({ assetPath }, "Weapon asset could not be resolved");
```

Do not log the same error at every layer.

Prefer logging where the error can be meaningfully handled or observed.

---

## Console Usage

Avoid `console.log` in production code unless the repository intentionally uses console logging.

Do not leave debugging statements behind:

```ts
console.log("here");
console.log(data);
console.log("test");
```

Use the established logger or remove them before finishing.

---

## Avoid Premature Abstraction

Do not design for hypothetical future requirements.

Do not add:

- Factories for one implementation.
- Strategy patterns for one algorithm.
- Interfaces for every module.
- Repositories over APIs that are already straightforward.
- Builders for simple object construction.
- Custom event buses for local state.
- Generic service containers.
- Complex dependency injection.
- Plugin systems for a single implementation.
- Adapter layers with no actual adaptation.
- Custom result types when the project already has an established error model.

Build what the current requirement needs.

---

## Avoid AI-Looking Architecture

Do not automatically create structures such as:

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
Controller
Facade
Orchestrator
```

These names are not inherently bad.

They are bad when they exist only because generated code tends to produce symmetrical architecture.

Use them only when they describe a real architectural role.

---

## Avoid Excessive Wrapper Functions

Do not write:

```ts
function fetchWeaponData(id: string) {
  return fetchWeapon(id);
}
```

unless the wrapper adds actual behavior or creates a meaningful boundary.

Every abstraction should earn its existence.

---

## Avoid Excessive Configuration

Do not make every literal configurable.

Bad:

```ts
interface WeaponProcessorOptions {
  enableParsing?: boolean;
  enableValidation?: boolean;
  enableNormalization?: boolean;
  enableTransformation?: boolean;
  enableLogging?: boolean;
}
```

when the feature always needs those operations.

Configuration is complexity.

Expose configuration only when callers genuinely need control.

---

## Avoid Boolean Parameter Piles

Avoid APIs such as:

```ts
processWeapon(weapon, true, false, true);
```

Use an options object if several independent behaviors genuinely need configuration:

```ts
processWeapon(weapon, {
  validate: true,
  normalize: false,
  includeMetadata: true,
});
```

However, do not create an options object when there is only one obvious argument.

---

## Immutability

Prefer avoiding surprising mutation.

Do not treat immutability as a religion.

This is perfectly reasonable:

```ts
const results: Weapon[] = [];

for (const asset of assets) {
  const weapon = parseWeapon(asset);

  if (weapon) {
    results.push(weapon);
  }
}
```

Do not rewrite readable imperative code into complex immutable expressions solely to avoid mutation.

---

## `readonly`

Use `readonly` where it communicates an actual invariant.

Example:

```ts
interface Weapon {
  readonly id: string;
  name: string;
}
```

Do not add `readonly` recursively across every object unless the architecture benefits from that guarantee.

---

## Formatting

Use the project's formatter.

Usually this means Prettier or an equivalent tool.

Do not manually enforce formatting conventions that fight automated tooling.

Avoid whitespace-only churn in unrelated files.

Do not reformat an entire file just because one function changed unless the project's tooling does so automatically.

---

## ESLint

Respect the project's ESLint configuration.

Do not disable lint rules merely to make generated code compile.

Avoid:

```ts
// eslint-disable-next-line
```

unless:

- The rule genuinely does not apply.
- The reason is understood.
- A better implementation would be worse.

Never add file-wide lint disables casually.

When disabling a non-obvious rule, explain why.

---

## Prettier

Let Prettier handle formatting when the project uses it.

Do not fight Prettier with manual alignment.

Avoid:

```ts
const short     = 1;
const muchLonger = 2;
```

Do not add prettier-ignore comments without a specific reason.

---

## `tsconfig`

Respect the project's TypeScript configuration.

Do not casually change:

- `target`
- `module`
- `moduleResolution`
- `strict`
- `jsx`
- `types`
- `lib`
- `esModuleInterop`
- `allowJs`
- `skipLibCheck`

These options can affect the entire codebase.

Only change them when required by the task and after understanding the runtime and build system.

---

## Compiler Errors

Do not fix compiler errors by weakening type safety globally.

Avoid changes such as:

```json
{
  "strict": false,
  "noImplicitAny": false
}
```

just to get the project compiling.

Fix the local type problem.

Likewise, do not solve errors by turning everything into:

```ts
any
```

or:

```ts
as SomeType
```

---

## External Libraries

Before adding a dependency, check whether:

- The platform already provides the functionality.
- The repository already has a library that does it.
- The implementation is simple enough to write directly.
- The dependency is maintained.
- The bundle or runtime cost is justified.

Do not install a package to solve five lines of straightforward code.

Also do not reimplement complex, security-sensitive, or standards-heavy functionality merely to avoid a dependency.

---

## Dependency Usage

Use library APIs directly when they are already reasonable.

Avoid unnecessary wrappers like:

```ts
class ZodValidationService {
  validate<T>(schema: ZodSchema<T>, value: unknown): T {
    return schema.parse(value);
  }
}
```

unless the application actually needs an abstraction over its validator.

---

## Dates

Use the project's date handling strategy.

JavaScript `Date` is sufficient for many simple cases.

Do not add a date library solely to format one timestamp.

Likewise, do not hand-roll timezone or calendar logic where a proven library or platform API is more appropriate.

---

## Regular Expressions

Use regular expressions for problems they express clearly.

Do not write enormous regexes when explicit parsing is easier to understand.

For complex expressions, assign them meaningful names.

Example:

```ts
const assetPathPattern =
  /^\/Game\/[A-Za-z0-9_/]+$/;
```

Explain non-obvious regex behavior if needed.

---

## Security

Do not weaken security for convenience.

Never:

- Insert secrets into source code.
- Disable TLS validation.
- Trust external input blindly.
- Build SQL queries through string concatenation.
- Insert untrusted HTML without sanitization.
- Use `eval`.
- Use `new Function` without an exceptional reason.
- Disable authentication or authorization checks to make a feature work.

Use established project mechanisms.

---

## HTML and Browser Security

Avoid assigning untrusted content through:

```ts
element.innerHTML = userInput;
```

Use:

```ts
element.textContent = userInput;
```

or an established sanitization mechanism when HTML is actually required.

In React, avoid `dangerouslySetInnerHTML` unless the content is trusted or sanitized.

---

## Tests

Tests should verify behavior, not implementation details.

Use the repository's existing testing framework and naming conventions.

Examples:

```ts
it("returns undefined when the weapon does not exist", () => {
  // ...
});
```

or:

```ts
test("parseWeapon rejects malformed weapon data", () => {
  // ...
});
```

Do not rewrite test style across the repository.

---

## Test Real Behavior

Prefer tests that exercise real logic.

Do not mock every dependency automatically.

Use real lightweight objects when practical.

Mock:

- Networks.
- Databases.
- External services.
- Time when necessary.
- Expensive or nondeterministic dependencies.

Do not mock simple pure functions or plain data objects merely because mocking is available.

---

## Avoid Overbuilt Test Helpers

Do not create:

```text
TestDataBuilder
MockFactory
FixtureManager
TestingUtilityService
```

for three simple tests.

Inline setup is often clearer.

Extract shared test utilities when repetition actually becomes burdensome.

---

## Test Names

Describe behavior and conditions.

Good:

```ts
it("returns null when the asset has no weapon definition", ...)
```

Avoid:

```ts
it("test weapon parser", ...)
```

or excessively formal names generated from templates.

---

## Do Not Test TypeScript Itself

Do not write tests for trivial language behavior.

Avoid tests such as:

```ts
expect([1, 2, 3].length).toBe(3);
```

Focus on application behavior.

---

## Frontend Components

Do not split components solely based on line count.

Extract components when they:

- Represent a meaningful UI concept.
- Are reused.
- Manage their own behavior.
- Make a parent substantially easier to understand.

Avoid turning every `<div>` into its own component.

Likewise, do not allow one giant component to own unrelated behaviors.

---

## CSS and Styling

Follow the project's styling system.

If it uses Tailwind:

- Use existing design tokens.
- Avoid arbitrary values unless necessary.
- Do not extract every class list into constants.
- Do not build unnecessary wrapper components purely to reuse class names.

If it uses CSS modules, styled-components, vanilla CSS, or another system, follow that system.

Do not introduce a second styling architecture casually.

---

## Event Handlers

Use descriptive names:

```ts
handleSubmit
handleSearchChange
handleWeaponSelect
```

or whatever pattern the project already uses.

Do not create handlers solely to forward a single call unless readability benefits.

This is fine:

```tsx
<button onClick={() => removeWeapon(id)}>
  Remove
</button>
```

A separate:

```ts
const handleRemoveWeaponClick = () => {
  removeWeapon(id);
};
```

is not automatically better.

---

## API Boundaries

Keep external transport models separate from domain models when they materially differ.

Do not create duplicate DTO/domain layers merely because enterprise architecture patterns say you should.

If the API representation and application representation are identical and stable, one type may be enough.

Add mapping when there is actual transformation or boundary value.

---

## Database Code

Follow the database library or ORM's established patterns.

Avoid adding repository abstractions on top of an ORM unless they create genuine value.

Do not turn:

```ts
db.weapon.findUnique({ where: { id } });
```

into several layers of pass-through services with no additional behavior.

---

## Return Types

Add explicit return types where they improve API clarity, especially exported functions.

Example:

```ts
export function parseWeapon(
  data: WeaponData,
): Weapon {
  // ...
}
```

For small local functions where inference is obvious, inferred return types are fine.

Do not annotate every local arrow function mechanically.

---

## Public APIs

Be more deliberate with exported APIs than internal implementation.

For public functions:

- Prefer clear parameter and return types.
- Avoid leaking unnecessary implementation-specific types.
- Preserve backward compatibility when appropriate.
- Document non-obvious behavior.
- Avoid adding parameters callers do not need.

Internal code can remain simpler.

---

## Function Parameters

Avoid large argument lists.

Bad:

```ts
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

Prefer an object when the parameters belong together:

```ts
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

Do not use an options object for every two-argument function.

---

## Avoid Unnecessary Object Wrapping

Do not convert:

```ts
loadWeapon(id);
```

into:

```ts
loadWeapon({
  id,
});
```

unless more related parameters are reasonably expected or already present.

Do not optimize APIs for imaginary future requirements.

---

## Pure Functions

Prefer pure functions where they naturally fit.

Do not force all code to be functional.

Side effects are normal for:

- I/O
- Logging
- DOM updates
- Database operations
- File operations
- Network requests
- Cache updates

Keep side effects obvious and located near appropriate boundaries.

---

## Caching

Do not introduce caching without a reason.

Caching adds:

- Invalidations.
- Stale data.
- Memory usage.
- Concurrency concerns.
- More states to debug.

If caching is required, define:

- What is cached.
- Cache lifetime.
- Invalidation behavior.
- Memory bounds if relevant.

Do not add memoization because an operation "might be expensive."

---

## Performance

Prefer clear code until there is a real performance concern.

Do not prematurely:

- Memoize everything.
- Pool small objects.
- Replace arrays with complex structures.
- Parallelize trivial work.
- Write unreadable loops to avoid allocations.
- Introduce worker threads.

When performance genuinely matters, optimize the measured bottleneck.

---

## Do Not Rewrite Working Code Unnecessarily

When modifying an existing feature:

1. Identify the smallest responsible area.
2. Understand existing behavior.
3. Preserve unrelated behavior.
4. Match the current architecture.
5. Change only what the task requires.
6. Update tests for behavior that changed.
7. Avoid unrelated cleanup.

Do not rewrite a working module merely because you would have implemented it differently.

---

## Preserve Behavior During Refactors

When asked specifically to refactor:

- Preserve observable behavior unless a change is requested.
- Separate structural changes from behavioral changes when practical.
- Keep public APIs stable unless changing them is part of the task.
- Run existing tests before and after where possible.
- Avoid simultaneous style migrations.

A refactor should make the code easier to maintain, not merely different.

---

## Do Not Produce Placeholder Architecture

Avoid generated placeholders such as:

```ts
// TODO: Implement advanced validation
// TODO: Add caching
// TODO: Add retry logic
// TODO: Add monitoring
```

unless the user explicitly asked for a scaffold.

Implement the requested behavior fully.

Do not leave speculative TODOs behind.

---

## Avoid Fake Robustness

Do not add error handling for impossible internal conditions solely to look defensive.

Bad:

```ts
if (!weapon) {
  throw new Error("Weapon is null");
}
```

inside a function whose parameter type is:

```ts
weapon: Weapon
```

Trust internal type contracts unless data has crossed an unsafe runtime boundary.

Strong types should eliminate unnecessary defensive code.

---

## Avoid Fake Flexibility

Do not create extension points no one needs.

Avoid:

```ts
interface WeaponProcessingStrategy {
  process(weapon: Weapon): Weapon;
}
```

with exactly one implementation.

Prefer direct code until multiple behaviors actually exist.

---

## Avoid AI-Generated Prose in Code

Comments, errors, and documentation should be concise.

Avoid phrases like:

```text
This function is responsible for...
This method ensures that...
The following code handles...
In order to...
It is important to note that...
This provides a robust and flexible solution...
```

Write like a maintainer, not a tutorial.

Example:

Bad:

```ts
// This function is responsible for ensuring that the weapon
// data is properly validated before proceeding with processing.
```

Better:

```ts
// External weapon manifests are not guaranteed to match our schema.
```

---

## Avoid Excessive Section Comments

Do not divide ordinary files with generated-looking blocks such as:

```ts
// ========================================
// INITIALIZATION
// ========================================
```

unless the repository consistently uses that convention.

Good file structure should usually make these unnecessary.

---

## Avoid Decorative Documentation

Do not write documentation purely to make a change appear complete.

Do not create README sections for trivial internal helpers.

Documentation should describe things a future developer actually needs to know.

---

## Modern TypeScript

Use modern TypeScript and JavaScript features when they improve readability and the project's runtime supports them.

Useful features may include:

- Optional chaining.
- Nullish coalescing.
- `satisfies`.
- Discriminated unions.
- Template literal types.
- `as const`.
- `readonly`.
- Top-level `await` where supported.
- Private class fields where appropriate.
- Object and array destructuring.
- `Promise.all`.
- `Map`.
- `Set`.

Do not use modern syntax merely to make the code appear sophisticated.

---

## `satisfies`

Use `satisfies` when you want to validate an object's shape without unnecessarily widening its inferred type.

Example:

```ts
const categories = {
  rifle: "Rifle",
  smg: "SMG",
  shotgun: "Shotgun",
} satisfies Record<string, string>;
```

Prefer it over a type assertion when it represents the actual intent.

Do not add it everywhere mechanically.

---

## `as const`

Use `as const` when literal preservation is useful.

Example:

```ts
const weaponCategories = [
  "rifle",
  "smg",
  "shotgun",
] as const;

type WeaponCategory =
  (typeof weaponCategories)[number];
```

Do not use `as const` as an automatic suffix on every object.

---

## Exhaustive Checks

For important discriminated unions, use exhaustive checks when they improve safety.

Example:

```ts
switch (result.status) {
  case "success":
    return result.weapon;

  case "not-found":
    return null;

  case "error":
    throw result.error;

  default: {
    const exhaustive: never = result;
    return exhaustive;
  }
}
```

Do not add exhaustive-check utilities to trivial code where the benefit is negligible.

---

## Before Finishing

Review the change and remove:

- Redundant comments.
- Tutorial-style prose.
- Unnecessary abstractions.
- Unused helpers.
- Unused types.
- Duplicate validation.
- Generic AI-style names.
- Excessive interfaces.
- Excessive generics.
- Unnecessary classes.
- Unnecessary factories.
- Unnecessary wrappers.
- Type assertions used to silence errors.
- Accidental `any`.
- Excessive logging.
- Redundant null checks.
- Speculative configuration.
- Speculative extensibility.
- Debugging statements.
- Unrelated refactors.
- Placeholder TODOs.

Then run the project's existing checks where available.

Typical TypeScript checks include:

```bash
npm run lint
npm run typecheck
npm test
npm run build
```

or their repository-specific equivalents.

Common underlying tools include:

```bash
eslint .
tsc --noEmit
prettier --check .
```

Do not assume these exact commands exist. Inspect `package.json` first and use the project's defined scripts.

The final code should look like it naturally belongs in the repository rather than like a standalone AI-generated solution.