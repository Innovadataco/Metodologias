<!-- fuente: sitio web/docs/integrations/providers.md -->
# Proveedores de IA# Proveedores de IAEsta página cubre la configuración de proveedores de inferencia para Hermes Agent, desde API en la nube como OpenRouter y Anthropic, hasta puntos finales autohospedados como Ollama y vLLM, y configuraciones avanzadas de enrutamiento y respaldo. Necesita al menos un proveedor configurado para utilizar Hermes.## Proveedores de inferenciaNecesita al menos una forma de conectarse a un LLM. Utilice el "modelo hermes" para cambiar de proveedor y modelo de forma interactiva, o configúrelo directamente:| Proveedor | Configuración |
|----------|-------|
| **Nosotros Portal** | `modelo hermes` (OAuth, basado en suscripción) |
| **Códice OpenAI** | `modelo hermes` (ChatGPT OAuth, utiliza modelos Codex) |
| **Copiloto de GitHub** | `modelo hermes` (flujo de código de dispositivo OAuth, `COPILOT_GITHUB_TOKEN`, `GH_TOKEN` o `gh auth token`) |
| **ACP copiloto de GitHub** | `hermes model` (genera `copilot --acp --stdio` local) |
| **Antrópico** | `modelo hermes` (Claude Max + créditos de uso adicionales a través de OAuth; también admite clave API Anthropic o token de configuración manual; consulte la nota a continuación) |
| **Enrutador abierto** | `OPENROUTER_API_KEY` en `~/.hermes/.env` |
| **NovitaAI** | `NOVITA_API_KEY` en `~/.hermes/.env` (proveedor: `novita`, más de 200 modelos, Model API, Agent Sandbox, GPU Cloud) |
| **z.ai/GLM** | `GLM_API_KEY` en `~/.hermes/.env` (proveedor: `zai`) |
| **Kimi / Disparo lunar** | `KIMI_API_KEY` en `~/.hermes/.env` (proveedor: `kimi-coding`) |
| **Kimi / Disparo lunar (China)** | `KIMI_CN_API_KEY` en `~/.hermes/.env` (proveedor: `kimi-coding-cn`; alias: `kimi-cn`, `moonshot-cn`) |
| **Arcee AI** | `ARCEEAI_API_KEY` en `~/.hermes/.env` (proveedor: `arcee`; alias: `arcee-ai`, `arceeai`) |
| **Nube GMI** | `GMI_API_KEY` en `~/.hermes/.env` (proveedor: `gmi`; alias: `gmi-cloud`, `gmicloud`) |
| **MiniMax** | `MINIMAX_API_KEY` en `~/.hermes/.env` (proveedor: `minimax`) |
| **MiniMax China** | `MINIMAX_CN_API_KEY` en `~/.hermes/.env` (proveedor: `minimax-cn`) |
| **xAI (Grok) — API de respuestas** | `XAI_API_KEY` en `~/.hermes/.env` (proveedor: `xai`) |
| **xAI Grok OAuth (SuperGrok)** | `hermes model` → "xAI Grok OAuth (SuperGrok / Premium+)" — inicio de sesión en el navegador, sin clave API. Ver [guía](../guides/xai-grok-oauth.md) |
| **Nube Qwen (Alibaba DashScope)** | `DASHSCOPE_API_KEY` en `~/.hermes/.env` (proveedor: `alibaba`) |
| **Alibaba Cloud (plan de codificación)** | `DASHSCOPE_API_KEY` (proveedor: `alibaba-coding-plan`, alias: `alibaba_coding`) — SKU de facturación independiente, punto final diferente |
| **Código Kilo** | `KILOCODE_API_KEY` en `~/.hermes/.env` (proveedor: `kilocode`) |
| **Xiaomi MiMo** | `XIAOMI_API_KEY` en `~/.hermes/.env` (proveedor: `xiaomi`, alias: `mimo`, `xiaomi-mimo`) |
| **Tencent TokenHub** | `TOKENHUB_API_KEY` en `~/.hermes/.env` (proveedor: `tencent-tokenhub`, alias: `tencent`, `tokenhub`, `tencentmaas`) |
| **Zen de código abierto** | `OPENCODE_ZEN_API_KEY` en `~/.hermes/.env` (proveedor: `opencode-zen`) |
| **OpenCode Ir** | `OPENCODE_GO_API_KEY` en `~/.hermes/.env` (proveedor: `opencode-go`) |
| **Búsqueda profunda** | `DEEPSEEK_API_KEY` en `~/.hermes/.env` (proveedor: `deepseek`) |
| **Cara abrazada** | `HF_TOKEN` en `~/.hermes/.env` (proveedor: `huggingface`, alias: `hf`) |
| **Google / Géminis** | `GOOGLE_API_KEY` (o `GEMINI_API_KEY`) en `~/.hermes/.env` (proveedor: `gemini`) |
| **API OpenAI (directa)** | `OPENAI_API_KEY` en `~/.hermes/.env` (proveedor: `openai-api`, opcional `OPENAI_BASE_URL`) |
| **Fundición Azure AI** | `modelo hermes` → "Azure AI Foundry" (proveedor: `azure-foundry`; utiliza el punto final y la clave de Azure OpenAI / Foundry) |
| **Base de AWS** | `modelo hermes` → "AWS Bedrock" (proveedor: `bedrock`; cadena de credenciales estándar de AWS a través de boto3) |
| **Construcción NVIDIA** | `NVIDIA_API_KEY` en `~/.hermes/.env` (proveedor: `nvidia`; modelos alojados en NIM en build.nvidia.com) |
| **Nube de Ollama** | `modelo hermes` → "Ollama Cloud" (proveedor: `ollama-cloud`; API de Ollama alojada en la nube) |
| **Qwen OAuth** | `modelo hermes` → "Qwen OAuth" (proveedor: `qwen-oauth`; inicio de sesión PKCE del navegador) |
| **MiniMax OAuth** | `modelo hermes` → "MiniMax (OAuth)" (proveedor: `minimax-oauth`; inicio de sesión PKCE del navegador) |
| **PasoDivertido** | `STEPFUN_API_KEY` en `~/.hermes/.env` (proveedor: `stepfun`) |
| **Estudio LM** | `modelo hermes` → "LM Studio" (proveedor: `lmstudio`, opcional `LM_API_KEY`) |
| **Punto final personalizado** | `modelo hermes` → elija "Punto final personalizado" (guardado en `config.yaml`) |Para conocer la ruta de la clave API oficial, consulte la [guía de Google Gemini](/guides/google-gemini) dedicada.:::tip Alias de clave de modelo
En la sección de configuración `model:`, puede usar `default:` o `model:` como nombre de clave para su ID de modelo. Tanto `modelo: {predeterminado: mi-modelo}` como `modelo: {modelo: mi-modelo}` funcionan de manera idéntica.
:::
### Portal Nuestro[Nous Portal](https://portal.nousresearch.com) es el portal de suscripción unificado de Nous Research y **la forma recomendada de ejecutar Hermes Agent**. Un inicio de sesión de OAuth cubre más de 300 modelos agentes de frontera (Claude, GPT, Gemini, DeepSeek, Qwen, Kimi, GLM, MiniMax, Grok, ...) más [Tool Gateway](/user-guide/features/tool-gateway) (búsqueda web, generación de imágenes, TTS, automatización del navegador) más [Nous Chat](https://chat.nousresearch.com), facturado contra su suscripción Nous en lugar de por separado. cuentas por proveedor.```golpecito
configuración de hermes --portal # instalación nueva: OAuth + proveedor + puerta de enlace en un solo comando
modelo hermes # instalación existente: seleccione "Nous Portal" de la lista
información del portal de hermes # inspeccionar el inicio de sesión + enrutamiento en cualquier momento
```¿Aún no tienes una suscripción? Obtenga uno en [portal.nousresearch.com/manage-subscription](https://portal.nousresearch.com/manage-subscription).**Para obtener todos los detalles:** consulte la [página de integración de Nous Portal] dedicada (/integraciones/nous-portal) (qué incluye la suscripción, catálogo de modelos, solución de problemas) y la [guía Ejecutar Hermes Agent con Nous Portal] paso a paso (/guides/run-hermes-with-nous-portal).**Identificación del cliente.** Cada solicitud de Portal del Agente Hermes lleva una etiqueta `client=hermes-client-v<versión>` (por ejemplo, `client=hermes-client-v0.13.0`) autoalineada con su versión instalada. Esto se envía en todas las rutas del Portal (bucle de chat principal, llamadas auxiliares, resumen de compresión, extracción web) y permite que la telemetría del lado del Portal distinga el tráfico de Hermes de otros clientes. No se requiere configuración; la etiqueta se actualiza automáticamente cuando "actualiza Hermes".**Autenticación JWT (automática).** Hermes prefiere JWT de `inferencia:invocar` con alcance para solicitudes de Portal con la ruta de clave de sesión opaca heredada como alternativa. No se requiere configuración: las credenciales las administra el flujo de OAuth y rotan de forma transparente. Los tokens de actualización revocados se ponen en cuarentena para evitar bucles de repetición.
:::info Nota del Códice
El proveedor de OpenAI Codex se autentica mediante el código del dispositivo (abra una URL, ingrese un código). Hermes almacena las credenciales resultantes en su propio almacén de autenticación en `~/.hermes/auth.json` y puede importar las credenciales CLI del Codex existentes desde `~/.codex/auth.json` cuando estén presentes. No se requiere instalación de Codex CLI.Si la actualización de un token falla con un error de terminal (HTTP 4xx, `invalid_grant`, concesión revocada, etc.), Hermes marca el token de actualización como inactivo y deja de reproducirlo para que no vea una avalancha de fallas de autenticación idénticas. En su lugar, la siguiente solicitud muestra un mensaje de nueva autenticación escrito. Ejecute `hermes auth add codex-oauth` (o `hermes model` → OpenAI Codex) para iniciar un nuevo inicio de sesión con código de dispositivo; la cuarentena desaparece en el próximo intercambio exitoso.
::::::advertencia
Incluso cuando se utiliza Nous Portal, Codex o un punto final personalizado, algunas herramientas (visión, resumen web, MoA) utilizan un modelo "auxiliar" independiente. De forma predeterminada (`auxiliary.*.provider: "auto"`), Hermes enruta estas tareas a su **modelo de chat principal**, el mismo modelo que eligió en `hermes model`. Puede anular cada tarea individualmente para enrutarla a un modelo más económico/rápido (por ejemplo, Gemini Flash en OpenRouter); consulte [Modelos auxiliares](/user-guide/configuration#auxiliary-models).
::::::tip Nous Tool Gateway
Los suscriptores pagos de Nous Portal también obtienen acceso a **[Tool Gateway](/user-guide/features/tool-gateway)**: búsqueda web, generación de imágenes, TTS y automatización del navegador a través de su suscripción. No se necesitan claves API adicionales. En una instalación nueva, `hermes setup --portal` inicia sesión, configura a Nous como su proveedor y activa la puerta de enlace con un solo comando. Los usuarios existentes pueden habilitarlo desde el "modelo Hermes" o por herramienta desde "herramientas Hermes". Inspeccione la ruta en cualquier momento con "hermes portal info".
:::### Dos comandos para la gestión de modelosHermes tiene **dos** comandos modelo que sirven para diferentes propósitos:| Comando | Dónde correr | Qué hace |
|---------|-------------|--------------|
| **`modelo hermes`** | Tu terminal (fuera de cualquier sesión) | Asistente de configuración completo: agregue proveedores, ejecute OAuth, ingrese claves API, configure puntos finales |
| **`/modelo`** | Dentro de una sesión de chat de Hermes | Cambio rápido entre proveedores y modelos **ya configurados** |Si está intentando cambiar a un proveedor que aún no ha configurado (por ejemplo, solo tiene OpenRouter configurado y desea usar Anthropic), necesita `hermes model`, no `/model`. Primero salga de su sesión (`Ctrl+C` o `/quit`), ejecute `hermes model`, complete la configuración del proveedor y luego inicie una nueva sesión.
### Antrópico (Nativo)Utilice los modelos de Claude directamente a través de la API de Anthropic; no se necesita ningún proxy OpenRouter. Admite tres métodos de autenticación::::precaución Requiere créditos de "uso adicional" de Claude Max
Cuando se autentica a través de `hermes model` → Anthropic OAuth (o mediante `hermes auth add anthropic --type oauth`), Hermes enruta como Claude Code a su cuenta de Anthropic. **Solo funciona si tienes un plan Claude Max y has comprado créditos de uso adicionales.** Hermes no consume la asignación básica del plan Max (el uso incluido en Claude Code de forma predeterminada), solo los créditos adicionales/excedentes que hayas agregado en la parte superior. Los suscriptores de Claude Pro no pueden utilizar esta ruta.Si no tiene créditos adicionales Max +, use `ANTHROPIC_API_KEY` en su lugar: las solicitudes se facturan de pago por token según la organización de esa clave (precio API estándar, independiente de cualquier suscripción a Claude).
:::```golpecito
# Con una clave API (pago por token)
exportar ANTHROPIC_API_KEY=***
hermes chat --proveedor antrópico --modelo claude-sonnet-4-6# Preferido: autenticarse a través del `modelo hermes`
# Hermes utilizará la tienda de credenciales de Claude Code directamente cuando esté disponible
modelo hermes# Anulación manual con un token de configuración (alternativo/heredado)
exportar ANTHROPIC_TOKEN=*** # token de configuración o token OAuth manual
hermes chat --proveedor antrópico# Detección automática de credenciales de Claude Code (si ya usa Claude Code)
hermes chat --provider anthropic # lee los archivos de credenciales de Claude Code automáticamente
```Cuando elige Anthropic OAuth a través del `modelo hermes`, Hermes prefiere el propio almacén de credenciales de Claude Code a copiar el token en `~/.hermes/.env`. Eso mantiene actualizadas las credenciales de Claude.O configúrelo permanentemente:
```yaml
modelo:
  proveedor: "antrópico"
  predeterminado: "claude-sonnet-4-6"
