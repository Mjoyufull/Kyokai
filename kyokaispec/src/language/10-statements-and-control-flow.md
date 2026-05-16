# Statements And Control Flow

[Rikona Kurasaki / Mjoyufull]
Statements are where control moves through the room. A value can be bound, a branch can split, a loop can turn back on itself, a cleanup can be promised for later, or the whole process can be told to stop. Kyokai keeps each of those motions named in source.

> Trace: D2, D8, D9, D16, D43, D71
> Covers: Kyokai is statement-oriented, uses explicit terminators and semicolons, and keeps control-flow exits visible.

A block is a sequence of statements. Empty executable blocks are legal and execute as no-ops. Newlines do not change statement meaning except where a token would otherwise be split illegally.

> Trace: D16, D180
> Covers: Blocks may be empty, semicolons terminate statements, and whitespace/newlines are not hidden control flow.

## Local Bindings

`let Pattern: Type := expr;` evaluates `expr` once, checks it against the annotated type, and binds the irrefutable pattern. `let Pattern := expr;` is legal only when the initializer already has exactly one fully determined denotable type before considering the omitted annotation.

> Trace: D46, D71, D87
> Covers: Immutable local inference is allowed only when the right-hand side already fixes the type; it is not target-typed guessing.

Plain `let` requires an irrefutable pattern. Refutable binding must use `let...else`, `case`, `while let`, or another explicit control-flow form.

> Trace: D15a, D38/D205/D206
> Covers: Plain binding cannot fail at runtime; fallible patterns require explicit failure syntax.

`var name: Type := expr;` declares a mutable local variable. Mutable locals always require an explicit type annotation. `var name := expr;` is illegal.

> Trace: D46, D59, D62
> Covers: Mutable local bindings are explicit and annotated; local inference is `let`-only.

No binding may shadow a still-live binding, parameter, pattern binding, declaration, or imported unqualified name in the same lookup reach. This includes names introduced by success and failure patterns.

> Trace: D60
> Covers: Binding and pattern introduction obey the no-shadowing rule.

## Assignment And Places

Assignment is a statement. `place := expr;` evaluates the assignee-place subexpressions left to right, then the right-hand side, then stores the value. Assignment yields no value and cannot be chained.

> Trace: D59, D71
> Covers: Assignment has specified sequencing and cannot appear where an expression is required.

A place may be a mutable local, a mutable field projection, a mutable index projection, or another assignable form admitted by the later place/borrow chapter. A place must be uniquely writable at the assignment point; the borrow checker rejects writes through aliased or non-mutable access.

> Trace: D14, D36/D132, D187, D238-D240
> Covers: Assignment targets must be valid mutable places and remain subject to borrow checking.

A mutable index assignment `a[i] := value;` writes through the selected `IndexableMut` protocol result and uses the same TPOE indexing contract as read indexing.

> Trace: D36/D132, D84
> Covers: Indexed assignment is protocol-based mutable indexing with checked indexing failure behavior.

## If And Case

An `if` statement evaluates its condition as `Bool`. If the condition is `true`, the `then` block runs. Otherwise the next `else if` condition is evaluated in source order, or the `else` block runs if present. If no branch is selected and no `else` exists, the statement completes normally.

> Trace: D9, D56, D71
> Covers: `if` control flow is source ordered, Boolean typed, and closed by `fi;`.

A `case` statement evaluates its scrutinee once, tests arms in source order, and executes the first matching arm. The `case` must be exhaustive. A non-exhaustive `case` is a compile-time error, not a runtime match failure.

> Trace: D13, D38/D205/D206, D71, D73
> Covers: `case` is source-ordered exhaustive structural matching with no runtime fallthrough failure.

Pattern guards do not exist. Boolean filtering after a structural match belongs inside the selected arm with an ordinary `if`.

> Trace: D38
> Covers: `case` destructures and `if` filters; guard clauses are rejected.

Each branch or arm has its own linear obligations. A branch that completes normally must leave all pre-existing live linear bindings in a state compatible with every other normally completing branch. A branch that exits through `return`, `break`, `continue`, `panic`, `todo`, or `unreachable` is checked on that exit path instead of at the join.

