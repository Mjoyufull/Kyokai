# Capabilities And Authority

[Rikona Kurasaki / Mjoyufull]
Authority in Kyokai is not a smell in the air. It is not something a function wakes up owning because it happens to run inside the same process. It is a value with a boundary around it. If code wants the filesystem, the clock, the network, a process handle, an unsafe operation, or a raw device edge, the authority has to cross the line in source.

That is the point. The reader should be able to look at a signature and see which doors the code can open. A dependency that never receives the key cannot quietly walk through the building at night.

> Trace: D20, D48, D67, D162, D211, D212, D255
> Covers: External authority is tracked through explicit capability values, not ambient process permission.

This chapter defines capability declarations, root authority, acquire/derive/split/borrow/surrender/transfer rules, unsafe authority, FFI authority flow, task behavior, tests, plugins, dynamic loading, raw I/O handles, and broker patterns.

> Trace: D20/D20a/D20b, D85, D211, D248, D255
> Covers: Capabilities connect language ownership, unsafe boundaries, standard-library contracts, and toolchain audit surfaces.

## Authority Boundary

Kyokai's built-in effect tracking boundary is external authority. The type system tracks access to host resources through capabilities. It does not type-track divergence, non-termination, blocking, transitive allocation, or possible TPOE; those facts are surfaced by API names and contracts.

> Trace: D211
> Covers: Capabilities are the language's authority effect boundary; other effects remain contract/naming obligations.

A function that can observe or mutate the host outside ordinary memory must receive authority explicitly through a capability, a handle derived from a capability, or an unsafe wrapper whose contract names the authority it uses. No safe function has ambient filesystem, network, environment, clock, process, terminal, random, signal, dynamic-loader, or unsafe authority.

> Trace: D20, D67, D85, D211
> Covers: Host authority is explicit in parameters, owned handles, or audited unsafe wrappers.

A capability controls permission to ask for an operation; it is not a promise that the operation will succeed. Permission failure, missing files, refused connections, timeouts, OS resource exhaustion, and similar environmental outcomes remain explicit `Result` or `Optional` data where the API contract says so.

> Trace: D24, D74, D85, D211
> Covers: Authority and fallibility are separate contracts.

## Capability Declarations

A `capability` declaration defines a sealed linear authority type. It has no user-visible fields, no literal form, no ordinary constructor, and no structural record representation that source code can build.

> Trace: D255
> Covers: Capabilities are first-class sealed authority tokens, not forgeable records.

Capability values are `Linear` unless a specific built-in capability contract says otherwise. They cannot be copied, cloned, implicitly duplicated, ignored while live, or fabricated by generic code.

> Trace: D195, D255
> Covers: Authority is single-owner by default and follows linear checking.

A capability declaration may state task-boundary metadata such as `task_transfer` or `task_local`. If no safe transfer contract exists for a linear capability, safe code may not move it to another task.

> Trace: D248, D255
> Covers: Capability thread affinity and transfer are explicit declaration or contract facts.

Unsafe modules do not gain the right to construct capabilities directly. `pragma Unsafe_Module` may expose operations that use raw host APIs, but it may not manufacture sealed capability values from raw bits, empty records, pointer casts, or representation tricks.

> Trace: D20, D245, D255
> Covers: Unsafe authority does not include capability forgery.

## Root Capability

At process startup, the runtime may mint a single `RootCapability` value for an entrypoint that declares it. An entrypoint may also choose to take no root authority.

> Trace: D48/D162, D255
> Covers: Root authority originates only at runtime bootstrap and only when the entrypoint asks for it.

`RootCapability` is the broadest safe authority token. It is not ambient. If `main` does not receive it, safe source code cannot later acquire it.

> Trace: D48/D162, D211
> Covers: Root authority is explicit entrypoint state, not global process state.

The root capability may be borrowed to derive narrower capabilities, moved to an owner that will manage authority, or surrendered through an explicit consuming operation. Once moved or surrendered, the previous binding cannot be used.

> Trace: D89, D195, D255
> Covers: Root authority obeys ordinary linear movement and consumption.

A program should split root authority early into narrow authority values and pass only those values to the code that needs them. This is a design rule for safe APIs: narrow capabilities are preferred over broad authority.

> Trace: D85, D211, D255
> Covers: Capability APIs should minimize authority by construction.

## Acquire, Derive, Split, And Surrender

