# Dashboard estadístico — mockup + data audit (borrador)

> Estado: armado por Claude el 22/5 EOD. Mockup + auditoría de qué tenemos loggeado hoy + qué falta instrumentar.

## Auditoría: qué loggeamos hoy

Fuente de verdad: `lib/logging.md` del skill `/relevo`.

**Eventos canónicos que el pipeline emite por cada artículo:**

| Evento | Payload útil para dashboard |
|---|---|
| `run_started` | inputs del editor (deporte, categoría) |
| `inventory_locked` | row_id |
| `research_started` / `research_completed` | duración stage (calc por diff de timestamps) |
| `draft_started` / `draft_completed` | word_count, model, duration_ms |
| `critique_started` / `critique_completed` | **score** |
| `revise_started` / `revise_completed` | score post-revise, # de iteraciones |
| `factcheck_started` / `factcheck_completed` | duración |
| `image_started` / `image_completed` | file path, duración |
| `wp_upload_started` / `wp_upload_completed` | wp_post_id |
| `asana_subtask_created` | asana_gid, reviewer |
| `run_completed` | **duration total**, final_score, word_count |
| `stage_retry` | stage, attempt, error |
| `dead_letter` | stage_failed, error, attempts |

**Schema de Table2 (Excel logs)**: timestamp · run_id · event · level · deporte · categoria · row_id · payload (JSON).

## KPIs que el dashboard puede mostrar hoy (sin instrumentar nada nuevo)

| KPI | Fuente | Cómo se calcula |
|---|---|---|
| **# artículos generados** | count(`run_completed`) | Filtros por fecha, deporte, categoría |
| **# en dead-letter** | count(`dead_letter`) | Indicador de salud del pipeline |
| **Score promedio (calidad)** | avg(payload.final_score from `run_completed`) | Línea de tendencia + breakdown por categoría |
| **Score distribution** | histograma de scores | Cuántos en 90+, 80-89, 70-79, <70 |
| **Tiempo total por artículo** | avg(payload.duration_seconds from `run_completed`) | P50, P90, P99 |
| **Tiempo por stage** | diff entre `{stage}_started` y `{stage}_completed` | Stacked bar: research / draft / critique / revise / factcheck / image / upload |
| **Word count** | avg(payload.word_count from `draft_completed`) | Distribución + outliers |
| **Iteraciones de revise** | count(`revise_completed`) por run_id | Indicador de "primera-pasada-OK" vs "necesitó retry" |
| **Throughput** | runs por día/semana | Para FIFA WC target de 50/día |
| **Cobertura por sport** | count agrupado por `deporte` | ¿Estamos cubriendo los 24 deportes? |
| **Cobertura por categoría** | count agrupado por `categoria` | Rules / Tactics / Origins balance |
| **Tasa de stage_retry** | count(`stage_retry`) / count(`run_started`) | Indicador de estabilidad del pipeline |

## KPIs que requieren instrumentar (gap)

| KPI | Por qué no se puede hoy | Cómo se llena |
|---|---|---|
| **Tokens consumidos** | `draft_completed` no loggea tokens, solo word_count | Agregar `tokens_input`/`tokens_output` al payload del stage cuando LiteLLM devuelva los counters |
| **Costo USD por artículo** | Depende de tokens × tarifa por modelo | Calcular post-hoc desde tokens + tabla de tarifas (claude-opus-4-7, gpt-image-2). Se puede agregar al `run_completed` |
| **Bolds coverage** (style guide v1.1.3) | El score actual es global, no hay sub-score de bolds | Agregar payload extendido a `critique_completed` con sub-scores |
| **Caracteres (charset)** | No se loggea explícitamente, derivable de word_count × ~5 | Trivial: agregar al `draft_completed` payload |
| **Tiempo de review humana** | El gate Asana no devuelve outcome al pipeline | Cuando se cierre la subtask en Asana, agregar webhook que loggee `human_review_completed` con duration |
| **Tasa pass / request-changes / reject del review** | Asana no notifica al pipeline | Mismo webhook + parsear outcome de la subtask |

## Mockup ASCII del dashboard

Sin instrumentar tokens / costo (Phase 4). Con lo que ya tenemos hoy:

