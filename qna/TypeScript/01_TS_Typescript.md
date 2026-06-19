# TypeScript – Interview Questions & Answers

**Question**: What are the key differences between TypeScript and JavaScript?

**Answer**: TypeScript is a statically typed superset of JavaScript that compiles to plain JS. It adds optional static typing, interfaces, generics, enums, and advanced type-level programming, catching errors at compile time rather than runtime. JavaScript is dynamically typed and runs directly in browsers/Node.js without a compilation step.

```typescript
// TypeScript catches type errors at compile time
function greet(name: string): string {
  return `Hello, ${name}`;
}
// greet(42); // ❌ Error: Argument of type 'number' is not assignable to parameter of type 'string'
```

---

**Question**: When should you use `interface` vs `type` alias?

**Answer**: Use `interface` when you need declaration merging or are defining object shapes for public APIs. Use `type` for unions, intersections, mapped types, or when you need computed properties. Both are largely interchangeable for object types, but interfaces are more idiomatic for class contracts.

```typescript
// Interface - supports declaration merging
interface User { name: string; }
interface User { age: number; } // Merged: User has both name and age

// Type alias - supports unions and primitives
type Status = 'active' | 'inactive';
type Point = { x: number; y: number; };
```

---

**Question**: Explain union types and intersection types.

**Answer**: Union types (`|`) allow a value to be one of several types, while intersection types (`&`) combine multiple types into one. Unions express "either/or" relationships; intersections express "all-of" combinations.

```typescript
// Union: value can be string OR number
function formatId(id: string | number): string {
  return `ID: ${id}`;
}

// Intersection: combines Admin and User into AdminUser
type Admin = { role: 'admin'; permissions: string[] };
type User = { name: string; email: string };
type AdminUser = Admin & User;
// { role: 'admin'; permissions: string[]; name: string; email: string }
```

---

**Question**: What are literal types and how does type narrowing work?

**Answer**: Literal types allow specific values as types (e.g., `'success'` or `42`). Type narrowing is the process of refining a broad type to a more specific one using control flow analysis like `typeof`, `instanceof`, or equality checks.

```typescript
type Direction = 'left' | 'right' | 'up' | 'down';

function move(direction: Direction): void {
  if (direction === 'left') {
    // narrowed to 'left'
  }
}

type Result = { status: 'success'; data: string } | { status: 'error'; message: string };
function handle(result: Result): void {
  if (result.status === 'success') {
    console.log(result.data); // narrowed: data is accessible
  }
}
```

---

**Question**: What is a discriminated union and why is it useful?

**Answer**: A discriminated union is a union type where each member shares a common literal property (the discriminant). TypeScript uses this property to narrow the type, enabling exhaustive pattern matching and ensuring all cases are handled at compile time.

```typescript
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; side: number }
  | { kind: 'triangle'; base: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':   return Math.PI * shape.radius ** 2;
    case 'square':   return shape.side ** 2;
    case 'triangle': return 0.5 * shape.base * shape.height;
  }
}
```

---

**Question**: What are type guards? Explain `typeof`, `instanceof`, and custom type guards.

**Answer**: Type guards are expressions that perform runtime checks to narrow a variable's type. `typeof` works for primitives, `instanceof` for class instances, and custom type guards are user-defined functions that return a type predicate (`x is T`).

```typescript
// typeof type guard
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

// instanceof type guard
class Dog { bark() {} }
class Cat { meow() {} }
function makeSound(animal: Dog | Cat): void {
  if (animal instanceof Dog) animal.bark();
  else animal.meow();
}

// Custom type guard
interface Fish { swim(): void; }
interface Bird { fly(): void; }
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

---

**Question**: What are assertion functions?

**Answer**: Assertion functions are functions that throw if a condition is false, narrowing the type for the remainder of scope. They use a special return type of `asserts condition` or `asserts x is T`.

```typescript
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== 'string') throw new Error('Not a string');
}

function processValue(value: unknown): void {
  assertIsString(value);
  value.toUpperCase(); // value is narrowed to string
}
```

---

**Question**: Explain generics with constraints, conditional types, and mapped types.

**Answer**: Generics allow creating reusable components that work with multiple types. Constraints (`extends`) limit what types are accepted. Conditional types select types based on conditions (`T extends U ? X : Y`). Mapped types transform properties of an existing type.

```typescript
// Generic with constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Conditional type
type IsString<T> = T extends string ? 'yes' : 'no';
type A = IsString<'hello'>; // 'yes'
type B = IsString<number>;  // 'no'

