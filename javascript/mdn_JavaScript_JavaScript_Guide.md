https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide

- [1. Grammar and types](#1-grammar-and-types)
- [2. Control flow and error handling](#2-control-flow-and-error-handling)
- [3. Functions](#3-functions)
- [4. Expressions and operators](#4-expressions-and-operators)
- [5. Numbers and strings](#5-numbers-and-strings)
- [6. Regular expressions](#6-regular-expressions)
- [7. Indexed collections](#7-indexed-collections)
- [8. Keyed collections](#8-keyed-collections)
- [9. Working with objects](#9-working-with-objects)
- [10. Using classes](#10-using-classes)
- [11. Using promises](#11-using-promises)
- [12. Iterators and generators](#12-iterators-and-generators)
- [13. Internationalization](#13-internationalization)
- [14. JavaScript modules](#14-javascript-modules)
- [15. JavaScript language overview](#15-javascript-language-overview)
- [16. JavaScript data types and data structures](#16-javascript-data-types-and-data-structures)
- [17. Enumerability and ownership of properties](#17-enumerability-and-ownership-of-properties)
- [18. Closures](#18-closures)
- [19. Inheritance and the prototype chain](#19-inheritance-and-the-prototype-chain)


# 1. Grammar and types

**Variable scope**

- Global scope: The default scope for all code running in script mode.
- Module scope: The scope for code running in module mode.
- Function scope: The scope created with a function.

In addition, variables declared with `let` or `const` can belong to an additional scope:

- Block scope: The scope created with a pair of curly braces (a block).

However, variables created with `var` are not block-scoped, but only local to the function (or global scope) that the block resides within.

**Variable hoisting**

```js
console.log(x === undefined); // true (global scope)
var x = 3;

(function () {
  console.log(x); // undefined (local function scope)
  var x = "local value";
})();
```

**Global variables**

In web pages, the global object is `window`. . In all environments, the `globalThis` variable may be used to read and set global variables. you can access global variables declared in one `window` or `frame` from another window or frame by specifying the window or frame name.

**Numbers and the '+' operator**

```js
// + operator
x = "The answer is " + 42; // "The answer is 42"
z = "37" + 7; // "377"

// other operators
"37" - 7; // 30
"37" * 7; // 259
```

**Converting strings to numbers**

`parseInt(), parseFloat(), Number()`

```js
parseInt("101", 2); // 5
+"1.1" + (+"1.1"); // 2.2
```

**Array literals**

```js
const fish = ["Lion", , "Angel"]; // [ 'Lion', <1 empty item>, 'Angel' ]
```

When using array-traversing methods like `Array.prototype.map`, empty slots are skipped. However, index-accessing `fish[1]` still returns `undefined`.

**Integer literals**
```
0, 117, 123456789123456789n             (decimal, base 10)
015, 0001, 0o777777777777n              (octal, base 8)
0x1123, 0x00111, 0x123456789ABCDEFn     (hexadecimal, "hex" or base 16)
0b11, 0b0011, 0b11101001010101010101n   (binary, base 2)
``` 

A trailing `n` suffix on an integer literal indicates a `BigInt` literal

**Floating-point literals**

`3.141, .123456789, 3.1E+12, .1e-23`

**Object literals**

```js
const obj = {
  __proto__: theProtoObj,
  handler,
  toString() {return `d ${super.toString()}`;},
  ["prop_" + (() => 42)()]: 42,
};
```

**[String literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types#string_literals)**

> [My Note](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#string_primitives_and_string_objects) where a method is to be invoked on a primitive string or a property lookup occurs, JavaScript will automatically wrap the string primitive
> ```js
> const strPrim = "foo"; // A literal is a string primitive
> const strPrim2 = String(1); // string primitive "1"
> const strPrim3 = String(true); //string primitive "true"
> const strObj = new String(strPrim); // String with new returns a string wrapper object.
> console.log(typeof strPrim); // "string"
> console.log(typeof strObj); // "object"
> ```

[Tagged templates](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#tagged_templates) are a compact syntax for specifying a template literal along with a call to a "tag" function for parsing it -

...

Using special characters in strings -

...

You can also escape line breaks by preceding them with backslash

```js
const str = "this string \
is broken";
```

# 2. Control flow and error handling

> Note: 
> ```js
> var x = 1;
> { var x = 2; }
> console.log(x); // 2
> ```

**Falsy values**

`false undefined null 0 NaN ""`

```js
const b = new Boolean(false);
if (b) {} // true
```

# 3. Functions

**Function hoisting**

Function hoisting only works with function declarations — not with function expressions. 

**Closures**

A closure is any piece of source code (most commonly, a function) that refers to some variables, and the closure "remembers" these variables even when the scope in which these variables were declared has exited.

**Using the arguments object**

```js
arguments[i];
arguments.length;
```

**Rest parameters**
```js
function multiply(multiplier, ...theArgs) {
  return theArgs.map((x) => multiplier * x);
}
```

**Arrow functions**

Until arrow functions, every new function defined its own `this` value (a new object in the case of a constructor, undefined in strict mode function calls, the base object if the function is called as an "object method", etc.). This proved to be less than ideal with an object-oriented style of programming.

```js
function Person() {
  // The Person() constructor defines `this` as itself.
  this.age = 0;
  setInterval(function growUp() {
    // In nonstrict mode, the growUp() `this` is global object
    this.age++;
  }, 1000);
}
const p = new Person();
```
solutions to fix -
```js
  const self = this; self.age = 0; ... self.age++; ...
```
or use `Function.prototype.bind()`. An arrow function does not have its own `this`; the `this` value of the enclosing execution context is used. 
```js
  setInterval(() => this.age++, 1000);
```

> [My Notes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) 
> - Arrow functions don't have their own bindings to `this`, `arguments`, or `super`, and should not be used as **methods**.
> - Arrow functions cannot be used as **constructors**. Calling them with `new` throws a `TypeError`. They also don't have access to the `new.target` keyword.
> - Arrow functions cannot use `yield` within their body and cannot be created as generator functions.
>
>  should only be used for non-method functions because they do not have their own `this`
> ```js
> const obj = {
>   i: 10,
>   b: () => console.log(this.i, this),
>   c() {console.log(this.i, this)},
> };
> obj.b(); // logs undefined, Window { /* … */ } (or the global object)
> obj.c(); // logs 10, Object { /* … */ }
> ```
> **class**'s body has a `this` context, arrow functions as [class fields](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Public_class_fields) close over the class's `this` context, and the `this` inside the arrow function's body will correctly point to the instance (or the class itself, for static fields). However, because it is a **closure**, not the function's own binding, the value of this will not change based on the execution context
> ```js
> class C {
>   a = 1;
>   autoBoundMethod = () => console.log(this.a);
> }
> const c = new C();
> c.autoBoundMethod(); // 1
> const { autoBoundMethod } = c;
> autoBoundMethod(); // 1, If it were a normal method, it should be undefined
> ```
> Arrow function properties are often said to be "auto-bound methods", because the equivalent with normal methods is:
> ```js
> class C {
>   a = 1;
>   constructor() {this.method = this.method.bind(this);}
>   method() {console.log(this.a);}
> }
>```
> Note: Class fields are defined on the instance, not on the prototype, so every instance creation would create a new function reference and allocate a new closure, potentially leading to more memory usage than a normal unbound method.

# 4. Expressions and operators

...

**Destructuring**

> [My Note](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring)
> ```js
> const [one, two] = ["one", "two"];
> const [one = "one", two] = [undefined, "two"];
> const { a, b, c } ={a,b,c};
> const { a: x, b: y, c: z } ={a,b,c}; // const x = a, y = b, z = c;
> const numbers = [];
> ({ a: numbers[0], b: numbers[1] } = { a: 1, b: 2 });
> ```
...

**Logical operators** 

...

[Nullish coalescing operator (??)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing) - `expr1 ?? expr2` - Returns `expr1` if it is neither `null` nor `undefined`; otherwise, returns `expr2`.

```js
const a5 = "Cat" && "Dog"; // t && t returns Dog
const o6 = false || "Cat"; // f || t returns Cat
const n3 = false ?? 3; // false
```

**Unary operators**

```js
delete Math.PI; // returns false (cannot delete non-configurable properties)
const myObj = { h: 4 };
delete myObj.h; // returns true (can delete user-defined properties)
typeof var; // returns "function" | "string" | "number" | "object" | "undefined" | "symbol" | "boolean"
typeof null; // returns "object"
typeof window.length; // returns "number"
typeof parseInt; // returns "function"
typeof Date; // returns "function"
typeof Math; // returns "object"
typeof String; // returns "function"
```

**Relational operators**

[in operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in) returns true if the specified property is in the specified object

```js
const trees = ["redwood", "bay", "cedar", "oak", "maple"];
0 in trees; // returns true
"bay" in trees; // returns false
"PI" in Math; // returns true
```
```js
const obj = new Map();
if (obj instanceof Map) {}
```

**[Optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)**

```js
maybeObject?.property;
maybeObject?.[property];
maybeFunction?.();

const adventurer = { cat: { name: "Dinah" }};
adventurer.dog?.name; // undefined
adventurer.someNonExistentMethod?.(); // undefined
```

# 5. Numbers and strings

In addition to being able to represent floating-point numbers, the number type has three symbolic values: [Infinity](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Infinity), `-Infinity`, and `NaN`.

**Numeric separators**

```
1_000_000_000_000
1_050.95
```

**Number object**

```js
const biggestNum = Number.MAX_VALUE;
const smallestNum = Number.MIN_VALUE;
const infiniteNum = Number.POSITIVE_INFINITY;
const negInfiniteNum = Number.NEGATIVE_INFINITY;
const notANum = Number.NaN;
...
```

**Math object**

...

**BigInts**

...

# 6. Regular expressions


The following pages provide lists of the different special characters that fit into each category, along with descriptions and examples -

- [Assertions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions/Assertions) include boundaries, which indicate the beginnings and endings of lines and words, and other patterns indicating in some way that a match is possible (including look-ahead, look-behind, and conditional expressions).
- [Character](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions/Character_classes) Distinguish different types of characters. For example, distinguishing between letters and digits.
- [Groups and backreferences](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions/Groups_and_backreferences) Groups group multiple patterns as a whole, and capturing groups provide extra submatch information when using a regular expression pattern to match against a string. Backreferences refer to a previously captured group in the same regular expression.
- [Quantifiers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions/Quantifiers) Indicate numbers of characters or expressions to match.

> Note: [A larger cheat sheet is also available]((https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions/Cheatsheet))

**Escaping**

If you need to use any of the special characters escape it by putting a backslash in front of it (` /a\*b/`).
> Note:  you can wrap it in a character class as an alternative to escaping, for example `/a[*]b/`.

to search for the string "/example/" - `/\/example\//`, to match the string `"C:\"` where "C" can be any letter, you'd use `/[A-Z]:\\/`.

`/a\*b/ `and `new RegExp("a\\*b")` create the same expression.

The `RegExp.escape()` function returns a new string where all special characters in regex syntax are escaped (`RegExp(RegExp.escape("a*b"))`).

**Using regular expressions in JavaScript**

Regular expressions are used with the `RegExp` methods `test()` and `exec()` and with the `String` methods `match(), matchAll(), replace(), replaceAll(), search(), and split()`.

**[Advanced searching with flags](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions#advanced_searching_with_flags)**

...

```js
const re = /pattern/flags;
const re = new RegExp("pattern", "flags");
```

`RegExp.prototype.exec()` method with the `g` flag returns each match and its position iteratively. `String.prototype.match()` method returns all matches at once, but without their position.

# 7. Indexed collections

This includes arrays and array-like constructs such as `Array` objects and [TypedArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) objects.

**[Array methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections#array_methods)**

...

**Working with array-like objects**

```js
function printArguments() {
  Array.prototype.forEach.call(arguments, (item) => console.log(item));
}
Array.prototype.forEach.call("a string", (chr) => console.log(chr));
```

# 8. Keyed collections

**Map object**

See also the [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) reference page for more examples and the complete API

```js
const sayings = new Map();
sayings.set("dog", "woof");
sayings.set("cat", "meow");
sayings.set("elephant", "toot");
sayings.size; // 3
sayings.get("dog"); // woof
sayings.get("fox"); // undefined
sayings.delete("dog");
sayings.has("dog"); // false

for (const [key, value] of sayings)
  console.log(`${key} goes ${value}`); // "cat goes meow", "elephant goes toot"

sayings.clear();
sayings.size; // 0
```

**Object and Map compared**

- The keys of an `Object` are `strings` or `symbols`, whereas they can be of any value for a `Map`.
- The iteration of maps is in insertion order of the elements.
- An Object has a prototype (bypassed using `Object.create(null)`.)

These three tips can help you to decide whether to use a Map or an Object:

- Use maps when keys are unknown, all keys are the same type, all values are the same type.
- Use maps if there is a need to store primitive values as keys (object treats each key as a string)
- Use objects when there is logic that operates on individual elements.

**Sets**

[Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) objects are collections of unique values. You can iterate its elements in insertion order.

**Converting between Array and Set**

```js
Array.from(mySet);
[...mySet2];

mySet2 = new Set([1, 2, 3, 4]);
```

**Array and Set compared**

- Deleting Array elements by value (`arr.splice(arr.indexOf(val), 1)`) is very slow.
- Set objects store unique values.

# 9. Working with objects

**Creating new objects**

- Using object initializers
- Using a constructor function
- Using the `Object.create()` method

**Objects and properties**

An object property name can be any JavaScript string or [symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)

Enumerating properties -

- `for...in` traverses enumerable properties of an object as well as its prototype chain.
- `Object.keys()`. returns an array with only the enumerable own string property names

```js
delete myObj.a;
console.log("a" in myObj); // false
```

**Defining getters and setters**

```js
const myObj = {
  a: 7,
  get b() { return this.a + 1;},
  set c(x) {this.a = x / 2;},
};
console.log(myObj.b); myObj.c = 50; 
```

# 10. Using classes

go through [this](../learn_web/mdn_Learn_web_development.md#313-classes-in-javascript)

```js
class MyClass {
  constructor(v) {
    this.bob = v; // Instance field
    // Constructor body
  } 
  myField = "foo"; // Instance field
  myMethod() {} // Instance method
  static myStaticField = "bar"; // Static field
  static myStaticMethod() {} // Static method
  // Fields, methods, static fields, and static methods all have "private" forms
  #myPrivateField = "bar";
}

// pre-es6
function MyClass(v) {
  this.bob = v; this.myField = "foo";
  // Constructor body
}
MyClass.prototype.myMethod = function () {};
MyClass.myStaticField = "bar";
MyClass.myStaticMethod = function () {};
```

**Private fields**

A class method can read the private fields of other instances, as long as they belong to the same class.

**Accessor fields**

```js
class Color {
  constructor(r, g, b) { this.values = [r, g, b];}
  get red() {return this.values[0];}
  set red(value) {this.values[0] = value;}
}
const red = new Color(255, 0, 0);
red.red = 0;
console.log(red.red); // 0
```

# 11. Using promises

If one of the promises in the array rejects,` Promise.all()` immediately rejects the returned promise. The other operations continue to run, but their outcomes are not available via the return value of `Promise.all()`

**Task queues vs. microtasks**

Promise callbacks are handled as a [microtask](./mdn_JavaScript_Reference.md#1-javascript-execution-model) whereas setTimeout() callbacks are handled as task queues.
Also see [youtube](https://www.youtube.com/watch?v=8aGhZQkoFbQ)

```js
const promise = new Promise((resolve, reject) => {
  console.log("Promise callback");
  resolve();
}).then((result) => console.log("Promise callback (.then)"));

setTimeout(() => console.log("event-loop cycle: Promise (fulfilled)", promise), 0);

console.log("Promise (pending)", promise);

// Promise callback
// Promise (pending) Promise {<pending>}
// Promise callback (.then)
// event-loop cycle: Promise (fulfilled) Promise {<fulfilled>}
```

# 12. Iterators and generators

**Iterators**

an iterator is any object which implements the [Iterator protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterator_protocol) by having a `next()` method that returns an object with two properties:

`value` The next value in the iteration sequence.

`done` This is true if the last value in the sequence has already been consumed. If value is present alongside done, it is the iterator's return value.

```js
function makeRangeIterator(start = 0, end = Infinity, step = 1) {
  let nextIndex = start;
  let iterationCount = 0;

  const rangeIterator = {
    next() {
      let result;
      if (nextIndex < end) {
        result = { value: nextIndex, done: false };
        nextIndex += step;
        iterationCount++;
        return result;
      }
      return { value: iterationCount, done: true };
    },
  };
  return rangeIterator;
}
```

**Generator functions**

Generator functions do not initially execute their code. Instead, they return a special type of iterator, called a *Generator*. When a value is consumed by calling the generator's `next` method, the Generator function executes until it encounters the `yield` keyword.

The function can be called as many times as desired, and returns a new Generator each time. Each Generator may only be iterated once.

```js
function* makeRangeIterator(start = 0, end = Infinity, step = 1) {
  let iterationCount = 0;
  for (let i = start; i < end; i += step) {
    iterationCount++;
    yield i;
  }
  return iterationCount;
}
```

**Iterables**

In order to be iterable, an object must implement the [Symbol.iterator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator) method. 

Iterables which can iterate only once (such as Generators) customarily return `this` from their `[Symbol.iterator]()` method, whereas iterables which can be iterated many times must return a new iterator on each invocation of `[Symbol.iterator]()`.

```js
function* makeIterator() {
  yield 1; yield 2;
}
const iter = makeIterator();
for (const itItem of iter) console.log(itItem);

console.log(iter[Symbol.iterator]() === iter); // true

// If we change the [Symbol.iterator]() method of `iter` to a function/generator
// which returns a new iterator/generator object, `iter` can iterate many times
iter[Symbol.iterator] = function* () {
  yield 2; yield 1;
};
```

iterables can be defined directly inside a class or object using a computed property:

```js
const iter = {
  *[Symbol.iterator]() { yield 1; yield 2; yield 3;}
};

for (const itItem of iter) console.log(itItem);
[...iter]; // [1, 2, 3]
```

`String, Array, TypedArray, Map and Set` are all built-in iterables, because their prototype objects all have a Symbol.iterator method.

Syntaxes expecting iterables - Some statements and expressions expect iterables. For example: the for...of loops, spread syntax, yield*, and destructuring syntax.

```js
for (const value of ["a", "b", "c"]) console.log(value);

[..."abc"]; // ["a", "b", "c"]

function* gen() { yield* ["a", "b", "c"];}
gen().next(); // { value: "a", done: false }

[a, b, c] = new Set(["a", "b", "c"]);
a;// "a"
```

**Advanced generators**

The `next()` method also accepts a value, which can be used to modify the internal state of the generator.

> Note: A value passed to the first invocation of next() is always ignored.

```js
function* fibonacci() {
  let current = 0; let next = 1;
  console.log("fibonacci 1");
  while (true) {
    console.log("fibonacci 2");
    const reset = yield current;
    [current, next] = [next, next + current];
    if (reset) { current = 0; next = 1;}
    console.log("fibonacci 3");
  }
}

const sequence = fibonacci();
console.log(sequence.next().value); // fibonacci 1 fibonacci 2 0
console.log(sequence.next().value); // fibonacci 3 fibonacci 2 1
console.log(sequence.next(true).value); // fibonacci 3 fibonacci 2 0
```

> [My Notes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) When the generator's `next()` method is called, the generator function's body is executed until one of the following: 
> - A `yield` expression. In this case, the `next()` method returns an object with a value property containing the yielded value and a `done` property that is always `false`. The next time `next()` is called, the `yield` expression evaluates to the value passed to `next()`.
> - A `yield*`, delegating to another iterator. In this case, this call and any future calls to `next()` on the generator is the same as calling `next()` on the delegated iterator
> - A `return` statement (that is not intercepted by a `try...catch...finally`), or the end of the control flow which implicitly means `return undefined`. In this case, the generator is finished, and the `next()` method returns an object with a `value` property containing the returned value and a `done` property that is always `true`. Any further `next()` calls have no effect and always return `{ value: undefined, done: true }`.
> - An error thrown inside the function, either via a `throw` statement or an unhandled exception. The `next()` method throws that error, and the generator is finished. Any further `next()` calls  return `{ value: undefined, done: true }`.

> [My Notes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/return) The `return()` method of `Generator` instances acts as if a `return` statement is inserted in the generator's body at the current suspended position.

Also see [Iteration_protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)

> My Note
>
> `throw()`, acts as if a `throw` statement was spawned inside the generator body at the exact `yield` line. It terminates the generator unless you catch it with a `catch` block.
> ```js
> function* standardGenerator() {
>   try {
>     console.log("Starting...");
>     yield "Paused waiting for next() or throw()"; 
>     console.log("This line will NEVER run because an error is injected above!");
>   } catch (error) {
>     console.log("Caught the injected error:", error.message);
>     yield "Recovered and paused again";
>   }
> }
> 
> const iterator = standardGenerator();
> // 1. Start the generator. It pauses at the first yield.
> console.log(iterator.next().value); 
> // 2. Instead of calling next(), inject an error into the paused position
> console.log(iterator.throw(new Error("External Interruption!")).value);
> 
> // Starting...
> // Paused waiting for next() or throw()
> // Caught the injected error: External Interruption!
> // Recovered and paused again
> ```
> `return()` acts as if a `return` statement was spawned inside the generator body at the exact `yield` line. It terminates the generator immediately unless you have a `finally` block, which will run its code first before returning.
> ```js
> function* cleanupGenerator() {
>   try {
>     console.log("Resource opened...");
>     yield "Working with resource...";
>   } finally {
>     console.log("Cleaning up resources before closing!");
>   }
>   console.log("This line will never run because the function returned.");
> }
> 
> const geo = cleanupGenerator();
> geo.next(); 
> // 2. Forcefully inject a return statement into the paused generator
> const result = geo.return("Aborted");
> console.log("Generator state:", result);
> 
> // Resource opened...
> // Cleaning up resources before closing!
> // Generator state: { value: 'Aborted', done: true }
> ```


# 13. Internationalization

...

# 14. JavaScript modules

```js
export const name = "square";
export function draw () {}
export { name, draw};
import { name, draw } from "./modules/square.js";
import { name as squareName, draw } from "./shapes/square.js";
```

**Importing modules using import maps**

...

benifits i see - 
- Remapping module paths like `"lodash": "/node_modules/lodash-es/lodash.js",`
- Scoped modules for version management
- Improve caching by mapping away hashed filenames

**Loading non-JavaScript resources**

...

**Applying the module to your HTML**

`<script type="module" src="main.js"></script>`

- There is no need to use the defer attribute
- Module-defined variables are scoped to the module unless explicitly attached to the global object.

**Default exports versus named exports**

default export — this is designed to make it easy to have a default function provided by a module, and also helps JavaScript modules to interoperate with existing CommonJS and AMD module systems (as explained nicely in [ES6 In Depth: Modules](https://hacks.mozilla.org/2015/08/es6-in-depth-modules/))

```js
export default randomSquare;
import randomSquare from "./modules/square.js";
import { default as randomSquare } from "./modules/square.js";
```

**Renaming imports and exports**

```js
import { function1 as newFunctionName} from "./modules/module.js";
// or 
export { function1 as newFunctionName };
```

**Creating a module object**

```js
import * as Module from "./modules/module.js";
```

This grabs all the exports available inside module.js, and makes them available as members of an object Module, effectively giving it its own namespace

**Aggregating modules**

```js
export * from "x.js";
export { name } from "x.js";
```

**Dynamic module loading**

```js
import("./modules/myModule.js").then((module) => {});
```

**Top level await**

Top level await is a feature available within modules. code can be evaluated before use in parent modules, but without blocking sibling modules from loading.

```js
const colors = fetch("...").then((response) => response.json());
export default await colors;
```

**Cyclic imports**

Cyclic imports don't always fail. The imported variable's value is only retrieved when the variable is actually used (hence allowing [live bindings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#imported_values_can_only_be_modified_by_the_exporter)), and only if the variable remains uninitialized at that time will a [ReferenceError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Cant_access_lexical_declaration_before_init) be thrown.

```js
// -- a.js --
import { b } from "./b.js";
setTimeout(() => console.log(b), 10); // 1
export const a = 2;
// -- b.js --
import { a } from "./a.js";
setTimeout(() => console.log(a), 10); // 2
export const b = 1;
```
at the time the module is evaluated, neither `b` nor `a` is actually read, so the rest of the code is executed as normal, and the two export declarations produce the values of `a` and `b`. Similarly -

```js
// -- a.js (entry module) --
import { b } from "./b.js";
console.log(b); // 1
export const a = 2;
// -- b.js --
import { a } from "./a.js";
setTimeout(() => console.log(a), 10);  // 2
export const b = 1;
```

**Authoring "isomorphic" modules**

make the code "isomorphic" — that is, exhibits the same behavior in every runtime (`browser & node`). 

- Separate your modules into "core" and "binding". For the "core", focus on pure JavaScript logic like computing the hash, without any DOM, network, filesystem access, and expose utility functions. the binding, which is usually lightweight, needs to be platform-specific.
- `typeof window === "undefined"`
- Use a polyfill to provide a fallback for missing features.
  ```js
  if (typeof fetch === "undefined") globalThis.fetch = (await import("node-fetch")).default;
  ```
  The `globalThis` variable is a global object that is available in every environment 

# 15. JavaScript language overview

**Numbers**

The Number type is a [IEEE 754 64-bit double-precision floating point value](https://en.wikipedia.org/wiki/Double_precision_floating-point_format), which means integers can be safely represented between [-(2^53 − 1)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MIN_SAFE_INTEGER) and [2^53 − 1](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER) without loss of precision. loating point numbers can be stored all the way up to [1.79 × 10^308](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_VALUE)

```js
console.log(3 / 2); // 1.5, not 1
console.log(0.1 + 0.2); // 0.30000000000000004
```

an apparent integer is in fact implicitly a float. Because of IEEE 754 encoding, sometimes floating point arithmetic can be imprecise. ([See](https://stackoverflow.com/questions/588004/is-floating-point-math-broken/63231948#63231948))

**Strings**

Strings in JavaScript are sequences of Unicode characters ([UTF-16 encoded](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#utf-16_characters_unicode_code_points_and_grapheme_clusters)). Strings have [utility methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#instance_methods) to manipulate the string and access information about the string. Because all primitives are immutable by design, these methods return new strings.

Also [see strings use immutable](https://stackoverflow.com/questions/51185/are-javascript-strings-immutable-do-i-need-a-string-builder-in-javascript)

**Variables**

```js
function foo(x, condition) {
  if (condition) {
    console.log(x);
    const x = 2;
    console.log(x);
  }
}
foo(1, true);
```
because each declaration occupies the entire scope, this would throw an error on the first `console.log`

# 16. JavaScript data types and data structures

**Primitive coercion**

- The `+` operator — if one operand is a string, string concatenation is performed; otherwise, numeric addition is performed.
- The `==` operator — if one operand is a primitive while the other is an object, the object is converted to a primitive value with no preferred type.

Objects are converted to primitives by calling its `[Symbol.toPrimitive]()`, `valueOf()`, and `toString()` methods, in that order.

```js
console.log({} + []); // '[object Object]' + '' = "[object Object]"
```
Neither `{}` nor `[]` have a `[Symbol.toPrimitive]()` method. Both inherit `valueOf()` which returns the object itself. Since the return value is an object, it is ignored. Therefore, `toString()` is called instead.

# 17. [Enumerability and ownership of properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Enumerability_and_ownership_of_properties)

**Querying object properties**

...

**Traversing object properties**

...

# 18. Closures

**Emulating private methods with closures**

```js
function makeCounter() {
  let privateCounter = 0;
  return {
    increment() {changeBy(1); },
    value() {return privateCounter;}
  };
}
```

**Creating closures in loops: A common mistake**

```js
  for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = function () {
      showHelp(item.help);
    };
  }
```
No matter what field you focus on, the message about your age will be displayed. functions assigned to onfocus form closures. Three closures have been created by the loop, but each one shares the same single lexical environment, which has a variable with changing values (`item`). `item` is declared with `var` and thus has function scope due to hoisting.

**Performance considerations**

when creating a new object/class, methods should normally be associated to the object's prototype rather than defined into the object constructor. The reason is that whenever the constructor is called, the methods would get reassigned (that is, for every object creation).

```js
function MyObject(name, message) {
  this.name = name.toString();
  this.getName = function () {return this.name;};
}
```

# 19. Inheritance and the prototype chain

> Note: Following the ECMAScript standard, the notation `someObject.[[Prototype]]` is used to designate the prototype of `someObject`. `obj.__proto__` should not be confused with the `func.prototype` property of functions, which instead specifies the `[[Prototype]]` to be assigned to all instances of objects created by the given function when used as a constructor. 

In an object literal like `{ a: 1, b: 2, __proto__: c }`, the value `c` will become the `[[Prototype]]`.

**Constructors**

a constructor function, which automatically sets the `[[Prototype]]` for every object manufactured. Constructors are functions called with `new`.

```js
function Box(value) {this.value = value;}
// all boxes created from the Box() constructor will have
Box.prototype.getValue = function () {return this.value;};
const boxes = [new Box(1), new Box(2), new Box(3)];
```

`Constructor.prototype` is only useful when constructing instances. It has nothing to do with `Constructor.[[Prototype]]`, which is the constructor function's own prototype that is, `Object.getPrototypeOf(Constructor) === Function.prototype`.

`Object.getPrototypeOf(Constructor.prototype) === Object.prototype`

**Implicit constructors of literals**

```js
// Object literals (without the `__proto__` key) automatically have `Object.prototype` as their `[[Prototype]]`
const object = { a: 1 };
Object.getPrototypeOf(object) === Object.prototype; // true
const array = [1, 2, 3]; // have `Array.prototype`
const regexp = /abc/; // have `RegExp.prototype`
```

