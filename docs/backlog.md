# Relevo Content Workflow — Backlog

Consolidated open items pulled from session memories, README follow-ups, and integration tests through 2026-05-19. Each item has a **priority** (P0 / P1 / P2 / P3), a **category**, an **owner**, and a brief **why**. The HTML version of this file lives at [`backlog.html`](backlog.html) and is rendered on the public site.

This document is a living artefact. As items close they move to the bottom under **Recently resolved**. As new items emerge (during reviewer feedback, infra changes, or v2 design) they get added in their natural priority bucket.

> See also: [README roadmap](../README.md#roadmap), [project overview](project-overview.html), [test journal](test-journal.html).

---

## Priority legend

| Priority | Meaning |
|---|---|
| **P0** | Blocks the **June 1** public launch. Must close by then. |
| **P1** | Quality / robustness gate for the first real publishes. Should close before June 1 but launch can survive a slip. |
| **P2** | Post-launch — scheduled after the FIFA window stabilises. Documented for visibility, no commit date. |
| **P3** | Tech debt and nice-to-have cleanup. No commitment. |

## Category legend

| Category | Scope |
|---|---|
| **Editorial** | Review flow, style guide, content quality decisions |
| **Pipeline** | The `/relevo` skill and its sub-commands |
| **Integrations** | SharePoint, WordPress, Asana, LiteLLM, Azure |
| **SEO / Schema** | Meta tags, structured data, theme alignment |
| **Infrastructure** | Hosts, DNS, certificates, secrets, observability |
| **Documentation** | Public docs, journals, internal references |

---

## P0 — Blocks the June 1 launch

### `EDIT-01` Review flow for the 148 seeded topics
- **Category:** Editorial
- **Owner:** Pablo + Joel (Joel reviews; Pablo runs the flip)
- **Why:** Nothing downstream runs until topics graduate from `pending_review` to `pending`. The Inventory has had 148 rows in `pending_review` since the May 18 bulk seed. `/relevo new` has no material to pick from until at least the first batch is approved.
- **Tools already in place:** `scripts/relevo_approve.py approve --sport <X> --category <Y>` (or `--ids 11,12,13`).
- **Decision pending:** how Joel performs the review at scale. Three options framed on 2026-05-18 EOD: (a) one giant Asana task with all 147 rows; (b) sport-by-sport with a 30-topic first batch; (c) approval in lots via `relevo_approve.py` after a visual review in the Excel.

### `EDIT-02` Style guide v1.1
- **Category:** Editorial
- **Owner:** Joel
- **Why:** The main content writer's feedback on the May 15 batch (10 articles, mean score 90.3) was supposed to land in a v1.1 of `prompts/style_guide.md` before the Friday May 22 workflow review. Phase 4 milestone depends on it.

### `INTEG-01` First real publish E2E
- **Category:** Integrations / Pipeline
- **Owner:** Pablo
- **Why:** The May 19 test reused the May 14 smoke article and pushed it as a draft (post 16091) — that validated the plumbing. The first **production** publish needs an approved topic from EDIT-01, the correct `post_author`, the SEO meta wired in, and the Asana subtask flowing to the actual reviewer.
- **Pre-reqs:** EDIT-01, SEO-01, INFRA-01.

### `EDIT-03` Workflow live review milestone — May 22
- **Category:** Editorial / Pipeline
- **Owner:** Pablo
- **Why:** The Friday May 22 review is the hard close of Phase 4. Demonstrate `/relevo content-plan` and `/relevo new` end-to-end with a real topic, real WP draft, real Asana subtask. The earlier `EDIT-01`/`INTEG-01` items have to feed into this.

---

## P1 — Quality and robustness gates

### `INFRA-01` `post_author` is overridden by the WP adapter
- **Category:** Integrations / Infrastructure
- **Owner:** Pablo (investigation), Pipeline (fix)
- **Why:** On May 19, `posts_create` was called with `post_author: 77` (Pablo's WP user ID) and the saved post had `post_author: 1` (admin default). Either the adapter ignores `post_author` on create, or there is a capability check we are missing.
- **Workarounds available:** (a) call `posts_update` immediately after create with the correct `post_author`; (b) set authorship manually in the admin. Either fine for the immediate publish; the real fix is to understand whether the adapter accepts the field.

### `SEO-01` Meta description plumbing
- **Category:** SEO / Schema
- **Owner:** Pablo + SEO team
- **Why:** The May 19 test stored the meta description in `post_excerpt`, which the Relevo theme **may** read as a fallback. We have not confirmed which key the theme actually reads:
  - `post_excerpt` (current placement)
  - `_yoast_wpseo_metadesc` (Yoast — not detected in `plugins_list`)
  - `ig-optimizer` plugin (present, v6.3.10.2)
- **Action:** confirm with SEO team, then update `cmd_new.md` step 7.b to set the right meta key.

### `SEO-02` Schema.org structured data
- **Category:** SEO / Schema
- **Owner:** Pablo + BI team
- **Why:** The `ig-schema` plugin is **absent** on `relevo-com.sub.plus`. If structured data matters (Article, FAQ, BreadcrumbList for the Rules pages), we need one of:
  - Install `ig-schema` plugin (request to Debanjan / BI team)
  - Generate JSON-LD inline in the article HTML during the pipeline
  - Skip schema for MVP, revisit post-launch

### `INTEG-02` Round-robin reviewer pool and SLA
- **Category:** Editorial
- **Owner:** Pablo + Joel
- **Why:** Asana subtasks currently all go to Joel because he is the only reviewer in the pool. Once a second reviewer joins, the `asana_notify.py` helper has a round-robin slot but the SLA criteria (return vs approve, response time) are not defined.

### `PIPE-01` Origins category — fresh topic generation
- **Category:** Pipeline
- **Owner:** Pablo (decision)
- **Why:** Rules and Tactics topics derive cleanly from the rulebook PDF. **Origins** (athlete biographies) needs current relevance — who is trending, who is breaking out, who recently transferred. The model does not know this without help.
- **Options:** (a) mandatory WebSearch in the Origins flow; (b) editorial curation (the editor pre-fills the candidate list and the skill only generates hooks); (c) sports data API; (d) hybrid.
- **Status:** Deferred during the May 18 bulk seed; the 48 Origins topics there were generated blind. Quality of those should be re-checked.

### `INTEG-03` Azure app — application permissions (path B)
- **Category:** Integrations / Infrastructure
- **Owner:** Debanjan (BI team)
- **Why:** Path A (device-code MSAL) works and is what `scripts/sharepoint_msal.py` uses today. Path B (application permissions + admin consent on the Azure AD app `n8n`) would let server-side automation run without Pablo's user session — needed for cron daily reports or any unattended workflow.
- **Status:** Debanjan said on May 18 the grant was applied, but client_credentials still returns `roles: []` and `/sites` + `/shares/.../driveItem` still 401. Pending: screenshot of the API permissions page to verify whether he added Delegated instead of Application, or did not click "Grant admin consent".

### `INFRA-02` Rotate the Azure client secret
- **Category:** Infrastructure
- **Owner:** Pablo + Debanjan
- **Why:** The client secret for the Azure AD app `n8n` was pasted into chat on May 15 to verify the credentials. The secret ends in `…CVqxKcYr`. Once path B (`INTEG-03`) lands, rotate the secret in Azure Portal → Certificates & secrets and update Passbolt.

### `INTEG-04` Sites.Selected scope confirmation with CTO
- **Category:** Infrastructure
- **Owner:** Pablo + CTO
- **Why:** The `Sites.Selected` permission for the Azure app was requested against the whole `seoberlin` site, which includes 55 sibling folders for other ES Market projects. Pablo chose this on May 19 to move faster, but the CTO might prefer narrower scope. Plan B is to create a dedicated `/sites/relevo/` site, move the Content folder, and grant against that. Estimated cost of Plan B: ~45 minutes + script URL updates.

### `INFRA-03` Image format quirk — Pillow re-encode
- **Category:** Pipeline / Infrastructure
- **Owner:** Pipeline
- **Why:** `scripts/imagen/strip-ai-metadata.py` re-encodes images via Pillow to strip C2PA / JUMBF / OpenAI metadata. The output sometimes ends up as PNG bytes inside a `.jpg` file (extension lies). WordPress and most tools accept it without complaint, but it is technically incorrect.
- **Fix:** make the strip script preserve the original format (detect from input bytes and write PNG to `.png`, JPEG to `.jpg`). Low priority — no functional impact today.

### `INFRA-04` Theme brand visual style for image generation
- **Category:** Pipeline / Editorial
- **Owner:** Pablo (define)
- **Why:** `lib/image.md` has a conservative placeholder for `vars.style` ("Premium editorial sports photography. Cinematic. ..."). It works but is generic. When Pablo defines the real brand visual (rim light? colour grade? composition rules?), update that field — zero code change required.

---

## P2 — Post-launch (no commit date, scheduled after FIFA WC)

### `V2-01` Search volume integration (Ahrefs / SEMrush)
- **Category:** Pipeline
- **Why:** Today topics are generated blind from the rulebook + WebSearch. Adding keyword volume per candidate would let the LLM rank or filter ideation against real search demand.
- **Blocker:** no API access is budgeted; integration would also slow ideation while we tune the prompt.

### `V2-02` Automatic priority logic (P0 / P1 / P2)
- **Category:** Pipeline
- **Why:** Every topic in the inventory is currently `priority = P1` hard-coded. v2 assigns priority automatically based on (a) search volume from `V2-01`, (b) sports calendar (football lifted to P0 during May–June 2026 for the FIFA WC, tennis P0 during Grand Slam weeks), (c) coverage gap analysis.
- **Blocker:** the human-in-the-loop priority review process is not designed yet. Defer until launch is behind us.

### `V2-03` Sports data API feed for Origins ideation
- **Category:** Pipeline
- **Why:** Origins topics currently use WebSearch to surface relevant athletes, which risks stale data. A real sports API (SportRadar, API-Football) would give fresh per-sport context — top scorers, transfers, breakout rookies — and eventually auto-suggest candidates.
- **Blocker:** cost + integration time. WebSearch + editorial curation is enough for the first production batches.

### `V2-04` Cron daily reports
- **Category:** Pipeline / Editorial
- **Why:** Pablo parked this on May 15 ("ya fue, después vemos"). The idea was a daily summary of queue state, generation throughput, scores, and outliers sent to a Slack / email digest.

### `V2-05` Asana MCP — wire into Claude Code settings.json
- **Category:** Integrations
- **Why:** The `asana_notify.py` helper covers the production need today, but a native Asana MCP would let the assistant query and create tasks inline. The setup requires a Claude Code restart (blast radius), so it is deferred until there is a concrete benefit.

---

## P3 — Tech debt and cleanup

### `INFRA-05` Cleanup old Cloudflare tunnel and nginx on the AWS VM
- **Category:** Infrastructure
- **Why:** The May 15 preview tunnel was deprecated when GitHub Pages took over. The tunnel still runs on the VM (`/tmp/relevo-nginx.conf`) until we confirm nothing depends on it.

### `INFRA-06` Cleanup smoke image hosted on the tunnel
- **Category:** Infrastructure
- **Why:** `https://challenge-gnome-kiss-beginning.trycloudflare.com/smoke-image-2026-05-18/` was set up to share the image-gen smoke output. Becomes obsolete once the tunnel is decommissioned.

### `DOC-01` Confirm Tiziana is fully removed from team references
- **Category:** Documentation
- **Why:** Tiziana left the project on May 18. Cross-check that she is not referenced in weekly reports, participants lists, or phase docs.

### `INFRA-07` DNS resiliency — `relevo-com.sub.plus` host entry
- **Category:** Infrastructure
- **Why:** A line in `/etc/hosts` maps `relevo-com.sub.plus → 192.168.20.187`. The provider's managed IP is unlikely to change, but if it does (or if the VM is rebuilt), regenerate via `dig relevo-com.sub.plus +short` from a host on the corporate VPN and edit `/etc/hosts`. No action needed unless it breaks.

---

## Recently resolved

For completeness — these were open at some point during May and have since closed.

- ✅ **WP MCP setup** (May 19) — `ig-mcp-proxy` wired to `relevo-com.sub.plus`, identity + connection persisted in `~/.ig-mcp/config.json`.
- ✅ **Image upload helper** (May 19) — `scripts/wp_upload.py` validates against full-size 1024×1024 originals, no recompression needed. Eliminates the base64-inline ceiling.
- ✅ **Image generation E2E** (May 18) — `gpt-image-2` via LiteLLM iGaming, C2PA / JUMBF / OpenAI markers stripped, integrated into `scripts/imagen/`.
- ✅ **148-topic bulk seed** (May 18) — all 24 sports × 6 categories seeded into the Excel inventory, status `pending_review`.
- ✅ **Helpers consolidated** (May 18) — `sharepoint_msal.py` refactor (`update_table_row`, retry), `relevo_approve.py` LIVE, `get_reglamento.py` LIVE.
- ✅ **24/24 rulebooks in SharePoint** (May 18) — Cycling UCI, Paralympics IPC, FIM GP added; `_pdf_name` validated against Graph for all 24 sports.
- ✅ **Schema reconciliation** (May 18) — `cmd_content_plan.md` rewritten against the 22-column live Excel schema (option A executed).
- ✅ **GitHub Pages live** (May 15) — public roadmap at `jpwaimann.github.io/relevo-content/`.
- ✅ **Asana reviewer subtasks** (May 15) — `asana_notify.py` functional with round-robin pool.
