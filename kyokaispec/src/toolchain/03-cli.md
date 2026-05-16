# Command Line Interface

[Rikona Kurasaki / Mjoyufull]
The `kyokai` command should feel direct, but direct does not mean loose. Every command enters through the same manifest door, resolves the same project graph, and either reports exactly what it did or fails before it invents a shape for the project.

> Trace: D26, D78, D83
> Covers: The CLI is manifest-driven, deterministic, and shared by all project-facing commands.

## Command Discovery And Scope

Unless a command explicitly accepts no project, `kyokai` discovers the nearest `kyokai.toml` from the current directory upward. A package manifest gives standalone-package scope. A workspace manifest gives workspace scope. If a member package is selected from within a workspace, dependency and lockfile behavior still belongs to the workspace that explicitly lists that member.

> Trace: D26, D78
> Covers: CLI project scope follows nearest manifest discovery and explicit workspace membership.

The global flag `--manifest-path <path>` selects a manifest directly. The path must name a `kyokai.toml` file, not just a directory. The tool must reject a path that does not parse as a valid package or workspace manifest before it reads source files.

> Trace: D26, D78, D29
> Covers: Explicit manifest selection is allowed, and invalid manifests fail before source interpretation.

The flags `--workspace` and `--package <name>` select scope inside a workspace. `--workspace` selects all members. `--package <name>` selects exactly one workspace member by package name. Using both flags in one command is an error unless that command explicitly defines a combined meaning; no Phase 12 command does.

> Trace: D26, D78
> Covers: Workspace and package selection is explicit and non-overlapping.

## Common Flags

| Flag | Meaning | Applies To | Trace |
| --- | --- | --- | --- |
| `--manifest-path <file>` | Use the named manifest instead of nearest-manifest discovery. | Project commands | D26, D78 |
| `--workspace` | Select all members of an enclosing workspace. | Project commands | D26, D78 |
| `-p`, `--package <name>` | Select one workspace member by package name. | Project commands | D26, D78 |
| `--profile <name>` | Select a build profile. | check, build, run, test, doc, bench | D26, D31 |
| `--release` | Exact alias for `--profile release`. | check, build, run, test, doc, bench | D26, D31 |
| `--target <triple>` | Select a legal target triple. | check, build, run, test, doc, bench | D19, D80, D149 |
| `--backend <c|llvm>` | Select the code generation backend. | build, run, test, bench | D31, D139, D149 |
| `--format human|json` | Select human or machine-readable diagnostics/report output. | All reporting commands | D29 |
| `--color auto|always|never` | Control terminal color in human output only. | All human-output commands | D29, D83 |
| `--verbose` | Print resolved plan facts before execution. | Project commands | D26, D83 |
| `--quiet` | Suppress non-error presentation output. | Project commands | D29 |
| `--out-dir <path>` | Select the user-visible output root for this command. | build, run, test, bench, doc, audit, semver-check | D26, D83, D264 |
| `--cache-dir <path>` | Select the disposable toolchain cache root for this command. | Project commands | D83, D144, D264 |

> Trace: D26, D29, D31, D78, D80, D83, D149, D264
> Covers: Common CLI flags have fixed meanings and do not create hidden command-specific policy.

`--verbose` must print the selected manifest, command scope, selected packages, selected target, selected backend, selected profile, output root, cache root, lockfile path, package cache path, target-spec files, resolved backend tools, and explicit extra flags. It must not print secrets from the environment. If a configured path is sensitive, the tool may redact user-home prefixes in presentation output, but not in deterministic artifact identity.

> Trace: D26, D29, D83, D149
> Covers: Verbose output exposes the build plan without leaking unrelated environment state.

## Exit Status

| Status | Meaning | Trace |
| ---: | --- | --- |
| `0` | The command completed successfully. | D26 |
| `1` | The command found ordinary user-facing errors: parse errors, type errors, test failures, audit policy failures, missing dependencies, invalid manifest, or SemVer break reports when configured as fatal. | D26, D29 |
| `2` | The CLI invocation is malformed: unknown flag, missing flag value, incompatible flags, or unknown subcommand. | D26 |
| `3` | The toolchain itself failed internally, including compiler bugs, corrupted caches the tool cannot recover from, or malformed toolchain-owned artifacts. | D26, D84 |
| `4` | Required external tool or target support is missing for the selected backend/target. | D31, D80, D149 |

> Trace: D26, D29, D80, D84, D149
> Covers: CLI exit behavior is stable enough for scripts and CI.

