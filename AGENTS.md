# AGENTS.md

Guidelines for AI coding agents working in this repository.

## Project overview

This repository provides a Docker Compose workflow for running
[Antigravity CLI](https://antigravity.google/docs/cli-overview) (`agy`) — Google's
AI-powered coding assistant — inside an isolated, reproducible container.
The container runs as a non-root `agent` user, mounts the current repository at
`/workspace`, and persists Gemini and application state in named Docker volumes.

## Repository structure

```
.
├── Dockerfile        # Multi-stage build: base → cli → antigravity
├── compose.yml       # Docker Compose service definition
├── .github/
│   ├── dependabot.yml
│   └── renovate.json
├── AGENTS.md         # This file
└── README.md
```

## Development workflow

Use Docker Compose for all local workflows.

Build the image:

```bash
docker compose build
```

Start an interactive Antigravity CLI session:

```bash
docker compose run --rm antigravity-cli
```

Run a one-off command without entering a session:

```bash
docker compose run --rm antigravity-cli -lc 'agy --version'
```

## Code standards

- **YAML** — 2-space indentation, lowercase keys.
- **Dockerfile** — uppercase instructions; all `RUN` layers use
  `bash -euo pipefail` for strict error handling.
- **Shell scripts** — use `#!/usr/bin/env bash` with `set -euo pipefail`.
- Keep each Dockerfile `RUN` layer focused on a single logical step to
  maximise layer caching.

## Testing approach

There are no unit tests. Validation is operational:

1. `docker compose build` — confirms the image builds cleanly.
2. `docker compose run --rm antigravity-cli -lc 'agy --version'` — smoke-tests
   the installed CLI.
3. CI runs Hadolint (Dockerfile linter) and other static checks automatically.

Run local QA before opening a pull request.

## Security

- **Never commit API keys or tokens.** Supply credentials exclusively through
  environment variables at runtime (see `compose.yml` for the supported set).
- The following secrets are consumed from the host environment and forwarded
  into the container:

  | Variable | Purpose |
  |---|---|
  | `GEMINI_API_KEY` | Authenticate with the Gemini API |
  | `GOOGLE_API_KEY` | Alternative Google API key |
  | `GOOGLE_APPLICATION_CREDENTIALS` | Path to a service-account JSON file for ADC |
  | `GITHUB_TOKEN` | Authenticate `gh` CLI operations |

- Only one of `GEMINI_API_KEY`, `GOOGLE_API_KEY`, or
  `GOOGLE_APPLICATION_CREDENTIALS` is required; the others may be left unset.

## Contribution guidelines

- Write clear, descriptive commit messages explaining *why* a change was made.
- Reference related issues or pull requests in the PR description.
- Document the local validation steps you ran (build, smoke test, QA) in the
  PR body.
- Keep changes minimal — avoid unrelated refactors in the same PR.
