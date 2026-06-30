<!-- source: website/docs/guides/minimax-oauth.md -->
# MiniMax OAuth

# MiniMax OAuth

Hermes Agente supports **MiniMax** through a browser-based OAuth login flow, using the same credentials as the [MiniMax portal](https://www.minimax.io). No API key or credit card is required — log in once and Hermes automatically refreshes your session.

The transport reuses the `anthropic_messages` adapter (MiniMax exposes an Anthropic Messages-compatible endpoint at `/anthropic`), so all existing herramienta-calling, streaming, and contexto features work without any adapter changes.

## Descripción General

| Item | Value |
|------|-------|
| Proveedor ID | `minimax-oauth` |
| Display name | MiniMax (OAuth) |
| Auth type | Browser OAuth (PKCE redirect flow) |
| Transport | Anthropic Messages-compatible (`anthropic_messages`) |
| Modelos | `MiniMax-M2.7`, `MiniMax-M2.7-highspeed` |
| Global endpoint | `https://api.minimax.io/anthropic` |
| China endpoint | `https://api.minimaxi.com/anthropic` |
| Requires env var | No (`MINIMAX_API_KEY` is **not** used for this proveedor) |

## Prerequisites

- Python 3.9+
- Hermes Agente installed
- A MiniMax account at [minimax.io](https://www.minimax.io) (global) or [minimaxi.com](https://www.minimaxi.com) (China)
- A browser available on the local machine (or use `--no-browser` for remote sessions)

## Quick Start

```bash
# Launch the proveedor and modelo picker
hermes modelo
# → Select "MiniMax (OAuth)" from the proveedor list
# → Hermes opens your browser to the MiniMax authorization page
# → Approve access in the browser
# → Select a modelo (MiniMax-M2.7 or MiniMax-M2.7-highspeed)
# → Start chatting

hermes
```

After the first login, credentials are stored under `~/.hermes/auth.json` and are refreshed automatically before each session.

## Logging In Manually

You can trigger a login without going through the modelo picker:

```bash
hermes auth add minimax-oauth
```

### China region

If your account is on the China platform (`minimaxi.com`), use the API-key-based `minimax-cn` proveedor instead — `minimax-cn` is registered with `auth_type="api_key"` only (no OAuth flow). Configure `MINIMAX_CN_API_KEY` (and optionally `MINIMAX_CN_BASE_URL`) directly:

```bash
echo 'MINIMAX_CN_API_KEY=your-key' >> ~/.hermes/.env
```

### Remote / headless sessions

On servers or containers where no browser is available:

```bash
hermes auth add minimax-oauth --no-browser
```

Hermes will print the verification URL and user code — open the URL on any device and enter the code when prompted.

## The OAuth Flow

Hermes implements a PKCE browser OAuth flow against the MiniMax OAuth endpoints:

1. Hermes generates a PKCE verifier / challenge pair and a random state value.
2. It POSTs to `{base_url}/oauth/code` with the challenge and receives a `user_code` and `verification_uri`.
3. Your browser opens `verification_uri`. If prompted, enter the `user_code`.
4. Hermes polls `{base_url}/oauth/token` until the token arrives (or the deadline passes).
5. Tokens (`access_token`, `refresh_token`, expiry) are saved to `~/.hermes/auth.json` under the `minimax-oauth` key.

Token refresh (standard OAuth `refresh_token` grant) runs automatically at each session start when the access token is within 60 seconds of expiry.

## Checking Login Status

```bash
hermes doctor
```

The `◆ Auth Proveedors` section will show:

```
✓ MiniMax OAuth  (logged in, region=global)
```

or, if not logged in:

```
⚠ MiniMax OAuth  (not logged in)
```

## Switching Modelos

```bash
hermes modelo
# → Select "MiniMax (OAuth)"
# → Pick from the modelo list
```

Or set the modelo directly:

```bash
hermes config set modelo.default MiniMax-M2.7
hermes config set modelo.proveedor minimax-oauth
```

## Configuración Reference

After login, `~/.hermes/config.yaml` will contain entries similar to:

```yaml
modelo:
  default: MiniMax-M2.7
  proveedor: minimax-oauth
  base_url: https://api.minimax.io/anthropic
```

### Region endpoints

| Proveedor id | Portal | Inference endpoint |
|-------------|--------|-------------------|
| `minimax-oauth` (global) | `https://api.minimax.io` | `https://api.minimax.io/anthropic` |
| `minimax-cn` (China) | `https://api.minimaxi.com` | `https://api.minimaxi.com/anthropic` |

### Proveedor aliases

All of the following resolve to `minimax-oauth`:

```bash
hermes --proveedor minimax-oauth    # canonical
hermes --proveedor minimax-portal   # alias
hermes --proveedor minimax-global   # alias
hermes --proveedor minimax_oauth    # alias (underscore form)
```

## Environment Variables

The `minimax-oauth` proveedor does **not** use `MINIMAX_API_KEY` or `MINIMAX_BASE_URL`. Those variables are for the API-key-based `minimax` and `minimax-cn` proveedors only.

| Variable | Effect |
|----------|--------|
| `MINIMAX_API_KEY` | Used by `minimax` proveedor only — ignored for `minimax-oauth` |
| `MINIMAX_CN_API_KEY` | Used by `minimax-cn` proveedor only — ignored for `minimax-oauth` |

To use `minimax-oauth` as the active proveedor, set `modelo.proveedor: minimax-oauth` in `config.yaml` (use `hermes setup` for the guided flow), or pass `--proveedor minimax-oauth` for a single invocation:

```bash
hermes --proveedor minimax-oauth
```

## Modelos

| Modelo | Best for |
|-------|----------|
| `MiniMax-M2.7` | Long-contexto reasoning, complex herramienta-calling |
| `MiniMax-M2.7-highspeed` | Lower latency, lighter tasks, auxiliary calls |

Both modelos support up to 200,000 tokens of contexto.

`MiniMax-M2.7-highspeed` is also used automatically as the auxiliary modelo for vision and delegation tasks when `minimax-oauth` is the primary proveedor.

## Troubleshooting

### Token expired — not re-logging in automatically

Hermes refreshes the token on every session start if it is within 60 seconds of expiry. If the access token is already expired (for example, after a long offline period), the refresh happens automatically on the next request. If refresh fails with `refresh_token_reused` or `invalid_grant`, Hermes marks the session as requiring re-login.

When the refresh failure is terminal (HTTP 4xx, `invalid_grant`, revoked grant, etc.), Hermes marks the refresh token as dead and quarantines it locally so it doesn't keep replaying the doomed exchange. The agente surfaces a single "re-autenticación required" message and stays out of the way until you log in again.

**Fix:** run `hermes auth add minimax-oauth` again to start a fresh login. The quarantine clears on the next successful exchange.

### Authorization timed out

The device-code flow has a finite expiry window. If you don't approve the login in time, Hermes raises a timeout error.

**Fix:** re-run `hermes auth add minimax-oauth` (or `hermes modelo`). The flow starts fresh.

### State mismatch (possible CSRF)

Hermes detected that the `state` value returned by the authorization server does not match what it sent.

**Fix:** re-run the login. If it persists, check for a proxy or redirect that is modifying the OAuth response.

### Logging in from a remote server

If `hermes` cannot open a browser window, use `--no-browser`:

```bash
hermes auth add minimax-oauth --no-browser
```

Hermes prints the URL and code. Open the URL on any device and complete the flow there.

### "Not logged into MiniMax OAuth" error at runtime

The auth store has no credentials for `minimax-oauth`. You have not logged in yet, or the credential file was deleted.

**Fix:** run `hermes modelo` and select MiniMax (OAuth), or run `hermes auth add minimax-oauth`.

## Logging Out

To remove stored MiniMax OAuth credentials:

```bash
hermes auth logout minimax-oauth
```

## See Also

- [AI Proveedors reference](../integrations/proveedors.md)
- [Environment Variables](../reference/environment-variables.md)
- [Configuración](../guia-usuario/configuración.md)
- [hermes doctor](../reference/cli-commands.md)

---