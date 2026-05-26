# Lib: inventory I/O

Cómo el skill lee y escribe el Excel inventory que vive en SharePoint. **Single source of truth** desde 18/5/2026 es el Excel; el CSV mock del 14/5 quedó deprecado.

## Excel target

- **Workbook**: `Relevo - Content - Article production DB.xlsx` en `igamingcom.sharepoint.com/sites/seoberlin/`
- **Tables**: `Table1` (articles, 23 cols since 26/5 INVENTORY-02), `Table2` (logs, 8 cols)
- **Acceso**: vía `scripts/sharepoint_msal.py` (MSAL device-code flow, refresh token en `~/.cache/relevo/msal_cache.bin`)
- Detalles vivos en memoria [[reference_inventory_excel_v2]] + [[project_sharepoint_msal_working]]

## Schema canonical de Table1 (articles)

23 columnas en orden fijo (NO reorder sin actualizar todo este file + cmd_content_plan + scripts):

| # | Campo | Tipo | Quién la llena | Notas |
|---|---|---|---|---|
| 0 | `id` | int autoincrement | content-plan al append | calculado con `_next_id(rows)` |
| 1 | `sport` | string | content-plan | Match exacto con `data/sports_config.json` (tildes incluidas: `Fútbol`, `Fórmula 1`) |
| 2 | `category` | enum | content-plan | `rules` / `tactics` / `origins` |
| 3 | `topic` | string | content-plan | descripción corta interna 5-10 palabras |
| 4 | `seo_title` | string ≤60 | content-plan | título público SEO |
| 5 | `url_slug` | kebab-case ASCII | content-plan | sin tildes ni eñes — `slugify` antes |
| 6 | `meta_description` | string 120-155 | content-plan | gancho SEO |
| 7 | `hub_slug` | string | content-plan | slug del deporte sin tildes (ej. `futbol`, `formula1`) |
| 8 | `sub_hub` | string | content-plan | uno de los `sub_hubs` del sport en config, o `""` |
| 9 | `name` | string | content-plan (Origins only) | nombre completo del jugador, `""` para Rules/Tactics |
| 10 | `hook` | kebab-case | content-plan (Origins only) | ángulo del artículo (`donde-nacio-y-su-familia`), `""` para Rules/Tactics |
| 11 | `priority` | enum | content-plan | `P0` / `P1` / `P2` — MVP hardcoded `P1` |
| 12 | `status` | enum | content-plan + /relevo new + reviewer | ver flow abajo |
| 13 | `created_by` | string | content-plan | typically `skill:content-plan`, `cli`, `manual`, `bulk-seed` |
| 14 | `created_at` | ISO 8601 Z | content-plan | UTC con `Z` suffix |
| 15 | `completed_at` | ISO 8601 Z | /relevo new al finalizar pipeline | "" hasta pipeline OK |
| 16 | `published_at` | ISO 8601 Z | reviewer / WP webhook | "" hasta publish |
| 17 | `quality_score` | int 0-100 | /relevo new tras self-critique | "" hasta tener score |
| 18 | `article_path` | string | /relevo new al final | `runs/{run_id}/` |
| 19 | `wp_url` | URL | reviewer / WP upload | "" hasta publish |
| 20 | `asana_subtask_gid` | string | /relevo new (asana_notify.py) | gid de la reviewer subtask del artículo |
| 21 | `notes` | string | content-plan + reviewer | contexto editorial libre + razón si blocked |
| 22 | `article_type` | enum | content-plan | `evergreen` / `news` / `casino` / `sportsbetting`. **Eje cardinal** que agrupa categorías (rules/tactics/origins/coaches → evergreen). Default `evergreen` si no se especifica. Agregado 26/5 vía INVENTORY-02 |

## Lifecycle del `status` field

```
content-plan      reviewer aprueba    /relevo new arranca       pipeline OK              reviewer publica WP
   |                    |                     |                        |                          |
   v                    v                     v                        v                          v
pending_review  →   pending          →    in_progress           →    review                →   published
                          |                     ↑                        |                       
                   reviewer rechaza      pipeline reset            reviewer rechaza
                          ↓                                              ↓
                       blocked                                        blocked
                                                                         
   (terminal: dead_letter si /relevo new falla después de retries)
```

**Status values** (validar antes de escribir):

- `pending_review` — content-plan recién appendeó. NO elegible para `/relevo new`.
- `pending` — reviewer aprobó. Elegible para `/relevo new` (pick por priority).
- `in_progress` — `/relevo new` lockeó la row (optimistic locking via update_table_row).
- `review` — pipeline terminó, hay draft+score+image, reviewer humano lo evalúa antes de publish.
- `published` — reviewer publicó en WP, `wp_url` poblado.
- `blocked` — rechazado (por reviewer pre-prod o pre-publish). Razón en `notes`.
- `dead_letter` — pipeline falló > N retries. Forensics en `runs/{run_id}/`.

## Schema canonical de Table2 (logs)

8 columnas. La escribe `lib/logging.md::log()`.

| # | Campo | Tipo |
|---|---|---|
| 0 | `id` | int autoincrement |
| 1 | `timestamp` | ISO 8601 Z |
| 2 | `article_id` | int (FK → articles.id) o vacío |
| 3 | `event` | enum (started, draft_done, critique1, revise, factcheck, image_completed, published, failed, plan_started, plan_completed, ...) |
| 4 | `score` | int 0-100 (si aplica) |
| 5 | `elapsed_seconds` | int |
| 6 | `tokens` | int (LLM tokens consumidos en el step) |
| 7 | `notes` | string |

