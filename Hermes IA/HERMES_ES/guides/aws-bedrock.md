<!-- source: website/docs/guides/aws-bedrock.md -->
# AWS Bedrock

# AWS Bedrock

Hermes Agente supports Amazon Bedrock as a native proveedor using the **Converse API** — not the OpenAI-compatible endpoint. This gives you full access to the Bedrock ecosystem: IAM autenticación, Guardrails, cross-region inference perfils, and all foundation modelos.

## Prerequisites

- **AWS credentials** — any source supported by the [boto3 credential chain](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html):
  - IAM instance role (EC2, ECS, Lambda — zero config)
  - `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY` environment variables
  - `AWS_PROFILE` for SSO or named perfils
  - `aws configure` for local development
- **boto3** — install with `cd ~/.hermes/hermes-agente && uv pip install -e ".[bedrock]"`
- **IAM permissions** — at minimum:
  - `bedrock:InvokeModelo` and `bedrock:InvokeModeloWithResponseStream` (for inference)
  - `bedrock:ListFoundationModelos` and `bedrock:ListInferencePerfils` (for modelo discovery)

:::tip EC2 / ECS / Lambda
On AWS compute, attach an IAM role with `AmazonBedrockFullAccess` and you're done. No API keys, no `.env` configuración — Hermes detects the instance role automatically.
:::

## Quick Start

```bash
# Install with Bedrock support
cd ~/.hermes/hermes-agente && uv pip install -e ".[bedrock]"

# Select Bedrock as your proveedor
hermes modelo
# → Choose "More proveedors..." → "AWS Bedrock"
# → Select your region and modelo

# Start chatting
hermes chat
```

## Configuración

After running `hermes modelo`, your `~/.hermes/config.yaml` will contain:

```yaml
modelo:
  default: us.anthropic.claude-sonnet-4-6
  proveedor: bedrock
  base_url: https://bedrock-runtime.us-east-2.amazonaws.com

bedrock:
  region: us-east-2
```

### Region

Set the AWS region in any of these ways (highest priority first):

1. `bedrock.region` in `config.yaml`
2. `AWS_REGION` environment variable
3. `AWS_DEFAULT_REGION` environment variable
4. Default: `us-east-1`

### Guardrails

To apply [Amazon Bedrock Guardrails](https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html) to all modelo invocations:

```yaml
bedrock:
  region: us-east-2
  guardrail:
    guardrail_identifier: "abc123def456"  # From the Bedrock console
    guardrail_version: "1"                # Version number or "DRAFT"
    stream_processing_mode: "async"       # "sync" or "async"
    trace: "disabled"                     # "enabled", "disabled", or "enabled_full"
```

### Modelo Discovery

Hermes auto-discovers available modelos via the Bedrock control plane. You can customize discovery:

```yaml
bedrock:
  discovery:
    enabled: true
    proveedor_filter: ["anthropic", "amazon"]  # Only show these proveedors
    refresh_interval: 3600                     # Cache for 1 hour
```

## Available Modelos

Bedrock modelos use **inference perfil IDs** for on-demand invocation. The `hermes modelo` picker shows these automatically, with recommended modelos at the top:

| Modelo | ID | Notes |
|-------|-----|-------|
| Claude Sonnet 4.6 | `us.anthropic.claude-sonnet-4-6` | Recommended — best balance of speed and capability |
| Claude Opus 4.6 | `us.anthropic.claude-opus-4-6-v1` | Most capable |
| Claude Haiku 4.5 | `us.anthropic.claude-haiku-4-5-20251001-v1:0` | Fastest Claude |
| Amazon Nova Pro | `us.amazon.nova-pro-v1:0` | Amazon's flagship |
| Amazon Nova Micro | `us.amazon.nova-micro-v1:0` | Fastest, cheapest |
| DeepSeek V3.2 | `deepseek.v3.2` | Strong open modelo |
| Llama 4 Scout 17B | `us.meta.llama4-scout-17b-instruct-v1:0` | Meta's latest |

:::info Cross-Region Inference
Modelos prefixed with `us.` use cross-region inference perfils, which provide better capacity and automatic failover across AWS regions. Modelos prefixed with `global.` route across all available regions worldwide.
:::

## Switching Modelos Mid-Session

Use the `/modelo` command during a conversation:

```
/modelo us.amazon.nova-pro-v1:0
/modelo deepseek.v3.2
/modelo us.anthropic.claude-opus-4-6-v1
```

## Diagnostics

```bash
hermes doctor
```

The doctor checks:
- Whether AWS credentials are available (env vars, IAM role, SSO)
- Whether `boto3` is installed
- Whether the Bedrock API is reachable (ListFoundationModelos)
- Number of available modelos in your region

## Puerta de enlace (Messaging Platforms)

Bedrock works with all Hermes puerta de enlace platforms (Telegram, Discord, Slack, Feishu, etc.). Configure Bedrock as your proveedor, then start the puerta de enlace normally:

```bash
hermes puerta de enlace setup
hermes puerta de enlace start
```

The puerta de enlace reads `config.yaml` and uses the same Bedrock proveedor configuración.

## Troubleshooting

### "No API key found" / "No AWS credentials"

Hermes checks for credentials in this order:
1. `AWS_BEARER_TOKEN_BEDROCK`
2. `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`
3. `AWS_PROFILE`
4. EC2 instance metadata (IMDS)
5. ECS container credentials
6. Lambda execution role

If none are found, run `aws configure` or attach an IAM role to your compute instance.

### "Invocation of modelo ID ... with on-demand throughput isn't supported"

Use an **inference perfil ID** (prefixed with `us.` or `global.`) instead of the bare foundation modelo ID. For example:
- ❌ `anthropic.claude-sonnet-4-6`
- ✅ `us.anthropic.claude-sonnet-4-6`

### "ThrottlingException"

You've hit the Bedrock per-modelo rate limit. Hermes automatically retries with backoff. To increase limits, request a quota increase in the [AWS Service Quotas console](https://console.aws.amazon.com/servicequotas/).

## One-Click AWS Deployment

For a fully automated deployment on EC2 with CloudFormation:

**[sample-hermes-agente-on-aws-with-bedrock](https://github.com/JiaDe-Wu/sample-hermes-agente-on-aws-with-bedrock)** — creates VPC, IAM role, EC2 instance, and configures Bedrock automatically. Deploy in any region with one click.

---