# Built-ins

[Rikona Kurasaki / Mjoyufull]
Built-ins are the pieces of Kyokai that are so close to the checker that pretending they came from a hidden module would be lying. They are the floorboards: `Bool`, `Unit`, `Never`, fixed integers, `Result`, `Optional`, target facts, borrow forms, raw pointer forms, and the small set of compiler-known protocols that syntax lowers through.

This chapter names that floor. Nothing here creates a prelude. Nothing here grants ambient authority. A programmer should not have to wonder whether a name appeared because a secret import drifted in through the back door.

> Trace: D24, D87, D147, D155, D214
> Covers: Kyokai built-ins are language-level names, protected constructor words, and compiler-known protocols rather than hidden prelude imports.

Kyokai keeps compatible Austral ideas where they still carry their weight, but it does not keep Austral's pervasive module model. `Austral.Pervasive.Option` becomes language-level `Optional`. `Austral.Pervasive.Either` does not become a language primitive. `Austral.Pervasive.abort` is replaced by the specified `panic`, TPOE, and runtime-fatal surfaces. `argumentCount()` and `nthArgument()` are replaced by startup passing `args` into the entrypoint. `Austral.Memory`-style low-level memory helpers move behind explicit unsafe, allocator, capability, layout, and FFI contracts.

> Trace: D5, D20, D24, D48/D162, D84, D147
> Covers: The Kyokai spec carries the useful inherited Austral concepts while replacing hidden pervasive imports, ambient arguments, and unspecific abort behavior with explicit language and startup rules.

## Built-in Name Classes

A language built-in name is available in every source file without an import. It cannot be introduced by a declaration, local binding, pattern binding, parameter, import alias, selective import, typeclass method, associated type, instance item, or generated name in a scope where it would shadow or collide with the built-in meaning.

> Trace: D24, D60, D179, D214
> Covers: Protected built-in names do not participate in ordinary shadowing or import-collision repair.

The protected constructor words `Ok`, `Err`, `Some`, and `None` are not ordinary user constructors. They are reserved by the language for the built-in `Result` and `Optional` families. `true`, `false`, and `nil` are likewise reserved for `Bool` and `Unit`. A user union may not declare variants with these names, and a user declaration may not bind them.

> Trace: D24, D58, D194, D214
> Covers: Built-in value constructors and literal values are protected syntax words, not names borrowed from a prelude module.

Some names are language built-ins but not syntax keywords. `Unit`, `Bool`, `Never`, the numeric types, `Index`, `String`, `StaticString`, `Result`, `Optional`, `target`, `Os`, `Arch`, `Abi`, and `Endian` are resolved before imports. They remain ordinary denotable type or value names in error messages, documentation, and interface artifacts, but source code cannot redefine or hide them.

> Trace: D24, D120, D190, D194, D214
> Covers: Built-in type and target names have stable language identity even though their spelling is used as source names.

Compiler-known protocol names are not all language built-ins. A typeclass such as `Displayable`, `Writable`, `Indexable`, `IndexableMut`, `Parsable`, or `StandardError` may live in a standard-library module, but the language can still know that a particular syntax form or built-in operation requires that protocol. Such a protocol must still be imported or qualified where ordinary source code names it directly, unless its own chapter declares it language-level.

> Trace: D23, D36/D106/D132, D40/D40a/D102, D66, D69, D259
> Covers: Syntax can lower through fixed protocols without turning every protocol name into a hidden prelude import.

## Core Value Types

| Built-in | Kind | Universe | Values or construction |
| --- | --- | --- | --- |
| `Unit` | Language type | `Free` | Single value `nil`. |
| `Bool` | Language type | `Free` | Values `true` and `false`. |
| `Never` | Language type | `Free` | No values; statically diverging expressions have this type. |
| `StaticString` | Language type | `Free` | Produced by `static "..."` and eligible compile-time text operations. |
| `String` | Standard owning text type recognized by language rules | `Linear` | Produced by ordinary string literals and explicit allocator-taking conversions. |

> Trace: D24, D30/D30a, D54, D58/D191, D120, D194
> Covers: Core built-in value types are closed, denotable, and classified by explicit universe rules.

