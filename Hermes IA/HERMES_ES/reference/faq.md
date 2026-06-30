<!-- fuente: sitio web/docs/reference/faq.md -->
# Preguntas frecuentes y solución de problemas# Preguntas frecuentes y solución de problemasRespuestas rápidas y soluciones para las preguntas y problemas más comunes.---## Preguntas frecuentes### ¿Qué proveedores de LLM trabajan con Hermes?Hermes Agent funciona con cualquier API compatible con OpenAI. Los proveedores admitidos incluyen:- **[OpenRouter](https://openrouter.ai/)**: acceda a cientos de modelos a través de una clave API (recomendado para mayor flexibilidad)
- **[Nous Portal](/integrations/nous-portal)** — Portal de suscripción de Nous Research — Más de 300 modelos más web/imagen/TTS/navegador a través de un inicio de sesión OAuth (recomendado para recién llegados)
- **OpenAI** — GPT-5.4, GPT-5-codex, GPT-4.1, GPT-4o, etc.
- **Anthropic** — Modelos Claude (API directa, OAuth a través de `hermes auth add anthropic`, OpenRouter o cualquier proxy compatible)
- **Google** — Modelos Gemini (API directa a través del proveedor `gemini`, OpenRouter o proxy compatible)
- **z.ai / ZhipuAI** — Modelos GLM
- **Kimi / Moonshot AI** — Modelos Kimi
- **MiniMax**: puntos finales globales y de China
- **Modelos locales** — a través de [Ollama](https://ollama.com/), [vLLM](https://docs.vllm.ai/), [llama.cpp](https://github.com/ggerganov/llama.cpp), [SGLang](https://github.com/sgl-project/sglang), o cualquier servidor compatible con OpenAIConfigure su proveedor con `hermes model` o editando `~/.hermes/.env`. Consulte la referencia [Variables de entorno](./environment-variables.md) para conocer todas las claves de proveedor.### ¿Funciona en Windows?**Sí, de forma nativa.** Hermes admite Windows nativo a través del instalador de PowerShell; no se requiere WSL. Ejecutar en PowerShell:```powershell
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```El instalador proporciona un PortableGit que respalda el shell de la herramienta terminal. Consulte la [Guía de Windows (nativo)] (../user-guide/windows-native.md) para obtener más detalles.WSL2 sigue siendo una alternativa totalmente compatible. Para ejecutar Hermes dentro de WSL2, instale [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install) y use el comando de instalación estándar:```golpecito
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | fiesta
```### Ejecuto Hermes en WSL2. ¿Cuál es la mejor manera de controlar mi Windows Chrome normal?Prefiere un puente MCP a `/browser connect`.Patrón recomendado:- ejecutar Hermes dentro de WSL2
- sigue usando tu Chrome normal con sesión iniciada en Windows
- agregue `chrome-devtools-mcp` como servidor MCP a través de `cmd.exe` o `powershell.exe`
- dejar que Hermes use las herramientas del navegador MCP resultantesEsto es más confiable que intentar forzar que el transporte del navegador principal de Hermes se conecte directamente a través del límite WSL2/Windows.Ver:- [Usar MCP con Hermes](../guides/use-mcp-with-hermes.md#wsl2-bridge-hermes-in-wsl-to-windows-chrome)
- [Automatización del navegador](../user-guide/features/browser.md#wsl2--windows-chrome-prefer-mcp-over-browser-connect)### ¿Funciona en Android/Termux?Sí, Hermes ahora tiene una ruta de instalación de Termux probada para teléfonos Android.Instalación rápida:```golpecito
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | fiesta
```Para conocer los pasos manuales totalmente explícitos, los extras admitidos y las limitaciones actuales, consulte la [guía de Termux](../getting-started/termux.md).Advertencia importante: el extra `.[all]` completo no está disponible actualmente en Android porque el extra `voice` depende de `faster-whisper` → `ctranslate2`, y `ctranslate2` no publica ruedas de Android. Utilice el extra `.[termux]` probado en su lugar.### ¿Se envían mis datos a algún lugar?Las llamadas API van **solo al proveedor LLM que configure** (por ejemplo, OpenRouter, su instancia local de Ollama). Hermes Agent no recopila telemetría, datos de uso ni análisis. Tus conversaciones, memoria y habilidades se almacenan localmente en `~/.hermes/`.### ¿Puedo usarlo sin conexión o con modelos locales?Sí. Ejecute `hermes model`, seleccione **Punto final personalizado** e ingrese la URL de su servidor:```golpecito
modelo hermes
# Seleccione: Punto final personalizado (ingrese la URL manualmente)
# URL base de API: http://localhost:11434/v1
# Clave API: ollama
# Nombre del modelo: qwen3.5:27b
# Longitud del contexto: 64000 ← mínimo de Hermes; configúrelo para que coincida con la ventana de contexto real de su servidor
```O configúrelo directamente en `config.yaml`:```yaml
modelo:
  predeterminado: qwen3.5:27b
  proveedor: personalizado
  URL_base: http://localhost:11434/v1
```Hermes conserva el punto final, el proveedor y la URL base en `config.yaml` para que sobreviva a los reinicios. Si su servidor local tiene exactamente un modelo cargado, `/model custom` lo detecta automáticamente. También puedes configurar `proveedor: personalizado` en config.yaml; es un proveedor de primera clase, no un alias para nada más.Esto funciona con Ollama, vLLM, servidor llama.cpp, SGLang, LocalAI y otros. Consulte la [Guía de configuración](../user-guide/configuration.md) para obtener más detalles.:::tip usuarios de Ollama
Si configura un `num_ctx` personalizado en Ollama (por ejemplo, `ollama run --num_ctx 64000`), asegúrese de establecer la longitud del contexto coincidente en Hermes: `/api/show` de Ollama informa el contexto *máximo* del modelo, no el `num_ctx` efectivo que configuró.
::::::tip Tiempos de espera con modelos locales
Hermes detecta automáticamente los puntos finales locales y relaja los tiempos de espera de transmisión (el tiempo de espera de lectura aumentó de 120 a 1800, la detección de transmisión obsoleta está deshabilitada). Si aún alcanza tiempos de espera en contextos muy grandes, configure `HERMES_STREAM_READ_TIMEOUT=1800` en su `.env`. Consulte la [guía local de LLM] (../guides/local-llm-on-mac.md#timeouts) para obtener más detalles.
:::### ¿Cuánto cuesta?Hermes Agent en sí es **gratuito y de código abierto** (licencia MIT). Paga solo por el uso de la API LLM del proveedor elegido. Los modelos locales son completamente gratuitos.### ¿Pueden varias personas usar una instancia?Sí. La [puerta de enlace de mensajería] (../user-guide/messaging/index.md) permite que varios usuarios interactúen con la misma instancia del Agente Hermes a través de Telegram, Discord, Slack, WhatsApp o Home Assistant. El acceso se controla a través de listas permitidas (ID de usuario específicas) y emparejamiento de DM (el primer usuario que envía el mensaje reclama acceso).### ¿Cuál es la diferencia entre memoria y habilidades?- **La memoria** almacena **hechos**: cosas que el agente sabe sobre usted, sus proyectos y preferencias. Los recuerdos se recuperan automáticamente según su relevancia.
- **Tienda de habilidades** **procedimientos**: instrucciones paso a paso sobre cómo hacer las cosas. Las habilidades se recuerdan cuando el agente se enfrenta a una tarea similar.Ambos persisten a lo largo de las sesiones. Consulte [Memoria](../user-guide/features/memory.md) y [Habilidades](../user-guide/features/skills.md) para obtener más detalles.### ¿Puedo usarlo en mi propio proyecto Python?Sí. Importe la clase `AIAgent` y use Hermes programáticamente:```pitón
desde run_agent importar AIAgentagente = AIAgent(model="anthropic/claude-opus-4.7")
respuesta = agente.chat("Explique brevemente la computación cuántica")
```Consulte la [guía de la biblioteca Python](../user-guide/features/code-execution.md) para conocer el uso completo de la API.---## Solución de problemas### Problemas de instalación#### `hermes: comando no encontrado` después de la instalación**Causa:** Su shell no ha recargado la RUTA actualizada.**Solución:**
```golpecito
# Recarga tu perfil de shell
fuente ~/.bashrc # bash
fuente ~/.zshrc # zsh# O iniciar una nueva sesión de terminal
```Si aún no funciona, verifique la ubicación de instalación:
```golpecito
cual hermes
ls ~/.local/bin/hermes
```:::consejo
El instalador agrega `~/.local/bin` a su RUTA. Si utiliza una configuración de shell no estándar, agregue `export PATH="$HOME/.local/bin:$PATH"` manualmente.
:::#### La versión de Python es demasiado antigua**Causa:** Hermes requiere Python 3.11 o posterior.**Solución:**
```golpecito
python3 --version # Verificar la versión actual# Instalar un Python más nuevo
sudo apt instalar python3.12 # Ubuntu/Debian
cerveza instalar python@3.12 # macOS
```El instalador maneja esto automáticamente; si ve este error durante la instalación manual, primero actualice Python.#### Los comandos de terminal dicen `nodo: comando no encontrado` (o `nvm`, `pyenv`, `asdf`,…)**Causa:** Hermes crea una instantánea del entorno por sesión ejecutando `bash -l` una vez al inicio. Un shell de inicio de sesión de bash lee `/etc/profile`, `~/.bash_profile` y `~/.profile`, pero **no genera `~/.bashrc`**, por lo que las herramientas que se instalan allí (`nvm`, `asdf`, `pyenv`, `cargo`, exportaciones personalizadas de `PATH`) permanecen invisibles en la instantánea. Esto sucede más comúnmente cuando Hermes se ejecuta bajo systemd o en un shell mínimo donde nada ha precargado el perfil de shell interactivo.**Solución:** Hermes genera automáticamente `~/.bashrc` de forma predeterminada. Si eso no es suficiente, p.e. eres un usuario de zsh cuya RUTA se encuentra en `~/.zshrc`, o inicias `nvm` desde un archivo independiente; enumera los archivos adicionales para obtener en `~/.hermes/config.yaml`:```yaml
terminales:
  archivos_init_shell:
    - ~/.zshrc # usuarios de zsh: introduce la RUTA administrada por zsh en la instantánea de bash
    - ~/.nvm/nvm.sh # inicio directo de nvm (funciona independientemente del shell)
    - /etc/profile.d/cargo.sh # archivos rc de todo el sistema
  # Cuando se establece esta lista, NO se agrega la fuente automática ~/.bashrc predeterminada:
  # inclúyelo explícitamente si quieres ambos:
  # - ~/.bashrc
  # - ~/.zshrc
```Los archivos faltantes se omiten silenciosamente. El abastecimiento ocurre en bash, por lo que los archivos que dependen de la sintaxis exclusiva de zsh pueden generar errores; si eso le preocupa, obtenga solo la parte de configuración de PATH (por ejemplo, `nvm.sh` de nvm directamente) en lugar de todo el archivo rc.Para deshabilitar el comportamiento de fuente automática (solo semántica estricta de shell de inicio de sesión):```yaml
terminales:
  auto_source_bashrc: falso
```#### `uv: comando no encontrado`**Causa:** El administrador de paquetes `uv` no está instalado o no está en PATH.**Solución:**
```golpecito
rizo -LsSf https://astral.sh/uv/install.sh | sh
fuente ~/.bashrc
```#### Errores de permiso denegado durante la instalación**Causa:** Permisos insuficientes para escribir en el directorio de instalación.**Solución:**
```golpecito
# No utilices sudo con el instalador: se instala en ~/.local/bin
# Si previamente instaló con sudo, limpie:
sudo rm /usr/local/bin/hermes
# Luego vuelva a ejecutar el instalador estándar.
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | fiesta
```---### Problemas con proveedores y modelos#### `/model` solo muestra un proveedor/no se puede cambiar de proveedor**Causa:** `/model` (dentro de una sesión de chat) solo puede cambiar entre proveedores que **ya hayas configurado**. Si solo ha configurado OpenRouter, eso es todo lo que se mostrará "/model".**Solución:** Salga de su sesión y use `hermes model` desde su terminal para agregar nuevos proveedores:```golpecito
# Salga primero de la sesión de chat de Hermes (Ctrl+C o /salir)# Ejecute el asistente de configuración completo del proveedor
modelo hermes# Esto le permite: agregar proveedores, ejecutar OAuth, ingresar claves API, configurar puntos finales
```Después de agregar un nuevo proveedor a través de "hermes model", inicie una nueva sesión de chat; "/model" ahora mostrará todos sus proveedores configurados.:::consejo Referencia rápida
| ¿Quieres... | Uso |
|-----------|-----|
| Agregar un nuevo proveedor | `modelo hermes` (desde la terminal) |
| Ingresar/cambiar claves API | `modelo hermes` (desde la terminal) |
| Cambiar de modelo a mitad de sesión | `/modelo <nombre>` (dentro de la sesión) |
| Cambiar a un proveedor configurado diferente | `/proveedor de modelo:model` (sesión interna) |
:::#### La clave API no funciona**Causa:** Falta la clave, está caducada, está configurada incorrectamente o es del proveedor incorrecto.**Solución:**
```golpecito
# Verifica tu configuración
espectáculo de configuración de hermes# Reconfigura tu proveedor
modelo hermes# O configurar directamente
conjunto de configuración de hermes OPENROUTER_API_KEY sk-or-v1-xxxxxxxxxxxx
```:::advertencia
Asegúrese de que la clave coincida con el proveedor. Una clave OpenAI no funcionará con OpenRouter y viceversa. Verifique `~/.hermes/.env` para ver si hay entradas conflictivas.
:::#### Modelo no disponible/modelo no encontrado**Causa:** El identificador del modelo es incorrecto o no está disponible en su proveedor.**Solución:**
```golpecito
# Lista de modelos disponibles para su proveedor
modelo hermes# Establecer un modelo válido
conjunto de configuración de hermes HERMES_MODEL antrópico/claude-opus-4.7# O especificar por sesión
chat de hermes --modelo openrouter/meta-llama/llama-3.1-70b-instruct
```#### Limitación de velocidad (429 errores)**Causa:** Has excedido los límites de tarifas de tu proveedor.**Solución:** Espere un momento y vuelva a intentarlo. Para un uso sostenido, considere:
- Actualización de su plan de proveedor
- Cambiar a un modelo o proveedor diferente
- Uso de `hermes chat --provider <alternative>` para enrutar a un backend diferente#### Se excedió la longitud del contexto**Causa:** La conversación se ha prolongado demasiado para la ventana de contexto del modelo o Hermes detectó una longitud de contexto incorrecta para su modelo.**Solución:**
```golpecito
# Comprimir la sesión actual
/comprimir# O iniciar una nueva sesión
chatear con hermes# Utilice un modelo con una ventana de contexto más grande
chat de hermes --modelo openrouter/google/gemini-3-flash-preview
```Si esto sucede en la primera conversación larga, es posible que Hermes tenga una longitud de contexto incorrecta para su modelo. Comprueba lo que detectó:Mire la línea de inicio de la CLI: muestra la longitud del contexto detectado (por ejemplo, `📊 Límite de contexto: 128000 tokens`). También puede consultar con `/usage` durante una sesión.Para corregir la detección de contexto, configúrelo explícitamente:```yaml
# En ~/.hermes/config.yaml
modelo:
  predeterminado: nombre-de-su-modelo
  context_length: 131072 # ventana de contexto real de su modelo
```O para puntos finales personalizados, agréguelo por modelo:```yaml
proveedores_personalizados:
  - nombre: "Mi servidor"
    base_url: "http://localhost:11434/v1"
    modelos:
      qwen3.5:27b:
        longitud_contexto: 64000
```Consulte [Detección de longitud de contexto](../integrations/providers.md#context-length-detection) para saber cómo funciona la detección automática y todas las opciones de anulación.---### Problemas con terminales#### Comando bloqueado por ser peligroso**Causa:** Hermes detectó un comando potencialmente destructivo (por ejemplo, `rm -rf`, `DROP TABLE`). Esta es una característica de seguridad.**Solución:** Cuando se le solicite, revise el comando y escriba "y" para aprobarlo. También puedes:
- Pídale al agente que utilice una alternativa más segura.
- Consulte la lista completa de patrones peligrosos en los [Documentos de seguridad] (../user-guide/security.md):::consejo
Esto funciona según lo previsto: Hermes nunca ejecuta silenciosamente comandos destructivos. El mensaje de aprobación le muestra exactamente qué se ejecutará.
:::#### `sudo` no funciona a través de la puerta de enlace de mensajería**Causa:** La puerta de enlace de mensajería se ejecuta sin una terminal interactiva, por lo que `sudo` no puede solicitar una contraseña.**Solución:**
- Evite `sudo` en los mensajes: pídale al agente que busque alternativas
- Si debe usar `sudo`, configure sudo sin contraseña para comandos específicos en `/etc/sudoers`
- O cambiar a la interfaz del terminal para tareas administrativas: `hermes chat`#### El backend de Docker no se conecta**Causa:** El demonio Docker no se está ejecutando o el usuario carece de permisos.**Solución:**
```golpecito
# Comprobar que Docker se esté ejecutando
información de la ventana acoplable# Agrega tu usuario al grupo de Docker
sudo usermod -aG ventana acoplable $USUARIO
ventana acoplable newgrp# Verificar
docker ejecuta hola mundo
```---### Problemas de mensajería#### Bot no responde a los mensajes**Causa:** El bot no se está ejecutando, no está autorizado o su usuario no está en la lista de permitidos.**Solución:**
```golpecito
# Comprobar si la puerta de enlace se está ejecutando
estado de la puerta de enlace de Hermes# Iniciar la puerta de enlace
inicio de la puerta de enlace de hermes# Verifique los registros en busca de errores
gato ~/.hermes/logs/gateway.log | cola -50
```#### Mensajes que no se entregan**Causa:** Problemas de red, token de bot caducado o configuración incorrecta del webhook de la plataforma.**Solución:**
- Verifique que su token de bot sea válido con `hermes gateway setup`
- Verifique los registros de la puerta de enlace: `cat ~/.hermes/logs/gateway.log | cola -50`
- Para plataformas basadas en webhooks (Slack, WhatsApp), asegúrese de que su servidor sea de acceso público#### Confusión en la lista de permitidos: ¿quién puede hablar con el bot?**Causa:** El modo de autorización determina quién obtiene acceso.**Solución:**| Modo | Cómo funciona |
|------|-------------|
| **Lista de permitidos** | Sólo los ID de usuario enumerados en la configuración pueden interactuar |
| **Emparejamiento DM** | El primer usuario que envía un mensaje por DM reclama acceso exclusivo |
| **Abierto** | Cualquiera puede interactuar (no recomendado para producción) |Configure en `~/.hermes/config.yaml` en la configuración de su puerta de enlace. Consulte los [Documentos de mensajería] (../user-guide/messaging/index.md).#### La puerta de enlace no se inicia**Causa:** Faltan dependencias, conflictos de puertos o tokens mal configurados.**Solución:**
```golpecito
# Instalar las dependencias principales de la puerta de enlace de mensajería
cd ~/.hermes/hermes-agent && uv pip install -e ".[messaging]" # Telegram, Discord, Slack y departamentos de puerta de enlace compartida# Verifique si hay conflictos de puertos
lsof -i :8080# Verificar configuración
espectáculo de configuración de hermes
```#### WSL: La puerta de enlace sigue desconectándose o falla el inicio de la puerta de enlace de Hermes**Causa:** El soporte systemd de WSL no es confiable. Muchas instalaciones de WSL2 no tienen systemd habilitado e incluso cuando están habilitados, es posible que los servicios no sobrevivan a los reinicios de WSL o a los cierres inactivos de Windows.**Solución:** Utilice el modo de primer plano en lugar del servicio systemd:```golpecito
# Opción 1: primer plano directo (más simple)
ejecución de la puerta de enlace de Hermes# Opción 2: Persistente a través de tmux (sobrevive al cierre del terminal)
tmux nuevo -s hermes 'ejecución de puerta de enlace de hermes'
# Volver a adjuntar más tarde: tmux adjuntar -t hermes# Opción 3: Fondo vía nohup
ejecución de puerta de enlace de nohup hermes > ~/.hermes/logs/gateway.log 2>&1 &
```Si quieres probar systemd de todos modos, asegúrate de que esté habilitado:1. Abra `/etc/wsl.conf` (créelo si no existe)
2. Agregue:
   ```ini
   [arranque]
   sistemad=verdadero
   ```
3. Desde PowerShell: `wsl --shutdown`
4. Vuelva a abrir su terminal WSL
5. Verifique: `systemctl is-system-running` debería decir "en ejecución" o "degradado":::consejo Inicio automático al iniciar Windows
Para un inicio automático confiable, use el Programador de tareas de Windows para iniciar WSL + la puerta de enlace al iniciar sesión:
1. Cree una tarea que ejecute `wsl -d Ubuntu -- bash -lc 'hermes gateway run'`
2. Configúrelo para que se active al iniciar sesión el usuario.
:::#### macOS: Node.js/ffmpeg/otras herramientas no encontradas por la puerta de enlace**Causa:** los servicios launchd heredan una RUTA mínima (`/usr/bin:/bin:/usr/sbin:/sbin`) que no incluye directorios de herramientas Homebrew, nvm, cargo u otros instalados por el usuario. Esto comúnmente rompe el puente de WhatsApp (`nodo no encontrado`) o la transcripción de voz (`ffmpeg no encontrado`).**Solución:** La puerta de enlace captura la RUTA de su shell cuando ejecuta `hermes gateway install`. Si instaló herramientas después de configurar la puerta de enlace, vuelva a ejecutar la instalación para capturar la RUTA actualizada:```golpecito
instalación de hermes gateway # Vuelve a tomar instantáneas de tu RUTA actual
inicio de puerta de enlace de hermes # Detecta el plist actualizado y recarga
```Puede verificar que el plist tenga la RUTA correcta:
```golpecito
/usr/libexec/PlistBuddy -c "Imprimir:Variables del entorno:RUTA" \
  ~/Library/LaunchAgents/ai.hermes.gateway.plist
```---### Problemas de rendimiento#### Respuestas lentas**Causa:** Modelo grande, servidor API distante o sistema pesado con muchas herramientas.**Solución:**
- Pruebe un modelo más rápido/más pequeño: `hermes chat --model openrouter/meta-llama/llama-3.1-8b-instruct`
- Reducir los conjuntos de herramientas activos: `hermes chat -t "terminal"`
- Verifique la latencia de su red con el proveedor.
- Para los modelos locales, asegúrese de tener suficiente GPU VRAM#### Alto uso de tokens**Causa:** Conversaciones largas, indicaciones detalladas del sistema o muchas llamadas a herramientas que acumulan contexto.**Solución:**
```golpecito
# Comprimir la conversación para reducir tokens
/comprimir# Verificar el uso del token de sesión
/uso
```:::consejo
Utilice `/compress` regularmente durante sesiones largas. Resume el historial de conversaciones y reduce significativamente el uso de tokens al tiempo que preserva el contexto.
:::#### La sesión se está volviendo demasiado larga**Causa:** Las conversaciones extendidas acumulan mensajes y resultados de herramientas, acercándose a los límites del contexto.**Solución:**
```golpecito
# Comprimir la sesión actual (conserva el contexto clave)
/comprimir# Iniciar una nueva sesión con una referencia a la anterior
chatear con hermes# Reanudar una sesión específica más tarde si es necesario
chat de hermes --continuar
```---### Problemas con MCP#### El servidor MCP no se conecta**Causa:** No se encontró el binario del servidor, ruta de comando incorrecta o falta tiempo de ejecución.**Solución:**
```golpecito
# Asegúrese de que las dependencias de MCP estén instaladas (ya incluidas en la instalación estándar)
cd ~/.hermes/hermes-agent && uv pip install -e ".[mcp]"# Para servidores basados en npm, asegúrese de que Node.js esté disponible
nodo --versión
npx --versión# Probar el servidor manualmente
npx -y @modelcontextprotocol/sistema-de-archivos-servidor /tmp
```Verifique su configuración de MCP `~/.hermes/config.yaml`:
```yaml
servidores_mcp:
  sistema de archivos:
    comando: "npx"
    argumentos: ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/docs"]
```#### Las herramientas no aparecen en el servidor MCP**Causa:** El servidor se inició pero el descubrimiento de herramientas falló, las herramientas fueron filtradas por la configuración o el servidor no admite la capacidad MCP que esperaba.**Solución:**
- Verifique los registros de puerta de enlace/agente para detectar errores de conexión de MCP
- Asegúrese de que el servidor responda al método RPC `tools/list`
- Revise cualquier configuración `tools.include`, `tools.exclude`, `tools.resources`, `tools.prompts` o `enabled` en ese servidor
- Recuerde que las herramientas de recursos/utilidades de aviso solo se registran cuando la sesión realmente admite esas capacidades.
- Utilice `/reload-mcp` después de cambiar la configuración```golpecito
# Verificar que los servidores MCP estén configurados
espectáculo de configuración de hermes | grep -A 12 servidores_mcp# Reinicie Hermes o vuelva a cargar MCP después de los cambios de configuración
chatear con hermes
```Ver también:
- [MCP (Protocolo de contexto del modelo)](/user-guide/features/mcp)
- [Usar MCP con Hermes](/guides/use-mcp-with-hermes)
- [Referencia de configuración de MCP](/reference/mcp-config-reference)#### Errores de tiempo de espera de MCP**Causa:** El servidor MCP tarda demasiado en responder o falla durante la ejecución.**Solución:**
- Aumente el tiempo de espera en la configuración de su servidor MCP si es compatible
- Compruebe si el proceso del servidor MCP todavía se está ejecutando
- Para servidores HTTP MCP remotos, verifique la conectividad de red:::advertencia
Si un servidor MCP falla en medio de una solicitud, Hermes informará un tiempo de espera. Verifique los propios registros del servidor (no solo los registros de Hermes) para diagnosticar la causa raíz.
:::---## Perfiles### ¿En qué se diferencian los perfiles de simplemente configurar HERMES_HOME?Los perfiles son una capa administrada encima de `HERMES_HOME`. *Podrías* configurar manualmente `HERMES_HOME=/some/path` antes de cada comando, pero los perfiles manejan todo el proceso por ti: crear la estructura del directorio, generar alias de shell (`hermes-work`), rastrear el perfil activo en `~/.hermes/active_profile` y sincronizar las actualizaciones de habilidades en todos los perfiles automáticamente. También se integran con la función de completar tabulaciones para que no tengas que recordar rutas.### ¿Pueden dos perfiles compartir el mismo token de bot?No. Cada plataforma de mensajería (Telegram, Discord, etc.) requiere acceso exclusivo a un token de bot. Si dos perfiles intentan utilizar el mismo token simultáneamente, la segunda puerta de enlace no podrá conectarse. Cree un bot separado por perfil: para Telegram, hable con [@BotFather](https://t.me/BotFather) para crear bots adicionales.### ¿Los perfiles comparten memoria o sesiones?No. Cada perfil tiene su propio almacén de memoria, base de datos de sesiones y directorio de habilidades. Están completamente aislados. Si desea iniciar un nuevo perfil con recuerdos y sesiones existentes, use `hermes perfil crear nuevo nombre --clone-all` para copiar todo desde el perfil actual, o agregue `--clone-from <perfil>` para copiar desde un perfil de origen específico.### ¿Qué sucede cuando ejecuto `hermes update`?`hermes update` extrae el código más reciente y reinstala las dependencias **una vez** (no por perfil). Luego sincroniza las habilidades actualizadas con todos los perfiles automáticamente. Sólo necesita ejecutar "hermes update" una vez; cubre todos los perfiles de la máquina.
### ¿Cuántos perfiles puedo ejecutar?No hay un límite estricto. Cada perfil es solo un directorio bajo `~/.hermes/profiles/`. El límite práctico depende de su espacio en disco y de cuántas puertas de enlace simultáneas puede manejar su sistema (cada puerta de enlace es un proceso ligero de Python). Ejecutar docenas de perfiles está bien; cada perfil inactivo no utiliza recursos.---## Flujos de trabajo y patrones### Uso de diferentes modelos para diferentes tareas (flujos de trabajo multimodelo)**Escenario:** Usas GPT-5.4 como tu controlador diario, pero Gemini o Grok escriben mejor contenido para redes sociales. Cambiar de modelo manualmente cada vez es tedioso.**Solución: configuración de delegación.** Hermes puede enrutar subagentes a un modelo diferente automáticamente. Configure esto en `~/.hermes/config.yaml`:```yaml
delegación:
  modelo: "google/gemini-3-flash-preview" # subagentes usan este modelo
  proveedor: "openrouter" # proveedor de subagentes
```Ahora, cuando le dices a Hermes "escríbeme un hilo de Twitter sobre X" y genera un subagente `delegate_task`, ese subagente se ejecuta en Gemini en lugar de tu modelo principal. Su conversación principal permanece en GPT-5.4.También puede ser explícito en su mensaje: *"Delegue una tarea para escribir publicaciones en las redes sociales sobre el lanzamiento de nuestro producto. Utilice su subagente para la redacción real".* El agente utilizará `delegate_task`, que selecciona automáticamente la configuración de delegación.Para cambios de modelo únicos sin delegación, use `/model` en la CLI:```golpecito
/model google/gemini-3-flash-preview # cambio para esta sesión
#...escribe tu contenido...
/model openai/gpt-5.4 # volver atrás
```Consulte [Delegación de subagente](../user-guide/features/delegation.md) para obtener más información sobre cómo funciona la delegación.### Ejecutar varios agentes en un número de WhatsApp (vinculación por chat)**Escenario:** En OpenClaw, tenía varios agentes independientes vinculados a chats específicos de WhatsApp: uno para un grupo de lista de compras familiar y otro para su chat privado. ¿Puede Hermes hacer esto?**Limitación actual:** Los perfiles de Hermes requieren cada uno su propio número/sesión de WhatsApp. No puedes vincular varios perfiles a diferentes chats en el mismo número de WhatsApp: el puente de WhatsApp (Baileys) utiliza una sesión autenticada por número.**Soluciones alternativas:**1. **Utilice un único perfil con cambio de personalidad.** Cree diferentes archivos contextuales `AGENTS.md` o utilice el comando `/personality` para cambiar el comportamiento por chat. El agente ve en qué chat está y puede adaptarse.2. **Utilice trabajos cron para tareas especializadas.** Para un rastreador de lista de compras, configure un trabajo cron que supervise un chat específico y administre la lista; no se necesita un agente independiente.3. **Utilice números separados.** Si necesita agentes verdaderamente independientes, vincule cada perfil con su propio número de WhatsApp. Los números virtuales de servicios como Google Voice funcionan para esto.4. **Utilice Telegram o Discord en su lugar.** Estas plataformas admiten la vinculación por chat de forma más natural: cada grupo de Telegram o canal de Discord tiene su propia sesión y usted puede ejecutar varios tokens de bot (uno por perfil) en la misma cuenta.Consulte [Perfiles](../user-guide/profiles.md) y [Configuración de WhatsApp](../user-guide/messaging/whatsapp.md) para obtener más detalles.### Controlar lo que aparece en Telegram (ocultar registros y razonamientos)**Escenario:** Ve los registros ejecutivos de la puerta de enlace, el razonamiento de Hermes y los detalles de las llamadas a las herramientas en Telegram en lugar de solo el resultado final.**Solución:** La configuración `display.tool_progress` en `config.yaml` controla cuánta actividad de la herramienta se muestra:```yaml
mostrar:
  tool_progress: "desactivado" # opciones: desactivado, nuevo, todo, detallado
```- **`off`** — Sólo la respuesta final. Sin llamadas a herramientas, sin razonamientos, sin registros.
- **`new`**: muestra nuevas llamadas a herramientas a medida que ocurren (breves frases ingeniosas).
- **`all`**: muestra toda la actividad de la herramienta, incluidos los resultados.
- **`verbose`**: detalles completos, incluidos los argumentos y resultados de la herramienta.Para las plataformas de mensajería, lo que normalmente desea es "desactivado" o "nuevo". Después de editar `config.yaml`, reinicie la puerta de enlace para que los cambios surtan efecto.También puedes alternar esto por sesión con el comando `/verbose` (si está habilitado):```yaml
mostrar:
  tool_progress_command: true # habilita /detallado en la puerta de enlace
```### Gestión de habilidades en Telegram (límite de comandos de barra diagonal)**Escenario:** Telegram tiene un límite de comandos de 100 barras y tus habilidades lo están superando. Quieres desactivar las habilidades que no necesitas en Telegram, pero la configuración de "hermes skills config" no parece tener efecto.**Solución:** Utilice `hermes skills config` para desactivar las habilidades por plataforma. Esto escribe en `config.yaml`:```yaml
habilidades:
  discapacitado: [] # habilidades globalmente discapacitadas
  plataforma_disabled:
    telegram: [habilidad-a, habilidad-b] # deshabilitado solo en telegram
```Después de cambiar esto, **reinicie la puerta de enlace** (`reinicio de la puerta de enlace de Hermes` o cierre y reinicie). El menú de comandos del bot de Telegram se reconstruye al inicio.:::consejo
Las habilidades con descripciones muy largas se truncan a 40 caracteres en el menú de Telegram para mantenerse dentro de los límites de tamaño de carga útil. Si las habilidades no aparecen, puede ser un problema con el tamaño total de la carga útil en lugar del límite de 100 comandos; deshabilitar las habilidades no utilizadas ayuda con ambos.
:::### Sesiones de hilos compartidos (varios usuarios, una conversación)**Escenario:** Tienes un hilo de Telegram o Discord donde varias personas mencionan el bot. Desea que todas las menciones en ese hilo formen parte de una conversación compartida, no de sesiones separadas por usuario.**Comportamiento actual:** Hermes crea sesiones codificadas por ID de usuario en la mayoría de las plataformas, de modo que cada persona obtiene su propio contexto de conversación. Esto es por diseño para privacidad y aislamiento del contexto.**Soluciones alternativas:**1. **Usa Slack.** Las sesiones de Slack se programan por subproceso, no por usuario. Varios usuarios en el mismo hilo comparten una conversación: exactamente el comportamiento que estás describiendo. Este es el ajuste más natural.2. **Utilice un chat grupal con un solo usuario.** Si una persona es el "operador" designado que transmite las preguntas, la sesión permanece unificada. Otros pueden seguir leyendo.3. **Utilice un canal de Discord.** Las sesiones de Discord se clasifican por canal, por lo que todos los usuarios del mismo canal comparten contexto. Utilice un canal dedicado para la conversación compartida.### Exportando Hermes a otra máquina**Escenario:** Ha acumulado habilidades, trabajos cron y memorias en una máquina y desea mover todo a una nueva máquina Linux dedicada.**Solución:**1. Instale Hermes Agent en la nueva máquina:
   ```golpecito
   curl -fsSL https://hermes-agent.nousresearch.com/install.sh | fiesta
   ```2. En la **máquina de origen**, cree una copia de seguridad completa:
   ```golpecito
   copia de seguridad de hermes
   ```
   Esto crea un zip de todo su directorio `~/.hermes/` (configuración, claves API, recuerdos, habilidades, sesiones y perfiles) guardado en su directorio de inicio como `~/hermes-backup-<marca de tiempo>.zip`.3. Copie el zip a la nueva máquina e impórtelo:
   ```golpecito
   # En la máquina fuente
   scp ~/hermes-backup-<marca de tiempo>.zip nueva máquina:~/# En la nueva máquina
   importación de hermes ~/hermes-backup-<marca de tiempo>.zip
   ```4. En la nueva máquina, ejecute `hermes setup` para verificar que las claves API y la configuración del proveedor estén funcionando.### Mover un solo perfil a otra máquina**Escenario:** Quiere mover o compartir un perfil específico, no su instalación completa.```golpecito
# En la máquina fuente
trabajo de exportación de perfil de hermes ./work-backup.tar.gz# Copie el archivo a la máquina de destino, luego:
importación de perfil de hermes ./work-backup.tar.gz trabajo
```El perfil importado tendrá todas las configuraciones, recuerdos, sesiones y habilidades de la exportación. Es posible que deba actualizar las rutas o volver a autenticarse con los proveedores si la nueva máquina tiene una configuración diferente.### `copia de seguridad de Hermes` frente a `exportación de perfil de Hermes`| Característica | `copia de seguridad de Hermes` | `exportación de perfil de Hermes` |
| :--- | :--- | :--- |
| **Caso de uso** | **Migración completa de la máquina** | **Transferir/compartir un perfil específico** |
| **Alcance** | Global (directorio completo `~/.hermes`) | Local (directorio de perfil único) |
| **Incluye** | Todos los perfiles, configuración global, claves API, sesiones | Perfil único: SOUL.md, recuerdos, sesiones, habilidades |
| **Credenciales** | **Incluido** (`.env` y `auth.json`) | **Excluido** (eliminado para compartir de forma segura) |
| **Formato** | `.zip` | `.tar.gz` |**Retroceso manual (rsync):** Si prefiere copiar archivos directamente, excluya el repositorio de código:
```golpecito
rsync -av --exclude='hermes-agent' ~/.hermes/ nuevamáquina:~/.hermes/
```:::consejo
`hermes backup` produce una instantánea consistente incluso mientras Hermes se está ejecutando activamente. El archivo restaurado excluye archivos de tiempo de ejecución locales de la máquina como `gateway.pid` y `cron.pid`.
:::### Permiso denegado al recargar el shell después de la instalación**Escenario:** Después de ejecutar el instalador de Hermes, `source ~/.zshrc` genera un error de permiso denegado.**Causa:** Esto suele ocurrir cuando `~/.zshrc` (o `~/.bashrc`) tiene permisos de archivo incorrectos o cuando el instalador no pudo escribir en él de forma limpia. No es un problema específico de Hermes, es un problema de permisos de configuración del shell.**Solución:**
```golpecito
# Verificar permisos
ls -la ~/.zshrc# Corregir si es necesario (debe ser -rw-r--r-- o 644)
chmod 644 ~/.zshrc# Luego recarga
fuente ~/.zshrc# O simplemente abra una nueva ventana de terminal: recoge los cambios de RUTA automáticamente
```Si el instalador agregó la línea PATH pero los permisos son incorrectos, puede agregarla manualmente:
```golpecito
echo 'exportar RUTA="$HOME/.local/bin:$RUTA"' >> ~/.zshrc
```### Error 400 en la primera ejecución del agente**Escenario:** La configuración se completa correctamente, pero el primer intento de chat falla con HTTP 400.**Causa:** Generalmente, el nombre del modelo no coincide: el modelo configurado no existe en su proveedor o la clave API no tiene acceso a él.**Solución:**
```golpecito
# Comprobar qué modelo y proveedor están configurados
espectáculo de configuración de hermes | cabeza -20# Volver a ejecutar la selección del modelo
modelo hermes# O pruebe con un modelo que se sepa que funciona bien
hermes chat -q "hola" --modelo antrópico/claude-opus-4.7
```Si usa OpenRouter, asegúrese de que su clave API tenga créditos. Un 400 de OpenRouter a menudo significa que el modelo requiere un plan pago o que la identificación del modelo tiene un error tipográfico.---## ¿Sigues atascado?Si su problema no está cubierto aquí:1. **Buscar problemas existentes:** [Problemas de GitHub](https://github.com/NousResearch/hermes-agent/issues)
2. **Pregúntale a la comunidad:** [Nous Research Discord](https://discord.gg/nousresearch)
3. **Presente un informe de error:** Incluya su sistema operativo, la versión de Python (`python3 --version`), la versión de Hermes (`hermes --version`) y el mensaje de error completo.---