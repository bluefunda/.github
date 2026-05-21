# CLAUDE.md

Org-wide CI/CD configuration repo for bluefunda. See [AGENTS.md](AGENTS.md) for pipeline context and available workflows.

## Conventions

- Conventional Commits for all commit messages and PR titles
- Workflow changes here propagate to all bluefunda repos immediately — test carefully
- Use `/go-review` to audit any Go code changes in this repo

## Key files

- `.github/workflows/` — reusable workflows; keep `workflow_call` interface stable
- `.claude/commands/go-review.md` — the `/go-review` prompt; also fetched by CI at runtime
- `profile/README.md` — public org profile; product-focused language only
