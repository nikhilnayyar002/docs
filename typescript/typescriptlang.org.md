https://www.typescriptlang.org/

- [1. The TypeScript Handbook](#1-the-typescript-handbook)
  - [1.1. The Basics](#11-the-basics)
  - [1.2. Everyday Types](#12-everyday-types)
  - [1.3. Narrowing](#13-narrowing)
  - [1.4. More on Functions](#14-more-on-functions)
  - [1.5. Object Types](#15-object-types)
  - [1.6. Generics](#16-generics)
  - [1.7. keyof Type Operator](#17-keyof-type-operator)
  - [1.8. typeof type operator](#18-typeof-type-operator)
  - [1.9. Indexed Access Types](#19-indexed-access-types)
  - [1.10. Conditional Types](#110-conditional-types)
  - [1.11. Mapped Types](#111-mapped-types)
  - [1.12. Template Literal Types](#112-template-literal-types)
  - [1.13. Classes](#113-classes)
  - [1.14. Modules](#114-modules)
- [2. Reference](#2-reference)
  - [2.1. Utility Types](#21-utility-types)
  - [2.2. Declaration Merging](#22-declaration-merging)
  - [2.3. Enums](#23-enums)
  - [2.4. Namespaces](#24-namespaces)
  - [2.5. Namespaces and Modules](#25-namespaces-and-modules)
  - [2.6. Triple-Slash Directives](#26-triple-slash-directives)
  - [2.7. Type Compatibility](#27-type-compatibility)
- [3. Modules Reference](#3-modules-reference)
  - [3.1. Modules - Theory](#31-modules---theory)
  - [3.2. Modules - Choosing Compiler Options](#32-modules---choosing-compiler-options)
  - [3.3. Modules - Reference](#33-modules---reference)
- [4. Declaration Files](#4-declaration-files)
  - [4.1. Declaration Reference](#41-declaration-reference)


# 1. The TypeScript Handbook

## 1.1. The Basics

**Strictness**

`noImplicitAny`, `strictNullChecks`

## 1.2. Everyday Types

**The primitives: `string , number , and boolean`**

**Arrays**

`number[]` or `Array<number>`

**Functions**

```ts
async function getFavoriteNumber(): Promise<number> {return 26;}
```

**Optional Properties**

Object types can also specify that some or all of their properties are optional. 

```ts
function printName(obj: { first: string; last?: string }) {}
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });
```

**Union Types**

```ts
function printId(id: number | string | string[]) {
  if (typeof id === "string") console.log(id.toUpperCase());
  else if (Array.isArray(id)) console.log(x.join(" and "));
  else console.log(id);
}
```

**Type Aliases & Interfaces**

```ts
type Point = {x: number;y: number;};
interface Point {x: number; y: number;}
```
the key distinction is that a type cannot be re-opened to add new properties vs an interface which is always extendable.

```ts
// Extending an interface
interface Bear extends Animal {honey: boolean;}
// Adding new fields to an existing interface
interface Bear {title: string;}
// Extending a type via intersections
type Bear = Animal & { honey: boolean; }
```

**Type Assertions**

```ts
// document.getElementById return some kind of HTMLElement
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
const a = expr as any as T;
```

**Literal Types**

```ts
let x: "hello" = "hello";
function printText(s: string, alignment: "left" | "right" | "center") {}
```

**Literal Inference**

```ts
declare function handleRequest(url: string, method: "GET" | "POST"): void;
 
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
```
> Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.

In the above example `req.method` is inferred to be `string`, not `"GET"`. There are two ways to work around this.

```ts
// Change 1:
const req = { url: "https://example.com", method: "GET" as "GET" };
// Change 2
handleRequest(req.url, req.method as "GET");
// Change 3
const req = { url: "https://example.com", method: "GET" } as const;
```
The `as const` suffix acts like `const` but for the type system, ensuring that all properties are assigned the literal type instead of a more general version like string or number.

**Non-null Assertion Operator (Postfix !)**

```ts
function liveDangerously(x?: number | null) {
  console.log(x!.toFixed());
}
```

Just like other type assertions, this doesn’t change the runtime behavior of your code, so it’s important to only use `!` when you know that the value can’t be `null` or `undefined`.

## 1.3. Narrowing

**`typeof` type guards**

In TypeScript, checking against the value returned by `typeof` is a type guard. It knows about some of its quirks in JavaScript.
```ts
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object")
    for (const s of strs) // error: 'strs' is possibly 'null'.
}
```
in JavaScript, `typeof null` is actually `"object"`. TypeScript lets us know that `strs` was only narrowed down to `string[] | null` instead of just `string[]`.

**Equality narrowing**

checking whether something `== null` actually not only checks whether it is specifically the value `null` - it also checks whether it’s potentially `undefined`. The same applies to `== undefined`

```ts
function multiplyValue(value: number | null | undefined) {
  // Remove both 'null' and 'undefined' from the type.
  if (container.value != null)
}
```

**The `in` operator narrowing**
```ts
type Fish = { swim: () => void };  type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };
function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) // animal: Fish | Human
}
```

**`instanceof` narrowing**
```ts
function logValue(x: Date | string) {
  if (x instanceof Date) console.log(x.toUTCString());
}
```

**Using type predicates**

To define a user-defined type guard, we simply need to define a function whose return type is a type predicate:

```ts
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
let pet = getSmallPet();
if (isFish(pet)) pet.swim(); // equivalently- if (isFish(pet) as Fish)
else pet.fly();
```
A predicate takes the form `parameterName is Type`, where `parameterName` must be the name of a parameter from the current function signature. TypeScript not only knows that `pet` is a `Fish` in the `if` branch; it also knows that in the `else` branch, you don’t have a `Fish`, so you must have a `Bird`.

classes can use [this is Type](https://www.typescriptlang.org/docs/handbook/2/classes.html#this-based-type-guards) to narrow their type.

**Assertion functions**

Types can also be narrowed using [Assertion functions](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions). `asserts condition` says that whatever gets passed into the `condition` parameter must be true if the `assert` returns
```ts
function yell(str) {
  assert(typeof str === "string");
  return str.toUppercase(); // error: Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
}
function assert(condition: any, msg?: string): asserts condition {
  if (!condition) throw new AssertionError(msg);
}
```

**Exhaustiveness checking**

When narrowing, you can reduce the options of a union to a point where you have removed all possibilities and have nothing left. In those cases, TypeScript will use a `never` type to represent a state which shouldn’t exist. The `never` type is assignable to every type; however, no type is assignable to `never` (except `never` itself). 

```ts
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.radius ** 2;
    case "square": return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```
Adding a new member to the `Shape` union, will cause a TypeScript error:
```ts
type Shape = Circle | Square | Triangle;
      ...
      const _exhaustiveCheck: never = shape; 
      // error: Type 'Triangle' is not assignable to type 'never'.
```

## 1.4. More on Functions

```ts
type GreetFunction = (a: string) => void;
```

**Call Signatures**

In JavaScript, functions can have properties in addition to being callable. If we want to describe something callable with properties, we can write a call signature in an object type:
```ts
type DescribableFunction = { description: string; (someArg: number): boolean; };
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}

function myFunc(someArg: number) {
  return someArg > 3;
}
myFunc.description = "default description";

doSomething(myFunc);
```

**Construct Signatures**

JavaScript functions can also be invoked with the `new` operator (constructors). 
```ts
type SomeConstructor = { new (s: string): SomeObject; };
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```
Some objects, like JavaScript’s `Date` object, can be called with or without `new`
```ts
interface CallOrConstruct { (n?: number): string; new (s: string): Date;}

function fn(ctor: CallOrConstruct) {
  console.log(ctor(10));
  console.log(new ctor("10"));
}
fn(Date);
```

**Generic Functions**

```ts
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}
// Parameter 'n' is of type 'string', 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));

function longest<Type extends { length: number }>(a: Type, b: Type) {
  return a.length >= b.length ? a : b;
}
```

**Optional Parameters**

```ts
function f(x?: number) {}
f(); // OK
f(10); // OK
```

**Function Overloads**

```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {...}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3); // error: No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.
```
first two signatures are called the overload signatures. Then, we wrote a function implementation with a compatible signature.

**Other Types to Know About**

- `void` represents the return value of functions which don’t return a value.
- `object` refers to any value that isn’t a primitive `(string, number, bigint, boolean, symbol, null, or undefined)`. This is different from the empty object type `{ }`, and also different from the global type `Object`.
- `unknown` is similar to the `any` type, but is safer because it’s not legal to do anything with an unknown value
  ```ts
  function f2(a: unknown) {
     a.b(); // error: 'a' is of type 'unknown'.
  }
  function safeParse(s: string): unknown {return JSON.parse(s);}
  const obj = safeParse(someRandomString);
  ```
- `never` represents values which are never observed. [(alse see Exhaustiveness checking)](#13-narrowing)
  
  ```ts
  function fail(msg: string): never { throw new Error(msg);}
  ```

**Rest Parameters and Arguments**

```ts
function multiply(n: number, ...m: number[]) {m.map((x) => n * x);}
```
TypeScript does not assume that arrays are immutable. This can lead to some surprising behavior:
```ts
// Inferred type is number[] -- "an array with zero or more numbers", not specifically two numbers
const args = [8, 5];
const angle = Math.atan2(...args); // error: A spread argument must either have a tuple type or be passed to a rest parameter.

//solution
const args = [8, 5] as const; // Inferred as 2-length tuple
```

**Parameter Destructuring**

```ts
function sum({ a, b, c }: { a: number; b: number; c: number }) {}
```

**Return type `void`**

Contextual typing with a return type of `void` does not force functions to not return something.

```ts
type voidFunc = () => void;
const f2: voidFunc = () => true;
const f3: voidFunc = function () {return true;};
const v2 = f2(); // v2 is void type
```
when a literal function definition has a `void` return type, that function must not return anything.

```ts
function f2(): void {return true;} // error
```

## 1.5. Object Types

**Property Modifiers**

```ts
interface SomeType {
  readonly prop: string; // readonly Properties
  [index: number]: string; // Index Signatures
}
```
[Note](https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures) that when using both `number` and `string` indexers, the type returned from a numeric indexer must be a subtype of the type returned from the string indexer. This is because when indexing with a `number`, JavaScript will actually convert that to a `string` before indexing into an object.

string index also enforce that all properties match their return type. This is because a string index declares that `obj.property` is also available as `obj["property"]`. However, properties of different types are acceptable if the index signature is a union of the property types:

```ts
interface NumberOrStringDictionary {
  [index: string]: number | string;
  length: number; name: string;
}
```

**Excess Property Checks**

Where and how an object is assigned a type can make a difference in the type system. 

```ts
interface SquareConfig { color?: string; width?: number; }
function createSquare(config: SquareConfig){}
let mySquare = createSquare({ colour: "red", width: 100 }); // error: Object literal may only specify known properties, but 'colour' does not exist
```

You could argue that this program is correctly typed, there’s no `color` property present, and the extra `colour` property is insignificant. Object literals get special treatment and undergo excess property checking when assigning them to other variables, or passing them as arguments. If an object literal has any properties that the “target type” doesn’t have, you’ll get an error.

```ts
// solution 1
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);

// solution 2
interface SquareConfig { ... [propName: string]: unknown;}

// solution 3
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

**Interface Extension vs. Intersection**

If interfaces are defined with the same name, TypeScript will attempt to merge them if the properties are compatible. If the properties are not compatible (i.e., they have the same property name but different types), TypeScript will raise an error. In the case of intersection types, properties with different types will be merged automatically (results in a `never` type) which may produce unexpected results.

**Generic Object Types**

```ts
interface Box { contents: unknown;}
let x: Box = { contents: "hello world"};
 
if (typeof x.contents === "string") x.contents.toLowerCase();
(x.contents as string).toLowerCase(); // type assertion
```
```ts
interface Box<Type> {contents: Type;}
type Box<Type> = {contents: Type;};
```

The `ReadonlyArray` Type
```ts
const roArray: ReadonlyArray<string> = ["red", "green", "blue"];
```

*Tuple Types* - is another sort of Array type that knows exactly how many elements it contains, and exactly which types it contains at specific positions. A tuple with a rest element has no set “length”

```ts
type StringNumberPair = [string, number];
type Either2dOr3d = [number, number, number?];
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
readonly [string, number]; // or [3, 4] as const;
```

## 1.6. Generics

```ts
function identity<Type>(arg: Type): Type {...}
let myIdentity: <Input>(arg: Input) => Input = identity;
let myIdentity: { <Type>(arg: Type): Type } = identity;

interface GenericIdentityFn<Type> {(arg: Type): Type;}
let myIdentity: GenericIdentityFn<number> = identity;
```
```ts
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}
```
When creating factories in TypeScript using generics, it is necessary to refer to class types by their constructor functions. 
```ts
class Animal {}
class Bee extends Animal {}
function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}
createInstance(Bee);
```

**Generic Parameter Defaults**

```ts
declare function create<T extends HTMLElement = HTMLDivElement, 
U extends HTMLElement[] = T[]>(element?: T,children?: U): Container<T, U>;
 
const div = create(); // Container<HTMLDivElement, HTMLDivElement[]>
```

## 1.7. keyof Type Operator

```ts
type Point = { x: number; y: number };
type P = keyof Point; // "x" | "y"
type Mapish = { [k: string]: boolean };
type M = keyof Mapish; // string | number as obj[0] is always the same as obj["0"]
```

## 1.8. typeof type operator

```ts
let n: typeof "hello"; // string
function f() {return { x: 10, y: 3 };}
type P = ReturnType<typeof f>; // { x: number; y: number; }
```

## 1.9. Indexed Access Types

```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"]; // number

// The indexing type is itself a type, so we can use unions, keyof, or other types entirely:
type I1 = Person["age" | "name"]; // string | number
type I2 = Person[keyof Person];  // string | number | boolean

const MyArray = [{ name: "Alice", age: 15 }];
type Person = typeof MyArray[number]; // { name: string; age: number; }
type Age = typeof MyArray[number]["age"]; // number
```

## 1.10. Conditional Types

`SomeType extends OtherType ? TrueType : FalseType;`

describing function overloads with conditional type -
```ts
interface IdLabel {}
interface NameLabel {}
type NameOrId<T extends number | string> = T extends number ? IdLabel : NameLabel;
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {}
```
```ts
// Conditional Type Constraints
type MessageOf<T extends { message: unknown }> = T["message"];
interface Email { message: string;}
type EmailMessageContents = MessageOf<Email>; // string
// default to something like never if a message property isn’t available
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

type Flatten<T> = T extends any[] ? T[number] : T;
type Str = Flatten<string[]>; // string
// Inferring Within Conditional Types using the infer keyword
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;

type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
  ? Return : never;
```

**Distributive Conditional Types**

```ts
type ToArray<Type> = Type extends any ? Type[] : never;
type StrArrOrNumArr = ToArray<string | number>; // string[] | number[]

// Typically, distributivity is the desired behavior. To avoid that behavior, you can surround each side of the extends keyword with square brackets.
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;
type ArrOfStrOrNum = ToArrayNonDist<string | number>; // (string | number)[]
```

## 1.11. Mapped Types

```ts
// take all the properties from the type Type and change their values to be a boolean.
type OptionsFlags<Type> = { [Property in keyof Type]: boolean; };
```

**Mapping Modifiers**

You can remove or add modifiers by prefixing with `-` or `+`.

```ts
// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};
type UnlockedAccount = CreateMutable<{ readonly id: string; }>;

// Removes 'optional' attributes from a type's properties
type Concrete<Type> = { [Property in keyof Type]-?: Type[Property]; };
type User = Concrete< { id?: string; }>;  // { id: string; }
```

**Key Remapping via `as`**

You can leverage features like *template literal types* to create new property names from prior ones:
```ts
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: 
    () => Type[Property]
};
type LazyPerson = Getters<{ name: string; }>; // { getName: () => string; }
```

You can filter out keys by producing `never` via a conditional type:
```ts
type RemoveKindField<Type> = {
    [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
};
```

You can map over arbitrary unions
```ts
type EventConfig<Events extends { kind: string }> = {
    [E in Events as E["kind"]]: (event: E) => void;
}
 
type SquareEvent = { kind: "square", x: number, y: number };
type CircleEvent = { kind: "circle", radius: number };
 
// { square: (event: SquareEvent) => void; circle: (event: CircleEvent) => void; }
type Config = EventConfig<SquareEvent | CircleEvent>;
```

**Further Exploration**

```ts
type ExtractPII<Type> = {
  [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false;
};
type DBFields = { id: { format: "." }; name: { pii: true }; };
type ObjectsNeedingGDPRDeletion = ExtractPII<DBFields>; // { id: false; name: true; }
```

## 1.12. Template Literal Types

```ts
type World = "world";
type Greeting = `hello ${World}`; // "hello world"

type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title";
// "welcome_email_id" | "email_heading_id" | "footer_title_id"
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
type Lang = "en" | "ja";
// "en_welcome_email_id" | "en_email_heading_id" | "en_footer_title_id" |  "ja_welcome_email_id" | "ja_email_heading_id" | "ja_footer_title_id"
type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`;
```

We generally recommend that people use ahead-of-time generation for large string unions, but this is useful in smaller cases.

Consider the case where a function (`makeWatchedObject`) adds a new function called `on()` to a passed object:

```js
const person = makeWatchedObject({ firstName: "Saoirse", age: 26});
// makeWatchedObject has added `on` to the anonymous Object
person.on("firstNameChanged", (newValue) => {});
```
```ts
type PropEventSource<Type> = {
    on(
      eventName: `${string & keyof Type}Changed`, 
      callback: (newValue: any) => void
    ): void;
};
declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;
```

Given change of a `firstName` (i.e. a `firstNameChanged` event), we should expect that the callback will receive an argument of type `string`.
```ts
type PropEventSource<Type> = {
    on<Key extends string & keyof Type>
        (
          eventName: `${Key}Changed`,
          callback: (newValue: Type[Key]) => void
        ): void;
};
```

**Intrinsic String Manipulation Types**

`Uppercase<StringType>, Lowercase<StringType>, Capitalize<StringType>, Uncapitalize<StringType>`

```ts
type ShoutyGreeting = Uppercase<"Hello, world">; //  "HELLO, WORLD"
type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`;
type MainID = ASCIICacheKey<"my_app">; // "ID-MY_APP"
```

## 1.13. Classes

**`implements` Clauses**

is only a check that the class can be treated as the interface type. It doesn’t change the type of the class or its methods at all. 
```ts
interface Checkable { check(name: string): boolean; }
class NameChecker implements Checkable { check(s:any) {} }
```

Overriding Methods: 
```ts
class Base { greet() {} }
class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) super.greet();
  }
}
```

[**`this` -based type guards**](https://www.typescriptlang.org/docs/handbook/2/classes.html#this-based-type-guards)

You can use `this is Type` in the return position for methods in classes and interfaces. When mixed with a type narrowing (e.g. `if` statements) the type of the target object would be narrowed to the specified `Type`.

**`abstract` Classes and Members**

```ts
abstract class Base { abstract getName(): string; printName() {} }
class Derived extends Base { getName() {return "world";} }
```

## 1.14. Modules

TypeScript has extended the `import` syntax with two concepts for declaring an import of a type:

- `import type`: statement which can only import type
  
  ```ts
  // @filename: animal.ts
  export type Cat = { breed: string; yearOfBirth: number };
  export type Dog = { breeds: string[]; yearOfBirth: number };
  // @filename: valid.ts
  import type { Cat, Dog } from "./animal.js";
  ```
- Inline `type` imports: allows for individual imports to be prefixed with `type`
  ```ts
  import { createCatName, type Cat, type Dog } from "./animal.js";
  ```

# 2. Reference

## 2.1. Utility Types

https://www.typescriptlang.org/docs/handbook/utility-types.html

## 2.2. Declaration Merging

**Merging Namespaces**

Non-exported members are only visible in the original (un-merged) namespace. This means that after merging, merged members that came from other declarations cannot see non-exported members.

**Merging Namespaces with Classes, Functions, and Enums**

Merging Namespaces with Classes gives the user a way of describing inner classes.

JavaScript practice of creating a function and then extending the function further by adding properties onto the function. TypeScript uses declaration merging to build up definitions like this in a type-safe way.

namespaces can be used to extend enums with static members

## 2.3. Enums

```ts
enum ShapeKind {Circle, Square};
enum Direction {Up = "UP"};
interface Circle {kind: ShapeKind.Circle;}
let c: Circle = {kind: ShapeKind.Circle};
type LogLevelStrings = keyof typeof ShapeKind; // "Circle" | "Square"
const num = ShapeKind["Circle"]; // 0
let nameOfA = ShapeKind[ShapeKind.Circle]; // "Circle"
// Note: string enum members do not get a reverse mapping generated at all
```

**Objects vs Enums**

you may not need an enum when an object with as const could suffice:
```ts
const ODirection = {Up: 0, ...} as const;
// It requires an extra line to pull out the values
type Direction = typeof ODirection[keyof typeof ODirection]; // 0|1|2|3
function run(dir: Direction) {}
run(ODirection.Right);
```

## 2.4. Namespaces

we want the interfaces and classes here to be visible outside the namespace, we preface them with `export`. As our application grows, we’ll want to split the code across multiple files to make it easier to maintain. Because there are dependencies between files, we’ll add reference tags to tell the compiler about the relationships between the files.

```ts
// Validation.ts
namespace Validation {
  export interface StringValidator {}
}
// LettersOnlyValidator.ts
/// <reference path="Validation.ts" />
namespace Validation {
  export class LettersOnlyValidator implements StringValidator {}
}
```

**Aliases**

you can simplify working with namespaces is to use `import q = x.y.z` to create shorter names for commonly-used objects
```ts
namespace Shapes {
  export namespace Polygons { export class Square {} }
}
import polygons = Shapes.Polygons;
let sq = new polygons.Square(); // Same as 'new Shapes.Polygons.Square()'
```

**Working with Other JavaScript Libraries**

We call declarations that don’t define an implementation **“ambient”**. Typically these are defined in `.d.ts` files. If you’re familiar with C/C++, you can think of these as .h files.

```ts
declare namespace D3 {
  export interface Selectors {...}
  export interface Event {...}
  export interface Base extends Selectors {event: Event;}
}
declare var d3: D3.Base;
```

## 2.5. Namespaces and Modules

**Needless Namespacing**

```ts
// shapes.ts
export namespace Shapes {
  export class Triangle {}
}
// shapeConsumer.ts
import * as shapes from "./shapes";
let t = new shapes.Shapes.Triangle(); // shapes.Shapes?
```
A key feature of modules in TypeScript is that two different modules will never contribute names to the same scope. Because the module file itself is already a logical grouping, and its top-level name is defined by the code that imports it, it’s unnecessary to use an additional module layer for exported objects.

## 2.6. Triple-Slash Directives

https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html

## 2.7. Type Compatibility

The basic rule for TypeScript’s structural type system is that `x` is compatible with `y` if `y` has at least the same members as `x`.

**Comparing two functions**

```ts
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;
y = x; // OK
x = y; // Error
```

Each parameter in `x` must have a corresponding parameter in `y` with a compatible type. The second assignment is an error, because y has a required second parameter that `x` does not have, so the assignment is disallowed. ignoring extra function parameters is actually quite common in JavaScript. For example, `Array#forEach` provides three parameters to the callback function. it’s very useful to provide a callback that only uses the first parameter

```ts
let x = () => ({ name: "Alice" });
let y = () => ({ name: "Alice", location: "Seattle" });
x = y; // OK
y = x; // Error, because x() lacks a location property
```

[**Advanced Topics**](https://www.typescriptlang.org/docs/handbook/type-compatibility.html#advanced-topics)

`any , unknown , object , void , undefined , null` , and `never` assignability

# 3. Modules Reference

https://www.typescriptlang.org/docs/handbook/modules/introduction.html


## 3.1. Modules - Theory

*host*—the system that ultimately consumes the output JavaScript (or raw TypeScript) to direct its module loading behavior, typically either a runtime (like Node.js) or bundler (like Webpack).

**Who is the host?**

- When the output code (whether produced by tsc or a third-party transpiler) is run directly in a runtime like Node.js, the runtime is the host.
- When there is no “output code” because a runtime consumes TypeScript files directly, the runtime is still the host.
- When a bundler consumes TypeScript inputs or outputs and produces a bundle, the bundler is the host, 

**The module output format**

what kinds of modules the host expects, so TypeScript can set its output format for each file to match. Sometimes, the host only supports one kind of module—ESM in the browser, or CJS in Node.js v11 and earlier, for example. Node.js v12 and later accepts both CJS and ES modules, but uses file extensions and `package.json` files to determine what format each file should be, and throws an error if the file’s contents don’t match the expected format.

The `module` compiler option provides this information to the compiler. Its primary purpose is to control the module format of any JavaScript that gets emitted during compilation, but it also serves to inform the compiler about how the module kind of each file should be detected, how different module kinds are allowed to import each other, and whether features like `import.meta` and top-level `await` are available.

...

**Module format detection**

- .`mjs` and `.cjs` files are always interpreted as ES modules and CJS modules, respectively. 
- `.js` files are interpreted as ES modules if the nearest `package.json` file contains a `type` field with the value `"module"`. If there is no `package.json` file, or if the `type` field is missing or has any other value, `.js` files are interpreted as CJS modules.

When the input file extension is `.mts or .cts`, TypeScript knows to treat that file as an ES module or CJS module, respectively, because Node.js will treat the output `.mjs` file as an ES module or the output `.cjs` file as a CJS module. When the input file extension is `.ts`, TypeScript has to consult the nearest `package.json` file to determine the module format, because this is what Node.js will do when it encounters the output `.js` file. 

The detected module format of input files is used by TypeScript to ensure it emits the output syntax that Node.js expects in each output file. If TypeScript were to emit /example.js with import and export statements in it, Node.js would crash when parsing the file. 

It’s worth mentioning again that TypeScript’s behavior in --module node16, --module node18, and --module nodenext is entirely motivated by Node.js’s behavior. Since TypeScript’s goal is to catch potential runtime errors at compile time, it needs a very accurate model of what will happen at runtime.

...


**Module resolution**

```ts
import sayHello from "greetings";
sayHello("world");
```

We know that the input syntax looks like ESM, but the output format depends on the `module` compiler option, potentially the file extension, and `package.json "type"` field. Just as module informs the compiler about the host’s expected module format, `moduleResolution`, along with a few customization options, specify the algorithm the host uses to resolve module specifiers to files. 

## 3.2. Modules - Choosing Compiler Options

A single tsconfig.json can only represent a single environment. If your app contains server code, DOM code, web worker code, test code, and code to be shared by all of those, each of those should have its own tsconfig.json, connected with [project references](https://www.typescriptlang.org/docs/handbook/project-references.html). Then, use this guide once for each tsconfig.json.

https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html

## 3.3. Modules - Reference

**`import()` types**

```ts
// Access an exported type:
type WriteFileOptions = import("fs").WriteFileOptions;
// Access the type of an exported value:
type WriteFileFunction = typeof import("fs").writeFile;
```

**`export =` and `import = require()`**

...

**Ambient modules**

TypeScript supports a syntax in script (non-module) files for declaring a module 

```ts
declare module "path" {
  export function normalize(p: string): string; ...
}

/// <referen  ce path="path.d.ts" />
import { normalize, join } from "path";
```

Ambient modules may use imports inside the module declaration body to refer to other modules without turning the containing file into a module

```ts
declare module "m" {
  // Moving this outside "m" would totally change the meaning of the file!
  import { SomeType } from "other";
  export function f(): SomeType;
}
```

A pattern ambient module contains a single * wildcard character in its name

```ts
declare module "*.html" {
  const content: string;
  export default content;
}
```

**The `module` compiler option**

...

**The `moduleResolution` compiler option**

...

paths -

```json
{
  "compilerOptions": {
    "module": "nodenext",
    "paths": {
      "https://esm.sh/lodash@4.17.21": ["./node_modules/@types/lodash/index.d.ts"],
      "@app/*": ["./src/*"],
      "*": ["./vendor/*", "./types/*"]
    }
  }
}
```
The paths option does not change the import path in the code emitted by TypeScript without those users setting up the same aliases for both TypeScript and their bundler. Both libraries and apps can consider `package.json "imports"` as a standard replacement for convenience paths aliases.

When `baseUrl` is provided, the values in each paths array are resolved relative to the `baseUrl`. Otherwise, they are resolved relative to the `tsconfig.json` file that defines them.

...

# 4. Declaration Files

## 4.1. Declaration Reference

```ts
let result = myLib.makeGreeting("hello, world");
console.log("The computed greeting is:" + result);
let count = myLib.numberOfGreetings;

// Declaration
declare namespace myLib {
  function makeGreeting(s: string): string;
  let numberOfGreetings: number;
}

// Overloaded Functions
declare function getWidget(n: number): Widget;
declare function getWidget(s: string): Widget[];

declare class Greeter {
  constructor(greeting: string);
  greeting: string;
  showGreeting(): void;
}

declare var foo: number; // is read-only, use 'declare const', 'declare let' if block-scoped.

declare function greet(greeting: string): void;
```

**Publishing**

...

**Consumption**

...