Acquiring a capability means calling an API that takes proof of broader authority and returns a narrower authority value or handle. The acquire operation must state its required authority, ownership behavior, failure mode, and whether the returned value is `task_transfer` or `task_local`.

> Trace: D67, D85, D211, D248
> Covers: Capability acquisition is explicit, contract-documented, and fallible where the host can refuse.

Deriving authority means producing a narrower capability or handle from a broader one. Derivation may consume the source authority, borrow it immutably, borrow it mutably, or return a split pair. The API signature decides which motion occurs and must preserve linear ownership.

> Trace: D14, D89, D195, D255
> Covers: Capability derivation is ordinary ownership/borrow movement with no hidden copy.

Splitting authority creates two or more narrower authority values whose combined reach is no broader than the source. A split operation must consume or mutably borrow the broader capability according to its signature, and it must specify whether the source remains usable after the split.

> Trace: D85, D195, D255
> Covers: Splitting authority is explicit and cannot duplicate broader authority accidentally.

Surrendering authority consumes the capability and releases or invalidates the associated authority according to its contract. After surrender, the value is destroyed or moved and the old binding is unusable.

> Trace: D2, D89, D195, D255
> Covers: Authority teardown is explicit linear consumption.

Destroying a capability is not the same as undoing all host effects it permitted. Closing a file handle, releasing a signal claim, or destroying an environment capability has only the cleanup semantics stated by that API.

> Trace: D85, D211, D212
> Covers: Capability cleanup is API-specific and not a magic rollback system.

## Borrowing Authority

An immutable borrow of a capability allows only operations whose signatures accept immutable authority. Those operations may observe host state only when the capability contract admits observation through shared access.

> Trace: D14, D67, D211
> Covers: Borrow kind controls what authority-bearing APIs may do.

A mutable borrow of a capability is required for safe production operations that mutate externally visible host state or advance linear handle state, unless the capability type's contract explicitly admits another access pattern.

> Trace: D14, D67, D212
> Covers: Mutating external state normally requires unique authority access.

Borrowing a capability does not copy it, does not extend its lifetime past the borrow region, and does not let a callee keep authority after the borrow ends. Storing or returning borrowed authority requires an admitted named-region relationship.

> Trace: D6, D14, D187, D255
> Covers: Borrowed authority cannot escape ordinary region rules.

Raw terminal, file, socket, and similar I/O handles are linear authority-bearing objects. Safe Kyokai does not insert hidden mutexes to make them shared across tasks.

> Trace: D212
> Covers: Raw I/O authority remains exclusive by handle and has no hidden synchronization.

## Authority Across Tasks

A capability may be captured by value into `spawn` only if its declaration or standard-library contract says it is `task_transfer`. The move happens only after task creation succeeds. On spawn failure, the value remains available to the failure arm.

> Trace: D88, D235, D248
> Covers: Owned task transfer of authority is explicit and failure-safe.

A capability marked `task_local` may not be captured by value into a child task, sent through a channel, or transferred to another task by safe code. A thread-affine host resource must stay with its owning task or be accessed through an explicit broker or synchronized wrapper whose contract says how that is safe.

> Trace: D248, D255
> Covers: Thread-affine authority cannot cross task boundaries silently.

Shared capture by immutable borrow is not generally available for capabilities. It is legal only for `Free` values and the closed synchronized set named by the concurrency chapter, or for a capability wrapper whose own contract explicitly admits shared access.

> Trace: D88, D100, D212, D248
> Covers: Capabilities do not become shared-safe merely because they are effectful.

If multiple tasks need one raw destination, the default safe pattern is a broker task that owns the authority-bearing handle and receives messages over channels. A synchronized writer type may exist only as a separate explicit abstraction with documented interleaving guarantees.

> Trace: D212, D236
> Covers: Multi-task authority use is brokered or separately synchronized, not hidden inside raw handles.

## Unsafe Capability And Unsafe Modules

`UnsafeCapability` is explicit authority to call APIs that are outside safe Kyokai's ordinary guarantees. It is not root authority and does not grant access to every host resource.

> Trace: D20, D20a/D20b, D245
> Covers: Unsafe authority is a specific capability, not ambient permission to do anything.

Unsafe operations must live behind source-level unsafe contracts that state what invariants the unsafe code assumes, what it preserves for safe callers, what capabilities it uses, what foreign calls or raw operations it performs, and how ownership crosses the boundary.

> Trace: D20, D245
> Covers: Unsafe contracts are auditable source obligations.

