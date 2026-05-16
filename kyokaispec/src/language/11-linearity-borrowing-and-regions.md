# Linearity, Borrowing, And Regions

[Rikona Kurasaki / Mjoyufull]
A linear value is not background scenery. It is a thing in somebody's hands. If the program moves it, the old hand is empty. If the program borrows it, the lender stands still until the borrow is done. If the program promises cleanup, that promise becomes part of the path through the room. Kyokai makes the compiler track those motions because this is where safety either becomes real or turns into a story people tell after the bug report arrives.

> Trace: D2, D6, D7b, D14, D72, D87, D187, D238-D240, D246
> Covers: Linearity and borrowing are checked after typed elaboration, including explicit nodes for admitted implicit borrow and reborrow completions.

This chapter defines the ownership state machine for local bindings, parameters, pattern bindings, temporaries, places, borrows, deferred cleanup, and movement. It is the checker contract. A compiler may use a different internal representation, but accepted programs and rejected programs must match this chapter.

> Trace: D73, D89, D143/D241, D228, D238
> Covers: The safe language has specified ownership behavior, no language-level undefined behavior, and a formal-core obligation around the sequential ownership model.

## Linearity Domain

A value is linear when its static type is in the `Linear` universe, when its static type contains a type parameter that may instantiate to `Linear`, or when a type constructor's universe rule classifies that instantiation as `Linear`. Linear checking applies before the compiler knows a generic value is safely `Free`; generic code must be written for the strongest admitted universe of the parameter.

> Trace: D24, D190/D192/D193/D195
> Covers: Linear checking follows universe classification and generic universe constraints instead of relying on concrete monomorphic luck.

A linear value must be consumed exactly once on every completing path that owns it. Consumption means moving the value into another owner, passing it to a consuming parameter, returning it by value, storing it into a place that takes ownership, sending it across an admitted ownership-transfer boundary, registering it in a consuming cleanup action, or calling an explicit consuming cleanup operation.

> Trace: D2, D89, D195, D246
> Covers: Linear ownership is exact-use ownership; cleanup is explicit source behavior, not hidden destruction.

A `Free` value may be copied, ignored, overwritten where mutation is legal, and allowed to leave scope without a consuming operation. The borrow checker may still restrict a `Free` place while it is borrowed, but `Free` values do not create exactly-once consumption obligations.

> Trace: D24, D194, D195
> Covers: `Free` classification removes linear consumption obligations while leaving ordinary borrow and mutability rules intact.

Borrow reference values of type `&[T]`, `&![T]`, `&[T, R]`, or `&![T, R]` are `Free` values. Copying a borrow reference value does not copy the referent, does not extend the region, and does not create ownership. The access represented by the borrow remains governed by its live region and aliasing state.

> Trace: D6, D14, D187
> Covers: Borrow references are non-owning access values with checker-managed aliasing and lifetime behavior.

## Checker States

For every local linear binding, field path, pattern-bound payload, temporary owner, and deferred cleanup capture that may own a linear value, the checker tracks a state. The minimum observable state set is `Live`, `Moved`, `Destroyed`, `Deferred`, `ErrDeferred`, `BorrowedRead`, `BorrowedMut`, `SuspendedByReborrow`, and `Unavailable`.

> Trace: D2, D7b, D14, D187, D246
> Covers: The checker has named ownership, borrow, and deferred-cleanup states rather than informal use counts.

| State | Meaning | Legal next motions | Rejected motions | Trace |
| --- | --- | --- | --- | --- |
| `Live` | The current scope owns the value and no conflicting borrow is active. | Move, destroy, return, send, store, read-borrow, mutable-borrow, defer, errdefer. | Duplicate binding of the same owner; scope exit without discharge. | D89/D195 |
| `Moved` | Ownership has left this place. | No value use; only diagnostics may refer to the old name. | Read, write, borrow, move again, destroy, defer, return. | D89 |
| `Destroyed` | A visible consuming cleanup operation has consumed the value. | No value use. | Any later use or cleanup. | D2/D195 |
| `Deferred` | A visible `defer` action has registered a consuming cleanup for this value. | Non-conflicting read borrows before scope exit. | Move, destroy now, return, send, store elsewhere, register another consuming cleanup. | D2/D2a/D246 |
| `ErrDeferred` | A visible `errdefer` action owns cleanup for structured error exits only. | Success-path move, return, destroy, or other discharge; non-conflicting borrows. | Duplicate cleanup on the same path; scope success without discharge. | D2b/D227/D246 |
| `BorrowedRead` | One or more immutable borrows are live. | More immutable borrows; reads through admitted access. | Move, destroy, mutable borrow, mutable write through the owner. | D14/D187 |
| `BorrowedMut` | Exactly one mutable borrow is live. | Reads/writes through that mutable borrow; reborrow under the rules below. | Move owner, destroy owner, second mutable borrow, ordinary read borrow from owner. | D14/D187 |
| `SuspendedByReborrow` | A mutable borrow is temporarily unusable while a nested reborrow is live. | Use the nested reborrow; end the nested region. | Use the suspended borrow, create conflicting borrows, move referent. | D7b/D187/D240 |
| `Unavailable` | Control path cannot continue normally, such as after `return`, `break`, `continue`, `panic`, `todo`, or `unreachable`. | Join only with other paths according to `Never` and control-flow rules. | Treating the path as if it produced live ownership state. | D58/D191/D84 |

