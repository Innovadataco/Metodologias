<!-- fuente: sitio web/docs/reference/environment-variables.md -->
# Variables de entorno# Referencia de variables de entornoHermes lee variables de entorno del entorno del proceso y, para los secretos administrados por el usuario, de `~/.hermes/.env`. Mantenga las claves de API, tokens de bot, secretos de OAuth y otras credenciales en `.env`; prefiera `config.yaml` para configuraciones de comportamiento no secretas cuando exista una clave de configuración. Algunas de las siguientes variables son anulaciones de proceso únicamente o variables de puente internas y no deben confirmarse en `.env` solo porque están documentadas aquí.## Proveedores de LLM| Variables | Descripción |
|----------|-------------|
| `OPENROUTER_API_KEY` | Clave API de OpenRouter (recomendada para mayor flexibilidad) |
| `OPENROUTER_BASE_URL` | Anule la URL base compatible con OpenRouter |
| `HERMES_OPENROUTER_CACHE` | Habilite el almacenamiento en caché de respuestas de OpenRouter (`1`/`true`/`yes`/`on`). Anula `openrouter.response_cache` en config.yaml. Consulte [Almacenamiento en caché de respuestas](https://openrouter.ai/docs/guides/features/response-caching). |
| `HERMES_OPENROUTER_CACHE_TTL` | Caché TTL en segundos (1-86400). Anula `openrouter.response_cache_ttl` en config.yaml. |
| `NOUS_BASE_URL` | Anular la URL base de Nous Portal (rara vez es necesario; solo desarrollo/pruebas) |
| `NOUS_INFERENCE_BASE_URL` | Anular el punto final de inferencia de Nous directamente |
| `OPENAI_API_KEY` | Clave API para puntos finales personalizados compatibles con OpenAI (usada con `OPENAI_BASE_URL`) |
| `OPENAI_BASE_URL` | URL base para punto final personalizado (VLLM, SGLang, etc.) |
| `LM_API_KEY` | Clave API para LM Studio (proveedor `lmstudio`). A menudo un marcador de posición para servidores locales |
| `LM_BASE_URL` | URL base de LM Studio (predeterminada: `http://localhost:1234/v1`) |
| `COPILOT_GITHUB_TOKEN` | Token de GitHub para API Copilot: primera prioridad (OAuth `gho_*` o PAT detallada `github_pat_*`; las PAT clásicas `ghp_*` **no son compatibles**) |
| `GH_TOKEN` | Token de GitHub: segunda prioridad para Copilot (también utilizado por la CLI `gh`) |
| `GITHUB_TOKEN` | Token de GitHub: tercera prioridad para Copilot |
| `HERMES_COPILOT_ACP_COMMAND` | Anular la ruta binaria CLI de Copilot ACP (predeterminado: `copilot`) |
| `COPILOT_CLI_PATH` | Alias ​​de `HERMES_COPILOT_ACP_COMMAND` |
| `HERMES_COPILOT_ACP_ARGS` | Anular los argumentos de Copilot ACP (predeterminado: `--acp --stdio`) |
| `COPILOT_ACP_BASE_URL` | Anular la URL base del ACP de Copilot |
| `COPILOT_API_BASE_URL` | Anular la URL base de la API de Copilot (proveedor `copilot`) |
| `GLM_API_KEY` | Clave API z.ai / ZhipuAI GLM ([z.ai](https://z.ai)) |
| `ZAI_API_KEY` | Alias ​​de `GLM_API_KEY` |
| `Z_AI_API_KEY` | Alias ​​de `GLM_API_KEY` |
| `GLM_BASE_URL` | Anular la URL base de z.ai (predeterminada: `https://api.z.ai/api/paas/v4`) |
| `KIMI_API_KEY` | Clave API de Kimi / Moonshot AI ([moonshot.ai](https://platform.moonshot.ai)) |
| `KIMI_CODING_API_KEY` | Clave de alias para el proveedor `kimi-coding` (aceptada junto con `KIMI_API_KEY`) |
| `KIMI_BASE_URL` | Anular la URL base de Kimi (predeterminada: `https://api.moonshot.ai/v1`) |
| `KIMI_CN_API_KEY` | Clave API de Kimi / Moonshot China ([moonshot.cn](https://platform.moonshot.cn)) |
| `ARCEEAI_API_KEY` | Clave API de Arcee AI ([chat.arcee.ai](https://chat.arcee.ai/)) |
| `ARCEE_BASE_URL` | Anular la URL base de Arcee (predeterminada: `https://api.arcee.ai/api/v1`) |
| `GMI_API_KEY` | Clave API de GMI Cloud ([gmicloud.ai](https://www.gmicloud.ai/)) |
| `GMI_BASE_URL` | Anular la URL base de GMI Cloud (predeterminada: `https://api.gmi-serving.com/v1`) |
| `MINIMAX_API_KEY` | Clave API MiniMax: punto final global ([minimax.io](https://www.minimax.io)). **No utilizado por `minimax-oauth`** (la ruta de OAuth utiliza el inicio de sesión del navegador). |
| `MINIMAX_BASE_URL` | Anule la URL base de MiniMax (predeterminada: `https://api.minimax.io/anthropic`; Hermes utiliza el punto final compatible con Anthropic Messages de MiniMax). **No utilizado por `minimax-oauth`**. |
| `MINIMAX_CN_API_KEY` | Clave API MiniMax: punto final de China ([minimaxi.com](https://www.minimaxi.com)). **No utilizado por `minimax-oauth`** (la ruta de OAuth utiliza el inicio de sesión del navegador). |
| `MINIMAX_CN_BASE_URL` | Anular la URL base de MiniMax China (predeterminada: `https://api.minimaxi.com/anthropic`). **No utilizado por `minimax-oauth`**. |
| `KILOCODE_API_KEY` | Clave API de Kilo Code ([kilo.ai](https://kilo.ai)) |
| `KILOCODE_BASE_URL` | Anular la URL base de Kilo Code (predeterminada: `https://api.kilo.ai/api/gateway`) |
| `XIAOMI_API_KEY` | Clave API de Xiaomi MiMo ([platform.xiaomimimo.com](https://platform.xiaomimimo.com)) |
| `XIAOMI_BASE_URL` | Anular la URL base de Xiaomi MiMo (predeterminada: `https://api.xiaomimimo.com/v1`) |
| `TOKENHUB_API_KEY` | Clave API de Tencent TokenHub ([tokenhub.tencentmaas.com](https://tokenhub.tencentmaas.com)) |
| `TOKENHUB_BASE_URL` | Anular la URL base de Tencent TokenHub (predeterminada: `https://tokenhub.tencentmaas.com/v1`) || `AZURE_FOUNDRY_API_KEY` | Clave API de Microsoft Foundry/Azure OpenAI ([ai.azure.com](https://ai.azure.com/)). No es necesario cuando `model.auth_mode: entra_id` |
| `AZURE_FOUNDRY_BASE_URL` | URL del punto final de Microsoft Foundry (por ejemplo, `https://<resource>.openai.azure.com/openai/v1` para el estilo OpenAI o `https://<resource>.services.ai.azure.com/anthropic` para el estilo Anthropic) |
| `AZURE_ANTHROPIC_KEY` | Clave de API de Azure Anthropic para `proveedor: anthropic` + `base_url` que apunta a una implementación de Microsoft Foundry Claude (alternativa a `ANTHROPIC_API_KEY` cuando tanto Anthropic como Azure Anthropic están configurados) |
| `AZURE_TENANT_ID` | ID de inquilino de Entra ID (flujos principales de servicio; respetado por `azure-identity` cuando `model.auth_mode: entra_id`) |
| `AZURE_CLIENT_ID` | ID de cliente de Entra ID (principal de servicio, identidad de carga de trabajo o identidad administrada asignada por el usuario) |
| `AZURE_CLIENT_SECRET` | Secreto principal del servicio utilizado por `EnvironmentCredential` |
| `AZURE_CLIENT_CERTIFICATE_PATH` | Certificado de entidad de servicio (alternativa a `AZURE_CLIENT_SECRET`) |
| `AZURE_FEDERATED_TOKEN_FILE` | Ruta de archivo de token federado para flujos OIDC/identidad de carga de trabajo de AKS |
| `AZURE_AUTHORITY_HOST` | Anulación de la autoridad de la nube soberana (por ejemplo, `https://login.microsoftonline.us` para Azure Government). Consulte la [guía de Azure Foundry](/guides/azure-foundry#sovereign-clouds-government-china) |
| `IDENTITY_ENDPOINT` / `MSI_ENDPOINT` | Punto final de identidad administrada para App Service, funciones y aplicaciones de contenedor; Las máquinas virtuales normalmente usan IMDS en su lugar y no configuran estos |
| `HF_TOKEN` | Token de cara de abrazo para proveedores de inferencia ([huggingface.co/settings/tokens](https://huggingface.co/settings/tokens)) |
| `HF_BASE_URL` | Anular la URL base de Hugging Face (predeterminada: `https://router.huggingface.co/v1`) |
| `GOOGLE_API_KEY` | Clave API de Google AI Studio ([aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)) |
| `GEMINI_API_KEY` | Alias ​​de `GOOGLE_API_KEY` |
| `GEMINI_BASE_URL` | Anular la URL base de Google AI Studio |
| `ANTHROPIC_API_KEY` | Clave API de la consola Anthropic ([console.anthropic.com](https://console.anthropic.com/)) |
| `ANTHROPIC_BASE_URL` | Anular la URL base de la API Anthropic |
| `ANTHROPIC_TOKEN` | Anulación de token de configuración/OAuth antrópico manual o heredado |
| `DASHSCOPE_API_KEY` | Clave API de Qwen Cloud (Alibaba DashScope) para modelos Qwen ([modelstudio.console.alibabacloud.com](https://modelstudio.console.alibabacloud.com/)) |
| `DASHSCOPE_BASE_URL` | URL base personalizada de DashScope (predeterminada: `https://dashscope-intl.aliyuncs.com/compatible-mode/v1`; utilice `https://dashscope.aliyuncs.com/compatible-mode/v1` para la región de China continental) |
| `ALIBABA_CODING_PLAN_API_KEY` | Clave API del plan de codificación Qwen (proveedor `alibaba-coding-plan`) |
| `ALIBABA_CODING_PLAN_BASE_URL` | Anular la URL base del plan de codificación Qwen |
| `DEEPSEEK_API_KEY` | Clave API de DeepSeek para acceso directo a DeepSeek ([platform.deepseek.com](https://platform.deepseek.com/api_keys)) |
| `DEEPSEEK_BASE_URL` | URL base personalizada de la API DeepSeek |
| `NOVITA_API_KEY` | Clave API de NovitaAI: nube nativa de IA para Model API, Agent Sandbox y GPU Cloud ([novita.ai/settings/key-management](https://novita.ai/settings/key-management)) |
| `NOVITA_BASE_URL` | Anular la URL base de NovitaAI (predeterminada: `https://api.novita.ai/openai/v1`) |
| `NVIDIA_API_KEY` | Clave API NVIDIA NIM: Nemotron y modelos abiertos ([build.nvidia.com](https://build.nvidia.com)) |
| `NVIDIA_BASE_URL` | Anular la URL base de NVIDIA (predeterminada: `https://integrate.api.nvidia.com/v1`; configurada en `http://localhost:8000/v1` para un punto final NIM local) |
| `STEPFUN_API_KEY` | Clave API de StepFun: modelos de la serie Step ([platform.stepfun.com](https://platform.stepfun.com)) |
| `STEPFUN_BASE_URL` | Anular la URL base de StepFun (predeterminada: `https://api.stepfun.com/v1`) |
| `OLLAMA_API_KEY` | Clave API de Ollama Cloud: catálogo de Ollama administrado sin GPU local ([ollama.com/settings/keys](https://ollama.com/settings/keys)) |
| `OLLAMA_BASE_URL` | Anular la URL base de Ollama Cloud (predeterminada: `https://ollama.com/v1`) |
| `XAI_API_KEY` | Clave API xAI (Grok) para chat + TTS + búsqueda web ([console.x.ai](https://console.x.ai/)) || `XAI_BASE_URL` | Anular la URL base de xAI (predeterminada: `https://api.x.ai/v1`) |
| `MISTRAL_API_KEY` | Clave API de Mistral para Voxtral TTS y Voxtral STT ([console.mistral.ai](https://console.mistral.ai)) |
| `AWS_REGION` | Región de AWS para inferencia de Bedrock (por ejemplo, `us-east-1`, `eu-central-1`). Leído por boto3. |
| `AWS_PROFILE` | Perfil con nombre de AWS para la autenticación Bedrock (lee `~/.aws/credentials`). Déjelo sin configurar para usar la cadena de credenciales boto3 predeterminada. |
| `BEDROCK_BASE_URL` | Anular la URL base del tiempo de ejecución de Bedrock (predeterminada: `https://bedrock-runtime.us-east-1.amazonaws.com`; normalmente no se establece y se usa `AWS_REGION` en su lugar) |
| `HERMES_QWEN_BASE_URL` | Anulación de URL base del Portal Qwen (predeterminado: `https://portal.qwen.ai/v1`) |
| `OPENCODE_ZEN_API_KEY` | Clave API OpenCode Zen: acceso de pago por uso a modelos seleccionados ([opencode.ai](https://opencode.ai/auth)) |
| `OPENCODE_ZEN_BASE_URL` | Anular la URL base de OpenCode Zen |
| `OPENCODE_GO_API_KEY` | Clave API OpenCode Go: suscripción de $ 10 al mes para modelos abiertos ([opencode.ai](https://opencode.ai/auth)) |
| `OPENCODE_GO_BASE_URL` | Anular la URL base de OpenCode Go |
| `CLAUDE_CODE_OAUTH_TOKEN` | Anulación explícita del token de Claude Code si exporta uno manualmente |
| `HERMES_MODEL` | Anular el nombre del modelo a nivel de proceso (usado por el programador cron; prefiera `config.yaml` para uso normal) |
| `VOICE_TOOLS_OPENAI_KEY` | Clave OpenAI preferida para proveedores de conversión de voz a texto y de texto a voz de OpenAI |
| `HERMES_LOCAL_STT_COMMAND` | Plantilla de comando local opcional de voz a texto. Admite marcadores de posición `{input_path}`, `{output_dir}`, `{language}` y `{model}` |
| `HERMES_LOCAL_STT_LANGUAGE` | Idioma predeterminado pasado a `HERMES_LOCAL_STT_COMMAND` o reserva CLI local `whisper` detectada automáticamente (predeterminado: `en`) |
| `HERMES_HOME` | Anule el directorio de configuración de Hermes (predeterminado: `~/.hermes`). También abarca el archivo PID de la puerta de enlace y el nombre del servicio systemd, por lo que se pueden ejecutar varias instalaciones simultáneamente |
| `HERMES_GIT_BASH_PATH` | **Solo Windows.** Anula el descubrimiento de `bash.exe` para la herramienta terminal. Puntos en cualquier bash: instalación completa de Git para Windows, bash WSL mediante enlace simbólico, MSYS2, Cygwin. El instalador configura esto automáticamente en el PortableGit que aprovisionó. Consulte la [Guía de Windows (nativa)](../user-guide/windows-native.md#how-hermes-runs-shell-commands-on-windows) |
| `HERMES_DISABLE_WINDOWS_UTF8` | **Solo Windows.** Establezca en `1` para deshabilitar la corrección estándar UTF-8 (`configure_windows_stdio()`) y regrese a la página de códigos locales de la consola. Útil para dividir errores de codificación; rara vez se encuentra el ajuste correcto en funcionamiento normal |
| `HERMES_KANBAN_HOME` | Anule la raíz compartida de Hermes que ancla el tablero kanban (base de datos + espacios de trabajo + registros de trabajadores). Vuelve a `get_default_hermes_root()` (el padre de cualquier perfil activo). Útil para pruebas e implementaciones inusuales |
| `HERMES_KANBAN_BOARD` | Fije el tablero kanban activo para este proceso. Tiene prioridad sobre `~/.hermes/kanban/current`; el despachador inyecta esto en el entorno del subproceso del trabajador para que los trabajadores no puedan ver físicamente las tareas en otros tableros. El valor predeterminado es "predeterminado". Validación de slug: caracteres alfanuméricos en minúsculas + guiones + guiones bajos, de 1 a 64 caracteres |
| `HERMES_KANBAN_DB` | Fije la ruta del archivo de la base de datos kanban directamente (prioridad más alta; supera a `HERMES_KANBAN_BOARD` y `HERMES_KANBAN_HOME`). El despachador inyecta esto en el subproceso del trabajador env para que los trabajadores del perfil converjan en el tablero del despachador |
| `HERMES_KANBAN_WORKSPACES_ROOT` | Fije la raíz de los espacios de trabajo kanban directamente (prioridad más alta para los espacios de trabajo; supera a `HERMES_KANBAN_HOME`). El despachador inyecta esto en el subproceso de trabajo env |
| `HERMES_KANBAN_DISPATCH_IN_GATEWAY` | Anulación del tiempo de ejecución para `kanban.dispatch_in_gateway`. Establezca en "0", "falso", "no" o "apagado" para evitar que la puerta de enlace inicie el despachador Kanban integrado; cualquier otro valor que no esté vacío lo habilita. Útil cuando un proceso despachador independiente es propietario de la placa. |## Autenticación del proveedor (OAuth)Para la autenticación antrópica nativa, Hermes prefiere los propios archivos de credenciales de Claude Code cuando existen porque esas credenciales se pueden actualizar automáticamente. **OAuth contra Anthropic requiere un plan Claude Max con créditos de uso adicionales comprados**: Hermes enruta como Claude Code, que solo se basa en los créditos adicionales/excedentes del plan Max, no en la asignación Max básica, y no funciona en Claude Pro. Sin créditos adicionales Max +, utilice una clave API en su lugar. Las variables de entorno como `ANTHROPIC_TOKEN` siguen siendo útiles como anulaciones manuales, pero ya no son la ruta preferida para iniciar sesión en Claude Max.| Variables | Descripción |
|----------|-------------|
| `HERMES_PORTAL_BASE_URL` | Anular la URL de Nous Portal (para desarrollo/pruebas) |
| `NOUS_INFERENCE_BASE_URL` | Anular la URL de la API de inferencia de Nous |
| `HERMES_NOUS_MIN_KEY_TTL_SECONDS` | TTL mínimo de clave de agente antes de volver a acuñar (predeterminado: 1800 = 30 min) |
| `HERMES_NOUS_TIMEOUT_SECONDS` | Tiempo de espera HTTP para flujos de credenciales/tokens de Nous |
| `HERMES_DUMP_REQUESTS` | Volcar cargas útiles de solicitudes de API en archivos de registro (`true`/`false`) |
| `HERMES_PREFILL_MESSAGES_FILE` | Ruta a un archivo JSON de mensajes de precarga efímeros inyectados en el momento de la llamada a la API |
| `HERMES_TIMEZONE` | Anulación de zona horaria de IANA (por ejemplo, "América/Nueva_York") |## API de herramientas| Variables | Descripción |
|----------|-------------|
| `PARALLEL_API_KEY` | Búsqueda web nativa de IA ([parallel.ai](https://parallel.ai/)) |
| `FIRECRAWL_API_KEY` | Web scraping y navegador en la nube ([firecrawl.dev](https://firecrawl.dev/)) |
| `FIRECRAWL_API_URL` | Punto final de API Firecrawl personalizado para instancias autohospedadas (opcional) |
| `TAVILY_API_KEY` | Clave API de Tavily para búsqueda, extracción y rastreo web nativo de IA ([app.tavily.com](https://app.tavily.com/home)) |
| `SEARXNG_URL` | URL de instancia de SearXNG para búsqueda web autohospedada gratuita: no se requiere clave API ([searxng.github.io](https://searxng.github.io/searxng/)) |
| `TAVILY_BASE_URL` | Anule el punto final de la API de Tavily. Útil para servidores proxy corporativos y backends de búsqueda autohospedados compatibles con Tavily. Mismo patrón que `GROQ_BASE_URL`. |
| `EXA_API_KEY` | Clave API de Exa para búsqueda y contenidos web nativos de IA ([exa.ai](https://exa.ai/)) |
| `BROWSERBASE_API_KEY` | Automatización del navegador ([browserbase.com](https://browserbase.com/)) |
| `BROWSERBASE_PROJECT_ID` | ID del proyecto de la base del navegador |
| `BROWSER_USE_API_KEY` | Navegador Utilice la clave API del navegador en la nube ([browser-use.com](https://browser-use.com/)) |
| `FIRECRAWL_BROWSER_TTL` | TTL de sesión del navegador Firecrawl en segundos (predeterminado: 300) |
| `BROWSER_CDP_URL` | URL del protocolo Chrome DevTools para el navegador local (configurada a través de `/browser connect`, por ejemplo, `ws://localhost:9222`) |
| `CAMOFOX_URL` | URL del navegador antidetección local Camofox (predeterminado: `http://localhost:9377`) |
| `CAMOFOX_USER_ID` | ID de usuario de Camofox administrado externamente opcional para sesiones visibles compartidas |
| `CAMOFOX_SESSION_KEY` | Clave de sesión de Camofox opcional utilizada al crear pestañas para `CAMOFOX_USER_ID` |
| `CAMOFOX_ADOPT_EXISTING_TAB` | Establezca en "verdadero" para reutilizar una pestaña Camofox existente antes de crear una nueva |
| `BROWSER_INACTIVITY_TIMEOUT` | Tiempo de espera de inactividad de la sesión del navegador en segundos |
| `AGENT_BROWSER_ARGS` | Indicadores de lanzamiento adicionales de Chromium (separados por comas o nueva línea). Hermes autoinyecta `--no-sandbox,--disable-dev-shm-usage` cuando se ejecuta como root o en espacios de nombres de usuario sin privilegios restringidos por AppArmor (Ubuntu 23.10+, DGX Spark, muchas imágenes de contenedores); configúrelo manualmente solo para anular o agregar otras banderas. |
| `FAL_KEY` | Generación de imágenes ([fal.ai](https://fal.ai/)) |
| `GROQ_API_KEY` | Clave API Groq Whisper STT ([groq.com](https://groq.com/)) |
| `ELEVENLABS_API_KEY` | Voces TTS premium de ElevenLabs ([elevenlabs.io](https://elevenlabs.io/)) |
| `STT_GROQ_MODEL` | Anule el modelo Groq STT (predeterminado: `whisper-large-v3-turbo`) |
| `GROQ_BASE_URL` | Anule el punto final STT compatible con Groq OpenAI |
| `STT_OPENAI_MODEL` | Anular el modelo OpenAI STT (predeterminado: `whisper-1`) |
| `STT_OPENAI_BASE_URL` | Anule el punto final STT compatible con OpenAI |
| `GITHUB_TOKEN` | Token de GitHub para Skills Hub (límites de tasa de API más altos, publicación de habilidades) |
| `HONCHO_API_KEY` | Modelado de usuarios entre sesiones ([honcho.dev](https://honcho.dev/)) |
| `HONCHO_BASE_URL` | URL base para instancias de Honcho autohospedadas (predeterminado: nube de Honcho). No se requiere clave API para instancias locales |
| `HINDSIGHT_TIMEOUT` | Tiempo de espera en segundos para llamadas a la API del proveedor de memoria en retrospectiva (predeterminado: `60`). Cambie esto si su instancia de Hindsight tarda en responder durante `/sync` o `on_session_switch` y ve tiempos de espera en `errors.log`. |
| `SUPERMEMORY_API_KEY` | Memoria semántica a largo plazo con recuperación de perfiles e ingesta de sesiones ([supermemory.ai](https://supermemory.ai)) |
| `DAYTONA_API_KEY` | Zonas de pruebas en la nube de Daytona ([daytona.io](https://daytona.io/)) |### Observabilidad de LangfuseVariables de entorno para el complemento [`observability/langfuse`](/user-guide/features/built-in-plugins#observabilitylangfuse) incluido. Configúrelos en `~/.hermes/.env`. El complemento también debe estar habilitado ("los complementos de hermes habilitan la observabilidad/langfuse", o marque la casilla en "complementos de hermes") antes de que cualquiera de estos surta efecto.| Variables | Descripción |
|----------|-------------|
| `HERMES_LANGFUSE_PUBLIC_KEY` | Clave pública del proyecto Langfuse (`pk-lf-...`). Requerido. |
| `HERMES_LANGFUSE_SECRET_KEY` | Clave secreta del proyecto Langfuse (`sk-lf-...`). Requerido. |
| `HERMES_LANGFUSE_BASE_URL` | URL del servidor Langfuse (predeterminado: `https://cloud.langfuse.com`). Configurado para autohospedado. |
| `HERMES_LANGFUSE_ENV` | Etiqueta de entorno en seguimientos (`producción`, `puesta en escena`,…) |
| `HERMES_LANGFUSE_RELEASE` | Etiqueta de lanzamiento/versión en seguimientos |
| `HERMES_LANGFUSE_SAMPLE_RATE` | Frecuencia de muestreo del SDK 0,0–1,0 (predeterminado: `1,0`) |
| `HERMES_LANGFUSE_MAX_CHARS` | Truncamiento por campo para cargas útiles serializadas (predeterminado: `12000`) |
| `HERMES_LANGFUSE_DEBUG` | `true` habilita el registro detallado del complemento en `agent.log` |
| `LANGFUSE_PUBLIC_KEY` / `LANGFUSE_SECRET_KEY` / `LANGFUSE_BASE_URL` | Nombres estándar del SDK de Langfuse. Se aceptan como alternativas cuando los equivalentes `HERMES_LANGFUSE_*` no están configurados. |### Puerta de enlace de Nous ToolEstas variables configuran la [Tool Gateway](/user-guide/features/tool-gateway) para suscriptores pagos de Nous o implementaciones de puerta de enlace autohospedada. La mayoría de los usuarios no necesitan configurarlos: la puerta de enlace se configura automáticamente mediante el "modelo Hermes" o las "herramientas Hermes".| Variables | Descripción |
|----------|-------------|
| `TOOL_GATEWAY_DOMAIN` | Dominio base para el enrutamiento de Tool Gateway (predeterminado: `nousresearch.com`) |
| `TOOL_GATEWAY_SCHEME` | Esquema HTTP o HTTPS para URL de puerta de enlace (predeterminado: `https`) |
| `TOOL_GATEWAY_USER_TOKEN` | Token de autenticación para Tool Gateway (normalmente se completa automáticamente desde Nous auth) |
| `FIRECRAWL_GATEWAY_URL` | Anular la URL específicamente para el punto final de la puerta de enlace de Firecrawl |## Terminal de fondo| Variables | Descripción |
|----------|-------------|
| `TERMINAL_ENV` | Backend: `local`, `docker`, `ssh`, `singularity`, `modal`, `daytona` |
| `HERMES_DOCKER_BINARY` | Anule el contenedor binario al que Hermes envía (por ejemplo, `podman`, `/usr/local/bin/docker`). Cuando no está configurado, Hermes descubre automáticamente "docker" o "podman" en "PATH". Es necesario cuando ambos están instalados y desea que no sean los predeterminados, o cuando el binario se encuentra fuera de `PATH`. |
| `TERMINAL_DOCKER_IMAGE` | Imagen de Docker (predeterminada: `nikolaik/python-nodejs:python3.11-nodejs20`) |
| `TERMINAL_DOCKER_FORWARD_ENV` | Matriz JSON de nombres de var de entorno para reenviar explícitamente a las sesiones del terminal Docker. Nota: las `required_environment_variables` declaradas por habilidades se reenvían automáticamente; solo necesitas esto para las variables no declaradas por ninguna habilidad. |
| `TERMINAL_DOCKER_VOLUMES` | Montajes de volúmenes Docker adicionales (pares `host:contenedor` separados por comas) |
| `TERMINAL_DOCKER_MOUNT_CWD_TO_WORKSPACE` | Opción avanzada: monte el cwd de inicio en Docker `/workspace` (`true`/`false`, predeterminado: `false`) |
| `TERMINAL_SINGULARITY_IMAGE` | Imagen de singularidad o ruta `.sif` |
| `TERMINAL_MODAL_IMAGE` | Imagen de contenedor modal |
| `TERMINAL_DAYTONA_IMAGE` | Imagen de la zona de pruebas de Daytona |
| `TERMINAL_TIMEOUT` | Tiempo de espera del comando en segundos |
| `TERMINAL_LIFETIME_SECONDS` | Vida útil máxima para sesiones de terminal en segundos |
| `TERMINAL_CWD` | Anulación directa obsoleta para sesiones de terminal de puerta de enlace/cron. Prefiera `terminal.cwd` en `config.yaml`; CLI todavía usa el directorio de inicio. |
| `SUDO_PASSWORD` | Habilite sudo sin mensaje interactivo |Para los backends de la zona de pruebas en la nube, la persistencia está orientada al sistema de archivos. `TERMINAL_LIFETIME_SECONDS` controla cuándo Hermes limpia una sesión de terminal inactiva, y las reanudaciones posteriores pueden recrear la zona de pruebas en lugar de mantener los mismos procesos activos en ejecución.## Servidor SSH| Variables | Descripción |
|----------|-------------|
| `TERMINAL_SSH_HOST` | Nombre de host del servidor remoto |
| `TERMINAL_SSH_USER` | Nombre de usuario SSH |
| `TERMINAL_SSH_PORT` | Puerto SSH (predeterminado: 22) |
| `TERMINAL_SSH_KEY` | Ruta a la clave privada |
| `TERMINAL_SSH_PERSISTENT` | Anular el shell persistente para SSH (predeterminado: sigue a `TERMINAL_PERSISTENT_SHELL`) |## Recursos de contenedores (Docker, Singularity, Modal, Daytona)| Variables | Descripción |
|----------|-------------|
| `TERMINAL_CONTENEDOR_CPU` | Núcleos de CPU (predeterminado: 1) |
| `TERMINAL_CONTAINER_MEMORY` | Memoria en MB (predeterminado: 5120) |
| `TERMINAL_CONTAINER_DISK` | Disco en MB (predeterminado: 51200) |
| `TERMINAL_CONTAINER_PERSISTENT` | Persistir el sistema de archivos contenedor entre sesiones (predeterminado: `true`) |
| `TERMINAL_SANDBOX_DIR` | Directorio de host para espacios de trabajo y superposiciones (predeterminado: `~/.hermes/sandboxes/`) |## Shell persistente| Variables | Descripción |
|----------|-------------|
| `TERMINAL_PERSISTENT_SHELL` | Habilite el shell persistente para backends no locales (predeterminado: "verdadero"). También se puede configurar a través de `terminal.persistent_shell` en config.yaml |
| `TERMINAL_LOCAL_PERSISTENT` | Habilite el shell persistente para el backend local (predeterminado: `falso`) |
| `TERMINAL_SSH_PERSISTENT` | Anular el shell persistente para el backend SSH (predeterminado: sigue a `TERMINAL_PERSISTENT_SHELL`) |## Mensajería| Variables | Descripción |
|----------|-------------|
| `TELEGRAMA_BOT_TOKEN` | Token de bot de Telegram (de @BotFather) |
| `TELEGRAMA_ALLOWED_USERS` | ID de usuario separados por comas permitidos para usar el bot (se aplica a mensajes directos, grupos y foros) |
| `TELEGRAM_GROUP_ALLOWED_USERS` | ID de usuario de remitente separados por comas autorizados solo en grupos/foros (NO otorga acceso a DM). Los valores en forma de ID de chat (que comienzan con `-`) todavía se respetan como ID de chat para la compatibilidad con configuraciones anteriores a #17686, con una advertencia de obsolescencia. |
| `TELEGRAM_GROUP_ALLOWED_CHATS` | ID de chat de foro/grupo separados por comas; cualquier miembro está autorizado |
| `TELEGRAMA_HOME_CHANNEL` | Chat/canal predeterminado de Telegram para entrega cron |
| `TELEGRAM_HOME_CHANNEL_NAME` | Nombre para mostrar del canal de inicio de Telegram |
| `TELEGRAMA_CRON_THREAD_ID` | ID del tema del foro para recibir entregas cron; anula `TELEGRAM_HOME_CHANNEL_THREAD_ID` solo para cron. Úselo en modo tema para que las respuestas a los mensajes cron abran una nueva sesión en lugar de acceder al lobby del sistema (#24409). |
| `TELEGRAM_WEBHOOK_URL` | URL HTTPS pública para el modo webhook (habilita el webhook en lugar del sondeo) |
| `TELEGRAMA_WEBHOOK_PORT` | Puerto de escucha local para el servidor webhook (predeterminado: `8443`) |
| `TELEGRAMA_WEBHOOK_SECRET` | El token secreto Telegram se repite en cada actualización para su verificación. **Obligatorio siempre que se configure `TELEGRAM_WEBHOOK_URL`**: la puerta de enlace se niega a iniciarse sin él (GHSA-3vpc-7q5r-276h). Generar con `openssl rand -hex 32`. |
| `TELEGRAMA_REACCIONES` | Habilitar reacciones emoji en mensajes durante el procesamiento (predeterminado: "falso") |
| `TELEGRAMA_REQUIRE_MENCIÓN` | Requerir un disparador explícito antes de responder en grupos de Telegram. Equivalente a `telegram.require_mention` en `config.yaml`. |
| `TELEGRAM_MENTION_PATTERNS` | Matriz JSON, lista separada por nueva línea o lista separada por comas de patrones de palabras de activación de expresiones regulares aceptadas cuando la activación de menciones de grupo de Telegram está habilitada. Equivalente a `telegram.mention_patterns`. |
| `TELEGRAM_EXCLUSIVE_BOT_MENTIONS` | Cuando está habilitado, las menciones explícitas de `@...bot` en los grupos de Telegram se dirigen solo a los nombres de usuario del bot mencionados antes de que se ejecuten las respuestas o las palabras de activación. Valor predeterminado: "verdadero". Equivalente a `telegram.exclusive_bot_mentions`. |
| `TELEGRAMA_REPLY_TO_MODE` | Comportamiento de referencia de respuesta: "desactivado", "primero" (predeterminado) o "todos". Coincide con el patrón de Discord. |
| `TELEGRAM_IGNORED_THREADS` | ID de temas/hilos del foro de Telegram separados por comas donde el bot nunca responde |
| `TELEGRAMA_PROXY` | URL proxy para conexiones de Telegram: anula `HTTPS_PROXY`. Admite `http://`, `https://`, `socks5://` |
| `DISCORD_BOT_TOKEN` | Ficha de robot de discordia |
| `DISCORD_ALLOWED_USERS` | ID de usuario de Discord separados por comas permitidos para usar el bot |
| `DISCORD_ALLOWED_ROLES` | ID de rol de Discord separados por comas permitidos para usar el bot (O con `DISCORD_ALLOWED_USERS`). Habilita automáticamente la intención de los miembros. Útil cuando los equipos de moderación cambian: las concesiones de roles se propagan automáticamente. |
| `DISCORD_ALLOWED_CHANNELS` | ID de canales de Discord separados por comas. Cuando está configurado, el bot solo responde en estos canales (más mensajes directos si está permitido). Anula `config.yaml` `discord.allowed_channels`. |
| `DISCORD_PROXY` | URL proxy para conexiones de Discord: anula `HTTPS_PROXY`. Admite `http://`, `https://`, `socks5://` |
| `DISCORD_HOME_CHANNEL` | Canal de Discord predeterminado para entrega cron |
| `DISCORD_HOME_CHANNEL_NAME` | Nombre para mostrar del canal de inicio de Discord |
| `DISCORD_COMMAND_SYNC_POLICY` | Política de sincronización de inicio del comando de barra diagonal de Discord: `seguro` (diferenciar y conciliar), `masivo` (heredado `tree.sync()`) o `apagado` |
| `DISCORD_REQUIRE_MENTION` | Requerir una @mención antes de responder en los canales del servidor |
| `DISCORD_FREE_RESPONSE_CHANNELS` | ID de canales separados por comas donde no es necesaria la mención |
| `DISCORD_AUTO_THREAD` | Subproceso automático de respuestas largas cuando sea compatible |
| `DISCORD_ALLOW_ANY_ATTACHMENT` | Cuando sea "verdadero", acepte archivos adjuntos de cualquier tipo de archivo (no solo la lista de permitidos integrada de PDF/texto/zip/office). Los tipos desconocidos se almacenan en caché y se muestran al agente como una ruta local para que pueda inspeccionarlos a través de `terminal`/`read_file`/`ffprobe`. Por defecto "falso". || `DISCORD_MAX_ATTACHMENT_BYTES` | Máximo de bytes por archivo adjunto que la puerta de enlace almacenará en caché. Predeterminado `33554432` (32 MiB). Establezca en `0` para que no haya límite (los archivos adjuntos se mantienen en la memoria mientras se escriben). |
| `DISCORD_REACTIONS` | Habilitar reacciones emoji en mensajes durante el procesamiento (predeterminado: "verdadero") |
| `DISCORD_IGNORED_CHANNELS` | ID de canal separados por comas donde el bot nunca responde |
| `DISCORD_NO_THREAD_CHANNELS` | ID de canal separados por comas donde el bot responde sin subprocesos automáticos |
| `DISCORD_REPLY_TO_MODE` | Comportamiento de referencia de respuesta: `desactivado`, `primero` (predeterminado) o `todos` |
| `DISCORD_ALLOW_MENTION_EVERYONE` | Permita que el bot haga ping a `@todos`/`@aquí` (predeterminado: `falso`). Consulte [Control de menciones] (../user-guide/messaging/discord.md#mention-control). |
| `DISCORD_ALLOW_MENTION_ROLES` | Permita que el bot haga ping a las menciones `@role` (predeterminado: `false`). |
| `DISCORD_ALLOW_MENTION_USERS` | Permita que el bot haga ping a menciones individuales de "@usuario" (predeterminado: "verdadero"). |
| `DISCORD_ALLOW_MENTION_REPLIED_USER` | Haga ping al autor cuando responda a su mensaje (predeterminado: "verdadero"). |
| `SLACK_BOT_TOKEN` | Token de bot flojo (`xoxb-...`) |
| `SLACK_APP_TOKEN` | Token de nivel de aplicación de Slack (`xapp-...`, requerido para el modo Socket) |
| `SLACK_ALLOWED_USERS` | ID de usuario de Slack separados por comas |
| `SLACK_HOME_CHANNEL` | Canal Slack predeterminado para entrega cron |
| `SLACK_HOME_CHANNEL_NAME` | Nombre para mostrar del canal de inicio de Slack |
| `GOOGLE_CHAT_PROJECT_ID` | Proyecto GCP que aloja el tema Pub/Sub (recurre a `GOOGLE_CLOUD_PROJECT`) |
| `GOOGLE_CHAT_SUBSCRIPTION_NAME` | Ruta de suscripción completa de Pub/Sub, `projects/{proj}/subscriptions/{sub}` (alias antiguo: `GOOGLE_CHAT_SUBSCRIPTION`) |
| `GOOGLE_CHAT_SERVICE_ACCOUNT_JSON` | Ruta al JSON de la cuenta de servicio, o el JSON en línea (recurre a `GOOGLE_APPLICATION_CREDENTIALS`) |
| `GOOGLE_CHAT_ALLOWED_USERS` | Los correos electrónicos de los usuarios separados por comas permiten chatear con el bot |
| `GOOGLE_CHAT_ALLOW_ALL_USERS` | Permitir que cualquier usuario de Google Chat active el bot (solo para desarrolladores) |
| `GOOGLE_CHAT_HOME_CHANNEL` | Espacio predeterminado (por ejemplo, `espacios/AAAA...`) para la entrega cron |
| `GOOGLE_CHAT_HOME_CHANNEL_NAME` | Nombre para mostrar del espacio de inicio de Google Chat |
| `GOOGLE_CHAT_MAX_MESSAGES` | Mensajes en curso máximos de Pub/Sub FlowControl (predeterminado: `1`) |
| `GOOGLE_CHAT_MAX_BYTES` | Bytes en tránsito máximos de Pub/Sub FlowControl (predeterminado: `16777216`, 16 MiB) |
| `GOOGLE_CHAT_BOOTSTRAP_SPACES` | ID de espacio adicional separados por comas para sondear al inicio al resolver los propios `usuarios/{id}` del bot |
| `GOOGLE_CHAT_DEBUG_RAW` | Establezca cualquier valor para registrar sobres de Pub/Sub redactados en el nivel DEBUG (solo depuración) |
| `QUÉTSAPP_ENABLED` | Habilitar el puente de WhatsApp (`true`/`false`) |
| `MODO_QUÉTSAPP` | `bot` (número separado) o `self-chat` (envíe mensajes a usted mismo) |
| `WHATSAPP_ALLOWED_USERS` | Números de teléfono separados por comas (con código de país, sin `+`) o `*` para permitir que todos los remitentes |
| `WHATSAPP_ALLOW_ALL_USERS` | Permitir a todos los remitentes de WhatsApp sin una lista de permitidos (`true`/`false`) |
| `QUÉTSAPP_DEBUG` | Registrar eventos de mensajes sin procesar en el puente para solucionar problemas (`true`/`false`) |
| `WHATSAPP_CLOUD_PHONE_NUMBER_ID` | ID de metanúmero de teléfono de la API de WhatsApp Business Cloud (15 a 17 dígitos; **no** el número de teléfono en sí) |
| `WHATSAPP_CLOUD_ACCESS_TOKEN` | Token de metaacceso (comienza con `EAA`); los tokens temporales caducan después de 24 horas, los tokens de usuario del sistema son permanentes |
| `WHATSAPP_CLOUD_APP_SECRET` | Secreto de aplicación hexadecimal de 32 caracteres utilizado para verificar firmas de webhooks entrantes |
| `WHATSAPP_CLOUD_VERIFY_TOKEN` | Secreto compartido para el protocolo de enlace de verificación del webhook de Meta (generado automáticamente por el asistente de configuración) |
| `WHATSAPP_CLOUD_ALLOWED_USERS` | `wa_id`s separados por comas (números de teléfono con código de país, sin `+`) permitidos para enviar mensajes al bot |
| `WHATSAPP_CLOUD_ALLOW_ALL_USERS` | Permitir a todos los remitentes de WhatsApp Cloud sin una lista de permitidos (`true`/`false`) |
| `WHATSAPP_CLOUD_APP_ID` ​​| ID de metaaplicación opcional (para futura integración de análisis) |
| `WHATSAPP_CLOUD_WABA_ID` | ID de cuenta empresarial de WhatsApp opcional (para futura integración de análisis) |
| `WHATSAPP_CLOUD_WEBHOOK_HOST` | Interfaz a la que se vincula el servidor webhook entrante (predeterminado `0.0.0.0`) || `WHATSAPP_CLOUD_WEBHOOK_PORT` | Puerto al que se vincula el servidor webhook entrante (predeterminado `8090`) |
| `WHATSAPP_CLOUD_WEBHOOK_PATH` | Ruta URL Meta publica mensajes entrantes en (predeterminado `/whatsapp/webhook`) |
| `WHATSAPP_CLOUD_API_VERSION` | Versión de Meta Graph API para llamar (predeterminada `v20.0`) |
| `WHATSAPP_CLOUD_HOME_CHANNEL` | `wa_id` para usar como canal de inicio del bot (para trabajos cron, etc.) |
| `WHATSAPP_CLOUD_DM_POLICY` | Puerta DM para el adaptador de nube (`open`/`allowlist`/`disabled`); vuelve a `WHATSAPP_DM_POLICY` cuando no está configurado |
| `WHATSAPP_CLOUD_ALLOW_FROM` | Se permiten remitentes separados por comas cuando `dm_policy: enablelist` (`wa_id`s básicos; los JID estilo Baileys están normalizados) |
| `WHATSAPP_CLOUD_GROUP_POLICY` | Puerta de grupo para el adaptador de nube (`open`/`allowlist`/`disabled`); vuelve a `WHATSAPP_GROUP_POLICY` cuando no está configurado |
| `WHATSAPP_CLOUD_GROUP_ALLOW_FROM` | Se permiten ID de chat grupal separados por comas cuando `group_policy: enablelist` |
| `SIGNAL_HTTP_URL` | Punto final HTTP del demonio signal-cli (por ejemplo, `http://127.0.0.1:8080`) |
| `SIGNAL_ACCOUNT` | Número de teléfono del bot en formato E.164 |
| `SIGNAL_ALLOWED_USERS` | Números de teléfono o UUID E.164 separados por comas |
| `SIGNAL_GROUP_ALLOWED_USERS` | ID de grupo separados por comas o `*` para todos los grupos |
| `SIGNAL_HOME_CHANNEL_NAME` | Nombre para mostrar del canal principal de Signal |
| `SIGNAL_IGNORE_STORIES` | Ignorar historias de Signal/actualizaciones de estado |
| `SIGNAL_ALLOW_ALL_USERS` | Permitir a todos los usuarios de Signal sin una lista de permitidos |
| `TWILIO_ACCOUNT_SID` | SID de cuenta Twilio (compartido con habilidad de telefonía) |
| `TWILIO_AUTH_TOKEN` | Twilio Auth Token (compartido con la habilidad de telefonía; también utilizado para la validación de firmas de webhooks) |
| `TWILIO_PHONE_NUMBER` | Número de teléfono de Twilio en formato E.164 (compartido con habilidad de telefonía) |
| `SMS_WEBHOOK_URL` | URL pública para la validación de firmas de Twilio: debe coincidir con la URL del webhook en la consola de Twilio (obligatorio) |
| `SMS_WEBHOOK_PORT` | Puerto de escucha de webhook para SMS entrantes (predeterminado: `8080`) |
| `SMS_WEBHOOK_HOST` | Dirección de enlace del webhook (predeterminado: `0.0.0.0`) |
| `SMS_INSECURE_NO_SIGNATURE` | Establezca en "verdadero" para deshabilitar la validación de firmas de Twilio (solo para desarrolladores locales, no para producción) |
| `SMS_ALLOWED_USERS` | Números de teléfono E.164 separados por comas autorizados para chatear |
| `SMS_ALLOW_ALL_USERS` | Permitir a todos los remitentes de SMS sin una lista de permitidos |
| `SMS_HOME_CHANNEL` | Número de teléfono para trabajo cron/entrega de notificaciones |
| `SMS_HOME_CHANNEL_NAME` | Nombre para mostrar del canal de inicio de SMS |
| `DIRECCIÓN_EMAIL_DIRECCIÓN` | Dirección de correo electrónico para el adaptador de puerta de enlace de correo electrónico |
| `EMAIL_PASSWORD` | Contraseña o contraseña de la aplicación para la cuenta de correo electrónico |
| `EMAIL_IMAP_HOST` | Nombre de host IMAP para el adaptador de correo electrónico |
| `EMAIL_IMAP_PORT` | Puerto IMAP |
| `EMAIL_SMTP_HOST` | Nombre de host SMTP para el adaptador de correo electrónico |
| `EMAIL_SMTP_PORT` | Puerto SMTP |
| `EMAIL_ALLOWED_USERS` | Direcciones de correo electrónico separadas por comas permitidas para enviar mensajes al bot |
| `EMAIL_HOME_ADDRESS` | Destinatario predeterminado para la entrega proactiva de correo electrónico |
| `EMAIL_HOME_ADDRESS_NAME` | Nombre para mostrar del destino de inicio del correo electrónico |
| `EMAIL_POLL_INTERVAL` | Intervalo de sondeo de correo electrónico en segundos |
| `EMAIL_ALLOW_ALL_USERS` | Permitir todos los remitentes de correo electrónico entrante |
| `DINGTALK_CLIENT_ID` | AppKey del bot DingTalk del portal para desarrolladores ([open.dingtalk.com](https://open.dingtalk.com)) |
| `DINGTALK_CLIENT_SECRET` | Bot DingTalk AppSecret del portal de desarrolladores |
| `DINGTALK_ALLOWED_USERS` | ID de usuario de DingTalk separados por comas permitidos para enviar mensajes al bot |
| `FEISHU_APP_ID` ​​| ID de la aplicación del bot Feishu/Lark de [open.feishu.cn](https://open.feishu.cn/) |
| `FEISHU_APP_SECRET` | Secreto de la aplicación Feishu/Lark bot |
| `FEISHU_DOMAIN` | `feishu` (China) o `alondra` (internacional). Predeterminado: `feishu` |
| `FEISHU_CONNECTION_MODE` | `websocket` (recomendado) o `webhook`. Predeterminado: `websocket` |
| `FEISHU_ENCRYPT_KEY` | Clave de cifrado opcional para el modo webhook |
| `FEISHU_VERIFICATION_TOKEN` | Token de verificación opcional para el modo webhook |
| `FEISHU_ALLOWED_USERS` | Se permiten ID de usuario de Feishu separados por comas para enviar mensajes al bot |
| `FEISHU_ALLOW_BOTS` | `ninguno` (predeterminado) / `menciones` / `todos`: acepta mensajes entrantes de otros bots. Consulte [mensajería de bot a bot](../user-guide/messaging/feishu.md#bot-to-bot-messaging) || `FEISHU_REQUIRE_MENTION` | `true` (predeterminado) / `false`: si los mensajes grupales deben @mencionar el bot. Anular por chat a través de `group_rules.<chat_id>.require_mention`. |
| `FEISHU_HOME_CHANNEL` | ID de chat de Feishu para entrega cron y notificaciones |
| `WECOM_BOT_ID` | ID de WeCom AI Bot desde la consola de administración |
| `WECOM_SECRET` | Secreto del robot AI de WeCom |
| `WECOM_WEBSOCKET_URL` | URL de WebSocket personalizada (predeterminada: `wss://openws.work.weixin.qq.com`) |
| `WECOM_ALLOWED_USERS` | ID de usuario de WeCom separados por comas permitidos para enviar mensajes al bot |
| `WECOM_HOME_CHANNEL` | ID de chat de WeCom para entrega cron y notificaciones |
| `WECOM_CALLBACK_CORP_ID` ​​| ID de WeCom Enterprise Corp para aplicación de devolución de llamada de creación propia |
| `WECOM_CALLBACK_CORP_SECRET` | Secreto corporativo para la aplicación de creación propia |
| `WECOM_CALLBACK_AGENT_ID` | ID de agente de la aplicación de creación propia |
| `WECOM_CALLBACK_TOKEN` | Token de verificación de devolución de llamada |
| `WECOM_CALLBACK_ENCODING_AES_KEY` | Clave AES para cifrado de devolución de llamada |
| `WECOM_CALLBACK_HOST` | Dirección de enlace del servidor de devolución de llamada (predeterminado: `0.0.0.0`) |
| `WECOM_CALLBACK_PORT` | Puerto del servidor de devolución de llamada (predeterminado: `8645`) |
| `WECOM_CALLBACK_ALLOWED_USERS` | ID de usuario separados por comas para la lista de permitidos |
| `WECOM_CALLBACK_ALLOW_ALL_USERS` | Establezca "verdadero" para permitir a todos los usuarios sin una lista de permitidos |
| `WEIXIN_ACCOUNT_ID` | ID de cuenta Weixin obtenida mediante inicio de sesión QR a través de iLink Bot API |
| `WEIXIN_TOKEN` | Token de autenticación Weixin obtenido mediante inicio de sesión QR a través de iLink Bot API |
| `WEIXIN_BASE_URL` | Anular la URL base de la API Weixin iLink Bot (predeterminada: `https://ilinkai.weixin.qq.com`) |
| `WEIXIN_CDN_BASE_URL` | Anular la URL base de Weixin CDN para medios (predeterminado: `https://novac2c.cdn.weixin.qq.com/c2c`) |
| `WEIXIN_DM_POLICY` | Política de mensajes directos: `abierto`, `lista de permitidos`, `emparejamiento`, `deshabilitado` (predeterminado: `abierto`) |
| `WEIXIN_GROUP_POLICY` | Política de mensajes de grupo: `abierto`, `lista de permitidos`, `deshabilitado` (predeterminado: `deshabilitado`) |
| `WEIXIN_ALLOWED_USERS` | Los ID de usuario de Weixin separados por comas permiten enviar mensajes directos al bot |
| `WEIXIN_GROUP_ALLOWED_USERS` | Los **ID de chat grupal** de Weixin separados por comas (no los ID de usuarios de miembros) permiten interactuar con el bot. El nombre de la variable es heredado: espera ID de grupo. Sólo entra en vigor cuando iLink realmente ofrece eventos grupales; Las identidades de bot de iLink con inicio de sesión QR (`...@im.bot`) normalmente no reciben mensajes grupales normales de WeChat. |
| `WEIXIN_HOME_CHANNEL` | ID de chat de Weixin para entrega cron y notificaciones |
| `WEIXIN_HOME_CHANNEL_NAME` | Nombre para mostrar del canal de inicio de Weixin |
| `WEIXIN_ALLOW_ALL_USERS` | Permitir a todos los usuarios de Weixin sin una lista de permitidos (`true`/`false`) |
| `BLUEBUBBLES_SERVER_URL` | URL del servidor BlueBubbles (por ejemplo, `http://192.168.1.10:1234`) |
| `BLUEBUBBLES_PASSWORD` | Contraseña del servidor BlueBubbles |
| `BLUEBUBBLES_WEBHOOK_HOST` | Dirección de enlace del oyente de webhook (predeterminado: `127.0.0.1`) |
| `BLUEBUBBLES_WEBHOOK_PORT` | Puerto de escucha de webhook (predeterminado: `8645`) |
| `BLUEBUBBLES_HOME_CHANNEL` | Teléfono/correo electrónico para entrega de cron/notificación |
| `BLUEBUBBLES_ALLOWED_USERS` | Usuarios autorizados separados por comas |
| `BLUEBUBBLES_ALLOW_ALL_USERS` | Permitir a todos los usuarios (`true`/`false`) |
| `QQ_APP_ID` ​​| ID de la aplicación QQ Bot de [q.qq.com](https://q.qq.com) |
| `QQ_CLIENT_SECRET` | Secreto de la aplicación QQ Bot de [q.qq.com](https://q.qq.com) |
| `QQ_STT_API_KEY` | Clave API para proveedor de respaldo STT externo (opcional, utilizada cuando el ASR integrado de QQ no devuelve texto) |
| `QQ_STT_BASE_URL` | URL base para proveedor STT externo (opcional) |
| `QQ_STT_MODEL` | Nombre del modelo para proveedor STT externo (opcional) |
| `QQ_ALLOWED_USERS` | Los openID de usuario de QQ separados por comas pueden enviar mensajes al bot |
| `QQ_GROUP_ALLOWED_USERS` | ID de grupo QQ separados por comas para acceso al grupo @-message |
| `QQ_ALLOW_ALL_USERS` | Permitir a todos los usuarios (`true`/`false`, anula `QQ_ALLOWED_USERS`) |
| `QQBOT_HOME_CHANNEL` | OpenID de usuario/grupo QQ para entrega cron y notificaciones |
| `QQBOT_HOME_CHANNEL_NAME` | Nombre para mostrar del canal de inicio de QQ |
| `QQ_PORTAL_HOST` | Anule el host del portal QQ (establecido en `sandbox.q.qq.com` para enrutar a través de la puerta de enlace de la zona de pruebas; valor predeterminado: `q.qq.com`). || `MATTERMOST_URL` | URL del servidor Mattermost (por ejemplo, `https://mm.example.com`) |
| `MATTERMOST_TOKEN` | Token de bot o token de acceso personal para Mattermost |
| `MATTERMOST_ALLOWED_USERS` | ID de usuario de Mattermost separados por comas permitidos para enviar mensajes al bot |
| `MATTERMOST_HOME_CHANNEL` | ID de canal para entrega proactiva de mensajes (cron, notificaciones) |
| `MATTERMOST_REQUIRE_MENTION` | Requerir `@mention` en los canales (predeterminado: `true`). Establezca en "falso" para responder a todos los mensajes. |
| `MATTERMOST_FREE_RESPONSE_CHANNELS` | ID de canal separados por comas donde el bot responde sin `@mención` |
| `MATTERMOST_REPLY_MODE` | Estilo de respuesta: `thread` (respuestas encadenadas) o `off` (mensajes planos, predeterminado) |
| `MATRIX_HOMESERVER` | URL del servidor doméstico de Matrix (por ejemplo, `https://matrix.org`) |
| `MATRIX_ACCESS_TOKEN` | Token de acceso a matriz para autenticación de bots |
| `MATRIX_USER_ID` | ID de usuario de Matrix (por ejemplo, `@hermes:matrix.org`): requerido para iniciar sesión con contraseña, opcional con token de acceso |
| `MATRIX_PASSWORD` | Contraseña Matrix (alternativa al token de acceso) |
| `MATRIX_ALLOWED_USERS` | ID de usuario de Matrix separados por comas permitidos para enviar mensajes al bot (por ejemplo, `@alice:matrix.org`) |
| `MATRIX_ALLOWED_ROOMS` | Se permiten ID de salas de Matrix separados por comas para activar respuestas de bot |
| `MATRIX_HOME_ROOM` | ID de sala para entrega proactiva de mensajes (por ejemplo, `!abc123:matrix.org`) |
| `MATRIX_ENCRYPTION` | Habilitar el cifrado de extremo a extremo (`true`/`false`, predeterminado: `false`) |
| `MATRIX_E2EE_MODE` | Comportamiento de Matrix E2EE: "desactivado", "opcional" o "obligatorio". Anula `MATRIX_ENCRYPTION` cuando se establece. |
| `MATRIX_DEVICE_ID` | ID de dispositivo Matrix estable para persistencia E2EE entre reinicios (por ejemplo, `HERMES_BOT`). Sin esto, las claves E2EE rotan en cada inicio y se interrumpe el descifrado de la sala histórica. |
| `MATRIX_REACTIONS` | Habilite las reacciones emoji del ciclo de vida del procesamiento en los mensajes entrantes (valor predeterminado: "verdadero"). Establezca en "falso" para deshabilitarlo. |
| `MATRIX_REQUIRE_MENTION` | Requerir `@mention` en las salas (predeterminado: `true`). Establezca en "falso" para responder a todos los mensajes. |
| `MATRIX_FREE_RESPONSE_ROOMS` | ID de sala separados por comas donde el bot responde sin `@mención` |
| `MATRIX_IGNORE_USER_PATTERNS` | Expresiones regulares separadas por comas para que se ignoren los ID de usuario fantasma de Matrix Bridge/appservice |
| `MATRIX_PROCESS_NOTICES` | Procesar eventos entrantes de Matrix `m.notice` (predeterminado: `false`) |
| `MATRIX_SESSION_SCOPE` | Alcance de la sesión de matriz para salas de proyecto: `auto`, `room` o `thread` (predeterminado: `auto`) |
| `MATRIX_TOOLS_ALLOW_CROSS_ROOM` | Permitir que las herramientas Matrix apunten a salas explícitas distintas a la sala actual (predeterminado: `falso`) |
| `MATRIX_TOOLS_ALLOW_CROSS_ROOM_DESTRUCTIVE` | Permitir herramientas de redacción de Matrix/invitación entre salas; requiere `MATRIX_TOOLS_ALLOW_CROSS_ROOM=true` (predeterminado: `false`) |
| `MATRIX_TOOLS_ALLOW_REDACTION` | Permitir la ejecución de la herramienta de redacción de mensajes Matrix (predeterminado: `falso`) |
| `MATRIX_TOOLS_ALLOW_INVITES` | Permitir la ejecución de la herramienta de invitación Matrix (predeterminado: `falso`) |
| `MATRIX_TOOLS_ALLOW_ROOM_CREATE` | Permitir la ejecución de la herramienta de creación de salas Matrix (predeterminado: `falso`) |
| `MATRIX_ALLOW_ROOM_MENTIONS` | Permitir menciones salientes de `@room` para notificar a todos los miembros de la sala (predeterminado: `false`) |
| `MATRIX_AUTO_THREAD` | Crear automáticamente hilos para mensajes de sala (predeterminado: `true`) |
| `MATRIX_DM_MENTION_THREADS` | Crear un hilo cuando el bot sea `@mencionado` en un DM (predeterminado: `falso`) |
| `MATRIX_APPROVAL_REQUIRE_SENDER` | Requerir que las reacciones de aprobación/selector de modelo provengan del solicitante original cuando se conozcan (valor predeterminado: "verdadero") |
| `MATRIX_APPROVAL_TIMEOUT_SECONDS` | Tiempo de espera para la aprobación de reacción de Matrix/indicaciones de selección de modelo (predeterminado: `300`) |
| `MATRIX_ALLOW_PUBLIC_ROOMS` | Permitir que las herramientas de creación de salas de Matrix creen salas públicas (predeterminado: `falso`) |
| `MATRIX_MAX_MEDIA_BYTES` | Tamaño máximo de carga/descarga de medios Matrix en bytes (predeterminado: `104857600`) |
| `MATRIX_RECOVERY_KEY` | Clave de recuperación para verificación de firma cruzada después de la rotación de la clave del dispositivo. Recomendado para configuraciones E2EE con firma cruzada habilitada. |
| `MATRIX_RECOVERY_KEY_OUTPUT_FILE` | Ruta opcional de un solo uso para una clave de recuperación de Matrix generada. Creado con el modo `0600` y nunca sobrescrito. || `HASS_TOKEN` | Token de acceso de larga duración de Home Assistant (habilita la plataforma HA + herramientas) |
| `HASS_URL` | URL de Home Assistant (predeterminada: `http://homeassistant.local:8123`) |
| `WEBHOOK_ENABLED` | Habilite el adaptador de plataforma webhook (`true`/`false`) |
| `WEBHOOK_PORT` | Puerto del servidor HTTP para recibir webhooks (predeterminado: `8644`) |
| `WEBHOOK_SECRET` | Secreto global de HMAC para la validación de firmas de webhooks (se utiliza como respaldo cuando las rutas no especifican las suyas propias) |
| `API_SERVER_ENABLED` | Habilite el servidor API compatible con OpenAI (`true`/`false`). Funciona junto con otras plataformas. |
| `API_SERVER_KEY` | Token de portador para la autenticación del servidor API. Requerido siempre que el servidor API esté habilitado. |
| `API_SERVER_CORS_ORIGINS` | Los orígenes del navegador separados por comas permitían llamar al servidor API directamente (por ejemplo, `http://localhost:3000,http://127.0.0.1:3000`). Valor predeterminado: deshabilitado. |
| `API_SERVER_PORT` | Puerto para el servidor API (predeterminado: `8642`) |
| `API_SERVER_HOST` | Dirección de host/enlace para el servidor API (predeterminado: `127.0.0.1`). `API_SERVER_KEY` todavía se requiere en el bucle invertido; utilice una lista de permitidos estrecha `API_SERVER_CORS_ORIGINS` para el acceso al navegador. |
| `API_SERVER_MODEL_NAME` | Nombre del modelo anunciado en `/v1/models`. El valor predeterminado es el nombre del perfil (o "hermes-agent" para el perfil predeterminado). Útil para configuraciones multiusuario donde interfaces como Open WebUI necesitan nombres de modelo distintos por conexión. |
| `GATEWAY_PROXY_URL` | URL de un servidor API de Hermes remoto al que reenviar mensajes ([modo proxy](/user-guide/messaging/matrix#proxy-mode-e2ee-on-macos)). Cuando está configurada, la puerta de enlace maneja solo la E/S de la plataforma: todo el trabajo del agente se delega al servidor remoto. También se puede configurar a través de `gateway.proxy_url` en `config.yaml`. |
| `GATEWAY_PROXY_KEY` | Token de portador para autenticar con el servidor API remoto en modo proxy. Debe coincidir con `API_SERVER_KEY` en el host remoto. |
| `MENSAJE_CWD` | Reserva de compatibilidad obsoleta para el directorio de trabajo de la puerta de enlace. Prefiera `terminal.cwd` en `config.yaml`. |
| `GATEWAY_ALLOWED_USERS` | Se permiten ID de usuario separados por comas en todas las plataformas |
| `GATEWAY_ALLOW_ALL_USERS` | Permitir a todos los usuarios sin listas permitidas (`true`/`false`, predeterminado: `false`) |### Panel web y escritorio HermesAutenticación para el [panel web](/user-guide/features/web-dashboard) y para conectar [Hermes Desktop a un backend remoto](/user-guide/features/web-dashboard#connecting-hermes-desktop-to-a-remote-backend). Según la convención de solo secretos, las credenciales pertenecen a `~/.hermes/.env`; Es mejor configurar OAuth `client_id` en `dashboard.oauth` en `config.yaml` (env gana cuando se configura).En la caja se envían tres proveedores de autenticación de panel. Para una conexión remota de Hermes Desktop o cualquier panel con acceso a Internet, el proveedor recomendado es **OAuth (Nous Portal)**: configure `HERMES_DASHBOARD_OAUTH_CLIENT_ID` (proporciónelo con `hermes Dashboard Register`). El proveedor de **nombre de usuario/contraseña** incluido (`HERMES_DASHBOARD_BASIC_AUTH_*`) es la opción más rápida para un backend en una LAN confiable o detrás de una VPN, pero no es adecuado para la exposición directa a Internet pública. Para autenticarse con su propio proveedor de identidad, utilice el proveedor **OIDC autohospedado** (`HERMES_DASHBOARD_OIDC_*`). De cualquier manera, un enlace sin bucle invertido (`hermes panel --host 0.0.0.0`) activa la puerta de autenticación. Consulte [Panel web → Autenticación](/user-guide/features/web-dashboard#authentication-gated-mode) para obtener una imagen completa.| Variables | Descripción |
|----------|-------------|
| `HERMES_DASHBOARD_BASIC_AUTH_USERNAME` | Nombre de usuario para el proveedor de autenticación del panel de nombre de usuario/contraseña incluido (`plugins/dashboard_auth/basic`). Activa el proveedor cuando se configura junto con una contraseña. Anula `dashboard.basic_auth.username`. |
| `HERMES_DASHBOARD_BASIC_AUTH_PASSWORD` | Contraseña de texto sin formato para el proveedor básico (en memoria hash durante la carga). Gana una configuración `password_hash` para que puedas rotar a través de env. Anula `dashboard.basic_auth.password`. |
| `HERMES_DASHBOARD_BASIC_AUTH_PASSWORD_HASH` | hash de contraseña scrypt para el proveedor básico (preferiblemente, sin texto sin formato en reposo). Calcule con `python -c "de plugins.dashboard_auth.basic import hash_password; print(hash_password('PW'))"`. Anula `dashboard.basic_auth.password_hash`. |
| `HERMES_DASHBOARD_BASIC_AUTH_SECRET` | Clave HMAC (más de 32 bytes, base64/hex/raw) que firma los tokens de sesión sin estado del proveedor básico. Establecer explícitamente para que las sesiones sobrevivan a los reinicios/abarquen a varios trabajadores; en blanco → aleatorio por proceso (se cerrará su sesión cada vez que se reinicie). Anula `dashboard.basic_auth.secret`. |
| `HERMES_DASHBOARD_BASIC_AUTH_TTL_SECONDS` | Duración del token de acceso para el proveedor básico (predeterminado 12 h). Anula `dashboard.basic_auth.session_ttl_segundos`. |
| `HERMES_DASHBOARD_OAUTH_CLIENT_ID` | ID de cliente de OAuth (`agent:{instance_id}`) para el panel privado/público, activando el proveedor Nous (`plugins/dashboard_auth/nous`). Anula `dashboard.oauth.client_id`. Provisiónelo con el "registro del panel de Hermes". |
| `HERMES_DASHBOARD_PUBLIC_URL` | URL pública completa a la que se accede al panel para la construcción de devolución de llamada de OAuth detrás de servidores proxy inversos. Anula `dashboard.public_url`. |
| `HERMES_DASHBOARD_OIDC_ISSUER` | URL del emisor de OIDC para el proveedor de OIDC autohospedado incluido (`plugins/dashboard_auth/self_hosted`). Requerido para activarlo. Anula `dashboard.oauth.self_hosted.issuer`. |
| `HERMES_DASHBOARD_OIDC_CLIENT_ID` | ID de cliente OIDC público (código de autorización + PKCE) para el proveedor OIDC autohospedado. Requerido para activarlo. Anula `dashboard.oauth.self_hosted.client_id`. |
| `HERMES_DASHBOARD_OIDC_SCOPES` | Ámbitos OIDC solicitados para el proveedor OIDC autohospedado (`correo electrónico de perfil openid` predeterminado). Anula `dashboard.oauth.self_hosted.scopes`. |
| `HERMES_DESKTOP_REMOTE_URL` | (Lado del escritorio) URL base del backend remoto, p. `http://host:9119`. Cuando se configura, anula la URL de la puerta de enlace en la aplicación; aún inicia sesión desde el panel de configuración de la puerta de enlace (redireccionamiento de OAuth o nombre de usuario/contraseña, lo que anuncie el backend). |
| `HERMES_DESKTOP_HERMES` | Anulación del comando de backend del escritorio. Utilizado por empaquetadores/Nix o solución de problemas para señalar a Electron a un ejecutable "hermes" específico después de la prueba de backend. |
| `HERMES_DESKTOP_HERMES_ROOT` | Anulación de verificación de fuente de escritorio utilizada por `hermes escritorio --hermes-root`; verificado antes de la instalación del primer lanzamiento del paquete o de un `hermes` existente en `PATH`. |
| `HERMES_DESKTOP_IGNORE_EXISTING` | Establezca en `1` para que el escritorio ignore un `hermes` existente en `PATH` durante la resolución del backend. Equivalente a `hermes escritorio --ignorar-existente`. |
| `HERMES_DESKTOP_CWD` | Directorio de proyecto inicial para sesiones de chat de escritorio. Establecido por `hermes escritorio --cwd`. |### Microsoft Graph (reuniones de equipos)Credenciales de solo aplicación para el cliente REST de Microsoft Graph utilizadas por la próxima canalización de resumen de reuniones de Teams. Consulte [Registrar una aplicación Microsoft Graph](/guides/microsoft-graph-app-registration) para obtener el tutorial de Azure Portal y los permisos de API exactos necesarios.| Variables | Descripción |
|----------|-------------|
| `MSGRAPH_TENANT_ID` | ID de inquilino de Azure AD (GUID del directorio) para el registro de la aplicación Graph. |
| `MSGRAPH_CLIENT_ID` | ID de la aplicación (cliente) del registro de la aplicación de Azure. |
| `MSGRAPH_CLIENT_SECRET` | Valor secreto del cliente para el registro de la aplicación. Almacenar en `~/.hermes/.env` con `chmod 600`; rotar periódicamente a través del portal de Azure. |
| `MSGRAPH_SCOPE` | Alcance de OAuth2 para la solicitud de token de credenciales de cliente (predeterminado: `https://graph.microsoft.com/.default`). |
| `MSGRAPH_AUTHORITY_URL` | Autoridad de la plataforma de identidad de Microsoft (predeterminada: `https://login.microsoftonline.com`). Anulación solo para nubes nacionales/soberanas (por ejemplo, `https://login.microsoftonline.us` para GCC High). |### Oyente de webhook de Microsoft GraphOyente de notificaciones de cambios entrantes para eventos de Graph (reuniones de Teams, calendario, chat, etc.). Consulte [Microsoft Graph Webhook Listener](/user-guide/messaging/msgraph-webhook) para configurar y reforzar la seguridad.| Variables | Descripción |
|----------|-------------|
| `MSGRAPH_WEBHOOK_ENABLED` | Habilite la plataforma de puerta de enlace `msgraph_webhook` (`true`/`1`/`yes`). |
| `MSGRAPH_WEBHOOK_PORT` | Puerto al que se vincula el oyente (predeterminado: `8646`). |
| `MSGRAPH_WEBHOOK_CLIENT_STATE` | El gráfico secreto compartido se hace eco en cada notificación; comparado con `hmac.compare_digest`. Generar con `openssl rand -hex 32`. |
| `MSGRAPH_WEBHOOK_ACCEPTED_RESOURCES` | Lista permitida separada por comas de rutas/patrones de recursos de Graph (por ejemplo, `comunicaciones/reuniones en línea,chats/*/mensajes`). El `*` final coincide con el prefijo. Vacío = aceptar todo. |
| `MSGRAPH_WEBHOOK_ALLOWED_SOURCE_CIDRS` | Rangos CIDR separados por comas permitidos para PUBLICAR al oyente (por ejemplo, `52.96.0.0/14,52.104.0.0/14`). Vacío = permitir todo (predeterminado). Restringir a los rangos de salida publicados de Microsoft Graph en producción. |### Entrega del resumen de la reunión de TeamsSolo se usa cuando el [complemento `teams_pipeline`](/user-guide/messaging/msgraph-webhook) está habilitado. Las configuraciones también se pueden configurar en `platforms.teams.extra` en `config.yaml`; las variables de entorno tienen prioridad cuando ambas están configuradas. Consulte [Microsoft Teams → Entrega de resumen de reunión](/user-guide/messaging/teams#meeting-summary-delivery-teams-meeting-pipeline).| Variables | Descripción |
|----------|-------------|
| `TEAMS_DELIVERY_MODE` | `gráfico` o `incoming_webhook`. |
| `TEAMS_INCOMING_WEBHOOK_URL` | URL de webhook generada por Teams; requerido cuando `TEAMS_DELIVERY_MODE=incoming_webhook`. |
| `TEAMS_GRAPH_ACCESS_TOKEN` | Token de acceso delegado adquirido previamente para la entrega de Graph. Rara vez es necesario: el escritor recurre a las credenciales de la aplicación `MSGRAPH_*` cuando no está configurado. |
| `TEAMS_TEAM_ID` | ID del equipo objetivo para la entrega del canal (modo `gráfico`). |
| `TEAMS_CHANNEL_ID` | ID del canal de destino (emparejado con `TEAMS_TEAM_ID`). |
| `TEAMS_CHAT_ID` | Objetivo 1:1 o ID de chat grupal (alternativa a equipo+canal para el modo "gráfico"). |### API de mensajería de LINEUtilizado por el complemento de plataforma LINE incluido (`plugins/platforms/line/`). Consulte [Messaging Gateway → LINE](/user-guide/messaging/line) para una configuración completa.| Variables | Descripción |
|----------|-------------|
| `LINE_CHANNEL_ACCESS_TOKEN` | Token de acceso al canal de larga duración desde LINE Developers Console (pestaña API de mensajería). Requerido. |
| `LINE_CHANNEL_SECRET` | Secreto de canal (pestaña Configuración básica); Se utiliza para la verificación de la firma del webhook HMAC-SHA256. Requerido. |
| `LINE_HOST` | Host de enlace de webhook (predeterminado: `0.0.0.0`). |
| `LINE_PORT` | Puerto de enlace de webhook (predeterminado: `8646`). |
| `LINE_PUBLIC_URL` | URL base HTTPS pública (por ejemplo, `https://my-tunnel.example.com`). Requerido para envíos de imágenes/audio/video: LINE solo acepta URL accesibles mediante HTTPS. |
| `LINE_ALLOWED_USERS` | Los ID de usuario separados por comas permiten enviar mensajes directos al bot (con el prefijo `U`). |
| `LINE_ALLOWED_GROUPS` | ID de grupo separados por comas en los que el bot responderá (con el prefijo `C`). |
| `LINE_ALLOWED_ROOMS` | ID de sala separados por comas en los que responderá el bot (con el prefijo `R`). |
| `LINE_ALLOW_ALL_USERS` | Trampilla de escape exclusiva para desarrolladores: acepta cualquier fuente. Valor predeterminado: "falso". |
| `LINE_HOME_CHANNEL` | Destino de entrega predeterminado para trabajos cron con `deliver: line`. |
| `LINE_SLOW_RESPONSE_THRESHOLD` | Segundos antes de que se active la devolución de datos lenta de los botones de plantilla LLM (predeterminado: `45`). Establezca `0` para deshabilitar y siempre Push-fallback. |
| `LINE_PENDING_TEXT` | Texto de burbuja que se muestra junto al botón de devolución. |
| `LINE_BUTTON_LABEL` | Etiqueta del botón de devolución (predeterminado: "Obtener respuesta"). |
| `LINE_DELIVERED_TEXT` | Responder cuando se vuelva a tocar una devolución de datos ya entregada (predeterminado: `Ya respondí ✅`). |
| `LINE_INTERRUPTED_TEXT` | Responder cuando se toca un botón de devolución de datos huérfano `/stop` (valor predeterminado: `La ejecución se interrumpió antes de completarse.`). |### ntfy (notificaciones push)[ntfy](https://ntfy.sh/) es un servicio ligero de notificaciones push basado en HTTP. Suscríbase a un tema desde la [aplicación móvil ntfy](https://ntfy.sh/docs/subscribe/phone/), publique en ese tema para hablar con el agente.| Variables | Descripción |
|----------|-------------|
| `NTFY_TOPIC` | Tema al que suscribirse (mensajes entrantes). Requerido. |
| `NTFY_SERVER_URL` | URL del servidor (predeterminado: `https://ntfy.sh`). Señale un ntfy autohospedado para mayor privacidad. |
| `NTFY_TOKEN` | Token de autenticación opcional. Token de portador (por ejemplo, `tk_xyz`) o `user:pass` para autenticación básica. |
| `NTFY_PUBLISH_TOPIC` | Tema para las respuestas salientes (el valor predeterminado es `NTFY_TOPIC`). |
| `NTFY_MARKDOWN` | Establezca "verdadero" para enviar respuestas con el encabezado "X-Markdown: verdadero". Valor predeterminado: "falso". |
| `NTFY_ALLOWED_USERS` | Lista de permitidos (tratada como ID de usuario; en ntfy estos son nombres de temas). Normalmente se establece en el mismo valor que `NTFY_TOPIC`. |
| `NTFY_ALLOW_ALL_USERS` | Escotilla de escape solo para desarrolladores: solo segura en temas privados con acceso controlado. Valor predeterminado: "falso". |
| `NTFY_HOME_CHANNEL` | Destino de entrega predeterminado para trabajos cron con `deliver: ntfy`. |
| `NTFY_HOME_CHANNEL_NAME` | Etiqueta humana para el canal de inicio (el valor predeterminado es el nombre del tema). |Consulte [la guía de mensajería ntfy](/user-guide/messaging/ntfy), en particular la sección **modelo de identidad**, antes de implementar temas que no sean de confianza.### Ajuste avanzado de mensajeríaPerillas avanzadas por plataforma para acelerar el lote de mensajes salientes. La mayoría de los usuarios nunca necesitan tocarlos; Los valores predeterminados están configurados para respetar los límites de velocidad de cada plataforma sin sentirse lento.| Variables | Descripción |
|----------|-------------|
| `HERMES_TELEGRAM_TEXT_BATCH_DELAY_SECONDS` | Ventana de gracia antes de vaciar un fragmento de texto de Telegram en cola (predeterminado: `0.6`). |
| `HERMES_TELEGRAM_TEXT_BATCH_SPLIT_DELAY_SECONDS` | Retraso entre fragmentos divididos cuando un único mensaje de Telegram excede el límite de longitud (predeterminado: `2.0`). |
| `HERMES_TELEGRAM_MEDIA_BATCH_DELAY_SECONDS` | Ventana de gracia antes de vaciar los medios de Telegram en cola (predeterminado: `0.6`). |
| `HERMES_TELEGRAM_FOLLOWUP_GRACE_SECONDS` | Retrase antes de enviar un seguimiento después de que el agente finalice, para evitar acelerar el último fragmento de transmisión. |
| `HERMES_TELEGRAM_HTTP_CONNECT_TIMEOUT` / `_READ_TIMEOUT` / `_WRITE_TIMEOUT` / `_POOL_TIMEOUT` | Anule los tiempos de espera HTTP subyacentes de `python-telegram-bot` (segundos). |
| `HERMES_TELEGRAM_HTTP_POOL_SIZE` | Máximo de conexiones HTTP simultáneas a la API de Telegram. |
| `HERMES_TELEGRAM_DISABLE_FALLBACK_IPS` | Deshabilite las IP alternativas codificadas de Cloudflare que se utilizan cuando falla el DNS (`verdadero`/`falso`). |
| `HERMES_DISCORD_TEXT_BATCH_DELAY_SECONDS` | Ventana de gracia antes de vaciar un fragmento de texto de Discord en cola (predeterminado: `0.6`). |
| `HERMES_DISCORD_TEXT_BATCH_SPLIT_DELAY_SECONDS` | Retraso entre fragmentos divididos cuando un mensaje de Discord excede el límite de longitud (predeterminado: `2.0`). |
| `HERMES_MATRIX_TEXT_BATCH_DELAY_SECONDS` / `_SPLIT_DELAY_SECONDS` | Equivalentes matriciales de los mandos por lotes de Telegram. |
| `HERMES_FEISHU_TEXT_BATCH_DELAY_SECONDS` / `_SPLIT_DELAY_SECONDS` / `_MAX_CHARS` / `_MAX_MESSAGES` | Ajuste del lotedor Feishu: retraso, retraso dividido, máximo de caracteres por mensaje, máximo de mensajes por lote. |
| `HERMES_FEISHU_MEDIA_BATCH_DELAY_SECONDS` | Retraso en el flujo de medios de Feishu. |
| `HERMES_FEISHU_DEDUP_CACHE_SIZE` | Tamaño de la caché de desduplicación del webhook Feishu (predeterminado: `1024`). |
| `HERMES_WECOM_TEXT_BATCH_DELAY_SECONDS` / `_SPLIT_DELAY_SECONDS` | Ajuste del dosificador WeCom. |
| `HERMES_VISION_DOWNLOAD_TIMEOUT` | Tiempo de espera en segundos para descargar una imagen antes de entregarla a los modelos de visión (predeterminado: `30`). |
| `HERMES_RESTART_DRAIN_TIMEOUT` | Puerta de enlace: segundos de espera para que las ejecuciones activas se agoten en `/restart` antes de forzar el reinicio (predeterminado: `900`). |
| `HERMES_GATEWAY_PLATFORM_CONNECT_TIMEOUT` | Tiempo de espera de conexión por plataforma durante el inicio de la puerta de enlace (segundos). |
| `HERMES_GATEWAY_BUSY_INPUT_MODE` | Comportamiento predeterminado de entrada ocupada de la puerta de enlace: "cola", "dirección" o "interrupción". Se puede anular por chat con `/busy`. |
| `HERMES_GATEWAY_BUSY_ACK_ENABLED` | Si la puerta de enlace envía un mensaje de confirmación (⚡/⏳/⏩) cuando un usuario envía información mientras el agente está ocupado (valor predeterminado: "verdadero"). Configúrelo en "falso" para suprimir estos mensajes por completo: la entrada aún está en cola/dirigida/interrumpida como de costumbre, solo se silencia la respuesta del chat. Puenteado desde `display.busy_ack_enabled` en `config.yaml`. |
| `HERMES_GATEWAY_NO_SUPERVISE` | Dentro de la imagen de Docker superpuesta de s6, opte por no participar en la supervisión automática cuando ejecute `hermes gateway run` y use la semántica de primer plano anterior a s6 (sin reinicio automático, la puerta de enlace es el proceso principal del contenedor). Valores veraces: `1`, `verdadero`, `sí`. Equivalente al indicador CLI `--no-supervise`. No operativo fuera de la imagen s6. |
| `HERMES_GATEWAY_BOOTSTRAP_STATE` | Dentro de la imagen de Docker superpuesta s6, declare el estado supervisado **inicial** de la puerta de enlace en un volumen nuevo. En un volumen en blanco no hay un `gateway_state.json` persistente, por lo que el reconciliador de arranque registra la ranura `gateway-default` pero la deja **inactiva** (solo se inicia automáticamente cuando el último estado registrado fue `running`). Configure esto en "en ejecución" y el enlace de configuración del primer inicio inicializa "gateway_state.json" *antes* de que se ejecute el reconciliador, de modo que la puerta de enlace se active en el primer inicio. Sólo se respeta el valor literal "corriendo". Solo primer arranque: un `gateway_state.json` existente nunca se sobrescribe, por lo que una puerta de enlace detenida deliberadamente permanece detenida durante los reinicios. No operativo fuera de la imagen s6. |
| `GATEWAY_RELAY_URL` | URL base del WebSocket del conector de retransmisión experimental. Cuando está configurada, la puerta de enlace registra el adaptador genérico de "relé" y marca el conector saliente. Refleja `gateway.relay_url` en `config.yaml`. || `GATEWAY_RELAY_ID` | Identificador de puerta de enlace de retransmisión asignado por "hermes gateway enroll" o autoaprovisionamiento gestionado. Refleja `gateway.relay_id`. |
| `GATEWAY_RELAY_SECRET` | Secreto de retransmisión por puerta de enlace utilizado para autenticar el WebSocket. Si esto ya está configurado, se omite el autoaprovisionamiento administrado. Refleja `gateway.relay_secret`. |
| `GATEWAY_RELAY_DELIVERY_KEY` | Se conserva la clave de entrega emitida por el conector para compatibilidad con autenticación de retransmisión/transmisión. Los mensajes entrantes de retransmisión actuales llegan al WebSocket saliente en lugar de a un receptor HTTP del lado de la puerta de enlace. |
| `GATEWAY_RELAY_ENROLL_TOKEN` | Token de inscripción consumido por "hermes gateway enroll" cuando "--token" no se pasa explícitamente. |
| `GATEWAY_RELAY_PLATFORM` | Nombre de plataforma opcional anunciado en el descriptor de capacidad de retransmisión. |
| `GATEWAY_RELAY_BOT_ID` | Identificador de bot opcional anunciado en el descriptor de capacidad de retransmisión. |
| `GATEWAY_RELAY_ENDPOINT` | Punto final de puerta de enlace opcional anunciado para modos de conector que necesitan una URL de devolución de llamada/paso a través; no es necesario para la ruta de retransmisión entrante predeterminada solo para WS. Refleja `gateway.relay_endpoint`. |
| `GATEWAY_RELAY_ROUTE_KEYS` | Claves de ruta de retransmisión separadas por comas anunciadas en el conector. Refleja `gateway.relay_route_keys`. |
| `HERMES_FILE_MUTATION_VERIFIER` | Habilite el pie de página del verificador de mutación de archivos por turno (predeterminado: "verdadero"). Cuando está habilitado, Hermes agrega un aviso que enumera todas las llamadas a `write_file`/`patch` que fallaron durante el turno y no fueron reemplazadas por una escritura exitosa. Establezca en "0", "falso", "no" o "desactivado" para suprimir. Refleja `display.file_mutation_verifier` en `config.yaml`; la var env gana cuando se establece. |
| `HERMES_CRON_TIMEOUT` | El tiempo de espera de inactividad para el agente de trabajos cron se ejecuta en segundos (predeterminado: `600`). El agente puede ejecutarse indefinidamente mientras llama activamente a herramientas o recibe tokens de transmisión; esto solo se activa cuando está inactivo. Establezca en `0` para ilimitado. |
| `HERMES_CRON_SCRIPT_TIMEOUT` | Tiempo de espera para scripts preejecutados adjuntos a trabajos cron en segundos (predeterminado: `120`). Anulación de secuencias de comandos que necesitan una ejecución más prolongada (por ejemplo, retrasos aleatorios para la sincronización anti-bot). También configurable a través de `cron.script_timeout_segundos` en `config.yaml`. |
| `HERMES_CRON_MAX_PARALLEL` | El máximo de trabajos cron se ejecutan en paralelo por tick (predeterminado: `4`). |## Comportamiento del agente| Variables | Descripción |
|----------|-------------|
| `HERMES_MAX_ITERACIONES` | Máximo de iteraciones de llamadas de herramientas por conversación (predeterminado: 90) |
| `HERMES_INFERENCE_MODEL` | Anule el nombre del modelo a nivel de proceso (tiene prioridad sobre `config.yaml` para la sesión). También se puede configurar mediante el indicador `-m`/`--model`. |
| `HERMES_YOLO_MODE` | Establezca en "1" para omitir las indicaciones de aprobación de comandos peligrosos. Equivalente a `--yolo`. |
| `HERMES_ACCEPT_HOOKS` | Apruebe automáticamente cualquier enlace de shell invisible declarado en `config.yaml` sin un mensaje TTY. Equivalente a `--accept-hooks` o `hooks_auto_accept: true`. |
| `HERMES_IGNORE_USER_CONFIG` | Omita `~/.hermes/config.yaml` y use los valores predeterminados integrados (las credenciales en `.env` aún se cargan). Equivalente a `--ignore-user-config`. |
| `HERMES_IGNORE_RULES` | Omita la inyección automática de `AGENTS.md`, `SOUL.md`, `.cursorrules`, memoria y habilidades precargadas. Equivalente a `--ignore-rules`. |
| `HERMES_SAFE_MODE` | Modo de solución de problemas: deshabilite TODAS las personalizaciones: omite el descubrimiento de complementos y la carga del servidor MCP. Se establece automáticamente mediante `--safe-mode` (que también establece los dos indicadores anteriores). |
| `HERMES_MD_NAMES` | Lista separada por comas de nombres de archivos de reglas para autoinyectar (predeterminado: `AGENTS.md,CLAUDE.md,.cursorrules,SOUL.md`). |
| `HERMES_TOOL_PROGRESS` | Variable de compatibilidad obsoleta para la visualización del progreso de la herramienta. Prefiera `display.tool_progress` en `config.yaml`. |
| `HERMES_TOOL_PROGRESS_MODE` | Variable de compatibilidad obsoleta para el modo de progreso de la herramienta. Prefiera `display.tool_progress` en `config.yaml`. |
| `HERMES_HUMAN_DELAY_MODE` | Ritmo de respuesta: `off`/`natural`/`custom` |
| `HERMES_HUMAN_DELAY_MIN_MS` | Mínimo rango de retardo personalizado (ms) |
| `HERMES_HUMAN_DELAY_MAX_MS` | Rango de retardo personalizado máximo (ms) |
| `HERMES_QUIET` | Suprimir la salida no esencial (`true`/`false`) |
| `CODEX_HOME` | Cuando [Codex app-server runtime](../user-guide/features/codex-app-server-runtime) está habilitado, anule el directorio desde el que Codex CLI lee su configuración + autenticación (predeterminado: `~/.codex`). La migración de Hermes escribe el bloque administrado en `<CODEX_HOME>/config.toml`. |
| `HERMES_KANBAN_TASK` | Lo establece el despachador de kanban al generar un trabajador (UUID de tarea). Los trabajadores y el subproceso MCP generado "hermes-tools" lo heredan, por lo que las herramientas kanban se activan correctamente. No lo configures manualmente. |
| `HERMES_API_TIMEOUT` | Tiempo de espera de llamada a la API de LLM en segundos (predeterminado: `1800`) |
| `HERMES_API_CALL_STALE_TIMEOUT` | Tiempo de espera de llamadas obsoletas sin transmisión en segundos (predeterminado: `90`). Se deshabilita automáticamente para proveedores locales cuando no se configura y puede ampliarse en contextos muy grandes. También se puede configurar a través de `providers.<id>.stale_timeout_segundos` o `providers.<id>.models.<model>.stale_timeout_segundos` en `config.yaml`. |
| `HERMES_STREAM_READ_TIMEOUT` | Tiempo de espera de lectura del socket de transmisión en segundos (predeterminado: `120`). Aumento automático a `HERMES_API_TIMEOUT` para proveedores locales. Aumente si los LLM locales agotan el tiempo de espera durante la generación de código largo. |
| `HERMES_STREAM_STALE_TIMEOUT` | Tiempo de espera de detección de flujo obsoleto en segundos (predeterminado: `180`). Desactivado automáticamente para proveedores locales. Activa la interrupción de la conexión si no llegan fragmentos dentro de esta ventana. |
| `HERMES_STREAM_RETRIES` | Número de intentos de reconexión a mitad de camino en errores de red transitorios (predeterminado: "3"). |
| `HERMES_AGENT_TIMEOUT` | Tiempo de espera de inactividad de la puerta de enlace para un agente en ejecución en segundos (predeterminado: `1800`, 30 minutos). Se reinicia en cada llamada de herramienta y token transmitido. Establezca en `0` para desactivar. |
| `HERMES_AGENT_TIMEOUT_WARNING` | Puerta de enlace: envía un mensaje de advertencia después de tantos segundos de inactividad (predeterminado: 75% de `HERMES_AGENT_TIMEOUT`). |
| `HERMES_AGENT_NOTIFY_INTERVAL` | Puerta de enlace: intervalo en segundos entre notificaciones de progreso en turnos de agente de larga duración. |
| `HERMES_CHECKPOINT_TIMEOUT` | Tiempo de espera para la creación del punto de control del sistema de archivos en segundos (predeterminado: `30`). |
| `HERMES_EXEC_ASK` | Habilitar mensajes de aprobación de ejecución en modo puerta de enlace (`true`/`false`) |
| `HERMES_ENABLE_PROJECT_PLUGINS` | Habilite el descubrimiento automático de complementos locales de repositorio desde `./.hermes/plugins/` tanto para el cargador del agente como para el servidor web del panel. Acepta el conjunto veraz estándar: `1`/`true`/`yes`/`on` (no distingue entre mayúsculas y minúsculas). Todo lo demás, incluido `0`, `false`, `no`, `off` y la cadena vacía, se trata como **deshabilitado** (predeterminado). Nota: a partir de GHSA-5qr3-c538-wm9j (#29156), el servidor web del panel se niega a importar automáticamente el archivo `api` de Python de un complemento de proyecto incluso cuando esta var está habilitada; los complementos de proyecto pueden extender la interfaz de usuario a través de JS/CSS estático, pero sus rutas de backend solo se cargan cuando se mueven bajo `~/.hermes/plugins/`. || `HERMES_PLUGINS_DEBUG` | `1`/`true` para mostrar registros detallados de descubrimiento de complementos en stderr: directorios escaneados, manifiestos analizados, motivos de omisión y rastreos completos en caso de error de análisis o `register()`. Dirigido a autores de complementos. |
| `HERMES_BACKGROUND_NOTIFICATIONS` | Modo de notificación de proceso en segundo plano en la puerta de enlace: `todos` (predeterminado), `resultado`, `error`, `apagado` |
| `HERMES_EPHEMERAL_SYSTEM_PROMPT` | Aviso efímero del sistema inyectado en el momento de la llamada a la API (nunca persiste en las sesiones) |
| `HERMES_PREFILL_MESSAGES_FILE` | Ruta a un archivo JSON de mensajes de precarga efímeros inyectados en el momento de la llamada a la API. |
| `HERMES_ALLOW_PRIVATE_URLS` | `true`/`false`: permite que las herramientas obtengan URL de host local/red privada. Desactivado de forma predeterminada en el modo puerta de enlace. |
| `HERMES_REDACT_SECRETS` | `true`/`false`: controla la redacción de secretos en la salida de la herramienta, los registros y las respuestas del chat (predeterminado: `true`). |
| `HERMES_WRITE_SAFE_ROOT` | Prefijo de directorio opcional que restringe las escrituras en `write_file`/`patch`; Los caminos exteriores requieren aprobación. |
| `HERMES_DISABLE_LAZY_INSTALLS` | La variable de puente interna se establece automáticamente en la imagen oficial de Docker para evitar que la dependencia del tiempo de ejecución se instale en el árbol inmutable `/opt/hermes`. El equivalente de cara al usuario es `security.allow_lazy_installs: false` en `config.yaml`; no configure esto en `.env`. |
| `HERMES_DISABLE_FILE_STATE_GUARD` | Establezca en `1` para desactivar la protección "archivo cambiado desde que lo leyó" en `patch`/`write_file`. |
| `HERMES_CORE_TOOLS` | Anulación separada por comas para la lista de herramientas principales canónicas (avanzada; rara vez es necesaria). |
| `HERMES_BUNDLED_SKILLS` | Anulación separada por comas para la lista de habilidades incluidas cargadas al inicio. |
| `HERMES_OPCIONAL_SKILLS` | Lista separada por comas de nombres de habilidades opcionales para instalar automáticamente en la primera ejecución. |
| `HERMES_DEBUG_INTERRUPT` | Establezca en `1` para registrar el seguimiento detallado de interrupciones/cancelaciones en `agent.log`. |
| `HERMES_DUMP_REQUESTS` | Volcar cargas útiles de solicitudes de API en archivos de registro (`true`/`false`) |
| `HERMES_DUMP_REQUEST_STDOUT` | Vuelque las cargas útiles de las solicitudes de API en la salida estándar en lugar de en los archivos de registro. |
| `HERMES_OAUTH_TRACE` | Establezca en `1` para registrar el intercambio de tokens de OAuth y los intentos de actualización. Incluye información de tiempo redactada. |
| `HERMES_OAUTH_FILE` | Anule la ruta utilizada para el almacenamiento de credenciales de OAuth (predeterminada: `~/.hermes/auth.json`). |
| `HERMES_AGENT_HELP_GUIDANCE` | Agregue texto de orientación adicional al mensaje del sistema para implementaciones personalizadas. |
| `HERMES_AGENT_LOGO` | Anule el logotipo del banner ASCII al iniciar CLI. |
| `DELEGATION_MAX_CONCURRENT_CHILDREN` | Número máximo de subagentes paralelos por lote "delegate_task" (predeterminado: "3", mínimo de 1, sin límite máximo). También se puede configurar a través de `delegation.max_concurrent_children` en `config.yaml`: el valor de configuración tiene prioridad. |## Interfaz| Variables | Descripción |
|----------|-------------|
| `HERMES_TUI` | Inicie [TUI](../user-guide/tui.md) en lugar de la CLI clásica cuando esté configurada en `1`. Equivale a pasar `--tui`. |
| `HERMES_TUI_DIR` | Ruta a un directorio `ui-tui/` prediseñado (debe contener `dist/entry.js` y `node_modules` completos). Utilizado por distribuciones y Nix para omitir la `instalación npm` del primer inicio. |
| `HERMES_TUI_RESUME` | Reanude una sesión TUI específica por ID al iniciar. Cuando está configurado, `hermes --tui` omite la creación de una sesión nueva y, en su lugar, retoma la sesión nombrada, lo que resulta útil para volver a conectarse después de una desconexión o falla del terminal. |
| `HERMES_TUI_THEME` | Fuerce el tema de color de la TUI: "claro", "oscuro" o un fondo hexadecimal sin formato de 6 caracteres (por ejemplo, "ffffff" o "1a1a2e"). Cuando no está configurado, Hermes detecta automáticamente el uso de `COLORFGBG` y consultas en segundo plano del terminal; esta variable anula la detección en terminales (Ghostty, Warp, iTerm2, etc.) que no configuran `COLORFGBG`. |
| `HERMES_INFERENCE_MODEL` | Fuerce el modelo para `hermes -z` / `hermes chat` sin mutar `config.yaml`. Se empareja con la bandera `--provider`. Útil para llamadores con script (barredora, CI, ejecutores por lotes) que necesitan anular el modelo predeterminado por ejecución. |## Configuración de sesión| Variables | Descripción |
|----------|-------------|
| `SESSION_IDLE_MINUTES` | Restablecer sesiones después de N minutos de inactividad (predeterminado: 1440) |
| `SESSION_RESET_HOUR` | Hora de reinicio diario en formato de 24 horas (predeterminado: 4 = 4 a.m.) |
| `HERMES_SESSION_ID` | **Exportado automáticamente a cada subproceso de herramienta** Hermes genera (`terminal`, `execute_code`, shell persistente, backends de Docker/Singularity, ejecuciones de subagente delegado). Establecido por el agente en el ID de sesión actual; Los scripts de usuario llamados desde herramientas pueden leerlo para correlacionar su salida, telemetría o efectos secundarios con la sesión de Hermes de origen. **No debe configurar esto manualmente**: anularlo desde un shell principal solo tiene efecto fuera de la ejecución del agente y se sobrescribe en el momento en que el agente inicia una sesión. |## Compresión de contexto (solo config.yaml)La compresión de contexto se configura exclusivamente a través de `config.yaml`; no hay variables de entorno para ello. La configuración de umbral se encuentra en el bloque `compresión:`, mientras que el modelo/proveedor de resumen se encuentra en `auxiliary.compression:`.```yaml
compresión:
  habilitado: verdadero
  umbral: 0,50
  target_ratio: 0,20 # fracción del umbral para conservar como cola reciente
  protect_last_n: 20 # mensajes mínimos recientes para mantener sin comprimir
```:::info Migración heredada
Las configuraciones más antiguas con `compression.summary_model`, `compression.summary_provider` y `compression.summary_base_url` se migran automáticamente a `auxiliary.compression.*` en la primera carga.
:::## Anulaciones de tareas auxiliares| Variables | Descripción |
|----------|-------------|
| `AUXILIARY_VISION_PROVIDER` | Anular proveedor para tareas de visión |
| `MODEL_VISIÓN_AUXILIAR` | Modelo de anulación para tareas de visión |
| `AUXILIARY_VISION_BASE_URL` | Punto final directo compatible con OpenAI para tareas de visión |
| `AUXILIARY_VISION_API_KEY` | Clave API emparejada con `AUXILIARY_VISION_BASE_URL` |
| `AUXILIARY_WEB_EXTRACT_PROVIDER` | Anular proveedor para extracción/resumen web |
| `AUXILIARY_WEB_EXTRACT_MODEL` | Anular modelo para extracción/resumen web |
| `AUXILIARY_WEB_EXTRACT_BASE_URL` | Punto final directo compatible con OpenAI para extracción/resumen web |
| `AUXILIARY_WEB_EXTRACT_API_KEY` | Clave API emparejada con `AUXILIARY_WEB_EXTRACT_BASE_URL` |Para puntos finales directos específicos de tareas, Hermes utiliza la clave API configurada de la tarea o `OPENAI_API_KEY`. No reutiliza `OPENROUTER_API_KEY` para esos puntos finales personalizados.## Proveedores alternativos (solo config.yaml)La cadena de respaldo del modelo principal se configura exclusivamente a través de `config.yaml`; no hay variables de entorno para ella. Agregue una lista de nivel superior `fallback_providers` con claves de `proveedor` y `modelo` para habilitar la conmutación por error automática cuando su modelo principal encuentre errores. Las tareas auxiliares cuyo proveedor es "automático" también consultan esta cadena antes que la cadena de descubrimiento auxiliar incorporada de Hermes.```yaml
proveedores_alternativos:
  - proveedor: enrutador abierto
    modelo: antrópico/claude-sonnet-4
```La antigua forma de proveedor único de nivel superior `fallback_model` todavía se lee para compatibilidad con versiones anteriores, pero la nueva configuración debe usar `fallback_providers`. Para una política auxiliar específica de la tarea, use `auxiliary.<task>.fallback_chain` en `config.yaml`; no existe ninguna variable de entorno equivalente.Consulte [Proveedores alternativos](/user-guide/features/fallback-providers) para obtener detalles completos.## Enrutamiento del proveedor (solo config.yaml)Estos van en `~/.hermes/config.yaml` en la sección `provider_routing`:| Clave | Descripción |
|-----|-------------|
| `ordenar` | Ordenar proveedores: `"precio"` (predeterminado), `"rendimiento"` o `"latencia"` |
| `sólo` | Lista de slugs de proveedores que se permitirán (por ejemplo, `["anthropic", "google"]`) |
| `ignorar` | Lista de babosas de proveedores que se deben omitir |
| `orden` | Lista de babosas de proveedores para probar en orden |
| `requerir_parámetros` | Utilice únicamente proveedores que admitan todos los parámetros de solicitud (`true`/`false`) |
| `recopilación_datos` | `"permitir"` (predeterminado) o `"denegar"` excluir proveedores de almacenamiento de datos |:::consejo
Utilice `hermes config set` para configurar las variables de entorno; las guarda automáticamente en el archivo correcto (`.env` para secretos, `config.yaml` para todo lo demás).
:::---