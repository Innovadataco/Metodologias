# 5.4 — Backlog y User Stories

## Product Backlog

### Definición
El **Product Backlog** es una lista priorizada y emergente de todo lo que podría ser necesario en el producto. Es la única fuente de requisitos.

### Características
- **Emergente:** Cambia constantemente
- **Priorizada:** Items ordenados por valor
- **Estimada:** Tiene tamaño relativo
- **Transparente:** Visible a todos los stakeholders

### Estructura

| Campo | Descripción |
|-------|-------------|
| **ID** | Identificador único |
| **Descripción** | Qué se necesita (como user story) |
| **Prioridad** | 1-N (1 = más importante) |
| **Story Points** | Tamaño relativo (1, 2, 3, 5, 8, 13, 21) |
| **Valor de negocio** | Alto / Medio / Bajo |
| **Riesgo** | Alto / Medio / Bajo |
| **Dependencias** | IDs de items dependientes |
| **Estado** | Por hacer / En progreso / Hecho |
| **Sprint asignado** | Sprint donde se abordará |

---

## User Stories

### Formato Estándar

```
Como [rol],
quiero [funcionalidad],
para que [beneficio].
```

### Ejemplo
```
Como gerente de proyectos,
quiero ver un dashboard con el estado de todos los proyectos,
para que pueda identificar rápidamente cuáles necesitan atención.
```

### Criterios de Aceptación

```
Dado que [contexto inicial],
cuando [acción],
entonces [resultado esperado].
```

### Ejemplo
```
Dado que estoy en la página principal,
cuando selecciono "Nuevo Proyecto",
entonces se abre un formulario con los campos: nombre, descripción, fecha inicio, fecha fin.
```

---

## Estimación con Story Points

### Escala de Fibonacci
| Story Points | Tamaño Relativo |
|--------------|-----------------|
| 1 | Muy pequeño (cambio menor) |
| 2 | Pequeño |
| 3 | Mediano |
| 5 | Grande |
| 8 | Muy grande |
| 13 | Enorme (debería dividirse) |
| 21 | Gigante (debe dividirse) |

### Factores de Complejidad
- Complejidad técnica
- Incertidumbre/riesgo
- Esfuerzo de trabajo
- Dependencias

---

## Definition of Done (DoD)

### Definición
Conjunto de criterios que un incremento debe cumplir para considerarse "terminado".

### Ejemplo de DoD
- [ ] Código escrito y revisado (code review)
- [ ] Pruebas unitarias pasan (>80% cobertura)
- [ ] Pruebas de integración pasan
- [ ] Documentación técnica actualizada
- [ ] Documentación de usuario actualizada
- [ ] Sin defectos críticos o altos abiertos
- [ ] Revisado por Product Owner
- [ ] Desplegado en ambiente de pruebas

---

**Fuente:** Guía PM²-Ágil v3.0.1 (2019)  
**Análisis:** ZEUS AI Architecture Studio