# Declarations

[Rikona Kurasaki / Mjoyufull]
Declarations are the names a module lets the rest of the program touch. They are not loose notes at the top of a file. A declaration says what exists, who may name it, what type it has, what contract it carries, and which body must later prove that promise true.

> Trace: D5, D17, D52, D78
> Covers: Kyokai declaration rules are part of the self-contained Kyokai spec and are checked across the `.kyo` interface and `.kai` body split.

A declaration is checked in three layers: syntax, visibility, and semantic well-formedness. Syntax decides whether the source form is a declaration at all. Visibility decides which modules may name it. Semantic well-formedness decides whether the declaration is legal after type checking, layout checking, contract checking, unsafe checking, and target selection.

> Trace: D17, D19, D19a, D52, D78, D155
> Covers: A declaration may parse successfully and still be rejected by later semantic checks; target guards, visibility, layout, contracts, and unsafe obligations are all compile-time declaration checks.

## Declaration Categories

A `.kyo` interface contains the importable declaration surface of the module. A `.kai` body contains implementations and private declarations for the same module. Interface declarations may be public or `internal`. Body-only declarations are private.

> Trace: D17, D52, D78, D79
> Covers: `.kyo` declarations form public or package-internal API, while `.kai` body-only declarations are private and cannot be imported or recorded as external API in `.koi` artifacts.

The legal interface declaration categories are constants, type aliases, opaque types, extern types, records, unions, capabilities, functions, typeclasses, instances, and generators. The legal body declaration categories are constant definitions, type aliases, extern types, records, unions, capabilities, function definitions, typeclasses, instance definitions, generator definitions, foreign blocks, and unsafe contracts. A body may also contain private helper declarations in these body-admitted categories.

> Trace: D17, D20, D20a, D20b, D52, D78, D198, D242, D242a, D245
> Covers: Interface files expose declarations; body files define implementations, private helpers, foreign blocks, and unsafe contracts.

`var` is not a module-level declaration. Kyokai has no user-defined mutable global variables, no safe `thread_local var`, and no hidden singleton state surface. Shared state must be represented by ordinary values, references, borrows, containers, capabilities, or unsafe contracts where a raw platform boundary is truly being used.

> Trace: D62, D114, D211, D245
> Covers: Module-level mutable variables are illegal; global or thread-local mutable state must be explicit in the value, capability, or unsafe boundary model.

A declaration may be guarded by `when expression`. A false declaration guard makes the declaration absent for the selected target before the module's visible API is finalized. A guard is not a runtime branch, not a statement, and not a way to make two incompatible declarations visible at the same time.

> Trace: D19, D19a, D117, D123
> Covers: Declaration-level `when` controls target selection at compile time; Kyokai rejects body-level target branching as the ordinary solution to platform variation.

## Constants

A constant declaration in a `.kyo` file gives a name and type without an initializer. The matching `.kai` body must provide exactly one compatible constant definition unless the declaration is eliminated by target selection. A constant definition gives the name, type, and initializer expression.

```kyokai
constant maxPathBytes: Index;

constant maxPathBytes: Index := 4096;
```

> Trace: D17, D52, D61, D78
> Covers: Interface constants declare importable names; body constants define values; foreign integer domains and bitflags may be modeled with aliases plus named constants.

A module-level constant is immutable. Its initializer must be pure enough for the constant-evaluation chapter to evaluate it without observing mutable ambient state, hidden I/O, allocation identity, task scheduling, foreign state, or time. If an initializer uses `comptime`, `static_assert`, other constants, or target descriptors, those forms are evaluated by the compile-time rules rather than by runtime module initialization order.

> Trace: D18, D18a, D19, D117, D202, D203, D215
> Covers: Constants are immutable and evaluated through the compile-time/constant model, not through an ambient runtime initialization sequence.

Module-level constants are forced lazily, cached after successful evaluation, and cycle-checked across module boundaries. Each constant is `Unforced`, `Evaluating`, or `Evaluated`. Forcing an already evaluating constant is a compile-time cycle error, and the diagnostic must show the dependency path that formed the cycle. Unrelated constants have no semantic order.

> Trace: D215
> Covers: Module constants are lazy, cached, cycle-checked, and unordered unless one constant depends on another.

## Type Aliases

A type alias is a transparent synonym for another type expression.

