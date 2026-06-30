<!-- fuente: sitio web/docs/guides/use-voice-mode-with-hermes.md -->
# Usar el modo de voz con Hermes# Usar el modo de voz con HermesEsta guía es el complemento práctico de la [referencia de la función Modo de voz](/user-guide/features/voice-mode).Si la página de funciones explica qué puede hacer el modo de voz, esta guía muestra cómo usarlo bien.:::consejo
[Nous Portal](/integrations/nous-portal) combina LLM y TTS a través de un OAuth: el modo de voz funciona de un extremo a otro sin credenciales adicionales.
:::## ¿Para qué sirve el modo de voz?El modo de voz es especialmente útil cuando:
- desea un flujo de trabajo CLI manos libres
- quieres respuestas habladas en Telegram o Discord
- quieres que Hermes esté sentado en un canal de voz de Discord para una conversación en vivo
- desea capturar ideas rápidamente, depurarlas o ir y venir mientras camina en lugar de escribir## Elija la configuración de su modo de vozEn realidad, hay tres experiencias de voz diferentes en Hermes.| Modo | Lo mejor para | Plataforma |
|---|---|---|
| Bucle de micrófono interactivo | Uso personal de manos libres mientras codifica o investiga | CLI |
| Respuestas de voz en el chat | Respuestas habladas junto con mensajes normales | Telegrama, Discordia |
| Bot de canal de voz en vivo | Conversación grupal o personal en vivo en un VC | Canales de voz de Discord |Un buen camino es:
1. hacer que el texto funcione primero
2. habilitar las respuestas de voz en segundo lugar
3. Pase al último canal de voz de Discord si desea la experiencia completa## Paso 1: asegúrese de que Hermes normal funcione primeroAntes de tocar el modo de voz, verifique que:
- Hermes comienza
- su proveedor está configurado
- el agente puede responder mensajes de texto normalmente```golpecito
hermes
```Pregunta algo sencillo:```texto
¿Qué herramientas tienes disponibles?
```Si eso aún no es sólido, arregle primero el modo texto.## Paso 2: instale los extras adecuados### Micrófono CLI + reproducción```golpecito
cd ~/.hermes/hermes-agent && uv pip install -e ".[voz]"
```### Plataformas de mensajería```golpecito
cd ~/.hermes/hermes-agent && uv pip install -e ".[mensajería]"
```### TTS premium de ElevenLabs```golpecito
cd ~/.hermes/hermes-agent && uv pip install -e ".[tts-premium]"
```### NeuTTS local (opcional)```golpecito
python -m pip install -U neutts[todos]
```### Todo```golpecito
cd ~/.hermes/hermes-agent && uv pip install -e ".[todos]"
```## Paso 3: instalar las dependencias del sistema### MacOS```golpecito
instalar cerveza portaudio ffmpeg opus
instalar cerveza hablando-ng
```### Ubuntu/Debian```golpecito
sudo apto instalar portaudio19-dev ffmpeg libopus0
sudo apt instalar espeak-ng
```Por qué son importantes:
- `portaudio` → entrada/reproducción de micrófono para modo de voz CLI
- `ffmpeg` → conversión de audio para TTS y entrega de mensajes
- `opus` → Compatibilidad con códec de voz Discord
- `espeak-ng` → backend de fonemizador para NeuTTS## Paso 4: elija proveedores de STT y TTSHermes admite pilas de voz tanto locales como en la nube.### Configuración más fácil y económicaUtilice STT local y Edge TTS gratuito:
- Proveedor STT: `local`
- Proveedor TTS: `borde`Este suele ser el mejor lugar para empezar.### Ejemplo de archivo de entornoAgregar a `~/.hermes/.env`:```golpecito
# Opciones de Cloud STT (local no necesita clave)
GROQ_API_KEY=***
VOICE_TOOLS_OPENAI_KEY=***# TTS premium (opcional)
ELEVENLABS_API_KEY=***
```### Recomendaciones del proveedor#### Voz a texto- `local` → mejor valor predeterminado para privacidad y uso sin costo
- `groq` → transcripción en la nube muy rápida
- `openai` → respaldo bien pagado#### Texto a voz- `edge` → gratis y lo suficientemente bueno para la mayoría de los usuarios
- `neutts` → TTS local/en el dispositivo gratuito
- `elevenlabs` → mejor calidad
- `openai` → buen término medio
- `mistral` → multilingüe, Opus nativo### Si usas `hermes setup`Si elige NeuTTS en el asistente de configuración, Hermes comprueba si `neutts` ya está instalado. Si falta, el asistente le indica que NeuTTS necesita el paquete de Python `neutts` y el paquete del sistema `espeak-ng`, le ofrece instalarlos, instala `espeak-ng` con el administrador de paquetes de su plataforma y luego ejecuta:```golpecito
python -m pip install -U neutts[todos]
```Si omite esa instalación o falla, el asistente recurre a Edge TTS.## Paso 5: configuración recomendada```yaml
voz:
  clave_registro: "ctrl+b"
  max_recording_segundos: 120
  auto_tts: falso
  beep_enabled: verdadero
  umbral_silencio: 200
  duración_silencio: 3.0tt:
  proveedor: "local"
  locales:
    modelo: "base"tts:
  proveedor: "borde"
  borde:
    voz: "en-US-AriaNeural"
