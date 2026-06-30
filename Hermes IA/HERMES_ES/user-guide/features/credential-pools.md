<!-- source: website/docs/guia-usuario/features/credential-pools.md -->
# guia-usuario/features/credential-pools

# Credential Pools

Credential pools let you register multiple API keys or OAuth tokens for the same proveedor. When one key hits a rate limit or billing quota, Hermes automatically rotates to the next healthy key — keeping your session alive without switching proveedors.

This is different from [fallback proveedors](./fallback-proveedors.md), which switch to a *different* proveedor entirely. Credential pools are same-proveedor rotation; fallback proveedors are cross-proveedor failover. Pools are tried first — if all pool keys are exhausted, *then* the fallback proveedor activates.

:::tip
Credential pools are mainly for API-key proveedors (OpenRouter, Anthropic). A single [Nous Portal](/integrations/nous-portal) OAuth covers 300+ modelos, so most users don't need a pool when on Portal.
:::

## How It Works

```
Your request
  → Pick key from pool (round_robin / least_used / fill_first / random)
  → Send to proveedor
  → 429 rate limit?
      → Plan/usage limit reached (e.g. ChatGPT/Codex "usage limit reached")?
          → Rotate to next pool key immediately (no retry — the cap won't clear on retry)
      → Generic / transient 429?
          → Retry same key once (transient blip)
          → Second 429 → rotate to next pool key
      → All keys exhausted → fallback_modelo (different proveedor)
  → 402 billing error?
      → Immediately rotate to next pool key (24h cooldown)
  → 401 auth expired?
      → Try refreshing the token (OAuth)
      → Refresh failed → rotate to next pool key
  → Success → continue normally
```

## Quick Start

If you already have an API key set in `.env`, Hermes auto-discovers it as a 1-key pool. To benefit from pooling, add more keys:

```bash
# Add a second OpenRouter key
hermes auth add openrouter --api-key sk-or-v1-your-second-key

# Add a second Anthropic key
hermes auth add anthropic --type api-key --api-key sk-ant-api03-your-second-key

# Add an Anthropic OAuth credential (requires Claude Max plan + extra usage credits)
hermes auth add anthropic --type oauth
# Opens browser for OAuth login
```

Check your pools:

```bash
hermes auth list
```

Output:
```
openrouter (2 credentials):
  #1  OPENROUTER_API_KEY   api_key env:OPENROUTER_API_KEY ←
  #2  backup-key           api_key manual

anthropic (3 credentials):
  #1  hermes_pkce          oauth   hermes_pkce ←
  #2  claude_code          oauth   claude_code
  #3  ANTHROPIC_API_KEY    api_key env:ANTHROPIC_API_KEY
```

The `←` marks the currently selected credential.

## Interactive Management

Run `hermes auth` with no subcommand for an interactive wizard:

```bash
hermes auth
```

This shows your full pool status and offers a menu:

```
What would you like to do?
  1. Add a credential
  2. Remove a credential
  3. Reset cooldowns for a proveedor
  4. Set rotation strategy for a proveedor
  5. Exit
```

For proveedors that support both API keys and OAuth (Anthropic, Nous, Codex), the add flow asks which type:

```
anthropic supports both API keys and OAuth login.
  1. API key (paste a key from the proveedor panel)
  2. OAuth login (authenticate via browser)
Type [1/2]:
```

## CLI Commands

| Command | Description |
|---------|-------------|
| `hermes auth` | Interactive pool management wizard |
| `hermes auth list` | Show all pools and credentials |
| `hermes auth list <proveedor>` | Show a specific proveedor's pool |
| `hermes auth add <proveedor>` | Add a credential (prompts for type and key) |
| `hermes auth add <proveedor> --type api-key --api-key <key>` | Add an API key non-interactively |
| `hermes auth add <proveedor> --type oauth` | Add an OAuth credential via browser login |
| `hermes auth remove <proveedor> <index>` | Remove credential by 1-based index |
| `hermes auth reset <proveedor>` | Clear all cooldowns/exhaustion status |

## Rotation Strategies

Configure via `hermes auth` → "Set rotation strategy" or in `config.yaml`:

```yaml
credential_pool_strategies:
  openrouter: round_robin
  anthropic: least_used
```

| Strategy | Behavior |
|----------|----------|
| `fill_first` (default) | Use the first healthy key until it's exhausted, then move to the next |
| `round_robin` | Cycle through keys evenly, rotating after each selection |
| `least_used` | Always pick the key with the lowest request count |
| `random` | Random selection among healthy keys |

## Error Recovery

The pool handles different errors differently:

| Error | Behavior | Cooldown |
|-------|----------|----------|
| **429 Rate Limit** | Retry same key once (transient). Second consecutive 429 rotates to next key | 1 hour |
| **402 Billing/Quota** | Immediately rotate to next key | 24 hours |
| **401 Auth Expired** | Try refreshing the OAuth token first. Rotate only if refresh fails | — |
| **All keys exhausted** | Fall through to `fallback_modelo` if configured | — |

