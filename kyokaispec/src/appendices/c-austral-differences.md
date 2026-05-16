# Austral Difference Index

[Rikona Kurasaki / Mjoyufull]
Kyokai is a fork, not a denial. Austral remains the source of many good bones: linear types, capability direction, explicit modules, strictness, and a readable spec tradition. This appendix maps where Kyokai carries Austral ideas forward, where it changes them, and where Borretti's Austral wording is credited as source material instead of being presented as Kyokai authorship.

> Trace: D5, D86, Attribution policy
> Covers: Kyokai is self-contained while preserving traceable Austral lineage and attribution.

## Difference Families

| Family | Kyokai Direction | Normative Home | Trace |
| --- | --- | --- | --- |
| Project identity | Kyokai uses `Kyokai.*`, `.kyo`, `.kai`, Kyokai package manifests, and Kyokai-owned terminology. | `language/00-introduction.md`, `language/04-modules-and-visibility.md`, `toolchain/02-module-resolution-and-koi.md` | D1, D5, D52, D78 |
| Spec shape | Kyokai has language, toolchain, stdlib, rationale, and appendix families instead of one inherited language-only document. | `language/00-introduction.md`, `toolchain/00-toolchain-overview.md`, `stdlib/00-stdlib-overview.md` | D86, D152, D229 |
| Syntax | Kyokai changes terminators, borrow spelling, pattern forms, fallible control flow, generics, records, modules, and comments. | `language/02-lexical-syntax.md`, `language/03-grammar.md` | D7b-D19a, D35-D43, D52, D63 |
| Error model | Kyokai splits `Result`, `Optional`, panic, TPOE, runtime-fatal, allocation failure, and `StandardError`. | `language/13-contracts-and-runtime-failure.md`, `stdlib/02-core-result-optional-display-error.md` | D24, D53, D74, D84, D253, D259 |
| Ownership/borrows | Kyokai keeps linear ownership but specifies regions, moves, invalidation, pinning, deferred cleanup, and elaboration. | `language/11-linearity-borrowing-and-regions.md` | D6, D77, D89, D89a-D89b, D187, D238-D240 |
| Capabilities | Kyokai keeps capability security and expands sealed declarations, root authority, stdlib authority APIs, audit, and task transfer. | `language/14-capabilities-and-authority.md`, `stdlib/08-io-files-env-process-time-random.md` | D48, D67, D171, D211-D212, D255 |
| Concurrency | Kyokai adds structured tasks, SPSC channels, select, pollers, locks, atomics, cancellation, and signal handling. | `language/15-concurrency.md`, `stdlib/09-concurrency-primitives.md` | D3, D90-D101, D146, D183-D184, D256 |
| Unsafe/FFI/backend | Kyokai specifies unsafe contracts, C ABI, dynamic loading, generated-C safety, LLVM path, volatile/MMIO, and no language UB. | `language/16-unsafe-ffi-and-abi.md`, `language/17-memory-layout-and-backend-contract.md` | D20, D73, D113a-D113b, D139, D228, D245 |
| Standard library | Kyokai replaces inherited stdlib assumptions with admitted `Kyokai.*` API families and explicit contracts. | `stdlib/00-stdlib-overview.md` through `stdlib/11-transitional-ffi-tracking.md` | D64, D85, D152, D229-D232 |
| Toolchain | Kyokai specifies CLI, manifests, packages, `.koi`, formatter, tests, docs, LSP, audit, reproducibility, releases, and output/cache layout. | `toolchain/00-toolchain-overview.md` through `toolchain/11-build-generation-and-playground.md` | D25-D31, D51, D78-D80, D83, D218, D264-D265 |

> Trace: D5, D86, D152, D229
> Covers: Differences are indexed by family and point to normative Kyokai chapters.

## Austral Source Replacement Map

The old inherited Austral chapter files are out of the active Kyokai build after replacement by Kyokai chapters. They remain important source history in git and in Borretti's Austral project, but the reader should use the Kyokai chapters below for current behavior. `src/appendix-a.md` is different: it carries the full GFDL text included with the built specification.

| Austral Source Area | Kyokai Replacement / Status |
| --- | --- |
| Introduction | `language/00-introduction.md` and rationale chapters. |
| Design goals | `language/01-goals-and-non-goals.md` plus rationale. |
| Syntax | `language/02-lexical-syntax.md`, `language/03-grammar.md`, `rationale/02-syntax.md`. |
| Modules | `language/04-modules-and-visibility.md`, toolchain module resolution chapters. |
| Types and type classes | `language/06-type-system.md`, `language/07-generics-and-typeclasses.md`. |
| Declarations | `language/05-declarations.md`. |
| Statements | `language/10-statements-and-control-flow.md`. |
| Expressions | `language/09-expressions-and-evaluation.md`. |
| Linearity | `language/11-linearity-borrowing-and-regions.md`, `rationale/04-linear-resources.md`. |
| Built-ins and pervasive APIs | `language/18-built-ins.md` and stdlib chapters. |
| Memory support | Memory/layout/backend and allocator stdlib chapters. |
| FFI | `language/16-unsafe-ffi-and-abi.md`, `stdlib/11-transitional-ffi-tracking.md`. |
| Examples and style | `language/19-examples.md`, syntax/naming rules, and future style guidance. |
| Rationale | `rationale/00-rationale-index.md` through `rationale/08-toolchain-philosophy.md`; Borretti wording is marked as source text when reused. |
| Full GFDL text | `src/appendix-a.md`, navigated by `appendices/a-license.md`. |

> Trace: D5, D86, Attribution policy
> Covers: Old Austral chapter files have Kyokai replacements, Borretti source wording is credited when used, and the GFDL text remains included.

## Reader Rule

If a new Kyokai chapter and an inherited Austral file overlap, read the Kyokai chapter as the current normative home. If Borretti's Austral wording is worth using, put it in the Kyokai chapter or rationale with a source note instead of making the reader reconcile two documents.

> Trace: D5, D86, D155
> Covers: The Kyokai spec is self-contained and inherited files do not override extracted Kyokai chapters.
