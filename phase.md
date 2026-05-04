# Kyokai Execution Phases

**Version:** 0.2.0  
**Date:** 2026-05-02  
**Status:** Active roadmap  
**Public decided shape:** `kyokaidecided.md`  
**Public shape tracker:** `Kyokaishape.md`  
**Future normative spec:** `kyokaispec/`

## 1. Purpose

This file defines the execution order for turning Kyokai from the decided language plan into a specified, tested, implemented, and eventually proven systems language.

This file is not a second language spec. It is not a reduced scope document. It is not an MVP list. `kyokaidecided.md` is the public extraction of accepted shape, `Kyokaishape.md` is the public D-point tracker, and `kyokaispec/` is the future normative spec. `phase.md` exists to answer these practical questions:

- what must be written before compiler work can safely move fast
- what parts of the Austral compiler/spec are forked, retired, or rewritten first
- when the Kyokai calculus must be written
- when the C backend, LLVM backend, stdlib, package manager, and tooling become valid targets
- what tests and proof artifacts close each phase
- which work is allowed to proceed in parallel and which work is gated by earlier semantics

The guiding rule is simple: implementation order must not create hidden semantics. If a feature cannot be implemented without guessing, the phase is blocked until the relevant rule is written in `kyokaispec/` or opened as a real public D-point in `Kyokaishape.md`.

## 2. Source-Of-Truth Order

Kyokai has several documents and code trees. They do not have equal authority.

1. `kyokaispec/` is the normative reference once a rule is written there.
2. `kyokaidecided.md` is the public accepted-shape extraction for rules not yet fully spec-extracted.
3. `Kyokaishape.md` is the public live D-point and pending-shape tracker.
4. `phase.md` is an implementation and proof-order roadmap only.
5. `lib/`, `standard/`, `test/`, and `test-programs/` are the active compiler, stdlib, and test trees in this repo.
6. Upstream Austral source/tests remain evidence for inherited behavior, not public Kyokai authority.

When a conflict appears, fix the lower-authority artifact. Do not silently reinterpret public Kyokai docs through stale inherited prose.

## 3. Current State

The initial language-design pass is closed enough to seed public development, but that does not mean Kyokai is implemented or proven. The public work now is to clean the accepted shape into `kyokaidecided.md`, write the normative spec in `kyokaispec/`, and track new public D-points in `Kyokaishape.md`.

Current assets:

- `kyokaidecided.md` contains the public accepted-shape extraction.
- `Kyokaishape.md` is the live public D-point tracker.
- `kyokaispec/` exists and is the target home for the normative spec.
- `lib/` is the forked Austral compiler codebase.
- `standard/` is the current inherited standard-library tree.
- `test/` and `test-programs/` are current compiler test assets.
Current liabilities:

- The old phase map used stale D-point meanings and has been replaced by this document.
- The current compiler and standard library are still largely Austral-shaped.
- `kyokaidecided.md` still needs public-voice cleanup and spec navigation.
- `kyokaispec/` is still mostly inherited Austral spec material and needs Kyokai rewriting.
- The paper proof required by D143/D241 has not been written yet.
- The future mechanized proof is intentionally after self-hosting, not an early bootstrap blocker.

## 4. Non-Negotiable Execution Rules

These rules apply to every phase.

1. No accepted safe Kyokai program may rely on language-level undefined behavior.
2. Backend lowering must preserve Kyokai semantics and must not encode Kyokai operations through C or LLVM undefined behavior.
3. Every user-visible semantic rule must live in a normative document, not only in compiler code.
4. A test can expose missing behavior, but a test cannot be the only specification of that behavior.
5. All sugar must lower through the D238 pipeline before later checks rely on it.
6. Implicit compiler completions must be recorded in the D239 registry and checked by the tautology pass.
7. Auto-reborrow and read-reborrow behavior must be backed by the D240 conformance matrix.
8. Unsafe operations require source-visible `unsafe contract ... audit;` coverage under D245.
9. Raw FFI cannot pretend to understand Kyokai linear ownership, sum-type layout, capabilities, or cleanup semantics.
10. Pure computation should be written safely in native Kyokai unless D229/D230/D231 explicitly justify a transitional or externally reviewed implementation.
11. The standard library must publish edge-case contracts before a module is admitted as stable.
12. Compiler behavior that is fuzzy enough to argue about is not ready to implement as hidden policy.

## 5. Maturity States

These states track implementation maturity. They are not language editions.

| State | Meaning | Exit Requirement |
| --- | --- | --- |
| `SHAPE_DECIDED` | The design point is decided in public accepted-shape docs. | `Kyokaishape.md`, `kyokaidecided.md`, and any public thread agree. |
| `SPEC_EXTRACTED` | The rule has a normative home outside the planning document. | Exact behavior, errors, examples, and cross-references are written. |
| `CALCULUS_DRAFTED` | The behavior is represented in `lambda_K` scope or explicitly excluded from it. | Syntax, static semantics, and dynamic semantics draft exist. |
| `CALCULUS_PROVEN_PAPER` | The sequential core proof obligation is discharged at paper level. | Preservation/progress-or-defined-failure proof is written and reviewed. |
| `PARSER_ACCEPTED` | Surface syntax is parsed into AST nodes with source spans. | Positive and negative parser tests pass. |
| `ELABORATED_CORE` | Surface constructs lower through the D238 ordered pipeline. | Elaborated output exposes all implicit completions and sugar nodes. |
| `CHECKED` | Name, type, borrow, linearity, capability, contract, and unsafe checks enforce the spec. | Negative conformance tests fail with stable diagnostics. |
| `LOWERED_SAFE` | Backend output implements the checked semantics without backend UB. | C backend and runtime tests cover UB-prone operations. |
| `CONFORMANCE_BACKED` | Behavior has executable tests and diagnostic goldens. | Positive, negative, and edge-case tests run in CI. |
| `STDLIB_ADMITTED` | A stdlib API has its contract, edge cases, tests, and implementation policy. | D229 admission checklist is satisfied. |
| `BOOTSTRAP_RELEASED` | The OCaml/Austral-derived compiler can compile practical Kyokai programs. | Real workloads build without local runtime shims for pure computation. |
| `SELF_HOSTING` | Important compiler components are written in Kyokai and built by Kyokai. | The Kyokai toolchain can build the self-hosting slice from source. |
| `MECHANIZED_PROVEN` | The relevant core theorem is machine-checked. | Proof assistant artifacts build in CI. |