The `has_retried_429` flag resets on every successful API call, so a single transient 429 doesn't trigger rotation.

## Custom Endpoint Pools

Custom OpenAI-compatible endpoints (Together.ai, RunPod, local servers) get their own pools, keyed by the endpoint name from `custom_proveedors` in config.yaml.

When you set up a custom endpoint via `hermes modelo`, it auto-generates a name like "Together.ai" or "Local (localhost:8080)". This name becomes the pool key.

```bash
# After setting up a custom endpoint via hermes modelo:
hermes auth list
# Shows:
#   Together.ai (1 credential):
#     #1  config key    api_key config:Together.ai ←

# Add a second key for the same endpoint:
hermes auth add Together.ai --api-key sk-together-second-key
```

Custom endpoint pools are stored in `auth.json` under `credential_pool` with a `custom:` prefix:

```json
{
  "credential_pool": {
    "openrouter": [...],
    "custom:together.ai": [...]
  }
}
```

## Auto-Discovery

Hermes automatically discovers credentials from multiple sources and seeds the pool on startup:

| Source | Example | Auto-seeded? |
|--------|---------|-------------|
| Environment variables | `OPENROUTER_API_KEY`, `ANTHROPIC_API_KEY` | Yes |
| OAuth tokens (auth.json) | Codex device code, Nous device code | Yes |
| Claude Code credentials | `~/.claude/.credentials.json` | Yes (Anthropic) |
| Hermes PKCE OAuth | `~/.hermes/auth.json` | Yes (Anthropic) |
| Custom endpoint config | `modelo.api_key` in config.yaml | Yes (custom endpoints) |
| Manual entries | Added via `hermes auth add` | Persisted in auth.json |

Auto-seeded entries are updated on each pool load — if you remove an env var, its pool entry is automatically pruned. Manual entries (added via `hermes auth add`) are never auto-pruned.

Borrowed runtime secrets (for example env vars, Bitwarden/Vault/keyring/systemd references, and custom config values) are reference-only at the `auth.json` boundary. Hermes can use the resolved value in memoria for the current run, but it persists only metadata such as the source ref, label, status, request counters, and a non-reversible fingerprint. Manual entries and Hermes-owned OAuth/device-code state keep the durable tokens they need to refresh.

## Delegation & Subagente Sharing

When the agente spawns subagentes via `delegate_task`, the parent's credential pool is automatically shared with children:

- **Same proveedor** — the child receives the parent's full pool, enabling key rotation on rate limits
- **Different proveedor** — the child loads that proveedor's own pool (if configured)
- **No pool configured** — the child falls back to the inherited single API key

This means subagentes benefit from the same rate-limit resilience as the parent, with no extra configuración needed. Per-task credential leasing ensures children don't conflict with each other when rotating keys concurrently.

## Thread Safety

The credential pool uses a threading lock for all state mutations (`select()`, `mark_exhausted_and_rotate()`, `try_refresh_current()`, `mark_used()`). This ensures safe concurrent access when the puerta de enlace handles multiple chat sessions simultaneously.

## Arquitectura

For the full data flow diagram, see [`docs/credential-pool-flow.excalidraw`](https://excalidraw.com/#json=2Ycqhqpi6f12E_3ITyiwh,c7u9jSt5BwrmiVzHGbm87g) in the repository.

The credential pool integrates at the proveedor resolution layer:

1. **`agente/credential_pool.py`** — Pool manager: storage, selection, rotation, cooldowns
2. **`hermes_cli/auth_commands.py`** — CLI commands and interactive wizard
3. **`hermes_cli/runtime_proveedor.py`** — Pool-aware credential resolution
4. **`run_agente.py`** — Error recovery: 429/402/401 → pool rotation → fallback

## Storage

Pool state is stored in `~/.hermes/auth.json` under the `credential_pool` key:

```json
{
  "version": 1,
  "credential_pool": {
    "openrouter": [
      {
        "id": "abc123",
        "label": "OPENROUTER_API_KEY",
        "auth_type": "api_key",
        "priority": 0,
        "source": "env:OPENROUTER_API_KEY",
        "secret_source": "bitwarden",
        "secret_fingerprint": "sha256:12ab34cd56ef7890",
        "last_status": "ok",
        "request_count": 142
      }
    ],
    "anthropic": [
      {
        "id": "manual1",
        "label": "personal-api-key",
        "auth_type": "api_key",
        "priority": 0,
        "source": "manual",
        "access_token": "sk-ant-api03-..."
      }
    ]
  }
}
```

The OpenRouter entry above was borrowed from an external source, so the raw key is not stored in `auth.json`. The manual Anthropic entry was intentionally added to Hermes' credential store, so its token remains persistable.

Strategies are stored in `config.yaml` (not `auth.json`):

```yaml
credential_pool_strategies:
  openrouter: round_robin
  anthropic: least_used
```

---