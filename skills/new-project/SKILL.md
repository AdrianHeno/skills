---
name: new-project
description: Scaffold a new project using Context-Driven Development — sets up desloppify, installs agent-specific tooling, and interviews the user to produce an AGENTS.md + Agent Context Stack (docs/SPEC.md, GLOSSARY.md, STATUS.md, decisions/). Supports Claude Code, Cursor, Copilot, Windsurf, and Codex. Use when starting a new project, when asked to "init project" or "set up a new project" for a repo, or when AGENTS.md and docs/ are missing.
---

# New Project

Scaffold a new project using **Context-Driven Development** — a practice of maintaining an **Agent Context Stack** (`AGENTS.md` + structured `docs/`) that gives any AI agent durable, phase-aware project context that survives conversation compression, multi-agent handoffs, and new sessions.

Work interactively. Ask one topic at a time, offer a recommendation for each, wait for the answer before moving on.

---

## Phase 1 — Tooling setup

### Step 1.1 — Get the repo

Ask:

> What is the GitHub repo URL for this project? (e.g. `https://github.com/you/your-repo`)

Once given, set `REPO` = the `owner/name` slug. All `gh` commands use this.

### Step 1.2 — Which agent(s)?

Ask:

> Which AI coding agent(s) are you using on this project? (Select all that apply)
> - Claude Code
> - Cursor
> - GitHub Copilot
> - Windsurf
> - Codex
> - Other

Record the selection as `AGENTS`. Set `IS_CLAUDE_CODE = true` if Claude Code is included.

This gates which tooling gets installed and which sections get written into `AGENTS.md`.

### Step 1.3 — Install Claude Code skills (Claude Code only)

**Skip this step if `IS_CLAUDE_CODE` is false.**

Run:

```bash
npx skills@latest add mattpocock/skills
```

Let the user select which skills they want. When they're done, confirm what was installed.

### Step 1.4 — Set up desloppify

Desloppify supports 29 languages including full plugin depth for TypeScript, Python, C#, Go, Rust, and more. It is installed as a Python tool regardless of the project's language stack.

```bash
pip install --upgrade "desloppify[full]"
```

Then install the workflow guide for each selected agent. The desloppify agent name mapping:

| Agent | Command |
|---|---|
| Claude Code | `desloppify update-skill claude` |
| Cursor | `desloppify update-skill cursor` |
| GitHub Copilot | `desloppify update-skill copilot` |
| Windsurf | `desloppify update-skill windsurf` |
| Codex | `desloppify update-skill codex` |

Run `update-skill` for each agent in `AGENTS`. This installs the full desloppify workflow guide so each agent knows the exact fix loop.

