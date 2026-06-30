# 2.7 — Puntos de Revisión de Fase (Phase Gates)

## Definición

Los **Phase Gates** (o puntos de revisión de fase) son **puntos de control formales** al final de cada fase del ciclo de vida del proyecto. Su propósito es asegurar que el proyecto avanza de modo controlado y que se cumplen los criterios de calidad antes de pasar a la siguiente fase.

---

## Los Tres Phase Gates de PM²

| Punto | Nombre Completo | Momento | Propósito |
|-------|-----------------|---------|-----------|
| **LpP** | Listo para Planificación | Final de Inicio | Asegurar justificación y autorización |
| **LpE** | Listo para Ejecución | Final de Planificación | Asegurar que planes son viables |
| **LpC** | Listo para Cierre | Final de Ejecución | Asegurar que entregables están completos |

```
┌──────────┐   LpP   ┌──────────┐   LpE   ┌──────────┐   LpC   ┌──────────┐
│  INICIO  │───────→│   PLAN   │───────→│  EJEC.   │───────→│  CIERRE  │
│          │        │          │        │          │        │          │
│ ¿Está    │        │ ¿Están   │        │ ¿Están   │        │ ¿Todo    │
│ justifi- │        │ listos   │        │ comple-  │        │ listo    │
│ cado?    │        │ para     │        │ tos?     │        │ para     │
│          │        │ empezar? │        │          │        │ cerrar?  │
└──────────┘        └──────────┘        └──────────┘        └──────────┘
```

---

## LpP — Listo para Planificación

### Propósito
Asegurar que el proyecto está debidamente justificado y autorizado antes de invertir recursos en planificación detallada.

### Criterios de Evaluación

| Categoría | Criterio | Pregunta |
|-----------|----------|----------|
| **Justificación** | Caso de negocio | ¿El caso de negocio es convincente y completo? |
| | Viabilidad | ¿El proyecto es técnicamente y financieramente viable? |
| **Autorización** | Acta de constitución | ¿El acta está firmada por el PP? |
| | Presupuesto | ¿Hay presupuesto preliminar aprobado? |
| **Organización** | Roles | ¿Los roles clave están asignados? |
| | DP | ¿El DP ha aceptado el nombramiento? |
| **Riesgos** | Iniciales | ¿Los riesgos principales son aceptables? |
| | Mitigaciones | ¿Hay planes para los riesgos críticos? |

### Quién Decide
**Comité de Dirección del Proyecto (CDP)**, presidido por el PP.

### Decisiones Posibles

| Decisión | Significado | Acción Siguiente |
|----------|-------------|-----------------|
| **APROBAR** | Proceder a Planificación | Iniciar fase de Planificación |
| **CONDICIONAR** | Proceder con ajustes | Identificar acciones, reevaluar en X días |
| **RECHAZAR** | No proceder | Documentar razones, archivar solicitud |
| **APLAZAR** | Posponer decisión | Obtener más información, reconvocar |

---

## LpE — Listo para Ejecución

### Propósito
Asegurar que el proyecto está planificado adecuadamente y el equipo está listo para comenzar el trabajo de ejecución.

### Criterios de Evaluación

| Categoría | Criterio | Pregunta |
|-----------|----------|----------|
| **Alcance** | Definición | ¿El alcance está definido claramente? |
| | WBS | ¿La estructura de trabajo está completa? |
| **Cronograma** | Realismo | ¿El cronograma es realista y aprobado? |
| | Hitos | ¿Los hitos son alcanzables? |
| **Recursos** | Equipo | ¿El equipo está asignado y disponible? |
| | Competencias | ¿El equipo tiene las habilidades necesarias? |
| **Presupuesto** | Aprobación | ¿El presupuesto detallado está aprobado? |
| | Contingencia | ¿Hay reserva para imprevistos? |
| **Riesgos** | Identificación | ¿Los riesgos están identificados? |
| | Planes | ¿Hay planes de respuesta para riesgos altos? |
| **Calidad** | Estándares | ¿Los estándares de calidad están definidos? |
| | Criterios | ¿Los criterios de aceptación son claros? |
| **Comunicación** | Plan | ¿El plan de comunicación está aprobado? |
| | Stakeholders | ¿Las partes interesadas están informadas? |

