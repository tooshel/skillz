# skillz

Notes from comparing the popular "agent skills" collections for Claude Code (and other coding agents), to answer: *is running the naked harness fine, or am I missing something?*

**👉 The write-up: [SKILLS-NOTES.md](SKILLS-NOTES.md)** — per-repo summaries, a comparison table, a recommendation, and a neutral evaluation prompt to reuse.

**👉 The neutral prompt, actually run: [NEUTRAL-PROMPT.md](NEUTRAL-PROMPT.md)** — a criteria-only evaluation (measured idle token cost, auto-fire vs. user-invoked triggers, opinion fit, reversibility/maintenance) with a max-three-skills recommendation and per-skill pass/fail evidence for a one-week trial.

## What's (not) in this repo

The three collections that were reviewed are cloned locally but gitignored — they're other people's repos. To recreate the working directory:

```sh
git clone git@github.com:addyosmani/agent-skills.git addyskills
git clone git@github.com:aewing/skills.git drewskills
git clone git@github.com:mattpocock/skills.git mattskills
```

## TL;DR of the notes

- CLAUDE.md/AGENTS.md load into every session eagerly; skills load lazily and user-invoked ones cost ~nothing until called. The "it gets in the way" feeling is about always-on instruction files, not the skill mechanism.
- Don't install a full pack — trial a few individual skills (Matt Pocock's `grill-me` and `diagnosing-bugs` first), read his `writing-for-agents` skill, and mine Addy Osmani's repo as a reference library.
- If a borrowed skill proves useful for a week, rewrite it in your own words; if nothing earns its keep, going back to naked is a fine outcome.
