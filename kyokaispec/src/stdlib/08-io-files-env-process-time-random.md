# I/O, Files, Environment, Process, Time, Random, And Network

[Rikona Kurasaki / Mjoyufull]
The outside world is not a global variable Kyokai forgot to name. Files, terminals, clocks, entropy, processes, sockets, and the environment are authority. Authority moves through values.

> Trace: D48, D67, D171-D173, D211-D212, D255
> Covers: External-world stdlib APIs are capability-gated and do not use ambient authority.

## Capability Surface

Authority-bearing APIs require explicit capability values. `RootCapability` is minted only by runtime bootstrap for entrypoints that request it. Safe code derives narrower capabilities from root or receives them as parameters; imports do not grant authority.

> Trace: D48, D162, D211, D255
> Covers: Runtime bootstrap, not imports, is the root of safe authority.

| Capability | Governs | Borrow Shape | Notes | Trace |
| --- | --- | --- | --- | --- |
| `FileCapability` | Filesystem namespace and handle creation. | Usually `&!` because operations observe or mutate external state. | Relative paths require explicit `Directory` handles. | D171 |
| `EnvCapability` | Environment variable reads/writes/removal. | `&!` because the environment is mutable process state. | No global `getenv`. | D67 |
| `ProcessCapability` | Child process spawning, process handles, exit status. | `&!` for spawning/control. | Arguments use OS strings, not raw text by accident. | D85, D211 |
| `TerminalCapability` | Terminal streams and interactive terminal control. | `&!` for observation/control. | Ordinary streams use `Readable`/`Writable` handles. | D66, D211 |
| `ClockCapability` | Wall-clock time, sleeps, timers, deadlines. | `&!` for observable clock/suspension authority. | Pure `Duration` arithmetic is ungated. | D85, D211 |
| `RandomCapability` | OS entropy and cryptographic randomness. | `&!` because entropy source state is external. | Non-crypto PRNGs are separate deterministic values. | D231 |
| `NetworkCapability` | Sockets, listeners, DNS, network namespace. | `&!` for creation/connection. | Admitted only with explicit networking contract. | D85, D211-D212 |
| `SignalCapability` | Signal notification watchers. | `&!` for registration. | Synchronous fault signals remain runtime-fatal. | D95-D96, D256 |

> Trace: D48, D66-D67, D85, D95-D96, D162, D171, D211-D212, D231, D255-D256
> Covers: Common external capabilities state authority surface, borrow shape, and rule source.

## Filesystem And Streams

`File` and `Directory` are linear handles. Opening, creating, renaming, removing, metadata lookup, permission changes, symlink operations, and directory traversal require `FileCapability` or an explicit `Directory` handle with the necessary authority. Ordinary failures return `Result[..., IoError]`.

> Trace: D66, D171, D212
> Covers: Filesystem authority and failures are explicit.

`Readable` and `Writable` are byte-stream protocols. They do not imply filesystem authority and do not secretly allocate. A stream implementation states whether operations may block, whether partial reads/writes are possible, whether deadlines are supported, and what error type is returned.

> Trace: D66, D85, D91
> Covers: Stream protocols separate byte transfer from authority and blocking behavior.

| API Family | Ownership | Allocation | Failure | Capabilities | Platform | Edge Cases | Tests | Trace |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| File open/create | Borrows capability and path/base directory; returns linear handle. | Handle allocation internal; path conversions take allocator. | `IoError`, `PathError`, `AllocError` where applicable. | `FileCapability` or `Directory`. | Target filesystem rules documented. | missing path, permission denial, symlinks, races, unsupported kind. | positive/negative target tests. | D171, D212 |
| File read/write | Mutably borrows handle and buffer/sink. | None unless helper allocates output buffer. | `IoError`; partial progress stated. | Authority carried by handle. | Blocking and short read/write rules documented. | EOF, closed handle, interrupted operation, partial write. | short I/O, close, error injection. | D66, D85, D91 |
| Directory traversal | Linear iterator or explicit cursor. | Allocator explicit for owned entries. | `IoError`, `AllocError`. | Directory handle. | Ordering deterministic only if contract says sorted. | concurrent mutation, permission denial, symlink loops. | platform fixtures and property checks. | D77, D83, D171 |

