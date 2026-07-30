# Metodología Operativa — Fábrica de Software IDC

| Campo | Detalle |
|---|---|
| **Documento** | Metodología Operativa de la Fábrica de Software |
| **Versión** | v2.1 (2026-07-29) |
| **Fecha** | 2026-07-21 |
| **Origen** | ACTA-001 (`Gestion-de-proyectos/00-GOBERNANZA/ACTAS/`) |
| **Base metodológica** | GitHub Spec Kit + modelo híbrido de dos agentes IDC |
| **Aplica a** | Todo desarrollo de producto de IDC |

---

## 1. Propósito y alcance

Definir **cómo se construye software en IDC** con agentes de IA: quién hace qué, en qué orden, cómo se traspasa el trabajo y cuándo algo se considera terminado. Es el manual operativo obligatorio para todo proyecto de la fábrica.

**Principio rector:** el criterio arquitectónico es humano-dirigido y centralizado (ZEUS); la ejecución es delegada y verificable (ODIN); la decisión final es del negocio (Jelkin).

## 2. Actores y responsabilidades

| Actor | Es | Hace | NO hace |
|---|---|---|---|
| **Jelkin** (CEO/PMO) | Dirección y negocio | Genera la idea, prioriza, aprueba, valida en pruebas | Escribir specs ni código |
| **ZEUS** (Claude Code/Opus) | Arquitecto y Líder de Fábrica | Constitución, brief de diseño, revisa specs/planes, audita código | **No escribe código de producto** |
| **ODIN** (agente implementador) | Desarrollador | Redacta spec/plan, implementa, prueba, despliega, commitea | **No decide arquitectura** |

> **Sobre la encarnación de los agentes (v1.9, 2026-07-27).** El rol es fijo; la herramienta que lo
> encarna no. En **PROTECCIÓN INFANTIL**, ODIN es **Kimi, ejecutado con Kimi Code** (antes era
> claude-code en VS Code) y ZEUS sigue siendo **Claude Code**. Consecuencias, aplicables a
> cualquier proyecto donde ODIN y ZEUS no compartan modelo:
> 1. El handoff **no puede asumir comportamientos** del modelo del arquitecto: prompt autosuficiente y por referencia (§6 R1/R4).
> 2. La **compuerta §4 deja de ser opcional** — ODIN se detiene tras `spec`+`plan`.
> 3. La **auditoría post-commit de ZEUS gana peso**: es el único filtro sobre código escrito por otro criterio (§10).
>
> Cada proyecto registra su encarnación vigente en sus **Decisiones** (en PI: `D-31`).

## 3. Las 5 Reglas de Oro (siempre, sin excepción)

1. **Aplicar Spec Kit** en todo trabajo.
2. **Subir los cambios** al repo de GitHub.
3. **Siempre hacer pruebas.**
4. **Siempre validar que se despliegue.**
5. **Siempre documentar** con Spec Kit.

> Estas 5 reglas son la Definición de Terminado mínima (ver §8). Ninguna se salta "por rapidez".

## 4. Matriz de propiedad de fases (modelo híbrido)

Spec Kit vanilla asume un solo agente para todo el ciclo. IDC lo divide en dos cerebros:

| Fase Spec Kit | Dueño | Cómo se ejecuta |
|---|---|---|
| **Constitución** | ZEUS (una vez) | La redacta ZEUS. ADN arquitectónico, no delegable. |
| **Idea → Brief de diseño** | ZEUS | Brief compacto: objetivo, alcance, restricciones, decisiones clave. |
| **`/speckit.specify`** | ODIN redacta | A partir del brief de ZEUS, en el repo. |
| **`/speckit.plan`** | ODIN redacta | Plan técnico a partir del spec aprobado. |
| **Revisión spec + plan** | ZEUS (compuerta) | ZEUS lee del repo → aprueba o devuelve con correcciones. |
| **`/speckit.tasks` + `/speckit.implement`** | ODIN | Implementa, prueba y despliega. |
| **Revisión final + validación** | ZEUS + Jelkin | Auditoría de ZEUS + prueba de Jelkin → ACTA-VALIDACION. |

**Regla de compuerta:** ODIN no avanza a la siguiente fase sin la aprobación de ZEUS en la anterior.

## 5. Flujo operativo (el bucle)

