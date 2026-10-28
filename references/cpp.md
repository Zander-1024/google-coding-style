# Google C++ Style Guide — Condensed Reference

This is an **editorially condensed adaptation** of Google's C++ Style Guide at commit `572c673d9058242c049fbc8248b17029726c7afa`. It preserves the guide's actionable rules, recommendation strengths, key exceptions, and section coverage while shortening rationale and examples. It is not a verbatim copy. Attribution, source, and licenses appear below.

## Background

This guide manages C++ complexity for readable, maintainable code; it assumes familiarity with C++. Examples illustrate individual rules and may omit comments or details required in submitted code; do not copy them as production templates.

### Goals of the Style Guide

Rules should justify their cost. Optimize for readers; make surprising behavior, such as ownership transfer, visible. Be consistent locally and across the codebase. Local consistency should not generally justify perpetuating old practices without considering newer ones. Follow the broader C++ community where appropriate, but constrain dangerous or unsuitable features. Avoid surprising constructs and complexity an average maintainer cannot manage. Consider costs at large codebase scale. Necessary performance optimizations may justify tradeoffs.

Common sense and good taste, grounded in the Google C++ community's conventions, should prevail. Absence of a ban does not endorse a clever or unusual construct. Ask project leads when uncertain; complex-feature waivers require justification, and restrictions preventing correctness hazards have a particularly high bar.

## C++ Version

Code should target **C++20** and should not use C++23 features. Do not use nonstandard extensions except as allowed below. Consider portability before using C++17/C++20 features. These rules describe the pinned source version, not a claim about future upstream policy.

## Header Files

In general each `.cc` file should have an associated `.h`; common exceptions include unit tests and small files containing only `main()`.

### Self-contained Headers

Headers should compile on their own, end in `.h`, have header guards, and directly include all prerequisites. Sparingly used files intended for unusual inclusion positions and unable to be self-contained should end in `.inc`.

Definitions of inline functions and templates that clients instantiate must be in the header or files it includes. Separately included `-inl.h` files are no longer allowed. A template definition may stay in one `.cc` when all instantiations occur there, explicitly or because only that file can access the definition.

### The #define Guard

All headers should use guards named `<PROJECT>_<PATH>_<FILE>_H_`, based on the full source-tree path for uniqueness. For project `foo`, file `foo/src/bar/baz.h`:

```cpp
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_
// ...
#endif  // FOO_BAR_BAZ_H_
```

### Include What You Use

When a source/header refers to a symbol defined elsewhere, it should directly include a header intended to provide that symbol's declaration/definition, and should not include headers for other reasons. Do not rely on transitive includes, including through the related header: `foo.cc` using a `bar.h` symbol should include `bar.h` even if `foo.h` includes it.

### Forward Declarations

Avoid forward declarations where possible; include the needed headers. Do not forward-declare entities your project does not own, including anything in `std`. For owned entities, first make the declaration's header cheap to include and try speeding up expensive compilation. A forward declaration is allowed if engineering judgment identifies it as the better way to fix a slowdown. Consider hidden dependencies, API evolution, tooling, and changes in overload resolution when replacing an include.

### Defining Functions in Header Files

Define a function at its public declaration only if short, roughly 10 lines or fewer. Longer bodies belong in `.cc` unless technical or performance reasons require the header. Even then, put long definitions in an internal part: a private class section, namespace containing `internal`, or below `// Implementation details only below here`.

Header definitions must be ODR-safe: use `inline` when necessary, or implicit inline status such as a function template or a function defined in a class body at its first declaration. These distinctions and rules also apply to variables where applicable.

### Names and Order of Includes

Header paths should be relative to the project's source directory, without `.` or `..`: `#include "base/logging.h"`. Angle brackets should be used only when the library requires them: standard C/C++ headers, POSIX/Linux/Windows system headers, and rare third-party headers such as `<Python.h>`.

For a source/test primarily implementing/testing a header, order nonempty groups below, separated by one blank line; includes within each group should be alphabetized:

1. Related header, even if in another directory.
2. C system and other angle-bracket headers ending in `.h`.
3. C++ standard headers without an extension.
4. Other libraries' headers.
5. Your project's headers.

```cpp
#include "foo/server/fooserver.h"

#include <sys/types.h>
#include <unistd.h>

#include <string>
#include <vector>

#include "base/basictypes.h"
#include "foo/server/bar.h"
#include "third_party/absl/flags/flag.h"
```

C headers and their C++ counterparts (`stddef.h`/`cstddef`) are both acceptable; follow existing practice. Old nonalphabetical ordering should be fixed when convenient. Conditional system-specific includes may follow other includes; keep such code small and localized.

## Scoping

### Namespaces

With few exceptions, place code in namespaces. Their names should be unique and based on the project, possibly including its path. Wrap the file after includes, flags definitions/declarations, and forward declarations from other namespaces. Do not indent namespace contents; close multiline namespaces with `// namespace name`. Prefer single-line nested declarations in new code (`namespace project::component {`), but they are not required. Use a `.proto` file's `package` specifier to namespace generated messages.

Do not declare anything in `std`; include the appropriate header. Do not use using-directives (`using namespace foo;`) or inline namespaces. At namespace scope in headers, namespace aliases are allowed only inside explicitly marked internal-only namespaces; local function aliases and aliases in `.cc` are allowed with appropriate names. Never put namespace aliases in a public API.

Use namespaces containing `internal` for API details users should not mention. Do not reference other libraries' internal names. Give each library a unique internal namespace including its filename, e.g. `gshoe::internal_widget`, rather than a collision-prone shared `gshoe::internal`.

```cpp
namespace my_project::my_component {
class MyClass {};
}  // namespace my_project::my_component
```

### Internal Linkage

Give `.cc` definitions not needed outside that file internal linkage via an unnamed namespace, or `static` for functions/variables. This is encouraged for all such code. Do not use these internal-linkage constructs in headers. Format unnamed namespaces normally, closing with `}  // namespace`.

### Nonmember, Static Member, and Global Functions

