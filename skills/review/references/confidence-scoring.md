# Confidence Scoring (Five-Tier)

Use this scale when applying confidence scores to potential issues during `/wf:review`. The scale is preserved verbatim from upstream `code-reviewer` (Anthropic `feature-dev` plugin) — this is a load-bearing rubric and should not be paraphrased.

## Scale

Rate each potential issue on a scale from 0-100:

- **0**: Not confident at all. This is a false positive that doesn't stand up to scrutiny, or is a pre-existing issue.
- **25**: Somewhat confident. This might be a real issue, but may also be a false positive. If stylistic, it wasn't explicitly called out in project guidelines.
- **50**: Moderately confident. This is a real issue, but might be a nitpick or not happen often in practice. Not very important relative to the rest of the changes.
- **75**: Highly confident. Double-checked and verified this is very likely a real issue that will be hit in practice. The existing approach is insufficient. Important and will directly impact functionality, or is directly mentioned in project guidelines.
- **100**: Absolutely certain. Confirmed this is definitely a real issue that will happen frequently in practice. The evidence directly confirms this.

**Only report issues with confidence ≥ 80.** Focus on issues that truly matter — quality over quantity.

## Why 80?

The threshold sits between "highly confident" (75) and "absolutely certain" (100). It is deliberately low enough to catch real problems but high enough to filter out nitpicks that would create review-fatigue and erode trust in the review skill. Lowering it floods the review with stylistic noise; raising it lets real bugs slip through.

In `/wf:review`, the threshold is also the **gate threshold**: any issue ≥ 80 forces the STATE phase back to `coding`. This means false positives at this threshold are not free — they incur a round of avoidable rework. Be precise.

## Severity Bands Within ≥ 80

For reporting structure inside the review document:

- **Critical (95-100)** — virtually certain bugs, security issues, broken functionality
- **Important (80-94)** — high-confidence issues that should be fixed but might survive scrutiny in edge cases

Both bands count equally toward the hard gate. The split is for human readability of the report, not for gate logic.

## What Does NOT Belong at ≥ 80

- Stylistic preferences not codified in `CLAUDE.md`
- "I'd write this differently" without a concrete defect
- Pre-existing issues not introduced by the current changes
- Refactoring opportunities (those belong in a separate `/wf:plan` cycle)
- Hypothetical issues that haven't been verified through the actual code

When in doubt about whether something is ≥ 80, write it down at the score you genuinely believe and review the list before publishing. If it slipped down, drop it from the report.
