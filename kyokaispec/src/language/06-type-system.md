# Type System

[Rikona Kurasaki / Mjoyufull]
Kyokai keeps Austral's hard center: the type system is not decoration around memory management. It is the first wall between ordinary values, resource-owning values, borrowed access, and authority. The wall has to be visible in source, because a hidden ownership rule is just another way to make the reader guess where the trapdoor is.

> Trace: D5, D14, D73, D143/D241, D195
> Covers: Kyokai inherits Austral's linear type foundation while stating the Kyokai universe, borrow, and no-UB rules directly in this spec.

A type is a compile-time classification of values. Every value expression has exactly one static type after elaboration. Every type belongs to exactly one universe for the purpose of copy, discard, move, and linear-use checking: `Free` or `Linear`. `Type` and `Auto` are source markers used to write generic constraints and declaration classification rules; they are not extra runtime universes for values.

> Trace: D190, D194, D195
> Covers: Runtime value types classify as `Free` or `Linear`; `Type` and `Auto` are source-level type-system markers with defined roles.

## Universes

`Free` types are unrestricted. A value of a `Free` type may be copied, discarded, passed, returned, stored, pattern-discarded, and overwritten subject to ordinary type, borrow, and control-flow rules.

> Trace: D195, D206
> Covers: `Free` values have ordinary copy/discard behavior and may be discarded by `ignore` in pattern positions.

`Linear` types are exact-use types. A value of a `Linear` type must be consumed exactly once along every reachable control-flow path, unless it is moved into another owner whose later consumption is checked. A linear value may not be copied, silently discarded, hidden behind `ignore`, thrown away by a catch-all pattern, or lost through an early exit.

> Trace: D2, D98, D195, D206, D246
> Covers: `Linear` values require exact consumption and cannot be hidden by discard syntax or cleanup folklore.

`Type` in a generic parameter means the parameter accepts either `Free` or `Linear` type arguments. Inside generic code, a value of such a parameter is treated conservatively: it may be moved, borrowed, stored, returned, or passed to APIs that accept it, but it may not be implicitly copied or silently discarded.

> Trace: D195
> Covers: `T: Type` admits both universes and gives generic code only operations sound for both.

`Auto` is legal only as a declaration-site classification marker for type constructors. It is not a generic bound and cannot appear where a type parameter constraint is expected. A type declared `Auto` chooses the instantiated type's universe by its declared classification rule.

> Trace: D194, D195
> Covers: `Auto` is a declaration classifier only; generic parameters use `Type`, `Free`, `Linear`, region constraints, or admitted const-generic parameter types.

## Constructor Classification

Universe membership comes from the type constructor's declaration or special-form rule. A non-generic type declared `Free` is always `Free`. A non-generic type declared `Linear` is always `Linear`. A generic type declared `Free` is always `Free`. A generic type declared `Linear` is always `Linear`. A generic type declared `Auto` is `Linear` iff at least one type argument is `Linear`; otherwise it is `Free`.

> Trace: D194, D195
> Covers: User and standard nominal type constructors use an explicit declaration classifier; generic `Auto` propagates linearity exactly when a type argument is linear.

Phantom parameters do not weaken classification. A phantom type argument participates in identity and instance resolution even when it has no runtime field, but a generic `Auto` type with only phantom linear arguments still follows the type constructor's declared classification rule. If a declaration wants phantom parameters not to affect runtime layout, it must still account for the identity and coherence effect.

> Trace: D181, D190, D194, D214
> Covers: Phantom parameters affect type identity and coherence; layout absence does not make the parameter semantically absent.

Language-defined and special forms have closed classification rules:

| Type Form | Classification |
| --- | --- |
| `Unit`, `Bool`, `Never`, `StaticString` | Always `Free`. |
| Fixed-width integers, fixed-width floats, `Index` | Always `Free`. |
| Built-in target descriptors such as `Os`, `Arch`, `Abi`, `Endian` | Always `Free`. |
| `FnPtr(...) : Ret` | Always `Free`. |
| `Address[T]`, `Pointer[T]` | Always `Free`; they are raw addresses, not owners. |
| `&[T]`, `&![T]`, and other borrow/view forms | Always `Free`; borrowing does not transfer ownership. |
| `Vector[T, N]` | Always `Free` because the portable vector element domain is fixed-width scalar or mask data. |
| `Optional[T]`, `Result[T, E]`, `Array[T, N]` | `Auto`. |
| `String`, `Buffer[T]`, `Box[T]`, `PinBox[T]`, `Atomic[T]`, `Sender[T]`, `Receiver[T]`, `Mutex[T]`, `RwLock[T]`, capabilities, and owning handles | `Linear` when declared as owning or synchronization resources. |