> Trace: D2, D58/D191, D98, D195, D205, D246
> Covers: Control-flow joins reconcile linear state only across paths that actually complete normally.

## Loops

A `while condition do ... od;` statement evaluates the condition at the start of each iteration. If it is `true`, the body runs. If it is `false`, the loop completes normally.

> Trace: D43, D71
> Covers: `while` is a statement, not an expression, and its condition is evaluated once per attempted iteration.

`while let Pattern := expr do ... od;` evaluates `expr` at the start of each iteration. If the refutable pattern matches, the body runs with the pattern bindings scoped to that iteration. If it does not match, the loop completes normally.

> Trace: D39, D71
> Covers: `while let` is refutable-pattern loop sugar that re-evaluates the expression per iteration and breaks on mismatch.

`for i from start to finish do ... od;` is an inclusive `Index` range loop. `for i from start below finish do ... od;` is a half-open `Index` range loop. The loop variable is an immutable `Index` binding scoped to the body.

> Trace: D32/D249, D210
> Covers: Range loops use explicit inclusive `to` and exclusive `below` forms over the `Index` domain.

Range endpoints are evaluated before the loop begins in source order. Literal endpoints may infer to `Index` when the range form supplies the expected type. Non-`Index` endpoint values require explicit conversion.

> Trace: D12, D32/D249, D71, D210
> Covers: Range loops do not accept ambient integer-like endpoints; `Index` is explicit except for ordinary literal inference.

`for Pattern in expr do ... od;` uses the language-defined iterator protocol. The iterable expression is evaluated once to produce the iterator state required by the protocol. Pattern binding over each yielded item follows the pattern rules, and loop desugaring consumes linear iterator state on every exit path.

> Trace: D32/D249, D98, D195
> Covers: `for-in` is protocol-based iteration with explicit pattern binding and linear iterator cleanup obligations.

Loops are statements and do not produce values. `break` carries no operand. Search-like value production uses an explicit outer binding, `Optional`, `Result`, or ordinary iterator helper APIs.

> Trace: D43, D128
> Covers: Kyokai has no loop expressions and no break-with-value.

## Break, Continue, And Labels

Unlabeled `break;` exits the innermost enclosing loop. Unlabeled `continue;` starts the next iteration of the innermost enclosing loop.

> Trace: D43, D122
> Covers: Plain loop control targets the innermost loop and carries no value.

A loop may be prefixed by a lexical label. `break label;` and `continue label;` target the named enclosing loop. The target must be an enclosing loop label in the current function body. Two loop labels whose scopes overlap may not reuse the same label name.

> Trace: D122
> Covers: Labeled break/continue targets are explicit lexical loop names and cannot target non-enclosing or non-loop constructs.

`break`, `continue`, and their labeled forms have type `Never` for expression-joining purposes inside control-flow constructs that admit diverging paths. They do not return values from loops.

> Trace: D43, D58/D191
> Covers: Loop exits are statically diverging at the statement site but do not make loops expressions.

## Return And Function Completion

`return expr;` evaluates `expr`, checks it against the function return type, runs eligible deferred actions for the exited scopes, and exits the function. `return;` is legal only where the function return type is `Unit`, and is equivalent to returning `nil` explicitly.

> Trace: D2a, D2b, D8, D71
> Covers: Return exits the function through structured cleanup and has explicit value rules.

A function declared `: Unit` may complete by reaching the end of its body. The compiler inserts the single possible `Unit` result at that end point. This does not apply to non-`Unit` functions and does not create Rust-style last-expression return.

> Trace: D8, D87, D238-D240
> Covers: Implicit completion exists only for end-of-body `Unit` fallthrough, because `Unit` has exactly one value.

A non-`Unit` function must return explicitly or otherwise never complete normally on every path. Falling off the end of a non-`Unit` function is a compile-time error.

> Trace: D8, D58/D191
> Covers: Non-`Unit` return values are never implicit.