## 6. Global Gates

These gates are stronger than ordinary phase completion. Public claims should not move past a gate until the gate is closed.

### Gate A: Plan-To-Spec Closure

Every decided D-point has one of these destinations:

- a normative language-spec section
- a normative toolchain-spec section
- a normative standard-library contract section
- an explicit implementation-only note with no semantic authority
- a historical-only note retained outside public spec flow

Done when there is no live behavior whose only source is an old recommendation block or informal note.

### Gate B: Sequential Core Soundness

D143/D241 require a paper-level proof for the sequential ownership-and-borrowing core before `v1.0`.

Done when `lambda_K` has:

- abstract syntax
- type and kind/universe rules
- linear and unrestricted contexts
- borrow and region rules
- operational semantics
- explicit TPOE / defined terminal failure states
- preservation theorem
- progress-or-defined-failure theorem
- a surface-to-core mapping for the features included in the first proof scope

### Gate C: Compiler Semantic Pipeline

The compiler must implement the D238/D239/D240 pipeline before ergonomic surface features are considered trustworthy.

Done when parser, typed elaboration, implicit-completion registry, tautology checking, sugar lowering, and linearity/borrow/capability checks are observable in tests and diagnostics.

### Gate D: Backend UB Closure

The C backend and later LLVM backend must not create C/LLVM undefined behavior for any safe Kyokai operation.

Done when backend-specific lowering rules exist for arithmetic, shifts, floats, enum/union tags, records, packed records, pointers, stack checks, volatile, atomics, panic/TPOE, and FFI calls.

### Gate E: Practical Bootstrap Usability

Kyokai must compile real systems code, not only toy examples.

Done when a bfetchaust-shaped workload can be written in Kyokai with native stdlib support for buffers, strings, POSIX I/O, process information, formatting, error handling, and cleanup.

### Gate F: Public Conformance Release

This gate is the earliest point where the project can claim the language surface is externally testable.

Done when the extracted spec, `lambda_K` paper proof, compiler conformance suite, stdlib admission records, package/workspace behavior, formatter, diagnostics, and generated docs agree.

### Gate G: Self-Hosting And Mechanization

Mechanized proof is after self-hosting by design.

Done when Kyokai compiler components are written in Kyokai, the bootstrap path is documented, and the proof assistant artifacts build against the chosen formalization.

## 7. Build Order

The phases below are ordered by dependency, not by excitement. Some implementation work can run in parallel after its input contracts exist, but no phase may bypass its gates by treating unspecified behavior as compiler policy.

### Phase 0: Repository Baseline And Decision Audit

**Purpose:** Establish the workspace facts and remove stale planning assumptions before implementation churn begins.

**Dependencies:** None.

**Subparts:**

0.1. Verify the initial decision batch has been extracted into public accepted shape or explicitly left as historical-only context. **Status: complete.** `kyokaidecided.md` now holds the accepted-shape extraction used for spec writing.

0.2. Record the source-of-truth order in this file and in public workflow docs. **Status: complete.** Current order is `kyokaispec/` once written, then `kyokaidecided.md`, then `Kyokaishape.md`, then linked public discussions/PRs, then `phase.md` for implementation order only.

0.3. Inventory the active local trees and assign each one a purpose. This means documenting what `lib/`, `standard/`, `kyokaispec/`, `test/`, `test-programs/`, `langtest/`, `bfetch/bfetchaust/`, and upstream Austral references are used for. It does not require moving them. **Status: complete enough for Phase 0.** The active Kyokai development files now live under `kyokailang/kyokai/`; deeper compiler-pass inventory belongs to later implementation phases.

0.4. Identify stale references in public docs, especially inherited Austral text, old path names, and scratch-note wording. **Status: mostly complete for Phase 0.** The root README, `kyokaispec/README.md`, and `kyokaispec/SPEC_COMPILER_TRACE.md` now state the inherited/staging status clearly. Remaining stale inherited docs can be cleaned as they are touched.

0.5. Create or update a D-point-to-work-area map covering parser, resolver, type checker, linearity checker, code generation, runtime, stdlib, package manager, and tooling. This is not only `Kyokaishape.md`: `Kyokaishape.md` tracks live/new D-points, while `kyokaidecided.md` and future `kyokaispec/` trace accepted decisions to implementation areas. **Status: partially complete through the `kyokaidecided.md` maturity tracker; still needs a spec-facing traceability index.**

0.6. Keep the extracted Kyokai spec under `kyokaispec/`. Inherited Austral spec material may be used as a seed, but Kyokai spec text must be reviewed and rewritten before it becomes normative. **Status: path chosen; Kyokai spec text not started.**

**Done when:**

- `phase.md` no longer contains stale D-point meanings.
- Every major local reference tree has an assigned purpose.
- The spec extraction target path is chosen.
- Stale path references are either fixed or tracked.
- Implementation work has a visible dependency map back to public shape/spec sections.

**May run in parallel with:** Early compiler inventory and test-harness inventory.

**Must not do:** Rewrite language behavior while calling it cleanup.

### Phase 1: Normative Kyokai Specification Extraction

**Purpose:** Convert the public accepted shape into maintainable normative specs under `kyokaispec/`.

**Dependencies:** Phase 0.

**Subparts:**

1.1. Create the Kyokai language spec structure under `kyokaispec/`. Use inherited Austral spec material as formatting and organization prior art only after checking it against actual Austral compiler behavior.

1.2. Extract lexical syntax, comments, docstrings, literals, numeric suffixes, keywords, operators, terminators, and source-file structure.

1.3. Extract declarations: modules, interfaces/bodies, records, unions, extern/packed records, type aliases, newtypes, typeclasses, associated types, capability declarations, constants, functions, tests, and visibility.

1.4. Extract expression and statement semantics: evaluation order, `let`, assignment, `case`, `if`, loops, `while let`, `for-in`, `let...else`, `or return`, `defer`, `errdefer`, `debug`, `panic`, `require`, `ensure`, and `old`.

1.5. Extract the type system: `Free`, `Linear`, `Auto`, region behavior, borrow types, reference syntax, field auto-deref, auto-reborrow, linear destructuring, `Never`, optional/result shapes, generics, const generics, and monomorphization.