```:::tip Alias
`--provider claude` y `--provider claude-code` también funcionan como abreviatura de `--provider anthropic`.
:::### Copiloto de GitHubHermes admite GitHub Copilot como proveedor de primera clase con dos modos:**`copilot` — API de Copilot directo** (recomendado). Utiliza su suscripción a GitHub Copilot para acceder a GPT-5.x, Claude, Gemini y otros modelos a través de la API de Copilot.```golpecito
hermes chat --proveedor copiloto --modelo gpt-5.4
```**Opciones de autenticación** (marcadas en este orden):1. Variable de entorno `COPILOT_GITHUB_TOKEN`
2. Variable de entorno `GH_TOKEN`
3. Variable de entorno `GITHUB_TOKEN`
4. Respaldo de la CLI `gh auth token`Si no se encuentra ningún token, el "modelo hermes" ofrece un **inicio de sesión con código de dispositivo OAuth**, el mismo flujo utilizado por la CLI de Copilot y el código abierto.:::tipos de tokens de advertencia
La API Copilot **no** admite tokens de acceso personal clásicos (`ghp_*`). Tipos de tokens admitidos:| Tipo | Prefijo | Cómo llegar |
|------|--------|------------|
| token de OAuth | `gho_` | `hermes model` → GitHub Copilot → Iniciar sesión con GitHub |
| PAT de grano fino | `github_pat_` | Configuración de GitHub → Configuración de desarrollador → Tokens detallados (necesita permiso **Solicitudes de copiloto**) |
| Ficha de aplicación GitHub | `ghu_` | A través de la instalación de la aplicación GitHub |Si su `gh auth token` devuelve un token `ghp_*`, utilice el `hermes model` para autenticarse a través de OAuth.
::::::info Comportamiento de autenticación del copiloto en Hermes
Hermes envía un token de GitHub compatible (`gho_*`, `github_pat_*` o `ghu_*`) directamente a `api.githubcopilot.com` e incluye encabezados específicos de Copilot (`Editor-Version`, `Copilot-Integration-Id`, `Openai-Intent`, `x-initiator`).En HTTP 401, Hermes ahora realiza una recuperación de credenciales de un solo paso antes del respaldo:1. Vuelva a resolver el token a través de la cadena de prioridad normal (`COPILOT_GITHUB_TOKEN` → `GH_TOKEN` → `GITHUB_TOKEN` → `gh auth token`)
2. Reconstruya el cliente OpenAI compartido con encabezados actualizados
3. Vuelva a intentar la solicitud una vezAlgunos servidores proxy comunitarios más antiguos utilizan flujos de intercambio `api.github.com/copilot_internal/v2/token`. Ese punto final puede no estar disponible para algunos tipos de cuentas (devuelve 404). Por lo tanto, Hermes mantiene la autenticación de token directo como ruta principal y se basa en la actualización y reintento de credenciales en tiempo de ejecución para mayor solidez.
:::**Enrutamiento API**: los modelos GPT-5+ (excepto `gpt-5-mini`) usan automáticamente la API de Respuestas. Todos los demás modelos (GPT-4o, Claude, Gemini, etc.) utilizan Finalizaciones de chat. Los modelos se detectan automáticamente desde el catálogo de Copilot en vivo.**`copilot-acp` — Backend del agente Copilot ACP**. Genera la CLI de Copilot local como un subproceso:```golpecito
hermes chat --proveedor copiloto-acp --modelo copiloto-acp
# Requiere la CLI de GitHub Copilot en PATH y una sesión de "inicio de sesión de copiloto" existente
```**Configuración permanente:**
```yaml
modelo:
  proveedor: "copiloto"
  predeterminado: "gpt-5.4"
```| Variable de entorno | Descripción |
|---------------------|-------------|
| `COPILOT_GITHUB_TOKEN` | Token de GitHub para API Copilot (primera prioridad) |
| `HERMES_COPILOT_ACP_COMMAND` | Anule la ruta binaria de la CLI de Copilot (predeterminado: `copilot`) |
| `HERMES_COPILOT_ACP_ARGS` | Anular argumentos de ACP (predeterminado: `--acp --stdio`) |### Proveedores de claves API de primera claseEstos proveedores tienen soporte integrado con ID de proveedor dedicados. Configure la clave API y use `--provider` para seleccionar:```golpecito
# API del modelo NovitaAI
hermes chat --proveedor novita --modelo moonshotai/kimi-k2.5
# Requiere: NOVITA_API_KEY en ~/.hermes/.env# z.ai / ZhipuAI GLM
hermes chat --proveedor zai --modelo glm-5
# Requiere: GLM_API_KEY en ~/.hermes/.env# Kimi / Moonshot AI (internacional: api.moonshot.ai)
hermes chat --proveedor kimi-coding --modelo kimi-for-coding
# Requiere: KIMI_API_KEY en ~/.hermes/.env# Kimi / Moonshot AI (China: api.moonshot.cn)
chat de hermes --proveedor kimi-coding-cn --modelo kimi-k2.5
# Requiere: KIMI_CN_API_KEY en ~/.hermes/.env# MiniMax (punto final global)
hermes chat --proveedor minimax --modelo MiniMax-M2.7
# Requiere: MINIMAX_API_KEY en ~/.hermes/.env# MiniMax (punto final de China)
hermes chat --proveedor minimax-cn --modelo MiniMax-M2.7
# Requiere: MINIMAX_CN_API_KEY en ~/.hermes/.env# Qwen Cloud / DashScope (modelos Qwen)
chat de hermes --proveedor alibaba --modelo qwen3.5-plus
# Requiere: DASHSCOPE_API_KEY en ~/.hermes/.env#Xiaomi MiMo
hermes chat --proveedor xiaomi --modelo mimo-v2-pro
# Requiere: XIAOMI_API_KEY en ~/.hermes/.env# Tencent TokenHub (vista previa de Hy3)
chat de hermes --proveedor tencent-tokenhub --modelo hy3-preview
# Requiere: TOKENHUB_API_KEY en ~/.hermes/.env# Arcee AI (modelos Trinity)
chat de hermes --proveedor arcee --modelo trinidad-pensamiento-grande
# Requiere: ARCEEAI_API_KEY en ~/.hermes/.env# Nube GMI
# Utilice el ID de modelo exacto devuelto por el punto final /v1/models de GMI.
chat de hermes --proveedor gmi --modelo zai-org/GLM-5.1-FP8
# Requiere: GMI_API_KEY en ~/.hermes/.env
```O configure el proveedor de forma permanente en `config.yaml`:
```yaml
modelo:
  proveedor: "gmi"
  predeterminado: "zai-org/GLM-5.1-FP8"
