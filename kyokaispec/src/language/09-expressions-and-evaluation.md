# Expressions And Evaluation

[Rikona Kurasaki / Mjoyufull]
An expression is where a program stops promising and starts producing a value. Kyokai makes that motion strict. The reader should be able to walk the source from left to right and know which call runs, which borrow starts, which bounds check fires, and which value is built before the next one moves.

> Trace: D71, D73, D87, D139, D228
> Covers: Kyokai expression evaluation is strict, source ordered, and specified by the language rather than by backend evaluation folklore.

This chapter defines ordinary expression evaluation. Control-flow statements, cleanup exits, and the elaboration pipeline are defined in the next language chapters. If this chapter says an expression may trigger TPOE, that failure is a language-defined contract violation, not undefined behavior.

> Trace: D73, D75, D84, D139, D228
> Covers: Runtime expression failures such as overflow or bounds violations have specified Kyokai termination behavior and cannot become backend undefined behavior.

## Evaluation Order

Kyokai evaluates subexpressions strictly in source order. Unary operators evaluate their operand before applying the operator. Binary operators evaluate the left operand, then the right operand, then apply the operator. Function calls evaluate the callee expression first, then arguments left to right, then perform the call.

> Trace: D23, D71
> Covers: Ordinary operators and calls evaluate their subexpressions exactly once in source order before the operation itself runs.

A UFCS call evaluates the receiver first, then the remaining arguments left to right, then performs the desugared call. Field access evaluates the base expression first. Indexing evaluates the container expression first, then the index expression, then performs the bounds/indexing operation.

> Trace: D7a, D34, D36/D132, D71, D106, D110/D254
> Covers: Receiver calls, field access, indexing, and slicing keep source-order evaluation after their surface sugar is resolved.

Record construction, record update, union construction, and array literals evaluate written field, override, payload, and element expressions in source order. Record field order does not change construction meaning, but it does not give the compiler permission to reorder the expressions the programmer wrote.

> Trace: D35, D55, D65, D71, D138
> Covers: Data construction evaluates supplied expressions in source order even when field identity is name-based.

`if`, `while`, `while let`, and `case` evaluate their condition or scrutinee before choosing a branch, arm, or loop body. Only the selected branch, selected arm, or active loop body is then evaluated. `let...else` evaluates its scrutinee exactly once before pattern testing.

> Trace: D13, D15a, D39, D71, D205
> Covers: Conditional and pattern-control forms evaluate the tested expression once before branch selection and never duplicate scrutinee evaluation.

Short-circuiting `and` and `or` are the only ordinary expression operators that conditionally skip an operand. `a and b` evaluates `a`; if `a` is `false`, `b` is not evaluated. `a or b` evaluates `a`; if `a` is `true`, `b` is not evaluated.

> Trace: D56, D57, D71
> Covers: Boolean `and` and `or` are source-ordered short-circuit operators and are the ordinary conditional-evaluation exception.

Assignment statements evaluate assignee-place subexpressions from left to right, then evaluate the right-hand side, then perform the store. The assignment itself is a statement and produces no value.

> Trace: D59, D71
> Covers: Assignment sequencing is specified, and assignment cannot be used as an expression or chained.

## Temporaries

A temporary created while evaluating a statement lives until that statement completes unless a more local rule explicitly says otherwise. The lifetime is statement-scoped, not block-extended by convenience and not shortened by backend lowering.

> Trace: D72, D139, D228
> Covers: Expression temporaries have statement lifetime and backend lowering must preserve that lifetime contract.

Immutable borrowing of an rvalue temporary is allowed only inside the same statement when the borrow is used immediately and cannot escape. A borrow of a temporary cannot be stored, returned, captured into a longer-lived value, registered for later cleanup, or otherwise given a lifetime beyond the statement that created it.

> Trace: D14, D72, D187, D238-D240
> Covers: Immutable rvalue temporary borrows are statement-local only and cannot escape through storage, return, capture, or deferred use.

Mutable borrowing of an rvalue temporary is illegal. If a mutating call needs a mutable borrow, the programmer must bind the value to a local first so the storage and lifetime are visible in source.

