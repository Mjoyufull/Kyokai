# Iterators And Generators

[Rikona Kurasaki / Mjoyufull]
Iteration is control flow wearing a library face. It can borrow, move, allocate, suspend, and clean up. Kyokai treats it with the same suspicion and care as a loop written by hand.

> Trace: D32, D77, D146, D198, D249
> Covers: Iterator, receiver, and generator contracts preserve linear state and explicit loop cleanup behavior.

## Iterator Protocol

`Iterator[T]` is the stdlib protocol consumed by `for-in`. An iterator value may be `Free` or `Linear` according to its state. A linear iterator is consumed on every loop exit path: ordinary exhaustion, `break`, `continue` to an outer loop, `return`, `panic`, and other specified exits all trigger the iterator cleanup required by the loop elaboration.

> Trace: D32, D77, D249
> Covers: `for-in` over linear iterators has explicit exactly-once cleanup on every exit path.

An iterator's `next` operation returns absence, a value, or recoverable failure according to the protocol instance. The protocol must state whether yielded values are borrowed views, moved values, copied/free values, or linear owned values.

> Trace: D24, D77, D85
> Covers: Yield ownership is part of the iterator contract.

| API Family | Ownership | Allocation | Failure | Linearity | Concurrency | Tests | Trace |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Borrowing iterators | Borrow source collection/range. | None unless adaptor states otherwise. | `next` absence is data; checked invalid state is TPOE. | Iterator cannot outlive borrowed source. | Follows source borrow/thread rules. | lifetime rejection, mutation invalidation, exhaustion. | D32, D77, D249 |
| Owning iterators | Consume source collection or stream. | Uses stored allocation only for cleanup unless adaptor allocates. | Source-specific errors are explicit. | Must consume/drop/return remaining linear state exactly once. | Transfer follows iterator state rules. | early exit, panic path, remaining cleanup. | D77, D84, D249 |
| Iterator adaptors | Consume or borrow upstream as named. | Allocating adaptors take allocator or store it. | Callback/source failure explicit in signature. | Preserve upstream linear obligations. | Capability needs flow through callbacks. | map/filter/take/zip edge cases, callback failure. | D77, D85, D201 |
| Collectors | Consume iterator. | Fresh owned result takes destination allocator and uses `collectIn`. | `AllocError`; source failure if iterator exposes it. | On failure, already-collected linear elements and remaining iterator state are cleaned by contract. | Follows destination collection. | allocation failure, partial collection cleanup. | D74, D77, D201 |

> Trace: D24, D32, D74, D77, D84-D85, D201, D249
> Covers: Iterator API families publish ownership, allocation, failure, linearity, concurrency, and tests.

## For-In Elaboration

`for pattern in expr do ... od;` evaluates `expr` once, creates the iterator according to the selected protocol, and runs `next` until exhaustion or exit. Pattern binding follows ordinary pattern linearity rules. A pattern that would discard a linear yielded value is rejected.

> Trace: D32, D38, D205-D206, D249
> Covers: `for-in` respects ordinary pattern and linearity rules.

Mutating the underlying collection while a borrowing iterator is live is rejected by safe borrowing rules unless the collection exposes a cursor API with a written invalidation and mutation contract.

> Trace: D77
> Covers: Iterator invalidation is rejected or specified by the collection contract.

## Receivers And Channels

`Receiver[T: Linear]` cannot close by discarding buffered values. It must complete through explicit `drain` or an equivalent operation that consumes or returns every buffered linear element. `Receiver[T: Free]` may close and discard buffered values only when the receiver contract says so.

> Trace: D146, D183
> Covers: Channel receiver iteration preserves linear buffered values.

Receiver iterators are linear when they own channel state, buffered values, or cancellation handles. Blocking behavior, cancellation behavior, and deadline-aware variants are named in the receiver's contract.

> Trace: D91, D146, D183
> Covers: Channel iteration states blocking, cancellation, and cleanup behavior.

## Generators

Generators are named stackless pull iterators. They are not general coroutines, async functions, or hidden schedulers. A generator can suspend at `yield`, resume through `next`, and destroy its suspended state through the generator cleanup contract.

> Trace: D156, D198
> Covers: Generators are a narrow iterator facility, not a broad coroutine model.

Generator suspended state is linear when it contains linear values, borrows, capabilities, or handles. Destroying a suspended generator runs the specified cleanup for that state. A generator cannot outlive borrows captured into its state.

> Trace: D77, D118, D198
> Covers: Generator state follows ordinary capture, borrow, and linear cleanup rules.

| Generator API | Ownership | Allocation | Failure | Linearity | Platform | Tests | Trace |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Generator construction | Captures values by explicit generator declaration rules. | None unless captured state construction allocates. | Construction failure explicit in signature. | State linearity follows captured fields. | Platform-independent unless captures platform handles. | capture, borrow lifetime, linear state rejection. | D118, D198 |
| `next` | Mutably borrows or consumes generator state by protocol. | None unless body allocates. | Body failure type explicit if admitted. | Yielded value ownership stated per generator. | Same as body. | yield order, exhaustion, failure, cleanup. | D198, D249 |
| destroy/drop | Consumes generator state. | Deallocates state through stored allocator if any. | No recoverable failure; runtime-fatal only if contract says so. | Consumes all suspended linear state. | Same as captured resources. | early destroy and partial execution cleanup. | D77, D198 |

> Trace: D77, D118, D198, D249
> Covers: Generator APIs publish ownership, allocation, failure, linearity, platform, and tests.

## Eager Helpers

Iterator helpers that allocate use `...In` names and explicit allocators. Helpers that only return views or lazy adaptors allocate nothing unless their contract says otherwise. `collectIn`, `toArrayIn`, `joinIn`, and similar helpers must state partial-output cleanup on failure.

> Trace: D11b, D74, D201
> Covers: Iterator allocation is visible in names and signatures.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
A loop should not get more mysterious because it was made elegant. Kyokai lets iteration be pleasant, but the borrowed view, the moved value, the blocked receiver, the suspended generator, and the allocator all stay visible.

> Trace: D32, D77, D146, D198, D249
> Covers: Iterator ergonomics remain bound to explicit ownership and cleanup contracts.
