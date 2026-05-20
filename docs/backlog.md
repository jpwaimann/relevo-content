# Relevo Content Workflow — Backlog

Consolidated open items pulled from session memories, README follow-ups, and integration tests through 2026-05-20. Each item has a **priority** (P0 / P1 / P2 / P3), a **category**, an **owner**, and a brief **why**. The HTML version of this file lives at [`backlog.html`](backlog.html) and is rendered on the public site.

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

### `EDIT-04` Regenerate the 11 articles of batch QA #1 under prompts v1.1
- **Category:** Editorial / Pipeline
- **Owner:** Pablo
- **Why:** After the reviewer's batch QA #1 feedback (Asana task `1214815492685881`) was incorporated as style guide v1.1 on May 20, the original 11 articles are obsolete. They must be regenerated under the new prompts before any of them ships, including a WebSearch fact-check pass to fix the per-article factual errors (Alcaraz palmarés through May 2026, Topuria as lightweight after the Oliveira KO, Wenger law tested in the Canadian league, etc. — see `prompts/joel-feedback-v1.md` for the per-article list). Closes the Phase 4 quality loop.
- **Pre-reqs:** EDIT-01 (so the regenerated topics flow through the normal `pending` queue), or run them as one-off `/relevo new` calls against the existing topic IDs.

### `EDIT-05` Rework the three category templates — ADR-003
- **Category:** Editorial
- **Owner:** Pablo (decision) + Joel (review)
- **Why:** Defect #11 of the batch QA #1 feedback: Rules, Tactics, and Origins articles all read with the same structural logic when they should diverge. Biographies need scenes, conflict, and direct quotes; rules need worked examples and current debates; tactical guides need a "when used / who represents it / what changed" frame. v1.1 of the style guide captures the transversal fixes; the category-specific ones belong in the templates. To be tracked as `docs/adr/003-templates-rework.md` when started.

### `INTEG-01` First real publish E2E
- **Category:** Integrations / Pipeline
- **Owner:** Pablo
- **Why:** The May 19 test reused the May 14 smoke article and pushed it as a draft (post 16091) — that validated the plumbing. The first **production** publish needs an approved topic from EDIT-01, the correct `post_author`, the SEO meta wired in, and the Asana subtask flowing to the actual reviewer.
- **Pre-reqs:** EDIT-01, SEO-01, INFRA-01.

### `EDIT-03` Workflow live review milestone — May 22
- **Category:** Editorial / Pipeline
- **Owner:** Pablo
- **Why:** The Friday May 22 review is the hard close of Phase 4. Demonstrate `/relevo content-plan` and `/relevo new` end-to-end with a real topic, real WP draft, real Asana subtask. The earlier `EDIT-01`/`INTEG-01`/`EDIT-04` items have to feed into this.