> Trace: D72, D213, D238-D240
> Covers: Mutable rvalue temporary borrowing is rejected, including through UFCS mutating calls.

UFCS does not bypass the temporary rules. If `expr.method(args)` desugars to a call whose first parameter requires `&![T]`, `expr` may not be an rvalue temporary. Consuming receivers and immutable-borrow receivers remain legal on rvalues only when the ordinary temporary-borrow rules are satisfied.

> Trace: D7a, D72, D213, D254
> Covers: Dot syntax is not a lifetime escape hatch; mutating UFCS on rvalue temporaries is rejected.

Static literals have their own language-defined storage duration. Borrowing a string literal or other static literal as static data is legal only through the explicit static-literal rules that give that value static storage.

> Trace: D54, D72, D120
> Covers: Static literal borrowing is admitted by explicit static storage rules, not by general temporary lifetime extension.

## Literal Expressions

`nil` has type `Unit`. `true` and `false` have type `Bool`. `None`, `Some(expr)`, `Ok(expr)`, and `Err(expr)` are built-in constructor expressions for the language-level `Optional` and `Result` families.

> Trace: D24, D58, D194
> Covers: Unit, Boolean, Optional, and Result literal/constructor forms are built-in language expressions, not prelude imports.

Integer and floating literals may be typed by a closed suffix or by contextual literal inference. Contextual literal inference applies only when the expected type is already known from a function parameter, explicitly annotated binding, operator operand, or another local expected-type site. Ambiguous literals remain compile-time errors.

> Trace: D12, D135/D261, D87
> Covers: Literal typing is a narrow implicit completion over known expected types and suffixes, not full expression inference or numeric conversion.

A suffixed literal has the suffixed type before contextual inference. Context may confirm that type but may not silently retarget it. If the literal value is not representable in the suffixed type, compilation fails.

> Trace: D135/D261, D75
> Covers: Numeric suffixes fix literal type and checked representability at compile time.

Ordinary string literals produce `String`. Raw multiline string literals produce `String` without escape processing. Code-point literals produce a `Nat32` Unicode scalar value. Byte literals produce `Nat8`. `static "..."` produces `StaticString`, and ordinary strings are not contextually retargeted into static strings.

> Trace: D30, D54, D120, D194
> Covers: Text, raw text, code-point, byte, and static text literals are distinct and have fixed built-in types.

## Names, Constants, And Paths

An identifier expression names the closest visible binding, parameter, pattern binding, local variable, or constant under the module and lexical lookup rules. Missing or ambiguous names are compile-time errors.

> Trace: D17, D60, D179, D214
> Covers: Expression name lookup follows lexical and import-visible name rules and rejects missing or ambiguous names.

A module-level constant expression forces the constant if its value is needed. Forcing uses the lazy cached constant-evaluation model: `Unforced`, `Evaluating`, and `Evaluated`. A dependency cycle is a compile-time error.

> Trace: D18, D18a, D215
> Covers: Constant expression use participates in lazy cached cycle-checked constant forcing.

Qualified paths may name only declarations visible to the current package. A path cannot reach private body-only declarations or another package's internal declarations.

> Trace: D17, D78/D179/D214
> Covers: Qualified expression paths respect public, internal, and private visibility boundaries.

## Calls And UFCS

A function call evaluates the callee and arguments, resolves generic arguments and typeclass obligations, applies admissible implicit completions, checks contracts, then transfers control to the callee. Arguments are matched positionally by the current Kyokai grammar.

> Trace: D53/D140/D142, D71, D82/D82a/D82b, D209/D161, D238-D240
> Covers: Calls are source-ordered, statically resolved, contract-checked, and use local argument-driven generic inference only.

Call-site generic type argument inference uses only the receiver and explicit value arguments of the call after ordinary desugaring. Return type context, assignment target type, `let` annotation, enclosing expression context, and surrounding obligations do not solve omitted type arguments.

> Trace: D12, D87, D209/D161
> Covers: Generic call inference is forward-only and argument-driven, never backward from result context.

