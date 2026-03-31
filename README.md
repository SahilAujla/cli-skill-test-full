# CLI Skill Test — Thin Skill + Full Skills Context

This repo tests whether an agent with the thin `alchemy-cli` skill PLUS the full Alchemy skills context (API references, gateway docs, recipes, operational guides) performs better than the thin skill alone.

## Setup

1. Make sure `@alchemy/cli` is installed: `npm i -g @alchemy/cli`
2. Make sure `ALCHEMY_API_KEY` is set in your environment
3. Open this repo in Cursor (or your agent platform)
4. Start a new agent session

## Run the test

Tell the agent:

> Read TEST_PLAN.md and run every task in order. Record your results in results/results.md using the format specified in the test plan.

## What's in this repo

- `skills/alchemy-cli/SKILL.md` — the thin CLI skill
- `skills/alchemy-api/` — full API skill with 63 reference files
- `skills/agentic-gateway/` — gateway skill with protocol rules
- `skills/alchemy-codex/` — Codex router skill
- `.cursor/rules/alchemy.md` — tells the agent to use all skills
- `TEST_PLAN.md` — 27 tasks to complete (same as the thin repo)
- `results/results.md` — where results get recorded
