# Module Resolution And Koi Artifacts

[Rikona Kurasaki / Mjoyufull]
Module resolution is where the written package shape becomes a language graph. By the time the type checker sees imports, the toolchain must have already answered the practical questions: which package is this, which module root is in force, which source files belong to the package, which target-specific declarations exist, and which dependency interfaces are trusted inputs.

> Trace: D19a, D52, D78, D79, D105
> Covers: The toolchain resolves package scope, module roots, source files, target selection, editions, and dependency artifacts before language checking consumes a single module graph.

This chapter specifies the toolchain side of the module boundary. The language chapter specifies what modules, imports, visibility, and name lookup mean once the graph exists. The two chapters meet at the resolved module graph and the checked interface artifact.

> Trace: D78, D79, D86, D155
> Covers: Toolchain module discovery and language name resolution are separate normative surfaces joined by explicit artifacts and graph construction.

## Logical Module Mapping

Within a package, a logical module name maps to files under `[layout].module_root`. A dot in a module name is always a directory separator. If `module_root = "src"`, then `Foo` maps to `src/Foo.kyo` and `src/Foo.kai`, while `Foo.Bar` maps to `src/Foo/Bar.kyo` and `src/Foo/Bar.kai`.

> Trace: D52, D78
> Covers: Kyokai maps dotted module names to `.kyo` and `.kai` file pairs under the explicit package module root.

There is no alternate dotted filename form such as `src/Foo.Bar.kyo`. There is no `mod.kyo` convention. There is no include path search. There is no fallback from one package's module root into another directory. One logical module path has one source spelling inside its package.

> Trace: D52, D78
> Covers: Kyokai rejects alternate module-file spellings, include-path search, and implicit fallback roots.

Prefix modules may coexist. `Foo` and `Foo.Bar` are distinct logical modules because directory segments are path segments, not implicit nested module declarations. Their coexistence is legal when both file pairs exist and both declarations match their logical module names.

> Trace: D78
> Covers: Prefix module names may coexist without creating nested visibility or resolution ambiguity.

A package may not contain two filesystem paths that resolve to the same logical module after canonical path normalization for the host and target rules. Case-insensitive filesystem collisions, symlink-equivalent duplicates, alternate separators, and generated-source overlays must be rejected or normalized before they can create two sources for one module.

> Trace: D78, D83, D155
> Covers: Duplicate logical modules are errors, and filesystem normalization cannot create ambiguous module identity.

## Source Discovery

For each package, the toolchain discovers source modules only under the declared module root after manifest validation. It must ignore files outside the module root unless another explicit toolchain rule names them as generated inputs, build scripts, tests, or non-source assets.

> Trace: D78, D83
> Covers: Source module discovery is confined to the package's explicit module root.

A discovered `.kyo` file is an interface source. A discovered `.kai` file is a body source. A source file with a Kyokai extension that does not match the required module declaration for its resolved logical path is rejected. A file with an old Austral extension such as `.aui` or `.aum` is not a Kyokai source file.

> Trace: D52, D78
> Covers: `.kyo` and `.kai` are the only Kyokai source extensions, and source declarations must match resolved module identity.

A complete source module for ordinary build checking has one selected interface and one selected body. If a package depends on a module from another package, the downstream checker consumes that dependency through the dependency package's `.koi` interface artifact rather than reading the dependency's private bodies. Within the same package, body source is available for implementation checking but does not become importable interface surface.

> Trace: D17, D52, D78, D79
> Covers: Ordinary source modules pair interface and body files, dependency checking consumes `.koi` interfaces, and same-package bodies remain implementation inputs rather than import surfaces.

## Target And Edition Selection

The package edition is read from `[package].edition` before parsing source files. The parser, resolver, formatter, diagnostics, and documentation generator must interpret each source file according to the package's declared edition.

> Trace: D105, D243
> Covers: Source parsing and source-facing tools are edition-aware and use the package manifest edition.

Whole-file target selection happens before module-graph construction. If a package supplies platform-specific bodies for a module, the build target selects exactly one body for the logical module before imports are resolved. The selected body must still declare the same module name as the interface.

