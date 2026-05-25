# SOP 01 — Content Production Pipeline

**Audience**: Juan Pablo Waimann (JPW) · Joel Sierra · Claude Code
**Status**: v1.0 — 2026-05-25
**Owner**: JPW
**Purpose**: Document the existing content production pipeline so every participant knows exactly what they do, at which step a human gate is required, and what handoffs look like. The pipeline itself does NOT change — this SOP only formalizes the roles and checkpoints so we can scale production with confidence.

---

## TL;DR — Who does what

| Step | Owner | Tool / artefact | Output |
|---|---|---|---|
| 0. Topic ideation | JPW + Joel | Inventory Excel (SharePoint) | Row with `status=pending_review` |
| 1. Topic approval | Joel | `relevo_approve.py` or Asana batch task | `status=pending` |
| 2. Article generation | Claude Code | `/relevo new` | WP draft + run dir + Asana review subtask |
| 3. Editorial review | Joel | Asana subtask + WP preview URL | ✅ / ⚠️ / ❌ verdict on the subtask |
| 4. Final publish decision | JPW | WP admin + Inventory Excel | Post moves to `pending` then `publish`; Inventory `status=published` |
| 5. Post-publish hygiene | — | Inventory + analytics | None — automatic |

**Gates that block downstream work:** 1, 3, and 4. Everything else is automated.

---

## 1. Roles

### Claude Code — the drafter
**What it does autonomously, no human needed:**
- Reads the Inventory Excel via Microsoft Graph (live source of truth).
- Picks the next row with `status=pending` (Joel-approved).
- Pulls the right rulebook PDF (Rules / Tactics) or runs WebSearch ideation (Origins / Coaches / newsworthy).
- Generates the article in multiple passes (research → draft → critique ≥ 80 → revise → fact-check).
- Runs the style guide self-check (currently v1.2, 25 binary tests).
- Generates the hero image via the iGaming LiteLLM proxy.
- Uploads the post to WordPress as a draft and attaches the hero.
- Creates the Asana review subtask under Phase 4 (current production parent task), assigned to the next reviewer.
- Updates the Inventory row to `status=review` with the WP post ID, the run directory, and the score.

**What it does NOT do:**
- It does **not** publish. Status moves to `publish` only after JPW's Stage 4 approval.
- It does **not** override a hard gate. If the self-check fails (e.g. bolds coverage < 0.80, a paragraph > 60 words, a Masters 1000 misattribution), the run halts and is reported as a blocker, not as a draft for review.
- It does **not** modify the Inventory schema or the style guide on its own. Schema changes need an ADR; style guide changes need JPW sign-off after a smoke test.

### Joel Sierra — the editorial reviewer
**Stage 1 — topic approval (asynchronous, in lots):**
- Sweeps Inventory rows with `status=pending_review`.
- For each row, decides ✅ (move to `pending`), ⚠️ (rewrite the angle, suggest edits in the `notes` column), or ❌ (drop).
- Typical batching: sport-by-sport, ~30 topics per session, via `relevo_approve.py approve --sport <X> --category <Y>` or via an Asana batch task.

**Stage 3 — editorial review on each draft:**
- Receives an Asana subtask per draft, with the WP preview URL and the score breakdown.
- Reviews the draft on the WP preview page. The review focuses on:
  - **Sport knowledge** — fact accuracy (palmarés, dates, ranks, geography, technical attributions). Joel's domain expertise is the moat.
  - **Editorial taste** — does the Relevo voice come through? Are bolds carrying scanning value? Are metaphors earned? Does the closing land?
  - **Internal coherence** — does the article actually answer the title's promise? Are the FAQs the questions a Relevo reader would actually ask?
- Posts a verdict on the Asana subtask:
  - **✅ Approve** — passes to Stage 4 (JPW).
  - **⚠️ Approve with notes** — flag specific issues. JPW decides per-article (inline fix vs full regen) or, if it's a cross-batch pattern, JPW patches the style guide and re-smokes.
  - **❌ Reject** — explain why. The article goes back to the Inventory as `status=pending` with a note for Claude to re-attempt.
- **What Joel does NOT do:** apply CSS / template fixes, modify the style guide directly, edit the WP draft body. Editorial input is text feedback on the subtask, not direct article edits.

