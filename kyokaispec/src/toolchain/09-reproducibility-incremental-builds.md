# Reproducibility And Incremental Builds

[Rikona Kurasaki / Mjoyufull]
Reproducibility is the build system telling the truth twice. Incremental compilation is the toolchain moving faster without changing that truth. Kyokai needs both: trust for releases, and speed for daily work.

> Trace: D83, D144
> Covers: Reproducible outputs and incremental compilation are both normative toolchain concerns.

## Reproducible Build Identity

Given the same build identity, a conforming toolchain must produce the same `.koi` artifacts, generated C where requested, object files when the backend/toolchain contract admits bit-identical objects, libraries, executables, documentation JSON, lockfile updates, and release provenance records. If a target external tool cannot make an output bit-identical, the target contract must state the exception and the reproducible artifact boundary that remains guaranteed.

> Trace: D27, D79, D83, D139, D149, D218, D225
> Covers: Reproducibility applies to specified artifacts, with target exceptions bounded by target contracts.

Build identity includes:

| Input | Included Facts | Trace |
| --- | --- | --- |
| Source | File bytes, logical module names, source origins, generated-source declarations, and doc-test extraction inputs. | D52, D78, D218 |
| Project | Package/workspace manifests, lockfile, package identities, versions, editions, module roots, build tables, profiles, and diagnostic policy. | D51, D78, D83, D105 |
| Dependencies | Pinned Git revisions, resolved package identities, `.koi` compatibility fields, yanked/advisory policy when fatal. | D51, D79, D244 |
| Target | Triple, support tier, target-spec bytes, backend selection, external tool versions, sysroot, linker, and admitted flags. | D31, D80, D149 |
| Compiler | Compiler version, `.koi` format version, language edition, backend compatibility class, diagnostic schema version. | D79, D83, D105 |
| Command | Flags that affect accepted source, generated artifacts, diagnostics as artifacts, profile, target, backend, output mode, output root, and cache root. | D26, D29, D31, D264 |
| Output paths | Output/cache roots only where artifact contents record paths; path remapping controls reproducible profiles. | D27, D83, D264 |

> Trace: D26, D29, D31, D51, D52, D78-D80, D83, D105, D149, D218, D244, D264
> Covers: Build identity is explicit and complete enough for reproducible artifacts.

Hidden host facts are excluded unless a chapter admits them. Excluded facts include current time, timezone, locale, process ID, random seed, current username, unrelated environment variables, shell aliases, host directory iteration order, and absolute build path after path-remapping policy.

> Trace: D83
> Covers: Ambient host state does not perturb reproducible builds.

## Paths And Debug Info

Source paths embedded in debug info, diagnostics-as-artifacts, docs JSON, generated C `#line` directives, source maps, and provenance records must use normalized logical paths unless the selected profile explicitly requests absolute paths. Path remapping happens before artifact hashing.

> Trace: D27, D83
> Covers: Path handling is deterministic and profile-controlled.

If absolute paths are requested, the artifact is still deterministic only for builds performed under the same remapped path identity. Release profiles should use remapped paths by default.

> Trace: D27, D83, D225
> Covers: Absolute debug paths are explicit and not the release default.

## Output And Cache Path Identity

The output tree path is not a source semantic input. It becomes a build-identity input only when artifact contents record paths, such as debug information, generated C line directives, source maps, docs JSON, provenance, or diagnostics-as-artifacts. In reproducible profiles, path remapping must make artifacts independent of the absolute checkout location unless the user explicitly opts into absolute path embedding.

> Trace: D27, D83, D264
> Covers: Output/cache paths affect reproducibility only through path-recording artifacts, and reproducible profiles remap paths by default.

The default output root `kyokai-out/` holds user-visible artifacts. The default cache root `.kyokai-cache/` holds disposable toolchain state. A cache entry must not be the only copy of a requested build product, and an output artifact must not be required as hidden compiler state unless it is also validated by ordinary artifact identity.

> Trace: D83, D144, D264
> Covers: User-visible artifacts and disposable compiler state remain separate even when a build reuses prior outputs.

## Package Cache

The package cache stores fetched Git dependencies by URL identity and immutable revision. A cache hit must verify the revision identity and package manifest hash before use. The cache must not satisfy a dependency from a branch name, tag name alone, mutable checkout, or unverified local directory unless that source kind is explicitly admitted by the manifest schema.

> Trace: D51, D83, D244
> Covers: Dependency cache reuse is pinned and verified.

A corrupted cache entry must be rejected and may be repaired by refetching from the declared immutable source. Repairing the cache must not edit manifests or lockfiles unless the command is an explicit dependency update.

> Trace: D51, D83
> Covers: Cache repair preserves dependency meaning.

## Incremental Compilation

Kyokai uses a hybrid incremental model: package-level artifacts are the public dependency boundary, module-level work may be reused inside a package, and query/fingerprint invalidation may be used inside the compiler. Incremental compilation is an optimization and must not change accepted programs, rejected programs, diagnostic meaning, artifact compatibility, or runtime behavior.

> Trace: D78, D79, D83, D144
> Covers: Incremental compilation is allowed under a package/module/query model but cannot change semantics.

Incremental cache keys include every build identity input that could affect the cached result. If the compiler cannot prove a cached result is valid, it must recompute. A stale cache hit that changes a result is a toolchain bug, not an accepted nondeterminism.

> Trace: D83, D144
> Covers: Incremental cache correctness is required.

A cache may store parsed syntax trees, name-resolution results, typechecking facts, borrow/linearity facts, target-guard evaluation, `.koi` readers, generated backend IR, generated C, private object files, documentation facts, diagnostics, and dependency build scratch. Each cached entry must record the compatibility class and input fingerprint needed to validate it. Cache state is stored under `<cache-root>/<toolchain-compat>/<target-triple>/<profile>/<backend>/<package-name>/` when those components apply.

> Trace: D29, D79, D83, D144, D218, D264
> Covers: Incremental caches may cover compiler and tool outputs only with explicit fingerprints and deterministic cache-root partitioning.

Deleting the incremental cache must not change command results except for timing, progress output, and cache-miss reporting. `kyokai clean` deletes cache state by default; `kyokai clean --outputs` and `kyokai clean --all` are the explicit output-removal modes defined by the CLI chapter.

> Trace: D83, D144, D264
> Covers: Incremental caches are performance artifacts, not semantic sources, and output deletion is explicit.

## Generated Sources

Generated sources participate in reproducibility through declared generator inputs, declared outputs, generator command identity, target/profile/edition inputs, and declared capabilities. Undeclared generated files under a module root are rejected unless the generation chapter admits their source origin.

> Trace: D78, D83, D150
> Covers: Generated source is a declared build input/output, not a hidden source side effect.

## Release Provenance

Release provenance records include build identity, source revision, toolchain version, target, profile, backend, lockfile hash, generated-source hashes, `.koi` hashes, artifact checksums, and signing/checksum metadata when produced. Provenance itself is an artifact and must be reproducible except for explicit signature bytes and timestamp authority fields named by the release chapter.

> Trace: D83, D225
> Covers: Release provenance names its deterministic and authority-backed fields.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
A build that changes because the clock ticked is not a build you can swear by. A cache that changes meaning is not a cache, it is a trap with a progress bar. Kyokai keeps the fast path, but it makes the fast path prove it is still walking on the same road.

> Trace: D83, D144
> Covers: Kyokai treats reproducibility as trust and incremental compilation as a checked optimization.
