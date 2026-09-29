# ⚡ Quickstart

Two commands. One approval.

```bash
# 1. Copy the toolkit into your project
cp -r path/to/claude-playbook/.claude /your/project/

# 2. Open Claude Code there
cd /your/project && claude
```

Then, inside Claude Code:

```
/init my-project                      # scaffold CLAUDE.md + docs/
/sprint add invoicing with PDF export # everything else
```

`/sprint` reads your prompt and your code, then comes back with a one-page
brief: what it thinks you want, how it'll build it, and one to three small
things you didn't ask for. You approve it (or strike what you don't like)
and it plans, builds, reviews, and commits task by task.

You'll get three questions at most, and usually none.

**Before your first run:** open `.claude/skills/you/` and fill in
`background.md`, `tech.md`, `writing.md`. The better these are, the fewer
wrong guesses you'll have to correct in the brief.

**Want to think it through first?** `/explore` and `/design` are still
here. They interview you about the problem and the architecture. Use them
for greenfield projects or risky changes.

**Full docs:** [README.md](./README.md)