Add `.desloppify/` to `.gitignore` (it contains local state that shouldn't be committed):

```bash
echo ".desloppify/" >> .gitignore
```

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

### Step 1.5 — GitHub issues as backlog

Ask:

> Should "add to backlog" mean creating a GitHub issue in this repo? (Recommended: yes, if this is on GitHub)

**If yes:** Confirm that whenever the user says "add [X] to the backlog", the agent will run:
```bash
gh issue create --repo REPO --title "..." --body "..." --label "backlog"
```
Record this in the `## Agent skills` block of `AGENTS.md`.

**If no:** Ask where their backlog lives (local markdown, Linear, Jira, etc.) and record accordingly.

### Step 1.6 — Project health hooks (Claude Code only)

**Skip this step if `IS_CLAUDE_CODE` is false.**

Copy `templates/claude-settings-hooks.json` (in this skill's directory) to the project's `.claude/settings.json`. If the project already has a `.claude/settings.json`, merge the `hooks` arrays instead of overwriting. All four hooks are shell commands — zero token cost.

**What each hook does:**
- **AGENTS.md / CLAUDE.md size** (PostToolUse) — warns if either exceeds 250 lines
- **docs/ file size** (PostToolUse) — warns if SPEC.md or GLOSSARY.md exceeds 400 lines
- **STATUS.md size** (PostToolUse) — warns if STATUS.md exceeds 150 lines; it must stay a one-screen dashboard, with dated content moved to CHANGELOG.md
- **STATUS.md staleness** (Stop) — warns at the end of any turn where source files are newer than STATUS.md

### Step 1.7 — Run /setup-matt-pocock-skills (Claude Code only)

**Skip this step if `IS_CLAUDE_CODE` is false.**

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

Use these answers to write the `## What This Project Is` block in `AGENTS.md` and stub out `docs/SPEC.md`.

**Recommendation to offer:** `docs/SPEC.md` is a core part of the Agent Context Stack — it's the deeper reference the agent reads before any non-trivial task. Start with a short spec and grow it. Without it, agents make architecture assumptions that conflict with your intent.

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

5. **Design Delegation** — The human owns architecture and interface design. The agent owns implementation. Surface design decisions as questions, not assumptions.
   > *Recommendation: Include as-is.*

6. **Vertical Slices** — Build end-to-end in thin slices. Always the thinnest slice that produces something testable and demonstrable.
   > *Recommendation: Include if the project has multiple layers (frontend + backend, pipeline + UI, etc.).*

After the principles, ask:

> Does this project have 1–3 **load-bearing invariants** — properties that must never break, because the product's credibility depends on them? (e.g. "no simulated result may exceed 120% of the published benchmark", "all money math in integer cents", "user data is never mutated by read paths")

**If yes:** Record them for a `## Load-Bearing Invariants` section near the top of `AGENTS.md`. For each invariant, capture: the rule, where it is enforced (test file, constant, gate), and what to do on violation (revert or gate before merging — never ship a breach). These are the highest-value lines in the whole file — a fresh agent that knows the invariants can be trusted with far more autonomy.

**If no:** Skip — but suggest revisiting once the project has a quality bar users rely on.

---

### Section C — Before starting any task

Ask:

> Which of these docs should the agent read before starting any task? (Select all that apply; you'll create stubs for them now)

Offer:
- `docs/SPEC.md` — architecture and design decisions
- `docs/GLOSSARY.md` — domain terms
- `docs/STATUS.md` — current phase, test counts, health score (a `docs/CHANGELOG.md` is created alongside it to hold dated history, so STATUS.md stays a one-screen dashboard)
- `docs/BACKLOG.md` — queued work items (or note that backlog is in GitHub Issues)
- `docs/decisions/` — numbered design decision docs (ADR-style)

**Recommendation:** All five. The checklist in `AGENTS.md` that says "read these before starting" is what makes long-session continuity work. Without it, agents reintroduce resolved decisions.

---

### Section D — Code style

Ask:

> What are your formatters, linters, and type-checking tools? Include the commands you run.

**Recommendation to offer (adapt to detected stack):**

- Python: Ruff for format + lint (`ruff format .`, `ruff check .`), strict type hints on all signatures, no bare `except`.
- TypeScript/React: TypeScript strict mode always on, Prettier for format, ESLint for lint, named exports, one component per file.
- General: no comments that restate what the code does; no error handling for impossible scenarios.

Ask: *Do you want a `## Code Style` section in `AGENTS.md` with these rules?*

---

### Section E — Current phase

Ask:

> What phase or milestone is the project at right now? (One sentence is fine — e.g. "Phase 1 — initial data pipeline" or "MVP feature complete, pre-launch polish")

**Recommendation:** Record this as a `## Current Phase` section in `AGENTS.md` with a pointer to `docs/STATUS.md` for detail.

**Hard rule — pointer, not copy:** the section must contain the one-sentence phase and the pointer only. Never duplicate test counts, benchmark numbers, or feature status into `AGENTS.md` — duplicated numbers go stale within weeks and then actively mis-brief every fresh session (an agent working from stale numbers is worse than one with no numbers). `docs/STATUS.md` is the single source of truth. Use the "Current Phase" template in `templates/agents-md-sections.md`.

---

### Section F — Environment notes

Ask:

> Any environment gotchas the agent should know? (Node version manager, Python path, Tailwind version, env vars, Docker setup, etc.)

**Recommendation:** If the project uses `fnm`, `nvm`, `pyenv`, or similar — record the activation commands explicitly. These are the first thing that breaks in a new session.

---

### Section G — Scope/monetisation boundaries (optional)

Ask:

> Are there any hard scope limits the agent should never cross? (e.g. "the free tier always includes X", "never add paid restrictions to core features", "this module is read-only")

Skip if none.

---

### Section H — Security boundaries (optional)

Ask:

> Does this project handle anything security-sensitive? (user authentication, payments, personal data, public APIs, secrets management)

**If no:** Skip. A minimal baseline will still be written into `AGENTS.md` unconditionally.

**If yes:** Ask:

> What are the hard security rules the agent should never violate? (e.g. "never log request bodies", "all user input validated at the API boundary", "no secrets in code — use env vars", "auth middleware must never be bypassed for convenience")

Use the answers to write a `## Security Boundaries` section in `AGENTS.md` below the baseline.

---

### Section I — Multi-agent workflow (Claude Code only, optional)

**Skip this section if `IS_CLAUDE_CODE` is false.**

Ask:

> Do you plan to use parallel Claude Code agents working on isolated git worktrees for this project?

**If yes:** Add the Multi-Agent Workflow section from `templates/agents-md-sections.md` — a set of battle-tested rules covering scope ownership, shared-file discipline, exit conditions, and git hygiene. Adapt the shared-file list and CI command to this project. Ask whether they want `.claude/worktrees/` created now.

**If no:** Skip.

---

### Section J — PR review (optional, GitHub projects)

**Skip this section if the project doesn't use pull requests.**

Ask:

> Do you want a two-layer PR review setup? Author-merged PRs with zero reviews are the default failure mode of solo + agent development. The two layers: (1) an automated review bot for generic bug patterns (e.g. Cursor Bugbot via the Cursor GitHub App — a one-time dashboard action), and (2) a domain checklist (`docs/PR-REVIEW-CHECKLIST.md`) that a read-only agent runs against each non-trivial PR to check the project-specific invariants no generic bot knows.

**If yes:**
- Create `docs/PR-REVIEW-CHECKLIST.md` from `templates/docs/PR-REVIEW-CHECKLIST.md`, seeding the blocking items from the load-bearing invariants captured in Section B and any architecture rules from `docs/SPEC.md`.
- If Cursor is in `AGENTS`, also create `.cursor/rules/<project>-domain.mdc` from `templates/cursor-domain.mdc` — Cursor's project-rules file, read automatically by Chat, Composer, and Bugbot on PRs. Keep it terse: it loads into every Cursor AI interaction. Note that enabling Bugbot itself is a Cursor dashboard action, not a CLI step.
- Add the "PR Review" section from `templates/agents-md-sections.md` to `AGENTS.md`: don't merge until the bot has run and blocking concerns are addressed; spawn the checklist reviewer for PRs touching core/shared modules.

**If no:** Skip.

---

## Phase 3 — Write

Once all sections are answered, produce:

**Length budget: keep `AGENTS.md` under 200 lines.** It is loaded as persistent context in every session — every line costs context window. Write principles as tight rules (2-3 lines each), not essays. If a section is getting long, move the detail to the relevant `docs/` file and add a pointer instead.

### `AGENTS.md` (primary — all agents)

Assemble all answered sections into `AGENTS.md` using the Agent Context Stack structure:
1. What This Project Is
2. Load-Bearing Invariants (if captured in Section B — keep near the top)
3. Working Principles (only the ones the user opted in to)
4. What To Do Before Starting Any Task (checklist of docs they confirmed)
5. Code Style
6. Current Phase (pointer to STATUS.md — never duplicated numbers)
7. Desloppify section
8. Conversation Strategy (Claude Code only)
9. When to Use Skills (Claude Code only)
10. Security Boundaries (always include baseline — expand if Section H answers warrant it)
11. Environment Notes
12. Monetisation Boundaries (if applicable)
13. Multi-Agent Workflow (Claude Code only, if applicable)
14. PR Review (if Section J answered yes)
15. Agent skills block (Claude Code only — from `/setup-matt-pocock-skills` output)

Sections 6–10, 13, and 14 have verbatim templates in `templates/agents-md-sections.md` (see the Templates section below). If GitHub Issues is the backlog, add the backlog line from the same file to section 4.

### `CLAUDE.md` (Claude Code only)

**If `IS_CLAUDE_CODE` is true**, create `CLAUDE.md` as a symlink to `AGENTS.md`:

```bash
ln -s AGENTS.md CLAUDE.md
```

Claude Code auto-loads `CLAUDE.md` at startup. The symlink means the content lives once in `AGENTS.md` and all agents share it — no duplication.

If the filesystem doesn't support symlinks (e.g. Windows without dev mode), copy the file instead and note that both files must be kept in sync.

---

### Templates — `templates/` in this skill's directory

All verbatim artefacts live as real files next to this SKILL.md — read them, substitute `[bracketed]` placeholders, and write them into the project. Do not improvise structure; the templates are the spec.

| Template | Written to | When |
|---|---|---|
| `templates/agents-md-sections.md` | sections of `AGENTS.md` | always — contains Current Phase, Desloppify, Conversation Strategy*, When to Use Skills*, Security Boundaries, Multi-Agent Workflow*, PR Review, and the GitHub-Issues backlog line |
| `templates/claude-settings-hooks.json` | `.claude/settings.json` | Claude Code only (Step 1.6) |
| `templates/docs/SPEC.md` | `docs/SPEC.md` | if confirmed in Section C |
| `templates/docs/GLOSSARY.md` | `docs/GLOSSARY.md` | if confirmed in Section C |
| `templates/docs/STATUS.md` | `docs/STATUS.md` | if confirmed in Section C |
| `templates/docs/CHANGELOG.md` | `docs/CHANGELOG.md` | always created alongside STATUS.md |
| `templates/docs/BACKLOG.md` | `docs/BACKLOG.md` | only if NOT using GitHub Issues |
| `templates/docs/decisions/000-template.md` | `docs/decisions/000-template.md` | if confirmed in Section C |
| `templates/docs/PR-REVIEW-CHECKLIST.md` | `docs/PR-REVIEW-CHECKLIST.md` | if Section J answered yes — seed blocking items from the Section B invariants |
| `templates/cursor-domain.mdc` | `.cursor/rules/<project>-domain.mdc` | if Section J answered yes AND Cursor in `AGENTS` |

*Claude Code only. Sections marked optional in `agents-md-sections.md` are skipped if the user didn't opt in.

Replace `[Project Name]` / `[project]` with the actual name in every copied file. In the Multi-Agent Workflow section, fill in the project's shared-file list and full-CI command. In the Security Boundaries section, always write the baseline; append the Section H answers if any.

### Commit

Once files are written, offer to run (include `.cursor/` only if it was created):
```bash
git add AGENTS.md CLAUDE.md docs/ .claude/ .cursor/
git commit -m "chore: add AGENTS.md and docs scaffolding"
```

---

## Done

Tell the user:
- Which agents were configured and what was installed for each
- The desloppify baseline score
- What to fill in next (SPEC.md, GLOSSARY.md entries, first decisions doc)
- That "add [X] to the backlog" will create a GitHub issue from now on (if configured)
