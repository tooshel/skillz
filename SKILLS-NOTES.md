# Skills Repos: Comparison & Recommendation

*Compiled 2026-08-15 from the three repos cloned in this directory. **Updated 2026-09-28**: re-cloned all three, re-measured, and added Rob Conery's `claude-playbook` (now in `./claude-playbook/`), which the first pass only described secondhand. See [What changed since 2026-08-15](#what-changed-since-2026-08-15).*

## TL;DR

You're not missing much by running naked — but you're conflating two different costs. `CLAUDE.md` / `AGENTS.md` load **eagerly** into every session whether relevant or not; skills load **lazily** (only their one-line descriptions sit in context until one triggers). A user-invoked skill (slash command) costs approximately nothing until you call it. So "skills get in the way" is mostly true of always-on instruction files and of skill packs with aggressive auto-trigger descriptions — not of the mechanism itself.

The naked harness is genuinely good in 2026. Skills earn their keep in exactly three situations:

1. **A workflow you re-prompt often** (interview me before coding, review this PR my way, write a spec first) — a skill is just a saved prompt with a trigger.
2. **Long autonomous runs** where the agent drifts, declares half-done work finished, or rationalizes skipping steps — process-discipline skills are countermeasures for that.
3. **Team consistency** — everyone's agent follows the same review/ship process.

If none of those describe your week, keep running naked and cherry-pick later.

---

## The four collections

### Matt Pocock — `mattskills/` (mattpocock/skills)

- **38 skills** (25 shipped in the plugin — same list as August; the 3 new ones sit in `skills/in-progress/`), median **~75 lines**, some as short as 7 lines. ~2,600 lines of `SKILL.md` (~4,100 counting every `.md` under `skills/`).
- Philosophy: "Skills for real engineers, not vibe coding." Small, composable primitives — thin orchestrator skills that just call two other skills, plus dense reference docs (`tdd`, `diagnosing-bugs`). Explicitly positioned *against* big frameworks (GSD, BMAD, Spec-Kit) that "take away your control."
- **Mostly user-invoked** — a formal invariant (`.agents/invocation.md`) marks which skills can auto-trigger and which only fire when you ask. 14 of the 25 shipped skills are user-invoked. This is the pack least likely to "get in the way."
- Not TypeScript-specific despite the author (2 of 38 skills). Clusters: requirements grilling, spec→ticket→implement pipeline, TDD, debugging, domain modeling, code review.
- Standouts: `grilling` (requirements interrogation — note it asks in **numbered rounds** with a recommended answer per question, not strictly one question at a time; that changed in July), `diagnosing-bugs` (six gated phases, "no red-capable command, no Phase 2"), `writing-for-agents` (a meta-skill that teaches how to write skills — context load vs. cognitive load, progressive disclosure, anti-negation).
- Install: official Claude Code plugin marketplace (`claude plugins install mattpocock-skills`) or `npx skills@latest add mattpocock/skills` for editable copies. Active (472 commits, ~90% one author plus AI-agent co-authors), but **no plugin release since v1.2.3 on 2026-08-06** — 12 changesets are queued.
- Since August: quiet. Three unreleased in-progress skills (`implement-spec`, `pr`, `retro`), a repo-wide em-dash purge, otherwise wording. The skills recommended below are unchanged.

### Drew Ewing — `drewskills/` (aewing/skills)

- **12 skills** (was 10), ~2,100 lines of `SKILL.md` (~5,100 counting references and scripts). Bimodal: five at ~60 lines, seven at 140–460.
- Theme: "rigor with calm" — stop "looks done" from passing as done (`execution-rigor`, 10 named phases with a forbidden-shortcut list and grep commands to detect them) and stop "plausible" from passing as true (`truth-loop`). Includes adversarial subagent phases where a cold-context reviewer tries to refute the work.
- Personality outliers: `roast-review` (evidence-required roast of your codebase), `wat` (ADHD-friendly answer-first response format), `tui-design`.
- **New since August** — the scope has widened beyond agent rigor:
  - `focus-group` (08-21): synthetic user panels for product review, with a 900-line Python validator — the repo's first real executable code.
  - `headless-lanes` (09-28, today): one overseer session driving other agent CLIs in parallel lanes.