```
╔══════════════════════════════════════════════════════════════════════════╗
║                      RELEVO PIPELINE DASHBOARD                           ║
║                          Última semana · 19–26 May                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                          ║
║   📊 VOLUMEN                              📈 CALIDAD                     ║
║   ┌─────────────────────┐                 ┌─────────────────────────┐   ║
║   │  Generados:    11   │                 │  Score promedio: 92.4   │   ║
║   │  Aprobados:     2   │                 │  ┌─┐                    │   ║
║   │  En review:     9   │                 │  │█│ 90+:   8 (72%)     │   ║
║   │  Dead-letter:   0   │                 │  │█│ 80-89: 3 (28%)     │   ║
║   │  Publicados:    0   │                 │  │ │ 70-79: 0           │   ║
║   └─────────────────────┘                 │  │ │ <70:   0           │   ║
║                                           └─────────────────────────┘   ║
║                                                                          ║
║   ⏱️  TIEMPO POR ARTÍCULO                                               ║
║   ┌────────────────────────────────────────────────────────────────┐    ║
║   │  P50: 9.4 min       P90: 11.2 min       P99: 14.8 min          │    ║
║   │                                                                │    ║
║   │  Breakdown (avg seg):                                          │    ║
║   │   research      ▓▓▓▓▓▓▓ 32                                     │    ║
║   │   draft         ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ 124                  │    ║
║   │   critique      ▓▓▓▓▓ 28                                       │    ║
║   │   revise        ▓▓▓▓▓▓▓ 38 (38% de runs necesitan revise)      │    ║
║   │   factcheck     ▓▓▓▓▓▓▓▓▓▓ 56                                  │    ║
║   │   image         ▓▓▓▓▓▓ 27                                      │    ║
║   │   upload        ▓▓ 10                                          │    ║
║   └────────────────────────────────────────────────────────────────┘    ║
║                                                                          ║
║   🏆 COBERTURA POR DEPORTE                                              ║
║   ┌────────────────────────────────────────────────────────────────┐    ║
║   │  Fútbol     ▓▓▓▓▓▓▓▓▓ 4   Tenis      ▓▓▓ 1   F1         ▓ 0   │    ║
║   │  Baloncesto ▓▓▓ 1         Boxeo      ▓▓▓ 1   MMA        ▓▓▓ 1 │    ║
║   │  Padel      ▓▓▓ 1         Motocicl.  ▓▓▓ 1   Ciclismo   ▓▓ 1  │    ║
║   │                                                                │    ║
║   │  Cobertura: 9/26 deportes (35%) — gap mayor: olímpicos +rugby  │    ║
║   └────────────────────────────────────────────────────────────────┘    ║
║                                                                          ║
║   📝 COBERTURA POR CATEGORÍA            🚨 SALUD DEL PIPELINE           ║
║   ┌────────────────────────┐            ┌────────────────────────────┐  ║
║   │  Rules    ▓▓▓▓▓▓ 7     │            │  Stage retries:  3 / 11    │  ║
║   │  Tactics  ▓▓▓ 3        │            │  Dead-letters:   0 / 11    │  ║
║   │  Origins  ▓ 1          │            │  Success rate:   100%      │  ║
║   └────────────────────────┘            └────────────────────────────┘  ║
║                                                                          ║
║   🎯 PROYECCIÓN FIFA WC (target: 50 art/día desde 11/6)                  ║
║   ┌────────────────────────────────────────────────────────────────┐    ║
║   │  Velocidad actual:    ~1.5 art/día                             │    ║
║   │  Necesario:           50 art/día (33x scale-up)                │    ║
║   │  Gap:                 estabilizar review pool + automatizar    │    ║
║   │                       HITL #2 (gate hero) en Phase 4           │    ║
║   └────────────────────────────────────────────────────────────────┘    ║
║                                                                          ║
╚══════════════════════════════════════════════════════════════════════════╝

   * Números son ilustrativos del batch 3. Para data real conectar al Excel logs
     (Table2 via sharepoint_msal.py) o al JSONL local (logs/runs.jsonl).
```

## Decisiones de diseño que el dashboard debe respetar

**Para audiencia editorial (no-técnica):**

1. **No mostrar tokens / costo** en la vista default. Es información que confunde a content writers. Dejarlo en una "vista admin" para JPW.
2. **Prioridad arriba: volumen + calidad + tiempo**. Esos 3 son el lenguaje común de editorial.
3. **Mostrar la proyección hacia FIFA WC**. Es el contexto motivacional del proyecto.
4. **Cobertura por deporte/categoría con visualización clara**. Equipo necesita ver "dónde estamos cubriendo y dónde falta".
5. **Salud del pipeline (retries, dead-letters)** explicada en lenguaje natural: "¿Cuántas veces el pipeline tuvo que intentar de nuevo?" en vez de "stage_retry count".

## Roadmap del dashboard (qué hacer cuando)

| Fase | Qué se entrega | Cuándo |
|---|---|---|
| **Demo lunes** | Mockup estático con datos del batch 3 — screenshots/slide | 26/5 |
| **Phase 4 ítem D-01** | Dashboard live conectado al JSONL local + Excel logs · sin tokens · audiencia editorial | semana del 26/5 |
| **Phase 4 ítem D-02** | Agregar tokens + costo USD instrumentando LiteLLM responses | post-launch |
| **Phase 4 ítem D-03** | Webhook Asana → loggear outcome del review humano | post-launch |
| **Phase 4 ítem D-04** | Vista admin con sub-scores (bolds coverage, factuality, length compliance) | post-launch |

## Tools posibles para el dashboard live (Phase 4)

| Opción | Pros | Contras | Veredicto |
|---|---|---|---|
| **Streamlit** (Python) | Rápido de armar, JSONL → DataFrame es trivial, JPW puede operar | UI no es polished | **Recomendado** para MVP |
| **Grafana** | Pro-grade, exporta a PDF, alerting | Necesita data source plugin (Loki / Prometheus), overkill | Defer post-launch |
| **Metabase** | Bonito, SQL-friendly | Requiere DB; logs están en JSONL/Excel | Defer |
| **Notion/Airtable embed** | Cero setup técnico | Update manual, no escala | Plan B si no hay tiempo |
| **Google Sheets + Charts** | Cero setup, equipo entiende | Update semi-manual | Plan B aceptable |

**Recomendación**: Streamlit para Phase 4. Pre-demo del lunes: mockup en slide con números reales del batch 3.

## Próximos pasos concretos

1. JPW confirma los KPIs prioritarios (los 5 cuadrantes del mockup arriba o ajustar).
2. JPW decide: dashboard live para el lunes o mockup screenshot.
3. Si live: armar Streamlit ~3 horas el sábado o lunes mañana, conectar al JSONL local.
4. Si mockup: completar con números reales del batch 3 leyendo `logs/runs.jsonl` y/o Excel logs.
5. Capturar para Phase 4 el gap de instrumentación (tokens, webhook Asana).
