# 🚀 Claude Code Starter Template

A ready-to-use `.claude/` directory that gives Claude Code a small product
team: a product owner, an architect, a builder, and a reviewer. Drop it
into a project, type one sentence about what you want, approve a one-page
brief, and get back committed, reviewed code.

This is the same toolkit I (Rob Conery) use day to day, with my personal
context stripped out and replaced with templates for you to fill in.

> 🎥 **Companion video series:** this template is what you'll build alongside
> the videos. I'm still working on this.

---

## 🎯 Guiding principle: Design for Change

Every command, agent, and skill here serves one principle: **the goal of
writing software is to be able to change it safely.** SOLID, GoF,
coupling/cohesion, BDD specs, schema conventions are all tactics in service
of that goal. If a rule doesn't make the next change easier, it's the
wrong rule.

## ⚖️ Second principle: spend tokens, not attention

There are two kinds of rigor in a process like this one.

| Kind | Examples | Costs |
|---|---|---|
| Machine rigor | review gate, specs, entry-point trace, smoke test | tokens |
| Human rigor | interviews, handoffs, approvals | your attention |

The first version of this playbook had a lot of both. It could ask you 25
questions before writing a line of code, and I got tired of answering
them. This version keeps all of the machine rigor and cuts the human rigor
down to **one approval per sprint**.

---

## 🧭 What's in the box

```
.claude/
├── commands/   # slash commands (/sprint, the build loop, git helpers)
├── agents/     # product-owner, architect, builder, reviewer
└── skills/     # horizontal knowledge skills, auto-surfaced by description
```

### The pipeline

```
/init ──▶ /sprint "what you want" ──▶ ✋ one approval ──▶ plan ──▶ build ──▶ accept
              │                                              │         │
      product owner + architect                          reviewer   product owner
        (in parallel)                                   gates each   uses it like
                                                           task      a customer

/quick-fix ── small, obvious fixes; skips the pipeline
/document  ── run anytime; reconciles docs ↔ reality
/explore, /design ── optional deep dives when you want to be interviewed
```

`/sprint` runs `/plan` and `/build-loop` for you. You can still run each
one by hand.

| Command | Owns | Question it answers |
|---|---|---|
| `/init` | `CLAUDE.md`, `docs/` skeleton | What files do we need? |
| `/sprint` | `docs/BRIEF.md` | What are we building, for whom, and how? |
| `/plan` | `docs/STORIES.md`, `docs/PLAN.md`, pending specs | What tasks, in what order? |
| `/build-loop` | the code + git history | Build it. |
| `/document` | `README.md`, `docs/MEMORY.md`, ARCHITECTURE prose | Do the docs still match the code? |
| `/quick-fix` | the code | It's a typo. Just fix it. |
| `/explore` | `docs/PROJECT.md` | (optional) What problem, for whom, why? |
| `/design` | `docs/ARCHITECTURE.md`, `docs/SPEC.md` | (optional) How do we build it? |
| `/spec` | spec files | (optional) Specs for stories added later |

### The agents

| Agent | Model | Can edit? | Job |
|---|---|---|---|
| `product-owner` | fable | no | Writes the brief. Adds one to three small extras. Proposes cuts. Accepts or rejects the finished work by running it. |
| `architect` | fable | no | Decides how to build it: seams, data changes, task slice. |
| `builder` | sonnet | yes | Implements one task, runs the tests, reports. Never commits unless told. |
| `reviewer` | fable | no | Read-only gate. Returns `PASS` or `FAIL`. |

Only the builder has edit tools. A gate that can fix its own findings
isn't a gate, and the same goes for a product owner.

### ✨ The product owner and the delight budget

I wanted somebody on the team who would do things I didn't ask for. Not a
lot, just a little: the empty state that tells you what to do next, the
error message that says how to fix it, the form that remembers what you
picked last time.

The problem is that if you tell a model to delight the customer, it builds
a second product. So the product owner works on a budget:

