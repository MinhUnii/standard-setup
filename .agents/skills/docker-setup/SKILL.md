---
name: docker-setup
description: Sets up and optimizes Dockerfile, docker-compose.yml, and .dockerignore for software projects. Use this skill when the user requests containerizing an application, writing Docker configurations, optimizing image size, or setting up local/development and production environments.
---

# Docker Setup Skill

This skill outlines how to write production-ready Docker configurations, focusing on security, performance, and developer experience (DX). Strictly follow these principles when handling Docker-related requests.

## Decision Tree

When a user requests Docker setup, analyze the context to provide the appropriate solution:
- **If the target is Production (or unspecified):** Create a `Dockerfile` using a multi-stage build architecture, optimize the image size, and apply a non-root user. Always provide a `.dockerignore` file.
- **If the target is Local/Development:** Create the standard `Dockerfile` as above, but include a `docker-compose.yml` file configured with `volumes` for code hot-reloading and overriding the startup command.
- **If optimizing an existing image:** Review the file to apply caching rules, consolidate `RUN` commands, and remove temporary/cache files.

## Core Guidelines

### 1. Multi-stage Build Architecture
Always separate the build and run environments to keep the final image lean.
- **Builder Stage:** Use this stage to install heavy compilation tools (like gcc, make) and download dependencies. Save the output to a dedicated directory (e.g., `/install` or a `virtualenv`).
- **Runner Stage:** Use a lightweight base image (`-slim` or `-alpine`). Only copy the compiled/installed dependencies from the Builder stage. DO NOT bring build tools into this stage.

### 2. Image Size & Performance Optimization
- Chain related `RUN` commands using `&&` to reduce the number of image layers.
- Clean up the package manager cache in the same layer it was used (e.g., `rm -rf /var/lib/apt/lists/*` for apt, or `apk cache clean`).
- For Python, always use the `--no-cache-dir` flag with pip.
- Set optimal runtime environment variables:
  - `PYTHONDONTWRITEBYTECODE=1`
  - `PYTHONUNBUFFERED=1`
- **Mandatory:** Always provide a `.dockerignore` file to exclude `.git`, `__pycache__`, `venv`, and local configuration files (`.env`).

### 3. Security First
- **Never** run the container as the default `root` user in the final Runner stage.
- Always create an unprivileged system user and group (e.g., `appuser`).
- Transfer ownership (`chown`) of the application code directories to this non-root user.
- Switch to this user using the `USER appuser` directive before defining `EXPOSE` and `CMD`.

### 4. Local Developer Experience (DX with Compose)
- Use `docker-compose.yml` to set up the development environment.
- Map the source code using `volumes` (e.g., `./src:/app/src`) to enable instant code updates (hot-reload) without needing to rebuild the image.
- Use the `command` property in compose to override the Dockerfile's default startup command, enabling live-reload tools (e.g., `uvicorn --reload`, `nodemon`).
- Explicitly declare backing services (like Databases, Redis) in the compose file.

## Review Checklist

Before delivering the Docker source code to the user, verify the following:
- [ ] Does the `Dockerfile` implement a multi-stage build (e.g., AS builder and AS runner)?
- [ ] Is there a command to create and use a non-root user (e.g., `adduser` and `USER appuser`)?
- [ ] Are OS and package caches properly cleared (e.g., `--no-cache-dir` or `rm -rf /var/lib/apt/lists/*`)?
- [ ] Is a `.dockerignore` file provided?
- [ ] (If for local dev) Does the `docker-compose.yml` include `volumes` mapped for hot-reloading?