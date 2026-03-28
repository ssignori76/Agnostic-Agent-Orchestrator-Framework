# Docker Rules

> **Purpose:** Container-first development strategy, Dockerfile patterns, Docker Compose
> best practices, and image management conventions for all AAOF projects.
>
> **Scope:** All projects targeting Docker Compose or Kubernetes. These rules are
> mandatory — ALL development must happen inside containers.

---

## 0. Docker Prerequisite Verification

**Docker is a mandatory prerequisite for AAOF.** The framework cannot operate without it.

### Verification Procedure (STEP 0)

The agent must verify Docker availability at the beginning of every session:

1. **Full Check (first session or `VAR_DOCKER_INSTALLED` is `false`):**
   ```bash
   docker --version          # Verify Docker CLI
   docker compose version    # Verify Docker Compose
   docker info               # Verify Docker daemon is running
   ```

2. **Quick Health Check (subsequent sessions where `VAR_DOCKER_INSTALLED` is `true`):**
   ```bash
   docker info               # Verify daemon is still running
   ```

### Auto-Installation Guide

If Docker is not found, the agent proposes installation based on the detected OS:

| OS | Installation Command |
| :--- | :--- |
| Ubuntu/Debian | `sudo apt-get update && sudo apt-get install -y docker.io docker-compose-plugin` |
| macOS | `brew install --cask docker` (requires Homebrew) or direct download from [Docker Desktop](https://docs.docker.com/desktop/install/mac-install/) |
| Windows | Direct download from [Docker Desktop](https://docs.docker.com/desktop/install/windows-install/) |
| Fedora/RHEL | `sudo dnf install -y docker docker-compose-plugin` |

After installation, verify with:
```bash
docker --version && docker compose version && docker info
```

### Blocking Behavior

- If the user **refuses** installation: the agent MUST stop completely. No step beyond STEP 0 is allowed.
- If Docker daemon is installed but **not running**: attempt `sudo systemctl start docker` (Linux) or instruct the user to start Docker Desktop (macOS/Windows).
- Session variable: `VAR_DOCKER_INSTALLED` tracks the state across sessions.

---

## 2. Container-First Mandate

**ALL development must happen inside Docker containers.** The host system is never used
as a development environment. This ensures:

- Reproducible environments across machines
- No "works on my machine" issues
- Isolation from the host OS
- The agent cannot accidentally modify the host system

### The Dev Container Pattern

For every project, the agent must generate a `Dockerfile.dev`:

1. The `Dockerfile.dev` contains all SDKs, runtimes, build tools, and development
   utilities needed for the project.
2. Source code is mounted as a volume — never copied into the dev container.
3. The dev container is started via `docker-compose.dev.yml`.
4. All build commands, tests, and scripts run **inside** this container.

See `templates/Dockerfile.dev` for the baseline template.

---

## 3. Multi-Stage Dockerfile for Production

Every production `Dockerfile` must use multi-stage builds:

```dockerfile
# Stage 1: Build
FROM <build-image> AS builder
WORKDIR /build
COPY . .
RUN <build-command>

# Stage 2: Runtime
FROM <minimal-runtime-image>
WORKDIR /app
COPY --from=builder /build/<artifact> .
USER nonroot
CMD ["<entrypoint>"]
```

- **Stage 1 (builder):** Use the full SDK image. Run all compilation and packaging here.
- **Stage 2 (runtime):** Use the minimal image (Alpine preferred). Copy only the final
  artifact. No build tools, no source code.
- See `rules/security_rules.md` for image and user requirements.

---

## 4. docker-compose.yml Best Practices

- **One compose file per environment:** `docker-compose.yml` (production/staging),
  `docker-compose.dev.yml` (development override).
- **Service names:** Use lowercase with hyphens. e.g., `api`, `db`, `cache`.
- **Always define `healthcheck`** for every service that other services depend on.
- **Use named volumes** for persistent data (not bind mounts in production).
- **Use `depends_on` with `condition: service_healthy`** for service startup ordering.
- **Resource limits:** Always define `deploy.resources.limits` (CPU and memory).
- **Networks:** Use explicit named networks. Avoid the default bridge network.
- **Env vars:** Use `.env` files with `env_file`. Never hardcode secrets inline.

```yaml
services:
  api:
    image: my-app-api:${APP_VERSION}
    env_file: .env
    networks:
      - backend
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: "0.50"
          memory: 256M
```

---

## 5. Image Naming Conventions

| Format | Example |
| :--- | :--- |
| `<project>-<service>:<version>` | `my-app-api:1.2.0` |
| Development tag | `my-app-api:dev` |
| Latest stable | `my-app-api:latest` |

- Use semantic versioning for production images.
- Never use `latest` as the only tag in production deployments.
- Tag images with the git commit SHA in CI: `my-app-api:abc1234`.

---

## 6. Volume Mounting for Development

In `docker-compose.dev.yml`:

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - ./src:/app/src        # Source code — hot reload
      - /app/node_modules     # Exclude node_modules from mount
    ports:
      - "3000:3000"           # Debug/dev port — not in production
      - "9229:9229"           # Debugger port (Node.js example)
```

---

## 7. Network Isolation

- Define at least two networks: `frontend` (exposed services) and `backend` (internal).
- Databases and caches should only be on the `backend` network.
- Only the reverse proxy / API gateway should be on both networks.

---

## 8. .dockerignore Requirements

Every project must include a `.dockerignore` file:

```
.git
.gitignore
*.md
node_modules/
__pycache__/
*.pyc
.env
.env.*
!.env.example
tests/
docs/
backups/
session/
```

---

## 9. Health Checks

Every service must define a health check either in the Dockerfile or in compose:

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

The agent uses health check results in STEP 5 (Validation) to confirm a successful
deployment.

---

## 10. Final Deliverable

The final deliverable for any AAOF project is **one or more Docker containers** with
the application inside. The output must include:

- `Dockerfile` (production, multi-stage)
- `docker-compose.yml` (production/staging)
- `docker-compose.dev.yml` (development)
- `.env.example` (never the actual `.env`)
- All application source code in `output/`
