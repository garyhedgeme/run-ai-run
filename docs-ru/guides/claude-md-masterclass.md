
# CLAUDE.md as Configuration, Not Documentation

> Most teams treat CLAUDE.md like a README. It is closer to a system prompt — and that one reframe changes how you write every line of it.

Every Claude Code session starts cold. The agent does not know your stack, your folder layout, or that your team prefers tabs and a particular branch-naming convention. Without a standing brief you re-explain the project every single session — *"this is a FastAPI service, SQLAlchemy for the DB, tests in `tests/`, run pytest, PEP 8 at 100 columns"* — over and over.

`CLAUDE.md` is the standing brief. Write it once; Claude reads it automatically at the start of every session. The rest of this guide is about writing it well, because the difference between a good one and a bloated one is large.

## The reframe that matters

A `CLAUDE.md` is **not documentation**. It becomes part of Claude's **system prompt** — and the agent follows system-level instructions more strictly than anything you type into the chat. That is the whole game: it sets the operating boundaries for every conversation in the project.

There is a catch baked into the loader. Claude Code wraps your file in a reminder that says, in effect, *"this context may or may not be relevant; ignore it unless it applies to the task."* So if you stuff the file with instructions that are not universally relevant, the agent learns to discount the file — including the parts that *do* matter.

That is why the operating principle is **less is more**. Every unnecessary line you add lowers the odds that the important lines are followed.

A useful mental model — `CLAUDE.md` is three things at once:

- **Project memory** — what carries across sessions (stack, layout, commands).
- **Operating boundaries** — the rules the agent should not cross.
- **Context primer** — enough that the agent starts informed instead of blank.

The moment you hold that model, you stop writing a README and start writing the highest-leverage configuration point you have.

## The hierarchy

`CLAUDE.md` files load in a defined order, and you use that order deliberately:

- **Global** (`~/.claude/CLAUDE.md`) — rules that apply to *every* project you touch.
- **Project** (`<repo>/CLAUDE.md`) — rules for this repository.
- **Nested** (`<repo>/apps/web/CLAUDE.md`, …) — rules that load **only** when Claude opens files in that subtree.

In a monorepo, nested files keep the main context lean: the frontend conventions load only when the agent is actually in the frontend. There are two filename variants worth knowing — committed `CLAUDE.md` (shared with the team) and gitignored `CLAUDE.local.md` (your personal overrides that should never hit the repo).

## Anatomy: WHAT, WHY, HOW

A well-structured file answers three questions and little else:

- **WHAT** — tech stack, project structure, the handful of key files.
- **WHY** — what the project is for and what each part does.
- **HOW** — the commands to run, the workflows to follow, the conventions that cause bugs when missed.

A compact skeleton:

```markdown
# <one line on what this project is and your working philosophy>

## About This Project
Stack, architecture pattern, key dependencies.

## Key Directories
- `src/` — source
- `tests/` — tests
- `docs/` — documentation

## Commands
```bash
npm run dev    # dev server
npm run test   # tests
npm run build  # production build
```

## Standards
Conventions that apply everywhere; testing requirements; commit format.

## Notes
Gotchas and warnings the agent should know.
```

Keep each section focused. When a section grows long, that is the signal to move it into its own file and reference it — not to keep expanding the brief.

## The instruction budget

Here is the constraint most people miss. Frontier models reliably follow roughly **150–200 instructions**; past that, instruction-following degrades — and not just for the newcomers, but **uniformly across all of them**. Claude Code's own system prompt already spends on the order of ~50 instructions. That leaves you a working budget of roughly **100–150** before the agent starts dropping things on the floor.

So treat instructions as a scarce resource and spend them on what earns its place:

**Always include**
- Project overview (1–2 sentences)
- Stack and key dependencies
- Essential commands (build, test, run)
- Directory structure (key folders only)
- Conventions that cause bugs if missed

**Include only if universally applicable**
- Branch-naming and commit-message conventions
- Testing requirements; deployment process

**Never include**
- Secrets (keys, credentials, connection strings)
- Detailed code-style rules — a linter does this better
- Task-specific instructions — those belong in their own files
- Anything the agent could learn by reading the code

**Move to separate files**
- Database schema details, complex procedures, architecture deep-dives, onboarding

A simple test: if an instruction is not relevant to ~80% of your sessions, it does not belong in `CLAUDE.md`. Move it to a referenced file or a custom command.

## Progressive disclosure