A UFCS expression `receiver.name(args)` desugars to first-argument call shape after name resolution admits it. The receiver-module fallback is narrow: it runs only when ordinary imported-name lookup finds no candidate, and it searches only the receiver type's defining module or another compiler-known receiver owner.

> Trace: D7a, D108, D110/D254, D179
> Covers: UFCS is first-argument call sugar with constrained receiver-module lookup, not virtual dispatch, global search, or a pipeline operator.

Kyokai has no pipeline operator. UFCS is the built-in left-to-right call-chain surface.

> Trace: D108
> Covers: `|>` is rejected; receiver-dot calls carry the accepted pipeline-like reading.

## Construction And Update

A record construction expression `Type { field: expr, ... }` constructs an ordinary record value. Every declared field must be supplied exactly once after shorthand expansion and any explicit update source is processed. There are no hidden defaults and no positional record construction.

> Trace: D35, D177
> Covers: Record construction is field-total, name-based, and has no default-filling or positional form.

Record update `Type { field: expr, ..., with source }` evaluates explicit field expressions and the source in source order. Explicit fields override matching fields from the source. For a `Free` record, remaining fields are copied from the source and the source remains usable. For a `Linear` record, the source is consumed and remaining fields are moved.

> Trace: D71, D98, D138, D195
> Covers: Record update names its source, uses source-order evaluation, and copies or moves according to the record's universe.

Union construction uses three forms. A multi-field variant uses braces and named fields. A single-payload variant uses parentheses. A zero-payload variant is the bare constructor name. These forms do not create tuple values.

> Trace: D47/D131, D65
> Covers: Union construction is arity-explicit and does not introduce tuple construction.

Array literals use `[e1, e2, ...]`. The length is inferred by counting written elements. The element type must resolve to one concrete type through ordinary typing and literal-inference rules. Empty `[]` is legal only when the context already fixes `Array[T, 0]`.

> Trace: D12, D55, D71, D194
> Covers: Array literals infer only length, evaluate elements in source order, and do not perform implicit promotions.

## Field Access, Indexing, And Slicing

Field access uses `.`. If the base expression has type `&[Record]` or `&![Record]`, one level of auto-deref is inserted for direct field access and field assignment places. The auto-deref depth is exactly one. `->` is not part of Kyokai.

> Trace: D34, D87, D238-D240
> Covers: Field access has one-level reference auto-deref only, with no `->` and no unlimited deref chain.

Indexing `a[i]` is built-in sugar over the fixed language-defined `Indexable` protocol family. Mutable element assignment `a[i] := value` uses the mutable indexing protocol. Invalid indexing for the selected type's domain triggers TPOE.

> Trace: D36/D132, D84
> Covers: `[]` is total-or-TPOE indexing over a fixed protocol family, not ad hoc name lookup or sentinel-returning lookup.

Fallible lookup must use an explicitly named API such as `get`, `tryGet`, or another ordinary function. Kyokai does not make `[]` return `Optional` or silently wrap out-of-bounds access.

> Trace: D36/D132, D84
> Covers: Indexing syntax has one failure policy: contract violation by TPOE.

Slice syntax is checked half-open span extraction. `a[i..j]` selects the range from `i` up to but not including `j`; `a[i..]` goes to the length; `a[..j]` starts at zero; `a[..]` selects the whole span view. The required relation is `i <= j <= length(a)`, and violation triggers TPOE.

> Trace: D106, D210
> Covers: Slicing uses explicit half-open bounds with checked relation semantics, not raw `j - i` folklore.

On an immutable source, slicing produces an immutable span view. On a mutable source, slicing produces a mutable span view when the mutable-borrow rules allow it. `String` does not gain direct slicing or indexing syntax unless a later explicit rule adds it; text-to-bytes access stays named.

> Trace: D30, D30a, D106, D187
> Covers: Slice result mutability follows source access, while text indexing/slicing is not implicit.

## Operators

Kyokai uses a small precedence table. Postfix field/call/indexing bind tightest; prefix borrow, dereference, unary negation, `not`, and `bnot` follow; then `*`, `/`, `%`; then `+`, binary `-`, `++`; then comparisons; then `and`; then `or`. Same-level operators associate left to right unless another rule says otherwise.

