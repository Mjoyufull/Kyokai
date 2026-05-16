# Syntax Rationale

[Rikona Kurasaki / Mjoyufull]
Syntax gets mocked because people fight over it. That does not make syntax fake. It is the shape a programmer sees at 2 a.m., the map they use to find the end of a block, the mark that says whether a value was borrowed, moved, matched, or returned. Kyokai treats syntax as ergonomics in service of semantics.

> Trace: D9, D11a, D14, D35, D52
> Covers: Kyokai syntax makes semantic boundaries visible through terminators, naming, borrows, construction, and file forms.

## Statement Orientation

[Rikona Kurasaki / Mjoyufull]
Kyokai keeps the statement/expression split because systems code benefits from visible sequencing. Expression-oriented languages gain symmetry, but symmetry can also invite dense nests where allocation, branching, binding, and cleanup blur into one shape. Kyokai lets expressions compute values and statements move the program.

> Trace: D43/D128, D59, D71
> Covers: Loops are statements, assignment is a statement, and expression/statement evaluation order is explicit.

This does not make the language hostile to concise code. It means control flow that changes ownership or cleanup state should look like control flow. `let ... else`, `or return`, `defer`, `errdefer`, `borrow ... drop;`, and `taskgroup ... join;` are visible because they change the proof the checker is carrying.

> Trace: D2, D15/D15a, D111/D127, D246, D252
> Covers: Syntax that changes failure, cleanup, borrow, or task state is source-visible.

## Words, Symbols, And Searchability

[Rikona Kurasaki / Mjoyufull]
Borretti's Austral argument still works: words are easier to search and remember than punctuation clusters. Kyokai keeps English-like keywords where the word names a semantic action, but it does not pretend source code is natural language. A formal language should not dress itself up as a conversation.

> Trace: D9, D13, D15, D41, D63
> Covers: Kyokai uses keyword operators and named control forms while keeping formal grammar boundaries.

Kyokai does use symbols where the symbol is already a strong visual contract: `&`, `&!`, and `&~` for borrowing, `!=` for inequality, `[T]` for generic arguments, and file extensions for interface/body separation. The point is not words over symbols as a superstition. The point is that the mark should teach the boundary faster than it decorates the page.

> Trace: D7b, D10, D14, D52, D158/D189
> Covers: Symbol use is admitted when it carries a clear, stable semantic boundary.

## Boundary Terminators

[Rikona Kurasaki / Mjoyufull]
Kyokai's reversed and semantic terminators are not a costume. `fi;`, `od;`, `esac;`, `qed;`, `drop;`, `spec;`, and `mon;` tell the reader what boundary just closed. Braces tell you something ended. Kyokai terminators tell you what ended.

> Trace: D9, D13, D111/D127, D182
> Covers: Kyokai terminators encode construct boundaries and keep nested code readable.

`qed;` is the proof-shaped close for functions and implementation bodies. `mon;` closes the foreign gate. `drop;` closes a borrow scope. The words are small, but they are not interchangeable; the spelling carries the semantic posture of the block.

> Trace: D20, D53, D111/D127, D155
> Covers: Function, foreign, and borrow boundaries use distinct terminators tied to their semantic role.

## Naming As Contract

[Rikona Kurasaki / Mjoyufull]
Kyokai naming is not a style contest. `as*`, `to*In`, `into*`, `into*In`, `cloneIn`, `collectIn`, and `must*` are small promises about borrowing, allocation, consumption, and fatal failure. A caller should not have to read a body to know whether a helper allocates or consumes.

> Trace: D11a-D11b, D74, D201
> Covers: Naming conventions expose ownership, allocation, and fatal-helper behavior.

## Honest Implicitness

[Rikona Kurasaki / Mjoyufull]
Kyokai is not allergic to convenience. It is allergic to guessing. The compiler may complete an operation only when the missing operation is forced by the typed program and adds no new effect story. That is why auto-reborrow can exist but hidden destructor calls cannot.

> Trace: D7b, D8, D34, D46, D87, D238-D240
> Covers: Implicit completions are tautological, effect-neutral, recorded, and checked in a fixed elaboration order.

## Syntax Result

[Rikona Kurasaki / Mjoyufull]
The result should feel strict but not ornate. Kyokai asks for a few extra marks so later readers can recover the shape of the program without archaeology: where the borrow ends, where the foreign boundary closes, where failure returns, where a resource moves, where a module surface begins.

> Trace: D52, D78, D111/D127, D155
> Covers: Syntax is designed to make program structure and semantic boundaries auditable.