// Mapped type
type Readonly<T> = { readonly [P in keyof T]: T[P] };
```

---

**Question**: What are the `keyof` and `typeof` operators in TypeScript?

**Answer**: `keyof` yields a union of a type's property keys. `typeof` (in type context) captures the type of a value at the type level. Together they enable type-safe access patterns and dynamic property handling.

```typescript
const user = { name: 'Alice', age: 30 };
type UserType = typeof user; // { name: string; age: number }
type UserKeys = keyof UserType; // 'name' | 'age'

function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

---

**Question**: What are indexed access types?

**Answer**: Indexed access types use the bracket syntax `T[K]` to look up a property type at a given key. They allow dynamic access to the type of a nested property.

```typescript
interface User {
  profile: {
    name: string;
    age: number;
  };
  settings: {
    theme: 'light' | 'dark';
  };
}

type ProfileType = User['profile']; // { name: string; age: number }
type AgeType = User['profile']['age']; // number
type Keys = 'profile' | 'settings';
type Subset = User[Keys]; // { name: string; age: number } | { theme: 'light' | 'dark' }
```

---

**Question**: Explain Partial, Required, Readonly, Pick, and Omit utility types.

**Answer**: These are built-in mapped types: `Partial<T>` makes all properties optional, `Required<T>` makes them mandatory, `Readonly<T>` prevents reassignment, `Pick<T, K>` selects specific keys, and `Omit<T, K>` excludes keys.

```typescript
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type PartialTodo = Partial<Todo>;      // { title?: string; description?: string; completed?: boolean }
type RequiredTodo = Required<PartialTodo>; // all required again
type ReadonlyTodo = Readonly<Todo>;      // { readonly title: string; ... }
type PickTodo = Pick<Todo, 'title' | 'completed'>; // { title: string; completed: boolean }
type OmitTodo = Omit<Todo, 'description'>; // { title: string; completed: boolean }
```

---

**Question**: Explain Record, Exclude, Extract, NonNullable, ReturnType, and Parameters.

**Answer**: `Record<K, V>` creates an object type with keys K and values V. `Exclude<T, U>` removes types from T. `Extract<T, U>` extracts types present in both. `NonNullable<T>` removes `null` and `undefined`. `ReturnType<T>` extracts the return type. `Parameters<T>` extracts parameter types.

```typescript
type RecordType = Record<'a' | 'b', number>; // { a: number; b: number }
type Excluded = Exclude<'a' | 'b' | 'c', 'a'>; // 'b' | 'c'
type Extracted = Extract<'a' | 'b' | 'c', 'a' | 'c'>; // 'a' | 'c'
type NonNull = NonNullable<string | null | undefined>; // string

function greet(name: string, age: number): boolean {
  return age > 18;
}
type GreetParams = Parameters<typeof greet>; // [string, number]
type GreetReturn = ReturnType<typeof greet>; // boolean
```

---

**Question**: What are template literal types?

**Answer**: Template literal types are types built from string literals using template syntax. They can be combined with unions to generate multiple string patterns at the type level.

```typescript
type EventName = 'click' | 'focus' | 'blur';
type HandlerName = `on${Capitalize<EventName>}`; // 'onClick' | 'onFocus' | 'onBlur'

type Path = `/${string}`;
function setRoute(path: Path): void {}
setRoute('/users'); // ✅
// setRoute('users'); // ❌
```

---

**Question**: How do conditional types work with the `extends ? :` syntax?

**Answer**: Conditional types select between two types based on a boolean condition expressed as `T extends U ? X : Y`. If T is assignable to U, the type resolves to X; otherwise Y. They are the type-level equivalent of ternary expressions.

```typescript
type IsArray<T> = T extends unknown[] ? 'array' : 'not array';
type A = IsArray<string[]>;  // 'array'
type B = IsArray<number>;    // 'not array'

type ExtractStrings<T> = T extends string ? T : never;
type Strings = ExtractStrings<'a' | 1 | 'b' | 2>; // 'a' | 'b'
```

---

**Question**: What is the `infer` keyword in conditional types?