1.6. Extract capability and authority rules: `RootCapability`, ordinary capabilities, task-transfer classifications, unsafe capability access, sealed capabilities, and authority flow across FFI and tasks.

1.7. Extract FFI and unsafe rules: `foreign`, `pragma Unsafe_Module`, `UnsafeCapability`, `unsafe contract ... audit;`, raw pointer/address rules, volatile, inline assembly, ABI records, sum-type boundary wrappers, ownership transfer wrappers, and failure contracts.

1.8. Extract concurrency rules: 1:1 OS-thread tasks, task groups, spawn capture lists, join, cancellation, SPSC channels, select, atomics, memory orders, happens-before edges, mutexes, `RwLock`, `Poller`, `SignalWatcher`, broker patterns, and process supervision.

1.9. Extract runtime-failure semantics: TPOE, `panic`, runtime-fatal/internal failures, stack overflow detection, OOM, failed assertions/contracts, arithmetic errors, and the rule that TPOE/panic are not catchable in process.

1.10. Extract backend and layout rules: C backend contract, future LLVM contract, record layout, packed layout, extern layout, endian transforms, C/LLVM UB avoidance, debug info, linking, binary outputs, and target profiles.

1.11. Extract the toolchain spec: `kyokai check`, `build`, `run`, `test`, `fmt`, `doc`, `lsp`, `audit`, `bench`, `repl`, package/workspace manifests, lockfiles, `.koi` artifacts, target triples, profiles, generated artifacts, diagnostics, JSON output, warning suppression, lints, and semver checking.

1.12. Extract the stdlib admission and contract spec: D229 admission criteria, D231 crypto policy, D232 numerical accuracy contracts, allocator policy, formatting, `StandardError`, collections, OS APIs, capabilities, and transitional FFI tracking.

1.13. Add a traceability index from each public D-point or accepted-shape entry to the normative spec section that owns it.

1.14. Keep historical research notes out of the final spec unless they are needed for rationale.

**Done when:**

- Gate A is closed.
- The extracted Kyokai spec can be read without needing informal planning context for normative behavior.
- Every D-point has a traceability destination.
- The spec has no phrases like "implementation-defined" unless the implementation choice is itself explicitly bounded and observable.
- The Austral spec material has either been forked into Kyokai wording or marked as inherited reference only.

**May run in parallel with:** Phase 2 research writing and compiler pass inventory.

**Must not do:** Treat inherited Austral spec text as authoritative when compiler source/tests disagree.

### Phase 2: Sequential `lambda_K` Core Calculus And Paper Proof

**Purpose:** Discharge the D143/D241 pre-`v1.0` proof obligation for Kyokai's sequential safety core.

**Dependencies:** Phase 1 core-language extraction should be far enough along to avoid proving the wrong language. Full toolchain spec extraction is not a blocker.

**Subparts:**

2.1. Freeze the first calculus scope in `kyokailang/kyokaicalculus/lambda_k_research.md` or a successor paper document.

2.2. Define the core syntax for variables, values, first-order functions, `let`, explicit consumption, borrow creation, borrow end, checked primitive operations, explicit TPOE, and minimal union/case support if needed.

2.3. Define core types and universes: unrestricted/free values, linear resources, immutable borrows, mutable borrows, base integers/bools/unit, and any minimal optional/result form included in the core.

2.4. Define typing contexts explicitly: declaration context, unrestricted context, linear context, region/borrow context, and any dynamic-store typing needed by the semantics.

2.5. Define static rules for linear use, no drop, no duplicate, borrow exclusivity, borrow lifetime, capability-as-linear-value behavior, function calls, and checked operations.

2.6. Define dynamic semantics with deterministic evaluation order and terminal configurations for values, normal steps, TPOE, and runtime-fatal states if included.

2.7. Prove preservation for all included reduction rules.

2.8. Prove progress-or-defined-failure: a well-typed configuration is a value, can step, or is in a defined terminal failure state such as TPOE.

2.9. Write the surface-to-core mapping for included features, especially `require`, `ensure`, `old`, UFCS-as-call, checked arithmetic, explicit cleanup states, and simple pattern control flow.

2.10. List exclusions explicitly: modules, packages, typeclasses, generics, FFI, allocators, formatting, OS, concurrency, atomics, channels, backend lowering, and unsafe reasoning.

**Done when:**

- Gate B is closed.
- The proof document is reviewable without relying on informal plan prose.
- Excluded features are named as future proof extensions, not hidden assumptions.
- The compiler's core IR/elaboration plan can point at the calculus for the sequential ownership core.

**May run in parallel with:** Parser/frontend bring-up after enough surface syntax is extracted.

**Must not do:** Try to prove the whole language in the first paper proof.

### Phase 3: Compiler Fork Identity And Frontend Surface Bring-Up

**Purpose:** Turn the Austral-shaped compiler into a Kyokai compiler at the source-language boundary.

**Dependencies:** Phase 1 lexical/syntax extraction. Phase 2 may be in progress.

**Subparts:**

3.1. Rename command, diagnostics, namespace defaults, standard-library roots, editor metadata, examples, and test names from Austral to Kyokai where they are no longer reference-only.

3.2. Establish file extensions and module pairing rules for `.kyo`, `.kai`, generated interfaces, and any compatibility/migration mode.

3.3. Implement lexer changes for Kyokai keywords, comments, docstrings, numeric literal separators, numeric literal suffixes, string/raw/code-point/byte literals, and rejected inherited tokens.

3.4. Implement parser support for Kyokai declaration terminators and semantic block terminators: `qed`, `build`, `seal`, `spec`, `od`, `fi`, `esac`, `join`, `pick`, `audit`, and other decided closures.

3.5. Implement parser support for major surface constructs: UFCS dot calls, record/union construction, pattern matching, `let...else`, `or return`, `while let`, `for-in`, `defer`, `errdefer`, `taskgroup`, `spawn`, `select`, `debug`, contracts, unsafe contract blocks, and inline tests.

3.6. Preserve source spans and enough AST metadata for D239 implicit-completion auditing and diagnostics.

3.7. Add parser negative tests for old Austral forms that Kyokai deliberately rejects unless a migration mode is explicitly selected.

3.8. Create early `langtest/kyokai` sync rules: examples must compile against decided syntax or be labeled as historical comparison artifacts.

