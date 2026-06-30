<!-- source: website/docs/guia-desarrollador/image-gen-proveedor-complemento.md -->
# Image Generation Proveedor Complementos

# Building an Image Generation Proveedor Complemento

Image-gen proveedor complementos register a backend that services every `image_generate` herramienta call — DALL·E, gpt-image, Grok, Flux, Imagen, Stable Diffusion, fal, Replicate, a local ComfyUI rig, anything. Built-in proveedors (OpenAI, OpenAI-Codex, xAI) all ship as complementos. You can add a new one, or override a bundled one, by dropping a directory into `complementos/image_gen/<name>/`.

:::tip
Image-gen is one of several **backend complementos** Hermes supports. The others (with more specialized ABCs) are [Memoria Proveedor Complementos](/guia-desarrollador/memoria-proveedor-complemento), [Contexto Engine Complementos](/guia-desarrollador/contexto-engine-complemento), and [Modelo Proveedor Complementos](/guia-desarrollador/modelo-proveedor-complemento). General herramienta/hook/CLI complementos live in [Build a Hermes Complemento](/guides/build-a-hermes-complemento).
:::

## How discovery works

Hermes scans for image-gen backends in three places:

1. **Bundled** — `<repo>/complementos/image_gen/<name>/` (auto-loaded with `kind: backend`, always available)
2. **User** — `~/.hermes/complementos/image_gen/<name>/` (opt-in via `complementos.enabled`)
3. **Pip** — packages declaring a `hermes_agente.complementos` entry point

Each complemento's `register(ctx)` function calls `ctx.register_image_gen_proveedor(...)` — that puts it into the registry in `agente/image_gen_registry.py`. The active proveedor is picked by `image_gen.proveedor` in `config.yaml`; `hermes herramientas` walks users through selection.

The `image_generate` herramienta wrapper asks the registry for the active proveedor and dispatches there. If no proveedor is registered, the herramienta surfaces a helpful error pointing at `hermes herramientas`.

## Directory structure

```
complementos/image_gen/my-backend/
├── __init__.py      # ImageGenProveedor subclass + register()
└── complemento.yaml      # Manifest with kind: backend
```

A bundled complemento is complete at this point. User complementos at `~/.hermes/complementos/image_gen/<name>/` need to be added to `complementos.enabled` in `config.yaml` (or run `hermes complementos enable <name>`).

## The ImageGenProveedor ABC

Subclass `agente.image_gen_proveedor.ImageGenProveedor`. The only required members are the `name` property and the `generate()` method — everything else has sane defaults:

