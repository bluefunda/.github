# AGENTS.md

This is the `bluefunda/.github` repo — org-wide CI/CD configuration for all bluefunda repositories.

## What lives here

| Path | Purpose |
|---|---|
| `.github/workflows/` | Reusable workflows consumed by all bluefunda repos |
| `.github/PULL_REQUEST_TEMPLATE.md` | Org-wide PR template |
| `.claude/commands/go-review.md` | `/go-review` slash command — idiomatic Go audit prompt |
| `.claude/commands/ts-review.md` | `/ts-review` slash command — idiomatic TypeScript audit prompt |
| `.github/actions/versions/` | Org version pins (Go toolchain, base images) and the composite action that exposes them |
| `.github/actions/setup-go/` | Composite action used by the Go workflows: pinned Go, `GOTOOLCHAIN=local`, private modules, optional protoc; safe on self-hosted runners |
| `docs/versions.md` | How the version pins work and how to upgrade them |
| `docs/dockerfile.md` | Dockerfile standard: non-root runtime, pinned bases, `GH_PAT` handling, gitops runtime defaults |
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
- `go-ci.yml` — build, test, lint with the pinned Go version; use `goprivate: github.com/bluefunda/*` for repos with private deps, `runner: self-hosted` for private repos
- `ts-review.yml` — posts advisory TypeScript review as PR comment; requires `ANTHROPIC_API_KEY`
- `ts-ci.yml` — typecheck, test, build, lint for TypeScript/Node.js repos
- `release-please.yml` — version bump, CHANGELOG, release PR; outputs `release_created` + `tag_name`; `auto-merge: true` approves and auto-merges the release PR
- `docker-deploy.yml` — build image, push GHCR, update gitops compose file for Komodo; checks the Dockerfile standard (`dockerfile-policy: warn|enforce|off`, default `warn`)
- `github-release-notes.yml` — generate PR-based release notes via the release-foundry tool (release-foundry is the tool; its workflow copies forward here)
- `go-binary-release.yml` — GoReleaser + Homebrew tap for CLI tools; `goreleaser-runner: macos-latest` to sign/notarize darwin binaries (no Docker on macOS)
- `pr-title-check.yml` — enforce conventional commit titles

## Conventions

- Do not duplicate workflow logic across repos — add it here and reference via `workflow_call`
- Every workflow must work on self-hosted runners too: no `sudo`, no `git config --global` with tokens, per-job paths under `$RUNNER_TEMP`, rootless Docker
- The `/go-review` prompt in `.claude/commands/go-review.md` is fetched at runtime by `go-review.yml` — keep it up to date here
- The `/ts-review` prompt in `.claude/commands/ts-review.md` is fetched at runtime by `ts-review.yml` — keep it up to date here
- Go and base image versions are pinned once in `.github/actions/versions/Dockerfile` ([docs/versions.md](docs/versions.md)); never hard-code them in a workflow or repo
- Dockerfiles must follow [docs/dockerfile.md](docs/dockerfile.md) — non-root runtime user, pinned base images, `GH_PAT` only via `.netrc` in the builder stage
- `profile/README.md` is public-facing; keep it product-focused, not engineering-internal