**Done when:**

- Basic Kyokai source files parse with stable source spans.
- Old Austral syntax is not accidentally accepted as Kyokai syntax.
- Parser tests cover accepted and rejected forms.
- CLI identity and module roots no longer present as accidental Austral branding in user-facing paths.

**May run in parallel with:** Toolchain skeleton work that only depends on parser invocation.

**Must not do:** Keep inherited syntax because it is easier unless the plan says Kyokai kept it.

### Phase 4: Name Resolution, Imports, Packages, And Interface Artifacts

**Purpose:** Implement Kyokai's compilation unit, package, visibility, and lookup rules before deep typing makes them harder to change.

**Dependencies:** Phase 1 declaration/toolchain extraction and Phase 3 parser support.

**Subparts:**

4.1. Implement module identity, package identity, module root layout, and import path resolution.

4.2. Implement `.kyo`/`.kai` interface-body pairing rules and generated interface rules where specified.

4.3. Implement `public`, `private`, `internal`, package boundary checks, import collision errors, and explicit `as` renaming.

4.4. Implement ordinary name lookup exactly as specified before UFCS fallback is added.

4.5. Implement D254 receiver-module UFCS fallback only after ordinary lookup finds no candidate.

4.6. Implement package/workspace manifests, lockfile reading, workspace member discovery, target/profile selection, and source file discovery.

4.7. Implement `.koi` interface artifact generation and consumption for package-level separate compilation.

4.8. Implement package-level generic/typeclass metadata requirements for downstream instantiation under the D82/D82a/D82b family.

4.9. Add cycle detection, duplicate logical-module detection, and reproducible artifact identity checks.

**Done when:**

- `kyokai check` can load a package/workspace and resolve modules without manual file ordering.
- Import ambiguity is an error at the import site.
- UFCS does not create global ADL or dependency-wide method search.
- `.koi` artifacts are deterministic enough to use in tests.

**May run in parallel with:** Early toolchain CLI plumbing and formatter work.

**Must not do:** Let import order or dependency order silently choose winners.

### Phase 5: Type System, Elaboration Pipeline, And Core IR

**Purpose:** Build the semantic frontend around the exact D238 lowering order.

**Dependencies:** Phases 1, 3, and 4. Phase 2 should be far enough to guide the core IR shape for the sequential subset.

**Subparts:**

5.1. Define the compiler's typed core IR and explicitly document which surface constructs elaborate into it.

5.2. Implement D238 pass order:

1. Parse source into surface AST.
2. Perform syntax-only lowering that does not need type facts.
3. Resolve names and call shapes.
4. Type-check and elaborate expressions.
5. Insert D87 implicit completions as explicit elaboration nodes.
6. Record every implicit completion in the compiler-maintained registry.
7. Run the D239 tautology-check pass over those elaboration nodes.
8. Lower typed sugar.
9. Run linearity, borrow, capability, contract, unsafe, and backend-readiness checks.

5.3. Implement universes and kind rules for `Free`, `Linear`, and `Auto`.

5.4. Implement records, unions, aliases, newtypes, fixed arrays, optional/result shapes, `Never`, and built-in types.

5.5. Implement generics, const generics, monomorphization ownership, static dispatch, typeclasses, associated types, default methods where specified, coherence, and orphan rules.

5.6. Implement literal typing, D12 bidirectional literal inference, D261 suffix typing, and all numeric representability checks.

5.7. Implement expression typing for calls, UFCS, indexing, bitwise keyword operators, arithmetic variants, comparisons, boolean operations, ranges, casts/conversions, and format DSL checks.

5.8. Implement pattern typing for `case`, `let...else`, `while let`, destructuring, exhaustive matches, and no-shadowing.

5.9. Implement contract typing for `require`, `ensure`, `old`, and TPOE-producing checks.

5.10. Implement typed sugar lowering for `or return`, `or break`, `or continue`, `for-in`, `while let`, UFCS, field auto-deref, and auto-reborrow.

5.11. Build a conformance view that can show the elaborated core for tests and debugging.

**Done when:**

- Gate C's frontend half is complete.
- Type errors and ambiguity errors are deterministic and diagnostic-coded.
- Elaboration output exposes implicit completions instead of hiding them.
- The sequential subset matches the `lambda_K` surface-to-core plan where applicable.

**May run in parallel with:** Initial stdlib interface drafting, but not with final stdlib admission.

**Must not do:** Run linearity on unelaborated sugar or let implicit completions bypass D239.

### Phase 6: Linearity, Borrows, Capabilities, Contracts, And Unsafe Checks

**Purpose:** Enforce the safety properties that make Kyokai distinct.

**Dependencies:** Phase 5 typed elaboration.

**Subparts:**

6.1. Port and adapt Austral's linearity checker only after reading its actual source and tests.

6.2. Implement linear states for available, moved/consumed, borrowed, deferred, and errdeferred values.

6.3. Implement explicit consumption rules, no implicit drop, no duplicate use, linear field movement, linear destructuring, and all exit-path consumption checks.

6.4. Implement immutable and mutable borrow creation, anonymous-by-default regions, explicit region syntax where needed, borrow end, and conflict checking.

6.5. Implement auto-reborrow and field auto-deref only as elaborated nodes that the checker can see.

6.6. Implement D240 conformance tests for auto-reborrow/read-reborrow success, failure, nested calls, field paths, temporary lifetimes, and linear payloads.

6.7. Implement `defer` and `errdefer` ordering, checker states, cleanup insertion, and interaction with `or return` / structured error exits.

6.8. Implement capability declarations as sealed authority values that unsafe code cannot forge.

6.9. Implement task-transfer and task-local classifications for types, capabilities, and runtime handles.

6.10. Implement unsafe-module checking: `pragma Unsafe_Module`, `UnsafeCapability`, raw operations, volatile, asm, FFI, and required `unsafe contract ... audit;` coverage.

6.11. Implement contract checks for `require`, `ensure`, asserts, bounds checks, overflow checks, and TPOE categories.

**Done when:**

- Safe code cannot leak, duplicate, forge, or use-after-consume linear resources.
- Borrow conflicts are caught before code generation.
- Deferred cleanup has exact ownership states and order.
- Unsafe operations without audit coverage fail compilation.
- Capability authority cannot be constructed from raw bits by safe or unsafe user code.