**Answer**: `infer` allows you to declare a type variable within the `extends` clause of a conditional type, enabling extraction of type information. It's commonly used to unwrap generic types.

```typescript
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;
type A = UnwrapPromise<Promise<string>>; // string
type B = UnwrapPromise<number>;          // number

type ReturnType<T> = T extends (...args: unknown[]) => infer R ? R : never;
type FirstArg<T> = T extends (first: infer F, ...rest: unknown[]) => unknown ? F : never;
```

---

**Question**: Explain the difference between `unknown`, `any`, and `never`.

**Answer**: `any` disables type checking entirely—use sparingly. `unknown` is the type-safe counterpart: you must narrow it before use. `never` represents values that never occur (unreachable code) and is the bottom type assignable to everything but nothing is assignable to it.

```typescript
let anyVal: any = 42;
anyVal.toUpperCase(); // ✅ no error, but unsafe

let unknownVal: unknown = 42;
// unknownVal.toUpperCase(); // ❌ Error: Object is of type 'unknown'
if (typeof unknownVal === 'string') {
  unknownVal.toUpperCase(); // ✅ narrowed
}

function throwError(): never {
  throw new Error('Always throws');
}
```

---

**Question**: What is the difference between `void` and `undefined`?

**Answer**: `void` is the return type of functions that don't return a value. `undefined` is an actual value. In JavaScript, a function with no return statement returns `undefined`, but in TypeScript, `void` is assignable from `undefined` only when `strictNullChecks` is off. `void` is preferred for callback return types to allow both `undefined` and implicit returns.

```typescript
function logMessage(msg: string): void {
  console.log(msg);
  // no return statement
}

type VoidFunc = () => void;
const fn: VoidFunc = () => 42; // ✅ void return type allows returning other values

type UndefinedFunc = () => undefined;
// const fn2: UndefinedFunc = () => 42; // ❌
```

---

**Question**: Explain the `readonly` modifier and `ReadonlyArray`.

**Answer**: `readonly` prevents property reassignment in types/interfaces. `ReadonlyArray<T>` (or `readonly T[]`) creates an immutable array where methods like `push` and `pop` are unavailable. The type enforces immutability at compile time only.

```typescript
interface Point {
  readonly x: number;
  readonly y: number;
}
const p: Point = { x: 1, y: 2 };
// p.x = 3; // ❌ Error: Cannot assign to 'x'

const arr: readonly number[] = [1, 2, 3];
// arr.push(4);  // ❌ Property 'push' does not exist
// arr[0] = 10;  // ❌ Index signature in readonly array
```

---

**Question**: What is `as const` and how do const assertions work?

**Answer**: `as const` tells TypeScript to infer the narrowest possible type for a value—literal types instead of widened types, and readonly tuples instead of arrays. It's essential for creating immutable constants and discriminated unions.

```typescript
const colors = ['red', 'green', 'blue'] as const;
type Color = (typeof colors)[number]; // 'red' | 'green' | 'blue'

const config = {
  api: 'https://api.example.com',
  timeout: 5000,
} as const;
// config.timeout = 6000; // ❌ Error: Cannot assign to 'timeout' because it is a read-only property
```

---

**Question**: What are function overloads in TypeScript?

**Answer**: Function overloads provide multiple type signatures for a single implementation. The implementation signature must be compatible with all overloads. Overloads improve type safety for functions that accept varying argument patterns.

```typescript
function process(input: string): string;
function process(input: number): number;
function process(input: string | number): string | number {
  if (typeof input === 'string') return input.toUpperCase();
  return input * 2;
}

const a = process('hello'); // string
const b = process(42);      // number
```

---

**Question**: When would you use function overloads vs rest parameters (`...args`)?

**Answer**: Use overloads when you need different return types based on argument types or when argument patterns don't fit a uniform structure. Use rest parameters with union types when the arguments follow a consistent pattern.

```typescript
// Overloads for different return types
function getValue(key: 'name'): string;
function getValue(key: 'count'): number;
function getValue(key: string): unknown {
  // implementation
}

// Rest parameters for variadic arguments of the same type
function sum(...args: number[]): number {
  return args.reduce((a, b) => a + b, 0);
}
```

---

**Question**: What is the `this` parameter in TypeScript functions?

**Answer**: The `this` parameter is a fake parameter that defines the expected type of `this` inside a function. It ensures methods are called with the correct context and prevents common `this`-related bugs.

