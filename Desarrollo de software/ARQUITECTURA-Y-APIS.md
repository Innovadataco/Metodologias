# Modelo de Datos (Data Schema)
## Proyecto: 001-2026-INNOVADATACO

### 1. Entidades Principales (PostgreSQL)

#### Tabla: `proyectos`
- `id`: uuid (PK)
- `codigo`: varchar (Ej: 001-2026-SINCELEJO)
- `nombre`: text
- `cliente`: text
- `estado`: enum (active, paused, completed)
- `created_at`: timestamp
- `updated_at`: timestamp

#### Tabla: `investigaciones`
- `id`: uuid (PK)
- `user_id`: uuid (FK)
- `agente_id`: text (ID del agente que disparó el proceso)
- `tema`: text
- `brief`: text (Resumen generado por IA)
- `status`: enum (processing, done, error)
- `metadata`: jsonb (Links de fuentes, fechas, versión de la metodología aplicada)

#### Tabla: `tareas_agentes`
- `id`: uuid (PK)
- `proyecto_id`: uuid (FK)
- `descripcion`: text
- `status`: enum (pending, running, success, fail)
- `session_id`: text (Link a la sesión de Hermes/Odin para trazabilidad)

#### Tabla: `configuraciones_maestras`
- `clave`: varchar (PK)
- `valor`: text
- `riesgo`: enum (low, medium, high) -- Determina si se puede modificar vía API de Agente o requiere humano.

### 2. Flujo de Activación (Ejemplo: SINNIT)
1. **Disparador:** Odin recibe comando en Telegram: "/investigar SINNIT".
2. **Acción:** Odin hace un `POST /api/v1/research { topic: "SINNIT" }`.
3. **Backend:** La plataforma inserta en `investigaciones` con estado `processing`.
4. **Respuesta:** El backend confirma a Odin recibo de tarea.
5. **Proceso:** Un Worker o Edge Function de la plataforma ejecuta la lógica profunda.
6. **Cierre:** Al terminar, la plataforma hace un Webhook hacia el Gateway de Odin con el resultado final.
