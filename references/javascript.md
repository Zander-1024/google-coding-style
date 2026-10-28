# Google JavaScript Style Guide

Editorially condensed adaptation of Google's JavaScript guide at commit
`572c673d9058242c049fbc8248b17029726c7afa`. Rules, their strength, exceptions
and representative examples are retained; repeated rationale and examples are
shortened. This unofficial adaptation is not endorsed by Google. Sources and
licenses are at the end; the rules can be used without opening the source.

> **Important: This guide is no longer being updated. Google recommends
> migrating to TypeScript, and following the [TypeScript guide](typescript.md).**

## 1 Introduction

This guide defines Google's JavaScript source standards, covering formatting
and other enforceable coding conventions.

### 1.1 Terminology notes

“Comment” means an implementation comment. “JSDoc” means human-readable text and
machine-readable annotations within `/** … */`.

The guide uses [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) meanings for
*must*, *must not*, *should*, *should not* and *may*. *Prefer* means *should*;
*avoid* means *should not*. Imperative and declarative statements prescribe
*must* requirements.

### 1.2 Guide notes

Example code is **non-normative**: it may illustrate one of several compliant
choices. Optional formatting choices in examples must not be enforced as rules.

## 2 Source file basics

### 2.1 File name

Names must be lowercase, may contain `_` or `-` but no other punctuation, and
must end in `.js`. Follow the project's convention.

### 2.2 File encoding: UTF-8

Source files are UTF-8.

### 2.3 Special characters

#### 2.3.1 Whitespace characters

Except for line terminators, ASCII space (0x20) is the only whitespace allowed
anywhere in source. Escape other whitespace in strings; never indent with tabs.

#### 2.3.2 Special escape sequences

Use special escapes (`\'`, `\"`, `\\`, `\b`, `\f`, `\n`, `\r`, `\t`, `\v`)
instead of numeric equivalents such as `\x0a`, `\u000a` or `\u{a}`. Never use
legacy octal escapes.

#### 2.3.3 Non-ASCII characters

For other non-ASCII characters, choose the literal character or hex/Unicode
escape solely for readability. Explanatory comments can help, especially for
escapes or non-printing characters. Do not sacrifice readability for broken
programs' inability to handle non-ASCII; those programs must be fixed.

```js
// Best:
const units = 'μs';
// Allowed, but unnecessary:
const escapedUnits = '\u03bcs'; // 'μs'
// Good: escape a non-printing character and explain it.
return '\ufeff' + content;  // Prepend a byte order mark.
// Poor: unexplained escape.
const unclearUnits = '\u03bcs';
```

## 3 Source file structure

All new files should be either `goog.module` files or ES modules (using
`import` / `export`). Sections occur in this order:

1. License/copyright, if present.
2. `@fileoverview` JSDoc, if present.
3. `goog.module`, for a `goog.module` file.
4. ES `import` statements, for an ES module.
5. `goog.require` / `goog.requireType` statements.
6. Implementation.

Separate present sections by **exactly one blank line**, except implementation
may have one or two preceding blank lines.

### 3.1 License or copyright information, if present

Put any license/copyright information here.

### 3.2 `@fileoverview` JSDoc, if present

