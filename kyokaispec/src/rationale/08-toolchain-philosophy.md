# Toolchain Philosophy

[Rikona Kurasaki / Mjoyufull]
A language without a toolchain spec is a city with beautiful laws and no roads. Kyokai does not leave build behavior, package resolution, diagnostics, formatting, docs, audit, and artifacts to rumor. The tools are part of the language experience, so their contracts are written down.

> Trace: D26, D78, D86, D155
> Covers: The toolchain spec is normative and shares the same public source-of-truth discipline as the language spec.

## One Tool, One Semantic Engine

`kyokai init`, `new`, `build`, `check`, `test`, `fmt`, `doc`, `lsp`, `audit`, `explain`, `fix`, `eval`, `repl`, health commands, and package commands all sit around the same manifest and compiler model. The compiler is the linter. The LSP is not a second opinion with different semantics. Formatting does not rewrite meaning.

> Trace: D25-D29, D148, D222, D266-D268
> Covers: Official tools share the compiler engine and preserve semantic consistency.

Diagnostics are part of the contract. Codes, spans, severity, suggestions, explanation pages, safe fixes, JSON output, ordering, and suppression policy matter because people build editors, CI, tests, and habits around them.

> Trace: D29, D225, D267
> Covers: Diagnostics have stable machine-readable and human-facing behavior.

## Packages And Reproducibility

Kyokai uses manifests, workspaces, lockfiles, pinned Git revisions, exact dependency identity, target specs, and `.koi` artifacts because reproducibility is a language trust problem, not just a build-system feature. If two builds mean the same thing, the inputs that decide that fact should be named.

> Trace: D51, D78-D80, D83, D105, D149, D264-D265
> Covers: Package resolution, target selection, build output layout, and `.koi` compatibility are explicit and reproducible.

The official package index is discovery, not a gatekeeper. It can point to packages, docs, versions, metadata, and advisories, and daily commands can search, explain, graph, and vendor those dependencies, but source revisions and lockfiles remain the reproducible truth.

> Trace: D221, D223-D224, D244, D269
> Covers: The package index does not replace pinned source identity or lockfile meaning.

## Artifacts And Interfaces

`.koi` is the checked interface artifact, not a pretty source dump and not an opaque cache. It records the public and internal interface facts downstream tools need: declarations, types, contracts, docs, unsafe audit metadata, imports, hashes, target identity, and compatibility data.

> Trace: D79, D265
> Covers: `.koi` has a concrete KBI-1 artifact format and compatibility contract.

Build outputs and caches are separate because human-inspectable products and disposable compiler machinery have different purposes. `kyokai-out/` is where selected artifacts live. `.kyokai-cache/` is where the toolchain can remember work without becoming part of source meaning.

> Trace: D144, D264
> Covers: Output/cache layout separates products from disposable incremental state.

## Audit And Release Honesty

`kyokai audit` reports dependency, unsafe, FFI, capability, native library, generated-source, license, reproducibility, and API risk facts. It does not bless a program by magic; it gives the review the map it needs.

> Trace: D150, D218, D245
> Covers: Audit tooling surfaces risk metadata without changing semantics or granting authority.

Releases carry compatibility classifications, checksums, provenance, setup metadata, OCI images where provided, and target support notes. Editions remain separate from package SemVer because source meaning and package version communication are not the same thing.

> Trace: D105, D157, D223, D225
> Covers: Release and compatibility policy distinguishes editions, SemVer, and artifact provenance.

## Result

[Rikona Kurasaki / Mjoyufull]
The toolchain philosophy is the same as the language philosophy: no quiet second source of truth. The command, manifest, artifact, diagnostic, explanation, fix, format, audit report, package graph, and release all say what they mean.

> Trace: D86, D155, D264-D270
> Covers: Toolchain behavior is specified rather than inherited from implementation habit.
