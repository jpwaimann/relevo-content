# Relevo content strategy v1

**Status**: Living document · v1 — 2026-05-27
**Owner**: JPW
**Audience**: JPW, Joel, future Claude Code sessions, iGaming stakeholders
**Asana anchor**: [Strategy] Relevo content strategy — 5 verticals (LIVING DOC) — `1215171204862093`

---

## Why Relevo exists today

Relevo.com was a Spanish sports journalism outlet between 2022 and 2025. In 2025 iGaming.com acquired the property to convert it into an advertising channel for sports bettors and casino players in Spain, while preserving its sports-media identity as the audience hook.

**The commercial thesis**: a sports-media front with high-trust editorial voice + a strong commercial back-end (betting / casino sections) attracts a higher-LTV reader than a pure affiliate site. The editorial verticals (evergreen + calendar + WC2026 + reactive) build the audience and the trust; the casino/sportsbetting verticals monetize it.

## Audience target

**Primary**: Spanish sports bettors. Demographics: peninsular Spain, 25–55, mostly male, smartphone-first, multi-sport consumers (La Liga + Champions + tennis + F1 + MMA + NBA + pádel).

**Secondary**: general Spanish sports fans pulled in by editorial content; converted to the betting/casino verticals via the on-site navigation.

## Strategy to reach this audience

A five-vertical content portfolio. Four are permanent, one is temporal (World Cup 2026). The setup balances:

- **SEO long-tail durability** → evergreen.
- **Event-driven traffic surges** → proactive calendar + WC2026.
- **Editorial authority** → reactive/contingency (when staffed).
- **Direct commercial conversion** → casino/sportsbetting.

Editorial standards (style guide v1.2.2, ADR-012) and the production SOP apply across verticals where Claude Code is the drafter. Verticals diverge in lifecycle, reviewer routing, compliance requirements, and source-of-data.

---

## The five verticals

### Vertical 1 — Evergreen (rules, tactics, origins, coaches)

**Foundation of the portfolio.** Four categories. Automated 95% by Claude Code (`/relevo new` skill), HITL gates from Joel (editorial review) and JPW (publish decision). Full SOP at `docs/sop/01-content-production.md`.

- **Rules**: how a sport works, category-by-category explainers. Examples: "Reglas del fútbol IFAB 2025/26", "Categorías de peso UFC", "Tipos de pases en baloncesto".
- **Tactics**: tactical concepts, formations, technical breakdowns. Examples: "Qué es el falso 9", "La presión alta en fútbol moderno".
- **Origins**: biographical pieces, focused on identity/family/origin. Examples: "Los orígenes de Lamine Yamal", "Carlos Alcaraz: la formación de un campeón".
- **Coaches**: era + palmarés explainers. Examples: "Los títulos de Simeone en el Atlético".

**Why this works for Relevo**: confirmed by Joel's GSC review of historic Relevo (`urls_to_keep_gsc_enriched.xlsx`):

- Origins of star figures dominate Tier-1 historic (Topuria 100k impressions long-range, Sinner, Bellingham, Mbappé, De Miñaur, Brahim, Leclerc, Peleteiro).
- Specific rules pieces also rank Tier 1 (voleibol completo 74k clicks; categorías peso UFC; posiciones baloncesto; tipos pases; golpes bádminton).
- Tactics has 0 URLs in Tier 1 GSC history — included by editorial decision, lower expected traffic impact.
- Coaches has documented Tier-1 examples in the "cuántos / palmarés" format.

### Vertical 2 — Proactive sports-calendar

**Anticipatory pieces tied to upcoming sport events.** NOT post-event news. Lead time: 3–7 days before the event (per Joel ronda 4 feedback, 27/05/2026 — better than 7–14 because the reader is mentally engaged with the upcoming event).

Examples:
- "Las 5 mejores finales de Wimbledon" (pre-Wimbledon).
- "Las selecciones que juegan por primera vez un Mundial" (pre-WC).
- "Los 10 mejores ciclistas históricos del Tour de France" (pre-Tour).
- "Cuántas veces ha ganado España el Eurobasket" (pre-Eurobasket).

**Anchor doc**: `docs/sports_calendar_2026_h2.md` (Joel-reviewed v1 → v2 incorporating round-1 feedback).

