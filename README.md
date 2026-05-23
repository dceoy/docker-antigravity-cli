# docker-antigravity-cli

Dockerfile and compose.yml for [Antigravity CLI](https://antigravity.google/docs/cli-overview)

[![CI/CD](https://github.com/dceoy/docker-antigravity-cli/actions/workflows/ci.yml/badge.svg)](https://github.com/dceoy/docker-antigravity-cli/actions/workflows/ci.yml)

This repository provides a Docker Compose workflow that:

- installs [Antigravity CLI](https://antigravity.google/docs/cli-overview) (`agy`) during the image build
- runs the container as a non-root `agent` user by default
- mounts the current repository at `/workspace`
- persists Antigravity CLI and application state in named Docker volumes

## Included tools

- [Antigravity CLI](https://antigravity.google/docs/cli-overview) (`agy`)
- [GitHub CLI](https://cli.github.com/) (`gh`)
- git, jq, npm, pipx, python3-pip, ripgrep, rsync, tree, unzip, uv, vim, wget, zsh
- [Oh My Zsh](https://ohmyz.sh/)
- [print-github-tags](https://github.com/dceoy/print-github-tags)

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) with the Compose plugin
- A Google account (sign-in happens interactively on first launch — see
  [Authentication](#authentication) below)

## Quick start

1. Build the image:

   ```bash
   docker compose build
   ```

2. Start an interactive Antigravity CLI session:

   ```bash
   docker compose run --rm antigravity-cli
   ```

   The service starts in `/workspace` and runs:

   ```bash
   zsh -lc 'agy --dangerously-skip-permissions'
   ```

## Authentication

`agy` authenticates via Google Sign-In, falling back to the system keyring for
subsequent sessions. Inside the container, the SSH/headless flow is used: on
first launch, `agy` prints an authorization URL — open it on your local
machine, complete the sign-in, and the CLI captures the resulting token.
Tokens are persisted in `/home/agent/.config` via the `config-data` Docker
volume, so you only need to sign in once per volume lifetime. Run `/logout`
from inside `agy` to clear saved credentials.

## Common commands

Override the default command to run a specific tool inside the container:

```bash
docker compose run --rm antigravity-cli -lc 'agy --version'
docker compose run --rm antigravity-cli -lc 'gh auth status'
docker compose run --rm antigravity-cli -lc 'git status'
```

## Runtime layout

- The repository root is mounted to `/workspace`.
- Antigravity CLI state is persisted in the `gemini-data` volume at `/home/agent/.gemini`.
- Application config is persisted in the `config-data` volume at `/home/agent/.config`.
- The Compose service repairs volume ownership on startup when previously created as `root:root`.
- Build args `USER_NAME`, `USER_UID`, and `USER_GID` default to `agent`, `1001`, and `1001`.

## Environment variables

| Variable       | Required | Description                                           |
| -------------- | -------- | ----------------------------------------------------- |
| `GITHUB_TOKEN` | No       | Authenticate `gh` CLI operations inside the container |

`agy` itself does not read API-key environment variables; it relies on OAuth
via the system keyring. See [Authentication](#authentication).

## Related projects

- [dceoy/docker-copilot-cli](https://github.com/dceoy/docker-copilot-cli) — the same Docker Compose pattern for GitHub Copilot CLI
