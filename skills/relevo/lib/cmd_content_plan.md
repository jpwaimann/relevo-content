# Comando: `/relevo content-plan`

Genera N topics nuevos al Excel inventory con `status=pending_review`. Puebla el inventory cuando se agrega un deporte/categoría o cuando hace falta refrescar topics. **No** produce artículos — eso lo hace `/relevo new` después de que el editor flippee las rows a `status=pending`.

## Pre-requisitos del skill al arrancar

- Leer `data/sports_config.json` (24 deportes, slugs, sub_hubs, reglamento_available, _pdf_name).
- Leer `prompts/style_guide.md` — siempre va al system prompt.
- Leer `prompts/topic_ideation_lineamientos.md` — siempre va al system prompt en este comando. Codifica L1–L5 (evitar topics elementales en tier-1, dedup obligatorio vs relevo.com, verificación de bios, curaduría editorial, ángulo FIFA WC 2026).
- Saber qué template corresponde a la categoría que pida el editor (`prompts/rules_template.md`, `prompts/tactics_template.md`, `prompts/origins_template.md`).
- Confirmar que `scripts/sharepoint_msal.py` está auth-eado (refresh token cacheado en `~/.cache/relevo/msal_cache.bin`). Si no, el primer comando dispara el device-code flow una vez.

## Flow

### Paso 1: AskUserQuestion al editor

```
Pregunta 1: "¿Para qué deporte querés generar topics nuevos?"
  Opciones: los 24 deportes (priorizar arriba: Fútbol, Tenis, Baloncesto, MMA, F1, Boxeo).
  Todos tienen reglamento disponible en SharePoint al 18/5/2026 — sin warnings.

Pregunta 2: "¿Qué categoría?"
  Opciones: [
    "Rules — Reglas y conceptos del deporte",
    "Tactics — Tácticas, posiciones, técnicas",
    "Origins — Biografías de jugadores"
  ]

Pregunta 3: "¿Cuántos topics querés generar?"
  Opciones: ["5 — test rápido", "10 — batch chico", "20 — batch estándar", "50 — bulk inicial"]
  Default: 20.

Pregunta 4 (opcional, free text): "¿Algún focus específico? (e.g. 'tácticas modernas LaLiga 2025-26', 'jugadores españoles up-and-coming', 'reglas técnicas avanzadas')"
```

### Paso 2: Crear run_id + log

```python
run_id = f"plan-{int(time.time())}"
mkdir -p /tmp/relevo-{run_id}/
log("plan_started", run_id, level="info",
    deporte=deporte, categoria=categoria,
    payload={"cantidad": N, "focus": focus or "(none)"})
```

### Paso 3a: Cargar reglamento (solo Rules / Tactics)

**Para Origins → skipear este paso completo.** Origins no usa reglamento; usa WebSearch sobre jugadores actuales (Paso 3c).

**Para Rules / Tactics**:

El `_pdf_name` exacto está en `data/sports_config.json` por deporte. Bajar el PDF del SharePoint folder vía Graph y extraerle el texto. **Gotcha confirmado en smoke test 18/5**: los items que Pablo subió manualmente al folder quedaron como **subcarpetas con el PDF adentro** (no como files directos), mientras que los uploads recientes vía Graph (`Cycling - UCI Road.pdf`, `Paralympics - IPC Classification Code.pdf`, `Motorcycling - FIM Grand Prix 2025.pdf`) sí son files. Hay que manejar ambos casos.

Pattern recomendado (usar el helper inline en Python — más limpio que el `curl`+`jq` chain):

