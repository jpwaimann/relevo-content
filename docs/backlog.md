# Relevo Content Workflow — Backlog

Consolidated open items pulled from session memories, README follow-ups, and integration tests through 2026-05-25 PM. Each item has a **priority** (P0 / P1 / P2 / P3), a **category**, an **owner**, and a brief **why**. The HTML version of this file lives at [`backlog.html`](backlog.html) and is rendered on the public site.

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

### `EDIT-16` Regenerate batch 4 — 7 articles from batch 3 under style guide v1.2
- **Category:** Editorial / Pipeline
- **Owner:** JPW (orchestration) + skill subagents (generation) + Joel (review under EDIT-17)
- **Why:** Joel batch 3 verdict (25/5 11:15 UTC, Asana `1215038718722890`): 2/9 ✅ (art-4, art-8) + 7/9 ⚠️ (arts 1, 3, 6, 7, 9, 10, 11). Same cross-batch patterns as batch 2 reappeared — v1.1.3 quantitative metric was met (95%+ bolds coverage) but the qualitative ones (decorative bolds, grandiloquent phrases, factual errors, etc.) were not. ADR-010 + style guide v1.2 refactor the 8 rules as binary pass/fail tests. Regen-batch methodology applies: regenerate the 7 ⚠️ under v1.2, NOT per-article fixes.
- **Pre-reqs:** Smoke v1.2 (1 article from the 7, suggested: art-6 Alcaraz because it concentrates 4 of the 8 patches — factual errors + grandiloquence + decorative bolds + closing). JPW signs off on smoke before launching the 7. art-4 + art-8 stay as published drafts with manual fine edits — no regen needed.

### `EDIT-17` Batch QA #4 with Joel — validate style guide v1.2 end-to-end
- **Category:** Editorial
- **Owner:** Joel (review) + JPW (coordination)
- **Why:** After EDIT-16, Joel reviews the 7 regenerated drafts in WP beta. Same Asana-subtask-per-article pattern as EDIT-12. Target: ≥ 6/7 sign-off (vs 2/9 in batch 3). If achieved → v1.2 is the production line that sustains until launch 01/06. If < 6/7 → v1.3 cycle before launch, with risk to the June 1 hard deadline.

