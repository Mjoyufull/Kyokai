# Modules And Visibility

[Rikona Kurasaki / Mjoyufull]
Kyokai keeps Austral's strongest module idea: a module has an interface and a body, and the interface is the surface other code can trust before the private work is even visible. Kyokai makes that old boundary sharper. The source extensions are Kyokai's own, the package boundary is explicit, `internal` has a defined reach, and imports have one mechanical lookup story.

> Trace: D5, D17, D52, D78, D86
> Covers: Kyokai preserves Austral's interface/body module split while specifying `.kyo` interfaces, `.kai` bodies, package-visible `internal`, and deterministic package-rooted module resolution in the Kyokai spec itself.

A Kyokai module is the language-level unit named by a dotted module path such as `Kyokai.Core.Result` or `App.Main`. A source module is represented by one interface file and one body file for the same module name: `Name.kyo` contains the importable interface surface, and `Name.kai` contains the implementation body selected for the current build. A dependency may also provide a checked `.koi` interface artifact instead of source text, as specified by the toolchain chapter.

> Trace: D52, D78, D79
> Covers: Modules are named language units, `.kyo` files provide source interfaces, `.kai` files provide source bodies, and `.koi` artifacts provide checked package interface contracts for downstream compilation.

The module declaration inside a source file must match the logical module path assigned by package module resolution. If `Foo.Bar` maps to `src/Foo/Bar.kyo` and `src/Foo/Bar.kai`, those files must declare `module Foo.Bar is ... seal;` and `module body Foo.Bar is ... seal;`. A mismatch is a compile-time error before name resolution inside the module proceeds.

> Trace: D52, D78
> Covers: The declared module name must match the manifest-rooted file mapping, so source paths and module declarations cannot drift into two different identities.

## Interface And Body

A `.kyo` interface file contains the declarations other modules may typecheck against. It may declare constants, types, records, unions, capabilities, functions, typeclasses, instances, generators, and other interface-admitted declarations from the declaration chapter. It does not contain private helper declarations that are meant only for the module body.

> Trace: D17, D52, D78
> Covers: `.kyo` files are the importable interface surface and contain declarations visible according to public or `internal` visibility.

A `.kai` body file contains the implementation for the same module. It may define body declarations, private helper types and functions, foreign blocks, unsafe contracts, and other body-admitted declarations from the declaration chapter. A declaration that exists only in the body is private to that module body and cannot be imported, selected, re-exported, named by qualified access from another module, or recorded as public API.

> Trace: D17, D20, D52, D78, D245
> Covers: `.kai` files own implementation and private declarations, and body-only declarations do not cross the module boundary.

An interface may be checked before the body is checked. A module that imports another module typechecks against the imported module's interface surface, not against its body. This is the old Austral modularity rule carried forward: clients depend on contracts, not implementation rooms they should not enter.

> Trace: D5, D78, D79
> Covers: Kyokai preserves interface-first checking: clients typecheck against `.kyo` or `.koi` interface contracts rather than private body source.

A module body must implement the declarations from its interface that require implementation under the declaration and typechecking chapters. Missing required definitions, incompatible signatures, weaker contracts, or representation exposure that contradicts the interface are compile-time errors. The precise compatibility checks for each declaration kind are defined in the later declaration, type, contract, and unsafe chapters.

> Trace: D17, D53, D78, D79, D155
> Covers: A body must satisfy its interface, and any interface/body divergence is a compile-time error resolved by the relevant normative declaration and type rules.

## Visibility Levels

Kyokai has exactly three source visibility levels.

| Visibility | Where It Is Written | Who May Name It |
| --- | --- | --- |
| Public | `.kyo` declaration with no `internal` marker | Any package that imports the module and has dependency access to the package. |
| Internal | `.kyo` declaration prefixed with `internal` | Only modules in the same package. |
| Private | Declaration present only in `.kai` body source | Only the declaring module body. |

