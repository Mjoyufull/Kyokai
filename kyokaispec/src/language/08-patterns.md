# Patterns

[Rikona Kurasaki / Mjoyufull]
A pattern is a hand reaching into a value. It can name what it takes, refuse what does not match, or step past free data with `ignore`. What it cannot do is make ownership vanish. If a payload owns something, the pattern must bring it into the light.

> Trace: D38, D98, D205, D206
> Covers: Kyokai patterns are structural, exhaustive where required, and cannot hide linear-resource discard.

Patterns appear in `let`, `let ... else`, `case`, `while let`, `for ... in`, and record/union destructuring contexts. The pattern grammar is shared, but each context decides whether the pattern must be irrefutable or may be refutable.

> Trace: D13, D15, D15a, D32, D39, D98, D205
> Covers: One structural pattern language is used across binding, matching, loop, and destructuring sites with context-specific refutability rules.

## Pattern Forms

An identifier pattern binds the matched value to that name. The new binding is subject to ordinary no-shadowing, visibility, type, borrow, and linear-use rules.

```kyokai
let item := value;
```

> Trace: D46, D60, D195
> Covers: Identifier patterns create ordinary bindings and cannot shadow still-live names.

`ignore` is the discard pattern. It matches a value and introduces no binding. `_` is not a Kyokai pattern token. `ignore` is contextual pattern syntax, not a global reserved identifier in every expression position.

> Trace: D38, D205, D206
> Covers: Kyokai uses `ignore` for discards, rejects `_`, and gives discard syntax pattern-only meaning.

A union constructor pattern matches a union variant. A zero-payload variant is written as the constructor. A one-payload variant is written as `Constructor(pattern)`. A named-field variant is written as `Constructor { field: pattern, ... }`, with shorthand `field` meaning `field: field`.

```kyokai
when None do
    return nil;
when Some(item) do
    process(item);
when RGB { red, green, blue } do
    draw(red, green, blue);
```

> Trace: D24, D65, D205
> Covers: Union patterns mirror union construction forms and may bind nested payloads.

A record pattern destructures a record by field name. `field: pattern` matches a field with a nested pattern. Shorthand `field` means `field: field` and binds the field value to a local with the same name.

```kyokai
let { sender, receiver } := makeChannel[Int32]();
let Point { x: px, y: py } := point;
```

> Trace: D35, D47, D98, D205
> Covers: Record patterns use named fields and do not create tuple destructuring.

Patterns may nest through records, unions, and combinations of both to any finite depth accepted by the implementation limits chapter. Nesting does not disable type checking, exhaustiveness, reachability, or linear-use checking.

> Trace: D98, D205, D206
> Covers: Nested structural patterns are legal and remain fully checked.

Pattern guards do not exist. A `case` arm is written `when Pattern do ...`. Boolean filtering after a structural match belongs inside the arm body with `if`.

> Trace: D38
> Covers: Kyokai separates structural matching from Boolean filtering and rejects guard clauses.

## Irrefutable And Refutable Patterns

An irrefutable pattern is guaranteed to match every value of its scrutinee type. Identifier patterns are irrefutable. Record patterns over a known record type are irrefutable when every nested pattern is irrefutable. A union constructor pattern is refutable unless the scrutinee type has exactly one visible variant and the nested payload pattern is irrefutable.

> Trace: D15, D15a, D38, D98, D205
> Covers: Refutability is structural and determined at compile time from the scrutinee type and nested pattern forms.

Plain `let Pattern := expr;` requires an irrefutable pattern. The initializer is evaluated once. The pattern then binds values from that result. If the programmer needs failure handling, they must use `let ... else`, an `or` suffix where admitted, `case`, or another explicit control-flow form.

> Trace: D15, D15a, D46, D87
> Covers: Plain `let` cannot fail at runtime; fallible binding must use explicit failure syntax.

`let Pattern := expr else Fallback do ... fi;` admits a refutable first pattern. The initializer is evaluated once. If the first pattern matches, its bindings enter scope after the statement. If it does not match, the `else` pattern and block handle the unmatched value according to the let-else chapter's control-flow rules. Any names introduced by the failure path are scoped only to that failure block.

> Trace: D15, D15a, D60, D205
> Covers: `let ... else` is the explicit fallible binding form with scoped success and failure bindings.

`while let Pattern := expr do ... od;` requires a refutable pattern. Each iteration evaluates `expr` once. A match enters the body with the pattern bindings in scope for that iteration. A mismatch terminates the loop immediately.

> Trace: D39
> Covers: `while let` is refutable-pattern loop sugar that re-evaluates the expression per iteration and breaks on mismatch.

`case expr of ... esac;` arms use refutable or irrefutable patterns, but the whole `case` must be exhaustive. Arms are tested in source order. The first matching arm executes. An unreachable arm is a compile-time error or diagnostic as specified by the exhaustiveness chapter.

> Trace: D13, D38, D205
> Covers: `case` is exhaustive, source-ordered structural matching with no guards.

`for Pattern in expr do ... od;` applies the pattern to each item produced by the iterator protocol. The pattern must be irrefutable for the iterator item type unless a later loop chapter explicitly defines a refutable-filtering loop form. Kyokai's accepted `for-in` does not silently skip nonmatching items.

> Trace: D32, D249, D205
> Covers: `for-in` pattern binding does not hide filtering; item matching must be total for the item type.

## Exhaustiveness And Reachability

