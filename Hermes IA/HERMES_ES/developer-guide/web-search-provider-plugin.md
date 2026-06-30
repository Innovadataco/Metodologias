<!-- source: website/docs/guia-desarrollador/web-search-proveedor-complemento.md -->
# Web Search Proveedor Complementos

# Building a Web Search Proveedor Complemento

Web-search proveedor complementos register a backend that services `web_search`, `web_extract`, and (optionally) deep-crawl herramienta calls. Built-in proveedors — Firecrawl, SearXNG, Tavily, Exa, Parallel, Brave Search (free tier), xAI, and DDGS — all ship as complementos under `complementos/web/<name>/`. You can add a new one, or override a bundled one, by dropping a directory next to them.

:::tip
Web search is one of several **backend complementos** Hermes supports. The others (with their own ABCs) are [Image Generation Proveedor Complementos](/guia-desarrollador/image-gen-proveedor-complemento), [Video Generation Proveedor Complementos](/guia-desarrollador/video-gen-proveedor-complemento), [Memoria Proveedor Complementos](/guia-desarrollador/memoria-proveedor-complemento), [Contexto Engine Complementos](/guia-desarrollador/contexto-engine-complemento), and [Modelo Proveedor Complementos](/guia-desarrollador/modelo-proveedor-complemento). General herramienta/hook/CLI complementos live in [Build a Hermes Complemento](/guides/build-a-hermes-complemento).
:::

## How discovery works

Hermes scans for web-search backends in three places:

1. **Bundled** — `<repo>/complementos/web/<name>/` (auto-loaded with `kind: backend`, always available)
2. **User** — `~/.hermes/complementos/web/<name>/` (opt-in via `complementos.enabled` or `hermes complementos enable <name>`)
3. **Pip** — packages declaring a `hermes_agente.complementos` entry point

Each complemento's `register(ctx)` function calls `ctx.register_web_search_proveedor(...)` — that puts the instance into the registry in `agente/web_search_registry.py`. The active proveedor for each capability is picked by config:

| Capability | Config key | Falls back to |
|---|---|---|
| `web_search` | `web.search_backend` | `web.backend` |
| `web_extract` | `web.extract_backend` | `web.backend` |
| Deep crawl modes inside `web_extract` | `web.extract_backend` | `web.backend` |

When neither key is set, Hermes auto-detects the backend from whichever API key/URL is present in the environment. `hermes herramientas` walks users through selection.

## Directory structure

```
complementos/web/my-backend/
├── __init__.py     # register() entry point
├── proveedor.py     # WebSearchProveedor subclass
└── complemento.yaml     # Manifest with kind: backend and provides_web_proveedors
```

`brave_free/` and `ddgs/` are the smallest in-tree references — `brave_free` for an API-key-gated search-only proveedor, `ddgs` for a no-key proveedor that lazy-installs its SDK.

## The WebSearchProveedor ABC