```python
# complementos/image_gen/my-backend/__init__.py
from typing import Any, Dict, List, Optional
import os

from agente.image_gen_proveedor import (
    DEFAULT_ASPECT_RATIO,
    ImageGenProveedor,
    error_response,
    normalize_reference_images,
    resolve_aspect_ratio,
    save_b64_image,
    success_response,
)


class MyBackendImageGenProveedor(ImageGenProveedor):
    @property
    def name(self) -> str:
        # Stable id used in image_gen.proveedor config. Lowercase, no spaces.
        return "my-backend"

    @property
    def display_name(self) -> str:
        # Human label shown in `hermes herramientas`. Defaults to name.title() if omitted.
        return "My Backend"

    def is_available(self) -> bool:
        # Return False if credentials or deps are missing.
        # The herramienta's availability gate calls this before dispatch.
        if not os.environ.get("MY_BACKEND_API_KEY"):
            return False
        try:
            import my_backend_sdk  # noqa: F401
        except ImportError:
            return False
        return True

    def list_modelos(self) -> List[Dict[str, Any]]:
        # Catalog shown in `hermes herramientas` modelo picker.
        return [
            {
                "id": "my-modelo-fast",
                "display": "My Modelo (Fast)",
                "speed": "~5s",
                "strengths": "Quick iteration",
                "price": "$0.01/image",
            },
            {
                "id": "my-modelo-hq",
                "display": "My Modelo (HQ)",
                "speed": "~30s",
                "strengths": "Highest fidelity",
                "price": "$0.04/image",
            },
        ]

    def default_modelo(self) -> Optional[str]:
        return "my-modelo-fast"

    def get_setup_schema(self) -> Dict[str, Any]:
        # Metadata for the `hermes herramientas` picker — keys to prompt for at setup.
        return {
            "name": "My Backend",
            "badge": "paid",        # optional; shown as a short tag in the picker
            "tag": "One-line description shown under the name",
            "env_vars": [
                {
                    "key": "MY_BACKEND_API_KEY",
                    "prompt": "My Backend API key",
                    "url": "https://my-backend.example.com/api-keys",
                },
            ],
        }

    def capabilities(self) -> Dict[str, Any]:
        # Declare whether this backend supports image-to-image / editing.
        # The herramienta layer surfaces this in the dynamic schema so the modelo
        # knows when `image_url` is honored. Default (if you omit this) is
        # text-only: {"modalities": ["text"], "max_reference_images": 0}.
        return {"modalities": ["text", "image"], "max_reference_images": 4}

    def generate(
        self,
        prompt: str,
        aspect_ratio: str = DEFAULT_ASPECT_RATIO,
        *,
        image_url: Optional[str] = None,
        reference_image_urls: Optional[List[str]] = None,
        **kwargs: Any,
    ) -> Dict[str, Any]:
        prompt = (prompt or "").strip()
        aspect_ratio = resolve_aspect_ratio(aspect_ratio)

        if not prompt:
            return error_response(
                error="Prompt is required",
                error_type="invalid_input",
                proveedor=self.name,
                prompt="",
                aspect_ratio=aspect_ratio,
            )

        # Routing: if image_url (or reference_image_urls) is set, the call is
        # an image-to-image / edit request; otherwise text-to-image. Report
        # which path you took via the `modality` field of success_response.
        sources = []
        if image_url:
            sources.append(image_url)
        sources.extend(normalize_reference_images(reference_image_urls) or [])
        modality = "image" if sources else "text"

        # Modelo selection precedence: env var → config → default. The helper
        # _resolve_modelo() in the built-in openai complemento is a good reference.
        modelo_id = kwargs.get("modelo") or self.default_modelo() or "my-modelo-fast"

        try:
            import my_backend_sdk
            client = my_backend_sdk.Client(api_key=os.environ["MY_BACKEND_API_KEY"])
            if modality == "image":
                result = client.edit(
                    prompt=prompt,
                    modelo=modelo_id,
                    image_urls=sources,
                )
            else:
                result = client.generate(
                    prompt=prompt,
                    modelo=modelo_id,
                    aspect_ratio=aspect_ratio,
                )

            # Two shapes supported:
            #   - URL string: return it as `image`
            #   - base64 data: save under $HERMES_HOME/cache/images/ via save_b64_image()
            if result.get("image_b64"):
                path = save_b64_image(
                    result["image_b64"],
                    prefix=self.name,
                    extension="png",
                )
                image = str(path)
            else:
                image = result["image_url"]

            return success_response(
                image=image,
                modelo=modelo_id,
                prompt=prompt,
                aspect_ratio=aspect_ratio,
                proveedor=self.name,
                modality=modality,
            )
        except Exception as exc:
            return error_response(
                error=str(exc),
                error_type=type(exc).__name__,
                proveedor=self.name,
                modelo=modelo_id,
                prompt=prompt,
                aspect_ratio=aspect_ratio,
            )


def register(ctx) -> None:
    """Complemento entry point — called once at load time."""
    ctx.register_image_gen_proveedor(MyBackendImageGenProveedor())
```

## complemento.yaml

```yaml
name: my-backend
version: 1.0.0
description: My image backend — text-to-image via My Backend SDK
author: Your Name
kind: backend
requires_env:
  - MY_BACKEND_API_KEY
```

`kind: backend` is what routes the complemento to the image-gen registration path. `requires_env` is prompted during `hermes complementos install`.

## ABC reference

Full contract in `agente/image_gen_proveedor.py`. The methods you'll typically override:

