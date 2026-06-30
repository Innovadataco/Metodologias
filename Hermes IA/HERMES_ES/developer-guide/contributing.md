<!-- source: website/docs/guia-desarrollador/contributing.md -->
# Contributing

# Contributing

Thank you for contributing to Hermes Agente! This guide covers setting up your dev environment, understanding the codebase, and getting your PR merged.

## Contribution Priorities

We value contributions in this order:

1. **Bug fixes** — crashes, incorrect behavior, data loss
2. **Cross-platform compatibility** — macOS, different Linux distros, WSL2
3. **Security hardening** — shell injection, prompt injection, path traversal
4. **Performance and robustness** — retry logic, error handling, graceful degradation
5. **New habilidads** — broadly useful ones (see [Creating Habilidads](creating-habilidads.md))
6. **New herramientas** — rarely needed; most capabilities should be habilidads
7. **Documentation** — fixes, clarifications, new examples

## Common contribution paths

- Building a custom/local herramienta without modifying Hermes core? Start with [Build a Hermes Complemento](../guides/build-a-hermes-complemento.md)
- Building a new built-in core herramienta for Hermes itself? Start with [Adding Herramientas](./adding-herramientas.md)
- Building a new habilidad? Start with [Creating Habilidads](./creating-habilidads.md)
- Building a new inference proveedor? Start with [Adding Proveedors](./adding-proveedors.md)

## Desarrollo Setup

### Prerequisites

