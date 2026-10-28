# Google Verilog Style Guide

This is an **editorially condensed adaptation** of Google's Verilog/SystemVerilog guide, covering synthesizable RTL and testbench/DV code. Repeated explanations and examples have been shortened; requirements, recommendations, permissions, and important exceptions are retained. See [Sources and licensing](#sources-and-licensing) for attribution, the fixed source, and licenses. This is unofficial and is not endorsed by Google.

## Basics

### Summary

Google uses Verilog for logic design. This guide promotes consistent, readable, reusable SystemVerilog across projects and applies to both synthesizable and testbench code.

### Terminology Conventions

- **Must**, **do not**, imperatives, and declarative rules express mandatory requirements or prohibitions.
- **Recommended** expresses the preferred choice; **not recommended** means unsuitable but permitted when the implications and reasons are understood.
- **May** expresses permission; **can** expresses possibility.

### Style Linter

[Verible's Verilog Style Linter](https://github.com/chipsalliance/verible/tree/master/verible/verilog/tools/lint) can check mandatory requirements.

### Style Guide Exceptions

Deviations required for a working design or tool workaround are allowed. Justify every exception with a brief comment and an appropriate linter waiver pragma where needed.

### Inclusive Language

Use inclusive language in identifiers and comments; avoid disrespectful or offensive terms, including “master/slave,” “blacklist/whitelist,” and “redline,” even when ostensibly neutral. Use gender-neutral language except when referring to a particular person with their pronouns: “they/them/their” for unspecified people and “it/its” for nonpeople. See also [Bus Protocols](#bus-protocols).

#### Non-Inclusive Language in External IP

Minimize propagation of non-inclusive industry-standard or third-party names while documenting mappings clearly:

- Wrap third-party modules whose module/port names contain such terms. Use preferred terminology, document the mapping in the wrapper, and instantiate the wrapper everywhere. Do not edit the third-party code directly.
- For standard interface signals, use preferred terms in code and document the original-to-preferred mapping beside the declarations.

### Which Verilog to Use

Prefer SystemVerilog-2017. All RTL and tests must use SystemVerilog following the [SystemVerilog-2017 standard](https://ieeexplore.ieee.org/document/8299595).

### What about VHDL?

Verilog is Google's established choice for logic design.

### Reusable IP

Reusable IP means common RTL intended for use across multiple projects.

## File Conventions

### Summary

Use the following names, structure, and reproducibility rules.

### File Extensions

Use `.sv` for SystemVerilog modules/packages and `.svh` for SystemVerilog global macro/include files. Legacy Verilog-2001 uses `.v` and `.vh` respectively.

### File Names

A filename must begin with its design element's name and indicate the dialect: module `interconnect` goes in `interconnect.sv`. Declare only one design element per file (module, package, program, interface, checker, primitive, or configuration). A variant may insert dot-delimited qualifiers before the extension, for example `interconnect.gate.sv`.

### Standard Design Files

Each design must have `[design]_top.sv` (top-level module) and `[design]_pkg.sv` (package named `[design]_pkg` containing related declarations). `[design]` is the unique design name. A design is a project top, cluster of blocks, or design block. A design block is a fundamental architectural/microarchitectural unit whose black-box behavior and every port's interface and behavior are fully documented.

### File Structure

Order each file as header comment, declaration, then implementation.

#### Header Comment

Every file must begin with a comment briefly describing the design element and the problem it solves. Explain non-obvious aspects of its theory of operation.

### Basic Template

```verilog
// Describe intent and non-obvious operation.
module my_module #(
    parameter int Width = 80,
    parameter int Height = 24
) (
    input logic clk,
    input logic rst,
    ...
);
  ...
endmodule
```

### Generated Code

Supply all inputs and scripts needed to reproduce generated code automatically, including code produced by preprocessors or other languages. In-place generators must retain generating macros/recipes as comments, be idempotent and repeatable, and not strip the recipes. Lines `/**` and `**/` delimit comments reserved for generators. Generated output must follow this guide.

## Aesthetics and Formatting

### Summary

Use a 100-character limit, two-space indentation, spaces instead of tabs, whitespace around keywords/binary operators, and preferably `//` comments.

### General File Appearance

#### Characters

Use ASCII and UNIX `\n` line endings.

#### POSIX File Endings

Every line of a nonempty file, including the final line, must end with a newline.

#### Line Length

Wrap code at 100 characters. Exceed this only where wrapping is impossible, such as a long include path. See [Line Wrapping](#line-wrapping).

#### No Tabs

Do not use tabs anywhere; indent and align with spaces.

#### No Trailing Spaces

Remove trailing whitespace.

#### No Redundant Semicolons

Remove redundant semicolons; consecutive `;;` usually indicate a typo (one semicolon suffices for a null statement/item).

### Eschatology

Use `begin`/`end` unless the whole semicolon-terminated statement fits on one line. Put `begin` on the preceding keyword's line, ending that line; start `end` on a new line. Write `end else begin` together, except a labeled `end` requires the following `else` on a new line. Single-line statements may either include or omit `begin`/`end`.

Treat every macro as potentially expanding to multiple statements: enclose macro calls in `begin`/`end` where necessary, particularly conditional and loop bodies.

```verilog
// Correct: wrapped statements and macro expansions are enclosed.
if (condition) begin
  `MACRO(...)
end else begin
  foo = bar;
end
if (condition) foo = bar;  // Also permitted: the entire statement fits.
else foo = bum;

// Labeled exception:
if (condition) begin : a
  foo = bar;
end : a
else begin : b
  foo = bum;
end : b

// Incorrect: wrapped statement lacks begin/end.
always @(posedge clk)
  q <= d;
// Incorrect: a multi-statement macro is not safely controlled.
if (condition) `MACRO(...)
```

Apply this to case items too: omit `begin`/`end` only when the expression and associated statement fit on one line; otherwise put `begin` on the case expression's line.

```verilog
unique case (state)
  kIdle: state_next = kA;
  kA: begin
    state_next = kIdle;
    foo = bar;
  end
  default: state_next = kIdle;
endcase
```

### Indentation

Indent two spaces per level, without tabs.

#### Indented Sections

Indent enclosed sections of all paired keywords (`begin/end`, `module/endmodule`, `package/endpackage`, `class/endclass`, `function/endfunction`, etc.) one additional level. Indent struct, union, and enum members two spaces.

#### Line Wrapping

Indent continuations four spaces, or align them with an opening parenthesis/brace when clearer. Nested struct expressions may align with the first member name. Otherwise minimize the number of lines. Wrapped operators may begin or end a line, consistently throughout the file.

```verilog
assign zulu = enabled && (
    alpha < bravo &&
    charlie < delta);
assign addr = addr_gen(thing, other_thing,
                      long_parameter_name);
assign structure = '{src: src,
                     dest: dest,
                     default: '0};
```

#### Preprocessor Directives

Branching directives (`` `ifdef ``, `` `ifndef ``, `` `else ``, `` `elsif ``, `` `endif ``) must be left-aligned even when nested. Indent their text as if the directives were absent. Nonbranching directives follow normal code indentation.

```verilog
package foo_pkg;
`ifdef FOO
  `include "foo.svh"
  parameter int A = 1;
`ifdef BAR
  parameter int B = 2;
`else
  parameter int B = 3;
`endif  // BAR
`endif  // FOO
endpackage : foo_pkg
```

### Spacing

#### Space Around Keywords

Whitespace must delimit keywords, except before a keyword immediately after a group opening, before a keyword at line start, or after one at line end: `if (foo) begin`, `always @(posedge clk) begin`.

#### Space Around Binary Operators

Put whitespace on both sides of binary operators. Compact simple vector declarations, indices, and slices are allowed: `[Width-1:0]`; spaced forms are also allowed. Space-delimit operators in complex indices: `[Factor1 - Factor0 - 1:0]`.

For multiline conditional expressions, use a layout resembling if/else:

```verilog
assign a = ((addr & mask) == MyAddress)
           ? matches_value
           : doesnt_match_value;
```

#### Comma-delimited Lists

Separate a comma from the next character by one space for multiple items on a line; extra alignment spaces are allowed: `{addr, parity, data}`, `myfunc(a, b)`.

#### Tabular Alignment

Align analogous parts of related lines with spaces where reasonable; this is recommended.

#### Array Dimensions in Declarations

Space around packed dimensions; do not space between an identifier and its unpacked dimensions or between adjacent dimensions. This includes dynamic/associative arrays and queues.

```verilog
logic [7:0][3:0] data[128][2];
typedef logic [31:0] word_t;
bit bit_array[512];
data_t some_array[];
data_t some_map[addr_t];
data_t some_q[$];
```

#### Parameterized Types

Use one space before a type's parameters unless part of a qualified name containing `::`, where there must be no space. Apply the comma-spacing rule.

```verilog
my_fifo #(.Width(4), .Depth(2)) my_fifo_2by4 (.in, .out);
class foo extends bar #(32, 8);
foo_h = my_class#(.X(1), .Y(0))::type_id::create("foo_h");
my_pkg::x_class#(8, 1) bar;
```

#### Labels

Block labels have a space on both sides of `:`; statement labels have none before and one after it.

```verilog
begin : foo
  my_statement_label: assert(ptr > 0);
end : foo
```

#### Case items

No space before a case item's colon; at least one after. `default` must have a colon: `default: begin`.

#### Module Header Spacing

Space after the module name and between parameter/port lists: `module foo (` or `module foo #(` followed by `) (`.

#### Module Instantiation Spacing

Space after the instance name and between parameter/port lists: `my_module #(...) my_module (...);`.

#### Function And Task Calls

No space between a function/task name and `(`: `process_packet(pkt);`.

#### Macro Calls

No space between a macro name and `(`: `` `uvm_error(ID, "failure") ``.

#### Line Continuation

Right-aligning macro continuation backslashes is recommended where practical. Put them at least one space beyond the macro's rightmost text (when that space does not split a token), within the line limit.

### Parentheses

Use parentheses whenever understanding operator precedence would require thought or consulting a precedence table.

#### Reduction Operator

Parenthesize a reduction immediately preceded by another operator: `b & (&a)`, not `b & &a`.

#### Ternary Expressions

Parenthesize nested ternaries:

```verilog
assign state_next = condition_b
    ? (condition_a ? a_and_b : b_and_not_a)
    : state;
```

### Comments

Strongly prefer `//` for single/multiline comments. Reserve `/**` for generators. A standalone comment explains the following block; an inline comment explains its own line. Multiline inline comments must make their referent clear.

#### Punctuation, Spelling, and Grammar

Use correct, consistent punctuation, spelling, grammar, capitalization, and readable narrative text. Complete sentences are often clearer; short inline comments may be less formal if consistent.

#### Dead code

Do not keep commented-out or unreachable code; use version control for history. Clearly document intent when dead code serves a special purpose.

## Naming Conventions

### Naming Overview

Use `lower_snake_case` for identifiers except as follows:

| Construct | Style |
| --- | --- |
| Macros | `ALL_CAPS` (with local/global qualifications below) |
| Tunable module/class/interface parameters | `UpperCamelCase` |
| Constants | `UpperCamelCase` |
| Type parameters; struct/union types | `lower_snake_case_t` |
| Constraints | `lower_snake_case_c` |
| Interface declarations | `lower_snake_case_if` |
| Enumeration types | `lower_snake_case_t` or `_e` |
| Enumeration values | `kUpperCamelCase` |
| Chip primary IOs | `UPPER_SNAKE_CASE` or `lower_snake_case` |

All other declarations, instances, nets, ports, genvars, labels, functions, tasks, and variables use `lower_snake_case`. External library identifiers and user overrides may retain the library's spelling, such as UVM `convert2string()`. `UpperCamelCase` names may separate numeric values with underscores, such as `IndexedThing_5_12`.

### Use Descriptive Names

Names must describe purpose and be clear to people outside the team. Minimize unfamiliar abbreviations, especially acronyms/initialisms; do not abbreviate by deleting internal letters. An abbreviation listed in Wikipedia is usually acceptable; maintain a project glossary for additional abbreviations. Recommended descriptiveness increases with scope (`n` can suit a five-line function but not a large module). Consider improving a name before adding an explanatory declaration comment. Do not reuse variables for convenience or overwrite function inputs; create new descriptively named variables.

### Prefer Positive Enablement

Name signals, ports, and parameters positively. Boolean signals are active-high unless suffixed `_n`; active-low is not recommended. Prefer boolean parameters `Foo` or `EnableFoo` over `DisableFoo`, regardless of defaults.

### Hierarchical consistency

Directly connected signals and ports must retain the same name across hierarchy. Exceptions include array-element connections and mapping generic ports to more specific design signals. Make such mappings unambiguous and consistent. Change a name when meaning changes, for example `write_byte_address` versus `write_word_address`.

### Clock and Reset

Use `clk` for clocks, `rst` for active-high synchronous reset, and `arst` for active-high asynchronous reset. Optional prefixes identify more specific use; clock-domain prefixes are required as below. Asynchronous reset deassertion belongs to a specific clock domain.

### Avoid signal names that end with underscore-number

Signals must never end with underscore plus number, such as `foo_1`; synthesis tools use this form when mapping buses into nets.

### Avoid reserved keywords as signal names

Never use Verilog/SystemVerilog reserved keywords as identifiers. When interoperating with other languages, also be mindful of their keywords.

### Prefixes

Use common prefixes for cooperating signal groups, such as `foo_valid`, `foo_ready`, `foo_data`. Multiclock modules must identify clock domains with prefixes.

#### Clock Domain Prefixes

Prefer single-clock modules. In a multiclock module, every port and internal signal must have a mnemonic prefix uniquely identifying its domain, matching the associated clock prefix. This includes asynchronous resets, which must deassert synchronously with their domain: `foo_clk`, `foo_arst`, `foo_payload_valid`; `bar_clk`, `bar_arst`, `bar_payload_valid`.

#### Source-Destination Prefixes

Optional connectivity prefixes take the form `<src>_<dest>__`. Endpoints must identify specific instances, or reasonable generic names if unknown. `<src>` is always the producer/requester and `<dest>` the consumer/completer of the communication, regardless of individual signal directions. A communication can be one flow-controlled channel or a protocol group; document its scope explicitly to prevent ambiguity.

Use these prefixes only at hierarchy levels where they clarify intent. When stripping them at a boundary, preserve the part after `__` unchanged. For a requester `foo` and completer `bar`, request-ready and even reverse-direction response signals in that same protocol group all start `foo_bar__`, never `bar_foo__` based on signal direction.

#### Pipeline Stage Prefixes

Clearly label pipeline variables by stage, such as `p1_x`, `p2_product`, `p3_accumulator`. A pipeline groups related computations/signals by stage; a standalone delayed copy instead uses the delay suffix below, even if a pipeline valid signal is also delayed.

### Suffixes

Use standard suffixes only with their standard meanings. In particular `_ready/_valid/_valid_next` imply Ready/Valid semantics; `_req/_ack` imply Request/Acknowledge semantics.

#### Specify units

Include units in signal names. Counters must identify what they count; addresses must identify indexed word size: `cycle_counter`, `packet_size_beats`, `write_byte_address`. Unitless signals and standardized interface names (such as AXI `waddr`) are exceptions.

Prefer zero-based logic vectors with units in their names. If a nonzero-based slice is necessary, encode its range: `logic [63:0] foo_byte_address`, `logic [61:0] foo_4byte_address`, or `logic [63:2] foo_63_2_byte_address`.

#### Units for Constants

Include constant units, preferably as suffixes: `ImageWidthPixels`, `SystemClockHz`, `InterfaceWidthBytes`. Inherently unitless constants and default units of bits are exceptions (`Mega`, `InterfaceWidth`).

#### Active-Low Signals

Avoid active-low signals where possible. If used, `_n` is required; other signals are assumed active-high.

#### Differential Pairs

Use `_p` and `_n` for positive/negative members of a differential pair, such as `in_p`, `in_n`.

#### Valid Qualifier Suffix

`foo_valid` qualifies the related `foo_*` signals; all qualified signals must share the prefix and must be ignored while valid is unasserted. `_valid_next` announces validity next cycle, distinct from a register's `_next`, and follows the same rules. A companion `_ready` invokes Ready/Valid flow control.

#### Register Signal Suffixes

Use `_next` for a register's D input (also permitted across module interfaces), `_le` for its optional load enable, `_inc` for a counter increment enable, and `_clear` for a register-specific synchronous clear. The unsuffixed name denotes Q.

```verilog
always @(posedge clk) begin
  if (foo_le) begin
    foo <= foo_next;
  end
end
```

#### Delayed Signal Suffix

An otherwise unchanged signal delayed one cycle must use `_d`, then `_d2`, `_d3`, etc. for successive delays (`data_valid_d2 <= data_valid_d;`). Use stage prefixes for pipelines, as distinguished above.

#### Fixed-Point Signals

Optional fixed-point suffixes must be `_sMpN` (signed) or `_uMpN` (unsigned), with M integer bits including the sign bit and N fractional bits. Useful when multiple formats coexist:

```verilog
logic signed [14:0] foo_s3p12;
logic [12:0] bar_u4p9;
logic signed [17:0] foo_bar_s6p12;
assign foo_bar_s6p12 = foo_s3p12 + signed'({1'b0, bar_u4p9, 3'd0});
```

#### Order of Suffixes

Append suffixes from inner meaning to outer time qualification, in this order:

1. Units/semantic qualifiers: `_bytes`, `_valid`, `_ready`, `_req`, `_ack`.
2. Fixed-point format: `_sMpN`, `_uMpN`.
3. Active-low polarity: `_n`.
4. Differential polarity: `_p`, `_n`.
5. Stage/delay/register qualifiers: `_next`, `_le`, `_inc`, `_clear`, `_d`, `_d<N>`.

Correct: `foo_valid_d`, `packet_len_bytes_next`, `coeff_s3p12_next`, `rst_n_d`, `sync_n_p`, `sync_n_n`, `dma_req_n_d`. Incorrect: `in_d_valid`, `packet_len_next_bytes`, `coeff_next_s3p12`, `sync_p_n`.

## Preprocessor Directives and Pragmas

### Global Macro Files

Global defines are permitted only in global macro files, which contain only global defines. They are project-wide and distinct from include files; do not `` `include `` them.

### Macro Definitions

Macro names must contain only uppercase letters, digits, and underscores. Prefix global defines with a related-group namespace and double underscore, such as `DMA_FOO__BAR`. Natural UVM extensions/abbreviations may follow UVM's `lower_snake_case` convention instead.

Local macros must start with a single underscore, be confined to one file, and be explicitly undefined after use. Do not include other files between their definition and `undef`.

```verilog
`define _MAKE_THING(_x) \
    thing thing_##_x (.clk(clk), .i(i##_x), .o(o##_x));
`_MAKE_THING(a)
`_MAKE_THING(b)
`undef _MAKE_THING
```

### Standard Macros

Reuse standard macro libraries whenever possible rather than inventing alternatives. Reference implementations follow.

#### STRINGIFY Macro

Use `STRINGIFY` to convert a macro argument to a string; token-pasting alternatives inside ordinary strings do not work with all tools.

```verilog
`define STRINGIFY(_x) `"_x`"
`define DISPLAY_VALUE(_x) $display("%s: %p", `STRINGIFY(_x), (_x))
```

#### BOUNDED\_LOOKUP Macro

For array rvalue indexing with a potentially oversized index, `BOUNDED_LOOKUP` discards unnecessary high bits (aliasing the array) and returns zero for other out-of-range indices, avoiding an ordinary lookup's out-of-bounds X.

```verilog
`define BOUNDED_LOOKUP(__arr, __idx) \
    (($size(__arr) == 1'b1) ? __arr[$high(__arr)] \
    : (__idx[(($high(__arr) == 0) ? 1 : $clog2($high(__arr) + 1)) - 1 : 0] < $low(__arr) || \
       __idx[(($high(__arr) == 0) ? 1 : $clog2($high(__arr) + 1)) - 1 : 0] > $high(__arr)) ? '0 \
    : __arr[__idx[(($high(__arr) == 0) ? 1 : $clog2($high(__arr) + 1)) - 1 : 0]])
```

#### X\_OF\_TYPE Macro

Use this standard typed-X cast:

```verilog
`define X_OF_TYPE(__type) \
  (__type'({$bits(__type){1'bx}}))
```

#### Assertion Macros

Use assertion libraries for a unified design-wide implementation. These reference macros check a clocked property outside reset, known-valued expressions, and elaboration-time/static conditions (especially parameters):

```verilog
`define ASSERT(__name, __prop, __clk, __rst) \
`ifdef ASSERT_ON \
  __name: assert property ( \
      @(posedge __clk) disable iff (__rst !== 0) (__prop)); \
`endif

`define ASSERT_KNOWN(__name, __expr, __clk, __rst) \
  `ASSERT(__name, ~$isunknown(__expr), __clk, __rst)

`define ASSERT_STATIC(__name, __expr) \
  `ifdef ASSERT_ON \
    if (!(__expr)) \
      $fatal(2, "Failed static assertion: %s: (%s) is not true.", \
        `"__name`", `"__expr`"); \
  `endif  // ASSERT_ON
```

### Synthesis Pragmas

Do not use pragmas that can cause simulation/synthesis differences: `translate_off/on`, `synthesis_off/on`, `read_comments_as_hdl on/off`, `parallel_case`/`parallel case`, or `full_case`/`full case`.

### `` `ifdef `` Directives

Avoid conditional compilation whenever possible; separate source units and generate-if usually suffice. Do not use `ifdef`, `ifndef`, `else`, or `elsif` except as prescribed below.

#### `` `ifdef SIMULATION ``

Permitted only additively for debug logic/logging. `SIMULATION` is typically defined for simulation and never elsewhere. Do not use it to select RTL versus behavioral models or wrap simple assertions (use `ASSERT_ON`).

#### `` `ifdef ASSERT_ON ``

Permitted only additively for assertions and assertion-specific modeling, allowing assertions to be disabled for simulation speed.

#### `` `ifdef FPV_ON ``

Permitted only additively for monitors/logic exclusively used by formal property verification tools.

#### `` `ifdef <simulator> ``

Guard simulator-specific debug features; never enclose functional RTL. Legal simulator identifiers are `VCS` (Synopsys), `INCA` (Cadence Incisive), `XCELIUM` (Cadence Xcelium), and `DSIM` (Metrics DSim).

#### `` `ifdef <technology> ``

Technology-specific implementation selection is allowed (libraries, emulation, processes, FPGA architectures). Always provide a valid default `else` implementation and unit tests exercising every technology case. Standard names include `EMULATION` and `FPGA`.

#### `` `ifdef `` Anti-patterns

Do not enable features with `ifdef`; use parameters and generate-if. Do not disable dead code with constructs such as `ifdef DISABLED`; delete it. Do not use include guards; use an ordered compilation file list.

#### `` `ifdef `` Exceptions

For necessary nonstandard conditionals:

- With no defines set, the code must represent the canonical deployed design.
- Document and justify the macro in project documentation; no ad hoc macros.
- Use a unique project namespace (`FOOBAR__DEBUG`, not `DEBUG`).
- Choose positive enablement names (`FOOBAR__PERF_COUNTERS`, not `FOOBAR__DISABLE_PERF_COUNTERS`).

### Includes

#### Include Files

Files intended for `include` are “include files.”

#### Include Directives

Use `include` only to include files into a package or snippets that cannot be wrapped as a module/package. Each file included in a package must belong to only one package. Do not include source merely to compile each file standalone: use an ordered file list and Multi-File Compilation Unit (MFCU) mode.

```verilog
// Correct: partitioning a package.
package foo_pkg;
  `include "foo_functions.svh"
endpackage
// Correct: a snippet that cannot be wrapped.
module foo (
    `include "generated_signals.svh"
);
endmodule
```

Incorrect: including `foo_pkg.sv` in a dependent module, or including common assertions that could instead be wrapped in a module.

#### Include Guards

Do not use include guards; the permitted include structure already prevents repeated expansion.

### Stopping Compilation

Fail compilation using the undefined `` `DIE() `` macro with an explanatory string. This preserves parsing by tools without preprocessing and requires no other macro files. Do not insert deliberate syntax errors.

```verilog
`ifndef USING_SIMULATOR_A
`ifndef USING_SIMULATOR_B
`DIE("Unknown or unsupported simulator.")
`endif  // USING_SIMULATOR_B
`endif  // USING_SIMULATOR_A
```

## Modules

### Module Header Definition

Use Verilog-2001 combined/ANSI port declarations; fully declare each port's name, type, and direction in the header. Do not use Verilog-95 name lists with later declarations. List clocks first, then all reset inputs.

For parameter/port lists containing declarations, put closing `)` on its own line in column zero. Opening `(` belongs on the module-name line or the preceding list's closing-`)` line. Exceptions:

- Lists without declarations may omit parentheses; if present, both must be on the same line.
- After imports, put the next list's opening parenthesis on its own line.

Each import statement must occupy its own line.

#### Module Header Definition Examples

```verilog
module foo #(
    parameter int Width = 8
) (
    input  logic clk,
    input  logic rst,
    input  logic [Width-1:0] d,
    output logic [Width-1:0] q
);
```

```verilog
module foo
  import foo_pkg::AddrWidth;
#(
    parameter int DataWidth = 8
) (
    input  logic clk,
    input  logic rst,
    input  logic [AddrWidth-1:0] addr,
    input  logic [DataWidth-1:0] d,
    output logic [DataWidth-1:0] q
);
```

#### Module Names

Module names use lowercase alphanumerics and underscores (`lower_snake_case`), including proper nouns. A design's pin-level top must be `[design]_top`; unit-test modules must end `_test`.

#### Parameterized Modules

#### When to parameterize

Use parameters to customize instances, specialize widths, or document design-wide parameters allowed to change within a module. Do not transmit constants through hierarchy with parameters (use package constants) or use them for internal constants (use `localparam`).

#### Naming module parameters

`UpperCamelCase` is recommended; historic `ALL_CAPS` may be permitted. Type parameters must use `lower_snake_case_t`.

#### Use ANSI-style parameter port lists

Declare tunable parameters in ANSI parameter port lists; never use `defparam`.

```verilog
module foo #(
    parameter int ParameterName = 1,
    parameter type type_parameter_name_t = logic
) (
    input type_parameter_name_t ib_headward,
    ...
);
```

#### Parameter defaults

Defaults must represent a valid, reasonable design. Omitting a default is permitted to avoid integration bugs (for example, requiring an integrator to set an IP clock frequency). Composite-type parameters must have defaults because tool support is inconsistent otherwise.

#### Specify parameter data types

Explicitly type parameters; use signed `int` for simple numeric quantities. Always assert parameter validity (bounds, etc.), for example:

```verilog
`ASSERT_STATIC(width_must_be_positive_A, Width > 0)
`ASSERT_STATIC(depth_greater_than_1_A, Depth > 1)
```

### Module Instances

#### Module Instance Names

Instance names must equal the module name plus an optional unique string/numeric suffix: `my_decoder`, `my_counter0`, `my_fifo_foo`. If an unwieldy library name makes this illegible, a project-specific pattern that readily identifies the original module is allowed.

#### Use named ports

Fully specify every instance with named ports in declaration order. `.port` shorthand is allowed when signal and port names match. All declared ports must appear: unused outputs as `.output_port()`, unused inputs explicitly tied to ground (`.unused_input_port(8'd0)`). Do not use positional ports or `.*` (see the verification-only bind exception).

#### Instance connection widths

Port connection widths must match correctly. Prefer explicit widths instead of relying on implicit extension/truncation: `.thirty_two_bit_input({16'd0, sixteen_bit_word})`.

#### Parameterized Module Instantiation

Use named parameters, never positional parameters or `defparam`:

```verilog
my_module #(
    .Height(5),
    .Width(10)
) my_module (...);
```

#### Recursive Instantiation

Modules must not instantiate themselves recursively.

### Generate Constructs

Explicitly name every generated branch (each if/case outcome) and every loop body, with a `gen_` prefix. Do not add a surrounding `begin` block or use `generate/endgenerate` regions.

```verilog
if (TypeIsPosedge) begin : gen_posedge_type
  always @(posedge clk) foo <= bar;
end else begin : gen_negedge_type
  always @(negedge clk) foo <= bar;
end
for (genvar bus = 0; bus < NumberOfBuses; bus++) begin : gen_my_buses
  my_bus #(.Index(bus)) my_bus (.foo(foo), .bar(bar[bus]));
end
```

#### Floating begin-end blocks

Bare generate `begin/end` blocks, outside loop/if/case generate constructs, are prohibited and not LRM-compliant. Ordinary sequential blocks remain allowed.

## Packages

### Package Declaration

No additional declaration rules apply beyond this guide's other conventions.

#### Package Names

Use lowercase alphanumerics and underscores, ending `_pkg`.

### Package Dependencies

Packages may use other packages' types/constants but must not have cyclic dependencies, even if permitted by the language.

### Package imports

Import only within a local scope such as a module/package, never at global scope. Prefer explicit imports; wildcard imports are allowed when reduced verbosity reasonably improves readability. Fully qualified references are also valid, including in ports.

```verilog
module foo
  import bar_pkg::AddressWidth;
(
    input bar_pkg::data_t data,
    input logic [AddressWidth-1:0] addr
);
  import foo_pkg::*;
  import bar_pkg::PageSizeBytes;
```

## Interfaces

### Overview

DV code uses interfaces. This guide does not adopt them for synthesizable code: tool support, ECO/global scope, physical-design bit-blasting, and declaration overhead remain challenges; structs provide many benefits. Adoption would require documented best practices addressing these issues.

## Ports, Signals, and Constants

### Overview

Explicitly declare typed signals, use `logic` for RTL and `lower_snake_case`, avoid input defaults, use named constants/types and packed RTL data, and follow packed/unpacked ordering.

### Declare all signals

Explicitly declare every signal and its data type before use; inferred/implicit nets are forbidden. Within modules, declarations of signals, types, enums, and localparams are recommended close to first use.

### Use `logic` for synthesis

Synthesizable signals must use 4-state types ultimately built from `logic`, rather than ambiguous integer primitives or 2-state types. Say `logic signed [31:0] x_velocity`, not RTL `int`/`bit` signals. `wire` is allowed when necessary and as continuous-assignment shorthand. Do not confuse declaration initialization with continuous assignment:

```verilog
wire [7:0] sum = a + b;  // Continuous assignment.
logic [7:0] acc = '0;    // Initialization.
```

Nets connected to `inout` ports, for example, require `wire`; justify exceptions to `logic` with a short comment. DV may use 2-state types, but every 4-state-to-2-state interface must assert that the 4-state net contains no X before conversion.

### Signal Naming

Nets, variables, and ports use lowercase alphanumerics/underscores in `lower_snake_case`. Ports directly connected to chip primary IO pads (die/package IOs or FPGA IO buffers) may use `UPPER_SNAKE_CASE`. Follow the detailed [Naming Conventions](#naming-conventions).

### Default Port Values

Do not give input ports default constant expressions. Constant expressions/initializations assigned to output ports are permitted, such as `output logic q = '0`.

### Constants

Constants differ from tunable parameters. Define design-wide constants in the design package; define local constants with `localparam`.

#### Declaring Constants

Use `parameter` for constants in packages and `localparam` in modules/classes. Prefer packages for collections of constants. Other packages may provide their own constants for reusable IP.

#### Naming Constants

`UpperCamelCase` is recommended for immutable constants; enum constants use `kCamelCase`.

#### Specify constant data types

Explicitly type constants, using signed `int` for simple numeric quantities.

#### Avoid Magic Numbers

Prefer symbolic names for commonly used numerical constants (`StartFrameDelimiter` rather than repeated `8'hD5`). Unnamed constants are acceptable when a name adds no value, such as an obvious counter increment by one.

#### `localparam` in parameter port lists

Normally put module constants, including parameter-derived constants, in the body close to use. If a derived constant controls port dimensions, declare it as `localparam` in the parameter port list so instantiations cannot override it. Do not make it an independently tunable `parameter`.

```verilog
module foo #(
    parameter int NumEntries = 1,
    localparam int IndexWidth = $clog2(NumEntries)
) (
    input logic [IndexWidth-1:0] foo
);
  localparam int Bar = 7;  // Body-only constant, close to use.
```

### Packed vs. Unpacked

Use packed arrays and structs for synthesizable signals. Unpacked arrays are appropriate for behavioral SRAM models and DV.

### Positive Array Bounds

Both bounds of vectors/arrays must be nonnegative: `[14:0]`, never `[13:-1]`.

### Packed Ordering

Packed arrays and bit vectors must be little-endian: left bound >= right bound (`logic [31:0] word`, `logic [1:0][7:0] bytes`).

### Unpacked Ordering

Unpacked arrays must be big-endian: left bound <= right bound. Use `[size]` for zero-based arrays (equivalent to `[0:size-1]`). Whole-unpacked-array assignment is allowed but not recommended because mixed packed/unpacked ordering can reverse data; prefer explicit indexed element assignments. Parameter arrays may initialize as a whole with care because individual assignments need constant functions.

```verilog
logic [15:0] word_array[3];
assign word_array[0] = word0;
assign word_array[1] = word1;
assign word_array[2] = word2;
// Allowed but not recommended for signals; index 0 receives word0:
assign word_array = '{word0, word1, word2};
// Allowed for parameters; Vec[0] == 0, Vec[3] == 3:
parameter int Vec[4] = '{0, 1, 2, 3};
```

This ordering rule applies to variables/parameters, not instance arrays. Order instance arrays correctly for their packed ports.

### Structs and Unions

#### Declaring Structs and Unions

Always typedef structs/unions; anonymous types are forbidden. Struct type names use lowercase alphanumerics/underscores ending `_t` (union type naming follows the naming overview).

```verilog
typedef struct packed {
  logic dir;
  logic [31:0] value;
} data_t;
data_t packet;
```

#### Naming Union Members

Prefix every union member `as_` to make its cast-like interpretation explicit:

```verilog
typedef union packed {
  read_id_t  as_read_id;
  write_id_t as_write_id;
} id_t;
assign write_id = cmd.id.as_write_id;
```

### Enumerations

Always typedef enums and explicitly specify storage type; synthesizable enums must use 4-state storage (`logic`, not `bit`). Anonymous enums are forbidden. Type names use lowercase alphanumerics/underscores and `_t` or `_e`, consistently project-wide. Values use `kCamelCase` (`kReadyToSend`).

```verilog
typedef enum logic {kWrite, kRead} access_t;
access_t req_access, resp_access;
```

### Vector Typedefs

Do not typedef local/single-use vectors; use raw vectors. For broadly reused vectors (such as a design-wide address type), a package typedef is recommended. Exception: a local typedef may be necessary to apply signedness to each packed-array element rather than the entire array:

```verilog
typedef logic signed [DataWidth-1:0] data_t;
data_t [NumChannels-1:0] my_channel_data;  // Signed elements.
// Incorrect for signed elements: signedness applies to the whole array.
logic signed [NumChannels-1:0][DataWidth-1:0] other_channel_data;
```

## Functions and Tasks

### Choosing Between Functions, Tasks, and Modules

Default to modules for synthesizable behavior. Tasks are forbidden in synthesizable RTL but permitted in testbenches. Functions may implement short, simple combinational logic with one output; prefer them when the relationship fits a direct assignment (`assign foo = check_foo(bar);`). Prefer modules for multiple outputs. Functions can be defined in packages for wider reuse.

### Templates

Use parenthesized formal lists, describe behavior/inputs/outputs, declare locals before statements, and follow this structure. Do not assign to input arguments.

```verilog
// Describe behavior, inputs, and output.
function automatic logic foo(logic x, logic y);
  logic z;
  z = x ^ y;
  return z;
endfunction : foo

// Describe behavior, inputs, and outputs.
task automatic bar(input logic x, output logic y);
  y = x;
endtask : bar
```

### Latches inside functions

Do not infer latches in functions; this can cause simulation/synthesis mismatch.

### Lifetime

Outside classes, explicitly declare every function/task `static` or `automatic`, preferring `automatic`. Synthesizable RTL functions must be `automatic`. Class methods are always automatic.

### Function Data Types

Explicitly declare a return type or `void`: `function automatic bit foo(int a);`, never an implicitly typed function.

### Argument Types

Explicitly specify storage type for every argument: `task automatic foo(int a, int b);`, not `foo(int a, b)`.

### Argument Directions

Directions may be omitted when all arguments are inputs. If any direction is specified, specify all. Otherwise inherited directions can surprise: `foo(ref int a, int b)` makes `b` a reference too and violates this rule.

## Sequential Logic

### `always_ff` vs `always`

Either is acceptable for clocked processes; be consistent within a project.

### Use Non-Blocking Assignments

Sequential blocks must use nonblocking `<=` exclusively; never mix assignment types in a block. Combinational process blocks use blocking `=`.

### Sequential Logic (Latches)

Latches are not recommended; use flip-flops/registers unless absolutely necessary. If needed, use `always_latch` rather than `always`, and exclusively nonblocking assignments.

### Sequential Logic (Registers)

Use the standard clocked format and only nonblocking assignments. Put next-state logic requiring blocking assignments into a separate combinational block. Keep sequential blocks simple; preferably just a register, optional load enable or increment. Consider splitting complex logic into `always_comb`.

For reusable IP, supporting synchronous reset, asynchronous reset, and configuration-time initialization is recommended here and required by [Resets](#resets), allowing integrators to tie off unused forms. Initialization is commonly recommended for SRAM-based FPGAs; synchronous reset for ASICs.

```verilog
logic foo_le;
logic [7:0] foo, foo_next;
initial foo = 8'hAB;
always @(posedge clk or posedge arst) begin
  if (arst) begin
    foo <= 8'hAB;
  end else if (rst) begin
    foo <= 8'hAB;
  end else if (foo_le) begin
    foo <= foo_next;
  end
end
```

Do not make multiple nonblocking assignments to the same bit. Even mutually exclusive separate `if`s must become `if`/`else if`. Exception: assigning defaults first, then specific overrides, is allowed, though a separate combinational block with blocking assignments is recommended.

```verilog
// Incorrect even if cond1 and cond2 are mutually exclusive:
if (cond1) abc <= 4'h1;
if (cond2) abc <= 4'h2;
// Correct:
if (cond1) abc <= 4'h1;
else if (cond2) abc <= 4'h2;
```

### Resets

Reusable IP must consistently implement synchronous reset, asynchronous reset, and configuration-time initialization so unused schemes can be disabled during implementation.

### Delay Modeling

Synthesizable modules must use zero-delay simulation methodology. Do not use any `#delay`, including `#0`.

#### Delays and Time Units

In nonsynthesizable code, every nonzero `#delay` must specify units (`#10ns`, `#(wait_in_ns * 1ns)`); `#0` is exempt. Units alone do not guarantee correct behavior: use project-wide `timeunit` and `timeprecision` settings in modules/packages/interfaces.

### Finite State Machines

Implement an FSM as an enum plus two processes: combinational decode for next state/outputs, and clocked update of state.

- List every state in the enum; put state-function comments in the decode case, not the enum. Use `kCamelCase`; name the initial idle state `kIdle` unless a clearer alternative or special circumstances justify otherwise.
- Prefer one FSM per module. With multiple FSMs, uniquely prefix/suffix each machine's states (`kRdIdle`, `kWrIdle`). Prefer names differing near the beginning for waveforms.
- Before the decode case, set defaults for every combinational output, including next state. Next state must default to current state. Code only deviations and transitions in cases; precede each state alternative with its function comment.
- The state register process must do nothing except reset or latch next state.

```verilog
typedef enum logic [1:0] {kIdle, kFrameStart, kWrite, kDone} fsm_state_t;
fsm_state_t state, state_next;
`ASSERT_KNOWN(frame_start_is_known_A, frame_start, clk, rst)
always_comb begin
  state_next = state;
  busy = 1'b0;
  unique case (state)
    // Wait for a frame.
    kIdle: begin
      if (frame_start) begin
        state_next = kFrameStart;
      end
    end
    // Start processing.
    kFrameStart: begin
      busy = 1'b1;
      state_next = kWrite;
    end
    // Write the frame.
    kWrite: begin
      busy = 1'b1;
      state_next = kDone;
    end
    // Finish processing.
    kDone: state_next = kIdle;
    default: begin
      // All four known encodings are covered, so X propagation is safe.
      state_next = `X_OF_TYPE(fsm_state_t);
      busy = 'X;
    end
  endcase
end
always_ff @(posedge clk) begin
  if (rst) begin
    state <= kIdle;
  end else begin
    state <= state_next;
  end
end
```

For non-power-of-two state sets, follow the no-don't-care rules below rather than assigning X for unused known encodings.

## Combinational Logic

### Use `always_comb` or `assign`

Use `always_comb` or `assign`; avoid sensitivity lists.

### Use blocking assignments

Synthesizable combinational process blocks must exclusively use blocking assignments. Nonblocking assignments belong only in sequential logic.

### X Propagation

Either propagate X or catch it with an assertion; never silently resolve an X input to a known output when that masks unknown behavior.

#### Logical operations and `if` statements

Logic propagates X when appropriate: `x == 1`, `x != 1`, `x & 1`, `x | 0`, and `!x` yield X; `x & 0` yields 0 and `x | 1` yields 1. But an X boolean condition is treated as false by `if`, which can hide defects. Use conditional operators for X propagation, or assertions if that is clearer. Explicit branches for 1, 0, and otherwise X are also allowed.

```verilog
assign result = value ? 1'b1 : 1'b0;  // X value propagates.
// Alternative: guard an if with a known-value assertion.
`ASSERT_KNOWN(value_is_known_A, value, clk, rst)
always_comb begin
  if (value) result = 1'b0;
  else result = 1'b1;
end
// Another allowed alternative:
always_comb begin
  if (value) result = 1'b0;
  else if (!value) result = 1'b1;
  else result = 1'bx;
end
```

#### X Propagation in Case Statements

Apply [Case Statements](#case-statements), including the distinction between X propagation and forbidden don't-cares.

#### Array indexing

Writing an array with an X index does not propagate X into its contents. Assert that indices are known:

```verilog
`ASSERT_KNOWN(index_is_known_A, index, clk, rst)
assign buf[index] = foo;
```

#### Clocks and Resets

Do not rely on normal X propagation for clocks/resets: `posedge` can trigger on 0-to-X and recommended reset coding does not propagate X. At every clock-domain hierarchy level, ensure clocks and resets are never X and add `!$isunknown()` assertions. Synthesis tools recommend `if` rather than conditional operators for asynchronous resets.

#### Further discussion

See Sutherland's [I'm Still In Love With My X!](http://sutherland-hdl.com/papers/2013-DVCon_In-love-with-my-X_paper.pdf) and Mills' [Being Assertive With Your X](http://lcdm-eng.com/papers/snug04_assertiveX.pdf).

### Don't-Care Assignments

Do not specify don't-care behavior in synthesizable code: explicit or implicit X assignments from known, non-X inputs. X propagation from unknown inputs is permitted. Don't-cares complicate ECOs and equivalence checking; use external logic-minimization tools if such optimization is needed.

##### Don't-Care in Explicit Assignments

Known inputs must produce known outputs. `assign wdata = valid ? data : 'x;` is forbidden; use `assign wdata = data; assign we = valid;`.

##### Don't-Care in Case Statements

Define known behavior for all known encodings as described below.

##### Don't-Care in Array Indexing

Never access out-of-range array indices, including indices with superfluous MSBs or non-power-of-two bounds. Specify out-of-range behavior or pad to a power of two, and add bounds assertions. For general arbitrary-size handling, the standard `BOUNDED_LOOKUP` macro is recommended.

```verilog
localparam int BufferSize = 5;
logic [BufferSize-1:0] buffer;
logic [2:0] index;
logic foo, bar;
logic [7:0] buffer_padded;
`ASSERT(index_within_array_bounds_A, index < BufferSize, clk, rst)
assign foo = (index < BufferSize) ? buffer[index] : '0;  // May need lint waiver.
assign buffer_padded = {3'b0, buffer};
assign bar = buffer_padded[index];
// General alternative:
assign foo = `BOUNDED_LOOKUP(buffer, index);
```

### Tri-State Logic

Do not use Z/three-state logic for on-chip logic such as muxes.

### Case Statements

Use `unique case` in synthesizable RTL, always include a `default:` item, define all behavior, and never use case-modifying synthesis pragmas. `unique` is recommended for all cases; `priority` is sometimes appropriate, though if/else chains are clearer for priority encoders. Nonsynthesizable code may use plain `case` when needed. `unique0` is not supported by all tools.

Unspecified unique-case items become synthesis don't-cares. A default is required even if all known values are enumerated, both to define behavior and avoid simulation latch inference for X inputs. Usually propagate X via default when all known encodings are covered:

```verilog
always_comb begin
  unique case (select)
    2'b00: foo = bar0;
    2'b01: foo = bar1;
    2'b10: foo = bar2;
    2'b11: foo = bar3;
    default: foo = 'X;
  endcase
end
```

#### Wildcards in case items

Use `case` unless wildcard matching is required, then use `casez` and `?` in wildcard items. Never use `casex`, which can match unknown inputs as wildcards. `casez` treats Z/`?` as wildcards and matches undriven X exactly (though its symmetric wildcard behavior still requires care).

```verilog
unique casez (select)
  3'b000: operand = accum0;
  3'b001: operand = accum0 >> 1;
  3'b010: operand = accum1;
  3'b011: operand = accum1 >> 1;
  3'b1??: operand = regfile[select[1:0]];
  default: operand = 'X;
endcase
```

##### Don't-care in case statements

All outputs must have non-X behavior for every known value of the case expression. If that suppresses X propagation, assert that the expression is known. For an enum using three of four encodings, `default: foo = 'x;` is forbidden because the fourth known encoding becomes don't-care:

```verilog
typedef enum logic [1:0] {kMode0, kMode1, kMode2} mode_t;
mode_t mode;
`ASSERT_KNOWN(mode_is_known_A, mode, clk, rst)
always_comb begin
  unique case (mode)
    kMode0: foo = bar0;
    kMode1: foo = bar1;
    kMode2: foo = bar2;
    default: foo = '0;  // Known behavior for 2'b11.
  endcase
end
```

If all four encodings are explicitly covered, default X is recommended for X propagation. An X assignment to an enum must be typecast; use `X_OF_TYPE` as recommended. Defaults outside the case are convenient for multiple outputs, but the unique case must still be fully specified, possibly with an empty default:

```verilog
`ASSERT_KNOWN(mode_is_known_A, mode, clk, rst)
always_comb begin
  out0 = in0;
  out1 = in1;
  unique case (mode)
    kForceLow:  {out1, out0} = '0;
    kForceHigh: {out1, out0} = '1;
    default: ;
  endcase
end
```

##### Upside-down unique case statements

Upside-down unique cases (`unique case (1'b1)`) can leave multiple-true conditions as synthesis don't-cares. Rewriting as a normal case, or manually reducing logic for timing, is recommended; forbidden don't-care behavior must still be eliminated.

```verilog
// Incorrect: simultaneous grants make out don't-care.
unique case (1'b1)
  c_grant[0]: out = 1'b0;
  c_grant[1]: out = 1'b1;
  default: out = 1'b0;
endcase
// Correct: behavior for simultaneous grants is explicit.
unique case (c_grant)
  2'b01: out = 1'b0;
  2'b10: out = 1'b1;
  default: out = 1'b0;
endcase
```

##### Synthesis Pragmas

Never use `full_case` or `parallel_case`. See Mills' [Yet Another Latch and Gotchas Paper](http://lcdm-eng.com/papers/snug12_Paper_final.pdf), Cummings' [SystemVerilog-2009 Enhancements: priority, unique, unique0](https://dvcon-proceedings.org/wp-content/uploads/systemverilog-2009-enhancements-priority-unique-unique.pdf), and Sutherland/Mills/Spear's [Gotcha Again](http://lcdm-eng.com/papers/snug07_Verilog%20Gotchas%20Part2.pdf).

### Signed Operations

Use signed constructs for signed operations. Before any mixed signed/unsigned operation, explicitly cast unsigned operands with `signed'`; otherwise one unsigned operand makes the operation unsigned. There must be no signed-to-unsigned warnings from simulation/synthesis tools.

##### Mixed-Sign Arithmetic Operations

Explicitly cast unsigned arithmetic operands, padding a leading zero where needed to preserve their positive value:

```verilog
logic signed [7:0] a = 8'sh80;  // -128.
logic incr = 1'b1;
logic signed [15:0] sum;
assign sum = a + signed'({1'b0, incr});  // -127.
// Incorrect: a + incr gives 129 due to implicit unsigned conversion.
```

##### Mixed-Sign Comparison Operations

Explicitly cast unsigned comparison operands and use explicitly signed constants for consistent tools:

```verilog
logic [7:0] unsigned_foo = 8'h00;
logic signed [7:0] signed_bar = 8'shFF;
assign bar_less_than_foo = signed_bar < signed'({1'b0, unsigned_foo});
assign bar_less_than_zero = signed_bar < 'sd0;
// Incorrect: signed_bar < unsigned_foo; signed_bar < 0.
```

##### Arithmetic Right Shift Operations

For arithmetic right shifts of signed values always use `>>>`, never `>>`. For `logic signed [7:0] foo = 8'sh80`, `foo >>> 1` is -64; `foo >> 1` inserts zero and gives +64.

### Bit Widths

Carefully control signal/expression widths.

#### Always be explicit about the widths of number literals.

Size literals to their final width. Binary literals must spell out every bit (`8'b0010_1111`, not `8'b001_1111`); decimal/hex literals need not spell all digits (`16'd2`, `64'h2000`). Avoid implicit zero extension/truncation (`8-bit foo = 2'd2` is incorrect).

Exceptions:

- Unsized integer literals may be used in parameterized-width expressions (`foo = bar + 1`) instead of awkward explicit padding. Narrower values are padded by the language; lint typically catches overly wide values.
- Context-sized `'0` and `'1` are allowed.
- Explicit widths are optional for literals assigned to integer variants (`bit`, `byte`, `shortint`, `int`, `integer`, `longint`).

#### Do not use multi-bit signals in a boolean context

Explicitly compare multibit values with zero instead of implicitly reducing in `if` or boolean operations. Unary reductions are allowed.

```verilog
assign out = (a != '0) && (b == '0);  // Correct for multibit a, b.
if (a != '0) begin ... end
if (&a) begin ... end                // Reduction is permitted.
// Incorrect: out = a && !b; if (a) ...
```

#### Prefer Logical Operators for Single-Bit Signals

Prefer logical `&&`, `||`, `!` to bitwise `&`, `|`, `~` when operating logically on single-bit signals; it makes their intent clear.

#### Handling Width Overflow

Shifts may produce wider results; bit selection/concatenation can be clearer for constant shifts. Addition/negation have an extra carry bit; silently dropping carry on assignment is an allowed width-matching exception (`abc_next = abc + 4'h1`).

### Bit Slicing

Slice only to select an actual portion of a vector. For `[7:0] a, b`, use `a = b` and `a = 8'd7`, not redundant `a = b[7:0]` or `a[7:0] = 8'd7`, which can hide width/lint warnings. A genuinely partial `a[7:1] = 7'd5` is fine.

### Alias statement

Do not use `alias` due to EDA support issues. Prefer normal port connections; use bidirectional `tran` if necessary.

### Implicitly Sized Expressions

Do not rely on implicit subexpression widths. Cast to the required width (aware of truncation) or split into explicitly sized intermediate variables. Self-determined sizes are surprising: multiplication yields `max($bits(a), $bits(b))`, and a shift inside a concatenation can remain only the left operand's width.

```verilog
// Incorrect: one-bit en_a/en_b shifts retain one-bit width, yielding zero.
assign en_bitmap = {(en_a << 4), (en_b << 4)};
// Correct: explicit casts.
assign en_bitmap = {4'(en_a << 3), 4'(en_b << 3)};
// Or explicit intermediate widths:
logic [3:0] en_a_bitmap, en_b_bitmap;
assign en_a_bitmap = en_a << 3;
assign en_b_bitmap = en_b << 3;
assign en_bitmap = {en_a_bitmap, en_b_bitmap};
```

## Design Conventions

### Summary

Use consistent timescale, flow control and CDC, maintain simple top levels, and prefer registered module outputs.

### Timescale

Each project must choose one consistent timescale-specification method for the whole project. Do not invent individual one-off solutions.

### Bus Protocols

Use industry-standard protocol terminology, applying the inclusive-language mapping rules above as necessary. For new protocols, name the initiator/controller `requester` and servicing agent `completer`. More accurate alternatives are allowed when this pattern does not fit, such as leader/follower.

### Flow Control

Use standard Ready/Valid semantics wherever possible. If an interface has no flow control, explain in a comment why backpressure and flow control are unnecessary.

#### Struct Usage

Never group flow-control signals and their qualified data in the same struct. All data in a struct must share the same external qualifiers/flow-control behavior. Put independently qualified data in separate groups.

```verilog
typedef struct packed {
  logic rw;
  foo_address_t addr;
  foo_data_t data;
} foo_t;
foo_t foo;
logic foo_valid;
bar_interrupt_t bar_intr;  // Different flow control, so separate.
```

#### Ready/Valid

The sender asserts Valid for available data; the receiver asserts Ready when it can accept. Transfer occurs on a cycle with both asserted.

- Use `_ready` and `_valid`, with a common prefix for all payload and handshake signals (`foo_ready`, `foo_valid`, `foo_data`).
- Once Valid asserts, keep it asserted and hold data constant until transfer.
- With `_valid_next`, assertion schedules transfer for the next cycle. If Ready is low then, the transfer remains pending and data must remain constant until completion.
- The receiver may deassert Ready at any time.
- A sender must have no combinational path from inputs to outputs. A receiver may derive Ready combinationally from Valid/data; this differs from AXI4's prohibition of all input-to-output combinational paths.
- If a protocol violates these assumptions, document it and use different signal names to show standard semantics do not apply (such as `foo_valid_for_two` for extra-transfer skid behavior).
- `wr_en` or `rd_en` may proxy for Valid when appropriate.

These semantics otherwise follow the [AMBA AXI handshake](https://developer.arm.com/documentation/ihi0022/latest/).

#### Credit/Valid

Credit/Valid decouples transmission from feedback via a credit counter. Consider it for very long, wide, or high-speed pipelines where single-cycle Ready/Valid backpressure is impractical.

#### Req/Ack

Req/Ack is functionally equivalent to Ready/Valid; Ready/Valid is recommended for most new code and Credit/Valid deserves consideration for very long/wide/high-speed buses.

`foo_req` requests an event and remains active until the rising edge of a cycle with `foo_ack` asserted. Ack indicates acknowledgment in that cycle and may assert in the same cycle as Req. Related payload signals must share the prefix and remain valid for the entire request. When Req is low, Ack is unconstrained (it may be tied high if never stalling); the transmitter must inspect Ack only when Req is valid.

### Synchronization

Do not build custom CDC synchronizers; use proven library implementations. In any module instantiating a CDC synchronizer, encapsulate each domain's functional logic in separate submodules. Do not place combinational/sequential logic at the CDC-instantiating hierarchy level. Simple same-domain structural assignments with no inferred logic are allowed (`assign bar_x = bar_y;`); a reduction (`assign bar_x = ^bar_y;`) is not.

Exception: CDC library implementations themselves necessarily mix domains and are exempt.

### Top-level Implementation

Recommended practices:

- Keep top level to submodule instances, with no logic/registers.
- Make port-to-signal mapping simple, preferably identical names. If necessary, consistently substitute prefixes or map ports to larger-bus slices. Interface usage may need consistent instance names distinct from interface/modport names.
- Consider automatically generating top levels, for example using [verilogpp](https://github.com/google/verilogpp).

### Registered Inputs and Outputs

Strongly prefer boundary registers on outputs. Clearly document unregistered outputs of nontrivial modules in the header comment. Top-level modules must have registered inputs and outputs for routing flexibility. Ordinary module inputs generally already come from upstream output flops and need no additional register. External-pin inputs must be retimed before use; top-level blocks may register both sides to maximize routing flexibility.

## Verification

### Summary

The following rules apply to Verilog design verification and include RTL assertions, tests, and UVM.

### Assertions

Express logic assumptions/invariants as runtime assertions, representing behavior that **always** holds, rather than flagging error conditions that simulation may exercise. Put assertions in RTL files near related logic. Use standard assertion macros whenever possible.

- Every input port except clocks, resets, test, debug, or unconstrained data must appear in at least one protocol assertion.
- Every output except test/debug must appear in at least one assertion covering functionality and signal protocol.
- Every synthesized functional register must be in an assertion's cone of influence and within three static logic levels of it.
- Assert important end-to-end behaviors and minimum performance requirements. Supplement with performance-measuring debug logic if necessary.

### Unit Tests and Designer Tests

Shared libraries and reusable-IP design blocks require thorough unit tests; all other blocks require designer tests.

#### Naming Unit Tests

Test-module names must end `_test`, for example `dma_controller_test`.

#### Minimum Testing Requirements

For reusable IP, exhaustively test every feature. Otherwise the block must:

1. Elaborate and pass lint.
2. Leave reset with no X on outputs.
3. Correctly pass and observe at least one transaction.

### Constraints

Constraint names use `lower_snake_case_c`, such as `legal_pkt_size_in_bytes_c`; do not put `c_` at the beginning.

### Plusargs

#### Plusarg Value Assignment

Plusargs must assign explicit values (`+en_foo=1`, never just `+en_foo`). Use `$value$plusargs`; do not use `$test$plusargs`.

```verilog
bit en_ingress;
bit plusarg_found = $value$plusargs("en_ingress=%d", en_ingress);
if (en_ingress) begin
  ...
end
```

#### Plusarg Naming

A plusarg's name must match its storage variable, ignoring prefixes/suffixes of either and case (the source permits uppercase equivalents). For `foo`, `+foo=42`, `+FOO=42`, and `+prefix_foo_suffix=42` are acceptable; `prefix_bar_suffix` may receive `+bar=123`. For `file_name`, `+filename=` or `+file=` is not acceptable.

### Text Formatting

#### Displayed Number Format

Prefix printed binary numbers with `0b` or `'b`, hex with `0x` or `'h`; decimal has no radix prefix. Use formats such as `"0x%0x"`, `"0b%0b"`, `"'h%0h"`, `"'b%0b"`, `"%0d"`. Do not print bare hex/binary or `0d`/`'d` decimal prefixes. Decode large structures field by field rather than making readers decode raw values.

#### String Formatting

Use `$sformatf()` for formatting string-typed format expressions; avoid `$display(str_type, args)` because tools differ. `$display("literal %s", arg)` is allowed; for variable formats use `$display($sformatf(str_type, arg))` or preformat into a string.

#### Line Continuation of String Literals

Never continue a string literal with backslash; concatenate strings with `{}`:

```verilog
$display({"a really long ",
          "long long quoted string"});
```

### Loop operators

Prefer `foreach` over equivalent loops where possible, including compact multidimensional indexing:

```verilog
foreach (data[burst_i, byte_i]) begin
  data[burst_i][byte_i] = 0;
end
```

### Code Within Asserts

Never place side effects in assertion expressions, including `randomize` calls; disabled assertions may not evaluate expressions. Perform the operation separately, then check its result:

```verilog
ret = item.randomize() with { ... };
assert(ret);
// Incorrect: assert(item.randomize()); assert(function_with_side_effects());
```

### Wait and Fork

Always isolate `wait fork` or `disable fork` in an enclosing `fork ... join`, preventing them from affecting earlier unrelated forks:

```verilog
fork begin : isolation_fork
  fork
    begin
      fork1();
    end
    begin
      fork2();
    end
  join_any
  disable fork;
end : isolation_fork
join
```

### Void Casts

Do not discard useful/important returns, including `uvm_config_db::get`, `uvm_hdl_read`, and `randomize`, with `void'`. Check and handle failure instead:

```verilog
if (!item.randomize() with { ... }) begin
  // Issue error.
end
if (!uvm_config_db#(object_type)::get(...)) begin
  // Issue error.
end
```

### Associative arrays

Never use wildcard `[*]` associative indices; specify the type, such as `bit channel_enable[int]`, `register_t registers[string]`, or `int object_count[index_type]`.

### Bind Statements

`bind` is permitted for verification and preferred for assertion-based monitors. `.*` is allowed when binding a module whose ports are all inputs.

### Program block

Do not use `program/endprogram`; use modules and clocking blocks to avoid DUT races.

### Simulator-specific code

Guard all simulator-specific code so every simulator can compile it. Use the simulator-defined names `VCS`, `INCA` (Incisive/NCsim/Irun), `XCELIUM` (Xcelium/xrun), or `DSIM`.

```verilog
`ifdef VCS
  $stack();
`endif  // VCS
```

### Forbidden System Tasks and Functions

- Do not use `$psprintf`; use standard `$sformatf`.
- Do not use `$random` or `$dist_*`, which break SystemVerilog random stability; use `$urandom` or `randomize()`.
- Do not use nonstandard `$srandom`; use `process::self().srandom()`.
- In UVM environments, do not log via `$display`, `$displayb/o/h`, `$write`, `$writeb/o/h`, or severity tasks `$info/$fatal/$error/$warning`; use UVM logging macros.

### UVM

#### Phase Methods

When overriding component phase methods (`build_phase`, `connect_phase`, `run_phase`, etc.), always call `super.<phase_name>(phase)`. This is mandatory for user-defined base classes. An exception is permitted when the immediate base is a UVM library class with a known-empty implementation (`uvm_component`, `uvm_test`, `uvm_env`, etc.); calling super is still recommended for consistency and future changes.

```verilog
virtual function void build_phase(uvm_phase phase);
  super.build_phase(phase);
  // Custom build logic.
endfunction
```

#### UVM Logging

Use only `` `uvm_info(ID, message, verbosity) `` for information, `` `uvm_error(ID, message) `` for issues needing attention, and `` `uvm_fatal(ID, message) `` for critical issues that stop simulation. Do not call underlying `uvm_report_*` functions or use `` `uvm_warning ``. Categorize as info/error/fatal; configure infrastructure to treat warning macros as errors as recommended.

```verilog
`uvm_error(get_full_name(), $sformatf("Unsupported burst type: %s", burst_type.name()))
```

#### UVM Macro Semicolons

Never append semicolons to UVM macro calls, including messaging and sequence actions (`uvm_info`, `uvm_do`, etc.). They can create empty statements and break conditional syntax.

```verilog
`uvm_info("ID", "Message", UVM_LOW)
`uvm_do(my_seq)
```

#### Sequences and Sequence_items

Call a sequence's `randomize()` or `randomize() with {}` before executing it.

#### Creating Objects and Components

Instantiate UVM objects/components with `type_id::create()` for factory overrides. `new()` is permitted for nonreusable objects/components (such as TLM ports, exports, analysis ports) or significant performance gains. For `uvm_component` derivatives, pass `this` as parent.

```verilog
env_h = tb_block_env::type_id::create("env_h", this);
ip_pkt = tx_seq_item::type_id::create("ip_pkt");
```

#### UVM Naming

The `create()` string name must match the assigned variable. Array names must include a square-bracketed index:

```verilog
my_agent = my_agent_type::type_id::create("my_agent", this);
my_agents[i] = my_agent_type::type_id::create($sformatf("my_agents[%0d]", i), this);
// Incorrect names: "different_name" or "my_agents_%0d".
```

#### Class Constructor Default Values

For `uvm_object` derivatives, constructor `name` must default to `""` and be passed to `super.new(name)`; never use nonempty defaults. For `uvm_component` derivatives, neither `name` nor `parent` may have defaults.

```verilog
// uvm_object derivative:
function new(string name = "");
  super.new(name);
endfunction
// uvm_component derivative:
function new(string name, uvm_component parent);
  super.new(name, parent);
endfunction
```

#### Test Execution

Always call `run_test()` with no arguments. Select the test on the command line using `+UVM_TESTNAME=test_name`; never hardcode it in `run_test("my_test_name")`.

## Sources and licensing

This file is an **editorially condensed adaptation**, by the local adaptation's maintainers, of the Google Style Guides' Verilog guide by Google and the contributors/authors named in the source. It is not a verbatim edition: repeated prose and examples were removed or shortened, selected examples were adapted, and presentation was converted to plain Markdown. The fixed source remains available for optional detailed comparison.

- [Fixed upstream source, commit `572c673d9058242c049fbc8248b17029726c7afa`](https://github.com/google/styleguide/blob/572c673d9058242c049fbc8248b17029726c7afa/verilogguide.md).
- [Official published guide](https://google.github.io/styleguide/verilogguide.html).
- The upstream README licenses documentation under [CC BY 3.0](https://creativecommons.org/licenses/by/3.0/) and code samples under [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0). Preserve any more specific attribution/license notices in the originals.

This is an unofficial adaptation; Google does not endorse this skill or adaptation.