A compiler may split these states into finer internal states. It may not collapse them in a way that admits a duplicate consumption, hidden drop, escaped borrow, conflicting alias, or path-dependent cleanup hole.

> Trace: D73, D195, D238-D240, D246
> Covers: Internal implementation freedom cannot weaken the observable ownership state machine.

## Movement

Moving a linear value transfers ownership and leaves the source in `Moved`. The source name or place remains syntactically present for diagnostics, but it no longer denotes a usable value.

> Trace: D89
> Covers: A moved-from linear place cannot be used again.

Kyokai's move model is as-if bytewise relocation. The language does not promise that the backend literally emits a byte copy, and it does not promise stable addresses for ordinary movable values. Optimizations may remove or fuse moves only when all observable ownership, borrow, layout, and backend-safety rules remain the same.

> Trace: D73, D89, D199, D228
> Covers: Move semantics are relocation semantics, not stable-address semantics, and backend lowering must preserve them without backend undefined behavior.

Moving a record, union, array, buffer, iterator, closure, task capture, capability, or owning handle follows the same rule: ownership transfers exactly once. A move of a composite linear value moves the whole composite. Partial moves exist only where a surface construct explicitly admits them and specifies the resulting owner state for every field.

> Trace: D89, D98, D195, D205-D206
> Covers: Composite movement is whole-value movement unless a specified construct exposes field-by-field ownership.

A `Free` value may be copied where the type admits copying. A `Linear` value is never copied by ordinary assignment, argument passing, return, generic dispatch, closure capture, task capture, formatting, debugging, or backend lowering. Any operation that truly duplicates a resource must be a named API with its own contract.

> Trace: D11b, D176, D228, D233
> Covers: Linear duplication is not hidden behind common expression forms or debug/profile behavior.

## Destruction And `drop;`

Kyokai has no hidden destructors. Leaving a scope does not call a type-specific cleanup operation merely because a linear value is live. If a linear value reaches ordinary scope exit still `Live` or success-live under `ErrDeferred`, the program is rejected.

> Trace: D2, D195, D246
> Covers: Scope exit is not implicit destruction; every linear owner must be visibly discharged.

A consuming cleanup operation is an ordinary function, method, or typeclass operation whose signature consumes the owner. Standard generic cleanup uses explicit contracts such as `Destroyable[T: Linear]`; the language does not synthesize structural destruction for user records or unions.

> Trace: D2, D82/D82a/D82b, D195, D246
> Covers: Generic cleanup is explicit static dispatch, not compiler-invented destructor behavior.

The `drop;` terminator closes a borrow scope. It does not destroy the borrowed referent and does not discharge an owning linear value. When `drop;` is reached, all borrow references whose region is that borrow scope become unusable, and the owner state resumes according to the borrow kind that ended.

> Trace: D9, D14, D111/D127, D187
> Covers: `drop;` ends borrow access only; it is not resource destruction.

`ignore` and omitted pattern subparts may discard only `Free` data. A pattern that would hide a linear payload behind `ignore`, an omitted field, or a catch-all is illegal. The program must bind that payload and visibly consume, move, return, defer, or otherwise discharge it.

> Trace: D38/D205/D206
> Covers: Pattern convenience cannot silently drop linear data.

## Regions

`&[T]` and `&![T]` are complete borrow types with anonymous scope-bounded regions. The programmer is not omitting a region argument; the anonymous region is what the common borrow type means. The compiler assigns internal region identities so it can reject escape and aliasing violations.

> Trace: D6
> Covers: Anonymous-by-default regions are source-level complete types, not hidden elision slots.

