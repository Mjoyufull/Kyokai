# Toolchain Overview

[Rikona Kurasaki / Mjoyufull]
Kyokai's toolchain is not a pile of side quests around the language. It is part of the language's public shape because the compiler, package manager, project scaffolder, formatter, docs generator, LSP, test runner, audit report, diagnostic explainer, fix engine, and release artifacts decide what programmers can trust before code ever reaches production.

> Trace: D26, D83, D86, D155
> Covers: The Kyokai toolchain spec is normative, user-visible tooling behavior is specified, and build behavior cannot drift through convention.

This toolchain spec is the companion to the language spec. The language spec owns syntax, typing, evaluation, ownership, contracts, capabilities, unsafe boundaries, concurrency, and backend semantic obligations. This toolchain spec owns how projects are shaped, how commands run, how artifacts are named, how diagnostics are emitted, how packages are resolved, how builds stay reproducible, and how public tooling exposes the same compiler truth.

> Trace: D26, D78, D79, D86
> Covers: Language behavior and toolchain behavior are separate normative surfaces with a shared package, module, artifact, and compiler-engine boundary.

A conforming Kyokai toolchain is centered on one executable named `kyokai`. Subcommands may call helper programs, C compilers, linkers, archive tools, sandbox runners, documentation renderers, or editors' LSP clients, but the user-facing contract is still the `kyokai` toolchain contract. A helper program cannot weaken these rules by being outside the main binary.

> Trace: D26, D31, D80, D86, D225
> Covers: The `kyokai` binary is the public entrypoint, and helper tools remain bound by the same explicit toolchain contract.

Every command that reads source uses the same manifest, workspace, package, module, edition, target, dependency, and `.koi` model. `kyokai check`, `kyokai build`, `kyokai test`, `kyokai doc`, `kyokai lsp`, `kyokai audit`, and `kyokai semver-check` are different views of one project graph, not separate semantic systems.

> Trace: D25-D29, D51, D78-D79, D105, D148, D218, D223
> Covers: All source-facing commands share one project graph and one compiler engine instead of inventing command-specific semantics.

No tool command may make a program legal by skipping a language check. A fast command may stop earlier than a full build, but it must say which later phases were not performed. A documentation command may render only public interfaces by default, but it must not accept an interface the compiler rejects. A formatter may refuse unparseable input, but it must not rewrite invalid code into accepted code by guessing intent.

> Trace: D25, D26, D29, D86, D155
> Covers: Tool speed and presentation choices do not weaken language acceptance or rejection rules.

## Conforming Toolchain Surface

| Area | Required Contract | Trace |
| --- | --- | --- |
| Project shape | `kyokai.toml`, package roots, workspace roots, lockfiles, and module roots are explicit. | D51, D78, D83 |
| Source graph | `.kyo` interfaces, `.kai` bodies, whole-file target selection, declaration guards, imports, and `.koi` artifacts produce one deterministic module graph. | D52, D78-D79, D105, D123 |
| CLI | One `kyokai` binary exposes project creation, health, check, build, run, test, fmt, doc, lsp, audit, explain, fix, bench, repl, eval, package, `.koi`, and release-support commands with stable exit behavior. | D26, D28, D151-D151a, D218, D225, D266-D270 |
| Build policy | Profiles, targets, backends, link mode, debug info, LTO, identical-code folding, and runner choice are written configuration. | D27, D31, D80, D149, D200 |
| Diagnostics | Human and JSON diagnostics carry stable codes, spans, notes, suggestions, explanation catalog entries, automatic-fix applicability, categories, and suppression rules. | D29, D267 |
| Formatting | `kyokai fmt` is deterministic, idempotent, zero-config, and source-semantics preserving. | D25 |
| Testing | Inline tests, doc tests, capability tests, property tests, fuzzing, coverage, and benches use ordinary language semantics with explicit capability authority and replayable reports. | D28, D137, D218, D220, D270 |
| Docs, LSP, audit | Documentation, editor services, and audit reports read the same compiler graph and expose visibility, contracts, unsafe, capability, and dependency facts. | D148, D150, D218 |
| Reproducibility | Artifacts are deterministic over the written build identity; hidden host inputs are excluded. | D83, D144 |
| Ecosystem | Discovery index, package inspection, vendoring, yanks, SemVer checks, CI installation, release artifacts, toolchain health, and provenance are specified. | D157, D221, D223-D225, D244, D268-D269 |
| Generation and exploration | Build generation, `@embedFile`, `eval`, REPL, Compiler Explorer, and sandbox runners are explicit and do not gain hidden authority. | D151-D151a, D226 |

