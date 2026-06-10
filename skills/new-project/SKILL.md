---
name: new-project
description: Scaffold a new project for Claude using Context-Driven Development — installs skills, sets up desloppify, and interviews the user to produce a CLAUDE.md + Agent Context Stack (docs/SPEC.md, GLOSSARY.md, STATUS.md, decisions/). Use when starting a new project, when asked to "init project" or "set up a new project" for a repo, or when CLAUDE.md and docs/ are missing.
---

# New Project

Scaffold a new project for Claude collaboration using **Context-Driven Development** — a practice of maintaining an **Agent Context Stack** (CLAUDE.md + structured docs/) that gives Claude durable, phase-aware project context that survives conversation compression and multi-agent handoffs.

Work interactively. Ask one topic at a time, offer a recommendation for each, wait for the answer before moving on.

---

## Phase 1 — Tooling setup

### Step 1.1 — Get the repo

Ask:

> What is the GitHub repo URL for this project? (e.g. `https://github.com/you/your-repo`)

Once given, set `REPO` = the `owner/name` slug. All `gh` commands use this.

### Step 1.2 — Install mattpocock/skills

Run:

```bash
npx skills@latest add mattpocock/skills
```

Let the user select which skills they want. When they're done, confirm what was installed.

### Step 1.3 — Set up desloppify

Desloppify supports 29 languages including full plugin depth for TypeScript, Python, C#, Go, Rust, and more. It is installed as a Python tool regardless of the project's language stack.

```bash
pip install --upgrade "desloppify[full]"
desloppify update-skill claude
```

`update-skill claude` installs the full desloppify workflow guide into the project so Claude knows the exact loop to follow.

After installing, exclude any directories that shouldn't be scanned (build output, vendor, generated code, worktrees) — check with the user before excluding anything non-obvious. Then run:

```bash
desloppify scan --path .
desloppify status
```

For monorepos or projects with a separate frontend and backend, scan each path separately:
```bash
desloppify --lang typescript scan --path ./frontend
desloppify --lang python scan --path ./backend
```

Show the user their initial score. This is baseline — tell them the workflow is: run `desloppify next` at the end of each phase and work through findings before moving on.

### Step 1.4 — GitHub issues as backlog

Ask:

> Should "add to backlog" mean creating a GitHub issue in this repo? (Recommended: yes, if this is on GitHub)

**If yes:** Confirm that whenever the user says "add [X] to the backlog", Claude will run:
```bash
gh issue create --repo REPO --title "..." --body "..." --label "backlog"
```
Record this in the `## Agent skills` block of `CLAUDE.md`.

**If no:** Ask where their backlog lives (local markdown, Linear, Jira, etc.) and record accordingly.

### Step 1.5 — Run /setup-matt-pocock-skills

Run `/setup-matt-pocock-skills` now. This configures the issue tracker, triage labels, and domain doc layout for the engineering skills. Walk through it before continuing.

---

## Phase 2 — Interview

Ask one section at a time. For each, present a recommendation grounded in Context-Driven Development principles, then wait. Adapt the recommendation to what you already know about the project from the repo.

---

### Section A — What is this project?

Ask:

> Describe the project in 2–3 sentences: what it does, who uses it, and the core problem it solves.

Then ask:

> What's the stack? (Languages, frameworks, databases, deployment)

Use these answers to write the `## What This Project Is` block in `CLAUDE.md` and stub out `docs/SPEC.md`.

**Recommendation to offer:** `docs/SPEC.md` is a core part of the Agent Context Stack — it's the deeper reference Claude reads before any non-trivial task. Start with a short spec and grow it. Without it, agents make architecture assumptions that conflict with your intent.

---

### Section B — Working principles

Present each principle below, explain it briefly, and ask whether to include it. These are the core Context-Driven Development principles — offer the default wording for each. The user can accept as-is, edit, or skip.

**Principles to offer (in order):**

1. **Ubiquitous Language** — Domain glossary lives in `docs/GLOSSARY.md`. All code, tests, and prompts use those exact terms. No synonyms.
   > *Recommendation: Include this. Even a short glossary prevents silent terminology drift across long sessions.*

2. **Test-Driven Development** — Failing test first, then implement. Tests validated against real behaviour.
   > *Recommendation: Include for any module with non-trivial logic. Ask the user which parts (all? backend only? simulator/business logic only?) require TDD.*