> Trace: D19, D19a, D78
> Covers: Target-specific body selection happens before module graph construction and must still preserve module identity.

Declaration-level `when` guards are evaluated during source checking for the selected target. A false guard makes the declaration semantically absent from the current build. In a shared selected module, overlapping active definitions for the same declaration and zero active definitions where one is required are compile-time errors.

> Trace: D19, D19a, D123
> Covers: Declaration-level target guards remove inactive declarations and reject overlapping or missing active definitions for the selected build target.

The language does not have body-level target branching. A target-specific implementation difference must be expressed through whole-file selection, declaration-level `when` guards, or typeclass abstraction.

> Trace: D19, D123
> Covers: Kyokai rejects body-level inline target branching and keeps platform variation at module, declaration, or abstraction boundaries.

## Import Graph Construction

The toolchain parses interface imports to construct the package module graph. Imports are file-scope only and may target modules in the same package or modules exported by dependency packages. A module import that cannot be resolved to exactly one visible interface is a compile-time error.

> Trace: D78, D79, D179, D214
> Covers: Import graph construction uses file-scope imports and requires each imported module to resolve to one visible interface.

Same-package imports may see public and internal declarations from the imported module's interface. Cross-package imports may see only public declarations recorded in the dependency interface artifact. Private body declarations are never candidates for import graph construction.

> Trace: D17, D78, D79
> Covers: Import graph visibility follows package boundaries, and private declarations never enter the import graph.

The import graph must be deterministic. The same manifest, lockfile, target, edition, source content, generated-source inputs, and compiler compatibility class must produce the same resolved module graph or the same diagnostics. Host directory iteration order, filesystem case behavior, and import declaration order must not change the selected graph.

> Trace: D78, D83, D214
> Covers: Module graph construction is deterministic and reproducible, independent of host iteration order and import order.

Cyclic imports through interfaces are illegal unless a later chapter defines an explicit cycle protocol. A body may call functions from another body only through declarations visible in interfaces or same-module private declarations; direct private-body cycles across modules are not a module-system feature.

> Trace: D78, D79, D155
> Covers: Interface import cycles are rejected under the current module model, and cross-module private body coupling is not a supported import mechanism.

## Koi Artifact Role

A `.koi` artifact is the checked interface product of a package. It is not an incremental cache blob and not an implementation detail. Downstream checking, documentation, auditing, separate compilation, and reproducible builds may depend on its specified contents and compatibility fields.

> Trace: D79, D83, D86
> Covers: `.koi` is a versioned package interface contract artifact, not an opaque cache.

A `.koi` artifact records at least the producing compiler version, language edition, `.koi` format version, target contract, package identity, package version, package module set, hashes or fingerprints of interface inputs, visibility-marked declarations, type definitions at their visible opacity level, typeclass definitions, legal instances, generic metadata needed for downstream checking and materialization, and any compatibility fields required by the backend and generics chapters.

> Trace: D79, D82a, D82b, D83, D105
> Covers: `.koi` artifacts record identity, compatibility, interface declarations, visible type/typeclass/instance data, hashes, and generic metadata needed by downstream compilation.

A `.koi` artifact may record internal declarations and internal instances because same-package incremental checking and documentation may need them. A downstream package outside the producing package must treat internal entries as nonexistent. Private `.kai` declarations that are not part of the package interface never appear in `.koi`.

> Trace: D17, D79
> Covers: `.koi` preserves internal metadata for same-package use while excluding private body declarations and hiding internals from external consumers.

A `.koi` artifact records enough information for dependency consumers to typecheck against public declarations without reading dependency source bodies. It does not grant access to private implementation, unsafe internals, or hidden source files.

> Trace: D17, D79, D245
> Covers: Dependency consumers typecheck against interface artifacts, not private body source or unsafe implementation details.

## Koi Binary Interface

A `.koi` file uses the canonical Koi Binary Interface container, version `KBI-1`. The binary file is the artifact authority. Human-readable JSON or text produced from it is a derived view and must not be treated as a second artifact format.

> Trace: D79, D83, D265
> Covers: `.koi` has one canonical binary artifact format and derived inspection views are not separate authorities.

