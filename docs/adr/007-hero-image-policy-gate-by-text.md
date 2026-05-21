# ADR-007: Hero image policy — gate-by-text con aprobación humana via Asana

**Status**: Approved
**Fecha**: 2026-05-21
**Drivers**: Joel Sierra (review editorial visual), Pablo Waimann (project lead)
**Relacionado**: ADR-001 (arquitectura skill `/relevo`), `feedback-asana-hitl-exclusive` (memoria de Pablo), `feedback-wp-mcp-image-upload-path` (memoria sobre wp_upload.py)

---

## 1. Contexto

El 21/05/2026 Joel Sierra entregó feedback visual (Asana task `1214965670270081`, "WP visual check") sobre los 3 artículos del batch 2 ya publicados como draft en WordPress (`relevo-com.sub.plus`). Entre otros temas (white space desktop, h1 muy largo, tabla art 2 sin estilizar — cubiertos fuera de este ADR), Joel marcó que **las hero images no ilustraban adecuadamente el concepto de los artículos 1 y 2**:

> *Art 1 "Fuera de juego en fútbol"* — hero image: foto de córner al atardecer. No ilustra fuera de juego. Joel marcó con X roja sobre la imagen.
>
> *Arts 1, 2, 3* — Joel pidió un "elemento visual interno" adicional (callout, pull-quote, diagrama) además de la hero.

### 1.1 El problema de fondo

El pipeline actual genera la hero image con **gpt-image-2 (vía LiteLLM iGaming)** al final del ciclo de redacción ([[project_image_gen_working]]). El prompt de la imagen lo construye el propio modelo a partir del título y el primer párrafo del artículo, sin revisión humana.

Las consecuencias visibles en batch 2:

1. **Imágenes que no ilustran el concepto central** del artículo (caso art 1).
2. **Tokens y minutos de cómputo desperdiciados** en imágenes que terminan siendo descartadas o reemplazadas a mano.
3. **Ausencia de auditoría editorial** sobre la decisión visual — Joel solo ve el resultado final, no la intención.
4. **Inconsistencia visual** entre artículos: cada uno genera su prompt sin un criterio editorial común.

### 1.2 La sugerencia inicial de Pablo

En la conversación del 21/05, Pablo propuso:

> *"basado en el artículo propongas una imagen escrita en texto (osea su descripción) y el auditor humano confirme o cancele"*

Y reforzó:

> *"hay que agregar una alerta de Asana de aprobación, recorda que usamos Asana exclusivamente para Human in the loop, asi que este caso es válido"*

Esto es consistente con la decisión durable de `feedback-asana-hitl-exclusive` — Asana como canal único para todo gate HITL del pipeline.

---

## 2. Decisión

Implementar un **gate-by-text de dos pasos** entre el cierre de redacción del artículo y la publicación final del draft en WP, con aprobación humana materializada como subtasks Asana:

```
[Article passes self-check]
            │
            ▼
┌─────────────────────────────────────────────────────────────┐
│  Pipeline stage: propose_image_prompt                       │
│  ──────────────────────────────────────────────             │
│  Modelo genera 1 prompt textual en castellano (1-3 frases): │
│  - Qué se debe ver en la imagen                             │
│  - Por qué ilustra el concepto central del artículo         │
│  - Estilo visual sugerido (foto realista / ilustración /…)  │
│                                                             │
│  Escribe el prompt al Excel row col `image_prompt`          │
│  Crea Asana subtask 1: [Image - Prompt review] {title}      │
└─────────────────────────────────────────────────────────────┘
            │
            ▼
       Reviewer humano (Joel) en Asana
       - Aprueba (complete) → sigue
       - Edita prompt en Asana o Excel → sigue con nuevo prompt
       - Rechaza (comment con razón) → pipeline pausa, escala a Pablo
            │
            ▼
┌─────────────────────────────────────────────────────────────┐
│  Pipeline stage: generate_image                             │
│  ──────────────────────────────────────────────             │
│  Lee `image_prompt` aprobado del Excel                      │
│  Invoca gpt-image-2 via scripts/imagen/                     │
│  Sube via scripts/wp_upload.py al WP media library          │
│  Escribe `image_url` al Excel row                           │
│  Crea Asana subtask 2: [Image - Visual review] {title}      │
└─────────────────────────────────────────────────────────────┘
            │
            ▼
       Reviewer humano (Joel) en Asana
       - Aprueba (complete) → setea featured image en draft WP
       - Rechaza (comment con razón) → pipeline pausa
         · Opción a) regenera con mismo prompt
         · Opción b) vuelve a stage propose_image_prompt
            │
            ▼
       Draft listo para publish
```

