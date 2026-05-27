# SOP quick reference — JPW + Joel

**1-pager for daily operations.** For the full SOP, see [01-content-production.html](01-content-production.html).

---

## What's running this week

170 evergreen drafts to WP (50 rules + 50 tactics + 50 origins + 20 coaches) under style guide v1.2.2 + coaches v1.1. Tier-1 figures flagged in the Inventory `notes` column with `[TIER-1: <figure>]` prefix.

---

## Pipeline at a glance

```
0. Topic ideation     JPW + Joel       → Inventory row, status=pending_review
1. Topic approval     Joel             → Inventory row, status=pending
2. Article generation Claude Code      → WP draft + Asana review subtask + status=review
3. Editorial review   Joel             → Asana verdict: ✅ / ⚠️ / ❌
4. Publish decision   JPW              → WP publish + Inventory status=published
```

**Gates that block downstream**: 1 (Joel), 3 (Joel), 4 (JPW). Everything else automated.

---

## Joel's daily checklist

1. Open the Asana subtasks under [Phase 4](https://app.asana.com/0/1213819517890615/1214844989980281) assigned to you. Each draft = one subtask.
2. For each subtask:
   - Open the WP preview link in the subtask description.
   - Review across 3 axes:
     - **Sport knowledge** (palmarés, dates, ranks, geography, technical attributions). This is your moat.
     - **Editorial taste** (Relevo voice, bolds carry scanning value, metaphors earned, closing lands).
     - **Internal coherence** (article answers the title's promise, FAQs are real questions).
   - Post verdict on the subtask:
     - **✅ Approve** → JPW (Stage 4).
     - **⚠️ Approve with notes** → flag specific issues. JPW decides per-article fix vs full regen.
     - **❌ Reject** → explain why. Article returns to Inventory `pending` for regen.
3. **TIER-1 subtasks** (prefix `TIER-1` in subtask title): extra scrutiny on facts. The Spanish audience knows these figures. A wrong date or palmarés destroys trust.
4. If a defect repeats across 3+ subtasks of the same batch → mention it on the parent Phase 4 task so JPW patches the style guide.

**What Joel does NOT do**: CSS / template fixes, edit the WP draft body, modify the style guide directly. All editorial input goes as text feedback on the subtask.

---

## JPW's daily checklist

1. **Sweep Joel's ✅ subtasks** → Stage 4 publish gate:
   - Header renders correctly (CSS gap pending, [tracked](https://app.asana.com/0/1213819517890615/1215166573030805) — don't block on it).
   - Hero image fits the angle.
   - Internal links resolve.
   - SEO title + meta description on-brand and within length.
   - Schema markup present.
   - **TIER-1 check**: independent factual audit on palmarés, dates, geography.
   - → Publish in WP. Update Inventory `status=published` with `wp_url` and `published_at`.
2. **Sweep Joel's ⚠️ subtasks**: per-article decision.
   - Inline fix in WP if minor (typo, missing internal link).
   - Full regen via `/relevo new <row_id>` if structural.
   - Patch style guide v1.2.x if it's a cross-article pattern (regenerate-batch methodology).
3. **Sweep Joel's ❌ subtasks**: Inventory row back to `pending` with the rejection note. Claude can re-attempt with the note as context.
4. **Update the JPW weekly Asana task** with the day's bullets (≤ 5 lines, English, Asana URLs prefixed).
5. **Refresh the executive one-pager** (`docs/one-pager.html`) at EOD if material changes.

---

## Claude Code — invoked on demand

```
/relevo new                  → pick next pending row, run full E2E
/relevo new <row_id>         → re-generate a specific row (regen scenario)
/relevo content-plan         → ideation for a sport / category
/relevo status               → snapshot of the queue
```

Claude does NOT publish. Claude does NOT override hard gates. Claude does NOT modify the style guide or schema autonomously.

---

## Where things live

- **Inventory Excel**: SharePoint, single source of truth. Editable via `scripts/sharepoint_msal.py` (MSAL device-code). Read live in the pipeline.
- **Style guide**: `prompts/style_guide.md` (current: v1.2.2). The pipeline injects this into every system prompt.
- **Templates**: `prompts/{rules,tactics,origins,coaches}_template.md`.
- **SOP**: `docs/sop/01-content-production.md` (canonical) / `01-content-production.html` (rendered).
- **Strategy doc**: `docs/strategy/content-strategy-v1.md` (5 verticals + tier-1 list + roadmap).
- **ADRs**: `docs/adr/`. Latest: ADR-013 (content strategy v1).
- **Asana parent for production reviews**: [Phase 4 — Production launch & FIFA WC readiness](https://app.asana.com/0/1213819517890615/1214844989980281).
- **Asana strategy task**: [Relevo content strategy — 5 verticals](https://app.asana.com/0/1213819517890615/1215171204862093).

---

## Tier-1 figures — current baseline (27/05/2026)

Football: Yamal, Nico Williams, Pedri, Gavi, Vinícius, Bellingham, Mbappé, Cristiano, Messi, Simeone, Guardiola, Ancelotti, Xavi, Xabi Alonso, Flick, Julián Álvarez, Griezmann, Haaland, Benzema, Sergio Ramos, Casillas, Iniesta, Florentino, Laporta, Brahim Díaz, Rodri, Dani Olmo, Mourinho.
Tennis: Alcaraz, Nadal, Djokovic, Sinner, Federer, Swiatek, Sabalenka.
MMA/Boxing: Topuria, McGregor, Khabib, Jon Jones, Adesanya.
F1/Moto: Fernando Alonso, Carlos Sainz, Verstappen, Hamilton, Leclerc, Norris, Russell, Piastri, Marc Márquez, Jorge Martín, Bagnaia, Palou, Rahm.
Basketball: Pau Gasol, Marc Gasol, Ricky Rubio, Llull, Doncic, Aldama.
Cycling: Pogačar, Vingegaard.
Pádel: Lebrón, Navarro, Coello, Galán, Triay.
Atletismo: Peleteiro, María Pérez, Cerezo, Tió.

(Extracted from `urls_to_keep_gsc_enriched.xlsx` Tier 1-2 + curated mainstream. Update via PR.)

---

## Out-of-scope failure modes

- Claude can't source a fact → article doesn't generate, failure log in run dir, row stays `pending`.
- Style guide hard gate fails → no delivery to Joel, blocker reported on subtask.
- Pattern affects 3+ articles in a batch → JPW patches style guide v1.2.x → smoke → bulk regen.
- WP / SharePoint outage → integration error reported, no Inventory update, re-run when back.

---

**Owner**: JPW · **Status**: v1.0 (2026-05-27) · **Source**: [github.com/jpwaimann/relevo-content](https://github.com/jpwaimann/relevo-content/blob/main/docs/sop/quick-reference-jpw-joel.md)
