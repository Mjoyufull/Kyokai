# Introduction

Kyokai is a systems programming language built around explicit boundaries: ownership boundaries, authority boundaries, module boundaries, failure boundaries, and toolchain boundaries. Kyokai inherits important ideas from Austral, including linear resource tracking, capability-oriented security, and rejection of hidden cleanup, but Kyokai is its own language and its own project. The `Kyokai.*` namespace marks that break. Existing Austral text is evidence and source material until it is rewritten, checked, and accepted as Kyokai text.

> Trace: D1, D5, D145
> Covers: Kyokai uses the `Kyokai.*` namespace, is a hard fork rather than a light rename of Austral, and is designed first for C programmers who want modern safety and codebase readability.

The Kyokai specification is a family of normative documents, but the language specification itself is not an addendum to the Austral specification. A reader should not have to open Austral's spec to understand the Austral-shaped parts of Kyokai, though Fernando Borretti's writing remains important source material for the fork. When Kyokai keeps an Austral rule, this specification restates that rule here, checks it against Kyokai's accepted shape, and marks any exact or near-exact wording as source text from Borretti's Austral specification.

> Trace: D5, D86, D155
> Covers: Kyokai is a hard fork that preserves compatible Austral rules in-place when they still apply, and the Kyokai language spec must be self-contained rather than requiring the reader to consult the Austral spec for inherited behavior.

The Kyokai specification family includes the core language specification, the companion toolchain specification, the standard-library contract specification, rationale chapters, and appendices. The core language specification defines source syntax, static checking, types, ownership, borrowing, evaluation, control flow, capabilities, contracts, unsafe code, FFI, runtime failure, layout, and backend-observable language behavior. The toolchain specification defines manifests, packages, workspaces, build artifacts, the CLI, diagnostics, formatting, LSP behavior, tests, documentation generation, target support, reproducibility, and related tool behavior. The standard-library contract specification defines the public API families, admission requirements, contracts, edge cases, allocation behavior, capability requirements, platform boundaries, and test obligations for the batteries-included Kyokai standard library.

> Trace: D85, D86, D152, D229
> Covers: Kyokai has separate normative language, toolchain, standard-library, rationale, and appendix materials; standard-library APIs require explicit semantic contract fields, the stdlib is a core project commitment, and stdlib admission requires contracts, edge-case rules, tests or oracles, and compatibility boundaries.

When a Kyokai rule is written in the normative spec, that spec text is the authority for the rule. Until a rule is extracted, `kyokaidecided.md` is the accepted public shape, `Kyokaishape.md` tracks live public decisions, and root `kyokaiplan.md` remains the older full decision archive and rationale source. `phase.md` gives implementation order; it does not define language semantics. If implementation behavior disagrees with the normative spec, the disagreement is a bug in the implementation, the spec, or both, and must be resolved explicitly.

> Trace: D86, D155
> Covers: Accepted behavior belongs in the normative language or toolchain spec, the spec/compiler relationship is governed by a public decision process, and divergence between accepted spec behavior and compiler behavior is not allowed to drift silently.

A conforming Kyokai implementation must define the behavior of every accepted safe Kyokai program at the language level. Safe Kyokai programs must not rely on language-level undefined behavior, backend poison, invalid aliasing, unchecked trap-producing operations, or C undefined behavior in generated code. If a behavior cannot be defined safely, the language must reject the program, route the operation through an explicitly unsafe contract, return an ordinary explicit failure value, or terminate through a specified Kyokai failure path.

> Trace: D87, D139, D143/D241, D228, D253, D262
> Covers: Kyokai treats zero language-level undefined behavior as a core language contract, requires formalization before `v1.0`, requires backend lowering to preserve Kyokai semantics without backend UB, and defines fatal paths such as panic, TPOE, and stack overflow instead of leaving them undefined.

Kyokai does not hide semantically important work. The compiler may insert or complete an omitted operation only when the operation is uniquely determined by static types and context, every alternative program is ill-typed, and the completion adds no new control flow, allocation, or side effects beyond what the explicit program already implies. If those conditions do not hold, the programmer must write the operation.

> Trace: D87, D238, D239, D240
> Covers: Kyokai permits only effect-neutral tautological implicit completions, fixes the elaboration order before checking, requires a compiler-maintained implicit-completion registry, and requires conformance tests for auto-reborrow and read-reborrow soundness.

Text from Fernando Borretti's Austral specification may be used when it is technically correct for Kyokai, still reads well, and is marked as source text from Borretti's Austral work. New Rikona Kurasaki / Mjoyufull prose is the default for Kyokai-owned explanation and rationale. Paragraphs that combine Kyokai wording with Austral source wording must say which part comes from Borretti's Austral work instead of presenting Borretti as a Kyokai author. Attribution credits prose origin only; it does not make source wording normative unless the paragraph also states a Kyokai rule that has been checked against accepted Kyokai behavior.

> Trace: Attribution policy
> Covers: Borretti wording is marked as source text from the Austral specification, Kyokai wording is authored under Kyokai, and attribution is separate from semantic authority; internal writing-style notes are not public trace sources.

The Kyokai specification keeps the existing FSF documentation license carried by the inherited spec tree: GNU Free Documentation License 1.3, included in the license appendix. This documentation license is separate from the code license boundary for Kyokai-owned compiler, toolchain, runtime, standard-library, startup, support, and target helper code.

> Trace: Appendix A, D263
> Covers: Kyokai spec prose remains under the existing GNU Free Documentation License 1.3 documentation license; the separate Kyokai code license boundary still distinguishes compiler/toolchain code from target-linked runtime, stdlib, and helper code.
