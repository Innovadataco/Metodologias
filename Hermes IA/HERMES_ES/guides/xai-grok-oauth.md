<!-- fuente: sitio web/docs/guides/xai-grok-oauth.md -->
# xAI Grok OAuth (SuperGrok / X Premium+)# xAI Grok OAuth (SuperGrok / X Premium+)Hermes Agent admite xAI Grok a través de un flujo de inicio de sesión OAuth basado en navegador contra [accounts.x.ai](https://accounts.x.ai), utilizando una **suscripción SuperGrok** ([grok.com](https://x.ai/grok)) o una **suscripción X Premium+** (cuenta X vinculada). No se requiere `XAI_API_KEY`: inicie sesión una vez y Hermes actualizará automáticamente su sesión en segundo plano.Cuando inicia sesión con una cuenta X que tiene Premium+, xAI vincula automáticamente el estado de la suscripción a su sesión de xAI, por lo que el flujo de OAuth funciona igual que para los suscriptores directos de SuperGrok.El transporte reutiliza el adaptador `codex_responses` (xAI expone un punto final estilo Responses), por lo que el razonamiento, la llamada a herramientas, la transmisión y el almacenamiento en caché de avisos funcionan sin ningún cambio en el adaptador.El mismo token de portador de OAuth también se reutiliza en todas las superficies directas a xAI en Hermes (TTS, generación de imágenes, generación de videos y transcripción), por lo que un único inicio de sesión cubre los cuatro.## Descripción general| Artículo | Valor |
|------|-------|
| Identificación del proveedor | `xai-oauth` |
| Nombre para mostrar | xAI Grok OAuth (SuperGrok / X Premium+) |
| Tipo de autenticación | Navegador OAuth 2.0 PKCE (devolución de llamada de bucle invertido) |
| Transporte | API de respuestas xAI (`codex_responses`) |
| Modelo predeterminado | `grok-build-0.1` |
| Punto final | `https://api.x.ai/v1` |
| Servidor de autenticación | `https://accounts.x.ai` |
| Requiere var env | No (`XAI_API_KEY` **no** se usa para este proveedor) |
| Suscripción | [SuperGrok](https://x.ai/grok) o [X Premium+](https://x.com/i/premium_sign_up) — consulte la nota a continuación |## Requisitos previos- Pitón 3.9+
- Agente Hermes instalado
- Una suscripción activa **SuperGrok** en tu cuenta xAI, **o** una suscripción **X Premium+** en la cuenta X con la que inicias sesión (xAI vincula la suscripción automáticamente)
- Un navegador disponible en la máquina local (o use `--no-browser` para sesiones remotas):::advertencia xAI puede restringir el acceso a la API de OAuth por nivel
El backend de xAI aplica su propia lista de permitidos en la superficie de la API de OAuth y se ha observado que rechaza los suscriptores estándar de SuperGrok con `HTTP 403` (consulte el problema [#26847](https://github.com/NousResearch/hermes-agent/issues/26847)) aunque la suscripción dentro de la aplicación esté activa. Si el inicio de sesión de OAuth se realiza correctamente en el navegador pero la inferencia devuelve 403, configure `XAI_API_KEY` y cambie a la ruta de la clave API (`proveedor: xai`); esa superficie no está sujeta a la misma restricción hoy.
:::## Inicio rápido```golpecito
# Inicie el proveedor y el selector de modelo.
modelo hermes
# → Seleccione "xAI Grok OAuth (SuperGrok / X Premium+)" de la lista de proveedores
# → Hermes abre su navegador en cuentas.x.ai
# → Aprobar acceso en el navegador
# → Elige un modelo (grok-build-0.1 está en la parte superior)
# → Empezar a chatearhermes
```Después del primer inicio de sesión, las credenciales se almacenan en `~/.hermes/auth.json` y se actualizan automáticamente antes de que caduquen.## Iniciar sesión manualmentePuede activar un inicio de sesión sin pasar por el selector de modelo:```golpecito
autenticación hermes agregar xai-oauth
```### Sesiones remotas/sin cabezaEn servidores, contenedores o sesiones SSH donde no hay un navegador disponible, Hermes detecta el entorno remoto e imprime la URL de autorización en lugar de abrir un navegador.**Importante:** el detector de bucle invertido todavía se ejecuta en la máquina remota en `127.0.0.1:56121`. La redirección xAI debe llegar a *ese* oyente, por lo que no se podrá abrir la URL en su computadora portátil ("No se pudo establecer la conexión. No pudimos comunicarnos con su aplicación") a menos que reenvíe el puerto:```golpecito
# En una terminal separada en su máquina local:
ssh -N -L 56121:127.0.0.1:56121 usuario@host-remoto# Luego en tu sesión SSH en la máquina remota:
autenticación hermes agregar xai-oauth --sin navegador
# Abra la URL de autorización impresa en su navegador local.
```A través de un cuadro de salto/bastión: agregue `-J jump-user@jump-host`.Consulte [OAuth sobre SSH / Hosts remotos] (./oauth-over-ssh.md) para conocer el paso a paso completo, incluidas las cadenas de ProxyJump, mosh/tmux y los errores de ControlMaster.### Controles remotos solo para navegador (Cloud Shell, Codespaces, EC2 Instance Connect)Si no tiene un cliente SSH normal (por ejemplo, está ejecutando Hermes dentro de GCP Cloud Shell, GitHub Codespaces, AWS EC2 Instance Connect, Gitpod u otra consola basada en navegador), la receta `ssh -L` anterior no está disponible. Utilice `--manual-paste` en su lugar: Hermes omite el detector de bucle invertido y le permite pegar la URL de devolución de llamada fallida directamente desde su navegador:```golpecito
hermes auth agregar xai-oauth --manual-pegar
# O mediante el selector de modelo:
modelo hermes --pasta-manual
```Consulte [OAuth sobre SSH/Hosts remotos](./oauth-over-ssh.md#browser-only-remote-cloud-shell--codespaces--ec2-instance-connect) para obtener el tutorial completo. Corrección de regresión para [#26923](https://github.com/NousResearch/hermes-agent/issues/26923).Si la página de consentimiento muestra el código de autorización directamente en la página (el comportamiento actual de xAI en consolas basadas en navegador) en lugar de redirigir a su `127.0.0.1:56121/callback`, pegue **solo el valor del código simple** en el mensaje `URL de devolución de llamada:`: Hermes acepta la URL completa, un fragmento de consulta `?code=...&state=...` simple o un código simple indistintamente.## Cómo funciona el inicio de sesión1. Hermes abre su navegador en `accounts.x.ai`.
2. Inicia sesión (o confirma su sesión existente) y aprueba el acceso.
3. xAI redirige nuevamente a Hermes y los tokens se guardan en `~/.hermes/auth.json`.
4. A partir de ese momento, Hermes actualiza el token de acceso en segundo plano: usted permanecerá conectado hasta que "hermes auth cierre sesión en xai-oauth" o revoque el acceso desde la configuración de su cuenta xAI.## Comprobando el estado de inicio de sesión```golpecito
doctor hermes
```La sección `◆ Proveedores de autenticación` mostrará el estado actual de cada proveedor, incluido `xai-oauth`.## Cambio de modelos```golpecito
modelo hermes
# → Seleccione "xAI Grok OAuth (SuperGrok / X Premium+)"
# → Elija de la lista de modelos (grok-build-0.1 está fijado en la parte superior)
```O configure el modelo directamente:```golpecito
conjunto de configuración de hermes model.default grok-build-0.1
conjunto de configuración de hermes model.provider xai-oauth
```## Referencia de configuraciónDespués de iniciar sesión, `~/.hermes/config.yaml` contendrá:```yaml
modelo:
  predeterminado: grok-build-0.1
  proveedor: xai-oauth
  URL_base: https://api.x.ai/v1
```### Alias ​​de proveedoresTodo lo siguiente se resuelve en `xai-oauth`:```golpecito
hermes --proveedor xai-oauth # canónico
hermes --proveedor grok-oauth # alias
hermes --provider x-ai-oauth # alias
hermes --proveedor xai-grok-oauth # alias
```## Herramientas directas a xAI (TTS/Imagen/Video/Transcripción/X Search)Una vez que haya iniciado sesión a través de OAuth, cada herramienta directa a xAI reutiliza el mismo token de portador automáticamente; **no hay una configuración separada** a menos que prefiera usar una clave API.Para elegir un backend para cada herramienta:```golpecito
herramientas hermes
# → Texto a voz → "xAI TTS"
# → Generación de imágenes → "xAI Grok Imagine (imagen)"
# → Generación de vídeo → "xAI Grok Imagine"
# → Búsqueda X (Twitter) → "xAI Grok OAuth (SuperGrok / X Premium+)"
```Si los tokens de OAuth ya están almacenados, el selector los confirma y omite la solicitud de credenciales. Si no se configuran ni OAuth ni `XAI_API_KEY`, el selector ofrece un menú de 3 opciones: iniciar sesión en OAuth, pegar la clave API u omitir.:::nota La generación de vídeo está desactivada de forma predeterminada
El conjunto de herramientas `video_gen` está deshabilitado de forma predeterminada. Habilítelo en `hermes tools` → `🎬 Video Generation` (presione el espacio) antes de que el agente pueda llamar a `video_generate`. De lo contrario, el agente puede recurrir a la habilidad ComfyUI incluida, que también está etiquetada para la generación de videos.
::::::nota La búsqueda X se habilita automáticamente cuando las credenciales xAI están presentes
El conjunto de herramientas `x_search` se habilita automáticamente cada vez que se configuran las credenciales xAI (un token SuperGrok / X Premium+ OAuth o `XAI_API_KEY`). Deshabilite explícitamente a través de `hermes tools` → `🐦 X (Twitter) Search` (presione espacio) si no desea esto. La herramienta se dirige a través de la API de respuestas `x_search` incorporada de xAI: funciona **ya sea** con su inicio de sesión OAuth de SuperGrok / X Premium+ o con una `XAI_API_KEY` paga, y prefiere OAuth cuando ambos están configurados (usa su cuota de suscripción en lugar del gasto de API). El esquema de la herramienta está oculto del modelo cuando no se configuran credenciales xAI, independientemente de si el conjunto de herramientas está habilitado.
:::### Modelos| Herramienta | Modelo | Notas |
|------|-------|-------|
| Charla | `grok-build-0.1` | Por defecto; seleccionado automáticamente cuando inicia sesión a través de OAuth |
| Charla | `grok-4.3` | Predeterminado anterior |
| Charla | `grok-4.20-0309-razonamiento` | Variante de razonamiento |
| Charla | `grok-4.20-0309-sin razonamiento` | Variante sin razonamiento |
| Charla | `grok-4.20-multi-agente-0309` | Variante multiagente |
| Imagen | `grok-imagina-imagen` | Por defecto; ~5–10 s |
| Imagen | `grok-imagina-calidad-de-imagen` | Mayor fidelidad; ~10–20 s |
| Vídeo | `grok-imagina-video` | Texto a vídeo |
| Vídeo | `grok-imagine-video-1.5-vista previa` | Imagen a vídeo; alias fechado `grok-imagine-video-1.5-2026-05-30` |
| ETT | (voz predeterminada) | Punto final xAI `/v1/tts` |El catálogo de chat se deriva en vivo del caché `models.dev` en el disco; Las nuevas versiones de xAI aparecen automáticamente una vez que se actualiza el caché. `grok-build-0.1` siempre está fijado en la parte superior de la lista.## Variables de entorno| Variables | Efecto |
|----------|--------|
| `XAI_BASE_URL` | Anule el punto final predeterminado `https://api.x.ai/v1` (rara vez es necesario). |Para seleccionar xAI como proveedor activo, configure `model.provider: xai-oauth` en `config.yaml` (use `hermes setup` para el flujo guiado) o pase `--provider xai-oauth` para una única invocación.## Solución de problemas### El token expiró: no se vuelve a iniciar sesión automáticamenteHermes actualiza el token antes de cada sesión y nuevamente de manera reactiva en un 401. Si la actualización falla con `invalid_grant` (el token de actualización fue revocado o la cuenta fue rotada), Hermes muestra un mensaje de nueva autenticación escrito en lugar de fallar.Cuando el error de actualización es terminal (HTTP 4xx, `invalid_grant`, concesión revocada, etc.), Hermes marca el token de actualización como inactivo y lo pone en cuarentena localmente; las llamadas posteriores omiten el intento de actualización condenado al fracaso en lugar de reproducir el mismo 401 una y otra vez. El agente muestra un único mensaje que dice "se requiere reautenticación" y permanece apartado hasta que usted vuelve a iniciar sesión.**Solución:** ejecute `hermes auth add xai-oauth` nuevamente para iniciar un nuevo inicio de sesión. La cuarentena termina con el próximo intercambio exitoso.### Autorización agotadaEl oyente de loopback tiene una ventana de vencimiento finita (el valor predeterminado es 180 s). Si no aprueba el inicio de sesión a tiempo, Hermes genera un error de tiempo de espera.**Solución:** vuelva a ejecutar `hermes auth add xai-oauth` (o `hermes model`). El flujo comienza de nuevo.### El estado no coincide (posible CSRF)Hermes detectó que el valor de "estado" devuelto por el servidor de autorización no coincide con lo que envió.**Solución:** vuelva a ejecutar el inicio de sesión. Si persiste, busque un proxy o redireccionamiento que esté modificando la respuesta de OAuth.### Iniciar sesión desde un servidor remotoEn sesiones SSH o de contenedor, Hermes imprime la URL de autorización en lugar de abrir un navegador. El detector de devolución de llamada de bucle invertido aún vincula `127.0.0.1:56121` en el host remoto; el navegador de su computadora portátil no puede acceder a él sin un reenvío local SSH:```golpecito
# Máquina local, terminal independiente:
ssh -N -L 56121:127.0.0.1:56121 usuario@host-remoto# Máquina remota:
autenticación hermes agregar xai-oauth --sin navegador
```Tutorial completo (cuadros de salto, mosh/tmux, conflictos de puertos): [OAuth sobre SSH/Hosts remotos](./oauth-over-ssh.md).### HTTP 403 después de un inicio de sesión exitoso (nivel/derecho)OAuth se completa en el navegador, los tokens se guardan, pero la inferencia o la actualización del token devuelve `HTTP 403` con un mensaje similar a *"La persona que llama no tiene permiso para ejecutar la operación especificada"*.Esto **no** es un problema de token obsoleto: volver a ejecutar el "modelo Hermes" no lo cambiará. Se ha observado que el backend de xAI restringe el acceso a la API de OAuth a niveles específicos de SuperGrok a pesar de que la suscripción en la aplicación está activa (problema [#26847](https://github.com/NousResearch/hermes-agent/issues/26847)).**Solución:** configure `XAI_API_KEY` y cambie a la ruta de la clave API:```golpecito
exportar XAI_API_KEY=xai-...
conjunto de configuración de hermes model.provider xai
```O actualice su suscripción en [x.ai/grok](https://x.ai/grok) si se requiere la ruta OAuth.### Error "No se encontraron credenciales xAI" en tiempo de ejecuciónEl almacén de autenticación no tiene ninguna entrada `xai-oauth` y no está configurada ninguna `XAI_API_KEY`. Aún no ha iniciado sesión o el archivo de credenciales se eliminó.**Solución:** ejecute `hermes model` y elija el proveedor xAI Grok OAuth, o ejecute `hermes auth add xai-oauth`.## Cerrar sesiónPara eliminar todas las credenciales de xAI Grok OAuth almacenadas:```golpecito
hermes auth cerrar sesión xai-oauth
```Esto borra tanto la entrada única de OAuth en `auth.json` como cualquier fila del grupo de credenciales para `xai-oauth`. Utilice `hermes auth remove xai-oauth <index|id|label>` si solo desea eliminar una única entrada del grupo (ejecute `hermes auth list xai-oauth` para verlas).## Ver también- [OAuth sobre SSH/Hosts remotos](./oauth-over-ssh.md): lectura obligatoria si Hermes está en una máquina diferente a la de su navegador
- [Referencia de proveedores de IA](../integrations/providers.md)
- [Variables de entorno](../reference/environment-variables.md)
- [Configuración](../user-guide/configuration.md)
- [Voz y TTS](../user-guide/features/tts.md)---