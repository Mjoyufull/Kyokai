# Kyokai Decided Shape

**Kyokai** (境界, "boundary") — a systems programming language forked from Austral.

Linear types enforcing boundaries on resource ownership. Every resource explicitly created, explicitly used, explicitly destroyed. Nothing hidden. Nothing implicit. Nothing undefined.

This document is the public readable extraction of the shape Kyokai has already accepted. It is not the full normative spec. The eventual normative home is `kyokaispec/`; new public D-points and live proposal shape belong in `Kyokaishape.md`.

---

## Table of Contents

0. [Maturity State Tracker](#maturity-state-tracker)
1. [Austral As It Stands](#1-austral-as-it-stands)
  - 1.1–1.5: Core Language, Compiler, Stdlib, Unfinished Work, Pain Points
  - 1.6: [Syntax Crimes Catalog](#16-syntax-crimes-catalog--the-full-evidence)
2. [What's Missing — The Full Gap Analysis](#2-whats-missing--the-full-gap-analysis)
3. [The Kyokai Philosophy](#3-the-kyokai-philosophy)
  - 3.1–3.3: Unbreakable Rules, Kyokai Additions, Changes from Austral
  - 3.4: [Readability Research — What Science Says](#34-readability-research--what-science-says)
  - 3.5: [Naming Convention Principles](#35-naming-convention-principles)
4. [Work Items — Prioritized by Hardness and Severity](#4-work-items--prioritized-by-hardness-and-severity)

For active public proposals and new D-points, see `Kyokaishape.md`.

## Maturity State Tracker

This tracker records how far accepted Kyokai shape has moved toward normative spec text, conformance tests, and implementation. It is not a roadmap and it does not reopen decided semantics. `phase.md` remains the implementation/proof ordering document.

Use the highest honest state that is currently true:

| State | Meaning | Evidence To Record Here |
| --- | --- | --- |
| `SHAPE_DECIDED` | The design point is decided in public accepted-shape docs. | D-point IDs, accepted-shape section, and any public thread/PR link if applicable. |
| `SPEC_EXTRACTED` | The rule has a normative home outside the planning document. | `kyokaispec/` path and short note on the covered rule scope. |
| `CALCULUS_DRAFTED` | The behavior is represented in `lambda_K` scope or explicitly excluded from it. | Calculus document path and whether the feature is included or explicitly out of scope. |
| `CALCULUS_PROVEN_PAPER` | The sequential core proof obligation is discharged at paper level. | Proof document path and theorem/scope name. |
| `PARSER_ACCEPTED` | Surface syntax is parsed into AST nodes with source spans. | Parser implementation path and positive/negative parser test path. |
| `ELABORATED_CORE` | Surface constructs lower through the D238 ordered pipeline. | Elaboration/lowering pass path and tests showing implicit completions/sugar exposure. |
| `CHECKED` | Name, type, borrow, linearity, capability, contract, and unsafe checks enforce the spec. | Checker implementation path and negative diagnostic/conformance test path. |
| `LOWERED_SAFE` | Backend output implements the checked semantics without backend UB. | Backend/runtime path plus UB-sensitive generated-code/runtime tests. |
| `CONFORMANCE_BACKED` | Behavior has executable tests and diagnostic goldens. | Conformance test path and diagnostic golden path if relevant. |
| `STDLIB_ADMITTED` | A stdlib API has its contract, edge cases, tests, and implementation policy. | Stdlib module path, admission/contract note, and edge-case test path. |
| `BOOTSTRAP_RELEASED` | The OCaml/Austral-derived compiler can compile practical Kyokai programs. | Release/build artifact path and workload/test path. |
| `SELF_HOSTING` | Important compiler components are written in Kyokai and built by Kyokai. | Self-hosting component path and bootstrap build instructions/test path. |
| `MECHANIZED_PROVEN` | The relevant core theorem is machine-checked. | Proof assistant artifact path and CI/build command. |

Current high-level tracker:

| Area | Key decisions | Current maturity | Spec home | Conformance | Implementation | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Project identity, philosophy, and license boundary | D1, D5, D87, D143/D241, D263 | `SPEC_EXTRACTED` | `kyokaispec/src/language/00-introduction.md`, `kyokaispec/src/language/01-goals-and-non-goals.md`, `kyokaispec/src/rationale/00-rationale-index.md`, `kyokaispec/src/rationale/01-language-design.md`, `kyokaispec/src/appendices/a-license.md`, `kyokaispec/src/appendices/c-austral-differences.md`, and `kyokaispec/src/appendices/d-formalization-roadmap.md` for identity, self-contained fork scope, no-language-UB policy, inherited goals, attribution, rationale boundaries, license/provenance, Austral-difference indexing, inherited-file retirement status, and proof-roadmap scope | planned | partial docs/tooling metadata | Rationale and appendix extraction is present; implementation/proof maturity still requires conformance tests and the actual `lambda_K` paper proof. |
| Core syntax and declarations | D8-D18, D23-D24, D35-D43, D46, D47, D50, D54-D59, D61, D71-D72, D75-D76, D78, D98, D106, D109, D128, D135/D261, D138, D179, D205-D206, D210, D213-D214 | `SPEC_EXTRACTED` | `kyokaispec/src/language/02-lexical-syntax.md`, `kyokaispec/src/language/03-grammar.md`, `kyokaispec/src/language/04-modules-and-visibility.md`, `kyokaispec/src/language/05-declarations.md`, `kyokaispec/src/language/08-patterns.md`, `kyokaispec/src/language/09-expressions-and-evaluation.md`, `kyokaispec/src/language/10-statements-and-control-flow.md`, `kyokaispec/src/language/11-linearity-borrowing-and-regions.md`, `kyokaispec/src/language/18-built-ins.md`, `kyokaispec/src/language/19-examples.md` for lexical, grammar, module/import/visibility, declarations, patterns, expressions, evaluation, statements, control-flow, ownership-visible pattern/control-flow joins, protected built-ins, and current syntax examples | planned | inherited Austral parser only | Later unsafe and broader toolchain chapters still need extraction. |
| Type system, ownership, borrows, and implicit completions | D6, D7a/D7b, D14, D24, D30/D30a, D34, D46, D72, D87, D104, D159, D188, D190, D192-D195, D213, D238-D240 | `SPEC_EXTRACTED` | `kyokaispec/src/language/06-type-system.md`, `kyokaispec/src/language/07-generics-and-typeclasses.md`, `kyokaispec/src/language/08-patterns.md`, `kyokaispec/src/language/09-expressions-and-evaluation.md`, `kyokaispec/src/language/11-linearity-borrowing-and-regions.md`, `kyokaispec/src/language/12-implicit-completions-and-elaboration.md`, `kyokaispec/src/language/18-built-ins.md`, `kyokaispec/src/language/19-examples.md` for universes, nominal identity, generics, typeclasses, inference, pattern movement, temporaries, checker states, regions, borrows, movement, built-in type families, and example applications | planned | inherited Austral checker only | Implementation still needs conformance tests and compiler work before checker maturity can move beyond spec extraction. |
| Error handling, contracts, defer, panic/TPOE | D2, D2a/D2b, D15/D15a, D24, D53, D58, D84, D89, D111, D119, D121-D122, D124-D125, D129, D140, D142, D207, D233, D246, D253, D259-D260 | `SPEC_EXTRACTED` | `kyokaispec/src/language/03-grammar.md`, `kyokaispec/src/language/05-declarations.md`, `kyokaispec/src/language/08-patterns.md`, `kyokaispec/src/language/10-statements-and-control-flow.md`, `kyokaispec/src/language/11-linearity-borrowing-and-regions.md`, `kyokaispec/src/language/12-implicit-completions-and-elaboration.md`, `kyokaispec/src/language/13-contracts-and-runtime-failure.md`, `kyokaispec/src/language/18-built-ins.md`, `kyokaispec/src/language/19-examples.md` for contract syntax, function contract declaration placement, result/old scoping, fallible patterns, `Result`/`Optional` built-ins, `or` lowering, defer/errdefer path behavior, deferred checker states, panic/todo/unreachable behavior, TPOE, runtime-fatal, diagnostics, and examples | planned | inherited Austral behavior only | Implementation still needs conformance tests and runtime/compiler work before maturity can move beyond spec extraction. |
| Capabilities and authority | D20/D20a/D20b, D48/D162, D67, D85, D211-D212, D245, D248, D255-D256 | `SPEC_EXTRACTED` | `kyokaispec/src/language/04-modules-and-visibility.md`, `kyokaispec/src/language/05-declarations.md`, `kyokaispec/src/language/06-type-system.md`, `kyokaispec/src/language/14-capabilities-and-authority.md`, `kyokaispec/src/language/18-built-ins.md`, `kyokaispec/src/language/19-examples.md` for sealed capability declarations, root authority, acquire/derive/split/surrender, borrowing, task transfer, unsafe authority, FFI authority flow, tests/tools/plugins, dynamic loading, raw I/O broker patterns, startup built-ins, and capability examples | planned | inherited Austral capability shape only | Stdlib authority APIs and toolchain audit surfaces still need extraction in their later chapters. |
| FFI, unsafe, ABI, layout, and backend safety | D20/D242/D242a, D31, D42, D73, D80, D89, D113a/D113b, D139, D199, D228, D245, D250-D251, D257 | `SPEC_EXTRACTED` | `kyokaispec/src/language/05-declarations.md`, `kyokaispec/src/language/06-type-system.md`, `kyokaispec/src/language/09-expressions-and-evaluation.md`, `kyokaispec/src/language/11-linearity-borrowing-and-regions.md`, `kyokaispec/src/language/12-implicit-completions-and-elaboration.md`, `kyokaispec/src/language/13-contracts-and-runtime-failure.md`, `kyokaispec/src/language/14-capabilities-and-authority.md`, `kyokaispec/src/language/16-unsafe-ffi-and-abi.md`, `kyokaispec/src/language/17-memory-layout-and-backend-contract.md` for unsafe modules, unsafe contracts, raw FFI, explicit C ABI surface, callbacks, unwinding, dynamic loading/plugins, volatile/MMIO, inline assembly, layout classes, move/result placement, generated-C safety, LLVM lowering, debug/source mapping, and backend failure rules | planned | inherited C backend only | Implementation still needs safe Kyokai boundary wrappers, full ABI/backend tests, and UB-closure generated-code tests before maturity can move beyond spec extraction. |
| Concurrency, atomics, channels, poller, and signals | D3, D3a/D3b, D90-D95, D100-D101, D141, D146, D156, D164, D168, D183-D184, D212, D234-D237, D247-D248, D252, D256, D258 | `SPEC_EXTRACTED` | `kyokaispec/src/language/03-grammar.md`, `kyokaispec/src/language/11-linearity-borrowing-and-regions.md`, `kyokaispec/src/language/14-capabilities-and-authority.md`, `kyokaispec/src/language/15-concurrency.md`, `kyokaispec/src/language/18-built-ins.md`, `kyokaispec/src/language/19-examples.md` for task groups, spawn/capture/failure, SPSC channels, select, cancellation/deadlines, poller/event loops, signals, atomics, locks, task transfer, broker patterns, happens-before, recognized concurrency built-ins, and spawn examples | planned | planned | Implementation still needs conformance tests, runtime primitives, and backend atomic validation before maturity can move beyond spec extraction. |
| Standard library foundation and admission policy | D40, D40a, D44, D64, D67, D74, D77, D85, D102, D117, D146, D152, D171, D201, D220, D229-D232, D250-D251, D259-D260, D263 | `SPEC_EXTRACTED` | `kyokaispec/src/stdlib/00-stdlib-overview.md`, `kyokaispec/src/stdlib/01-admission-contracts.md`, `kyokaispec/src/stdlib/02-core-result-optional-display-error.md`, `kyokaispec/src/stdlib/03-allocators-and-memory-containers.md`, `kyokaispec/src/stdlib/04-text-bytes-paths-and-strings.md`, `kyokaispec/src/stdlib/05-collections.md`, `kyokaispec/src/stdlib/06-iterators-and-generators.md`, `kyokaispec/src/stdlib/07-math-and-numerics.md`, `kyokaispec/src/stdlib/08-io-files-env-process-time-random.md`, `kyokaispec/src/stdlib/09-concurrency-primitives.md`, `kyokaispec/src/stdlib/10-crypto-policy.md`, and `kyokaispec/src/stdlib/11-transitional-ffi-tracking.md` for stdlib admission, contract fields, pure-Kyokai implementation policy, transitional FFI tracking, allocators, containers, text/bytes/paths, collections, iterators/generators, numerics/math accuracy, capability-gated external-world APIs, concurrency primitives, crypto policy, and core formatting/error protocols; language chapters remain the syntax and built-in source for the same rules | planned | inherited Austral stdlib only | Implementation still needs admitted Kyokai stdlib modules, conformance tests, audit metadata, and transitional FFI records before maturity can move beyond spec extraction. |
| Toolchain, package manager, diagnostics, formatter, docs, releases | D25-D29, D31, D51, D78-D80, D83, D86, D105, D137, D144, D148-D151a, D155, D157, D200, D218, D220-D226, D243-D245, D264-D270 | `SPEC_EXTRACTED` | `kyokaispec/src/toolchain/00-toolchain-overview.md`, `kyokaispec/src/toolchain/01-manifest-package-workspace.md`, `kyokaispec/src/toolchain/02-module-resolution-and-koi.md`, `kyokaispec/src/toolchain/03-cli.md`, `kyokaispec/src/toolchain/04-build-profiles-targets-linking.md`, `kyokaispec/src/toolchain/05-diagnostics.md`, `kyokaispec/src/toolchain/06-formatter.md`, `kyokaispec/src/toolchain/07-testing-coverage-bench.md`, `kyokaispec/src/toolchain/08-docs-lsp-audit.md`, `kyokaispec/src/toolchain/09-reproducibility-incremental-builds.md`, `kyokaispec/src/toolchain/10-package-index-semver-releases-ci.md`, `kyokaispec/src/toolchain/11-build-generation-and-playground.md` for CLI, project shape, artifacts, diagnostics, formatting, tests, docs, LSP, audit, reproducibility, package ecosystem, releases, generation, REPL/eval, playground contracts, build output/cache layout, concrete `.koi` artifact format, project creation, diagnostic explanation/fixing, toolchain health, package inspection/offline workflows, and property/fuzz daily-use controls | planned | inherited Austral CLI only | Implementation still needs compiler/package-manager/tooling work and conformance tests before maturity can move beyond spec extraction. |
| Formal calculus and proof | D143/D241 | `SPEC_EXTRACTED` | `kyokaispec/src/appendices/d-formalization-roadmap.md` for proof scope, paper-proof obligations, exclusions, milestones, research-note status, and later mechanization plan | not applicable | research note exists | `lambda_K` paper proof is still not drafted; maturity has only moved from accepted shape to normative roadmap extraction. |

Update this tracker whenever a row moves to any higher maturity state from the shared `phase.md` maturity vocabulary. Do not mark maturity based on intent; mark only completed work with a path or concrete artifact.

---

## 1. Austral As It Stands

This section profiles the Austral base Kyokai is forked from. The main evidence is the inherited spec material now living under `kyokaispec/`, the current compiler source in `lib/`, Borretti's blog posts, and the upstream Austral compiler/tests where inherited behavior still matters.

### 1.1 Core Language Rules

Austral is defined by a small set of invariants that Kyokai **must preserve**:

**Type Universe System** (spec: `4.types.md`, lines 6–75):

- Every type belongs to exactly one of two **universes**: `Free` or `Linear`.
- `Free` types can be used any number of times (integers, booleans, records containing only Free types).
- `Linear` types must be used **exactly once** — not zero times, not two times. This is the foundation of resource safety.
- The `Auto` classifier lets generic types defer universe selection: `Box[Int32]` is `Free`, `Box[SomeLinearType]` is `Linear`.
- Linearity is **viral**: if a record contains a `Linear` field, the record itself becomes `Linear`. You cannot sneak a linear type into a free container.

**Reference: spec `4.types.md` lines 17–24**: "A type T is affine if: 1. It contains another affine type (structurally affine). 2. It is declared to be an affine type (declared affine)."

**Module System** (spec: `3.modules.md`, lines 1–98):

- Every module has two files: an **interface** (`.aui` in Austral, `**.kyo`** in Kyokai) and a **body** (`.aum` in Austral, `**.kai`** in Kyokai).
- The interface declares public API. The body provides implementations plus private declarations.
- Types can be **opaque** (importable but not constructible from outside), **public** (fully visible), or **private** (body-only).
- Modules that use FFI or unsafe operations must be marked with `pragma Unsafe_Module`.

**Reference: spec `3.modules.md` lines 6–11**: "The interface contains declarations that are importable by other modules, as well as an optional private section of declarations that are available within the module but not importable."

**Linearity Rules** (blog: `how-australs-linear-type-checker-works.md`, lines 88–571):

The linearity checker is ~600 lines of OCaml doing abstract interpretation. It enforces 11 rules:


| Rule | What It Enforces                                                            |
| ---- | --------------------------------------------------------------------------- |
| 1    | Variables of a linear type cannot appear zero times in their scope          |
| 2    | Values of a linear type cannot be silently discarded                        |
| 3    | Linear variables outside an `if` must be consumed in ALL branches or NONE   |
| 4    | Same as Rule 3 but for `case` statements                                    |
| 5    | Linear variables defined outside a loop cannot be consumed inside the loop  |
| 6    | Accessing a `Free` field of a `Linear` record doesn't count as consuming it |
| 7    | Every linear variable must be consumed before a `return` statement          |
| 8    | Borrowing cannot happen after consumption                                   |
| 9    | Same variable cannot be mutably borrowed multiple times in one expression   |
| 10   | A linear variable cannot be consumed inside a `borrow` that borrows it      |
| 11   | Cannot mutably borrow what's already mutably borrowed                       |


**Reference: blog `how-australs-linear-type-checker-works.md` lines 520–573**: The algorithm uses a state table mapping variable names to `(loop_depth, var_state)` where `var_state` is one of `Unconsumed | BorrowedRead | BorrowedWrite | Consumed`.

**Borrowing** (blog: `introducing-austral.md` lines 767–796):

- Two reference types: `&[T, R]` (immutable) and `&![T, R]` (mutable), parameterized by type and region.
- Shorthand syntax: `&x` and `&!x` create anonymous-region references that cannot escape the call site.
- General syntax: `borrow x as ref in R do ... end` gives the region a name.
- Regions exist only in their lexical scope — the type of an escaped reference literally cannot be written.

**Reference: blog `how-australs-linear-type-checker-works.md` lines 430–431**: "And the way Austral ensures that references do not outlive the thing they reference is very simple. There's no need to do sophisticated control flow analysis. There's no way to write the type of a reference outside the scope where that reference is defined."

**Capability-Based Security** (blog: `how-capabilities-work-austral.md`, lines 9–167):

- Capabilities are linear types representing unforgeable permission tokens.
- `RootCapability` is the base: only available as the first argument of the entrypoint. Cannot be created in userspace.
- Each capability type defines `acquire(root: &![RootCapability, R]): MyCapability` and `surrender(cap: MyCapability): Unit`.
- Because capabilities are linear, they cannot be duplicated. Because there's no global state, they can't be stashed.
- Because of opaque types + strict module encapsulation, capabilities cannot be forged.

**No Hidden Control Flow** (blog: `introducing-austral.md` lines 152–193 — "Anti-Features"):

- No garbage collection, no destructors, no exceptions, no stack unwinding.
- No implicit function calls, no implicit type conversions.
- No global state, no runtime reflection, no macros, no annotations.
- No type inference (types flow in one direction only).
- No operator precedence (all binary expressions deeper than one level must be parenthesized).
- No variable shadowing, no uninitialized variables, no pre/post increment.

**Error Handling** (spec rationale: `rationale/2.error-handling.md`, lines 1–532):

Austral categorizes errors following Sutter/Midori into 5 categories:

1. **Physical failure** — nothing can be done.
2. **Abstract machine corruption** (stack overflow) — terminate.
3. **Contract violations** (overflow, bounds, assertions) — **terminate program** (TPOE). This is the critical decision.
4. **Allocation failure** — return `Optional` type. Programmer must handle explicitly.
5. **Error conditions** ("file not found") — values + control flow.

**Why TPOE over exceptions**: The spec rationale dedicates 20+ pages to proving that RAII + destructors + unwinding is fundamentally flawed:

- **Double-throw problem** (`rationale/2.error-handling.md` lines 294–337): What happens when a destructor throws? C++ aborts. Rust ignores errors in `Drop`. Both are unsatisfactory.
- **Libraries can't rely on destructors** (`rationale/2.error-handling.md` lines 409–438): `panic=abort` vs `panic=unwind` is a compile-time toggle that the *application* decides, not the library.
- **Hidden control flow**: Destructor calls are inserted by the compiler invisibly. This violates Austral's visibility principle.
- **Affine types cannot force cleanup** (`rationale/2.error-handling.md` lines 326–337): With affine types, dropping without consuming triggers the destructor. The compiler won't force you to call `close()` — it'll just silently insert destructor calls. This is exactly the kind of hidden behavior Austral rejects.

**Why linear types over affine** (`rationale/3.resource-types.md` lines 1–243):

- Linear types **force** consumption. You MUST call `closeFile(f)`. The compiler won't silently clean up after you.
- This means every resource lifecycle is visible in source code. No surprise `Drop` calls.
- The tradeoff: you have to type more (thread values through, use borrowing). But the code is honest.

**Reference: `rationale/3.resource-types.md` lines 223–230**: "Austral takes the approach that a language should be simple enough that it can be understood entirely by a single person reading the specification. Consequently, a programmer should be able to read a brief set of linearity checker rules, and afterwards be able to write code without fighting the system."



### 2.3 Standard Library Gaps

This is the largest gap. Organized by what can be implemented **in pure Kyokai** (no unsafe, no FFI) vs what **requires unsafe internals**.

#### Pure Kyokai — No Unsafe Required

These can be implemented using only existing language features. The compiler already has everything needed.


| Module                            | What It Provides                                                                                                                                                                                  | Hardness                                                                                                                                                           |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Kyokai.Math.Int`                 | `abs()`, `min()`, `max()`, `clamp()`, `gcd()`, `lcm()`, `isPowerOfTwo()`, `nextPowerOfTwo()`, `countLeadingZeros()`, `countTrailingZeros()`, `popcount()`, wrapping/saturating/checked arithmetic | 2/10 — pure integer arithmetic                                                                                                                                     |
| `Kyokai.Math.Float`               | `abs()`, `min()`, `max()`, `clamp()`, `isNan()`, `isInf()`, `isFinite()`, `floor()`, `ceil()`, `round()`, `trunc()` — all implementable via bit manipulation on IEEE 754                          | 4/10 — need to understand IEEE 754 bit layout                                                                                                                      |
| `Kyokai.Math.Trig`                | `sin()`, `cos()`, `tan()`, `asin()`, `acos()`, `atan()`, `atan2()` — Taylor/Chebyshev polynomial approximations. NOT wrapping libm. Pure computation.                                             | 6/10 — needs careful numerical analysis for precision. Reference: FDLIBM, Cephes, musl libm source. These are pure math — no syscalls, no state, just polynomials. |
| `Kyokai.Math.Exp`                 | `exp()`, `ln()`, `log2()`, `log10()`, `pow()`, `sqrt()` via Newton-Raphson and range reduction                                                                                                    | 6/10 — same as trig, pure computation                                                                                                                              |
| `Kyokai.Compare`                  | `PartialEquality`, `Equality`, `PartialOrder`, `TotalOrder` + instances for ALL builtin types. Currently only interfaces exist.                                                                   | 2/10 — mechanical                                                                                                                                                  |
| `Kyokai.Compare.Span`             | `spanEquals()`, `spanCompare()`, `spanStartsWith()`, `spanEndsWith()`, `spanContains()`, `spanFind()`, `spanFindLast()` — the kind of byte comparison code `bfetchaust` had to carry locally        | 3/10 — pure byte comparison loops                                                                                                                                  |
| `Kyokai.String.Ops`               | String comparison, concatenation, slicing, searching, trimming, case conversion, all built on top of `Span` and `Buffer`                                                                          | 4/10 — builds on Compare.Span                                                                                                                                      |
| `Kyokai.String.Format`            | Integer-to-string, float-to-string formatting. What bfetchaust's `appendIndexDecimal` does but generalized.                                                                                       | 5/10 — number formatting is surprisingly tricky (Dragonbox/Ryū for floats)                                                                                         |
| `Kyokai.Collections.SortedBuffer` | In-place sorting of `Buffer` contents. Quicksort, mergesort, insertion sort. Pure algorithms on existing `Buffer`.                                                                                | 3/10                                                                                                                                                               |
| `Kyokai.Collections.RingBuffer`   | Fixed-capacity ring buffer. Useful for I/O buffering.                                                                                                                                             | 3/10                                                                                                                                                               |
| `Kyokai.Collections.Deque`        | Double-ended queue built on ring buffer                                                                                                                                                           | 3/10                                                                                                                                                               |
| `Kyokai.Bits`                     | Bitwise utilities: `rotateLeft()`, `rotateRight()`, `byteSwap()`, `setBit()`, `clearBit()`, `testBit()`                                                                                           | 1/10                                                                                                                                                               |
| `Kyokai.Ascii`                    | `isDigit()`, `isAlpha()`, `isAlnum()`, `isSpace()`, `isUpper()`, `isLower()`, `toUpper()`, `toLower()`                                                                                            | 1/10                                                                                                                                                               |
| `Kyokai.Args`                     | Command-line argument parsing. Uses existing `argumentCount()` and `nthArgument()` builtins. Flag/option/positional parsing.                                                                      | 4/10                                                                                                                                                               |
| `Kyokai.Result`                   | `Result[T, E]` union type. `ok()`, `err()`, `isOk()`, `isErr()`, `unwrapOr()`, `map()`, `flatMap()`. This is what bfetchaust lacks for error propagation.                                         | 2/10                                                                                                                                                               |
| `Kyokai.Optional`                 | Enhanced `Optional[T]` with `map()`, `flatMap()`, `unwrapOr()`, `isSome()`, `isNone()`                                                                                                            | 2/10                                                                                                                                                               |


**Why not wrap libm?** Functions like `sin()`, `cos()`, and `sqrt()` are pure mathematical computations. They take a number and return a number. No syscalls, no state, no side effects. There is no good language-design reason to call through C FFI for what is fundamentally polynomial evaluation and range reduction. FDLIBM (Freely Distributable LIBM) and musl's libm provide reference implementations that are well-tested, well-documented, and can be translated to Kyokai directly. The result is zero unsafe code for the core math library.

**Reference for pure math implementations**:

- FDLIBM source: `https://www.netlib.org/fdlibm/` — Sun Microsystems' reference math library, public domain.
- musl libc `src/math/` — each function is a standalone file, BSD-licensed.
- The Cephes mathematical library — Stephen Moshier's comprehensive implementation.

#### Requires Unsafe Internals — Thin Trust Boundary

These need C FFI or pointer arithmetic internally, but expose a safe linear API.


| Module                                                      | What It Provides                                                                                                                                             | Hardness             | Why Unsafe                                           |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------- | ---------------------------------------------------- |
| `Kyokai.IO.File`                                            | File I/O: `open()`, `read()`, `write()`, `close()`, `seek()`, `stat()`. This is what bfetchaust's `BfetchAust.Posix` does.                                   | 5/10                 | Syscalls (`open`, `read`, `write`, `close`, `fstat`) |
| `Kyokai.IO.Terminal`                                        | Terminal I/O with capability-secure API.                                                                                                                     | 4/10                 | `isatty()`, terminal ioctl                           |
| `Kyokai.IO.Stdin` / `Kyokai.IO.Stdout` / `Kyokai.IO.Stderr` | Standard streams with capability                                                                                                                             | 3/10                 | File descriptors 0/1/2                               |
| `Kyokai.IO.Dir`                                             | Directory operations: `openDir()`, `readDir()`, `closeDir()`, `mkdir()`, `rmdir()`                                                                           | 5/10                 | `opendir`, `readdir`, `closedir` syscalls            |
| `Kyokai.IO.Path`                                            | Path manipulation — this CAN be mostly pure (string operations on paths) with syscall for `realpath`/`stat`                                                  | 4/10                 | `realpath` needs syscall, rest is pure               |
| `Kyokai.Memory.Allocator`                                   | Allocator abstraction. Currently everything uses the default allocator. This would let users provide custom allocators to containers.                        | 7/10                 | `malloc`/`free`/`realloc`                            |
| `Kyokai.Env`                                                | Environment variable access: `getEnv()`, `setEnv()`                                                                                                          | 2/10                 | `getenv` syscall                                     |
| `Kyokai.Process`                                            | Process spawning: `fork()`, `exec()`, `waitpid()`, `pipe()`                                                                                                  | 7/10                 | All POSIX process syscalls                           |
| `Kyokai.Time`                                               | Monotonic and wall clock time: `now()`, `elapsed()`, `sleep()`                                                                                               | 4/10                 | `clock_gettime` syscall                              |
| `Kyokai.Random`                                             | Cryptographically secure RNG via `getrandom()` syscall, plus PRNG (xoshiro256) for non-crypto use. The PRNG itself is pure — only seeding needs the syscall. | 4/10                 | `getrandom` for seeding                              |
| `Kyokai.Net.Socket`                                         | TCP/UDP sockets: `socket()`, `bind()`, `listen()`, `accept()`, `connect()`, `send()`, `recv()`, `close()`                                                    | 8/10                 | All networking syscalls                              |
| `Kyokai.Net.DNS`                                            | DNS resolution: `getaddrinfo()`                                                                                                                              | 6/10                 | `getaddrinfo`                                        |
| `Kyokai.Collections.HashMap`                                | Hash map with linear keys/values. Needs internal pointer arithmetic for the hash table.                                                                      | 7/10                 | Internal array management, hashing                   |
| `Kyokai.Collections.HashSet`                                | Hash set built on HashMap                                                                                                                                    | 3/10 (after HashMap) | Delegates to HashMap                                 |


**c-ward reference** (`c-ward/`): The c-ward project demonstrates which POSIX functions can be implemented safely. Key finding: most `string.h` functions (`strlen`, `memcpy`, `memcmp`, `memset`) can be implemented purely. Most `stdio.h` buffered I/O can be implemented safely on top of raw `read`/`write`. The unsafe boundary is narrower than expected — it's primarily the syscall interface itself.

---

## 3. The Kyokai Philosophy

Extending Austral while respecting its vision. These are the Kyokai invariants that go beyond Austral's.

### 3.1 Unbreakable Rules (carried from Austral)

1. **Zero undefined behavior** — this is the intended language contract. The spec aims to define behavior for every program the compiler accepts, and D143 commits the project to formalize that claim rather than leaving it as prose alone.
2. **No hidden control flow** — if it's not in source, it's not happening. No invisible destructors, no implicit copies, no hidden allocations.
3. **Compile-time safety checking with proof-oriented intent** — resource safety is enforced statically rather than by hidden runtime machinery. D143 commits the project to a paper proof for the sequential core before `v1.0`, followed by a later mechanized proof after self-hosting.
4. **Everything consumed** — every linear value must be explicitly consumed. The compiler will never silently clean up after you.
5. **No hidden or semantically significant implicitness** — the compiler may insert or complete an omitted operation only when the D87 tautology rule is satisfied; otherwise the programmer must write the operation explicitly.

### 3.2 Kyokai Additions

1. **Minimize unsafe to the absolute physical minimum** — if a function can be implemented in pure Kyokai, it WILL be. Math, string ops, sorting, data structures — no wrapping C when the algorithm is pure computation. Unsafe exists only at the syscall boundary and nowhere else.
2. **100% correct or it doesn't ship** — every module is verified against its specification. Every function has documented behavior for all inputs. Edge cases are handled, not hand-waved.
3. **No compromise on correctness for ergonomics** — we'll add syntax sugar (like `defer`) only when it preserves full visibility. If a feature would hide behavior, it doesn't get added.
4. **The Implicit-Operation Rule (Tautology Rule)** — Kyokai's philosophy is NOT "no implicit anything." It IS "no ambiguous, effectful, or semantically surprising implicitness." Specifically: **the compiler may insert or complete an omitted operation implicitly if and only if (1) it is uniquely determined by the static types and surrounding context, (2) all alternative programs are statically ill-typed, and (3) the implicit completion does not introduce any new control flow, allocation, or side effects beyond what the explicit program already implies.** This rule governs D7b (auto-reborrow), D8 (implicit Unit return), D12 (literal inference), D15 (`or return` sugar), D34 (auto-deref for fields), and D46 (let inference). If a proposed feature cannot satisfy all three conditions, it is not eligible for implicit treatment. **[STAGE: DECIDED_CORE_SEMANTICS | D87 → formal tautology rule for effect-neutral implicit completion]**

### 3.3 What Kyokai Changes From Austral

These are deliberate departures:

- `**Kyokai.`* namespace** instead of `Standard.`* — clean break, clearly different project. "Boundary" (境界) is the project identity — every type, resource, and capability has explicit boundaries. The namespace reflects that. **[STAGE: DECIDED_CORE_SEMANTICS | D1 → Option A]**
- **Richer stdlib** — Austral's stdlib is intentionally minimal because Borretti was focused on the compiler. Kyokai's stdlib is the goal.
- **Kyokai uses copyleft for the toolchain and a runtime exception for target-linked runtime/stdlib code** — the compiler stays reciprocal, while programs built with Kyokai are not forced to adopt the compiler's GPL license merely because they link the Kyokai runtime, standard library, or compiler-emitted helper code.
**Rules**:
  1. Kyokai-owned compiler and toolchain source files are licensed under `GPL-3.0-or-later` unless a file has an explicitly stated compatible license notice.
  2. "Compiler and toolchain" includes the frontend, parser, resolver, type checker, linearity checker, capability checker, optimizer, C backend, future LLVM backend, package manager, formatter, LSP, documentation generator, test runner, conformance harness, and ordinary compiler support tools.
  3. Kyokai-owned runtime library, standard library, startup code, compiler support library, target-side panic/TPOE helpers, allocation/runtime shims, and compiler-emitted target helper code that may be linked, copied, embedded, or otherwise combined into user target programs are licensed under `GPL-3.0-or-later WITH GCC-exception-3.1`.
  4. Source files covered by the runtime exception must carry an SPDX notice of `SPDX-License-Identifier: GPL-3.0-or-later WITH GCC-exception-3.1` or an equivalent explicit notice naming GPLv3-or-later plus the GCC Runtime Library Exception 3.1.
  5. Compiler/toolchain-only source files must carry `SPDX-License-Identifier: GPL-3.0-or-later` or an equivalent explicit GPLv3-or-later notice.
  6. The repository must include the full GPLv3 license text and the full GCC Runtime Library Exception 3.1 text before any public source release that claims this license policy. Recommended paths are `COPYING` or `LICENSES/GPL-3.0-or-later.txt` for GPLv3 and `COPYING.RUNTIME` or `LICENSES/GCC-exception-3.1.txt` for the exception.
  7. A user's Kyokai source code and resulting target program are treated as independent modules for license-policy purposes when they merely use the Kyokai runtime, standard library, public interfaces, or exception-covered target helpers through the ordinary compilation/linking process.
  8. Such target programs may be distributed under terms chosen by the program's author, including non-GPL terms, subject to the user's own code and dependency licenses and the conditions of the runtime exception.
  9. The runtime exception does not relicense the Kyokai compiler, package manager, formatter, LSP, backend, or other toolchain source. Distributing modified Kyokai toolchain code still follows GPLv3-or-later.
  10. The runtime exception also does not allow taking Kyokai runtime or standard-library source code itself and distributing modified versions outside the GPLv3-or-later plus exception terms.
  11. Public release tooling must be able to distinguish compiler/toolchain files from target-linked runtime/stdlib/helper files so generated license manifests, source headers, packages, and binary distributions do not blur the boundary.
  12. Before a public legal release, the project must verify the exact notice text and file layout against the official GNU/FSF texts and SPDX identifiers. If legal review requires a Kyokai-specific runtime exception text because the official GCC exception text is GCC-specific, that review may replace the notice wording while preserving this decided intent: reciprocal Kyokai toolchain, permissive licensing freedom for user target programs linked with Kyokai runtime/stdlib code.
  **Why this fits Kyokai**: Kyokai wants an open compiler ecosystem without making commercial, proprietary, permissive, or differently copylefted Kyokai applications legally unusable. The license boundary mirrors the technical boundary: compiler/toolchain changes remain shared, while target programs remain the user's work unless they actually incorporate modified Kyokai runtime/stdlib code outside the exception.
  **[STAGE: DECIDED_CORE_SEMANTICS | D263 → compiler/toolchain `GPL-3.0-or-later`; runtime/stdlib/target helpers `GPL-3.0-or-later WITH GCC-exception-3.1`]**
- **Build output and cache layout are fixed by the toolchain contract** — `kyokai build` does not leave artifact locations to convention. A workspace build writes user-visible products under the workspace root by default; a standalone package build writes them under the package root by default. The default output root is `kyokai-out/`; the default disposable cache root is `.kyokai-cache/`.
**Rules**:
  1. The owner root is the workspace root for workspace builds and the package root for standalone package builds.
  2. User-visible build artifacts are written under `<out-root>/<target-triple>/<profile>/<backend>/<package-name>/`.
  3. Standard output subdirectories are `bin/`, `lib/`, `koi/`, `gen/`, `doc/`, `reports/`, and inspectable `obj/` when a profile or flag asks for object files as user-visible artifacts.
  4. Tool-private incremental state is written under `<cache-root>/<toolchain-compat>/<target-triple>/<profile>/<backend>/<package-name>/`.
  5. `--out-dir <path>` selects the user-visible output root for the command. `--cache-dir <path>` selects the disposable cache root for the command.
  6. `kyokai clean` removes cache state by default. `kyokai clean --outputs` may remove output artifacts. `kyokai clean --all` removes both selected output and cache roots, but not source files, `kyokai.toml`, `kyokai.lock`, or package index/cache state outside the selected cache root.
  7. `kyokai run` executes from the output tree unless a target runner requires staging. Test/bench harness private state may live in the cache tree, while requested reports live under `reports/` or stdout.
  8. The output path is not a source semantic input. It becomes a build-identity input only when artifact contents record paths; reproducible profiles must use path remapping unless absolute path embedding is explicitly requested.
  **Why this fits Kyokai**: target/profile/backend/package facts already affect artifact identity, so the directory tree should say those facts out loud. Keeping `kyokai-out/` separate from `.kyokai-cache/` also keeps user-inspectable products separate from disposable compiler machinery.
  **[STAGE: DECIDED_CORE_SEMANTICS | D264 -> default `kyokai-out/` plus `.kyokai-cache/`; target/profile/backend/package output partition; explicit clean and override behavior]**
- **`.koi` has a concrete canonical artifact format** — `.koi` is not an opaque compiler cache and not a pretty source file. It is the canonical checked package interface artifact, stored as a structured binary container named Koi Binary Interface version 1 (`KBI-1`), with official inspection commands for humans and tools.
**Rules**:
  1. A `.koi` file begins with the magic bytes `KOI\n`, then fixed-width little-endian container version fields, section count, and section-table offset.
  2. The section table is sorted by numeric section id. Duplicate section ids are illegal. Unknown required sections are unsupported; unknown optional sections may be skipped but remain covered by artifact hashes.
  3. Required KBI-1 sections are `manifest`, `producer`, `target`, `sources`, `imports`, `declarations`, `types`, `typeclasses`, `instances`, `generics`, `contracts`, `unsafe_audit`, `docs`, and `hashes`.
  4. `.koi` represents the checked interface graph after parsing, name resolution, target selection, declaration-guard evaluation, type checking, typeclass checking, contract checking, capability checking, and unsafe-audit coverage checking for interface-affecting inputs.
  5. `.koi` does not preserve unchecked source syntax, private body declarations, comments except through doc metadata, or statement bodies except where generic materialization metadata is explicitly required.
  6. Public declarations are visible to downstream packages. Internal declarations may be present for same-package tooling and incremental checking but must be ignored outside the producing package. Private `.kai` declarations never become name-resolvable `.koi` declarations.
  7. Types are encoded as canonical typed graph nodes, not pretty-printed source strings. Typeclasses, instances, contracts, capabilities, unsafe audit data, and generic materialization metadata have explicit records and compatibility classes.
  8. A compiler may consume a `.koi` only when edition, KBI major version, target contract identity, backend/generic materialization compatibility class, required built-in/stdlib interface identity, dependency package identity, and dependency artifact hashes match the lockfile and compatibility table.
  9. `kyokai koi verify`, `kyokai koi print --format json|text`, and `kyokai koi diff` are official inspection commands. Derived print output is not a second artifact authority.
  10. Malformed structure, invalid UTF-8 strings, noncanonical ordering, hash mismatch, unsupported KBI major version, edition mismatch, target mismatch, dependency hash mismatch, missing required references, and visibility violations are rejection errors.
  **Why this fits Kyokai**: separate compilation needs rich metadata, but Kyokai's tooling also needs inspectability for docs, audit, SemVer, LSP, releases, and debugging. A canonical binary artifact with official inspection keeps one source of truth without reducing `.koi` to a compiler memory dump.
  **[STAGE: DECIDED_CORE_SEMANTICS | D265 -> canonical KBI-1 `.koi` binary container with required sections, explicit compatibility, and official verify/print/diff commands]**
- **Project creation commands are part of the toolchain contract** — a language that requires explicit package and module roots must not leave the first project layout to tutorial folklore. `kyokai init` and `kyokai new` create explicit manifests and source roots instead of relying on hidden defaults.
**Rules**:
  1. `kyokai init` creates a project in the current directory and refuses to overwrite an existing package/workspace by default.
  2. `kyokai new <path>` creates a new directory and initializes it from an official template.
  3. Official templates are `package`, `workspace`, `library`, `executable`, and `empty`.
  4. Generated package manifests include `[package]`, `version`, `edition`, and `[layout].module_root = "src"` unless the user explicitly chooses another valid relative module root.
  5. Workspace templates write `[workspace].members` and do not infer package membership from directory shape.
  6. Template expansion is deterministic for the same toolchain version and flags.
  7. The commands may create starter `.kyo`/`.kai`, test, doc, or CI files only through documented template behavior or explicit flags.
  8. Project creation commands do not resolve dependencies, contact package indexes, or run generated source unless a future explicit flag says so.
  **Why this fits Kyokai**: daily use starts at project creation. If the first command hides layout policy, the whole explicit package model starts with a lie.
  **[STAGE: DECIDED_CORE_SEMANTICS | D266 -> `kyokai init` and `kyokai new` with deterministic templates and explicit layout]**
- **Diagnostic explanation and safe fix application are first-party daily tooling** — diagnostic codes matter only if users can ask what they mean, and automatic fixes matter only if they are bounded by the same compiler truth as the diagnostic.
**Rules**:
  1. Every released diagnostic code has a local explanation catalog entry shipped with the toolchain.
  2. `kyokai explain <code-or-category>` prints the installed toolchain's explanation for a diagnostic code, warning category, lint category, audit category, or exit status.
  3. The explanation includes meaning, common causes, repair patterns where known, related codes, and spec/doc anchors when available.
  4. Online documentation may mirror the catalog, but the installed local catalog is authoritative for codes emitted by that installed compiler.
  5. `kyokai fix` is separate from `kyokai fmt`: formatting changes layout; fixing applies compiler suggestions.
  6. `kyokai fix` applies only `machine-applicable` suggestions by default. `maybe-incorrect` and `manual-only` suggestions require explicit user action and must not be silently applied.
  7. Fix application rejects stale spans, overlapping edit sets not already merged by the diagnostic engine, and edits that fail parse/format validation.
  8. If validation fails, the command leaves original files unchanged or restores them before reporting failure.
  **Why this fits Kyokai**: linearity, borrowing, capabilities, and contracts will create unfamiliar errors. The compiler must teach the rule and apply only the repairs it can honestly prove.
  **[STAGE: DECIDED_CORE_SEMANTICS | D267 -> local diagnostic explanations plus checked `kyokai fix` for machine-applicable suggestions]**
- **Local toolchain health is inspectable through `--version` and `doctor`** — installation and target setup failures should not be discovered only after a long build reaches the linker.
**Rules**:
  1. `kyokai --version` works without a project and prints toolchain version, source/release identity, supported editions, diagnostic schema version, host triple, default backend, and KBI compatibility range.
  2. `kyokai doctor` works without a project and checks release provenance, checksum/signature status where available, host support, C/LLVM backend discovery, configured target tools, cache/output writability, package index access, and explicitly admitted environment variables.
  3. `doctor` reports findings as diagnostics and suggestions.
  4. `doctor` must not edit source files, manifests, lockfiles, or project output artifacts.
  5. A project-aware `doctor` mode may also inspect the selected manifest, target tables, lockfile freshness, and configured native dependencies, but it must say which project it inspected.
  **Why this fits Kyokai**: explicit build identity is useless if the tool cannot explain its own identity and host setup.
  **[STAGE: DECIDED_CORE_SEMANTICS | D268 -> `kyokai --version` and `kyokai doctor` as first-party toolchain identity and health commands]**
- **Package inspection and offline workflows are first-class read-only or explicit-write commands** — daily package work needs more than `add` and `update`, but those commands must not smuggle registry trust back into Kyokai.
**Rules**:
  1. `kyokai remove <name>` removes a direct dependency entry and updates the lockfile, while reporting if the dependency remains reachable transitively.
  2. `kyokai search` queries configured discovery indexes without editing manifests or lockfiles.
  3. `kyokai info` reports package metadata, source revision, license, docs, yanked/advisory state, public interface summary, and audit summary when known.
  4. `kyokai tree` prints the resolved dependency graph deterministically from the selected manifest/lockfile.
  5. `kyokai why <package>` explains dependency paths that cause a package to be present.
  6. `kyokai outdated` compares pinned dependencies against configured update policy and index metadata, reporting newer revisions, yanks, and advisories without editing files.
  7. `kyokai vendor` materializes exact locked dependency sources into an explicit vendor directory with metadata tying each source tree to package identity, source URL, revision, checksum where available, and lockfile identity.
  8. Offline builds may use vendored sources only when manifest, lockfile, and vendor metadata agree.
  **Why this fits Kyokai**: users need to inspect the dependency graph every day, but inspection and vendoring must preserve the pinned-source model instead of recreating a hidden registry.
  **[STAGE: DECIDED_CORE_SEMANTICS | D269 -> package remove/search/info/tree/why/outdated/vendor with read-only inspection and pinned offline identity]**
- **Property and fuzz testing have daily replay, corpus, and minimization controls** — committing to property testing and fuzzing is not enough if failures cannot be reproduced.
**Rules**:
  1. `kyokai test --seed <value>` fixes deterministic generator and fuzz seed behavior where the selected runner supports it.
  2. Property failure reports record the test name, seed, shrink path or minimized input where available, target, backend, profile, and toolchain version.
  3. `kyokai test --replay <id-or-file>` replays recorded property failures, fuzz crashes, or minimized reproducers.
  4. `kyokai test --fuzz` runs explicit fuzz targets under the ordinary test toolchain contract.
  5. `--corpus <path>` selects an explicit fuzz corpus directory.
  6. `--minimize <id-or-file>` minimizes a recorded failing input while preserving the failure category.
  7. `--list` lists discovered tests, and `--failed` reruns tests recorded as failed by a previous compatible report.
  8. Crash reproducers and minimized cases are user-visible report/corpus artifacts, not hidden cache facts.
  **Why this fits Kyokai**: a safety-focused language needs tests that can be rerun exactly when the compiler, stdlib, or backend changes. Randomness without replay is just theater.
  **[STAGE: DECIDED_CORE_SEMANTICS | D270 -> property/fuzz seed, replay, corpus, minimization, list, and failed-test rerun controls]**
- **Kyokai carries an explicit formalization roadmap instead of pretending prose alone closes the soundness story** — the project treats the core calculus as part of the language plan, not as optional future polish.
  **Rules**:
  1. Immediate spec wording uses "design goal", "language contract", or "intended invariant" where a property has not yet been discharged by a formal proof.
  2. Before `v1.0`, Kyokai MUST produce a paper proof for a small sequential core calculus (`λ_K` / `λ_K-seq`) covering the safety-critical ownership-and-borrowing core.
  3. That first calculus MUST define abstract syntax, typing judgments, small-step operational semantics, and a soundness theorem adapted to Kyokai's defined TPOE outcome rather than pretending checked failure does not exist.
  4. The first calculus deliberately excludes concurrency, FFI, and backend lowering; those remain later extensions after the sequential core is nailed down.
  5. Once Kyokai is being written in Kyokai and the project is self-hosting, the next formalization step is a proper mechanized proof in the easiest suitable proof assistant on Linux, with Coq the current likely first choice.
  6. The working research note for this effort is `kyokailang/kyokaicalculus/lambda_k_research.md`; that note is not itself the normative proof, but it records the current proof scope, theorem shape, and prior-art map.
  **Why this fits Kyokai**: zero-UB and ownership claims stay explicit, but the document stops bluffing about what has and has not yet been proven.
  **[STAGE: DECIDED_CORE_SEMANTICS | D143/D241 → phased formalization plan; paper proof for sequential `λ_K` before `v1.0`; `lambda_k_research.md` is the current research note; mechanized proof after self-hosting, likely in Coq]**
- `**defer` statement** — Zig-style scope-exit semantics. `defer destroyByteBuf(x);` appears at declaration point and runs when the enclosing scope exits. This matches Austral's linearity scope model — linear variables must be consumed before their scope exits, and scope-exit defer does exactly that. Includes future `errdefer` extension for error-path cleanup. **[STAGE: DECIDED_CORE_SEMANTICS | D2 → Option B, Zig-style]**
- **Concurrency: Structured Concurrency + Linear Channels** — option C, implemented incrementally (spawn/join → channels → task groups). All communication through channels, no shared memory. Linear types enforce data-race freedom by construction. Everything explicit — no implicit priority inheritance, no hidden thread scheduling. **[STAGE: DECIDED_CORE_SEMANTICS | D3 → Option C, all explicit]**
- **Kyokai tasks are 1:1 OS-thread executions, not green-thread or M:N runtime tasks** — structured concurrency does not hide a scheduler underneath the language's explicit task surface.
**Rules**:
  1. Every live child task created by `spawn` executes on one operating-system thread for the duration of that task's execution.
  2. Kyokai has no language-level M:N scheduler, no green-thread runtime, no virtual-thread layer, and no user-transparent task migration semantics.
  3. A blocking operation in a task blocks that task's OS thread unless a separate explicit API contract says otherwise.
  4. Foreign calls execute on the ordinary OS thread of the calling task. Kyokai does not insert a hidden scheduler boundary around FFI calls.
  5. An implementation may reuse OS-thread resources only after the previous task running on that thread has fully exited. Such reuse does not change rules 1 through 4 and does not create language-level M:N semantics.
  **Why this fits Kyokai**: concurrency remains auditable at the same level as ownership and capabilities, blocking behavior stays honest, and FFI-facing systems code does not inherit a second hidden execution model.
  **[STAGE: DECIDED_CORE_SEMANTICS | D164 → `spawn` is 1:1 with OS threads; no M:N/green-thread scheduler semantics]**
- **Spawned child tasks use explicit capture lists and cannot implicitly close over parent bindings** — Kyokai aligns child-task capture with D118's “capture lists carry information” rule instead of letting concurrency smuggle in ambient closure semantics.
**Syntax**:
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
  **Rules**:
  1. A spawned child task is written as `spawn [captures] do ... od;` inside a `taskgroup do ... join;` block.
  2. The capture list is mandatory. `spawn [] do ... od;` is the zero-capture form.
  3. `name` captures by value. If `name` has a `Free` type, the child receives its own value as of the spawn point and the parent may continue using its own binding. If `name` has a `Linear` type, ownership transfers to the child and the parent binding is consumed at the spawn point.
  4. `&name` captures by immutable borrow. This form is legal only when `name` has a `Free` type or belongs to the closed shared-access concurrency set: `Atomic[T]`, `Mutex[T]`, or `RwLock[T]`.
  5. Capability objects, files, sockets, terminal handles, and other ordinary `Linear` runtime handles do not gain shared-borrow spawn capture merely because they are effectful. Cross-task use of such values must happen by ownership transfer or through a separate explicit synchronization or broker abstraction.
  6. `&!name` capture is illegal in `spawn`.
  7. A `&name` capture creates a borrow whose lifetime lasts until that child task completes at the `join;` point of the enclosing task group. Ordinary borrow rules continue to apply in the parent during that interval.
  8. Because D3 is structured concurrency, the enclosing `taskgroup` may not complete until all child tasks spawned within that task group complete.
  9. `defer` and `errdefer` are same-task scope-exit constructs, not child tasks, and therefore do not use spawn-capture rules.
  **Why this fits Kyokai**: cross-task ownership transfer stays visible in source, shared-read captures stay explicit, and concurrency does not become the one place where the language secretly permits ambient closure behavior.
  **[STAGE: DECIDED_CORE_SEMANTICS | D88 → explicit spawn capture lists; by-value transfer for `Linear`, by-value copy for `Free`, shared `&` capture only for `Free`, `Atomic[T]`, `Mutex[T]`, and `RwLock[T]`; no `&!` capture]**
- **Task groups make structured joins visible instead of hiding them at an arbitrary `od;`** — Kyokai uses a named `taskgroup do ... join;` boundary so borrowing, task lifetime, and blocking behavior are explicit at the source level.
**Rules**:
  1. `spawn [captures] do ... od;` is legal only inside a `taskgroup do ... join;` block.
  2. `join;` is the blocking structured join point for that task group. Execution after `join;` begins only after every child task spawned directly in the group has completed.
  3. Child tasks may be spawned sequentially inside a task group; the group does not wait after each `spawn` statement.
  4. Nested `taskgroup` blocks are legal. A child task may contain its own task group, but the outer group observes only completion of the direct child task.
  5. A borrow captured by `&name` is considered live until the captured child task completes, and for parent-side checking it remains unavailable for conflicting use until the enclosing group reaches `join;`.
  6. The current statement-form `spawn` produces no join handle and no completion value. Values and recoverable errors cross task boundaries only through explicit channels or synchronization objects.
  7. `panic` and TPOE are not task values and are not recoverable at `join;`; they keep D84's process-level termination semantics.
  8. `join;` is a D9 semantic boundary terminator for a task group, not a reversed keyword form.
**Why this fits Kyokai**: the program still gets structured concurrency and borrow-safe child tasks, but the source shows exactly where the parent may block and where captured borrows become available again.
  **[STAGE: DECIDED_CORE_SEMANTICS | D252 → explicit `taskgroup do ... join;`; `join;` is the visible blocking structured join point; no join handles or task-result wrappers]**
- **Thread creation failure is explicit at the `spawn` site unless capacity was reserved before the task group** — OS thread creation is an environmental failure, not TPOE, and Kyokai does not pretend a child started when no child exists.
**Syntax**:
  ```kyokai
  taskgroup do
      spawn [sender] do
          runWorker(sender);
      od else err do
          closeSender(sender);
          return Err(ThreadSpawnFailed(err));
      fi;
  join;
  ```
**Rules**:
  1. Creating a child task may fail before the child begins execution, for example because the OS refuses to create another thread.
  2. Such failure is reported as `ThreadSpawnError`; it is not `panic`, TPOE, or a recoverable child-task result.
  3. If a `spawn` statement fails, the child body does not begin execution, no spawn-start happens-before edge is created, and no child is added to the enclosing task group.
  4. By-value captures transfer to the child only after task creation succeeds. On spawn failure, by-value captures remain available to the failure arm, including linear values that would have transferred on success.
  5. Borrow captures are not extended to a child on spawn failure because no child exists.
  6. A fallible `spawn` must either have an `else err do ... fi;` failure arm or occur inside a task group whose task capacity was explicitly reserved before entering the group.
  7. A task-capacity reservation API is an explicit fallible API. If reservation succeeds, spawn statements covered by that reservation do not individually need failure arms for thread creation failure; any other spawn failure not covered by the reservation remains explicit.
  8. `ThreadSpawnError` is a standard-library error type describing resource exhaustion, permission failure, target unsupportedness, or other implementation-defined OS refusal categories that the target contract exposes. It must not erase ownership state.
**Why this fits Kyokai**: resource exhaustion remains an ordinary explicit failure path, while linear ownership transfer remains exact: values move into a child only if a child actually starts.
  **[STAGE: DECIDED_CORE_SEMANTICS | D235 → fallible `spawn` with explicit failure arm or pre-reserved task capacity; no TPOE for OS thread exhaustion]**
- **Task-boundary transfer is declaration-controlled rather than inherited from `Linear` or expressed through Rust-style `Send`/`Sync` marker interfaces** — Kyokai keeps cross-task movement explicit without importing auto-trait machinery.
**Rules**:
  1. Kyokai has a declaration-level task-boundary classification with two outcomes: `task_transfer` and `task_local`.
  2. Built-in `Free` value types are `task_transfer` unless a specific built-in contract says otherwise.
  3. User-defined `Free` nominal types are `task_transfer` unless their declaration explicitly marks them `task_local`.
  4. User-defined `Linear`, capability, and runtime-handle nominal types must either declare `task_transfer` or `task_local`, or be covered by an explicit standard-library authority/handle contract that states the classification.
  5. `Linear` by itself does not imply task-transfer admissibility.
  6. A `task_local` value may not be captured by value into `spawn`, sent through a channel, or otherwise transferred to another task by safe code.
  7. Capabilities and raw I/O handles remain single-owner values. They may be transferred across tasks only when their own capability/handle contract admits owned task transfer.
  8. Shared cross-task capture by immutable borrow remains limited to `Free` values and the closed synchronized set `Atomic[T]`, `Mutex[T]`, and `RwLock[T]`, unless a later D-point adds another named synchronized primitive.
  9. Safe Kyokai does not provide user-implemented `Send` or `Sync` marker typeclasses, auto-traits, or structural inference for task-transfer or shared-task safety.
  10. If a thread-affine host resource needs cross-task access, safe code must use an explicit broker task, ownership transfer back to the owning task, or a separately specified synchronized wrapper whose contract names its interleaving and affinity rules.
**Surface examples**:
  ```kyokai
  record ByteBuffer: Linear, task_transfer is ... build;
  record GlContext: Linear, task_local is ... build;
  capability WindowServerCapability: task_local;
  ```
**Why this fits Kyokai**: the language answers the real soundness issue without turning task movement into an inferred trait folklore layer. Authority, ownership, and thread affinity stay attached to the type's explicit contract.
  **[STAGE: DECIDED_CORE_SEMANTICS | D248 → no Rust-style `Send`/`Sync`; task-boundary transfer is explicit declaration/contract metadata; thread-affine types are `task_local`]**
- **Child-task error propagation stays explicit and does not create a second recoverable task-failure taxonomy** — Kyokai keeps task boundaries honest: ordinary recoverable errors are just ordinary values, while `panic` and TPOE remain process-level termination.
**Rules**:
  1. The current statement-form `spawn [captures] do ... od;` does not itself yield a completion value to the parent.
  2. If a child task must communicate `T` or `Result[T, E]` to its parent or to sibling tasks, it does so through explicit channels or other explicitly passed synchronization objects defined by the language.
  3. A `Result[T, E]` that crosses a task boundary by such an explicit mechanism crosses unchanged as `Result[T, E]`. The language does not wrap `Err(E)` in `TaskError[E]`, `JoinError`, or any other implicit task-failure envelope.
  4. `Err(E)` crossing a task boundary is ordinary recoverable program data, not a runtime task failure category.
  5. `panic(message)` and TPOE are not task values and are not recoverable at a structured join point. Under D84 and D3a they terminate the whole process.
  6. A child task communicating `Err(E)` does not implicitly cancel siblings or force parent termination. Cancellation remains explicit under D91.
  **Why this fits Kyokai**: channels remain the visible transport mechanism, ordinary errors remain typed data, and the language refuses Rust-style join wrappers that would pretend process-fatal failure is just another recoverable branch.
  **[STAGE: DECIDED_CORE_SEMANTICS | D168 → task-boundary error propagation is explicit-value transport only; no implicit `TaskError` wrapper; `panic`/TPOE remain process termination]**
- **Channel capacity is always explicit at construction; Kyokai provides no default channel constructor** — every channel's capacity behavior, blocking discipline, and endpoint topology is visible at the construction site and in the operation names.
**Topology**: Kyokai channels are **single-producer, single-consumer (SPSC)**. `Sender[T]` and `Receiver[T]` are unique linear endpoints. They are not cloneable and cannot be split. Fan-in, fan-out, and broadcast topologies are expressed through explicit relay/broker tasks that compose SPSC channels — every connection is visible in source. MPSC, MPMC, and broadcast are not part of this decision; if they are needed later, they require their own design with explicit shared-ownership lifecycle semantics.
**Constructors**:
  ```kyokai
  record ChannelEndpoints[T: Type] is
      sender: Sender[T];
      receiver: Receiver[T];
  build;

  // Bounded: fixed capacity, backpressure via blocking
  function makeBoundedChannel[T: Type](capacity: Index): ChannelEndpoints[T];

  // Growable: requires an allocator, starts at initialCapacity, grows on demand
  function makeGrowableChannel[T: Type, A: Allocator](
      alloc: &![A], initialCapacity: Index
  ): ChannelEndpoints[T];
  ```
  **Why no bare `makeChannel[T]()`**: a channel's capacity is its most consequential property — it determines whether the program can OOM, whether it deadlocks, and what the backpressure behavior is. Hiding that behind a default violates the same principle as D44 (no hidden default allocator) and D74 (OOM is explicit). The programmer writes `makeBoundedChannel[Int32](64)` or `makeGrowableChannel[Int32](&!heap, 64)` and the construction site tells the full story.
  **Why `makeGrowableChannel` instead of `makeUnboundedChannel`**: the name signals "this thing allocates and grows" rather than "this thing has no limit." The allocator parameter makes the memory cost visible (D44). The `initialCapacity` parameter gives the programmer a sizing hint without hiding the growth behavior.
  **Why SPSC only**: MPSC requires cloneable senders, which means reference counting or shared ownership — both violate linearity. MPMC requires cloneable senders AND receivers. Broker tasks make topology explicit in source code: if you want 3 producers feeding one consumer, you spawn a broker with 3 receivers and one output channel. Every connection is visible. No hidden shared ownership.
  **Operation signatures**:
  ```kyokai
  // Blocking operations: the name says "Blocking" because the suspension point
  // must be visible at every call site (same principle as D11b naming conventions).
  // sendBlocking: mutable borrow of sender (you keep using it), value consumed (ownership transfers)
  // Returns Result because the receiver may be closed or (growable) allocation may fail
  function sendBlocking(sender: &![Sender[T]], value: T): Result[Unit, SendError[T]];

  // recvBlocking: returns Optional — None when channel is closed and drained
  function recvBlocking(receiver: &![Receiver[T]]): Optional[T];

  // Non-blocking variants — never suspend
  function trySend(sender: &![Sender[T]], value: T): Result[Unit, TrySendError[T]];
  function tryRecv(receiver: &![Receiver[T]]): Result[T, TryRecvError];

  // Explicit endpoint completion
  function closeSender(sender: Sender[T]): Unit;
  function closeReceiver[T: Free](receiver: Receiver[T]): Unit;
  function drain[T: Linear](receiver: Receiver[T]): DrainIterator[T];
  ```
  **Why `sendBlocking`/`recvBlocking` instead of `send`/`recv`**: Kyokai's rule is "if it's happening, it must be visible in source." A function that blocks the calling task is doing something important — hiding that behind a generic name `send` violates the same principle that D8 applies to return values and D11b applies to ownership. The blocking point is in the name. No effect system needed, no colored functions — just honest naming. This also leaves clean namespace space for future deadline-based variants (`sendUntil`, `recvUntil`) without ambiguity.
  **Error types**:
  ```kyokai
  // SendError returns the value to the caller — linear values are not leaked on failure
  union SendError[T: Type] is
      case Disconnected(value: T);   // receiver closed
      case AllocFailed(value: T);    // growable channel could not grow (D74)
  build;

  union TrySendError[T: Type] is
      case Full(value: T);           // bounded channel at capacity
      case Disconnected(value: T);   // receiver closed
      case AllocFailed(value: T);    // growable channel could not grow
  build;

  union TryRecvError is
      case Empty;                    // no value available right now
      case Disconnected;             // sender closed, channel drained
  build;
  ```
  **Blocking rules**:
  1. On a bounded channel, `sendBlocking` blocks until the channel has capacity or the receiver is closed.
  2. On a growable channel, `sendBlocking` attempts to grow the internal buffer. If the allocator returns `AllocError`, `sendBlocking` returns `Err(AllocFailed(value))` without blocking.
  3. `recvBlocking` blocks until a value is available or the channel is closed and drained.
  4. `trySend` and `tryRecv` never block.
  **Transfer ordering rules**:
  1. Each SPSC channel is FIFO. Successful sends are observed by receives in sender program order.
  2. `closeSender(sender)` never lets closure observation overtake earlier successful sends. The receiver must first drain every buffered value from sends that completed before the close, in FIFO order.
  3. A failed `trySend(Full(...))`, failed `sendBlocking(...)->Err(...)`, `tryRecv(Empty)`, or any other non-transferring channel result creates no cross-task ordering guarantee by itself.
  **Closure rules**:
  1. `Sender[T]` and `Receiver[T]` are `Linear`. Forgetting to close an endpoint is a compile-time linearity error.
  2. Closing the sender signals the receiver: subsequent `recvBlocking` calls drain any buffered values and then return `None`, and `tryRecv` returns `Disconnected` only after the same drain is complete.
  3. For `T: Free`, `closeReceiver(receiver)` is available. It closes the receiver immediately, discards any buffered `Free` values, and causes subsequent sender operations to return `Err(Disconnected(value))`.
  4. For `T: Linear`, `Receiver[T]` exposes no direct close/discard operation. The only completion path is `drain(receiver)`, which consumes the receiver and yields buffered and subsequently arriving values in FIFO order until the sender side is closed and the channel is empty.
  5. Exhausting the `DrainIterator[T]` completes receiver teardown for `T: Linear`. There is no second close step after the drain finishes.
  6. Abandoning a `DrainIterator[T]` before exhaustion is illegal for `T: Linear`, because doing so would abandon queued linear values.
  7. If the sender remains live and continues to hold the channel open, `drain(receiver)` may block indefinitely while waiting for eventual sender closure. That liveness obligation is explicit program behavior, not hidden cleanup.
  8. Sending on a known-closed channel (where the sender has already received `Err(Disconnected(...))` and ignores it) is ordinary error handling, not TPOE. TPOE applies only when specified by contract (e.g., the spec may define that using a sender endpoint after it has already been consumed is a linearity violation, which the compiler catches statically).
  **D84 termination interaction**:
  1. `panic(message)` terminates the whole process (D84 rule 2). There is no per-task panic recovery. `panic` runs `defer` cleanup in exiting scopes (D84 rule 3), which consumes linear channel endpoints through their deferred close. No special channel-unblocking logic is needed — the process is terminating.
  2. TPOE terminates the whole process immediately (D84 rule 4). No `defer` runs. No channel cleanup. The process is dead.
  3. There is no scenario where one task dies and another task continues. Structured concurrency means child tasks are joined before the parent scope exits; `panic` and TPOE terminate the whole process, not individual tasks.
  **Buffered values on channel teardown**:
  1. For `T: Free`, `closeReceiver(receiver)` may discard buffered values.
  2. For `T: Linear`, buffered values are never silently discarded by receiver teardown. They are yielded through `drain(receiver)` and must be consumed by user code.
  3. `panic` and TPOE remain process-level termination paths under D84. Kyokai does not define a continuing-program receiver-teardown shortcut that silently destroys buffered linear messages.
  **Why this fits Kyokai**: every aspect of channel behavior — capacity, allocation, blocking, closure, topology, error recovery, and termination interaction — is either visible in the source or follows mechanically from existing decided rules (D44, D74, D84, D2b). Nothing is hidden, nothing is defaulted, and the D84 termination contract is respected without exceptions. Blocking is in the name. Topology is in the type. Capacity is in the constructor.
  **[STAGE: DECIDED_CORE_SEMANTICS | D3a → SPSC channels; explicit bounded/growable constructors; `sendBlocking`/`recvBlocking` with visible blocking point; linear endpoints; D84-compliant process-level termination]**
- **Complex channel topologies are explicit broker tasks over SPSC channels, not new cloneable channel endpoint types** — Kyokai keeps the SPSC topology contract from D3a and standardizes the broker pattern as visible source structure rather than hidden shared ownership.
**Rules**:
  1. Kyokai does not add MPSC, MPMC, or broadcast channel endpoint types to the core channel model.
  2. `Sender[T]` and `Receiver[T]` remain unique linear endpoints. They are not cloneable, splitable, reference-counted, or implicitly shared.
  3. Fan-in, fan-out, work-queue, logging, and broadcast-like designs are expressed through explicit broker tasks that own the necessary SPSC endpoints and relay messages using ordinary channel operations and `select`.
  4. The standard library may provide broker helper functions or templates, but those helpers must construct, accept, and own visible SPSC endpoints. They do not create hidden MPSC/MPMC state.
  5. Broker helpers must document ordering, fairness or non-fairness, shutdown, backpressure, cancellation/deadline interaction, and linear-payload drain behavior.
  6. If a future design wants true MPSC, MPMC, or broadcast endpoints, it requires a separate D-point covering shared endpoint ownership, teardown, ordering, memory use, and task-transfer rules.
  **Why this fits Kyokai**: many-producer and broadcast workflows remain possible, but topology is still visible in source and the language does not smuggle `Arc`-style ownership into channels.
  **[STAGE: DECIDED_CORE_SEMANTICS | D236 → standard broker pattern over SPSC channels; no MPSC/MPMC/broadcast endpoint primitives]**
- **Linear receiver teardown is explicit drain, not close-and-discard** — Kyokai does not let a `Receiver[T]` for linear payloads silently abandon buffered values or synthesize hidden destruction behavior.
**Rules**:
  1. `Receiver[T]` where `T: Linear` does not expose `closeReceiver`.
  2. The only receiver-completion path for `T: Linear` is `drain(receiver): DrainIterator[T]`.
  3. `drain(receiver)` consumes the receiver and yields every buffered value in FIFO order, then any later values, until the sender side is closed and the channel is empty.
  4. Exhausting the drain iterator completes teardown. There is no second receiver-close operation after exhaustion.
  5. Abandoning the drain before exhaustion is illegal for `T: Linear`.
  6. `Receiver[T]` where `T: Free` may use `closeReceiver(receiver)` directly, and buffered `Free` values may be discarded.
  **Why this fits Kyokai**: linear messages remain explicit program obligations all the way through channel teardown. The language does not invent a cleanup exception at exactly the point where values are easiest to lose sight of.
  **[STAGE: DECIDED_CORE_SEMANTICS | D146 → `Receiver[T: Linear]` completes through explicit `drain`; `Receiver[T: Free]` may close directly and discard buffered values]**
- `**Atomic[T]` (along with `Mutex[T]` and `RwLock[T]`) is one of the closed set of language-defined types where shared access may change storage** — its operations are explicitly ordered and this property is stated by the spec, not derived from a general interior mutability mechanism. No other type outside this explicit concurrency set may exhibit this behavior. Atomics live in `Kyokai.Atomic` as a first-class standard library module. They are memory-ordering primitives for implementing lock-free data structures and cross-task observable state. They are not a coordination mechanism between tasks. Task coordination uses channels (D3/D3a).
**Type design**:
  ```kyokai
  // Atomic[T] is Linear — one owner (the scope that created it).
  // Child tasks access it through structured concurrency scope rules.
  // T is constrained to types with hardware atomic support.
  record Atomic[T: AtomicType] is
      // opaque internal representation
  build;

  // AtomicType: a typeclass constraining which types can be atomic.
  // Implementations: Int8, Int16, Int32, Int64, Nat8, Nat16, Nat32, Nat64, Index, Bool.
  // Not arbitrary T — only types with hardware atomic semantics.

  // No default ordering. Every operation names its ordering.
  union MemoryOrder is
      case Relaxed;
      case Acquire;
      case Release;
      case AcqRel;
      case SeqCst;
  build;

  // CAS result is a named union, not Result[T, T].
  // CAS failure is an expected algorithmic branch, not an error.
  union CompareExchangeResult[T: AtomicType] is
      case Exchanged(previous: T);   // CAS succeeded, previous value returned
      case Failed(observed: T);      // CAS failed, actual observed value returned
  build;
  ```
  **API**:
  ```kyokai
  function makeAtomic[T: AtomicType](initial: T): Atomic[T];

  function load(a: &[Atomic[T]], order: MemoryOrder): T;
  function store(a: &[Atomic[T]], value: T, order: MemoryOrder): Unit;

  function fetchAdd(a: &[Atomic[T]], value: T, order: MemoryOrder): T;
  function fetchSub(a: &[Atomic[T]], value: T, order: MemoryOrder): T;
  function fetchAnd(a: &[Atomic[T]], value: T, order: MemoryOrder): T;
  function fetchOr(a: &[Atomic[T]], value: T, order: MemoryOrder): T;
  function fetchXor(a: &[Atomic[T]], value: T, order: MemoryOrder): T;

  function compareExchange(
      a: &[Atomic[T]],
      expected: T,
      desired: T,
      successOrder: MemoryOrder,
      failureOrder: MemoryOrder
  ): CompareExchangeResult[T];

  function compareExchangeWeak(
      a: &[Atomic[T]],
      expected: T,
      desired: T,
      successOrder: MemoryOrder,
      failureOrder: MemoryOrder
  ): CompareExchangeResult[T];  // Failed(observed) may occur spuriously even when observed == expected

  function fence(order: MemoryOrder): Unit;
  ```
  **Ordering-argument validity rules**:
  1. Every `MemoryOrder` argument to `load`, `store`, read-modify-write operations, `compareExchange`, `compareExchangeWeak`, and `fence` must be a comptime-known `MemoryOrder` variant, not a runtime variable.
  2. `load` accepts only `Relaxed`, `Acquire`, or `SeqCst`.
  3. `store` accepts only `Relaxed`, `Release`, or `SeqCst`.
  4. Read-modify-write operations such as `fetchAdd`, `fetchSub`, `fetchAnd`, `fetchOr`, and `fetchXor` accept `Relaxed`, `Acquire`, `Release`, `AcqRel`, or `SeqCst`.
  5. `fence(Relaxed)` is illegal. Legal fence orders are `Acquire`, `Release`, `AcqRel`, and `SeqCst`.
  6. For `compareExchange` and `compareExchangeWeak`, `failureOrder` may only be `Relaxed`, `Acquire`, or `SeqCst`.
  7. For `compareExchange` and `compareExchangeWeak`, `failureOrder` must not be stronger than `successOrder`. The only legal `(successOrder, failureOrder)` pairs are:
     - `(Relaxed, Relaxed)`
     - `(Acquire, Relaxed)` and `(Acquire, Acquire)`
     - `(Release, Relaxed)`
     - `(AcqRel, Relaxed)` and `(AcqRel, Acquire)`
     - `(SeqCst, Relaxed)`, `(SeqCst, Acquire)`, and `(SeqCst, SeqCst)`
  **Why `Atomic[T]` is `Linear`**: atomics are shared-access values, but the *storage* must have a single owner. In structured concurrency, the parent scope owns the `Atomic[T]`. Child tasks access it because the parent scope outlives all children. When the scope exits and all children have joined, the atomic is solely the parent's again and can be consumed normally. This eliminates the need for reference-counting wrappers like Rust's `Arc<AtomicU64>`.
  **Why all operations take `&[Atomic[T]]`**: `Atomic[T]` (along with `Mutex[T]` and `RwLock[T]`) is part of a closed set of language-defined synchronized shared-mutation primitives. Its operations are explicitly ordered and are one of the only cases where shared access may change storage. The programmer sees `Atomic` in the type and `MemoryOrder` in every operation — there is no scenario where a reader is surprised by mutation. This is not a general interior mutability mechanism. Kyokai does not have `Cell`, `RefCell`, `UnsafeCell`, or any user-extensible way to mutate through `&[T]`. The compiler knows `Atomic[T]` is part of a specific type family with this property and does not generalize it. No other type, including future standard library types, may acquire this property without a new explicit decision point.
  **Why not a third borrow mode (`&@[T]` or similar)**: a third access mode that applies to exactly one type family is more machinery for the same semantics. The compiler still needs to know `Atomic[T]` is special — with `&@`, the specialness is "this is the only type that can be behind `&@`"; with the named exception, the specialness is "this is the only type where `&[T]` permits mutation." Same information, different encoding. Adding `&@` also creates pressure to generalize ("why not `&@` for mutexes? for concurrent data structures?"), while the closed exception explicitly says "this is `Atomic[T]` and nothing else, ever." Kyokai's reference syntax is already four operators (`&`, `&!`, `~`, `&~`); a fifth adds learning cost for one type family.
  **Why `CompareExchangeResult[T]` instead of `Result[T, T]`**: CAS failure is not an error — it is an expected algorithmic branch. Using `Result` makes it look like ordinary error propagation and tempts `or return` usage, which is wrong. `Exchanged(previous)` and `Failed(observed)` are self-documenting field names. `compareExchangeWeak` uses the same result type; `Failed(observed)` may occur spuriously even when `observed == expected`.
  **Why not FFI-only**: atomics are needed to implement the channel runtime itself. If atomics are FFI-only, the foundational concurrency primitive is outside the language. FFI has no linearity checking, no memory-ordering type safety, and no compiler visibility into what the operation does. Every program that needs a simple atomic counter would require a C shim file. This violates the Kyokai philosophy of "if it's commonly needed, provide it properly."
  **Why `Kyokai.Atomic` is not `pragma Unsafe_Module`**: atomics do not create memory unsafety. They create ordering complexity, but the operations themselves are safe — the hardware guarantees atomicity, and the explicit `MemoryOrder` argument makes the ordering choice visible in source. The module is safe standard library code.
  **C backend lowering rules**:
  1. On the C backend, `Atomic[T]` lowers through C11 `<stdatomic.h>` only.
  2. Generated C represents atomic storage as `_Atomic T` or the corresponding standard atomic typedef form.
  3. `load`, `store`, read-modify-write operations, compare-exchange operations, and fences lower to the matching `atomic_*_explicit` operations.
  4. The C backend must never lower safe atomic operations to plain reads/writes or to `volatile` accesses.
  5. The backend must ensure the generated storage representation satisfies the selected target/toolchain's C11 atomic alignment requirements. If a source-level combination would violate that contract, compilation fails for that target/backend path.
  6. Kyokai does not promise lock-free atomics for every admitted atomic type on every target. It promises the specified semantics.
  7. If the selected C backend target/toolchain contract cannot provide conforming C11 atomic semantics for the used `Atomic[T]` operations, the build fails rather than silently weakening the memory model.
  **Usage pattern**:
  ```kyokai
  import Kyokai.Atomic (Atomic, MemoryOrder, makeAtomic, load, store, fetchAdd, destroyAtomic);

  function doWork(): Unit is
      var counter: Atomic[Int32] := makeAtomic(0);
      defer destroyAtomic(counter);

      // child tasks access counter through structured concurrency scope
      taskgroup do
          spawn [&counter] do
              counter.fetchAdd(1, SeqCst);
          od;

          spawn [&counter] do
              let val: Int32 := counter.load(Acquire);
          od;
      join;

      // join point: all children done, counter is solely ours again
      let final: Int32 := counter.load(SeqCst);
  qed;
  ```
  **Why this fits Kyokai**: atomics are explicit, typed, ordering-annotated, and provided as a proper language-visible module. The `Linear` ownership + structured concurrency scope rules guarantee storage lifetime without reference counting. The shared-mutation property of `Atomic[T]` is stated directly in the spec as a closed, non-generalizable exception. The programmer always sees `Atomic` in the type name and `MemoryOrder` in every operation call — nothing is hidden.
  **[DECIDED: D3b/D141 → `Kyokai.Atomic` module; `Atomic[T]` is `Linear`; all operations take `&[Atomic[T]]` with explicit `MemoryOrder`; `Atomic[T]` is part of the closed set of types where shared access may change storage; `CompareExchangeResult[T]` for CAS; no general interior mutability; C backend lowers through C11 atomics only]**
- **Mutex and RwLock Primitives (Kyokai.Sync)** — explicit `Linear` synchronization primitives for shared-state concurrency, complementing channels (D3a) and atomics (D3b).
**Type design**:
  ```kyokai
  // 1. The Mutex is Linear (sole ownership)
  record Mutex[T: Type]: Linear is
      // opaque internal representation
  build;

  // 2. The Guard is Linear (must be explicitly consumed to unlock)
  record MutexGuard[T: Type]: Linear is
      // opaque internal representation
  build;

  record RwLock[T: Type]: Linear is ... build;
  record ReadGuard[T: Type]: Linear is ... build;
  record WriteGuard[T: Type]: Linear is ... build;
  ```
**Operations**:
  ```kyokai
  // Creation (No capability required)
  function makeMutex[T: Type](value: T): Mutex[T];

  // Locking takes an IMMUTABLE borrow of the Mutex (shared among tasks)
  function lockBlocking[T: Type](m: &[Mutex[T]]): MutexGuard[T];

  union TryLockError is case Locked; build;
  function tryLock[T: Type](m: &[Mutex[T]]): Result[MutexGuard[T], TryLockError];

  // Accessing the data requires mutably borrowing the Guard.
  function access[T: Type](guard: &![MutexGuard[T]]): &![T];

  // Unlocking consumes the Guard.
  function unlock[T: Type](guard: MutexGuard[T]): Unit;

  // Destruction consumes the Mutex and returns the inner data.
  function destroyMutex[T: Type](m: Mutex[T]): T;
  ```
  **Why `Mutex[T]` requires no capability**: capabilities represent unforgeable external OS authority. Memory synchronization is internal to the process and does not grant authority over the outside world.
  **Why locking takes `&[Mutex[T]]`**: Multiple child tasks capture immutable borrows (`&m`) under D88. `Mutex[T]` and `RwLock[T]` join `Atomic[T]` as the closed set of types where shared access permits interior mutability.
  **Why `MutexGuard[T]` is `Linear`**: it forces the programmer to explicitly call `unlock(guard)`. A forgotten unlock is a compile-time linearity error.
  **[STAGE: DECIDED_CORE_SEMANTICS | D100 → `Mutex[T]`/`RwLock[T]` with linear scoped guards; explicit `access` via guard; immutable borrow to lock; no capabilities required]**
- **Official concurrency guidance is ordered: channels first for coordination and ownership transfer, mutexes for truly shared mutable structures, atomics for narrow low-level synchronization only** — Kyokai exposes multiple concurrency primitives, but it does not leave their ordinary intended use as folklore.
**Guidance rules**:
  1. Channels are the default coordination mechanism for task-to-task communication, work pipelines, ownership transfer, shutdown signaling, and other explicit data-flow patterns.
  2. `Mutex[T]` and `RwLock[T]` are the recommended tool only when multiple tasks genuinely need shared access to one in-memory mutable structure and a broker-or-channel design would be materially worse.
  3. `Atomic[T]` is for simple flags, counters, sequence numbers, and low-level synchronization internals. It is not the default shared-state coordination tool.
  4. If more than one design is plausible, the default choice order is: channels first, mutex/rwlock second, atomics last.
**Why this fits Kyokai**: the language keeps one clear beginner-to-expert gradient for concurrency instead of pushing every user directly into low-level synchronization choices.
**[STAGE: DECIDED_CORE_SEMANTICS | D184 → official concurrency guidance: channels by default, mutexes/rwlocks for genuine shared mutable structures, atomics only for narrow low-level synchronization]**
- **Raw capability-bearing I/O surfaces are single-owner and are not secretly synchronized by the runtime** — Kyokai does not hide a mutex inside terminal/file/socket authority in order to make cross-task output "just work."
**Rules**:
  1. Safe production I/O operations that mutate externally visible stream state or advance handle state require a mutable borrow of the capability-bearing object or I/O handle being used.
  2. Raw terminal, file, socket, and similar capability-bearing I/O objects remain `Linear` and are not safe for concurrent shared use merely because they wrap OS resources.
  3. Safe Kyokai does not permit two live tasks to perform concurrent raw I/O through the same capability or handle object.
  4. The language and toolchain must not insert hidden mutexes into `TerminalCapability`, `File`, sockets, or other raw I/O capability surfaces in order to fabricate thread safety.
  5. If multi-task output to one destination is needed, the sanctioned default pattern is an explicit broker task that owns the destination and receives messages over channels.
  6. A separately designed synchronized writer abstraction may exist as its own explicit type with its own documented interleaving guarantees, but that is a different API surface from the raw capability-bearing one.
  7. Distinct handles may still refer to the same underlying OS resource. Any atomicity, file-offset-sharing, or interleaving guarantees across distinct handles must be documented per API and are not implied by capability linearity alone.
  **Why this fits Kyokai**: capability ownership remains honest, concurrency does not open a hidden runtime-synchronization exception, and programs that need shared-output policy must spell that policy explicitly in source.
  **[STAGE: DECIDED_CORE_SEMANTICS | D212 → raw capability-bearing I/O is exclusive by handle/capability; no hidden synchronization; shared multi-task output uses explicit broker or explicit synchronized wrapper types]**
- **Shared Ownership and Reference Counting** — formally rejected.
  **Why no `Rc[T]` or `Arc[T]`**: reference counting implies shared ownership, which fundamentally violates the linear type system's invariant that every resource has exactly one owner responsible for its destruction. In Kyokai, ownership is strictly single.
  **How to share data without reference counting**:
  1. Concurrent access is achieved through shared borrowing (`&[Mutex[T]]` or `&[Atomic[T]]`) managed by structured concurrency scopes (children cannot outlive parents).
  2. Sequential sharing transfers ownership entirely via channels.
  3. Graph data structures must use arenas (D96) or array indices.
  **[STAGE: DECIDED_CORE_SEMANTICS | D101 → formally reject `Rc[T]` and `Arc[T]`; structured scopes and `Mutex[T]` replace shared-ownership concurrency]**
- **Cooperative Cancellation and Deadlines** — preemption is banned because it violates linear resource cleanup (defers wouldn't run). All cancellation is cooperative.
  **Deadlines**: explicit variants `sendUntil` and `recvUntil` take a deadline and return `TimedOut` on failure.
  **Cancellation**: `CancellationToken` is a `Free` type (backed by an atomic) created by the parent and passed immutably `&[CancellationToken]` to child tasks. Children explicitly poll `token.isCancelled()`. Blocking operations optionally accept a token and wake up to return `Err(Cancelled)` if triggered.
  **Cleanup**: Cancellation is a structured exit, not TPOE. It runs all `defer` blocks normally.
  **[STAGE: DECIDED_CORE_SEMANTICS | D91 → cooperative cancellation only; `CancellationToken`; deadline-based blocking variants; runs `defer`]**
- **Rendezvous channels are a distinct explicit constructor rather than a magic capacity value** — synchronous handoff has meaningfully different blocking behavior from buffered channels, so Kyokai names it directly instead of smuggling it through `capacity = 0`.
**API**:
  ```kyokai
  function makeRendezvousChannel[T: Type](): ChannelEndpoints[T];
  ```
  **Rules**:
  1. `makeBoundedChannel[T](capacity)` requires `capacity >= 1`. `capacity = 0` is illegal for the bounded-channel constructor.
  2. A rendezvous channel has no internal element buffer and performs synchronous sender/receiver pairing.
  3. `sendBlocking(sender, value)` on a rendezvous channel blocks until a matching receive operation pairs with that send and takes ownership of `value`.
  4. `recvBlocking(receiver)` on a rendezvous channel blocks until a matching send operation pairs with it, or until the sender side is closed and no future pairing can occur.
  5. `trySend` on a rendezvous channel succeeds only when a receiver is already waiting; otherwise it returns the channel family's ordinary non-success send result without transferring ownership.
  6. `tryRecv` on a rendezvous channel succeeds only when a sender is already waiting; otherwise it returns the channel family's ordinary empty/non-ready receive result.
  7. Rendezvous channels still carry an explicit liveness obligation: if no matching peer ever arrives and no deadline/cancellation path is used, the blocking operation may wait indefinitely.
  8. Deadline and cancellation variants such as D91's `sendUntil` and `recvUntil` are the explicit way to bound that waiting behavior.
  **Why this fits Kyokai**: synchronous handoff remains available, but the program spells that stronger coordination contract at the constructor site instead of hiding it inside one numeric argument edge case.
  **[STAGE: DECIDED_CORE_SEMANTICS | D183 → explicit `makeRendezvousChannel`; bounded channels require capacity ≥ 1; rendezvous remains synchronous and liveness-explicit]**
- **Non-Blocking I/O and Poller Contract** — no hidden global event loop. I/O multiplexing uses an explicit, linear `Poller` API.
  **Types**: `Poller` is `Linear` and wraps the OS event queue (`epoll`/`kqueue`).
  **Capabilities**: Creating a `Poller` requires a `ProcessCapability` (or similar OS authority capability).
  **Registration**: Explicit `register(poller: &![Poller], handle: &![FileDescriptor], interest)` — requires mutable borrows of both the poller and the handle.
  **Waiting**: `wait(poller: &![Poller], eventsOut, deadline)` blocks the task until events occur. Portable semantics are level-triggered.
  **[STAGE: DECIDED_CORE_SEMANTICS | D93 → explicit `Linear` `Poller` API; requires capability to create; explicit registration and wait]**
- **The event-loop / reactor boundary is explicit library code over `Poller`, not a hidden runtime service** — Kyokai supports high-concurrency I/O without adding a global executor or invisible suspension points.
**Rules**:
  1. Kyokai has no hidden global reactor, hidden event loop, hidden executor, green-thread scheduler, or language-level async suspension.
  2. High-concurrency I/O is expressed through explicit `Poller` values and libraries built on top of them.
  3. An event loop is ordinary Kyokai code that owns or mutably borrows a `Poller`; it is not a language runtime service.
  4. Handles used with such an event loop must be explicitly registered with the `Poller` or with an explicitly specified equivalent readiness mechanism.
  5. Blocking behavior remains visible in API names and result types under D237; the reactor boundary does not silently turn plain blocking calls into cancellable or nonblocking calls.
**Why this fits Kyokai**: C10K-style I/O remains possible, but the program states where readiness is tracked and who owns the polling object.
  **[STAGE: DECIDED_CORE_SEMANTICS | D234 → explicit event loops over `Poller`; no hidden reactor, executor, or runtime suspension]**
- **Cancellation-aware blocking syscall wrappers use explicit readiness mechanisms; plain blocking calls may wait indefinitely** — Kyokai does not pretend an OS thread blocked in a synchronous syscall can be safely interrupted by hidden runtime magic.
**Rules**:
  1. Safe Kyokai blocking I/O APIs are split into plain blocking operations and cancellation/deadline-aware operations.
  2. A plain blocking operation such as `readBlocking` or `writeBlocking` may block the calling OS thread until the OS operation completes, fails, or the resource-specific close/error condition occurs. If the peer or device never becomes ready and no deadline/cancellation surface was requested, the operation may wait indefinitely.
  3. A cancellation-aware or deadline-aware blocking operation must make that extra exit path visible in the API name, parameters, or result type, for example through `readUntil`, `writeUntil`, `Cancelled`, or `TimedOut`.
  4. Cancellation-aware and deadline-aware I/O must be implemented through D93's `Poller`-compatible readiness path, or through an equivalently explicit target-specific readiness mechanism specified by the toolchain contract.
  5. Kyokai does not use asynchronous thread cancellation, hidden signal injection, hidden scheduler wakeups, stack unwinding, or forced foreign-frame interruption to cancel a blocked syscall.
  6. If an underlying OS syscall returns because of an ordinary host interruption such as POSIX `EINTR`, the safe wrapper either retries transparently when no Kyokai cancellation/deadline condition is active, or returns the explicitly documented Kyokai cancellation/deadline/error case when such a condition has been observed.
  7. Cancellation remains D91 cooperative cancellation. It is a structured ordinary exit, runs `defer` according to D2b, and is not TPOE or `panic`.
  **Why this fits Kyokai**: the source code tells the truth about liveness. `readBlocking` means the programmer chose an unbounded wait, while `readUntil`/deadline/cancellation APIs spell the bounded exit path and route it through the explicit `Poller` model instead of inventing preemptive cancellation.
  **[STAGE: DECIDED_CORE_SEMANTICS | D237 → cooperative cancellation only through Poller-backed or explicitly readiness-backed operations; plain blocking calls may wait indefinitely]**
- **Kyokai's concurrency surface excludes language-level async/await and futures** — structured tasks, channels, cancellation, atomics, `select`, and the explicit `Poller` are the whole concurrency model. High-concurrency I/O is expressed through library layers built on that substrate rather than by adding a second colored effect language.
**Rules**:
  1. The language defines no `async fn`, `await`, async blocks, async closures, async typeclass methods, `Future`-like core abstraction, or implicit executor/runtime model.
  2. There is no hidden global event loop. D93's `Poller` plus libraries built on top of it are the sanctioned path for multiplexed I/O.
  3. D198 `yield` remains generator-only suspension. It is not generalized into task suspension or async control flow.
  4. Cancellation remains exactly D91 cooperative cancellation for structured tasks. The language defines no second async-task cancellation model.
  5. If Kyokai ever adds a language-level async facility, it requires a new explicit D-point family covering suspension points, borrow/live-value rules across suspension, cleanup of suspended state, executor/runtime ownership, FFI interaction, and backend lowering. No such facility exists implicitly.
  **Why this fits Kyokai**: D3/D91/D93 already provide one explicit concurrency story. Adding async/await would introduce a competing second mechanism, colored control flow, and a runtime model the language has not otherwise chosen.
  **[STAGE: DECIDED_CORE_SEMANTICS | D156 → structured concurrency only; no language-level async/await/futures/executors; high-concurrency I/O layers build over the explicit `Poller`]**
- **Safe signal handling is notification-based and pollable; arbitrary Kyokai signal handlers do not exist** — POSIX signal-handler contexts are too restricted for ordinary Kyokai code, so the safe surface converts signals into explicit readiness notifications instead of pretending user callbacks are sound there.
**Rules**:
  1. Safe Kyokai provides no API to register an arbitrary Kyokai function as a signal handler.
  2. The runtime's actual signal-handler entrypoint is an internal tiny async-signal-safe shim only. It may perform only the minimal signal-safe notification work needed to wake the safe watcher path; it may not run user code, allocate, touch ordinary linear values, or perform non-signal-safe I/O.
  3. The safe surface is a capability-gated `SignalWatcher` type that exposes a pollable readiness source compatible with D93's `Poller`.
  4. `SignalWatcher` covers only catchable external signals such as termination, interrupt, hangup, and user-defined signals. Safe Kyokai does not expose synchronous fault signals such as `SIGSEGV`, `SIGBUS`, `SIGILL`, `SIGFPE`, or `SIGABRT` through this API.
  5. Synchronous fault signals cause runtime-fatal process termination; they are not translated into recoverable Kyokai values, cancellation, TPOE, or ordinary `panic`.
  6. `SIGPIPE` from safe I/O is translated into an explicit broken-pipe I/O result where the target platform permits that behavior; safe Kyokai I/O must not unexpectedly terminate the process merely because the default host disposition for `SIGPIPE` would do so.
  7. Signal claims are process-global and exclusive per signal number. Attempting to create a second safe watcher for a signal already claimed by another safe watcher fails explicitly, for example with `SignalInUse`.
  8. Delivery is notification-based, not "run the handler body now." Repeated identical signals may be coalesced before observation; one readiness observation means "at least one such signal arrived" rather than "exactly one arrival happened."
  9. Destroying a `SignalWatcher` consumes it, releases its process-global signal claim, and restores the previous signal disposition.
  10. Raw signal-handler registration remains an unsafe-only / FFI-only facility under D20 for code that must work directly at the POSIX boundary.
  **Why this fits Kyokai**: Unix signal support exists, but the unsafe kernel/C boundary stays explicit and the safe language surface remains poll-driven, capability-gated, and free of hidden control-flow injections.
  **[STAGE: DECIDED_CORE_SEMANTICS | D95/D256 → capability-gated `SignalWatcher`; no safe arbitrary signal handlers; raw registration stays unsafe-only; synchronous fault signals are runtime-fatal]**
- **Select / Multi-Channel Waiting**: multiplexing channel readiness requires a dedicated block so code can wait on several communication events without inventing hidden priority or polling folklore.
  **Syntax**:
  ```kyokai
  select
      when recvBlocking(&!rx1) as Some(val) do
          // handle val
      when recvBlocking(&!rx_shutdown) as Some(ignore) do
          // handle shutdown
      when timeout(deadline) do
          // handle timeout
  pick;
  ```
  **Why `pick;` as the terminator**: D9 requires block terminators to be either reversed keywords or semantic boundary words. Reversing `select` yields `tceles;`, which is unreadable. `pick;` is the right boundary word because the block picks one ready communication event. `esac;` remains for pattern matching only and is not reused here.
  **Rules**:
  1. A `select` block contains one or more `when` arms and may contain at most one `timeout(deadline)` arm.
  2. Each non-timeout arm names exactly one admitted blocking channel operation, together with any explicit result-pattern binding surface defined for that operation.
  3. `select` waits until at least one arm becomes ready.
  4. When one or more arms are ready, exactly one ready arm is chosen and exactly one arm body executes.
  5. If multiple arms are simultaneously ready, the language guarantees no fixed source-order priority. The implementation may choose any ready arm, but "first arm wins" is not part of the language semantics.
  6. A `timeout(deadline)` arm becomes ready when its deadline is reached before any other chosen arm completes.
  7. Any values, borrows, or ownership transfers associated with non-selected arms do not occur.
  8. If a selected arm transfers ownership, that transfer occurs exactly as though the selected blocking operation had been written alone.
  9. The same linear endpoint may not appear in multiple arms of one `select`.
  10. `select` is part of the core language concurrency model, not a library macro or toolchain convention.
  **Why this fits Kyokai**: the control-flow boundary is explicit, the terminator says what the construct does, and multiplexed waiting becomes part of the same auditable concurrency story as channels, cancellation, and polling.
  **[STAGE: DECIDED_CORE_SEMANTICS | D92/D258 -> `select ... when ... do ... pick;` block for multi-channel waiting with explicit non-priority selection semantics]**
- **Cross-task ordering follows a closed, language-level synchronization model** — Kyokai does not treat “whatever the backend thread library happened to do” as a specification.
**Rules**:
  1. Only the synchronization edges explicitly enumerated by D90a create language-level cross-task ordering.
  2. Under the current language design, those edges arise only from three primitive families: structured child-task boundaries, D3a channel transfer/closure observation, and D3b atomic/fence synchronization.
  3. Any cross-task interaction not covered by those edges provides no ordering guarantee, even if some hardware or backend happens to behave more strongly.
  **Why this fits Kyokai**: concurrency behavior becomes auditable in the same way ownership and failure behavior are auditable. If an ordering guarantee exists, the spec names it.
  **[STAGE: DECIDED_CORE_SEMANTICS | D90 → closed formal synchronization model for structured tasks, channels, and atomics/fences]**
- **Kyokai's happens-before inventory is explicit, closed, and minimum-complete for the currently decided concurrency primitives** — the language states the exact cross-task edges rather than gesturing at “C++-like” behavior and leaving the rest to folklore.
**HB edges**:
  1. **HB1: spawn-start** — the spawn point of `spawn [captures] do ... od;` happens-before the first operation of that child task.
  2. **HB2: child-finish / structured join** — the last operation of a child task happens-before the parent continues past that child task's structured join point.
  3. **HB3: channel value transfer** — a successful `sendBlocking(v)` or successful `trySend(v)` on an SPSC channel happens-before the successful `recvBlocking()` or `tryRecv()` that returns that same value `v`. Because D3a channels are FIFO, this matching receive is unique.
  4. **HB4: sender-close observation** — `closeSender(sender)` happens-before the receiver observes channel closure after drain, either by `recvBlocking()` returning `None` or by `tryRecv()` returning `Disconnected`.
  5. **HB5: release/acquire atomic synchronization** — an atomic write or successful read-modify-write operation with `Release`, `AcqRel`, or `SeqCst` semantics happens-before an atomic load or successful read-modify-write operation with `Acquire`, `AcqRel`, or `SeqCst` semantics on the same atomic location when the later operation reads a value from the earlier release sequence.
  6. **HB6: SeqCst total order** — all `SeqCst` atomic operations and `SeqCst` fences participate in one single total order consistent with each task's program order and each atomic location's modification order.
  7. **HB7: standard fence synchronization** — `fence` uses the ordinary acquire/release/seqcst fence semantics equivalent to C11/C++20: release-fence plus following atomic publication, acquire-fence plus prior atomic observation, and fence-to-fence synchronization through a shared atomic publication/observation pair all establish happens-before exactly as in that model.
**Side rules**:
  1. The seven HB edges above are the complete set of happens-before guarantees in Kyokai's current concurrency model.
  2. Failed channel operations, including `trySend(Full(...))`, `sendBlocking(...)->Err(...)`, and `tryRecv(Empty)`, create no happens-before edge.
  3. `Relaxed` atomic operations provide atomicity and per-location coherence only. They create no happens-before edge by themselves.
  4. The compiler, optimizer, and backend may reorder operations within a task as long as single-task semantics and the listed HB edges are preserved.
  5. Any future synchronization primitive must add its own happens-before edges explicitly before it becomes part of the sound safe-concurrency surface.
  6. Sequential consistency is available through explicit `SeqCst` operations and fences, but Kyokai does not force every atomic operation to be sequentially consistent. Lower-level orderings are admitted only through D3b's explicit `MemoryOrder` argument and D3b's operation-specific validity rules.
  **Why this fits Kyokai**: the memory-order story stays as conventional as necessary for correctness, while the language-specific edges for structured tasks and channels are stated just as explicitly as the ownership rules around them.
  **[STAGE: DECIDED_CORE_SEMANTICS | D90a/D247 → closed seven-edge happens-before inventory for structured tasks, channels, atomics, and fences; explicit full memory-order hierarchy, not SeqCst-only]**
- **Backend policy: C stays, LLVM becomes primary later, and the language is not constrained by “must lower nicely to portable C”** — Kyokai keeps C emission because it is valuable for bootstrap, inspectability, toolchain leverage, and bring-up on awkward targets, but the existence of the C backend does not get veto power over the language's long-term semantics or feature set.
**Rules**:
  1. Kyokai language semantics are backend-independent. No language rule is defined as "whatever the C backend does" or "whatever LLVM happens to do."
  2. The C backend remains a first-class backend for bootstrap, reference implementation work, inspectable output, and target bring-up. It is not a temporary embarrassment to be discarded as soon as LLVM exists.
  3. Once the compiler is being written in Kyokai and the project can afford deeper backend work, the LLVM backend becomes the primary optimizing production backend.
  4. LLVM is the planned long-term home for features whose best implementation depends on direct SSA-level control, richer optimization pipelines, better debug info, or first-class vector support.
  5. The language design is NOT limited to features that can be expressed as clean, maximally portable, human-maintainable ISO C source. A feature is judged first on whether it is right for Kyokai; backend support is then solved explicitly.
  6. If a backend/target pair cannot support some feature with the required language semantics, that limitation must be stated explicitly in the backend/target support contract rather than solved by silently weakening the language.
  7. The C backend may use explicit C toolchain facilities such as standard intrinsics, implementation-defined extensions, or generated helper code when needed, provided the selected toolchain contract is explicit and valid Kyokai programs do not rely on C undefined behavior.
  8. "Kyokai emits C" is a backend capability, not a promise that generated C is Kyokai's stable public interchange format or that every advanced feature must round-trip into pretty portable C suitable for direct hand-maintenance.
  9. Near-term engineering priority remains language completion and self-hosting progress, not trying to out-build LLVM immediately. Keeping the C backend healthy serves that priority; later LLVM work serves the performance and feature ceiling.
  **Why this fits Kyokai**: C keeps the language grounded and portable, while LLVM gives the project a clear path to advanced optimization and target features. The important line is explicit: C remains part of Kyokai, but C does not own Kyokai.
  **[STAGE: DECIDED_CORE_SEMANTICS | D4 → C retained as bootstrap/reference/portability backend; LLVM becomes the long-term primary optimizing backend after self-hosting; backend constraints do not define the language]**
- **Hard fork** — Kyokai is not Austral. Borretti's philosophy ("intentionally minimal") is different from Kyokai's goal ("production-ready systems language"). The compiler is ~12K lines of OCaml — manageable for a hard fork. Cherry-picking upstream bugfixes is still possible, but Kyokai is its own project with its own direction. **[STAGE: DECIDED_CORE_SEMANTICS | D5 → Option A]**
- **Anonymous-by-default regions** — Kyokai makes `&[T]` a complete type meaning "borrow of T in an anonymous, scope-bounded region the compiler manages internally." There is no hidden slot, no elision rule, no inference — `&[T]` IS the type, the way `Int32` IS a 32-bit integer without anyone calling stack allocation "implicit." Named regions (`&[T, R]` with `generic [R: Region]`) remain available for the rare case (~1% of functions) where a return type references a region from an input parameter. Borretti himself identified the region parameter boilerplate as "the only downside" of Austral's borrow system and stated his intent to make reference types "just `&T`" (blog: `2023-06-12-second-class-references.md`, lines 139–141). Austral's own compiler ALREADY creates anonymous regions at expression level — `&x` in source calls `anonymous_region()` in `DesugarBorrows.ml` (lines 140–143) without the programmer naming anything. Option E extends this same principle from expressions to type signatures. Eliminates ~40 lines of `generic [R: Region]` headers from bfetchaust. Full research and compiler implementation notes in the D6 section below.
**Why Option E over Option A (fully explicit)**: Option A forces every function that takes a borrow to declare `generic [R: Region]` — pure boilerplate that adds zero semantic information in 99% of cases. The region parameter is not a choice the programmer makes; it's compiler bookkeeping. Forcing it on every signature violates the readability research (section 3.4): eye-tracking shows boilerplate is wasted fixation time.
**Why Option E over Option C (Rust-style elision)**: Option C has the compiler apply rules to fill in blanks (`_`), which IS implicit — "what the person wrote down isn't what's happening." Option E has no blanks to fill; the type is what you wrote.
**[STAGE: DECIDED_CORE_SEMANTICS | D6 → Option E, anonymous-by-default regions]**
- **Auto-reborrow**: when `out: &![T]` and a function expects `&![T]`, the compiler automatically inserts `&~` (reborrow). This is not hidden behavior - it follows the same tautology logic as D8 implicit `Unit` return. When a mutable reference is passed where a mutable reference is expected, exactly one valid operation exists.

  | Expression      | Meaning                                              | Valid? | Why                                                   |
  | --------------- | ---------------------------------------------------- | ------ | ----------------------------------------------------- |
  | `out` (consume) | Move the reference, can't use `out` again            | ❌      | You need `out` on the next line                       |
  | `&out`          | Immutable borrow of the reference itself             | ❌      | Wrong type - function expects `&![T]`, not `&[&![T]]` |
  | `&!out`         | New mutable borrow                                   | ❌      | `&!` takes owned values, not references               |
  | `&~out`         | Reborrow - new temporary reference from existing one | ✅      | Only valid option                                     |

  Since `&~` is the only thing that compiles, writing it explicitly adds zero bits of information. The complete borrow-insertion and read-reborrow table is:
  - `T` (owned) -> function expects `&![T]` -> compiler inserts `&!`
  - `T` (owned) -> function expects `&[T]` -> compiler inserts `&`
  - `&![T]` (mutable ref) -> function expects `&![T]` -> compiler inserts `&~`
  - `&![T]` (mutable ref) -> function expects `&[T]` -> compiler inserts a temporary immutable read reborrow of the referent
  - `&[T]` -> function expects `&![T]` is never legal

  Eliminates 200+ `&~out` tokens from bfetchaust. The programmer still sees the function signature declaring `&![T]` - mutability remains visible at the declaration site.
  **Why not keep explicit `&~` (Austral's approach)**: `&~` repeats information already encoded in the type system. When `out: &![ByteBuf]` and `appendByte` takes `&![ByteBuf]`, there is no second valid interpretation to distinguish.
  **Why not UFCS alone**: Pure UFCS without auto-reborrow provides near-zero benefit for bfetchaust's primary pattern. `(&~out).appendByte(27)` is the same character count as `appendByte(&~out, 27)` and just relocates the ceremony.
  **[DECIDED: D7b -> auto-reborrow, same logic as D8]**
- **Mutable-to-immutable borrow coercion is a read reborrow, not a subtype relation**: Kyokai allows `&![T]` where `&[T]` is expected, but this is specified as a temporary immutable reborrow of the referent rather than as general borrow subtyping.
  **Rules**:
  1. When an expression of type `&![T]` appears in a context expecting `&[T]`, the compiler may insert a temporary immutable reborrow of the referent.
  2. This is a coercion rule over expected-type positions, not a general subtype relation between `&![T]` and `&[T]`.
  3. The original mutable borrow remains suspended for the lifetime of the temporary immutable reborrow under the ordinary borrow rules.
  4. This coercion is legal in ordinary expected-type positions including call arguments, `let` initializers, return expressions, and branch/result type unification.
  5. `&[T]` to `&![T]` is never legal.
  6. No additional borrow coercions are implied beyond the explicit D7b/D187 table.
  **Why this fits Kyokai**: it captures the one obvious read-only interpretation programmers expect from Rust, Zig, and C++ style prior art, but it does so with a closed explicit coercion table instead of importing ambient variance or borrow subtyping machinery.
  **[STAGE: DECIDED_CORE_SEMANTICS | D187 -> mutable-to-immutable borrow coercion by temporary immutable read reborrow; not a subtype relation]**
- **UFCS dot syntax** — `expr.f(args)` is pure syntactic sugar for a call with `expr` as the first argument. The dot does not create virtual dispatch, implicit `self`, hidden allocation, or a second callable value model.
**What UFCS is**: Argument reordering plus explicit lookup. `out.appendByte(27)` lowers to a call whose first argument is `out`. The compiler first resolves `appendByte` through ordinary file-scope imports exactly as it would for `appendByte(out, 27)`. D254 adds one narrow fallback when ordinary lookup finds no candidate: receiver-module extension lookup over an explicit exported surface of the receiver type's defining module. There is no global method search, no dependency-wide ADL, no virtual dispatch, and no lookup based on arbitrary type signatures.
**What UFCS enables**: Subject-Verb-Object (SVO) ordering. Research (section 3.4) shows ~~80% of natural languages use SVO or SOV word order; Austral's `appendByte(&~~out, 27)` is VSO — the least common ordering. UFCS gives the programmer the OPTION to write SVO when it improves readability, while the function-call form remains valid for cases where VSO is clearer or where there's no natural subject.
**Where UFCS helps** (clear subject, repeated operations on same receiver):
  ```austral
  -- Before (VSO, with D6-E + D7b auto-reborrow + D12 integer inference):
  appendByte(out, 27);
  appendByte(out, '[');
  appendIndexDecimal(out, code);
  appendByte(out, 'm');

  -- After (SVO, with UFCS):
  out.appendByte(27);
  out.appendByte('[');
  out.appendIndexDecimal(code);
  out.appendByte('m');
  ```
  The SVO form makes it immediately clear that all four operations target `out`. The reader's eye tracks a vertical column of `out.` prefixes — the pattern is chunkable (section 3.4 finding: consistent shapes aid pattern recognition).
  **Where UFCS does NOT help** (no clear subject, multi-argument functions):
  ```austral
  -- These stay as regular function calls:
  writeAll(stdout_fd, &out);      -- neither argument is "the object"
  renderFetch(out, system_type, distro, kernel, ...);  -- 11 args, no natural subject
  ```
  **How the full bfetchaust `ansi` function looks with all decided changes (D6-E + D7a + D7b + D8 + D12)**:
  ```austral
  -- Current Austral:
  generic [R: Region]
  function ansi(out: &![ByteBuf, R], code: Index): Unit is
      appendByte(&~out, 27 : Nat8);
      appendByte(&~out, '[');
      appendIndexDecimal(&~out, code);
      appendByte(&~out, 'm');
      return nil;
  end;
  ```
  ```kyokai
  // Kyokai:
  function ansi(out: &![ByteBuf], code: Index): Unit is
      out.appendByte(27);
      out.appendByte('[');
      out.appendIndexDecimal(code);
      out.appendByte('m');
  qed;
  ```
  8 lines → 6 lines. Every remaining token carries information. The generic header is gone (D6-E). The `&~` tokens are gone (D7b). The `: Nat8` annotations are gone (D12). The `return nil` is gone (D8). The SVO ordering is readable (D7a). Nothing is hidden — `out: &![ByteBuf]` in the signature says "mutable borrow," and every `.appendByte(27)` call is visibly resolved by ordinary imports or D254's explicit receiver-module fallback with `out` as first argument.
  **Parser implementation** (`Parser.mly`): Currently `PERIOD identifier` produces `CSlotAccessor` (field access, line 498). UFCS requires a new parse rule: when `PERIOD identifier argument_list` follows an `atomic_expression`, it produces a `CMethodCall` AST node that desugars to `CFuncall(identifier, expr :: args)` in the abstraction pass. The `path` grammar rule (line 488) needs extending to allow `path_rest` entries that include argument lists, distinguishing field access (`obj.field`) from UFCS calls (`obj.func(args)`) by the presence of parentheses.
  **[STAGE: DECIDED_CORE_SEMANTICS | D7a/D254 → UFCS argument reordering with ordinary lookup plus narrow receiver-module extension fallback]**
- **UFCS conflict handling is ordinary call conflict handling plus D254's narrow receiver-module fallback** — the dot form does not gain C++-style ADL, import-order priority, or global method search.
**Rules**:
  1. If ordinary imported-name lookup for `expr.f(args)` finds a unique candidate, the call lowers exactly as `f(expr, args)`.
  2. If ordinary imported-name lookup finds multiple candidates, the UFCS form is ambiguous and the compiler rejects it. The receiver-module fallback is not used to override an import collision.
  3. If ordinary imported-name lookup finds no candidate, D254 may search the receiver type's defining module for explicitly exported receiver-callable functions for that type.
  4. The programmer disambiguates with an ordinary qualified function call or with an explicit import rename. UFCS does not invent local method aliases or hidden precedence rules.
  5. UFCS does not override typeclass dispatch; receiver-module lookup is a name-resolution fallback for ordinary functions only.
  **Why this fits Kyokai**: the dot form stays honest call sugar while solving the repeated `length`/`isEmpty` import-collision problem without importing C++-style ADL folklore.
  **[STAGE: DECIDED_CORE_SEMANTICS | D110/D254 → UFCS ambiguity is ordinary call ambiguity; receiver-module lookup is a no-import fallback only]**
- **No pipeline operator** — Kyokai does not add `|>`. UFCS is the language's one built-in left-to-right call-chain surface, and a second spelling for the same first-argument flow would create needless style bifurcation.
**Rules**:
  1. `|>` is not part of Kyokai syntax.
  2. Left-to-right chaining is expressed with UFCS `expr.f(args)` when the underlying call shape is first-argument application.
  3. When a call does not fit UFCS naturally, the programmer writes the ordinary function-call form.
  **Why this fits Kyokai**: one visible left-to-right call style is enough, and Kyokai does not need a second pipeline surface that overlaps almost perfectly with UFCS.
  **[STAGE: DECIDED_CORE_SEMANTICS | D108 → reject `|>`; UFCS remains the sole built-in left-to-right chaining sugar]**
- **Implicit Unit return** — functions declared `: Unit` do not require `return nil;` at the end of their body. When execution reaches the end of a `: Unit` function, the function returns `Unit`. This is NOT implicit return of arbitrary expressions (like Rust's last-expression-is-return-value). It applies ONLY to `: Unit` functions, ONLY at the end of the function body, and ONLY because `Unit` has exactly ONE inhabitant (`nil`). `return nil` communicates zero bits of information — the function signature `: Unit` already declares there is nothing to return, and `nil` is the only value of type `Unit`. This is the same principle as auto-reborrow (D7b): when there is exactly one valid operation, requiring the programmer to write it explicitly is ceremony, not explicitness. Explicit early returns (`return nil;` or `return;` mid-function) remain required because they signal that control flow is leaving the function body early — that IS meaningful information. Eliminates 18 `return nil;` lines from bfetchaust.
**Why not keep `return nil;` required (Austral's approach)**: `return nil` is a tautology in information theory — it states the only possible outcome. The function signature already makes the return type explicit. Requiring `return nil` is like requiring you to write `if true then ... end if` — technically more explicit, but the explicitness carries no information.
**Why not implicit return of expressions (Rust-style)**: Rust's `fn foo() -> i32 { 42 }` where the last expression is the return value is a DIFFERENT feature. That's implicit because the programmer chose not to write `return`. Kyokai's implicit Unit return is not a choice — there is only one value of type `Unit`, so there is nothing to choose. This distinction matters: Kyokai will NOT adopt Rust-style last-expression-as-return for non-Unit functions.
**[STAGE: DECIDED_CORE_SEMANTICS | D8 → Option B, implicit Unit return]**
- **Two-category block terminators** — Kyokai replaces Austral's generic `end X;` terminators with a system split into two categories, each with a clear rationale.
**Category 1: Pure Control Flow — Reversed Keywords (Algol 68 style).** For branching and looping, the closing keyword is the reverse spelling of the opening keyword. This is battle-tested from Algol 68 (the grandfather of Bash's `if/fi`, `case/esac`).

  | Construct     | Open               | Close   | Origin                   |
  | ------------- | ------------------ | ------- | ------------------------ |
  | Conditional   | `if ... then`      | `fi;`   | Algol 68                 |
  | Pattern match | `case ... of`      | `esac;` | Algol 68                 |
  | Loops         | `while/for ... do` | `od;`   | Algol 68 (`do` reversed) |

  `fi` closes the entire `if/else if/else` chain — one terminator for the whole construct, no nesting ambiguity:
  ```kyokai
  if ty != Linux then
      return;
  else if ty == Gentoo then
      gentooArt(out);
  else
      defaultArt(out);
  fi;
  ```
  Both `for` and `while` loops use `do` to open the body, so `od` closes both — the terminator matches the body-initiator, not the outer keyword:
  ```kyokai
  for i from 0 to 10 do
      out.appendByte(i);
  od;

  while running do
      poll();
  od;
  ```
  **Category 2: State & Boundaries — Semantic Boundary Words.** You can't reverse `module` (`eludom`) or `borrow` (`worrob`) — they look like typos. Instead, Kyokai uses terminators that describe what is happening at the boundary. Since Kyokai literally means "boundary" (境界), these terminators make the resource/scope lifecycle explicit.

  | Construct   | Open                  | Close    | Semantic meaning                                                                                                                                                                                               |
  | ----------- | --------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | Functions   | `function ... is`     | `qed;`   | "quod erat demonstrandum" — proof complete. Functions in Kyokai ARE proofs: the linearity checker proves every resource is consumed. 3 chars, completely unambiguous, no collision with any identifier.        |
  | Methods     | `method ... is`       | `qed;`   | Methods are functions. Same terminator.                                                                                                                                                                        |
  | Instances   | `instance ... is`     | `qed;`   | The MOST mathematically justified use — an instance is a **proof** (witness) that a type satisfies a typeclass contract.                                                                                       |
  | Type defs   | `record/union ... is` | `build;` | Type definition constructed. Both records and unions are type constructions at the compiler level — splitting them onto different terminators adds a rule to memorize for zero gain. 6 chars.                  |
  | Typeclasses | `typeclass ... is`    | `spec;`  | A typeclass declares a contract/specification, not an implementation. 5 chars, unambiguous.                                                                                                                    |
  | Borrows     | `borrow ... do`       | `drop;`  | The reference is dropped. `drop` is the established term in linear/affine type systems (Rust, etc.). 5 chars. `release` was considered but rejected — too natural a function name in resource management code. |
  | Modules     | `module body ... is`  | `seal;`  | The module's symbol table is sealed. 5 chars vs Austral's `end module body.` (16 chars).                                                                                                                       |
  | FFI blocks  | `foreign "C" is`      | `mon;`   | The foreign boundary is a gate/portal (`門`). `mon;` marks that the raw ABI gateway is now closed and keeps the FFI boundary visually distinct from proof bodies.                                              |

  **Full example showing both categories together:**
  ```kyokai
  module body Kyokai.Fetch is

      record ByteBuf is
          data: Address[Nat8];
          capacity: Index;
          size: Index;
      build;

      typeclass Serialize[T] is
          method serialize(val: &[T]): ByteBuf;
      spec;

      instance Serialize[Point] is
          method serialize(val: &[Point]): ByteBuf is
              let buf: ByteBuf := makeByteBuf(64);
              buf.appendByte(val.x);
              buf.appendByte(val.y);
              return buf;
          qed;
      qed;

      function ansi(out: &![ByteBuf], code: Index): Unit is
          out.appendByte(27);
          out.appendByte('[');
          out.appendIndexDecimal(code);
          out.appendByte('m');
      qed;

      function main(): ExitCode is
          var out: ByteBuf := makeByteBuf(16384);
          defer out.destroyByteBuf();

          if showHelp then
              out.appendHelp();
          else if showVersion then
              out.appendVersion();
          else
              fetchAndRender(out);
          fi;

          writeAll(stdout_fd, &out);
          return ExitSuccess();
      qed;

  seal;
  ```
  **Why not Austral's `end X;` (Option A)**: `end if; end for; end if; end for; end;` is verbose boilerplate that carries information only when nesting is deep. Kyokai code is shallow by design (linear types prevent deep branching). The verbosity cost is paid on every construct; the readability benefit is collected only at deep nesting that almost never occurs.
  **Why not braces `{}` (Option D)**: Braces work but are generic — `}` tells you nothing about what it closes. Kyokai's terminators are self-documenting: `fi` tells you an `if` ended, `qed` tells you a proof ended, `drop` tells you a borrow ended, and `mon` tells you a foreign gateway ended. This is strictly more informative than `}` at zero extra cost (most terminators are 2–5 chars).
  **Why not uniform `end;` (Option B/C)**: Gets the worst of both worlds — verbose like Ada but uninformative like braces.
  **`build` keyword collision note**: `build` conflicts with the builder pattern (`buf.build()`). Convention: name builder-finalization functions `new` instead. This is already natural in Kyokai since constructors are just functions.
  **[STAGE: DECIDED_CORE_SEMANTICS | D9 → Two-category terminators: reversed keywords (`fi`/`od`/`esac`) + semantic boundary words (`qed`/`build`/`spec`/`drop`/`seal`/`mon`/`pick`/`join`/`audit`)]**
- **The D111 revisit keeps the existing terminator names intact; later D127 adds `mon;` for foreign blocks without renaming the already decided spellings** — borrow scopes continue to close with `drop;`, and Kyokai does not introduce synonym terminators for the existing constructs.
**Rules**:
  1. `drop;` remains the borrow-scope terminator.
  2. Kyokai does not add `unborrow` as a second borrow-scope terminator spelling.
  3. The terminator inventory now stands at `fi;`, `esac;`, `od;`, `qed;`, `build;`, `spec;`, `drop;`, `seal;`, `mon;`, `pick;`, `join;`, and `audit;` under the current language design.
  **Why this fits Kyokai**: the terminator system is already coherent, and renaming one piece would add churn without resolving any real semantic problem.
  **[STAGE: DECIDED_CORE_SEMANTICS | D111 → keep D9 unchanged; `drop` retained]**
- **Not-equal operator: `!=`** — Kyokai switches from Austral's `/=` (Ada/Haskell convention) to `!=` (C/Rust/Zig/Go/Python/JavaScript convention). `/=` is used by exactly two actively maintained languages (Haskell and Ada) and is visually ambiguous — it looks like "divide-equals" (Python's `/=` IS a divide-and-assign operator). `!=` is universally recognized; the cognitive cost of parsing it is zero. Trivial compiler change (lexer token swap).
**Why not keep `/=`**: Creates genuine confusion for programmers from Python/JavaScript/etc. where `/=` means divide-assign. Kyokai gains nothing from preserving Ada heritage in a symbol that 99% of programmers read incorrectly on first encounter.
**[STAGE: DECIDED_CORE_SEMANTICS | D10 → Option B, switch to `!=`]**
- **Keep `camelCase` naming** — Kyokai keeps Austral's naming convention: `camelCase` for functions/variables, `PascalCase` for types and modules. Research (Binkley et al. 2009) shows `camelCase` has slightly higher accuracy, and Sharif & Maletic (2010) shows `snake_case` is slightly faster on first fixation — but the difference is small; **consistency dominates both**. Changing from Austral's convention would force every existing Austral program to be rewritten for zero measurable benefit.
**Why not `snake_case` (Option B)**: The research difference is negligible. Consistency matters more than convention choice. Austral is already consistent with `camelCase`. Migration cost is high, benefit is near-zero.
**[STAGE: DECIDED_CORE_SEMANTICS | D11a → Option A, keep camelCase]**
- **Ownership-signaling naming patterns** — Kyokai adopts Rust's API naming conventions (adapted to camelCase) as a mandatory convention for the standard library and recommended for user code. These patterns encode **cost, ownership, and allocator semantics** directly into function names, which is especially powerful in a linear type system where the most important question at any call site is "does this consume my resource, borrow it, or allocate a fresh owned result?"
**The rules**:
  1. `**as*`** (e.g., `asSpan`, `asBytes`): Free conversion. Borrows input (`&[T]`), returns a view or borrow. **No allocation, no ownership transfer.**
  2. `**to*In`** (e.g., `toStringIn`, `toBufferIn`): Expensive borrowing conversion. Borrows input (`&[T]`), takes an explicit destination allocator, and returns a new owned value. **Allocates memory, no ownership transfer.**
  3. `**into*`** (e.g., `intoBuffer`, `intoBytes`): Consuming conversion. Takes input by value (`T`), returns a new type. **Consumes the original resource, and is reserved for conversions that transfer/reuse storage or otherwise do not need a fresh destination allocator.**
  4. `**into*In`** (e.g., `intoBufferIn`, `intoStringIn`): Consuming conversion that still needs fresh destination allocation. Takes input by value plus an explicit destination allocator. **Consumes the source and allocates the result.**
  5. `**make*`** (e.g., `makeByteBuf`): Default constructor. Creates a new resource from scratch. (Austral already does this.)
  6. `**from*`** (e.g., `fromSpan`): Conversion constructor. Creates a new resource by copying/parsing another. If it allocates, the allocator must still be explicit per D201.
  7. **No `get` prefix**: Getters drop the `get` — `buf.length()` not `buf.getLength()`. Actions use verbs (`buf.clear()`).
  **Why this is a philosophical home run for Kyokai**: The language is literally named "Boundary" (境界) and focuses on strict resource management. Baking ownership boundaries into the VOCABULARY means the call site tells you the ownership story before you even read the signature. Combined with UFCS (D7a), it makes Kyokai code read as natural data pipelines: `string.asBytes().intoBuffer()`.
  **Enforcement: compiler warnings, not an external linter.** Kyokai's compiler already knows the function signatures — it knows whether a function takes `T` (consumes) vs `&[T]` (borrows). So the compiler can verify that names match behavior:
  - `into*` function whose first param is `&[T]` (not consuming) → **warning**: "function named `into*` should consume its first argument"
  - `as*` function that returns a `Linear` type (allocates) → **warning**: "function named `as*` should return a Free view, not allocate"
  - `to*In` function that does not take an explicit destination allocator → **warning**: "allocating borrowed conversion should use `...In` and take an explicit allocator"
  - `to*In` function that consumes its first argument → **warning**: "consuming function should be named `into*`/`into*In`, not `to*In`"
  - `into*` function that may allocate a fresh destination result → **warning**: "allocating consuming conversion should be named `into*In`"
  No external linter needed. The compiler IS the linter. This fits Kyokai's "one tool, the compiler" philosophy. These are warnings, not errors — the programmer can ignore them with good reason.
  **[DECIDED: D11b → Adopt ownership-signaling naming patterns, compiler-warned]**
- **Bidirectional integer literal inference** — when a numeric literal appears in a position where the expected type is known (function parameter, binary operator operand, variable initializer with explicit type annotation), the compiler infers the literal's type from context. `makeByteBuf(16384)` instead of `makeByteBuf(16384 : Index)`. This is NOT full type inference — it pushes a KNOWN type from a declaration into a literal. The type is visible in the function signature. Ambiguous cases (multiple valid types) still require explicit annotation.
**Why not keep fully explicit (Option A)**: `(16384 : Index)` restates information already in the function signature. The parameter is declared `cap0: Index` — writing `: Index` again at the call site adds zero information. Same principle as D7b (auto-reborrow) and D8 (implicit Unit return): when there is exactly one valid interpretation, requiring the programmer to restate it is ceremony.
**[STAGE: DECIDED_CORE_SEMANTICS | D12 → Option B, bidirectional inference for literals]**
- **Numeric literals may use `_` separators as purely lexical readability markers** — Kyokai adopts the modern consensus spelling for large constants while keeping the lexer rules fully explicit.
**Rules**:
  1. `_` may appear only between two digits in the same digit run of an integer or floating literal.
  2. `_` may not appear at the start or end of a literal, immediately after a base prefix (`0x`, `0o`, `0b`), or adjacent to a decimal point or exponent marker.
  3. Separators do not affect the value; the lexer discards them after validation.
  4. Examples: `1_000_000`, `0xDEAD_BEEF`, `0b1010_0101`, `3.141_592`, `1.0e10_0`.
  **Why this fits Kyokai**: the feature is lexical only, universally understood by modern systems programmers, and adds no hidden semantics beyond clearer digit grouping.
  **[STAGE: DECIDED_CORE_SEMANTICS | D135 → `_` separators between digits in integer and floating literals under tight lexical rules]**
- **Numeric literal suffixes are explicit built-in literal typing, not C-style conversion folklore** — suffixes complement D12's contextual literal inference for constants whose expected type is not otherwise visible or where the type is part of the constant's meaning.
**Rules**:
  1. Integer literals may use exactly these suffixes: `i8`, `i16`, `i32`, `i64`, `n8`, `n16`, `n32`, `n64`, and `index`.
  2. The suffixes map respectively to `Int8`, `Int16`, `Int32`, `Int64`, `Nat8`, `Nat16`, `Nat32`, `Nat64`, and `Index`.
  3. Floating-point literals may use exactly `f32` and `f64`, mapping to `Float32` and `Float64`.
  4. A suffixed literal has the suffixed type before D12 contextual inference. Context may confirm that type, but it may not silently retarget the literal to another type.
  5. If the literal value is not representable in the suffixed type, compilation fails for the comptime-known literal.
  6. Literal suffixes do not create implicit numeric conversions, promotions, signedness changes, or C-style partial suffix composition.
  7. `_` digit separators from D135 remain lexical separators and may appear inside the digit run before the suffix, never inside the suffix itself.
  **Why this fits Kyokai**: the programmer can write compact fixed-width constants without restating a bulky annotation, but the suffix set is closed, readable, and has no hidden conversion behavior.
  **[STAGE: DECIDED_CORE_SEMANTICS | D261 → closed numeric literal suffix set for fixed-width integer, `Index`, and floating literals]**
- **Keep `do` in pattern matching** — Kyokai keeps Austral's `case x of when Foo do ... esac;` syntax with `do` instead of switching to `=>`. This keeps the symbol count low — `do` is a keyword (searchable, pronounceable, consistent with `for/while ... do`), while `=>` is a symbol that adds visual noise in a language already using `->` for something else. The `do` keyword is also already established in D9's terminator system (`do`/`od` for loops) — reusing it for `when` clauses maintains internal consistency.
**Why not `=>` (Ada convention)**: Even though Ada uses `=>`, Kyokai is not Ada. `=>` adds another symbol to memorize, and Kyokai already uses `->` (function return type arrow). Keeping `do` means one less symbol in the language. The `when Foo do` reads naturally as English: "when it's Foo, do this."
**Why not `match` (Option C)**: `case`/`when` is already Austral's vocabulary. Changing the keyword gains nothing and breaks compatibility.
**[STAGE: DECIDED_CORE_SEMANTICS | D13 → Keep `do` in when-clauses]**
- **Keep reference/reborrow syntax** — Kyokai keeps Austral's reference operators unchanged: `&x` (immutable borrow), `&!x` (mutable borrow), `~x` (dereference), `&~x` (reborrow). The `~` vs `*` debate is cosmetic — changing it would affect 200+ occurrences for zero semantic benefit. With UFCS (D7a) and auto-reborrow (D7b) already decided, most `&~` occurrences are eliminated anyway (UFCS removes the explicit first argument, auto-reborrow handles the remaining cases). The few remaining explicit uses of `~` and `&~` are infrequent enough that the syntax choice is a non-issue.
**Why not change `~` to `*`**: Kyokai is not C and is not Rust. `~` is Austral's operator, it works, it's compositional (`&` + `~` = `&~`), and UFCS/auto-reborrow make it rare in practice. Changing it is churn for zero gain.
**[STAGE: DECIDED_CORE_SEMANTICS | D14 → Keep current reference/reborrow syntax]**
- **Two-layer error propagation** — Kyokai introduces a two-layer system for error handling on `Result[T, E]` types, solving the "arrow anti-pattern" (nested `case`/`esac`) without hiding any control flow or data construction.
**Layer 1: `let...else` (the foundation — 100% explicit)**
A diverging pattern match that flattens nesting by binding the success variant in the outer scope and requiring the else block to diverge (`return`, `break`, `continue`, or `panic`). Works for ANY two-variant union type (Result, Optional, custom unions).
  ```kyokai
  // Full form (multi-statement else block):
  let Ok(file) := openFile(path) else Err(e) do
      logError(e);
      return Err(e);
  fi;

  // Common form (single-statement):
  let Ok(file) := openFile(path) else Err(e) do return Err(e); fi;

  // Works for Optional too:
  let Some(value) := lookupTable(key) else None do return defaultResult; fi;
  ```
  Rules:
  1. **Two-variant limitation**: Valid ONLY for union types with exactly two variants.
  2. **Forced divergence**: The compiler enforces that the `else` block MUST diverge. If it doesn't, it's a compile error — not a lint, a hard error.
  3. **Zero magic**: The compiler does not implicitly cast errors, construct variants, or auto-return. The programmer writes everything.
  4. **Terminator**: `do`/`fi` — the `else` block is a conditional divergence (if pattern doesn't match → diverge), so `fi` (the conditional terminator from D9) closes it.
  5. **Scope**: The successfully matched variable (`file`, `value`) is bound in the OUTER scope after the `fi;`, not inside a nested block.
  **Layer 2: `or return` / `or break` / `or continue` (defined sugar)**
  Syntactic sugar with a PRECISE, TRACEABLE desugaring into the `let...else` form. The programmer opts into the sugar — it's never forced.
  ```kyokai
  // Sugar:
  let file: File := openFile(path) or return;
  let file: File := openFile(path) or return err => ConfigError.Io(err);
  // Desugars EXACTLY to:
  let Ok(file) := openFile(path) else Err(__e) do return Err(__e); fi;
  ```
  The `or` family:

  | Operator                    | Context                     | Desugaring                                        |
  | --------------------------- | --------------------------- | ------------------------------------------------- |
  | `or return`                 | Function returning `Result` | `else Err(e) do return Err(e); fi;`               |
  | `or return name => expr`    | Function returning `Result` | `else Err(name) do return Err(expr); fi;`         |
  | `or break`                  | Inside a loop               | `else Err(ignore) do break; fi;`                  |
  | `or continue`               | Inside a loop               | `else Err(ignore) do continue; fi;`               |

  Restrictions (statement-level only):
  - `or return` can ONLY appear at the end of a `let` binding statement.
  - NOT an expression-level operator — cannot appear inside subexpressions.
  - There is no implicit error conversion. A type change requires the explicit-binder form `or return name => expr`.
  - Grammar: `let ID : TYPE := EXPR or return ;` | `let ID : TYPE := EXPR or return ID => EXPR ;` | `let ID : TYPE := EXPR or break ;` | `let ID : TYPE := EXPR or continue ;`
  **Why `or return` hides `Err(e)` construction — and why it's acceptable:**
  `or return` implicitly constructs `Err(e)` in the function's return type. This IS implicit data construction. But by the D7b/D8 principle ("when there is exactly ONE valid interpretation, requiring the programmer to restate it is ceremony"), the `Err(e)` wrapping is the ONLY valid operation — there's nothing else to return. The `let...else` form exists as the escape hatch for when you need full control (logging, error mapping, cleanup).
  **Why two layers instead of just one:**
  1. `let...else` alone is verbose for the 90% case (simple propagation). 3 lines × 5 ops = 15 extra lines.
  2. `or return` alone has no escape hatch for complex error handling. You fall back to full `case`/`esac` (deeply nested).
  3. Together: three levels of control — `or return` (1 line, sugar), `let...else` (3 lines, fully explicit), `case`/`esac` (multi-branch). Each fills a gap.
  **Prior art:**
  - **Rust**: Has BOTH `?` (sugar) AND `let...else` (RFC 3137, stabilized 1.65). They coexist — proving even with sugar, the explicit form is wanted.
  - **Swift**: `guard let x = expr else { return }` — diverging pattern match since Swift 1.0, considered one of the language's best features.
  - **Odin**: GingerBill tried prefix `try`, removed it because it "optimizes for typing not reading", replaced with suffix `or_return` — 350+ uses in `core:math/big` alone.
  - **Borretti**: "all potentially-throwing operations return union types. This can be made less onerous through syntactic sugar." He explicitly validated sugar for linear type systems.
  `**defer` vs `errdefer` patterns (prerequisite from D2):**
  ```kyokai
  // Pattern A: Acquire-Use-Release (use defer — consumed on ALL exits)
  function readConfig(path: Span[Nat8, Static]): Result[Config, IoError] is
      let file: File := openFile(path) or return;
      defer closeFile(file);
      let content: String := readAll(&file) or return;
      return Ok(parseConfig(&content));
  qed;

  // Pattern B: Build-and-Return (use errdefer — consumed on ERROR exits only)
  function openConfigFile(path: Span[Nat8, Static]): Result[File, IoError] is
      let file: File := openFile(path) or return;
      errdefer closeFile(file);
      validateMagicBytes(&file) or return;
      return Ok(file);  // file consumed here on happy path
  qed;
  ```
  The linearity checker requires two new variable states beyond Austral's current four: `Deferred` (registered with `defer`, consumed on ALL exits, borrows allowed) and `ErrDeferred` (registered with `errdefer`, consumed on ERROR exits only, must be explicitly consumed on happy path). This is a D2 implementation detail that must be specified before D15 can work.
  **[STAGE: DECIDED_CORE_SEMANTICS | D15 → Two-layer error propagation: `let...else` base + `or return` sugar]**
- **`defer` and `errdefer` create explicit linearity-checker states, not hidden reference capture** — cleanup is registered where the programmer writes it, and the checker records which future exit path consumes the linear value.
**Rules**:
  1. `defer action(value);` registers a consuming scope-exit action immediately. Any linear value consumed by that deferred action enters the `Deferred` checker state at the `defer` statement.
  2. A `Deferred` value may be borrowed under the ordinary borrow rules, but it may not be moved, consumed by another operation, reassigned, returned, sent, or registered in another consuming cleanup action.
  3. On every structured scope exit that runs ordinary cleanup, `defer` actions execute in reverse registration order.
  4. Deferred actions run after borrow scopes that were live in the body have ended. A deferred action may not capture or depend on a borrow whose lifetime does not reach the deferred action's execution point.
  5. `errdefer action(value);` registers a consuming cleanup action for structured error exits only. Any linear value consumed by that action enters the `ErrDeferred` checker state at the `errdefer` statement.
  6. An `ErrDeferred` value may be borrowed under ordinary borrow rules. On the success path, the program must explicitly consume, move, return, or otherwise discharge the value before the scope exits.
  7. On a structured error exit, `errdefer` actions execute before ordinary `defer` actions from the same scope, with each category executing in reverse registration order.
  8. `errdefer` does not run on ordinary success completion, `break`, `continue`, `panic`, or TPOE unless another already-decided rule classifies that exit as a structured error exit for the relevant scope.
  9. TPOE remains immediate hard termination under D84 and does not run user deferred actions.
**Why this fits Kyokai**: the programmer still writes every cleanup action explicitly, but the compiler gets exact states for values that have already been promised to future cleanup.
  **[STAGE: DECIDED_CORE_SEMANTICS | D246 → `Deferred` and `ErrDeferred` checker states; defer captures by immediate cleanup registration, not by late reference evaluation]**
- `**let...else` and the `or ...` sugars have exact typing and linearity rules** — Kyokai does not leave these control-flow forms to ad hoc checker special cases.
**Rules for `let...else`:**
  1. In `let P := E else Q do B fi;`, the scrutinee expression `E` is evaluated exactly once.
  2. `let...else` is legal only if `E` has a two-variant union type.
  3. The success pattern `P` must be refutable for the type of `E`. An irrefutable pattern makes the construct illegal.
  4. The success pattern `P` and else pattern `Q` must be exhaustive over the two variants of `E`'s type.
  5. The else block `B` must type-check as `Never`.
  6. An expression does not satisfy rule 5 merely because it may trigger TPOE at runtime. Potential contract failure is not the same thing as statically known divergence.
  7. Names bound by `P` enter scope only after `fi;`.
  8. Names bound by `Q` exist only inside the else block.
  9. All bindings introduced by `P` and `Q` obey D60's no-shadowing rule.
  **Linearity rules:**
  1. The union value produced by `E` is consumed exactly once by the pattern test.
  2. On the success path, any linear payload bound by `P` becomes the outer binding after `fi;` and must be consumed on that path in the ordinary way.
  3. On the else path, any linear payload bound by `Q` must be consumed inside `B`.
  4. Because `B` has type `Never`, there is no post-`fi` branch join that must reconcile else-path linear bindings with the success path.
  **Rules for `or return` / `or break` / `or continue`:**
  1. These are statement forms only. They may appear only at the end of a `let` binding statement.
  2. They are defined by exact desugaring, not by custom checker behavior.
  3. The `or ...` sugars apply only to `Result[T, E]`, not to arbitrary two-variant unions.
  4. Plain `or return` is legal only inside a function whose return type is `Result[U, E]` with the same error type `E`.
  5. `or return name => expr` is legal only inside a function whose return type is `Result[U, F]`, where the scrutinee has type `Result[T, E]`, `name` binds the inner error payload inside `expr`, and `expr` has type `F`.
  6. `or break` and `or continue` are legal only inside loop scopes.
  7. `or break` and `or continue` are legal only when the `Err` payload type is `Free`. If the error payload is `Linear`, the programmer must use `let...else` and explicitly consume that payload before `break` or `continue`.
  8. There is no implicit error conversion. Any type change on `or return` exists only because the programmer wrote an explicit mapping expression.
  9. For `errdefer` and error-path analysis, `or return` counts as a structured error exit because it is exact sugar over a `return Err(...)` path. `or break` and `or continue` do not count as error exits for `errdefer`; they are ordinary loop-control exits.
  **Exact desugarings:**
  ```kyokai
  let x: T := expr or return;
  ```
  desugars to:
  ```kyokai
  let Ok(x) := expr else Err(__e) do return Err(__e); fi;
  ```
  ```kyokai
  let x: T := expr or return err => mapError(err);
  ```
  desugars to:
  ```kyokai
  let Ok(x) := expr else Err(err) do return Err(mapError(err)); fi;
  ```
  ```kyokai
  let x: T := expr or break;
  ```
  desugars to:
  ```kyokai
  let Ok(x) := expr else Err(ignore) do break; fi;
  ```
  ```kyokai
  let x: T := expr or continue;
  ```
  desugars to:
  ```kyokai
  let Ok(x) := expr else Err(ignore) do continue; fi;
  ```
  **Why this fits Kyokai**: the user-facing forms stay compact, but the type checker, linearity checker, and defer/error-path behavior all follow one explicit set of rules with no hidden branch semantics.
  **[STAGE: DECIDED_CORE_SEMANTICS | D15a → exact typing, scope, desugaring, path-local linearity rules, and explicit-binder inline error mapping for `let...else` / `or ...`]**
- **Keep semicolons** — Semicolons stay. Borretti's argument is sound: semicolons provide redundancy that aids both reading and parser error recovery. `let x : T := f(a, b, c;` — the parser detects the missing `)` immediately at the `;`. Kyokai's statement-oriented, keyword-heavy syntax is complemented by semicolons, not hindered by them. Removing them would require grammar changes and introduce ambiguity risks for zero real benefit.
**Why not remove them**: The cognitive cost of typing `;` is negligible. The "modern feel" argument is aesthetic, not functional. Kyokai's syntax is already distinctive through its keywords and terminators (D9) — removing semicolons would not make it feel more modern, just more ambiguous.
**[STAGE: DECIDED_CORE_SEMANTICS | D16 → Keep semicolons]**
- **Compile-time evaluation: `constant` expressions + `comptime` call-site keyword** — Two tools for two jobs. `constant` declarations are extended to support arithmetic/bitwise expressions over other constants (emitted as C constant expressions — zero compiler cost). `static_assert(expr, "msg")` validates invariants at compile time (emitted as `_Static_assert`). For computed constants (lookup tables, precomputed arrays), the `comptime` keyword at the **call site** forces compile-time evaluation of any pure function over `Free` types. This is Zig's model, not Rust's: the programmer writes `comptime computeEscapeTable()`, making the phase determination explicit and visible in source. There is no `const fn` annotation on function definitions — a function is a function, and whether it runs at compile time or runtime is determined where it's called. Only `Free`-universe types are eligible for `comptime` evaluation (linear types represent runtime resources and cannot exist at compile time). Kyokai's existing `Free`/`Linear` universe distinction makes this check trivial — the compiler already knows which universe every type belongs to.
**Why `comptime` call-site, not `const fn` definition-site**: Rust's `const fn` creates implicit phase determination — the same function call silently evaluates at compile time in a `const` block and at runtime in a `let` block. Identical syntax, completely different execution. This violates Kyokai's principle: "if two programs look identical, they behave identically." The `comptime` keyword makes the phase visible, the same way `defer` makes scope-exit visible.
**Why both D and B together**: They serve different purposes and don't compete. `constant` handles simple named values (`constant PAGE_SIZE: Index := 4096;`). `comptime` handles computed values (`constant ESCAPE_TABLE: Array[Bool, 256] := comptime computeEscapeTable();`). Linear types make `comptime` simpler to implement than Rust's `const fn` was — there's no need for a "const-safe subset" analysis because the `Free`/`Linear` boundary already exists.
**Implementation**:
  ```kyokai
  // Simple constants (Option D):
  constant PAGE_SIZE: Index := 4096;
  constant MAX_HEADERS: Index := PAGE_SIZE * 2;
  static_assert(MAX_HEADERS >= 512, "Buffer too small");

  // Computed constants (comptime call-site keyword):
  function computeEscapeTable(): Array[Bool, 256] is
      // pure arithmetic over Free types, nothing special about this function
      ...
  qed;

  constant ESCAPE_TABLE: Array[Bool, 256] := comptime computeEscapeTable();

  // Same function, runtime — no ambiguity:
  let table: Array[Bool, 256] := computeEscapeTable();
  ```
  **`comptime` eligibility rules** — the full set is small and exhaustive:

  | Requirement                                                                                         | Why                                                                |
  | --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
  | All argument values must be compile-time known (literals, `constant`s, or other `comptime` results) | Can't evaluate what doesn't exist yet                              |
  | Return type must be `Free`                                                                          | Linear types are runtime resources                                 |
  | All parameter types must be `Free`                                                                  | Same reason                                                        |
  | Function body must not call FFI / use `pragma Unsafe_Module`                                        | Can't call `malloc` at compile time                                |
  | Called functions must themselves be comptime-eligible (transitive)                                  | If `f` calls `g` which calls `read()`, `f` isn't comptime-eligible |

  The check is **lazy** — the compiler doesn't analyze functions for comptime eligibility upfront. When someone writes `comptime f(x)`, the compiler _tries_ to evaluate it right there. If `f`'s body contains a non-const operation, the error points at the offending line inside `f`. If `x` is a runtime value, the error points at `x`. No upfront annotation burden. This is better than Rust's model where you find out at the definition site — in Kyokai you find out at the usage site when you actually need it.
  **Example diagnostic** (per D29 quality standards):
  ```
  error[KYO-E0042]: comptime argument must be a compile-time constant
    --> src/Escaper.kyo:14:32
     |
  14 | constant TABLE := comptime makeTable(userInput);
     |                                      ^^^^^^^^^
     |                                      `userInput` is a runtime variable
     |
     = note: all arguments to a `comptime` call must be literals, constants,
             or other `comptime` expressions
     = help: if you need this value at runtime, remove `comptime`
  ```
  **[STAGE: DECIDED_CORE_SEMANTICS | D18 → Option D + B: const expressions + `comptime` call-site keyword]**
- `**comptime` evaluation is deterministic, host-independent, and governed only by explicit compile-time inputs** — Kyokai does not allow compile-time execution to smuggle hidden host state into the language or the build result.
**Rules**:
  1. For a fixed source program, selected target triple, language edition, compiler version, and explicit build options, a `comptime` expression has exactly one language result or one compile-time error.
  2. `comptime` evaluation may depend only on explicit compile-time inputs such as literals, `constant` values, other `comptime` results, `target`, and other compile-time built-ins explicitly defined by the language.
  3. `comptime` evaluation may not observe or depend on filesystem contents, environment variables, wall-clock time, timezone, locale, randomness, network state, process state, thread scheduling, FFI, or runtime capabilities.
  4. Runtime allocators and runtime resources do not exist at `comptime`. Any compiler scratch allocation used to perform evaluation is an implementation detail and must not be observable in language semantics.
  5. Integer arithmetic, checks, and floating-point results in `comptime` follow the same target-visible language rules as ordinary execution, including D75 and D76. `comptime` does not use a separate host arithmetic model.
  6. The compiler may enforce an explicit deterministic evaluation budget to reject nonterminating or excessively costly `comptime` evaluation. Exhausting that budget is a compile-time error. If the budget is configurable, the configuration is an explicit build input.
  **Why this fits Kyokai**: explicit compile-time execution remains part of the reproducible language contract instead of becoming a side channel into host state or toolchain folklore.
  **[STAGE: DECIDED_CORE_SEMANTICS | D18a → deterministic, host-independent `comptime`; only explicit compile-time inputs; explicit deterministic evaluation budget allowed]**
- **`comptime` evaluation uses one deterministic step budget per evaluation root** — Kyokai defines the cost model in language terms instead of leaving "too much compile-time work" to implementation folklore.
**Rules**:
  1. Each top-level `comptime` evaluation root has one step budget.
  2. A top-level evaluation root is any one `constant` initializer, `static_assert` condition, `when` guard, or explicit `comptime` expression currently being evaluated.
  3. The default step budget is `1_000_000` steps.
  4. The budget may be changed only through an explicit build input such as `[comptime] step_limit = N` in `kyokai.toml`.
  5. Nested `comptime` calls, transitive constant forcing, and helper-function calls do not receive fresh budgets; they consume from the current root's budget.
  6. At minimum, each function entry, each loop iteration, each branch/arm selection, and each language-defined `comptime` builtin invocation consumes one step.
  7. Exhausting the step budget is a compile-time error.
  8. The diagnostic for budget exhaustion must identify the evaluation root, the configured limit, and the fact that the limit is configurable through explicit build inputs.
  9. Wall-clock time is not part of the language budget model.
  **Why this fits Kyokai**: the limit is deterministic, host-independent, and explicit, while still giving implementations a precise way to reject nonterminating or unreasonably expensive compile-time evaluation.
  **[STAGE: DECIDED_CORE_SEMANTICS | D202 → one deterministic step budget per top-level comptime evaluation root; default `1_000_000`; configurable only through explicit build inputs]**
- **Compiler scratch memory may be used during `comptime`, but it is not a language-visible allocator and does not make runtime owning containers available at compile time** — Kyokai allows implementations to evaluate compile-time programs efficiently without smuggling the runtime allocation model into the language.
**Rules**:
  1. During one top-level `comptime` evaluation root, the implementation may use transient scratch memory to perform evaluation.
  2. That scratch memory is an implementation resource, not a language value.
  3. Safe Kyokai code cannot name, store, borrow, pass, compare, or otherwise observe the identity of that scratch memory.
  4. Exhausting comptime scratch memory is a compile-time error.
  5. All scratch memory associated with one evaluation root is discarded when that root finishes, whether evaluation succeeds or fails.
  6. A successful `comptime` result must be a self-contained `Free` value with no dependency on scratch-memory addresses, allocator identity, or runtime resources.
  7. This decision does not make runtime allocators available at `comptime`.
  8. A function that requires an ordinary allocator parameter is not made comptime-eligible merely because the implementation uses scratch memory internally.
  9. Runtime owning `Linear` containers such as `Buffer[T]`, `String`, and other allocator-backed runtime resource types remain outside ordinary `comptime` values under D18, D165, and D204.
  **Why this fits Kyokai**: the implementation gets room to evaluate compile-time code, but the language still keeps runtime allocation and compile-time evaluation as separate explicit worlds.
  **[STAGE: DECIDED_CORE_SEMANTICS | D203 → transient comptime scratch memory is allowed as an implementation resource only; it is not a language-visible allocator and does not make runtime owning containers comptime-eligible]**
- **Const generics are first-class over `Index` values, but they remain a separate explicit mechanism rather than turning all generics into Zig-style comptime programming**: value-level type parameters exist for fixed-size data and other shape-bearing APIs without replacing the ordinary generic system.
**Syntax**:
  ```kyokai
  generic [N: Index]
  record FixedBuffer is
      data: Array[Nat8, N];
      len: Index;
  build;
  ```
  **Rules**:
  1. A generic parameter may be an `Index`-typed const parameter, written in the ordinary generic header as `N: Index`.
  2. Const-generic arguments are compile-time `Index` values and follow the ordinary D18, D18a, D202, and D203 comptime model.
  3. Type equality for const-generic instantiations uses evaluated value equality of the const arguments.
  4. Const arguments participate in monomorphization identity. Distinct evaluated const values produce distinct generic instantiations unless another explicit rule says they normalize to the same value.
  5. Const generics do not replace ordinary type parameters, typeclasses, or explicit `comptime` call-site evaluation. They are a distinct value-level generic facility.
  6. D188 defines the exact admitted argument forms, equality boundary, and in-scope use sites for this facility.
  **Why this fits Kyokai**: arrays, matrices, fixed buffers, and similar APIs need value-level shape parameters, but Kyokai still benefits from keeping compile-time evaluation and the generic system as separate explicit mechanisms.
  **[STAGE: DECIDED_CORE_SEMANTICS | D159 -> full `Index` const generics with deterministic comptime-evaluable arguments, value-based type equality, and monomorphization participation]**
- **Const generic parameters have one exact admitted expression and equality model**: Kyokai does not leave const-generic arguments half-specified or host-defined.
  **Syntax**:
  ```kyokai
  generic [T: Type, Rows: Index, Cols: Index]
  record Matrix is
      data: Array[T, Rows * Cols];
  build;
  ```
  **Rules**:
  1. Every const generic parameter has type `Index`.
  2. A const generic argument must be a compile-time expression whose type is exactly `Index`.
  3. Admitted const-generic argument forms are:
     - `Index` literals
     - named `constant` values of type `Index`
     - earlier const generic parameters
     - parenthesized `Index` expressions built from admitted inputs using ordinary deterministic comptime arithmetic
     - explicit `comptime` calls whose result type is `Index`
  4. Const generic arguments are evaluated using the ordinary D18, D18a, D202, and D203 comptime rules. There is no separate host-dependent const-evaluation model.
  5. Two instantiations of the same generic declaration are the same type iff each corresponding const argument evaluates to the same canonical `Index` value.
  6. Equality is by evaluated value, not by source spelling.
  7. Failure to evaluate a const argument, step-budget exhaustion, overflow or trap under Kyokai's ordinary numeric rules, or production of a non-`Index` result is a compile-time error.
  8. Const parameters are in scope as compile-time `Index` values throughout the declaration body, including nested type expressions and contracts.
  9. Const parameters may appear in type positions such as array lengths and other const-generic arguments.
  10. Const parameters participate in monomorphization identity exactly by their evaluated values.
  **Why this fits Kyokai**: it closes the real semantic holes around shape-bearing types while keeping the facility narrow, explicit, deterministic, and auditable.
  **[STAGE: DECIDED_CORE_SEMANTICS | D188 -> exact `Index` const-generic argument forms, value-equality rule, evaluation failure behavior, and in-scope use of const parameters]**
- **`StaticString` gives `comptime` text a distinct `Free` type without collapsing it into runtime `String`** — compile-time text metaprogramming is supported, but Kyokai does not retarget ordinary string literals contextually or imply a hidden compile-time heap for runtime strings.
**Syntax**:
  ```kyokai
  constant banner: StaticString := static "Kyokai";
  constant full: StaticString := comptime concat(static "Kyo", static "kai");

  let msg: String := full.toStringIn(&!heap) or return;
  ```
  **Rules**:
  1. `StaticString` is a distinct text type from `String`. It represents immutable UTF-8 text backed by compiler-managed or program-static read-only storage and is a `Free` type.
  2. Ordinary `"..."` literals remain `String` in all contexts. Kyokai does not contextually reinterpret them as `StaticString`.
  3. The literal bridge to `StaticString` is explicit: `static "..."`.
  4. `comptime` functions may take and return `StaticString` because it is `Free`.
  5. During compile-time evaluation, `StaticString` results live in compiler-managed memory. When materialized into the compiled program, they become embedded read-only data.
  6. There is no implicit conversion between `String` and `StaticString`.
  7. Converting `StaticString` to an owned runtime `String` is explicit and allocator-taking, for example `toStringIn(alloc) -> Result[String, AllocError]`.
  8. `StaticString` supports the ordinary explicit text operations needed for metaprogramming, including `length`, `isEmpty`, `slice`, `concat`, `startsWith`, `endsWith`, `find`, and `asBytes() -> Span[Nat8, Static]`.
  9. D18 does not imply general heap-style compile-time allocation of runtime `String` values. Compile-time text metaprogramming goes through `StaticString`, not hidden runtime-string construction.
  **Why this fits Kyokai**: compile-time string work becomes powerful enough for metaprogramming, format checking, and generated tables, while the boundary between static text and owned runtime text remains explicit.
  **[STAGE: DECIDED_CORE_SEMANTICS | D120 → distinct `StaticString` with explicit `static "..."` bridge; no contextual literal retargeting; explicit allocator-taking conversion to `String`]**
- **Conditional compilation: three-tier model with `when` declaration guards** — Three tools for three shapes of platform-specific code. **Tier 1 (whole-file)**: When a module is inherently platform-specific (syscall wrappers, renderer backends), use separate `.kai` body files sharing the same `.kyo` interface. Build system (C01) selects which to compile. **Tier 2 (`when` declaration guards)**: When only a few functions differ by platform, use `when` guards on the declaration signature: `function pageSize(): Index when target.os == Os.Linux is ... qed;`. A false `when` guard makes the declaration semantically absent from the current build. For alternate platform definitions of the same declaration in a selected shared module, exactly one definition must be active for the current target; overlapping active matches and zero active matches are both compile-time errors. `when` guards are **declaration-level only** — never inside function bodies. Body-level platform branching should extract into a `when`-guarded helper. **Tier 3 (typeclass abstraction)**: For large platform abstractions (renderer, windowing), use typeclasses. `typeclass Renderer[R: Linear]` with platform-specific instances. This is an existing language feature — no D19 work needed.
`**target` is a language-level built-in** — `target` is a compile-time constant record with four fields: `target.os` (`Os` enum), `target.arch` (`Arch` enum), `target.abi` (`Abi` enum), and `target.endianness` (`Endian` enum). All four are compile-time constants set by the build system or derived deterministically from the selected target, and they are evaluable by D18's comptime machinery. The `os`/`arch`/`abi` fields follow the LLVM target triple model (`arch-os-abi`), while `endianness` gives the target byte order explicitly for D117. The fields are built-in enums (not strings) — this is what makes exhaustiveness checking real, same as `case` on a sum type.
`**Os`, `Arch`, `Abi`, and `Endian` define target vocabulary, not an automatic cartesian product** — the set of legal target triples and their support guarantees is fixed explicitly by D80. An enum variant existing does not by itself make every `os/arch/abi` combination valid.
**Initial enum variants** (extendable — adding variants later is backward-compatible):

  | Enum   | Variants                                                          | Covers                                   |
  | ------ | ----------------------------------------------------------------- | ---------------------------------------- |
  | `Os`   | `Linux`, `Android`, `MacOS`, `FreeBSD`, `Windows`, `Freestanding` | Hosted targets + bare metal escape hatch |
  | `Arch` | `X86_64`, `Aarch64`, `Riscv64`                                    | Primary architectures                    |
  | `Abi`  | `Gnu`, `Musl`, `Bionic`, `Msvc`, `None`                           | libc / environment families              |
  | `Endian` | `Little`, `Big`                                                 | Target byte order                        |

  `**Abi.None` is explicit, not magic absence**: `Abi.None` means "no extra ABI/environment selector beyond the target's hosted default contract or the freestanding contract." It does **not** mean "no calling convention exists" or "no ABI rules apply."
  `**when` guards compose with D18**: The `when` condition is a comptime boolean expression over `target` fields. Guards can combine fields: `when target.os == Os.Linux and target.abi == Abi.Musl`. The compiler evaluates it using the same constant-evaluation machinery that evaluates `constant PAGE_SIZE := 4096 * 2`. Near-zero implementation cost after D18 ships.
  **Feature flags are module-level only**: Feature flags defined in `kyokai.toml` control which modules are compiled (Tier 1). You cannot use `when feature.http2` on a function declaration. If a feature needs to gate a single function, that function belongs in its own conditionally-included module.
  **No `All` target variant**: A declaration without a `when` guard exists on ALL platforms by default. That IS the "all" case. `when` guards restrict a declaration to specific platforms — absence of a guard means universal. An explicit `All` value would invert this (requiring opt-in to universality), which is backwards.
  **Implementation**:
  ```kyokai
  // Tier 2: declaration-level when guards
  function pageSize(): Index when target.os == Os.Linux is
      return 4096;
  qed;

  function pageSize(): Index when target.os == Os.MacOS is
      return 16384;
  qed;

  // Combined field guards:
  function defaultLibcPath(): String when target.os == Os.Linux and target.abi == Abi.Musl is
      return "/lib/ld-musl-x86_64.so.1";
  qed;

  function defaultLibcPath(): String when target.os == Os.Linux and target.abi == Abi.Gnu is
      return "/lib64/ld-linux-x86-64.so.2";
  qed;
  ```
  **[STAGE: DECIDED_CORE_SEMANTICS | D19 → Three-tier: whole-file + `when` guards + typeclass pattern]**
- **Tier selection and `when` guarding happen at explicit semantic phases** — Kyokai separates “not part of this build” from parser implementation strategy.
**Rules**:
  1. Tier 1 module and feature selection happens before module-graph construction for the current build configuration.
  2. A declaration whose `when` guard evaluates to `false` is semantically absent from the current build: it contributes no names, no types, no obligations, and no code.
  3. The language does not require a compiler to skip parsing excluded declarations; parser recovery and tooling strategy are implementation details.
  4. For a set of alternate `when`-guarded declarations of the same declaration name and signature, at most one may be active in a build. Multiple active matches are a compile-time error.
  5. In a selected shared module, zero active declarations for such an alternate set is also a compile-time error. If an API is intentionally absent on some targets, that absence must be modeled with Tier 1 module selection rather than with maybe-missing declarations inside a selected shared module.
  **Why this fits Kyokai**: target selection becomes part of program identity, false `when` branches are truly absent, and the spec stays explicit without dictating one parser architecture.
  **[STAGE: DECIDED_CORE_SEMANTICS | D19a → Tier 1 selection before module-graph construction; false `when` is semantically absent; overlap and zero-match are errors in selected shared modules]**
- **Inline platform branching inside function bodies is not part of Kyokai** — D19's three-tier model is the whole story: whole-file selection, declaration-level `when` guards, and typeclass abstraction.
**Rules**:
  1. Kyokai does not add `comptime case target.os of ...` or any other body-level target-selection construct.
  2. If a function body needs small target-specific differences, the code must extract those differences into helper declarations and place the target split on those declarations with `when`.
  3. If the target split is broad enough that helper extraction becomes unnatural, the code should use Tier 1 whole-file separation or Tier 3 typeclass abstraction instead.
  **Why this fits Kyokai**: platform boundaries stay at declaration and module boundaries, where they remain inspectable in APIs and build structure instead of becoming statement-level conditional worlds inside one function body.
  **[STAGE: DECIDED_CORE_SEMANTICS | D123 → keep D19's declaration/module/typeclass model; no body-level inline platform branching]**
- `**Result`/`Optional` are language-level built-in types, not library types** — D15's error propagation (`let...else`, `or return`) depends on `Result` and `Optional` the same way branching depends on `Bool`. If the language's own control flow constructs operate on these types, they're already type system primitives — treating them as library types is architectural fiction. `Result[T, E]`, `Optional[T]`, `Ok`, `Err`, `Some`, `None` are built-in constructors alongside `true`, `false`, `nil`. No module, no import, no prelude entry. They simply ARE the language.
`**Ok`/`Err`/`Some`/`None` are keywords** — same as `true`, `false`, `nil`. They cannot be shadowed or used as variable names. If `None` could be a variable name, `or return Err(e)` would become ambiguous when `Err` is shadowed. The cost is four reserved identifiers. The full keyword list: `true`, `false`, `nil`, `Ok`, `Err`, `Some`, `None`.
**Why not prelude (Option B)**: A prelude is a hidden import. "Nothing hidden, nothing implicit" means if you can't see the import, it shouldn't exist. Making them built-in types eliminates the import entirely — there's nothing to import because they're not in a module.
**What IS and ISN'T built-in**:

  | Item                          | Classification                                  | Treatment                      |
  | ----------------------------- | ----------------------------------------------- | ------------------------------ |
  | `Result[T, E]`, `Ok`, `Err`   | Type system primitive (D15 depends on it)       | Language-level built-in        |
  | `Optional[T]`, `Some`, `None` | Type system primitive (nullability replacement) | Language-level built-in        |
  | `target`, `Os`, `Arch`, `Abi`, `Endian` | Compile-time constants + enums (D19/D117) | Language-level built-in        |
  | `Array[T, N]`                 | Container type (stdlib)                         | Explicit import — not built-in |
  | All stdlib functions          | Library code                                    | Explicit import                |
  | All I/O types                 | Capability-gated                                | Explicit import                |
  | All collections               | Data structures                                 | Explicit import                |

  **Built-in keywords**: `true`, `false`, `nil`, `Ok`, `Err`, `Some`, `None`.
  **Precedent**: Zig's `?T` (optional) and `T!E` (error union) are language-level constructs with no module backing them.
  **[STAGE: DECIDED_CORE_SEMANTICS | D24 → `Result`/`Optional`/`target` are language-level built-ins, no prelude expansion]**
- **FFI: `foreign "C"` blocks + `pragma Unsafe_Module` + `UnsafeCapability` + exact C-ABI surface** — Three mechanisms at three levels. `foreign "C" is ... mon;` blocks group raw foreign declarations, making the boundary visible. `pragma Unsafe_Module` marks a module as containing raw foreign-boundary code; safe modules cannot call raw foreign declarations. `UnsafeCapability` is a linear capability type that must be threaded to raw foreign call sites, creating an auditable type-level chain.
**Raw-call rule**: In Kyokai source, every declaration inside a `foreign` block is called as though it had an additional leading parameter of type `&![UnsafeCapability]`. That leading argument is part of Kyokai's safety contract and audit trail, not part of the foreign ABI; lowering erases it before the actual C call.
**Wrapper rule**: Safe APIs may wrap raw foreign declarations inside an unsafe module and translate them into ordinary Kyokai results and capabilities. Code that bypasses such wrappers must accept and thread `UnsafeCapability` explicitly.
**Type mapping** (every type crossing the FFI boundary has exactly one valid mapping):

  | Kyokai type         | C type               | Notes                                                                      |
  | ------------------- | -------------------- | -------------------------------------------------------------------------- |
  | `Address[T]`        | `T`* (nullable)      | Raw pointer. Free type.                                                    |
  | `Pointer[T]`        | `T`* (non-null)      | Non-null pointer. Free type. Trap on null.                                 |
  | `Int8`–`Int64`      | `int8_t`–`int64_t`   | Exact-width signed                                                         |
  | `Nat8`–`Nat64`      | `uint8_t`–`uint64_t` | Exact-width unsigned                                                       |
  | `Float32`/`Float64` | `float`/`double`     | IEEE 754                                                                   |
  | `Unit`              | `void` (return only) |                                                                            |
  | `Bool`              | `_Bool`              | Foreign APIs using integer booleans must spell an integer type explicitly. |

  **Additional ABI and failure-boundary rules**:
  1. `foreign "C"` uses the selected target's C ABI exactly.
  2. A record type may cross the boundary as a typed aggregate only if it is declared `extern record`.
  3. `extern type Name;` declares an opaque foreign type with unknown size and layout. An `extern type` may appear only behind pointers in foreign declarations and never by value.
  4. Arrays do not decay implicitly in foreign declarations. If the foreign ABI wants a pointer, the declaration must spell `Address[T]`, `Pointer[T]`, or another explicitly specified pointer-like form.
  5. Only `FnPtr(...)` may cross the foreign boundary as a typed bare callback. `Callable`, `CallableMut`, and `CallableOnce` do not cross FFI directly.
  6. Passing or returning unions by value is illegal. Untagged C unions remain outside Kyokai's typed FFI surface unless a separate decision specifies their ABI model.
  7. C varargs are not part of the FFI surface.
  8. Non-default calling conventions are not part of the FFI surface.
  9. Imported C symbols are never implicitly mangled. Any symbol renaming requires explicit FFI syntax.
  10. `errno` is foreign runtime state, not Kyokai semantics. Safe wrappers must translate it into explicit Kyokai results.
  11. A raw foreign call carries no implied guarantee about retry-safety, EINTR handling, allocation behavior, thread safety, or other foreign conventions unless the wrapper contract states those rules explicitly.
  12. Foreign code may not unwind, `longjmp`, or otherwise skip Kyokai frames. Doing so is an unsafe-boundary contract violation.
  13. Any foreign API that can call back into Kyokai must specify callback ABI, lifetime, reentrancy, and thread/termination rules explicitly.
  **FFI ownership and sum-type boundary rules**:
  1. Raw `foreign "C"` declarations may not take or return Kyokai `Linear` values by value.
  2. The raw foreign surface is limited to FFI-admitted `Free` scalars, raw address/pointer forms, `FnPtr`, `extern record`, and pointers to `extern type`.
  3. A safe wrapper may consume a linear Kyokai value before a foreign call only by explicitly decomposing it into FFI-legal raw parts inside a `pragma Unsafe_Module`.
  4. The module's `unsafe contract` must state whether ownership is retained by Kyokai, transferred to foreign code, borrowed only for the duration of the call, or returned through a named handle/result.
  5. Foreign code is never assumed to respect Kyokai linearity. Any retained pointer, callback, ownership handoff, destructor obligation, or aliasing promise must be represented by an explicit wrapper type and covered by an unsafe contract.
  6. Kyokai unions, `Optional`, `Result`, and other sum types have no implicit C ABI and may not cross raw FFI by value.
  7. C tagged-union APIs must be modeled with explicit ABI-shaped declarations: `extern record` for containing structs, integer/enum-like tags as explicit fixed-width values, and opaque/raw payload representation only where the target C ABI actually specifies it.
  8. Safe wrappers translate C ABI records, tags, error codes, and payload contracts into Kyokai unions only after validating the tag and payload contract.
  9. There is no implementation-defined compiler-generated tag/payload layout for FFI.
  **Implementation**:
  ```kyokai
  pragma Unsafe_Module;

  module body MyPosix is
      extern type FILE;

      foreign "C" is
          function c_open(path: Address[Nat8], flags: Int32): Int32;
          function c_fopen(path: Address[Nat8], mode: Address[Nat8]): Address[FILE];
      mon;

      function openFile(cap: &![UnsafeCapability], path: Address[Nat8]): Result[Int32, IoError] is
          let fd: Int32 := c_open(cap, path, 0);
          if fd < 0 then
              return Err(IoError.fromErrno());
          fi;
          return Ok(fd);
      qed;
  seal;
  ```
  **[STAGE: DECIDED_CORE_SEMANTICS | D20/D242/D242a → `foreign` blocks + `pragma Unsafe_Module` + `UnsafeCapability` + explicit ABI, failure, ownership-transfer, and sum-type boundary rules]**
- **Unsafe modules require source-level unsafe contracts tied to actual unsafe operations** — Kyokai's audit trail is not an informal review habit or a detached `SAFETY.md`; it is structured source metadata checked against the unsafe surface the module uses.
**Syntax shape**:
  ```kyokai
  pragma Unsafe_Module;

  unsafe contract MyPosix is
      covers foreign c_open;
      assumes c_open_abi_matches_target;
      preserves no_capability_forgery;
      maps errno_to IoError;
      forbids foreign_unwind;
  audit;
  ```
**Rules**:
  1. Every `pragma Unsafe_Module` must contain at least one `unsafe contract Name is ... audit;` block.
  2. An unsafe contract block is part of the module body source. It is not an external prose file and not an optional comment convention.
  3. The contract must enumerate each unsafe facility used by the module: `foreign` declarations, unsafe intrinsics, volatile operations, raw dynamic loading, raw signal handlers, raw pointer/address conversions, trusted capability acquisition, or any future unsafe primitive.
  4. For each covered facility, the contract states the preconditions assumed, the Kyokai invariants preserved, the failure mapping exposed to safe callers, and the safe wrapper exports or trusted constructors that rely on it.
  5. The compiler rejects an unsafe module when an unsafe operation is not covered by an unsafe contract entry.
  6. The compiler rejects an unsafe contract entry that names no unsafe operation in the module, unless it is explicitly marked as a module-wide invariant entry.
  7. Unsafe contracts are machine-discoverable audit metadata. Documentation tools, package audit tools, and compiler diagnostics must be able to list unsafe contracts and the operations they cover.
  8. An unsafe contract is not a formal proof and does not weaken D73. The unsafe operation's semantics remain the individually specified language/toolchain contract for that operation.
  9. Safe wrappers exported by an unsafe module must mention the unsafe contract entries they rely on, either directly in the wrapper's contract metadata or through a documented module-wide invariant entry.
**Why this fits Kyokai**: Austral already gates unsafe at module boundaries, and Kyokai already threads `UnsafeCapability`; this adds the missing auditable source link between each low-level escape hatch and the invariant it claims to preserve.
  **[STAGE: DECIDED_CORE_SEMANTICS | D245 → source-level `unsafe contract ... audit;` blocks required for `Unsafe_Module`; compiler checks coverage of unsafe operations]**
- **`result` is a contextual keyword scoped only to `ensure` clauses** — Kyokai keeps contract syntax readable without globally reserving one of the most common local variable names in ordinary code.
**Rules**:
  1. `result` has special meaning only while parsing and type-checking an `ensure` clause.
  2. Outside `ensure` clauses, `result` is an ordinary identifier and may be used anywhere an identifier is otherwise legal.
  3. The lexer does not need a globally reserved `result` token. Parser context is what gives `result` its contract meaning.
  4. This decision governs only name reservation and parsing. The ownership and observation semantics of `result` inside `ensure` remain governed by D53 and any later clarification decisions that mention `ensure result`.
  **Why this fits Kyokai**: `ensure result > 0` stays readable, while ordinary code keeps `let result := ...;` and similar names with zero ceremony.
  **[STAGE: DECIDED_CORE_SEMANTICS | D125 → `result` is contextual inside `ensure`; unreserved elsewhere]**
- **`foreign` blocks close with `mon;`** — the FFI boundary is a gate/portal, not a proof body. `mon` makes the raw boundary visually distinct while still keeping the terminator set small and memorable.
**Rules**:
  1. The only closing form for a foreign declaration block is `mon;`: `foreign "C" is ... mon;`.
  2. `mon` is reserved as a foreign-block terminator and is not reused to close any other construct.
  3. Grammar, formatting, examples, and tooling must treat `mon;` as the canonical foreign-block terminator.
  4. D9's semantic boundary terminator family therefore includes `qed;`, `build;`, `spec;`, `drop;`, `seal;`, `mon;`, `pick;`, `join;`, and `audit;`.
  **Why this fits Kyokai**: `mon` names the FFI boundary directly instead of overloading `qed` for a block that is about crossing a gateway, not finishing a proof body.
  **[STAGE: DECIDED_CORE_SEMANTICS | D127 → `foreign "C"` blocks close with `mon;`]**
- **Generic linear cleanup uses an explicit `Destroyable` typeclass; Kyokai never invents structural destruction for user-defined types** — generic code may destroy a `Linear` value only when the type explicitly supplies the cleanup behavior.
**Core typeclass**:
  ```kyokai
  typeclass Destroyable(T: Linear) is
      method destroy(self: T): Unit;
  spec;
  ```
  **Rules**:
  1. `Destroyable` lives in `Kyokai.Core` as a standard typeclass, not as a language primitive.
  2. `destroy(self: T)` is an ordinary consuming method. Its body may perform any explicit teardown required by the type before the value is considered fully consumed.
  3. The compiler does not auto-derive `Destroyable` instances for user-defined types and does not synthesize recursive field-by-field destruction as fallback behavior.
  4. Generic code that may need to consume stored, buffered, or deferred `T` values must require `T: Destroyable`. This includes buffered channels, generic containers, and consuming iterators.
  5. Types may still expose richer domain-specific consuming operations such as `close`, `flushAndClose`, `commit`, or `cancel`; `Destroyable.destroy` is the generic cleanup surface, not a ban on more precise APIs.
  6. Calling `destroy` is always explicit source code. Kyokai never inserts implicit `destroy` calls at scope exit, on ordinary linearity failure, or as hidden runtime cleanup.
  7. If a language feature introduces a generated type with a specified destroy operation, that generation must be part of that feature's explicit contract; it does not create a general auto-derivation rule for arbitrary user-defined types.
  **Why this fits Kyokai**: channels, containers, and iterators get a real generic cleanup contract without reintroducing the hidden destructor behavior Kyokai is explicitly rejecting.
  **[STAGE: DECIDED_CORE_SEMANTICS | D124 → explicit `Destroyable` typeclass in `Kyokai.Core`; no auto-derivation or implicit destruction]**
- **Raw dynamic library loading exists only as an explicit unsafe boundary facility** — Kyokai does not pretend `dlopen`/`dlsym` become safe merely by wrapping them in nicer names. Loading foreign code is OS authority, and typing a looked-up symbol is an unsafe claim about ABI reality.
**Rules**:
  1. Raw dynamic loading is available only in `pragma Unsafe_Module` code.
  2. Opening a dynamic library requires an explicit `DynamicLoadCapability` acquired from `RootCapability`.
  3. `DynLibrary` is a `Linear` handle. Opening returns `Result[DynLibrary, DynLoadError]`.
  4. Targets that do not support dynamic loading must reject acquisition of `DynamicLoadCapability` or return an explicit unsupported-target failure when opening is attempted. Kyokai does not invent fake dynamic-loading support on freestanding targets.
  5. Raw symbol lookup returns a raw symbol handle, not a typed callable/function value.
  6. Converting a raw symbol handle into a typed function pointer or typed data address is an unsafe operation that requires `UnsafeCapability`; the programmer is asserting that the requested type, calling convention, and ownership contract match reality.
  7. Any symbol-derived typed view is valid only while the originating `DynLibrary` remains loaded. Safe code must not obtain such typed views from the raw API.
  8. Closing a library consumes its `DynLibrary` handle. After close, all symbol handles and any typed views derived from them are invalid by contract.
  9. Loader-side foreign constructors, foreign destructors, and other loader-managed side effects are part of the foreign trust boundary, not safe Kyokai semantics.
  **Why this fits Kyokai**: the language exposes the OS facility honestly for runtimes, kernels, and advanced FFI users, but it refuses to blur "I found a symbol by name" into a safe type-checked operation.
  **[STAGE: DECIDED_CORE_SEMANTICS | D113a → raw dynamic library loading is an unsafe capability-gated boundary with linear library handles and unsafe typed symbol casts]**
- **Safe runtime extensibility uses verified Kyokai plugin contracts, not safe `dlsym`** — Kyokai's safe plugin story is a separate facility from raw dynamic loading. A safe plugin is a Kyokai-defined contract artifact whose compatibility, authority, and lifetime rules are checked explicitly at load time.
**Rules**:
  1. Safe plugin loading requires `DynamicLoadCapability`; loading code at runtime remains an explicit authority.
  2. A safe plugin artifact must carry a machine-readable contract descriptor that records at least: target contract, language edition, `.koi` format version, plugin ABI version, contract/interface hash, declared exported entry surface, and declared capability requirements.
  3. Loading succeeds only on exact compatibility with the host's expected plugin contract. There is no "close enough" ABI matching and no type-erased fallback.
  4. The safe plugin surface is closed and monomorphic. It does not use trait objects, existential values, or runtime type dictionaries forbidden by D82.
  5. `LoadedPlugin[C]` is a `Linear` handle parameterized by the expected contract `C`.
  6. Safe use of a plugin happens through a typed entry surface borrowed from `LoadedPlugin[C]`; the host does not obtain arbitrary raw symbol access from the safe API.
  7. Any plugin-defined value whose validity depends on the plugin remaining loaded must be represented as a plugin-tied type that cannot outlive the relevant `LoadedPlugin[...]` borrow/handle relation.
  8. Safe plugin loading grants no ambient authority. Any filesystem, process, network, TLS, terminal, or other effectful capability a plugin may use must cross the contract explicitly as a Kyokai capability parameter or capability-bearing object.
  9. Safe plugins do not rely on hidden module constructors. Initialization and shutdown are explicit contract entrypoints invoked by the loader as part of load/unload.
  10. Unloading consumes the `LoadedPlugin[...]` handle and is illegal while plugin-tied values or entry borrows derived from it are still live.
  **Why this fits Kyokai**: Kyokai gets a real safe plugin model, but the safety comes from explicit contract verification, explicit capability flow, and explicit lifetime ties rather than from pretending arbitrary foreign shared libraries are type-safe.
  **[DECIDED: D113b → safe plugins are verified Kyokai contract artifacts with exact compatibility checks, explicit init/shutdown, explicit capability flow, and linear loaded-plugin handles]**
- **Function pointers + standardized `Callable` typeclasses form the callback substrate** — `FnPtr(A): R` is the bare callback form for C interop and dispatch tables, while the `Callable` family is the shared substrate for stateful callbacks, whether written manually or through D118's explicit-capture closure literals.
**Standard Callable hierarchy** (maps to Kyokai's existing borrow model):

  | Typeclass            | Self parameter      | Captures              | Call count | Rust analogue |
  | -------------------- | ------------------- | --------------------- | ---------- | ------------- |
  | `Callable[A, R]`     | `&[Self]` (borrow)  | `Free`, read-only     | Many       | `Fn`          |
  | `CallableMut[A, R]`  | `&![Self]` (mutref) | `Free`, mutable       | Many       | `FnMut`       |
  | `CallableOnce[A, R]` | `Self` (consumed)   | `Linear` OK, one-shot | Once       | `FnOnce`      |

  **Fixed arity family**:
  1. Arity-1 callbacks use `Callable[A, R]`, `CallableMut[A, R]`, and `CallableOnce[A, R]`.
  2. Arity-2 through arity-4 callbacks use the matching `Callable2`/`Callable3`/`Callable4` family, with corresponding `CallableMut*` and `CallableOnce*` variants.
  3. Code that truly needs more than four callback parameters must use an explicit named record payload with the arity-1 family.
  **No implicit capture and no hidden closure machinery.** Capture sets remain a programmer choice, not a tautology, so they stay visible in source. D118 adds explicit-capture closure literals, but those literals still lower to this same `Callable` substrate rather than introducing implicit environment analysis or hidden heap semantics.
  **Why `Callable` typeclasses instead of per-callback typeclasses (Option D's ceremony)**: Parallel to D23 — `Equality` and `TotalOrder` exist as standard typeclasses rather than opening general operator overloading. `Callable`/`CallableOnce` exist as standard typeclasses rather than requiring a new typeclass per callback shape.
  **Implementation**:
  ```kyokai
  // Bare function pointer (Free, no captures):
  let cmp: FnPtr(&[Int32], &[Int32]): Int32 := &compareAscending;
  sort(buffer, cmp);

  // Stateful one-shot callback via CallableOnce:
  record ThresholdFilter is
      threshold: Int32;
  build

  instance CallableOnce[Int32, Bool] for ThresholdFilter is
      method callOnce(self: ThresholdFilter, arg: Int32): Bool is
          return arg > self.threshold;
      qed;
  qed;

  let filter: ThresholdFilter := ThresholdFilter { threshold: 42 };
  processItems(buffer, filter);  // filter consumed, linear enforced

  // Mutable stateful callback via CallableMut:
  record Counter is
      count: Int32;
  build

  instance CallableMut[Unit, Int32] for Counter is
      method callMut(self: &![Counter], arg: Unit): Int32 is
          self.count := self.count + 1;
          return self.count;
      qed;
  qed;

  // Two-argument callback via Callable2:
  instance Callable2[Int32, Int32, Bool] for LessThan is
      method call(self: &[LessThan], a: Int32, b: Int32): Bool is
          return a < b;
      qed;
  qed;
  ```
  **[STAGE: DECIDED_CORE_SEMANTICS | D21 → FnPtr + fixed callable-family substrate; explicit-capture closure literals are added separately in D118]**
- **Multi-argument callbacks use a fixed arity callable family, not anonymous inline callback-record syntax** — Kyokai extends the existing callback substrate by a small closed family instead of adding tuple packs, inline record type expressions, or compiler-invented anonymous product types.
**Rules**:
  1. The standard library provides `Callable[A, R]`, `Callable2[A, B, R]`, `Callable3[A, B, C, R]`, and `Callable4[A, B, C, D, R]`.
  2. The same arity family exists for mutable and one-shot callbacks: `CallableMut2`, `CallableMut3`, `CallableMut4`, and `CallableOnce2`, `CallableOnce3`, `CallableOnce4`.
  3. `FnPtr` follows the same arity model for bare function pointers.
  4. D118 closure literals lower to the family member matching the closure's parameter arity and D197-selected ownership mode.
  5. Kyokai does not add anonymous inline record syntax or tuple-like packs in callback type positions.
  6. Code that needs callback arity greater than four must package its arguments in an explicit named record and use the arity-1 callable family.
  **Why this fits Kyokai**: the callback surface grows only by a small closed family that matches the already-decided `FnPtr` model, while avoiding anonymous structural product syntax that would open a much larger design surface.
  **[STAGE: DECIDED_CORE_SEMANTICS | D126 → fixed `Callable`/`CallableMut`/`CallableOnce` arity family through 4 parameters; no inline callback-record syntax]**
- **Explicit-capture closure literals are built-in sugar over the `Callable` substrate** — Kyokai allows closure literals only when the capture set is written explicitly; the language does not infer or hide environment capture.
**Syntax**:
  ```kyokai
  let pred := fn [limit, &cfg] (x: Int32): Bool is
      return x < limit and cfg.isValid();
  qed;

  let pred2 := fn [limit, &cfg] (x: Int32): Bool => x < limit and cfg.isValid();
  ```
  **Rules**:
  1. A closure literal is introduced with `fn [captures] (params): Ret is ... qed;` or the one-expression shorthand `fn [captures] (params): Ret => expr`.
  2. The capture list is mandatory. `[]` is written explicitly for a zero-capture closure literal.
  3. Each capture entry states its mode explicitly: `name` for by-value capture, `&name` for immutable borrow capture, and `&!name` for mutable borrow capture.
  4. Kyokai performs no implicit environment capture and adds no hidden capture entries.
  5. Closure literals lower to the same D21/D126 callable-family substrate rather than introducing a separate runtime closure object model.
  6. Closure literals do not imply hidden heap allocation; any storage strategy must preserve the same explicit ownership and borrow semantics as the equivalent manual `Callable` implementation.
  7. The precise rule that determines which member of the `Callable` family a given closure literal implements is specified separately by D197.
  **Why this fits Kyokai**: callback ceremony drops sharply, but capture sets, borrow modes, and allocation visibility stay explicit instead of becoming compiler folklore.
  **[STAGE: DECIDED_CORE_SEMANTICS | D118 → explicit-capture closure literals in block and one-expression forms; no implicit capture; lowers to the D21 `Callable` substrate]**
- **Closure classification is determined by the strongest environment access the closure actually requires, with owned linear captures forcing one-shot semantics** — Kyokai keeps capture lists explicit, but it still classifies each closure into exactly one member of the D21/D126 callable family using closed rules instead of folklore.
**Rules**:
  1. Every closure literal lowers to a compiler-generated environment type containing exactly the listed captures plus an implementation of exactly one member of the callable family whose arity matches the closure parameter list.
  2. If any by-value capture has `Linear` type, the closure environment is `Linear` and the closure implements the appropriate-arity `CallableOnce` family member only.
  3. Otherwise, if the capture list contains any `&!name` entry, the closure implements the appropriate-arity `CallableMut` family member.
  4. Otherwise, if the body performs assignment to a by-value captured binding or takes/passes a mutable borrow derived from a by-value captured binding, the closure implements the appropriate-arity `CallableMut` family member.
  5. Otherwise, the closure implements the appropriate-arity `Callable` family member.
  6. A closure type is `Linear` iff it owns at least one by-value `Linear` capture. Otherwise the closure type is `Free`.
  7. Borrow captures `&name` and `&!name` are ordinary borrows tied to the lifetime of the closure value. The closure may not outlive those borrow regions.
  8. A borrow derived from a captured borrow may not escape beyond the ordinary lifetime of that captured borrow.
  9. A zero-capture closure is `Free` and implements `Callable`.
  10. The `CallableOnce` case dominates the other cases: if rule 2 applies, the closure does not also implement a `CallableMut` or `Callable` family member.
  **Why this fits Kyokai**: the capture list still carries the important source-level ownership information, but the callable-family choice now reflects the real environment access the closure body needs instead of a too-shallow syntactic guess.
  **[STAGE: DECIDED_CORE_SEMANTICS | D197 → closure lowers to one matching-arity D21/D126 callable family member; owned linear captures force `CallableOnce`; otherwise mutable environment use yields `CallableMut`; remaining cases use `Callable`]**
- **Named stackless pull generators are a first-class iterator declaration, not a general coroutine system** — Kyokai adds `generator` declarations with `yield`, but it keeps the feature in the D32 iteration world rather than turning it into hidden async control flow or opaque return types.
**Syntax**:
  ```kyokai
  generator Countdown(start: Int32): Int32 is
      var i: Int32 := start;
      while i > 0 do
          yield i;
          i := i - 1;
      od;
  qed;

  for n in makeCountdown(3) do
      debug n;
  od;
  ```
  **Rules**:
  1. `generator Name(params): Item is ... qed;` declares a nominal generator type `Name`.
  2. The declaration also defines a constructor function `makeName(params): Name`.
  3. `Name` implements D32's `Iterator` typeclass with associated `Item = Item`.
  4. `yield expr;` is a statement and is legal only inside a generator body. `expr` must have the generator's declared item type.
  5. Executing `yield expr;` suspends the generator and causes the current `next(&!gen)` call to return `Some(expr)`.
  6. Reaching the end of the generator body or executing bare `return;` completes the generator. Once completed, all later `next` calls return `None`.
  7. Generator iterators are fused by default.
  8. Generator values are `Linear` and single-consumer.
  9. `yield` is suspension, not scope exit. It does not run `defer` or `errdefer` for still-live suspended scopes.
  10. Every generator type gets a compiler-generated consuming `destroy(self: Name): Unit` operation for abandoning an incomplete generator value explicitly.
  11. Destroying an incomplete generator runs all currently pending ordinary `defer` actions for its suspended scopes in reverse registration order and does not run `errdefer`, because generator destruction is not an error exit.
  12. `yield` is illegal inside `defer` and `errdefer` bodies.
  13. A local borrow created inside the generator body may not remain live across a `yield`.
  14. A borrow stored across suspension must come from a parameter or previously stored generator-state field whose region outlives the generator value under the ordinary borrow rules; otherwise the program is ill-typed.
  15. Generators do not create concurrency, stackful coroutines, symmetric transfer, or opaque return types. They interact with callers only through the ordinary D32 `Iterator.next` protocol.
  **Why this fits Kyokai**: infinite streams, paginated traversal, and tree walking become ergonomic, but the feature remains explicit, single-consumer, iterator-shaped, and fully tied into the language's existing borrow and cleanup model.
  **[STAGE: DECIDED_CORE_SEMANTICS | D198 → named stackless `generator` declarations with `yield`; nominal linear iterator types; explicit destroy semantics for suspended state]**
- **Explicit package/workspace model with deterministic module resolution** — Kyokai uses a three-level project model: **module** (the thing imported in source), **package** (the unit defined by a `kyokai.toml` and the boundary for dependencies, artifacts, and D17 `internal` visibility), and **workspace** (an explicit manifest-defined collection of packages built together). Kyokai intentionally collapses Rust's crate/package split into one concept: **package**. There is no inferred project structure. The nearest ancestor containing `kyokai.toml` is the package root. A workspace exists only when a manifest explicitly declares `[workspace]`; a directory tree with several child packages is NOT automatically a workspace.
**Exact workspace syntax**: the workspace manifest uses an explicit member list, e.g.
  ```toml
  [workspace]
  members = [
      "packages/core",
      "packages/net",
      "packages/cli",
  ]
  ```
  **Workspace root may NOT also be a package**: a manifest is either a workspace manifest or a package manifest, never both. This avoids nearest-manifest ambiguity, root-level `src/` confusion, and lockfile confusion.
  **Intra-workspace dependencies are by package name, not by path**: paths belong in `[workspace].members`; dependencies should name the package identity:
  ```toml
  [package]
  name = "net"
  version = "0.1.0"

  [layout]
  module_root = "src"

  [dependencies]
  core = { workspace = "core" }
  pcre = { git = "https://github.com/kyokai/pcre", rev = "a1b2c3d4..." }
  ```
  Package names must therefore be unique within a workspace.
  **Package manifests declare the module root explicitly**: a package manifest must contain `[layout] module_root = "<relative-dir>"`. There is no implicit default module root. The path is interpreted relative to the package root, must be non-empty, must not be absolute, and must not escape the package root via `..`.
  **Lockfile rule**: a standalone package has `package-root/kyokai.lock`. A workspace has exactly one `workspace-root/kyokai.lock` covering all member packages. Member packages inside a workspace do not own separate lockfiles.
  **Module mapping rule**: `.` is always a directory separator in module names, with no exceptions. If `[layout] module_root = "src"`, then `import Foo` resolves to `src/Foo.kyo` + `src/Foo.kai`, and `import Foo.Bar` resolves to `src/Foo/Bar.kyo` + `src/Foo/Bar.kai`. There is no `mod.kyo`, no alternate dotted filename form like `src/Foo.Bar.kyo`, and no include-path search. One import path, one file pair.
  **Import scope rule**: imports are file-scope declarations only. They may appear at the top of a `.kyo` or `.kai` file, and selective symbol imports plus nicknames are allowed, but there are no function-local, block-local, or expression-local imports.
  **Prefix modules may coexist**: `Foo` and `Foo.Bar` are distinct logical modules and may both exist in the same package. That is not ambiguous because directory segments are path segments, not implicit module nesting. What is forbidden is multiple filesystem spellings resolving to the SAME logical module.
  **Why this fits Kyokai**: boundaries are declared rather than inferred, import resolution is purely mechanical, package identity is the correct unit for unsafe auditing and visibility, and tooling can answer "what am I building?" from the manifest without heuristics.
  **Why not infer workspaces from folder layout**: that would make project structure an ambient convention rather than part of the contract. If the workspace exists, it must be written down.
  **Why not make `internal` workspace-visible**: visibility should track the package boundary, not the accidental fact that two packages happen to live in the same repository.
  **[STAGE: DECIDED_CORE_SEMANTICS | D78 → explicit package/workspace model + exact manifest-declared module-root file mapping]**
- **Import collisions are errors at the import site, and selective imported names use explicit `as` renaming when disambiguation is needed** — Kyokai does not let import order silently choose winners, and D254's receiver-module UFCS fallback never repairs an ambiguous file scope.
**Rules**:
  1. A selective import may rename an introduced unqualified name with `as`, for example `import PkgA.Util (Hash as UtilHash);`.
  2. Collision checking happens after applying any explicit import renames.
  3. If two import declarations in the same file would introduce the same unqualified name after renaming, the file is ill-formed.
  4. Qualified module imports do not by themselves create unqualified-name collisions merely because the referenced modules happen to export the same member names.
  5. Wildcard imports remain illegal.
  6. Built-in language names such as `Ok`, `Err`, `Some`, `None`, `true`, and `false` may not be introduced or shadowed by imports.
  **Why this fits Kyokai**: file scope stays mechanically understandable, import order never changes meaning, and the programmer has one explicit escape hatch when two useful names would otherwise collide.
  **[STAGE: DECIDED_CORE_SEMANTICS | D214 → import collisions are import-site errors; selective imports use explicit `as` renaming; built-in names cannot be shadowed by imports]**
- **Import syntax has exactly three file-scope forms: qualified module import, qualified module alias, and selective unqualified member import** — Kyokai keeps imports explicit and readable without adding `open`-style namespace flooding.
**Syntax**:
  ```kyokai
  import Foo.Bar;
  import Foo.Bar as Bar;
  import Foo.Bar (baz, qux as localQux);
  ```
  **Rules**:
  1. `import Foo.Bar;` introduces the module name `Foo.Bar` for qualified access only.
  2. `import Foo.Bar as Bar;` introduces the same module for qualified access under the alias `Bar`.
  3. `import Foo.Bar (baz, qux as localQux);` introduces only the listed exported names unqualified into file scope, applying any per-name `as` rename before D214 collision checking.
  4. Selective imports name direct exports of the referenced module only. They do not recurse through other modules or create transitive namespace injection beyond the module's ordinary export surface.
  5. If code wants both qualified module access and selective unqualified names, it must write both import declarations explicitly.
  6. Wildcard imports are illegal.
  7. `open`, `using namespace`, and any other form that implicitly brings an entire module's exports into unqualified scope are illegal.
  8. Imports remain file-scope declarations only under D78.
  **Why this fits Kyokai**: module access stays legible at the import site, selective unqualified names remain explicit, and the language never needs a second "where did this name come from?" lookup model.
  **[STAGE: DECIDED_CORE_SEMANTICS | D179 → exact three-form import surface: qualified import, module alias, selective import with per-name `as`; no wildcard or `open`]**
- **UFCS receiver-module extension lookup is a narrow fallback, not C++ ADL or global method search** — Kyokai solves common names like `length` without making the entire dependency graph part of method resolution.
**Rules**:
  1. UFCS lookup has two phases: ordinary imported-name lookup first, then receiver-module extension lookup only if ordinary lookup finds no candidate.
  2. Ordinary imported-name lookup follows D179 and D214 exactly. A unique imported function wins; an import collision is an error and is not repaired by receiver lookup.
  3. Receiver-module extension lookup searches only the defining module of the receiver's nominal type, or the module that explicitly owns the relevant built-in/special form's receiver-callable surface.
  4. Receiver-module lookup considers only exported functions explicitly marked as receiver-callable for that receiver type. It does not consider every function whose first parameter happens to match.
  5. Receiver-module lookup is not global, not dependency-wide, not transitive through re-exports, not typeclass dispatch, and not based on arbitrary argument-dependent lookup.
  6. If receiver-module lookup finds zero candidates or more than one candidate after ordinary type checking, compilation fails.
  7. Qualified function calls and explicit import aliases remain the disambiguation mechanisms.
  **Why this fits Kyokai**: the programmer gets ergonomic `buf.length()` and `text.isEmpty()` without wildcard imports or alias spam, while the lookup boundary remains small enough for tooling and audits to explain.
  **[STAGE: DECIDED_CORE_SEMANTICS | D254 → type-directed UFCS only as explicit receiver-module extension lookup after ordinary lookup fails]**
- **Dependency model: workspace package references + git commits as identity, tags as checked labels** — Kyokai has exactly two dependency sources: another package in the same workspace, or an external Git repository. Workspace dependencies are named by package identity, not by filesystem path. External Git dependencies are pinned by commit hash. A Git tag may be included as a human-readable release label, but it is never the source of truth — `rev` is the source of truth. Branches are forbidden in manifests because they are moving targets and violate reproducibility.
**The allowed manifest shapes are**:
  ```toml
  [dependencies]
  core = { workspace = "core" }
  pcre = { git = "https://github.com/kyokai/pcre", rev = "a1b2c3d4..." }
  pcre = { git = "https://github.com/kyokai/pcre", tag = "v1.2.3", rev = "a1b2c3d4..." }
  ```
  **Rules**:
  1. Exactly one of `workspace` or `git` must appear.
  2. If `git` appears, `rev` is mandatory.
  3. `tag` is optional metadata. If present, the package manager must verify that the tag resolves to the declared `rev` when the dependency is added or updated.
  4. `branch` is illegal in `kyokai.toml`.
  5. Workspace dependency names refer to package names, not paths. Package names must therefore be unique within a workspace.
  6. The lockfile records the fully resolved dependency graph, including exact Git revisions.
  **Why not tag-only dependencies**: tags are human-friendly but not immutable enough to serve as the identity of a dependency.
  **Why not branches**: a branch name is a moving pointer; identical manifests could resolve to different code at different times.
  **Why workspace deps by name instead of path**: package identity should not depend on repository layout. Paths belong in `[workspace].members`; dependencies should name the package being depended on.
  **Why this fits Kyokai**: it is explicit, reproducible, auditable, and easy for tooling to validate.
  **[STAGE: DECIDED_CORE_SEMANTICS | D51 → workspace deps by name + git `rev` required + optional checked `tag`]**
- **Package yanks are append-only index metadata: existing lockfiles keep building, new resolution avoids withdrawn revisions** — yanking is withdrawal from new selection, not deletion or source mutation.
**Rules**:
  1. The official package index is append-only. A yank is an append-only metadata record marking a package version/revision as withdrawn for new dependency resolution.
  2. Yanking does not delete source, mutate source, rewrite tags, change artifact hashes, or alter the meaning of an existing lockfile.
  3. New dependency resolution must not select a yanked version unless the user explicitly opts into that exact yanked revision.
  4. Existing `kyokai.lock` files remain reproducible and may continue to resolve yanked entries.
  5. Yank metadata records package identity, version/revision identity, timestamp, actor identity according to the index trust model, and an optional reason or security advisory link.
  6. Tooling must surface yanked status during dependency resolution, update, audit, and lockfile reporting.
  7. If a yanked revision is also compromised in a way that should block even locked builds, that is a separate security-policy mechanism and must not be smuggled into yank semantics.
  **Why this fits Kyokai**: package resolution stays reproducible and append-only while giving the ecosystem a visible way to stop new projects from selecting known-bad releases.
  **[STAGE: DECIDED_CORE_SEMANTICS | D244 → append-only yanks ignored by new resolution but honored by existing lockfiles]**
- **Package names have one canonical grammar** — Package names are stable manifest identities, not loose human labels. They appear in `[package].name`, `[dependencies]` workspace references, CLI package selection, lockfiles, and `.koi` artifacts, so the grammar must be fixed and toolable.
**Grammar**: a package name must match `^[a-z][a-z0-9-]{0,63}$`.
**Additional restrictions**:
  1. Only lowercase ASCII letters, ASCII digits, and `-` are allowed.
  2. `_`, `.`, whitespace, and uppercase letters are illegal.
  3. A name may not end in `-`.
  4. A name may not contain `--`.
  5. Names are compared byte-for-byte; there is no case folding or punctuation normalization.
  6. Names that collide with Windows reserved device names such as `con`, `prn`, `aux`, `nul`, `com1`-`com9`, and `lpt1`-`lpt9` are illegal.
  7. Package names must be unique within a workspace.
  **Why this fits Kyokai**: there is one legal spelling, one comparison rule, and no hidden normalization behavior. Tooling can validate names deterministically across platforms.
- **Official source file extensions are `.kyo` for interfaces and `.kai` for bodies** — These are the normative Kyokai source extensions. They are part of the language/toolchain contract, not just a temporary repository convention.
**Meaning**:
  1. `Foo.Bar` resolves to `Foo/Bar.kyo` for the interface and `Foo/Bar.kai` for the body under the package module root defined by D78.
  2. `.kyo` is the importable interface surface; `.kai` is the implementation body.
  3. Tooling, editors, docs, and build-system code should treat `.kyo`/`.kai` as the official Kyokai source pair.
  4. Austral's `.aui`/`.aum` extensions are not part of Kyokai.
  **Why this fits Kyokai**: the fork has its own stable identity, and D78's deterministic module mapping now has a formally settled filename contract.
  **[STAGE: DECIDED_CORE_SEMANTICS | D52 → `.kyo` interface + `.kai` body]**
- **Single intermediate visibility level: `internal` is package-visible, not workspace-visible** — Kyokai keeps the two-file interface/body model and adds exactly one new visibility level between public and private. A declaration in a `.kyo` interface with no visibility modifier is **public** and importable by dependent packages. A declaration in a `.kyo` interface prefixed with `internal` is **package-visible** and importable only by modules in the same package rooted at the nearest `kyokai.toml` (D78). A declaration that exists only in the `.kai` body is **private** and visible only inside that module body. This gives Kyokai the one missing capability Austral lacks: sharing helpers across sibling modules without turning them into public API.
`**internal` is legal only in interface files**: body-only declarations are already private, so `internal` in a `.kai` file is an error.
`**internal` is never widened by workspace membership**: two packages in the same workspace do not gain privileged access to each other. Workspace layout is a build concern, not a visibility rule.
**Types follow the same visibility split**: a type declared `internal` in the interface is visible only within the package. Whether that type is opaque or transparent is still determined by the type declaration form; `internal` changes WHO can name/use it, not whether its representation is exposed. Body-only types remain module-private.
**Typeclasses and instances follow the same rule**: an `internal typeclass` or `internal instance` exists only within the package. Instance resolution across package boundaries must ignore internal instances from dependencies.
**No re-export across a package boundary**: an internal declaration cannot be re-exported or surfaced as public API by another package-level mechanism.
**Artifacts and docs follow visibility**: `.koi` artifacts may record internal declarations for same-package checking, but generated public documentation and dependency import surfaces must exclude them by default.
**Why not Rust-style path visibility**: Kyokai has a package boundary and a module boundary. It does not need a third visibility system tied to nested module paths.
**Why not keep Austral's binary public/private split**: without `internal`, any shared helper becomes accidental public API. That is bad library hygiene and bad boundary design.
**[STAGE: DECIDED_CORE_SEMANTICS | D17 → `internal` keyword for package-level visibility]**
- `**.koi` artifacts are explicit interface contracts, not cache blobs** — Separate compilation in Kyokai is built around a real artifact contract. A `.koi` file is not "whatever the compiler happened to cache"; it is the checked interface product of a package, consumed by downstream compilation and tooling. Incremental caches may exist internally, but they are not the language/toolchain boundary.
**A `.koi` artifact records at least**: producing compiler version, language edition, `.koi` format version, target contract, package identity, the package's module set, hashes/fingerprints of interface inputs, visibility-marked declarations (`public` and `internal`), type definitions at their visible opacity level, typeclass definitions, legal instances, and any metadata required by the current generic materialization / instantiation decisions (D82a and D82b) for downstream type checking and code generation.
**Private declarations do not cross the boundary**: body-only `.kai` declarations that are not part of the package interface never appear in `.koi`.
**Visibility is preserved inside the artifact**: `internal` declarations may appear in `.koi` because same-package compilation and tooling need them, but import resolution from a different package must treat those entries as nonexistent.
**Compatibility is explicit**: a compiler may consume a `.koi` artifact only when the language edition, `.koi` format version, target contract, and any explicitly versioned generic/codegen contract all match exactly. The producing compiler version is recorded for provenance and diagnostics, but a compiler-version mismatch by itself is NOT automatically an incompatibility if the compatibility-class fields still match.
**Support is explicit too**: a compiler may reject `.koi` format versions it does not implement, even if the artifact was produced by another compiler binary from the same language edition family.
`**.koi` is a toolchain contract artifact**: `kyokai check`, separate compilation, documentation generation, and downstream package builds may depend on its format and semantics. That contract must not silently change under "cache implementation details."
**Why this fits Kyokai**: the interface boundary becomes inspectable, versioned, and auditable instead of implicit compiler state.
**[STAGE: DECIDED_CORE_SEMANTICS | D79 → `.koi` is a versioned per-package interface artifact contract]**
- **Language editions are explicit manifest-selected source-semantics modes** — an edition tells the parser, resolver, and edition-aware tools how to interpret source text. It is not a loose marketing version, and it does not weaken D79's exact `.koi` compatibility contract.
**Rules**:
  1. Every package manifest with a `[package]` table must declare exactly one edition field, written as `edition = "2026"` for the first Kyokai edition.
  2. The declared edition applies to every `.kyo` and `.kai` file in that package. A workspace may contain packages that declare different editions.
  3. An edition may change only source-language interpretation and source-facing tool behavior for code that opts into that edition. This includes grammar, reserved words, contextual keywords, name-resolution rules, desugaring rules, edition-gated default diagnostics, and other explicitly documented source-semantic choices.
  4. An edition must NOT silently reinterpret code that still declares an older edition. When a newer compiler supports an older edition, that older-edition source keeps its older-edition parsing and semantics.
  5. Moving a package to a newer edition is an explicit source rewrite step, not automatic reinterpretation. The toolchain provides `kyokai migrate --edition <edition>` for edition migration.
  6. The compiler, formatter, checker, documentation generator, and diagnostics are edition-aware. They must parse, format, analyze, and report each package according to that package's declared edition.
  7. Mixed-edition workspaces are legal as repository structure, but cross-edition compatibility is not implied. D79 still governs artifact compatibility, and D79 requires exact language-edition match for `.koi` consumption.
  8. Therefore, under the current design, a Kyokai package may not consume a `.koi` artifact from a different language edition. Any normalized cross-edition interface contract would require its own later explicit decision; it does not exist implicitly.
  9. The language edition is part of the reproducible-build identity under D83 and must be recorded in `.koi` artifacts under D79.
  **Why this fits Kyokai**: editions give the project room to evolve source syntax and defaults without ever making old code mean something new behind the programmer's back.
  **[STAGE: DECIDED_CORE_SEMANTICS | D105 → manifest-declared editions as source-semantics modes; mixed-edition workspaces allowed structurally, but `.koi` compatibility still requires exact edition match]**
- **Standard-library compatibility follows editions for source-semantics surfaces and SemVer for ordinary package-like API evolution** — D105 already owns language editions; the remaining stdlib rule is how `Kyokai.*` changes without surprising old source.
**Rules**:
  1. Language source-semantics changes are governed by D105 editions. Release cadence remains governed by D157.
  2. Standard-library APIs whose behavior is part of the language/core contract or automatically available with the toolchain may not make breaking source or semantic changes without an edition boundary or an explicitly named compatibility mode.
  3. Ordinary package-style standard-library modules may evolve by the D223 SemVer convention, with additive APIs and deprecations allowed without an edition.
  4. Removing or changing a public stdlib API that older-edition source may rely on requires either an edition-gated replacement path or an explicit compatibility shim whose behavior is documented.
  5. A `.koi` artifact records the exact language edition and stdlib/interface identity needed for downstream checking. No cross-edition compatibility is inferred.
  **Why this fits Kyokai**: editions stay the rare source-semantics boundary, while the larger standard library still gets a concrete compatibility policy instead of Python-style surprise breaks.
  **[STAGE: DECIDED_CORE_SEMANTICS | D243 → resolved by D105/D157 plus stdlib compatibility policy: edition-gated core breaks, SemVer for ordinary stdlib modules]**
- **Typeclass coherence is mandatory: one resolved program, one applicable instance** — Kyokai adopts explicit coherence and orphan rules so typeclass instance selection is deterministic, auditable, and independent of import order. In plain language: when the compiler resolves a typeclass call for a concrete type, there must be exactly one legal answer.
**Uniqueness rule**: for any fully resolved `(Typeclass, Type arguments)` pair visible at a call site, there must be exactly one applicable instance.
**Orphan rule**: an instance is legal only if the defining package owns the typeclass or owns at least one concrete head type named in the instance head. A third-party package may not implement a foreign typeclass for an entirely foreign type.
**No scoped orphan exceptions**: `internal` or private visibility does NOT relax the orphan rule. Kyokai does not permit package-scoped or module-scoped foreign-typeclass-on-foreign-type instances.
**Overlap is forbidden**: if two instances could both apply after substitution, the program is ill-formed even if one "looks more specific." Kyokai does not have implicit specialization precedence.
**Import order never matters**: adding or reordering imports must not change which instance is chosen.
**Local scope tricks are forbidden**: no function-local or block-local instance declarations, and no hidden "current instance" context.
**Blanket/generic instances are allowed only when they preserve uniqueness**: a generic instance is legal only if the compiler can still prove that no second instance can apply to the same fully resolved call.
**The sanctioned escape hatch is a wrapper/newtype**: if a package wants custom behavior for a foreign typeclass on a foreign type, it must introduce a local wrapper type and implement the instance for that wrapper. This keeps instance ownership explicit instead of making resolution depend on visibility scope.
**Why this fits Kyokai**: typeclass resolution remains part of the language contract, not a search heuristic. The programmer can audit which package owns which authority to define behavior.
**[STAGE: DECIDED_CORE_SEMANTICS | D81 → coherence + orphan rules + no overlap + no import-order dependence]**
- **Coherence diagnostics for overlapping instances must identify the actual overlap, not merely state that one exists** — once D81 rejects overlap, the minimum diagnostic contract must still tell the programmer which two instance declarations conflict and why.
**Rules**:
  1. Coherence and overlap checking consider all instances visible to the current build after package resolution.
  2. If two instances overlap, compilation fails even when one appears more specific than the other. Kyokai still has no implicit specialization precedence.
  3. The diagnostic must name the typeclass, both conflicting instance headers, and the package/module where each conflicting instance was defined.
  4. For generic overlap, the diagnostic must report at least one concrete witness substitution that demonstrates the conflict.
  5. For example, if `instance Destroyable(Container[T]) where T: Destroyable` overlaps `instance Destroyable(Container[SpecificType])`, the diagnostic should report that the overlap witness is `T = SpecificType`.
  **Why this fits Kyokai**: explicit coherence rules lose practical value if the failure mode collapses into folklore-grade "instance overlaps" messages. The programmer must be able to audit the exact conflicting behavior boundary.
  **[STAGE: DECIDED_CORE_SEMANTICS | D216 → overlap diagnostics name the conflicting instances, their defining packages/modules, and at least one overlap witness when generics are involved]**
- **Generic and typeclass dispatch is static; runtime dictionaries and trait objects are not part of Kyokai's language contract** — Instance resolution happens at compile time under D81. Generic calls and typeclass-dispatched calls do not become erased dynamic dispatch, hidden dictionary passing, or witness-table passing as part of the language semantics.
**No implicit runtime dictionaries**: generic calls do not gain hidden typeclass dictionary parameters, hidden witness tables, or trait-object-style runtime dispatch.
**No built-in erased trait objects**: code that wants runtime heterogeneity must use an explicit union or another explicitly chosen data representation rather than a `dyn Trait` analogue.
**Explicit runtime dispatch objects are a different category**: a runtime value/handle chosen explicitly in source code is ordinary program state, not a hidden typeclass dictionary just because its representation may contain function pointers.
**Code materialization is a separate decision family**: D82 fixes dispatch semantics only. The questions of how concrete bodies are materialized, where cross-package instantiations are emitted, and which deduplication optimizations are allowed are split into D82a and D82b.
**Why this fits Kyokai**: runtime behavior stays explicit and no hidden polymorphic machinery enters the calling convention, while compile-time and code-size strategy is forced into its own explicit decisions instead of being smuggled in as backend folklore.
**[STAGE: DECIDED_CORE_SEMANTICS | D82 → static generic/typeclass dispatch; no runtime dictionaries or trait objects]**
- **Allocator dispatch under D44 is explicit runtime state, not a D82 dictionary exception** — D82 bans hidden compiler-inserted dictionaries for generic and typeclass dispatch. It does not ban an allocator handle that the programmer explicitly chose, passed, and caused a container to store as part of its own runtime state.
**Rules**:
  1. D82's ban continues to apply to hidden generic/typeclass witness passing and trait-object-style dispatch.
  2. A container-stored allocator handle under D44 is ordinary runtime state chosen explicitly in source code at construction or other explicit allocator-taking boundaries.
  3. The runtime dispatch needed to call `allocate`, `deallocate`, and `reallocate` through that stored allocator handle is therefore not a language-level generic/typeclass dictionary and does not weaken D82.
  4. This ruling creates no ambient allocator default and no hidden allocator propagation. Allocator choice remains explicit at construction and fresh-allocation boundaries under D44 and D201.
  5. The standard library may offer allocator-specialized container families as ordinary library types if they are ever justified, but D44's value-level allocator model remains the canonical and sufficient language-facing model.
  **Why this fits Kyokai**: the language keeps D82's ban on hidden polymorphic machinery intact while also keeping D44 honest about allocator choice being explicit program data rather than compiler folklore.
  **[STAGE: DECIDED_CORE_SEMANTICS | D130 → D44 allocator handles are explicit runtime state, not a D82 runtime-dictionary exception]**
- **Reproducible builds are the default contract** — Given the same source contents, `kyokai.toml`, `kyokai.lock`, compiler version, language edition, target triple, and declared build options, Kyokai's specified primary artifacts must be bit-identical unless an output mode explicitly opts out. Reproducibility is not a best-effort quality-of-implementation goal; it is the default toolchain contract.
**The build identity includes**: package/workspace source contents, manifest contents, lockfile contents, enabled features, selected build profile, compiler version, language edition, target triple, and any explicit flags that the spec says affect code generation or artifact format.
**Forbidden hidden inputs**: timestamps, random seeds, filesystem traversal order, host locale, host timezone, unstable hash iteration order, and unrelated environment state must not change reproducible artifacts unless a mode explicitly says they are part of the output contract.
**Generated C, `.koi`, and final binaries/libraries all inherit this rule**: if Kyokai emits them as normative build products, they must obey the reproducibility contract.
**Path handling must be explicit**: if a build profile allows absolute source paths in debug information, that choice must be written down. Otherwise the toolchain must normalize or remap paths so build location does not perturb the artifact.
**Opting out must be explicit**: if a future profile or flag wants nondeterministic build IDs, embed timestamps, or include host-specific diagnostics, that must be an explicit documented mode, not default behavior.
**Why this fits Kyokai**: explicit languages should have explicit build identity. "Same inputs, same outputs" is the build-system form of "nothing hidden."
**[STAGE: DECIDED_CORE_SEMANTICS | D83 → reproducible by default, with explicit build identity]**
- **One `kyokai` binary, with explicit package/workspace/profile/target semantics** — Kyokai's toolchain surface is a single command-line binary named `kyokai`. The CLI is manifest-driven: the nearest relevant `kyokai.toml` determines whether the current scope is a standalone package or a workspace root (D78). The CLI does not guess hidden project structure beyond that manifest lookup.
**Core subcommands**: `build`, `run`, `check`, `test`, `fmt`, `doc`, `repl`, `eval`, `lsp`, `audit`, `init`, `new`, `add`, `clean`.
`**new` and `init` must write explicit layout information**: package manifests generated by toolchain commands must include the required `[layout] module_root = "src"` entry unless the user explicitly chooses a different relative module-root path at creation time.
**Package/workspace selection**:
  - If the current manifest is a package manifest, `build`, `check`, `doc`, and `test` operate on that package.
  - If the current manifest is a workspace manifest, `build`, `check`, `doc`, and `test` operate on all workspace members by default.
  - `-p` / `--package <name>` restricts the command to one workspace member.
  - `--workspace` forces whole-workspace scope when run from inside a member package.
  **Run semantics**:
  - `kyokai run` builds then executes a runnable package target.
  - If the selected scope contains more than one runnable package, `--package <name>` is required.
  - `kyokai run -- <args...>` passes subsequent arguments to the program being run.
  **Profile selection**:
  - `--profile <name>` selects a named build profile defined by D31.
  - `--release` is exact syntactic sugar for `--profile release`. It does not introduce hidden optimization behavior beyond the `release` profile written in the manifest.
  - If no profile is specified, the default profile for `build`, `run`, `check`, `test`, and `doc` is `debug`.
  **Target selection**:
  - `--target <arch-os-abi>` selects the compilation target triple.
  - `--backend <c|llvm>` selects the code generation backend. The package manifest may name a default backend in `[build].backend`; the CLI flag overrides it explicitly.
  - The selected triple must be one of Kyokai's legal D80 target triples. Unknown or illegal `os/arch/abi` combinations are a front-end error, not a late codegen surprise.
  - This determines the values of the language-level built-ins `target.arch`, `target.os`, and `target.abi` (D19), and selects the matching `[target.<triple>]` toolchain configuration from D31 and D149.
  - If the selected backend has no conforming toolchain configuration for the selected target, the build fails. There is no silent fallback to another backend.
  `**check` is not "build without honesty"**:
  - `kyokai check` performs parsing, name resolution, import resolution, type checking, linearity checking, instance resolution, and interface/artifact validation.
  - `check` may skip final code generation and linking, so codegen-only or link-only failures may still be discovered later by `build`.
  - The CLI must document this distinction explicitly; "fast feedback" is not a license for semantic ambiguity.
  `**add` follows D51 exactly**:
  - Adding an existing workspace package writes `{ workspace = "name" }`.
  - Adding an external Git dependency without an explicit pin is an error; the command must not silently choose the current default-branch HEAD.
  - `kyokai add --git <url> --rev <rev>` writes `{ git = "<url>", rev = "<rev>" }`.
  - `kyokai add --git <url> --tag <tag>` resolves the tag to a commit and writes both `tag` and `rev`.
  - If a future convenience flag such as `--head` exists, it must resolve HEAD immediately and write the resulting `rev`; the manifest still stores the immutable pin, not the moving reference.
  `**doc` follows visibility**: generated documentation includes public API by default and excludes `internal` declarations unless a future explicit flag says otherwise.
  **Verbosity must expose the resolved build plan**: `--verbose` prints the selected manifest root, workspace/package scope, target triple, backend name, profile name, resolved compiler/linker tools, and the exact additional flags applied from target/profile configuration.
  **Why this fits Kyokai**: one tool is simpler, but the important part is that its behavior is manifest-defined and inspectable rather than conventional or magical.
  **[STAGE: DECIDED_CORE_SEMANTICS | D26 → single `kyokai` binary with explicit scope/profile/target semantics]**
- **Build profiles and target/toolchain configuration live in `kyokai.toml`, not in folklore** — Kyokai exposes binary/output policy through explicit manifest tables rather than hard-wired release folklore. Profiles describe optimization/debug/strip/LTO/identical-code-folding policy. Target tables describe which backend tools and extra flags implement a particular target triple. Package build tables describe what artifact kind a package produces.
**Profile tables**:
  - Profiles are named using `[profile.<name>]`.
  - `debug` and `release` are conventional names, not magic compiler modes.
  - Custom profiles are allowed; if a custom profile omits a field, it may inherit from another named profile with `inherits = "<name>"`.
  - In a standalone package build, profile tables are read from that package manifest.
  - In a workspace build, profile tables are read from the workspace root manifest; member-package profile tables are ignored to avoid ambiguity.
  - The standardized profile policy surface includes at least `optimization`, `debug_info`, `strip`, `lto`, and `identical_code_folding`.
  **Target/toolchain tables**:
  - Toolchain selection lives in `[target.<triple>]`.
  - `<triple>` must be a legal D80 target triple. Manifest configuration may choose which legal targets it knows how to build for; it may not invent new triples outside the language/toolchain target matrix.
  - Supported common keys are:
    - `spec` — optional relative path to a reusable target-spec TOML file imported into this target table
    - `sysroot` — explicit sysroot or SDK root for this target
    - `linker` — linker or linker-driving command shared by backend paths unless overridden below
    - `archiver` — tool used to build static libraries when needed
    - `runner` — optional execution command for `kyokai run --target ...`
  - Backend-specific overrides live in `[target.<triple>.backend.<name>]`.
  - The standardized backend names are `c` and `llvm`.
  - Supported C-backend keys are:
    - `cc` — C compiler used for generated C compilation
    - `cflags` — extra C compiler flags
    - `ldflags` — extra linker flags for the C-backend path
  - Supported LLVM-backend keys are:
    - `ldflags` — extra linker flags for the LLVM-backend path
  - Common fields are inherited by backend-specific subtables unless overridden explicitly.
  - Per-target, per-profile overrides live in `[target.<triple>.profile.<name>]`.
  - Per-target, per-backend, per-profile overrides live in `[target.<triple>.backend.<name>.profile.<name>]`.
  **Package build table**:
  - Package output settings live in `[build]` in the package manifest.
  - `backend = "c" | "llvm"`
  - `output_type = "executable" | "static-lib" | "dynamic-lib"`
  - `link = "target-default" | "static" | "dynamic"`
  - `backend` is the manifest-declared default code generation backend for that package. CLI `--backend` overrides it explicitly.
  - `target-default` means "use the selected target/toolchain default behavior"; it is explicit, not implicit.
  - If `static` or `dynamic` is requested and the selected target/toolchain cannot honor that request, the build fails. There is no silent fallback.
  **Example**:
  ```toml
  [profile.debug]
  optimization = 0
  debug_info = true
  strip = false
  lto = false
  identical_code_folding = false

  [profile.release]
  optimization = 2
  debug_info = false
  strip = true
  lto = true
  identical_code_folding = true

  [profile.size]
  inherits = "release"
  optimization = 2
  strip = true
  lto = true
  identical_code_folding = true

  [target.x86_64-linux-gnu]
  sysroot = "/usr"
  linker = "clang"
  archiver = "ar"

  [target.x86_64-linux-gnu.backend.c]
  cc = "clang"
  cflags = []
  ldflags = []

  [target.x86_64-linux-gnu.backend.llvm]
  ldflags = []

  [target.x86_64-linux-musl]
  spec = "targets/x86_64-linux-musl.toml"

  [target.x86_64-linux-musl.backend.c.profile.release]
  cflags = ["-Os"]
  ldflags = []

  [build]
  backend = "c"
  output_type = "executable"
  link = "target-default"
  ```
  **Dead code elimination after H06**: once package-level separate compilation exists, dead stripping is primarily a section/linker/LTO story, not a "whole program compiler sees everything" story. Release builds should therefore emit/link in a way that supports dead-section elimination.
  **Why this fits Kyokai**: users can choose exact policy per target and per profile without undocumented `--release` assumptions, and the difference between `target.abi = Musl` and `cc = musl-gcc` stays explicit.
  **[STAGE: DECIDED_CORE_SEMANTICS | D31 → explicit manifest-defined profiles + target toolchains + package output settings]**
- **Build profiles do not weaken safe-language runtime checks** — Kyokai separates performance/debugging policy from safety semantics and does not let release mode silently turn contract violations into unchecked behavior.
**Rules**:
  1. Build profiles may change optimization settings, debug information, symbol emission, `debug` stripping behavior under D45, and default hosted backtrace policy under D170.
  2. Build profiles may NOT weaken, remove, or reinterpret safe-language overflow checks, bounds checks, contract checks, `unreachable;`, or any other TPOE-triggering rule defined by the language.
  3. The semantic behavior of safe Kyokai code is therefore profile-invariant with respect to these checks.
  4. If a programmer wants unchecked behavior, it must come from explicit unsafe or explicitly named unchecked operations, not from selecting a faster build profile.
  **Why this fits Kyokai**: release optimization remains a performance concern instead of becoming a second semantics mode that quietly changes the safety contract.
  **[STAGE: DECIDED_CORE_SEMANTICS | D185 → debug/release profiles may change diagnostics and optimization, but not safe-language runtime checks; unchecked behavior must be explicit]**
- **Legal target triples and support guarantees are explicit** — the `Os`, `Arch`, and `Abi` enums define Kyokai's target vocabulary, but the valid target set is not the cartesian product. Each legal `arch-os-abi` triple has an explicit support promise, and every unlisted combination is invalid.
**Support tiers**:
  - **Tier 1**: compiler, stdlib, tests, and CI must pass. Regressions on a Tier 1 target are release-blocking.
  - **Tier 2**: compiler and stdlib must build, and the maintained target smoke suite must pass before release. CI coverage may be reduced or periodic rather than per-change.
  - **Experimental**: the target triple is recognized and may have working compiler and stdlib support, but codegen, runtime support, tooling, and regression handling are best-effort.
  **Legal target matrix**:

  | Target triple               | Tier         | Notes                                                                               |
  | --------------------------- | ------------ | ----------------------------------------------------------------------------------- |
  | `x86_64-linux-gnu`          | Tier 1       | Primary hosted Linux/glibc target                                                   |
  | `x86_64-linux-musl`         | Tier 1       | Primary hosted Linux/musl target                                                    |
  | `x86_64-freebsd-none`       | Tier 1       | Primary BSD target                                                                  |
  | `x86_64-windows-msvc`       | Tier 1       | Primary Windows target                                                              |
  | `aarch64-linux-gnu`         | Tier 2       | Hosted ARM64 Linux                                                                  |
  | `aarch64-linux-musl`        | Tier 2       | Hosted ARM64 Linux with musl                                                        |
  | `aarch64-freebsd-none`      | Tier 2       | Hosted ARM64 BSD                                                                    |
  | `aarch64-windows-msvc`      | Tier 2       | Hosted ARM64 Windows                                                                |
  | `x86_64-freestanding-none`  | Tier 2       | x86_64 bare-metal / kernel-space target                                             |
  | `aarch64-freestanding-none` | Tier 2       | ARM64 bare-metal / kernel-space target                                              |
  | `x86_64-macos-none`         | Experimental | Recognized target, but below primary hosted targets until real test coverage exists |
  | `aarch64-macos-none`        | Experimental | Recognized target, but below primary hosted targets until real test coverage exists |
  | `x86_64-android-bionic`     | Experimental | Recognized Android target                                                           |
  | `aarch64-android-bionic`    | Experimental | Primary Android ABI shape, but still late bring-up                                  |
  | `riscv64-linux-gnu`         | Experimental | Hosted RISC-V Linux                                                                 |
  | `riscv64-linux-musl`        | Experimental | Hosted RISC-V Linux with musl                                                       |
  | `riscv64-freebsd-none`      | Experimental | Hosted RISC-V BSD                                                                   |
  | `riscv64-freestanding-none` | Experimental | Bare-metal RISC-V target                                                            |

  **Rules**:
  - Any `arch-os-abi` triple not listed in the legal target matrix is invalid and must be rejected explicitly before build planning, code generation, or linking.
  - Android is represented by `target.os = Os.Android` with `target.abi = Abi.Bionic`.
  - `Abi.None` means "no extra ABI/environment selector beyond the target's hosted or freestanding contract." It does not mean "no ABI exists."
  - If Kyokai later wants to recognize another target triple, that target must be added explicitly to this matrix; legality is never inferred from enum membership alone.
  **Why this fits Kyokai**: target support becomes an explicit contract rather than folklore. The language can expose `target.os`, `target.arch`, and `target.abi` without pretending every combination is real, and the toolchain can reject impossible or unsupported targets honestly and early.
  **[STAGE: DECIDED_CORE_SEMANTICS | D80 → explicit support-tier contract + legal target matrix; invalid triples rejected early]**
- **Local variable inference is allowed only when the initializer already determines the type** — Kyokai permits `let name := expr;` only for immutable local bindings whose initializer has one fully determined, denotable type without needing target typing from the left-hand side. This removes redundant repetition while keeping types mechanically obvious from the right-hand side.
**Rules**:
  1. `let name := expr;` is legal only if `expr` has exactly one known type before considering the omitted left-hand-side annotation.
  2. `var name: Type := expr;` still requires an explicit type. Mutable locals are always annotated.
  3. Ambiguous literals are not rescued by this rule. `let x := 5;` is illegal unless the literal's type is already fixed by the initializer expression itself.
  4. Polymorphic constructors or empty values that need target typing remain illegal without an explicit annotation.
  5. Pattern-binding forms keep using the type information from the matched expression; D46 does not add any new pattern-inference mechanism beyond that.
  **Examples**:
  - Legal: `let p := Point { x: 5, y: 10 };`
  - Legal: `let file := openConfig(path);` if `openConfig` already has a single concrete return type.
  - Illegal: `let x := 5;`
  - Illegal: `var x := makeThing();`
  - Illegal: `let xs := List.empty();`
  **Why this fits Kyokai**: the rule removes repeated type names when the initializer already says everything, but it does not introduce target-typed guessing or hidden coercions.
  **[STAGE: DECIDED_CORE_SEMANTICS | D46 → immutable `let :=` only when RHS already fixes one type]**
- **Assignment is a statement, never an expression** — `:=` performs mutation and yields no value. It cannot appear where an expression is required.
**Rules**:
  1. `a := b;` is a statement only.
  2. It yields nothing, not even `Unit`.
  3. It cannot be chained (`a := b := c` is illegal).
  4. It cannot appear in conditions, argument lists, return expressions, or any other expression position.
  5. If compound assignments are added later, they inherit the same statement-only rule.
  **Why this fits Kyokai**: mutation stays visually separate from value production, and the language permanently eliminates assignment-in-condition bugs.
  **[STAGE: DECIDED_CORE_SEMANTICS | D59 → assignment is statement-only and yields no value]**
- **No binding may shadow another binding that is still in scope** — Kyokai bans shadowing across local scopes, nested scopes, patterns, loops, and parameters. A new binding may reuse a name only after the earlier binding has gone out of scope.
**This applies to**: function parameters, local `let` bindings, local `var` bindings, pattern-bound names, `let...else` success bindings, loop variables, and any future binding form unless that form explicitly states otherwise.
**Rules**:
  1. If an identifier names any currently in-scope binding, introducing a new binding with that identifier is a compile-time error.
  2. Consuming a linear value does not end the binding's lexical scope. A consumed binding remains in scope until its enclosing block or scope ends.
  3. Therefore, a consumed binding's identifier cannot be rebound or shadowed in that same scope.
  4. Reuse of the same identifier after the old binding's lexical scope has ended is allowed.
  5. This is a name-resolution error, not a warning or style lint.
  **Why this fits Kyokai**: names refer to one thing at a time, and name resolution stays purely lexical instead of depending on dynamic linearity state such as "already consumed."
  **[STAGE: DECIDED_CORE_SEMANTICS | D60 → no shadowing of any still-in-scope binding]**
- **Module-level mutable state is banned** — Kyokai modules may define constants at module scope, but not mutable variables. There is no language-level notion of a user-defined mutable global.
**Rules**:
  1. `constant` declarations are allowed at module scope.
  2. `var` declarations are illegal at module scope.
  3. If a program needs shared mutable state, it must be represented explicitly as a value passed through capabilities, function parameters, or explicit references/borrows.
  4. The language does not treat "global singleton" patterns as a special case.
  **Why this fits Kyokai**: mutable ambient state undermines the capability model, weakens auditability, and hides dependencies that should be visible in function signatures and module imports.
  **[STAGE: DECIDED_CORE_SEMANTICS | D62 → no module-level mutable variables]**
- **Module-level constants are forced lazily, cached, and checked for dependency cycles across module boundaries** — Kyokai does not assign semantic meaning to the eager initialization order of pure constants.
**Rules**:
  1. Importing or compiling a module does not by itself force evaluation of every module-level `constant` in that module.
  2. A module-level constant is forced only when its value is needed to evaluate another constant, a `static_assert`, a `when` guard, a `comptime` expression, or generated code that depends on that constant's value.
  3. Each module-level constant has one of three evaluation states: `Unforced`, `Evaluating`, or `Evaluated`.
  4. Forcing an `Unforced` constant changes its state to `Evaluating`, then recursively forces its constant dependencies, then computes and caches its value, then changes its state to `Evaluated`.
  5. Forcing an already `Evaluated` constant reuses its cached value.
  6. Encountering a dependency on a constant already in state `Evaluating` is a constant-dependency cycle and is a compile-time error.
  7. The cycle diagnostic must report the dependency path forming the cycle.
  8. These rules apply across module boundaries exactly as they apply within one module.
  9. Because module-level constants are pure and may not observe ambient state, unrelated constants have no semantic evaluation order. An implementation may force them in any deterministic order, or not force them at all if they are never needed.
  **Why this fits Kyokai**: pure constants stay reproducible and explicit, cycles become precise compile-time errors, and the language does not invent a fake initialization-order story for values that are semantically just pure computations.
  **[STAGE: DECIDED_CORE_SEMANTICS | D215 → module-level constants are lazy, cached, and cycle-checked across modules; unrelated constants have no semantic evaluation order]**
- **Safe thread-local storage is keyed and capability-gated; ambient `thread_local var` does not exist in safe Kyokai** — Kyokai does allow per-thread mutable state, but only through explicit key objects and an explicit `TlsCapability`. Raw platform TLS remains available only at the unsafe boundary.
**Rules**:
  1. Safe Kyokai has no module-scope `thread_local var` declaration form.
  2. `TlsCapability` is an explicit capability acquired from `RootCapability`.
  3. Safe TLS keys are explicit values created through `TlsCapability`. `ThreadLocalKey[T]` is an opaque `Linear` key type.
  4. Safe TLS payload type `T` must be `Free`. Safe TLS may not store `Linear` values.
  5. Each `ThreadLocalKey[T]` denotes one per-thread slot family. For every thread, a newly created key's slot starts empty.
  6. Child threads do not inherit parent TLS slot values implicitly. New thread means empty slots unless user code writes values explicitly.
  7. Access to safe TLS requires both the relevant `TlsCapability` and a borrow of the relevant `ThreadLocalKey[T]`, so TLS effects are visible in function signatures and data flow.
  8. Safe TLS operations act only on the current thread's slot for that key. There is no API for directly reading or mutating another thread's slot.
  9. Setting a TLS slot value is explicit. Replacing or clearing a slot returns the old value explicitly if the API removes one.
  10. Borrowing a value inside a TLS slot yields an ordinary borrow tied to the borrow scope of the access operation; it may not escape that scope.
  11. Destroying a `ThreadLocalKey[T]` consumes the key. After destruction, all access through that key is invalid.
  12. If a thread exits while a safe TLS slot still contains a `Free` value, that value is discarded as ordinary `Free` state. Because `Linear` values are forbidden here, thread exit does not create hidden linear cleanup obligations.
  13. Raw platform TLS interop remains available only through `pragma Unsafe_Module` code and the unsafe boundary. It is a separate facility from safe keyed TLS.
  **Why this fits Kyokai**: thread-local state remains available for per-thread allocators, seeds, and scratch state, but the capability model stays intact because safe TLS access is explicit in signatures rather than hidden behind ambient module state.
  **[STAGE: DECIDED_CORE_SEMANTICS | D114 → safe TLS is keyed + `TlsCapability`; no ambient `thread_local var`; raw TLS interop remains unsafe-only]**
- **Comments are line comments; documentation comments are attached comments; block comments do not exist** — Kyokai uses C-family line comment syntax, but gives documentation comments explicit structural meaning.
**Syntax**:
  1. `//` starts a normal line comment.
  2. `///` starts a documentation comment attached to the declaration that immediately follows.
  3. `//!` starts a module/file documentation comment describing the containing file/module.
  **Rules**:
  1. `/* ... */` block comments are not part of the language.
  2. A `///` doc comment must be immediately followed by the declaration it documents; unattached doc comments are errors.
  3. A `//!` module doc comment must appear before the first non-doc token in the file.
  4. Public documentation tools read doc comments from `.kyo` interface files by default; comments in `.kai` bodies remain ordinary source comments unless a tool explicitly documents private/internal code.
  **Why this fits Kyokai**: line comments are familiar, doc comments are mechanically attached to declarations instead of being free-floating prose, and banning block comments keeps lexing and tooling simpler.
  **[STAGE: DECIDED_CORE_SEMANTICS | D63 → `//`, `///`, `//!`, and no block comments]**
- **`Type`, `Free`, and `Linear` are parameter constraints, while `Auto` is a declaration-site universe classifier** — Kyokai keeps these roles separate so generic APIs do not blur "what arguments may this parameter accept?" with "what universe does this instantiated type belong to?"
**Rules**:
  1. `T: Type` means `T` may be instantiated with either a `Free` type or a `Linear` type.
  2. `T: Free` means `T` may be instantiated only with a `Free` type.
  3. `T: Linear` means `T` may be instantiated only with a `Linear` type.
  4. Inside generic code, a value whose type parameter is constrained as `T: Type` must be treated conservatively. It may be moved, borrowed, stored, returned, and passed through explicit APIs, but it may not be implicitly copied or silently discarded.
  5. Inside generic code, a value whose type parameter is constrained as `T: Free` follows the ordinary free-value rules, including the language's ordinary copy/discard behavior for free values.
  6. Inside generic code, a value whose type parameter is constrained as `T: Linear` follows the ordinary linear-use rules.
  7. `Type`, `Free`, and `Linear` constrain type parameters only. They do NOT by themselves determine the universe of the enclosing generic type constructor.
  8. `Auto` is not a parameter constraint and may not appear where a type-parameter universe constraint is expected. `Auto` is used only at a type declaration site to say that the instantiated type's universe is chosen by the constructor's classification rule.
  **Examples**:
  - If `record Wrapper[T: Type]: Auto is ... build;`, then `Wrapper[Int32]` is `Free` while `Wrapper[File]` is `Linear`.
  - If `record HandleBox[T: Type]: Linear is ... build;`, then both `HandleBox[Int32]` and `HandleBox[File]` are `Linear`.
  - If `record CopyOnly[T: Free]: Free is ... build;`, then only `Free` arguments are legal and the resulting type is always `Free`.
  **Why this fits Kyokai**: generic constraints stay explicit, the line between "accepts any type" and "becomes linear when instantiated with a linear argument" remains visible, and the type checker gets one closed rule per concept instead of a muddy hybrid.
  **[STAGE: DECIDED_CORE_SEMANTICS | D195 → `Type`/`Free`/`Linear` are parameter constraints; `Auto` is a declaration-site classifier only]**
- **Phantom type parameters are allowed on nominal types, but Kyokai does not adopt a general zero-sized-type culture or a dedicated marker-field crutch** — type-level distinctions matter, but most Rust-style ZST machinery does not pull its weight in Kyokai.
**Rules**:
  1. A nominal type may declare a type parameter that is not represented in its runtime field layout.
  2. Such a phantom parameter still participates fully in type identity, generic instantiation, and instance resolution.
  3. Kyokai does not require or standardize a dedicated marker field or `Kyokai.Phantom[T]` helper type for phantom parameters.
  4. User-defined zero-field `record` declarations are illegal.
  5. Kyokai does not introduce a general user-defined zero-sized runtime-type facility beyond the language's already-decided zero-payload forms such as `Unit` and zero-field union cases.
  **Why this fits Kyokai**: useful type-level distinctions remain available without importing a broader ZST design culture that adds little value to Kyokai's systems-language surface.
  **[STAGE: DECIDED_CORE_SEMANTICS | D181 → phantom type parameters allowed without marker fields; no dedicated `Phantom` helper; user-defined zero-field records are illegal]**
- **Universe membership comes from each type constructor's declared classification rule, not from a blanket "all generic built-ins are Auto" shortcut** — Kyokai states the classification rule explicitly for each built-in or special type form so linearity never depends on folklore.
**Constructor classification rules**:
  1. A non-generic type declared `Free` is always `Free`.
  2. A non-generic type declared `Linear` is always `Linear`.
  3. A generic type declared `Auto` is `Linear` iff at least one of its type arguments is `Linear`; otherwise it is `Free`.
  4. A generic type declared `Free` is always `Free`, regardless of its type arguments.
  5. A generic type declared `Linear` is always `Linear`, regardless of its type arguments.
**Language-defined and special-form table**:
  1. Always `Free`: `Unit`, `Bool`, `Never`, `StaticString`, all fixed-width integer types, all fixed-width floating-point types, `Index`, and the built-in target enums and other built-in enum-valued compile-time descriptors.
  2. Always `Free`: `FnPtr[...]`, `Address[T]`, `Pointer[T]`, immutable borrows `&[T]`, mutable borrows `&![T]`, and other borrow/view special forms whose purpose is observation or access rather than ownership.
  3. Always `Free`: `Vector[T, N]`, because D104 restricts the portable vector element domain to fixed-width scalars and masks that are themselves `Free`.
  4. `Auto`: `Optional[T]`, `Result[T, E]`, and `Array[T, N]`.
  5. Always `Linear`: owning and synchronization primitives whose declarations explicitly say so, including `String`, `Buffer[T]`, `Box[T]`, `PinBox[T]`, `Atomic[T]`, `Sender[T]`, `Receiver[T]`, `Mutex[T]`, `RwLock[T]`, capabilities, and other explicitly resource-owning runtime handle types.
  6. Ordinary user-defined and standard-library nominal types follow the universe marker on their own declarations: `Free`, `Linear`, or `Auto`.
  7. No type gains a universe from its name, intended use, or ABI shape. The declaration or special-form classification rule is the only source of truth.
  **Why this fits Kyokai**: the linearity checker gets a closed auditable rule set, `Auto` stays a precise mechanism instead of a catch-all slogan, and special cases like borrows, raw pointers, and function pointers stay visibly non-owning.
  **[STAGE: DECIDED_CORE_SEMANTICS | D194 → explicit constructor-by-constructor universe classification for built-ins and special forms; no blanket generic-built-in rule]**
- **Type identity is recursive and nominal, not structural** — Kyokai does not equate types because they "look the same." Two fully resolved types are identical iff their canonical forms are identical under the language's nominal rules.
**Canonicalization**:
  1. Name resolution must identify the declaration referenced by every nominal type name.
  2. `type alias` names from D50 are expanded away.
  3. Surface sugar is desugared into the underlying type form.
  4. Associated-type projections from D33 are normalized when the governing instance is known.
**Identity rules after canonicalization**:
  1. Two primitive built-in types are identical iff they are the same built-in type.
  2. Two nominal user-declared types are identical iff they refer to the same package-qualified and module-qualified declaration.
  3. Two applications of the same type constructor are identical iff the constructor is the same and each corresponding type argument is identical.
  4. For type forms that carry compile-time value parameters, the corresponding value parameters must match under that type form's own equality rule.
  5. Two reference types are identical iff they have the same reference kind and identical referent type, plus identical explicit region arguments when the named-region form is used.
  6. Two associated-type projections are identical iff, after normalization through the selected instance, the resulting canonical types are identical.
  7. `type alias` introduces no new nominal identity.
  8. Structural coincidence does not create identity. Two records, unions, or function types with the same apparent shape are still different if they arise from different declarations or different constructors.
  9. ABI equality does not create type equality. A single-field record may have the same representation as its field type while still being a distinct nominal type.
**Explicit consequences**:
  1. `UserId` is not identical to `Nat64` merely because it has the same layout.
  2. `Result[Int32, IoError]` is identical only to the same constructor applied to identical arguments.
  3. Kyokai has no structural subtyping for records, unions, or function types.
  **Why this fits Kyokai**: nominal boundaries stay visible, aliases remain honest synonyms instead of stealth newtypes, and the core type-equality relation is fully specified for generics, associated types, and special type forms.
  **[STAGE: DECIDED_CORE_SEMANTICS | D190 → recursive nominal type identity after alias expansion, desugaring, and projection normalization]**
- **Recursive and mutually recursive nominal types are legal only when their fully expanded representation has finite size** — Kyokai allows recursion, but it does not bless any type by name or permit infinite inline layout cycles.
**Rules**:
  1. A nominal type may refer to itself directly or indirectly only if every cycle in the fully expanded layout-dependency graph passes through at least one indirection-bearing field.
  2. An indirection-bearing field is one whose representation stores a fixed-size handle, pointer, or borrow rather than embedding the full referent inline.
  3. Language-defined indirection-bearing forms include `Box[T]`, `PinBox[T]`, `Address[T]`, `Pointer[T]`, `&[T]`, and `&![T]`.
  4. Other type constructors break recursion only when their own layout rules explicitly state that their parameter is represented indirectly. No type gains recursion-breaking status from its name alone.
  5. Inline constructors such as ordinary records, unions, single-field wrappers, `Optional[T]`, `Result[T, E]`, arrays, and aliases do not by themselves break a recursive layout cycle.
  6. A recursive cycle composed entirely of inline edges is ill-formed and must be rejected as an infinite-size type.
  7. The recursion check runs after alias expansion and over the whole strongly connected declaration group involved in the cycle.
  8. Mutually recursive nominal types must be declared in one explicit same-module declaration group so the compiler can validate the whole strongly connected component together.
  9. The compile-time error for an illegal recursive layout must report at least one offending cycle and identify at least one legal indirection strategy.
  **Why this fits Kyokai**: recursive data structures stay available, but layout legality is mechanical, representation-based, and free of folklore such as “`Box` is magic because the compiler felt like it.”
  **[DECIDED: D160/D217 → recursive and mutually recursive nominal types are allowed only under a representation-based finite-layout rule; explicit same-module declaration groups; infinite inline cycles are ill-formed]**
- **Kyokai generics are rank-1 only** — the language does not allow nested universal quantifiers inside value-level type expressions.
**Rules**:
  1. Type parameters may be introduced only on declarations of functions, methods, types, unions, records, typeclasses, and instances.
  2. A type expression appearing in a parameter type, field type, return type, local binding type, associated type definition, or other value-level type position may not itself introduce a new universal quantifier.
  3. Kyokai therefore has no surface form equivalent to `forall T. ...` in value positions and no rank-2, rank-N, or impredicative polymorphism.
  4. A generic function may be referenced or passed only through an ordinary fully resolved callable type at the use site. Kyokai does not make "polymorphic function value" a separate first-class type form.
  **Why this fits Kyokai**: generic signatures stay readable, diagnostics stay local, and the type system avoids a major increase in abstraction complexity that would buy little for Kyokai's systems-language surface.
  **[STAGE: DECIDED_CORE_SEMANTICS | D192 → rank-1 generics only; no higher-rank or impredicative polymorphism]**
- **Kyokai has no existential types and no opaque return types** — the language does not hide an implementation type behind a typeclass bound or `impl Trait`-style surface.
**Rules**:
  1. A type position may not quantify existentially or hide a concrete implementation type behind a bound such as `some T: Trait`, `exists T`, `impl Trait`, or any equivalent construct.
  2. A function's return type must name a fully visible concrete type expression.
  3. Stored values likewise use fully visible types. Kyokai has no erased trait-object or existential container model.
  4. Heterogeneous collections and sum-shaped abstraction boundaries use explicit unions or other explicit nominal wrapper types.
  5. This rule is consistent with D82: Kyokai has no hidden runtime dictionaries, witness tables, or trait-object dispatch.
  **Why this fits Kyokai**: abstraction boundaries remain auditable, `.koi` interfaces stay explicit, and the language does not smuggle dynamic dispatch or representation hiding in through a second typing mechanism.
  **[STAGE: DECIDED_CORE_SEMANTICS | D193 → no existential types, no opaque return types, and no trait-object-style erased containers]**
- **Typeclasses may provide default method bodies, but associated types still do not have defaults** — Kyokai allows declaration-site method reuse without adding a second dispatch model.
**Rules**:
  1. A typeclass method may be declared either as a required signature or with a default body.
  2. Every instance must implement each method that lacks a default body.
  3. An instance may override any default method body by providing its own implementation.
  4. A default body may call other methods of the same typeclass and may refer to the typeclass's associated types.
  5. Associated types themselves still have no defaults.
  **Why this fits Kyokai**: common typeclass APIs can share ordinary derived behavior without changing D81 coherence or D82's static-dispatch model.
  **[STAGE: DECIDED_CORE_SEMANTICS | D182 → default method bodies are allowed in typeclasses; instances may override; associated types still have no defaults]**
- **Kyokai does not adopt a general variance system; instead it gives `Never` a closed, explicit lifting table for the built-in sum constructors that need it** — this preserves the narrow ergonomic benefit behind variance pressure without importing hidden subtyping machinery.
**Rules**:
  1. Kyokai has no general generic-subtyping relation beyond D191's expression-site `Never` coercion.
  2. User-defined generic nominal types are invariant in all type parameters.
  3. Kyokai defines a closed built-in `Never`-lifting rule for the following constructors only:
     - `Optional[Never]` coerces to `Optional[T]` for any `T`
     - `Result[Never, E]` coerces to `Result[T, E]` for any `T`
     - `Result[T, Never]` coerces to `Result[T, E]` for any `E`
  4. No other generic constructor gains automatic lifting or variance unless a later explicit decision adds it.
  5. This rule is a closed coercion table, not inferred variance and not a user-extensible annotation system.
  6. Borrow/reference coercions such as D187 are separate explicit rules and are not implied by D186.
  **Why this fits Kyokai**: the language gets the practical `Never` ergonomics it actually needs while keeping generic type relations explicit, closed, and mechanically auditable.
  **[STAGE: DECIDED_CORE_SEMANTICS | D186 → no general variance system; user-defined generics invariant; closed `Never`-lifting for `Optional` and `Result` only]**
- **Typeclasses may declare associated types, and associated types are the only built-in way to express "this secondary type is determined by the instance"** — Kyokai adopts Rust/Swift-style associated types and does not add Haskell-style functional dependencies.
**Syntax**:
  ```kyokai
  typeclass Iterable(Self: Type) is
      type Item;
      method next(iter: &![Self]): Optional[Self.Item];
  spec;

  instance Iterable(Buffer[Int32]) is
      type Item := Int32;
      method next(iter: &![Buffer[Int32]]): Optional[Int32] is
          // implementation
      qed;
  qed;
  ```
  **Rules**:
  1. Associated types may be declared only inside a `typeclass`.
  2. Every instance must define each associated type exactly once.
  3. A projected associated type is written as `Self.Item` or `T.Item`, where the receiver type parameter is constrained by the relevant typeclass.
  4. Once the instance is chosen, the associated type is fully determined; it is not a second free dispatch parameter.
  5. Associated types have no defaults.
  6. Functional dependencies, open type families, and any second mechanism for expressing the same relation are not part of Kyokai.
  **Why this fits Kyokai**: it keeps iterator, indexing, and operator typeclasses readable without forcing every call site to restate a secondary type parameter, and it gives the language one explicit mechanism instead of two overlapping ones.
  **[STAGE: DECIDED_CORE_SEMANTICS | D33 → associated types only; no functional dependencies]**
- **Conjunctive generic constraints use one explicit `where` surface**: Kyokai does not split "simple" and "complex" generic obligations across competing multi-bound syntaxes.
**Syntax**:
  ```kyokai
  function dedupKeys[T: Type](map: &[HashMap[T, Int32]]): Bool
  where
      T: Equality,
      T: Hashable
  is
      // implementation
  qed;

  function printAll[C: Iterable](iter: &[C]): Unit
  where
      C.Item: Displayable
  is
      // implementation
  qed;
  ```
  **Rules**:
  1. The `generic [...]` header declares generic parameters and may carry one inline classifier or baseline bound per parameter such as `T: Type`, `R: Region`, or `A: Allocator`.
  2. Any additional conjunctive typeclass obligations are written in a `where` clause.
  3. Multiple obligations on the same parameter are written as repeated entries in `where`, not as `T: Foo + Bar`.
  4. Associated-type constraints are also written in the same `where` clause surface.
  5. The `where` clause appears after the generic header and before `require`, `ensure`, and `is`.
  6. Bounds inside `where` are conjunctive. Order has no semantic meaning.
  7. Duplicate constraints are compile-time errors.
  **Why this fits Kyokai**: the parameter list keeps doing one job, while all extra obligations live in one visibly separate place that scales from one additional bound to associated-type constraints without inventing a second shorthand grammar.
  **[STAGE: DECIDED_CORE_SEMANTICS | D158 -> conjunctive generic bounds use a single `where` clause surface; no `+` bound syntax]**
- **The `where` clause has one closed grammar for non-header generic obligations**: Kyokai does not leave the associated-type and equality side of generic constraints open-ended.
**Syntax**:
  ```kyokai
  function printAll[C: Iterable](iter: &[C]): Unit
  where
      C.Item: Displayable
  is
      // implementation
  qed;

  function collectBytes[I: Iterable](iter: &[I]): Unit
  where
      I.Item == Nat8
  is
      // implementation
  qed;
  ```
  **Rules**:
  1. The `where` clause appears after the generic header and before `require`, `ensure`, and `is`.
  2. `where` obligations are comma-separated and purely conjunctive.
  3. The admitted obligation forms are `T: Trait`, `Projection: Trait`, and `Projection == TypeExpr`.
  4. `Projection` means an associated-type projection such as `C.Item`.
  5. `TypeExpr` is an ordinary type expression in the current generic scope.
  6. `T: Foo + Bar` syntax does not exist. Repeated obligations stay as repeated comma-separated `where` entries.
  7. Equality obligations are checked after ordinary canonicalization, alias expansion, and associated-type projection normalization under D190.
  8. Duplicate obligations are compile-time errors. Contradictory obligations are compile-time errors.
  9. Value-level predicates do not belong in `where`; they belong in contracts or other explicit compile-time checks.
  **Why this fits Kyokai**: one bounded grammar keeps generic obligations readable, scales to associated-type constraints, and avoids a second mini-language of open-ended generic logic.
  **[STAGE: DECIDED_CORE_SEMANTICS | D189 -> one closed `where` grammar for trait bounds, associated-type bounds, and associated-type equality constraints]**
- **There is no general `Cloneable` typeclass — deep duplication remains type-specific and allocator-visible rather than becoming a blanket generic promise** — Kyokai distinguishes ordinary `Free` copy semantics from explicit creation of fresh owned duplicates.
**Rules**:
  1. The standard library defines no `Cloneable[T]` typeclass and the language provides no implicit deep-copy hook.
  2. Ordinary copy/discard behavior for `Free` values remains exactly the universe rule from D195; it is not a user-extensible cloning mechanism.
  3. Any API that creates a fresh owned duplicate of existing data is explicit and type-specific.
  4. If producing the duplicate may allocate, the API must take an explicit destination allocator and use the `...In` naming pattern from D201, for example `cloneIn(&!alloc)`.
  5. Generic code may not assume it can duplicate arbitrary `T` values unless the caller explicitly supplies the duplication operation or the concrete type's API is named directly.
  6. Linear resource types do not gain a blanket duplication escape hatch through typeclass bounds.
  **Why this fits Kyokai**: duplication cost, allocation, and ownership effects stay visible in the API that actually performs them instead of being smuggled in as a generic convenience bound.
  **[STAGE: DECIDED_CORE_SEMANTICS | D176 → no general `Cloneable` typeclass; deep duplication remains explicit, type-specific, and allocator-taking when allocation is required]**
- **There is no `Default[T]` typeclass and no generic "fabricate a value from nothing" surface** — Kyokai does not normalize partially initialized or sentinel-driven generic programming into a language trait.
**Rules**:
  1. The standard library defines no `Default[T]` typeclass and no derive-like defaulting surface.
  2. Generic code may not demand an ambient zero, empty, or sentinel value for `T`.
  3. Empty, zero, or initial values are expressed through explicit constructors, builder APIs, named constants, or caller-supplied values/callbacks.
  4. Collection and buffer APIs that need element initialization must take the element value, initialization callback, or construction operation explicitly; they do not silently fill with `T.default()`.
  5. A zero bit pattern is not a language-level value-construction rule. If some concrete type supports zero-initialized construction, that fact must be exposed by an explicit API of that type.
  **Why this fits Kyokai**: generics stay honest about where values come from, and the language does not create a universal escape hatch for half-initialized states or magic empty values.
  **[STAGE: DECIDED_CORE_SEMANTICS | D177 → no `Default[T]`; generic code must use explicit constructors, constants, or caller-supplied initialization]**
- **Call-site generic type argument inference is argument-driven, forward-only, and local to the call** — Kyokai allows useful omission of explicit type arguments when the call's inputs already determine them, but it does not let return-type context or surrounding expression context silently solve generics from a distance.
**Rules**:
  1. If a call omits explicit type arguments, the compiler may infer them only from the receiver and explicit value arguments of that call after ordinary desugaring such as UFCS.
  2. Inference proceeds left to right through the desugared call argument list.
  3. Earlier arguments may constrain later arguments. Later arguments do not retroactively reinterpret earlier argument expressions.
  4. Expected return type, assignment target type, enclosing expression context, pattern context, and surrounding generic obligations do not participate in solving omitted type arguments.
  5. Literal typing under D12 may use type arguments already solved from earlier arguments in the same call.
  6. If argument-side information leaves any omitted type parameter unsolved or ambiguous, the call is a compile-time error and the programmer must supply explicit type arguments.
  7. This rule applies uniformly to ordinary function calls, UFCS calls, generic constructors, and typeclass method calls after desugaring.
  8. Assignment target type, `let` binding annotation, typed receiving parameter, enclosing match arm type, and any other expected-result context do not solve omitted type arguments.
  9. Therefore a call such as `let buf: Buffer[Int32] := Buffer.new(alloc);` is legal without explicit type arguments only when `alloc` and any earlier explicit call arguments already determine `Int32` under rules 1 through 7.
  **Why this fits Kyokai**: common ergonomic cases such as `buf.push(42)` work, but inference stays local, predictable, and free of hidden backward reasoning from the surrounding context.
  **[DECIDED: D209/D161 → forward-only, argument-driven, left-to-right generic type argument inference; no return-context, target-type, or surrounding-expression inference]**
- **Temporaries live for the statement that created them, and immutable borrows of rvalues are allowed only within that statement** — Kyokai makes temporary lifetime extension explicit and narrow enough that numeric code stays readable without allowing borrowed temporaries to leak into longer-lived state.
**Rules**:
  1. A temporary created while evaluating a statement lives until that statement completes, unless a more local rule is explicitly stated elsewhere in the language.
  2. Immutable borrowing of an rvalue temporary is allowed within that statement when the borrow is used immediately and does not escape.
  3. Mutable borrowing of an rvalue temporary is illegal.
  4. A borrow of a temporary may not escape the statement. It cannot be stored in a local, returned, captured into a longer-lived structure, or otherwise given a lifetime beyond the statement that created the temporary.
  5. String literals and other static literals may be borrowed as `Static` data because their storage duration is defined directly by the language.
  6. `defer` captures the values and borrows visible at registration time; it does not retroactively extend the lifetime of a temporary that would otherwise be illegal to borrow.
  **Examples**:
  - Legal: `let n := norm(a + b);`
  - Legal: `let y := (m * v) + w;`
  - Illegal: `let r := &(a + b);`
  - Illegal: `return &(m * v);`
  - Illegal: `foo(&!makeBuffer());`
  **Why this fits Kyokai**: the rule is simple enough to read mechanically, supports terse math-heavy expressions, and still forbids the hidden lifetime extension behavior that makes temporary borrowing hard to audit.
  **[STAGE: DECIDED_CORE_SEMANTICS | D72 → statement-scoped temporaries; immutable rvalue borrows may not escape]**
- **UFCS does not bypass D72's rvalue-temporary borrow ban** — dot syntax stays pure desugaring, so a mutating UFCS call on an rvalue is illegal for the same reason the equivalent ordinary call is illegal.
**Rules**:
  1. If `expr.f(args)` desugars to a call whose first parameter requires `&![T]`, `expr` may not be an rvalue temporary.
  2. Therefore `getBuffer().push(42)` is a compile-time error whenever `push` expects `&![Buffer[Int32]]`.
  3. The programmer must bind the rvalue to a local first before making the mutating call.
  4. UFCS on rvalues remains legal for consuming receivers (`self: T`) and immutable-borrow receivers (`&[T]`) when the ordinary D72 temporary rules are satisfied.
  **Why this fits Kyokai**: UFCS remains one exact rewrite rule, and D72 remains the single source of truth for temporary-borrow legality.
  **[STAGE: DECIDED_CORE_SEMANTICS | D213 → mutating UFCS calls on rvalue temporaries are type errors; bind first; consuming and immutable-borrow UFCS remain legal subject to D72]**
- **Surface sugar, implicit completions, and linearity checks run in one fixed semantic order** — Kyokai does not leave desugaring order to compiler folklore or let implicit operations interact differently depending on implementation details.
**Compiler semantic order**:
  1. Parse source into a surface AST. No implicit operations are inserted during parsing.
  2. Resolve modules, imports, visibility, and names enough to reject import/name ambiguity.
  3. Perform syntax-only lowering for constructs whose meaning does not depend on types, including block-form normalization and UFCS call-shape parsing into an unresolved call-with-receiver node.
  4. Type-check and elaborate expressions, resolving overloads, generic arguments, literal types, expected-type sites, pattern types, and D254 receiver-module UFCS lookup.
  5. Insert only the closed set of D87-approved implicit completions: auto-borrow, auto-reborrow, mutable-to-immutable read reborrow, one-level field auto-deref, implicit `Unit` fallthrough, `Never` expression-site coercion, literal typing, and approved `let :=` inference.
  6. Record every inserted implicit completion as an explicit elaboration node with its source span and rule id.
  7. Run the D239 tautology-check pass over those elaboration nodes.
  8. Lower typed sugar such as `or return`, `let...else`, and `while let` into the checked core form while preserving the recorded ownership and pattern obligations.
  9. Run linearity, borrow, exhaustiveness, and capability checks on the elaborated core, not on the raw surface AST.
  10. Backend lowering starts only after these checks succeed.
  **Why this fits Kyokai**: programmers and compiler implementers can reason about one pipeline. Sugar becomes explicit before ownership checking, and every implicit completion is made auditable before backend lowering.
  **[STAGE: DECIDED_CORE_SEMANTICS | D238 → fixed syntax-lowering and typed-elaboration order before linearity]**
- **Every D87 implicit completion is recorded and checked by a mandatory tautology-check pass** — Kyokai's implicit-operation rule is a compiler invariant, not a style promise.
**Rules**:
  1. The compiler maintains a closed registry of all implicit completions admitted by D87.
  2. Each inserted completion records its rule id, source span, expected type or syntactic context, inserted core operation, and the source construct that required it.
  3. For each inserted completion, the tautology-check pass verifies that the operation is uniquely determined by the static types and surrounding context.
  4. The pass verifies that every alternative interpretation of the same source expression is statically ill-typed.
  5. The pass verifies that the insertion adds no allocation, side effect, blocking operation, capability use, or new control-flow edge beyond what the explicit source construct already implies.
  6. A compilation unit is rejected if any implicit insertion lacks a registry entry or fails one of the listed obligations.
  7. Adding a new implicit completion to Kyokai requires a new D-point or an explicit update to an existing D-point, plus a registry entry and conformance tests.
  **Why this fits Kyokai**: D87 becomes mechanically enforceable. The compiler cannot quietly grow new implicit behavior merely because an implementation shortcut was convenient.
  **[STAGE: DECIDED_CORE_SEMANTICS | D239 → compiler-maintained implicit-completion registry plus mandatory tautology-check pass]**
- **Auto-reborrow and mutable-to-immutable read reborrow require a conformance matrix over the elaborated core** — the ergonomic removal of `&~` is allowed only because the compiler proves and tests that the explicit borrow core remains sound.
**Required conformance coverage**:
  1. Owned value to immutable borrow.
  2. Owned value to mutable borrow.
  3. Mutable reference to mutable reborrow.
  4. Mutable reference to immutable read reborrow.
  5. Rejection of immutable reference to mutable borrow.
  6. Nested calls where temporary reborrows end at the correct statement boundary.
  7. Rejection of overlapping mutable reborrows.
  8. Suspension of the original mutable borrow while a read reborrow is live.
  9. Interaction with UFCS after desugaring.
  10. Interaction with `or return`, `defer`, and early exits.
  11. Rejection of escaped temporaries or returned reborrowed references.
**Rules**:
  1. The linearity checker must see explicit elaborated borrow/reborrow nodes.
  2. Auto-reborrow may never be implemented as a post-linearity backend trick.
  3. These requirements are compiler/toolchain conformance obligations. They do not add syntax or call-site verbosity to user programs.
  **Why this fits Kyokai**: the surface program stays readable, while the proof burden moves to the compiler tests where it belongs.
  **[STAGE: DECIDED_CORE_SEMANTICS | D240 → mandatory conformance matrix for auto-reborrow and read-reborrow soundness]**
- **Kyokai has no language-level undefined behavior, and unsafe exists only as a tiny set of individually specified primitives** — the language does not expose a general “here be dragons” escape hatch whose semantics are left to backend folklore.
**Rules**:
  1. For every program the Kyokai compiler accepts, the language semantics define the outcome of every Kyokai operation in that program. There is no open-ended language-level UB category in either safe or unsafe Kyokai.
  2. `unsafe` does not mean “anything can happen.” It means the programmer is using an operation whose preconditions and effects are explicitly specified and whose misuse may trigger a specified failure mode such as TPOE or runtime-fatal termination.
  3. Unsafe memory operations are exposed only through individually specified intrinsics, primitives, or FFI wrappers. If an operation cannot yet be specified precisely, it is not part of Kyokai.
  4. Kyokai does not include a general `transmute`, general type punning primitive, arbitrary pointer-to-integer / integer-to-pointer roundtrip, or any other blanket reinterpretation escape hatch.
  5. Safe Kyokai code cannot observe uninitialized memory, violate alignment, forge references, or access storage through an incompatible representation.
  6. Any future primitive dealing with raw addresses, initialization state, byte copying, unaligned access, or representation reinterpretation must state its exact alignment, provenance, lifetime, and initialization rules in the language or toolchain spec before it ships.
  7. The C backend must preserve Kyokai semantics for valid Kyokai programs without relying on C undefined behavior. If the backend needs low-level implementation tricks, it must use C patterns whose behavior is defined under the selected toolchain contract.
  8. The FFI boundary is a trust boundary. Once control enters foreign code, Kyokai's no-UB guarantee applies only to the Kyokai side of the boundary and to foreign behavior that the wrapper contract explicitly models.
  **Why this fits Kyokai**: zero UB is one of the language's unbreakable rules. The only honest way to keep that rule is to keep unsafe tiny, explicit, and fully specified rather than importing the open-ended unsafe folklore of C-family systems languages.
  **[STAGE: DECIDED_CORE_SEMANTICS | D73 → no language-level UB; only individually specified unsafe primitives; FFI is the external trust boundary]**
- **The C backend is constrained by a defined generated-C contract, and UB avoidance is enforced by both code-generation rules and required compiler flags** — Kyokai does not trust backend folklore or host-optimizer luck as its safety story.
**Rules**:
  1. Ordinary generated C for valid Kyokai programs must stay within a defined C11-compatible subset unless the selected target toolchain contract explicitly admits a named extension or intrinsic family for that backend path.
  2. The selected C compiler family, version floor, required flags, and any admitted extension families are part of the target toolchain contract under D31, D80, and D86.
  3. The C backend must preserve D71 evaluation order explicitly. It may not rely on unspecified C expression evaluation order; when needed, it introduces temporaries and statement sequencing in generated C.
  4. Kyokai integer semantics must not be defined by host-C overflow behavior. Checked arithmetic lowers to explicit checks or checked intrinsics, and the resulting TPOE behavior comes from those checks rather than from `-fwrapv`.
  5. The C backend must not emit strict-aliasing violations. Representation reinterpretation, byte copying, and type punning use `memcpy`-style or equally defined patterns only.
  6. The C backend must not emit UB-producing shifts, division/modulo by zero, uninitialized reads, or misaligned typed accesses for valid Kyokai programs. If a Kyokai operation violates its own contract, the generated code must produce the Kyokai-specified failure behavior instead.
  7. For the GCC and Clang support contract, the required defensive flags include at least `-std=c11`, `-fwrapv`, `-fno-strict-aliasing`, and `-fno-delete-null-pointer-checks`.
  8. If the backend cannot emit conforming C for the selected source construct, target, backend path, and supported C toolchain contract, the build fails. It does not silently weaken Kyokai semantics.
  9. The toolchain may define an explicit extra-assurance C-backend profile using CompCert where target support and emitted-C subset compatibility exist. Such a profile is additional assurance, not the baseline requirement for every Kyokai target.
  **Why this fits Kyokai**: the zero-UB rule remains a real backend contract instead of dissolving into "probably okay on GCC/Clang," while C emission stays valuable for bootstrap and portability work.
  **[STAGE: DECIDED_CORE_SEMANTICS | D139 → C backend must emit UB-free C under an explicit supported-toolchain contract; GCC/Clang defensive flags are mandatory; CompCert may exist as an explicit extra-assurance profile]**
- **Lowering from elaborated Kyokai to any backend must preserve Kyokai outcomes without relying on backend undefined behavior** — backend IR, generated C, LLVM metadata, and optimization choices are implementation tools, not sources of language semantics.
**Rules**:
  1. Lowering from elaborated Kyokai into C, LLVM IR, or any later backend must preserve the source program's specified Kyokai outcomes.
  2. Backend undefined behavior, LLVM `poison`/`undef`, C signed overflow, invalid aliasing assumptions, unchecked trap-producing operations, or unreachable assumptions may not be used as the mechanism for implementing safe Kyokai semantics.
  3. TPOE and runtime-fatal paths lower to explicit no-return termination operations or checked branches whose existence cannot be optimized away by assuming the failed condition is impossible.
  4. The compiler may attach aliasing, lifetime, `noalias`, alignment, initialization, or non-null metadata only when justified by the elaborated borrow, linearity, and type model.
  5. If the compiler cannot justify stronger backend metadata for a construct, it must omit the metadata or fail the build rather than silently weakening Kyokai semantics.
  6. Surface constructs with specified desugarings must lower through the D238 elaboration pipeline before linearity, borrow, capability, and backend-lowering checks rely on them.
  7. Backend optimizations may remove redundant checks only after proving the removed failure path is unreachable under Kyokai semantics, not because the target backend would treat the failing case as undefined.
  **Why this fits Kyokai**: zero language-level UB is only meaningful if code generation cannot reintroduce UB as an optimizer assumption behind the user's back.
  **[STAGE: DECIDED_CORE_SEMANTICS | D228 → defined lowering contract: backend lowering preserves Kyokai semantics and cannot rely on backend UB]**
- **Volatile memory access exists as an unsafe operation-level primitive for MMIO and other externally observable memory, not as a general type qualifier and not as synchronization** — Kyokai chooses Rust-style explicit volatile operations rather than C/Zig-style volatile-in-the-type-system propagation.
**Rules**:
  1. `readVolatile(addr: Address[T]): T` and `writeVolatile(addr: Address[T], value: T): Unit` are unsafe built-in operations available only in `pragma Unsafe_Module` code.
  2. A volatile access is externally observable. The compiler may not elide it, merge it with another volatile access, or move another volatile access across it.
  3. Volatile is not synchronization. It creates no happens-before edges and does not replace atomics, fences, mutexes, or channels for inter-thread communication.
  4. The volatile-legal type domain is closed: fixed-width integer types, `Bool`, raw address/pointer-like machine values explicitly admitted by the FFI/memory model, and `extern record` / `packed record` aggregates whose fields are recursively volatile-legal.
  5. Ordinary `record`, `union`, borrow/reference, capability, and `Linear` resource types are not volatile-legal unless a separate decision admits them explicitly.
  6. Volatile operations require naturally aligned addresses for `T` unless a separately specified unaligned volatile primitive is used.
  7. If the address is invalid, misaligned, unmapped, or otherwise faults at the hardware/runtime level, the result is runtime-fatal termination, not language-level UB.
  8. The C backend lowers volatile operations through C `volatile` loads/stores for the admitted type domain. The LLVM backend lowers them through LLVM volatile load/store operations. Backend lowering must preserve the language contract in either case.
  9. Kyokai does not add a `volatile &[T]` or `volatile &![T]` reference kind. Volatile semantics belong to the access operation itself.
  **Why this fits Kyokai**: MMIO and other externally observed accesses are available where systems code needs them, but the language avoids infecting the entire type system with C-style volatile folklore or pretending volatile has synchronization meaning it does not actually have.
  **[STAGE: DECIDED_CORE_SEMANTICS | D94/D257 → unsafe operation-level volatile access only; explicit non-synchronization contract; closed volatile-legal type domain]**
- **Moves have as-if bytewise relocation semantics, and safe movable values may not depend on their own address** — Kyokai's move model is explicit and backend-independent rather than left to folklore about what "ownership transfer" means physically.
**Rules**:
  1. Moving a value has as-if bytewise relocation semantics: the destination receives the exact representation bytes of the source value, and the source location becomes logically dead immediately after the move.
  2. The language contract is the as-if rule, not a requirement that the backend literally emit a `memcpy` call at every move site.
  3. The compiler may optimize moves away, fuse them, lower them to hidden output pointers, or keep values in registers, provided observable behavior matches the as-if bytewise relocation model.
  4. Safe code may not define or construct ordinary movable values whose validity depends on the stable address of their own storage. Self-referential values are therefore banned in safe code.
  5. Unsafe code may construct self-referential structures using raw addresses, but ordinary safe move semantics still apply unless a separate rule provides stable-address indirection or a non-movable type property.
  6. The C backend must preserve this language contract using C patterns with defined behavior; struct assignment and `memcpy`-style lowering are implementation techniques, not the language definition.
  **Why this fits Kyokai**: the programmer gets an explicit physical model for moves without tying the language to one backend primitive spelling, and the self-referentiality ban follows directly from that model instead of being folklore.
  **[STAGE: DECIDED_CORE_SEMANTICS | D89 → as-if bytewise relocation semantics; safe self-referential values banned]**
- **Large return values use guaranteed direct result placement; no build profile may reintroduce a mandatory full-width return copy** — Kyokai treats this as part of the concrete value-semantics performance contract, not as optimizer luck.
**Rules**:
  1. If a function's return type has size greater than two machine words on the selected target, the implementation must use direct result placement semantics for that return.
  2. Under direct result placement semantics, the callee initializes storage that is already designated as the caller's final result object. The implementation must not require an additional full-width relocation of that completed result merely to hand it back to the caller.
  3. This guarantee is independent of optimization level, debug-vs-release profile, and backend choice.
  4. For return types of size two machine words or smaller, the implementation may use registers, direct result placement, or another equivalent lowering, provided D89's as-if move semantics are preserved.
  5. Direct result placement does not create a safe-language stable-address guarantee and does not weaken D89's ban on ordinary self-referential movable values. It is a calling/result-lowering rule, not a pinning rule.
  6. Backends may still eliminate intermediate temporaries, keep parts of a result in registers, or lower the calling convention however they like, so long as rules 1 through 5 hold.
  **Why this fits Kyokai**: large value returns stop depending on optimization folklore, but the rule adds no hidden control flow, allocation, or side effects. It only forbids wasteful lowerings that would contradict the explicit move model.
  **[STAGE: DECIDED_CORE_SEMANTICS | D199 → guaranteed direct result placement for return types larger than two machine words; no profile/backends may reintroduce a mandatory full-width return copy]**
- **Unsafe code may not hand a movable self-referential value directly to safe code; crossing the unsafe→safe boundary requires stable-address indirection** — the unsafe boundary contract closes the soundness hole created by D89's move model without pretending the compiler already enforces first-class pinning.
**Rules**:
  1. If unsafe code constructs a self-referential value whose internal validity depends on its storage address, that value MUST cross into safe code only through a stable-address indirection container.
  2. For this rule, a stable-address indirection container is an owning value whose own move does not relocate the pointee storage.
  3. `Box[T]` is such a container for purposes of this rule: moving a `Box[T]` never relocates the boxed `T` value.
  4. A stable-address indirection container used for this rule must not provide a safe operation that extracts the protected self-referential pointee by value in a way that would relocate it. Safe access is by borrow/reference to the pointee, not by moving the pointee back out.
  5. Returning or otherwise exposing a bare movable self-referential value from `pragma Unsafe_Module` to safe code is a violation of the unsafe contract.
  6. This rule is part of the unsafe boundary specification. The compiler does not currently prove it automatically.
  7. D89b separately defines Kyokai's first-class pinned-type model; D89a remains the minimum unsafe-boundary rule even when no declaration-site pinned type is involved.
  **Why this fits Kyokai**: the current soundness hole is closed with an explicit boundary rule that is simple to audit, and the needed container semantics are stated directly instead of being left implicit in the name `Box`.
  **[STAGE: DECIDED_CORE_SEMANTICS | D89a → stable-address indirection required across unsafe→safe self-reference boundary; `Box[T]` gives non-relocating pointee storage for this purpose; first-class pinned types are defined in D89b]**
- **Kyokai has explicit declaration-site pinned types and a dedicated pinned owner; ordinary `Box[T]` remains ordinary indirection** — address stability is opt-in, visible in the type declaration, and enforced by ordinary move-checking rules rather than hidden wrapper magic.
**Rules**:
  1. Kyokai adds a declaration-site `pinned` modifier on aggregate type declarations: `pinned record` and `pinned union`.
  2. A type declared `pinned` does not participate in ordinary move semantics. Any safe operation that would relocate a pinned value is a compile-time error.
  3. Relocation-forbidden operations include passing a pinned value by value, returning it by value, assigning it by value after initialization, swapping it by value, destructuring or pattern-moving it, extracting one of its fields by value, and storing it inline in storage whose safe operations may later relocate elements.
  4. Borrows of pinned values use the ordinary reference types `&[T]` and `&![T]`. Kyokai does not add a separate `Pin[&![T]]`, `Pin[Box[T]]`, or other wrapper-based borrow family.
  5. A type that contains a pinned field inline MUST itself be declared `pinned`.
  6. `Movable` is the intrinsic opposite of `pinned`: types are movable unless declared `pinned`. Generic code may perform by-value relocation only for `T: Movable`.
  7. Any standard-library or user-defined container whose safe operations may relocate stored elements MUST require `T: Movable` for inline storage. Pinned values may appear there only behind indirection or by borrow.
  8. Safe construction of a pinned value must place it directly into its final storage location. Safe code may not construct a pinned temporary and then rely on an implicit move into place.
  9. Kyokai provides `PinBox[T]` as an owning stable-address container for pinned values. Moving a `PinBox[T]` never relocates the pointee.
  10. `PinBox[T]` must construct and destroy the pointee in place and must not provide a safe operation that extracts the pointee by value.
  11. `Box[T]` remains ordinary owning indirection. It gives separate pointee storage, but it does not itself mean "pinned" and it does not silently add a non-movable language rule.
  12. An ordinary `Box[T]` API that moves the pointee out by value, such as `unbox`, is available only when `T: Movable`.
  13. D89a's unsafe-boundary rule remains in force independently: unsafe code may pass address-sensitive ordinary types into safe code only through stable-address indirection, and safe code must not move such protected pointees out by value.
  **Why this fits Kyokai**: both mechanisms exist, but they stay explicit. `Box[T]` keeps its ordinary physical meaning, while first-class non-movability becomes a declaration-site-visible rule set that is easy to audit and much simpler than Rust's `Pin<P>` wrapper layering.
  **[DECIDED: D89b → declaration-site `pinned` types plus `PinBox[T]`; plain `Box[T]` remains ordinary indirection; relocating generics and containers require `Movable`]**
- **Out-of-memory is an explicit resource failure in ordinary APIs, not a hidden abort and not ordinary TPOE** — Kyokai treats allocation failure as a first-class error case unless an API deliberately opts into a stronger contract with a clearly visible name.
**Rules**:
  1. Ordinary allocating standard-library APIs return `Result[..., AllocError]`.
  2. OOM is not ordinary TPOE by default, because allocation failure is a resource failure rather than a violated programmer contract.
  3. If the standard library provides fatal-on-OOM convenience operations, they must be explicitly named as such (`mustReserve`, `mustPush`, `mustCreate`, or equivalent) rather than silently serving as the default API surface.
  4. Explicit fatal-on-OOM convenience operations terminate through D84's runtime-fatal category, not through hidden abort behavior and not through ordinary contract-failure TPOE wording.
  5. Zero-sized allocation requests are legal. They return a stable non-null token suitable for later deallocation with the same allocator/layout contract, but not for ordinary dereference.
  6. Reallocation failure leaves the original allocation valid, owned by the caller, and unchanged.
  7. `AllocError` is for allocation failure under a valid request. Passing an invalid size, alignment, layout, or allocator object to a low-level allocation primitive is a contract violation and triggers TPOE unless that API explicitly says otherwise.
  **Why this fits Kyokai**: hidden abort-on-OOM violates “no hidden control flow,” while treating every allocation failure as TPOE confuses resource exhaustion with programmer error. Fallible-by-default APIs keep failure visible and auditable without weakening the runtime model.
  **[STAGE: DECIDED_CORE_SEMANTICS | D74 → `AllocError` by default; explicit named fatal variants only; zero-size and realloc-failure semantics specified]**
- **Operator overloading exists only through a fixed built-in family of operator typeclasses** — Kyokai allows math-friendly operator syntax for conventional arithmetic and comparison, but it does not allow custom operator tokens, custom precedence, or arbitrary syntax-level dispatch tricks.
**Built-in operator families**:
  1. Arithmetic operators `+`, binary `-`, `*`, `/`, and unary `-` map to fixed standard typeclasses such as `Add`, `Sub`, `Mul`, `Div`, and `Neg`.
  2. Comparison operators `==`, `!=`, `<`, `<=`, `>`, and `>=` map to the standard comparison typeclasses (`Equality` and `TotalOrder`).
  3. Indexing, assignment, boolean short-circuiting, borrow syntax, field access, and any future custom operator token are outside this decision and are not user-overloadable unless a separate decision explicitly adds them.
  **Typeclass shape**:
  ```kyokai
  typeclass Add(Lhs: Type, Rhs: Type) is
      type Output;
      method add(lhs: &[Lhs], rhs: &[Rhs]): Output;
  spec;
  ```
  **Rules**:
  1. Only the language-defined operator slots may be overloaded. Users cannot declare new operators or change precedence/fixity.
  2. Overloaded arithmetic operators dispatch through their fixed standard typeclasses and associated `Output` type.
  3. Overloaded arithmetic operators receive immutable borrows of their operands by default. They are read-only, non-consuming operations.
  4. Operations that mutate or consume values must use named functions or methods such as `addAssign`, `scaleInPlace`, or `intoProduct`; they do not hide behind symbolic operators.
  5. No implicit numeric conversions are introduced by operator overloading. Operand types must already match some legal instance.
  6. D81 coherence applies: for any concrete operand types, there must be exactly one applicable operator instance.
  7. Diagnostics for operator resolution failures must report the fixed operator family involved and the candidate operand/output types.
  **Why this fits Kyokai**: numeric and matrix code stays compact, but the set of operator meanings is closed, mechanically knowable, and tied to standard typeclasses rather than user-invented syntax.
  **[STAGE: DECIDED_CORE_SEMANTICS | D23 → fixed built-in operator typeclasses only]**
- **Kyokai defines multiple runtime termination categories, and each category has its own cleanup and diagnostic contract** — the language does not collapse ordinary exit, explicit `panic`, and contract-violation TPOE into one vague `abort`.
**Termination categories**:
  1. **Ordinary completion**: returning an `ExitCode` from `main` or falling off a `Unit`-returning scope is normal control flow, not a failure mode.
  2. `**panic(message)`**: explicit programmer-requested abnormal termination.
  3. **TPOE**: runtime contract-violation termination, including bounds failures, checked-arithmetic traps, failed `require`/`ensure`, invalid checked narrowing, and any other operation the language classifies as a contract violation.
  4. **Runtime-fatal/internal failure**: hard termination for failures outside the contract system, such as runtime corruption or impossible internal runtime/toolchain failures. This category exists so the spec does not pretend every fatal stop is user-visible TPOE.
  **Rules**:
  1. Ordinary completion runs the normal structured cleanup path for the scopes that exit.
  2. `panic(message)` terminates the whole process and is never recoverable inside Kyokai.
  3. `panic(message)` runs ordinary `defer` cleanup on the scopes it exits before terminating, but it does not become a catchable exception or unwinding mechanism.
  4. TPOE is immediate hard termination. User `defer` code does not run on TPOE.
  5. Runtime-fatal/internal failure follows the same hard-stop contract as TPOE unless a more specific runtime rule explicitly says otherwise.
  6. Neither `panic` nor TPOE may unwind across an FFI boundary. Crossing a foreign boundary with language-level unwinding is not part of Kyokai.
  7. `panic` and TPOE may emit a best-effort diagnostic message before termination, but stack trace format, debug metadata policy, and other diagnostic presentation details belong to the debugging/toolchain contract rather than the core language semantics.
  8. Program exit by `ExitCode` is semantically distinct from `panic` and TPOE even if an operating system ultimately represents all three outcomes with process exit codes.
  9. The exact numeric exit-code conventions for runtime-reserved failures belong to the toolchain/runtime contract, not the core language.
  **Why this fits Kyokai**: the programmer can tell the difference between an expected return, an explicit "stop now" request, and a contract failure that invalidates the current execution. The language stays explicit without drifting into exception-style recovery semantics.
  **[STAGE: DECIDED_CORE_SEMANTICS | D84 → explicit split between ordinary exit, `panic`, TPOE, and runtime-fatal termination]**
- **Fault isolation is process-supervision, not in-process catching of `panic` or TPOE** — Kyokai preserves D84's hard-stop semantics while still giving high-availability software an explicit architecture for isolating faulty work.
**Rules**:
  1. Kyokai provides no `catch panic`, `catch TPOE`, in-process panic recovery, task-level fault recovery, or ordinary structured-concurrency recovery from D84 fatal categories.
  2. A Kyokai task, OS thread, channel, `select`, `Poller`, or structured-concurrency scope is not a fault-isolation boundary.
  3. Malformed external input must be handled as ordinary data through parsing, validation, and `Result`, not by relying on contract failure and recovery.
  4. The standard fault-isolation mechanism for servers and other high-availability systems is supervised OS worker processes.
  5. A supervisor process may spawn workers through D178 `ProcessCapability`, communicate through explicit IPC/channel-like APIs, observe worker exit status, and restart workers according to an explicit policy.
  6. Capabilities do not cross into a worker implicitly. Any authority granted to a worker must be passed explicitly through worker configuration or the worker bootstrap/IPC protocol.
  7. Worker crash, `panic`, TPOE, runtime-fatal termination, or abnormal OS termination is reported to the supervisor as ordinary process-status data.
  8. Kyokai does not define in-process isolates in the core language. If in-process isolates are ever added, they require a separate D-point covering isolated heaps, capability membranes, FFI restrictions, resource limits, scheduler interaction, and termination semantics.
  **Why this fits Kyokai**: a contract failure means the current execution is invalid. The language should not fake recovery inside the same execution context; the honest boundary is another process with explicit authority and IPC.
  **[STAGE: DECIDED_CORE_SEMANTICS | D253 → no in-process catch for `panic`/TPOE; fault isolation uses supervised OS worker processes]**
- **Stack overflow is never language-level UB on a conforming Kyokai target; it is a defined runtime-fatal termination that must be detected before silent corruption on both hosted and freestanding targets** — Kyokai does not weaken its safety story at the exact boundary where systems targets are most vulnerable.
**Rules**:
  1. Safe Kyokai stack overflow is never language-level UB.
  2. Stack overflow enters D84's runtime-fatal/internal-failure category, not ordinary TPOE. It is an abstract-machine failure rather than a violated programmer contract.
  3. Hosted targets must guarantee detection before silent stack corruption. Guard pages alone are sufficient only when the target/runtime contract guarantees they cannot be skipped by large frame growth; otherwise stack probing or an equivalent mechanism is required.
  4. Freestanding targets must also guarantee detection before silent stack corruption. A conforming freestanding Kyokai target must provide an explicit stack-bounds contract that the runtime/toolchain can enforce.
  5. Stack-bound enforcement on freestanding targets is part of target conformance, not an optional safety flag. If a target configuration cannot provide the required enforcement, it is not a conforming safe Kyokai target.
  6. Runtime/toolchain mechanisms for this enforcement may include guard pages, stack probes, explicit bound checks, fixed-limit metadata, or any equivalent implementation technique, provided the observable language contract remains the same.
  7. The exact probe sequence, page size strategy, linker symbol names, and other implementation details belong to the toolchain/runtime contract, but they may not weaken rules 1 through 5.
  **Why this fits Kyokai**: the language keeps its zero-UB guarantee intact on embedded and hosted systems alike, while still leaving backend/runtime engineers freedom in how they implement the necessary checks.
  **[STAGE: DECIDED_CORE_SEMANTICS | D115/D262 → stack overflow is runtime-fatal, never UB; hosted and freestanding targets must detect before corruption using guard pages, probes, bounds checks, or equivalent enforcement]**
- **Program startup inputs are passed explicitly to `main`, not exposed through ambient global functions** — command-line arguments are ordinary program inputs and therefore belong in the entry-point signature instead of being readable from anywhere in the program.
**Syntax**:
  ```kyokai
  function main(root: RootCapability, args: &[Span[String]]): ExitCode is
      // ...
  qed;
  ```
  **Rules**:
  1. The program entry point receives the root capability and the command-line argument list explicitly.
  2. `args` is an immutable borrowed view of the argument vector supplied by the runtime.
  3. The argument list excludes the program name. It contains only the user-supplied command-line arguments.
  4. `ExitCode` is a standard library built-in union, not a raw integer:
    ```kyokai
     union ExitCode is
         case ExitSuccess;
         case ExitFailure(code: Int32);
     build;
    ```
  5. There are no global `argumentCount()` / `nthArgument()` built-ins in Kyokai.
  6. Any code below `main` can observe arguments only if `main` passes them onward explicitly.
  **Why this fits Kyokai**: command-line inputs stop being hidden global state, capability-based startup remains honest, and the program's external inputs are visible in the entry-point signature.
  **[STAGE: DECIDED_CORE_SEMANTICS | D48 → pass CLI arguments explicitly to `main`; no ambient argument globals]**
- **Hosted and freestanding targets both have explicit bootstrap contracts, and user code never constructs `RootCapability`** — startup is part of the language contract, not ambient folklore hiding behind a CRT.
**Rules**:
  1. `RootCapability` has no user-visible constructor. It is minted only by the target's language-defined bootstrap path.
  2. On hosted targets, the toolchain provides the ABI entry shim, gathers the hosted startup inputs required by the target contract, constructs the unique `RootCapability`, materializes `args`, and then calls `main(root, args)`.
  3. Hosted startup glue may be implemented with target-specific CRT or ABI machinery, but that machinery serves one fixed Kyokai startup contract rather than introducing a second ambient runtime semantics.
  4. `SystemAllocator` is not a hidden global and not an implicit extra parameter to `main`. Code obtains it explicitly from `RootCapability` or from another capability or handle whose derivation from the root is itself explicitly specified.
  5. If a target contract does not provide a process heap, the target must not pretend that `SystemAllocator` exists. Allocation availability is an explicit part of target conformance.
  6. On hosted targets, startup must either supply `main` with arguments representable under the language's argument/text contract or terminate before `main` in D84's runtime-fatal/internal-failure category. It may not fabricate lossy or ill-formed `String` values.
  7. On freestanding targets, the target contract defines the raw entry symbol and bootstrap sequence that first acquires the platform inputs and then enters ordinary Kyokai code with a minted `RootCapability`.
  8. After bootstrap hands control to ordinary Kyokai code, startup inputs and authority remain explicit parameters. Freestanding targets do not get ambient globals as a substitute for a hosted CRT.
  **Why this fits Kyokai**: authority has an auditable origin, startup behavior stops being implementation folklore, and freestanding support stays explicit without inventing a hidden general-purpose runtime.
  **[STAGE: DECIDED_CORE_SEMANTICS | D162 → explicit hosted bootstrap to `main(root, args)`; explicit freestanding bootstrap contract; `RootCapability` is minted only by startup; allocator availability is target-contract-defined]**
- **Authority tokens use sealed `capability` declarations and cannot be forged by unsafe modules** — Kyokai makes the unforgeability property explicit instead of relying on ordinary record opacity plus informal unsafe-module discipline.
**Syntax**:
  ```kyokai
  capability RootCapability;
  capability FileCapability;
  capability ProcessCapability;
  ```
**Rules**:
  1. A `capability` declaration defines a linear authority token type with no user-visible constructors, no record fields, and no literal form.
  2. Capability values may be minted only by the language-defined runtime bootstrap or by explicitly specified acquire, split, derive, or surrender/return operations that already require the appropriate parent capability.
  3. `pragma Unsafe_Module` does not grant permission to forge capabilities, bypass capability constructors, manufacture authority from raw bits, or construct a capability by layout knowledge.
  4. Capability construction, derivation, and destruction operations are part of each capability's documented authority contract and must state which parent authority they require.
  5. Unsafe modules may wrap foreign authority only by returning a capability through a specified trusted acquire/constructor path documented in that module's unsafe contract.
  6. A safe or unsafe module that needs new external authority must receive or derive the appropriate capability explicitly; no module may synthesize authority from absence.
  **Why this fits Kyokai**: capability-based security depends on authority tokens being unforgeable even at the unsafe boundary. Unsafe code can perform specified low-level operations, but it does not get to mint ambient authority outside the documented bootstrap/acquire chain.
  **[STAGE: DECIDED_CORE_SEMANTICS | D255 → first-class sealed `capability` declarations; unsafe code cannot construct capabilities directly]**
- `**panic(message)` is the built-in syntax for explicit programmer-requested abnormal termination** — it is not a synonym for TPOE and it is not a general-purpose `abort` primitive with ambiguous cleanup behavior.
**Rules**:
  1. `panic(message)` is a core-language construct, not an ordinary library function.
  2. It enters the `panic` termination category defined by D84.
  3. It is never recoverable inside Kyokai.
  4. It runs ordinary `defer` cleanup exactly as specified by D84 and D2b.
  5. It does not classify the stop as TPOE; manual programmer-requested termination and runtime contract failure remain different semantic categories.
  6. The message expression must be a string literal or a `String`.
  **Why this fits Kyokai**: the programmer writes an explicit stop request with explicit message text, but the language keeps that path semantically distinct from contract-violation hard failure.
  **[STAGE: DECIDED_CORE_SEMANTICS | D49 → `panic(message)` as built-in explicit abnormal termination]**
- **Hosted `panic` and TPOE have concrete abnormal-termination behavior, while freestanding targets route through an explicit fatal hook** — Kyokai fixes the observable stop path instead of leaving it to backend folklore.
**Rules**:
  1. On hosted targets, `panic`, TPOE, and runtime-fatal/internal failure perform a best-effort diagnostic write to `stderr` before invoking the hosted abnormal-termination primitive.
  2. The hosted diagnostic must preserve the termination category textually at minimum for `panic` versus `TPOE`. Runtime-fatal/internal failure may use its own distinct category label.
  3. Failure to emit the diagnostic does not change the termination path.
  4. After any cleanup already required by D84 has completed, hosted abnormal termination proceeds through the target's abnormal-termination primitive rather than through ordinary `ExitCode` return.
  5. On POSIX-like hosted targets, that abnormal-termination primitive is `abort()`.
  6. A non-POSIX hosted target may use an equivalent target-native abnormal-termination primitive only if it preserves D84's no-recovery, no-language-level-unwinding contract.
  7. TPOE does not unwind and does not run user `defer`; `panic` runs only the `defer` cleanup already required by D84 before the abnormal-termination primitive is invoked.
  8. On freestanding targets, `panic`, TPOE, and runtime-fatal/internal failure transfer control to the target's fatal-termination hook rather than assuming `stderr` or `abort()` exists.
  9. That freestanding fatal hook must be non-returning and must receive enough information to distinguish at least `panic` from `TPOE`, plus the diagnostic message text.
  10. Numeric exit-code conventions for these fatal paths remain toolchain/runtime policy under D84 rather than new core-language semantics.
  11. Backtrace capture and formatting remain governed by D170 and do not alter the concrete termination primitive defined here.
  **Why this fits Kyokai**: process-fatal paths become observable and portable at the semantic level without collapsing distinct failure categories into one vague “abort somehow” rule.
  **[STAGE: DECIDED_CORE_SEMANTICS | D163 → hosted fatal paths are best-effort `stderr` diagnostics plus abnormal termination; freestanding fatal paths go through a non-returning target hook carrying category + message]**
- **Freestanding fatal termination uses one compiler-known root-module hook with a fixed signature** — Kyokai gives bare-metal and kernel targets an explicit non-returning customization point without splitting fatal behavior across multiple ad hoc names or attributes.
**Syntax**:
  ```kyokai
  union FatalKind is
      case Panic;
      case Tpoe;
      case RuntimeFatal;
  build;

  function fatalHandler(kind: FatalKind, message: &[String], trace: Optional[Backtrace]): Never;
  ```
  **Rules**:
  1. `fatalHandler` is a reserved root-module hook name for freestanding executable targets.
  2. On freestanding targets, every fatal termination path from `panic`, TPOE, and runtime-fatal/internal failure transfers control to `fatalHandler`.
  3. `fatalHandler` must be non-returning.
  4. `kind` distinguishes at least `Panic`, `Tpoe`, and `RuntimeFatal`.
  5. `message` is the fatal diagnostic text prepared under D163.
  6. `trace` carries backtrace data when D170 says one exists on the current target and build; otherwise it is `None`.
  7. Missing `fatalHandler` on a freestanding executable target is a compile-time error.
  8. A `fatalHandler` declaration with the wrong signature is a compile-time error.
  9. Hosted targets do not consult `fatalHandler` for language-level fatal termination.
  10. Kyokai does not add a second handler name for TPOE versus panic. `FatalKind` is the explicit distinction surface.
  **Why this fits Kyokai**: freestanding targets get an explicit hook that can log over UART, halt, reset, or trap, but the language keeps one visible fatal path and one mechanically checkable signature.
  **[STAGE: DECIDED_CORE_SEMANTICS | D169 → reserved root-module `fatalHandler(kind, message, trace): Never` for freestanding fatal termination; missing or mismatched hook is a compile-time error]**
- **Backtrace policy is explicit, build-sensitive on hosted targets, and passed as data on freestanding targets** — Kyokai keeps fatal diagnostics useful without pretending every target has the same trace machinery or hiding policy inside unnamed runtime defaults.
**Rules**:
  1. Backtrace policy applies to `panic`, TPOE, and runtime-fatal/internal failure.
  2. On hosted debug builds, the runtime must attempt to capture and print a backtrace by default.
  3. On hosted test builds, the runtime must also attempt to capture and print a backtrace by default.
  4. On hosted release builds, the runtime attempts to capture and print a backtrace only when `KYOKAI_BACKTRACE=1`.
  5. If backtrace capture is unavailable or fails, termination still proceeds; only the trace is omitted.
  6. The base fatal diagnostic from D163 is emitted regardless of backtrace availability.
  7. `Backtrace` is an opaque runtime/toolchain-defined `Free` value type representing captured stack-trace data for a fatal event.
  8. On freestanding targets, there is no environment-variable-controlled backtrace policy.
  9. Freestanding targets communicate trace availability through the `trace: Optional[Backtrace]` argument passed to `fatalHandler`.
  10. If the freestanding target/runtime cannot produce a trace, it passes `None`.
  11. Textual rendering format, frame metadata policy, symbolization quality, and debugger integration belong to the toolchain/runtime spec under D86 rather than the core language.
  **Why this fits Kyokai**: hosted CLI behavior stays practical, freestanding behavior stays explicit, and the language avoids both silent trace folklore and fake promises that every target can symbolize a stack.
  **[STAGE: DECIDED_CORE_SEMANTICS | D170 → hosted debug/test builds attempt backtraces by default, hosted release is `KYOKAI_BACKTRACE=1`-gated, and freestanding targets pass trace data to `fatalHandler` as `Optional[Backtrace]`]**
- **`todo;` and `unreachable;` are built-in diverging statements with distinct failure categories** — Kyokai standardizes both developer stubs, but it does not blur them into one generic abort.
**Rules**:
  1. `todo;` has type `Never`.
  2. Executing `todo;` enters D84's `panic` category with a compiler-provided diagnostic message that includes source location.
  3. The compiler should warn on `todo;` in non-test code; profile policy may escalate that warning, but the runtime semantics remain an explicit `panic`.
  4. `unreachable;` has type `Never`.
  5. Executing `unreachable;` enters D84's TPOE category.
  6. `unreachable;` is not optimizer-only poison and not UB. If execution reaches it, the language-defined result is TPOE.
  7. `unreachable;` may inform control-flow analysis that the path is semantically impossible, but that analysis may not change rule 6.
  **Why this fits Kyokai**: unfinished code and impossible code paths get first-class spellings, while the language still keeps programmer-requested failure (`todo;`/`panic`) distinct from contract-violation failure (`unreachable;`/TPOE).
  **[STAGE: DECIDED_CORE_SEMANTICS | D121 → built-in `todo;` and `unreachable;` with explicit panic-vs-TPOE semantics]**
- **Deferred actions run in LIFO order within each lexical scope, filtered by the exit path that is actually being taken** — Kyokai treats `defer`/`errdefer` registration like a per-scope stack so cleanup order is mechanically predictable.
**Rules**:
  1. Each lexical scope maintains its own deferred-action stack.
  2. Entering a nested lexical scope creates a new deferred-action stack for that scope.
  3. Exiting a scope walks that scope's deferred-action stack in reverse source order.
  4. Only deferred actions whose trigger condition matches the current exit path run; ineligible deferred actions are skipped, not reordered.
  5. Inner-scope deferred actions always run before outer-scope deferred actions.
  **Why this fits Kyokai**: it matches the natural acquisition/release stack discipline, keeps resource lifetimes locally understandable, and stays explicit even when `defer` and `errdefer` coexist in the same scope.
  **[STAGE: DECIDED_CORE_SEMANTICS | D2a → LIFO per lexical scope, over all deferred actions eligible on the current exit path]**
- **The exit paths that run deferred actions are defined explicitly** — Kyokai does not leave "scope exit" as an implementation guess.
**Rules**:
  1. Ordinary `defer` actions run on ordinary scope completion and on every structured exit from the scope: normal fallthrough, `return`, `break`, `continue`, `or return`, `or break`, and `or continue`.
  2. Ordinary `defer` actions also run on `panic(message)`, because `panic` is explicit language-level abnormal control flow rather than a contract-failure trap.
  3. `errdefer` runs only on the structured error exits for that scope.
  4. The only structured error exits currently defined by the language are `return Err(value)` and `or return`, because `or return` is exact sugar for a `return Err(...)` path.
  5. `break`, `continue`, `or break`, `or continue`, normal fallthrough, successful `return`, and `panic(message)` are not `errdefer`-triggering exits.
  6. TPOE does not run user `defer` or `errdefer` code. TPOE is immediate hard termination.
  7. If leaving one scope also exits outer scopes, each exited scope applies these rules independently, using D2a's reverse-order rule within that scope.
  **Exit-path matrix**:
  | Exit path                         | `defer` | `errdefer` |
  | --------------------------------- | :-----: | :--------: |
  | Normal fallthrough                |   Yes   |     No     |
  | `return value` on success path    |   Yes   |     No     |
  | `return Err(value)`               |   Yes   |    Yes     |
  | `or return`                       |   Yes   |    Yes     |
  | `break` / `continue`              |   Yes   |     No     |
  | `or break` / `or continue`        |   Yes   |     No     |
  | `panic(message)`                  |   Yes   |     No     |
  | TPOE                              |    No   |     No     |
  **Example**:
  ```kyokai
  function readChecked(path: Span[Nat8, Static]): Result[Buffer[Nat8], IoError] is
      let file: File := openFile(path) or return;
      defer closeFile(file);
      let bytes: Buffer[Nat8] := readAll(&file) or return;
      errdefer bytes.destroy();
      validateMagicBytes(&bytes) or return;
      return Ok(bytes);
  qed;
  ```
  If `openFile(path) or return;` fails, nothing has been registered yet, so no deferred action runs. If `readAll(&file) or return;` fails, the `or return` exit runs `closeFile(file)` through ordinary `defer`. If `validateMagicBytes(&bytes) or return;` fails, the `or return` exit runs `bytes.destroy()` through `errdefer`, then runs `closeFile(file)` through ordinary `defer`. On the success path, `bytes` is moved into `Ok(bytes)`, so `errdefer bytes.destroy();` does not run, while `defer closeFile(file);` still does.
  **Why this fits Kyokai**: explicit abnormal termination (`panic`) still behaves like visible language control flow, while TPOE remains the hard contract-failure stop that does not execute user cleanup code.
  **[DECIDED: D2b → structured exits and `panic` run ordinary `defer`; TPOE runs no user deferred actions]**
- **Deferred bodies may not perform nonlocal structured control flow** — Kyokai does not let cleanup code replace or redirect the exit path that caused the cleanup to run.
**Rules**:
  1. A `defer` or `errdefer` body may use ordinary local control flow that stays entirely inside that deferred body, including `if`, `case`, and nested loops whose `break`/`continue` target those nested loops.
  2. A `defer` or `errdefer` body may not perform a nonlocal structured exit from the surrounding function or surrounding non-deferred scope.
  3. The banned nonlocal exits are `return`, `or return`, `break`, `or break`, `continue`, and `or continue` whenever they would leave the surrounding scope rather than an inner loop contained in the deferred body.
  4. Violating rule 3 is a compile-time error.
  5. `panic(message)` is legal inside a `defer` or `errdefer` body.
  **Why this fits Kyokai**: cleanup stays cleanup. The programmer can write local logic inside a deferred body, but the body cannot silently replace the outer control-flow path or corrupt the deferred-action stack.
  **[STAGE: DECIDED_CORE_SEMANTICS | D207 → deferred bodies allow only local control flow; nonlocal structured exits are illegal; `panic` remains legal]**
- `**Never` is a built-in bottom type for expressions that are statically known not to complete normally** — Kyokai does not leave diverging control-flow typing to compiler-only special cases.
**Rules**:
  1. `Never` has no values and no constructors.
  2. `return`, `break`, `continue`, and `panic(message)` have type `Never`.
  3. Any expression form that the language statically knows cannot complete normally also has type `Never`.
  4. `Never` may appear in user-written type positions, especially as a function return type for functions that never return normally.
  5. `Never` coerces to any other type for branch/type-join purposes.
  6. An expression does not become `Never` merely because it may trigger TPOE at runtime. Potential contract failure is not the same thing as statically known divergence.
  7. An expression that may trigger TPOE at runtime still has its ordinary result type for typing purposes; Kyokai does not track "may panic" or "may TPOE" in the type system.
  **Examples**:
  - `function fail(msg: String): Never is panic(msg); qed;`
  - `let x: Int32 := if ok then value else return;`
  - `let y: Float64 := parseRatio(s) or return;`
  **Why this fits Kyokai**: the type system states directly when control flow cannot continue, which makes `let...else`, `or return`, explicit `panic`, and no-return APIs type-check for principled reasons instead of checker magic.
  **[STAGE: DECIDED_CORE_SEMANTICS | D58 → built-in, denotable `Never` bottom type]**
- **`Never` coercion is an expression-site rule, not a general subtyping relation** — Kyokai uses `Never` to type-check statically diverging expressions where an ordinary value is expected, but it does not turn `Never` into a full subtype lattice that would silently rewrite enclosing generic types.
**Rules**:
  1. An expression of type `Never` may satisfy any expected type at that same expression site.
  2. This coercion applies only where the language is typing an expression against an immediate expected type or joining expression branches, including function return expressions, argument positions with known parameter types, `if` joins, `case` joins, `let ... else`, and `or return`.
  3. The coercion does not rewrite enclosing type constructors. `Optional[Never]`, `Result[Never, E]`, `Array[Never, N]`, and similar instantiated types remain ordinary distinct types rather than becoming universal subtypes.
  4. Kyokai therefore has no general subtype relation induced by `Never`. D186's variance rules, if any, are a separate question and are not inherited automatically from D58.
  5. `Never` in a user-written type position remains an ordinary denotable zero-inhabitant type under D58. The special rule here is only the expression-site coercion.
  **Why this fits Kyokai**: it solves the actual typing problem for diverging control flow without forcing the whole generic system to acquire hidden subtype and variance behavior.
  **[STAGE: DECIDED_CORE_SEMANTICS | D191 → `Never` coercion is expression-site only; no general subtyping and no automatic lifting through enclosing type constructors]**
- **Expression evaluation order is strict and uniform** — Kyokai evaluates subexpressions in source order instead of leaving sequencing to backend folklore or construct-specific exceptions.
**Rules**:
  1. Unary operators evaluate their operand first, then perform the operator.
  2. Binary operators evaluate the left operand, then the right operand, then perform the operator.
  3. D23 overloaded operators follow the same sequencing rule: both operands are evaluated exactly once, left to right, before the fixed operator-typeclass method is invoked.
  4. Function calls evaluate the callee expression first, then each argument from left to right, then perform the call.
  5. UFCS calls evaluate the receiver first, then the remaining arguments from left to right, then perform the call.
  6. Field access evaluates the base expression first.
  7. Indexing evaluates the container expression first, then the index expression, then performs bounds checking and element access.
  8. Record and array literal fields and elements evaluate in source order.
  9. `if`, `while`, and `case` evaluate their condition or scrutinee before any branch or body selection; only the selected branch/body is then evaluated.
  10. Assignment statements evaluate the assignee-place subexpressions from left to right, then evaluate the right-hand side, then perform the store.
  11. D15a's `let...else` scrutinee is evaluated exactly once before pattern testing; the language does not permit any desugaring that duplicates scrutinee evaluation.
  12. Short-circuiting `and` / `or` is the only ordinary conditional-evaluation exception.
  **Why this fits Kyokai**: the reader can predict exactly which side effects, borrows, checks, and possible TPOE points happen first by reading left to right. The rule is more explicit than Go's partial sequencing, cleaner than Rust's assignment special cases, and intentionally rejects the operand-order looseness found in Ada and C-family prior art.
  **[STAGE: DECIDED_CORE_SEMANTICS | D71 → strict left-to-right evaluation for all ordinary subexpressions and statement parts]**
- **Built-in integer arithmetic is exact-by-spec, checked by default, and free of implicit promotions** — Kyokai defines integer operators in terms of fixed-width concrete types and explicit trap behavior instead of inheriting C-family conversion rules.
**Rules**:
  1. Built-in signed integer types are two's-complement fixed-width integers. Unsigned integer types are fixed-width modulo-2^N integers used only where their declared type says so.
  2. There are no implicit promotions, no usual arithmetic conversions, and no mixed signed/unsigned operator rules.
  3. A built-in binary integer operator is legal only when both operands have the same concrete integer type after literal inference and any explicit conversions have already been applied.
  4. For `+`, binary `-`, `*`, and unary `-`, the language computes the exact mathematical result and produces that value only if it is representable in the concrete operand/result type; otherwise the operation triggers TPOE.
  5. Integer division truncates toward zero.
  6. Integer remainder is the truncating remainder and has the sign of the dividend.
  7. `x / 0`, `x % 0`, and `IntMin / -1` for a signed two's-complement type trigger TPOE.
  8. Comparisons on built-in integers compare values of the same concrete integer type only; cross-type comparison requires an explicit conversion first.
  9. Wrapping, saturating, overflow-reporting, and other non-default arithmetic policies exist only through explicitly named library APIs.
  10. No build profile, target setting, backend flag, or user-supplied toolchain flag may silently change these language-level semantics.
  **Why this fits Kyokai**: overflow becomes an explicit, portable part of the language contract and of D84's TPOE model, while the absence of promotions prevents the hidden type motion that makes C-family integer code error-prone.
  **[STAGE: DECIDED_CORE_SEMANTICS | D75 → no promotions; checked-by-default exact integer arithmetic with TPOE on overflow]**
- **Floating-point arithmetic follows a strict IEEE 754 language contract** — Kyokai does not merely "use the platform float"; it specifies the visible model that the backend must preserve.
**Rules**:
  1. `Float32` and `Float64` are IEEE 754 binary32 and binary64 respectively.
  2. The default rounding mode is round-to-nearest, ties-to-even.
  3. Arithmetic, conversion, and comparison results must be those of the IEEE 754 model for the declared operand/result types; the backend may not rely on excess precision as part of the portable language result.
  4. Subnormals and gradual underflow are part of the language contract. Flush-to-zero and denormals-are-zero behavior are not silently permitted.
  5. Signed zero is preserved and observable.
  6. Floating-point division by zero does not trigger TPOE solely because the divisor is zero; it yields the IEEE 754 result.
  7. Floating overflow, underflow, infinities, and NaNs follow IEEE 754 behavior rather than the integer TPOE model.
  8. Ordered comparisons involving NaN are false; `!=` involving NaN is true; ordinary float comparison is therefore a partial order, not a total order.
  9. NaN payload bits are outside the portable language contract. Portable Kyokai code may rely on the result being NaN, but not on a specific payload-propagation policy.
  10. User-visible mutation of the floating-point environment is not part of the language contract unless a separate decision adds it explicitly.
  11. The compiler, backend, and toolchain configuration may not silently enable fast-math, reassociation, excess-precision drift, or implicit FMA contraction that changes language-visible results.
  12. If the language later adds a relaxed or fast floating-point mode, that must be a separate explicit decision with separate syntax and semantics; it is not implied by optimization level or profile name.*
  **Why this fits Kyokai**: float behavior stays portable and auditable across the C backend, target triples, and profile settings. The language gets Java-style explicitness about the visible model without inheriting C's "whatever the toolchain did" semantics.
  **[STAGE: DECIDED_CORE_SEMANTICS | D76 → strict IEEE 754 binary32/binary64 model; portable NaN-ness but not payload bits]**
- **Standard-library floating-point math APIs must publish tested accuracy contracts without adding call-site tier types** — Kyokai requires numerical honesty as part of the API contract, but it does not make ordinary users carry accuracy parameters through every math call.
**Allowed contract forms**:
  1. `Exact`: the result is exact for every input in the documented domain.
  2. `CorrectlyRounded`: the result is the IEEE 754 correctly rounded result under D76's active rounding rule.
  3. `MaxUlp(n)`: the result is within `n` ULP of the mathematical real-number result over the documented input domain.
  4. `AbsError(bound)` or `RelError(bound)`: allowed only when ULP is not the right error statement for the function, and the input domain must be stated explicitly.
  5. `ImplementationDefinedAccuracy` is not an allowed contract for safe `Kyokai.Math` APIs.
**Rules**:
  1. Every public `Kyokai.Math` floating-point function must declare one of the allowed accuracy contracts in its API documentation and conformance metadata.
  2. Accuracy contracts do not change ordinary function signatures. `sin(x)` remains `sin(x)`, not `sin[Tier](x)`.
  3. Accuracy tiers are documentation, tests, admission criteria, and toolchain-doc metadata, not programmer-facing type parameters.
  4. Each function must document special-value behavior for `NaN`, infinities, signed zero, overflow, underflow, and domain boundaries under D76.
  5. A math implementation may not be admitted to the standard library until tests demonstrate the declared contract over the documented domain using reference implementations, high-precision oracles, exhaustive checks where feasible, and fuzzing where exhaustive checks are infeasible.
  **Why this fits Kyokai**: RIIK math stays pure Kyokai, but the rewrite must be numerically accountable. The programmer gets ordinary readable calls, while the standard library carries the explicit error bounds and tests that make those calls trustworthy.
  **[STAGE: DECIDED_CORE_SEMANTICS | D232 → math APIs publish tested accuracy contracts; no call-site numerical tier verbosity]**
- **Default numeric conversion uses a fixed UFCS-style conversion family; alternative conversion policies stay named** — Kyokai keeps numeric conversion concise without adopting Rust's overloaded `as`, C's cast syntax, or pseudo-constructor conversion forms.
**Syntax**:
  ```kyokai
  let a: Int64 := x.toInt64();
  let b: Int32 := y.toInt32();
  let c: Float32 := z.toFloat32();
  let d: Optional[Int32] := y.tryToInt32();
  let e: Int32 := y.truncateToInt32();
  let f: Int32 := y.saturatingToInt32();
  ```
  **Rules**:
  1. For built-in numeric target types only, `expr.toTargetType()` is the syntax for the default explicit numeric conversion.
  2. This conversion family is fixed by the language for built-in numeric conversions; it is not user-overloadable and does not automatically generalize to user-defined types.
  3. Integer-to-integer conversions using `toTargetType` preserve the numeric value if representable in the target type and otherwise trigger TPOE.
  4. Integer sign changes are checked by the same rule: converting a negative signed integer to an unsigned target, or any out-of-range value to any target, triggers TPOE.
  5. Float-to-integer conversion using `toTargetType` truncates toward zero and then checks the truncated value against the integer target range. NaN, infinity, and out-of-range results trigger TPOE.
  6. Integer-to-float and float-to-float conversion using `toTargetType` follow D76's IEEE 754 conversion rules for the target format. They do not TPOE solely because precision is lost during ordinary rounding to the target float format.
  7. Non-default conversion policies remain named APIs in the same family. At minimum, the language/library surface provides distinct named forms for fallible conversion (`tryToTargetType` style), truncating/wrapping conversion (`truncateToTargetType` style), and saturating conversion when the standard library chooses to provide it.
  8. There are no implicit numeric conversions.
  9. Rust-style `x as TargetType`, C-style `(TargetType)x`, and pseudo-constructor conversion syntax such as `TargetType(expr)` are not part of Kyokai.
**Why this fits Kyokai**: the default conversion stays short, but the spelling is visibly a conversion name rather than a pseudo-constructor or generic cast form. It aligns with UFCS, keeps alternative policies explicit, and avoids collapsing every conversion into one ambiguous operator.
**[STAGE: DECIDED_CORE_SEMANTICS | D37 → UFCS-style `x.toTargetType()` for default numeric conversion; named APIs for non-default policies]**
- **`Index` is a distinct concrete integer type, and cross-integer mixing remains explicit even at indexing and length boundaries** — Kyokai closes the last hole left by D75's fixed-width focus by stating exactly how `Index` interacts with the rest of the integer family.
**Rules**:
  1. `Index` is its own concrete integer type. It is not interchangeable with any `Int*` or `Nat*` type merely because a target may represent them with the same machine width.
  2. Built-in arithmetic and comparison over integer-like types, including `Index`, are legal only when both operands have the same concrete type after D12 literal inference and any explicit D37 conversions.
  3. `Int8 + Int32`, `Nat32 < Index`, and assignment or binding of an `Int*`/`Nat*` value to an `Index`-typed place without an explicit conversion are compile-time errors.
  4. Default explicit conversions between `Index` and the fixed-width integer families use D37's built-in conversion family such as `x.toIndex()`, `x.toNat64()`, or `x.toInt32()`.
  5. `toIndex()` succeeds only when the source numeric value is representable in `Index`; otherwise it triggers TPOE under D37's checked-conversion rule.
  6. Integer literals may still infer to `Index` under D12 when the expected type is known, so code such as `buf.length() + 1` remains legal when the literal has exactly one valid inferred type.
  7. Standard indexing, length, capacity, and other shape-bearing APIs use `Index` explicitly. Kyokai defines no ambient rule that "any integer-like type may be used as an index."
  **Why this fits Kyokai**: array sizes, lengths, and indexing stay honest about using a dedicated machine-sized domain, while arithmetic never quietly crosses nominal numeric boundaries behind the programmer's back.
  **[STAGE: DECIDED_CORE_SEMANTICS | D210 → `Index` is distinct; integer arithmetic/comparison require same concrete type including `Index`; cross-family use requires explicit D37 conversion]**
- `**String` means well-formed UTF-8 text, not “bytes by convention”** — Kyokai gives text a real language-level contract instead of treating human text and arbitrary bytes as the same thing.
**Rules**:
  1. `String` is an owned text type whose contents are always well-formed UTF-8.
  2. This UTF-8 validity guarantee applies on every construction path from raw bytes.
  3. `String` does not imply Unicode normalization. The language does not silently normalize to NFC, NFD, or any other normalization form.
  4. Text operations on `String` are defined against this UTF-8 contract; they do not treat `String` as an arbitrary byte buffer.
  5. The standard Unicode-oriented helpers use explicit names such as `codePointAt`, `codePointLen`, and `byteLen`; Kyokai does not use vague names like `charLen` for code-point-count APIs.
  6. Locale-sensitive behavior is not implicit in `String` operations.
  **Why this fits Kyokai**: if a value has type `String`, both the programmer and the compiler know it is valid text. Kyokai keeps text semantics explicit instead of inheriting the C/Go habit where “string” often really means “some bytes.”
  **[STAGE: DECIDED_CORE_SEMANTICS | D30 → `String` is guaranteed well-formed UTF-8 text; no implicit normalization or locale magic]**
- **Text and raw bytes are separate types with named, explicit bridges** — Kyokai does not collapse UTF-8 text, binary payloads, borrowed byte views, and fixed-size byte arrays into one universal “string” concept.
**Core split**:
  ```kyokai
  String          // owned UTF-8 text
  ByteBuf         // owned arbitrary bytes
  Span[Nat8]      // borrowed bytes
  Array[Nat8, N]  // fixed-size raw bytes
  ```
  **Conversions**:
  ```kyokai
  let text: Result[String, Utf8Error] := String.fromUtf8(rawBytes);
  let raw: ByteBuf := text.intoBytes();
  let bytes: Span[Nat8] := text.asBytes();
  ```
  **Rules**:
  1. Raw bytes live in `ByteBuf`, `Span[Nat8]`, `Array[Nat8, N]`, or future explicitly byte-oriented types, not in `String`.
  2. There is no implicit conversion between text and bytes.
  3. Any conversion from bytes to `String` that establishes UTF-8 validity is named and explicit.
  4. Any conversion from `String` to a borrowed byte view is named and explicit and must not silently allocate.
  5. Any conversion from `String` to an owned byte buffer is named and explicit and preserves the underlying UTF-8 byte sequence exactly.
  6. OS-facing and foreign-facing string boundaries are not silently treated as `String`; environment variables, C strings, paths, and similar native-text surfaces must define their own explicit conversion rules in their own decisions.
  **Why this fits Kyokai**: text and bytes are genuinely different concepts in systems code. Kyokai gives each concept its own type and makes every bridge between them visible.
  **[STAGE: DECIDED_CORE_SEMANTICS | D30a → explicit split between UTF-8 text and raw bytes; no implicit conversions]**
- **`String` reuses the owning byte-buffer storage discipline, but remains a distinct nominal UTF-8 text type** — Kyokai does not duplicate dynamic-container machinery for text, and it does not blur text into raw bytes just because the storage strategy is shared.
**Rules**:
  1. `String` is a `Linear`, owning text type.
  2. `String` is nominally distinct from raw byte containers and views. It is not an alias and not a transparent substitution for raw bytes in type checking.
  3. `String` uses the same storage discipline as Kyokai's owning contiguous byte-buffer containers: owned heap storage, explicit byte length, explicit capacity, and stored allocator identity under D44.
  4. The UTF-8 invariant from D30 applies to every live `String` value.
  5. Safe operations on `String` must preserve that UTF-8 invariant.
  6. Safe code may obtain an immutable borrowed byte view of a `String` through an explicit named operation such as `asBytes()`, but safe code may not obtain mutable raw-byte access that could violate the UTF-8 invariant.
  7. `StringBuilder` is the standard mutable construction path for incrementally built text; finishing a builder yields a `String`.
  8. Sharing the storage discipline with byte buffers does not automatically make all byte-buffer APIs part of the `String` surface.
  **Why this fits Kyokai**: the runtime only needs one owning contiguous-buffer model, but the type system still keeps "valid text" and "arbitrary bytes" separate.
  **[STAGE: DECIDED_CORE_SEMANTICS | D165 → `String` is a nominal `Linear` UTF-8 text type that reuses the owning byte-buffer storage discipline without collapsing into raw bytes]**
- **`String` has one exact runtime state model and no alternate safe representation class** — Kyokai fully fixes what an owning string stores instead of leaving representation folklore to the implementation.
**Rules**:
  1. The concrete runtime state of a `String` consists of exactly four logical components: data pointer, byte length, byte capacity, and allocator identity.
  2. `String` length counts bytes, not Unicode scalar values, grapheme clusters, or display columns.
  3. `String` has no small-string optimization, tagged inline variant, or other alternate safe-language representation class.
  4. Borrowing a `String` does not expose mutable access to those underlying bytes in safe code.
  5. The exact runtime state fixed here does not erase the nominal distinction from raw byte containers; `String` remains its own type even though the stored state is the same kind of owning buffer state.
  **Why this fits Kyokai**: the language gets an exact owning-text representation contract, while all text-vs-bytes semantics remain explicit at the type level.
  **[STAGE: DECIDED_CORE_SEMANTICS | D204 → exact `String` runtime state is pointer + byte length + byte capacity + allocator identity; no SSO or alternate safe representation]**
- **C NUL-terminated strings use dedicated interop types instead of “just use `String`” folklore** — Kyokai treats C strings as explicit FFI-boundary byte contracts with their own invariants rather than as ordinary text or naked pointers by convention.
**Core types**:
  ```kyokai
  CString  // owned C-string value
  CStr     // borrowed C-string view
  ```
  **Rules**:
  1. `CString` is an owned byte sequence that ends with exactly one trailing NUL byte and contains no interior NUL bytes.
  2. `CStr` is a borrowed view of a valid NUL-terminated byte sequence. It does not own storage and does not imply UTF-8 validity, Unicode semantics, or locale semantics.
  3. `CString` and `CStr` are distinct from `String`, `ByteBuf`, and `Span[Nat8]`. There is no implicit conversion between any of these types.
  4. Creating a `CString` from text or bytes is explicit and validating. The operation must reject interior NUL bytes, establish the trailing NUL byte, and obey D44's explicit allocator rules for owned allocation.
  5. Borrowing a `CStr` from a `CString` is explicit and allocation-free.
  6. Constructing a `CStr` directly from a raw foreign pointer is only legal at the unsafe boundary and only under an explicit contract that the pointed-to storage is alive, readable, and properly NUL-terminated.
  7. Raw `foreign "C"` declarations continue to spell C-string parameters with pointer types such as `Address[Nat8]` or `Pointer[Nat8]` under D20. `CString` and `CStr` are wrapper-side interop types that make the boundary explicit and validated.
  8. Safe wrapper APIs should prefer `&[CStr]` or another explicitly chosen C-string wrapper type over naked byte pointers when the contract is “valid NUL-terminated string”.
  **Why this fits Kyokai**: text stays text, bytes stay bytes, and the C-string boundary becomes an explicit validated contract instead of an ambient pointer convention inherited from C.
  **[STAGE: DECIDED_CORE_SEMANTICS | D68 → dedicated `CString` and `CStr` interop types; no implicit conversion from text/bytes; raw-pointer entry only at the unsafe boundary]**
- **Iteration is split cleanly between range syntax and a minimal iterable protocol** — Kyokai supports both numeric loops and collection loops, but does not import Rust's closure-heavy iterator ecosystem.
**Range syntax**:
  1. `for i from a to b do ... od;` is inclusive and iterates `i = a, a + 1, ..., b`.
  2. `for i from a below b do ... od;` is exclusive and iterates `i = a, a + 1, ..., b - 1`.
  3. `below` is the preferred form for 0-based indexing and length-bounded loops; Kyokai does not force `to (n - 1)` boilerplate for half-open ranges.
  **Iterable protocol**:
  ```kyokai
  typeclass Iterator(Iter: Type) is
      type Item;
      method next(iter: &![Iter]): Optional[Iter.Item];
  spec;

  typeclass Iterable(Self: Type) is
      type Iter;
      method iter(self: &[Self]): Self.Iter;
  spec;
  ```
  **Rules**:
  1. `for item in expr do ... od;` evaluates `expr` exactly once.
  2. If the resulting type implements `Iterator`, the loop uses that value directly as the iteration state.
  3. Otherwise, the resulting type must implement `Iterable`, and the loop creates an iterator by calling `iter`.
  4. If a type implements both `Iterator` and `Iterable`, the direct-`Iterator` path wins.
  5. In the direct-`Iterator` path, the loop consumes the iterator value into its private loop state. In the `Iterable` path, the iterator returned by `iter` is the private loop state.
  6. Iterators used by Kyokai's standard `for-in` protocol are fused by default: once `next` returns `None`, it must keep returning `None`.
  7. The base `for-in` form is for borrowing iteration over collections and direct iteration over iterator values. Mutable iteration and consuming iteration over collection-like types still use explicit named forms such as `iterMut()` and `intoIter()` that return iterator values implementing `Iterator`.
  8. If the compiler-generated loop state for `for-in` has a `Linear` type, that iterator type must implement `Destroyable` or have a feature-specified generated consuming `destroy` operation such as D198 generators.
  9. The `for-in` desugaring consumes a linear iterator exactly once on every loop exit path: normal exhaustion, `break`, `return`, `or return`, `panic` cleanup, and any other control-flow path that exits the loop scope.
  10. A linear `item` yielded into the loop body must be consumed by the ordinary linearity rules before the next iteration begins or before any exit from the loop body.
  11. The core iteration contract stops at `iter`, `next`, direct-iterator acceptance, linear-iterator finalization, and `for-in` desugaring. Iterator adapter APIs such as `map`, `filter`, `fold`, `collect`, `zip`, and `enumerate` are not part of D32; their existence, ownership model, allocation behavior, and callback surface are a separate decision point (D32a).
  **`for-in` desugaring**:
  ```kyokai
  // Iterable source:
  for item in collection do
      process(item);
  od;

  // Desugars to:
  var __iter := collection.iter();
  while true do
      case __iter.next() of
          when Some(item) do
              process(item);
          when None do
              break;
      esac;
  od;
  ```
  ```kyokai
  // Direct iterator source:
  for item in makeCountdown(3) do
      process(item);
  od;

  // Desugars to:
  var __iter := makeCountdown(3);
  while true do
      case __iter.next() of
          when Some(item) do
              process(item);
          when None do
              break;
      esac;
  od;
  ```
  **Why this fits Kyokai**: the language gets ergonomic `for-in` loops and user-extensible iteration without importing a giant hidden-adapter ecosystem, forcing range users to write `n - 1` everywhere, or leaking linear iterator state on early exit.
  **[STAGE: DECIDED_CORE_SEMANTICS | D32/D249 → dual range forms (`to` and `below`) + minimal `Iterable`/`Iterator` protocol, fused by default; linear iterators are consumed by the loop desugaring on every exit path]**
- **Records use brace-plus-colon construction, not call syntax** — Kyokai makes data construction visibly distinct from calls so that record literals remain obvious even with UFCS and named factory functions in the language.
**Syntax**:
  ```kyokai
  let p: Point := Point { x: 5, y: 10 };
  let r: Resource := Resource { id: 1, active: true };
  let q: Point := Point { x, y };
  ```
  **Rules**:
  1. Record construction is always written as `TypeName { field: expr, ... }`.
  2. Positional record construction does not exist.
  3. Every declared field must appear exactly once in a record construction expression. Duplicate fields are illegal.
  4. There is no partial construction, omitted-default filling, or hidden field initialization.
  5. Field order is semantically irrelevant, but field expressions are evaluated in source order per D71.
  6. Shorthand `TypeName { field }` is allowed only when a binding with the same name is already in scope, and it desugars exactly to `TypeName { field: field }`.
  7. Record construction syntax is distinct from function-call syntax. Parentheses remain for calls; braces remain for data construction.
  8. Anonymous record spread syntax and hidden default-filling are not part of Kyokai. The only standardized record-update form is D138's explicit `TypeName { ..., with source }`.
  **Why this fits Kyokai**: parens mean calls, braces mean construction, and the language does not blur data literals with functions or hidden default-filling behavior.
  **[STAGE: DECIDED_CORE_SEMANTICS | D35 → brace-plus-colon record construction `TypeName { field: expr }`; no positional or partial construction]**
- **Records also have an explicit type-led update form; omitted fields come only from a named source record, never from hidden defaults** — Kyokai reduces schema-change churn without introducing anonymous structural-update syntax or a second unrelated record grammar.
**Syntax**:
  ```kyokai
  let next: Point := Point { z: 0, with oldPoint };
  let moved: Job := Job { state: Done, with oldJob };
  ```
  **Rules**:
  1. The update form is `TypeName { field1: expr1, ..., with source }`.
  2. `with source` may appear at most once and must be the final item inside the braces.
  3. `source` must have exactly the same record type as `TypeName`.
  4. Fields written explicitly override the matching fields from `source`.
  5. After applying the explicit overrides, every field of the record must be supplied exactly once. Missing fields and duplicate fields are compile-time errors.
  6. For a `Free` record type, remaining fields are copied from `source` and `source` remains usable.
  7. For a `Linear` record type, `source` is consumed and every remaining field is moved from `source` into the new record.
  8. Kyokai does not add a separate `old with { ... }` or anonymous structural-update form. Record update remains visibly a `TypeName { ... }` construction.
  **Why this fits Kyokai**: the type stays visible at the construction site, move/copy behavior remains explicit, and omitted fields are never magical because the programmer names the exact source record supplying them.
  **[STAGE: DECIDED_CORE_SEMANTICS | D138 → explicit type-led record update `TypeName { ..., with source }`; `Free` copies remaining fields, `Linear` consumes and moves from source]**
- **Single-field records are Kyokai's nominal wrapper mechanism, and Kyokai adds a one-line declaration form for them instead of a separate `newtype` feature** — domain wrappers stay ordinary records rather than splitting into a second nominal-type construct.
**Syntax**:
  ```kyokai
  record UserId(value: Int64): Free;
  ```
  is exact sugar for:
  ```kyokai
  record UserId: Free is
      value: Int64;
  build;
  ```
  **Rules**:
  1. Kyokai has no dedicated `newtype` keyword or separate newtype declaration category.
  2. A one-line record declaration form is allowed for records with exactly one field.
  3. The one-line form preserves the same name, generics, universe annotation, and field type as the corresponding ordinary record declaration.
  4. Construction still uses ordinary record construction syntax: `UserId { value: 42 }`.
  5. D65's single-field paren form remains union-only and does not apply to records.
  6. D196 defines the precise transparent-wrapper representation guarantee for ordinary single-field records.
  **Why this fits Kyokai**: there is one nominal-wrapper mechanism, not two, and the only extra sugar added is declaration-site compression for the common single-field case.
  **[STAGE: DECIDED_CORE_SEMANTICS | D109 → no dedicated `newtype`; single-field records remain the wrapper mechanism; one-line single-field record declaration added]**
- **Ordinary single-field records have transparent wrapper representation inside Kyokai** — Kyokai's nominal wrappers are zero-cost in the language's own layout and calling model, but the guarantee is stated narrowly enough not to conflict with `extern record` and `packed record`.
**Rules**:
  1. This guarantee applies only to ordinary `record` declarations with exactly one field.
  2. For such a record `Wrapper`, `sizeOf(Wrapper)` equals `sizeOf(FieldType)`, `alignOf(Wrapper)` equals `alignOf(FieldType)`, and the sole field has offset `0`.
  3. Passing, returning, storing, and loading such a wrapper in Kyokai's own call/return and ordinary layout model must behave as though the wrapper had the same representation as its field type, with no hidden tag, padding-only wrapper shell, or extra indirection.
  4. The wrapper remains a distinct nominal type under D190. Representation equality does not create type equality, implicit conversion, or instance sharing.
  5. This guarantee does not apply to `packed record`, whose layout is governed by D42's packed rules.
  6. This guarantee does not apply to `extern record`, whose FFI layout and ABI are governed by D20a and D42.
  **Why this fits Kyokai**: single-field records stay the one wrapper mechanism, and they are guaranteed zero-cost without pretending foreign C structs or packed layouts obey the same rule.
  **[STAGE: DECIDED_CORE_SEMANTICS | D196 → ordinary single-field `record` wrappers are transparent in Kyokai layout/call ABI; still nominally distinct; excludes `extern record` and `packed record`]**
- **Record destructuring is total: destructuring a record consumes the whole record and binds its fields as independent values** — Kyokai does not adopt Rust-style partial-move record semantics.
**Rules**:
  1. Any record destructuring form (`let { ... } := record;` or a `case` arm that destructures a record) consumes the record value as a whole.
  2. Destructuring binds each mentioned field as a separate local binding; from that point on, the original record value no longer exists.
  3. There is no partial record destructuring that removes one field while leaving the original record alive.
  4. Each bound field keeps its own universe and linearity. Linear fields must still be consumed exactly once; Free fields may be ignored.
  5. Omitting a field or binding it to `ignore` is legal only when that field is `Free`. A linear field must be bound to a real name and explicitly consumed or explicitly destroyed through the ordinary APIs.
  6. If only one field is needed, the programmer still destructures the whole record and handles the remaining linear fields explicitly, often with `defer`.
  **Why this fits Kyokai**: the rule matches Austral's simple "destructure = consume" model, keeps linear resource flow visible, and avoids the extra complexity of partial-move state tracking.
  **[STAGE: DECIDED_CORE_SEMANTICS | D98 → total record destructuring; no partial moves; linear fields remain explicit]**
- **Field access through references uses `.` with one level of auto-deref** — Kyokai keeps member access uniform without importing unlimited deref chains or a separate `->` operator.
**Rules**:
  1. If `r` has type `&[Record]` or `&![Record]`, `r.field` is legal and means "dereference one reference level, then access `field`."
  2. This auto-deref applies to direct field access and field assignment places only. It does not add hidden method dispatch, typeclass lookup, or unlimited deref search.
  3. The auto-deref depth is exactly one level. If `rr` is a reference to a reference, `rr.field` is illegal; the programmer must write `(~rr).field` or another explicitly dereferenced form.
  4. `~` remains the explicit dereference operator for cases where the programmer wants to show the dereference directly.
  5. `->` is not part of Kyokai.
  **Why this fits Kyokai**: `.` remains the single member-access operator, field access through a reference is still mechanically obvious, and the language avoids Rust-style implicit deref chains.
  **[STAGE: DECIDED_CORE_SEMANTICS | D34 → one-level auto-deref for field access through `.`; no `->`]**
- **Pattern matching has no guard clauses** — Kyokai keeps `case` purely structural and leaves boolean filtering to `if`.
**Rules**:
  1. A `when` arm is written as `when Pattern do ...`; there is no `when Pattern if condition do` form.
  2. Conditions that refine a matched pattern are written with ordinary `if` / `else` inside the selected `when` body.
  3. Exhaustiveness checking is structural only. Guard conditions do not participate in the language's exhaustiveness model because guard syntax does not exist.
  **Why this fits Kyokai**: `case` destructures and `if` filters. The language keeps those concerns separate instead of weakening exhaustiveness with guard logic.
  **[STAGE: DECIDED_CORE_SEMANTICS | D38 → no pattern guards; use `if` inside `when`]**
- **`case` matching is exhaustive, supports nested structural patterns, and uses `ignore` as the discard pattern** — Kyokai does not restrict matching to one constructor layer, and it does not use `_` as a pattern token.
**Rules**:
  1. Every `case ... of ... esac;` must be exhaustive over the scrutinee type. A non-exhaustive `case` is a compile-time error.
  2. Patterns may nest through union constructors, record destructuring, and combinations of those forms to arbitrary finite depth.
  3. `case` arms are tested in source order. The first arm whose pattern matches is the arm that executes.
  4. `ignore` is the contextual discard pattern. It matches any value at that pattern position and introduces no binding.
  5. `_` is not a pattern token in Kyokai.
  6. Exhaustiveness and unreachable-arm checking are defined over these nested structural patterns; nesting does not disable compile-time coverage checking.
  **Why this fits Kyokai**: nested `Result`/`Optional` composition stays directly matchable, exhaustiveness remains mandatory, and the language avoids introducing symbolic wildcard punctuation for a single discard concept.
  **[STAGE: DECIDED_CORE_SEMANTICS | D205 → exhaustive nested structural patterns; `ignore` is the discard pattern; `_` is not a pattern token]**
- **`ignore` and omitted subpatterns are legal only at `Free` pattern positions** — Kyokai does not let discard syntax hide linear-resource destruction.
**Rules**:
  1. `ignore` may appear only at a pattern position whose matched value is `Free`.
  2. In record destructuring, omitting a field is exact sugar for matching that field with `ignore`; it is therefore legal only for `Free` fields.
  3. A catch-all arm written as `when ignore do ...` is illegal if any still-unmatched value that could reach that arm may contain a `Linear` payload.
  4. When a matched variant or field may contain a `Linear` value, the pattern must bind that value to an explicit name, and the selected arm must satisfy ordinary linearity by consuming it explicitly.
  **Why this fits Kyokai**: catch-all syntax stays available for ordinary `Free` data, while linear resources remain visible in the surface program instead of being silently discarded by pattern syntax.
  **[STAGE: DECIDED_CORE_SEMANTICS | D206 → `ignore` and omitted subpatterns are `Free`-only; linear payloads must bind explicit names]**
- `**while let` is explicit sugar for "re-evaluate, pattern-match, continue on match, break on mismatch"** — Kyokai admits this loop form because the desugaring is mechanical and the pattern is common in explicit, closure-free code.
**Syntax**:
  ```kyokai
  while let Some(item) := queue.dequeue() do
      process(item);
  od;
  ```
  **Rules**:
  1. `while let Pattern := expr do ... od;` re-evaluates `expr` once at the start of each iteration.
  2. If the value of `expr` matches `Pattern`, the pattern bindings enter scope for the loop body and that iteration runs.
  3. If the value of `expr` does not match `Pattern`, the loop terminates immediately.
  4. `Pattern` must be refutable. Irrefutable patterns are illegal in `while let`.
  5. Bound names from `Pattern` are scoped only to the loop body of the successful iteration.
  6. `while let` is general pattern sugar; it is not restricted to `Optional` or `Result`.
  **Desugaring**:
  ```kyokai
  // Source:
  while let Pattern := expr do
      body;
  od;

  // Desugars to:
  while true do
      case expr of
          when Pattern do
              body;
          when ignore do
              break;
      esac;
  od;
  ```
  **Why this fits Kyokai**: the syntax removes a common seven-line manual loop pattern without adding hidden semantics or special Optional-only behavior.
  **[STAGE: DECIDED_CORE_SEMANTICS | D39 → general refutable-pattern `while let` sugar with exact desugaring]**
- **Grammar whitespace is insignificant, trailing commas are widely permitted, and empty executable blocks are legal no-op blocks; formatting tools may not rewrite semantics** — Kyokai keeps the grammar permissive while leaving style enforcement to `kyokai fmt` and diagnostics.
**Rules**:
  1. Newlines are ordinary insignificant whitespace except where a token would otherwise be split illegally.
  2. Trailing commas are legal in all comma-delimited list forms of the language grammar.
  3. Empty executable blocks are legal and mean ordinary no-op execution of that block.
  4. `kyokai fmt` may enforce canonical formatting choices such as multiline trailing commas, but it may not change program semantics by inserting or removing executable statements.
  5. A lint or warning may diagnose suspicious empty blocks, but the grammar and runtime meaning remain as stated by rules 1 through 4.
  **Why this fits Kyokai**: parsing stays simple, source formatting stays deterministic, and tooling does not become a hidden semantic rewrite layer.
  **[STAGE: DECIDED_CORE_SEMANTICS | D180 → insignificant newlines; trailing commas allowed in all comma-delimited lists; empty executable blocks are legal no-op blocks; `fmt` may not rewrite semantics]**
- **Bitwise and bit-shift operations use explicit keyword operators** — Kyokai keeps bit manipulation readable without overloading `&` or relying on symbolic precedence folklore.
**Operators**:
  - `a band b`
  - `a bor b`
  - `a bxor b`
  - `bnot a`
  - `a shl n`
  - `a shr n`
  - `a rotl n`
  - `a rotr n`
  **Rules**:
  1. These operators are built-in for the concrete integer types only. They are not user-overloadable.
  2. `band`, `bor`, and `bxor` require both operands to have the same concrete integer type and return that same type.
  3. `bnot` takes one concrete integer operand and returns that same concrete integer type.
  4. `shl` and `shr` require a concrete integer left operand and an integer shift count. Negative counts trigger TPOE. Counts greater than or equal to the operand bit width trigger TPOE.
  5. `shl` shifts left within the operand's fixed bit width, zero-filling low bits and discarding high bits as part of the defined bitwise result.
  6. `shr` is a logical right shift for `Nat*` types and an arithmetic right shift for `Int*` types.
  7. `rotl` and `rotr` rotate within the operand's fixed bit width. Negative counts trigger TPOE. Non-negative counts are reduced modulo the bit width.
  8. There are no implicit promotions or hidden count conversions in these operators.
  9. Precedence and parenthesization for these operators are governed by D56's limited precedence rules.
  **Why this fits Kyokai**: the operators are readable, searchable, unambiguous with borrow syntax, and explicit about shift-count failure and rotation semantics.
  **[STAGE: DECIDED_CORE_SEMANTICS | D41 → keyword bitwise operators plus `rotl`/`rotr`; explicit shift-count semantics]**
- **Record layout is a closed, explicit type-level choice: `record`, `extern record`, and `packed record`** — Kyokai does not let the backend silently choose ordinary record layout, and it does not define the language's normal layout by appealing to the C backend. Ordinary records use a language-defined Kyokai layout. `extern record` is the explicit foreign-ABI form. `packed record` is the explicit byte-tight form. These are the only layout classes in the language; Kyokai does not adopt an open-ended `repr(...)` attribute system.
**Syntax**:
  ```kyokai
  record Point is
      x: Int32;
      y: Int32;
  build

  extern record Stat is
      size: Nat64;
      mode: Nat32;
  build

  packed record Header is
      tag: Nat8;
      length: Nat16;
  build
  ```
  **`record` (Kyokai layout)**:
  1. Fields remain in source order. The compiler never reorders fields.
  2. Each field starts at the smallest offset greater than or equal to the end of the previous field that satisfies that field type's alignment.
  3. Record alignment is the maximum alignment of its fields.
  4. Record size is rounded up to a multiple of the record alignment.
  5. User-specified over-alignment, under-alignment, bitfields, and hidden niche/layout optimizations are not part of the language.
  **`extern record` (target C ABI layout)**:
  1. `extern record` uses the selected target's C ABI layout rules for that field list and field order.
  2. Field order remains the source order written by the programmer; Kyokai does not reorder `extern record` fields.
  3. A record type may appear in a `foreign "C"` declaration only if it is an `extern record`.
  4. Every field of an `extern record` must itself be FFI-safe under D20/D20a. Otherwise the declaration is illegal.
  **`packed record` (byte-tight layout)**:
  1. Fields remain in source order with no implicit padding between fields.
  2. Packed record alignment is 1.
  3. Packed record size is the sum of field sizes.
  4. Reading or writing a packed field uses copy semantics, not reference semantics: the implementation must behave as though it copies bytes into or out of a properly aligned temporary of the field type.
  5. Taking `&field` or `&!field` of a packed field is illegal. Packed fields cannot produce borrows because that would create potentially misaligned references.
  6. `packed` does not imply bitfields, byte swapping, endianness conversion, or C ABI compatibility.
  **Mutual exclusion**:
  1. A record is exactly one of `record`, `extern record`, or `packed record`.
  2. `extern packed record` and any equivalent combination are illegal.
  3. If a foreign API requires a packed C struct, the boundary must use an explicit unsafe wrapper that marshals bytes or raw storage deliberately; Kyokai does not pretend that case is ordinary.
  **Layout introspection**:
  1. `sizeOf(T)`, `alignOf(T)`, and `offsetOf(T, field)` are comptime-only built-ins.
  2. They are written with the same visible phase marker as other compile-time evaluation:
  ```kyokai
  constant HEADER_SIZE: Index := comptime sizeOf(Header);
  constant HEADER_ALIGN: Index := comptime alignOf(Header);
  constant LENGTH_OFFSET: Index := comptime offsetOf(Header, length);
  ```
  1. These built-ins use the semantics of the record's declared layout class (`record`, `extern record`, or `packed record`), not backend guesses.
  **Why this fits Kyokai**: ordinary language layout is specified by the language, foreign layout is explicit at the type declaration, and packed layout is available without relying on backend folklore or a pile of ad hoc repr flags. The reader can see the layout contract directly from the type declaration.
  **[STAGE: DECIDED_CORE_SEMANTICS | D42 → closed record-layout system: language-defined `record`, explicit `extern record`, explicit `packed record`]**
- `**break` never carries a value and loops are not expressions** — Kyokai keeps loop control separate from value production.
**Rules**:
  1. `break` takes no operand.
  2. `for`, `while`, and `while let` are statements, not expressions, and do not produce values.
  3. To get a value out of loop control, code must use an explicit outer binding, `Optional`, `Result`, or another ordinary data structure rather than a loop-expression result.
  **Why this fits Kyokai**: the control-flow model stays uniform, there is no Rust-style special loop-expression case, and the source of an output value remains explicit in ordinary bindings.
  **[STAGE: DECIDED_CORE_SEMANTICS | D43 → no break-with-value; loops remain statements]**
- **Search ergonomics are solved with ordinary `Kyokai.Iter` helpers, not loop-expression results or dedicated search syntax** — Kyokai keeps D43's loop model intact and resolves the common "find the first matching element" pattern with library functions over the already-decided iterator and closure substrate.
**Rules**:
  1. D43 stands unchanged: loops remain statements, and `break` never carries a value.
  2. `Kyokai.Iter` provides ordinary search helpers such as `find` and `findIndex` in addition to D32a's other eager helpers and reductions.
  3. These helpers use the D21/D126 callback family and become lightweight in ordinary code through D118 explicit-capture closure literals.
  4. Kyokai does not add dedicated `find item in collection where ...` syntax.
  **Why this fits Kyokai**: the common search pattern becomes short without turning loops into a partial expression system or adding one-off syntax for a problem that the library surface already solves cleanly.
  **[STAGE: DECIDED_CORE_SEMANTICS | D128 → keep D43; provide ordinary `Kyokai.Iter` search helpers instead of break-with-value or dedicated search syntax]**
- **Loop labels are lexical loop names, and `break`/`continue` may target them explicitly** — nested-loop exits stay visible without inventing loop expressions or state-flag boilerplate.
**Syntax**:
  ```kyokai
  outer: for i from 0 below rows do
      inner: for j from 0 below cols do
          if shouldStop(i, j) then
              break outer;
          fi;
          if shouldSkipRow(i) then
              continue outer;
          fi;
      od;
  od;
  ```
  **Rules**:
  1. A loop statement may be prefixed with a label as `label: for ...` or `label: while ...`.
  2. Unlabeled `break;` and `continue;` still apply to the innermost enclosing loop.
  3. `break label;` and `continue label;` target the lexically named enclosing loop.
  4. The target must name an enclosing loop label in the current function body. Targeting a non-enclosing label or a non-loop construct is illegal.
  5. Two loop labels whose scopes overlap may not reuse the same label name.
  6. D43 remains unchanged: labeled `break` still carries no value and does not make loops expressions.
  **Why this fits Kyokai**: the control-flow jump is explicit in source, compiles to ordinary structured jumps, and avoids both hidden flags and alternate `break` spellings that attach punctuation semantics to the keyword itself.
  **[STAGE: DECIDED_CORE_SEMANTICS | D122 → lexical loop labels with `break label;` / `continue label;`]**
- **Kyokai has no tuple syntax or tuple types** — multi-value structure must be named explicitly, and positional helper records do not become language tuples by stealth.
**Rules**:
  1. The language has no tuple types, tuple literals, tuple destructuring, or positional multi-return syntax.
  2. Parentheses group expressions and delimit call arguments; they do not create tuple values.
  3. Functions that conceptually return multiple values must return a named record or another named type.
  4. Libraries may provide ordinary named record types such as `Pair` or `Triple`, but those are regular records, not special language tuples and not language sugar.
  5. Public APIs should prefer domain-named records whenever field roles matter. Positional helper records are for generic plumbing, not the default API style.
  6. The standard channel constructors follow this rule and return `ChannelEndpoints[T]` with named `sender` and `receiver` fields rather than `Pair[Sender[T], Receiver[T]]`.
  **Why this fits Kyokai**: multi-value data remains self-documenting instead of hiding meaning behind positions, while still allowing plain library records where the domain meaning really is just "first/second/third."
  **[DECIDED: D47/D131 → no tuples; `Pair`/`Triple` remain ordinary records, but public APIs prefer domain-named result types]**
- **Text, code-point, byte, and raw-string literals are distinct** — Kyokai gives each literal family one type and one clearly bounded meaning.
**Literal forms**:
  - `"..."` -> `String`
  - `"""..."""` -> raw multi-line `String`
  - `'A'` -> `Nat32` Unicode code point literal
  - `b'A'` -> `Nat8` byte literal
  **Rules**:
  1. Ordinary string literals produce `String` and process escapes.
  2. The standard escape family is `\\`, `\"`, `\'`, `\n`, `\r`, `\t`, `\0`, `\xNN`, and `\u{HEX...}`.
  3. Raw multi-line string literals process no escapes and preserve their contents exactly between the delimiters.
  4. A code-point literal must denote exactly one Unicode scalar value after escape processing. Surrogate code points are illegal.
  5. A byte literal must denote exactly one byte value after escape processing. Bare source characters in `b'...'` form must be ASCII; non-ASCII byte values require escapes such as `b'\xFF'`.
  6. There is no ambiguous C-style `char` literal family that changes meaning by platform or signedness.
  **Why this fits Kyokai**: systems code gets both raw bytes and Unicode text without inheriting C's `char` confusion or hidden encoding assumptions.
  **[STAGE: DECIDED_CORE_SEMANTICS | D54 → explicit string, raw-string, code-point, and byte literal families]**
- **Array literals use `[...]`, infer only their length, and stay explicit about element typing** — Kyokai treats array-literal size inference as counting, not as magical type deduction.
**Syntax**:
  ```kyokai
  let primes: Array[Int32, 3] := [2, 3, 5];
  let empty: Array[Nat8, 0] := [];
  ```
  **Rules**:
  1. `[e1, e2, ..., en]` constructs an `Array[T, N]` literal with `N` equal to the number of elements written.
  2. Array-literal elements are evaluated in source order per D71.
  3. The element type must resolve to one concrete type under the ordinary type rules and literal-inference rules; ambiguity is a compile-time error.
  4. There are no implicit promotions inside array literals.
  5. Bare `[]` is legal only when context already fixes the target type to `Array[T, 0]`; otherwise it is illegal because the element type is not known.
  6. Repeat syntax, spread syntax, and other array-construction shorthands are not part of Kyokai.
  **Why this fits Kyokai**: the length inference is purely mechanical, but the element type remains explicit or locally forced by ordinary typing rules.
  **[STAGE: DECIDED_CORE_SEMANTICS | D55 → `[e1, ...]` array literals with length inference only; empty literal needs `Array[T, 0]` context]**
- **Kyokai uses a small explicit precedence table instead of either full Austral-style parenthesization or a large C-family precedence ladder** — arithmetic stays readable while bitwise and mixed-operator code stays explicit.
**Precedence levels**:
  1. Postfix: field access `.`, call `(...)`, and indexing `[...]`
  2. Prefix: `&`, `&!`, `~`, unary `-`, `not`, `bnot`
  3. Multiplicative: `*`, `/`, `%`
  4. Additive and concatenation: `+`, binary `-`, `++`
  5. Comparison and equality: `<`, `<=`, `>`, `>=`, `=`, `!=`
  6. Boolean `and`
  7. Boolean `or`
  **Rules**:
  1. Operators at the same precedence level associate left-to-right unless another rule explicitly says otherwise.
  2. Comparison and equality operators do not chain. `a < b < c` and `a = b = c` are illegal without explicit restructuring.
  3. Bitwise and rotate operators (`band`, `bor`, `bxor`, `shl`, `shr`, `rotl`, `rotr`) do not mix implicitly with arithmetic, comparison, or boolean operators. Parentheses are required whenever they are nested with those other binary families.
  4. Different bitwise/shift/rotate operator names do not mix implicitly with one another either. `a band b bor c` and `a shl n shr m` require parentheses.
  5. Same-operator chaining such as `a + b + c`, `a * b * c`, or `a band b band c` is allowed and associates left-to-right.
  **Why this fits Kyokai**: ordinary numeric code does not drown in parentheses, but the operators most likely to cause precedence bugs still require the programmer to show grouping explicitly.
  **[STAGE: DECIDED_CORE_SEMANTICS | D56 → limited explicit precedence; bitwise/shift/rotate mixes require parentheses]**
- **Union construction follows the same visual distinction as record construction, with special cases only for arity one and arity zero** — Kyokai keeps variant syntax predictable without forcing verbose field names for the common `Some(x)` case.
**Forms**:
  - Multi-field variant: `RGB { red: 255, green: 0, blue: 0 }`
  - Single-field variant: `Some(42)`
  - Zero-field variant: `None`
  **Rules**:
  1. A multi-field variant is constructed with `VariantName { field: expr, ... }` and follows the same field rules as D35: every field exactly once, no partial construction, no hidden defaults, and source-order evaluation of field expressions.
  2. A variant with exactly one payload field is constructed with `VariantName(expr)`.
  3. A variant with no payload fields is written as the bare constructor name with no parentheses.
  4. `=>` is not part of variant construction syntax.
  **Why this fits Kyokai**: multi-field variants stay aligned with record construction, single-field variants stay ergonomic, and zero-field variants stop pretending to be nullary calls.
  **[STAGE: DECIDED_CORE_SEMANTICS | D65 → multi-field braces, single-field parens, zero-field bare constructor names]**
- **Indexing syntax is built-in surface sugar over a fixed language-defined indexing protocol family** — Kyokai allows `a[i]` and `a[i] := value` only through compiler-known `Indexable` / `IndexableMut` typeclasses, not through ad hoc name lookup or a special-case list hidden in the compiler.
**Protocol family**:
  ```kyokai
  typeclass Indexable(Self: Type, Idx: Type) is
      type Item;
      method index(self: &[Self], idx: Idx): &[Self.Item];
  spec;

  typeclass IndexableMut(Self: Type, Idx: Type) is
      type Item;
      method indexMut(self: &![Self], idx: Idx): &![Self.Item];
  spec;
  ```
  **Rules**:
  1. `a[i]` is legal only when the selected type implements `Indexable` for the type of `a` and the type of `i`.
  2. `a[i] := value` is legal only when the selected mutable place implements `IndexableMut`; the assignment writes through the returned mutable element borrow.
  3. Invalid indexing according to that type's indexing domain is a uniform language contract violation and triggers TPOE. Implementations may not silently return sentinel values, wrap indices, or choose a different fallback.
  4. Fallible or non-TPOE lookup must use an explicitly named API such as `get`, `tryGet`, or another ordinary function. `[]` is the total-or-TPOE surface only.
  5. The standard library provides `Indexable` / `IndexableMut` instances for arrays, fixed-size arrays, spans, buffers, and any other containers whose own decisions explicitly opt into this protocol family.
  6. `String` is not indexable with `[]` unless a future decision explicitly adds an instance; text indexing is intentionally not treated as ordinary random-access character lookup.
  **Why this fits Kyokai**: the language gets one explicit indexing mechanism instead of a magic whitelist or a loose naming convention, and every `[]` operation keeps the same visible TPOE contract.
  **[DECIDED: D36/D132 → built-in `[]` sugar over fixed `Indexable` / `IndexableMut` typeclasses with uniform TPOE indexing semantics]**
- **Slice syntax is built-in checked half-open sugar for span extraction on standard sequential containers** — Kyokai allows `a[i..j]` because it is the same closed, compiler-known container family as D36 indexing, and its safety contract is stated directly rather than hidden behind ad hoc helper calls.
**Rules**:
  1. Slice syntax is legal only on the same standard sequential container family and ordinary borrowed/mutable-borrowed forms that provide compiler-known sequential indexing under D36.
  2. `a[i..j]` denotes the half-open range beginning at `i` and ending just before `j`.
  3. `a[i..]` means from `i` to `length(a)`. `a[..j]` means from `0` to `j`. `a[..]` means the whole span view.
  4. The required bound relation is `i <= j <= length(a)` after omitted endpoints are filled in. Violation triggers TPOE.
  5. The language contract is this checked range relation, not a raw `j - i` arithmetic formulation.
  6. On an immutable source, the result is `Span[T]`. On a mutable source, the result is `SpanMut[T]`.
  7. `String` does not gain this syntax directly. Text-to-bytes bridging remains explicit through named operations such as `asBytes()`.
  **Why this fits Kyokai**: the everyday half-open slicing pattern becomes readable without weakening the closed-container model, and the bounds contract is explicit instead of being folklore inherited from another language.
  **[STAGE: DECIDED_CORE_SEMANTICS | D106 → built-in checked half-open slice syntax for standard sequential containers; direct bound relation, not `j - i` folklore]**
- **Human-readable formatting is standardized through `Displayable` plus a dedicated formatting-sink abstraction, not through raw byte-stream I/O** — Kyokai separates "render text" from "transport bytes" so display logic does not inherit low-level partial-write semantics.
**Core typeclasses**:
  ```kyokai
  typeclass FormatSink(S: Type) is
      type Error;
      method emitUtf8(out: &![S], chunk: &[Span[Nat8]]): Result[Unit, S.Error];
  spec;

  typeclass Displayable(T: Type) is
      method display[S: FormatSink](value: &[T], out: &![S]): Result[Unit, S.Error];
  spec;
  ```
  **Rules**:
  1. `Displayable` is the standard protocol for human-readable formatting.
  2. `FormatSink` is the rendering-target abstraction. Its `emitUtf8` contract is whole-chunk-or-error, not D66 partial-write semantics.
  3. The bytes passed to `emitUtf8` must encode valid UTF-8 text. Supplying invalid UTF-8 to a safe formatting sink is a contract violation and triggers TPOE.
  4. `StringBuilder` implements `FormatSink` with `Error = AllocError` and follows D74's explicit allocation-failure rules when growth is required.
  5. `Displayable` implementations may emit multiple chunks and may compose by calling `display` on nested fields.
  6. The standard library provides built-in `Displayable` instances for the language's primitive displayable types, including integers, floats, booleans, and strings.
  7. Formatting through `Displayable` is locale-independent and deterministic. The meaning of decimal separators, boolean text, and other standard textual forms does not vary by host locale.
  8. Generic APIs may require `T: Displayable` to render values in a type-safe way, and generic formatting infrastructure may require `S: FormatSink`.
  9. String-embedded placeholder syntax is not part of D40 itself; D40a defines the allocating interpolation path, and D102 defines the direct-to-stream non-allocating path.
  **Why this fits Kyokai**: the display protocol becomes reusable across allocating and non-allocating output paths, while the boundary between text rendering and byte-stream transport stays explicit instead of being hidden inside `Displayable`.
  **[STAGE: DECIDED_CORE_SEMANTICS | D40 → `Displayable` + `FormatSink`; rendering stays separate from raw stream transport; interpolation split to D40a/D102]**
- **Standard error reporting is a standalone diagnostic protocol, not a superclass of `Displayable` and not a requirement on all error payloads** — recoverable errors remain ordinary typed values, while generic logging/reporting gets one explicit optional surface.
**Core typeclass**:
  ```kyokai
  typeclass StandardError(E: Type) is
      method errorName(err: &[E]): StaticString;
      method writeError[S: FormatSink](err: &[E], out: &![S]): Result[Unit, S.Error];
  spec;
  ```
**Rules**:
  1. `StandardError` is optional. A type may be used as the `E` in `Result[T, E]` without implementing `StandardError`.
  2. `StandardError` does not extend `Displayable`, and `Displayable` does not imply `StandardError`.
  3. `errorName` returns a stable, non-localized diagnostic name suitable for logs, metrics, and terse messages.
  4. `writeError` emits human-readable diagnostic text through D40's `FormatSink` and must be deterministic and locale-independent unless a separate localization API is explicitly used.
  5. `StandardError` is for diagnostics, logs, and generic reporting only. It does not participate in ordinary error propagation, `or return`, `let...else`, or type conversion.
  6. Structured error causes/chains are not part of the base interface. If Kyokai adds causal error chains later, that requires a separate D-point covering ownership, allocation, and cycle rules.
**Why this fits Kyokai**: generic diagnostics become possible without making every error printable, importing Rust's trait inheritance shape, or pretending error causality is free.
  **[STAGE: DECIDED_CORE_SEMANTICS | D259 → standalone optional `StandardError` diagnostic typeclass; no `Displayable` inheritance and no base `source()` chain]**
- **I/O stream abstraction is byte-oriented, minimal, and layered** — Kyokai uses small `Readable`/`Writable` typeclasses for byte streams and puts convenience helpers on top instead of baking every policy into the core trait.
**Core typeclasses**:
  ```kyokai
  typeclass Readable(T: Type) is
      method read(stream: &![T], buf: &![SpanMut[Nat8]]): Result[Index, IoError];
  spec;

  typeclass Writable(T: Type) is
      method write(stream: &![T], buf: &[Span[Nat8]]): Result[Index, IoError];
      method flush(stream: &![T]): Result[Unit, IoError];
  spec;
  ```
  **Rules**:
  1. These stream traits are byte-oriented. Text decoding and text encoding wrappers live above them; they are not implicit parts of the core I/O contract.
  2. `Span[T]` is an immutable borrowed view of contiguous memory. `SpanMut[T]` is the mutable equivalent, providing a view of contiguous memory that allows modifying the elements. It is an alias for `&![Span[T]]` or a distinct type, depending on backend representation, but conceptually it represents a mutable slice.
  3. `read` may perform a partial read. `Ok(0)` means end-of-file or end-of-stream in the normal stream sense.
  4. `write` may perform a partial write. `Ok(n)` means exactly `n` bytes were accepted. `Ok(0)` means no bytes were accepted; `writeAll`-style helpers must treat repeated zero-byte success on non-empty input as a no-progress error rather than looping forever.
  5. I/O failure is reported through `Result[..., IoError]`, not through TPOE, unless a separate API explicitly states a TPOE contract.
  6. Convenience operations such as `readExact`, `writeAll`, and `copyAll` are ordinary library functions layered on top of these minimal traits.
  7. Buffered wrappers remain explicit adapter types; core `Readable`/`Writable` does not imply buffering.
  **Why this fits Kyokai**: the abstraction is strong enough for generic file/socket/memory-copy code, but still small, explicit, and aligned with the language's text-vs-bytes separation and result-based I/O error handling.
  **[STAGE: DECIDED_CORE_SEMANTICS | D66 → minimal byte-oriented `Readable`/`Writable` with explicit helpers and flush]**
- **Allocator choice is explicit, value-level, and never ambient** — Kyokai supports custom allocators without infecting every container type with an allocator parameter and without hiding allocation policy behind a global default.
**Core typeclass**:
  ```kyokai
  typeclass Allocator(A: Type) is
      method allocate(alloc: &![A], size: Index, align: Index): Result[Address[Nat8], AllocError];
      method deallocate(alloc: &![A], ptr: Address[Nat8], size: Index, align: Index): Unit;
      method reallocate(
          alloc: &![A],
          ptr: Address[Nat8],
          oldSize: Index,
          newSize: Index,
          align: Index
      ): Result[Address[Nat8], AllocError];
  spec;
  ```
  **Rules**:
  1. Core allocating APIs require an explicit allocator value at the construction site. Kyokai does not use an ambient global allocator as the silent default for dynamic containers.
  2. Allocators are ordinary values/capabilities, not container type parameters. `Buffer[T]` does not become `Buffer[T, A]`.
  3. Owning dynamic containers must store allocator identity as runtime state so that in-place growth, shrink, and destruction use the same allocator that created the storage.
  4. Operations that create a new owned container from existing data require an explicit destination allocator unless a more specific rule is stated. The standard naming pattern is `...In`, such as `cloneIn`, `concatIn`, or `collectIn`.
  5. Ordinary allocation failure is reported through `Result[..., AllocError]` by default, per D74. Fatal convenience forms are allowed only when their names make the policy explicit, such as `mustMakeByteBuf`.
  6. Kyokai's ordinary `Allocator` abstraction is for allocators that support individual deallocation and reallocation. Pool-style allocators fit here if they satisfy that contract.
  7. Pure bump/arena allocators are not standardized as ordinary general-purpose container allocators. D96 defines them separately as scoped non-escaping region allocators rather than as ordinary `Allocator` implementations.
  8. Kyokai does not define hidden allocator propagation rules such as "new containers use the left-hand operand's allocator". If an operation allocates a fresh owned result, the allocator choice must be explicit unless a narrower rule is written down.
  9. **Container destruction**: Containers store explicit allocator runtime state internally so that destruction and in-place growth use the same allocator choice that created the storage. This stored allocator state is part of the container's own runtime representation under D44/D130, not a borrow to ambient caller state and not a hidden typeclass dictionary. Arena-style scoped bulk allocators belong to D96 instead of this ordinary container-destruction path.
  10. Kyokai does not provide `Allocator.default()`, ambient allocator lookup, implicit lexical allocator context, or thread-local allocator selection.
  11. Deep call stacks that need allocation should pass an ordinary explicit context value containing the allocator and any other needed authority or policy. Such context values are user-visible parameters, not language-level implicit parameters.
  **Examples**:
  ```kyokai
  var heap := root.systemAllocator();

  let Ok(out) := heap.makeByteBuf(16384) else Err(ignore) do
      return ExitOutOfMemory();
  fi;
  defer out.destroy();

  let Ok(copy) := out.cloneIn(&!heap) else Err(ignore) do
      return ExitOutOfMemory();
  fi;
  defer copy.destroy();
  ```
  ```kyokai
  var heap := root.systemAllocator();

  var out := heap.mustMakeByteBuf(16384);
  defer out.destroy();
  ```
  **Why this fits Kyokai**: allocator policy remains visible in source, OOM behavior stays explicit, containers avoid viral allocator type parameters, and ordinary allocator-backed containers stay separate from D96's scoped arena model instead of quietly depending on hidden lifetime coupling.
  **[STAGE: DECIDED_CORE_SEMANTICS | D44/D250 → explicit value-level allocator choice; no hidden/default/thread-local allocator; ordinary `Allocator` for individually deallocatable storage only]**
- **Allocator participation in stdlib APIs is determined by the storage effect of the operation, and fresh owned results never silently inherit a destination allocator** — Kyokai makes allocation choice visible at the exact boundary where new owned storage may appear.
**Rules**:
  1. Operations that mutate an existing owning value in place use that value's stored allocator when they need to grow, shrink, or reallocate. They do not take a separate destination allocator parameter.
  2. Operations that return only borrowed views or other non-owning results do not take an allocator parameter and must not allocate.
  3. Operations that produce a fresh owned value from borrowed data, from iteration, or from multiple inputs require an explicit destination allocator unless a more specific rule states that the result reuses existing storage without fresh allocation.
  4. The standard naming pattern for rule 3 is `...In`, such as `cloneIn`, `toStringIn`, `concatIn`, and `collectIn`.
  5. Consuming conversions named `into*` may omit a destination allocator only when their contract guarantees that the result is formed by ownership transfer, storage reuse, or another non-allocating transformation.
  6. If a consuming conversion may need fresh destination allocation as part of ordinary successful execution, it must take an explicit destination allocator and use an allocator-explicit spelling such as `intoBytesIn`.
  7. Built-in allocating constructs that are not ordinary methods or library functions, such as `format(alloc, ...)`, still follow the same principle: destination allocation is explicit even when the surface does not use an `...In` suffix.
  8. Kyokai defines no hidden destination-allocator inheritance from the receiver, the left-hand operand, the current module, the current task, or any ambient default allocator.
  9. Standard-library documentation and `.kyo` interfaces must make each API's allocator behavior class obvious: in-place using stored allocator, non-allocating view/borrow, consuming storage-reuse conversion, or fresh-allocation-with-explicit-destination.
  10. Owning containers store allocator identity as ordinary runtime state chosen explicitly in source. This is not an existential type, not a trait object, not a hidden generic dictionary, and not a viral allocator type parameter.
  11. `Buffer[T]`, `String`, and ordinary owning containers remain parameterized by their element or domain types only. They do not become `Buffer[T, A]` or acquire default allocator type parameters.
  **Examples**:
  ```kyokai
  let text: String := bytes.toStringIn(&!heap) or return;
  let copy: Buffer[Nat8] := buf.cloneIn(&!heap) or return;
  let merged: Buffer[Nat8] := collectIn(&!heap, iter) or return;

  buf.push(byte) or return;           // uses buf's stored allocator if growth is needed
  let span := buf.asSpan();           // view only, no allocation
  let raw: ByteBuf := text.intoBytes(); // consuming storage transfer, no destination allocator
  ```
  **Why this fits Kyokai**: every fresh owned allocation site stays explicit, existing containers still mutate ergonomically through their stored allocator, and the naming surface now tells both the ownership story and the allocator story without hidden propagation rules.
  **[STAGE: DECIDED_CORE_SEMANTICS | D201/D251 → stdlib allocator participation follows storage effect; fresh owned results require explicit destination allocator; allocator identity is explicit runtime state, not type erasure or allocator type parameters]**
- **Arenas are a separate linear scoped-allocation model, not ordinary `Allocator` implementations** — Kyokai supports arena/bump-style bulk allocation safely, but only through the region system and only as an explicit non-escaping model rather than by pretending arena-backed ordinary containers are the same thing as heap-backed owned containers.
**Rules**:
  1. `Arena` is a `Linear` region allocator type distinct from D44's ordinary `Allocator` abstraction.
  2. Creating an arena is explicit and fallible. Any backing-storage allocation uses an ordinary allocator and therefore follows D74's `AllocError` rules unless an explicitly named fatal convenience API is chosen.
  3. Safe arena allocation occurs only through an explicit borrow of the arena. Arena-derived values and borrows are tied to that borrow region under D6; they cannot escape the borrow scope that produced them.
  4. Individual deallocation of arena allocations does not exist.
  5. Bulk free is explicit and consuming. Destroying an arena consumes it. Resetting an arena, if provided, also consumes the old arena state and returns a fresh empty arena state.
  6. Because bulk free consumes the arena, it is statically illegal while arena borrows or arena-derived values tied to the active borrow region remain live.
  7. Arenas do not implement the ordinary `Allocator` interface for `Buffer[T]`, `StringBuilder`, or other ordinary owning container types.
  8. If the standard library provides growable arena-local collection types, they are distinct arena-scoped types whose validity remains tied to the arena borrow/region. They are not secretly arena-backed ordinary owning containers.
  9. Moving data from arena-local storage into an ordinary owned container requires an explicit copy into an ordinary allocator-backed destination. There is no implicit escape or hidden promotion from arena-local storage to general owned storage.
  10. Unsafe code may still take raw addresses into arena storage, but safe code gets no escape hatch around the region/non-escape rules.
  **Why this fits Kyokai**: parsers, compilers, game loops, and batch pipelines get real bulk-allocation performance, but Kyokai keeps the lifetime story explicit by making arenas live inside the existing region model rather than hand-waving about "allocator lifetime" in ordinary owned containers.
  **[STAGE: DECIDED_CORE_SEMANTICS | D96 → arenas are separate linear region allocators; arena-derived values are non-escaping; bulk free/reset are explicit consuming operations]**
- **OS-native strings and paths are dedicated types, not UTF-8 `String` by convention** — filesystems and process boundaries are not uniformly UTF-8 across targets, so Kyokai models native text and paths explicitly instead of pretending `String` is the right type everywhere.
**Core types**:
  ```kyokai
  OsString   // owned platform-native text
  OsStr      // borrowed platform-native text view
  PathBuf    // owned path buffer wrapping OsString
  Path       // borrowed path view wrapping OsStr
  ```
  **Rules**:
  1. `OsString` is an owned platform-native string type and is not guaranteed UTF-8. `OsStr` is its borrowed view.
  2. `PathBuf` is the owned path type and wraps `OsString`. `Path` is the borrowed path-view type and wraps `OsStr`.
  3. On Unix-like targets, the platform-native representation is an arbitrary non-NUL byte sequence. On Windows, it is the platform's native wide-path representation. Safe Kyokai does not collapse those into one fake UTF-8 contract.
  4. Filesystem APIs accept borrowed `Path` values and return `PathBuf` values. They do not use `String` as the path type.
  5. Path operations are lexical only. The standard path surface includes operations such as `join`, `parent`, `fileName`, `extension`, `components`, `isAbsolute`, and `isRelative`.
  6. Path operations perform no implicit normalization, canonicalization, case-folding, separator rewriting, symlink resolution, or `.` / `..` collapse.
  7. Equality on `OsStr`, `OsString`, `Path`, and `PathBuf` is stored-representation equality, not filesystem identity.
  8. Conversion from `OsStr`/`Path` to `String` is explicit and fallible because the platform-native representation may not be valid UTF-8.
  9. Conversion from `String` to `OsString`/`PathBuf` is explicit and validating. It must reject interior NUL.
  10. There is no implicit conversion between `String` and OS-native string/path types in either direction.
  11. This decision does not silently settle argv, environment, process-title, or other OS-text boundaries. Those surfaces need their own explicit contracts; they must not inherit `String` merely because `OsString` now exists.
  **Why this fits Kyokai**: paths and native OS text stop being folklore carried by `String`, while the exact places where UTF-8 validity is or is not guaranteed remain visible in the type system.
  **[STAGE: DECIDED_CORE_SEMANTICS | D97 → dedicated `OsString`/`OsStr` + `PathBuf`/`Path`; lexical-only path ops; no implicit UTF-8/path conversion]**
- **Type aliases are explicit transparent synonyms, not new nominal types** — Kyokai allows aliasing existing types for ergonomics, but the syntax must say clearly that an alias does not create a new type identity.
**Syntax**:
  ```kyokai
  type alias FileDescriptor := Int32;
  type alias IoResult[T] := Result[T, IoError];
  ```
  **Rules**:
  1. `type alias Name := Target;` creates a transparent synonym for `Target`.
  2. An alias introduces no new nominal identity, no new ABI identity, and no separate typeclass/coherence identity.
  3. Alias and target type are identical for assignment, parameter passing, instance lookup, pattern typing, and foreign layout.
  4. Generic aliases are allowed.
  5. Cyclic aliases are illegal.
  6. If a programmer needs a semantically distinct domain type or invariant boundary, they must use a record/newtype rather than an alias.
  **Why this fits Kyokai**: the programmer writes exactly what they mean, aliases help FFI and library ergonomics, and the language does not pretend a pure synonym is a safety feature.
  **[STAGE: DECIDED_CORE_SEMANTICS | D50 → explicit `type alias` syntax for transparent synonyms only]**
- **Foreign integer constant domains and bitflags use explicit integer aliases plus named constants, not a separate language-level C-enum kind** — Kyokai keeps its own semantic enums as unions and models raw C-style integer domains honestly at the FFI edge.
**Syntax**:
  ```kyokai
  type alias OpenFlags := Int32;
  constant O_RDONLY: OpenFlags := 0;
  constant O_CLOEXEC: OpenFlags := 0x80000;
  ```
  **Rules**:
  1. When a foreign API exposes a named integer domain or bitflag set and its ABI representation is explicitly fixed, Kyokai models that raw surface as a `type alias` to the chosen integer type plus named `constant` declarations.
  2. This model does not create a closed set or a nominal type. Any value representable by the underlying integer type is also representable by the alias.
  3. Kyokai does not add a separate language-level “C enum” construct. Kyokai's semantic enums remain unions with exhaustive pattern matching.
  4. If safe Kyokai code wants a closed semantic domain, it must wrap or translate the raw integer form into a union or another nominal type.
  5. Kyokai never guesses the representation of a foreign `enum`. If the binding contract does not make the integer representation explicit, the API must go through a C shim or another separately specified mapping.
  6. Bitflag combination and testing use Kyokai's ordinary explicit bitwise operators from D41 on the chosen integer representation.
  **Why this fits Kyokai**: the raw boundary stays ABI-honest and explicit, while safe wrappers can still expose stronger domain semantics when the foreign API actually supports them.
  **[STAGE: DECIDED_CORE_SEMANTICS | D61 → foreign integer constant domains and bitflags use explicit integer aliases + constants; closed semantics belong in wrappers]**
- **Non-byte-aligned field layouts use `bitrecord`, not C-style bitfields** — Kyokai supports readable register and protocol definitions, but it defines them against explicit integer bit positions rather than backend-defined memory layout folklore.
**Syntax**:
  ```kyokai
  bitrecord TcpFlags: Nat16 is
      reserved bits 15..13;
      field dataOffset: bits 12..9;
      reserved bits 8..6;
      field urg: bit 5;
      field ack: bit 4;
      field psh: bit 3;
      field rst: bit 2;
      field syn: bit 1;
      field fin: bit 0;
  build;
  ```
  **Rules**:
  1. `bitrecord Name: NatN is ... build;` defines a nominal value type backed by exactly one fixed-width unsigned integer type: `Nat8`, `Nat16`, `Nat32`, or `Nat64`.
  2. Bit positions are numbered from `0` at the least-significant bit to `N - 1` at the most-significant bit of the backing integer value.
  3. `field name: bit k;` defines a one-bit boolean field. `field name: bits hi..lo;` defines an unsigned integer field over the inclusive range `hi` down to `lo`.
  4. Every bit of the backing integer must be covered exactly once by either a `field` declaration or a `reserved` declaration. Overlap, gaps, empty ranges, or out-of-range positions are compile-time errors.
  5. A `bitrecord` supports explicit raw conversion with `toBits(self)` and `fromBits(bits)`. `toBits` returns the backing integer exactly. `fromBits` preserves the backing integer exactly.
  6. Named construction is explicit and field-based: `TcpFlags { dataOffset: 5, urg: false, ack: true, ... }`. Every non-reserved field must be provided exactly once, and all reserved bits are set to zero by named construction.
  7. Projecting `value.field` copies the decoded field value out of the backing integer. One-bit fields have type `Bool`. Multi-bit fields have the backing integer type with the selected bit range shifted down so its least-significant field bit becomes bit `0`.
  8. Assigning a multi-bit field value through named construction is checked against the declared field width. If any non-zero bits would be truncated, the program is rejected at compile time for compile-time-known values and TPOE at runtime otherwise.
  9. Bitrecord fields have no address and no borrow semantics. Taking `&value.field` or `&!value.field` is illegal. A bitrecord does not expose C-style lvalue field overlays.
  10. `bitrecord` is distinct from D42's `record`, `extern record`, and `packed record` layout classes. When a `bitrecord` value is stored in memory, its storage is exactly the storage of its backing integer type.
  11. Endianness is handled separately by D117. Serializing or parsing a `bitrecord` across a byte boundary must go through the backing integer plus explicit endian conversion / byte-encoding helpers.
  12. The C backend lowers bitrecord access with masks, shifts, and equivalent helper code only. It never emits C bitfields as the semantic implementation strategy.
  **Why this fits Kyokai**: hardware headers and protocol words become readable without importing C's compiler-dependent bitfield rules or leaving truncation/layout behavior implicit.
  **[STAGE: DECIDED_CORE_SEMANTICS | D116 → explicit `bitrecord` values over fixed-width unsigned backing integers; masks/shifts only; no C-style or backend-defined bitfields]**
- **Endianness operations are explicit fixed-width integer transforms plus explicit byte encoding helpers** — Kyokai does not hide byte order in FFI, packed layout, or I/O boundaries; the programmer names the conversion and the exact bytes they want.
**Rules**:
  1. This decision applies only to the built-in fixed-width integer families such as `Int8`, `Int16`, `Int32`, `Int64`, `Nat8`, `Nat16`, `Nat32`, and `Nat64`. It does not apply to floats, records, unions, `String`, or target-dependent integer types such as `Index`.
  2. Every fixed-width integer type provides `swapBytes()`, `toBigEndian()`, `toLittleEndian()`, `fromBigEndian()`, and `fromLittleEndian()`.
  3. For one-byte integer types, all of these operations are identity operations.
  4. `swapBytes()` means unconditional byte reversal.
  5. `toBigEndian()` / `toLittleEndian()` and `fromBigEndian()` / `fromLittleEndian()` are defined in terms of `target.endianness`. On a target whose endianness already matches the requested form, the operation is a no-op; otherwise it is the corresponding byte swap.
  6. `target.endianness` is a language-level comptime constant of enum type `Endian` with variants `Endian.Little` and `Endian.Big`.
  7. The standard library also provides explicit byte-array encoding and decoding helpers for fixed-width integers, equivalent to `toBigEndianBytes()`, `toLittleEndianBytes()`, `fromBigEndianBytes(...)`, and `fromLittleEndianBytes(...)`, because value-level endianness transforms alone do not specify wire-format byte layout.
  8. Kyokai performs no automatic byte swapping in `packed record`, FFI boundaries, file I/O, network I/O, or containers. Any endianness conversion at those boundaries must be written explicitly.
  **Why this fits Kyokai**: network and file-format code gets the operations it actually needs, while byte order stays a named visible step instead of a backend- or platform-driven surprise.
  **[STAGE: DECIDED_CORE_SEMANTICS | D117/D260 → fixed-width integer endianness methods + explicit byte-array encode/decode helpers; no automatic boundary swapping]**
- `**debug` is a language-level built-in keyword for development-only console output; production console I/O requires `TerminalCapability`** — Kyokai splits console output into two completely separate mechanisms with different rules, different build behavior, and different philosophical status.
**Debug output — the `debug` keyword**:
  ```kyokai
  debug expr;    // built-in keyword — prints Displayable repr to stderr + newline
  debug "hello"; // string literal form
  debug x;       // any Displayable (D40) expression
  ```
  **Rules**:
  1. `debug expr;` is a language-level built-in construct, like `panic(message)` or `return expr`. It is not a function call, not a module import, and not UFCS.
  2. `expr` must satisfy `Displayable` (D40).
  3. Output goes to stderr.
  4. In release builds, the compiler strips `debug` statements entirely — no code is emitted.
  5. In debug and test builds, `debug` emits to stderr.
  6. The compiler emits a warning if `debug` appears in non-test code under release profile.
  7. `debug` does NOT require any capability, does NOT appear in the module's interface, and is NOT part of the program's observable behavior contract.
  8. `debug` is instrumentation — like a debugger breakpoint. It exists outside the capability model.
  9. **Linearity interaction**: `debug x;` immutably borrows `x` (`&[x]`) to pass to `Displayable`. It does NOT consume linear values. This is crucial because `debug` statements disappear in release builds; if they consumed values, release builds would fail linearity checks.
  10. In debug and test builds, `debug` formats through the same D40/D40a/D102 machinery and performs a best-effort stderr write. If stderr output fails, the failure is ignored and `debug` does not change program control flow.
  11. The operand of `debug expr;` is a debug-observation expression, not an unrestricted ordinary expression.
  12. A debug-observation expression may observe an already-existing value through local names, constants, literals, immutable field projection, immutable index/slice projection, and parenthesized composition of those forms.
  13. A debug-observation expression may not contain ordinary function calls, UFCS calls, constructors that allocate, arithmetic or comparison operations that may TPOE, assignment, `comptime`, `panic`, `return`, `break`, `continue`, `or ...`, `defer`, or any capability-using operation.
  14. If the programmer wants to debug a computed value, the computation must be bound in ordinary code first, and then that binding may be observed with `debug`.
  15. Formatting and best-effort stderr emission performed by the `debug` machinery are debug/test-profile instrumentation effects only; they may not affect ordinary program state, capability flow, or linear obligations.
  **Debug expression purity**: D233 closes the profile-equivalence hole in D45. Because release builds strip `debug`, the expression inside `debug` cannot be the only place where a value is consumed, a mutable borrow is created, a side effect happens, or a possible contract failure is introduced.
  **[STAGE: DECIDED_CORE_SEMANTICS | D233 → `debug` observes existing values only; no linear consumption, mutable borrow creation, ordinary side effects, or profile-dependent control flow]**
  **Production output — capability-gated**:
  ```kyokai
  function main(root: RootCapability, args: &[Span[String]]): ExitCode is
      let terminal: TerminalCapability := acquireTerminal(&!root);
      defer releaseTerminal(terminal);
      Io.writeLn(&!terminal, "Hello, world");
  qed;
  ```
  **Rules**:
  1. `Io.writeLn`, `Io.write`, and all production console output functions require `TerminalCapability`.
  2. `TerminalCapability` is acquired from `RootCapability` — the capability chain is explicit.
  3. This is ordinary capability-gated I/O, no different from file I/O or network I/O.
  **Build behavior**:

  | Build mode | `debug expr;`                                              | `Io.writeLn(...)`                             |
  | ---------- | ---------------------------------------------------------- | --------------------------------------------- |
  | Debug      | Available. Writes to stderr.                               | Available. Requires capability.               |
  | Release    | **Stripped** (calls removed). Compiler warning if present. | Available. Requires capability.               |
  | Test       | Available. Writes to stderr.                               | Available. Test runner provides capabilities. |

  **Why `debug` is a keyword, not `debug.print()` or `Debug.print()`**: `debug.print()` looks like module access (`Module.function()`), which would imply it's imported, has a module interface, and participates in the normal module system. It is none of those things. Making it a keyword puts it in the same syntactic category as `panic` — a built-in construct the compiler knows about and handles specially.
  **Why this is philosophically acceptable**: `debug` is explicitly NOT part of the program's intended behavior. It is development instrumentation. The `debug` keyword makes it visually distinct, the compiler warns about it, and release builds strip it. The capability model is not compromised because `debug` is defined as outside the model — the same way `panic` is outside normal control flow.
  **Why not full capability enforcement (Option A)**: every language with capability-based I/O provides an escape hatch for debugging. Haskell has `Debug.Trace.trace`. Requiring `TerminalCapability` for ephemeral debug prints would force capability threading through every function just to add a temporary print statement. That is ceremony that impedes development for zero safety benefit — debug prints are ephemeral and stripped before shipping.
  **[STAGE: DECIDED_CORE_SEMANTICS | D45 → `debug` keyword for development-only output; production I/O via `TerminalCapability`; `debug` is stripped in release builds]**
- **The standard library follows the Rewrite-It-In-Kyokai (RIIK) principle: pure Kyokai implementations over FFI wrappers whenever mathematically and logically possible** — this is not a guideline; it is a design constraint that governs every standard library decision.
**Rules**:
  1. **Pure computation must be written in pure Kyokai.** This includes: math functions (H02, H03), string parsing (D69), sorting (M03), hashing (H04), cryptographic primitives, compression, encoding/decoding, data structure implementations, and any algorithm that takes inputs and produces outputs without interacting with the host OS.
  2. **OS interaction is the only legitimate use of FFI in the standard library.** This includes: syscalls, file I/O, networking, threading primitives, memory allocation (the allocator itself, not containers built on top), process spawning, and signal handling. These require FFI because they interact with the host kernel — there is no pure alternative.
  3. **Even FFI-allowed code must have a thin trust boundary.** The FFI call itself lives in a `pragma Unsafe_Module`. The public API exposed to safe Kyokai code must be a safe wrapper that enforces linearity, capability requirements, and error handling (D20/D20a/D20b). The unsafe surface is as small as possible.
  4. **Complex legacy protocols may use FFI transitionally.** External codebases like SQLite or TLS libraries may be wrapped via FFI initially. The long-term roadmap prioritizes pure Kyokai ports where feasible, but transitional FFI is acceptable when the alternative is no functionality.
  **Why this is a design constraint, not a preference**:
  1. **Safety**: every FFI call is an escape hatch from the linear type system, memory safety, and capability model. Pure Kyokai code is mechanically verified by the compiler. FFI code is trusted on faith.
  2. **Portability**: relying on `libc` or `libm` ties the language to the host OS's C implementation quirks. A pure Kyokai `sin()` behaves identically across all platforms.
  3. **Capability model**: FFI calls often hide global state or side effects (e.g., `errno`). Pure Kyokai implementations respect the capability model natively.
  4. **Reproducibility**: D83 (reproducible builds) is undermined if stdlib behavior depends on which `libm` version the host shipped.
  **Why this is already the operating principle**: H02 (math library) explicitly requires pure Kyokai implementations. H03 (trig/exponentials) explicitly requires pure implementations. D30/D30a (string encoding) define text handling in Kyokai terms, not as wrappers around `libc` string functions. D44 (allocator abstraction) wraps the syscall-level allocator but builds all container logic in pure Kyokai. The RIIK principle is already threaded through the plan — D64 formalizes it.
  **[STAGE: DECIDED_CORE_SEMANTICS | D64 → RIIK principle: pure Kyokai for computation, FFI only for OS interaction, thin trust boundaries on all FFI, transitional FFI for legacy protocols]**
- **Standard-library admission requires explicit correctness evidence before an API ships as `Kyokai.*`** — RIIK is not permission to rewrite hard domains casually; standard-library code must carry a stated contract and enough evidence to make that contract credible.
**Rules**:
  1. A public `Kyokai.*` API is admitted only with a written semantic contract, edge-case behavior, allocation/failure behavior, portability notes, and conformance tests.
  2. Algorithms with external standards, reference test vectors, or mature independent implementations must name the standard or oracle used for admission tests.
  3. Numerics, parsing, encodings, compression, cryptography, collections, and other correctness-sensitive domains require tests against an oracle or independent reference where one exists, plus fuzzing or property tests where input space makes exhaustive checking infeasible.
  4. Pure computation should be implemented in safe Kyokai unless the domain has a specific boundary reason to use unsafe or FFI.
  5. Unsafe or FFI-backed implementations require an unsafe contract and must expose a safe API whose behavior is fully specified.
  6. Legacy, insecure, or compatibility-only algorithms may exist only under explicitly named compatibility modules. They are not presented as preferred modern defaults.
  7. Documentation and conformance metadata must state the admitted contract clearly enough for `kyokai doc`, tests, and package audit tooling to surface it.
  **Why this fits Kyokai**: the standard library is part of the language's safety story, so its algorithms need explicit evidence rather than folklore trust.
  **[STAGE: DECIDED_CORE_SEMANTICS | D229 → stdlib admission requires contracts, edge-case rules, oracle/tests, and explicit legacy/compatibility boundaries]**
- **Transitional FFI is allowed for bootstrap and hard external boundaries, but it must be tracked and wrapped rather than normalized as the final shape of pure computation** — Kyokai can be pragmatic while still keeping RIIK as the destination for ordinary algorithms.
**Rules**:
  1. Bootstrap and transitional implementations may use FFI wrappers when the wrapped facility is an OS or hardware boundary, a mature external dependency needed before Kyokai can self-host, or a temporary bridge recorded in the implementation plan.
  2. Every transitional FFI wrapper must live in a `pragma Unsafe_Module`, have an unsafe contract, expose a safe Kyokai API when used by safe code, and document whether it is permanent boundary code or replacement-target bootstrap code.
  3. Pure computation should not remain FFI-backed once a correct native Kyokai implementation is available and admitted under D229, unless a separate decision justifies the exception.
  4. Transitional FFI does not relax D20, D64, D73, D242, D242a, or D245.
  5. The compiler, standard library, and toolchain may use OCaml, C, or other implementation languages during bootstrap, but the language design may not depend on those implementation-language escape hatches as user-visible semantics.
  **Why this fits Kyokai**: the project can reach self-hosting without pretending every temporary bridge is philosophically permanent.
  **[STAGE: DECIDED_CORE_SEMANTICS | D230 → transitional FFI allowed for bootstrap/external boundaries under unsafe contracts and replacement tracking]**
- **Cryptography in the standard library is standards-bound, test-vector-bound, and side-channel-explicit; Kyokai does not invent crypto** — crypto is admitted only with stronger evidence than ordinary pure algorithms, but it is not forced to remain FFI-only forever.
**Rules**:
  1. `Kyokai.Crypto` may include only named, modern, externally specified algorithms and protocols with published test vectors and documented security properties.
  2. The standard library must not invent novel cryptographic primitives, modes, protocols, padding schemes, key derivation schemes, or random constructions.
  3. A native Kyokai crypto implementation is admissible only when it has independent review appropriate to the primitive's risk, passes official test vectors, documents constant-time and side-channel claims, and has conformance tests that protect those claims where tooling can observe them.
  4. FFI-backed crypto is allowed and often preferred when wrapping a mature audited library, but the wrapper must state ownership, initialization, randomness, error, threading, version, algorithm-availability, and cleanup contracts.
  5. OS entropy acquisition remains an OS boundary and must use capability-gated APIs under the randomness contract.
  6. APIs must separate cryptographic and non-cryptographic use. Convenience speed-oriented RNGs, hashes, and checksums must not be presented as cryptographic primitives.
  7. Deprecated or compatibility-only crypto may exist only in explicitly named compatibility modules with warnings in contract metadata.
  **Why this fits Kyokai**: crypto needs more than pure-language memory safety. The language can still RIIK where justified, but only after the algorithm, tests, and side-channel contract are explicit.
  **[STAGE: DECIDED_CORE_SEMANTICS | D231 → stdlib crypto requires modern external specs, test vectors, side-channel contracts, and review; FFI allowed but not mandatory forever]**
- **Environment variables require `EnvCapability` — the environment is global mutable state and must be capability-gated** — reading environment variables can change program behavior based on external factors; modifying them affects child processes. Kyokai treats the process environment as external state that requires the same explicit capability discipline as the filesystem or network.
**API**:
  ```kyokai
  function getEnv(env: &![EnvCapability], key: &[String]): Optional[String];
  function setEnv(env: &![EnvCapability], key: &[String], value: &[String]): Result[Unit, EnvError];
  function removeEnv(env: &![EnvCapability], key: &[String]): Result[Unit, EnvError];
  ```
  **Rules**:
  1. `EnvCapability` is acquired from `RootCapability`, like all other capability types.
  2. `getEnv` returns `Optional[String]` — `None` if the variable is not set. No TPOE on missing variables.
  3. `setEnv` and `removeEnv` return `Result` because the OS may reject the operation.
  4. There is no global `getenv()` function. Any code that reads environment variables must receive `EnvCapability` explicitly.
  5. The capability is mutably borrowed (`&!`) because even reading the environment is observation of external mutable state.
  **Why this fits Kyokai**: the environment is hidden global state in every other systems language. Capability-gating it makes the dependency on external state visible in function signatures. If a function reads `$HOME`, its signature says so.
  **[STAGE: DECIDED_CORE_SEMANTICS | D67 → `EnvCapability` for all environment variable access; no ambient `getenv`]**
- **Filesystem access is capability-gated, path-typed, and free of ambient current-directory semantics** — Kyokai defines safe file and directory operations explicitly instead of inheriting a process-global cwd model from C and POSIX folklore.
**Core types**:
  ```kyokai
  record File is
      // opaque
  build;

  record Directory is
      // opaque
  build;

  union OpenMode is
      case ReadOnly;
      case WriteOnly;
      case ReadWrite;
      case Append;
  build;

  union SeekFrom is
      case FromStart(offset: Index);
      case FromCurrent(delta: Int64);
      case FromEnd(delta: Int64);
  build;

  union FileKind is
      case Regular;
      case Directory;
      case Symlink;
      case CharacterDevice;
      case BlockDevice;
      case Fifo;
      case Socket;
      case Other;
  build;

  record Metadata is
      kind: FileKind;
      size: Index;
  build;
  ```
  **Rules**:
  1. `FileCapability` is acquired explicitly from `RootCapability`.
  2. Safe filesystem APIs are capability-gated. There are no ambient global file or directory functions.
  3. Filesystem APIs take `Path` / `PathBuf`, not `String`.
  4. `File` and `Directory` are linear resource handles.
  5. `File` implements D66's `Readable` and `Writable` contracts when opened in a mode that permits those operations.
  6. `File` exposes explicit operations such as `seek`, `flush`, `sync`, and `metadata`; ordinary failures return `Result[..., IoError]`.
  7. Namespace operations such as opening files, creating files, renaming, removing files, creating directories, removing directories, and querying metadata are exposed through `FileCapability` and explicit `Directory` handles.
  8. Safe Kyokai does not rely on an ambient process current working directory for path resolution.
  9. Any operation on a relative path must name an explicit base `Directory` handle.
  10. Top-level `FileCapability` path operations therefore work on absolute paths only.
  11. If a hosted target exposes the process working directory, it does so only through an explicit capability-gated API returning a `Directory` handle.
  12. Filesystem errors are ordinary `IoError` results, not TPOE, unless some separate API explicitly states a stronger contract.
  **Why this fits Kyokai**: file authority stays explicit, path encoding stays honest through `Path`/`OsString`, and the language does not quietly reintroduce hidden process-global state through cwd-based resolution.
  **[STAGE: DECIDED_CORE_SEMANTICS | D171 → `FileCapability` plus explicit `File`/`Directory` handles, `Path`-typed APIs, no ambient cwd semantics, and ordinary `IoError` failures]**
- **Time splits into pure monotonic measurement values and capability-gated observable clock/sleep authority** — Kyokai keeps benchmarking and timeout arithmetic lightweight while still treating wall-clock observation and suspension as explicit external authority.
**Core types**:
  ```kyokai
  record Duration is
      // value type
  build;

  record Instant is
      // opaque monotonic timestamp
  build;

  record SystemTime is
      // opaque wall-clock timestamp
  build;
  ```
  **Rules**:
  1. `Duration` is a pure `Free` value type for elapsed-time quantities and arithmetic.
  2. `Instant` is an opaque monotonic timestamp type.
  3. Reading the monotonic clock is ungated: `monotonicNow(): Instant`.
  4. Arithmetic and comparison on `Instant` and `Duration` are pure and ungated.
  5. `SystemTime` is an opaque wall-clock timestamp type.
  6. `ClockCapability` is acquired explicitly from `RootCapability`.
  7. Reading wall-clock time requires `ClockCapability` and returns `Result[SystemTime, TimeError]`.
  8. Sleeping and other APIs that actually delay execution require `ClockCapability` and return `Result[Unit, TimeError]`.
  9. Calendar, timezone, and civil-time conversion APIs require `ClockCapability`.
  10. `Duration`, `Instant`, and `SystemTime` are ordinary values; capability requirements attach to the operations that observe or block on real time, not to storing or passing the values.
  11. `comptime` evaluation may not observe monotonic time, wall-clock time, timezone state, or sleep behavior.
  **Why this fits Kyokai**: monotonic measurement stays usable in ordinary code, while wall-clock and suspension authority remain explicit and capability-auditable.
  **[STAGE: DECIDED_CORE_SEMANTICS | D172 → ungated monotonic `Instant`/`Duration`; wall-clock, sleep, and calendar/timezone APIs require `ClockCapability`]**
- **Randomness is split between explicit entropy authority and explicit RNG state values** — Kyokai does not permit ambient global RNGs, hidden thread-local generators, or blurred crypto-versus-fast randomness surfaces.
**Core types**:
  ```kyokai
  record Seed256 is
      // opaque seed material
  build;

  record FastRng is
      // opaque deterministic fast RNG state
  build;

  record CryptoRng is
      // opaque cryptographic RNG state
  build;
  ```
  **Rules**:
  1. `RandomCapability` is acquired explicitly from `RootCapability`.
  2. There is no ambient global RNG and no hidden thread-local safe RNG.
  3. OS entropy acquisition is capability-gated.
  4. Safe APIs provide explicit entropy-fill and seed-construction operations returning `Result[..., RandomError]` on ordinary failure.
  5. `FastRng` and `CryptoRng` are explicit mutable RNG-state values, not ambient services.
  6. RNG state values are linear so mutation stays explicit through `&!` operations.
  7. Constructing RNG state from explicit seed material requires no capability.
  8. Constructing or reseeding RNG state from OS entropy requires `RandomCapability`.
  9. Fast non-cryptographic randomness and cryptographic randomness are distinct types, not a mode bit on one generic RNG surface.
  10. Once an RNG state value has been constructed, ordinary draw and fill operations on that state do not require `RandomCapability`.
  11. The standard library includes both the explicit fast RNG surface and the explicit cryptographic RNG surface; Kyokai does not defer cryptographic randomness to FFI-only folklore.
  **Why this fits Kyokai**: entropy authority stays visible, deterministic replay remains possible through explicit seeds, and security-sensitive randomness is not confused with convenience PRNG state.
  **[STAGE: DECIDED_CORE_SEMANTICS | D173 → `RandomCapability` gates entropy acquisition; explicit `FastRng` and `CryptoRng` state values; no ambient RNG]**
- **Child-process creation is capability-gated through `ProcessCapability` and uses explicit OS-native text and process-configuration types** — spawning a process is operating-system authority and must not smuggle in shell parsing or ambient process-global state.
**Core types**:
  ```kyokai
  record Process is
      // opaque child-process handle
  build;

  record ProcessConfig is
      // executable path, argv, environment mode, stdio config, working directory
  build;
  ```
  **API**:
  ```kyokai
  function spawn(proc: &![ProcessCapability], config: &[ProcessConfig]): Result[Process, ProcessError];
  ```
  **Rules**:
  1. `ProcessCapability` is acquired explicitly from `RootCapability`.
  2. Safe process APIs live in `Kyokai.Process` and require explicit `ProcessCapability`.
  3. The executable path is expressed with `Path`/`PathBuf`, not `String`.
  4. Argument and environment surfaces use `OsStr`/`OsString`-based types rather than `String` because process boundaries are OS-native text boundaries under D97.
  5. Safe spawning does not invoke a shell implicitly, does not parse one command string into argv, and does not perform shell expansion.
  6. The child working directory, environment behavior, and stdio behavior are explicit parts of `ProcessConfig`, not hidden ambient defaults. Any inheritance mode is spelled by an explicit configuration choice.
  7. `spawn` returns `Result[Process, ProcessError]` on ordinary OS failure, and `Process` is a linear handle.
  8. Waiting, signaling, piping, and stdio redirection are explicit `Kyokai.Process` APIs over `Process`, `File`, and related handle types; safe code does not gain raw `system()`-style string execution.
  **Why this fits Kyokai**: child-process creation is real authority, argv/env text stays encoding-honest, and the language rejects the most common source of process-launch ambiguity: invisible shell interpretation.
  **[STAGE: DECIDED_CORE_SEMANTICS | D178 → `ProcessCapability` gates safe process creation; executable paths use `Path`; args/env use OS-native text; no implicit shell]**
- **String-to-value parsing uses a `Parsable` typeclass returning `Result[T, ParseError]`** — parsing user input is inherently fallible. Malformed input is expected, not a contract violation, so TPOE is wrong here.
**API**:
  ```kyokai
  typeclass Parsable(T: Type) is
      method parse(s: &[String]): Result[T, ParseError];
  spec;

  // Standard implementations for built-in numeric types:
  // instance Parsable(Int32)
  // instance Parsable(Int64)
  // instance Parsable(Nat32)
  // instance Parsable(Float64)
  // etc.
  ```
  **Rules**:
  1. `parse` takes an immutable borrow of a `String` and returns `Result[T, ParseError]`.
  2. `ParseError` contains the failure reason (invalid format, overflow, empty input, etc.) and the position in the input where parsing failed.
  3. Standard implementations exist for all built-in numeric types.
  4. Parsing is exact: `"123abc".parse[Int32]()` fails rather than returning `123`. Partial parsing, if needed, is a separate API.
  5. Leading/trailing whitespace handling is explicitly specified per implementation rather than silently stripped.
  **Why this fits Kyokai**: parsing is the inverse of `Displayable` (D40). Both use typeclasses, both are explicit, and both have predictable behavior. The `Result` return makes error handling mandatory — you cannot ignore a parse failure.
  **[STAGE: DECIDED_CORE_SEMANTICS | D69 → `Parsable` typeclass with `Result[T, ParseError]`; standard implementations for numeric types]**
- **I/O is unbuffered by default; explicit `BufferedWriter[T]` and `BufferedReader[T]` wrappers provide buffering** — buffering hides memory allocations and flush timing. In a language where "if it's happening, it must be visible in source," hidden buffering violates the core principle.
**API**:
  ```kyokai
  // Unbuffered I/O is the default — every write is a syscall:
  Io.write(&!terminal, &data);  // one syscall per call

  // Buffered I/O wraps a Writable in an explicit buffer:
  let writer: BufferedWriter[File] := makeBufferedWriter(&!heap, file, 8192);
  defer writer.flush();          // explicit flush before teardown
  defer destroyBufferedWriter(writer);

  writer.write(&data);           // writes to buffer, not to OS
  writer.flush();                // explicit: pushes buffer to OS now
  ```
  **Rules**:
  1. Raw `Readable`/`Writable` (D66) operations are unbuffered — each call maps to one OS interaction.
  2. `BufferedWriter[T]` and `BufferedReader[T]` are explicit wrappers that add buffering over any `Writable` or `Readable`.
  3. `BufferedWriter[T]` and `BufferedReader[T]` are `Linear` — they own the internal buffer and must be explicitly flushed and destroyed.
  4. The buffer requires an allocator (D44) — `makeBufferedWriter(alloc, inner, bufferSize)`. The allocation is visible at construction.
  5. `flush()` is explicit. There is no implicit flush-on-newline, flush-on-full, or flush-on-scope-exit beyond what the programmer writes with `defer`.
  6. Destroying a `BufferedWriter` without flushing is a linearity error — the writer must be flushed before it can be consumed by the destructor.
  7. Capability surfaces do not smuggle in buffering. A terminal, file, or socket capability yields raw unbuffered stream authority; ergonomic helpers may construct a buffered wrapper only through an explicit API that names buffering, takes an allocator, and takes a buffer size, such as `term.bufferedWriter(&!heap, 8192)`.
  **Why this fits Kyokai**: `bfetchaust` had to implement its own `ByteBuf` because Austral provided no buffering abstraction. Kyokai provides one, but makes it explicit. The programmer sees the buffer size at construction, the allocator that backs it, and every flush point. No hidden allocations, no silent flushes, and no special capability-layer exception to the rule.
  **[DECIDED: D70/D133 → unbuffered by default; explicit `BufferedWriter[T]`/`BufferedReader[T]` wrappers only; capability APIs do not implicitly buffer]**
- **Functions may declare `require` preconditions and `ensure` postconditions as part of their signature; failed contracts are TPOE** — Design by Contract is the syntactic surface for the value contracts that linear types cannot express. Linear types enforce lifecycle contracts (resource creation, use, destruction) at compile time. `require`/`ensure` enforce value contracts (input ranges, output guarantees, domain restrictions) at runtime.
**Syntax**:
  ```kyokai
  function divide(a: Int32, b: Int32): Int32
      require b != 0;
      ensure result > 0;
  is
      return a / b;
  qed;

  function clamp(x: Int32, lo: Int32, hi: Int32): Int32
      require lo <= hi;
      ensure result >= lo;
      ensure result <= hi;
  is
      if x < lo then
          return lo;
      else if x > hi then
          return hi;
      fi;
      return x;
  qed;
  ```
  **Rules**:
  1. `require` clauses appear between the function signature and `is`. Each is a `Bool` expression evaluated at function entry. Failed `require` is TPOE (D84 rule 3).
  2. `ensure` clauses appear in the same position. Each is a `Bool` expression evaluated after the function body produces its return value but before the return value is delivered to the caller. Failed `ensure` is TPOE.
  3. In `ensure` context, `result` is the contextual keyword naming a read-only view of the function's return slot. It is not a second owned value.
  4. Semantically, `result` is an immutable borrow of the produced return value. It may be used only for pure observation and may not be moved, consumed, or mutably borrowed.
  5. For `: Unit` functions, `ensure` is allowed but `result` is not available.
  6. `require` and `ensure` are always checked. No build profile, optimization level, or compiler flag may strip or weaken them. They are language semantics, not debug assertions.
  7. `require` and `ensure` clauses appear in `.kyo` interface files and are part of the function's public contract. They are visible to callers, to documentation generators (M08), and to the compiler's diagnostic engine (D29).
  8. Multiple `require` and `ensure` clauses are allowed. They evaluate in source order. Each clause is independent — the first failure triggers TPOE.
  9. `old expr` is available in `ensure` clauses as specified by D129.
  10. `require`/`ensure` expressions must be pure — they may not call functions with side effects, mutate state, or consume linear values. They are observation-only.
  11. Every subexpression of a `require` or `ensure` clause uses the same value and arithmetic semantics as ordinary runtime code, including D75 overflow behavior.
  12. Overflow during contract evaluation triggers TPOE. It is not reinterpreted as "precondition false" or "postcondition false."
  13. There are no type-level `invariant` clauses. Contracts are per-function only. Type invariants interact badly with linearity — a mutable borrow that temporarily violates an invariant during mutation would TPOE mid-operation.
  **Why this fits Kyokai**: Borretti praised Design by Contract as a "mechanical process" for safety that is "independent of the skill of the programmer" but never implemented it in Austral. Kyokai closes this gap. `require`/`ensure` is the syntactic surface for TPOE — the contract is in the signature where it belongs, not buried in an `if` statement in the body. Combined with linear types (compile-time lifecycle contracts) and TPOE (runtime value contracts), Kyokai provides two complementary contract systems covering the full space of program correctness.
  **[DECIDED: D53/D140/D142 → `require`/`ensure` clauses on functions; `result` is a read-only postcondition view of the return slot; `old` for pure entry-state Free expressions; contracts always checked; ordinary arithmetic semantics apply inside contracts; no type invariants; contracts visible in `.kyo` interfaces]**
- **`old` snapshots pure entry-state expressions of `Free` data for use in `ensure` clauses** — Kyokai supports relational postconditions without violating linearity by restricting snapshots to values that are copyable and observable at function entry.
**Rules**:
  1. `old expr` is legal only inside an `ensure` clause.
  2. `expr` must be pure and must depend only on values that are available at function entry.
  3. Every value observed by `expr` must have `Free` universe type. If `expr` directly or indirectly observes a `Linear` value, the use of `old` is a compile-time error.
  4. Each distinct `old expr` is evaluated exactly once at function entry before any `require` clause is checked, and the `ensure` clause later observes that saved entry-state value.
  5. `old` may be applied to any qualifying pure entry-state expression, not only to bare parameter names.
  **Why this fits Kyokai**: common relational contracts such as "result is greater than the input" become expressible, while linear values remain uncopyable and the entry-state snapshot model stays fully explicit.
  **[STAGE: DECIDED_CORE_SEMANTICS | D129 → `old expr` in `ensure` for pure entry-state `Free` expressions only]**
- **Container mutation invalidation is mostly unrepresentable in safe code; remaining cases are specified per container** — the linear type system prevents the most dangerous invalidation scenarios statically. D77 formalizes what the borrow checker already does and fills the gap for unsafe internals.
**Rules**:
  1. The borrow checker prevents most invalidation statically: a mutable borrow `&![Container]` required for mutation excludes all immutable borrows `&[Container]`, element borrows `&[Element]`, and iterator borrows from coexisting. This is not a D77-specific rule — it is the borrow checker doing its job.
  2. Iterator objects that borrow the container immutably (`&[Container]`) prevent mutation for the duration of iteration. The programmer cannot call `push`, `remove`, or any `&![Container]` operation while an iterator is alive. This is a compile-time guarantee.
  3. Any operation that can reallocate internal storage (e.g., growing a `Vec` past capacity) invalidates all raw `Address[T]` values derived from the previous storage. This only matters in `pragma Unsafe_Module` code — safe code cannot obtain raw addresses from containers.
  4. Safe collection APIs prefer returning element borrows (`&[T]`) rather than raw addresses (`Address[T]`). The borrow system then prevents invalidation naturally.
  5. Each standard library collection must include an explicit invalidation table documenting which operations reallocate, which operations preserve element addresses, and what guarantees (if any) hold for `Address[T]` values across operations.
  6. The invalidation table is part of the D85 semantic contract for each collection and appears in the collection's documentation.
  **Why this fits Kyokai**: in most systems languages, iterator invalidation is a class of bugs discovered at runtime (C++) or prevented by the borrow checker with complex lifetime rules (Rust). Kyokai's simpler borrow model (`&[T]` / `&![T]` with no lifetime parameters in the common case) makes the most dangerous scenarios statically impossible, and D77 requires the remaining edge cases to be documented explicitly per container rather than left to programmer folklore.
  **[STAGE: DECIDED_CORE_SEMANTICS | D77 → borrow checker prevents most invalidation statically; remaining cases specified per container in D85 invalidation tables; safe APIs prefer borrows over raw addresses]**
- **The core language spec and a companion toolchain spec are separate normative documents** — Kyokai's specification is split into two documents, both normative, covering different domains of specified behavior.
**Core language spec** (the Kyokai spec under `kyokaispec/`, forked from inherited Austral spec material where useful):
  1. Syntax and grammar
  2. Type system (linear types, universes, borrowing, regions)
  3. Evaluation semantics (D71, D75, D76)
  4. Memory model (D73, D42)
  5. Control flow (`if`/`while`/`case`, `defer`, `panic`, TPOE)
  6. Module system (imports, visibility, interfaces/bodies)
  7. FFI rules (D20/D20a/D20b)
  8. Runtime termination contract (D84)
  9. Design by Contract semantics (D53)
  10. Built-in types, operators, and language-level constructs (`debug`, `panic`, `comptime`)
  **Companion toolchain spec** (new document):
  1. `kyokai.toml` manifest schema (D78, D31)
  2. Package resolution and workspace rules (D78)
  3. `.koi` artifact format (D79)
  4. Build profiles, target toolchains, and supported C-backend compiler contracts (D31, D80, D139, D141)
  5. Incremental and separate compilation behavior (D144)
  6. CLI tool subcommands and behavior (D26)
  7. Diagnostics JSON schema (D29)
  8. Formatter behavior (D25)
  9. LSP behavior (M07)
  10. Documentation generator behavior (M08)
  11. Test harness behavior (D28, D137)
  12. Target support tier contract (D80)
  13. Reproducible build requirements (D83)
  **Rules**:
  1. Both documents are normative. "Toolchain spec" does not mean "optional" or "implementation-defined."
  2. A topic must not be omitted merely because it "belongs to tooling" if user-visible behavior depends on it.
  3. The core language spec is the source of truth for what a conforming Kyokai compiler must accept and reject. The toolchain spec is the source of truth for what a conforming Kyokai toolchain must do beyond compilation.
  4. Accepted behavior belongs in the normative language or toolchain spec, not in informal notes or discussion artifacts.
  **Why this fits Kyokai**: the language is explicit about everything. The specification should be explicit about what it specifies. Mixing "what does `let` mean" with "what does `kyokai build --profile release` do" in one document creates a specification that is hard to reference and hard to implement against. Splitting them keeps both focused and both normative.
  **[STAGE: DECIDED_CORE_SEMANTICS | D86 → split into core language spec (syntax, types, evaluation, memory, control flow, FFI, runtime) and companion toolchain spec (manifest, packages, artifacts, CLI, diagnostics, formatter, LSP, testing, targets, reproducibility); both normative]**
- **Every standard library module must specify the same set of semantic contract fields** — "the language is explicit" is not enough if the standard library remains implicit. Each public API in the stdlib must document its behavior in a fixed set of categories so that programmers, documentation generators, and tooling can rely on uniform contract coverage.
**Required contract fields per API**:

  | Field                                        | What it specifies                                                                             | Example                                                                                                                         | Governing decision      |
  | -------------------------------------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
  | **Allocation behavior**                      | Whether the operation allocates, which allocator it uses, whether allocation can fail         | "Allocates via the provided `Allocator`; returns `AllocError` on failure"                                                       | D44, D74                |
  | **Capability requirements**                  | Which capabilities the operation requires                                                     | "Requires `TerminalCapability`" or "No capability required"                                                                     | D45, D67                |
  | **Ownership/borrowing behavior**             | Whether inputs are consumed, borrowed immutably, or borrowed mutably; what the caller retains | "Consumes `value` (linear move); borrows `sender` mutably"                                                                      | D11b naming conventions |
  | **Failure mode**                             | Which failure mechanism the operation uses                                                    | "`Result[T, ParseError]`", "`Optional[T]`", "TPOE on bounds violation", "`panic` on impossible state", "Cannot fail"            | D84, D53                |
  | **Complexity class**                         | Time and space complexity where relevant                                                      | "O(1) amortized, O(n) worst case on reallocation"                                                                               | —                       |
  | **Determinism / iteration-order guarantees** | Whether output depends on insertion order, hash seed, platform, etc.                          | "Iteration order matches insertion order" or "Iteration order is unspecified"                                                   | D83                     |
  | **Invalidation behavior**                    | Which operations invalidate borrows, iterators, views, or raw addresses                       | "Reallocation invalidates all `Address[T]` derived from previous storage"                                                       | D77                     |
  | **Concurrency behavior**                     | Whether the type is safe for cross-task access and under what conditions                      | "`Linear` — single owner, no cross-task sharing. Use channels to transfer." or "`Atomic[T]` — shared access via `&[Atomic[T]]`" | D3, D3a, D3b            |
  | **Platform-specific caveats**                | Any behavior that varies by target or OS                                                      | "On Linux, uses `epoll`; on other targets, uses `poll`"                                                                         | D80, D64                |

  **Rules**:
  1. Every public function, method, and typeclass instance in the standard library must specify all applicable fields from the table above.
  2. Fields that do not apply to a given API are explicitly marked "N/A" rather than omitted.
  3. These fields appear in `///` doc comments in `.kyo` interface files and are extracted by the documentation generator (M08).
  4. The contract fields are the documentation form of D53's `require`/`ensure` — where a contract can be expressed as a `require`/`ensure` clause, it should be. Where it cannot (e.g., complexity guarantees, iteration order), it is documented in the contract fields.
  5. The contract fields are part of the toolchain spec (D86) rather than the core language spec, because they govern library documentation requirements, not language semantics.
  6. User libraries are encouraged but not required to follow the same contract format.
**Why this fits Kyokai**: the standard library is the largest body of code most Kyokai programmers will interact with. If its behavior is documented ad-hoc — some functions mention allocation, some don't, some mention failure modes, some leave them implicit — then the library contradicts the language's core promise. Uniform contract fields make the stdlib as explicit as the language itself.
**[STAGE: DECIDED_CORE_SEMANTICS | D85 → every stdlib API specifies allocation, capabilities, ownership, failure mode, complexity, determinism, invalidation, concurrency, and platform caveats; fields appear in `.kyo` doc comments; part of toolchain spec (D86)]**
- **Kyokai does not grow a general effect system beyond capabilities, explicit control-flow typing, and explicit API contracts** — the language chooses one narrow effect-tracking mechanism and makes the rest visible through names and documentation rather than through a second type layer.
**Rules**:
  1. Kyokai has no general effect rows, checked exception system, algebraic-effect surface, or `async`-style effect coloring in function types.
  2. The type system tracks external authority through capability parameters and capability-bearing handle types only.
  3. The type system does not track divergence, non-termination, blocking, scheduler interaction, transitive allocation, or "may TPOE."
  4. D58's `Never` models only control-flow paths that are statically known not to complete normally. It is not a general effect marker.
  5. Effects not tracked in types must instead be surfaced through explicit API naming and explicit contract documentation. This includes at minimum capability requirements, allocation behavior, failure mode, blocking/concurrency behavior, and determinism where relevant.
  6. D85's contract fields and the companion toolchain spec under D86 are the normative place where these non-capability effect facts are made explicit for the standard library and toolchain surfaces.
  **Why this fits Kyokai**: the language keeps the one effect boundary it truly cares about in the type system, while refusing to smuggle a broad secondary effect calculus into signatures and generic constraints.
  **[STAGE: DECIDED_CORE_SEMANTICS | D211 → capabilities are the only built-in effect-tracking mechanism; divergence/allocation/blocking/TPOE are not type-tracked and must be surfaced by explicit naming and contracts]**
- **Kyokai has a backend-independent `asm` block syntax for inline assembly; it lives inside `pragma Unsafe_Module` and targets the instruction set, not the backend** — inline assembly is inherently unsafe and ISA-specific, but it should not be backend-specific. The syntax works with both the C backend (emitting `__asm__` in generated C) and the future LLVM backend (emitting LLVM inline asm constraints).
**Syntax**:
  ```kyokai
  // Only legal inside pragma Unsafe_Module
  asm(target.arch == x86_64,
      "syscall",
      out("rax") result,
      in("rax") sysNumber,
      in("rdi") arg1,
      in("rsi") arg2,
      clobber("rcx", "r11", "memory")
  );
  ```
  **Rules**:
  1. `asm(...)` is only legal inside `pragma Unsafe_Module`. Using it in safe code is a compile error.
  2. The first argument is a comptime `Bool` guard — typically `target.arch == x86_64` or similar (D19). If the guard is false, the block is not compiled. This prevents ISA-mismatch errors at compile time.
  3. The assembly string is a string literal in the target ISA's syntax (AT&T for x86, standard for ARM/RISC-V).
  4. Operand bindings use named forms: `in("reg") expr`, `out("reg") binding`, `inout("reg") binding`, `clobber("reg1", "reg2", ...)`.
  5. The C backend translates this to GCC-style `__asm__ volatile(...)` with appropriate constraints.
  6. The LLVM backend translates this to LLVM inline asm with appropriate constraints.
  7. The compiler does not attempt to understand or optimize assembly contents. It trusts the programmer's operand and clobber declarations.
  8. `asm` blocks do not participate in linearity checking — they are outside the safe language model. The surrounding unsafe module is responsible for correctness.
  **Why not defer to FFI only (old Option D)**: D3b (atomics) proved that pushing core primitives to FFI loses type safety and compiler visibility. For the tiny number of functions that genuinely need inline asm (syscall wrappers, specific hardware instructions), a proper `asm` syntax is better than maintaining separate `.c` shim files. The syntax is designed once, works with both backends.
  **[STAGE: DECIDED_CORE_SEMANTICS | D22 → backend-independent `asm` block syntax; `pragma Unsafe_Module` only; comptime ISA guard; named operand bindings; C backend emits `__asm__`, LLVM backend emits LLVM inline asm]**
- **SIMD is a portable language feature with an explicit split between portable vectors and ISA-specific intrinsics** — Kyokai does not defer vector programming until the LLVM backend exists, and it does not define SIMD as "whatever intrinsics the current backend happens to expose."
**Core type**: `Vector[T, N]`, where `T` is a fixed-width integer type, a fixed-width floating-point type, or `Bool` for mask vectors, and `N` is a positive comptime lane count.
**Rules**:
  1. `Vector[T, N]` is a first-class value type. Lane numbering is explicit and stable: lane `0` is the first source-order lane and lane `N - 1` is the last.
  2. The portable SIMD contract is value-level and lane-wise. Arithmetic, bitwise, comparison, lane extraction/replacement, shuffles, and explicitly named reductions are defined in terms of the vector's abstract lanes, not in terms of a particular instruction selection strategy.
  3. A portable vector operation exists only when the corresponding scalar operation is already defined for `T`. Kyokai does not invent vector-only arithmetic rules that disagree with the scalar language.
  4. Conversion between `Vector[T, N]` and `Array[T, N]` is explicit and preserves lane order exactly. The portable core does not introduce hidden alignment requirements, aliasing rules, or address-taking semantics.
  5. Portable vector operations may lower to hardware SIMD, compiler vector extensions, generated helper code, or scalar code. All are conforming if the defined lane semantics are preserved.
  6. The C backend may implement the portable core with compiler intrinsics/extensions or scalarization. The LLVM backend may implement it with native vector IR. Backend choice does not change the language semantics.
  7. ISA-specific operations are separate from the portable core and live in explicit target-gated modules or declarations. Using them requires the selected target contract to declare the required ISA and CPU-feature baseline explicitly.
  8. If an ISA-specific intrinsic is unavailable for the selected backend, target, or declared CPU-feature baseline, compilation fails. ISA-specific operations may NOT silently scalarize or degrade into some other operation family.
  9. Runtime host CPU-feature detection is explicit library surface. If a program ships multiple ISA-specialized implementations, the dispatch between them must be written explicitly in source or through an explicit standard-library dispatch combinator. Kyokai performs no hidden auto-multiversioning.
  10. Any selected CPU-feature baseline used to enable ISA-specific modules is part of the target contract for D79/D83 purposes. It must be recorded explicitly so `.koi` compatibility and reproducible builds remain auditable.
  **Why this fits Kyokai**: performance-critical code gets a real portable vector model now, while backend-specific power stays honest, explicit, and separate from the language's semantic core.
  **[STAGE: DECIDED_CORE_SEMANTICS | D104 → portable `Vector[T, N]` core with explicit lane semantics; ISA-specific intrinsics are target-gated, explicit, and never silently scalarized]**
- `**kyokai fmt` is an opinionated, zero-configuration code formatter built into the `kyokai` CLI** — there is one canonical formatting style for Kyokai code. It is not configurable. All Kyokai code looks the same.
**Rules**:
  1. `kyokai fmt` formats all `.kyo` and `.kai` files in the project.
  2. The formatter has zero configuration. There are no `kyokai.toml` options for indent width, line length, brace style, or any other formatting preference.
  3. Output is deterministic — the same input always produces the same output, regardless of platform or compiler version.
  4. The formatter is idempotent — running it twice produces the same result as running it once.
  5. `kyokai fmt --check` exits with a non-zero status if any file would be changed, without modifying files. Suitable for CI.
  **Formatting choices** (part of the toolchain spec, D86):
  1. Indentation: 4 spaces. No tabs.
  2. Line length: 100 characters.
  3. No vertical alignment (breaks on rename refactors).
  4. Trailing commas required in multiline lists.
  5. One blank line between top-level declarations.
  6. No trailing whitespace.
  **Why zero configuration**: Go proved it. Python (Black) proved it. Zig proved it. Style debates are a solved problem — the solution is removing the debate. Kyokai already makes opinionated choices everywhere (explicit types, keywords over symbols, linear consumption). The formatter follows the same principle: one right way.
  **[STAGE: DECIDED_CORE_SEMANTICS | D25 → opinionated `kyokai fmt`; zero configuration; 4-space indent; 100-char line; deterministic and idempotent; part of toolchain spec (D86)]**
- **Debugging support uses `#line` directives in generated C for source-level debugging today; the LLVM backend will emit DWARF directly** — the C backend is the current reality and `#line` gives immediate GDB/LLDB support. The LLVM backend is the future and will emit proper debug info natively.
**C backend rules**:
  1. The C backend emits `#line N "path/to/File.kai"` directives in generated `.c` files, mapping generated C lines back to Kyokai source lines.
  2. When the programmer runs GDB or LLDB on the compiled binary, breakpoints, single-stepping, and stack traces show Kyokai source file names and line numbers.
  3. The `debug` profile (D31) compiles with `-g` (or equivalent) to include debug symbols.
  4. Variable names in generated C should preserve Kyokai variable names where possible for debugger inspection.
  **LLVM backend rules** (future):
  1. The LLVM backend will emit DWARF debug information directly, mapping LLVM IR to Kyokai source locations.
  2. This provides full source-level debugging without the C intermediary.
  3. The debug info format and level of detail are part of the toolchain spec (D86).
  **Why `#line` is sufficient for now**: `#line` is standardized C (ISO C99 §6.10.4). Every C compiler supports it. GCC and Clang both propagate `#line` information into DWARF debug info. This means Kyokai gets source-level debugging for free through the C backend — no custom debugger needed. It's not perfect (complex expressions may not map cleanly), but it's immediately useful.
  **[STAGE: DECIDED_CORE_SEMANTICS | D27 → C backend uses `#line` directives for source-level debugging; LLVM backend will emit DWARF directly; debug profile compiles with `-g`]**
- **Tests are inline `test` blocks in module bodies, discovered and run by `kyokai test`** — Zig-style co-located test blocks. Tests live with the code they test, have access to private declarations, and are excluded from production builds.
**Syntax**:
  ```kyokai
  module body Kyokai.Math.Int is
      function abs(x: Int32): Int32 is
          if x < 0 then return 0 - x; fi;
          return x;
      qed;

      test "abs of positive" is
          assert(abs(5) == 5);
      qed;

      test "abs of negative" is
          assert(abs(-5) == 5);
      qed;

      test "abs of zero" is
          assert(abs(0) == 0);
      qed;
  seal;
  ```
  **Rules**:
  1. `test "description" is ... qed;` blocks appear inside module bodies, at the same level as function declarations.
  2. Test blocks are compiled only when `kyokai test` is invoked. Normal `kyokai build` excludes them entirely — no test code in production binaries.
  3. Test blocks have access to the module's private declarations because they are inside the module body.
  4. Test blocks do not appear in `.kyo` interface files.
  5. Each test block runs independently. A failed assertion in one test does not prevent other tests from running.
  6. `kyokai test` discovers all `test` blocks in the project, compiles with tests included, runs each test, and reports pass/fail/skip with test names and source locations.
  7. `kyokai test "pattern"` filters tests by name substring.
  **Assertions** (built into `Kyokai.Test`, auto-available in test blocks):
  1. `assert(condition: Bool)` — TPOE if false.
  2. `assertEqual(left: T, right: T)` — TPOE if not equal. Requires `Equality` typeclass. Reports both values on failure.
  3. `assertErr(result: Result[T, E])` — TPOE if not `Err`.
  4. `assertOk(result: Result[T, E])` — TPOE if not `Ok`.
  5. Test assertion failure is caught by the test runner — it does not terminate the entire test suite (unlike ordinary TPOE). The test runner treats assertion failure as a test failure, reports it, and continues to the next test.
  **Why inline over separate files**: tests co-located with code are more likely to be written and maintained. Access to private declarations enables unit testing of internal functions without exposing them publicly. This is the Zig model, and it works.
  **[STAGE: DECIDED_CORE_SEMANTICS | D28 → inline `test` blocks in module bodies; `kyokai test` discovers and runs them; excluded from production builds; private access; `Kyokai.Test` assertions; part of toolchain spec (D86)]**
- **Tests request capabilities explicitly; there is no ambient test-only root or hidden harness power** — pure tests stay capability-free, while effectful tests must spell out that they want authority.
**Syntax**:
  ```kyokai
  test "pure arithmetic" is
      assert(add(2, 2) == 4);
  qed;

  test "reads env" with (root: RootCapability) is
      let env := root.env();
      let home := getEnv(&!env, "HOME");
      assert(home.isSome());
  qed;
  ```
  **Rules**:
  1. Pure tests use the ordinary form `test "description" is ... qed;` and receive no capabilities.
  2. A capability-using test must declare a `with` clause explicitly. The standardized authority-bearing form is `with (root: RootCapability)`.
  3. There is no ambient `root`, no implicit `TestCapability`, and no hidden test-only authority channel.
  4. The test runner constructs a fresh `RootCapability` for each individual test execution.
  5. The runner may realize this by subprocess-per-test or another harness strategy with equivalent semantics, but the contract remains "fresh root per test."
  6. A capability-using test derives `TerminalCapability`, `EnvCapability`, filesystem capabilities, and any other authority from `root` exactly as ordinary code derives them in `main`.
**Why this fits Kyokai**: integration tests stay practical, pure tests stay lightweight, and the capability model does not grow an invisible exception for test code.
**[STAGE: DECIDED_CORE_SEMANTICS | D137 → tests default to no capabilities; capability-using tests spell `with (root: RootCapability)`; runner provides a fresh root per test; no ambient test-only authority]**
- **`kyokai doc` is an official interface-driven documentation generator with first-class contract rendering and doc-test extraction** — Kyokai's doc surface is not an afterthought because the language already depends on explicit `///`, `//!`, `require`, `ensure`, and D85 contract fields.
**Rules**:
  1. `kyokai doc` is an official toolchain command.
  2. By default it reads package `.kyo` interface files plus module/file `//!` docs and generates documentation for the public interface surface only.
  3. `internal` declarations may be included only through an explicit package-internal documentation mode. Private `.kai`-only declarations are never part of public generated docs.
  4. Generated item pages must render at least: declaration signature, visibility, generic constraints, associated types, `require`/`ensure` clauses, and all applicable D85 contract fields.
  5. The official output formats are static HTML and machine-readable JSON.
  6. Cross-reference resolution is explicit and declaration-based. Local `[Name]` and qualified `[Module.Name]`-style symbol links resolve through the documented interface graph rather than through best-effort text search.
  7. `kyokai test --doc` extracts fenced `kyokai` code blocks from documentation comments and module docs under the toolchain's explicit doc-test rules.
  8. Doc-test snippets are ordinary Kyokai code, not a separate documentation mini-language.
  9. Capability-using doc examples remain bound by D137's explicit-authority rules. The documentation toolchain does not invent hidden authority for examples.
  **Why this fits Kyokai**: the interface file already carries the explicit semantic material. A real documentation generator makes that material visible without creating a second folklore channel for contracts and API behavior.
  **[STAGE: DECIDED_CORE_SEMANTICS | D218 → official `kyokai doc`; public-interface-driven HTML and JSON output; first-class rendering of contracts/contract fields; explicit doc-test extraction via `kyokai test --doc`]**
- **Coverage is a first-class toolchain feature, but it is reported in Kyokai source terms rather than backend artifact terms** — generated C or LLVM IR may implement the program, but the coverage contract belongs to the source language the programmer wrote.
**Rules**:
  1. `kyokai test --coverage` is an official toolchain mode.
  2. Coverage is reported against Kyokai source files and Kyokai source spans, not generated C, LLVM IR, or backend helper files.
  3. The toolchain must not count backend-generated scaffolding such as overflow-check lowering, helper wrappers, or other codegen artifacts as user-visible Kyokai coverage points.
  4. Minimum outputs are a terminal summary and LCOV export.
  5. HTML coverage reports are provided through an explicit report flag.
  6. Line or statement coverage is required.
  7. Branch coverage is defined over explicit Kyokai control-flow constructs such as `if`, `case`, loop conditions, `select`, and explicit short-circuit or sugar-defined control-flow sites when the toolchain can map them faithfully.
  8. By default coverage reports the current package's source and excludes dependencies and generated shims unless the user explicitly requests otherwise.
  9. If a backend/target combination cannot provide conforming Kyokai-source-level coverage for the requested mode, the toolchain must fail explicitly rather than emit misleading coverage data.
  **Why this fits Kyokai**: coverage remains auditable at the language boundary the programmer actually reasons about, and backend choice is prevented from changing what the tool claims was or was not exercised.
  **[STAGE: DECIDED_CORE_SEMANTICS | D219 → official `kyokai test --coverage`; Kyokai-source-level reporting only; LCOV plus HTML reporting; explicit failure when faithful source mapping cannot be provided]**
- **Benchmarking is standardized as `kyokai bench`, but benchmark discovery syntax remains a toolchain-spec detail rather than a new core-language block form** — Kyokai standardizes the benchmark workflow and minimum measurement behavior without pretending that microbenchmark methodology belongs in the core grammar.
  **Rules**:
  1. The reference toolchain MUST provide a `kyokai bench` entry point.
  2. Benchmark execution runs in a dedicated harness that includes an explicit warm-up phase before measured iterations.
  3. Measured execution uses repeated iterations rather than one-shot timings and reports aggregate results in both human-readable and machine-readable forms.
  4. Benchmark code is not pulled into ordinary `kyokai build` unless the chosen discovery mechanism explicitly includes it.
  5. Exact benchmark discovery syntax and output schema belong to the toolchain spec (D86), not to Kyokai core syntax. The language does not currently standardize a `bench ... qed;` block.
  **Why this fits Kyokai**: performance work gets a real first-class workflow, but the language surface avoids a fake symmetry where timing methodology is crammed into syntax just because tests use inline blocks.
  **[STAGE: DECIDED_CORE_SEMANTICS | D136 → standardize `kyokai bench` with warm-up, repeated measurements, and human/machine-readable output; discovery syntax stays toolchain-spec detail]**
- **Compiler diagnostics follow the Rust-quality model: source spans, underline markers, diagnostic codes, suggestions, and structured JSON output** — every diagnostic has a code, a source span, context lines, and optionally a suggested fix. Error messages are written for humans, structured output is available for tools.
**Every diagnostic MUST include**:
  1. **Severity**: `error` (compilation fails), `warning` (compilation succeeds), `note` (additional context).
  2. **Diagnostic code**: `KYO-E0001` for errors, `KYO-W0001` for warnings. Codes are stable across compiler versions.
  3. **Source span**: File path, start line:column, end line:column.
  4. **Source context**: 1-3 lines of source code with underline markers (`^^^`).
  5. **Message**: Human-readable description of the problem.
  6. **Suggestion** (optional): "did you mean X?" with the suggested fix.
  **Output modes**:
  1. **Terminal** (default): Colored output using ANSI codes. Respects `NO_COLOR` environment variable.
  2. **JSON** (`--diagnostics=json`): Machine-readable output for editor integration (LSP, M07).
  **Warning categories and suppression**:
  1. Warnings grouped by category: `naming` (D11b), `unused`, `linearity`, `style`.
  2. Per-project suppression in `kyokai.toml` `[warnings]` section.
  3. No per-line suppression comments (`// nolint`). If the compiler warns about it, fix it or suppress it project-wide.
  **Error recovery**: The compiler continues after the first error and reports as many independent errors as possible. Maximum error count before stopping is configurable (default: 20).
  **Example**:
  ```
  error[KYO-E0042]: linear variable `file` not consumed
    --> src/Fetch.kyo:42:9
     |
  42 |     let file: File := openFile(path);
     |         ^^^^ variable declared here
     |
     = note: linear variables must be consumed exactly once
     = help: consider adding `defer closeFile(file);` after the declaration
  ```
  **Why Rust-quality is the target**: Elm proved that good error messages change how people feel about a language. Rust proved it scales to a complex type system. Kyokai's linear type system will produce errors that are unfamiliar to most programmers — clear diagnostics with suggestions are essential for adoption. The diagnostic codes (`KYO-E0042`) enable `kyokai explain KYO-E0042` for detailed documentation of each error, which is especially important for linearity errors.
  **[STAGE: DECIDED_CORE_SEMANTICS | D29 → Rust-quality diagnostics; source spans; diagnostic codes; suggestions; JSON output; per-project warning suppression; no per-line suppression; part of toolchain spec (D86)]**
- **Iterator adapters are a minimal set of eager, allocation-explicit collection helpers and stateless iteration helpers — no lazy adapter chains** — Even with D118's explicit-capture closure literals, Kyokai rejects Rust-style lazy adapter stacks and instead provides explicit helpers that keep allocation, ownership, and control flow visible.
**Collection-building helpers** (eager, allocation-explicit):
  ```kyokai
  // mapInto: applies f to each element of source, pushes results into dest
  function mapInto[T, U, F](
      source: &[Span[T]], dest: &![Buffer[U]], f: F
  ): Unit
      where F: Callable[&[T], U];

  // filterInto: pushes elements satisfying predicate into dest
  function filterInto[T, F](
      source: &[Span[T]], dest: &![Buffer[T]], f: F
  ): Unit
      where F: Callable[&[T], Bool];
  ```
  **Search helpers** (no allocation, short-circuit on first match):
  ```kyokai
  function find[T, F](iter: &![Iterator[T]], predicate: F): Optional[T]
      where F: Callable[&[T], Bool];

  function findIndex[T, F](iter: &![Iterator[T]], predicate: F): Optional[Index]
      where F: Callable[&[T], Bool];
  ```
  **Iteration helpers** (no allocation, thin wrapper iterators):
  ```kyokai
  function enumerate[I: Iterator](iter: I): EnumerateIterator[I];
  function zip[A: Iterator, B: Iterator](a: A, b: B): ZipIterator[A, B];
  ```
  **Reduction helpers** (no allocation, consume the iterator):
  ```kyokai
  function fold[T, Acc, F](iter: &![Iterator[T]], initial: Acc, f: F): Acc
      where F: Callable2[Acc, T, Acc];

  function any[T, F](iter: &![Iterator[T]], predicate: F): Bool
      where F: Callable[&[T], Bool];

  function all[T, F](iter: &![Iterator[T]], predicate: F): Bool
      where F: Callable[&[T], Bool];

  function count[T](iter: &![Iterator[T]]): Index;
  ```
  **Rules**:
  1. No lazy adapters. All collection-building helpers are eager and take an explicit destination container.
  2. `mapInto`/`filterInto` take a `&![Buffer[U]]` destination — allocation is the caller's responsibility. There is no `collect()` that magically picks the right container type.
  3. `find` and `findIndex` are ordinary short-circuiting library helpers. They solve the common search pattern without making loops expression-typed and without introducing dedicated search syntax.
  4. `enumerate`/`zip` return thin wrapper iterators with no allocation. They satisfy `Iterator` and work with `for-in`.
  5. `fold`/`any`/`all`/`count` are reductions that consume the iterator.
  6. These are ordinary library functions in `Kyokai.Iter`, not typeclass methods on `Iterator`.
  7. Callbacks use the D21/D126 callable family — named functions, `FnPtr` values, or D118 closure literals lowered to the matching family member.
  8. For most transformations, an explicit `for` loop with `push` is the idiomatic Kyokai pattern. The helpers exist for cases where the loop body is a single function application or a straightforward search predicate.
  **Why no lazy adapters**: lazy adapters need closures to be ergonomic. Without closures, `map(filter(iter, isEvenFn), doubleFn)` is inside-out and unreadable. Explicit loops are clearer, and Kyokai's `for-in` + `while let` cover the iteration patterns that lazy adapters serve in other languages.
  **[STAGE: DECIDED_CORE_SEMANTICS | D32a → minimal eager helpers (`mapInto`, `filterInto`, `find`, `findIndex`) with explicit destinations where applicable; stateless iteration wrappers (`enumerate`, `zip`); reductions; no lazy adapters; callbacks via the D21/D126 callable family]**
- `**format` is the fallible allocating interpolation path, using comptime-checked `{}` placeholders only** — no width/radix/alignment mini-language. The format string is parsed at compile time to verify argument count and `Displayable` constraints.
**Syntax**:
  ```kyokai
  let msg := format(&!heap, "x = {}, y = {}", x, y) or return;

  // Comptime checks:
  // 1. Number of {} matches number of trailing arguments
  // 2. Each argument satisfies Displayable (D40)
  // 3. Mismatches are compile errors with KYO-E diagnostic codes
  ```
  **Rules**:
  1. `format(alloc, template, args...)` is a built-in construct (like `debug` and `panic`), not a library function. The compiler parses the template string at comptime.
  2. The template string uses `{}` as the only placeholder. No positional arguments (`{0}`), no format specifiers (`{:04x}`), no named arguments (`{name}`).
  3. Comptime validation: the number of `{}` placeholders must exactly match the number of trailing arguments. Each argument must satisfy `Displayable` (D40). Mismatches are compile errors.
  4. `format` requires an allocator (D44) because it produces a heap-allocated `String`, and it follows D74: allocation failure returns `Err(AllocError)` rather than terminating implicitly.
  5. `format` is specified in terms of the D40 sink model: create a `StringBuilder`, render through the shared formatting engine, then finish into a `String`. Intermediate partial builder state is not externally observable on failure.
  6. If the standard library provides a fatal-on-OOM convenience form, it must be explicitly named (for example `mustFormat`) and follow D74's runtime-fatal naming rule.
  7. For hex, padding, width, or alignment — use explicit named conversions: `x.toHex()`, `padLeft(s, 10, ' ')`. These are ordinary functions, not mini-language specifiers embedded in the format string.
  8. `{{` produces a literal `{` in output. `}}` produces a literal `}`. This is the only escape.
  9. A richer formatting mini-language (width, radix, alignment, debug representation) is explicitly deferred to a future decision point if demand warrants it.
  **Why minimal**: every formatting specifier is a mini-language feature that must be parsed, validated, documented, and maintained. Rust's `format!` supports `{:>10.2}` — that's a significant embedded DSL. Kyokai's philosophy is "no hidden complexity." If you need formatted numeric output, the formatting function is named and visible in source (`x.toHex()`, `padLeft(...)`), not hidden inside a format string that only the compiler can parse.
  **[STAGE: DECIDED_CORE_SEMANTICS | D40a → built-in `format(alloc, template, args...) -> Result[String, AllocError]` with comptime-checked `{}` only; explicit D74-aligned allocation failure; richer DSL explicitly deferred]**
- **Non-allocating formatted output uses `writeFmt` over the same placeholder language, with explicit stream-failure semantics** — Kyokai provides a direct formatted-output path for streams without forcing an intermediate `String` allocation.
**Syntax**:
  ```kyokai
  writeFmt(&!stream, "x = {}, y = {}", x, y) or return;
  stream.writeFmt("x = {}, y = {}", x, y) or return;
  ```
  **Rules**:
  1. `writeFmt(&!stream, template, args...)` is a built-in construct for any `Writable` target. The UFCS spelling `stream.writeFmt(...)` is ordinary D7a sugar.
  2. `writeFmt` uses the same `{}` placeholder grammar, comptime placeholder-count validation, and `Displayable` constraints as D40a.
  3. `writeFmt` performs no whole-result allocation. It renders each argument through the D40 `Displayable` model and writes the resulting UTF-8 chunks directly to the target stream.
  4. `writeFmt` is specified through a transient formatting-sink adapter over the `Writable` target. That adapter uses `writeAll`-style behavior internally so `Displayable` implementations see whole-chunk-or-error semantics rather than D66 partial writes.
  5. `writeFmt` returns `Result[Unit, IoError]`. Any underlying write/flush failure is surfaced explicitly.
  6. `writeFmt` is non-transactional. If a failure occurs after some output was already accepted by the underlying stream, that written prefix remains written; `writeFmt` does not buffer the entire result merely to provide rollback.
  7. After a `writeFmt` failure, the stream remains usable unless that stream type's own contract says otherwise.
  8. `writeFmt` does not change D66's byte-oriented I/O model; it is a formatting layer above `Writable`, not a second ambient text-stream system.
  9. `format` is the allocating path and `writeFmt` is the non-allocating path. They share one placeholder language and one `Displayable` rendering model.
  **Why this fits Kyokai**: CLI programs, logging, diagnostics, and text protocols get an honest zero-allocation formatting path, while allocation failure and I/O failure remain distinct and explicit rather than being collapsed into one hidden mechanism.
  **[STAGE: DECIDED_CORE_SEMANTICS | D102 → built-in `writeFmt` over `Writable`; shared placeholder language with D40a; explicit `IoError`; non-transactional prefix-preserving failure semantics]**
- **Error context is a concrete standard-library wrapper, not a language feature or ambient error stack** — Kyokai allows ergonomic explanatory wrapping at layer boundaries, but it keeps the mechanism explicit, concrete, and type-directed.
**API shape**:
  ```kyokai
  record ContextError[E: Type] is
      message: String;
      inner: E;
  build;

  function context[T, E](result: Result[T, E], message: String): Result[T, ContextError[E]];

  let cfg := openFile(path).context("failed to open config") or return;
  ```
  **Rules**:
  1. `ContextError[E]` is a standard-library type, not a language-level error feature.
  2. `.context(message)` is ordinary UFCS over a library function from `Result[T, E]` to `Result[T, ContextError[E]]`. Kyokai adds no special syntax beyond UFCS.
  3. The message is an explicit value supplied by the caller. The helper consumes that value and, on `Err(e)`, returns `Err(ContextError { message, inner: e })`.
  4. The helper performs no implicit conversion of the surrounding function's error type. If the caller wants to propagate the result directly, the surrounding return type must explicitly be `Result[..., ContextError[E]]` or the caller must apply an explicit D119 mapping.
  5. There is no trait-object erasure, ambient source-chain registry, implicit backtrace attachment, or hidden error-conversion machinery.
  6. Nested context remains concrete and explicit: repeated wrapping yields types such as `ContextError[ContextError[E]]`.
  7. `ContextError[E]` implements `Displayable` when `E` implements `Displayable`, rendering the outer message and then the wrapped inner error.
  8. `ContextError` is for layer-boundary explanation, not Kyokai's universal error architecture. D119-style explicit mapping remains the primary mechanism for domain error conversion.
  **Why this fits Kyokai**: the common "add human context and propagate" case becomes ergonomic without introducing dynamic-error folklore, hidden conversions, or a second error model separate from `Result`.
  **[STAGE: DECIDED_CORE_SEMANTICS | D103 → concrete `ContextError[E]` + `.context(message)` UFCS helper; no implicit conversions or dynamic error machinery]**
- **Errors have an explicit standard-library classification typeclass, but Kyokai does not adopt a dynamic error-object system** — generic APIs can talk about "error types" directly, while rendering, wrapping, and transport remain separate explicit mechanisms.
**Rules**:
  1. `Error` is a standard-library typeclass used to classify types intended to represent ordinary recoverable error domains.
  2. `Error` is a marker-style typeclass. It does not introduce hidden allocation, hidden backtraces, hidden source chains, implicit conversion, or dynamic error erasure machinery.
  3. Human-readable rendering remains the job of `Displayable`, not `Error`. A generic API that needs both "this is an error type" and "this can be rendered" must require both constraints explicitly.
  4. `ContextError[E]` from D103 implements `Error` whenever `E` implements `Error`.
  5. Kyokai has no `dyn Error`, trait-object error box, or existential error surface under this decision.
  6. If the standard library later provides typed source-chaining helpers, they must remain concrete and typed wrappers rather than erased runtime polymorphism.
  **Why this fits Kyokai**: generic code gains an explicit way to talk about error domains, but the language still rejects the dynamic "anything implementing Error can be boxed and passed around" model that would conflict with D82, D103, and D193.
  **[STAGE: DECIDED_CORE_SEMANTICS | D166 → standard-library marker `Error` typeclass; rendering stays with `Displayable`; no dynamic error-object system]**
- **Hashing uses a hasher-driven core protocol, with one-shot `Nat64` helpers kept explicit and algorithm-named** — the owner of the hash table or helper picks the algorithm; value types contribute structure, not an ambient default hash number.
**Core protocol**:
  ```kyokai
  typeclass Hashable(Self: Type) is
      method hash(self: &[Self], hasher: &![Hasher]): Unit;
  spec;
  ```
  **Rules**:
  1. `Hashable` is the core language-facing hashing protocol for user types and standard-library types.
  2. `Hasher` lives under `Kyokai.Hash` as the explicit state object that receives bytes and produces a final digest such as `Nat64`.
  3. Containers such as `HashMap` and `HashSet` choose their hashing algorithm through the concrete hasher they instantiate or carry; `Hashable` implementations do not hard-code one ambient `Nat64` algorithm.
  4. The standard library may provide explicit one-shot helpers tied to named algorithms, such as `SipHash24.hash64(&value)` or `WyHash.hash64(&value)`, but those are convenience APIs layered on top of the core `Hashable` protocol.
  5. There is no language-wide ambient `hash(value) -> Nat64` default that hides which algorithm was chosen.
  6. Standard `Hashable` instances exist for the built-in integer types, `Bool`, `String`, enums, and other standard-library key types.
  **Why this fits Kyokai**: algorithm choice stays visible where it matters, containers remain free to choose secure vs fast hashing explicitly, and callers still get explicit one-shot helpers when that is the actual goal.
  **[STAGE: DECIDED_CORE_SEMANTICS | D134 → hasher-based `Hashable`; explicit named one-shot hash helpers allowed, but no ambient default `hash() -> Nat64`]**
- **`Result` gets a small linearity-aware combinator surface based on one-shot callbacks** — Kyokai keeps explicit `case` and `let...else`, but it also standardizes the common single-branch transformations so error plumbing does not become needless boilerplate.
**Required surface**:
  - `map`
  - `mapErr`
  - `andThen`
  - `orElse`
  **Rules**:
  1. Each `Result` combinator consumes its input `Result` exactly once.
  2. `map` applies its callback only on the `Ok` branch and returns the `Err` branch unchanged.
  3. `mapErr` applies its callback only on the `Err` branch and returns the `Ok` branch unchanged.
  4. `andThen` applies its callback only on the `Ok` branch; that callback returns another `Result`.
  5. `orElse` applies its callback only on the `Err` branch; that callback returns another `Result`.
  6. These combinators are specified against the D21 `CallableOnce` substrate because each callback is invoked at most once.
  7. If the transformed branch payload is `Linear`, the callback consumes that payload explicitly in the ordinary way. The combinator does not duplicate it, drop it, or invent any hidden cleanup path.
  8. The untouched branch payload is preserved exactly and is not rewrapped in a second hidden envelope.
  **Why this fits Kyokai**: the library gets the ergonomics of standard `Result` transformation without giving up linear visibility or replacing `case`/`let...else` as the fully explicit escape hatch.
  **[STAGE: DECIDED_CORE_SEMANTICS | D167 → `Result` provides `map`/`mapErr`/`andThen`/`orElse`, specified in terms of one-shot callbacks and ordinary linear consumption]**
- **Generic dispatch is static and every concrete instantiation behaves as if a fully specialized body exists — the compiler may share or deduplicate identical bodies as a non-observable optimization** — this is the "as-if monomorphization" model. The language semantics are monomorphized; the implementation strategy has freedom within those semantics.
**Rules**:
  1. Every concrete use of a generic function or type behaves as if a fully specialized body was materialized with all type parameters resolved. This is the semantic contract.
  2. The compiler may share or deduplicate concrete bodies that are identical at the machine code level, provided this does not change observable behavior, `require`/`ensure` contract evaluation, debug identity (source locations, function names in stack traces), or diagnostic output.
  3. The compiler must NOT share bodies that differ in layout, alignment, `sizeOf`, `alignOf`, or any type-dependent property.
  4. There is no "shared generic body with runtime type info" mode. D82 explicitly rejected runtime dictionaries.
  5. The materialization strategy is an implementation detail of the compiler — the language contract is the "as-if" rule, not a mandate for how code is generated.
  **Why "as-if" rather than mandating full monomorphization**: mandating that every concrete use emits a fresh body would prevent legitimate size optimizations (merging `fn foo[T: Addable](x: T)` for `Int32` and `Int64` if they produce identical machine code). The "as-if" rule gives the compiler freedom while keeping the programmer's model simple: "my generic function is specialized for my types."
  **[STAGE: DECIDED_CORE_SEMANTICS | D82a → as-if monomorphization; every concrete instantiation behaves as if fully specialized; compiler may deduplicate identical bodies without changing observable behavior]**
- **Cross-package generic instantiation ownership is coordinated explicitly by the toolchain; `.koi` artifacts carry enough metadata for downstream materialization; the reference toolchain should avoid redundant materialization via workspace-level caching** — this separates the language contract (what `.koi` must carry) from the toolchain strategy (how to avoid redundant work).
**Language/artifact contract**:
  1. `.koi` artifacts (D79) carry serialized generic body metadata sufficient for a downstream package to materialize concrete bodies from upstream generics without re-parsing upstream source.
  2. The toolchain coordinates which compilation unit emits each concrete instantiation. The spec mandates that ownership is explicit and deterministic, but does not mandate caller-side or callee-side emission.
  3. Duplicate materialization of identical instantiations is legal but wasteful. The toolchain should avoid it.
  **Toolchain-level strategy** (part of toolchain spec, D86):
  1. The reference workspace build should avoid redundant materialization of identical generic instantiations when the required inputs and compatibility class match.
  2. A deterministic workspace-level instantiation cache is the preferred mechanism, keyed by `(package identity, generic identity, concrete type arguments, compiler compatibility class, target, profile)`.
  3. Reuse of cached materializations must be semantics-preserving and reproducible (D83).
  4. Cache invalidation tracks upstream generic body changes through `.koi` fingerprints.
  5. This strategy is a toolchain quality-of-implementation concern, not a frozen language invariant. A conforming compiler that re-materializes every instantiation is correct (just slow). A high-quality compiler avoids redundant work.
  **Why this matters for compile times**: Kyokai's generic surface is smaller than Rust's (no trait objects, no `dyn`, no deep adapter chains, simpler associated types). But the architecture matters more than the surface area. By naming workspace-level caching as the preferred strategy and requiring `.koi` metadata to support it, Kyokai's toolchain is designed for fast incremental builds from the start rather than backing into a model that requires linker-level deduplication of redundant work already done.
  **[DECIDED: D82b → `.koi` carries generic body metadata for downstream materialization; toolchain coordinates instantiation ownership explicitly; workspace-level cache is preferred strategy; redundant materialization is legal but discouraged; strategy is toolchain-level, not language invariant]**
- **Incremental compilation is layered: modules are the within-package reuse unit, packages are the cross-package artifact unit, and fingerprint/query machinery is the implementation technique underneath** — Kyokai does not force a false choice between Go-style package caching, OCaml-style module recompilation boundaries, and Rust-style dependency fingerprinting.
**Rules**:
  1. A logical module (`.kyo` + `.kai`) is the separate-compilation and invalidation unit within a package.
  2. A package and its `.koi` artifact are the cross-package compatibility and downstream invalidation unit.
  3. Each module produces a semantic summary digest capturing exactly the declarations and facts that sibling modules inside the package may depend on, including used `internal` declarations.
  4. If a module body changes without changing that module's semantic summary digest, unrelated sibling modules in the same package need not be recompiled.
  5. If a package rebuild leaves its serialized `.koi` artifact unchanged, downstream packages must not be recompiled.
  6. Cache and invalidation keys include at least the language edition, compiler compatibility class, target contract, selected CPU-feature baseline, active profile, and dependency interface digests.
  7. Independent modules inside one package and independent packages inside one workspace should be compiled in parallel.
  8. D82b's generic-instantiation cache is part of this model rather than a separate ad hoc cache layer.
  9. The reference compiler should realize this through dependency fingerprints or query-style invalidation, but the normative external contract is the module/package reuse boundary above, not one mandatory internal compiler architecture.
  **Why this fits Kyokai**: cross-package builds get a hard `.koi` boundary, same-package edits do not collapse back to whole-package recompilation, and the implementation still has room to use modern dependency tracking instead of pretending timestamps alone are enough.
  **[STAGE: DECIDED_CORE_SEMANTICS | D144 → hybrid incremental model: module-level within packages, package-level across packages, fingerprint/query machinery underneath]**
- **Monomorphized code-size mitigation is explicit and layered: compiler sharing is governed by D82a, and optimizing profiles request post-codegen identical-code folding where the selected toolchain can do it without violating the active debug/diagnostic contract** — Kyokai does not treat "same size/alignment" as a semantic equivalence rule, and it does not leave binary-size policy to unnamed linker folklore.
**Rules**:
  1. Compiler-level sharing of generic bodies is governed by D82a. Matching size or alignment alone is never sufficient justification for sharing.
  2. A compiler may share concrete instantiations only when it can prove that the resulting emitted body preserves D82a's requirements, including all type-dependent behavior, layout-sensitive operations, contract evaluation, and diagnostics.
  3. Toolchain-level identical-code folding is a separate post-codegen optimization. It may merge emitted machine-code sections only when doing so preserves Kyokai language semantics and the active profile's debugging/diagnostic contract.
  4. The standardized profile control for this optimization is D31's `identical_code_folding` field.
  5. The conventional `debug` profile sets `identical_code_folding = false`. The conventional `release` and `size` profiles set `identical_code_folding = true`.
  6. The exact linker flags or backend passes used to realize identical-code folding are target/toolchain configuration under D31. The language does not standardize one flag spelling such as `--icf=all`.
  7. If a profile explicitly requests identical-code folding and the selected backend/toolchain cannot provide a conforming implementation, the build fails rather than silently pretending the request was honored.
  **Why this fits Kyokai**: compiler sharing stays semantically rigorous, optimizing profiles get an explicit size-reduction lever, and toolchain behavior becomes part of the written contract rather than a hidden linker accident.
  **[STAGE: DECIDED_CORE_SEMANTICS | D200 → layered monomorphization size mitigation: D82a-proof-based compiler sharing plus explicit profile-controlled toolchain identical-code folding]**
- **Kyokai names its target audience explicitly and uses that audience as a design filter** — the language is not designed for "everyone who might ever write systems software." It is designed first for programmers coming from C who want modern safety without paying for hidden behavior or unreadable code at scale.
**Rules**:
  1. Kyokai's primary audience is C programmers who want memory safety, resource safety, capability security, and faster codebase comprehension than C gives them.
  2. Kyokai's secondary audience is broader systems programmers, but secondary audience pressure does not override the primary readability-and-explicitness target.
  3. The language is optimized for code that a competent C programmer can read, audit, and reason about quickly.
  4. Readability under real codebase scale is part of the design contract, not an aesthetic preference.
  5. When a feature increases abstraction power but makes ordinary systems code harder to scan without delivering a commensurate safety gain, Kyokai rejects it.
  **Why this fits Kyokai**: it resolves the plan's identity question directly. Kyokai is not "Austral with more stuff" and not "Rust minus a few features." It is a modern systems language built around explicit ownership, explicit authority, and code that stays legible under pressure.
  **[STAGE: DECIDED_CORE_SEMANTICS | D145 → explicit target audience: primary C programmers seeking modern safety and codebase readability; secondary broader systems programmers]**
- **Kyokai carries an explicit non-goals list, and crossing one of those boundaries requires a new full decision point** — the language does not let scope creep hide behind "maybe later" ambiguity.
**Normative non-goals**:
  1. No garbage collector.
  2. No implicit destructors, `Drop`, or compiler-inserted cleanup calls at ordinary scope exit.
  3. No exceptions or stack unwinding.
  4. No runtime reflection.
  5. No null pointers as an ordinary language value model.
  6. No safe ambient global mutable state.
  7. No inheritance or class-hierarchy object model.
  8. No macro system or syntactic metaprogramming surface.
  9. No erased trait-object or hidden runtime-dictionary polymorphism.
  10. No competing second mechanism when one explicit mechanism already covers the semantic job.
  11. No weakening of safety checks or analysis obligations merely to win compile-time benchmarks.
  12. No language-level `async`/`await`, `Future`-style colored concurrency surface, or hidden executor model.
  **Rules**:
  1. This list is normative project boundary, not aspirational marketing.
  2. Reversing any item requires a new explicit D-point that names the reversal and its consequences.
  3. New non-goals may be added only by the same explicit-decision process.
  **Why this fits Kyokai**: boundaries are part of the product. A language named "boundary" should say where its boundaries are.
  **[STAGE: DECIDED_CORE_SEMANTICS | D147 → explicit normative non-goals list; changes require a new D-point]**
- **The official language server is part of the toolchain and shares the compiler's analysis engine rather than re-implementing it** — Kyokai does not accept a split-brain tooling model where the editor and compiler disagree because they are separate semantic systems.
**Rules**:
  1. The official entrypoint is `kyokai lsp`.
  2. The language server ships as part of the same reference toolchain release as the compiler.
  3. Parsing, name resolution, type checking, linearity checking, `.koi` loading, and incremental dependency state are shared compiler-library components used by both batch commands and the LSP.
  4. `kyokai check` and `kyokai lsp` must agree on diagnostics for the same source and configuration.
  5. The implementation must support broken and partial editor buffers through parser recovery and incremental snapshots rather than by inventing a second weaker semantic model.
  6. The official feature floor includes diagnostics, hover/type information, go-to-definition, find references, semantic tokens, and explicit visibility into borrow/consumption state.
  7. The official feature target includes completion, rename, code actions, workspace symbol search, and inlay hints for type arguments, capability flow, and other semantically relevant information the shared engine can expose.
  **Why this fits Kyokai**: linearity, capabilities, and comptime eligibility are too central to leave editor support as an afterthought. Sharing the compiler engine keeps the tool honest and avoids rust-analyzer-style duplicate-logic drift.
  **[STAGE: DECIDED_CORE_SEMANTICS | D148 → official in-tree `kyokai lsp` sharing the compiler engine; compiler and LSP are developed together, not as separate semantic systems]**
- **Cross-compilation uses one manifest-centered configuration model that can import reusable target-spec files, and both backends are described explicitly under that same model** — Kyokai does not split target configuration between ad hoc CLI folklore, one manifest world for ordinary targets, and a second custom-target format for embedded work.
**Rules**:
  1. `kyokai build --target <triple>` selects a legal D80 target triple.
  2. Backend selection is explicit through `[build].backend` or CLI `--backend <c|llvm>`.
  3. `[target.<triple>]` is the primary configuration surface for cross-compilation.
  4. A target table may import a reusable target-spec TOML file with `spec = "<relative-path>"`.
  5. Imported target-spec data and manifest-local overrides participate in one merged configuration model; target specs are not a second independent configuration system.
  6. Shared target fields live at `[target.<triple>]`. Backend-specific overrides live at `[target.<triple>.backend.c]` and `[target.<triple>.backend.llvm]`.
  7. CLI overrides manifest-local overrides imported target-spec values. Environment variables may assist tool discovery only where the manifest/configuration model explicitly permits them; they are never the primary contract.
  8. The resolved target-spec inputs are part of the reproducible build identity under D83.
  9. If the selected target/backend combination lacks a conforming toolchain configuration, the build fails rather than silently changing backend or target behavior.
  **Why this fits Kyokai**: it keeps cross-compilation explicit, reviewable, and backend-aware without forcing users into two unrelated config languages or pretending the LLVM and C backends can share one unnamed folklore contract.
  **[STAGE: DECIDED_CORE_SEMANTICS | D149 → manifest-centered cross-compilation with importable target-spec TOML files and explicit backend-specific target configuration for both C and LLVM]**
- **`kyokai audit` is a first-class authority-audit tool built on top of Kyokai's capability model rather than a separate sandbox mechanism** — the language already enforces explicit authority flow; auditing makes that authority surface inspectable and CI-checkable.
**Rules**:
  1. The official entrypoint is `kyokai audit`.
  2. `kyokai audit` loads the locked dependency graph and computes, for each package, both:
     - the public capability surface exposed through public API
     - the full implementation capability ceiling used anywhere inside the package
  3. `kyokai audit` also reports unsafe-boundary facts, including `pragma Unsafe_Module`, raw `foreign` usage, `UnsafeCapability`, dynamic loading, and plugin loading when present.
  4. Package manifests may declare an explicit audit policy in `[audit]`.
  5. The standardized manifest field `capability_ceiling = [...]` is checked against the inferred full implementation capability ceiling, not merely against the public API surface.
  6. Standardized booleans such as `uses_unsafe`, `uses_dynamic_load`, and `uses_plugins` must match the inferred package behavior or the audit fails.
  7. Application-root audit output includes the transitive union across the locked dependency graph.
  8. This tool is orthogonal to CVE scanning. It is about authority and unsafe-boundary visibility, not vulnerability databases.
  **Why this fits Kyokai**: package auditing becomes the supply-chain expression of the same explicit-authority philosophy used inside ordinary source code.
  **[STAGE: DECIDED_CORE_SEMANTICS | D150 → first-class `kyokai audit`; public-surface plus implementation-ceiling reporting; manifest-declared audit policy checked against inferred capability and unsafe usage]**
- **Exploratory tooling is provided through compiler-backed `kyokai eval` and `kyokai repl`, not through a separate subset interpreter** — learning and experimentation must use the real language rules rather than a toy semantics that teaches the wrong ownership, capability, or cleanup model.
**Rules**:
  1. The official exploratory entrypoints are `kyokai eval <file.kai>` and `kyokai repl`.
  2. Both commands use the same parser, resolver, type checker, linearity checker, and runtime contracts as ordinary compilation. They are not a reduced subset language and not a separate interpreter semantics.
  3. `kyokai eval` executes a single source file as a synthetic runnable unit. The file format is: zero or more file-scope `import` declarations followed by an ordinary statement body.
  4. `kyokai eval` lowers that statement body to the synthetic entrypoint `main(root: RootCapability, args: &[Span[String]]): ExitCode`.
  5. Inside `kyokai eval`, the names `root` and `args` are the ordinary entrypoint bindings of that synthetic `main`, and reaching the end of the file returns `ExitSuccess`.
  6. `kyokai repl` is a persistent interactive session whose session-state semantics are defined by D151a.
  7. `build`, `check`, `repl`, `eval`, and `lsp` share the same core compiler-library components under D148; exploratory mode does not get a weaker semantic engine.
  **Why this fits Kyokai**: it lowers experimentation friction without teaching a fake language, and it keeps exploratory tooling aligned with the same explicit contracts users will meet in real packages.
  **[STAGE: DECIDED_CORE_SEMANTICS | D151 → official `kyokai eval` + `kyokai repl`, both using the real compiler engine and ordinary language semantics]**
- **`kyokai repl` is one persistent session scope with ordinary linear obligations, not a garbage-collected scratchpad** — interactive use does not suspend Kyokai's ownership rules or invent hidden session cleanup.
**Rules**:
  1. A REPL session maintains two persistent environments:
     - a synthetic module environment for accepted imports and declarations
     - a persistent session-execution scope for top-level executable bindings and deferred actions
  2. Accepted imports and declarations extend the synthetic module environment for later inputs.
  3. Accepted executable inputs run in the same persistent session-execution scope unless they create their own inner lexical scopes. Top-level bindings introduced in one input remain available to later inputs subject to ordinary no-shadowing and linearity rules.
  4. `Free` bindings may persist across turns normally.
  5. `Linear` bindings may also persist across turns, but they remain ordinary live linear obligations and must still be consumed exactly once.
  6. A top-level `defer` registered in the session-execution scope remains pending until that session scope exits. Inner-scope `defer` and `errdefer` still run at their ordinary lexical exits within the current input.
  7. Top-level `errdefer`, `return`, `or return`, `break`, and `continue` are illegal in REPL session scope. They are legal only inside ordinary nested Kyokai constructs introduced by the current input where such exits have a real enclosing target.
  8. Bare expression input is legal only when the resulting type is both `Free` and `Displayable`; the REPL prints that value. A bare expression whose result type is `Linear` is a compile-time error unless the value is explicitly bound, consumed, or moved in the same input.
  9. Compile-time errors and rejected inputs do not mutate session state.
  10. Runtime `panic(message)` and TPOE follow ordinary program semantics and terminate the whole REPL session. They are not catchable as recoverable REPL events.
  11. `:quit` and `:reset` are host commands, not Kyokai syntax. They attempt ordinary exit of the session-execution scope: pending ordinary `defer` actions run under D2a/D2b, and the command is rejected if live linear bindings would remain unconsumed after those deferred actions.
  12. `:reset` starts a fresh empty session only after the previous session has exited legally under rule 11.
  13. Every new REPL session begins with explicit bindings `root: RootCapability` and `args: &[Span[String]]`, mirroring D48's entrypoint contract. `args` is the argument slice supplied after `--` to `kyokai repl`, or an empty slice when none is supplied.
  **Why this fits Kyokai**: REPL state stays real program state, top-level cleanup remains visible, and quitting a session does not become the one place where the language silently drops linear values.
  **[STAGE: DECIDED_CORE_SEMANTICS | D151a → REPL session is one persistent scope; linear values may persist across turns; `:quit`/`:reset` run ordinary `defer` and reject leftover live linear obligations]**
- **Kyokai's standard library is a batteries-included systems standard library, not a placeholder waiting for a future ecosystem** — the language project commits to shipping the foundations a systems programmer reasonably expects to use without third-party package hunting.
**Rules**:
  1. The official standard library is part of Kyokai's core value proposition, not a temporary bootstrap convenience.
  2. The committed standard-library scope includes at minimum facilities for core types, collections, formatting, text, filesystem, process management, time, randomness, networking, encoding, crypto, testing, documentation/testing support, and concurrency support libraries built on the decided language primitives.
  3. These categories are in-scope commitments of the project even when their exact APIs are phased across releases or split into later detailed D-points.
  4. Standard-library APIs must still obey D85 semantic contract documentation, capability gating, D44/D201 allocator explicitness, D73's no-language-UB model, and Kyokai's no-hidden-behavior rules.
  5. "Batteries included" does not mean open-ended application-domain expansion. The boundary is systems programming: if the absence of a facility would make ordinary Kyokai systems programming feel unfinished, it belongs in stdlib scope.
  **Why this fits Kyokai**: Austral's minimal-library posture is one of the main pressures that created Kyokai in the first place. Kyokai's lane is a readable, explicit systems language that is still usable out of the box.
  **[STAGE: DECIDED_CORE_SEMANTICS | D152 → batteries-included systems stdlib is a core project commitment; listed facility families are in scope even when API sequencing is phased]**
- **The core stdlib collection surface is committed explicitly rather than left to an open-ended future roadmap** — Kyokai will ship the ordinary systems-programming collection families directly instead of pretending `Buffer` plus one map type is an adequate long-term answer.
**Rules**:
  1. The committed core owning-collection families are `Buffer[T]`, `HashMap[K, V]`, `HashSet[T]`, `BTreeMap[K, V]`, `BTreeSet[T]`, `Deque[T]`, and `PriorityQueue[T]`.
  2. These are standard-library commitments, not optional ecosystem placeholders.
  3. All owning collection types are linear.
  4. Collection construction and any operation that allocates fresh owned storage remain allocator-explicit under D44 and D201.
  5. Hash-based collections use D134's explicit hasher-based model rather than an ambient default hash algorithm.
  6. Ordered collections rely on the explicit ordering contracts of the language rather than hidden comparator folklore.
  7. This decision does not commit Kyokai to `LinkedList` as part of the core collection set.
  8. This decision also does not introduce GC-dependent persistent collections, shared-ownership collection models, or other surfaces that would contradict Kyokai's explicit ownership boundary.
  **Why this fits Kyokai**: the language keeps its batteries-included promise for ordinary systems work while still drawing a firm boundary around what belongs in the core collection story.
  **[STAGE: DECIDED_CORE_SEMANTICS | D174 → explicit committed core collection families: `Buffer`, `HashMap`, `HashSet`, `BTreeMap`, `BTreeSet`, `Deque`, and `PriorityQueue`; allocator-explicit and linear; no `LinkedList` commitment]**
- **Sorting and ordered search are explicit standard-library operations built on `TotalOrder`, with key-projection helpers as the preferred customization surface** — Kyokai provides ordinary in-place sorting and binary search without making comparator folklore the primary API.
**Rules**:
  1. `Buffer[T]` provides `sort(&!buffer): Unit` when `T: TotalOrder`.
  2. `Buffer[T]` also provides `sortByKey(&!buffer, key): Unit`, where `key` is a callable that maps an element to some `K: TotalOrder`.
  3. The standard library provides `binarySearch(buffer, needle)` for buffers sorted under the element type's `TotalOrder`.
  4. The standard library also provides `binarySearchByKey(buffer, needle, key)` for buffers sorted by the same key projection used for the search.
  5. Standard `sort` and `sortByKey` are stable.
  6. Comparator-driven forms, if provided, are secondary explicit APIs such as `sortWith` or `binarySearchWith` taking a callable that returns `Ordering`; they do not replace the primary `TotalOrder` and `...ByKey` surfaces.
  7. Ordered collections such as `BTreeMap` and `BTreeSet` rely on the same D23/D175 ordering contracts rather than storing hidden per-container comparator state.
  **Why this fits Kyokai**: the common case stays concise, key-based customization covers the usual "sort by field" need without callback boilerplate, and lower-level comparator control remains explicit instead of becoming the only story.
  **[STAGE: DECIDED_CORE_SEMANTICS | D175 → stable `sort`/`sortByKey` plus `binarySearch`/`binarySearchByKey`; comparator forms remain secondary explicit APIs]**
- **Official learning material is staged as guide-plus-reference first, full book after self-hosting** — Kyokai does not outsource onboarding to future community folklore.
**Rules**:
  1. Before self-hosting, the project provides official reference documentation plus an official guide/tutorial site in the spirit of `zig.guide`.
  2. The early guide teaches the real Kyokai model directly: ownership, borrowing, capabilities, TPOE, package/toolchain flow, and standard-library usage patterns.
  3. After Kyokai is written in Kyokai, the project produces a full official book as the deeper canonical teaching text.
  4. The guide is the primary early learning path; the later book is the long-form canonical teaching text.
  **Why this fits Kyokai**: the language has too much semantic weight to rely on scattered posts and examples, but forcing the full book to exist before self-hosting would delay useful learning material for no real gain.
  **[STAGE: DECIDED_CORE_SEMANTICS | D153 → official guide/tutorial plus reference docs before self-hosting; full official book after self-hosting]**
- **Kyokai provides an Austral migration guide and difference reference, not a required mechanical translator** — the realistic migration audience is small enough that documentation beats translator engineering.
**Rules**:
  1. The project provides an Austral-to-Kyokai migration guide.
  2. The project also provides a syntax/semantics difference reference that maps Austral constructs to their Kyokai equivalents and explicitly calls out real semantic changes.
  3. The migration material includes before/after examples for each important surface or semantic difference.
  4. No mechanical translator is required by the plan.
  **Why this fits Kyokai**: it serves the actual likely early audience without spending design and implementation effort on translator machinery for a very small ecosystem.
  **[STAGE: DECIDED_CORE_SEMANTICS | D154 → Austral migration is documentation-first: migration guide + difference reference + before/after examples; no required translator]**
- **Language evolution uses a maintainer-led public decision-point process, and spec/compiler divergence is a bug** — Kyokai does not let accepted behavior drift between discussion threads, compiler experiments, and normative documents.
**Rules**:
  1. Non-trivial language, toolchain, standard-library-surface, compatibility, or governance changes begin as a short public proposal or issue.
  2. If the change affects semantics, syntax, toolchain contract, compatibility, or project boundary in a way that needs real design discussion, it must become an explicit D-point in the public decision tracker before it is considered accepted.
  3. Discussion happens publicly on that D-point in `Kyokaishape.md`, GitHub Discussions, issues, or PRs with the D-point label.
  4. The final written shape must exist before acks close the point.
  5. A public D-point needs at least three community acks on the final shape before it is considered decided, unless the maintainer explicitly marks an emergency or bookkeeping correction.
  6. Once a point is decided, the normative Kyokai spec becomes the source of truth for the accepted behavior; supporting notes and discussion artifacts are non-normative.
  7. The reference compiler, official toolchain components, and normative spec must converge on the same accepted behavior. Spec/compiler disagreement is a bug, not an alternate interpretation.
  8. The compiler may prototype undecided ideas only behind clearly non-default experimental flags or branch-local work. Such experiments do not change the language contract until a D-point closes and the normative spec is updated.
  **Why this fits Kyokai**: it preserves decision velocity while keeping the design auditable, explicit, and publicly traceable, and it directly rejects compiler-first drift as a source of truth.
  **[STAGE: DECIDED_CORE_SEMANTICS | D155 → maintainer-led public D-point governance; final shape plus 3 acks; normative spec is source of truth; spec/compiler divergence is a bug]**
- **Release cadence and language editions are separate clocks** — shipping rhythm is a toolchain policy, while editions remain rare source-semantics boundaries under D105.
**Rules**:
  1. During early development, releases ship when the maintainer judges them ready.
  2. Once the compiler, stdlib, and toolchain are operationally stable enough to support regular consumer-facing releases, the project adopts a regular release train with a default target cadence of four weeks.
  3. The release train ships completed work only; the calendar does not force incomplete features into a release.
  4. Language editions have no fixed time cadence. A new edition is cut only when a real source-semantic break or edition-gated default shift is worth the migration cost.
  5. Edition cadence is deliberately sparse and independent of the ordinary release train.
  6. D105's exact edition semantics and `.koi` compatibility rules remain unchanged by release cadence.
  **Why this fits Kyokai**: users get predictable toolchain releases once the project is mature enough, but editions remain rare deliberate source-boundary events rather than a second calendar ritual.
  **[STAGE: DECIDED_CORE_SEMANTICS | D157 → early releases ship when ready; later default to a four-week release train; editions remain rare and demand-driven on a separate clock]**

- **CI integration is defined by one portable release-artifact and installation contract**: Kyokai does not make CI support depend on one host platform, one forge, or a second hidden installer semantics.
**Rules**:
  1. Every official toolchain release publishes versioned binaries for supported host platforms, SHA-256 checksums, and provenance or signature metadata.
  2. Releases are available at stable, version-addressable URLs.
  3. Kyokai provides an official `kyokai/setup-kyokai` GitHub Action that installs an explicitly requested toolchain version from those same official release artifacts.
  4. Kyokai also publishes official OCI images containing the toolchain for CI use.
  5. CI guidance prefers immutable version tags or digests over floating tags.
  6. The GitHub Action is convenience tooling only; it must not define a second installation semantics.
  7. Any CI system capable of downloading the official release artifacts or OCI images can run Kyokai in a fully supported way.
  8. CI use of `kyokai check`, `kyokai build`, `kyokai test`, and `kyokai fmt --check` follows the same language and toolchain semantics as local use for the same explicit inputs.
  9. Cache behavior, if provided by official tooling, is an optimization only and must not affect language or build correctness.
  **Why this fits Kyokai**: the toolchain stays explicit and reproducible while still making first-party CI use trivial on common platforms.
  **[STAGE: DECIDED_CORE_SEMANTICS | D225 -> official release artifacts, official `setup-kyokai` action, official OCI images, and one portable CI installation contract]**
- **The public browser-playground story is Compiler Explorer plus one explicit sandbox-runner contract**: Kyokai should support try-it-in-browser use without requiring a bespoke hosted playground to be the language's only public surface.
**Rules**:
  1. The project supports a public "try Kyokai" experience through Compiler Explorer integration when feasible.
  2. The toolchain also defines an official sandbox runner contract for browser-based compilation and execution.
  3. A conforming playground runner compiles each snippet from source using an explicitly selected toolchain version, backend, and target.
  4. Execution is sandboxed with no ambient filesystem persistence, no ambient network access, and explicit CPU, memory, and wall-time limits.
  5. Standard output, standard error, compile diagnostics, and formatter output are captured explicitly and reported separately.
  6. Playground execution grants no extra capabilities beyond what the sandbox runner explicitly provides.
  7. Shared links or permalinks must capture the source text and the explicit toolchain settings needed to reproduce the result.
  8. A dedicated hosted `play.kyokai.dev` service may exist as one implementation of this runner contract, but it is not the only acceptable public surface.
  9. Any official hosted playground must use the same runner semantics as the documented runner contract. There is no second hidden web-only execution model.
  **Why this fits Kyokai**: it gives the project a realistic one-maintainer adoption path while still making the execution contract explicit and portable.
  **[STAGE: DECIDED_CORE_SEMANTICS | D226 -> Compiler Explorer plus an official sandbox-runner contract; hosted `play.kyokai.dev` remains optional]**
- **Kyokai provides both property-based testing and coverage-guided fuzzing** — testing a safety-focused language requires both generator-driven specification checking and mutation-driven coverage exploration.
**Rules**:
  1. Kyokai provides both property-based testing and coverage-guided fuzzing.
  2. Property-based testing lives in an official stdlib/testing surface such as `Kyokai.Test.Property`.
  3. The property surface includes typed generators `Gen[T]`, property runners, shrinking for built-in core data types, reproducible seeding, and deterministic replay by explicit seed.
  4. `kyokai test` can run property tests as part of the ordinary test workflow.
  5. `kyokai test --fuzz` is an official toolchain mode for coverage-guided fuzzing.
  6. Fuzzing includes corpus management, crash reproducers, deterministic replay, and integration with the decided coverage infrastructure from D219.
  7. Fuzz targets are explicit test declarations or explicit test-adjacent declarations under toolchain-defined discovery rules; Kyokai does not invent hidden execution entrypoints.
  8. Fuzzing and property testing are complementary, not competing mechanisms: property testing is generator/spec-driven, fuzzing is mutation/coverage-driven.
  **Why this fits Kyokai**: a safety-focused language should make correctness verification easy. Both mechanisms are committed as part of the toolchain/stdlib plan, not phased or deferred.
  **[STAGE: DECIDED_CORE_SEMANTICS | D220 -> property-based testing and coverage-guided fuzzing as committed toolchain/stdlib facilities]**
- **Kyokai provides an official read-only package index service** — Go-style discovery infrastructure over git-hosted packages, not a centralized publish registry.
**Rules**:
  1. Kyokai provides an official read-only package index service.
  2. Packages remain git-hosted; the index is discovery infrastructure, not the canonical storage location.
  3. Index inclusion requires a reachable repository containing a valid `kyokai.toml`.
  4. The index records package metadata, repository URL, docs link, declared versions, tags when present, and dependency metadata derivable from manifests.
  5. `kyokai add <name>` may resolve through the official index, but the resulting manifest entry still records an explicit git source plus explicit revision as required by D51.
  6. The index does not introduce a `publish` command, registry-only package names, or registry-mediated trust semantics.
  7. The index may host generated docs and API metadata, but those are derived mirrors, not the source of truth.
  8. Alternative third-party indexes are allowed; the language/toolchain model does not require exactly one global index.
  **Why this fits Kyokai**: discovery is essential for ecosystem growth, but D51's git-pinned dependency model remains the source of truth. The index is a lens, not a gatekeeper.
  **[STAGE: DECIDED_CORE_SEMANTICS | D221 -> official read-only package index (Go model); discovery only, not canonical storage; D51 git+rev dependency model unchanged]**
- **Kyokai has no separate `kyokai lint` semantic tool; the compiler is the linter** — all linting logic lives in the compiler/toolchain engine used by build, test, check, and LSP.
**Rules**:
  1. All linting logic lives in the compiler/toolchain engine used by build, test, check, and LSP.
  2. Lints are categorized at minimum into: hard error, warning, and style.
  3. Correctness and soundness violations stay compile errors, not optional lints.
  4. New non-style lints default to `warn`, not `error`, unless they enforce an already-decided language rule.
  5. Per-project lint configuration and suppression live in `kyokai.toml`.
  6. There are no per-line suppression comments such as `nolint`, `allow`, or pragma-style inline escapes.
  7. LSP surfaces the same lint set as the compiler; there is no second analyzer with a divergent rulebook.
  **Why this fits Kyokai**: "one semantic engine" means the compiler, the LSP, and the linter are the same program. No second tool with different opinions.
  **[STAGE: DECIDED_CORE_SEMANTICS | D222 -> compiler-integrated lints; tiered error/warning/style; `kyokai.toml` suppression; no per-line suppression comments; no separate lint tool]**
- **Kyokai uses SemVer as the official package-versioning convention** — SemVer for human communication, git `rev` for reproducible resolution.
**Rules**:
  1. Kyokai uses SemVer as the official package-versioning convention.
  2. The `version` field in `kyokai.toml` is meaningful package metadata, but reproducible dependency resolution still uses explicit git revision pinning under D51.
  3. Tooling provides `kyokai semver-check` to compare two public API surfaces using `.kyo` interfaces and classify changes as breaking, additive, or patch-compatible.
  4. The check is advisory tooling, not language-level enforcement.
  5. The source of truth for compatibility is the declared public interface surface, not implementation details in `.kai`.
  6. A package may choose bad version numbers, but the tooling can report the mismatch clearly.
  **Why this fits Kyokai**: SemVer gives humans a shared vocabulary for what changed. `rev` gives the build system exact reproducibility. Advisory tooling bridges the gap without forcing enforcement.
  **[STAGE: DECIDED_CORE_SEMANTICS | D223 -> SemVer convention + advisory `kyokai semver-check` over `.kyo` interface surfaces; `rev` remains the reproducibility mechanism]**
- **Build-time code generation is manifest-declared, not hidden in auto-executed language files** — explicit `[generate]` steps plus narrow comptime embedding replace the `build.rs` model.
**Rules**:
  1. Build-time code generation is declared in `kyokai.toml`, not hidden in auto-executed language files.
  2. The toolchain provides an explicit `kyokai generate` command.
  3. Generation steps are listed in a manifest `[generate]` surface with declared inputs, outputs, and commands.
  4. Generated outputs are ordinary files in the source tree or declared generated directories with explicit paths; they are not hidden compiler-side ephemeral artifacts unless separately decided.
  5. Reproducibility rules from D83 apply to generation inputs and outputs.
  6. Comptime remains sandboxed and does not gain ambient filesystem or process access through this decision.
  7. Asset embedding is provided by an explicit comptime facility such as `@embedFile(...)` or Kyokai-equivalent surface, with deterministic byte-for-byte embedding semantics.
  8. FFI bindgen, protocol codegen, and asset embedding are thus handled by explicit manifest generation plus narrow explicit comptime embedding, not by a general executable build script language.
  9. No `build.kai`, no auto-running package code during dependency resolution, and no hidden pre-build execution model.
  **Why this fits Kyokai**: codegen is visible in the manifest, embedding is visible in source, and no package code runs as a side effect of dependency resolution.
  **[STAGE: DECIDED_CORE_SEMANTICS | D224 -> manifest-declared `[generate]` steps + `@embedFile` comptime extension; no `build.kai` or auto-execution]**

### 3.4 Readability Research - What Science Says

This section synthesizes findings from cognitive science, eye-tracking studies, and programming language theory research to inform Kyokai's syntax decisions. Every claim here is grounded in published research or verified observation.

#### How Programmers Actually Read Code

**Code comprehension uses the brain's executive network, NOT language processing.** fMRI studies (Siegmund et al. 2014, "Understanding, Understanding Source Code with Functional Magnetic Resonance Imaging") show that reading source code activates the **multiple demand network** (MDN) — the same network used for logic puzzles, mathematics, and spatial reasoning. Crucially, it does NOT primarily activate Broca's area or Wernicke's area, which handle natural language processing.

**What this means for syntax design**: Making code "read like English" is a misleading goal. The brain doesn't process code as language — it processes it as **structured patterns**. Syntax should optimize for:

1. **Pattern recognition** — consistent shapes that the MDN can chunk
2. **Working memory load** — fewer symbols to hold in active memory
3. **Dependency locality** — things that depend on each other should be close together

**Eye-tracking reveals fixation patterns.** Studies (Busjahn et al. 2015, "Eye Movements in Code Reading") show that experienced programmers fixate primarily on:

1. **Identifiers** (variable and function names) — ~60% of fixation time
2. **Operators** — ~20% of fixation time
3. **Keywords** — ~10% of fixation time
4. **Delimiters** (braces, semicolons) — ~10% of fixation time

**What this means**: The NAMES of things matter more than the syntax sugar around them. This validates Austral's emphasis on meaningful identifiers and explicit type annotations. But it also means that BOILERPLATE (which contains zero useful identifier information) is literally wasted fixation time — the programmer's eyes scan past `generic [R: Region]` 18 times without extracting any useful information.

#### SVO Ordering and Dependency Locality

**Subject-Verb-Object is the most common word order in natural languages** — approximately 80% of the world's languages use either SVO (English, Mandarin, Spanish) or SOV (Japanese, Hindi, Korean) ordering. Only ~10% use VSO (Arabic, Irish, Hawaiian).

**Applied to programming syntax**:

- `buffer.append(byte)` — SVO: subject=buffer, verb=append, object=byte
- `appendByte(&~out, byte)` — VSO: verb=appendByte, subject=&~out, object=byte
- `out := append(out, byte)` — SOV: subject=out, verb=append, object=byte

Austral uses **VSO (function-call) syntax** for everything — `appendByte(&~out, byte)`. This is the LEAST common ordering in natural languages. But Austral has principled reasons: it avoids method syntax (which would require a receiver concept and implicit `self` parameter, violating "no hidden anything").

**The tradeoff**: VSO syntax is more explicit (the function name comes first, you see EXACTLY what's being called) but less ergonomic (the subject is buried in the argument list). Kyokai could add **method call syntax as sugar** — `out.appendByte(byte)` desugaring to `appendByte(&~out, byte)` — but this would need to be carefully specified to avoid hidden behavior. See **D7**.

#### Naming and Cognitive Load

**Research consistently shows that naming consistency is the #1 factor in code readability** — more important than syntax choice, indentation style, or language features. Studies (Lawrie et al. 2006, 2007) demonstrate that:

- **Full words win**: `buffer_length` is read faster than `bufLen` which is faster than `bl`
- **Consistent casing matters more than which casing**: A codebase that consistently uses `camelCase` is read faster than one that mixes `camelCase` and `snake_case`, regardless of which convention is "better"
- **snake_case vs camelCase**: Binkley et al. (2009) found `camelCase` had slightly higher accuracy for individual identifier recognition, but Sharif & Maletic (2010) found `snake_case` was identified faster on first fixation and preferred by experienced programmers. The difference is small; consistency dominates.

**What Kyokai inherits from Austral**: `camelCase` for functions/variables, `PascalCase` for types and modules. This is consistent with itself. Changing it would impose a migration cost for no strong scientific reason. The recommendation is to **keep Austral's convention** and enforce it consistently.

#### Nesting Depth and Comprehension

**Shallow code is faster to understand.** Multiple studies correlate nesting depth with comprehension difficulty. Each additional nesting level adds to working memory load.

**Austral naturally encourages shallow code** because:

1. Linear types require balanced consumption on all branches — deep conditionals with linearity create a combinatorial explosion of consumption paths
2. No exceptions means no try/catch nesting
3. No closures/lambdas means no deeply nested callback patterns
4. Explicit resource management encourages factoring into small functions

This means Austral's `end if` / `end for` terminators have LESS value than in languages that allow deep nesting (like C/C++). See **D9** for analysis.

### 3.5 Naming Convention Principles

Drawing from the Rust API Guidelines (`api-guidelines/src/naming.md`, `api-guidelines/src/predictability.md`), adapted for Austral/Kyokai's linear type system:

#### Conversion Functions and Linearity

Rust defines a naming convention for conversion functions that encodes the **cost and ownership semantics** in the function name:


| Prefix  | Rust Meaning                                                  | Austral/Kyokai Mapping                                       |
| ------- | ------------------------------------------------------------- | ------------------------------------------------------------ |
| `as`_   | Free conversion, no allocation, borrows input, borrows output | Read-borrow in (`&[T, R]`), returns `Free` view or `&[U, R]` |
| `to`_   | Potentially expensive, allocates new value, borrows input     | Read-borrow in (`&[T, R]`), returns owned `Free` value       |
| `into`_ | Consumes input, returns owned output                          | Takes `Linear` value by move, returns new owned value        |


In Kyokai, this maps beautifully to the linearity system:

```austral
-- as_: borrow in, borrow out (or Free return). No ownership change.
generic [R: Region]
function asSpan(buf: &[ByteBuf, R]): Span[Nat8, R];

-- to_*In: borrow in, owned out, allocator explicit. Creates new allocation.
generic [R: Region, A: Allocator]
function toStringIn(buf: &[ByteBuf, R], alloc: &![A]): Result[String, AllocError];

-- into_: consume input, produce output. Ownership transfers.
function intoBuffer(s: String): ByteBuf;
```

The `into_` prefix is the most Austral-native because it maps directly to linear consumption — you give up `s`, you get back `ByteBuf`. The `as_` prefix maps to borrowing. The `to_*In` pattern maps to borrowing-plus-explicit-allocation when the operation creates a fresh owned result.

#### Constructor Naming

Following Rust (`predictability.md` lines 147–168):


| Pattern        | Meaning                        | Kyokai Example                        |
| -------------- | ------------------------------ | ------------------------------------- |
| `new` / `make` | Default constructor            | `makeByteBuf(cap: Index): ByteBuf`    |
| `with_*`       | Constructor with configuration | `withCapacity(cap: Index): ByteBuf`   |
| `from_*`       | Constructor from another type  | `fromSpan(s: Span[Nat8, R]): ByteBuf` |


Austral already uses `make` (e.g., `makeByteBuf`). Kyokai should keep this and add `from` variants.

#### Method-like vs Function Naming

If Kyokai adds method syntax (see **D7**), naming changes:

- **Without methods**: `bufferLength(buf)` — verb-object for actions, noun for queries
- **With methods**: `buf.length()` — no prefix needed, subject is implicit

The Rust guideline: getters do NOT use `get_` prefix. `buf.length()` not `buf.getLength()`. If we add methods, follow this.

---

## 4. Work Items — Prioritized by Hardness and Severity

No phases. No weeks. Items are ordered by severity (how badly Kyokai needs it) × hardness (how difficult it is to implement). Work is picked based on these scores and current project pressure.

### Severity Scale

- **S-CRITICAL**: Can't write real programs without it
- **S-HIGH**: Major pain point for any real usage
- **S-MEDIUM**: Would improve experience significantly
- **S-LOW**: Nice to have

### Hardness Scale

- **H-1 to H-3**: Straightforward implementation, well-understood algorithms
- **H-4 to H-6**: Requires careful design, moderate complexity
- **H-7 to H-10**: Requires deep compiler work, novel design decisions, or complex algorithms

---

### S-CRITICAL Items

#### [C01] Build System — H-5

**What**: A `kyokai build` command that reads a package or workspace manifest, resolves packages and modules according to D78, consumes the lockfile defined by D78/D51, and invokes the compiler.

**Why critical**: Currently you must manually pass every `.kyo` and `.kai` file in dependency order to `kyokai compile`. This is unusable for any project beyond trivial.

**Implementation reference**:

- The compiler already handles module ordering internally (`design-austral-compiler.md` — extraction pass resolves imports). A build system just needs to discover files and construct the correct ordering.
- Consider Zig's approach: a single `build.zig` file that IS the build system. For Kyokai: a `kyokai.toml` manifest.

**What it needs**:

- Nearest-manifest discovery for package roots
- `[workspace]` vs `[package]` validation (mutually exclusive per D78)
- Explicit workspace member loading from `[workspace].members`
- File discovery under the package module root declared by `[layout].module_root`
- D78 module resolution (`[layout].module_root`, `.` = directory separator, one import path = one `.kyo`/`.kai` pair)
- Duplicate logical-module detection
- Package/module dependency resolution (parse imports, topological ordering)
- Consume `kyokai.lock` from the workspace root or package root as appropriate
- Resolve the selected build profile and target triple
- Resolve `[target.<triple>]` toolchain settings and any per-profile target overrides
- Invoke compiler with correct file ordering
- Output binary / artifact according to profile

**Reference**: Zig's `std.Build` (`zig/lib/std/Build.zig`), Cargo's manifest format.

**See also**: D19 (conditional compilation / platform modules), D26 (CLI specification), D27 (debugging / `#line` directives), D31 (binary size / linking / build profiles), D51 (dependency declaration syntax), D78 (package/workspace model).

---

#### [C02] Result/Optional Types with Ergonomic Patterns — H-3

**What**: Proper `Result[T, E]` and enhanced `Optional[T]` types with `map`, `flatMap`, `unwrapOr`, and pattern matching sugar.

**Why critical**: Currently there's no standard way to handle errors-as-values. The spec rationale (`rationale/2.error-handling.md`) declares that Error Conditions should be "represented as values, and error handling done using standard control flow" (line 79-80), but provides no standard types for this.

**Implementation**:

```kyokai
// In pure Kyokai, no unsafe needed
union Result[T: Type, E: Type]: Auto is
    case Ok(value: T);
    case Err(error: E);
build;
```

Linear type interaction: `Result[File, Error]` is `Linear` because `File` is `Linear`. You MUST pattern match and handle both cases — the linear type system forces exhaustive error handling automatically.

---

#### [C03] String and Span Comparison — H-2

**What**: `spanEquals()`, `spanCompare()`, `stringEquals()`, `stringCompare()`, plus `startsWith()`, `endsWith()`, `contains()`, `find()`.

**Why critical**: bfetchaust had to manually implement byte-by-byte comparison (`Fetch.aum` lines 50-64). This is the most basic operation for any text processing.

**Implementation**: Pure Kyokai. No unsafe. Just byte comparison loops over `Span[Nat8, R]`. Implement `Equality` and `TotalOrder` typeclass instances for `Span` and `String`.

---

#### [C04] POSIX I/O Layer — H-5

**What**: Safe, capability-secure file I/O. This is what bfetchaust's `BfetchAust.Posix` and `BfetchAust.FastIO` provide, but generalized and made part of the stdlib.

**Why critical**: Without this, every program that does I/O must build its own POSIX wrapper from scratch. `bfetchaust` already proves the shape is needed; Kyokai should productionize it.

**Implementation**: The bfetchaust runtime modules are the reference implementation:

- `BfetchAust.Posix` shows which syscalls are needed: `read`, `write`, `open`, `close`, `stat`, `uname`, `sysinfo`, `readlink`, `access`, `fork`, `waitpid`, `pipe`, `dup2`, `execvp`.
- `BfetchAust.FastIO` shows the ByteBuf pattern: linear buffer with `make()`, `destroy()`, `append*()`, `writeAll()`.
- Wrap these in capability-secure API following the pattern from Borretti's capabilities blog post.

**Reference**: `bfetch/bfetchaust/runtime/BfetchAust.Posix.aui` for the syscall list, `bfetch/bfetchaust/runtime/BfetchAust.FastIO.aui` for the buffer API.

---

#### [C05] Concurrency Model — H-10

**What**: Safe concurrent programming primitives.

**Why critical**: Borretti deliberately punted on this. But any real systems program needs threads or async I/O.

**Why hardness 10**: This is the biggest design decision Kyokai faces. Options:

1. **Message-passing with linear channels** (Erlang-like): Each thread gets its own linear capability. Communication via channels that transfer ownership of linear values. No shared memory → no data races. Most Austral-compatible because linearity already prevents aliasing.
2. **Structured concurrency** (like Zig's or Swift's): Spawned tasks cannot outlive their parent scope. This aligns with Austral's lexical scoping model for regions.
3. **Fork-join** (like Rayon): Safe parallel iterators. Simpler but less general.

**Key insight**: Linear types already solve the hardest part of concurrency — aliasing. A linear value has exactly one owner. If you send it across a channel, you lose it. No data races by construction. This is explicitly stated in the resource-types rationale (`rationale/3.resource-types.md` lines 35-38): "Safe concurrency. A value of a linear type, by definition, has only one owner: we cannot duplicate it because we can only use it once. So imperative mutation is safe, since we know that no other thread can write to our linear value while we work with it."

**Decision required**: Which model? This should be prototyped and experimented with before committing.

---

### S-HIGH Items

#### [H01] `defer` Statement — H-6

**What**: `defer consume(x);` at variable declaration point. Compiler ensures the deferred call happens on every exit path from the current scope.

**Why high severity**: bfetchaust's 11-line destroy chain (`Fetch.aum` lines 387-397) demonstrates the problem. This is the #1 ergonomic complaint with linear types.

**Why hardness 6**: Requires compiler modification:

- Parse new `defer` keyword
- In linearity checker: `defer consume(x)` marks `x` as "will be consumed" at scope exit
- Code generation: insert deferred calls before every `return` in scope

**Key constraint**: The `defer` MUST appear in source code. It IS source code. There are no hidden destructor calls. The programmer writes `defer destroyByteBuf(x);` and that's what happens. It's syntactic sugar equivalent to writing the destroy call before every return statement, nothing more.

**Reference**: Zig's `defer` and `errdefer`. Go's `defer`. Both have visible-in-source semantics.

---

#### [H02] Integer and Float Math Library — H-4

**What**: `Kyokai.Math.Int` and `Kyokai.Math.Float` — pure integer and float operations.

**Implementation**: All pure Kyokai. No unsafe. No FFI.

For integers: `abs()`, `min()`, `max()`, `clamp()`, `gcd()` (Euclidean algorithm), `lcm()`, `isPowerOfTwo()` (single AND operation), `nextPowerOfTwo()` (bit manipulation), `countLeadingZeros()`, `countTrailingZeros()`, `popcount()` (all implementable via bit twiddling), wrapping/saturating/checked arithmetic variants.

For floats: IEEE 754 bit manipulation. `isNan(x)` = `x != x`. `isInf(x)` = check exponent bits. `abs(x)` = clear sign bit. `floor()`/`ceil()`/`round()` via exponent inspection.

**Reference**: Hacker's Delight (Henry S. Warren) for bit manipulation algorithms. musl `src/math/__fpclassify.c` for float classification.

---

#### [H03] Trigonometry and Exponentials — Pure Implementation — H-6

**What**: `sin()`, `cos()`, `tan()`, `exp()`, `log()`, `sqrt()`, `pow()` — all in pure Kyokai.

**Why not wrap libm**: These are polynomial evaluations. `sin(x)` is a Chebyshev polynomial after range reduction to `[-π/4, π/4]`. `sqrt(x)` is Newton-Raphson iteration. `exp(x)` is range reduction + polynomial + ldexp. There are no syscalls, no state, no side effects. It's pure math.

**Implementation plan**:

1. Port FDLIBM's `s_sin.c`, `s_cos.c`, `s_tan.c` — each is ~100 lines of C doing argument reduction + minimax polynomial evaluation. The algorithms are public domain (Sun Microsystems, 1993).
2. Port FDLIBM's `e_sqrt.c` — bit manipulation + Newton-Raphson. ~50 lines.
3. Port FDLIBM's `e_exp.c` — range reduction + Horner polynomial. ~80 lines.
4. Verify precision against mpfr/glibc test vectors.

**Reference**: FDLIBM source at `https://www.netlib.org/fdlibm/`. Each function file includes detailed mathematical comments explaining the algorithm and error bounds.

---

#### [H04] HashMap — H-7

**What**: Linear-safe hash map. Keys and values can be `Linear` or `Free`.

**Why hard**: Hash tables require internal arrays (pointer arithmetic) and hashing. The linear type system means:

- Inserting a linear key/value transfers ownership to the map.
- Removing returns ownership back to the caller.
- The map itself is `Linear` and must be explicitly destroyed, which destroys all remaining key/value pairs.

**Implementation sketch**:

- Open addressing (Robin Hood hashing) for cache friendliness.
- Internal `Buffer` for storage (already exists and is linear-safe).
- Hash via the hasher-based `Hashable` protocol: the container selects a concrete hasher, feeds each key through `Hashable.hash(&key, &!hasher)`, and uses the hasher's explicit `finish` result.
- Must handle: insert, remove, get (returns borrow), contains, iterate, destroy.

**Reference**: Zig's `std.HashMap` implementation, Rust's `hashbrown` for Robin Hood hashing strategy.

---

#### [H05] Allocator Abstraction — H-7

**What**: A typeclass-based allocator interface so containers can use custom allocators.

Current state: Everything in the stdlib directly calls `malloc`/`free` through `Austral.Memory`. This means:

- No arena allocators
- No stack allocators
- No pool allocators
- No custom alignment

**Design**:

```kyokai
typeclass Allocator(A: Type) is
    method allocate(alloc: &![A, R], size: Index): Result[Address[Nat8], AllocError];
    method deallocate(alloc: &![A, R], ptr: Address[Nat8], size: Index): Unit;
    method reallocate(alloc: &![A, R], ptr: Address[Nat8], old_size: Index, new_size: Index): Result[Address[Nat8], AllocError];
spec;
```

Every container (`Buffer`, `HashMap`, `String`, etc.) would be parameterized by allocator.

**Reference**: Zig's `std.mem.Allocator` interface. Rust's `std::alloc::Allocator` trait.

---

#### [H06] Separate Compilation — H-8

**What**: Compile **packages** independently, reuse their interface/code artifacts downstream, and link later. Module-by-module incremental recompilation inside a package remains desirable, but it is a secondary optimization layer, not the primary artifact boundary.

**Why hard**: The current compiler is whole-program. Generic body materialization currently happens globally. After D78, D79, D82, D82a, D82b, and D83, separate compilation now has a much more explicit shape:

- Package boundary from D78
- `.koi` interface artifact contract from D79
- Static-dispatch contract from D82
- Generic materialization / instantiation ownership from D82a and D82b
- Reproducible artifact identity from D83

So H06 is no longer "some future cache." It is the package-level compilation model of the language toolchain.

**What it requires**:

- Deterministic per-package `.koi` artifacts
- Reusable compiled code artifacts for nongeneric concrete code
- Enough generic/typeclass metadata for downstream instantiation where D82a and D82b require it
- Incremental dependency/fingerprint tracking for rebuild invalidation
- Link-time or semantics-preserving post-codegen deduplication of identical instantiations where profitable

**Important clarification**:

- **Required first**: package-level separate compilation
- **Wanted later**: module-level incremental recompilation within a package

That means the first implementation target is "depend on compiled packages without recompiling their whole source tree," not "every module is its own independently shipped artifact."

**Reference**: Austral's README explicitly identifies lack of separate compilation as the bootstrapping compiler's main limitation (`README.md`, Status section). OCaml's split between compiled interface artifacts and compiled object code is useful prior art for the package-artifact model.

---

#### [H07] Package Manager — H-6

**What**: `kyokai add`, `kyokai install`, `kyokai publish`, `kyokai update`. Resolve and fetch dependencies according to D51, maintain `kyokai.lock`, and surface package-level audit information.

**Tied to**: The capability-based security model. Borretti's blog post on capabilities (`how-capabilities-work-austral.md` lines 424-445) describes an auditing system where each dependency's unsafe modules must be reviewed. The package manager should implement this.

**Must implement**:

- Workspace dependencies by package name (`core = { workspace = "core" }`)
- External Git dependencies with mandatory `rev`
- Optional `tag`, verified against `rev` at add/update time
- `kyokai add` behavior that never leaves a moving reference in the manifest: unpinned Git adds are errors; tag-based adds resolve and write both `tag` and `rev`; any future HEAD convenience flow must resolve immediately to a stored `rev`
- Rejection of `branch` in manifests
- One lockfile per workspace, otherwise one lockfile per standalone package
- Package-name uniqueness checks within a workspace
- Fetch/update/cache behavior for pinned Git revisions
- Audit surfaces for unsafe modules in dependencies

**Reference**: Borretti's "Dependency Resolution Made Simple" blog post (referenced in the capabilities article). Zig's package manager. Cargo's lockfile format.

**See also**: D17 (visibility / `internal` keyword — requires package concept), D26 (CLI specification — `kyokai add`), D51 (dependency model), D78 (package/workspace model).

---

#### [H08] Fixed-Size Array Type — H-5

**What**: `Array[T, N]` where `N` is a compile-time constant. Stack-allocated, bounds-checked.

**Why needed**: Currently only `Buffer` (heap, dynamic) and `Span` (view, no ownership) exist. Many systems programs need fixed-size stack arrays.

**Requires**: Const-generic or dependent-type mechanism for the size parameter. This is a language-level change.

**See also**: D18 (compile-time evaluation — required for const-generic `N` parameter).

---

### S-MEDIUM Items

#### [M01] `defer` Integration with Linearity Checker — H-4

Ensure the linearity checker treats `defer consume(x)` as consuming `x` at every scope exit. Must validate that deferred consumptions don't conflict.

#### [M02] ASCII/Unicode Base Library — H-3

`Kyokai.Ascii` for byte classification and case conversion. Pure Kyokai, no unsafe. Later: UTF-8 validation and iteration.

**See also**: D30 (Unicode / string encoding — `String` vs `ByteBuf` model, UTF-8 guarantee).

#### [M03] Sorting Algorithms — H-3

`sort()` for `Buffer`. Quicksort + insertion sort hybrid. Pure Kyokai, operates on `Buffer` via mutable borrow. Requires `TotalOrder` typeclass constraint.

#### [M04] Region Parameter Simplification — H-8

Explore region inference for same-scope borrows. This is a compiler change affecting the type system. Must be very careful not to introduce hidden behavior. Possible approach: when all borrows in a function call come from the same scope, allow a single Region parameter to cover all of them. The programmer must be able to opt-in or out.

#### [M05] Cross-Compilation Support — H-5

Wire the C backend to emit code for different targets. Since we generate C, this mostly means setting correct `sizeof` and `alignof` values and using the right cross-compiler.

#### [M06] Error Reporting Improvements — H-4

Better error messages with source context, color output, suggestions. JSON error output for tooling.

**See also**: D29 (compiler diagnostics — full specification with diagnostic codes, output formats, warning categories, error recovery).

#### [M07] LSP Server — H-7

Language Server Protocol implementation for editor support. Needs incremental parsing, type information caching.

#### [M08] Documentation Generator — H-4

Extract docstrings from `.kyo` interface files, generate HTML/Markdown documentation. Austral already has a docstring syntax (triple-backtick strings).

#### [M09] Process Spawning — H-6

`Kyokai.Process`: `spawn()`, `waitpid()`, `pipe()`, `dup2()`. Capability-secure: requires `ProcessCapability`.

#### [M10] Networking — H-8

`Kyokai.Net.Socket`: TCP/UDP. Capability-secure. This is one of the hardest stdlib modules because networking is inherently stateful and error-prone.

---

### S-LOW Items

#### [L01] Typeclass Default Methods — H-4

#### [L02] `while let` Pattern Matching — H-3

#### [L03] Integer Literal Inference for Unambiguous Contexts — H-5

#### [L04] Structured Binding Improvements — H-3

#### [L05] LLVM Backend — H-9

#### [L06] Formal Specification Update — H-3

#### [L07] Test Framework — H-5

**See also**: D28 (testing framework — full specification with inline test blocks, assertions API, `kyokai test` runner).

---
