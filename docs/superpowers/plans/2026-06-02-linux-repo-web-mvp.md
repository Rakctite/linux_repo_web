# Linux Repo Web MVP Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the first working internal read-only repository portal with Docker Compose, a FastAPI metadata API, a React/Vite web UI, sample synchronized files, and backing repository services.

**Architecture:** The MVP uses Docker Compose to run `web`, `api`, `nginx`, `registry`, `pypiserver`, and a simple APT static repository path. The API scans `data/inbox`, writes read-only metadata into SQLite, and exposes JSON endpoints consumed by the web UI. The web UI shows Dashboard, Inventory, Docker, APT/Python, Shell Script, and Initial Setup tabs.

**Tech Stack:** Python 3.12, FastAPI, SQLite, pytest, React, Vite, TypeScript, Vitest, Nginx, Docker Compose, `registry:2`, `pypiserver`.

---

## File Structure

Create this structure during implementation:

```text
linux_repo_web/
  .gitattributes
  .gitignore
  .env.example
  README.md
  docker-compose.yml

  apps/
    api/
      Dockerfile
      pyproject.toml
      app/
        __init__.py
        config.py
        database.py
        main.py
        models.py
        routers/
          __init__.py
          dashboard.py
          inventory.py
          repository.py
          scripts.py
        services/
          __init__.py
          file_scanner.py
          inventory_csv.py
          metadata_store.py
          repository_status.py
      tests/
        conftest.py
        fixtures/
          inventory.csv
        test_dashboard_api.py
        test_file_scanner.py
        test_inventory_csv.py
        test_metadata_store.py

    web/
      Dockerfile
      index.html
      package.json
      tsconfig.json
      vite.config.ts
      src/
        App.tsx
        main.tsx
        styles.css
        api/
          client.ts
        components/
          CommandBox.tsx
          Layout.tsx
          StatusCard.tsx
        pages/
          AptPythonPage.tsx
          DashboardPage.tsx
          DockerPage.tsx
          InitialSetupPage.tsx
          InventoryPage.tsx
          ScriptsPage.tsx
      src/__tests__/
        App.test.tsx

  infra/
    nginx/
      nginx.conf
    registry/
      config.yml
    apt/
      rebuild-index.sh
    pypi/
      README.md

  samples/
    inbox/
      inventory/
        latest.csv
      docker/
        edge-agent/
          1.0.0/
            Dockerfile
            docker-compose.yml
            README.md
      apt/
        packages/
          README.md
      python/
        packages/
          README.md
      scripts/
        install-agent.sh
      manuals/
        initial-setup.md

  docs/
    operations/
      sync-flow.md
```

## Task 1: Repository Hygiene And Sample Inbox

**Files:**
- Create: `.gitattributes`
- Modify: `.gitignore`
- Create: `.env.example`
- Create: `README.md`
- Create: `samples/inbox/inventory/latest.csv`
- Create: `samples/inbox/docker/edge-agent/1.0.0/Dockerfile`
- Create: `samples/inbox/docker/edge-agent/1.0.0/docker-compose.yml`
- Create: `samples/inbox/docker/edge-agent/1.0.0/README.md`
- Create: `samples/inbox/apt/packages/README.md`
- Create: `samples/inbox/python/packages/README.md`
- Create: `samples/inbox/scripts/install-agent.sh`
- Create: `samples/inbox/manuals/initial-setup.md`

- [ ] **Step 1: Add failing repository checks**

Run:

```bash
git status --short
test -f .gitattributes
test -f .env.example
test -f samples/inbox/inventory/latest.csv
```

Expected: at least one `test` command fails because these files do not exist.

- [ ] **Step 2: Create `.gitattributes`**

```text
* text=auto
*.sh text eol=lf
*.md text eol=lf
*.py text eol=lf
*.ts text eol=lf
*.tsx text eol=lf
*.yml text eol=lf
*.yaml text eol=lf
```

- [ ] **Step 3: Update `.gitignore` to keep operational data out but allow samples**

Replace `.gitignore` with:

```gitignore
.superpowers/
data/
*.xlsx
*.xls
*.csv
*.db
*.sqlite
*.sqlite3
.env
node_modules/
__pycache__/
.pytest_cache/
dist/
build/

!samples/**/*.csv
```

- [ ] **Step 4: Create `.env.example`**

```dotenv
REPO_PUBLIC_HOST=repo.local
API_PORT=8000
WEB_PORT=5173
HTTP_PORT=80
DOCKER_REGISTRY_PORT=5000
DATA_DIR=./data
SCAN_INTERVAL_SECONDS=300
```

- [ ] **Step 5: Create `README.md`**

```markdown
# Linux Repo Web

Internal read-only web portal and repository bundle for Linux deployment work.

## MVP scope

- Dashboard for sync and repository status.
- Inventory view from `data/inbox/inventory/latest.csv`.
- Dockerfile and compose file browser.
- Docker registry service.
- APT package file browser and repository route.
- Python package file browser and pypiserver route.
- Shell script and manual viewer.

## Local development

```bash
cp .env.example .env
docker compose up --build
```

The web portal is served through Nginx at `http://localhost`.
The API health endpoint is `http://localhost/api/health`.
The Docker registry endpoint is `http://localhost:5000/v2/`.

## Data flow

External sync jobs write files into `data/inbox`. The API scans that inbox and stores generated metadata in SQLite under `data/metadata`.
```

- [ ] **Step 6: Create sample inventory CSV**

Create `samples/inbox/inventory/latest.csv`:

```csv
device_name,ip_address,device_id,production_date,site,device_type,owner
Edge Gateway A,192.168.10.21,EGW-A-001,2026-03-17,Plant 1,Gateway,T_IoT
Sensor Hub B,192.168.10.22,SHB-002,2026-03-18,Plant 1,Hub,T_IoT
Camera Node C,192.168.20.31,CNC-003,2026-03-19,Plant 2,Camera,T_IoT
```

- [ ] **Step 7: Create sample Docker artifacts**

Create `samples/inbox/docker/edge-agent/1.0.0/Dockerfile`:

```dockerfile
FROM alpine:3.20
RUN apk add --no-cache ca-certificates curl
CMD ["sh", "-c", "echo edge-agent sample && sleep 3600"]
```

Create `samples/inbox/docker/edge-agent/1.0.0/docker-compose.yml`:

```yaml
services:
  edge-agent:
    image: repo.local:5000/edge-agent:1.0.0
    restart: unless-stopped
```

Create `samples/inbox/docker/edge-agent/1.0.0/README.md`:

```markdown
# edge-agent 1.0.0

Sample Docker artifact for the Linux Repo Web MVP.
```

- [ ] **Step 8: Create sample package placeholders, script, and manual**

Create `samples/inbox/apt/packages/README.md`:

```markdown
# APT package drop directory

Place `.deb` files here in production sync jobs.
```

Create `samples/inbox/python/packages/README.md`:

```markdown
# Python package drop directory

Place `.whl` and `.tar.gz` files here in production sync jobs.
```

Create `samples/inbox/scripts/install-agent.sh`:

```sh
#!/usr/bin/env sh
set -eu
echo "Installing sample edge agent"
```

Create `samples/inbox/manuals/initial-setup.md`:

```markdown
# Initial Setup

1. Configure internal DNS for `repo.local`.
2. Register Docker, APT, and Python repository endpoints.
3. Run the required install scripts for the target device.
```

- [ ] **Step 9: Verify and commit**

Run:

```bash
git status --short
git add .gitattributes .gitignore .env.example README.md samples
git commit -m "chore: add repository hygiene and samples"
```

Expected: commit succeeds with the hygiene and sample files.

## Task 2: FastAPI Project Skeleton And Health Endpoint

**Files:**
- Create: `apps/api/pyproject.toml`
- Create: `apps/api/Dockerfile`
- Create: `apps/api/app/__init__.py`
- Create: `apps/api/app/config.py`
- Create: `apps/api/app/main.py`
- Create: `apps/api/tests/conftest.py`
- Create: `apps/api/tests/test_dashboard_api.py`

- [ ] **Step 1: Write failing health endpoint test**

Create `apps/api/tests/test_dashboard_api.py`:

```python
from fastapi.testclient import TestClient

