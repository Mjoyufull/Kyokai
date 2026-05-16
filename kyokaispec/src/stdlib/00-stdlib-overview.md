# Standard Library Overview

[Rikona Kurasaki / Mjoyufull]
Kyokai's standard library is not a decorative appendix to the language. It is the place where ownership, allocation, authority, failure, platform behavior, and ordinary systems work meet. A language can be strict in the compiler and still become vague in the library; Kyokai does not get to do that.

> Trace: D85, D152, D229, D243
> Covers: The Kyokai standard library is a committed systems-library surface with explicit contracts and compatibility policy.

The public namespace for the standard library is `Kyokai.*`. Public Kyokai APIs must not expose inherited `Standard.*` or `Austral.*` names as their canonical surface. Compatibility shims, migration guides, or historical examples may mention inherited names, but stable Kyokai programs import `Kyokai.*` modules.

> Trace: D1, D5, D152
> Covers: The stdlib namespace follows Kyokai project identity and retires inherited public naming.

The standard library is batteries-included for systems programming. The admitted surface includes core result/optional/error/display protocols, allocators, memory containers, byte/text/path types, collections, iterators, numerics, I/O, filesystems, environment access, process control, time, random, concurrency primitives, testing helpers, crypto policy, and tracked transitional FFI.

> Trace: D152, D229-D232
> Covers: Kyokai commits to broad systems-library families while still requiring admission criteria per API.

## Authority And Import Model

The standard library does not receive ambient authority. File, environment, process, terminal, network, time, random, signal, and OS-specific operations require explicit capabilities. Pure modules may be imported without authority, but importing a module never grants runtime permission to use authority-bearing operations.

> Trace: D67, D85, D211
> Covers: Stdlib authority follows capability flow, not imports or module names.

Every stdlib API must state whether it is pure, capability-requiring, unsafe-internal, FFI-backed, platform-specific, blocking, allocation-using, or fatal-capable. These fields are part of the public contract and are extracted by documentation and audit tooling.

> Trace: D85, D150, D218, D229
> Covers: Documentation and audit can surface stdlib semantic fields consistently.

## Contract Fields

Every public stdlib type, function, typeclass, method, and instance documents the following fields. A field that does not apply is written as `N/A`; it is not omitted.

| Field | Required Meaning | Trace |
| --- | --- | --- |
| Ownership | Whether each argument is consumed, borrowed immutably, borrowed mutably, stored, returned, or invalidated. | D11b, D77, D85 |
| Allocation | Whether the API allocates, which allocator it uses, whether allocation can fail, and whether the result stores allocator identity. | D44, D74, D201 |
| Failure | All `Result`, `Optional`, TPOE, `panic`, runtime-fatal, and impossible-failure cases. | D53, D74, D84, D85 |
| Capabilities | Required capability values, derivation source, and whether authority is borrowed, consumed, or split. | D67, D85, D211 |
| Linearity | Whether returned or stored values are `Linear`, how they are destroyed, and what happens on early exit. | D2, D77, D85 |
| Concurrency | Task-transfer status, synchronization behavior, blocking behavior, memory-order needs, and thread-affinity. | D3, D90-D95, D100-D101, D212 |
| Platform | Target support, OS-specific behavior, path/encoding caveats, and unsupported-target failures. | D80, D85, D149 |
| Determinism | Iteration order, randomization, hash seeding, clock dependence, locale dependence, and reproducibility effect. | D83, D85 |
| Complexity | Time and space complexity, amortized behavior, and worst-case notes where relevant. | D85, D229 |
| Edge Cases | Empty input, zero length, invalid encodings, invalid paths, closed handles, allocation edge cases, and boundary values. | D74, D85, D229-D232 |
| Tests | Required unit tests, property tests, fuzz tests, oracle/reference vectors, or cross-platform tests. | D220, D229-D232 |
| Implementation | Safe native Kyokai, unsafe internal, permanent FFI boundary, transitional FFI, or externally reviewed implementation. | D229-D231 |
| Compatibility | SemVer and edition impact, deprecation path, and whether the API is stable, experimental, or compatibility-only. | D157, D223, D243 |

> Trace: D11b, D44, D53, D67, D74, D77, D80, D83-D85, D90-D95, D100-D101, D150, D157, D201, D211-D212, D220, D223, D229-D232, D243
> Covers: The common stdlib contract table is mandatory and covers ownership, allocation, failure, authority, linearity, concurrency, platform, determinism, complexity, edge cases, tests, implementation, and compatibility.

## Module Status

Each stdlib module has a release status: `stable`, `experimental`, `compatibility`, `transitional`, or `internal`. Stable modules must satisfy the admission criteria. Experimental modules are public but may change under documented SemVer/edition policy. Compatibility modules exist for legacy algorithms or migration and must not be presented as preferred defaults. Transitional modules expose temporary bootstrap or FFI-backed behavior with replacement criteria. Internal modules are not public API.

> Trace: D152, D157, D223, D229-D230, D243
> Covers: Stdlib module status controls admission, compatibility, and deprecation promises.

A stable module cannot depend on an internal module's undocumented behavior. If a stable module uses unsafe or FFI internally, its public contract still carries the complete safe behavior and the audit surface identifies the trust boundary.

> Trace: D20, D85, D150, D229-D230, D245
> Covers: Stable safe APIs may use unsafe internals only through documented safe contracts and audit metadata.

## Implementation Policy

Pure computation is written in safe native Kyokai unless a stdlib admission record gives a stronger reason. Sorting, hashing, text parsing, formatting, integer helpers, containers, encoders, decoders, and ordinary math algorithms are not wrapped through C merely because C code exists.

> Trace: D229-D230
> Covers: Rewrite-It-In-Kyokai is the default for pure computation.

FFI is legitimate for OS/hardware boundaries, transitional bootstrap bridges, externally reviewed libraries, and domains where Kyokai requires more evidence than a fresh native rewrite can currently provide. Every FFI-backed public API requires an unsafe contract and a safe wrapper unless the API itself is explicitly unsafe.

> Trace: D20, D230-D231, D245
> Covers: FFI is allowed only at explicit trust boundaries with contracts and wrappers.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
A standard library can become a second language hiding under function names. Kyokai refuses that. The function name should tell you the ownership story. The signature should show the allocator and capability story. The contract should tell you how it fails. The docs should say what changes across platforms. No one should have to read a source file in the rain to find out whether `push` can allocate, whether an iterator dies after mutation, or whether a file call secretly uses the current directory.

> Trace: D85, D152, D229
> Covers: Kyokai's stdlib contract exists so library behavior stays as explicit as language behavior.
