# Container Image

The image is built automatically from the [MariaDB/mcp](https://github.com/MariaDB/mcp) upstream repository and published to the **GitHub Container Registry (GHCR)**.

## Image Reference

```
ghcr.io/<your-github-user>/mariadb-mcp
```

## Tags

| Tag | Description |
|-----|-------------|
| `latest` | Always the most recent build from `main` |
| `<short-sha>` | Immutable build identified by the 7-character Git commit hash |

## Usage

### Docker

```bash
docker pull ghcr.io/<your-github-user>/mariadb-mcp:latest
```

```bash
docker run -p 9001:9001 \
  -e DB_HOST=<host> \
  -e DB_PORT=3306 \
  -e DB_USER=<user> \
  -e DB_PASSWORD=<password> \
  -e DB_NAME=<database> \
  ghcr.io/<your-github-user>/mariadb-mcp:latest
```

### Docker Compose

```yaml
services:
  mcp:
    image: ghcr.io/<your-github-user>/mariadb-mcp:latest
    ports:
      - "9001:9001"
    environment:
      DB_HOST: mariadb
      DB_PORT: 3306
      DB_USER: root
      DB_PASSWORD: secret
      DB_NAME: mydb
```

## Technical Details

| Property | Value |
|----------|-------|
| Base image | `python:3.11-slim` |
| Exposed port | `9001` |
| Transport | SSE (`--transport sse`) |
| Entrypoint | `python src/server.py --host 0.0.0.0 --transport sse` |
| Build tool | [uv](https://github.com/astral-sh/uv) |

The image uses a **multi-stage build**:
1. **Builder stage** – installs build dependencies, uv, and the Python project into a local `.venv`
2. **Final stage** – lean `python:3.11-slim` containing only the `.venv` and the `src/` directory

## Automatic Update Process

The workflow [`.github/workflows/sync-upstream-and-publish.yml`](.github/workflows/sync-upstream-and-publish.yml) runs **every 6 hours** and performs the following steps:

1. Compares the fork with `upstream/main` (MariaDB/mcp)
2. If new upstream commits exist: merges them into the fork's `main` and pushes
3. Builds a new Docker image and publishes it with the `latest` and `<short-sha>` tags to GHCR

The workflow can also be triggered manually at any time via **Actions → Sync Upstream & Publish Image → Run workflow**.

## Prerequisites (Fork Settings)

The following settings must be configured in the fork for the workflow to function:

- **Settings → Actions → General → Workflow permissions**: enable *Read and write permissions*
- **Settings → Packages**: set the package to *Public* after the first push if public access is desired

No custom secret is required — the workflow exclusively uses the automatically provided `GITHUB_TOKEN`.
