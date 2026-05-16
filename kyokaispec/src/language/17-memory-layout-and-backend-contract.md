# Memory Layout And Backend Contract

[Rikona Kurasaki / Mjoyufull]
A program has a shape before the machine ever touches it. Fields sit in order. Values move. A result lands somewhere. A check either guards the edge or it does not. If the backend is allowed to redraw that shape in secret, the language becomes a rumor told by generated C.

Kyokai does not let that happen. C can carry the program for a while. LLVM can carry it later. Neither one gets to decide what the program means.

> Trace: D4, D42, D73, D89, D139, D199, D228
> Covers: Layout and lowering are language/backend contracts, not backend folklore or optimizer luck.

This chapter defines ordinary layout, extern layout, packed layout, layout introspection, move and result-placement lowering, backend-independent semantics, generated C obligations, LLVM obligations, debug/source mapping, and target/backend failure behavior.

> Trace: D27, D31, D80, D83, D141, D196, D217, D247
> Covers: Backend output must preserve Kyokai values, checks, atomics, source mapping, reproducibility, and target contracts.

## Backend Independence

Kyokai language semantics are backend-independent. No language rule is defined as "whatever the C backend does", "whatever LLVM happens to optimize", or "whatever the target compiler accepts today".

> Trace: D4, D73, D228
> Covers: Backends implement Kyokai; they do not define it.

The C backend is a first-class backend for bootstrap, reference work, generated-code inspection, and target bring-up. It is not a promise that Kyokai source is constrained to pretty portable hand-maintainable C.

> Trace: D4, D139
> Covers: C remains important without owning the language design.

The LLVM backend is the planned long-term primary optimizing production backend after self-hosting and after the project can afford the deeper backend work. Features are judged first by the Kyokai language contract, then backend support is solved explicitly.

> Trace: D4, D104, D228
> Covers: LLVM is the long-term performance and feature ceiling, not the source of semantics.

If a backend/target pair cannot implement a source construct with the required Kyokai semantics, the build fails for that backend/target pair. The compiler must not silently weaken checks, change layout, change memory ordering, drop debug/source obligations, or reinterpret failure behavior to make the backend path convenient.

> Trace: D4, D31, D80, D139, D228
> Covers: Backend limits are explicit build failures, not semantic drift.

## Layout Classes

Every record is exactly one of three layout classes: ordinary `record`, `extern record`, or `packed record`.

> Trace: D42
> Covers: Record layout is a closed type-level choice.

Kyokai has no open-ended `repr(...)` attribute system, no backend-chosen ordinary layout, no hidden field reordering, no automatic C-layout inference, and no hidden niche/layout optimization in ordinary language layout.

> Trace: D42, D73
> Covers: Layout does not depend on backend guessing.

## Ordinary Record Layout

An ordinary `record` uses Kyokai layout. Field order is the source order written by the programmer. The compiler never reorders ordinary record fields.

> Trace: D35, D42
> Covers: Source field order is semantic for ordinary records.

Each field starts at the smallest offset greater than or equal to the end of the previous field that satisfies that field type's alignment. The record alignment is the maximum alignment of its fields. The record size is rounded up to a multiple of that record alignment.

> Trace: D42
> Covers: Ordinary record offsets, alignment, and size are defined by the language.

User-specified over-alignment, under-alignment, C bitfields, backend-defined bitfields, and hidden niche optimizations are not part of ordinary record layout.

> Trace: D42, D116
> Covers: Ordinary records do not import C layout folklore.

An ordinary single-field record is representation-transparent inside Kyokai's own layout and calling model: its size, alignment, and sole field offset match the field type, and passing, returning, storing, and loading behave as though the wrapper had the same representation as its field.

> Trace: D109/D196
> Covers: Nominal wrappers are zero-cost inside Kyokai layout.

Representation transparency does not create type equality, implicit conversion, alias identity, shared typeclass instances, or FFI eligibility. The wrapper remains a distinct nominal type.

> Trace: D190, D196
> Covers: ABI equality is not type identity.

## Extern Record Layout

