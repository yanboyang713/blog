---
title: "Use Codex to Debug Docker Services"
date: 2026-06-29
tags: ["codex", "docker", "ubuntu", "debugging"]
draft: false
---

## How to use Codex on Ubuntu to debug Docker Compose services {#how-to-use-codex-on-ubuntu-to-debug-docker-compose-services}

When an application runs as several Docker containers, debugging usually means inspecting Compose configuration, container state, logs, health checks, networks, ports, mounts, environment variables, and service dependencies. [Codex CLI]({{< relref "2025-10-09-180435-coding_agent.md#codex-cli" >}}) can automate much of that investigation from an Ubuntu terminal because it can run the same local tools a developer would run manually.

Codex does not need a special Docker plugin for this workflow. In a local CLI session, the practical path is:

```text
Codex CLI
  |
  | runs docker commands
  v
Docker CLI
  |
  | talks through a Unix socket
  v
/var/run/docker.sock
  |
  v
Docker daemon
  |
  v
Containers and Docker Compose services
```

For example, when Codex runs:

```bash
docker compose ps
```

the installed `docker` client contacts the Docker daemon through the normal Docker endpoint. Codex itself is not reimplementing the Docker API; it is executing the local command-line tool inside the permissions available to the Codex session.


### Prerequisites {#prerequisites}

The Ubuntu workstation needs:

-   [Docker]({{< relref "20230216042646-docker.md" >}}) Engine;
-   Docker Compose;
-   Codex CLI;
-   a project directory containing `compose.yaml` or `docker-compose.yml`.

First verify that Docker is running:

```bash
sudo systemctl status docker
```

If it is stopped, enable and start it:

```bash
sudo systemctl enable --now docker
```

Then verify the installed tooling:

```bash
docker version
docker compose version
docker ps
```

If those commands work, the local Docker environment is ready for Codex-assisted debugging.


### Docker daemon access for the current user {#docker-daemon-access-for-the-current-user}

On many Ubuntu systems, Docker initially requires `sudo`:

```bash
sudo docker ps
```

Codex normally runs as the current Linux user. For repeated local debugging, it is convenient for that user to run Docker commands directly:

```bash
sudo usermod -aG docker "$USER"
```

Log out and log back in so the new group membership is applied. For the current terminal only, this may also work:

```bash
newgrp docker
```

Verify the result:

```bash
docker ps
```

Treat Docker group membership as administrative access. A process that can control the Docker daemon can create privileged containers, mount host directories, modify networks, stop services, and potentially reach sensitive host resources. Do not grant Docker access to untrusted users or unattended agents.


### Install and start Codex CLI {#install-and-start-codex-cli}

OpenAI documents a standalone installer for macOS and Linux:

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

The first run prompts for authentication using a ChatGPT account or an API key. After installation, verify the CLI:

```bash
codex --version
```

Start Codex from the project root so it can see the Compose file, Dockerfiles, application code, and local configuration:

```bash
cd ~/projects/my-application
codex
```

A typical Compose project might look like:

```text
my-application/
|-- compose.yaml
|-- backend/
|   |-- Dockerfile
|   `-- src/
|-- frontend/
|   |-- Dockerfile
|   `-- src/
|-- nginx/
|   `-- nginx.conf
`-- .env
```


### Sandbox and approval model {#sandbox-and-approval-model}

Codex uses sandbox and approval controls for local commands. A practical starting point for Docker debugging is the lower-risk local preset:

```bash
codex \
  --sandbox workspace-write \
  --ask-for-approval on-request
```

In this mode, Codex can read files, edit files in the workspace, and run routine local commands inside the sandbox. When a command needs to cross a boundary, such as contacting a blocked local socket, Codex asks for approval.

Inside an interactive Codex session, inspect or change the active mode with:

```text
/permissions
```

For occasional Docker work, approve Docker commands one by one. This keeps privileged operations visible. It is especially appropriate when the host has important containers, persistent databases, or unrelated workloads.

For frequent Docker debugging, use a narrowly scoped permission profile instead of full access. OpenAI documents permission profiles as the newer way to describe filesystem and network access together; they should not be mixed with older `sandbox_mode` settings in the same session.

Example `~/.codex/config.toml` profile:

```toml
default_permissions = "docker-debug"

[permissions.docker-debug]
description = "Edit the current project and access the local Docker daemon."
extends = ":workspace"

[permissions.docker-debug.network]
enabled = true
mode = "limited"

[permissions.docker-debug.network.unix_sockets]
"/var/run/docker.sock" = "allow"
```

