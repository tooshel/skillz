# Skills Repos: Comparison & Recommendation

*Compiled 2026-08-15 from the three repos cloned in this directory.*

## TL;DR

You're not missing much by running naked — but you're conflating two different costs. `CLAUDE.md` / `AGENTS.md` load **eagerly** into every session whether relevant or not; skills load **lazily** (only their one-line descriptions sit in context until one triggers). A user-invoked skill (slash command) costs approximately nothing until you call it. So "skills get in the way" is mostly true of always-on instruction files and of skill packs with aggressive auto-trigger descriptions — not of the mechanism itself.

The naked harness is genuinely good in 2026. Skills earn their keep in exactly three situations:

1. **A workflow you re-prompt often** (interview me before coding, review this PR my way, write a spec first) — a skill is just a saved prompt with a trigger.
2. **Long autonomous runs** where the agent drifts, declares half-done work finished, or rationalizes skipping steps — process-discipline skills are countermeasures for that.
3. **Team consistency** — everyone's agent follows the same review/ship process.

If none of those describe your week, keep running naked and cherry-pick later.

---

## The three repos

### Matt Pocock — `mattskills/` (mattpocock/skills)

- **35 skills** (25 shipped in the plugin), median **~75 lines**, some as short as 7 lines. Total skill markdown: ~3,800 lines.
- Philosophy: "Skills for real engineers, not vibe coding." Small, composable primitives — thin orchestrator skills that just call two other skills, plus dense reference docs (`tdd`, `diagnosing-bugs`). Explicitly positioned *against* big frameworks (GSD, BMAD, Spec-Kit) that "take away your control."
- **Mostly user-invoked** — a formal invariant marks which skills can auto-trigger and which only fire when you ask. This is the pack least likely to "get in the way."
- Not TypeScript-specific despite the author. Clusters: requirements grilling, spec→ticket→implement pipeline, TDD, debugging, domain modeling, code review.
- Standouts: `grilling` (one-question-at-a-time requirements interrogation), `diagnosing-bugs` (six gated phases, "no red-capable command, no phase 2"), `writing-for-agents` (a meta-skill that teaches how to write skills — context load vs. cognitive load, progressive disclosure, anti-negation).
- Install: official Claude Code plugin marketplace (`claude plugins install mattpocock-skills`) or `npx skills@latest add mattpocock/skills` for editable copies. Actively maintained (437 commits, changesets releases, commits as recent as today).

### Drew Ewing — `drewskills/` (aewing/skills)

- **10 skills**, ~2,600 lines total. Narrow and deliberate: agent *behavior* discipline, not engineering process.
- Theme: "rigor with calm" — stop "looks done" from passing as done (`execution-rigor`, 10 named phases with a forbidden-shortcut list and grep commands to detect them) and stop "plausible" from passing as true (`truth-loop`). Includes adversarial subagent phases where a cold-context reviewer tries to refute the work.
- Personality outliers: `roast-review` (evidence-required roast of your codebase), `wat` (ADHD-friendly answer-first response format), `tui-design`.
- Unusual: `execution-rigor` will *refuse* a mid-task "just make it work" shortcut request, on the grounds you invoked the skill to be defended from yourself. The repo also audits its own prose with its own `goodtalk` skill and publishes the audit ledger.
- Install: plugin marketplace or a clean 77-line `install.sh` that copies into `~/.claude/skills`. Single author, 11 commits — newest and least battle-tested of the three.

### Addy Osmani — `addyskills/` (addyosmani/agent-skills)