`Never` is not a panic flavor and not a possible runtime value. It is a type for source positions that the checker can prove do not complete normally, such as `return`, `break`, `continue`, `panic`, `todo`, and `unreachable` in the contexts that admit expression joining. A possible integer overflow, bounds failure, or contract failure inside an ordinary expression does not make that expression `Never`.

> Trace: D58/D191, D73, D84, D121
> Covers: Static divergence and possible runtime failure remain separate.

`Never` has only expression-site coercion. It can satisfy an immediate expected type because it cannot produce the wrong value, but it does not create subtyping, variance, inheritance, or implicit lifting through arbitrary generic constructors. The only generic lifting attached to `Never` is the closed `Optional` and `Result` table named in the type-system chapter.

> Trace: D58/D191, D186, D194
> Covers: Bottom typing stays narrow and does not infect the rest of the type system.

## Numeric Built-ins

Kyokai's integer built-ins are `Int8`, `Int16`, `Int32`, `Int64`, `Nat8`, `Nat16`, `Nat32`, `Nat64`, and `Index`. Fixed-width types have exactly the width in their names. `Index` is the target-selected size and indexing domain, but it is still a distinct concrete type, not a quiet alias for whatever C type happens to be nearby.

> Trace: D75, D210, D261
> Covers: Integer types are exact, same-type, and explicit, with `Index` as its own domain.

Integer arithmetic operators require the same concrete integer type on both operands after contextual literal inference and explicit conversions. There are no usual arithmetic conversions, no signed/unsigned mixing rule, and no promotion to `Int32` or target word size. Checked overflow, invalid division, invalid remainder, and invalid shift or rotate counts trigger TPOE.

> Trace: D12, D41, D75, D84, D210
> Covers: Integer arithmetic has specified Kyokai failure behavior and no C-style promotion folklore.

The floating built-ins are `Float32` and `Float64`. Their ordinary operations use strict IEEE 754 binary32 and binary64 semantics with round-to-nearest, ties-to-even, signed zero, infinities, subnormals, and NaNs as specified by the numeric chapter. Backend fast-math, excess precision, reassociation, and hidden FMA contraction cannot change language-visible results.

> Trace: D76, D139, D228
> Covers: Floating behavior is part of Kyokai semantics, not a backend profile accident.

Numeric literal suffixes are exactly `i8`, `i16`, `i32`, `i64`, `n8`, `n16`, `n32`, `n64`, `index`, `f32`, and `f64`. A suffixed literal has that type before context. Literal suffixes do not imply a cast, a promotion, or a family of C-style suffix combinations.

> Trace: D12, D135/D261
> Covers: Literal suffixes are a closed built-in typing surface.

Default numeric conversions use named APIs such as `toInt32`, `toNat64`, `toIndex`, and `toFloat64`. They are not cast syntax and not type constructors. Checked default conversions reject or TPOE on invalid narrowing according to the numeric conversion contract; alternate policies such as wrapping, saturating, truncating, and fallible conversion use separately named APIs.

> Trace: D37, D75, D76, D210
> Covers: Numeric conversion is explicit, named, and policy-specific.

## Text, Bytes, And Foreign Strings

Ordinary string literals produce `String`, a nominal linear UTF-8 text owner. `String` stores pointer, length, capacity, and allocator identity according to the string and allocator contracts. It has no small-string optimization and does not become `StaticString` merely because a context asks for one.

> Trace: D30, D30a, D44, D120, D194
> Covers: Runtime text is owning, linear, allocator-aware UTF-8 data with no contextual static retargeting.

`StaticString` is the compile-time text type produced by `static "..."`. It is `Free`, compiler-managed or embedded read-only text. Converting it into a runtime `String` is an explicit allocator-taking operation that can fail with allocation failure.

> Trace: D120, D40, D44, D74
> Covers: Compile-time text and runtime owned text meet only through explicit conversion.

Byte storage is not text. `Nat8`, byte literals, byte spans, `Buffer[Nat8]`, and byte-oriented I/O APIs do not implicitly become `String`. Text validation, decoding, encoding, and ownership must be named by APIs with failure contracts.

> Trace: D30a, D54, D66, D85
> Covers: Bytes and UTF-8 text stay separated at the type and API boundary.

