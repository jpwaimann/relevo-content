# Lib: retry strategy

## Patrón general

Stage-level retries con backoff exponencial. Cuando una etapa del pipeline falla con error retriable, esperar y reintentar. Después de N attempts, decisión: continuar al siguiente paso del pipeline si tolerable (e.g., el mirror log falla pero el draft está hecho), o ir a dead-letter.

## Backoff

```
attempt 1 falla  → wait 30s  → attempt 2
attempt 2 falla  → wait 2 min → attempt 3
attempt 3 falla  → wait 8 min → attempt 4 (último)
attempt 4 falla  → dead-letter
```

Total tiempo entre primer fallo y dead-letter: ~10.5 min. Para operaciones críticas como `draft` que tardan ~1 min cada attempt, el ciclo completo puede tomar ~15 min.

## Clasificación de errores

### Retriables (vale la pena reintentar)
- `429 Too Many Requests` — rate limit
- `5xx` — server error transitorio
- `requests.exceptions.Timeout` / `ConnectionError` — network blip
- JSON malformado en response — modelo glitch, retry probablemente funciona
- WP MCP error genérico cuando contiene "timeout" o "unavailable"

### NO retriables (directo a dead-letter)
- `400 Bad Request` — algo está mal con nuestra request, retry no ayuda
- `401 Unauthorized` — token inválido, requiere intervención humana
- `403 Forbidden` — permisos, requiere humano
- `404 Not Found` — el recurso no existe, retry no ayuda
- Validation errors locales (e.g., row no encontrada en inventory)

### Especiales

- **Quality gate failure** (score < 80) NO es un retry de la etapa — es un revise pass diferente. Tracked aparte (max 2 revises). Si después de 2 revises sigue < 80, va a dead-letter.

## Implementación

```python
import time, functools

RETRY_DELAYS = [30, 120, 480]  # seconds
RETRIABLE_HTTP = {429, 500, 502, 503, 504}

class NonRetriableError(Exception):
    pass

def retry_stage(stage_name, run_id):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt, delay in enumerate(RETRY_DELAYS + [None], start=1):
                try:
                    return func(*args, **kwargs)
                except NonRetriableError as e:
                    log("stage_failed_nonretriable", run_id, level="error",
                        stage=stage_name, error=str(e))
                    raise
                except Exception as e:
                    is_retriable = _is_retriable(e)
                    if not is_retriable or delay is None:
                        log("stage_failed_final", run_id, level="error",
                            stage=stage_name, attempt=attempt, error=str(e))
                        raise NonRetriableError(f"{stage_name} failed after {attempt} attempts: {e}")
                    log("stage_retry", run_id, level="warn",
                        stage=stage_name, attempt=attempt, error=str(e), next_retry_in=delay)
                    time.sleep(delay)
        return wrapper
    return decorator

def _is_retriable(exc):
    msg = str(exc).lower()
    if "401" in msg or "403" in msg or "404" in msg or "400" in msg:
        return False
    if any(s in msg for s in ["timeout", "unavailable", "rate limit", "429", "500", "502", "503", "504"]):
        return True
    return True  # default: try once more
```

## Stuck detection (housekeeping)

Cada `/relevo new` arranca con un sweep:

```python
from datetime import datetime, timedelta

def sweep_stuck_rows(provider):
    """Detectar rows con status=in_progress más viejas que 30 min sin actividad reciente en logs."""
    threshold = datetime.utcnow() - timedelta(minutes=30)
    stuck_rows = []
    for row in provider.list_topics(filters={"status": "in_progress"}):
        last_event = find_last_log_event_for_row(row["id"])
        if last_event and parse_timestamp(last_event["timestamp"]) < threshold:
            stuck_rows.append(row)
    return stuck_rows
```

Si encuentra stuck rows, pregunta al editor con AskUserQuestion:
- "Encontré N artículos colgados en in_progress hace más de 30 min. ¿Quieres rescatarlos?"
  - "Reintentar el último" → continúa pipeline desde la última etapa que loggeó
  - "Marcar como pending" → reset status a pending, vuelve a la cola
  - "Marcar como dead-letter" → resigna y va a `_dead_letter`
  - "Ignorar por ahora" → continúa normal

## Cuándo NO usar retry

- **Operaciones idempotentes con efectos visibles**: si la operación ya creó algo (WP draft) y nosotros no sabemos el ID, retry crearía duplicado. Mejor pasar a humano que duplicar.
- **Mirror log falla**: continuar pipeline, no bloquear todo el trabajo por un log que no escribe. Loggear `mirror_failed` como warn y seguir.
- **Imagen falla**: si el script de imágenes falla, el artículo puede subirse a WP sin imagen y un humano la agrega después. No vale la pena bloquear todo el pipeline por una imagen.

Estas son decisiones de diseño que el lib file `cmd_new.md` aplica explícitamente, no automatización ciega.