- **24 skills** organized by SDLC phase (Define → Plan → Build → Verify → Review → Ship), ~7,700 lines of skill markdown plus 4 agent personas, 8 slash commands (triplicated for Claude/Gemini/Antigravity), hooks, shared reference checklists, and — rare for skill packs — a **CI-gated eval harness** with adversarial "pressure cases."
- Philosophy: production engineering discipline, heavily sourced from *Software Engineering at Google* (Hyrum's Law, Beyoncé Rule, trunk-based dev, ~100-line changes). Every skill has an anti-rationalization table rebutting the excuses an agent makes to skip steps.
- Pronounced **web/frontend/TS tilt**: Core Web Vitals, Chrome DevTools MCP browser testing, WCAG, a `/webperf` command.
- Standouts: `doubt-driven-development` (adversarial fresh-context self-review, optionally escalating to Gemini/Codex as a second model), `interview-me`, the `simplify-ignore` hook that physically hides protected code from the model, and an honest `docs/comparison.md` sizing itself against competitors — including Matt's pack.
- Install: nine different tool targets. The most comprehensive, and by far the heaviest — 24 model-invocable skill descriptions is the largest always-resident trigger surface of the three.

### Rob Conery (not cloned) — build-your-own

Different bet entirely: don't adopt anyone's opinions, use tooling to extract skills from *your own* repeated workflows. Philosophically this is the endgame — a skill encoding how *you* ship beats a generic one — but it has a cold-start cost: you need to notice your own patterns first. Note that Matt's `writing-for-agents` skill is effectively a free tutorial for this path, and Anthropic ships a `skill-creator` skill too. You don't need Rob's toolkit to try the idea.

---

## Comparison at a glance

| | Matt | Drew | Addy |
|---|---|---|---|
| Skills | 35 (25 shipped) | 10 | 24 |
| Skill markdown | ~3.8k lines | ~2.6k lines | ~7.7k lines |
| Median skill | ~75 lines | bimodal 60/230 | ~294 lines |
| Focus | engineering workflow primitives | agent behavior/rigor | full SDLC process |
| Trigger style | mostly user-invoked | mostly model-invoked | mostly model-invoked |
| Idle context cost | lowest | low | highest |
| Maturity | 437 commits, evals via issues | 11 commits, 1 author | 437 commits, CI evals |
| Best if | you want à-la-carte saved workflows | agents keep declaring half-done work "done" | you want a team-wide production process (esp. web) |

## Recommendation

Given that naked works for you and you're time-poor: **don't install any full pack.** Do this instead, in order:

1. **Read one file: Matt's `writing-for-agents`** (`mattskills/skills/productivity/writing-for-agents/SKILL.md`, ~15 min). It's the best explanation of *why* your CLAUDE.md gets in the way (eager context load, negation traps, stale duplication) and doubles as the tutorial for the Conery build-your-own path.
2. **Trial 2–3 individual skills for a week**, not a pack:
   - Matt's `grill-me`/`grilling` — user-invoked, fires only when you ask, addresses the most common real failure (agent starts coding before requirements are clear).
   - Matt's `diagnosing-bugs` — for the "it keeps guessing at fixes" failure mode.
   - Drew's `execution-rigor` — only if your actual pain is long runs that end half-finished. Skip if you work in short interactive sessions.
3. **Treat Addy's repo as a reference library, not an install.** Read `docs/comparison.md` and the `references/` checklists; steal individual ideas (the anti-rationalization tables are a genuinely good pattern). Install the pack only if you're standardizing a team on a web stack.
4. **If any borrowed skill survives the week, rewrite it in your own words** — that's the Conery path, arrived at with evidence instead of ideology.

Exit criterion: if after a week no skill has fired usefully at least twice, delete them all and go back to naked. That's a fine outcome.

---

## Prompts

### Original prompt (what I asked)

> seems like everyone has some set of "skills" they want to share and they boast they have the best. Theo.gg (the youtuber) recently tried out Matt's skills and he liked them. I have other friends that swear by them too. One other friend has his own set of skills and of course he says is best. and Addy is a famous former googler. meanwhile, Rob Conery says you should build your own skills and has some toolkit for it. I don't have time to look at all these! And honestly, I find that the skills and agents.md and claude.md gets in the way and the regular claude code harness running naked works fine! what am I missing? which of these options should I spend time looking into? I cloned addy, drew, and matt's repos here Can you check them out, give me a summary and a recommendation of what to check out?

### Better, neutral version to try

The original leans on social proof (Theo liked it, famous ex-googler, friends swear by it), which nudges a model toward validating the popular choice. This version strips authority cues, forces criteria-based evaluation, and anchors on your actual baseline:

> I have three agent-skill collections cloned here: ./addyskills, ./drewskills, ./mattskills. My baseline is Claude Code with no CLAUDE.md and no skills, and it currently works well for me. Evaluate each collection on these criteria only — do not weigh author reputation, popularity, or endorsements:
>
> 1. **Idle context cost**: what loads eagerly vs. on-demand if installed; roughly how many tokens sit in every session doing nothing.
> 2. **Trigger model**: which skills auto-fire on model judgment vs. only when I invoke them, and the realistic risk of one firing when I don't want it.
> 3. **Opinionatedness fit**: what workflow/stack opinions are baked in, and whether they match mine. [My context: I mostly do ___, my stack is ___, my typical session is ___ minutes and interactive/autonomous.]
> 4. **Reversibility and maintenance**: install/uninstall story, cherry-pick support, commit activity, single- vs. multi-maintainer risk.
>
> For each collection, name the specific failure modes of my naked baseline it claims to fix, and state whether my usage pattern would actually hit those failure modes. Then recommend **at most three individual skills total across all collections — or zero if none clear the bar** — for a one-week trial, and for each, tell me what observable evidence at the end of the week would prove it earned its context cost.

Why it's better: it makes "install nothing" an explicitly legitimate answer, forces per-skill rather than per-pack recommendations, asks for falsifiable success criteria instead of vibes, and the fill-in-the-blank context slot makes the answer about your workflow instead of the general case.
