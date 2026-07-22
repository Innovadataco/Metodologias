# Metodología Operativa — Fábrica de Software IDC

| Campo | Detalle |
|---|---|
| **Documento** | Metodología Operativa de la Fábrica de Software |
| **Versión** | v1.0 (aprobada 2026-07-21) |
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
CONTEXTO: Proyecto <id>, feature <nombre>.
BASE: Sigue la Constitución (.specify/memory/constitution.md) y AGENTS.md del repo.
TAREA: <qué hacer, 1-3 líneas>
REFERENCIA: <ruta del spec/plan a leer, sección>
RESTRICCIONES: <decisiones de arquitectura innegociables>
DEFINICIÓN DE TERMINADO: aplica las 5 reglas de oro. Pruebas incluidas.
ENTREGA: commit con mensaje convencional + resumen de qué tocaste.
```

**Regla 2 — La revisión NO se pega.** Jelkin dice *"ZEUS, revisa el commit `<hash>` del proyecto `<id>`"*. ZEUS lee el diff directo del repo.

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
- [ ] Código subido a `main` en GitHub. *(Regla 2)*
- [ ] Pruebas escritas y pasando. *(Regla 3)*
- [ ] Despliegue validado en ambiente objetivo. *(Regla 4)*
- [ ] Revisión de arquitectura de ZEUS sobre el commit: aprobada.
- [ ] Validación funcional de Jelkin: aprobada.
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

- **Ramas:** trabajo **directo sobre `main`** (decisión CEO). No se usan ramas por feature.
- **Compuerta de calidad:** al no haber PR, la **auditoría de ZEUS sobre cada commit** es el control — ZEUS revisa el diff después de que ODIN sube.
- **Cierre:** un feature solo se da por Terminado con ACTA-VALIDACION completa.
- **Secrets:** siempre por variables de entorno, nunca en el código.
- **Código sensible:** preferir revisión con modelos locales.

> ⚠️ **Nota de arquitecto:** trabajar directo sobre `main` gana velocidad pero elimina la red de seguridad del PR. Por eso la **auditoría post-commit de ZEUS es innegociable** — es el único filtro antes de que un error quede en la rama principal.

## 11. Arquitectura de memoria (3 capas)

El contexto vive en archivos versionados, no en la cabeza de ningún agente:
1. **Verdad del proyecto** → repo con Spec Kit (spec, plan, tasks).
2. **Decisiones y actas** → repo `Gestion-de-proyectos` + Obsidian.
3. **Memoria operativa de ZEUS** → `~/.claude/.../memory/` (solo punteros cortos).

## 12. Control de versiones de este documento

| Versión | Fecha | Cambios | Autor |
|---|---|---|---|
| v1.0 | 2026-07-21 | Versión inicial derivada de ACTA-001 | ZEUS |

---
*Documento vivo. Toda modificación se versiona aquí y se refleja de forma condensada en el `AGENTS.md` de los repos para que ODIN la tenga presente.*
