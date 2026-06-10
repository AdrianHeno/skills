# skills

My personal Claude Code skills.

## Install

```bash
npx skills@latest add AdrianHeno/skills
```

## Skills

### `/new-project`

Scaffolds a new project for Claude collaboration. Installs [mattpocock/skills](https://github.com/mattpocock/skills), sets up [desloppify](https://github.com/peteromallet/desloppify), and interviews you to produce `CLAUDE.md` and `docs/` stubs — following the same pattern I use across my projects.

What it does:
1. Asks for your GitHub repo URL
2. Installs mattpocock/skills via `npx skills@latest add`
3. Installs desloppify and runs an initial scan (supports all languages)
4. Asks if GitHub Issues should be the backlog (`"add X to the backlog"` → `gh issue create`)
5. Runs `/setup-matt-pocock-skills` to configure issue tracker and triage labels
6. Interviews you section by section to fill out `CLAUDE.md` — working principles, code style, current phase, environment notes, and more — with a recommendation for each based on proven patterns
7. Creates `docs/SPEC.md`, `docs/GLOSSARY.md`, `docs/STATUS.md`, `docs/BACKLOG.md`, and `docs/decisions/` stubs
8. Offers to commit everything
