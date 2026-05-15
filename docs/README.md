# Relevo Content Workflow — Skill technical reference

Claude Code skill for automated SEO content generation on relevo.com. Covers 24 sports across 3 categories (Rules & Basics, Tactics Explainers, Origins).

> This document describes the **skill** (the prompt logic, templates, and pipeline). For the project-level overview written for non-engineers, see [project-overview.html](project-overview.html).

## Quick start — content editors

Once IT has run the bootstrap script for your user on the shared AWS host, open Claude Code in any directory and use:

```
/relevo new            # Generate one article from an existing topic in the inventory
/relevo content-plan   # Add N new topics to the inventory (status: pending_review)
/relevo status         # See the queue and recent metrics
```

The skill prompts for whatever it needs (sport, category, topic) interactively.

## Quick start — engineers

```bash
# Clone the repo on the shared host
cd /opt
sudo git clone <repo-url> relevo-content
cd relevo-content

# Per-user setup (each editor runs this once)
bash scripts/bootstrap-relevo.sh

# Verify
claude        # opens the Claude Code CLI
> /relevo status
```

## Architecture — executive summary

- **No server of our own.** The skill runs inside the editor's Claude Code session. Zero infrastructure to operate.
- **Three sub-commands**: `/relevo new`, `/relevo content-plan`, `/relevo status`.
- **Multi-pass pipeline** for `/relevo new`: research → draft → critique → revise → fact-check → image → publish.
- **Style guide + templates** versioned as markdown under `prompts/`.
- **State** lives in a SharePoint Excel workbook (with a CSV mock under `data/inventory.csv` for testing).
- **Images** generated via a Python toolkit using `gpt-image-2` and stripping C2PA metadata.
- **Human notifications** as Asana subtasks (Microsoft Teams integration is deferred).
- **Logs** appended to JSONL locally and mirrored to a `_logs` sheet.

Full architectural rationale lives in `docs/adr/001-architecture.md`.

## Repository layout

```
relevo-content/
├── README.md                          # this file
├── docs/
│   └── adr/001-architecture.md        # full architectural decision record
├── prompts/                           # versioned markdown — content team PRs
│   ├── style_guide.md                 # Relevo style guide (voice, lexicon, HTML)
│   ├── rules_template.md              # Rules & Basics category template
│   ├── tactics_template.md            # Tactics Explainers category template
│   └── origins_template.md            # Origins (biography) category template
├── templates/                         # placeholder for v2 (JSON templates)
├── data/
│   ├── inventory.csv                  # mock inventory (MVP)
│   └── sports_config.json             # 24 sports + slugs + sub-hubs + rulebook availability
├── image_gen/
│   ├── generate-images.py             # Nexern standard image script (gpt-image-2 via LiteLLM or OpenAI)
│   ├── strip-ai-metadata.py           # C2PA stripping
│   ├── jobs.template.json             # canonical shape of jobs.json
│   └── .env.example                   # credentials template
├── .claude/
│   ├── commands/
│   │   └── relevo.md                  # /relevo slash command (router)
│   └── skills/
│       └── relevo/
│           ├── SKILL.md               # skill entry point
│           └── lib/
│               ├── cmd_new.md         # command: /relevo new
│               ├── cmd_content_plan.md# command: /relevo content-plan
│               ├── cmd_status.md      # command: /relevo status
│               ├── pipeline.md        # multi-pass pipeline stages
│               ├── inventory.md       # InventoryProvider abstraction
│               ├── retry.md           # retry strategy + stuck detection
│               ├── logging.md         # JSONL logging + sheet mirror
│               └── image.md           # image-gen toolkit invocation
├── scripts/
│   └── bootstrap-relevo.sh            # per-user setup of symlinks (~30 seconds)
└── logs/                              # local logs (gitignored)
```

## Modes

### Mock mode (default — for testing and MVP without production infra)

- Inventory: `data/inventory.csv` (read/write locally).
- WordPress upload: writes the HTML to `/tmp/relevo-{run_id}/article.html`.
- Asana notification: writes a JSON file to `/tmp/relevo-{run_id}/asana_subtask.json`.
- Image generation: real (uses LiteLLM if `.env` is configured).

### Production mode (after full setup)

- Inventory: Graph API against the SharePoint Excel workbook.
- WordPress: via MCP, status `pending_review`.
- Asana: via MCP, subtasks assigned round-robin to reviewers.
- Image generation: real, via LiteLLM.

Mode is controlled by the `INVENTORY_SOURCE=mock|sharepoint` environment variable and credentials stored in Passbolt.

## Cost

For 50 articles/day with 2 editors:

- **Text**: $0 marginal (covered by the editor's Claude Code subscription).
- **Images**: ~$0.04–$0.16 per image × 50/day × 1–2 images = **$60–$240/month**.
- **Other APIs (WordPress, Asana, Graph)**: $0.
- **Infrastructure**: $0 (no server of our own).

## Pre-production checklist

See `docs/adr/001-architecture.md` section 7.

## Contributing to the prompt or templates

Prompts live in `prompts/*.md` and are versioned. To change the voice or a template:

1. Edit the file in a branch.
2. Open a PR with rationale and an example of the expected output.
3. Code review + a smoke test with `/relevo new` before merging.

## Ownership

Project owner: **Pablo Waimann** (`pablo.waimann@igaming.com`)
Technology: Claude Code skill, no server of our own
IT contact: Debanjan (BI team) for Azure, SharePoint, Passbolt.
