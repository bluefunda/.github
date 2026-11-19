# Dockerfile standard

Applies to every bluefunda image built by [`docker-deploy.yml`](../.github/workflows/docker-deploy.yml).
The workflow checks the first two rules on every release (see [Enforcement](#enforcement)). Repos still deployed with release-foundry's copy of `docker-deploy.yml` aren't checked yet ([#32](https://github.com/bluefunda/.github/issues/32)).

## Rules

1. **The runtime runs as a non-root user.**
   - Static binary that needs nothing else at runtime → `gcr.io/distroless/static:nonroot` (uid 65532).
   - Anything that needs a shell, CA bundle tweaks, `git`, etc. → Alpine with `appuser`, uid/gid 1000, and `USER appuser`.
   - The final `USER` must not be empty, `root` or `0`. Do not drop privileges in an entrypoint script (`su-exec`, `gosu`): the image metadata then still says root and the check fails.
2. **Base images come from the org pins.** `FROM` lines use the `GO_IMAGE`, `ALPINE_IMAGE` and `DISTROLESS_IMAGE` build args, which `docker-deploy.yml` sets from [the org version pins](versions.md). Their `ARG` defaults only matter for local builds and must still be pinned to a version, never `:latest` or untagged.
3. **`GH_PAT` is used only in the builder stage, through `.netrc`.** Write `.netrc` and delete it in the same `RUN`, so the token is in no layer. `url.insteadOf` with the token in the URL no longer works: libcurl 8.23 drops URL credentials across GitHub's redirect.
4. **No secrets in image layers.** This includes builder stages: `docker-deploy.yml` exports every layer to the GHA cache (`mode=max`).
5. **Writable state lives under one documented directory** (`/data` by default), mounted as a volume and owned by the runtime user. Everything else is read-only (see [Runtime defaults](#runtime-defaults-gitops-stacks)).

## Reference Dockerfiles

The `ARG` defaults only matter for local `docker build`; in CI, `docker-deploy.yml` overrides them with the org pins (tag and digest).

### Static binary → distroless

```dockerfile
ARG GO_IMAGE=golang:1.27.4-alpine
ARG DISTROLESS_IMAGE=gcr.io/distroless/static:nonroot

FROM ${GO_IMAGE} AS builder
ARG GH_PAT
ENV GOPRIVATE=github.com/bluefunda/* CGO_ENABLED=0
RUN apk add --no-cache git
WORKDIR /src
COPY go.mod go.sum ./
RUN if [ -n "$GH_PAT" ]; then \
      printf 'machine github.com\nlogin x-access-token\npassword %s\n' "$GH_PAT" > /root/.netrc; \
    fi \
 && go mod download \
 && rm -f /root/.netrc
COPY . .
RUN go build -trimpath -ldflags='-s -w' -o /out/app ./cmd/app

FROM ${DISTROLESS_IMAGE}
COPY --from=builder /out/app /app
USER nonroot:nonroot
ENTRYPOINT ["/app"]
```

### Needs a userland → Alpine + `appuser`

```dockerfile
ARG GO_IMAGE=golang:1.27.4-alpine
ARG ALPINE_IMAGE=alpine:3.24

# Builder stage as above.

FROM ${ALPINE_IMAGE}
RUN apk add --no-cache ca-certificates git \
 && addgroup -g 1000 appuser \
 && adduser -D -u 1000 -G appuser appuser \
 && mkdir /data && chown appuser:appuser /data
COPY --from=builder /out/app /usr/local/bin/app
USER appuser
VOLUME /data
ENTRYPOINT ["/usr/local/bin/app"]
```

## Enforcement

`docker-deploy.yml` takes a `dockerfile-policy` input:

| Value | Behaviour |
|---|---|
| `warn` (default) | Violations are reported as workflow warnings; the image still ships. |
| `enforce` | Violations fail the job before anything is pushed. |
| `off` | Checks skipped. |

Before pushing, the workflow:

1. Runs [hadolint](https://github.com/hadolint/hadolint) on the Dockerfile. DL3002 (last `USER` is root), DL3006 (untagged image) and DL3007 (`:latest`) are policy violations; other findings are shown as notices. A `.hadolint.yaml` in the repo root is honoured.
2. Builds the image for the runner's platform and checks that `docker inspect --format '{{.Config.User}}'` is not empty, `root` or `0`. This catches a missing `USER`, which hadolint does not flag.

The default moves to `enforce` once existing repos comply.

## Runtime defaults (gitops stacks)

Every service in a gitops `compose.yaml` should set:

```yaml
services:
  app:
    image: ghcr.io/bluefunda/app:v1.2.3
    read_only: true
    cap_drop: [ALL]
    security_opt:
      - no-new-privileges:true
    tmpfs:
      - /tmp
    volumes:
      - app-data:/data   # only if the service keeps state
```

Add back individual capabilities (`cap_add`) only with a comment saying why.