### Juan Pablo Waimann (JPW) — the publisher / orchestrator
**Stage 4 — final publish decision:**
- Sweeps Asana subtasks that Joel marked ✅.
- Does a final pass on the WP draft: header rendering, hero image fit, internal linking, SEO title and meta description, schema markup.
- Decides:
  - **Publish** — moves post to `pending` (then human-published in WP). Updates Inventory to `status=published` with `wp_url` and `published_at`.
  - **Hold** — leaves as draft with an Asana note. Reasons: timing (calendar alignment), bundling with another piece, missing internal link target.
  - **Send back to Claude** — for regen with specific instructions.

**Cross-stage role:**
- Owns the Inventory schema and the style guide.
- Owns the Asana project structure (Phase 4 = current parent task for production reviews).
- Owns the public roadmap, the backlog, and the executive one-pager.

---

## 2. Pipeline stages — detailed

### Stage 0 · Ideation
- **Inputs:** sport calendar, FIFA WC window, search demand signals (urls_to_keep_gsc_enriched.xlsx, YouTube watch lists), editorial intuition.
- **Output:** one or more new Inventory rows with `status=pending_review`. Columns required: `id`, `sport`, `category` (Rules / Tactics / Origins / Coaches / Newsworthy / Betting), `title`, `slug`, `hub_slug`, `sub_hub`, `nombre`, `gancho`, `created_by`.
- **Trigger:** weekly editorial planning, on-demand for newsworthy content (FIFA WC kick-off, transfer window, etc.).

### Stage 1 · Topic approval
- **Owner:** Joel.
- **Cadence:** in lots — typically a weekly sweep of pending_review rows, sport by sport.
- **Tools:** `relevo_approve.py approve --sport <X> --category <Y>` (CLI on the AWS VM), or an Asana batch task with the row IDs to approve.
- **Output:** Inventory rows flipped to `status=pending`. Claude only picks from `pending`.

### Stage 2 · Article generation (Claude Code)
- **Entry:** JPW (or future scheduler) types `/relevo new` in Claude Code.
- **Claude reads:** Inventory live → first `pending` row → fetches rulebook PDF (Rules/Tactics) or runs WebSearch (Origins/Coaches/Newsworthy).
- **Multi-pass generation:**
  1. Research and source the facts (with WebSearch for live data).
  2. Draft the article under the relevant template (Rules v2 / Tactics v2 / Origins v2 / Coaches v1).
  3. Self-critique against the 25-point checklist of style guide v1.2.
  4. Revise until score ≥ 80 AND all hard gates pass (bolds coverage, no paragraph > 60 words, no Masters 1000 misattribution, etc.).
  5. Fact-check pass — every live data point cross-checked against WebSearch (the verifiable lists in style guide §2 are the floor).
  6. Hero image via gpt-image-2 through the iGaming LiteLLM proxy. AI metadata stripped (C2PA / JUMBF / OpenAI markers).
- **Upload:** `scripts/wp_upload.py` uploads the image; `posts.create` writes the draft with `post_excerpt=entradilla` and `post_content` starting at the first `<h2>` (per ADR-008 §2.4 amended).
- **Asana subtask:** auto-created under Phase 4 parent, assigned to the next reviewer in the round-robin (currently always Joel).
- **Inventory write:** `status=review`, `wp_post_id`, `wp_url`, `run_dir`, `score`, `critique_path`.

### Stage 3 · Editorial review (Joel)
- **Cadence:** Joel reviews subtasks daily during production weeks.
- **Inputs:** Asana subtask description (WP preview URL + score breakdown + factual fixes applied), and the WP draft preview itself.
- **Decision tree:**
  - ✅ → JPW (Stage 4).
  - ⚠️ with notes → JPW decides (inline fix, full regen, or systemic style guide patch).
  - ❌ → Inventory row back to `pending` with rejection notes; Claude can attempt again.
- **Patterns vs per-article**: if the same defect shows up in 3+ subtasks of the same batch, that's a systemic pattern and goes to a style guide patch (v1.2.x), not a per-article fix. The regenerate-batch methodology (memory: `feedback_regenerate_batch_methodology`) applies.