Subclass `agente.web_search_proveedor.WebSearchProveedor`. The only required members are `name`, `is_available()`, and whichever of `search()` / `extract()` you implement. (Deep crawling is not a separate method — it's a mode of `extract()`.)

```python
# complementos/web/my-backend/proveedor.py
from __future__ import annotations

import os
from typing import Any, Dict, List

from agente.web_search_proveedor import WebSearchProveedor


class MyBackendWebSearchProveedor(WebSearchProveedor):
    """Minimal search-only proveedor against the My Backend HTTP API."""

    @property
    def name(self) -> str:
        # Stable id used in web.search_backend / web.extract_backend / web.backend
        # config keys. Lowercase, no spaces; hyphens permitted.
        return "my-backend"

    @property
    def display_name(self) -> str:
        # Human label shown in `hermes herramientas`. Defaults to `name`.
        return "My Backend"

    def is_available(self) -> bool:
        # Cheap check — env var present, optional dep importable, etc.
        # MUST NOT make network calls (runs on every `hermes herramientas` paint).
        return bool(os.getenv("MY_BACKEND_API_KEY", "").strip())

    def supports_search(self) -> bool:
        return True

    def supports_extract(self) -> bool:
        return False

    def search(self, query: str, limit: int = 5) -> Dict[str, Any]:
        import httpx

        api_key = os.environ["MY_BACKEND_API_KEY"]
        try:
            resp = httpx.get(
                "https://api.example.com/search",
                params={"q": query, "count": max(1, min(int(limit), 20))},
                headers={"Authorization": f"Bearer {api_key}"},
                timeout=15,
            )
            resp.raise_for_status()
            data = resp.json()
        except httpx.HTTPError as exc:
            return {"success": False, "error": str(exc)}

        # Response shape is fixed — see "Response shape" below.
        return {
            "success": True,
            "data": {
                "web": [
                    {
                        "title": item.get("title", ""),
                        "url": item.get("url", ""),
                        "description": item.get("snippet", ""),
                        "position": idx + 1,
                    }
                    for idx, item in enumerate(data.get("results", []))
                ],
            },
        }
```

```python
# complementos/web/my-backend/__init__.py
from complementos.web.my_backend.proveedor import MyBackendWebSearchProveedor


def register(ctx) -> None:
    """Complemento entry point — called once at load time."""
    ctx.register_web_search_proveedor(MyBackendWebSearchProveedor())
```

## complemento.yaml

```yaml
name: web-my-backend
version: 1.0.0
description: "My Backend web search — Bearer-auth REST API"
author: Your Name
kind: backend
provides_web_proveedors:
  - my-backend
requires_env:
  - MY_BACKEND_API_KEY
```

| Key | Purpose |
|---|---|
| `kind: backend` | Routes the complemento through the backend-loading path |
| `provides_web_proveedors` | List of proveedor `name`s this complemento registers — used by the loader to advertise the complemento in `hermes herramientas` even before `register()` runs |
| `requires_env` | Interactive credential prompt during `hermes complementos install` (see [Build a Hermes Complemento](/guides/build-a-hermes-complemento#gate-on-environment-variables) for the rich format) |

## ABC reference

Full contract in `agente/web_search_proveedor.py`. Methods you may override:

| Member | Required | Default | Purpose |
|---|---|---|---|
| `name` | ✅ | — | Stable id used in `web.*_backend` config |
| `display_name` | — | `name` | Label shown in `hermes herramientas` |
| `is_available()` | ✅ | — | Cheap availability gate — env vars, optional deps |
| `supports_search()` | — | `True` | Capability flag for `web_search` routing |
| `supports_extract()` | — | `False` | Capability flag for `web_extract` routing |
| `search(query, limit)` | conditional | raises | Required when `supports_search()` returns `True` |
| `extract(urls, **kwargs)` | conditional | raises | Required when `supports_extract()` returns `True` |

Proveedors can advertise multiple capabilities from a single class — Firecrawl, Tavily, Exa, and Parallel all implement both search and extract. Brave Search and DDGS are search-only; SearXNG is search-only with a documented "pair me with an extract proveedor" workflow.

## Response shape

The herramienta wrapper expects a fixed envelope so it doesn't have to translate between backends.

**Search success:**

```python
{
    "success": True,
    "data": {
        "web": [
            {"title": str, "url": str, "description": str, "position": int},
            ...
        ],
    },
}
```

**Extract success:**

```python
{
    "success": True,
    "data": [
        {
            "url": str,
            "title": str,
            "content": str,
            "raw_content": str,
            "metadata": dict,    # optional
            "error": str,        # optional, only on per-URL failure
        },
        ...
    ],
}
```

**Either capability, on failure:**

```python
{"success": False, "error": "human-readable message"}
```

Both `search()` and `extract()` may be `async def` — the dispatcher detects coroutine functions via `inspect.iscoroutinefunction` and awaits accordingly. Sync implementations that do blocking I/O (HTTP, SDK calls) are fine for small backends; the dispatcher handles threading.

## Capability flags

Hermes routes calls to the right proveedor based on the `supports_*` flags. A common multi-proveedor setup:

```yaml
# ~/.hermes/config.yaml
web:
  search_backend: "brave-free"     # search-only, fast, free 2k/mo
  extract_backend: "firecrawl"     # extract + crawl, paid quota
```

When `web.search_backend` or `web.extract_backend` aren't set, both fall through to `web.backend`. When that's also unset, Hermes picks the first available proveedor that supports the requested capability based on env-var presence.

If your proveedor only supports one capability, leave the other flags at their default (`False`) and the registry will skip it for that herramienta — users won't see misleading "proveedor X failed" errors when they're using X only for search and asking the agente to extract.

## How Hermes wires it into the herramientas

The `web_search` and `web_extract` herramientas live in `herramientas/web_herramientas.py`. At call time they:

1. Read the relevant config key (`web.search_backend` for `web_search`, `web.extract_backend` for `web_extract`)
2. Ask the registry for the proveedor with that `name`
3. Check `is_available()` and the matching `supports_*()` flag
4. Dispatch to `search()` / `extract()` (deep crawl runs as a mode inside `extract()`), awaiting if the method is a coroutine
5. JSON-serialize the response envelope and hand it back to the LLM

Errors surface as the herramienta result; the LLM decides how to explain them. If no proveedor is registered (or every available one fails the capability gate), the herramienta returns a helpful error pointing at `hermes herramientas`.

## Lazy-installing optional dependencies

If your proveedor wraps a third-party SDK (like DDGS does with the `ddgs` package), don't `import` it at module top level. Use `herramientas.lazy_deps.ensure(...)` inside `is_available()` or `search()` — Hermes will install the package on first use, gated by `security.allow_lazy_installs`. See [Build a Hermes Complemento → Lazy-install](/guides/build-a-hermes-complemento#lazy-install-optional-python-dependencies) for the security modelo.

## Referencia implementations

- **`complementos/web/brave_free/`** — small, API-key-gated, search-only HTTP proveedor. Good starting template.
- **`complementos/web/ddgs/`** — no-key proveedor that lazy-installs its SDK. Useful pattern for backends that wrap a Python package.
- **`complementos/web/firecrawl/`** — full multi-capability proveedor (search + extract + crawl) with multiple format modes.
- **`complementos/web/searxng/`** — self-hosted, URL-configured backend with no auth.
- **`complementos/web/xai/`** — LLM-backed search via Grok's server-side `web_search` herramienta. Shows how to reuse an existing OAuth/env-var credential surface (`herramientas/xai_http.py`) without adding new env vars, and how to write a cheap `is_available()` that honors the no-network contract.

## Distribute via pip

```toml
# pyproject.toml
[project.entry-points."hermes_agente.complementos"]
my-backend-web = "my_backend_web_package"
```

`my_backend_web_package` must expose a top-level `register` function. See [Distribute via pip](/guides/build-a-hermes-complemento#distribute-via-pip) in the general complemento guide for the full setup.

## Related pages

- [Web Search](/guia-usuario/features/web-search) — user-facing feature documentation and per-backend configuración
- [Complementos overview](/guia-usuario/features/complementos) — all complemento types at a glance
- [Build a Hermes Complemento](/guides/build-a-hermes-complemento) — general herramientas/hooks/slash commands guide

---