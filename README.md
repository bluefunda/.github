# bluefunda/.github

Org-wide GitHub configuration: reusable workflows, PR templates, Claude Code commands, and branch protection defaults.

---

## Pipeline Overview

Every bluefunda Go repo follows this end-to-end flow:

```
PR opened
  └─ go-review.yml          advisory Claude Code review posted as PR comment (non-blocking)
  └─ pr-title-check.yml     conventional commit format enforced

PR merged to main
  └─ go-ci.yml              build, test (race detector), lint

Push to main
  └─ release-please.yml     bumps version, updates CHANGELOG, opens release PR

Release PR merged
  └─ docker-deploy.yml      builds image → pushes to GHCR → updates gitops repo
  └─ github-release-notes.yml   generates structured release notes → populates GitHub release body
```

The gitops repo is watched by [Komodo](https://komo.do) — it deploys automatically when the compose file is updated.

---

## Reusable Workflows

All workflows are in [`.github/workflows/`](.github/workflows/) and consumed via:

```
uses: bluefunda/.github/.github/workflows/<name>.yml@main
```

| Workflow | Trigger point | Purpose |
|---|---|---|
| `go-review.yml` | PR opened / updated | Advisory Claude Code review comment |
| `go-ci.yml` | PR + push to main | Build, test, lint |
| `release-please.yml` | Push to main | Version bump + CHANGELOG + release PR |
| `docker-deploy.yml` | After release | Build image, push GHCR, update gitops |
| `github-release-notes.yml` | After release | Generate release notes via release-foundry |
| `go-binary-release.yml` | After release | GoReleaser binary + Homebrew tap |
| `pr-title-check.yml` | PR opened / updated | Enforce conventional commit titles |

---

## Setting Up a New Repo

Copy the workflow stubs from [`template-go`](https://github.com/bluefunda/template-go) — they are pre-wired to call the workflows above.

```
.github/workflows/
  ci.yml              → go-ci.yml
  go-review.yml       → go-review.yml
  pr-title-check.yml  → pr-title-check.yml
  release-please.yml  → release-please.yml + github-release-notes.yml chain
                        (docker-deploy job commented out — uncomment if deploying via Komodo)
```

Required org secrets (already configured for private repos):

| Secret | Used by |
|---|---|
| `GH_PAT` | All workflows that push commits or access private modules |
| `ANTHROPIC_API_KEY` | `go-review.yml` |

---

## Claude Code Commands

[`.claude/commands/go-review.md`](.claude/commands/go-review.md) — the `/go-review` slash command prompt. Available in any Claude Code session opened in a bluefunda repo. Also fetched at runtime by the `go-review.yml` workflow so the CI review always uses the latest version.

---

## Other Files

| File | Purpose |
|---|---|
| `CODEOWNERS` | `* @bluefunda/maintainers` — applies to all repos |
| `.github/PULL_REQUEST_TEMPLATE.md` | Org-wide PR template |
| `.github/ISSUE_TEMPLATE/` | Issue templates |
| `profile/README.md` | Public org profile shown on github.com/bluefunda |
