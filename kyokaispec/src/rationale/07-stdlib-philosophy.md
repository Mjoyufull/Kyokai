# Standard Library Philosophy

[Rikona Kurasaki / Mjoyufull]
A language can have a beautiful core and still betray the programmer in the library. Kyokai's stdlib therefore has to speak the same language as the compiler: ownership, allocation, failure, capability needs, platform behavior, determinism, tests, and implementation boundary all written in public.

> Trace: D85, D152, D229
> Covers: The standard library is batteries-included and every public API family carries explicit contract fields.

## Batteries Included, But Admitted

Kyokai wants a serious systems standard library: results, optionals, formatting, allocators, memory containers, text, bytes, paths, collections, iterators, math, I/O, filesystem, environment, process, time, random, concurrency, crypto, testing helpers, and transitional FFI tracking. That scope is large, so admission has to be strict.

> Trace: D152, D229-D232
> Covers: Broad stdlib scope is paired with admission records, edge cases, tests, and compatibility policy.

A useful API is not automatically a stable API. Stable admission requires contract fields, edge cases, tests or external vectors where needed, implementation policy, compatibility notes, and platform behavior. The rule is not paperwork; it is how Kyokai keeps the stdlib from becoming a second undocumented language.

> Trace: D85, D220, D229, D243
> Covers: Stable stdlib APIs require written contracts, evidence, and compatibility boundaries.

## RIIK With Discipline

RIIK means Rewrite-It-In-Kyokai, not rewrite-it-carelessly. Pure computation should be safe native Kyokai by default, especially containers, text handling, parsing, formatting, hashing, ordinary algorithms, and math once admitted. FFI remains correct at OS/hardware boundaries, for mature reviewed libraries, and as tracked bootstrap bridge code.

> Trace: D64, D229-D230
> Covers: Pure computation defaults to safe native Kyokai while FFI remains for explicit boundaries and tracked transitional use.

The reason is practical, not vanity. A pure Kyokai implementation can obey Kyokai's ownership, allocation, error, and no-language-UB contracts directly. A foreign implementation must be wrapped, audited, tested, and tracked so the trust boundary is visible.

> Trace: D20, D245, D230
> Covers: FFI-backed stdlib code requires unsafe boundaries and safe wrappers.

## Allocation And Containers

There is no hidden allocator. Fresh owned results take explicit destination allocators. Containers store allocator identity when they need it for growth or destruction. Fatal allocation helpers exist only under names that announce fatal behavior.

> Trace: D44, D74, D201, D250-D251
> Covers: Stdlib allocation is explicit and recoverable by default.

Collections and iterators preserve linear payloads. A map cannot quietly drop a linear value during rehash. A receiver cannot close over a buffer full of linear values. An iterator cannot die on `break` without consuming its state as specified.

> Trace: D77, D146, D249
> Covers: Containers, channels, and iterators preserve exactly-once linear obligations.

## Math And Crypto

Numerics and crypto are where confidence needs evidence. Math APIs publish accuracy contracts and test against oracles. Crypto APIs require modern external specs, vectors, side-channel claims, review evidence, and explicit randomness/key cleanup behavior.

> Trace: D231-D232
> Covers: Math and crypto stdlib APIs need stronger external evidence than ordinary helper code.

Kyokai can write native math and native crypto, but the admission bar is different. A wrong formatting helper is a bug. A wrong constant-time claim is a security incident.

> Trace: D229-D232
> Covers: Correctness evidence scales with domain risk.

## Result

[Rikona Kurasaki / Mjoyufull]
The stdlib philosophy is this: make ordinary systems work possible without making ordinary systems behavior vague. If an API allocates, blocks, fails, uses authority, changes platform behavior, or crosses FFI, the contract says so.

> Trace: D85, D152, D229
> Covers: Kyokai's stdlib exposes semantic behavior instead of hiding it behind convenience.