Named regions use `&[T, R]` or `&![T, R]` with an explicit `generic [R: Region]` parameter. A named region is required only when a signature must express a relationship that survives across an API boundary, such as returning a borrow tied to an input borrow.

> Trace: D6, D159/D188
> Covers: Named regions are explicit API relationships for rare escaping-borrow signatures.

A region cannot outlive the owner, temporary, or borrow scope that created it. A borrow value whose type contains an anonymous region may not be stored in a longer-lived place, returned from the function, captured by a closure or task beyond the region, placed into a global, or hidden in a container whose lifetime is not bounded by that region.

> Trace: D6, D72, D118/D197, D164/D248
> Covers: Anonymous-region borrow values are non-escaping and cannot be smuggled through storage or capture.

Named regions do not create ownership and do not permit arbitrary lifetime extension. They only name a relationship the checker can prove. If the owner or borrow source ends, moves, is destroyed, or enters a conflicting borrow state, every region derived from it ends or becomes unavailable according to the same rule.

> Trace: D6, D14, D187
> Covers: Region names express proven relationships; they do not override ownership state.

## Borrow Creation

`&place` creates an immutable borrow when `place` is addressable and no live mutable borrow conflicts with it. Multiple immutable borrows of the same referent may coexist while the referent is not mutably borrowed and is not moved or consumed.

> Trace: D14, D72
> Covers: Immutable borrow creation requires an addressable place and rejects conflicts with mutable access or movement.

`&!place` creates a mutable borrow when `place` is mutable, addressable, uniquely available, and not live under any immutable or mutable borrow. Exactly one mutable borrow of a referent may be live at a time.

> Trace: D14, D72, D187
> Covers: Mutable borrowing requires unique mutable access and excludes all other live access to the same referent.

A borrow of a field, index projection, or slice projection borrows the selected place and every owner path needed to keep that projection valid. The checker must prevent mutation, movement, destruction, or container reallocation that would invalidate the borrowed projection before the region ends.

> Trace: D34, D36/D106/D132, D77, D187
> Covers: Projection borrows carry enough owner-state restriction to prevent invalid field, element, slice, and iterator access.

A static literal may be borrowed because its storage duration is part of the literal contract. An ordinary rvalue temporary may be immutably borrowed only for an immediate non-escaping use in the same statement. An ordinary rvalue temporary may not be mutably borrowed.

> Trace: D72/D213
> Covers: Temporary borrowing is statement-scoped, non-escaping, and immutable-only for rvalues.

## Reborrowing

`&~borrow` creates an explicit reborrow from an existing mutable borrow. During the reborrow region, the original mutable borrow enters `SuspendedByReborrow` and cannot be used until the nested reborrow ends.

> Trace: D7b, D14, D187
> Covers: Reborrow creates nested temporary access and suspends the source mutable borrow.

When a call expects `&![T]` and the argument expression is already `&![T]`, the compiler may insert that same mutable reborrow automatically only through the elaboration pipeline. This is legal because the expected type leaves exactly one valid operation and the inserted node is checked like explicit `&~`.

> Trace: D7b, D87, D238-D240
> Covers: Auto-reborrow is a tautological typed completion, not a backend trick or unchecked aliasing exception.

When a call expects `&[T]` and the argument expression is `&![T]`, the compiler may insert a temporary immutable read reborrow of the referent. This is not subtyping, variance, or permanent weakening of the original mutable borrow. The mutable borrow is suspended while the read reborrow is live.

> Trace: D187, D238-D240
> Covers: Mutable-to-immutable use is a read reborrow with ordinary region and suspension behavior.

No implicit rule converts `&[T]` to `&![T]`, creates a borrow from a non-addressable value except the admitted immutable rvalue case, extends a temporary beyond its statement, or invents a named region relationship.

> Trace: D6, D72, D87, D187
> Covers: Borrow completion is a closed table and rejects widening, escaping, or guessed lifetime relationships.

## Control-Flow Joins

At every normal join point after `if`, `case`, `let...else`, loop bodies, and lowered sugar, the checker reconciles ownership states across all paths that can complete normally. A live linear value must have the same usable owner on every normally completing path, or the program is rejected.

> Trace: D15a, D38/D205/D206, D58/D191, D195, D238
> Covers: Branch joins merge only compatible continuing ownership states.

A path that exits through `return`, `break`, `continue`, `panic`, `todo`, or `unreachable` is checked at the exit. It does not have to provide a state for the later normal join because control does not reach that join.

