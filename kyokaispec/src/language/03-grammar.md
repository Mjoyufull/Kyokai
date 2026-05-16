# Grammar

[Rikona Kurasaki / Mjoyufull]
Kyokai keeps Austral's central grammatical idea: a module has an importable interface and an implementation body. The split still matters because the boundary still matters. A `.kyo` file is the public contract surface. A `.kai` file is where the work is done, where private helpers live, and where platform-specific bodies can be selected without changing the interface people import.

> Trace: D5, D17, D52, D78, D86
> Covers: Kyokai is a fork that keeps Austral's useful interface/body split while standardizing `.kyo` interfaces, `.kai` bodies, package-visible `internal`, and deterministic module resolution.

This chapter gives the source grammar shape. Later chapters define name resolution, type checking, ownership, borrowing, evaluation, layout, contracts, FFI, unsafe obligations, concurrency, and runtime failure. If this chapter admits a form, that does not make every use type-correct; it only says the parser can recognize the shape.

> Trace: D86, D87, D155
> Covers: Syntax admission is separate from semantic acceptance, and accepted Kyokai behavior is defined by the normative spec rather than by inherited implementation accidents.

## Start Symbols

[Rikona Kurasaki / Mjoyufull]
Kyokai has two source-file start symbols.

```ebnf
interface file = file docs, {import declaration}, module interface;
body file = file docs, {pragma declaration}, {import declaration}, module body;
```

A `.kyo` file must match `interface file`. A `.kai` file must match `body file`. Imports are file-scope declarations. A source file may not contain more than one module declaration.

> Trace: D52, D78, D179
> Covers: `.kyo` and `.kai` have distinct start symbols, imports are file-scope only, and Kyokai keeps one module per source file.

`file docs` is zero or more `//!` documentation comments. Declaration documentation uses `///` immediately before the declaration it documents. Documentation comments do not change the grammar category of the item they document.

> Trace: D63
> Covers: Kyokai documentation comments use `//!` and `///` line forms rather than Austral triple-quote docstrings.

## Modules And Imports

[Rikona Kurasaki / Mjoyufull]
A module boundary is sealed. The reader should be able to walk into a file, see its imports at the front, see the module name, and know exactly when the symbol table closes.

```ebnf
module interface = "module", module name, "is", {interface declaration}, "seal", ";";
module body = "module", "body", module name, "is", {body declaration}, "seal", ";";
```

> Trace: D9, D52, D78
> Covers: Kyokai modules use `module Name is ... seal;` and `module body Name is ... seal;`, with `seal;` as the module-boundary terminator.

Kyokai import syntax has exactly three forms.

```ebnf
import declaration = qualified import | module alias import | selective import;
qualified import = "import", module name, ";";
module alias import = "import", module name, "as", module identifier, ";";
selective import = "import", module name, "(", [import item list], ")", ";";
import item list = import item, {",", import item}, [","];
import item = identifier, ["as", identifier];
```

`import Foo.Bar;` introduces the module for qualified access. `import Foo.Bar as Bar;` introduces the module under the alias `Bar`. `import Foo.Bar (baz, qux as localQux);` introduces only the listed exported names unqualified after applying explicit renames. Wildcard imports, `open`, function-local imports, block-local imports, and expression-local imports do not exist.

> Trace: D78, D179, D214
> Covers: Kyokai imports are file-scope only, have exactly qualified/module-alias/selective forms, reject wildcard or open imports, and report import collisions at the import site.

## Declarations

[Rikona Kurasaki / Mjoyufull]
Interface declarations are the surface another module can rely on. Body declarations are the module's own machinery. The grammar keeps those categories separate so visibility is not a rumor carried by convention.

```ebnf
interface declaration = declaration docs, ["internal"], interface item;
interface item = constant declaration
               | type alias declaration
               | opaque type declaration
               | extern type declaration
               | record declaration
               | union declaration
               | capability declaration
               | function declaration
               | typeclass declaration
               | instance declaration
               | generator declaration;

body declaration = declaration docs, body item;
body item = constant definition
          | type alias declaration
          | extern type declaration
          | record declaration
          | union declaration
          | capability declaration
          | function definition
          | typeclass declaration
          | instance definition
          | generator definition
          | foreign block
          | unsafe contract;
```

`internal` is legal only in `.kyo` interface files. A declaration with no `internal` marker in an interface is public. A declaration that exists only in a `.kai` body is private to that module. Module-level `var` is illegal.

