# Generics And Typeclasses

[Rikona Kurasaki / Mjoyufull]
Generics are where a language can start lying kindly. It can let a type drift in from somewhere offscreen, let a method find a hidden witness, let a return type pull answers backward through the program, and the call still looks clean. Kyokai keeps the useful power and refuses the fog: parameters are written at declarations, obligations sit in one place, and dispatch is static.

> Trace: D82, D158, D189, D192, D193, D195
> Covers: Kyokai generics are explicit, rank-1, statically dispatched, and do not use hidden runtime dictionaries, existential types, or opaque return types.

## Generic Parameters

A declaration may introduce generic parameters on records, unions, type aliases, opaque types, functions, methods, typeclasses, instances, and generators. Type parameters are introduced in brackets after the declared name.

```kyokai
record Singleton[T: Type]: Auto is
    value: T;
build;

function identity[T: Type](x: T): T is
    return x;
qed;
```

> Trace: D158, D192, D195
> Covers: Generic parameters are declaration-site parameters on named declarations; Kyokai does not use a separate old-style generic header as the canonical surface.

A type parameter may be constrained by `Type`, `Free`, or `Linear`. `T: Type` accepts any value type and treats values conservatively inside generic code. `T: Free` accepts only free types. `T: Linear` accepts only linear types. `Auto` is not legal as a type-parameter constraint.

> Trace: D194, D195
> Covers: `Type`, `Free`, and `Linear` are parameter constraints; `Auto` is declaration-site classification only.

A generic parameter may also be an `Index` const parameter. Const parameters are compile-time values, not types. They are used for shape-bearing declarations such as arrays, fixed buffers, matrices, and protocol widths.

```kyokai
record Matrix[T: Type, Rows: Index, Cols: Index]: Auto is
    data: Array[T, Rows * Cols];
build;
```

> Trace: D159, D188
> Covers: Kyokai supports first-class `Index` const generics with deterministic compile-time value equality.

Every const-generic argument must evaluate at compile time to exactly `Index`. Legal forms are `Index` literals, named constants of type `Index`, earlier const parameters, parenthesized deterministic `Index` expressions built from admitted inputs, and explicit `comptime` calls that return `Index`. Failure to evaluate, overflow, trap, step-budget exhaustion, or a non-`Index` result is a compile-time error.

> Trace: D18, D18a, D159, D188, D202, D203
> Covers: Const generic arguments use one deterministic compile-time model and fail at compile time when they cannot produce an `Index` value.

Region parameters are admitted only by the borrow and region chapters. The common borrow types `&[T]` and `&![T]` do not require a region parameter in ordinary signatures.

> Trace: D6, D14, D187
> Covers: Anonymous borrow regions are the ordinary surface; named region parameters are explicit advanced generic parameters only where the region rules admit them.

## Rank And Quantification

Kyokai generics are rank-1 only. Type parameters may be introduced only on declarations. A type expression in a parameter, field, local binding, associated type definition, return type, or other value-level type position may not introduce a nested universal quantifier.

> Trace: D192
> Covers: Kyokai rejects higher-rank, rank-N, and impredicative polymorphism.

A generic function is not a first-class polymorphic function value. It may be referenced, called, or passed only after the use site resolves it to an ordinary fully instantiated callable type.

> Trace: D21, D82, D126, D192
> Covers: Generic functions must be instantiated before use as ordinary callable values; polymorphic function values are not a separate type form.

Kyokai has no existential types, no `impl Trait`-style opaque returns, no hidden trait objects, and no erased generic containers. A function return type must name a concrete visible type expression. Heterogeneous abstraction uses explicit unions, nominal wrappers, or explicit callback/capability APIs.

> Trace: D82, D193
> Covers: Functions and stored values do not hide concrete implementation types behind existential or opaque-return surfaces.

## Generic Type Argument Inference

Call-site generic type argument inference is local, argument-driven, forward-only, and left-to-right. If explicit type arguments are omitted, the compiler may infer them only from the receiver and explicit value arguments of that call after ordinary desugaring such as UFCS.

> Trace: D7a, D12, D82, D209/D161, D254
> Covers: Omitted call type arguments are solved from call inputs only, after receiver-call desugaring, and never from distant context.

