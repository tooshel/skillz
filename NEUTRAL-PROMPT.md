# Neutral Evaluation: Three Agent-Skill Collections

*Run 2026-08-15 against the clones in this directory. Criteria-only evaluation; author reputation, popularity, and endorsements deliberately excluded. Measurements taken directly from the repos (description lengths, invocation flags, git history).*

## The prompt

> I have three agent-skill collections cloned here: ./addyskills, ./drewskills, ./mattskills. My baseline is Claude Code with no CLAUDE.md and no skills, and it currently works well for me. Evaluate each collection on these criteria only — do not weigh author reputation, popularity, or endorsements:
>
> 1. **Idle context cost**: what loads eagerly vs. on-demand if installed; roughly how many tokens sit in every session doing nothing.
> 2. **Trigger model**: which skills auto-fire on model judgment vs. only when I invoke them, and the realistic risk of one firing when I don't want it.
> 3. **Opinionatedness fit**: what workflow/stack opinions are baked in, and whether they match mine. [My context: I mostly do ___, my stack is ___, my typical session is ___ minutes and interactive/autonomous.]
> 4. **Reversibility and maintenance**: install/uninstall story, cherry-pick support, commit activity, single- vs. multi-maintainer risk.
>
> For each collection, name the specific failure modes of my naked baseline it claims to fix, and state whether my usage pattern would actually hit those failure modes. Then recommend at most three individual skills total across all collections — or zero if none clear the bar — for a one-week trial, and for each, tell me what observable evidence at the end of the week would prove it earned its context cost.

*Note: the context blanks were left unfilled, so criterion 3 and the "would you actually hit this" judgments are given as conditionals on session style. Fill in the blanks and re-run for a sharper answer.*

---

## ./addyskills — 24 skills + agents + commands + hooks

**Idle context cost: highest of the three, ~3.5–4.5k tokens per session.**
Eager: all 24 skill descriptions (~6.8k chars ≈ 1,700 tokens), plus a SessionStart hook that injects the full 191-line `using-agent-skills` meta-skill into *every* session (~1.5–2k tokens), plus 8 slash-command and 4 agent-persona descriptions. On-demand: skill bodies are large — median ~294 lines, up to 499 (`security-and-hardening`) — so each trigger pulls roughly 2–4k more tokens, sometimes plus shared `references/` checklists.

**Trigger model: all 24 auto-fire.** Zero skills set `disable-model-invocation`; every description is written with "Use when…" triggers across the whole SDLC (planning, testing, review, security, shipping, git). Realistic misfire risk is the highest here simply because the trigger surface is broadest — routine work like "commit this" or "fix this test" overlaps several skills' trigger phrases, and the injected meta-skill actively instructs the model to route through the catalog. Only the 8 slash commands are purely user-invoked.

