# 4.1 — Índice de Artefactos PM²

## Convención de Nomenclatura

```
ES (OPM2-XX.Categoria.Tipo.vX.X).Nombre.(Proyecto).(Fecha).(Version).ext
```

| Elemento | Significado | Ejemplo |
|----------|-------------|---------|
| **ES** | Idioma (Español) | ES |
| **OPM2-XX** | Código del artefacto | OPM2-06 |
| **Categoría** | I/P/E/C (Inicio/Plan/Ejecución/Cierre) | P |
| **Tipo** | TPL/CHL/Log/MC/E | TPL |
| **vX.X** | Versión del artefacto | v3.0.1 |
| **Nombre** | Nombre descriptivo | Plan_de_Trabajo |
| **Proyecto** | Nombre del proyecto | MiProyecto |
| **Fecha** | Fecha de creación | 15-05-2026 |
| **Versión** | Versión del documento | v1.0 |

---

## Tipos de Artefactos

| Tipo | Descripción | Color |
|------|-------------|-------|
| **TPL** | Plantilla (Template) | Documento base |
| **CHL** | Lista de Control (Checklist) | Verificación |
| **Log** | Registro (Log) | Seguimiento |
| **MC** | Matriz de Control (Matrix) | Estructura tabular |
| **E** | Entregable/Evolucionable | Informes, actas |

---

## Índice Completo

### Fase 1: INICIO (I)

| ID | Nombre | Tipo | Propósito |
|----|--------|------|-----------|
| OPM2-01 | Solicitud de Inicio de Proyecto | TPL | Formalizar la petición |
| OPM2-02 | Caso de Negocio | TPL | Justificar la inversión |
| OPM2-03 | Acta de Constitución del Proyecto | TPL | Autorizar formalmente |

### Fase 2: PLANIFICACIÓN (P)

| ID | Nombre | Tipo | Propósito |
|----|--------|------|-----------|
| OPM2-04 | Manual del Proyecto | TPL | Documentar estrategia |
| OPM2-05 | Matriz de Partes Interesadas | MC | Analizar stakeholders |
| OPM2-06 | Plan de Trabajo del Proyecto | TPL | Cronograma y recursos |
| OPM2-07 | Plan de Externalización | TPL | Gestión de terceros |
| OPM2-08 | Plan de Aceptación de Entregables | TPL | Criterios de aceptación |
| OPM2-09 | Plan de Transición | TPL | Migración a operaciones |
| OPM2-10 | Plan de Implementación en el Negocio | TPL | Despliegue y formación |
| OPM2-11 | Plan de Gestión de Requisitos | TPL | Gestión de requisitos |
| OPM2-12 | Plan de Gestión de Cambios | TPL | Control de cambios |
| OPM2-13 | Plan de Gestión de Riesgos | TPL | Gestión de riesgos |
| OPM2-14 | Plan de Gestión de Incidencias | TPL | Gestión de problemas |
| OPM2-15 | Plan de Gestión de Calidad | TPL | Aseguramiento de calidad |
| OPM2-16 | Plan de Gestión de la Comunicación | TPL | Plan de comunicación |

### Fase 3: EJECUCIÓN (E)

| ID | Nombre | Tipo | Propósito |
|----|--------|------|-----------|
| OPM2-17 | Agenda de Reunión | TPL | Preparar reuniones |
| OPM2-18 | Acta de Reunión | TPL | Documentar acuerdos |
| OPM2-19 | Informe de Situación del Proyecto | TPL | Comunicar estado |
| OPM2-20 | Informe de Situación del Proyecto (GVG) | TPL | Informe para alta dirección |
| OPM2-21 | Formulario de Solicitud de Cambio | TPL | Solicitar cambios |
| OPM2-22 | Informe de Progreso del Proyecto | TPL | Reportar avances |
| OPM2-23 | Registro de Riesgos | Log | Tracking de riesgos |
| OPM2-24 | Registro de Incidencias | Log | Tracking de problemas |
| OPM2-25 | Registro de Decisiones | Log | Tracking de decisiones |
| OPM2-26 | Registro de Cambios | Log | Tracking de cambios |

### Listas de Control (CHL) — Transversales

| ID | Nombre | Tipo | Propósito |
|----|--------|------|-----------|
| OPM2-27 | Lista de Control de Salida de Fase | CHL | Verificar fases |
| OPM2-29 | Lista de Control de Aceptación de Entregables | CHL | Verificar entregables |
| OPM2-30 | Lista de Control de Transición | CHL | Verificar transición |
| OPM2-31 | Lista de Control de Implementación en el Negocio | CHL | Verificar despliegue |
| OPM2-32 | Lista de Control de Partes Interesadas | CHL | Verificar stakeholders |

### Fase 4: CIERRE (C)

| ID | Nombre | Tipo | Propósito |
|----|--------|------|-----------|
| OPM2-33 | Informe de Fin de Proyecto | TPL | Formalizar cierre |

---

## Mapa de Artefactos por Fase

```
INICIO (I)          PLANIFICACIÓN (P)     EJECUCIÓN (E)        CIERRE (C)
├─ OPM2-01           ├─ OPM2-04            ├─ OPM2-17           ├─ OPM2-33
├─ OPM2-02           ├─ OPM2-05            ├─ OPM2-18
├─ OPM2-03           ├─ OPM2-06            ├─ OPM2-19
                     ├─ OPM2-07            ├─ OPM2-20
                     ├─ OPM2-08            ├─ OPM2-21
                     ├─ OPM2-09            ├─ OPM2-22
                     ├─ OPM2-10            ├─ OPM2-23
                     ├─ OPM2-11            ├─ OPM2-24
                     ├─ OPM2-12            ├─ OPM2-25
                     ├─ OPM2-13            ├─ OPM2-26
                     ├─ OPM2-14            ├─ OPM2-27
                     ├─ OPM2-15            ├─ OPM2-29
                     ├─ OPM2-16            ├─ OPM2-30
                     └─ OPM2-32            └─ OPM2-31
```

---

## Artefactos por Rol

| Rol | Artefactos Principales |
|-----|----------------------|
| **PP** | OPM2-01, OPM2-02, OPM2-03, OPM2-33 |
| **DP** | OPM2-04, OPM2-06, OPM2-11-16, OPM2-17-26, OPM2-33 |
| **RN** | OPM2-05, OPM2-08, OPM2-09, OPM2-10, OPM2-29, OPM2-31 |
| **EDP** | Entregables del proyecto |

---

**Fuente:** PM² Overview ES 2019, Guía PM² v3.1 (2023)  
**Análisis:** ZEUS AI Architecture Studio
