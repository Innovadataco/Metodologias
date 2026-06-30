# 5.3 — Sprints e Iteraciones

## Definición de Sprint

Un **Sprint** (o Iteración) es un ciclo de tiempo fijo (generalmente 1-4 semanas) durante el cual se crea un **incremento funcional** del producto. Al final de cada sprint, el resultado debe ser potencialmente entregable.

---

## Estructura de un Sprint

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPRINT (ej: 2 semanas)                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Día 1: SPRINT PLANNING                                          │
│  ├── Revisar Product Backlog (con PP/RN)                          │
│  ├── Seleccionar items para Sprint Backlog                       │
│  ├── Definir Definition of Done                                  │
│  └── Estimar esfuerzo (story points)                             │
│                                                                  │
│  Días 2-N-1: DESARROLLO + DAILY STAND-UPS                       │
│  ├── Daily: ¿Qué hice ayer? ¿Qué haré hoy? ¿Impedimentos?       │
│  ├── Trabajo en items del Sprint Backlog                          │
│  └── Actualizar tablero/Burndown                                  │
│                                                                  │
│  Día N-1: SPRINT REVIEW (Demo)                                   │
│  ├── Demostrar incremento a stakeholders                           │
│  ├── Recoger feedback                                              │
│  └── Actualizar Product Backlog                                    │
│                                                                  │
│  Día N: SPRINT RETROSPECTIVE                                     │
│  ├── ¿Qué funcionó bien?                                           │
│  ├── ¿Qué podemos mejorar?                                         │
│  ├── ¿Qué acciones tomaremos?                                      │
│  └── Documentar lecciones aprendidas                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Sprint Planning

### Participantes
- Product Owner (PP/RN)
- Scrum Master (DP)
- Equipo de Desarrollo (EDP)

### Agenda
| Tiempo | Actividad |
|--------|-----------|
| 0-30 min | Revisar visión y objetivos del sprint |
| 30-60 min | Seleccionar items del Product Backlog |
| 60-90 min | Descomponer en tareas, estimar |
| 90-120 min | Definir Definition of Done, compromiso |

### Outputs
- Sprint Backlog definido
- Sprint Goal establecido
- Definition of Done acordado

---

## Daily Stand-up

### Formato
- **Duración:** Máximo 15 minutos
- **Participantes:** EDP completo + DP
- **Preguntas:**
  1. ¿Qué hice ayer que contribuyó al sprint?
  2. ¿Qué haré hoy que contribuya al sprint?
  3. ¿Tengo impedimentos?

### Reglas
- Solo información relevante al sprint
- No resolver problemas en el stand-up (se resuelve después)
- Todos de pie (para que sea breve)
- A la misma hora, mismo lugar

---

## Sprint Review (Demo)

### Participantes
- EDP (presenta)
- PP, RN, PS (evalúan)
- Stakeholders invitados

### Agenda
| Tiempo | Actividad |
|--------|-----------|
| 0-10 min | Contexto: ¿Qué se planeó? |
| 10-40 min | Demo del incremento |
| 40-50 min | Feedback de stakeholders |
| 50-60 min | Discusión de próximos pasos, actualización backlog |

### Outputs
- Feedback documentado
- Product Backlog actualizado
- Nuevos items identificados

---

## Sprint Retrospective

### Formato
| Fase | Duración | Actividad |
|------|----------|-----------|
| **Preparación** | 5 min | Recordar acciones del sprint anterior |
| **Recolección** | 15 min | ¿Qué funcionó? ¿Qué no? ¿Ideas? |
| **Análisis** | 15 min | Agrupar temas, identificar patrones |
| **Acciones** | 15 min | Definir 2-3 acciones de mejora |
| **Cierre** | 5 min | Compromiso del equipo |

### Técnicas Comunes
- **Start-Stop-Continue:** ¿Qué empezar? ¿Qué parar? ¿Qué continuar?
- **4Ls:** Liked, Learned, Lacked, Longed for
- **Timeline:** Visualizar eventos del sprint
- **Dot voting:** Priorizar temas a discutir

---

## Métricas de Sprint

| Métrica | Descripción | Fórmula |
|---------|-------------|---------|
| **Velocity** | Puntos completados por sprint | Suma story points completados |
| **Burndown** | Trabajo pendiente vs. tiempo | Gráfico diario |
| **Burnup** | Trabajo completado acumulado | Gráfico acumulativo |
| **Cumulative Flow** | Distribución de items por estado | Diagrama de flujo |
| **Defect Rate** | Defectos encontrados vs. completados | % de defectos |

---

**Fuente:** Guía PM²-Ágil v3.0.1 (2019)  
**Análisis:** ZEUS AI Architecture Studio