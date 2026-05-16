# Kyokai: What Changed from Austral and Why

**Kyokai** (境界, "boundary") is a systems programming language forked from Austral. It preserves Austral's core safety guarantees — linear types, **capability-based security** (unforgeable linear capabilities; entry/bootstrap in **D48**/**D162**), **D211**'s **effect-system boundary** (Kyokai tracks **external authority** through capabilities only; divergence, non-termination, blocking, transitive allocation, and potential TPOE are **not** type-tracked and must be surfaced by naming and contracts), **no language-level undefined behavior in accepted safe Kyokai code (D73)** with unsafe confined to individually specified primitives and explicit FFI/toolchain contracts, **no hidden control flow**, plus **D143**'s roadmap (honest claims now; **paper proof for sequential λ\_K before `v1.0`**; mechanized proof after self-hosting, likely Coq) — while fixing ergonomic pain points, adding missing infrastructure, and building out a complete toolchain.

This document explains every change Kyokai makes from Austral, why each change was made, and what Kyokai code looks like compared to Austral code. It is written for someone who wants to understand the language without reading the full design specification.

---

## Normative basis (kyokaiplan.md)

**Authoritative design source:** [`kyokaidecided.md`](./kyokaidecided.md) is the record of every decided **`D`** point and its rationale for Kyokai. Per **D86**, eventual **normative** text splits into a core language spec (derived from `australspec/`) and a companion toolchain spec; **`kyokaiplan.md` is neither of those** — it is the design plan that informs both. For *this* comparative document, if anything here disagrees with the plan on a decided point, **`kyokaiplan.md` wins** on substance.

**Final verification pass (2026-05-01):**

1. **Decision IDs:** All **167** distinct canonical **`D###` / `d###`** tokens in this file resolve to a defined decision in `kyokaiplan.md` when matched against **`### D…:`** headings, appendix **`| D…:`** rows, and inline **`D… →`** markers (including **`D7a`** / **`D7b`**, which are sub-points under **`### D7:`**, not separate `### D7a:` headers).
2. **Normative Kyokai claims:** Statements that describe *decided* Kyokai semantics are written to follow the cited **`D`**-points and **`kyokaiplan.md` §3.3**; those passages are **kyokaiplan-verified** in the strict sense that each carries traceability to the plan via **`D-index`** / inline **`D###`** citations.
3. **Illustrative code:** Fenced examples **without** a tying **`D-index`** (or labeled as **baseline** / **sketch** / **example only**) are for intuition and may use placeholder names or CLI flags — they are **not** normative unless the surrounding **`D-index`** pins them to a specific **`D`**.

Together, (1)–(2) are what “**100% `kyokaiplan.md`–verified**” means here: **full traceability and alignment for every cited decision and every D-indexed normative claim**; (3) is explicitly out of that scope.

---

## Table of Contents