## API que el skill usa

Vive en `scripts/sharepoint_msal.py`. Importar como módulo:

```python
import sys
sys.path.insert(0, 'scripts')
from sharepoint_msal import (
    get_token, resolve_drive_item, read_table_rows,
    append_table_row, update_table_row, find_row_by_id,
    ARTICLES_TABLE, LOGS_TABLE,
)
```

### Operaciones disponibles

#### `list_pending(sport=None, category=None) → list[row]`

Patrón. No es función nativa, se compone:

```python
token = get_token()
item = resolve_drive_item(token)
rows = read_table_rows(token, item, ARTICLES_TABLE)
pending = []
for r in rows:
    v = (r.get("values") or [[]])[0]
    if not v: continue
    if v[12] != "pending": continue
    if sport and v[1] != sport: continue
    if category and v[2] != category: continue
    pending.append(r)
# Sort by priority then created_at (P0 first, oldest within priority)
pending.sort(key=lambda r: (r["values"][0][11], r["values"][0][14]))
```

#### `lock_row(row_id) → bool`

Optimistic lock: leer la row, verificar status=pending, escribir status=in_progress vía update_table_row. Si Excel ya tiene otro status (carrera con otro editor), retornar False y dejar que el caller elija otra row.

```python
row = find_row_by_id(rows, row_id)
if not row: raise ValueError(f"row {row_id} not found")
vals = list(row["values"][0])
if vals[12] != "pending":
    return False  # alguien más se la llevó
vals[12] = "in_progress"
update_table_row(token, item, row["index"], vals, ARTICLES_TABLE)
return True
```

#### `update_row(row_id, fields_dict) → None`

Helper para actualizar un subconjunto de campos manteniendo el resto. Pattern:

```python
COL_IDX = {"id":0, "sport":1, "category":2, "topic":3, "seo_title":4,
           "url_slug":5, "meta_description":6, "hub_slug":7, "sub_hub":8,
           "name":9, "hook":10, "priority":11, "status":12, "created_by":13,
           "created_at":14, "completed_at":15, "published_at":16,
           "quality_score":17, "article_path":18, "wp_url":19,
           "asana_subtask_gid":20, "notes":21, "article_type":22}

row = find_row_by_id(rows, row_id)
vals = list(row["values"][0])
for k, v in fields_dict.items():
    vals[COL_IDX[k]] = v
update_table_row(token, item, row["index"], vals, ARTICLES_TABLE)
```

Ejemplo de uso al final de `/relevo new`:

```python
update_row(row_id, {
    "status": "review",
    "completed_at": iso_now,
    "quality_score": final_score,
    "article_path": f"runs/{run_id}/",
    "asana_subtask_gid": asana_gid,
})
```

#### `append_row(fields_dict) → int (new_id)`

Helper de content-plan. Construye una lista de 22 elementos con defaults razonables para campos no provistos.

```python
defaults = {
    "priority": "P1",
    "status": "pending_review",
    "created_by": "skill:content-plan",
    "created_at": iso_now,
}
COLS = ["id", "sport", ..., "notes"]  # las 22
new_id = _next_id(rows)
vals = [fields_dict.get(c, defaults.get(c, "")) for c in COLS]
vals[0] = new_id
append_table_row(token, item, vals, ARTICLES_TABLE)
```

#### `append_log(article_id, event, **kwargs)`

Escribe a Table2. TODO: helper específico para logs, no construido todavía. Por ahora se construye inline siguiendo el schema de 8 cols.

## Eventual consistency gotcha

Confirmado durante el bulk seed del 18/5: una append puede reportar HTTP 201 OK pero un read INMEDIATO no ver la nueva row. La row aparece en reads tras ~2-3 segundos.

**Pattern recomendado**:
- Si necesitás verificar que algo se escribió, esperar 2-3 segundos antes del read-back.
- No reaccionar a "row missing" sin retry — puede ser cache de Graph.
- Para bulk writes secuenciales (varios appends seguidos), no es problema: cada append es independiente y la response confirma con `index`.

`append_table_row` y `update_table_row` ahora tienen retry automático para HTTP 429/503/504 (ver `_call_with_retry` en `sharepoint_msal.py`). Esto cubre ~95% de los errores transitorios.

## Migración desde el CSV mock (histórico)

El `data/inventory.csv` del 14/5 tenía un schema viejo con campos en español (`deporte`, `categoria`, `tipo`, `nombre`, `gancho`, `attempts`, `wp_post_id`) y otros que no existen en el Excel (`tipo`, `attempts`, `wp_post_id`). **Se considera deprecado** desde 18/5; no usar como referencia.

Mapeo histórico (referencia, no se usa en código nuevo):

| CSV (14/5) | Excel (18/5) |
|---|---|
| `deporte` | `sport` |
| `categoria` | `category` |
| `tipo` | (removido — el `topic` field cubre la variante) |
| `nombre` | `name` |
| `gancho` | `hook` |
| `attempts` | (a logs si hace falta tracking) |
| `wp_post_id` | (`wp_url` cubre la referencia) |

## Conecta con

- [[reference_inventory_excel_v2]] — schema live + share URL + driveId
- [[project_sharepoint_msal_working]] — auth flow + helpers
- [[reference_relevo_sports_list]] — los 24 deportes canonical
- `scripts/sharepoint_msal.py` — implementación de las helpers
- `scripts/relevo_approve.py` — CLI para flips bulk de status
