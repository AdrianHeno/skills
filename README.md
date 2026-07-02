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

These aren't model failures. They're information failures. The agent is capable — it just doesn't have what it needs.

---

## What is Context-Driven Development?

Context-Driven Development (CDD) is the practice of maintaining a structured set of documents — an **Agent Context Stack** — that gives an AI agent durable, phase-aware project context that survives conversation compression, multi-agent handoffs, and new sessions.

It's not a new idea. It applies well-established computer science and software engineering principles to the specific constraints of agentic development:

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

### Defensive Programming — applied to context

Defensive programming anticipates failure modes and adds guards. CDD applies the same thinking to agent context: assume the context will drift, assume files will grow, assume decisions will be forgotten. The Agent Context Stack is a set of guards against those failure modes, backed by automated hooks that catch drift early — before it becomes expensive to fix.

---

## The Agent Context Stack

```
AGENTS.md              # working principles, load-bearing invariants, pre-task checklist,
                       # code style, current phase (pointer only), skill guidance,
                       # security boundaries — read by all agents
CLAUDE.md              # symlink to AGENTS.md (Claude Code auto-loads this filename)
docs/
  SPEC.md              # architecture and key interfaces — read before any non-trivial task
  GLOSSARY.md          # domain terms (ubiquitous language) — no synonyms
  STATUS.md            # one-screen dashboard: phase, test counts, desloppify score
  CHANGELOG.md         # dated session narratives and fix logs — keeps STATUS.md one screen
  BACKLOG.md           # queued work items (or GitHub Issues)
  decisions/           # numbered ADRs — resolved design choices
  PR-REVIEW-CHECKLIST.md  # domain invariants a spawned reviewer checks per PR (optional)
.claude/
  settings.json        # health hooks: size guards + STATUS.md staleness check (Claude Code)
.cursor/
  rules/*.mdc          # project rules for Cursor Chat/Composer/Bugbot (Cursor only, optional)
```

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
6. Sets up four project health hooks in `.claude/settings.json` *(Claude Code only)* — zero token cost, shell only:
   - Warns if `AGENTS.md` or `CLAUDE.md` exceeds 250 lines
   - Warns if `docs/SPEC.md` or `docs/GLOSSARY.md` exceeds 400 lines
   - Warns if `docs/STATUS.md` exceeds 150 lines (dated history belongs in `docs/CHANGELOG.md`)
   - Warns at end of each session if source files are newer than `docs/STATUS.md`
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
9. Writes `## Conversation Strategy` and `## When to Use Skills` into `AGENTS.md` *(Claude Code only)*
10. Creates `AGENTS.md` as the primary file; creates `CLAUDE.md` as a symlink to it *(Claude Code only)*
11. Creates all `docs/` stubs and `docs/decisions/` with an ADR template
12. Offers to commit everything