```typescript
interface User {
  name: string;
}

function greet(this: User, greeting: string): string {
  return `${greeting}, ${this.name}`;
}

const user: User = { name: 'Alice' };
greet.call(user, 'Hello'); // ✅ 'Hello, Alice'
// greet('Hello'); // ❌ Error: 'this' context is missing
```

---

**Question**: What are `abstract` classes and methods?

**Answer**: Abstract classes cannot be instantiated directly and serve as base classes. Abstract methods define a signature that derived classes must implement. They enforce a contract while allowing shared implementation.

```typescript
abstract class Animal {
  constructor(protected name: string) {}
  abstract makeSound(): void; // must be implemented by subclasses
  move(): void {
    console.log(`${this.name} is moving`);
  }
}

class Dog extends Animal {
  makeSound(): void {
    console.log('Woof!');
  }
}
// const animal = new Animal('generic'); // ❌ Cannot create instance of abstract class
```

---

**Question**: Explain access modifiers: `public`, `private`, `protected`, and `readonly`.

**Answer**: `public` (default) is accessible everywhere. `private` is accessible only within the declaring class. `protected` is accessible within the class and its subclasses. `readonly` prevents reassignment after initialization. TypeScript's `private` uses a soft compile-time check—use `#` for true JavaScript private fields.

```typescript
class Person {
  public name: string;
  private ssn: string;
  protected age: number;
  readonly id: number;

  constructor(name: string, ssn: string, age: number, id: number) {
    this.name = name;
    this.ssn = ssn;
    this.age = age;
    this.id = id;
  }
}
```

---

**Question**: What are decorators and what types of decorators exist?

**Answer**: Decorators are functions that modify classes, methods, properties, or parameters. They use the `@expression` syntax and are commonly used for metadata injection, logging, and dependency injection. TypeScript supports class, method, accessor, property, and parameter decorators.

```typescript
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor): void {
  const original = descriptor.value;
  descriptor.value = function (...args: unknown[]) {
    console.log(`Called ${propertyKey} with`, args);
    return original.apply(this, args);
  };
}

class Calculator {
  @Log
  add(a: number, b: number): number {
    return a + b;
  }
}
```

---

**Question**: What is declaration merging?

**Answer**: Declaration merging is when TypeScript combines multiple declarations of the same name into a single definition. It primarily applies to interfaces, enums, and namespaces. This is useful for augmenting types from external libraries.

```typescript
interface Box {
  height: number;
}
interface Box {
  width: number;
}
// Box now has both height and width
const box: Box = { height: 10, width: 20 };

// Cannot merge type aliases
type Person = { name: string };
// type Person = { age: number }; // ❌ Duplicate identifier
```

---

**Question**: What is module augmentation?

**Answer**: Module augmentation allows adding new members to types declared in external modules. It uses `declare module 'module-name'` to extend or modify the module's types, often used to add custom properties to third-party types.

```typescript
// express.d.ts
import 'express';

declare module 'express' {
  interface Request {
    user?: { id: string; name: string };
  }
}

// In application code
app.use((req, res, next) => {
  console.log(req.user?.name); // ✅ augmented property
  next();
});
```

---

**Question**: What is the difference between namespaces and modules?

**Answer**: Namespaces (formerly "internal modules") are TypeScript-specific constructs for organizing code within a global scope, but they don't follow the ES module standard. Modules (using `import`/`export`) are the modern standard aligned with ES6+, providing better tree-shaking, static analysis, and interoperability with bundlers.

```typescript
// Namespace (legacy, avoid in new code)
namespace Validation {
  export interface StringValidator {
    isValid(s: string): boolean;
  }
}

// Module (modern standard)
export interface StringValidator {
  isValid(s: string): boolean;
}
import { StringValidator } from './validators';
```

---

**Question**: What are triple-slash directives?

**Answer**: Triple-slash directives are single-line comments that serve as compiler directives, used primarily in declaration files. `/// <reference path="..." />` indicates a dependency on another file, `/// <reference types="..." />` references type declarations, and `/// <reference lib="..." />` includes a built-in lib file.

```typescript
/// <reference path="./types.d.ts" />
/// <reference types="node" />
/// <reference lib="es2020" />
```

---

**Question**: What are the key `tsconfig.json` compiler options?

