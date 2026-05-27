# ADR-013: Content strategy v1 — 5 verticals + tier-1 marking + 170-draft bulk + hero gate dropped

**Status**: Approved
**Fecha**: 2026-05-27
**Drivers**: JPW (post team-call alignment)
**Relacionado**: ADR-007 (hero image gate-by-text — now superseded for bulk), ADR-011 (coaches template v1.0), ADR-012 (style guide v1.2.2)
**Asana anchor**: [Strategy] Relevo content strategy — 5 verticals (`1215171204862093`)

---

## 1. Context

A team-wide alignment call on 27/05/2026 closed out the strategy direction for Relevo.com content production under the iGaming.com mandate. The acquisition mandate (audience = Spanish sports bettors, monetization via casino/sportsbetting verticals back-end) reshapes the content portfolio from "content automation MVP" to "5-vertical strategic portfolio with one immediate production sprint".

Three decisions emerged that need to be recorded here, because each one supersedes or extends prior ADRs:

1. **Five-vertical content portfolio** (evergreen + calendar + WC2026 + reactive + casino) replaces the previous "evergreen as MVP" framing. Documented in `docs/strategy/content-strategy-v1.md`.
2. **Tier-1 figure marking** in the Inventory as cross-cutting QA mechanism for high-recognition Spanish sports figures.
3. **Hero image gate-by-text superseded** for bulk production runs (170+ drafts). ADR-007's per-piece Asana 2-step does not scale.

Companion: the same call confirmed an aggressive production target — **170 evergreen drafts to WP by end of week** (50 rules + 50 tactics + 50 origins + 20 coaches), under style guide v1.2.2 + coaches v1.1.

## 2. Decisions

### 2.1 Five verticals codified

Full descriptions in `docs/strategy/content-strategy-v1.md`. Summary:

| Vertical | Lifecycle | Reviewer | Compliance | Status |
|---|---|---|---|---|
| V1 Evergreen | Atemporal | Joel | Standard | **Active production this week** |
| V2 Calendar | 3–7 days lead | Joel | Standard | Calendar v2 in flight |
| V3 WC2026 | Until 19/07 | Joel | Standard | Topic list with Joel |
| V4 Reactive | 24–72h | TBD journalist | Standard | Roadmap only — no staffing |
| V5 Casino | Always-on | Commercial + Legal | High | Template design pending |

Strategic implication: the pipeline `/relevo new` switch must eventually route by `article_type` (not just `category`) to support V3/V5 with different style guides and compliance gates. Out of scope for this ADR — captured in roadmap.

### 2.2 Tier-1 figure marking

A row in the Inventory is **Tier-1** if its subject (jugador / entrenador / equipo / figura) satisfies either:

1. Appears in `urls_to_keep_gsc_enriched.xlsx` with tier ≤ 2 (proven Relevo historical GSC pull).
2. Is on the curated mainstream-recognition list of ~110 names (see `docs/strategy/content-strategy-v1.md` §Tier-1).

Implementation:

- **Inventory column**: notes (col 21) prefix `[TIER-1: <figure>]`. No schema change. Non-destructive — the prefix coexists with existing free-form notes.
- **Pipeline behavior**: the `/relevo new` skill detects the prefix on the row it picks. When set:
  - WebSearch reinforced on factual claims (palmarés, dates, ages, families, attributions, geography).
  - Asana review subtask gets a `TIER-1` label and a sentence in its description: "this article touches a Tier-1 Spanish-audience figure; verify all factual claims independently before sign-off."
  - JPW Stage 4 publish gate gets an additional factual audit step.

**Why not a dedicated column** (e.g. col 24 `is_tier_1` boolean): a schema change requires an ADR + helper update + WP MCP plugin update + docs. The notes prefix achieves the same routing in zero schema cost. If usage becomes heavy (e.g. analytical queries by tier), promote to a column in INVENTORY-03.

**27/05 baseline**: 61 rows marked Tier-1 across the 170 batch (21 origins + 18 rules + 12 tactics + 8 coaches + 2 newly seeded).

### 2.3 Hero image gate-by-text superseded for bulk

ADR-007 specified a per-piece Asana 2-step where the hero image text-only spec is approved before image generation, to avoid AI artifacts on real figures. That gate does not scale at 170 drafts in 5 days.

**New policy for bulk production**:

- Pipeline auto-selects the hero following the 4-tier policy from style guide v1.2.1 §Imagen destacada (Wikimedia → stock contextual → editorial licensed → ilustración estilizada NO fotorrealista).
- AI-generated photoreal portraits of real figures still PROHIBITED (Ley Orgánica 1/1982 + EU AI Act binding).
- Hero ships with the WP draft. Joel + JPW review the hero alongside the article body. Feedback in the Asana subtask.
- No standalone hero-only approval step.

**Why this is safe enough**:

