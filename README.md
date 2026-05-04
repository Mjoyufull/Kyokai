# Kyokai

Kyokai is a systems programming language forked from Austral and built around explicit resource ownership, linear types, capability-based authority, and defined failure behavior.

Kyokai is currently in early public development. The accepted language shape has been extracted, but the normative Kyokai spec and compiler implementation are still being brought up from the inherited Austral codebase.

## Current Status

| Area | Status |
| --- | --- |
| Accepted language shape | Present in `kyokaidecided.md`. |
| Live public D-points | Tracked in `Kyokaishape.md`. |
| Normative Kyokai spec | Not started yet; future home is `kyokaispec/`. |
| Compiler | Inherited Austral OCaml compiler, being forked toward Kyokai. |
| Standard library | Inherited Austral stdlib, being redesigned/admitted under Kyokai rules. |
| Roadmap | `phase.md`. |
| Code standards | `CODE_STANDARDS.md`. |
| Project workflow | `PROJECT_STANDARDS.md`. |

Inherited Austral examples, commands, docs, and spec files may still exist while the fork is being converted. Treat accepted Kyokai shape and future Kyokai spec text as the language authority, not stale inherited prose.

## Core Direction

Kyokai keeps Austral's most important safety ideas and pushes them toward a production systems language:

- linear ownership for resources
- capability-based authority instead of ambient global access
- no language-level undefined behavior
- explicit allocation, blocking, cleanup, and FFI boundaries
- TPOE for contract violations
- safe native standard-library implementations where practical
- explicit compiler/spec/test traceability

## Source-Of-Truth Order

1. `kyokaispec/` once a Kyokai rule is written there.
2. `kyokaidecided.md` for accepted Kyokai shape not yet spec-extracted.
3. `Kyokaishape.md` for live public D-points and pending shape.
4. Linked public discussions, issues, and PRs.
5. `phase.md` for implementation order only.

`phase.md` is not a language spec. It records ordering, gates, and status.

## Repository Map

| Path | Purpose |
| --- | --- |
| `lib/` | Current OCaml compiler source inherited from Austral. |
| `standard/` | Current inherited standard-library tree. |
| `test/` | Unit-level compiler tests. |
| `test-programs/` | End-to-end compiler tests inherited from Austral. |
| `kyokaispec/` | Future Kyokai specification workspace; currently inherited Austral staging material. |
| `kyokaidecided.md` | Accepted Kyokai shape and maturity tracker. |
| `Kyokaishape.md` | Public live D-point tracker. |
| `phase.md` | Implementation/proof roadmap. |
| `CODE_STANDARDS.md` | Mandatory code standards. |
| `PROJECT_STANDARDS.md` | Public project workflow. |

## Building The Current Inherited Compiler

The compiler is still inherited from Austral, so some package names and commands may still say `austral` until the fork identity work is complete.

With Nix:

```bash
nix-shell
make
```

Without Nix, install OCaml, Dune, and opam dependencies, then run:

```bash
opam install --deps-only -y .
make
```

Run tests with:

```bash
./run-tests.sh
```

Build the inherited standard library with:

```bash
cd standard
make
```

## Spec Workspace

`kyokaispec/` is intentionally kept inside this repository so spec text, compiler changes, tests, and phase tracking can move together.

The spec source can later be published as a GitHub Pages site by building HTML/PDF from `kyokaispec/` and deploying only generated artifacts to a `gh-pages` branch or generated-docs path. The source should stay with the compiler unless there is a clear maintenance reason to split it.

## License

Kyokai follows a split license boundary:

- compiler and toolchain code: `GPL-3.0-or-later`
- runtime, standard library, startup code, compiler support library, target-side helpers, and compiler-emitted target helper code: `GPL-3.0-or-later WITH GCC-exception-3.1`

The runtime exception keeps ordinary Kyokai target programs under the program author's chosen license when they merely use the Kyokai runtime, standard library, or exception-covered target helpers through the normal compilation/linking process.

See `LICENSE`, `COPYING`, and `COPYING.RUNTIME` for details.

Inherited Austral files may still carry Apache-2.0 WITH LLVM-exception notices. Preserve those notices unless the file is replaced or relicensed by the relevant copyright holder.