```
1. Jelkin da la idea            → chat corto y enfocado con ZEUS
2. ZEUS piensa y propone        → se acuerda en mesa
3. ZEUS genera el BRIEF/PROMPT  → compacto, por referencia (§6)
4. Jelkin pasa el prompt a ODIN
5. ODIN redacta spec/plan       → commitea
6. Jelkin: "ZEUS, revisa X"     → ZEUS lee el repo (NO se pega salida)
7. ZEUS aprueba o devuelve
8. ODIN implementa, prueba, despliega, commitea
9. ZEUS audita el diff + Jelkin prueba → ACTA-VALIDACION
10. Feature cerrado             → se cierra el chat
```

## 6. Protocolo de handoff (ZEUS → ODIN)

**Regla 1 — Cáscara delgada sobre Spec Kit, no por copia.** El instructivo **no repite lo que Spec Kit ya da** (tarea medible → **FR**; criterio de éxito → **SC**; aceptación → **Acceptance Scenarios**; definición de terminado → **T0xx + Checkpoints** de `tasks.md`; dudas → **`[NEEDS CLARIFICATION]` + `/speckit.clarify`**; revisión cruzada → **`/speckit.analyze`**). Solo lleva lo que Spec Kit **no cruza** a una ventana limpia ni conoce del proyecto; el resto es un *puntero*. La **compuerta §4 es el punto donde para la cadena de comandos**.

**Plantilla de prompt de ZEUS para ODIN (cáscara):**
```
# <PRODUCTO>-<NNN> · <nombre-corto> — SPEC-<NNN>          ← §13 (manda el nombre)
> radicado: <PRODUCTO> · <n> · SPEC-<NNN> · rama <rama>   ← §13 primera línea obligatoria

## Contexto puente (máx 5 líneas)   ← lo único que Spec Kit NO cruza a ventana limpia
Qué se hace, por qué importa, y la decisión vinculante que lo rige. Autosuficiente.

## Cadena de comandos  = la tarea Y la compuerta
- [ ] §4 DISEÑO :  /speckit.specify → /speckit.plan   ⟶ PARA · ZEUS aprueba antes de seguir
- [ ] DIRECTO   :  /speckit.tasks → /speckit.analyze → /speckit.implement
(dudas ⇒ [NEEDS CLARIFICATION] + /speckit.clarify · ODIN no inventa)

## Leer primero
AGENTS.md · .specify/memory/constitution.md · specs/<NNN>/{spec,plan,tasks}.md · [D-xx/I-xx del PM2 + commit]

## Candados (lo que Spec Kit no sabe)
- [archivos que se LEEN, no se tocan] · [qué NO reconciliar / NO silenciar]

## Señales (ver Regla 3)
```
La auditoría de ZEUS corre contra **FR/SC + Acceptance** (`spec.md`) y **T0xx** (`tasks.md`): esos criterios **no se copian** al prompt.

**Regla 2 — ODIN sube su propio trabajo.** ODIN **commitea Y hace push** en el mismo acto: un commit sin push no cumple la Regla de Oro 2 y deja el trabajo solo en la MacStudio, sin respaldo. **ZEUS nunca hace push de código de producto** — no es suyo (§2). Si ZEUS encuentra commits sin subir, los devuelve a ODIN; no los sube.

**Regla 3 — Protocolo de señales: una línea, un verbo de acción.** El chat del CEO es un recurso escaso y ZEUS lee el repo de todos modos (§9). El ciclo se mueve con señales cortas que muestran **acción, no silencio**. Cada actor emite la suya:

```
ZEUS → repo + CEO :  <RAD>  RADICADA · pégala a ODIN
CEO  → ODIN       :  [pega la cáscara]
ODIN → CEO/ZEUS   :  <RAD>  REVISADA · arranco        ← acuse OBLIGATORIO: la entendí, voy
ODIN → CEO/ZEUS   :  <RAD>  REALIZADO · <hash> · <media línea>   [Nota: …]
CEO  → ZEUS       :  audita
ZEUS → CEO        :  <RAD>  REVISO <hash> → CUMPLE ✓ | NO CUMPLE ✗ · para Jelkin: <…>
```

