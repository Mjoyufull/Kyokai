# Concurrency And Memory Model

[Rikona Kurasaki / Mjoyufull]
Concurrency is where a program stops being a hallway and becomes a city at night. Doors open in different buildings. Lights change before anyone on the next street sees them. A message leaves one room and arrives in another with work still warm on it. If the language does not say what crossed, what waited, and what became visible, the rest is weather.

Kyokai does not hide a scheduler under the floor. It gives the program tasks, channels, synchronization objects, cancellation tokens, atomics, and pollers, then makes each boundary visible.

> Trace: D3, D3a/D3b, D90-D90a, D156, D164, D247, D252
> Covers: Concurrency is structured, explicit, and governed by a closed memory-order model.

This chapter defines tasks, task groups, spawn failure, task capture, SPSC channels, selection, cancellation, deadlines, pollers, signals, atomics, locks, happens-before, and the rejection of language-level async/await or hidden executors.

> Trace: D88, D91-D95, D100-D101, D141, D168, D183-D184, D234-D237, D248, D256, D258
> Covers: Every concurrency primitive states ownership, blocking, cancellation, failure, and memory-order consequences.

## Concurrency Model

A Kyokai task created by `spawn` runs as a 1:1 operating-system thread execution. Kyokai has no language-level M:N scheduler, green-thread runtime, virtual-thread layer, hidden task migration, or implicit scheduling boundary around foreign calls.

> Trace: D164
> Covers: Spawned tasks are OS-thread tasks and blocking behavior is honest.

A blocking operation blocks the calling task's OS thread unless the API contract explicitly says it uses a `Poller` or equivalent readiness mechanism. The language does not silently turn blocking operations into asynchronous suspension points.

> Trace: D156, D164, D234, D237
> Covers: Blocking is source/API visible and never hidden behind a runtime scheduler.

Kyokai defines no `async fn`, `await`, async block, async closure, async typeclass method, core `Future` abstraction, hidden executor, or global reactor. High-concurrency I/O is expressed through explicit `Poller` values and libraries built on top of them.

> Trace: D93, D156, D234
> Covers: Structured concurrency and pollers are the whole current concurrency substrate.

## Task Groups And Spawn

`spawn [captures] do ... od;` is legal only inside `taskgroup do ... join;`. The capture list is mandatory; `spawn [] do ... od;` is the zero-capture form.

> Trace: D88, D252
> Covers: Spawn is structured and never uses ambient closure capture.

`join;` is the blocking structured join point for the task group. Execution after `join;` begins only after every direct child task spawned in the group has completed. Nested task groups are legal; the outer group observes completion of its direct child task, and that child is responsible for its own nested group.

> Trace: D252
> Covers: Child lifetime is bounded by an explicit blocking join boundary.

A `spawn` statement produces no join handle and no completion value. If a child task needs to send `T` or `Result[T, E]` to a parent or sibling, it must do so through explicit channels or synchronization objects.

> Trace: D168, D252
> Covers: Task results are explicit data transport, not hidden join wrappers.

`panic` and TPOE in any task terminate the whole process according to the runtime-failure chapter. They are not child-task result values, not recoverable at `join;`, and not translated into `TaskError`.

> Trace: D84, D168, D253
> Covers: Fatal task failure remains process-fatal.

## Spawn Failure

Creating an OS thread may fail before a child begins execution. That failure is `ThreadSpawnError`, ordinary recoverable data handled at the `spawn` site. It is not TPOE, not `panic`, and not a child-task result.

> Trace: D235
> Covers: Thread resource refusal is explicit recoverable failure.

A fallible spawn must have an `else err do ... fi;` arm unless the enclosing task group has successfully reserved task capacity that covers the spawn. If spawn fails, the child body does not begin, no child is added to the task group, and no spawn-start happens-before edge exists.

> Trace: D235, D90a
> Covers: Failed spawn creates no child and no synchronization edge.

By-value captures transfer only after task creation succeeds. On spawn failure, captures remain available to the failure arm, including linear values that would have moved to the child on success. Borrow captures are not extended because no child exists.

> Trace: D88, D235, D248
> Covers: Spawn failure preserves ownership exactly.

## Capture And Task Transfer

A capture written as `name` captures by value. If `name` has `Free` type, the child receives its own value as of the spawn point and the parent may continue using its binding. If `name` has `Linear` type, ownership transfers to the child and the parent binding is consumed at the spawn point.

> Trace: D88, D89, D195
> Covers: By-value capture follows ordinary copy or move semantics.

A capture written as `&name` captures an immutable borrow. It is legal only for `Free` values and for the closed shared-access concurrency set: `Atomic[T]`, `Mutex[T]`, and `RwLock[T]`, unless a later explicit contract admits another synchronized primitive. The borrow lives until the child completes at `join;`.

