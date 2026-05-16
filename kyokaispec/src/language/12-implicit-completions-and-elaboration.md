# Implicit Completions And Elaboration

[Rikona Kurasaki / Mjoyufull]
Kyokai is not allergic to every omitted token. It is allergic to lies. If the compiler fills something in, there must be only one thing it could have filled, every other reading must be ill-typed, and the insertion must not smuggle in allocation, authority, blocking, cleanup, or control flow that the source did not already demand.

> Trace: D87, D238, D239, D240
> Covers: Kyokai admits only a closed set of tautological implicit completions, records them during elaboration, and checks them before ownership and backend lowering.

This chapter defines the order in which surface syntax becomes checked core. It is a compiler contract and a reader contract. The compiler may optimize the implementation, but diagnostics, ownership checks, borrow checks, capability checks, and backend lowering must behave as though this order happened.

> Trace: D73, D139, D228, D238
> Covers: Elaboration order is normative for source meaning even if implementation internals are optimized.

## Tautology Rule

An implicit completion is legal only when the operation is uniquely determined by static types and surrounding syntax, all alternate interpretations are statically ill-typed, and the insertion adds no new allocation, side effect, blocking operation, capability use, cleanup action, or control-flow edge beyond what the explicit source construct already implies.

> Trace: D87, D239
> Covers: The compiler may insert only effect-neutral, uniquely determined operations whose alternatives are ill-typed.

If a proposed insertion fails any part of that rule, the program is rejected unless the programmer writes an explicit operation that the language admits. The compiler does not repair ambiguous code by guessing.

> Trace: D87, D239
> Covers: Ambiguous or effectful implicit behavior is not permitted and must become explicit source or a compile-time error.

The set of implicit completions is closed. Adding a new implicit completion requires an explicit accepted language decision, a registry entry, and conformance tests.

> Trace: D87, D239
> Covers: Kyokai cannot grow new implicit behavior through implementation convenience.

## Elaboration Order

The compiler first parses source into a surface AST. No implicit operations are inserted during parsing.

> Trace: D238
> Covers: Parsing recognizes source forms only and does not perform semantic repair.

The compiler resolves modules, imports, visibility, and names enough to reject import and name ambiguity. Syntax-only lowering then normalizes constructs whose meaning does not depend on types, including unresolved UFCS call shape as a call-with-receiver node.

> Trace: D17, D78/D179/D214, D238
> Covers: Name/module ambiguity is rejected before typed insertion, and UFCS is represented without pretending receiver lookup is already solved.

The compiler type-checks and elaborates expressions, resolving operator families, typeclass obligations, generic arguments, literal types, expected-type sites, pattern types, and the constrained receiver-module UFCS fallback.

> Trace: D23, D82/D82a/D82b, D110/D254, D209/D161, D238
> Covers: Typed elaboration resolves overload-like surfaces, generic arguments, and receiver lookup before ownership checking.

Only after those facts are known may the compiler insert approved implicit completions. Every insertion becomes an explicit elaboration node carrying the source span, expected type or syntactic context, inserted core operation, and the rule that admitted it.

> Trace: D87, D238, D239
> Covers: Implicit completions are typed elaboration nodes, not invisible backend rewrites.

A mandatory tautology-check pass validates every inserted completion. It rejects any insertion with no registry entry, any insertion that is not uniquely determined, any insertion whose alternatives are not ill-typed, and any insertion that adds forbidden effects.

> Trace: D87, D239
> Covers: The compiler audits its own inserted operations before later checks rely on them.

Typed sugar such as `or return`, `let...else`, and `while let` lowers into checked core only after the relevant types, patterns, and inserted completions are recorded. Linearity, borrow, exhaustiveness, capability, contract, and unsafe checks run on the elaborated core, not on the raw surface AST.

> Trace: D15a, D39, D98, D205, D238-D240
> Covers: Ownership-sensitive sugar is made explicit before the checkers prove it safe.

Backend lowering starts only after elaboration and all semantic checks succeed. The backend may not add a new language-level implicit behavior or rely on backend undefined behavior to implement one.

> Trace: D73, D139, D228, D238
> Covers: Backend lowering consumes checked elaborated core and cannot invent or weaken source semantics.

## Registry Of Accepted Completions

The current implicit-completion registry contains exactly these source-level insertions: literal typing from known context, immutable `let :=` type inference from a fully determined initializer, auto-borrow for addressable places, auto-reborrow from mutable borrows, mutable-to-immutable read reborrow, one-level field auto-deref, implicit end-of-body `Unit` return, expression-site `Never` coercion, closed `Never` lifting for `Optional`/`Result`, and the exact `or ...` result-propagation sugar.

