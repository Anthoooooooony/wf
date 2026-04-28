# PRD and ADR Templates

Use these templates when drafting documents during `/wf:plan`.

## PRD Template

Filename: `.wf/plans/PRD-<feature_id>.md`

````markdown
---
feature_id: <feature_id>
status: draft | approved | superseded
created_at: <ISO 8601 with timezone>
brainstorm_ref: .wf/brainstorm/<feature_id>.md
---

# <Feature title>

## Background

Why this feature exists. Pull from the brainstorm spec. 2-4 sentences.

## Problem Statement

What is broken or missing today. Concrete, not abstract.

## Goals

- Numbered, measurable outcomes this feature must deliver.

## Non-Goals

- Numbered list of things explicitly out of scope. This section is at least as important as Goals.

## User Stories

- As a <role>, I want <capability>, so that <outcome>.

## Functional Requirements

Numbered. Each requirement should be testable.

## Non-Functional Requirements

Performance, accessibility, security, observability constraints. Only include the ones that actually matter for this feature.

## Architecture Notes

Reference the chosen ADRs by number. Note any conflicts with existing ADRs. Do not duplicate ADR content here.

## Open Questions

Tracked questions that don't block planning approval but need resolving during `/wf:code`.

## References

- Brainstorm spec: `.wf/brainstorm/<feature_id>.md`
- Related ADRs: `.wf/adr/ADR-NNN-*.md`
- Related code: `<file>:<line>` references
````

## ADR Template

Filename: `.wf/adr/ADR-<NNN>-<kebab-slug>.md`

````markdown
---
number: NNN
status: proposed | accepted | superseded | rejected
date: <ISO 8601 date, e.g. 2026-04-28>
feature_id: <feature_id>
supersedes: ADR-NNN     # optional
superseded_by: ADR-NNN  # optional, set when status becomes superseded
---

# ADR-NNN: <Decision summary in ≤10 words>

## Status

<status, with date if relevant>

## Context

The forces at play. What constraints, what existing code, what brainstorm requirement drives this decision. 3-6 sentences.

## Decision

The decision in plain language. One paragraph.

## Alternatives Considered

For each alternative:
- **<Option name>** — what it is, why it was rejected. 1-3 sentences.

At least two alternatives. "Do nothing" counts if applicable.

## Consequences

### Positive
- What gets better.

### Negative
- What gets worse, what we now have to live with.

### Neutral
- Trade-offs that aren't strictly better or worse, just different.

## References

- Related code: `<file>:<line>` references
- Related ADRs: ADR-NNN
- Brainstorm spec: `.wf/brainstorm/<feature_id>.md`
````

## Numbering

- ADR numbers are sequential across the entire project, padded to three digits.
- Read existing ADRs in `.wf/adr/` (and `docs/adr/` if your project has migrated some) before picking the next number.
- Start at `001` if no ADRs exist.

## When the User Promotes Documents to git

By default `.wf/` is gitignored. If the user wants to track PRDs or ADRs in version control, they manually `mv` (or `cp`) the file to `docs/plans/` or `docs/adr/`. The plan skill never moves files into tracked directories on its own.
