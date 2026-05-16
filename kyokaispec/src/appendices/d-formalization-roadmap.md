# Formalization Roadmap

[Rikona Kurasaki / Mjoyufull]
This appendix records the proof road. It does not pretend the proof is already done. Kyokai's prose can state the language contract, but the sequential ownership core still needs the paper theorem before `v1.0` and later mechanization after self-hosting.

> Trace: D143/D241
> Covers: Kyokai requires a paper proof for the sequential core before `v1.0` and later mechanized proof work.

## Proof Scope

The first calculus is `lambda_K-seq`: a small sequential core for ownership, borrowing, moves, regions, linear consumption, pattern binding, control-flow joins, checked failure, and TPOE. It deliberately excludes concurrency, FFI, backend lowering, package resolution, the full stdlib, and toolchain behavior.

> Trace: D143/D241, D90, D245
> Covers: The first formalization is sequential and excludes concurrency, FFI, backend, and stdlib/toolchain layers.

## Required Paper Artifact

The paper proof must define abstract syntax, typing judgments, store/ownership state, borrow-region state, small-step operational semantics, checked failure/TPOE outcomes, progress, preservation, and the exact theorem statement for the safe sequential core. Any gap must be named as an assumption or excluded feature.

> Trace: D73, D143/D241
> Covers: The paper proof must specify syntax, typing, semantics, failure outcomes, and soundness theorem scope.

## Milestones

| Milestone | Output | Status Meaning |
| --- | --- | --- |
| Scope freeze | Feature list for `lambda_K-seq` with exclusions. | The calculus knows what it is proving. |
| Core syntax | Abstract syntax for expressions, statements, owners, borrows, regions, and failure forms. | Surface syntax is not required; elaborated core is enough. |
| Static judgments | Typing, ownership, borrowing, movement, region, and control-flow join judgments. | The checker rules have a proof-facing shape. |
| Dynamic semantics | Small-step semantics with store, ownership state, and named failure outcomes. | Evaluation behavior is explicit. |
| Soundness theorem | Progress/preservation adapted to linear ownership and TPOE. | The theorem says what cannot go wrong and what may terminate. |
| Paper proof | Human-checkable proof document. | Required before `v1.0`. |
| Mechanized plan | Proof assistant selection and encoding strategy. | Planned after self-hosting; Coq is the current likely first target. |

> Trace: D143/D241
> Covers: Formalization work proceeds through explicit artifacts instead of vague proof intent.

## Relation To Existing Notes

`kyokailang/kyokaicalculus/lambda_k_research.md` is the current research note and scope map. It is not itself the normative proof. When a proof artifact is produced, this appendix must point to the theorem, proof document, assumptions, and excluded features.

> Trace: D143/D241
> Covers: The research note informs proof work but does not replace the required proof artifact.

## Later Extensions

After the sequential proof, later formalization work can add concurrency happens-before reasoning, task transfer, channels, cancellation, unsafe/FFI contracts, backend lowering preservation, stdlib contract models, and mechanized proof. These layers should not be started by blurring the first theorem; they should extend it after the core is stable.

> Trace: D90-D101, D143/D241, D228, D245
> Covers: Concurrency, unsafe/FFI, backend, and stdlib proof work are later extensions.

## Proof Honesty Rule

Until a theorem is actually discharged, spec text may say language contract, design goal, or intended invariant. It must not claim a mechanized or paper proof exists. When the proof changes a rule, the normative chapter and traceability appendix must be updated in the same pass.

> Trace: D143/D241, D155
> Covers: The spec must not overclaim proof status and must keep proof-driven rule changes traceable.
