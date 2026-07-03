# Arquitectura de Sistema y Diseño Gráfico
## Proyecto: 001-2026-INNOVADATACO

### 1. Arquitectura Técnica (API-First)
Se utilizará una **Arquitectura de Micro-Frontends** envuelta en un monorepo, con un **API Gateway** centralizado para agentes.

- **Frontend Core:** Next.js (SSG/SSR) + Tailwind.
- **Backend/API:** Next.js Request Handlers (API Routes) protegidas por `Bearer Tokens` específicos para Agentes.
- **Data Layer:** Supabase (Postgres) + Edge Functions para lógica pesada de investigación.
- **Orquestación:** Webhooks salientes para que la plataforma "hable" de vuelta a los bots (Telegram/Odin).

### 2. Endpoints Críticos para Agentes (MVP)
- `POST /api/v1/projects`: Crear/Actualizar proyectos (Ej: Proyecto SET Sincelejo).
- `POST /api/v1/research`: Disparar procesos de investigación profunda.
- `GET /api/v1/knowledge`: Consulta semántica a la base oficial.
- `PATCH /api/v1/tasks/:id`: Actualizar estado de una tarea desde el agente.

### 2. Concepto de Diseño Gráfico (Minimalismo Técnico)

**A. Paleta de Colores (Dark Mode First):**
- **Background:** `#050505` (Negro Carbono)
- **Cards/Surface:** `#121212` (Gris profundo con opacidad 0.8)
- **Primary/Action:** `#00F0FF` (Cian Eléctrico - Pantalla Neon)
- **Secondary:** `#FFB800` (Ámbar - Solo para alertas/notificaciones)
- **Text Primary:** `#F5F5F5` (Blanco Humo)

**B. Tipografía:**
- **Titulares:** `Geist Sans` (Bold/Black) para impacto y modernidad.
- **Cuerpo y Código:** `Geist Mono` para resaltar ese look "técnico/agente".

**C. Componentes Clave:**
1. **The Command Bar:** Un input central flotante (Glassmorphism) para interactuar con la plataforma vía comandos de texto.
2. **Tableros Líquidos:** Columnas de Kanban sin bordes, separadas solo por sutiles gradientes de luz.
3. **Módulo de Investigación:** Visualización de documentos en modo "Reader" tipo Notion, enfocado en el contenido.

**D. Móvil:**
- Interfaz basada en **Gestos y Bottom Sheets**. Los menús no se clickean, se deslizan desde la parte inferior para facilidad de uso con una sola mano.
