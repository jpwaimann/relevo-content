# Comando: `/relevo status`

Muestra estado actual del pipeline: cola pending, in_progress, review, dead_letter + métricas recientes.

## Flow

### Paso 1: leer Inventory completo

```python
all_rows = provider.list_topics(filters={})  # sin filtros, todo
```

### Paso 2: leer logs recientes (últimos 7 días)

```python
import json, datetime
recent_logs = []
cutoff = datetime.datetime.utcnow() - datetime.timedelta(days=7)
with open("logs/runs.jsonl") as f:
    for line in f:
        evt = json.loads(line)
        ts = datetime.datetime.fromisoformat(evt["timestamp"].rstrip("Z"))
        if ts > cutoff:
            recent_logs.append(evt)
```

(En production, leer del JSONL local y opcionalmente complementar con sheet `_logs` para cross-host visibility.)

### Paso 3: calcular agregados

```python
status_counts = {
    "pending_review": 0,
    "pending": 0,
    "in_progress": 0,
    "review": 0,
    "published": 0,
    "dead_letter": 0,
}
by_deporte = defaultdict(lambda: dict.fromkeys(status_counts.keys(), 0))
by_categoria = defaultdict(lambda: dict.fromkeys(status_counts.keys(), 0))

for row in all_rows:
    status = row.get("status", "")
    if status in status_counts:
        status_counts[status] += 1
        by_deporte[row["deporte"]][status] += 1
        by_categoria[row["categoria"]][status] += 1

# Métricas de los últimos 7 días desde logs
run_completed_events = [e for e in recent_logs if e["event"] == "run_completed"]
articles_generated_7d = len(run_completed_events)
avg_duration = sum(e["payload"]["duration_seconds"] for e in run_completed_events) / max(1, len(run_completed_events))
avg_score = sum(e["payload"]["final_score"] for e in run_completed_events) / max(1, len(run_completed_events))

dead_letter_events = [e for e in recent_logs if e["event"] == "dead_letter"]
dead_letter_7d = len(dead_letter_events)
```

### Paso 4: output al editor

```markdown
## 📊 Relevo Status

### Inventory snapshot
| Status | Count |
|---|---|
| pending_review (esperando aprobación) | {N1} |
| pending (listos para `/relevo new`) | {N2} |
| in_progress (corriendo ahora) | {N3} |
| review (esperando reviewer) | {N4} |
| published | {N5} |
| dead_letter | {N6} |

### Distribución por deporte (top 5 con más actividad)
| Deporte | Pending | In progress | Review | Published |
|---|---|---|---|---|
| {sport_1} | {p} | {ip} | {r} | {pub} |
| ... | ... | ... | ... | ... |

### Distribución por categoría
| Categoría | Pending | In progress | Review | Published |
|---|---|---|---|---|
| Rules | {p} | {ip} | {r} | {pub} |
| Tactics | {p} | {ip} | {r} | {pub} |
| Origins | {p} | {ip} | {r} | {pub} |

### Últimos 7 días
- **Artículos generados**: {articles_generated_7d}
- **Tiempo promedio por artículo**: {avg_duration:.0f}s
- **Score promedio**: {avg_score:.1f}/100
- **Dead-letters**: {dead_letter_7d}

### Stuck rows (in_progress > 30 min)
{listar si hay, con row_id, deporte, último evento timestamp}

### Próximos topics a generar (top 5 por priority)
{listar los 5 primeros pending por priority desc}
```

### Paso 5: AskUserQuestion opcional

Después del resumen, ofrecer:

```
"¿Querés hacer algo más?"
  Opciones:
    - "Generar el siguiente artículo (/relevo new auto-pick)"
    - "Inspeccionar un topic específico (te muestro el row completo)"
    - "Ver más detalles de un dead-letter"
    - "Nada, gracias"
```

## Reglas

- **Read-only**: este comando NO modifica nada. Solo consulta y muestra.
- **Rápido**: si la queue es grande, mostrar solo top-N por agregado, no listas enteras.
- **Bonus**: si hay stuck rows, mostrar prominentemente al inicio para que el editor sepa que hay limpieza pendiente.