```Las URL base se pueden anular con variables de entorno `NOVITA_BASE_URL`, `GLM_BASE_URL`, `KIMI_BASE_URL`, `MINIMAX_BASE_URL`, `MINIMAX_CN_BASE_URL`, `DASHSCOPE_BASE_URL`, `XIAOMI_BASE_URL`, `GMI_BASE_URL` o `TOKENHUB_BASE_URL`.:::nota Detección automática de terminales Z.AI
Cuando se utiliza el proveedor Z.AI/GLM, Hermes explora automáticamente múltiples puntos finales (global, China, variantes de codificación) para encontrar uno que acepte su clave API. No es necesario configurar `GLM_BASE_URL` manualmente: el punto final en funcionamiento se detecta y se almacena en caché automáticamente.
:::### xAI (Grok) — API de respuestas + almacenamiento en caché de mensajesxAI está conectado a través de la API de Respuestas (transporte `codex_responses`) para soporte de razonamiento automático en los modelos Grok 4; no se necesita el parámetro `reasoning_effort`, el servidor razona de forma predeterminada. Establezca `XAI_API_KEY` en `~/.hermes/.env` y seleccione xAI en `hermes model`, o suelte `grok` como acceso directo a `/model grok-4-fast-reasoning`.Los suscriptores de SuperGrok y X Premium+ pueden iniciar sesión con el navegador OAuth en lugar de usar una clave API: seleccione **xAI Grok OAuth (SuperGrok / Premium+)** en `hermes model` o ejecute `hermes auth add xai-oauth`. El mismo token portador de OAuth se reutiliza automáticamente mediante herramientas directas a xAI (TTS, generación de imágenes, generación de videos, transcripción). Consulte la [guía de xAI Grok OAuth](../guides/xai-grok-oauth.md) para conocer el flujo completo; y si Hermes se ejecuta en un host remoto, consulte también [OAuth sobre SSH / Hosts remotos](../guides/oauth-over-ssh.md) para conocer el túnel `ssh -L` requerido.Cuando se utiliza xAI como proveedor (cualquier URL base que contenga `x.ai`), Hermes habilita automáticamente el almacenamiento en caché enviando el encabezado `x-grok-conv-id` con cada solicitud de API. Esto enruta las solicitudes al mismo servidor dentro de una sesión de conversación, lo que permite que la infraestructura de xAI reutilice las indicaciones del sistema almacenadas en caché y el historial de conversaciones.No se necesita configuración: el almacenamiento en caché se activa automáticamente cuando se detecta un punto final xAI y hay una ID de sesión disponible. Esto reduce la latencia y el costo de las conversaciones de varios turnos.xAI también incluye un punto final TTS dedicado (`/v1/tts`). Seleccione **xAI TTS** en `hermes tools` → Voz y TTS, o consulte la página [Voz y TTS](../user-guide/features/tts.md#text-to-speech) para la configuración.**Migración del modelo xAI retirada (15 de mayo de 2026):** xAI retirará `grok-4*`, `grok-3`, `grok-code-fast-1` y `grok-imagine-image-pro` el 15 de mayo de 2026. El inicio de `hermes doctor` y `hermes chat` detecta cualquier configuración que aún apunte a una referencia retirada e imprime el reemplazo recomendado. Utilice `hermes migrar xai` para una reescritura de configuración de una sola vez; ejecute en seco de forma predeterminada, agregue `--apply` para escribir los cambios (se crea automáticamente una copia de seguridad con marca de tiempo `config.yaml.bak-pre-migrate-xai-*`).```golpecito
Hermes migra xai # reemplazos de vista previa
hermes migra xai --apply # reescribe ~/.hermes/config.yaml en su lugar
```**backend de búsqueda web de xAI.** Cuando el conjunto de herramientas [Búsqueda web](../user-guide/features/web-search.md) está habilitado, `web.backend: xai` enruta la búsqueda a través del punto final de búsqueda alojado de xAI utilizando las mismas credenciales `XAI_API_KEY`/OAuth. No se requiere configuración adicional si xAI ya está configurado como proveedor.### Novita AI[NovitaAI](https://novita.ai) es la nube nativa de IA para constructores y agentes. Sus tres líneas de productos son Model API para más de 200 modelos, Agent Sandbox para crear y ejecutar agentes de IA y GPU Cloud para computación escalable, todos disponibles desde una sola plataforma.```golpecito
# Utilice cualquier modelo disponible
hermes chat --proveedor novita --modelo moonshotai/kimi-k2.5
# Requiere: NOVITA_API_KEY en ~/.hermes/.env# Alias corto
chat de hermes --proveedor novita-ai --modelo deepseek/deepseek-v3-0324
```O configúrelo permanentemente en `config.yaml`:
```yaml
modelo:
  proveedor: "novita"
  predeterminado: "moonshotai/kimi-k2.5"
  base_url: "https://api.novita.ai/openai/v1"
```Obtenga su clave API en [novita.ai/settings/key-management](https://novita.ai/settings/key-management). La URL base se puede anular con `NOVITA_BASE_URL`.### Ollama Cloud: modelos de Ollama administrados, OAuth + clave API[Ollama Cloud](https://ollama.com/cloud) aloja el mismo catálogo de peso abierto que Ollama local pero sin el requisito de GPU. Elíjalo en `hermes model` como **Ollama Cloud**, pegue su clave API de [ollama.com/settings/keys](https://ollama.com/settings/keys) y Hermes descubrirá automáticamente los modelos disponibles.```golpecito
modelo hermes
# → elige "Nube Ollama"
# → pega tu OLLAMA_API_KEY
# → seleccione entre los modelos descubiertos (gpt-oss:120b, glm-4.6:cloud, qwen3-coder:480b-cloud, etc.)
```O `config.yaml` directamente:
```yaml
modelo:
  proveedor: "ollama-cloud"
  predeterminado: "gpt-oss:120b"
```El catálogo de modelos se recupera dinámicamente de `ollama.com/v1/models` y se almacena en caché durante una hora. La notación `model:tag` (por ejemplo, `qwen3-coder:480b-cloud`) se conserva mediante la normalización; no utilice guiones.:::tip Ollama Cloud vs Ollama local
Ambos hablan la misma API compatible con OpenAI. Cloud es un proveedor de primera clase (`--provider ollama-cloud`, `OLLAMA_API_KEY`); Se llega a Ollama local a través del flujo de punto final personalizado (URL base `http://localhost:11434/v1`, sin clave). Utilice la nube para modelos grandes que no puede ejecutar localmente; use local para privacidad o trabajo fuera de línea.
:::### Base de AWSAnthropic Claude, Amazon Nova, DeepSeek v3.2, Meta Llama 4 y otros modelos a través de AWS Bedrock. Utiliza la cadena de credenciales AWS SDK (`boto3`): sin clave API, solo autenticación estándar de AWS.```golpecito
# Más simple: perfil con nombre en ~/.aws/credentials
hermes chat --proveedor bedrock --modelo us.anthropic.claude-sonnet-4-6# O con variables de entorno explícitas
AWS_PROFILE=miperfil AWS_REGION=us-east-1 chat de hermes --proveedor bedrock --modelo us.anthropic.claude-sonnet-4-6
```O permanentemente en `config.yaml`:
```yaml
modelo:
  proveedor: "base de roca"
  predeterminado: "us.anthropic.claude-sonnet-4-6"
base de roca:
  región: "us-east-1" # o establezca AWS_REGION
  # perfil: "miperfil" # o establece AWS_PROFILE
  # descubrimiento: verdadero # región de descubrimiento automático de IAM
  # barandilla: # barandillas Bedrock opcionales
  # guardrail_identifier: "tu-id-de-guardrail"
  # guardrail_version: "BORRADOR"
```La autenticación utiliza la cadena boto3 estándar: `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`, `AWS_PROFILE` de `~/.aws/credentials` explícito, función de IAM en EC2/ECS/Lambda, IMDS o SSO. No se requiere env var si ya está autenticado con la AWS CLI.Bedrock utiliza la **API de Converse** internamente: las solicitudes se traducen a la forma independiente del modelo de Bedrock, por lo que la misma configuración funciona para los modelos Claude, Nova, DeepSeek y Llama. Configure `BEDROCK_BASE_URL` solo si llama a un punto final regional no predeterminado.Consulte la [guía de AWS Bedrock](/guides/aws-bedrock) para obtener un tutorial sobre la configuración de IAM, la selección de regiones y la inferencia entre regiones.### Portal Qwen (OAuth)Portal Qwen de Alibaba con inicio de sesión OAuth basado en navegador. Elija **Qwen OAuth (Portal)** en "hermes model", inicie sesión a través del navegador y Hermes conservará el token de actualización.```golpecito
modelo hermes
# → seleccione "Qwen OAuth (Portal)"
# → se abre el navegador; inicia sesión con tu cuenta de Alibaba
# → confirmar: las credenciales se guardan en ~/.hermes/auth.jsonHermes chat # utiliza el punto final portal.qwen.ai/v1
```O configure `config.yaml`:
```yaml
modelo:
  proveedor: "qwen-oauth"
  predeterminado: "qwen3-coder-plus"
```Configure `HERMES_QWEN_BASE_URL` solo si el punto final del portal se reubica (predeterminado: `https://portal.qwen.ai/v1`).:::consejo Qwen OAuth frente a Qwen Cloud (Alibaba DashScope)
`qwen-oauth` utiliza el Portal Qwen orientado al consumidor con inicio de sesión OAuth, ideal para usuarios individuales. El proveedor `alibaba` utiliza Qwen Cloud (Alibaba DashScope) con `DASHSCOPE_API_KEY`, ideal para cargas de trabajo programáticas/de producción. Ambos se dirigen a modelos de la familia Qwen pero viven en puntos finales diferentes.
:::### Alibaba Cloud (plan de codificación)Si está suscrito al **Plan de codificación** de Alibaba (un SKU de precios independiente del acceso estándar a la API de DashScope), Hermes lo expone como su propio proveedor de primera clase: "alibaba-coding-plan". Punto final: `https://coding-intl.dashscope.aliyuncs.com/v1`. Es compatible con OpenAI como el proveedor habitual "alibaba", pero con una URL base y una superficie de facturación diferentes.```yaml
modelo:
  proveedor: alibaba_coding # alias para alibaba-coding-plan
  modelo: qwen3-coder-plus
```O desde la CLI:```golpecito
chat de hermes --proveedor alibaba_coding --modelo qwen3-coder-plus
````alibaba_coding` usa la misma `DASHSCOPE_API_KEY` que ya usa su entrada `alibaba`; no se necesita una clave separada, solo un destino de enrutamiento diferente. Antes de que se registrara este proveedor, los usuarios que configuraban `provider: alibaba_coding` en `config.yaml` silenciosamente pasaban al enrutamiento de OpenRouter.### MiniMax (OAuth)MiniMax-M2.7 a través del inicio de sesión OAuth del navegador: no se necesita clave API. Elija **MiniMax (OAuth)** en "hermes model", inicie sesión a través del navegador y Hermes conservará los tokens de acceso y actualización. Utiliza el punto final compatible con Anthropic Messages (`/anthropic`) oculto.```golpecito
modelo hermes
# → seleccione "MiniMax (OAuth)"
# → se abre el navegador; inicie sesión con su cuenta MiniMax (global o región CN)
# → confirmar: las credenciales se guardan en ~/.hermes/auth.jsonHermes chat # utiliza api.minimax.io/anthropic punto final
```O configure `config.yaml`:
```yaml
modelo:
  proveedor: "minimax-oauth"
  predeterminado: "MiniMax-M2.7"