C string interop uses dedicated `CString` and `CStr` standard types. They are not aliases for `Pointer[Nat8]`. Conversion into or out of them validates nul termination, interior nul rules, lifetime, ownership, and allocator behavior in the API contract.

> Trace: D20/D20a, D30a, D68, D242
> Covers: Foreign string boundaries use explicit safe wrappers instead of naked pointer folklore.

## Result And Optional

`Optional[T]` is the language-level absence family. Its constructors are `Some(value)` and `None`. `None` carries no payload. It is not null, not a pointer sentinel, and not a value every type can secretly hold.

> Trace: D24, D58/D191, D194
> Covers: Optional absence is explicit sum data with protected built-in constructors.

`Result[T, E]` is the language-level recoverable-failure family. Its constructors are `Ok(value)` and `Err(error)`. `Result` is used by `let...else`, `or return`, `or break`, `or continue`, allocator failure, I/O failure, parsing failure, spawn failure, and other ordinary recoverable edges.

> Trace: D15/D15a, D24, D74, D235
> Covers: Recoverable failure is ordinary data, and language error-propagation sugar targets `Result` exactly.

`Optional` and `Result` are `Auto`: they are `Linear` when any payload type is linear and `Free` only when all payloads are free. Pattern matching, `let...else`, and result propagation do not gain permission to discard linear payloads.

> Trace: D24, D98, D194, D195, D206
> Covers: Built-in sum families preserve linear ownership of their payloads.

`Result` and `Optional` replace Austral's pervasive `Option` for core control flow. `Either` is not a language built-in because Kyokai does not need a second general-purpose two-sided sum in the core language. Programs can declare domain-specific unions or use ordinary library types when the shape is not recoverable failure or absence.

> Trace: D5, D24, D47, D147
> Covers: Kyokai keeps the sum shapes the language itself uses and leaves broader sum modeling to named user or library types.

## Target Built-ins

`target` is a compile-time constant record with fields `target.os`, `target.arch`, `target.abi`, and `target.endianness`. Their types are the built-in enums `Os`, `Arch`, `Abi`, and `Endian`. They are values for compile-time selection, static assertions, and declaration-level `when` guards.

> Trace: D18/D18a, D19/D19a, D24, D117
> Covers: Target facts are typed compile-time data, not strings or build-script rumors.

The initial target vocabulary is closed by the target contract, not by wishful cartesian product. The enum variants include hosted and freestanding operating-system families, primary architecture families, ABI environment families, and explicit endian values. Adding variants later is a compatibility event governed by the toolchain and edition rules.

> Trace: D19, D80, D105, D117
> Covers: Target enum existence and target support are separate contracts.

`target` may be used in constant evaluation and declaration-level `when` guards. Kyokai does not have body-level target branching. Platform differences belong in selected body files, declaration guards, or ordinary typeclass abstraction.

> Trace: D19/D19a, D123
> Covers: Target selection stays at module, declaration, and abstraction boundaries.

## Arrays, Views, Pointers, And Function Pointers

`Array[T, N]` is the fixed-length array family with `N: Index`. Array literals infer length only by counting elements. Element type inference follows ordinary literal and expression rules; the compiler does not solve arbitrary target types from an empty array literal.

> Trace: D55, D159, D188, D194
> Covers: Fixed arrays have explicit `Index` const lengths and narrow literal inference.

Borrow forms `&[T]` and `&![T]` are language type forms. They are always `Free` because they are access, not ownership. Copying the borrow value does not copy the referent, and the borrow checker still controls aliasing, lifetime, reborrow, mutation, and escape.

> Trace: D6, D14, D187, D194, D238-D240
> Covers: Borrow types are built-in non-owning access values governed by the borrow checker.

`Address[T]` and `Pointer[T]` are raw non-owning address forms. They are always `Free`. Safe code may carry them only as raw values; operations that treat them as valid storage, create safe borrows, read, write, or transfer ownership require unsafe modules and unsafe contracts.

> Trace: D20/D20a, D73, D194, D242, D245
> Covers: Raw addresses have no safe validity magic.