> Trace: D17, D62, D78
> Covers: Kyokai has public interface declarations, package-visible `internal` interface declarations, private body-only declarations, and no module-level mutable variables.

A declaration may carry a declaration-level `when` guard. A false guard makes the declaration semantically absent for the selected target. `when` guards are not statements and are not allowed inside function bodies.

```ebnf
guarded declaration suffix = ["when", expression];
```

> Trace: D19, D19a, D123
> Covers: Kyokai conditional compilation uses whole-file selection, declaration-level `when` guards, and typeclass abstraction, with no body-level target branching.

## Constants, Types, Records, Unions, And Capabilities

[Rikona Kurasaki / Mjoyufull]
Top-level constants are immutable. They may be declared in an interface and defined in a body, or defined directly where the declaration category permits a definition.

```ebnf
constant declaration = "constant", identifier, ":", type, guarded declaration suffix, ";";
constant definition = "constant", identifier, ":", type, ":=", expression, guarded declaration suffix, ";";
type alias declaration = "type", "alias", type name, [generic parameters], ":=", type, guarded declaration suffix, ";";
opaque type declaration = "type", type name, [generic parameters], ":", universe, guarded declaration suffix, ";";
extern type declaration = "extern", "type", type name, guarded declaration suffix, ";";
capability declaration = "capability", type name, guarded declaration suffix, ";";
```

> Trace: D17, D24, D50, D61, D78, D255
> Covers: Kyokai admits constants, aliases, opaque types, extern types, and sealed capability declarations while keeping capability constructors unforgeable.

Records have three layout classes: ordinary Kyokai records, C-ABI extern records, and byte-tight packed records. These are grammar choices, not backend hints.

```ebnf
record declaration = record header, record body;
record header = ["extern" | "packed"], "record", type name, [generic parameters], [":", universe];
record body = "is", {field declaration}, "build", ";"
            | "(", single field, ")", ":", universe, ";";
field declaration = declaration docs, identifier, ":", type, ";";
single field = identifier, ":", type;
```

The one-line record form is legal only for a single-field ordinary record. `extern record` and `packed record` use the block form so their layout boundary stays visible.

> Trace: D35, D42, D109, D116, D196
> Covers: Kyokai has `record`, `extern record`, and `packed record`; single-field records are the nominal wrapper mechanism; and record declarations close with `build;`.

Unions declare named variants. A variant may carry no payload, one unnamed payload type, or named fields.

```ebnf
union declaration = "union", type name, [generic parameters], [":", universe], "is", {union variant}, "build", ";";
union variant = "case", constructor name, ";"
              | "case", constructor name, "(", type, ")", ";"
              | "case", constructor name, "is", {field declaration};
```

> Trace: D47, D54, D65, D131
> Covers: Kyokai sum types are named unions, have explicit variant payload forms, close with `build;`, and do not create tuple syntax.

## Functions, Contracts, Typeclasses, And Instances

[Rikona Kurasaki / Mjoyufull]
A function signature is a small contract room: name, parameters, return type, generic obligations, value obligations, then the body if this file owns one.

```ebnf
function declaration = "function", identifier, [generic parameters], "(", [parameter list], ")", ":", type,
                       [where clause], {contract clause}, guarded declaration suffix, ";";
function definition = "function", identifier, [generic parameters], "(", [parameter list], ")", ":", type,
                      [where clause], {contract clause}, guarded declaration suffix,
                      "is", block, "qed", ";";
parameter list = parameter, {",", parameter}, [","];
parameter = identifier, ":", type;
contract clause = require clause | ensure clause;
require clause = "require", expression, ";";
ensure clause = "ensure", expression, ";";
```

`require` and `ensure` clauses sit between the signature and the body or terminating semicolon. `result` is available only inside `ensure` clauses for non-`Unit` functions as a read-only view of the produced return value. `old expr` is available only inside `ensure` and only for pure entry-state expressions over `Free` data.

> Trace: D53, D125, D129, D140, D142
> Covers: Kyokai function contracts use `require` and `ensure`, `result` is contextual inside postconditions, `old` snapshots pure entry-state `Free` expressions, and contract failures are TPOE.

Generic parameters use brackets after the declared name. `Type`, `Free`, and `Linear` are parameter constraints. `Auto` is a declaration-site classifier, not a generic bound users can pass around as a loose promise.

