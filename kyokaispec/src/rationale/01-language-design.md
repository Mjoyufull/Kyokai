# Language Design Rationale

[Rikona Kurasaki / Mjoyufull]
Kyokai starts from Austral but does not live inside Austral's shadow. It keeps the bones that still carry weight: linear resources, explicit modules, strict checking, capability security, and distrust of hidden runtime magic. Then it widens the house around those bones until the language, toolchain, stdlib, proof story, and audit story can all stand in one weather.

> Trace: D5, D86, D145, D152
> Covers: Kyokai is a hard fork with a self-contained spec family and a production systems-language scope.

## Fits In Head, But Not Small By Fear

[Rikona Kurasaki / Mjoyufull]
Austral's inherited simplicity goal still matters, but Kyokai cannot confuse simplicity with refusing to specify the world. A small language with an underspecified toolchain is not simpler for the person building real software. It only moves the complexity into tribal memory, build scripts, unspoken defaults, and library boundaries.

> Trace: D86, D147, D155
> Covers: Kyokai preserves simplicity as low mechanism overlap and explicit behavior, not as dependence on unwritten implementation custom.

Kyokai's version of fits-in-head simplicity has three tests. First, one semantic job should not have two competing mechanisms. Second, a boundary should be visible where the reader needs it: ownership, authority, failure, allocation, blocking, unsafe code, platform behavior. Third, the rule should be in the spec before programmers are expected to trust it.

> Trace: D85, D87, D155, D211
> Covers: The design favors visible semantic boundaries and public accepted rules over hidden effects.

## No Language-Level Undefined Behavior

[Rikona Kurasaki / Mjoyufull]
Undefined behavior is not a mystery; it is debt with a legal name. Kyokai's safe language does not get to say, after accepting a program, that a bad edge belongs to the compiler backend, the optimizer, or an old C habit. The language rejects the program, returns data, checks and terminates through a named path, or marks the operation unsafe.

> Trace: D73, D87, D139, D228, D253, D262
> Covers: Accepted safe Kyokai programs have specified behavior; unsafe and backend operations remain individually specified.

This is stricter than ordinary C-family practice and closer in spirit to the parts of Rust that make safe code meaningful. Kyokai also differs from Rust by refusing unwinding/destructor recovery as the normal contract-violation story. A bug that violates a contract does not become ordinary control flow merely because the runtime can walk the stack.

> Trace: D53, D84, D253
> Covers: Contract violation, panic, and runtime-fatal paths are distinct and not ordinary recoverable errors.

## Proof Honesty

[Rikona Kurasaki / Mjoyufull]
A language can speak like mathematics without having done the math. Kyokai avoids that pose. The spec may state the intended ownership and borrowing contract now, but before `v1.0` the sequential core needs a paper proof that names syntax, typing, operational semantics, and the theorem it is claiming.

> Trace: D143/D241
> Covers: Kyokai requires a paper proof for the sequential ownership-and-borrowing core before `v1.0` and later mechanization after self-hosting.

The first proof deliberately stays narrow. It covers the sequential core where ownership, borrowing, moves, regions, and TPOE can be made small enough to reason about cleanly. Concurrency, FFI, backend lowering, and the full stdlib are later layers, not excuses to delay the core theorem forever.

> Trace: D143/D241, D90, D245
> Covers: The first formalization excludes concurrency and FFI while leaving them as later proof extensions.

## Prior Art Pressure

[Rikona Kurasaki / Mjoyufull]
Austral gives Kyokai its closest prior art: linear types, capability direction, and a preference for explicitness. Ada and the Wirth line give Kyokai a readable statement-oriented surface. Rust proves that ownership can carry mainstream systems code, while also showing what Kyokai does not want: hidden destructor culture, broad trait-object escape hatches, and feature accretion that makes the mental model harder to hold. Zig shows the value of explicit toolchain thinking and comptime discipline, while Kyokai chooses a stronger type and authority story.

> Trace: D145, D147, D176-D177, D211, D229
> Covers: Kyokai borrows lessons from Austral, Ada, Rust, and Zig while keeping its own boundaries.

## The Design Bet

[Rikona Kurasaki / Mjoyufull]
The bet is simple and hard: make the programmer write the boundary once, then make the compiler and tools defend it every time after that. That is why Kyokai is strict about visible ownership, visible authority, visible failure, visible allocation, visible unsafe code, and visible platform contracts.

> Trace: D85, D155, D211, D229
> Covers: Kyokai's language design centers explicit semantic boundaries enforced by specification, compiler, and tooling.