> Trace: D88, D100, D248, D252
> Covers: Shared task capture is narrow, explicit, and scoped to structured join.

`&!name` capture is illegal in `spawn`. A mutable borrow cannot be shared into a child task.

> Trace: D88
> Covers: Spawn cannot smuggle cross-task mutable borrowing.

Task-boundary transfer is controlled by declaration or standard-library contract metadata: `task_transfer` or `task_local`. `Linear` by itself does not imply task-transfer permission. Safe Kyokai has no user-implemented `Send` or `Sync` marker typeclasses and no structural inference for task-transfer safety.

> Trace: D248
> Covers: Cross-task ownership is explicit metadata, not Rust-style auto-trait inference.

## Channels

Kyokai channels are single-producer, single-consumer. `Sender[T]` and `Receiver[T]` are unique linear endpoints. They are not cloneable, splitable, reference-counted, broadcast, MPSC, or MPMC endpoints.

> Trace: D3a, D101, D236
> Covers: Channel topology is SPSC and preserves single ownership.

Channel capacity is explicit at construction. `makeBoundedChannel[T](capacity)` creates a bounded channel and requires capacity at least one. `makeGrowableChannel[T](alloc, initialCapacity)` creates a growable channel with visible allocator use. `makeRendezvousChannel[T]()` creates a synchronous no-buffer handoff channel.

> Trace: D3a, D74, D183, D250-D251
> Covers: Capacity, allocation, and synchronous handoff are named at construction.

There is no bare default channel constructor. A channel's capacity is a semantic fact because it controls backpressure, allocation, possible deadlock, and possible allocation failure.

> Trace: D3a, D74, D183
> Covers: Channel capacity behavior is never hidden behind a default.

## Channel Operations

`sendBlocking(sender, value)` takes a mutable borrow of the sender and consumes `value` on success. On a bounded channel, it blocks until capacity exists or the receiver is closed. On a growable channel, it may allocate; allocation failure returns the value in the error payload.

> Trace: D3a, D74, D195
> Covers: Blocking send transfers ownership only on success and returns linear values on failure.

`recvBlocking(receiver)` takes a mutable borrow of the receiver and blocks until a value is available or the channel is closed and drained. Closure after drain is reported as `None` or the operation's specified empty/closed result.

> Trace: D3a, D146
> Covers: Blocking receive waits for data or drained closure.

`trySend` and `tryRecv` never block. Failed `trySend` returns the unsent value in the error payload. Failed non-transferring channel operations create no cross-task happens-before edge.

> Trace: D3a, D90a
> Covers: Nonblocking operations preserve ownership and create ordering only on actual transfer.

`closeSender(sender)` consumes the sender and signals closure. Receiver observation of closure cannot overtake earlier successful sends; buffered values sent before closure must be received first in FIFO order.

> Trace: D3a, D90a
> Covers: Sender close is ordered after prior successful sends.

For `Receiver[T]` where `T: Free`, `closeReceiver(receiver)` may discard buffered `Free` values. For `Receiver[T]` where `T: Linear`, there is no close-and-discard operation. The only completion path is `drain(receiver)`, which yields every buffered and later arriving value until sender closure and channel empty.

> Trace: D146, D195
> Covers: Linear channel payloads are never silently discarded during receiver teardown.

A rendezvous channel has no internal element buffer. Send and receive pair synchronously. A blocking send waits for a matching receiver, and a blocking receive waits for a matching sender or closed sender side. Deadline and cancellation variants are the explicit way to bound that wait.

> Trace: D183, D91
> Covers: Synchronous handoff is explicit and may wait indefinitely without bounded variants.

## Broker Topologies

Fan-in, fan-out, logging, work queues, and broadcast-like designs use explicit broker tasks over SPSC channels. Core Kyokai does not add cloneable endpoint types to hide many-producer or many-consumer state.

> Trace: D236
> Covers: Complex topology is visible source structure over SPSC endpoints.

Standard-library broker helpers may exist, but they must accept, construct, and own visible SPSC endpoints. They must document ordering, fairness or non-fairness, shutdown, backpressure, cancellation/deadline behavior, and linear-payload drain rules.

> Trace: D85, D236
> Covers: Broker helpers do not create hidden shared channel ownership.

## Select And `pick;`

A `select ... pick;` block waits on multiple admitted channel readiness operations. It contains one or more `when` arms and at most one `timeout(deadline)` arm.

> Trace: D92/D258
> Covers: Multi-channel waiting is a core block, not a macro or OS-poller bypass.

Each non-timeout arm names exactly one admitted blocking channel operation plus its result pattern. `select` waits until at least one arm is ready. Exactly one ready arm is chosen and exactly one arm body executes.