Earlier arguments may constrain later arguments. Later arguments do not retroactively reinterpret earlier argument expressions. Literal typing may use type arguments already solved from earlier arguments in the same call. If any omitted type parameter remains unsolved or ambiguous after argument-side information is processed, the call is a compile-time error and the program must supply explicit type arguments.

> Trace: D12, D87, D209/D161
> Covers: Generic inference is left-to-right, local, deterministic, and rejects unsolved or ambiguous omissions.

Expected return type, assignment target type, `let` annotation, enclosing expression context, pattern context, match arm result context, and surrounding generic obligations do not solve omitted type arguments. A call such as `let buf: Buffer[Int32] := Buffer.new(alloc);` is legal without explicit type arguments only when `alloc` or earlier explicit arguments determine `Int32` by the call-inference rules.

> Trace: D46, D87, D209/D161
> Covers: Kyokai rejects return-context and target-type inference for generic call arguments.

Generic constructors, ordinary function calls, UFCS calls, and typeclass method calls use the same inference rule after desugaring. Import order and visible instance order do not provide a tie-breaker for inference.

> Trace: D82, D179, D209/D161, D214, D254
> Covers: Generic inference is uniform across call surfaces and cannot depend on import order.

## Where Clauses

A `where` clause states generic obligations that do not belong as a single baseline constraint in the parameter list. It appears after the parameter list and before contracts or `is`.

```kyokai
function printAll[C: Iterable](iter: &[C]): Unit
where
    C.Item: Displayable
is
    // body
qed;
```

> Trace: D158, D189
> Covers: Non-header generic obligations use one explicit `where` surface.

Where obligations are comma-separated and conjunctive. The admitted forms are `T: Trait`, `Projection: Trait`, and `Projection == TypeExpr`. A projection is an associated-type projection such as `C.Item`. `TypeExpr` is an ordinary type expression in the current generic scope.

> Trace: D33, D158, D189, D190
> Covers: `where` has a closed grammar for typeclass bounds, associated-type bounds, and associated-type equality.

`T: Foo + Bar` does not exist. Multiple obligations are written as repeated entries. Constraint order has no semantic meaning. Duplicate obligations are compile-time errors. Contradictory obligations are compile-time errors after alias expansion and associated-type normalization.

> Trace: D33, D158, D189, D190
> Covers: Where clauses are unordered conjunctions with duplicate and contradiction checks.

Value-level predicates do not belong in `where`. Preconditions, postconditions, and compile-time Boolean checks are written as contracts, `static_assert`, or other explicit value-level forms.

> Trace: D18a, D53, D140, D142, D189
> Covers: Type obligations and value obligations stay separated; `where` is not a dependent predicate language.

## Associated Types

A typeclass may declare associated types. Associated types express that a secondary type is determined by the selected instance.

```kyokai
typeclass Iterable[Self: Type] is
    type Item;
    method next(iter: &![Self]): Optional[Self.Item];
spec;
```

> Trace: D33, D82, D189
> Covers: Associated types are declared inside typeclasses and are selected statically through the governing instance.

Every instance must define each associated type exactly once. Associated types have no defaults. Once the instance is chosen, the associated type is fully determined; it is not another free dispatch parameter.

```kyokai
instance Iterable for Buffer[Int32] is
    type Item := Int32;

    method next(iter: &![Buffer[Int32]]): Optional[Int32] is
        // body
    qed;
qed;
```

> Trace: D33, D182, D214
> Covers: Instances fill associated types exactly once, and associated types do not use default definitions.

Associated-type projections are written with dot syntax such as `Self.Item` or `C.Item` where the receiver type is constrained by the relevant typeclass. Projections normalize during type canonicalization when the selected instance is known.

> Trace: D33, D189, D190
> Covers: Associated-type projections are ordinary type expressions that normalize through known static instances.

Functional dependencies, open type families, generalized associated type constructors, and a second mechanism for expressing the same instance-determined relation are not part of Kyokai.

> Trace: D33, D147
> Covers: Associated types are the one built-in instance-determined secondary-type mechanism.

## Typeclasses

A typeclass is a static contract over types. It may contain required methods, default method bodies, and associated type declarations. It does not create a runtime interface object, hidden vtable, hidden dictionary, or dynamic dispatch path.

> Trace: D33, D82, D182, D193
> Covers: Typeclasses are compile-time contracts and do not imply runtime trait objects or dictionaries.

