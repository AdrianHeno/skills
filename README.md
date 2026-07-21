# skills

Skills built around **Context-Driven Development** — a pattern for AI-assisted software development that applies proven computer science principles to the problem of keeping an agent reliably informed across sessions, team members, and time.

Supports Claude Code, Cursor, GitHub Copilot, Windsurf, and Codex.

## Install

```bash
npx skills@latest add AdrianHeno/skills
```

---

## The problem this solves

Working with an AI coding agent on a real project exposes a set of failure modes that don't exist when working with a human team:

- **Context amnesia** — each session starts from scratch. Without explicit briefing, the agent re-learns the domain, re-litigates resolved decisions, and drifts from established conventions.
- **Terminology drift** — agents invent synonyms for domain concepts. Over time, the codebase uses five words for the same thing.
- **Architecture assumption** — without a spec, agents fill in design gaps themselves. The resulting code is locally reasonable but globally inconsistent.
- **Invisible progress** — no shared understanding of what phase the project is in, what's done, or what the quality baseline is.
- **Skill blindness** — agents don't proactively reach for the right tool at the right moment unless explicitly told when to.
- **Confident staleness** — the context you *did* write drifts out of date, and an agent acting on stale claims is worse than an unbriefed one: it doesn't ask, because it believes it already knows.

These aren't model failures. They're information failures. The agent is capable — it just doesn't have what it needs.

---

## What is Context-Driven Development?

Context-Driven Development (CDD) is the practice of maintaining a structured set of documents — an **Agent Context Stack** — that gives an AI agent durable, phase-aware project context that survives conversation compression, multi-agent handoffs, and new sessions.

It's not a new idea, and it isn't theoretical. It applies well-established computer science and software engineering principles to the specific constraints of agentic development — and every rule in the stack earned its place from a documented failure in a real production codebase running parallel agents: a status block that went two weeks stale and mis-briefed every fresh session, docs that referenced a directory that never existed, a `git add .` in a worktree that silently swept 97 files of another agent's in-flight work into a commit. The methodology is the accumulated fixes.

### Ubiquitous Language — from Domain-Driven Design (Evans, 2003)

DDD's core insight is that a shared vocabulary between engineers and domain experts is a first-class engineering concern, not a documentation afterthought. The same problem is acute with AI agents: without a glossary, every new session risks coining a new synonym for an existing concept. Terminology drift compounds silently across hundreds of interactions.

CDD makes the domain glossary (`docs/GLOSSARY.md`) a non-negotiable artefact and encodes a rule in `AGENTS.md`: use the exact terms from the glossary. No synonyms.

### Architecture Decision Records — from Nygard (2011)

ADRs capture not just what was decided but why, and what the trade-offs were. Their value is that they make the decision visible to anyone who wasn't in the room — including a future agent session that has no memory of the conversation where the choice was made.

Without ADRs, agents relitigate architectural decisions. They propose changes that were already considered and rejected, because the reasoning lives only in compressed conversation history. `docs/decisions/` is a lightweight ADR log that agents read before starting any non-trivial task.

### Deep Modules — from Ousterhout, *A Philosophy of Software Design* (2018)

Ousterhout's central argument is that the best modules have simple interfaces and deep implementations — complexity is hidden, not spread across call sites. For agentic development, this principle applies to the agent's own behaviour: agents should surface design decisions as questions rather than making assumptions and implementing deeply. The interface (the decision) belongs to the human; the implementation belongs to the agent.

CDD encodes this as the **Design Delegation** principle: the human owns architecture, the agent owns implementation.

### Information Hiding and Separation of Concerns

A well-structured Agent Context Stack separates concerns that belong to different audiences and timescales:

- `AGENTS.md` — non-negotiable working principles and pre-task checklist (session-level)
- `docs/SPEC.md` — architecture and interfaces (module-level)
- `docs/GLOSSARY.md` — domain language (project-level)
- `docs/STATUS.md` — current phase and health metrics (phase-level)
- `docs/decisions/` — resolved design choices (permanent record)

Each file has a single owner, a single purpose, and a clear update cadence. Agents know which file to read for which class of question.

### Single Source of Truth — DRY applied to context (Hunt & Thomas, 1999)

The most dangerous failure mode of a context stack isn't absence — it's staleness. An agent acting on a two-week-old status block is *confidently* wrong, in ways an unbriefed agent is not: it won't ask, because it believes it already knows.

CDD's rule: every volatile fact — test counts, benchmark numbers, current phase — lives in exactly one file (`docs/STATUS.md`), and every other document points to it rather than copying it. STATUS.md carries an "as of" date and stays a one-screen dashboard; dated history moves to `docs/CHANGELOG.md`. A **Phase-End Truth-Up** ritual closes each phase by verifying every factual claim in the stack against reality — run the tests, check the paths, re-run the documented commands.

### Design by Contract (Meyer, 1997) — load-bearing invariants

Most projects have one to three properties their entire credibility rests on: "no simulated result may exceed 120% of the published benchmark", "all money math in integer cents", "read paths never mutate user data". CDD names these **load-bearing invariants** and puts them near the top of `AGENTS.md` — each with the rule, where it is enforced, and what to do on violation.

This is what makes delegation safe. An agent that knows exactly what it must never break can be trusted with far more autonomy everywhere else. The narrower and more explicit the contract, the deeper the delegation.

### Defensive Programming — applied to context