```python
import sys, base64, os, requests
sys.path.insert(0, 'scripts')
from sharepoint_msal import get_token, GRAPH

token = get_token()
FOLDER_SHARE = "https://igamingcom.sharepoint.com/:f:/s/seoberlin/IgAiLCRySbFDRLJN8ZyCiYpjAWscrbjKBb1lFB6Ud3tyWKo"
enc = "u!" + base64.urlsafe_b64encode(FOLDER_SHARE.encode()).rstrip(b"=").decode()
folder = requests.get(f"{GRAPH}/shares/{enc}/driveItem", headers={"Authorization": f"Bearer {token}"}).json()
drive, fid = folder["parentReference"]["driveId"], folder["id"]

# Find the entry matching the sport's _pdf_name (could be a file OR a folder containing 1 PDF)
children = requests.get(f"{GRAPH}/drives/{drive}/items/{fid}/children", headers={"Authorization": f"Bearer {token}"}).json()["value"]
entry = next(c for c in children if c["name"] == PDF_NAME_FROM_CONFIG)

if "folder" in entry:
    # Pablo manual upload — navigate one level deeper
    inner = requests.get(f"{GRAPH}/drives/{drive}/items/{entry['id']}/children", headers={"Authorization": f"Bearer {token}"}).json()["value"]
    pdf_file = next(c for c in inner if c["name"].lower().endswith(".pdf"))
else:
    # Recent Graph upload — direct file
    pdf_file = entry

# Use the pre-authenticated download URL (avoid 404 weirdness on /content for some items)
item_meta = requests.get(f"{GRAPH}/drives/{drive}/items/{pdf_file['id']}", headers={"Authorization": f"Bearer {token}"}).json()
dl_url = item_meta["@microsoft.graph.downloadUrl"]
content = requests.get(dl_url, timeout=180).content

out = f"/tmp/relevo-{run_id}/reglamento.pdf"
with open(out, "wb") as f:
    f.write(content)
```

Después extraer texto:
```bash
pdftotext /tmp/relevo-{run_id}/reglamento.pdf /tmp/relevo-{run_id}/reglamento.txt
```

Log: `log("reglamento_loaded", run_id, payload={"pdf_name": PDF_NAME, "txt_bytes": <bytes>})`

**Si el PDF >5 MB de texto extraído** (raro pero pasa con Football IFAB / Swimming WAQU): truncar al primer 60% (suele cubrir las rules core, técnico/anexos van al final). Marcar en log: `payload={"truncated": true, "original_bytes": N}`.

### Paso 3b: WebSearch existing coverage (solo Rules / Tactics)

Antes de generar topics, **investigar qué ya existe**. Doble propósito: dedup soft contra cobertura ya hecha + inspiración del ángulo periodístico que usan medios tier-1 ES.

```
WebSearch queries (3-4 queries simples — NO usar `site:X OR site:Y`, en smoke test
del 18/5 retornó 0 resultados con esa sintaxis; usar queries naturales y dejar que
el ranking traiga tier-1):

  - {deporte} reglas explicación {concepto-clave}
  - "qué es" {concepto-deporte-categoria} explicación
  - {concepto} {deporte} 2025 OR 2026 reglamento oficial
  - {deporte} {sub-concepto} cómo funciona

Si querés focalizar a una sola fuente, hacer una query separada por fuente:
  - site:relevo.com {deporte} {keyword}
  - site:marca.com {deporte} {keyword}
  - site:as.com {deporte} {keyword}
```

Capturar para el system prompt del Paso 5:
- Lista de titulares vistos (10-15) — para evitar clones literales
- Cualquier ángulo distintivo que Relevo todavía no cubrió pero los competidores sí

Log: `log("web_research_done", run_id, payload={"queries_run": N, "results_captured": M})`

### Paso 3c: WebSearch Origins research (solo Origins)

**No usa reglamento.** Usa WebSearch obligatorio para identificar jugadores relevantes hoy para audiencia española.

```
WebSearch queries (3-5 por deporte, ajustadas al focus si lo hay):
  - "jugadores {deporte} promesas 2026 españoles"
  - "{deporte} estrellas {liga relevante para ES}"
  - "best {deporte} players 2026" + site filter a fuentes deportivas
  - Si deporte=fútbol y mes ∈ {abril, mayo, junio}: "FIFA World Cup 2026 squad players Spain rivals up-and-coming"
  - Si Pablo dio focus en pregunta 4: traducir focus a queries
```

Para cada jugador candidato, capturar: nombre completo, edad, equipo actual, contexto (¿por qué relevante hoy?), 1-2 ganchos potenciales (origen humilde / familia / momento clave / fichaje / lesión que superó / etc.).

Output esperado: lista interna de 20-40 candidatos jugador→ganchos, de la cual el LLM elige N para generar el batch de topics.

Log: `log("origins_research_done", run_id, payload={"queries_run": N, "candidates_found": M})`

### Paso 4: Leer inventory existente para dedup

