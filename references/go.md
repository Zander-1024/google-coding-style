# Google Go style documents — editorially condensed adaptation

This is an **editorially condensed adaptation**, in one Markdown file, of Google's Go overview, Style Guide, Style Decisions, and Best Practices at commit `572c673d9058242c049fbc8248b17029726c7afa`. It preserves actionable guidance and important qualifications while shortening background and examples; it is not complete or verbatim. The fixed sources and official site are linked below for optional consultation. No external document is required to apply this adaptation.

## Overview and authority

| Source | Normative | Canonical | Role |
| --- | --- | --- | --- |
| Style Guide | Yes | Yes | Definitive, enduring foundation for authors and reviewers |
| Style Decisions | Yes | No | More detailed, potentially evolving guidance for consistent readability reviews; subordinate to the Guide |
| Best Practices | No | No | Recommended patterns, applied with judgment; not additional mandatory rules |

**Canonical** means prescriptive, enduring guidance expected of old and new code. **Normative** means agreed style for consistent review suggestions, terminology, and reasoning; authors are not expected to memorize the noncanonical decisions. **Idiomatic** means common and familiar Go usage, normally preferable when equally suitable.

These documents establish principles and idioms, explain tradeoffs, and reduce surprises in reviews. They are neither exhaustive review checklists nor replacements for judgment, and do not justify large-scale style churn. Follow the guide even where personal preference differs; improve nearby code as appropriate without nitpicking every deviation. Familiarity with [Effective Go](https://go.dev/doc/effective_go) is assumed. Additional references are optional learning/review context, not expected knowledge: the [language specification](https://go.dev/ref/spec), [FAQ](https://go.dev/doc/faq), [memory model](https://go.dev/ref/mem), [Go Data Structures](https://research.swtch.com/godata), [Go Interfaces](https://research.swtch.com/interfaces), and [Go Proverbs](https://go-proverbs.github.io/). The overview lists Go Tips and Unit Testing Practices as future resources. Its testing articles cover [identifier naming](https://testing.googleblog.com/2017/10/code-health-identifiernamingpostforworl.html), [state versus interactions](https://testing.googleblog.com/2013/03/testing-on-toilet-testing-state-vs.html), [effective testing](https://testing.googleblog.com/2014/05/testing-on-toilet-effective-testing.html), [risk-driven testing](https://testing.googleblog.com/2014/05/testing-on-toilet-risk-driven-testing.html), and [change-detector tests](https://testing.googleblog.com/2015/01/testing-on-toilet-change-detector-tests.html). Further perspectives: [Go and Dogma](https://research.swtch.com/dogma), [Less is exponentially more](https://commandcenter.blogspot.com/2012/06/less-is-exponentially-more.html), [Esmerelda's Imagination](https://commandcenter.blogspot.com/2011/12/esmereldas-imagination.html), [regular expressions for parsing](https://commandcenter.blogspot.com/2011/08/regular-expressions-in-lexing-and.html), and [gofmt's uniformity](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=8m43s).

## Style Guide — normative and canonical

### Style principles

Apply these priorities **in order**: clarity, simplicity, concision, maintainability, consistency.

- **Clarity:** Optimize for the reader. Names, comments, whitespace, organization, and small functions should reveal what code does. Explain why when language nuances, business rules, performance choices, API hazards, or mathematical conversions would otherwise require reverse engineering. Prefer self-describing code; redundant, stale, or contradictory comments add clutter. Unusual code should draw attention because it needs unusual care.
- **Simplicity:** Choose the simplest implementation and API that satisfy behavior and performance needs. Read top to bottom without unnecessary abstractions, hidden value propagation, or assumed prior knowledge. Complexity can be justified to simplify correct API use, support real customers, or meet performance needs; add it deliberately and document it, with tests and examples showing correct use.
- **Least mechanism:** First use core language constructs, then the standard library, then suitable existing core libraries before a new dependency or invention. A `map[T]bool` often suffices for membership; a set library needs more demanding operations. In tests, override a flag's bound value directly rather than calling `flag.Set`, unless testing the CLI itself.
- **Concision:** Maximize signal, not minimum character count. Remove needless repetition, syntax, abstractions, and whitespace while making important differences visible. Use idioms; when code subtly departs from an idiom, call it out, for example `if err == nil { // if NO error`.
- **Maintainability:** Make future correct changes easy. Use APIs that can grow, explicit assumptions, problem-oriented abstractions, minimal coupling and dependencies, and comprehensive tests with actionable diagnostics. Interfaces hide concrete information and should provide enough benefit to justify that cost. Avoid easily missed semantic differences such as assignment instead of declaration in an `if`, or a buried negation; split or comment them. Reuse predictable names for identical concepts. Do not depend on undocumented internals; consider likely evolution when choosing a design.
- **Consistency:** Prefer consistency with the broader codebase, package, and file when higher priorities tie. Package consistency matters immediately but does not override the Guide or global principles.

### Core guidelines

- **Formatting:** All Go source **must** match `gofmt`. Generated code should generally be formatted too, for example with `go/format.Source`.
- **MixedCaps:** Use `MixedCaps` for exported names and `mixedCaps` for unexported names, including constants and locals; not `MAX_LENGTH` or `max_length`.
- **Line length:** There is **no fixed code line limit**. Refactor an awkward long line; let it remain long if already as short as practical. Do not wrap immediately before an indentation change, or split a long string such as a URL merely to shorten lines.
- **Naming:** Use context-sensitive, nonrepetitive names; omit concepts already clear at the use site. Go names are often shorter than in other languages.
- **Local consistency:** Where the Guide is silent, follow a consistent nearby choice or choose freely if none exists. Error formatting `%s` versus `%v`, or buffered channels versus mutexes, can be local choices; code line limits and assertion libraries cannot. A limited existing deviation outside a change's scope can be tracked with a bug. Local consistency no longer justifies a change that worsens or spreads a deviation, exposes it through more APIs, or introduces a bug: clean up, refactor first, or choose an alternative.

## Style Decisions — normative, not canonical

These decisions standardize readability review guidance and are not exhaustive. The canonical Guide wins in a conflict. Material explicitly called a **tip** or **best practice** remains advisory.

### Naming decisions

- **Underscores:** Names should generally have none. Exceptions: packages only imported by generated code; Test, Benchmark, and Example function names in `*_test.go`; and rare OS/cgo interoperability names such as `syscall` identifiers. Filenames are not identifiers and may contain underscores.
- **Packages:** Names **must** be concise lowercase letters and digits: `k8s`, `oauth2`, `tabwriter`, not `tabWriter` or `tab_writer`. Avoid names commonly shadowed by locals and vague names such as `util`, `common`, `helper`, `model`, or `testhelper`. Rename imported underscore-containing packages to valid local names, except the generated-code-only case. Black-box tests for `linkedlist` use `linkedlist_test`; functional/integration tests may use names such as `linked_list_service_test`; package documentation examples may use `_test`. Renamed imports follow these rules and should use the same local name consistently across nearby files.
- **Receivers:** Names **must** be short, usually one or two letters, abbreviate the type, and be consistent for every method on it: `t Tray`, `ri *ResearchInfo`, `w *ReportWriter`. Do not use `this`, `self`, or `_`; omit an unused receiver name.
- **Constants:** Names **must** use MixedCaps, without a `K`/`k` prefix. Explain the role (`MaxPacketSize`, `ExecuteBit`), not merely the literal value (`Twelve`). A value with no role beyond itself need not become a named constant.
- **Initialisms:** Keep each initialism's conventional case: `URLPony`, `urlPony`, `XMLAPI`, `xmlAPI`, `ID`/`id`, `DB`/`db`, and `Txn` rather than `TXN`. Preserve mixed-case acronyms such as `iOS`, `gRPC`, `DDoS`, unless exportedness requires changing the first letter; then use consistent case for the whole initialism: `IOS`, `GRPC`, `ddos`. Do not use `Url`, `XmlApi`, or `Id`.
- **Getters:** Usually omit `Get`/`get`: `Counts`, not `GetCounts`. It is appropriate when the underlying concept itself is “get,” such as HTTP GET. `Compute` or `Fetch` can signal expensive computation, blocking, or possible failure.
- **Variables:** Name length should grow with scope and shrink with frequent use; line-count ranges in the source are rough aids, not limits. Multiple similar values may need longer names. Familiar, specific abbreviations such as `db` can stay clear in large scopes. Describe current contents and use, not origin. Start with `count` or `options`, disambiguate with `userCount` if needed, and do not merely drop letters (`Sandbox`, not `Sbx`). Usually omit type words: `users`, not `userSlice`; use `ageString` when both textual and parsed forms coexist. Single letters suit receivers, familiar `r`/`w`, indices and coordinates, or short loops where the full word is obvious.
- **Repetition:** Judge names at the use site. Prefer `widget.New`, `widget.NewWithName`, `db.Load`, `sqldb.Connection`, and `(*Project).Name` over repeated package/type names. A package exporting one same-named type uses `New` as its canonical constructor name if needed. Omit type and surrounding context from locals unless distinguishing representations (`limitRaw` or `limitStr` versus parsed `limit`). Code repetition can also conceal meaningful differences.

### Commentary decisions

- **Comment line length:** No fixed limit. Wrap prose for tools without automatic wrapping; 80 or 100 columns are common choices, not cutoffs. Be consistent in a file and preserve long URLs or other text when breaking hurts. Avoid huge single-line paragraphs.
- **Doc comments:** Every top-level exported name **must** have one; unexported types/functions with unobvious meaning or behavior should too. Use full sentences beginning with the described name, optionally preceded by an article: “A Request represents …”. Write for package users. A comment in a struct can document a group of following fields. As a best practice, use the same naming convention for unexported doc comments.
- **Comment sentences:** Complete sentences use English capitalization and punctuation; a clear uncapitalized identifier may start a sentence, preferably at paragraph start. Fragments and short trailing field comments need not be sentences. Doc comments should always be complete sentences.
- **Examples:** Clearly document intended usage; try to provide runnable examples in test files, where Godoc can display them and tests can run them. If infeasible, use conventionally formatted code in comments. Previewing rendered docs and examples is a best practice. **Tip:** Godoc code/list text should usually be indented to avoid wrapping; other decoration should generally be avoided.
- **Named results:** Name results when names add useful meaning, such as `(left, right *Node, err error)` or a returned `cancel` action the caller must invoke. Avoid redundant `(node *Node)` and names added merely to save a local declaration or enable a naked return. Naked returns are acceptable only in small functions; use explicit returned values once medium-sized. Naming a result changed by a deferred closure is always acceptable. **Tip:** Types can often be clearer than names in signatures; a named result type such as `context.CancelFunc` can be clearer than a name plus raw function type.
- **Package comments:** Exactly one per package, immediately above the `package` clause with no blank line. Use `Package math …` for libraries. For `main`, describe the binary with its name exactly as in its BUILD target, for example “The seed_generator command …”; capitalize it if it is the first word. Put a long comment, or one with no obvious primary file, in `doc.go` if useful. Indented command/API examples and multiline comments are fine. Whole-file maintainer comments typically go after imports and are not package Godoc.

### Import decisions

Do not normally rename imports. A collision with another import **must** be resolved; prefer renaming the most local or project-specific one. A generated protocol buffer package **must** have a valid local name ending in `pb`, for example `foosvcpb "path/foo_service_go_proto"`. An uninformative nongenerated name (`util`, `v1`) **may** be renamed sparingly if nearby context is insufficient; prefer fixing its package name. For a desired local variable colliding with a still-needed package, prefer a `pkg` suffix (`urlpkg`). Ordinary shadowing does not itself require renaming. Keep aliases consistent.

Group imports in order, separated by blank lines: (1) standard library, (2) other project and vendored packages, (3) protocol buffer packages, (4) side-effect imports.

Blank imports (`_`) **may only** appear in `main` or tests requiring them, except to bypass a disallowed-import check in `nogo`, or importing `embed` in a file using `//go:embed`. **Tip:** A library indirectly needing a production side-effect import should document its intended usage so main can supply it. **Do not use dot imports** (`import .`); keep symbols qualified.

### Error decisions

- **Returning errors:** Use `error` to signal failure, conventionally last; `nil` indicates success. On a nonnil error, callers **must** treat other results as unspecified unless explicitly documented. Exported functions should return the interface `error`, not a concrete error pointer that can become a nonnil interface containing nil. **Tip:** A function accepting a context should usually return an error so cancellation is observable.
- **Strings:** Error strings should start lowercase, except exported identifiers, proper nouns, or acronyms, and have no final punctuation. A complete displayed log, test, API, or UI message usually starts uppercase.
- **Handling:** Deliberately handle, return, or exceptionally terminate for each error. Do not normally discard with `_`; explain why ignoring a particular error is safe, such as `bytes.Buffer.Write` never failing. `log.Fatal` is exceptional; `panic` is a last resort. `log` here is Google's logging variant, described below.
- **In-band errors:** Prefer an additional final `error`, or `bool` if no explanation is needed, instead of sentinel data such as `-1`, empty string, or nil. For example `Lookup(key) (value string, ok bool)` makes unchecked composition impossible. Some standard-library string APIs use in-band errors; do not generalize that into new Google APIs.
- **Indent error flow:** Handle a terminal/error case first and leave the normal path unindented after it; do not add `else` after return, continue, panic, or fatal termination. **Tip:** If a value will be used for more than a few lines, declaring before the `if` is generally clearer than using an `if` initializer.

```go
// Good: normal flow remains at the main indentation level.
x, err := f()
if err != nil {
    return err
}
use(x)

// Bad: success is buried in else.
if x, err := f(); err != nil {
    return err
} else {
    use(x)
}
```

### Language decisions

**Composite literal formatting:** Prefer literals over field-by-field construction when possible.

- Fields **must** be named for structs from other packages; field order is not generally public API. Local structs may omit names, but use them for clarity, especially many fields.
- A closing brace should be at the indentation of its opening line. For a multiline literal put a trailing comma after the final value and the closing brace on its own matching line, rather than attaching it to the final field.
- Cuddled braces for slice/array literals are permitted **only** when indentation matches and inner values are literals or proto builders, not variables or other expressions.
- Repeated element type names may be elided in slice/map literals. Repetition is reasonable for unusual complex types or elements far apart; **Tip:** `gofmt -s` can simplify repetitive type names.
- Zero-value fields may be omitted if clarity survives. Named fields help table rows omit irrelevant zeros; explicitly include a zero/nil field when that value is the point of a test.

```go
// Good: elided types, named fields, and matching braces.
good := []*Type{
    {Key: "multi"},
    {Key: "line"},
}
// Also permitted: all inner elements are literals and braces match.
cuddled := []*Type{{
    Key: "first",
}, {
    Key: "second",
}}
// Bad: closing brace shares the final value's line.
bad := []*Type{
    {Key: "multi"},
    {Key: "line"}}
```

**Nil slices:** Prefer `var s []string` for an empty local slice, especially one returned to callers. Nil supports length, capacity, range, and append normally. Do not design APIs that distinguish nil from a nonnil empty slice; use `len(s) == 0` for emptiness and explicit errors for failure.

**Indentation confusion and function formatting:** Do not wrap a line so its continuation looks like the following indented body. If unavoidable, separate continuation and body with a blank line. Function/method declaration signatures should stay on one line. Do not split calls solely for length; factor local variables or redesign excessive parameters. Avoid inline comments on individual arguments; prefer an option struct or API documentation. Line breaks are permitted when the API cannot be changed or semantic groupings improve comprehension, such as one polygon vertex per line. Keep long format strings intact; group following arguments by meaning.

**Conditionals and loops:** Keep `if`, `for`, `switch`, and `case` clauses on one line rather than artificial wraps. Extract boolean operands only when short-circuit behavior is unnecessary; factor repetitive expressions and move loop conditions into a body `if`/`break` when useful. Closures and multiline literals in conditionals should keep matching braces. For an exceptionally long case list, put `case` alone, indent *all* its values, and put a blank line before its body. Put the variable on the left in comparisons: `result == "foo"`, not `"foo" == result`.

**Copying:** Be careful copying other packages' structs; never copy synchronization objects such as `sync.Mutex`. A `bytes.Buffer` copy may alias internal storage. Generally do not copy `T` when its methods belong to `*T`. A value receiver silently copies too. APIs for structs with noncopyable fields should generally take/return pointers. **Tip:** Check the type's Godoc for whether it is safe or unsafe to copy.

**Panic and Must functions:** Do not panic for normal errors; return errors. In `main`/initialization, consider `log.Exit` for fatal configuration errors without useful stack traces; it calls `os.Exit`, so defers do not run. For impossible conditions representing bugs, returning an error or `log.Fatal` can be reasonable. Setup helpers terminating on failure use `MustXYZ`/`mustXYZ`, generally only early startup/package initialization, such as `regexp.MustCompile` on a constant. Test value-construction helpers may use this convention; in tests, these helpers should generally call `t.Helper` and `t.Fatal` on error. They should not be called where it is difficult to ensure an error would be caught, or where errors should be checked (for example, many request handlers), nor when ordinary error handling is possible, including with some refactoring. For constant inputs this permits tests to check that Must arguments are well-formed; for nonconstant inputs it permits tests to validate normal error handling or propagation.

**Goroutine lifetimes:** Make when and whether spawned goroutines exit evident, or document it. Blocked goroutines are not garbage-collected. Unfinished goroutines can leak resources, send on closed channels, race with mutated inputs, and make tests or shutdown indeterminate. Usually confine synchronization to a function, factor business logic into synchronous functions, arrange cancellation, and join workers before returning (for example with a `WaitGroup`). Contexts, signal channels, synchronized variables, and condition variables are possible mechanisms; an obvious lifetime is the requirement.

**Interfaces:** Avoid creating one before a real need exists; model required behavior, not abstract “service/repository” patterns. Do not manually wrap RPC clients just for abstraction/testing; test real transports. Do not add back doors or exported doubles solely for testing; test the real public API. Keep interfaces small, document contracts/edges/errors, and keep internal interfaces unexported. Consumers should own them; a producer may expose an interface when the common protocol is the product to avoid redefinition. Usually accept interfaces and return concrete types so callers keep the full API; interface returns can suit encapsulation (`error`) or command/chaining/factory/strategy patterns.

**Generics:** Allowed when real requirements justify them. Prefer existing slices, maps, or interfaces where sufficient; do not generalize a data structure instantiated with only one type in practice. Adding polymorphism later is easier than removing needless abstraction. Document exported generic APIs and strongly consider motivating runnable examples. Do not build DSLs, error-handling frameworks, assertion libraries, or tests with poor diagnostics using generics. Consider an existing useful interface first; generics can be preferable to `any` plus excessive type switches.

**Passing values:** Do not use pointers just to save a few bytes, especially `*string` or `*io.Reader` read only via dereference. This does not rule out pointers to large or growing structs. Generally pass protocol buffer messages by pointer: their pointer type implements `proto.Message`, and messages may be large and grow.

**Receiver type:** Correctness takes priority over speed and brevity.

| Situation | Receiver guidance |
| --- | --- |
| Method mutates receiver value itself, reslices/reallocates slice, or struct contains noncopyable fields | **Must** use pointer for required mutation; use pointer for noncopyable fields |
| Slice without reslice/reallocation; map, function, channel; immutable built-in scalar/string | Use value |
| Large struct/array, uncertain future growth, or doubt | Prefer pointer |
| Concurrent modifications | Use value if those changes should be invisible; otherwise pointer |
| Struct/array contains pointers to mutable state | Prefer pointer to communicate mutability |
| Small natural value with no mutable fields/pointers | Usually value |

Generally keep a type's methods uniformly pointer or uniformly value methods. Do not base the choice on folklore about allocations; use realistic benchmarks if performance matters.

**Switch/break:** Do not put redundant unlabelled `break` at case ends. Use a comment for an intentional empty clause. A `break` in a switch exits that switch, not an enclosing loop; use `break label` to exit the loop.

**Synchronous functions:** Prefer functions that return their result and finish callbacks/channel operations before returning. Callers can add a goroutine; they cannot easily remove unwanted concurrency. Local lifetimes are easier to test and reason about.

**Type aliases, quoting, any:** Define a new type with `type T1 T2`; `type T1 = T2` is a rare alias mainly for package migration, and should not be used unnecessarily. Prefer `%q` over manually quoting `%s`, particularly for human-facing possibly empty/control-character strings. Prefer `any` over `interface{}` in new code (Go 1.18+).

### Common library decisions

**Flags:** Google's internal `flag` variant resembles the standard library. Prefer snake_case flag names and corresponding camelCase variables: `poll_interval` / `pollInterval`. Flags **must only** be defined in `main` or equivalent; libraries should expose explicit Go configuration APIs. In an extremely rare necessary exception, the flag name **must** identify its package. Put global flags in a separate `var` group following imports.

**Logging:** Here `log` means Google's internal logging package, with open-source analogue [glog](https://pkg.go.dev/github.com/golang/glog), rather than the standard `log` API. `log.Fatal` aborts with a stack trace; `log.Exit` exits without one; there is no `log.Panic`. **Tip:** Prefer `log.Info(v)` to `log.Infof("%v", v)` when no formatting is needed; equivalent non-formatting APIs apply at other logging levels. Do not silently substitute standard-library exit behavior in examples.

**Contexts:** Explicitly propagate `context.Context` through the call chain. It is **always first** when passed as a parameter, including test helpers. Exceptions: HTTP handlers get it from `req.Context()`, streaming RPCs from the stream, and Test/Benchmark/Fuzz entrypoints from `testing.TB.Context()`. Use `context.Background()` at true entrypoints (`main`, library `init`). Prefer a caller context in intermediate/library code; Google's server frameworks may create request-scoped contexts populated from incoming request data. Outside entrypoints, discuss a perceived need for Background with Google's Go style group before implementation. Background operations that must outlive cancellation can use Google's internal detachment mechanism. Identical immutable contexts may serve multiple calls sharing deadlines, credentials, traces, and cancellation.

Do not store a context in a struct: pass it to each method needing it. The rare exception is matching an uncontrollable standard/third-party interface signature, discussed with Google's style group before implementation/review. In tests prefer `TB.Context()` (added in Go 1.24); explicitly pass it to setup, helper, and double functions. **Do not create custom context types or accept alternate context interfaces; there are no exceptions.** Put application data in appropriate ordinary parameters, receivers, globals, or context values only if it truly belongs there.

**Random keys:** Never use `math/rand` for keys, even throwaway ones. Use `crypto/rand`, handle failures, and hex/base64 encode if text is needed.

### Useful test failure decisions

A failure should be diagnosable without reading the test source: show what failed, relevant inputs, actual result, and expectation.

- **Assertion libraries:** Do not create assertion libraries combining validation and failure reporting, or third-party test DSLs. Use Go control flow, `fmt`, and comparison tools. Domain-specific validation helpers should generally return a value/error for the test to report instead of taking `*testing.T`. Testing a nontrivial helper independently is a best practice. **Tip:** Further test-helper versus assertion-helper guidance appears under [Testing practices](#testing-practices).
- **Function and input:** Usually print the function name and short inputs: `YourFunc(%v) = %v, want %v`. Use descriptive named cases for large or opaque inputs, and print their description. Subtest names do not remove the requirement to identify inputs.
- **Got before want:** Print actual before expected and use “got”/“want.” Diffs are the exception to a fixed ordering: either ordering is allowed but **include a direction legend**.
- **Full structures:** Construct expected compound data and compare it deeply rather than manually checking every field, except when irrelevant fields obscure the test. `cmpopts` can express semantic/approximate equality or ignore incomparable/irrelevant fields. Multiple result values may be compared individually without wrapping in a struct.
- **Stable results:** Compare stable semantic information, not a dependency's incidental output bytes. For example parse JSON and compare meaning rather than require `json.Marshal`'s exact encoding.
- **Keep going:** Prefer `t.Error` for independent mismatches to report all failures in one run. Use `t.Fatal` when later checks would be meaningless or misleading, such as decoding a wrongly encoded value. For tables, consider subtests with `t.Fatal` rather than `t.Error` plus `continue`.
- **Equality:** `==` compares language-defined values, and pointers by identity. For compound values use [go-cmp](https://pkg.go.dev/github.com/google/go-cmp/cmp): `cmp.Equal` or `cmp.Diff`. Protobuf comparisons require `protocmp.Transform()`. Prefer cmp for new tests; existing `pretty` may remain for consistency, but its visual equality ignores distinctions, compares unexported fields, handles large values poorly, and is inappropriate for protobufs. Do not use `reflect.DeepEqual`; update such code. `cmp` can panic on misuse and is for testing, not production.
- **Detail:** Describe which interaction/call and relevant system state failed. It is acceptable to show only important parts of boilerplate-heavy data without obscuring it; setup failures need less detail. `%q` clarifies strings, `%+v` small structs, and diffs large objects. **Tip:** Trigger failures during development to inspect the diagnostics and whether a maintainer can effectively respond.
- **Diffs:** Prefer `cmp.Diff` for large objects; string diff tools may suit multiline strings. Match the legend to actual argument order, e.g. `(-want +got)` for `cmp.Diff(want, got)`, and print a newline before a multiline diff. Other tools with arrow output need an appropriate arrow legend.
- **Error semantics:** Do not identify error kinds through message equality or vanilla cmp; use nil presence, `errors.Is`, or `cmpopts.EquateErrors`. Checking properties of your own package's message (e.g. includes a parameter name) is permitted. Ask a dependency owner to add semantic information rather than parse its prose. If all expectations are nil or AnyError, use a `wantErr bool` instead of cmp machinery.

```go
// Good: full protobuf comparison and an explicit diff legend.
if diff := cmp.Diff(want, got, protocmp.Transform()); diff != "" {
    t.Errorf("Foo(%q) returned unexpected difference (-want +got):\n%s", input, diff)
}
// Good: error presence needs no comparison framework.
if gotErr := err != nil; gotErr != test.wantErr {
    t.Errorf("f(%q) = %v, want error presence = %v", test.input, err, test.wantErr)
}
```

### Test structure decisions

**Subtests:** Useful, especially in tables, but optional. Cases should not depend on other cases for success or initial state, including when filtered with `go test -run` or Bazel filters. Choose short descriptive identifier-like names readable after test-runner escaping and easy to type. Avoid spaces/nonprinting characters (escaped or replaced) and especially `/` (filter separator). Keep long prose in a description field/log and inputs in failure messages.

**Table-driven tests:** Use a table when many cases share logic, whether exact output comparison or common invariants. Separate tests are appropriate for cases needing different test logic; a simple error-presence branch can be fine but should not grow a conditional test framework. Cases with equal setup but differing logic can be sequential subtests. Separating normal and error tables is often best. Do not use row values to choose substantially different setup/implementations; explicit duplicated tests can be clearer. Never identify a row only by its numeric index: use a description/subtest name and **always identify inputs**.

**Helpers:** A helper performs setup/cleanup; failures are environmental, not validations of the subject. If it takes `*testing.T`, call `t.Helper()` and place `t` after optional context and before other arguments. Do not use Helper to hide assertion failures or break the connection to failure conditions. Similar guidance applies to benchmark/fuzz helpers.

**Test packages:** Same-package tests use `foo_test.go`, `package foo`, and no explicit self-import; they can access unexported names, but examples omit the qualification users need. A separate `foo_test` package suits integration tests with no obvious library or cases where same-package tests create import cycles. For Bazel same-package tests, embed the tested library with the test rule's `library` attribute; different-package tests import it normally.

**Testing framework:** The standard `testing` package is the **only permitted** test framework for Google's Go code. Third-party testing frameworks and assertion libraries are not allowed; use its tests, benchmarks, examples, subtests, logs, and failure methods with ordinary Go.

### Non-decisions

The source intentionally leaves these choices to authors, subject to local consistency and optional review suggestions: `var i int` versus `i := 0`; `&File{}` versus `new(File)` and map literal versus `make`; got/want order in `cmp.Diff` **with a legend**; `errors.New("foo")` versus unformatted `fmt.Errorf("foo")`. The following Best Practices section may advise on these without making the choices normative.

## Best Practices — nonnormative, noncanonical advice

This entire section offers **recommendations**, common applications, and alternatives, not new mandatory style rules. Apply judgment to the circumstances; where it repeats a Decisions requirement, that requirement's authority comes from Decisions. The Guide and Decisions above take precedence.

### Naming practices

**Function and method names:** Read names as clients will call them. Omit repeated input/output types, receiver types or pointer-ness, package names, and argument/result names when already apparent: `yamlconfig.Parse`, `(*Config).WriteTo`, rather than `ParseYAMLConfig` or `WriteConfigTo`. Keep a disambiguator when needed (`WriteTextTo`/`WriteBinaryTo`). Nouns suit returned things; verbs suit actions. Avoid `Get` as described above. When the same operation exists for different types, a type suffix can distinguish versions (`ParseInt`, `ParseInt64`); the primary version can omit it (`Marshal`, `MarshalText`).

**Test double and helper packages:** A conventional `<production>test` package avoids collisions with production names. Match “stub,” “fake,” “mock,” and “spy” to actual behavior. If one production type needs a double, prefer `creditcardtest.Stub`; data-only types need not acquire doubles. Different double behaviors can use names such as `AlwaysCharges`/`AlwaysDeclines`. When several production types have doubles, include the type, for example `StubService`/`StubStoredValue`. In Bazel mark helper libraries `testonly = True`. Local variable names should make relevant distinctions clear; `spyCC` may be clearer than `cc` when the test-double nature matters.

**Shadowing:** With `:=`, an existing same-scope variable can be assigned while another is newly declared; that is fine if the old value is no longer needed. In an inner block, the same spelling instead declares a shadow. If a changed value must survive the block, predeclare additional variables and use assignment. Intentional shadowing can be useful; use a different name when clearer. Avoid standard-library package names for locals except in tiny scopes; package authors should avoid forcing clients to shadow or rename imports.

```go
// Good: the derived context remains in use after the block.
if timeout > 0 {
    var cancel context.CancelFunc
    ctx, cancel = context.WithTimeout(ctx, timeout)
    defer cancel()
}
// Bad for that intent: ctx below is a new, inner variable.
if timeout > 0 {
    ctx, cancel := context.WithTimeout(ctx, timeout)
    defer cancel()
    use(ctx)
}
```

**Utility names:** Prefer focused names over bare `util`, `helper`, or `common`, though these words can be useful parts of meaningful names. Evaluate full call sites: `spannertest.NewDatabaseFromFile`, `io.SeekStart`, `elliptic.Marshal`.

### Package size practices

Group cohesive, tightly coupled implementation types, and types clients must use together. Separate distinct concepts; avoid putting a whole project into one package or splitting it into tiny packages that cannot stand on their own. There is no one-type/one-file rule or fixed package/file size limit. Organize focused files so maintainers can find code, avoiding both giant files and unnecessary tiny files. `doc.go` can hold extensive package docs. Bazel permits multiple targets in a directory, but one package per directory is useful for eventual open sourcing.

### Import practices

Apply the Decisions grouping. For generated messages use a descriptive, preferably whole-word prefix and `pb` suffix (`foopb`, `pushqueueservicepb`) for `go_proto_library`; generated service stubs from `go_grpc_library` use `grpc` (`foogrpc`). In new code avoid vague aliases like `pb`/`xpb`. Older short aliases need not be renamed and are not precedent for new names.

### Error handling practices

Choose deliberately where an error is best produced, enriched, and handled. Usually do not ignore errors, although a group of related operations may need only its first error, as with `errgroup`.

**Error structure:** If callers/tests need to distinguish conditions, supply structured errors, sentinel values, fields, or appropriate codes rather than strings or regexes. Direct sentinel equality works without wrapping; use `errors.Is` for wrapped errors and `errors.As` for typed inspection. gRPC status errors are optional; if used, choose canonical codes meaningfully.

**Added context and wrapping:** Add interesting information absent below, such as what an operation means, not an already-reported filename or an empty “failed:”. Merely being an error already conveys failure.

- `%v` inside `fmt.Errorf` makes a new error from text and discards the original structure. Use it for human-readable annotation when programmatic inspection is not intended, or to deliberately translate/hide internal errors at RPC, IPC, or storage boundaries.
- `%w` preserves the underlying error for `errors.Is`/`errors.As`. Choose it when callers are intended to inspect that error; exposed underlying errors become an API contract to document and test. At external boundaries, translating to a standardized error space such as canonical gRPC codes is often preferable to leaking raw internals.
- Inspecting with `errors.Unwrap` is generally discouraged because it does not handle multi-errors.

Prefer putting an underlying `%w` at the end, so printed contexts read newest to oldest. This is a presentation recommendation, not a change to the error chain. **Exception:** Put a classification sentinel first when it is the primary category; a secondary underlying cause may be formatted with `%v`.

```go
// Good: ordinary underlying-cause context.
return fmt.Errorf("read configuration: %w", err)
// Also good: the category is the important part.
return fmt.Errorf("%w: invalid configuration: %v", ErrInvalid, err)
// Bad: the annotation adds no information.
return fmt.Errorf("failed: %v", err)
```

**Logging:** Provide useful context, protect sensitive information, and avoid returning and logging the same error at successive levels. Let the appropriate caller choose recovery, logging, rate limiting, or termination; its location may be more useful than the original error's location. ERROR logs should be actionable and used sparingly; flushing them can be expensive. Monitoring/alerts may be more reliable than hoping someone reads logs.

**Verbosity:** Establish a small coherent convention, for example V(1) for extra information, V(2) for tracing, V(3) for large state. Guard expensive argument evaluation explicitly: `log.V(2).Infof("%v", expensive())` still calls `expensive()` even if nothing is logged.

```go
// Good: the costly formatting work is conditional.
if log.V(2) {
    log.Infof("Query plan: %v", sql.Explain())
}
```

**Initialization, program checks, and panics:** Propagate bad flags/configuration to main and report an actionable `log.Exit`; a stack trace rarely helps the user fix startup input. Libraries should return errors, especially for transient failures. An unrecoverable violated invariant may justify `log.Fatal`. Panic runs defers, which may deadlock or corrupt state while unwinding; broad recovery of unexpected panics hides bugs. The HTTP server's historical recovery behavior is not general advice to imitate it.

Panic can suit API misuse or core bugs (the standard library lacks Google's logging). In rare tightly coupled internals, panic/recover can implement private control flow, but contain it within the public package boundary, recognize your own panic types, re-panic unrelated values, and manage cleanup/unlocking correctly. `panic("unreachable")` after a nonreturning call can satisfy the compiler. Before flags are parsed, logging is unavailable: initialization/Must failures may panic instead of using fatal logging. Normal error handling remains the Decisions rule.

### Documentation practices

Accurate familiar documentation and runnable examples help prevent misuse. Augment the Commentary decisions with these conventions:

- **Parameters/configuration:** Do not enumerate every obvious parameter, option, or field. Explain nonobvious or error-prone behavior and its consequences at a depth appropriate to the audience.
- **Contexts:** Conventional interruption on cancellation and `ctx.Err()` need not be repeated. Document deviations: a different error, different stop/lifetime behavior, asynchronous cancellation, or required context lineage/values/lifetime. Avoid imposing unusual demands such as prohibiting deadlines; documenting an unavoidable demand is not a recommendation to design one.
- **Concurrency:** Conceptually read-only operations are normally assumed safe for concurrent use; mutating operations are not. Document ambiguous cases such as an LRU lookup that mutates state, synchronization the implementation supplies, and concurrency obligations imposed on a consumer-defined interface. A whole-type guarantee generally belongs on the type.
- **Cleanup:** Document every explicit cleanup callers must perform and how when unclear, such as Stop, cancel, or closing a response body when the call succeeds.
- **Errors:** Document significant sentinels and types callers can check, including whether the returned dynamic type is a pointer or value. Keep function return types as `error`, avoiding concrete-pointer nil-interface bugs. Put widespread package error conventions in package docs.

**Preview and Godoc formatting:** Preview rendered docs during development/review. Separate paragraphs with a blank comment line. Runnable test examples appear in docs. Additional indentation creates verbatim code/list/table blocks; runnable examples are often preferable. In the source's Godoc convention, a capitalized line without punctuation (except parentheses/commas), followed by a paragraph, becomes a heading. Use formatting that renders as intended rather than decorative source markup.

**Signal boosting:** Comment unusual conditions resembling a familiar idiom when readers might miss the difference, e.g. `if err == nil { // if NO error`. This does not forbid success-path checks.

### Variable declaration practices

- Prefer `:=` for a new local initialized to a nonzero value.
- A zero-value declaration communicates an empty, ready-to-use value: `var p Point`, rather than repeating all zero fields. For a zero-valued pointer target, both `new(T)` and `&T{}` are fine; `new` may signal that nonzero initialization would need a constructor.
- A mutex or other noncopyable field can be a value field, but then pass the enclosing struct by pointer and use pointer receivers. A composite local value is fine until code repeatedly takes its address or returns it; in those cases consider declaring the pointer initially. Use pointers for protocol buffer messages because `*pb.T` implements `proto.Message`.
- A nil map can be read, but must be initialized before writes. Use a composite literal when initial elements are known. Avoid noisy explicit empty values when a zero declaration says enough.
- **Size hints:** Base preallocation in performance-sensitive code on empirical analysis. A known final size is a reasonable optional hint, not a readability requirement. Over-allocation can hurt memory and performance; default to zero values/literals absent a reason.
- **Channel direction:** Specify `<-chan` or `chan<-` when possible to communicate ownership and prevent inappropriate sends/closes at compile time.

These are preferences; the Decisions non-decisions still permit equivalent zero-initialization and empty-construction forms.

### Function argument practices

Avoid signatures with many confusing positional arguments. Separate use-specific APIs when practical. For necessary options choose a structure or variadic options according to real call sites, especially exported APIs, balancing clarity against implementation machinery.

**Option structures:** Usually put an options struct last; export it only when the API requires exported use. Named fields document call sites; omitted fields take defaults, options can be shared, fields can grow compatibly, and fields can carry docs. Never put context into the options. This approach is often best when callers commonly supply options, many options are in use, or functions share an options object.

**Variadic options:** Functions can return option closures accepting a mutable configuration, usually an unexported struct pointer. This costs substantial implementation code and is justified only when benefits exceed that cost. It can suit mostly default calls, rarely used or numerous options, multiargument/validated options, extensive option docs, and deliberately allowed external options. A named option type groups documentation; options are values callers can share/accumulate.

Accept option values, rather than use the option's mere presence: `FailFast(enable bool)` and `Format(formatEnum)` support dynamic composition better than `EnableFailFast()` or one function per enum member. Process options in order; the last conflicting/noncumulative option wins, while cumulative options may append. An unexported options-closure parameter is a good default to restrict option definitions to the package; enable external extension only deliberately.

### Complex CLI practices

For more complex command trees, prefer [subcommands](https://pkg.go.dev/github.com/google/subcommands) when its features suffice: it is simple and uses Go flags. Cobra provides getopt-style flags and more features and is common externally. With Cobra use `cmd.Context()`, not a new Background context; subcommands supplies context as an argument. Do not mechanically require one package per subcommand. A reusable library separate from the command wrapper can help.

### Testing practices

**Leave validation in Test:** Setup/cleanup helpers manage environment; assertion helpers validate the subject and obscure failure context. Keep checks and failure reporting in the Test function. Duplicate simple logic, use a suitable table, or have reusable validation return errors/values/`cmp.Option`s that Test can report. Test helpers remain real code and should not terminate unless continuing is pointless.

**Extensible validation and acceptance tests:** A package implementing a protocol may offer black-box validation for user-supplied implementations. This inversion of control is a special need, not a general exemption for assertion libraries. Prefer a `<pkg>test` API accepting an implementation and returning nil/error identifying broken invariants. For a fast validator, fail-fast returned errors are simplest; use sentinels/structured data when callers need introspection. For slow tests, consider aggregating failures. Continue when useful; fatal termination is for exceptional setup failure or a broken invariant making continuation invalid. Use Helper/Cleanup appropriately; the caller reports returned validation errors.

**Real transports:** Prefer a real HTTP/RPC client talking to a test backend over a hand-implemented client or a new interface wrapping generated code. Use testing facilities supplied by the service author where possible.

**Error versus fatal:** Generally continue to expose more failures. Fatal termination can suit impossible setup, including whole-table setup. Without subtests, an entry that cannot continue can report `t.Error` then `continue`; inside `t.Run`, `t.Fatal` stops just that subtest. Stop only when following checks would be invalid.

**Helper errors:** Setup/cleanup helpers may report failure directly with useful context, call `t.Helper`, and register cleanup via `t.Cleanup`. If a helper cannot fail, it needs neither a testing parameter nor `t.Helper`. A helper's setup failure must not be confused with an assertion about the system under test.

**Goroutines:** As required by `testing`, `FailNow`/`Fatal` **must only run in the goroutine executing Test or its subtest**. Worker goroutines can use `t.Error` and return; they must not call Fatal. `t.Parallel()` does not invalidate Fatal in the test's own goroutine. Setup helpers can Fatal when called there; unnecessary passing of `*testing.T` can obscure ownership.

```go
// Good: a worker reports failure and ends its own work.
go func() {
    if err := work(); err != nil {
        t.Errorf("work() failed: %v", err)
        return
    }
}()
// The test must still synchronize with the worker before returning.
```

**Struct fields:** Named fields make table cases readable, especially tall tables, adjacent fields of equal type, and omitted zeros.

**Scoped setup:** Explicitly initialize expensive resources in the tests that need them. Avoid `init`/package-global setup that runs even for unrelated filtered tests. Keep tests hermetic, including resetting changed globals.

**TestMain:** Not the first choice. It can be appropriate when *all* tests need shared setup **and teardown**, often expensive functional tests. Prefer ordinary helpers or other simpler arrangements first; put unrelated tests elsewhere. To use defers, let a `runMain` function return the exit code, then call `os.Exit` outside it; `os.Exit` skips defers.

**Amortized setup:** Optional `sync.Once` setup is suitable only if initialization is expensive, only some tests need it, and it needs **no teardown**. Store the result/error and have the helper report useful failure. Once has no uniform cleanup owner and cannot easily cancel waiting initialization when the waiting caller's context is canceled.

### String construction practices

For a few strings, prefer `+`. For formatting, prefer `fmt.Sprintf`; if writing to an `io.Writer`, use `fmt.Fprintf` directly instead of constructing a temporary string. For more complex rendering consider `text/template` or Google's safe HTML template facilities. Use `strings.Builder` for sequential piecemeal construction: amortized linear work avoids potentially quadratic repeated `+`/Sprintf growth. Use backtick raw literals for constant multiline strings.

### Global state practices

Prefer independent instances and explicit dependencies passed through constructors, parameters, or fields. Global behavior couples unrelated clients/tests, restricts parallelism/filtering/sharding, creates call-order and initialization problems, complicates errors/concurrency, and can cause duplicate registrations. Scrutinize it especially in infrastructure libraries; historical or standard-library global APIs are not precedent.

Common forms include exported/private mutable top-level variables, a global service locator (an instance locator need not be global), registries/callbacks, and backend/client/resource singletons.

**Litmus tests:** Be concerned when independent functions/tests interact through package state, users replace it for testing, or the API needs a special call order. Limited safer cases, after avoiding those traps, include logically constant state, observationally stateless caches, state with no externally visible leakage, or situations without an expectation of predictability. Shared sidecars can span processes or external systems; apply the same stateless/idempotent/local considerations. Pure stateless decoder registration like `image.RegisterFormat` does not justify stateful resource registration.

**Default instances:** A convenience facade over a default instance can be acceptable, though not recommended, if **all** these conditions hold:

1. Clients can create independent isolated instances.
2. Global functions are thin proxies to the default instance.
3. Only binaries use the global API, apart from migration; importing infrastructure libraries do not depend on package-global state.
4. Document/enforce lifecycle and concurrency invariants and provide a way to reset to a known-good default state.

### Interface practices

**Avoid unnecessary interfaces:** A concept named service/repository does not itself require an interface. Use one for an actual substitution need, multiple concrete implementations, narrowing a huge API, or breaking a cycle while examining package structure. Reuse suitable existing/generated interfaces. Avoid manual RPC wrappers for tests; use real transports. Avoid exported testing back doors without a material substitution need.

**Ownership/visibility:** Generally keep internal interfaces private and let the consumer define the methods it actually calls. A producer can own an interface when a shared protocol is the product (`io.Writer`, hash interfaces, or generated protobuf contracts), when that prevents widespread perfectly mirrored definitions, or to break a dependency cycle. A neutral, implementation-free protocol package can avoid heavy imports/cycles, but a little duplication is sometimes cheaper than a dependency.

**Effective design:** Keep interfaces small and composable. Document contracts, edges, errors, and concurrency requirements. A single-method interface is usually sufficiently documented on the type; each method of a multimethod interface requires its own documentation. Consider private-interface docs as well.

Usually accept interfaces and return concrete types. Concrete results preserve the full API. Returning an interface may be appropriate for meaningful encapsulation or provider flexibility; merely hiding a concrete name is insufficient because exported methods remain discoverable by type assertion. If a function is designed to return different concrete types chosen at runtime, it must return an interface, as in some command, chaining, factory, and strategy designs. These patterns are guidelines, not rules: avoid forcing an interface if one robust concrete type can handle the abstraction internally, as `sql.DB` does. If returning a concrete type would import a package that already imports the current package, the return type must be an interface to break the circular dependency. Also examine whether the cycle reflects overly fragmented packages that should be combined; do not add interfaces automatically without considering the design.

## Sources, attribution, and licensing

This file is an **editorially condensed adaptation**, not a complete or verbatim reproduction. It combines the following fixed sources and shortens/reorganizes explanatory prose and examples while retaining actionable rules, recommendation levels, and important exceptions. Examples may be shortened or adapted. Original material is by **Google and the contributors/authors named in the source documents**, from Google Style Guides at commit `572c673d9058242c049fbc8248b17029726c7afa`.

| Source | Fixed upstream source | Official published document |
| --- | --- | --- |
| `go/index.md` | [fixed overview](https://github.com/google/styleguide/blob/572c673d9058242c049fbc8248b17029726c7afa/go/index.md) | [Go Style overview](https://google.github.io/styleguide/go/) |
| `go/guide.md` | [fixed guide](https://github.com/google/styleguide/blob/572c673d9058242c049fbc8248b17029726c7afa/go/guide.md) | [Style Guide](https://google.github.io/styleguide/go/guide) |
| `go/decisions.md` | [fixed decisions](https://github.com/google/styleguide/blob/572c673d9058242c049fbc8248b17029726c7afa/go/decisions.md) | [Style Decisions](https://google.github.io/styleguide/go/decisions) |
| `go/best-practices.md` | [fixed best practices](https://github.com/google/styleguide/blob/572c673d9058242c049fbc8248b17029726c7afa/go/best-practices.md) | [Best Practices](https://google.github.io/styleguide/go/best-practices) |

The upstream repository README licenses documentation under [CC BY 3.0](https://creativecommons.org/licenses/by/3.0/) and code samples under [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0). Preserve any more specific original notices. This is an unofficial adaptation; **Google does not endorse it**. Optional source consultation does not change the guidance hierarchy or require an agent to navigate elsewhere before using this file.