**Answer**: Key options include `strict` (enables all strict checks), `strictNullChecks` (prevents `null`/`undefined` from being assignable to other types), `noImplicitAny` (errors on inferred `any`), `target` (output JS version), `module` (module system), `lib` (type definitions), `outDir`, `rootDir`, `sourceMap`, and `esModuleInterop`.

```json
{
  "compilerOptions": {
    "strict": true,
    "strictNullChecks": true,
    "noImplicitAny": true,
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2022", "DOM"],
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true
  }
}
```

---

**Question**: Explain path mapping and module resolution strategies in TypeScript.

**Answer**: Path mapping in `tsconfig.json` allows defining custom module aliases via the `paths` option relative to `baseUrl`. TypeScript supports two resolution strategies: `classic` (legacy, for AMD) and `node` (default, mimics Node.js resolution with `.ts`/`.d.ts` extensions).

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    }
  }
}
// Now import Foo from '@/components/Foo';
```

---

**Question**: What are `@types` and DefinitelyTyped?

**Answer**: DefinitelyTyped is the community-maintained repository of type definitions for npm packages that don't ship their own types. `@types/package-name` packages are published to npm and automatically resolved by TypeScript's compiler. The `types` option in `tsconfig.json` controls which `@types` packages are included.

---

**Question**: What are ambient declarations (`declare module`, `declare global`)?

**Answer**: Ambient declarations tell TypeScript about the shape of values that exist at runtime without providing implementation. `declare module 'foo'` types a module without installing types. `declare global` augments the global scope inside a module file.

```typescript
// ambient-module.d.ts
declare module 'my-untyped-lib' {
  export function doSomething(): void;
  export const version: string;
}

// global-augmentation.d.ts
export {};

declare global {
  interface Window {
    myAppVersion: string;
  }
}
```

---

**Question**: What are type-only imports (`import type`)?

**Answer**: `import type` imports only the type information at compile time and is completely erased during runtime. This avoids bundler issues with circular dependencies and enables transpilers like Babel to safely handle type imports. `export type` works similarly for re-exports.

```typescript
import type { User } from './types'; // Only type info, no runtime code
import { createUser } from './utils'; // Runtime import

export type { User }; // Re-export only the type
```

---

**Question**: What is the `satisfies` operator (TS 4.9+)?

**Answer**: `satisfies` validates that a value's type matches a given type without changing the inferred type. It ensures type safety while preserving the narrowest inferred literal type, unlike type annotations which widen the type.

```typescript
type Colors = 'red' | 'green' | 'blue';
type Palette = Record<string, Colors>;

const palette = {
  primary: 'red',
  secondary: 'blue',
} satisfies Palette;
// palette.primary is 'red' (literal), not Colors (widened)

// Without satisfies:
const palette2: Palette = {
  primary: 'red', // widened to Colors
  secondary: 'blue',
};
```

---

**Question**: What are variadic tuple types?

**Answer**: Variadic tuple types use generic spread syntax (`...T`) to create tuples with variable length and element types. They enable type-safe operations on tuple lengths and element type mapping, introduced in TypeScript 4.0+.

```typescript
function concat<T extends unknown[], U extends unknown[]>(
  a: [...T],
  b: [...U]
): [...T, ...U] {
  return [...a, ...b];
}
const result = concat([1, 2] as const, ['a', 'b'] as const);
// type: readonly [1, 2, 'a', 'b']

type Head<T extends unknown[]> = T extends [infer First, ...unknown[]] ? First : never;
type Tail<T extends unknown[]> = T extends [unknown, ...infer Rest] ? Rest : never;
```

---

**Question**: How do template literal types combine with unions?

**Answer**: When template literal types contain union types in placeholders, TypeScript generates the Cartesian product of all combinations. This is powerful for generating event handlers, CSS properties, and API paths at the type level.

```typescript
type Size = 'sm' | 'md' | 'lg';
type Color = 'red' | 'blue';

type ClassName = `${Size}-${Color}`;
// 'sm-red' | 'sm-blue' | 'md-red' | 'md-blue' | 'lg-red' | 'lg-blue'