```python
# Vía sharepoint_msal.py — lee Table1 (articles)
import subprocess, json
result = subprocess.run(
    ['.venv/bin/python', 'scripts/sharepoint_msal.py', 'read'],
    capture_output=True, text=True, check=True,
)
rows = [json.loads(line) for line in result.stdout.strip().splitlines() if line.strip()]

# Filtrar por deporte + categoria
existing = [r for r in rows if r[1] == deporte and r[2] == categoria]
existing_seo_titles = {r[4] for r in existing}  # col 4 = seo_title
existing_url_slugs  = {r[5] for r in existing}  # col 5 = url_slug
```

Log: `log("inventory_dedup_loaded", run_id, payload={"existing_count": len(existing)})`

### Paso 5: Generación de topics con LLM

System prompt construction (en este orden):

```
[CONTENIDO DE prompts/style_guide.md]
---
[CONTENIDO DE prompts/topic_ideation_lineamientos.md]
  L1: Evitar topics elementales en deportes de tier-1 (fútbol, tenis, MMA, F1, baloncesto).
  L2: Dedup obligatorio contra relevo.com — el LLM debe asumir que cualquier topic obvio ya está cubierto.
  L3: Verificación de nombres propios en bios (no inventar atletas, usar segundo apellido en homónimos).
  L4: Curaduría editorial sobre bios (peso real para el lector español).
  L5: Ángulo FIFA WC 2026 cuando aplica (fútbol, mayo-julio 2026).
---
[CONTENIDO DEL TEMPLATE DE LA CATEGORÍA]
  Rules    → prompts/rules_template.md
  Tactics  → prompts/tactics_template.md
  Origins  → prompts/origins_template.md
---
## Fuentes de referencia para esta sesión

[Para Rules/Tactics:]
Reglamento oficial (extracto):
{contenido de reglamento.txt}

Coverage existente en relevo.com + tier-1 ES (titulares):
{lista de titulares vistos en Paso 3b}

[Para Origins:]
Candidatos de jugadores (de Paso 3c):
{lista de 20-40 jugadores con ganchos sugeridos}

---
## Tarea

Generar exactamente {N} ideas de artículos para Relevo.com sobre {deporte} en categoría {categoria}.

Cada idea debe ser un objeto JSON con TODOS estos campos:

- "sport": "{deporte tal cual aparece en sports_config — con tildes correctas}"
- "category": "rules" | "tactics" | "origins"
- "topic": string corto interno (5-10 palabras), describe el artículo desde la cocina periodística
- "seo_title": ≤60 chars, siguiendo el patrón del template; el título público
- "url_slug": kebab-case, en español, sin tildes ni eñes, derivado del seo_title
- "meta_description": 120-155 chars, gancho SEO
- "hub_slug": "{slug del deporte tal cual aparece en sports_config — sin tildes}"
- "sub_hub": uno de los sub_hubs disponibles para este deporte, o "" si no aplica
- "name": SOLO para Origins. Nombre completo del jugador. "" para Rules/Tactics.
- "hook": SOLO para Origins. Hook slug en español (e.g. "donde-nacio-y-su-familia"). "" para Rules/Tactics.
- "notes": 1 línea contextual (≤120 chars) — qué hace especial al artículo o por qué este ángulo

## Restricciones (NO romper)

1. NO repetir seo_titles ni url_slugs que ya existan. Lista a evitar:
   seo_titles existentes: {existing_seo_titles serializados}
   url_slugs existentes:  {existing_url_slugs serializados}

2. Sport y hub_slug DEBEN coincidir con los valores canonical de sports_config (Fútbol/futbol, no Futbol/fútbol).

3. Distribución por categoría:
   - Rules    → mix entre rules principales (1 pillar) y rules específicas
   - Tactics  → distribución entre tactics_types, tactics_positions, tactics_strategy
   - Origins  → mix español + internacional relevante en ligas/torneos que mira España; mix de superstar + up-and-coming

4. Focus adicional del editor: {focus o "ninguno"}

5. Adherir al manual de estilo (voz periodística española, anti-IA-isms, no superlativos vacíos, etc.) — ya inyectado arriba.

6. Adherir a los lineamientos de ideación L1–L5 — ya inyectados arriba. En particular:
   - Si {deporte} ∈ {Fútbol, Tenis, MMA, Fórmula 1, Baloncesto} → **prohibido** generar topics elementales tipo *"qué es X"*, *"cuánto dura Y"*, *"cuándo se sanciona Z"*. Cada topic debe atacarse desde ángulo no obvio (evolución de la regla, dato cuantitativo, decisión histórica, comparativa, cambio cultural, debate contemporáneo).
   - Para deportes de cobertura menor (rugby, balonmano, vóley, fútbol sala, hockey patines/hierba, vela, wrestling, etc.) los topics elementales sí son válidos.
   - Para Origins: verificación de existencia + deporte + posición. Usar segundo apellido si hay homonimia conocida.

## Output

Un JSON array con exactamente {N} objetos. Sin texto antes ni después. Sin code fences.
Ejemplo de shape (1 objeto):
{ "sport": "...", "category": "...", "topic": "...", "seo_title": "...", "url_slug": "...", "meta_description": "...", "hub_slug": "...", "sub_hub": "...", "name": "", "hook": "", "notes": "..." }
```