```ebnf
generic parameters = "[", generic parameter, {",", generic parameter}, [","], "]";
generic parameter = type parameter | const generic parameter | region parameter;
type parameter = type name, ":", generic classifier;
generic classifier = "Type" | "Free" | "Linear";
const generic parameter = identifier, ":", "Index";
region parameter = identifier, ":", "Region";
where clause = "where", where obligation, {",", where obligation}, [","];
where obligation = type, ":", type name
                 | associated type projection, ":", type name
                 | associated type projection, "==", type;
```

> Trace: D158, D159, D188, D189, D190, D192, D193, D195
> Covers: Kyokai uses rank-1 generic parameter lists, admits explicit `Index` const generic parameters, uses a closed `where` grammar for constraints and associated-type equality, and rejects higher-rank, existential, and opaque return type surfaces.

Typeclasses define contracts. Instances provide witnesses. Both proof bodies close with the same `qed;` boundary as functions when they contain implementation.

```ebnf
typeclass declaration = "typeclass", type name, [generic parameters], "is", {method declaration}, "spec", ";";
method declaration = "method", identifier, [generic parameters], "(", [parameter list], ")", ":", type,
                     [where clause], {contract clause}, ";"
                   | "method", identifier, [generic parameters], "(", [parameter list], ")", ":", type,
                     [where clause], {contract clause}, "is", block, "qed", ";";
instance declaration = "instance", type name, [generic parameters], "for", type, [where clause], guarded declaration suffix, ";";
instance definition = "instance", type name, [generic parameters], "for", type, [where clause], guarded declaration suffix,
                      "is", {method definition}, "qed", ";";
method definition = "method", identifier, [generic parameters], "(", [parameter list], ")", ":", type,
                    [where clause], {contract clause}, "is", block, "qed", ";";
```

> Trace: D182, D195, D214
> Covers: Kyokai typeclasses close with `spec;`, may contain default method bodies, and instances close with `qed;` while remaining subject to ordinary name and visibility rules.

## Foreign Blocks And Unsafe Contracts

[Rikona Kurasaki / Mjoyufull]
Foreign code is a gate, not a hallway. The grammar makes the gate visible, then the unsafe chapter defines what must be audited before anyone is allowed to walk through it.

```ebnf
pragma declaration = "pragma", pragma name, ";";
foreign block = "foreign", string literal, "is", {foreign declaration}, "mon", ";";
foreign declaration = "function", identifier, "(", [parameter list], ")", ":", type, ";"
                    | "constant", identifier, ":", type, ";";
unsafe contract = "unsafe", "contract", type name, "is", {unsafe contract item}, "audit", ";";
```

Only `foreign "C" is ... mon;` is admitted by this grammar. Raw foreign declarations are legal only in a module marked with `pragma Unsafe_Module;`, and that module must contain source-level unsafe contracts covering the unsafe operations it uses.

> Trace: D20, D20a, D20b, D127, D242, D242a, D245
> Covers: Kyokai raw FFI uses `foreign "C" is ... mon;`, requires `pragma Unsafe_Module;`, forbids implicit linear ownership transfer and implicit sum-type ABI across raw C, and requires audited unsafe contracts.

## Types

[Rikona Kurasaki / Mjoyufull]
Type syntax names ownership and authority boundaries directly. References are not comments in the margin; they are part of the type.

```ebnf
type = type atom
     | module path
     | type application
     | immutable reference type
     | mutable reference type
     | function pointer type;
type application = type atom, "[", type argument, {",", type argument}, [","], "]";
immutable reference type = "&", "[", type, [",", region], "]";
mutable reference type = "&!", "[", type, [",", region], "]";
function pointer type = "FnPtr", "(", [type list], ")", ":", type;
universe = "Type" | "Free" | "Linear" | "Auto";
```

`&[T]` is an immutable borrow type. `&![T]` is a mutable borrow type. Region arguments may be written when the region chapter admits them; the common spelling omits them. `FnPtr(...) : Ret` is the bare function-pointer surface for C interop and dispatch tables. Tuples do not exist.

> Trace: D14, D21, D47, D126, D131, D187, D195
> Covers: Kyokai type syntax includes explicit immutable and mutable references, a bare `FnPtr` callback form, universe constraints, and no tuple type syntax.

## Statements And Blocks

[Rikona Kurasaki / Mjoyufull]
A block is a sequence of statements. Empty blocks are allowed. Statements end in semicolons unless their closing keyword already includes the semicolon.

