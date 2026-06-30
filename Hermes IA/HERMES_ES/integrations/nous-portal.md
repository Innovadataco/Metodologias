<!-- fuente: sitio web/docs/integrations/nous-portal.md -->
#NousPortal#NousPortal[Nous Portal](https://portal.nousresearch.com) es el portal de suscripción unificado de Nous Research y **la forma recomendada de ejecutar Hermes Agent**. Un inicio de sesión de OAuth reemplaza el acto de malabarismo entre cuentas separadas, claves API y relaciones de facturación en cada laboratorio modelo, API de búsqueda, generador de imágenes y proveedor de navegador que de otro modo necesitaría conectar manualmente.Si solo tienes tiempo para configurar una cosa, configura esto. El camino más rápido:```golpecito
configuración de hermes --portal
```Ese único comando ejecuta Portal OAuth, le permite elegir un modelo Nous, establece Nous como su proveedor de inferencia en `config.yaml` y activa Tool Gateway. Estás listo para "charlar con Hermes" inmediatamente después.¿Aún no tienes una suscripción? [portal.nousresearch.com/manage-subscription](https://portal.nousresearch.com/manage-subscription): regístrese, luego regrese y ejecute el comando anterior.## ¿Qué hay en la suscripción?### Más de 300 modelos fronterizos, un billeteEl Portal ofrece un catálogo seleccionado de modelos agentes de todo el ecosistema, facturados contra su suscripción Nous en lugar de un saldo de crédito por laboratorio.| Familia | Modelos |
|--------|--------|
| **Claude antrópico** | Opus 4.7, Opus 4.6, Soneto 4.6, Haiku 4.5 |
| **OpenAI** | GPT-5.5, GPT-5.5 Pro, GPT-5.4 Mini, GPT-5.4 Nano, GPT-5.3 Códice |
| **Google Géminis** | Vista previa de Gemini 3 Pro, Vista previa de Gemini 3 Flash, Vista previa de Gemini 3.1 Pro, Vista previa de Gemini 3.1 Flash Lite |
| **Búsqueda profunda** | DeepSeek V4 Pro |
| **Qwen** | Qwen3.7-Max, Qwen3.6-35B-A3B |
| **Kimi / Disparo lunar** | Kimi K2.6 |
| **GLM / Zhipu** | GLM-5.1 |
| **MiniMax** | MiniMax M2.7 |
| **xAI** | Grok 4.3 |
| **NVIDIA** | Nemotrón-3 Súper 120B-A12B |
| **Tencent** | Vista previa de Hunyuan 3 |
| **Xiaomi** | MiMo V2.5 Pro |
| **PasoDivertido** | Paso 3.5 Destello |
| **Hermes** | Hermes-4-70B, Hermes-4-405B (chat, ver [nota a continuación](#a-note-on-hermes-4)) |
| **+ todo lo demás** | Más de 280 modelos adicionales: toda la frontera agente |El enrutamiento se realiza internamente a través de OpenRouter, por lo que la disponibilidad del modelo y el comportamiento de conmutación por error coinciden con lo que obtendría con una clave de OpenRouter; en su lugar, se factura contra su suscripción a Nous. Cambie entre Claude Sonnet 4.6 para código y Gemini 3 Pro para un contexto prolongado con `/model` a mitad de sesión: sin nuevas credenciales, sin recargas, sin errores sorpresa de saldo cero.### La puerta de enlace de Nous ToolLa misma suscripción desbloquea [Tool Gateway](/user-guide/features/tool-gateway), que enruta las llamadas a herramientas del Agente Hermes a través de la infraestructura administrada por Nous. Cinco backends, un inicio de sesión:| Herramienta | Socio | Qué hace |
|------|---------|--------------|
| **Búsqueda y extracción web** | Arrastre de fuego | Búsqueda de nivel de agente y extracción de página completa. Sin clave API de Firecrawl, sin límite de velocidad de cuidado de niños. |
| **Generación de imágenes** | Falta | Nueve modelos bajo un punto final: FLUX 2 Klein 9B, FLUX 2 Pro, Z-Image Turbo, Nano Banana Pro (Gemini 3 Pro Image), GPT Image 1.5, GPT Image 2, Ideogram V3, Recraft V4 Pro, Qwen Image. |
| **Texto a voz** | OpenAI TTS | TTS de alta calidad sin una clave OpenAI separada. Habilita [modo de voz](/user-guide/features/voice-mode) en todas las plataformas de mensajería. |
| **Automatización del navegador en la nube** | Uso del navegador | Sesiones de Chromium sin cabeza para `browser_navigate`, `browser_click`, `browser_type`, `browser_vision`. No se necesita una cuenta de Browserbase. |
| **Zona de pruebas de terminal en la nube** | modales | Sandboxes de terminal sin servidor para ejecución de código (complemento opcional). |Sin la puerta de enlace, conectar cada uno de ellos significa una cuenta Firecrawl, una cuenta FAL, una cuenta de uso del navegador, una clave OpenAI y una cuenta Modal: cinco registros separados, cinco paneles de control separados, cinco flujos de recarga separados. Con la puerta de enlace, todo se dirige a través de una suscripción.También puede habilitar solo herramientas de puerta de enlace específicas (por ejemplo, búsqueda web pero no generación de imágenes); consulte [Combinación de la puerta de enlace con sus propios backends](#mezcla-de-la-puerta-de-puerta-con-sus-propios-backends) a continuación.### ChatSu cuenta del Portal también cubre [chat.nousresearch.com](https://chat.nousresearch.com): la interfaz de chat web de Nous Research con el mismo catálogo de modelos. Útil cuando está lejos de su terminal o para trabajos de conversación sin agente.### No hay credenciales en sus archivos de puntosDebido a que todo se dirige a través de una sesión de Portal autenticada por OAuth, no se acumula un archivo `.env` con una docena de claves API de larga duración. El token de actualización en `~/.hermes/auth.json` es la única credencial en el disco, y Hermes genera JWT de corta duración por solicitud; consulte [Manejo de tokens](#token-handling) a continuación.### Paridad multiplataforma[Native Windows](/user-guide/windows-native) hace que la configuración de la clave API por herramienta sea su ventaja: instalar una cuenta Firecrawl, una cuenta FAL, una cuenta de uso del navegador y una clave OpenAI de Windows es la parte de mayor fricción para obtener un agente útil. Una suscripción al Portal facilita eso: un OAuth cubre el modelo y cada herramienta de puerta de enlace, por lo que los usuarios de Windows obtienen la misma experiencia que macOS/Linux sin configurar manualmente cuatro backends.## Una nota sobre Hermes 4La propia familia **Hermes 4** de Nous Research (Hermes-4-70B, Hermes-4-405B) está disponible a través del Portal a precios con grandes descuentos. Estos son **modelos de chat de razonamiento híbrido de frontera**: fuertes en matemáticas, ciencias, seguimiento de instrucciones, adherencia a esquemas, juegos de roles y escritura extensa.Sin embargo, **no se recomienda su uso dentro de Hermes Agent**. Hermes 4 está diseñado para charlar y razonar, no para el ciclo rápido de llamadas de herramientas en el que confía el agente. Úselos para [Nous Chat](https://chat.nousresearch.com), para flujos de trabajo de investigación o a través del [proxy de suscripción](/user-guide/features/subscription-proxy) de otras herramientas, pero para el trabajo de agente, elija un modelo de agencia de frontera del catálogo:```golpecito
/model anthropic/claude-sonnet-4.6 # mejor modelo agente de propósito general
/model openai/gpt-5.5-pro # razonamiento sólido + llamada a herramientas
/model google/gemini-3-pro-preview # ventana contextual enorme
/model deepseek/deepseek-v4-pro # codificador rentable
```La propia [página de información del modelo] (https://portal.nousresearch.com/info) del Portal incluye la misma advertencia, por lo que esta no es una opinión del lado de Hermes, es la guía oficial de Nous Research.## Configuración### Instalación nueva: un comando```golpecito
configuración de hermes --portal
```Esto ejecuta la configuración completa de una sola vez:1. Abre su navegador en portal.nousresearch.com para iniciar sesión en OAuth.
2. Almacena el token de actualización en `~/.hermes/auth.json`
3. Te permite elegir un modelo Nous de la lista seleccionada (u omitirlo para conservar el actual)
4. Establece Nous como su proveedor de inferencia en `~/.hermes/config.yaml` (cuando elige un modelo)
5. Enciende Tool Gateway (web, imágenes, TTS, enrutamiento del navegador)
6. Te regresa a tu terminal listo para `hermes chat`Si aún no tiene una suscripción, regístrese primero en [portal.nousresearch.com/manage-subscription](https://portal.nousresearch.com/manage-subscription).### Instalación existente: agregue Portal junto con otros proveedoresSi ya tiene Hermes configurado con OpenRouter, Anthropic o cualquier otro proveedor y desea agregar el Portal junto con ellos:```golpecito
modelo hermes
# elige "Nous Portal" de la lista de proveedores
# se abre el navegador, inicia sesión, listo
```Sus proveedores existentes permanecen configurados. Puede alternar entre ellos con `/model` a mitad de sesión o `hermes model` entre sesiones: el Portal se convierte en uno de sus proveedores disponibles, no en el único.### Configuración sin cabeza/SSH/remotaOAuth necesita un navegador, pero la devolución de llamada en bucle se ejecuta en la máquina donde se ejecuta Hermes. Para hosts remotos, consulte [OAuth sobre SSH/Hosts remotos](/guides/oauth-over-ssh): los mismos patrones funcionan para el Portal que para cualquier otro proveedor basado en OAuth (`ssh -L` reenvío de puertos, `--manual-paste` para entornos de solo navegador como Cloud Shell/Codespaces).### Configuración del perfilSi utiliza [perfiles de Hermes](/user-guide/profiles), el token de actualización del Portal se comparte automáticamente entre todos los perfiles a través de un almacén de tokens compartido. Inicie sesión una vez en cualquier perfil y el resto lo hará automáticamente; no es necesario repetir el flujo de OAuth por perfil.## Uso del Portal en el día a día### Inspeccionando lo que está cableado```golpecito
portal hermes # iniciar sesión en Nous Portal + configurarlo (incorporación única)
Información del portal de Hermes # estado de inicio de sesión, información de suscripción, modelo + enrutamiento de la puerta de enlace
Estado del portal de Hermes # alias para `información del portal`
Herramientas del portal Hermes # catálogo detallado de Tool Gateway con enrutamiento por herramienta
portal hermes abierto # abra la página de administración de suscripciones en su navegador
````hermes portal` (sin subcomando) es el alias legible por humanos para `hermes auth add nous --type oauth`: inicia sesión, le permite elegir un modelo Nous, establece Nous como su proveedor de inferencia y ofrece la opción Tool Gateway (idéntica a `hermes setup --portal`, y el mismo flujo Nous que la configuración rápida por primera vez).`hermes portal info` le brinda una descripción general de alto nivel:```
  Nuestro Portal
  ───────────
  Autenticación: ✓ iniciado sesión
  Portal: https://portal.nousresearch.com
  Modelo: ✓ usando Nous como proveedor de inferenciasPuerta de enlace de herramientas
  ────────────
  Búsqueda y extracción web a través de Nous Portal
  Generación de imágenes a través de Nous Portal
  Texto a voz a través de Nous Portal
  Automatización del navegador a través de Nous Portal
  Terminal en la nube no configurado