### 2.1 Cambios al schema de Excel

Inventory v2 actualmente tiene 22 columnas ([[reference_inventory_excel_v2]]). Este ADR introduce **5 columnas nuevas** (schema crece a 27 cols):

| Col | Tipo | Propósito |
|---|---|---|
| `image_prompt` | text | Prompt textual propuesto / aprobado para gpt-image-2. Editable por reviewer humano antes de invocar la generación. |
| `image_status` | enum | FSM del proceso: `pending_prompt_review` → `prompt_approved` → `image_pending_visual_review` → `image_approved` → `rejected` |
| `image_url` | text | URL del media item subido a WP tras generación. Vacío hasta que pasa visual review. |
| `image_asana_prompt_gid` | text | GID de la subtask Asana del primer paso (prompt review) — traceability |
| `image_asana_visual_gid` | text | GID de la subtask Asana del segundo paso (visual review) — traceability |

La migración del schema se hace via `sharepoint_msal.py` con un script de one-shot que añade las 5 columnas a Table1 (procedimiento documentado en `lib/inventory.md`).

### 2.2 Cambios al pipeline.md del skill `/relevo`

El skill agrega 4 stages nuevos entre `self_check_pass` y `wp_publish_draft`:

| Stage nuevo | Función |
|---|---|
| `propose_image_prompt` | Modelo genera el prompt textual. Escribe a Excel + crea subtask Asana 1. |
| `wait_image_prompt_approval` | Pipeline pausa hasta que `image_status` pasa a `prompt_approved` (mecanismo: Excel polling o webhook Asana). |
| `generate_image` | Invoca gpt-image-2 con el prompt aprobado, sube via wp_upload.py, crea subtask Asana 2. |
| `wait_image_visual_approval` | Pipeline pausa hasta `image_status` = `image_approved`. |

Los stages `wait_*` no bloquean el resto del pipeline — `wp_publish_draft` espera, pero stages independientes (FAQs, schema, internal linking) pueden ejecutar en paralelo si la cadencia lo justifica.

### 2.3 Formato de las subtasks Asana

**Subtask 1 (prompt review)** — `[Image - Prompt review] {article_title}`:
- Assignee: Joel Sierra (default; configurable)
- Due date: same-day o next-day según volumen
- Descripción densa:
  ```
  Artículo: {title}
  Sport / Categoría: {sport} / {category}
  Link al draft (relevo-com.sub.plus): {draft_url}
  Excel row: {row_id}

  PROMPT PROPUESTO:
  > {image_prompt}

  POR QUÉ ESTE PROMPT (rationale del modelo, 1-2 frases):
  > {rationale}

  ─────────────────
  Para aprobar: marca la subtask como complete.
  Para editar el prompt: edítalo en la celda image_prompt del Excel
  (file: Relevo - Content - Article production DB.xlsx) y luego
  marca la subtask como complete.
  Para rechazar: agrega un comment explicando por qué y NO marques
  como complete — el pipeline pausará y escalará a Pablo.
  ```

