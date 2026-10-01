# Part 0: Set up the agent before layer one

Most bad AI-built SaaS apps fail here, before a single feature is written. An agent with no project memory makes fresh, inconsistent decisions every session.

## 0.1 Project memory file

- Claude Code reads `CLAUDE.md` in the repo root at the start of every session.
- Codex reads `AGENTS.md` the same way.
- Keep one file and symlink or copy it so both agents share the same rules (e.g. `ln -s CLAUDE.md AGENTS.md`).

Use `assets/CLAUDE.md.template` as the starting point. When helping someone fill it in, ask for (or infer) product description, stack, tenancy model and commands, and produce a completed file rather than a blank template.

Update it every time a decision is made. It's the single highest-leverage habit in agentic coding.

## 0.2 The working loop

1. Plan first, code second. Claude Code: plan mode. Codex: ask for a plan and say not to write code yet.
2. Review the plan. Catch bad decisions cheaply.
3. Implement in small steps, on a branch.
4. Make the agent verify its own work: tests, lint, typecheck, with output shown.
5. Review the diff yourself before merging.
6. Record the decision in CLAUDE.md / AGENTS.md or `/docs/decisions/`.

## 0.3 Agent features to lean on

- **Custom slash commands / reusable prompts.** Save prompts from this playbook as files (Claude Code: `.claude/commands/`) so `/security-review` replaces pasting. Offer to generate these files.
- **Subagents.** Claude Code can delegate to specialised subagents, e.g. a "reviewer" that only critiques. Useful for security and test reviews.
- **Hooks.** Run commands automatically on events, e.g. lint after every file edit.
- **MCP servers.** Connect the agent to real tools (database, GitHub, error tracker, docs) so it works from real data.

Features change quickly: check https://docs.claude.com/en/docs/claude-code/overview and OpenAI's Codex docs.

## 0.4 The universal "don't fool me" prompt

Append to any big task:

```
Before you say this is done:
1. Run lint, typecheck and the full test suite, and show me the output.
2. List every file you changed and why.
3. List anything you skipped, stubbed, mocked or hard-coded.
4. List assumptions you made that I should confirm.
Do not describe work as complete if any test is failing or skipped.
```
