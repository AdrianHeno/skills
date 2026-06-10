# skills

My personal Claude Code skills, built around the **Context-Driven Development** pattern.

## Install

```bash
npx skills@latest add AdrianHeno/skills
```

## What is Context-Driven Development?

Context-Driven Development (CDD) is the practice of maintaining a structured set of documents — an **Agent Context Stack** — that gives an AI agent durable, phase-aware project context that survives conversation compression and multi-agent handoffs.

It draws on well-established practices applied to AI-assisted development:

- **Ubiquitous Language** (Domain-Driven Design) — a glossary of domain terms that all code, tests, and prompts use exactly, preventing silent terminology drift across sessions
- **Architecture Decision Records** — a numbered log of design decisions so agents don't relitigate resolved choices
- **Deep Modules** (Ousterhout) — simple interfaces, complex internals; surface design decisions as questions rather than assumptions
- **Spec Before Code** — propose interfaces and wait for approval before implementing, eliminating rework
- **Working Agreement** — non-negotiable working principles recorded alongside the code, not in someone's head

The result is a `CLAUDE.md` + `docs/` structure that acts as a persistent briefing for every agent session:

```
CLAUDE.md              # working principles, pre-task checklist, code style, current phase
docs/
  SPEC.md              # architecture and key interfaces
  GLOSSARY.md          # domain terms (ubiquitous language)
  STATUS.md            # current phase, test counts, quality score
  BACKLOG.md           # queued work (or GitHub Issues)
  decisions/           # numbered ADRs
```

## Skills

### `/new-project`

Scaffolds a new project using Context-Driven Development. Installs [mattpocock/skills](https://github.com/mattpocock/skills), sets up [desloppify](https://github.com/peteromallet/desloppify) (supports all languages), and interviews you to produce the full Agent Context Stack.

What it does:
1. Asks for your GitHub repo URL
2. Installs mattpocock/skills via `npx skills@latest add`
3. Installs desloppify and runs an initial scan to establish a quality baseline
4. Asks if GitHub Issues should be the backlog (`"add X to the backlog"` → `gh issue create`)
5. Runs `/setup-matt-pocock-skills` to configure issue tracker and triage labels
6. Interviews you section by section to fill out `CLAUDE.md` — working principles, code style, current phase, environment notes, and more — with a recommendation for each
7. Creates `docs/SPEC.md`, `docs/GLOSSARY.md`, `docs/STATUS.md`, `docs/BACKLOG.md`, and `docs/decisions/` stubs
8. Offers to commit everything