0. [Normative basis (kyokaiplan.md)](#normative-basis-kyokaiplanmd)
1. [What Stayed the Same](#1-what-stayed-the-same)
    - 1.1 [New Foundational Commitments](#11-new-foundational-commitments)
2. [Syntax Changes](#2-syntax-changes)
   - 2.1 [Comments](#21-comments)
   - 2.2 [Block Terminators](#22-block-terminators)
   - 2.3 [Operators](#23-operators)
   - 2.4 [File Extensions](#24-file-extensions)
   - 2.5 [Semicolons](#25-semicolons)
   - 2.6 [Numeric Literals](#26-numeric-literals)
   - 2.7 [String and Character Literals](#27-string-and-character-literals)
   - 2.8 [Pattern Matching](#28-pattern-matching)
   - 2.9 [Let-Inference](#29-let-inference)
   - 2.10 [`todo` and `unreachable`](#210-todo-and-unreachable)
   - 2.11 [Named Loop Labels](#211-named-loop-labels)
   - 2.12 [Dual Range Forms](#212-dual-range-forms)
   - 2.13 [`while let` Pattern Matching](#213-while-let-pattern-matching)
3. [Type System Changes](#3-type-system-changes)
    - 3.1 [Universe Constraints vs Classifiers](#31-universe-constraints-vs-classifiers)
    - 3.2 [Fixed-Size Arrays](#32-fixed-size-arrays)
    - 3.3 [Function Types](#33-function-types)
    - 3.4 [The `Never` Bottom Type](#34-the-never-bottom-type)
    - 3.5 [Const Generics](#35-const-generics)
    - 3.6 [Static Strings and Comptime Literals](#36-static-strings-and-comptime-literals)
    - 3.7 [Bitrecords](#37-bitrecords)
    - 3.8 [Indexing Syntax](#38-indexing-syntax)
    - 3.9 [Pinned Types](#39-pinned-types)
    - 3.10 [Operator Overloading via Typeclasses](#310-operator-overloading-via-typeclasses)
    - 3.11 [String Formatting (`Displayable`)](#311-string-formatting-displayable)
    - 3.12 [`Destroyable[T]` Typeclass](#312-destroyablet-typeclass)
    - 3.13 [Record Construction and Update](#313-record-construction-and-update)
    - 3.14 [Type Aliases](#314-type-aliases)
4. [Borrowing and References](#4-borrowing-and-references)
   - 4.1 [Region Inference](#41-region-inference)
   - 4.2 [Auto-Reborrow](#42-auto-reborrow)
   - 4.3 [Mutable-to-Immutable Coercion](#43-mutable-to-immutable-coercion)
   - 4.4 [Reference Syntax](#44-reference-syntax)
5. [Expressions and Statements](#5-expressions-and-statements)
   - 5.1 [UFCS (Uniform Function Call Syntax)](#51-ufcs-uniform-function-call-syntax)
   - 5.2 [Integer Literal Inference](#52-integer-literal-inference)
   - 5.3 [Implicit Unit Return](#53-implicit-unit-return)
   - 5.4 [Operator Precedence](#54-operator-precedence)
   - 5.5 [`for-in` Loops](#55-for-in-loops)
   - 5.6 [Let-Else and `or return`](#56-let-else-and-or-return)
   - 5.7 [`defer` and `errdefer`](#57-defer-and-errdefer)
    - 5.8 [Closures](#58-closures)
    - 5.9 [Generators and `yield`](#59-generators-and-yield)
    - 5.10 [`when` Guards for Platform Branching](#510-when-guards-for-platform-branching)
6. [Module and Package System](#6-module-and-package-system)
   - 6.1 [Module Structure](#61-module-structure)
   - 6.2 [Package System](#62-package-system)
   - 6.3 [Visibility](#63-visibility)
7. [Error Handling](#7-error-handling)
   - 7.1 [Design by Contract](#71-design-by-contract)
   - 7.2 [Result Type](#72-result-type)
8. [Concurrency](#8-concurrency)
   - 8.1 [Structured Concurrency](#81-structured-concurrency)
   - 8.2 [Channels](#82-channels)
   - 8.3 [Atomics](#83-atomics)
   - 8.4 [Mutex and RwLock](#84-mutex-and-rwlock)
   - 8.5 [Select](#85-select)
    - 8.6 [Cancellation and Cooperative Cancellation](#86-cancellation-and-cooperative-cancellation)
    - 8.7 [Rendezvous Channels](#87-rendezvous-channels)
    - 8.8 [Non-Blocking I/O and `Poller`](#88-non-blocking-io-and-poller)
    - 8.9 [Signal Handling](#89-signal-handling)
    - 8.10 [No async/await](#810-no-asyncawait)
9. [FFI and Unsafe Code](#9-ffi-and-unsafe-code)
    - [Inline Assembly](#inline-assembly)
10. [Compile-Time Evaluation](#10-compile-time-evaluation)
    - [`debug` Keyword Stripping](#debug-keyword-stripping)
11. [Toolchain](#11-toolchain)
    - 11.1 [Build System and CLI](#111-build-system-and-cli)
    - 11.2 [Package Manager](#112-package-manager)
    - 11.3 [Formatter](#113-formatter)
    - 11.4 [Testing](#114-testing)
    - 11.4a [Benchmarking](#114a-benchmarking)
    - 11.4b [Doc Tests](#114b-doc-tests)
    - 11.5 [LSP](#115-lsp)
    - 11.6 [Compiler Backend](#116-compiler-backend)
    - 11.7 [Diagnostics](#117-diagnostics)
    - 11.8 [Linting](#118-linting)
    - 11.9 [Code Coverage](#119-code-coverage)
    - 11.10 [Fuzzing and Property Testing](#1110-fuzzing-and-property-testing)
    - 11.11 [Package Index](#1111-package-index)
    - 11.12 [SemVer Checking](#1112-semver-checking)
    - 11.13 [Code Generation](#1113-code-generation)
    - 11.14 [Debugging](#1114-debugging)
    - 11.15 [REPL](#1115-repl)
12. [Standard Library](#12-standard-library)
    - [Pure Kyokai (no unsafe, no FFI)](#pure-kyokai-no-unsafe-no-ffi)
    - [Requires Unsafe Internals (thin trust boundary)](#requires-unsafe-internals-thin-trust-boundary)
    - [Capability-Gated APIs](#capability-gated-apis)
    - [OsString / Path Types](#osstring-path-types)
    - [Unbuffered I/O by Default](#unbuffered-io-by-default)
    - [Vector SIMD](#vector-simd)
13. [Naming Conventions](#13-naming-conventions)
14. [Complete Syntax Comparison](#14-complete-syntax-comparison)

---

## 1. What Stayed the Same

Kyokai preserves the core invariants that make Austral what it is. These are not negotiable:

- **Linear types**. Every linear value must be consumed exactly once. No silent drops, no garbage collector, no destructors. The compiler enforces this statically.
- **Two-universe type system**. Every type is either `Free` (copyable, usable any number of times) or `Linear` (must be used exactly once). Linearity is viral — a record containing a `Linear` field is itself `Linear`.
- **Capability-based security**. Capabilities are linear types representing unforgeable permission tokens. `RootCapability` is only available at the program entrypoint (**D48**/**D162**); capabilities cannot be duplicated, stashed, or forged.
- **Effect-system boundary (D211)**. Kyokai tracks **external authority** through capabilities only; divergence, non-termination, blocking, transitive allocation, and potential TPOE remain **outside** the type system and must be surfaced by explicit naming and contracts.
- **No hidden control flow**. No exceptions, no destructors, no implicit function calls, no implicit type conversions, no garbage collection.
- **TPOE (Terminate Program On Error) (D84)** for **contract violations** (bounds failures, checked-arithmetic traps, failed **`require`** / **`ensure`**, and other operations the language classifies that way — see **D84** / §3.3). TPOE is **immediate hard termination**; it is **not** the same category as ordinary exit with an **`ExitCode`**, explicit **`panic`**, or **runtime-fatal** internal failures, and each mode has its own cleanup/diagnostic contract under **D84**.
- **No variable shadowing**. No uninitialized variables. No global mutable state.
- **Second-class references**. References (`&[T]` and `&![T]`) are parameterized by a region that exists only in the lexical scope of the borrow. The type of an escaped reference literally cannot be written — references are unforgeable and unexportable by construction.
- **Explicit function signatures and one-direction typing**. Function signatures remain fully annotated; types flow in one direction with **no Hindley–Milner** inference. Locals may use **`let name := expr`** only when the RHS already fixes exactly one type (**D46**, tautology rule **D87**) — not general local type inference.
- **Module system with interface/body split**. Public API is declared in the interface file; implementation is in the body file.

Austral's 11 linearity checking rules are preserved exactly. The borrow checker works the same way. The capability system works the same way. The fundamental safety story is unchanged.

### 1.1 New Foundational Commitments

Beyond what Austral provides, Kyokai makes explicit commitments that shape the entire language:

**`Kyokai.*` reserved namespace (D1)**: All standard library types, modules, and capabilities live in the `Kyokai` namespace. User code cannot define types or modules in this namespace. This ensures the standard library has a clean, unambiguous home.

**RIIK (Rewrite-It-In-Kyokai) and pure Kyokai stdlib (D64)**: Per **D64**, pure Kyokai covers computation (math, strings, collections, algorithms); OS interaction stays behind thin syscall FFI and capability wrappers (**D20**), not a hidden libc layer for those domains. No C stdlib dependency for core algorithms. This keeps the stdlib auditable, portable, and subject to linear type enforcement.

**1:1 OS thread model (D164)**: Kyokai uses a 1:1 model — one Kyokai task maps to one OS thread. No green threads, no goroutine-style scheduling. High-concurrency programs spawn real OS threads and use the concurrency primitives for synchronization. This keeps the mental model simple: when you call a blocking operation, a real OS thread blocks.

**Explicit allocator model (D44)**: Memory allocation requires an explicit allocator parameter. The global heap is not implicit. Functions that allocate take `allocator: Allocator` as a parameter. This makes allocation visible, auditable, and controllable.

**Formalization roadmap (D143)**: Per **D143**, Kyokai commits to accurate wording today, a **paper proof for sequential λ\_K before `v1.0`**, and a **mechanized proof after self-hosting** (likely Coq). That sequence documents the soundness argument for implementors; the sequential-core target includes linear types, regions, capabilities, and the borrow checker.

---

## 2. Syntax Changes

### 2.1 Comments

**Austral**: `--` line comments (Ada/Haskell style). `"""..."""` docstrings. No block comments.

**Kyokai**: `//` line comments. `///` doc comments. `//!` module doc comments. No block comments.

```austral
-- This is an Austral comment
function foo(): Unit is
    return nil;
end;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// This is a Kyokai comment
function foo(): Unit is
    // body here
qed;

/// This documents the function below
function bar(x: Int32): Int32 is
    return x;
qed;
```
**D-index**
d63 — `//` / `///` / `//!` comment + doc-comment surface (no block comments).
d9 — `fi` / `qed` terminators vs Austral `end if` / `end` in the paired examples.

**Why**: **D63** owns the `//`/`///`/`//!` lexer surface; **D9** still supplies the proof/control terminators (`qed`, `fi`, …). Doc comments (`///`) attach to the following declaration per §3.3 — unattached doc comments are compile errors. Module docs (`//!`) must appear before the first non-doc token.

### 2.2 Block Terminators

**Austral**: Every block ends with `end` plus the construct name: `end if;`, `end for;`, `end while;`, `end case;`, `end;` (for functions), `end module body.`

**Kyokai**: Two categories of terminators that tell you what ended:

**Category 1 — Reversed keywords** (Algol 68 style, for pure control flow):

| Construct     | Opens with       | Closes with |
|---------------|------------------|-------------|
| Conditional   | `if ... then`    | `fi;`       |
| Pattern match | `case ... of`    | `esac;`     |
| Loops         | `for/while ... do` | `od;`     |

**Category 2 — Semantic boundary words** (for state and type boundaries):

| Construct   | Opens with             | Closes with | Meaning                              |
|-------------|------------------------|-------------|--------------------------------------|
| Functions   | `function ... is`      | `qed;`      | Proof complete (linearity verified)  |
| Methods     | `method ... is`        | `qed;`      | Methods are functions                |
| Instances   | `instance ... is`      | `qed;`      | Proof that type satisfies typeclass  |
| Type defs   | `record/union ... is`  | `build;`    | Type definition constructed          |
| Typeclasses | `typeclass ... is`     | `spec;`     | Contract/specification declared      |
| Borrows     | `borrow ... do`        | `drop;`     | Reference is dropped                 |
| Modules     | `module body ... is`   | `seal;`     | Module symbol table sealed           |
| FFI blocks  | `foreign "C" is`       | `mon;`      | Foreign gateway (門) closed          |
| `select`    | `select … when … do`   | `pick;`     | Multi-channel wait boundary (not `esac`) |
| Task groups | `taskgroup … do`       | `join;`     | Structured join / borrow capture to `join` |
| Unsafe audit| `unsafe contract … is` | `audit;`    | `pragma Unsafe_Module` unsafe coverage (**D245**) |

```austral
-- Austral
function main(): ExitCode is
    if x > 0 then
        return ExitSuccess();
    end if;
    return ExitFailure();
end;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai
function main(): ExitCode is
    if x > 0 then
        return ExitSuccess();
    fi;
    return ExitFailure();
qed;
```
**D-index**
d9 — reversed control (`fi`, `od`, `esac`) + semantic boundaries (`qed`, `build`, `spec`, `drop`, `seal`, `mon`, `pick`, `join`, `audit`).

**Why**: **D9** splits reversed control words (`fi`, `od`, `esac`) from semantic boundary words (`qed`, `build`, `spec`, `drop`, `seal`, `mon`, `pick`, `join`, `audit`) so each closer names what ended — more informative than a bare `}` or generic `end if;`. Shallow control flow is still mostly a consequence of linearity, but the terminator vocabulary is a deliberate syntactic choice, not an accident of parsing.

### 2.3 Operators

#### Not-Equal: `!=` instead of `/=`

**Austral**: `/=` (Ada/Haskell convention).

**Kyokai**: `!=` (C/Rust/Zig/Go/Python/JavaScript convention).

```austral
-- Austral
if left_len /= right_len then
    return false;
end if;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai
if left_len != right_len then
    return false;
fi;
```
**D-index**
d10 — `!=` vs Austral `/=`.

**Why**: **D10** picks `!=` because `/=` reads like divide-assign in C-family languages where `/=` really is compound assignment; `!=` matches the mainstream not-equal token.

#### Bitwise Operators: Keywords instead of Symbols

**Austral**: Uses C-style `&`, `|`, etc. for bitwise operations.

**Kyokai**: Uses keyword operators: `band`, `bor`, `bxor`, `bnot`, `shl`, `shr`, `rotl`, `rotr`.

```kyokai
let mask: Nat32 := flags band 0xFF;
let combined: Nat32 := a bor b;
let shifted: Nat32 := value shl 4;
let rotated: Nat32 := bits rotl 8;
```
**D-index**
d41 — keyword bitwise / shift / rotate ops (`band`, `bor`, …, `rotl`/`rotr`) vs Austral symbolic `&`/`|` (borrow `&` stays unambiguous).
d56 — limited explicit precedence; bit/shift/rotate mixes with arithmetic still require parentheses.

**Why**: **D41** keeps borrow `&` disjoint from bitwise work — keyword ops (`band`, `bor`, …, `rotl`/`rotr`) stay grep-friendly and leave no "`&` means two different things" ambiguity.

#### Operator Precedence

**Austral**: Flat expression grammar. ALL binary expressions deeper than one level must be parenthesized. `a + b * c` is a parse error — you must write `a + (b * c)`.

**Kyokai**: Small explicit precedence table. Standard mathematical precedence applies. Bit operations and logical operations still require parentheses when mixed with arithmetic.

**Why**: **D56** replaces Austral's flat grammar with a small precedence table so ordinary arithmetic reads like mainstream languages, while bit/shift/rotate/arithmetic mixes stay parenthesized because the plan treats those mixes as genuinely ambiguous at a glance.

### 2.4 File Extensions

**Austral**: `.aui` (interface) and `.aum` (body/module).

**Kyokai**: `.kyo` (interface) and `.kai` (body/implementation), per **D52** (normative extension pair under **D78**'s deterministic module-root mapping).

The interface declares public API. The body provides implementations plus private declarations. This split is preserved exactly — only the file extensions change.

### 2.5 Semicolons

**Austral**: Uses semicolons.

**Kyokai**: Keeps semicolons. No change.

**Why**: **D16** keeps semicolons for redundancy that aids reading and parser error recovery (Borretti's rationale in Austral's `rationale/1.syntax.md`); this section documents explicit parity with that decided tradeoff, not an undecided grammar detail.

### 2.6 Numeric Literals

**Austral**: Integer literals require explicit type annotations everywhere: `(0 : Index)`, `(16384 : Index)`. No digit separators.

**Kyokai**: Bidirectional integer literal inference (D12) plus digit separators (D135).

```austral
-- Austral
var out: ByteBuf := makeByteBuf(16384 : Index);
if left_len > (0 : Index) then
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai — type inferred from context
var out: ByteBuf := makeByteBuf(16384);
if left_len > 0 then

// Digit separators for readability
let million: Int64 := 1_000_000;
let flags: Nat32 := 0xDEAD_BEEF;
let bits: Nat8 := 0b1010_0101;
```
**D-index**
d12 — bidirectional literal typing vs explicit `(n : Index)` everywhere.
d135 — digit `_` separators in literals.
d9 — `fi` terminator in the Kyokai branch example.

**How inference works**: When a numeric literal appears where the expected type is known — function parameter, binary operator peer, variable initializer with explicit type — the compiler infers the literal's type. The literal is still bounds-checked at compile time. If the context is ambiguous, the compiler requires an explicit annotation. The rule only applies to literal expressions, not to variables or complex expressions.

**Why**: **D12** plus **D87**'s tautology rule: when the callee already demands `Index`, repeating `(16384 : Index)` restates a type the context already fixes — the literal form is the information-carrying token.

### 2.7 String and Character Literals

**Austral**: Basic `"..."` strings. `"""..."""` for docstrings but not for raw strings.

**Kyokai**: Four literal families:

| Literal | Syntax | Type | Purpose |
|---------|--------|------|---------|
| String | `"hello"` | `String` | UTF-8 text |
| Raw string | `"""..."""` | `String` | No escape processing |
| Code point | `'A'` | `Nat32` | Unicode code point |
| Byte | `b'A'` | `Nat8` | ASCII byte |

**Why**: **D54** splits `String`, raw `"""…"""`, code-point `'…'`, and byte `b'…'` (`Nat8`) so bytes, Unicode scalars, and UTF-8 text do not alias each other the way C's `char` does; raw strings cover regex/SQL/FFI literals without escape-escape games.

### 2.8 Pattern Matching

**Austral**: `case x of when Foo do ... end case;`

**Kyokai**: `case x of when Foo do ... esac;` — same syntax, only the terminator changes.

```kyokai
case mode of
    when GentooMode do
        gentooArt(&out);
    when CachyMode do
        cachyArt(&out);
esac;
```
**D-index**
d9 — `esac` vs `end case`
d13 — keep `when … do` (not `=>`).

Kyokai keeps `do` in when-clauses (not `=>`). `do` is a keyword — searchable, pronounceable, consistent with `for/while ... do`.

---

### 2.9 Let-Inference

**Austral**: `let name: Type := expr;` — explicit type annotation required.

**Kyokai**: `let name := expr;` — type inferred from the right-hand side. Explicit annotation is optional:

```kyokai
let x := 42;              // inferred as Int
let name: String := "A";  // explicit annotation still allowed
```
**D-index**
d46 — `let :=` only when RHS fixes one type (tautology / D87)
d210 — default `Int` inference for `42` only when consistent with `Index`/family rules in plan.

**Why**: **D46** allows `let name := expr` only when the RHS fixes exactly one type; **D210** pins default integer literal choice to `Int` only when that stays consistent with `Index`/family rules — still no HM-style general inference.

---

### 2.10 `todo` and `unreachable`

**Austral**: No built-in markers for unimplemented or logically impossible code.

**Kyokai**: Two built-in divergent expressions with distinct semantics:

```kyokai
let unfinished := todo;         // explicit panic-category stub (D121); not TPOE
let impossible := unreachable;  // explicit TPOE path (D121); not UB
```
**D-index**
d121 — `todo;` is a **panic-category** stub and warning target; `unreachable;` is an explicit **TPOE** path (still not UB).
d58 — denotable `Never` for divergent expressions.

Both have type `Never` (**D58**). **`unreachable;`** marks a control-flow path the programmer asserts is impossible; **`todo;`** marks deliberately incomplete code and follows the **`panic`** termination family, not contract-violation TPOE (**D121**).

---

### 2.11 Named Loop Labels

**Austral**: No named break/continue.

**Kyokai**: Loops can be labeled with a name. `break` and `continue` can target a specific label:

```kyokai
outer: for i from 0 below 10 do
    for j from 0 below 10 do
        if condition(i, j) then
            break outer;  // exits the outer loop
        fi;
    od;
od;
```
**D-index**
d122 — lexical `break label` / `continue label`
d9 — `fi`/`od` terminators
d32 — `from`/`to`/`below` range surface.

Labels are declared before the loop keyword: `name: for ... do ... od;`. `continue name;` is also supported for nested loops.

---

### 2.12 Dual Range Forms (`to` and `below`)

**Austral**: No range literals.

**Kyokai**: Two range operators create iterator ranges:

```kyokai
for i from 0 to 10 do   // 0, 1, 2, ..., 10 (inclusive)
    process(i);
od;

for i from 0 below 10 do // 0, 1, 2, ..., 9 (exclusive upper bound)
    process(i);
od;
```
**D-index**
d32 — inclusive `to` vs exclusive `below` (+ `Iterable`/`Iterator` core protocol; fusion default per plan).
d32a — eager helper / reduction / collection-building adapters layered on that core (no lazy adapter zoo).
d249 — linear iterators are consumed on every `for-in` exit path; linear yielded items follow ordinary linearity.
d9 — `od` loop terminator.

`to` and `below` are intentionally different: `to` is inclusive (`<=` upper bound), `below` is exclusive (`<` upper bound). This avoids forcing `to n - 1` boilerplate for the common half-open case while keeping inclusive loops explicit.

---

### 2.13 `while let` Pattern Matching

**Austral**: No `while let` sugar. Iterating until an `Optional` is `None` means writing an explicit loop whose body performs a refutable `case` (or equivalent projection) on each iteration — more tokens and control-flow noise than Kyokai’s `while let`, but no magic beyond ordinary Austral constructs.

**D-index**
baseline — contrast intent only; Kyokai deltas are in the paired block and cite the listed `D` points.

**Kyokai**: `while let` provides concise refutable-pattern iteration:

```kyokai
while let Some(item) := stack.pop() do
    process(item);
od;
```
**D-index**
d39 — general refutable-pattern `while let` with exact desugaring (not only `Optional`).

Desugars to a loop that checks the pattern, binds the variable, and loops while the pattern matches. Covers `Optional`, results, and any type with a refutable pattern.

---

## 3. Type System Changes

### 3.1 Universe Constraints vs Classifiers

**Austral**: `Free`, `Linear`, `Type`, and `Region` are used both as parameter constraints and as universe classifiers on type definitions. The distinction is not always clear in the syntax.

**Kyokai**: Explicit separation:
- `T: Type` — parameter accepts either `Free` or `Linear` types (treated conservatively)
- `T: Free` — parameter accepts only `Free` types
- `T: Linear` — parameter accepts only `Linear` types
- `Auto` — declaration-site classifier meaning "universe depends on type parameters"

```kyokai
// T: Type means this works with Free or Linear types
record Box[T: Type]: Auto is
    value: T;
build;

// Box[Int32] is Free (Int32 is Free)
// Box[File] is Linear (File is Linear)
```
**D-index**
d195 — `Type`/`Free`/`Linear` constraints vs declaration-site `Auto`
d9 — `build` terminator on records.

**Why**: **D195** separates constraint kinds (`Type` vs `Free` vs `Linear`) from declaration-site **`Auto`**, so generic APIs do not accidentally smuggle universe answers into parameter constraints (or vice versa).

### 3.2 Fixed-Size Arrays

**Austral**: No fixed-size arrays. Only `Buffer` (dynamic) and `Span` (view).

**Kyokai**: `Array[T, N]` — stack-allocated, fixed-size, compile-time-known length.

```kyokai
let table: Array[Nat8, 256] := comptime makeEscapeTable();
```
**D-index**
d159 — `Index`-typed const generics / `Array[T,N]`
d18 — `comptime` call-site forcing for table materialization when used that way.

**Why**: **D159**'s `Array[T, N]` keeps fixed-width storage in the value world (stack/static per lowering) so tables and small buffers avoid implicit heap traffic; **D18**'s `comptime` call site is how materialization stays explicit when the initializer must run at compile time.

### 3.3 Function Types

**Austral**: Functions are not values. No callbacks without typeclasses.

**Kyokai**: `Callable[A, R]`, `CallableMut[A, R]`, `CallableOnce[A, R]` typeclass families (up to 4 parameters) provide function-value semantics through the existing typeclass dispatch system. Kyokai also supports explicit-capture closure literals that lower to this callable substrate.

**Why**: **D21** supplies the typed callable/`FnPtr` substrate, **D118** keeps closure capture lists explicit (no Rust-style inference), and **D126** caps arity at the four-parameter families the plan actually standardizes.

---

### 3.4 The `Never` Bottom Type

**Austral**: No named bottom type. Diverging expressions have an implicit bottom type.

**Kyokai**: Built-in `Never` bottom type, denotable in signatures:

```kyokai
function divByZero(): Never is
    panic("division by zero");
qed;
```
**D-index**
d58 — built-in `Never` type
d84 — interaction with `panic`/TPOE termination (process-level paths per plan).

`Never` types diverging paths: `panic(...)`, `unreachable;` (TPOE family per **D121**), `todo;` (panic family per **D121**), and other non-returning control. It enables `or return` chains: if a branch produces `Never`, the `else` branch determines the overall type.

---

### 3.5 Const Generics

**Austral**: No const generics.

**Kyokai**: Value-level generic parameters with `Index` constraint:

```kyokai
generic [N: Index]
function makeBuffer(): Array[Nat8, N] is
    return Array[Nat8, N]::new();
qed;
```
**D-index**
d159 — const generic `N: Index` with value equality + monomorphization rules
d188 — admitted const-index expression forms.

Const generic arguments must be comptime-evaluable. Value-based type equality means `Array[T, 3]` and `Array[T, 4]` are distinct types (enabling monomorphization). Const generics participate in type checking and optimization.

---

### 3.6 Static Strings and Comptime Literals

**Austral**: All string literals are `String` type.

**Kyokai**: Distinct `StaticString` with explicit `static "..."` bridge:

```kyokai
let name: StaticString := static "Kyokai";
let allocName: String := name.toStringIn(&!heap) or return;
```
**D-index**
d120 — `StaticString` + explicit `static "…"` bridge; allocator-taking `toStringIn` conversion
d44 — explicit allocator at runtime string materialization boundary.

`static "..."` creates a `StaticString` backed by static/compile-time data. Required for safe metaprogramming: pattern matching on string content, compile-time checks, and embedding data into compiled binaries all use `StaticString`. Conversion to runtime `String` is explicit and allocator-taking.

---

### 3.7 Bitrecords

**Austral**: No bit-level record types.

**Kyokai**: Explicit `bitrecord` values over fixed-width unsigned backing integers:

```kyokai
bitrecord IPv4Header of UInt32 is
    ver:  BitRange[4];    // version (4 bits)
    ihl:  BitRange[4];    // header length (4 bits)
    dscp: BitRange[6];    // differentiated services (6 bits)
    len:  BitRange[16];   // total length (16 bits)
build;
```
**D-index**
d116 — `bitrecord` values over fixed-width backing integers; masks/shifts only (no C bitfields)
d117 — endian/explicit byte-serialization helpers are separate from field decode (when crossing bytes)
d9 — `build` terminator.

Only masks and shifts — no C-style layout folklore. The backing integer type is explicit. Bitrecord fields are read/written via bit masks and shifts only; no undefined layout.

---

### 3.8 Indexing Syntax

**Austral**: No `[]` operator. Access via explicit methods.

**Kyokai**: Built-in `[]` sugar over fixed `Indexable` / `IndexableMut` typeclasses:

```kyokai
let second: Nat8 := arr[1];
arr[0] := 42;
```
**D-index**
d36/d132 — built-in `[]` / `[]:=` sugar over fixed `Indexable`/`IndexableMut` with uniform TPOE indexing
d106 — checked half-open slice syntax for standard sequential containers (where slice forms apply).

The semantics are uniform: out-of-bounds access is TPOE (Terminate Program On Error), same as all other contract violations. `Indexable` and `IndexableMut` are standard typeclasses. Slice syntax (`a[i..j]`) uses the same mechanism for span extraction.

---

### 3.9 Pinned Types

**Austral**: No pinned/non-movable distinction.

**Kyokai**: `pinned record` declaration and `PinBox[T]` for heap-allocated pinned values:

```kyokai
pinned record MutexHandle is
    state: State;
build;

let pinned_handle: PinBox[MutexHandle] := PinBox[MutexHandle]::new(allocator, init);
```
**D-index**
d89b — declaration-site `pinned` + `PinBox[T]` stable-address ownership (plain `Box[T]` stays ordinary).

Pinned types cannot be moved after initialization. Required for self-referential structures and types that expose interior mutability through a stable memory address.

---

### 3.10 Operator Overloading via Typeclasses

**Austral**: No operator overloading.

**Kyokai**: Fixed built-in operator typeclasses only — no general overloading:

```kyokai
typeclass Add[L, R] is
    function add(lhs: L, rhs: R): Self;
spec;

instance AddInt64 for Add[Int64, Int64] is
    function add(lhs: Int64, rhs: Int64): Int64 is
        return __add_int64(lhs, rhs);
    qed;
qed;
```
**D-index**
d23 — fixed operator typeclass set (no user-defined operator symbols)
d9 — `spec`/`qed` terminators for typeclass/instance bodies.

Arithmetic (`Add`, `Sub`, `Mul`, `Div`), comparison (`Eq`, `TotalOrder`), and logical (`BitAnd`, `BitOr`, `BitXor`, `Shl`, `Shr`) use standard typeclass dispatch. No user-defined operators beyond the fixed set.

---

### 3.11 String Formatting (`Displayable`)

**Austral**: No standard formatting protocol.

**Kyokai**: `Displayable` typeclass and `FormatSink` for rendering:

```kyokai
typeclass Displayable(T: Type) is
    method display[S: FormatSink](value: &[T], out: &![S]): Result[Unit, S.Error];
spec;
```
**D-index**
d40 — `Displayable` + `FormatSink` separation from raw I/O (**d259** — optional standalone `StandardError` diagnostic typeclass; not `Displayable`; no base `source()` chain).
d40a — `format(alloc, …)` checked `{}` interpolation
d102 — `writeFmt` non-allocating stream path.

Rendering stays separate from raw stream transport. `format(alloc, template, args...) -> Result[String, AllocError]` uses comptime-validated `{}` placeholders. `writeFmt` writes directly to `Writable` streams without intermediate allocation.

---

### 3.12 `Destroyable[T]` Typeclass

**Austral**: No generic destruction protocol.

**Kyokai**: Explicit `Destroyable` typeclass in `Kyokai.Core`:

```kyokai
typeclass Destroyable(T: Linear) is
    method destroy(self: T): Unit;
spec;
```
**D-index**
d124 — explicit `Destroyable` in `Kyokai.Core` for generic linear teardown; no auto-derive.

No auto-derivation or implicit destruction. Enables generic linear cleanup for channels, containers, and iterators. Types that own resources implement `Destroyable` explicitly.

---

### 3.13 Record Construction and Update

**Austral**: Constructor syntax `MkTypeName(field: value, ...)`.

**Kyokai**: Record construction and update:

```kyokai
// Construction
let rect: Rectangle := Rectangle { width: 10, height: 20 };

// Update (D138): `with source` is last; overrides listed fields; `Free` copies other fields
let taller: Rectangle := Rectangle {
    height: 30,
    with rect,
};
```
**D-index**
d35 — brace-plus-colon record construction `Type { field: expr }` (no tuple/partial positional forms)
d138 — type-led functional update `Type { …, with source }` (`Free` copies fields; `Linear` consumes source per plan).

**D138** requires the trailing `with source` form — `source` must have the same record type, overrides precede it, and `Linear` sources are consumed rather than silently copied.

---

### 3.14 Type Aliases

**Austral**: No type alias syntax.

**Kyokai**: Explicit `type alias` for transparent type synonyms:

```kyokai
type alias UserId: Nat64;
type alias Callback[T]: Callable[Box[Allocator], T, Unit];
```
**D-index**
d50 — transparent `type alias` synonyms only (no new nominal identity).

Type aliases are transparent — the alias is interchangeable with the underlying type. Supports generic aliases for complex type constructions.

---

## 4. Borrowing and References

### 4.1 Region Inference

**Austral**: Every function that takes any borrow must declare `generic [R: Region]` and name every region parameter explicitly, even when there is exactly one region.

```austral
-- Austral: 18 lines of region boilerplate in FastIO.aui
generic [R: Region]
function length(buf: &[ByteBuf, R]): Index;

generic [R: Region]
function capacity(buf: &[ByteBuf, R]): Index;

generic [R: Region]
function clear(buf: &![ByteBuf, R]): Unit;

-- 11 region parameters for one function
generic [R: Region, S: Region, T: Region, U: Region, V: Region,
         W: Region, X: Region, Y: Region, Z: Region, A0: Region, A1: Region]
function renderFetch(
    out: &![ByteBuf, R],
    distro: &[ByteBuf, S],
    kernel: &[ByteBuf, T],
    -- ... 11 Region parameters
): Unit is
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

**Kyokai**: Region parameters are inferred when unambiguous. The `generic [R: Region]` line disappears for the common case.

```kyokai
// Kyokai: region inferred — zero boilerplate
function length(buf: &[ByteBuf]): Index;
function capacity(buf: &[ByteBuf]): Index;
function clear(buf: &![ByteBuf]): Unit;

// Multiple borrows: each gets its own inferred region
function renderFetch(
    out: &![ByteBuf],
    distro: &[ByteBuf],
    kernel: &[ByteBuf],
    // ... no region parameters needed
): Unit is
```
**D-index**
d6 — anonymous-by-default `&[T]` / `&![T]` signatures vs explicit `generic [R: Region]` boilerplate.

**Why**: **D6** makes `&[T]` / `&![T]` the anonymous-region default so the common single-scope borrow case drops the `generic [R: Region]` header entirely; named regions remain for the rare APIs that actually relate two different region parameters in their signatures.

### 4.2 Auto-Reborrow

**Austral**: When passing a mutable borrow to another function, you must manually dereference and re-borrow with `&~`:

```austral
-- Austral: &~ appears 200+ times in bfetchaust
generic [R: Region]
function ansi(out: &![ByteBuf, R], code: Index): Unit is
    appendByte(&~out, 27 : Nat8);
    appendByte(&~out, '[');
    appendIndexDecimal(&~out, code);
    appendByte(&~out, 'm');
    return nil;
end;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

**Kyokai**: The compiler inserts re-borrows automatically when passing `&![T]` to a function expecting `&![T]`:

```kyokai
// Kyokai: auto-reborrow — no &~ needed
function ansi(out: &![ByteBuf], code: Index): Unit is
    out.appendByte(27);
    out.appendByte('[');
    out.appendIndexDecimal(code);
    out.appendByte('m');
qed;
```
**D-index**
d7b — compiler-inserted `&~` when passing `&![T]` to `&![T]` (tautology / D87)
d12 — literal `27` without `: Nat8` where context fixes it
d8 — omitted trailing `return nil` on `: Unit` where shown
d9 — `qed` vs `end`.

**Why**: **D7b** plus **D87**: when callee and caller both want `&![T]`, the compiler may insert the forced `&~` reborrow — there is only one well-typed completion, so spelling it every time is pure ceremony.

### 4.3 Mutable-to-Immutable Coercion

**Austral**: No coercion between `&![T, R]` and `&[T, R]`.

**Kyokai**: `&![T]` can be passed where `&[T]` is expected. This is specified as a temporary immutable reborrow, not a subtype relation. The original mutable borrow is suspended for the lifetime of the temporary immutable reborrow. The reverse (`&[T]` to `&![T]`) is never legal.

**Why**: **D187** (not subtyping): passing `&![T]` where `&[T]` is expected inserts a temporary immutable read-reborrow; the mutable borrow is suspended for that call, and widening back to mutable is still forbidden.

### 4.4 Reference Syntax

**Austral**: `&x` (immutable borrow), `&!x` (mutable borrow), `&~x` (re-borrow), `~x` (dereference), `&[T, R]` (immutable ref type), `&![T, R]` (mutable ref type).

**Kyokai**: Same operators, but regions are inferred in types:
- `&x` (immutable borrow) — unchanged
- `&!x` (mutable borrow) — unchanged
- `~x` (dereference) — unchanged
- `&~x` (re-borrow) — still valid but rarely needed due to auto-reborrow
- `&[T]` (immutable ref type) — region inferred
- `&![T]` (mutable ref type) — region inferred

---

## 5. Expressions and Statements

### 5.1 UFCS (Uniform Function Call Syntax)

**Austral**: All function calls are VSO (verb-subject-object): `appendByte(&~out, 27)`.

**Kyokai**: SVO (subject-verb-object) via UFCS: `out.appendByte(27)`. The first parameter becomes the receiver.

```austral
-- Austral: VSO
appendByte(&~out, 27 : Nat8);
appendByte(&~out, '[');
appendIndexDecimal(&~out, code);
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai: SVO via UFCS + auto-reborrow
out.appendByte(27);
out.appendByte('[');
out.appendIndexDecimal(code);
```
**D-index**
d7a — UFCS `x.f(args)` desugars to `f(x,args)` with import-based resolution
d7b — auto-reborrow in the Kyokai column
d12 — literal inference for `27` / `'['` where context fixes types

**Rules**:
- `x.f(args)` desugars to `f(x, args)`.
- It is always syntactic sugar. No method resolution, no vtable, no dispatch magic.
- Both forms (`x.f(args)` and `f(x, args)`) remain valid and interchangeable.
- Method-style is preferred for "object does action" patterns. Free-function style is preferred for "transform value" patterns.

**Why**: **D7a** UFCS makes the receiver the subject (`out.appendByte(27)` desugars to `appendByte(out, 27)`), which pairs naturally with **D7b** so the `&~` soup disappears from the common path.

### 5.2 Integer Literal Inference

**Austral**: No type inference at all. Every integer literal requires explicit annotation: `(0 : Index)`, `(16384 : Index)`.

**Kyokai**: Bidirectional inference for literals only. When the expected type is known from context, the compiler infers the literal's type.

```austral
-- Austral
var out: ByteBuf := makeByteBuf(16384 : Index);
if left_len > (0 : Index) then
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai
var out: ByteBuf := makeByteBuf(16384);
if left_len > 0 then
```
**D-index**
d12 — literal-only bidirectional typing
d9 — `fi` terminator.

**Why**: **D12** is literal-only bidirectional typing: the peer `Index` operand fixes `0`'s type, so annotating `(0 : Index)` repeats tautological information (**D87** again).

### 5.3 Implicit Unit Return

**Austral**: Every function returning `Unit` must explicitly write `return nil;` at the end.

```austral
-- Austral: return nil appears 18 times in bfetchaust
function nl(out: &![ByteBuf, R]): Unit is
    appendByte(&~out, 10 : Nat8);
    return nil;
end;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

**Kyokai**: Functions declared `: Unit` may omit the trailing `return nil;`. The compiler inserts `return nil;` if control reaches the end of a `: Unit` function without an explicit return.

```kyokai
// Kyokai: implicit unit return
function nl(out: &![ByteBuf]): Unit is
    out.appendByte(10);
qed;
```
**D-index**
d8 — implicit trailing `nil` return for `: Unit` bodies
d7b — auto-reborrow in calls
d6 — anonymous regions in `&![ByteBuf]`
d9 — `qed`.

**Why**: **D8** applies only at the end of `: Unit` bodies: `nil` is the unique inhabitant, so trailing `return nil;` carries no information; mid-body exits still use explicit `return` / `return;` because control flow there is *not* forced.

### 5.4 Operator Precedence

**Austral**: Completely flat. All compound expressions require explicit parentheses.

**Kyokai**: Small explicit precedence table with standard mathematical rules. `*` and `/` bind tighter than `+` and `-`. Comparison operators have lower precedence than arithmetic. Logical operators have lowest precedence. Bit operations require parentheses when mixed with other categories.

**Why**: Same **D56** story as §2.3 — math precedence is standard, but bit/shift/rotate/arithmetic mixes stay parenthesized because the plan treats them as visually ambiguous without explicit structure.

### 5.5 `for-in` Loops

**Austral**: `for i from 0 to n do ... end for;` — range-based only.

**Kyokai**: Adds `for item in iterable do ... od;` using the `Iterable` typeclass.

```kyokai
for item in buffer do
    process(item);
od;

// Range loops still exist
for i from 0 to n do
    process(i);
od;
```
**D-index**
d32 — `for … in …` over `Iterable` plus range `for … from … to/below …`
d249 — linear iterators / linear yielded items follow `for-in` consumption rules on every exit path.
d9 — `od`.

**Why**: **D32**'s `Iterable` hook plus `for … in` sugar covers the common "walk this container" case; **D249** pins linear iterator/value cleanup across `break`/`continue`/normal completion so early exits cannot leak linear payloads.

### 5.6 Let-Else and `or return`

**Austral**: No pattern-matching let. Unwrapping `Optional` or `Result` requires a full `case` expression.

**Kyokai**: Two layers of error-handling sugar.

**Layer 1: `let...else`** — diverging pattern match that binds the success variant in the outer scope:

```kyokai
let Ok(file) := openFile(path) else Err(e) do
    logError(e);
    return Err(e);
fi;
// 'file' is now in scope
```
**D-index**
d15 — diverging `let … else` over two-variant unions (base layer per D15).

**Layer 2: `or return` / `or break` / `or continue`** — syntactic sugar with precise desugaring:

```kyokai
// Sugar:
let file: File := openFile(path) or return;

// Desugars EXACTLY to:
let Ok(file) := openFile(path) else Err(__e) do
    return Err(__e);
fi;
```
**D-index**
d15 — `let … else` is the desugar target for the sugar below.
d15a — `or return` / `or break` / `or continue` exact desugarings + `Result`-only restriction.

With error transformation:
```kyokai
let file: File := openFile(path) or return err => ConfigError.Io(err);
```
**D-index**
d15a — explicit-binder `or return err => expr` inline error mapping (D15a); no implicit error conversion.

**Rules**:
- `let...else` works for any two-variant union (D15); `or return` / `or break` / `or continue` are `Result`-only sugar with exact D15a typing and desugaring rules.
- The else block MUST diverge — this is a hard compile error, not a lint.
- Zero magic — the compiler does not implicitly cast errors or auto-return.
- The desugaring is publicly documented and mechanical.

### 5.7 `defer` and `errdefer`

**Austral**: No defer mechanism. Linear types require explicit consumption, leading to long destroy chains:

```austral
-- Austral: 11 manual destroys in bfetchaust
destroyByteBuf(packages);
destroyByteBuf(gpu);
destroyByteBuf(cpu);
destroyByteBuf(shell);
destroyByteBuf(terminal);
destroyByteBuf(wm);
destroyByteBuf(memory);
destroyByteBuf(uptime);
destroyByteBuf(kernel);
destroyByteBuf(distro);
destroyByteBuf(out);
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

**Kyokai**: `defer` and `errdefer` register cleanup actions at the point of creation:

```kyokai
// Kyokai: cleanup visible at creation point
var out: ByteBuf := makeByteBuf(16384);
defer out.destroy();

var kernel: ByteBuf := makeByteBuf(256);
defer kernel.destroy();

// ... use out and kernel ...
// destroys happen automatically in LIFO order at scope exit
```
**D-index**
d2 — Zig-style `defer` / `errdefer` scope hooks (linear cleanup visible in source).
d2a — LIFO ordering per lexical scope.
d2b — exit-path matrix with `panic` vs TPOE vs structured exits (**D208**, **D227**).

**Rules** (per **D2a**/**D2b**, **D208**, **D227**):
- **Structured exits** (fallthrough, `return`, `break`, `continue`) run ordinary **`defer`** in LIFO order.
- **`panic(message)`** runs ordinary **`defer`** (then abnormal termination per **D84**); it does **not** run **`errdefer`**.
- **`or return`** is a structured **error** exit: prior **`defer`** and **`errdefer`** registrations for that scope run in LIFO order.
- **`break` / `continue` / `or break` / `or continue`** run ordinary **`defer`** only (no **`errdefer`** on those paths unless another decided rule applies).
- **TPOE** (contract traps, bounds, failed `require`, etc.) runs **neither** user **`defer`** nor **`errdefer`** — immediate hard termination (**D2b**).
- Deferred statements are visible in source — not hidden destructors.
- **`defer`** / **`errdefer`** do not consume linear values at registration time; the deferred body runs at the relevant exit and performs consumption then.
- Deferred bodies may use only local control flow — `return`, `break`, `continue` inside a deferred body are compile-time errors.
- `panic(message)` is legal in deferred bodies.

---

### 5.8 Closures

**Austral**: No closures. Callbacks must be explicit named records implementing a typeclass.

**Kyokai**: Explicit closure literals with captured variables:

```kyokai
let adder: Callable[Int32, Int32] := fn [base] (x: Int32): Int32 => base + x;
```
**D-index**
d118 — explicit capture list closure literals
d197 — lowers to the correct `Callable`/`CallableMut`/`CallableOnce` member
d21 — callable substrate.

The explicit capture list makes the environment visible; there is no ambient closure capture (**D118**). **D197** then picks `Callable` / `CallableMut` / `CallableOnce` from the **strongest** access the body needs: owned linear captures force **`CallableOnce`**, mutable borrows or mutable use of by-value captures force **`CallableMut`**, and the remaining cases use **`Callable`**. No hidden captures and no Rust-style capture inference.

---

### 5.9 Generators and `yield`

**Austral**: No generators.

**Kyokai**: Named stackless pull generators with `yield`:

```kyokai
generator countdown(start: Int32): Int32 is
    var i: Int32 := start;
    while i > 0 do
        yield i;
        i := i - 1;
    od;
qed;

for n in countdown(10) do
    printInt(n);
od;
```
**D-index**
d198 — nominal `generator` + `yield` iterator model (not async)
d32 — `for … in` consumes iterator protocol.

Generators are first-class iterator declarations. They do not create concurrency, stackful coroutines, or opaque return types. They interact with callers through the ordinary `Iterator` protocol — `yield` produces a value and suspends, `next` resumes execution.

---

### 5.10 `when` Guards for Platform Branching

**Austral**: Platform-specific code requires separate compilation units or conditional compilation at the module level.

**Kyokai**: `when` guards at the declaration level for feature-conditional compilation:

```kyokai
function pageSize(): Index when target.os == Os.Linux is
    return 4096;
qed;

function pageSize(): Index when target.os == Os.MacOS is
    return 16384;
qed;
```
**D-index**
d19 — declaration-level `when target…` guards + three-tier conditional model
d19a — tier-1 selection ordering + overlap/zero-match errors in shared modules
d123 — no body-level `comptime case target.os` branching.

Tier 1 selection happens before module graph construction. A false `when`-guarded declaration is semantically absent. Overlap and zero-match among alternate declarations in selected shared modules are compile-time errors. No body-level inline target branching is part of the model.

---

## 6. Module and Package System

### 6.1 Module Structure

**Austral**: `.aui` (interface) + `.aum` (body). Module declaration: `module Foo is ... end module.` / `module body Foo is ... end module body.`

**Kyokai**: `.kyo` (interface) + `.kai` (body). Module declaration: `module Foo is ... seal;` / `module body Foo is ... seal;`

```kyokai
// Foo.kyo (interface)
module Foo is
    function bar(x: Int32): Int32;
seal;

// Foo.kai (body)
module body Foo is
    function bar(x: Int32): Int32 is
        return x + 1;
    qed;
seal;
```
**D-index**
d9 — `seal` / `qed` terminators vs `end module.` / `end module body`.
d52 — official `.kyo` / `.kai` extensions paired with **D78**'s deterministic module-root mapping.

### 6.2 Package System

**Austral**: No package system. No build system. `austral compile` takes a flat file list.

**Kyokai**: Full package system with `kyokai.toml` manifests, git-based dependency resolution, and a lockfile.

```toml
# kyokai.toml
[package]
name = "myproject"
version = "0.1.0"

[dependencies]
somelib = { git = "https://github.com/user/somelib", rev = "abc123" }
```
**D-index**
d26 — CLI/manifest toolchain surface (`kyokai.toml`, profiles, backends per plan)
d51 — git + pinned `rev` dependency model (and lockfile reproducibility)
d224 — `[generate]`-style build steps where shown in §11.13 examples.

**Key design decisions**:
- Dependencies are always git-based with explicit revision pinning. No central publish registry.
- The lockfile records exact resolved revisions for reproducible builds.
- `kyokai add <name>` can resolve through an official read-only package index, but the manifest always records an explicit git source + revision.
- No `kyokai publish` command. Packages live in git.

### 6.3 Visibility

**Austral**: Module-level public/private only. Types can be opaque, public, or private.

**Kyokai**: Three-level visibility — `public`, `internal`, and `private`:

```kyokai
public function apiCall(...);        // visible to all importers
internal function sharedHelper(...); // visible within this package only
// private is default for items in .kai files
```
**D-index**
d17 — `public` / `internal` / `private` visibility lattice.

`internal` enables shared helpers between sibling modules without exposing them as public API. The workspace boundary does not affect visibility — `internal` is package-visible, never workspace-visible.

---

## 7. Error Handling

### 7.1 Design by Contract

**Austral**: No contract syntax. Manual `if not cond then abort();` checks.

**Kyokai**: `require` preconditions and `ensure` postconditions in the function signature:

```kyokai
function binarySearch(buf: &[Buffer[Int32]], target: Int32): Optional[Index]
    require buf.length() > 0
    ensure result.isSome() implies buf.nth(result.unwrap()) = target
is
    // implementation
qed;
```
**D-index**
d53 — `require`/`ensure` contracts on signatures (+ contextual `result` per **D125** when referenced)
d129 — `old(expr)` only for pure entry-state `Free` snapshots in `ensure`.

**Rules**:
- Preconditions (`require`) go between the parameter list and `is`.
- Postconditions (`ensure`) follow preconditions. `result` is a contextual keyword binding the return value.
- `old expr` is allowed in `ensure` for pure entry-state expressions over `Free` data only.
- Always checked — no build profile strips them.
- Failed contracts are TPOE (program terminates).
- No type invariants.
- Contracts appear in `.kyo` interface files — they are part of the public API contract.

### 7.2 Result Type

**Austral**: No `Result` type in stdlib.

**Kyokai**: `Result[T, E]`, `Optional[T]`, and the `target` configuration object are language-level built-ins (not stdlib imports — **D24**). `Result[T, E]` is defined as:

```kyokai
union Result[T: Type, E: Type]: Auto is
    case Ok(value: T);
    case Err(error: E);
build;
```
**D-index**
d24 — `Result` / `Optional` / `target` as language built-ins (not prelude modules).
d195 — `Auto` universe on parameterized `Result` when parameters vary.

Because `Result[File, Error]` is `Linear` when `File` is `Linear`, the type system forces exhaustive handling of both cases — you cannot silently ignore an error.

---

## 8. Concurrency

Austral has no concurrency model at all. Borretti explicitly deferred it. Kyokai adds a complete concurrency system: **D3**'s structured model, **D252**'s mandatory `taskgroup do … join;` around `spawn`, **D164**'s 1:1 OS threads, **D156**'s explicit rejection of async/await, plus the channel/atomic/`select`/cancellation/`Poller` decisions cited in each subsection below.

### 8.1 Structured Concurrency

All concurrency is structured — **`spawn` is legal only inside `taskgroup do … join;`**, and **`join;`** is the visible blocking join point (**D252**, **D9**).

```kyokai
taskgroup do
    spawn [] do
        tick();
    od;

    spawn [&cfg, sender, &counter] do
        if cfg.enabled() then
            counter.fetchAdd(1, SeqCst);
            let _ : Unit := sendBlocking(&!sender, value) or return;
            closeSender(sender);
        fi;
    od;
join;
```
**D-index**
d252 — mandatory `taskgroup do … join;` around `spawn`; `join;` is the structured join barrier.
d3 — structured concurrency substrate (channels, tasks, explicit joins).
d164 — 1:1 OS thread tasks (no M:N).
d88 — explicit `spawn [captures]` lists (no ambient closure capture).
d168 — explicit channel/value error propagation (no implicit task error wrapper).
d15a — `or return` inside tasks when used exactly as sugar.
d9 — `join;` is a **D9** semantic boundary terminator (not a reversed keyword).

**Rules**:
- The capture list is mandatory. `spawn [] do ... od;` is the zero-capture form.
- `spawn` appears only inside `taskgroup do ... join;` (**D252**).
- `name` captures by value (ownership transfer for `Linear` types).
- `&name` captures by immutable borrow (only for `Free` types or shared-access concurrency types like `Atomic`, `Mutex`, `RwLock`); that borrow lives until the child completes at `join;`.
- `&!name` capture is illegal in `spawn`.
- `join;` blocks until every child task spawned in the group has finished.

### 8.2 Channels

Typed, ownership-transferring communication:

```kyokai
// Create a bounded channel with capacity 10
let (sender, receiver): ChannelEndpoints[Message] :=
    makeBoundedChannel(10);
```
**D-index**
d3a — SPSC linear endpoints + explicit bounded/growable constructors + blocking operation names
d146 — linear receiver teardown via `drain` when payloads are linear.
d236 — standard broker over visible SPSC for multi-producer patterns; no MPSC/MPMC/broadcast endpoint primitives.

- **Bounded**: fixed capacity, backpressure via blocking.
- **Growable**: requires an allocator, starts at initial capacity, grows on demand.
- `sendBlocking`: blocks until space available. Returns `Result` (receiver may be closed).
- `recvBlocking`: blocks until data available. Returns `Optional` (None when closed and drained).
- `trySend` / `tryRecv`: non-blocking variants.
- `closeSender` / `closeReceiver`: explicit endpoint completion (linear types enforce this).

### 8.3 Atomics

```kyokai
let counter: Atomic[Int64] := makeAtomic(0);
counter.fetchAdd(1, SeqCst);
let current: Int64 := counter.load(Acquire);
```
**D-index**
d3b — `Atomic[T]` module + explicit `MemoryOrder` on every op; **`CompareExchangeResult[T]`** names CAS outcomes (not `Result[T, T]`).
d141 — C11 `<stdatomic.h>` lowering contract for the C backend (separate from the source-level CAS result type in **D3b**).

- `Atomic[T]` is `Linear` — one owner.
- `T` is constrained to types with hardware atomic support (`Int8`-`Int64`, `Nat8`-`Nat64`, `Index`, `Bool`).
- No default ordering — every operation names its ordering explicitly.
- CAS returns a named union `CompareExchangeResult[T]` (D3b), not `Result[T, T]`.

### 8.4 Mutex and RwLock

```kyokai
let m: Mutex[Buffer[Int32]] := makeMutex(emptyBuffer());

// Locking takes an immutable borrow (shared among tasks)
let guard: MutexGuard[Buffer[Int32]] := lockBlocking(&m);

// Accessing data requires mutably borrowing the guard
access(&!guard).insertBack(42);

// Unlocking consumes the guard (linear type enforces this)
unlock(guard);
```
**D-index**
d100 — mutex/rwlock linear guards + `access`/`unlock` patterns
d88 — `&[Mutex[T]]` shared capture under structured scopes.

- `Mutex[T]` wraps any `Linear` type.
- `lockBlocking` takes an immutable borrow `&[Mutex[T]]` and returns a `MutexGuard[T]`.
- `access` takes a mutable borrow of the guard and returns `&![T]`.
- `unlock` consumes the guard — forgetting to unlock is a compile-time linearity error.
- `RwLock[T]` provides `readLockBlocking()` and `writeLockBlocking()`.
- `tryLock()` variants return immediately.

### 8.5 Select

Multi-channel waiting:

```kyokai
select
    when recvBlocking(&!rx1) as Some(msg) do
        process(msg);
    when recvBlocking(&!rx2) as Some(msg) do
        handleOther(msg);
    when timeout(deadline) do
        handleTimeout();
pick;
```
**D-index**
d92 — `select … when … do … pick;` multi-channel wait without fixed arm priority.
d258 — **resolved by D92**; normative multi-channel wait is `select … pick;` (plan D258: no separate OS-poller bypass for channels).
d9 — `pick;` is a **D9** semantic boundary terminator (not `esac`).

- Each arm names exactly one blocking channel operation.
- Exactly one ready arm is chosen and its body executes.
- No fixed source-order priority — the language does not guarantee "first arm wins."
- `timeout(deadline)` becomes ready when the deadline passes.

### 8.6 Cancellation and Cooperative Cancellation

**Austral**: No built-in cancellation.

**Kyokai**: Cooperative cancellation via `CancellationToken`:

```kyokai
let token: CancellationToken := makeCancellationToken();

taskgroup do
    spawn [token] do
        for i from 0 below 10000 do
            if token.isCancelled() then
                break;
            fi;
            process(i);
        od;
    od;
join;

// Parent requests cancellation through the token API (D91); children poll `isCancelled()` / deadline variants.
```
**D-index**
d252 — `taskgroup` / `join;` wrapper required around `spawn` in real programs.
d91 — cooperative `CancellationToken` + polling / deadline-aware ops.
d32 — range loop spelling in the example.

Cancellation is cooperative (D91) — tasks poll `token.isCancelled()` (and/or use deadline/token-aware blocking ops that surface `Err(Cancelled)`). Structured cancellation is an ordinary control-flow exit for `defer`; `errdefer` follows D15a (error-exit paths only), not cooperative poll/`break` exits.

### 8.7 Rendezvous Channels

**Austral**: N/A — Austral has no channel type at all; this row exists only to contrast upstream with Kyokai.

**Kyokai**: Explicit `makeRendezvousChannel` for synchronous handoff (**D183**: bounded channel constructors require `capacity >= 1`, so rendezvous is not a magic `0` capacity on `makeBoundedChannel`):

```kyokai
let (sender, receiver): ChannelEndpoints[Signal] :=
    makeRendezvousChannel();
```
**D-index**
d183 — explicit `makeRendezvousChannel` vs overloading `capacity = 0` on bounded channels.

Bounded channels require `capacity >= 1`. Rendezvous is synchronous — the sender blocks until the receiver has taken the value, and vice versa. This keeps handoff liveness explicit in the API surface.

### 8.8 Non-Blocking I/O and `Poller`

**Austral**: No non-blocking I/O model.

**Kyokai**: Portable readiness-based `Poller` API:

```kyokai
let poller: Poller := makePoller(capability);
poller.register(readableFd, ReadReady);
poller.register(writableFd, WriteReady);

let events: Events := poller.wait(timeout);
for event in events do
    case event.kind of
        when ReadReady do handleRead(event.fd);
        when WriteReady do handleWrite(event.fd);
    esac;
od;
```
**D-index**
d93 — explicit linear `Poller` + capability-gated creation + registration/wait API
d9 — `case`/`esac` pattern match terminator.

`Poller` is `Linear` — exactly one owner. Requires a capability to create. There is no hidden global event loop — `Poller` plus libraries built on top of it are the sanctioned path for multiplexed I/O.

### 8.9 Signal Handling

**Austral**: No safe signal handling.

**Kyokai**: Capability-gated `SignalWatcher` type:

```kyokai
let watcher: SignalWatcher := makeSignalWatcher(capability, sigNum);
let events: Events := watcher.wait(timeout);
case events.nth(0) of
    when Some(event) do handleSignal(event.signal);
    when None do handleTimeout();
esac;
```
**D-index**
d95 — `SignalWatcher` poll surface (no safe arbitrary POSIX handlers).

No safe arbitrary signal handlers. Raw registration stays unsafe-only. `SignalWatcher` exposes a pollable readiness source compatible with `Poller`.

### 8.10 No async/await

**Austral**: N/A (no concurrency primitives).

**Kyokai**: Explicit rejection of async/await. Structured concurrency only. High-concurrency I/O builds over the explicit `Poller` API.

No language-level futures, executors, or async function types (D156). Kyokai does not add a second colored async control-flow model; concurrency stays explicit tasks, channels, cancellation (D91), and the D93 `Poller` substrate.

---

## 9. FFI and Unsafe Code

**Austral**: `pragma Unsafe_Module` marks an entire module as unsafe. C FFI is wired through declarations plus the **`External_Name`** pragma (see Austral spec §unsafe modules), not a Kyokai-style `foreign "C" is … mon;` block.

**Kyokai**: Fine-grained unsafe capabilities plus explicit foreign blocks:

```kyokai
// Foreign block with explicit C linkage
foreign "C" is
    function malloc(size: Index): Address[Nat8];
    function free(ptr: Address[Nat8]): Unit;
mon;
```
**D-index**
d20 — `foreign "C" is … mon;` + `pragma Unsafe_Module` + `UnsafeCapability` threading at raw calls (**D245** folds in `unsafe contract … audit;` coverage).
d127 — `mon` terminator distinct from `qed`.

**Key changes**:
- `foreign "C" is ... mon;` syntax with the `mon;` (門, gate/portal) terminator visually distinguishes FFI boundaries.
- **D20** / **D245**: `pragma Unsafe_Module` modules carry source-level `unsafe contract … audit;` blocks so unsafe operations stay auditable; raw calls still thread `&![UnsafeCapability]` at the FFI surface (erased in the emitted C ABI), and safe Kyokai wraps that boundary instead of inventing separate `Unsafe.Pointer` / `Unsafe.Foreign` receiver types in the plan surface.
- `Address[T]` is Kyokai's pointer type — explicitly named to distinguish from references.
- The trust boundary is thin: unsafe internals are wrapped in safe linear APIs.

### Inline Assembly

**Austral**: No inline assembly.

**Kyokai**: `asm` block for low-level code generation:

```kyokai
let result: Int64 := asm("addq %rax, %rbx", Int64) {
    in(reg) value,
    out(reg) result,
    clobbers("rax", "rbx")
};
```
**D-index**
d22 — `asm` blocks (unsafe-only) with operand/clobber contracts.

Inline asm is always unsafe. The block specifies input/output operands, register constraints, and clobbered registers explicitly. The compiler validates operands and clobbers against the target architecture.

---

## 10. Compile-Time Evaluation

**Austral**: No compile-time evaluation.

**Kyokai**: Two mechanisms:

**`constant` expressions**: Extended to support arithmetic/bitwise expressions over other constants.

```kyokai
constant MAX_BUFFER: Index := 1024 * 1024;
constant MASK: Nat32 := 0xFF band 0x0F;
```
**D-index**
d18 — `constant` fold + call-site `comptime` (no Rust-style `const fn` implicit phase).
d18a — deterministic, host-independent `comptime` evaluation contract.
d41 — keyword `band` in constant expressions (vs Austral symbolic bitwise).

**`comptime` call-site keyword**: Forces compile-time evaluation of any pure function over `Free` types.

```kyokai
constant TABLE: Array[Nat8, 256] := comptime makeEscapeTable();
```
**D-index**
d18 — call-site `comptime` forces compile-time evaluation of the RHS.
d18a — evaluation must stay deterministic and host-independent per plan.

**Rules**:
- `comptime` is a call-site keyword, not a definition-site annotation. There is no `const fn`. A function is a function — whether it runs at compile time or runtime is determined where it is called.
- Only `Free`-universe types are eligible (linear types represent runtime resources).
- `static_assert(expr, "msg")` validates invariants at compile time.

**Why not Rust's `const fn`**: **D18** keeps phase selection at the call site (`comptime e`) instead of Rust-style implicit `const fn` bodies where the same call means compile time or runtime depending on context; **D18a** still demands deterministic, host-independent `comptime` evaluation either way.

**Asset embedding**: `@embedFile("path")` embeds file contents as a byte array at compile time with deterministic byte-for-byte semantics.

### `debug` Keyword Stripping

**Austral**: No debug printing mechanism.

**Kyokai**: `debug expr;` for diagnostic output, stripped entirely in release builds:

```kyokai
debug x;  // prints to stderr in debug/test builds; removed in release
```
**D-index**
d45 — `debug expr;` dev-only stderr output; stripped in release; production console I/O stays capability-gated
d40 — operand must be `Displayable` (rendering protocol).

The `debug` keyword:
- Formats any `Displayable` expression to stderr with a trailing newline.
- Is stripped completely from release builds (zero runtime cost).
- Ignores output failures — a failed debug print never changes program control flow.
- In test builds, also prints file/line information.

---

## 11. Toolchain

Austral ships with a bare compiler that takes a flat file list. No build system, no package manager, no formatter, no test runner, no LSP, no debugger integration. Kyokai provides all of these.

### 11.1 Build System and CLI

The `kyokai` binary provides:

| Command | Purpose |
|---------|---------|
| `kyokai build` | Compile the project |
| `kyokai check` | Type-check without codegen |
| `kyokai test` | Run tests |
| `kyokai fmt` | Format source code |
| `kyokai doc` | Generate documentation |
| `kyokai run` | Build and run |
| `kyokai add` | Add a dependency |
| `kyokai generate` | Run declared code generation steps |
| `kyokai semver-check` | Advisory SemVer compatibility check |
| `kyokai test --coverage` | Run tests with coverage |
| `kyokai test --fuzz` | Coverage-guided fuzzing |

No `kyokai lint` — the compiler IS the linter.

### 11.2 Package Manager

- Git-based dependencies with explicit revision pinning.
- `kyokai.toml` manifest.
- Lockfile for reproducible builds.
- Official read-only package index (Go model) for discovery. No publish registry.

### 11.3 Formatter

`kyokai fmt` is an opinionated, deterministic formatter with zero configuration. It produces byte-identical output for semantically identical input. `kyokai fmt --check` exits nonzero if formatting differs (for CI).

### 11.4 Testing

```kyokai
test "buffer append works" is
    var buf: Buffer[Int32] := Buffer.empty();
    defer buf.destroy();
    buf.insertBack(42);
    assert buf.length() = 1;
    assert buf.nth(0) = 42;
qed;
```
**D-index**
d28 — first-class `test` blocks + `kyokai test` runner
d2 — `defer` in tests as in normal code.

- Test blocks are first-class language constructs.
- `kyokai test` discovers and runs all test blocks.
- Property-based testing via `Kyokai.Test.Property` with typed generators `Gen[T]`, shrinking, and deterministic replay.
- Coverage-guided fuzzing via `kyokai test --fuzz` with corpus management and crash reproducers.

### 11.4a Benchmarking

`kyokai bench` runs microbenchmark suites defined in source:

```bash
# Example invocation only — exact flags live in the D26 CLI/toolchain spec
kyokai bench --iterations 10000
```
**D-index**
d136 — normative **`kyokai bench`** workflow (warm-up, repeated measurements, reporting; benchmark discovery syntax stays a toolchain-spec detail).
d26 — `kyokai` CLI surface/manifest integration for subcommands.

Per **D136**, the reference toolchain provides **warm-up**, **repeated measurements**, and **human- and machine-readable** benchmark output; exact metric sets, discovery syntax, and output schema stay **toolchain-spec** detail (with **D26** for CLI/manifest integration), not additional core-language rules in this summary.

Optional wall-clock profiling modes, when offered, remain ordinary **D136**/**D26** toolchain options rather than new Kyokai surface syntax.

### 11.4b Doc Tests

Doc comments containing executable code blocks are automatically tested:

```kyokai
/// Example from Kyokai.Math
/// ```
/// assert 2 * 3 == 6;
/// ```
function multiply(a: Int32, b: Int32): Int32 is
    return __mul_int32(a, b);
qed;
```
**D-index**
d218 — official **`kyokai doc`** plus **`kyokai test --doc`**: extracts fenced `kyokai` examples from `///` / `//!` under explicit doc-test rules.

**D218** names **`kyokai test --doc`** as the doc-test entry point; whether a bare **`kyokai test`** run also executes doc tests is a **D26** CLI default (not duplicated here as a second normative source).

### 11.5 LSP

Official Language Server Protocol implementation. Surfaces the same lint set as the compiler — no second analyzer with divergent opinions.

### 11.6 Compiler Backend

**Austral**: C backend only.

**Kyokai**: C retained as bootstrap/reference/portability backend. LLVM becomes the long-term primary optimizing backend after self-hosting. Backend constraints do not define the language.

### 11.7 Diagnostics

Structured error messages with error codes, source spans, color, suggestions, and machine-readable JSON output:

```
error[KYO-E0042]: linear variable `file` not consumed
  --> src/Fetch.kyo:42:9
   |
42 |     let file: File := openFile(path);
   |         ^^^^ variable declared here
   |
   = note: linear variables must be consumed exactly once
   = help: consider adding `defer file.close();` after this line
```
**D-index**
d29 — structured diagnostics / machine-readable output examples (when not source code).

### 11.8 Linting

All lints live in the compiler. No separate lint tool. Lints are tiered:
- **Error**: correctness and soundness violations (not optional).
- **Warning**: likely bugs (default for new lints).
- **Style**: idiom violations.

Per-project configuration in `kyokai.toml`. No per-line suppression comments (`nolint`, `allow`, etc.).

### 11.9 Code Coverage

`kyokai test --coverage` reports Kyokai-source-level coverage. LCOV plus HTML output. Explicit failure when faithful source mapping cannot be provided (no misleading backend-level coverage).

### 11.10 Fuzzing and Property Testing

Both are committed toolchain facilities:
- Property-based testing: `Kyokai.Test.Property` with `Gen[T]` generators, property runners, shrinking, reproducible seeding.
- Fuzzing: `kyokai test --fuzz` with corpus management, crash reproducers, deterministic replay, and D219 coverage integration.

### 11.11 Package Index

Official read-only package index (Go model). Crawls git repos containing `kyokai.toml`. Provides search, docs, version listing. No `kyokai publish`. Packages stay in git. Alternative third-party indexes are allowed.

### 11.12 SemVer Checking

`kyokai semver-check` compares two public API surfaces using `.kyo` interface files. Classifies changes as breaking (MAJOR), additive (MINOR), or patch-compatible. Advisory only — not enforced. The source of truth for compatibility is the declared public interface surface, not implementation details.

### 11.13 Code Generation

Build-time code generation is declared in `kyokai.toml`:

```toml
[generate]
steps = [
    { inputs = ["proto/*.proto"], outputs = ["gen/proto.kyo", "gen/proto.kai"], command = "protoc-kyokai" }
]
```
**D-index**
d26 — CLI/manifest toolchain surface (`kyokai.toml`, profiles, backends per plan)
d51 — git + pinned `rev` dependency model (and lockfile reproducibility)
d224 — `[generate]`-style build steps where shown in §11.13 examples.

- `kyokai generate` runs declared steps.
- Generated outputs are ordinary files in the source tree.
- Reproducibility rules apply to generation inputs and outputs.
- `@embedFile("path")` handles asset embedding.
- No `build.kai`, no auto-running package code during dependency resolution.

### 11.14 Debugging

Generated code includes source mapping for debuggers. LLVM backend enables standard DWARF debug info. The goal is that GDB/LLDB show Kyokai source, not generated C.

### 11.15 REPL

Official `kyokai repl` for interactive exploration. Declarations persist across inputs within a session. Top-level `errdefer`, `return`, `break`, and `continue` are illegal in REPL scope.

---

## 12. Standard Library

Austral's standard library has 10 modules with basic functionality. Kyokai commits to a batteries-included systems standard library:

### Pure Kyokai (no unsafe, no FFI)

| Module | What It Provides |
|--------|-----------------|
| `Kyokai.Math.Int` | `abs`, `min`, `max`, `clamp`, `gcd`, `lcm`, `isPowerOfTwo`, `popcount`, wrapping/saturating/checked arithmetic |
| `Kyokai.Math.Float` | `abs`, `min`, `max`, `isNan`, `isInf`, `floor`, `ceil`, `round`, `trunc` (IEEE 754 bit manipulation) |
| `Kyokai.Math.Trig` | `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `atan2` (pure polynomial approximations, not libm wrappers) |
| `Kyokai.Math.Exp` | `exp`, `ln`, `log2`, `log10`, `pow`, `sqrt` |
| `Kyokai.Compare` | `PartialEquality`, `Equality`, `PartialOrder`, `TotalOrder` for all builtin types |
| `Kyokai.Compare.Span` | `spanEquals`, `spanCompare`, `spanStartsWith`, `spanEndsWith`, `spanContains`, `spanFind` |
| `Kyokai.String.Ops` | String comparison, concatenation, slicing, searching, trimming, case conversion |
| `Kyokai.String.Format` | Integer-to-string, float-to-string formatting |
| `Kyokai.Collections.*` | `SortedBuffer`, `RingBuffer`, `Deque` |
| `Kyokai.Bits` | `rotateLeft`, `rotateRight`, `byteSwap`, `setBit`, `clearBit`, `testBit` |
| `Kyokai.Ascii` | `isDigit`, `isAlpha`, `isAlnum`, `isSpace`, `toUpper`, `toLower` |
| `Kyokai.Args` | Command-line argument parsing |
| `Kyokai.Result` (helpers) | `Result[T, E]` is a **D24** language built-in (`Ok`/`Err` are keywords); stdlib may expose extra named helpers/combinators without pretending the type lives in a user module |
| `Kyokai.Optional` (helpers) | `Optional[T]` is likewise D24 built-in; stdlib may expose convenience APIs on top |

**Why pure Kyokai math?** **D64** (RIIK) plus the stdlib H02/H03 commitments in the plan: those routines are pure algorithms over IEEE-754 or integer bits, so **D20**'s syscall-sized FFI boundary buys nothing for the core `Kyokai.Math.*` paths.

### Requires Unsafe Internals (thin trust boundary)

| Module | What It Provides | Why Unsafe |
|--------|-----------------|------------|
| `Kyokai.IO.File` | File I/O: open, read, write, close, seek, stat | Syscalls |
| `Kyokai.IO.Terminal` | Terminal I/O with capability-secure API | `isatty`, ioctl |
| `Kyokai.IO.Dir` | Directory operations | `opendir`, `readdir` |
| `Kyokai.Net.Tcp` | TCP networking | Socket syscalls |
| `Kyokai.Net.Dns` | DNS resolution | Network syscalls |
| `Kyokai.Process` | Process spawning | `fork`, `exec` |
| `Kyokai.Env` | Environment variables | `getenv` |
| `Kyokai.Time` | Monotonic and wall-clock time | `clock_gettime` |
| `Kyokai.Random` | Cryptographic and fast PRNG | `getrandom` or ChaCha |
| `Kyokai.Collections.HashMap` | Hash map | Pointer arithmetic in internals |
| `Kyokai.Collections.HashSet` | Hash set | Built on HashMap |

### Capability-Gated APIs

Kyokai uses capability-based security for system resources. Operations on environment variables, files, clocks, and random number generation require explicit capabilities:

```kyokai
// EnvCapability is acquired from RootCapability (D67); APIs take &![EnvCapability]
let home: Optional[String] := getEnv(&!envCap, &keyHome);

// FileCapability from root (D171); Path-typed opens; no ambient cwd — relative paths need a base Directory
let opened: Result[File, IoError] := openForRead(&!fileCap, &absolutePath, mode);

// D172: monotonic Instant/Duration reads and pure arithmetic are ungated; wall clock / sleep / calendar need ClockCapability + Result
let t0: Instant := monotonicNow();
let wall: Result[SystemTime, TimeError] := systemTime(&!clockCap);

// D173: OS entropy / reseeding requires RandomCapability; FastRng/CryptoRng are explicit state after construction
let rng: FastRng := seedFastRngFromOs(&!randCap) or return;
```
**D-index**
d211 — **D211** boundary: capabilities are the only built-in **external-authority** tracking; divergence, blocking, transitive allocation, and potential TPOE are not type-tracked (surface via naming/contracts).
d212 — capability-bearing I/O stays exclusive-by-handle in safe Kyokai; no hidden runtime mutexing; multi-task shared output uses an explicit broker or synchronized wrappers.
d67 — `EnvCapability`; **d171** — `FileCapability` / paths / no ambient cwd; **d172** — time split (ungated monotonic vs gated wall/sleep/calendar); **d173** — randomness split.

| Capability | Operations Gated |
|------------|------------------|
| `EnvCapability` | Reading environment variables |
| `FileCapability` | File and directory operations; no ambient cwd semantics |
| `ClockCapability` | Wall-clock observation, sleep/delay, calendar/timezone (**D172**: monotonic `Instant`/`Duration` reads and pure arithmetic stay **ungated**) |
| `RandomCapability` | Entropy acquisition for RNG initialization |

### OsString / Path Types

**Austral**: No dedicated path types — `String` used for all.

**Kyokai**: Dedicated `OsString`/`OsStr` + `PathBuf`/`Path` types:

```kyokai
let path: PathBuf := PathBuf.fromOsString(rawOsString);
let osStr: OsStr := path.asOsStr();

// Explicit fallible UTF-8 conversion
let pathStr: Result[String, EncodingError] := path.toString();
```
**D-index**
d97 — `OsString`/`Path` types + explicit UTF-8 conversion policy.

Filenames are not guaranteed UTF-8 on all operating systems. Path operations are lexical-only. No implicit UTF-8 conversion or path canonicalization.

### Unbuffered I/O by Default

**Austral**: No buffering specification.

**Kyokai**: Raw I/O streams are unbuffered by default. Explicit `BufferedWriter[T]` / `BufferedReader[T]` wrappers:

```kyokai
// Raw (unbuffered) capability stream after open (D66/D171 — exact open API is stdlib surface)
let raw: Writable := fileWriter;

// Explicit buffering
let buffered: BufferedWriter[File] := BufferedWriter[File]::new(raw);
```
**D-index**
d70 — raw capability streams unbuffered; explicit buffered wrappers.

All raw capability surfaces remain unbuffered. Buffering exists only through explicit wrapper construction. No implicit capability buffering.

### Vector SIMD

**Austral**: No SIMD support.

**Kyokai**: `Vector[T, N]` for fixed-width SIMD vectors:

```kyokai
generic [N: Index where N: BitWidth[256]]
function dotProduct(a: Vector[Float32, N], b: Vector[Float32, N]): Float32 is
    let result: Vector[Float32, N] := a * b;
    return result.horizontalSum();
qed;
```
**D-index**
d104 — `Vector[T,N]` portable lane core + target-gated intrinsics policy
d159 — const `Index` lane counts where shown generically.

Const generics determine the SIMD lane count. Standard widths (128, 256, 512 bits) are validated via `BitWidth` constraints. Vector operations are hardware intrinsics on supporting platforms.

---

## 13. Naming Conventions

**Austral**: No formal naming conventions.

**Kyokai**: Ownership-signaling naming patterns enforced by compiler warnings:

| Prefix | Meaning | Ownership |
|--------|---------|-----------|
| `as*` | Borrowed view, no allocation | `&[T] -> &[U]` |
| `to*` | Borrowed conversion, may allocate | `&[T] -> U` (new allocation) |
| `into*` | Consuming conversion | `T -> U` (ownership transfers) |
| `to*In` | Borrowed conversion with explicit allocator | `&[T], &![Alloc] -> U` |
| `into*In` | Consuming conversion with explicit allocator | `T, &![Alloc] -> U` |

**Why**: **D11b** standardizes `as*` / `into*` / `to*` so skim-reading a call tells you whether storage is borrowed, consumed/reused, or freshly allocated — aligned with the language's "boundary" theme without inventing ad-hoc verb names per module.

The compiler knows whether a function consumes (`T`), borrows (`&[T]`), or mutably borrows (`&![T]`) its first argument. It can verify that names match behavior and emit warnings when they don't.

---

## 14. Complete Syntax Comparison

Here is a complete side-by-side comparison of equivalent programs in Austral and Kyokai.

### Module Declaration

```austral
-- Austral
module Example is
    function greet(name: Span[Nat8, Static]): Unit;
end module.

module body Example is
    function greet(name: Span[Nat8, Static]): Unit is
        -- implementation
        return nil;
    end;
end module body.
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai
module Example is
    function greet(name: &[String]): Unit;
seal;

module body Example is
    function greet(name: &[String]): Unit is
        // implementation
    qed;
seal;
```
**D-index**
d6 — anonymous `&[String]` parameter vs `Span[Nat8, Static]` style in Austral sketch
d9 — `seal`/`qed` module + function terminators.

### Record and Union Types

```austral
-- Austral
record Point: Free is
    x: Float64;
    y: Float64;
end;

union Shape: Free is
    case Circle(radius: Float64);
    case Rectangle(width: Float64, height: Float64);
end;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai
record Point: Free is
    x: Float64;
    y: Float64;
build;

union Shape: Free is
    case Circle(radius: Float64);
    case Rectangle(width: Float64, height: Float64);
build;
```
**D-index**
d9 — `build` vs `end` on type definitions.

### Generic Function with Borrowing

```austral
-- Austral
generic [T: Type, R: Region]
function length(buf: &[Buffer[T], R]): Index is
    return bufferLength(buf);
end;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai — region inferred, UFCS, generic parameter explicit
generic [T: Type]
function length(buf: &[Buffer[T]]): Index is
    return buf.bufferLength();
qed;
```
**D-index**
d6 — drop `generic [R: Region]` when borrows are anonymous
d7a — UFCS `buf.bufferLength()` form.

### Typeclass Definition and Instance

```austral
-- Austral
typeclass Printable(T: Free) is
    generic [R: Region]
    method print(value: &[T, R], out: &![ByteBuf, R]): Unit;
end;

instance Printable(Int32) is
    generic [R: Region]
    method print(value: &[Int32, R], out: &![ByteBuf, R]): Unit is
        appendIndexDecimal(out, value);
        return nil;
    end;
end;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai
typeclass Printable(T: Free) is
    method print(value: &[T], out: &![ByteBuf]): Unit;
spec;

instance PrintableInt32 for Printable[Int32] is
    method print(value: &[Int32], out: &![ByteBuf]): Unit is
        out.appendIndexDecimal(value);
    qed;
qed;
```
**D-index**
d6 — drop per-signature `generic [R: Region]` in the common borrow case
d9 — `spec`/`qed` terminators + instance `qed`
d216 — orphan/coherence rules apply to instances (compiler error paths per plan).

### Control Flow

```austral
-- Austral
if x > (0 : Int32) then
    doSomething();
else if x = (0 : Int32) then
    doNothing();
else
    handleNegative();
end if;

for i from 0 to (n - 1) do
    process(i);
end for;

case shape of
    when Circle(r: Float64) do
        computeCircle(r);
    when Rectangle(w: Float64, h: Float64) do
        computeRect(w, h);
end case;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai
if x > 0 then
    doSomething();
else if x = 0 then
    doNothing();
else
    handleNegative();
fi;

for i from 0 to n - 1 do
    process(i);
od;

case shape of
    when Circle(r: Float64) do
        computeCircle(r);
    when Rectangle(w: Float64, h: Float64) do
        computeRect(w, h);
esac;
```
**D-index**
d9 — `fi`/`od`/`esac` vs `end if`/`end for`/`end case`
d12 — `0` literal without `: Int32` where context fixes it
d32 — range `for` spelling (`to`/`below`) as shown.

### Borrowing

```austral
-- Austral
generic [R: Region]
function ansi(out: &![ByteBuf, R], code: Index): Unit is
    appendByte(&~out, 27 : Nat8);
    appendByte(&~out, '[');
    appendIndexDecimal(&~out, code);
    appendByte(&~out, 'm');
    return nil;
end;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai — no generic region, auto-reborrow, UFCS, implicit unit return
function ansi(out: &![ByteBuf], code: Index): Unit is
    out.appendByte(27);
    out.appendByte('[');
    out.appendIndexDecimal(code);
    out.appendByte('m');
qed;
```
**D-index**
d6 — anonymous regions on `&![ByteBuf]` / `&[ByteBuf]` parameters
d7b — auto-reborrow on `out.appendByte(...)` calls
d7a — UFCS method call sugar
d8 — implicit trailing `nil` on `: Unit` where control falls off the end
d12 — literal `27` / `'['` / `'m'` typing from callee expectations

### Resource Cleanup

```austral
-- Austral: manual destroy chains
let out: ByteBuf := makeByteBuf(16384 : Index);
let kernel: ByteBuf := makeByteBuf(256 : Index);
let distro: ByteBuf := makeByteBuf(256 : Index);
-- ... use them ...
destroyByteBuf(distro);
destroyByteBuf(kernel);
destroyByteBuf(out);
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai: defer at creation point
var out: ByteBuf := makeByteBuf(16384);
defer out.destroy();

var kernel: ByteBuf := makeByteBuf(256);
defer kernel.destroy();

var distro: ByteBuf := makeByteBuf(256);
defer distro.destroy();

// ... use them ...
// destroys happen in LIFO order at scope exit
```
**D-index**
d2 — `defer` replaces manual destroy tail chains
d12 — literal inference for capacities.

### Error Handling

```austral
-- Austral: manual case matching for every Result
case openFile(path) of
    when Ok(f: File) do
        case readAll(f) of
            when Ok(data: Buffer) do
                -- use data
                destroyBuffer(data);
            when Err(e: Error) do
                -- handle error
        end case;
        closeFile(f);
    when Err(e: Error) do
        -- handle error
end case;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai: let-else + or return + defer
let file: File := openFile(path) or return;
defer file.close();

let data: Buffer := readAll(&file) or return;
defer data.destroy();

// use data — cleanup happens automatically
```
**D-index**
d15a — `or return` sugar + explicit `defer` pairing.

### FFI

```austral
-- Austral
pragma Unsafe_Module;
module body Posix is
    function read(fd: Int32, buf: Address[Nat8], count: Index): Int32 is
        -- foreign call
    end;
end module body.
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai: explicit foreign block with mon; terminator
module body Posix is
    foreign "C" is
        function read(fd: Int32, buf: Address[Nat8], count: Index): Int32;
        function write(fd: Int32, buf: Address[Nat8], count: Index): Int32;
    mon;
seal;
```
**D-index**
d87 — implicit/inserted forms only under the tautology rule when this block relies on them.
xref — see `kyokaiplan.md` §3.3 for the authoritative `D` list for this subsection.

### Design by Contract

```austral
-- Austral: manual assertions
function binarySearch(buf: &[Buffer[Int32], R], target: Int32): Optional[Index] is
    if bufferLength(buf) = (0 : Index) then
        abort("buffer must not be empty");
    end if;
    -- implementation
end;
```
**D-index**
baseline — Austral surface shown for contrast; Kyokai deltas are in the paired block and cite the listed `D` points.

```kyokai
// Kyokai: first-class contracts
function binarySearch(buf: &[Buffer[Int32]], target: Int32): Optional[Index]
    require buf.length() > 0
    ensure result.isSome() implies buf.nth(result.unwrap()) = target
is
    // implementation
qed;
```
**D-index**
d53 — `require`/`ensure` vs manual `abort` checks
d6 — anonymous `&[Buffer[Int32]]` in the sketch.

---

## Summary of Changes at a Glance

Authoritative `D` numbering lives in `kyokaiplan.md` §3.3 and the appendix index; the table below is a shorthand crosswalk, not a second spec. In the plan appendix, **D7a** (UFCS) and **D7b** (auto-reborrow) are documented as sub-points under **`### D7:`**, not as separate top-level `### D7a:` headers. For the meaning of **kyokaiplan-verified** (including the **167** resolved canonical `D` / `d` tokens), see [Normative basis (kyokaiplan.md)](#normative-basis-kyokaiplanmd) above.

| Category | Austral | Kyokai |
|----------|---------|--------|
| Comments | `--` | `//`, `///`, `//!` |
| Terminators | `end if;`, `end for;`, `end;` | `fi;`, `od;`, `esac;`, `qed;`, `build;`, `spec;`, `drop;`, `seal;`, `mon;`, `pick;`, `join;`, `audit;` |
| Not-equal | `/=` | `!=` |
| Bitwise ops | `&`, `\|` | `band`, `bor`, `bxor`, `bnot`, `shl`, `shr` |
| File extensions | `.aui`, `.aum` | `.kyo`, `.kai` |
| Regions | Explicit `generic [R: Region]` | Inferred |
| Re-borrow | Manual `&~x` | Auto-reborrow |
| Function calls | VSO: `f(x, args)` | SVO: `x.f(args)` via UFCS |
| Integer literals | `(0 : Index)` | `0` (inferred from context) |
| Unit return | `return nil;` required | Implicit at end of `: Unit` functions |
| Cleanup | Manual destroy chains | `defer` and `errdefer` |
| Error unwrap | Full `case` expression | `let...else`, `or return` |
| Contracts | Manual assertions | `require` / `ensure` in signature |
| Concurrency | None | `taskgroup`/`join` (**D252**), channels, atomics, `select`/`pick` |
| Build system | None | `kyokai build`, `kyokai.toml` |
| Package manager | None | Git-based with lockfile |
| Testing | None | First-class `test` blocks, property testing, fuzzing |
| Formatter | None | `kyokai fmt` |
| LSP | None | Official implementation |
| Comptime | None | `comptime` keyword, `@embedFile` |
| Debugger | C-level only | Source-level DWARF via LLVM |
| Stdlib | 10 basic modules | Batteries-included systems library |

---

## Appendix: D204 through D263 decision titles (synced with `kyokaidecided.md`)

Rows are the **appendix short titles** from the current plan (≈ lines 10950–11010). Full prose for each decision lives under **`### Dnnn:`** in `kyokaiplan.md` and in **§3.3**.

| ID | Appendix title |
|----|----------------|
| **D204** | String type representation |
| **D205** | Exhaustiveness and pattern depth |
| **D206** | Wildcard on linear payloads |
| **D207** | `defer` control flow ban |
| **D208** | Exit-path matrix |
| **D209** | Generic inference direction |
| **D210** | Integer mixing rules |
| **D211** | Effect system boundary |
| **D212** | Capability I/O thread safety |
| **D213** | UFCS on rvalue temporaries |
| **D214** | Import name collision |
| **D215** | Constant dependency ordering |
| **D216** | Orphan rule diagnostics |
| **D217** | Recursive type layout rules |
| **D218** | Documentation generator |
| **D219** | Code coverage |
| **D220** | Fuzzing / property testing |
| **D221** | Package discovery index |
| **D222** | Linting architecture |
| **D223** | Package versioning / SemVer |
| **D224** | Build-time code generation |
| **D225** | CI/CD integration |
| **D226** | Web playground |
| **D227** | `defer` + `or return` interaction |
| **D228** | Defined lowering contract |
| **D229** | Stdlib correctness admission |
| **D230** | Transitional FFI policy |
| **D231** | Cryptography policy |
| **D232** | Numerical accuracy tiers |
| **D233** | Debug expression purity |
| **D234** | Explicit event loop/reactor boundary |
| **D235** | Thread resource limits / spawn failure |
| **D236** | Broker pattern standardization |
| **D237** | Blocking syscall cancellation |
| **D238** | Desugaring order |
| **D239** | Tautology formal check |
| **D240** | Auto-reborrow tests |
| **D241** | Formal semantics sketch |
| **D242** | FFI ownership transfer |
| **D242a** | ABI boundaries for sum types |
| **D243** | Edition compatibility / stdlib versioning |
| **D244** | Package yank policy |
| **D245** | Unsafe audit trail |
| **D246** | Defer capture and linear ordering |
| **D247** | Memory model and atomics |
| **D248** | Task-boundary transfer for capabilities |
| **D249** | Iterator linear consumption |
| **D250** | Default allocator strategy |
| **D251** | Allocator storage/erasure |
| **D252** | Structured concurrency join semantics |
| **D253** | Fault isolation |
| **D254** | UFCS namespace resolution / ADL |
| **D255** | Capability forgery |
| **D256** | Signal handling policy |
| **D257** | Volatile / memory-mapped I/O |
| **D258** | Multi-channel select / poll |
| **D259** | Standard error interface |
| **D260** | Endianness control |
| **D261** | Numeric literal suffixes |
| **D262** | Stack size / guard page contract |
| **D263** | Compiler/runtime licensing |

---