An `extern record` uses the selected target's C ABI layout rules for its exact field list and field order. Field order remains source order; Kyokai does not reorder extern fields.

> Trace: D20a, D42, D80
> Covers: Extern records are explicit target-C-ABI aggregates.

A record may cross raw `foreign "C"` by value only if it is an `extern record` and every field is FFI-admitted under the unsafe/FFI chapter. Ordinary records and packed records do not cross raw C by value merely because their current representation looks compatible.

> Trace: D20a, D42, D242a
> Covers: By-value aggregate FFI requires explicit extern layout.

If a target C ABI cannot define the extern record's field layout, alignment, passing convention, or return convention under the selected toolchain contract, the declaration is illegal for that target/backend path.

> Trace: D20a, D31, D80
> Covers: Extern record support is target-contract checked.

`extern record` does not permit fields of `extern type` by value, Kyokai sum types by value, ordinary borrows, capabilities, or raw types outside the admitted FFI surface.

> Trace: D20a, D42, D242a, D255
> Covers: Extern layout does not open the whole Kyokai type system to C.

## Packed Record Layout

A `packed record` is byte-tight. Fields remain in source order with no implicit padding between fields. The record alignment is 1. The record size is the sum of its field sizes.

> Trace: D42
> Covers: Packed records have exact byte-tight Kyokai layout.

Reading or writing a packed field uses copy semantics, not reference semantics. The implementation must behave as though it copies bytes into or out of a properly aligned temporary of the field type when alignment requires that.

> Trace: D42, D73
> Covers: Packed access cannot create misaligned safe references.

Taking `&field` or `&!field` of a packed field is illegal. Packed fields cannot produce ordinary Kyokai borrows because that could create potentially misaligned references.

> Trace: D14, D42, D73
> Covers: Packed layout preserves borrow/reference alignment guarantees.

`packed` does not imply C ABI compatibility, bitfields, byte swapping, network byte order, device register semantics, or endianness conversion. Any byte-order conversion at FFI, file, network, packed-layout, or container boundaries must be written explicitly.

> Trace: D42, D117/D260
> Covers: Packed layout and endianness are separate contracts.

`extern packed record` and equivalent combined layout classes are illegal. If a foreign API requires a packed C struct, the boundary must use explicit unsafe marshaling or raw byte storage under an unsafe contract.

> Trace: D20a, D42, D245
> Covers: Packed C ABI edge cases go through unsafe wrappers.

## Bitrecords And Non-Byte Layout

Non-byte-aligned field descriptions use `bitrecord`, not C-style bitfields. A `bitrecord` is a value over a fixed-width unsigned backing integer with explicit bit positions.

> Trace: D116
> Covers: Register and protocol fields are defined by masks and shifts, not C bitfields.

When stored in memory, a `bitrecord` has exactly the storage of its backing integer type. The C backend lowers bitrecord access with masks, shifts, and equivalent helper code. It never emits C bitfields as the semantic implementation strategy.

> Trace: D116, D139
> Covers: Bitrecord lowering avoids backend-defined bitfield layout.

## Layout Introspection

`sizeOf(T)`, `alignOf(T)`, and `offsetOf(T, field)` are compile-time-only built-ins. They are evaluated through `comptime` and use the declared layout class of `T`.

```kyokai
constant HeaderSize: Index := comptime sizeOf(Header);
constant HeaderAlign: Index := comptime alignOf(Header);
constant LengthOffset: Index := comptime offsetOf(Header, length);
```

> Trace: D18/D18a, D42
> Covers: Layout introspection is deterministic compile-time evaluation.

For ordinary records, these built-ins use Kyokai layout. For `extern record`, they use the selected target C ABI layout. For `packed record`, they use byte-tight packed layout. They never use backend guesses.

> Trace: D42, D80
> Covers: Layout introspection follows the type's declared layout class.

`sizeOf`, `alignOf`, and `offsetOf` are illegal for `extern type` by value because the type has unknown size and layout. They are legal for pointer/address forms that mention an `extern type`, because the pointer/address representation is known by the target ABI.

