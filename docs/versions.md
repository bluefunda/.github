# Org version pins

The Go toolchain and the base images used by every bluefunda build are pinned in one file:
[`.github/actions/versions/Dockerfile`](../.github/actions/versions/Dockerfile).

| Stage | Pins | Used by |
|---|---|---|
| `go` | Go builder image, tag + digest. Its tag also sets the Go toolchain version. | `go-ci`, `go-binary-release`, `docker-deploy` |
| `alpine` | Alpine runtime image, tag + digest | `docker-deploy` |
| `distroless` | `gcr.io/distroless/static:nonroot`, digest | `docker-deploy` |

The file is never built. It's a Dockerfile so that Dependabot (weekly, docker ecosystem) opens PRs to bump the tags and digests.

## Upgrading

Merge the Dependabot PR, or edit the file by hand for a minor Go version. **That is the only change needed.** Every repo's next CI run and next image build uses the new pins. Running containers don't change until their image is rebuilt.

Check before merging a Go minor bump:
- `golangci-lint-version` in `go-ci.yml` must support the new Go version.
- `template-go` CI passes against the new pins.

## How workflows use the pins

The [`versions`](../.github/actions/versions/action.yml) composite action reads the file and outputs `go-version`, `go-image`, `alpine-image` and `distroless-image`. The reusable workflows call it at `@main`:

- **`go-ci`, `go-binary-release`:** `setup-go` installs the pinned Go version, and `GOTOOLCHAIN=local` stops Go from downloading a different toolchain. A `go-version` input still overrides the pin.
- **`docker-deploy`:** passes the build args `GO_IMAGE`, `ALPINE_IMAGE`, `DISTROLESS_IMAGE` and `GO_VERSION` (bare version, kept for existing Dockerfiles), and records the pins in the image labels `com.bluefunda.pins.*`.

Dockerfiles consume the build args as described in [the Dockerfile standard](dockerfile.md).

## `go.mod`

The `go` line in each repo's `go.mod` is that module's minimum Go version. It doesn't need to match the pin, but it must be **at or below** it. If a dependency upgrade raises it above the pin, CI fails with `go.mod requires go >= X (running go Y; GOTOOLCHAIN=local)`. Bump the pin here first.

## Not covered yet

Repos deployed with release-foundry's `docker-deploy.yml` don't receive these build args until [#32](https://github.com/bluefunda/.github/issues/32) is settled.
