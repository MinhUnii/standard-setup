---
name: write-dockerfile
description: Generates, refactors, and optimizes secure, production-ready, multi-stage Dockerfiles across various programming languages and runtimes. Use when asked to write a Dockerfile, generate container build instructions, optimize layer caching, or containerize an application codebase.
---

# Write Dockerfile

This skill provides step-by-step instructions and best practices for authoring secure, minimal, reproducible, and high-performance `Dockerfile` configurations tailored to any application stack.

## Standard Operating Procedure (SOP)

1. **Detect Stack & Requirements:**
   - Identify the programming language, framework, runtime version, package manager, and entrypoint (e.g., Node.js / pnpm, Python / uv, Go, Rust, Java / Maven, .NET).
   - Determine target environment (production, testing, CI/CD).

2. **Select Base Images:**
   - Choose official, minimal base images (e.g., `alpine`, `slim`, or Google Container Tools `distroless`).
   - Pin specific version tags (e.g., `node:20-alpine`, `python:3.11-slim`, `golang:1.22-alpine`) — **never use `latest`**.

3. **Design Multi-Stage Build Architecture:**
   - **Dependencies / Build Stage (`builder`):** Install build toolchains (compilers, build headers), download dependencies, compile static binaries, or build frontend assets.
   - **Production Runtime Stage (`runner`):** Start from a clean, lightweight image. Copy *only* compiled artifacts and production dependencies from the build stage. Discard compilers, caches, and dev tooling.

4. **Maximize Layer Caching:**
   - Order instructions from least frequently changed to most frequently changed.
   - Copy dependency manifests (e.g., `package.json`, `package-lock.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`) and install dependencies *before* copying application source code.
   - Combine related `RUN` commands with `&&` and remove package manager caches in the same layer (e.g., `apt-get clean && rm -rf /var/lib/apt/lists/*`).

5. **Enforce Non-Root Security:**
   - Create a dedicated non-privileged user and group (e.g., `appuser` with UID/GID 10001).
   - Set proper ownership on the application directory (`chown -R appuser:appuser /app`).
   - Switch to the user with `USER appuser` before the runtime `ENTRYPOINT` or `CMD`.

6. **Define Runtime Interface & Signals:**
   - Specify `WORKDIR /app`.
   - Expose required network ports with `EXPOSE <port>`.
   - Use the **exec form** (JSON array syntax) for `ENTRYPOINT` and `CMD` (e.g., `CMD ["node", "dist/index.js"]`) so Unix signals (like `SIGTERM`, `SIGINT`) propagate properly to the application process.
   - Provide a `HEALTHCHECK` instruction when appropriate.

7. **Pair with `.dockerignore`:**
   - Always ensure a corresponding `.dockerignore` is generated or verified to prevent leaking local artifacts, secrets, logs, `.git`, or `node_modules` into the build context.

---

## Decision Tree

### By Language / Stack Archetype

* **Compiled Languages (Go, Rust, C++):**
  * **Build Stage:** Use official compiler image (`golang:X-alpine`, `rust:X-slim`). Compile statically linked binaries (`CGO_ENABLED=0` for Go).
  * **Runtime Stage:** Use `scratch` or `alpine` or `gcr.io/distroless/static`. Copy only the compiled binary and root CA certificates (`ca-certificates`).
* **Interpreted / Dynamic Languages (Node.js, Bun, Python, Ruby):**
  * **Build Stage:** Install dev dependencies, compile TypeScript / native addons / virtual environments.
  * **Runtime Stage:** Use `-slim` or `-alpine`. Copy only production modules or virtual environment (`/opt/venv`).
  * **Next.js / SSR Frameworks:** Use Next.js `standalone` output mode to bundle minimal server runtime.
* **JVM Languages (Java, Kotlin, Scala):**
  * **Build Stage:** Use JDK image (`eclipse-temurin:X-jdk-alpine`) with Maven or Gradle wrapper.
  * **Runtime Stage:** Use minimal JRE (`eclipse-temurin:X-jre-alpine`) or generate custom runtime with `jlink`.

---

## Language Reference Templates

### Node.js (TypeScript / Multi-Stage)
```dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Stage 2: Builder
FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build && npm prune --production

# Stage 3: Runner
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup -S -g 10001 appgroup && adduser -S -u 10001 -G appgroup appuser
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup /app/package.json ./package.json
USER appuser
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Python (uv / Multi-Stage)
```dockerfile
# Stage 1: Builder
FROM python:3.11-slim AS builder
WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1
RUN apt-get update && apt-get install -y --no-install-recommends build-essential \
    && rm -rf /var/lib/apt/lists/*
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Stage 2: Runner
FROM python:3.11-slim AS runner
WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/opt/venv/bin:$PATH"
RUN groupadd -g 10001 appuser && useradd -u 10001 -g appuser -s /bin/sh appuser
COPY --from=builder --chown=appuser:appuser /opt/venv /opt/venv
COPY --chown=appuser:appuser . .
USER appuser
EXPOSE 8000
CMD ["python", "main.py"]
```

### Go (Minimal Static Binary)
```dockerfile
# Stage 1: Builder
FROM golang:1.22-alpine AS builder
WORKDIR /app
RUN apk add --no-cache ca-certificates git
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o /app/server .

# Stage 2: Runner
FROM scratch
WORKDIR /app
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /app/server /app/server
USER 10001:10001
EXPOSE 8080
ENTRYPOINT ["/app/server"]
```

---

## Checklists & Best Practices

* **Layer Optimization:**
  * Clean package manager caches within the same `RUN` step.
  * Use `--no-install-recommends` (Debian/Ubuntu) or `--no-cache` (Alpine).
  * Consolidate commands using `&& \`.
* **Security & Hardening:**
  * Never run containers as root in production.
  * Do not embed API keys, secrets, or `.env` files into image layers.
  * Use specific, pinned base image tags instead of floating tags (`:latest`).
* **Execution & Signal Handling:**
  * Always use exec form `CMD ["executable", "param"]` rather than shell form `CMD executable param`.
  * Set `WORKDIR` explicitly instead of using relative paths or `/`.

---

## Review Checklist

Before presenting the `Dockerfile` to the user, verify:
- [ ] Are explicit base image tags pinned (no `:latest`)?
- [ ] Is a multi-stage build implemented to keep the final image minimal?
- [ ] Are dependency manifests copied and installed before copying source code for optimal caching?
- [ ] Is a non-root user (`USER <user>`) configured with appropriate directory permissions?
- [ ] Are temporary build dependencies and caches cleaned in the layer they are created?
- [ ] Are `CMD` and `ENTRYPOINT` defined in JSON array (exec) format?
- [ ] Is a `.dockerignore` file provided or checked?