> Trace: D7b, D8, D12, D15a, D34, D46, D58/D191, D87, D186, D187, D238-D240
> Covers: The accepted registry is closed and names every current implicit completion directly.

Generic type argument inference is not a global implicit conversion system. It is a local call elaboration rule that may omit explicit type arguments only when receiver and argument expressions determine them left to right.

> Trace: D209/D161
> Covers: Generic argument solving is local to a call and does not infer from return or target context.

The registry does not include implicit numeric conversions, implicit error conversions, implicit destructors, implicit imports, hidden prelude expansion, hidden allocator lookup, hidden capability acquisition, hidden environment capture, hidden task capture, hidden buffering, or hidden async/runtime scheduling.

> Trace: D11b, D15a, D37, D62, D82/D82a/D82b, D118, D152, D211, D234, D237
> Covers: Common convenience mechanisms rejected by Kyokai remain explicit or absent.

## Literal Typing

A numeric literal may receive a type from a known expected type at its expression site. The expected type may come from a function parameter, explicitly annotated local binding, operator operand, array element context, const-generic context, or another immediately known type site.

> Trace: D12, D135/D261, D87
> Covers: Literal inference pushes an already-known type into a literal at the local expression site.

A suffixed literal does not need contextual inference. Its suffix fixes the type before context. Context may confirm the suffix type but may not retarget it to another type.

> Trace: D135/D261
> Covers: Literal suffixes are explicit literal typing and not implicit conversions.

Literal typing may not choose among multiple valid numeric types by preference. `let x := 5;` is illegal unless the initializer itself already fixes a single type. Mixed numeric operators do not use literal typing to perform promotions between non-literal operands.

> Trace: D12, D46, D75, D210
> Covers: Literal inference rejects ambiguity and does not create C-style promotions.

## Let Inference

`let name := expr;` may omit the type annotation only when `expr` already has exactly one fully determined, denotable type before considering the omitted annotation.

> Trace: D46, D87
> Covers: Immutable local inference removes repetition only when the right-hand side already determines the type.

The rule applies to immutable `let` bindings only. Mutable `var` bindings require explicit type annotations. Empty literals, ambiguous numeric literals, generic constructors needing target typing, and polymorphic calls whose return type is not already fixed require explicit annotations or explicit type arguments.

> Trace: D46, D209/D161
> Covers: Let inference is not target typing and does not solve polymorphism from the binding annotation.

Pattern bindings use the type of the scrutinee expression. The pattern language does not introduce a separate inference engine.

> Trace: D46, D98, D205
> Covers: Pattern typing flows from the matched expression, not from guessed binding shapes.

## Auto-Borrow And Reborrow

When a call expects `&[T]` and the argument expression is an addressable place of type `T`, the compiler may insert an immutable borrow of that place. When a call expects `&![T]` and the argument is a mutable addressable place of type `T`, the compiler may insert a mutable borrow if no aliasing or lifetime rule is violated.

> Trace: D14, D72, D87, D238-D240
> Covers: Auto-borrow is allowed only for addressable places where the expected borrow type is uniquely known and the borrow checker can prove legality.

The compiler may not mutable-borrow an rvalue temporary. It may not insert a borrow whose lifetime would escape the statement or borrow scope. It may not insert a borrow that conflicts with an existing live borrow.

> Trace: D72, D187, D238-D240
> Covers: Auto-borrow respects temporary lifetime, escape, and aliasing rules.

When a call expects `&![T]` and the argument is an existing mutable borrow `&![T]`, the compiler may insert a temporary mutable reborrow. The original mutable borrow is suspended for the reborrow's lifetime.

> Trace: D7b, D87, D238-D240
> Covers: Auto-reborrow removes repeated `&~` only when the expected mutable borrow is the sole valid operation.

When a call expects `&[T]` and the argument is an existing mutable borrow `&![T]`, the compiler may insert a temporary immutable read reborrow. This is not subtyping. The original mutable borrow is suspended while the read reborrow is live.

> Trace: D187, D238-D240
> Covers: Mutable-to-immutable borrowing is a read reborrow, not variance or subtype conversion.

Auto-borrow and reborrow insertions must be visible in the elaborated core. The linearity and borrow checkers must see explicit borrow/reborrow nodes; a backend may not synthesize these operations after checking.

> Trace: D238-D240
> Covers: Borrow ergonomics are surface sugar over explicit checked core borrow operations.

## Field Auto-Deref

For direct field access or field assignment, if the base has type `&[Record]` or `&![Record]`, the compiler may insert one dereference level and then access the field. This insertion is limited to one reference level.

