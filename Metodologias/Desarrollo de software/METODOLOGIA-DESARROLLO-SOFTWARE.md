# Metodología Técnica de Desarrollo de Software con Agentes IA
## Innovadataco — Documento Maestro (Hermes Agent)

**Versión:** 3.0 — ESTÁNDAR SUPREMO Y OBLIGATORIO
**Estado:** MAESTRO / OPERATIVO
**Rol:** Guía de ejecución para Agentes IA y Auditoría Humana.

---

## 🕒 Historial de Versiones

| Versión | Fecha | Autor | Cambios Principales |
| :--- | :--- | :--- | :--- |
| 1.0 | 2026-06-15 | Innovadataco | Versión inicial de Spec-Driven Development. |
| 2.0 | 2026-06-30 | Innovadataco | Integración obligatoria con Hermes Agent y Quality Gates. |
| 3.0 | 2026-07-02 | Hermes Agent (Odin) | **Nivel DIOS:** Validación de entorno (Paso 0), escaneo de secretos local, nomenclatura semántica de proyectos, matriz de riesgo con escalada de alerta por Gateway y registro de deuda técnica. |

---

## ⚠️ Instrucción Pre-Vuelo para el Agente IA

Antes de tocar una sola tecla, el Agente **debe confirmar cumplimiento** de estos 3 pilares:
1. **Validación de Identidad:** ¿Estoy en el perfil correcto y tengo acceso al repo `productos`?
2. **Contexto de Proyecto:** Ubicar o crear la carpeta `NNN-AAAA-CLIENTE-PROYECTO-INNOVADATACO`.
3. **Escaneo de Seguridad Local:** Prohibido subir commits sin pasar el scanner de secretos local.

---

## 1. El Ciclo de Vida "Nivel Dios" (6 Pasos)

```
0. Env Check ➔ 1. Spec ➔ 2. Plan/Tasks ➔ 3. Test-Plan ➔ 4. TDD/Secret-Scan ➔ 5. PR/Merge
```

### Paso 0: Validación de Entorno (`env-check.md`)
Antes de proponer soluciones, el agente verifica la viabilidad técnica:
- Consultar versiones de runtime (Python, Node, Go, etc.) y dependencias críticas.
- Validar conectividad con APIs externas necesarias.
- **Resultado:** Un breve archivo en `specs/<feature>/env-check.md` confirmando que el stack actual soporta el cambio solicitado sin causar conflictos de versiones.

### Paso 1: Especificación (`spec.md`)
- Definir comportamiento esperado (Given/When/Then).
- **Cero mención de código.** Solo lógica de negocio y criterios de aceptación (AC).

### Paso 2: Plan Técnico y División de Tareas (`plan.md`, `tasks.md`)
- Diseño de arquitectura.
- División en tareas atómicas (commits de máximo 400 líneas).
- **Novedad:** Identificación de **Deuda Técnica Preexistente** que será afectada.

### Paso 3: Plan de Pruebas Bloqueado (`test-plan.md`)
- Definición de casos de prueba para cada AC del Paso 1.
- **Gate Inamovible:** Este archivo se commitea ANTES de la implementación. El éxito no se negocia durante el codeo.

### Paso 4: Implementación y "Pre-Push Security"
- Ciclo TDD: **Rojo ➔ Verde ➔ Refactor**.
- **Regla de Seguridad Crítica:** Antes de cada `git push`, el agente ejecutará un comando de escaneo de secretos (ej. `trufflehog` o `gitleaks`) en el diff local. Si hay un hallazgo, el push se cancela y se debe rotar la clave si fue expuesta localmente.

### Paso 5: Pull Request, Decision Log y Deuda
- El PR debe incluir el **Decision Log** (por qué se eligió A sobre B).
- **Registro de Deuda:** Documentar en el PR cualquier "atajo" o mejora pendiente para evitar que se olvide.

---

## 2. Estructura de Repositorios y Nomenclatura Semántica

### 2.1 Repositorio `Innovadataco/productos`
Cada proyecto debe seguir el formato de carpeta de **Alta Visibilidad**:

`NNN-AAAA-CLIENTE-SISTEMA-INNOVADATACO`

*   `NNN`: Correlativo global de 3 dígitos (ej: `042`).
*   `AAAA`: Año de creación.
*   `CLIENTE`: El nombre del cliente (ej: `BANCOLOMBIA`).
*   `SISTEMA`: El nombre del producto (ej: `WALLET`).
*   **Ejemplo Final:** `042-2026-BANCOLOMBIA-WALLET-INNOVADATACO`

### 2.2 Estructura Interna del Proyecto
```
042-2026-BANCOLOMBIA-WALLET-INNOVADATACO/
├── specs/
│   └── <nombre-feature>/
│       ├── env-check.md    (Paso 0)
│       ├── spec.md         (Paso 1)
│       ├── plan.md         (Paso 2)
│       ├── tasks.md        (Paso 2)
│       └── test-plan.md    (Paso 3)
├── src/
├── tests/
└── tech-debt.md            (Historial acumulado de deuda técnica del proyecto)
```

---

## 3. Flujo de Git y Conventional Commits

*   **Estrategia:** Trunk-based development con ramas de vida corta.
*   **Commits Atómicos:** Prohibidos los mensajes tipo "fix", "wip", "update".
*   **Formato Obligatorio:** `type(scope): description`.
*   **Merge:** *Squash and Merge* únicamente cuando el CI esté en verde.

---

## 4. Matriz de Riesgo y Escalada de Aprobación

| Nivel | Descripción | Requisito de Merge |
| :--- | :--- | :--- |
| **BAJO** | Docs, estilos, tests existentes. | CI Verde. |
| **MEDIO** | Lógica de negocio, nuevos módulos. | CI Verde + 1 Agente Revisor. |
| **ALTO** | **Auth, Secretos, Infra, Base de Datos.** | CI Verde + 1 Humano (Aprobación explícita). |

### El Mecanismo de Alerta (Vía Gateway)
Para cambios de **Riesgo ALTO**, el agente no se quedará esperando en silencio:
1. El agente generará un mensaje automático en el canal de administración (Telegram/Slack).
2. "🚨 **SOLICITUD DE REVISIÓN CRÍTICA:** PR #[Número] en [Proyecto]. Requiere aprobación humana para procededer."
3. Si no hay respuesta en el tiempo definido por el PM, el agente debe pausar la tarea y notificar el bloqueo.

---

## 5. Checklist de Oro del Agente (Obligatorio en cada turno)

- [ ] **Entorno:** ¿Hice el `env-check` para no romper dependencias?
- [ ] **Estructura:** ¿La carpeta usa el formato `NNN-AAAA-CLIENTE-SISTEMA-INNOVADATACO`?
- [ ] **Secuencia:** ¿Hice commit del `test-plan.md` ANTES de la implementación?
- [ ] **Seguridad:** ¿Ejecuté el escaneo de secretos local antes de este `push`?
- [ ] **Commits:** ¿Mis mensajes de commit son semánticos y atómicos?
- [ ] **Riesgo:** ¿Identifiqué correctamente si este cambio requiere aprobación humana (Riesgo Alto)?
- [ ] **Deuda:** ¿Actualicé el archivo `tech-debt.md` con las optimizaciones pendientes?

---

## 6. Resumen Ejecutivo
> **"Validar entorno, asegurar secretos localmente antes de subir, definir el éxito antes de codear y alertar proactivamente ante cambios críticos en una estructura de carpetas semántica e impecable."**
