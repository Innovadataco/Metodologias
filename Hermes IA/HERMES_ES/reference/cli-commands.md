<!-- fuente: sitio web/docs/reference/cli-commands.md -->
# Referencia de comandos CLI# Referencia de comandos CLIEsta página cubre los **comandos de terminal** que ejecuta desde su shell.Para comandos de barra diagonal en el chat, consulte [Referencia de comandos de barra diagonal] (./slash-commands.md).## Punto de entrada global```golpecito
hermes [opciones-globales] <comando> [subcomando/opciones]
```### Opciones globales| Opción | Descripción |
|--------|-------------|
| `--versión`, `-V` | Mostrar versión y salir. |
| `--profile <nombre>`, `-p <nombre>` | Seleccione qué perfil de Hermes usar para esta invocación. Anula el valor predeterminado fijo establecido por "uso del perfil de Hermes". |
| `--resume <sesión>`, `-r <sesión>` | Reanudar una sesión anterior por ID o título. |
| `--continuar [nombre]`, `-c [nombre]` | Reanudar la sesión más reciente o la sesión más reciente que coincida con un título. |
| `--worktree`, `-w` | Comience en un árbol de trabajo de Git aislado para flujos de trabajo de agentes paralelos. |
| `--yolo` | Omita las indicaciones de aprobación de comandos peligrosos. |
| `--pass-id-sesión` | Incluya el ID de la sesión en el mensaje del sistema del agente. |
| `--ignore-user-config` | Ignore `~/.hermes/config.yaml` y recurra a los valores predeterminados integrados. Las credenciales en `.env` todavía están cargadas. |
| `--ignorar-reglas` | Omita la inyección automática de `AGENTS.md`, `SOUL.md`, `.cursorrules`, memoria y habilidades precargadas. |
| `--tui` | Inicie [TUI](../user-guide/tui.md) en lugar de la CLI clásica. Equivalente a `HERMES_TUI=1`. Siempre gana sobre `display.interface`. |
| `--cli` | Fuerce el REPL clásico de Prompt_Toolkit. Utilice esto para anular `display.interface: tui` para una única invocación. |
| `--dev` | Con `--tui`: ejecute las fuentes de TypeScript directamente a través de `tsx` en lugar del paquete prediseñado (para contribuyentes de TUI). |## Comandos de nivel superior| Comando | Propósito |
|---------|---------|
| `chat de hermes` | Chat interactivo o one-shot con el agente. |
| `modelo hermes` | Elija de forma interactiva el proveedor y el modelo predeterminados. |
| `refuerzo de Hermes` | Administrar los proveedores alternativos que se intentaron cuando el modelo principal presenta errores. |
| `puerta de enlace de hermes` | Ejecute o administre el servicio de puerta de enlace de mensajería. |
| `hermes proxy` | Proxy local compatible con OpenAI que adjunta credenciales de proveedor de OAuth. Consulte [Proxy de suscripción](../user-guide/features/subscription-proxy.md). |
| `hermes lsp` | Administrar la integración del protocolo de servidor de idiomas (diagnóstico semántico para write_file/parche). |
| `configuración de Hermes` | Asistente de configuración interactivo para toda o parte de la configuración. |
| `hermes whatsapp` | Configura y empareja el puente de WhatsApp. |
| `hermes flojo` | Ayudantes de Slack (actualmente: generan el manifiesto de la aplicación con cada comando como una barra diagonal nativa). |
| `hermes autenticación` | Administre credenciales: agregue, enumere, elimine, restablezca, estado, cierre sesión. Maneja flujos de OAuth para Codex/Nous/Anthropic. |
| `hermes iniciar sesión` / `cerrar sesión` | **Obsoleto**: utilice `hermes auth` en su lugar. |
| `hermes envía` | Envía un mensaje único a una plataforma de mensajería configurada (Telegram, Discord, Slack, Signal, SMS,…). Útil para scripts de shell, trabajos cron, enlaces de CI y demonios de monitoreo: sin bucle de agente ni LLM. |
| `secretos de hermes` | Administre fuentes secretas externas (actualmente Bitwarden Secrets Manager) para extraer claves API al inicio del proceso en lugar de hacerlo desde `~/.hermes/.env`. |
| `hermes migra` | Diagnosticar y (opcionalmente) reescribir `config.yaml` para reemplazar referencias a modelos retirados o configuraciones obsoletas (por ejemplo, `migrar xai`). |
| `estado de hermes` | Muestra el estado del agente, la autenticación y la plataforma. |
| `hermes cron` | Inspeccione y marque el programador cron. |
| `hermes kanban` | Tablero de colaboración multiperfil (tareas, enlaces, despachador). |
| `hermes webhook` | Administre suscripciones dinámicas a webhooks para activación basada en eventos. |
| `ganchos de hermes` | Inspeccione, apruebe o elimine los enlaces de script de shell declarados en `config.yaml`. |
| `doctor hermes` | Diagnosticar problemas de configuración y dependencia. |
| `auditoría de seguridad de hermes` | Auditoría de la cadena de suministro bajo demanda (OSV.dev) para venv, requisitos de complementos y servidores MCP anclados. |
| `vertedero de hermes` | Resumen de configuración que se puede copiar y pegar para soporte/depuración. |
| `hermes pronto-tamaño` | Muestra un desglose de bytes del mensaje del sistema + esquemas de herramientas (índice de habilidades, memoria, perfil). Se ejecuta sin conexión. |
| `depuración de hermes` | Herramientas de depuración: cargue registros e información del sistema para obtener soporte. |
| `copia de seguridad de Hermes` | Haga una copia de seguridad del directorio de inicio de Hermes en un archivo zip. |
| `puntos de control de hermes` | Inspeccionar / podar / borrar `~/.hermes/checkpoints/` (el almacén oculto utilizado por `/rollback`). Ejecute sin argumentos para obtener una descripción general del estado. |
| `hermes importación` | Restaure una copia de seguridad de Hermes desde un archivo zip. |
| `registros de hermes` | Ver, seguir y filtrar archivos de registro de errores, puertas de enlace y agentes. |
| `configuración de hermes` | Mostrar, editar, migrar y consultar archivos de configuración. |
| `emparejamiento de hermes` | Aprobar o revocar códigos de emparejamiento de mensajería. |
| `habilidades de hermes` | Explore, instale, publique, audite y configure habilidades. |
| `paquetes de hermes` | Agrupe varias habilidades bajo un solo comando de barra diagonal `/<nombre>`. Consulte [Paquetes de habilidades](../user-guide/features/skills.md#skill-bundles). |
| `curador de hermes` | Mantenimiento de habilidades en segundo plano: estado, ejecución, pausa, fijación. Consulte [Curador](../user-guide/features/curator.md). |
| `memoria de hermes` | Configurar el proveedor de memoria externa. Los subcomandos específicos del complemento (por ejemplo, `hermes honcho`) se registran automáticamente cuando su proveedor está activo. |
| `hermes acp` | Ejecute Hermes como servidor ACP para la integración del editor. |
| `hermes mcp` | Administre las configuraciones del servidor MCP y ejecute Hermes como servidor MCP. |
| `complementos de hermes` | Administre los complementos del Agente Hermes (instalar, habilitar, deshabilitar, eliminar). |
| `portal de hermes` | Estado de Nous Portal, enlace de suscripción y enrutamiento de Tool Gateway. Consulte [Puerta de enlace de herramientas] (../user-guide/features/tool-gateway.md). |
| `herramientas hermes` | Configurar herramientas habilitadas por plataforma. |
| `hermes uso de la computadora` | Instale o verifique el backend del cua-driver (uso de computadora macOS). || `sesiones de hermes` | Explore, exporte, elimine, cambie el nombre y elimine sesiones. |
| `perspectivas de Hermes` | Mostrar análisis de token/coste/actividad. |
| `garra de hermes` | Ayudantes de migración de OpenClaw. |
| `tablero de Hermes` | Inicie el panel web para administrar la configuración, las claves API y las sesiones. |
| `perfil de hermes` | Administre perfiles: múltiples instancias aisladas de Hermes. |
| `terminación de hermes` | Imprima scripts de finalización de shell (bash/zsh/fish). |
| `versión hermes` | Mostrar información de la versión. |
| `actualización de Hermes` | Extraiga el código más reciente y reinstale las dependencias. `--check` vistas previas sin instalar; `--backup` toma una instantánea de `HERMES_HOME` previa a la extracción. |
| `desinstalación de hermes` | Elimine Hermes del sistema. |## `charla de hermes````golpecito
chat de hermes [opciones]
```Opciones comunes:| Opción | Descripción |
|--------|-------------|
| `-q`, `--query "..."` | Mensaje único y no interactivo. |
| `-m`, `--model <modelo>` | Anule el modelo para esta ejecución. |
| `-t`, `--toolsets <csv>` | Habilite un conjunto de conjuntos de herramientas separados por comas. |
| `--proveedor <proveedor>` | Forzar un proveedor: `auto`, `openrouter`, `nous`, `openai-codex`, `copilot-acp`, `copilot`, `anthropic`, `gemini`, `huggingface`, `novita` (alias `novita-ai`, `novitaai`), `openai-api`, `zai`, `kimi-coding`, `kimi-coding-cn`, `minimax`, `minimax-cn`, `minimax-oauth`, `kilocode`, `xiaomi`, `arcee`, `gmi`, `alibaba`, `alibaba-coding-plan` (alias `alibaba_coding`), `deepseek`, `nvidia`, `ollama-cloud`, `xai` (alias `grok`), `xai-oauth` (alias `grok-oauth`), `qwen-oauth`, `bedrock`, `opencode-zen`, `opencode-go`, `azure-foundry`, `lmstudio`, `stepfun`, `tencent-tokenhub` (alias `tencent`, `tokenhub`). |
| `-s`, `--skills <nombre>` | Precargue una o más habilidades para la sesión (se pueden repetir o separar por comas). |
| `-v`, `--verbose` | Salida detallada. |
| `-Q`, `--quiet` | Modo programático: suprime las vistas previas de banner/spinner/herramientas. |
| `--image <ruta>` | Adjunte una imagen local a una sola consulta. |
| `--reanudar <sesión>` / `--continuar [nombre]` | Reanudar una sesión directamente desde `chat`. |
| `--árbol de trabajo` | Cree un árbol de trabajo de git aislado para esta ejecución. |
| `--puntos de control` | Habilite los puntos de control del sistema de archivos antes de cambios destructivos en los archivos. |
| `--yolo` | Omita las indicaciones de aprobación. |
| `--pass-id-sesión` | Pase el ID de la sesión al indicador del sistema. |
| `--ignore-user-config` | Ignore `~/.hermes/config.yaml` y use los valores predeterminados integrados. Las credenciales en `.env` todavía están cargadas. Útil para ejecuciones de CI aisladas, informes de errores reproducibles e integraciones de terceros. |
| `--ignorar-reglas` | Omita la inyección automática de `AGENTS.md`, `SOUL.md`, `.cursorrules`, memoria persistente y habilidades precargadas. Combínelo con `--ignore-user-config` para una ejecución completamente aislada. |
| `--modo-seguro` | Modo de solución de problemas: deshabilite TODAS las personalizaciones: configuración de usuario, reglas/inyección de memoria, complementos y servidores MCP (implica `--ignore-user-config` y `--ignore-rules`). Úselo para aislar si un problema proviene de su configuración o del propio Hermes. |
| `--fuente <etiqueta>` | Etiqueta de origen de sesión para filtrado (predeterminado: `cli`). Utilice la "herramienta" para integraciones de terceros que no deberían aparecer en las listas de sesiones de usuarios. |
| `--max-turns <N>` | Iteraciones máximas de llamada de herramientas por turno de conversación (predeterminado: 90 o `agent.max_turns` en la configuración). |Ejemplos:```golpecito
hermes
hermes chat -q "Resumir las últimas relaciones públicas"
hermes chat --proveedor openrouter --modelo antrópico/claude-sonnet-4.6
hermes chat --conjuntos de herramientas web,terminal,habilidades
hermes chat --quiet -q "Devolver solo JSON"
hermes chat --worktree -q "Revisar este repositorio y abrir un PR"
hermes chat --ignore-user-config --ignore-rules -q "Repro sin mi configuración personal"
hermes chat --safe-mode -q "¿Este error es mío o de Hermes?"
```### `hermes -z <prompt>` — one-shot con guiónPara los llamadores programáticos (scripts de shell, CI, cron, procesos principales canalizados en un mensaje), `hermes -z` es el punto de entrada de un solo paso más puro: **entrada de mensaje único, salida de texto de respuesta final, nada más en stdout o stderr.** Sin banner, sin control giratorio, sin vistas previas de herramientas, sin línea `Session:`; solo la respuesta final del agente como texto sin formato.```golpecito
hermes -z "¿Cuál es la capital de Francia?"
# → París.# Los scripts principales pueden capturar claramente la respuesta:
respuesta=$(hermes -z "resumir esto" < /ruta/al/archivo.txt)
```Anulaciones por ejecución (sin mutación en `~/.hermes/config.yaml`):| Bandera | var env equivalente | Propósito |
|---|---|---|
| `-m` / `--model <modelo>` | `HERMES_INFERENCE_MODEL` | Anular el modelo para esta ejecución |
| `--proveedor <proveedor>` | _(ninguno)_ | Anule el proveedor para esta ejecución |```golpecito
hermes -z "..." --proveedor openrouter --modelo openai/gpt-5.5
# o:
HERMES_INFERENCE_MODEL=antrópico/claude-sonnet-4.6 hermes -z "…"
```El mismo agente, las mismas herramientas, las mismas habilidades: simplemente elimina cada capa interactiva/cosmética. Si también necesita resultados de herramientas en la transcripción, utilice `hermes chat -q` en su lugar; `-z` es explícitamente para "Solo quiero la respuesta final".## `modelo hermes`Proveedor interactivo + selector de modelo. **Este es el comando para agregar nuevos proveedores, configurar claves API y ejecutar flujos de OAuth.** Ejecútelo desde su terminal, no desde una sesión de chat activa de Hermes.```golpecito
modelo hermes
```Utilízalo cuando quieras:
- **agregar un nuevo proveedor** (OpenRouter, Anthropic, Copilot, DeepSeek, personalizado, etc.)
- iniciar sesión en proveedores respaldados por OAuth (Anthropic, Copilot, Codex, Nous Portal)
- ingresar o actualizar claves API
- elija entre listas de modelos específicos del proveedor
- configurar un punto final personalizado/autohospedado
- guarde el nuevo valor predeterminado en la configuración:::advertencia modelo hermes vs /modelo: conozca la diferencia
**`hermes model`** (ejecutado desde su terminal, fuera de cualquier sesión de Hermes) es el **asistente de configuración completo del proveedor**. Puede agregar nuevos proveedores, ejecutar flujos de OAuth, solicitar claves API y configurar puntos finales.**`/model`** (escrito dentro de una sesión de chat activa de Hermes) solo puede **cambiar entre proveedores y modelos que ya haya configurado**. No puede agregar nuevos proveedores, ejecutar OAuth ni solicitar claves API.**Si necesita agregar un nuevo proveedor:** Primero salga de su sesión de Hermes (`Ctrl+C` o `/quit`), luego ejecute `hermes model` desde el indicador de su terminal.
:::### Comando de barra diagonal `/model` (a mitad de sesión)Cambie entre modelos ya configurados sin salir de una sesión:```
/model # Muestra el modelo actual y las opciones disponibles
/model claude-sonnet-4 # Modelo de conmutador (proveedor de detección automática)
/model zai:glm-5 # Cambia de proveedor y modelo
/model custom:qwen-2.5 # Utilice el modelo en su punto final personalizado
/model custom # Detectar automáticamente el modelo desde el punto final personalizado
/model custom:local:qwen-2.5 # Utilice un proveedor personalizado con nombre
/model openrouter:anthropic/claude-sonnet-4 # Volver a la nube
```De forma predeterminada, los cambios en `/model` se aplican **solo a la sesión actual**. Agregue `--global` para conservar el cambio en `config.yaml`:```
/model claude-sonnet-4 --global # Cambiar y guardar como nuevo valor predeterminado
```:::info ¿Qué pasa si solo veo modelos OpenRouter?
Si solo ha configurado OpenRouter, `/model` solo mostrará los modelos de OpenRouter. Para agregar otro proveedor (Anthropic, DeepSeek, Copilot, etc.), salga de su sesión y ejecute `hermes model` desde la terminal.
:::Los cambios de URL base y de proveedor se conservan en `config.yaml` automáticamente. Al cambiar de un punto final personalizado, la URL base obsoleta se borra para evitar que se filtre a otros proveedores.## `puerta de enlace de hermes````golpecito
puerta de enlace de hermes <subcomando>
```Subcomandos:| Subcomando | Descripción |
|------------|-------------|
| `ejecutar` | Ejecute la puerta de enlace en primer plano. Recomendado para WSL, Docker y Termux. |
| `inicio` | Inicie el servicio en segundo plano systemd/launchd instalado. |
| `detener` | Detenga el servicio (o proceso en primer plano). |
| `reiniciar` | Reinicie el servicio. |
| `estado` | Mostrar el estado del servicio. |
| `lista` | Enumere **todos los perfiles** y si la puerta de enlace de cada perfil se está ejecutando actualmente (con PID cuando esté disponible). Útil cuando ejecuta varios perfiles uno al lado del otro y desea una descripción general única. |
| `instalar` | Instálelo como un servicio en segundo plano systemd (Linux) o launchd (macOS). |
| `desinstalar` | Eliminar el servicio instalado. |
| `configuración` | Configuración de plataforma de mensajería interactiva. |
| `inscribirse` | Experimental: inscriba esta puerta de enlace con un conector de retransmisión y guarde las credenciales de retransmisión para plataformas respaldadas por conector. |Opciones:| Opción | Descripción |
|--------|-------------|
| `--todos` | Al `iniciar`/`reiniciar`/`detener`: actúe en la puerta de enlace de **cada perfil**, no solo en el `HERMES_HOME` activo. Útil si ejecuta varios perfiles uno al lado del otro y desea reiniciarlos todos después de la "actualización de Hermes". |
| `--no-supervisar` | En "ejecutar": dentro de la imagen de Docker superpuesta de s6, opte por no participar en la supervisión automática y utilice la semántica de primer plano anterior a s6: la puerta de enlace se ejecuta como el proceso principal del contenedor sin reinicio automático. No operativo fuera de la imagen s6. Equivale a configurar `HERMES_GATEWAY_NO_SUPERVISE=1`. |`hermes gateway enroll` acepta `--token`, `--connector-url` y `--gateway-id`. Intercambia el token de inscripción con el conector y escribe los valores resultantes `GATEWAY_RELAY_ID`, `GATEWAY_RELAY_SECRET`, `GATEWAY_RELAY_DELIVERY_KEY` y opcionales `GATEWAY_RELAY_URL` en el `.env` del perfil activo.:::consejo para usuarios de WSL
Utilice `hermes gateway run` en lugar de `hermes gateway start`: el soporte systemd de WSL no es confiable. Envuélvalo en tmux para persistencia: `tmux new -s hermes 'hermes gateway run'`. Consulte las [Preguntas frecuentes sobre WSL](/reference/faq#wsl-gateway-keeps-disconnecting-or-hermes-gateway-start-fails) para obtener más detalles.
:::## `hermes lsp````golpecito
hermes lsp <subcomando>
```Gestionar la integración del protocolo de servidor de idiomas. LSP funciona de verdad
servidores de idiomas (pyright, gopls, Rust-analyzer,…) en el
antecedentes e introduce sus diagnósticos en la verificación posterior a la escritura
utilizado por `write_file` y `patch`. Cerrado en la detección del espacio de trabajo de git
— LSP solo se ejecuta cuando el archivo cwd o editado está dentro de un git
árbol de trabajo.Subcomandos:| Subcomando | Descripción |
|------------|-------------|
| `estado` | Muestra el estado del servicio, servidores configurados, estado de instalación. |
| `lista` | Imprima el registro de servidores compatibles. Pase `--installed-only` para omitir los que faltan. |
| `instalar <id>` | Instale con entusiasmo el binario de un servidor. |
| `instalar todo` | Instale cada servidor con una receta de instalación automática conocida. |
| `reiniciar` | Elimina los clientes en ejecución para que vuelva a aparecer la siguiente edición. |
| `cual <id>` | Imprima la ruta binaria resuelta para un servidor. |Consulte [LSP — Diagnóstico semántico](/user-guide/features/lsp) para
la guía completa, los idiomas admitidos y los botones de configuración.## `configuración de Hermes````golpecito
configuración de hermes [modelo|tts|terminal|puerta de enlace|herramientas|agente] [--no interactivo] [--reset] [--quick] [--reconfigure] [--portal]
```**Ruta más fácil:** `hermes setup --portal` — OAuth en Nous Portal y opte por [Tool Gateway](../user-guide/features/tool-gateway.md) de una sola vez.**Primera ejecución:** inicia el asistente por primera vez.**Usuario recurrente (ya configurado):** ingresa directamente al asistente de reconfiguración completo: cada mensaje muestra su valor actual como predeterminado, presione Entrar para conservar o escriba un nuevo valor. Sin menú.Vaya a una sección en lugar del asistente completo:| Sección | Descripción |
|---------|-------------|
| `modelo` | Configuración de proveedores y modelos. |
| `terminal` | Configuración del terminal backend y sandbox. |
| `puerta de enlace` | Configuración de plataforma de mensajería. |
| `herramientas` | Activar/desactivar herramientas por plataforma. |
| `agente` | Configuración del comportamiento del agente. |Opciones:| Opción | Descripción |
|--------|-------------|
| `--rápido` | En ejecuciones de usuarios que regresan: solo solicita elementos que faltan o no están configurados. Omita los elementos que ya haya configurado. |
| `--no interactivo` | Utilice valores predeterminados/de entorno sin indicaciones. |
| `--restablecer` | Restablezca la configuración a los valores predeterminados antes de la instalación. |
| `--reconfigurar` | Alias ​​de compatibilidad con versiones anteriores: la `configuración de hermes` básica en una instalación existente ahora hace esto de forma predeterminada. |
| `--portal` | Configuración única de Nous Portal: inicie sesión a través de OAuth, configure Nous como proveedor de inferencia y opte por [Tool Gateway] (../user-guide/features/tool-gateway.md). Se salta el resto del asistente. |## `portal de hermes````golpecito
portal hermes [estado|abierto|herramientas]
```Inspeccione la autenticación de Nous Portal, el enrutamiento de Tool Gateway y acceda a la página de suscripción. La invocación sin subcomando ejecuta "estado".| Subcomando | Descripción |
|------------|-------------|
| `estado` (predeterminado) | Estado de autenticación del portal + resumen de enrutamiento de Tool Gateway por herramienta. También se muestra cuando no se proporciona ningún subcomando. |
| `abierto` | Abra `portal.nousresearch.com/manage-subscription` en su navegador predeterminado. |
| `herramientas` | Enumere todos los socios de Tool Gateway (Firecrawl, FAL, OpenAI TTS, Browser Use, Modal) y cuáles se enrutan a través de Nous. |Para la configuración de la puerta de enlace en sí, consulte [Tool Gateway](../user-guide/features/tool-gateway.md). Para conocer la ruta de configuración única, consulte `hermes setup --portal` más arriba.## `hermes whatsapp````golpecito
hermes whatsapp
```Ejecuta el flujo de configuración/emparejamiento de WhatsApp, incluida la selección de modo y el emparejamiento de código QR.## `hermes flojo````golpecito
manifiesto de holgura de hermes # imprimir manifiesto en la salida estándar
manifiesto de holgura de hermes --escribir # escribir en ~/.hermes/slack-manifest.json
manifiesto de holgura de hermes --slashes-only # solo la matriz características.slash_commands
```Genera un manifiesto de aplicación de Slack que registra cada comando de puerta de enlace en
`COMMAND_REGISTRY` (`/btw`, `/stop`, `/model`, …) como primera clase
Comando de barra diagonal floja: iguala la paridad de Discord y Telegram. Pega el
salida a la configuración de su aplicación Slack en
[https://api.slack.com/apps](https://api.slack.com/apps) → tu aplicación →
**Características → Manifiesto de aplicación → Editar**, luego **Guardar**. Indicaciones flojas para
reinstale si los alcances o los comandos de barra cambiaron.| Bandera | Predeterminado | Propósito |
|------|---------|---------|
| `--escribir [RUTA]` | salida estándar | Escriba en un archivo en lugar de en la salida estándar. El `--write` desnudo escribe `$HERMES_HOME/slack-manifest.json`. |
| `--nombre NOMBRE` | `Hermes` | Nombre para mostrar del bot en Slack. |
| `--descripción DESC` | propaganda predeterminada | Descripción del bot que se muestra en el directorio de la aplicación Slack. |
| `--sólo barras diagonales` | apagado | Emita solo `features.slash_commands` para fusionarlos en un manifiesto mantenido manualmente. |Ejecute `hermes slack manifest --write` nuevamente después de `hermes update` para seleccionar
cualquier comando nuevo.
## `hermes envía````golpecito
hermes envía --a <destino> "texto del mensaje"
hermes envía --a <destino> --archivo <ruta>
eco "mensaje" | hermes envía --a <destino>
hermes enviar --lista [plataforma]
```Envíe un mensaje único a una plataforma de mensajería configurada sin activar un bucle de agente o puerta de enlace. Reutiliza las credenciales ya configuradas de la puerta de enlace (`~/.hermes/.env` + `~/.hermes/config.yaml`) para que los scripts de operaciones, trabajos cron, enlaces de CI y demonios de monitoreo puedan publicar actualizaciones de estado sin volver a implementar el cliente REST de cada plataforma.Para las plataformas de token de bot (Telegram, Discord, Slack, Signal, SMS, WhatsApp-CloudAPI) no se requiere una puerta de enlace en ejecución: "hermes envía" habla directamente con el punto final REST de la plataforma. Las plataformas de complementos que necesitan un adaptador persistente aún requieren una puerta de enlace activa.| Opción | Descripción |
|--------|-------------|
| `-t`, `--a <OBJETIVO>` | Objetivo de entrega. Formatos: `plataforma` (usa el canal de inicio), `plataforma:chat_id`, `plataforma:chat_id:thread_id` o `plataforma:#nombre-canal`. Ejemplos: `telegrama`, `telegrama:-1001234567890`, `discord:#ops`, `slack:C0123ABCD`, `señal:+15551234567`. |
| `-f`, `--file <RUTA>` | Lea el cuerpo del mensaje desde `PATH` (solo archivos de texto: registros, informes, rebajas). Pase `-` para forzar la lectura desde la entrada estándar. Para enviar una imagen u otro archivo binario, utilice `MEDIA:<ruta>` (ver más abajo). |
| `-s`, `--subject <LÍNEA>` | Anteponga una línea de asunto/encabezado antes del cuerpo del mensaje. |
| `-l`, `--list [plataforma]` | Enumere los objetivos configurados en todas las plataformas (o solo en la plataforma determinada). |
| `-q`, `--quiet` | Suprimir la salida estándar en caso de éxito: útil en scripts (confíe únicamente en el código de salida). |
| `--json` | Emita un resultado JSON sin formato en lugar de un resultado legible por humanos. |Si no se proporciona un argumento posicional de "mensaje" ni "--file", "hermes send" lee desde la entrada estándar cuando no es un TTY. Códigos de salida: `0` en caso de éxito, `1` en caso de error de entrega/backend, `2` en caso de errores de uso.### Envío de imágenes y otros medios`--file` es solo para cuerpos de *texto*. Para entregar un archivo de imagen, documento, video o audio como un archivo adjunto de plataforma nativa, haga referencia a él dentro del texto del mensaje con la directiva `MEDIA:<local_path>`:```golpecito
hermes envía --to telegrama "MEDIA:/tmp/screenshot.png"
hermes envía --to telegram "Crear gráfico para hoy MEDIA:/tmp/chart.png" # con título
hermes envía --to discord:#ops "MEDIA:/tmp/report.pdf"
```De forma predeterminada, los archivos de imagen se envían como fotografías (las plataformas como Telegram las recomprimen). Agregue `[[as_document]]` al mensaje para entregarlos como archivos adjuntos sin comprimir:```golpecito
hermes envía --to telegram "[[as_document]] MEDIA:/tmp/screenshot.png"
```Ejemplos:```golpecito
hermes envía --to telegram "implementación finalizada"
eco "RAM 92%" | hermes envía --a telegrama:-1001234567890
hermes envía --a discordia:#ops --file /tmp/report.md
hermes envía --to slack:#eng --subject "[CI]" --file build.log
hermes envía --lista # todas las plataformas
hermes enviar --list telegram # filtrar por plataforma
```
## `secretos de hermes````golpecito
secretos de hermes bitwarden <subcomando>
secretos de hermes bw <subcomando> # alias corto
```Extraiga las claves API de un administrador secreto externo al inicio del proceso en lugar de almacenarlas en `~/.hermes/.env`. Actualmente es compatible con **Bitwarden Secrets Manager**. Consulte la guía completa: [Integración de Bitwarden](../user-guide/secrets/bitwarden.md).Subcomandos `bitwarden` (alias `bw`):| Subcomando | Descripción |
|------------|-------------|
| `configuración` | Asistente interactivo: instale el binario `bws` anclado, almacene un token de acceso y elija un proyecto. Acepta `--project-id`, `--access-token` y `--server-url` para uso no interactivo. |
| `estado` | Muestra la configuración actual, la ruta/versión binaria y la última información obtenida. |
| `sincronizar` | Obtenga secretos ahora e informe qué cambió. Agregue `--apply` para exportar los secretos al entorno del shell actual (el valor predeterminado es la ejecución en seco). |
| `instalar` | Descargue y verifique el binario `bws` anclado. `--force` se vuelve a descargar incluso si ya existe una copia administrada. |
| `deshabilitar` | Desactive la integración de Bitwarden. |
## `hermes migra````golpecito
Hermes migra <tipo>
```Diagnosticar y (opcionalmente) reescribir el `config.yaml` activo para reemplazar referencias a modelos retirados o configuraciones obsoletas. Se realiza una copia de seguridad con marca de tiempo del `config.yaml` original antes de cualquier reescritura (omita con `--no-backup`).| Subcomando | Descripción |
|------------|-------------|
| `xaï` | Escanee `config.yaml` en busca de referencias a modelos xAI programados para retirarse el 15 de mayo de 2026 y (con `--apply`) reescríbalos in situ para los reemplazos oficiales según la guía de migración de xAI. El valor predeterminado es el funcionamiento en seco. |Banderas comunes para subcomandos de migración:| Bandera | Descripción |
|------|-------------|
| `--aplicar` | Vuelva a escribir `config.yaml` in situ (predeterminado: ejecución en seco, sin escrituras). |
| `--sin-copia de seguridad` | Omita la copia de seguridad con marca de tiempo de `config.yaml` al realizar la solicitud. |> No debe confundirse con `hermes claw migrar` (importación única de la configuración de OpenClaw a Hermes): `hermes migrar` es el comando de reescritura de configuración de nivel superior.
## `proxy de hermes````golpecito
proxy de hermes <subcomando>
```Ejecute un servidor HTTP local compatible con OpenAI que reenvíe solicitudes a un proveedor ascendente autenticado por OAuth (por ejemplo, Nous Portal, xAI). Las aplicaciones externas pueden apuntar al proxy con cualquier token de portador; el proxy adjunta sus credenciales reales de OAuth al salir. Consulte [Proxy de suscripción](../user-guide/features/subscription-proxy.md) para obtener la guía completa.| Subcomando | Descripción |
|------------|-------------|
| `inicio` | Ejecute el proxy en primer plano. Banderas: `--provider <nous\|xai>` (predeterminado `nous`), `--host <addr>` (predeterminado `127.0.0.1`; use `0.0.0.0` para exponer en LAN), `--port <int>` (predeterminado `8645`). |
| `estado` | Muestre qué flujos ascendentes de proxy están listos (credenciales presentes, OAuth válido). |
| `proveedores` | Enumere los proveedores de proxy ascendentes disponibles. |
## `seguridad hermes````golpecito
seguridad de hermes <subcomando>
```Análisis de vulnerabilidades bajo demanda contra [OSV.dev](https://osv.dev). Cubre Hermes venv (distribuciones PyPI instaladas), dependencias de Python declaradas por complementos en `~/.hermes/plugins/` y servidores MCP `npx`/`uvx` anclados en `config.yaml`. NO analiza paquetes instalados globalmente ni extensiones de editor/navegador.| Subcomando | Descripción |
|------------|-------------|
| `auditoría` | Ejecute una auditoría única de la cadena de suministro. |Banderas de "auditoría":| Bandera | Predeterminado | Descripción |
|------|---------|-------------|
| `--json` | apagado | Emita JSON legible por máquina en lugar de texto legible por humanos. |
| `--fallo <nivel>` | `crítico` | Salga de un valor distinto de cero cuando algún hallazgo cumpla con esta gravedad ("baja", "moderada", "alta", "crítica"). |
| `--skip-venv` | apagado | Omita escanear el venv de Hermes Python. |
| `--skip-plugins` | apagado | Omita los archivos de requisitos del complemento de escaneo. |
| `--skip-mcp` | apagado | Omita el escaneo de servidores MCP anclados en `config.yaml`. |
## `iniciar sesión en hermes` / `cerrar sesión en hermes` *(obsoleto)*:::precaución
Se ha eliminado el `hermes login`. Utilice `hermes auth` para administrar las credenciales de OAuth, `hermes model` para seleccionar un proveedor o `hermes setup` para una configuración interactiva completa.
:::## `autenticación de hermes`Administre grupos de credenciales para la rotación de claves del mismo proveedor. Consulte [Grupos de credenciales](/user-guide/features/credential-pools) para obtener la documentación completa.```golpecito
autenticación hermes # asistente interactivo
lista de autenticación de hermes # Mostrar todos los grupos
lista de autenticación de hermes openrouter # Mostrar proveedor específico
hermes auth agregar openrouter --api-key sk-or-v1-xxx # Agregar clave API
hermes auth agregar anthropic --type oauth # Agregar credencial OAuth
hermes auth eliminar openrouter 2 # Eliminar por índice
hermes auth reset openrouter # Borrar tiempos de reutilización
estado de autenticación de hermes antrópico # Mostrar estado de autenticación para un proveedor
hermes auth logout anthropic # Cerrar sesión y borrar el estado de autenticación almacenado
hermes auth spotify # Autenticar Hermes con Spotify a través de PKCE
```Subcomandos: `agregar`, `lista`, `eliminar`, `reiniciar`, `estado`, `cerrar sesión`, `spotify`. Cuando se llama sin subcomando, inicia el asistente de administración interactivo.## `estado de hermes````golpecito
estado de hermes [--todos] [--deep]
```| Opción | Descripción |
|--------|-------------|
| `--todos` | Muestre todos los detalles en un formato redactado que se pueda compartir. |
| `--profundo` | Realice comprobaciones más profundas que pueden llevar más tiempo. |## `hermes cron````golpecito
hermes cron <lista|crear|editar|pausa|reanudar|ejecutar|eliminar|estado|tick>
```| Subcomando | Descripción |
|------------|-------------|
| `lista` | Mostrar trabajos programados. |
| `crear` / `agregar` | Cree un trabajo programado a partir de un mensaje, adjuntando opcionalmente una o más habilidades mediante `--skill` repetido. |
| `editar` | Actualice el cronograma, el mensaje, el nombre, la entrega, el recuento de repeticiones o las habilidades adjuntas de un trabajo. Admite `--clear-skills`, `--add-skill` y `--remove-skill`. |
| `pausa` | Pausar un trabajo sin eliminarlo. |
| `reanudar` | Reanudar un trabajo en pausa y calcular su próxima ejecución futura. |
| `ejecutar` | Activa un trabajo en el siguiente tic del programador. |
| `eliminar` | Eliminar un trabajo programado. |
| `estado` | Compruebe si el programador cron se está ejecutando. |
| `tick` | Ejecute los trabajos pendientes una vez y salga. |El cron **trigger** se puede conectar mediante la clave de configuración `cron.provider`. vacio
(el valor predeterminado) utiliza el ticker de proceso incorporado. Configúrelo en `chronos` (el
Proveedor administrado por NAS para puertas de enlace alojadas de escala a cero): configurado a través de
Claves `cron.chronos.*` (`portal_url`, `callback_url`, `expected_audience`,
`nas_jwks_url`) — o nombre un proveedor personalizado en `plugins/cron/<nombre>/` o
`$HERMES_HOME/plugins/<nombre>/`. Un proveedor desconocido o no disponible recurre a
el incorporado, por lo que cron nunca se queda sin un disparador. Ver el
[cron internals](../developer-guide/cron-internals.md#gateway-integration) doc.## `hermes kanban````golpecito
hermes kanban [--board <slug>] <acción> [opciones]
```Tablero de colaboración multiperfil y multiproyecto. Cada instalación puede albergar muchos tableros (uno por proyecto, repositorio o dominio); cada placa es una cola independiente con su propia base de datos SQLite y alcance del despachador. Las nuevas instalaciones comienzan con una placa llamada `default`, cuya base de datos es `~/.hermes/kanban.db` para compatibilidad con versiones anteriores; tableros adicionales se encuentran en `~/.hermes/kanban/boards/<slug>/kanban.db`. El despachador integrado en la puerta de enlace barre cada tablero por tic.**Indicadores globales (se aplican a todas las acciones siguientes):**| Bandera | Propósito |
|------|---------|
| `--tablero <slug>` | Operar en una placa específica. El valor predeterminado es el tablero actual (configurado mediante `hermes kanban boards switch`, la var env `HERMES_KANBAN_BOARD` o `default`). |**Esta es la superficie humana/de secuencias de comandos.** Los trabajadores agentes generados por el despachador conducen el tablero a través de un `kanban_*` [conjunto de herramientas](/user-guide/features/kanban#how-workers-interact-with-the-board) (`kanban_show`, `kanban_complete`, `kanban_block`, `kanban_create`, `kanban_link`, `kanban_comment`, `kanban_heartbeat`; Los perfiles de orquestador también obtienen `kanban_list` y `kanban_unblock`) en lugar de bombardear a `hermes kanban`. Los trabajadores tienen `HERMES_KANBAN_BOARD` fijado en su entorno para que físicamente no puedan ver otros tableros.| Acción | Propósito |
|--------|---------|
| `inicio` | Cree `kanban.db` si falta. Idempotente. |
| `lista de tableros` / `tableros ls` | Enumere todos los tableros con recuentos de tareas. `--json`, `--all` (incluir archivado). |
| `tableros crean <slug>` | Crea un nuevo tablero. Banderas: `--name`, `--description`, `--icon`, `--color`, `--switch` (activar). Slug es un estuche de kebab, con reducción automática. |
| `interruptor de placas <slug>` / `uso de placas` | Conserve `<slug>` como tablero activo (escribe `~/.hermes/kanban/current`). |
| `tableros muestran` / `tableros actuales` | Imprima el nombre de la placa actualmente activa, la ruta de la base de datos y el recuento de tareas. |
| `los tableros cambian el nombre de <slug> "<nombre>"` | Cambiar el nombre para mostrar de un tablero. Slug es inmutable. |
| `tableros rm <slug>` | Archivar (predeterminado) o eliminar definitivamente un tablero. `--delete` omite el paso de archivar. Los tableros archivados se mueven a `boards/_archived/<slug>-<ts>/`. Rechazado por "incumplimiento". |
| `crear "<título>"` | Crea una nueva tarea en el tablero activo. Banderas: `--body`, `--assignee`, `--parent` (repetible), `--workspace scratch\|worktree\|dir:<path>`, `--tenant`, `--priority`, `--triage`, `--idempotency-key`, `--max-runtime`, `--max-retries`, `--skill` (repetible). |
| `lista` / `ls` | Enumere las tareas en el tablero activo. Filtre con `--mine`, `--assignee`, `--status`, `--tenant`, `--archived`, `--json`. |
| `mostrar <id>` | Mostrar una tarea con comentarios y eventos. `--json` para salida de la máquina. |
| `asignar <id> <perfil>` | Asignar o reasignar. Utilice "ninguno" para desasignar. Se rechazó mientras la tarea se estaba ejecutando. |
| `enlace <padre> <hijo>` | Añade una dependencia. Ciclo detectado. Ambas tareas deben estar en el mismo tablero. |
| `desvincular <padre> <hijo>` | Eliminar una dependencia. |
| `reclamar <id>` | Reclama atómicamente una tarea lista. Imprime la ruta del espacio de trabajo resuelta. |
| `comentario <id> "<texto>"` | Añade un comentario. El siguiente trabajador que reclama la tarea la lee como parte de su respuesta `kanban_show()`. |
| `completo <id>` | Marcar tarea realizada. Banderas: `--resultado`, `--summary`, `--metadata`. |
| `bloque <id> "<razón>"` | Marcar tarea bloqueada para entrada humana. También se añade el motivo como comentario. |
| `programar <id> "<razón>"` | Estacione el trabajo de seguimiento/retraso de tiempo en "programado" para que no se muestre como un bloqueador humano. |
| `desbloquear <id>` | Devuelve una tarea bloqueada o programada a lista (o "todo" si las dependencias aún están abiertas). |
| `archivo <id>` | Ocultar de la lista predeterminada. `gc` eliminará los espacios de trabajo borrador. |
| `cola <id>` | Siga el flujo de eventos de una tarea. |
| `despacho` | Un pase de despachador en el tablero activo. Banderas: `--dry-run`, `--max N`, `--failure-limit N`, `--json`. |
| `contexto <id>` | Imprima el contexto completo que vería un trabajador (título + cuerpo + resultados principales + comentarios). |
| `especificar <id>` / `especificar --todos` | Desarrolle una tarea de columna de clasificación en una especificación concreta (título + cuerpo con objetivo, enfoque, criterios de aceptación) a través del LLM auxiliar y luego promuévala a "todo". Banderas: `--tenant` (alcance `--all` para un inquilino), `--author`, `--json`. Configure el modelo en `auxiliary.triage_specifier` en `config.yaml`. |
| `descomponer <id>` / `descomponer --todos` | Distribuya una tarea de columna de clasificación en un gráfico de tareas secundarias dirigidas a perfiles de especialistas por descripción. Se recurre a la promoción de tarea única de estilo específico cuando el LLM decide que la tarea no se beneficia de la distribución. Mismas banderas que "especificar". Configure el modelo de descomposición en `auxiliary.kanban_decomposer` en `config.yaml`; `kanban.orchestrator_profile` solo controla quién es el propietario de la tarea raíz/orquestación después de la distribución. También ejecuta automáticamente cada tick del despachador cuando `kanban.auto_decompose: true` (el valor predeterminado). Consulte [Orquestación automática frente a manual](/user-guide/features/kanban#auto-vs-manual-orchestration). |
| `gc` | Elimine espacios de trabajo temporales para tareas archivadas. |Ejemplos:```golpecito
# Crea un segundo tablero y ponle una tarea sin cambiar.
Los tableros kanban de Hermes crean atm10-server --name "ATM10 Server" --icon 🎮
hermes kanban --board atm10-server crear "Reiniciar servidor" --operaciones asignadas# Cambie el tablero activo para llamadas posteriores.
Hermes kanban tableros cambiar atm10-server
La lista kanban de Hermes # muestra las tareas del servidor atm10# Archivar un tablero (recuperable) o eliminarlo por completo.
tableros kanban de hermes rm atm10-server
tableros kanban de hermes rm atm10-server --eliminar
```Orden de resolución del tablero (la prioridad más alta primero): `--board <slug>` flag → `HERMES_KANBAN_BOARD` env var → `~/.hermes/kanban/current` file → `default`.Todas las acciones también están disponibles como un comando de barra diagonal en la puerta de enlace (`/kanban…`), con la misma superficie de argumento, incluidos los subcomandos `boards` y el indicador `--board`.Para ver el diseño completo (comparación con Cline Kanban / Paperclip / NanoClaw / Gemini Enterprise, ocho patrones de colaboración, cuatro historias de usuarios, prueba de corrección de la concurrencia), consulte `docs/hermes-kanban-v1-spec.pdf` en el repositorio o en la [guía del usuario de Kanban](/user-guide/features/kanban).## `hermes webhook````golpecito
webhook de hermes <suscribir|lista|eliminar|prueba>
```Administre suscripciones dinámicas a webhooks para la activación de agentes basada en eventos. Requiere que la plataforma webhook esté habilitada en la configuración; si no está configurada, imprime las instrucciones de configuración.| Subcomando | Descripción |
|------------|-------------|
| `suscribir` / `agregar` | Cree una ruta de webhook. Devuelve la URL y el secreto HMAC para configurar en su servicio. |
| `lista` / `ls` | Mostrar todas las suscripciones creadas por agentes. |
| `eliminar`/`rm` | Eliminar una suscripción dinámica. Las rutas estáticas de config.yaml no se ven afectadas. |
| `prueba` | Envíe una POST de prueba para verificar que una suscripción esté funcionando. |### `hermes webhook suscribirse````golpecito
hermes webhook suscribirse <nombre> [opciones]
```| Opción | Descripción |
|--------|-------------|
| `--prompt` | Plantilla de aviso con referencias de carga útil `{dot.notation}`. |
| `--eventos` | Tipos de eventos separados por comas para aceptar (por ejemplo, `problemas, pull_request`). Vacío = todo. |
| `--descripción` | Descripción legible por humanos. |
| `--habilidades` | Nombres de habilidades separados por comas que se cargarán para la ejecución del agente. |
| `--entregar` | Destino de entrega: `log` (predeterminado), `telegram`, `discord`, `slack`, `github_comment`. |
| `--deliver-chat-id` | ID de canal/chat de destino para entrega multiplataforma. |
| `--secreto` | Secreto HMAC personalizado. Generado automáticamente si se omite. |
| `--solo entrega` | Omita el agente: entregue el "--prompt" representado como mensaje literal. Costo cero de LLM, entrega en menos de un segundo. Requiere que `--deliver` sea un objetivo real (no `log`). |Las suscripciones persisten en `~/.hermes/webhook_subscriptions.json` y el adaptador de webhook las recarga en caliente sin reiniciar la puerta de enlace.## `doctor hermes````golpecito
doctor hermes [--fix]
```| Opción | Descripción |
|--------|-------------|
| `--arreglar` | Intente realizar reparaciones automáticas siempre que sea posible. |## `vertedero de Hermes````golpecito
volcado de hermes [--show-keys]
```Genera un resumen compacto y en texto plano de toda su configuración de Hermes. Diseñado para copiar y pegar en Discord, problemas de GitHub o Telegram cuando se solicita soporte: sin colores ANSI, sin formato especial, solo datos.| Opción | Descripción |
|--------|-------------|
| `--mostrar-claves` | Muestre los prefijos de clave API redactados (primeros y últimos 4 caracteres) en lugar de simplemente "establecer"/"no establecer". |### Qué incluye| Sección | Detalles |
|---------|---------|
| **Encabezado** | Versión de Hermes, fecha de lanzamiento, hash de confirmación de git |
| **Medio ambiente** | SO, versión Python, versión OpenAI SDK |
| **Identidad** | Nombre del perfil activo, ruta HERMES_HOME |
| **Modelo** | Modelo y proveedor predeterminados configurados |
| **Terminal** | Tipo de backend (local, acoplable, ssh, etc.) |
| **Claves API** | Comprobación de presencia de las 22 claves API de proveedor/herramienta |
| **Características** | Conjuntos de herramientas habilitados, recuento de servidores MCP, proveedor de memoria |
| **Servicios** | Estado de la puerta de enlace, plataformas de mensajería configuradas |
| **Carga de trabajo** | Recuento de trabajos cron, recuento de habilidades instaladas |
| **Anulaciones de configuración** | Cualquier valor de configuración que difiera de los valores predeterminados |### Salida de ejemplo```
--- volcado de hermes ---
versión: 0.8.0 (2026.4.8) [af4abd2f]
Sistema operativo: Linux 6.14.0-37-generic x86_64
pitón: 3.11.14
openai_sdk: 2.24.0
perfil: predeterminado
hermes_home: ~/.hermes
modelo: antrópico/claude-opus-4.6
proveedor: enrutador abierto
terminales: localesclaves_api:
  conjunto de enrutador abierto
  openai no configurado
  conjunto antrópico
  nosotros no establecido
  conjunto de fuegos artificiales
  ...características:
  conjuntos de herramientas: todos
  servidores_mcp: 0
  proveedor_de_memoria: incorporado
  puerta de enlace: en ejecución (systemd)
  plataformas: telegrama, discordia
  cron_jobs: 3 activos / 5 en total
  habilidades: 42config_overrides:
  agente.max_turns: 250
  Umbral.de.compresión: 0,85
  display.streaming: Verdadero
