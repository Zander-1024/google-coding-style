# Google Python Style Guide

Editorially condensed adaptation of Google's Python guide at commit
`572c673d9058242c049fbc8248b17029726c7afa`. Rules, their strength, exceptions,
and representative examples are retained; repeated rationale and examples are
shortened. This unofficial adaptation is not endorsed by Google. Sources and
licenses are listed at the end; the rules can be used without opening them.

## 1 Background

These are Google's Python dos and don'ts. A
[Vim settings file](https://google.github.io/styleguide/google_python_style.vim)
is available; Emacs defaults should be fine. Many teams use
[Black](https://github.com/psf/black) or [Pyink](https://github.com/google/pyink)
to automate formatting.

## 2 Python Language Rules

### 2.1 Lint

Run `pylint` over your code using Google's
[pylintrc](https://google.github.io/styleguide/pylintrc). It finds bugs and style
problems, though occasional warnings are incorrect. Suppress inappropriate
warnings so other issues remain visible. Use symbolic warning names
(`empty-docstring`; Google-specific names start with `g-`) and explain a
suppression when its name does not make the reason clear. Prefer
`pylint: disable` to deprecated `pylint: disable-msg`.

```python
def do_PUT(self):  # WSGI name, so pylint: disable=invalid-name
    ...
```

Use `pylint --list-msgs` to list warnings and `pylint --help-msg=invalid-name`
for details. Suppress unused arguments by deleting them at the function's
start, always with an explanatory comment ("Unused." is sufficient):

```python
def viking_cafe_order(spam: str, beans: str, eggs: str | None = None) -> str:
    del beans, eggs  # Unused by vikings.
    return spam + spam + spam
```

Naming unused arguments `_` or `unused_...`, or assigning them to `_`, is
allowed but no longer encouraged: renaming breaks keyword callers and these
forms do not enforce non-use.

### 2.2 Imports

Use `import` for packages and modules, not individual types, classes or
functions:

- Use `import x` for a package or module.
- Use `from x import y` where `x` is the package prefix and `y` the unprefixed
  module name.
- Use `from x import y as z` when two modules have the same name, `y` conflicts
  with a top-level name or common public API parameter (e.g. `features`), `y` is
  inconveniently long, or `y` is too generic in context (e.g.
  `from storage.file_system import options as fs_options`).
- Use `import y as z` only for a standard abbreviation, e.g. `import numpy as np`.

```python
# Correct:
from sound.effects import echo
echo.EchoFilter(input, output, delay=0.7, atten=4)
```

Do not use relative import names, even within the same package; use its full
package name to avoid unintentionally importing a package twice.

##### 2.2.4.1 Exemptions

Symbols from `typing`, `collections.abc` and `typing_extensions` used for static
analysis and type checking are exempt. See [Imports For Typing](#31912-imports-for-typing).

### 2.3 Packages

All new code should import each module by its full package name. Either
of these forms is correct:

```python
import absl.flags
from doctor.who import jodie
_FOO = absl.flags.DEFINE_string(...)
```

```python
from absl import flags
from doctor.who import jodie
_FOO = flags.DEFINE_string(...)
```

`import jodie` is incorrect for a sibling `doctor/who/jodie.py`: it means a
third-party or top-level package. You should not assume the main binary's directory is
in `sys.path`, even when it happens to be in some environments.

### 2.4 Exceptions

Exceptions are allowed but must be used carefully:

- Use built-in exception classes when appropriate, e.g. `ValueError` for a
  violated precondition such as an invalid argument.
- Do not replace conditionals or precondition validation with `assert`, or
  make assertions critical to application logic: removing them must not break
  the code, since their conditions may not be evaluated. Assertions are okay
  and expected for verifying expectations in pytest tests.
- Libraries/packages may define exceptions; they must inherit from an existing
  exception class. Their names should end in `Error` without repetition such
  as `foo.FooError`.
- Never use bare `except:`, or catch `Exception` or `StandardError`, unless
  re-raising or creating an isolation point that records and suppresses
  exceptions (e.g. the outermost block protecting a thread from crashing).
  Catch-all handlers also catch unexpected errors, exits and interrupts.
- Minimize code in each `try`/`except` block to avoid hiding unrelated errors.
- Use `finally` for code that must run whether or not `try` raises, such as
  cleanup.

```python
# Correct: validate explicitly; a removable assertion checks an invariant.
if minimum < 1024:
    raise ValueError(f'Min. port must be at least 1024, not {minimum}.')
port = self._find_next_open_port(minimum)
if port is None:
    raise ConnectionError(f'Could not connect on port {minimum} or higher.')
assert port >= minimum, f'Unexpected port {port} when minimum was {minimum}.'
return port

# Incorrect: logic and return-type checking depend on removable assertions.
assert minimum >= 1024
port = self._find_next_open_port(minimum)
assert port is not None
return port
```

Document `ConnectionError` as part of that interface; you should not document
its precondition `ValueError` in `Raises:`, making API-misuse behavior contractual.

### 2.5 Mutable Global State

Avoid mutable global state (module values or class attributes mutated during
execution). It breaks encapsulation and may change behavior at import time.

In rare justified cases, it should be declared at module level or as a class
attribute and made internal with a leading `_`. Explain the design in a comment
or a document linked from one. Necessary external access must use public functions
or class methods.

Module constants are permitted and encouraged; names must use capitals and
underscores: `_MAX_HOLY_HANDGRENADE_COUNT = 3` (internal) or
`SIR_LANCELOTS_FAVORITE_COLOR = 'blue'` (public). See [Naming](#316-naming).

### 2.6 Nested/Local/Inner Classes and Functions

Inner classes are fine. Avoid nested functions or local classes except when
closing over a local value other than `self` or `cls`. Nested functions have
read-only access to enclosing variables; nesting makes direct testing harder.
Do not nest merely to hide a function from module users: define it at module
level with a leading `_`, so tests can still access it.

### 2.7 Comprehensions & Generator Expressions

List/dict/set comprehensions and generator expressions are allowed for simple
cases. Multiple `for` clauses or filter expressions are not permitted.
Optimize for readability rather than conciseness; use loops for complex cases.

```python
# Correct:
result = [mapping_expr for value in iterable if filter_expr]
result = [
    is_valid(metric={'key': value})
    for value in interesting_iterable
    if a_longer_filter_expression(value)
]
result = []
for x in range(10):
    for y in range(5):
        if x * y > 10:
            result.append((x, y))
squares = (x**2 for x in range(10))
unique_names = {user.name for user in users if user is not None}

# Incorrect:
result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]
```

### 2.8 Default Iterators and Operators

Use default iterators and operators (`in`, `not in`) for supporting types, such as lists,
dictionaries and files. Prefer iterator methods to methods returning lists.
You should not mutate a container while iterating over it.

```python
# Correct:
for key in adict: ...
if obj in alist: ...
for line in afile: ...
for k, v in adict.items(): ...

# Incorrect:
for key in adict.keys(): ...
for line in afile.readlines(): ...
```

### 2.9 Generators

Use generators as needed. They preserve local state between yields and avoid
building a whole list, but keep locals alive until exhausted or collected.
Use `Yields:` instead of `Returns:` in generator docstrings. Force cleanup if a
generator manages an expensive resource; wrapping it in a context manager is a
good approach (see [PEP 533](https://peps.python.org/pep-0533/)).

### 2.10 Lambda Functions

Lambdas are allowed for one-liners. Prefer generator expressions over
`map()` or `filter()` with a lambda. If a lambda spans multiple lines or exceeds
60–80 characters, a regular [nested function](#216-lexical-scoping) might be
better. For common operations use `operator` functions, e.g. `operator.mul`
rather than `lambda x, y: x * y`.

### 2.11 Conditional Expressions

Conditional expressions are okay for simple cases. Each portion—the true
expression, condition and false expression—must fit on one line. Use a full
`if` statement for more complex cases.

```python
# Correct:
one_line = 'yes' if predicate(value) else 'no'
slightly_split = ('yes' if predicate(value)
                  else 'no, nein, nyet')
long_result = (
    'yes, true, affirmative, confirmed, correct'
    if predicate(value)
    else 'no, false, negative, nay')

# Incorrect: splits a portion or breaks between else and its expression.
bad_line_breaking = ('yes' if predicate(value) else
                     'no')
portion_too_long = ('yes'
                   if some_long_module.some_long_predicate_function(
                       really_long_variable_name)
                   else 'no')
```

### 2.12 Default Argument Values

Default argument values are okay in most cases, but are evaluated once when
the module loads. Do not use mutable defaults, even with type annotations;
also beware values intended to reflect call time or flags parsed later.

```python
# Correct:
def foo(a, b: Sequence | None = None):
    if b is None:
        b = []
def foo(a, b: Sequence = ()):  # Immutable empty tuple is okay.
    ...

# Incorrect:
def foo(a, b=[]): ...
def foo(a, b: Mapping = {}): ...  # May reach unchecked mutating code.
def foo(a, b=time.time()): ...  # Module-load time, not call time.
def foo(a, b=_FOO.value): ...  # sys.argv has not yet been parsed.
```

### 2.13 Properties

Properties may control getting/setting attributes or compute trivially derived
values. They should only be used when necessary. Property implementations must
match normal attribute-access expectations: cheap, straightforward and unsurprising. Simply wrapping both a read and
write of an internal attribute is not allowed: make the attribute public.
Otherwise follow [Getters and Setters](#315-getters-and-setters).

Properties should be created with `@property`; a manually implemented property descriptor
is a [power feature](#219-power-features). Do not use properties for computations
that a subclass may ever need to override and extend.

### 2.14 True/False Evaluations

Prefer implicit truth tests: `if foo:` rather than `if foo != []:`. Empty
values such as `0`, `None`, `[]`, `{}` and `''` are false. Caveats:

- Always test `None` with `is None` or `is not None`; a supplied value may be
  false without being `None`.
- Never compare a boolean to `False` using `==`; use `if not x:`. To distinguish
  `False` from `None`, combine tests, e.g. `if not x and x is not None:`.
- For sequences prefer `if seq:` / `if not seq:` to `if len(seq):` /
  `if not len(seq):` (or `len(seq) == 0`).
- Comparing a known integer to `0` is allowed except for results of `len()`;
  implicit false may accidentally treat `None` as zero. Prefer
  `if i % 10 == 0:` over `if not i % 10:`.
- String `'0'` is true.
- Numpy arrays may raise in a boolean context. Prefer `.size` for emptiness,
  e.g. `if not users.size:`.

```python
# Correct:
def f(x=None):
    if x is None:
        x = []

# Incorrect: replaces other false values too.
def f(x=None):
    x = x or []
```

### 2.16 Lexical Scoping

Lexical scoping is okay. Nested functions can refer to enclosing variables;
assignment makes a name local to its block, even before that assignment, unless
declared global. Enclosing bindings are resolved from program text, not from
the caller. Keep the potential for surprising captured variables in mind:

```python
# Correct:
def get_adder(summand1: float) -> Callable[[float], float]:
    def adder(summand2: float) -> float:
        return summand1 + summand2
    return adder

# Pitfall:
i = 4
def foo(x: Iterable[int]):
    def bar():
        print(i, end='')
    for i in x:  # Local to foo; also the binding seen by bar.
        print(i, end='')
    bar()
# foo([1, 2, 3]) prints the values 1 2 3 3, not 1 2 3 4.
```

### 2.17 Function and Method Decorators

Use decorators judiciously when clearly advantageous. They transform a
function (`@deco` means `function = deco(function)`) and execute at definition
time, including import time for module-level objects; failures may be
unrecoverable.

- Decorators should follow function import/naming rules, and their docstrings
  should clearly state that they are decorators. Write unit tests for decorators.
- Avoid external dependencies in a decorator itself, such as files, sockets
  or database connections: these may be unavailable when importing, including
  from `pydoc`. A validly parameterized decorator should, as far as possible,
  always succeed. See [Main](#317-main) for top-level execution cautions.
- Never use `staticmethod` unless an existing library API forces it. Otherwise
  write a module-level function.
- Use `classmethod` only for named constructors or class-specific routines that
  modify necessary global state, e.g. a process-wide cache.

### 2.18 Threading

Do not rely on built-in types' atomicity, including atomic variable assignment.
Operations on dictionaries, for example, may invoke Python `__hash__` or `__eq__`
methods and cease to be atomic.

Prefer `queue.Queue` for inter-thread communication. Otherwise use `threading`
and its locking primitives. Prefer condition variables / `threading.Condition`
over lower-level locks.

### 2.19 Power Features

Avoid custom metaclasses, bytecode access, on-the-fly compilation, dynamic
inheritance, object reparenting, import hacks, reflection (e.g. some uses of
`getattr()`), system-internal modification, custom cleanup in `__del__`, and
similar power features: they make code difficult to read, understand and debug.
Standard library modules/classes that internally use them are okay, e.g.
`abc.ABCMeta`, `dataclasses`, and `enum`.

### 2.20 Modern Python: from \_\_future\_\_ imports

Use of `from __future__ import` statements is encouraged to enable newer
semantics per file on earlier runtimes. Use other future imports as you see
fit. For code that may run on Python 3.5 rather than only 3.7+, import:

```python
from __future__ import generator_stop
```

Remove a future import only when confident all execution environments are new
enough to enable its feature without it. Keep it even if the file does not
currently use that feature, to prevent later changes depending on old behavior.
See [future statement definitions](https://docs.python.org/3/library/__future__.html).

### 2.21 Type Annotated Code

Python supports type annotations on arguments, returns and variables:

```python
def func(a: int) -> list[int]: ...
a: SomeType = some_func()
```

Type-check at build time with a tool such as [pyrefly](https://pyrefly.org/).
When feasible, annotations normally belong in source; third-party or extension
modules may use [`.pyi` stubs](https://peps.python.org/pep-0484/#stub-files).

Enabling type analysis when updating code is strongly encouraged. When adding
or modifying public APIs, include annotations and enable build-system checking.
If side effects such as incorrect inference prevent adoption, authors are
encouraged to add a comment with a TODO or bug link explaining the blocker in
the BUILD file or code, as appropriate. Keep annotations up to date.

## 3 Python Style Rules

### 3.1 Semicolons

Do not terminate your lines with semicolons, and do not use semicolons to put
two statements on the same line.

### 3.2 Line length

Maximum line length is *80 characters*.

Explicit exceptions to the 80 character limit:

-   Long import statements.
-   URLs, pathnames, or long flags in comments.
-   Long string module-level constants not containing whitespace that would be
    inconvenient to split across lines such as URLs or pathnames.
-   Pylint disable comments. (e.g.: `# pylint: disable=invalid-name`)

Do not use a backslash for
[explicit line continuation](https://docs.python.org/3/reference/lexical_analysis.html#explicit-line-joining).

Instead, make use of Python's
[implicit line joining inside parentheses, brackets and braces](http://docs.python.org/reference/lexical_analysis.html#implicit-line-joining).
If necessary, you can add an extra pair of parentheses around an expression.

Note that this rule doesn't prohibit backslash-escaped newlines within strings
(see [below](#310-strings)).

**Correct example:**

```python
Yes: foo_bar(self, width, height, color='black', design=None, x='foo',
             emphasis=None, highlight=0)
```

**Correct example:**

```python

Yes: if (width == 0 and height == 0 and
         color == 'red' and emphasis == 'strong'):

     (bridge_questions.clarification_on
      .average_airspeed_of.unladen_swallow) = 'African or European?'

     with (
         very_long_first_expression_function() as spam,
         very_long_second_expression_function() as beans,
         third_thing() as eggs,
     ):
       place_order(eggs, beans, spam, beans)
```

**Incorrect example:**

```python

No:  if width == 0 and height == 0 and \
         color == 'red' and emphasis == 'strong':

     bridge_questions.clarification_on \
         .average_airspeed_of.unladen_swallow = 'African or European?'

     with very_long_first_expression_function() as spam, \
           very_long_second_expression_function() as beans, \
           third_thing() as eggs:
       place_order(eggs, beans, spam, beans)
```

When a literal string won't fit on a single line, use parentheses for implicit
line joining.

**Correct example:**

```python
x = ('This will build a very long long '
     'long long long long long long string')
```

Prefer to break lines at the highest possible syntactic level. If you must break
a line twice, break it at the same syntactic level both times.

**Correct example:**

```python
Yes: bridgekeeper.answer(
         name="Arthur", quest=questlib.find(owner="Arthur", perilous=True))

     answer = (a_long_line().of_chained_methods()
               .that_eventually_provides().an_answer())

     if (
         config is None
         or 'editor.language' not in config
         or config['editor.language'].use_spaces is False
     ):
       use_tabs()
```

**Incorrect example:**

```python
No: bridgekeeper.answer(name="Arthur", quest=questlib.find(
        owner="Arthur", perilous=True))

    answer = a_long_line().of_chained_methods().that_eventually_provides(
        ).an_answer()

    if (config is None or 'editor.language' not in config or config[
        'editor.language'].use_spaces is False):
      use_tabs()

```

Within comments, put long URLs on their own line if necessary.

**Correct example:**

```python
Yes:  # See details at
      # http://www.example.com/us/developer/documentation/api/content/v2.0/csv_file_name_extension_full_specification.html
```

**Incorrect example:**

```python
No:  # See details at
     # http://www.example.com/us/developer/documentation/api/content/\
     # v2.0/csv_file_name_extension_full_specification.html
```

Make note of the indentation of the elements in the line continuation examples
above; see the [indentation](#34-indentation) section for explanation.

[Docstring](#381-docstrings) summary lines must remain within the 80 character
limit.

In all other cases where a line exceeds 80 characters, and the
[Black](https://github.com/psf/black) or
[Pyink](https://github.com/google/pyink) auto-formatter does not help bring the
line below the limit, the line is allowed to exceed this maximum. Authors are
encouraged to manually break the line up per the notes above when it is
sensible.

### 3.3 Parentheses

Use parentheses sparingly.

It is fine, though not required, to use parentheses around tuples. Do not use
them in return statements or conditional statements unless using parentheses for
implied line continuation or to indicate a tuple.

**Correct example:**

```python
Yes: if foo:
         bar()
     while x:
         x = bar()
     if x and y:
         bar()
     if not x:
         bar()
     # For a 1 item tuple the ()s are more visually obvious than the comma.
     onesie = (foo,)
     return foo
     return spam, beans
     return (spam, beans)
     for (x, y) in dict.items(): ...
```

**Incorrect example:**

```python
No:  if (x):
         bar()
     if not(x):
         bar()
     return (foo)
```

### 3.4 Indentation

Indent your code blocks with *4 spaces*.

Never use tabs. Implied line continuation should align wrapped elements
vertically (see [line length examples](#32-line-length)), or use a hanging
4-space indent. Closing (round, square or curly) brackets can be placed at the
end of the expression, or on separate lines, but then should be indented the
same as the line with the corresponding opening bracket.

**Correct example:**

```python
Yes:   # Aligned with opening delimiter.
       foo = long_function_name(var_one, var_two,
                                var_three, var_four)
       meal = (spam,
               beans)

       # Aligned with opening delimiter in a dictionary.
       foo = {
           'long_dictionary_key': value1 +
                                  value2,
           ...
       }

       # 4-space hanging indent; nothing on first line.
       foo = long_function_name(
           var_one, var_two, var_three,
           var_four)
       meal = (
           spam,
           beans)

       # 4-space hanging indent; nothing on first line,
       # closing parenthesis on a new line.
       foo = long_function_name(
           var_one, var_two, var_three,
           var_four
       )
       meal = (
           spam,
           beans,
       )

       # 4-space hanging indent in a dictionary.
       foo = {
           'long_dictionary_key':
               long_dictionary_value,
           ...
       }
```

**Incorrect example:**

```python
No:    # Stuff on first line forbidden.
       foo = long_function_name(var_one, var_two,
           var_three, var_four)
       meal = (spam,
           beans)

       # 2-space hanging indent forbidden.
       foo = long_function_name(
         var_one, var_two, var_three,
         var_four)

       # No hanging indent in a dictionary.
       foo = {
           'long_dictionary_key':
           long_dictionary_value,
           ...
       }
```

#### 3.4.1 Trailing commas in sequences of items?

Trailing commas in sequences of items are recommended only when the closing
container token `]`, `)`, or `}` does not appear on the same line as the final
element, as well as for tuples with a single element. The presence of a trailing
comma is also used as a hint to our Python code auto-formatter
[Black](https://github.com/psf/black) or
[Pyink](https://github.com/google/pyink) to direct it to auto-format the
container of items to one item per line when the `,` after the final element is
present.

**Correct example:**

```python
Yes:   golomb3 = [0, 1, 3]
       golomb4 = [
           0,
           1,
           4,
           6,
       ]
```

**Incorrect example:**

```python
No:    golomb4 = [
           0,
           1,
           4,
           6,]
```

### 3.5 Blank Lines

Two blank lines between top-level definitions, be they function or class
definitions. One blank line between method definitions and between the docstring
of a `class` and the first method. No blank line following a `def` line. Use
single blank lines as you judge appropriate within functions or methods.

Blank lines need not be anchored to the definition. For example, related
comments immediately preceding function, class, and method definitions can make
sense. Consider if your comment might be more useful as part of the docstring.

### 3.6 Whitespace

Follow standard typographic rules for the use of spaces around punctuation.

No whitespace inside parentheses, brackets or braces.

**Correct example:**

```python
Yes: spam(ham[1], {'eggs': 2}, [])
```

**Incorrect example:**

```python
No:  spam( ham[ 1 ], { 'eggs': 2 }, [ ] )
```

No whitespace before a comma, semicolon, or colon. Do use whitespace after a
comma, semicolon, or colon, except at the end of the line.

**Correct example:**

```python
Yes: if x == 4:
         print(x, y)
     x, y = y, x
```

**Incorrect example:**

```python
No:  if x == 4 :
         print(x , y)
     x , y = y , x
```

No whitespace before the open paren/bracket that starts an argument list,
indexing or slicing.

**Correct example:**

```python
Yes: spam(1)
```

**Incorrect example:**

```python
No:  spam (1)
```

**Correct example:**

```python
Yes: dict['key'] = list[index]
```

**Incorrect example:**

```python
No:  dict ['key'] = list [index]
```

No trailing whitespace.

Surround binary operators with a single space on either side for assignment
(`=`), comparisons (`==, <, >, !=, <>, <=, >=, in, not in, is, is not`), and
Booleans (`and, or, not`). Use your better judgment for the insertion of spaces
around arithmetic operators (`+`, `-`, `*`, `/`, `//`, `%`, `**`, `@`).

**Correct example:**

```python
Yes: x == 1
```

**Incorrect example:**

```python
No:  x<1
```

Never use spaces around `=` when passing keyword arguments or defining a default
parameter value, with one exception:
[when a type annotation is present](#3194-default-values), *do* use spaces
around the `=` for the default parameter value.

**Correct example:**

```python
Yes: def complex(real, imag=0.0): return Magic(r=real, i=imag)
Yes: def complex(real, imag: float = 0.0): return Magic(r=real, i=imag)
```

**Incorrect example:**

```python
No:  def complex(real, imag = 0.0): return Magic(r = real, i = imag)
No:  def complex(real, imag: float=0.0): return Magic(r = real, i = imag)
```

Don't use spaces to vertically align tokens on consecutive lines, since it
becomes a maintenance burden (applies to `:`, `#`, `=`, etc.):

**Correct example:**

```python
Yes:
  foo = 1000  # comment
  long_name = 2  # comment that should not be aligned

  dictionary = {
      'foo': 1,
      'long_name': 2,
  }
```

**Incorrect example:**

```python
No:
  foo       = 1000  # comment
  long_name = 2     # comment that should not be aligned

  dictionary = {
      'foo'      : 1,
      'long_name': 2,
  }
```

### 3.7 Shebang Line

Most `.py` files do not need to start with a `#!` line. Start the main file of a
program with `#!/usr/bin/env python3` (to support virtualenvs) or
`#!/usr/bin/python3` per [PEP-394](https://peps.python.org/pep-0394/).

This line is used by the kernel to find the Python interpreter, but is ignored
by Python when importing modules. It is only necessary on a file intended to be
executed directly.

### 3.8 Comments and Docstrings

Use the appropriate style for module, function, method and inline documentation.

#### 3.8.1 Docstrings

A docstring is the first statement of a package, module, class or function;
tools extract it through `__doc__`. Always use `"""`, per
[PEP 257](https://peps.python.org/pep-0257/). A docstring should start with a
summary line terminated by a period, question mark or exclamation point. The
summary must fit one physical line of at most 80 characters. Further documentation
is encouraged; it must follow a blank line and start aligned with the first
line's opening quote.

#### 3.8.2 Modules

Every file should contain the appropriate project license boilerplate. Files
should start with a docstring describing the module's contents and usage; optionally
describe exported classes/functions and show usage:

```python
"""A one-line summary of the module or program.

An overall description of the contents and usage.

Typical usage example:

  foo = ClassFoo()
  bar = foo.function_bar()
"""
```

##### 3.8.2.1 Test modules

Test modules need no module docstring; `unittest.TestCase` subclasses need no
class docstring and their `test_` methods need no method docstring. Docstrings
should be included when they add information, e.g. how to run the test, unusual setup or
external dependencies. Docstrings that only repeat the name should be omitted, such as
`"""Tests for foo.bar."""` or `"""Tests that bar is applied to foo."""` on
`test_foo_is_barred`.

#### 3.8.3 Functions and Methods

Here "function" includes methods, generators and properties. A docstring is
mandatory for every public API, nontrivially sized function, or function with
non-obvious logic. It should enable callers to use the function without reading
its code: describe calling syntax, semantics and relevant side effects (e.g.
mutating arguments). Put implementation details irrelevant to callers in code
comments.

Descriptive (`"""Fetches rows from a Bigtable."""`) and imperative (`"""Fetch
rows from a Bigtable."""`) styles are both allowed; the style should be consistent within a file.
A `@property` docstring should describe an attribute/argument, e.g. `"""The
Bigtable path."""`, rather than `"""Returns the Bigtable path."""`.

Relevant aspects should be documented in the following sections. Each heading
ends with a colon; section bodies should maintain a hanging indent of two or four
spaces, consistently within a file. Sections
may be omitted when the name/signature and a one-line docstring describe the
function sufficiently.

- **Args:** List each parameter by name. A description should follow each name,
  separated by a colon and a space or newline. Long descriptions use a hanging
  indent two or four spaces beyond the name, consistent with the file. Descriptions
  should include required types absent from annotations. Varargs should be listed
  as `*foo` and `**bar`.
- **Returns:** (or **Yields:** for generators) Describe return semantics and
  type details absent from annotations. This section may be omitted for a function
  returning only `None`; it may also be omitted if the opening sentence starts with "Return",
  "Returns", "Yield" or "Yields" and fully describes the value. Describe a
  returned tuple as one tuple, e.g. "A tuple (mat_a, mat_b), where mat_a is ...,
  and mat_b is ...", rather than individually named return values as in older
  NumPy style. These auxiliary names need not match internal names. For a
  generator, the `Yields:` section should describe the object returned by `next()`, not the generator object.
- **Raises:** List interface-relevant exceptions with descriptions, using the
  same name, colon and hanging-indent form as `Args:`. You should not document
  exceptions caused by violating the documented API, since that would make
  behavior under misuse part of the API.

```python
def fetch_smalltable_rows(
    table_handle: smalltable.Table,
    keys: Sequence[bytes | str],
    require_all_keys: bool = False,
) -> Mapping[bytes, tuple[str, ...]]:
    """Fetches rows from a Smalltable.

    String keys will be UTF-8 encoded.

    Args:
        table_handle: An open smalltable.Table instance.
        keys: Keys of the rows to fetch. Strings will be UTF-8 encoded.
        require_all_keys: If True, return only rows with values for all keys.

    Returns:
        A dict mapping byte keys to rows represented as tuples of strings,
        e.g. {b'Serak': ('Rigel VII', 'Preparer')}. A missing key means the
        row was not found (and require_all_keys must have been False).

    Raises:
        IOError: An error occurred accessing the smalltable.
    """
```

The following line-break form is also allowed:

```python
    Args:
      table_handle:
        An open smalltable.Table instance.
      keys:
        Keys of the rows to fetch.
```

A decorated function's docstring must describe its behavior after decoration,
which callers observe. For example, a function decorated with
`@contextlib.contextmanager` that yields a remote file should document a
**Returns:** value of "A context manager providing a reference to a remote
file", rather than documenting its undecorated generator behavior.

##### 3.8.3.1 Overridden Methods

An overriding method explicitly decorated with `@override` needs no docstring
unless it materially refines the base contract or needs additional details
(e.g. extra side effects); then document at least the differences. Without
`@override`, a docstring is required. Do not add trivial `"""See base class."""`
documentation to an `@override` method.

```python
from typing import override

class Parent:
    def do_something(self):
        """Parent method, including its contract."""

class Child(Parent):
    @override
    def do_something(self):
        pass
```

#### 3.8.4 Classes

Classes should have a docstring immediately below the definition. It should start
with a one-line summary of what an instance represents; this also applies to
exceptions. Avoid redundant wording like "Class that ..."; use
`"""The address of a cheese shop."""`. An exception should describe what it
represents (`"""No more cheese is available."""`), rather than when it occurs
(`"""Raised when no more cheese is available."""`).

Public attributes except properties should be documented in an `Attributes:` section,
formatted like function `Args:`. A decorated class's docstring must describe
its behavior after decoration.

```python
class SampleClass:
    """A sample object with a spam preference.

    Attributes:
        likes_spam: Whether we like SPAM.
        eggs: The count of eggs laid.
    """

    def __init__(self, likes_spam: bool = False):
        """Initializes the instance based on spam preference.

        Args:
          likes_spam: Whether the instance likes SPAM.
        """
        self.likes_spam = likes_spam
        self.eggs = 0

    @property
    def butter_sticks(self) -> int:
        """The number of butter sticks we have."""
```

#### 3.8.5 Block and Inline Comments

Tricky code you would need to explain during review should be commented. Put a few lines
before complicated operations and an end-of-line comment on non-obvious ones.
Inline comments should start at least two spaces after code, with `#` then at least one
space before text. Explain intent; never merely describe the Python code.

```python
# Correct:
if i & (i-1) == 0:  # True if i is 0 or a power of 2.

# Incorrect: merely narrates the code.
# Now go through b and check that whenever i occurs, the next element is i+1.
```

#### 3.8.6 Punctuation, Spelling, and Grammar

Use readable narrative text with proper capitalization, punctuation, spelling
and grammar. Complete sentences are often clearer. Short inline comments may
be less formal; keep their style consistent.

### 3.10 Strings

Use an
[f-string](https://docs.python.org/3/reference/lexical_analysis.html#f-strings),
the `%` operator, or the `format` method for formatting strings, even when the
parameters are all strings. Use your best judgment to decide between string
formatting options. A single join with `+` is okay but do not format with `+`.

**Correct example:**

```python
Yes: x = f'name: {name}; score: {n}'
     x = '%s, %s!' % (imperative, expletive)
     x = '{}, {}'.format(first, second)
     x = 'name: %s; score: %d' % (name, n)
     x = 'name: %(name)s; score: %(score)d' % {'name':name, 'score':n}
     x = 'name: {}; score: {}'.format(name, n)
     x = a + b
```

**Incorrect example:**

```python
No: x = first + ', ' + second
    x = 'name: ' + name + '; score: ' + str(n)
```

Avoid using the `+` and `+=` operators to accumulate a string within a loop. In
some conditions, accumulating a string with addition can lead to quadratic
rather than linear running time. Although common accumulations of this sort may
be optimized on CPython, that is an implementation detail. The conditions under
which an optimization applies are not easy to predict and may change. Instead,
add each substring to a list and `''.join` the list after the loop terminates,
or write each substring to an `io.StringIO` buffer. These techniques
consistently have amortized-linear run-time complexity.

**Correct example:**

```python
Yes: items = ['<table>']
     for last_name, first_name in employee_list:
         items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
     items.append('</table>')
     employee_table = ''.join(items)
```

**Incorrect example:**

```python
No: employee_table = '<table>'
    for last_name, first_name in employee_list:
        employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
    employee_table += '</table>'
```

Be consistent with your choice of string quote character within a file. Pick `'`
or `"` and stick with it. It is okay to use the other quote character on a
string to avoid the need to backslash-escape quote characters within the string.

**Correct example:**

```python
Yes:
  Python('Why are you hiding your eyes?')
  Gollum("I'm scared of lint errors.")
  Narrator('"Good!" thought a happy Python reviewer.')
```

**Incorrect example:**

```python
No:
  Python("Why are you hiding your eyes?")
  Gollum('The lint. It burns. It burns us.')
  Gollum("Always the great lint. Watching. Watching.")
```

Prefer `"""` for multi-line strings rather than `'''`. Projects may choose to
use `'''` for all non-docstring multi-line strings if and only if they also use
`'` for regular strings. Docstrings must use `"""` regardless.

Multi-line strings do not flow with the indentation of the rest of the program.
If you need to avoid embedding extra space in the string, use either
concatenated single-line strings or a multi-line string with
[`textwrap.dedent()`](https://docs.python.org/3/library/textwrap.html#textwrap.dedent)
to remove the initial space on each line:

**Incorrect example:**

```python
  No:
  long_string = """This is pretty ugly.
Don't do this.
"""
```

**Correct example:**

```python
  Yes:
  long_string = """This is fine if your use case can accept
      extraneous leading spaces."""
```

**Correct example:**

```python
  Yes:
  long_string = ("And this is fine if you cannot accept\n" +
                 "extraneous leading spaces.")
```

**Correct example:**

```python
  Yes:
  long_string = ("And this too is fine if you cannot accept\n"
                 "extraneous leading spaces.")
```

**Correct example:**

```python
  Yes:
  import textwrap

  long_string = textwrap.dedent("""\
      This is also fine, because textwrap.dedent()
      will collapse common leading spaces in each line.""")
```

Note that using a backslash here does not violate the prohibition against
[explicit line continuation](#32-line-length); in this case, the backslash is
[escaping a newline](https://docs.python.org/3/reference/lexical_analysis.html#string-and-bytes-literals)
in a string literal.

#### 3.10.1 Logging

For logging functions that expect a pattern-string (with %-placeholders) as
their first argument: Always call them with a string literal (not an f-string!)
as their first argument with pattern-parameters as subsequent arguments. Some
logging implementations collect the unexpanded pattern-string as a queryable
field. It also prevents spending time rendering a message that no logger is
configured to output.

**Correct example:**

```python
  Yes:
  import tensorflow as tf
  logger = tf.get_logger()
  logger.info('TensorFlow Version is: %s', tf.__version__)
```

**Correct example:**

```python
  Yes:
  import os
  from absl import logging

  logging.info('Current $PAGER is: %s', os.getenv('PAGER', default=''))

  homedir = os.getenv('HOME')
  if homedir is None or not os.access(homedir, os.W_OK):
    logging.error('Cannot write to home directory, $HOME=%r', homedir)
```

**Incorrect example:**

```python
  No:
  import os
  from absl import logging

  logging.info('Current $PAGER is:')
  logging.info(os.getenv('PAGER', default=''))

  homedir = os.getenv('HOME')
  if homedir is None or not os.access(homedir, os.W_OK):
    logging.error(f'Cannot write to home directory, $HOME={homedir!r}')
```

#### 3.10.2 Error Messages

Error messages (such as: message strings on exceptions like `ValueError`, or
messages shown to the user) should follow three guidelines:

1.  The message needs to precisely match the actual error condition.

2.  Interpolated pieces need to always be clearly identifiable as such.

3.  They should allow simple automated processing (e.g. grepping).

**Correct example:**

```python
  Yes:
  if not 0 <= p <= 1:
    raise ValueError(f'Not a probability: {p=}')

  try:
    os.rmdir(workdir)
  except OSError as error:
    logging.warning('Could not remove directory (reason: %r): %r',
                    error, workdir)
```

**Incorrect example:**

```python
  No:
  if p < 0 or p > 1:  # PROBLEM: also false for float('nan')!
    raise ValueError(f'Not a probability: {p=}')

  try:
    os.rmdir(workdir)
  except OSError:
    # PROBLEM: Message makes an assumption that might not be true:
    # Deletion might have failed for some other reason, misleading
    # whoever has to debug this.
    logging.warning('Directory already was deleted: %s', workdir)

  try:
    os.rmdir(workdir)
  except OSError:
    # PROBLEM: The message is harder to grep for than necessary, and
    # not universally non-confusing for all possible values of `workdir`.
    # Imagine someone calling a library function with such code
    # using a name such as workdir = 'deleted'. The warning would read:
    # "The deleted directory could not be deleted."
    logging.warning('The %s directory could not be deleted.', workdir)
```

### 3.11 Files, Sockets, and similar Stateful Resources

Explicitly close files, sockets and similar resources when done, including
database connections, `mmap` mappings, h5py files and matplotlib figure windows.
Do not couple cleanup to object destruction or rely on finalizers: runtime
cleanup timing varies and lingering references may keep resources alive.
Unclosed resources can exhaust limits, prevent filesystem operations, or allow
accidental use after their logical lifetime.

Prefer `with`:

```python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print(line)
```

For file-like objects without context-manager support, use
`contextlib.closing()`:

```python
import contextlib

with contextlib.closing(urllib.urlopen("http://www.python.org/")) as front_page:
    for line in front_page:
        print(line)
```

In rare cases where context management is infeasible, documentation must
clearly explain how resource lifetime is managed.

### 3.12 TODO Comments

Use `TODO` comments for code that is temporary, a short-term solution, or
good-enough but not perfect.

A `TODO` comment begins with the word `TODO` in all caps, and a parenthesized
link to a resource that contains the context, ideally a bug reference. A bug
reference is preferable because bugs are tracked and have follow-up comments.
Follow this piece of context with a colon and an explanatory string.

The purpose is to have a consistent `TODO` format that can be searched to find
out how to get more details.

**Correct example:**

```python
# TODO(crbug.com/192795): Investigate cpufreq optimizations.
```

Old style, formerly recommended, but discouraged for use in new code:

```python
# TODO: crbug.com/192795 - Investigate cpufreq optimizations.
```

Avoid adding TODOs that refer to an individual or team as the context:

**Incorrect example:**

```python
# TODO(@yourusername): File an issue and use a '*' for repetition.
```

If your `TODO` is of the form "At a future date do something" make sure that you
either include a very specific date ("Fix by November 2009") or a very specific
event ("Remove this code when all clients can handle XML responses.") that
future code maintainers will comprehend. Issues are ideal for tracking this.

### 3.13 Imports formatting

Imports should be on separate lines; there are
[exceptions for `typing` and `collections.abc` imports](#31912-imports-for-typing).

E.g.:

**Correct example:**

```python
Yes: from collections.abc import Mapping, Sequence
     import os
     import sys
     from typing import Any, NewType
```

**Incorrect example:**

```python
No:  import os, sys
```

Imports are always put at the top of the file, just after any module comments
and docstrings and before module globals and constants. Imports should be
grouped from most generic to least generic:

1.  Python future import statements. For example:

    **Correct example:**

    ```python
    from __future__ import annotations
    ```

    See [above](#220-modern-python-from-__future__-imports) for more information about those.

2.  Python standard library imports. For example:

    **Correct example:**

    ```python
    import sys
    ```

3.  [third-party](https://pypi.org/) module or package imports. For example:

    **Correct example:**

    ```python
    import tensorflow as tf
    ```

4.  Code repository sub-package imports. For example:

    **Correct example:**

    ```python
    from otherproject.ai import mind
    ```

5.  **Deprecated:** application-specific imports that are part of the same
    top-level sub-package as this file. For example:

    **Correct example:**

    ```python
    from myproject.backend.hgwells import time_machine
    ```

    You may find older Google Python Style code doing this, but it is no longer
    required. **New code is encouraged not to bother with this.** Simply treat
    application-specific sub-package imports the same as other sub-package
    imports.

Within each grouping, imports should be sorted lexicographically, ignoring case,
according to each module's full package path (the `path` in
`from path import ...`). Code may optionally place a blank line between import
sections.

**Correct example:**

```python
import collections
import queue
import sys

from absl import app
from absl import flags
import bs4
import cryptography
import tensorflow as tf

from book.genres import scifi
from myproject.backend import huxley
from myproject.backend.hgwells import time_machine
from myproject.backend.state_machine import main_loop
from otherproject.ai import body
from otherproject.ai import mind
from otherproject.ai import soul

# Older style code may have these imports down here instead:
#from myproject.backend.hgwells import time_machine
#from myproject.backend.state_machine import main_loop
```

### 3.14 Statements

Generally only one statement per line.

However, you may put the result of a test on the same line as the test only if
the entire statement fits on one line. In particular, you can never do so with
`try`/`except` since the `try` and `except` can't both fit on the same line, and
you can only do so with an `if` if there is no `else`.

**Correct example:**

```python
Yes:

  if foo: bar(foo)
```

**Incorrect example:**

```python
No:

  if foo: bar(foo)
  else:   baz(foo)

  try:               bar(foo)
  except ValueError: baz(foo)

  try:
      bar(foo)
  except ValueError: baz(foo)
```

### 3.15 Getters and Setters

Getter and setter functions (also called accessors and mutators) should be used
when they provide a meaningful role or behavior for getting or setting a
variable's value.

In particular, they should be used when getting or setting the variable is
complex or the cost is significant, either currently or in a reasonable future.

If, for example, a pair of getters/setters simply read and write an internal
attribute, the internal attribute should be made public instead. By comparison,
if setting a variable means some state is invalidated or rebuilt, it should be a
setter function. The function invocation hints that a potentially non-trivial
operation is occurring. Alternatively, [properties](#213-properties) may be an
option when simple logic is needed, or refactoring to no longer need getters and
setters.

Getters and setters should follow the [Naming](#316-naming) guidelines, such
as `get_foo()` and `set_foo()`.

If the past behavior allowed access through a property, do not bind the new
getter/setter functions to the property. Any code still attempting to access the
variable by the old method should break visibly so they are made aware of the
change in complexity.

### 3.16 Naming

`module_name`, `package_name`, `ClassName`, `method_name`, `ExceptionName`,
`function_name`, `GLOBAL_CONSTANT_NAME`, `global_var_name`, `instance_var_name`,
`function_parameter_name`, `local_var_name`, `query_proper_noun_for_thing`,
`send_acronym_via_https`.

Names should be descriptive. This includes functions, classes, variables,
attributes, files and any other type of named entities.

Avoid abbreviation. In particular, do not use abbreviations that are ambiguous
or unfamiliar to readers outside your project, and do not abbreviate by deleting
letters within a word.

Always use a `.py` filename extension. Never use dashes.

#### 3.16.1 Names to Avoid

-   single character names, except for specifically allowed cases:

    -   counters or iterators (e.g. `i`, `j`, `k`, `v`, et al.)
    -   `e` as an exception identifier in `try/except` statements.
    -   `f` as a file handle in `with` statements
    -   private [type variables](#31910-type-variables) with no constraints (e.g.
        `_T = TypeVar("_T")`, `_P = ParamSpec("_P")`)
    -   names that match established notation in a reference paper or algorithm
        (see [Mathematical Notation](#3165-mathematical-notation))

    Please be mindful not to abuse single-character naming. Generally speaking,
    descriptiveness should be proportional to the name's scope of visibility.
    For example, `i` might be a fine name for 5-line code block but within
    multiple nested scopes, it is likely too vague.

-   dashes (`-`) in any package/module name

-   `__double_leading_and_trailing_underscore__` names (reserved by Python)

-   offensive terms

-   names that needlessly include the type of the variable (for example:
    `id_to_name_dict`)

#### 3.16.2 Naming Conventions

-   In general, follow [PEP 8 naming conventions][PEP-8-naming-conventions].

-   "Internal" means internal to a module, or protected or private within a
    class.

-   Prepending a single underscore (`_`) has some support for protecting module
    variables and functions (linters will flag protected member access). Note
    that it is okay for unit tests to access protected constants from the
    modules under test.

-   Prepending a double underscore (`__` aka "dunder") to an instance variable
    or method effectively makes the variable or method private to its class
    (using name mangling); we discourage its use as it impacts readability and
    testability, and isn't *really* private. Prefer a single underscore.

-   Place related classes and top-level functions together in a module. Unlike
    Java, there is no need to limit yourself to one class per module.

-   Use CapWords for class names, but lower\_with\_under.py for module names.
    Although there are some old modules named CapWords.py, this is now
    discouraged because it's confusing when the module happens to be named after
    a class. ("wait -- did I write `import StringIO` or
    `from StringIO import StringIO`?")

    -   Note that [PEP 8 says][PEP-8-acronyms]: "When using acronyms in
        CapWords, capitalize all the letters of the acronym. Thus
        HTTPServerError is better than HttpServerError."

-   New *unit test* files follow PEP 8 compliant lower\_with\_under method
    names, for example, `test_<method_under_test>_<state>`. For consistency(\*)
    with legacy modules that follow CapWords function names, underscores may
    appear in method names starting with `test` to separate logical components
    of the name. One possible pattern is `test<MethodUnderTest>_<state>`.

[PEP-8-naming-conventions]: https://peps.python.org/pep-0008/#naming-conventions
[PEP-8-acronyms]: https://peps.python.org/pep-0008/#descriptive-naming-styles

##### 3.16.2.1 Exemptions

The following exemptions apply to the general naming conventions above:

-   When Python code is tightly coupled with an external or non-Python codebase,
    such as foreign function interfaces (FFIs) to C++, consistency with the
    other codebase may warrant following a different convention
    ([Consistency](#4-parting-words)).

#### 3.16.3 File Naming

Python filenames must have a `.py` extension and must not contain dashes (`-`).
This allows them to be imported and unittested. If you want an executable to be
accessible without the extension, use a symbolic link or a simple bash wrapper
containing `exec "$0.py" "$@"`.

#### 3.16.4 Guidelines derived from [Guido](https://en.wikipedia.org/wiki/Guido_van_Rossum)'s Recommendations

| Type | Public | Internal |
| --- | --- | --- |
| Packages | `lower_with_under` |  |
| Modules | `lower_with_under` | `_lower_with_under` |
| Classes | `CapWords` | `_CapWords` |
| Exceptions | `CapWords` |  |
| Functions | `lower_with_under()` | `_lower_with_under()` |
| Global/Class Constants | `CAPS_WITH_UNDER` | `_CAPS_WITH_UNDER` |
| Global/Class Variables | `lower_with_under` | `_lower_with_under` |
| Instance Variables | `lower_with_under` | `_lower_with_under` (protected) |
| Method Names | `lower_with_under()` | `_lower_with_under()` (protected) |
| Function/Method Parameters | `lower_with_under` |  |
| Local Variables | `lower_with_under` |  |

#### 3.16.5 Mathematical Notation

For mathematically-heavy code, short variable names that would otherwise violate
the style guide are preferred when they match established notation in a
reference paper or algorithm.

When using names based on established notation:

1.  Cite the source of all naming conventions, preferably with a hyperlink to
    academic resource itself, in a comment or docstring. If the source is not
    accessible, clearly document the naming conventions.
2.  Prefer PEP 8-compliant `descriptive_names` for public APIs, which are much
    more likely to be encountered out of context.
3.  Use a narrowly-scoped `pylint: disable=invalid-name` directive to silence
    warnings. For just a few variables, use the directive as an endline comment
    for each one; for more, apply the directive at the beginning of a block.

### 3.17 Main

In Python, `pydoc` as well as unit tests require modules to be importable. If a
file is meant to be used as an executable, its main functionality should be in a
`main()` function, and your code should always check `if __name__ == '__main__'`
before executing your main program, so that it is not executed when the module
is imported.

When using [absl](https://github.com/abseil/abseil-py), use `app.run`:

**Correct example:**

```python
from absl import app
...

def main(argv: Sequence[str]):
    # process non-flag arguments
    ...

if __name__ == '__main__':
    app.run(main)
```

Otherwise, use:

**Correct example:**

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

All code at the top level will be executed when the module is imported. Be
careful not to call functions, create objects, or perform other operations that
should not be executed when the file is being `pydoc`ed.

### 3.18 Function length

Prefer small, focused functions. There is no hard length limit; above about
40 lines, consider splitting without harming program structure. When existing
long functions are difficult to modify or debug, or portions are needed in
several contexts, consider extracting smaller, manageable functions.

### 3.19 Type Annotations

#### 3.19.1 General Rules

*   Familiarize yourself with
    [type hints](https://docs.python.org/3/library/typing.html).

*   Annotating `self` or `cls` is generally not necessary.
    [`Self`](https://docs.python.org/3/library/typing.html#typing.Self) can be
    used if it is necessary for proper type information, e.g.

    **Correct example:**

    ```python
    from typing import Self

    class BaseClass:
      @classmethod
      def create(cls) -> Self:
        ...

      def difference(self, other: Self) -> float:
        ...
    ```

*   Similarly, don't feel compelled to annotate the return value of `__init__`
    (where `None` is the only valid option).

*   If any other variable or a returned type should not be expressed, use `Any`.

*   You are not required to annotate all the functions in a module.

    -   At least annotate your public APIs.
    -   Use judgment to get to a good balance between safety and clarity on the
        one hand, and flexibility on the other.
    -   Annotate code that is prone to type-related errors (previous bugs or
        complexity).
    -   Annotate code that is hard to understand.
    -   Annotate code as it becomes stable from a types perspective. In many
        cases, you can annotate all the functions in mature code without losing
        too much flexibility.

#### 3.19.2 Line Breaking

Try to follow the existing [indentation](#34-indentation) rules.

After annotating, many function signatures will become "one parameter per line".
To ensure the return type is also given its own line, a comma can be placed
after the last parameter.

**Correct example:**

```python
def my_method(
    self,
    first_var: int,
    second_var: Foo,
    third_var: Bar | None,
) -> int:
  ...
```

Always prefer breaking between variables, and not, for example, between variable
names and type annotations. However, if everything fits on the same line, go for
it.

**Correct example:**

```python
def my_method(self, first_var: int) -> int:
  ...
```

If the combination of the function name, the last parameter, and the return type
is too long, indent by 4 in a new line. When using line breaks, prefer putting
each parameter and the return type on their own lines and aligning the closing
parenthesis with the `def`:

**Correct example:**

```python
Yes:
def my_method(
    self,
    other_arg: MyLongType | None,
) -> tuple[MyLongType1, MyLongType1]:
  ...
```

Optionally, the return type may be put on the same line as the last parameter:

**Correct example:**

```python
Okay:
def my_method(
    self,
    first_var: int,
    second_var: int) -> dict[OtherLongType, MyLongType]:
  ...
```

`pylint` allows you to move the closing parenthesis to a new line and align with
the opening one, but this is less readable.

**Incorrect example:**

```python
No:
def my_method(self,
              other_arg: MyLongType | None,
             ) -> dict[OtherLongType, MyLongType]:
  ...
```

As in the examples above, prefer not to break types. However, sometimes they are
too long to be on a single line (try to keep sub-types unbroken).

**Correct example:**

```python
def my_method(
    self,
    first_var: tuple[list[MyLongType1],
                     list[MyLongType2]],
    second_var: list[dict[
        MyLongType3, MyLongType4]],
) -> None:
  ...
```

If a single name and type is too long, consider using an
[alias](#3196-type-aliases) for the type. The last resort is to break after the
colon and indent by 4.

**Correct example:**

```python
Yes:
def my_function(
    long_variable_name:
        long_module_name.LongTypeName,
) -> None:
  ...
```

**Incorrect example:**

```python
No:
def my_function(
    long_variable_name: long_module_name.
        LongTypeName,
) -> None:
  ...
```

#### 3.19.3 Forward Declarations

If you need to use a class name (from the same module) that is not yet
defined -- for example, if you need the class name inside the declaration of
that class, or if you use a class that is defined later in the code -- either
use `from __future__ import annotations` or use a string for the class name.

**Correct example:**

```python
Yes:
from __future__ import annotations

class MyClass:
  def __init__(self, stack: Sequence[MyClass], item: OtherClass) -> None:

class OtherClass:
  ...
```

**Correct example:**

```python
Yes:
class MyClass:
  def __init__(self, stack: Sequence['MyClass'], item: 'OtherClass') -> None:

class OtherClass:
  ...
```

#### 3.19.4 Default Values

As per [PEP-008](https://peps.python.org/pep-0008/#other-recommendations), use
spaces around the `=` *only* for arguments that have both a type annotation and
a default value.

**Correct example:**

```python
Yes:
def func(a: int = 0) -> int:
  ...
```

**Incorrect example:**

```python
No:
def func(a:int=0) -> int:
  ...
```

#### 3.19.5 NoneType

In the Python type system, `NoneType` is a "first class" type, and for typing
purposes, `None` is an alias for `NoneType`. If an argument can be `None`, it
has to be declared! You can use `|` union type expressions (recommended in new
Python 3.10+ code), or the older `Optional` and `Union` syntaxes.

Use explicit `X | None` instead of implicit. Earlier versions of type checkers
allowed `a: str = None` to be interpreted as `a: str | None = None`, but that is
no longer the preferred behavior.

**Correct example:**

```python
Yes:
def modern_or_union(a: str | int | None, b: str | None = None) -> str:
  ...
def union_optional(a: Union[str, int, None], b: Optional[str] = None) -> str:
  ...
```

**Incorrect example:**

```python
No:
def nullable_union(a: Union[None, str]) -> str:
  ...
def implicit_optional(a: str = None) -> str:
  ...
```

#### 3.19.6 Type Aliases

You can declare aliases of complex types.

*   The name of an alias should be CapWorded. If the alias is used only in this
    module, it should be \_Private.
*   The alias should be type-annotated as `typing.TypeAlias`, to distinguish it
    from a non-type global variable.
*   Do not use type aliases as a substitute for the `from module import symbol`
    idiom, which is [not allowed](#22-imports).

**Correct example:**

```python
Yes:
from typing import TypeAlias

_LossAndGradient: TypeAlias = tuple[tf.Tensor, tf.Tensor]
ComplexTFMap: TypeAlias = Mapping[str, _LossAndGradient]
```

**Incorrect example:**

```python
No:
import immutabledict

immutabledict = immutabledict.immutabledict
```

```python
# Acceptable for convenience (provided the alias is not used in a public API,
# such as a public function's signature, of this module).
_ShortName: TypeAlias = other_module.SignificantlyInconvenientlyLongName
```

#### 3.19.7 Ignoring Types

You can disable type checking on a line with the special comment
`# type: ignore`.

`pyrefly` has a disable option for specific errors (similar to lint):

**Correct example:**

```python
# pyrefly: ignore[attribute-error]
```

#### 3.19.8 Typing Variables

- [*Annotated Assignments*](#3198-typing-variables)

    If an internal variable has a type that is hard or impossible to infer,
    specify its type with an annotated assignment - use a colon and type between
    the variable name and value (the same as is done with function arguments
    that have a default value):

    **Correct example:**

    ```python
    a: Foo = SomeUndecoratedFunction()
    ```

- [*Type Comments*](#3198-typing-variables)

    Though you may see them remaining in the codebase (they were necessary
    before Python 3.6), do not add any more uses of a `# type: <type name>`
    comment on the end of the line:

    **Incorrect example:**

    ```python
    a = SomeUndecoratedFunction()  # type: Foo
    ```

#### 3.19.9 Tuples vs Lists

Typed lists can only contain objects of a single type. Typed tuples can either
have a single repeated type or a set number of elements with different types.
The latter is commonly used as the return type from a function.

**Correct example:**

```python
a: list[int] = [1, 2, 3]
b: tuple[int, ...] = (1, 2, 3)
c: tuple[int, str, float] = (1, "2", 3.5)
```

#### 3.19.10 Type variables

The Python type system has
[generics](https://docs.python.org/3/library/typing.html#generics). A type
variable, such as `TypeVar` and `ParamSpec`, is a common way to use them.

Example:

**Correct example:**

```python
from collections.abc import Callable
from typing import ParamSpec, TypeVar
_P = ParamSpec("_P")
_T = TypeVar("_T")
...
def next(l: list[_T]) -> _T:
  return l.pop()

def print_when_called(f: Callable[_P, _T]) -> Callable[_P, _T]:
  def inner(*args: _P.args, **kwargs: _P.kwargs) -> _T:
    print("Function was called")
    return f(*args, **kwargs)
  return inner
```

A `TypeVar` can be constrained:

**Correct example:**

```python
AddableType = TypeVar("AddableType", int, float, str)
def add(a: AddableType, b: AddableType) -> AddableType:
  return a + b
```

A common predefined type variable in the `typing` module is `AnyStr`. Use it for
multiple annotations that can be `bytes` or `str` and must all be the same type.

**Correct example:**

```python
from typing import AnyStr
def check_length(x: AnyStr) -> AnyStr:
  if len(x) <= 42:
    return x
  raise ValueError()
```

A type variable must have a descriptive name, unless it meets all of the
following criteria:

*   not externally visible
*   not constrained

**Correct example:**

```python
Yes:
  _T = TypeVar("_T")
  _P = ParamSpec("_P")
  AddableType = TypeVar("AddableType", int, float, str)
  AnyFunction = TypeVar("AnyFunction", bound=Callable)
```

**Incorrect example:**

```python
No:
  T = TypeVar("T")
  P = ParamSpec("P")
  _T = TypeVar("_T", int, float, str)
  _F = TypeVar("_F", bound=Callable)
```

#### 3.19.11 String types

> Do not use `typing.Text` in new code. It's only for Python 2/3 compatibility.

Use `str` for string/text data. For code that deals with binary data, use
`bytes`.

**Correct example:**

```python
def deals_with_text_data(x: str) -> str:
  ...
def deals_with_binary_data(x: bytes) -> bytes:
  ...
```

If all the string types of a function are always the same, for example if the
return type is the same as the argument type in the code above, use
[AnyStr](#31910-type-variables).

#### 3.19.12 Imports For Typing

For symbols (including types, functions, and constants) from the `typing` or
`collections.abc` modules used to support static analysis and type checking,
always import the symbol itself. This keeps common annotations more concise and
matches typing practices used around the world. You are explicitly allowed to
import multiple specific symbols on one line from the `typing` and
`collections.abc` modules. For example:

**Correct example:**

```python
from collections.abc import Mapping, Sequence
from typing import Any, Generic, cast, TYPE_CHECKING
```

Given that this way of importing adds items to the local namespace, names in
`typing` or `collections.abc` should be treated similarly to keywords, and not
be defined in your Python code, typed or not. If there is a collision between a
type and an existing name in a module, import it using `import x as y`.

**Correct example:**

```python
from typing import Any as AnyType
```

When annotating function signatures, prefer abstract container types like
`collections.abc.Sequence` over concrete types like `list`. If you need to use a
concrete type (for example, a `tuple` of typed elements), prefer built-in types
like `tuple` over the parametric type aliases from the `typing` module (e.g.,
`typing.Tuple`).

**Incorrect example:**

```python
from typing import List, Tuple

def transform_coordinates(original: List[Tuple[float, float]]) ->
    List[Tuple[float, float]]:
  ...
```

**Correct example:**

```python
from collections.abc import Sequence

def transform_coordinates(original: Sequence[tuple[float, float]]) ->
    Sequence[tuple[float, float]]:
  ...
```

#### 3.19.13 Conditional Imports

Use conditional imports only in exceptional cases where the additional imports
needed for type checking must be avoided at runtime. This pattern is
discouraged; alternatives such as refactoring the code to allow top-level
imports should be preferred.

Imports that are needed only for type annotations can be placed within an
`if TYPE_CHECKING:` block.

-   Conditionally imported types need to be referenced as strings, to be forward
    compatible with Python 3.6 where the annotation expressions are actually
    evaluated.
-   Only entities that are used solely for typing should be defined here; this
    includes aliases. Otherwise it will be a runtime error, as the module will
    not be imported at runtime.
-   The block should be right after all the normal imports.
-   There should be no empty lines in the typing imports list.
-   Sort this list as if it were a regular imports list.

**Correct example:**

```python
import typing
if typing.TYPE_CHECKING:
  import sketch
def f(x: "sketch.Sketch"): ...
```

#### 3.19.14 Circular Dependencies

Circular dependencies that are caused by typing are code smells. Such code is a
good candidate for refactoring. Although technically it is possible to keep
circular dependencies, various build systems will not let you do so because each
module has to depend on the other.

Replace modules that create circular dependency imports with `Any`. Set an
[alias](#3196-type-aliases) with a meaningful name, and use the real type name from
this module (any attribute of `Any` is `Any`). Alias definitions should be
separated from the last import by one line.

**Correct example:**

```python
from typing import Any

some_mod = Any  # some_mod.py imports this module.
...

def my_method(self, var: "some_mod.SomeType") -> None:
  ...
```

#### 3.19.15 Generics

When annotating, prefer to specify type parameters for
[generic](https://docs.python.org/3/library/typing.html#generics) types in a
parameter list; otherwise, the generics' parameters will be assumed to be
[`Any`](https://docs.python.org/3/library/typing.html#the-any-type).

**Correct example:**

```python
# Yes:
def get_names(employee_ids: Sequence[int]) -> Mapping[int, str]:
  ...
```

**Incorrect example:**

```python
# No:
# This is interpreted as get_names(employee_ids: Sequence[Any]) -> Mapping[Any, Any]
def get_names(employee_ids: Sequence) -> Mapping:
  ...
```

If the best type parameter for a generic is `Any`, make it explicit, but
remember that in many cases [`TypeVar`](#31910-type-variables) might be more
appropriate:

**Incorrect example:**

```python
# No:
def get_names(employee_ids: Sequence[Any]) -> Mapping[Any, str]:
  """Returns a mapping from employee ID to employee name for given IDs."""
```

**Correct example:**

```python
# Yes:
_T = TypeVar('_T')
def get_names(employee_ids: Sequence[_T]) -> Mapping[_T, str]:
  """Returns a mapping from employee ID to employee name for given IDs."""
```

## 4 Parting Words

Be consistent with surrounding code, especially for local choices the global
guide leaves unspecified. Examine existing style before editing. Consistency
does not generally justify retaining an old style without considering newer
styles' benefits or the codebase's gradual convergence on them.

## Sources and licensing

- `pyguide.md`: [fixed upstream source](https://github.com/google/styleguide/blob/572c673d9058242c049fbc8248b17029726c7afa/pyguide.md); [official published guide](https://google.github.io/styleguide/pyguide.html).

Upstream: Google Style Guides, by Google and the contributors or authors named in the source documents, at commit `572c673d9058242c049fbc8248b17029726c7afa`. The upstream repository README states [CC BY 3.0](https://creativecommons.org/licenses/by/3.0/) for documentation and [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0) for code samples. Preserve any more specific attribution or license notices in the original documents.

Local changes: this is an **editorially condensed adaptation**, not a verbatim
copy. Site templates and presentational HTML were removed; rules, exceptions
and representative examples were consolidated into one Markdown language file.
Repeated rationale and examples were shortened; some examples were edited.
The fixed source is available above for optional comparison. This adaptation is
unofficial; Google does not endorse this skill.
