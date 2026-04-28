---
name: code
description: Stage 3 of the wf workflow. Implement the feature according to the approved PRD and ADRs. Dispatches code-explorer / code-architect / code-reviewer in parallel for codebase understanding, implementation blueprint, and inner-loop self-review. Slash-only.
disable-model-invocation: true
---

# wf:code

Implement the feature according to the approved PRD and ADRs from `/wf:plan`. This is stage 3 of 4 in the wf workflow.

The high-level architecture decisions are already settled — this skill turns them into running code with the help of three specialized sub-agents.

<HARD-GATE>
Do NOT advance the STATE phase to `done` from this skill. Phase advancement happens in `/wf:review` only, and only when the reviewer reports zero issues at confidence ≥ 80. This skill leaves `phase: coding` in place at every exit.
</HARD-GATE>

## Phase Validation

Before doing anything else:

1. Read `.wf/STATE.md`. If absent, refuse — tell the user to run `/wf:brainstorm` first.
2. If `phase` is not `coding`, refuse and suggest the matching `/wf:*` command.
3. Read `.wf/plans/PRD-<feature_id>.md`. Refuse if missing — STATE is corrupt.
4. Read all ADRs in `.wf/adr/` referenced by the PRD.

## Checklist

You MUST create a task for each item:

1. **Validate phase, load PRD and ADRs**
2. **Codebase exploration** — dispatch 2-3 `code-explorer` sub-agents in parallel (see Sub-Agent Dispatch below)
3. **Read files identified by explorers** — build the deep context the explorers' summaries don't fully convey
4. **Implementation blueprint** — dispatch a single `code-architect` sub-agent to produce an actionable file-level blueprint refining the ADR-level decisions into concrete files, functions, and build sequence
5. **User reviews blueprint** — confirm before writing any production code
6. **Implement** — write the code, following CLAUDE.md conventions strictly, adhering to the architecture blueprint
7. **Inner-loop self-review** — dispatch 3 `code-reviewer` sub-agents in parallel with different focuses (see below)
8. **Address inner-loop findings** — fix any confidence ≥ 80 issues before exiting; lower-confidence issues may be deferred
9. **Hand off to /wf:review** — leave `phase: coding`, tell the user to run `/wf:review` for the formal review pass

## Sub-Agent Dispatch

The three sub-agents under `wf/agents/` are **only** dispatched from this skill. They are tuned to the feature-development context — invoking them from `/wf:plan` or `/wf:review` would distort their role.

### code-explorer (parallel, 2-3 instances)

Each instance targets a different aspect of the codebase. Example prompts:

- "Find features similar to <PRD title> and trace through their implementation comprehensively"
- "Map the architecture and abstractions for <area>, tracing through the code"
- "Analyze the current implementation of <existing feature/area>"
- "Identify UI patterns, testing approaches, or extension points relevant to <feature>"

Each agent should return 5-10 key files for the main agent to read directly. After agents return, **read those files** before continuing.

### code-architect (single instance)

Produces a complete implementation blueprint refining the PRD/ADR into concrete files, functions, and build sequence. This is one level of detail below the ADRs — ADRs say "use pattern X"; the architect says "create file `src/foo/bar.ts` with `parseInput(s: string): Result`."

Do not dispatch multiple architect instances with different focuses (that pattern belongs in `/wf:plan`). The plan stage already settled the trade-offs; the architect here just elaborates the chosen path.

### code-reviewer (parallel, 3 instances) — inner-loop only

After implementation, dispatch three reviewers with different focuses:

- "Review the changes for simplicity, DRY, and elegance"
- "Review the changes for bugs, logic errors, and security issues"
- "Review the changes for adherence to project conventions and abstractions in CLAUDE.md"

This is an **inner loop** — fix confidence ≥ 80 issues immediately before handoff. The formal external review is `/wf:review`, which the user runs explicitly.

## Implementation Discipline

- **Read identified files first** — sub-agent summaries are not a substitute for reading the actual code.
- **Follow CLAUDE.md conventions strictly** — naming, error handling, logging language, line endings, comment language.
- **Keep edits scoped** — do not refactor adjacent code unless an ADR explicitly authorizes it.
- **Use TodoWrite** — track all progress as discrete tasks; mark each done as soon as it is finished.

## After Implementation

1. Run inner-loop reviewers (parallel three instances).
2. Fix all confidence ≥ 80 findings before handoff.
3. Update `.wf/STATE.md` phase log: append a coding-pass entry with the current ISO 8601 timestamp. Leave `phase: coding`.
4. Tell the user the implementation is ready for formal review and to run `/wf:review`.

Do NOT advance `phase` to `reviewing` or `done` — `/wf:review` does that.

## UI Tasks

If the implementation includes frontend/UI work and the `frontend-design` skill is available (typically from Anthropic's `document-skills` plugin), invoke it via the Skill tool — its visual design output is more polished than rolling your own. The skill is **soft-referenced**: it is not declared as a dependency, and the implementation must not fail if it is missing.

If `frontend-design` is not available, follow this inline checklist (distilled from its prose):

- **Typography** — pick a distinctive display + refined body pairing. Avoid Inter, Roboto, Arial, system fonts.
- **Color** — commit to a cohesive palette via CSS variables. Dominant colors with sharp accents outperform timid, evenly-distributed palettes.
- **Motion** — focus on high-impact moments (one well-orchestrated load animation) over scattered micro-interactions. Prefer CSS-only solutions.
- **Spatial composition** — use asymmetry, overlap, generous negative space, or controlled density. Avoid generic centered grid layouts.
- **NEVER** use purple-gradient-on-white, predictable hero/feature/footer scaffolds, or cookie-cutter component patterns that lack context-specific character.
- **Match implementation complexity to aesthetic vision** — maximalist designs need elaborate code; minimalist designs need restraint and precision.

## Key Principles

- **Architecture is settled** — do not re-litigate ADR decisions. If a decision needs to change, return to `/wf:plan`.
- **Read what explorers find** — summaries don't replace direct reading.
- **Inner-loop review is mandatory** — confidence ≥ 80 issues must be fixed before handoff.
- **Do not advance phase** — phase advancement is `/wf:review`'s job.