> Trace: D66, D77, D83, D85, D91, D171, D212
> Covers: File and stream APIs publish ownership, allocation, failure, capability, platform, edge cases, and tests.

## Environment

Environment APIs require `EnvCapability`. `getEnv` returns `Optional` for absence. `setEnv` and `removeEnv` return `Result` because the host may reject mutation. Keys and values use OS-string aware contracts where the target cannot promise UTF-8.

> Trace: D30-D30a, D67
> Covers: Environment access is capability-gated and absence is ordinary data.

There is no global ambient environment lookup. If a function reads `$HOME`, its parameters or captured values show the authority.

> Trace: D67, D211
> Covers: Environment dependence is visible at API boundaries.

## Process

Process spawning requires `ProcessCapability`. Child arguments and environment overlays use OS string/path contracts, not accidental `String` conversion. Standard input/output/error are explicit handles. Exit status is returned as data; spawn or wait failures return typed errors.

> Trace: D30-D30a, D66, D85, D211
> Covers: Process APIs expose arguments, streams, environment, and failure categories explicitly.

Process handles are linear. Dropping, waiting, killing, detaching, and pipe cleanup semantics are specified per API. If a child may outlive the handle after detach, the contract says so.

> Trace: D77, D84, D85
> Covers: Process lifetime is part of the handle contract.

## Time

`Duration` is a pure value type. `Instant` is monotonic timestamp data. Arithmetic and comparison on `Duration` and `Instant` are pure where they do not observe a clock. Wall-clock reads, sleeps, timers, and deadline registration require clock authority.

> Trace: D85, D91, D211
> Covers: Time separates pure arithmetic from observable clock/suspension authority.

Clock APIs must state monotonicity, resolution, overflow behavior, cancellation/deadline behavior, and whether they can block. `SystemTime` conversions must state timezone and calendar scope rather than inheriting host locale behavior silently.

> Trace: D83, D85, D91
> Covers: Clock and calendar APIs have explicit determinism and platform contracts.

## Random

Cryptographic randomness requires `RandomCapability` and returns recoverable failure if the OS entropy source is unavailable. Non-cryptographic pseudo-random generators are ordinary deterministic values with explicit seed construction and reproducible replay.

> Trace: D83, D220, D231
> Covers: Crypto entropy and deterministic PRNGs are separate APIs.

Random APIs must state whether output is cryptographic, reproducible, deterministic by seed, or host-entropy backed. A fast PRNG, hash seed generator, or test generator must not be presented as cryptographic unless admitted under the crypto policy.

> Trace: D220, D231
> Covers: Randomness APIs do not blur security and testing use cases.

## Network

Networking is admitted only through explicit `NetworkCapability` contracts. Socket, listener, DNS, address, TLS, and poller integration APIs state blocking behavior, cancellation/deadline support, byte-order behavior, platform support, and ownership of handles and buffers.

> Trace: D91, D93, D117, D211-D212, D260
> Covers: Network APIs are capability-gated and byte-order-explicit.

Network handles are linear by default. Multi-task access requires explicit brokers, channels, or synchronization wrappers; raw capability-bearing I/O is not silently shared by hidden runtime locks.

> Trace: D100-D101, D212, D236
> Covers: Network handle sharing follows explicit concurrency and authority rules.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
The operating system is too powerful to be treated like background weather. Kyokai makes the handle visible, the capability visible, the path base visible, the block visible, and the failure visible.

> Trace: D67, D85, D171, D211-D212
> Covers: External-world APIs expose authority and failure instead of hiding them in globals.