- The hardest prohibition (AI photoreal of real figures) is binary and binds the auto-selector — not relaxed by this ADR.
- Joel and JPW would have to look at the hero anyway during their pass; doing it once instead of twice removes a step, not a check.
- Sample diversity: at 170 drafts, the editorial team's volume-based judgment about hero quality is more reliable than a per-piece pre-approval that becomes pro-forma at scale.

**Re-evaluation**: quarterly. If post-publish complaints land on hero quality, restore ADR-007 for the failing category.

### 2.4 Production sprint — 170 drafts this week

Target: 50 rules + 50 tactics + 50 origins + 20 coaches → WP drafts by end of week 27/05 → 02/06. HITL review afterwards.

Operational state (verified 27/05):

- Inventory: 170 rows ready after curation + seeding (50/50/50/20).
- Tier-1 prefix applied to 61 rows.
- 21 origin rows marked `[deferred-batch-27may]` in notes (not in this sprint, available for next batch).
- 2 rows seeded (id 150 tactics "Tipos de tiros libres en el fútbol", id 151 coaches "Mourinho en el Real Madrid").
- Pipeline live: style guide v1.2.2 + coaches v1.1 + image gen + WP MCP + Asana notify.

Method:

- Smoke run pre-bulk: 4 pieces (1 per category) under v1.2.2 + coaches v1.1 to verify pipeline E2E. If clean → bulk. If issues → fix before bulk.
- Bulk in batches of 5 (parallelized cautiously; Tier-1 rows run serial with reinforced WebSearch).
- Memory snapshot after every batch (anti-VPN-drop).
- No publish wave during the bulk — drafts only. Joel + JPW publish wave later in the week.

## 3. Consequences

### 3.1 Immediate

- 170 drafts go to WP this week. Joel + JPW have a tangible review queue for ronda 5 onwards. The aggressive volume is balanced against editorial perfection — JPW explicit on the call: "vamos a sacar esos 150 artículos de todas maneras y él aplicará los cambios que sean relevantes. Lo más importante es NO meter la pata con los hechos comprobables."
- The CSS header gap (`ADR-009`) does not block this sprint — JPW: "Lo de los CSS es corregible, no me preocuparía mucho por eso."
- Hero per-piece gate removed → throughput up by ~50% on the image stage.

### 3.2 Strategic

- Reactive vertical (V4) becomes the bottleneck for completing the 5-vertical portfolio. Hiring conversation outside the AI sprint scope.
- Casino vertical (V5) needs a separate template + style guide + compliance reviewer pipeline. Captured in roadmap; activates post-WC kickoff.
- Tier-1 list will grow as new figures emerge (transfers, retirements, breakthrough stars). Updates land via PRs to `docs/strategy/content-strategy-v1.md`.

### 3.3 Risks

- **Factual errors on Tier-1 pieces** (the highest-cost failure mode per JPW). Mitigation: the reinforced WebSearch + audit step on Tier-1 rows. If post-publish complaints land, escalate Tier-1 to a per-piece factual review subtask before publish (smaller queue than the current 61, manageable for Joel).
- **Joel review queue overflow**: 170 drafts is large. Mitigation: drafts land in batches over the week, Joel + JPW share the review load, deferred-batch rows held for next week.
- **Style guide regressions**: smoke pre-bulk verifies v1.2.2 + coaches v1.1 land cleanly. If smoke shows the same patterns Joel marked in Simeone smoke v1.2.1 (negritas decorativas, cierre tono-manifiesto), iterate before bulk. v1.2.2 patches the harder patterns (hitos with 3 capas, categorías históricas, pasajes técnicos) — bolds and closing depend on execution discipline of the subagent.

## 4. Plan of record

1. **Doc + commit + push**: `docs/strategy/content-strategy-v1.md` + this ADR + tier-1 marking + curation prefix applied to Inventory.
2. **SOP 1-pager**: short operator-facing handoff (JPW + Joel), already linked in the SOP HTML.
3. **Smoke pre-bulk**: 4 pieces (rules tier-1 + tactics + origins tier-1 + coaches). Validate v1.2.2 + coaches v1.1 + image gen + WP upload + Asana notify.
4. **Bulk production** in batches of 5. Memory snapshot after every batch.
5. **Post-bulk dispatch**: Joel review queue + JPW publish queue. Tier-1 audit step in JPW workflow.

## 5. Open follow-ups (not blocking this sprint)

- CSS header gap (ADR-009): assigned in Asana, off the critical path for this sprint.
- GitLab Ownership: JPW carry-over, GitHub remains source of truth.
- INVENTORY-03 (column promotion for tier-1 if usage warrants): post-sprint review.
- TEMPLATE-05 (news for V2 + V3): post-Joel-calendar-v2-signoff.
- TEMPLATE-06 (casino for V5): post-WC kickoff priority.

---

**Anchor doc**: `docs/strategy/content-strategy-v1.md`
**Asana strategy task**: `1215171204862093` (parent), 5 vertical subtasks + 1 calendar sub-sub-task linked.
**Style guide**: v1.2.2 (29-point self-check).
**Coaches template**: v1.1.