--- finalizar el volcado ---
```### Cuándo usar- Informar un error en GitHub: pegue el volcado en su problema
- Solicitar ayuda en Discord: compártela en un bloque de código
- Comparar su configuración con la de otra persona
- Comprobación rápida de cordura cuando algo no funciona:::consejo
`hermes dump` está diseñado específicamente para compartir. Para diagnósticos interactivos, utilice "hermes doctor". Para obtener una descripción general visual, utilice "estado de Hermes".
:::## `depuración de Hermes````golpecito
compartir depuración de hermes [opciones]
```Cargue un informe de depuración (información del sistema + registros recientes) en un servicio de pegado y obtenga una URL que se pueda compartir. Útil para solicitudes de soporte rápido: incluye todo lo que un ayudante necesita para diagnosticar su problema.| Opción | Descripción |
|--------|-------------|
| `--líneas <N>` | Número de líneas de registro que se incluirán por archivo de registro (predeterminado: 200). |
| `--expire <días>` | Caducidad de pegado en días (predeterminado: 7). |
| `--local` | Imprima el informe localmente en lugar de cargarlo. |El informe incluye información del sistema (sistema operativo, versión de Python, versión de Hermes), registros recientes del agente, puerta de enlace, GUI/panel y escritorio (límite de 512 KB por archivo) y estado de la clave API redactada. Las claves siempre están redactadas: no se cargan secretos.Los servicios de pegado se probaron en orden: paste.rs, dpaste.com.### Ejemplos```golpecito
hermes debug share # Cargar informe de depuración, imprimir URL
hermes debug share --lines 500 # Incluir más líneas de registro
hermes debug share --expire 30 # Mantener pegado durante 30 días
compartir depuración de hermes --local # Imprimir informe en la terminal (sin cargar)
```## `copia de seguridad de Hermes````golpecito
copia de seguridad de hermes [opciones]
```Cree un archivo zip de su configuración, habilidades, sesiones y datos de Hermes. La copia de seguridad excluye el código base del agente hermes.| Opción | Descripción |
|--------|-------------|
| `-o`, `--salida <ruta>` | Ruta de salida para el archivo zip (predeterminado: `~/hermes-backup-<marca de tiempo>.zip`). |
| `-q`, `--quick` | Instantánea rápida: solo archivos de estado críticos (config.yaml, state.db, .env, auth, cron jobs). Mucho más rápido que una copia de seguridad completa. |
| `-l`, `--label <nombre>` | Etiqueta para la instantánea (solo se usa con `--quick`). |La copia de seguridad utiliza la API `backup()` de SQLite para una copia segura, por lo que funciona correctamente incluso cuando Hermes se está ejecutando (modo WAL seguro).**Qué está excluido del zip:**- `*.db-wal`, `*.db-shm`, `*.db-journal` — WAL/memoria compartida/sidecars de diario de SQLite. El archivo `*.db` ya obtuvo una instantánea consistente a través de `sqlite3.backup()`; enviar los sidecars en vivo junto a él permitiría que una restauración vea un estado medio comprometido.
- `checkpoints/` — cachés de trayectoria por sesión. Con clave hash y regenerada por sesión; De todos modos, no se trasladaría limpiamente a otra instalación.
- El código `hermes-agent` en sí (esta es una copia de seguridad de los datos del usuario, no una instantánea del repositorio).### Ejemplos```golpecito
copia de seguridad de hermes # Copia de seguridad completa en ~/hermes-backup-*.zip
hermes backup -o /tmp/hermes.zip # Copia de seguridad completa en una ruta específica
copia de seguridad de hermes --quick # Instantánea rápida de solo estado
copia de seguridad de hermes --quick --label "pre-upgrade" # Instantánea rápida con etiqueta
```## `puntos de control de Hermes````golpecito
puntos de control de hermes [COMANDO]
```Inspeccione y administre la tienda Shadow Git en `~/.hermes/checkpoints/`: la capa de almacenamiento detrás del comando `/rollback` durante la sesión. Seguro para ejecutar en cualquier momento; no requiere que el agente esté ejecutándose.| Subcomando | Descripción |
|------------|-------------|
| `estado` (predeterminado) | Muestre el tamaño total, el recuento de proyectos y el desglose por proyecto. Los "puntos de control de Hermes" desnudos son equivalentes. |
| `lista` | Alias ​​de "estado". |
| `poda` | Fuerce un barrido de limpieza: elimine proyectos huérfanos y obsoletos, GC en la tienda, aplique el límite de tamaño. Ignora el marcador de idempotencia de 24 horas. |
| `claro` | Elimina toda la base del punto de control. Irreversible; pide confirmación a menos que sea `-f`. |
| `claro-legado` | Elimine solo los archivos `legacy-<timestamp>/` producidos por la migración v1→v2. |### Opciones| Opción | Subcomando | Descripción |
|--------|------------|-------------|
| `--límite N` | `estado`, `lista` | Máximo de proyectos para enumerar (predeterminado 20). |
| `--días-de-retención N` | `poda` | Elimine proyectos cuyo `last_touch` tenga más de N días (predeterminado 7). |
| `--max-size-mb N` | `poda` | Después del pase huérfano/obsoleto, elimine la confirmación más antigua por proyecto hasta que el tamaño total del almacén sea ≤ N MB (predeterminado 500). |
| `--mantener-huérfanos` | `poda` | Omita la eliminación de proyectos cuyo directorio de trabajo ya no exista. |
| `-f`, `--force` | `claro`, `claro-legado` | Omita el mensaje de confirmación. |### Ejemplos```golpecito
puntos de control de hermes # descripción general del estado
Hermes puntos de control podar --retención-días 3 # limpieza agresiva
Hermes puntos de control podar --max-size-mb 200 # apretar la tapa de tamaño una vez
puntos de control de hermes clear-legacy -f # soltar directorios de archivo v1
puntos de control de hermes borrar -f # borrar todo
```Consulte [Puntos de control y `/rollback`](../user-guide/checkpoints-and-rollback.md) para conocer la arquitectura completa y los comandos de la sesión.## `importación de hermes````golpecito
importar hermes <archivo zip> [opciones]
```Restaure una copia de seguridad de Hermes creada previamente en su directorio de inicio de Hermes. Todos los archivos del archivo sobrescriben los archivos existentes en su hogar Hermes; `--force` solo omite el mensaje de confirmación que se activa cuando el objetivo ya tiene una instalación de Hermes.| Opción | Descripción |
|--------|-------------|
| `-f`, `--force` | Omita el mensaje de confirmación de la instalación existente. |:::advertencia
Detenga la puerta de enlace antes de importar para evitar conflictos con los procesos en ejecución.
:::### Ejemplos
```golpecito
hermes import ~/hermes-backup-20260423.zip # Indicaciones antes de sobrescribir la configuración existente
importación de hermes ~/hermes-backup-20260423.zip --force # Sobrescribir sin preguntar
```## `registros de hermes````golpecito
registros de hermes [nombre_registro] [opciones]
```Vea, siga y filtre archivos de registro de Hermes. Todos los registros se almacenan en `~/.hermes/logs/` (o `<profile>/logs/` para perfiles no predeterminados).### Archivos de registro| Nombre | Archivo | Lo que captura |
|------|------|-----------------|
| `agente` (predeterminado) | `agente.log` | Toda la actividad de los agentes: llamadas API, envío de herramientas, ciclo de vida de la sesión (INFO y superior) |
| `errores` | `errores.log` | Sólo advertencias y errores: un subconjunto filtrado de agent.log |
| `puerta de enlace` | `puerta de enlace.log` | Actividad de la puerta de enlace de mensajería: conexiones de plataforma, envío de mensajes, eventos de webhook |
| `gui` | `gui.log` | Panel de control / TUI-gateway / PTY-bridge / eventos websocket |
| `escritorio` | `desktop.log` | Aplicación de escritorio Electron: arranque, salida de generación de backend y rastreos recientes de Python |### Opciones| Opción | Descripción |
|--------|-------------|
| `nombre_registro` | Qué registro ver: `agente` (predeterminado), `errores`, `puerta de enlace` o `lista` para mostrar los archivos disponibles con tamaños. |
| `-n`, `--lines <N>` | Número de líneas a mostrar (predeterminado: 50). |
| `-f`, `--follow` | Siga el registro en tiempo real, como `tail -f`. Presione Ctrl+C para detener. |
| `--nivel <NIVEL>` | Nivel de registro mínimo para mostrar: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`. |
| `--sesión <ID>` | Filtre líneas que contengan una subcadena de ID de sesión. |
| `--desde <TIEMPO>` | Muestra líneas de hace un tiempo relativo: `30m`, `1h`, `2d`, etc. Admite `s` (segundos), `m` (minutos), `h` (horas), `d` (días). |
| `--componente <NOMBRE>` | Filtrar por componente: `gateway`, `agent`, `tools`, `cli`, `cron`. |### Ejemplos```golpecito
# Ver las últimas 50 líneas de agent.log (predeterminado)
registros de hermes# Sigue a agent.log en tiempo real
registros de hermes -f# Ver las últimas 100 líneas de gateway.log
puerta de enlace de registros de hermes -n 100# Mostrar solo advertencias y errores de la última hora
registros de hermes --nivel ADVERTENCIA --desde 1h# Filtrar por una sesión específica
registros de hermes --sesión abc123# Seguir errores.log, desde hace 30 minutos
Hermes registra errores --desde 30m -f# Listar todos los archivos de registro con sus tamaños
lista de registros de hermes
```### FiltradoLos filtros se pueden combinar. Cuando hay varios filtros activos, una línea de registro debe pasar **todos** para que se muestre:```golpecito
# WARNING+ líneas de las últimas 2 horas que contienen la sesión "tg-12345"
registros de hermes --nivel ADVERTENCIA --desde 2h --sesión tg-12345
```Las líneas sin una marca de tiempo analizable se incluyen cuando `--since` está activo (pueden ser líneas de continuación de una entrada de registro de varias líneas). Las líneas sin un nivel detectable se incluyen cuando `--level` está activo.### Rotación de registrosHermes usa `RotatingFileHandler` de Python. Los registros antiguos se rotan automáticamente; busque `agent.log.1`, `agent.log.2`, etc. El subcomando `hermes logs list` muestra todos los archivos de registro, incluidos los rotados.
## `tamaño del mensaje de Hermes````golpecito
tamaño del mensaje de hermes [--plataforma <nombre>] [--json]
```Informa el presupuesto fijo fijo para una nueva sesión: lo que se envía en cada
Llamada API *antes* de cualquier contenido de conversación. Útil cuando un adaptador descendente o
proxy tiene un presupuesto rápido más ajustado que la ventana de contexto del modelo, o cuando
Quiero ver qué bloque (índice de habilidades, memoria, perfil) domina.Crea el mismo mensaje del sistema que haría el agente y luego lo desglosa:- **Total de mensajes del sistema**: mensaje completo (identidad, orientación, habilidades
  índice, archivos de contexto, memoria, perfil, marca de tiempo).
- **Índice de habilidades**: el bloque `<available_skills>`. Este suele ser el más grande
  bloque único cuando se instalan muchas habilidades.
- **Memoria** y **perfil de usuario**: tus instantáneas `MEMORY.md`/`USER.md`.
- **Niveles rápidos**: estable/contexto/volátil, que coincide con las capas de Hermes
  el mensaje de compatibilidad con caché.
- **Esquemas de herramientas**: el JSON para todas las herramientas habilitadas (la otra mitad del
  carga útil fija por llamada).Se ejecuta completamente sin conexión: sin llamadas API, funciona sin credenciales configuradas.```golpecito