> Trace: D20a, D42
> Covers: Opaque foreign types remain opaque to layout introspection.

## Recursive Layout

Recursive and mutually recursive nominal types are legal only when the fully expanded representation has finite size. Every cycle in the layout-dependency graph must pass through an indirection-bearing field.

> Trace: D160/D217
> Covers: Recursive layout legality is representation-based, not name-based folklore.

Inline constructors, inline record fields, and inline union payloads do not break a layout cycle. `Box[T]`, raw pointer/address forms, and other specified indirection-bearing forms may break cycles according to their type contracts.

> Trace: D160/D217
> Covers: Infinite-size representations are rejected at declaration checking.

A compiler may compute recursive layout legality with any internal graph algorithm. The observable result must match the finite-layout rule and produce diagnostics that identify the cycle and the missing indirection boundary.

> Trace: D29, D160/D217
> Covers: Recursive layout diagnostics expose the actual representation cycle.

## Move Representation

Moving a value has as-if bytewise relocation semantics. The destination receives the exact representation bytes of the source value, and the source location becomes logically dead immediately after the move.

> Trace: D89
> Covers: Move semantics are physical enough to reason about and backend-independent.

The language contract is the as-if rule, not a requirement that the backend emit `memcpy` at every move site. A backend may optimize moves away, fuse them, lower them through registers, use hidden output storage, or otherwise avoid physical copying when observable behavior matches the as-if relocation model.

> Trace: D89, D228
> Covers: Optimizing moves is legal only under the Kyokai move contract.

Safe movable values may not depend on their own storage address. Self-referential ordinary movable values are banned in safe code. Unsafe code that constructs address-sensitive values must expose them through stable-address indirection or pinned-type rules before safe code can observe them.

> Trace: D89, D89a/D89b
> Covers: Address-sensitive values cannot cross into safe code as ordinary movable values.

A moved-from place is dead. Later reads, borrows, moves, destruction, deferred cleanup registration, or assignment uses that would observe the old value are compile errors unless the assignment rule explicitly reinitializes a mutable place before observation.

> Trace: D89, D195
> Covers: Backend move optimization cannot weaken moved-state checking.

## Pinned And Stable-Address Values

A `pinned record` or `pinned union` does not participate in ordinary move semantics. Safe operations that relocate it are compile errors: by-value passing, by-value return, assignment after initialization, swapping, destructuring, field extraction by value, and storage in containers whose safe operations may relocate elements.

> Trace: D89b
> Covers: Pinned types are declaration-site non-movable values.

A type containing a pinned field inline must itself be declared pinned. Generic code may relocate a type parameter only when its constraints include `T: Movable` or an equivalent intrinsic movable condition.

> Trace: D89b
> Covers: Non-movability propagates through inline representation.

`PinBox[T]` owns stable-address storage for pinned values. Moving the `PinBox[T]` value never relocates the pointee. The API must not expose a safe operation that extracts the pinned pointee by value.

> Trace: D89b
> Covers: Stable pointee storage is explicit and API-enforced.

Ordinary `Box[T]` is still ordinary indirection. It gives separate pointee storage, but it does not make `T` pinned. Any operation that moves the pointee out of a box is legal only when `T: Movable`.

> Trace: D89a/D89b
> Covers: Plain boxes are not hidden pin wrappers.

## Direct Result Placement

If a function's return type has size greater than two machine words on the selected target, the implementation must use direct result placement semantics.

> Trace: D199
> Covers: Large return values have a guaranteed lowering shape.

Under direct result placement, the callee initializes storage already designated as the caller's final result object. The implementation must not require an additional full-width relocation of that completed result merely to hand it back to the caller.

> Trace: D89, D199
> Covers: Large returns are not dependent on optimizer folklore.

The guarantee is independent of debug/release profile, optimization level, and backend choice. For return types of size two machine words or smaller, the implementation may use registers, direct result placement, or another equivalent lowering, provided Kyokai's as-if move semantics are preserved.

> Trace: D31, D89, D199
> Covers: Return placement semantics do not change by profile.