from app.main import create_app


def test_health_endpoint_returns_ok():
    client = TestClient(create_app())

    response = client.get("/health")

    assert response.status_code == 200
    assert response.json() == {"status": "ok"}
```

- [ ] **Step 2: Add API package configuration**

Create `apps/api/pyproject.toml`:

```toml
[project]
name = "linux-repo-web-api"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
  "fastapi==0.115.6",
  "uvicorn[standard]==0.34.0",
  "pydantic-settings==2.7.1"
]

[project.optional-dependencies]
test = [
  "httpx==0.28.1",
  "pytest==8.3.4"
]

[tool.pytest.ini_options]
pythonpath = ["."]
testpaths = ["tests"]
```

Create `apps/api/app/__init__.py`:

```python
"""Linux Repo Web API package."""
```

Create `apps/api/tests/conftest.py`:

```python
from pathlib import Path


def fixture_path(name: str) -> Path:
    return Path(__file__).parent / "fixtures" / name
```

- [ ] **Step 3: Run test to verify it fails**

Run:

```bash
cd apps/api
python -m pip install -e ".[test]"
pytest tests/test_dashboard_api.py::test_health_endpoint_returns_ok -v
```

Expected: FAIL because `app.main` does not exist.

- [ ] **Step 4: Implement config and app factory**

Create `apps/api/app/config.py`:

```python
from functools import lru_cache
from pathlib import Path

from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    inbox_dir: Path = Path("data/inbox")
    metadata_db: Path = Path("data/metadata/app.db")
    static_base_url: str = "/files"
    registry_url: str = "http://registry:5000"
    scan_interval_seconds: int = 300

    model_config = SettingsConfigDict(env_prefix="LINUX_REPO_WEB_")


@lru_cache
def get_settings() -> Settings:
    return Settings()
```

Create `apps/api/app/main.py`:

```python
from fastapi import FastAPI


def create_app() -> FastAPI:
    app = FastAPI(title="Linux Repo Web API")

    @app.get("/health")
    def health() -> dict[str, str]:
        return {"status": "ok"}

    return app


app = create_app()
```

- [ ] **Step 5: Run test to verify it passes**

Run:

```bash
cd apps/api
pytest tests/test_dashboard_api.py::test_health_endpoint_returns_ok -v
```

Expected: PASS.

- [ ] **Step 6: Add API Dockerfile**

Create `apps/api/Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY pyproject.toml ./
RUN pip install --no-cache-dir -e .

COPY app ./app

EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

- [ ] **Step 7: Commit**

Run:

```bash
git add apps/api
git commit -m "feat(api): add FastAPI skeleton"
```

Expected: commit succeeds.

## Task 3: Inventory CSV Parser

**Files:**
- Create: `apps/api/app/models.py`
- Create: `apps/api/app/services/inventory_csv.py`
- Create: `apps/api/tests/fixtures/inventory.csv`
- Create: `apps/api/tests/test_inventory_csv.py`

- [ ] **Step 1: Write parser tests**

Create `apps/api/tests/fixtures/inventory.csv`:

```csv
device_name,ip_address,device_id,production_date,site,device_type,owner
Edge Gateway A,192.168.10.21,EGW-A-001,2026-03-17,Plant 1,Gateway,T_IoT
Sensor Hub B,192.168.10.22,SHB-002,2026-03-18,Plant 1,Hub,T_IoT
```

Create `apps/api/tests/test_inventory_csv.py`:

```python
from pathlib import Path

import pytest

from app.services.inventory_csv import parse_inventory_csv


def test_parse_inventory_csv_returns_columns_and_rows():
    snapshot = parse_inventory_csv(Path("tests/fixtures/inventory.csv"))

    assert snapshot.columns == [
        "device_name",
        "ip_address",
        "device_id",
        "production_date",
        "site",
        "device_type",
        "owner",
    ]
    assert len(snapshot.rows) == 2
    assert snapshot.rows[0]["device_name"] == "Edge Gateway A"
    assert snapshot.rows[0]["ip_address"] == "192.168.10.21"


def test_parse_inventory_csv_rejects_empty_file(tmp_path):
    csv_path = tmp_path / "latest.csv"
    csv_path.write_text("", encoding="utf-8")

    with pytest.raises(ValueError, match="Inventory CSV has no header"):
        parse_inventory_csv(csv_path)
```

- [ ] **Step 2: Run tests to verify failure**

Run:

```bash
cd apps/api
pytest tests/test_inventory_csv.py -v
```

Expected: FAIL because `app.services.inventory_csv` does not exist.

- [ ] **Step 3: Implement models and parser**

Create `apps/api/app/models.py`:

```python
from pydantic import BaseModel


class InventorySnapshot(BaseModel):
    columns: list[str]
    rows: list[dict[str, str]]
    source_path: str
    updated_at: str


class FileItem(BaseModel):
    name: str
    path: str
    relative_path: str
    size: int
    modified_at: str
    kind: str


class DashboardStatus(BaseModel):
    inventory_updated_at: str | None
    inventory_count: int
    docker_artifact_count: int
    script_count: int
    manual_count: int
    apt_package_count: int
    python_package_count: int
    last_scan_status: str
    last_scan_error: str | None
```

Create `apps/api/app/services/__init__.py`:

```python
"""Service modules for Linux Repo Web API."""
```

Create `apps/api/app/services/inventory_csv.py`:

```python
import csv
from datetime import UTC, datetime
from pathlib import Path

from app.models import InventorySnapshot


def parse_inventory_csv(path: Path) -> InventorySnapshot:
    with path.open("r", encoding="utf-8-sig", newline="") as handle:
        reader = csv.DictReader(handle)
        if not reader.fieldnames:
            raise ValueError("Inventory CSV has no header")

        columns = [column.strip() for column in reader.fieldnames if column]
        rows: list[dict[str, str]] = []

        for row in reader:
            normalized = {
                column: (row.get(column) or "").strip()
                for column in columns
            }
            if any(normalized.values()):
                rows.append(normalized)

    stat = path.stat()
    updated_at = datetime.fromtimestamp(stat.st_mtime, tz=UTC).isoformat()
    return InventorySnapshot(
        columns=columns,
        rows=rows,
        source_path=str(path),
        updated_at=updated_at,
    )
```

- [ ] **Step 4: Run parser tests**

Run:

```bash
cd apps/api
pytest tests/test_inventory_csv.py -v
```

Expected: PASS.

- [ ] **Step 5: Commit**

Run:

```bash
git add apps/api/app/models.py apps/api/app/services apps/api/tests
git commit -m "feat(api): parse inventory csv"
```

Expected: commit succeeds.

## Task 4: Inbox File Scanner

**Files:**
- Create: `apps/api/app/services/file_scanner.py`
- Create: `apps/api/tests/test_file_scanner.py`

- [ ] **Step 1: Write scanner tests**

Create `apps/api/tests/test_file_scanner.py`:

```python
from pathlib import Path

from app.services.file_scanner import scan_inbox_files


def test_scan_inbox_files_groups_known_materials(tmp_path):
    (tmp_path / "docker" / "edge-agent" / "1.0.0").mkdir(parents=True)
    (tmp_path / "scripts").mkdir()
    (tmp_path / "manuals").mkdir()
    (tmp_path / "apt" / "packages").mkdir(parents=True)
    (tmp_path / "python" / "packages").mkdir(parents=True)

    (tmp_path / "docker" / "edge-agent" / "1.0.0" / "Dockerfile").write_text("FROM alpine\n", encoding="utf-8")
    (tmp_path / "docker" / "edge-agent" / "1.0.0" / "docker-compose.yml").write_text("services: {}\n", encoding="utf-8")
    (tmp_path / "scripts" / "install-agent.sh").write_text("#!/usr/bin/env sh\n", encoding="utf-8")
    (tmp_path / "manuals" / "initial-setup.md").write_text("# Setup\n", encoding="utf-8")
    (tmp_path / "apt" / "packages" / "agent_1.0.0_amd64.deb").write_bytes(b"deb")
    (tmp_path / "python" / "packages" / "agent-1.0.0-py3-none-any.whl").write_bytes(b"wheel")

    result = scan_inbox_files(tmp_path)

    assert len(result["docker"]) == 2
    assert result["docker"][0].kind == "docker"
    assert len(result["scripts"]) == 1
    assert result["scripts"][0].relative_path == "scripts/install-agent.sh"
    assert len(result["manuals"]) == 1
    assert len(result["apt_packages"]) == 1
    assert len(result["python_packages"]) == 1
```