```Este es un buen valor predeterminado conservador para la mayoría de la gente.Si en su lugar desea TTS local, cambie el bloque `tts` a:```yaml
tts:
  proveedor: "neutts"
  neutros:
    ref_audio: ''
    texto_ref: ''
    modelo: neuphonic/neutts-air-q4-gguf
    dispositivo: CPU
```## Caso de uso 1: modo de voz CLI## EnciéndeloIniciar Hermes:```golpecito
hermes
```Dentro de la CLI:```texto
/voz encendida
```### Flujo de grabaciónClave predeterminada:
- `Ctrl+B`Flujo de trabajo:
1. presione `Ctrl+B`
2. hablar
3. Espere a que la detección de silencio deje de grabar automáticamente.
4. Hermes transcribe y responde
5. si TTS está activado, dice la respuesta
6. el bucle puede reiniciarse automáticamente para uso continuo### Comandos útiles```texto
/voz
/voz encendida
/voz apagada
/voz tts
/estado de voz
```### Buenos flujos de trabajo CLI#### Depuración directaDecir:```texto
Sigo recibiendo un error de permiso de la ventana acoplable. Ayúdame a depurarlo.
```Luego continúa con manos libres:
- "Leer el último error nuevamente"
- "Explique la causa raíz en términos más simples"
- "Ahora dame la solución exacta"#### Investigación / lluvia de ideasGenial para:
- caminar mientras piensa
- dictar ideas a medio formar
- pedirle a Hermes que estructure tus pensamientos en tiempo real#### Accesibilidad/sesiones de mecanografía bajaSi escribir le resulta incómodo, el modo de voz es una de las formas más rápidas de mantenerse al tanto del ciclo completo de Hermes.## Ajuste del comportamiento de la CLI### Umbral de silencioSi Hermes arranca o se detiene de forma demasiado agresiva, ajuste:```yaml
voz:
  umbral_silencio: 250
```Umbral más alto = menos sensible.### Duración del silencioSi haces muchas pausas entre oraciones, aumenta:```yaml
voz:
  duración_silencio: 4.0
```### Clave de grabaciónSi `Ctrl+B` entra en conflicto con su terminal o hábitos de tmux:```yaml
voz:
  record_key: "ctrl+espacio"
