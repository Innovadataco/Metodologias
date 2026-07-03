# Proyecto: 001-2026-INNOVADATACO-PLATAFORMACORE
## Spec de Producto: MVP Plataforma Innovadataco

**Estado:** BORRADOR / PARA REVISIÓN
**Versión:** 1.0

### 1. Objetivo
Desarrollar la plataforma web/móvil centralizada de Innovadataco para la gestión de activos intelectuales, operativos y de configuración, bajo una interfaz minimalista de alto rendimiento.

### 2. Alcance del MVP (Módulos)
- **Investigación:** Subida, búsqueda y resumen de documentos (IA-Powered).
- **Proyectos:** Kanban agentic que integra tareas de agentes y humanos.
- **Base Oficial:** Repositorio de documentos maestros (Metodologías, Legal).
- **Configuración:** Gestión de llaves API y perfiles de agentes.

### 3. Criterios de Aceptación (Givne/When/Then)
- **Escenario 1: Dashboard Multiplataforma**
  - **Given:** Un usuario accede desde móvil o desktop.
  - **When:** Carga la URL oficial.
  - **Then:** La UI se adapta (Responsive) manteniendo la estética minimalista negra/cian sin pérdida de funcionalidad.

- **Escenario 2: Notificación Real-Time**
  - **Given:** Un agente termina una tarea en el módulo Proyectos.
  - **When:** El estado cambia a "Terminado".
  - **Then:** El dashboard del usuario se actualiza instantáneamente vía WebSockets.

### 4. Stack Tecnológico (Plan Preliminar)
- **UI:** Next.js + Tailwind + Framer Motion (para animaciones fluidas).
- **Backend:** Node.js + Supabase (Auth, Real-time DB, Storage).
- **IA Integration:** Hermes API Gateway + Pinecone (Vector Search).