```### Cambiando de modeloDentro de una sesión:```golpecito
/modelo antrópico/claude-sonnet-4.6
/modelo openai/gpt-5.5-pro
/modelo google/gemini-3-pro-vista previa
```O abre el selector:```golpecito
/modelo
# teclas de flecha, ingresa para seleccionar
```Fuera de una sesión (el asistente de configuración completo, útil al agregar un nuevo proveedor):```golpecito
modelo hermes
```### Mezclando la puerta de enlace con tus propios backendsSi ya tiene, digamos, una cuenta de Browserbase y desea seguir usándola mientras dirige la búsqueda web y la generación de imágenes a través de Nous, eso es compatible. Utilice `hermes herramientas` para seleccionar backends por herramienta:```golpecito
herramientas hermes
# → Búsqueda web → "Suscripción Nous"
# → Generación de imágenes → "Suscripción Nous"
# → Navegador → "Browserbase" (su clave existente)
# → TTS → "Suscripción Nous"
```Tool Gateway se acepta por herramienta, no por todo o nada. Los backends administrados aparecen en las "herramientas de Hermes", ya sea que haya iniciado sesión en Nous Portal o no; si elige "Suscripción Nous" antes de autenticarse, Hermes ejecuta el inicio de sesión del Portal en línea (no cambiará su proveedor de inferencia ni tocará sus otras herramientas). Consulte los [documentos de Tool Gateway](/user-guide/features/tool-gateway) para obtener la matriz de configuración completa por herramienta.### Gestión de suscripcionesAdministre su plan, vea el uso o actualice/cancele en cualquier momento:- **Web:** [portal.nousresearch.com/manage-subscription](https://portal.nousresearch.com/manage-subscription)
- **Atajo CLI:** `hermes portal open` (abre la misma página en su navegador predeterminado)## Referencia de configuraciónDespués de `hermes setup --portal`, `~/.hermes/config.yaml` se verá así:```yaml
modelo:
  proveedor: nosotros
  predeterminado: anthropic/claude-sonnet-4.6 # o cualquier modelo que hayas elegido
  URL_base: https://inference-api.nousresearch.com/v1