```kyokai
type alias FileDescriptor := Int32;
type alias IoResult[T: Type] := Result[T, IoError];
```

An alias introduces no nominal identity, no separate ABI identity, no new typeclass coherence identity, and no conversion boundary. After name resolution, alias expansion is part of ordinary type canonicalization. Cyclic aliases are illegal.

> Trace: D50, D61, D190
> Covers: `type alias` creates transparent synonyms only; aliases expand during type identity checking and cannot act as newtypes.

When a foreign C API exposes a fixed-representation integer domain or bitflag set, Kyokai models the raw ABI surface with an integer alias and named constants. This does not create a closed set. Safe wrappers that need a closed semantic domain must translate the raw integer into a union or nominal wrapper.

> Trace: D20, D20a, D61
> Covers: Foreign integer constants and bitflags use aliases plus constants at the raw boundary; closed Kyokai semantics require a wrapper type.

## Opaque And Extern Types

An opaque type declaration names a nominal type whose representation is not exposed at the declaration site.

```kyokai
type FileHandle: Linear;
type Token[T: Type]: Auto;
```

The declared universe marker is part of the type's public contract. A body may provide a concrete representation only in a way that satisfies the interface's opacity, universe classification, layout, and visibility rules. Client modules may name the opaque type, pass it, borrow it, store it where its universe allows, and call visible functions over it; they may not construct, deconstruct, inspect, or assume its representation unless another public declaration exposes that operation.

> Trace: D17, D52, D78, D190, D194, D195
> Covers: Opaque types are nominal public or internal names whose representation is hidden from clients while their universe contract remains visible.

An `extern type` names an opaque foreign type with unknown Kyokai size and layout. It may appear only behind FFI-admitted pointer/address forms or other specifically allowed ABI wrappers. It may not be passed by value, stored by value as an ordinary Kyokai object, destructured, measured with ordinary layout introspection, or pattern matched.

```kyokai
extern type FILE;
```

> Trace: D20, D20a, D20b, D242, D242a
> Covers: `extern type` is an opaque C-boundary type and may not cross raw FFI by value or gain guessed Kyokai layout.

## Records

A `record` declares a nominal product type with named fields. Field names are unique within the record. Field order is the source order used by layout, construction checking, pattern checking, and diagnostics. A record declaration may be ordinary, `extern`, or `packed`; the three layout classes are mutually exclusive.

```kyokai
record Point: Free is
    x: Int32;
    y: Int32;
build;

extern record Stat: Free is
    size: Nat64;
    mode: Nat32;
build;

packed record Header: Free is
    tag: Nat8;
    length: Nat16;
build;
```

> Trace: D35, D42, D116, D190
> Covers: Records are nominal named-field product types with one of exactly three layout classes: ordinary, `extern`, or `packed`.

Ordinary records use Kyokai layout. Fields stay in source order, each field is placed at the next offset satisfying its alignment, the record alignment is the maximum field alignment, and the total size is rounded up to that alignment. The compiler does not reorder fields, insert hidden niche optimizations, or infer a foreign ABI layout for an ordinary record.

> Trace: D42, D73, D139, D228
> Covers: Ordinary record layout is language-defined and cannot depend on backend folklore or C compiler undefined behavior.

`extern record` uses the selected target C ABI for the exact field list and source order. A record may cross raw `foreign "C"` by value only if it is an `extern record` and every field is FFI-admitted. Kyokai does not silently treat an ordinary record as a C struct.

> Trace: D20, D20a, D42, D242
> Covers: Raw by-value aggregate FFI requires `extern record`; ordinary records do not inherit C ABI layout.

`packed record` is byte-tight. Fields have no implicit padding between them, the record alignment is 1, and field access behaves as copy-in/copy-out through aligned temporaries when needed. Taking `&field` or `&!field` of a packed field is illegal because it could create a misaligned reference. Packed layout does not imply C compatibility, bitfields, or endian conversion.

> Trace: D42, D116
> Covers: Packed records are explicit byte-tight records with copy field access and no field borrowing.

A user-defined record must have at least one field. Phantom type parameters are allowed, but Kyokai does not use zero-field records or a general zero-sized-type culture as a marker system. A phantom parameter still participates in type identity, instantiation, and instance resolution.

> Trace: D181, D190, D214
> Covers: Phantom type parameters are legal on nominal records, but zero-field user records are illegal and phantom parameters still affect identity and coherence.