A required method has a signature and no body. A default method has a body in the typeclass declaration. A default body may call other methods of the same typeclass and may refer to associated types. An instance must implement every required method and may override any default method.

> Trace: D33, D182
> Covers: Default method bodies are allowed, required methods must be implemented, and instances may override defaults.

Typeclass methods may have contracts. Those contracts are part of the method obligation. An instance method may strengthen implementation detail internally, but it may not expose a weaker public contract than the typeclass method requires.

> Trace: D53, D140, D142, D182
> Covers: Typeclass method contracts are part of static method obligations and remain always checked.

Kyokai does not define a general `Cloneable[T]` typeclass. Ordinary `Free` copy behavior is not user-extensible deep cloning. APIs that create fresh owned duplicates must be explicit and type-specific, and allocation-producing duplication must take an explicit allocator.

> Trace: D176, D195, D201
> Covers: Deep duplication is not a blanket generic promise; allocation and ownership effects stay visible.

Kyokai does not define a general `Default[T]` typeclass. Generic code cannot fabricate a value from nothing. Empty, zero, initial, or sentinel values must come from explicit constructors, constants, callbacks, or concrete type APIs.

> Trace: D177
> Covers: There is no generic default-value hook or zero-bit-pattern construction rule.

Generic cleanup of linear values uses an explicit standard typeclass such as `Destroyable[T: Linear]`. The language does not invent structural destruction for user types and does not run hidden destructors at scope exit.

> Trace: D2, D89, D157, D195
> Covers: Linear cleanup is explicit through APIs/typeclasses and visible cleanup statements, not hidden RAII or structural destruction.

## Instances And Coherence

An instance is legal only where the package owns the typeclass or owns at least one concrete head type named by the instance. Internal or private visibility does not relax this orphan rule.

> Trace: D17, D214
> Covers: Kyokai uses an orphan/coherence rule for instances and does not allow visibility tricks to hide orphan conflicts.

For any fully resolved typeclass application visible at a call site, there must be exactly one applicable legal instance. If no instance applies, the call is a compile-time error. If more than one legal visible instance could apply, the program is rejected.

> Trace: D82, D214, D216
> Covers: Typeclass dispatch is deterministic and rejects missing or overlapping applicable instances.

Blanket and generic instances are allowed only when uniqueness is still provable. If a generic overlap exists, the diagnostic must name the conflicting instances, their defining packages/modules, and at least one concrete witness substitution that demonstrates the conflict.

> Trace: D214, D216
> Covers: Generic instances must preserve uniqueness and overlap diagnostics must include a witness when possible.

Instance visibility follows module and package visibility. Public instances are visible to importing packages. Internal instances are visible only inside the package and may be recorded in `.koi` artifacts for same-package checking, but downstream packages treat them as absent.

> Trace: D17, D79, D214
> Covers: Instance visibility respects public/internal/private boundaries and artifact consumption rules.

Import order never chooses an instance. A module cannot make a typeclass call mean something different by reordering imports. If two visible instances overlap, the program is ill-formed rather than order-dependent.

> Trace: D179, D214
> Covers: Instance resolution is independent of import order.

## Static Dispatch And Materialization

A generic/typeclass call is resolved statically. The compiler may materialize specialized code, share code where that is semantics-preserving, or use an implementation strategy recorded by the toolchain contract, but the source language does not gain hidden runtime dictionary parameters or trait-object dispatch.

> Trace: D82, D82a, D82b, D139, D228
> Covers: Generic and typeclass dispatch is static; implementation strategies must preserve the source semantics without hidden runtime dictionaries.

Allocator handles, stream handles, capability values, and other explicit runtime values are not typeclass dictionaries. If a container stores an allocator chosen by the programmer, later calls through that stored value are ordinary explicit runtime state, not a violation of static typeclass dispatch.

> Trace: D44, D82
> Covers: Explicit runtime state remains legal; D82 bans hidden compiler-inserted generic/typeclass witnesses.

`.koi` interface artifacts must record enough generic and typeclass metadata for downstream type checking, materialization, instance resolution, and diagnostics without requiring source bodies. The artifact is a checked contract, not a cache blob.

> Trace: D79, D82a, D82b, D83
> Covers: Interface artifacts carry generic/typeclass contract metadata needed for separate compilation and deterministic checking.