Direct result placement does not create a safe-language stable-address guarantee, does not permit self-referential movable values, and does not weaken pinned-type rules. It is a calling/result-lowering rule, not a pinning rule.

> Trace: D89, D89b, D199
> Covers: Result placement and stable-address typing stay separate.

## Defined Lowering Contract

Lowering from elaborated Kyokai into C, LLVM IR, or any later backend must preserve the source program's specified Kyokai outcomes.

> Trace: D228
> Covers: Backend lowering is preservation of Kyokai semantics.

Backend undefined behavior, LLVM `poison` or `undef`, C signed overflow, invalid aliasing assumptions, unchecked trap-producing operations, target-specific unreachable assumptions, or speculative removal of checked failure paths may not be used as the mechanism for implementing safe Kyokai semantics.

> Trace: D73, D139, D228
> Covers: Safe Kyokai semantics cannot be implemented by backend UB.

TPOE and runtime-fatal paths lower to explicit no-return termination operations or checked branches whose existence cannot be optimized away by assuming the failed condition is impossible.

> Trace: D84, D139, D228
> Covers: Fatal checks remain real backend control flow until proven unreachable by Kyokai semantics.

The compiler may attach aliasing, lifetime, `noalias`, alignment, initialization, non-null, range, or dereferenceability metadata only when justified by the elaborated borrow, linearity, type, and contract model.

> Trace: D14, D73, D89, D195, D228
> Covers: Backend metadata must be earned by checked Kyokai facts.

If the compiler cannot justify stronger backend metadata for a construct, it must omit that metadata or fail the build. It must not emit unjustified metadata to recover performance.

> Trace: D228
> Covers: Optimizer hints cannot become lies.

Surface constructs with specified desugarings lower through the elaboration pipeline before linearity, borrow, capability, contract, unsafe, and backend checks rely on them.

> Trace: D238-D240, D228
> Covers: Backends see checked elaborated core, not raw sugar.

Backend optimizations may remove redundant checks only after proving the removed failure path is unreachable under Kyokai semantics. They may not remove checks merely because the backend would treat the failing case as undefined.

> Trace: D73, D84, D228
> Covers: Check removal is proof-driven, not UB-driven.

## C Backend Contract

Generated C for valid Kyokai programs must stay inside a defined C11-compatible subset unless the selected target toolchain contract explicitly admits a named extension or intrinsic family.

> Trace: D31, D80, D139
> Covers: Generated C is constrained by a written supported-toolchain contract.

The selected C compiler family, version floor, required flags, sysroot, target triple, and admitted extension families are part of the toolchain and target contract. Unknown or unsupported combinations fail the build.

> Trace: D31, D80, D139
> Covers: C backend behavior is not host-compiler guessing.

The C backend must preserve Kyokai's left-to-right evaluation order. It may not rely on unspecified C expression evaluation order. When needed, it introduces temporaries and statement sequencing in generated C.

> Trace: D71, D139
> Covers: C expression-order holes cannot change Kyokai evaluation.

Kyokai integer semantics are not C integer semantics. Checked arithmetic lowers to explicit checks or checked intrinsics. TPOE behavior comes from those checks, not from C overflow, not from `-fwrapv`, and not from backend traps.

> Trace: D75, D84, D139
> Covers: Integer checks survive the C backend.

The C backend must not emit strict-aliasing violations. Representation reinterpretation, byte copying, and type punning use `memcpy`-style or equally defined C patterns only.

> Trace: D73, D139
> Covers: C aliasing rules are respected explicitly.

The C backend must not emit UB-producing shifts, division/modulo by zero, uninitialized reads, null dereference, misaligned typed accesses, invalid lifetime use, or invalid object access for valid Kyokai programs. If a Kyokai operation violates its own contract, generated code must produce the Kyokai-specified failure behavior.

> Trace: D73, D75, D84, D139
> Covers: Backend traps are not substituted for Kyokai checks.

For the GCC and Clang support contract, required defensive flags include at least `-std=c11`, `-fwrapv`, `-fno-strict-aliasing`, and `-fno-delete-null-pointer-checks`. These flags are defensive support, not the source of Kyokai semantics.

