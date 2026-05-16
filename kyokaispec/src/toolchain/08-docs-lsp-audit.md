# Documentation, LSP, And Audit

[Rikona Kurasaki / Mjoyufull]
Docs, editor tooling, and audit reports look like separate tools from the street. Inside Kyokai they stand on the same floor: the checked interface graph, the package boundary, visibility, contracts, capabilities, and unsafe facts the compiler already knows.

> Trace: D148, D150, D218
> Covers: Documentation, LSP, and audit tooling share the compiler engine and project graph.

## Documentation Generator

`kyokai doc` generates documentation from checked `.kyo` interfaces and the `.koi` artifacts of dependencies. Public documentation shows public declarations by default. `internal` declarations are shown only for same-package/internal documentation modes. Private body declarations are excluded from public docs.

> Trace: D17, D79, D218
> Covers: Documentation follows visibility and interface artifacts.

Documentation output includes HTML for humans and JSON for tools. Both outputs include package identity, package version, language edition, module names, public declarations, type signatures, typeclass and instance surfaces, associated types, contracts, documented failure behavior, capability requirements, unsafe markers, deprecation markers, examples, and doc-test metadata.

> Trace: D53, D79, D105, D150, D218, D223
> Covers: Docs expose the API facts needed by users, tools, SemVer checks, and audits.

Documentation must render `require` and `ensure` contracts as first-class API facts. It must not bury contract clauses as unstructured comments when the compiler has checked them as contracts.

> Trace: D53, D218
> Covers: Contracts are visible documentation surface.

The documentation JSON schema must be versioned. A tool consuming docs must be able to distinguish a declaration removed from output, a declaration hidden by visibility mode, and a declaration whose source failed to check.

> Trace: D17, D29, D218
> Covers: Documentation JSON has stable schema and meaningful absence states.

## LSP

`kyokai lsp` is the official language server. It uses the same parser, resolver, type checker, borrow checker, capability checker, target guard evaluator, `.koi` reader, formatter, and diagnostic engine as the command-line compiler. It may cache and schedule work differently, but it must not define a second semantic model.

> Trace: D29, D79, D148
> Covers: The LSP shares compiler truth instead of becoming a parallel compiler.

The LSP must respect the same manifest discovery and workspace rules as the CLI. An editor workspace containing several manifests is not automatically a Kyokai workspace unless a `kyokai.toml` declares `[workspace]` and members.

> Trace: D78, D148
> Covers: Editor project shape follows the same manifest contract.

LSP diagnostics use the same diagnostic codes and JSON-compatible fields as CLI diagnostics. The LSP may stream partial diagnostics while the user edits, but completed diagnostics for a stable source snapshot must match `kyokai check` for the phases the LSP reports.

> Trace: D29, D148
> Covers: LSP diagnostics are compatible with CLI diagnostics.

Editor features such as go-to-definition, find-references, rename, completion, hover, inlay hints, semantic tokens, code actions, and formatting must observe visibility, target guards, editions, imports, typeclass resolution, and generated-source boundaries. A code action that rewrites source must be represented as an ordinary edit and must not bypass `kyokai fmt` or compiler validation.

> Trace: D25, D78-D79, D105, D148
> Covers: LSP features follow normal language and toolchain rules.

## Audit

`kyokai audit` reports supply-chain, unsafe, FFI, capability, native dependency, generated-source, target, license, and public API risk facts. It does not change language semantics and does not grant authority. It reads manifests, lockfiles, source interfaces/bodies where available, `.koi` artifacts, package index metadata, target-spec files, and configured audit policy.

> Trace: D20, D51, D79, D150, D221, D244, D245
> Covers: Audit is a read-only risk report over package, source, dependency, and artifact facts.

Audit output includes at least these categories:

| Category | Required Facts | Trace |
| --- | --- | --- |
| Dependencies | Git URL, pinned revision, tag label, package identity, version, yanked status, advisories when known. | D51, D221, D244 |
| Unsafe | Unsafe modules, unsafe contracts, raw pointer use, volatile/MMIO, inline assembly, dynamic loading, and unsafe capability access. | D20, D22, D94, D150, D245, D257 |
| FFI/native | Foreign blocks, exported ABI surfaces, native libraries, link flags, ownership/failure contracts, and transitional FFI status. | D20, D31, D150, D230 |
| Capabilities | Public APIs requiring capabilities, capability derivation/splitting/surrender, task-transfer capability surfaces, and root authority entrypoints. | D48, D137, D150, D211, D255 |
| Generation | Build-time generators, declared inputs/outputs, generator capabilities, and generated source provenance. | D83, D150 |
| Reproducibility | Hidden-input risks, path remapping gaps, target-spec drift, lockfile freshness, and non-reproducible output modes. | D83 |
| API | Public declarations exposed by unsafe modules, SemVer-relevant interface changes, deprecated APIs, and contract changes. | D17, D218, D223 |

> Trace: D17, D20, D22, D31, D48, D51, D83, D94, D137, D150, D211, D218, D221, D223, D230, D244-D245, D255, D257
> Covers: Audit reports cover dependencies, unsafe code, FFI/native links, capabilities, generation, reproducibility, and API risk.

An audit policy may promote categories to errors. For example, a project may reject yanked dependencies, public unsafe surfaces, undeclared native libraries, unreviewed build generators, or packages whose `.koi` provenance cannot be verified. Policy failures use audit diagnostics and exit status `1`.

> Trace: D29, D150, D244
> Covers: Audit policy is explicit and CI-enforceable.

Audit must distinguish implementation ceiling from public surface. A package may use unsafe internally without exposing unsafe authority publicly, but the audit report must show both facts so reviewers can tell the difference between contained risk and exported risk.

> Trace: D17, D150, D245
> Covers: Audit separates internal implementation risk from public API risk.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
Documentation is the face of the API. The LSP is the hand on your shoulder while you write it. Audit is the person checking the locks after sunset. They cannot be three different stories. If the compiler knows a function needs authority, or an unsafe wrapper holds the line against C, those tools should show it plainly.

> Trace: D148, D150, D218
> Covers: Docs, LSP, and audit are trustworthy only when they share the compiler's semantic facts.