User message: `Generá los ${N} topics ahora.`

Log: `log("llm_generation_done", run_id, payload={"requested": N, "raw_output_bytes": len(output)})`

### Paso 6: Validar y dedup output del LLM

```python
import json, re, unicodedata

def slugify(s):
    s = unicodedata.normalize('NFKD', s).encode('ascii','ignore').decode().lower()
    return re.sub(r'[^a-z0-9]+', '-', s).strip('-')

def is_valid_kebab(s):
    return bool(re.fullmatch(r'[a-z0-9]+(-[a-z0-9]+)*', s or ''))

raw = output.strip()
if raw.startswith('```'): raw = raw.split('```')[1].lstrip('json').strip()
candidates = json.loads(raw)

valid = []
seen_slugs, seen_titles = set(), set()
for t in candidates:
    # Required fields presentes
    for k in ('sport','category','topic','seo_title','url_slug','meta_description','hub_slug','notes'):
        if not t.get(k): break
    else:
        # Validaciones de forma
        if len(t['seo_title']) > 60: continue
        if not (120 <= len(t['meta_description']) <= 155): continue
        if not is_valid_kebab(t['url_slug']): continue
        # Dedup local + inventory
        if t['seo_title'] in seen_titles or t['seo_title'] in existing_seo_titles: continue
        if t['url_slug']  in seen_slugs  or t['url_slug']  in existing_url_slugs:  continue
        # Origins-specific
        if t['category']=='origins' and not (t.get('name') and t.get('hook')): continue
        seen_titles.add(t['seo_title']); seen_slugs.add(t['url_slug'])
        valid.append(t)

if len(valid) < max(3, N // 2):
    log("llm_generation_below_threshold", run_id, level="warn",
        payload={"requested": N, "valid": len(valid)})
```

Si `len(valid) < N/2` → marcar warning, reportar al editor, **no reintentar automáticamente**.

Guardar `valid` en `/tmp/relevo-{run_id}/topics_generated.json` para forensics.

### Paso 7: Append al Excel inventory

Orden EXACTO de las 23 columnas de `Table1` (cross-verified live 26/5 post INVENTORY-02):

```
0: id                   ← Excel autoincrement, mandar "" (no se respeta, igual lo asigna Excel)
1: sport                ← t["sport"]
2: category             ← t["category"]
3: topic                ← t["topic"]
4: seo_title            ← t["seo_title"]
5: url_slug             ← t["url_slug"]
6: meta_description     ← t["meta_description"]
7: hub_slug             ← t["hub_slug"]
8: sub_hub              ← t["sub_hub"] or ""
9: name                 ← t["name"]   (Origins only; "" en Rules/Tactics)
10: hook                ← t["hook"]   (Origins only; "" en Rules/Tactics)
11: priority            ← "P1"        (HARDCODED MVP — todo P1 hasta que armemos human-in-the-loop)
12: status              ← "pending_review"
13: created_by          ← "skill:content-plan"
14: created_at          ← datetime.now(datetime.UTC).isoformat(timespec="seconds").replace("+00:00","Z")
15: completed_at        ← ""
16: published_at        ← ""
17: quality_score       ← ""
18: article_path        ← ""
19: wp_url              ← ""
20: asana_subtask_gid   ← ""  (se llena en Paso 8 si Asana succeed)
21: notes               ← t["notes"]
22: article_type        ← t.get("article_type", "evergreen")  (rules/tactics/origins/coaches → evergreen; news/casino/sportsbetting si aplica)
```

