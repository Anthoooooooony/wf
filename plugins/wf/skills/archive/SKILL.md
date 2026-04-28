---
name: archive
description: Optional cleanup after the wf workflow completes. Moves a finished feature's artifacts (brainstorm, plans, reviews) into .wf/archive/<feature_id>/ and resets STATE.md so the next /wf:brainstorm can start fresh. ADRs are kept in .wf/adr/ since they remain project-level decisions. Slash-only.
disable-model-invocation: true
---

# wf:archive

Collapse a finished feature's scattered artifacts into a single archive directory. This skill is **optional** — if you prefer to keep finished features inline (so brainstorm / plans / reviews accumulate inline as a project history), simply do not run it.

## Phase Validation

Before doing anything else:

1. Read `.wf/STATE.md`. If absent, refuse — there is nothing to archive.
2. If `phase` is not `done`, refuse and tell the user the current phase. The user must finish the workflow (pass `/wf:review`) before archiving.
3. Read `feature_id` from STATE.md frontmatter.

## Checklist

You MUST create a task for each item:

1. **Validate phase=done and load feature_id**
2. **Collect artifacts** matching this feature_id:
   - `.wf/brainstorm/<feature_id>.md`
   - `.wf/brainstorm/<feature_id>-visual-*.html` (if any)
   - `.wf/plans/PRD-<feature_id>.md`
   - `.wf/reviews/<feature_id>-*.md` (one per review round)
   - **Do NOT** collect ADRs from `.wf/adr/` — they are project-level decisions, not feature-scoped, and should remain reusable across future features.
3. **Confirm with the user** — list every source path and its archive destination, then ask for explicit approval. Do NOT skip this confirmation.
4. **Create archive directory** `.wf/archive/<feature_id>/` (and `.wf/archive/<feature_id>/reviews/` for review subdir)
5. **Move files** with `git mv` if the project is a git repository, otherwise plain `mv`. Apply the renaming table below.
6. **Reset STATE.md** — clear `feature_id`, blank `phase`, reset `review_round` to 0, append a phase-log entry recording the archive action.
7. **Tell the user** — confirm what was moved, suggest next actions: start a new feature with `/wf:brainstorm`, or `git add` the archive directory if they want it tracked (note: `.wf/` is gitignored by default; the archive directory inherits that).

## File Renaming on Move

When moving files into `.wf/archive/<feature_id>/`, drop the redundant feature_id prefix from each filename — the containing directory already names the feature.

| Original path | Archive path |
|---|---|
| `.wf/brainstorm/<feature_id>.md` | `.wf/archive/<feature_id>/brainstorm.md` |
| `.wf/brainstorm/<feature_id>-visual-1.html` | `.wf/archive/<feature_id>/brainstorm-visual-1.html` |
| `.wf/plans/PRD-<feature_id>.md` | `.wf/archive/<feature_id>/PRD.md` |
| `.wf/reviews/<feature_id>-1.md` | `.wf/archive/<feature_id>/reviews/round-1.md` |
| `.wf/reviews/<feature_id>-2.md` | `.wf/archive/<feature_id>/reviews/round-2.md` |

## STATE.md Reset

After archiving, rewrite `.wf/STATE.md` to:

```yaml
---
feature_id:
phase:
created_at:
review_round: 0
---

## Current Task

## Phase Log
- <ISO 8601 timestamp> archived <feature_id> to .wf/archive/<feature_id>/
```

The next `/wf:brainstorm` invocation will detect the empty STATE and treat it as a fresh start.

## What This Skill Does NOT Do

- **Does not delete anything** — files are moved into `archive/`, never removed. If you want to delete a feature's artifacts entirely, do it manually after archiving.
- **Does not touch ADRs** — `.wf/adr/` is left untouched since ADRs are project-level decisions reusable across features.
- **Does not commit to git** — `.wf/` is gitignored. If you want the archive tracked, you must explicitly `git add` it and force past the gitignore yourself.
- **Does not run automatically** — slash-only invocation, like every other wf skill.

## When to Use This Skill

Run after `/wf:review` reports a `pass` verdict (STATE phase becomes `done`). Use it when:

- You want a clean `.wf/` for the next feature.
- You want all artifacts of a finished feature collated in one place for future reference.
- You are about to start a new `/wf:brainstorm` and want the previous feature's scattered files out of the way.

If you prefer keeping finished features inline as accumulating project history, do not run this skill — it is optional, not a required terminal step.

## Key Principles

- **Optional, not mandatory** — workflow completion is `/wf:review` passing, not archiving.
- **Move, never delete** — archiving is a relocation, not a destruction.
- **ADRs are project-scoped** — they stay in `.wf/adr/` across feature cycles.
- **Confirm before moving** — always show the file-move plan and get explicit approval before touching the filesystem.