A one-line record declaration is sugar for an ordinary single-field record.

```kyokai
record UserId(value: Int64): Free;
```

It is exactly the single-field ordinary record mechanism, not a separate `newtype` declaration category. It has the same construction, field access, type identity, layout, and instance rules as the expanded block form.

> Trace: D109, D196
> Covers: Single-field records are Kyokai's nominal wrapper mechanism; the one-line form is declaration sugar only.

An ordinary single-field record is representation-transparent in Kyokai layout and Kyokai calling convention: size, alignment, and offset match the single field. The wrapper remains a distinct nominal type. Representation equality does not create implicit conversion, alias identity, or shared typeclass instances. The guarantee does not apply to `extern record` or `packed record`.

> Trace: D109, D190, D196
> Covers: Ordinary single-field records are zero-cost wrappers while remaining nominally distinct.

## Unions

A `union` declares a nominal tagged sum type. Each variant has a constructor name and either no payload, one unnamed payload type, or named fields.

```kyokai
union Optional[T: Type]: Auto is
    case None;
    case Some(T);
build;

union Color: Free is
    case RGB is
        red: Nat8;
        green: Nat8;
        blue: Nat8;
    case Greyscale(Nat8);
build;
```

> Trace: D24, D47, D54, D65, D190, D194
> Covers: Unions are nominal tagged sums with explicit zero-field, one-field, and named-field variant forms; tuples are not created by variant payloads.

Variant constructors are ordinary construction surfaces governed by the expression chapter. A zero-payload variant is written as the bare constructor. A one-payload variant is written with parentheses. A named-field variant is written with braces. Field order inside a named-field variant is semantically irrelevant for construction, but expressions evaluate in source order.

> Trace: D65, D71
> Covers: Union construction uses the three explicit variant construction forms and source-order evaluation for supplied expressions.

A union is exhaustive at pattern sites. Adding or removing a visible variant changes the set of patterns that exhaust the type, and clients must update matching code accordingly unless the type is opaque to them.

> Trace: D38, D205, D206
> Covers: Union variants drive structural pattern exhaustiveness, with no pattern guards and no hidden discard of linear payloads.

## Capabilities

A capability declaration creates a sealed nominal authority type.

```kyokai
capability FileSystemCapability;
```

Capability values cannot be forged by constructors, raw memory tricks, imports, unsafe module pragmas, or name visibility. They enter a program only through language-defined authority roots, standard-library authority-splitting functions, or audited unsafe contracts that state the trusted acquisition boundary.

> Trace: D20, D211, D245, D255
> Covers: Capability declarations create sealed unforgeable authority types; unsafe code must not fabricate authority without an audited trusted boundary.

Capabilities are ordinary values with respect to type checking, borrowing, passing, and linearity, but their constructors are not ordinary declarations. If a capability is `Linear`, it follows exact-use rules. If a wrapper exposes a narrower capability, the wrapper must state what authority is split, retained, or consumed.

> Trace: D195, D211, D255
> Covers: Capabilities participate in ordinary type and ownership rules while remaining sealed authority tokens.

## Functions And Contracts

A function declaration gives a signature without a body. A function definition gives the same signature and a body. A body must satisfy its interface declaration exactly where the signature is fixed, and may not weaken the interface contracts visible to callers.

```kyokai
function divide(a: Int32, b: Int32): Int32
    require b != 0;
    ensure result * b = a;
;

function divide(a: Int32, b: Int32): Int32
    require b != 0;
    ensure result * b = a;
is
    return a / b;
qed;
```

> Trace: D17, D52, D53, D78, D140, D142
> Covers: Function declarations live in interfaces, definitions live in bodies, and interface-visible contracts are part of the signature clients check against.

Parameters are immutable bindings unless the parameter type itself is a mutable reference such as `&![T]`. A parameter name is in scope for later parameter default-free type checking only where the grammar and contract chapters allow it; parameter names may not shadow any still-live binding.

> Trace: D14, D60, D187, D195
> Covers: Function parameters are ordinary bindings with explicit mutability through reference types and no shadowing.

`require` clauses are Boolean preconditions evaluated at function entry. `ensure` clauses are Boolean postconditions evaluated after the body produces a return value and before that value is delivered to the caller. Contract expressions must be pure, may not mutate state, may not consume linear values, and use ordinary Kyokai runtime semantics including arithmetic trap behavior. A failed contract terminates the program.