# Desglose legible por humanos para la plataforma CLI (predeterminado)
Hermes tamaño rápido# Simular el mensaje de una plataforma de mensajería (pista de plataforma diferente)
hermes tamaño rápido --plataforma telegrama# Salida legible por máquina para scripts
tamaño del mensaje de hermes --json
```:::consejo
El índice de habilidades y los esquemas de herramientas varían según la cantidad de habilidades y herramientas que tenga.
habilitado. Para reducir el mensaje, deshabilite los conjuntos de herramientas no utilizados ("hermes tools") o
desinstale habilidades que no necesita (`hermes skills`). Archivos de contexto (AGENTS.md,
.cursorrules) en su directorio actual también cuentan para el total.
:::## `configuración de hermes````golpecito
configuración de hermes <subcomando>
```Subcomandos:| Subcomando | Descripción |
|------------|-------------|
| `mostrar` | Mostrar los valores de configuración actuales. |
| `editar` | Abra `config.yaml` en su editor. |
| `establecer <clave> <valor>` | Establezca un valor de configuración. |
| `ruta` | Imprima la ruta del archivo de configuración. |
| `ruta-env` | Imprima la ruta del archivo `.env`. |
| `verificar` | Compruebe si hay configuraciones faltantes o obsoletas. |
| `migrar` | Agregue opciones recién introducidas de forma interactiva. |## `emparejamiento de hermes````golpecito
emparejamiento de hermes <lista|aprobar|revocar|claro-pendiente>
```| Subcomando | Descripción |
|------------|-------------|
| `lista` | Mostrar usuarios pendientes y aprobados. |
| `aprobar <plataforma> <código>` | Aprobar un código de emparejamiento. |
| `revocar <plataforma> <id-usuario>` | Revocar el acceso de un usuario. |
| `claro-pendiente` | Borrar códigos de emparejamiento pendientes. |## `habilidades de hermes````golpecito
habilidades de hermes <subcomando>
```Subcomandos:| Subcomando | Descripción |
|------------|-------------|
| `navegar` | Navegador paginado para registros de habilidades. |
| `buscar` | Buscar registros de habilidades. |
| `instalar` | Instalar una habilidad. |
| `inspeccionar` | Obtenga una vista previa de una habilidad sin instalarla. |
| `lista` | Listar las habilidades instaladas. |
| `verificar` | Verifique las habilidades del concentrador instalado para obtener actualizaciones ascendentes. |
| `actualizar` | Reinstale las habilidades del concentrador con los cambios ascendentes cuando estén disponibles. |
| `auditoría` | Vuelva a escanear las habilidades del concentrador instalado. |
| `desinstalar` | Eliminar una habilidad instalada en el concentrador. |
| `restablecer` | Desbloquee una habilidad incluida marcada como "usuario_modificado" borrando su entrada de manifiesto. Con `--restore`, también reemplaza la copia del usuario con la versión incluida. |
| `optar por no participar` | Evite que las habilidades agrupadas se incluyan en el perfil activo. Escribe un marcador `.no-bundled-skills` para que el instalador, `hermes update` y cualquier sincronización omitan la inicialización de habilidades incluidas. Seguro por defecto: no se toca nada del disco. Con `--remove`, también elimina las habilidades empaquetadas ya presentes que están **sin modificar** (las habilidades editadas por el usuario, instaladas en el concentrador y escritas a mano nunca se eliminan; obtiene una vista previa y confirma primero, `--yes` para omitir). |
| `optar por participar` | Deshaga la "exclusión voluntaria" eliminando el marcador ".no-bundled-skills" para que las habilidades incluidas se vuelvan a sembrar en la próxima "actualización de Hermes". Con `--sync`, vuelva a sembrar inmediatamente. |
| `publicar` | Publicar una habilidad en un registro. |
| `instantánea` | Exportar/importar configuraciones de habilidades. |
| `toque` | Administre fuentes de habilidades personalizadas. |
| `config` | Configuración interactiva de activación/desactivación de habilidades por plataforma. |Ejemplos comunes:```golpecito
habilidades de hermes navegar
Exploración de habilidades de Hermes --fuente oficial
búsqueda de habilidades de hermes reaccionar --fuente habilidades-sh
Búsqueda de habilidades de Hermes https://mintlify.com/docs --fuente conocida
Habilidades de Hermes inspeccionar oficial/seguridad/1contraseña
habilidades de hermes inspeccionar habilidades-sh/vercel-labs/json-render/json-render-react
Habilidades de Hermes instalar oficial/migración/openclaw-migration
habilidades de hermes instalar habilidades-sh/anthropics/skills/pdf --force
Instalación de habilidades de Hermes https://sharethis.chat/SKILL.md # URL directa (SKILL.md de un solo archivo)
instalación de habilidades de hermes https://example.com/SKILL.md --name my-skill # Anular el nombre cuando frontmatter no tiene ninguno
verificación de habilidades de hermes
actualización de habilidades de hermes
configuración de habilidades de hermes
Habilidades de Hermes restablecer espacio de trabajo de Google
Habilidades de Hermes restablecer google-workspace --restore --yes
Optar por no participar en las habilidades de Hermes # detener la futura siembra de habilidades incluidas (no se elimina nada)
Optar por no participar en las habilidades de Hermes --remove --yes # también eliminar las habilidades incluidas SIN MODIFICAR
Hermes Skills opt-in --sync # deshacer: eliminar el marcador y volver a sembrar ahora
```Notas:
- `--force` puede anular bloques de políticas no peligrosas para habilidades de terceros/comunidad.
- `--force` no anula un veredicto de escaneo `peligroso`.
- `--source skills-sh` busca en el directorio público `skills.sh`.
- `--source well-known` le permite señalar a Hermes un sitio que expone `/.well-known/skills/index.json`.
- `--source browser-sh` busca en el catálogo de [browse.sh](https://browse.sh) más de 200 habilidades de automatización de navegador específicas del sitio. Los identificadores se parecen a `browse-sh/airbnb.com/search-listings-ddgioa`.
- Pasar una URL `http(s)://…/*.md` instala un archivo único SKILL.md directamente. Cuando el frontmatter no tiene `nombre:` y el slug de URL no es un identificador válido, una terminal interactiva solicita un nombre; Las superficies no interactivas (`/skills install` dentro de la TUI, plataformas de puerta de enlace) requieren `--name <x>` en su lugar.## `paquetes de hermes````golpecito
paquetes de hermes <subcomando>
```Los paquetes de habilidades agrupan varias habilidades bajo un comando de barra diagonal `/<bundle-name>`. Al invocar el paquete se cargan todas las habilidades a las que se hace referencia en un único mensaje de usuario combinado. Almacenamiento: `~/.hermes/skill-bundles/<slug>.yaml`. Consulte [Paquetes de habilidades](../user-guide/features/skills.md#skill-bundles) para conocer el esquema y el comportamiento de YAML.Subcomandos:| Subcomando | Descripción |
|------------|-------------|
| `lista` | Listar paquetes instalados (predeterminado cuando no se proporciona ningún subcomando) |
| `mostrar <nombre>` | Mostrar el nombre, la descripción, las habilidades y la ruta del archivo de un paquete |
| `crear <nombre>` | Crea un nuevo paquete. Pase `--skill <id>` (repetir) u omita para la entrada interactiva. `--description`, `--instruction`, `--force` disponibles. |
| `eliminar <nombre>` | Eliminar un archivo de paquete |
| `recargar` | Vuelva a escanear `~/.hermes/skill-bundles/` e informe los paquetes agregados/eliminados |Ejemplos:```golpecito
Los paquetes de Hermes crean backend-dev \
  --skill revisión-de-código-github \
  --desarrollo basado en pruebas de habilidades \
  --skill github-pr-flujo de trabajo \
  -d "Funcionamiento de funciones de backend"lista de paquetes de hermes