```Modelos compatibles: `MiniMax-M2.7` (principal) y `MiniMax-M2.7-highspeed` (cableado como modelo auxiliar predeterminado). La ruta OAuth ignora `MINIMAX_API_KEY`/`MINIMAX_BASE_URL`.:::consejo MiniMax OAuth frente a clave API
`minimax-oauth` utiliza el portal orientado al consumidor de MiniMax con inicio de sesión OAuth, sin necesidad de configuración de facturación. Los proveedores `minimax` y `minimax-cn` usan `MINIMAX_API_KEY` / `MINIMAX_CN_API_KEY` — para acceso programático. Consulte la [guía MiniMax OAuth](/guides/minimax-oauth) para obtener un tutorial completo.
:::### NVIDIA NIMNemotron y otros modelos de código abierto a través de [build.nvidia.com](https://build.nvidia.com) (clave API gratuita) o un punto final NIM local.```golpecito
# Nube (build.nvidia.com)
hermes chat --proveedor nvidia --modelo nvidia/nemotron-3-super-120b-a12b
# Requiere: NVIDIA_API_KEY en ~/.hermes/.env# Punto final NIM local: anular la URL base
NVIDIA_BASE_URL=http://localhost:8000/v1 hermes chat --proveedor nvidia --modelo nvidia/nemotron-3-super-120b-a12b
```O configúrelo permanentemente en `config.yaml`:
```yaml
modelo:
  proveedor: "nvidia"
  predeterminado: "nvidia/nemotron-3-super-120b-a12b"
```:::sugerencia NIM local
Para implementaciones locales (DGX Spark, GPU local), configure `NVIDIA_BASE_URL=http://localhost:8000/v1`. NIM expone la misma API de finalización de chat compatible con OpenAI que build.nvidia.com, por lo que cambiar entre la nube y local es un cambio de env-var de una sola línea.
:::Hermes adjunta automáticamente el encabezado de origen de facturación de NIM en cada solicitud a `build.nvidia.com`, sin necesidad de configuración. Esto dirige el consumo hacia el origen correcto en el panel de facturación de NVIDIA.### Nube GMIModelos abiertos y de razonamiento a través de [GMI Cloud](https://www.gmicloud.ai/): API compatible con OpenAI, autenticación de clave API.```golpecito
# Nube GMI
chat de hermes --proveedor gmi --modelo deepseek-ai/DeepSeek-V3.2
# Requiere: GMI_API_KEY en ~/.hermes/.env
```O configúrelo permanentemente en `config.yaml`:
```yaml
modelo:
  proveedor: "gmi"
  predeterminado: "deepseek-ai/DeepSeek-V3.2"
```La URL base se puede anular con `GMI_BASE_URL` (predeterminado: `https://api.gmi-serving.com/v1`).### Paso divertidoModelos de la serie Step a través de [StepFun](https://platform.stepfun.com): API compatible con OpenAI, autenticación de clave API.```golpecito
# Paso divertido
hermes chat --proveedor stepfun --modelo paso-3.5-flash
# Requiere: STEPFUN_API_KEY en ~/.hermes/.env
```O configúrelo permanentemente en `config.yaml`:
```yaml
modelo:
  proveedor: "stepfun"
  predeterminado: "paso-3.5-flash"
```La URL base se puede anular con `STEPFUN_BASE_URL` (predeterminado: `https://api.stepfun.com/v1`).### Proveedores de inferencias de caras abrazadas[Proveedores de inferencia de Hugging Face](https://huggingface.co/docs/inference-providers) enruta a más de 20 modelos abiertos a través de un punto final unificado compatible con OpenAI (`router.huggingface.co/v1`). Las solicitudes se enrutan automáticamente al backend más rápido disponible (Groq, Together, SambaNova, etc.) con conmutación por error automática.```golpecito
# Utilice cualquier modelo disponible
hermes chat --proveedor abrazando cara --modelo Qwen/Qwen3.5-397B-A17B
# Requiere: HF_TOKEN en ~/.hermes/.env# Alias corto
chat de hermes --proveedor hf --modelo deepseek-ai/DeepSeek-V3.2
```O configúrelo permanentemente en `config.yaml`:
```yaml
modelo:
  proveedor: "cara de abrazo"
  predeterminado: "Qwen/Qwen3.5-397B-A17B"
```Obtenga su token en [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens): asegúrese de habilitar el permiso "Realizar llamadas a proveedores de inferencia". Nivel gratuito incluido (crédito de $0,10/mes, sin recargo en las tarifas del proveedor).Puede agregar sufijos de enrutamiento a los nombres de los modelos: `:fastest` (predeterminado), `:cheapest` o `:provider_name` para forzar un backend específico.La URL base se puede anular con `HF_BASE_URL`.## Proveedores de LLM personalizados y autohospedadosHermes Agent funciona con **cualquier punto final API compatible con OpenAI**. Si un servidor implementa `/v1/chat/completions`, puedes señalarlo con Hermes. Esto significa que puede utilizar modelos locales, servidores de inferencia de GPU, enrutadores de múltiples proveedores o cualquier API de terceros.### Configuración generalTres formas de configurar un punto final personalizado:**Configuración interactiva (recomendada):**
```golpecito
modelo hermes
# Seleccione "Punto final personalizado (autohospedado/VLLM/etc.)"
# Ingrese: URL base de API, clave de API, nombre del modelo
```**Configuración manual (`config.yaml`):**
```yaml
# En ~/.hermes/config.yaml
modelo:
  predeterminado: nombre-de-su-modelo
  proveedor: personalizado
  URL_base: http://localhost:8000/v1
  api_key: tu-clave-o-déjala-vacía-para-local
```:::advertencia de variables de entorno heredadas
`LLM_MODEL` en `.env` está **eliminado**: `config.yaml` es la única fuente de verdad para la configuración del modelo y del punto final. `OPENAI_BASE_URL` todavía se respeta, pero **sólo** para el proveedor `openai-api` (anula el punto final de OpenAI para el acceso directo a la clave API). Para otros proveedores y puntos finales personalizados, use `hermes model` o configure `model.base_url` en `config.yaml` directamente. Si tiene entradas obsoletas en su `.env`, se borran automáticamente en la siguiente `configuración de Hermes` o migración de configuración.
:::Ambos enfoques persisten en `config.yaml`, que es la fuente de verdad para el modelo, el proveedor y la URL base.### Cambiando de modelo con `/model`:::advertencia modelo hermes vs /modelo
**`hermes model`** (ejecutado desde tu terminal, fuera de cualquier sesión de chat) es el **asistente de configuración completo del proveedor**. Úselo para agregar nuevos proveedores, ejecutar flujos de OAuth, ingresar claves API y configurar puntos finales personalizados.**`/model`** (escrito dentro de una sesión de chat activa de Hermes) solo puede **cambiar entre proveedores y modelos que ya haya configurado**. No puede agregar nuevos proveedores, ejecutar OAuth ni solicitar claves API. Si solo ha configurado un proveedor (por ejemplo, OpenRouter), `/model` solo mostrará modelos para ese proveedor.**Para agregar un nuevo proveedor:** Salga de su sesión (`Ctrl+C` o `/quit`), ejecute `hermes model`, configure el nuevo proveedor y luego inicie una nueva sesión.
:::Una vez que haya configurado al menos un punto final personalizado, puede cambiar de modelo a mitad de sesión:```
/model custom:qwen-2.5 # Cambia a un modelo en tu punto final personalizado
/model custom # Detecta automáticamente el modelo desde el punto final
/model openrouter:claude-sonnet-4 # Volver a un proveedor de nube
```Si tiene **proveedores personalizados con nombre** configurados (consulte a continuación), utilice la sintaxis triple:```
/model custom:local:qwen-2.5 # Utilice el proveedor personalizado "local" con el modelo qwen-2.5
/model custom:work:llama3 # Usa el proveedor personalizado "work" con llama3
```Al cambiar de proveedor, Hermes conserva la URL base y el proveedor para configurar para que el cambio sobreviva al reinicio. Al cambiar de un punto final personalizado a un proveedor integrado, la URL base obsoleta se borra automáticamente.:::consejo
`/model custom` (simple, sin nombre de modelo) consulta la API `/models` de tu terminal y selecciona automáticamente el modelo si hay exactamente uno cargado. Útil para servidores locales que ejecutan un solo modelo.
:::Todo lo siguiente sigue este mismo patrón: simplemente cambie la URL, la clave y el nombre del modelo.---### Ollama: modelos locales, configuración cero[Ollama](https://ollama.com/) ejecuta modelos de peso abierto localmente con un solo comando. Ideal para: experimentación local rápida, trabajo sensible a la privacidad y uso sin conexión. Admite llamadas de herramientas a través de la API compatible con OpenAI.```golpecito
# Instalar y ejecutar un modelo
ollama tire qwen2.5-coder:32b
servicio de ollama # Comienza en el puerto 11434
```Luego configura Hermes:```golpecito
modelo hermes
# Seleccione "Punto final personalizado (autohospedado/VLLM/etc.)"
# Ingrese la URL: http://localhost:11434/v1
# Omitir clave API (Ollama no necesita una)
# Ingrese el nombre del modelo (por ejemplo, qwen2.5-coder:32b)
```O configure `config.yaml` directamente:```yaml
modelo:
  predeterminado: qwen2.5-codificador: 32b
  proveedor: personalizado
  URL_base: http://localhost:11434/v1
  context_length: 64000 # Ver advertencia a continuación
```:::precaución Ollama utiliza por defecto longitudes de contexto muy bajas
Ollama **no** usa la ventana de contexto completa de su modelo de forma predeterminada. Dependiendo de su VRAM, el valor predeterminado es:| VRAM disponible | Contexto predeterminado |
|----------------|----------------|
| Menos de 24 GB | **4.096 fichas** |
| 24–48 GB | 32.768 fichas |
| 48+GB | 256.000 fichas |Hermes Agent requiere al menos **64 000 tokens** de contexto para el uso del agente con herramientas. Las ventanas más pequeñas se rechazan al inicio porque el indicador del sistema, los esquemas de herramientas y el estado de la conversación de trabajo necesitan suficiente espacio para flujos de trabajo confiables de varios pasos.**Cómo aumentarlo** (elige uno):```golpecito
# Opción 1: configurar en todo el servidor mediante una variable de entorno (recomendado)
OLLAMA_CONTEXT_LENGTH=64000 servicio de ollama# Opción 2: Para Ollama administrado por systemd
sudo systemctl editar ollama.servicio
# Agregar: Entorno="OLLAMA_CONTEXT_LENGTH=64000"
# Entonces: sudo systemctl daemon-reload && sudo systemctl restart ollama# Opción 3: convertirlo en un modelo personalizado (persistente por modelo)
echo -e "DESDE qwen2.5-coder:32b\nPARAMETER num_ctx 64000" > Archivo de modelo
ollama crea qwen2.5-coder-64k -f Modelfile
```**No puede establecer la longitud del contexto a través de la API compatible con OpenAI** (`/v1/chat/completions`). Debe configurarse en el lado del servidor o mediante un Modelfile. Esta es la fuente número uno de confusión al integrar Ollama con herramientas como Hermes.
:::**Verifique que su contexto esté configurado correctamente:**```golpecito
ollama ps
# Mire la columna CONTEXTO: debería mostrar su valor configurado
```:::consejo
Listar modelos disponibles con `lista ollama`. Extraiga cualquier modelo de la [biblioteca de Ollama] (https://ollama.com/library) con `ollama pull <model>`. Ollama maneja la descarga de GPU automáticamente; no se necesita configuración para la mayoría de las configuraciones.
:::---### vLLM: inferencia de GPU de alto rendimiento[vLLM](https://docs.vllm.ai/) es el estándar para la entrega de LLM de producción. Ideal para: rendimiento máximo en hardware GPU, servicio a modelos grandes y procesamiento por lotes continuo.```golpecito
instalación de pip vllm
vllm sirve meta-llama/Llama-3.1-70B-Instruct\
  --puerto 8000\
  --max-modelo-len 65536 \
  --tensor-paralelo-tamaño 2 \
  --enable-elección-de-herramienta-automática \
  --tool-call-parser hermes
