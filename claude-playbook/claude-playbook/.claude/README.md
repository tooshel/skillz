# Personal Claude Code Toolkit

A portable `.claude/` configuration that gives Claude Code a small product
team and a pipeline to run it. Drop it into a project and one command takes
a sentence to committed, reviewed code, with one approval from you in the
middle.

There is no application code here. This repository **is** the `.claude/`
directory: commands, agents, and skills. You use it *from inside other
projects*.

## 🎯 Guiding principle: Design for Change

Every command, agent, and skill in this collection serves one principle:
**the goal of writing software is to be able to change it safely.** SOLID,
the GoF patterns, coupling/cohesion, BDD specs, schema conventions are
tactics in service of that goal.

Concretely, every artifact this toolkit produces should optimize for:

- Low coupling, high cohesion. One reason to change per module (SRP).
- Open to extension, closed to modification. Stable seams behind interfaces.
- Localized blast radius. A change in one place doesn't ripple.
- Composition over shared mutable state.
- Names that telegraph intent so the next person finds the seam.
- A small diff for the next change.

If a rule in here doesn't make the next change easier, it's the wrong rule.

## ⚖️ Second principle: spend tokens, not attention

Rigor that costs tokens (review gates, specs, entry-point traces, smoke
tests) stays. Rigor that costs the human's attention (interviews,
handoffs, approvals) is cut to one approval per sprint. Agents guess, write
the guess down, and let the human correct it.

## Layout

```
.claude/
├── commands/   # /sprint, the phases it runs, git helpers
├── agents/     # product-owner, architect, builder, reviewer
└── skills/     # knowledge + workflow skills, surfaced by description
```

## Commands

```
/init ──▶ /sprint ──▶ ✋ approve the brief ──▶ /plan ──▶ /build-loop ──▶ acceptance
                                              └────── run by /sprint ──────┘

/quick-fix ── trivial fix, no pipeline
/document  ── run anytime; reconciles docs ↔ reality
/explore, /design, /spec ── optional, run by hand
```

### The main path

| Command | Model | Question it answers | Owns |
|---|---|---|---|
| `/init` | sonnet | What files do we need? | `CLAUDE.md`, `docs/` skeleton, `README.md` stub, `.gitignore` |
| `/sprint` | fable | What are we building, for whom, and how? | `docs/BRIEF.md` (old briefs move to `docs/briefs/`) |
| `/plan` | fable | In what order, as what tasks? | `docs/STORIES.md`, `docs/PLAN.md`, pending specs |
| `/build-loop` | sonnet | Build it. | the code + git history |
| `/document` | sonnet | Do the docs still match reality? | `README.md`, `docs/ARCHITECTURE.md` prose, `docs/MEMORY.md` |
| `/quick-fix` | sonnet | Trivial fix on the current branch. | the code |

### Optional deep dives

| Command | Model | When to use it | Owns |
|---|---|---|---|
| `/explore` | fable | The idea is fuzzy and you want to be interviewed about it. | `docs/PROJECT.md` |
| `/design` | fable | Greenfield or risky, and you want to make each architecture call yourself. | `docs/ARCHITECTURE.md`, `docs/SPEC.md` |
| `/spec` | sonnet | Stories were added after `/plan` ran and need specs. | spec files |

If `PROJECT.md` or `SPEC.md` exist, `/sprint` reads them and builds on
them.

`docs/MEMORY.md` is the project's decision log. Every command *appends*
dated entries; `/document` curates it. It is distinct from the `~/.claude`
memory system.

### Git helpers

Small commands that defer to the `github` skill.

| Command | Model | Purpose |
|---|---|---|
| `/git-commit` | sonnet | Stage and commit using Conventional Commits; branch-or-trunk by size. |
| `/git-issue` | sonnet | Open a GitHub issue via `gh issue create`. |
| `/git-pr` | sonnet | Open a PR with summary, verification plan, risk note. |
| `/git-merge` | sonnet | Merge current branch into `main` safely: checks, confirms, merges. |
| `/git-remote` | sonnet | Stand up the GitHub remote (name, license, README, contributing, security, issue templates). |