Defensive programming anticipates failure modes and adds guards. CDD applies the same thinking to agent context: assume the context will drift, assume files will grow, assume decisions will be forgotten. Context rots in two ways — **bloat** (files grow past what an agent can usefully load) and **staleness** (claims quietly stop being true) — and the stack ships zero-token shell hooks against both: size budgets on every context file, git-based staleness warnings at session start, a freshness check on STATUS.md's "as of" date, and a dead-pointer check that flags documented paths that no longer exist.

---

## The Agent Context Stack

```
AGENTS.md              # working principles, load-bearing invariants, pre-task checklist,
                       # code style, current phase (pointer only), skill guidance,
                       # security boundaries — read by all agents
CLAUDE.md              # one-line `@AGENTS.md` import (Claude Code auto-loads this filename)
docs/
  SPEC.md              # architecture and key interfaces — read before any non-trivial task
  GLOSSARY.md          # domain terms (ubiquitous language) — no synonyms
  STATUS.md            # one-screen dashboard: phase, test counts, desloppify score
  CHANGELOG.md         # dated session narratives and fix logs — keeps STATUS.md one screen
  BACKLOG.md           # queued work items (or GitHub Issues)
  decisions/           # numbered ADRs — resolved design choices
  PR-REVIEW-CHECKLIST.md  # domain invariants a spawned reviewer checks per PR (optional)
.claude/
  settings.json        # health hooks: bloat guards + staleness, freshness, and
                       # dead-pointer checks (Claude Code)
.cursor/
  rules/*.mdc          # project rules for Cursor Chat/Composer/Bugbot (Cursor only, optional)
```

---

## Why files, not agent memory?

Modern harnesses give agents private persistent memory, which raises a fair question: why maintain documents at all?

Because the two serve different audiences. Agent memory is private and unaudited — no one reviews what an agent remembers, no other agent or tool can read it, and it doesn't survive a change of tooling. The Agent Context Stack is git-versioned, PR-reviewed, and read by *everything*: every agent brand, every teammate, review bots, and CI. For a project worked on by more than one mind — human or artificial — the reviewed, shared artefact wins.

The generated `AGENTS.md` draws the boundary explicitly: memory is a **staging area** for personal preferences and provisional observations; anything shared, durable, or decision-shaping gets promoted into the stack through a normal reviewed change. On conflict, the stack wins — it was reviewed; memory wasn't.

---

## Skills

### `/new-project`

Scaffolds a new project using Context-Driven Development. Asks which agent(s) you use, installs agent-specific tooling, sets up [desloppify](https://github.com/peteromallet/desloppify) (supports 29 languages), and interviews you to produce the full Agent Context Stack.

**Supports:** Claude Code, Cursor, GitHub Copilot, Windsurf, Codex (select multiple)

**What it does:**

1. Asks for your GitHub repo URL
2. Asks which AI agent(s) you're using — gates agent-specific setup on the answer
3. Installs [mattpocock/skills](https://github.com/mattpocock/skills) *(Claude Code only)*
4. Installs desloppify, runs an initial scan to establish a quality baseline, and installs the desloppify workflow guide for each selected agent
5. Asks if GitHub Issues should be the backlog (`"add X to the backlog"` → `gh issue create`)
6. Sets up six project health hooks in `.claude/settings.json` *(Claude Code only)* — zero token cost, shell only. They defend against the two ways context rots: bloat and staleness:
   - Warns at session start (and turn end) if source has been committed since `docs/STATUS.md` was last updated — git-based, so it survives fresh clones
   - Warns at session start if STATUS.md's "as of" date is more than 14 days old
   - Warns if `AGENTS.md` or `CLAUDE.md` exceeds 250 lines
   - Warns if `docs/SPEC.md` or `docs/GLOSSARY.md` exceeds 400 lines
   - Warns if `docs/STATUS.md` exceeds 150 lines (dated history belongs in `docs/CHANGELOG.md`)
   - Warns when `AGENTS.md` or `docs/` references a file path that doesn't exist (dead pointers mis-brief agents)
7. Runs `/setup-matt-pocock-skills` to configure issue tracker and triage labels *(Claude Code only)*
8. Interviews you section by section to fill out `AGENTS.md`:
   - Project description and stack
   - Which working principles to adopt (each with a recommendation)
   - Load-bearing invariants — the 1–3 properties that must never break
   - Which docs the agent reads before every task
   - Code style (formatters, linters, type checking)
   - Current phase (recorded as a pointer to `docs/STATUS.md` — numbers are never duplicated into `AGENTS.md`)
   - Security boundaries
   - Environment notes
   - Multi-agent worktree workflow *(Claude Code only, optional)* — battle-tested rules: PR-before-long-validation, explicit-path staging, append-only shared files, honest deferral
   - Two-layer PR review *(optional)* — a review bot for generic bugs + a domain checklist (`docs/PR-REVIEW-CHECKLIST.md`) run by a spawned read-only agent; generates a `.cursor/rules/` file for Cursor users
9. Writes `## Conversation Strategy`, `## When to Use Skills`, and `## Memory vs the Agent Context Stack` into `AGENTS.md` *(Claude Code only)*, plus a `## Phase-End Truth-Up` checklist for all agents — verify every factual claim in the stack against reality before closing a phase
10. Creates `AGENTS.md` as the primary file; creates `CLAUDE.md` as a one-line `@AGENTS.md` import *(Claude Code only)*
11. Creates all `docs/` stubs and `docs/decisions/` with an ADR template
12. Offers to commit everything
