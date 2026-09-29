# Your Tech Profile

**Purpose:** Directive reference for any AI agent working on your behalf.
These are not suggestions. They are how you work. Fill them in honestly and
the AI will stop guessing.

---

## Stack

**Language:** [TypeScript / Python / Go / etc. — and any nuance, e.g. "TS or
JS, I don't care which"]

**Database (development/testing):** [SQLite / Postgres / whatever]

**Database (production):** [Postgres / MySQL / etc.] — and a NEVER list:
[Mongo? Dynamo? Firebase?]

**ORM / query layer:** [Drizzle / Prisma / raw SQL / no preference]

**Frameworks you reach for:** [Next.js / Hono / Bun + Elysia / Fastify / etc.]

**Frameworks to avoid (and why):**

- **[Framework]** — [one-line reason]

**Frontend:** [React / vanilla / HTMX / Svelte / etc. — and how heavy you'll
let it get]

**Deploy target:** [Vercel free tier / Cloudflare Workers / Fly / a VPS you
ssh into]

---

## Architecture priorities

Pick the two or three things you actually care about. Examples:

1. **[e.g. The data model is correct and normalized.]**
2. **[e.g. Performance is acceptable — no obvious N+1, no >1s page loads.]**
3. **[e.g. The project layout is obvious at a glance.]**

Then: "everything else is the AI's call."

---

## Project structure

Describe the layout you want at the project root. Example:

- `services/` — business logic
- `models/` — data models
- `config/` — configuration
- `views/` — templates/UI
- Keep the root clean. No deep nesting.

Or: "follow framework defaults, I don't care."

---

## What you'll sacrifice for shipping speed

[e.g. test coverage, elegant abstractions, code review polish]

## What you will NEVER sacrifice

[e.g. data integrity, security, design quality]

---

## How you read code

Be honest. Options:

- "I read every line." → AI should optimize for human readability.
- "I skim PRs, read commits." → AI should write tight commit messages.
- "I don't read code, I read docs and look at the UI." → AI should document
  decisions in markdown and never assume you'll grep for context.

This single answer changes a lot of downstream behavior.

---

## Decision-making priorities

Stack-rank what matters when there's a trade-off. Example:

1. Data model first.
2. Ship fast.
3. Everything else.
