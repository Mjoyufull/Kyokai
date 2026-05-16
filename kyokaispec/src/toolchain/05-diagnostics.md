# Diagnostics

[Rikona Kurasaki / Mjoyufull]
Diagnostics are part of the language experience, but they are also part of the toolchain contract. A compiler that knows the rule but cannot say what went wrong has left the programmer outside in the rain with a locked door and a key that almost fits.

> Trace: D29
> Covers: Kyokai diagnostics are a normative toolchain surface, not a polish-only feature.

## Diagnostic Records

Every diagnostic has a stable code, severity, primary message, optional source spans, optional labels, optional notes, optional help text, optional machine-applicable suggestions, and optional related diagnostics. Human rendering may change wording across releases, but the code, severity category, span meaning, and JSON shape must remain compatible within the same diagnostic schema version.

> Trace: D29, D157
> Covers: Diagnostics have stable structured identity while allowing human wording improvements.

Diagnostic codes are spelled `KYO-E0001` for errors, `KYO-W0001` for warnings, `KYO-L0001` for lints, `KYO-A0001` for audit findings, and `KYO-I0001` for informational notes that may appear as top-level records in machine output. A code is never reused for a different rule after release.

> Trace: D29, D150
> Covers: Diagnostic code namespaces are stable and rule-specific.

Severity is one of `error`, `warning`, `lint`, `audit`, or `info`. Errors prevent the command from succeeding. Warnings do not prevent success unless promoted by policy. Lints are style or maintainability diagnostics. Audit findings are risk/policy diagnostics from `kyokai audit` or audit-enabled commands. Info diagnostics never change command success.

> Trace: D29, D150
> Covers: Severity has fixed command-success behavior.

## Source Spans

A source span identifies a file, byte range, line range, column range, and source origin. The origin is one of `source`, `generated`, `embedded`, `koi`, `manifest`, `lockfile`, `target-spec`, or `command-line`. Byte ranges are over UTF-8 bytes in the exact input file after line-ending normalization rules from the lexical spec.

> Trace: D29, D52, D79, D83
> Covers: Diagnostics can point at source, generated files, artifacts, manifests, lockfiles, target specs, and flags.

A diagnostic with multiple relevant sites must mark exactly one primary span unless the error is genuinely source-less. Other spans are related spans. For example, an import collision has the import site as primary and the conflicting exported declarations as related spans.

> Trace: D29, D78, D214
> Covers: Multi-site errors remain navigable and deterministic.

If a diagnostic arises from a `.koi` artifact, the tool must report the artifact path and package identity. If the artifact records original source spans and the source is available, the diagnostic may also include original source locations as related spans. Missing source must not make the diagnostic lie about which artifact failed.

> Trace: D29, D79
> Covers: Artifact diagnostics name artifacts honestly and use source spans only when available.

## JSON Schema

`--format json` emits one UTF-8 JSON object per line unless a command-specific chapter defines a single JSON document mode. Each object has at least these keys:

```json
{
  "schema": "kyokai-diagnostic-v1",
  "code": "KYO-E0001",
  "severity": "error",
  "message": "short human message",
  "primary_span": null,
  "spans": [],
  "notes": [],
  "suggestions": [],
  "command": "check",
  "package": null
}
```

> Trace: D29, D225
> Covers: Machine diagnostics use a stable line-delimited JSON record by default.

Unknown keys in a JSON diagnostic must be ignored by consumers. Removing a key, changing a key's meaning, changing a severity string, or reusing a code for a different rule requires a new schema version.

> Trace: D29, D157
> Covers: Diagnostic JSON evolves compatibly by schema version.

A suggestion has a message, applicability, and edits. Applicability is `machine-applicable`, `maybe-incorrect`, or `manual-only`. A machine-applicable suggestion must preserve parseability and must not change semantics except for the exact error repair described by the diagnostic.

> Trace: D25, D29
> Covers: Suggestions distinguish safe automatic edits from human judgment.

## Diagnostic Explanation Catalog

Every released diagnostic code has an explanation catalog entry shipped with the toolchain. The entry includes the code, severity, rule name, short explanation, longer explanation, common causes, examples where useful, repair patterns, suggestion applicability if any, related diagnostic codes, and links or local anchors to relevant spec chapters. The catalog is versioned with the diagnostic schema.