Append vía helper (un row por llamada, en loop):

```python
import subprocess
for t in valid:
    values = [
        "", t["sport"], t["category"], t["topic"], t["seo_title"], t["url_slug"],
        t["meta_description"], t["hub_slug"], t.get("sub_hub",""),
        t.get("name",""), t.get("hook",""), "P1", "pending_review",
        "skill:content-plan", iso_now, "", "", "", "", "", "", t["notes"],
        t.get("article_type", "evergreen"),
    ]
    # Llamada directa al helper Python (más limpio que CLI args con espacios)
    from sharepoint_msal import get_token, resolve_drive_item, append_table_row
    token = get_token()
    item = resolve_drive_item(token)
    append_table_row(token, item, values)
```

> **Nota MVP**: `sharepoint_msal.py` actualmente sólo tiene un `append` CLI de 5 argumentos. Para batch usar la lib internamente. Si Pablo prefiere CLI: extender el script con un sub-comando `append-row --json '{...}'`. (Tarea follow-up, no bloqueante.)

Log por row: `log("inventory_row_appended", run_id, row_id=new_id, payload={"seo_title": t["seo_title"]})`.

### Paso 8: Asana subtask de batch review

Crear UNA subtask agregada bajo el parent task de Relevo Content - AI (`1214109786765359`):

- Subject: `[Content Plan] {deporte}/{categoria} — {len(valid)} topics para review (batch {run_id})`
- Notes (densas, mini-spec):
  - run_id + timestamp
  - deporte, categoria, focus dado
  - cuántos pidió Pablo vs cuántos generados (N requested vs valid)
  - tabla markdown con: # | seo_title | url_slug | (name + hook si Origins) | notes
  - Próximo paso explícito: "Aprobar = editar status=pending en el Excel para esos rows. Rechazar = editar status=blocked + agregar razón en notes."
- Asignee: reviewer jefe del momento (MVP: Joel) — referenciar el reviewer pool del config del skill (`config/reviewers.json` o env var `RELEVO_DEFAULT_REVIEWER`).
- Due: today + 2 business days.

Patrón de invocación: extender `scripts/asana_notify.py` con flag `--mode=batch_plan` (o crear `scripts/asana_batch_plan.py` siguiendo el mismo shape). Para esta primera versión MVP: invocar inline en el skill con `curl` contra Asana REST API si el helper no existe todavía.

Log: `log("asana_batch_subtask_created", run_id, payload={"asana_gid": gid})`.

### Paso 9: Output al editor en chat

Markdown table corta para el editor:

```markdown
## ✅ Topic plan generado — run `{run_id}`

**Deporte**: {deporte} · **Categoría**: {categoria}
**Generados**: {len(valid)} / {N} pedidos (status=`pending_review`)
**Focus aplicado**: {focus or "ninguno"}

### Topics nuevos en el Inventory

| # | seo_title | url_slug | name | sub_hub |
|---|---|---|---|---|
| 1 | … | … | … | … |
| 2 | … | … | … | … |
| … | … | … | … | … |

**Asana subtask**: [batch review #{run_id}](https://app.asana.com/0/0/{gid})
**Reviewer asignado**: {reviewer}
**Due**: {due_date}

### Próximos pasos
1. Reviewer abre la subtask Asana, contrasta cada topic con el Excel inventory.
2. **Aprobar**: editar columna `status` de la row a `pending` (queda disponible para `/relevo new`).
3. **Rechazar**: status a `blocked` + razón en `notes`.

{Si len(valid) < N/2: bloque de warning explicando por qué el batch quedó pobre — probable focus mal definido, inventory saturado, o reglamento truncado}
```

### Paso 10: log plan_completed

```python
log("plan_completed", run_id, level="info", payload={
    "duration_seconds": time.time() - start_ts,
    "topics_generated": len(valid),
    "topics_requested": N,
    "dedup_rejected": N - len(valid),
    "asana_subtask_gid": gid,
})
```

## Reglas de invocación

