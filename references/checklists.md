# Golden rules & pre-launch checklist

## Golden rules of agentic coding for SaaS

1. **You are the architect; the agent is the builder.** Decide, then delegate.
2. **Plan before code, every time.** Fixing a plan is cheap; fixing code is expensive.
3. **Write it down.** CLAUDE.md / AGENTS.md and /docs/ are the agent's memory. Without them every session starts from zero.
4. **Small tasks, small diffs.** One feature per branch, one concern per commit.
5. **The agent must prove it works.** Tests, lint, typecheck, output shown.
6. **Never let it edit a test to pass.** Bugs are fixed in code.
7. **Review with a fresh pair of eyes.** New session or reviewer subagent for security and code review.
8. **Server is the source of truth.** Validation, permissions, limits: always enforced on the backend.
9. **Every tenant query is scoped.** organization_id everywhere, tested.
10. **Read the diff.** If a change isn't understood, ask the agent to explain it before merging.

## Pre-launch checklist

Grouped by severity for gap assessments. Blocking items should be fixed before any real customer data goes in.

### Blocking
- [ ] Cross-tenant data access tested and blocked (Layers 3, 4)
- [ ] Auth via a proven provider; no custom password handling (Layer 4)
- [ ] Every endpoint validates input and checks permissions on the server (Layers 4, 5)
- [ ] Payments driven by verified, idempotent webhooks (Layer 5)
- [ ] Backups enabled and a restore actually tested (Layer 3)
- [ ] Separate staging and production environments and databases (Layer 9)
- [ ] No tenant data cached at shared layers (Layer 12)

### Important
- [ ] CI blocks merges on failing tests; production deploy has rollback (Layer 7)
- [ ] Critical flows covered by end-to-end tests (Layer 8)
- [ ] Security review done; dependencies audited (Layer 10)
- [ ] Rate limits on login, signup and expensive endpoints (Layer 11)
- [ ] Errors reach the error tracker; logs are structured and redacted (Layer 13)
- [ ] Uptime checks and alerts, each with a runbook (Layer 14)
- [ ] Billing alerts set on every provider (Layer 9)
- [ ] Terms of service and privacy policy published (Beyond)

### Before meaningful traffic
- [ ] Load tested at expected launch traffic (Layer 15)

AI wrote the code. The human still holds all of these.