The first bytes of a `KBI-1` file are the magic bytes `0x4B 0x4F 0x49 0x0A`, spelling `KOI\n`, followed by `container_major: u16`, `container_minor: u16`, `section_count: u32`, and `section_table_offset: u64`. Fixed-width integers in the container are little-endian unsigned integers. Strings are UTF-8 byte strings with unsigned length prefixes and no reader-side normalization.

> Trace: D79, D83, D265
> Covers: The `.koi` container header, integer encoding, and string encoding are fixed.

The section table is sorted by numeric section id. Duplicate section ids are illegal. Unknown required sections make the artifact unsupported. Unknown optional sections may be skipped by a reader that does not understand them, but skipped optional sections remain covered by artifact hashes.

> Trace: D79, D83, D265
> Covers: Section ordering, duplicate handling, and unknown-section handling are deterministic and versioned.

`KBI-1` requires these sections:

| Id | Section | Required Contents | Trace |
| ---: | --- | --- | --- |
| 1 | `manifest` | Package identity, package version, edition, module root, workspace/package owner facts. | D78, D105, D265 |
| 2 | `producer` | Compiler version, compiler compatibility classes, KBI version, diagnostic schema where relevant. | D79, D265 |
| 3 | `target` | Target triple, target contract hash, backend contract class, CPU-feature baseline if it affects interface shape. | D80, D149, D265 |
| 4 | `sources` | Module set, source-origin records, interface hashes, selected body hashes where interface-affecting. | D52, D78, D83, D265 |
| 5 | `imports` | Dependency package identities, dependency `.koi` hashes, lockfile dependency ids. | D51, D79, D83, D265 |
| 6 | `declarations` | Visibility-marked public/internal declaration graph. | D17, D79, D265 |
| 7 | `types` | Nominal type ids, universes, visible opacity, visible layout facts. | D42, D79, D265 |
| 8 | `typeclasses` | Typeclass definitions, associated types, method surfaces, default-method availability. | D82, D182, D265 |
| 9 | `instances` | Legal exported/internal instances and coherence keys. | D79, D82, D265 |
| 10 | `generics` | Generic signatures and materialization metadata required for downstream checking/code materialization. | D82a, D82b, D265 |
| 11 | `contracts` | `require`/`ensure` surfaces, failure summaries, capability requirements. | D53, D85, D265 |
| 12 | `unsafe_audit` | Unsafe modules, unsafe contracts, FFI surfaces, capability audit surface. | D20, D150, D245, D265 |
| 13 | `docs` | Documentation comment hashes and doc extraction metadata. | D218, D265 |
| 14 | `hashes` | Canonical section hashes and whole-artifact hash. | D83, D265 |

> Trace: D17, D20, D42, D51-D53, D78-D80, D82-D82b, D83, D85, D105, D149-D150, D182, D218, D245, D265
> Covers: `KBI-1` required sections cover identity, provenance, target, sources, imports, declarations, types, typeclasses, instances, generics, contracts, unsafe audit, docs, and hashes.

## Koi Semantic Contents

A `.koi` artifact represents the checked interface graph after parsing, name resolution, target selection, declaration-guard evaluation, type checking, typeclass checking, contract checking, capability checking, and unsafe-audit coverage checking for the selected target/profile inputs that affect interface shape.

> Trace: D19, D29, D53, D79, D105, D150, D245, D265
> Covers: `.koi` stores checked interface semantics, not unchecked source text.

A `.koi` artifact does not preserve unchecked source syntax, comments except through documentation metadata, private body declarations, private statement bodies, or compiler memory layouts. It stores generic body materialization metadata only where the generics contract requires downstream packages to materialize checked generic code without reading upstream source.

> Trace: D17, D79, D82b, D218, D265
> Covers: `.koi` excludes private source and unchecked syntax while allowing explicitly versioned generic materialization metadata.

Every declaration record stores declaration kind, module path, visibility, source span fingerprint when available, canonical name, type and universe information, generic parameters, `where` obligations, contract summary, capability requirements, unsafe marker when present, deprecation or compatibility metadata, and links to referenced type/typeclass/instance ids.