```ebnf
block = {statement};
statement = let statement
          | let else statement
          | var statement
          | assignment statement
          | expression statement
          | return statement
          | break statement
          | continue statement
          | defer statement
          | errdefer statement
          | panic statement
          | todo statement
          | unreachable statement
          | debug statement
          | if statement
          | case statement
          | while statement
          | while let statement
          | for range statement
          | for in statement
          | borrow statement
          | taskgroup statement
          | spawn statement
          | select statement
          | yield statement;
```

> Trace: D9, D16, D180
> Covers: Kyokai has statement-oriented syntax with semicolons, explicit block terminators, empty no-op blocks, and insignificant newlines.

Local immutable binding uses `let`; local mutable binding uses `var`. Assignment is a statement and never an expression.

```ebnf
let statement = "let", pattern, [":", type], ":=", expression, [or clause], ";";
let else statement = "let", pattern, [":", type], ":=", expression, "else", pattern, "do", block, "fi", ";";
var statement = "var", identifier, ":", type, [":=", expression], ";";
assignment statement = place, ":=", expression, ";";
or clause = "or", "return", [identifier, "=>", expression]
          | "or", "break", [loop label]
          | "or", "continue", [loop label];
```

`or return`, `or break`, and `or continue` are statement suffixes for fallible binding forms. They are not general expression operators. Assignment produces no value and cannot be chained.

> Trace: D15, D15a, D58, D59, D60
> Covers: Kyokai has `let...else` and `or ...` fallible binding sugar, statement-only assignment, no binding shadowing, and expression-site `Never` coercion for diverging exits.

Control-flow statements use explicit open and close words.

```ebnf
if statement = "if", expression, "then", block, {"else", "if", expression, "then", block}, ["else", block], "fi", ";";
case statement = "case", expression, "of", {case arm}, "esac", ";";
case arm = "when", pattern, "do", block;
while statement = "while", expression, "do", block, "od", ";";
while let statement = "while", "let", pattern, ":=", expression, "do", block, "od", ";";
for range statement = "for", identifier, "from", expression, ("to" | "below"), expression, "do", block, "od", ";";
for in statement = "for", pattern, "in", expression, "do", block, "od", ";";
```

`case` arms do not have guard clauses. Boolean filtering belongs inside the arm body with `if`. `for ... from ... to ...` is inclusive. `for ... from ... below ...` is exclusive. `for ... in ...` uses the language-defined iterator protocol.

> Trace: D13, D32, D38, D39, D56, D180, D205, D206, D249
> Covers: Kyokai uses `if/fi`, `case/esac`, `while/od`, range loops, `for-in`, exhaustive structural pattern matching, `while let`, and no pattern guards.

Borrow scopes make reference lifetime visible in source.

```ebnf
borrow statement = "borrow", identifier, ":=", borrow expression, "do", block, "drop", ";";
borrow expression = "&", place | "&!", place | "&~", place;
```

`&` creates an immutable borrow, `&!` creates a mutable borrow, and `&~` is the explicit reborrow surface where the borrow chapter admits it. The borrow scope ends at `drop;`.

> Trace: D7b, D14, D34, D87, D111, D187, D238-D240
> Covers: Kyokai borrow syntax is explicit, borrow scopes close with `drop;`, and accepted implicit reborrow completions are checked through the elaboration pipeline rather than guessed by syntax.

`defer` and `errdefer` register visible cleanup work. `debug` observes existing values only. `todo`, `panic`, and `unreachable` are explicit fatal or divergent statement forms, not optimizer folklore.

```ebnf
defer statement = "defer", statement;
errdefer statement = "errdefer", statement;
return statement = "return", [expression], ";";
break statement = "break", [loop label], ";";
continue statement = "continue", [loop label], ";";
panic statement = "panic", expression, ";";
todo statement = "todo", [string literal], ";";
unreachable statement = "unreachable", ";";
debug statement = "debug", expression, ";";
```

> Trace: D2, D8, D84, D89, D121, D122, D191, D233, D246
> Covers: Kyokai has visible cleanup statements, explicit divergence and fatal paths, named break/continue labels, debug-observation purity, and implicit `Unit` completion only at the end of `Unit` functions.

## Concurrency Statements

[Rikona Kurasaki / Mjoyufull]
Task syntax is built like a lit room at night: you can see where children start, what they carry, where the parent waits, and where thread creation can fail.

