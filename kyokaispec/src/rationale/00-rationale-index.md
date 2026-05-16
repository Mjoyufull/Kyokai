# Rationale Index

[Rikona Kurasaki / Mjoyufull]
Rationale is not the law. It is the light left on beside the law, so a reader can see why the rule has this shape and why another tempting shape was refused. Kyokai keeps this separation on purpose: normative chapters say what programs mean; rationale chapters explain why the design was worth choosing.

> Trace: D86, D155
> Covers: Rationale is non-normative commentary and accepted behavior remains in the language, toolchain, stdlib, and appendix chapters.

## Reading Order

| Chapter | Subject | Normative Home |
| --- | --- | --- |
| `01-language-design.md` | Fork identity, simplicity, no language-level UB, and proof honesty. | `language/00-introduction.md`, `language/01-goals-and-non-goals.md` |
| `02-syntax.md` | Statement orientation, boundary terminators, naming, and explicit surface forms. | `language/02-lexical-syntax.md`, `language/03-grammar.md` |
| `03-error-handling-and-tpoe.md` | Recoverable failures, contracts, panic, TPOE, runtime-fatal, and allocation failure. | `language/10-statements-and-control-flow.md`, `language/13-contracts-and-runtime-failure.md` |
| `04-linear-resources.md` | Linear ownership, borrows, resource lifecycles, and visible cleanup. | `language/11-linearity-borrowing-and-regions.md` |
| `05-capabilities.md` | Capability security, root authority, unsafe boundaries, and ambient authority rejection. | `language/14-capabilities-and-authority.md` |
| `06-concurrency.md` | Structured tasks, SPSC channels, pollers, locks, atomics, and cancellation. | `language/15-concurrency.md`, `stdlib/09-concurrency-primitives.md` |
| `07-stdlib-philosophy.md` | Batteries-included stdlib, admission records, RIIK, math, crypto, and FFI tracking. | `stdlib/00-stdlib-overview.md` through `stdlib/11-transitional-ffi-tracking.md` |
| `08-toolchain-philosophy.md` | Packages, `.koi`, CLI, diagnostics, formatter, docs, audit, reproducibility, and releases. | `toolchain/00-toolchain-overview.md` through `toolchain/11-build-generation-and-playground.md` |

> Trace: D85, D86, D152, D229
> Covers: Rationale chapters map back to the normative language, toolchain, and stdlib specs.

## Attribution Rule

Rationale may use Fernando Borretti's Austral design claims where Kyokai still shares the rule. Exact or near-exact wording from the Austral specification must be marked as source text from Borretti's Austral work, while Kyokai-owned explanatory prose is by Rikona Kurasaki / Mjoyufull. Do not use joint attribution tags that imply Borretti helped author Kyokai.

> Trace: Attribution policy
> Covers: Public source notes credit Austral prose lineage without making Borretti a Kyokai author or making rationale normative.

## How To Use This Section

A reader should use rationale to understand pressure, tradeoffs, and prior art, then return to the normative chapter for exact syntax, static rules, dynamic behavior, and conformance obligations. If rationale and normative text ever disagree, the normative chapter wins and the rationale must be repaired.

> Trace: D155
> Covers: Spec governance prevents rationale from becoming a hidden source of behavior.