- **NUNCA** appendear con `status=pending` directo — siempre `pending_review` para forzar el gate humano. El editor flippea a `pending` después de validar en Asana.
- **NUNCA** sobreescribir topics existentes — dedup estricto por `seo_title` Y por `url_slug`, ambos local-al-batch y vs Excel inventory.
- **Priority HARDCODED a `P1`** en MVP. La lógica P0/P1/P2 (search volume, calendario deportivo, coverage gaps) está parkeada en task #7 del kickoff de content planning — ver [[reference: `project_relevo_content_planning_kickoff.md` memoria]] y la sección v2 features del repo público.
- **Si el modelo genera < N/2 topics válidos**: no reintentar automáticamente. Reportar al editor y dejar que pida más con otro foco o ajuste el reglamento.
- **Origins**: el modelo elige nombres de jugadores basándose en WebSearch + curaduría — el batch review humano es **crítico** acá. Puede equivocarse de nombres, citar players con poco search volume, o mezclar contextos.
- **Reglamento PDF >5 MB extraído**: truncar al primer 60%. Loggear truncation. Las rules core suelen estar al inicio.
- **Si `status=pending_review` molesta a alguna validación downstream**: alternativa MVP es escribir `status=blocked` con `notes` empezando en `"AWAITING_REVIEW:"` — pero más limpio mantener el valor explícito.

## Helpers que asume este comando

- `scripts/sharepoint_msal.py` — auth + read/append/download a SharePoint. **LIVE**.
- `pdftotext` (poppler-utils) — extracción PDF. Asumir instalado en el host AWS; si falta, `apt install poppler-utils`.
- `scripts/asana_batch_plan.py` — crea la subtask de batch review. **LIVE** desde 18/5/2026 (smoke test post-batch). Acepta `--run-id`, `--sport`, `--category`, `--reviewer-gid`, `--topics-json` (path al JSON con los rows generados).
- `scripts/asana_notify.py` — crea la subtask de review por artículo (post-pipeline en `/relevo new`). Distinto de batch_plan. **LIVE**.
- `data/sports_config.json` — canonical mapping deporte → slug + sub_hubs + _pdf_name. **LIVE** (actualizado 18/5).
- `prompts/style_guide.md`, `prompts/*_template.md` — system prompt ingredients. **LIVE**.

## Gotchas confirmados en smoke test 18/5/2026 (Fútbol + Rules, 5 topics)

1. **PDFs como subcarpetas**: Los items que Pablo subió manualmente al folder `Sports Official Rules` quedaron como folders, no files. El skill DEBE chequear si la entry tiene `"folder"` y navegar un nivel adentro para encontrar el PDF real. Los uploads via Graph (Cycling/Paralympics/Motorcycling) sí son files directos. Ambos casos manejados en el snippet del Paso 3a.

2. **WebSearch `site:X OR site:Y` no funciona**: confirmado experimentalmente, retorna 0 resultados. Usar queries simples o site filter de a una fuente por query. Actualizado el Paso 3b.

3. **`datetime.utcnow()` deprecation**: Python 3.12+ avisa. Usar `datetime.now(datetime.UTC).isoformat(timespec="seconds").replace("+00:00", "Z")`. Actualizado el Paso 7.

4. **Pre-authenticated downloadUrl > /content endpoint**: en el smoke test, GET `/drives/{drive}/items/{id}/content` dio 404 para items en este drive aunque el item existe. El pattern que sí funcionó: GET `/drives/{drive}/items/{id}` (sin /content), leer `@microsoft.graph.downloadUrl`, descargar de ahí. Es una URL pre-autenticada de corta duración (~1h), no requiere Bearer header.

## Follow-ups conocidos (no bloqueantes)

- ~~Alinear `lib/inventory.md` al schema nuevo de 22 cols~~ ✅ done (schema doc cross-verified live + ampliado a 23 cols el 26/5 via INVENTORY-02).
- Mini-comando `/relevo approve <run_id>` para flip-eo masivo `pending_review → pending` por batch (en vez de editar row por row en Excel).
- Helper `scripts/get_reglamento.py <sport_slug> <dest>` para esconder los `curl` del Paso 3a detrás de una sola call. Cosmético, no bloqueante.
- Cuando IT haga grant de application permissions en la Azure app `n8n` ([[reference_azure_n8n_app]]), el skill seguirá funcionando sin cambios — solo el helper `sharepoint_msal.py::get_token` cambia internamente.