- [ ] **Step 2: Run test to verify failure**

Run:

```bash
cd apps/api
pytest tests/test_file_scanner.py -v
```

Expected: FAIL because `app.services.file_scanner` does not exist.

- [ ] **Step 3: Implement file scanner**

Create `apps/api/app/services/file_scanner.py`:

```python
from datetime import UTC, datetime
from pathlib import Path

from app.models import FileItem


def _item(path: Path, root: Path, kind: str) -> FileItem:
    stat = path.stat()
    return FileItem(
        name=path.name,
        path=str(path),
        relative_path=path.relative_to(root).as_posix(),
        size=stat.st_size,
        modified_at=datetime.fromtimestamp(stat.st_mtime, tz=UTC).isoformat(),
        kind=kind,
    )


def _scan(root: Path, pattern: str, kind: str) -> list[FileItem]:
    return [
        _item(path, root, kind)
        for path in sorted(root.glob(pattern))
        if path.is_file()
    ]


def scan_inbox_files(inbox_dir: Path) -> dict[str, list[FileItem]]:
    return {
        "docker": _scan(inbox_dir, "docker/**/*", "docker"),
        "scripts": _scan(inbox_dir, "scripts/*.sh", "script"),
        "manuals": _scan(inbox_dir, "manuals/*.md", "manual"),
        "apt_packages": _scan(inbox_dir, "apt/packages/*.deb", "apt_package"),
        "python_packages": _scan(inbox_dir, "python/packages/*", "python_package"),
    }
```

- [ ] **Step 4: Run scanner tests**

Run:

```bash
cd apps/api
pytest tests/test_file_scanner.py -v
```

Expected: PASS.

- [ ] **Step 5: Commit**

Run:

```bash
git add apps/api/app/services/file_scanner.py apps/api/tests/test_file_scanner.py
git commit -m "feat(api): scan synchronized inbox files"
```

Expected: commit succeeds.

## Task 5: SQLite Metadata Store And Rescan Service

**Files:**
- Create: `apps/api/app/database.py`
- Create: `apps/api/app/services/metadata_store.py`
- Create: `apps/api/tests/test_metadata_store.py`

- [ ] **Step 1: Write metadata store tests**

Create `apps/api/tests/test_metadata_store.py`:

```python
from pathlib import Path

from app.services.metadata_store import MetadataStore


def test_metadata_store_records_inventory_and_scan_status(tmp_path):
    db_path = tmp_path / "app.db"
    store = MetadataStore(db_path)
    store.initialize()

    store.save_inventory(
        columns=["device_name", "ip_address"],
        rows=[{"device_name": "Edge Gateway A", "ip_address": "192.168.10.21"}],
        source_path="inventory/latest.csv",
        updated_at="2026-06-02T00:00:00+00:00",
    )
    store.save_scan_status("success", None)

    inventory = store.get_inventory()
    status = store.get_scan_status()

    assert inventory["columns"] == ["device_name", "ip_address"]
    assert inventory["rows"][0]["device_name"] == "Edge Gateway A"
    assert status["status"] == "success"
    assert status["error"] is None


def test_metadata_store_keeps_inventory_after_failed_scan(tmp_path):
    db_path = tmp_path / "app.db"
    store = MetadataStore(db_path)
    store.initialize()

    store.save_inventory(
        columns=["device_name"],
        rows=[{"device_name": "Existing Device"}],
        source_path="inventory/latest.csv",
        updated_at="2026-06-02T00:00:00+00:00",
    )
    store.save_scan_status("failed", "bad csv")

    inventory = store.get_inventory()
    status = store.get_scan_status()

    assert inventory["rows"][0]["device_name"] == "Existing Device"
    assert status["status"] == "failed"
    assert status["error"] == "bad csv"
```

- [ ] **Step 2: Run tests to verify failure**

Run:

```bash
cd apps/api
pytest tests/test_metadata_store.py -v
```

Expected: FAIL because `app.services.metadata_store` does not exist.

- [ ] **Step 3: Implement SQLite store**

Create `apps/api/app/database.py`:

```python
import sqlite3
from pathlib import Path


def connect(db_path: Path) -> sqlite3.Connection:
    db_path.parent.mkdir(parents=True, exist_ok=True)
    connection = sqlite3.connect(db_path)
    connection.row_factory = sqlite3.Row
    return connection
```

Create `apps/api/app/services/metadata_store.py`:

```python
import json
from datetime import UTC, datetime
from pathlib import Path

from app.database import connect


class MetadataStore:
    def __init__(self, db_path: Path):
        self.db_path = db_path

    def initialize(self) -> None:
        with connect(self.db_path) as db:
            db.execute(
                """
                CREATE TABLE IF NOT EXISTS kv (
                    key TEXT PRIMARY KEY,
                    value TEXT NOT NULL
                )
                """
            )

    def _set_json(self, key: str, value: object) -> None:
        with connect(self.db_path) as db:
            db.execute(
                "INSERT OR REPLACE INTO kv (key, value) VALUES (?, ?)",
                (key, json.dumps(value, ensure_ascii=False)),
            )

    def _get_json(self, key: str, default: object) -> object:
        with connect(self.db_path) as db:
            row = db.execute("SELECT value FROM kv WHERE key = ?", (key,)).fetchone()
        if row is None:
            return default
        return json.loads(row["value"])

    def save_inventory(
        self,
        columns: list[str],
        rows: list[dict[str, str]],
        source_path: str,
        updated_at: str,
    ) -> None:
        self._set_json(
            "inventory",
            {
                "columns": columns,
                "rows": rows,
                "source_path": source_path,
                "updated_at": updated_at,
            },
        )

    def get_inventory(self) -> dict[str, object]:
        return self._get_json(
            "inventory",
            {"columns": [], "rows": [], "source_path": None, "updated_at": None},
        )

    def save_file_groups(self, groups: dict[str, list[dict[str, object]]]) -> None:
        self._set_json("file_groups", groups)

    def get_file_groups(self) -> dict[str, list[dict[str, object]]]:
        return self._get_json(
            "file_groups",
            {
                "docker": [],
                "scripts": [],
                "manuals": [],
                "apt_packages": [],
                "python_packages": [],
            },
        )

    def save_scan_status(self, status: str, error: str | None) -> None:
        self._set_json(
            "scan_status",
            {
                "status": status,
                "error": error,
                "checked_at": datetime.now(tz=UTC).isoformat(),
            },
        )

    def get_scan_status(self) -> dict[str, str | None]:
        return self._get_json(
            "scan_status",
            {"status": "never_run", "error": None, "checked_at": None},
        )
```

- [ ] **Step 4: Run metadata tests**

Run:

```bash
cd apps/api
pytest tests/test_metadata_store.py -v
```

Expected: PASS.

- [ ] **Step 5: Commit**

Run:

```bash
git add apps/api/app/database.py apps/api/app/services/metadata_store.py apps/api/tests/test_metadata_store.py
git commit -m "feat(api): persist generated metadata"
```

Expected: commit succeeds.

## Task 6: API Routers For Dashboard, Inventory, Repositories, And Scripts

**Files:**
- Modify: `apps/api/app/main.py`
- Create: `apps/api/app/routers/__init__.py`
- Create: `apps/api/app/routers/dashboard.py`
- Create: `apps/api/app/routers/inventory.py`
- Create: `apps/api/app/routers/repository.py`
- Create: `apps/api/app/routers/scripts.py`
- Create: `apps/api/app/services/repository_status.py`
- Modify: `apps/api/tests/test_dashboard_api.py`