- **One to three extras per sprint.** Zero is fine.
- **Each one is small:** no new dependency, no schema change, no new
  service or route, one task, one commit.
- **Each one is removable.** Nothing depends on it.
- **Each one has a one-line why**, and you can strike it at the approval.

The product owner can also propose **cuts**, and comes back at the end to
run the app as a customer would. Small problems go back through the build
loop as polish tasks. Bigger ideas get written down under "Next".

Builders don't get to freelance. Extras reach the code through the brief
and the plan, and a builder that adds something on its own fails review.

The rules live in `.claude/skills/product-owner/SKILL.md`. Change the
budget there if you want more or less.

### The skills

Skills are markdown files that auto-load when a task description matches.
Some are horizontal ("every programmer needs this"), some are stack-specific
("only if you use Postgres"). See the **Pick your skills** section below.

---

> 🏃 **Just want to start?** See [QUICKSTART.md](./QUICKSTART.md).

## ⚡ Install

From the project you want to use this in:

```bash
# 1. Copy .claude into your project root
cp -r path/to/claude-playbook/.claude /your/project/

# 2. Open Claude Code in that project
cd /your/project && claude
```

That's it. Slash commands and skills are auto-discovered.

**Optional but recommended:**
- Copy `.gitignore` from this repo into your project (or merge it with
  yours).
- Copy `.claude/settings.example.json` → `.claude/settings.json` and keep
  the hooks you want (prettier-on-edit, block `.env` writes, etc.).

---

## 🎯 First-run checklist

Do these **before** your first sprint:

1. **Fill in your personal context.** Open `.claude/skills/you/` and edit the
   three files (`background.md`, `tech.md`, `writing.md`). Be opinionated.
   Strong rules ("Postgres in prod, always") work better than soft
   preferences. `/sprint` guesses in place of asking, and this is what it
   guesses from.

2. **Fill in `design-aesthetic` if you're building UI.** The product owner
   reads it.

3. **Prune skills you won't use.** See the table below. Deleting unused
   skills keeps the AI from getting distracted.

4. **Decide your stack.** The template ships with both `postgres-dba` and
   `sqlite-dev`. Pick one. Same with `typescript-best-practices`: delete it
   if you're a Python shop.

5. **Run `/init`** in your project to scaffold `CLAUDE.md` and `docs/`.

Then run `/sprint` with whatever you want built.

---

## 🧱 Pick your skills

| Skill | Keep if… | Drop if… |
|---|---|---|
| `product-owner` | You want the brief, the extras, and the acceptance pass | You want the pipeline to do only what you typed |
| `solid-principles` | You write OO code in any language | You're doing pure FP / scripts |
| `design-principles` | Always, it's horizontal | Never |
| `gof-patterns` | You write OO code | Pure FP |
| `typescript-best-practices` | You write TS or JS | You don't |
| `postgres-dba` | Postgres in prod | Different DB |
| `sqlite-dev` | Local dev on SQLite with Bun | Different stack |
| `security-web` | You ship anything network-facing | Pure CLI / offline tooling |
| `user-stories` | You want structured backlog grooming | You hate ceremony |
| `bdd-specs` | You want tests-first | You write tests after |
| `agent-teams` | You'll run parallel agents | Solo serial loop is enough |
| `github` | You use git + GitHub | You don't |

---

## 🎨 What ships, what you fill in

Some scaffolds are already here for the most common gaps. Fill them in,
don't re-invent them:

- 🐍 **`lang-template/`**: copy this skill folder and rename it to
  `python-best-practices/` (or Go/Rust/Ruby/Elixir). The frontmatter and
  section headings are pre-filled.
- 🎨 **`design-aesthetic/`**: color tokens, typography, spacing, motion,
  anti-patterns. Edit before your first UI build or the AI defaults to
  generic Tailwind.
- 🧠 **`you/memory-bootstrap.md`**: a starter block to paste into
  `docs/MEMORY.md` after `/init`, so day-one decisions (stack, DB, deploy)
  are written down instead of re-litigated.

