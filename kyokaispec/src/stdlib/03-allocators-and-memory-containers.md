# Allocators, Memory, And Containers

[Rikona Kurasaki / Mjoyufull]
Memory APIs are where a systems standard library either earns trust or starts lying. Kyokai makes the allocator visible, the failure visible, and the lifetime of stored values visible.

> Trace: D44, D74, D77, D201, D250-D251
> Covers: Allocation and container APIs use explicit allocator identity, explicit failure, and explicit invalidation contracts.

## Allocator Model

`Allocator` is an ordinary runtime value chosen explicitly by source code. There is no hidden default allocator, ambient thread-local allocator, lexical allocator context, or allocator type parameter machinery. APIs that allocate fresh owned storage take an allocator or use allocator identity already stored by the container.

> Trace: D44, D130, D201, D250-D251
> Covers: Allocator choice is explicit runtime state and not a hidden generic dictionary or ambient default.

Low-level allocator operations return `Result[..., AllocError]` for valid allocation requests that cannot be satisfied. Invalid size, alignment, layout, or allocator state is a contract violation unless the specific low-level API says it reports that case as data.

> Trace: D53, D74
> Covers: Allocation failure and invalid allocation requests are distinct failure categories.

Zero-sized allocation requests are legal. They return a stable non-null token suitable for later deallocation with the same allocator/layout contract, but not for ordinary dereference. Reallocation failure leaves the original allocation valid, owned by the caller, and unchanged.

> Trace: D74
> Covers: Zero-sized allocation and reallocation failure semantics are explicit.

| API Family | Ownership | Allocation | Failure | Capabilities | Tests | Trace |
| --- | --- | --- | --- | --- | --- | --- |
| `allocate`/`deallocate` | Allocator mutably borrowed; returned block is linear owned storage. | Direct allocator operation. | `AllocError`; invalid layout is TPOE. | None for ordinary heap; OS-backed allocators state authority separately. | zero-size, alignment, OOM, double-free compile/runtime boundary. | D44, D74 |
| `reallocate` | Old allocation remains owned until success transfers ownership to new block. | Direct allocator operation. | `AllocError` leaves old block valid. | Same as allocator. | growth, shrink, fail-preserves-old. | D74 |
| `must*` allocation helpers | Same as underlying operation. | Same as underlying operation. | OOM is named runtime-fatal behavior. | Same as underlying operation. | fatal path and naming diagnostics. | D74, D84 |

> Trace: D44, D74, D84, D85
> Covers: Allocator APIs state ownership, allocation, failure, authority, and tests.

## Core Containers

`Array[T, N]` is fixed-size inline storage with `N: Index`. `Span[T]` and byte/text views are non-owning borrowed views. `Buffer[T]` is growable owned storage that stores allocator identity when it may reallocate or destroy allocated storage.

> Trace: D55, D77, D201
> Covers: Fixed arrays, spans, and buffers have distinct ownership and allocation contracts.

A container that stores linear elements is itself linear unless a closed built-in rule says otherwise. Destroying or clearing a container with linear elements must consume or return every element according to the API contract. No container may silently drop linear payloads.

> Trace: D77, D146, D205-D206
> Covers: Containers preserve linear element obligations.

Mutation that may reallocate or reorder storage requires a mutable borrow of the container. Live element borrows, iterators, cursors, spans, or raw unsafe addresses are invalidated according to the container contract. Safe APIs must let the borrow checker reject common invalidation; remaining unsafe raw-address cases are documented per container.

> Trace: D77, D85
> Covers: Container invalidation is part of the public API contract.

| Container | Ownership | Allocation | Failure | Invalidation | Concurrency | Trace |
| --- | --- | --- | --- | --- | --- | --- |
| `Array[T, N]` | Owns `N` inline elements. | None after construction. | Construction failure only from element construction. | Inline borrows follow ordinary borrow rules. | Transfer follows element/task rules. | D55, D77 |
| `Span[T]` | Non-owning borrow view. | None. | Bounds TPOE for checked indexing. | Ends with borrow region; cannot outlive owner. | Not shareable beyond borrow rules. | D6, D77 |
| `Buffer[T]` | Owns elements and stored allocator identity. | Grows with stored allocator. | `AllocError` on growth; fatal variants named. | Reallocation invalidates views/raw addresses; mutable borrow excludes live views. | Linear unless synchronized wrapper says otherwise. | D44, D74, D77, D201 |
| `Box[T]` | Owns separately allocated `T`. | Allocates on construction, deallocates on destroy. | `AllocError`; moving box does not move pointee storage. | Borrowed pointee follows ordinary borrow rules. | Transfer follows `T` rules. | D44, D74, D89a-D89b |
| `PinBox[T]` | Owns stable-address pinned `T`. | Allocates in final pointee storage. | `AllocError`; no safe by-value extraction. | Pointee never relocates through safe API. | Transfer follows pinned/task rules. | D89b |

> Trace: D6, D44, D55, D74, D77, D89a-D89b, D201
> Covers: Core containers state ownership, allocation, failure, invalidation, concurrency, and pinned behavior.

## Naming And Allocation Effects

In-place mutation uses stored allocator identity. View operations allocate nothing and use `as*` names. Fresh owned results require an explicit destination allocator and use `to*In`, `cloneIn`, `collectIn`, or another `...In` name. Consuming conversions that reuse storage use `into*`; consuming conversions that allocate fresh destination storage use `into*In`.

> Trace: D11b, D201
> Covers: Container API names expose allocation and ownership effects.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
A hidden allocator is a hidden policy decision. A silent reallocation is a hidden invalidation. A dropped linear element is a broken promise. Kyokai containers make those promises public, because memory is not background scenery in systems code.

> Trace: D44, D74, D77, D201
> Covers: Kyokai memory containers expose allocator, ownership, and invalidation behavior directly.
