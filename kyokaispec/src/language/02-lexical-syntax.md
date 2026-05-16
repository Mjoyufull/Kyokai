# Lexical Syntax

[Rikona Kurasaki / Mjoyufull]
Kyokai source is written as UTF-8 text and is specified here with the same clean EBNF discipline inherited from Austral: `=` defines a production, `;` ends it, concatenation is written with `,`, alternation with `|`, optional syntax with `[...]`, and zero-or-more repetition with `{...}`. That inherited shape is still useful. The city changed, the street signs changed, but the map-reading tool still works.

> Trace: D5, D52, D78, D86
> Covers: Kyokai keeps useful inherited Austral specification machinery while replacing the source language surface with `.kyo` interface files and `.kai` body files under the Kyokai module/package model.

## Source Files And Text

[Rikona Kurasaki / Mjoyufull]
A Kyokai source file is either a `.kyo` interface file or a `.kai` body file. A module path such as `Foo.Bar` resolves, under the manifest-declared module root, to `Foo/Bar.kyo` for the interface and `Foo/Bar.kai` for the body. Austral's `.aui` and `.aum` extensions are not Kyokai source extensions.

> Trace: D52, D78
> Covers: Kyokai source uses `.kyo` for interfaces and `.kai` for bodies, and dotted module names map mechanically to directory paths under the package module root.

Source text is a sequence of Unicode scalar values encoded as UTF-8. Lexical grammar that names ASCII letters, ASCII digits, keywords, punctuation, and operators matches those exact Unicode code points. A conforming implementation must reject malformed UTF-8 before parsing.

> Trace: D54, D87
> Covers: Source text must be well-formed UTF-8, while Kyokai's language-level behavior remains specified rather than depending on backend or lexer undefined behavior.

Newlines are ordinary whitespace except where splitting a token would make that token illegal. Empty executable blocks are legal no-op blocks. Trailing commas are legal in every comma-delimited list form in the grammar. A formatter may choose canonical layout, but it must not insert or remove executable behavior.

> Trace: D180
> Covers: Kyokai newlines are insignificant whitespace, trailing commas are allowed in comma-delimited lists, empty executable blocks are legal no-ops, and formatting is not a semantic rewrite layer.

## Comments And Documentation Comments

[Rikona Kurasaki / Mjoyufull]
Kyokai comments use C-family line syntax. A normal comment begins with `//` and continues to the next line break or end of file. A declaration documentation comment begins with `///`. A module or file documentation comment begins with `//!`. Block comments do not exist.

```ebnf
line comment = "//", {not line break}, line end;
declaration doc comment = "///", {not line break}, line end;
module doc comment = "//!", {not line break}, line end;
```

> Trace: D63
> Covers: Kyokai uses `//`, `///`, and `//!` comments, replaces Austral's `--` and triple-quote docstring surface, and has no block comments.

Documentation comments attach to the next declaration or file/module item according to the documentation chapter. They are comments first: they do not create string literals, declarations, hidden attributes, or executable statements.

> Trace: D63, D29
> Covers: Documentation comments are lexical comment forms with specified attachment behavior, not runtime strings or hidden syntax forms.

## Identifiers And Module Names

[Rikona Kurasaki / Mjoyufull]
Identifiers stay narrow on purpose. A language about boundaries should not begin by making the spelling of a name depend on invisible normalization. Kyokai identifiers are ASCII and case-sensitive.

```ebnf
identifier = letter, {letter | digit | "_"};
module identifier = uppercase, {letter | digit | "_"};
module name = module identifier, {".", module identifier};
letter = uppercase | lowercase;
uppercase = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
          | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
          | "U" | "V" | "W" | "X" | "Y" | "Z";
lowercase = "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
          | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
          | "u" | "v" | "w" | "x" | "y" | "z";
digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9";
```

> Trace: D11a, D78, D179, D214
> Covers: Kyokai keeps Austral's `camelCase` functions and values plus `PascalCase` modules and types, with deterministic ASCII module paths and import names.

Functions, variables, parameters, fields, and local bindings use `camelCase`. Types, type constructors, module segments, typeclasses, capability types, and union constructors use `PascalCase`. The compiler may warn on convention violations where the category is statically known, but name resolution is case-sensitive and does not normalize spellings.

> Trace: D11a, D11b
> Covers: Kyokai keeps the Austral naming convention while adding ownership-signaling naming patterns as compiler-warned convention for APIs.

No still-live binding may be shadowed by another local binding, parameter, pattern binding, import, or declaration in the same lookup scope. Imports may not introduce names that collide after explicit `as` renaming. Built-in language names cannot be shadowed by imports.

> Trace: D60, D214
> Covers: Kyokai rejects variable shadowing and import-order-dependent name meaning, and built-in names remain protected from imported collisions.

## Keywords, Contextual Words, And Built-Ins

[Rikona Kurasaki / Mjoyufull]
The following words are reserved as language keywords and cannot be used as ordinary identifiers:

