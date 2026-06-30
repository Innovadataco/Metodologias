# 5.6 — Métricas Ágiles

## Métricas Fundamentales

### Velocity (Velocidad)

**Definición:** Cantidad de trabajo completado por sprint, medido en story points.

**Cálculo:**
```
Velocity = Suma de story points de items completados en el sprint
```

**Uso:**
- Predecir capacidad del equipo
- Planificar sprints futuros
- Identificar tendencias (mejora/degradación)

**Ejemplo:**
| Sprint | Story Points Completados |
|--------|--------------------------|
| Sprint 1 | 23 |
| Sprint 2 | 25 |
| Sprint 3 | 21 |
| Sprint 4 | 28 |
| **Promedio** | **24.25** |

---

### Burndown Chart

**Definición:** Gráfico que muestra el trabajo pendiente vs. el tiempo transcurrido en el sprint.

**Componentes:**
- **Eje X:** Días del sprint
- **Eje Y:** Story points (o tareas) pendientes
- **Línea ideal:** Pendiente decreciente uniforme
- **Línea real:** Pendiente actual del equipo

**Interpretación:**
- Línea real **por debajo** de ideal: equipo va rápido
- Línea real **por encima** de ideal: riesgo de no terminar

---

### Burnup Chart

**Definición:** Gráfico que muestra el trabajo completado acumulado.

**Componentes:**
- **Eje X:** Días/sprints
- **Eje Y:** Story points completados (acumulado)
- **Línea de alcance:** Total de story points del proyecto

**Interpretación:**
- Útil para predecir fecha de finalización
- Muestra cambios de alcance (línea de alcance se mueve)

---

### Cumulative Flow Diagram (CFD)

**Definición:** Diagrama que muestra la distribución de items por estado a lo largo del tiempo.

**Estados típicos:**
- Por hacer (To Do)
- En progreso (In Progress)
- En revisión (In Review)
- Hecho (Done)

**Interpretación:**
- **Amasamiento** en "En progreso": cuello de botella
- **Amasamiento** en "Por hacer": equipo no tiene capacidad

---

### Defect Rate (Tasa de Defectos)

**Definición:** Porcentaje de defectos encontrados vs. items completados.

**Cálculo:**
```
Defect Rate = (Defectos encontrados / Items completados) × 100
```

**Uso:**
- Medir calidad del proceso
- Identificar necesidad de mejora en QA

---

## Métricas Avanzadas

### Lead Time
**Tiempo desde que se solicita un item hasta que se completa.**

### Cycle Time
**Tiempo desde que se inicia el trabajo en un item hasta que se completa.**

### WIP (Work In Progress)
**Cantidad de items en progreso simultáneamente.**
- Objetivo: Limitar WIP para mejorar flujo
- Regla: WIP máximo = número de personas × 2

---

## Tablero de Métricas Recomendado

| Métrica | Frecuencia | Propósito | Quién la revisa |
|---------|-----------|-----------|-----------------|
| Velocity | Por sprint | Capacidad de planificación | Equipo + PO |
| Burndown | Diario | Progreso del sprint | Equipo |
| Burnup | Por sprint | Predicción de finalización | PO + stakeholders |
| CFD | Semanal | Identificar cuellos de botella | DP |
| Defect Rate | Por sprint | Calidad del proceso | Equipo |
| Lead/Cycle Time | Por item | Eficiencia del flujo | DP |
| WIP | Diario | Flujo de trabajo | Equipo |

---

**Fuente:** Guía PM²-Ágil v3.0.1 (2019)  
**Análisis:** ZEUS AI Architecture Studio