**Opinionatedness: strongest and most specific.** Google-derived engineering process (trunk-based dev, ~100-line changes, test pyramid ratios, anti-rationalization tables that argue back at shortcuts), with a pronounced web/frontend/TypeScript tilt: Core Web Vitals, Chrome DevTools MCP, WCAG checklists, Jest/Playwright examples. Great fit if you ship production web software on a team and *want* enforced ceremony; poor fit for solo exploratory work, non-web stacks, or if you resent process overhead — the pack is explicitly designed to override your (and the model's) judgment about when process applies.

**Reversibility/maintenance: good on both counts.** Cherry-picking is first-class (`npx skills add --skill <name>`, though per-skill installs miss the shared `references/` — a known upstream gap) and plugin uninstall is clean. Most active repo measured: 243 commits in the last 90 days, genuinely multi-maintainer (top contributor ~44% of commits, several regulars) — lowest abandonment risk.

**Baseline failure modes it targets:** the model skipping tests/review/security under time pressure, rationalizing shortcuts, hallucinating framework APIs instead of citing docs, shipping without launch checks. **Would you hit them?** In short interactive sessions where you review every diff, mostly no — you *are* the process. These bite in team settings and long autonomous runs.

## ./drewskills — 10 skills

**Idle context cost: lowest, ~800–1,000 tokens.** Ten descriptions (~3.2k chars ≈ 800 tokens), no hooks, no agents, no commands. On-demand bodies are bimodal: five skills at ~60 lines, five at 140–460.

**Trigger model: all 10 auto-fire**, and the descriptions are deliberately trigger-dense (the README says the description is the tuning knob). Misfire risk is moderate: the rigor skills (`execution-rigor`, `truth-loop`, `verification-hygiene`, `keep-going`) trigger on generic situations — "implement this", "is this done", "keep working" — so they can engage ceremony on tasks too small to warrant it. One skill, `execution-rigor`, will explicitly *refuse* a mid-task "just make it work" instruction; that's the desired behavior for its audience but is the single highest "firing when I don't want it" consequence in any of the three collections.

**Opinionatedness: narrow but intense.** No stack opinions at all — nothing about languages, frameworks, or git workflow. The opinions are entirely about agent behavior: named phase loops with JSON state objects, forbidden-shortcut lists (`as any`, `it.skip`, silent catch) with grep commands to detect them, mandatory adversarial subagent review with cold context. Fit is binary: if your pain is agents declaring half-finished work done during long autonomous runs, this is purpose-built; if your sessions are short and interactive, nearly the whole collection is solving a problem you don't have.

**Reversibility/maintenance: trivially reversible, weakest maintenance signal.** Skills are plain directories; `install.sh` copies per-skill and never touches unrelated files, or use the plugin route. But: 11 commits total, all on a single day (2026-08-15 — today), one author. This is a v1 published this morning, not a maintained project yet. No track record of fixes, and single-maintainer risk is maximal.

**Baseline failure modes it targets:** premature "done" claims, plausible-but-unverified reasoning, drift during long runs, verbose non-answers (`wat`). **Would you hit them?** Only if you run autonomous sessions long enough to stop reading every step. Interactive users hit at most the `wat` verbosity itch.

## ./mattskills — 35 skills (25 shipped in plugin)

**Idle context cost: lowest per installed skill, ~600–900 tokens for the full plugin.** Of the 25 promoted skills, 14 are user-invoked with `disable-model-invocation: true` — their descriptions don't enter model context at all. Only the 11 model-invoked skills' descriptions load eagerly (~2.3k chars ≈ 600 tokens). On-demand bodies are the smallest measured: median ~75 lines, several under 25, so even a trigger costs only a few hundred tokens.

**Trigger model: split by design, and formally enforced.** A written invariant (`.agents/invocation.md`) separates user-invoked workflows (`grill-me`, `to-spec`, `implement`, `triage`, `handoff`…) from model-invoked primitives (`tdd`, `diagnosing-bugs`, `code-review`, `research`…). Misfire risk concentrates in the 11 auto-fire skills, and their bodies are small enough that a wrong trigger costs little. This is the only collection where "fires when I don't want it" was treated as a design constraint rather than an afterthought.

**Opinionatedness: moderate, process-shaped, stack-agnostic.** Assumes an issue tracker, a test suite, and willingness to answer questions before code gets written (spec → tickets → implement, red-before-green TDD, gated bug diagnosis: "no red-capable command, no Phase 2"). Almost nothing is TypeScript-specific despite expectations — 2 of 35 skills. Fits engineers on long-lived codebases who want structure à la carte; the user-invoked design means you opt into ceremony per-task instead of having it imposed.

**Reversibility/maintenance: best-balanced.** Two install routes with different reversibility (managed auto-updating plugin vs. `skills.sh` editable copies you own), per-skill cherry-picking, and skills are self-contained directories. Very active: 359 commits in the last 90 days, changesets releases, a deprecation policy (retired skills get deleted, not hoarded). Effectively single-maintainer (~90% one author plus bots) — real bus-factor risk, mitigated by the fact that copied skills keep working unmaintained.

**Baseline failure modes it targets:** coding before requirements are understood, guess-and-check bug fixing without a reproduction, context lost between sessions (handoff), unstructured spec→implementation flow. **Would you hit them?** The first two are the only failure modes in any collection that bite *interactive* users too — under-specified prompts and premature fixes happen in 20-minute sessions, not just long runs.

---

## Comparison on the four criteria

| | addyskills | drewskills | mattskills |
|---|---|---|---|
| Idle cost (est.) | ~3.5–4.5k tokens + per-session hook | ~800–1,000 tokens | ~600–900 tokens |
| Auto-fire skills | 24 of 24 | 10 of 10 | 11 of 25 shipped |
| Misfire risk | highest (broad SDLC triggers + routing hook) | moderate (generic rigor triggers, one skill refuses instructions) | lowest (invariant-enforced split, small bodies) |
| Opinions | Google-style process, web/TS tilt | agent-behavior rigor only, stack-free | tracker+tests process, stack-agnostic |
| Cherry-pick | yes (refs gap) | yes | yes |
| 90-day commits / maintainers | 243 / multi | 11 (day one) / solo | 359 / solo+bots |

## Recommendation: two skills now, a conditional third — not zero, but close

Zero is a defensible answer given a working baseline — but two candidates have idle costs so low that the bar drops from "improves my sessions" to "gets used at all." Trial these:

1. **`mattskills` → `grill-me` (with its `grilling` dependency)** — user-invoked only, so its idle cost is ~0 tokens and its misfire risk is literally zero; the trial risks nothing. Targets under-specified requests, the one failure mode interactive users hit weekly. **Week-end evidence it earned its keep:** on ≥2 occasions, a question it forced changed your design or scope *before* code was written (you'd have found the issue after implementation otherwise). If you never invoked it, delete it — that's the whole test for a user-invoked skill.

2. **`mattskills` → `diagnosing-bugs`** — model-invoked, so this one has a real (small) cost: one eager description plus auto-fire risk on bug-shaped requests. Targets guess-and-check fixing. **Evidence:** on ≥1 real bug it forced a failing reproduction before the fix, and that fix stuck without a follow-up "actually it's still broken" session. **Disqualifying evidence:** it auto-fired unhelpfully (ceremony on a typo-level fix) more than twice in the week.

3. **`drewskills` → `execution-rigor` — conditional.** Install only if you run autonomous sessions longer than ~30 minutes that you don't fully review. **Evidence:** it catches ≥1 forbidden shortcut (stub TODO, skipped test, silent catch, `as any`) that you verify you would otherwise have shipped. If your week is all interactive sessions, skip it and the recommendation is two. Also weigh the day-one maintenance signal: pin the copy you install rather than tracking the repo.

**Not recommended for trial despite quality:** anything from `addyskills` as an *installed* skill — its per-session floor (~4k tokens plus a routing hook that actively steers the model) is the exact "gets in the way" cost a working naked baseline shouldn't pay. Its checklists read fine as documents; that's free.

**Uninstall criterion for the whole experiment:** any skill with zero qualifying evidence after one week comes out. Reverting to naked is a success condition of the trial, not a failure.
