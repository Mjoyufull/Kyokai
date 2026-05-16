# Build Profiles, Targets, And Linking

[Rikona Kurasaki / Mjoyufull]
A build profile is not a mood. A target is not a wish. A backend is not a secret second language. Kyokai writes these choices down because optimization, debug info, link mode, target ABI, and backend tool contracts can change what a programmer receives from the toolchain.

> Trace: D27, D31, D80, D83, D139, D149, D200
> Covers: Build profiles, targets, backends, and linking are explicit toolchain contracts.

## Profiles

Profiles are named manifest tables under `[profile.<name>]`. The standardized profile names are `debug`, `test`, `release`, and `bench`. Custom profiles are allowed when their fields are explicit or inherited from another named profile.

> Trace: D26, D31
> Covers: Build profiles are named manifest configuration, with conventional standard names and explicit custom inheritance.

A profile may contain these fields:

| Field | Type | Required Meaning | Trace |
| --- | --- | --- | --- |
| `inherits` | string | Copy unspecified fields from another profile. Cycles are illegal. | D31 |
| `optimization` | integer `0`, `1`, `2`, or `3` | Backend optimization level. It must not disable safety checks. | D31, D73 |
| `debug_info` | boolean | Emit source/debug metadata for supported targets. | D27 |
| `strip` | boolean | Remove symbols/debug sections only after required diagnostics/provenance artifacts are produced. | D27, D225 |
| `lto` | boolean | Enable link-time optimization when supported by the selected backend/target. | D31 |
| `identical_code_folding` | boolean | Permit profile-controlled folding of proven identical generated code. | D200 |
| `panic_backtrace` | boolean | Include runtime backtrace metadata for `panic`/fatal reporting when the target supports it. | D84 |
| `path_remap` | array of strings | Map absolute source prefixes for deterministic debug info. | D27, D83 |

> Trace: D27, D31, D73, D83, D84, D200, D225
> Covers: Profile fields are typed, bounded, and visible.

`optimization` must never remove runtime checks required by the language spec. Bounds checks, integer traps where checked arithmetic applies, contract checks, borrow/linearity consequences already enforced at compile time, panic paths, TPOE paths, stack probes required by the target contract, and safe concurrency checks remain semantically present in all profiles unless a future unsafe-only feature explicitly defines a separate source-visible opt-out.

> Trace: D53, D73, D75-D76, D84, D139, D262
> Covers: Release optimization cannot erase Kyokai safety semantics.

The default profile for `check`, `build`, `run`, and `doc` is `debug`. The default profile for `test` is `test` if present, otherwise `debug`. The default profile for `bench` is `bench` if present, otherwise `release`. `--release` is exactly `--profile release`.

> Trace: D26, D31
> Covers: Profile defaults are fixed and command-visible.

## Target Triples And Support Tiers

A target triple is spelled `arch-os-abi`. A target triple is legal only if it appears in Kyokai's target matrix or an imported target-spec file explicitly admitted by the toolchain version. The existence of `target.arch`, `target.os`, or `target.abi` enum values does not create a legal target triple by cartesian product.

> Trace: D19, D80, D149
> Covers: Target triples are closed, explicit support promises, not inferred enum combinations.

A target support entry records the triple, tier, supported backends, runner availability, C compiler contract when using the C backend, LLVM version/feature contract when using the LLVM backend, atomic support, stack-overflow detection strategy, object format, dynamic-link support, and known unsupported standard-library families.

> Trace: D31, D80, D139, D141, D149, D262
> Covers: Target support declares exactly what the toolchain promises for each target.

| Tier | Promise | Required Behavior | Trace |
| --- | --- | --- | --- |
| Tier 1 | Release-blocking supported target. | CI builds compiler, stdlib, conformance tests, package tools, and release artifacts for the target. | D80, D225 |
| Tier 2 | Supported but not fully release-blocking. | Compiler and stdlib build regularly; gaps are documented by target support entry. | D80 |
| Experimental | Admitted for development. | May be incomplete, but the tool must report unsupported features explicitly and must not silently lower safe Kyokai through UB. | D73, D80, D139 |

> Trace: D73, D80, D139, D225
> Covers: Target tiers are user-visible promises with defined missing-feature behavior.

An unsupported target triple is a front-end configuration error. A supported triple with an unsupported selected backend is a target/backend configuration error. The tool must not silently select the host target, another ABI, another backend, or another linker.

> Trace: D26, D80, D149
> Covers: Target/backend mismatch fails early rather than falling back silently.

## Target Configuration

Target configuration lives in `[target.<triple>]` and backend-specific subtables. The key `spec = "path/to/target.toml"` imports a reusable target-spec TOML file. The importing table may override fields only where the target-spec schema admits override.

> Trace: D31, D80, D149
> Covers: Cross-compilation uses manifest-centered target tables and importable target-spec files.

A target-spec file is part of build identity. Its bytes, normalized path identity, declared schema version, and resolved imported files must be included in reproducibility fingerprints. A missing or unreadable target-spec file is a build configuration error.

> Trace: D83, D149
> Covers: Target-spec files are deterministic build inputs.

Backend-specific configuration uses standardized backend names: `c` and `llvm`. A conforming toolchain may omit a backend for a target, but if the backend is present its configuration must state the external tools and version floors needed to preserve Kyokai semantics.