> Trace: D31, D139
> Covers: Toolchain flags reinforce but do not define language behavior.

The C backend may use standard intrinsics, implementation-defined extensions, generated helper functions, C11 atomics, compiler builtins, or inline assembly facilities only when the selected target/toolchain contract admits them by name.

> Trace: D4, D22, D31, D80, D139, D141
> Covers: C extensions are explicit backend contracts.

The toolchain may provide an extra-assurance C backend profile using CompCert where target support and emitted-C subset compatibility exist. That profile is additional assurance, not the baseline requirement for every Kyokai target.

> Trace: D139
> Covers: CompCert is optional higher assurance, not required everywhere.

## LLVM Backend Contract

The LLVM backend must obey the same defined-lowering contract. LLVM IR `poison`, `undef`, `unreachable`, `nonnull`, `noalias`, alignment metadata, lifetime markers, and range metadata may be used only when justified by elaborated Kyokai facts.

> Trace: D4, D73, D228
> Covers: LLVM does not get a wider semantic escape hatch than C.

LLVM traps, UB-triggering operations, speculative assumptions, or optimizer-only unreachable states may not implement safe Kyokai checked failures. TPOE and runtime-fatal paths remain explicit until removed by proof under Kyokai semantics.

> Trace: D84, D228
> Covers: LLVM lowering preserves checked failure semantics.

The LLVM backend may use native vector IR, target intrinsics, debug metadata, and optimization passes when their use preserves the portable Kyokai contract for the source construct. If an ISA-specific intrinsic is unavailable for the selected backend, target, or CPU-feature baseline, compilation fails instead of scalarizing or changing operation family silently.

> Trace: D104, D228
> Covers: LLVM power is explicit and target-gated.

## Atomics And Memory Model Lowering

Safe atomic operations lower through the memory model defined in the concurrency chapter. Backend choice does not change `MemoryOrder`, happens-before, compare-exchange result, or fence semantics.

> Trace: D90/D90a, D141, D247
> Covers: Shared-memory behavior is language-level, not backend accident.

On the C backend, `Atomic[T]` lowers through C11 `<stdatomic.h>` only. The backend must not lower safe atomic operations to plain reads/writes or to `volatile` accesses.

> Trace: D141
> Covers: C volatile is not atomics.

The backend must ensure generated atomic storage satisfies the selected target/toolchain's C11 or LLVM atomic alignment requirements. If a source-level combination cannot satisfy that contract, compilation fails for that target/backend path.

> Trace: D141, D228
> Covers: Atomic alignment failures are build errors, not weakened memory behavior.

## Volatile Lowering

Volatile operations lower according to the unsafe chapter. The C backend uses C `volatile` loads/stores for the admitted volatile type domain. The LLVM backend uses LLVM volatile load/store operations.

> Trace: D94/D257, D139, D228
> Covers: Volatile access has backend-specific implementations with one language contract.

Volatile lowering preserves the externally observable volatile access ordering required by the unsafe chapter. It does not create synchronization, atomicity, or happens-before edges.

> Trace: D90/D90a, D94/D257, D247
> Covers: Volatile backend code does not grow hidden concurrency semantics.

## Debug Information And Source Mapping

The C backend emits `#line N "path/to/File.kai"` directives in generated C where needed to map generated lines back to Kyokai source lines. Debug profiles compile generated C with the target contract's debug-symbol flag, such as `-g` for GCC/Clang-style toolchains.

> Trace: D27, D31
> Covers: Current source-level debugging flows through generated C line mapping.

When the programmer debugs a compiled program, breakpoints, stack traces, and single-stepping should report Kyokai source locations where the backend can faithfully preserve them. Complex lowered expressions may map to multiple generated locations, but the mapping must not point to unrelated user source.

> Trace: D27, D29
> Covers: Debug mapping is useful and honest rather than decorative.

The LLVM backend emits DWARF or the selected target's equivalent source-level debug information directly. The exact debug-info level belongs to the toolchain/profile contract, but the language-level obligation is that debug metadata maps backend instructions to Kyokai source spans where such mapping is available.

