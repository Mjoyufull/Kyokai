# Math And Numerics

[Rikona Kurasaki / Mjoyufull]
Numerics are where confidence can become arrogance fast. A wrong byte order, a silent overflow, a host `libm` result changing under your feet, a signed zero nobody named: all of that becomes program behavior. Kyokai makes the number story testable.

> Trace: D37, D41, D75-D76, D117, D232, D260
> Covers: Numeric stdlib APIs specify conversion, bit operations, overflow, endianness, floating behavior, and math accuracy contracts.

## Integer Helpers

Integer helper APIs follow the language rules for checked arithmetic, same-type operands, explicit conversion names, and distinct `Index`. A helper that wraps, saturates, overflows into a pair, or traps must say so in its name and contract.

> Trace: D37, D75-D76, D210
> Covers: Integer helper APIs do not weaken checked arithmetic or conversion rules.

Bitwise, shift, and rotate helpers follow the keyword-operator contracts. Shift and rotate counts are checked according to the language rule; any helper that masks counts instead must be separately named as such.

> Trace: D41, D75
> Covers: Bit operations state count behavior explicitly.

Fixed-width integer endian transforms are explicit methods and byte-array encode/decode helpers. Kyokai does not automatically byte-swap at FFI, packed-record, file, network, or container boundaries.

> Trace: D117, D260
> Covers: Endianness is explicit at every boundary.

| API Family | Ownership | Allocation | Failure | Capabilities | Platform | Tests | Trace |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Integer conversion | Consumes or borrows scalar value. | None. | Range failure returns `Result` or named TPOE helper. | None. | Fixed-width behavior identical across targets. | min/max, negative, `Index`, narrowing, widening. | D37, D75-D76, D210 |
| Bit/shift/rotate | Value operands only. | None. | Invalid count is TPOE unless checked helper returns `Result`. | None. | Width fixed by type. | zero count, width count, overflow-like edges. | D41, D75 |
| Endian encode/decode | Borrows output/input byte array or returns value. | None unless fresh byte buffer helper takes allocator. | Bad length returns `Result` or compile-time rejection for fixed arrays. | None. | Byte order explicit and target-independent. | big/little/native helpers, round trips, protocol vectors. | D117, D260 |

> Trace: D37, D41, D75-D76, D117, D210, D260
> Covers: Integer API families publish ownership, allocation, failure, platform, and tests.

## Floating-Point And Accuracy Contracts

Floating operations follow Kyokai's strict IEEE 754 contract where the language specifies scalar floating evaluation. Math-library functions must publish an accuracy contract before stable admission. Accepted contract forms include `Exact`, `CorrectlyRounded`, `MaxUlp(n)`, `AbsError(bound)`, and `RelError(bound)`.

> Trace: D75-D76, D232
> Covers: Floating math APIs carry explicit tested accuracy contracts.

Accuracy tiers are API documentation and conformance obligations. They are not call-site type parameters, not ordinary overload selectors, and not hidden target options. A function's contract says what accuracy it promises for each supported type and target.

> Trace: D232
> Covers: Numerical accuracy stays visible without adding call-site verbosity.

Math functions must state behavior for NaN, infinities, signed zero, subnormal values, overflow, underflow, domain errors, range errors, and rounding-sensitive boundaries. If an operation exposes errno-like or floating-status behavior for compatibility, that behavior is in a compatibility module and is not the preferred pure Kyokai surface.

> Trace: D75-D76, D85, D232
> Covers: Floating edge cases are contract text, not inherited host folklore.

## Implementation And Oracles

Pure numeric computation is written in safe native Kyokai by default. FFI to host `libm` is not the stable default for ordinary `Kyokai.Math` because host libraries vary by target, version, flags, and platform policy. Transitional FFI may exist while native implementations are admitted, but it is tracked under the FFI replacement policy.

> Trace: D64, D229-D230, D232
> Covers: `Kyokai.Math` follows RIIK while allowing tracked transitional bridges.

Numerical admission tests use appropriate independent oracles: official vectors where available, high-precision reference computation, cross-library comparison, exhaustive tests for small domains, property tests for algebraic identities where they are mathematically sound under floating rules, and regression vectors for edge cases.

> Trace: D220, D229, D232
> Covers: Math correctness evidence scales by domain and uses independent references.

| Math Family | Ownership | Allocation | Failure | Determinism | Accuracy | Tests | Trace |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Pure scalar math | Value inputs and outputs. | None. | Domain/range behavior specified per function; no ambient errno. | Deterministic for supported target contract. | One of the admitted accuracy forms. | special values, random vectors, oracle comparison. | D75-D76, D232 |
| Parsing/formatting numerics | Borrows text/bytes; output owned only by allocator-taking helpers. | Allocator explicit for owned text. | `ParseError`, `AllocError`, sink error. | Locale-independent unless locale parameter exists. | Round-trip guarantee stated per formatter. | accepted syntax, overflow, round trip, invalid input. | D40-D40a, D69, D232 |
| Vector/math intrinsics | Value/vector inputs. | None. | Target unsupported diagnostic or explicit fallback contract. | No silent scalarization unless contract says so. | Contract per lane/function. | target gating, lane semantics, oracle vectors. | D104, D232 |

> Trace: D40-D40a, D69, D75-D76, D104, D220, D229, D232
> Covers: Math API families publish allocation, failure, determinism, accuracy, and tests.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
The point is not to sound brave about writing math in Kyokai. The point is to prove the promise in public. If `sin` says `MaxUlp(1)`, the tests must hunt that number. If a byte helper says big-endian, no boundary gets to swap behind your back.

> Trace: D117, D229-D232, D260
> Covers: Numeric APIs are trusted through contracts and tests, not borrowed confidence.