**Subtask 2 (visual review)** — `[Image - Visual review] {article_title}`:
- Assignee: Joel Sierra
- Descripción densa:
  ```
  Artículo: {title}
  Hero image generada: {image_url}
  Prompt usado: {image_prompt}
  Link al draft (relevo-com.sub.plus): {draft_url}

  ─────────────────
  Para aprobar: marca la subtask como complete; la imagen quedará
  como featured image del draft.
  Para regenerar con el mismo prompt: agrega un comment con la palabra
  "regenerate" — el pipeline volverá a generate_image.
  Para reescribir el prompt: agrega un comment con "rewrite-prompt"
  y el pipeline volverá al stage propose_image_prompt.
  ```

### 2.4 Lo que NO está en scope de este ADR

- **Elementos visuales internos** (callout boxes, pull-quotes, diagramas) — Joel los pidió en el mismo feedback pero requieren decisión separada sobre style guide + theme CSS. Punto deferido — capturado en backlog como `EDIT-12 (Internal visual elements policy)`.
- **Imágenes secundarias / dentro del cuerpo del artículo** — este ADR cubre la hero image. Las imágenes intra-cuerpo (si las hay) reciben una decisión separada cuando se aborden.
- **Auto-approve de prompts repetitivos** — para hub bios o artículos de la misma categoría con prompt similar, se podría pre-aprobar un patrón. Optimización para v2.
- **Implementación real del pipeline** — este ADR define la política. La implementación (los 4 stages nuevos, la migración de schema, los helpers de subtask creation) entra en `EDIT-13 (Implement hero image gate)` con due la próxima semana.

---

## 3. Alternativas consideradas

### 3.1 Sin gate — confiar en el prompt automático del modelo (status quo)

**Rechazada.** Es el escenario que produjo los fallos del batch 2 (art 1 hero image que no ilustra el concepto). No tiene auditoría editorial. Cero traceability.

### 3.2 Gate POST-generación únicamente (solo subtask de visual review)

**Rechazada.** Ahorra una subtask pero desperdicia el cómputo de la imagen rechazada. gpt-image-2 cuesta tiempo (~25s) y tokens; gastar eso para que Joel rechace al final del flujo es ineficiente. El gate-by-text previo cuesta 0 tokens de imagen y es 5x más rápido de reviewar (texto vs imagen renderizada).

### 3.3 Gate POST-generación con feedback loop solo si rechazo

**Considerada pero rechazada.** Optimiza cuando la imagen sale bien, pero penaliza cuando sale mal (costo de regen completo). El gate-by-text se elimina el costo de regen para los casos de prompt malo.

### 3.4 Email / Slack / Excel-only — sin Asana

**Rechazada.** Conflictúa con `feedback-asana-hitl-exclusive` — Asana es el canal único para HITL approvals en este pipeline. Email y Slack no dan traceability vinculada al artículo. Comentarios en Excel pierden el evento de aprobación (no hay timestamp claro de "approved at"). Asana subtasks tienen estado de completion explícito + assignee + due date.

### 3.5 Auto-generate múltiples prompts y dejar al reviewer elegir

**Considerada parcialmente.** Genera 2-3 prompts diferentes para que Joel elija el mejor. Aporta variedad pero (i) infla la descripción de la subtask, (ii) Joel tendría que elegir 50 veces por día (cognitive load), (iii) la opción de editar el prompt seleccionado ya cubre la necesidad de variabilidad. Decisión: single prompt + edición libre. Reabrible si Joel se queja de la limitación.

### 3.6 Modelo más fuerte (Sonnet 4.6 / Opus 4.7) para generar el prompt sin gate

**Rechazada por separado.** El feedback de Joel no es que el modelo genera prompts malos; es que ningún reviewer humano los valida. Aún con prompts perfectos, la auditoría sigue siendo deseable porque Joel es quien define la línea editorial visual. Un mejor modelo ahorra rejecciones pero no elimina la necesidad del gate.

---