> Trace: D10, D41, D56, D57
> Covers: Operator grouping is limited and explicit, with left-to-right associativity at the same level.

Comparison and equality do not chain. `a < b < c` and `a = b = c` are illegal. Bitwise, shift, and rotate operators do not mix implicitly with arithmetic, comparison, Boolean operators, or different bitwise/shift/rotate operators. Parentheses are required for mixed families.

> Trace: D41, D56
> Covers: Risky operator mixes and chained comparisons are rejected unless grouping is explicit.

Built-in integer arithmetic has no promotions, no usual arithmetic conversions, and no mixed signed/unsigned operator rules. A binary integer operator is legal only when both operands have the same concrete integer type after literal inference and explicit conversions. Overflow, division by zero, remainder by zero, and signed minimum divided by `-1` trigger TPOE where the integer rule names them.

> Trace: D12, D75, D84, D210
> Covers: Integer arithmetic is same-type, checked by default, and traps by TPOE for specified invalid results.

Integer division truncates toward zero. Integer remainder is the truncating remainder and has the sign of the dividend. Wrapping, saturating, overflow-reporting, and other arithmetic policies exist only as explicitly named APIs.

> Trace: D75
> Covers: Default integer division and remainder semantics are fixed, and alternate arithmetic policies are not implicit.

Floating-point arithmetic uses IEEE 754 binary32 and binary64 for `Float32` and `Float64`. The default rounding mode is round-to-nearest, ties-to-even. Subnormals, signed zero, infinities, and NaNs follow the strict floating contract. Backend fast-math, reassociation, excess precision, and implicit FMA contraction may not change language-visible results.

> Trace: D76, D139, D228
> Covers: Float operations have a strict portable IEEE 754 contract and cannot be weakened by backend flags or profiles.

Floating division by zero does not trigger TPOE merely because the divisor is zero; it yields the IEEE 754 result. Ordered comparisons involving NaN are false. `!=` involving NaN is true. NaN payload propagation is not a portable language guarantee.

> Trace: D76
> Covers: Floating exceptional values follow IEEE 754 visible behavior rather than integer TPOE rules.

Operator overloading exists only through the fixed built-in operator typeclass family. Users cannot declare new operator tokens, change precedence, or make indexing, assignment, borrow syntax, Boolean short-circuiting, or field access user-overloadable by naming a method.

> Trace: D23, D82/D82a/D82b, D214
> Covers: Operator dispatch is closed, coherent, and static; arbitrary operator overloading does not exist.

## Conversions And Formatting

Kyokai has no implicit numeric conversions. Default numeric conversion uses the fixed UFCS-style built-in family such as `x.toInt32()`, `x.toIndex()`, and `x.toFloat64()`. Integer-to-integer and float-to-integer default conversions check representability and trigger TPOE on invalid checked narrowing. Non-default conversion policies use named APIs such as fallible, truncating, wrapping, or saturating forms.

> Trace: D37, D75, D76, D210
> Covers: Numeric conversion is explicit, named, checked by default, and never a cast expression or pseudo-constructor.

`Index` is a distinct concrete integer type. It is not interchangeable with `Int*` or `Nat*` merely because a target may represent them with the same width. Standard indexing, length, and capacity APIs use `Index` explicitly, and other integer families must convert explicitly.

> Trace: D37, D210
> Covers: Index-domain values remain explicit and do not inherit ambient integer interchangeability.

Text/byte and OS/foreign string conversions are never implicit. `String`, byte buffers, spans of bytes, C strings, OS strings, and paths have their own named conversion APIs and failure contracts.

> Trace: D30, D30a, D68, D97
> Covers: Text, bytes, C interop text, OS-native text, and paths do not silently convert into each other.

`format(alloc, template, args...)` is a built-in allocating formatting construct. The template is compile-time checked, uses `{}` placeholders only, requires the number of placeholders to match the number of trailing arguments, and requires every argument to satisfy `Displayable`. It returns `Result[String, AllocError]`; allocation failure is explicit.