type VerticalAlign = 'top' | 'bottom';
type HorizontalAlign = 'left' | 'right';
type Corner = `${VerticalAlign}-${HorizontalAlign}`;
// 'top-left' | 'top-right' | 'bottom-left' | 'bottom-right'
```

---

**Question**: What are branded types or nominal typing in TypeScript?

**Answer**: Branded types simulate nominal (name-based) typing in TypeScript's structural type system by adding a unique brand property. This prevents accidental mixing of types that have the same shape but represent different logical concepts.

```typescript
type Brand<T, B extends string> = T & { __brand: B };
type UserId = Brand<string, 'UserId'>;
type OrderId = Brand<string, 'OrderId'>;

function getUser(id: UserId): void {}
function getOrder(id: OrderId): void {}

const uid = 'abc123' as UserId;
const oid = 'xyz789' as OrderId;

getUser(uid); // ✅
// getUser(oid); // ❌ Type 'OrderId' is not assignable to parameter of type 'UserId'
```

---

**Question**: What is `NoUncheckedIndexedAccess`?

**Answer**: `NoUncheckedIndexedAccess` is a `tsconfig.json` option that adds `undefined` to every indexed access. This forces you to handle the case where an index might not exist, preventing runtime "undefined is not a function" errors.

```json
{
  "compilerOptions": {
    "noUncheckedIndexedAccess": true
  }
}
```

```typescript
interface StringMap {
  [key: string]: string;
}

const map: StringMap = {};
const value = map['hello'];
// value is string | undefined (not just string)
// value.toUpperCase(); // ❌ Error: Object is possibly 'undefined'
if (value) {
  value.toUpperCase(); // ✅ narrowed to string
}
```

---

**Question**: How would you implement utility types like `Partial` and `Pick` from scratch?

**Answer**: These are mapped types that iterate over keys. `Partial` makes each property optional, `Required` makes them required, `Readonly` adds `readonly`, `Pick` selects keys from a type, and `Record` creates an object type with specified keys and values.

```typescript
type MyPartial<T> = { [P in keyof T]?: T[P] };
type MyRequired<T> = { [P in keyof T]-?: T[P] };
type MyReadonly<T> = { readonly [P in keyof T]: T[P] };
type MyPick<T, K extends keyof T> = { [P in K]: T[P] };
type MyRecord<K extends keyof any, V> = { [P in K]: V };

interface Todo { title: string; desc: string; done: boolean; }
type PartialTodo = MyPartial<Todo>; // { title?: string; desc?: string; done?: boolean }
type PickTodo = MyPick<Todo, 'title' | 'done'>; // { title: string; done: boolean }
```

---

**Question**: How do `target` and `lib` settings affect compilation?

**Answer**: `target` determines the ECMAScript version of the output JavaScript (e.g., `ES5` vs `ES2022`), affecting syntax downleveling. `lib` specifies which environment type definitions to include (e.g., `DOM`, `ES2022.Promise`). Setting `target: "ES5"` without the proper `lib` may cause errors when using modern APIs.

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM"],
    "module": "ESNext"
  }
}
```

---

**Question**: What are project references and how do they help monorepos?

**Answer**: Project references allow splitting a TypeScript codebase into smaller projects that reference each other. They enable incremental builds, faster compilation, and clear dependency boundaries. Each sub-project has its own `tsconfig.json` with `composite: true` and `references` fields.

```json
// root tsconfig.json
{
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/utils" },
    { "path": "./packages/app" }
  ]
}
// Each sub-project tsconfig.json includes "composite": true
```

---

**Question**: What are the trade-offs between `enum` and union types?

**Answer**: Enums provide both runtime values and reverse mappings but produce runtime code and can be unpredictable with const enums. Union types (`type Status = 'active' | 'inactive'`) are simpler, tree-shakeable, and align better with TypeScript's structural typing philosophy. Prefer union types for simple string constants; use enums when you need numeric values or runtime iteration.

```typescript
// Enum - generates runtime code, supports reverse mapping
enum Status { Active = 'ACTIVE', Inactive = 'INACTIVE' }

// Union type - no runtime cost, simpler
type StatusUnion = 'active' | 'inactive';
```

---

**Question**: How do you configure strict mode in TypeScript?

**Answer**: Set `"strict": true` in `tsconfig.json`. This enables a suite of stricter checks: `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `noImplicitThis`, and `alwaysStrict`. For stricter type safety, also consider `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, and `noUnusedLocals`.

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

---

**Question**: How do you implement a type-safe event emitter?