**May run in parallel with:** Backend design for already-checked operations.

**Must not do:** Treat unsafe as a blanket escape hatch from capability, ownership, or audit rules.

### Phase 7: Runtime Semantics And C Backend Safety

**Purpose:** Make generated code preserve Kyokai semantics without relying on C undefined behavior.

**Dependencies:** Phase 5 for typed IR and Phase 6 for checked ownership/capability state.

**Subparts:**

7.1. Define backend lowering contracts for every core operation in the extracted spec.

7.2. Implement D84 termination categories: normal return, recoverable `Result` values, TPOE, `panic`, runtime-fatal/internal failure, and OS process exit.

7.3. Implement TPOE and panic paths as hard process termination, not recoverable exceptions.

7.4. Implement stack overflow detection for hosted and freestanding targets using guard pages, probes, bounds checks, or an equivalent documented mechanism before corruption.

7.5. Implement integer operations with defined overflow, checked, wrapping, saturating, modular, shift, rotate, divide-by-zero, and representability behavior.

7.6. Implement floating semantics, NaN handling, classification, conversion, and math edge cases according to admitted contracts.

7.7. Implement record, union, enum/tag, packed-record, extern-record, and address lowering without layout folklore.

7.8. Implement memory operations, allocator calls, pointer/address operations, volatile operations, atomics, fences, and barriers with target contracts.

7.9. Implement C backend emission patterns that avoid signed overflow UB, invalid shift UB, invalid aliasing assumptions, invalid enum values, uninitialized reads, invalid pointer provenance assumptions, and fallthrough into unreachable states.

7.10. Add backend conformance tests that inspect generated C for known-dangerous patterns and execute edge cases under sanitizers where practical.

7.11. Keep LLVM backend out of the critical path until the C backend semantic contract is stable.

**Done when:**

- Gate D is closed for the C backend.
- Generated C is boring, explicit, and auditable.
- Safe Kyokai operations never depend on C UB for optimization or convenience.
- Runtime-fatal and TPOE paths are visible in generated code and tests.

**May run in parallel with:** Tooling and stdlib work that only depends on stable ABI/runtime hooks.

**Must not do:** Use `__builtin_unreachable`, signed-overflow assumptions, unchecked C shifts, or layout punning as semantic shortcuts for safe Kyokai.

### Phase 8: Toolchain Skeleton, Diagnostics, Formatter, And Test Harness

**Purpose:** Build the operator and contributor tools needed to keep implementation honest.

**Dependencies:** Phases 3 and 4 for package loading; Phase 5 for typed checks; parts can begin earlier as CLI shells.

**Subparts:**

8.1. Implement `kyokai check` as the first reliable command.

8.2. Implement `kyokai build`, `run`, and profile/target selection after package loading and backend invocation are stable.

8.3. Implement `kyokai test` with inline test blocks, production-build exclusion, test filtering, failure reporting, and no hidden test-only language behavior.

8.4. Implement `kyokai fmt` as deterministic, idempotent, and zero-configuration.

8.5. Implement diagnostics with stable diagnostic codes, source spans, notes, suggestions, warning categories, per-project suppression, and JSON output.

8.6. Implement `kyokai doc` from docstrings and interfaces, including generated stdlib documentation.

8.7. Implement `kyokai audit` for unsafe modules, package unsafe surfaces, capability requirements, and dependency review.

8.8. Implement lints inside the compiler, including ownership-signaling naming warnings.

8.9. Implement conformance-test runner layout: parser tests, type tests, negative tests, backend execution tests, diagnostic goldens, and stdlib contract tests.

8.10. Sync `langtest/kyokai` examples against the decided spec and mark obsolete comparison files clearly.

**Done when:**

- Contributors can run one command to check language conformance tests.
- Diagnostics are stable enough for golden tests.
- Formatter output is canonical.
- Docs and audit output are generated from compiler-understood facts, not hand-maintained guesses.

**May run in parallel with:** Standard library implementation once core compiler checks are usable.

**Must not do:** Let tooling hide compiler errors or provide behavior the compiler cannot enforce.

### Phase 9: Core Standard Library Foundation

**Purpose:** Admit the safe pure and low-level foundation modules required by real programs.

**Dependencies:** Phase 5 for typing, Phase 6 for linearity, Phase 7 for runtime/backends, Phase 8 for tests.

**Subparts:**

9.1. Establish the stdlib module layout under `Kyokai.*` and retire inherited `Standard.*` naming from public Kyokai APIs.

9.2. Implement `Unit`, `Bool`, fixed integers, floats, `Index`, `Optional`, `Result`, `Never`, `Pair`, `Triple`, and domain-named result types where needed.

9.3. Implement `Span`, `Array`, `Buffer`, `String`, `StaticString`, byte strings, UTF-8 validation, ASCII helpers, Unicode base helpers, and string/span comparison.

9.4. Implement `Equality`, `Order`, `Hashable`, `Displayable`, `FormatSink`, `StandardError`, `Parsable`, and related typeclass instances.

9.5. Implement allocator interfaces and core allocators with explicit value-level allocator choice and no hidden default allocator.

9.6. Implement formatting: allocating `format`, sink-based `writeFmt`, interpolation rules, checked formatter DSL, and exact allocation/failure behavior.

9.7. Implement integer math, bitwise helpers, endian transforms, byte encode/decode, sorting, hashing, and pure algorithms without FFI unless admission policy says otherwise.

9.8. Implement floating math under D232 accuracy contracts, with test vectors and explicit edge-case behavior.

9.9. Implement collections: linear-safe `HashMap`, sets, queues/deques where admitted, iterator support, and explicit destroy/drain behavior for linear payloads.

9.10. Implement eager iterator helpers, fused iteration contracts, and linear iterator finalization under D249.

9.11. Apply D229 admission records to every module: contract, edge cases, oracle/test source, unsafe/FFI policy, compatibility boundary, and release status.

**Done when:**

- Real text, buffer, collection, error, formatting, and math code can be written without local shims.
- Pure computation modules are safe native Kyokai unless explicitly admitted otherwise.
- Every admitted API has tests for edge cases and linear cleanup behavior.
- `bfetch`-style buffer and string logic no longer needs custom runtime modules for pure operations.

