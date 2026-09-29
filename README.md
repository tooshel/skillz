# skillz

Notes from comparing the popular "agent skills" collections for Claude Code (and other coding agents), to answer: *is running the naked harness fine, or am I missing something?*

**👉 The write-up: [SKILLS-NOTES.md](SKILLS-NOTES.md)** — per-collection summaries, a comparison table, a recommendation, what changed since the first pass, and a neutral evaluation prompt to reuse.

**👉 The neutral prompt, actually run: [NEUTRAL-PROMPT.md](NEUTRAL-PROMPT.md)** — a criteria-only evaluation (measured idle token cost, auto-fire vs. user-invoked triggers, opinion fit, reversibility/maintenance) with a max-three-skills recommendation and per-skill pass/fail evidence for a one-week trial.

*First pass 2026-08-15 (three repos). Updated 2026-09-28: re-cloned and re-measured, and added Rob Conery's `claude-playbook`.*

## What's (not) in this repo

The four collections that were reviewed live in this directory but are gitignored, because they're other people's work. To recreate the working directory:

```sh
git clone https://github.com/addyosmani/agent-skills.git addyskills
git clone https://github.com/aewing/skills.git drewskills
git clone https://github.com/mattpocock/skills.git mattskills
# Rob Conery's claude-playbook was shared as a zip; unzip it to ./claude-playbook/
```

## TL;DR of the notes

- CLAUDE.md/AGENTS.md load into every session eagerly; skills load lazily and user-invoked ones cost ~nothing until called. The "it gets in the way" feeling is about always-on instruction files, not the skill mechanism.
- Don't install a full pack. Trial a few individual skills (Matt Pocock's `grill-me` and `diagnosing-bugs` first), read his `writing-for-agents` skill, and use Addy Osmani's and Rob Conery's repos as reference libraries.
- Rob's `claude-playbook` is a whole pipeline (`/sprint` → one approval → build/review/commit per task → product-owner acceptance), heavily TypeScript/OO. Don't adopt it wholesale. Its read-only `reviewer` agent, which traces from the deployed entry point and fails on ghost code, is the piece worth borrowing if you run long autonomous builds.
- If a borrowed skill proves useful for a week, rewrite it in your own words; if nothing earns its keep, going back to naked is a fine outcome.
