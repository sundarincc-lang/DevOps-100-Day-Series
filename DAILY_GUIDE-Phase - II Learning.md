# Phase 2 Daily Guide: Containers & Orchestration (Days 21–40)

A day-by-day, hands-on companion to the [Phase 2 README](./README.md). Every day has a goal, concepts, a lab you can run, a checkpoint, a **project outcome**, and references.

## How to use this guide

- Budget **1.5–2 hours per day**. Days marked *Part 1 / Part 2* split one topic across two days.
- All labs build **one capstone project**, *TaskBoard*, so by Day 40 you have a portfolio-ready repo.
- Commands assume Linux/macOS (or WSL2 on Windows). Replace `<placeholders>` with your own values.
- Commit to Git at the end of every day: `git add -A && git commit -m "day NN: <topic>"`.
- Version-sensitive tools (Kubernetes, Envoy Gateway, Istio, Knative) change quickly, so the guide links to each project's current install page instead of hard-coding versions.

## Capstone project: TaskBoard

A small task-tracking API backed by PostgreSQL and Redis, containerized on Days 26–29 and run on Kubernetes from Day 30 onward.

```mermaid
flowchart LR
    C[Client] --> GW[Gateway API<br/>Envoy Gateway]
    GW --> API[api Deployment<br/>Flask + Gunicorn<br/>2-6 replicas via HPA]
    API --> PG[(postgres StatefulSet<br/>PVC)]
    API --> R[(redis Deployment)]
    CFG[ConfigMap + Secret] -.-> API
    CFG -.-> PG
```

**Final repo layout**

```text
taskboard/
├── api/
│   ├── app.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .dockerignore
├── compose.yaml
├── .env.example
├── kind-config.yaml
├── k8s/
│   ├── base/            # namespace, redis, postgres, api, hpa, netpol, rbac, kustomization
│   ├── overlays/dev/
│   ├── overlays/prod/
│   └── gateway.yaml
├── charts/api/          # Helm chart (Day 38)
├── scripts/smoke-test.sh
├── notes/               # your daily notes
├── Makefile
└── README.md
```

## Roadmap at a glance

| Day | Topic | Project outcome |
|---|---|---|
| 21 | Container fundamentals (1) | Repo initialized; first containers run |
| 22 | Container fundamentals (2): namespaces, cgroups, layers | Notes proving isolation and limits |
| 23 | Docker CLI essentials | Base images pulled; lifecycle mastered |
| 24 | Docker volumes & bind mounts | Postgres data survives container deletion |
| 25 | Docker networking | Postgres + Redis on a private network |
| 26 | Dockerfile basics | TaskBoard API image v1 running |
| 27 | Multi-stage builds & optimization | Small, non-root `taskboard-api:1.0.0` |
| 28 | Docker Compose | Full stack with one command |
| 29 | Registries & tagging | Image pushed with immutable tags |
| 30 | Kubernetes architecture | Local 3-node cluster running |
| 31 | Pods | Pods, init/sidecar containers, debugging drills |
| 32 | Deployments & ReplicaSets | TaskBoard running on Kubernetes |
| 33 | Rolling updates & rollbacks | Zero-downtime release and rollback runbook |
| 34 | Services & Gateway API | App reachable through a Gateway |
| 35 | ConfigMaps, Secrets, persistent storage | Config externalized; data survives Pod deletion |
| 36 | Resources, probes, autoscaling | Self-healing, right-sized, autoscaled API |
| 37 | Container & cluster security | Scanned image, restricted Pods, RBAC, NetworkPolicies |
| 38 | Helm & Kustomize | Chart for the API; dev/prod overlays |
| 39 | Advanced: service mesh & serverless | Mesh and Knative labs completed |
| 40 | Capstone | Rebuild from zero with `make`, smoke tests pass |

---

# Day 21: Container Fundamentals (Part 1)

**Goal:** Explain what a container is, install Docker, and run your first containers.

### Concepts
- A container is an **isolated process** on a shared host kernel, started from an image. A VM runs a full guest OS on a hypervisor.
- **Image** = immutable, layered filesystem template. **Container** = running instance of an image.
- **OCI** (Open Container Initiative) standardizes image and runtime formats, so images work across Docker, containerd, Podman, and Kubernetes.
- Docker architecture: `docker` CLI → `dockerd` → `containerd` → `runc`.

### Hands-on
```bash
# 1. Install Docker Desktop (macOS/Windows) or Docker Engine (Linux), then verify
docker version
docker run --rm hello-world

# 2. Same kernel, different userland
uname -r
docker run --rm alpine uname -r      # Docker Desktop shows its Linux VM kernel

# 3. Process isolation: the container sees only its own processes
docker run --rm alpine ps

# 4. Containers start fast
time docker run --rm alpine echo "hello from a container"

# 5. Inspect what is on your machine
docker images
docker ps -a
docker system df
```

```bash
# 6. Start the project repo
mkdir -p taskboard/{api,notes,scripts,k8s/base} && cd taskboard
git init
echo "# TaskBoard" > README.md
```

### Checkpoint
1. Why does a container start faster than a VM?
2. What is the difference between an image and a container?
3. Which component actually creates the container process: `dockerd`, `containerd`, or `runc`?

### Project outcome
`taskboard/` Git repo initialized, Docker verified, and `notes/day21.md` with your answers.