> Trace: D40, D40a, D44, D74
> Covers: Allocating formatting is built in, compile-time checked, allocator-explicit, and fallible by `AllocError`.

`writeFmt` is the non-allocating formatted-output surface over `Writable`. It uses the same placeholder language and `Displayable` constraints as `format`, renders directly to the stream, and reports stream failure through an explicit `Result`.

> Trace: D40, D102
> Covers: Direct formatted output shares formatting checks but does not allocate a whole `String` result.

## Compile-Time Expressions

`comptime expr` forces evaluation of `expr` at compile time. The phase is visible at the call site. Kyokai has no definition-site `const fn` distinction that makes the same call syntax run in different phases depending on context.

> Trace: D18, D18a
> Covers: Compile-time evaluation is requested at the expression site and is not inferred from definition annotations.

A `comptime` expression may depend only on explicit compile-time inputs: literals, constants, other `comptime` results, `target`, and language-defined compile-time built-ins. It may not observe filesystem contents, environment variables, wall-clock time, locale, randomness, network state, process state, thread scheduling, runtime capabilities, or FFI.

> Trace: D18a, D117, D202, D203
> Covers: Compile-time evaluation is deterministic, host-independent, and cannot smuggle host state into a build.

All argument values and called functions used by a `comptime` evaluation must be eligible for compile-time evaluation. Parameter and return types must be `Free`; runtime resources and linear owning values cannot exist as compile-time values. Called functions are checked lazily when a `comptime` expression tries to evaluate them.

> Trace: D18, D18a, D194, D195, D202, D203
> Covers: `comptime` evaluates pure `Free` data only and rejects runtime resources or non-eligible operations at the use site.

Each top-level compile-time evaluation root has one deterministic step budget. A root is a constant initializer, `static_assert` condition, `when` guard, or explicit `comptime` expression currently being evaluated. Exhausting the budget is a compile-time error. Wall-clock time is not part of the language budget model.

> Trace: D18a, D202, D203, D215
> Covers: Compile-time evaluation has deterministic budgeted execution and budget failure is a compile-time error.

`static_assert(condition, "message")` evaluates its Boolean condition at compile time. A false condition is a compile-time error carrying the supplied message. It is not a runtime assertion and cannot be stripped by a build profile.

> Trace: D18a, D202, D203
> Covers: Static assertions are compile-time checks with no runtime profile weakening.

## Closures And Function Pointers

`FnPtr(...) : Ret` is a bare function pointer with no captured environment. Closure literals use explicit capture lists and lower to the callable-family substrate. A zero-capture closure writes `[]`.

> Trace: D21, D118, D126, D197
> Covers: Function pointers and closures are distinct callable surfaces; closures never infer hidden captures.

A closure may capture only the names written in its capture list. Captures are by value, immutable borrow, or mutable borrow according to the written capture item. The closure chapter defines the full callable-family lowering, but expression evaluation here is simple: the capture list is evaluated at closure creation in source order.

> Trace: D118, D126, D197
> Covers: Closure creation has explicit source-written captures and no implicit environment analysis.

## Result Placement And Backend Preservation

Moving a value has as-if bytewise relocation semantics. The backend may optimize moves, use registers, or lower through output pointers, but the observable result must match the Kyokai move model.

> Trace: D89, D139, D228
> Covers: Move semantics are defined by Kyokai and cannot be replaced by backend folklore.

Large return values use guaranteed direct result placement when the return type is larger than two machine words on the selected target. The callee initializes storage designated as the caller's final result object. This is a performance and lowering contract, not a stable-address guarantee for safe self-referential values.

> Trace: D89, D199, D228
> Covers: Large return values avoid mandatory full-width return copies while preserving ordinary move semantics.

Backend lowering must preserve Kyokai evaluation order, checks, fatal paths, aliasing assumptions, and failure behavior. If a backend cannot implement a construct without relying on backend undefined behavior or weakening Kyokai semantics, the build fails.

> Trace: D73, D139, D228
> Covers: Expression lowering cannot use C or LLVM undefined behavior as the implementation of safe Kyokai semantics.
