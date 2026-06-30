<!-- fuente: sitio web/docs/guides/work-with-skills.md -->
# Trabajar con habilidades# Trabajar con habilidadesLas habilidades son documentos de conocimiento bajo demanda que le enseñan a Hermes cómo manejar tareas específicas, desde generar arte ASCII hasta administrar relaciones públicas de GitHub. Esta guía le guiará en su uso diario.Para obtener la referencia técnica completa, consulte [Sistema de habilidades](/user-guide/features/skills).---## Encontrar habilidadesCada instalación de Hermes viene con habilidades integradas. Vea lo que está disponible:```golpecito
# En cualquier sesión de chat:
/habilidades# O desde la CLI:
lista de habilidades de hermes
```Esto muestra una lista compacta con nombres y descripciones:```
ascii-art Genera arte ASCII usando pyfiglet, cowsay, boxes...
arxiv Busque y recupere artículos académicos de arXiv...
github-pr-workflow Ciclo de vida completo de relaciones públicas: crear ramas, confirmar...
plan Modo plan: inspecciona el contexto, escribe una rebaja...
excalidraw Crea diagramas de estilo dibujados a mano usando Excalidraw...
```### Buscando una habilidad```golpecito
# Buscar por palabra clave
/ ventana acoplable de búsqueda de habilidades
/búsqueda de habilidades música
```### El centro de habilidadesLas habilidades opcionales oficiales (habilidades más pesadas o especializadas que no están activas de forma predeterminada) están disponibles a través del Hub:```golpecito
# Explorar habilidades opcionales oficiales
/habilidades navegar# Buscar en el centro
/ búsqueda de habilidades blockchain
```---## Usando una habilidadCada habilidad instalada es automáticamente un comando de barra diagonal. Simplemente escriba su nombre:```golpecito
# Carga una habilidad y dale una tarea.
/ascii-art Haz un banner que diga "HOLA MUNDO"
/plan Diseñar una API REST para una aplicación de tareas pendientes
/github-pr-workflow Crear un PR para la refactorización de autenticación# Solo el nombre de la habilidad (sin tarea) lo carga y te permite describir lo que necesitas
/excalidraw
```También puedes activar habilidades a través de una conversación natural: pídele a Hermes que use una habilidad específica y la cargará a través de la herramienta `skill_view`.### Divulgación progresivaLas habilidades utilizan un patrón de carga eficiente en tokens. El agente no carga todo a la vez:1. **`skills_list()`** — lista compacta de todas las habilidades (~3k tokens). Cargado al inicio de la sesión.
2. **`skill_view(nombre)`** — contenido completo de SKILL.md para una habilidad. Se carga cuando el agente decide que necesita esa habilidad.
3. **`skill_view(name, file_path)`**: un archivo de referencia específico dentro de la habilidad. Sólo se carga si es necesario.Esto significa que las habilidades no cuestan fichas hasta que realmente se usan.---## Instalación desde el concentradorLas habilidades opcionales oficiales se envían con Hermes pero no están activas de forma predeterminada. Instálelos explícitamente:```golpecito
# Instalar una habilidad opcional oficial
Habilidades de Hermes instalar oficial/investigación/arxiv# Instalar desde el centro en una sesión de chat
/instalación oficial de habilidades/creativo/composición-y-ai-music# Instale un SKILL.md de un solo archivo directamente desde cualquier URL HTTP(S)
Instalación de habilidades de Hermes https://sharethis.chat/SKILL.md
/instalación de habilidades https://example.com/SKILL.md --nombre mi-habilidad
```Qué pasa:
1. El directorio de habilidades se copia en `~/.hermes/skills/`
2. Aparece en la salida `skills_list`
3. Está disponible como un comando de barra diagonal.:::consejo
Las habilidades instaladas entran en vigor en nuevas sesiones. Si desea que esté disponible en la sesión actual, use `/reset` para comenzar de nuevo, o agregue `--now` para invalidar el caché de aviso inmediatamente (cuesta más tokens en el siguiente turno).
:::### Verificando la instalación```golpecito
# Comprueba que está ahí
lista de habilidades de hermes | grep arxiv# O en el chat
/búsqueda de habilidades arxiv
```---## Habilidades proporcionadas por complementosLos complementos pueden agrupar sus propias habilidades usando nombres con espacios de nombres (`plugin:skill`). Esto evita colisiones de nombres con habilidades integradas.```golpecito
# Cargar una habilidad de complemento por su nombre calificado
Skill_view ("superpoderes: planes de escritura")# La habilidad incorporada con el mismo nombre base no se ve afectada
Skill_view("planes-de-escritura")
```Las habilidades del complemento **no** aparecen en el indicador del sistema y no aparecen en `skills_list`. Son opcionales: cárguelos explícitamente cuando sepa que un complemento proporciona uno. Cuando se carga, el agente ve un banner que enumera las habilidades de los hermanos del mismo complemento.Para saber cómo enviar habilidades en su propio complemento, consulte [Crear un complemento de Hermes → Combinar habilidades](/guides/build-a-hermes-plugin#bundle-skills).---## Configuración de ajustes de habilidadesAlgunas habilidades declaran la configuración que necesitan en su frontmatter:```yaml
metadatos:
  hermes:
    configuración:
      - clave: tenor.api_key
        descripción: "Clave API de Tenor para búsqueda de GIF"
        mensaje: "Ingrese su clave API de Tenor"
        URL: "https://developers.google.com/tenor/guides/quickstart"
