# Admission Contracts

[Rikona Kurasaki / Mjoyufull]
A Kyokai standard-library API is not admitted because it is useful. Useful is only the beginning. It is admitted when the contract is written, the edge cases are named, the tests exist, and the implementation boundary is honest.

> Trace: D85, D229
> Covers: Stdlib admission requires explicit contracts, edge-case behavior, tests, and implementation policy.

## Admission States

| State | Meaning | Required Evidence | Trace |
| --- | --- | --- | --- |
| `internal` | Toolchain or stdlib implementation detail. | No public import path; no stability promise. | D152, D229 |
| `experimental` | Public preview API. | Contract fields present; tests sufficient for experimentation; breaking changes allowed under stated policy. | D157, D223, D229 |
| `stable` | Public API covered by ordinary compatibility policy. | Full contract fields, edge cases, tests/oracles, docs, audit status, SemVer classification. | D85, D223, D229, D243 |
| `compatibility` | Legacy or migration API. | Contract fields, warning/deprecation policy, reason for retaining legacy behavior. | D223, D229, D243 |
| `transitional` | Temporary bootstrap/FFI bridge. | Unsafe/FFI contract, safe wrapper, replacement criteria, owner, removal condition. | D230 |

> Trace: D85, D157, D223, D229-D230, D243
> Covers: Admission status encodes stability and evidence requirements.

A public API cannot be marked stable until all common contract fields are present in its `.kyo` documentation and any mechanically checkable fields are available to `kyokai doc` and `kyokai audit`.

> Trace: D85, D150, D218, D229
> Covers: Stable APIs must have machine-extractable contract fields.

## Admission Record

Each admitted module has an admission record. The record names the module, status, owner, public modules exported, implementation policy, unsafe/FFI status, capability surfaces, allocation policy, failure policy, platform support, conformance tests, oracle/reference source where applicable, fuzz/property tests where applicable, and compatibility boundary.

> Trace: D85, D150, D220, D229-D232
> Covers: Module admission records carry the evidence needed for docs, audit, and tests.

An admission record for a pure algorithm must name why the implementation is safe native Kyokai or why a non-native implementation is temporarily or permanently justified. Convenience, performance, or existing C availability is not enough to make pure computation FFI-backed forever.

> Trace: D229-D230
> Covers: Pure algorithms default to safe native Kyokai and exceptions require evidence.

## Edge Cases

Every API family must list its edge cases before stable admission. Required edge-case families include empty input, maximum sizes, zero sizes, invalid encodings, invalid paths, closed handles, exhausted iterators, allocation failure, integer overflow, floating NaN/infinity/signed zero where relevant, target unsupportedness, cancellation/deadline behavior, and capability denial.

> Trace: D74, D75-D76, D77, D80, D85, D229-D232
> Covers: Stable APIs list edge cases instead of inheriting folklore behavior.

If an edge case is impossible by type, the contract says that. If it is rejected at compile time, the contract says that. If it is a runtime `Result`, `Optional`, TPOE, runtime-fatal, or target-unsupported diagnostic, the contract names that category.

> Trace: D53, D74, D84-D85, D229
> Covers: Edge cases map to explicit Kyokai failure categories.

## Tests And Oracles

Stable APIs require executable tests. The required test level scales with risk. Simple pure helpers require unit and boundary tests. Collections require ownership, invalidation, allocation-failure, and property tests. Math requires accuracy vectors and edge-case tests. Crypto requires modern external vectors and side-channel/review evidence. OS APIs require target-specific positive and negative tests or documented unsupported-target behavior.

> Trace: D220, D229-D232
> Covers: Test burden scales with domain risk and API behavior.

A test cannot be the only specification of behavior. Tests check the contract; they do not replace it. When a test reveals a missing rule, the rule is written into the contract or opened as a public D-point before stable admission.

> Trace: D85, D155, D229
> Covers: Tests validate written behavior and cannot become hidden specs.

## Failure And Fatality

Recoverable environmental failures use typed error results. Allocation failure is `AllocError` by default. I/O and OS failures use domain error types. Contract violations use TPOE. Runtime-fatal failures are reserved for the runtime-fatal category and must not be hidden behind ordinary helpers unless the helper name announces the fatal contract.

> Trace: D53, D74, D84, D85, D229
> Covers: Stdlib APIs classify failure through Kyokai's explicit failure taxonomy.

Fatal convenience functions are allowed only with names that say the contract out loud, such as `mustReserve`, `mustPush`, or an equivalent `must*` form. They are not the default API surface.

> Trace: D74, D85, D229
> Covers: Fatal-on-failure helpers must be visibly named and non-default.

## Compatibility

Stable stdlib APIs evolve by SemVer-style package compatibility unless the change is a source-semantics change requiring an edition. Removing a stable API, changing ownership behavior, changing allocation behavior, changing failure categories, changing capability requirements, changing invalidation behavior, or changing platform support is compatibility-relevant.

> Trace: D105, D157, D223, D243
> Covers: Stdlib compatibility tracks semantic API fields, not just names and signatures.

Deprecation must state replacement API, behavior difference, minimum removal horizon if removal is planned, and whether the change is ordinary SemVer, compatibility-module-only, or edition-gated.

> Trace: D157, D223, D243
> Covers: Deprecations explain compatibility and migration.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
RIIK is powerful only if the rewrite is disciplined. A naive rewrite is just a fresh place to hide old mistakes. Admission records force each module to walk into the room carrying its tests, contracts, edge cases, and trust boundary where everyone can see them.

> Trace: D229-D230
> Covers: Admission criteria keep native Kyokai implementations trustworthy instead of merely ideological.