The important entry is the explicit Unix-socket allow rule:

```toml
[permissions.docker-debug.network.unix_sockets]
"/var/run/docker.sock" = "allow"
```

This profile keeps normal workspace protections, grants access to the Docker socket, and avoids `danger-full-access`. If endpoint tests must reach a local web service from inside the sandbox, allow only the necessary local targets:

```toml
[permissions.docker-debug.network.domains]
"localhost" = "allow"
"127.0.0.1" = "allow"
```

Avoid these combinations unless the environment is isolated and disposable:

-   `--dangerously-bypass-approvals-and-sandbox`
-   `--sandbox danger-full-access`
-   `approval_policy = "never"` with broad local access
-   broad Docker cleanup commands without an explicit confirmation step


### Rootless Docker {#rootless-docker}

Rootless Docker may not use `/var/run/docker.sock`. Its socket is often under the current user's runtime directory, for example:

```text
/run/user/1000/docker.sock
```

Find the active endpoint:

```bash
docker context inspect \
  --format '{{ (index .Endpoints "docker").Host }}'
```

Example output:

```text
unix:///run/user/1000/docker.sock
```

Remove the `unix://` prefix and allow that absolute path instead:

```toml
[permissions.docker-debug.network.unix_sockets]
"/run/user/1000/docker.sock" = "allow"
```

Also check whether `DOCKER_HOST` overrides the default endpoint:

```bash
echo "$DOCKER_HOST"
```


### A structured debugging prompt {#a-structured-debugging-prompt}

Once Docker access works, give Codex a constrained investigation prompt:

```text
Inspect this Docker Compose project.

1. Read the Compose file and identify all services.
2. Run docker compose config to validate the effective configuration.
3. Run docker compose ps.
4. Identify stopped, unhealthy, or restarting services.
5. Read recent logs for affected services.
6. Inspect exit codes, health checks, ports, mounts,
   environment variables, dependencies, and networks.
7. Explain the likely root cause before changing anything.
8. Make only the minimum necessary changes.
9. Rebuild and restart only the affected services.
10. Verify the fix using service status, logs, health checks,
    and an endpoint request.

Do not delete volumes, prune Docker resources, remove databases,
or recreate unrelated services without asking me first.
```

The final restriction matters. These commands can delete persistent data or disrupt unrelated workloads:

```bash
docker compose down -v
docker system prune -a
docker volume prune
docker rm -f
```

They should require explicit approval in an ordinary debugging session.


### Useful Docker commands for Codex {#useful-docker-commands-for-codex}

Validate the effective Compose configuration:

```bash
docker compose config
```

This catches invalid YAML, missing environment variables, bad service references, invalid volume definitions, port conflicts, and dependency mistakes.

Check service state:

```bash
docker compose ps
docker compose ps --all
```

Look for services in states such as:

```text
Exited
Restarting
Unhealthy
Created
```

Read logs:

```bash
docker compose logs --tail=200
docker compose logs --tail=200 backend
```

Use long-running log follows only when needed:

```bash
docker compose logs -f backend
```

Inspect selected container state instead of dumping full JSON:

```bash
docker inspect \
  --format '{{.State.Status}} {{.State.ExitCode}} {{.State.Error}}' \
  CONTAINER_NAME

docker inspect \
  --format '{{json .State.Health}}' \
  CONTAINER_NAME
```

Run non-interactive checks inside a container:

```bash
docker exec CONTAINER_NAME env
docker exec CONTAINER_NAME ps aux
docker exec CONTAINER_NAME cat /etc/resolv.conf
```

Inspect networks:

```bash
docker network ls
docker network inspect NETWORK_NAME
```

Use this to check whether services are attached to the expected network, whether container DNS names resolve, whether a service is isolated from a dependency, and whether an `external` network is wrong.

Check resource usage:

```bash
docker stats --no-stream
```

Rebuild only the affected service:

```bash
docker compose build backend
docker compose up -d backend
```

Or rebuild and start one affected service in one step:

```bash
docker compose up -d --build backend
```

Avoid rebuilding the whole application unless the problem affects shared images or shared configuration.


### Host-side endpoint verification {#host-side-endpoint-verification}

A running container is not proof that the application works. After startup, test the exposed endpoint from the host:

```bash
curl -v http://localhost:8080/health
```

For HTTPS with a development certificate:

```bash
curl -vk https://localhost:8443/health
```

A complete verification loop should include:

```bash
docker compose ps
docker compose logs --tail=100 SERVICE_NAME
curl -v http://localhost:PORT/health
```

The process may be alive while its HTTP endpoint, database connection, dependency integration, or health check is failing.


### Example: backend restart loop {#example-backend-restart-loop}

Suppose the backend service repeatedly restarts. Ask Codex:

```text
Investigate why the backend service is restarting.
Do not modify files until you explain the root cause.
```

Codex should gather evidence first:

```bash
docker compose ps
docker compose logs --tail=200 backend
docker inspect backend
docker compose config
```

If the logs show:

```text
connection refused: database:5432
```

then inspect the dependency:

```bash
docker compose ps database
docker compose logs --tail=200 database
docker network inspect PROJECT_DEFAULT_NETWORK
```

If the backend starts before PostgreSQL is ready, a minimal fix may be a health check plus a health-gated dependency:

```yaml
services:
  database:
    image: postgres:17
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 10

  backend:
    build: ./backend
    depends_on:
      database:
        condition: service_healthy
```

Then validate and restart only the affected services:

```bash
docker compose config
docker compose up -d --build database backend
docker compose ps
docker compose logs --tail=100 backend
```

The pattern is evidence first, narrow fix second, targeted restart third, and explicit verification last.


### Project-specific Docker rules for Codex {#project-specific-docker-rules-for-codex}

For repeated use, put Docker safety rules in `AGENTS.md` at the project root:

```markdown
# Docker Debugging Rules

- Inspect logs and status before modifying files.
- Explain the root cause before applying a fix.
- Prefer changes limited to the affected service.
- Do not run `docker compose down -v`.
- Do not run Docker prune commands.
- Do not delete named volumes.
- Do not reset or initialize databases.
- Do not expose additional ports without explaining why.
- Ask before stopping unrelated containers.
- After changes, validate with `docker compose config`.
- Verify fixes with service status, logs, health checks,
  and endpoint tests.
```

Codex reads `AGENTS.md` as durable project guidance at session start. Keep the file concise and place stricter rules closer to specialized subdirectories when only part of the repository needs them.


### Troubleshooting Docker access {#troubleshooting-docker-access}

Permission denied on the Docker socket:

```text
permission denied while trying to connect to the Docker daemon socket
```

Check socket permissions and group membership:

```bash
ls -l /var/run/docker.sock
id
getent group docker
```

If the user was recently added to the `docker` group, log out and log back in.

Docker daemon unavailable:

```text
Cannot connect to the Docker daemon
```

Check the service and recent daemon logs:

```bash
sudo systemctl status docker
sudo journalctl -u docker --no-pager -n 100
sudo systemctl start docker
```

Docker works manually but not from Codex:

```bash
docker ps
```

If that works in the terminal but fails inside Codex, the likely issue is the Codex sandbox or permission profile rather than Docker itself. Check:

-   whether Codex requested approval for the Docker command;
-   the active mode with `/permissions`;
-   whether the configured Unix socket matches the active Docker context;
-   whether older `sandbox_mode` settings are being mixed with permission profiles.

Wrong Docker context:

```bash
docker context show
docker context ls
docker context use default
```

If the active context points to a remote daemon, Codex may be debugging a different Docker host than expected.


### Recommended security posture {#recommended-security-posture}

For a development workstation:

-   run Codex as a normal Linux user;
-   keep project files under a dedicated workspace directory;
-   start with `workspace-write` and `on-request` approvals;
-   approve Docker commands after reviewing them;
-   use a Docker-specific permission profile only for frequent debugging;
-   allow only the Docker Unix socket and any required localhost endpoints;
-   avoid `danger-full-access` unless the host is isolated;
-   never allow unattended volume deletion or Docker pruning;
-   keep secrets in protected environment files;
-   review code and configuration changes before rebuilding services;
-   maintain backups for databases and persistent volumes.

Codex is most useful here as an evidence-gathering and change-minimizing debugging agent. It should inspect status, logs, health checks, networks, configuration, and endpoint behavior before it edits files or restarts services.


## Reference List {#reference-list}

1.  <https://developers.openai.com/codex/environment-variables>
2.  <https://developers.openai.com/codex/concepts/sandboxing>
3.  <https://developers.openai.com/codex/permissions>
4.  <https://developers.openai.com/codex/guides/agents-md>
5.  <https://docs.docker.com/engine/install/ubuntu/>
6.  <https://docs.docker.com/compose/>
7.  <https://docs.docker.com/engine/security/rootless/>
