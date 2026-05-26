---
name: relevo
description: |
  Workflow de generación de contenido SEO para Relevo.com. Invocar cuando el usuario pida un sub-comando del skill:
  - "/relevo new demo" / "/relevo demo" / "demo del pipeline" → ejecutar lib/cmd_demo.md (modo demostración, no genera artículo nuevo)
  - "/relevo new" / "/relevo nuevo" / "generar artículo Relevo" → ejecutar lib/cmd_new.md
  - "/relevo content-plan" / "/relevo plan" / "generar topics" / "idear artículos" → ejecutar lib/cmd_content_plan.md
  - "/relevo status" / "/relevo queue" / "ver cola Relevo" → ejecutar lib/cmd_status.md
  El skill orquesta el pipeline entero dentro de la sesión CC del editor: lee Inventory (CSV mock o SharePoint Excel), genera artículo multi-pass (research → draft → critique → revise → fact-check → imagen), sube a WordPress, notifica reviewer en Asana. Documentación arquitectónica completa en docs/adr/001-architecture.md.
---

# Skill `/relevo` — Relevo content generation workflow

## Cuándo invocar este skill

El editor escribe en su sesión CC algún sub-comando:

| Comando | Acción | Lib file |
|---|---|---|
| `/relevo new demo` (o `/relevo demo`) | Simula el pipeline E2E para una demostración. NO genera artículo nuevo — mapea a un draft pre-aprobado de Joel. | `lib/cmd_demo.md` |
| `/relevo new` | Genera 1 artículo de un topic existente del Inventory | `lib/cmd_new.md` |
| `/relevo content-plan` | Genera N topics nuevos al Inventory (status pending_review) | `lib/cmd_content_plan.md` |
| `/relevo status` | Consulta cola y métricas | `lib/cmd_status.md` |

## Cómo enrutar

Cuando el editor invoca `/relevo <subcomando>`:

1. **Caso especial — modo demo**: si el comando contiene la palabra `demo` (e.g. `/relevo new demo`, `/relevo demo`, `/relevo nuevo demo`), **siempre** rutear a `lib/cmd_demo.md`. El modo demo tiene precedencia sobre `cmd_new.md` cuando ambos podrían matchear.
2. Si subcomando no está en la lista, responde con el listado de comandos disponibles y termina.
3. Si está, **leer y ejecutar** el lib file correspondiente paso a paso. Esos files contienen las instrucciones operativas.

## Recursos compartidos (todos los comandos los necesitan)

- **Style guide**: `prompts/style_guide.md` — inyectar como base del system prompt en cualquier llamada de generación
- **Templates por categoría**:
  - Rules: `prompts/rules_template.md`
  - Tactics: `prompts/tactics_template.md`
  - Origins: `prompts/origins_template.md`
- **Sports config**: `data/sports_config.json` — 24 deportes, slugs, sub_hubs, reglamento availability
- **Inventory data**:
  - Mock mode: `data/inventory.csv`
  - Real mode: SharePoint Excel workbook (cuando esté creado)
- **Image gen**: `image_gen/generate-images.py` + `image_gen/strip-ai-metadata.py`
- **Lib helpers**:
  - `lib/pipeline.md` — pipeline multi-pass (research → draft → critique → revise → fact-check)
  - `lib/retry.md` — retry strategy con backoff
  - `lib/inventory.md` — abstracción InventoryProvider (Mock vs SharePoint)
  - `lib/logging.md` — formato JSONL + mirror a sheet `_logs`
  - `lib/image.md` — cómo invocar el script de imágenes

## Modo de operación

**Mock mode (default para testing/MVP)**:
- `INVENTORY_SOURCE=mock` — lee/escribe `data/inventory.csv`
- WordPress upload → escribe el HTML a `/tmp/relevo-{run_id}/article.html`
- Asana notif → escribe a `/tmp/relevo-{run_id}/asana_subtask.json`
- Image gen → real (usa LiteLLM si la `.env` está configurada, sino simula)

**Production mode** (cuando esté lista la infraestructura):
- `INVENTORY_SOURCE=sharepoint` — Graph API contra workbook real
- WordPress upload via WP MCP
- Asana notif via Asana MCP
- Image gen → real con LiteLLM

El editor no setea estos modes manualmente — vienen de `/etc/relevo/config.env` (deployment) o de defaults de mock para dev.

## Logs

Cualquier evento significativo se loggea en dos lugares:
1. JSONL append-only en `logs/runs.jsonl` (en mock; en producción será `/var/log/relevo/runs.jsonl`)
2. Mirror al sheet `_logs` (en producción) o a `logs/_logs_mirror.csv` (en mock)

Formato JSONL: `{"timestamp": "...", "run_id": "...", "event": "...", "level": "info|warn|error", "deporte": "...", "categoria": "...", "row_id": "...", "payload": {...}}`

## Asana — solo notificaciones humanas

Asana NO es state DB. Solo se crean subtasks para llamar la atención humana:
- Cuando un artículo termina y queda en `review` → subtask asignada round-robin
- Cuando un batch de topics nuevos termina `content-plan` → subtask de review batch
- Cuando un artículo va a `dead_letter` (2+ fails) → subtask `needs_manual_attention`

Parent task fijo: `1214527443265898` ("Pedidos Relevo - Inbox").

## Convenciones de naming

- `run_id` = UUID corto `{categoria}-{timestamp_unix}` (e.g., `rules-1747251234`)
- Archivos temporales en `/tmp/relevo-{run_id}/` (article.md, hero.jpg, asana_subtask.json, jobs.json)
- Slug deporte = lowercase, sin tildes, espacios → `-` (e.g., "Fórmula 1" → `formula1`)