> Trace: D43, D58/D191, D84, D121-D122
> Covers: Diverging and exiting paths are ownership-checked locally and do not fake normal continuation.

Inside loops, a linear value owned before the loop may not be consumed by the loop body unless the loop form's desugaring proves exactly one consumption across every possible zero-iteration, one-iteration, many-iteration, `break`, `continue`, and `return` path. Ordinary loops do not provide that proof for outside owners.

> Trace: D32/D249, D43, D195
> Covers: Repeated control flow cannot consume an outside linear owner an unknown number of times.

A linear value may be created inside a loop body. It must be discharged before each iteration path leaves the body through fallthrough, `continue`, `break`, `return`, `panic`, or lowered sugar. No iteration may depend on a previous iteration's hidden cleanup.

> Trace: D2, D32/D249, D43, D195, D246
> Covers: Loop-local linear obligations are per-iteration obligations with explicit cleanup or movement.

## `defer` And `errdefer` States

A `defer` that consumes a linear value transitions that value to `Deferred` at the registration point. The value is not consumed by ordinary execution at that moment, but it is reserved for the registered cleanup action. The checker treats duplicate consumption before scope exit as an error.

> Trace: D2, D2a, D246
> Covers: Ordinary deferred cleanup reserves ownership immediately and runs later in visible LIFO order.

A `Deferred` value may be immutably borrowed, and may be mutably borrowed only if the borrow ends before the deferred action can run and the deferred action still receives the value in a valid state. A deferred value may not be moved, returned, sent, reassigned, destroyed by another action, or registered again for consuming cleanup.

> Trace: D14, D187, D246
> Covers: Deferred ownership is still borrow-checkable, but cannot be stolen from the registered cleanup path.

An `errdefer` that consumes a linear value transitions that value to `ErrDeferred` for structured error exits from the scope. On success paths, the value remains an obligation: it must be moved, returned, destroyed, deferred, or otherwise discharged before the scope completes normally.

> Trace: D2b, D227, D246
> Covers: Error-only cleanup does not silently clean success paths.

If a scope exits through `or return` or `return Err(value)`, eligible `errdefer` and ordinary `defer` actions run in reverse registration order for that scope. If a scope exits through `break`, `continue`, `or break`, `or continue`, or `panic`, only ordinary `defer` actions run. If execution reaches TPOE, no user `defer` or `errdefer` action runs.

> Trace: D2a, D2b, D15a, D84, D208, D227
> Covers: Cleanup is selected by explicit exit category, with TPOE as immediate hard termination.

## Patterns And Destructuring

Pattern matching over a linear scrutinee consumes that scrutinee exactly once and transfers ownership of bound linear payloads into the selected arm. Every selected-arm path must discharge those bound payloads.

> Trace: D38/D205/D206, D98, D195
> Covers: Linear pattern matching moves ownership into explicit bindings instead of duplicating or discarding payloads.

Record destructuring must be total for the record form being destructured. If the record contains linear fields, each linear field must be bound and discharged. Partial record moves and hidden field drops are not part of Kyokai's pattern semantics.

> Trace: D35, D98, D206
> Covers: Record destructuring cannot leave linear fields behind in an unspecified state.

A `case` over a union must be exhaustive. Exhaustiveness is checked structurally, including nested patterns. A catch-all shape cannot hide possible linear payloads; linear alternatives must expose the payload names needed for ownership checking.

> Trace: D38/D205/D206
> Covers: Exhaustive matching remains ownership-visible for nested union payloads.

## Closures, Generators, And Tasks

A closure literal captures only what its explicit capture list names. Capturing a linear value by value moves it into the closure environment and normally makes the closure `CallableOnce`. Capturing a mutable borrow, or mutably using a by-value capture, selects `CallableMut`. Remaining captures select `Callable`.

> Trace: D118/D126/D197
> Covers: Closure environment ownership is explicit and determines the callable family statically.

A closure may not capture an anonymous-region borrow if the closure can outlive that region. A closure that stores, returns, or transfers a borrow must use a named region relationship admitted by the signature and still obey the owner-state rules.

> Trace: D6, D72, D118/D197
> Covers: Closure capture cannot extend borrow lifetimes by hiding them in an environment.

A generator is a named linear iterator type. Its suspended state owns whatever linear values remain live across `yield`, and its destroy operation must consume that suspended state exactly once. A generator may not suspend with live borrows that outlive their region.

> Trace: D198, D249
> Covers: Generator suspension is linear state, not a hidden coroutine runtime with erased ownership.