A command that exits nonzero must emit at least one diagnostic or structured error record explaining the reason, unless the process is killed externally before it can report. Human output may be localized in a future edition only if JSON diagnostic keys remain stable and the language of diagnostic codes remains unchanged.

> Trace: D29, D225
> Covers: Failed commands explain themselves and keep machine-readable diagnostics stable.

## Core Commands

| Command | Required Behavior | Trace |
| --- | --- | --- |
| `kyokai --version` | Print the toolchain version, language-edition support, host triple, default backend, and diagnostic schema version without requiring a project. | D26, D225, D268 |
| `kyokai doctor` | Inspect the host toolchain, supported targets, configured C/LLVM tools, cache/output roots, release provenance, and common setup problems without reading source as language input. | D31, D80, D149, D225, D268 |
| `kyokai init` | Create a package manifest in the current directory, write explicit layout information, and refuse to overwrite an existing package/workspace unless an explicit force flag is passed. | D26, D78, D266 |
| `kyokai new` | Create a new package or workspace directory from an official template, including `kyokai.toml`, module roots, initial `.kyo`/`.kai` files, and optional test/doc skeletons. | D26, D78, D266 |
| `kyokai check` | Parse, resolve modules/imports, validate `.koi`, typecheck, check contracts syntactically/semantically, resolve instances, check linearity, borrow rules, capabilities, unsafe contracts, and target guards. It may skip final code generation and linking. | D26, D29, D79 |
| `kyokai build` | Perform `check`, generate backend artifacts, compile/link as required by package output type, and emit requested build products. | D26, D31, D80, D139 |
| `kyokai run` | Build one executable package target and execute it through the selected runner; `--` separates program arguments. | D26, D80 |
| `kyokai test` | Build and run inline tests and optionally doc tests using ordinary semantics and explicit capability policy. | D28, D137, D218 |
| `kyokai bench` | Build and run benchmark declarations or bench-marked tests under the selected profile and target runner. | D28, D137 |
| `kyokai fmt` | Format source deterministically and idempotently without configuration knobs. | D25 |
| `kyokai doc` | Generate public-interface documentation and JSON from checked interfaces and contracts. | D17, D218 |
| `kyokai lsp` | Run the official language server using the same compiler engine. | D148 |
| `kyokai audit` | Report dependency, unsafe, FFI, capability, generation, and public-surface risk facts. | D150 |
| `kyokai explain` | Print detailed documentation for a diagnostic code, warning category, lint category, audit category, or command exit status. | D29, D267 |
| `kyokai fix` | Apply selected machine-applicable suggestions after checking that every edit still parses, formats, and preserves the diagnostic's stated repair semantics. | D25, D29, D267 |
| `kyokai repl` | Start a persistent interactive session using ordinary compiler semantics. | D151-D151a |
| `kyokai eval` | Compile and run a one-shot expression, statement block, or file fragment using ordinary compiler semantics. | D151 |
| `kyokai add` | Add a workspace or pinned Git dependency and update the lockfile. | D51 |
| `kyokai remove` | Remove a dependency entry, update the lockfile, and report any packages that still require the removed dependency. | D51, D83, D269 |
| `kyokai update` | Re-resolve selected dependencies within explicit pins/policies and update the lockfile. | D51, D83, D244 |
| `kyokai search` | Query the configured discovery index and print package metadata without changing manifests, lockfiles, or caches except for ordinary index cache state. | D221, D269 |
| `kyokai info` | Print package, version, source revision, license, docs, yanked/advisory, public interface, and audit metadata for a dependency or index result. | D51, D150, D221, D269 |
| `kyokai tree` | Print the resolved dependency graph from the lockfile or selected manifest resolution in deterministic order. | D51, D83, D269 |
| `kyokai why` | Explain why a package appears in the dependency graph by printing one or more dependency paths from selected roots to that package. | D51, D269 |
| `kyokai outdated` | Compare pinned dependencies against explicit update policy and index metadata, reporting available newer revisions, yanks, and advisories without editing files. | D51, D221, D244, D269 |
| `kyokai vendor` | Materialize pinned dependency sources into an explicit vendor directory and rewrite or record resolution metadata so offline builds use the same revisions. | D51, D83, D269 |
| `kyokai publish` | Validate package metadata and release policy, then prepare or submit package discovery metadata. | D221, D223, D244 |
| `kyokai semver-check` | Compare public interface surfaces and report source/API compatibility changes. | D223 |
| `kyokai clean` | Remove selected cache state by default, remove selected output artifacts with `--outputs`, and remove both selected output/cache roots with `--all` without deleting source, manifests, lockfiles, or package index/cache state outside the selected cache root. | D83, D144, D264 |
| `kyokai koi verify` | Validate `.koi` container structure, sections, hashes, compatibility fields, and canonical ordering. | D79, D265 |
| `kyokai koi print` | Print a derived JSON or text view of a `.koi` artifact without making the derived view authoritative. | D79, D265 |
| `kyokai koi diff` | Compare two `.koi` artifacts and classify public API, internal API, contract, generic metadata, target, and hash changes. | D79, D223, D265 |