- Unusual: `execution-rigor` will *refuse* a mid-task "just make it work" shortcut request, on the grounds you invoked the skill to be defended from yourself. The repo also audits its own prose with its own `goodtalk` skill and publishes the audit ledger.
- Install: plugin marketplace (now v1.3.0) or a clean ~76-line `install.sh` that copies into `~/.claude/skills`. Single author, 14 commits across three days (08-15, 08-21, 09-28) — still the newest and least battle-tested, but no longer a one-day drop.

### Addy Osmani — `addyskills/` (addyosmani/agent-skills)

- **25 skills** organized by SDLC phase (Define → Plan → Build → Verify → Review → Ship), ~7,500 lines of `SKILL.md` (~8,400 with skill-local references) plus 4 agent personas, 9 slash commands (triplicated for Claude/Gemini/Antigravity), hooks, shared reference checklists, and — rare for skill packs — a **CI-gated eval harness** with adversarial "pressure cases" (routing floor raised from 80% to 95% since August).
- Philosophy: production engineering discipline, heavily sourced from *Software Engineering at Google* (Hyrum's Law, Beyoncé Rule, trunk-based dev, ~100-line changes). Nearly every skill has an anti-rationalization table rebutting the excuses an agent makes to skip steps.
- Web/frontend/TS tilt: Core Web Vitals, Chrome DevTools MCP browser testing, WCAG, a `/webperf` command — somewhat diluted since August by backend/DB depth in `performance-optimization`, SLO release gates, and runbooks.
- **Biggest change since August: the plugin no longer injects the `using-agent-skills` meta-skill via SessionStart hook** (removed 2026-09-15, because on Claude Code it's a second router on top of the native one). That cut its idle cost roughly in half; it's now in the same range as Rob's pack.
- New skill: `constraint-driven-development` — writes your quality bar to `CONSTRAINTS.md` and flags diffs that lower it (suppressions, skipped tests), with a matching `/constraints` command.
- Standouts: `doubt-driven-development` (adversarial fresh-context self-review, optionally escalating to Gemini/Codex as a second model), `interview-me`, the `simplify-ignore` hook that physically hides protected code from the model, and an honest `docs/comparison.md` sizing itself against competitors — including Matt's pack.
- Install: nine+ tool targets (Copilot CLI added). Most active and most multi-maintainer repo: 590 commits, 286 in the last 90 days from 46 authors. Still the heaviest — 25 model-invoked skill descriptions is the largest trigger surface by skill count.

### Rob Conery — `claude-playbook/` (zip, not a git repo)

The first pass described this as "build-your-own tooling." Having now read it: **it's a complete, opinionated `.claude/` directory you copy into each project**, with a build-your-own ethos layered on top ("start lean… notice every time the AI guesses wrong, and write a skill that prevents that specific wrong guess").

- **15 skills, 4 agents, 14 slash commands.** ~1,900 lines of `SKILL.md`, median ~131 lines; ~8,800 lines total with references and templates (TS, SQL, Drizzle).
- **It's a pipeline, not a library.** `/sprint "add invoicing"` dispatches a `product-owner` and an `architect` agent in parallel → you approve a one-page brief (**the only human gate**) → `/plan` writes stories, `PLAN.md`, and pending BDD specs → `/build-loop` runs builder (sonnet) → read-only reviewer (fable) → smoke test through the real entry point → one commit per task → the product owner runs the finished app "as a customer" and returns `ACCEPT` / `POLISH` / `REJECT`. `/quick-fix` bypasses all of it for small stuff.
- Philosophy: "**spend tokens, not attention.**" Machine rigor (review gates, specs, smoke tests) stays; human rigor (interviews, handoffs) is cut to one approval. Agents write guesses as `> ASSUMPTION:` lines and get **three questions per sprint, max**. Rob says v1 "could ask you 25 questions before writing a line of code, and I got tired of answering them" — a direct counterpoint to Matt's `grill-me` approach.
- The **delight budget**: the product owner must add 1–3 small unrequested extras per sprint (empty states, better error messages, remembered defaults), each removable, no new deps/schema/routes, ≤15% of tasks. Builders who add anything unrequested fail review.
- The **reviewer agent** is the most transplantable idea in the pack: read-only, and it fails a task if a hand-trace from the deployed entry point doesn't reach the promised side effect, if ghost code (`as any`, `{} as Env`, `TODO`, deprecated no-ops) is on the production path, or if runtime-specific APIs (`Bun.*`, `node:fs`) ship to a different runtime. That targets "tests green, feature not actually wired in" — a failure mode none of the other three packs address directly.
- **Heavily opinionated stack**: builder and reviewer are hard-coded to TypeScript (Next.js or Bun); skills for SOLID, GoF, TS best practices, Postgres (plpgsql business logic, snake_case) and SQLite+Drizzle; Conventional Commits. You're expected to prune what doesn't fit and fill in the `you/` (background, tech rules, writing voice) and `design-aesthetic` templates.
- Gotchas:
  - `/init` writes a CLAUDE.md with Rob's always-on rules ("DO NOT SEARCH node_modules… GO ONLINE", "Tokens are water, we're in the desert", emoji in markdown) — eager context of exactly the kind you said gets in the way.
  - Its `/init` shares a name with Claude Code's built-in `/init`.
  - Agents and planning commands use `model: fable`; swap to `opus` if your plan lacks it.
  - `/build-loop` retries build→review with **no cap** on required tasks.
  - The `you` and `design-aesthetic` skills auto-surface even while still full of `[bracketed placeholders]`.
- Install: `cp -r .claude` into a project; uninstall is deleting it. Distributed as a zip (`__MACOSX` litter and all) — no version history, no changelog, single author, companion video series still in progress.

---

## Comparison at a glance

| | Matt | Drew | Addy | Rob |
|---|---|---|---|---|
| Shape | à-la-carte primitives | behavior-discipline kit | SDLC skill library | end-to-end project pipeline |
| Skills | 38 (25 shipped) | 12 | 25 (+9 cmds, 4 agents) | 15 (+14 cmds, 4 agents) |
| `SKILL.md` lines | ~2.6k | ~2.1k | ~7.5k | ~1.9k (+~6.8k refs/templates) |
| Median skill | ~75 lines | bimodal 60/230 | ~300 lines | ~131 lines |
| Focus | engineering workflow primitives | agent behavior/rigor (+ new: synthetic users, parallel lanes) | full SDLC process | brief → build → review → accept, one human gate |
| Trigger style | mostly user-invoked | all model-invoked | all model-invoked | all model-invoked; pipeline started by `/sprint` |
| Idle context cost | lowest (~550 tok) | low (~1k tok) | ~2.6k tok (was ~4k w/ hook) | ~2.8k tok + CLAUDE.md from `/init` |
| Stack opinions | nearly none | none | web/TS tilt, lessening | strong: TS/Bun/Next, OO, Postgres/SQLite |
| Maturity | 472 commits, no release since 08-06 | 14 commits, 1 author | 590 commits, 46 authors, CI evals | zip drop, no history, 1 author |
| Best if | you want à-la-carte saved workflows | agents keep declaring half-done work "done" | you want a team-wide production process (esp. web) | you want to hand off whole TS features and review finished work, not steps |

## Recommendation

Given that naked works for you and you're time-poor: **don't install any full pack.** Rob's is the one most tempting to install wholesale, because it's the only one that promises "one sentence in, reviewed commits out" — but that's also why it's the least reversible in practice. It writes `docs/`, a CLAUDE.md, and a git history shaped around its own pipeline. Do this instead, in order:

1. **Read one file: Matt's `writing-for-agents`** (`mattskills/skills/productivity/writing-for-agents/SKILL.md`, ~15 min). It's the best explanation of *why* your CLAUDE.md gets in the way (eager context load, negation traps, stale duplication) and doubles as the tutorial for writing your own skills. Then skim **Rob's `README.md`** (~10 min) for the opposing design argument: "spend tokens, not attention."
2. **Trial 2–3 individual skills for a week**, not a pack:
   - Matt's `grill-me`/`grilling` — user-invoked, fires only when you ask, addresses the most common real failure (agent starts coding before requirements are clear). If you find the questioning tiresome, that's the signal Rob's assume-then-show style fits you better. Try writing a tiny skill that does that instead.
   - Matt's `diagnosing-bugs` — for the "it keeps guessing at fixes" failure mode.
   - **One** of these, only if you run long autonomous sessions:
     - Drew's `execution-rigor` — if your pain is runs that end half-finished.
     - Rob's `reviewer` agent — if your pain is runs that end "done" with tests green but the feature not actually wired to the entry point. Copy `claude-playbook/claude-playbook/.claude/agents/reviewer.md`, strip the TypeScript/Bun specifics that don't match your stack, and drop the `simplify` dependency if you don't use it.

   Skip all three if you work in short interactive sessions.
3. **Treat Addy's and Rob's repos as reference libraries, not installs.** From Addy, read `docs/comparison.md` and the `references/` checklists; the anti-rationalization tables are a genuinely good pattern. From Rob, steal the delight budget and the "three questions max, write assumptions down" rules if you like them. Install Addy's pack only if you're standardizing a team on a web stack. Try Rob's pipeline only on a throwaway greenfield TS side project, to see what hands-off feature delivery feels like.
4. **If any borrowed skill survives the week, rewrite it in your own words** — that's the build-your-own path Rob advocates, arrived at with evidence instead of ideology.

Exit criterion: if after a week no skill has fired usefully at least twice, delete them all and go back to naked. That's a fine outcome.

---

## What changed since 2026-08-15

| Repo | Commits since | What matters |
|---|---|---|
| Addy | 152 | **SessionStart meta-skill injection removed** (idle cost ~4k → ~2.6k tok); +`constraint-driven-development` skill & `/constraints`; security patterns moved into a reference file; eval routing floor 80→95%; Copilot CLI install guide. |
| Drew | 3 | +`focus-group` (synthetic user panels, Python validator), +`headless-lanes` (multi-CLI orchestration); original 10 skills untouched. |
| Matt | 35 | +3 unreleased in-progress skills (`implement-spec`, `pr`, `retro`); em-dash purge; no plugin release since 08-06. Recommended skills unchanged. |
| Rob | n/a | First actual read — see above. |

Corrections to the August notes:
- Matt's "~3,800 lines of skill markdown" counted every `.md`, not just `SKILL.md`.
- `grilling` had already moved to round-based questions before August.
- Addy's "anti-rationalization table in every skill" was slightly overstated (`idea-refine` and `using-agent-skills` lack one).
- Addy and Matt showing the same "437 commits" in the table was an error. Current figures are above.

---

## Prompts

### Original prompt (what I asked)

> seems like everyone has some set of "skills" they want to share and they boast they have the best. Theo.gg (the youtuber) recently tried out Matt's skills and he liked them. I have other friends that swear by them too. One other friend has his own set of skills and of course he says is best. and Addy is a famous former googler. meanwhile, Rob Conery says you should build your own skills and has some toolkit for it. I don't have time to look at all these! And honestly, I find that the skills and agents.md and claude.md gets in the way and the regular claude code harness running naked works fine! what am I missing? which of these options should I spend time looking into? I cloned addy, drew, and matt's repos here Can you check them out, give me a summary and a recommendation of what to check out?

### Better, neutral version to try

The original leans on social proof (Theo liked it, famous ex-googler, friends swear by it), which nudges a model toward validating the popular choice. This version strips authority cues, forces criteria-based evaluation, and anchors on your actual baseline:

> I have four agent-skill collections here: ./addyskills, ./drewskills, ./mattskills, ./claude-playbook. My baseline is Claude Code with no CLAUDE.md and no skills, and it currently works well for me. Evaluate each collection on these criteria only — do not weigh author reputation, popularity, or endorsements:
>
> 1. **Idle context cost**: what loads eagerly vs. on-demand if installed; roughly how many tokens sit in every session doing nothing.
> 2. **Trigger model**: which skills auto-fire on model judgment vs. only when I invoke them, and the realistic risk of one firing when I don't want it.
> 3. **Opinionatedness fit**: what workflow/stack opinions are baked in, and whether they match mine. [My context: I mostly do ___, my stack is ___, my typical session is ___ minutes and interactive/autonomous.]
> 4. **Reversibility and maintenance**: install/uninstall story, cherry-pick support, commit activity, single- vs. multi-maintainer risk.
>
> For each collection, name the specific failure modes of my naked baseline it claims to fix, and state whether my usage pattern would actually hit those failure modes. Then recommend **at most three individual skills total across all collections — or zero if none clear the bar** — for a one-week trial, and for each, tell me what observable evidence at the end of the week would prove it earned its context cost.

Why it's better: it makes "install nothing" an explicitly legitimate answer, forces per-skill rather than per-pack recommendations, asks for falsifiable success criteria instead of vibes, and the fill-in-the-blank context slot makes the answer about your workflow instead of the general case.
