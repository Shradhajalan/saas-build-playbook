# SaaS Build Playbook (Claude skill)

A Claude skill for taking a vibe-coded app to a production-grade SaaS with agentic coders like Claude Code or OpenAI Codex.

It covers 15 layers in build order: system design, architecture, database and storage, auth and permissions, APIs, frontend, CI/CD, testing, hosting, security, rate limiting, caching, logging, monitoring and scaling. Each layer comes with what agents typically get wrong, the decisions you own, copy-paste prompts, and a verification checklist.

## Structure

```
SKILL.md                      Workflow, layer map, and routing
references/
  00-agent-setup.md           CLAUDE.md / AGENTS.md, working loop, "don't fool me" prompt
  01-plan.md                  Layers 1–2
  02-foundation.md            Layers 3–5
  03-product-and-ship.md      Layers 6–9
  04-harden.md                Layers 10–12
  05-operate-and-grow.md      Layers 13–15 and beyond
  checklists.md               Golden rules and pre-launch checklist
assets/
  CLAUDE.md.template          Project memory file starter
```

## Install

**Claude.ai:** download `saas-build-playbook.skill` from the [latest release](https://github.com/Shradhajalan/saas-build-playbook/releases/latest) and upload it under Settings → Capabilities → Skills.

**Claude Code:** copy this folder into `~/.claude/skills/saas-build-playbook/` (personal) or `.claude/skills/saas-build-playbook/` in a project.

## Usage

Once installed, it triggers on things like:

- "What am I missing before I launch my SaaS?"
- "Give me Claude Code prompts to add multi-tenant auth with Supabase"
- "Write a CLAUDE.md for my Next.js + Stripe project"
- "How do I make my Razorpay webhooks idempotent?"

## License

MIT. See [LICENSE](LICENSE).
