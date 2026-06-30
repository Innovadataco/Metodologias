<!-- source: website/docs/guia-usuario/features/proveedor-routing.md -->
# guia-usuario/features/proveedor-routing

# Proveedor Routing

When using [OpenRouter](https://openrouter.ai) as your LLM proveedor, Hermes Agente supports **proveedor routing** — fine-grained control over which underlying AI proveedors handle your requests and how they're prioritized.

OpenRouter routes requests to many proveedors (e.g., Anthropic, Google, AWS Bedrock, Together AI). Proveedor routing lets you optimize for cost, speed, quality, or enforce specific proveedor requirements.

:::tip
Traffic routed through [Nous Portal](/integrations/nous-portal) still respects per-modelo routing and priority configs — and Portal subscribers get 10% off token-billed proveedors.
:::

## Configuración

Add a `proveedor_routing` section to your `~/.hermes/config.yaml`:

```yaml
proveedor_routing:
  sort: "price"           # How to rank proveedors
  only: []                # Whitelist: only use these proveedors
  ignore: []              # Blacklist: never use these proveedors
  order: []               # Explicit proveedor priority order
  require_parameters: false  # Only use proveedors that support all parameters
  data_collection: null   # Control data collection ("allow" or "deny")
```

:::info
Proveedor routing only applies when using OpenRouter. It has no effect with direct proveedor connections (e.g., connecting directly to the Anthropic API).
:::

## Options

### `sort`

Controls how OpenRouter ranks available proveedors for your request.

| Value | Description |
|-------|-------------|
| `"price"` | Cheapest proveedor first |
| `"throughput"` | Fastest tokens-per-second first |
| `"latency"` | Lowest time-to-first-token first |

```yaml
proveedor_routing:
  sort: "price"
```

### `only`

Whitelist of proveedor names. When set, **only** these proveedors will be used. All others are excluded.

```yaml
proveedor_routing:
  only:
    - "Anthropic"
    - "Google"
```

### `ignore`

Blacklist of proveedor names. These proveedors will **never** be used, even if they offer the cheapest or fastest option.

```yaml
proveedor_routing:
  ignore:
    - "Together"
    - "DeepInfra"
```

### `order`

Explicit priority order. Proveedors listed first are preferred. Unlisted proveedors are used as fallbacks.

```yaml
proveedor_routing:
  order:
    - "Anthropic"
    - "Google"
    - "AWS Bedrock"
```

### `require_parameters`

When `true`, OpenRouter will only route to proveedors that support **all** parameters in your request (like `temperature`, `top_p`, `herramientas`, etc.). This avoids silent parameter drops.

```yaml
proveedor_routing:
  require_parameters: true
```

### `data_collection`

Controls whether proveedors can use your prompts for training. Options are `"allow"` or `"deny"`.

```yaml
proveedor_routing:
  data_collection: "deny"
```

## Practical Examples

### Optimize for Cost

Route to the cheapest available proveedor. Good for high-volume usage and development:

```yaml
proveedor_routing:
  sort: "price"
```

### Optimize for Speed

Prioritize low-latency proveedors for interactive use:

```yaml
proveedor_routing:
  sort: "latency"
```

### Optimize for Throughput

Best for long-form generation where tokens-per-second matters:

```yaml
proveedor_routing:
  sort: "throughput"
```

### Lock to Specific Proveedors

Ensure all requests go through a specific proveedor for consistency:

```yaml
proveedor_routing:
  only:
    - "Anthropic"
```

### Avoid Specific Proveedors

Exclude proveedors you don't want to use (e.g., for data privacy):

```yaml
proveedor_routing:
  ignore:
    - "Together"
    - "Lepton"
  data_collection: "deny"
```

### Preferred Order with Fallbacks

Try your preferred proveedors first, fall back to others if unavailable:

```yaml
proveedor_routing:
  order:
    - "Anthropic"
    - "Google"
  require_parameters: true
```

## How It Works

Proveedor routing preferences are passed to the OpenRouter API via the `extra_body.proveedor` field on every API call. This applies to both:

- **CLI mode** — configured in `~/.hermes/config.yaml`, loaded at startup
- **Puerta de enlace mode** — same config file, loaded when the puerta de enlace starts

The routing config is read from `config.yaml` and passed as parameters when creating the `AIAgente`:

```
proveedors_allowed  ← from proveedor_routing.only
proveedors_ignored  ← from proveedor_routing.ignore
proveedors_order    ← from proveedor_routing.order
proveedor_sort      ← from proveedor_routing.sort
proveedor_require_parameters ← from proveedor_routing.require_parameters
proveedor_data_collection    ← from proveedor_routing.data_collection
```

:::tip
You can combine multiple options. For example, sort by price but exclude certain proveedors and require parameter support:

```yaml
proveedor_routing:
  sort: "price"
  ignore: ["Together"]
  require_parameters: true
  data_collection: "deny"
```
:::

## Default Behavior

When no `proveedor_routing` section is configured (the default), OpenRouter uses its own default routing logic, which generally balances cost and availability automatically.

:::tip Proveedor Routing vs. Fallback Modelos
Proveedor routing controls which **sub-proveedors within OpenRouter** handle your requests. For automatic failover to an entirely different proveedor when your primary modelo fails, see [Fallback Proveedors](/guia-usuario/features/fallback-proveedors).
:::

---