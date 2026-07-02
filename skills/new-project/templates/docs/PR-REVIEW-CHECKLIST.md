# PR Reviewer Checklist — [project] domain pass

Invoke via `Agent({subagent_type: "Plan", prompt: "Read docs/PR-REVIEW-CHECKLIST.md
and review PR #N against these criteria. Fetch the diff with 'gh pr diff N'.
Return a structured concerns list."})`.

This checklist complements — does not replace — automated review bots (generic
bug patterns) and CI (formatting, tests). The reviewer's job is what CI and
bots can't see: [project]-specific invariants.

## Triage — what applies to this PR?

If the diff touches no source files, this is a docs/infra/config PR — skip the
blocking items (they presuppose code changes), apply only the non-blocking
flags, and return quickly.

## Blocking concerns — MUST flag; PR must not merge without addressing

[One numbered item per load-bearing invariant: what to look for, the accepted
pattern(s), the known anti-pattern(s), where the invariant is enforced.]

## Non-blocking flags — comment for maintainer awareness

[Smells worth a comment: untested guards, missing issue links, PR body drifting
from the diff after follow-up commits.]

## Non-goals — do NOT spend review time on these

- Formatting / lint / type errors — CI covers these. Trust the CI signals.
- Restating the diff — the maintainer already saw it; the value is the domain angle.
- Style opinions — unless a name actively misleads.

## Output format

```markdown
## Domain review — PR #N

**Blocking concerns:** (empty list is fine)
- [file:line] concern + suggested fix

**Non-blocking flags:**
- [file:line] flag + reasoning

**Summary:** APPROVE / REQUEST CHANGES / DISCUSS
```