A safe wrapper around unsafe or foreign code must expose a normal Kyokai signature that accounts for authority, ownership, borrowing, failure, allocation, blocking, and platform behavior. The safe wrapper may not hide authority use behind a pure-looking function.

> Trace: D20/D20a/D20b, D85, D211, D242
> Covers: Unsafe wrappers translate raw behavior into explicit Kyokai contracts.

The build system must be able to enumerate unsafe modules and authority-bearing imports through package metadata and interface artifacts so audits can find the real boundary.

> Trace: D79, D83, D218, D245
> Covers: Unsafe and authority surfaces are discoverable by tools.

## FFI Authority Flow

Foreign code is permissionless from Kyokai's point of view. Safe capability guarantees exist only at the Kyokai wrapper boundary unless the wrapper proves and documents a stricter contract.

> Trace: D20/D20a/D20b, D73, D242
> Covers: FFI is an external trust boundary.

A foreign declaration that can touch host resources must be inside an unsafe module and must be wrapped by safe Kyokai APIs that require the relevant capability or authority-bearing handle. Calling raw foreign code is never proof that the caller had authority.

> Trace: D20, D67, D211, D245
> Covers: FFI cannot bypass capability requirements in safe surfaces.

Raw FFI cannot take or return Kyokai `Linear` values by value as if C understood Kyokai ownership. Ownership transfer across FFI must be represented by wrapper-owned handles, explicit unsafe contracts, or ABI types whose semantics are stated by the wrapper.

> Trace: D20/D242/D242a, D245
> Covers: Foreign code cannot pretend to participate in Kyokai linearity without a wrapper contract.

## Tests, Tools, Plugins, And Dynamic Loading

Test code receives no ambient authority. Test harness APIs may provide controlled capabilities, temporary directories, fake clocks, deterministic random sources, or process authority only when the test request names that authority.

> Trace: D28, D67, D85, D211
> Covers: Tests use explicit authority just like production code.

Build-time plugins, code generation, documentation extraction, LSP analysis, package scripts, and dynamic loading do not receive ambient host authority by default. Any execution authority must be declared in the manifest or tool command contract and audited as a toolchain capability boundary.

> Trace: D224, D245, D255
> Covers: Tooling execution is authority-gated and not hidden in package metadata.

Dynamic loading, when supported by the toolchain and target contract, requires explicit loader/process authority and an unsafe or audited wrapper boundary. Loading code does not grant that code new capabilities unless the host passes them explicitly.

> Trace: D20, D85, D211, D245
> Covers: Dynamic code receives authority only by explicit value passing.

## Standard Capability Families

Filesystem, environment, process, clock, randomness, terminal, network, signal, dynamic-loader, and unsafe authority are separate capability families. A broad capability may derive narrower capabilities, but no narrower capability can derive a broader one unless its contract explicitly says it carries that broader authority.

> Trace: D67, D85, D95/D256, D211, D255
> Covers: Standard authority families are separated and hierarchical where appropriate.

Environment-variable access requires `EnvCapability`. There is no ambient safe `getenv`, `setenv`, or environment iteration.

> Trace: D67
> Covers: Environment access is capability-gated.

Clock access is capability-gated where precision or timing observation can expose side channels. Standard-library clock APIs must state precision, monotonicity, determinism, and authority requirements.

> Trace: D85, D211
> Covers: Time observation is authority-bearing when the API contract says so.

Signal watching is capability-gated and notification-based. Safe Kyokai exposes `SignalWatcher` as a pollable authority-bearing value, not arbitrary signal callbacks.

> Trace: D95/D256
> Covers: Signal authority is explicit and safe signal handling is pollable.

## Forgery Rejection Table

| Attempt | Result | Trace |
| --- | --- | --- |
| Construct a capability with record syntax | Compile error. | D255 |
| Pattern-match a capability to inspect fields | Compile error. | D255 |
| Use a literal or zero value as a capability | Compile error. | D255 |
| Copy or clone a capability | Compile error unless a specific capability is explicitly `Free`, which ordinary authority capabilities are not. | D195/D255 |
| Cast raw bits or pointers into a capability in unsafe code | Compile error or unsafe-contract violation; safe capability is not produced. | D20/D245/D255 |
| Import an unsafe module and gain its authority | Compile error or ordinary visibility failure; imports do not grant capability values. | D20/D245 |
| Send a `task_local` capability to a child task | Compile error. | D248 |
| Share a raw I/O capability across tasks through hidden synchronization | Rejected; use broker or explicit synchronized wrapper. | D212/D236 |