3. **Deep Modules** — Simple interfaces, complex internals. If the interface is getting complicated, refactor before proceeding.
   > *Recommendation: Include. Pair it with a concrete example of the project's top-level interface.*

4. **Spec Before Code** — For non-trivial modules, propose the interface and wait for approval before implementing.
   > *Recommendation: Include. This principle alone saves enormous rework.*

5. **Design Delegation** — The human owns architecture and interface design. Claude owns implementation. Surface design decisions as questions, not assumptions.
   > *Recommendation: Include as-is.*

6. **Vertical Slices** — Build end-to-end in thin slices. Always the thinnest slice that produces something testable and demonstrable.
   > *Recommendation: Include if the project has multiple layers (frontend + backend, pipeline + UI, etc.).*

---

### Section C — Before starting any task

Ask:

> Which of these docs should Claude read before starting any task? (Select all that apply; you'll create stubs for them now)

Offer:
- `docs/SPEC.md` — architecture and design decisions
- `docs/GLOSSARY.md` — domain terms
- `docs/STATUS.md` — current phase, test counts, health score
- `docs/BACKLOG.md` — queued work items (or note that backlog is in GitHub Issues)
- `docs/decisions/` — numbered design decision docs (ADR-style)

**Recommendation:** All five. The checklist in `CLAUDE.md` that says "read these before starting" is what makes long-session continuity work. Without it, agents reintroduce resolved decisions.

---

### Section D — Code style

Ask:

> What are your formatters, linters, and type-checking tools? Include the commands you run.

**Recommendation to offer (adapt to detected stack):**

- Python: Ruff for format + lint (`ruff format .`, `ruff check .`), strict type hints on all signatures, no bare `except`.
- TypeScript/React: TypeScript strict mode always on, Prettier for format, ESLint for lint, named exports, one component per file.
- General: no comments that restate what the code does; no error handling for impossible scenarios.

Ask: *Do you want a `## Code Style` section in `CLAUDE.md` with these rules?*

---

### Section E — Current phase

Ask:

> What phase or milestone is the project at right now? (One sentence is fine — e.g. "Phase 1 — initial data pipeline" or "MVP feature complete, pre-launch polish")

**Recommendation:** Record this as a `## Current Phase` section in `CLAUDE.md` with a pointer to `docs/STATUS.md` for detail. Update it at the end of each phase.

---

### Section F — Environment notes

Ask:

> Any environment gotchas Claude should know? (Node version manager, Python path, Tailwind version, env vars, Docker setup, etc.)

**Recommendation:** If the project uses `fnm`, `nvm`, `pyenv`, or similar — record the activation commands explicitly. These are the first thing that breaks in a new session.

---

### Section G — Scope/monetisation boundaries (optional)

Ask:

> Are there any hard scope limits Claude should never cross? (e.g. "the free tier always includes X", "never add paid restrictions to core features", "this module is read-only")

Skip if none.

---

### Section I — Security boundaries (optional)

Ask:

> Does this project handle anything security-sensitive? (user authentication, payments, personal data, public APIs, secrets management)

**If no:** Skip. A minimal baseline will still be written into `CLAUDE.md` unconditionally.

**If yes:** Ask:

> What are the hard security rules Claude should never violate? (e.g. "never log request bodies", "all user input validated at the API boundary", "no secrets in code — use env vars", "auth middleware must never be bypassed for convenience")

Use the answers to write a `## Security Boundaries` section in `CLAUDE.md` below the baseline.

---

### Section H — Multi-agent workflow (optional)

Ask:

> Do you plan to use parallel Claude Code agents working on isolated git worktrees for this project?

**If yes:** Add the multi-agent rules block (stream ownership, interfaces-first, one class per entity, exit conditions, PR workflow). Ask whether they want `.claude/worktrees/` created now.

**If no:** Skip.

---

## Phase 3 — Write

Once all sections are answered, produce:

**Length budget: keep the generated `CLAUDE.md` under 200 lines.** It is loaded as persistent context in every session — every line costs context window. Write principles as tight rules (2-3 lines each), not essays. If a section is getting long, move the detail to the relevant `docs/` file and add a pointer instead.

### `CLAUDE.md`

Assemble all answered sections into `CLAUDE.md` using the Agent Context Stack structure:
1. What This Project Is
2. Working Principles (only the ones the user opted in to)
3. What To Do Before Starting Any Task (checklist of docs they confirmed)
4. Code Style
5. Current Phase
6. Desloppify section (see template below)
7. When to Use Skills (always include — see template below)
8. Security Boundaries (always include baseline — expand if Section I answers warrant it)
9. Environment Notes
10. Monetisation Boundaries (if applicable)
11. Multi-Agent Workflow (if applicable)
12. Agent skills block (from `/setup-matt-pocock-skills` output)

#### Desloppify section template

```markdown
## Desloppify (Quality Review)

Run after completing each development phase:
\```bash
desloppify scan
desloppify score
desloppify next   # work through findings as an agent
\```

Run `desloppify next` and work through findings before moving to the next phase.
```

#### When to Use Skills template (always include)

```markdown
## When to Use Skills

- **`/grill-with-docs`** — before implementing anything non-trivial. Stress-tests the plan against GLOSSARY.md and docs/decisions/, and updates docs inline as decisions crystallise. Use it before writing code for a new module or feature.
- **`/improve-codebase-architecture`** — when an interface is getting complicated or a module feels tangled. Run it before starting a new phase, not just reactively.
- **`/tdd`** — for any business logic, data pipeline, or backend module. Write the failing test first.
- **`/diagnose`** — when something is broken and the cause isn't obvious. Don't just start changing code.
- **`/to-issues`** — when turning a plan, spec, or conversation into tracked GitHub issues.
- **`/triage`** — when the user asks to review, prioritise, or manage open issues.
- **`desloppify next`** — at the end of each phase before marking it complete. Not optional.
```

#### Security Boundaries template

Always write the baseline. Expand with project-specific rules from Section I if applicable.

```markdown
## Security Boundaries

- No secrets, tokens, or credentials in code — use environment variables
- No sensitive data (tokens, passwords, PII) in logs or error messages
- Validate all input at system boundaries (user input, external APIs) — trust internal code
- Never bypass authentication or authorisation as a convenience shortcut

[Project-specific rules from Section I, if any]
```

#### Backlog section (if GitHub issues)

Add to the "What To Do Before Starting Any Task" section:

```markdown
**Backlog:** Tracked in GitHub Issues. To add an item mid-conversation, say "add [description] to the backlog" — Claude will run `gh issue create --repo OWNER/REPO --title "..." --body "..."`.
```

### `docs/` stubs

For each doc the user confirmed, create a stub:

**`docs/SPEC.md`**
```markdown
# [Project Name] — Specification

This file is the source of truth for architecture decisions and module interfaces.
Read it before building anything non-trivial.

## Architecture

[To be filled in]

## Key Interfaces

[To be filled in]
```

**`docs/GLOSSARY.md`**
```markdown
# [Project Name] — Glossary

Authoritative reference for domain terms. Use these terms exactly as written in code, tests, comments, and prompts. Do not invent synonyms.

---

[Terms to be added]
```

**`docs/STATUS.md`**
```markdown
# [Project Name] — Project Status

Tracks current phase, test counts, and health score.
Update when phases complete or numbers change.

---

## Current numbers

| Metric | Value |
|---|---|
| Tests passing | — |
| Desloppify strict | [from initial scan] |
| Phase | [Phase 1] |
```

**`docs/BACKLOG.md`** (only if not using GitHub Issues)
```markdown
# [Project Name] — Backlog

Queued work items.

| Priority | Item | Notes |
|---|---|---|
```

**`docs/decisions/`** — create the folder and a `000-template.md`:
```markdown
# [NNN] — [Decision Title]

**Date:** [YYYY-MM-DD]
**Status:** Accepted

## Context

[What is the situation that led to this decision?]

## Decision

[What was decided?]

## Consequences

[What are the trade-offs?]
```

### Commit

Once files are written, offer to run:
```bash
git add CLAUDE.md docs/
git commit -m "chore: add CLAUDE.md and docs scaffolding"
```

---

## Done

Tell the user:
- What was installed and configured
- Which skills are now active and what they do (one line each)
- The desloppify baseline score
- What to fill in next (SPEC.md, GLOSSARY.md entries, first decisions doc)
- That "add [X] to the backlog" will create a GitHub issue from now on (if configured)