**Answer**: Use a generic map type where keys are event names and values are payload types. The `emit` and `on` methods use mapped types and `keyof` to enforce correct payload types for each event.

```typescript
type EventMap = {
  userLoggedIn: { userId: string };
  dataLoaded: { data: unknown; timestamp: number };
  error: { code: number; message: string };
};

class TypedEmitter<T extends Record<string, unknown>> {
  private handlers = new Map<keyof T, Set<(...args: unknown[]) => void>>();

  on<K extends keyof T>(event: K, handler: (payload: T[K]) => void): void {
    if (!this.handlers.has(event)) this.handlers.set(event, new Set());
    this.handlers.get(event)!.add(handler as (...args: unknown[]) => void);
  }

  emit<K extends keyof T>(event: K, payload: T[K]): void {
    this.handlers.get(event)?.forEach(h => h(payload));
  }
}

const emitter = new TypedEmitter<EventMap>();
emitter.on('userLoggedIn', (payload) => console.log(payload.userId));
// emitter.on('userLoggedIn', (payload) => console.log(payload.data)); // ❌ Error
```

---

**Question**: How do you implement a builder pattern with TypeScript types?

**Answer**: Use a class with method chaining where each method returns `this`. For stricter type safety, use a generic type parameter with a "state" union that tracks which required fields have been set, preventing incomplete builds at compile time.

```typescript
class PizzaBuilder {
  private toppings: string[] = [];
  private size: 'small' | 'large' = 'small';

  addTopping(topping: string): this {
    this.toppings.push(topping);
    return this;
  }

  setSize(size: 'small' | 'large'): this {
    this.size = size;
    return this;
  }

  build(): { size: string; toppings: string[] } {
    return { size: this.size, toppings: this.toppings };
  }
}

const pizza = new PizzaBuilder()
  .setSize('large')
  .addTopping('cheese')
  .addTopping('pepperoni')
  .build();
```

---

**Question**: How do you create fluent APIs with generics?

**Answer**: Use generics to thread type information through chained method calls. Each method returns a new type that accumulates the effects of previous calls, enabling autocompletion and type safety across the chain.

```typescript
class QueryBuilder<T extends Record<string, unknown>> {
  private selects: (keyof T)[] = [];

  select<K extends keyof T>(key: K): QueryBuilder<T> {
    this.selects.push(key);
    return this;
  }

  execute(): Pick<T, typeof this.selects[number]> {
    // implementation
    return {} as any;
  }
}

interface User { id: number; name: string; email: string; }
const query = new QueryBuilder<User>()
  .select('id')
  .select('name')
  .execute();
// query has type Pick<User, 'id' | 'name'>
```

---

**Question**: How do you design type-safe API clients?

**Answer**: Use a generic client with an endpoint map that defines paths, request types, and response types. Method chaining or separate functions for HTTP verbs ensure the correct payload type is sent and the correct response type is returned for each endpoint.

```typescript
interface Endpoints {
  '/users': { GET: { response: { id: number; name: string }[] }; POST: { body: { name: string }; response: { id: number } } };
  '/users/:id': { GET: { response: { id: number; name: string } }; DELETE: { response: void } };
}

class ApiClient<T extends Record<string, any>> {
  get<K extends keyof T>(path: K): Promise<T[K]['GET']['response']> {
    return fetch(path as string).then(r => r.json());
  }

  post<K extends keyof T>(path: K, body: T[K]['POST']['body']): Promise<T[K]['POST']['response']> {
    return fetch(path as string, { method: 'POST', body: JSON.stringify(body) }).then(r => r.json());
  }
}

const client = new ApiClient<Endpoints>();
client.get('/users').then(users => users[0].name); // ✅ typed
```

---

**Question**: What are error handling patterns with typed errors?

**Answer**: Use discriminated unions for errors or a Result type that wraps success and failure states. Pattern matching on the discriminant ensures all error cases are handled. Custom error classes can extend `Error` for instanceof checks.

```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

function parseJson(input: string): Result<unknown> {
  try {
    return { success: true, data: JSON.parse(input) };
  } catch (e) {
    return { success: false, error: e as Error };
  }
}

const result = parseJson('{"valid": true}');
if (result.success) {
  console.log(result.data); // narrowed
} else {
  console.error(result.error.message);
}
```
