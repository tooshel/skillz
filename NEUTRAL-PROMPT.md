# Neutral Evaluation: Four Agent-Skill Collections

*First run 2026-08-15 against three clones. **Re-run 2026-09-28** after re-cloning all three and adding Rob Conery's `./claude-playbook`. Criteria-only evaluation; author reputation, popularity, and endorsements deliberately excluded. Measurements taken directly from the repos (description lengths, invocation flags, git history). Token figures use ~4 chars/token.*

## The prompt

> I have four agent-skill collections here: ./addyskills, ./drewskills, ./mattskills, ./claude-playbook. My baseline is Claude Code with no CLAUDE.md and no skills, and it currently works well for me. Evaluate each collection on these criteria only — do not weigh author reputation, popularity, or endorsements:
>
> 1. **Idle context cost**: what loads eagerly vs. on-demand if installed; roughly how many tokens sit in every session doing nothing.
> 2. **Trigger model**: which skills auto-fire on model judgment vs. only when I invoke them, and the realistic risk of one firing when I don't want it.
> 3. **Opinionatedness fit**: what workflow/stack opinions are baked in, and whether they match mine. [My context: I mostly do ___, my stack is ___, my typical session is ___ minutes and interactive/autonomous.]
> 4. **Reversibility and maintenance**: install/uninstall story, cherry-pick support, commit activity, single- vs. multi-maintainer risk.
>
> For each collection, name the specific failure modes of my naked baseline it claims to fix, and state whether my usage pattern would actually hit those failure modes. Then recommend at most three individual skills total across all collections — or zero if none clear the bar — for a one-week trial, and for each, tell me what observable evidence at the end of the week would prove it earned its context cost.

*Note: the context blanks were left unfilled, so criterion 3 and the "would you actually hit this" judgments are given as conditionals on session style. Fill in the blanks and re-run for a sharper answer.*

---

## ./addyskills — 25 skills + 9 commands + 4 agents + hooks

**Idle context cost: ~2.6k tokens per session — down from ~3.5–4.5k in August.**
Eager: 25 skill descriptions (~8.7k chars ≈ 2.2k tokens) plus 9 slash-command and 4 agent-persona descriptions (~1.6k chars ≈ 400 tokens). The SessionStart hook that injected the full `using-agent-skills` meta-skill into every session **was removed on 2026-09-15**; the script remains only for hosts without native skill routing. On-demand: skill bodies are still large — median ~300 lines, max 496 (`performance-optimization`) — so each trigger pulls roughly 2–4k more tokens, sometimes plus a skill-local `references/` file.

**Trigger model: all 25 auto-fire.** Zero skills set `disable-model-invocation`; every description is written with "Use when…" triggers across the whole SDLC, and ~14 descriptions were *widened* with more trigger phrases since August. Misfire risk is still high on breadth — routine work like "commit this" or "fix this test" overlaps several skills — but lower than before now that no hook actively instructs the model to route through the catalog. Only the 9 slash commands are purely user-invoked.

**Opinionatedness: strong and process-shaped.** Google-derived engineering process (trunk-based dev, ~100-line changes, test pyramid ratios, anti-rationalization tables), with a web/frontend/TypeScript tilt (Core Web Vitals, Chrome DevTools MCP, WCAG) that has been diluted somewhat by new backend/DB, SLO, and runbook material. New `constraint-driven-development` records your quality floor in `CONSTRAINTS.md` and flags diffs that lower it. Good fit if you ship production software on a team and *want* enforced ceremony. Poor fit for solo exploratory work, or if you resent process overhead.

**Reversibility/maintenance: best of the four.** Cherry-picking is first-class (`npx skills add --skill <name>`; the per-skill `references/` gap is partly closed by moving references into skill directories). Plugin uninstall is clean. 286 commits in the last 90 days from 46 authors — the only genuinely multi-maintainer project here — with CI-gated routing evals now at a 95% floor.

**Baseline failure modes it targets:** the model skipping tests/review/security under time pressure, rationalizing shortcuts, silently lowering the quality bar (suppressions, skipped tests), hallucinating framework APIs, shipping without launch checks. **Would you hit them?** In short interactive sessions where you review every diff, mostly no — you *are* the process. These bite in team settings and long autonomous runs.