Prefer nonmember functions in namespaces; use completely global functions rarely. Nonmember functions should not depend on external variables and should nearly always be namespaced. File-only functions should have internal linkage. Do not create classes only to group static members without a good reason, such as policy/traits, type semantics for a representation, or an interface closed to extension. Consider whether functions with significant dependencies or external resources instead belong in a class.

### Local Variables

Declare variables in the narrowest practical scope, close to first use, and initialize at declaration rather than assign afterward. Variables needed by `if`, `while`, and `for` should normally be declared within those statements. Prefer `std::vector<int> v = {1, 2};` over default construction followed by pushes. Move a loop-local object outside the loop when repeated construction/destruction would be inefficient.

### Static and Global Variables

Objects with static storage duration (globals, static members, function-local statics) are **forbidden unless trivially destructible**, including their members and bases. Fundamental types, pointers, and arrays of trivial types qualify. A reference is not an object, but a lifetime-extended temporary still must meet the destruction rule.

```cpp
constexpr std::array<int, 3> kArray = {1, 2, 3};  // Allowed.
const std::string kFoo = "foo";  // Bad: nontrivial destruction.
const std::string& kBar = StrCat("a", "b");  // Bad: temporary.
```

Constant initialization is always allowed and should be enforced with `constexpr` or `constinit`. A nonlocal static not so marked should be presumed dynamically initialized and reviewed carefully. Nonlocal dynamic initialization is discouraged and generally forbidden; it is allowed only when no program behavior depends on its sequencing relative to **all** other initializations. For example `int p = getpid();` is allowed if no other static initializer uses `p`. Function-local static dynamic initialization is allowed. References still obey initialization restrictions; `static T& t = *new T;` is allowed locally.

For global strings, consider `constexpr` string views, character arrays, or character pointers to literals. For fixed maps/sets, consider arrays of trivial entries and linear search (or sorted arrays and binary search). Static smart pointers and ordinary dynamic standard containers are forbidden because their destructors perform cleanup. Custom static-data types should have trivial destructors and `constexpr` constructors. When necessary, use a function-local static pointer/reference to a dynamically allocated object that is never deleted, e.g. `static const auto& impl = *new T(args...);`.

### thread_local Variables

Prefer `thread_local` to other thread-local mechanisms. At class or namespace scope it must have true constant initialization, enforced by **`constinit`** (or, rarely, `constexpr`). Function-scope `thread_local` may initialize dynamically; a function/static method may expose it to simulate wider scope.

Thread-local destruction can cause use-after-free at thread exit: a destructor must not transitively refer to a potentially destroyed thread-local. Prefer trivial types or types provably running no user code during destruction. Do not apply the global-object leak workaround casually: leaking per-thread objects accumulates resources as threads exit.

## Classes

### Doing Work in Constructors

Constructors should never call virtual functions. Avoid initialization that can fail without a way to report errors. Termination may be appropriate; otherwise consider a factory or `Init()` method. Avoid `Init()` on objects whose only state distinction is being semi-constructed, because callers can forget the required step.

### Implicit Conversions

Do not define implicit conversions. In class definitions, conversion operators and constructors callable with one argument **must** be `explicit`, including constructors whose other parameters have defaults. Copy/move constructors should not be `explicit`. A constructor taking one `std::initializer_list` should also omit `explicit` to support `MyType m = {1, 2};`. Other constructors may omit it. Interchangeable representations sometimes justify implicit conversions; request a waiver from project leads.

### Copyable and Movable Types

The public API **must make clear** whether a class is copyable, move-only, or neither. Classes should usually explicitly declare/delete the appropriate public operations: copyable types should declare copy operations, move-only types should declare move operations, and types that are neither copyable nor movable should delete copy operations. Copyable types may additionally declare efficient moves. Declaring/deleting all four operations is allowed, not required. Providing a copy/move assignment operator **requires the corresponding constructor**.

```cpp
class MoveOnly {
 public:
  MoveOnly(MoveOnly&&) = default;
  MoveOnly& operator=(MoveOnly&&) = default;
  // Copies are implicitly deleted; explicit deletion is also allowed.
};
class Neither {
 public:
  Neither(const Neither&) = delete;
  Neither& operator=(const Neither&) = delete;
  // Moves are implicitly disabled.
};
```

Omit these declarations only when behavior is obvious from public members of a type with no private section, or from a clearly noncopyable/nonmovable base. An interface-only base leaving operations implicit does not establish concrete subclasses' behavior. Explicitly declaring/deleting one copy operation makes its partner nonobvious and requires declaring/deleting it too; likewise for moves.

Types should not support copying/moving when the meaning is unclear or the cost unexpected. For copyable types, define moves only when significantly more efficient than copies. Prefer designing copies so defaults are correct, and review defaulted operations as carefully as other code. To prevent slicing, prefer abstract base classes (protected constructors/destructors or pure virtual members), and avoid deriving from concrete classes.

### Structs vs. Classes

Use `struct` only for passive data carriers; use `class` otherwise, including when unsure. All struct fields **must be public**; the type must not impose cross-field invariants. It may have constants, constructors, destructors, and helpers, but they must not enforce or require invariants. Particular uses may have their own requirements. Prefer a class for more functionality, invariants, or wide visibility with expected evolution. For STL consistency, stateless traits, metafunctions, and some functors may use `struct`. Class and struct members have different naming rules.

### Structs vs. Pairs and Tuples

Prefer structs with meaningful field names over pairs/tuples. Pairs/tuples may fit generic code without specific element meanings or be required by existing APIs.

### Inheritance

All inheritance should be `public`; use composition in place of private inheritance. Prefer composition generally, and restrict inheritance to genuine “is-a” relationships. `final` is allowed on classes not intended as bases. Limit `protected` to functions subclasses may need; data should be private. Annotate overrides of virtual functions/destructors with **exactly one** of `override` or `final`, and do not repeat `virtual`. Multiple inheritance is allowed; multiple implementation inheritance is **strongly discouraged** (see Windows exception).

### Operator Overloading