> Trace: D17, D78
> Covers: Kyokai visibility is public, package-internal, or module-private; there is no workspace visibility and no path-relative visibility lattice.

`internal` is legal only in interface files. Writing `internal` in a `.kai` body is a compile-time error because body-only declarations are already private. `internal` changes who may name a declaration; it does not change whether a type is opaque or transparent, does not change layout, does not change typeclass coherence, and does not grant unsafe authority.

> Trace: D17, D20, D245
> Covers: `internal` is an interface-only package visibility marker and does not alter opacity, layout, coherence, or unsafe authority.

Workspace membership does not widen visibility. Two packages in the same workspace are still separate packages for `internal`. A package may not import another package's internal declaration merely because both packages are listed in the same `[workspace].members` array.

> Trace: D17, D78
> Covers: `internal` is package-visible, never workspace-visible.

An internal declaration cannot be surfaced across a package boundary by import tricks, aliasing, documentation generation, `.koi` consumption, or a later public declaration that simply exposes the same name. Public APIs may mention only declarations that are public to the consuming package. If a public declaration's signature mentions an internal type from the same package, that public declaration is not importable outside the package unless the later declaration/type chapters define a safe opaque exposure form for that exact case.

> Trace: D17, D79, D229
> Covers: Internal declarations cannot leak through imports, artifacts, docs, or public signatures as accidental external API.

## Imports

Imports are file-scope declarations and must appear before the module declaration's body items. There are no function-local imports, block-local imports, expression-local imports, wildcard imports, `open`, `using namespace`, or import-order priority rules.

> Trace: D78, D179, D214
> Covers: Imports are file-scope only and Kyokai rejects wildcard/open imports and import-order-dependent meaning.

Kyokai has exactly three import forms:

```kyokai
import Foo.Bar;
import Foo.Bar as Bar;
import Foo.Bar (baz, qux as localQux);
```

The first form introduces the module path for qualified access. The second introduces the same module for qualified access through the alias. The third introduces only the listed direct exports as unqualified names, applying each `as` rename before collision checking.

> Trace: D179, D214
> Covers: Kyokai imports support qualified module access, module aliases, and selective unqualified imports with per-name renaming.

Selective imports name direct exports of the imported module only. They do not recursively import exports from modules that the imported module imports, and they do not create transitive namespace injection. If code wants both qualified module access and unqualified selected names, it writes both imports explicitly.

> Trace: D179, D214
> Covers: Selective imports are direct and non-transitive, and qualified access is separate from unqualified member import.

A qualified module import does not introduce each exported declaration as an unqualified name. A selective import does introduce unqualified names. If two imports would introduce the same unqualified name after renaming, the source file is ill-formed at the import site. Import order never chooses the winner.

> Trace: D179, D214
> Covers: Qualified imports avoid unqualified collisions, selective import collisions are compile-time errors, and import order has no semantic effect.

Built-in language names cannot be introduced or shadowed by imports. This includes `Ok`, `Err`, `Some`, `None`, `true`, `false`, `target`, and the built-in type and target descriptor names specified in the built-ins chapter. A selective import or alias that would collide with a protected built-in name is a compile-time error.

> Trace: D24, D214
> Covers: Built-in language names are protected from import shadowing.

## Name Lookup

Name lookup starts from lexical scope, then file-scope imports, then qualified module paths where the program writes a qualified name. A name that is not in scope is a compile-time error. A name that resolves to more than one candidate is a compile-time error unless the source disambiguates with a qualified path or explicit import rename.

> Trace: D60, D179, D214, D254
> Covers: Kyokai name lookup is lexical and import-explicit; missing names and ambiguous names are compile-time errors.

No still-live binding may be shadowed by another binding, pattern binding, parameter, declaration, or import-introduced unqualified name in the same lookup reach. This rule is especially important for linear values: a language that lets one name hide another can make ownership obligations disappear from the reader's eyes.