```Luego configura Hermes:```golpecito
modelo hermes
# Seleccione "Punto final personalizado (autohospedado/VLLM/etc.)"
# Ingrese la URL: http://localhost:8000/v1
# Omitir la clave API (o ingresar una si configuró vLLM con --api-key)
# Ingrese el nombre del modelo: meta-llama/Llama-3.1-70B-Instruct
```**Longitud del contexto:** vLLM lee `max_position_embeddings` del modelo de forma predeterminada. Si eso excede la memoria de su GPU, genera un error y le pide que establezca `--max-model-len` en un valor inferior. También puedes usar `--max-model-len auto` para encontrar automáticamente el máximo que se ajuste. Configure `--gpu-memory-utilization 0.95` (predeterminado 0.9) para incluir más contexto en la VRAM.**La llamada a herramientas requiere indicadores explícitos:**| Bandera | Propósito |
|------|---------|
| `--enable-auto-tool-choice` | Requerido para `tool_choice: "auto"` (el valor predeterminado en Hermes) |
| `--tool-call-parser <nombre>` | Analizador del formato de llamada a la herramienta del modelo |Analizadores compatibles: `hermes` (Qwen 2.5, Hermes 2/3), `llama3_json` (Llama 3.x), `mistral`, `deepseek_v3`, `deepseek_v31`, `xlam`, `pythonic`. Sin estos indicadores, las llamadas a herramientas no funcionarán: el modelo generará las llamadas a herramientas como texto.**Analizadores de razonamiento Qwen:** Hermes conserva metadatos de razonamiento estructurado como "razonamiento", "reasoning_content" y deltas de razonamiento transmitidos cuando los servidores compatibles con OpenAI los devuelven. Esos metadatos se tratan como datos de rastreo de razonamiento/pensamiento, no como un reemplazo de la respuesta visible del asistente. Para los modelos de razonamiento Qwen proporcionados por vLLM, asegúrese de que la respuesta final visible para el usuario todavía aparezca en el "contenido". Si `--reasoning-parser qwen3` deja `content` vacío en su implementación, desactive ese analizador o pase una opción de solicitud admitida por el servidor como `chat_template_kwargs.enable_thinking: false` a través de `extra_body`.:::consejo
vLLM admite tamaños legibles por humanos: `--max-model-len 64k` (k minúscula = 1000, K mayúscula = 1024).
:::---### SGLang: servicio rápido con RadixAttention[SGLang](https://github.com/sgl-project/sglang) es una alternativa a vLLM con RadixAttention para la reutilización de caché KV. Ideal para: conversaciones de varios turnos (almacenamiento en caché de prefijos), decodificación restringida, salida estructurada.```golpecito
instalación de pip "sglang[todos]"
python -m sglang.launch_server \
  --modelo meta-llama/Llama-3.1-70B-Instruct\
  --puerto 30000\
  --longitud del contexto 65536 \
  --tp2\
  --herramienta-llamada-parser qwen
```Luego configura Hermes:```golpecito
modelo hermes
# Seleccione "Punto final personalizado (autohospedado/VLLM/etc.)"
# Ingrese la URL: http://localhost:30000/v1
# Ingrese el nombre del modelo: meta-llama/Llama-3.1-70B-Instruct
```**Longitud del contexto:** SGLang lee desde la configuración del modelo de forma predeterminada. Utilice `--context-length` para anular. Si necesita exceder el máximo declarado del modelo, establezca `SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN=1`.**Llamada de herramientas:** Utilice `--tool-call-parser` con el analizador apropiado para su familia de modelos: `qwen` (Qwen 2.5), `llama3`, `llama4`, `deepseekv3`, `mistral`, `glm`. Sin esta marca, las llamadas a herramientas se devuelven como texto sin formato.:::precaución SGLang tiene como valor predeterminado 128 tokens de salida máximos
Si las respuestas parecen truncadas, agregue `max_tokens` a sus solicitudes o configure `--default-max-tokens` en el servidor. El valor predeterminado de SGLang es solo 128 tokens por respuesta si no se especifica en la solicitud.
:::---### llama.cpp / llama-server — CPU e inferencia de metales[llama.cpp](https://github.com/ggml-org/llama.cpp) ejecuta modelos cuantificados en CPU, Apple Silicon (Metal) y GPU de consumo. Ideal para: ejecutar modelos sin GPU de centro de datos, usuarios de Mac e implementación perimetral.```golpecito
# Construir e iniciar llama-server
cmake -B build && cmake --build build --config Lanzamiento
./build/bin/llama-servidor \
  --jinja-fa\
  -c 64000\
  -ingl 99\
  -m modelos/qwen2.5-coder-32b-instruct-Q4_K_M.gguf \
  --puerto 8080 --host 0.0.0.0