```La configuración de Tool Gateway se encuentra en sus respectivas secciones de herramientas:```yaml
web:
  backend: nous # rutas de búsqueda/extracción web a través de Tool Gatewaygeneración_imagen:
  proveedor: nosotrostts:
  proveedor: nosotrosnavegador:
  backend: nosotros
```El token de actualización de OAuth se almacena por separado en `~/.hermes/auth.json` (no en `config.yaml`; las credenciales y la configuración se mantienen separadas por diseño).## Manejo de tokensHermes genera un JWT de corta duración a partir del token de actualización del Portal almacenado en cada llamada de inferencia en lugar de reutilizar una clave API de larga duración. El ciclo de vida del token es completamente automático (actualizar, acuñar, reintentar en 401 transitorio) y nunca lo ve.Si el Portal invalida el token de actualización (cambio de contraseña, revocación manual, vencimiento de la sesión), el token de actualización no válido se **pone en cuarentena localmente** para que Hermes deje de reproducirlo y no vea un flujo de 401 idénticos. La siguiente llamada muestra un mensaje claro de "se requiere reautenticación". Ejecute `hermes auth add nous` para iniciar sesión nuevamente; la cuarentena desaparece en el siguiente inicio de sesión exitoso.## Solución de problemas### `hermes portal info` muestra "no iniciado sesión"No has completado el flujo de OAuth o se borró tu token de actualización. Correr:```golpecito
portal de hermes
```o utilizar `modelo hermes` y volver a seleccionar Nous Portal.### Recibí un mensaje que decía "se requiere reautenticación" a mitad de la sesiónSu token de actualización del Portal fue invalidado (cambio de contraseña, revocación manual o vencimiento de la sesión). Ejecute `hermes auth add nous` y su próxima solicitud utilizará las nuevas credenciales. Cualquier cuarentena en el token antiguo se borra automáticamente al volver a iniciar sesión correctamente.### Quiere utilizar un modelo de proveedor específico que el Portal no exponeEl Portal actúa mediante proxy a través de OpenRouter, por lo que cualquier modelo que admita OpenRouter está generalmente disponible. Si un modelo específico no aparece en `/model`, pruebe directamente el slug estilo OpenRouter:```golpecito
/modelo antrópico/claude-opus-4.6
```Si realmente falta un modelo, [abra un problema](https://github.com/NousResearch/hermes-agent/issues): mostramos el catálogo del Portal a Hermes y los espacios vacíos generalmente significan una configuración de enrutamiento que podemos actualizar.### Las facturas no aparecen en mi cuenta del PortalPrimero verifique `hermes portal info`; si muestra que está usando un proveedor diferente (`Modelo: actualmente openrouter` en lugar de `usar Nous como proveedor de inferencia`), su configuración local se ha desviado. Ejecute "hermes model", elija Nous Portal y la siguiente solicitud se dirigirá a través de su suscripción.## Ver también- **[Tool Gateway](/user-guide/features/tool-gateway)**: detalles completos sobre cada herramienta de gateway, configuración por herramienta y precios
- **[Proxy de suscripción](/user-guide/features/subscription-proxy)**: use su suscripción al Portal desde herramientas que no sean de Hermes (otros agentes, scripts, clientes de terceros)
- **[Modo de voz](/user-guide/features/voice-mode)** — Conversaciones de voz usando OpenAI TTS del Portal
- **[Proveedores de IA](/integraciones/proveedores)** — Catálogo completo de proveedores si desea comparar alternativas
- **[OAuth over SSH](/guides/oauth-over-ssh)**: inicie sesión desde hosts remotos o entornos de solo navegador
- **[Perfiles](/user-guide/profiles)** — Múltiples configuraciones de Hermes que comparten un inicio de sesión en el Portal---