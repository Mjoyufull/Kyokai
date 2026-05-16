# Goals And Non-Goals

[Rikona Kurasaki / Mjoyufull]
Kyokai inherits more than a compiler skeleton from Austral. It inherits pressure: the belief that linear types, explicit modules, capability security, strictness, and restraint can make systems code easier to reason about. Kyokai changes a lot around that core. It grows a real toolchain, a larger standard library contract, a stronger no-language-UB promise, explicit runtime failure categories, and a broader production surface. But when an Austral goal still belongs to Kyokai, this specification carries it forward here instead of making the reader go dig through another spec.

> Trace: D5, D86, D145, D155
> Covers: Kyokai is both a hard fork and a continuation of compatible Austral design goals, and the Kyokai spec must restate inherited rules directly so it stands alone as a full language specification.

[Rikona Kurasaki / Mjoyufull]
The first goal is fits-in-head simplicity. Borretti's Austral phrase still matters because it names the real test: a programmer should be able to read the specification and understand the whole language model without needing to partake in compiler archaeology, or reading a second language's manual. Kyokai is larger than Austral because it specifies the toolchain and batteries-included standard library too, so simplicity cannot mean smallness alone. It means the language has few overlapping mechanisms, one explicit surface for each semantic job, and no silent second way hiding under the first.

> Trace: D5, D86, D87, D147
> Covers: Kyokai keeps the inherited simplicity goal while applying it to a larger self-contained language, toolchain, and stdlib spec; simplicity means low overlapping mechanism count and directly specified behavior, not dependence on Austral as an external manual.

[Rikona Kurasaki / Mjoyufull]
The second goal is readability. Borretti's Austral line is still clean: "We are not typists, we are programmers." Kyokai optimizes for code that can be read under pressure, even when that costs a few extra characters. Terminators, explicit borrows, explicit capabilities, explicit failure arms, and named construction all follow the same rule: the source should show the boundary that matters.

> Trace: D9, D11a, D14, D15, D15a, D35, D53, D145
> Covers: Kyokai keeps readability as a design goal and favors visible boundaries in syntax, ownership, construction, contracts, and failure handling.

The third goal is defined safe execution. A safe Kyokai program accepted by a conforming implementation must have specified language behavior. Runtime checks may fail, contracts may terminate, allocation may return explicit errors, and unsafe contracts may impose obligations, but safe source code must not fall through a gap where the language refuses to say what happens.

> Trace: D84, D87, D139, D143/D241, D228, D253, D262
> Covers: Kyokai rejects language-level undefined behavior for safe programs, defines panic/TPOE/fatal outcomes, requires backend lowering that preserves Kyokai semantics, and records formalization work as a required pre-`v1.0` obligation.

The fourth goal is explicit ownership. Linear values must be consumed exactly through visible language operations. Kyokai does not insert hidden destructors, hidden copies, hidden `Drop` calls, or invisible cleanup at ordinary scope exit. Cleanup helpers such as `defer` and `errdefer` are source-level control-flow constructs with explicit checker states and specified exit behavior.

> Trace: D2, D2b, D15, D15a, D87, D124, D207, D208, D227, D246
> Covers: Kyokai uses visible cleanup constructs, rejects implicit destruction, specifies defer and errdefer states and exit-path behavior, and keeps ownership effects explicit in source.

The fifth goal is explicit authority. Code does not receive ambient access to the machine merely because it exists in the process. Files, environment variables, process creation, randomness, wall-clock time, dynamic loading, unsafe operations, and similar authority-bearing operations must be reachable through explicit capabilities, handles, unsafe contracts, or standard-library APIs whose contracts say what authority is required.

> Trace: D45, D67, D95, D113a, D113b, D162, D171, D172, D173, D178, D211, D212, D245, D255, D256
> Covers: Kyokai uses capability values and sealed authority tokens, startup mints root authority explicitly, stdlib authority APIs require explicit capabilities, unsafe operations require audited unsafe contracts, and capabilities are not forgeable from raw bits.

[Rikona Kurasaki / Mjoyufull]
The sixth goal is correctness through mechanical aid. Human review matters, but tired people under schedule pressure cannot be the only line between a program and memory corruption, authority leaks, or broken resource lifetimes. Kyokai follows Austral's restraint here: type checking, linearity checking, capability checking, contracts, static assertions, runtime checks, and formalization all exist because the machine should shoulder every rule it can check without making the language too tangled to implement or understand.

> Trace: D53, D84, D87, D143/D241, D145, D211, D229
> Covers: Kyokai uses mechanical checks for correctness and security while balancing those checks against implementability, specification clarity, and the no-language-UB contract.

