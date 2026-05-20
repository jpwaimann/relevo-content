# ADR-001: Arquitectura del workflow de generación de contenido Relevo

**Status**: Approved
**Fecha**: 2026-05-14
**Supersedes**: n8n workflow `Vw2UmDIHsC2XEuzy` (v17)

---

## 1. Contexto

Relevo.com necesita producción escalable de contenido SEO evergreen para 24 deportes en 3 categorías (Rules / Tactics / Origins). Target inicial 50 artículos/día, escalable a 500+/día.

**Mandato C-level**: "todo en Claude Code" como UX para todo el equipo. Hay precedente interno con proyectos similares en modalidad skills/agents.

**Equipo**: 2 content editors no-técnicos, acceden vía SSH a un host AWS compartido con usuarios individuales.

**Stack ya disponible**:
- Passbolt (secrets company-wide)
- Azure AD app única (tenant `32a0dce7-...`, client_id `7eca9e80-...`) — cubre Graph (Excel/SharePoint/Drive/Outlook/Teams)
- LiteLLM proxy `litellm.bidwhig.com` (texto + imágenes)
- WordPress beta (credenciales pendientes)
- Asana (parent task `1214527443265898`)
- Image generation toolkit Python estandarizado (`generate-images.py` + `strip-ai-metadata.py` con C2PA stripping, modelo `gpt-image-2`)
- Style guide Relevo v1 + templates Rules & Basics + Tactics Explainers + Origins (entregados 14/5)
- 21 reglamentos PDF en SharePoint (carpeta `seoberlin/.../IgAiLCRyS...`)

**Antecedente abandonado**: n8n workflow v17 reached production-ready pero hit límites estructurales (Excel Tables empty, jsCode budget, UI version drift, nodo Excel hardcoded a `/me/drive`).

---

## 2. Decisión

### 2.1 Compute y orquestación

**Pure Claude Code, sin orchestrator externo.** El skill `/relevo` orquesta el pipeline entero dentro de la sesión CC del editor. **No hay** Python orchestrator separado, **no hay** remote agents, **no hay** container infra.

El skill expone **tres sub-comandos**:

```
/relevo new           → genera 1 artículo de un topic existente
/relevo content-plan  → genera N topics nuevos al Inventory (pending_review)
/relevo status        → consulta queue y métricas
```

### 2.2 Flow del comando `/relevo new` (generación de artículo)

```
Editor CC → /relevo new
   ↓
AskUserQuestion (deporte / categoría / tema / notas)
   ↓
Pipeline dentro de la misma sesión CC:
  1. Sweep stuck rows (rescue stale in_progress > 30 min)
  2. Lee Inventory + locks topic (pending → in_progress)
  3. Research (WebSearch + URL fetching)
  4. Draft (Claude nativo + style guide + template categoría)
  5. Self-critique (15-point checklist del style guide)
  6. Revise (basado en críticas, max 2 ciclos)
  7. Fact-check (anti-hallucination con WebSearch)
  8. Image gen (Python script Nexern: gpt-image-2 + strip C2PA)
  9. Upload draft a WordPress (status pending_review)
  10. Update state row → "review" + append events a sheet _logs
  11. Notify reviewer (Asana subtask round-robin)
```

### 2.3 Flow del comando `/relevo content-plan` (ideación de topics)

```
Editor CC → /relevo content-plan
   ↓
AskUserQuestion (deporte / categoría / cantidad / focus opcional)
   ↓
Pipeline ideación:
  1. Lee reglamento PDF del deporte desde SharePoint Drive
  2. Lee Inventory actual del deporte (para dedup)
  3. Lee style guide + template específico de la categoría
  4. Claude genera N topics estructurados con metadata SEO completo
  5. Self-check: dedup, validación SEO patterns, naming conventions
  6. Append rows a Inventory con status=pending_review
  7. Asana subtask: "Review batch de N topics — aprobar/editar"
```

**Gate humano clave**: topics nuevos NO pasan directo a status `pending`. Pasan a `pending_review`. El editor jefe revisa el batch, hace bulk-approve en la sheet, y recién ahí quedan disponibles para `/relevo new`.