### `SHAREPOINT-01` SharePoint → GitLab content migration
- **Category:** Integrations / Infrastructure
- **Owner:** JPW
- **Why:** The new official repository at [`git.igaming.com/relevo/content`](https://git.igaming.com/relevo/content) replaces the public GitHub mirror as the canonical home for everything Relevo content. All editorial assets currently in SharePoint need to migrate (inventory Excel, rulebook PDFs, ancillary folders under `seoberlin/ES MARKET/Relevo.com/Content/`). Pending decisions: (a) inventory Excel — convert to versioned CSV/JSON or keep SharePoint as source with mirror; (b) reglamento PDFs — git LFS vs external hosting; (c) full audit of the `seoberlin` workspace for assets not yet inventoried.
- **Status:** Audit + scope decisions targeted for Friday 22/5. Execution starts Monday 26/5.

---

## P1 — Quality and robustness gates

### `TEMPLATE-04` Coach Era & Trophies — 4th category template
- **Category:** Editorial / Pipeline
- **Owner:** JPW (define) + Claude (implement under JPW spec)
- **Status (2026-05-25 PM):** Template **shipped** (commit `40689b5`). Pending integration items remain — see "Open items" below.
- **Why:** Joel + JPW introduced a 4th content category for coach-trophy enumeration pages (`titulos-ganados-{equipo}-{entrenador}`). Base template lived in SharePoint at `seoberlin/ES MARKET/Relevo.com/Content/Relevo - Coach Era & Trophies [Base Template].docx` (35 KB) and was adapted to Relevo's prompt style by mirroring `origins_template.md` v2.0. Editorial angle: `"Los títulos que ha ganado {equipo} con {entrenador} al frente"` — enumerative-narrative, 800–1.200 words, narrative scaffolding around a chronological trophy list, era identity carries the piece (cholismo, tiki-taka, mourinhismo). Applies primarily to Fútbol (club + selección), then Baloncesto (NBA / ACB), F1 (team principal), Rugby. NOT for individual sports (tenis, atletismo, MMA, golf).
- **Shipped (2026-05-25):**
  1. ✅ `prompts/coaches_template.md` v1.0 (355 lines) — mirrors `origins_template.md` v2.0 structure: changelog, qué cubre, foco editorial vs other categories, URL pattern, SEO title formulas, meta description formulas, variable inventory (18 vars: `{entrenador}`, `{equipo}`, `{identidad_era}`, `{total_titulos}`, `{total_club_historico}`, `{predecesor}`, `{rol_doble}`, `{rival_clasico}`, slugs), editorial structure (3-para intro era-setting → H3 chronological trophy walk → H3 palmarés bullet → optional player-coach section → H2 5 standard FAQs), hard rules, internal-linking rules, sport-specific adaptations, Simeone/Atlético sample resumido. Inherits style guide v1.2 (25-point self-check).
  2. ✅ `docs/adr/011-coaches-template-v1-0.md` — documents the decision, editorial differentiation vs Rules/Tactics/Origins (table comparing voice, central question, structure, tone, word count), alternatives considered + reversibility, sport-specific adaptations.
- **Open items (not yet shipped):**
  - `lib/cmd_new.md` integration — extend the category switch to handle the `Coaches` branch like Rules/Tactics/Origins. Depends on `INVENTORY-02` (schema extension).
  - Hero image prompt variant for coach portraits (less action-oriented than athlete prompts — on the touchline, in the dugout, lifting the trophy).
  - First seed pieces: Simeone/Atlético (canonical), Ancelotti/Real Madrid, Guardiola/Barça as smoke runs once the `cmd_new.md` branch lands.
- **Pre-reqs for full deployment:** `INVENTORY-02` (schema extension).

### `INVENTORY-02` Extend Inventory schema for Coaches + Newsworthy + Betting
- **Category:** Pipeline / Data model
- **Owner:** JPW
- **Why:** Constraint from JPW: one and only one Excel inventory holds all content the project produces. The current schema (22 columns, ADR-001) was designed for Rules / Tactics / Origins. New content types coming in (Coaches via `TEMPLATE-04`, Newsworthy via `NEWS-02`, Betting / Casino pages via `BETTING-01`) need either new categorical values in `category` and/or new columns specific to their content type. Open design questions: (a) do `equipo` / `entrenador` go in `nombre` / `sub_hub`, or do they need their own columns? (b) does the newsworthy category need a `publish_window` field for time-sensitive pieces? (c) do betting / casino pages need a `comparison_set` column (which operators to compare)? Decision drives the rest of the workstreams.
- **Pre-reqs:** None — JPW can decide unilaterally and document via an ADR.

### `NEWS-01` Share the GSC enriched URLs spreadsheet
- **Category:** Editorial / Data
- **Owner:** JPW + Joel
- **Why:** The file `urls_to_keep_gsc_enriched.xlsx` (SharePoint: `seoberlin/ES MARKET/...`) holds the URLs where Relevo already ranks well. It is a high-signal input for ideation: any topic that Relevo already covers and ranks for should not be duplicated, and any near-miss is a candidate for a refresh / expansion piece. Joel and JPW need to look through it together and pick what feeds into the inventory next.
- **Status:** Share happens this week. After review, ideation candidates derived from it become new `pending_review` rows in the Inventory under the appropriate category (Origins refresh, Tactics expansion, or new Newsworthy entries).

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

### `PIPE-02` Sonnet smoke test — validate Option B/C model split
- **Category:** Pipeline
- **Owner:** Claude (run) · JPW (verdict)
- **Why:** [ADR-015](adr/015-anti-limit-production-strategy.md) is shipping under **Option A** (Opus draft+critique+fact-check, Sonnet only for research+revise — ~40-45 articles/day capacity). Options B (tiered Opus-draft on tier-1 only) and C (Sonnet draft everywhere except critique) could lift capacity to ~60-120/day but introduce voice/quality risk. Decide whether to migrate after the current Opus-A batch closes.
- **Mechanics:** 1 smoke run on 1 Tenis row with Sonnet 4.6 substituted in the draft stage. Same critique gates (score ≥85, all binary tests pass). Compare against an Opus-drafted control on a similar row from today's batch. If Sonnet draft passes critique on the first round + scores within 3 points of the Opus control → propose Option B (Sonnet draft for non-tier-1 only). If Sonnet draft requires 2+ revise rounds OR drops below 85 → keep Option A and shelve B/C.
- **Status:** Scheduled — runs **after** today's Option A batch finishes, before any decision on subsequent batches.

### `INFRA-08` Finalize-as-CLI — push WP+Asana+inventory into sub-agent scope
- **Category:** Pipeline / Infrastructure
- **Owner:** Claude (engineering)
- **Why:** Today the sub-agent stops at `article_final.json`; the main session does the final 4 steps (WP hero upload, WP post create, WP post update for author fix, Asana subtask, inventory advance). This costs ~25k Opus per article on the main session — the binding-constraint bucket per [ADR-015](adr/015-anti-limit-production-strategy.md). If those steps move into CLI scripts that sub-agents can run via Bash, the main-session cost drops to near zero and per-window capacity rises ~25-30% on top of the model-split win.
- **Mechanics:** The `ig-mcp-proxy` is a stdio process; any Python script can subprocess it. `scripts/wp_upload.py` and `scripts/asana_notify.py` already do this. Wrappers shipped on 2026-05-28: `scripts/wp_post.py` (mirrors `posts.create` / `posts.update` over stdio) and `scripts/finalize_article.py` (orchestrator running upload + create + author fix + Asana subtask + inventory advance with per-step `finalize_result.json` checkpoint).
- **Status:** SHIPPED 2026-05-28 PM (commit pending). Smoke test before first production use.

### `INFRA-09` LiteLLM helper — Gemini 2.5 Pro (research) + GPT-5 (fact-check)
- **Category:** Pipeline / Infrastructure
- **Owner:** Claude (engineering)
- **Why:** Real token-reduction levers selected by JPW on 2026-05-28 PM (see [ADR-015 update](adr/015-anti-limit-production-strategy.md#update-2026-05-28-pm-option-a-original-architecture-failed-revised-plan)). Original Option A (nested Sonnet sub-agents) failed in this harness because the Agent tool is not available inside sub-agents. Bucket-shifting to LiteLLM Opus 4.6 was rejected on the grounds that it disguises consumption rather than reducing it. The selected real-reduction combination is: Gemini 2.5 Pro for the research stage (~10x cheaper per token), GPT-5 for the fact-check stage (lower per-token cost than Opus), plus prompt caching on `style_guide.md` + templates (~90% input discount on repeated reads).
- **Mechanics:** `scripts/llm_call.py` is the LiteLLM wrapper. Reads prompt from a file (avoids inlining 30k+ token style guides into Bash args). Calls `litellm.bidwhig.com` Chat Completions endpoint with `cache_control` on the cacheable style-guide prefix where the upstream supports it. Returns generated text + a `usage` block (input_tokens, output_tokens, cached_input_tokens) that the sub-agent merges into the article's running `token_usage` artifact.
- **Models exposed by the active virtual key (`sk-7avXxrt...`, validated 2026-05-28):** GPT-5 family + Claude Opus 4.5/4.6 + Gemini 2.5 Pro/Flash + Gemini image gen. NO Claude Sonnet on any current key (memory `reference_litellm_proxy.md` notes the path is to request a new virtual key with Sonnet access from Debanjan if needed).
- **Pre-req:** None. INFRA-08 is independent and ships in parallel.
- **Status:** SHIPPED 2026-05-28 PM. `scripts/llm_call.py` validated end-to-end on smoke (row 16) + Wave 2 + Wave 3 (8 articles total). Gemini 2.5 Pro and GPT-5 both callable; usage block captured per article (input/output/cached tokens).

### `PIPE-02` superseded by INFRA-09 path
- **Status:** Superseded 2026-05-28. The Sonnet smoke is no longer the gating experiment — the architecture chose Gemini for research and GPT-5 for fact-check (INFRA-09), bypassing the Sonnet question. The Asana task `1215199406629492` reflects the original PIPE-02 plan and can be closed at next review.

### `FINANCE-01` LiteLLM multi-model spend reporting
- **Category:** Finance / Operations
- **Owner:** JPW (drive) · Nexern team (visibility)
- **Why:** The multi-model pipeline routes some stages to LiteLLM (Gemini + GPT-5 calls billed to Nexern's accounts, not JPW's Claude Code Team seat). The team needs visibility on monthly spend attributable to the Relevo project and confirmation that the routing is acceptable from a budget perspective. Asana task `1215215371482362` tracks the open question with the internal team.
- **Status:** Open. Does not block production.

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

### `NEWS-02` Newsworthy / YouTube / Mundial ideation flow into the Inventory
- **Category:** Editorial / Pipeline
- **Owner:** Joel + JPW (ideation) · Gustavo (SEO templates) · Claude (drafting)
- **Why:** Beyond the evergreen Rules / Tactics / Origins / Coaches content, the project will produce more newsworthy / atado-a-la-actualidad pieces driven by the sport calendar (Mundial WC focus, transfer windows, big finals). Initial concepts JPW + Joel raised on 2026-05-25: `Predicciones Plus` (betting predictions with editorial analysis), `Por qué el {Mundial X} fue el peor / mejor`. Source seeds: YouTube watch lists, editorial intuition, GSC near-misses from `NEWS-01`. Constraint: rows live in the SAME Inventory Excel under a new category value (`Newsworthy` or per-format more granular).
- **Mechanics to design:** (a) seeds enter as `pending_review` rows like every other content type; (b) Gustavo turns each approved seed into an SEO template per format (`Predicciones Plus`, `Mundial retrospectiva`, etc.) that lives in `prompts/`; (c) Claude reads the seed + the per-format template and generates the article like any other category; (d) Asana review goes to Joel like every other piece.
- **Pre-reqs:** `INVENTORY-02` (schema extension), 1+ SEO template per newsworthy format authored by Gustavo.

### `BETTING-01` Betting + casino pages (payment methods, fast withdrawals, best casinos)
- **Category:** Editorial / Pipeline
- **Owner:** JPW + Joel (editorial line) · Gustavo (SEO templates inspired by live iGaming sites)
- **Why:** The site needs the classic betting / casino reference pages (payment methods, withdrawal speed comparisons, best casino lists, etc.). Reference: other live iGaming.com sites already cover this format and can be used as the template inspiration. Constraint: rows live in the SAME Inventory Excel under a new category value (`Betting` or per-page-type more granular). NOT a sport page — distinct content type with different sourcing (operator data, regulator references) and different SEO objective.
- **Mechanics to design:** (a) SEO template per page type (mirror what's working on sister sites); (b) data source for operator data — manual or via partner feed; (c) compliance / legal review gate before publish (regulatory implications); (d) maintenance cadence — these pages need periodic refresh as operators change terms.
- **Pre-reqs:** `INVENTORY-02` (schema extension), legal / compliance sign-off on the content type, decision on data sourcing.

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

### `HEADER-REDESIGN` Header redesign with fixed-height + 16:9 hero — Phase 4
- **Category:** Pipeline / Theme / Image generation
- **Owner:** JPW
- **Why:** The `.layout-article-header` of `single.php` shows a visible white gap in 2-column desktop layout because the hero image (generated 3:2, 1024×683) is naturally taller than the text block. Five CSS attempts during Friday 22/5 diagnosis confirmed no CSS-only fix preserves all three of: 2-col layout, full uncropped image, current 3:2 aspect. Root cause and Phase 4 plan documented in [ADR-009](adr/009-wp-visual-header-gap-deferred.md). On 22/5 EOD the CSS was reverted to the original state (gap visible, known/stable) and the fix deferred to Phase 4 to avoid contaminating Joel's batch 3 QA over the weekend.
- **Phase 4 plan (3 coordinated steps, ~45 min total execution):**
  1. CSS: `.layout-article-header` gets `min-height: 350px` (breakpoint-tuned) + text column gets `display: flex; flex-direction: column; justify-content: center;`. ~10 min.
  2. Image generation: change `scripts/imagen/` / `lib/image.md` from 3:2 (1024×683) to 16:9 (1792×1024). Smoke test with 1 article. ~15 min.
  3. Regenerate batch 3 hero images (10) and swap featured images on posts 21195–21219 via WP MCP. ~20 min.
- **Smoke test required before bulk regen.** A single article must run end-to-end with the new image size before touching the 10 production drafts.
- **Asana ref:** new subtask under Phase 4 to be created (replaces the Phase 3 subtask `1215031100850822` which closed with the deferral decision).

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

- ✅ **SOP-01 — Content Production Pipeline SOP v1.0** (May 25, late PM) — Documented the existing pipeline in [`docs/sop/01-content-production.md`](sop/01-content-production.md). Covers the 6 stages (ideation → topic approval → article generation → editorial review → publish decision → post-publish), the 3 roles (Claude / Joel / JPW), the HITL gates (Stages 1, 3, 4), the Asana conventions, the single-Inventory constraint, and a per-role daily reference. The pipeline itself does not change — this SOP formalizes the roles so production can scale without ambiguity at the handoffs.
- ✅ **Smoke v1.2 — art-6 Alcaraz Origins** (May 25, 20:21 UTC) — Score 92/100, verdict GO. 25-point self-check passes 25/25. Hard gates 11/11 green (bolds coverage 95.7%, zero paragraphs > 60 words, M1000 list verified — Cincinnati 2023 + Pekín 2024 corrected, table HTML structure, list internal consistency, closing without prefab connectors). WP draft `21204` updated in-place. Asana subtask [`1215104026151305`](https://app.asana.com/1/9544544431392/project/1211672636050481/task/1215104026151305) re-assigned to Joel after JPW initial approval. Sits as the gate for batch 4 launch tomorrow (`EDIT-16`).
- ✅ **Style guide v1.2 + ADR-010 — binary tests refactor** (May 25, 12:55 UTC) — 8 patches on `prompts/style_guide.md` reframing the rules as pass/fail checks (valor scan for bolds, valor cero for grandiloquence, binary paragraph word count, list consistency, table HTML structure, verifiable lists for facts, cursivas inventory scan, actionable closing rules). Self-check 22 → 25 points. Commit `126a6e1`. Trigger: Joel batch 3 verdict (7/9 ⚠️) showing the qualitative rules were not actionable.
- ✅ **EDIT-12 — Batch QA #3 with Joel under style guide v1.1.3** (May 25, 11:15 UTC) — Joel delivered the verdict on 9 drafts (Asana `1215038718722890`): 2/9 ✅ (art-4, art-8) + 7/9 ⚠️ (arts 1, 3, 6, 7, 9, 10, 11). Same cross-batch patterns as batch 2 reappeared despite v1.1.3 — confirming that the rules were not yet actionable as binary checks. Two factual errors flagged (art-6 Cincinnati 2023 + Pekín 2024 incorrectly attributed to Alcaraz; art-9 Halle geographically misplaced). Verdict drove [ADR-010](adr/010-style-guide-v1-2-binary-tests.md) (style guide v1.2 — 8 patches refactoring rules as binary tests) and spawned `EDIT-16` (regen batch 4 under v1.2) + `EDIT-17` (QA #4 with Joel).
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
