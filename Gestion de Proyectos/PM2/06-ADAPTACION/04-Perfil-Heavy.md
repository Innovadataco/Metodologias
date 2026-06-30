# 6.4 — Perfil Heavy para Innovadataco

## Definición

Perfil **Heavy** para proyectos grandes, de larga duración y alta complejidad. Gobierna completa, documentación extensiva.

---

## Características

| Aspecto | Especificación |
|---------|---------------|
| **Duración** | > 6 meses |
| **Equipo** | > 6 personas, múltiples equipos |
| **Presupuesto** | > $100K USD |
| **Complejidad técnica** | Alta |
| **Riesgos** | Altos, múltiples |
| **Stakeholders** | Muchos, complejos |
| **Subproyectos** | Sí, posiblemente |

---

## Ciclo de Vida Completo

```
┌─────────┐    ┌─────────┐    ┌──────────────────────────────┐    ┌─────────┐
│  INICIO │───→│   PLAN  │───→│  EJECUCIÓN (con subfases)   │───→│ CIERRE  │
│ (2-4 sem)│    │ (4-8 sem)│    │ Fase 3a → LpC → Fase 3b... │    │ (2-4 sem)│
└─────────┘    └─────────┘    └──────────────────────────────┘    └─────────┘
       LpP           LpE                 LpC (intermedios)             LpC final
```

---

## Artefactos Completos

### Todas las plantillas PM²
- OPM2-01 a OPM2-33 según aplique
- Planes de gestión completos (OPM2-07 a OPM2-16)
- Listas de control (OPM2-27 a OPM2-32)

### Artefactos Adicionales
- Plan de gestión de subproyectos
- Plan de integración
- Plan de gestión de configuración
- Matriz de trazabilidad de requisitos
- Plan de capacitación extensivo

---

## Subproyectos

Para proyectos heavy, dividir en subproyectos manejables:

| Subproyecto | Duración | Equipo | Entregable |
|-------------|----------|--------|------------|
| Subproyecto 1: Análisis | 1-2 meses | 2-3 personas | Documento de requisitos |
| Subproyecto 2: Diseño | 1-2 meses | 2-3 personas | Arquitectura, prototipos |
| Subproyecto 3: Desarrollo | 3-6 meses | 4-6 personas | Producto funcional |
| Subproyecto 4: Implementación | 1-2 meses | 2-3 personas | Despliegue, formación |

---

## Gobierno

### Comité de Dirección del Proyecto (CDP)
- Reuniones quincenales o semanales
- Agenda formal
- Actas de reuniones
- Decisiones documentadas

### Escalamiento
```
Nivel 1: EDP → DP
Nivel 2: DP → CDP
Nivel 3: CDP → Gobierno superior
```

---

## Ejemplos en Innovadataco

| Proyecto | Duración | Equipo | Notas |
|----------|----------|--------|-------|
| Plataforma enterprise para sector público | 12 meses | 8 personas | Subproyectos, múltiples fases LpC |
| Transformación digital institucional | 18 meses | 10+ personas | PM² Heavy + Agile en desarrollo |
| Producto SaaS completo | 9 meses | 6 personas | PM² Híbrido, releases trimestrales |

---

**Fuente:** Adaptación PM² Heavy para INNOVADATACO  
**Análisis:** ZEUS AI Architecture Studio
