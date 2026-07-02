# AGENTS.md section templates

Referenced from SKILL.md Phase 3. Each template below is wrapped in a four-backtick
fence — copy the fence *contents* into `AGENTS.md`, substituting `[bracketed]`
values. Skip any section the user didn't opt in to.

---

## Current Phase (always — pointer, never a copy of status numbers)

````markdown
## Current Phase — see `docs/STATUS.md`

[One sentence, e.g. "v1 launched; per-module quality push ongoing."]

**Do not duplicate status numbers here.** `docs/STATUS.md` is the single
source of truth for test counts, metrics, and phase detail. Numbers copied
here go stale and mis-brief fresh sessions.
````

---

## Desloppify (always)

````markdown
## Desloppify (Quality Review)

Run after completing each development phase:
```bash
desloppify scan
desloppify score
desloppify next   # work through findings as an agent
```

Run `desloppify next` and work through findings before moving to the next phase.
````

---

## Conversation Strategy (Claude Code only)

````markdown
## Conversation Strategy

Treat this conversation like the main branch in GitHub Flow — it's for planning,
design decisions, and review, not implementation.

For any task involving meaningful implementation:
1. Plan and agree the approach here
2. Use `/handoff` to delegate implementation to a sub-agent
3. The sub-agent gets the full Agent Context Stack and executes independently
4. It reports back with what was built, any decisions made, and a PR to review

This keeps the main conversation focused on decisions and prevents context
compaction from eating the planning context that makes it useful.
````

---

## When to Use Skills (Claude Code only)

````markdown
## When to Use Skills

- **`/grill-with-docs`** — before implementing anything non-trivial. Stress-tests the plan against GLOSSARY.md and docs/decisions/, and updates docs inline as decisions crystallise. Use it before writing code for a new module or feature.
- **`/improve-codebase-architecture`** — when an interface is getting complicated or a module feels tangled. Run it before starting a new phase, not just reactively.
- **`/tdd`** — for any business logic, data pipeline, or backend module. Write the failing test first.
- **`/diagnose`** — when something is broken and the cause isn't obvious. Don't just start changing code.
- **`/to-issues`** — when turning a plan, spec, or conversation into tracked GitHub issues.
- **`/triage`** — when the user asks to review, prioritise, or manage open issues.
- **`/handoff`** — when starting any meaningful implementation task. Plan here, implement in a sub-agent. Keeps this conversation focused on decisions.
- **`desloppify next`** — at the end of each phase before marking it complete. Not optional.
````

---

## Security Boundaries (always — write the baseline; expand with Section H answers)

````markdown
## Security Boundaries

- No secrets, tokens, or credentials in code — use environment variables
- No sensitive data (tokens, passwords, PII) in logs or error messages
- Validate all input at system boundaries (user input, external APIs) — trust internal code
- Never bypass authentication or authorisation as a convenience shortcut

[Project-specific rules from Section H, if any]
````

---

## Multi-Agent Workflow (Claude Code only, if Section I answered yes)

Adapt the bracketed parts to the project. Every rule below was learned from a
real multi-agent failure — keep them all unless the user objects.

````markdown
## Multi-Agent Workflow

Parallel agents work on isolated git worktrees. Rules:

1. **Know your scope.** Do not write to files outside your assigned scope — even
   if it seems helpful. Shared files ([list the hot files many streams touch])
   are off-limits to stream agents: if a shared-file gap blocks your work, file
   an issue and stop. Do not compound the multi-agent conflict surface.
2. **APPEND-ONLY discipline for shared files.** When a shared-file edit is
   unavoidable, add new entries (dict keys, whitelist rows) — never modify
   existing dispatch/parsing paths another stream may depend on.
3. **Exit condition is "PR URL reported and CI green."** Not "validation
   completed", not "tests passed locally". Open the PR BEFORE running any
   long-running validation (benchmarks, full E2E) — agents that die mid-run
   orphan their work in a worktree. Post validation results as a PR comment after.
4. **Run the full CI script before declaring done** ([command]). Running only
   your module's tests is not sufficient.
5. **Never `git add .` or `git add -u`** — worktree spillover can silently pull
   another agent's in-flight files into your commit. Stage explicit paths only.
6. **Never `--no-verify`.** Fix the underlying issue.
7. **Do not merge your own PR.** Report the URL.
8. **Defer honestly.** If the target is out of reach within scope, land what you
   have, report the residual gap, and file an issue for the next agent. A small
   honest result beats a large one that fudges numbers off-camera.
9. If results are measured against a noisy metric, report per-item deltas —
   never claim aggregate movement from a single run.
````

---

## PR Review (if Section J answered yes)

````markdown
## PR Review

Two-layer review on every non-trivial PR, plus CI:

1. **Review bot (automatic, generic-bug pass)** — [bot name, e.g. Cursor Bugbot]
   auto-comments on PRs at open. Do not merge until it has run and blocking
   concerns are addressed.
2. **Domain reviewer (spawned)** — for PRs touching core or shared modules:
   ```
   Agent({ subagent_type: "Plan", prompt: "Read docs/PR-REVIEW-CHECKLIST.md and review PR #N. Fetch the diff with 'gh pr diff N'. Return the structured concerns list from that file." })
   ```
   The checklist codifies the project invariants a generic bot doesn't know.
````

---

## Backlog line (if GitHub Issues is the backlog — add to "What To Do Before Starting Any Task")

````markdown
**Backlog:** Tracked in GitHub Issues. To add an item mid-conversation, say "add [description] to the backlog" — the agent will run `gh issue create --repo OWNER/REPO --title "..." --body "..."`.
````
