# Transitional FFI Tracking

[Rikona Kurasaki / Mjoyufull]
Transitional code is not shameful. Untracked transitional code is. Kyokai can borrow a bridge while the road is being built, but the map has to mark that bridge and say whether it stays.

> Trace: D20, D64, D229-D230, D245
> Covers: Transitional FFI is allowed only with unsafe contracts, safe wrappers, and replacement tracking.

## Allowed Reasons

FFI-backed stdlib implementation is allowed for OS and hardware boundaries, mature externally reviewed libraries, bootstrap dependencies needed before Kyokai can self-host, and temporary bridges recorded in the implementation plan. Pure computation does not remain FFI-backed merely because a C implementation already exists.

> Trace: D64, D229-D230
> Covers: FFI is a boundary or bridge, not the default home of pure algorithms.

A permanent FFI boundary must say why the boundary is permanent. A transitional FFI boundary must say what native Kyokai or safer boundary will replace it, what evidence is needed for replacement, and what condition removes or downgrades the bridge.

> Trace: D229-D230, D243
> Covers: Permanent and transitional FFI have different documented obligations.

## Required Record

Each transitional FFI item has a tracking record containing module path, public safe API, unsafe module path, foreign library or ABI, reason for FFI, permanent-or-transitional status, replacement target, owner, admission status, license/provenance notes, supported targets, test/oracle source, audit status, and removal or stabilization criteria.

> Trace: D20, D85, D229-D230, D263
> Covers: FFI tracking records carry provenance, ownership, testing, and replacement information.

| Record Field | Required Meaning | Trace |
| --- | --- | --- |
| Public API | Safe Kyokai module/function/type exposed to users. | D20, D245 |
| Unsafe Boundary | `pragma Unsafe_Module`, foreign declarations, callbacks, dynamic loading, or raw pointer surface. | D20, D242-D242a, D245 |
| Reason | OS/hardware boundary, reviewed external dependency, bootstrap bridge, or separate justified exception. | D64, D230-D231 |
| Replacement Target | Native Kyokai implementation, safer wrapper, permanent external boundary, or explicit compatibility retirement. | D229-D230 |
| Contracts | Ownership, allocation, failure, callback, thread, lifetime, panic/TPOE, and capability behavior. | D20, D85, D245 |
| Tests | Conformance vectors, oracle comparison, ABI tests, fuzz/property tests where applicable. | D220, D229-D232 |
| License/Provenance | Source project, license compatibility, generated/bundled status, and attribution needs. | D263 |
| Compatibility | Stable, experimental, transitional, compatibility-only, deprecation/removal rule. | D223, D243 |

> Trace: D20, D64, D85, D220, D223, D229-D232, D242-D245, D263
> Covers: Transitional FFI records are auditable across safety, tests, license, and compatibility boundaries.

## Unsafe Module And Safe Wrapper Rule

Every FFI wrapper used by safe stdlib code lives behind an unsafe module boundary. Safe code calls a safe Kyokai wrapper whose contract fully states ownership, lifetime, allocation, failure, callback, concurrency, and capability behavior. The wrapper cannot export raw foreign behavior as safe folklore.

> Trace: D20, D242-D242a, D245
> Covers: Safe stdlib APIs over FFI require documented unsafe boundaries and safe wrappers.

FFI must not pass raw by-value linear values or Kyokai sum types across the C boundary unless a specific unsafe ABI contract admits the representation and ownership transfer. Callback APIs must state whether foreign code can call back after the initiating call returns, on which task/thread, and with what authority.

> Trace: D20, D42, D77, D113a-D113b, D245
> Covers: Linear ownership, sum representation, callbacks, and authority do not cross FFI silently.

## Replacement And Admission

A transitional FFI API may be public only as `experimental`, `transitional`, or through a stable safe wrapper whose internal FFI boundary is accepted as either permanent or tracked for replacement. The public contract remains stable only if replacing the implementation with native Kyokai preserves the same behavior.

> Trace: D157, D223, D229-D230, D243
> Covers: Implementation replacement cannot change public stdlib behavior without compatibility process.

When a native Kyokai replacement is proposed, it must satisfy the same admission record as any other stdlib API: semantic contract, edge cases, tests/oracles, allocation/failure behavior, portability notes, and compatibility impact. RIIK is not an exemption from evidence.

> Trace: D64, D220, D229-D232
> Covers: Native replacements are admitted by evidence, not by ideology.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
The point is to be practical without letting practical become permanent by accident. Kyokai can touch C, the OS, hardware, and mature libraries, but every touch leaves a record: why it exists, what it promises, and whether it is meant to disappear.

> Trace: D20, D64, D229-D230, D245
> Covers: Transitional FFI stays useful, visible, and replaceable.
