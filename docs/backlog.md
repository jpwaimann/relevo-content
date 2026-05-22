# Relevo Content Workflow — Backlog

Consolidated open items pulled from session memories, README follow-ups, and integration tests through 2026-05-22 EOD. Each item has a **priority** (P0 / P1 / P2 / P3), a **category**, an **owner**, and a brief **why**. The HTML version of this file lives at [`backlog.html`](backlog.html) and is rendered on the public site.

This document is a living artefact. Pruned periodically — items that no longer block the next milestone get downgraded or moved to **Recently resolved**. JPW periodically does a sweep against the 80/20 principle: the top of P0 must be the items that actually move the needle for the next milestone, not a long tail of nice-to-haves.

> See also: [README roadmap](../README.md#roadmap), [project overview](project-overview.html), [test journal](test-journal.html), [one-pager](one-pager.html).

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
- **Owner:** JPW + Joel (Joel reviews; JPW runs the flip)
- **Why:** Nothing downstream runs until topics graduate from `pending_review` to `pending`. The Inventory has had 175 rows (148 original + 27 added via EDIT-07) in `pending_review` since the May 18 bulk seed. `/relevo new` has no material to pick from until at least the first batch is approved.
- **Tools already in place:** `scripts/relevo_approve.py approve --sport <X> --category <Y>` (or `--ids 11,12,13`).
- **Decision pending:** how Joel performs the review at scale. Three options framed on 2026-05-18 EOD: (a) one giant Asana task with all 175 rows; (b) sport-by-sport with a 30-topic first batch; (c) approval in lots via `relevo_approve.py` after a visual review in the Excel.

### `EDIT-03` Workflow live review milestone — May 22
- **Category:** Editorial / Pipeline
- **Owner:** JPW
- **Why:** The Friday May 22 review is the hard close of Phase 3. Demonstrate `/relevo content-plan` and `/relevo new` end-to-end with a real topic, real WP draft, real Asana subtask. Depends on the WP visual gap being fixed.

### `EDIT-12` Batch QA #3 with Joel — validate style guide v1.1.3 end-to-end
- **Category:** Editorial
- **Owner:** Joel (review) + JPW (coordination)
- **Why:** 9 articles regenerated under v1.1.3 published as WP drafts overnight (posts 21195/21198/21201/21204/21207/21210/21213/21216/21219). Asana task `1215038718722890` consolidates the review. Metric target: pass from 2/11 sign-off (batch 2) to ≥ 7/9 sign-off, with the 7 cross-batch patterns reduced to per-article defects only. Expected output: sign-off to scale production OR a v1.2 cycle if a new systemic pattern emerges.

### `SHAREPOINT-01` SharePoint → GitLab content migration
- **Category:** Integrations / Infrastructure
- **Owner:** JPW
- **Why:** The new official repository at [`git.igaming.com/relevo/content`](https://git.igaming.com/relevo/content) replaces the public GitHub mirror as the canonical home for everything Relevo content. All editorial assets currently in SharePoint need to migrate (inventory Excel, rulebook PDFs, ancillary folders under `seoberlin/ES MARKET/Relevo.com/Content/`). Pending decisions: (a) inventory Excel — convert to versioned CSV/JSON or keep SharePoint as source with mirror; (b) reglamento PDFs — git LFS vs external hosting; (c) full audit of the `seoberlin` workspace for assets not yet inventoried.
- **Status:** Audit + scope decisions targeted for Friday 22/5. Execution starts Monday 26/5.

### `WP-VISUAL-GAP` WordPress visual header gap on the beta theme
- **Category:** Pipeline / Theme
- **Owner:** JPW
- **Why:** The `.layout-article-header` of `single.php` shows a visible white space between the hero image lower third and the byline banner across all article drafts on the beta. Content is correct, layout is not. Three CSS iterations on 21/5 did not resolve. Three diagnosis paths queued for Friday morning: (a) dev-tools inspection of the exact gap source, (b) hero aspect ratio change (3:2 → 16:9 / 21:9), (c) refactor of `single.php` to a single-column header if CSS alone cannot fix it.
- **Asana ref:** `1215031100850822` (subtask of Phase 3).

---

## P1 — Quality and robustness gates

### `INTEG-02` Round-robin reviewer pool and SLA
- **Category:** Editorial
- **Owner:** JPW + Joel
- **Why:** Asana subtasks currently all go to Joel because he is the only reviewer in the pool. Once a second reviewer joins, the `asana_notify.py` helper has a round-robin slot but the SLA criteria (return vs approve, response time) are not defined.

### `PIPE-01` Origins category — fresh topic generation
- **Category:** Pipeline
- **Owner:** JPW (decision)
- **Why:** Rules and Tactics topics derive cleanly from the rulebook PDF. **Origins** (athlete biographies) needs current relevance — who is trending, who is breaking out, who recently transferred. The model does not know this without help.
- **Options:** (a) mandatory WebSearch in the Origins flow; (b) editorial curation (the editor pre-fills the candidate list and the skill only generates hooks); (c) sports data API; (d) hybrid.
- **Status:** Deferred during the May 18 bulk seed; the 48 Origins topics there were generated blind. Quality of those should be re-checked.

### `INTEG-03` Azure app — application permissions (path B)
- **Category:** Integrations / Infrastructure
- **Owner:** Debanjan (BI team)
- **Why:** Path A (device-code MSAL) works and is what `scripts/sharepoint_msal.py` uses today. Path B (application permissions + admin consent on the Azure AD app `n8n`) would let server-side automation run without a user session — needed for cron daily reports or any unattended workflow.
- **Status:** Debanjan said on May 18 the grant was applied, but client_credentials still returns `roles: []` and `/sites` + `/shares/.../driveItem` still 401. Pending: screenshot of the API permissions page to verify whether he added Delegated instead of Application, or did not click "Grant admin consent".

### `INFRA-02` Rotate the Azure client secret
- **Category:** Infrastructure
- **Owner:** JPW + Debanjan
- **Why:** The client secret for the Azure AD app `n8n` was pasted into chat on May 15 to verify the credentials. Once path B (`INTEG-03`) lands, rotate the secret in Azure Portal → Certificates & secrets and update Passbolt.

### `INTEG-04` Sites.Selected scope confirmation with CTO
- **Category:** Infrastructure
- **Owner:** JPW + CTO
- **Why:** The `Sites.Selected` permission for the Azure app was requested against the whole `seoberlin` site, which includes 55 sibling folders for other ES Market projects. JPW chose this on May 19 to move faster, but the CTO might prefer narrower scope. Plan B is to create a dedicated `/sites/relevo/` site, move the Content folder, and grant against that. Estimated cost of Plan B: ~45 minutes + script URL updates.

### `INFRA-04` Theme brand visual style for image generation
- **Category:** Pipeline / Editorial
- **Owner:** JPW (define)
- **Why:** `lib/image.md` has a conservative placeholder for `vars.style` ("Premium editorial sports photography. Cinematic. ..."). It works but is generic. When the real brand visual is defined (rim light? colour grade? composition rules?), update that field — zero code change required.

---

## P2 — Post-launch (no commit date, scheduled after FIFA WC)

### `EDIT-09` Automated dedup pass against relevo.com
- **Category:** Pipeline / Editorial
- **Owner:** JPW
- **Why:** The dedup principle is now policy (L2 in `topic_ideation_lineamientos.md`) but automated execution is blocked: relevo.com disallows Anthropic's crawler so the in-tool WebSearch can't query it directly, and DuckDuckGo's HTML search rate-limits past ~1 scripted query before returning captcha. The 3 rugby duplicates Joel explicitly cited (ids 53, 55, 56) were handled inline. A full pass over the remaining ~170 rows needs (a) a Google Custom Search API key (paid), (b) a paid scraping setup, or (c) an internal relevo.com search endpoint. Joel's manual QA covers any clear duplicates in the meantime — non-blocking for launch.

### `INTEG-05` Dynamic regulation fetch for umbrella hubs (Más motor / Más deporte)
- **Category:** Pipeline
- **Owner:** JPW
- **Why:** ADR-004 introduces two umbrella hubs (Más motor with sub_hubs indycar/nascar/formula-e/endurance, Más deporte initially empty). These hubs do not have a single canonical rulebook PDF — each sub-discipline has its own. When the first Rules or Tactics topic lands in an umbrella hub, the command needs an alternative path: fetch the appropriate regulation dynamically via WebSearch or curate per-sub_hub rulebook URLs. Not blocking — current bios for these hubs are Origins only, which uses WebSearch already.

### `INTEG-06` Confirm WP category mapping for new umbrella hubs
- **Category:** Integrations
- **Owner:** JPW
- **Why:** ADR-004 adds the slugs `mas-motor` and `mas-deporte` to the sport list. Before the first publish in either hub, confirm via `posts_meta_set` / theme query that the slug renders under the right category and that breadcrumbs / canonical URLs match the existing website structure. Becomes blocking only when the first umbrella article is queued for publish.

### `EDIT-13` Document the WP upload pattern definitively in `lib/cmd_new.md`
- **Category:** Pipeline / Doc
- **Owner:** JPW
- **Why:** ADR-008 §2.4 documents the canonical pattern (`post_excerpt = entradilla`, `post_content` starts at first `<h2>`) to avoid the theme rendering duplication. Today it is an implicit convention from the ADR; it should become an explicit instruction in the pipeline doc. Post-launch hygiene.

### `EDIT-14` Implement ADR-007 hero image gate-by-text policy
- **Category:** Pipeline / Editorial
- **Owner:** JPW
- **Why:** ADR-007 defines a 2-step human approval flow in Asana for hero images. Today the pipeline generates and uploads without the gate (JPW approved each smoke hero in-line). The gate becomes critical when the batch runs unattended at scale, which is post-launch. Implementation: extend `cmd_new.md` Stage 6 with 2 Asana-gated sub-steps before `wp_upload.py --set-as-thumbnail-for`.
- **Pre-reqs:** Asana MCP fully wired (or a Python helper that creates a subtask + polls for state).

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
- **Why:** Origins topics currently use WebSearch to surface relevant athletes, which risks stale data. A real sports API (SportRadar, API-Football) would give fresh per-sport context. Cost + integration time make it post-launch only.

### `V2-04` Cron daily reports
- **Category:** Pipeline / Editorial
- **Why:** Parked on May 15. Idea: daily summary of queue state, generation throughput, scores, and outliers sent to a Slack / email digest.

### `V2-05` Asana MCP — wire into Claude Code settings.json
- **Category:** Integrations
- **Why:** The `asana_notify.py` helper covers the production need today. A native Asana MCP would let the assistant query and create tasks inline, but the setup requires a Claude Code restart (blast radius), so it is deferred.

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

### `SEO-04` (nice-to-have, downgraded from SEO-02) Change `@type: NewsArticle` to `Article` / `HowTo` for Rules/Tactics
- **Category:** SEO / Schema
- **Owner:** JPW + BI team (theme change)
- **Why:** The theme hard-codes `@type: NewsArticle` for every post. Rules and Tactics articles are evergreen reference content, not news — they would be better classified as `Article` (Rules) or `HowTo` (Tactics). Requires a filter in the theme; not a skill change. Google accepts `NewsArticle` for evergreen content too, so this is purely a precision improvement.

---

## Recently resolved

For completeness — these were open at some point during May and have since closed.

- ✅ **EDIT-11 — Regenerate batch 3 (9 articles) under style guide v1.1.3** (May 21–22 overnight) — 9 articles regenerated in parallel under v1.1.3 (arts 1/3/4/6/7/8/9/10/11 from batch 2). Score range 91–95/100, bolds coverage 0.947–1.00 (all clear of ≥0.80 gate), word range 1939–2636 prose. Heroes generated via gpt-image-2 and attached as featured. All 9 published as WP drafts (posts 21195–21219). Fact-check pass caught 3 baseline errata Joel had not flagged manually (Topuria birthplace city, Iniesta historical role under Rijkaard, Ronaldo Nazário 2002 punterazo). Asana review task `1215038718722890` (=`EDIT-12`) handed off to Joel for QA #3.
- ✅ **INTEG-01 — First real publish E2E** (May 19, late PM) — inventory row 5 (Fútbol / rules / penalti) processed end-to-end through every pipeline stage. Final quality score 91, WP post 16100 in `pending`, inventory advanced to `status=review`. Validated all stages wired together work in sequence on a real topic.
- ✅ **EDIT-06 / EDIT-08 — Topic + Batch QA #2 with Joel** (May 21) — closed, absorbed into **ADR-006** (style guide v1.1.2) + **ADR-008** (style guide v1.1.3). Two reviewer rounds in one day. 7 cross-batch patterns + bolds coverage rule both incorporated. End-to-end editorial validation now passes to `EDIT-12` (batch 3 review).
- ✅ **EDIT-07 — Apply Joel's topic ideation feedback to the inventory** (May 20) — Inventory 148 → 175 rows. 3 elementary topics reformulated, 4 factual bio fixes, 4 angle topics + 22 obligatory bios seeded. Full source breakdown in [`joel-topic-feedback-v1.md`](https://github.com/jpwaimann/relevo-content/blob/main/prompts/joel-topic-feedback-v1.md). Lineamientos durables live in [`topic_ideation_lineamientos.md`](https://github.com/jpwaimann/relevo-content/blob/main/prompts/topic_ideation_lineamientos.md) and [ADR-003](https://github.com/jpwaimann/relevo-content/blob/main/docs/adr/003-topic-ideation-lineamientos.md).
- ✅ **EDIT-04 — Regenerate the 11 articles batch under v1.1 + templates v2** (May 20) — Eleven parallel sub-agents regenerated each article applying per-article factual fixes. Total 24,142 words / 11 articles / zero paragraphs over 60 words. Artifacts in `runs/regen-batch-v2-1779298867/`. Awaited Joel QA #2.
- ✅ **EDIT-05 — Templates v2 rework (Rules/Tactics/Origins)** (May 20) — three category templates reworked to diverge editorially on four axes. Full rationale in [ADR-005](https://github.com/jpwaimann/relevo-content/blob/main/docs/adr/005-templates-v2-rework.md). Validated via smoke test on `tie-break-quinto-set`.
- ✅ **EDIT-10 — Sport-list coverage for Álex Palou** (May 20) — Sport list expanded 24 → 26 with two umbrella hubs (`Más motor` and `Más deporte`). Palou re-classified under `Más motor`. Full rationale in [ADR-004](https://github.com/jpwaimann/relevo-content/blob/main/docs/adr/004-umbrella-hubs-mas-motor-mas-deporte.md).
- ✅ **EDIT-02 — Style guide v1.1** (May 20) — Reviewer feedback on the May 15 batch incorporated into [`prompts/style_guide.md`](https://github.com/jpwaimann/relevo-content/blob/main/prompts/style_guide.md) v1.1. Full details in [ADR-002](https://github.com/jpwaimann/relevo-content/blob/main/docs/adr/002-style-guide-v1-1.md).
- ✅ **First fully autonomous E2E production run** (May 19, late PM) — Validated end-to-end pipeline against a real topic. Quality 91, WP post 16100.
- ✅ **SEO-03 — FAQPage JSON-LD generator** (May 19) — `scripts/seo_jsonld.py` integrated as Stage 7 of the pipeline (`cmd_new.md` step 7.b).
- ✅ **INFRA-03 — Pillow format quirk fix** (May 19) — `scripts/imagen/strip-ai-metadata.py` honours extension as format contract.
- ✅ **INFRA-01 — `post_author` workaround** (May 19) — `posts_create` followed by `posts_update` two-call sequence.
- ✅ **SEO-01 — meta description plumbing** (May 19) — `post_excerpt = meta_description` is sufficient.
- ✅ **SEO-02 — schema.org JSON-LD baseline** (May 19) — Theme emits `BreadcrumbList` + `NewsArticle` out of the box.
- ✅ **WP MCP setup** (May 19) — `ig-mcp-proxy` wired to `relevo-com.sub.plus`.
- ✅ **Image upload helper** (May 19) — `scripts/wp_upload.py` LIVE.
- ✅ **Image generation E2E** (May 18) — `gpt-image-2` via LiteLLM iGaming.
- ✅ **148-topic bulk seed** (May 18) — All 24 sports × 6 categories seeded.
- ✅ **Helpers consolidated** (May 18) — `sharepoint_msal.py`, `relevo_approve.py`, `get_reglamento.py` LIVE.
- ✅ **24/24 rulebooks in SharePoint** (May 18).
- ✅ **Schema reconciliation** (May 18) — `cmd_content_plan.md` aligned with 22-column live schema.
- ✅ **GitHub Pages live** (May 15).
- ✅ **Asana reviewer subtasks** (May 15) — `asana_notify.py` functional.