Returning from a function must satisfy all live linear, borrow, defer, errdefer, and capability obligations for that path before control leaves the function.

> Trace: D2, D2a, D2b, D195, D246
> Covers: Function exits remain ownership-checked and cleanup-aware.

## Let-Else And Or Suffixes

`let P := E else Q do B fi;` evaluates `E` exactly once and pattern-tests the result. The form is legal only when `E` has a two-variant union type, `P` is refutable, `P` and `Q` are exhaustive over the two variants, and the else block `B` has type `Never`.

> Trace: D15a, D58/D191, D71
> Covers: `let...else` is a diverging fallible binding form over two-variant unions with one scrutinee evaluation.

Bindings introduced by the success pattern enter scope after `fi;`. Bindings introduced by the else pattern exist only inside the else block. Linear payloads bound in either pattern must be consumed on their path.

> Trace: D15a, D60, D98, D195
> Covers: Let-else has explicit success/failure scopes and path-local linear obligations.

`or return`, `or return name => expr`, `or break`, and `or continue` are statement suffixes for `let` binding statements over `Result[T, E]`. They are not expression operators and cannot appear inside subexpressions.

> Trace: D15, D15a, D119
> Covers: The `or ...` family is statement-only result propagation sugar.

Plain `or return` is legal only inside a function whose return type is `Result[U, E]` with the same error type as the scrutinee. `or return name => expr` permits explicit error mapping by binding the inner error payload and returning `Err(expr)` in the surrounding function's error type.

> Trace: D15a, D119
> Covers: Error propagation does not perform implicit error conversion; type changes require the explicit mapping expression.

`or break` and `or continue` are legal only inside loop scopes and only when the `Err` payload type is `Free`. If the error payload is linear, the programmer must use `let...else` and consume the payload explicitly before leaving the path.

> Trace: D15a, D195, D206
> Covers: Loop-control result sugar cannot discard linear error payloads.

For `errdefer`, `or return` counts as a structured error exit because it desugars to `return Err(...)`. `or break` and `or continue` are ordinary loop-control exits and do not trigger `errdefer`.

> Trace: D2b, D15a, D246
> Covers: Result propagation has explicit cleanup-path classification.

## Defer And Errdefer

`defer statement;` registers a cleanup action immediately in the current lexical scope. If the deferred action consumes a linear value, that value enters the `Deferred` checker state at the `defer` statement. A deferred value may be borrowed under ordinary rules but may not be moved, consumed again, reassigned, returned, sent, or registered in another consuming cleanup action.

> Trace: D2, D2a, D246
> Covers: `defer` is explicit cleanup registration with a visible checker state, not hidden scope-exit destruction.

`errdefer statement;` registers a cleanup action for structured error exits from the current lexical scope. A value consumed by that action enters the `ErrDeferred` checker state. On success paths, the program must explicitly move, return, destroy, or otherwise discharge that value before scope exit.

> Trace: D2, D2b, D246
> Covers: `errdefer` handles structured error exits only and leaves success-path ownership explicit.

Each lexical scope has its own deferred-action stack. Exiting a scope walks that scope's eligible deferred actions in reverse registration order. Inner-scope deferred actions run before outer-scope deferred actions.

> Trace: D2a
> Covers: Deferred actions are per-scope LIFO stacks filtered by the current exit path.

Ordinary `defer` runs on normal fallthrough, `return`, `break`, `continue`, `or return`, `or break`, `or continue`, and `panic`. `errdefer` runs only on structured error exits: currently `return Err(value)` and `or return`. TPOE runs no user deferred actions.

> Trace: D2b, D15a, D84
> Covers: Cleanup triggering is explicit by exit category; TPOE is immediate hard termination without user cleanup.

A deferred body may use local control flow inside itself, but it may not perform nonlocal `return`, `or return`, `break`, `or break`, `continue`, or `or continue` that exits the surrounding non-deferred scope. `panic` remains legal inside a deferred body.

> Trace: D207
> Covers: Cleanup code cannot replace or redirect the control-flow path that caused cleanup to run.

