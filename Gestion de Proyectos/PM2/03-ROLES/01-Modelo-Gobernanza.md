# 3.1 — Modelo de Gobernanza PM²

## Estructura de Capas

PM² define un modelo de gobernanza con **cuatro capas organizativas** más la capa de gobernanza superior:

```
┌─────────────────────────────────────────┐
│     CAPA DE GOBERNANZA (Governance)    │
│    Comités de gestión de alta dirección │
│    Definen prioridades, decisiones de   │
│    inversión, asignación de recursos    │
├─────────────────────────────────────────┤
│      CAPA RECTORA (Steering)           │
│   Dirección y orientaciones generales    │
│   Mantiene proyecto centrado en objetivos│
│   Informa al órgano de gobernanza       │
│   (Roles de dirección y gestión)        │
├─────────────────────────────────────────┤
│       CAPA DE DIRECCIÓN (Direct)        │
│    Lidera el proyecto, responsable del   │
│    caso de negocio, moviliza recursos,   │
│    supervisa desempeño                   │
│    • Propietario de proyecto (PP)        │
│    • Proveedor de soluciones (PS)       │
├─────────────────────────────────────────┤
│        CAPA DE GESTIÓN (Manage)         │
│    Operaciones día a día, organiza,      │
│    supervisa, controla trabajo           │
│    • Responsable de negocio (RN)        │
│    • Director de proyecto (DP)          │
├─────────────────────────────────────────┤
│       CAPA DE OPERACIÓN (Operate)        │
│    Lleva a cabo el trabajo del proyecto  │
│    • Equipo de desarrollo (EDP)         │
│    • Grupo de implementación (GIN)      │
└─────────────────────────────────────────┘
```

**Nota importante:** Solo hay **un equipo de proyecto**, compuesto por las personas que asuman las funciones definidas en las capas de dirección, gestión y operación.

---

## Roles Principales

| Rol | Capa | Función Principal |
|-----|------|-------------------|
| **PP** — Propietario del Proyecto | Dirección | Cliente del proyecto. Responsable del éxito global. |
| **PS** — Proveedor de Soluciones | Dirección | Provee recursos y soluciones técnicas. |
| **RN** — Responsable de Negocio | Gestión | Gestiona la relación con el negocio/usuarios. |
| **DP** — Director de Proyecto | Gestión | Gestiona el proyecto día a día. |
| **EDP** — Equipo de Desarrollo | Operación | Realiza el trabajo técnico del proyecto. |
| **GIN** — Grupo de Implementación | Operación | Implementa la solución en el negocio. |
| **CDP** — Comité de Dirección | Rectora | Toma decisiones estratégicas. |

---

## Matriz RASCI

La matriz RASCI define quién es **R**esponsable, **A**ccountable, **S**oporta, **C**onsulta, **I**nforma para cada actividad:

| Actividad | PP | PS | RN | DP | EDP | GIN |
|-----------|----|----|----|----|-----|-----|
| Definir objetivos de negocio | A/R | C | R | I | I | I |
| Autorizar proyecto | A/R | C | C | I | I | I |
| Asignar presupuesto | A | R | C | C | I | I |
| Definir alcance | A | C | R | R | C | I |
| Planificar proyecto | C | C | C | A/R | R | I |
| Ejecutar trabajo | I | C | I | A | R | I |
| Asegurar calidad | I | R | C | A/R | R | C |
| Gestionar riesgos | C | C | C | A/R | R | I |
| Comunicar avances | I | I | R | A/R | C | I |
| Aceptar entregables | A | C | R | R | C | I |
| Cerrar proyecto | A/R | C | R | R | C | I |
| Transición a operaciones | C | C | A/R | R | C | R |

**Leyenda:**
- **A** = Accountable (Responsable final, toma decisión)
- **R** = Responsible (Ejecuta el trabajo)
- **C** = Consulted (Consultado antes de decidir)
- **I** = Informed (Informado después de decidir)

---

## Comité de Dirección del Proyecto (CDP)

### Composición
Cuatro funciones de las capas de gestión y dirección:
- PP (Propietario) — **Presidente**
- PS (Proveedor)
- RN (Responsable de Negocio)
- DP (Director de Proyecto)

+ otros roles opcionales según necesidades

### Función Principal
Órgano principal de **toma de decisiones y resolución de incidencias**.

### Responsabilidades
- Aprobar documentos clave (caso de negocio, planes, informes)
- Resolver incidencias importantes
- Decidir solicitudes de cambios relevantes
- Elevar y decidir decisiones que afecten al proyecto
- Aprobar transiciones de fase (LpP, LpE, LpC)

---

## Diagrama de Relaciones

```
                    ┌─────────────────┐
                    │   GOBERNANZA    │
                    │  (Alta Dirección)│
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │       CDP       │
                    │  (Comité Dirección)│
                    │  PP + PS + RN + DP│
                    └────────┬────────┘
                             │
           ┌─────────────────┼─────────────────┐
           │                 │                 │
           ▼                 ▼                 ▼
    ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
    │     PP      │   │     RN      │   │     DP      │
    │  Propietario│   │  Responsable│   │  Director   │
    │  del Proyecto│   │  de Negocio │   │  de Proyecto│
    └─────────────┘   └─────────────┘   └──────┬──────┘
                                              │
                                              ▼
                                       ┌─────────────┐
                                       │     EDP     │
                                       │  Equipo de  │
                                       │  Desarrollo │
                                       └─────────────┘
```

---

**Fuente:** PM² Overview ES 2019, Guía PM² v3.1 (2023)  
**Análisis:** ZEUS AI Architecture Studio