## ./drewskills — 12 skills

**Idle context cost: low, ~1,000 tokens.** Twelve descriptions (~4.2k chars ≈ 1,040 tokens, up from ~760 with two new skills). No Claude hooks, agents, or commands. On-demand bodies are bimodal: five skills at ~60 lines, seven at 140–460.

**Trigger model: all 12 auto-fire**, with deliberately trigger-dense descriptions. Misfire risk is moderate. The rigor skills (`execution-rigor`, `truth-loop`, `verification-hygiene`, `keep-going`) trigger on generic situations — "implement this", "is this done", "keep working" — and can bring ceremony to tasks too small to warrant it. The new `headless-lanes` (drive other agent CLIs in parallel) and `focus-group` (synthetic user panels) widen the surface further. `execution-rigor` still explicitly *refuses* a mid-task "just make it work" instruction. That has the highest firing-when-unwanted consequence of any skill here.

**Opinionatedness: narrow but intense, and broadening.** Still no language or framework opinions. The core is agent-behavior discipline: phase loops with JSON state, forbidden-shortcut lists with grep detectors, and adversarial cold-context review. The two new skills move it toward product research and multi-agent orchestration. Fit for the core is binary: purpose-built if your pain is agents declaring half-finished work done during long autonomous runs, irrelevant if your sessions are short and interactive.

**Reversibility/maintenance: trivially reversible, weak but improving maintenance signal.** Skills are plain directories. `install.sh` copies per-skill and picks up new ones automatically, or use the plugin (now v1.3.0). 14 commits across three dates (08-15, 08-21, 09-28), one author. That's no longer a single-day drop, but the track record is still thin. The original 10 skills haven't changed since launch, which is a stability plus and a no-fixes-yet minus.

**Baseline failure modes it targets:** premature "done" claims, plausible-but-unverified reasoning, drift during long runs, verbose non-answers (`wat`), and now building features no user asked for (`focus-group`). **Would you hit them?** Only if you run autonomous sessions long enough to stop reading every step. Interactive users hit at most the `wat` verbosity itch.

## ./mattskills — 38 skills (25 shipped in plugin)

**Idle context cost: lowest, ~550 tokens for the full plugin.** Of the 25 shipped skills, 14 are user-invoked with `disable-model-invocation: true`, so their descriptions don't enter model context. Only the 11 model-invoked descriptions load eagerly (~2.2k chars ≈ 550 tokens). On-demand bodies are the smallest measured: median ~75 lines, several under 25.

**Trigger model: split by design, and formally enforced.** The written invariant (`.agents/invocation.md`) separates user-invoked workflows (`grill-me`, `to-spec`, `implement`, `triage`, `handoff`…) from model-invoked primitives (`tdd`, `diagnosing-bugs`, `code-review`, `research`…). It's still in place. Misfire risk concentrates in the 11 auto-fire skills, and their bodies are small enough that a wrong trigger costs little.

**Opinionatedness: moderate, process-shaped, stack-agnostic.** It assumes you have an issue tracker and a test suite, and that you're willing to answer questions before code gets written. The flow is spec → tickets → implement, red-before-green TDD, and gated bug diagnosis. `grilling` now asks in numbered rounds, each question with a recommended answer. Only 2 of 38 skills are TypeScript-specific.

**Reversibility/maintenance: well-balanced, with a release lull.** There are two install routes: a managed plugin, or editable `skills.sh` copies you own. Per-skill cherry-picking works, and each skill is a self-contained directory. 311 commits in the last 90 days, ~90% from one author, with the rest mostly AI-agent co-authors. **No plugin release since v1.2.3 (2026-08-06)**, with 12 changesets queued. The 3 new skills are unreleased in `in-progress/`. Bus-factor risk is real but mitigated, since copied skills keep working unmaintained.

**Baseline failure modes it targets:** coding before requirements are understood, guess-and-check bug fixing without a reproduction, context lost between sessions (handoff), unstructured spec→implementation flow. **Would you hit them?** The first two are the failure modes most likely to bite *interactive* users too. Under-specified prompts and premature fixes happen in 20-minute sessions, not just long runs.