> Trace: D25-D29, D51, D79, D83, D137, D148-D151a, D218, D221, D223-D224, D244, D264-D270
> Covers: Required CLI commands and their top-level obligations are specified, including project creation, diagnostics explanation, safe fixes, toolchain health, and package inspection commands.

## Project Creation Commands

`kyokai init` operates on the current directory. It writes `kyokai.toml`, creates the selected module root, and creates initial interface/body files only when those paths do not already exist. The default package template writes `[package]`, `version`, `edition`, and `[layout].module_root = "src"`. The default workspace template writes `[workspace].members = []` and does not invent packages unless the user asks for a package member template.

> Trace: D26, D78, D266
> Covers: Project initialization is explicit, non-destructive by default, and writes the required manifest layout instead of relying on inferred source roots.

`kyokai new <path>` creates a new directory and then follows the same manifest and template rules as `init`. Official templates are `package`, `workspace`, `library`, `executable`, and `empty`. A template may add tests, documentation examples, or CI files only when the command line says so or the selected template's documented contract says so. Template expansion must be deterministic for the same toolchain version and flags.

> Trace: D26, D78, D83, D266
> Covers: New-project scaffolding is deterministic, template-driven, and does not hide generated project shape.

## Check Is A Real Compiler Pass

`kyokai check` is not allowed to be a parser-only command. It must run the same resolver, type checker, contract checker, linearity checker, capability checker, target guard evaluator, instance resolver, and `.koi` compatibility checker that `build` uses. It may skip backend lowering, C compilation, LLVM emission, assembly, archiving, linking, and final executable runner checks.

> Trace: D26, D29, D79, D86
> Covers: Fast checking is semantically honest but may omit backend and link phases.

If `check` succeeds and `build` later fails, the failure must belong to a phase `check` is allowed to skip, such as backend availability, backend-specific lowering bug, target toolchain rejection, link failure, missing native library, runner failure, or code-size/linker limit. If a later build finds a source semantic error that `check` should have found, the toolchain is non-conforming.

> Trace: D26, D31, D80, D139, D149
> Covers: `check` success narrows later failure causes to backend, link, target, and runner phases.

## Package Commands

`kyokai add --workspace <name>` writes a workspace dependency entry. `kyokai add --git <url> --rev <rev>` writes a pinned Git dependency. `kyokai add --git <url> --tag <tag>` resolves the tag to a commit and writes both `tag` and `rev`. A command that cannot determine an immutable revision must fail without editing the manifest.

> Trace: D51, D83
> Covers: Adding dependencies always writes immutable resolution information.

`kyokai update` may update Git revisions only when the selected dependency source and policy allow it. It must rewrite the lockfile deterministically and report the old revision, new revision, package identity, package version, and whether the new revision is yanked, superseded, or advisory-affected when such metadata is available.

> Trace: D51, D83, D221, D244
> Covers: Dependency updates are explicit lockfile changes with visible revision movement.

`kyokai publish` does not make the package index the source of code truth. Publishing records discovery metadata and release metadata; the immutable source remains the declared repository and revision model. A publish command must reject package metadata whose manifest identity, source revision, version, and `.koi` interface summary do not agree.

> Trace: D51, D221, D223, D244
> Covers: Publishing supports discovery without replacing pinned source identity.

`kyokai remove <name>` edits the manifest only when the named dependency is directly declared in the selected package. It must update the lockfile or report why the dependency remains reachable through another package. The command must not delete source code or vendor directories unless a separate explicit flag names that cleanup.

> Trace: D51, D83, D269
> Covers: Dependency removal is manifest-scoped and does not pretend transitive dependency cleanup is source cleanup.

`kyokai search`, `kyokai info`, `kyokai tree`, `kyokai why`, and `kyokai outdated` are read-only inspection commands unless a command-specific update flag is explicitly added later. They may refresh package-index metadata in the tool cache, but they must not edit `kyokai.toml`, `kyokai.lock`, source files, or output artifacts.