**Estados:** `RADICADA → REVISADA → REALIZADO → REVISO → CUMPLE/NO CUMPLE`. El `REVISADA · arranco` es **obligatorio** (regla en `AGENTS.md`): el CEO nunca se queda sin saber si ODIN arrancó; si algo bloquea, `REVISADA · dudas: <…> → PARA`. La **"Nota"** de `REALIZADO` se reserva para lo que el diff NO muestra: una desviación, un hallazgo preexistente, un riesgo asumido. **Nada de tablas, listas de commits ni descripciones de lo hecho** — eso vive en el repo y en `tasks.md`.

**Cuándo va Nota — test binario.** Antes de escribir una Nota, pregúntate: *¿cambiaría el veredicto de ZEUS si no la lee?* **No → no hay Nota.**

| ✅ SÍ Nota (cambia el veredicto / pide validación) | ❌ NO Nota (el diff ya lo dice) |
|---|---|
| Me desvié de lo pedido | "No toqué X" / "respeté el candado" |
| Hallazgo que afecta lo auditado | Estado del working tree ajeno |
| Riesgo asumido **en este cambio** | "Cumplí los checkpoints" |
| Supuesto que tomé para desempatar | Descripción de lo que hice |

**Aplica igual a ZEUS y al CEO.** Toda respuesta —incluido el veredicto de ZEUS— es una línea salvo que **(a)** sea necesaria para el veredicto o **(b)** requiera validación del CEO. Si CUMPLE y no hay nada que validar, ZEUS responde solo `<RAD> · CUMPLE ✓` y para. Sin prosa de relleno.

**Regla 4 — Ventana nueva por frente.** Cuando la ventana de ODIN se satura (o se cierra un frente), **no se recupera el hilo: se abre una ventana limpia**. El contexto no vive en el chat sino en los archivos (§11), así que basta con un **prompt de reapertura** que le diga qué leer y en qué orden:

```
CONTEXTO: <PRODUCTO> (repo <ruta>) · SPEC-<NNN>
Ventana nueva. No tienes historial: todo tu contexto está en archivos.

LEE ANTES DE ACTUAR, EN ESTE ORDEN:
1. AGENTS.md del repo — roles, puertos intocables, ramas, aislamiento.
2. .specify/memory/constitution.md — principios rectores del producto.
3. Metodología Operativa §6, §8, §10, §13.
4. PM2 del proyecto: 01-REGISTROS-AVANCE (estado real),
   05-DECISIONES (vinculantes), 04-INCIDENCIAS, 06-ACTAS-VALIDACION.

ESTADO: <qué está Terminado, qué está congelado, rama de trabajo>
TAREA: <...>
```

Quien arranca sin leer eso repite trabajo, contradice decisiones ya tomadas o toca el producto equivocado. **Releer es más barato que rehacer.**

**Regla 5 — La revisión NO se pega.** Jelkin dice *"ZEUS, revisa el commit `<hash>` del proyecto `<id>`"*. ZEUS lee el diff directo del repo. La referencia sigue §13: producto por **nombre**, nunca por número.

## 7. DO / DON'T por actor

**ZEUS — DO:** exigir Spec Kit; revisar contra la Constitución; ser honesto y no complaciente; leer del repo; dejar puntero en memoria de decisiones clave.
**ZEUS — DON'T:** escribir código de producto; aprobar sin leer; manejar tokens/credenciales; redactar specs verbosos (eso es de ODIN).

**ODIN — DO:** redactar spec/plan desde el brief; implementar con pruebas; commits convencionales; documentar.
**ODIN — DON'T:** decidir arquitectura por su cuenta; saltar fases sin aprobación de ZEUS; dar por terminado sin ACTA-VALIDACION; subir secrets.

**Jelkin — DO:** dar ideas claras; validar en pruebas; aprobar formalmente; manejar personalmente tokens/credenciales.
**Jelkin — DON'T:** pegar tokens en el chat; pasar prompts sin acuerdo previo con ZEUS.

## 8. Definición de "Terminado" (DoD) y ACTA-VALIDACION

Un feature está **Terminado** solo si cumple las 5 reglas de oro, verificadas en una **ACTA-VALIDACION**:

**Checklist ACTA-VALIDACION (obligatorio antes de dar por Terminado):**
- [ ] Spec Kit aplicado (spec + plan + tasks documentados). *(Regla 1, 5)*
- [ ] Código subido a GitHub, en la rama que corresponda al repo (§10). *(Regla 2)*
- [ ] Pruebas escritas y pasando. *(Regla 3)*
- [ ] Despliegue validado *(Regla 4)* → **definición:** la app queda **desplegada y accesible por web** para que el CEO pueda verla y probarla (levantar la app + healthcheck OK + URL accesible). No basta con que compile.
- [ ] Revisión de arquitectura de ZEUS sobre el commit: aprobada.
- [ ] Validación funcional de Jelkin **(condicional)**: obligatoria en **funcionalidades completas** de cara al usuario; en **fixes/correcciones internas basta la auditoría de ZEUS**. Jelkin puede pedir la doble revisión cuando quiera.
- [ ] Sin secrets ni datos sensibles en el código.

> **Sin ACTA-VALIDACION no se despliega ni se da por Terminado.**

## 9. Protocolo de eficiencia de tokens — "ZEUS Ligero"

1. La verdad vive en archivos, no en el chat.
2. No se pega la salida de ODIN; ZEUS la lee del repo.
3. Un chat = un frente de trabajo; se cierra al terminar.
4. ZEUS lee dirigido, no explora repos completos.
5. Prompts por referencia, no por copia.

> Se ahorra en plomería (pegar, repetir, chats zombis), **nunca en criterio** (arquitectura y revisión).

## 10. Gobernanza técnica

- **Ramas (decisión CEO):** separadas por tipo de repo.
  - **Código** (`productos`) → **dos ramas, y solo dos**: la **rama de pruebas** `feature/001-scaffolding` (nombre heredado del scaffolding) es donde ocurre el trabajo diario de los tres frentes; **`main` es el tronco, que se mantiene al día** con el trabajo ya probado. **No se abren ramas por feature.**
    > **Aclaración del CEO (2026-07-24, v1.8):** actualizar `main` es **sincronizar el tronco**, no una ceremonia de liberación. IDC **aún no tiene ambiente productivo** — ADR_001 §5 sigue sin definirlo—, así que nada se despliega desde `main`. Cuando ese ambiente exista, se le añadirá a `main` la semántica de despliegue y esta seccion se enmienda.
  - **Documentación** (`Gestion-de-proyectos`, `Metodologias`) → **una sola rama: `main`**. Son documentos oficiales: no se ramifican ni se mantienen versiones en paralelo. Su control de versiones es el **bloque de control de cada documento** (§12), no la rama.
- **Actualización de `main`:** merge de la rama de pruebas al tronco, **solo con la rama verde** (suite, `tsc --noEmit` y build limpios). Es sincronización rutinaria: **no** exige acta propia ni ceremonia. Nunca se reescribe la historia de una rama compartida.
- **Compuerta de calidad:** el control **no es el PR**, es la **auditoría de ZEUS sobre cada commit** — ZEUS revisa el diff después de que ODIN sube.
- **Cierre:** un feature solo se da por Terminado con ACTA-VALIDACION completa.
- **Los comandos de la integración Spec Kit SE VERSIONAN** (v1.9). Cada integración instala sus comandos en su propio directorio (`.claude/`, `.clinerules/`, `.kimi-code/skills/`…). **Ese directorio no puede quedar en `.gitignore`.** Caso real que origina la regla: en PROTECCIÓN INFANTIL la integración era `cline`, con sus comandos en `.clinerules/` — ignorado desde el primer día. Los comandos **nunca existieron en el repo**, ningún agente los tuvo, y cuatro specs (098/100/101/102) se escribieron a mano y llegaron **sin `plan.md` ni `tasks.md`**. No fue indisciplina: faltaba la herramienta. Al cambiar de agente se usa `specify integration switch <clave>` (no re-inicializar: preserva la constitución) y **se commitea el directorio de comandos en el acto**.
- **Secrets:** siempre por variables de entorno, nunca en el código.
- **Código sensible:** preferir revisión con modelos locales.

> ⚠️ **Nota de arquitecto:** trabajar sobre una rama larga sin PR por feature gana velocidad pero elimina la red de seguridad de la revisión previa al merge. Por eso la **auditoría post-commit de ZEUS es innegociable** — es el único filtro antes de que un error se consolide en la rama de desarrollo.

## 11. Arquitectura de memoria (3 capas)