> Trace: D24, D89b, D104, D194, D195, D255
> Covers: Built-in and special type forms have explicit constructor-by-constructor universe rules; no classification is inferred from naming folklore.

## Built-In Type Families

`Unit` is the type of no useful value. Its only value is `nil`. `Bool` is the type of Boolean control and comparison results. Its only values are `true` and `false`. `Never` is the type of expressions that do not produce a value because they diverge or terminate.

> Trace: D24, D58, D191, D194
> Covers: `Unit`, `Bool`, and `Never` are built-in `Free` types with fixed value and divergence roles.

Integer types are exact-width signed and unsigned types plus `Index`. Fixed-width signed types are `Int8`, `Int16`, `Int32`, and `Int64`. Fixed-width unsigned types are `Nat8`, `Nat16`, `Nat32`, and `Nat64`. `Index` is the language's index and size-count type selected by the target contract. Integer operations have defined Kyokai semantics; overflow behavior and trap behavior are specified by the expression and numeric chapters, not inherited from the backend.

> Trace: D12, D41, D75, D135/D261, D139, D228
> Covers: Built-in integers are `Free`, literal typing is explicit/contextual, and numeric semantics cannot rely on C undefined behavior.

Floating-point types are `Float32` and `Float64`. They are `Free`. Their operation semantics, exceptional values, and target requirements are specified by the numeric chapter and target contract, not by vague host defaults.

> Trace: D75, D139, D194, D228
> Covers: Floating-point types are built-in `Free` types whose semantics are specified by Kyokai and target contracts.

`StaticString` is a `Free` compile-time text type produced by `static "..."`. Ordinary runtime `String` is a linear owning type and is not produced by ordinary string literals through contextual retargeting.

> Trace: D120, D194, D195
> Covers: Static text and runtime owning strings are distinct; `StaticString` is `Free`, while owning `String` is linear.

`Optional[T]` and `Result[T, E]` are language-level built-in sum families with constructors `Some`, `None`, `Ok`, and `Err`. They are `Auto`: they become linear when any payload type is linear. They are not imported from a prelude module.

> Trace: D15, D24, D194, D195
> Covers: `Optional` and `Result` are built-in `Auto` sum families used by fallible binding and error surfaces.

`Array[T, N]` is the fixed-length array family. Its length parameter `N` is an `Index` const generic. `Array` is `Auto`, so arrays of linear elements are linear and arrays of free elements are free.

> Trace: D55, D159, D188, D194
> Covers: Fixed arrays have compile-time `Index` length parameters and linearity follows element classification through `Auto`.

## References, Raw Pointers, And Function Pointers

`&[T]` is an immutable borrow type. `&![T]` is a mutable borrow type. A borrow type is `Free` because the borrow value is access, not ownership. Copying or discarding a borrow value does not duplicate or destroy the referent, but the borrow checker still controls aliasing, lifetime, reborrow, and escape.

> Trace: D6, D7b, D14, D187, D238-D240
> Covers: Borrow reference types are explicit, non-owning, always `Free`, and governed by the borrow checker rather than by ownership transfer.

The common borrow spelling omits a named region. `&[T]` means an anonymous scope-bounded borrow region managed by the compiler. Named regions may be written only where the region chapter admits them, such as rare signatures that return a borrow tied to an input region.

> Trace: D6, D14, D187
> Covers: Anonymous-by-default borrow regions are complete source types; named regions are the explicit advanced form.

`Address[T]` is a nullable raw address and `Pointer[T]` is a non-null raw pointer. Both are `Free` because they do not own the referenced storage. Safe Kyokai does not infer validity, lifetime, alignment, aliasing, initialization, or ownership from these types alone. Operations that turn raw addresses into safe values or borrows belong to unsafe modules and unsafe contracts.

> Trace: D20, D20a, D73, D77, D194, D245
> Covers: Raw pointer-like forms are non-owning `Free` values and require unsafe contracts for validity-sensitive operations.

`FnPtr(...) : Ret` is the bare function pointer type family. It is always `Free`. It represents a callable address without an environment. Closure and callable-family types are separate higher-level facilities and do not cross raw FFI directly unless a later chapter specifies an explicit wrapper.

> Trace: D20a, D21, D82, D118, D126, D194
> Covers: Bare function pointers are free, environmentless, and distinct from closure/callable values.

## Nominal Identity

Kyokai type identity is nominal and recursive after canonicalization. Name resolution identifies the declaration behind every nominal type name. Type aliases expand away. Surface sugar is desugared. Associated-type projections normalize when the governing instance is known.

> Trace: D33, D50, D190
> Covers: Type identity checks use canonical forms after alias expansion, sugar removal, and associated-type normalization.