### Stage 4 · Final publish decision (JPW)
- **Inputs:** Joel ✅ verdict + WP draft.
- **Checklist:**
  - Header renders correctly (until the WP CSS fix lands — currently tracked as deferred work).
  - Hero image fits the angle.
  - Internal links resolve.
  - SEO title and meta description on-brand and within length.
  - Schema markup present (FAQPage + NewsArticle baseline).
- **Output:** post moved to `pending` then `publish` in WP. Inventory row to `status=published` with `wp_url` and `published_at`.

### Stage 5 · Post-publish
- No human action required by default.
- If a published article needs edits (live data update, ranking change), open an Asana task referencing the post ID. Claude can re-fetch live data and propose a patch.

---

## 3. Asana conventions

- **Parent task for current production reviews:** Phase 4 — Production launch & FIFA WC readiness (`1214844989980281`).
- **Subtask name format:** `[Editorial QA] <article slug or topic> (Joel review)` or `[Editorial QA] Smoke v1.x — <topic> (JPW pre-Joel review)` for smoke runs.
- **Language:** English only on Asana (project convention).
- **Daily entries in the JPW weekly task:** ≤ 5 lines per day, each line prefixed with the relevant Asana URL.
- **Assignee:** only assign when the reviewer is the next actor. Topic ideation rows in the Inventory are NOT Asana tasks — they live in the Excel.

---

## 4. Inventory Excel — single source of truth

**Constraint set by JPW:** one and only one Excel inventory holds **all** content the project produces — Rules, Tactics, Origins, Coaches (incoming), Newsworthy (incoming), Betting / Casino pages (incoming).

Adding new content types means **extending the schema** (new categorical values in the `category` column, plus any category-specific columns), **not** spinning up a parallel spreadsheet. Open follow-up: `INVENTORY-02` in the backlog tracks the schema extension for Coaches + Newsworthy + Betting.

---

## 5. Quick reference — what each person does in a given day

### Joel — production day
1. Open the weekly Asana task under his name → see today's pending subtasks.
2. For each subtask: open the WP preview, review against the 3 axes (sport knowledge, editorial taste, internal coherence), post verdict.
3. If a pattern emerges across subtasks, mention it on the parent task so JPW can decide on a style guide patch.

### JPW — production day
1. Sweep Joel's ✅ subtasks → run Stage 4 checks → publish / hold / send back.
2. Sweep Joel's ⚠️ subtasks → decide per-article action.
3. Sweep Joel's ❌ subtasks → mark Inventory row back to `pending` with the rejection note.
4. Update the JPW weekly Asana task with the day's bullets (≤ 5 lines, English, Asana URLs prefixed).
5. Refresh the executive one-pager (`docs/one-pager.html`) at EOD.

### Claude Code — invoked on demand
1. `/relevo new` → pick a `pending` row, run Stage 2 end-to-end, hand off the draft to Joel via Asana.
2. `/relevo content-plan` → ideation for a sport / category; writes new `pending_review` rows.
3. `/relevo status` → snapshot of the queue (`pending_review`, `pending`, `review`, `published`).
4. `/relevo demo` → simulated run for stakeholder demos, no production write.

---

## 6. Out-of-scope failure modes

- **Claude can't source a fact** → article does not generate. Run dir keeps the failure log. JPW or Joel can override by providing the source manually in a follow-up `/relevo new` call.
- **Style guide hard gate fails** → no delivery to Joel. The block is reported on the subtask, and the row remains in `status=pending` for a re-attempt or a manual edit.
- **Joel returns ⚠️ for a pattern that affects 3+ articles in a batch** → JPW patches the style guide to v1.2.x with the pattern as a binary check, runs a smoke on one representative article, signs off, and only then re-generates the batch.
- **WP outage / SharePoint outage** → Claude reports the integration error on the subtask. The draft remains in the run dir; no Inventory row is updated. Re-run when the integration is back.

---

## 7. Version log

- **v1.0 (2026-05-25)** — Initial SOP. Captures the pipeline as it stands after the Phase 3 closing demo: style guide v1.2 (ADR-010), templates Rules v2 / Tactics v2 / Origins v2, Coaches template incoming, single Inventory Excel constraint.
