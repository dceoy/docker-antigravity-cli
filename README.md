# docker-antigravity-cli

Dockerfile and compose.yml for [Antigravity CLI](https://antigravity.google/docs/cli-overview)

[![CI/CD](https://github.com/dceoy/docker-antigravity-cli/actions/workflows/ci.yml/badge.svg)](https://github.com/dceoy/docker-antigravity-cli/actions/workflows/ci.yml)

This repository provides a Docker Compose workflow that:

- installs [Antigravity CLI](https://antigravity.google/docs/cli-overview) (`agy`) during the image build
- runs the container as a non-root `agent` user by default
- mounts the current repository at `/workspace`
- persists Gemini and application state in named Docker volumes

## Included tools

- [Antigravity CLI](https://antigravity.google/docs/cli-overview) (`agy`)
- [GitHub CLI](https://cli.github.com/) (`gh`)
- git, jq, npm, pipx, python3-pip, ripgrep, rsync, tree, unzip, uv, vim, wget, zsh
- [Oh My Zsh](https://ohmyz.sh/)
- [print-github-tags](https://github.com/dceoy/print-github-tags)

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) with the Compose plugin
- A valid Google credential — one of:
  - `GEMINI_API_KEY`
  - `GOOGLE_API_KEY`
  - `GOOGLE_APPLICATION_CREDENTIALS` (path to a service-account JSON file)

## Quick start

1. Export your Google credentials:

   ```bash
   export GEMINI_API_KEY=your_key_here
   ```

2. Build the image:

   ```bash
   docker compose build
   ```

3. Start an interactive Antigravity CLI session:

   ```bash
   docker compose run --rm antigravity-cli
   ```

   The service starts in `/workspace` and runs:

   ```bash
   zsh -lc 'agy --dangerously-skip-permissions'
   ```

## Common commands

Override the default command to run a specific tool inside the container:

```bash
docker compose run --rm antigravity-cli -lc 'agy --version'
docker compose run --rm antigravity-cli -lc 'gh auth status'
docker compose run --rm antigravity-cli -lc 'git status'
```

## Runtime layout

- The repository root is mounted to `/workspace`.
- Gemini state is persisted in the `gemini-data` volume at `/home/agent/.gemini`.
- Application config is persisted in the `config-data` volume at `/home/agent/.config`.
- The Compose service repairs volume ownership on startup when previously created as `root:root`.
- Build args `USER_NAME`, `USER_UID`, and `USER_GID` default to `agent`, `1001`, and `1001`.

## Environment variables

| Variable | Required | Description |
|---|---|---|
| `GEMINI_API_KEY` | One of these three | Authenticate with the Gemini API |
| `GOOGLE_API_KEY` | One of these three | Alternative Google API key |
| `GOOGLE_APPLICATION_CREDENTIALS` | One of these three | Path to a service-account JSON file for ADC |
| `GITHUB_TOKEN` | No | Authenticate `gh` CLI operations |

## Related projects

- [dceoy/docker-copilot-cli](https://github.com/dceoy/docker-copilot-cli) — the same Docker Compose pattern for GitHub Copilot CLI