## 4. Impacto y consecuencias

### 4.1 Impacto positivo esperado

- **0 imágenes desperdiciadas** por hero image que no ilustra el concepto (caso art 1 del batch 2 se elimina).
- **Joel queda en el loop** con cognitive load mínimo (lee 1-3 frases de texto en lugar de revisar imagen renderizada).
- **Trazabilidad completa**: cada artículo tiene 2 subtasks Asana con timestamps de prompt-approved y visual-approved.
- **Auditoría editorial**: la línea visual del sitio queda documentada implícitamente en las decisiones de aprobación / rechazo / edición de prompts.
- **Pre-launch (01/06)**: cero hero images embarazosas en el sitio público al lanzamiento.

### 4.2 Costo

- **Latencia adicional por artículo**: 2 humanos-in-the-loop agregan tiempo entre redacción y publish. Mitigación: stages independientes pueden ejecutar en paralelo; el bottleneck queda en `wp_publish_draft` que es donde la imagen es necesaria.
- **Volumen para Joel**: 2 subtasks adicionales × N artículos por día. A 50 artículos/día (capacidad teórica del pipeline), son 100 subtasks Asana adicionales para Joel. Mitigación: prompts pre-aprobables por categoría en una v2 si el volumen lo justifica; por ahora 2 subtasks por artículo es manejable y Joel ya está pre-aprobado para el rol QA.
- **Implementación**: 4 stages nuevos al pipeline + 5 columnas nuevas a Excel + helpers de creación de subtask Asana. Capturado como `EDIT-13` con due la próxima semana.
- **Riesgo de pausa indefinida**: si Joel no responde, el pipeline queda colgado. Mitigación: SLA implícito 24h, después de 24h sin respuesta el pipeline crea subtask escalada a Pablo. Detalle de implementación en `EDIT-13`.

### 4.3 Validación

- **Smoke test** sobre 1 artículo nuevo bajo la nueva política — verifica que el flujo end-to-end funciona: prompt propuesto → subtask Asana → aprobación → image gen → subtask visual → aprobación → featured image en draft. Pre-requisito antes del batch 3 si la implementación llega a tiempo.
- **Métrica de éxito a 1 semana**: % de prompts aprobados al primer intento (objetivo: ≥ 70%); % de imágenes aprobadas al primer intento tras prompt aprobado (objetivo: ≥ 80%). Si menos, ajustar.

---

## 5. Decisiones derivadas

- **Backlog `EDIT-12 (Internal visual elements policy)`** (nuevo): callouts, pull-quotes, diagramas. Decisión separada; involve style_guide.md + theme CSS.
- **Backlog `EDIT-13 (Implement hero image gate)`** (nuevo): los 4 stages al pipeline + 5 cols al Excel + helpers Asana. Due semana próxima.
- **Backlog `EDIT-14 (Migrate hero image gate to batch 3)`** (nuevo): aplicar el flow al batch 3 regenerado en `EDIT-10` antes de publicar.
- **Memoria persistente**: `project_relevo_hero_image_policy_pending` se creará para trackear el estado de la implementación cross-session.

---

## 6. Referencias

- `prompts/style_guide.md` v1.1.2 — la guía editorial que el prompt visual debe respetar implícitamente.
- `scripts/imagen/` — implementación de gpt-image-2 vía LiteLLM iGaming ([[project_image_gen_working]]).
- `scripts/wp_upload.py` — upload helper para WordPress media library ([[feedback_wp_mcp_image_upload_path]]).
- `scripts/sharepoint_msal.py` — read/write del Excel inventory.
- `scripts/asana_notify.py` — helper de creación de subtasks Asana ([[reference_asana_notify_helper]]).
- Asana task `1214965670270081` — feedback original de Joel con screenshots.
- Memoria persistente `feedback-asana-hitl-exclusive` — canon de la política HITL.
- ADR-001 — arquitectura del skill.