Still worth adding yourself when a real project needs them:

- 🗄️ **Database skill** for your actual DB: MySQL, MongoDB, DynamoDB, etc.
- 🔐 **Auth / billing / infra skills** for the services you actually use:
  Stripe, Clerk, Supabase, AWS, Cloudflare, etc.
- 🧪 **Testing skill.** Vitest? Jest? Bun's runner? Playwright? Pick one
  and write down the conventions so the builder agent stops guessing.
- 🚀 **Deploy skill.** Vercel? Workers? Fly? A VPS? The build-loop ends
  with a commit; deploy is a separate concern, but it should have a home.

> 💡 **My suggestion:** start lean. Use it on a real project, notice every
> time the AI guesses wrong, and write a skill that prevents that specific
> wrong guess. Don't try to author skills speculatively. They get stale and
> you stop trusting them.

---

## 🛠️ Customizing

### Adding a slash command

Drop a markdown file in `.claude/commands/`. The filename becomes the
command name (`docs.md` → `/docs`). Frontmatter:

```yaml
---
description: One-liner that shows up in the picker
---
```

Then write the prompt. Be explicit about: what the command owns, what it
reads, what stops it (missing inputs), and what it produces.

### Adding a skill

Skills live in `.claude/skills/<name>/SKILL.md`. The `description` in
frontmatter is **everything**. It's how Claude decides whether to surface
the skill. Be specific about *when* to use it ("when reviewing Postgres
schemas" beats "Postgres stuff").

### Adding an agent

Agents live in `.claude/agents/<name>.md`. Used for sub-tasks the main
session shouldn't be doing itself: long research, dedicated review,
parallel work. The `builder`/`reviewer` pair here is a template for the
loop pattern. Copy it if you want, say, a `migration-writer` + `db-reviewer`
pair.

---

## 🧩 Conventions worth keeping

- **One gate per sprint.** You approve the brief. After that, the next
  thing you look at is finished work.
- **Assume, then show.** Agents write their guesses down as
  `> ASSUMPTION:` lines in place of asking. They only ask when a wrong
  guess is expensive to undo (money, auth, deleting data, the deploy
  target), and they get three questions per sprint.
- **One owner per document.** Every doc has exactly one command that writes
  it. Other commands can add to it, and they don't rewrite it.
- **Re-entrant phases.** Re-running `/plan` should refine, not restart. Never
  drop completed `- [x]` tasks.
- **Extras never block.** A `delight:` task that can't pass review gets
  skipped and logged. The thing you asked for still ships.
- **Model choice is deliberate.** Two tiers. Where the thinking decides
  the outcome (the brief, the architecture, the plan, the review gate) →
  `fable`, the frontier model. Everything else → `sonnet`, and nothing
  runs below it. The `model:` lines use aliases, so they follow the
  current model in each family. If your plan doesn't include `fable`,
  change those lines to `opus`.

---

## 🤔 What you might be missing (a quick gut-check)

Things this template **doesn't** decide for you, that you'll want to think
about:

- 🔑 **Auth model.** Single-tenant? Multi-tenant? Sessions? JWTs? Magic links?
- 💳 **Billing model.** One-time? Subscription? Usage-based?
- 🌍 **Where does state live?** Local SQLite for the demo, Postgres in prod,
  KV for sessions. Write it down in `you/tech.md` so the AI stops guessing.
- 📊 **Observability.** Logs, metrics, traces. Nothing here sets this
  up. Worth a skill or a `/observability` command.
- 🚦 **CI.** The build-loop runs tests locally and commits on PASS. CI is
  separate and not opinionated here. Add a `.github/workflows/` skill if you
  want one.
- 🎬 **The first 5 minutes of a project.** If `/sprint` feels heavy for a
  one-file experiment, that's what `/quick-fix` is for.

---

## 📖 Further reading

The deep dive on each command, agent role, and the build-loop orchestration
pattern lives in `.claude/README.md`. Read that next.