- [ ] **Step 1: Extend API tests**

Replace `apps/api/tests/test_dashboard_api.py` with:

```python
from pathlib import Path

from fastapi.testclient import TestClient

from app.main import create_app
from app.services.metadata_store import MetadataStore


def test_health_endpoint_returns_ok():
    client = TestClient(create_app())

    response = client.get("/health")

    assert response.status_code == 200
    assert response.json() == {"status": "ok"}


def test_dashboard_endpoint_returns_counts(tmp_path):
    db_path = tmp_path / "app.db"
    store = MetadataStore(db_path)
    store.initialize()
    store.save_inventory(
        columns=["device_name"],
        rows=[{"device_name": "Edge Gateway A"}],
        source_path="inventory/latest.csv",
        updated_at="2026-06-02T00:00:00+00:00",
    )
    store.save_file_groups(
        {
            "docker": [{"name": "Dockerfile"}],
            "scripts": [{"name": "install-agent.sh"}],
            "manuals": [{"name": "initial-setup.md"}],
            "apt_packages": [],
            "python_packages": [],
        }
    )
    store.save_scan_status("success", None)

    client = TestClient(create_app(metadata_db=db_path))

    response = client.get("/dashboard")

    assert response.status_code == 200
    body = response.json()
    assert body["inventory_count"] == 1
    assert body["docker_artifact_count"] == 1
    assert body["script_count"] == 1
    assert body["last_scan_status"] == "success"


def test_inventory_endpoint_returns_rows(tmp_path):
    db_path = tmp_path / "app.db"
    store = MetadataStore(db_path)
    store.initialize()
    store.save_inventory(
        columns=["device_name"],
        rows=[{"device_name": "Edge Gateway A"}],
        source_path="inventory/latest.csv",
        updated_at="2026-06-02T00:00:00+00:00",
    )

    client = TestClient(create_app(metadata_db=db_path))

    response = client.get("/inventory")

    assert response.status_code == 200
    assert response.json()["rows"][0]["device_name"] == "Edge Gateway A"
```

- [ ] **Step 2: Run tests to verify failure**

Run:

```bash
cd apps/api
pytest tests/test_dashboard_api.py -v
```

Expected: FAIL because `create_app` does not accept `metadata_db` and routers do not exist.

- [ ] **Step 3: Implement router modules**

Create `apps/api/app/routers/__init__.py`:

```python
"""API router package."""
```

Create `apps/api/app/routers/dashboard.py`:

```python
from pathlib import Path

from fastapi import APIRouter

from app.models import DashboardStatus
from app.services.metadata_store import MetadataStore


def router(metadata_db: Path) -> APIRouter:
    api = APIRouter()

    @api.get("/dashboard", response_model=DashboardStatus)
    def dashboard() -> DashboardStatus:
        store = MetadataStore(metadata_db)
        store.initialize()
        inventory = store.get_inventory()
        groups = store.get_file_groups()
        scan = store.get_scan_status()
        return DashboardStatus(
            inventory_updated_at=inventory.get("updated_at"),
            inventory_count=len(inventory.get("rows", [])),
            docker_artifact_count=len(groups.get("docker", [])),
            script_count=len(groups.get("scripts", [])),
            manual_count=len(groups.get("manuals", [])),
            apt_package_count=len(groups.get("apt_packages", [])),
            python_package_count=len(groups.get("python_packages", [])),
            last_scan_status=str(scan.get("status")),
            last_scan_error=scan.get("error"),
        )

    return api
```

Create `apps/api/app/routers/inventory.py`:

```python
from pathlib import Path

from fastapi import APIRouter

from app.services.metadata_store import MetadataStore


def router(metadata_db: Path) -> APIRouter:
    api = APIRouter()

    @api.get("/inventory")
    def inventory() -> dict[str, object]:
        store = MetadataStore(metadata_db)
        store.initialize()
        return store.get_inventory()

    return api
```

Create `apps/api/app/routers/repository.py`:

```python
from pathlib import Path

from fastapi import APIRouter

from app.services.metadata_store import MetadataStore
from app.services.repository_status import repository_commands


def router(metadata_db: Path) -> APIRouter:
    api = APIRouter()

    @api.get("/repositories")
    def repositories() -> dict[str, object]:
        store = MetadataStore(metadata_db)
        store.initialize()
        groups = store.get_file_groups()
        return {
            "docker": groups.get("docker", []),
            "apt_packages": groups.get("apt_packages", []),
            "python_packages": groups.get("python_packages", []),
            "manuals": groups.get("manuals", []),
            "commands": repository_commands(),
        }

    return api
```

Create `apps/api/app/routers/scripts.py`:

```python
from pathlib import Path

from fastapi import APIRouter

from app.services.metadata_store import MetadataStore


def router(metadata_db: Path) -> APIRouter:
    api = APIRouter()

    @api.get("/scripts")
    def scripts() -> dict[str, object]:
        store = MetadataStore(metadata_db)
        store.initialize()
        groups = store.get_file_groups()
        return {"scripts": groups.get("scripts", [])}

    return api
```

Create `apps/api/app/services/repository_status.py`:

```python
def repository_commands() -> dict[str, str]:
    return {
        "docker_login_note": "Internal network registry does not require login in MVP.",
        "docker_pull_example": "docker pull repo.local:5000/edge-agent:1.0.0",
        "apt_source_example": "echo 'deb [trusted=yes] http://repo.local/apt ./' | sudo tee /etc/apt/sources.list.d/company.list",
        "pip_index_example": "pip install --index-url http://repo.local/pypi/simple/ package-name",
        "script_example": "curl -fsSL http://repo.local/files/scripts/install-agent.sh | bash",
    }
```

- [ ] **Step 4: Update app factory**

Replace `apps/api/app/main.py` with:

```python
from pathlib import Path

from fastapi import FastAPI

from app.config import get_settings
from app.routers import dashboard, inventory, repository, scripts


def create_app(metadata_db: Path | None = None) -> FastAPI:
    settings = get_settings()
    db_path = metadata_db or settings.metadata_db
    app = FastAPI(title="Linux Repo Web API")

    @app.get("/health")
    def health() -> dict[str, str]:
        return {"status": "ok"}

    app.include_router(dashboard.router(db_path))
    app.include_router(inventory.router(db_path))
    app.include_router(repository.router(db_path))
    app.include_router(scripts.router(db_path))
    return app


app = create_app()
```

- [ ] **Step 5: Run API tests**

Run:

```bash
cd apps/api
pytest -v
```

Expected: PASS.

- [ ] **Step 6: Commit**

Run:

```bash
git add apps/api
git commit -m "feat(api): expose read-only metadata endpoints"
```

Expected: commit succeeds.

## Task 7: Scanner Command For Metadata Refresh

**Files:**
- Create: `apps/api/app/services/refresh.py`
- Modify: `apps/api/app/main.py`
- Create: `apps/api/tests/test_refresh.py`

- [ ] **Step 1: Write refresh test**

Create `apps/api/tests/test_refresh.py`:

```python
from app.services.metadata_store import MetadataStore
from app.services.refresh import refresh_metadata


def test_refresh_metadata_reads_inventory_and_files(tmp_path):
    inbox = tmp_path / "inbox"
    metadata_db = tmp_path / "metadata" / "app.db"
    (inbox / "inventory").mkdir(parents=True)
    (inbox / "scripts").mkdir()
    (inbox / "inventory" / "latest.csv").write_text(
        "device_name,ip_address\nEdge Gateway A,192.168.10.21\n",
        encoding="utf-8",
    )
    (inbox / "scripts" / "install-agent.sh").write_text("#!/usr/bin/env sh\n", encoding="utf-8")

    refresh_metadata(inbox, metadata_db)

    store = MetadataStore(metadata_db)
    inventory = store.get_inventory()
    groups = store.get_file_groups()
    status = store.get_scan_status()
    assert inventory["rows"][0]["device_name"] == "Edge Gateway A"
    assert groups["scripts"][0]["name"] == "install-agent.sh"
    assert status["status"] == "success"
```

