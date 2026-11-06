# 🐳 Docker for Backend & Application Modules — A Student Lab Notes

**Goal:** understand containers clearly, then use them to build, run, and debug a real multi-module app (API + database + cache) efficiently.

**How to use this notebook**
1. Read the short theory cell, then run the code cell below it.
2. Run every command block in a terminal. Blocks labelled **Create file** are files to save inside the `docker-lab/` folder (create the folders `docker-lab/` and `docker-lab/api/` first).
3. Do the **✏️ Try it** tasks. Learning happens there, not in reading.
4. Write your own answers under each **📝 My notes** heading.

**Prerequisites:** Docker Desktop or Docker Engine + Compose v2 installed (`docker compose`, not `docker-compose`).

**Contents**
0. Mental model
1. Setup check
2. Everyday CLI
3. Build your own image (Dockerfile)
4. Data: volumes & bind mounts
5. Networking between containers
6. Config & secrets
7. Docker Compose: multi-module app
8. Development workflow
9. Debugging & troubleshooting
10. Production-minded practices
11. Cleanup
12. Exercises & mini project

## 0. Mental model (read this first)

| Concept | Plain meaning | Analogy |
|---|---|---|
| **Image** | Read-only template: OS files + runtime + your code + start command | A recipe / class |
| **Container** | A running (or stopped) instance of an image, isolated process(es) | A dish / object |
| **Dockerfile** | Text instructions to build an image | The recipe card |
| **Registry** | Server that stores images (Docker Hub, GHCR) | GitHub for images |
| **Volume** | Storage managed by Docker that outlives containers | External hard drive |
| **Network** | Virtual network so containers find each other by name | Private LAN |
| **Compose** | One YAML file describing many containers working together | Orchestra score |

**Container vs VM:** a VM virtualizes hardware and runs a full OS. A container shares the host kernel and isolates only processes, filesystem, and network, so it starts in about a second and is small.

```
Your code ──(docker build)──▶ Image ──(docker run)──▶ Container(s)
                                │
                          docker push/pull
                                ▼
                            Registry
```

**Three rules that prevent 80% of confusion**
1. Containers are **disposable**. Anything written inside the container filesystem is lost when it's removed. Persist data with volumes.
2. `localhost` inside a container means **that container**, not your laptop and not another container. Use service names.
3. **Image = what to run. Container = running it.** Rebuild the image after changing code that is *copied* into it (unless you bind-mount for dev).

### 📝 My notes
_In 3 sentences, explain image vs container to a friend:_

>

## 1. Setup check

```bash
docker --version
docker compose version
docker info --format 'Server: {{.ServerVersion}} | OS: {{.OperatingSystem}} | Containers: {{.Containers}}'
echo "--- hello-world test ---"
docker run --rm hello-world | head -n 5
```

If this fails: start Docker Desktop, or on Linux run `sudo systemctl start docker` and add yourself to the group with `sudo usermod -aG docker $USER` (then log out/in).

## 2. Everyday CLI

Format: `docker <object> <action>`. Learn these and you can handle most situations.

| Task | Command |
|---|---|
| Download image | `docker pull nginx:1.27` |
| Run in background, name it, map port | `docker run -d --name web -p 8080:80 nginx:1.27` |
| List running / all | `docker ps` / `docker ps -a` |
| Logs (follow) | `docker logs -f web` |
| Shell inside | `docker exec -it web sh` |
| Stop / start / restart | `docker stop web` / `docker start web` / `docker restart web` |
| Remove container | `docker rm -f web` |
| List / remove images | `docker images` / `docker rmi nginx:1.27` |
| Live resource use | `docker stats --no-stream` |
| Inspect details (JSON) | `docker inspect web` |

**Port mapping `-p HOST:CONTAINER`**: `-p 8080:80` means your machine's port 8080 forwards to port 80 inside the container.

```bash
docker rm -f web 2>/dev/null
docker run -d --name web -p 8080:80 nginx:1.27-alpine
sleep 2
echo "--- running containers ---"
docker ps --filter name=web --format 'table {{.Names}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}'
echo "--- HTTP check ---"
curl -s -o /dev/null -w "HTTP %{http_code}\n" http://localhost:8080
echo "--- last logs ---"
docker logs --tail 3 web
echo "--- run a command inside ---"
docker exec web nginx -v
docker rm -f web
```

