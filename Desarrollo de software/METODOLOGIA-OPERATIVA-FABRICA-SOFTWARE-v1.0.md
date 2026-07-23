# Metodología Operativa — Fábrica de Software IDC

| Campo | Detalle |
|---|---|
| **Documento** | Metodología Operativa de la Fábrica de Software |
| **Versión** | v1.5 (2026-07-23) |
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
| **ODIN** (VS Code + claude-code) | Desarrollador | Redacta spec/plan, implementa, prueba, despliega, commitea | **No decide arquitectura** |

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

**Regla 1 — Prompt por referencia, no por copia.** El prompt apunta a los archivos del repo; no se pega el contenido.

**Plantilla de prompt de ZEUS para ODIN:**
```
CONTEXTO: <PRODUCTO> (repo <ruta>) · SPEC-<NNN> <nombre-corto>.    ← §13
BASE: Sigue la Constitución (.specify/memory/constitution.md) y AGENTS.md del repo.
TAREA: <qué hacer, 1-3 líneas>
REFERENCIA: <ruta del spec/plan a leer, sección>
RESTRICCIONES: <decisiones de arquitectura innegociables>
DEFINICIÓN DE TERMINADO: aplica las 5 reglas de oro. Pruebas incluidas.
ENTREGA: commit con mensaje convencional + resumen de qué tocaste.
```

**Regla 2 — La revisión NO se pega.** Jelkin dice *"ZEUS, revisa el commit `<hash>` del proyecto `<id>`"*. ZEUS lee el diff directo del repo. La referencia sigue §13: producto por **nombre**, nunca por número.

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
  - **Código** (`productos`) → **dos ramas, y solo dos**: `main` es **producción** y únicamente recibe **merges de liberación** (nunca commits directos); el trabajo diario va a la **rama de pruebas** `feature/001-scaffolding` (nombre heredado del scaffolding inicial, no describe su función). **No se abren ramas por feature.** Estado actual: todo vive en la rama de pruebas; aún no se ha liberado nada a producción.
  - **Documentación** (`Gestion-de-proyectos`, `Metodologias`) → **una sola rama: `main`**. Son documentos oficiales: no se ramifican ni se mantienen versiones en paralelo. Su control de versiones es el **bloque de control de cada documento** (§12), no la rama.
- **Liberación a producción:** merge de la rama de desarrollo a `main`, precedido de auditoría de ZEUS sobre el conjunto y registrado en el ACTA-VALIDACION de las features que libera. Un feature está Terminado en desarrollo; está **en producción** solo tras ese merge.
- **Compuerta de calidad:** el control **no es el PR**, es la **auditoría de ZEUS sobre cada commit** — ZEUS revisa el diff después de que ODIN sube.
- **Cierre:** un feature solo se da por Terminado con ACTA-VALIDACION completa.
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
| v1.5 | 2026-07-23 | Añade §13 **Nomenclatura**: los números de carpeta están invertidos entre el repo de gestión y el de código, así que el mismo número designa productos distintos. Se fija que manda el **nombre** (`PRODUCTO · TIPO-NNN`) y que todo prompt a ODIN abre con producto y ruta del repo | ZEUS |
| v1.4 | 2026-07-23 | Completa §10 con el modelo de ramas tal como lo fija el CEO: el repo de **código** tiene **dos ramas y solo dos** (pruebas y `main` = producción, que solo recibe merges de liberación); los repos de **documentación** tienen **una sola rama, `main`**, por ser documentos oficiales. Define el paso de liberación: auditoría de ZEUS sobre el conjunto + registro en el ACTA. Sincroniza la versión de la cabecera, que se había quedado en v1.2 | ZEUS |

---
*Documento vivo. Toda modificación se versiona aquí y se refleja de forma condensada en el `AGENTS.md` de los repos para que ODIN la tenga presente.*
