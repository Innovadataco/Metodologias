# 6.6 — Matriz RASCI para Innovadataco

## Roles en Innovadataco

| Rol PM² | Persona(s) | Responsabilidad Principal |
|---------|------------|---------------------------|
| **PP** | Jelkin (CEO) | Visión, aprobaciones, caso de negocio |
| **DP** | Jelkin / Asignado | Gestión diaria, coordinación, riesgos |
| **RN** | Jelkin / Diana | Requisitos, aceptación, transición |
| **EDP** | Equipo técnico + ZEUS | Desarrollo, entregables técnicos |
| **PS** | Juan / Externo | Arquitectura técnica, viabilidad |

---

## Matriz RASCI Simplificada

### Proyectos Light (< 2 meses)

| Actividad | PP | DP | EDP |
|-----------|----|----|-----|
| Definir objetivos | A/R | C | I |
| Autorizar proyecto | A/R | C | I |
| Planificar trabajo | C | A/R | C |
| Ejecutar trabajo | I | A | R |
| Aceptar entregables | A | R | C |
| Cerrar proyecto | A/R | R | C |

### Proyectos Medium (2-6 meses)

| Actividad | PP | DP | RN | EDP | PS |
|-----------|----|----|----|-----|----|
| Caso de negocio | A/R | C | C | I | I |
| Plan de trabajo | C | A/R | C | R | C |
| Ejecución técnica | I | A | I | R | C |
| Asegurar calidad | I | A/R | C | R | R |
| Comunicar avances | I | A/R | R | C | I |
| Gestionar riesgos | C | A/R | C | R | C |
| Cerrar proyecto | A/R | R | R | C | I |

### Proyectos Heavy (> 6 meses)

| Actividad | PP | DP | RN | EDP | PS | CDP |
|-----------|----|----|----|-----|----|-----|
| Decisiones estratégicas | A | C | C | I | I | R |
| Aprobar fases | A | C | C | I | I | R |
| Cambios significativos | A | C | C | C | C | R |
| Escalamiento de riesgos | A | C | C | C | C | R |

---

## Reglas para Innovadataco

1. **PP (Jelkin)** siempre es Accountable para decisiones estratégicas
2. **DP** puede ser la misma persona que PP en proyectos pequeños
3. **EDP** incluye ZEUS para generación de documentos y análisis
4. **PS** (Juan) participa en proyectos técnicos complejos
5. **CDP** se forma solo para proyectos Medium/Heavy

---

## Ejemplo: Proyecto de Desarrollo de Producto

| Actividad | PP (Jelkin) | DP (Asignado) | RN (Jelkin) | EDP (Equipo+ZEUS) | PS (Juan) |
|-----------|-------------|---------------|-------------|-------------------|-----------|
| Definir visión producto | A/R | C | C | I | C |
| Crear backlog | A | C | R | C | C |
| Planificar sprint | I | A/R | C | R | I |
| Desarrollar features | I | A | I | R | C |
| Revisar código | I | A | I | R | R |
| Demo sprint | A | R | R | R | I |
| Aceptar release | A/R | R | R | C | I |
| Desplegar producción | I | A | R | R | R |

---

**Fuente:** Adaptación para INNOVADATACO  
**Análisis:** ZEUS AI Architecture Studio