Use [file-level JSDoc formatting](#75-topfile-level-comments).

### 3.3 `goog.module` statement

A `goog.module` file must declare exactly one module name on one unwrapped line,
even beyond 80 columns. The argument is a namespace: the package name reflecting
its directory fragment, optionally followed by the main class/enum/interface in
`lowerCamelCase`, e.g. `goog.module('search.urlHistory.urlHistoryService');`.

#### 3.3.1 Hierarchy

A module namespace must never be a direct child of another module namespace:
`foo.bar` and `foo.bar.baz` cannot both name modules. Directory hierarchy
reflects namespace hierarchy: deeper children are subdirectories of parents.

#### 3.3.2 `goog.module.declareLegacyNamespace`

The module statement may be followed by
`goog.module.declareLegacyNamespace();`, but avoid this when possible. Such a
module must not be an ancestor or descendant of any other module, including
indirect descendants such as `parent.child.grandchild` of `parent`: legacy
namespace creation depends on parent creation order. See the transitional
[legacy rules](#9445-googmoduledeclarelegacynamespace).

#### 3.3.3 `goog.module` Exports

Use the `exports` object, defining symbols on it or declaring them locally and
exporting separately. Export only symbols intended for use outside the module.
Do not annotate unexported locals `@private`; no ordering of locals and exports
is prescribed. Do not annotate the `exports` object `@const` (the compiler
already treats it as constant), or use default exports.

```js
// Correct:
const /** !Array<number> */ exportedArray = [1, 2, 3];
/** @return {number} */
function exportedFunction() {
  return exportedArray.length;
}
exports = {exportedArray, exportedFunction};
/** @const {number} */
exports.CONSTANT_ONE = 1;

// Incorrect:
/** @const */
exports = {exportedFunction};
exports = FancyClass;  // Default export.
```

### 3.4 ES modules

ES modules use `import` and `export` keywords.

#### 3.4.1 Imports

Import statements must not wrap, even beyond 80 columns.

##### 3.4.1.1 Import paths

Use `import` to import another ES module; do not `goog.require` it.

```js
import './sideeffects.js';
import * as parent from '../parent.js';
import {name} from './sibling.js';
```

###### 3.4.1.1.1 File extensions in import paths

The `.js` extension is mandatory: `import '../directory/file.js';`, never
`import '../directory/file';`.

##### 3.4.1.2 Importing the same file multiple times

Do not import the same file more than once; combine its imports.

##### 3.4.1.3 Naming imports

###### 3.4.1.3.1 Naming module imports

`import * as name` uses a `lowerCamelCase` name derived from the file/path:
`fileOne` for `file-one.js`, `fileTwo` for `file_two.js`, `fileThree` for
`filethree.js`, or `libString` / `vectorMath` to distinguish paths. The only
library exception is threejs, whose widely established prefix is `THREE`.

###### 3.4.1.3.2 Naming default imports

Derive default import names from filenames using the
[identifier-type rules](#62-rules-by-identifier-type), e.g. `MyClass`,
`myFunction`, `SOME_CONSTANT`. Default imports are only for modules that do not
conform to this guide; default exports here are banned.

###### 3.4.1.3.3 Naming named imports

Named imports should retain their names. Avoid aliases; prefer module imports
(`import *`) or renaming exports to resolve collisions. When an alias is needed,
include components of the imported file/path:

```js
import {Cat as BigCat} from './biganimals.js';
import {Cat as DomesticatedCat} from './domesticatedanimals.js';
```

#### 3.4.2 Exports

Export only intended external symbols. Unexported locals are not `@private`;
there is no prescribed ordering of exported and local symbols.

##### 3.4.2.1 Named vs default exports

Use named exports: `export class Foo {}` or a declaration followed by `export {Foo};`.
Do not use `export default`.

##### 3.4.2.2 Mutability of exports

Exported variables must not be mutated after module initialization; this also
applies to exported functions/classes. If mutation is needed, export a constant
reference to an object with mutable fields, or accessor/wrapper functions over
module-local mutable data.

```js
// Incorrect:
export let foo = 0;
export function mutateFoo() {
  ++foo;
}

// Correct alternative (in a separate module):
let foo = 0;
/** @return {number} */
export function getFoo() {
  return foo;
}
export function mutateFoo() {
  ++foo;
}
```

##### 3.4.2.3 export from

Neither form may wrap, even beyond 80 columns:

```js
export {specificName} from './other.js';
export * from './another.js';
```

#### 3.4.3 Circular Dependencies in ES modules

Do not create ES module cycles, including those formed by `export from` as well
as `import`, even though ECMAScript permits them.

#### 3.4.4 Interoperating with Closure

##### 3.4.4.1 Referencing goog

Import Closure's `goog.js` to reference `goog`. It exports only a subset of the
global `goog` properties usable by ES modules.

```js
import * as goog from '../closure/goog/goog.js';
const {compute} = goog.require('a.name');
export const CONSTANT = compute();
```

##### 3.4.4.2 goog.require in ES modules

It works as in `goog.module` files and may load any Closure namespace created
by `goog.provide` or `goog.module`, returning its value.

##### 3.4.4.3 Declaring Closure Module IDs in ES modules

An ES module may use `goog.declareModuleId` to declare an ID loadable by
`goog.require`, `goog.module.get`, etc., like a `goog.module` without a legacy
namespace. It does not create a global symbol. Loading this ID always returns
the module object (like `import *`), so the ID should end in `lowerCamelCase`.
This should only be used for in-place migration of Closure files to ES modules
with named exports. Calling `goog.module.declareLegacyNamespace` in an ES
module is an error; no direct ES-module legacy-namespace association exists.

```js
import * as goog from '../closure/goog.js';
goog.declareModuleId('my.esm');
export class Class {}
```

### 3.5 `goog.setTestOnly`

In `goog.module` files an optional `goog.setTestOnly()` follows `goog.module`
and any legacy-namespace call. In ES modules it follows the imports.

### 3.6 `goog.require` and `goog.requireType` statements

Use `goog.require` for code and type uses; `goog.requireType` imports may be
used only in type annotations. Put them in one contiguous top-level block with
no blank lines, following the module declaration with one blank line (or after
ES imports as above). They must not appear elsewhere. The argument is a module
namespace defined in a separate file.

Assign each require to a single `const` alias or destructure into `const`
aliases; these are the only permitted dependency references in code and types.
Never use fully qualified namespaces except as require arguments. **Exception:**
externs declarations must use fully qualified names in code and annotations.

A single alias must match the namespace's entire last component. Longer aliases
may add components from that namespace to disambiguate or substantially improve
readability; retain original casing so the identifier still indicates its type.
Longer aliases **must** prevent masking native types such as `Element`, `Event`,
`Error`, `Map` or `Promise`. Put a space after colons when renaming destructured
aliases. Do not add arbitrary words or drop part of the final component.

A file should not require and requireType the same namespace. When both code
and types need it, a single `goog.require` should be used. Side-effect-only imports must
use `goog.require`; assignment may be omitted, but explain the need in a comment
and suppress the compiler warning.

Sort first single-alias requires alphabetically by alias, then destructuring
requires alphabetically by left-hand names, then standalone calls. Never wrap
a require line, even beyond 80 columns. IDE sorting may be used.

```js
const asserts = goog.require('goog.asserts');
const testingAsserts = goog.require('goog.testing.asserts');
const {Element: RendererElement} = goog.require('web.renderer');
const {MyType} = goog.requireType('other.package');
const {clear, clone} = goog.require('goog.array');
/** @suppress {extraRequire} Initializes MyFramework. */
goog.require('my.framework.initialization');

// Incorrect:
const Map = goog.require('jspb.Map');  // Must use JspbMap.
const randomName = goog.require('something.else');
const MyClass = goog.require('some.package.MyClassForBizzing');
const {Foo:FooProto} = goog.require('some.package.proto.Foo');
goog.requireType('some.package.with.a.Type');  // Missing alias.
```

Some legacy modules default-export one class, enum or record; do not create
such exports in new code. When consuming them, prefer destructuring into
aliases. To disambiguate a legacy `some.package.Class`, prefer `PackageClass`
over `SomeClass`, since the additional component comes from the namespace.

### 3.7 The file’s implementation

Implementation follows all dependency information, separated by at least one
blank line. It may contain any module-local declarations and exported symbols.

## 4 Formatting

A *block-like construct* is a class/function/method body or brace-delimited
code block; array and object literals may optionally be treated as block-like.
`clang-format` is useful for formatting, but is not required.

### 4.1 Braces

#### 4.1.1 Braces are used for all control structures

Use braces for all control structures (`if`, `else`, `for`, `do`, `while`, etc.),
even single statements. A nonempty block's first statement starts on a new
line. **Only exception:** an unwrapped, single-line `if` with no `else` may omit
braces/newlines when it improves readability: `if (shortCondition()) foo();`.

#### 4.1.2 Nonempty blocks: K&R style

For nonempty blocks and block-like constructs:

- No newline before `{`; newline after it and before `}`.
- Newline after `}` when it ends a statement, function/class statement body or
  class method. No newline if followed by `else`, `catch`, `while`, comma,
  semicolon or right parenthesis.

```js
if (condition) {
  try {
    something();
  } catch (err) {
    recover();
  }
} else {
  fallback();
}
```

#### 4.1.3 Empty blocks: may be concise

An empty block may use `{}` with nothing between the braces, **unless** part
of a multi-block `if`/`else` or `try`/`catch`/`finally` statement. Thus
`function doNothing() {}` is allowed, but `try { work(); } catch (e) {}` and
`if (a) {} else {}` are not.

### 4.2 Block indentation: +2 spaces

Each opened block/block-like construct adds two spaces; return to the previous
level on closing. This applies to both code and comments.

#### 4.2.1 Array literals: optionally "block-like"

Array literals may use block formatting. Other combinations are allowed,
especially to emphasize semantic groups, but should not be used solely to
reduce large arrays' vertical size. These are representative allowed choices:

```js
const a = [
  0,
  1,
];
const b =
    [0, 1, 2];
const c = [0, 1, 2];
someMethod(foo, [
  0, 1, 2,
], bar);
```

#### 4.2.2 Object literals: optionally "block-like"

Object literals have the same formatting options as arrays:

```js
const a = {
  a: 0,
  b: 1,
};
const b =
    {a: 0, b: 1};
const c = {a: 0, b: 1};
```

#### 4.2.3 Class literals

Indent class declarations/expressions as blocks. No semicolon after a method or
class declaration's closing brace; a statement containing a class expression
still ends in a semicolon. For inheritance, use only `extends` unless the
superclass is generic; generic subclasses must give explicit `@extends` type
parameters even when passing through the same template name.

```js
/** @extends {Foo<number>} */
class Bar extends Foo {
  constructor() {
    super(42);
  }
}
exports.Baz = class extends Bar {};  // No unnecessary @extends {Bar}.
```

#### 4.2.4 Function expressions

An anonymous function used as a call argument has its body indented +2 from the
preceding call's indentation depth, including when the call is a wrapped chain:

```js
someFunction(arg, () => {
  work();
});
someCall()
    .then((result) => {
      result.use();
    });
```

#### 4.2.5 Switch statements

Indent contents +2, labels on their own lines, statements after each label +2
further. A subsequent label returns to the previous level. An explicit block
may be used for lexical scope. A blank line between a `break` and next case is
optional. See the [switch example](#5831-fall-through-commented).

### 4.3 Statements

#### 4.3.1 One statement per line

Follow each statement with a line break.

#### 4.3.2 Semicolons are required

Terminate every statement with a semicolon; automatic insertion is forbidden.
Function/class declarations are subject to the exceptions stated above.

### 4.4 Column limit: 80

Wrap lines beyond 80 characters, except:

- `goog.module`, `goog.require`, `goog.requireType`, ES `import`, and
  `export from` statements, which must not wrap.
- Lines where wrapping is impossible or harms discoverability, e.g. clickable
  URLs, copyable shell commands, or string literals that need whole-string
  copying/searching (such as long paths).

### 4.5 Line-wrapping

There may be several valid wraps; code under 80 columns may also wrap at the
author's discretion. Extracting a method/local can avoid a difficult wrap.

#### 4.5.1 Where to break

Prefer a higher syntactic level. In `estimate = calc(estimate + x * estimate) /
2.0`, assignment is higher than division, call, parameters and number constant.
Aim for clarity rather than minimum line count.

- Break **after** an operator; this does not apply to dot (`.`).
- Keep method/constructor names with their following `(`, and commas with the
  preceding token.
- Never split `return` from its value.
- Break JSDoc type annotations after `{`; optional-type tags such as `@const`,
  `@private` and `@param` do not scan the next line for the opening brace.

```js
// Preferred:
currentEstimate =
    calc(currentEstimate + x * currentEstimate) /
        2.0;
// Discouraged:
currentEstimate = calc(currentEstimate + x *
    currentEstimate) / 2.0;
```

#### 4.5.2 Indent continuation lines at least +4 spaces

Except where block indentation applies, continuation lines are indented at
least +4 from the original line. Beyond +4, indentation may vary appropriately;
In general, deeper syntax uses larger multiples of four, and two lines use the
same indentation iff they begin with syntactically parallel elements. Horizontal
alignment is discouraged.

### 4.6 Whitespace

#### 4.6.1 Vertical whitespace

Use a single blank line between consecutive methods in classes/object literals;
between adjacent object property definitions it is optional for logical groups.
Within methods use blanks sparingly for logical groups; none at the start/end
of a function body. A blank before the first/after the last method is optional,
neither encouraged nor discouraged. Follow other required section spacing.
Multiple consecutive blank lines are permitted, never required or encouraged.

#### 4.6.2 Horizontal whitespace

No trailing whitespace. Apart from language/style requirements and literals,
comments or JSDoc, a single internal ASCII space occurs **only**:

1. Between a reserved word and following `(`, except `function` and `super`.
2. Between `}` and a following reserved word such as `else` or `catch`.
3. Before `{`, except an object literal as first call argument/array element
   (`foo({a: [{c: d}]})`) or template expansion (`` `ab${1 + 2}cd` ``).
4. On both sides of binary/ternary operators.
5. After `,` or `;`, never before them.
6. After an object-literal colon.
7. On both sides of end-of-line `//`; multiple spaces are allowed here.
8. After opening and on both sides of closing block-comment delimiters:
   `this.foo = /** @type {number} */ (bar)`, `function(/** string */ foo) {`,
   `baz(/* buzz= */ true)`.

#### 4.6.3 Horizontal alignment: discouraged

Adding variable spaces to align tokens with previous lines is allowed but
generally discouraged; maintaining existing alignment is not required.

```js
// Preferred:
const x = {tiny: 42, longer: 435};
// Also allowed, but discouraged:
const y = {
  tiny:   42,
  longer: 435,
};
```

#### 4.6.4 Function arguments

Prefer all arguments on the function-name line. If beyond 80 columns, wrap
readably: close to 80 or one argument per line. Indentation should be four
spaces; aligning to `(` is allowed but discouraged. When all arguments fit on
the second line, putting them together there with +4 is preferred. Wrapping at
80 can use less space but is not recommended when it violates the rectangle
rule: each wrapped part should fit its own bounding rectangle containing all
and only that part's text.

```js
doSomething(
    descriptiveArgumentOne, descriptiveArgumentTwo, descriptiveArgumentThree);
doSomething(
    veryDescriptiveArgumentNumberOne,
    veryDescriptiveArgumentTwo,
    artichokeDescriptorAdapterIterator);
```

### 4.7 Grouping parentheses: recommended

Omit optional grouping only when author and reviewer agree there is no
reasonable risk of misreading and parentheses would not improve readability.
Do not assume readers memorize operator precedence. Do not unnecessarily wrap
the entire expression after `delete`, `typeof`, `void`, `return`, `throw`,
`case`, `in`, `of` or `yield`. Casts require parentheses:
`/** @type {!Foo} */ (foo)`.

### 4.8 Comments

These are implementation comments; see [JSDoc](#7-jsdoc) for documentation.

#### 4.8.1 Block comment style

Indent comments with surrounding code. `/* … */` and `//` are allowed. In
multiline `/* … */`, subsequent lines start with `*` aligned with the preceding
`*`. Do not draw comment boxes or use `/** … */` for implementation comments.

```js
/*
 * A multiline comment.
 */
// Another allowed
// multiline form.
```

#### 4.8.2 Parameter Name Comments

These should be used when value and method name do not convey meaning and a
clarifying refactor is infeasible. Prefer a comment before the value with `=`;
for consistency with nearby code a comment after the value without `=` is
allowed:

```js
someFunction(obviousParam, /* shouldRender= */ true, /* name= */ 'hello');
someFunction(obviousParam, true /* shouldRender */, 'hello' /* name */);
```

## 5 Language features

Features not discussed here may be used without further recommendations.

### 5.1 Local variable declarations

#### 5.1.1 Use `const` and `let`

Declare locals with `const` by default; use `let` only when reassignment is
needed. Do not use `var` except under the [legacy rules](#94-exceptions-for-legacy-platforms).

#### 5.1.2 One variable per declaration

Each local declaration declares one variable, never `let a = 1, b = 2;`.

#### 5.1.3 Declared when needed, initialized as soon as possible

Declare close to first use, within reason, to minimize scope; initialize as soon
as possible. Do not habitually hoist declarations to the block's start.

#### 5.1.4 Declare types as needed

JSDoc types may go above a declaration, or inline before the variable name if
there is no other JSDoc. Do not mix both styles: the compiler ignores the second.
Types are particularly useful for empty generic containers or variables changed
in closures, where inference may miss template parameters.

```js
const /** !Array<number> */ data = [];
/**
 * Some description.
 * @type {!Array<number>}
 */
const moreData = [];

// Incorrect:
/** Some description. */
const /** !Array<number> */ ignoredType = [];
```

### 5.2 Array literals

#### 5.2.1 Use trailing commas

Include a trailing comma when a newline separates the final element from `]`.

#### 5.2.2 Do not use the variadic `Array` constructor

Use array literals (`[x1, x2]`, `[x1]`, `[]`), not `new Array(...)`: one numeric
argument has different semantics. Explicit `new Array(length)` allocation is
allowed when appropriate.

#### 5.2.3 Non-numeric properties

Do not define/use array non-numeric properties other than `length`; use `Map`
or `Object` instead.

#### 5.2.4 Destructuring

Array destructuring is allowed in assignments and parameters. A final rest
element may be used with no space after `...`; unused elements should be omitted. Optional
destructured array parameters must default to `[]`, with element defaults on
the left. JSDoc must supply a parameter name even though it is ignored.

```js
const [a, b, c, ...rest] = generateResults();
let [, second,, fourth] = someArray;
/** @param {!Array<number>=} param1 */
function good([a = 4, b = 2] = []) {}
// Incorrect:
function bad([a, b] = [4, 2]) {}
```

For unpacking/packing multiple parameters or returns, prefer object
destructuring where possible, for names and differing element types.

#### 5.2.5 Spread operator

Array spread may flatten iterables; prefer it over awkward `Array.prototype`
constructs. No space after `...`: `[...foo]` rather than
`Array.prototype.slice.call(foo)`, and `[...foo, ...bar]` rather than
`foo.concat(bar)`.

### 5.3 Object literals

#### 5.3.1 Use trailing commas

Include a trailing comma when a newline separates the final property from `}`.

#### 5.3.2 Do not use the `Object` constructor

Use `{}` / an object literal, not the `Object` constructor.

#### 5.3.3 Do not mix quoted and unquoted keys

A literal is either a *struct* (unquoted and/or symbol keys) or a *dict* (quoted
and/or computed keys); do not mix them. This also applies to property names
passed to functions such as `hasOwnProperty`: compiler renaming cannot update
string literals for struct properties.

```js
// Incorrect: mixed key styles.
const mixed = {width: 42, 'maxWidth': 43};
/** @type {{width: number, maxWidth: (number|undefined)}} */
const o = {width: 42};
// Incorrect: o.hasOwnProperty('maxWidth')
// Correct:
if (o.maxWidth != null) { /* ... */ }
```

#### 5.3.4 Computed property names

Computed names such as `{['key' + foo()]: 42}` are allowed and count as dict
keys unless symbols (e.g. `[Symbol.iterator]`), so do not mix non-symbol
computed keys with unquoted ones. Enum values may be computed keys, but should
not mix with non-enum keys in one literal.

#### 5.3.5 Method shorthand

Object methods may use `{method() { ... }}` instead of a colon plus function or
arrow literal. `this` in shorthand/`function` refers to the object itself;
arrows capture the surrounding scope's `this`.

```js
return {
  stuff: 'candy',
  method() {
    return this.stuff;  // 'candy'.
  },
  outer: () => this.stuff,  // Surrounding instance's stuff.
};
```

#### 5.3.6 Shorthand properties

Object shorthand properties are allowed, e.g. `const obj = {foo, bar};`.

#### 5.3.7 Destructuring

Object destructuring is allowed in assignments. Destructured parameters should
be simple: a single level of unquoted shorthand properties; nesting and computed
properties are forbidden. Put defaults on the left; an optional object must
default to `{}`. JSDoc may use any parameter name; it is unused but required.

```js
/** @param {{num: (number|undefined), str: (string|undefined)}=} param1 */
function good({num, str = 'some default'} = {}) {}
// Incorrect parameter patterns:
function nested({x: {num, str}}) {}
function renamed({num: a, str: b} = {}) {}
function computed({a, b, [a + b]: c}) {}
function wrongDefaults({a, b} = {a: 2, b: 4}) {}
```

Destructuring is also allowed in requires, which must remain entirely on one
line regardless of length.

#### 5.3.8 Enums

Define enums with `@enum` on an object literal, module-local or directly on
`exports`, never nested under a type/object. Enums must be constant; do not add
properties later. Values must be string literals or numbers. String values must
be statically initialized: no arithmetic, template interpolation, calls or
variable references. For complex values consider a const object without `@enum`.

```js
/**
 * Supported temperature scales.
 * @enum {string}
 */
const TemperatureScale = {
  CELSIUS: 'celsius',
  FAHRENHEIT: 'fahrenheit',
};
// Incorrect enum values: ABSOLUTE_ZERO, 0 + '°F', `${value}°F`, getValue().
```

### 5.4 Classes

#### 5.4.1 Constructors

Constructors are optional. Subclass constructors must call `super()` before
setting fields or accessing `this`. Interfaces should declare non-method
properties in the constructor.

#### 5.4.2 Fields

Define all concrete fields (non-method properties) in the constructor, never
within a nested scope or on the concrete class's prototype. Mark never-reassigned
fields `@const` (deep immutability is unnecessary) and non-public fields with
`@private`, `@protected` or `@package`. Private names may end in `_`.
Properties should never be added/removed after construction; fields initialized
later should explicitly be `undefined` in the constructor. `@struct` checks
undeclared properties; classes have it by default.

```js
class Foo {
  constructor() {
    /** @private @const {!Bar} */
    this.bar_ = computeBar();
    /** @protected @const {!Baz} */
    this.baz = computeBaz();
  }
}
```

#### 5.4.3 Computed properties

Only symbols may be computed class properties; dict-style quoted/non-symbol
computed keys are forbidden. Logically iterable classes should define
`[Symbol.iterator]`. Otherwise `Symbol` should be used sparingly; other built-in symbols
may lack compiler polyfills and fail in old browsers.

#### 5.4.4 Static methods

Prefer module-local functions over private static methods where readability
permits. Code should not depend on dynamic static-method dispatch; methods
should be called only on their defining base class. Dynamic constructor-variable
calls are discouraged and require `@nocollapse` if used. Never call an inherited
static method directly on a subclass that does not define it. Do not use `this`
in static methods.

```js
class Base {
  /** @nocollapse */
  static foo() {}
}
class Sub extends Base {}
// Discouraged: cls.foo(), when cls may be a base or subclass constructor.
// Forbidden: Sub.foo(); inherited statics must not be called on Sub.
```

#### 5.4.5 Old-style class declarations

Prefer ES6 classes. They may be infeasible when existing/framework-created
subclasses cannot yet migrate, or a framework needs `this` before the superclass
constructor. Other rules still apply; `let`, `const`, defaults, rest and arrows
should be used when appropriate.

For new fallback code prefer `goog.defineClass`; traditional constructors plus
`goog.inherits` are also allowed. Per-instance properties should follow the
superclass call in the constructor, and methods should be on the prototype.
Using Closure's `goog.inherits` is best for correct prototype hierarchies.

```js
let C = goog.defineClass(S, {
  /** @param {string} value */
  constructor(value) {
    S.call(this, 2);
    /** @const */
    this.prop = value;
  },
  /** @return {number} */
  method() {
    return 0;
  },
});
// Traditional alternative:
/**
 * @constructor
 * @extends {S}
 * @param {string} value
 */
function D(value) {
  S.call(this, 2);
  /** @const */
  this.prop = value;
}
goog.inherits(D, S);
/** @return {number} */
D.prototype.method = function() {
  return 0;
};
```

#### 5.4.6 Do not manipulate `prototype`s directly

Ordinary implementation code must not manipulate prototypes except for old-style
class definitions above. Mixins and built-in prototype changes are forbidden.
Frameworks such as Polymer/Angular may need prototypes and should not use worse
workarounds merely to avoid them.

#### 5.4.7 Getters and Setters

Do not use JavaScript getter/setter properties; provide ordinary methods.
**Exception:** unavoidable framework binding or unchangeable external APIs may
use them cautiously, only via `get`/`set` shorthand or
`Object.defineProperties`, never `Object.defineProperty` (which harms renaming).
Getters must not change observable state; `get next() { return this.nextId++; }`
is forbidden.

#### 5.4.8 Overriding toString

It may be overridden but must always succeed without visible side effects.
Beware calls to other methods that may lead to error-driven infinite loops.

#### 5.4.9 Interfaces

Interfaces may use `@interface` or `@record`; a record can be explicitly (`@implements`) or
implicitly implemented by a class/object literal. Interface methods must be
non-static with empty bodies. Fields must be uninitialized members in the
constructor.

```js
/**
 * A frobnicator.
 * @record
 */
class Frobnicator {
  constructor() {
    /** @type {number} Number of attempts before giving up. */
    this.attempts;
  }

  /** @param {!FrobnicationStrategy} strategy */
  frobnicate(strategy) {}
}
```

#### 5.4.10 Abstract Classes

Use abstract classes when appropriate. Abstract classes and methods must have
`@abstract`; do not use `goog.abstractMethod`.

#### 5.4.11 Do not create static container classes

Do not create classes solely to namespace static methods/properties. Export the
individual functions and constants instead.

#### 5.4.12 Do not define nested namespaces

Do not define a nested class, typedef, enum, interface or other type on another
module-local name (`Foo.Bar = class {}` is forbidden). They should be top-level
exports with clear names, e.g. `FooConverter`. Consider removing redundancy
with the module name: `foo.Foo` and `foo.Converter`. Importers can add a prefix
for clarity, while namespace imports cannot easily remove one.

### 5.5 Functions

#### 5.5.1 Top-level functions

They may be defined on `exports`, or locally and optionally exported:
`exports.processString = (str) => { ... };` or a local `const` and
`exports = {processString};`.

#### 5.5.2 Nested functions and closures

Nested functions are allowed; a useful name should be assigned to a local `const`.

#### 5.5.3 Arrow functions

Prefer arrows for nested functions (except object method shorthand), and over
`f.bind(this)`, `goog.bind(f, this)` or `const self = this`. They also allow
explicit callback parameter selection rather than passing all bound arguments.

Parameter parentheses are optional for one non-destructured parameter; with
parentheses inline parameter types are allowed. Always using parentheses can
avoid accidental behavior changes when adding parameters.

The body defaults to a block. An implicitly returned expression is allowed
only when logic requires a returned value, or `void` precedes a single
function/method call (ensuring `undefined`, preventing leaked values and showing
intent). Prefer a single expression when clearer; otherwise use a block.

```js
// Correct:
const add = (a, b) => a + b;
getValue((result) => void alert(`Got ${result}`));
// Inside a class constructor/method:
getNullableValue((/** ?number */ result) => {
  this.cachedValue_ = result == null ? 0 : result;
});
// Incorrect when no return value is intended:
const action = () => anotherFunction();
```

#### 5.5.4 Generators

Generators may be used as needed. Attach `*` to `function` (when present), with
a space before the name; attach it to `yield` in delegation:

```js
/** @return {!Iterator<number>} */
function* gen1() {
  yield 42;
}
/** @return {!Iterator<number>} */
const gen2 = function*() {
  yield* gen1();
};
class SomeClass {
  /** @return {!Iterator<number>} */
  * gen() {
    yield 42;
  }
}
```

#### 5.5.5 Parameter and return types

Parameter/return types should usually be documented with [JSDoc](#78-method-and-function-comments).

##### 5.5.5.1 Default parameters

Optional parameters may use defaults. They must have spaces around `=`, ordinary
names (no `opt_`), a JSDoc `=` type suffix, occur after required parameters, and
have no observably side-effectful initializers. Concrete functions must default
every optional parameter, even to `undefined`; abstract/interface methods must
omit defaults.

```js
/**
 * @param {string} required
 * @param {string=} optional
 * @param {!Node=} node
 */
function concrete(required, optional = '', node = undefined) {}
/** @interface */
class MyInterface {
  /** @param {string=} optional */
  someMethod(optional) {}
}
```

Use defaults sparingly; prefer destructuring when more than a few optional
parameters lack a natural order. New mutable objects such as `{}`/`[]` are
allowed: defaults evaluate on each use. Initializers should be as simple as possible; avoid
exposing shared mutable state.

##### 5.5.5.2 Rest parameters

Use rest instead of `arguments`. Rest must be last, have no space after `...`,
and use a JSDoc `...` type prefix, e.g. `@param {...number} numbers` for
`function variadic(array, ...numbers) {}`. Do not name it `var_args`, or name any
local/parameter `arguments`.

#### 5.5.6 Generics

Declare generic functions/methods as needed with `@template TYPE` in the JSDoc
above the definition.

#### 5.5.7 Spread operator

Call spread is allowed. Prefer `f(...array, ...iterable)` over
`Function.prototype.apply` for unpacking a variadic function's arguments.
No space after `...`.

### 5.6 String literals

#### 5.6.1 Use single quotes

Use single quotes for ordinary strings; they must not span lines. Consider a
template string to avoid escaping an embedded single quote.

#### 5.6.2 Template literals

Use backtick templates over complex concatenation, especially with multiple
string literals. They may span lines; they need not follow enclosing block
indentation, but may if extra whitespace does not matter.

#### 5.6.3 No line continuations

Never end a string line with backslash to continue it, for ordinary or template
strings. Instead concatenate literals:

```js
const longString = 'A long string with ' +
    'a continuation without embedded indentation.';
```

### 5.7 Number literals

Decimal, hex, octal and binary are allowed. Prefixes are exactly lowercase
`0x`, `0o`, `0b`; no other leading zero is allowed.

### 5.8 Control structures

#### 5.8.1 For loops

All three forms are allowed; prefer `for-of` where possible. `for-in` is only
for dict objects, should not iterate arrays, and should use
`Object.prototype.hasOwnProperty` to exclude prototype properties. Prefer
`for-of` and `Object.keys` over `for-in`.

#### 5.8.2 Exceptions

Exceptions should be used for exceptional conditions. Always throw `Error` or a subclass,
never strings/other objects; always construct errors with `new`. The same
applies to Promise rejections (like throws in async functions). Custom errors
should be defined/used where native `Error` is insufficient. Prefer exceptions
over ad-hoc error containers or objects with error properties.

##### 5.8.2.1 Empty catch blocks

A catch that intentionally does nothing must explain why in a comment. Do not
test failure by calling `fail()` inside `try` then ignoring its catch: that
also catches `fail`'s error. Use `assertThrows()` instead.

```js
try {
  return handleNumericResponse(response);
} catch (ok) {
  // Not numeric; continue with the text path.
}
return handleTextResponse(response);
```

#### 5.8.3 Switch statements

A statement group has one or more `case`/`default` labels followed by statements.

##### 5.8.3.1 Fall-through: commented

Each group must terminate abruptly (`break`, `return`, `throw`) or have a
comment communicating possible fall-through; no such comment is required on
the final group.

```js
switch (input) {
  case 1:
  case 2:
    prepareOneOrTwo();
  // fall through
  case 3:
    handleOneTwoOrThree();
    break;
  default:
    handleLargeNumber(input);
}
```

##### 5.8.3.2 The `default` case is present

Every switch includes a `default` group, even empty; it must be last.

### 5.9 this

Use `this` only in class constructors and methods, arrows defined within them, or functions
with explicit `@this` in the immediately enclosing function's JSDoc. Never use it
for the global object, eval context, event target or unnecessarily called/applied
functions.

### 5.10 Equality Checks

Use `===` / `!==` except below.

#### 5.10.1 Exceptions Where Coercion is Desirable

`value == null` may catch both `null` and `undefined`, for objects/primitives,
without catching other false values such as `0` or `''`.

### 5.11 Disallowed features

#### 5.11.1 with

Do not use `with`.

#### 5.11.2 Dynamic code evaluation

Do not use `eval` or `Function(...string)`, except in code loaders. They also
fail in environments enforcing Content Security Policy.

#### 5.11.3 Automatic semicolon insertion

Always use semicolons except the function/class declaration cases above.

#### 5.11.4 Non-standard features

Use only current ECMA-262 / WHATWG features: no removed features, proposals at
any stage, incomplete web standards or proprietary browser features. Specific
API projects (e.g. Chrome extensions or Node.js) may use those APIs.
Non-standard language extensions from transpilers are forbidden.

#### 5.11.5 Wrapper objects for primitive types

Never use `new Boolean`, `new Number`, `new String` or `new Symbol`, or those
wrappers in type annotations. Function calls for coercion/symbol creation are
allowed; prefer `Boolean(x)`, `Number(x)`, `String(x)` over unary `+` or empty
string concatenation. `new Boolean(false)` is a truthy object; `Boolean(0)` is
the primitive `false`.

#### 5.11.6 Modifying builtin objects

Never modify built-in types' constructors or prototypes. Avoid libraries doing
so. Only the compiler runtime's standards-compliant polyfills may modify them.
Do not add globals unless absolutely necessary, e.g. a third-party API requires
them.

#### 5.11.7 Omitting `()` when invoking a constructor

Always use parentheses: `new Foo()`, never `new Foo`. `new Foo().Bar()` and
`new Foo.Bar()` have different meanings.

## 6 Naming

### 6.1 Rules common to all identifiers

Identifiers use only ASCII letters and digits, and, in a small number of cases
noted below, underscores and very rarely (when required by frameworks like
Angular) dollar signs.

Give as descriptive a name as possible, within reason. Do not worry about saving
horizontal space as it is far more important to make your code immediately
understandable by a new reader. Do not use abbreviations that are ambiguous or
unfamiliar to readers outside your project, and do not abbreviate by deleting
letters within a word.

**Correct example:**

```js
errorCount          // No abbreviation.
dnsConnectionIndex  // Most people know what "DNS" stands for.
referrerUrl         // Ditto for "URL".
customerId          // "Id" is both ubiquitous and unlikely to be misunderstood.
```

Disallowed:

**Incorrect example:**

```js
n                   // Meaningless.
nErr                // Ambiguous abbreviation.
nCompConns          // Ambiguous abbreviation.
wgcConnections      // Only your group knows what this stands for.
pcReader            // Lots of things can be abbreviated "pc".
cstmrId             // Deletes internal letters.
kSecondsPerDay      // Do not use Hungarian notation.
```

**Exception**: Variables that are in scope for 10 lines or fewer, including
arguments that are *not* part of an exported API, *may* use short (e.g. single
letter) variable names.

### 6.2 Rules by identifier type

#### 6.2.1 Package names

Package names are all `lowerCamelCase`. For example, `my.exampleCode.deepSpace`,
but not `my.examplecode.deepspace` or `my.example_code.deep_space`.

**Exception**: The package name may conform to TypeScript's path-based pattern.
This is typically all lower case with underscores where present in filenames.

#### 6.2.2 Class names

Class, interface, record, and typedef names are written in `UpperCamelCase`.
Unexported classes are simply locals: they are not marked `@private`.

Type names are typically nouns or noun phrases. For example, `Request`,
`ImmutableView`, or `VisibilityMode`. Additionally, interface names may
sometimes be adjectives or adjective phrases instead (for example, `Readable`).

#### 6.2.3 Method names

Method names are written in `lowerCamelCase`. Names for `@private` methods may
optionally end with a trailing underscore.

Method names are typically verbs or verb phrases. For example, `sendMessage` or
`stop_`. Getter and setter methods for properties are never required, but if
they are used they should be named `getFoo` (or optionally `isFoo` or `hasFoo`
for booleans), or `setFoo(value)` for setters.

Underscores may also appear in JsUnit test method names to separate logical
components of the name. One typical pattern is
`test<MethodUnderTest>_<state>_<expectedOutcome>`, for example
`testPop_emptyStack_throws`. There is no One Correct Way to name test methods.

#### 6.2.4 Enum names

Enum names are written in `UpperCamelCase`, similar to classes, and should
generally be singular nouns. Individual items within the enum are named in
`CONSTANT_CASE`.

#### 6.2.5 Constant names

Constant names use `CONSTANT_CASE`: all uppercase letters, with words separated
by underscores. There is no reason for a constant to be named with a trailing
underscore, since private static properties can be replaced by (implicitly
private) module locals.

##### 6.2.5.1 Definition of “constant”

Every constant is a `@const` static property or a module-local `const`
declaration, but not all `@const` static properties and module-local `const`s
are constants. Before choosing constant case, consider whether the field really
feels like a *deeply immutable* constant. For example, if any of that instance's
observable state can change, it is almost certainly not a constant. Merely
intending to never mutate the object is generally not enough.

Examples:

**Correct example:**

```js
// Constants
const NUMBER = 5;
/** @const */ exports.NAMES = goog.debug.freeze(['Ed', 'Ann']);
/** @enum */ exports.SomeEnum = { ENUM_CONSTANT: 'value' };

// Not constants
let letVariable = 'non-const';

class MyClass {
  constructor() { /** @const {string} */ this.nonStatic = 'non-static'; }
};
/** @type {string} */
MyClass.staticButMutable = 'not @const, can be reassigned';

const /** Set<string> */ mutableCollection = new Set();

const /** MyImmutableContainer<SomeMutableType> */ stillMutable =
    new MyImmutableContainer(mutableInner);

const {Foo} = goog.require('my.foo');  // mirrors imported name

const logger = log.getLogger('loggers.are.not.immutable');
```

Constants’ names are typically nouns or noun phrases.

##### 6.2.5.2 Local aliases

Local aliases should be used whenever they improve readability over
fully-qualified names. Follow the same rules as `goog.require`s
([3.6 `goog.require` and `goog.requireType` statements](#36-googrequire-and-googrequiretype-statements)),
maintaining the last part of the aliased name. Aliases may also be used within
functions. Aliases must be `const`.

Examples:

**Correct example:**

```js
const staticHelper = importedNamespace.staticHelper;
const CONSTANT_NAME = ImportedClass.CONSTANT_NAME;
const {assert, assertInstanceof} = asserts;
```

#### 6.2.6 Non-constant field names

Non-constant field names (static or otherwise) are written in `lowerCamelCase`,
with an optional trailing underscore for private fields.

These names are typically nouns or noun phrases. For example, `computedValues`
or `index_`.

#### 6.2.7 Parameter names

Parameter names are written in `lowerCamelCase`. Note that this applies even if
the parameter expects a constructor.

One-character parameter names should not be used in public methods.

**Exception**: When required by a third-party framework, parameter names may
begin with a `$`. This exception does not apply to any other identifiers (e.g.
local variables or properties).

#### 6.2.8 Local variable names

Local variable names are written in `lowerCamelCase`, except for module-local
(top-level) constants, as described above. Constants in function scopes are
still named in `lowerCamelCase`. Note that `lowerCamelCase` is used even if the
variable holds a constructor.

#### 6.2.9 Template parameter names

Template parameter names should be concise, single-word or single-letter
identifiers, and must be all-caps, such as `TYPE` or `THIS`.

#### 6.2.10 Module-local names

Module-local names that are not exported are implicitly private. They are not
marked `@private`. This applies to classes, functions, variables, constants,
enums, and other module-local identifiers.

### 6.3 Camel case: defined

Sometimes there is more than one reasonable way to convert an English phrase
into camel case, such as when acronyms or unusual constructs like "IPv6" or
"iOS" are present. To improve predictability, Google Style specifies the
following (nearly) deterministic scheme.

Beginning with the prose form of the name:

1.  Convert the phrase to plain ASCII and remove any apostrophes. For example,
    "Müller's algorithm" might become "Muellers algorithm".
2.  Divide this result into words, splitting on spaces and any remaining
    punctuation (typically hyphens).
    1.  Recommended: if any word already has a conventional camel case
        appearance in common usage, split this into its constituent parts (e.g.,
        "AdWords" becomes "ad words"). Note that a word such as "iOS" is not
        really in camel case per se; it defies any convention, so this
        recommendation does not apply.
3.  Now lowercase everything (including acronyms), then uppercase only the first
    character of:
    1.  … each word, to yield `UpperCamelCase`, or
    2.  … each word except the first, to yield `lowerCamelCase`
4.  Finally, join all the words into a single identifier.

Note that the casing of the original words is almost entirely disregarded.

Examples of `lowerCamelCase`:

Prose form              | Correct             | Incorrect
:---------------------: | :-----------------: | :-----------------:
"XML HTTP request"      | `xmlHttpRequest`    | `XMLHTTPRequest`
"new customer ID"       | `newCustomerId`     | `newCustomerID`
"inner stopwatch"       | `innerStopwatch`    | `innerStopWatch`
"supports IPv6 on iOS?" | `supportsIpv6OnIos` | `supportsIPv6OnIOS`
"YouTube importer"      | `youTubeImporter`   | `youtubeImporter`*

*Acceptable, but not recommended.

For examples of `UpperCamelCase`, uppercase the first letter of each correct
`lowerCamelCase` example.

Note: Some words are ambiguously hyphenated in the English language: for example
"nonempty" and "non-empty" are both correct, so the method names `checkNonempty`
and `checkNonEmpty` are likewise both correct.

## 7 JSDoc

Use JSDoc on all classes, fields and methods. The following rules define its
format and annotation requirements.

### 7.1 General form

JSDoc must be well-formed. Single-line form is allowed; if it wraps, put `/**`
and `*/` on their own lines and use the multiline style:

```js
/**
 * Does something with a number.
 * @param {number} arg A number to use.
 */
function doSomething(arg) { /* ... */ }
/** @const @private {!Foo} A short description. */
this.foo_ = foo;
```

### 7.2 Markdown

Write Markdown; HTML may be used when necessary. Use real Markdown lists,
rather than relying on indentation that extraction tools may collapse:

```js
/**
 * Computes weight from:
 *
 * - items sent
 * - items received
 * - last timestamp
 */
```

### 7.3 JSDoc tags

Only the supported subset in [the tag reference](#91-jsdoc-tag-reference) is
allowed. Most tags start at the beginning of their own line; do not combine
`@param`s. Simple tags without additional data (`@private`, `@const`, `@final`,
`@export`) may share a line, optionally with a type. Put complex tags such as
`@implements` and `@template` on their own lines. There is no hard combination
or ordering rule; be consistent.

```js
/**
 * @export @final
 * @implements {Iterable<TYPE>}
 * @template TYPE
 */
class MyClass {}
```

### 7.4 Line wrapping

Indent wrapped block tags four spaces. Aligning wrapped descriptions with their
previous text is allowed but discouraged. Do not indent wrapped `@desc` or
`@fileoverview` descriptions.

```js
/**
 * @param {string} foo A description too long for one line continues
 *     with four spaces.
 */
```

### 7.5 Top/file-level comments

A file overview is optional, generally recommended for files consisting of more
than a single class definition. Copyright, author information and default visibility are also
optional. An overview orients unfamiliar readers and may describe contents,
dependencies or compatibility; wrapped lines are not indented.

```js
/**
 * @fileoverview Description of the file and its uses,
 * dependencies or compatibility.
 * @package
 */
```

### 7.6 Class comments

Classes, interfaces and records must have descriptions and appropriate template,
implements, visibility and other tags. The description should explain how/when
to use the class and considerations needed for correct use. Constructor text
may be omitted. With `class`, do not use `@constructor` or `@extends`, except
`@extends` for a generic superclass. Interface/record subclasses use `@extends`,
never the `extends` keyword.

```js
/**
 * A record of sequential values.
 * @extends {Iterator<TYPE>}
 * @record
 * @template TYPE
 */
class Listable {
  /** @return {TYPE} The next item. */
  next() {}
}
```

### 7.7 Enum and typedef comments

Every enum/typedef must have a preceding JSDoc with `@enum` / `@typedef`;
public ones also require descriptions. Items may have preceding JSDoc. Typedefs
are useful for short records and union/function/generic aliases, but avoid them
for large records with many fields: individual field documentation, templates
and recursion are unavailable. Prefer `@record` for large records.

```js
/**
 * A reusable union.
 * @typedef {!FruitType|!FruitTypeEnum}
 */
let CoolUnionType;
/**
 * Types of fruit.
 * @enum {string}
 */
const FruitTypeEnum = {
  /** A sour kind. */
  SOUR: 'sour',
};
```

### 7.8 Method and function comments

Methods and named functions must document parameter/return types, including
same-signature overrides. The `this` type should be documented when necessary. Return types
may be omitted when there are no nonempty `return` statements. Descriptions of
methods, parameters and returns may be omitted if obvious from the signature
or remaining JSDoc. Method descriptions start with a third-person verb phrase
(implied “This method ...”), not an imperative.

Overrides must have `@override` and explicit `@param` / `@return` annotations,
even when they refine no superclass types.

```js
/**
 * @param {string} param
 * @return {string}
 * @override
 */
overriddenMethod(param) {
  // ...
}
```

When documenting only types, inline signature JSDoc is optional and has no
tags: `function /** string */ foo(/** number */ arg) { ... }`. If descriptions
or tags are needed, use a single JSDoc above the method; for example,
value-returning methods need `@return`. Do not put tagged method JSDoc or
function descriptions inline:

```js
// Incorrect:
class MyClass {
  /** @return {string} */ foo() { /* ... */ }
}
/** Function description. */ function bar() {}
function /** Function description. */ baz() {}
```

Anonymous-function annotations are generally optional. If inference is
insufficient or explicit types improve readability, annotate like this:

```js
promise.then(
    /** @return {string} */
    (/** !Array<string> */ items) => {
      doSomethingWith(items);
      return items[0];
    });
```

### 7.9 Property comments

Property types must be documented. Private descriptions may be omitted when
name/type suffice. Comment publicly exported constants the same way.

```js
/**
 * Maximum number of attempts.
 * @const {number}
 */
MyClass.RETRY_COUNT = 33;
```

### 7.10 Type annotations

Types go on `@param`, `@return`, `@this`, `@type`, and optionally `@const`,
`@export` or visibility tags. Tag-attached types must be enclosed in braces.

#### 7.10.1 Nullability

`!` means non-null and `?` nullable; modifiers precede the type.

- Primitives (`string`, `number`, `boolean`, `symbol`, `undefined`, `null`) and
  function/record literals are non-null by default: omit redundant `!`; use `?`
  to allow null.
- Reference types (generally UpperCamelCase, including qualified names for
  classes, enums, records and typedefs) always require explicit `!` or `?`;
  names do not establish nullability.

```js
// Correct:
const /** ?MyObject */ myObject = null;
const /** number */ someNum = 5;
const /** ?number */ nullableNum = null;
const /** {foo: string, bar: number} */ record = getRecord();
const /** !MyTypeDef */ def = getDef();
// Incorrect types: MyObject, !number, number?, !{foo: string}, MyTypeDef.
```

#### 7.10.2 Type Casts

If inference is inaccurate and `goog.asserts` cannot remedy it, a type comment
may narrow an expression. Parentheses are required: `/** @type {number} */ (x)`.

#### 7.10.3 Template Parameter Types

Always supply template parameters, e.g. `!Object<string, !User>`,
`!Array<string>`, `!Promise<!Response>`, `!Promise<undefined>`,
`!Object<string, *>`, rather than bare `!Object`, `!Array`, `!Promise`.
**Exception:** parameters should not be used when `Object` is a hierarchy type
rather than a map-like structure.

#### 7.10.4 Function type expressions

Where a function definition is present, including anonymous functions and
function-valued const assignments, do not use a `function(...)` type expression.
Use `@param` / `@return` or inline annotations. Function type expressions are
needed inside typedefs/parameters/returns and for function variables/properties
not immediately initialized by a function definition.

Always give their return type explicitly; otherwise it defaults to unknown
(`?`). `function(): *` may return anything; `function(): undefined` returns
nothing. Do not use `function()` when one of these is intended.

```js
/** @private {function(string): string} */
this.idGenerator_ = googFunctions.identity;
```

#### 7.10.5 Whitespace

After each type comma/colon require one space or newline. Other newlines may
aid readability/80-column wrapping; these breaks should follow normal continuation/block indentation.
No other whitespace is allowed:

```js
// Correct:
/** @type {function(string): number} */
/** @type {{foo: number, bar: number}} */
/** @type {number|string} */
/** @type {!Object<string, string>} */
// Incorrect: function(string) : number; {foo:number,bar:number}; number | string.
```

### 7.11 Visibility annotations

`@private`, `@package`, `@protected` may appear in `@fileoverview` or on exported
symbols/properties. Never give visibility to function/module locals. Private
names may end in `_`.

## 8 Policies

### 8.1 Issues unspecified by Google Style: Be Consistent!

For any style question that isn't settled definitively by this specification,
prefer to do what the other code in the same file is already doing. If that
doesn't resolve the question, consider emulating the other files in the same
package.

### 8.2 Compiler warnings

#### 8.2.1 Use a standard warning set

As far as possible projects should use `--warning_level=VERBOSE`.

#### 8.2.2 How to handle a warning

Before doing anything, make sure you understand exactly what the warning is
telling you. If you're not positive why a warning is appearing, ask for help.

Once you understand the warning, attempt the following solutions in order:

1.  **First, fix it or work around it.** Make a strong attempt to actually
    address the warning, or find another way to accomplish the task that avoids
    the situation entirely.
2.  **Otherwise, determine if it's a false alarm.** If you are convinced that
    the warning is invalid and that the code is actually safe and correct, add a
    comment to convince the reader of this fact and apply the `@suppress`
    annotation.
3.  **Otherwise, leave a TODO comment.** This is a **last resort**. If you do
    this, **do not suppress the warning.** The warning should be visible until
    it can be taken care of properly.

#### 8.2.3 Suppress a warning at the narrowest reasonable scope

Warnings are suppressed at the narrowest reasonable scope, usually that of a
single local variable or very small method. Often a variable or method is
extracted for that reason alone.

Example

**Correct example:**

```js
/** @suppress {uselessCode} Unrecognized 'use asm' declaration */
function fn() {
  'use asm';
  return 0;
}
```

Even a large number of suppressions in a class is still better than blinding the
entire class to this type of warning.

### 8.3 Deprecation

Mark deprecated methods, classes or interfaces with `@deprecated` annotations. A
deprecation comment must include simple, clear directions for people to fix
their call sites.

### 8.4 Code not in Google Style

You will occasionally encounter files in your codebase that are not in proper
Google Style. These may have come from an acquisition, or may have been written
before Google Style took a position on some issue, or may be in non-Google Style
for any other reason.

#### 8.4.1 Reformatting existing code

When updating the style of existing code, follow these guidelines.

1.  It is not required to change all existing code to meet current style
    guidelines. Reformatting existing code is a trade-off between code churn and
    consistency. Style rules evolve over time and these kinds of tweaks to
    maintain compliance would create unnecessary churn. However, if significant
    changes are being made to a file it is expected that the file will be in
    Google Style.
2.  Be careful not to allow opportunistic style fixes to muddle the focus of a
    CL. If you find yourself making a lot of style changes that aren’t critical
    to the central focus of a CL, promote those changes to a separate CL.

#### 8.4.2 Newly added code: use Google Style

Brand new files use Google Style, regardless of the style choices of other files
in the same package.

When adding new code to a file that is not in Google Style, reformatting the
existing code first is recommended, subject to the advice in
[8.4.1 Reformatting existing code](#841-reformatting-existing-code).

If this reformatting is not done, then new code should be as consistent as
possible with existing code in the same file, but must not violate the style
guide.

### 8.5 Local style rules

Teams and projects may adopt additional style rules beyond those in this
document, but must accept that cleanup changes may not abide by these additional
rules, and must not block such cleanup changes due to violating any additional
rules. Beware of excessive rules which serve no purpose. The style guide does
not seek to define style in every possible scenario and neither should you.

### 8.6 Generated code: mostly exempt

Source code generated by the build process is not required to be in Google
Style. However, any generated identifiers that will be referenced from
hand-written source code must follow the naming requirements. As a special
exception, such identifiers are allowed to contain underscores, which may help
to avoid conflicts with hand-written identifiers.

## 9 Appendices

### 9.1 JSDoc tag reference

JSDoc controls documentation and tooling, including Closure's type system.

#### 9.1.1 Type annotations and other Closure Compiler annotations

Optional detailed references: [Closure annotations](https://github.com/google/closure-compiler/wiki/Annotating-JavaScript-for-the-Closure-Compiler)
and [Closure types](https://github.com/google/closure-compiler/wiki/Types-in-the-Closure-Type-System).
The enforceable rules in this guide are included above.

#### 9.1.2 Documentation annotations

The following additional tags are supported for documentation purposes.

##### 9.1.2.1 `@author` or `@owner` - *Not recommended.*

Syntax: `@author username@google.com (First Last)`. Not recommended. They
identify a file author/test owner and are generally only in `@fileoverview`.
The unit-test dashboard uses `@owner` to attribute results.

##### 9.1.2.2 `@bug`

Syntax: `@bug bugnumber`, e.g. `@bug 1234567`, identifies bugs regression-tested
by a test function. Multiple bugs should each have a separate `@bug` line.

##### 9.1.2.3 `@code` - *Deprecated. Do not use.*

Never use `{@code ...}`; use Markdown backticks for code, e.g. `BatchItem`.

##### 9.1.2.4 `@desc`

Syntax: `@desc Message description`, e.g. a description before a `goog.getMsg`
message. Wrapped descriptions do not indent.

##### 9.1.2.5 `@link`

Syntax: `{@link ...}` for generated-documentation cross-references, e.g.
`{@link BatchItem}`. For external links always use Markdown `[text](URL)`.

##### 9.1.2.6 `@see`

Syntax: `@see Link`, referencing a class/function/method, e.g. `@see #addSafely`,
`@see goog.Collect`, or `@see goog.RecklessAdder#add`.

##### 9.1.2.7 `@supported`

Syntax: `@supported Description`, used in a file overview to indicate supported
browsers, e.g. `@supported IE10+, Chrome, Safari`.

Other third-party JSDoc Toolkit tags are not part of valid Google style.

#### 9.1.3 Framework specific annotations

##### 9.1.3.1 `@ngInject` for Angular 1

This tag is specific to Angular 1.

##### 9.1.3.2 `@polymerBehavior` for Polymer

This tag is specific to Polymer; see [Polymer Pass](https://github.com/google/closure-compiler/wiki/Polymer-Pass).

#### 9.1.4 Notes about standard Closure Compiler annotations

##### 9.1.4.1 `@expose` - *Deprecated. Do not use.*

Use `@export` and/or `@nocollapse` instead.

##### 9.1.4.2 `@inheritDoc` - *Deprecated. Do not use.*

Use `@override` instead.

### 9.2 Commonly misunderstood style rules

- Neither copyright nor author credit is required or explicitly recommended.
- No hard rule orders class members.
- Empty blocks may usually use `{}`, subject to multi-block exceptions.
- Prefer wrapping at higher syntax levels.
- Non-ASCII is allowed in strings/comments/JSDoc, and recommended where more
  readable than escapes.

### 9.3 Style-related tools

#### 9.3.1 Closure Compiler

Performs type/other checks, optimizations and transformations such as ES5 lowering.

#### 9.3.2 `clang-format`

Its JavaScript output complies with this guide and includes some optional
readability choices. It is not required; authors/reviewers may change/request
changes to output and resolve disputes normally. Subtrees may opt into local
enforcement.

#### 9.3.3 Closure compiler linter

Checks missteps and anti-patterns.

#### 9.3.4 Conformance framework

Closure's framework can add project checks beyond defaults, e.g. forbidding
property/function access or unknown types, to enforce critical/security
restrictions or improve quality. See [JS Conformance Framework](https://github.com/google/closure-compiler/wiki/JS-Conformance-Framework).

### 9.4 Exceptions for legacy platforms

#### 9.4.1 Overview

These exceptions/additional rules apply **only where modern ECMAScript syntax
is unavailable**: `var`, `arguments`, and optional parameters without defaults
are allowed. They do not authorize old syntax in ordinary new modern code.

#### 9.4.2 Use `var`

##### 9.4.2.1 `var` declarations are NOT block-scoped

`var` is scoped to the beginning of its nearest function/script/module, not a
block. Closures in loops can therefore share the same final value:

```js
// Pitfall: logs 2, 2, 2, rather than 0, 1, 2.
for (var i = 0; i < 3; ++i) {
  var iteration = i;
  setTimeout(function() { console.log(iteration); }, i * 1000);
}
```

##### 9.4.2.2 Declare variables as close as possible to first use

Declarations should be as close as possible to first use. Do not declare a
variable inside a block if it is referenced outside that block:

```js
function countItems() {
  var count = 0;  // Not inside the loop.
  for (var x in y) {
    count++;
  }
  console.log(count);
}
```

##### 9.4.2.3 Use @const for constants variables

For global declarations that would use `const` if available, annotate `var`
with `@const`; this is optional for local variables.

#### 9.4.3 Do not use block scoped functions declarations

Do not declare functions in blocks: old runtimes disagree and ES5 strict mode
bans it. Instead initialize a `var` with a function expression:

```js
if (x) {
  var foo = function() {};  // Not: function foo() {}
}
```

#### 9.4.4 Dependency management with `goog.provide`/`goog.require`

##### 9.4.4.1 Summary

**`goog.provide` dependency management is deprecated.** These rules apply only
to pre-existing provide files. All new files should use `goog.module`, even in
projects with old provide files.

- Group provides first, requires second, with one blank line between lists.
- Sort entries alphabetically, uppercase first.
- Never wrap provide/require statements; exceed 80 columns if needed.
- Provide only top-level symbols. All class members should be in one file;
  provide only the top-level class, never each nested class/enum/constant/
  typedef/static method separately. Namespace members may also be provided.

```js
goog.provide('namespace.MyClass');
goog.provide('namespace.helperFoo');

goog.require('goog.dom');
goog.require('goog.dom.TagName');
goog.require('goog.dom.classes');
// Do not separately provide namespace.MyClass.Enum or .staticMethod.
// Namespace members such as foo.bar, foo.bar.CONSTANT and foo.bar.method may
// each be provided.
```

##### 9.4.4.2 Aliasing with `goog.scope`

**Deprecated:** new files should not use `goog.scope`, even in projects that
already do. It may shorten stable namespaced references in provide/require
code. At most one invocation is allowed per file, in global scope.

Put exactly one blank line before `goog.scope(function() {`, after provides,
requires or top-level comments. Close it on the file's last line, with exactly
two spaces before `// goog.scope`. Do not indent its contents: start at column
zero. Only alias names that will not be reassigned to other objects (typically
constructors/enums/namespaces), retaining the final global property name. Alias
new types after their constructor declaration.

```js
goog.provide('my.module.SomeType');

goog.require('goog.dom');
goog.require('goog.ui.Button');

goog.scope(function() {
var Button = goog.ui.Button;
var dom = goog.dom;
my.module.SomeType = function() {};
var SomeType = my.module.SomeType;
SomeType.prototype.findButton = function() {
  this.button = new Button(dom.getElement('my-button'));
};
});  // goog.scope
```

##### 9.4.4.3 `goog.forwardDeclare`

Prefer `goog.requireType` to break same-library cycles; unlike `goog.require`,
it may import a namespace before definition. Forward declarations follow the
same style as requires/requireTypes; sort their entire combined block
alphabetically. The legacy cross-library `goog.forwardDeclare` cycle pattern
should not be used; code should avoid these cycles through library splitting
or merging.

##### 9.4.4.4 `goog.module.get(name)`

A provide file depending on a module must both `goog.require` it and fetch its
exports with `goog.module.get('module.name')`, since normally there is no global
export name. `goog.module.get` alone does not create a build dependency.

##### 9.4.4.5 `goog.module.declareLegacyNamespace()`

**Transitional use only:** migrate a provide file and its consumers to modules.
Update consumers to `goog.module` and remove legacy declarations whenever
possible. If consumers cannot soon migrate, wrap the consumer file in
`goog.scope`, import via `goog.module.get`, then remove the module's legacy
declaration. A legacy declaration otherwise exposes the module namespace
globally like provide, allowing non-module consumers without `goog.module.get`.

## Sources and licensing

- `jsguide.md`: [fixed upstream source](https://github.com/google/styleguide/blob/572c673d9058242c049fbc8248b17029726c7afa/jsguide.md); [official published guide](https://google.github.io/styleguide/jsguide.html).

Upstream: Google Style Guides, by Google and the contributors or authors named in the source documents, at commit `572c673d9058242c049fbc8248b17029726c7afa`. The upstream repository README states [CC BY 3.0](https://creativecommons.org/licenses/by/3.0/) for documentation and [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0) for code samples. Preserve any more specific attribution or license notices in the original documents.

Local changes: this is an **editorially condensed adaptation**, not a verbatim
copy. Site templates and presentational HTML were removed; rules, exceptions
and representative examples were consolidated into one Markdown language file.
Repeated rationale and examples were shortened; some examples were edited.
The fixed source is available above for optional comparison. This adaptation is
unofficial; Google does not endorse this skill.