> Trace: D92/D258
> Covers: Select executes one selected communication path.

If multiple arms are ready, the language gives no fixed source-order priority. An implementation may choose any ready arm. Programs that require priority must encode priority explicitly outside `select`.

> Trace: D92/D258
> Covers: Select has non-priority ready-arm semantics.

Values, borrows, and ownership transfers associated with non-selected arms do not occur. The same linear endpoint may not appear in multiple arms of one `select`.

> Trace: D92/D258, D195
> Covers: Select does not speculatively move or borrow unchosen paths.

## Cancellation, Deadlines, And Plain Blocking

Cancellation is cooperative. Kyokai has no asynchronous thread cancellation, hidden signal injection, forced unwinding, or foreign-frame interruption to stop blocked work.

> Trace: D91, D237
> Covers: Cancellation never preempts arbitrary code.

A `CancellationToken` is a `Free` value backed by safe synchronization. A parent may pass immutable borrows of it to child tasks, and children explicitly poll or pass it to APIs that accept cancellation.

> Trace: D91, D3b, D88
> Covers: Cancellation is explicit data shared through admitted synchronization.

Plain blocking operations such as `readBlocking`, `writeBlocking`, `sendBlocking`, or `recvBlocking` may wait indefinitely if no peer, device, deadline, cancellation token, or close/error condition makes progress. The word `Blocking` is part of the source-level honesty contract.

> Trace: D11b, D91, D237
> Covers: Unbounded waiting is visible in API names and contracts.

Cancellation-aware or deadline-aware operations must make the extra exit path visible in the API name, parameters, or result type, such as `Cancelled` or `TimedOut`. They must use a `Poller`-compatible readiness path or an explicitly specified target readiness mechanism.

> Trace: D91, D93, D237
> Covers: Bounded blocking exits are explicit and readiness-backed.

Cancellation is ordinary structured control flow when code chooses to act on it. It runs `defer` according to the ordinary exit path and is not TPOE or `panic`.

> Trace: D2b, D91
> Covers: Cooperative cancellation is not fatal termination.

## Poller And Event Loops

`Poller` is a linear value representing an explicit OS readiness object such as an event queue. Creating a `Poller` requires the appropriate process or OS authority capability.

> Trace: D93, D211
> Covers: Polling authority and ownership are explicit.

Registering a handle with a `Poller` requires mutable borrows of the poller and the handle, or another contract that states exactly how registration owns and invalidates the involved values. Waiting on a poller is a visible blocking operation.

> Trace: D93, D234
> Covers: Readiness registration and waiting are explicit ownership operations.

An event loop is ordinary Kyokai code over a `Poller`. It is not a hidden runtime service. Handles used with it must be explicitly registered, and plain blocking APIs are not silently made cancellable by the presence of an event loop.

> Trace: D234, D237
> Covers: Reactor boundaries are library code, not hidden language machinery.

## Signals

Safe Kyokai provides no API to register an arbitrary Kyokai function as a signal handler. The safe surface is a capability-gated `SignalWatcher` that exposes a pollable readiness source compatible with the `Poller` model.

> Trace: D95/D256, D93
> Covers: Safe signal handling is notification-based and pollable.

A `SignalWatcher` may cover catchable external signals such as interrupt, termination, hangup, or user-defined signals. Safe Kyokai does not expose synchronous fault signals as recoverable events.

> Trace: D95/D256
> Covers: External signal watching is separate from fatal host faults.

Signal claims are process-global and exclusive per signal number. Creating a second safe watcher for an already claimed signal fails explicitly. Destroying a watcher consumes it, releases the claim, and restores the previous disposition according to its contract.

> Trace: D95/D256, D85
> Covers: Signal ownership and teardown are explicit.

## Atomics

`Atomic[T]` is a linear synchronization type in the closed set of types where shared immutable access may change storage. This is a named exception, not a general interior-mutability rule.

> Trace: D3b, D100, D184
> Covers: Shared mutation is admitted only for named synchronization primitives.

`T` for `Atomic[T]` is restricted to types with specified atomic support, such as fixed-width integers, `Index`, and `Bool`, as defined by the atomic type contract. Arbitrary user types are not atomically accessed by default.

> Trace: D3b, D141
> Covers: Atomic element types are constrained by the atomic contract.

Every atomic operation that accepts a `MemoryOrder` must receive a comptime-known order. `load` accepts only `Relaxed`, `Acquire`, or `SeqCst`. `store` accepts only `Relaxed`, `Release`, or `SeqCst`. Read-modify-write operations accept `Relaxed`, `Acquire`, `Release`, `AcqRel`, or `SeqCst` where the operation contract admits them.

> Trace: D3b, D141, D247
> Covers: Atomic order arguments are explicit, comptime, and operation-validated.

