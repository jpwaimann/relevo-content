# Demo lunes 26/5 — Scope (borrador)

> **Audiencia confirmada**: equipo Relevo editorial — content writers, editorial leader, SEO, off-page. **Cero perfil técnico.**
> **Estado**: borrador armado por Claude el 22/5 EOD mientras JPW estaba afuera. Pendiente review + ajustes de JPW.

## Objetivo de la demo

**Mostrar al equipo editorial qué va a cambiar en su día a día** cuando el pipeline `/relevo` esté en producción durante FIFA WC 2026. No es una demo técnica — es una conversación de equipo sobre cómo se trabaja distinto.

## Premisa narrativa: "antes vs ahora"

| | **Antes** (modo manual) | **Ahora** (con el pipeline) |
|---|---|---|
| Generar 1 artículo | 4–6 horas humanas | 10 min de cómputo + 30 min de QA editorial |
| Investigación + fact-check | Manual, búsqueda libre | Pipeline lo hace contra reglamentos oficiales |
| Imagen hero | Buscar / pedir / diseñar | Generada por AI, gate-by-text editorial |
| Subida a WP | Manual, copy-paste | Automática, draft listo para review |
| Review editorial | Único filtro de calidad | Sigue siendo el único filtro de calidad (no cambia) |
| Asignación de revisor | Email / Slack | Asana subtask automática al pool de reviewers |
| Volumen sostenible | ~5 art/semana por writer | ~50 art/día end-to-end (target FIFA WC) |

**Key message para el equipo**: el pipeline **no reemplaza a nadie** — multiplica la cobertura editorial. Joel sigue siendo el filtro de calidad. Content writers pasan de "redactar desde cero" a "guiar y curar". SEO sigue definiendo lineamientos y midiendo. Off-page sigue su track independiente.

## Estructura propuesta (30 min total)

**Opción A: 30 min live + Q&A** (recomendado para esta audiencia)

| Minuto | Bloque | Quién habla | Qué pasa |
|---|---|---|---|
| 0–3 | Apertura + premisa | JPW | "El equipo viene generando ~5 art/semana. Para FIFA WC necesitamos 50/día. Pasamos los últimos 3 meses construyendo esto." |
| 3–8 | Tour del backlog editorial actual | JPW | Mostrar el Inventory en vivo: 175 topics, status (pending_review, pending, published). Cómo se llenó (Joel topic feedback + bulk seed). Explicar las 3 categorías: Rules, Tactics, Origins. |
| 8–18 | **Demo en vivo del pipeline** | JPW + sistema | Ejecutar `/relevo new` con un topic real. Audiencia ve: editor elige sport + categoría → AI investiga, redacta, se autoevalúa, corrige, fact-check, genera imagen → sube a WP → notifica reviewer en Asana. ~10 min real. **Plan B si falla**: pre-grabado del mismo flujo. |
| 18–23 | Recorrido del output | JPW + Joel (si está) | Abrir el draft generado en WP. Comparar con un artículo del batch 3 ya revisado. Mostrar la subtask Asana con el "quality journey". |
| 23–28 | Dashboard de métricas | JPW | Mostrar el dashboard estadístico (en construcción — ver `03_dashboard_mockup.md`). Tiempo promedio, score, volumen, costo. |
| 28–30 | Cierre + Q&A | JPW + equipo | "El lunes 26/5 arrancamos Phase 4. ¿Preguntas?" |

**Plan B si el live falla**: tener pre-grabado un screencast de la última corrida exitosa (post 21195 sirve). Tenerlo listo en un tab del browser antes de empezar.

## Lo que NO mostramos

- Código Python / Bash / CSS.
- MCPs, MSAL, Azure permissions, GitLab MRs.
- ADRs salvo referencia rápida si alguien pregunta "¿esto está documentado?" → "Sí, en GitHub. Si querés profundizar te paso el link".
- Backlog técnico (P0/P1/P2/P3) — confunde a audiencia no-técnica.

## Materiales que necesitamos antes del lunes

| Material | Estado | Quién |
|---|---|---|
| Pre-grabado del pipeline (10 min) como plan B del live | **Pendiente** | JPW. Sugerencia: grabar el lunes a la mañana con un topic Rules de Fútbol. |
| Slides de apertura (3-4 slides: premisa, objetivo, equipo) | **Pendiente** | JPW |
| Dashboard funcional o mockup en pantalla | **En boceto** | Ver `03_dashboard_mockup.md`. Si para el lunes no está conectado a data live, usar el mockup con números del batch 3. |
| Lista pre-curada de 3 topics buenos para usar en el live | **Pendiente** | Sugerencia: 1 Rules sencillo (Fútbol/penalti), 1 Tactics medio (Tenis/saque), 1 Origins (Tenis/Alcaraz). Elegir al inicio cuál se usa. |
| Sala / herramienta de videollamada confirmada | **Pendiente** | JPW |

## Decisiones que JPW debe tomar antes del lunes

1. ¿La demo es 100% en vivo, o slides + live drill-down? (Recomiendo live + pre-grabado de fallback.)
2. ¿Joel asiste? (Si sí, agrega valor en el bloque 18–23 al hacer el "compare con un artículo revisado".)
3. ¿El equipo va a poder ver el dashboard en vivo o mockup? Depende de si tenemos tiempo lunes mañana de conectar el dashboard a data real.
4. ¿Qué topic se usa para el live? Definir lunes a la mañana basado en estado del Inventory.