> Trace: D29, D267
> Covers: Diagnostic codes have first-party explanations that can be rendered offline and tied back to the spec.

`kyokai explain <code-or-category>` reads the local explanation catalog by default. It may print a human explanation or JSON when `--format json` is selected. If the installed toolchain does not know the requested code, the command must say so without searching the network by default. Online documentation may mirror the same catalog, but the local toolchain remains the authority for the codes it emits.

> Trace: D29, D225, D267
> Covers: Explanation lookup works offline, matches the installed compiler version, and keeps online docs as mirrors rather than hidden authority.

## Automatic Fixes

A diagnostic may carry edits only when the compiler can state the exact repair being offered. `machine-applicable` means the edit is safe for `kyokai fix` under the current source snapshot. `maybe-incorrect` means the compiler can suggest a plausible edit but cannot prove user intent. `manual-only` means the diagnostic can explain a repair but must not emit an automatic edit.

> Trace: D25, D29, D267
> Covers: Automatic fixes are gated by diagnostic applicability rather than by a separate refactoring folklore layer.

`kyokai fix` applies only selected machine-applicable suggestions by default. It must reject stale suggestions whose source spans no longer match the checked snapshot, reject overlapping edits unless the diagnostic engine has already merged them into one edit set, rerun parsing after edits, and format only the changed files through `kyokai fmt` rules. If validation fails, the command must leave the original files unchanged or restore them before reporting failure.

> Trace: D25, D29, D83, D267
> Covers: Safe fixes are snapshot-checked, overlap-checked, parse-checked, and formatter-integrated before file changes become visible.

## Warning And Lint Policy

Warnings are grouped by category. Required categories include `unused`, `style`, `compatibility`, `deprecated`, `unsafe-surface`, `capability-surface`, `ffi-surface`, `reproducibility`, and `toolchain`. The compiler may add categories only by documenting them in this spec or a compatible extension registry.

> Trace: D29, D150, D155
> Covers: Warnings and lints are categorized and not free-floating messages.

Project-level diagnostic policy lives in `kyokai.toml`. Source-level suppression is allowed only through explicit attributes or pragmas defined by the language spec. A suppression must name a diagnostic code or category and may include a reason string. Blanket suppression of all errors is illegal.

> Trace: D29, D155
> Covers: Diagnostic suppression is explicit, bounded, and auditable.

A suppression that matches no emitted diagnostic should itself produce a warning unless the policy marks unused suppressions as allowed. This prevents dead suppressions from becoming old dust nobody remembers to clean.

> Trace: D29
> Covers: Suppression policy catches stale suppressions.

## Determinism

Given the same inputs, target, profile, and diagnostic policy, diagnostics must be emitted in a deterministic order. The order is by compiler phase, package dependency order, logical module name, source span, and code, unless a chapter gives a more specific order for a command.

> Trace: D29, D83
> Covers: Diagnostic ordering is reproducible.

Human rendering may use color, underlines, source excerpts, and terminal width. Those presentation choices must not change diagnostic JSON, command success, lockfiles, artifacts, or cache keys.

> Trace: D29, D83
> Covers: Human diagnostic rendering is presentation-only.

## Required Quality

A diagnostic should name the rule violated, the entity involved, the source location, and the nearest actionable correction when the compiler can know it. For ownership, borrow, capability, target, package, and `.koi` errors, the diagnostic must include enough related context that a user can find the other side of the conflict without reading compiler internals.

> Trace: D29, D78-D79, D137, D150
> Covers: Diagnostics for core Kyokai safety boundaries carry actionable context.

A diagnostic must not blame a later phase when an earlier rule is the true cause. A missing symbol caused by a hidden import collision should report the import collision. A link failure caused by undeclared native dependency should report the undeclared dependency before invoking the linker when the tool can prove it.

> Trace: D29, D31, D78, D150
> Covers: Diagnostics point at the first meaningful rule violation.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
A good diagnostic does not flatter you. It shows the wound, names the blade, and points to the hand that has to move. Kyokai needs that because a language with linear values, contracts, unsafe walls, and capabilities can become a maze if the compiler only says no.

> Trace: D29
> Covers: Kyokai diagnostic quality is required because the language's safety model must be teachable through errors.
