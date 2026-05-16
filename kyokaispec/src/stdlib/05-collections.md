# Collections

[Rikona Kurasaki / Mjoyufull]
A collection is not just a clever box. It owns values, borrows values, moves values, may allocate, may reorder, may invalidate, and may make performance promises people build whole systems on. Kyokai writes those promises down.

> Trace: D44, D77, D85, D176-D177, D201, D229
> Covers: Public collection APIs require explicit ownership, allocator, invalidation, equality/hash/order, and testing contracts.

## Collection Admission

Stable `Kyokai.Collections` modules include only collections with written semantic contracts, allocator behavior, invalidation behavior, complexity bounds, and property tests. Compatibility collections may exist for migration or legacy data structures, but preferred stable collections must state why they belong in the batteries-included surface.

> Trace: D85, D152, D220, D229, D243
> Covers: Collections enter the stdlib through the same admission process as other API families.

Kyokai has no general `Cloneable` or `Default` requirement. Collection APIs cannot fabricate elements, duplicate arbitrary elements, or discard linear elements through a generic bound that does not actually exist. Duplication and default construction stay type-specific and explicit.

> Trace: D176-D177
> Covers: Generic collection design does not smuggle in universal clone/default semantics.

## Core Collection Families

The admitted baseline families are sequence buffers, maps, sets, queues/deques, priority queues, and sorting/search helpers. More specialized structures require their own admission records.

> Trace: D152, D229
> Covers: Collection families are broad enough for systems programming but not open-ended folklore.

| API Family | Ownership | Allocation | Failure | Linearity | Invalidation | Determinism | Tests | Trace |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Sequence buffers | Own elements in indexed order. | Construction/growth uses explicit or stored allocator. | `AllocError`; checked index violations are TPOE. | Linear if element storage is linear. | Growth/reorder invalidates views by contract. | Preserves insertion order unless named operation reorders. | boundary, allocation failure, invalidation, linear cleanup. | D44, D74, D77, D201 |
| Maps | Own key/value pairs. | Construction takes allocator; mutation uses stored allocator. | `AllocError`; lookup absence returns `Optional`. | Linear when key or value is linear. | Rehash/removal invalidates stated cursors/views. | Iteration order explicitly stable, insertion-ordered, sorted, or unspecified-per-run by contract. | property tests against reference model, hash collision tests. | D77, D83, D176-D177, D229 |
| Sets | Own values as keys. | Same as maps. | `AllocError`; membership absence is data. | Linear when elements are linear. | Same as map family. | Same order rule as backing set contract. | algebraic set properties, collision tests. | D77, D83, D229 |
| Queues/deques | Own elements in removal order. | Construction/growth uses allocator contract. | `AllocError`; pop empty returns `Optional`. | Linear if elements are linear; `clear` must consume or return all elements. | Growth invalidates views; cursors documented. | FIFO/deque order is deterministic. | sequence-model properties, wraparound, allocation failure. | D77, D146, D201 |
| Priority queues | Own elements ordered by comparator. | Construction/growth uses allocator contract. | `AllocError`; pop empty returns `Optional`. | Linear if elements are linear. | Mutation invalidates heap views. | Equal-priority order is specified or explicitly unspecified. | heap invariant, comparator edge cases. | D77, D85, D229 |
| Sorting/search helpers | Borrow or mutably borrow ranges; consuming variants named. | In-place variants state scratch allocation; fresh results take allocator. | `AllocError` for allocating algorithms; comparator failure only if signature exposes it. | Must not duplicate or drop linear elements. | Mutable range borrow excludes live element borrows. | Stability and ordering semantics stated per function. | permutation, ordering, stability, adversarial comparator tests. | D77, D85, D201 |

> Trace: D44, D74, D77, D83, D85, D146, D176-D177, D201, D220, D229
> Covers: Collection API families publish ownership, allocation, failure, linearity, invalidation, determinism, and test contracts.

## Equality, Hashing, And Ordering

Collections that compare, hash, or order elements must name the required protocol. Equality and ordering callbacks are pure unless their type explicitly accepts capabilities. Hashing must state seed behavior and reproducibility behavior. A deterministic build or test mode may require explicit seed injection rather than ambient process randomness.

> Trace: D83, D85, D229
> Covers: Collection determinism and hash seeding are part of the public contract.

Hash maps and hash sets must define behavior under collisions. Collision handling cannot introduce memory unsafety, cannot drop linear values, and cannot turn lookup absence into fatal behavior. Denial-of-service resistance, reproducible iteration, and deterministic tests are separate contract fields, not guesses.

> Trace: D73, D77, D83, D85
> Covers: Hash collection collision behavior is specified and tested.

## Linear Elements

When a collection owns linear elements, all removal, replacement, drain, retain, clear, destroy, and error paths must consume, return, or keep each element exactly once. An API that may discard buffered linear elements is rejected unless it is restricted to `Free` elements or exposes an explicit drain/destruction contract.

> Trace: D77, D146, D205-D206
> Covers: Collections preserve exactly-once linear ownership obligations.

Bulk operations over linear elements use callbacks or receiver functions that state what happens if the callback fails, panics, or terminates early. The remaining collection state after early exit is part of the contract.

> Trace: D77, D84, D85
> Covers: Collection callback APIs state cleanup and remaining-state behavior.

## Examples And Tests

Each stable collection module includes examples for construction, insertion, lookup, iteration, mutation during borrowed views, removal, drain, allocation failure, and linear payload handling where applicable. Property tests compare against a simple reference model whenever the data structure has algebraic behavior.

> Trace: D220, D229
> Covers: Collection examples and property tests are required admission evidence.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
Collections are where a language often starts whispering. Kyokai does not whisper. If a map can rehash, the invalidation is written. If a queue can hold a linear handle, the drain story is written. If a hash seed changes iteration, the determinism story is written.

> Trace: D77, D83, D85, D229
> Covers: Collection behavior remains explicit instead of living as accidental implementation custom.
