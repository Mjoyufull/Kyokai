# Concurrency Primitives

[Rikona Kurasaki / Mjoyufull]
Concurrency is where invisible behavior becomes expensive. A hidden share, a forgotten close, a lock nobody knows can block, a task boundary that moved authority without saying so: Kyokai gives these things names before they can become habits.

> Trace: D3-D3b, D90-D101, D146, D164, D183-D184, D212
> Covers: Stdlib concurrency primitives follow structured tasks, explicit transfer, SPSC channels, locks, atomics, pollers, and capability-sharing rules.

## Primitive Set

The stable primitive set includes structured task support from the language, SPSC channels, rendezvous and bounded channels, `Mutex[T]`, `RwLock[T]`, `Atomic[T]`, fences, `Poller`, `SignalWatcher`, and explicit broker/synchronized-wrapper patterns. Kyokai does not provide `Rc[T]` or `Arc[T]` shared ownership primitives.

> Trace: D3-D3b, D93, D95, D100-D101, D146, D183-D184
> Covers: The concurrency surface is explicit and rejects hidden reference-counted sharing.

MPSC and MPMC queues are not the primitive channel model. If admitted, they are built as library structures with their own contracts or brokers over simpler primitives. They must state ownership, fairness, memory ordering, wakeup, close, and drain behavior.

> Trace: D3-D3b, D90, D146, D183
> Covers: Multi-producer/multi-consumer behavior is not silently inherited from the primitive channel contract.

## Channels

SPSC channels transfer ownership between tasks. Rendezvous channels perform synchronous handoff. Bounded channels require capacity at least one. Channel close and receiver drain behavior must preserve linear payloads.

> Trace: D3-D3b, D146, D183
> Covers: Channel capacity, transfer, close, and drain rules are explicit.

| API Family | Ownership | Allocation | Failure | Linearity | Concurrency | Tests | Trace |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Channel construction | Returns sender/receiver linear endpoints. | Allocator explicit for buffered channels. | `AllocError`; invalid capacity is TPOE. | Endpoints linear; payload transfer consumes sender-side value. | SPSC unless module states otherwise. | capacity, close, transfer, allocation failure. | D74, D146, D183 |
| Send/receive | Mutably borrows or consumes endpoint as named. | None after channel construction unless contract says. | Closed/cancelled/deadline cases are data. | Linear payload moved exactly once. | Establishes channel happens-before edge. | ordering, close, blocking, cancellation. | D90-D91, D146 |
| Receiver drain | Consumes or mutably borrows receiver by contract. | None unless collector allocates. | Source error and allocation error explicit. | Required for `Receiver[T: Linear]` completion. | Same as receiver. | buffered linear drain, early exit cleanup. | D146, D249 |

> Trace: D74, D90-D91, D146, D183, D249
> Covers: Channel APIs publish ownership, allocation, failure, linearity, concurrency, and tests.

## Locks

`Mutex[T]` and `RwLock[T]` are linear synchronization primitives. Lock guards are linear borrowed-access values. Unlock occurs by consuming the guard through the ordinary linear cleanup path; APIs must state whether lock acquisition blocks, can return cancellation/deadline results, or can fail due to poisoning-like state. Kyokai does not inherit poisoning semantics by default.

> Trace: D77, D91, D100, D184
> Covers: Lock and guard behavior is linear and blocking-explicit.

Sharing capability-bearing handles through locks is allowed only when the wrapper contract states the authority-sharing policy. Raw capability-bearing I/O remains exclusive by handle unless a broker or synchronized wrapper makes sharing visible.

> Trace: D100, D212, D236
> Covers: Locks do not hide authority sharing.

| Lock Family | Ownership | Allocation | Failure | Linearity | Concurrency | Tests | Trace |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `Mutex[T]` | Owns `T`; guard borrows protected value. | Construction may allocate only if contract says and allocator is explicit. | Lock interruption/deadline explicit; ordinary unlock no failure. | Guard linear; protected linear values cannot be duplicated. | Mutual exclusion and happens-before on unlock/lock. | contention, cancellation, guard cleanup. | D90-D91, D100 |
| `RwLock[T]` | Owns `T`; read/write guards borrow. | Same as mutex. | Same as mutex. | Write guard exclusive; read guards obey `T` sharing rules. | Reader/writer fairness policy documented. | readers, writer exclusion, starvation policy. | D90-D91, D100, D184 |

> Trace: D90-D91, D100, D184, D212
> Covers: Lock APIs publish ownership, allocation, failure, linearity, concurrency, and tests.

## Atomics And Memory Ordering

`Atomic[T]` is admitted only for types with a supported atomic representation. Memory order is an explicit argument or fixed by the API name. Relaxed operations do not synchronize by themselves. Fences and compare/exchange operations state exact ordering requirements and failure ordering rules.

> Trace: D90-D90a, D141, D247
> Covers: Atomics use a closed memory-order model and supported lowering contract.

Atomic APIs do not grant ownership sharing. They operate on admitted scalar/atomic representations; shared access to larger resources still uses channels, locks, brokers, or explicit unsafe contracts.

> Trace: D90, D100-D101, D141
> Covers: Atomics are synchronization tools, not hidden shared ownership.

## Pollers, Signals, And Blocking

`Poller` is explicit linear readiness state. Event loops are ordinary Kyokai code over pollers. Plain blocking operations may wait indefinitely; deadline and cancellation-aware variants are separate named APIs returning explicit cases.

> Trace: D91, D93-D94, D237
> Covers: Readiness, cancellation, and indefinite blocking are visible API choices.

`SignalWatcher` is capability-gated and notification-based. Synchronous fault signals are runtime-fatal and are not delivered as ordinary safe events.

> Trace: D95-D96, D256
> Covers: Safe signal handling is narrow and explicit.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
Kyokai wants concurrency you can audit from the signature and the cleanup path. The task transfer is named. The channel drain is named. The lock guard is linear. The atomic order is written where the operation happens.

> Trace: D90-D101, D146, D183-D184, D212
> Covers: Concurrency primitives expose transfer, cleanup, blocking, ordering, and authority-sharing behavior.
