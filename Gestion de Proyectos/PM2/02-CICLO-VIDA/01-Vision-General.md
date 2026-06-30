# 2.1 — Visión General del Ciclo de Vida PM²

## Estructura del Ciclo de Vida

El ciclo de vida PM² tiene **cuatro fases principales** más el proceso transversal de **Seguimiento y Control**:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   INICIO    │───→│ PLANIFICACIÓN│───→│  EJECUCIÓN  │───→│   CIERRE    │
│             │    │             │    │             │    │             │
│ Establecer  │    │ Desarrollar │    │ Coordinar   │    │ Formalizar  │
│ fundamentos │    │ planes      │    │ ejecución   │    │ cierre      │
│             │    │             │    │             │    │             │
│ LpP         │    │ LpE         │    │ LpC         │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       ↑                                                    │
       └──────────────── SEGUIMIENTO Y CONTROL ─────────────┘
              (a lo largo de todo el proyecto)
```

---

## Las Cuatro Fases

### Fase 1: INICIO (Start)
**Objetivo:** Definir los resultados deseados, crear el caso de negocio, definir el alcance, comenzar adecuadamente el proyecto.

| Aspecto | Detalle |
|---------|---------|
| **Agente clave** | Propietario del Proyecto (PP) |
| **Punto de revisión** | **LpP (Listo para Planificación)** |
| **Artefactos principales** | Solicitud de inicio, Caso de negocio, Acta de constitución |
| **Duración típica** | 2-4 semanas |

### Fase 2: PLANIFICACIÓN (Plan)
**Objetivo:** Asignar el equipo de desarrollo, desarrollar el alcance, planificar el trabajo.

| Aspecto | Detalle |
|---------|---------|
| **Agente clave** | Director de Proyecto (DP) |
| **Punto de revisión** | **LpE (Listo para Ejecución)** |
| **Artefactos principales** | Manual del proyecto, Plan de trabajo, Planes de gestión |
| **Duración típica** | 4-8 semanas |

### Fase 3: EJECUCIÓN (Execute)
**Objetivo:** Coordinar la ejecución de los planes, producir entregables.

| Aspecto | Detalle |
|---------|---------|
| **Agente clave** | Equipo de Desarrollo del Proyecto (EDP) |
| **Punto de revisión** | **LpC (Listo para Cierre)** |
| **Artefactos principales** | Entregables del proyecto, Informes de progreso |
| **Duración típica** | Variable (mayor parte del proyecto) |

### Fase 4: CIERRE (Close)
**Objetivo:** Coordinar aceptación formal, informar sobre desempeño, registrar lecciones aprendidas, cerrar administrativamente.

| Aspecto | Detalle |
|---------|---------|
| **Agente clave** | Partes Interesadas (evalúan entregables) |
| **Punto de revisión** | Cierre formal |
| **Artefactos principales** | Informe de fin de proyecto, Lecciones aprendidas |
| **Duración típica** | 2-4 semanas |

---

## Seguimiento y Control (Proceso Transversal)

Aplica **a lo largo de todo el proyecto**, desde el inicio hasta el cierre:

### Áreas de Seguimiento

| # | Área | Descripción |
|---|------|-------------|
| 1 | **Seguimiento del progreso** | Medir avance contra plan |
| 2 | **Control del calendario** | Gestionar desviaciones temporales |
| 3 | **Control de costes** | Gestionar desviaciones presupuestarias |
| 4 | **Gestión de partes interesadas** | Mantener engagement y comunicación |
| 5 | **Gestión de requisitos** | Asegurar trazabilidad y cumplimiento |
| 6 | **Gestión de cambios** | Procesar solicitudes de cambio |
| 7 | **Gestión de riesgos** | Identificar, evaluar y mitigar riesgos |
| 8 | **Gestión de incidencias** | Resolver problemas y decisiones |
| 9 | **Gestión de calidad** | Asegurar conformidad con estándares |
| 10 | **Gestión de aceptación** | Validar entregables con usuarios |
| 11 | **Gestión de transición** | Preparar migración a operaciones |
| 12 | **Gestión de implementación** | Desplegar en el negocio |
| 13 | **Gestión de externalización** | Controlar contratistas y terceros |

---

## Puntos de Revisión de Fase (Phase Gates)

| Punto | Momento | Propósito |
|-------|---------|-----------|
| **LpP (Listo para Planificación)** | Final de Inicio | Revisar que el proyecto está justificado y autorizado antes de invertir en planificación detallada |
| **LpE (Listo para Ejecución)** | Final de Planificación | Revisar que los planes son viables y el equipo está listo antes de comenzar el trabajo |
| **LpC (Listo para Cierre)** | Final de Ejecución | Revisar que todos los entregables están completos y aceptados antes de cerrar |

### Propósito de los Phase Gates
1. **Control de calidad global:** Asegurar que el proyecto avanza de modo controlado
2. **Toma de decisiones:** Punto formal para aprobar, rechazar o solicitar cambios
3. **Asignación de recursos:** Decidir si se asignan recursos para la siguiente fase
4. **Riesgo y viabilidad:** Reevaluar si el proyecto sigue siendo viable

---

## Características del Ciclo de Vida

### Solapamiento de Fases
- Aunque en cada fase predomina un tipo de actividad, las actividades pueden **solaparse** entre fases
- La planificación también se repite en ejecución (planificación continua)
- El cierre puede comenzar antes de que termine la ejecución (cierre gradual)

### Transiciones
- El proyecto pasa a la siguiente fase cuando se consideran logrados los objetivos de la fase actual
- La aprobación se realiza en reunión formal (o no tan formal) de salida de fase
- La decisión de avanzar es del Comité de Dirección del Proyecto (CDP)

---

## Adaptación del Ciclo de Vida

### Perfiles de Proyecto

| Perfil | Complejidad | Duración típica | Fases |
|--------|------------|-----------------|-------|
| **Light** | Baja | < 3 meses | Inicio simplificado, planificación rápida |
| **Medium** | Media | 3-12 meses | Ciclo completo estándar |
| **Heavy** | Alta | > 12 meses | Ciclo completo + subproyectos |

### Adaptación para Innovadataco

| Tipo de Proyecto | Perfil Recomendado | Adaptación |
|-----------------|-------------------|------------|
| Consultoría rápida (diagnóstico) | Light | Inicio + Ejecución (sin planificación extensa) |
| Consultoría completa (con entregables) | Medium | Ciclo completo estándar |
| Desarrollo de producto tecnológico | Medium/Heavy | PM² Agile: iteraciones en fase de ejecución |
| Proyecto institucional (sector público) | Heavy | Ciclo completo + gobernanza estricta |

---

**Fuente:** PM² Overview ES 2019, Guía PM² v3.1 (2023)  
**Análisis:** ZEUS AI Architecture Studio