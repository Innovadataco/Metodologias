# 4.3 — Registros y Logs

## OPM2-23 — Registro de Riesgos

### Propósito
Documentar, evaluar y hacer seguimiento a todos los riesgos del proyecto.

### Estructura

| Columna | Descripción |
|---------|-------------|
| **ID** | Identificador único (R01, R02, ...) |
| **Fecha identificación** | Cuándo se detectó |
| **Descripción** | Qué es el riesgo |
| **Categoría** | Técnico, organizacional, externo, gerencial |
| **Probabilidad** | 1-5 (Muy baja a Muy alta) |
| **Impacto** | 1-5 (Muy bajo a Muy alto) |
| **Puntuación** | Probabilidad × Impacto |
| **Estrategia** | Evitar, Mitigar, Transferir, Aceptar |
| **Acciones** | Qué se va a hacer |
| **Responsable** | Quién gestiona el riesgo |
| **Estado** | Activo, Mitigado, Ocurrido, Cerrado |
| **Fecha revisión** | Próxima revisión |

### Umbrales de Respuesta

| Puntuación | Color | Acción |
|------------|-------|--------|
| 1-4 | 🟢 Verde | Monitorear |
| 5-9 | 🟡 Amarillo | Plan de contingencia |
| 10-16 | 🔴 Rojo | Acción inmediata |
| 17-25 | ⚫ Negro | Escalar a CDP |

---

## OPM2-24 — Registro de Incidencias

### Propósito
Documentar problemas, decisiones y acciones correctivas.

### Estructura

| Columna | Descripción |
|---------|-------------|
| **ID** | Identificador único (I01, I02, ...) |
| **Fecha reporte** | Cuándo se reportó |
| **Reportado por** | Quién lo identificó |
| **Descripción** | Qué es el problema |
| **Impacto** | Alto / Medio / Bajo |
| **Urgencia** | Alta / Media / Baja |
| **Prioridad** | Impacto × Urgencia |
| **Asignado a** | Quién lo resuelve |
| **Fecha límite** | Cuándo debe resolverse |
| **Estado** | Abierto / En progreso / Resuelto / Cerrado |
| **Resolución** | Cómo se resolvió |
| **Lección aprendida** | Qué aprendimos |

---

## OPM2-25 — Registro de Decisiones

### Propósito
Documentar todas las decisiones importantes tomadas durante el proyecto.

### Estructura

| Columna | Descripción |
|---------|-------------|
| **ID** | Identificador único (D01, D02, ...) |
| **Fecha** | Cuándo se tomó |
| **Tema** | Sobre qué se decidió |
| **Contexto** | Por qué se tuvo que decidir |
| **Opciones consideradas** | Alternativas evaluadas |
| **Decisión** | Qué se decidió |
| **Razón** | Por qué se eligió esa opción |
| **Impacto** | En alcance, tiempo, costos, calidad |
| **Decidido por** | Quién tomó la decisión |
| **Comunicado a** | Quiénes fueron informados |

---

## OPM2-26 — Registro de Cambios

### Propósito
Documentar solicitudes de cambio, evaluaciones y decisiones.

### Estructura

| Columna | Descripción |
|---------|-------------|
| **ID** | Identificador único (C01, C02, ...) |
| **Fecha solicitud** | Cuándo se solicitó |
| **Solicitado por** | Quién lo pidió |
| **Descripción** | Qué se quiere cambiar |
| **Justificación** | Por qué se necesita |
| **Impacto alcance** | Sí / No / Parcial |
| **Impacto tiempo** | Días/hitos afectados |
| **Impacto costos** | Costo adicional |
| **Impacto calidad** | Mejora / Neutral / Degradación |
| **Evaluado por** | Quién analizó |
| **Decisión** | Aprobado / Rechazado / Diferido |
| **Decidido por** | Quién aprobó/rechazó |
| **Implementado** | Fecha de implementación |

---

**Fuente:** PM² Overview ES 2019, Guía PM² v3.1 (2023)  
**Análisis:** ZEUS AI Architecture Studio