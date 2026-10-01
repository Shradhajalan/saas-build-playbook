---
name: saas-build-playbook
description: A 15-layer playbook for taking a vibe-coded app to a real, production-grade SaaS with agentic coders like Claude Code or OpenAI Codex. Covers system design, architecture, database, auth and permissions, APIs, frontend, CI/CD, testing, hosting, security, rate limiting, caching, logging, monitoring and scaling, with copy-paste agent prompts and verification checklists for each. Use this skill whenever someone is building, planning, hardening or launching a SaaS or multi-tenant web app with AI coding agents, asks "what am I missing before launch", wants a CLAUDE.md or AGENTS.md for a project, wants prompts for Claude Code or Codex, asks how to add auth, multi-tenancy, payments, webhooks, rate limits, monitoring or backups to an AI-built app, or wants a production-readiness or security review of a vibe-coded project — even if they never say "playbook" or "SaaS".
---

# SaaS Build Playbook

A prompt gets you a website. A SaaS needs fifteen more layers underneath it. This skill helps a person (usually a solo founder or small team using Claude Code or Codex) build those layers in the right order, with the agent doing the building and the human owning the decisions.

The core insight behind everything here: **agents optimise for "it runs on my machine."** Each layer has predictable blind spots that the agent will not cover unless explicitly asked. The playbook names those blind spots, hands over the prompts that close them, and gives a checklist so the human verifies instead of trusting.

## Figure out how you're being used

Work out which of these situations you're in before doing anything else, because the output differs:

1. **Advisor mode** (typical in chat): the person is driving Claude Code or Codex themselves. Your job is to tell them where they are, which decisions they need to make, and hand them tailored prompts to paste, filled in with their stack and product details rather than left as `[placeholders]`.
2. **Builder mode** (you *are* the coding agent, e.g. in Claude Code with repo access): apply the layer's rules directly to your own work. Follow the working loop below, and treat the "What agents get wrong" lists as things you personally must not do.
3. **Review mode**: the person has an existing app and wants to know what's missing. Run the gap assessment (below) and return a prioritised list, not a lecture on all fifteen layers.

If unclear, ask one short question about which it is, or infer from context (uploaded repo, tools available, how they phrase things).

## The 15 layers, in build order

The order matters: later layers assume earlier ones. Don't let someone add caching before tenancy is scoped, or monitoring before there's an error tracker.

| # | Layer | Phase | Reference file |
|---|---|---|---|
| 0 | Agent setup (CLAUDE.md / AGENTS.md, working loop) | Before anything | `references/00-agent-setup.md` |
| 1 | System design | Plan | `references/01-plan.md` |
| 2 | System architecture | Plan | `references/01-plan.md` |
| 3 | Databases & storage | Foundation | `references/02-foundation.md` |
| 4 | Auth & permissions | Foundation | `references/02-foundation.md` |
| 5 | APIs & backend logic | Foundation | `references/02-foundation.md` |
| 6 | Frontend | Product | `references/03-product-and-ship.md` |
| 7 | CI/CD & version control | Ship | `references/03-product-and-ship.md` |
| 8 | Testing | Ship | `references/03-product-and-ship.md` |
| 9 | Hosting & cloud | Ship | `references/03-product-and-ship.md` |
| 10 | Security | Harden | `references/04-harden.md` |
| 11 | Rate limiting | Harden | `references/04-harden.md` |
| 12 | Caching & CDN | Harden | `references/04-harden.md` |
| 13 | Error tracking & logs | Operate | `references/05-operate-and-grow.md` |
| 14 | Monitoring & alerts | Operate | `references/05-operate-and-grow.md` |
| 15 | Scaling | Grow | `references/05-operate-and-grow.md` |
| 16+ | Emails, analytics, flags, legal, admin, audit logs… | Beyond | `references/05-operate-and-grow.md` |

Golden rules and the pre-launch checklist live in `references/checklists.md`. A ready-to-fill project memory file is at `assets/CLAUDE.md.template`.

Read only the reference file(s) for the layer(s) actually in play. Each layer section has the same four parts: what it is, what agents get wrong, decisions the human owns, prompts, and a verify checklist.

## How to help on any given layer

1. **Locate them.** Which layer are they on? Have earlier layers been done? If they're asking about layer 11 but have no tenancy model (layer 1/3), say so briefly; it changes the answer.
2. **Surface the decisions they own first.** The agent can recommend; the human chooses. Present these as a short list with your recommendation for their stage (e.g. "modular monolith", "Postgres", "use a managed auth provider"). Don't silently decide for them.
3. **Name the blind spots** for that layer, so they know what to watch for in the agent's output.
4. **Give the prompts**, tailored. Substitute their real stack, roles, plans, region and limits. If you don't know a value, keep the bracket but say what it should be. Append the "don't fool me" block (in `00-agent-setup.md`) to any large task.
5. **Give the verify checklist** and frame it as things *they* do with their own hands (change an ID in DevTools, kill the DB in staging, hit login 20 times). Verification is the point; an agent's "done" isn't evidence.
6. **Remind them to record the decision** in CLAUDE.md / AGENTS.md or `/docs/decisions/`.

Keep responses proportionate. Someone asking "how do I add rate limiting" wants layer 11, not all fifteen.

## The working loop (every layer, every task)

1. Plan first, code second (Claude Code plan mode; in Codex, ask for a plan and no code).
2. Human reviews the plan. Bad decisions are cheap to fix here.
3. Implement in small steps on a branch.
4. Agent verifies its own work: lint, typecheck, tests, with output shown.
5. Human reads the diff before merging.
6. Record the decision.

In builder mode, follow this yourself: propose before editing, keep diffs small, run the checks and show output, and never describe work as complete with failing or skipped tests.

## Gap assessment (review mode)

When asked "what am I missing" or "is this ready to launch":

1. Read `references/checklists.md` (pre-launch checklist).
2. If you have repo access, check each item against the actual code: look for `organization_id` scoping, a central authorization function, migration files, webhook idempotency, CI config, `.env.example`, rate limiter store, structured logger, error tracker init, health endpoint. If you don't have access, ask the person to answer the checklist or paste relevant files.
3. Report as a prioritised list: blocking issues (data leaks between tenants, auth holes, no backups, non-idempotent payments) first, then important, then nice-to-have. For each gap, point to the layer and the specific prompt that fixes it.

The highest-severity items almost always are: cross-tenant data access (IDOR), permissions checked only in the frontend, hand-rolled auth, non-idempotent webhooks, and untested backups. Check those first.

## Non-negotiables to keep repeating

These are the ideas people (and agents) most often lose, so restate them whenever relevant:

- **Every tenant query is scoped by `organization_id`, and tested.** Cross-tenant leaks are the signature failure of AI-built SaaS.
- **The server is the source of truth.** Hiding a button is not authorization.
- **Never let the agent edit a test to make it pass.** Bugs get fixed in code.
- **Review with fresh eyes.** Security and code reviews go to a new session or a reviewer subagent, not the one that wrote the code.
- **Use proven providers** for auth and payments; don't hand-roll password hashing or sessions.
- **Write it down.** CLAUDE.md / AGENTS.md is the agent's memory; without it every session starts from zero.

For anything handling payments, health or financial data, note that AI security review doesn't replace a professional audit.

Tool and agent features change quickly; when specifics matter (slash commands, hooks, subagents, MCP), suggest checking the current Claude Code docs at https://docs.claude.com/en/docs/claude-code/overview or OpenAI's Codex docs.