## Borrow Scopes

A `borrow name := &place do ... drop;` scope binds an immutable borrow for the body. A `borrow name := &!place do ... drop;` scope binds a mutable borrow. `&~place` is the explicit reborrow surface. The borrow scope ends at `drop;`.

> Trace: D7b, D14, D111, D187
> Covers: Borrow statements make borrow lifetime visible and close with `drop;`.

The borrow chapter defines the full checker states. This chapter fixes control-flow interaction: no control-flow path may leave a borrow scope while pretending the borrow still exists outside that scope unless the borrow type and region rules explicitly permit that escape.

> Trace: D6, D14, D72, D187, D238-D240
> Covers: Borrow scopes are control-flow boundaries and escaping borrows are rejected by the region/borrow rules.

## Panic, Todo, And Unreachable

`panic(message);` is explicit programmer-requested abnormal termination. It is not catchable inside Kyokai. It runs ordinary `defer` cleanup for exited scopes before process termination, but it does not run `errdefer` unless a more specific structured error rule says so.

> Trace: D2b, D84
> Covers: `panic` is explicit hard process termination with ordinary cleanup, not an exception or recoverable result.

`todo;` and `todo "message";` are built-in diverging developer-stub statements. Executing `todo` enters the `panic` category with source-location diagnostic information. The compiler should warn on `todo` in non-test code, but runtime behavior remains explicit panic.

> Trace: D121
> Covers: `todo` is a deliberate panic-category stub and has type `Never`.

`unreachable;` is a built-in diverging statement for paths the program claims cannot execute. If execution reaches it, the result is TPOE. It is not optimizer-only poison and not undefined behavior.

> Trace: D73, D84, D121
> Covers: `unreachable` is a checked contract-failure path, not backend UB.

`return`, `break`, `continue`, `panic`, `todo`, and `unreachable` are statically diverging at the statement site and have type `Never` where the type system needs to join branches. A possible TPOE inside an ordinary expression does not make that expression statically `Never`.

> Trace: D58/D191, D121
> Covers: Only statically known non-completion has `Never`; possible runtime traps keep the ordinary expression type.

## Debug Statement

`debug expr;` is a language-level development instrumentation statement. It is not a function call, not a module import, and not UFCS. It formats through `Displayable` to stderr in debug/test builds and is stripped in release builds.

> Trace: D40, D45, D233
> Covers: `debug` is built-in profile-sensitive instrumentation, not ordinary capability-gated output.

`debug` does not require a terminal capability, does not appear in module interfaces, and is not part of the program's ordinary observable behavior contract. Production console I/O uses explicit capability-gated APIs.

> Trace: D45, D211, D233
> Covers: Debug output is outside the capability model by design, while production I/O remains capability-gated.

The operand of `debug` is a debug-observation expression. It may observe existing values through local names, constants, literals, immutable field projection, immutable index/slice projection, and parenthesized composition. It may not contain ordinary function calls, UFCS calls, allocating constructors, arithmetic or comparisons that may TPOE, assignment, `comptime`, `panic`, `return`, `break`, `continue`, `or ...`, `defer`, or capability-using operations.

> Trace: D45, D233
> Covers: Debug stripping cannot change ownership, side effects, control flow, or possible contract-failure behavior.

`debug x;` observes `x` by immutable borrow and does not consume linear values. If a programmer wants to debug a computed value, that computation must be bound in ordinary code first, and the binding may then be observed.

> Trace: D14, D195, D233
> Covers: Debug observation cannot be the only consumer of a linear value and cannot hide computation behind profile-dependent instrumentation.

## Statement Forms Not In Kyokai

Kyokai has no `skip` keyword, no expression-level assignment, no pre/post increment, no exceptions, no `try/catch`, no in-process `catch panic`, no in-process `catch TPOE`, no loop expressions, no break-with-value, no statement-local imports, and no hidden destructor call at block exit.

> Trace: D43, D59, D84, D147, D253
> Covers: Rejected statement/control mechanisms remain absent rather than unspecified.
