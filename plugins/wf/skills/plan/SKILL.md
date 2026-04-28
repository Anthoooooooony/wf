---
name: plan
description: Stage 2 of the wf workflow. Translate an approved brainstorm spec into a PRD and ADRs. Produces .wf/plans/PRD-<feature_id>.md and .wf/adr/ADR-NNN-*.md. Slash-only — does not auto-commit and does not auto-invoke /wf:code.
disable-model-invocation: true
---

# wf:plan

Translate the approved brainstorm spec into an executable plan: a Product Requirements Document (PRD) and one or more Architecture Decision Records (ADRs). This is stage 2 of 4 in the wf workflow.

<HARD-GATE>
Do NOT write any production code, modify any source files, or invoke /wf:code until the user has explicitly approved the PRD and the chosen architecture approach. The output of /wf:plan is documentation, not implementation.
</HARD-GATE>

## Phase Validation

Before doing anything else:

1. Read `.wf/STATE.md`. If absent, refuse — tell the user to run `/wf:brainstorm` first.
2. If `phase` is not `planning`, refuse and suggest the matching `/wf:*` command.
3. Read the corresponding `.wf/brainstorm/<feature_id>.md`. If absent, refuse — STATE is corrupt.

## Checklist

You MUST create a task for each item:

1. **Validate phase & load brainstorm spec**
2. **Survey existing codebase** (only if modifying an existing project) — see Codebase Survey below
3. **Surface architectural friction** (only if modifying) — apply the deletion test, depth audit, and locality check from `references/architecture-glossary.md`
4. **Propose 2-3 architecture-level approaches** — different topology / data flow / layering choices, with trade-offs. Product/UX alternatives belong in `/wf:brainstorm`, not here.
5. **User selects one approach**
6. **Draft PRD** to `.wf/plans/PRD-<feature_id>.md` using the template in `references/prd-adr-templates.md`
7. **Draft ADRs** to `.wf/adr/ADR-NNN-<slug>.md` for each load-bearing decision (one ADR per decision; see "When to write an ADR" below)
8. **Spec self-review** — same four-axis check as brainstorm (placeholder / consistency / scope / ambiguity)
9. **User reviews PRD and ADRs**
10. **Hand off to /wf:code** — set `phase: coding` in STATE.md and tell the user to run `/wf:code` when ready

## Codebase Survey (existing projects only)

If the project already has source code (i.e., this isn't greenfield), survey it before drafting the PRD:

1. Read `CLAUDE.md` if present — its conventions trump anything you'd otherwise infer.
2. Use `Glob` / `Grep` / `Read` directly for narrow lookups.
3. If the survey requires deep exploration across many files, dispatch a generic `Explore` subagent via the Agent tool. **Do not** dispatch `code-explorer` / `code-architect` from `wf/agents/` — those are reserved for the `wf:code` phase to keep their roles unambiguous.
4. Apply the architectural glossary in `references/architecture-glossary.md`:
   - **Deletion test** on candidate modules — if deleting them concentrates complexity, they earn their keep
   - **Depth audit** — find shallow modules that obscure rather than abstract
   - **Locality check** — where does a single conceptual change require edits in many places?
5. Note conflicts with existing ADRs in `.wf/adr/` or `docs/adr/`. Surface these in the PRD's "Architecture Notes" section.

## Architecture Approaches

Always propose 2-3 architecture-level alternatives. Each approach should describe:

- **Topology** — which modules are added / modified / removed; the resulting interface graph
- **Trade-offs** — what each approach is good and bad at, in concrete terms (test surface, change locality, performance, complexity)
- **Cost** — rough estimate of files touched and depth of refactor
- **Recommendation** — your pick with reasoning

Lead with your recommendation. Wait for user confirmation before drafting the PRD.

## When to Write an ADR

Write an ADR (`.wf/adr/ADR-NNN-<slug>.md`) for any decision that is:

- **Load-bearing** — the rest of the implementation depends on it
- **Reversible only at high cost** — once chosen, switching costs significant work
- **Likely to be re-litigated** — future contributors (or future you) will ask "why this and not X?"

Skip ADRs for ephemeral choices ("not worth it right now") or self-evident ones. Borrow this measure from upstream `improve-codebase-architecture`:

> "Want to record this as an ADR so future architecture reviews don't re-suggest it?"

If the user rejects an architecture candidate with a load-bearing reason, offer an ADR for the rejection itself — that is exactly the kind of decision future explorers need to know about.

## ADR Numbering

ADR numbers (`NNN`) are sequential across the project. Read `.wf/adr/` (and `docs/adr/` if it exists) and pick the next available number, padded to three digits. If no ADRs exist yet, start at `001`.

## After Drafting

**Spec Self-Review** (apply to both PRD and each ADR):

1. **Placeholder scan** — any TBD, TODO, vague requirement? Fix.
2. **Internal consistency** — do sections contradict each other? Does the PRD match the chosen ADRs?
3. **Scope check** — is this still focused on a single feature, or has it sprawled?
4. **Ambiguity check** — could a sentence be interpreted two different ways? Pick one and make it explicit.

Fix issues inline, then ask the user to review.

**Handoff:**

After user approval:

1. Update `.wf/STATE.md`: set `phase: coding`, append a phase-log entry with the current ISO 8601 timestamp.
2. Tell the user to run `/wf:code` when ready.

Do NOT auto-invoke `/wf:code` or any other skill.

## Key Principles

- **Architecture-level alternatives only** — product/UX choices were settled in `/wf:brainstorm`
- **Deletion test before depth claims** — don't call something "deep" without imagining its absence
- **Use the project's domain glossary vocabulary** — names from `CLAUDE.md` or `CONTEXT.md` trump generic ones
- **One ADR per decision** — bundling makes them harder to find later
- **Reject = ADR opportunity** — if the user rejects a path with a load-bearing reason, capture it