```ebnf
taskgroup statement = "taskgroup", "do", block, "join", ";";
spawn statement = "spawn", capture list, "do", block, "od", [spawn failure arm], ";";
spawn failure arm = "else", identifier, "do", block, "fi";
capture list = "[", [capture item list], "]";
capture item list = capture item, {",", capture item}, [","];
capture item = identifier | "&", identifier | "&!", identifier;
```

A `spawn` statement is legal only inside a `taskgroup`. Spawn capture lists are mandatory. For spawned tasks, by-value capture and immutable-borrow capture are admitted by the concurrency chapter; mutable `&!` capture is rejected for `spawn` even though closure literals use the same visual capture list family.

> Trace: D3, D88, D164, D235, D248, D252
> Covers: Kyokai uses structured `taskgroup do ... join;`, explicit `spawn [captures] do ... od`, fallible spawn failure arms, 1:1 OS tasks, and no implicit child-task capture.

`select` is the structured channel-wait form. Its exact channel-arm expressions and readiness semantics are defined by the concurrency chapter; this grammar fixes the boundary and arm shape.

```ebnf
select statement = "select", {select arm}, [select timeout arm], "pick", ";";
select arm = "when", expression, "do", block;
select timeout arm = "timeout", "(", expression, ")", "do", block;
```

> Trace: D3a, D3b, D90, D91, D92, D93
> Covers: Kyokai has a visible `select ... pick;` concurrency boundary whose channel and timeout semantics are specified in the concurrency chapter.

## Expressions

[Rikona Kurasaki / Mjoyufull]
Expressions are where Kyokai stays plain on purpose. Calls look like calls. Construction looks like construction. Field access is not secretly pointer syntax. The reader should not need a folklore ladder from C taped to the wall.

```ebnf
expression = literal
           | identifier
           | module path
           | "nil"
           | "true" | "false"
           | "Ok", "(", expression, ")"
           | "Err", "(", expression, ")"
           | "Some", "(", expression, ")"
           | "None"
           | call expression
           | ufcs expression
           | field access expression
           | index expression
           | slice expression
           | record construction
           | union construction
           | array literal
           | closure literal
           | comptime expression
           | static string expression
           | static assert expression
           | unary expression
           | binary expression
           | parenthesized expression;
```

> Trace: D24, D35, D36, D47, D54, D65, D106, D118, D120, D131
> Covers: Kyokai expression grammar includes built-in result/optional constructors, calls, UFCS, field access, indexing, slicing, construction, arrays, closures, compile-time forms, and no tuple expression syntax.

Calls use parentheses. UFCS `receiver.name(args)` is sugar for first-argument function call according to the name-resolution chapter, with the narrow receiver-module fallback only when ordinary imported lookup finds no candidate. Field access uses `.`, including one level of auto-deref through `&[Record]` or `&![Record]`. `->` is not a field-access operator.

```ebnf
call expression = expression, "(", [argument list], ")";
ufcs expression = expression, ".", identifier, "(", [argument list], ")";
field access expression = expression, ".", identifier;
argument list = expression, {",", expression}, [","];
```

> Trace: D7a, D34, D110, D254
> Covers: Kyokai uses call syntax, UFCS as first-argument call sugar with narrow receiver-module fallback, one-level field auto-deref, and no `->` operator.

Record construction uses braces and field names. Record update names the source explicitly with `with source`. Union construction uses braces for multi-field variants, parentheses for one-field variants, and a bare constructor for zero-field variants.

```ebnf
record construction = type, "{", record field init list, [",", "with", expression], [","], "}";
record field init list = [record field init, {",", record field init}];
record field init = identifier, ":", expression | identifier;
union construction = constructor name, "{", record field init list, [","], "}"
                   | constructor name, "(", expression, ")"
                   | constructor name;
array literal = "[", [expression, {",", expression}, [","]], "]";
```

> Trace: D35, D55, D65, D98, D109, D138
> Covers: Kyokai construction uses visible record and union forms, explicit `with source` record update, array literals with length inference only, and no hidden defaults or positional record construction.

Indexing and slicing use brackets. Indexing is the total-or-TPOE surface over the language-defined indexing protocol. Slicing is half-open and checked.

```ebnf
index expression = expression, "[", expression, "]";
slice expression = expression, "[", [expression], "..", [expression], "]";
```

> Trace: D36, D106
> Covers: Kyokai uses `a[i]` indexing and `a[i..j]` half-open slicing through closed checked container protocols.