```Cuando se carga por primera vez una habilidad con configuración, Hermes le solicita los valores. Están almacenados en `config.yaml` en `skills.config.*`.Administre la configuración de habilidades desde la CLI:```golpecito
# Configuración interactiva para una habilidad específica
configuración de habilidades de hermes búsqueda de gifs# Ver todas las configuraciones de habilidades
espectáculo de configuración de hermes | grep '^habilidades\.config'
```---## Creando tu propia habilidadLas habilidades son solo archivos de rebajas con contenido frontal YAML. Crear uno lleva menos de cinco minutos.### 1. Crear el directorio```golpecito
mkdir -p ~/.hermes/skills/mi-categoría/mi-habilidad
```### 2. Escribe SKILL.md```título de reducción="~/.hermes/skills/my-category/my-skill/SKILL.md"
---
nombre: mi-habilidad
descripción: Breve descripción de lo que hace esta habilidad
versión: 1.0.0
metadatos:
  hermes:
    etiquetas: [mi-etiqueta, automatización]
    categoría: mi-categoría
---# Mi habilidad## Cuándo utilizar
Utilice esta habilidad cuando el usuario pregunte sobre [tema específico] o necesite [tarea específica].## Procedimiento
1. Primero, verifique si [requisito previo] está disponible
2. Ejecute el `comando --with-flags`
3. Analizar el resultado y presentar los resultados.## trampas
- Fallo común: [descripción]. Solución: [solución]
- Cuidado con [caso límite]## Verificación
Ejecute `check-command` para confirmar que el resultado es correcto.
```### 3. Agregar archivos de referencia (opcional)Las habilidades pueden incluir archivos de soporte que el agente carga según demanda:```
mi-habilidad/
├── SKILL.md # Documento principal de habilidades
├── referencias/
│ ├── api-docs.md # Referencia de API que el agente puede consultar
│ └── ejemplos.md # Ejemplo de entradas/salidas
├── plantillas/
│ └── config.yaml # Archivos de plantilla que el agente puede usar
└── guiones/
    └── setup.sh # Scripts que el agente puede ejecutar
```Haga referencia a estos en su SKILL.md:```rebaja
Para obtener detalles de la API, cargue la referencia: `skill_view("my-skill", "references/api-docs.md")`
```### 4. PruébaloComienza una nueva sesión y prueba tu habilidad:```golpecito
hermes chat -q "/my-skill ayúdame con la cosa"
```La habilidad aparece automáticamente, no es necesario registrarse. Colóquelo en `~/.hermes/skills/` y estará activo.:::información
El agente también puede crear y actualizar habilidades usando `skill_manage`. Después de resolver un problema complejo, Hermes puede ofrecer guardar el enfoque como una habilidad para la próxima vez.
:::---## Gestión de habilidades por plataformaControle qué habilidades están disponibles en qué plataformas:```golpecito
habilidades de hermes
```Esto abre una TUI interactiva donde puedes habilitar o deshabilitar habilidades por plataforma (CLI, Telegram, Discord, etc.). Útil cuando desea que ciertas habilidades solo estén disponibles en contextos específicos; por ejemplo, mantener las habilidades de desarrollo fuera de Telegram.---## Habilidades vs MemoriaAmbos son persistentes entre sesiones, pero tienen diferentes propósitos:| | Habilidades | Memoria |
|---|---|---|
| **Qué** | Conocimiento procedimental: cómo hacer las cosas | Conocimiento fáctico: qué son las cosas |
| **Cuando** | Cargado bajo demanda, sólo cuando sea relevante | Inyectado en cada sesión automáticamente |
| **Tamaño** | Puede ser grande (cientos de líneas) | Debe ser compacto (solo datos clave) |
| **Costo** | Cero tokens hasta que se carguen | Costo simbólico pequeño pero constante |
| **Ejemplos** | "Cómo implementar en Kubernetes" | "El usuario prefiere el modo oscuro, vive en PST" |
| **Quién crea** | Usted, el agente, o instalado desde Hub | El agente, a partir de conversaciones |**Regla general:** Si lo pones en un documento de referencia, es una habilidad. Si lo pusieras en una nota adhesiva, es memoria.---## Consejos**Mantenga las habilidades enfocadas.** Una habilidad que intente cubrir "todo DevOps" será demasiado larga y vaga. Una habilidad que cubre "implementar una aplicación Python en Fly.io" es lo suficientemente específica como para ser realmente útil.**Deje que el agente cree habilidades.** Después de una tarea compleja de varios pasos, Hermes a menudo ofrecerá guardar el enfoque como una habilidad. Diga sí: estas habilidades creadas por agentes capturan el flujo de trabajo exacto, incluidos los obstáculos que se descubrieron en el camino.**Utilice categorías.** Organice las habilidades en subdirectorios (`~/.hermes/skills/devops/`, `~/.hermes/skills/research/`, etc.). Esto mantiene la lista manejable y ayuda al agente a encontrar habilidades relevantes más rápidamente.**Actualiza las habilidades cuando se vuelven obsoletas.** Si usas una habilidad y encuentras problemas que no cubre, dile a Hermes que actualice la habilidad con lo que aprendiste. Las habilidades que no se mantienen se convierten en pasivos.---*Para obtener la referencia completa de habilidades (campos preliminares, activación condicional, directorios externos y más), consulte [Sistema de habilidades](/user-guide/features/skills).*---