### Quién Decide
**Comité de Dirección del Proyecto (CDP)**.

### Decisiones Posibles

| Decisión | Significado | Acción Siguiente |
|----------|-------------|-----------------|
| **APROBAR** | Proceder a Ejecución | Iniciar trabajo del proyecto |
| **CONDICIONAR** | Proceder con ajustes | Completar acciones pendientes |
| **RECHAZAR** | No proceder | Revisar planificación o cancelar |
| **APLAZAR** | Posponer | Resolver problemas identificados |

---

## LpC — Listo para Cierre

### Propósito
Asegurar que todos los entregables están completos, aceptados y el proyecto está listo para cerrar formalmente.

### Criterios de Evaluación

| Categoría | Criterio | Pregunta |
|-----------|----------|----------|
| **Entregables** | Completitud | ¿Todos los entregables están completos? |
| | Aceptación | ¿Han sido aceptados formalmente? |
| **Calidad** | Conformidad | ¿Cumplen los criterios de calidad? |
| | Pruebas | ¿Las pruebas fueron exitosas? |
| **Requisitos** | Cumplimiento | ¿Todos los requisitos están satisfechos? |
| | Trazabilidad | ¿Hay trazabilidad completa? |
| **Documentación** | Completa | ¿La documentación está completa? |
| | Archivada | ¿Está lista para archivar? |
| **Lecciones** | Capturadas | ¿Se han capturado lecciones aprendidas? |
| | Compartidas | ¿Están disponibles para otros proyectos? |
| **Transición** | Plan | ¿El plan de transición está aprobado? |
| | Operaciones | ¿El equipo de operación está listo? |
| **Administrativo** | Contratos | ¿Los contratos están cerrados? |
| | Recursos | ¿Los recursos están liberados? |

### Quién Decide
**Comité de Dirección del Proyecto (CDP)** + Partes interesadas clave.

### Decisiones Posibles

| Decisión | Significado | Acción Siguiente |
|----------|-------------|-----------------|
| **APROBAR** | Proceder a Cierre | Iniciar cierre formal |
| **CONDICIONAR** | Completar pendientes | Resolver acciones antes de cerrar |
| **RECHAZAR** | Continuar ejecución | Completar entregables pendientes |
| **APLAZAR** | Posponer cierre | Reevaluar en fecha futura |

---

## Formato de la Reunión de Phase Gate

### Agenda Típica

| Tiempo | Tema | Responsable |
|--------|------|-------------|
| 0:00-0:05 | Bienvenida y objetivo | PP (presidente CDP) |
| 0:05-0:20 | Presentación del estado | DP |
| 0:20-0:35 | Revisión de criterios | CDP |
| 0:35-0:45 | Preguntas y discusión | Todos |
| 0:45-0:50 | Decisión formal | CDP |
| 0:50-0:55 | Próximos pasos | DP |
| 0:55-1:00 | Cierre | PP |

### Documentos Requeridos

| Phase Gate | Documentos |
|------------|-----------|
| **LpP** | Solicitud de inicio, Caso de negocio, Acta de constitución |
| **LpE** | Manual del proyecto, Plan de trabajo, Planes de gestión, Matriz de stakeholders |
| **LpC** | Informe de progreso final, Registro de riesgos (actualizado), Entregables, Lecciones aprendidas |

---

## Listas de Control (Checklists) para Phase Gates

| Phase Gate | Lista de Control |
|------------|-----------------|
| LpP | OPM2-27 (adaptada) |
| LpE | OPM2-27 (completa) |
| LpC | OPM2-27 + OPM2-29 + OPM2-30 + OPM2-31 |

---

**Fuente:** PM² Overview ES 2019, Guía PM² v3.1 (2023)  
**Análisis:** ZEUS AI Architecture Studio