Closure literals have explicit capture lists. A zero-capture closure writes `[]`.

```ebnf
closure literal = "fn", capture list, "(", [parameter list], ")", ":", type, "is", block, "qed", ";"
                | "fn", capture list, "(", [parameter list], ")", ":", type, "=>", expression;
```

> Trace: D21, D118, D126, D197
> Covers: Kyokai closure literals have mandatory explicit capture lists, block and one-expression forms, and lower to the fixed callable-family substrate.

Compile-time evaluation is visible at the call site. `static_assert` is a compile-time assertion form. `static "..."` is the bridge from ordinary escaped source text to `StaticString`.

```ebnf
comptime expression = "comptime", expression;
static string expression = "static", string literal;
static assert expression = "static_assert", "(", expression, ",", string literal, ")";
```

> Trace: D18, D18a, D120
> Covers: Kyokai uses call-site `comptime`, compile-time static assertions, and explicit `static "..."` text bridging.

Operator precedence is deliberately small.

| Level | Operators |
| --- | --- |
| 1 | postfix `.`, call `(...)`, indexing/slicing `[...]` |
| 2 | prefix `&`, `&!`, `~`, unary `-`, `not`, `bnot` |
| 3 | `*`, `/`, `%` |
| 4 | `+`, binary `-`, `++` |
| 5 | `<`, `<=`, `>`, `>=`, `=`, `!=` |
| 6 | `and` |
| 7 | `or` |

Operators at the same level associate left-to-right unless another rule says otherwise. Comparisons and equality do not chain. Bitwise, shift, and rotate operators do not mix implicitly with arithmetic, comparison, boolean operators, or each other except for same-operator chaining; parentheses are required.

> Trace: D10, D41, D56, D57
> Covers: Kyokai uses `!=`, short-circuiting boolean operators, keyword bitwise operators, and a limited precedence table with explicit grouping for risky mixes.

## Patterns

[Rikona Kurasaki / Mjoyufull]
Patterns are structural, but they do not become a trapdoor for throwing resources away.

```ebnf
pattern = identifier
        | "ignore"
        | constructor name
        | constructor name, "(", pattern, ")"
        | constructor name, "{", [record pattern fields], [","], "}"
        | "{", [record pattern fields], [","], "}";
record pattern fields = record pattern field, {",", record pattern field};
record pattern field = identifier
                     | identifier, ":", pattern;
```

`ignore` is the discard pattern. `_` is not a pattern token. Nested patterns are allowed. Pattern guards do not exist. Omitting a record field in a pattern is sugar for discarding that field, so it is legal only when the omitted field is `Free`. Linear payloads must be bound and consumed explicitly.

> Trace: D38, D98, D205, D206
> Covers: Kyokai patterns support nested union and record structure, use contextual `ignore`, reject `_`, reject guards, and forbid hidden discard of linear values.

## Generators

[Rikona Kurasaki / Mjoyufull]
A generator is a named iterator declaration. It can pause, but it does not open the door to async, stackful coroutines, or opaque return types.

```ebnf
generator declaration = generator header, [where clause], guarded declaration suffix, ";";
generator definition = generator header, [where clause], guarded declaration suffix, "is", block, "qed", ";";
generator header = "generator", type name, [generic parameters], "(", [parameter list], ")", ":", type;
yield statement = "yield", expression, ";";
```

`yield` is legal only inside a generator body. A generator declaration in an interface exposes the generator's source-level contract. The matching generator definition in a body creates the nominal linear iterator type and constructor function according to the generator chapter.

> Trace: D32, D118, D193, D198, D249
> Covers: Kyokai has named stackless pull generators with `yield`, nominal linear iterator types, explicit destruction for suspended state, and no general coroutine or async surface.

## Grammar Forms Not In Kyokai

[Rikona Kurasaki / Mjoyufull]
The grammar intentionally rejects several shapes that would make the language bigger without making programs clearer: tuple syntax, class declarations, inheritance syntax, exceptions, `try/catch`, wildcard imports, block-local imports, macro definitions, pipeline `|>`, body-level target `when`, pattern guards, `_` discard patterns, `Drop` declarations, implicit destructor hooks, general `async`/`await`, and safe module-level mutable globals.

> Trace: D47, D62, D108, D123, D147, D156, D205, D207, D208
> Covers: Kyokai's grammar enforces the accepted non-goals instead of leaving forbidden mechanisms as unspecified parser gaps.