Overload only with obvious, unsurprising meanings consistent with built-in operators. Define operators only on owned types, in the same headers, `.cc` files, and namespaces as those types. Avoid operator templates when possible: all possible argument types must satisfy this rule. Define sensible related operators consistently.

Prefer nonmember, nonmodifying binary operators. For equality-comparable `T`, define nonmember `operator==` and document equality. If a single obvious ordering exists, `operator<=>` may be defined consistently with equality; prefer not to overload the other comparison/ordering operators. Use natural `==`, `=`, `<<` rather than needless named substitutes, but do not invent unnatural ordering just to satisfy a container; supply a comparator.

Do not overload `&&`, `||`, comma, unary `&`, or `operator""`. Do not introduce or use user-defined literals, including standard-library ones. Conversion, assignment, streams, and general overload rules also apply.

### Access Control

Make class data members private, except constants. GoogleTest fixtures defined in their use-site `.cc` may have protected data for technical reasons; fixtures in headers/other files must keep data private.

### Declaration Order

Sections should usually be ordered `public`, `protected`, `private`. Omit empty sections. Group similar declarations; prefer this order within each section:

1. Types/aliases, enums, nested types, friend types.
2. Optionally, nonstatic data for structs only.
3. Static constants.
4. Factories.
5. Constructors and assignment operators.
6. Destructor.
7. Other functions, including static/nonstatic/friend functions.
8. Remaining static/nonstatic data.

Do not put large method bodies inline in a class; usually only trivial or performance-critical, very short methods qualify. Follow header-definition rules.

## Functions

### Inputs and Outputs

Prefer return values to output parameters. Prefer returning by value, then by reference; avoid raw-pointer returns unless nullable. Nonoptional inputs should usually be values or const references; nonoptional output/in-out parameters should usually be references. For optional by-value inputs use `std::optional`; for optional reference-like inputs use const pointers, and optional output/in-out parameters use nonconst pointers.

Avoid requiring a reference parameter to outlive the call. Eliminate the requirement (e.g. copy) or use a pointer and document lifetime/non-null requirements. Normally put input-only parameters before outputs, including newly added inputs. In-out arguments, related API consistency, and variadic functions may justify other ordering.

### Write Short Functions

Prefer small, focused functions. There is no hard length limit; beyond about 40 lines, consider splitting without harming structure. Refactor difficult existing functions when debugging, modifying, or reusing parts warrants it.

### Function Overloading

Overload only when a reader can understand a call without determining its exact overload. Variants may differ in types, qualifiers, or argument count, but must have no semantic differences requiring that distinction. Consider `string_view` instead of separate string/character-pointer overloads. Prefer one comprehensive umbrella comment before the first overload; add specific comments where connecting the common documentation would be difficult.

### Default Arguments

Allowed on nonvirtual functions only when the default is guaranteed always to evaluate to the same value. They obey overload restrictions. Banned on virtual functions and for varying defaults such as `counter++`. Prefer overloads when default-argument readability does not outweigh their costs; when uncertain, use overloads.

### Trailing Return Type Syntax

Usually place the return type before the function name. Use trailing return types only when required (e.g. explicit lambda return types), or leading syntax would be impractical or much less readable, a rare case mostly involving complex templates.

## Google-Specific Magic

### cpplint