A `spawn` capture list moves owned captures by value or takes admitted shared captures by immutable borrow. Mutable-borrow capture is illegal. A borrowed task capture remains live until the child completes at the enclosing `join;`, and the parent cannot use the borrowed owner in a conflicting way during that interval.

> Trace: D88/D235/D252, D164/D248
> Covers: Task capture is explicit, structured, and checked through the parent-child lifetime ending at `join;`.

## Iterators And Containers

`for Pattern in expr do ... od;` owns or borrows the iterator state defined by the iterator protocol. If the iterator state is linear, the desugaring must consume that state exactly once on every loop exit path, including natural exhaustion, `break`, `continue`, `return`, `or return`, `or break`, `or continue`, and `panic` where ordinary cleanup runs.

> Trace: D32/D249, D2b, D246
> Covers: Linear iterators remain legal because loop lowering owns cleanup on every admitted exit path.

Each yielded item follows ordinary pattern and linearity rules. A yielded linear item must be bound and consumed on that iteration path. A yielded borrowed item cannot escape the region tied to the iterator or container borrow.

> Trace: D38/D205/D206, D195, D249
> Covers: Iteration does not relax item ownership or borrow escape rules.

Safe collection APIs should return element borrows or iterator values whose regions are tied to the container borrow. While such a borrow or iterator is live, mutating operations that could reallocate, remove, reorder, or otherwise invalidate the selected storage are rejected unless the container API specifies a stronger stable-address contract.

> Trace: D77, D85, D187
> Covers: Container invalidation is mostly statically prevented and otherwise must be specified per container contract.

Raw addresses, pointer-like values, and unsafe container internals do not weaken safe borrow rules. If an unsafe implementation exposes safe APIs, the safe API contract must preserve the same no-dangling, no-conflicting-access, and no-hidden-drop guarantees.

> Trace: D20/D20a/D20b, D73, D77, D245
> Covers: Unsafe internals may implement containers, but safe surfaces must keep Kyokai ownership guarantees.

## Pinning And Self-Reference

Ordinary safe Kyokai values are movable unless their type declares otherwise. Because ordinary moves are relocation, safe code may not construct self-referential values that depend on an internal address staying stable after movement.

> Trace: D89, D89a
> Covers: Safe self-reference is rejected under ordinary movable value semantics.

`Box[T]` provides ordinary heap indirection but does not by itself make `T` non-movable. Moving a `Box[T]` moves the owning handle; the allocation may remain at a stable address, but generic code cannot assume pinned semantics unless the API and type contract say so.

> Trace: D89a/D89b
> Covers: Heap indirection and first-class pinning are separate contracts.

A `pinned` type declaration and `PinBox[T]` provide the admitted stable-address boundary. Once initialized behind the pinning owner, a pinned value may not be moved by safe code. Generic containers, algorithms, and returns that relocate elements must require a `Movable` capability, bound, or contract and must reject pinned values unless they provide a pin-preserving representation.

> Trace: D89b, D82/D82a/D82b
> Covers: Non-movability is declaration-site visible and generic relocation must be explicit.

Pinning does not disable borrowing, destruction, capability rules, or unsafe-contract obligations. It only forbids relocation through safe movement. A pinned value still must be destroyed exactly once through an explicit path admitted by its API.

> Trace: D2, D89b, D195, D245
> Covers: Stable address is not a license to hide cleanup or bypass ownership checking.

## Diagnostics And Conformance

A linearity error must report the owner, the state it is in, the operation that caused that state, and the later operation or scope exit that made the program invalid. For branch and loop errors, diagnostics must identify the divergent ownership states across paths.

> Trace: D29, D216, D240
> Covers: Ownership diagnostics must explain the state transition and path conflict, not only say that linearity failed.

The conformance suite must include positive and negative tests for move-after-move, missing consumption, duplicate destruction, branch-state mismatch, loop consumption of outside owners, `ignore` over linear payloads, anonymous borrow escape, named-region returns, mutable alias rejection, read-reborrow suspension, auto-reborrow elaboration, rvalue borrow limits, deferred duplicate consumption, errdefer success-path obligations, linear iterator early exit, closure capture movement, task borrow capture until `join;`, container invalidation, and pinned-value relocation rejection.

> Trace: D6, D7b, D14, D72, D87, D187, D197, D205-D206, D238-D240, D246, D249
> Covers: These rules require explicit conformance coverage across ownership, borrow, cleanup, and escape boundaries.