> Trace: D60
> Covers: Kyokai rejects shadowing of still-live bindings, including shadowing introduced by patterns or imports.

Qualified access through `Foo.Bar.name` may name only declarations visible to the current package. From a different package, only public declarations are visible. From the same package, public and internal declarations are visible. Private body-only declarations are not visible through qualified access from any other module.

> Trace: D17, D179, D214
> Covers: Qualified access respects package visibility and never exposes private body declarations.

UFCS receiver-module lookup is not a fourth import form. It is a narrow fallback used only after ordinary imported-name lookup fails, and it searches only the receiver type's defining module or the explicit owner of a compiler-known receiver surface. It does not search the dependency graph, repair import collisions, or act like C++ argument-dependent lookup.

> Trace: D110, D179, D214, D254
> Covers: UFCS receiver-module lookup is a constrained fallback, not global method search or collision repair.

## Instances And Coherence Across Modules

Typeclass instances follow ordinary visibility and import rules, but coherence is global over the resolved program. For any fully resolved typeclass application visible at a call site, there must be exactly one applicable legal instance. If two legal visible instances could apply, the program is rejected.

> Trace: D17, D214
> Covers: Instance visibility follows module/package visibility, and typeclass calls require one deterministic applicable instance.

An instance may be declared only where the package owns the typeclass or owns at least one concrete head type named by the instance. Internal or private visibility does not relax this orphan rule. A package cannot create a private foreign-typeclass-for-foreign-type instance and rely on scope to hide the coherence problem.

> Trace: D17, D214
> Covers: Kyokai uses an orphan/coherence rule for typeclass instances, and internal/private visibility does not create orphan exceptions.

Internal instances are visible only inside the package. `.koi` artifacts may record internal instances so same-package compilation can use them, but downstream packages must treat those internal entries as nonexistent. Public documentation generated for external consumers must exclude internal instances by default.

> Trace: D17, D79
> Covers: Internal instances may be present in artifacts for same-package checking but are invisible to external package consumers and public docs by default.

## Unsafe Modules And Importability

`pragma Unsafe_Module;` is a module-body marker for modules that contain raw unsafe operations, raw foreign declarations, or other unsafe facilities admitted by the unsafe chapter. The pragma does not make a declaration public, does not widen `internal`, and does not let callers forge capabilities or bypass ordinary visibility.

> Trace: D20, D245, D255
> Covers: `pragma Unsafe_Module;` marks a raw unsafe implementation boundary but does not alter visibility or grant authority by itself.

A safe module may import safe wrapper declarations from an unsafe module when those declarations are public or same-package internal according to the normal visibility rules. A safe module may not call raw foreign declarations or unsafe primitives merely because it can import the module name. Raw unsafe access requires the unsafe chapter's explicit contracts and capabilities.

> Trace: D20, D245, D255
> Covers: Safe code may import safe wrappers exposed by unsafe modules, but raw unsafe operations remain gated by unsafe contracts and explicit capabilities.

Unsafe contracts are source-level declarations that document and bind the unsafe obligations of a module. They are part of the audit surface, and toolchain audit output must be able to locate them through module/package metadata. They are not imports, not capabilities, and not a substitute for visibility rules.

> Trace: D20, D79, D245
> Covers: Unsafe contracts are audit metadata and source obligations for unsafe modules, not a visibility escape hatch.

## Module Resolution Boundary

The language chapter defines what a module, import, visible declaration, and name lookup mean. The toolchain chapter defines how a package manifest chooses roots, how source files are discovered, how `.koi` artifacts are produced and consumed, and how target selection chooses a body. These rules meet at one hard boundary: by the time the language checker resolves imports, the toolchain must provide one selected module graph with one interface surface for every imported module.

> Trace: D19a, D52, D78, D79, D105
> Covers: The toolchain selects package roots, source files, target-specific bodies, editions, and artifacts before language name resolution consumes a single resolved module graph.