```**Longitud del contexto (`-c`):** Las compilaciones recientes tienen por defecto `0`, que lee el contexto de entrenamiento del modelo a partir de los metadatos de GGUF. Para modelos con un contexto de entrenamiento de más de 128k, esto puede OOM intentar asignar el caché KV completo. Establezca `-c` explícitamente en al menos 64.000 tokens para Hermes. Si se utilizan ranuras paralelas (`-np`), el contexto total se divide entre las ranuras; con `-c 64000 -np 4`, cada ranura solo obtiene 16k, lo que está por debajo del mínimo de Hermes por sesión activa.Luego configure Hermes para que apunte a él:```golpecito
modelo hermes
# Seleccione "Punto final personalizado (autohospedado/VLLM/etc.)"
# Ingrese la URL: http://localhost:8080/v1
# Omitir clave API (los servidores locales no la necesitan)
# Ingrese el nombre del modelo o déjelo en blanco para detectar automáticamente si solo hay un modelo cargado
```Esto guarda el punto final en `config.yaml` para que persista en todas las sesiones.:::Se requiere precaución `--jinja` para llamar a la herramienta
Sin `--jinja`, llama-server ignora por completo el parámetro `tools`. El modelo intentará llamar a las herramientas escribiendo JSON en su texto de respuesta, pero Hermes no lo reconocerá como una llamada a la herramienta; verá JSON sin formato como `{"name": "web_search", ...}` impreso como un mensaje en lugar de una búsqueda real.Soporte de llamadas de herramientas nativas (mejor rendimiento): Llama 3.x, Qwen 2.5 (incluido Coder), Hermes 2/3, Mistral, DeepSeek, Functionary. Todos los demás modelos utilizan un controlador genérico que funciona pero puede ser menos eficiente. Consulte los [documentos de llamada de función llama.cpp] (https://github.com/ggml-org/llama.cpp/blob/master/docs/function-calling.md) para obtener la lista completa.Puede verificar que el soporte de la herramienta esté activo marcando `http://localhost:8080/props`; el campo `chat_template` debe estar presente.
::::::consejo
Descargue modelos GGUF de [Hugging Face](https://huggingface.co/models?library=gguf). La cuantificación Q4_K_M ofrece el mejor equilibrio entre calidad y uso de memoria.
:::---### LM Studio: aplicación de escritorio con modelos locales[LM Studio](https://lmstudio.ai/) es una aplicación de escritorio para ejecutar modelos locales con una GUI. Ideal para: usuarios que prefieren una interfaz visual, pruebas rápidas de modelos, desarrolladores en macOS/Windows/Linux.Inicie el servidor desde la aplicación LM Studio (pestaña Desarrollador → Iniciar servidor) o use la CLI:```golpecito
Inicio del servidor lms # Comienza en el puerto 1234
lms carga qwen2.5-coder --context-length 64000
```Luego configura Hermes:```golpecito
modelo hermes
# Seleccione "Estudio LM"
# Presione Enter para usar http://localhost:1234/v1
# Elige uno de los modelos descubiertos.
# Si la autenticación del servidor LM Studio está habilitada, ingrese LM_API_KEY cuando se le solicite
```Hermes cargará automáticamente un modelo de LM Studio con una longitud de contexto de 64KPara cambiar la longitud del contexto en LM Studio:1. Haga clic en el ícono de ajustes al lado del selector de modelo.
2. Establezca "Longitud del contexto" en al menos 64000 para una experiencia fluida
3. Vuelva a cargar el modelo para que el cambio surta efecto.
4. Si su máquina no puede caber en 64000, considere usar un modelo más pequeño con longitudes de contexto más grandes.Alternativamente, use la CLI: `lms load model-name --context-length 64000`Puede utilizar la CLI para estimar si el modelo se ajustará: `lms load model-name --context-length 64000 --estimate-only`Para establecer valores predeterminados persistentes por modelo: pestaña Mis modelos → icono de engranaje en el modelo → establecer el tamaño del contexto.
:::**Llamada de herramientas:** Compatible desde LM Studio 0.3.6. Los modelos con entrenamiento nativo de llamada de herramientas (Qwen 2.5, Llama 3.x, Mistral, Hermes) se detectan automáticamente y se muestran con una insignia de herramienta. Otros modelos utilizan un respaldo genérico que puede ser menos confiable.---### Redes WSL2 (usuarios de Windows)Dado que Hermes Agent requiere un entorno Unix, los usuarios de Windows lo ejecutan dentro de WSL2. Si su servidor modelo (Ollama, LM Studio, etc.) se ejecuta en el **host de Windows**, necesita cerrar la brecha de la red: WSL2 usa un adaptador de red virtual con su propia subred, por lo que "localhost" dentro de WSL2 se refiere a la máquina virtual de Linux, **no** al host de Windows.:::consejo ¿Ambos en WSL2? Ningún problema.
Si su servidor modelo también se ejecuta dentro de WSL2 (común para vLLM, SGLang y llama-server), "localhost" funciona como se esperaba: comparten el mismo espacio de nombres de red. Salta esta sección.
:::#### Opción 1: Modo de red reflejada (recomendado)Disponible en **Windows 11 22H2+**, el modo reflejado hace que "localhost" funcione bidireccionalmente entre Windows y WSL2: la solución más sencilla.1. Cree o edite `%USERPROFILE%\.wslconfig` (por ejemplo, `C:\Users\YourName\.wslconfig`):
   ```ini
   [wsl2]
   modo de red=reflejado
   ```2. Reinicie WSL desde PowerShell:
   ```powershell
   wsl --apagar
   ```3. Vuelva a abrir su terminal WSL2. `localhost` ahora llega a los servicios de Windows:
   ```golpecito
   curl http://localhost:11434/v1/models # Ollama en Windows: funciona
   ```:::nota Firewall Hyper-V
En algunas compilaciones de Windows 11, el firewall Hyper-V bloquea las conexiones reflejadas de forma predeterminada. Si `localhost` aún no funciona después de habilitar el modo reflejado, ejecute esto en un **Admin PowerShell**:
```powershell
Set-NetFirewallHyperVVMSetting -Nombre '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}' -DefaultInboundAction Permitir
```
:::#### Opción 2: usar la IP del host de Windows (Windows 10 o versiones anteriores)Si no puede usar el modo reflejado, busque la IP del host de Windows desde WSL2 y úsela en lugar de `localhost`:```golpecito
# Obtener la IP del host de Windows (la puerta de enlace predeterminada de la red virtual de WSL2)
mostrar ruta ip | grep -i predeterminado | awk '{ imprimir $3 }'
# Salida de ejemplo: 172.29.192.1
```Utilice esa IP en su configuración de Hermes:```yaml
modelo:
  predeterminado: qwen2.5-codificador: 32b
  proveedor: personalizado
  base_url: http://172.29.192.1:11434/v1 # IP del host de Windows, no del host local
```:::tip Ayudante dinámico
La IP del host puede cambiar al reiniciar WSL2. Puedes tomarlo dinámicamente en tu shell:
```golpecito
exportar WSL_HOST=$(ip route show | grep -i default | awk '{ print $3 }')
echo "Host de Windows en: $WSL_HOST"
curl http://$WSL_HOST:11434/v1/models # Prueba Ollama
```O use el nombre mDNS de su máquina (requiere `libnss-mdns` en WSL2):
```golpecito
sudo apto instalar libnss-mdns
curl http://$(nombre de host).local:11434/v1/models
```
:::#### Dirección de enlace del servidor (requerida para el modo NAT)Si está utilizando la **Opción 2** (modo NAT con la IP del host), el servidor modelo en Windows debe aceptar conexiones desde fuera de `127.0.0.1`. De forma predeterminada, la mayoría de los servidores solo escuchan en el host local: las conexiones WSL2 en modo NAT provienen de una subred virtual diferente y serán rechazadas. En modo reflejado, `localhost` se asigna directamente, por lo que el enlace predeterminado `127.0.0.1` funciona bien.| Servidor | Enlace predeterminado | Cómo solucionarlo |
|--------|-------------|------------|
| **Ollama** | `127.0.0.1` | Establezca la variable de entorno `OLLAMA_HOST=0.0.0.0` antes de iniciar Ollama (Configuración del sistema → Variables de entorno en Windows, o edite el servicio Ollama) |
| **Estudio LM** | `127.0.0.1` | Habilite **"Servir en red"** en la pestaña Desarrollador → Configuración del servidor |
| **llama-servidor** | `127.0.0.1` | Agregue `--host 0.0.0.0` al comando de inicio |
| **vLLM** | `0.0.0.0` | Ya se vincula a todas las interfaces de forma predeterminada |
| **SGLang** | `127.0.0.1` | Agregue `--host 0.0.0.0` al comando de inicio |**Ollama en Windows (detallado):** Ollama se ejecuta como un servicio de Windows. Para configurar `OLLAMA_HOST`:
1. Abra **Propiedades del sistema** → **Variables de entorno**
2. Agregue una nueva **variable de sistema**: `OLLAMA_HOST` = `0.0.0.0`
3. Reinicie el servicio Ollama (o reinicie)#### Cortafuegos de WindowsWindows Firewall trata a WSL2 como una red separada (tanto en modo NAT como reflejado). Si las conexiones aún fallan después de los pasos anteriores, agregue una regla de firewall para el puerto de su servidor modelo:```powershell
# Ejecute en Admin PowerShell: reemplace PORT con el puerto de su servidor
New-NetFirewallRule -DisplayName "Permitir WSL2 al servidor modelo" -Dirección entrante -Acción Permitir -Protocolo TCP -LocalPort 11434
```Puertos comunes: Ollama `11434`, vLLM `8000`, SGLang `30000`, llama-server `8080`, LM Studio `1234`.#### Verificación rápidaDesde dentro de WSL2, pruebe que puede acceder a su servidor modelo:```golpecito
# Reemplace la URL con la dirección y el puerto de su servidor
curl http://localhost:11434/v1/models # Modo reflejado
curl http://172.29.192.1:11434/v1/models # Modo NAT (use la IP de su host real)
```Si recibe una respuesta JSON que enumera sus modelos, está bien. Utilice esa misma URL como `base_url` en su configuración de Hermes.---### Solución de problemas de modelos localesEstos problemas afectan a **todos** los servidores de inferencia locales cuando se utilizan con Hermes.#### "Conexión rechazada" de WSL2 a un servidor modelo alojado en WindowsSi está ejecutando Hermes dentro de WSL2 y su servidor modelo en el host de Windows, `http://localhost:<puerto>` no funcionará en el modo de red NAT predeterminado de WSL2. Consulte [Redes WSL2](#wsl2-networking-windows-users) más arriba para obtener la solución.#### Las llamadas a herramientas aparecen como texto en lugar de ejecutarseEl modelo genera algo como `{"name": "web_search", "arguments": {...}}` como un mensaje en lugar de llamar realmente a la herramienta.**Causa:** Su servidor no tiene habilitadas las llamadas a herramientas o el modelo no las admite a través de la implementación de llamadas a herramientas del servidor.| Servidor | Arreglar |
|--------|-----|
| **llama.cpp** | Agregue `--jinja` al comando de inicio |
| **vLLM** | Agregue `--enable-auto-tool-choice --tool-call-parser hermes` |
| **SGLang** | Agregue `--tool-call-parser qwen` (o analizador apropiado) |
| **Ollama** | La llamada a herramientas está habilitada de forma predeterminada; asegúrese de que su modelo la admita (consulte con `ollama show model-name`) |
| **Estudio LM** | Actualice a 0.3.6+ y use un modelo con soporte de herramientas nativas |#### El modelo parece olvidar el contexto o dar respuestas incoherentes**Causa:** La ventana de contexto es demasiado pequeña. Cuando la conversación excede el límite de contexto, la mayoría de los servidores descartan silenciosamente los mensajes más antiguos. Los esquemas de herramientas y avisos del sistema de Hermes por sí solos pueden usar tokens de 4k a 8k.**Diagnóstico:**```golpecito
# Comprueba lo que Hermes cree que es el contexto.
# Mire la línea de inicio: "Límite de contexto: X tokens"# Verifique el contexto real de su servidor
# Ollama: ollama ps (columna CONTEXTO)
# llama.cpp: curl http://localhost:8080/props | jq '.default_generación_configuración.n_ctx'
# vLLM: marque --max-model-len en los argumentos de inicio
```**Solución:** Establezca el contexto en al menos **64 000 tokens** para uso del agente. Consulte la sección anterior de cada servidor para conocer la bandera específica.#### "Límite de contexto: 2048 tokens" al inicioHermes detecta automáticamente la longitud del contexto desde el punto final `/v1/models` de su servidor. Si el servidor informa un valor bajo (o no informa ninguno), Hermes usa el límite declarado del modelo, que puede ser incorrecto.**Solución:** Configúrelo explícitamente en `config.yaml`:```yaml
modelo:
  predeterminado: tu-modelo
  proveedor: personalizado
  URL_base: http://localhost:11434/v1
  longitud_contexto: 64000
```#### Las respuestas se cortan a mitad de la frase.**Posibles causas:**
1. **Límite de salida bajo (`max_tokens`) en el servidor**: SGLang tiene por defecto 128 tokens por respuesta. Configure `--default-max-tokens` en el servidor o configure Hermes con `model.max_tokens` en config.yaml. Nota: `max_tokens` controla únicamente la longitud de la respuesta; no tiene relación con la duración del historial de conversaciones (es decir, `context_length`).
2. **Agotamiento del contexto**: el modelo llenó su ventana de contexto. Aumente `model.context_length` o habilite la [compresión de contexto](/user-guide/configuration#context-compression) en Hermes.---### Proxy LiteLLM: puerta de enlace multiproveedor[LiteLLM](https://docs.litellm.ai/) es un proxy compatible con OpenAI que unifica más de 100 proveedores de LLM detrás de una única API. Lo mejor para: cambiar entre proveedores sin cambios de configuración, equilibrio de carga, cadenas de respaldo, controles de presupuesto.```golpecito
# Instalar y comenzar
instalación de pip "litellm[proxy]"
litellm --modelo antrópico/claude-sonnet-4 --puerto 4000# O con un archivo de configuración para múltiples modelos:
litellm --config litellm_config.yaml --puerto 4000
```Luego configure Hermes con `modelo hermes` → Punto final personalizado → `http://localhost:4000/v1`.Ejemplo `litellm_config.yaml` con respaldo:
```yaml
lista_modelo:
  - nombre_modelo: "mejor"
    litellm_params:
      modelo: antrópico/claude-sonnet-4
      api_key: sk-hormiga-...
  - nombre_modelo: "mejor"
    litellm_params:
      modelo: openai/gpt-4o
      api_key: sk-...
configuración_enrutador:
  route_strategy: "enrutamiento basado en latencia"
```---### ClawRouter: enrutamiento con costos optimizados[ClawRouter](https://github.com/BlockRunAI/ClawRouter) de BlockRunAI es un proxy de enrutamiento local que selecciona automáticamente modelos según la complejidad de la consulta. Clasifica las solicitudes en 14 dimensiones y las dirige al modelo más barato que puede realizar la tarea. El pago se realiza mediante criptomoneda USDC (sin claves API).```golpecito
# Instalar y comenzar
npx @blockrun/clawrouter # Comienza en el puerto 8402
```Luego configure Hermes con `hermes model` → Punto final personalizado → `http://localhost:8402/v1` → nombre de modelo `blockrun/auto`.Perfiles de enrutamiento:
| Perfil | Estrategia | Ahorros |
|---------|----------|---------|
| `blockrun/auto` | Equilibrio calidad/coste | 74-100% |
| `blockrun/eco` | Lo más barato posible | 95-100% |
| `blockrun/premium` | Modelos de mejor calidad | 0% |
| `blockrun/gratis` | Sólo modelos gratuitos | 100% |
| `blockrun/agentic` | Optimizado para uso de herramientas | varía |:::nota
ClawRouter requiere una billetera financiada por USDC en Base o Solana para realizar el pago. Todas las solicitudes se enrutan a través de la API backend de BlockRun. Ejecute `npx @blockrun/clawrouter doctor` para verificar el estado de la billetera.
:::---### Otros proveedores compatiblesCualquier servicio con una API compatible con OpenAI funciona. Algunas opciones populares:| Proveedor | URL básica | Notas |
|----------|----------|-------|
| [Juntos AI](https://together.ai) | `https://api.together.xyz/v1` | Modelos abiertos alojados en la nube |
| [Groq](https://groq.com) | `https://api.groq.com/openai/v1` | Inferencia ultrarrápida |
| [Seek profundo] (https://deepseek.com) | `https://api.deepseek.com/v1` | Modelos DeepSeek |
| [Fuegos artificiales AI](https://fireworks.ai) | `https://api.fireworks.ai/inference/v1` | Alojamiento rápido de modelo abierto |
| [Nube GMI](https://www.gmicloud.ai/) | `https://api.gmi-serving.com/v1` | Inferencia gestionada compatible con OpenAI |
| [Cerebras](https://cerebras.ai) | `https://api.cerebras.ai/v1` | Inferencia de chips a escala de oblea |
| [IA Mistral](https://mistral.ai) | `https://api.mistral.ai/v1` | Modelos Mistral |
| [OpenAI](https://openai.com) | `https://api.openai.com/v1` | Acceso directo a OpenAI |
| [Azure OpenAI](https://azure.microsoft.com) | `https://TU.openai.azure.com/` | OpenAI empresarial |
| [AI local](https://localai.io) | `http://localhost:8080/v1` | Autohospedado, multimodelo |
| [Enero](https://jan.ai) | `http://localhost:1337/v1` | Aplicación de escritorio con modelos locales |Configure cualquiera de estos con `hermes model` → Punto final personalizado, o en `config.yaml`:```yaml
modelo:
  predeterminado: meta-llama/Llama-3.1-70B-Instruct-Turbo
  proveedor: personalizado
  URL_base: https://api.together.xyz/v1
  api_key: su-clave-juntos
```---### Detección de longitud de contexto:::nota Dos configuraciones, fáciles de confundir
**`context_length`** es la **ventana de contexto total**: el presupuesto combinado para los tokens de entrada *y* de salida (por ejemplo, 200.000 para Claude Opus 4.6). Hermes usa esto para decidir cuándo comprimir el historial y validar las solicitudes de API.**`model.max_tokens`** es el **límite de salida**: la cantidad máxima de tokens que el modelo puede generar en una *respuesta única*. No tiene nada que ver con la duración del historial de conversaciones. El nombre estándar de la industria `max_tokens` es una fuente común de confusión; Desde entonces, la API nativa de Anthropic le ha cambiado el nombre a `max_output_tokens` para mayor claridad.Establezca `context_length` cuando la detección automática tenga un tamaño de ventana incorrecto.
Configure `model.max_tokens` solo cuando necesite limitar la duración de las respuestas individuales.
:::Hermes utiliza una cadena de resolución de múltiples fuentes para detectar la ventana de contexto correcta para su modelo y proveedor:1. **Anulación de configuración** — `model.context_length` en config.yaml (prioridad más alta)
2. **Proveedor personalizado por modelo** — `custom_providers[].models.<id>.context_length`
3. **Caché persistente**: valores descubiertos previamente (sobrevive a los reinicios)
4. **Punto final `/models`**: consulta la API de su servidor (puntos finales locales/personalizados)
5. **Anthropic `/v1/models`** — consulta la API de Anthropic para `max_input_tokens` (solo usuarios de clave API)
6. **API de OpenRouter**: metadatos del modelo en vivo de OpenRouter
7. **Nous Portal**: el sufijo coincide con los ID del modelo Nous con los metadatos de OpenRouter
8. **[models.dev](https://models.dev)**: registro mantenido por la comunidad con longitudes de contexto específicas del proveedor para más de 3800 modelos en más de 100 proveedores
9. **Valores predeterminados de respaldo**: patrones de familia de modelos amplios (valor predeterminado de 128K)Para la mayoría de las configuraciones, esto funciona de inmediato. El sistema es consciente del proveedor: el mismo modelo puede tener diferentes límites de contexto dependiendo de quién lo proporcione (por ejemplo, `claude-opus-4.6` es 1M en Anthropic direct pero 128K en GitHub Copilot).Para establecer la longitud del contexto explícitamente, agregue `context_length` a la configuración de su modelo:```yaml
modelo:
  predeterminado: "qwen3.5:9b"
  base_url: "http://localhost:8080/v1"
  longitud_contexto: 131072 # tokens
```Para puntos finales personalizados, también puede establecer la longitud del contexto por modelo:```yaml
proveedores_personalizados:
  - nombre: "Mi LLM local"
    base_url: "http://localhost:11434/v1"
    modelos:
      qwen3.5:27b:
        longitud_contexto: 64000
      búsqueda profunda-r1: 70b:
        longitud_contexto: 65536
```El `hermes model` solicitará la longitud del contexto al configurar un punto final personalizado. Déjelo en blanco para la detección automática.:::consejo Cuándo configurar esto manualmente
- Estás usando Ollama con un `num_ctx` personalizado que es inferior al máximo del modelo.
- Desea limitar el contexto por debajo del máximo del modelo (por ejemplo, 8k en un modelo de 128k para ahorrar VRAM)
- Estás ejecutando detrás de un proxy que no expone `/v1/models`
:::---### Proveedores personalizados nombradosSi trabaja con varios puntos finales personalizados (por ejemplo, un servidor de desarrollo local y un servidor GPU remoto), puede definirlos como proveedores personalizados con nombre en `config.yaml`:```yaml
proveedores_personalizados:
  - nombre: local
    URL_base: http://localhost:8080/v1
    # api_key omitido: Hermes utiliza "no-key-required" para servidores locales sin clave
  - nombre: trabajo
    URL_base: https://gpu-server.internal.corp/v1
    clave_env: CORP_API_KEY
    api_mode: chat_completions # establecido explícitamente por `hermes model` → Asistente de punto final personalizado; la detección automática todavía ocurre como alternativa
  - nombre: proxy-antrópico
    URL_base: https://proxy.example.com/anthropic
    key_env: ANTHROPIC_PROXY_KEY
    api_mode: anthropic_messages # para proxies compatibles con Anthropic
```Algunos puntos finales compatibles con OpenAI necesitan campos de cuerpo de solicitud específicos del proveedor. Agregue un mapa `extra_body` al proveedor personalizado coincidente y Hermes lo fusionará en cada solicitud de finalización de chat para ese punto final:```yaml
proveedores_personalizados:
  - nombre: gemma-local
    URL_base: http://localhost:8080/v1
    modelo: google/gemma-4-31b-it
    cuerpo_extra:
      enable_thinking: verdadero
      esfuerzo_razonamiento: alto
```Utilice la forma de los documentos de su servidor. Por ejemplo, las implementaciones de vLLM Gemma y algunos puntos finales de NVIDIA NIM esperan "enable_thinking" en "chat_template_kwargs" en lugar de como un campo "extra_body" de nivel superior:```yaml
cuerpo_extra:
  chat_template_kwargs:
    enable_thinking: verdadero
```Para los modelos de razonamiento Qwen proporcionados por vLLM, esta misma forma se puede usar para desactivar el pensamiento cuando un analizador de razonamiento separa todo el texto generado en campos de razonamiento y deja el "contenido" del asistente vacío:```yaml
cuerpo_extra:
  chat_template_kwargs:
    enable_thinking: falso
```El asistente `hermes model` → Custom Endpoint ahora solicita `api_mode` explícitamente y persiste su respuesta en `config.yaml`. La detección automática basada en URL (por ejemplo, rutas `/anthropic` → `anthropic_messages`) todavía ocurre como alternativa cuando el campo se deja en blanco.**Visión nativa para modelos de proveedores personalizados.** Si su punto final personalizado sirve un modelo con capacidad de visión que no está en models.dev, configure `model.supports_vision: true` para que Hermes enrute las imágenes adjuntas de forma nativa (como partes de `image_url`) en lugar de preprocesarlas a través de `vision_analyze`. Perilla única: no es necesario configurar también `agent.image_input_mode: native`.```yaml
modelo:
  proveedor: personalizado
  URL_base: http://localhost:8080/v1
  predeterminado: qwen3.6-35b-a3b
  support_vision: true # envía imágenes de forma nativa; de lo contrario, vision_analyze los describe previamente
```La misma clave se respeta en los modelos por proveedor designado (`custom_providers[*].models[*].supports_vision`) y acepta valores booleanos YAML estándar (`true/false/yes/no/on/off/1/0`).Cambia entre ellos a mitad de sesión con la triple sintaxis:```
/model custom:local:qwen-2.5 # Utilice el punto final "local" con qwen-2.5
/model custom:work:llama3-70b # Usa el punto final "trabajo" con llama3-70b
/model custom:anthropic-proxy:claude-sonnet-4 # Usa el proxy
```También puede seleccionar proveedores personalizados con nombre en el menú interactivo "modelo Hermes".---### Libro de cocina: Juntos AI, Groq, PerplejidadTodos los proveedores de nube enumerados en [Otros proveedores compatibles](#otros-proveedores-compatibles) hablan el dialecto REST de OpenAI, por lo que se conectan de la misma manera en `custom_providers:`. A continuación se presentan tres recetas trabajadas. Cada uno cae en `~/.hermes/config.yaml` y la clave API correspondiente va en `~/.hermes/.env`.#### Juntos IAAlberga modelos de peso abierto (Llama, MiniMax, Gemma, DeepSeek, Qwen) a precios significativamente inferiores a las API propias. Buen valor predeterminado para flotas de varios modelos.```yaml
# ~/.hermes/config.yaml
proveedores_personalizados:
  - nombre: juntos
    URL_base: https://api.together.xyz/v1
    key_env: TOGETHER_API_KEY
    # api_mode: chat_completions # predeterminado: no es necesario configurarlomodelo:
  predeterminado: MiniMaxAI/MiniMax-M2.7 # o cualquier modelo de together.ai/models
  proveedor: personalizado: juntos
``````golpecito
# ~/.hermes/.env
TOGETHER_API_KEY=su-llave-juntos
```Cambiar de modelo a mitad de sesión:```
/modelo personalizado:juntos:meta-llama/Llama-3.3-70B-Instruct-Turbo
/modelo personalizado:juntos:google/gemma-4-31b-it
/modelo personalizado:juntos:deepseek-ai/DeepSeek-V3
```El punto final `/v1/models` de Together funciona, por lo que `hermes model` puede descubrir automáticamente los modelos disponibles.#### GroqInferencia ultrarrápida (~500 tok/s en Llama-3.3-70B). Catálogo pequeño pero potente para uso interactivo sensible a la latencia.```yaml
# ~/.hermes/config.yaml
proveedores_personalizados:
  - nombre: groq
    URL_base: https://api.groq.com/openai/v1
    key_env: GROQ_API_KEYmodelo:
  predeterminado: llama-3.3-70b-versatile
  proveedor: personalizado:groq
``````golpecito
# ~/.hermes/.env
GROQ_API_KEY=tu-clave-groq
```#### PerplejidadÚtil cuando desea un modelo que realice búsquedas y citas web en vivo automáticamente. Estricto sobre qué modelos están disponibles: consulte [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api) para ver la lista actual.```yaml
# ~/.hermes/config.yaml
proveedores_personalizados:
  - nombre: perplejidad
    URL_base: https://api.perplexity.ai
    key_env: PERPLEXITY_API_KEYmodelo:
  predeterminado: sonar
  proveedor: personalizado: perplejidad
``````golpecito
# ~/.hermes/.env
PERPLEXITY_API_KEY=tu-clave-de-perplejidad
```#### Múltiples proveedores en una configuraciónLas tres recetas se componen: úselas todas juntas y cambie por turno con `/model custom:<nombre>:<modelo>`:```yaml
proveedores_personalizados:
  - nombre: juntos
    URL_base: https://api.together.xyz/v1
    key_env: TOGETHER_API_KEY
  - nombre: groq
    URL_base: https://api.groq.com/openai/v1
    key_env: GROQ_API_KEY
  - nombre: perplejidad
    URL_base: https://api.perplexity.ai
    key_env: PERPLEXITY_API_KEYmodelo:
  predeterminado: MiniMaxAI/MiniMax-M2.7
  proveedor: personalizado: juntos # arrancar en Juntos; cambiar libremente después
```:::consejo Solución de problemas
- `hermes doctor` no debería imprimir advertencias de `Proveedor desconocido` para ninguno de estos nombres después de que el validador CLI corrija el número 15083.
- Si el punto final `/v1/models` de un proveedor es inalcanzable (Perplexity es el más común), el `modelo hermes` persistirá en el modelo con una advertencia en lugar de rechazarlo por completo; consulte el n.° 15136.
- Para omitir `custom_providers:` por completo y usar `provider: custom` con la var de entorno `CUSTOM_BASE_URL`, consulte el n.° 15103.
:::---### Elegir la configuración adecuada| Caso de uso | Recomendado |
|----------|-------------|
| **Solo quiero que funcione** | OpenRouter (predeterminado) o Nous Portal |
| **Modelos locales, fácil configuración** | Ollamá |
| **Servicio de GPU de producción** | vLLM o SGLang |
| **Mac/sin GPU** | Ollama o llama.cpp |
| **Enrutamiento multiproveedor** | Proxy LiteLLM u OpenRouter |
| **Optimización de costes** | ClawRouter u OpenRouter con `sort: "precio"` |
| **Máxima privacidad** | Ollama, vLLM o llama.cpp (completamente local) |
| **Empresa/Azure** | Azure OpenAI con punto final personalizado |
| **Modelos de IA chinos** | z.ai (GLM), Kimi/Moonshot (`kimi-coding` o `kimi-coding-cn`), MiniMax, Xiaomi MiMo o Tencent TokenHub (proveedores de primera clase) |:::consejo
Puede cambiar de proveedor en cualquier momento con el "modelo Hermes", sin necesidad de reiniciar. Su historial de conversaciones, memoria y habilidades se conservan independientemente del proveedor que utilice.
:::## Claves API opcionales| Característica | Proveedor | Variable de entorno |
|---------|----------|--------------|
| Raspado web | [Firecrawl](https://firecrawl.dev/) | `FIRECRAWL_API_KEY`, `FIRECRAWL_API_URL` |
| Automatización del navegador | [Base del navegador](https://browserbase.com/) | `BROWSERBASE_API_KEY`, `BROWSERBASE_PROJECT_ID` |
| Generación de imágenes | [FAL](https://fal.ai/) | `FAL_KEY` |
| Voces TTS premium | [ElevenLabs](https://elevenlabs.io/) | `ELEVENLABS_API_KEY` |
| OpenAI TTS + transcripción de voz | [OpenAI](https://platform.openai.com/api-keys) | `VOICE_TOOLS_OPENAI_KEY` |
| Mistral TTS + transcripción de voz | [Mistral](https://console.mistral.ai/) | `MISTRAL_API_KEY` |
| Modelado de usuarios entre sesiones | [Jefe](https://honcho.dev/) | `HONCHO_API_KEY` |
| Memoria semántica a largo plazo | [Supermemoria](https://supermemory.ai) | `SUPERMEMORY_API_KEY` |### Firecrawl autohospedadoDe forma predeterminada, Hermes utiliza la [API de la nube de Firecrawl](https://firecrawl.dev/) para la búsqueda web y el scraping. Si prefiere ejecutar Firecrawl localmente, puede apuntar a Hermes a una instancia autohospedada. Consulte [SELF_HOST.md] de Firecrawl (https://github.com/firecrawl/firecrawl/blob/main/SELF_HOST.md) para obtener instrucciones de configuración completas.**Lo que obtienes:** No se requiere clave API, sin límites de velocidad, sin costos por página, total soberanía de datos.**Lo que pierdes:** La versión en la nube utiliza el "motor de bomberos" patentado de Firecrawl para eludir anti-bots avanzados (Cloudflare, CAPTCHA, rotación de IP). El alojamiento propio utiliza Fetch + Playwright básico, por lo que algunos sitios protegidos pueden fallar. La búsqueda utiliza DuckDuckGo en lugar de Google.**Configuración:**1. Clonar e iniciar la pila Firecrawl Docker (5 contenedores: API, Playwright, Redis, RabbitMQ, PostgreSQL; requiere ~4-8 GB de RAM):
   ```golpecito
   clon de git https://github.com/firecrawl/firecrawl
   cd firecrawl
   # En .env, establezca: USE_DB_AUTHENTICATION=false, HOST=0.0.0.0, PORT=3002
   ventana acoplable componer -d
   ```2. Apunte Hermes a su instancia (no se necesita clave API):
   ```golpecito
   conjunto de configuración de hermes FIRECRAWL_API_URL http://localhost:3002
   ```También puede configurar tanto `FIRECRAWL_API_KEY` como `FIRECRAWL_API_URL` si su instancia autohospedada tiene la autenticación habilitada.## Enrutamiento del proveedor OpenRouterAl utilizar OpenRouter, puede controlar cómo se enrutan las solicitudes entre proveedores. Agregue una sección `provider_routing` a `~/.hermes/config.yaml`:```yaml
proveedor_enrutamiento:
  ordenar: "rendimiento" # "precio" (predeterminado), "rendimiento" o "latencia"
  # únicamente: ["anthropic"] # Utilice únicamente estos proveedores
  # ignorar: ["deepinfra"] # Omitir estos proveedores
  # orden: ["anthropic", "google"] # Pruebe proveedores en este orden
  # require_parameters: true # Utilice únicamente proveedores que admitan todos los parámetros de solicitud
  # data_collection: "deny" # Excluir proveedores que puedan almacenar/entrenar datos
```**Atajos:** Agregue `:nitro` a cualquier nombre de modelo para clasificar el rendimiento (por ejemplo, `anthropic/claude-sonnet-4:nitro`) o `:floor` para clasificar por precios.## Enrutador de código Pareto OpenRouterOpenRouter envía un enrutador de modelo de codificación experimental en `openrouter/pareto-code` que enruta automáticamente las solicitudes al modelo más barato que cumpla con una barra de calidad de codificación (clasificado por [Análisis artificial] (https://artificialanalysis.ai/)). Elija este modelo y ajuste la perilla `min_coding_score` en `~/.hermes/config.yaml`:```yaml
modelo:
  proveedor: enrutador abierto
  modelo: openrouter/pareto-codeenrutador abierto:
  puntuación_codificación mínima: 0,65 # 0,0–1,0; mayor = codificadores más fuertes (más caros). Predeterminado 0,65.
```Notas:- `min_coding_score` se **sólo** se envía cuando `model.model` es `openrouter/pareto-code`. En cualquier otro modelo, el valor no es operativo.
- Establezca una cadena vacía (o elimine la línea) para permitir que OpenRouter elija el codificador más potente disponible: su comportamiento documentado cuando se omite el bloque de complementos.
- La selección es determinista por puntuación en un día determinado, pero el modelo real elegido puede cambiar a medida que se mueve la frontera de Pareto (nuevos modelos, actualizaciones de puntos de referencia).
- Consulte los [documentos del enrutador Pareto] de OpenRouter (https://openrouter.ai/docs/guides/routing/routers/pareto-router) para conocer el comportamiento completo del enrutador.
- Para usar el enrutador de código de Pareto para una **tarea auxiliar** específica (compresión, visión, etc.) en lugar del agente principal, configure `extra_body.plugins` en esa tarea; consulte [Modelos auxiliares → Enrutamiento de OpenRouter y código de Pareto para tareas auxiliares](/user-guide/configuration#openrouter-routing--pareto-code-for-auxiliary-tasks).## Proveedores alternativosConfigure una cadena de proveedores de respaldo que Hermes intenta en orden cuando falla el modelo principal (límites de velocidad, errores del servidor, fallas de autenticación). El formato canónico es una lista `fallback_providers:` de nivel superior:```yaml
proveedores_alternativos:
  - proveedor: enrutador abierto
    modelo: antrópico/claude-sonnet-4
  - proveedor: antrópico
    modelo: claude-soneto-4
    # base_url: http://localhost:8000/v1 # opcional, para puntos finales personalizados
    # api_mode: chat_completions # anulación opcional
```El dict `fallback_model:` heredado de un solo par aún se acepta para compatibilidad con versiones anteriores:```yaml
modelo_alternativo:
  proveedor: enrutador abierto
  modelo: antrópico/claude-sonnet-4
```Cuando se activa, el respaldo intercambia el modelo y el proveedor a mitad de la sesión sin perder la conversación. La cadena se prueba entrada por entrada; La activación es única por sesión.Proveedores compatibles: `openrouter`, `nous`, `novita`, `openai-codex`, `copilot`, `copilot-acp`, `anthropic`, `gemini`, `qwen-oauth`, `huggingface`, `zai`, `kimi-coding`, `kimi-coding-cn`, `minimax`, `minimax-cn`, `minimax-oauth`, `deepseek`, `nvidia`, `xai`, `xai-oauth`, `ollama-cloud`, `bedrock`, `azure-foundry`, `opencode-zen`, `opencode-go`, `kilocode`, `xiaomi`, `arcee`, `gmi`, `stepfun`, `lmstudio`, `alibaba`, `alibaba-coding-plan`, `tencent-tokenhub`, `personalizado`.:::consejo
El respaldo se configura exclusivamente a través de `config.yaml`, o de forma interactiva a través de `hermes fallback`. Para obtener detalles completos sobre cuándo se activa, cómo avanza la cadena y cómo interactúa con las tareas auxiliares y la delegación, consulte [Proveedores alternativos](/user-guide/features/fallback-providers).
:::---## Ver también- [Configuración](/user-guide/configuration) — Configuración general (estructura de directorios, precedencia de configuración, backends de terminal, memoria, compresión y más)
- [Variables de entorno](/reference/environment-variables): referencia completa de todas las variables de entorno---