Two primitive built-in types are identical iff they are the same built-in type. Two user-declared nominal types are identical iff they refer to the same package-qualified and module-qualified declaration. Two applications of the same type constructor are identical iff the constructor is identical and every corresponding type argument or const argument is identical under that argument's equality rule.

> Trace: D159, D188, D190
> Covers: Nominal identity includes declaration identity and evaluated const-generic argument equality.

Structural coincidence does not create type identity. Two records with the same field list are different if declared separately. Two unions with the same variants are different if declared separately. A single-field wrapper is not identical to its field type. ABI equality is not type equality.

> Trace: D42, D109, D190, D196
> Covers: Kyokai has no structural typing or ABI-based type equality.

Kyokai has no tuple types, tuple literals, tuple destructuring, or positional multi-return types. Multi-value structure is represented by named records or other nominal types.

> Trace: D47, D131
> Covers: Tuples do not exist in the type system; named nominal types carry structure.

## Subtyping, Coercions, And Variance

Kyokai has no general subtyping relation. A value has its static type, and type compatibility is based on identity, explicit construction, explicit conversion APIs, or a closed language coercion rule. User-defined generic nominal types are invariant in all type parameters.

> Trace: D186, D190, D191
> Covers: There is no structural subtyping, no inheritance subtyping, and no user-extensible variance system.

`Never` has an expression-site coercion rule for diverging expressions. A `Never` expression may be used where another type is expected because it produces no value. This is not a general subtype lattice.

> Trace: D58, D191
> Covers: Divergence uses a narrow expression-site `Never` coercion only.

Kyokai has a closed `Never` lifting table for built-in `Optional` and `Result`: `Optional[Never]` may lift to `Optional[T]`, `Result[Never, E]` may lift to `Result[T, E]`, and `Result[T, Never]` may lift to `Result[T, E]`. No other generic constructor gains lifting from this rule.

> Trace: D15, D186, D191
> Covers: Generic `Never` lifting is closed to the built-in optional/result cases and does not imply variance.

Borrow/reference conversions are separate explicit rules. For example, mutable-to-immutable read reborrow belongs to the borrow elaboration rules, not to general variance or subtyping.

> Trace: D7b, D14, D187, D238-D240
> Covers: Borrow conversions are checked by the borrow system and are not implied by generic variance.

## Recursive Types And Layout Finiteness

Recursive and mutually recursive nominal types are legal only when the fully expanded representation has finite size. Every cycle in the layout-dependency graph must pass through at least one indirection-bearing field.

> Trace: D160, D217
> Covers: Recursive nominal types are allowed only under a representation-based finite-layout rule.

Indirection-bearing forms include `Box[T]`, `PinBox[T]`, `Address[T]`, `Pointer[T]`, `&[T]`, and `&![T]`. A type constructor breaks a recursive layout cycle only when its own layout rule explicitly says it stores the parameter indirectly. Ordinary records, unions, single-field wrappers, `Optional`, `Result`, arrays, and aliases are inline for this purpose unless their own declaration says otherwise.

> Trace: D42, D89b, D160, D217
> Covers: Recursion breaking is based on representation, not on a type name or a compiler guess.

The recursion check runs after alias expansion over the whole strongly connected declaration group. An illegal recursive layout is a compile-time error. The diagnostic must report at least one offending cycle and at least one legal indirection strategy.

> Trace: D50, D160, D217
> Covers: Infinite inline layout cycles are rejected after alias expansion with actionable diagnostics.

## Type Well-Formedness

A type expression is well-formed only when every named type, type parameter, associated type, region parameter, and const parameter is in scope; every type argument satisfies its universe constraint; every const argument has type `Index` and is compile-time evaluable where required; and every associated-type projection has a governing typeclass obligation.

> Trace: D33, D158, D188, D189, D190, D195
> Covers: Type well-formedness checks names, constraints, const arguments, and associated-type projections before values are checked.

A public type signature may mention only declarations visible to the consumer. Public API cannot leak private body-only names. Public API cannot leak same-package `internal` names to outside packages unless an explicit opaque exposure rule allows that exact shape.

> Trace: D17, D78, D79, D229
> Covers: Type visibility is enforced through signatures and artifacts; hidden names cannot leak as accidental public API.

A type whose layout or ownership behavior depends on unsafe or target-specific facts must make that dependency explicit through `extern record`, `extern type`, target guards, unsafe contracts, or a standard-library semantic contract. Safe accepted Kyokai programs do not depend on language-level undefined behavior.

> Trace: D20, D42, D73, D139, D228, D245
> Covers: Target and unsafe-dependent type behavior must be specified explicitly and cannot rely on UB.