Los paquetes de Hermes muestran el desarrollo backend.
paquetes de hermes eliminar backend-dev
```En una sesión de chat, `/bundles` enumera los paquetes instalados y `/<bundle-name>` carga uno.## `curador de hermes````golpecito
curador de hermes <subcomando>
```El curador es una tarea en segundo plano del modelo auxiliar que revisa periódicamente las habilidades creadas por los agentes, elimina las obsoletas, consolida las superposiciones y archiva las habilidades obsoletas. Las habilidades empaquetadas e instaladas en el hub nunca se tocan. Los archivos son recuperables; La eliminación automática nunca ocurre.| Subcomando | Descripción |
|------------|-------------|
| `estado` | Mostrar estado del curador y estadísticas de habilidades |
| `ejecutar` | Activar una revisión del curador ahora (se bloquea hasta que finaliza el pase LLM) |
| `ejecutar --fondo` | Inicie el pase de LLM en un hilo en segundo plano y regrese inmediatamente |
| `ejecutar --ejecución en seco` | Solo vista previa: genere el informe de revisión sin mutaciones |
| `copia de seguridad` | Tome una instantánea tar.gz manual de `~/.hermes/skills/` (el curador también toma instantáneas automáticamente antes de cada ejecución real) |
| `revertir` | Restaurar `~/.hermes/skills/` desde una instantánea (predeterminado a más reciente) |
| `revertir --lista` | Lista de instantáneas disponibles |
| `revertir --id <ts>` | Restaurar una instantánea específica por id |
| `revertir -y` | Saltar el mensaje de confirmación |
| `pausa` | Pausa el curador hasta que se reanude |
| `reanudar` | Reanudar un curador pausado |
| `pin <habilidad>` | Fijar una habilidad para que el curador nunca realice una transición automática en ella |
| `desanclar <habilidad>` | Desanclar una habilidad |
| `restaurar <habilidad>` | Restaurar una habilidad archivada |
| `archivar <habilidad>` | Archivar una habilidad manualmente |
| `poda` | Podar manualmente las habilidades que el curador normalmente limpiaría |
| `lista-archivada` | Lista de habilidades archivadas (recuperables mediante `restore`) |En una instalación nueva, el primer pase programado se difiere en un `interval_hours` completo (7 días de forma predeterminada); la puerta de enlace no se curará inmediatamente en el primer tic después de la `actualización de Hermes`. Utilice `hermes curador run --dry-run` para obtener una vista previa antes de que eso suceda.Consulte [Curator](../user-guide/features/curator.md) para conocer el comportamiento y la configuración.## `retroceso de Hermes````golpecito
reserva de hermes <subcomando>
```Gestionar la cadena de proveedores alternativos. Los proveedores alternativos se prueban en orden cuando el modelo principal falla con errores de límite de velocidad, sobrecarga o conexión.| Subcomando | Descripción |
|------------|-------------|
| `lista` (alias: `ls`) | Mostrar la cadena alternativa actual (predeterminada cuando no hay subcomando) |
| `agregar` | Elija un proveedor + modelo (el mismo selector que "modelo Hermes") y agréguelo a la cadena |
| `eliminar` (alias: `rm`) | Elija una entrada para eliminar de la cadena |
| `claro` | Eliminar todas las entradas alternativas |Consulte [Proveedores alternativos](../user-guide/features/fallback-providers.md).## `ganchos de hermes````golpecito
ganchos hermes <subcomando>
```Inspeccione los ganchos de script de shell declarados en `~/.hermes/config.yaml`, pruébelos con cargas útiles sintéticas y administre la lista de permisos de consentimiento de primer uso en `~/.hermes/shell-hooks-allowlist.json`.| Subcomando | Descripción |
|------------|-------------|
| `lista` (alias: `ls`) | Lista de enlaces configurados con estado de coincidencia, tiempo de espera y consentimiento |
| `prueba <evento>` | Dispara cada gancho que coincida con `<evento>` contra una carga útil sintética |
| `revocar` (alias: `eliminar`, `rm`) | Eliminar las entradas de la lista de permitidos de un comando (entra en vigor en el próximo reinicio) |
| `médico` | Verifique cada enlace configurado: bit ejecutivo, lista de permitidos, deriva de mtime, validez de JSON y sincronización de ejecución sintética |Consulte [Hooks](../user-guide/features/hooks.md) para conocer las firmas de eventos y las formas de carga útil.## `memoria de hermes````golpecito
memoria de hermes <subcomando>
```Configure y administre complementos de proveedores de memoria externa. Proveedores disponibles: honcho, openviking, mem0, retrospectiva, holográfica, retencióndb, byterover, supermemory. Sólo un proveedor externo puede estar activo a la vez. La memoria incorporada (MEMORY.md/USER.md) siempre está activa.Subcomandos:| Subcomando | Descripción |
|------------|-------------|
| `configuración` | Selección y configuración interactiva de proveedores. |
| `estado` | Mostrar la configuración actual del proveedor de memoria. |
| `apagado` | Deshabilite el proveedor externo (solo integrado). |:::info Subcomandos específicos del proveedor
Cuando un proveedor de memoria externa está activo, puede registrar su propio comando `hermes <proveedor>` de nivel superior para la gestión específica del proveedor (por ejemplo, `hermes honcho` cuando Honcho está activo). Los proveedores inactivos no exponen sus subcomandos. Ejecute `hermes --help` para ver qué está conectado actualmente.
:::## `hermes acp````golpecito
hermes acp
```Inicia Hermes como un servidor stdio ACP (Agent Client Protocol) para la integración del editor.Puntos de entrada relacionados:```golpecito
hermes-acp
python -m acp_adapter
```Instale el soporte primero:```golpecito
cd ~/.hermes/hermes-agent && uv pip install -e '.[acp]'
```Consulte [Integración del editor ACP](../user-guide/features/acp.md) y [ACP Internals](../developer-guide/acp-internals.md).## `hermes mcp````golpecito
hermes mcp <subcomando>
```Administre las configuraciones del servidor MCP (Model Context Protocol) y ejecute Hermes como servidor MCP.| Subcomando | Descripción |
|------------|-------------|
| *(ninguno)* o `selector` | Selector de catálogo interactivo: explore los MCP aprobados por Nous e instálelos/habilite/deshabilite. |
| `catálogo` | Enumere los MCP aprobados por Nous (texto sin formato, programables). |
| `instalar <nombre>` | Instale una entrada de catálogo (por ejemplo, `hermes mcp install n8n`). |
| `servir [-v\|--verbose]` | Ejecute Hermes como servidor MCP: exponga las conversaciones a otros agentes. |
| `agregar <nombre> [--url URL] [--command CMD] [--auth oauth\|encabezado] [--args ...]` | Agregue un servidor MCP personalizado con descubrimiento automático de herramientas. `--args` pasa el argv restante al comando stdio, así que colóquelo al final. |
| `eliminar <nombre>` (alias: `rm`) | Elimine un servidor MCP de la configuración. |
| `lista` (alias: `ls`) | Enumere los servidores MCP configurados. |
| `prueba <nombre>` | Pruebe la conexión a un servidor MCP. |
| `configurar <nombre>` (alias: `config`) | Alternar selección de herramientas para un servidor. |
| `iniciar sesión <nombre>` | Fuerce la reautenticación para un servidor MCP basado en OAuth. |Consulte [Referencia de configuración de MCP] (./mcp-config-reference.md), [Usar MCP con Hermes] (../guides/use-mcp-with-hermes.md) y [Modo de servidor MCP] (../user-guide/features/mcp.md#running-hermes-as-an-mcp-server).## `complementos de hermes````golpecito
complementos de hermes [subcomando]
```Gestión unificada de complementos: complementos generales, proveedores de memoria y motores de contexto en un solo lugar. Al ejecutar `hermes plugins` sin subcomando se abre una pantalla interactiva compuesta con dos secciones:- **Complementos generales**: casillas de selección múltiple para habilitar/deshabilitar los complementos instalados
- **Complementos de proveedor**: configuración de selección única para el proveedor de memoria y el motor de contexto. Presione ENTER en una categoría para abrir un selector de radio.| Subcomando | Descripción |
|------------|-------------|
| *(ninguno)* | Interfaz de usuario interactiva compuesta: alternancia de complementos generales + configuración de complementos del proveedor. |
| `instalar <identificador> [--force]` | Instale un complemento desde una URL de Git o "propietario/repositorio". |
| `actualizar <nombre>` | Obtenga los últimos cambios para un complemento instalado. |
| `eliminar <nombre>` (alias: `rm`, `desinstalar`) | Eliminar un complemento instalado. |
| `habilitar <nombre>` | Habilite un complemento deshabilitado. |
| `deshabilitar <nombre>` | Deshabilite un complemento sin eliminarlo. |
| `lista` (alias: `ls`) | Enumere los complementos instalados con estado habilitado/deshabilitado. |Las selecciones de complementos del proveedor se guardan en `config.yaml`:
- `memory.provider` — proveedor de memoria activo (vacío = solo integrado)
- `context.engine` — motor de contexto activo ("compressor"` = valor predeterminado incorporado)La lista general de complementos deshabilitados se almacena en `config.yaml` en `plugins.disabled`.Consulte [Complementos](../user-guide/features/plugins.md) y [Crear un complemento de Hermes](../guides/build-a-hermes-plugin.md).## `herramientas hermes````golpecito
herramientas hermes [--resumen]
```| Opción | Descripción |
|--------|-------------|
| `--resumen` | Imprima el resumen actual de herramientas habilitadas y salga. |Sin `--summary`, esto inicia la interfaz de usuario de configuración de la herramienta interactiva por plataforma.## `uso de computadora hermes````golpecito
uso de la computadora hermes <subcomando>
```Subcomandos:| Subcomando | Descripción |
|------------|-------------|
| `instalar` | Ejecute el instalador ascendente de cua-driver (solo macOS). |
| `instalar --actualizar` | Vuelva a ejecutar el instalador incluso si cua-driver ya está en PATH. El script ascendente siempre extrae la última versión, por lo que realiza una actualización in situ. |
| `estado` | Imprima si `cua-driver` está en `$PATH` y qué versión está instalada. |`hermes computer-use install` es el punto de entrada estable para instalar el
[cua-driver](https://github.com/trycua/cua) binario utilizado por el
Conjunto de herramientas `computer_use`. Ejecuta el mismo instalador ascendente que
`hermes tools` se invoca cuando habilitas el uso de computadora por primera vez, por lo que es seguro
para usar para volver a ejecutar la instalación si la alternancia del conjunto de herramientas no se activó
(por ejemplo, en configuraciones de usuarios recurrentes).`hermes update` vuelve a ejecutar automáticamente el instalador ascendente al final
de la actualización si cua-driver está en PATH, por lo que la mayoría de los usuarios no necesitarán
llame a `--upgrade` manualmente. Úselo cuando upstream envíe una solución que desee
ahora mismo sin esperar la próxima actualización de Hermes.## `sesiones de hermes````golpecito
sesiones de hermes <subcomando>
```Subcomandos:| Subcomando | Descripción |
|------------|-------------|
| `lista` | Lista de sesiones recientes. |
| `navegar` | Selector de sesiones interactivo con búsqueda y currículum. |
| `exportar <salida> [--ID de sesión]` | Exportar sesiones a JSONL. |
| `eliminar <id-sesión>` | Eliminar una sesión. |
| `poda` | Eliminar sesiones antiguas. |
| `estadísticas` | Mostrar estadísticas del almacén de sesiones. |
| `renombrar <id-sesión> <título>` | Establecer o cambiar el título de una sesión. |## `perspectivas de Hermes````golpecito
ideas de hermes [--días N] [--plataforma fuente]
```| Opción | Descripción |
|--------|-------------|
| `--días <n>` | Analizar los últimos `n` días (predeterminado: 30). |
| `--source <plataforma>` | Filtre por fuente como "cli", "telegram" o "discord". |## `garra de hermes````golpecito
Hermes garra migrar [opciones]
```Migre su configuración de OpenClaw a Hermes. Lee desde `~/.openclaw` (o una ruta personalizada) y escribe en `~/.hermes`. Detecta automáticamente nombres de directorios heredados (`~/.clawdbot`, `~/.moltbot`) y nombres de archivos de configuración (`clawdbot.json`, `moltbot.json`).| Opción | Descripción |
|--------|-------------|
| `--ejecución en seco` | Obtenga una vista previa de lo que se migraría sin escribir nada. |
| `--preset <nombre>` | Ajuste preestablecido de migración: "completo" (todas las configuraciones compatibles) o "datos de usuario" (excluye la configuración de infraestructura). Ninguno de los ajustes preestablecidos importa secretos: pase `--migrate-secrets` explícitamente. |
| `--sobrescribir` | Sobrescriba los archivos Hermes existentes en caso de conflictos (predeterminado: rechace aplicar cuando el plan tenga conflictos). |
| `--migrar-secretos` | Incluir claves API en la migración. Requerido incluso bajo `--preset full`. |
| `--sin-copia de seguridad` | Omita la instantánea zip previa a la migración de `~/.hermes/` (de forma predeterminada, se escribe un único archivo de punto de restauración en `~/.hermes/backups/pre-migration-*.zip` antes de aplicar; se puede restaurar con `hermes import`). |
| `--fuente <ruta>` | Directorio OpenClaw personalizado (predeterminado: `~/.openclaw`). |
| `--workspace-target <ruta>` | Directorio de destino para las instrucciones del espacio de trabajo (AGENTS.md). |
| `--skill-conflict <modo>` | Manejar colisiones de nombres de habilidades: "omitir" (predeterminado), "sobrescribir" o "cambiar nombre". |
| `--sí` | Omita el mensaje de confirmación. |### Qué se migraLa migración cubre más de 30 categorías entre personas, memoria, habilidades, proveedores de modelos, plataformas de mensajería, comportamiento de los agentes, políticas de sesión, servidores MCP, TTS y más. Los artículos se **importan directamente** a equivalentes de Hermes o se **archivan** para su revisión manual.**Importado directamente:** SOUL.md, MEMORY.md, USER.md, AGENTS.md, habilidades (4 directorios de origen), modelo predeterminado, proveedores personalizados, servidores MCP, tokens de plataforma de mensajería y listas permitidas (Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Mattermost), valores predeterminados del agente (esfuerzo de razonamiento, compresión, retraso humano, zona horaria, zona de pruebas), políticas de restablecimiento de sesión, reglas de aprobación, configuración TTS, configuración del navegador, herramienta configuración, tiempo de espera de ejecución, lista de comandos permitidos, configuración de puerta de enlace y claves API de 3 fuentes.**Archivado para revisión manual:** Trabajos cron, complementos, ganchos/webhooks, backend de memoria (QMD), configuración de registro de habilidades, UI/identidad, registro, configuración de múltiples agentes, enlaces de canales, IDENTITY.md, TOOLS.md, HEARTBEAT.md, BOOTSTRAP.md.**Resolución de clave API** verifica tres fuentes en orden de prioridad: valores de configuración → `~/.openclaw/.env` → `auth-profiles.json`. Todos los campos de token manejan cadenas simples, plantillas de entorno ("${VAR}`) y objetos SecretRef.Para obtener la asignación completa de claves de configuración, los detalles de manejo de SecretRef y la lista de verificación posterior a la migración, consulte la **[guía de migración completa](../guides/migrate-from-openclaw.md)**.### Ejemplos```golpecito
# Vista previa de lo que se migraría
Hermes garra migrar - ejecución en seco# Migración completa (todas las configuraciones compatibles, sin secretos)
Hermes garra migrar --preset completo# Migración completa incluyendo claves API
Hermes garra migrar --preset completo --migrar-secretos# Migrar solo datos de usuario (sin secretos), sobrescribir conflictos
migración de garra de hermes --datos de usuario preestablecidos --sobrescribir# Migrar desde una ruta personalizada de OpenClaw
migrar la garra de Hermes --fuente /home/user/old-openclaw
```## `panel de control de Hermes````golpecito
Panel de control de Hermes [opciones]
```Inicie el panel web: una interfaz de usuario basada en navegador para administrar la configuración, las claves API y las sesiones de monitoreo. Requiere `cd ~/.hermes/hermes-agent && uv pip install -e ".[web]"` (FastAPI + Uvicorn). La pestaña Chat del navegador integrado siempre está disponible y además necesita el extra `pty` (`cd ~/.hermes/hermes-agent && uv pip install -e ".[web,pty]"`) más un entorno POSIX PTY como Linux, macOS o WSL2. Consulte [Panel web](/user-guide/features/web-dashboard) para obtener la documentación completa.| Opción | Predeterminado | Descripción |
|--------|---------|-------------|
| `--puerto` | `9119` | Puerto para ejecutar el servidor web |
| `--host` | `127.0.0.1` | Dirección de enlace |
| `--no-abierto` | — | No abrir automáticamente el navegador |
| `--inseguro` | apagado | Permitir el enlace a hosts no locales. Expone las credenciales del panel en la red; Úselo solo detrás de controles de red confiables. |
| `--aislado` | apagado | Cuando se inicia desde un perfil con nombre ("panel de trabajo"), ejecute un servidor dedicado por perfil en lugar de enrutarlo al panel de la máquina. |
| `--detener` | — | Deje de ejecutar los procesos del "panel de control de Hermes" y salga. |
| `--estado` | — | Enumere los procesos en ejecución del `hermes panel` y salga. |```golpecito
# Predeterminado: abre el navegador en http://127.0.0.1:9119
tablero de hermes# Puerto personalizado, sin navegador
tablero de hermes --puerto 8080 --no abierto# Desde un alias de perfil: rutas al panel de la máquina con el
# perfil preseleccionado en el selector de la barra lateral (adjuntar si se está ejecutando)
tablero del trabajador
```## `perfil de hermes````golpecito
perfil de hermes <subcomando>
```Administre perfiles: múltiples instancias aisladas de Hermes, cada una con su propia configuración, sesiones, habilidades y directorio de inicio.| Subcomando | Descripción |
|------------|-------------|
| `lista` | Lista todos los perfiles. |
| `usar <nombre>` | Establece un perfil predeterminado fijo. |
| `crear <nombre> [--clone] [--clone-all] [--clone-from <fuente>] [--no-alias]` | Crea un nuevo perfil. `--clone` copia la configuración, `.env`, `SOUL.md` y las habilidades del perfil activo. `--clone-all` copia todo el estado. `--clone-from` especifica un perfil de origen e implica clonación de configuración a menos que se combine con `--clone-all`. |
| `eliminar <nombre> [-y]` | Eliminar un perfil. |
| `mostrar <nombre>` | Mostrar detalles del perfil (directorio de inicio, configuración, etc.). |
| `alias <nombre> [--remove] [--name NOMBRE]` | Administre scripts contenedores para un acceso rápido al perfil. |
| `renombrar <antiguo> <nuevo>` | Cambiar el nombre de un perfil. |
| `exportar <nombre> [-o ARCHIVO]` | Exporte un perfil a un archivo `.tar.gz` (copia de seguridad local). |
| `importar <archivo> [--name NOMBRE]` | Importe un perfil desde un archivo `.tar.gz` (restauración local). |
| `instalar <fuente> [--nombre N] [--alias] [--force] [-y]` | Instale una distribución de perfil desde una URL de git o un directorio local. |
| `actualizar <nombre> [--force-config] [-y]` | Volver a tirar de una distribución; conserva los datos del usuario (memorias, sesiones, autenticación). |
| `información <nombre>` | Mostrar el manifiesto de distribución de un perfil (versión, requisitos, fuente). |Ejemplos:```golpecito
lista de perfiles de hermes
perfil de hermes crear trabajo --clonar
trabajo de uso del perfil de hermes
perfil de hermes alias trabajo --nombre h-trabajo
trabajo de exportación de perfil de hermes -o work-backup.tar.gz
Importación de perfil de Hermes work-backup.tar.gz --nombre restaurado
Instalación del perfil de Hermes en github.com/user/my-distro --alias
Trabajo de actualización del perfil de Hermes.
hermes -p chat de trabajo -q "Hola desde el perfil de trabajo"
```## `finalización de Hermes````golpecito
finalización de hermes [bash|zsh|fish]
```Imprima un script de finalización del shell en la salida estándar. Obtenga la salida en su perfil de shell para completar con pestañas los comandos, subcomandos y nombres de perfiles de Hermes.Ejemplos:```golpecito
# Golpe
bash de finalización de hermes >> ~/.bashrc#Zsh
finalización de hermes zsh >> ~/.zshrc# pescado
pez de finalización de hermes > ~/.config/fish/completions/hermes.fish
```## `actualización de Hermes````golpecito
actualización de hermes [--gateway] [--check] [--no-backup] [--backup] [--yes]
```Extrae el último código `hermes-agent` y reinstala las dependencias en el venv administrado, luego vuelve a ejecutar los enlaces posteriores a la instalación (servidores MCP, sincronización de habilidades, instalación completa). Es seguro ejecutarlo en una instalación en vivo. Utilice `--check` para ver si su pago está detrás de `origin/main` sin realizar la instalación.`hermes update` extrae la rama de actualización configurada (predeterminada: `main`). Si su pago se realiza en otra rama, Hermes puede verificar la rama de actualización antes de realizar la extracción. Confirme el trabajo de la rama antes de actualizar cuando desee mantenerlo fuera del flujo de actualización automática.| Opción | Descripción |
|--------|-------------|
| `--puerta de enlace` | Modo interno utilizado por el comando de mensajería `/update`. Utiliza IPC basado en archivos para indicaciones y transmisión de progreso en lugar de leer desde la entrada estándar del terminal. No es un indicador de reinicio de la puerta de enlace. |
| `--verificar` | Compruebe si hay una actualización disponible sin extraer, instalar dependencias ni reiniciar nada. |
| `--sin-copia de seguridad` | Omita la copia de seguridad previa a la actualización para esta ejecución, incluso si `updates.pre_update_backup` está habilitado en `config.yaml`. |
| `--copia de seguridad` | Cree una instantánea previa a la actualización etiquetada de `HERMES_HOME` (configuración, autenticación, sesiones, habilidades, datos de emparejamiento) antes de extraerla. El valor predeterminado es **desactivado**: el comportamiento anterior de realizar siempre una copia de seguridad agregaba minutos a cada actualización en hogares grandes. Actívelo permanentemente a través de `updates.pre_update_backup: true` en `config.yaml`. |
| `--sí`, `-y` | Suponga que sí para indicaciones interactivas como migración de configuración y restauración de almacenamiento oculto. Se omite la entrada de la clave API; ejecute `hermes config migre` por separado para esos. |Comportamiento adicional:- **Reinicio de la puerta de enlace.** Después de una actualización exitosa, Hermes intenta reiniciar automáticamente todos los perfiles de puerta de enlace en ejecución para que recojan el nuevo código. Utilice `hermes gateway restart` cuando desee reiniciar una puerta de enlace sin aplicar una actualización.
- **Cambios en el origen local.** Para las instalaciones de git, los archivos sucios con seguimiento y los archivos sin seguimiento se guardan automáticamente antes de la extracción o extracción de la rama (`git stash push --include-untracked`). Las actualizaciones del terminal interactivo se solicitan antes de restaurar el alijo. Las actualizaciones no interactivas lo restauran de forma predeterminada; configure `updates.non_interactive_local_changes: descartar` solo en instalaciones administradas donde las ediciones de fuentes locales deben descartarse después de una extracción exitosa. Si la restauración del alijo entra en conflicto o la extracción falla, el alijo se deja en su lugar para su recuperación manual.
- **npm lockfile churn.** Antes de ocultar o cambiar de rama, Hermes realiza un esfuerzo de limpieza de las diferencias rastreadas de `package-lock.json` producidas por los pasos de instalación/compilación de npm. Confirme o guarde manualmente las ediciones intencionales del archivo de bloqueo antes de ejecutar la "actualización de Hermes".
- **Instantánea de datos de emparejamiento.** Incluso cuando `--backup` está desactivado, `hermes update` toma una instantánea ligera de `~/.hermes/pairing/` y las reglas de comentarios de Feishu antes de `git pull`. Puede revertirlo con `hermes backup recovery --state pre-update` si una extracción reescribe un archivo que estaba editando.
- **Advertencia de `hermes.service` heredado.** Si Hermes detecta una unidad systemd `hermes.service` previamente renombrada (en lugar del actual `hermes-gateway.service`), imprime una sugerencia de migración única para que pueda evitar problemas de flap-loop.
- **Códigos de salida.** `0` en caso de éxito, `1` en errores de extracción/instalación/posteriores a la instalación, `2` en cambios inesperados en el árbol de trabajo que bloquean `git pull`.## Comandos de mantenimiento| Comando | Descripción |
|---------|-------------|
| `versión hermes` | Imprimir información de la versión. |
| `actualización de Hermes` | Extraiga los últimos cambios y reinstale las dependencias. |
| `hermes postinstalación` | Arranque interno. Se ejecuta una vez después de que el script de instalación aprovisiona Hermes (o después de la `actualización de hermes`) para instalar dependencias que no son de Python y que pip no puede proporcionar (tiempo de ejecución de Node.js, navegador sin cabeza, ripgrep, ffmpeg) y luego activa la `configuración de hermes` si el perfil aún no se ha configurado. Es seguro volver a ejecutarlo de forma idempotente. |
| `desinstalación de hermes [--full] [--gui] [--yes]` | Elimine Hermes y, opcionalmente, elimine todas las configuraciones/datos. `--gui` elimina solo la GUI de Chat del escritorio, dejando el agente intacto; `--full` también elimina configuración/datos; `--yes` omite las indicaciones. |## Ver también- [Referencia de comandos de barra diagonal] (./slash-commands.md)
- [Interfaz CLI] (../user-guide/cli.md)
- [Sesiones](../user-guide/sessions.md)
- [Sistema de habilidades](../user-guide/features/skills.md)
- [Máscaras y temas](../user-guide/features/skins.md)---