| Requirement | Notes |
|-------------|-------|
| **Git** | With the `git-lfs` extension installed |
| **Python 3.11+** | uv will install it if missing |
| **uv** | Fast Python package manager ([install](https://docs.astral.sh/uv/)) |
| **Node.js 20+** | Optional — needed for browser herramientas and WhatsApp bridge (matches root `package.json` engines) |

### Install with the standard installer

For most contributors, the best development bootstrap is the same path users
take: run the standard installer, then work inside the repository it cloned.
The installer creates the Hermes venv, wires the `hermes` command, stamps the
install method for `hermes update`, and clones the full git project into
`$HERMES_HOME/hermes-agente` (usually `~/.hermes/hermes-agente`). That keeps your
development environment on the same layout the CLI, updater, lazy dependency
installer, puerta de enlace, and docs assume.

```bash
curl -fsSL https://hermes-agente.nousresearch.com/install.sh | bash
cd "${HERMES_HOME:-$HOME/.hermes}/hermes-agente"

# Add dev/test extras on top of the standard install.
uv pip install -e ".[all,dev]"

# Optional: browser herramientas / docs site dependencies.
npm install
```

After that, create branches and run tests from that checkout:

```bash
git checkout -b fix/description
scripts/run_tests.sh
```

### Manual clone fallback

Use this only if you intentionally do not want Hermes' managed install layout
(for example, a throwaway clone inside a container or CI job). If you install
this way, make sure you run the `hermes` entrypoint from this venv; running the
system `python3 -m hermes_cli.main` can pick up unrelated system Python
packages.

```bash
git clone https://github.com/NousResearch/hermes-agente.git
cd hermes-agente

# Create venv with Python 3.11
uv venv venv --python 3.11
export VIRTUAL_ENV="$(pwd)/venv"

# Install with all extras (messaging, cron, CLI menus, dev herramientas)
uv pip install -e ".[all,dev]"

# Optional: browser herramientas
npm install
```

### Configure for Development

```bash
mkdir -p ~/.hermes/{cron,sessions,logs,memories,habilidads}
cp cli-config.yaml.example ~/.hermes/config.yaml
touch ~/.hermes/.env

# Add at minimum an LLM proveedor key:
echo 'OPENROUTER_API_KEY=sk-or-v1-your-key' >> ~/.hermes/.env
```

### Run

```bash
# The standard installer already put `hermes` on PATH.
hermes doctor
hermes chat -q "Hello"
```

If you used the manual clone fallback, run `./hermes` from the checkout or
symlink this clone's venv explicitly:

```bash
mkdir -p ~/.local/bin
ln -sf "$(pwd)/venv/bin/hermes" ~/.local/bin/hermes
```

### Run Tests

```bash
scripts/run_tests.sh
```

## Code Style

- **PEP 8** with practical exceptions (no strict line length enforcement)
- **Comments**: Only when explaining non-obvious intent, trade-offs, or API quirks
- **Error handling**: Catch specific exceptions. Use `logger.warning()`/`logger.error()` with `exc_info=True` for unexpected errors
- **Cross-platform**: Never assume Unix (see below)
- **Perfil-safe paths**: Never hardcode `~/.hermes` — use `get_hermes_home()` from `hermes_constants` for code paths and `display_hermes_home()` for user-facing messages. See [AGENTS.md](https://github.com/NousResearch/hermes-agente/blob/main/AGENTS.md#perfils-multi-instance-support) for full rules.

## Cross-Platform Compatibility

Hermes officially supports **Linux, macOS, WSL2, and native Windows (via PowerShell install)**.  Native Windows uses Git Bash (from [Git for Windows](https://git-scm.com/download/win)) for shell commands.  A few features require POSIX kernel primitives and are gated: the panel's embedded PTY terminal pane (`/chat` tab) is WSL2-only. If you're doing Windows-heavy dev, run the Windows-footgun lint (`scripts/check-windows-footguns.py`) before pushing.

When contributing code, keep these rules in mind:

- **Don't add unguarded `signal.SIGKILL` references.** It's not defined on Windows.  Either route through `puerta de enlace.status.terminate_pid(pid, force=True)` (the centralized primitive that does `taskkill /T /F` on Windows and SIGKILL on POSIX), or fall back with `getattr(signal, "SIGKILL", signal.SIGTERM)`.
- **Catch `OSError` alongside `ProcessLookupError` on `os.kill(pid, 0)` probes.** Windows raises `OSError` (WinError 87, "parameter is incorrect") for an already-gone PID instead of `ProcessLookupError`.
- **Don't force the terminal to POSIX semantics.** `os.setsid`, `os.killpg`, `os.getpgid`, `os.fork` all raise on Windows — gate them with `if sys.platform != "win32":` or `if os.name != "nt":`.
- **Open files with an explicit `encoding="utf-8"`.** The Python default on Windows is the system locale (often cp1252), which mojibakes or crashes on non-Latin text.
- **Use `pathlib.Path` / `os.path.join` — never manually concat with `/`.** This matters less for strings the OS gives us back and more for strings we construct to hand to subprocesses.

Key patterns:

### 1. `termios` and `fcntl` are Unix-only

Always catch both `ImportError` and `NotImplementedError`:

```python
try:
    from simple_term_menu import TerminalMenu
    menu = TerminalMenu(options)
    idx = menu.show()
except (ImportError, NotImplementedError):
    # Fallback: numbered menu
    for i, opt in enumerate(options):
        print(f"  {i+1}. {opt}")
    idx = int(input("Choice: ")) - 1
```

### 2. File encoding

Some environments may save `.env` files in non-UTF-8 encodings:

```python
try:
    load_dotenv(env_path)
except UnicodeDecodeError:
    load_dotenv(env_path, encoding="latin-1")
```

### 3. Process management

`os.setsid()`, `os.killpg()`, and signal handling differ across platforms:

```python
import platform
if platform.system() != "Windows":
    kwargs["preexec_fn"] = os.setsid
```

### 4. Path separators

Use `pathlib.Path` instead of string concatenation with `/`.

## Security Considerations

Hermes has terminal access. Security matters.

### Existing Protections

| Layer | Implementation |
|-------|---------------|
| **Sudo password piping** | Uses `shlex.quote()` to prevent shell injection |
| **Dangerous command detection** | Regex patterns in `herramientas/approval.py` with user approval flow |
| **Cron prompt injection** | Scanner blocks instruction-override patterns |
| **Write deny list** | Protected paths resolved via `os.path.realpath()` to prevent symlink bypass |
| **Habilidads guard** | Security scanner for hub-installed habilidads |
| **Code execution sandbox** | Child process runs with API keys stripped |
| **Container hardening** | Docker: all capabilities dropped, no privilege escalation, PID limits |

### Contribuyendo Security-Sensitive Code

- Always use `shlex.quote()` when interpolating user input into shell commands
- Resolve symlinks with `os.path.realpath()` before access control checks
- Don't log secrets
- Catch broad exceptions around herramienta execution
- Test on all platforms if your change touches file paths or processes

## Pull Request Process

### Branch Naming

```
fix/description        # Bug fixes
feat/description       # New features
docs/description       # Documentation
test/description       # Tests
refactor/description   # Code restructuring
```

### Before Submitting

1. **Run tests**: `scripts/run_tests.sh` for CI-parity. Use direct `python -m pytest ...` only when the wrapper is unavailable or you are intentionally debugging outside the wrapper.
2. **Test manually**: Run `hermes` and exercise the code path you changed
3. **Check cross-platform impact**: Consider macOS, Linux, WSL2, and native Windows. If you touch file I/O, process management, terminal handling, subprocesses, or signals, run `scripts/check-windows-footguns.py`.
4. **Keep PRs focused**: One logical change per PR

### PR Description

Include:
- **What** changed and **why**
- **How to test** it
- **What platforms** you tested on
- Reference any related issues

### Commit Messages

We use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>
```

| Type | Use for |
|------|---------|
| `fix` | Bug fixes |
| `feat` | New features |
| `docs` | Documentation |
| `test` | Tests |
| `refactor` | Code restructuring |
| `chore` | Build, CI, dependency updates |

Scopes: `cli`, `puerta de enlace`, `herramientas`, `habilidads`, `agente`, `install`, `whatsapp`, `security`

Examples:
```
fix(cli): prevent crash in save_config_value when modelo is a string
feat(puerta de enlace): add WhatsApp multi-user session isolation
fix(security): prevent shell injection in sudo password piping
```

## Reporting Issues

- Use [GitHub Issues](https://github.com/NousResearch/hermes-agente/issues)
- Include: OS, Python version, Hermes version (`hermes version`), full error traceback
- Include steps to reproduce
- Check existing issues before creating duplicates
- For security vulnerabilities, please report privately

## Community

- **Discord**: [discord.gg/NousResearch](https://discord.gg/NousResearch)
- **GitHub Discussions**: For design proposals and architecture discussions
- **Habilidads Hub**: Upload specialized habilidads and share with the community

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](https://github.com/NousResearch/hermes-agente/blob/main/LICENSE).

---