## ./claude-playbook — 15 skills + 14 commands + 4 agents (a project template)

**Idle context cost: ~2.8k tokens, plus whatever `/init` writes to CLAUDE.md.** Eager: 15 skill descriptions (~8.9k chars ≈ 2.2k tokens — several are 600–1,000 chars each), 14 command descriptions (~1.4k chars), 4 agent descriptions (~0.85k chars). `/init` then creates a CLAUDE.md that loads every session in that project: a block of always-on rules ("DO NOT SEARCH node_modules… GO ONLINE", "Tokens are water, we're in the desert") plus the stack, commands, and doc-ownership conventions. On-demand bodies are mid-sized (median ~131 lines). But a `/sprint` run is **deliberately token-hungry**, per the pack's own design principle "spend tokens, not attention". It runs parallel product-owner and architect agents, then a builder → reviewer loop per task with no retry cap on required tasks, plus a smoke test and an acceptance pass, and the planning, review and acceptance agents run on `fable`.

**Trigger model: all 15 skills auto-fire; the pipeline itself is user-started.** No `disable-model-invocation` anywhere. The commands must stay model-invocable because `/sprint` chains `/init`, `/plan`, and `/build-loop` through the Skill tool. The broad knowledge skills carry the misfire risk. `typescript-best-practices`, `solid-principles`, `design-principles`, and `gof-patterns` trigger on "writing new TypeScript", "doing code review", or "designing modules", and `github` triggers on "whenever the user wants to commit". Those misfires inject reference material rather than ceremony. The heavy process only starts when you type `/sprint`. `you` and `design-aesthetic` are described as "auto-surfaced", and until you fill them in they inject bracketed placeholders.

**Opinionatedness: the strongest stack opinions of the four.** Builder and reviewer are hard-coded to TypeScript (Next.js or Bun). Skills encode OO design (SOLID, all 23 GoF patterns), a `toString()`/`toJSON()`-on-every-class rule, Postgres conventions (plpgsql business logic, snake_case, enums over lookup tables), and SQLite + Drizzle. The process opinions are just as specific:
- one human approval per sprint
- agents state `> ASSUMPTION:` lines rather than asking, with three questions max
- a mandatory "delight budget" of 1–3 unrequested UX extras
- one commit per reviewed task, Conventional Commits
- a `docs/` tree where every file has exactly one owning command

It's built to be customized: prune skills, fill in `you/tech.md`, copy `lang-template`. Out of the box, though, it's a fit only for a TS web shop.

**Reversibility/maintenance: easy to remove, weakest maintenance signal.** Install is `cp -r .claude` into a project, and uninstall is deleting it. But the files it generates (`CLAUDE.md`, `docs/BRIEF.md`, `PLAN.md`, `STORIES.md`, `MEMORY.md`, pending spec files) and the one-commit-per-task history stay behind. Cherry-picking plain files is easy, but pieces reference each other by name: agents invoke specific skills, and `/sprint` calls `/plan` and `/build-loop`. Distribution is a zip with no git history, changelog, or version number, from one author. There's no way to tell what changed between releases, so pin your copy and treat it as yours. Its `/init` also shares a name with Claude Code's built-in `/init`.

**Baseline failure modes it targets:**
- your attention consumed by interviews and handoffs
- features that pass unit tests but aren't wired to the deployed entry point (the reviewer hand-traces from the export to the side effect and greps for `as any`, `{} as Env`, stubs, and deprecated no-ops)
- runtime mismatch (Bun APIs shipped to Workers)
- builder scope creep
- bare-minimum UX (missing empty states, unhelpful errors)
- docs drifting from code (`/document`)

**Would you hit them?** The unwired-feature and runtime-mismatch failures bite when you delegate whole features and don't trace the code yourself. In interactive sessions you'd usually catch them by running the thing. The pipeline as a whole only pays off if you want to hand off entire features and review finished work instead of steps. That is the opposite of a working naked, interactive baseline.

---

## Comparison on the four criteria

