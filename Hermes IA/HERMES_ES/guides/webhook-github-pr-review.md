<!-- fuente: sitio web/docs/guides/webhook-github-pr-review.md -->
# Comentarios de relaciones públicas de GitHub automatizados con webhooks# Comentarios de relaciones públicas de GitHub automatizados con webhooksEsta guía lo guía a través de la conexión de Hermes Agent a GitHub para que obtenga automáticamente la diferencia de una solicitud de extracción, analice los cambios de código y publique un comentario, activado por un evento de webhook sin solicitud manual.Cuando se abre o actualiza un PR, GitHub envía una POST de webhook a su instancia de Hermes. Hermes ejecuta el agente con un mensaje que le indica que recupere la diferencia a través de la CLI `gh` y la respuesta se envía al hilo de relaciones públicas.:::consejo ¿Quiere una configuración más sencilla sin un punto final público?
Si no tiene una URL pública o simplemente desea comenzar rápidamente, consulte [Crear un agente de revisión de relaciones públicas de GitHub] (./github-pr-review-agent.md): utiliza trabajos cron para sondear relaciones públicas según un cronograma, funciona detrás de NAT y firewalls.
::::::info Documentos de referencia
Para obtener la referencia completa de la plataforma webhook (todas las opciones de configuración, tipos de entrega, suscripciones dinámicas, modelo de seguridad), consulte [Webhooks](/user-guide/messaging/webhooks).
::::::advertencia Riesgo de inyección inmediata
Las cargas útiles de webhook contienen datos controlados por el atacante: los títulos de relaciones públicas, los mensajes de confirmación y las descripciones pueden contener instrucciones maliciosas. Cuando el punto final de su webhook esté expuesto a Internet, ejecute la puerta de enlace en un entorno de espacio aislado (Docker, backend SSH). Consulte la [sección de seguridad](#notas-de-seguridad) a continuación.
:::---## Requisitos previos- Agente Hermes instalado y ejecutándose (`hermes gateway`)
- [`gh` CLI](https://cli.github.com/) instalado y autenticado en el host de la puerta de enlace (`gh auth login`)
- Una URL de acceso público para su instancia de Hermes (consulte [Pruebas locales con ngrok](#local-testing-with-ngrok) si se ejecuta localmente)
- Acceso de administrador al repositorio de GitHub (requerido para administrar webhooks)---## Paso 1: habilite la plataforma de webhookAgregue lo siguiente a su `~/.hermes/config.yaml`:```yaml
plataformas:
  gancho web:
    habilitado: verdadero
    adicional:
      puerto: 8644 # predeterminado; cambiar si otro servicio ocupa este puerto
      rate_limit: 30 # solicitudes máximas por minuto por ruta (no es un límite global)rutas:
        github-pr-revisión:
          secreto: "tu-webhook-secreto-aquí" # debe coincidir exactamente con el secreto del webhook de GitHub
          eventos:
            - pull_request# Se le indica al agente que obtenga la diferencia real antes de revisarla.
          # {number} y {repository.full_name} se resuelven desde la carga útil de GitHub.
          mensaje: |
            Se recibió un evento de solicitud de extracción (acción: {acción}).PR #{número}: {pull_request.title}
            Autor: {pull_request.user.login}
            Rama: {pull_request.head.ref} → {pull_request.base.ref}
            Descripción: {pull_request.body}
            URL: {pull_request.html_url}Si la acción está "cerrada" o "etiquetada", deténgase aquí y no publique ningún comentario.De lo contrario:
            1. Ejecute: gh pr diff {número} --repo {repositorio.nombre_completo}
            2. Revise los cambios del código para verificar que sean correctos, tengan problemas de seguridad y sean claros.
            3. Escriba un comentario de revisión conciso y práctico y publíquelo.entregar: github_comment
          entregar_extra:
            repositorio: "{repositorio.full_name}"
            pr_number: "{número}"
```**Campos clave:**| Campo | Descripción |
|---|---|
| `secreto` (nivel de ruta) | Secreto HMAC para esta ruta. Vuelve a `extra.secret` global si se omite. |
| `eventos` | Lista de valores de encabezado `X-GitHub-Event` para aceptar. Lista vacía = aceptar todo. |
| `rápido` | Plantilla; `{field}` y `{nested.field}` se resuelven desde la carga útil de GitHub. |
| `entregar` | Publicaciones de `github_comment` a través de `gh pr comment`. `log` simplemente escribe en el registro de la puerta de enlace. |
| `deliver_extra.repo` | Resuelve, por ej. `org/repo` de la carga útil. |
| `entregar_extra.pr_número` | Resuelve el número PR de la carga útil. |:::nota La carga útil no contiene código
La carga útil del webhook de GitHub incluye metadatos de relaciones públicas (título, descripción, nombres de sucursales, URL), pero **no la diferencia**. El mensaje anterior indica al agente que ejecute "gh pr diff" para recuperar los cambios reales. La herramienta `terminal` está incluida en el conjunto de herramientas predeterminado `hermes-webhook`, por lo que no se necesita configuración adicional.
:::---## Paso 2: iniciar la puerta de enlace```golpecito
puerta de enlace de hermes
```Deberías ver:```
[webhook] Escuchando en 0.0.0.0:8644 - rutas: github-pr-review
```Verifique que esté funcionando:```golpecito
rizo http://localhost:8644/health
# {"estado": "ok", "plataforma": "webhook"}
```---## Paso 3: registrar el webhook en GitHub1. Vaya a su repositorio → **Configuración** → **Webhooks** → **Agregar webhook**
2. Complete:
   - **URL de carga útil:** `https://your-public-url.example.com/webhooks/github-pr-review`
   - **Tipo de contenido:** `aplicación/json`
   - **Secreto:** el mismo valor que estableciste para "secreto" en la configuración de ruta
   - **¿Qué eventos?** → Seleccione eventos individuales → marque **Solicitudes de extracción**
3. Haga clic en **Agregar webhook**GitHub enviará inmediatamente un evento "ping" para confirmar la conexión. Se ignora de forma segura (`ping` no está en su lista de `eventos`) y devuelve `{"status": "ignored", "event": "ping"}`. Solo se registra en el nivel DEBUG, por lo que no aparecerá en la consola en el nivel de registro predeterminado.---## Paso 4: abrir un PR de pruebaCree una sucursal, impulse un cambio y abra un PR. En un plazo de 30 a 90 segundos (según el tamaño y el modelo del PR), Hermes debería publicar un comentario de revisión.Para seguir el progreso del agente en tiempo real:```golpecito
cola -f "${HERMES_HOME:-$HOME/.hermes}/logs/gateway.log"
```---## Pruebas locales con ngrokSi Hermes se está ejecutando en su computadora portátil, use [ngrok](https://ngrok.com/) para exponerlo:```golpecito
ngrok http 8644
```Copie la URL `https://...ngrok-free.app` y úsela como su URL de carga útil de GitHub. En el nivel gratuito de ngrok, la URL cambia cada vez que se reinicia ngrok: actualice su webhook de GitHub en cada sesión. Las cuentas pagas de ngrok obtienen un dominio estático.Puede probar una ruta estática directamente con `curl`; no se necesita una cuenta de GitHub ni relaciones públicas reales.:::tip Utilice `deliver: log` cuando realice pruebas localmente
Cambie `deliver: github_comment` a `deliver: log` en su configuración mientras realiza la prueba. De lo contrario, el agente intentará publicar un comentario en el repositorio falso `org/repo#99` en la carga útil de prueba, lo cual fallará. Vuelva a `deliver: github_comment` una vez que esté satisfecho con el resultado del mensaje.
:::```golpecito
SECRET="tu-webhook-secreto-aquí"
BODY='{"action":"opened","number":99,"pull_request":{"title":"Prueba PR","body":"Agrega un feature.","user":{"login":"testuser"},"head":{"ref":"feat/x"},"base":{"ref":"main"},"html_url":"https://github.com/org/repo/pull/99"},"repository":{"full_name":"org/repo"}}'
SIG=$(printf '%s' "$BODY" | openssl dgst -sha256 -hmac "$SECRET" -hex | awk '{print "sha256="$2}')curl -s -X POST http://localhost:8644/webhooks/github-pr-review \
  -H "Tipo de contenido: aplicación/json" \
  -H "Evento X-GitHub: pull_request" \
  -H "Firma-X-Hub-256: $SIG" \
  -d "$CUERPO"
# Esperado: {"status":"aceptado","route":"github-pr-review","event":"pull_request","delivery_id":"..."}
```Luego observe cómo ejecuta el agente:
```golpecito
cola -f "${HERMES_HOME:-$HOME/.hermes}/logs/gateway.log"
```:::nota
`hermes webhook test <nombre>` solo funciona para **suscripciones dinámicas** creadas con `hermes webhook subscribe`. No lee rutas de `config.yaml`.
:::---## Filtrado por acciones específicasGitHub envía eventos `pull_request` para muchas acciones: `abierto`, `sincronizar`, `reabierto`, `cerrado`, `etiquetado`, etc. La lista de `eventos` filtra solo por el valor del encabezado `X-GitHub-Event`; no puede filtrar por subtipo de acción en el nivel de enrutamiento.El mensaje del Paso 1 ya maneja esto al indicarle al agente que se detenga temprano para eventos "cerrados" y "etiquetados".:::advertencia El agente aún se ejecuta y consume tokens
La instrucción "detener aquí" impide una revisión significativa, pero el agente aún se ejecuta hasta su finalización para cada evento "pull_request", independientemente de la acción. Los webhooks de GitHub solo pueden filtrar por tipo de evento (`pull_request`, `push`, `issues`, etc.), no por subtipo de acción (`opened`, `closed`, `labeled`). No existe un filtro de nivel de enrutamiento para subacciones. Para repositorios de gran volumen, acepte este costo o filtre en sentido ascendente con un flujo de trabajo de GitHub Actions que llame a la URL de su webhook de forma condicional.
:::> No existe Jinja2 ni sintaxis de plantilla condicional. `{field}` y `{nested.field}` son las únicas sustituciones admitidas. Todo lo demás se transmite palabra por palabra al agente.---## Usar una habilidad para lograr un estilo de revisión consistenteCargue una [habilidad de Hermes](/user-guide/features/skills) para darle al agente una persona de revisión consistente. Agregue `skills` a su ruta dentro de `platforms.webhook.extra.routes` en `config.yaml`:```yaml
plataformas:
  gancho web:
    habilitado: verdadero
    adicional:
      rutas:
        github-pr-revisión:
          secreto: "tu-webhook-secreto-aquí"
          eventos: [pull_request]
          mensaje: |
            Se recibió un evento de solicitud de extracción (acción: {acción}).
            PR #{número}: {pull_request.title} por {pull_request.user.login}
            URL: {pull_request.html_url}Si la acción está "cerrada" o "etiquetada", deténgase aquí y no publique ningún comentario.De lo contrario:
            1. Ejecute: gh pr diff {número} --repo {repositorio.nombre_completo}
            2. Revise la diferencia utilizando sus pautas de revisión.
            3. Escriba un comentario de revisión conciso y práctico y publíquelo.
          habilidades:
            - revisión
          entregar: github_comment
          entregar_extra:
            repositorio: "{repositorio.full_name}"
            pr_number: "{número}"
```> **Nota:** Solo se carga la primera habilidad de la lista que se encuentra. Hermes no acumula múltiples habilidades: las entradas posteriores se ignoran.---## Enviar respuestas a Slack o Discord en su lugarReemplace los campos `deliver` y `deliver_extra` dentro de su ruta con su plataforma de destino:```yaml
# Dentro de plataformas.webhook.extra.routes.<nombre-ruta>:# holgura
entregar: flojo
entregar_extra:
  chat_id: "C0123456789" # ID del canal de Slack (omita usar el canal de inicio configurado)# discordia
entregar: discordia
entregar_extra:
  chat_id: "987654321012345678" # ID del canal de Discord (omitir el uso del canal de inicio)
```La plataforma de destino también debe estar habilitada y conectada en la puerta de enlace. Si se omite `chat_id`, la respuesta se envía al canal de inicio configurado de esa plataforma.Valores válidos de `deliver`: `log` · `github_comment` · `telegram` · `discord` · `slack` · `signal` · `sms`---## Soporte de GitLabEl mismo adaptador funciona con GitLab. GitLab usa `X-Gitlab-Token` para la autenticación (coincidencia de cadena simple, no HMAC); Hermes maneja ambos automáticamente.Para el filtrado de eventos, GitLab establece `X-GitLab-Event` en valores como `Merge Request Hook`, `Push Hook`, `Pipeline Hook`. Utilice el valor exacto del encabezado en "eventos":```yaml
eventos:
  - Gancho de solicitud de fusión
```Los campos de carga útil de GitLab difieren de los de GitHub, p. `{object_attributes.title}` para el título de MR y `{object_attributes.iid}` para el número de MR. La forma más sencilla de descubrir la estructura completa de la carga útil es el botón **Probar** de GitLab en la configuración de su webhook, combinado con el registro **Entregas recientes**. Alternativamente, omita "prompt" de su configuración de ruta: Hermes pasará la carga útil completa como formato JSON directamente al agente, y la respuesta del agente (visible en el registro de la puerta de enlace con "deliver: log") describirá su estructura.---## Notas de seguridad- **Nunca use `INSECURE_NO_AUTH`** en producción; deshabilita por completo la validación de firmas. Es sólo para el desarrollo local.
- **Rote su secreto de webhook** periódicamente y actualícelo tanto en GitHub (configuración de webhook) como en su `config.yaml`.
- **El límite de velocidad** es de 30 solicitudes/min por ruta de forma predeterminada (configurable a través de `extra.rate_limit`). Superarlo devuelve "429".
- **Las entregas duplicadas** (reintentos de webhook) se deduplican mediante un caché de idempotencia de 1 hora. La clave de caché es `X-GitHub-Delivery` si está presente, luego `X-Request-ID` y luego una marca de tiempo de milisegundos. Cuando no se establece ningún encabezado de ID de entrega, los reintentos **no** se deduplican.
- **Inyección rápida:** los títulos de relaciones públicas, las descripciones y los mensajes de confirmación están controlados por el atacante. Los RP maliciosos podrían intentar manipular las acciones del agente. Ejecute la puerta de enlace en un entorno aislado (Docker, VM) cuando esté expuesto a la Internet pública.---## Solución de problemas| Síntoma | Consultar |
|---|---|
| `401 Firma no válida` | El secreto en config.yaml no coincide con el secreto del webhook de GitHub |
| `404 Ruta desconocida` | El nombre de la ruta en la URL no coincide con la clave en `rutas:` |
| `429 Límite de tasa excedido` | Se excedieron 30 solicitudes/min por ruta: común cuando se vuelven a entregar eventos de prueba desde la interfaz de usuario de GitHub; espere un minuto o suba `extra.rate_limit` |
| Ningún comentario publicado | `gh` no está instalado, no está en PATH o no está autenticado (`gh auth login`) |
| El agente se ejecuta pero no hay comentarios | Verifique el registro de la puerta de enlace: si la salida del agente estaba vacía o simplemente "SALTAR", aún se intenta la entrega |
| Puerto ya en uso | Cambie `extra.port` en config.yaml |
| El agente se ejecuta pero revisa solo la descripción del PR | El mensaje no incluye la instrucción `gh pr diff`; la diferencia no está en la carga útil del webhook |
| No puedo ver el evento de ping | Los eventos ignorados devuelven `{"status":"ignored","event":"ping"}` solo en el nivel de registro DEBUG: verifique el registro de entrega de GitHub (repositorio → Configuración → Webhooks → su webhook → Entregas recientes) |**La pestaña Entregas recientes de GitHub** (repositorio → Configuración → Webhooks → su webhook) muestra los encabezados de solicitud exactos, la carga útil, el estado HTTP y el cuerpo de respuesta para cada entrega. Es la forma más rápida de diagnosticar fallas sin tocar los registros de su servidor.---## Referencia de configuración completa```yaml
plataformas:
  gancho web:
    habilitado: verdadero
    adicional:
      host: "0.0.0.0" # dirección de enlace (predeterminado: 0.0.0.0)
      puerto: 8644 # puerto de escucha (predeterminado: 8644)
      secreto: "" # secreto de reserva global opcional
      rate_limit: 30 # solicitudes por minuto por ruta
      max_body_bytes: 1048576 # límite de tamaño de carga útil en bytes (predeterminado: 1 MB)rutas:
        <nombre-ruta>:
          secreto: "requerido por ruta"
          eventos: [] # [] = aceptar todo; de lo contrario, enumere los valores de X-GitHub-Event
          mensaje: "" # {field} / {nested.field} resuelto desde la carga útil
          habilidades: [] # se carga la primera habilidad coincidente (solo una)
          entregar: "registro" # registro | comentario_github | telegrama | discordia | flojo | señal | mensaje de texto
          entregar_extra: {} # repositorio + pr_number para github_comment; chat_id para otros
```---## ¿Qué sigue?- **[Revisiones de relaciones públicas basadas en Cron](./github-pr-review-agent.md)**: encuesta para relaciones públicas según un cronograma, no se necesita un punto final público
- **[Referencia de webhook](/user-guide/messaging/webhooks)**: referencia de configuración completa para la plataforma webhook
- **[Crear un complemento](/guides/build-a-hermes-plugin)** — lógica de revisión del paquete en un complemento que se puede compartir
- **[Perfiles](/user-guide/profiles)**: ejecuta un perfil de revisor dedicado con su propia memoria y configuración---