`FnPtr(...) : Ret` is the bare environmentless function pointer form. It is always `Free` and is separate from closures, callable values, and callback wrappers that carry environment or ownership. Raw FFI callbacks use `FnPtr` only under the ABI and lifetime rules of the unsafe chapter.

> Trace: D20/D20a, D21, D118, D126, D194
> Covers: Function pointers are explicit callable addresses, not hidden closure values.

`Vector[T, N]` is the portable SIMD vector form. Its element domain is fixed-width integers, fixed-width floats, and Boolean masks admitted by the vector contract. Portable operations have lane-wise semantics; target-specific intrinsics are gated by target support and must fail compilation if unavailable rather than silently scalarizing.

> Trace: D104, D139, D194, D228
> Covers: Vector behavior is portable where declared and target-gated where intrinsic-specific.

## Compiler-known Protocols

Indexing syntax `a[i]` lowers through the fixed `Indexable` protocol family. Mutable indexing and indexed assignment lower through `IndexableMut`. The operation is total-or-TPOE: out-of-domain indexing is a contract violation, not `Optional`, not `Result`, and not a sentinel.

> Trace: D36/D106/D132, D84
> Covers: `[]` has one protocol path and one checked failure policy.

Slice syntax `a[i..j]`, `a[i..]`, `a[..j]`, and `a[..]` is built-in half-open span extraction over admitted sequential containers. Bounds must satisfy `i <= j <= length(a)`. `String` is not directly indexable or sliceable by this syntax; text operations must name their Unicode and byte behavior explicitly.

> Trace: D30/D30a, D106, D132, D210
> Covers: Slicing is checked and half-open, while text indexing remains explicit.

Operator overloading exists only through a fixed built-in operator typeclass family. The family covers admitted arithmetic, comparison, equality, concatenation, and similar standard operator meanings. Users cannot declare new operator tokens, change precedence, overload field access, overload assignment, overload borrow syntax, or replace Boolean short-circuiting.

> Trace: D23, D41, D56/D57, D82, D214
> Covers: Operator dispatch is closed, static, and coherent.

Formatting uses `Displayable` and `FormatSink`. `format(alloc, template, args...)` is a built-in checked formatting construct that returns `Result[String, AllocError]`. `writeFmt(writer, template, args...)` writes to a `Writable` sink with the same placeholder language and returns an explicit I/O result. The format string uses `{}` placeholders only in this accepted surface.

> Trace: D40, D40a, D44, D66, D74, D102
> Covers: Formatting is compile-time checked, allocator-explicit when allocating, and stream-fallible when writing.

Parsing uses `Parsable[T]` where the relevant standard API admits it. Parsing returns `Result[T, ParseError]`; it does not use exceptions, sentinel values, or ambient locale. Standard numeric parsers must state accepted syntax, range behavior, and failure categories.

> Trace: D69, D75/D76, D85
> Covers: String-to-value conversion is explicit recoverable parsing data.

`StandardError` is an optional diagnostic protocol for error values. It does not inherit from `Displayable`, does not force every error to have a source chain, and does not make all errors printable by default. APIs that surface diagnostics must say whether they require `Displayable`, `StandardError`, both, or neither.

> Trace: D259, D40
> Covers: Error diagnostics are a separate protocol from ordinary display formatting.

## Compile-time Built-ins

`comptime expr` is the language's call-site compile-time evaluation form. It may evaluate only deterministic, host-independent, eligible `Free` computations. It may not observe filesystem state, environment variables, wall-clock time, locale, randomness, network state, process state, task scheduling, runtime capabilities, or FFI.

> Trace: D18/D18a, D202/D203
> Covers: Compile-time execution is deterministic and cannot smuggle host state into source semantics.

`static_assert(condition, "message")` is a compile-time check. A false condition is a compile-time error carrying the supplied message. It is not a runtime assertion and cannot be stripped by build profile.

> Trace: D18/D18a, D202/D203
> Covers: Static assertions are compile-time-only obligations.

`sizeOf(T)`, `alignOf(T)`, and `offsetOf(T, field)` are compile-time layout introspection built-ins. They follow ordinary Kyokai layout, extern C ABI layout, or packed layout according to the declared layout class. They are illegal for by-value opaque `extern type` because that layout is unknown.