```text
Ok Err Some None and as audit band bnot body borrow bor break build bxor capability
case comptime constant continue debug defer do else ensure errdefer esac extern
false fi for foreign function generator if import in instance internal is join
let mon module nil not od of or packed panic pick pragma record require return
rotl rotr seal select shl shr spawn spec static static_assert taskgroup then
todo true type alias typeclass union unreachable unsafe var when where while
with yield
```

> Trace: D8-D21, D24, D41, D52, D54, D63, D78, D111, D118, D120, D127, D179, D198, D214, D235, D252
> Covers: Kyokai reserves the accepted syntax surface for declarations, control flow, contracts, FFI, unsafe contracts, tasks, generators, compile-time forms, and semantic terminators.

Some words are contextual. `result` is recognized as the postcondition result view only inside `ensure` clauses. `old` is recognized only inside `ensure` clauses. `ignore` is recognized as the discard pattern only in pattern position. Outside those contexts, they are ordinary identifiers unless another chapter gives a narrower rule.

> Trace: D125, D129, D205, D206
> Covers: `result`, `old`, and `ignore` have context-limited syntax roles rather than becoming global reserved words.

The built-in names `Unit`, `Bool`, `Never`, `Int8`, `Int16`, `Int32`, `Int64`, `Nat8`, `Nat16`, `Nat32`, `Nat64`, `Float32`, `Float64`, `Index`, `String`, `StaticString`, `Result`, `Optional`, `Ok`, `Err`, `Some`, `None`, `target`, `Os`, `Arch`, `Abi`, and `Endian` are language-level names, not an imported prelude. They are available without an import and cannot be shadowed by imports.

> Trace: D24, D120, D191, D194, D214
> Covers: Kyokai has language-level built-ins for primitive types, result/optional forms, target descriptors, and compile-time static strings, without relying on a hidden prelude expansion.

## Numeric Literals

[Rikona Kurasaki / Mjoyufull]
A numeric literal has no sign token. `-42` is parsed as unary `-` applied to the literal `42`. This matters because overflow, suffix typing, and literal representability are checked on the literal's own token and then on the surrounding expression.

> Trace: D12, D75, D261
> Covers: Signs are operators, numeric literal typing is explicit or context-driven, and overflow remains part of the language semantics rather than lexer folklore.

Integer literals are decimal, hexadecimal, binary, or octal. Non-decimal integer literals use modern `0x`, `0b`, and `0o` prefixes. Austral's `#x`, `#b`, and `#o` prefixes are not Kyokai syntax.

```ebnf
decimal integer = digit, {digit separator or digit}, [integer suffix];
hex integer = "0x", hex digit, {digit separator or hex digit}, [integer suffix];
binary integer = "0b", bin digit, {digit separator or bin digit}, [integer suffix];
octal integer = "0o", oct digit, {digit separator or oct digit}, [integer suffix];
integer literal = decimal integer | hex integer | binary integer | octal integer;
hex digit = digit | "a" | "b" | "c" | "d" | "e" | "f"
                  | "A" | "B" | "C" | "D" | "E" | "F";
bin digit = "0" | "1";
oct digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7";
```

> Trace: D135, D261
> Covers: Kyokai numeric literals use `_` separators and `0x`/`0b`/`0o` base prefixes, replacing Austral's apostrophe separators and `#x`/`#b`/`#o` prefixes.

`_` is a digit separator only when it appears between two digits in the same digit run. It may not appear at the start or end of a literal, immediately after a base prefix, adjacent to a decimal point, adjacent to an exponent marker, or inside a suffix. Apostrophe separators are not Kyokai syntax.

> Trace: D135, D261
> Covers: Kyokai allows `_` separators only in tightly defined digit positions and rejects apostrophe numeric separators.

Integer suffixes are exactly `i8`, `i16`, `i32`, `i64`, `n8`, `n16`, `n32`, `n64`, and `index`, mapping to `Int8`, `Int16`, `Int32`, `Int64`, `Nat8`, `Nat16`, `Nat32`, `Nat64`, and `Index`. Floating suffixes are exactly `f32` and `f64`, mapping to `Float32` and `Float64`. A suffixed literal has the suffixed type before contextual literal inference. Context may confirm that type, but it may not silently retarget the literal.

> Trace: D12, D261
> Covers: Kyokai has a closed numeric literal suffix set and no C-style suffix composition, promotions, or signedness folklore.

Floating-point literals are decimal. They may contain `_` separators under the same digit-run rules as integer literals and may use an exponent. Hexadecimal floating-point literals are not part of this phase's accepted surface.

```ebnf
float literal = digit run, ".", [digit run], [exponent], [float suffix]
              | digit run, exponent, [float suffix];
exponent = ("e" | "E"), ["+" | "-"], digit run;
digit run = digit, {digit | "_"};
integer suffix = "i8" | "i16" | "i32" | "i64" | "n8" | "n16" | "n32" | "n64" | "index";
float suffix = "f32" | "f64";
```