**✏️ Try it:** run the same container on port `9090`. Then use `docker exec -it web sh` in a terminal and `ls /usr/share/nginx/html`. What do you see?

## 3. Build your own image (Dockerfile)

We create a small project folder `docker-lab/` with a **FastAPI backend** that talks to **PostgreSQL** and **Redis**. First, the app code.

### Key Dockerfile instructions
| Instruction | Purpose |
|---|---|
| `FROM` | Base image to start from |
| `WORKDIR` | Set working directory inside image |
| `COPY` | Copy files from your folder into the image |
| `RUN` | Execute a command **at build time** (install packages) |
| `ENV` | Set environment variable |
| `EXPOSE` | Document the port (does not publish it) |
| `USER` | Run as non-root user |
| `CMD` | Default command **at run time** |
| `ENTRYPOINT` | Fixed executable; `CMD` becomes its arguments |

### Layer caching (the #1 speed trick)
Each instruction creates a cached layer. If a layer changes, all layers **after** it rebuild. So copy rarely-changing files (`requirements.txt`) and install dependencies **before** copying frequently-changing source code.

```bash
mkdir -p docker-lab/api
echo "Created $(pwd)/docker-lab"
```

**Create file:** `docker-lab/api/main.py`

```python
import os
import time
from contextlib import asynccontextmanager

import psycopg2
import redis
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://app:secret@db:5432/appdb")
REDIS_URL = os.getenv("REDIS_URL", "redis://cache:6379/0")

cache = redis.Redis.from_url(REDIS_URL, decode_responses=True)


def get_conn():
    return psycopg2.connect(DATABASE_URL)


def init_db(retries: int = 10):
    # Database may still be starting: retry instead of crashing.
    for attempt in range(1, retries + 1):
        try:
            with get_conn() as conn, conn.cursor() as cur:
                cur.execute(
                    "CREATE TABLE IF NOT EXISTS items ("
                    "id SERIAL PRIMARY KEY, name TEXT NOT NULL)"
                )
            return
        except psycopg2.OperationalError:
            print(f"DB not ready (attempt {attempt}/{retries})", flush=True)
            time.sleep(2)
    raise RuntimeError("Database never became ready")


@asynccontextmanager
async def lifespan(app: FastAPI):
    init_db()
    yield


app = FastAPI(title="Docker Lab API", lifespan=lifespan)


class Item(BaseModel):
    name: str


@app.get("/health")
def health():
    return {"status": "ok"}


@app.get("/visits")
def visits():
    return {"visits": cache.incr("visits")}


@app.post("/items", status_code=201)
def create_item(item: Item):
    with get_conn() as conn, conn.cursor() as cur:
        cur.execute("INSERT INTO items (name) VALUES (%s) RETURNING id", (item.name,))
        new_id = cur.fetchone()[0]
    return {"id": new_id, "name": item.name}


@app.get("/items")
def list_items():
    with get_conn() as conn, conn.cursor() as cur:
        cur.execute("SELECT id, name FROM items ORDER BY id")
        return [{"id": r[0], "name": r[1]} for r in cur.fetchall()]


@app.get("/items/{item_id}")
def get_item(item_id: int):
    with get_conn() as conn, conn.cursor() as cur:
        cur.execute("SELECT id, name FROM items WHERE id = %s", (item_id,))
        row = cur.fetchone()
    if not row:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"id": row[0], "name": row[1]}
```

**Create file:** `docker-lab/api/requirements.txt`

```text
fastapi==0.115.0
uvicorn[standard]==0.30.6
psycopg2-binary==2.9.9
redis==5.0.8
```

**Create file:** `docker-lab/api/Dockerfile`