The seventh goal is honest implicitness. Kyokai is not a language where every convenience is banned. It is a language where omitted operations are allowed only when the missing operation is forced by the surrounding program and does not add hidden control flow, allocation, or side effects. This permits ergonomics such as carefully bounded reborrows and implicit `Unit` completion while rejecting guesswork.

> Trace: D7b, D8, D12, D34, D46, D87, D187, D238, D239, D240
> Covers: Kyokai admits only tautological effect-neutral implicit completions, including selected borrow and inference conveniences, and checks those completions through a fixed elaboration pipeline.

[Rikona Kurasaki / Mjoyufull]
The eighth goal is maintainability. Borretti's Austral point is still right: code does not stay alive by magic. Interfaces rot when they are vague, dependencies shift when the boundary is soft, and language features become debt when nobody can say no. Kyokai aims for code that can survive years because the module surface, package surface, contract surface, and standard-library surface are written down before people build towers on top of them.

> Trace: D78, D79, D85, D86, D105, D147, D155, D229
> Covers: Kyokai treats stable interfaces, explicit artifacts, editions, package/workspace boundaries, stdlib contracts, and public decision governance as maintainability requirements.

[Rikona Kurasaki / Mjoyufull]
The ninth goal is modularity. Kyokai keeps the inherited interface/body split because it is still one of Austral's strongest bones: a module can publish a checked surface before its body exists, and another module can typecheck against that surface without seeing the private implementation. Kyokai then tightens the world around it with `.kyo`/`.kai` source files, package-visible `internal`, deterministic module resolution, and `.koi` interface artifacts.

> Trace: D17, D52, D78, D79, D179, D214
> Covers: Kyokai preserves Austral's module-interface/module-body separation while specifying file extensions, import forms, visibility, module resolution, and interface artifacts as Kyokai rules.

[Rikona Kurasaki / Mjoyufull]
The tenth goal is strictness with purpose. Kyokai should break the build when a boundary is unclear, when a resource has not been consumed, when an import collision would make a name unstable, when a contract is violated, or when a platform-specific declaration does not resolve cleanly. This brittleness is not aesthetic punishment. It is the language refusing to let uncertainty pass as meaning.

> Trace: D15a, D17, D19a, D53, D60, D78, D87, D214, D246
> Covers: Kyokai intentionally rejects ambiguous names, unresolved platform declarations, hidden resource disposal, and vague contracts at compile time or through specified failure paths.

The eleventh goal is production completeness. Kyokai is not only a core calculus with a small compiler around it. The language must ship with a serious toolchain, package model, diagnostics, formatter, tests, documentation, LSP, audit support, reproducible builds, and a batteries-included systems standard library whose APIs have explicit contracts.

> Trace: D25, D26, D28, D29, D51, D78, D83, D85, D86, D136, D137, D148, D150, D152, D218, D219, D220, D221, D222, D225, D226, D229
> Covers: Kyokai commits to a normative toolchain spec, package/workspace behavior, diagnostics, formatting, testing, docs, LSP, audit, reproducibility, playground support, and a batteries-included stdlib with admission and contract requirements.

The twelfth goal is implementation honesty. The C backend remains important for bootstrap, reference behavior, inspection, and target bring-up, but C does not define Kyokai. Backend and target limitations must be written as backend or target contracts. They must not silently weaken the language or turn accepted safe Kyokai into C undefined behavior.

> Trace: D4, D31, D80, D139, D141, D149, D228
> Covers: Kyokai keeps C as a first-class backend, plans LLVM as the long-term optimizing backend after self-hosting, and requires backend contracts and generated-code behavior to preserve Kyokai semantics explicitly.

The thirteenth goal is formal honesty. Kyokai may state intended invariants in prose while the proof work is still being built, but it must not pretend that prose has already discharged the proof burden. Before `v1.0`, Kyokai must produce a paper proof for the sequential ownership-and-borrowing core, with later mechanized proof work after self-hosting.

> Trace: D143/D241
> Covers: Kyokai requires a pre-`v1.0` paper proof for the sequential `lambda_K` core, treats `lambda_k_research.md` as a research note rather than a normative proof, and plans later mechanization after self-hosting.

Kyokai has the following normative non-goals:

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

> Trace: D147
> Covers: Kyokai's non-goals are normative project boundaries, not soft preferences; reversing any listed boundary requires a new explicit public decision.

A later chapter may add a new non-goal through the same public decision process, but no chapter may quietly cross one of the boundaries above. If a feature appears to need a forbidden mechanism, the spec must either reject the feature, redesign it around an existing explicit Kyokai mechanism, or open a new decision that names the boundary being crossed and the consequences of crossing it.

> Trace: D147, D155
> Covers: Non-goals may change only through the explicit public decision process, and accepted behavior must remain traceable instead of drifting through adjacent spec edits.
