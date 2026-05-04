# Kyokai Code Standards

**Document Version:** 0.2.0  
**Last Updated:** 2026-05-02  
**Scope:** Kyokai compiler, runtime, standard library, generated code, tests, tools, examples, documentation-adjacent code, and future Kyokai source code

This is the code-quality handbook for Kyokai. It is not a short checklist. It is the standard for how code in this repository should be written, reviewed, split, tested, and maintained while Kyokai moves from an Austral-derived compiler into its own language, standard library, toolchain, and spec.

the compiler is mostly OCaml inherited from Austral, the generated backend currently emits C, the runtime and standard library touch OS and ABI boundaries, and future source will be Kyokai itself. The same engineering values still apply, but the concrete rules need to fit OCaml, Austral's architecture, and Kyokai's language philosophy.

The short version is this: write code that makes invariants visible. If a behavior matters, put it in the type, the module boundary, the pass boundary, the API contract, the diagnostic, or the test. Do not rely on magic.

---

## Table Of Contents

1. [Core Philosophy](#1-core-philosophy)
2. [Scope And Authority](#2-scope-and-authority)
3. [Reference Material](#3-reference-material)
4. [Kyokai-Specific Engineering Rules](#4-kyokai-specific-engineering-rules)
5. [Repository Layout And Ownership](#5-repository-layout-and-ownership)
6. [OCaml Compiler Code](#6-ocaml-compiler-code)
7. [Compiler Pass Architecture](#7-compiler-pass-architecture)
8. [Data Modeling And Invariants](#8-data-modeling-and-invariants)
9. [Error Handling And Diagnostics In The Compiler](#9-error-handling-and-diagnostics-in-the-compiler)
10. [Linearity, Borrowing, And Capability Checks](#10-linearity-borrowing-and-capability-checks)
11. [Backend, Runtime, FFI, And Generated C](#11-backend-runtime-ffi-and-generated-c)
12. [Kyokai Standard Library Code](#12-kyokai-standard-library-code)
13. [Future Kyokai Source Code](#13-future-kyokai-source-code)
14. [Tests And Conformance](#14-tests-and-conformance)
15. [Documentation In And Around Code](#15-documentation-in-and-around-code)
16. [Dependencies, Tooling, And Build Hygiene](#16-dependencies-tooling-and-build-hygiene)
17. [Performance, Allocation, And Memory Discipline](#17-performance-allocation-and-memory-discipline)
18. [Concurrency, Blocking, And Task Boundaries](#18-concurrency-blocking-and-task-boundaries)
19. [Reviews And Change Shape](#19-reviews-and-change-shape)
20. [Security, Safety, And Platform Boundaries](#20-security-safety-and-platform-boundaries)
21. [Concrete Patterns](#21-concrete-patterns)
22. [What Not To Do](#22-what-not-to-do)
23. [Command Checklist](#23-command-checklist)
24. [Summary](#24-summary)

---

## 1. Core Philosophy

### 1.1 Clarity Beats Cleverness

Kyokai code should be readable under pressure. A compiler bug usually appears far away from the line that caused it: a parser shortcut becomes a bad AST shape, a type-checker shortcut becomes a wrong diagnostic, a lowering shortcut becomes invalid C, and a runtime shortcut becomes a language-contract hole.

Rules:

- Prefer boring, explicit code over clever compression.
- Prefer names that state the invariant over names that merely state the representation.
- Prefer a little more structure over a shared helper that hides control flow.
- Prefer direct pattern matching over layers of callbacks when the state machine is easier to see directly.
- Prefer a small amount of duplication over an abstraction that blurs distinct language phases.

This does not mean writing verbose code. It means avoiding code where the reader has to infer the important rule from a side effect, a naming convention, or a helper buried in another pass.

### 1.2 Behavior First, Implementation Second

The compiler implementation must follow accepted Kyokai behavior. It must not become the source of truth by accident.

Rules:

- If behavior is accepted, link or cite the public shape/spec source in the PR when it is non-obvious.
- If behavior is experimental, keep it behind a non-default flag or branch-local work.
- If implementation reveals a missing semantic question, open or update a D-point instead of filling the gap silently.
- If a diagnostic, runtime check, lowering rule, or stdlib API depends on a subtle guarantee, write that guarantee down.

Good code is not just code that passes tests. Good Kyokai code makes it obvious which language rule it implements.

### 1.3 Make Illegal States Unrepresentable Where Practical

OCaml gives us variants, records, modules, signatures, phantom types, and abstract types. Use them. Do not move invalid compiler states around as strings, booleans, or partial records when a proper type would make the code harder to misuse.

Examples:

- Use a variant for a compiler phase state instead of a string tag.
- Use a `module_name` or `identifier` type instead of raw `string` after parsing.
- Use separate AST types for surface, typed, lowered, and monomorphic forms when those stages have different invariants.
- Use an abstract module interface to prevent callers from manufacturing an invalid environment.
- Use result variants for expected failures instead of magic values.

Do not over-model tiny local state. The rule is practical: if a wrong state would create a real compiler bug, encode the distinction.

### 1.4 Explicit Boundaries

Kyokai as a language is about visible ownership, authority, failure, allocation, blocking, and runtime behavior. The implementation should mirror that.

Important boundaries:

- source text to CST
- CST to AST
- AST to typed AST
- typed AST to checked typed AST
- checked typed AST to lowered IR
- lowered IR to generated C
- generated C to target compiler
- runtime/stdlib to OS and ABI
- public API to internal implementation

At each boundary, the code should answer:

- What invariants are required on input?
- What invariants are guaranteed on output?
- What failures can happen?
- What source location information is preserved?
- What language rule or toolchain rule does this boundary enforce?

### 1.5 Tests Are Part Of The Design

Tests are not only regression protection. In Kyokai, tests are executable evidence for language rules.

Rules:

- Parser syntax needs accept and reject tests.
- Type-system rules need positive, negative, and diagnostic tests.
- Linearity changes need branch, loop, borrow, move, defer, and failure-path tests where relevant.
- Backend changes need generated-code or runtime-behavior tests.
- Runtime/FFI changes need edge-case tests and target notes.
- Stdlib APIs need normal, boundary, and failure-contract tests.

When a behavior has no test, say why. If the reason is "the harness does not exist yet," add the missing harness to the roadmap or PR notes.

---

## 2. Scope And Authority

### 2.1 What This File Governs

This file governs code quality for:

- OCaml compiler code in `lib/`, `bin/`, tests, and support tools
- Kyokai runtime and backend glue
- generated C shape and runtime contracts
- Kyokai standard library modules
- future `.kyo` and `.kai` source
- test programs and fixtures
- small scripts that support compiler/spec/test work
- public examples that claim to be valid Kyokai

`PROJECT_STANDARDS.md` governs workflow, branches, public D-points, PR process, releases, and review flow. This file governs how the code itself is written.

### 2.2 When Rules Conflict

Use this order:

1. Accepted Kyokai language/spec behavior.
2. Safety and correctness.
3. This file.
4. Local style in the touched module.
5. Convenience.

If a local file follows inherited Austral style and this file says something stricter, do not churn the whole file. Apply the stricter rule to the changed code and leave broad migration for a separate cleanup.

### 2.3 Code That Is Not Yet Kyokai-Native

The repository still contains inherited Austral code and docs. That is normal during the fork.

Rules:

- Do not rewrite inherited code just to make it look new.
- Do not treat inherited names as permanent Kyokai names when the accepted shape says otherwise.
- Do not mix mechanical rename work with semantic compiler changes.
- When modifying inherited code, preserve its working behavior unless the PR is intentionally changing that behavior.
- When the inherited behavior conflicts with Kyokai, add tests that prove the new Kyokai behavior.

### 2.4 Public Shape And Implementation

A code change may do one of four things:

| Change kind | Requirement |
| --- | --- |
| Implements accepted shape | Link the accepted shape/spec and add tests. |
| Repairs implementation to match accepted shape | Explain the mismatch and add regression tests. |
| Prototypes undecided behavior | Keep it non-default and clearly experimental. |
| Reveals a missing decision | Open/update the D-point before treating it as a contract. |

Do not smuggle language design through a code PR.

---

## 3. Reference Material

### 3.1 Local Project References

Use local sources first for project reality.

| Path | Use |
| --- | --- |
| `PROJECT_STANDARDS.md` | Workflow, PR, D-point, branch, release, and review process. |
| `Kyokaishape.md` | Public live D-points and proposal shape. |
| `kyokaidecided.md` | Public accepted shape while the full spec is being written. |
| `kyokaispec/` | Normative spec as it is extracted and written. |
| `phase.md` | Implementation order, proof gates, and done-when checks. |
| `lib/` | Current compiler source. |
| `docs/pipeline.dot` | Current compiler pipeline map. |
| `docs/walkthrough.md` | Current inherited compiler walkthrough. |
| `standard/` | Current inherited standard library surface. |
| `test/` and `test-programs/` | Existing unit and end-to-end test shape. |

### 3.2 OCaml References

These rules are informed by current OCaml practice:

- OCaml Programming Guidelines: https://ocaml.org/docs/guidelines
- OCaml Error Handling guide: https://ocaml.org/docs/error-handling
- OCaml Modules guide: https://ocaml.org/docs/modules
- OCaml manual, module system: https://ocaml.org/manual/5.4/modules.html
- Dune documentation: https://dune.readthedocs.io/en/stable/

Important takeaways for Kyokai work:

- Readability matters more than typing speed.
- Small functions are easier to master.
- Interfaces should document exported things.
- Avoid broad `open` directives when they make names ambiguous.
- Pattern matches should be exhaustive and specific.
- Do not hide expected errors in raw values.
- Use `option` and `result` for expected failures where the caller should handle the case.
- Document exceptions when exceptions are part of an exposed OCaml API.
- Use `Fun.protect` or equivalent patterns when host resources must be cleaned up after exceptions.

### 3.3 Austral References

The current compiler is inherited from Austral. Trust compiler source and tests over old prose when they disagree.

Important inherited pass names and concepts:

- parsing into separate interface/body CSTs
- insertion of pervasive imports
- combining interface/body declarations
- extraction into the environment
- typing into TAST
- return checking
- linearity checking
- body extraction for monomorphization
- monomorphization
- C AST generation
- C rendering
- wrapper generation

Kyokai may change these stages, but changes should be deliberate and tested.

### 3.4 Prior-Art Use

Prior art is useful, but not binding.

Use prior art like this:

- Austral: inherited implementation and design baseline.
- OCaml: implementation-language style and module discipline.
- Rust: diagnostics, ownership naming, safety culture, testing expectations.
- Zig: explicit build/runtime/system style, allocation visibility.
- C: ABI and backend hazard map, not a semantic model to copy.

Do not justify a Kyokai rule by saying another language does it. Explain why it fits Kyokai's explicitness, linearity, capability, and TPOE model.

---

## 4. Kyokai-Specific Engineering Rules

### 4.1 No Language-Level Undefined Behavior

Kyokai must not expose C-style undefined behavior as a language outcome.

Implementation rules:

- Reject source programs that cannot be given defined Kyokai behavior.
- Insert checked runtime operations where the language promises TPOE.
- Use target-specific lowering only when the target contract is explicit.
- Do not rely on signed overflow, invalid shifts, out-of-bounds access, invalid aliasing, uninitialized reads, or lifetime violations in generated C.
- If C cannot express a Kyokai operation safely, call a runtime helper that implements the check.

### 4.2 TPOE Is A Contract, Not A Crash Excuse

Terminate Program On Error means a defined failure mode for contract violations. It does not mean arbitrary compiler crashes, host exceptions, segfaults, or unstructured aborts.

Rules:

- TPOE sites must be named in the language/runtime/std API contract.
- Generated checks should map to a consistent runtime termination path.
- Diagnostics should distinguish compile-time rejection from runtime TPOE.
- Runtime TPOE should include enough context for debugging where feasible.
- Do not replace recoverable value errors with TPOE because it is easier to implement.

### 4.3 Linearity Is Not Optional

Linearity is the center of Kyokai's resource model.

Rules:

- Every compiler transform must preserve linear obligations.
- Desugaring must not duplicate, discard, or reorder linear values unless the language rule explicitly allows it.
- Lowered representations must still make ownership flow auditable.
- Runtime helpers must not consume or retain linear values unless their contract says so.
- Standard library APIs must state whether they consume, borrow, return, split, join, or transfer ownership.

### 4.4 Capabilities Are Authority

A capability value is not just a parameter. It is authority.

Rules:

- APIs that perform I/O, environment access, process control, clocks, randomness, networking, filesystem mutation, or target introspection must require the relevant capability.
- Do not read ambient process state deep inside ordinary library code.
- Boundary code may read ambient state, but it should translate it into typed capability-backed values.
- Capability errors should say which authority was missing.
- Tests should verify capability denial paths, not only successful authorized paths.

### 4.5 Visible Allocation

Allocation is part of the contract.

Rules:

- Public APIs that allocate must say where allocation comes from.
- Do not introduce hidden default allocator behavior in stdlib code.
- Compiler code may allocate normally, but hot paths should avoid accidental quadratic allocation.
- Runtime allocation failure must follow the decided Kyokai failure model.
- Conversion names should distinguish borrowed views, owned outputs, and allocator-taking operations.

### 4.6 Visible Blocking

Blocking is part of the contract.

Rules:

- Blocking APIs should state blocking behavior in the name, type, or contract.
- Poller/event-loop APIs should expose wait boundaries explicitly.
- Plain blocking calls may wait forever if the OS/resource never becomes ready; that must be documented where such calls exist.
- Cancellation, timeout, detach, and join behavior must be explicit.
- No API should hide a background task that outlives its owner without a contract.

### 4.7 No Compiler Folklore

If the compiler inserts, lowers, defaults, completes, or rejects something, the rule must be findable.

Rules:

- Sugar lowering must be represented by a pass with tests.
- Implicit completions must be closed, explicit, and test-backed.
- Diagnostics should describe the rule, not merely the parser symptom.
- Do not use "obvious" as a reason to skip writing down a behavior.

---

## 5. Repository Layout And Ownership

### 5.1 Current Layout

Expected current layout:

```text
kyokailang/kyokai/
  bin/                 compiler entrypoints
  lib/                 OCaml compiler implementation
  standard/            inherited/current standard library sources
  test/                OCaml/unit-level tests
  test-programs/       end-to-end language tests
  examples/            public examples
  docs/                compiler and project docs
  kyokaispec/          normative spec work
  Kyokaishape.md       public D-point tracker
  kyokaidecided.md     public accepted-shape extraction
  phase.md             roadmap and implementation gates
  PROJECT_STANDARDS.md workflow standards
  CODE_STANDARDS.md    this code handbook
```

Do not create a parallel compiler, runtime, or stdlib tree without a migration plan.

### 5.2 File Ownership

A file should have a clear reason to exist.

Good file purposes:

- one compiler pass
- one AST or IR definition family
- one diagnostic domain
- one environment/index abstraction
- one backend representation
- one runtime boundary
- one stdlib module
- one test harness or fixture family

Bad file purposes:

- miscellaneous helpers
- everything related to type checking and lowering
- all runtime code
- all target-specific hacks
- temporary dumps that become permanent

### 5.3 Module Size

There is no hard line-count law, but large modules must earn their size.

Rules:

- A long type definition module is acceptable if it defines one coherent IR.
- A long pattern-match module is acceptable if it is one compiler pass and branches are simple.
- A long algorithm module needs section comments and tests for the major cases.
- A module with unrelated responsibilities should be split.
- New code should not make already-large inherited modules worse without a reason.

Signals that a module should split:

- it imports nearly every compiler stage
- tests cannot target one behavior without constructing half the compiler
- helpers have names like `misc`, `util`, `stuff`, or `common2`
- small edits routinely cause unrelated behavior changes
- the module has several independent mutable states

### 5.4 Interface Files

OCaml `.mli` files are important boundaries.

Rules:

- Public compiler modules should have `.mli` files unless there is a concrete reason not to.
- Interfaces should expose types abstractly when callers should not construct invalid values.
- Interfaces should document exported functions when misuse would break an invariant.
- Avoid exposing helper functions only because tests want them; prefer testing through the public pass boundary or use a test-only module when needed.
- Keep implementation-only constructors hidden when they encode invariants.

### 5.5 Scripts And One-Off Tools

Scripts tend to become part of the toolchain.

Rules:

- Prefer Dune targets or Makefile entries for repeatable tasks.
- If a script is committed, document how to run it.
- Avoid shell scripts that depend on the caller's current directory unless that is checked.
- Use strict shell flags for non-trivial shell scripts.
- Do not leave generated files stale after changing a generator.

---

## 6. OCaml Compiler Code

### 6.1 General OCaml Style

Use normal OCaml style.

Rules:

- Function and value names use `snake_case`.
- Module names use `CamelCase`.
- Variant constructors use `CamelCase`.
- Record fields use `snake_case`.
- Type names use `snake_case`.
- Prefer explicit type annotations at public boundaries and complicated helpers.
- Prefer local inference for simple expressions where the type is obvious.
- Avoid point-free or highly combinator-heavy code when it hides compiler state.

### 6.2 Formatting

Rules:

- Follow the formatting style of nearby files until a formatter policy is established.
- Do not fight the formatter if OCamlFormat is adopted for a subtree.
- Keep line breaks around pattern matches and records readable.
- Avoid horizontal compression of complex AST constructors.
- Use vertical formatting for values with many fields.

Good:

```ocaml
let expr =
  TCall {
    callee;
    type_args;
    args;
    span;
  }
```

Bad:

```ocaml
let expr = TCall { callee; type_args; args; span }
```

The second is fine for tiny records. It is bad when the fields carry different invariants.

### 6.3 Naming

Names should reveal compiler stage and invariant.

Preferred patterns:

- `surface_expr`, `surface_stmt`
- `typed_expr`, `typed_stmt`
- `lowered_expr`, `lowered_stmt`
- `mono_module`, `mono_decl`
- `borrow_state`, `move_state`
- `capability_req`, `effect_req`
- `source_span`, `file_id`
- `module_name`, `qualified_name`

Avoid:

- `thing`
- `stuff`
- `data`
- `obj`
- `node2`
- `helper` as an exported function name
- `tmp` outside tiny local scopes
- reusing the same variable name for source and lowered values in the same function

### 6.4 Complex Arguments

For complex function arguments, bind them first.

Good:

```ocaml
let actual_ty = infer_expr env expr in
let expected_ty = instantiate_scheme env scheme in
check_assignable ~span env actual_ty expected_ty
```

Bad:

```ocaml
check_assignable ~span env (infer_expr env expr) (instantiate_scheme env scheme)
```

The bad version is shorter but makes debugging and instrumentation harder.

### 6.5 Anonymous Functions

Use anonymous functions for small local transformations. Name them when the body is complex or reused.

Good:

```ocaml
let lower_arg arg =
  lower_expr env arg
in
List.map lower_arg args
```

Bad:

```ocaml
List.map (fun arg ->
  let typed = elaborate_arg env arg in
  let checked = check_arg env typed in
  lower_arg env checked) args
```

The named helper gives the transformation a place to grow and a name for traces.

### 6.6 Pattern Matching

Pattern matching is one of OCaml's strengths. Use it directly.

Rules:

- Prefer explicit constructor lists over catch-all cases for compiler ASTs.
- Avoid `_` when matching a closed language datatype unless all remaining cases truly have the same behavior.
- Let new constructors produce warnings in places that must be updated.
- Use guards sparingly; a guard can hide exhaustiveness issues.
- Keep side effects out of match scrutinees when possible.

Good:

```ocaml
match universe with
| FreeUniverse -> allow_copy ()
| LinearUniverse -> require_single_use span
| TypeUniverse -> internal_err "type universe cannot be used as a value"
```

Bad:

```ocaml
match universe with
| LinearUniverse -> require_single_use span
| _ -> allow_copy ()
```

The bad version silently accepts future universes.

### 6.7 Exceptions In OCaml Compiler Code

OCaml exceptions are acceptable, but use them deliberately.

Use exceptions for:

- internal compiler errors
- top-level abort paths already used by the inherited compiler
- short-circuiting internal algorithms where the exception cannot escape the pass
- host failures that are naturally exception-based and are converted at the boundary

Prefer `result` or structured diagnostic values for:

- expected source-program rejection
- parser/type-checker/linearity diagnostics
- recoverable toolchain failures
- APIs used by tests that need to inspect failure values

Rules:

- Do not catch all exceptions unless you re-raise unknown ones.
- Do not turn source-program errors into host crashes.
- Document exposed functions that can raise.
- Use `Fun.protect` or equivalent cleanup patterns for host resources.
- Keep exception-based control flow local and obvious.

### 6.8 Mutation

Mutation is allowed when it makes a pass simpler or faster, but it must have an owner.

Good mutation:

- a local worklist in a graph traversal
- a memo table with clear lifetime
- an environment builder hidden behind an abstract interface
- a generated-name counter local to a lowering pass

Bad mutation:

- global pass state
- mutable fields shared across unrelated passes
- caches with unclear invalidation
- mutation hidden behind a function that looks pure
- tests depending on execution order because state leaked from a previous test

### 6.9 Recursion And Loops

OCaml code often uses recursion instead of loops.

Rules:

- Use recursive functions for tree traversals and state-threading algorithms.
- Use `List.fold_left` or `List.fold_right` when the accumulator is simple and named.
- Avoid nested folds when a direct recursive function would be clearer.
- Use arrays and loops for performance-sensitive indexed data, but state the invariant.
- Make recursive functions tail-recursive when they can process large inputs.

### 6.10 Standard Library Use

Rules:

- Prefer explicit pattern matching over partial functions like `List.hd` and `List.tl`.
- Use `_opt` functions when missing values are expected.
- Avoid `Option.get` in production compiler code unless the invariant is immediately proven and commented.
- Avoid stringly maps when typed identifiers are available.
- Prefer `Format` for structured pretty-printing and diagnostics once a formatter path exists.

---

## 7. Compiler Pass Architecture

### 7.1 Current Pipeline

The inherited compiler pipeline is roughly:

1. parse interface/body files into CSTs
2. insert pervasive imports
3. combine interface/body into a combined module
4. resolve imports and extract declarations into the environment
5. type check into a typed AST
6. check returns
7. check linearity
8. extract bodies for monomorphization
9. monomorphize generic declarations
10. generate C AST
11. render C code
12. generate wrappers and entrypoints

Kyokai may add or split passes, especially for syntax changes, typed sugar, implicit completions, capabilities, target checks, stdlib contracts, and richer diagnostics.

### 7.2 Pass Contract Template

Every non-trivial pass should be explainable with this template:

```text
Pass name:
Input representation:
Input invariants:
Output representation:
Output invariants:
Errors produced:
Source locations preserved:
Environment reads:
Environment writes:
Language rules enforced:
Tests:
```

This template does not need to be pasted into every file. The pass design should be clear enough that a reviewer can answer it.

### 7.3 Pass Boundaries

A pass should have one job.

Good pass jobs:

- parse tokens into CST
- combine interface/body declarations
- resolve imports
- desugar syntax-only forms
- elaborate names and types
- type check expressions
- register implicit completions
- lower typed sugar
- check returns
- check linearity
- check capabilities and unsafe boundaries
- check target availability
- monomorphize
- generate C AST
- render C

Bad pass jobs:

- type check and lower because both touch expressions
- desugar and check linearity because both walk statements
- render C while deciding target semantics
- insert runtime checks in the renderer without IR representation
- accept syntax and decide semantics in parser recovery

### 7.4 Desugaring

Desugaring must not hide semantic work.

Rules:

- Syntax-only sugar can lower before type checking if it does not need type information.
- Typed sugar must lower after type checking or after the relevant elaboration pass.
- Desugaring that creates moves, borrows, defers, drops, or TPOE checks must be represented in a form later passes can inspect.
- Tests should cover both the surface form and the lowered behavior.
- Diagnostics should point to user source, not only generated nodes.

### 7.5 Environment Discipline

The compiler environment is a central authority. Treat it carefully.

Rules:

- Do not add unrelated fields to the environment because it is globally available.
- Separate global declarations from pass-local state.
- Environment writes should happen in passes that own the relevant facts.
- If a pass reads an environment field, it should be clear why that field is already populated.
- Avoid cyclic pass dependencies through environment side channels.

### 7.6 Source Locations

Source locations are not optional.

Rules:

- AST nodes that can produce diagnostics should carry spans or be traceable to spans.
- Generated nodes should retain origin spans when possible.
- Multi-token constructs should preserve the most useful span for diagnostics.
- If a generated node has no direct span, store an origin reason or parent span.
- Backend/runtime diagnostics should map back to Kyokai source when feasible.

### 7.7 Lowering To C

C is a target representation, not Kyokai's semantic model.

Rules:

- Decide Kyokai semantics before rendering C.
- Do not let C precedence, overflow, aliasing, or evaluation order define Kyokai behavior.
- Generate C that is boring and deterministic.
- Use helper functions when an operation needs checks or target-specific behavior.
- Keep generated names stable enough for tests and debugging.

---

## 8. Data Modeling And Invariants

### 8.1 Variants

Use variants for closed choices.

Good variant domains:

- token kinds
- expression forms
- statement forms
- universes
- borrow modes
- move states
- visibility
- target tiers
- ABI classifications
- diagnostic severity
- typeclass relation kinds

Rules:

- Avoid string tags for closed domains.
- Do not add an `Unknown` constructor to avoid handling cases unless the source language actually has an unknown state.
- Keep variant constructors precise.
- Prefer nested variants over one giant variant when domains are independent.

### 8.2 Records

Use records when fields have names that matter.

Rules:

- Use records for AST nodes with multiple fields.
- Prefer field names that state source meaning, not storage detail.
- Avoid large records with optional fields for different node kinds; use variants.
- Make invalid combinations impossible when practical.
- Use record update syntax only when unchanged fields are genuinely unchanged semantically.

### 8.3 Phantom Types And Abstract Types

OCaml phantom types and abstract modules can encode pass state.

Useful examples:

```ocaml
type surface
type typed
type lowered

type 'stage expr
```

This is useful when the same shape appears across stages but only some operations are valid in each stage.

Rules:

- Use phantom stages only when they prevent real mistakes.
- Do not add type-level ceremony for tiny local helpers.
- Hide constructors if callers should not manufacture values.
- Keep conversion functions named after the pass boundary.

### 8.4 Strings

Strings are acceptable at boundaries. They are suspicious in core compiler data.

Rules:

- Source text is a string.
- File contents are strings.
- Raw identifiers may be strings in the lexer/parser.
- After parsing, prefer `identifier`, `module_name`, `qualified_name`, or similar types.
- Do not represent types, universes, capabilities, or visibility as strings.
- Do not construct diagnostics by ad hoc string concatenation when structured diagnostic pieces exist.

### 8.5 Booleans

A boolean is fine for a local predicate. It is weak for domain state.

Bad:

```ocaml
type borrow = {
  mutable_: bool;
  owned: bool;
}
```

Good:

```ocaml
type borrow_mode =
  | SharedBorrow
  | MutableBorrow
```

Rules:

- Avoid boolean parameters in public functions.
- Prefer variants for modes.
- If a boolean field remains, name it as a predicate: `is_public`, `requires_allocator`, `may_block`.

### 8.6 Partial Values

Avoid building partially valid compiler values.

Rules:

- Do not use placeholder spans unless the node is truly synthetic and has an origin note.
- Do not create dummy types that later passes must remember to replace.
- Do not store unresolved names in typed AST unless the type says they are unresolved.
- Do not use `None` to mean several different missing states.

---

## 9. Error Handling And Diagnostics In The Compiler

### 9.1 Error Categories

Separate these categories:

| Category | Meaning | Handling |
| --- | --- | --- |
| Source diagnostic | User program violates Kyokai rules. | Structured diagnostic. |
| Toolchain diagnostic | Manifest/CLI/target/build input is invalid. | Structured diagnostic. |
| Host failure | Filesystem/process/env failure. | Convert at boundary with operation/path. |
| Internal compiler error | Compiler invariant broken. | Loud internal error with context. |
| Runtime TPOE | Compiled program violates runtime contract. | Generated/runtime termination path. |

Do not blur these categories.

### 9.2 Source Diagnostics

Rules:

- Include a primary message.
- Include a source span when available.
- Include notes when a rule needs context.
- Include related spans when the problem involves two declarations or uses.
- Avoid generic messages like `type error` when the compiler knows the mismatch.
- Keep wording stable when tests cover it.

Examples of useful diagnostic subjects:

- duplicate module declaration
- imported name not found
- visibility violation
- type mismatch
- universe mismatch
- linear value not consumed
- linear value consumed twice
- borrow after move
- mutable borrow conflict
- missing capability
- unsafe operation outside unsafe boundary
- target feature unavailable
- comptime evaluation failure

### 9.3 Internal Errors

Internal errors should help fix the compiler.

Rules:

- State the broken invariant.
- Include the pass name when feasible.
- Include the relevant identifier/type/node kind.
- Do not show a vague `assert false` without context in new code.
- If an inherited `assert false` is touched, improve it when practical.

Good:

```ocaml
internal_err
  ("LinearityCheck: variable `" ^ ident_string name ^
   "` missing from state table after name resolution")
```

Bad:

```ocaml
assert false
```

### 9.4 Result And Option

Use `option` for absence. Use `result` for expected failure with information.

Rules:

- `find_*` operations may return `option` when missing is normal.
- Operations that explain failure should return `result` or a diagnostic type.
- Avoid returning `None` for several distinct failures.
- Avoid `Option.get` except directly after a proof.
- Name exception-raising variants with `_exn` if both forms exist in a module.

### 9.5 Exceptions

Rules:

- Do not catch all exceptions in library code unless cleaning up and re-raising.
- Do not convert every exception to a generic diagnostic.
- Do not use exceptions for ordinary source rejection in new APIs unless the surrounding inherited compiler path requires it.
- If an API can raise and is exported in `.mli`, document it.
- Use cleanup protection for open files and other host resources.

### 9.6 Diagnostics As Tests

Diagnostics need tests when:

- wording teaches a language rule
- spans are important
- a previous bug produced a confusing error
- the diagnostic is part of public tooling output
- JSON/structured output changes

Negative tests should assert the relevant code/message/span, not just non-zero exit.

---

## 10. Linearity, Borrowing, And Capability Checks

### 10.1 Linearity State

Linearity tracking should stay explicit.

Rules:

- Model states like unconsumed, borrowed, moved, consumed, pending, and deferred cleanup explicitly.
- Keep loop-depth and branch-merge rules visible.
- Do not hide linearity updates inside generic expression traversal helpers without naming them.
- Tests must cover branch consistency and loop restrictions.
- When adding a new expression or statement form, update the linearity checker deliberately.

### 10.2 Borrow Checking

Rules:

- Shared and mutable borrows are distinct states.
- Borrow scopes must be represented explicitly enough for diagnostics.
- Auto-reborrow or implicit borrow completions must be closed and test-backed.
- Lowering must not extend or shorten borrow lifetimes accidentally.
- Diagnostics should say whether the problem is moved value, outstanding borrow, duplicate mutable borrow, or use after consumption.

### 10.3 Defer And Cleanup

Rules:

- `defer` and `errdefer` behavior must be represented so linearity can see cleanup obligations.
- Cleanup order must be deterministic.
- Cleanup on early return, error path, and normal scope exit must be tested separately.
- Deferred operations must not hide ownership transfer.
- If cleanup can fail, the failure rule must be explicit.

### 10.4 Capabilities

Rules:

- Capability requirements should be checked before lowering to backend operations.
- Capability errors should name the missing authority.
- Capability values should not be forged by runtime helpers.
- Code generation should not bypass capability checks by calling raw runtime operations from ordinary code.
- Tests should prove that capability-less programs are rejected.

### 10.5 Unsafe Boundaries

Rules:

- Unsafe modules/functions/operations need a narrow marker.
- The unsafe marker should not bless unrelated code in the same large module if a smaller boundary is possible.
- Unsafe operations should have documented preconditions.
- Safe wrappers must discharge those preconditions before exposing safe APIs.
- Tests should cover wrapper behavior, not just raw unsafe calls.

---

## 11. Backend, Runtime, FFI, And Generated C

### 11.1 Backend Contract

The backend implements Kyokai semantics. It does not invent them.

Rules:

- Lowering to C must preserve Kyokai evaluation order.
- Runtime checks must be inserted before operations that require TPOE.
- Generated temporaries must not duplicate linear values.
- Generated C must avoid relying on unspecified or undefined C behavior.
- Target-specific code paths need target tests or documented target gating.

### 11.2 Generated C Style

Generated C should be boring.

Rules:

- Prefer explicit temporaries over clever expressions with unclear order.
- Parenthesize generated expressions conservatively.
- Use fixed-width integer types when Kyokai width matters.
- Avoid macros for semantic operations unless macros are the chosen runtime interface and are tested.
- Keep generated names deterministic.
- Include enough comments only when generated code is meant for debugging.

### 11.3 C Hazards

Never rely on these for Kyokai behavior:

- signed integer overflow
- shift by negative amount or amount greater/equal to width
- out-of-bounds pointer arithmetic
- type punning that violates aliasing rules
- uninitialized reads
- use after free
- double free
- invalid alignment
- order of evaluation that C does not specify
- null dereference

If Kyokai permits the source operation, generate checked code or call a runtime helper.

### 11.4 Runtime Helpers

Rules:

- Runtime helpers should have small contracts.
- Each helper should state ownership of input and output values.
- Helpers that may allocate must state allocator behavior.
- Helpers that may block must state blocking behavior.
- Helpers that may terminate must state the TPOE condition.
- Helpers that wrap OS calls must translate OS errors into Kyokai's chosen error shape.

### 11.5 FFI Bindings

Every FFI binding needs this contract:

```text
Foreign function:
Kyokai wrapper:
Header/library:
Target availability:
Input ownership:
Output ownership:
Borrow/lifetime requirements:
Nullability:
Aliasing requirements:
Error return:
Errno or platform error use:
Blocking behavior:
Thread/task restrictions:
Cleanup requirement:
Safety argument:
Tests or manual verification:
```

This template can be condensed in code comments, but the information must exist.

### 11.6 ABI And Layout

Rules:

- Do not promise layout compatibility unless the layout is defined.
- Types crossing FFI must have explicit representation rules.
- Endianness, alignment, integer width, and calling convention assumptions must be target-gated or specified.
- Do not expose raw C layout through safe Kyokai APIs unless that is the API's purpose.
- If layout is opaque, keep it opaque.

---

## 12. Kyokai Standard Library Code

### 12.1 Standard Library Role

The stdlib should make correct Kyokai code natural without hiding resource behavior.

Rules:

- Prefer safe Kyokai implementations for pure computation.
- Use runtime/FFI only for true OS, hardware, ABI, allocator, and platform boundaries.
- Keep authority explicit through capabilities.
- Keep allocation explicit through allocator parameters or contracts.
- Keep blocking explicit through names or contracts.
- Keep failure typed unless the accepted rule is TPOE.

### 12.2 API Contract Fields

Public stdlib APIs should document these fields when relevant:

| Field | Meaning |
| --- | --- |
| Ownership | Consumes, borrows, returns, transfers, splits, joins, or destroys values. |
| Linearity | Whether linear obligations are created or discharged. |
| Allocation | Whether allocation happens and which allocator is used. |
| Failure | `Result`, `Optional`, TPOE, OS error, target rejection, or impossible. |
| Blocking | Whether the call can wait and under what condition. |
| Capability | Required authority. |
| Target | Supported OS/ABI/architecture constraints. |
| Determinism | Whether result/order is deterministic. |
| Complexity | Big-O when relevant. |
| Runtime/FFI | Boundary dependency, if any. |

### 12.3 Naming For Ownership And Allocation

Follow accepted Kyokai naming rules.

Typical meanings:

| Pattern | Meaning |
| --- | --- |
| `as*` | Borrowed view, no ownership transfer, no allocation. |
| `to*In` | Borrow input and allocate an owned result in an explicit allocator/destination. |
| `into*` | Consume input and transform/reuse ownership. |
| `into*In` | Consume input and allocate an owned result in an explicit allocator/destination. |
| `make*` | Construct a new value/resource. |
| `from*` | Construct from another representation; allocator visible if allocation occurs. |
| `destroy*` | Consume and release a resource. |
| `try*` | Operation may fail in a typed recoverable way. |
| `*Blocking` | Operation may block. |

Do not use names that hide allocation, authority, or ownership transfer.

### 12.4 Collections

Rules:

- Define iterator invalidation and mutation behavior per collection.
- State allocation and reallocation behavior.
- State ownership of inserted and removed linear values.
- Do not hide cloning/copying of values.
- Hashing, ordering, and equality requirements must be explicit typeclass constraints.
- Edge cases such as empty collections, capacity growth, and removal during iteration need tests.

### 12.5 Text And Bytes

Rules:

- Keep bytes and Unicode/code-point concepts distinct.
- Do not treat byte length and character count as the same thing.
- APIs should say whether they operate on bytes, scalar values, grapheme clusters, or validated UTF-8 strings.
- FFI string boundaries must state null-termination and encoding rules.
- Text APIs that allocate must take or identify the allocator.

### 12.6 Numerics

Rules:

- Integer overflow, division by zero, invalid shifts, and conversion overflow must follow accepted Kyokai semantics.
- Floating-point APIs must state NaN, infinity, signed zero, rounding, and platform behavior where relevant.
- Prefer native Kyokai implementations for pure math once the language can support them.
- FFI math wrappers must document libm/platform differences.
- Tests should cover boundaries, not only normal cases.

### 12.7 I/O And OS APIs

Rules:

- I/O APIs require capabilities.
- Blocking behavior must be explicit.
- Partial read/write behavior must be explicit.
- File descriptor/resource ownership must be linear.
- Error values should preserve relevant OS error detail without exposing raw platform quirks as the main API.
- Cleanup must be explicit and testable.

---

## 13. Future Kyokai Source Code

### 13.1 General Kyokai Code Style

Future Kyokai code should be explicit, direct, and readable.

Rules:

- Use accepted syntax only in default tests and examples.
- Keep modules focused.
- Keep interfaces honest about public API and opaque types.
- Use linear values exactly once.
- Use borrows for temporary access.
- Use capabilities for authority.
- Use `Result`/`Optional` for expected failure where accepted.
- Use TPOE only for contract violations defined as TPOE.

### 13.2 Resource Flow

Rules:

- Every resource has one obvious owner.
- Functions that consume a value should make that clear by type and name.
- Failed operations that do not take ownership must return the original linear value or otherwise preserve ownership according to the contract.
- Do not bury resource transfer inside a helper with an innocent name.
- Cleanup should be visible through explicit consumption, `defer`, `errdefer`, or a named destructor path.

### 13.3 Module Interfaces

Rules:

- Interfaces should expose only the intended public surface.
- Opaque types should hide representation but not hide ownership contract.
- Public modules should document capability requirements and allocation behavior.
- Do not export raw runtime details unless the module is specifically a boundary module.

### 13.4 Examples

Examples are part of the language's public face.

Rules:

- Examples must use accepted syntax unless marked experimental.
- Examples should compile once the relevant feature exists.
- Examples should not rely on hidden capabilities, hidden allocators, or hidden cleanup.
- Examples should show error handling when the operation can fail.
- Keep examples small but complete.

---

## 14. Tests And Conformance

### 14.1 Test Layers

Use multiple layers.

| Layer | Purpose |
| --- | --- |
| OCaml unit tests | Test compiler helpers, passes, environments, diagnostics. |
| Parser tests | Accept/reject syntax and spans. |
| Type-checker tests | Accepted/rejected typing rules. |
| Linearity tests | Moves, borrows, branch/loop rules, cleanup. |
| Backend tests | Generated C and runtime behavior. |
| Stdlib tests | API contracts, edge cases, failures. |
| Toolchain tests | CLI, packages, manifests, lockfiles, targets. |
| Conformance tests | Public language behavior across implementations. |

### 14.2 Positive Tests

Positive tests prove programs are accepted and behave as expected.

Rules:

- Keep positive tests focused.
- Avoid giant examples that test too many features at once.
- Include runtime expected output when behavior matters.
- Include compile-only tests for type-system acceptance.
- If a test needs several features, state which feature it primarily protects.

### 14.3 Negative Tests

Negative tests are essential for Kyokai.

Rules:

- Assert the diagnostic category/code/message, not just failure.
- Include source spans where the harness supports them.
- Test one rejection reason per file when possible.
- Avoid tests that fail earlier than the intended rule.
- Keep expected-error comments close to the offending line if the harness supports it.

### 14.4 Regression Tests

Every fixed bug should have a regression test unless impossible.

A good regression test:

- is small
- fails before the fix
- passes after the fix
- names the rule or bug
- avoids depending on unrelated behavior

### 14.5 Golden Files

Rules:

- Golden files must be deterministic.
- Update goldens intentionally, not as a reflex.
- Review golden diffs like code.
- Do not hide broad output churn inside a semantic change.
- Prefer stable diagnostic codes to brittle full-message matching when appropriate.

### 14.6 Test Data

Rules:

- Keep fixtures small.
- Put large fixtures in a named directory with a short explanation.
- Do not use random data without a fixed seed and failure minimization path.
- Do not rely on host-specific paths unless the test is target-specific.
- Mark target-specific tests clearly.

---

## 15. Documentation In And Around Code

### 15.1 Comments

Comments should explain difficulty, not narrate syntax.

Good comments explain:

- a non-obvious invariant
- why a pass order matters
- why a target-specific workaround exists
- why an apparently redundant check is required
- the safety argument for an unsafe boundary
- the ownership contract for a runtime helper

Bad comments say:

- `increment i`
- `match on expr`
- `create list`
- `call helper`

### 15.2 Interface Documentation

`.mli` comments should document exported contracts.

Include:

- what the function does
- input invariants
- output guarantees
- diagnostics or exceptions
- mutation or environment effects
- source-location behavior if relevant

### 15.3 TODOs

Rules:

- Do not write vague TODOs.
- A TODO should name the missing rule, issue, D-point, or condition for removal.
- Do not leave TODOs in accepted semantic paths without a tracking note.
- Remove obsolete TODOs when touching nearby code.

Good:

```ocaml
(* TODO(D314): replace this target check with the normalized Target.t once the
   target parser lands. *)
```

Bad:

```ocaml
(* TODO fix later *)
```

### 15.4 Public Docs Near Code

Rules:

- Public docs should use maintainer voice.
- Do not write chat-style provenance in public docs.
- Docs that describe behavior should point to accepted shape or spec where practical.
- Inherited Austral wording must be updated before it is presented as Kyokai behavior.
- Keep examples aligned with feature status.

---

## 16. Dependencies, Tooling, And Build Hygiene

### 16.1 Dependencies

Rules:

- Prefer existing dependencies and the OCaml standard library when enough.
- Add dependencies only when they clearly reduce complexity or improve correctness.
- Check license compatibility before adding a dependency.
- Check maintenance status before adding a dependency.
- Avoid dependencies for tiny helpers.
- Avoid dependencies that force broad toolchain changes without a plan.

### 16.2 Dune

Rules:

- Keep Dune files simple and explicit.
- Do not hide important build behavior in generated Dune fragments without documenting the generator.
- Add test aliases for new test families.
- Keep library/executable boundaries clear.
- Prefer reproducible Dune targets over manual command sequences.

### 16.3 Generated Files

Rules:

- Generated files should say how they are generated when not obvious.
- Generators should be deterministic.
- Generated files should not be hand-edited unless the file says that is allowed.
- Changes to a generator should include regenerated outputs or explain why not.

### 16.4 Shell And Support Scripts

Rules:

- Use shell for simple orchestration.
- Use OCaml or another real language for complex parsing or transformations.
- Quote paths.
- Check command failures.
- Avoid relying on caller-specific environment.
- Document required tools.

### 16.5 Tool Versions

Rules:

- Pin versions when reproducibility depends on them.
- Keep opam, Dune, Nix, and CI expectations aligned.
- Do not silently require a newer compiler/tool version without updating docs.
- If a feature needs OCaml 5 behavior, say so.

---

## 17. Performance, Allocation, And Memory Discipline

### 17.1 Compiler Performance

Correctness comes first, but compiler performance matters.

Rules:

- Avoid obvious quadratic behavior in name lookup, import resolution, type checking, and monomorphization.
- Use maps/sets for repeated lookup.
- Cache only when invalidation is clear.
- Keep source spans and diagnostics without duplicating huge source strings in every node.
- Measure before large rewrites.

### 17.2 Allocation In Compiler Code

OCaml allocation is normal, but avoid accidental churn in hot paths.

Rules:

- Avoid repeated string concatenation in loops.
- Use buffers or `Format` for large output.
- Avoid rebuilding whole environments for tiny updates unless the structure is persistent and intended.
- Avoid converting maps to lists repeatedly in inner loops.
- Keep large generated code assembly efficient.

### 17.3 Runtime Memory

Rules:

- Runtime memory ownership must match Kyokai's linear model.
- Allocation and deallocation functions must pair cleanly.
- Do not expose raw memory handles through safe APIs without ownership wrappers.
- Out-of-memory behavior must follow accepted policy.
- Tests should include allocation failure paths when the harness supports them.

### 17.4 Numerics And Low-Level Operations

Rules:

- Use checked helpers for operations that can violate Kyokai runtime contracts.
- Keep integer width explicit.
- Do not depend on host `int` size for Kyokai target integer semantics.
- Floating-point behavior should be documented when target/libm differences matter.
- Edge cases belong in tests.

---

## 18. Concurrency, Blocking, And Task Boundaries

### 18.1 Compiler Concurrency

The compiler does not need concurrency until there is a clear plan.

Rules:

- Do not add shared mutable global state that blocks future parallel compilation.
- Keep caches scoped by compiler instance or workspace when possible.
- Make environment dependencies explicit enough for future incremental/parallel work.
- If parallelism is added, define ordering and determinism rules.

### 18.2 Kyokai Runtime Concurrency

Rules:

- Task creation, join, cancel, detach, and failure propagation must follow accepted shape.
- No implicit join hidden at scope close unless the accepted rule says so.
- Channel endpoint ownership must be linear and visible.
- Transfer across task boundaries must be explicitly allowed by the relevant declaration/contract.
- Poller behavior must state readiness, timeout, cancellation, and ownership rules.

### 18.3 Blocking APIs

Rules:

- Blocking calls should be named or documented as blocking.
- Nonblocking calls should state what they return when work is not ready.
- Timeout APIs should define clock source and timeout semantics.
- Cancellation APIs should define ownership of partially completed operations.
- Tests should cover ready, not-ready, timeout, closed, and error cases where possible.

---

## 19. Reviews And Change Shape

### 19.1 Review Priorities

Review for correctness first.

Priority order:

1. Language/spec correctness.
2. Safety, UB, ownership, capability, and runtime contract.
3. Tests and diagnostics.
4. Maintainability and pass boundaries.
5. Performance and allocation.
6. Style consistency.

Style matters, but style should serve correctness and readability.

### 19.2 Change Size

Rules:

- Keep semantic changes focused.
- Do not mix broad formatting with behavior changes.
- Do not mix inherited rename churn with compiler logic changes.
- Split PRs when they touch unrelated passes.
- Include migration notes for large refactors.

### 19.3 PR Description Expectations

A code PR should state:

- what changed
- why it changed
- accepted shape/spec link if semantic
- affected passes/modules
- tests run
- diagnostics changed
- runtime/FFI/backend safety impact if relevant
- remaining gaps

### 19.4 Review Checklist

Before code is ready:

- [ ] The behavior is accepted shape, spec text, or clearly experimental.
- [ ] The change has a clear owner module/pass.
- [ ] Data types encode important invariants.
- [ ] Source locations are preserved where diagnostics need them.
- [ ] Expected failures are diagnostics/results, not host crashes.
- [ ] Internal errors state the broken invariant.
- [ ] Linearity and capability effects are explicit.
- [ ] Runtime/FFI/backend safety is documented.
- [ ] Tests cover success, failure, and diagnostics where relevant.
- [ ] Generated output is deterministic.
- [ ] Public docs/spec/shape are updated when behavior changed.

---

## 20. Security, Safety, And Platform Boundaries

### 20.1 Security Model

Kyokai's capability model should make authority visible.

Rules:

- Do not add ambient filesystem, network, process, environment, clock, random, or OS access in ordinary code.
- Capability-bearing APIs should be easy to audit.
- Denial paths should be tested.
- Boundary code should minimize the amount of trusted code.

### 20.2 Unsafe Code

Unsafe code is allowed only with a safety argument.

A safety argument should say:

- what the unsafe operation assumes
- who checks each assumption
- what happens if the assumption is violated
- why the wrapper is safe for callers
- how tests exercise the wrapper

### 20.3 Platform Support

Rules:

- Target support tiers must match public policy.
- Target-specific behavior should be isolated.
- Unsupported target/feature combinations should be rejected clearly.
- Do not silently degrade semantics on weaker targets.
- Platform probes should be deterministic and cacheable where practical.

### 20.4 Licensing And Runtime Linking

Rules:

- Compiler, runtime, and standard library licensing must be reflected in source headers and license files once finalized.
- Runtime-library exception notices belong on runtime/stdlib files that are linked into user programs.
- Generated user code should not accidentally inherit compiler-only licensing terms.
- New third-party code needs license review before vendoring.

---

## 21. Concrete Patterns

### 21.1 Good Pass Function Shape

```ocaml
val check_module_linearity : typed_module -> unit
```

A simple pass entrypoint is good when:

- input type states the stage
- output is clear
- errors go through the diagnostic path
- pass-local helpers stay hidden

For passes that transform:

```ocaml
val lower_module : env -> typed_module -> lowered_module
```

If the pass mutates the environment:

```ocaml
val extract : env -> combined_module -> env * linked_module
```

State environment mutation in the interface comment.

### 21.2 Good Diagnostic Construction

```ocaml
raise_linear_error span [
  Text "Linear value ";
  Code (ident_string name);
  Text " is consumed in one branch but not the other.";
]
```

Rules:

- include the value name
- include the operation
- include the branch/scope context
- point at the source span

### 21.3 Good FFI Wrapper Comment

```ocaml
(* Wrapper for POSIX read.
   Ownership: does not take ownership of the file descriptor capability.
   Buffer: caller provides a valid mutable byte span for the duration of call.
   Blocking: may block until data, EOF, signal, or OS error.
   Failure: returns an OS error value; does not TPOE.
   Safety: raw pointer and length are produced from a live Kyokai span. *)
```

### 21.4 Good Stdlib Contract Sketch

```text
readBlocking(file: &!File, buf: SpanMut[Nat8, R], cap: &ReadCap): Result[ReadCount, IOError]

Ownership: borrows file and buffer; consumes nothing.
Allocation: none.
Blocking: may wait until data, EOF, signal, or OS error.
Failure: returns IOError; ownership remains with caller.
Capability: requires ReadCap.
Target: POSIX-backed targets that support file descriptors.
```

### 21.5 Good Test Naming

Use names like:

- `linearity_rejects_move_in_one_branch.kyo`
- `borrow_rejects_mutable_alias.kyo`
- `parser_accepts_raw_string_literal.kyo`
- `backend_checks_shift_width.kyo`
- `stdlib_result_preserves_linear_error_value.kyo`

Avoid names like:

- `test1.kyo`
- `new_feature.kyo`
- `weird_bug.kyo`

---

## 22. What Not To Do

Do not:

- implement language semantics by accident and document them later
- weaken linearity checks to get examples compiling
- hide resource cleanup in magic helpers
- hide allocation behind convenience APIs
- hide blocking behind ordinary-looking calls
- hide authority behind globals or ambient OS access
- rely on C undefined behavior in generated code
- represent compiler states as raw strings when variants exist
- catch all OCaml exceptions and continue with corrupted state
- use `_` catch-all patterns on closed compiler datatypes when explicit cases matter
- add parser acceptance for undecided syntax in default mode
- let diagnostics lose source spans during lowering
- add broad dependencies for small helpers
- mix large formatting churn with semantic changes
- treat inherited Austral docs as Kyokai behavior without checking current source and accepted shape
- leave generated files stale
- write public examples using syntax that is not accepted or clearly experimental
- treat tests as optional for type-system, linearity, backend, runtime, or stdlib behavior

---

## 23. Command Checklist

Use focused commands. Do not run expensive broad checks when a narrow command proves the change, but do run enough to defend the PR.

Common commands:

```bash
dune build
dune runtest
./run-tests.sh
./run-examples.sh
```

Useful inspection commands:

```bash
rg -n "pattern" lib test test-programs
find lib -maxdepth 1 -type f | sort
sed -n '1,220p' docs/walkthrough.md
```

Before finishing a code change:

- run the relevant test command
- run formatting if the subtree has a formatter
- inspect generated output if it changed
- inspect diagnostics if they changed
- state what was not run and why

---

## 24. Summary

Kyokai code should age like a compiler people can trust. That means typed invariants, narrow passes, explicit ownership, visible authority, clear failure behavior, deterministic generated output, serious tests, and diagnostics that explain the rule being enforced.

The inherited Austral compiler gives Kyokai a working base. OCaml gives the implementation strong tools for modeling invariants. Kyokai's own philosophy supplies the stricter bar: no hidden semantics, no language-level undefined behavior, visible resource flow, and no compiler folklore.

Write code that makes those promises easier to keep.
