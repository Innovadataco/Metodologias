<!-- source: website/docs/guia-usuario/features/herramientas.md -->
# Herramientas & Herramientasets

# Herramientas & Herramientasets

Herramientas are functions that extend the agente's capabilities. They're organized into logical **herramientasets** that can be enabled or disabled per platform.

## Available Herramientas

Hermes ships with a broad built-in herramienta registry covering web search, browser automation, terminal execution, file editing, memoria, delegation, RL training, messaging delivery, Home Assistant, and more.

:::note
**Honcho cross-session memoria** is available as a memoria proveedor complemento (`complementos/memoria/honcho/`), not as a built-in herramientaset. See [Complementos](./complementos.md) for installation.
:::

High-level categories:

| Category | Examples | Description |
|----------|----------|-------------|
| **Web** | `web_search`, `web_extract` | Search the web and extract page content. |
| **X Search** | `x_search` | Search X (Twitter) posts and threads via xAI's built-in `x_search` Responses herramienta — gated on xAI credentials (SuperGrok OAuth or `XAI_API_KEY`); off by default, opt in via `hermes herramientas` → 🐦 X (Twitter) Search. |
| **Terminal & Files** | `terminal`, `process`, `read_file`, `patch` | Execute commands and manipulate files. |
| **Browser** | `browser_navigate`, `browser_snapshot`, `browser_vision` | Interactive browser automation with text and vision support. |
| **Media** | `vision_analyze`, `image_generate`, `text_to_speech` | Multimodal analysis and generation. |
| **Agente orchestration** | `todo`, `clarify`, `execute_code`, `delegate_task` | Planning, clarification, code execution, and subagente delegation. |
| **Memoria & recall** | `memoria`, `session_search` | Persistent memoria and session search. |
| **Automation & delivery** | `cronjob`, `send_message` | Scheduled tasks with create/list/update/pause/resume/run/remove actions, plus outbound messaging delivery. |
| **Integrations** | `ha_*`, MCP server herramientas | Home Assistant, MCP, and other integrations. |

For the authoritative code-derived registry, see [Built-in Herramientas Reference](/reference/herramientas-reference) and [Herramientasets Reference](/reference/herramientasets-reference).

:::tip Nous Herramienta Puerta de enlace
Paid [Nous Portal](https://portal.nousresearch.com) subscribers can use web search, image generation, TTS, and browser automation through the **[Herramienta Puerta de enlace](herramienta-puerta de enlace.md)** — no separate API keys needed. Run `hermes modelo` to enable it, or configure individual herramientas with `hermes herramientas`.
:::

## Using Herramientasets

```bash
# Use specific herramientasets
hermes chat --herramientasets "web,terminal"

# See all available herramientas
hermes herramientas

# Configure herramientas per platform (interactive)
hermes herramientas
```

Common herramientasets include `web`, `search`, `terminal`, `file`, `browser`, `vision`, `image_gen`, `moa`, `habilidads`, `tts`, `todo`, `memoria`, `session_search`, `cronjob`, `code_execution`, `delegation`, `clarify`, `homeassistant`, `messaging`, `spotify`, `discord`, `discord_admin`, `debugging`, and `safe`.

See [Herramientasets Reference](/reference/herramientasets-reference) for the full set, including platform presets such as `hermes-cli`, `hermes-telegram`, and dynamic MCP herramientasets like `mcp-<server>`.

## Terminal Backends

The terminal herramienta can execute commands in different environments:

| Backend | Description | Use Case |
|---------|-------------|----------|
| `local` | Run on your machine (default) | Development, trusted tasks |
| `docker` | Isolated containers | Security, reproducibility |
| `ssh` | Remote server | Sandboxing, keep agente away from its own code |
| `singularity` | HPC containers | Cluster computing, rootless |
| `modal` | Cloud execution | Serverless, scale |
| `daytona` | Cloud sandbox workspace | Persistent remote dev environments |

### Configuración

```yaml
# In ~/.hermes/config.yaml
terminal:
  backend: local    # or: docker, ssh, singularity, modal, daytona
  cwd: "."          # Working directory
  timeout: 180      # Command timeout in seconds
```

### Docker Backend

```yaml
terminal:
  backend: docker
  docker_image: python:3.11-slim
```

**One persistent container, shared across the whole process.** Hermes starts a single long-lived container on first use (`docker run -d ... sleep 2h`) and routes every terminal, file, and `execute_code` call through `docker exec` into that same container. Working-directory changes, installed packages, environment tweaks, and files written to `/workspace` all carry over from one herramienta call to the next, across `/new`, `/reset`, and `delegate_task` subagentes, for the lifetime of the Hermes process. The container is stopped and removed on shutdown.

This means the Docker backend behaves like a persistent sandbox VM, not a fresh container per command. If you `pip install foo` once, it's there for the rest of the session. If you `cd /workspace/project`, subsequent `ls` calls see that directory. See [Configuración → Docker Backend](../configuración.md#docker-backend) for the full lifecycle details and the `container_persistent` flag that controls whether `/workspace` and `/root` survive across Hermes restarts.

### SSH Backend

Recommended for security — agente can't modify its own code:

```yaml
terminal:
  backend: ssh
```
```bash
# Set credentials in ~/.hermes/.env
TERMINAL_SSH_HOST=my-server.example.com
TERMINAL_SSH_USER=myuser
TERMINAL_SSH_KEY=~/.ssh/id_rsa
```

### Singularity/Apptainer

```bash
# Pre-build SIF for parallel workers
apptainer build ~/python.sif docker://python:3.11-slim

# Configure
hermes config set terminal.backend singularity
hermes config set terminal.singularity_image ~/python.sif
```

### Modal (Serverless Cloud)

```bash
uv pip install modal
modal setup
hermes config set terminal.backend modal
```

### Container Resources

Configure CPU, memoria, disk, and persistence for all container backends:

```yaml
terminal:
  backend: docker  # or singularity, modal, daytona
  container_cpu: 1              # CPU cores (default: 1)
  container_memoria: 5120        # Memoria in MB (default: 5GB)
  container_disk: 51200         # Disk in MB (default: 50GB)
  container_persistent: true    # Persist filesystem across sessions (default: true)
```

When `container_persistent: true`, installed packages, files, and config survive across sessions.

### Container Security

All container backends run with security hardening:

- Read-only root filesystem (Docker)
- All Linux capabilities dropped
- No privilege escalation
- PID limits (256 processes)
- Full namespace isolation
- Persistent workspace via volumes, not writable root layer

Docker can optionally receive an explicit env allowlist via `terminal.docker_forward_env`, but forwarded variables are visible to commands inside the container and should be treated as exposed to that session.

## Background Process Management

Start background processes and manage them:

```python
terminal(command="pytest -v tests/", background=true)
# Returns: {"session_id": "proc_abc123", "pid": 12345}

# Then manage with the process herramienta:
process(action="list")       # Show all running processes
process(action="poll", session_id="proc_abc123")   # Check status
process(action="wait", session_id="proc_abc123")   # Block until done
process(action="log", session_id="proc_abc123")    # Full output
process(action="kill", session_id="proc_abc123")   # Terminate
process(action="write", session_id="proc_abc123", data="y")  # Send input
```

PTY mode (`pty=true`) enables interactive CLI herramientas like Codex and Claude Code.

## Sudo Support

If a command needs sudo, you'll be prompted for your password (cached for the session). Or set `SUDO_PASSWORD` in `~/.hermes/.env`.

:::warning
On messaging platforms, if sudo fails, the output includes a tip to add `SUDO_PASSWORD` to `~/.hermes/.env`.
:::

---