El contexto vive en archivos versionados, no en la cabeza de ningún agente:
1. **Verdad del proyecto** → repo con Spec Kit (spec, plan, tasks).
2. **Decisiones y actas** → repo `Gestion-de-proyectos` + Obsidian.
3. **Memoria operativa de ZEUS** → `~/.claude/.../memory/` (solo punteros cortos).

## 12. Estándar de control documental

Todo documento de gestión de IDC (repo `Gestion-de-proyectos`) cumple 4 reglas:

1. **Control por carpeta** — cada carpeta tiene un `CONTROL_<CARPETA>.md` que lista y enlaza sus documentos. El `README.md` del proyecto es el maestro que enlaza todos los `CONTROL_*`.
2. **Bloque de control por documento** — cada documento cierra con: `versión · fecha · hora · autor`.
3. **Enlaces cruzados** — cada documento enlaza a los que menciona, con **enlaces markdown relativos** (funcionan en GitHub y Obsidian; NO wikilinks).
4. **Fuente única de la verdad** — cada dato tiene un solo dueño (ej. el presupuesto vive solo en `09-PLAN-COSTOS.md`); los demás documentos **enlazan** al dueño en vez de repetir el valor.

Estructura PM2 de proyecto (formato de referencia): `00-META · 01-INICIO · 02-PLANIFICACION · 03-EJECUCION · 04-CIERRE · 05-ENTREGABLES`.

## 13. Nomenclatura: cómo se nombra el trabajo

**Problema que resuelve esta sección:** los números de carpeta **no coinciden** entre el repo de gestión y el de código. El mismo número significa dos productos distintos:

| Producto | `Gestion-de-proyectos/01-PROYECTOS/` | `productos/` |
|---|---|---|
| **Protección Infantil** | `001-2026-PROTECCION_INFANTIL` | `002-2026-PROTECCION-INFANTIL` |
| **SICOV-OTPC** | `002-2026-SICOV-OTPC` | `003-2026-SICOV-OTPC` |
| **INNOVADATACO** | `003-2026-INNOVADATACO` | `001-2026-INNOVADATACO` |

Decir *"el proyecto 003"* es ambiguo: es INNOVADATACO en gestión y SICOV en código. Dado que la regla de aislamiento entre productos es innegociable (ADR_002), esa ambigüedad es un riesgo operativo real, no una molestia de forma.

### Regla: manda el NOMBRE, nunca el número

> **Toda referencia a un trabajo se nombra `PRODUCTO · TIPO-NNN`.**

| Se dice | No se dice |
|---|---|
| `INNOVADATACO · SPEC-004` | "la spec 4", "el proyecto 003" |
| `INNOVADATACO · I-009` | "la incidencia 9" |
| `INNOVADATACO · D-036` | "la decisión 36" |
| `SICOV · SPEC-005` | "la 005" |
| `ADR_004` | (los ADR son transversales: no llevan producto) |

Productos válidos como nombre: **INNOVADATACO**, **PROTECCIÓN INFANTIL**, **SICOV**.

**Por qué no se renumeran las carpetas:** cada número aparece en rutas relativas de specs, ADR, actas, planes y archivos `CONTROL_*` de ambos repos. Renumerar rompería cientos de enlaces para ganar una coherencia que la regla del nombre ya resuelve a coste cero. Si algún día se hace, es una tarea propia con su spec.

### Prompt a ODIN: primera línea obligatoria

Todo prompt de ZEUS a ODIN abre identificando producto y trabajo sin ambigüedad:

```
CONTEXTO: INNOVADATACO (repo productos/001-2026-INNOVADATACO) · SPEC-004
```

Nombre **y** ruta del repo. Con eso ningún agente puede equivocarse de producto.

## 14. Control de versiones de este documento