Use `cpplint.py` to detect style errors, following project instructions when provided. It has false positives/negatives. If no project tool provides it, [cpplint.py](https://raw.githubusercontent.com/cpplint/cpplint/HEAD/cpplint.py) can be obtained separately.

## Other C++ Features

### Ownership and Smart Pointers

Prefer single, fixed owners for dynamically allocated objects. If allocation is necessary, prefer keeping ownership with the allocator; give other code copies or borrowed pointers/references where appropriate. Prefer `std::unique_ptr` for explicit ownership transfer:

```cpp
std::unique_ptr<Foo> FooFactory();
void FooConsumer(std::unique_ptr<Foo> ptr);
```

Do not design shared ownership without a very good reason. To avoid expensive copies, shared ownership should be used only when the performance benefit is significant and the object immutable (`std::shared_ptr<const Foo>`). When using shared ownership, prefer `std::shared_ptr`. Never use `std::auto_ptr`; use `std::unique_ptr`.

### Rvalue References

Do not use rvalue references or `&&`-qualified methods except for:

- Move constructors and move assignment.
- `&&`-qualified methods that logically consume `*this`, leaving it empty/unusable. For consumed ordinary parameters, prefer passing by value.
- Forwarding references with `std::forward` for perfect forwarding.
- Paired overloads taking `Foo&&` and `const Foo&` when they improve performance; usually prefer passing by value. Require evidence for added complexity justified by performance.

### Friends

Friend classes/functions are allowed within reason. Friends should usually be defined in the same file so private-member users are easy to find; builders and tests can be appropriate friends. Most classes should interact solely through public members.

### Exceptions

Do not use C++ exceptions, including handling-related facilities such as `std::exception_ptr` and `std::nested_exception`. Existing code is not exception-tolerant; use appropriate alternatives such as error codes/assertions. The Windows compatibility exception below does not permit writing exception-handling code.

### noexcept

Specify `noexcept` when useful and correct: an escaping exception must represent a fatal error. Move constructors may be assumed to benefit meaningfully; discuss claimed significant benefits for other functions with project leads.

Prefer unconditional `noexcept` when exceptions are entirely disabled. Otherwise use simple conditional specifications that evaluate false only for actual potential throws, e.g. nothrow-operation traits or allocator traits. You should prioritize a simple interface over covering every possible throwing scenario: document unsupported throwing operations and use unconditional `noexcept` when appropriate. Treat allocation failure as fatal where suitable. Removing a promised `noexcept` later may break callers.

### Run-Time Type Information (RTTI)

Avoid RTTI (`typeid`, `dynamic_cast`) when possible outside tests; think twice in new code. It may be used freely in unit tests. Prefer virtual methods for subtype-dependent behavior, or double dispatch/Visitor when processing belongs outside the object. When program logic guarantees the dynamic subtype, `dynamic_cast` may be used freely; `absl::down_cast` is usually better. Type-based decision trees indicate design trouble. Do not hand-implement RTTI substitutes such as type tags.

### Casting

Do not use C casts such as `(int)x`, except casts to `void`. Use `T(x)` only for class types. For explicit conversion, use the first **correctly applicable** choice below:

1. Brace initialization for arithmetic conversion (`int64_t{x}`), rejecting narrowing.
2. Function-style cast when converting to a class (`std::string(cord)`).
3. `absl::implicit_cast` for safe upcasts or adding const where explicit syntax is needed.
4. `static_cast` for value conversion, explicit pointer upcasts, or a downcast known to match the actual subtype.
5. `absl::down_cast` for a superclass-to-subclass pointer, only when the actual subtype is certain.
6. `const_cast` to remove const qualification, subject to const rules.
7. `reinterpret_cast` for unsafe pointer/integer/pointer conversions (including `void*`), only with understanding of aliasing. Consider dereferencing and `std::bit_cast` instead.
8. `std::bit_cast` to reinterpret a value's bits as a same-size type.

Apply RTTI rules to `dynamic_cast`.

### Streams

Use streams only when best suited, usually for ad hoc, local, developer-facing human-readable I/O. Follow established surrounding/codebase tools: logging libraries generally beat `std::cerr`/`std::clog`, and `absl/strings` or equivalents generally beat stringstreams. Avoid streams for external-user I/O or untrusted data; use appropriate templating libraries for localization and security.

Avoid stateful stream facilities other than error state (`imbue`, `xalloc`, `register_callback`, formatting flags/manipulators). Use explicit formatting such as `absl::StreamFormat`. Overload streaming `<<` only for value types and emit their human-readable user-visible value, not internals. Use named debugging methods such as `DebugString()` for internals.

### Preincrement and Predecrement

Use prefix `++i`/`--i` unless the expression needs the old value provided by postfix semantics.

### Use of const

Strongly prefer meaningful, accurate const in APIs: parameters, methods, and nonlocal variables. A promised-unmodified pointer/reference argument should be `const T*`/`const T&`. Top-level const on by-value parameters has no caller effect and is not recommended in declarations. Declare methods const unless they alter logical state, enable such alteration, or cannot be invoked safely concurrently. Local-variable const is neither encouraged nor discouraged.

All const operations of a class should be safe concurrently with each other; if infeasible, explicitly document the class as thread-unsafe. Prefer `const int*` to `int const*`, without requiring it; be locally consistent.

### Use of constexpr, constinit, and consteval

Use `constexpr` for true constants and functions supporting their definitions, and to ensure constant initialization. Use `constinit` to enforce constant initialization of nonconstant variables. `consteval` may be used for compile-time-only code. Do not complicate function definitions to make them constexpr, or use constexpr/consteval to force inlining.

### Integer Types

Of built-in integer types, only `int` should be used; for a guaranteed size use exact-width types from `<stdint.h>`, e.g. `int16_t`, `int64_t`. Standard aliases such as `size_t`/`ptrdiff_t` are welcome where appropriate. Prefer omitting `std::` for standard integer types. You should assume int has at least 32 bits, but do not assume it has more. Use int for small values such as ordinary loop counters. If values could reach `2^31`, or intermediate calculations might require it, use a 64-bit type. Use `int64_t` for potentially big integers; when uncertain choose larger types. Containers' size-return types must accommodate all possible uses.

Unsigned types should not be used except for a valid reason such as bit patterns or required modulo-`2^N` arithmetic. Do not use unsigned merely to claim nonnegativity; use assertions. Avoid mixing signedness, prefer iterators/containers over pointer-size arithmetic, and handle conversions/promotions carefully to avoid undefined behavior and security bugs.

### Floating-Point Types

Use only `float` and `double`; they may be assumed IEEE-754 binary32/binary64. Do not use nonportable `long double`.

### Architecture Portability

Write architecture-portable code; do not rely on features specific to one processor. Prefer type-safe numeric formatting (`absl::StrCat`, `Substitute`, `StrFormat`, streams) to printf-family functions. Serialize structured data with a serialization library, such as Protocol Buffers, rather than copying its memory representation. Store integer representations of addresses in `uintptr_t`, not arbitrary 32/64-bit types. Use portable integer/floating types. Construct 64-bit constants with braces when needed:

```cpp
int64_t my_value{0x123456789};
uint64_t my_mask{uint64_t{3} << 48};
```

### Preprocessor Macros

Avoid macros, especially in headers; prefer inline functions, enums, const variables, or references. Avoid conditional compilation except necessary header guards; it complicates testing. Before using unavoidable macro features, consider a nonmacro solution. Do not use macros to declare pieces of a C++ API; request a project-lead waiver if needed.

When using macros, follow these practices whenever possible: avoid defining them in `.h`; define just before use and undefine just afterward; do not undefine someone else's macro to replace it—choose a unique name; avoid unbalanced C++ expansions or document them clearly; prefer not using `##` to generate names.

Exporting macros from headers is **extremely strongly discouraged**. Any exported macro **must** have a globally unique name prefixed with the uppercase project namespace. Macro names should use uppercase and underscores with a project prefix.

### 0 and nullptr/NULL

Use `nullptr` for pointers and `'\0'` for the null character, rather than integer `0` or `NULL`.

### sizeof

Prefer `sizeof(variable)` when measuring a particular variable, so type changes remain correct. `sizeof(type)` is allowed for code unrelated to a particular variable, e.g. data-format handling.

```cpp
memset(&data, 0, sizeof(data));      // Good.
memset(&data, 0, sizeof(MyStruct));  // Bad when measuring data.
```

### Type Deduction (including auto)

Use deduction only to make code clearer to readers unfamiliar with the project, or safer; not merely to avoid writing types. `make_unique<Foo>()` has an obvious return type; `MyWidgetFactory()` may not. Apply this to all deduction forms:

- **Function template arguments:** almost always OK, as templates usually expect deduction.
- **Local variables:** omit obvious/irrelevant type information, but preserve useful information with explicit local types as needed. Iterator types often contain boilerplate, while the value type remains informative. Use `decltype(auto)` only if a simpler choice will not work. CTAD has its own restriction below.
- **Return types:** use deduction for functions/lambdas only with a very small number of returns, very little other code, and very narrow scope. Public header functions should almost never deduce return types; implementation would become interface.
- **Parameters:** avoid deduction if a specific type suffices. One-type functions should almost always spell the type unless defined/called very close together, or a lambda goes to a sufficiently familiar interface such as `std::sort`. Do not use `auto` parameters in non-lambda functions; use named template parameters. In a lambda, prefer an explicit template parameter if its type is referenced later.
- **Lambda init captures:** follow the more specific capture rules below.
- **Structured bindings:** meaningful names can add readability, especially for pairs/tuples supplied by existing APIs and map key/value entries, where explicitly missing a const key can accidentally copy. Generally prefer a named struct over introducing pairs/tuples. For renamed struct fields, recommend comments identifying original fields, e.g. `auto [/*field_name=*/bound_name, ...] = value;`.

```cpp
auto widget = std::make_unique<Widget>(arg);  // Clear.
auto foo = x.add_foo();                      // Bad if type is unclear.
template <typename T> void F(T arg);         // Good.
// void F(auto arg);                         // Forbidden.
auto lambda = []<typename T>(T x) { T y; /*...*/ };
```

### Class Template Argument Deduction

Use CTAD only if maintainers explicitly opted in by providing at least one explicit deduction guide; all `std` templates are presumed opted in. This should be enforced with a compiler warning when available. General deduction readability/safety rules still apply.

### Designated Initializers

Use only the C++20-compliant form, with initializers in declaration order. Fields may be omitted, but may not be reordered: for `{x, y, z}`, `.x` then `.z` is valid; `.y` then `.x` is not.

### Lambda Expressions

Use lambdas where appropriate; avoid excessive length/nesting. Prefer explicit captures when the lambda may escape its scope. Capturing a pointer by value does not copy its referent or solve its lifetime problem.

- Use `[&]` only when the lambda's lifetime is obviously shorter than all potential captures.
- Use `[=]` only to bind a few obvious variables in a short lambda, without implicitly capturing `this`. A member-function lambda accessing nonstatic members must capture `this` explicitly or use `[&]`. Prefer avoiding long/complex default-by-value lambdas.
- Captures must actually capture enclosing variables. Do not use init captures to introduce unrelated new names or substantially change an existing name's meaning. Declare a normal variable first and capture it, or define a function object. Moving an existing variable into its capture is allowed.
- Follow deduction rules for parameter/return types and formatting rules below.

```cpp
executor->Schedule([&] { Frobnicate(foo); });     // Bad: hidden lifetimes.
executor->Schedule([&foo] { Frobnicate(foo); });  // Clearer, still check lifetime.
[foo = std::move(foo)] { /*...*/ };              // Captures existing foo.
```

### Template Metaprogramming

Avoid complicated templates. Consider whether ordinary maintainers can understand the code and users can understand errors; recursive instantiation, typelists, metafunctions, expression templates, SFINAE, and sizeof-overload tricks often go too far. Such techniques are best confined to a few widely used low-level components.

If metaprogramming is needed, you should minimize and isolate its complexity, hide it as an implementation detail where possible, keep public headers readable, comment tricky code especially well, and document use and the resulting generated code. Compiler diagnostics should be made understandable and actionable. Treat diagnostics as part of the interface.

### Concepts and Constraints

Use concepts sparingly, generally only where templates would have been used before C++20. Prefer equivalent standard concepts over traits (`std::integral` rather than `std::is_integral_v`) and modern `requires(Condition)` over legacy SFINAE such as enable_if. Avoid `template<Concept T>`; use `template <typename T> requires(Concept<T>)`. Do not reimplement existing concepts/traits.

New concepts should be rare and kept internal within the library. Avoid defining them in headers unless those headers are marked internal; they should not be exposed at API boundaries. Do not duplicate function bodies in constraints or impose insignificant/obvious requirements. Prefer an ordinary template unless a significant benefit, e.g. clearer errors for deep requirements, can be demonstrated. Concepts should be statically verifiable: do not define concepts that are not compiler-enforced, or use concepts whose main benefit is unenforced semantic requirements; those contracts should instead use comments, assertions, or tests.

### C++20 modules

Do not use C++20 modules.

### Coroutines

Use C++20 coroutines only through libraries approved by project leads for project-wide use. Do not implement your own promise or awaitable types.

### Disallowed standard library features

Do not use compile-time rational numbers (`<ratio>`), floating-point environment headers (`<cfenv>`, `<fenv.h>`), or `<filesystem>`.

### Third-party Libraries

Use the library already customary for a problem in the surrounding codebase. Otherwise prefer, in roughly this order: Abseil; permitted standard-library facilities; existing first-party libraries; third-party libraries. Follow codebase third-party policies when selecting external dependencies.

### Nonstandard Extensions

Do not use nonstandard extensions unless specifically allowed. Examples include compiler attributes/intrinsics, SIMD, pragmas, inline assembly, `__COUNTER__`, `__PRETTY_FUNCTION__`, statement expressions, variable-length arrays, `alloca()`, and the Elvis `a?:b` extension. Portability wrappers using extensions are allowed only when supplied by a designated project-wide portability header. See Windows-specific allowances below.

### Aliases

Prefer `using` to `typedef` in new C++. A header alias is public API unless inside a function, private class section, or explicitly internal namespace; `.cc` aliases are implementation details. Public aliases must benefit clients, not just shorten implementation code. Document their intended use and compatibility promise: always identical to the underlying type, same API, or only limited opaque use. Do not place namespace aliases in public APIs. Local convenience aliases in implementation areas are fine. Apply naming rules in the new alias's context.

```cpp
// Stores measurements; clients should treat this as an opaque pointer.
// The underlying type may change.
using DataPoint = ::foo::Bar*;
```

### Switch Statements

Switches not testing enums should always have a default; treat an impossible default as an error. For enums, the compiler can warn about unhandled values. Fallthrough must be annotated with `[[fallthrough]];`; the annotation should be placed at the execution point where fallthrough occurs; consecutive case labels with no intervening code need no annotation.

```cpp
switch (x) {
  case 41:
  case 43:
    DoSomethingSpecial();
    [[fallthrough]];
  default:
    DoSomethingGeneric();
    break;
}
```

## Inclusive Language

Use inclusive language throughout names, comments, and code. Avoid disrespectful/offensive terms (including “master/slave”, “blacklist/whitelist”, and “redline”) even when ostensibly neutral. Use gender-neutral language unless referring to a specific person with their pronouns: singular “they/them/their” for unspecified people; “it/its” for software and other nonpeople.

## Naming

Follow consistent category-specific conventions; English words are lowercase with underscores or capitalized as described below.

### Choosing Names

Make purpose/intent understandable to a new reader from another team. Names should remain descriptive far from declarations, should not repeat immediate context, and should scale in descriptiveness with visibility. Clarity matters more than horizontal space.

Minimize unfamiliar abbreviations, acronyms, and initialisms; do not delete internal letters (`cstmr`). Prefer to capitalize abbreviations as one word (`StartRpc`, not `StartRPC`). Wikipedia-listed abbreviations are a useful rule of thumb; universally understood `i` loop counters and `T` template parameters are fine. Rare new, widely reused vocabulary names require additional design review and long-form documentation.

### File Names

Filenames should be lowercase, with `_` or `-` following project practice; prefer `_` without a local convention. C++ files should use `.cc`, and headers should use `.h`; use `.inc` for special textually included fragments. Tests use `_test`; `_unittest` and `_regtest` are deprecated. Do not reuse `/usr/include` filenames such as `db.h`. Prefer specific names (`http_server_logs.h`, not `logs.h`); paired `foo_bar.h`/`.cc` commonly define `FooBar`.

### Type Names

Type names should use PascalCase without underscores: `UrlTable`, `MyExcitingEnum`. Applies to classes, structs, aliases, enums, and type template parameters.

### Concept Names

Follow type naming rules.

### Variable Names

Use `snake_case` for variables and parameters. Class data members, static or nonstatic, add a trailing underscore (`table_name_`), except static constants following constant rules. Struct data members have no trailing underscore, including static ones (`num_entries`).

### Constant Names

For const/constexpr variables whose value is fixed throughout the program, use `k` plus PascalCase (`kDaysInAWeek`); underscores may separate otherwise inseparable components (`kAndroid8_0_0`). All such static-storage variables should follow this convention, including static class constants and template constants whose values differ by instantiation. For other storage classes it is optional; ordinary variable rules otherwise apply. A per-call const value that varies between calls is not a `k` constant:

```cpp
const int kDaysInAWeek = 7;
const std::string combined = absl::StrCat(prefix, suffix);  // Not kCombined.
```

### Function Names

Ordinarily use PascalCase (`AddTableEntry`, `DeleteUrl`). API constants meant to look like functions follow this rule too. Accessors/mutators may use variable-style `snake_case`, e.g. `count()`/`set_count()`, whether or not tied to actual data members.

### Namespace Names

Use `snake_case`. Top-level namespaces must be globally unique and recognizable; each should be owned by one project/team and named for it. Code should normally live in corresponding directories. Outside a namespace in headers, qualify names rather than introducing convenience public aliases. Nested names should avoid matching well-known top-level namespaces, especially `std`/`absl`; nesting does not prevent collisions.

### Enumerator Names

Scoped and unscoped enum values should use constant-style `kEnumName` rather than macro-style `ENUM_NAME`. New code should use this convention even where old code predates it.

### Template Parameter Names

Type parameters should follow type naming; non-type parameters should follow variable or constant naming.

### Macro Names

Macros should generally be avoided; when needed they should use uppercase words/underscores and a project prefix (`MYPROJECT_ROUND`).

### Aliases

Treat an alias name as any other new name, following its category in the context where the alias is defined rather than the original context.

### Exceptions to Naming Rules

An entity analogous to an existing C/C++ entity (or Rust via interop) may follow its existing naming scheme: `bigopen()` like `open()`, STL-like `sparse_hash_map`, or `LONGLONG_MAX` like `INT_MAX`.

## Comments

Write for the next maintainer. Prefer clear self-documenting names/code to explaining obscurity; comment the nonobvious.

### Comment Style

Use `//` or `/* */` consistently; `//` is much more common.

### File Comments

Start each file with project-appropriate license boilerplate. A file declaring multiple user-facing abstractions should describe their collection and boundaries; put individual abstraction details with those abstractions. A header collection comment need not be repeated in its `.cc` or test. A `.cc` containing a collection without an associated header must provide the file comment.

Every file should contain license boilerplate. New files should usually omit copyright notices and author lines. Consider deleting an author line after significant changes.

### Struct and Class Comments

Every nonobvious class/struct declaration should explain purpose, use, and considerations needed for correct use. Document synchronization assumptions, and rules/invariants for multithreaded use. A small focused usage example can help. Usage documentation should accompany the interface; implementation commentary should accompany the implementation.

### Function Comments

Almost every declaration, including private methods and `.cc` functions, should have an immediately preceding comment describing what it does and how to use it. Omit only for simple, obvious functions. Function comments should use an implied “This function” subject and start with a verb phrase (“Opens the file”, not “Open the file”). They should describe use rather than implementation, including relevant inputs/outputs, whether pointer/reference arguments are retained, each pointer's nullability and null behavior, whether output/in-out state is appended/overwritten, and performance implications. Implementation details should be left to definition comments. Backticks around argument names help indexing tools.

Do not state the obvious or be needlessly verbose. For overrides, document specifics rather than repeat base documentation; often no extra comment is needed. Constructor/destructor comments should explain nonobvious ownership, argument use, or cleanup, not merely “constructs/destroys”.

Definition comments should explain tricky operation, coding techniques, steps, choices, and locking decisions. They should not simply repeat declaration comments; a brief recap is OK, but the focus should be on how/why.

### Variable Comments

Each class member's purpose must be clear. Comment invariants, relationships, lifetimes, and sentinel meanings (e.g. `-1` or null) not clear from type/name; no comment is needed when they suffice. All globals should explain what they are, their use, and, if unclear, why global scope is needed.

### Implementation Comments

Tricky, nonobvious, interesting, or important implementation parts should have comments; explanations should precede complicated blocks. For unclear call arguments, consider named constants for values tacitly shared among calls, enum instead of bool, an options struct for multiple settings, or named variables for nested expressions. As a last resort use `/*argument_name=*/` comments at the call site.

```cpp
ProductOptions options;
options.set_precision_decimals(7);
CalculateProduct(values, options, /*completion_callback=*/nullptr);
```

Explain higher-level intent/why; do not narrate obvious C++ operations. Prefer self-describing code when possible.

### Punctuation, Spelling, and Grammar

Write readable comments with proper capitalization, punctuation, spelling, and grammar. Complete sentences are often clearer; short end-of-line comments may be less formal, consistently.

### TODO Comments

Use `TODO` for temporary, short-term, or imperfect code. `TODO`s should be uppercase and include an identifier for the person/issue with context: `// TODO(bug 12345): Remove the old feature.` Names normally identify the author adding it, not a commitment to fix. Future work must name a specific date or event, not vague timing.

## Formatting

### Line Length

Maximum **80 characters**, except unsplittable readable/copyable/autolinked comments (e.g. commands/URLs), string literals that cannot reasonably be wrapped, include statements, guards, and using-declarations. Except in tests, such long literals should be at namespace scope near the file top. Disable formatting around unsplittable content if necessary. Embedded languages, meaningful newlines, and semantically significant URLs may justify the literal exception.

### Non-ASCII Characters

Non-ASCII should be rare and **must be UTF-8**. You should not hard-code user-facing text, even English. Delimiters for foreign data and unlocalized tests may justify Unicode. Hex/Unicode escapes are allowed and encouraged when clearer, especially invisible characters (`"\uFEFF"`). Avoid `u8` when possible because its type semantics change across language versions. You should not use `char16_t`/`char32_t` or `wchar_t`, except that `wchar_t` may be used for Windows API interaction.

### Spaces vs. Tabs

Use spaces only, indenting two spaces. You should configure the editor to insert spaces for Tab.

### Function Declarations and Definitions

Keep return type, name, and parameters on one line when they fit. Wrap like calls: align to the first parameter, or break after `(` and indent parameters four spaces. If return type/name cannot share a line, break between them without indenting the name. `(` always shares the name's line with no intervening space; no padding inside parentheses. Put the opening `{` at the end of the final signature line. There should be one space between `)` and `{`. Closing `}` stands alone or shares the opening brace's line. Default body indent is two spaces; parameters should be aligned when possible.

```cpp
ReturnType ClassName::FunctionName(Type par_name1, Type par_name2) {
  DoSomething();
}
ReturnType LongClassName::ReallyLongFunctionName(
    Type par_name1, Type par_name2) {
  DoSomething();
}
```

Choose meaningful parameter names. Omit names only for unused parameters; if meaning is nonobvious, the name should be commented out in the definition (`void Circle::Rotate(double /*radians*/) {}`). Obvious unused copy parameters may be unnamed. Attributes/macros expanding to attributes precede the return type at the very start of a declaration/definition.

### Lambda Expressions

Format parameters/bodies like functions and captures like comma-separated lists. No space between `&` and a reference capture name. Short lambdas may appear inline as call arguments:

```cpp
auto x_plus_n = [&x](int n) -> int { return x + n; };
```

### Floating-point Literals

Floating-point literals should always include a radix point and digits on both sides, including exponent forms: `1.0f`, `-0.5`, `1248.0e6`; not `1.f`, `-.5`, `1248e6`. Integer literals may initialize floating variables when exactly representable; exponent notation is never an integer literal.

### Function Calls

Keep calls on one line when possible. Otherwise align continuations with the first argument, or put all arguments on subsequent lines with four-space indentation. No padding inside parentheses. Absent other considerations, minimize lines and allow multiple arguments per line; do not impose one argument per line merely for easier editing.

Name complex arguments or comment a confusing argument on its own line. Put an argument alone only for its specific readability benefit. Formatting that exposes meaningful structure, e.g. matrix rows, is allowed.

```cpp
bool result = DoSomething(long_argument,
                          argument2, argument3);
bool result2 = DoSomething(
    argument1, argument2,
    argument3, argument4);
```

### Braced Initializer List Format

Format like a function call in the same place: a preceding type/variable name is the call name; otherwise assume a zero-length name. Examples: `return {foo, bar};`, `Function({foo, bar});`, `Type value{foo, bar};`. Apply call wrapping/alignment to multiline/nested lists.

### Looping and Branching Statements

Statement keywords, parenthesized condition/iteration, and controlled blocks should be separated by single spaces, not line breaks. Do not pad inside condition parentheses. After each semicolon in a condition/iteration, use a space or line break unless the next token is `)` or another semicolon. Use braces around controlled statements, with a line break immediately inside each brace.

```cpp
if (condition) {
  DoOneThing();
} else if (int a = f(); a != 3) {
  DoAnotherThing(a);
} else {
  DoNothing();
}
for (int i = 0; i < 10; ++i) {
  RepeatAThing();
}
do {
  RepeatAThing();
} while (condition);
```

For historical reasons, allowed but discouraged: omit internal brace line breaks when the entire statement fits one line, or omit braces if it fits one line or exactly two lines (single-line condition, single-line body). This exception **does not apply to multi-keyword** `if ... else` or `do ... while`. A comment or multiline condition makes the braceless form too long. Use only for brief statements; some projects require braces always.

```cpp
if (x == kFoo) return new Foo();  // Allowed, discouraged.
if (x == kBar)
  Bar(arg1, arg2);                // Allowed, discouraged.
while (condition) {}             // Allowed empty loop.
while (condition) continue;      // Allowed empty loop.
// while (condition);            // Bad: semicolon body.
```

Case blocks may use braces or omit them. With braces, case labels should be indented two spaces and the body four, with `{` after `case n:` and its close at the case indent. Empty loop bodies should use `{}` or unbraced `continue` rather than a lone semicolon; normal brace/newline rules and their brief-statement exception apply.

### Pointer and Reference Expressions and Types

No spaces around member `.`/`->`, or after unary `*`/`&`: `x = *p;`, `p = &x;`, `r->y`. In declarations/types, do not space before `*`/`&`; separate the completed type from the name: `char* c;`, `const std::string& str;`, `std::vector<char*>`. Multiple declarations are allowed if helpful (`int x, y;`) but forbidden if any declarator has pointer/reference decoration (`int x, *y;` is forbidden).

### Boolean Expressions

Break long expressions consistently with all logical operators at line ends (more common) or all at beginnings. Use parentheses judiciously without overuse. You should always use punctuation operators (`&&`, `~`), not word forms (`and`, `compl`).

### Return Values

Parenthesize a return expression only where the equivalent assignment expression would need it. Use `return value;`, not `return (value);` or `return(value);`.

### Variable and Array Initialization

`=`, `()`, and `{}` are allowed. Nonempty braces prefer an initializer-list constructor; empty braces use an available default constructor. Use parentheses to force another constructor. Braces reject narrowing:

```cpp
std::vector<int> v(100, 1);  // 100 elements, each 1.
std::vector<int> w{100, 1};  // Two elements.
int pi{3.14};               // Error: narrowing.
```

### Preprocessor Directives

A directive’s `#` should always be in column one, even inside indented code. Spaces after `#` are allowed but not required for nested directives.

### Class Format

Base names should be on the class-name line when the line limit permits. Access sections should be ordered public/protected/private, and their labels should be indented **one** space; indent declarations two spaces. Subsequent access labels should have a preceding blank line (optional for small classes); do not put a blank line immediately after a label. Follow declaration ordering above.

```cpp
class MyClass : public OtherClass {
 public:
  explicit MyClass(int var);
  int some_var() const { return some_var_; }

 private:
  int some_var_;
};
```

### Constructor Initializer Lists

Keep on one line if possible. Otherwise break before `:` with four-space indentation. If spanning several lines, put each member on its own line, aligned with the first initializer:

```cpp
MyClass::MyClass(int var)
    : some_var_(var),
      some_other_var_(var + 1) {
  DoSomething();
}
```

An empty body may use `{}` on the same line.

### Namespace Formatting

Do not indent namespace contents. Close with the namespace comment; unnamed namespaces use `// namespace`.

### Horizontal Whitespace

Never introduce trailing whitespace. Remove it when changing its line, or separately in a cleanup preferably when others are not editing. Use two spaces before end-of-line comments. An opening body brace should always have a preceding space. Put spaces around inheritance/initializer colons and around inline body contents (`void Reset() { x_ = 0; }`); empty braces have no internal spaces. Semicolons normally have no preceding space. Braced-list internal spaces are optional but must be symmetric.

Use space after control keywords and around `else`; usually none inside condition parentheses (rare local consistent padding is allowed). For-loop semicolons have following spaces as above; preceding space is rare but allowed. Range-for colons have spaces on both sides. Case labels have no space before `:`, and space after it if followed by code.

Assignments always have surrounding spaces. Other binary operators usually do; factor spacing may be omitted (`w*x + y/z`). Parentheses should have no internal padding; do not separate unary operators from operands. Do not space before or inside template `<...>`, between cast `>` and `(`, or between a type and its pointer decoration: `static_cast<char*>(x)`.

### Vertical Whitespace

Use blank lines sparingly to separate meaningful related chunks. Do not add them at block beginnings/ends where indentation already delineates structure. Within a declaration/statement, usually break only to obey the length limit or attach a comment to part of it.

## Exceptions to the Rules

The coding conventions above are mandatory, with the stated recommendation strengths and the exceptions below.

### Existing Non-conformant Code

When modifying code written to other conventions, local consistency may require divergence. Ask the original author/current owner if uncertain.

### Windows Code

Follow common C++ conventions: no Hungarian notation; use Google naming and `.cc`. Windows API type synonyms (`DWORD`, `HANDLE`) are acceptable and encouraged for API calls, but stay close to underlying C++ types (`const TCHAR*` rather than `LPCTSTR`). Set MSVC warning level at least 3 and treat warnings as errors. Use path-based standard guards, not `#pragma once`. Avoid pragmas/`__declspec` and other extensions unless absolutely necessary; `__declspec(dllimport/dllexport)` is allowed through disableable macros such as `DLLIMPORT`/`DLLEXPORT`.

Windows-specific exceptions:

- Multiple implementation inheritance may implement COM or ATL/WTL interfaces/classes.
- With ATL, you should define `_ATL_NO_EXCEPTIONS` and should investigate disabling STL exceptions; if impossible, enabling compiler exceptions only to compile the STL is OK. Still do not write exception handling.
- Avoid explicitly including precompiled headers except in `precompile.cc`; use `/FI` to include automatically.
- Macro-only resource headers, typically `resource.h`, need not follow these guidelines.

## Sources and licensing

- Original title: **Google C++ Style Guide**; attributed to Google and the upstream contributors.
- Fixed source: [google/styleguide — cppguide.html at 572c673d9058242c049fbc8248b17029726c7afa](https://github.com/google/styleguide/blob/572c673d9058242c049fbc8248b17029726c7afa/cppguide.html).
- Official published guide: [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html).
- Documentation: [Creative Commons Attribution 3.0 Unported (CC BY 3.0)](https://creativecommons.org/licenses/by/3.0/).
- Code examples: [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0), as stated in the upstream README.
- Adaptation notice: this is an **editorially condensed adaptation**, not verbatim upstream prose. It shortens background, repeated rationale, and examples while retaining actionable rules, their strength, key exceptions, and original topic coverage. The attribution section and adaptation notices were added; source HTML presentation and navigation were removed. Consult the pinned source for full discussion.
- This is unofficial and is not affiliated with or endorsed by Google.