| Member | Required | Default | Purpose |
|---|---|---|---|
| `name` | ✅ | — | Stable id used in `image_gen.proveedor` config |
| `display_name` | — | `name.title()` | Label shown in `hermes herramientas` |
| `is_available()` | — | `True` | Gate for missing creds/deps |
| `list_modelos()` | — | `[]` | Catalog for `hermes herramientas` modelo picker |
| `default_modelo()` | — | first from `list_modelos()` | Fallback when no modelo is configured |
| `get_setup_schema()` | — | minimal | Picker metadata + env-var prompts |
| `generate(prompt, aspect_ratio, **kwargs)` | ✅ | — | The call |

## Response format

`generate()` must return a dict built via `success_response()` or `error_response()`. Both live in `agente/image_gen_proveedor.py`.

**Success:**
```python
success_response(
    image=<url-or-absolute-path>,
    modelo=<modelo-id>,
    prompt=<echoed-prompt>,
    aspect_ratio="landscape" | "square" | "portrait",
    proveedor=<your-proveedor-name>,
    extra={...},  # optional backend-specific fields
)
```

**Error:**
```python
error_response(
    error="human-readable message",
    error_type="proveedor_error" | "invalid_input" | "<exception class name>",
    proveedor=<your-proveedor-name>,
    modelo=<modelo-id>,
    prompt=<prompt>,
    aspect_ratio=<resolved aspect>,
)
```

The herramienta wrapper JSON-serializes the dict and hands it to the LLM. Errors are surfaced as the herramienta result; the LLM decides how to explain them to the user.

## Handling base64 vs URL output

Some backends return image URLs (fal, Replicate); others return base64 payloads (OpenAI gpt-image-2). For the base64 case, use `save_b64_image()` — it writes to `$HERMES_HOME/cache/images/<prefix>_<timestamp>_<uuid>.<ext>` and returns the absolute `Path`. Pass that path (as `str`) as `image=` in `success_response()`. Puerta de enlace delivery (Telegram photo bubble, Discord attachment) recognizes both URLs and absolute paths.

## User overrides

Drop a user complemento at `~/.hermes/complementos/image_gen/<name>/` with the same `name` property as a bundled one and enable it via `hermes complementos enable <name>` — the registry is last-writer-wins, so your version replaces the built-in. Useful for pointing an `openai` complemento at a private proxy, or swapping in a custom modelo catalog.

## Testing

```bash
export HERMES_HOME=/tmp/hermes-imggen-test
mkdir -p $HERMES_HOME/complementos/image_gen/my-backend
# …copy __init__.py + complemento.yaml into that dir…

export MY_BACKEND_API_KEY=your-test-key
hermes complementos enable my-backend

# Pick it as the active proveedor
echo "image_gen:" >> $HERMES_HOME/config.yaml
echo "  proveedor: my-backend" >> $HERMES_HOME/config.yaml

# Exercise it
hermes -z "Generate an image of a corgi in a spacesuit"
```

Or interactively: `hermes herramientas` → "Image Generation" → select `my-backend` → enter API key if prompted.

## Referencia implementations

- **`complementos/image_gen/openai/__init__.py`** — gpt-image-2 at low/medium/high tiers as three virtual modelo IDs sharing one API modelo with different `quality` params. Good example of tiered modelos under a single backend + config.yaml precedence chain.
- **`complementos/image_gen/xai/__init__.py`** — Grok Imagine via xAI. Different shape (URL output, simpler catalog).
- **`complementos/image_gen/openai-codex/__init__.py`** — Codex-style Responses API variant reusing the OpenAI SDK with a different routing base URL.

## Distribute via pip

```toml
# pyproject.toml
[project.entry-points."hermes_agente.complementos"]
my-backend-imggen = "my_backend_imggen_package"
```

`my_backend_imggen_package` must expose a top-level `register` function. See [Distribute via pip](/guides/build-a-hermes-complemento#distribute-via-pip) in the general complemento guide for the full setup.

## Related pages

- [Image Generation](/guia-usuario/features/image-generation) — user-facing feature documentation
- [Complementos overview](/guia-usuario/features/complementos) — all complemento types at a glance
- [Build a Hermes Complemento](/guides/build-a-hermes-complemento) — general herramientas/hooks/slash commands guide

---