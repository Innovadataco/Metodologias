<!-- fuente: sitio web/docs/guides/use-soul-with-hermes.md -->
# Utilice SOUL.md con Hermes# Utilice SOUL.md con Hermes`SOUL.md` es la **identidad principal** para su instancia de Hermes. Es lo primero que aparece en el mensaje del sistema: define quién es el agente, cómo habla y qué evita.Si desea que Hermes se sienta como el mismo asistente cada vez que habla con él, o si desea reemplazar la personalidad de Hermes por la suya propia, este es el archivo que debe utilizar.## ¿Para qué sirve SOUL.md?Utilice `SOUL.md` para:
- tono
- personalidad
- estilo de comunicación
- qué tan directo o cálido debe ser Hermes
- lo que Hermes debería evitar estilísticamente
- cómo debería relacionarse Hermes con la incertidumbre, el desacuerdo y la ambigüedadEn resumen:
- `SOUL.md` trata sobre quién es Hermes y cómo habla Hermes.## Para qué no sirve SOUL.mdNo lo utilices para:
- convenciones de codificación específicas del repositorio
- rutas de archivos
- comandos
- puertos de servicio
- notas de arquitectura
- instrucciones de flujo de trabajo del proyectoEsos pertenecen a `AGENTS.md`.Una buena regla:
- si se aplica en todas partes, póngalo en `SOUL.md`
- si solo pertenece a un proyecto, póngalo en `AGENTS.md`## Donde viveHermes ahora usa solo el archivo SOUL global para la instancia actual:```texto
~/.hermes/ALMA.md
```Si ejecuta Hermes con un directorio de inicio personalizado, se convierte en:```texto
$HERMES_HOME/ALMA.md
```## Comportamiento de primera ejecuciónHermes genera automáticamente un iniciador `SOUL.md` si aún no existe uno.Eso significa que la mayoría de los usuarios ahora comienzan con un archivo real que pueden leer y editar inmediatamente.Importante:
- si ya tienes un `SOUL.md`, Hermes no lo sobrescribe
- si el archivo existe pero está vacío, Hermes no agrega nada al mensaje## Cómo lo usa HermesCuando Hermes inicia una sesión, lee `SOUL.md` de `HERMES_HOME`, lo escanea en busca de patrones de inyección de mensajes, lo trunca si es necesario y lo usa como **identidad de agente**: ranura número 1 en el mensaje del sistema. Esto significa que SOUL.md reemplaza completamente el texto de identidad predeterminado incorporado.Si SOUL.md falta, está vacío o no se puede cargar, Hermes recurre a una identidad predeterminada incorporada.No se agrega ningún lenguaje contenedor alrededor del archivo. El contenido en sí importa: escriba la forma en que desea que su agente piense y hable.## Una buena primera ediciónSi no hace nada más, abra el archivo y cambie solo unas pocas líneas para que se sienta como usted.Por ejemplo:```rebaja
Eres directo, tranquilo y técnicamente preciso.
Prefiere la sustancia al teatro de cortesía.
Retroceda claramente cuando una idea sea débil.
Mantenga las respuestas compactas a menos que sea útil obtener detalles más profundos.
```Sólo eso puede cambiar notablemente cómo se siente Hermes.## Estilos de ejemplo### 1. Ingeniero pragmático```rebaja
Eres un ingeniero senior pragmático.
Le importa más la corrección y la realidad operativa que sonar impresionante.## Estilo
- Sea directo
- Sea conciso a menos que la complejidad requiera profundidad.
- Decir cuando algo es una mala idea.
- Prefieren compensaciones prácticas a abstracciones idealizadas.## evitar
- adulación
- lenguaje exagerado
- Explicar demasiado cosas obvias.
```### 2. Socio de investigación```rebaja
Eres un colaborador de investigación reflexivo.
Eres curioso, honesto acerca de la incertidumbre y te entusiasman las ideas inusuales.## Estilo
- Explorar posibilidades sin pretender certeza
- Distinguir especulación de evidencia.
- Hacer preguntas aclaratorias cuando el espacio de la idea no esté especificado.
- Prefiere la profundidad conceptual a la integridad superficial
```### 3. Profesor/explicador```rebaja
Eres un profesor técnico paciente.
Te importa la comprensión, no el desempeño.## Estilo
- Explica claramente
- Utilice ejemplos cuando le ayuden.
- No asumir conocimientos previos a menos que el usuario lo indique
- Construir desde la intuición hasta los detalles.
```### 4. Crítico duro```rebaja
Eres un crítico riguroso.
Eres justo, pero no suavizas las críticas importantes.## Estilo
- Señalar directamente los supuestos débiles.
- Priorizar la corrección sobre la armonía.
- Sea explícito sobre los riesgos y las compensaciones.
- Prefiero una claridad contundente a una diplomacia vaga.
```## ¿Qué hace que un ALMA.md sea fuerte?Un `SOUL.md` fuerte es:
- estable
- ampliamente aplicable
- específico en voz
- no sobrecargado con instrucciones temporalesUn `SOUL.md` débil es:
- lleno de detalles del proyecto
- contradictorio
- Tratar de microgestionar cada forma de respuesta.
- relleno en su mayoría genérico como "ser útil" y "ser claro"Hermes ya intenta ser útil y claro. `SOUL.md` debería agregar personalidad y estilo reales, no reafirmar valores predeterminados obvios.## Estructura sugeridaNo necesitas títulos, pero ayudan.Una estructura simple que funciona bien:```rebaja
# Identidad
Quién es Hermes.# Estilo
Cómo debería sonar Hermes.# evitar
Lo que Hermes no debería hacer.# Valores predeterminados
Cómo debería comportarse Hermes cuando aparece la ambigüedad.
```## ALMA.md vs /personalidadEstos son complementarios.Utilice `SOUL.md` para su línea de base duradera.
Utilice `/personality` para cambios de modo temporales.Ejemplos:
- tu ALMA predeterminada es pragmática y directa
- luego, para una sesión usas `/profesor de personalidad`
- luego vuelves a cambiar sin cambiar tu archivo de voz base## ALMA.md vs AGENTES.mdEste es el error más común.### Pon esto en SOUL.md
- “Sé directo”.
- “Evite el lenguaje publicitario”.
- “Prefiere respuestas cortas a menos que la profundidad ayude”.
- “Retroceder cuando el usuario se equivoque”.### Pon esto en AGENTES.md
- "Utilice pytest, no unittest".
- "El frontend vive en `frontend/`".
- “Nunca edites las migraciones directamente”.
- "La API se ejecuta en el puerto 8000".## Cómo editarlo```golpecito
nano ~/.hermes/SOUL.md
```o```golpecito
vim ~/.hermes/SOUL.md
```Luego reinicie Hermes o inicie una nueva sesión.## Un flujo de trabajo práctico1. Comience con el archivo predeterminado inicializado.
2. Recorta todo lo que no se parezca a la voz que deseas
3. Agregue de 4 a 8 líneas que definan claramente el tono y los valores predeterminados.
4. Habla con Hermes un rato.
5. Ajústelo en función de lo que todavía se siente malEse enfoque iterativo funciona mejor que intentar diseñar la personalidad perfecta de una sola vez.## Solución de problemas### Edité SOUL.md pero Hermes todavía suena igualComprobar:
- editaste `~/.hermes/SOUL.md` o `$HERMES_HOME/SOUL.md`
- no es un repositorio local `SOUL.md`
- el archivo no está vacío
- su sesión se reinició después de la edición
- una superposición `/personalidad` no domina el resultado### Hermes está ignorando partes de mi ALMA.mdPosibles causas:
- las instrucciones de mayor prioridad lo anulan
- el archivo incluye orientación contradictoria
- el archivo es demasiado largo y se truncó
- parte del texto se parece al contenido de inyección rápida y el escáner puede bloquearlo o modificarlo### Mi SOUL.md se volvió demasiado específico del proyectoMueva las instrucciones del proyecto a `AGENTS.md` y mantenga `SOUL.md` enfocado en la identidad y el estilo.## Documentos relacionados- [Personalidad y ALMA.md](/user-guide/features/personality)
- [Archivos de contexto](/user-guide/features/context-files)
- [Configuración](/guía-usuario/configuración)
- [Consejos y mejores prácticas](/guides/tips)---