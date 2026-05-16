# Manifest, Packages, And Workspaces

[Rikona Kurasaki / Mjoyufull]
A Kyokai project does not become a package because a tool guessed right while walking through folders. The package boundary is written down. The workspace boundary is written down. The module root is written down. That is the whole point: the build should not feel like a hallway full of unmarked doors where the compiler is the only one who knows which one opens.

> Trace: D78, D83, D155
> Covers: Kyokai packages and workspaces are manifest-declared boundaries, not inferred directory folklore, and tool behavior must be specified rather than discovered by accident.

The normative manifest file is `kyokai.toml`. A directory containing `kyokai.toml` is either a package root or a workspace root, never both. A tool must reject a manifest that contains both `[package]` and `[workspace]` tables.

> Trace: D78
> Covers: `kyokai.toml` is the manifest file, and a manifest is exactly one of package manifest or workspace manifest.

A package is the unit of dependency, visibility, build artifacts, edition selection, and `.koi` interface production. A workspace is an explicit collection of packages built together under one lockfile. A module is the source-level thing imported by Kyokai code. Kyokai deliberately collapses Rust's crate/package split into one package concept so there is one unit for dependency identity and artifact identity.

> Trace: D78, D79, D105
> Covers: Kyokai distinguishes modules, packages, and workspaces, and package identity controls dependencies, visibility, edition selection, and interface artifacts.

## Package Manifests

A package manifest contains a `[package]` table, exactly one package name, exactly one version string, exactly one edition string, and a `[layout]` table with exactly one `module_root` field.

```toml
[package]
name = "net"
version = "0.1.0"
edition = "2026"

[layout]
module_root = "src"
```

> Trace: D78, D105, D243
> Covers: Package manifests declare package identity, version, language edition, and explicit module root.

A package name must match `^[a-z][a-z0-9-]{0,63}$`. It uses only lowercase ASCII letters, ASCII digits, and `-`; it may not end in `-`; it may not contain `--`; it may not contain `_`, `.`, whitespace, or uppercase letters; it is compared byte-for-byte without case folding or punctuation normalization. Names that collide with Windows reserved device names such as `con`, `prn`, `aux`, `nul`, `com1` through `com9`, and `lpt1` through `lpt9` are illegal.

> Trace: D78
> Covers: Kyokai package names have one canonical cross-platform grammar and comparison rule.

The `edition` field selects the source-semantics mode for every `.kyo` and `.kai` file in the package. For the first edition, the value is `"2026"`. A workspace may contain packages with different declared editions, but `.koi` compatibility is exact by edition unless a later spec chapter defines a cross-edition interface normalization format.

> Trace: D79, D105, D243
> Covers: Language editions are manifest-selected source-semantics modes, mixed-edition workspaces are structurally legal, and `.koi` consumption requires exact edition compatibility under the current design.

The `[layout].module_root` value is a non-empty relative path interpreted from the package root. It must not be absolute. It must not escape the package root through `..` or symlink-equivalent resolution. There is no implicit default module root.

> Trace: D78
> Covers: Every package declares a module root explicitly, and the root must remain inside the package.

## Workspace Manifests

A workspace manifest contains a `[workspace]` table and an explicit `members` array. A workspace manifest does not contain `[package]`, `[layout]`, or package dependencies for itself.

```toml
[workspace]
members = [
    "packages/core",
    "packages/net",
    "packages/cli",
]
```

> Trace: D78
> Covers: Workspaces exist only by explicit manifest declaration and list member packages directly.

Each workspace member path is a non-empty relative path from the workspace root to a package root containing its own `kyokai.toml`. Member paths must not be absolute, must not escape the workspace root through `..` or symlink-equivalent resolution, and must not identify the workspace root itself.

> Trace: D78
> Covers: Workspace members are explicit package-root paths contained by the workspace root.

Package names must be unique within a workspace. If two workspace members declare the same `[package].name`, the workspace is ill-formed. A dependency written with `{ workspace = "name" }` resolves by package name, not by filesystem path.

> Trace: D51, D78
> Covers: Workspace package identity is name-based, package names are unique in a workspace, and workspace dependencies refer to package identity rather than paths.

A directory tree containing several packages is not a workspace unless a `kyokai.toml` explicitly declares `[workspace]`. Tools must not infer a workspace from folder layout, nested manifests, a repository root, or a shared parent directory.

> Trace: D78
> Covers: Kyokai rejects inferred workspaces; workspace structure is a written contract.

## Manifest Discovery

When a tool command runs from a directory and no manifest path is passed explicitly, the tool searches that directory and its ancestors for the nearest `kyokai.toml`. If the nearest manifest is a package manifest, the command runs in standalone-package scope. If the nearest manifest is a workspace manifest, the command runs in workspace scope.