`compareExchange` and `compareExchangeWeak` return `CompareExchangeResult[T]`, not `Result`. CAS failure is an expected algorithmic branch. The failure ordering must be no stronger than the success ordering in the way the atomic contract specifies, and invalid order pairs are compile errors.

> Trace: D3b, D141, D247
> Covers: CAS semantics are typed and ordering-checked.

The C backend lowers atomics through conforming C11 atomics. It must not emit plain reads/writes for atomic storage, and `volatile` is not a substitute for atomic operations. If the selected toolchain cannot provide required atomic semantics, the build fails.

> Trace: D141, D228
> Covers: Backend atomic lowering preserves the language memory model.

## Mutexes And RwLocks

`Mutex[T]` and `RwLock[T]` are linear synchronization primitives. Locking takes an immutable borrow of the synchronization object so multiple tasks can hold shared borrows under structured concurrency.

> Trace: D100, D88
> Covers: Locks are shared through immutable borrow of the lock object.

A lock operation returns a linear guard. The guard must be consumed to unlock. Access to protected data happens through a mutable borrow of the guard, not through hidden global state.

> Trace: D100, D195
> Covers: Lock release is explicit linear guard consumption.

Mutexes and rwlocks require no external authority capability because memory synchronization inside the process is not host-resource authority. They still carry blocking, failure, fairness, poisoning or non-poisoning, and platform caveats in their standard-library contracts.

> Trace: D85, D100, D184, D211
> Covers: Synchronization is not external authority but still requires explicit API contracts.

## Memory Model

Kyokai's happens-before model is closed. Only synchronization edges named by this chapter create language-level cross-task ordering. Host or hardware behavior stronger than this chapter does not become portable Kyokai semantics.

> Trace: D90, D90a, D247
> Covers: Cross-task ordering exists only where the spec names it.

HB1: a successful spawn point happens-before the first operation of that child task. HB2: the last operation of a child task happens-before the parent continues past that child's structured `join;`.

> Trace: D90a, D252
> Covers: Structured task boundaries create start and finish ordering.

HB3: a successful send of a value on an SPSC channel happens-before the successful receive that returns that same value. HB4: `closeSender(sender)` happens-before receiver observation of closure after all prior successful sends have drained.

> Trace: D3a, D90a
> Covers: Channel transfer and closure observation create ordering.

HB5: release/acquire atomic synchronization on the same atomic location creates happens-before when the later acquire reads from the earlier release sequence. HB6: all `SeqCst` atomic operations and `SeqCst` fences participate in one total order consistent with task program order and per-location modification order. HB7: fences synchronize through the standard acquire/release/seqcst fence rules equivalent to C11/C++20.

> Trace: D3b, D90a, D141, D247
> Covers: Atomic and fence ordering follows explicit memory-order rules.

Failed channel operations create no happens-before edge. `Relaxed` atomic operations provide atomicity and per-location coherence but no happens-before edge by themselves. The compiler and backend may reorder within a task only when single-task semantics and every listed HB edge are preserved.

> Trace: D90a, D247
> Covers: Non-synchronizing operations do not accidentally order tasks.

## Data-Race Boundary

Safe Kyokai code cannot create two unsynchronized mutable accesses to the same non-atomic storage. Ordinary shared mutation across tasks must use channels, `Mutex[T]`, `RwLock[T]`, `Atomic[T]`, or another future primitive that explicitly names its synchronization and happens-before rules.

> Trace: D3, D3b, D90, D100, D247
> Covers: Safe cross-task mutation is limited to named synchronization primitives.

Raw pointers, volatile access, MMIO, inline assembly, foreign calls, and unsafe synchronization are outside the safe concurrency model unless wrapped by an unsafe contract and safe API that restores the same guarantees.

> Trace: D20, D73, D94, D245, D257
> Covers: Unsafe concurrency edges require explicit contracts.

`Rc[T]` and `Arc[T]` are not part of Kyokai. Shared ownership is rejected. Sequential sharing transfers ownership; concurrent sharing uses structured borrows of synchronization objects; graph-like sharing uses arenas, indices, or explicitly designed data structures.

> Trace: D101
> Covers: Reference-counted shared ownership is not a Kyokai primitive.

## Concurrency Guidance

The preferred order is channels first, mutexes and rwlocks second, atomics last. Channels express ownership transfer and coordination. Locks express genuinely shared mutable structures. Atomics express low-level flags, counters, sequence numbers, and implementation internals.

> Trace: D184
> Covers: The standard concurrency style has an explicit gradient.

This guidance does not change legality. A program may use any admitted primitive when its contract fits, but public APIs should choose the primitive whose ownership, blocking, cancellation, and ordering story is easiest to audit.

> Trace: D85, D184
> Covers: Guidance informs API design without adding hidden semantics.
