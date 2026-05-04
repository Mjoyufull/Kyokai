# Kyokai Shape And Public D-Points

This file tracks public Kyokai design shape while the full spec is being written in `kyokaispec/`.

Use this file for:

- new public D-points
- links to GitHub Discussions, issues, or PRs that contain design proposals
- accepted shape that has not yet been moved into `kyokaispec/`
- short notes about public design direction that are too live for `kyokaidecided.md`

Keep this file public, concise, and traceable.

## Source-Of-Truth Order

1. `kyokaispec/` once a rule is written there.
2. `kyokaidecided.md` for public accepted shape not yet fully spec-extracted.
3. `Kyokaishape.md` for live public D-points and pending shape.
4. Public GitHub Discussions/issues/PRs linked from this file.
5. `phase.md` for implementation order only.

## Public D-Point Flow

1. Open a proposal in `Kyokaishape.md`, GitHub Discussions, an issue, or a PR labeled as a D-point.
2. Debate the shape publicly.
3. Write the final proposed rule text.
4. Gather at least 3 community acks on the final shape.
5. Maintainer marks the point decided or sends it back for wording.
6. Move the decided shape into `kyokaidecided.md` and then `kyokaispec/` when the spec section exists.

The acks happen after final wording, not before. Early approval of the general direction is useful, but it does not close the point.

## Status Words

- `PROPOSED`: opened but not shaped enough to decide.
- `SHAPE_DEBATING`: real options are being debated.
- `FINAL_TEXT_PROPOSED`: final wording exists and can be acked.
- `ACKED`: final wording reached 3 acks.
- `DECIDED`: maintainer accepted the final shape.
- `SPEC_EXTRACTED`: normative text exists in `kyokaispec/`.
- `CONFORMANCE_BACKED`: executable tests exist.
- `IMPLEMENTED`: compiler/toolchain/stdlib implementation exists.

## D-Point Template

````markdown
### D300: Short Name **[PROPOSED | NAV: pending kyokaispec section]**

**The question**: What exactly are we deciding?

**Use case**: What real Kyokai code, tooling, stdlib work, or spec guarantee needs this?

**Current state**: What is currently decided, implemented, inherited from Austral, or missing?

**Prior art**:

| System | Shape | Notes |
| --- | --- | --- |
| Austral | ... | ... |
| Rust | ... | ... |
| Zig | ... | ... |
| C | ... | ... |

**Options**:

| Option | Shape | Pros | Cons |
| --- | --- | --- | --- |
| A | ... | ... | ... |
| B | ... | ... | ... |

**Recommendation**: Which option should Kyokai use, and why?

**Proposed shape**:

```text
Write the actual rule here. It should be close enough to become spec text.
```

**Consequences**:

- What this makes simpler.
- What this makes harder.
- Which existing decisions/spec sections it touches.

**Ack state**:

- Final wording posted: no
- Acks: 0/3
- Decided: no
````

## Decided Entry Template

````markdown
### D300: Short Name **[DECIDED | SPEC_EXTRACTED | NAV: kyokaispec/src/path.md]**

**Naved to spec**: `kyokaispec/src/path.md`.

**The question**: What was decided?

**Use case**: Why does real Kyokai need this?

**Justification**: Why this is the Kyokai shape.
````

## Active D-Points

No active public D-points yet.

## Decided But Not Yet Spec-Extracted

The initial decided shape is being cleaned into `kyokaidecided.md` first. As the public spec gets written, decisions will move from there into `kyokaispec/` and be linked here only when useful.