> Trace: D12, D135, D261
> Covers: Floating literals use decimal syntax, optional exponents, `_` digit separators, and the closed `f32`/`f64` suffix set.

## Text, Code-Point, Byte, And Static-String Literals

[Rikona Kurasaki / Mjoyufull]
Kyokai does not make one C-style character token carry three different histories in its mouth. Text, raw text, code points, and bytes are separate literal families.

```text
"..."        ordinary escaped String
"""..."""  raw multi-line String
'A'          Nat32 Unicode scalar value
b'A'         Nat8 byte
static "..." StaticString bridge expression
```

> Trace: D54, D120, D165
> Covers: Kyokai has distinct ordinary string, raw string, code-point, byte, and explicit `static "..."` compile-time text forms.

An ordinary string literal produces a `String` and processes escapes. A raw multi-line string literal produces a `String`, processes no escapes, and preserves the content between its delimiters exactly. A code-point literal produces `Nat32` and must denote exactly one Unicode scalar value after escape processing; surrogate code points are illegal. A byte literal produces `Nat8` and must denote exactly one byte after escape processing; non-ASCII byte values require escapes such as `b'\xFF'`.

> Trace: D54, D30a, D165
> Covers: `String` is UTF-8 text, byte values remain separate, code-point literals are Unicode scalar values, and Kyokai has no platform-dependent `char` literal family.

The standard escape family is `\\`, `\"`, `\'`, `\n`, `\r`, `\t`, `\0`, `\xNN`, and `\u{HEX...}`. `\xNN` names exactly two hexadecimal digits. `\u{HEX...}` names one Unicode scalar value. An escape that would produce an invalid code point, invalid byte, or invalid UTF-8 contribution for the target literal family is rejected.

> Trace: D54, D87
> Covers: Escape processing is closed and checked, with invalid literal contents rejected rather than left implementation-defined.

`static "..."` is not a separate token. It is a grammar form that applies the `static` keyword to an ordinary escaped string literal and produces a `StaticString`. Ordinary string literals remain runtime `String` in all contexts; Kyokai does not contextually reinterpret them as `StaticString`.

> Trace: D120
> Covers: Compile-time text uses explicit `static "..."` and does not retarget ordinary string literals through context.

## Operators, Delimiters, And Terminators

[Rikona Kurasaki / Mjoyufull]
Kyokai punctuation is small enough that the reader should be able to hold it in their head without guessing which symbol secretly means authority, movement, or control flow.

```text
( ) [ ] { }
, . .. : ; :=
= != < <= > >=
-> =>
+ - * / % ++
& &! &~ ~
```

> Trace: D7a, D7b, D10, D14, D15, D15a, D34, D35, D36, D56, D65, D106, D119
> Covers: Kyokai reserves punctuation for calls, indexing, construction, assignment, equality, explicit references, dereference, UFCS/member access, slicing, function types, and `or return` error mapping.

Keyword operators are `and`, `or`, `not`, `band`, `bor`, `bxor`, `bnot`, `shl`, `shr`, `rotl`, and `rotr`. `and` and `or` are short-circuiting boolean operators. Bitwise and shift operators are keywords so they do not collide with borrow syntax.

> Trace: D41, D56, D57
> Covers: Boolean operators short-circuit, and Kyokai uses keyword bitwise, shift, and rotate operators with limited precedence rules.

Statement and boundary terminators are part of the syntax, not decoration: `fi;`, `esac;`, `od;`, `qed;`, `build;`, `spec;`, `drop;`, `seal;`, `mon;`, `join;`, `pick;`, and `audit;`. The terminator says what kind of boundary just closed.

> Trace: D9, D111, D127, D245, D252
> Covers: Kyokai uses reversed control-flow terminators and semantic boundary terminators for functions, types, typeclasses, borrows, modules, FFI blocks, task groups, selects, and unsafe audits.

The token `=>` is reserved for the explicit error mapping arm in `or return name => expr`. It is not a match arrow, not a union-construction operator, and not a general lambda arrow. One-expression closure literals use the whole form `fn [captures] (params): Ret => expr`, where `=>` belongs to that closure production.

> Trace: D65, D118, D119
> Covers: `=>` is not Austral union construction; Kyokai uses it only in accepted explicit mapping and closure-expression grammar forms.

## Rejected Lexical Forms

[Rikona Kurasaki / Mjoyufull]
The lexer rejects inherited or neighboring-language spellings that would create a second surface for the same meaning: Austral `--` comments, triple-quote docstrings as documentation syntax, block comments, apostrophe digit separators, `#x`/`#b`/`#o` integer prefixes, C-style compound numeric suffixes, platform-shaped `char` literals, `_` wildcard patterns, `->` field access, `/=` not-equal, `|>` pipelines, and wildcard imports.

> Trace: D10, D54, D63, D65, D108, D135, D179, D205, D261
> Covers: Kyokai rejects obsolete Austral spellings and extra neighboring-language surfaces when accepted Kyokai syntax already gives one explicit form.
