# Formatter

[Rikona Kurasaki / Mjoyufull]
The formatter is where style stops being a debate in every doorway. Kyokai has one official format, because source code is hard enough when ownership matters without also carrying a private weather map of indentation habits.

> Trace: D25
> Covers: Kyokai has an official formatter with one deterministic style.

## Contract

`kyokai fmt` formats `.kyo` and `.kai` source files. It is deterministic, idempotent, zero-configuration, and parse-preserving. Running it twice on the same source must produce byte-identical output the second time. Formatting must not change program semantics, module identity, comments' attachment to declarations, documentation comments' meaning, or diagnostic suppression scope.

> Trace: D25, D29, D52, D83
> Covers: Formatting is stable, semantic-preserving, and safe to automate.

The formatter must parse the source according to the package edition when a manifest context is available. If no manifest context is available, it may use the current default edition only for standalone file formatting, and it must report which edition was used in verbose or JSON mode.

> Trace: D25, D105
> Covers: Formatting is edition-aware and reports standalone assumptions.

A file with parse errors is not formatted by default. The formatter may emit diagnostics and leave the file unchanged. A future recovery mode may format valid subtrees only if it is explicitly selected and clearly reports that the output is not a full canonical format.

> Trace: D25, D29
> Covers: The formatter does not guess through invalid syntax by default.

## Style Rules

The official style uses 4-space indentation, spaces instead of tabs for indentation, LF line endings, UTF-8 source, and a 100-column preferred line width. The 100-column rule is a formatter target, not a parser rule; long literals, long URLs in comments, and unbreakable identifiers may exceed it.

> Trace: D25, D63
> Covers: Core formatting width, indentation, encoding, and line-ending policy are fixed.

The formatter owns whitespace around operators, delimiters, declarations, blocks, parameter lists, type arguments, `where` clauses, contract clauses, and terminators. It preserves blank lines only where they separate meaningful groups; multiple blank lines are collapsed according to the official style.

> Trace: D25
> Covers: Whitespace layout is formatter-owned.

The formatter must preserve ordinary comments and documentation comments. It may move a comment only when the comment remains attached to the same syntax node or trivia position by the formatter's documented attachment rules. It must not reflow code examples inside documentation comments unless a future doc-format mode explicitly opts in.

> Trace: D25, D218
> Covers: Formatting preserves comment and documentation meaning.

The formatter must not sort imports unless an import-sorting mode is specified by this chapter. Phase 12 does not define import sorting. Preserving import order avoids surprising comments, grouping, and review diffs while the language already treats import-order conflict resolution as illegal.

> Trace: D25, D78, D214
> Covers: Import order is preserved because conflict semantics do not depend on order.

## Modes

| Mode | Behavior | Trace |
| --- | --- | --- |
| `kyokai fmt` | Format selected package or workspace files in place. | D25, D78 |
| `kyokai fmt --check` | Report files that would change and exit nonzero when any change is needed. | D25, D225 |
| `kyokai fmt --stdin --filename <path>` | Read one file from stdin, infer context from filename when possible, write formatted source to stdout. | D25, D105 |
| `kyokai fmt --diff` | Print a deterministic textual diff instead of editing files. | D25, D83 |
| `kyokai fmt --format json` | Emit machine-readable file status and diagnostics. | D25, D29 |

> Trace: D25, D29, D78, D83, D105, D225
> Covers: Formatter modes are specified for local use, CI, editor integration, and scripts.

`--check` must not write files. `--diff` must not write files. `--stdin` must not read or write project files except for manifest discovery if a filename is provided and the command needs edition context.

> Trace: D25, D83
> Covers: Formatter modes have clear filesystem effects.

## File Selection

In package scope, `kyokai fmt` selects all `.kyo` and `.kai` files under the package module root and any explicitly declared generated-source outputs currently present when the generation chapter marks them formatter-owned. In workspace scope, it selects member packages in deterministic package-name order.

> Trace: D25, D78, D83
> Covers: Formatter file selection follows package/workspace/module roots deterministically.

Files outside module roots are not formatted unless passed explicitly by path. Explicit paths must still use a recognized Kyokai source extension unless a future chapter admits additional source-like files.

> Trace: D25, D52, D78
> Covers: Formatter discovery does not wander through unrelated repository files.

## No Semantic Rewrites

The formatter must not insert missing imports, rename bindings, expand implicit completions, lower sugar, reorder declarations, choose pattern forms, simplify expressions, change numeric literal bases, change string literal families, or apply compiler suggestions. Those are compiler or refactoring actions, not formatting.

> Trace: D25, D29, D87, D238
> Covers: Formatting is not semantic rewriting or refactoring.

If a formatting choice would obscure a language boundary, the boundary wins. Terminators such as `qed;`, `build;`, `fi;`, `od;`, `esac;`, `seal;`, `audit;`, `mon;`, and `drop;` remain visually attached to the construct they close.

> Trace: D9, D25
> Covers: Formatter style preserves Kyokai's visible boundary syntax.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
A formatter should not be clever in the places where the language is already doing heavy work. Make the code clean. Leave the meaning alone. Let review talk about ownership, contracts, and names instead of arguing over how far a `where` clause leaned to the right.

> Trace: D25
> Covers: Kyokai formatting removes style noise without touching semantics.