```dockerfile
# 1. Small, pinned base image (never use :latest in real projects)
FROM python:3.12-slim

# 2. Sensible Python defaults for containers
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# 3. Dependencies first => cached until requirements.txt changes
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 4. Source code last => changes often, only this layer rebuilds
COPY . .

# 5. Don't run as root
RUN useradd --create-home appuser
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=15s --timeout=3s --start-period=10s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Create file:** `docker-lab/api/.dockerignore`

```text
__pycache__/
*.pyc
.git
.env
.venv/
*.md
Dockerfile
```

`.dockerignore` keeps junk and **secrets** (like `.env`) out of the image and speeds up builds.

```bash
cd docker-lab/api
docker build -t docker-lab-api:1.0 .
echo "--- image size ---"
docker images docker-lab-api
echo "--- layers ---"
docker history docker-lab-api:1.0 --format 'table {{.CreatedBy}}\t{{.Size}}' | head -n 12
```

**✏️ Try it (feel the cache):**
1. Run the build cell again → everything says `CACHED`.
2. Add a comment to `main.py` and rebuild → only the `COPY . .` layer and later ones rebuild; pip install stays cached.
3. Add a package to `requirements.txt` and rebuild → pip install runs again. Why?

**Tagging:** `docker tag docker-lab-api:1.0 myuser/docker-lab-api:1.0` then `docker push myuser/docker-lab-api:1.0`.

## 4. Data: volumes and bind mounts

| Type | Syntax | Use for |
|---|---|---|
| **Named volume** | `-v pgdata:/var/lib/postgresql/data` | Databases, anything that must persist; Docker manages location |
| **Bind mount** | `-v $(pwd)/api:/app` | Live code editing in development |
| **tmpfs** | `--tmpfs /tmp` | Temporary in-memory data |

Experiment: prove that data dies with a container unless stored in a volume.

```bash
docker rm -f t1 t2 2>/dev/null; docker volume rm demo-vol 2>/dev/null

echo "### Without volume"
docker run --name t1 alpine sh -c 'echo hello > /data.txt && cat /data.txt'
docker rm t1
docker run --rm alpine sh -c 'cat /data.txt 2>&1 || true'   # file is gone

echo; echo "### With named volume"
docker run --rm -v demo-vol:/data alpine sh -c 'echo persistent > /data/file.txt'
docker run --rm -v demo-vol:/data alpine cat /data/file.txt  # still there

docker volume ls | grep demo-vol
docker volume rm demo-vol
```

## 5. Networking between containers

- On a **user-defined network**, containers resolve each other **by name** (built-in DNS).
- The default `bridge` network does **not** give you name resolution. Always create your own (Compose does this automatically).
- Only **publish ports (`-p`)** for things you need to reach from your host. The database usually should *not* be published in production.

```bash
docker rm -f cache-demo 2>/dev/null; docker network rm lab-net 2>/dev/null
docker network create lab-net
docker run -d --name cache-demo --network lab-net redis:7-alpine
sleep 2
echo "--- another container reaches it by NAME ---"
docker run --rm --network lab-net redis:7-alpine redis-cli -h cache-demo ping
docker rm -f cache-demo
docker network rm lab-net
```

**✏️ Try it:** repeat without `--network lab-net`. What error do you get, and why?

## 6. Configuration & secrets

Follow the **12-factor** idea: the same image runs everywhere; **environment variables** change behaviour.

```bash
docker run -e DATABASE_URL=postgresql://... myimage        # single variable
docker run --env-file .env myimage                         # many variables
```

**Never**
- bake passwords into the Dockerfile (`ENV PASSWORD=...` is visible in `docker history`)
- commit `.env` to Git (add it to `.gitignore`)

**For real deployments** use orchestrator secrets (Docker/Swarm secrets, Kubernetes Secrets, cloud secret managers).

## 7. Docker Compose: running the whole app

Compose describes **services**, **networks**, and **volumes** in one file. One command starts everything in the right order.

Our stack:
```
 Browser/curl ──▶ api (FastAPI :8000) ──▶ db (Postgres)
                                    └───▶ cache (Redis)
```

**Create file:** `docker-lab/.env`

```bash
POSTGRES_USER=app
POSTGRES_PASSWORD=secret
POSTGRES_DB=appdb
```

**Create file:** `docker-lab/docker-compose.yml`

```yaml
services:
  api:
    build: ./api
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
      REDIS_URL: redis://cache:6379/0
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    env_file: .env
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 5s
      timeout: 3s
      retries: 10
    # no "ports:" => reachable only by other containers (safer)

  cache:
    image: redis:7-alpine
    volumes:
      - redisdata:/data

volumes:
  pgdata:
  redisdata:
```

**Compose commands**
| Command | What it does |
|---|---|
| `docker compose up -d --build` | Build (if needed) and start everything in background |
| `docker compose ps` | Status of services |
| `docker compose logs -f api` | Follow one service's logs |
| `docker compose exec api sh` | Shell into running service |
| `docker compose restart api` | Restart one service |
| `docker compose down` | Stop and remove containers + network (keeps volumes) |
| `docker compose down -v` | Same **and deletes volumes** (data loss!) |

```bash
cd docker-lab
docker compose up -d --build
echo "--- waiting for health ---"
sleep 12
docker compose ps
```

```bash
echo "--- health ---";   curl -s localhost:8000/health; echo
echo "--- visits (Redis) ---"; curl -s localhost:8000/visits; echo; curl -s localhost:8000/visits; echo
echo "--- create items (Postgres) ---"
curl -s -X POST localhost:8000/items -H 'Content-Type: application/json' -d '{"name":"notebook"}'; echo
curl -s -X POST localhost:8000/items -H 'Content-Type: application/json' -d '{"name":"laptop"}'; echo
echo "--- list ---"; curl -s localhost:8000/items; echo
echo "--- 404 ---"; curl -s -i localhost:8000/items/999 | head -n 1
echo; echo "Interactive docs: http://localhost:8000/docs"
```

### Prove persistence
Remove the containers (not volumes) and bring them back: the items survive.

```bash
cd docker-lab
docker compose down
docker compose up -d
sleep 12
curl -s localhost:8000/items; echo
```

### Look inside the database directly

```bash
cd docker-lab
docker compose exec -T db psql -U app -d appdb -c "SELECT * FROM items;"
docker compose exec -T cache redis-cli GET visits
```

## 8. Efficient development workflow

Rebuilding the image after every code edit is slow. In development, **bind-mount your source and use auto-reload**. Compose automatically merges `docker-compose.override.yml` on top of `docker-compose.yml`, so dev behaviour stays out of your main file.

**Create file:** `docker-lab/docker-compose.override.yml`

```yaml
services:
  api:
    command: uvicorn main:app --host 0.0.0.0 --port 8000 --reload
    volumes:
      - ./api:/app
```

```bash
cd docker-lab
docker compose up -d
sleep 5
docker compose logs --tail 5 api
```

**✏️ Try it:** edit `docker-lab/api/main.py` (e.g. change the `/health` response to `{"status": "ok", "v": 2}`), save, then `curl localhost:8000/health`. No rebuild needed!

**Daily dev loop**
1. `docker compose up -d` once in the morning
2. Edit code → auto-reload
3. `docker compose logs -f api` in a side terminal
4. Changed `requirements.txt` or `Dockerfile`? → `docker compose up -d --build api`
5. Done? `docker compose stop` (keeps everything) or `down` (removes containers)

**Run one-off commands** (migrations, tests, scripts) in a throwaway container:
```bash
docker compose run --rm api python -c "print('hello from a one-off container')"
```

## 9. Debugging & troubleshooting

### Systematic approach
1. **Is it running?** `docker compose ps` (look for `Exited`, `Restarting`, `unhealthy`)
2. **Why did it stop?** `docker compose logs --tail 50 <service>`
3. **Can I look inside?** `docker compose exec <service> sh` (if crashed: `docker compose run --rm --entrypoint sh <service>`)
4. **Is the network right?** from inside api: `getent hosts db` or `python -c "import socket;print(socket.gethostbyname('db'))"`
5. **Is config right?** `docker compose config` (shows the final merged YAML) and `docker compose exec api env`

### Common problems
| Symptom | Likely cause | Fix |
|---|---|---|
| `port is already allocated` | Host port in use | Change left side of `-p`, or stop the other process |
| `connection refused` to `localhost:5432` from API | Using `localhost` inside container | Use service name `db` |
| Container exits immediately | Main process finished/crashed | Read `docker logs`; ensure `CMD` runs a foreground process |
| App can't reach DB on first start | DB not ready yet | `healthcheck` + `depends_on: condition: service_healthy`, plus retry logic |
| Code changes not showing | Code baked in image, no bind mount | Rebuild, or use dev override with volumes |
| `permission denied` on mounted files | UID mismatch | Match user IDs or `chown`; avoid root-owned files |
| Build is slow every time | Bad layer order / no `.dockerignore` | Install deps before `COPY . .` |
| Disk filling up | Old images/volumes/build cache | `docker system df`, then prune (section 11) |
| Env change not applied | Container not recreated | `docker compose up -d` (recreates) |

```bash
cd docker-lab
echo "--- merged config (first lines) ---"; docker compose config | head -n 25
echo "--- DNS from inside api ---"
docker compose exec -T api python -c "import socket; print('db ->', socket.gethostbyname('db')); print('cache ->', socket.gethostbyname('cache'))"
echo "--- who am I inside api? ---"
docker compose exec -T api id
echo "--- resource usage ---"
docker stats --no-stream --format 'table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}'
```

## 10. Production-minded practices

**Images**
- Pin versions (`python:3.12-slim`, not `latest`).
- Prefer `-slim` / `-alpine` bases. (Alpine uses musl, which can break some Python wheels. Slim is the safer default.)
- One process per container. Keep containers stateless.
- Run as a **non-root** user.
- Use `.dockerignore`; never put secrets in images.
- Use **multi-stage builds** to leave compilers and build tools out of the final image.

**Runtime**
- Add `healthcheck` and `restart: unless-stopped`.
- Limit resources: `deploy.resources.limits` or `mem_limit` / `cpus` in Compose.
- Log to stdout/stderr (Docker collects it), not to files.
- Don't publish database ports publicly.

**Multi-stage example** (compile heavy dependencies in one stage, ship only the result):
```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY . .
RUN useradd -m appuser
USER appuser
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Other modules, same pattern.** To add a Node/React frontend, an Nginx reverse proxy, or a background worker, add another service to Compose:
```yaml
  worker:
    build: ./api
    command: python worker.py     # same image, different command
    depends_on: [db, cache]

  frontend:
    build: ./frontend
    ports: ["3000:80"]
    depends_on: [api]
```
Each module gets its own folder, Dockerfile, and service entry; they talk to each other by service name.