> Trace: D31, D80, D139, D149
> Covers: Backend availability is explicit per target.

## Backend Contracts

The C backend emits C only inside Kyokai's supported C dialect and compiler contract. Generated C must preserve Kyokai evaluation order, checked failures, layout, atomics, volatile operations, stack checks, and source mapping without relying on C undefined behavior for safe Kyokai operations.

> Trace: D27, D73, D94, D139, D141, D228, D257
> Covers: The C backend is constrained by Kyokai semantics and cannot outsource safety to C UB.

The LLVM backend, when present, lowers directly to LLVM IR or an equivalent LLVM pipeline. It must preserve the same semantic rules as the C backend and must not introduce LLVM poison, undef, invalid aliasing metadata, misdeclared alignment, or invalid lifetime markers for safe Kyokai operations.

> Trace: D73, D139, D228
> Covers: LLVM lowering is bound by the same no-backend-UB rule.

If a backend cannot represent a Kyokai operation on a selected target, the build fails with a target/backend diagnostic. The toolchain must not silently use a weaker operation, scalarize a vector intrinsic that is specified as target-specific, ignore volatile semantics, replace atomics with non-atomic accesses, or erase stack-overflow detection.

> Trace: D73, D80, D94, D104, D141, D257, D262
> Covers: Backend limitations are diagnostics, not semantic substitutions.

## Package Output Types

Package output settings live in `[build]` in the package manifest. `output_type` is one of `executable`, `static-lib`, or `dynamic-lib`. `backend` names the default backend. `link` is `target-default`, `static`, or `dynamic`.

> Trace: D26, D31, D80
> Covers: Package output kind and default backend are manifest-declared.

If `output_type = "executable"`, the package must define exactly one entrypoint selected by the language spec and target contract. If more than one runnable target is admitted by a future chapter, the manifest must name which one `run` uses by default.

> Trace: D26, D48, D80
> Covers: Executable packages have unambiguous entrypoint behavior.

If `output_type = "static-lib"` or `dynamic-lib`, exported symbols must come only from declarations explicitly admitted for export by the language/FFI spec. Ordinary Kyokai public declarations are source-interface public; they are not automatically C or platform ABI exports.

> Trace: D17, D20, D31, D139
> Covers: Source visibility and binary export visibility are separate.

## Build Output Tree

User-visible build artifacts are written under `<out-root>/<target-triple>/<profile>/<backend>/<package-name>/`. The backend component is required for backend-produced artifacts. A backend-independent command may omit the backend component only when backend selection cannot affect the artifact identity.

> Trace: D26, D31, D78, D80, D83, D149, D264
> Covers: Build outputs are partitioned by target, profile, backend, and package so cross-build artifacts cannot collide.

The standard output subdirectories are `bin/` for executables, `lib/` for static and dynamic libraries, `koi/` for checked package interface artifacts, `gen/` for declared generated source/backend files meant for inspection, `doc/` for generated HTML and documentation JSON, `reports/` for coverage/bench/audit/SemVer/timing/provenance reports, and `obj/` only for object files that the selected profile or flag marks as user-inspectable artifacts.

> Trace: D27, D31, D79, D83, D218, D223, D225, D264
> Covers: User-visible output subdirectories have fixed meanings, including `.koi`, generated files, docs, reports, and optional object output.

Private object files, backend scratch, temporary IR, dependency build scratch, fingerprints, and incremental query state belong in the cache root, not in the output tree, unless a profile or command explicitly asks to expose them as inspectable products.

> Trace: D83, D144, D264
> Covers: Disposable compiler machinery is separated from artifacts users may keep, inspect, or package.

## Linking

Linking uses the selected target's linker configuration, package output type, dependency graph, backend artifacts, native libraries declared by unsafe/FFI contracts, and target profile overrides. Link order must be deterministic.

> Trace: D20, D31, D80, D83, D149
> Covers: Linking has deterministic inputs and explicit native dependency sources.

A safe Kyokai package cannot gain native link dependencies by ambient discovery. Native libraries required by foreign blocks, runtime support, target helpers, or unsafe wrappers must be declared in the package manifest, target-spec file, or toolchain target contract.

> Trace: D20, D83, D150, D230
> Covers: Native dependencies are explicit and auditable.

Dynamic linking is allowed only when the selected target and package configuration admit it. If dynamic linking is selected, the runtime search path policy, install names, sonames, import libraries, and platform loader assumptions must be explicit target-toolchain fields or rejected.

> Trace: D31, D80, D83, D149
> Covers: Dynamic linking behavior is written configuration.

## Identical Code Folding

`identical_code_folding = true` permits the toolchain to merge generated code only when the compiler proves the folded bodies are semantically identical for all observable Kyokai behavior and the selected profile allows the optimization. Folding must not merge functions when doing so would change stack traces, exported symbol identity, address-sensitive unsafe contracts, debug requirements for the selected profile, or provenance obligations.

> Trace: D27, D83, D200
> Covers: Code-size optimization is explicit and constrained by observable semantics.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
C taught systems programmers to ask what the target really is because the machine always answers eventually. Kyokai asks earlier. The profile says what it optimizes. The target says what it supports. The backend says what tools carry it. When the answer is no, the compiler says no before the linker leaves you staring at smoke.

> Trace: D31, D80, D83, D139, D149
> Covers: Explicit target and build policy keeps systems programming practical without returning to backend folklore.