A `case` over `Bool` must cover `true` and `false` unless an earlier pattern already makes a later case unreachable. A `case` over a union must cover every visible variant and every nested structural payload possibility relevant to the patterns written. A `case` over a `Free` type may use `ignore` as a catch-all only when that does not discard possible linear payloads.

> Trace: D24, D38, D205, D206
> Covers: Exhaustiveness is structural over booleans, unions, nested payloads, and legal catch-all patterns.

A catch-all `when ignore do ...` is legal only when every still-unmatched value reaching that arm is `Free`. If any possible unmatched value contains a `Linear` payload, the catch-all must be replaced by explicit patterns that bind the linear payloads and consume them.

> Trace: D195, D206
> Covers: Catch-all patterns cannot discard linear resources.

Pattern ordering matters only for reachability, not for meaning of an already selected pattern. If one arm makes a later arm unreachable, the compiler reports it. Import order, declaration order outside the union definition, and body-private variants of opaque types do not alter a client's visible exhaustiveness set.

> Trace: D17, D38, D205, D214
> Covers: Pattern arm selection is source ordered, and exhaustiveness is based on the visible type surface.

A non-exhaustive `case` is a compile-time error. Kyokai does not use a runtime match failure for accepted safe programs.

> Trace: D73, D84, D139, D205
> Covers: Safe accepted programs do not reach undefined or implicit runtime match failure because exhaustiveness is checked statically.

## Record Destructuring

Record destructuring is total. Destructuring a record consumes the whole record value and binds its fields as independent values. There is no partial move that leaves the original record alive.

> Trace: D98
> Covers: Record destructuring consumes the entire record and rejects Rust-style partial-move state.

Every linear field in a destructured record must be bound to a real name or nested pattern that ultimately binds and consumes it. Omitting a field is exact sugar for matching that field with `ignore`, so omission is legal only for `Free` fields.

> Trace: D98, D195, D206
> Covers: Omitted record pattern fields are `Free`-only; linear fields must stay visible.

A record pattern may mention fields in any order. Each mentioned field must exist exactly once. Duplicate field patterns are compile-time errors. A field not mentioned is omitted and therefore subject to the `Free`-only discard rule.

> Trace: D35, D98, D205, D206
> Covers: Record patterns are named, order-insensitive, duplicate-checked, and omission-checked.

A single-field record pattern is still a record pattern. It does not become a tuple pattern and does not unwrap by positional syntax. A single-field wrapper remains nominal even when its representation is transparent.

> Trace: D47, D109, D190, D196
> Covers: Single-field records destructure by named field only and remain nominal wrappers.

## Linear Movement Through Patterns

Pattern matching moves ownership of matched linear values into the selected bindings unless the scrutinee is borrowed and the pattern context is explicitly a borrow-pattern context defined by the borrow chapter. A moved linear binding must be consumed exactly once along every path from the binding's scope.

> Trace: D98, D195, D206, D238-D240
> Covers: Patterns move linear ownership by default and remain subject to exact-use checking.

A pattern may copy a `Free` value when the context needs the original value to remain usable. It may not copy a `Linear` value. When a record update or destructuring operation over a linear record moves remaining fields, the source value is consumed.

> Trace: D98, D138, D195
> Covers: Free pattern data may be copied where allowed; linear pattern data moves and consumes the source.

Every arm of a `case` has its own linear obligations. A linear value bound in one arm must be consumed in that arm. A linear value that exists before the `case` must have a valid ownership state after every arm that can complete normally. `return`, `break`, `continue`, `panic`, `todo`, and `unreachable` interact with these obligations through the control-flow and cleanup chapters.

> Trace: D2, D84, D121, D191, D195, D205, D246
> Covers: Linear obligations are checked per arm and per control-flow exit path.

`ignore` never calls a destructor, `Destroyable.destroy`, `defer`, or any other cleanup operation. If a linear value needs cleanup, the program must bind it and call the cleanup API or register visible cleanup before leaving the path.

> Trace: D2, D89, D157, D195, D206, D246
> Covers: Discard syntax does not perform hidden cleanup; linear cleanup remains explicit.

## Pattern Type Checking

A pattern is type checked against the scrutinee type supplied by its context. Plain `let name := expr;` may infer the binding type only when the initializer already has one fully determined type. Pattern forms do not add a new independent inference engine.

> Trace: D46, D87, D190
> Covers: Pattern typing flows from the scrutinee expression and allowed local binding inference, not from a separate pattern-inference system.

Constructor patterns resolve by the scrutinee type and visible constructors. If a constructor name is ambiguous or not visible, the program must qualify or import names so ordinary name lookup resolves it. Pattern matching does not get a wildcard import or global constructor search exception.

> Trace: D17, D24, D65, D179, D214
> Covers: Pattern constructor lookup follows ordinary visibility and import rules.

Associated-type projections and generic scrutinee types must be resolved before exhaustiveness is decided. If an associated type remains unknown because no governing instance is known, the pattern site is ill-formed rather than guessed.

> Trace: D33, D82, D189, D190, D205
> Covers: Exhaustiveness over generic and associated-type surfaces requires resolved static type information.

A pattern cannot match by runtime type identity, reflection, subclass tests, exception type, trait object shape, or hidden representation. Kyokai patterns are structural over the static type forms admitted here.

> Trace: D47, D147, D190, D193, D205
> Covers: Pattern matching has no runtime typecase, inheritance, trait-object, exception, or reflection matching surface.