## Agents

| Agent | Model | Tools | Role |
|---|---|---|---|
| `product-owner` | fable | Read, Glob, Grep, Bash, Skill, WebSearch, WebFetch | Writes the product half of the brief. Adds extras inside the delight budget. Proposes cuts. Runs the finished work and returns `ACCEPT`, `POLISH`, or `REJECT`. |
| `architect` | fable | Read, Glob, Grep, Bash, Skill | Writes the technical half of the brief: approach, seams, data changes, decisions, task slice. |
| `builder` | sonnet | Read, Write, Edit, Glob, Grep, Bash, Skill | Implements one PLAN.md task. Stays in its files. Never commits unless told. |
| `reviewer` | fable | Read, Glob, Grep, Bash, Skill | Read-only gate. Returns `PASS` or `FAIL` with findings. |

Only the builder can edit. Everyone else reports.

## How a sprint runs

```
  /sprint "add invoicing with PDF export"
     │
     ▼
  1. THINK    product-owner ─┐  dispatched together,
              architect ─────┘  both get your exact words
     │
     ▼
  2. MERGE    lead builds a one-page brief. Every extra is checked
              against the delight budget. Failures move to "Next".
     │
     ▼
  3. GATE ✋   "Here's what we think." You approve, strike extras,
              correct assumptions, answer up to 3 questions.
     │
     ▼
  4. WRITE    docs/BRIEF.md, ARCHITECTURE.md additions, MEMORY.md
     │
     ▼
  5. PLAN     stories, PLAN.md (extras tagged delight:), pending specs
     │
     ▼
  6. BUILD    the build loop, below
     │
     ▼
  7. ACCEPT   product-owner runs the app as a customer
```

### The delight budget

The product owner adds things you didn't ask for. The budget keeps that
small:

- one to three extras per sprint, zero allowed
- no new dependency, no schema change, no new service or route
- one task, one commit
- removable: nothing depends on it
- a one-line why, written from the customer's side
- extras are at most about 15% of the sprint's tasks

Anything over budget goes under **Next** in the brief. The full rules are
in `skills/product-owner/SKILL.md`.

## The build loop

`/build-loop [path-to-PLAN.md]` (default `docs/PLAN.md`) is the only
command that writes feature code. It runs in the lead thread and
dispatches subagents. It never writes or reviews code itself.

**Preflight:** resolve the plan, confirm it is a checkbox list (`- [ ]`),
confirm a clean git working tree. Stop and report if any of these fail.

**The loop**, for each unchecked task, top to bottom in dependency order:

```
  ┌─────────────────────────────────────────────────────┐
  │  task: - [ ]                                         │
  │     │                                                │
  │     ▼                                                │
  │  1. BUILD   → builder subagent (sonnet)               │
  │              activates the task's pending specs,      │
  │              implements, runs the tests, reports.     │
  │              Does NOT commit.                         │
  │     │                                                │
  │     ▼                                                │
  │  2. REVIEW  → reviewer subagent (fable)               │
  │              read-only; security-web + simplify +     │
  │              scope check; returns PASS or FAIL.       │
  │     │                                                │
  │     ├── FAIL ─▶ findings → fresh builder ─┐ (recurse) │
  │     │          ◀───────────────────────────          │
  │     ▼                                                │
  │  3. SMOKE   → one real request through the deployed   │
  │              entry point.                             │
  │     │                                                │
  │     ▼                                                │
  │  4. COMMIT  → builder commits ONLY this task's diff.  │
  │     │                                                │
  │     ▼                                                │
  │  5. CHECK   → flip - [ ] to - [x] in PLAN.md, commit. │
  └─────────────────────────────────────────────────────┘
            ▼  next task
```

**Extras never block.** A `delight:` or `polish:` task that fails review
three times is reverted, marked `- [-]`, logged, and skipped.

**Acceptance.** When every box is checked, the product owner runs the real
app and goes through it as a customer:

