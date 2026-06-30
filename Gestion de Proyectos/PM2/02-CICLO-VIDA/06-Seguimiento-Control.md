# 2.6 — Seguimiento y Control

## Definición

El **Seguimiento y Control** es un **proceso transversal** que aplica a lo largo de todo el ciclo de vida del proyecto, desde el inicio hasta el cierre.

> *"Realizar seguimiento de variables del proyecto, medir el progreso, gestionar cambios, abordar riesgos e incidencias, identificar medidas correctoras."*

---

## Las 13 Áreas de Seguimiento y Control

| # | Área | Descripción | Fase Principal |
|---|------|-------------|---------------|
| 1 | **Seguimiento del progreso** | Medir avance real vs. planificado. Identificar desviaciones. | Ejecución |
| 2 | **Control del calendario** | Gestionar desviaciones temporales. Actualizar cronograma. | Ejecución |
| 3 | **Control de costes** | Gestionar desviaciones presupuestarias. Actualizar presupuesto. | Ejecución |
| 4 | **Gestión de partes interesadas** | Mantener engagement. Resolver conflictos. Actualizar matriz. | Todo |
| 5 | **Gestión de requisitos** | Asegurar trazabilidad. Verificar cumplimiento. Gestionar cambios. | Ejecución |
| 6 | **Gestión de cambios** | Procesar solicitudes. Evaluar impacto. Decidir. Implementar. | Ejecución |
| 7 | **Gestión de riesgos** | Identificar nuevos riesgos. Reevaluar existentes. Implementar respuestas. | Todo |
| 8 | **Gestión de incidencias** | Reportar problemas. Evaluar. Resolver. Documentar. | Ejecución |
| 9 | **Gestión de calidad** | Verificar conformidad. Realizar QA. Documentar no conformidades. | Ejecución |
| 10 | **Gestión de aceptación** | Validar entregables con usuarios. Obtener aprobaciones formales. | Ejecución/Cierre |
| 11 | **Gestión de transición** | Preparar migración a operaciones. Planificar despliegue. | Ejecución/Cierre |
| 12 | **Gestión de implementación** | Desplegar en el negocio. Formar usuarios. Establecer soporte. | Cierre |
| 13 | **Gestión de externalización** | Controlar contratistas. Verificar entregables. Gestionar contratos. | Ejecución |

---

## Ciclo de Control

```
┌─────────────────────────────────────────────────────────────────┐
│              CICLO DE SEGUIMIENTO Y CONTROL                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐       │
│   │   PLAN      │───→│   EJECUTAR  │───→│   MEDIR     │       │
│   │ (Baseline)  │    │  (Trabajo)  │    │  (Avance)   │       │
│   └─────────────┘    └─────────────┘    └──────┬──────┘       │
│                                                 │              │
│                                                 ▼              │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐       │
│   │  CORREGIR   │←───│   DECIDIR   │←───│  COMPARAR   │       │
│   │ (Acciones)  │    │  (¿Ajustar? │    │ (Real vs Plan)│      │
│   └─────────────┘    └─────────────┘    └─────────────┘       │
│         │                                                        │
│         └───────────────────────────────────────────────────────→│
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Herramientas de Seguimiento

### Indicadores Clave (KPIs)

| Indicador | Fórmula | Frecuencia |
|-----------|---------|-----------|
| **SPI (Schedule Performance Index)** | EV / PV | Semanal |
| **CPI (Cost Performance Index)** | EV / AC | Semanal |
| **EAC (Estimate at Completion)** | BAC / CPI | Mensual |
| **ETC (Estimate to Complete)** | EAC - AC | Mensual |
| **Desviación de cronograma** | Fecha real - Fecha planificada | Semanal |
| **Desviación de costos** | Costo real - Costo planificado | Mensual |
| **Cumplimiento de alcance** | % requisitos completados | Quincenal |
| **Satisfacción de stakeholders** | Encuesta 1-5 | Mensual |

### Técnicas de Medición

| Técnica | Uso |
|---------|-----|
| **Valor Ganado (EVM)** | Medir progreso objetivo |
| **Diagrama de Gantt** | Visualizar cronograma |
| **Curva S** | Comparar acumulado plan vs. real |
| **Tablero de control (Dashboard)** | Visión integrada del estado |
| **Semáforos** | Rápida identificación de problemas |
| **Reuniones de seguimiento** | Sincronización del equipo |

---

## Frecuencia de Seguimiento por Perfil

| Perfil | Reuniones | Informes | Revisión de riesgos |
|--------|-----------|----------|---------------------|
| **Light** | Semanal | Mensual | Quincenal |
| **Medium** | Bisemanales | Quincenales | Semanal |
| **Heavy** | Semanales | Semanales | Bisemanales |

---

## Artefactos de Seguimiento

| ID | Nombre | Tipo | Uso |
|----|--------|------|-----|
| OPM2-22 | Informe de Progreso | TPL | Estado periódico |
| OPM2-23 | Registro de Riesgos | Log | Tracking de riesgos |
| OPM2-24 | Registro de Incidencias | Log | Tracking de problemas |
| OPM2-25 | Registro de Decisiones | Log | Tracking de decisiones |
| OPM2-26 | Registro de Cambios | Log | Tracking de cambios |
| OPM2-27 | Lista de Control de Salida de Fase | CHL | Verificar fases |

---

## Escalamiento de Problemas

| Nivel | Problema | Quién decide |
|-------|----------|-------------|
| **1** | Incidencia técnica menor | EDP / DP |
| **2** | Desviación de cronograma | DP / RN |
| **3** | Solicitud de cambio significativa | CDP |
| **4** | Riesgo crítico / Presupuesto | CDP / Alta dirección |
| **5** | Cancelación del proyecto | Alta dirección / PP |

---

**Fuente:** PM² Overview ES 2019, Guía PM² v3.1 (2023)  
**Análisis:** ZEUS AI Architecture Studio