Rather than inlining everything, point to it:

```markdown
## Additional Documentation
Before a specific task, read the relevant file:
- Building: `agent_docs/building_the_project.md`
- Testing: `agent_docs/running_tests.md`
- Database work: `agent_docs/database_schema.md`
Read only what is relevant to the current task.
```

Claude loads those files on demand, so the main context stays lean and the deep material is there when — and only when — it is needed.

And do not hand-write a code-style section. Fifty lines of "use 2 spaces, prefer const, trailing commas…" is exactly the kind of bloat that dilutes the file. Use the real tools — ESLint/Prettier, Black/Ruff, rustfmt — and wire them to a pre-commit or a Claude Code hook so they run automatically. The file states the rule exists; the hook enforces it.

## Three ways to create one

- **`/init`** — Claude scans the codebase and generates a baseline (build commands, tests, key directories, detected conventions). Treat the output as a *starting point*, never a finished file — it catches the obvious and misses your nuances. Review and trim.
- **Manual** — `touch CLAUDE.md` and write it yourself when you want full control or are following a template.
- **The `#` shortcut** — while working, type `#` followed by an instruction to append it on the fly. This is how you capture a rule the moment you notice the agent keeps missing it.

The best files are built iteratively: start from `/init`, refine by hand, and grow it with `#` as you discover what actually matters.

## Patterns once you are past the basics

- **Index files** for large or unfamiliar codebases — a map of modules and key functions. Add the line *"these indexes may or may not be up to date; verify against the actual files"* so the agent treats the map as a hint, not gospel.
- **Section boundaries** — clear markdown headers (Commands, Standards, Workflows, File Boundaries, Tooling) stop instructions in one area from bleeding into another.
- **Explicit workflows** — number the steps for recurring tasks (adding an endpoint, a schema migration) so the agent plans before it codes.
- **Variant files** — keep `CLAUDE.development.md` / `CLAUDE.deployment.md` under `.claude/` and swap the active one when focus changes.
- **Conditional rules** — *"when working in `src/api/` … when working in `tests/` … when working in `scripts/` …"* keep behavior tight to context.
- **Document your MCP servers** — when to use each, and the limits (read-only replica, rate caps), so the agent uses them correctly.

## Where it compounds: hooks and subagents

`CLAUDE.md` is strong alone and becomes a system when paired with the rest of Claude Code:

- **CLAUDE.md** defines the rules.
- **Hooks** enforce them automatically — linting, formatting, secret checks — so the agent does not spend tokens (or your patience) checking by hand.
- **Subagents** run specialized tasks in their own context window, and your file lets them get oriented fast without inheriting the whole conversation.

A short `## Subagent Guidelines` section — *"security reviews use a fresh subagent; exploration reads the index first; docs work may touch `docs/` freely"* — pays for itself.

## Keep it alive

A `CLAUDE.md` is not set-and-forget. Projects change and patterns improve, so:

- Add **"CLAUDE.md updated if workflows changed"** to your PR checklist.
- Use the **`#` shortcut** the instant you spot a recurring miss.
- **Review quarterly** — are the commands still accurate, any workflow stale, anything you can delete?
- **Commit it** — the team inherits every improvement, and you get history.

## Conclusion

Get `CLAUDE.md` right and every other feature — hooks, subagents, MCP — works better on top of it. The takeaways fit on a sticky note:

- It is configuration, not documentation; the agent treats it as system-level rules.
- Less is more — past ~100–150 instructions you are actively making things worse.
- Use the hierarchy: global → project → nested.
- Prefer progressive disclosure over one bloated file.
- Iterate continuously with `#`.

## CTA

This guide lives in [run-ai-run](https://github.com/garyhedgeme/run-ai-run) — a public, scrubbed mirror of runbooks, guides, and small apps for working with Claude Code. Browse the rest there, or read it on the web via GitHub Pages.

## Sources

- Original article: Joe Njenga, *"CLAUDE.md Masterclass: With Hooks, Subagents & Pro Automation"*, Claude Code Masterclass newsletter — https://open.substack.com/pub/claudecodemasterclass/p/claudemd-masterclass-from-start-to (published 2026-01-28). This guide is an independent distillation of that article's principles, in my own words.
- Surfaced via [@samuel_wong_](https://x.com/samuel_wong_/status/2027908993818235078) on X.
- Anthropic — Claude Code documentation (CLAUDE.md, hooks, subagents).