### 2.4 State y persistencia

| Storage | Función | Editado por |
|---|---|---|
| **SharePoint Excel** (workbook nuevo) | Inventory, logs, metrics, config, dead_letter | Inventory: content team. Logs/Metrics: skill auto. |
| **SharePoint Drive** | 21 PDFs reglamentos | Pablo agrega faltantes |
| **WordPress beta** | Drafts `pending_review` | Skill upload, reviewer publish |
| **Asana** | Subtasks de review + batch-review topics | Skill |
| **JSONL local** `/var/log/relevo/runs.jsonl` | Logs estructurados | Skill append-only |

### 2.5 Sheets del workbook

- `Inventory` — schema unificado (ver 2.6)
- `_logs` — append-only por evento
- `_metrics` — daily roll-up
- `_config` — mapping deporte → URL slug, hub_url, sport canonical names
- `_dead_letter` — fallados 2+ retries

### 2.6 Schema unificado del sheet `Inventory`

Una sola sheet con todas las columnas, null donde no aplica:

| Columna | Aplica a | Ejemplo |
|---|---|---|
| `id` | todos | uuid o secuencial |
| `deporte` | todos | "Fútbol" |
| `categoria` | todos | rules / tactics / origins |
| `tipo` | todos | rules_main, rules_sub, tactics_types, tactics_positions, tactics_strategy, origins |
| `seo_title` | todos | siguiendo fórmula del template |
| `meta_description` | todos | 120-155 chars |
| `url_slug` | todos | "reglas-futbol" |
| `hub_slug` | todos | "futbol" |
| `sub_hub` | todos | "liga-primera" o vacío |
| `nombre` | origins | "Lamine Yamal" |
| `gancho` | origins | "donde nacio y su familia" |
| `priority` | todos | 1-5 |
| `status` | todos | pending_review / pending / in_progress / review / published / dead_letter |
| `notes` | todos | notas del editor |
| `created_at` | todos | timestamp |
| `created_by` | todos | "skill:content-plan" o "human:pablo" |
| `wp_post_id` | post-publish | id en WordPress |
| `quality_score` | post-generate | 0-100 del self-critique |
| `attempts` | todos | counter retries |

### 2.7 Status workflow

```
pending_review  → pending  → in_progress  → review  → published
   ↑ creado por      ↑ aprobado     ↑ /relevo     ↑ generado   ↑ reviewer
   content-plan      por humano     new agarra    OK           publica
                     (gate)
                                    ↓ falla 2x
                              dead_letter
```

### 2.8 Auth y secrets

- **Passbolt** = source-of-truth de secrets
- **Bootstrap creds** en `/etc/relevo/.passbolt-creds` (perms 600, grupo `relevo-content`)
- Skill lee Passbolt al startup
- **Azure AD** (compartida con n8n): cubre Graph completo
- **LiteLLM virtual key** en Passbolt
- **Asana via MCP**: OAuth flow una vez, refresh token cached
- **WordPress**: credenciales pendientes

🔒 Rotación post-MVP requerida: Azure secret + LiteLLM key + Asana secret han pasado por chat.

### 2.9 Abstracción InventoryProvider

Para permitir testing antes que el workbook exista:
- `MockProvider` — lee/escribe CSV local en `data/inventory.csv`
- `SharePointProvider` — lee/escribe via Graph API contra workbook real

Switch por config: `INVENTORY_SOURCE=mock` vs `INVENTORY_SOURCE=sharepoint`.

### 2.10 Distribución al equipo

```
Host AWS compartido:
  /opt/relevo-content/         # repo (git pull central)
  /etc/relevo/.passbolt-creds  # bootstrap secret
  /var/log/relevo/             # logs estructurados
  /var/lib/relevo/             # cache opcional

Cada user del content team:
  ~/.claude/skills/relevo → symlink a /opt/relevo-content/.claude/skills/relevo

Setup IT por usuario: bash bootstrap-relevo.sh (~30 seg)
```

### 2.11 Retry strategy