- [ ] **Step 2: Run test to verify failure**

Run:

```bash
cd apps/api
pytest tests/test_refresh.py -v
```

Expected: FAIL because `app.services.refresh` does not exist.

- [ ] **Step 3: Implement refresh service**

Create `apps/api/app/services/refresh.py`:

```python
from pathlib import Path

from app.services.file_scanner import scan_inbox_files
from app.services.inventory_csv import parse_inventory_csv
from app.services.metadata_store import MetadataStore


def refresh_metadata(inbox_dir: Path, metadata_db: Path) -> None:
    store = MetadataStore(metadata_db)
    store.initialize()

    try:
        inventory_path = inbox_dir / "inventory" / "latest.csv"
        if inventory_path.exists():
            inventory = parse_inventory_csv(inventory_path)
            store.save_inventory(
                columns=inventory.columns,
                rows=inventory.rows,
                source_path=inventory.source_path,
                updated_at=inventory.updated_at,
            )

        groups = scan_inbox_files(inbox_dir)
        store.save_file_groups(
            {
                key: [item.model_dump() for item in value]
                for key, value in groups.items()
            }
        )
        store.save_scan_status("success", None)
    except Exception as exc:
        store.save_scan_status("failed", str(exc))
        raise
```

- [ ] **Step 4: Add manual refresh endpoint**

Modify `apps/api/app/main.py` by adding this import:

```python
from app.services.refresh import refresh_metadata
```

Inside `create_app`, after the health endpoint, add:

```python
    @app.post("/refresh")
    def refresh() -> dict[str, str]:
        refresh_metadata(settings.inbox_dir, db_path)
        return {"status": "refreshed"}
```

The complete `apps/api/app/main.py` should be:

```python
from pathlib import Path

from fastapi import FastAPI

from app.config import get_settings
from app.routers import dashboard, inventory, repository, scripts
from app.services.refresh import refresh_metadata


def create_app(metadata_db: Path | None = None) -> FastAPI:
    settings = get_settings()
    db_path = metadata_db or settings.metadata_db
    app = FastAPI(title="Linux Repo Web API")

    @app.get("/health")
    def health() -> dict[str, str]:
        return {"status": "ok"}

    @app.post("/refresh")
    def refresh() -> dict[str, str]:
        refresh_metadata(settings.inbox_dir, db_path)
        return {"status": "refreshed"}

    app.include_router(dashboard.router(db_path))
    app.include_router(inventory.router(db_path))
    app.include_router(repository.router(db_path))
    app.include_router(scripts.router(db_path))
    return app


app = create_app()
```

- [ ] **Step 5: Run API tests**

Run:

```bash
cd apps/api
pytest -v
```

Expected: PASS.

- [ ] **Step 6: Commit**

Run:

```bash
git add apps/api
git commit -m "feat(api): refresh metadata from inbox"
```

Expected: commit succeeds.

## Task 8: React/Vite Web Shell And Dashboard

**Files:**
- Create: `apps/web/package.json`
- Create: `apps/web/index.html`
- Create: `apps/web/tsconfig.json`
- Create: `apps/web/vite.config.ts`
- Create: `apps/web/src/main.tsx`
- Create: `apps/web/src/App.tsx`
- Create: `apps/web/src/styles.css`
- Create: `apps/web/src/api/client.ts`
- Create: `apps/web/src/components/Layout.tsx`
- Create: `apps/web/src/components/StatusCard.tsx`
- Create: `apps/web/src/pages/DashboardPage.tsx`
- Create: `apps/web/src/__tests__/App.test.tsx`

- [ ] **Step 1: Create failing web test**

Create `apps/web/src/__tests__/App.test.tsx`:

```tsx
import { render, screen } from "@testing-library/react";
import { describe, expect, it } from "vitest";
import App from "../App";

describe("App", () => {
  it("renders the dashboard tab", () => {
    render(<App />);
    expect(screen.getByRole("button", { name: "Dashboard" })).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Create web package files**

Create `apps/web/package.json`:

```json
{
  "name": "linux-repo-web",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite --host 0.0.0.0",
    "build": "tsc && vite build",
    "test": "vitest run"
  },
  "dependencies": {
    "@vitejs/plugin-react": "latest",
    "vite": "latest",
    "typescript": "latest",
    "react": "latest",
    "react-dom": "latest",
    "lucide-react": "latest"
  },
  "devDependencies": {
    "@testing-library/jest-dom": "latest",
    "@testing-library/react": "latest",
    "@testing-library/user-event": "latest",
    "@types/react": "latest",
    "@types/react-dom": "latest",
    "jsdom": "latest",
    "vitest": "latest"
  }
}
```

Create `apps/web/index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Linux Repo Web</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

Create `apps/web/tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["DOM", "DOM.Iterable", "ES2020"],
    "allowJs": false,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"],
  "references": []
}
```

Create `apps/web/vite.config.ts`:

```ts
import react from "@vitejs/plugin-react";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["@testing-library/jest-dom/vitest"]
  },
  server: {
    proxy: {
      "/api": {
        target: "http://localhost:8000",
        rewrite: (path) => path.replace(/^\/api/, "")
      }
    }
  }
});
```

- [ ] **Step 3: Run web test to verify failure**

Run:

```bash
cd apps/web
npm install
npm test -- --runInBand
```

Expected: FAIL because `src/App.tsx` does not exist.

- [ ] **Step 4: Implement API client, layout, dashboard, and styles**

Create `apps/web/src/api/client.ts`:

```ts
export type DashboardStatus = {
  inventory_updated_at: string | null;
  inventory_count: number;
  docker_artifact_count: number;
  script_count: number;
  manual_count: number;
  apt_package_count: number;
  python_package_count: number;
  last_scan_status: string;
  last_scan_error: string | null;
};

export async function getDashboard(): Promise<DashboardStatus> {
  const response = await fetch("/api/dashboard");
  if (!response.ok) {
    throw new Error("Failed to load dashboard");
  }
  return response.json();
}
```

Create `apps/web/src/components/StatusCard.tsx`:

```tsx
type StatusCardProps = {
  label: string;
  value: string | number;
  detail?: string;
};

export function StatusCard({ label, value, detail }: StatusCardProps) {
  return (
    <section className="status-card">
      <div className="status-label">{label}</div>
      <div className="status-value">{value}</div>
      {detail ? <div className="status-detail">{detail}</div> : null}
    </section>
  );
}
```

Create `apps/web/src/components/Layout.tsx`:

```tsx
import type { ReactNode } from "react";

export type TabKey = "dashboard" | "inventory" | "docker" | "apt-python" | "scripts" | "setup";

const tabs: Array<{ key: TabKey; label: string }> = [
  { key: "dashboard", label: "Dashboard" },
  { key: "inventory", label: "Inventory" },
  { key: "docker", label: "Docker" },
  { key: "apt-python", label: "APT / Python" },
  { key: "scripts", label: "Shell Script" },
  { key: "setup", label: "Initial Setup" }
];

type LayoutProps = {
  activeTab: TabKey;
  onTabChange: (tab: TabKey) => void;
  children: ReactNode;
};

export function Layout({ activeTab, onTabChange, children }: LayoutProps) {
  return (
    <div className="app-shell">
      <header className="topbar">
        <div>
          <h1>Linux Repo Web</h1>
          <p>Internal read-only deployment portal</p>
        </div>
      </header>
      <nav className="tabs" aria-label="Main tabs">
        {tabs.map((tab) => (
          <button
            key={tab.key}
            className={activeTab === tab.key ? "tab active" : "tab"}
            type="button"
            onClick={() => onTabChange(tab.key)}
          >
            {tab.label}
          </button>
        ))}
      </nav>
      <main className="page">{children}</main>
    </div>
  );
}
```

Create `apps/web/src/pages/DashboardPage.tsx`:

```tsx
import { useEffect, useState } from "react";
import { getDashboard, type DashboardStatus } from "../api/client";
import { StatusCard } from "../components/StatusCard";

const fallback: DashboardStatus = {
  inventory_updated_at: null,
  inventory_count: 0,
  docker_artifact_count: 0,
  script_count: 0,
  manual_count: 0,
  apt_package_count: 0,
  python_package_count: 0,
  last_scan_status: "not_loaded",
  last_scan_error: null
};

export function DashboardPage() {
  const [status, setStatus] = useState<DashboardStatus>(fallback);

  useEffect(() => {
    getDashboard().then(setStatus).catch(() => setStatus(fallback));
  }, []);

  return (
    <div className="stack">
      <section>
        <h2>Dashboard</h2>
        <p className="muted">Current sync and repository summary.</p>
      </section>
      <div className="status-grid">
        <StatusCard label="Inventory rows" value={status.inventory_count} detail={status.inventory_updated_at ?? "No CSV loaded"} />
        <StatusCard label="Docker artifacts" value={status.docker_artifact_count} />
        <StatusCard label="Scripts" value={status.script_count} />
        <StatusCard label="APT packages" value={status.apt_package_count} />
        <StatusCard label="Python packages" value={status.python_package_count} />
        <StatusCard label="Last scan" value={status.last_scan_status} detail={status.last_scan_error ?? "No error"} />
      </div>
    </div>
  );
}
```

Create `apps/web/src/App.tsx`:

```tsx
import { useState } from "react";
import { Layout, type TabKey } from "./components/Layout";
import { DashboardPage } from "./pages/DashboardPage";
import "./styles.css";

export default function App() {
  const [activeTab, setActiveTab] = useState<TabKey>("dashboard");

  return (
    <Layout activeTab={activeTab} onTabChange={setActiveTab}>
      <DashboardPage />
    </Layout>
  );
}
```

Create `apps/web/src/main.tsx`:

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root") as HTMLElement).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

Create `apps/web/src/styles.css`:

```css
:root {
  color: #172026;
  background: #f4f6f8;
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

button {
  font: inherit;
}

.app-shell {
  min-height: 100vh;
}

.topbar {
  background: #24313a;
  color: #ffffff;
  padding: 20px 28px;
}

.topbar h1 {
  margin: 0;
  font-size: 24px;
}

.topbar p {
  margin: 4px 0 0;
  color: #d7dee4;
}

.tabs {
  display: flex;
  gap: 4px;
  overflow-x: auto;
  padding: 10px 20px;
  background: #ffffff;
  border-bottom: 1px solid #d7dee4;
}

.tab {
  border: 0;
  background: transparent;
  padding: 10px 12px;
  cursor: pointer;
  color: #3c4a53;
}

.tab.active {
  color: #ffffff;
  background: #176b87;
  border-radius: 6px;
}

.page {
  padding: 24px;
}

.stack {
  display: grid;
  gap: 18px;
}

.muted {
  color: #60717d;
}

.status-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 12px;
}

.status-card {
  background: #ffffff;
  border: 1px solid #d7dee4;
  border-radius: 8px;
  padding: 16px;
}

.status-label {
  color: #60717d;
  font-size: 13px;
}

.status-value {
  margin-top: 8px;
  font-size: 26px;
  font-weight: 700;
}

.status-detail {
  margin-top: 8px;
  color: #60717d;
  font-size: 13px;
}
```

- [ ] **Step 5: Run web tests**

Run:

```bash
cd apps/web
npm test
```

Expected: PASS.

- [ ] **Step 6: Commit**

Run:

```bash
git add apps/web
git commit -m "feat(web): add dashboard shell"
```

Expected: commit succeeds.

## Task 9: Web Feature Tabs

**Files:**
- Create: `apps/web/src/components/CommandBox.tsx`
- Create: `apps/web/src/pages/InventoryPage.tsx`
- Create: `apps/web/src/pages/DockerPage.tsx`
- Create: `apps/web/src/pages/AptPythonPage.tsx`
- Create: `apps/web/src/pages/ScriptsPage.tsx`
- Create: `apps/web/src/pages/InitialSetupPage.tsx`
- Modify: `apps/web/src/api/client.ts`
- Modify: `apps/web/src/App.tsx`
- Modify: `apps/web/src/styles.css`
- Modify: `apps/web/src/__tests__/App.test.tsx`

- [ ] **Step 1: Extend web tests for tabs**

Replace `apps/web/src/__tests__/App.test.tsx` with:

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, expect, it } from "vitest";
import App from "../App";