> Trace: D27, D31, D86
> Covers: LLVM debug info is native, but still source-oriented.

Variable names in debug output should preserve Kyokai names where possible after hygiene, monomorphization, and lowering. Generated temporaries and helper variables must be distinguishable from source variables.

> Trace: D27, D29
> Covers: Debugger-visible state does not confuse generated scaffolding with user bindings.

## Coverage And Generated Helpers

Coverage is reported in Kyokai source terms, not generated C, LLVM IR, or helper code terms. Backend-generated scaffolding for overflow checks, contract checks, wrappers, drops of helper state, or fatal-path plumbing must not count as user-visible coverage points.

> Trace: D86, D228
> Covers: Coverage observes the source language, not backend artifacts.

If a backend/target combination cannot provide conforming Kyokai-source coverage for the requested profile, the toolchain fails explicitly rather than emitting misleading coverage.

> Trace: D31, D80, D86
> Covers: Source coverage does not silently degrade.

## Reproducible Backend Artifacts

Generated C, LLVM IR, object files, `.koi` artifacts, final binaries, and libraries inherit Kyokai's reproducible build contract where they are normative build products for a selected mode.

> Trace: D83
> Covers: Backend output is part of the explicit build identity.

Backend output must not vary because of timestamps, filesystem traversal order, host locale, host timezone, random seeds, unstable hash iteration order, or unrelated environment state unless the selected output mode explicitly admits that input.

> Trace: D83
> Covers: Hidden host state cannot perturb reproducible artifacts.

Source paths in debug information are controlled by the selected build profile and target/toolchain contract. If a profile does not allow absolute paths, the toolchain must normalize or remap them so build location does not perturb artifacts.

> Trace: D27, D83
> Covers: Debug path behavior is part of reproducibility.

## Backend Failure Rules

If backend lowering cannot preserve Kyokai semantics, compilation fails. The diagnostic must name the construct, backend, target, and missing support contract when that information is available.

> Trace: D29, D31, D80, D228
> Covers: Backend rejection is explicit and diagnosable.

A backend may reject a program because the target lacks conforming atomics, an extern record has no ABI-supported passing convention, an inline assembly block cannot be lowered, a SIMD intrinsic is unavailable, debug/coverage mode cannot be honored, or the selected C toolchain contract cannot support required defined behavior.

> Trace: D22, D31, D80, D104, D139, D141, D228
> Covers: Unsupported backend features are build errors with named reasons.

A backend may not accept the program and then silently change Kyokai semantics, drop checked failure paths, lower atomics as volatile, treat raw pointers as safe references, discard required cleanup, discard source mapping in a profile that requires it, or depend on UB-sensitive host behavior.

> Trace: D73, D84, D139, D141, D228
> Covers: Backend acceptance commits to the language contract.

## Backend Conformance Obligations

A conforming backend test suite must include generated-code or runtime tests for:

| Area | Required evidence |
| --- | --- |
| Evaluation order | Side-effecting operands execute left-to-right. |
| Checked arithmetic | Overflow, invalid shifts, division by zero, and bounds checks produce TPOE. |
| Alias and movement | Moves, borrows, packed fields, and raw-address wrappers do not generate invalid aliasing behavior. |
| Layout | `record`, `extern record`, `packed record`, single-field wrappers, and layout introspection match target contracts. |
| FFI | Raw C imports/exports obey admitted type tables and reject linear/sum values by value. |
| Atomics | C11/LLVM atomic lowering preserves memory orders and rejects unsupported targets. |
| Volatile | Volatile operations are preserved and do not become synchronization. |
| Fatal paths | `panic`, TPOE, runtime-fatal, and stack overflow terminate through specified paths. |
| Debug/source maps | Debug and coverage outputs map to Kyokai source where profile requires it. |
| Reproducibility | Same declared inputs produce bit-identical normative artifacts. |

> Trace: D27, D31, D42, D73, D75, D83, D139, D141, D228
> Covers: Backend conformance tests target the places where backend folklore usually leaks in.
