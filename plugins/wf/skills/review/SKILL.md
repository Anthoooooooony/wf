---
name: review
description: Stage 4 of the wf workflow. Independent quality review applying five-tier confidence scoring. If any issue scores >= 80 confidence, hard-resets STATE phase to `coding` and ends the session — the user must run /wf:code to continue. Only zero >=80 findings advance to `done`. Slash-only.
disable-model-invocation: true
---

# wf:review

Independent quality review of the implementation produced by `/wf:code`. This is stage 4 of 4 in the wf workflow.

The main agent performs the review directly — it does **not** dispatch the `code-reviewer` sub-agent (that one is reserved as the inner-loop reviewer inside `/wf:code`). The review here is the user-facing, formal review pass with hard-gate phase enforcement.

<HARD-GATE>
After producing the review report, count issues with confidence ≥ 80.

- If count > 0: STATE.md phase MUST be set back to `coding`, `review_round` MUST be incremented by 1, and this session MUST end. The user has to explicitly run `/wf:code` to continue addressing issues. Do NOT keep the session active "to also fix them right now" — that defeats the round counter and the audit trail.
- If count == 0: STATE.md phase MAY advance to `done`. Tell the user the feature is ready to commit / open a PR, and that `/wf:archive` is available as an optional cleanup step.
</HARD-GATE>

## Phase Validation

Before doing anything else:

1. Read `.wf/STATE.md`. If absent, refuse — tell the user to run `/wf:brainstorm` first.
2. If `phase` is `coding`, accept and update phase to `reviewing` for the duration of this run.
3. If `phase` is anything else (`brainstorm`, `planning`, `done`), refuse and suggest the matching `/wf:*` command.
4. Read `.wf/plans/PRD-<feature_id>.md` and all ADRs referenced by the PRD.

## Checklist

You MUST create a task for each item:

1. **Validate phase, load PRD and ADRs, set phase=reviewing**
2. **Identify review scope** — by default, all unstaged + uncommitted changes since `/wf:code` last started; the user may narrow this
3. **Read all changed files end-to-end**
4. **Apply confidence scoring** to each potential issue (see `references/confidence-scoring.md`)
5. **Group findings by severity** — Critical (≥ 95) and Important (≥ 80); discard < 80 from the report
6. **Write the review report** to `.wf/reviews/<feature_id>-<round>.md` (where `round` is the current `review_round + 1`)
7. **Apply the hard gate** — see HARD-GATE above
8. **Tell the user the next action** — either `/wf:code` to address issues, or commit / PR / `/wf:archive` if `done`

## Review Scope

Default: every file changed since the most recent coding-pass entry in `STATE.md`'s Phase Log. Concretely:

- `git diff` against the working tree
- Plus any untracked files mentioned in the PRD

The user may scope-down ("just review src/foo.ts" or "just the auth module"). Honor narrower scope, but record the narrowed scope in the review report's frontmatter.

## Confidence Scoring

Apply the five-tier scale from `references/confidence-scoring.md` to every potential issue. Only issues with confidence ≥ 80 are reported in the review document and counted against the hard gate. Lower-confidence findings can be mentioned in passing in chat but must NOT appear in the review document — that keeps the audit trail clean and avoids accidental phase regressions for noise.

## Project Guidelines Compliance

The primary review axis is project conventions in `CLAUDE.md` and equivalent. A violation of an explicit project guideline is automatically high confidence. Other axes (in priority order):

1. **Bugs / logic errors / security vulnerabilities / race conditions / null handling**
2. **CLAUDE.md guideline violations**
3. **PRD requirement gaps** — does the implementation actually satisfy each Functional Requirement?
4. **ADR adherence** — does the implementation follow the chosen architecture, or has it drifted?
5. **Code quality** — duplication, missing critical error handling, accessibility, test coverage gaps

## Review Report Format

Filename: `.wf/reviews/<feature_id>-<round>.md`

````markdown
---
feature_id: <feature_id>
round: <N>
phase_before: coding
created_at: <ISO 8601 with timezone>
threshold: 80
scope: <"all changed files" | narrowed scope description>
issues_above_threshold: <count>
verdict: pass | fail
---

# Review Round <N> — <feature title>

## Summary

One paragraph: what was reviewed, how many ≥ 80 issues, the verdict.

## Critical Issues (confidence ≥ 95)

For each:
- **Confidence:** <score>
- **File:** `<path>:<line>`
- **Problem:** <description>
- **Fix:** <concrete suggestion>

## Important Issues (confidence 80-94)

Same format.

## Notes

Optional: lower-confidence observations the reviewer wants to flag without forcing a regression. These do NOT count toward the gate.
````

## Hard Gate Enforcement

After writing the report:

```
issues_above_threshold = count of issues with confidence >= 80
```

### If issues_above_threshold > 0

1. Set `verdict: fail` in the report frontmatter.
2. Update `.wf/STATE.md`:
   - `phase: coding`
   - Increment `review_round` by 1
   - Append a phase-log entry: `<ISO timestamp> review round <N> failed (<count> issues at confidence >= 80) — phase reset to coding`
3. Tell the user:
   > "Review round <N> found <count> issues at confidence >= 80. STATE phase reset to `coding`. Run `/wf:code` to address them."
4. **End the session.** Do not offer to fix the issues immediately — that bypasses the round counter and undermines the audit trail.

### If issues_above_threshold == 0

1. Set `verdict: pass` in the report frontmatter.
2. Update `.wf/STATE.md`:
   - `phase: done`
   - Append a phase-log entry: `<ISO timestamp> review round <N> passed (0 issues at confidence >= 80) — phase advanced to done`
3. Tell the user:
   > "Review round <N> passed. The feature is ready. Suggested next steps: commit changes, open a PR, or run `/wf:archive` to collapse this feature's artifacts into `.wf/archive/<feature_id>/` and reset STATE for the next feature."

## Why This Skill Does Not Use code-reviewer

The `code-reviewer` sub-agent under `wf/agents/` is the **inner-loop** reviewer for `/wf:code`. Reusing it here would conflate two distinct review purposes:

- Inner loop (in `/wf:code`): catch obvious problems before declaring the implementation ready
- Outer loop (in `/wf:review`): user-facing formal review with hard-gate phase enforcement

Keeping these separate makes the gate honest. The main agent applies confidence scoring directly in this skill so the threshold logic and STATE manipulation live in one place.

## Key Principles

- **Confidence threshold is non-negotiable** — 80 is the line. Do not soften it for ergonomics.
- **One round = one report** — never edit a prior round's report; write a new one.
- **End the session on fail** — let the user re-enter `/wf:code` deliberately, not be auto-bridged into it.
- **Pass means done** — no extra ceremony at the `done` transition; the user decides what's next (commit, PR, `/wf:archive`).