### References
- [Docker overview](https://docs.docker.com/get-started/docker-overview/)
- [Install Docker](https://docs.docker.com/get-started/get-docker/)
- [Open Container Initiative overview](https://opencontainers.org/about/overview/)

---

# Day 22: Container Fundamentals (Part 2): Namespaces, cgroups, Layers

**Goal:** See *how* containers isolate and limit processes, and how image layers work.

### Concepts
- **Namespaces** isolate what a process can see (PID, network, mount, UTS, IPC, user).
- **cgroups** limit what a process can use (CPU, memory, I/O).
- **Union filesystems / layers**: each Dockerfile instruction adds a read-only layer; a container adds a thin writable layer on top.

### Hands-on
```bash
# 1. Namespaces without Docker (Linux or WSL2; needs sudo)
sudo unshare --fork --pid --mount-proc bash
ps aux            # you see only a few processes, and bash is PID 1
exit

# 2. cgroups through Docker: limit memory and CPU
docker run -d --name limited --memory 64m --cpus 0.5 nginx:alpine
docker stats --no-stream limited
docker inspect -f 'mem={{.HostConfig.Memory}} nanoCPUs={{.HostConfig.NanoCpus}}' limited
docker rm -f limited

# 3. Read the limit from inside the container (cgroup v2)
docker run --rm --memory 64m alpine cat /sys/fs/cgroup/memory.max     # 67108864

# 4. Layers
docker pull python:3.12-slim
docker history python:3.12-slim
docker image inspect python:3.12-slim --format '{{len .RootFS.Layers}} layers'
```

**Experiment:** run `docker run --rm --memory 20m python:3.12-slim python -c "x=bytearray(100*1024*1024)"`. The process is killed by the OOM killer (exit code 137). Confirm with `echo $?`.

### Checkpoint
1. Which kernel feature gives a container its own process tree? Which one caps its memory?
2. Why do two images based on the same base share disk space?
3. What does exit code 137 mean?

### Project outcome
`notes/day22.md` describing the OOM experiment and the layer list of `python:3.12-slim`.

### References
- [namespaces(7)](https://man7.org/linux/man-pages/man7/namespaces.7.html), [cgroups(7)](https://man7.org/linux/man-pages/man7/cgroups.7.html)
- [Docker resource constraints](https://docs.docker.com/engine/containers/resource_constraints/)
- [About storage drivers / layers](https://docs.docker.com/engine/storage/drivers/)

---

# Day 23: Docker CLI Essentials

**Goal:** Master the container lifecycle and day-to-day debugging commands.

### Concepts
Lifecycle: `create → start → (pause) → stop → rm`. Containers are **ephemeral**: changes in the writable layer vanish when the container is removed.

### Hands-on
```bash
# Run, publish a port, set an env var, name it
docker run -d --name web -p 8080:80 -e GREETING=hi nginx:alpine
curl -I http://localhost:8080

# Observe and debug
docker ps
docker logs -f web          # Ctrl+C to stop following
docker exec -it web sh      # explore, then exit
docker inspect web | head -40
docker stats --no-stream

# Prove ephemerality
docker exec web sh -c 'echo changed > /usr/share/nginx/html/index.html'
curl http://localhost:8080
docker rm -f web
docker run -d --name web -p 8080:80 nginx:alpine
curl http://localhost:8080      # back to the default page

# Copy files and clean up
docker cp web:/etc/nginx/nginx.conf ./nginx.conf
docker stop web && docker rm web
docker container prune -f
```

```bash
# Project prep: pull the images you will use for the rest of the phase
docker pull python:3.12-slim
docker pull postgres:16-alpine
docker pull redis:7-alpine
docker images --format 'table {{.Repository}}\t{{.Tag}}\t{{.Size}}'
```

### Checkpoint
1. What is the difference between `docker stop` and `docker kill`?
2. What does `-p 8080:80` mean (which side is the host)?
3. Where did your edit to `index.html` go?

### Project outcome
Three base images pulled and their sizes recorded in `notes/day23.md`.

### References
- [Docker CLI reference](https://docs.docker.com/reference/cli/docker/)
- [`docker run` reference](https://docs.docker.com/reference/cli/docker/container/run/)

---

# Day 24: Docker Volumes & Bind Mounts

**Goal:** Persist data outside the container lifecycle.

### Concepts
| Type | Managed by | Use for |
|---|---|---|
| **Named volume** | Docker | Databases, anything that must persist |
| **Bind mount** | You (host path) | Local development, config files |
| **tmpfs** | Memory | Secrets or scratch data that must not hit disk |

### Hands-on
```bash
# 1. Named volume: Postgres data outlives its container
docker volume create taskboard-pgdata
docker run -d --name pg \
  -e POSTGRES_PASSWORD=devpass -e POSTGRES_DB=taskboard \
  -v taskboard-pgdata:/var/lib/postgresql/data postgres:16-alpine

# wait a few seconds for startup, then write data
docker exec pg psql -U postgres -d taskboard -c "CREATE TABLE t(id int); INSERT INTO t VALUES (42);"

docker rm -f pg      # destroy the container

docker run -d --name pg \
  -e POSTGRES_PASSWORD=devpass -e POSTGRES_DB=taskboard \
  -v taskboard-pgdata:/var/lib/postgresql/data postgres:16-alpine
docker exec pg psql -U postgres -d taskboard -c "SELECT * FROM t;"   # still 42
docker rm -f pg

# 2. Bind mount: live-edit a site
mkdir -p site && echo '<h1>Bind mount works</h1>' > site/index.html
docker run -d --name web -p 8080:80 -v "$PWD/site:/usr/share/nginx/html:ro" nginx:alpine
curl http://localhost:8080
echo '<h1>Edited on the host</h1>' > site/index.html
curl http://localhost:8080
docker rm -f web

# 3. Inspect and back up a volume
docker volume ls
docker volume inspect taskboard-pgdata
docker run --rm -v taskboard-pgdata:/data -v "$PWD:/backup" alpine tar czf /backup/pgdata.tgz -C /data .
```

### Checkpoint
1. Why is a named volume preferred over a bind mount for databases?
2. What happens to a named volume on `docker rm`? How do you delete it?
3. What does `:ro` do?

### Project outcome
Proof (in `notes/day24.md`) that Postgres data survives container deletion, plus a `pgdata.tgz` backup. Add `*.tgz` to `.gitignore`.

### References
- [Volumes](https://docs.docker.com/engine/storage/volumes/)
- [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- [Postgres image on Docker Hub](https://hub.docker.com/_/postgres)

---

# Day 25: Docker Networking

**Goal:** Connect containers by name on a user-defined network.

### Concepts
- The default `bridge` network has **no name-based DNS**; user-defined bridges do.
- Publishing a port (`-p`) exposes a container to the host. Containers on the same network need no published ports to talk to each other.
- `localhost` inside a container refers to that container only.

### Hands-on
```bash
# 1. See the difference
docker network ls
docker network create taskboard-net

# 2. Start Postgres and Redis on the private network (no published ports)
docker run -d --name db --network taskboard-net \
  -e POSTGRES_USER=taskboard -e POSTGRES_PASSWORD=devpass -e POSTGRES_DB=taskboard \
  -v taskboard-pgdata:/var/lib/postgresql/data postgres:16-alpine
docker run -d --name cache --network taskboard-net redis:7-alpine

# 3. Reach them by name from other containers
docker run --rm --network taskboard-net redis:7-alpine redis-cli -h cache ping          # PONG
docker run --rm --network taskboard-net -e PGPASSWORD=devpass postgres:16-alpine \
  psql -h db -U taskboard -d taskboard -c "SELECT version();"

# 4. Prove they are not exposed on the host
nc -zv localhost 5432   # should fail

# 5. Inspect
docker network inspect taskboard-net --format '{{range .Containers}}{{.Name}} {{.IPv4Address}}{{"\n"}}{{end}}'
```

Save this as `scripts/dev-infra.sh` (then `chmod +x`) so you can recreate the setup in seconds:

```bash
#!/usr/bin/env bash
set -euo pipefail
docker network create taskboard-net 2>/dev/null || true
docker rm -f db cache 2>/dev/null || true
docker run -d --name db --network taskboard-net \
  -e POSTGRES_USER=taskboard -e POSTGRES_PASSWORD=devpass -e POSTGRES_DB=taskboard \
  -v taskboard-pgdata:/var/lib/postgresql/data postgres:16-alpine
docker run -d --name cache --network taskboard-net redis:7-alpine
```

> If the `taskboard-pgdata` volume still holds data from Day 24 under a different user or database, run `docker volume rm taskboard-pgdata` first so Postgres initializes with the `taskboard` user.

### Checkpoint
1. Why can `cache` be reached by name on `taskboard-net` but not on the default bridge?
2. When do you need `-p`, and when not?
3. What breaks if your app connects to `localhost:5432` from inside its own container?

### Project outcome
Postgres and Redis running on `taskboard-net`, reproducible with `scripts/dev-infra.sh`.

### References
- [Docker networking overview](https://docs.docker.com/engine/network/)
- [Bridge network driver](https://docs.docker.com/engine/network/drivers/bridge/)

---

# Day 26: Dockerfile Basics

**Goal:** Write the TaskBoard API and package it with a Dockerfile.

### Concepts
- Key instructions: `FROM`, `WORKDIR`, `COPY`, `RUN`, `ENV`, `EXPOSE`, `USER`, `CMD`/`ENTRYPOINT`.
- **Layer caching:** Docker reuses a layer if the instruction and its inputs are unchanged. Order instructions from least to most frequently changing.
- `CMD ["exec", "form"]` runs your process as PID 1 so it receives signals.

### Hands-on: the application

`api/requirements.txt`
```text
flask>=3.0,<4
gunicorn>=22
psycopg[binary]>=3.2,<4
redis>=5,<7
```

`api/app.py`
```python
import os
import time

import psycopg
import redis
from flask import Flask, jsonify, request

app = Flask(__name__)

DATABASE_URL = os.environ["DATABASE_URL"]
REDIS_URL = os.environ.get("REDIS_URL", "redis://cache:6379/0")
cache = redis.Redis.from_url(REDIS_URL, decode_responses=True)


def db():
    return psycopg.connect(DATABASE_URL, connect_timeout=3)


def init_db():
    for _ in range(15):
        try:
            with db() as conn:
                conn.execute(
                    "CREATE TABLE IF NOT EXISTS tasks ("
                    "id SERIAL PRIMARY KEY, title TEXT NOT NULL, "
                    "done BOOLEAN NOT NULL DEFAULT FALSE)"
                )
            return
        except psycopg.OperationalError:
            time.sleep(2)
    raise RuntimeError("database not reachable")


@app.get("/healthz")
def healthz():
    return jsonify(status="ok")


@app.get("/readyz")
def readyz():
    try:
        with db() as conn:
            conn.execute("SELECT 1")
        cache.ping()
    except Exception as exc:  # noqa: BLE001
        return jsonify(status="not ready", error=str(exc)), 503
    return jsonify(status="ready")


@app.get("/tasks")
def list_tasks():
    with db() as conn:
        rows = conn.execute("SELECT id, title, done FROM tasks ORDER BY id").fetchall()
    cache.incr("list_calls")
    return jsonify([{"id": r[0], "title": r[1], "done": r[2]} for r in rows])


@app.post("/tasks")
def create_task():
    title = (request.get_json(silent=True) or {}).get("title", "").strip()
    if not title:
        return jsonify(error="title is required"), 400
    with db() as conn:
        row = conn.execute(
            "INSERT INTO tasks (title) VALUES (%s) RETURNING id", (title,)
        ).fetchone()
    return jsonify(id=row[0], title=title, done=False), 201


@app.get("/stats")
def stats():
    return jsonify(list_calls=int(cache.get("list_calls") or 0))


init_db()
```

### Hands-on: a naive Dockerfile, then a cache-friendly one

`api/Dockerfile` (v1: works, but slow to rebuild and large)
```dockerfile
FROM python:3.12
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 8080
CMD ["gunicorn", "--preload", "--workers", "2", "--bind", "0.0.0.0:8080", "app:app"]
```

```bash
./scripts/dev-infra.sh
docker build -t taskboard-api:dev-single ./api
docker images taskboard-api

docker run -d --name api --network taskboard-net -p 8080:8080 \
  -e DATABASE_URL=postgresql://taskboard:devpass@db:5432/taskboard \
  -e REDIS_URL=redis://cache:6379/0 taskboard-api:dev-single

curl localhost:8080/healthz
curl -X POST localhost:8080/tasks -H 'Content-Type: application/json' -d '{"title":"learn containers"}'
curl localhost:8080/tasks
curl localhost:8080/stats
```

**Cache experiment.** Edit a comment in `app.py` and rebuild: dependencies reinstall every time, because `COPY . .` precedes `pip install`. Fix it by copying `requirements.txt` first:

```dockerfile
FROM python:3.12
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
EXPOSE 8080
CMD ["gunicorn", "--preload", "--workers", "2", "--bind", "0.0.0.0:8080", "app:app"]
```
Rebuild twice (second time after editing `app.py`) and watch the `CACHED` lines in the output.

### Checkpoint
1. Why does instruction order affect rebuild time?
2. Why is the JSON-array form of `CMD` preferred?
3. How does the API find the database inside the container network?

### Project outcome
API source committed; image `taskboard-api:dev-single` working against Postgres and Redis.

### References
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Build cache](https://docs.docker.com/build/cache/)
- [Flask quickstart](https://flask.palletsprojects.com/en/stable/quickstart/)
- [Gunicorn docs](https://docs.gunicorn.org/en/stable/)

---

# Day 27: Multi-Stage Builds & Image Optimization

**Goal:** Produce a small, non-root, health-checked production image.

### Concepts
- **Multi-stage builds** keep compilers and caches out of the final image.
- Smaller images pull faster and expose less attack surface.
- Run as a **non-root numeric UID** so Kubernetes can verify `runAsNonRoot`.
- `.dockerignore` keeps junk and secrets out of the build context.

### Hands-on

`api/.dockerignore`
```text
.git
__pycache__/
*.pyc
.env
*.md
Dockerfile
```

`api/Dockerfile` (final)
```dockerfile
# ---- build stage ----
FROM python:3.12-slim AS build
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# ---- runtime stage ----
FROM python:3.12-slim
ENV PYTHONUNBUFFERED=1 PYTHONDONTWRITEBYTECODE=1
RUN useradd --uid 10001 --no-create-home --shell /usr/sbin/nologin appuser
WORKDIR /app
COPY --from=build /install /usr/local
COPY app.py .
USER 10001
EXPOSE 8080
HEALTHCHECK --interval=15s --timeout=3s --start-period=10s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8080/healthz')"
CMD ["gunicorn", "--preload", "--workers", "2", "--bind", "0.0.0.0:8080", "app:app"]
```

```bash
docker rm -f api
docker build -t taskboard-api:1.0.0 ./api

# Compare
docker images taskboard-api --format 'table {{.Tag}}\t{{.Size}}'
docker history taskboard-api:1.0.0

# Verify non-root and health
docker run -d --name api --network taskboard-net -p 8080:8080 \
  -e DATABASE_URL=postgresql://taskboard:devpass@db:5432/taskboard taskboard-api:1.0.0
docker exec api id                       # uid=10001
sleep 20 && docker ps --format '{{.Names}}: {{.Status}}'   # (healthy)

# Optional: lint the Dockerfile
docker run --rm -i hadolint/hadolint < api/Dockerfile
```

### Checkpoint
1. How much smaller is `1.0.0` than `dev-single`, and why?
2. Why a numeric `USER 10001` instead of a name?
3. What does `docker ps` show for a container whose `HEALTHCHECK` fails?

### Project outcome
`taskboard-api:1.0.0`: multi-stage, non-root, health-checked. Record before/after sizes in `notes/day27.md`.

### References
- [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)
- [Dockerfile best practices](https://docs.docker.com/build/building/best-practices/)
- [Hadolint](https://github.com/hadolint/hadolint)

---

# Day 28: Docker Compose

**Goal:** Run the whole stack with a single command.

### Concepts
Compose describes multi-container apps declaratively: services, networks, volumes, health checks, and start-up ordering (`depends_on` with `condition: service_healthy`).

### Hands-on

`.env.example`
```text
POSTGRES_USER=taskboard
POSTGRES_PASSWORD=change-me
POSTGRES_DB=taskboard
```

```bash
cp .env.example .env
printf '.env\n*.tgz\n' >> .gitignore
```

`compose.yaml`
```yaml
services:
  api:
    build: ./api
    image: taskboard-api:1.0.0
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
      REDIS_URL: redis://cache:6379/0
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 3s
      retries: 10

  cache:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 10

volumes:
  pgdata:
```

```bash
# Clean up earlier manual containers first
docker rm -f api db cache 2>/dev/null; docker network rm taskboard-net 2>/dev/null

docker compose up -d --build
docker compose ps
docker compose logs -f api            # Ctrl+C to leave

curl -X POST localhost:8080/tasks -H 'Content-Type: application/json' -d '{"title":"compose works"}'
curl localhost:8080/tasks

# Failure drill: stop the DB and watch readiness
docker compose stop db
curl -i localhost:8080/readyz         # 503
docker compose start db && sleep 5 && curl localhost:8080/readyz   # ready

# Persistence: down keeps volumes, down -v deletes them
docker compose down
docker compose up -d && curl localhost:8080/tasks     # data still there
docker compose down -v
```

### Checkpoint
1. What does `condition: service_healthy` solve that plain `depends_on` does not?
2. What is the difference between `docker compose down` and `down -v`?
3. Why is `.env` in `.gitignore` while `.env.example` is committed?

### Project outcome
`docker compose up -d` brings up API + Postgres + Redis; `.env.example` committed.

### References
- [Docker Compose overview](https://docs.docker.com/compose/)
- [Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Control startup order](https://docs.docker.com/compose/how-tos/startup-order/)

---

# Day 29: Registries & Image Tagging

**Goal:** Publish images with a sound tagging strategy.

### Concepts
| Tag style | Example | Mutable? | Use |
|---|---|---|---|
| Semantic version | `1.0.0` | Treat as immutable | Releases |
| Git SHA | `sha-3f2a9c1` | Immutable | Traceability, CI |
| `latest` | `latest` | Yes | Avoid in production |
| Digest | `@sha256:...` | Immutable | Strongest pinning |

### Hands-on
```bash
# 1. A local registry for practice
docker run -d --name registry -p 5001:5000 registry:2
docker tag taskboard-api:1.0.0 localhost:5001/taskboard-api:1.0.0
docker push localhost:5001/taskboard-api:1.0.0
curl http://localhost:5001/v2/_catalog
docker image rm localhost:5001/taskboard-api:1.0.0
docker pull localhost:5001/taskboard-api:1.0.0
docker rm -f registry

# 2. GitHub Container Registry (token needs the write:packages scope)
export GH_USER=<your-github-username-lowercase>
echo "$GHCR_TOKEN" | docker login ghcr.io -u "$GH_USER" --password-stdin
GIT_SHA=$(git rev-parse --short HEAD)
docker tag taskboard-api:1.0.0 ghcr.io/$GH_USER/taskboard-api:1.0.0
docker tag taskboard-api:1.0.0 ghcr.io/$GH_USER/taskboard-api:sha-$GIT_SHA
docker push ghcr.io/$GH_USER/taskboard-api:1.0.0
docker push ghcr.io/$GH_USER/taskboard-api:sha-$GIT_SHA

# 3. Find the digest and pin by it
docker inspect --format '{{index .RepoDigests 0}}' ghcr.io/$GH_USER/taskboard-api:1.0.0
```

**Optional: automate with GitHub Actions** (`.github/workflows/image.yaml`)
```yaml
name: build-and-push
on:
  push:
    branches: [main]
    tags: ["v*"]
permissions:
  contents: read
  packages: write
jobs:
  image:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v6
        with:
          context: ./api
          push: true
          tags: ghcr.io/${{ github.repository_owner }}/taskboard-api:sha-${{ github.sha }}
```
GHCR requires lowercase image names; if your username has capitals, hard-code the lowercase form.

### Checkpoint
1. Why is `latest` risky in production?
2. How does pinning by digest differ from pinning by tag?
3. What does a private registry require from Kubernetes later? (hint: `imagePullSecrets`, Day 35)

### Project outcome
`taskboard-api:1.0.0` pushed to GHCR (or a local registry) with version and SHA tags. Tag your Git commit: `git tag v1.0.0`.

### References
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Docker Hub](https://docs.docker.com/docker-hub/)
- [Distribution (registry:2)](https://distribution.github.io/distribution/)
- [docker/build-push-action](https://github.com/docker/build-push-action)

---


# Day 30: Kubernetes Architecture

**Goal:** Understand the control plane and nodes, and stand up a local multi-node cluster.

### Concepts
```mermaid
flowchart TB
    subgraph CP[Control Plane]
        API[kube-apiserver] --- ETCD[(etcd)]
        SCH[kube-scheduler] --> API
        CM[kube-controller-manager] --> API
    end
    subgraph N1[Worker Node]
        K1[kubelet] --> CRI1[containerd] --> P1[Pods]
        KP1[kube-proxy]
    end
    API --> K1
```
- **kube-apiserver:** the only component you (and every other component) talk to; stores state in **etcd**.
- **Scheduler** assigns Pods to nodes; **controllers** reconcile *desired* vs *actual* state; **kubelet** runs Pods on a node via the container runtime.
- Everything is **declarative**: you describe the desired state in YAML and controllers make it true.

### Hands-on
Install `kubectl`, `kind`, and `helm` (links under References), then:

`kind-config.yaml`
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30080   # NodePort used on Day 34
        hostPort: 8080
        protocol: TCP
  - role: worker
  - role: worker
```

```bash
kind create cluster --name taskboard --config kind-config.yaml
kubectl cluster-info --context kind-taskboard
kubectl get nodes -o wide
kubectl get pods -n kube-system          # the control plane itself runs as Pods

# Explore the API
kubectl api-resources | head -20
kubectl explain pod.spec.containers.resources

# Follow one request through the system
kubectl run hello --image=nginx:alpine
kubectl describe pod hello               # Events: Scheduled -> Pulling -> Pulled -> Created -> Started
kubectl get pod hello -o yaml | head -40
kubectl delete pod hello

# Namespaces and a convenient default
kubectl create namespace taskboard
kubectl config set-context --current --namespace=taskboard
alias k=kubectl
```

### Checkpoint
1. Which component decides *which node* a Pod runs on? Which one actually starts the containers?
2. What happens if you delete a bare Pod (not managed by a controller)?
3. Why is the API server the only component that talks to etcd?

### Project outcome
A 3-node `kind` cluster named `taskboard` and a `taskboard` namespace. Sketch the request flow for `kubectl run` in `notes/day30.md`.

### References
- [Kubernetes components](https://kubernetes.io/docs/concepts/overview/components/)
- [kind quick start](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [Install kubectl](https://kubernetes.io/docs/tasks/tools/) · [Install Helm](https://helm.sh/docs/intro/install/)
- [kubectl quick reference](https://kubernetes.io/docs/reference/kubectl/quick-reference/)

---

# Day 31: Pods

**Goal:** Create and debug Pods, including init and sidecar containers.

### Concepts
- A **Pod** is the smallest deployable unit: one or more containers sharing network (one IP) and volumes.
- **Init containers** run to completion before app containers start. **Sidecars** run alongside the main container.
- Bare Pods are not rescheduled if their node dies; use a Deployment in real workloads (Day 32).

### Hands-on

`k8s/pod-demo.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-demo
  namespace: taskboard
  labels:
    app: web-demo
spec:
  initContainers:
    - name: init-content
      image: busybox:1.36
      command: ["sh", "-c", "echo '<h1>Hello from an init container</h1>' > /work/index.html"]
      volumeMounts:
        - name: html
          mountPath: /work
  containers:
    - name: web
      image: nginx:1.27-alpine
      ports:
        - containerPort: 80
      volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
    - name: log-sidecar
      image: busybox:1.36
      command: ["sh", "-c", "while true; do date; sleep 10; done"]
  volumes:
    - name: html
      emptyDir: {}
```

```bash
kubectl apply -f k8s/pod-demo.yaml
kubectl get pod web-demo -w                 # Init:0/1 -> PodInitializing -> Running 2/2
kubectl describe pod web-demo
kubectl logs web-demo -c log-sidecar
kubectl port-forward pod/web-demo 8081:80 &
curl localhost:8081; kill %1
kubectl exec -it web-demo -c web -- sh
```

**Failure drills** (this is how you learn to debug):
```bash
# 1. Bad image -> ImagePullBackOff
kubectl run bad-image --image=nginx:does-not-exist
kubectl describe pod bad-image | tail -15

# 2. Crashing container -> CrashLoopBackOff
kubectl run crasher --image=busybox:1.36 -- sh -c 'echo boom; exit 1'
kubectl logs crasher --previous
kubectl get pod crasher                      # note RESTARTS

# 3. Unschedulable -> Pending
kubectl run hungry --image=nginx:alpine --overrides='{"spec":{"containers":[{"name":"hungry","image":"nginx:alpine","resources":{"requests":{"cpu":"100"}}}]}}'
kubectl describe pod hungry | tail -8        # Insufficient cpu

kubectl delete pod bad-image crasher hungry web-demo
```

**Debug flow to memorize:** `kubectl get pod` → `kubectl describe pod` (read *Events*) → `kubectl logs [--previous]` → `kubectl exec` → `kubectl get events --sort-by=.lastTimestamp`.

### Checkpoint
1. Why do containers in one Pod reach each other on `localhost`?
2. What is the difference between `ImagePullBackOff` and `CrashLoopBackOff`?
3. When would you use an init container instead of a sidecar?

### Project outcome
Create `notes/debug-cheatsheet.md` listing each drill, the symptom, and the fix. Keep it; you will use it on Days 33 and 37.

### References
- [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Init containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
- [Debug Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)

---

# Day 32: Deployments & ReplicaSets

**Goal:** Run TaskBoard on Kubernetes with self-healing replicas.

### Concepts
- **Deployment → ReplicaSet → Pods.** The Deployment manages ReplicaSets (one per revision); the ReplicaSet keeps the desired number of Pods alive.
- **Labels and selectors** connect everything (Deployments to Pods, Services to Pods).
- Today's Postgres uses `emptyDir` and inline credentials on purpose. **Day 35 replaces both** with a StatefulSet, PVC, and Secret.

### Hands-on

`k8s/base/namespace.yaml`
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: taskboard
```

`k8s/base/redis.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: taskboard
spec:
  replicas: 1
  selector:
    matchLabels: {app: redis}
  template:
    metadata:
      labels: {app: redis}
    spec:
      containers:
        - name: redis
          image: redis:7-alpine
          args: ["--save", "", "--appendonly", "no"]   # cache only, nothing to persist
          ports:
            - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: taskboard
spec:
  selector: {app: redis}
  ports:
    - port: 6379
      targetPort: 6379
```

`k8s/base/postgres.yaml` (temporary version)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: taskboard
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels: {app: postgres}
  template:
    metadata:
      labels: {app: postgres}
    spec:
      containers:
        - name: postgres
          image: postgres:16-alpine
          env:
            - {name: POSTGRES_USER, value: taskboard}
            - {name: POSTGRES_PASSWORD, value: devpass}
            - {name: POSTGRES_DB, value: taskboard}
          ports:
            - containerPort: 5432
          volumeMounts:
            - {name: data, mountPath: /var/lib/postgresql/data}
      volumes:
        - name: data
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: taskboard
spec:
  selector: {app: postgres}
  ports:
    - port: 5432
      targetPort: 5432
```

`k8s/base/api.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: taskboard
spec:
  replicas: 2
  selector:
    matchLabels: {app: api}
  template:
    metadata:
      labels: {app: api}
    spec:
      containers:
        - name: api
          image: taskboard-api:1.0.0
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
          env:
            - name: DATABASE_URL
              value: postgresql://taskboard:devpass@postgres:5432/taskboard
            - name: REDIS_URL
              value: redis://redis:6379/0
          readinessProbe:
            httpGet: {path: /readyz, port: 8080}
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: api
  namespace: taskboard
spec:
  selector: {app: api}
  ports:
    - port: 80
      targetPort: 8080
```

```bash
# kind nodes cannot see your local Docker images, so load the image into the cluster
kind load docker-image taskboard-api:1.0.0 --name taskboard

kubectl apply -f k8s/base/namespace.yaml
kubectl apply -f k8s/base/redis.yaml -f k8s/base/postgres.yaml -f k8s/base/api.yaml
kubectl get all -n taskboard
kubectl rollout status deployment/api

# Use it
kubectl port-forward svc/api 8080:80 &
curl -X POST localhost:8080/tasks -H 'Content-Type: application/json' -d '{"title":"first task on k8s"}'
curl localhost:8080/tasks
kill %1
```

**Self-healing and scaling drills**
```bash
kubectl get pods -l app=api -w &
kubectl delete pod "$(kubectl get pod -l app=api -o name | head -1)"      # a replacement Pod appears
kubectl scale deployment/api --replicas=4
kubectl get rs                                # one ReplicaSet owns the Pods
kubectl get pods --show-labels
kubectl get pods -l app=api -o wide           # spread across worker nodes
kubectl scale deployment/api --replicas=2
kill %1
```

### Checkpoint
1. Who recreates a deleted Pod: the Deployment or the ReplicaSet?
2. Why does the Postgres Deployment use `strategy: Recreate`?
3. What would break if the Service selector did not match the Pod labels?

### Project outcome
TaskBoard (API ×2, Redis, Postgres) running in the `taskboard` namespace and answering `POST`/`GET /tasks` through `port-forward`.

### References
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Labels and selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
- [Loading images into kind](https://kind.sigs.k8s.io/docs/user/quick-start/#loading-an-image-into-your-cluster)

---

# Day 33: Rolling Updates & Rollbacks

**Goal:** Ship a new version with zero downtime, break one on purpose, and roll back.

### Concepts
- A rolling update replaces Pods gradually, guided by `maxSurge` and `maxUnavailable` and gated by **readiness probes**.
- Every change to the Pod template creates a new **revision**; `rollout undo` returns to a previous one.
- A Deployment never removes old Pods faster than new ones become Ready, so a bad release stalls instead of causing an outage.

### Hands-on: build v1.1.0
Add a version endpoint to `api/app.py` (above `init_db()`):
```python
APP_VERSION = os.environ.get("APP_VERSION", "dev")


@app.get("/version")
def version():
    return jsonify(version=APP_VERSION)
```
Add two lines to the **runtime stage** of `api/Dockerfile`, after the `ENV PYTHONUNBUFFERED...` line:
```dockerfile
ARG APP_VERSION=dev
ENV APP_VERSION=$APP_VERSION
```
Update `k8s/base/api.yaml`: set `replicas: 3` and add, under `spec:` (sibling of `replicas`):
```yaml
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

```bash
docker build --build-arg APP_VERSION=1.0.0 -t taskboard-api:1.0.0 ./api    # rebuild baseline with /version
docker build --build-arg APP_VERSION=1.1.0 -t taskboard-api:1.1.0 ./api
kind load docker-image taskboard-api:1.0.0 taskboard-api:1.1.0 --name taskboard
kubectl apply -f k8s/base/api.yaml
kubectl rollout status deployment/api
```

### Hands-on: release with live traffic
Terminal 1 sends continuous requests *inside the cluster* (port-forward would pin to one Pod and hide the rollout):
```bash
kubectl run watcher --rm -it --image=busybox:1.36 --restart=Never -- \
  sh -c 'while true; do wget -q -O- -T 2 http://api/version || echo FAIL; echo; sleep 0.5; done'
```
Terminal 2:
```bash
kubectl annotate deployment/api kubernetes.io/change-cause="release 1.1.0" --overwrite
kubectl set image deployment/api api=taskboard-api:1.1.0
kubectl rollout status deployment/api
kubectl rollout history deployment/api
```
Terminal 1 shows `1.0.0` flipping to `1.1.0` with no `FAIL` lines.

### Hands-on: a bad release and a rollback
```bash
kubectl annotate deployment/api kubernetes.io/change-cause="bad image (drill)" --overwrite
kubectl set image deployment/api api=taskboard-api:9.9.9
kubectl rollout status deployment/api --timeout=45s     # times out
kubectl get pods                                         # old Pods still serving; new one ErrImagePull
kubectl rollout undo deployment/api
kubectl rollout status deployment/api
kubectl rollout history deployment/api

# Other handy commands
kubectl rollout restart deployment/api                   # roll Pods without changing the image
kubectl rollout undo deployment/api --to-revision=1
```

### Checkpoint
1. Why did the bad release not cause downtime?
2. What do `maxSurge: 1` and `maxUnavailable: 0` mean for 3 replicas?
3. How is `rollout undo` different from editing the YAML back by hand?

### Project outcome
Write `docs/runbook-rollout.md` documenting: how to release, how to watch it, how to detect a stuck rollout, how to roll back. Image `taskboard-api:1.1.0` is now the current version.

### References
- [Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)
- [Rolling back a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)
- [kubectl rollout](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_rollout/)

---

# Day 34: Services & Gateway API

**Goal:** Expose the app inside and outside the cluster.

### Concepts
| Service type | Reachable from | Typical use |
|---|---|---|
| `ClusterIP` (default) | Inside the cluster | Service-to-service traffic |
| `NodePort` | Node IP + high port | Debugging, simple setups |
| `LoadBalancer` | Cloud load balancer | Production exposure |

- Every Service gets a DNS name: `<svc>.<namespace>.svc.cluster.local`.
- **Gateway API** (`GatewayClass`, `Gateway`, `HTTPRoute`) is the modern way to route external HTTP traffic. It separates roles: platform teams own the Gateway, app teams own routes.
- **Important:** the community **Ingress NGINX controller was retired in March 2026** and no longer receives fixes. The Kubernetes project recommends Gateway API. You will still meet `Ingress` objects in existing clusters, so learn to read them, but build new work on Gateway API.

### Hands-on: Services and DNS
```bash
kubectl get svc,endpointslices -n taskboard
kubectl run -it --rm dns-test --image=busybox:1.36 --restart=Never -- sh -c '
  nslookup api.taskboard.svc.cluster.local
  wget -qO- http://api/healthz'

# NodePort: reachable from your laptop because kind-config.yaml maps 30080 -> localhost:8080
cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: api-nodeport
  namespace: taskboard
spec:
  type: NodePort
  selector: {app: api}
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
YAML
curl localhost:8080/healthz
kubectl delete svc api-nodeport -n taskboard
```

### Hands-on: Gateway API with Envoy Gateway
Install Envoy Gateway using the **current version shown on its quickstart page** (link below); it also installs the Gateway API CRDs.
```bash
helm install eg oci://docker.io/envoyproxy/gateway-helm \
  --version <ENVOY_GATEWAY_VERSION> -n envoy-gateway-system --create-namespace
kubectl wait --timeout=5m -n envoy-gateway-system deployment/envoy-gateway --for=condition=Available
```

`k8s/gateway.yaml`
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: eg
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: taskboard-gw
  namespace: taskboard
spec:
  gatewayClassName: eg
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: Same
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: api
  namespace: taskboard
spec:
  parentRefs:
    - name: taskboard-gw
  hostnames: ["taskboard.localhost"]
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: api
          port: 80
```

```bash
kubectl apply -f k8s/gateway.yaml
kubectl get gateway,httproute -n taskboard          # ADDRESS may stay empty on kind; that is expected

# kind has no cloud load balancer, so port-forward to the Envoy proxy Service
ENVOY_SVC=$(kubectl get svc -n envoy-gateway-system \
  --selector=gateway.envoyproxy.io/owning-gateway-namespace=taskboard,gateway.envoyproxy.io/owning-gateway-name=taskboard-gw \
  -o jsonpath='{.items[0].metadata.name}')
kubectl -n envoy-gateway-system port-forward svc/$ENVOY_SVC 8888:80 &

curl -H "Host: taskboard.localhost" localhost:8888/version
curl -H "Host: taskboard.localhost" localhost:8888/tasks
```

**Stretch:** add a second rule that routes `/v2` to a different backend with a `weight` split (canary), using `backendRefs` weights.

### Checkpoint
1. Why does `curl http://api/healthz` work from a Pod in `taskboard` but needs `api.taskboard` from another namespace?
2. What are the three Gateway API objects and who typically owns each?
3. Why is `Ingress NGINX` a poor choice for new clusters today?

### Project outcome
TaskBoard reachable through a Gateway at `taskboard.localhost` (via the port-forward on kind). Commit `k8s/gateway.yaml`.

### References
- [Service](https://kubernetes.io/docs/concepts/services-networking/service/) · [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Gateway API](https://gateway-api.sigs.k8s.io/) · [Gateway API guides](https://gateway-api.sigs.k8s.io/guides/)
- [Envoy Gateway quickstart](https://gateway.envoyproxy.io/docs/tasks/quickstart/)
- [Ingress NGINX retirement notice](https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/)
- [Ingress (legacy API)](https://kubernetes.io/docs/concepts/services-networking/ingress/)

---

# Day 35: ConfigMaps, Secrets & Persistent Storage

**Goal:** Externalize configuration and make Postgres data survive Pod deletion.

### Concepts
- **ConfigMap:** non-sensitive config. **Secret:** sensitive values. Secrets are only **base64-encoded by default, not encrypted**; restrict access with RBAC and enable encryption at rest in real clusters.
- **PersistentVolumeClaim (PVC):** a request for storage; a **StorageClass** provisions the volume. `kind` ships a default local-path StorageClass.
- **StatefulSet:** stable identity (`postgres-0`) and per-Pod volumes via `volumeClaimTemplates`. Use it for databases.

### Hands-on

`k8s/base/config.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: taskboard-config
  namespace: taskboard
data:
  DB_HOST: postgres
  DB_NAME: taskboard
  REDIS_URL: redis://redis:6379/0
```

```bash
# Create the Secret from the command line; never commit real secrets to Git
kubectl -n taskboard create secret generic db-credentials \
  --from-literal=POSTGRES_USER=taskboard \
  --from-literal=POSTGRES_PASSWORD="$(openssl rand -hex 16)"
kubectl -n taskboard get secret db-credentials -o jsonpath='{.data.POSTGRES_USER}' | base64 -d; echo
```

`k8s/base/postgres.yaml` (replace the temporary version)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: taskboard
spec:
  clusterIP: None            # headless Service for the StatefulSet
  selector: {app: postgres}
  ports:
    - name: pg
      port: 5432
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: taskboard
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels: {app: postgres}
  template:
    metadata:
      labels: {app: postgres}
    spec:
      containers:
        - name: postgres
          image: postgres:16-alpine
          ports:
            - {name: pg, containerPort: 5432}
          env:
            - name: POSTGRES_USER
              valueFrom: {secretKeyRef: {name: db-credentials, key: POSTGRES_USER}}
            - name: POSTGRES_PASSWORD
              valueFrom: {secretKeyRef: {name: db-credentials, key: POSTGRES_PASSWORD}}
            - name: POSTGRES_DB
              valueFrom: {configMapKeyRef: {name: taskboard-config, key: DB_NAME}}
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
          volumeMounts:
            - {name: data, mountPath: /var/lib/postgresql/data}
          readinessProbe:
            exec:
              command: ["sh", "-c", "pg_isready -U \"$POSTGRES_USER\""]
            periodSeconds: 5
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

In `k8s/base/api.yaml`, replace the container's `env:` block with:
```yaml
          env:
            - name: POSTGRES_USER
              valueFrom: {secretKeyRef: {name: db-credentials, key: POSTGRES_USER}}
            - name: POSTGRES_PASSWORD
              valueFrom: {secretKeyRef: {name: db-credentials, key: POSTGRES_PASSWORD}}
            - name: DB_HOST
              valueFrom: {configMapKeyRef: {name: taskboard-config, key: DB_HOST}}
            - name: DB_NAME
              valueFrom: {configMapKeyRef: {name: taskboard-config, key: DB_NAME}}
            - name: DATABASE_URL      # must come after the variables it references
              value: postgresql://$(POSTGRES_USER):$(POSTGRES_PASSWORD)@$(DB_HOST):5432/$(DB_NAME)
            - name: REDIS_URL
              valueFrom: {configMapKeyRef: {name: taskboard-config, key: REDIS_URL}}
```

```bash
kubectl apply -f k8s/base/config.yaml
kubectl delete deployment/postgres service/postgres -n taskboard     # remove the temporary version
kubectl apply -f k8s/base/postgres.yaml
kubectl get pvc,pv -n taskboard
kubectl rollout status statefulset/postgres
kubectl apply -f k8s/base/api.yaml
kubectl rollout status deployment/api

# Persistence drill
ENVOY_SVC=$(kubectl get svc -n envoy-gateway-system \
  --selector=gateway.envoyproxy.io/owning-gateway-namespace=taskboard,gateway.envoyproxy.io/owning-gateway-name=taskboard-gw \
  -o jsonpath='{.items[0].metadata.name}')
pkill -f "port-forward svc/$ENVOY_SVC" 2>/dev/null   # drop any stale forward from Day 34
kubectl -n envoy-gateway-system port-forward svc/$ENVOY_SVC 8888:80 &
sleep 2
curl -H "Host: taskboard.localhost" -H 'Content-Type: application/json' \
  -d '{"title":"survives restarts"}' localhost:8888/tasks
kubectl delete pod postgres-0 -n taskboard
kubectl wait --for=condition=Ready pod/postgres-0 -n taskboard --timeout=120s
curl -H "Host: taskboard.localhost" localhost:8888/tasks              # the task is still there
```

**Private registry (reference).** If your image lives in a private GHCR repo, create a pull secret and reference it in the Pod spec:
```bash
kubectl -n taskboard create secret docker-registry ghcr-pull \
  --docker-server=ghcr.io --docker-username=$GH_USER --docker-password="$GHCR_TOKEN"
# then add under spec.template.spec:  imagePullSecrets: [{name: ghcr-pull}]
```

### Checkpoint
1. Why is a base64-encoded Secret not "secure" on its own?
2. What happens to the PVC when you delete `postgres-0`? When you delete the StatefulSet?
3. Why must `DATABASE_URL` be listed after the variables it interpolates?

### Project outcome
Credentials live in a Secret, settings in a ConfigMap, and Postgres uses a PVC-backed StatefulSet. Data survives Pod deletion.

### References
- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) · [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) · [Good practices for Secrets](https://kubernetes.io/docs/concepts/security/secrets-good-practices/)
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [Pull an image from a private registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

---

# Day 36: Resources, Probes & Autoscaling

**Goal:** Make the API self-healing, right-sized, and automatically scalable.

### Concepts
- **Requests** drive scheduling; **limits** cap usage. Exceeding a memory limit gets the container OOM-killed; exceeding a CPU limit throttles it.
- **QoS classes:** `Guaranteed` (requests = limits), `Burstable`, `BestEffort`. Under node pressure BestEffort Pods are evicted first.
- **Probes:** `startupProbe` (gives slow starters time), `readinessProbe` (in or out of Service endpoints), `livenessProbe` (restart when stuck).
- **Keep liveness probes cheap and dependency-free.** If liveness checked the database, a DB outage would restart every API Pod and make things worse. That is why `/healthz` does not touch the DB and `/readyz` does.
- **HPA** adjusts replicas from metrics (needs **metrics-server** and CPU *requests*).

### Hands-on
Add to the API container in `k8s/base/api.yaml` (replace the existing `readinessProbe`):
```yaml
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 256Mi
          startupProbe:
            httpGet: {path: /healthz, port: 8080}
            periodSeconds: 2
            failureThreshold: 30
          readinessProbe:
            httpGet: {path: /readyz, port: 8080}
            periodSeconds: 5
            failureThreshold: 3
          livenessProbe:
            httpGet: {path: /healthz, port: 8080}
            periodSeconds: 10
            failureThreshold: 3
```
Also **remove the `replicas:` line** from the Deployment so the HPA owns the replica count.

```bash
kubectl apply -f k8s/base/api.yaml
kubectl rollout status deployment/api
kubectl describe pod -l app=api | grep -E "QoS|Limits|Requests" 

# Liveness drill: stop PID 1 and watch the container restart
kubectl exec deploy/api -- sh -c 'kill 1'
kubectl get pods -l app=api            # RESTARTS increments

# Readiness drill: scale Postgres to 0 and watch the API Pods leave the Service
kubectl scale statefulset/postgres --replicas=0
kubectl get endpointslices -n taskboard -l kubernetes.io/service-name=api
kubectl scale statefulset/postgres --replicas=1
```

**Install metrics-server (kind needs one extra flag):**
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch -n kube-system deployment metrics-server --type=json \
  -p '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
kubectl top nodes && kubectl top pods -n taskboard
```

`k8s/base/hpa.yaml`
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api
  namespace: taskboard
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 6
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

```bash
kubectl apply -f k8s/base/hpa.yaml

# Generate load from a separate namespace
kubectl create namespace loadtest
kubectl -n loadtest create deployment load --image=busybox:1.36 --replicas=4 -- \
  sh -c 'while true; do wget -q -O- http://api.taskboard/tasks > /dev/null; done'
kubectl get hpa -n taskboard -w        # TARGETS climbs, REPLICAS grows toward 6

kubectl delete namespace loadtest      # stop the load; replicas scale back down after a few minutes
```

### Checkpoint
1. Which QoS class does the API Pod have? How do you know?
2. Why does the HPA need CPU *requests* to be set?
3. What goes wrong if the liveness probe depends on the database?

### Project outcome
API with resources, three probes, and an HPA (2–6 replicas). Screenshot or paste `kubectl get hpa` output showing scale-up into `notes/day36.md`.

### References
- [Resource management for containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Pod QoS classes](https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/)
- [Liveness, readiness, and startup probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [HPA walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
- [metrics-server](https://github.com/kubernetes-sigs/metrics-server)

---

# Day 37: Container & Cluster Security

**Goal:** Scan images, run Pods with least privilege, restrict access with RBAC, and limit traffic with NetworkPolicies.

### Concepts
| Layer | Control | Tool / object |
|---|---|---|
| Image | Vulnerability scanning, minimal base, non-root | Trivy, Grype, Docker Scout |
| Pod | Drop capabilities, read-only root FS, seccomp, no privilege escalation | `securityContext` |
| Namespace | Enforce Pod Security Standards (`privileged` / `baseline` / `restricted`) | Pod Security Admission labels |
| API access | Least-privilege identities | RBAC (`Role`, `RoleBinding`, `ServiceAccount`) |
| Network | Default deny, explicit allow | `NetworkPolicy` |
| Secrets | Not in Git, not in images, encrypted at rest | Secrets, external secret managers |

### Hands-on 1: scan the image
```bash
# Pin the Trivy image to a released version in CI instead of :latest
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image --severity HIGH,CRITICAL taskboard-api:1.1.0

# Fail a build on findings
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image --exit-code 1 --severity CRITICAL taskboard-api:1.1.0

# Scan your manifests for misconfigurations
docker run --rm -v "$PWD:/work" aquasec/trivy:latest config /work/k8s
```
Fix findings by rebuilding on a newer base image (`docker build --pull`), then rescan. Record before/after counts in `notes/day37.md`.

### Hands-on 2: harden the workloads
`k8s/base/api.yaml` (full hardened version)
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api
  namespace: taskboard
automountServiceAccountToken: false     # the API never calls the Kubernetes API
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: taskboard
  labels: {app: api}
spec:
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate: {maxSurge: 1, maxUnavailable: 0}
  selector:
    matchLabels: {app: api}
  template:
    metadata:
      labels: {app: api}
    spec:
      serviceAccountName: api
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        runAsGroup: 10001
        seccompProfile: {type: RuntimeDefault}
      containers:
        - name: api
          image: taskboard-api:1.1.0
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
          env:
            - name: POSTGRES_USER
              valueFrom: {secretKeyRef: {name: db-credentials, key: POSTGRES_USER}}
            - name: POSTGRES_PASSWORD
              valueFrom: {secretKeyRef: {name: db-credentials, key: POSTGRES_PASSWORD}}
            - name: DB_HOST
              valueFrom: {configMapKeyRef: {name: taskboard-config, key: DB_HOST}}
            - name: DB_NAME
              valueFrom: {configMapKeyRef: {name: taskboard-config, key: DB_NAME}}
            - name: DATABASE_URL
              value: postgresql://$(POSTGRES_USER):$(POSTGRES_PASSWORD)@$(DB_HOST):5432/$(DB_NAME)
            - name: REDIS_URL
              valueFrom: {configMapKeyRef: {name: taskboard-config, key: REDIS_URL}}
          resources:
            requests: {cpu: 100m, memory: 128Mi}
            limits: {cpu: 500m, memory: 256Mi}
          startupProbe:
            httpGet: {path: /healthz, port: 8080}
            periodSeconds: 2
            failureThreshold: 30
          readinessProbe:
            httpGet: {path: /readyz, port: 8080}
            periodSeconds: 5
          livenessProbe:
            httpGet: {path: /healthz, port: 8080}
            periodSeconds: 10
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities: {drop: ["ALL"]}
          volumeMounts:
            - {name: tmp, mountPath: /tmp}     # gunicorn needs a writable temp dir
      volumes:
        - name: tmp
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: api
  namespace: taskboard
spec:
  selector: {app: api}
  ports:
    - port: 80
      targetPort: 8080
```

Add these to **Postgres** (`postgres:16-alpine` runs as UID 70) under `spec.template.spec`:
```yaml
      securityContext:
        runAsNonRoot: true
        runAsUser: 70
        runAsGroup: 70
        fsGroup: 70
        seccompProfile: {type: RuntimeDefault}
```
and under its container:
```yaml
          securityContext:
            allowPrivilegeEscalation: false
            capabilities: {drop: ["ALL"]}
```
Add these to **Redis** (`redis:7-alpine` runs as UID 999) the same way, with `runAsUser: 999`, `runAsGroup: 999`, and `readOnlyRootFilesystem: true` on the container (Redis is configured not to write to disk).

Make the namespace enforce the `restricted` profile, declaratively. Update `k8s/base/namespace.yaml`:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: taskboard
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
```
```bash
# Preview what would be rejected, without changing anything
kubectl label --dry-run=server --overwrite ns taskboard pod-security.kubernetes.io/enforce=restricted

kubectl apply -f k8s/base/namespace.yaml
kubectl apply -f k8s/base/api.yaml -f k8s/base/postgres.yaml -f k8s/base/redis.yaml
kubectl get pods -n taskboard
kubectl get events -n taskboard --sort-by=.lastTimestamp | tail    # admission errors, if any

# Prove the policy works: this must be rejected
kubectl -n taskboard run rogue --image=nginx:alpine --privileged
```
If a Pod is rejected, `kubectl describe` on the Deployment/StatefulSet/ReplicaSet shows which `restricted` rule it broke. Fixing that is the learning goal.

### Hands-on 3: RBAC
`k8s/base/rbac.yaml`
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-viewer
  namespace: taskboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: viewer
  namespace: taskboard
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log", "services", "configmaps"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets", "statefulsets"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-viewer
  namespace: taskboard
subjects:
  - kind: ServiceAccount
    name: dev-viewer
    namespace: taskboard
roleRef:
  kind: Role
  name: viewer
  apiGroup: rbac.authorization.k8s.io
```
```bash
kubectl apply -f k8s/base/rbac.yaml
SA=system:serviceaccount:taskboard:dev-viewer
kubectl auth can-i list pods     -n taskboard --as=$SA   # yes
kubectl auth can-i delete pods   -n taskboard --as=$SA   # no
kubectl auth can-i get secrets   -n taskboard --as=$SA   # no
kubectl auth can-i list pods     -n default   --as=$SA   # no (namespaced Role)
```

### Hands-on 4: NetworkPolicies
`k8s/base/netpol.yaml`
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: taskboard
spec:
  podSelector: {}
  policyTypes: [Ingress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-gateway-to-api
  namespace: taskboard
spec:
  podSelector:
    matchLabels: {app: api}
  policyTypes: [Ingress]
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: envoy-gateway-system
      ports:
        - {protocol: TCP, port: 8080}
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-postgres
  namespace: taskboard
spec:
  podSelector:
    matchLabels: {app: postgres}
  policyTypes: [Ingress]
  ingress:
    - from:
        - podSelector:
            matchLabels: {app: api}
      ports:
        - {protocol: TCP, port: 5432}
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-redis
  namespace: taskboard
spec:
  podSelector:
    matchLabels: {app: redis}
  policyTypes: [Ingress]
  ingress:
    - from:
        - podSelector:
            matchLabels: {app: api}
      ports:
        - {protocol: TCP, port: 6379}
```
```bash
kubectl apply -f k8s/base/netpol.yaml

# From another namespace, direct access to the API and DB should now time out
kubectl run probe -n default --rm -it --image=busybox:1.36 --restart=Never -- \
  sh -c 'wget -qO- -T 3 http://api.taskboard/healthz || echo BLOCKED'

# Through the Gateway it must still work
curl -H "Host: taskboard.localhost" localhost:8888/healthz
```
> NetworkPolicies only take effect if the cluster's network plugin enforces them. Depending on your `kind` version, the default CNI may not. If `probe` is **not** blocked, the manifests are still correct; to see enforcement, create a cluster with a policy-enforcing CNI (Calico or Cilium) following the `kind` docs. Note that `kubectl port-forward` bypasses NetworkPolicies, so test with a Pod as above.

**Stretch:** add a default-deny **egress** policy and allow only DNS plus the specific destinations the API needs.

### Checkpoint
1. What three `securityContext` settings give the biggest hardening for the least effort?
2. Why disable `automountServiceAccountToken` for the API?
3. What does a default-deny NetworkPolicy do, and why must you add explicit allows afterwards?

### Project outcome
Scanned image, `restricted` namespace, hardened workloads, read-only RBAC identity, and default-deny ingress with explicit allows. Add a "Security" section to your project README.

### References
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) · [Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/)
- [Configure a security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Trivy](https://trivy.dev/) · [Docker Scout](https://docs.docker.com/scout/)
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html) · [OWASP Kubernetes Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)

---

# Day 38: Helm & Kustomize

**Goal:** Package and parameterize manifests for multiple environments.

### Concepts
| | Helm | Kustomize |
|---|---|---|
| Model | Templates + values, versioned **releases** | Plain YAML + **patches/overlays** |
| Strengths | Packaging, sharing, upgrade/rollback history | No templating language, built into `kubectl` |
| Use when | Distributing an app or consuming third-party charts | Managing per-environment variations of your own manifests |

They combine well: many teams use Helm for third-party software and Kustomize for their own.

### Hands-on 1: a Helm chart for the API
```bash
mkdir -p charts/api/templates
```
`charts/api/Chart.yaml`
```yaml
apiVersion: v2
name: api
description: TaskBoard API
type: application
version: 0.1.0
appVersion: "1.1.0"
```
`charts/api/values.yaml`
```yaml
replicaCount: 2
autoscaling:
  enabled: true        # when true, the HPA owns replicas
image:
  repository: taskboard-api
  tag: "1.1.0"
  pullPolicy: IfNotPresent
service:
  port: 80
resources:
  requests: {cpu: 100m, memory: 128Mi}
  limits: {cpu: 500m, memory: 256Mi}
```
`charts/api/templates/deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
  labels:
    app: {{ .Chart.Name }}
    app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      automountServiceAccountToken: false
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        runAsGroup: 10001
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: api
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: 8080
          env:
            - name: POSTGRES_USER
              valueFrom: {secretKeyRef: {name: db-credentials, key: POSTGRES_USER}}
            - name: POSTGRES_PASSWORD
              valueFrom: {secretKeyRef: {name: db-credentials, key: POSTGRES_PASSWORD}}
            - name: DB_HOST
              valueFrom: {configMapKeyRef: {name: taskboard-config, key: DB_HOST}}
            - name: DB_NAME
              valueFrom: {configMapKeyRef: {name: taskboard-config, key: DB_NAME}}
            - name: DATABASE_URL
              value: postgresql://$(POSTGRES_USER):$(POSTGRES_PASSWORD)@$(DB_HOST):5432/$(DB_NAME)
            - name: REDIS_URL
              valueFrom: {configMapKeyRef: {name: taskboard-config, key: REDIS_URL}}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          readinessProbe:
            httpGet: {path: /readyz, port: 8080}
            periodSeconds: 5
          livenessProbe:
            httpGet: {path: /healthz, port: 8080}
            periodSeconds: 10
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          volumeMounts:
            - {name: tmp, mountPath: /tmp}
      volumes:
        - name: tmp
          emptyDir: {}
```
`charts/api/templates/service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Chart.Name }}
spec:
  selector:
    app: {{ .Chart.Name }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 8080
```
```bash
helm lint charts/api
helm template api charts/api -n taskboard | head -50

# Hand the API over from raw manifests to Helm (names clash otherwise)
kubectl delete deployment/api service/api -n taskboard
helm install api charts/api -n taskboard --set image.tag=1.0.0
helm list -n taskboard

# Upgrade, inspect, roll back
helm upgrade api charts/api -n taskboard --set image.tag=1.1.0
helm history api -n taskboard
helm rollback api 1 -n taskboard
curl -H "Host: taskboard.localhost" localhost:8888/version      # back on 1.0.0
helm upgrade api charts/api -n taskboard                          # defaults to 1.1.0 again
```

### Hands-on 2: Kustomize overlays for environments
`k8s/base/kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespace.yaml
  - config.yaml
  - redis.yaml
  - postgres.yaml
  - api.yaml
  - hpa.yaml
  - netpol.yaml
  - rbac.yaml
```
`k8s/overlays/dev/kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: taskboard-dev
resources:
  - ../../base
images:
  - name: taskboard-api
    newTag: "1.1.0"
patches:
  - target: {kind: HorizontalPodAutoscaler, name: api}
    patch: |-
      - op: replace
        path: /spec/maxReplicas
        value: 3
```
`k8s/overlays/prod/kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: taskboard-prod
resources:
  - ../../base
images:
  - name: taskboard-api
    newTag: "1.1.0"
patches:
  - target: {kind: HorizontalPodAutoscaler, name: api}
    patch: |-
      - op: replace
        path: /spec/minReplicas
        value: 3
      - op: replace
        path: /spec/maxReplicas
        value: 10
```
```bash
kubectl kustomize k8s/overlays/dev | less        # review the rendered YAML first

kubectl create namespace taskboard-dev
kubectl -n taskboard-dev create secret generic db-credentials \
  --from-literal=POSTGRES_USER=taskboard \
  --from-literal=POSTGRES_PASSWORD="$(openssl rand -hex 16)"
kubectl apply -k k8s/overlays/dev
kubectl get all -n taskboard-dev

kubectl delete -k k8s/overlays/dev               # clean up; keep your main taskboard namespace
```
Secrets stay out of the overlays on purpose; create them per namespace as shown, or use an external secrets tool in real setups.

### Checkpoint
1. What does `helm template` do, and why is it useful in CI?
2. How does `helm rollback` differ from `kubectl rollout undo`?
3. In Kustomize, what is the difference between a base and an overlay?

### Project outcome
`charts/api` installed and upgraded via Helm, and `dev`/`prod` overlays that render and deploy the whole stack.

### References
- [Helm docs](https://helm.sh/docs/) · [Chart best practices](https://helm.sh/docs/chart_best_practices/)
- [Kustomize in Kubernetes docs](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/) · [Kustomize guides](https://kubectl.docs.kubernetes.io/guides/)

---

# Day 39: Advanced Topics: Service Mesh & Serverless on Kubernetes

**Goal:** See what a service mesh and scale-to-zero serverless add, and when they are worth the complexity.

### Concepts
**Service mesh** (Istio, Linkerd, Cilium service mesh) moves cross-cutting network concerns out of application code:
- **mTLS** between services, identity-based authorization
- Retries, timeouts, circuit breaking, traffic splitting (canaries)
- Uniform metrics, logs, and traces

Costs: more moving parts, resource overhead, and a steeper debugging curve. Rule of thumb: adopt a mesh when you have many services with security or traffic-management needs that libraries and NetworkPolicies cannot meet cleanly. TaskBoard does **not** need one, so run this lab in a separate namespace.

**Serverless on Kubernetes** (Knative Serving) gives request-driven autoscaling, including **scale to zero**, plus revisions and traffic splitting. Trade-off: cold starts when scaling from zero.

### Lab A: Istio sample app with mTLS
Use the official Getting Started guide for the **current** install steps (it is version-specific), then:
```bash
# After installing Istio per the guide:
kubectl create namespace mesh-demo
kubectl label namespace mesh-demo istio-injection=enabled
# From the unpacked Istio release directory:
kubectl apply -n mesh-demo -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl get pods -n mesh-demo          # READY shows 2/2: app + Envoy sidecar
istioctl analyze -n mesh-demo
```
Require mTLS and prove it:
```yaml
apiVersion: security.istio.io/v1       # use v1beta1 on older Istio releases
kind: PeerAuthentication
metadata:
  name: default
  namespace: mesh-demo
spec:
  mtls:
    mode: STRICT
```
```bash
kubectl apply -f peer-auth.yaml
# A Pod outside the mesh can no longer talk to a meshed service:
kubectl run curl -n default --rm -it --image=curlimages/curl --restart=Never -- \
  curl -sS -m 5 http://productpage.mesh-demo:9080/productpage
```
Keep the mesh out of `taskboard`: its `restricted` Pod Security level blocks the sidecar-injection init container unless you use Istio's CNI or ambient mode. That conflict is a good real-world lesson about how controls interact.

### Lab B: Knative scale-to-zero
Use a **separate** cluster so it cannot interfere with TaskBoard:
```bash
# Install the kn CLI and the quickstart plugin per the Knative docs, then:
kn quickstart kind                     # creates its own kind cluster named "knative"
kn service create hello --image <sample-image-from-Knative-docs> --env TARGET="Phase 2"
kn service list
curl <URL printed by kn service list>
kubectl get pods -w                    # idle Pods terminate after about a minute: scale to zero
curl <URL>                             # a new Pod starts on demand (cold start)

# Return to your main cluster and clean up
kubectl config use-context kind-taskboard
kind delete cluster --name knative
```

**Stretch:** split Knative traffic 50/50 between two revisions with `kn service update hello --traffic ...`.

### Checkpoint
1. What does the sidecar proxy do that your application code no longer has to?
2. Name one situation where a mesh is overkill.
3. What is a cold start, and how does it affect user-facing latency?

### Project outcome
`notes/day39.md` with a short decision memo: "Would TaskBoard benefit from a mesh or Knative? Why / why not?" plus screenshots or output of both labs.

### References
- [Istio Getting Started](https://istio.io/latest/docs/setup/getting-started/) · [Istio mTLS](https://istio.io/latest/docs/tasks/security/authentication/mtls-migration/)
- [Linkerd](https://linkerd.io/2/overview/)
- [Knative quickstart](https://knative.dev/docs/getting-started/quickstart-install/)
- [CNCF Cloud Native Landscape](https://landscape.cncf.io/)

---

# Day 40: Capstone & Review

**Goal:** Prove the whole system is reproducible from nothing, document it, and plan next steps.

### Hands-on 1: automate everything
`scripts/smoke-test.sh` (then `chmod +x`)
```bash
#!/usr/bin/env bash
set -euo pipefail
NS=${NS:-taskboard}
HOST=${HOST_HEADER:-taskboard.localhost}

ENVOY_SVC=$(kubectl get svc -n envoy-gateway-system \
  --selector=gateway.envoyproxy.io/owning-gateway-namespace=$NS,gateway.envoyproxy.io/owning-gateway-name=taskboard-gw \
  -o jsonpath='{.items[0].metadata.name}')
kubectl -n envoy-gateway-system port-forward "svc/$ENVOY_SVC" 8888:80 >/dev/null 2>&1 &
PF=$!
trap 'kill $PF 2>/dev/null || true' EXIT

for _ in $(seq 1 30); do
  curl -fsS -H "Host: $HOST" localhost:8888/healthz >/dev/null 2>&1 && break || sleep 1
done

curl -fsS -H "Host: $HOST" localhost:8888/healthz
curl -fsS -H "Host: $HOST" localhost:8888/readyz
curl -fsS -H "Host: $HOST" localhost:8888/version
curl -fsS -H "Host: $HOST" -H 'Content-Type: application/json' \
  -d '{"title":"smoke test"}' localhost:8888/tasks
curl -fsS -H "Host: $HOST" localhost:8888/tasks | grep -q "smoke test"
echo; echo "SMOKE TEST PASSED"
```

`Makefile` (recipe lines **must start with a TAB**)
```make
CLUSTER ?= taskboard
VERSION ?= 1.1.0
NS      ?= taskboard

.PHONY: all cluster build load secret deploy envoy gateway metrics test clean

all: cluster build load secret deploy envoy gateway metrics test

cluster:
	kind get clusters | grep -qx $(CLUSTER) || kind create cluster --name $(CLUSTER) --config kind-config.yaml

build:
	docker build --build-arg APP_VERSION=$(VERSION) -t taskboard-api:$(VERSION) ./api

load:
	kind load docker-image taskboard-api:$(VERSION) --name $(CLUSTER)

secret:
	kubectl apply -f k8s/base/namespace.yaml
	kubectl -n $(NS) get secret db-credentials >/dev/null 2>&1 || \
	  kubectl -n $(NS) create secret generic db-credentials \
	    --from-literal=POSTGRES_USER=taskboard \
	    --from-literal=POSTGRES_PASSWORD=$$(openssl rand -hex 16)

deploy:
	kubectl apply -k k8s/base
	kubectl -n $(NS) rollout status statefulset/postgres --timeout=180s
	kubectl -n $(NS) rollout status deployment/api --timeout=180s

envoy:
	@test -n "$(ENVOY_GATEWAY_VERSION)" || (echo "Set ENVOY_GATEWAY_VERSION (see Envoy Gateway quickstart)"; exit 1)
	helm upgrade --install eg oci://docker.io/envoyproxy/gateway-helm \
	  --version $(ENVOY_GATEWAY_VERSION) -n envoy-gateway-system --create-namespace
	kubectl wait --timeout=5m -n envoy-gateway-system deployment/envoy-gateway --for=condition=Available

gateway:
	kubectl apply -f k8s/gateway.yaml

metrics:
	kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
	kubectl patch -n kube-system deployment metrics-server --type=json \
	  -p '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]' || true

test:
	./scripts/smoke-test.sh

clean:
	kind delete cluster --name $(CLUSTER)
```

### Hands-on 2: rebuild from zero
```bash
make clean
time make ENVOY_GATEWAY_VERSION=<version-from-envoy-quickstart> all
```
Success looks like `SMOKE TEST PASSED` at the end. Goal: under 15 minutes on a normal laptop with images already pulled.

### Hands-on 3: break it, then fix it (game day)
Pick three and write what you observed and how you recovered in `notes/day40-gameday.md`:
1. Delete a random `api` Pod during a request loop.
2. `kubectl delete pod postgres-0` and verify data survives.
3. Push a bad image tag and recover with `rollout undo`.
4. Scale the node load with the Day 36 load test and watch the HPA.
5. Remove a NetworkPolicy allow-rule and diagnose the resulting failure.

### Hands-on 4: finish the repository
Your project `README.md` should contain: a one-paragraph overview, the architecture diagram, prerequisites, quick start (`make ...`), a section per topic (containers, Kubernetes objects, security, packaging), troubleshooting tips, and "what I would do next".

### Project outcome: final acceptance criteria

| # | Criterion | Day |
|---|---|---|
| 1 | Multi-stage, non-root image under a sensible size; no HIGH/CRITICAL findings you could fix | 27, 37 |
| 2 | `docker compose up` runs the stack locally | 28 |
| 3 | Image pushed to a registry with version and SHA tags | 29 |
| 4 | API runs as a Deployment with 2+ replicas, probes, requests/limits | 32, 36 |
| 5 | Zero-downtime rolling update and a tested rollback (runbook written) | 33 |
| 6 | Traffic enters through a Gateway API `Gateway` + `HTTPRoute` | 34 |
| 7 | Secrets and ConfigMaps externalized; Postgres on a PVC-backed StatefulSet | 35 |
| 8 | HPA scales the API under load | 36 |
| 9 | Namespace enforces `restricted`; RBAC identity is read-only; default-deny ingress with explicit allows | 37 |
| 10 | Helm chart for the API and Kustomize overlays render and deploy | 38 |
| 11 | `make all` rebuilds everything and the smoke test passes | 40 |

### Self-review
Answer in `notes/day40-review.md`:
1. Draw the path of a request from `curl` to Postgres and back, naming every Kubernetes object it touches.
2. Which three decisions most improved reliability? Which three most improved security?
3. What would break first in production, and how would you find out? (Hint: observability is the missing piece.)

### Stretch goals
- **CI:** GitHub Actions to build, Trivy-scan, and push on every commit; deploy with `helm upgrade`.
- **GitOps:** let Argo CD or Flux deploy from the repo.
- **TLS:** cert-manager with an HTTPS listener on the Gateway.
- **Observability:** Prometheus + Grafana, plus structured JSON logs.
- **Resilience:** PodDisruptionBudget and topology spread constraints.

### What's next
- **Phase 3:** CI/CD and Infrastructure as Code.
- Consider the [CKAD certification](https://training.linuxfoundation.org/certification/certified-kubernetes-application-developer-ckad/); your capstone already covers most of its hands-on skills.

---

# Appendix A: Troubleshooting Cheat Sheet

| Symptom | Likely cause | First commands |
|---|---|---|
| `ImagePullBackOff` / `ErrImagePull` | Wrong tag, private registry, image not loaded into `kind` | `kubectl describe pod`, `kind load docker-image ...` |
| `CrashLoopBackOff` | App exits on start (config, DB unreachable) | `kubectl logs --previous`, check env and Secrets |
| `Pending` | No node fits (resources), PVC unbound | `kubectl describe pod` (Events), `kubectl get pvc` |
| `CreateContainerConfigError` | Missing Secret/ConfigMap key | `kubectl describe pod`, `kubectl get secret,cm` |
| Pod rejected on create | Pod Security `restricted` violation | `kubectl describe rs/sts`, `kubectl get events` |
| Service has no endpoints | Selector mismatch or Pods not Ready | `kubectl get endpointslices`, compare labels |
| `OOMKilled` (exit 137) | Memory limit too low or leak | `kubectl describe pod`, `kubectl top pod` |
| HPA shows `<unknown>` | metrics-server missing or no CPU requests | `kubectl top pods`, check `resources.requests` |
| Gateway route returns 404/503 | Host header mismatch, wrong backend, NetworkPolicy | `kubectl describe httproute`, check `Host` header |

# Appendix B: Command Quick Reference

```bash
# Docker
docker build -t name:tag .    docker run -d -p H:C --name n image    docker logs -f n
docker exec -it n sh          docker compose up -d --build           docker system prune

# Kubernetes
kubectl get pods,svc,deploy,sts,pvc,hpa -n taskboard
kubectl describe <kind> <name>        kubectl logs -f <pod> [-c container] [--previous]
kubectl exec -it <pod> -- sh          kubectl port-forward svc/<svc> LOCAL:REMOTE
kubectl rollout status|history|undo deployment/<name>
kubectl top pods                      kubectl auth can-i <verb> <resource> --as=<user>
kubectl get events --sort-by=.lastTimestamp
kubectl explain <resource>.<field>    kubectl diff -f file.yaml

# Helm / Kustomize
helm lint|template|install|upgrade|rollback|history|list
kubectl kustomize <dir>               kubectl apply -k <dir>
```

# Appendix C: Further Reading & Practice

- [Kubernetes documentation](https://kubernetes.io/docs/home/) and [tutorials](https://kubernetes.io/docs/tutorials/)
- [Docker documentation](https://docs.docker.com/) and [Play with Docker](https://labs.play-with-docker.com/)
- [Killercoda](https://killercoda.com/): free interactive Kubernetes scenarios
- [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way): learn the internals by building a cluster manually
- *Kubernetes: Up and Running* (Burns, Beda, Hightower, Evenson) and *Docker Deep Dive* (Nigel Poulton)
- [CNCF Landscape](https://landscape.cncf.io/)
