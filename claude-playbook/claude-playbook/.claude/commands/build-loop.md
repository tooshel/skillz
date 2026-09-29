---
description: Drive a PLAN.md to completion task-by-task. Builder builds, reviewer gates, commit only on pass, product owner accepts at the end.
argument-hint: [path-to-PLAN.md]
model: sonnet
---

# build-loop

> 🎯 **Design for change.** Each task's diff should be small, local, and
> behind a stable seam. The reviewer fails work that couples modules or
> bloats blast radius. The builder should pre-empt that.

Execute the build plan in `$ARGUMENTS` (default: `docs/PLAN.md`) one task at a
time. You are the orchestrator (the lead thread). You do **not** write feature
code or review it yourself. You dispatch and gate.

## Scope

- IN: build each unchecked task, get it through review, commit it, then
  get the finished work accepted by the product owner.
- OUT: planning, writing PLAN.md, authoring specs, refactor/refinement passes.
  This command **consumes** a plan; it does not author one.

## Preflight

1. Resolve the plan: use the path in `$ARGUMENTS`; else `docs/PLAN.md`; else
   **stop and ask**. Do not infer a plan.
2. Read it. It must be a checklist of tasks (`- [ ]` / `- [x]`). If it has no
   checkboxes, stop and report. It is not a build plan.
3. Confirm a clean git working tree. If dirty, stop and report; do not build on
   top of uncommitted changes.

## The loop

For each task still unchecked (`- [ ]`), in order, top to bottom:

1. **Build.** Dispatch a `builder` subagent (Agent tool,
   `subagent_type: builder`, model **sonnet**). Give it: the exact task text,
   the relevant section of PLAN.md, the files it owns, the story's spec
   file, and an instruction to invoke the project's TypeScript + DB skills,
   activate the pending specs for this task, and run the suite
   (`bun test` / project test command) before reporting back. The builder
   **does not commit**.

2. **Review.** Dispatch a `reviewer` subagent (Agent tool,
   `subagent_type: reviewer`, model **fable**; reviewer must be ≥ builder).
   It invokes `security-web` and `simplify`, reads the builder's diff, and
   returns a verdict: `PASS` or `FAIL` with specific findings.

3. **Recurse.** If `FAIL`: send the findings back to a fresh `builder` for the
   same task. Repeat build → review until `PASS`. No cap. A task is not done
   until it passes code **and** security review. Nothing reaches git history
   before `PASS`.

4. **Smoke the entry point.** Before commit, build/start the production
   artifact and drive **one real request through the deployed entry point**
   (one `curl` against `wrangler dev`, one `bun build` + invoke, one route
   fetch, whatever applies to the deploy target). Fake adapters (DB, email,
   storage) are fine, but the request must travel through the exported
   handler, not an internal function. Assert the spec's side effect actually
   occurred. If the smoke fails, recurse to step 3 with a finding. Unit-test
   green is not enough to ship.

5. **Commit.** On `PASS` + smoke green, have the builder commit *only this
   task's changes* with a message naming the task. One commit per task.

6. **Check the box.** Edit PLAN.md: `- [ ]` → `- [x]` for the completed task.
   Commit that PLAN.md change with the task commit or immediately after.

7. Next task.

## Extras

Tasks tagged `delight:` go through the same loop as everything else. Two
differences:

- If a `delight:` task fails review three times, **skip it**. Revert its
  changes, mark it `- [-]` in PLAN.md with a one-line reason, log it in
  `docs/MEMORY.md`, and move on. An extra never blocks the sprint.
- The reviewer checks it against the budget: no new dependency, no schema
  change, nothing else depends on it.

## Acceptance

When every box is checked, run the full test suite, then:

1. Dispatch a `product-owner` subagent in **acceptance mode** (Agent tool,
   `subagent_type: product-owner`). Give it `docs/BRIEF.md` (or
   `docs/PROJECT.md` and `docs/SPEC.md` if there's no brief) and the run
   command from `CLAUDE.md`.
2. Act on the verdict:
   - `ACCEPT`: go to Finish.
   - `POLISH`: append each item to PLAN.md as a `polish:` task and run
     them through the loop above, review gate included. Same skip rule as
     extras. **One round only.** Don't dispatch the product owner again.
   - `REJECT`: the named brief line didn't happen. Append a task that
     fixes it, run it through the loop, then re-run acceptance once. If
     it's rejected again, stop and report.
3. Copy anything the product owner listed under "Next" into the **Next**
   section of `docs/BRIEF.md`.

If the project has no `product-owner` agent, skip this section and say so
in the report.

## Finish

Report a summary: tasks completed, extras shipped and skipped, polish
items, commits, anything still red, what's under Next, and how to run it.

## Rules

- Sequential, dependency-ordered. This is a pipeline, not a parallel team. Use
  the Agent tool (subagents), not Agent Teams.
- Builder owns code; reviewer owns the gate; product owner owns acceptance;
  you own sequencing and the checkbox state. Never collapse these roles.
- **Builders don't freelance.** Extras come from the plan. A diff that
  includes something the task didn't ask for is a review finding.
- If a gate can't pass after repeated attempts and the builder is stuck, stop
  and report the task + findings rather than committing degraded code.