> Trace: D17, D29, D53, D79, D85, D158, D189, D265
> Covers: Declaration metadata is explicit enough for downstream checking, documentation, diagnostics, and SemVer/audit tools.

Every declaration id is derived from package id, logical module path, declaration kind, canonical source name, generic parameter shape, and a disambiguating ordinal where Kyokai admits same-name overload families. Declaration ids must not include filesystem traversal order, host paths, memory addresses, or unstable hash iteration order.

> Trace: D78, D83, D214, D265
> Covers: Declaration identity is deterministic and independent of host iteration or memory layout.

Types are encoded as canonical typed graph nodes, not as pretty-printed source strings. Nominal identities, universe classification, type parameters, const parameters, associated-type projections, borrow/reference types, arrays, built-ins, records, unions, extern records, packed records, and opaque types each have explicit tags. Visible layout information is recorded only at the opacity level promised by the source interface and ABI/layout rules.

> Trace: D6, D24, D33, D42, D55, D79, D159, D188, D265
> Covers: Type metadata is structured, canonical, and visibility/opacity aware.

Typeclass records store method signatures, associated types, default method availability, and coherence identity. Instance records store dispatch type, implementing package/module, satisfied obligations, associated type bindings, exported/internal visibility, and overlap/coherence key. Instance bodies are not exposed except through generic materialization metadata explicitly admitted by the generics contract.

> Trace: D79, D82, D82b, D182, D216, D265
> Covers: Typeclass and instance metadata supports static dispatch and coherence without exposing hidden runtime dictionaries.

The generics section records enough checked metadata for downstream packages to typecheck and materialize required generic bodies without re-parsing upstream source. This metadata is not a runtime dictionary and must not encode hidden dynamic dispatch. It records checked generic body IR or another versioned materialization description, compatibility class, referenced declarations, captured constants/comptime values, and invalidation fingerprints.

> Trace: D18, D82, D82a, D82b, D83, D144, D265
> Covers: Generic materialization data is explicit, versioned, cache-invalidatable, and still static-dispatch only.

## Koi Canonicalization And Inspection

All maps inside `.koi` are serialized in bytewise sorted key order. Lists whose source order is semantically meaningful preserve source order. Lists whose order is not semantically meaningful use canonical sorted order. Hashes are computed over normalized section bytes after path remapping and before any future compression wrapper.

> Trace: D83, D265
> Covers: `.koi` serialization and hashing are deterministic.

Compression is not part of `KBI-1`. A future compressed wrapper must hash the uncompressed canonical bytes and must not change compatibility semantics.

> Trace: D79, D83, D265
> Covers: Future compression cannot change artifact identity or compatibility meaning.

The toolchain provides `kyokai koi verify <file>`, `kyokai koi print <file> --format json`, `kyokai koi print <file> --format text`, and `kyokai koi diff <old> <new>`. `verify` checks container structure, required sections, section hashes, compatibility fields, and deterministic ordering. `print` emits a derived view. `diff` classifies public API, internal API, contract, generic metadata, target, and hash changes.

> Trace: D29, D79, D83, D223, D265
> Covers: `.koi` artifacts are inspectable, verifiable, and diffable without making derived text authoritative.

A compiler must reject a `.koi` with malformed container structure, unsupported required section, duplicate section id, invalid UTF-8 string, noncanonical ordering, hash mismatch, unsupported KBI major version, edition mismatch, target contract mismatch, dependency hash mismatch, missing required declaration/type reference, or visibility violation.

> Trace: D29, D79, D83, D105, D265
> Covers: Malformed or incompatible `.koi` artifacts fail with explicit diagnostics.

A `.koi` artifact is not a stable ABI promise by itself, not an archive of object code, not a source distribution format, and not a documentation format. It must not expose private implementation merely because exposing it would make an early compiler easier to write.

> Trace: D17, D79, D139, D218, D265
> Covers: `.koi` non-goals preserve the boundary between interface metadata, binary ABI, source distribution, docs, and private implementation.

## Koi Compatibility