**May run in parallel with:** OS stdlib wrappers after allocator/buffer/string contracts stabilize.

**Must not do:** Wrap libc/libm by default for pure computation just because it is faster to bootstrap.

### Phase 10: OS, FFI Boundary, Capabilities, And Runtime Standard Library

**Purpose:** Provide safe systems APIs at the OS/hardware boundary without smuggling authority or ownership through FFI.

**Dependencies:** Phases 6, 7, 8, and core Phase 9 allocator/string/buffer types.

**Subparts:**

10.1. Implement `RootCapability`, terminal, filesystem, process, environment, clock, random, network, signal, and platform capabilities.

10.2. Implement paths, OS strings, C strings, byte spans, file descriptors/handles, sockets, process IDs, exit statuses, and platform error types.

10.3. Implement safe file I/O: open/read/write/close/stat/readlink/access/directory operations with linear handles and explicit partial-read/write behavior.

10.4. Implement process spawning, pipes, dup/redirect operations, wait/status observation, supervised worker process patterns, and no in-process panic/TPOE catching.

10.5. Implement networking: TCP/UDP sockets, address parsing, DNS policy if admitted, listener/connection lifecycle, and explicit blocking/non-blocking operations.

10.6. Implement `Poller` and readiness-backed APIs. Blocking cancellation is cooperative only through Poller-backed or explicitly readiness-backed operations.

10.7. Implement `SignalWatcher` as the safe signal surface. Raw handler registration remains unsafe-only. Synchronous fault signals are runtime-fatal.

10.8. Implement volatile/MMIO APIs for the closed legal type domain under unsafe operation-level contracts.

10.9. Implement FFI wrappers for OS boundaries with explicit ABI records, ownership-transfer wrappers, sum-type translations, and unsafe audit contracts.

10.10. Track transitional FFI wrappers under D230 with replacement criteria and do not let them become unreviewed permanent stdlib internals.

**Done when:**

- Gate E can be closed for file/process/terminal/buffer workloads.
- OS authority is visible through capabilities.
- FFI ownership and ABI boundaries are wrapper-modeled and audited.
- Blocking behavior is visible in API names or explicit operation contracts.

**May run in parallel with:** Concurrency primitives after `Poller`, atomics, and capability transfer rules are implemented.

**Must not do:** Add ambient globals for OS authority, hidden runtime reactors, or raw FFI shortcuts in safe APIs.

### Phase 11: Concurrency, Atomics, Channels, And Synchronization

**Purpose:** Implement Kyokai's explicit structured concurrency model.

**Dependencies:** Phase 6 task-transfer/capability checks, Phase 7 memory/runtime semantics, Phase 10 `Poller` and OS threading hooks where needed.

**Subparts:**

11.1. Implement 1:1 OS-thread task creation with explicit spawn failure handling or pre-reserved task capacity.

11.2. Implement `taskgroup do ... join;` as the only safe structured spawn boundary.

11.3. Implement spawn capture lists: by-value copy for `Free`, by-value transfer for eligible `Linear`, immutable borrow for allowed shared forms, and no `&!` capture.

11.4. Implement task-boundary classifications: `task_transfer`, `task_local`, and standard-library handle/capability contracts.

11.5. Implement memory orders, atomics, fences, and the closed happens-before inventory from D247.

11.6. Implement `Mutex[T]`, `RwLock[T]`, poisoning policy if any, lock lifecycle, lock guards, and linear payload interaction.

11.7. Implement SPSC channels with explicit capacity constructors, close/drain behavior, blocking and non-blocking operation names, and failure types that return linear values on failed send.

11.8. Implement `select ... when ... do ... pick;` for multi-channel waiting with explicit non-priority selection semantics.

11.9. Implement broker-pattern library helpers over SPSC channels without adding MPSC/MPMC/broadcast endpoint primitives.

11.10. Implement cancellation tokens and cooperative cancellation for Poller-backed operations.

11.11. Add stress tests for ownership transfer, task-local rejection, channel closure/drain, select fairness/non-priority semantics, atomics, locks, and process-fatal panic/TPOE behavior.

**Done when:**

- Safe Kyokai supports structured parallel programs without shared mutable data races.
- Every task lifetime and blocking join point is visible in source.
- Channels remain SPSC primitives; fan-in/fan-out are explicit broker tasks.
- Memory-order behavior is backend-independent and tested.

**May run in parallel with:** Networking/event-loop examples after `Poller` is stable.

**Must not do:** Add hidden green threads, cloneable channel endpoints, implicit join on `od`, or Rust-style `Send`/`Sync` auto traits.

### Phase 12: Package Manager, Index, Build Artifacts, And Ecosystem Tooling

**Purpose:** Make Kyokai packages reproducible, auditable, and usable outside the repository.

**Dependencies:** Phase 4 package/workspace basics, Phase 8 tooling, Phase 9 stdlib basics.

**Subparts:**

12.1. Implement `kyokai add`, `update`, `publish`, package cache, lockfile update, and pinned Git revision behavior.

12.2. Implement official read-only package index support, package discovery, metadata fetching, and append-only yanks that affect new resolution but preserve existing lockfiles.

12.3. Implement package audit surfaces for unsafe modules, capabilities, FFI, build-time code generation, and dependency authority.

12.4. Implement SemVer checking for public API changes and stdlib compatibility policy under language editions.

12.5. Implement manifest-declared build-time code generation and `@embedFile` without hidden execution authority.

12.6. Implement release artifact generation, checksums, provenance, official CI setup action, OCI images, and portable installation contract.

12.7. Implement package docs publishing, search indexing, and generated API reference integration.

12.8. Implement Compiler Explorer integration and sandbox-runner contract. Hosted playground remains optional until the sandbox contract is real.

**Done when:**

- External packages can be pinned, fetched, audited, built, tested, documented, and reproduced.
- Yanks, lockfiles, and SemVer checks obey the spec.
- Build-time code generation has explicit authority and dependency tracking.

**May run in parallel with:** LLVM backend and self-hosting preparation after core package artifacts are stable.

**Must not do:** Allow moving branches, unaudited build scripts, or package resolution that changes existing lockfiles behind the user's back.

### Phase 13: LLVM Backend, Cross Compilation, Optimization, And Debuggability

**Purpose:** Add the optimizing backend and target matrix after the semantic backend contract is already stable.