| Verdict | What happens |
|---|---|
| `ACCEPT` | Done. Report. |
| `POLISH` | Up to five small items go back through the loop as `polish:` tasks. One round only. |
| `REJECT` | A line in the brief didn't happen. A fix task runs, then acceptance runs once more. |

**Role separation is strict.** Builder owns code; reviewer owns the gate;
product owner owns acceptance; the loop owns sequencing and checkbox
state. Nothing reaches git before a `PASS`.

**This loop is a pipeline, not a parallel team.** Sequential,
dependency-ordered subagents via the Agent tool, *not* Agent Teams. For
parallel multi-agent work, see the `agent-teams` skill.

## Skills

Skills under `.claude/skills/` are auto-surfaced by description; commands and
agents also invoke them explicitly via the Skill tool.

### Product

- `product-owner`: assume-don't-interview, the delight budget, cutting, the acceptance pass, the `BRIEF.md` template
- `design-aesthetic`: your visual identity. A template, fill it in before your first UI build

### Language & design

- `typescript-best-practices`: strict TS, type modeling, Result errors, `toString`/`toJSON` rule
- `solid-principles`: the five SOLID principles, with idiomatic TS examples
- `design-principles`: coupling/cohesion, DRY/YAGNI/KISS, Demeter, Tell Don't Ask, CQS, fail-fast
- `gof-patterns`: all 23 Gang of Four patterns in TypeScript
- `lang-template`: a starting point for your own language skill

### Data

- `postgres-dba`: snake_case, surrogate keys, NOT NULL FKs, enums, JSONB, plpgsql
- `sqlite-dev`: Bun + Drizzle on SQLite, designed to port cleanly to Postgres

### Security

- `security-web`: XSS/CSRF/injection/SSRF/auth review for Next.js / Nuxt / Bun / Hono

### Process & specs

- `user-stories`: agile stories + Given/When/Then acceptance criteria → `docs/STORIES.md`
- `bdd-specs`: executable specs from STORIES.md (Feature > Scenario > Specification), pending until a builder activates them
- `agent-teams`: orchestrate parallel agent teammates (the multi-agent counterpart to `/build-loop`)

### Workflow

- `github`: Conventional Commits, branch-or-trunk, `gh` issues/PRs

### Personal context

- `you`: your background, tech preferences, and writing voice. `/sprint` guesses from this, so fill it in

Each skill is a directory with a `SKILL.md` whose frontmatter `name` +
`description` control when it triggers, plus optional `references/` and
`templates/`.

## Conventions

- **One gate per sprint.** The human approves the brief. After that, an
  agent that wants to ask something makes the call, logs it in
  `docs/MEMORY.md`, and keeps going. The exception is anything destructive
  or expensive to undo.
- **Assume, then show.** Guesses are written as `> ASSUMPTION:` lines.
  Three questions per sprint, only for money, auth, deleting data, the
  deploy target, or anything public.
- **One owner per document.** Every stub names its owning command at the
  top. Other commands may add to a doc. They don't rewrite it.
- **Re-entrant phases.** Re-running a command refines existing output and
  preserves completed work (e.g. `/plan` never drops `- [x]` tasks).
- **Builders don't freelance.** Extras reach the code through the brief
  and the plan.
- **Stop for real blockers only.** A task that can't pass review, a smoke
  test that won't go green, a missing secret.
- **Model choice is deliberate.** Two tiers. Where the thinking decides
  the outcome → `fable`, the frontier model (`/sprint`, `/plan`,
  `/explore`, `/design`, product-owner, architect, reviewer). Everything
  else → `sonnet`, git helpers included. Nothing runs below `sonnet`.
  Aliases, not version-pinned IDs. No `fable` on your plan? Change those
  lines to `opus`.

## Usage

Copy `.claude/` into a project, then from Claude Code in that project:

```
/init my-project                        # scaffold CLAUDE.md + docs/
/sprint add invoicing with PDF export   # brief → approve → plan → build → accept
/document                               # whenever docs drift from the code
```
