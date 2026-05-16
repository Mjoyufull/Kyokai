# Concurrency Rationale

[Rikona Kurasaki / Mjoyufull]
Concurrency does not need poetry from the runtime. It needs rules. Which task owns this value? Which operation synchronizes? Can this call block forever? What happens to a buffered linear value if a receiver closes? Kyokai asks those questions at the surface instead of letting a scheduler answer them in secret.

> Trace: D3-D3b, D90-D101, D146, D183-D184
> Covers: Kyokai concurrency uses structured tasks, explicit transfer, channels, locks, atomics, and written cleanup contracts.

## Structured Tasks

Kyokai uses `taskgroup`, `spawn`, and `join` because child work should have a lexical home. Captures are explicit. Failed spawn transfers no captures. Panic and TPOE are not child result values. If a child needs to communicate a value or error, it uses channels or synchronization.

> Trace: D88/D235/D252, D168, D235
> Covers: Task spawning, capture transfer, failure arms, and child communication are explicit.

The design borrows from structured concurrency research rather than from detached-thread habit. A task should not become an invisible resource leak merely because it was easy to start.

> Trace: D3-D3b, D252
> Covers: Structured task scopes tie child lifetime to visible source constructs.

## Channels First

Kyokai's primitive channel model is SPSC, with rendezvous and bounded forms. This keeps ownership transfer clear and makes the happens-before story small enough to specify. MPSC and MPMC behavior can be built as admitted libraries or brokers, but they do not become the primitive everyone must understand first.

> Trace: D3-D3b, D90, D146, D183
> Covers: Primitive channels transfer ownership through specified SPSC/rendezvous/bounded contracts.

Linear payloads make channel closure serious. A `Receiver[T: Linear]` cannot throw away buffered values. It must drain them or otherwise consume/return them exactly once.

> Trace: D146
> Covers: Receiver close/drain behavior preserves linear buffered values.

## Blocking And Cancellation

Cancellation is cooperative. Kyokai does not use hidden thread cancellation, signal injection, stack unwinding, or foreign-frame interruption to break a synchronous call. Plain blocking calls may wait indefinitely. Deadline-aware and cancellation-aware variants use explicit readiness mechanisms or equivalent named paths.

> Trace: D91/D237, D93/D234
> Covers: Blocking, cancellation, and deadlines are visible API choices.

This is a place where honesty beats cleverness. An operation that can block says so. An operation that can be cancelled says how. An operation waiting on multiple channels uses `select ... pick;` with one selected ready arm and no source-order priority guarantee.

> Trace: D92/D258
> Covers: Multi-channel waiting has a specified selection model.

## Locks And Atomics

Locks exist, but they are not the first design answer. `Mutex[T]` and `RwLock[T]` use linear guards because unlocking is a resource transition. Atomics exist, but relaxed operations do not synchronize by magic, and memory order is explicit.

> Trace: D90/D90a/D247, D100/D184
> Covers: Locks and atomics publish guard, happens-before, and memory-order rules.

Kyokai rejects `Rc[T]` and `Arc[T]` as general shared ownership. Sharing should happen through ownership transfer, structured borrows, synchronization, arenas, indices, or explicit unsafe contracts. Reference counting is too easy to make authority and lifetime feel ambient again.

> Trace: D101
> Covers: General reference-counted shared ownership is rejected.

## Pollers And Signals

`Poller` is explicit readiness state. Event loops are ordinary Kyokai code. Signals are capability-gated notification streams where that is safe, and synchronous fault signals remain runtime-fatal.

> Trace: D93/D234, D95/D256
> Covers: Polling and safe signal handling use explicit state and capability-gated watchers.

## Result

[Rikona Kurasaki / Mjoyufull]
Kyokai's concurrency model is intentionally less glamorous than a hidden executor. The ownership transfer is visible, the synchronization edge is named, the blocking behavior is in the contract, and the cleanup story remains linear.

> Trace: D90-D101, D146, D183-D184, D212
> Covers: Concurrency keeps ownership, synchronization, blocking, and authority sharing explicit.
