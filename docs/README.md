# Relevo Content Workflow

Skill de Claude Code para generación automatizada de contenido SEO en Relevo.com. Cubre 24 deportes en 3 categorías (Rules, Tactics, Origins).

## Quick start (para content editors)

Una vez que IT corrió el bootstrap en tu user del host AWS compartido, simplemente abrí Claude Code en cualquier directorio y usá:

```
/relevo new            # Generar 1 artículo de un topic existente
/relevo content-plan   # Generar N topics nuevos al inventory
/relevo status         # Ver estado de la cola
```

El skill te va a preguntar todo lo que necesita (deporte, categoría, tema, etc.).

## Quick start (para devs)

```bash
# Clonar repo en el host compartido
cd /opt
sudo git clone <repo-url> relevo-content
cd relevo-content

# Setup per-user (cada editor corre esto una vez)
bash scripts/bootstrap-relevo.sh

# Verificar
claude  # Abrir CC
> /relevo status
```

## Arquitectura — resumen ejecutivo

- **Sin server propio**: el skill corre dentro de la sesión CC del editor. Cero infra a operar.
- **3 sub-comandos**: `/relevo new`, `/relevo content-plan`, `/relevo status`
- **Pipeline multi-pass** para `/relevo new`: research → draft → critique → revise → fact-check → image → publish
- **Style guide + templates** versionados en `prompts/` (markdown)
- **State** en SharePoint Excel (mock CSV en `data/inventory.csv` para testing)
- **Imágenes** via toolkit Python `gpt-image-2` + C2PA stripping
- **Notifs** humanas en Asana subtasks (Teams diferido)
- **Logs** en JSONL local + mirror al sheet `_logs`

Detalle completo en `docs/adr/001-architecture.md`.

## Estructura del repo

```
relevo-content/
├── README.md                          # este archivo
├── docs/
│   └── adr/001-architecture.md        # decisión arquitectónica completa
├── prompts/                           # markdown versionado — PRs del content team
│   ├── style_guide.md                 # manual de estilo Relevo (voz, léxico, HTML)
│   ├── rules_template.md              # template categoría Rules & Basics
│   ├── tactics_template.md            # template categoría Tactics Explainers
│   └── origins_template.md            # template categoría Origins (bios)
├── templates/                         # placeholder para v2 (templates JSON)
├── data/
│   ├── inventory.csv                  # mock inventory (MVP)
│   └── sports_config.json             # 24 deportes + slugs + sub_hubs + reglamento availability
├── image_gen/
│   ├── generate-images.py             # script Nexern estándar (gpt-image-2 via LiteLLM o OpenAI)
│   ├── strip-ai-metadata.py           # C2PA stripping
│   ├── jobs.template.json             # estructura tipo de jobs.json
│   └── .env.example                   # template de credenciales
├── .claude/
│   ├── commands/
│   │   └── relevo.md                  # slash command /relevo (router)
│   └── skills/
│       └── relevo/
│           ├── SKILL.md               # entry point del skill
│           └── lib/
│               ├── cmd_new.md         # comando: /relevo new
│               ├── cmd_content_plan.md# comando: /relevo content-plan
│               ├── cmd_status.md      # comando: /relevo status
│               ├── pipeline.md        # etapas del pipeline multi-pass
│               ├── inventory.md       # abstracción InventoryProvider
│               ├── retry.md           # retry strategy + stuck detection
│               ├── logging.md         # logging JSONL + mirror
│               └── image.md           # invocación del image gen toolkit
├── scripts/
│   └── bootstrap-relevo.sh            # setup per-user de symlinks (~30 seg)
└── logs/                              # logs locales (gitignored)
```

## Modes

### Mock mode (default — para testing y MVP sin infra prod)

- Inventory: `data/inventory.csv` (lectura/escritura local)
- WordPress upload: escribe HTML a `/tmp/relevo-{run_id}/article.html`
- Asana notif: escribe JSON a `/tmp/relevo-{run_id}/asana_subtask.json`
- Image gen: real (usa LiteLLM si `.env` configurada)

### Production mode (post-setup completo)

- Inventory: Graph API contra SharePoint Excel workbook
- WordPress: vía MCP, status `pending`
- Asana: vía MCP, subtasks round-robin a reviewers
- Image gen: real con LiteLLM

Switch entre modes: env var `INVENTORY_SOURCE=mock|sharepoint` y secrets en Passbolt.

## Costos

Para 50 artículos/día con 2 editores:
- **Texto**: $0 marginal (cubierto por subscripción CC del editor)
- **Imágenes**: ~$0.04-0.16 × 50/día × 1-2 imgs = **$60-240/mes**
- **API otros (WP/Asana/Graph)**: $0
- **Infra**: $0 (sin server propio)

## Pendientes pre-producción

Ver `docs/adr/001-architecture.md` sección 7.

## Contribuir cambios al prompt o templates

Los prompts viven en `prompts/*.md` y son versionados. Para cambiar la voz o un template:

1. Editar el archivo en una rama
2. PR con justificación + ejemplo de output esperado
3. Code review + test con `/relevo new` antes de mergear

## Quién mantiene esto

Project owner: **Pablo Waimann** (`pablo.waimann@nexern.com`)
Tech: Claude Code skill, sin server propio
IT contact: Debanjan (BI Team) para Azure/SharePoint/Passbolt