### `EDIT-06` Batch QA #2 — validate style guide v1.1 with the reviewer
- **Category:** Editorial
- **Owner:** Joel (review) + Pablo (coordination)
- **Why:** The first round of reviewer feedback drove style guide v1.1 (see [editorial guide](editorial-guide.html) and [ADR-002](https://github.com/jpwaimann/relevo-content/blob/main/docs/adr/002-style-guide-v1-1.md)). After `EDIT-04` regenerates the 11 articles under the new prompts, a second QA cycle with the reviewer validates that v1.1 actually addresses the 12 systematic defects without regressing what already worked. Output: either a v1.2 patch or sign-off to scale production.
- **Pre-reqs:** EDIT-04.

### `EDIT-07` Apply Joel's topic ideation feedback to the inventory
- **Category:** Editorial / Pipeline
- **Owner:** Pablo
- **Why:** Joel passed a second-pass feedback (via Teams, May 20 PM) on the **ideation** side — the 148 topics seeded into the Excel inventory on May 18. The general principles are now codified as [`topic_ideation_lineamientos.md`](https://github.com/jpwaimann/relevo-content/blob/main/prompts/topic_ideation_lineamientos.md) and injected into `/relevo content-plan` (see [ADR-003](https://github.com/jpwaimann/relevo-content/blob/main/docs/adr/003-topic-ideation-lineamientos.md)). What remains is the **tactical** application to the current inventory: reformulate 3 elementary topics for tier-1 sports (duración, penalti, tie-break) into angle-driven versions, fix 4 factual errors in bios (Topuria/Halle, Lin futsal vs goalkeeper, Marc Márquez 125cc, Hugo González de Oliveira disambiguation), run a dedup pass against relevo.com, and append the 4 angle topics + ~22 bios that Joel suggested. Full breakdown in [`joel-topic-feedback-v1.md`](https://github.com/jpwaimann/relevo-content/blob/main/prompts/joel-topic-feedback-v1.md).

### `EDIT-08` Topic QA #2 — validate inventory after EDIT-07
- **Category:** Editorial
- **Owner:** Joel (review) + Pablo (coordination)
- **Why:** Once EDIT-07 has applied all the tactical fixes (reformulations, factual corrections, dedup, expansion), Joel does a second pass to confirm the inventory is clean before any further `/relevo new` runs at scale. Same pattern as EDIT-06 but for the ideation side instead of the writing side.
- **Pre-reqs:** EDIT-07.

### `EDIT-09` Automated dedup pass against relevo.com
- **Category:** Pipeline / Editorial
- **Owner:** Pablo
- **Why:** The dedup principle is now policy (L2 in `topic_ideation_lineamientos.md`) but automated execution is blocked: relevo.com disallows Anthropic's crawler so the in-tool WebSearch can't query it directly, and DuckDuckGo's HTML search rate-limits past ~1 scripted query before returning captcha. The 3 rugby duplicates Joel explicitly cited (ids 53, 55, 56) were handled inline as part of EDIT-07. A full pass over the remaining ~170 rows needs either (a) a Google Custom Search API key (paid), (b) a paid scraping setup that handles DDG's anti-bot, or (c) an internal relevo.com search endpoint if one exists. Joel's `EDIT-08` QA #2 catches any clear duplicates manually in the meantime.

### `EDIT-10` Decide sport-list coverage for IndyCar / Álex Palou
- **Category:** Editorial / Pipeline
- **Owner:** Pablo
- **Why:** Álex Palou (3x IndyCar champion, biggest Spanish name in US single-seaters) is in Joel's list of mandatory bios but does not fit cleanly into the current 24-sport list. The closest existing slot is Fórmula 1, but he competes in IndyCar — a different series. Three options: (a) classify Palou under F1 with an explanatory note in his bio, (b) expand the sport list to include IndyCar (or a broader "Automovilismo" umbrella covering F1 + IndyCar + Rallies), (c) drop Palou. Until Pablo decides, the Palou row is `status=blocked` with the rationale in `notes`.

---

## P1 — Quality and robustness gates

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

### `SEO-04` (nice-to-have, downgraded from SEO-02) Change `@type: NewsArticle` to `Article` / `HowTo` for Rules/Tactics
- **Category:** SEO / Schema
- **Owner:** Pablo + BI team (theme change)
- **Why:** The theme hard-codes `@type: NewsArticle` for every post. Rules and Tactics articles are evergreen reference content, not news — they would be better classified as `Article` (Rules) or `HowTo` (Tactics). Requires a filter in the theme; not a skill change. Google accepts `NewsArticle` for evergreen content too, so this is purely a precision improvement.

---

## Recently resolved

For completeness — these were open at some point during May and have since closed.

- ✅ **EDIT-02 — Style guide v1.1** (May 20) — reviewer feedback on the May 15 batch (Asana task `1214815492685881`, 12 subtasks, 23 attachments) landed and was incorporated into [`prompts/style_guide.md`](https://github.com/jpwaimann/relevo-content/blob/main/prompts/style_guide.md) v1.1. The 12 systematic editorial defects identified across the 11 articles drove 11 new rules; the twelfth (template divergence) is deferred to ADR-003 / `EDIT-05`. The biggest single change is the reinterpretation of bold formatting: 5-7 consecutive words forming a complete idea, one per paragraph — not scattered single words. Full details in the public [editorial guide](editorial-guide.html) and [ADR-002](https://github.com/jpwaimann/relevo-content/blob/main/docs/adr/002-style-guide-v1-1.md). Source feedback consolidated at [`prompts/joel-feedback-v1.md`](https://github.com/jpwaimann/relevo-content/blob/main/prompts/joel-feedback-v1.md).
- ✅ **First fully autonomous E2E production run** (May 19, late PM) — inventory row 5 (Fútbol / rules / penalti) processed end-to-end through every pipeline stage without human intervention. Final quality score 91, WP post 16100 in `pending`, inventory advanced to `status=review`. Validated that all stages wired together work in sequence on a real topic. Two minor invocation-side issues caught and resolved (`update_table_row` argument shape, schema column indexing for hand-written calls). Full details in [test journal](test-journal.html).
- ✅ **SEO-03 — FAQPage JSON-LD generator** (May 19) — implemented `scripts/seo_jsonld.py` with deterministic FAQ detection and idempotent injection. Integrated into the skill as Stage 7 of the pipeline (`cmd_new.md` step 7.b). Smoke-tested against the penalti article: detected the 6 FAQ questions, emitted a clean `FAQPage` JSON-LD block, and skipped re-injection on a second run.
- ✅ **INFRA-03 — Pillow format quirk fix** (May 19) — updated `scripts/imagen/strip-ai-metadata.py` to honour the file extension as the format contract (`.jpg` in → JPEG out, `.png` in → PNG out). Added a `normalise` path that fixes extension/content mismatches even when the file has no AI metadata markers. Smoke-tested: a PNG-bytes-inside-.jpg file was correctly rewritten to JPEG bytes.
- ✅ **INFRA-01 — `post_author` workaround** (May 19) — confirmed that `posts_create` ignores `post_author` silently (writes admin default), but `posts_update` immediately after create accepts and persists the field correctly. Skill step 7.d now does the two-call sequence. Post 16096 verified: create returned `post_author=1`, update with `post_author=77` returned `post_author=77`. Re-verified under real conditions during the autonomous E2E run (post 16100).
- ✅ **SEO-01 — meta description plumbing** (May 19) — verified against a live published post (5608) that the Relevo theme renders `og:description` directly from `post_excerpt` and does not emit `<meta name="description">` at all. No SEO plugin is in use (Yoast / Rank Math / AIO SEO all absent in `plugins_list`). Conclusion: setting `post_excerpt = meta_description` in `posts_create` is sufficient — no custom meta keys required.
- ✅ **SEO-02 — schema.org JSON-LD baseline** (May 19) — verified that the Relevo theme automatically emits `BreadcrumbList` + `NewsArticle` JSON-LD on every post (with headline, description from excerpt, datePublished, dateModified, mainEntityOfPage, author, publisher, image, articleSection). Baseline structured data covered out of the box. Two refinements remain as nice-to-have: see `SEO-03` (FAQPage for articles with FAQ section) and `SEO-04` (`@type` precision for Rules/Tactics).
- ✅ **WP MCP setup** (May 19) — `ig-mcp-proxy` wired to `relevo-com.sub.plus`, identity + connection persisted in `~/.ig-mcp/config.json`.
- ✅ **Image upload helper** (May 19) — `scripts/wp_upload.py` validates against full-size 1024×1024 originals, no recompression needed. Eliminates the base64-inline ceiling.
- ✅ **Image generation E2E** (May 18) — `gpt-image-2` via LiteLLM iGaming, C2PA / JUMBF / OpenAI markers stripped, integrated into `scripts/imagen/`.
- ✅ **148-topic bulk seed** (May 18) — all 24 sports × 6 categories seeded into the Excel inventory, status `pending_review`.
- ✅ **Helpers consolidated** (May 18) — `sharepoint_msal.py` refactor (`update_table_row`, retry), `relevo_approve.py` LIVE, `get_reglamento.py` LIVE.
- ✅ **24/24 rulebooks in SharePoint** (May 18) — Cycling UCI, Paralympics IPC, FIM GP added; `_pdf_name` validated against Graph for all 24 sports.
- ✅ **Schema reconciliation** (May 18) — `cmd_content_plan.md` rewritten against the 22-column live Excel schema (option A executed).
- ✅ **GitHub Pages live** (May 15) — public roadmap at `jpwaimann.github.io/relevo-content/`.
- ✅ **Asana reviewer subtasks** (May 15) — `asana_notify.py` functional with round-robin pool.