**Dependencies:** Phase 7 C backend safety, Phase 8 conformance harness, Phase 4 target/profile model.

**Subparts:**

13.1. Define the LLVM IR lowering contract in the same semantic terms as the C backend.

13.2. Implement target triples, data layouts, calling conventions, debug info, symbol visibility, linking modes, and build profile integration.

13.3. Lower integer, float, pointer/address, record, union, packed, volatile, atomic, panic/TPOE, stack-check, and FFI operations without relying on LLVM poison/undef folklore.

13.4. Run C backend and LLVM backend against the same conformance suite.

13.5. Add cross-compilation support for tiered targets only after target contracts define sizes, alignment, atomics, stack behavior, libc/OS boundary expectations, and linker behavior.

13.6. Add optimization passes only when they preserve elaborated linear/capability semantics and do not erase required checks.

13.7. Add debugger-quality source maps and DWARF behavior matching the D27 debug contract.

**Done when:**

- C and LLVM backends agree on conformance tests for each supported target tier.
- LLVM lowering does not introduce UB/poison assumptions forbidden by Kyokai semantics.
- Cross-compilation failures are explicit target unsupportedness, not silent miscompilation.

**May run in parallel with:** Self-hosting compiler-component work once frontend artifacts are stable.

**Must not do:** Use LLVM as a semantic oracle for behavior Kyokai has already defined differently.

### Phase 14: Self-Hosting Transition

**Purpose:** Move compiler components into Kyokai once the language and stdlib are strong enough to support compiler implementation.

**Dependencies:** Gate F, package artifacts, core stdlib, parser/typechecker maturity, and stable bootstrap compiler.

**Subparts:**

14.1. Choose self-hosting slices with low semantic risk first: lexer, parser helpers, diagnostic rendering, source maps, formatter, manifest parsing, or test harness support.

14.2. Keep the OCaml/Austral-derived compiler as a reference bootstrap while Kyokai components grow.

14.3. Track transitional FFI or OCaml bridge code under D230 replacement policy.

14.4. Port pure computation compiler utilities to safe Kyokai before OS/backend boundary code.

14.5. Add cross-check tests where OCaml and Kyokai implementations process the same input and produce the same typed/elaborated result.

14.6. Move larger frontend passes only after the Kyokai implementation can express the required invariants cleanly.

14.7. Document the bootstrap chain: what compiler builds what source, which artifacts are trusted, and how reproducibility is checked.

**Done when:**

- A meaningful compiler slice is written in Kyokai and built by the Kyokai toolchain.
- The bootstrap path is reproducible and documented.
- Transitional bridges have owners and removal criteria.

**May run in parallel with:** Mechanized proof preparation after the core calculus is stable.

**Must not do:** Rewrite difficult compiler passes in Kyokai before the language can express their invariants and tests.

### Phase 15: Mechanized Proof And Long-Term Governance

**Purpose:** Move from paper proof to machine-checked proof and keep the spec/compiler relationship healthy.

**Dependencies:** Phase 2 paper proof and Phase 14 self-hosting progress. The proof assistant choice should be made when the project is ready to sustain the proof.

**Subparts:**

15.1. Choose the proof assistant. Coq is the current likely first choice, but the final choice should be based on Linux availability, ecosystem fit, contributor ability, and maintainability.

15.2. Mechanize the sequential `lambda_K` syntax, static semantics, dynamic semantics, and soundness theorem.

15.3. Add proof builds to CI.

15.4. Extend the formal model in separate layers only after the core mechanization is stable: generics, modules, typeclasses, contracts, unsafe/FFI boundary models, concurrency, memory model, and backend simulation.

15.5. Establish spec-change governance: any semantic change updates the spec, conformance tests, implementation, and proof impact notes in the same tracked change.

15.6. Establish edition governance under the already-decided edition/release policy rather than inventing a second compatibility system.

15.7. Keep planning/rationale material separate once the specs own normative behavior.

**Done when:**

- Gate G is closed for the sequential core.
- Proof artifacts build in CI.
- New semantic changes have an explicit proof-impact and conformance-impact path.

**May run in parallel with:** Later backend and stdlib growth after the core proof is stable.

**Must not do:** Let the mechanized proof become stale ceremonial evidence disconnected from the compiler and spec.

## 8. Cross-Phase Ordering Rules

These rules are the short version of the dependency graph.

| Work | Must Happen Before | Reason |
| --- | --- | --- |
| D-point traceability | spec extraction closure | No behavior should vanish during extraction. |
| Language spec extraction | broad compiler rewrites | The compiler should not encode guessed semantics. |
| `lambda_K` paper proof | public conformance release | D143/D241 require it before `v1.0`. |
| Parser/source spans | diagnostics, formatter, tooling | Tooling needs stable syntax trees. |
| Name/package resolution | typeclass and UFCS behavior | Lookup rules control ambiguity and fallback. |
| D238 pipeline | linearity/borrow/capability checking | Checks must run on elaborated semantics. |
| D239 registry | accepted implicit completions | Implicit completions must be auditable. |
| D240 matrix | auto-reborrow confidence | Ergonomics cannot weaken borrow soundness. |
| Linearity and borrow checks | backend codegen | Codegen must not receive illegal programs. |
| Backend UB contract | optimization and LLVM | Optimization cannot define semantics. |
| Allocator model | containers and strings | Storage ownership must be explicit first. |
| Buffer/string/span | POSIX/network/std formatting | OS APIs and formatting need byte/text foundations. |
| Capabilities | OS stdlib | Authority cannot be retrofitted safely. |
| Atomics/memory model | channels/locks/tasks | Concurrency primitives need HB and ordering rules. |
| `Poller` | cancellation and event loops | Cancellation is readiness-backed, not magic. |
| Package artifacts | package manager and self-hosting | Reuse and reproducibility require stable artifacts. |
| C backend conformance | LLVM backend | LLVM must match already-defined semantics. |
| Self-hosting | mechanized proof priority shift | D143 places mechanization after self-hosting. |

## 9. Parallel Work Lanes

Some work can proceed concurrently if dependencies are respected.

### Lane A: Specification And Proof

- Extract normative spec sections.
- Write D-point traceability index.
- Expand `lambda_K` research into the paper proof.
- Keep calculus exclusions explicit.

This lane blocks public conformance claims.

