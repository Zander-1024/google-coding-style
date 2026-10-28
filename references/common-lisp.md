# Google Common Lisp guides

This is an **editorially condensed adaptation** of Google's Common Lisp Style Guide
and Common Lisp Effective Programming at upstream commit
`572c673d9058242c049fbc8248b17029726c7afa`, by Google and the upstream contributors.
It retains the source sections, requirements, recommendations, key exceptions, and
selected examples while removing repeated explanations and background. The
Effective Programming material remains a supplement, not an additional set of
style rules. This is unofficial and is not endorsed by Google. Original sources,
official publications, attribution, and licenses are listed in
[Sources and licensing](#sources-and-licensing); consulting them is optional.

## Google Common Lisp Style Guide

### Introduction

This guide recommends formatting, structure, and stylistic choices; it is not a
Common Lisp tutorial. Balance these recommendations against consistency when
maintaining existing code. Upstream feedback can be filed in the
[Google styleguide issue tracker](https://github.com/google/styleguide/issues),
with `[Lisp]` in the title.

#### References

- [Effective Programming](#google-common-lisp-effective-programming), below, is
  the companion guide.
- [Practical Common Lisp](https://www.gigamonkeys.com/book/) introduces the language.
- [Common Lisp HyperSpec](https://www.lispworks.com/documentation/HyperSpec/Front/index.htm)
  is the language reference.
- [Norvig and Pitman's style guide](https://www.cs.umd.edu/~nau/cmsc421/norvig-lisp-style.pdf)
  offers further guidance, to be taken with a pinch of salt.

### Existing Code

Fix violations encountered during normal work when the fix is easy and does not
substantially enlarge the change; consider a separate change for the style fix.
You **must not** make mass style fixes without warning and coordinating with
other developers, because they complicate branch merges.

### Formatting

#### Punctuation, Spelling and Grammar

Use correct spelling in code and comments, and proper grammar in comments and
documentation. Comments should read as narrative text, with capitalization and
punctuation; complete sentences are often clearer. Short end-of-line comments
may be less formal, but keep their style consistent.

#### Line length

Format source so no line exceeds **100 characters**.

#### Form length

Prefer small, focused top-level forms. There is no hard limit, but above about
**40 lines**, consider breaking a function or macro up without harming the
program's structure. Also consider extraction when a long function becomes hard
to debug or a part is needed in several contexts.

#### Indentation

Use spaces, **not tabs**; configure your editor not to insert tabs. In Emacs:
`(setq-default indent-tabs-mode nil)`.

- Binding data should be indented four spaces when starting on a new line;
  otherwise it should align with the first binding-data form.
- Binding-form bodies should be indented two spaces when starting on a new line;
  otherwise they should align with the first body form.
- Function arguments should align with the function name when the first argument
  is on a new line; otherwise they should align with the first argument.
- Special parameters in declarations (`&key`, `&optional`, etc.) should align with
  the special-parameter declaration when starting on a new line, or with its first
  parameter otherwise.
- Prefer all function parameters on one line if they fit, or one per line.
- `loop` keywords and their arguments should be indented two spaces when the first
  argument starts on a new line, or align with the first argument otherwise.

Prefer SLIME's default `cl-indent` indentation (`slime-indentation` in
`slime-setup`). When necessary for consistency in an existing project, customize
`define-common-lisp-style` via `:variables` and `:indentation` (particularly for
macros with non-function syntax), and use `common-lisp-set-style` from a
`lisp-mode-hook`. See [SLIME contrib loading](https://www.common-lisp.net/project/slime/doc/html/Loading-Contribs.html).

Break complex calls between arguments so their number and boundaries stay clear.

**Incorrect:**

```lisp
(do-something first-argument second-argument (lambda (x)
  (frob x)) fourth-argument last-argument)
```

**Correct:**

```lisp
(do-something first-argument
              second-argument
              (lambda (x) (frob x))
              fourth-argument
              last-argument)

(multiple-value-bind (a b c d)
    (function-returning-four-values x y)
  (declare (ignore c))
  (something-using a)
  (also-using b d))
```

#### File Header

Begin every source file with a brief contents description. Its first code form
should be `(in-package #:package-name)`. Avoid file-level
`(declaim (optimize ...))` in favor of smaller scopes; any file-specific
optimization declaration not covered by an ASDF `:around-compile` hook belongs
after `in-package`.

```lisp
;;;; Variable length encoding for integers and floating point numbers.

(in-package #:varint)
(declaim #.*optimize-default*)
```

You should not put authorship information at the top: version control gives
better information. You should not put copyright information in individual source files,
**except** files intended for standalone dissemination.

#### Vertical white space

Put one blank line between top-level forms. It may be omitted between simple,
one-line, closely related definitions of the same kind, such as related
constants or type declarations.

Blank lines may separate parts of a complex function, but prefer splitting it
into smaller functions. If that is not possible, explain each separated part
with a `;;` comment.

#### Horizontal white space

- **Do not** put whitespace immediately after `(` or before `)`.
- Use one space between forms, including function and macro parameters.
- **Do not** put consecutive trailing parentheses on separate lines or leave
  right parentheses alone on a line. The exception is a top-level `eval-when`
  enclosing several definitions; add `; eval-when` after its lone closing paren.
- Do not add spaces to align forms in the middle of consecutive lines, **except**
  to emphasize an important symmetry that would otherwise be invisible.
- **Do** align nested forms that span multiple lines.

**Incorrect:**

```lisp
( defun factorial ( limit )
  ( let (( product 1 ))
    ( loop for i from 1 upto limit
          do (setf product ( * product i ) ) )
    product
  )
)

(let* ((low    1)
       (high   2)
       (sum    (+ (* low low) (* high high))))
  ...)
```

**Correct:**

```lisp
(defun factorial (limit)
  (let ((product 1))
    (loop for i from 1 upto limit
          do (setf product (* product i)))
    product))

(let* ((low 1)
       (high 2)
       (sum (+ (* low low) (* high high))))
  ...)

(defun munge (a b c)
  (* (+ a b)
     c))
```

### Documentation

#### Docstrings

Use documentation strings wherever the language permits, including top-level
functions, types, classes, variables, and macros, unless painfully
self-explanatory. Docstrings describe the API: how to use it, arguments, return
values, side effects, conditions signaled, and pitfalls. Explain intended meaning
at the appropriate abstraction level; implementation details and future work
belong in comments.

Use uppercase Lisp symbol names, including arguments, in docstrings. A long
docstring may start with a one-sentence summary. Type names may use a backquote
and closing apostrophe so Emacs highlights them and supports definition lookup.

```lisp
(defun small-prime-number-p (n)
  "True if N, an integer, is a prime number."
  ...)

(defgeneric clear-table (table)
  (:documentation
    "Like clrhash, empties TABLE of all associations, and returns TABLE."))
```

Each method whose specialization affects behavior beyond the generic's
documentation should be independently documented. A specialization that breaks the generic's contract
must be avoided; if one does, document that specialization.

#### Non-Obvious Code

- A non-obvious bug fix **must** have a comment explaining why the code is
  necessary; reference the bug number if available (recommended).
- **Comment complicated or obscure forms**, explaining the need for the
  complication, the purpose, and what the code does. Comments should also guide
  readers to the relevant parts of the code.
- Use a TODO for incomplete work, questions, breakage, danger, or other matters
  requiring attention. State the problem type, its nature, and possible remedies.
  Ideally include a bug number; a username or email is also acceptable so readers
  can find more information (usually use your own).
- Be specific about dates/releases; use **YYYY-MM-DD** dates.

```lisp
;; TODO(b/1234567): use circular lists for fizzbuzz
;; TODO(zippy@foo.com): Refactor to provide a better API.
;; TODO(brown): Remove this code after release 1.7 or before 2012-11-30.
```

Design DSLs to be readable by people familiar with the domain, and **document
all DSLs properly**, both declarations and uses. For terse DSLs, explain each
program unless its purpose is painfully obvious from context.

For regular expressions (e.g. CL-PPCRE), **always include a comment**, usually
`;;` on the preceding line, explaining at least what the regex does or why it is
used. Readers should be able to follow the logic without parsing the regex.

#### Comment semicolons

- `;;;` for file-level, section, or top-level-form comments.
- `;;` for comments between lines inside a top-level form.
- `;` for parenthetical end-of-line remarks. Separate these from code with
  spaces; try to align consecutive related end-of-line comments vertically.
- Include a space between the semicolons and the comment text.

```lisp
;;; Prime numbers
(defun small-prime-number-p (n)
  (cond ((< n 2) nil)
        ((= n 2) t) ; Special case
        ;; Test the remaining divisors.
        (t (loop for i from 3 upto (sqrt n) by 2
                 never (divisorp i n)))))
```

### Naming

#### Symbol guidelines

Use lowercase symbols with hyphens separating words and correct spelling. Common
Lisp case-converts symbol names to uppercase; do not try to distinguish symbols
by case. Do not escape characters to force lowercase **unless needed for
third-party interoperability**.

Do not substitute `/` or `.` for `-` **without a well-documented overarching
reason and permission from the developers reviewing the proposal**. Follow
[established punctuation conventions](https://www.cliki.net/Naming+conventions);
do not use other symbol punctuation.

Name symbols for their high-level **intent**, not their representation. Avoid
embedding `list`, `array`, or `hash-table` in variable names unless implementing
a generic algorithm for that representation. A variable representing a row can
be `row` even if its abstract type is implemented by a string. Introduce abstract
types (`defclass` or `deftype`) for new kinds of intent in protocols. Common
suffixes such as `-p` or `-v` may be exceptions, but usually prefer declarations
and type-specific accessors to convey types.

Use descriptive function, variable, and file names; do not sacrifice clarity to
save space. Avoid ambiguous abbreviations in wide-scope symbols (`defvar`,
`defun`, etc.); centrally document unfamiliar domain abbreviations. Limited-scope
variables may be abbreviated freely. Keep names consistent across functions:
pass `row` on as `row`, rather than renaming it `value`.

**Incorrect:** `*default-contextname*`, `*maximum-widget-cnt*`, `*nerr*`,
`*n-comp-conns*`, `*wgc-connections*`, `*pc-reader*`, `*cstmr-id*`.

**Correct:** `*default-context-name*`, `*maximum-widget-count*`,
`*price-count-reader*`, `*num-errors*`, `*num-dns-connections*`.

#### Global variables and constants

Global constant names should start and end with plus characters (`+like-this+`);
global variable names should start and end with asterisks (`*like-this*`, earmuffs),
so dynamically bound globals do not collide with locals.

Some projects use a leading, non-trailing `$` for parameters not normally
modified/bound, but available for experiments or exceptional cases. Follow that
convention consistently **only if the project already uses it**; otherwise avoid
it.

You should not fake global lexical variables using another global plus
`define-symbol-macro` **unless you first publish a library abstracting the trick**.

```lisp
(defconstant +hash-results+ #xbd49d10d10cbee50)
(defvar *maximum-search-depth* 100)
```

#### Predicate names

For functions returning a generalized Boolean whose value has **no meaning beyond
truth**, use a trailing `p` for single-word names, or `-p` for multiword names.
`nil` is false; any other value is true. Do not use this suffix for functions
with a meaningful non-Boolean result, such as `member`, even if usable as a
Boolean. For Boolean variables, the same suffix is suggested when no other
convenient name expresses their Boolean nature.

#### Omit library prefixes

Do not repeat a library/package name as a prefix within internal or external
symbol names: prefer `varint:length64` to `varint:varint-length64`.

Exceptions:

- Variables likely to clash in packages using this one may include a prefix,
  e.g. ASDF's `*asdf-verbose*` affects ASDF, not the whole program.
- Retain idiomatic concept names when they place the package name in a
  **non-prefix** position: `foobar:define-foobar`, `foobar:do-foobar`,
  `foobar:with-frobed-foobar`, `foobar:make-foobar`, `foobar:new-foobar`.

#### Package names

Choose consistent package names usable as symbol prefixes. For widely exposed
packages, a good rule of thumb is `<org>.<library>.<unit>`, e.g.
`acme.core.macro`. Separate hierarchy with dots and compound words with hyphens,
e.g. `ace.container.hash-table`; use lowercase even for `http`.

The last component usually becomes an alias in client code. Design exported
names to read well with it: `fifo:add-to`, `fifo:clear-all`, rather than
`fifo:fifo-clear-all`. Prefer singular package names, e.g. `string:prefixp`
rather than `strings:prefixp`. (Do not create global package nicknames; see
[Packages](#packages).)

#### File names

Use hyphens rather than underscores for Lisp and related source filenames.
Match the package hierarchy as far as possible; with one package per file,
`.../core/string.lisp` contains `ace.core.string`. Lisp sources end in `.lisp`;
C FFI stubs end in `-ci.cc`.

### Code structure and organization

#### Packages

Use packages to define namespaces and external API symbols; prefer **one package
per file**, with explicit dependencies in its definition and a hierarchical,
non-cyclic dependency structure.

You should never access another package's internal symbols using `::` in
production code; fix the interface and export genuinely public symbols. Exceptions are unit tests
of that package's internals, very temporary hacks, and REPL work. When API
implementations/extenders need internals that normal users do not, expose those
through a **second package** rather than exporting them from the normal API.

Design new packages for **qualified use** (`time:now`) rather than `:use`, avoiding
redundant exported names where possible. Necessary repetitions such as a
`time:time` type are fine. Exceptions are internal implementation packages and
well-known, widely used core packages; choose exported names accordingly.

For another package designed for unqualified use, prefer `:import-from` over
`:use` to make dependencies explicit; `:use` may be preferable when importing
many symbols.

Package nicknames **MUST NOT** be used: they pollute the global package namespace
and allow inconsistent naming by different users.

```lisp
;; Incorrect: global nicknames
(defpackage :regression-test
  (:nicknames :rtest #-foo :rt))
```

Your package **must not shadow Common Lisp (`common-lisp`/`cl`) symbols**, except
in extremely rare, well-justified cases:

- Explicitly replacing a standard symbol with a safer or more capable version.
- Exporting a justified conflicting name from a package not intended for `:use`,
  such as `log:error` or `log:warn`.

#### CLOS

**New uses of CLOS are disallowed; new `print-object` methods are allowed.**
This is the upstream guide's rule.

### Language usage guidelines

#### Mostly Functional Style

Use side effects only when they improve clarity/code. Prefer rebinding local
variables to modifying them, initialize slots when creating objects rather than
creating and then `setf`ing them, and make classes as immutable as possible.
Avoid slot setters where possible (slots remain accessible through `with-slots`).

#### Special variables

Use special/dynamically bound variables **sparingly**, only where unsurprising
and significantly beneficial. Each adds hidden state and interactions that
readers, tests, isolation, and backtraces must account for. Natural candidates
are “the current” database connection, data source, or request shared through a
processing chain, where explicit argument plumbing adds little clarity.

Treat specials as **per-thread** variables. Do not provide a default top-level
binding unless it is valid for **all** threads. Each thread needing the variable
should bind it explicitly so misuse is an unbound-variable error. Variables
with a default global value should usually be locally rebound at thread creation.

#### Assignment

Keep assignment conventions consistent within each package; choose a sensible
convention for a new one. Existing packages may group assignments in one
`setf`/`psetf`, or split them into separate forms; follow that package. For new
packages, prefer grouping:

```lisp
(setf foo bar
      baz quux)
```

Likewise follow the package's preference between `setf` everywhere and `setq`
where the place is a variable/symbol-macro. Prefer **`setf`** for new packages.
Consider solutions with fewer assignments in keeping with mostly functional style.

#### Assertions and Conditions

Use `assert`, `check-type`, and `etypecase` liberally to detect **internal bugs
only**. Assert invariants whose failure means broken software; invalid external
input must instead receive proper runtime handling. Prefer `etypecase` to
`typecase`, and `check-type` to type declarations for function inputs. Do not use
`assert` data/argument forms to select a condition type; debugging messages in
those forms are fine.

Validate input such as file contents and use application-specific **`error`**
conditions for bad data, requests, permissions, or unusual outcomes. Production
`error` calls **must specify a condition type**, never just a string, to support
internationalization. Use `(error 'foo-error ...)`, not the longer equivalent
`(error (make-condition 'foo-error ...))`.

A function that signals conditions should document that behavior in its contract,
which should clearly name the condition classes; subclasses are permitted.
Signaling an undocumented condition is a bug. Never return a
condition object among multiple values, or encode an unusual outcome in a
special return type; signal it with `error`.

- Prefer logging a warning to `warn`.
- **Never call `signal` directly**; use `error`.
- Do not use `throw`/`catch` when restarts suffice, or restarts when
  `handler-bind` suffices.
- Do not handle all conditions (`t`) or use `ignore-errors`; let unrelated and
  unknown conditions propagate. In the rare justified case of handling “all”
  conditions, handle **only `error`**, never `t` or `serious-condition`.
- **Never signal a condition or exit nonlocally out of a cleanup handler**, e.g.
  via `invoke-restart` from an `unwind-protect` cleanup.
- **Never clean up by resignaling**; it hides the original stack trace. Use
  `unwind-protect` so the original error propagates.

**Incorrect:**

```lisp
(handler-case
    (etd-process-blocks)
  (error (c)
    (reset-parser-values)
    (error c))) ; Hides the original error site
```

**Correct:**

```lisp
(unwind-protect
    (etd-process-blocks)
  (reset-parser-values))
```

#### Type Checking and Declarations

Use `check-type` on API contracts, special variables acting as implicit
arguments, and where it helps detect input bugs. Checking an implicit special
also tells readers the function uses it as an argument.

Use `(declare (type ...))` for **non-API helper contracts only**, and for
optimization in internal low-level functions. Declarations can tell a compiler
to **assume** a type without checking, depending on speed/safety settings, so do
not use them to declare API contracts. For optimization, follow
[Unsafe Operations](#unsafe-operations).

### Meta-language guidelines

#### Macros

Use macros to express repeated patterns clearly, reusing existing macros where
possible. **Never use a macro when a (possibly inline) function suffices**;
use `flet`/`labels` rather than `macrolet` when possible. Complex macros need an
expressiveness benefit that justifies their cost.

Macros should only process syntax. Minimize expanded code and move logic to
(possibly inline) implementation functions in **call-with style**: splice `&body`
forms into a lambda passed to the helper. The helper's name should prefix the outer name with `%`, e.g.
`foo` delegates to `%foo`.

Macros should generally be hygienic: bind caller-provided symbols or fresh
`gensym`s. Give `gensym` a readable string name; `alexandria:with-gensyms` is
convenient. **Clearly document** bindings of other symbols (including globals)
while caller code executes, and side effects in expansion forms not supplied by
the caller.

A supplied form should be **expanded once** and evaluated only as many times as
the caller expects (normally once for an argument). `alexandria:once-only` or
call-with style can enforce this. Do not duplicate `&body` forms. If several
branches need the same form, it can be placed in a local function called by each
branch. That function should have a unique gensym name with `flet` and should be
declared `dynamic-extent` if applicable.

Leave an argument list before the body even for parameterless body macros, to
allow future optional/keyword arguments without breaking calls:

**Incorrect:**

```lisp
(defmacro with-lights-on (&body b) ...)
(with-lights-on ...)
```

**Correct:**

```lisp
(defmacro with-lights-on (() &body b) ...)
(with-lights-on () ...)
;; A later version can accept keyword parameters in that same list.
```

**Do not replace a function with a regular macro for performance.** If profiling
justifies inlining, document the need/results and use
`(declaim (inline function-name))`. Compiler macros may provide performance
transformations, but can interfere with tracing the optimized function.

Avoid macro-defining macros; if necessary, document and comment them especially
clearly. **Do not install reader macros without developer consensus**. Use
`cl-syntax`, `named-readtables`, or similar controls; reader macros **must not
leak** into clients or other systems. Clients must be able to use your system
without adopting its reader macros.

#### Read-time evaluation

**Do not use `#.` when another idiom works**: use `eval-when` for compile-time side effects,
a macro for a compile-time expression, or `load-time-value` for load-time
computation. You should consider naming the value with `defconstant` or a variant, possibly
inside `eval-when`.

Any `#.` evaluation **MUST NOT have side effects or depend on variable global
state**. Treat it as a way to force important constant folding the compiler
cannot perform. Another use is macro-like expansion in places that are neither
expressions nor (quasi)quotations, such as lambda lists; if frequent, consider
macros that extend the consumers of that syntax instead.

#### EVAL

**Never use `eval` at runtime in production code.** Appropriate uses are rare;
you must consult reviewers. Use `symbol-value` for a symbol's value; often a macro is the
right solution. Do not use runtime `eval` to interpret even a supposedly safe
language subset; write a special-purpose evaluator instead.

Permitted contexts:

- Interactive development tools.
- Build infrastructure.
- Testing-framework backdoors (**never in production**).
- Macros folding constants at compile time.
- Macros registering definitions in metadata, where registration must happen
  both during compilation (for other macros) and in the expansion at load time.

In the last case, a non-top-level macro, `deftype` expansion, or macro helper may
not be able to emit the needed top-level definitions. Use
`asdf-finalizers:eval-at-toplevel` when needed: it evaluates immediately at
expansion time and saves the form for `(asdf-finalizers:final-forms)`, which you
**must include at the end of the compiled file, or before the form is needed**.
That ensures registration also occurs when loading a fasl without compiling;
ASDF-FINALIZERS reports an error if final forms are omitted.

#### INTERN and UNINTERN

**Never use `intern` or `unintern` at runtime**, and do not dynamically intern
new symbols. Interning can allocate permanent symbols, expose internals, leak
memory, enable denial of service, and create clashes. Uninterning can break
dynamic binding and debugging.

Do not intern a string merely to compare it with a keyword; use `string=` or
`string-equal` (`string-equal` accepts symbols via their `symbol-name`).

**Incorrect:** `(member (intern str :keyword) $keys)`.

**Correct:** `(member str $keys :test #'string-equal)`.

Compile-time interning in macros is allowed, but prefer abstractions such as
`alexandria:symbolicate` or `alexandria:format-symbol`.

### Data Representation

#### Using NIL

Avoid ambiguous representations:

- False: use `nil`; test with `not` or `null`.
- Empty list: use `nil`; test with `endp` for a known proper list, `null` otherwise.
- Unspecified value: `nil` is allowed only if unambiguous everywhere; otherwise
  use an explicit descriptive symbol/keyword.
- A value known not to exist: use an explicit descriptive symbol rather than `nil`.

**Do not introduce ambiguity**; if using `nil`, document the distinction well.
Often a second return value indicating whether the primary value was known/found
is better than an “unknown” sentinel. Database `null` and Lisp `nil` need not map
to each other; choose representations for their respective needs.

#### Lists

Use the appropriate data structure, not lists simply because Lisp makes them
easy. Lists should contain **like-typed elements**, and **must** suit the
algorithm's performance requirements: sequential traversal of the whole list.
Exceptions include lists known to stay very short (roughly **fewer than 16**
elements), macro/source processing that walks lists sequentially, and literal
constants converted into better structures at compile/load time. A short-named
builder for such literals is useful. Use suitable data-structure libraries such
as `cl-containers` or `lisp-interface-library` when needed; see
[Using Libraries](#using-libraries) if none suffices.

Prefer **proper** lists; use improper lists only with limited scope where
performance is critical.

- Proper lists: `first`, `second`, etc.; `rest` for the tail; `endp` for the end.
- Improper lists: `car`, `cdr`, and `consp` to test for the end.
- Manual destructuring may exceptionally use `cdadr` and related accessors (and
  `car`/`cdr` instead of `first`/`rest`). Prefer higher-level
  `destructuring-bind` or `optima:match` when appropriate. This should be rare
  outside alists; use structures/classes and tree libraries where suitable.

Do not represent sets as lists unless they are known to stay small. List access
is O(n), versus O(log n) for suitable balanced trees; for arbitrarily large sets,
use a proper set structure, e.g. from `lisp-interface-library`.

**Incorrect:** `(member foo (union list-1 list-2))` unnecessarily allocates, and
`union` can itself be O(n²).

**Correct:** `(or (member foo list-1) (member foo list-2))`.

#### Arrays

Use arrays for homogeneous collections requiring non-sequential/random access;
use `aref` instead of O(n) `elt` on a list. A small known list outside the critical
path is an exception.

#### Structures and Classes

Use a `defstruct` structure or `defclass` object for a heterogeneous group of
values. **Do not pass heterogeneous values in an ad hoc list** except when
destructuring a function's argument list or constructing arguments for `apply`.
Use multiple return values only for a **small number** that the caller will
consume immediately; use a structure/class when they will be passed together to
further functions. The source's separate [CLOS rule](#clos) still applies.

#### Pairs

Use cons pairs rarely, only for **exactly two closely related values of
homogeneous types**. Access them with `car` and `cdr`; represent absence with
`nil` and test it with `null` in this context.

### Proper Forms

Follow the prescribed uses of familiar functions, macros, and special forms.

#### Defining Functions

Avoid `&aux` and `&allow-other-keys`; keep accepted keywords explicit as part of
the contract. In generic methods, unused keyword arguments should be explicitly declared
`ignore`. **Do not use `&allow-other-keys` unless you intend to disable allowed-key
checking for all methods when this method matches.** An appropriate exception is
a wrapper forwarding a `&rest` plist to other functions that may vary. The
generic's contract belongs in `defgeneric`, not `defmethod`.

Avoid combining `&optional` and `&key` (keywords cannot be supplied after only
some optional arguments). If combined, optional defaults **must not be non-`nil`**.
For portable libraries, unused **required** method arguments should be declared
`ignorable`: implementations disagree about warnings for `ignore` or no declaration.

Avoid excessive nesting of binding forms; split the function into functions or
macros. For a truly single conceptual unit, consider `fare-utils:nest`. It is bad
form to use `nest` in ordinary functions with **four or fewer levels**, and bad
form not to use it in exceptional long functions with **ten or more**. Use
judgment and consult reviewers.

#### \#'FUN vs. 'FUN

Use **`#'fun`**, not `'fun`, for functional arguments (`mapcar`, `apply`, `:test`,
`:key`, etc.). `#'` refers to the lexical function object; quoting a symbol uses
its global `fdefinition` when called.

Exceptions: deliberately request dynamic linking to an updated global function,
or deliberately access a global function rather than a shadowing lexical one.
The latter should be rare; usually name the lexical function differently.

Use **`(lambda ...)` without `#'`**: the lambda macro already expands to the
function-quoted form, so the longer spelling adds nothing.

#### Conditional Expressions

- One alternative: `when`/`unless`. Two: `if`. Several: `cond`.
- Do not put `progn` in an `if` clause; use `cond`, `when`, or `unless`.
- Prefer `cond`, `case`, or variants to nested `if`s.
- Prefer `and`/`or` when more concise **and side-effect-free**, except that the
  final clause of an `or` may call `error`.
- Prefer **`ecase`/`etypecase`** to `case`/`typecase` to catch unexpected values.

`when`/`unless` return `nil` if their condition fails; use that property. An
explicit `if` returning `nil`, final `(t nil)` in `cond`, or `(otherwise nil)` in
`case` may emphasize that the return value matters and that case is expected.
Such fall-through clauses should be omitted when used only for side effects.

```lisp
;; Incorrect: silently returns NIL on unexpected X
(case x ((bar) :bar) ((baz) :baz))
;; Correct: reports the unexpected value
(ecase x ((bar) :bar) ((baz) :baz))
```

Do not use `ccase` or `ctypecase`; at minimum, do not use them in server processes
without robust error handling and protection against leaking sensitive data or
control. They are intended for interactive use. See the supplemental
[Case and Ecase pitfalls](#case-and-ecase).

#### Comparison Operators

Prefer **`eql`** to `eq`, all else equal, except in performance-critical low-level
code. `eq` is equivalent to `eql` plus type assumptions; treat its optimization
like [Unsafe Operations](#unsafe-operations), since numbers/characters may fail
`eq` unexpectedly.

Use string comparison bounds instead of allocating substrings:
`(string-equal s1 s2 :start1 2 :end1 6)`, rather than
`(string-equal (subseq s1 2 6) s2)`.

Use `zerop`, `plusp`, or `minusp` instead of comparisons with `0` or `0.0`. See
[the supplemental comparison guidance](#comparison-operators-1) for semantics.

#### Iteration

Prefer `dolist`/`dotimes` over `loop` in simple cases that do not use bindings,
collection, block return, or other loop facilities. Prefer `loop`'s `with` over
an enclosing `let` when it removes nesting; `let` is fine when clearer,
including to return a bound variable rather than use `finally (return ...)`.

Do not set iteration variables in an iteration macro's body, particularly
`dotimes`. Prefer unadorned current-package symbols for `loop` keywords, except
to match surrounding code or make a macro-generated expansion more readable.

#### I/O

Server code **must use logging**, not standard streams (`*standard-output*`,
`*error-output*`, etc.), for messages/debug output. **Do not use `print-object`
for user communication**: it is for debugging. Changes to its methods must not
break public interfaces.

Prefer one `format` to a sequence of `write-xxx` forms, so its control string can
later be parameterized. Prefer `write-char` for one character, and
`princ-to-string` to `(format nil "~A" value)`.

Use `~<Newline>` or `~@<Newline>` in format strings to stay within 100-column
editors and indent clauses readably. Use `"~:@(~A~)"`/`"~(~A~)"` for case changes
instead of applying `string-upcase`/`string-downcase` to format parameters.

## Google Common Lisp Effective Programming

### Introduction

This is the upstream **supplement** to the [Style Guide](#google-common-lisp-style-guide).
It covers optimization techniques and pitfalls important to new Lisp programmers,
rather than defining additional style requirements. Normative words below retain
the source supplement's force within that guidance; they do not turn all of its
performance advice into mandatory style rules.

### Optimization

#### Using Libraries

Prefer existing libraries to reimplementation. Reimplement only for a
**well-documented performance need** or specialized functionality they lack.
You **MUST NOT** start a new library without establishing that no existing one
can be fixed or completed to meet the need.

Whichever old/new library is chosen, you **MUST get permission to incorporate
third-party code**, discuss its use on the appropriate mailing list, and have domain/Lisp
ecosystem experts review it where available. Be ready to explain why it is
better suited than alternatives. Libraries with incompatible licenses must not
be treated as available; understand licensing issues or consult someone who does.

#### Avoid Allocation

Ensure objects stop being reachable when no longer needed, and avoid unnecessary
consing/allocation. Garbage collection does not remove reachable unused data,
and excessive allocation harms performance.

An array's fill pointer does not limit its underlying storage: resetting it to
zero leaves stored objects reachable beyond the pointer. `vector-pop` likewise
does **not** clear the returned element from storage.

#### Unsafe Operations

**Do not define or use unsafe operations without both profiling showing the need
and careful documentation explaining why they are safe.** Invariants may include
type/range restrictions; violating them can produce overflow, undefined behavior,
crashes, or silent wrong answers. Fixnum-only operators such as `i+` are examples.

Unsafe operations should be restricted to **internal package functions**; document unsafe
argument cases and optimization declarations. Use `check-type` in exported functions to sanitize
arguments so internals never receive illegal inputs.

High `speed`/low `safety` optimization with type declarations can create unsafe
operations. On compilers with type inference, those declarations can also expose
compile-time bugs. Safer development settings may interpret them as runtime
assertions to locate errors; using those slower settings for production defeats
the purpose of this performance technique.

#### DYNAMIC-EXTENT

Treat `dynamic-extent` as an **unsafe operation**: profile its use and document
why it is safe. It asserts that every object ever held by the variable (including
its subobjects), or the local function's closure, becomes inaccessible after the
innermost declaring function exits **by any means**. The implementation may then
stack-allocate `&rest` lists, other lists/vectors/structures, or closures.
An escaping object can cause later crashes, hangs, or silent corruption; future
changes can invalidate an initially correct assertion.

Use it **sparingly**, only when:

1. There is good reason to expect noticeable overall performance benefit
   (usually very frequent allocation, such as an inner loop).
2. The lifetime assertion is absolutely clear.
3. Future changes are unlikely to invalidate it.

Modern generational GC already handles ephemeral objects efficiently. Checking a
local function's uses/passing destinations is relatively tractable, so be wary
of declaring functions; tracking every possible variable value and subobject,
including future changes, is much harder, so be **extremely wary** of declaring
variables.

The source recognizes a measurement tradeoff for reusable libraries: if you are
**very sure** no value/subobject escapes, but the savings are unclear and hard
to measure, adding the declaration can be preferable. This qualified exception
does not remove the need to reason about safety or likely future changes.

#### REDUCE vs APPLY

Prefer `reduce` over `apply` to a freshly consed list **when the operator's
semantics make the results equivalent**. For example:

**Incorrect:**

```lisp
(apply #'+ (mapcar #'acc frobs))
```

**Correct:**

```lisp
(reduce #'+ frobs :key #'acc :initial-value 0)
```

This avoids allocation and `call-arguments-limit`/stack risks on long lists.
Use `apply` when it has the required semantics and `reduce` does not; where
appropriate it can be preferable for readability/performance.

**Do not use `reduce` if it needlessly worsens complexity**: `reduce` over
concatenation can be O(n²), and over `append` is O(n²) unless `:from-end t`.
In such cases, **do not substitute `apply` over `strcat`/`append` either**; use a
suitable library abstraction (contributing one if necessary), such as
`uiop:reduce/strcat`, that handles the operation efficiently without exposing
implementation details.

#### NCONC and MAPCAN

You should almost never use `nconc`, `mapcan`, or `loop`'s `nconc`. Prefer `append`,
`alexandria:appendf` when updating a variable, `alexandria:mappend` in place of
`mapcan`, and `loop`'s `append`.

You should probably not rely on games with the current cons's `cdr`; if you do,
you must **include a prominent comment explaining the `nconc` use**, and you should
probably reconsider the representation.
`nconc` rarely improves asymptotic time or unshared-list space complexity over
`append`, but increases bug risk. If that small cost is limiting, prefer a
constant-time-append sequence or accumulate a tree and flatten it once in linear
time.

The only exception permits **rare, documented low-level performance-critical**
uses, where lists have been vetted as appropriate because they stay short, and
the producer's **explicit contract promises fresh lists**. A constant quoted or
backquoted expression does not qualify.

### Pitfalls

#### Format Conditional Directives

Remember the differing `format` conditional parameters:

| Directive | Argument handling and choice |
| --- | --- |
| `~[a~;b~;c~]` | Consumes an integer; selects a zero-based clause. Out of range prints nothing. `~:;` before the last clause makes it the default, e.g. `~[Siamese~;Manx~;Persian~:;Alley~] Cat`. |
| `~:[a~;b~]` | Consumes one argument; `nil` selects the first clause, any other value the second. |
| `~@[Siamese ~a~]` | A true next argument selects the clause **without consuming** the argument (the clause usually uses it); a false argument is consumed and prints nothing. |
| `~#[ none~; ~s~; ~s and ~s~]` | Selects by the number of remaining arguments; otherwise works like the unparameterized directive. |

A combined example:

```lisp
"Items:~#[ none~; ~S~; ~S and ~S~:;~@{~#[~; and~] ~S~^ ,~}~]."
```

#### Comparison Operators

Understand the distinctions among `eq`, `eql`, `equal`, `equalp`, and type-specific
`=`, `char=`, `char-equal`, `string=`, `string-equal`. See also the
[Style Guide comparison rules](#comparison-operators).

- You should use **`eql` for object/symbol identity**. **Never use `eq` for numbers or
  characters**; `eql` objects of those types need not be `eq`.
- `char=` is case-sensitive; `char-equal` ignores case. Likewise `string=`
  versus `string-equal` for strings.
- For string `search`, the `:test` receives **characters**, so use `char=` or
  `char-equal`. `string=`/`string-equal` may appear to work but can allocate a
  one-character string per comparison and be much slower.
- **Use `=` for numerical equality**, unless `0`, `0.0`, and `-0.0` should differ,
  when `eql` is appropriate. **Do not normally compare floating-point values for
  exact equality**; compare absolute differences to a threshold to accommodate
  rounding error.
- Monetary amounts should use decimal/rational numbers to avoid binary
  floating-point rounding issues. A library such as
  [wu-decimal](https://github.com/Wukix/wu-decimal) may help; follow
  [Using Libraries](#using-libraries) if it does not suffice.

#### Case and Ecase

Use `case`/`ecase` **only for numbers, characters, and symbols** (including
Booleans/keywords). They compare with `eql`: strings, pathnames, or structures
may not match as expected, and `1` differs from `1.0`.

**Do not add single quotes** to keys: `'bar` means `(quote bar)`, making both
`quote` and `bar` keys, so the first such clause can unexpectedly catch `quote`.

**Incorrect:**

```lisp
(ecase x
  ('bar :bar)
  ('baz :baz))
```

**Correct:**

```lisp
(ecase x
  ((bar) :bar)
  ((baz) :baz))
```

In `case`, **use `otherwise`, not `t`, for the fallback**. To match the symbols
literally, use `((t) ...)` and `((nil) ...)`; a bare `nil` key list matches
nothing, and a bare `t` fallback matches anything. For a Boolean-only mapping:

```lisp
(ecase x
  ((nil) :bar)
  ((t) :quux))
```

This still errors on non-Booleans.

#### Defining Constants

Use proper defining forms. **Do not use `defconstant` for values other than
numbers, characters, or symbols** (including Booleans/keywords). SBCL signals a
condition if a constant is redefined to a non-`eql` value; equal-content strings
or lists need not be `eql`. Follow the project's alternative consistently.

Open-source libraries may use `alexandria:define-constant`; its `:test` option
selects equality:

```lisp
;; Incorrect: reloads may supply non-EQL values
(defconstant +google-url+ "https://www.google.com/")
(defconstant +valid-colors+ '(red green blue))

;; Correct for open-source code
(alexandria:define-constant +google-url+ "https://www.google.com/"
                            :test #'string=)
(alexandria:define-constant +valid-colors+ '(red green blue))
```

With optimizing implementations, constants may be impossible to redefine without
uninterning and recompiling all clients. If a value might change during the
server process's lifetime, including live patches, consider `defparameter`,
`defvar`, or a suitable global-lexical variant instead; retaining `+plus+` names
can document constant intent. `load-time-value` can also avoid defining constants.

#### EVAL-WHEN

Almost always use **all three** situations:

```lisp
(eval-when (:compile-toplevel :load-toplevel :execute)
  ...)
```

Unless doing truly advanced macrology, this is the valid combination. **Use it
for functions, types, classes, constants, variables, etc. used by macros**.
Omitting `:execute` usually breaks source loading; omitting `:load-toplevel`
usually breaks loading dependent files or interactive compilation unless the
current file was compiled in the same session. A compile-time-only change to
readtables or compiler settings is an exception to the latter.

**Do not depend on compile-time and runtime effects being either shared or
isolated**: macro expansion may occur in a different process. Valid uses of macro
variables include:

- Definition dictionaries/metadata: if needed at runtime or in other files,
  register definitions **both at compile time and in load-time expansion code**.
  Top-level defining forms usually emit this code; non-top-level cases may use
  `load-time-value`, or in extreme cases `asdf-finalizers:eval-at-toplevel`.
- Temporary compile-time data confined to the same file and cleaned up at its
  compilation's end, including `*readtable*` and optimization settings. ASDF
  `:around-compile` hooks can manage these variables.

#### Pathnames

Use **UIOP** where appropriate for portable pathname operations, and account for
these pitfalls:

- Lisp pathname syntax varies by implementation/OS and has quoting for pathname
  characters as well as string quoting. Use `uiop:native-namestring` and
  `uiop:parse-native-namestring` to convert to/from OS syntax. SBCL/CCL handle
  this well; some implementations fall back to `namestring`/`parse-namestring`
  and may fail to escape wildcard characters.
- `merge-pathnames` can unexpectedly replace an absolute pathname's host with
  one inherited from `*default-pathname-defaults*` when a relative pathname was
  created. Prefer `uiop:merge-pathnames*` or `uiop:subpathname`, particularly
  when expecting Unix/Windows-style relative paths.
- `directory` behavior for wildcards, subdirectories, symlinks, etc. is not
  portable. UIOP has useful but imperfect abstractions; IOLib is a more complete
  portable solution, though its Windows support lags.
- **Logical pathnames should not be used in portable code**: implementations have bugs or
  incompatible extensions, and standard character restrictions limit usefulness.
  Prefer `asdf:system-relative-pathname`, `uiop:subpathname`, or
  `uiop:parse-unix-namestring`.
- Build-time paths may be invalid when a saved image runs. Reset irrelevant paths
  and initialize search paths from the **current environment**. Reset ASDF paths
  with `uiop:clear-configuration`; use UIOP's pre-dump hooks to reset or
  `makunbound` relevant variables.

#### SATISFIES

Take care with `satisfies` in type specifiers. Most implementations cannot
optimize it alone; many can optimize a structural type combined with it, e.g.
`(and integer (satisfies prime-number-p))` rather than just
`(satisfies prime-number-p)`.

Type-specifier **`and` is symmetric**, permits reordering, and is **not** the
expression language's left-to-right short circuit. The `integer` test does not
protect the predicate from non-integer arguments. Implementations may also call
the predicate on assorted objects at compile time.

Therefore every `satisfies` predicate **MUST accept objects of any type**, and
**MUST be defined in `eval-when`**, along with any variables/functions it uses.

**Incorrect:** a predicate that calls `abs` without first checking `integerp`,
or is not available at compile time.

**Correct:**

```lisp
(eval-when (:compile-toplevel :load-toplevel :execute)
  (defun prime-number-p (n)
    (when (integerp n)
      (let ((m (abs n)))
        (if (<= m *prime-number-cutoff*)
            (small-prime-number-p m)
            (big-prime-number-p m))))))
```

Its dependencies must also satisfy the compile-time availability requirement.
Even the standard's `(and integer (satisfies evenp))` example is unsafe:
`evenp` errors rather than returning `nil` for non-integers.

If `deftype` expands to a `satisfies` using a dynamically generated function,
remember that expansion timing is uncontrolled, the expansion cannot include
expression-language code/function definitions, and use may occur in a different
process. **Creating the function only via `eval` as an expansion side effect is
insufficient**. Use `asdf-finalizers:eval-at-toplevel` as described in
[EVAL](#eval) so the definition also exists when loading compiled code.

## Sources and licensing

- `lispguide.md`: [fixed upstream source](https://github.com/google/styleguide/blob/572c673d9058242c049fbc8248b17029726c7afa/lispguide.md);
  [official published guide](https://google.github.io/styleguide/lispguide.html).
- `lispprogrammingguide.md`: [fixed upstream source](https://github.com/google/styleguide/blob/572c673d9058242c049fbc8248b17029726c7afa/lispprogrammingguide.md);
  [official published guide](https://google.github.io/styleguide/lispprogrammingguide.html).

Upstream: Google Style Guides, by Google and the contributors or authors named in
the source documents, at commit `572c673d9058242c049fbc8248b17029726c7afa`.
The upstream repository README states
[CC BY 3.0](https://creativecommons.org/licenses/by/3.0/) for documentation and
[Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0) for code samples.
Preserve any more specific attribution or license notices in the originals.

Local changes: this is an **editorially condensed adaptation**, combining both
sources in one Markdown file. Repeated rules, background, and redundant examples
were shortened; selected examples were retained or adapted, and malformed sample
parentheses were corrected. Site templates and presentational HTML were removed,
and links/headings were normalized. It is not a verbatim or complete reproduction.
Effective Programming remains supplemental guidance. Original-source links are
optional references. This is an unofficial adaptation; Google does not endorse
this skill.