| Versión | Fecha | Cambios | Autor |
|---|---|---|---|
| v1.0 | 2026-07-21 | Versión inicial derivada de ACTA-001 | ZEUS |
| v1.1 | 2026-07-22 | Añade §12 estándar de control documental (control por carpeta, bloque de control, enlaces relativos, fuente única) | ZEUS |
| v1.2 | 2026-07-22 | Define "validar despliegue" (app accesible por web para que el CEO pruebe) y hace **condicional** la validación funcional del CEO: obligatoria en funcionalidades completas, opcional en fixes internos (basta auditoría de ZEUS) | ZEUS |
| v1.3 | 2026-07-22 | Corrige §10: el "directo a `main`" aplica a los repos de **documentación**; el repo de **código** trabaja sobre la rama única `feature/001-scaffolding`. Alinea la nota de arquitecto | ZEUS |
| v2.1 | 2026-07-29 | §6 R3: **la Nota se rige por un test binario** —¿cambia el veredicto de ZEUS?— con lista blanca/negra; y la regla *"una línea salvo que sea necesario o requiera validación del CEO"* **aplica igual a ZEUS y al CEO**, no solo a ODIN. Motivo: la Nota de 002-PI-044 reportó working tree ajeno (ruido); el CEO pidió que las notas/prosa vayan solo cuando importan. Se radica a ODIN en `AGENTS.md` (002-PI-045) | ZEUS |
| v2.0 | 2026-07-29 | §6: **el instructivo es una cáscara delgada sobre Spec Kit** (no repite FR/SC/Acceptance/tasks; la compuerta §4 es el punto donde para la cadena de comandos) y §6 R3 pasa a **protocolo de señales** (`RADICADA→REVISADA→REALIZADO→REVISO→CUMPLE`, con acuse `REVISADA·arranco` obligatorio de ODIN). Motivo (mesa [ACTA_ARQ_06]): el CEO cargaba tokens, contexto perdido y copy-paste sin leer, y quedaba de *stopper* entre las dos IAs. Se radica a ODIN la obligación en `AGENTS.md` ([D-44]) | ZEUS |
| v1.9 | 2026-07-27 | §2: el **rol es fijo, la encarnación no** — ODIN pasa a ser **Kimi (Kimi Code)** en PROTECCIÓN INFANTIL y se fijan las 3 consecuencias de que ODIN y ZEUS no compartan modelo (handoff por referencia, compuerta §4 obligatoria, auditoría de ZEUS como único filtro). §10: **los comandos de la integración Spec Kit se versionan** — el directorio de comandos no puede estar en `.gitignore`. Origen: `.clinerules/` ignorado hizo que 4 specs se escribieran a mano, sin `plan.md` ni `tasks.md` | ZEUS |
| v1.8 | 2026-07-24 | §10: **`main` es el tronco que se mantiene al día**, no "producción" (aclaración del CEO). Actualizarlo es sincronización rutinaria con la rama verde, sin ceremonia ni acta propia. Se registra que IDC **aún no tiene ambiente productivo** (ADR_001 §5 pendiente), por lo que la semántica de despliegue se añadirá cuando exista | ZEUS |
| v1.7 | 2026-07-23 | §6 regla 4: **ventana nueva por frente** con plantilla de prompt de reapertura. La saturación de contexto de ODIN es esperada, no una avería: el contexto vive en archivos (§11) y se recupera leyéndolos en orden, no arrastrando el hilo | ZEUS |
| v1.6 | 2026-07-23 | §6: **ODIN commitea y hace push** en el mismo acto (ZEUS nunca sube código de producto) y su **reporte es de una línea** — `ZEUS revisa <hash>` más una nota solo si hay algo que el diff no muestra. Motivo: los reportes largos saturan el contexto del CEO y la verdad ya vive en el repo (§9) | ZEUS |
| v1.5 | 2026-07-23 | Añade §13 **Nomenclatura**: los números de carpeta están invertidos entre el repo de gestión y el de código, así que el mismo número designa productos distintos. Se fija que manda el **nombre** (`PRODUCTO · TIPO-NNN`) y que todo prompt a ODIN abre con producto y ruta del repo | ZEUS |
| v1.4 | 2026-07-23 | Completa §10 con el modelo de ramas tal como lo fija el CEO: el repo de **código** tiene **dos ramas y solo dos** (pruebas y `main` = producción, que solo recibe merges de liberación); los repos de **documentación** tienen **una sola rama, `main`**, por ser documentos oficiales. Define el paso de liberación: auditoría de ZEUS sobre el conjunto + registro en el ACTA. Sincroniza la versión de la cabecera, que se había quedado en v1.2 | ZEUS |

---
*Documento vivo. Toda modificación se versiona aquí y se refleja de forma condensada en el `AGENTS.md` de los repos para que ODIN la tenga presente.*