describe("App", () => {
  it("renders all top-level tabs", () => {
    render(<App />);
    expect(screen.getByRole("button", { name: "Dashboard" })).toBeInTheDocument();
    expect(screen.getByRole("button", { name: "Inventory" })).toBeInTheDocument();
    expect(screen.getByRole("button", { name: "Docker" })).toBeInTheDocument();
    expect(screen.getByRole("button", { name: "APT / Python" })).toBeInTheDocument();
    expect(screen.getByRole("button", { name: "Shell Script" })).toBeInTheDocument();
    expect(screen.getByRole("button", { name: "Initial Setup" })).toBeInTheDocument();
  });

  it("switches to inventory page", async () => {
    render(<App />);
    await userEvent.click(screen.getByRole("button", { name: "Inventory" }));
    expect(screen.getByRole("heading", { name: "Inventory" })).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Run tests to verify incomplete behavior**

Run:

```bash
cd apps/web
npm test
```

Expected: second test fails because `App` always renders `DashboardPage`.

- [ ] **Step 3: Extend API client**

Append to `apps/web/src/api/client.ts`:

```ts
export type InventoryResponse = {
  columns: string[];
  rows: Array<Record<string, string>>;
  source_path: string | null;
  updated_at: string | null;
};

export type RepositoryResponse = {
  docker: Array<Record<string, string | number>>;
  apt_packages: Array<Record<string, string | number>>;
  python_packages: Array<Record<string, string | number>>;
  manuals: Array<Record<string, string | number>>;
  commands: Record<string, string>;
};

export type ScriptsResponse = {
  scripts: Array<Record<string, string | number>>;
};

export async function getInventory(): Promise<InventoryResponse> {
  const response = await fetch("/api/inventory");
  if (!response.ok) {
    throw new Error("Failed to load inventory");
  }
  return response.json();
}

export async function getRepositories(): Promise<RepositoryResponse> {
  const response = await fetch("/api/repositories");
  if (!response.ok) {
    throw new Error("Failed to load repositories");
  }
  return response.json();
}

export async function getScripts(): Promise<ScriptsResponse> {
  const response = await fetch("/api/scripts");
  if (!response.ok) {
    throw new Error("Failed to load scripts");
  }
  return response.json();
}
```

- [ ] **Step 4: Add CommandBox component**

Create `apps/web/src/components/CommandBox.tsx`:

```tsx
type CommandBoxProps = {
  label: string;
  command: string;
};

export function CommandBox({ label, command }: CommandBoxProps) {
  return (
    <div className="command-box">
      <div className="command-label">{label}</div>
      <code>{command}</code>
      <button type="button" onClick={() => navigator.clipboard?.writeText(command)}>
        Copy
      </button>
    </div>
  );
}
```

- [ ] **Step 5: Add page components**

Create `apps/web/src/pages/InventoryPage.tsx`:

```tsx
import { useEffect, useMemo, useState } from "react";
import { getInventory, type InventoryResponse } from "../api/client";

const emptyInventory: InventoryResponse = {
  columns: [],
  rows: [],
  source_path: null,
  updated_at: null
};

export function InventoryPage() {
  const [inventory, setInventory] = useState(emptyInventory);
  const [query, setQuery] = useState("");

  useEffect(() => {
    getInventory().then(setInventory).catch(() => setInventory(emptyInventory));
  }, []);

  const filteredRows = useMemo(() => {
    const needle = query.trim().toLowerCase();
    if (!needle) return inventory.rows;
    return inventory.rows.filter((row) =>
      Object.values(row).some((value) => String(value).toLowerCase().includes(needle))
    );
  }, [inventory.rows, query]);

  return (
    <div className="stack">
      <section>
        <h2>Inventory</h2>
        <p className="muted">Read-only device inventory from latest.csv.</p>
      </section>
      <input className="search" value={query} onChange={(event) => setQuery(event.target.value)} placeholder="Search devices, IPs, IDs, sites" />
      <div className="table-wrap">
        <table>
          <thead>
            <tr>{inventory.columns.map((column) => <th key={column}>{column}</th>)}</tr>
          </thead>
          <tbody>
            {filteredRows.map((row, index) => (
              <tr key={index}>
                {inventory.columns.map((column) => <td key={column}>{row[column]}</td>)}
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

Create `apps/web/src/pages/DockerPage.tsx`:

```tsx
import { useEffect, useState } from "react";
import { getRepositories, type RepositoryResponse } from "../api/client";
import { CommandBox } from "../components/CommandBox";

const emptyRepos: RepositoryResponse = {
  docker: [],
  apt_packages: [],
  python_packages: [],
  manuals: [],
  commands: {}
};

export function DockerPage() {
  const [repos, setRepos] = useState(emptyRepos);

  useEffect(() => {
    getRepositories().then(setRepos).catch(() => setRepos(emptyRepos));
  }, []);

  return (
    <div className="stack">
      <section>
        <h2>Docker</h2>
        <p className="muted">Dockerfiles, compose files, and registry usage.</p>
      </section>
      {repos.commands.docker_pull_example ? <CommandBox label="Pull example" command={repos.commands.docker_pull_example} /> : null}
      <div className="list">
        {repos.docker.map((item) => <div className="list-row" key={String(item.relative_path)}>{String(item.relative_path)}</div>)}
      </div>
    </div>
  );
}
```

Create `apps/web/src/pages/AptPythonPage.tsx`:

```tsx
import { useEffect, useState } from "react";
import { getRepositories, type RepositoryResponse } from "../api/client";
import { CommandBox } from "../components/CommandBox";

const emptyRepos: RepositoryResponse = {
  docker: [],
  apt_packages: [],
  python_packages: [],
  manuals: [],
  commands: {}
};

export function AptPythonPage() {
  const [repos, setRepos] = useState(emptyRepos);

  useEffect(() => {
    getRepositories().then(setRepos).catch(() => setRepos(emptyRepos));
  }, []);

  return (
    <div className="stack">
      <section>
        <h2>APT / Python</h2>
        <p className="muted">Internal package repository setup commands and files.</p>
      </section>
      {repos.commands.apt_source_example ? <CommandBox label="APT source" command={repos.commands.apt_source_example} /> : null}
      {repos.commands.pip_index_example ? <CommandBox label="pip index" command={repos.commands.pip_index_example} /> : null}
      <h3>APT packages</h3>
      <div className="list">{repos.apt_packages.map((item) => <div className="list-row" key={String(item.relative_path)}>{String(item.name)}</div>)}</div>
      <h3>Python packages</h3>
      <div className="list">{repos.python_packages.map((item) => <div className="list-row" key={String(item.relative_path)}>{String(item.name)}</div>)}</div>
    </div>
  );
}
```

Create `apps/web/src/pages/ScriptsPage.tsx`:

```tsx
import { useEffect, useState } from "react";
import { getScripts, type ScriptsResponse } from "../api/client";
import { CommandBox } from "../components/CommandBox";

const emptyScripts: ScriptsResponse = { scripts: [] };

export function ScriptsPage() {
  const [response, setResponse] = useState(emptyScripts);

  useEffect(() => {
    getScripts().then(setResponse).catch(() => setResponse(emptyScripts));
  }, []);

  return (
    <div className="stack">
      <section>
        <h2>Shell Script</h2>
        <p className="muted">Read-only script list and execution commands.</p>
      </section>
      {response.scripts.map((script) => {
        const path = String(script.relative_path);
        return (
          <div className="list-row" key={path}>
            <strong>{String(script.name)}</strong>
            <CommandBox label="Run" command={`curl -fsSL http://repo.local/files/${path} | bash`} />
          </div>
        );
      })}
    </div>
  );
}
```

Create `apps/web/src/pages/InitialSetupPage.tsx`:

```tsx
import { CommandBox } from "../components/CommandBox";

export function InitialSetupPage() {
  return (
    <div className="stack">
      <section>
        <h2>Initial Setup</h2>
        <p className="muted">Base commands for new Linux devices.</p>
      </section>
      <CommandBox label="Docker registry check" command="curl -fsSL http://repo.local:5000/v2/" />
      <CommandBox label="APT source" command="echo 'deb [trusted=yes] http://repo.local/apt ./' | sudo tee /etc/apt/sources.list.d/company.list" />
      <CommandBox label="pip index" command="pip config set global.index-url http://repo.local/pypi/simple/" />
    </div>
  );
}
```

- [ ] **Step 6: Route pages in `App.tsx`**

Replace `apps/web/src/App.tsx` with:

```tsx
import { useState } from "react";
import { Layout, type TabKey } from "./components/Layout";
import { AptPythonPage } from "./pages/AptPythonPage";
import { DashboardPage } from "./pages/DashboardPage";
import { DockerPage } from "./pages/DockerPage";
import { InitialSetupPage } from "./pages/InitialSetupPage";
import { InventoryPage } from "./pages/InventoryPage";
import { ScriptsPage } from "./pages/ScriptsPage";
import "./styles.css";

function activePage(activeTab: TabKey) {
  if (activeTab === "inventory") return <InventoryPage />;
  if (activeTab === "docker") return <DockerPage />;
  if (activeTab === "apt-python") return <AptPythonPage />;
  if (activeTab === "scripts") return <ScriptsPage />;
  if (activeTab === "setup") return <InitialSetupPage />;
  return <DashboardPage />;
}

export default function App() {
  const [activeTab, setActiveTab] = useState<TabKey>("dashboard");

  return (
    <Layout activeTab={activeTab} onTabChange={setActiveTab}>
      {activePage(activeTab)}
    </Layout>
  );
}
```

- [ ] **Step 7: Extend CSS**

Append to `apps/web/src/styles.css`:

```css
.search {
  width: min(520px, 100%);
  padding: 10px 12px;
  border: 1px solid #b8c5ce;
  border-radius: 6px;
}

.table-wrap {
  overflow-x: auto;
  background: #ffffff;
  border: 1px solid #d7dee4;
  border-radius: 8px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 10px 12px;
  text-align: left;
  border-bottom: 1px solid #e5eaee;
  white-space: nowrap;
}

.list {
  display: grid;
  gap: 8px;
}

.list-row {
  background: #ffffff;
  border: 1px solid #d7dee4;
  border-radius: 8px;
  padding: 12px;
}

.command-box {
  display: grid;
  grid-template-columns: minmax(120px, 180px) 1fr auto;
  gap: 8px;
  align-items: center;
  background: #ffffff;
  border: 1px solid #d7dee4;
  border-radius: 8px;
  padding: 10px;
}

.command-box code {
  overflow-x: auto;
  white-space: nowrap;
}

.command-box button {
  border: 0;
  border-radius: 6px;
  background: #176b87;
  color: #ffffff;
  padding: 8px 10px;
  cursor: pointer;
}

.command-label {
  color: #60717d;
}

@media (max-width: 720px) {
  .command-box {
    grid-template-columns: 1fr;
  }
}
```

- [ ] **Step 8: Run web tests and build**

Run:

```bash
cd apps/web
npm test
npm run build
```

Expected: both commands PASS.

- [ ] **Step 9: Commit**

Run:

```bash
git add apps/web
git commit -m "feat(web): add read-only portal tabs"
```

Expected: commit succeeds.

## Task 10: Docker Compose And Infrastructure

**Files:**
- Create: `docker-compose.yml`
- Create: `apps/web/Dockerfile`
- Create: `infra/nginx/nginx.conf`
- Create: `infra/registry/config.yml`
- Create: `infra/apt/rebuild-index.sh`
- Create: `infra/pypi/README.md`

- [ ] **Step 1: Add failing compose config check**

Run:

```bash
docker compose config
```

Expected: FAIL because `docker-compose.yml` does not exist.

- [ ] **Step 2: Create web Dockerfile**

Create `apps/web/Dockerfile`:

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:1.27-alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

- [ ] **Step 3: Create Nginx config**

Create `infra/nginx/nginx.conf`:

```nginx
events {}

http {
  server {
    listen 80;

    location /api/ {
      proxy_pass http://api:8000/;
    }

    location /files/ {
      alias /srv/files/;
      autoindex on;
    }

    location /apt/ {
      alias /srv/apt/;
      autoindex on;
    }

    location /pypi/ {
      proxy_pass http://pypi:8080/;
    }

    location / {
      proxy_pass http://web:80/;
    }
  }
}
```

- [ ] **Step 4: Create registry config**

Create `infra/registry/config.yml`:

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
```

- [ ] **Step 5: Create APT rebuild script**

Create `infra/apt/rebuild-index.sh`:

```sh
#!/usr/bin/env sh
set -eu

REPO_DIR="${1:-/repo}"
cd "$REPO_DIR"

if command -v dpkg-scanpackages >/dev/null 2>&1; then
  dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
else
  echo "dpkg-scanpackages is not installed" >&2
  exit 1
fi
```

- [ ] **Step 6: Create PyPI note**

Create `infra/pypi/README.md`:

```markdown
# Internal PyPI

The MVP uses `pypiserver` with no authentication on the internal network.

Packages are mounted from `data/pypi`.
```

- [ ] **Step 7: Create Compose file**

Create `docker-compose.yml`:

```yaml
services:
  api:
    build:
      context: ./apps/api
    environment:
      LINUX_REPO_WEB_INBOX_DIR: /data/inbox
      LINUX_REPO_WEB_METADATA_DB: /data/metadata/app.db
      LINUX_REPO_WEB_REGISTRY_URL: http://registry:5000
    volumes:
      - ./data:/data
      - ./samples/inbox:/data/inbox:ro
    ports:
      - "8000:8000"

  web:
    build:
      context: ./apps/web

  nginx:
    image: nginx:1.27-alpine
    depends_on:
      - api
      - web
      - pypi
    ports:
      - "80:80"
    volumes:
      - ./infra/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./samples/inbox/scripts:/srv/files/scripts:ro
      - ./samples/inbox/manuals:/srv/files/manuals:ro
      - ./data/apt-repo:/srv/apt:ro

  registry:
    image: registry:2
    ports:
      - "5000:5000"
    volumes:
      - ./infra/registry/config.yml:/etc/docker/registry/config.yml:ro
      - ./data/registry:/var/lib/registry

  pypi:
    image: pypiserver/pypiserver:latest
    command: ["run", "-P", ".", "-a", ".", "/data/packages"]
    volumes:
      - ./data/pypi:/data/packages
```

- [ ] **Step 8: Validate compose config**

Run:

```bash
docker compose config
```

Expected: PASS and prints normalized Compose configuration.

- [ ] **Step 9: Commit**

Run:

```bash
git add docker-compose.yml apps/web/Dockerfile infra
git commit -m "feat(infra): add compose repository stack"
```

Expected: commit succeeds.

## Task 11: Operations Documentation

**Files:**
- Create: `docs/operations/sync-flow.md`
- Modify: `README.md`

- [ ] **Step 1: Create sync flow documentation**

Create `docs/operations/sync-flow.md`:

```markdown
# Sync Flow

The Linux Repo Web server does not access the team cloud document location directly.

An external PC or automation job prepares files and pushes them into the server's `data/inbox` directory.

## Required inventory output

The API reads:

```text
data/inbox/inventory/latest.csv
```

The CSV must be UTF-8 or UTF-8 with BOM. The first row must contain headers.

## Suggested external PC steps

1. Export the cloud Excel inventory to CSV.
2. Copy the CSV to a staging directory as `latest.csv`.
3. Copy Dockerfiles, compose files, packages, scripts, and manuals into the expected folder layout.
4. Transfer the staging directory to the Linux server with `scp` or `rsync`.
5. Call `POST http://repo.local/api/refresh` or wait for the scheduled scanner once that scheduler is enabled.

## Atomic copy recommendation

Upload new files into a temporary directory and move them into place after transfer completes.

Example:

```bash
rsync -av ./staging/ repo.local:/opt/linux_repo_web/data/inbox.next/
ssh repo.local 'rm -rf /opt/linux_repo_web/data/inbox.prev && mv /opt/linux_repo_web/data/inbox /opt/linux_repo_web/data/inbox.prev && mv /opt/linux_repo_web/data/inbox.next /opt/linux_repo_web/data/inbox'
```
```

- [ ] **Step 2: Update README operations section**

Append to `README.md`:

```markdown

## Refresh metadata

After files are synchronized into `data/inbox`, refresh metadata:

```bash
curl -X POST http://localhost/api/refresh
```

## Operations docs

- [Sync Flow](docs/operations/sync-flow.md)
- [Design Spec](docs/superpowers/specs/2026-06-01-linux-repo-web-design.md)
```

- [ ] **Step 3: Verify docs links exist**

Run:

```bash
test -f docs/operations/sync-flow.md
test -f docs/superpowers/specs/2026-06-01-linux-repo-web-design.md
```

Expected: both commands pass.

- [ ] **Step 4: Commit**

Run:

```bash
git add README.md docs/operations/sync-flow.md
git commit -m "docs: document sync operations"
```

Expected: commit succeeds.

## Task 12: MVP Verification And Push

**Files:**
- No new source files.
- Verify all files from prior tasks.

- [ ] **Step 1: Run API tests**

Run:

```bash
cd apps/api
pytest -v
```

Expected: PASS.

- [ ] **Step 2: Run web tests and build**

Run:

```bash
cd apps/web
npm test
npm run build
```

Expected: PASS.

- [ ] **Step 3: Validate Compose**

Run:

```bash
docker compose config
```

Expected: PASS.

- [ ] **Step 4: Start stack**

Run:

```bash
docker compose up --build -d
```

Expected: services start successfully.

- [ ] **Step 5: Refresh metadata**

Run:

```bash
curl -X POST http://localhost/api/refresh
curl http://localhost/api/dashboard
```

Expected: dashboard JSON shows `inventory_count` greater than `0` and `last_scan_status` as `success`.

- [ ] **Step 6: Check service routes**

Run:

```bash
curl http://localhost/api/health
curl http://localhost:5000/v2/
curl http://localhost/
```

Expected:

```text
API health returns {"status":"ok"}.
Docker registry route returns an empty JSON object or a registry API response.
Web route returns HTML.
```

- [ ] **Step 7: Stop local stack**

Run:

```bash
docker compose down
```

Expected: all local containers stop cleanly.

- [ ] **Step 8: Push branch**

Run:

```bash
git status --short
git push
```

Expected: working tree is clean before push, and `main` pushes to `origin/main`.

## Self-Review

Spec coverage:

- Internal-only read-only portal: covered by Tasks 8, 9, 10, and 11.
- CSV canonical inventory source: covered by Tasks 1, 3, 6, 7, and 9.
- Docker registry service: covered by Task 10.
- APT repository route: covered by Tasks 10 and 11.
- Python package index: covered by Task 10.
- Shell script viewer and command copy: covered by Tasks 4, 6, and 9.
- Initial setup commands: covered by Task 9.
- Sync flow documentation: covered by Task 11.
- API, frontend, and integration verification: covered by Task 12.

Placeholder scan:

- This plan uses concrete implementation steps and exact file paths.
- Sample data and commands are concrete.

Type consistency:

- API metadata uses `InventorySnapshot`, `FileItem`, and `DashboardStatus`.
- Web client types match API endpoint response keys.
- File group keys are consistently `docker`, `scripts`, `manuals`, `apt_packages`, and `python_packages`.