**Template**: TEMPLATE-05 (news) NOT yet designed. Blocked on calendar v2 sign-off.

### Vertical 3 — WC2026 (temporal)

**Exists only for the FIFA World Cup 2026** (kickoff 11/06/2026, final 19/07/2026). After the tournament, retired.

Decision (Joel, 27/05): focus 100% on Mundial 2026; no histórico de otros Mundiales. Joel is curating a list of ~100 World Cup-related topics in his own Asana task (`1215152974998057`).

Production model after topic approval: bulk-seed into Inventory with `article_type=news` + `vertical=wc2026` tag → Claude Code drafts → Joel reviews → publish under tighter timing (WC kickoff is 11/06 hard).

### Vertical 4 — Reactive / contingency

**Editorial reactions to events 24–72h after they happen, with the Relevo angle.** Currently on the strategic roadmap, NOT in active development — needs a journalist or KOL not currently on the team.

Possible interim if a tier-1 event happens (España wins WC final, major coach exit, etc.): JPW + Joel co-write one piece manually. Not scalable, kept as override option.

### Vertical 5 — Casinos / sportsbetting (commercial)

**Operator rankings + reviews. The direct commercial conversion vertical.** Reference / inspiration: [es.motorsport.com/apuestas](https://es.motorsport.com/apuestas/) — existing iGaming.com property using the same model.

Content types:
- Master ranking ("Mejores casas de apuestas en España 2026").
- Payment-method filters ("Mejores casas con Bizum").
- Feature-specific ("Casas con cash out instantáneo", "Depósito 5€").
- Per-operator reviews.

Format observed in motorsport apuestas: long-form (5,000–8,000 words), ranking table with `operador / bono / depósito mínimo / tiempo retiro / payout % / mini-review / CTA "Visitar"`. Tone profesional pero conversacional. Affiliate links transparent. Mandatory: "Publicidad | 18+ | Aplican términos y condiciones".

**Operational approach** (per JPW, 27/05): recycle existing iGaming.com content as seed → automate growth with Claude Code afterwards.

**Compliance**: NOT covered by Joel — needs legal/compliance review (DGOJ, Real Decreto 958/2020, Ley 13/2011). Disclaimers + 18+ + responsible-gambling block per page. Geo-gating per Spanish territories.

**Reviewer routing for V5**: TBD — Joel is editorial-only. Commercial + compliance approver pending.

**Template**: TEMPLATE-06 (casino) NOT yet designed. Style guide v1.2.2 does NOT apply directly — V5 needs its own due to compliance + commercial framing.

---

## Tier-1 figure handling (cross-vertical)

A subject is **Tier-1** if it satisfies either:

1. Appears in `urls_to_keep_gsc_enriched.xlsx` with tier ≤ 2 (proven historical GSC pull on Relevo).
2. Is mainstream-recognized by the Spanish sports audience even without GSC history (curated list of ~110 names below).

Tier-1 implications:

- Reinforced WebSearch on every factual claim (palmarés, dates, ages, families, ATP/M1000 attributions, geographic facts).
- Flag in the Asana review subtask: `TIER-1 — extra scrutiny on facts`.
- Mandatory factual audit pass before publish, even with Joel ✅.
- Inventory `notes` column prefix: `[TIER-1: <figure>]`.

**Rationale**: the Spanish sports audience recognizes these figures very well. A factual error on a Tier-1 piece destroys trust at the article level and at the brand level. The remaining ~64% of pieces (non-Tier-1) also follow the standard self-check, but the audit budget concentrates on Tier-1.

### Curated Tier-1 list (27/05/2026 baseline, growing)

Football: Lamine Yamal, Nico Williams, Pedri, Gavi, Vinícius, Bellingham, Mbappé, Cristiano Ronaldo, Messi, Simeone, Guardiola, Ancelotti, Xavi, Xabi Alonso, Flick, Julián Álvarez, Griezmann, Haaland, Benzema, Sergio Ramos, Casillas, Iniesta, Florentino Pérez, Laporta, Brahim Díaz, Rodri, Dani Olmo, Mourinho.
Tennis: Alcaraz, Nadal, Djokovic, Sinner, Federer, Swiatek, Sabalenka.
MMA/Boxing: Topuria, McGregor, Khabib, Jon Jones, Adesanya.
F1/Moto: Fernando Alonso, Carlos Sainz, Verstappen, Hamilton, Leclerc, Norris, Russell, Piastri, Marc Márquez, Jorge Martín, Bagnaia, Alex Palou, Jon Rahm.
Basketball: Pau Gasol, Marc Gasol, Ricky Rubio, Sergio Llull, Doncic, Santi Aldama.
Cycling: Pogačar, Vingegaard.
Other: Ana Peleteiro, María Pérez, Adriana Cerezo, Iris Tió, Miguel Molina, Juan Lebrón, Paquito Navarro, Arturo Coello, Alejandro Galán, Gemma Triay.

(Extracted via regex from `urls_to_keep_gsc_enriched.xlsx` Tier 1-2 titles + curated mainstream list. Update as needed via PRs to this doc.)

---

## Hero image policy (revised 27/05/2026)

**Decision (JPW)**: production-time human gate on hero images does not scale with batches of 170. Drop the gate-by-text Asana 2-step (ADR-007) for the bulk pipeline.

**New policy**:

- Pipeline auto-selects the best available hero (4-tier policy from style guide v1.2.1 §Imagen destacada still applies: Wikimedia Commons → stock contextual genérico → editorial licensed → ilustración estilizada NO fotorrealista).
- Hero ships with the WP draft, no pre-publish human gate.
- Joel and JPW review the hero alongside the article body during their normal review pass. Feedback on heroes lives in the Asana subtask comments.
- AI-generated photoreal portraits of real figures still NOT allowed (Ley Orgánica 1/1982 + EU AI Act risk). Anti-AI rules from style guide v1.2.1 §Imagen-destacada still binding.

**ADR-007 status**: superseded for bulk production. Re-evaluated quarterly.

---

## Pipeline state (27/05/2026)

- Drafter: Claude Code via `/relevo new`. Live.
- Style guide: v1.2.2 (29-point self-check, binary tests). Committed: `b94ebc9` for HTML SOP; `f9db877` for style guide v1.2.2 + coaches v1.1 + ADR-012.
- Templates: rules v2 / tactics v2 / origins v2 / coaches v1.1.
- Inventory: 195 rows in Excel Table1, accessible via `scripts/sharepoint_msal.py` (MSAL device-code).
- HITL: Joel (Stage 3) + JPW (Stage 4) via Asana subtasks.
- Image gen: gpt-image-2 via LiteLLM iGaming, integrated into the skill.
- WP MCP: live. Posts/media/terms.
- Asana notify: auto-creates review subtask per draft (`scripts/asana_notify.py`).

## Current week target (27/05 → 02/06/2026)

170 evergreen drafts published as WP drafts, ready for HITL review:

| Category | Target | Ready in inventory | Note |
|---|---|---|---|
| Rules | 50 | 50 | Sharp cap to top-50 by tier-1 + sport diversity |
| Tactics | 50 | 50 (49 pre-seed + 1 seeded 27/05) | Seed: "Tipos de tiros libres en el fútbol" |
| Origins | 50 | 50 (curated from 71 ready; 21 deferred to next batch) | Curation by tier-1 score + sport diversity |
| Coaches | 20 | 20 (19 pre-seed + 1 seeded 27/05) | Seed: "Mourinho en el Real Madrid" |
| **Total** | **170** | **170** | ✅ |

Tier-1 marked in inventory: 61 rows (≈ 36% of the batch).

---

## Roadmap (post-current-week)

| Vertical | Next milestone | Dependency |
|---|---|---|
| V1 Evergreen | 170 drafts → Joel review → publish wave 1 | This week |
| V2 Calendar | Calendar v2 → TEMPLATE-05 design → news seeding | Joel calendar v2 sign-off |
| V3 WC2026 | 100 topics → bulk seed → drafts | Joel WC topics list |
| V4 Reactive | Roadmap only | Needs journalist/KOL hire |
| V5 Casino | Recycle iGaming source content → TEMPLATE-06 design → compliance routing | Post-WC kickoff priority |

---

## Open strategic questions

1. Casino reviewer: who approves V5 commercially + legally?
2. WC2026 lifecycle: dedicated TEMPLATE-X or reuse TEMPLATE-05 (news)?
3. Cross-vertical hub structure: how are the 5 verticals navigationally exposed on relevo.com home?
4. Reactive vertical: hire roadmap or strategic pause?
