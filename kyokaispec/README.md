# Kyokai Specification Workspace

This directory is the home of the Kyokai specification family: the core language specification, the companion toolchain specification, the standard-library contract specification, rationale chapters, and appendices.

The current extracted Kyokai chapters live under:

| Directory | Role |
| --- | --- |
| `src/language/` | Normative language rules: syntax, types, ownership, evaluation, contracts, capabilities, concurrency, unsafe/FFI, layout, built-ins, and examples. |
| `src/toolchain/` | Normative toolchain rules: manifests, packages, `.koi`, CLI, profiles, diagnostics, formatting, tests, docs, LSP, audit, reproducibility, package index, releases, generation, and playground behavior. |
| `src/stdlib/` | Normative standard-library contract rules: admission, API contract fields, allocators, containers, text/paths, collections, iterators, math, I/O, concurrency, crypto, and transitional FFI tracking. |
| `src/rationale/` | Non-normative rationale with attribution. These chapters explain why rules exist; they do not replace the normative chapters. |
| `src/appendices/` | License/provenance, decision traceability, Austral difference index, source-material replacement map, and formalization roadmap. |

## Source-Of-Truth Order

1. `kyokaispec/src/language/`, `kyokaispec/src/toolchain/`, `kyokaispec/src/stdlib/`, and `kyokaispec/src/appendices/` once a rule is written there.
2. `../kyokaidecided.md` for accepted Kyokai shape not yet spec-extracted.
3. `../Kyokaishape.md` for live public D-points and pending shape.
4. Linked public discussions, issues, and PRs.
5. `../phase.md` for implementation/proof order only.
6. Root `../../kyokaiplan.md` as the older full design archive when it does not conflict with accepted shape or extracted spec text.

The old inherited Austral chapter files have been removed from the active build after their Kyokai replacements were extracted. Austral remains source material and lineage, but current Kyokai rules live in the Kyokai chapters listed above. See `src/appendices/c-austral-differences.md` for the replacement map.

## Status

| Area | Status | Notes |
| --- | --- | --- |
| Kyokai spec structure | Extracted through Phase 15 | `kyokaispecdirection.md` records Phases 0-15 as complete. |
| Kyokai language spec text | Extracted | `src/language/00-introduction.md` through `src/language/19-examples.md`. |
| Kyokai toolchain spec text | Extracted | `src/toolchain/00-toolchain-overview.md` through `src/toolchain/11-build-generation-and-playground.md`. |
| Kyokai stdlib contract spec | Extracted | `src/stdlib/00-stdlib-overview.md` through `src/stdlib/11-transitional-ffi-tracking.md`. |
| Rationale rewrite | Extracted | `src/rationale/00-rationale-index.md` through `src/rationale/08-toolchain-philosophy.md`. |
| Appendices and traceability | Extracted | `src/appendices/a-license.md` through `src/appendices/d-formalization-roadmap.md`. |
| Austral source material | Replaced in active spec tree | Replacements are indexed in `src/appendices/c-austral-differences.md`; Borretti's Austral wording is credited when directly used. |
| Compiler/source conformance | Not yet verified as Kyokai implementation | `SPEC_COMPILER_TRACE.md` separates extracted spec text from inherited Austral compiler evidence and implementation gaps. |
| Formal proof | Roadmap extracted, proof not drafted | `src/appendices/d-formalization-roadmap.md` records the `lambda_K-seq` paper-proof obligation. |

## Building The Extracted Specification

To verify the source list used by the build:

```bash
make check-sources
```

To generate the default HTML output from the extracted Kyokai chapter tree:

```bash
make
```

To build both HTML and PDF, install a Pandoc-supported PDF engine such as `pdflatex` and run:

```bash
make all
```

To build only the PDF, use:

```bash
make pdf
```

If your PDF engine is not `pdflatex`, pass it explicitly:

```bash
make pdf PDF_ENGINE=xelatex
```

To remove generated output:

```bash
make clean
```

The generated `spec.pdf` and `spec.html` are build products. The source-of-truth text remains the Markdown files under `src/`.

## License

The specification prose keeps the GNU Free Documentation License 1.3-or-later documentation license from the inherited spec tree, with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts. See `COPYING`, `src/appendix-a.md`, and `src/appendices/a-license.md`.

This documentation license is separate from the Kyokai code license boundary recorded in `../kyokaidecided.md` and `src/appendices/a-license.md`.
