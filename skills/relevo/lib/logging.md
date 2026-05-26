# Lib: logging strategy

Cada evento significativo del pipeline se loggea en DOS lugares:

1. **JSONL append-only** en `logs/runs.jsonl` (mock) o `/var/log/relevo/runs.jsonl` (prod) — para tooling/grep
2. **Mirror** en sheet `_logs` del workbook (prod) o `logs/_logs_mirror.csv` (mock) — para humanos scaneando

## Schema de evento

```json
{
  "timestamp": "2026-05-14T20:30:15.123Z",
  "run_id": "rules-1747251234",
  "event": "draft_completed",
  "level": "info",
  "deporte": "Fútbol",
  "categoria": "rules",
  "row_id": "2",
  "payload": {"word_count": 1850, "model": "claude-opus-4-7", "duration_ms": 45200}
}
```

## Niveles

- `info` — eventos normales del pipeline
- `warn` — algo subóptimo pero no fatal (retry, fallback de modelo)
- `error` — falla retriable o no-retriable

## Eventos canónicos a emitir

Lista de eventos que cada comando debería emitir. Mantener nombres exactos para tooling consistente:

### `/relevo new` pipeline

- `run_started` — al inicio, payload incluye inputs del editor
- `inventory_locked` — después de marcar row in_progress
- `research_started` / `research_completed`
- `draft_started` / `draft_completed`
- `critique_started` / `critique_completed` — payload con score
- `revise_started` / `revise_completed` — payload con score post-revise
- `factcheck_started` / `factcheck_completed`
- `image_started` / `image_completed` — payload con file path
- `wp_upload_started` / `wp_upload_completed` — payload con wp_post_id
- `asana_subtask_created` — payload con asana task gid
- `run_completed` — payload con duration total
- `stage_retry` — cuando una etapa reintenta. payload: stage, attempt, error
- `dead_letter` — cuando llega al cap de retries
- `stuck_rescued` — cuando el sweep detecta y rescata un in_progress viejo

### `/relevo content-plan` pipeline

- `plan_started` — payload con deporte, categoria, cantidad, focus
- `reglamento_loaded` — payload con path/url
- `inventory_dedup_done` — payload con N topics existentes considerados
- `topics_generated` — payload con N topics
- `topics_appended` — payload con array de nuevos row_ids
- `plan_completed`

## Cómo escribir

### JSONL

```python
import json, datetime, pathlib
LOG_FILE = pathlib.Path("logs/runs.jsonl")
LOG_FILE.parent.mkdir(exist_ok=True)

def log(event, run_id, level="info", **payload):
    record = {
        "timestamp": datetime.datetime.utcnow().isoformat() + "Z",
        "run_id": run_id,
        "event": event,
        "level": level,
        **payload,
    }
    with LOG_FILE.open("a") as f:
        f.write(json.dumps(record) + "\n")
```

### Mirror CSV (mock)

```python
import csv, pathlib
MIRROR = pathlib.Path("logs/_logs_mirror.csv")
HEADERS = ["timestamp","run_id","event","level","deporte","categoria","row_id","payload"]

if not MIRROR.exists():
    with MIRROR.open("w", newline="") as f:
        csv.writer(f).writerow(HEADERS)

def mirror_log(event, run_id, level="info", deporte="", categoria="", row_id="", **payload):
    with MIRROR.open("a", newline="") as f:
        csv.writer(f).writerow([
            datetime.datetime.utcnow().isoformat()+"Z",
            run_id, event, level, deporte, categoria, row_id,
            json.dumps(payload) if payload else "",
        ])
```

### Mirror Excel (prod)

POST a la table `logs` del sheet `_logs` via Graph API:
```
POST https://graph.microsoft.com/v1.0/drives/{drive_id}/items/{workbook_id}/workbook/worksheets/_logs/tables/logs/rows/add
Body: {"values": [[timestamp, run_id, event, level, deporte, categoria, row_id, json_payload]]}
```

## Reglas

- **Siempre** loggea timestamp en UTC ISO-8601.
- **Nunca** loggear secrets, PII completa, ni el body del artículo (solo word count + score).
- **Append-only** — los logs NO se editan. Si una entrada es errónea, agregar otra que la corrija.
- **Run_id** es la columna que junta todos los eventos de una ejecución; siempre incluirlo.
- Si el mirror (Excel/CSV) falla, **no abortar la ejecución** — sigue con JSONL local y registra `mirror_failed` como warn.