## 11. Cleanup

| Goal | Command |
|---|---|
| Stop this project, keep data | `docker compose down` |
| Stop and **delete data** | `docker compose down -v` |
| Remove stopped containers | `docker container prune` |
| Remove unused images | `docker image prune -a` |
| Remove build cache | `docker builder prune` |
| Everything unused (careful!) | `docker system prune -a --volumes` |
| Check disk usage | `docker system df` |

```bash
docker system df
# Uncomment to fully clean this lab (deletes the lab database data!):
# cd docker-lab && docker compose down -v --rmi local
```

## 12. Exercises & mini project

### Level 1: Basics
1. Run `nginx` on port 9000 and `redis` with no published ports. Show both with `docker ps`.
2. Create a named volume, write a file from one container, read it from another.
3. Find the IP address of a container with `docker inspect`.

### Level 2: Images
4. Add a `/time` endpoint to the API that returns the current time. Rebuild and time the build. Then change only a comment and rebuild again. Explain the difference.
5. Convert the Dockerfile to a multi-stage build. Compare image sizes with `docker images`.
6. Add `tests/test_api.py` and run it with `docker compose run --rm api pytest`.

### Level 3: Compose & modules
7. Add an **Nginx** service that proxies `localhost:8080` → `api:8000`, and remove the API's published port.
8. Add a **worker** service that reads jobs from a Redis list.
9. Add `mem_limit: 256m` to the API and verify with `docker stats`.
10. Create a `docker-compose.prod.yml` (no bind mounts, no reload) and start it with `docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d`.

### Capstone checklist
- [ ] Each module has its own Dockerfile and `.dockerignore`
- [ ] `docker compose up` starts the entire app from a clean clone
- [ ] Data survives `docker compose down` / `up`
- [ ] No secrets committed; `.env.example` provided
- [ ] Health checks on API and DB
- [ ] README with run, test, and debug commands

### One-page cheat sheet
```bash
docker build -t name:tag .          docker run -d --name n -p H:C img
docker ps -a                        docker logs -f n
docker exec -it n sh                docker stop|start|rm n
docker compose up -d --build        docker compose down [-v]
docker compose logs -f svc          docker compose exec svc sh
docker compose config               docker system df / prune
```

### 📝 Self-assessment (answer without looking)
1. Why copy `requirements.txt` before the rest of the code?
2. Why does `localhost:5432` fail from the API container?
3. What is the difference between `down` and `down -v`?
4. When do you use a bind mount versus a named volume?
5. Why run as a non-root user?
