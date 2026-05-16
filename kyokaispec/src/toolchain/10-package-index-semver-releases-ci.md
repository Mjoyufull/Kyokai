# Package Index, SemVer, Releases, And CI

[Rikona Kurasaki / Mjoyufull]
An ecosystem needs discoverability, but Kyokai does not need a central altar where source truth goes to be renamed. Packages are discovered through an index, resolved through pinned source, checked through interfaces, and released with provenance people can actually inspect.

> Trace: D51, D221, D223-D225, D244
> Covers: Package discovery, versioning, yanks, release artifacts, and CI are explicit ecosystem contracts.

## Package Discovery Index

The official package index is a read-only discovery index. It records package names, versions, source repository URLs, immutable source revisions, checksums where available, yanked status, advisory metadata, documentation links, license metadata, and `.koi` interface summaries where produced. It is not the canonical source store and does not replace Git `rev` pinning.

> Trace: D51, D221, D223, D244
> Covers: The package index helps users find packages without becoming the source authority.

A dependency resolved through the index still writes an immutable source revision into the lockfile. A manifest may record version intent, but the lockfile records the exact revision used for the build.

> Trace: D51, D83, D221, D223
> Covers: Index discovery does not weaken lockfile reproducibility.

Index metadata is append-only for released versions except for yanks, advisories, documentation links, and explicitly versioned metadata corrections. A metadata correction must not change the source revision or package identity of an existing version.

> Trace: D221, D244
> Covers: Published version identity is stable while safety metadata can grow.

## Yanks

A yank marks a package version or source revision as unavailable for new resolution. Existing lockfiles that already name the yanked revision continue to build unless audit policy, advisory policy, or an explicit security block rejects them.

> Trace: D83, D244
> Covers: Yanks affect new selection without breaking existing lockfile meaning.

A yank record includes package name, version, source revision, reason category, timestamp authority of the index, and optional advisory link. The reason category is one of `security`, `soundness`, `licensing`, `accidental-publish`, `replaced`, or `other`.

> Trace: D221, D244
> Covers: Yank metadata is inspectable and categorized.

## Package Inspection And Offline Workflows

`kyokai search` queries configured package discovery indexes and prints package metadata. `kyokai info` prints one package or dependency's identity, source revision, version, license, docs link, yanked/advisory state, public interface summary, and audit summary when those facts are known. Both commands are read-only with respect to manifests and lockfiles.

> Trace: D51, D150, D221, D269
> Covers: Package discovery and package fact lookup are daily read-only operations.

`kyokai tree` prints the selected dependency graph in deterministic order. `kyokai why <package>` prints dependency paths explaining why a package is present. `kyokai outdated` compares lockfile revisions to configured update policy and index metadata, reporting newer versions, newer revisions, yanks, and advisories without editing files.

> Trace: D51, D83, D221, D244, D269
> Covers: Dependency graph explanation and update visibility are available without changing the build.

`kyokai vendor` materializes exact locked dependency sources into an explicit vendor directory. Vendor metadata records package identity, source URL, revision, checksums where available, and lockfile identity. Offline builds may use vendored sources only when that metadata agrees with the lockfile; a vendor directory is never a hidden package registry.

> Trace: D51, D83, D269
> Covers: Offline use keeps pinned dependency identity and does not create a second dependency authority.

## SemVer Convention

The `version` field follows SemVer as an ecosystem convention for package API intent. It does not replace immutable revision pins and it does not define language editions. The source revision and lockfile remain the reproducibility mechanism.

> Trace: D105, D223, D243
> Covers: SemVer guides package evolution without becoming edition or resolution identity.

`kyokai semver-check` compares public `.kyo` interface surfaces and `.koi` API metadata. It reports removed declarations, changed signatures, changed type visibility, changed contracts, changed capability requirements, changed failure behavior where represented in contracts/docs, changed exported ABI surfaces, deprecated declarations, and added declarations.

> Trace: D17, D20, D53, D79, D218, D223
> Covers: SemVer checking observes public interface and contract changes.

The checker is advisory by default. CI or manifest policy may promote SemVer findings to errors. A SemVer pass does not prove behavioral compatibility beyond the surfaces the checker actually compares, and the report must say which comparison domains were included.

> Trace: D29, D223, D225
> Covers: SemVer checks are useful but honest about their proof boundary.

## Release Cadence

Early Kyokai releases ship when the project has something coherent to release. Once the toolchain reaches steady public use, ordinary toolchain releases default to a four-week train. Language editions remain rare, demand-driven, and separate from the ordinary release train.

> Trace: D105, D157, D243
> Covers: Toolchain release cadence and language editions are separate clocks.

A release note must classify changes as bug fix, toolchain behavior change, diagnostic change, package ecosystem change, stdlib API change, backend/target support change, SemVer-relevant API change, or edition/source-semantics change.

> Trace: D105, D157, D223, D243
> Covers: Releases explain compatibility impact explicitly.

## Official Artifacts

Official releases provide source archives, compiler/toolchain binaries for supported hosts, checksums, provenance records, signatures when signing infrastructure exists, setup action metadata, OCI images, and target support notes. Runtime, stdlib, startup, compiler support, and target helper license boundaries follow the project licensing decision; spec documentation keeps its existing documentation license.

> Trace: D225, D263
> Covers: Release artifacts and licensing boundaries are visible.

Checksums cover every distributed archive and binary. Provenance records name source revision, toolchain version, build identity, target, profile, backend, lockfile hash, artifact hash, and builder identity class. If a release artifact is rebuilt, the new artifact must either match or carry a new provenance record explaining the changed identity.

> Trace: D83, D225
> Covers: Release artifacts are verifiable and provenance-backed.

## CI Installation Contract

The official CI contract provides one portable installation path for supported CI systems. `setup-kyokai` or the equivalent official action installs a requested toolchain version, verifies checksums, exposes `kyokai` on `PATH`, reports the installed version, and optionally primes package caches without changing project lockfiles.

> Trace: D225
> Covers: CI installs are standard, verified, and lockfile-preserving.

Official OCI images include the toolchain, target support declared by the image tag, checksums/provenance labels, and documented default user/environment behavior. Image tags that move, such as `latest`, are convenience labels and must not be used as reproducible release identity.

> Trace: D83, D225
> Covers: OCI images support CI while preserving immutable release identity.

## Local Toolchain Health

The installed toolchain reports its identity through `kyokai --version`: toolchain version, source revision or release id, supported language editions, diagnostic schema version, host triple, default backend, and KBI compatibility range. This command does not need a project and must not read source files.

> Trace: D105, D225, D265, D268
> Covers: Users and CI can inspect the installed compiler identity without constructing a project.

`kyokai doctor` checks local setup: release provenance, checksum/signature status where available, host support, configured target tools, C/LLVM backend discovery, cache and output writability, package index access, environment variables admitted by the toolchain spec, and common path or permission mistakes. It reports diagnostics and suggested repairs but does not modify project files.

> Trace: D31, D80, D149, D225, D268
> Covers: Host setup failures are diagnosable through a first-party command instead of surfacing as late build folklore.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
Discoverability is good. Blind trust is not. Kyokai's index is a map, not the land. The lockfile says where you stood. The `.koi` says what the package promised. The release artifact says who built what, from which source, for which target. That is enough ceremony to matter, and not so much that the ecosystem drowns in it.

> Trace: D51, D83, D221, D223-D225, D244
> Covers: Kyokai's ecosystem model combines discoverability with pinned, auditable, reproducible source identity.