> Trace: D51, D83, D221, D244, D269
> Covers: Package discovery and graph inspection are safe daily commands with clear filesystem effects.

`kyokai vendor` writes only into the selected vendor directory and records enough metadata to prove that vendored sources correspond to the same immutable revisions in the lockfile. Offline builds may use vendored sources only when the manifest, lockfile, and vendor metadata agree. A vendored source tree is not a registry and does not change package identity.

> Trace: D51, D83, D269
> Covers: Offline and vendored workflows preserve the same pinned dependency identity as online resolution.

## Explanation And Fix Commands

`kyokai explain <code-or-category>` reads the versioned diagnostic explanation catalog shipped with the toolchain. The output must include the diagnostic code or category, severity, short meaning, longer explanation, common causes, at least one repair pattern when a repair is known, and links or local anchors to the relevant spec chapter when available.

> Trace: D29, D267
> Covers: Diagnostic codes have a first-party explanation path instead of forcing users to search compiler source or web pages.

`kyokai fix` is separate from `kyokai fmt`. Formatting changes layout only; fixing applies compiler-suggested semantic edits. By default `fix` applies only machine-applicable suggestions for diagnostics already emitted by `check`, refuses overlapping edits, writes a deterministic summary, and reruns parsing plus formatting validation before committing file changes. Suggestions marked `maybe-incorrect` or `manual-only` require an explicit flag and still must not be applied silently.

> Trace: D25, D29, D267
> Covers: Automatic fixes are opt-in, suggestion-class aware, and validated through the compiler instead of being hidden formatter behavior.

## Toolchain Health Commands

`kyokai --version` and `kyokai doctor` do not require a project. `--version` prints the public identity of the installed toolchain. `doctor` checks host support, external compiler/linker discovery, selected default backend, supported target triples, release provenance, cache writability, configured index access, and common environment problems. It reports findings as ordinary diagnostics and must not modify project files.

> Trace: D31, D80, D149, D225, D268
> Covers: Toolchain identity and host setup problems are inspectable without making a dummy project or reading source files.

## Output And Cache Roots

A standalone package owns its default output and cache roots at the package root. A workspace owns its default output and cache roots at the workspace root. Member packages in a workspace do not create independent default output/cache roots for workspace builds.

> Trace: D78, D83, D264
> Covers: Output/cache ownership follows the same package/workspace owner as lockfiles and build scope.

The default user-visible output root is `<owner-root>/kyokai-out/`. The default disposable toolchain cache root is `<owner-root>/.kyokai-cache/`. `--out-dir <path>` and `--cache-dir <path>` override those roots for the current command. Relative override paths are interpreted from the current working directory.

> Trace: D26, D83, D144, D264
> Covers: Kyokai has explicit default and override roots for build products and tool-owned cache state.

`kyokai clean` removes cache state by default. `kyokai clean --outputs` may remove output artifacts. `kyokai clean --all` removes both selected output and cache roots. It must not remove source files, `kyokai.toml`, `kyokai.lock`, package index metadata outside the selected cache root, or user-selected paths outside the owner root unless those paths were explicitly passed as `--out-dir` or `--cache-dir`.

> Trace: D26, D83, D144, D264
> Covers: Clean behavior is bounded and cannot delete source or unrelated package state by accident.

`kyokai run` executes the selected executable from the output tree unless a target runner requires staging. `kyokai test` and `kyokai bench` may place harness executables and private runner state in the cache tree, but user-requested reports go under the output tree's `reports/` directory or to stdout when requested.

> Trace: D26, D28, D83, D137, D264
> Covers: Run, test, and bench distinguish user-visible reports from private harness/cache state.

## Standard Streams

Human diagnostics and progress output go to stderr. Command results intended for another program, such as JSON reports or `kyokai eval` values requested in machine mode, go to stdout. `kyokai run` reserves stdout and stderr after launch for the child program, except for runner setup failures reported before execution.

> Trace: D26, D29, D225
> Covers: CLI output streams are stable for scripts and CI.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
The command line is a pact. If `kyokai check` says yes, it should mean a real yes for the parts it claims to check. If `kyokai add` writes a dependency, it should not smuggle a moving branch into tomorrow's build. This is the difference between a tool that helps you and a tool that leaves a note after the damage is already done.

> Trace: D26, D51, D83
> Covers: Kyokai CLI commands are explicit enough to make automation and trust possible.