| | addyskills | drewskills | mattskills | claude-playbook |
|---|---|---|---|---|
| Idle cost (est.) | ~2.6k tokens (hook removed) | ~1,000 tokens | ~550 tokens | ~2.8k tokens + generated CLAUDE.md |
| Auto-fire skills | 25 of 25 | 12 of 12 | 11 of 25 shipped | 15 of 15 (pipeline user-started) |
| Misfire risk | high breadth, no router hook anymore | moderate (generic rigor triggers, one skill refuses instructions) | lowest (invariant-enforced split, small bodies) | moderate (broad TS/OO reference triggers; ceremony only via `/sprint`) |
| Opinions | Google-style process, web/TS tilt | agent-behavior rigor, stack-free | tracker+tests process, stack-agnostic | TS/Bun/Next + OO + Postgres, one-gate pipeline |
| Cherry-pick | yes (refs gap mostly closed) | yes | yes | yes, but pieces reference each other by name |
| 90-day commits / maintainers | 286 / 46 authors | 14 total / solo | 311 / solo + AI co-authors | none visible (zip) / solo |

## Recommendation: two skills now, a conditional third — not zero, but close

Zero is a defensible answer given a working baseline. But two candidates have idle costs so low that the bar drops from "improves my sessions" to "gets used at all." The re-run doesn't change the top two. The conditional third slot now has two alternatives, and you should pick at most one.

1. **`mattskills` → `grill-me` (with its `grilling` dependency).**
   - **Why:** it's user-invoked only, so its idle cost is ~0 tokens and it can't misfire. It targets under-specified requests, the one failure mode interactive users hit weekly.
   - **Evidence it earned its keep:** on ≥2 occasions in the week, a question it forced changed your design or scope *before* code was written.
   - **Delete it if:** you never invoked it.
   - **Counter-signal:** if you find yourself impatient with the rounds of questions, that's evidence for `claude-playbook`'s opposite philosophy — the agent writes its assumptions down and you correct them. The cheap way to test that is a 10-line user-invoked skill of your own, not installing the pipeline.

2. **`mattskills` → `diagnosing-bugs`.**
   - **Why:** it targets guess-and-check fixing. It's model-invoked, so it has a real but small cost: one eager description, plus auto-fire risk on bug-shaped requests.
   - **Evidence:** on ≥1 real bug it forced a failing reproduction before the fix, and the fix stuck without a follow-up "actually it's still broken" session.
   - **Disqualifying evidence:** it auto-fired unhelpfully (ceremony on a typo-level fix) more than twice.

3. **Conditional — only if you run autonomous sessions longer than ~30 minutes that you don't fully review. Pick at most one, by symptom:**
   - **`drewskills` → `execution-rigor`**, if runs end half-finished and get called done.
     - **Evidence:** it catches ≥1 forbidden shortcut (a stub TODO, skipped test, silent catch, or `as any`) that you verify you would otherwise have shipped.
   - **`claude-playbook` → the `reviewer` agent**, if runs end with green tests but the feature isn't reachable from the real entry point. Copy `.claude/agents/reviewer.md` alone, edit out the TypeScript/Bun assumptions that don't match your stack, and don't bring the pipeline. An agent description is ~50 tokens, and it runs only when you dispatch it.
     - **Evidence:** ≥1 `FAIL` for an unwired path, ghost code, or a runtime mismatch that you confirm was real.
     - **Disqualifying evidence:** it FAILs on stack conventions you don't follow.

   If your week is all interactive sessions, skip both and the recommendation is two.

**Not recommended as installs despite quality:**
- **addyskills:** it's much cheaper now that the router hook is gone. But 25 auto-firing skills with widened triggers is still the "gets in the way" cost a working naked baseline shouldn't pay. Read its checklists and `constraint-driven-development` as documents instead, which is free.
- **claude-playbook** as a whole: it replaces your workflow instead of patching it. It adds eager CLAUDE.md rules, a `docs/` tree, a commit cadence, and strong TS/OO opinions. It's worth a weekend on a throwaway greenfield TS project if hands-off feature delivery appeals to you. It isn't a trial item for a baseline that already works.

**Uninstall criterion for the whole experiment:** any skill with zero qualifying evidence after one week comes out. Reverting to naked is a success condition of the trial, not a failure.