> Trace: D25-D29, D31, D51, D78-D80, D83, D105, D137, D144, D148-D151a, D157, D200, D218, D220-D226, D244, D266-D270
> Covers: The toolchain spec surface includes every accepted toolchain decision family needed for a self-contained Kyokai project.

## Normative Words

The words `must`, `must not`, `required`, `illegal`, `error`, and `reject` describe conformance requirements. The words `may`, `allowed`, and `permitted` describe admitted behavior. The word `should` describes the recommended behavior for quality or ecosystem consistency, but if user-visible behavior depends on it, a later chapter must turn it into a `must` rule.

> Trace: D86, D155
> Covers: Toolchain conformance language is explicit and cannot hide required behavior behind recommendation wording.

If this spec says an implementation may choose a value, the allowed domain, where the choice is recorded, and how users observe it must be stated. If those are not stated, the behavior is outside the current spec and must be rejected or opened as a real design point before implementation relies on it.

> Trace: D83, D86, D155
> Covers: Kyokai does not use open-ended toolchain behavior.

## Shared Inputs

The common build identity includes package and workspace manifests, lockfiles, source contents, generated source declarations and declared inputs, selected profile, selected target, selected backend, language edition, compiler compatibility class, `.koi` compatibility class, target-spec files, explicit environment variables admitted by this spec, and command-line flags that affect output or acceptance.

> Trace: D79, D83, D105, D144, D149
> Covers: Build identity is explicit and shared across artifacts, dependency checking, incremental caches, and reproducible outputs.

Hidden inputs are forbidden. Host locale, timezone, current time, process ID, random seed, directory iteration order, unrelated environment variables, absolute build path, and user shell configuration must not affect accepted source, diagnostics meaning, `.koi` contents, generated C, object code, libraries, or executables unless a chapter defines an explicit opt-in mode.

> Trace: D83
> Covers: Reproducible builds exclude ambient host state by default.

## Shared Outputs

A toolchain may produce human text, machine JSON, `.koi` artifacts, dependency lockfiles, generated source files, generated C, object files, static libraries, dynamic libraries, executables, documentation HTML, documentation JSON, LSP responses, coverage reports, bench reports, audit reports, source maps, debug information, and provenance records. Every normative output must have a deterministic identity rule or be marked as explicitly presentation-only.

> Trace: D27, D29, D79, D83, D144, D218, D225
> Covers: Toolchain outputs are classified as deterministic artifacts or presentation-only reports with stable meaning.

A presentation-only report may include wall-clock durations, terminal color, progress spinners, local path spelling, or other host-presentational facts, but those facts must not be inputs to later compilation, dependency resolution, `.koi` compatibility, SemVer checking, or release provenance.

> Trace: D83, D223, D225
> Covers: Non-reproducible presentation details cannot leak into semantic or release artifacts.

## Governance Boundary

Changing a command's acceptance behavior, manifest schema, artifact compatibility rule, diagnostic JSON schema, target support tier, package-resolution rule, or reproducibility identity is a spec change. Toolchain releases may fix bugs and improve performance, but they must not silently redefine the public contract under an existing edition or compatibility class.

> Trace: D105, D155, D157, D243
> Covers: Toolchain behavior changes require public governance and compatibility handling rather than silent drift.

When an older behavior is retired, the new behavior must state whether it is an edition change, a toolchain-version compatibility change, a package SemVer concern, or a bug fix that restores this spec's existing rule. The release notes must name the category.

> Trace: D105, D157, D223, D243
> Covers: Toolchain changes are categorized through editions, compatibility classes, SemVer, or bug-fix status.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
A language that says ownership is explicit cannot let the build system live off vibes. The command line is where a lot of languages let ghosts into the room: search paths, target defaults, profile folklore, hidden build scripts, registry state, and whatever the editor guessed yesterday. Kyokai makes the doorframe visible. The spec says what the tool reads, what it writes, and what cannot quietly matter.

> Trace: D26, D83, D86, D155
> Covers: Kyokai keeps the toolchain explicit for the same reason it keeps language authority, ownership, and failure explicit.
