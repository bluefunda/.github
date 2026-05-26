# AGENTS.md

This is the `bluefunda/.github` repo — org-wide CI/CD configuration for all bluefunda repositories.

## What lives here

| Path | Purpose |
|---|---|
| `.github/workflows/` | Reusable workflows consumed by all bluefunda repos |
| `.github/PULL_REQUEST_TEMPLATE.md` | Org-wide PR template |
| `.claude/commands/go-review.md` | `/go-review` slash command — idiomatic Go audit prompt |
| `CODEOWNERS` | `* @bluefunda/maintainers` |
| `profile/README.md` | Public org profile on github.com/bluefunda |

## Pipeline

See [README.md](README.md) for the full end-to-end pipeline diagram.

```
PR → go-review (advisory) + pr-title-check
Merge → go-ci
Release → docker-deploy → gitops (Komodo) + github-release-notes
```

## Available workflows (consumed via `uses: bluefunda/.github/.github/workflows/<name>.yml@main`)

- `go-review.yml` — posts advisory Claude Code review as PR comment; requires `ANTHROPIC_API_KEY`
- `go-ci.yml` — build, test, lint; use `goprivate: github.com/bluefunda/*` for repos with private deps
- `release-please.yml` — version bump, CHANGELOG, release PR; outputs `release_created` + `tag_name`
- `docker-deploy.yml` — build image, push Docker Hub, update gitops compose file for Komodo
- `github-release-notes.yml` — generate PR-based release notes via release-foundry binary
- `go-binary-release.yml` — GoReleaser + Homebrew tap for CLI tools
- `pr-title-check.yml` — enforce conventional commit titles

## Conventions

- Do not duplicate workflow logic across repos — add it here and reference via `workflow_call`
- The `/go-review` prompt in `.claude/commands/go-review.md` is fetched at runtime by `go-review.yml` — keep it up to date here
- `profile/README.md` is public-facing; keep it product-focused, not engineering-internal