> Trace: D78, D26
> Covers: Tool commands use nearest-manifest discovery and distinguish package scope from workspace scope by manifest contents.

If a package is a member of a workspace, commands run from inside that package may still need workspace context for dependency resolution and lockfile ownership. The tool must identify the enclosing workspace by the explicit workspace membership relation, not by guessing that any ancestor with a manifest owns the package.

> Trace: D78
> Covers: Workspace context comes from explicit membership, not accidental ancestor layout.

A package manifest nested under another package's module root is illegal unless it is also an explicitly listed workspace member and the outer package's module discovery excludes that directory. A package cannot silently contain another package's source as ordinary modules.

> Trace: D78, D155
> Covers: Nested package boundaries must be explicit and cannot be swallowed by module discovery.

## Dependencies

A package's `[dependencies]` table admits exactly two dependency source kinds: a package in the same workspace or an external Git repository pinned by commit.

```toml
[dependencies]
core = { workspace = "core" }
pcre = { git = "https://github.com/kyokai/pcre", rev = "a1b2c3d4..." }
pcre-release = { git = "https://github.com/kyokai/pcre", tag = "v1.2.3", rev = "a1b2c3d4..." }
```

> Trace: D51, D78
> Covers: Dependencies are either workspace package references or pinned Git dependencies.

A dependency entry must contain exactly one of `workspace` or `git`. If `git` appears, `rev` is mandatory. `tag` is optional metadata, and if present the package manager must verify that the tag resolves to the declared `rev` when adding or updating the dependency. `branch` is illegal in `kyokai.toml`.

> Trace: D51
> Covers: Git dependencies are pinned by commit, tags are checked labels, and moving branch references are rejected.

The dependency key is the local package dependency name used by the manifest and lockfile. For workspace dependencies, the `workspace` value names the package identity declared by the target package. For Git dependencies, the fetched package's manifest still defines the package identity used in artifacts and diagnostics.

> Trace: D51, D78, D79
> Covers: Dependency manifests separate local dependency entries from package identity, and artifacts record package identity from the package manifest.

A package may not depend on itself. A workspace package dependency cycle is illegal unless a later toolchain chapter defines a specific cycle-breaking artifact protocol. No such protocol exists in this phase.

> Trace: D78, D79, D155
> Covers: Package dependency graphs must be acyclic under the current separate-compilation model.

## Lockfiles

A standalone package owns `package-root/kyokai.lock`. A workspace owns exactly one `workspace-root/kyokai.lock` covering all member packages. Member packages inside a workspace do not own separate lockfiles for that workspace build.

> Trace: D51, D78, D83
> Covers: Lockfile ownership follows package or workspace scope, with one lockfile for a workspace and one for a standalone package.

The lockfile records the fully resolved dependency graph, including exact Git revisions, checked tag metadata where present, package identities, versions, editions, and any artifact identity inputs required by reproducible-build rules. A manifest never relies on a moving external reference after resolution.

> Trace: D51, D79, D83
> Covers: Lockfiles make dependency resolution reproducible and include exact external revisions and artifact identity inputs.

Existing lockfiles continue to resolve yanked package revisions unless a separate security policy explicitly blocks the build. Yanking affects new dependency resolution, not the meaning of an existing lockfile.

> Trace: D244
> Covers: Package yanks are append-only metadata that prevent new selection without breaking existing lockfile reproducibility.

## Tool Obligations

A conforming tool must validate the manifest shape before discovering modules. If the manifest is invalid, the tool reports manifest diagnostics and does not partially interpret source files under a guessed project shape.

> Trace: D26, D78, D155
> Covers: Manifest validation precedes module discovery, and invalid manifests do not fall back to guessed layout.

The formatter, documentation generator, LSP, test runner, package manager, audit tooling, and build command all consume the same manifest/package/workspace model. They may expose different commands, but they must not invent alternate package roots, alternate module roots, alternate dependency resolution, or alternate visibility boundaries.

> Trace: D25, D26, D28, D29, D78, D83, D155
> Covers: All Kyokai tools share the same package/workspace/module-root contract.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
C lets header paths grow into a private weather system. Python ties module names to filesystem shape but leaves public/private access mostly to naming culture. Rust splits crate and package in a way that works, but it gives Kyokai one more concept than it needs. Kyokai chooses the boring door with the label on it: one package identity, one written module root, one workspace list, one lockfile owner, and one way for `Foo.Bar` to become files.

> Trace: D78
> Covers: Kyokai chooses explicit package/workspace/module-root declarations to avoid ambient include paths, naming conventions, inferred workspaces, and extra package/crate identity layers.