### Lane B: Compiler Frontend

- Parser and lexer changes.
- Module/package resolution.
- Typed elaboration and core IR.
- Linearity/borrow/capability checks.

This lane blocks safe execution.

### Lane C: Backend And Runtime

- Runtime failure paths.
- C backend UB avoidance.
- Stack checks.
- Target contracts.
- Later LLVM lowering.

This lane blocks executable trust.

### Lane D: Standard Library

- Core types and containers.
- Allocators.
- Formatting and diagnostics.
- Math/numerics.
- OS capabilities and safe wrappers.
- Concurrency primitives.

This lane blocks real programs.

### Lane E: Tooling And Ecosystem

- `check/build/test/fmt/doc/audit/lsp`.
- Package manager and index.
- CI setup.
- Playground/compiler explorer.
- Documentation publishing.

This lane blocks external adoption.

## 10. Feature Admission Checklists

### 10.1 Language Feature Checklist

A language feature is not complete until:

- normative text exists
- syntax and grammar are written
- type rules are written
- runtime behavior is written
- error behavior is written
- interaction with linearity, borrows, capabilities, contracts, unsafe, and backend lowering is either defined or explicitly irrelevant
- parser tests exist
- type/checker tests exist
- negative tests exist
- diagnostic codes exist for common failure modes
- conformance examples exist

### 10.2 Standard Library API Checklist

A stdlib API is not admitted until:

- its authority/capability requirements are stated
- ownership transfer and cleanup are stated
- allocator behavior is stated
- blocking behavior is visible in the name or contract
- error type and failure behavior are stated
- TPOE/runtime-fatal cases are stated
- edge cases are listed
- implementation policy is stated: safe native Kyokai, unsafe internal, FFI wrapper, or transitional FFI
- oracle/test-vector source is listed where applicable
- compatibility boundary is stated

### 10.3 Backend Lowering Checklist

A backend lowering is not complete until:

- source Kyokai operation is named
- target representation is documented
- C/LLVM UB hazards are listed
- mitigation strategy is implemented
- debug/source-span behavior is defined
- sanitizer or edge-case tests exist where practical
- behavior agrees with the interpreter/core semantics if such a runner exists

### 10.4 Unsafe/FFI Checklist

An unsafe or FFI surface is not complete until:

- it is inside an unsafe module when required
- it has `UnsafeCapability` access when required
- every unsafe operation is covered by an `unsafe contract ... audit;` block
- ownership transfer is wrapper-modeled
- raw FFI does not take or return Kyokai `Linear` values by value
- sum types are translated through explicit ABI records/tags
- failure modes are represented as values or documented fatal behavior
- safe wrapper tests cover success, failure, cleanup, and invalid input rejection

## 11. Status Tracker

Current status is planning/spec extraction. No implementation phase is complete just because the plan decisions are closed.

| Phase | Status | Blocking Gate |
| --- | --- | --- |
| Phase 0: Repository Baseline And Decision Audit | In progress: 0.1-0.4 complete enough for Phase 0; 0.5 traceability index still needed; 0.6 path chosen/spec not started | Gate A preparation |
| Phase 1: Normative Kyokai Specification Extraction | Not started | Gate A |
| Phase 2: Sequential `lambda_K` Core Calculus And Paper Proof | Research note exists | Gate B |
| Phase 3: Compiler Fork Identity And Frontend Surface Bring-Up | Not started | Parser/source gate |
| Phase 4: Name Resolution, Imports, Packages, And Interface Artifacts | Not started | Package/spec gate |
| Phase 5: Type System, Elaboration Pipeline, And Core IR | Not started | Gate C |
| Phase 6: Linearity, Borrows, Capabilities, Contracts, And Unsafe Checks | Not started | Safety checker gate |
| Phase 7: Runtime Semantics And C Backend Safety | Not started | Gate D |
| Phase 8: Toolchain Skeleton, Diagnostics, Formatter, And Test Harness | Not started | Conformance infrastructure |
| Phase 9: Core Standard Library Foundation | Not started | D229 admission |
| Phase 10: OS, FFI Boundary, Capabilities, And Runtime Standard Library | Not started | Gate E |
| Phase 11: Concurrency, Atomics, Channels, And Synchronization | Not started | Memory model/runtime gate |
| Phase 12: Package Manager, Index, Build Artifacts, And Ecosystem Tooling | Not started | Ecosystem gate |
| Phase 13: LLVM Backend, Cross Compilation, Optimization, And Debuggability | Not started | C backend stability |
| Phase 14: Self-Hosting Transition | Not started | Gate F |
| Phase 15: Mechanized Proof And Long-Term Governance | Not started | Gate G |

## 12. Near-Term Work Queue

The next concrete work should happen in this order.

1. Build the D-point traceability index from `Kyokaishape.md` and `kyokaidecided.md` into the spec tree.
2. Rewrite `kyokaispec/` from inherited Austral material into Kyokai spec text, starting with goals, syntax, modules, types, expressions, and linearity.
3. Extract lexical syntax, declarations, type system, and runtime semantics first.
4. Expand `lambda_k_research.md` into a paper outline with formal notation and the exact first proof scope.
5. Inventory the compiler passes in `lib/` against the D238 pipeline.
6. Create the first conformance directory shape for parser, type, linearity, backend, stdlib, and diagnostics tests.
7. Start compiler frontend changes only after the relevant spec sections are extracted.
8. Prioritize `kyokai check` before `kyokai build`, and `build` before package manager publishing.
9. Prioritize allocator, buffer, string/span, result/optional, and formatting before large OS or concurrency APIs.
10. Prioritize C backend correctness before LLVM backend work.

## 13. Changelog

### 0.2.0 - 2026-05-02

- Replaced the stale phase list with a full roadmap based on the decided Kyokai shape.
- Added source-of-truth order, maturity states, global gates, detailed phase breakdowns, cross-phase dependencies, feature admission checklists, and a near-term queue.
- Corrected stale D-point mappings by grounding recent work in D143/D241, D238/D239/D240, D228, D229-D232, D242-D262, and the current work-item section.
- Made spec extraction, Austral-spec-to-Kyokai-spec work, and the sequential `lambda_K` paper proof explicit early phases.

### 0.1.0 - historical

- Initial short execution-phase sketch. Superseded because it used stale D-point meanings after later decision closures.