```## Caso de uso 2: respuestas de voz en Telegram o DiscordEste modo es más sencillo que los canales de voz completos.Hermes sigue siendo un robot de chat normal, pero puede responder con voz.### Iniciar la puerta de enlace```golpecito
puerta de enlace de hermes
```### Activar respuestas de vozDentro de Telegram o Discord:```texto
/voz encendida
```o```texto
/voz tts
```### Modos| Modo | Significado |
|---|---|
| `apagado` | sólo texto |
| `solo_voz` | hablar sólo cuando el usuario envió voz |
| `todos` | habla cada respuesta |### Cuándo usar qué modo- `/voice on` si desea respuestas habladas solo para mensajes originados por voz
- `/voice tts` si quieres un asistente hablado todo el tiempo### Buenos flujos de trabajo de mensajería#### Asistente de Telegram en tu teléfonoÚselo cuando:
- estás lejos de tu máquina
- desea enviar notas de voz y obtener respuestas habladas rápidas
- desea que Hermes funcione como un asistente de operaciones o de investigación portátil#### Discord DM con salida habladaÚtil cuando desea una interacción privada sin un comportamiento de mención del canal del servidor.## Caso de uso 3: canales de voz de DiscordEste es el modo más avanzado.Hermes se une a Discord VC, escucha el discurso del usuario, lo transcribe, ejecuta la canalización normal del agente y transmite las respuestas al canal.## Permisos de discordia requeridosAdemás de la configuración normal del robot de texto, asegúrese de que el bot tenga:
- Conectar
- hablar
- Preferiblemente usar actividad de vozHabilite también intenciones privilegiadas en el Portal de desarrollador:
- Intención de presencia
- Intención de los miembros del servidor
- Intención del contenido del mensaje## Únete y veteEn un canal de texto de Discord donde el bot está presente:```texto
/unión por voz
/voz dejar
/estado de voz
```### ¿Qué pasa cuando te unes?- los usuarios hablan en el VC
- Hermes detecta los límites del habla.
- las transcripciones se publican en el canal de texto asociado
- Hermes responde en texto y audio.
- el canal de texto es aquel donde se emitió `/voice join`### Mejores prácticas para el uso de Discord VC- mantén apretado `DISCORD_ALLOWED_USERS`
- utilice un canal de prueba/bot dedicado al principio
- Verifique que STT y TTS funcionen en el modo de voz de chat de texto normal antes de probar el modo VC## Recomendaciones de calidad de voz### Configuración de la mejor calidad- STT: local `large-v3` o Groq `whisper-large-v3`
- TTS: ElevenLabs### Configuración de mejor velocidad/conveniencia- STT: `base` local o Groq
- TTS: borde### La mejor configuración sin costo- STT: local
- TTS: borde## Modos de falla comunes### "No se encontró ningún dispositivo de audio"Instale `portaudio`.### "El robot se une pero no oye nada"Comprobar:
- tu ID de usuario de Discord está en `DISCORD_ALLOWED_USERS`
- no estás silenciado
- los intentos privilegiados están habilitados
- el bot tiene permisos de Conectar/Hablar### "Transcribe pero no habla"Comprobar:
- Configuración del proveedor TTS
- Clave API/cuota para ElevenLabs u OpenAI
- Instalación de `ffmpeg` para rutas de conversión de Edge### "Whisper genera basura"Prueba:
- ambiente más tranquilo
- mayor `silence_threshold`
- diferente proveedor/modelo de STT
- expresiones más cortas y claras### "Funciona en DM pero no en canales de servidor"Esa es la política que a menudo se menciona.De forma predeterminada, el bot necesita una `@mención` en los canales de texto del servidor Discord a menos que se configure lo contrario.## Configuración sugerida para la primera semanaSi desea el camino más corto hacia el éxito:1. hacer que Hermes funcione
2. instale `hermes-agent[voz]`
3. use el modo de voz CLI con STT + Edge TTS local
4. luego habilite `/voz activada` en Telegram o Discord
5. solo después de eso, prueba el modo Discord VCEsa progresión mantiene pequeña la superficie de depuración.## Dónde leer a continuación- [Referencia de la función Modo de voz](/user-guide/features/voice-mode)
- [Puerta de enlace de mensajería](/guía-usuario/mensajería)
- [Configuración de Discord](/user-guide/messaging/discord)
- [Configuración de Telegram](/user-guide/messaging/telegram)
- [Configuración](/guía-usuario/configuración)---