> Trace: D34, D87, D238-D240
> Covers: Field access through a borrow has exactly one auto-deref level.

Field auto-deref does not apply to arbitrary method dispatch, does not chase nested references, does not invoke user code, and does not create a `->` operator. If the base is a reference to a reference, the programmer must write an explicit dereference where the language admits it.

> Trace: D34, D110/D254
> Covers: Auto-deref is field-only, non-recursive, and not method lookup magic.

## Unit Fallthrough

At the end of a function declared `: Unit`, if control reaches the end of the function body, the compiler inserts the single `Unit` result. The insertion happens only at that end-of-body fallthrough point.

> Trace: D8, D87, D238-D240
> Covers: Implicit `Unit` completion is end-of-body only and is justified by `Unit` having exactly one value.

An early exit must still be written. `return;`, `return nil;`, `break;`, `continue;`, and `panic(message);` all carry meaningful control-flow information and are not inferred from context.

> Trace: D8, D43, D84
> Covers: Unit fallthrough does not infer early control-flow exits.

Non-`Unit` functions receive no implicit return value. Kyokai does not have last-expression return for arbitrary functions.

> Trace: D8
> Covers: Rust-style implicit expression return is rejected.

## Never Coercion

An expression of type `Never` may satisfy any immediate expected type at the expression site because it cannot produce a value. This applies to branch joins, return expression checking, argument positions with known parameter types, and other immediate expected-type sites.

> Trace: D58/D191, D87
> Covers: `Never` coercion is expression-site only and explains branch typing for statically diverging paths.

A possible runtime TPOE does not make an expression type `Never`. The expression keeps its ordinary result type unless the language statically knows it cannot complete normally.

> Trace: D58/D191, D75, D84, D121
> Covers: Potential contract failure is not static divergence.

`Never` coercion does not create a general subtype lattice and does not rewrite arbitrary enclosing generic constructors. The only generic lifting tied to `Never` is the closed built-in `Optional`/`Result` table specified by the type-system chapter.

> Trace: D58/D191, D186
> Covers: `Never` has narrow expression-site coercion plus closed Optional/Result lifting only.

## Or-Sugar Desugaring

`or return`, `or return name => expr`, `or break`, and `or continue` are accepted implicit result-propagation sugars because their expansion is exact and locally determined.

> Trace: D15, D15a, D87
> Covers: Result propagation sugar is an admitted implicit completion only because its desugaring is fixed.

`let x: T := expr or return;` lowers exactly to `let Ok(x) := expr else Err(__e) do return Err(__e); fi;`. `or return name => map(name)` lowers exactly to `let Ok(x) := expr else Err(name) do return Err(map(name)); fi;`.

> Trace: D15a, D119
> Covers: Plain and mapped `or return` have explicit Result-only desugarings and no implicit error conversion.

`or break` lowers to an else arm that breaks, and `or continue` lowers to an else arm that continues. Their discarded error payload must be `Free`; a linear error payload requires explicit `let...else` handling.

> Trace: D15a, D195, D206
> Covers: Loop-control result sugar cannot hide linear error payload discard.

For ownership and cleanup analysis, the checker analyzes the lowered form. `or return` is a structured error exit for `errdefer`; `or break` and `or continue` are ordinary loop-control exits.

> Trace: D2b, D15a, D238, D246
> Covers: Result propagation cleanup behavior comes from the explicit lowered control-flow path.

## Conformance Obligations

The compiler must maintain tests or other conformance evidence for owned value to immutable borrow, owned value to mutable borrow, mutable reference to mutable reborrow, mutable reference to immutable read reborrow, rejection of immutable reference to mutable borrow, nested temporary reborrow lifetimes, overlapping mutable reborrow rejection, read-reborrow suspension of the original mutable borrow, UFCS interaction, `or return` and `defer` interaction, and rejection of escaped temporaries or returned reborrowed references.

> Trace: D240
> Covers: Borrow/reborrow ergonomics require explicit conformance coverage over the elaborated core.

Every inserted implicit completion must be diagnosable. A diagnostic involving an inserted operation must be able to point to the source span that required it and name the rule in ordinary language, such as "inserted one statement-scoped immutable borrow" or "inserted end-of-body Unit return".

> Trace: D29, D239, D240
> Covers: Inserted operations remain auditable to users and tools through source-span diagnostics.

If an insertion fails, the diagnostic must explain the explicit source form that would make the program legal where such a form exists. If no explicit form is legal, the diagnostic must say which semantic rule rejects the program.

> Trace: D29, D87, D239
> Covers: Failed implicit completions produce actionable diagnostics rather than silent fallback behavior.
