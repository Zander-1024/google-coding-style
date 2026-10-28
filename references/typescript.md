# Google TypeScript Style Guide

This is an **editorially condensed adaptation** of Google's TypeScript Style Guide, pinned to the source revision in “Sources and licensing”. It preserves the guide's executable rules, their requirement levels, and key exceptions while shortening explanations and examples. The original guide is based on Google's internal guide with internal sections removed. Google's environment has different constraints; this advice is particularly useful for code intended for import into Google and may not apply in other environments. The upstream public guide is updated on demand by volunteers.

## Introduction

### Terminology notes

This guide uses [RFC 2119](https://tools.ietf.org/html/rfc2119): *must*, *must not*, *should*, *should not*, and *may*. *Prefer* means *should*; *avoid* means *should not*. Imperative and declarative statements are prescriptive and mean *must*.

### Guide notes

All examples are **non-normative** illustrations. They do not show the only acceptable implementation; optional formatting in an example must not be enforced as a rule. The pinned original is available for optional further explanation, not as a prerequisite to using this adaptation.

## Source file basics

### File encoding: UTF-8

Encode source files in **UTF-8**.

#### Whitespace characters

Except for line terminators, ASCII horizontal space (0x20) is the only whitespace character permitted anywhere in source. Escape other whitespace inside string literals.

#### Special escape sequences

Use a character's special escape sequence (`\'`, `\"`, `\\`, `\b`, `\f`, `\n`, `\r`, `\t`, `\v`) rather than a numeric escape such as `\x0a`, `\u000a` or `\u{a}`. Never use legacy octal escapes.

#### Non-ASCII characters

Use actual Unicode characters for remaining non-ASCII characters. For non-printable characters, hex or Unicode escapes may be used with an explanatory comment.

~~~typescript
// Good: readable Unicode and an explained non-printable character.
const units = 'μs';
const output = '\ufeff' + content;  // byte order mark

// Bad: unnecessary escape, or an unexplained invisible character.
const obscureUnits = '\u03bcs';
const obscureOutput = '\ufeff' + content;
~~~

## Source file structure

Put these sections in order, omitting absent sections, with **exactly one blank line** between sections:

1. Copyright information.
2. `@fileoverview` JSDoc.
3. Imports.
4. Implementation.

### Copyright information

Put necessary license or copyright information in a JSDoc at the top of the file.

### `@fileoverview` JSDoc

A top-level `@fileoverview` JSDoc is optional. It may describe content, uses or dependencies. Do not indent wrapped description lines.

~~~typescript
/**
 * @fileoverview Description of the file, its uses, or dependencies.
 * Wrapped description lines are not indented.
 */
~~~

### Imports

| Import form | Example | Use |
| --- | --- | --- |
| Namespace (also called module) | `import * as foo from '...';` | TypeScript imports |
| Named (also called destructuring) | `import {SomeThing} from '...';` | TypeScript imports |
| Default | `import SomeThing from '...';` | Only external code that requires it |
| Side effect | `import '...';` | Only libraries needed for load-time side effects, such as custom elements |

#### Import paths

TypeScript code *must* use paths to import other TypeScript code. Paths *may* be relative (`.` or `..`) or rooted at a base directory (`root/path/to/file`). For files in the same logical project, code *should* use relative imports so the project can move without rewriting them. Consider limiting parent steps such as `../../../`.

#### Namespace versus named imports

Both forms are allowed. Prefer named imports for frequently used symbols or self-explanatory names such as `describe` and `it`; `as` aliases may clarify names. Prefer namespace imports for many symbols from a large API or common names such as `Model`. A namespace import names the module's exports; it is not a wildcard import in the sense used by other languages.

~~~typescript
// Good: namespace avoids repeating an artificial prefix.
import * as tableview from './tableview';
let item: tableview.Item|undefined;

// Good: these function names already convey their purpose.
import {describe, it, expect} from './testing';

// Avoid a long list of aliases such as Item as TableviewItem,
// Header as TableviewHeader, Model as TableviewModel.
// Avoid unnecessary testing.describe(), testing.it(), etc.
~~~

##### Special case: Apps JSPB protos

Apps JSPB protos *must* use named imports, even if the line is long. Import only the needed symbols, e.g. `import {Foo, Bar} from './foo.proto';`. This enables finer-grained build dependencies and dead code elimination.

#### Renaming imports

Code *should* resolve name collisions through namespace imports or by renaming exports. It *may* rename imports when needed, particularly for collisions, generated symbol names, or names unclear on their own (e.g. `from as observableFrom`).

### Exports

Use named exports in all code. Do not use default exports. Prefer file scope for namespacing rather than putting unrelated items in a class.

~~~typescript
// Good:
export const SOME_CONSTANT = 1;
export function someHelpfulFunction() {}
export class Foo {}

// Bad: no canonical exported name.
export default class Bar {}
~~~

#### Export visibility

Export only symbols used outside the module; generally minimize the exported API. TypeScript cannot otherwise restrict an exported symbol's visibility.

#### Mutable exports

Mutable exports (`export let`) are not allowed. For externally accessible mutable bindings, code *should* use explicit getter functions. When conditionally exporting one of several values, choose first and export afterward; all exports must be final when the module body finishes.

~~~typescript
let current = 3;
export function getCurrent() { return current; }
export const SomeApi = useOtherApi() ? OtherApi : RegularApi;
~~~

#### Container classes

Do not create classes with static methods or properties solely to namespace them. Export individual constants and functions instead.

### Import and export type

#### Import type

You *may* use `import type` for symbols used only as types; use regular imports for values. Both separate and inline type imports are allowed:

~~~typescript
import type {Foo} from './foo';
import {Bar} from './foo';
// Alternatively:
import {type Foo, Bar} from './foo';
~~~

Type-only imports support fast, file-by-file transpilation without full type information; full type checking validates their use. To force a runtime load for side effects, use `import '...';`.

#### Export type

Use `export type` when re-exporting a type, e.g. `export type {AnInterface} from './foo';`. This supports file-by-file transpilation. It does not prevent downstream consumers from importing a value through another path; split type and value APIs into distinct symbols (e.g. `UserService` and `AjaxUserService`) when that separation needs a guarantee.

#### Use modules not namespaces

Code *must* reference other files with ES6 imports and exports. It *must not* use `namespace Foo {...}`, old-style internal `module Foo {...}`, `<reference>` directives, or `require` imports such as `import x = require('mydep');`. Namespaces *may* be used only when required to interface with external third-party code. Otherwise use separate files to namespace code.

## Language features

Features not discussed here *may* be used without a style recommendation.

### Local variable declarations

#### Use const and let

Always declare variables with `const` or `let`; default to `const` unless reassignment is needed. Never use `var`. Variables *must not* be used before declaration.

#### One variable per declaration

Declare one variable per local declaration. Do not write `let a = 1, b = 2;`.

### Array literals

#### Do not use the `Array` constructor

Do not call `Array()` with or without `new`. Use array literals, or `Array.from` for a sized array. To create holes, a literal's `length` can be assigned.

~~~typescript
const values = [2, 3];
const holes: number[] = [];
holes.length = 2;
const zeroes = Array.from<number>({length: 5}).fill(0);

// Bad: argument count changes the meaning.
const ambiguous = new Array(2);
~~~

#### Do not define properties on arrays

Do not define or use non-numeric array properties other than `length`. Use a `Map` or object instead.

#### Using spread syntax

Array spread may shallow-copy or concatenate iterables: `const combined = [5, ...values, 6];`. The spread value *must* match the collection being created: only iterables for arrays. Do not spread primitives, including `null` and `undefined`.

~~~typescript
// Good:
const extras = shouldUseFoo ? [7] : [];
const result = [5, ...extras];
// Bad: may spread a primitive.
const broken = [5, ...(shouldUseFoo && [7])];
~~~

#### Array destructuring

Array destructuring *may* unpack arrays or iterables in assignments and parameters. A final rest element is allowed; do not put a space after `...`. Unused elements *should* be omitted.

~~~typescript
const [a, b, ...rest] = generateResults();
const [, second,, fourth] = someArray;
~~~

Optional destructured array parameters must default to `[]`, with element defaults on the left-hand side:

~~~typescript
// Good:
function destructured([a = 4, b = 2] = []) {}
// Bad:
function badDestructuring([a, b] = [4, 2]) {}
~~~

For multiple function parameters or return values, prefer object over array destructuring when possible, to give elements meaningful names and distinct types.

### Object literals

#### Do not use the `Object` constructor

Use object literals (`{}` or `{a: 0, b: 1}`), never the `Object` constructor.

#### Iterating objects

Do not use unfiltered `for...in`: it includes enumerable prototype properties. Filter explicitly with an `if`, or use `for...of` with `Object.keys` or `Object.entries`.

~~~typescript
for (const key in someObj) {
  if (!someObj.hasOwnProperty(key)) continue;
  use(key, someObj[key]);
}
for (const [key, value] of Object.entries(someObj)) {
  use(key, value);
}
~~~

#### Using spread syntax

Object spread shallow-copies enumerable own properties; later values replace earlier values at the same key. Only objects *may* be spread into objects. Arrays and primitives (including `null` and `undefined`) *must not* be spread. Avoid spreading non-plain objects such as class definitions, instances and functions: their prototype properties are not copied.

~~~typescript
const foo = {num: 7};
const laterWins = {...foo, num: 5};  // num is 5.
const earlierLoses = {num: 5, ...foo};  // num is 7.
const optional = shouldUseFoo ? foo : {};
const merged = {num: 5, ...optional};

// Bad: may spread a primitive; arrays produce numbered keys, not an array.
const broken = {num: 5, ...(shouldUseFoo && foo)};
const wrongShape = {...['a', 'b']};
~~~

#### Computed property names

Computed names (`{['key' + foo()]: 42}`) are allowed and count as dict-style quoted keys; do not mix them with unquoted keys. Symbol keys such as `[Symbol.iterator]` are the exception.

#### Object destructuring

Object destructuring *may* unpack assignments. Destructured function parameters *should* be simple: one level of unquoted shorthand properties. Deeper nesting and computed properties are disallowed in parameter destructuring. Put property defaults on the left-hand side; an optional destructured object must default to `{}`.

~~~typescript
interface Options {
  num?: number;
  str?: string;
}
// Good:
function destructured({num, str = 'default'}: Options = {}) {}
// Bad:
function nested({x: {num, str}}: {x: Options}) {}
function nontrivialDefault({num, str}: Options = {num: 42, str: 'default'}) {}
~~~

### Classes

#### Class declarations

Class declarations *must not* end with semicolons; statements containing class expressions *must*. Blank lines just inside class declaration braces are neither encouraged nor discouraged.

~~~typescript
class Foo {}  // Good: no semicolon.
export const Baz = class extends Bar {
  method(): number { return this.x; }
};  // Good: statement ends with a semicolon.

// Bad: class Foo {};
~~~

#### Class method declarations

Do not separate method declarations with semicolons. Methods *should* have a single blank line separating them from surrounding code.

~~~typescript
class Foo {
  doThing() {
    work();
  }

  getOtherThing(): number {
    return 4;
  }
}
~~~

##### Overriding toString

`toString` *may* be overridden but must always succeed and have no visible side effects. Calls to other methods can cause infinite recursion in exceptional conditions.

#### Static methods

##### Avoid private static methods

Prefer module-local functions over private static methods when readability permits.

##### Do not rely on dynamic dispatch

Code *should not* rely on static-method dynamic dispatch. Static methods *should* be called only on the class that directly defines them. Calling through a variable holding a constructor or subclass constructor is discouraged; if done, the method *must* have `@nocollapse`. A method *must not* be called directly on a subclass that does not define it.

~~~typescript
class Base {
  /** @nocollapse */ static foo() {}
}
class Sub extends Base {}

// Discouraged:
function callFoo(cls: typeof Base) { cls.foo(); }
// Disallowed:
Sub.foo();
~~~

##### Avoid static `this` references

Code *must not* use `this` in static contexts. Refer to the declaring class explicitly; static fields are inherited and can be overridden, making `this` surprising.

#### Constructors

Constructor calls *must* include parentheses, even with no arguments: `new Foo()`, not `new Foo`. Note that `new Foo().Bar()` differs from `new Foo.Bar()`.

Empty constructors and constructors that only delegate to a parent are unnecessary. However, constructors with parameter properties, visibility modifiers or parameter decorators *should not* be omitted even with empty bodies:

~~~typescript
class ParameterProperties {
  constructor(private myService: MyService) {}
}
class ParameterDecorators {
  constructor(@SideEffectDecorator myService: MyService) {}
}
class NoInstantiation {
  private constructor() {}
}
~~~

Constructors *should* have a single blank line separating them from surrounding code above and below.

#### Class members

##### No #private fields

Do not use `#private` identifiers. Use TypeScript visibility annotations, e.g. `private ident = 1;`. This rule reflects Google's emit-size, performance and compilation constraints.

##### Use readonly

Mark properties never reassigned outside the constructor `readonly`. Deep immutability is not required.

##### Parameter properties

Use parameter properties instead of explicitly plumbing an obvious constructor parameter into a field:

~~~typescript
class Foo {
  constructor(private readonly barService: BarService) {}
}
~~~

Document a parameter property with an `@param` JSDoc tag when documentation is needed.

##### Field initializers

Initialize a member at its declaration if it is not a parameter. This can eliminate a constructor:

~~~typescript
class Foo {
  private readonly userList: string[] = [];
}
~~~

Properties *should* never be added or removed after construction. Optional fields populated later *should* be explicitly initialized to `undefined`, avoiding instance shape changes.

##### Properties used outside of class lexical scope

Properties used outside their class's lexical scope (including in templates) *must not* be `private`. Use `protected` or `public` as appropriate. Angular and AngularJS template properties *should* be `protected`; Polymer properties *should* be `public`.

Code *must not* bypass visibility with `obj['foo']`. Static analysis and build optimization rely on declared visibility.

##### Getters and setters

Class accessors *may* be used. Getters *must* be pure: consistent results and no changes to observable state. A hidden property's name *may* include a whole word such as `internal` or `wrapped`; access it through its accessor whenever possible.

At least one accessor for a property *must* be non-trivial. Do not add pass-through accessors solely to hide a property; make it public, or consider `readonly` instead of a getter-only wrapper. Do not define accessors with `Object.defineProperty`, which interferes with property renaming.

~~~typescript
class Foo {
  private wrappedBar = '';
  get bar() {
    return this.wrappedBar || 'bar';
  }
  set bar(value: string) {
    this.wrappedBar = value.trim();
  }
}
// Bad: get next() { return this.nextId++; }
// Bad: get bar() { return this.barInternal; } paired only with a trivial setter.
~~~

##### Computed properties

Classes may have computed properties only for symbol keys. Quoted or computed non-symbol (dict-style) keys are disallowed. Logically iterable classes *should* define `[Symbol.iterator]`; otherwise use `Symbol` sparingly. Other built-in symbols such as `Symbol.isConcatSpreadable` are not compiler-polyfilled and may not work in older browsers.

#### Visibility

Limit symbol visibility as much as possible. Consider moving private methods to non-exported file-local functions and private properties to a separate non-exported class.

Symbols default to public. Never write `public` except for non-readonly public constructor parameter properties:

~~~typescript
class Foo {
  bar = new Bar();
  constructor(public baz: Baz, readonly qux: Qux) {}
}
// Bad: public bar = new Bar();
// Bad: constructor(public readonly baz: Baz) {}
~~~

#### Disallowed class patterns

##### Do not manipulate `prototype`s directly

Ordinary implementation code must use `class` rather than manipulating prototypes. Mixins and modifying built-in prototypes are explicitly forbidden.

**Exception:** Framework code such as Polymer or Angular *may* need prototypes and *should not* use worse workarounds to avoid them.

### Functions

#### Terminology

A **function declaration** is a declaration using `function`; a **function expression** is an expression using `function`. An **arrow function** uses `=>`. Its **block body** has braces; its **concise body** is an expression without braces. This section does not cover methods or classes/constructors.

#### Prefer function declarations for named functions

Prefer declarations (`function foo() { return 42; }`) over arrows or function expressions for named functions. Arrows *may* be used, for example, when an explicit function type is required:

~~~typescript
interface SearchFunction {
  (source: string, subString: string): boolean;
}
const fooSearch: SearchFunction = (source, subString) => {
  return source.includes(subString);
};
~~~

#### Nested functions

Nested functions *may* use declarations or arrows as appropriate. Prefer arrows inside methods, where they capture the enclosing `this`.

#### Do not use function expressions

Use arrows instead of function expressions (`bar(() => { this.doSomething(); });` rather than `bar(function() {...});`).

**Exception:** Function expressions *may* be used only to dynamically rebind `this` (discouraged) or for generators, which have no arrow syntax.

#### Arrow function bodies

Use block or concise bodies as appropriate. Only use a concise body when its return value is used; otherwise use a block or explicit `void` to prevent leaking a return value. An annotation of `() => void` alone does not prevent that leak.

~~~typescript
const longThings = values.filter(v => v.length > 1000);
myPromise.then(v => {
  console.log(v);
});
myPromise.then(v => void console.log(v));  // Also good.

// Bad: return value is unused.
myPromise.then(v => console.log(v));
// Bad: still returns 1 at runtime.
let f: () => void;
f = () => 1;
~~~

Blocks *may* also be used for readability when returning a value.

#### Rebinding `this`

Function declarations and function expressions *must not* use `this` unless they exist specifically to rebind it. Prefer arrows or explicit parameters to avoid rebinding, and prefer arrows over `f.bind(this)`, `goog.bind(f, this)` or `const self = this`.

~~~typescript
// Good: the target is explicit.
document.body.onclick = () => {
  document.body.textContent = 'hello';
};
// Bad: implicit event-target binding.
function clickHandler() {
  this.textContent = 'Hello';
}
~~~

#### Prefer passing arrow functions as callbacks

Avoid passing named callbacks to higher-order functions unless both call signatures are known to be stable, including optional arguments. Prefer arrows that explicitly forward intended arguments. Locally defined functions designed as callbacks may be passed directly.

~~~typescript
// Bad: map passes indices as parseInt's radix.
const bad = ['11', '5', '10'].map(parseInt);
// Good: forwards only the intended argument.
// This illustrates callback forwarding; numeric parsing rules still apply.
const good = ['11', '5', '3'].map(n => parseInt(n));

function dayFilter(element: string|null|undefined) {
  return element != null && element.endsWith('day');
}
const days = ['tuesday', undefined, 'juice'].filter(dayFilter);
~~~

#### Arrow functions as properties

Classes usually *should not* have arrow-initialized properties. Code *should* invoke instance methods through arrows and *should not* obtain or pass bare instance-method references. Explicit call-time binding keeps the behavior understandable:

~~~typescript
setTimeout(() => {
  this.patienceTracker();
}, 5000);
// Avoid: setTimeout(this.patienceTracker, 5000);
~~~

Use judgment: arrow properties can improve readability in specific cases such as template bindings. Event handlers needing uninstallation are another exception.

#### Event handlers

Handlers *may* be anonymous arrows if they need no uninstallation (e.g. events emitted by the class itself). If uninstallation is required, use an arrow function property: it captures `this` and has a stable reference. Do not call `bind` in the installing expression; each call makes a new reference that cannot be removed by a subsequent `bind`.

~~~typescript
class Component {
  onAttached() {
    window.addEventListener('beforeunload', this.listener);
  }
  onDetached() {
    window.removeEventListener('beforeunload', this.listener);
  }
  private listener = () => {
    confirm('Do you want to exit the page?');
  };
}
// Bad: addEventListener(..., this.listener.bind(this));
// A later removeEventListener(..., this.listener.bind(this)) cannot remove it.
~~~

#### Parameter initializers

Optional parameters *may* have default initializers. Initializers *must not* have observable side effects and *should* be as simple as possible. Do not increment counters or expose shared mutable default state.

~~~typescript
function process(name: string, extraContext: string[] = []) {}
function activate(index = 0) {}
// Bad: function newId(index = globalCounter++) {}
~~~

Use defaults sparingly. Prefer object destructuring when more than a small handful of optional parameters lack a natural order.

#### Prefer rest and spread when appropriate

Use rest parameters instead of `arguments`. Never name local variables or parameters `arguments`. Use function spread instead of `Function.prototype.apply`.

~~~typescript
function variadic(array: string[], ...numbers: number[]) {}
variadic(array, ...numbers);
~~~

#### Formatting functions

Do not put blank lines at the start or end of function bodies. A single blank line *may* sparingly separate logical groups within a body. Generators *should* attach `*` to `function` and `yield`: `function* foo()`, `yield* iter`. Parentheses around a single arrow argument are recommended but optional. Do not put a space after rest/spread `...`.

### this

Use `this` only in class constructors and methods, functions declaring an explicit `this` type (`function func(this: ThisType, ...)`), or arrows within a scope where `this` is allowed.

Never use it for the global object, an `eval` context, an event target, or unnecessarily `call()`ed or `apply()`ed functions.

### Interfaces

The pinned source adds no separate rules under this heading; interface rules appear in “Type system”.

### Primitive literals

#### String literals

##### Use single quotes

Delimit ordinary strings with single quotes. If a string contains an apostrophe, consider a template literal to avoid escaping it.

##### No line continuations

Do not end a source line inside an ordinary or template string with a continuation backslash. Concatenate strings instead. A single long string is acceptable when breaking it would hinder search discoverability.

~~~typescript
const LONG_STRING = 'First part of a long string. ' +
    'Second part without indentation embedded in the text.';
const SINGLE_STRING =
    'http://it.is/acceptable_to_keep_a_long_string_for_search_discoverability';
~~~

##### Template literals

Use template literals over complex string concatenation, particularly with multiple string literals. They *may* span lines. Multiline content need not follow enclosing indentation; it may do so when added whitespace does not matter.

~~~typescript
function arithmetic(a: number, b: number) {
  return `Here are the results:
${a} + ${b} = ${a + b}
${a} - ${b} = ${a - b}`;
}
~~~

#### Number literals

Decimal, hexadecimal, octal and binary literals are allowed. Use lowercase prefixes `0x`, `0o` and `0b`. Never use a leading zero unless immediately followed by `x`, `o` or `b`.

#### Type coercion

Code *may* coerce with `String()`, `Boolean()` (without `new`), template literals or `!!`. String concatenation for coercion is discouraged.

Enum values, including unions with other types, *must not* be coerced with `Boolean()` or `!!`. Compare explicitly:

~~~typescript
let enabled = level !== SupportLevel.NONE;
enabled = maybeLevel !== undefined && maybeLevel !== SupportLevel.NONE;
// Bad: Boolean(level), !!maybeLevel.
~~~

Code *must* parse numeric values with `Number()` and explicitly check for `NaN` unless failure is impossible from context. Empty/whitespace strings become `0`; `'Infinity'` and exponential overflow can become infinities and may need handling. Do not use unary `+` for string-to-number coercion. Do not parse numbers with `parseInt` or `parseFloat`, except non-base-10 strings: before `parseInt` with a non-10 radix, code *must* check that the entire input contains only valid digits.

~~~typescript
const value = Number(someString);
if (isNaN(value)) handleError();
// Use isFinite instead when infinities are also invalid.

if (!/^[a-fA-F0-9]+$/.test(hexString)) throw new Error('Invalid hexadecimal');
// tslint:disable-next-line:ban
const hex = parseInt(hexString, 16);  // Allowed only for radix != 10.
~~~

For decimal integers, use `Number()`, validate it, then `Math.floor` or `Math.trunc` where available. `parseInt` and `parseFloat` can silently ignore trailing text.

##### Implicit coercion

Do not add explicit boolean coercions to `if`, `for` or `while` conditions, which already coerce: use `if (foo)` rather than `if (!!foo)`.

Enum values, including unions with other types, *must not* be implicitly coerced either. Compare them explicitly (`level !== SupportLevel.NONE`; check `undefined` separately when needed). Other values *may* be implicitly coerced or explicitly compared: both `if (arr.length)` and `if (arr.length > 0)` are allowed.

### Control structures

#### Control flow statements and blocks

Control statements (`if`, `else`, `for`, `do`, `while`, etc.) always use braced bodies, even for one statement. The first statement of a nonempty block must start on its own line.

**Exception:** An `if` fitting entirely on one line *may* omit braces: `if (x) x.doFoo();`.

##### Assignment in control statements

Prefer assignments outside conditions. If an assignment inside a control statement is preferable, surround it with extra parentheses to show intent:

~~~typescript
x = someFunction();
if (x) {
  use(x);
}
// Also allowed when intentional:
while ((x = someFunction())) {
  use(x);
}
~~~

##### Iterating containers

Prefer `for...of` for arrays. `Array.prototype.forEach` and conventional `for` loops are also allowed; `entries()` provides index and value.

`for...in` *may* be used only on dict-style objects, never arrays. It yields string indices, not array values. `Object.prototype.hasOwnProperty` *should* be used to exclude prototype properties. Prefer `for...of` with `Object.keys`, `Object.values` or `Object.entries` where possible.

~~~typescript
for (const value of array) {
  use(value);
}
for (const [index, value] of array.entries()) {
  use(index, value);
}
for (const key in obj) {
  if (!obj.hasOwnProperty(key)) continue;
  use(key, obj[key]);
}
~~~

#### Grouping parentheses

Omit optional grouping parentheses only if author and reviewer agree that the code cannot reasonably be misread and parentheses would not aid readability. Do not assume every reader memorizes precedence.

Do not put unnecessary parentheses around the whole expression after `delete`, `typeof`, `void`, `return`, `throw`, `case`, `in`, `of` or `yield`.

#### Exception handling

Exceptions *should* be used for exceptional cases; custom exceptions *should* be defined and used where native `Error` lacks needed information. Prefer exceptions over ad-hoc approaches such as error-container arguments or return objects with error properties.

##### Instantiate errors using `new`

Always instantiate with `new Error(...)`, not `Error(...)`.

##### Only throw errors

Only throw `Error` instances or subclasses. The same rule applies to promise rejection values, including `Promise.reject` and a promise executor's `reject` callback. Rejecting with a string, `undefined` or another non-error loses stack information. A promise with no rejection is fine.

~~~typescript
class MyError extends Error {}
throw new MyError('Failure');
Promise.reject(new Error('Failure'));
// Bad: throw 'Failure'; Promise.reject(); Promise.reject('Failure');
~~~

##### Catching and rethrowing

Code *should* assume caught values are `Error` instances. An assertion helper can narrow `unknown`:

~~~typescript
function assertIsError(e: unknown): asserts e is Error {
  if (!(e instanceof Error)) throw new Error('e is not an Error');
}
try {
  doSomething();
} catch (e: unknown) {
  assertIsError(e);
  displayError(e.message);
  throw e;  // Rethrow when appropriate.
}
~~~

Handlers *must not* defensively handle non-`Error` values unless the called API is conclusively known to throw them. In that case, a comment *should* identify the source, e.g. `// This external API throws strings.`.

##### Empty catch blocks

If doing nothing is appropriate, explain why in a comment. Do not test expected exceptions by placing `fail('expected an error')` inside the same `try`: the catch will also swallow that failure. Use `assertThrows()`.

~~~typescript
try {
  return handleNumericResponse(response);
} catch (e: unknown) {
  // Response is not numeric; continue as text.
}
return handleTextResponse(response);
~~~

#### Switch statements

Every `switch` *must* have a `default` statement group, even if empty, and it must be last. Nonempty groups *must not* fall through; terminate with `break`, `return` or `throw`. Empty groups may fall through.

~~~typescript
switch (x) {
  case X:
  case Y:
    doSomething();
    break;
  default:
    // Nothing to do.
}
~~~

#### Equality checks

Always use `===` and `!==` instead of `==` and `!=`.

**Exception:** Comparisons with literal `null` *may* use `==` or `!=` to cover both `null` and `undefined`: `if (foo == null) {...}`.

#### Type and non-nullability assertions

`x as SomeType` and `y!` silence the compiler without runtime checks. Code *should not* use them without an obvious or explicit reason. Prefer an actual runtime check (`instanceof`, null check, etc.).

If local reasoning makes an assertion safe, code *should* explain why in a comment. The comment *may* be omitted when the reason is obvious in context, such as a well-known backend guarantee for a generated nullable proto field.

~~~typescript
if (x instanceof Foo) {
  x.foo();
}
// x is a Foo because the caller validated its source.
(x as Foo).foo();
// y cannot be null because initialization completed above.
y!.bar();
~~~

##### Type assertion syntax

Assertions *must* use `as`, never angle brackets. Member access must be parenthesized: `(z as Foo).length` rather than `(<Foo>z).length` or `<Foo>z.length`.

##### Double assertions

If a justified assertion is between types that do not overlap sufficiently, cast through `unknown`: `(x as unknown as Foo).fooMethod();`. Use `unknown` as the intermediate type, never `any` or `{}`.

##### Type assertions and object literals

Use annotations (`: Foo`), not assertions (`as Foo`), for object literals; annotations detect missing or renamed fields at their declaration. Annotate function return types when needed for returned object literals.

~~~typescript
interface Foo {
  bar: number;
  baz?: string;
}
const foo: Foo = {
  bar: 123,
  // bam: 'abc',  // Would be caught here after a rename to baz.
};
function func(): Foo {
  return {bar: 123};
}
// Bad: const foo = {bar: 123, bam: 'abc'} as Foo;
~~~

#### Keep try blocks focused

Limit code inside a `try` when readability permits. Move non-throwing work outside so readers can see what throws. Trivial non-throwing inline calls may remain if a temporary variable adds undue complexity.

**Exception:** Widening a `try` around an entire loop is allowed to avoid performance issues from placing it inside the loop.

### Decorators

Do not define new decorators. Only use framework-defined decorators from Angular (such as `@Component` and `@NgModule`) or Polymer (such as `@property`). This restriction reflects the upstream guide's concerns about experimental decorators diverging from TC39 proposals.

A decorator *must* immediately precede the decorated symbol without blank lines. Field decorators may share a line with the field or appear on the preceding line. Put JSDoc before decorators.

~~~typescript
/** Component description. */
@Component({...})
class MyComp {
  @Input() myField: string;

  @Input()
  myOtherField: string;
}
~~~

### Disallowed features

#### Wrapper objects for primitive types

Code *must not* instantiate `String`, `Boolean` or `Number` wrappers. They may be called as functions for coercion without `new`; the coercion rules above still apply. Function calls for creating symbols are also allowed.

#### Automatic Semicolon Insertion

Do not rely on ASI. Explicitly terminate statements with semicolons.

#### Const enums

Code *must not* use `const enum`; use plain `enum`. Enums are already immutable; `const enum` can make them invisible to JavaScript consumers.

#### Debugger statements

Production code *must not* contain `debugger` statements.

#### `with`

Do not use `with`.

#### Dynamic code evaluation

Do not use `eval` or the `Function(...string)` constructor, except in code loaders. They also fail under strict Content Security Policies.

#### Non-standard features

Do not use non-standard ECMAScript or Web Platform features, including deprecated or removed features, non-standard transpiler extensions, TC39 proposals/current working-draft features not in the current ECMA-262 specification, and WHATWG proposals that have not completed the proposal process.

Projects targeting particular runtimes (e.g. latest Chrome, Chrome extensions, Node.js or Electron) may use those runtimes' APIs. Be cautious with proprietary APIs implemented in only some browsers; consider a common abstraction library.

#### Modifying builtin objects

Never add methods to built-in constructors or prototypes. Avoid libraries that modify built-ins. Do not add global symbols unless absolutely necessary, such as a third-party API requirement.

## Naming

### Identifiers

Identifiers *must* contain only ASCII letters, digits, underscores (constants and structured test names), and rarely `$`.

#### Naming style

Names *should not* encode information already in the type. Do not prefix/suffix private members with underscores or prefix optional parameters with `opt_`. Do not mark interfaces with `I` or `Interface` unless idiomatic in the environment; name the role, e.g. `TodoItemStorage` for serialization/storage of `TodoItem`.

Teams may choose the common `$` suffix for Observables if it clarifies observable versus concrete values, but *should* apply the choice consistently within projects. Hidden accessor storage may include a whole word such as `internal` or `wrapped`.

#### Descriptive names

Names *must* be descriptive and clear to a new reader. Do not use ambiguous abbreviations, unfamiliar project-only jargon, or abbreviate by deleting letters from words. Familiar abbreviations such as `dnsConnectionIndex`, `referrerUrl` and `customerId` are fine; `nErr`, `wgcConnections`, `cstmrId` and Hungarian-style `kSecondsPerDay` are not.

**Exception:** Variables scoped to ten lines or fewer, including arguments outside exported APIs, *may* have short names such as a single letter.

#### Camel case

Treat acronyms as words: `loadHttpUrl`, not `loadHTTPURL`. Preserve names required by a platform, e.g. `XMLHttpRequest`.

#### Dollar sign

Identifiers generally *should not* use `$` except for third-party framework conventions or the project-consistent Observable convention above.

### Rules by identifier type

Most identifiers *should* follow this table:

| Case | Categories |
| --- | --- |
| `UpperCamelCase` | Classes, interfaces, types, enums, decorators, type parameters, TSX component functions, JSXElement type parameter |
| `lowerCamelCase` | Variables, parameters, functions, methods, properties, module aliases |
| `CONSTANT_CASE` | Global constant values, including enum values; see “Constants” |
| `#ident` | Never used |

#### Type parameters

Type parameters *may* be a single uppercase letter (`T`) or `UpperCamelCase`.

#### Test names

xUnit-style test method names *may* use underscore-separated structure, e.g. `testX_whenY_doesZ()`.

#### `_` prefix/suffix

Do not prefix or suffix identifiers with `_`, or use `_` alone to indicate an unused parameter. Skip unused destructured array/tuple entries with commas: `const [a, , b] = [1, 5, 10];`.

#### Imports

Namespace imports use `lowerCamelCase` while filenames use `snake_case`: `import * as fooBar from './foo_bar';`.

The only exceptions for overwhelmingly common library conventions are jQuery (`$`) and three.js (`THREE`).

#### Constants

`CONSTANT_CASE` indicates an intent that users *must not* modify a value; it *may* be used even for technically mutable objects. Constants may be `static readonly` class properties.

Only module-level symbols, static fields of module-level classes, and module-level enum values *may* use `CONSTANT_CASE`. Values that can be instantiated more than once per program lifetime (e.g. function locals or statics of classes nested in functions) *must* use `lowerCamelCase`. An arrow function implementing an interface *may* use `lowerCamelCase`.

~~~typescript
const UNIT_SUFFIXES = {'milliseconds': 'ms', 'seconds': 's'};
class Foo {
  private static readonly MY_SPECIAL_NUMBER = 5;
}
~~~

#### Aliases

Local aliases *must* match the source identifier's naming and format. Use `const` for variable aliases and `readonly` for field aliases. Template-exposed aliases must also use appropriate visibility.

~~~typescript
const {BrewStateEnum} = SomeType;
const CAPACITY = 5;
class Teapot {
  readonly BrewStateEnum = BrewStateEnum;
  readonly CAPACITY = CAPACITY;
}
~~~

## Type system

### Type inference

Code *may* rely on compiler inference for variables, fields, returns and other type expressions. Omit annotations for trivially inferred variables/parameters initialized to string, number, boolean or RegExp literals or `new` expressions.

~~~typescript
const count = 15;  // Not: const count: number = 15;
const names = new Set<string>();  // Generic argument may be necessary.
~~~

Explicit generic arguments may be necessary when empty arrays, objects, Maps or Sets would infer `unknown`. For complex expressions, annotations may clarify a type (`const value: string[] = await rpc.getSomeValue().transform();`). The reviewer decides when such an annotation is required.

#### Return types

Return annotations are the author's choice. Reviewers *may* request them for complex types; projects *may* require them locally. There is no general requirement. Explicit annotations can document intent and expose future return-type changes sooner.

### Undefined and null

Both `undefined` and `null` may express absence; choose according to context, with no general preference. Many JavaScript APIs return `undefined`; DOM and Google APIs often use `null`. Write unions such as `string|null` and `string|undefined`; there is no special combined syntax.

#### Nullable/undefined type aliases

Type aliases *must not* include `|null` or `|undefined`. Add them only where the alias is used. Code *should* handle null values close to where they originate.

~~~typescript
type CoffeeResponse = Latte|Americano;
class CoffeeService {
  getLatte(): CoffeeResponse|undefined { ... }
}
// Bad: type CoffeeResponse = Latte|Americano|undefined;
~~~

#### Prefer optional over `|undefined`

Use optional fields and parameters (`?`) instead of `|undefined`. Optional parameters include `undefined` and can be omitted; optional fields can be absent. For classes, preferably avoid absence altogether by initializing as many fields as possible.

~~~typescript
interface CoffeeOrder {
  sugarCubes: number;
  milk?: Whole|LowFat|HalfHalf;
}
function pourCoffee(volume?: Milliliter) {}
class MyClass {
  field = '';
}
~~~

### Use structural types

A value structurally matches a type when it recursively has at least the required properties with matching types. When providing a structural implementation, explicitly annotate its declaration so contract errors are reported there rather than at a distant call site. Define structural types with interfaces, not classes.

~~~typescript
interface Animal {
  sound: string;
  name: string;
}
const dog: Animal = {
  sound: 'bark',
  name: 'MrPickles',
};
// const horse: Animal = {sound: 'neigh'};  // Error at declaration.
~~~

### Prefer interfaces over type literal aliases

For object types, use interfaces rather than aliases of object literal expressions: `interface User { firstName: string; lastName: string; }` rather than `type User = {...}`. Type aliases may name primitives, unions, tuples and other type expressions.

### `Array<T>` Type

For simple types containing only alphanumeric characters and dots, use `T[]` or `readonly T[]` rather than `Array<T>` or `ReadonlyArray<T>`. Use `T[][]` etc. for multidimensional non-readonly arrays of simple types. Use the long form for complex types. Apply these rules at every nesting level.

~~~typescript
// Good:
let a: string[];
let b: readonly string[];
let c: ns.MyObj[];
let d: string[][];
let e: Array<{n: number, s: string}>;
let f: Array<string|number>;
let g: ReadonlyArray<string|number>;
let h: InjectionToken<string[]>;
let i: ReadonlyArray<string[]>;
let j: Array<readonly string[]>;

// Bad:
// Array<string>, ReadonlyArray<string>, Array<ns.MyObj>, Array<string[]>
// {n: number, s: string}[], (string|number)[], readonly (string|number)[]
// InjectionToken<Array<string>>, readonly string[][], (readonly string[])[]
~~~

### Indexable types / index signatures (`{[key: string]: T}`)

Object associative arrays may use an index signature. Give its key a meaningful label: `{[userName: string]: number}`, not `{[key: string]: number}`.

Consider ES6 `Map` and `Set` instead: they express intent, avoid surprising object behavior and support non-string keys/values. `Record<Keys, ValueType>` describes a statically known key set, which differs from an associative array.

### Mapped and conditional types

Mapped and conditional types, including standard operators such as `Record`, `Partial`, `Readonly` and `Pick`, *may* be used with care. Always use the simplest construct that expresses the code; a little repetition is often cheaper than complex type expressions.

Such operators require readers to evaluate type expressions, especially across inferred types and file boundaries. Their inference/evaluation can change with compiler versions and hinder refactoring, reference search and code navigation. Prefer explicit properties, interface extension or a nested object when clearer:

~~~typescript
interface FoodPreferences {
  favoriteIcecream: string;
  favoriteChocolate: string;
}
interface User extends FoodPreferences {
  shoeSize: number;
}
// Often clearer than deriving FoodPreferences with
// Pick<User, 'favoriteIcecream'|'favoriteChocolate'>.
~~~

### `any` Type

Consider avoiding `any`: it bypasses property and assignment checking and can hide serious errors. Prefer a more specific type or `unknown`; if `any` is justified, suppress the lint warning and document why.

#### Providing a more specific type

Use an interface for data such as server JSON, a type alias for repeated unions, an inline object type for a complex return, or a generic for a library that works with arbitrary caller-supplied types. Generics may have constraints; avoid return-type-only generics.

~~~typescript
interface MyUserJson {
  name: string;
  email: string;
}
type MyType = number|string;
function getTwoThings(): {something: number, other: string} {
  return {something, other};
}
function nicestElement<T>(items: T[]): T { ... }
~~~

#### Using `unknown` over `any`

When expressing only that a value's type is unknown, use `unknown`. It accepts any value, including `null` and `undefined`, but requires narrowing with a type guard or an assertion before use; it does not allow arbitrary unchecked property access.

~~~typescript
const val: unknown = value;
// Bad: const danger: any = value; danger.whoops();
~~~

#### Suppressing `any` lint warnings

For legitimate `any` use, such as an intentionally incomplete test mock, add a lint suppression and explain why it is safe enough in that context.

~~~typescript
// This test needs only get(); a missing operation would fail obviously.
// This is an intentionally unsafe partial mock.
// tslint:disable-next-line:no-any
const mockBookService = ({get() { return mockBook; }} as any) as BookService;
~~~

### `{}` Type

`{}` is an empty-interface type accepting any non-nullish value, including primitives; it gives no usable properties. Google3 code *should not* use it for most purposes. Prefer:

- `unknown` for opaque values, including `null` and `undefined`.
- `Record<string, T>` for dictionaries with explicit value type `T` (possibly `unknown`).
- `object` for non-nullish objects and functions, excluding primitives but assuming no properties.

### Tuple types

Use a tuple instead of creating a generic `Pair` interface, e.g. `function splitInHalf(input: string): [string, string]`. Often meaningful named properties are clearer. If an interface is too heavyweight, use an inline object type such as `{host: string, port: number}`; object destructuring also provides tuple-like unpacking.

### Wrapper types

Do not use wrapper types `String`, `Boolean` or `Number`; always use primitive `string`, `boolean` and `number`. Avoid `Object`: `{}` accepts everything except `null`/`undefined`, and `object` further excludes primitives, including `symbol` and `bigint`. Never invoke wrapper types with `new`.

### Return type only generics

Avoid APIs with return-type-only generics. When using existing APIs of this kind, always specify their generic arguments explicitly.

## Toolchain requirements

These requirements reflect Google's toolchain and applicable local frameworks.

### TypeScript compiler

All TypeScript files must pass type checking with the standard toolchain.

#### @ts-ignore

Do not use `@ts-ignore`, `@ts-expect-error` or `@ts-nocheck`. Resolve the underlying error; the approaches for justified `any` use may help.

**Exception:** Unit tests *may* use `@ts-expect-error`, though generally *should not*: it suppresses every error on the affected line and can hide unrelated failures. When testing runtime handling of unchecked values, consider an expression-local cast to the expected type or `any` with an explanatory comment, or a documented lint suppression.

### Conformance

Google TypeScript uses conformance frameworks such as [tsetse](https://tsetse.info) and [tsec](https://github.com/google/tsec), for critical restrictions, security patterns and code quality. Google-style TypeScript must obey applicable global and framework-local conformance rules.

## Comments and documentation

#### JSDoc versus comments

Use `/** JSDoc */` for documentation that users of the code should read; tools consume it. Use `//` line comments for implementation details.

#### Multi-line comments

Indent multiline implementation comments with surrounding code. They *must* be consecutive `//` lines, not `/* ... */` blocks. Do not draw comment boxes with asterisks or other characters.

### JSDoc general form

JSDoc *must* be well formed. Short comments may fit on one line; when they wrap, `/**` and `*/` must occupy their own lines:

~~~typescript
/** Describes a short operation. */
function shortOperation() {}

/**
 * Describes a longer operation with normally wrapped text.
 * @param arg A number to operate on.
 */
function doSomething(arg: number) {}
~~~

### Markdown

Write JSDoc in Markdown; HTML *may* be used when necessary. Plain indented text does not create a rendered list:

~~~typescript
/**
 * Computes weight from:
 *
 * - items sent
 * - items received
 * - last timestamp
 */
~~~

### JSDoc tags

Google style permits a subset of JSDoc tags. Most tags must start their own line at its beginning; do not combine `@param left @param right` on one line.

### Line wrapping

Indent wrapped block-tag text four spaces. Aligning wrapped descriptions horizontally with the preceding description *may* be done but is discouraged. Do not indent wrapped `@desc` or `@fileoverview` descriptions.

~~~typescript
/**
 * @param foo A particularly long description that continues onto
 *     another line.
 * @return A result with another long description continuing onto
 *     another line.
 */
~~~

### Document all top-level exports of modules

Document top-level exports with JSDoc that informs users, rather than restating names. Code *should* also document non-obvious properties and methods, exported/public or otherwise, as judged by the reviewer.

**Exception:** Symbols exported only for tooling, such as `@NgModule` classes, do not require comments.

### Class comments

Class JSDoc *should* explain when and how to use the class and other considerations for correct use. A constructor's textual description *may* be omitted.

### Method and function comments

Method, parameter and return descriptions *may* be omitted when obvious from the remaining JSDoc, method name and type signature. Method descriptions start with a third-person verb phrase, as if preceded by “This method ...”, not an imperative command.

### Parameter property comments

A constructor parameter prefixed with `private`, `protected`, `public` or `readonly` declares and initializes an instance property. Document it with JSDoc `@param` so editors show the description at constructor calls and property accesses:

~~~typescript
class ParamProps {
  /**
   * @param percolator The percolator used for brewing.
   * @param beans The beans to brew.
   */
  constructor(
    private readonly percolator: Percolator,
    private readonly beans: CoffeeBean[]) {}
}
~~~

Ordinary fields receive their own JSDoc above the field.

### JSDoc type annotations

Do not duplicate TypeScript types in `@param` or `@return`. Do not use `@implements`, `@enum`, `@private`, `@override`, etc. where the corresponding TypeScript keyword already expresses the information.

### Make comments that actually add information

Avoid comments that repeat a name or type. Non-exported symbols may need no more than their name and type, but useful documentation is usually beneficial. `@param` and `@return` are required only when they add information; otherwise they *may* be omitted.

~~~typescript
/**
 * POSTs the request to start coffee brewing.
 * @param amountLitres The amount to brew. Must fit the pot size!
 */
brew(amountLitres: number, logger: Logger) { ... }
~~~

#### Comments when calling a function

Parameter-name comments *should* be used when the method name and argument value do not convey meaning. First consider an interface parameter and destructuring to improve call-site readability.

Comments precede the value and end the parameter name with `=`:

~~~typescript
someFunction(obviousParam, /* shouldRender= */ true, /* name= */ 'hello');
~~~

For consistency in an existing file, the legacy trailing style without `=` is acceptable: `someFunction(obviousParam, true /* shouldRender */, 'hello' /* name */);`.

### Place documentation prior to decorators

Put JSDoc before decorators, never between a decorator and its declaration:

~~~typescript
/** Component that prints "bar". */
@Component({selector: 'foo', template: 'bar'})
export class FooComponent {}
~~~

## Policies

### Consistency

For questions this specification leaves unresolved, follow the same file's existing style; if that is inconclusive, consider neighboring files.

New files *must* follow Google Style regardless of neighboring styles. When adding code to an existing non-Google-style file, reformatting first is recommended subject to the next section. If not reformatted, new code *should* be as consistent as possible with the file but *must not* violate this guide.

#### Reformatting existing code

Existing code need not all be updated as rules evolve: balance consistency against churn. Significant changes to a file are expected to bring it into Google Style. Keep opportunistic style fixes from obscuring a change's focus; move extensive unrelated style changes into a separate CL.

### Deprecation

Mark deprecated methods, classes and interfaces with `@deprecated` JSDoc. Include simple, clear directions for fixing call sites.

### Generated code: mostly exempt

Build-generated source need not follow Google Style, but generated identifiers referenced from hand-written code must follow naming rules. As a special exception, those identifiers may contain underscores to avoid conflicts.

#### Style guide goals

Prefer local decisions and leave rules out when a choice depends on context. Global rules are justified by these goals:

1. Avoid patterns known to cause problems, particularly for newcomers.
2. Keep projects consistent across superficial, equivalent variations, such as casing, assertion syntax and array notation.
3. Preserve long-term maintainability: autoformat code for automated changes, use a common compiler flag set, explicitly import dependencies (“strict deps”), and write tests so language/toolchain changes can be checked.
4. Keep reviewers focused on code quality. Automatable rules are useful; obscure cases and unlikely bugs may not justify a rule.

## Footnotes

### Footnote 1

Namespace imports are often called “module imports”.

### Footnote 2

Named imports are sometimes called “destructuring imports” because of their similar syntax.

## Sources and licensing

- Original title: **Google TypeScript Style Guide**, by Google and the upstream contributors.
- Pinned source: [google/styleguide — tsguide.html at 572c673d9058242c049fbc8248b17029726c7afa](https://github.com/google/styleguide/blob/572c673d9058242c049fbc8248b17029726c7afa/tsguide.html).
- Official published guide: [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html).
- Documentation: [Creative Commons Attribution 3.0 Unported (CC BY 3.0)](https://creativecommons.org/licenses/by/3.0/), attributed to Google and the upstream contributors.
- Code examples: [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0), as stated in the upstream README.
- Adaptation: **editorially condensed adaptation** from the pinned HTML source. Explanations, background and repeated examples were shortened; rules, requirement levels, key exceptions and necessary examples were retained. Heading structure is preserved. Examples remain non-normative. Page scripts, styles and generated navigation were removed; this attribution and adaptation notice were added.
- This unofficial adaptation is not affiliated with or endorsed by Google. The pinned original and official guide are optional references for additional context.