A compiler may consume a `.koi` artifact only when the language edition, KBI major version, target contract, built-in/stdlib interface identity required by the package, dependency artifact hashes, and explicitly versioned generic/codegen compatibility fields match exactly. The producing compiler version is recorded for provenance and diagnostics, but a compiler-version mismatch alone is not automatically incompatible if the compatibility-class fields match.

> Trace: D79, D83, D105, D265
> Covers: `.koi` compatibility is exact over edition, KBI major version, target contract, dependency artifact hashes, required built-in/stdlib identity, and versioned generic/codegen contracts, with compiler version recorded separately for provenance.

A compiler may reject a `.koi` format version it does not implement. It must not reinterpret an unsupported artifact as though it were a best-effort cache. Rejection must produce a diagnostic that names the unsupported field or format version when known.

> Trace: D79, D29
> Covers: Unsupported `.koi` versions are explicit diagnostics, not silent best-effort reinterpretations.

Mixed-edition workspaces are legal as repository structure, but cross-edition `.koi` consumption is not implied. Under the current design, a package may not consume a `.koi` artifact produced for a different language edition.

> Trace: D79, D105, D243
> Covers: Mixed-edition workspaces may exist, but `.koi` dependencies require exact language-edition match.

## Separate Compilation Boundary

Package-level separate compilation is the required artifact boundary. A dependency package can be checked and compiled to reusable artifacts, and downstream packages can consume its `.koi` without rechecking the dependency's private source bodies. Module-level incremental recompilation inside a package is permitted as an optimization, but it is not the public dependency boundary.

> Trace: D78, D79, D83
> Covers: Separate compilation is package-level first, with module-level incremental behavior treated as an internal optimization.

Compiled code artifacts for a package are separate from `.koi` interface artifacts, but their identities must be tied to the same package, edition, target, source, dependency, and compatibility inputs used for reproducible builds. A downstream package may not link code whose interface artifact is incompatible with the checked dependency interface it used.

> Trace: D79, D83, D139
> Covers: Code artifacts and interface artifacts share reproducible identity inputs, and linking must not pair incompatible code with a different checked interface.

Generic and typeclass materialization metadata stored in `.koi` is governed by the generics and backend chapters. The artifact must not hide runtime dictionaries or erased trait-object machinery that the language rejects. If downstream compilation needs generic bodies or materialization descriptions, the `.koi` compatibility contract must state exactly what is available.

> Trace: D79, D82, D82a, D82b, D193
> Covers: `.koi` generic metadata must respect Kyokai's static-dispatch and no-hidden-runtime-dictionary model.

## Diagnostics And Auditing

Module-resolution diagnostics must name the package, logical module name, source path or artifact, import declaration, and visibility rule involved when that information is available. A diagnostic must not report only a filesystem failure when the real error is module identity, package visibility, edition mismatch, duplicate logical modules, or incompatible artifact format.

> Trace: D29, D78, D79, D214
> Covers: Module-resolution diagnostics identify the relevant package, module, import, visibility, and artifact rule.

Audit tooling must be able to list, per package, modules marked `Unsafe_Module`, unsafe contracts, foreign blocks, public declarations exposed by unsafe modules, dependencies with unsafe surfaces, and `.koi` artifact provenance. This audit report reads module/package metadata; it does not change language visibility or grant authority.

> Trace: D20, D79, D245, D255
> Covers: Audit tooling exposes unsafe module and artifact provenance while preserving ordinary visibility and capability rules.

Documentation generation must follow visibility. Public docs for external consumers show public declarations. Same-package/internal docs may include internal declarations when explicitly requested. Private body declarations are excluded from public docs by default.

> Trace: D17, D29, D79
> Covers: Documentation output respects public/internal/private visibility and uses `.koi` metadata where appropriate.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
The `.koi` file is the sealed envelope at the package boundary. Not a cache. Not whatever the last compiler happened to remember. It is the checked interface contract: names, visibility, types, instances, edition, target, and enough generic truth for the next package to stand on it without walking through private rooms.

> Trace: D79, D83
> Covers: `.koi` makes package interfaces inspectable, reproducible, and consumable without exposing private implementation.