> Trace: D18/D18a, D20a, D42, D80
> Covers: Layout introspection is deterministic and layout-class-aware.

## Concurrency Built-ins And Recognized Types

`Atomic[T]`, `MemoryOrder`, and `CompareExchangeResult[T]` are part of the accepted atomic model. `Atomic[T]` is linear storage and one of the closed safe shared-access forms. Atomic operations require explicit memory-order arguments where the operation semantics need them.

> Trace: D3b, D90/D90a/D247, D141, D194
> Covers: Atomics are explicit synchronization objects, not volatile fields or general interior mutability.

`Mutex[T]` and `RwLock[T]` are linear synchronization primitives with linear scoped guards. Locking through an immutable borrow is allowed only because the lock type is in the closed synchronized set; Kyokai does not generalize that into arbitrary shared mutation.

> Trace: D100/D184, D194, D248
> Covers: Locks are admitted shared-state primitives without creating general interior mutability.

`Sender[T]` and `Receiver[T]` are linear SPSC channel endpoints. Channel topology, capacity, blocking behavior, close behavior, linear payload drainage, and happens-before edges are specified in the concurrency chapter. Cloneable endpoints, MPSC, MPMC, broadcast channels, and hidden endpoint sharing are not language built-ins.

> Trace: D3a, D90/D90a, D101, D146, D183, D236
> Covers: Channel endpoints are unique linear values with explicit topology and transfer semantics.

## Unsafe And Authority Built-ins

`RootCapability` is startup-minted authority. It is not a pervasive global value. A hosted entrypoint receives it only when the entrypoint signature asks for it, and safe code cannot acquire it later by import, constructor, or unsafe representation trick.

> Trace: D48/D162, D211, D255
> Covers: Root authority enters source through the entrypoint contract only.

`UnsafeCapability` is explicit authority to perform unsafe operations under unsafe contracts. It is not root authority and does not grant filesystem, network, clock, process, terminal, dynamic-loader, signal, or device authority by itself.

> Trace: D20/D20a/D20b, D245, D255
> Covers: Unsafe authority is narrow and contract-bound.

Volatile access, inline assembly, raw FFI calls, raw dynamic loading, and raw pointer-to-borrow conversion are not vague escape hatches. Each one is an individually specified unsafe primitive family with preconditions, postconditions, backend constraints, authority requirements, and audit obligations.

> Trace: D20, D22, D73, D94/D257, D113a/D113b, D242/D242a/D245
> Covers: Unsafe built-ins remain specified even when they step outside safe proof.

## Standard Built-ins That Are Not Language Names

`ExitCode` is a standard-library built-in startup union, not a raw integer and not a language primitive. Hosted `main` returns it, and the runtime maps it to the host process status. The ordinary language does not treat every integer as an exit code.

> Trace: D48/D162, D84
> Covers: Program exit is a standard startup contract with explicit result data.

`Span[T]`, `Buffer[T]`, allocator handles, `Box[T]`, `PinBox[T]`, file handles, sockets, paths, OS strings, pollers, streams, locks, channels, and ordinary containers are named standard-library or special-form types with explicit imports and contracts unless a preceding section says their spelling is language-level. They do not arrive through a prelude.

> Trace: D44, D66, D77, D85, D89a/D89b, D93, D152, D229
> Covers: The batteries-included standard library is explicit even when the language recognizes some of its contracts.

## Rejected Built-ins

Kyokai has no hidden prelude, no wildcard built-in module import, no tuple type, no tuple literal, no tuple destructuring, no `Rc`, no `Arc`, no garbage collector, no exceptions, no `try/catch`, no in-process panic catch, no in-process TPOE catch, no `async`/`await`, no hidden executor, no hidden default allocator, no ambient command-line argument functions, and no implicit environment access.

> Trace: D47/D131, D67, D84/D253, D101, D147, D156, D177, D250-D251
> Covers: Rejected conveniences remain absent instead of half-specified.

A future feature may add a built-in only by updating the public language spec, its traceability entry, and its implementation tests. It cannot arrive by standard-library naming convention, backend convenience, or compiler-source drift.

> Trace: D155, D229
> Covers: Built-in status is governed, public, and testable.
