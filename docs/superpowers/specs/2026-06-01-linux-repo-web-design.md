# Linux Repo Web Design

## Goal

Build an internal-only read-only web portal for company Linux deployment materials. The system will show device installation data, Docker artifacts, APT/Python repository information, shell scripts, and initial setup manuals from one browser UI while also running the backing repository services in Docker Compose.

## Decisions

- The service runs on a Linux server with Docker Compose.
- Access is internal network only. There is no login in the first version.
- The web UI is read-only: users can view, search, download, and copy commands, but cannot edit files.
- Source documents are not edited on the server.
- A separate PC or automation job periodically pushes normalized files to the server.
- Device inventory uses `latest.csv` as the canonical readable data source.
- Excel files may be stored as original reference/download copies, but the API does not depend on parsing Excel.
- The first screen is a dashboard plus top-level tabs.
- Real repository services are included in the Compose stack.

## Architecture

The system is a single Docker Compose platform. The portal and API provide a read-only operational view, while repository functions are handled by dedicated containers.

Services:

- `web`: React/Vite web portal.
- `api`: FastAPI backend for metadata, scanning, status checks, and read-only file information.
- `registry`: Docker registry based on `registry:2`.
- `apt-repo`: APT repository file server and index generator.
- `pypi`: internal Python package index, initially `pypiserver`.
- `static-files`: static download/view service for Dockerfiles, compose files, scripts, and manuals.
- `nginx`: reverse proxy for web, API, static files, APT, and Python package routes.

Persistent areas:

- `data/inbox`: files pushed by the external sync process.
- `data/metadata`: SQLite database and generated metadata.
- `data/registry`: Docker registry storage.
- `data/apt-repo`: generated APT repository.
- `data/pypi`: Python package storage.
- `data/static`: static files served for viewing/downloading.

## User Interface

The first screen is `Dashboard`.

Dashboard content:

- Last successful inventory CSV update time.
- Docker registry status.
- APT and Python repository status.
- Shell script count and last changed time.
- Recent synchronized files.
- Quick copy commands for Docker, APT, pip, and shell scripts.

Top-level tabs:

- `기기 설치목록`
- `Docker`
- `APT / Python`
- `Shell Script`
- `초기 세팅`

### 기기 설치목록

Shows device installation data from `latest.csv`.

Required capabilities:

- Search by device name, IP, ID, production date, location/category if present.
- Filter by columns discovered from the CSV.
- Show detail rows without editing.
- Show CSV last update time.
- Provide original Excel download only if an original file exists.

### Docker

Shows Docker-related materials and registry data.

Required capabilities:

- List images and tags from the internal registry.
- Show versioned Dockerfile files from static storage.
- Show matching `docker-compose.yml` files.
- Provide direct download links.
- Provide copyable `docker pull`, build, and compose commands.

### APT / Python

Shows Linux package repository and Python package information.

Required capabilities:

- Show APT source setup commands.
- List `.deb` packages, versions, sizes, and modified times.
- Show pip index setup/install commands.
- List Python packages from `.whl` and `.tar.gz` files.

### Shell Script

Shows internal shell scripts that are currently served through Apache-like file access.

Required capabilities:

- List scripts.
- Show script content read-only.
- Provide download links.
- Provide copyable execution commands such as `curl -fsSL http://repo.local/scripts/install-agent.sh | bash`.
- No edit or save button.

### 초기 세팅

Shows setup manuals and commands for new machines.

Required capabilities:

- Repository setup guide.
- Docker setup and registry usage.
- APT source registration.
- Python package index usage.
- Recovery or reconfiguration commands.

## Sync And Metadata Flow

The server does not reach into the team's cloud document location. A separate PC or automation job pushes files to the server on a schedule.

Expected inbox layout:

```text
data/inbox/
  inventory/
    latest.csv
    latest.xlsx
  docker/
    <image-name>/
      <version>/
        Dockerfile
        docker-compose.yml
        README.md
  apt/
    packages/
      *.deb
  python/
    packages/
      *.whl
      *.tar.gz
  scripts/
    *.sh
  manuals/
    *.md
```

The API scans `data/inbox` on a schedule, initially every five minutes.

Scan behavior:

- Parse `inventory/latest.csv`.
- Preserve the previous valid metadata if a new CSV is invalid.
- Record last success time and last failure details.
- Build file metadata for Dockerfiles, compose files, scripts, and manuals.
- Build package metadata for APT and Python files.
- Expose the metadata through read-only API endpoints.

The first implementation should support scheduled scanning. A read-only manual rescan endpoint can be added if useful, but it must not modify source file contents.

## Repository Services

### Docker Registry

Use `registry:2`.

- Internal address example: `http://repo.local:5000`.
- Supports image push and pull inside the internal network.
- The portal reads registry catalog/tag APIs to show image status.
- Dockerfiles and compose files are stored separately under static files, not inside the registry.

### APT Repository

Use a simple file-based APT repository first.

- External sync pushes `.deb` files into `data/inbox/apt/packages`.
- An index rebuild job generates the APT package index.
- Clients use setup commands shown in the portal.
- The portal shows package metadata from generated scan results.

### Python Package Index

Use `pypiserver` first.

- External sync pushes `.whl` and `.tar.gz` files.
- Clients use pip commands shown in the portal.
- The portal shows package names, versions, files, and install examples.

## Project Structure

```text
linux_repo_web/
  docker-compose.yml
  .env.example
  README.md

  apps/
    web/
      src/
      package.json
      Dockerfile

    api/
      app/
        main.py
        config.py
        routers/
        services/
        models/
      tests/
      pyproject.toml
      Dockerfile

  infra/
    nginx/
      nginx.conf
    apt/
      Dockerfile
      rebuild-index.sh
    pypi/
      README.md
    registry/
      config.yml

  data/
    inbox/
    metadata/
    registry/
    pypi/
    apt-repo/
    static/

  docs/
    operations/
    superpowers/
      specs/
      plans/
```

## Security And Operations

Security assumptions:

- Internal network only.
- No public internet exposure.
- Firewall should allow only approved company network ranges.
- No login or role system in the first version.
- No browser-side editing of scripts, CSV, Dockerfiles, packages, or manuals.

Operations:

- Run with `docker compose up -d`.
- Use server directories or Docker volumes for persistent `data` paths.
- External sync should write files atomically where possible.
- Failed scans keep the previous valid metadata.
- Dashboard shows last sync and scan status.

## Testing Strategy

API tests:

- CSV parsing with a valid sample.
- CSV parsing failure keeps previous metadata.
- File scanning for Dockerfile and compose layouts.
- Script listing and content retrieval.
- Package metadata extraction for sample APT and Python files.
- Repository status endpoints.

Frontend tests:

- Dashboard status cards render.
- Top-level tabs render.
- Inventory search and filters work from sample API data.
- Command copy components render correct commands.
- Script viewer is read-only.

Integration checks:

- `docker compose up` starts all services.
- Web portal loads.
- API health endpoint responds.
- Docker registry endpoint responds.
- APT repository route responds.
- Python package route responds.
- Static script/manual routes respond.

## Open Implementation Notes

- The first implementation should not add authentication.
- The first implementation should not edit synchronized files.
- The API should prefer standard-library CSV parsing before adding heavier dependencies.
- SQLite is enough for metadata until there is a proven need for a separate database service.
- The current local Excel copy appears not to be a standard readable `.xlsx`; this supports using CSV as the canonical server input.