Stage-level con backoff exponencial: 30s → 2min → 8min. Max 3 attempts.
- **Retriables**: 429, 5xx, network timeout, malformed JSON
- **No retriables**: 401/403/404/400 → dead-letter directo

Stuck detection: cada `/relevo new` arranca con sweep de rows `in_progress` > 30 min sin log activity.

### 2.12 Quality gate

Score 0-100 contra 15-point checklist del style guide. Threshold ≥80 pasa a humano. <80 → revise pass. <80 dos veces → dead-letter.

### 2.13 Imágenes

`generate-images.py` + `strip-ai-metadata.py` de Nexern tal cual. Modelo `gpt-image-2` via LiteLLM. C2PA stripping built-in.

### 2.14 Notificaciones

- **Asana subtask** — único canal MVP
- **Teams** — DEFERRED a v2

### 2.15 Modelos

- Texto: Claude nativo en la sesión del editor (Opus 4.7 1M context asumido)
- Imágenes: `gpt-image-2` via LiteLLM proxy

---

## 3. Consecuencias

### Positivas
- Cero ops infra
- 100% Claude Code UX (alineado mandato C-level)
- Costo marginal ~$60-240/mes
- Reusa stack existente (Passbolt, Azure AD, Asana, SharePoint, image gen)
- Topic ideation + article generation unificados

### Negativas
- Sesión CC editor bloqueada ~10 min/artículo
- Sin Teams notifs MVP
- 2 editores × 25 art/día puede chocar caps del plan CC (a validar)

### Neutrales
- Asana descartado como state DB
- Prompts versionados markdown → PRs para cambios

---

## 4. Alternativas descartadas

| Alternativa | Por qué |
|---|---|
| Container Docker Nexern | Viola mandato "all-in-CC" |
| Python orchestrator separado | Doble pago LLM, sobre-ingeniería |
| Anthropic remote agents | Overkill para 50/día |
| AWS Lambda + Step Functions | 15-min cap fights pipeline |
| n8n v17 continuar | Limitaciones estructurales |
| Teams bot trigger | Deferred a v2 |
| Excel polling trigger | Lento, desordenado |
| CC plugin distribution | Innecesario en shared host |
| Asana-as-state-DB | Asana = solo notifs humanas |

---

## 5. Riesgos a validar empíricamente

| # | Riesgo | Cómo validar | Mitigación |
|---|---|---|---|
| R1 | Caps plan CC con 25 art/día por editor | Métricas día 1 | Upgrade plan |
| R2 | Race condition inventory | Test 2 sesiones paralelas | Optimistic locking |
| R3 | Score self-critique vs aprobación humana | Comparar 20 piezas | Re-calibrar pesos |
| R4 | WP MCP no permite drafts pending_review | Test al recibir creds | Negociar admin |
| R5 | Style guide consume contexto excesivo | Medir tokens primer run | Lazy-load |
| R6 | `gpt-image-2` no disponible en LiteLLM | Test inmediato | Fallback OpenAI |

---

## 6. Alcance MVP

**Incluido**:
- Skill `/relevo` con sub-comandos `new`, `content-plan`, `status`
- Soporte para los **21 deportes con reglamento PDF disponible**
- Las 3 categorías: Rules, Tactics, Origins
- Pipeline multi-pass con retries y quality gate
- Image gen integrado
- MockProvider para testing sin SharePoint
- SharePointProvider para producción
- WordPress upload via MCP
- Asana notifs round-robin
- Logs JSONL + mirror SharePoint

**Excluido (post-MVP)**:
- Teams integration
- `/relevo bootstrap-sport`
- Dashboard de métricas web
- Migración Azure AD app dedicada
- Optimización modelo per-stage

---

## 7. Pendientes pre-producción

1. Spreadsheet nuevo creado con permisos
2. Credenciales WordPress beta
3. Reviewer Asana user IDs
4. Screenshots Relevo para brand visual
5. Path Graph API a reglamentos folder

---

## 8. Sign-off

- [x] Pablo Waimann (Product owner) — **2026-05-14**
- [x] Implementation start: **2026-05-14 post-approval**
