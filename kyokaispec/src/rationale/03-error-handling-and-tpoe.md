# Error Handling And TPOE Rationale

[Rikona Kurasaki / Mjoyufull]
Austral's brutal lesson still belongs in Kyokai: not every bad thing is the same kind of bad thing. A missing file, an exhausted allocator, a violated precondition, a panic, and stack corruption should not wear the same mask just because a runtime could route them through one mechanism.

> Trace: D24, D53, D74, D84, D253, D259, D262
> Covers: Kyokai separates recoverable failure values, allocation failure, contract violation, panic, TPOE, runtime-fatal, and diagnostic error protocols.

## Failure As Data

[Rikona Kurasaki / Mjoyufull]
Recoverable conditions are values. `Optional[T]` says absence. `Result[T, E]` says an operation could fail in a way the caller can choose to handle. `StandardError[E]` says an error type can report structured diagnostic information without becoming a hidden exception object system.

> Trace: D24, D119, D259
> Covers: Recoverable failure uses typed values and explicit error mapping.

This is where Kyokai keeps the useful part of error-code culture while refusing the weak part. A typed result is not a brittle integer returned through folklore. Linearity and pattern rules can force the caller to account for a value that ordinary C would let them ignore.

> Trace: D38/D205/D206, D119
> Covers: Pattern and linearity rules prevent silent discard of meaningful result payloads.

## Contract Violations

[Rikona Kurasaki / Mjoyufull]
A contract violation is not a failed business condition. It is evidence that the program has crossed a line it promised not to cross. Kyokai therefore treats preconditions, postconditions, checked arithmetic traps, bounds violations, and `unreachable` as specified failure paths, not ordinary recovery edges.

> Trace: D53/D140/D142, D58/D191, D73, D75-D76, D84/D253
> Covers: Contract and checked-operation failures have specified TPOE/runtime-failure behavior rather than ordinary catchable exceptions.

TPOE is harsh because a violated contract destroys trust in the surrounding state. Running destructors, unwinding frames, or resuming from some handler would make the failure look recoverable. Kyokai chooses the simpler security posture: terminate through the specified path and do not pretend the broken invariant is still a program state worth nursing.

> Trace: D2a/D2b/D207, D84/D253
> Covers: TPOE is distinct from panic and ordinary cleanup; it is not caught inside the program.

## Panic And Runtime-Fatal

[Rikona Kurasaki / Mjoyufull]
`panic` is still not a recoverable exception, but it is not the same path as TPOE. Panic runs ordinary eligible `defer` cleanup. TPOE does not. Runtime-fatal events, including stack overflow, are their own category where the runtime must terminate before safe invariants can be corrupted.

> Trace: D2a/D2b/D207, D84/D253, D262
> Covers: Panic, TPOE, and runtime-fatal termination have separate cleanup and termination contracts.

This separation matters because it lets tests, diagnostics, and runtime design speak honestly. A user-triggered panic, a violated language contract, and a guard against stack corruption are not the same engineering event.

> Trace: D29, D84/D253, D262
> Covers: Tooling and runtime reports preserve distinct failure categories.

## Allocation Failure

[Rikona Kurasaki / Mjoyufull]
Borretti's Austral allocation argument remains important: allocation failure is pervasive and cannot be waved away by Linux overcommit folklore. Kyokai makes allocation failure recoverable by default through `AllocError`, while allowing fatal convenience helpers only when the name says so.

> Trace: D44, D74, D201
> Covers: Allocation failure is ordinary data by default and allocator-taking APIs expose allocation behavior.

The rule changes the shape of standard-library APIs. `format` returns `Result[String, AllocError]`. Container growth can fail. `mustPush` can exist, but the fatal behavior is in the name instead of hiding behind a pleasant helper.

> Trace: D40-D40a, D74, D102, D201
> Covers: Formatting and container APIs make allocation and fatal-on-failure variants explicit.

## Why Not Exceptions

[Rikona Kurasaki / Mjoyufull]
Exceptions make too many paths look like one path. They cross function boundaries without appearing in the ordinary call shape, interact badly with exactly-once resource reasoning, and turn cleanup into a hidden runtime performance. Kyokai would rather make the failure branch visible and make the hard bug terminate.

> Trace: D2, D15/D15a, D84, D119, D253
> Covers: Kyokai rejects exceptions and stack unwinding in favor of visible result flow, visible cleanup, panic, TPOE, and runtime-fatal categories.

## Result

[Rikona Kurasaki / Mjoyufull]
The error model is strict because it is trying to keep the ground from moving. User-facing failure stays in values. Bugs terminate through named paths. Allocation is handled unless you visibly ask for fatal behavior. Cleanup happens where the source says cleanup can happen.

> Trace: D24, D53, D74, D84, D119, D253, D259
> Covers: Kyokai's error model preserves explicit control flow and exact failure categories.
