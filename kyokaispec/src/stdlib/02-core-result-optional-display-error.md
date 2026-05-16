# Core Result, Optional, Display, And Error Protocols

[Rikona Kurasaki / Mjoyufull]
The core library shapes are the language's daily grammar for absence, failure, rendering, and diagnostic identity. They have to be small enough to remember and explicit enough that they do not grow into hidden control flow.

> Trace: D24, D40, D40a, D102, D259
> Covers: Core `Optional`, `Result`, formatting, and standard error protocols are stdlib contract surfaces tied to language built-ins.

## Optional And Result

`Optional[T]` represents absence and presence. `Result[T, E]` represents success or recoverable failure. Both are language-known built-in type families with stdlib protocol support. Their pattern behavior is specified by the language chapters; this chapter specifies stdlib helper contracts.

> Trace: D24, D65, D205-D206
> Covers: `Optional` and `Result` are language-known shapes with stdlib helper APIs.

Helpers over `Optional[T]` and `Result[T, E]` must preserve linearity. A helper that receives or returns a `Linear` payload must consume, return, or bind that payload exactly as its contract states. No helper may silently discard a linear payload on an ignored branch.

> Trace: D24, D77, D205-D206
> Covers: Core combinators respect linear payload obligations.

`Result` transformation helpers must not perform implicit error conversion. Error mapping is explicit through named functions or callbacks, matching `or return err => expr` behavior in the language.

> Trace: D119, D259
> Covers: Error conversion remains explicit in both syntax and stdlib helpers.

| API Family | Ownership | Allocation | Failure | Capabilities | Tests | Trace |
| --- | --- | --- | --- | --- | --- | --- |
| `Optional` predicates | Borrow input; do not consume payload unless named `into*`. | None. | Cannot fail. | None. | Some/None and linear payload compile-fail cases. | D24, D205-D206 |
| `Result` predicates | Borrow input; do not consume payload unless named `into*`. | None. | Cannot fail. | None. | Ok/Err and linear payload compile-fail cases. | D24, D119 |
| `map`/`mapErr` helpers | Consume the selected payload and callback result exactly once. | Only if callback allocates; helper itself none. | Callback failure shape is explicit in signature. | Callback capability needs are ordinary parameters. | Branch, cleanup, linearity, callback failure. | D119, D205-D206 |
| `unwrap`-style fatal helpers | Consume container and return payload or terminate by named fatal contract. | None. | Failure branch is named fatal helper behavior. | None. | Success, fatal path, linear cleanup. | D74, D84 |

> Trace: D24, D74, D84, D119, D205-D206, D259
> Covers: Optional/Result helper families state ownership, allocation, failure, authority, and tests.

## Displayable And FormatSink

`Displayable[T]` is the standard protocol for rendering a value as human-readable text. Rendering is not raw stream transport. A value may be displayable without knowing anything about terminals, files, sockets, or allocation.

> Trace: D40, D102
> Covers: Display rendering is separated from I/O transport.

`FormatSink` is the sink protocol used by non-allocating formatting. A sink receives bytes or text chunks according to its contract and reports sink-specific failure. `writeFmt` writes through a `FormatSink`/`Writable` path and returns ordinary stream failure rather than allocating a `String`.

> Trace: D40, D102
> Covers: Non-allocating formatted output uses an explicit sink and typed stream failure.

`format(alloc, template, args...)` allocates a `String` using the explicit destination allocator and returns `Result[String, AllocError]`. The template language is comptime-checked and limited to `{}` placeholders unless a later decision adds width, radix, alignment, or localization. Formatting functions for those features should be named ordinary functions rather than hidden format-string mini-languages.

> Trace: D40, D40a, D74
> Covers: Allocating formatting is allocator-explicit, fallible, and uses a narrow checked placeholder language.

| API Family | Ownership | Allocation | Failure | Capabilities | Determinism | Trace |
| --- | --- | --- | --- | --- | --- | --- |
| `display(value, sink)` | Borrows value; mutably borrows sink. | None unless sink allocates by its own contract. | Sink error only. | Sink may require capability by construction, not by `Displayable`. | Deterministic over value and sink. | D40, D102 |
| `format(alloc, template, args...)` | Borrows or consumes args according to display instance; allocator mutably borrowed. | Allocates destination `String`. | `AllocError`; template errors are compile-time diagnostics. | None. | Deterministic over args and template. | D40a, D74 |
| `writeFmt(sink, template, args...)` | Mutably borrows sink; observes args. | None by formatting layer. | Stream/sink error; prefix writes may have occurred. | Capability only through sink value. | Prefix-preserving failure. | D102 |

> Trace: D40-D40a, D74, D85, D102
> Covers: Formatting APIs state ownership, allocation, failure, capability, and determinism.

## StandardError

`StandardError[E]` is an optional diagnostic protocol for error types. It is separate from `Displayable`; an error can be displayable without implementing structured diagnostics, and an API can return an error type without forcing a dynamic error-object system.

> Trace: D259
> Covers: `StandardError` is standalone and does not inherit from `Displayable`.

`StandardError` does not provide a mandatory `source()` chain. Error context is expressed by concrete wrapper types such as `ContextError[E]`, not by ambient dynamic stacks or hidden exception chains.

> Trace: D119, D259
> Covers: Error context remains concrete and explicit.

`StandardError` instances may expose category, stable code, message rendering through a sink, and optional structured fields. They must state allocation behavior and must not allocate unless the method name/signature exposes an allocator.

> Trace: D40, D85, D259
> Covers: Structured error diagnostics remain allocator-explicit and sink-oriented.

## Naming Contracts

Core helpers follow ownership-signaling names. `as*` borrows and returns a view, `to*In` borrows plus allocates into an explicit allocator, `into*` consumes without a fresh destination allocator, and `into*In` consumes while allocating into an explicit allocator.

> Trace: D11b, D201
> Covers: Core helper names signal ownership and allocation boundaries.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
`Result` should not become exceptions wearing a different coat. `Displayable` should not become I/O authority. `StandardError` should not become a dynamic object system. These shapes stay small because Kyokai wants the branch, the allocator, the sink, and the error mapping to be visible at the call site.

> Trace: D24, D40, D102, D119, D259
> Covers: Core stdlib protocols keep absence, failure, rendering, and diagnostics explicit.