> Trace: D53, D75, D84, D140, D142
> Covers: Function contracts are always checked runtime value contracts; failure is TPOE and contract expressions are observation-only.

Inside an `ensure` clause for a non-`Unit` function, `result` is a contextual name for a read-only view of the produced return value. It is not a second owned value. `old expr` is legal only inside `ensure`, is evaluated once at function entry, and may snapshot only pure expressions over `Free` entry-state data.

> Trace: D125, D129, D140, D142, D195
> Covers: `result` and `old` are scoped contract mechanisms with no ownership escape or linear copying.

A function returning `Unit` may complete at the end of its body without writing `return nil;`. No other return value is implicit. An explicit `return expr;` exits the function only after all linear, borrow, defer, and errdefer obligations on that path are satisfied by the later control-flow and ownership chapters.

> Trace: D2, D8, D87, D246
> Covers: Only `Unit` fallthrough is implicit, and function exits remain subject to cleanup and linear-use rules.

## Typeclasses, Instances, And Generators

A typeclass declaration defines a static contract over type parameters. It may contain required methods, default method bodies, and associated type declarations. A typeclass closes with `spec;` because it is a contract surface, not a runtime block.

> Trace: D33, D82, D182, D195
> Covers: Typeclasses are static contracts with methods, default methods, and associated types; they do not imply runtime dictionaries.

An instance declaration in an interface exposes that an instance exists. An instance definition in a body supplies associated types and method bodies. Every required method and every associated type must be supplied exactly once unless a method has a default body. Associated types do not have defaults.

> Trace: D33, D82, D182, D214
> Covers: Instances provide static witnesses, must fill associated types, and may rely on or override default method bodies.

Generators are declarations that create named stackless pull iterators. A generator definition may use `yield` inside its body. The generated iterator type is nominal and linear when it owns suspended state; destruction of suspended state is explicit and checked by the generator and ownership chapters.

> Trace: D32, D193, D198, D249
> Covers: Generators are named stackless pull iterators, not general coroutines, async functions, or opaque return types.

## Foreign Blocks And Unsafe Contracts

A `foreign "C" is ... mon;` block declares raw foreign functions and constants for the selected target C ABI. Such blocks are legal only in a module body marked with `pragma Unsafe_Module;`. Raw foreign declarations are private unsafe machinery unless the module exposes safe wrappers through ordinary interface declarations.

> Trace: D20, D20a, D20b, D127, D242, D242a
> Covers: Raw C FFI declarations live in visible foreign blocks inside unsafe module bodies and do not bypass normal visibility.

Every raw foreign call in Kyokai source is treated as if it has an additional leading `&![UnsafeCapability]` parameter. This authority argument is erased from the actual C ABI lowering but remains part of Kyokai's source-level audit and call contract. Raw foreign declarations may not take or return Kyokai linear values or sum types by value.

> Trace: D20, D20a, D20b, D242, D242a, D245
> Covers: Raw FFI requires explicit unsafe authority, exact ABI-admitted types, and no implicit linear or sum-type crossing.

An unsafe module must contain `unsafe contract ... audit;` blocks that enumerate the unsafe operations used by the module and state the assumptions, preserved invariants, failure mappings, wrapper exports, and trusted capability boundaries involved. The compiler rejects unsafe operations not covered by an unsafe contract entry.

> Trace: D20, D245, D255
> Covers: Unsafe contracts are source-level audit declarations tied to actual unsafe operations and cannot be replaced by comments or external prose.

## Compile-Time Declarations And Assertions

`static_assert(expr, "message")` is a compile-time assertion. When it appears in a declaration initializer, contract, guard, or other compile-time-admitted context, the assertion expression must evaluate to `Bool` at compile time. A false result is a compile-time error with the supplied message.

> Trace: D18, D18a, D202, D203
> Covers: Static assertions are compile-time checks, not runtime assertions and not optional debug checks.

A declaration that depends on a `comptime` result must receive a self-contained `Free` value from the compile-time evaluator. Runtime resources, allocator identity, task identity, foreign state, and linear owning values cannot escape into a compile-time declaration value.

> Trace: D18, D18a, D202, D203, D215
> Covers: Compile-time declaration values are deterministic `Free` values and cannot smuggle runtime resources into static program shape.
