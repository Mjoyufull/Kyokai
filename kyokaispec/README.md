# Kyokai Specification Workspace

This directory is the home of the normative Kyokai language and toolchain specification.


## Source-Of-Truth Order

1. This directory, once a Kyokai rule is actually written here.
2. `../kyokaidecided.md` for accepted Kyokai shape not yet spec-extracted.
3. `../Kyokaishape.md` for live public D-points and pending shape.
4. Linked public discussions, issues, and PRs.
5. `../phase.md` for implementation order only.

Until a section is rewritten as Kyokai spec text, inherited Austral prose in this directory is evidence and reference material only.

## Status

| Area | Status | Notes |
| --- | --- | --- |
| Kyokai spec structure | Not started | Needs directory/chapter plan before normative extraction. |
| Kyokai language spec text | Not started | Will be extracted from `../kyokaidecided.md`. |
| Kyokai toolchain spec text | Not started | Needed for CLI, packages, diagnostics, formatter, targets, and reproducibility. |
| Kyokai stdlib contract spec | Not started | Needed for D229 admission and stable APIs. |
| Inherited Austral spec material | Present as staging/reference | Not normative for Kyokai unless rewritten and accepted. |
| Traceability file | Staging | `SPEC_COMPILER_TRACE.md` currently records inherited Austral trace evidence and will be converted for Kyokai. |

## Building The Current Staging Document

The inherited spec build machinery is still present. To generate the current staging PDF/HTML:

```bash
make
```

To remove build output:

```bash
make clean
```

Generated outputs from this command are not a completed Kyokai spec until the source chapters have been rewritten.

## License

The inherited Austral spec material is under the GNU Free Documentation License. See `COPYING` for the current inherited license text.
