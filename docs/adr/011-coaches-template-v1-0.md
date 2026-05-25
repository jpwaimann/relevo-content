# ADR-011: Coaches template v1.0 — 4th content category

**Status**: Approved
**Fecha**: 2026-05-25
**Drivers**: Juan Pablo Waimann (JPW) — instruction to adapt the SharePoint base template into a Relevo-style prompt mirroring `origins_template.md` v2.0
**Source artefact**: SharePoint `seoberlin/ES MARKET/Relevo.com/Content/Relevo - Coach Era & Trophies [Base Template].docx` (35 KB), authored by the Relevo editorial team
**Output artefact**: `prompts/coaches_template.md` v1.0 (355 lines)
**Relacionado**: ADR-001 (architecture), ADR-005 (templates v2 rework — Rules / Tactics / Origins), ADR-010 (style guide v1.2), [SOP v1.0](../sop/01-content-production.md), `INVENTORY-02` (open prerequisite — schema extension)

---

## 1. Context

The project ships three content categories today: Rules, Tactics, Origins (templates v2 under ADR-005). All three were defined for an athlete-centric or rulebook-centric narrative.

Coach-trophy enumeration pages — `titulos-ganados-{equipo}-{entrenador}` — sit outside that taxonomy. The narrative is:

- **Enumerative**: the trophy list is the spine.
- **Era-centric**: the identity of the coaching era (cholismo, tiki-taka, mourinhismo, heavy-metal football) is what carries the piece editorially.
- **Comparative**: the framing always anchors on a before/after data point (Champions participations, league finishes, share of historical trophies).

The SharePoint base template captures the structure well but is authored in mixed English/Spanish, lives in a Word doc not version-controlled, and is not wired into the `/relevo new` pipeline. The 4th category needs a Relevo prompt mirroring the Origins v2.0 pattern so the pipeline can pick it up like any other category.

## 2. Decision

Add **Coaches** as the 4th content category. Ship `prompts/coaches_template.md` v1.0 derived from the SharePoint base template, restructured to mirror `origins_template.md` v2.0. Key parameters:

| Axis | Value |
|---|---|
| **Word count** | 800–1.200 (much tighter than Origins 2.500–3.000) |
| **Schema** | Article + Person (coach) + SportsTeam (club / national team) + FAQPage |
| **URL pattern** | `/{hub_slug}/{sub_hub}/titulos-ganados-{equipo_slug}-{entrenador_slug}` |
| **SEO title formula** | `Títulos de {entrenador} con {equipo}: ¿cuántos lleva?` (or interrogative variant with apodo) |
| **Meta description formula** | `¿Qué títulos ganó {entrenador} con {equipo}? Descubre el palmarés de la era {identidad_era}: {total_titulos} campeonatos, finales históricas y datos clave.` |
| **Structure** | H1 → 3 intro paragraphs (no H2/H3) → H3 chronological trophy walk → H3 palmarés bullet list → optional H3 player-coach section + palmarés → H2 FAQs (5 standard) → narrative closing without "Cierre" H2 |
| **Applies to** | Fútbol (club + selección), Baloncesto (NBA/ACB), F1 (team principal), Rugby — **NOT** individual sports (tenis, atletismo, MMA, golf) |
| **Variables** | 18 (entrenador / equipo / identidad_era / era_inicio / era_fin / total_titulos / total_club_historico / predecesor / evento_destitucion / rival_clasico / rol_doble / + slugs) |

The template inherits style guide v1.2 (25-point self-check, valor scan test for bolds, valor cero test for grandiloquence, list internal consistency, table HTML structure, verifiable lists for facts, actionable closing). Coaches-specific rules layered on top:

- **Antes/después framing obligatorio** in paragraph 1 with a hard data point.
- **Era naming obligatorio** in paragraph 3.
- **Chronological trophy walk** (not grouped by competition).
- **Lost finals that define the era treated as load-bearing**, same depth as won finals (the two Champions finals lost by Simeone in 2014 and 2016 are central to the cholismo story).
- **Live-data verification** via WebSearch on every active-coach piece (current title count, last season disputed, rival + score of each final).
- **Club institution quote when applicable** ("Ganar y ganar y volver a ganar" for Atleti, "Més que un club" for Barça).

## 3. Editorial differentiation vs other templates

| Axis | Coaches | Rules | Tactics | Origins |
|---|---|---|---|---|
| Voice | Enumerative narrative with era anchor | Normative explanation | Analysis with visual anchor | Profile journalism with scenes |
| Central question | What trophies + what was the era called? | What does the rule say + consequences? | When is it used + who represents it + what changed? | Where does this person come from + what defines them? |
| Structure | 3-para intro + chronological walk + palmarés bullet | Thematic H2s | H3s item-by-item inside H2s | Narrative H2s by anchor |
| Tone | Era chronicle — before/after, identity, milestones | Explanatory with real cases | Analytical with canonical examples | Profile — scenes, conflict, quotes |
| Word count | 800–1.200 | 1.500–2.000 | 1.500–2.500 | 2.500–3.000 |

Coaches is the **shortest** format the project ships. The instinct to inflate toward Origins length is incorrect — the reader is searching for the enumeration; the narrative frames it, does not dilate it.

## 4. Sport-specific adaptations

| Sport | Specifics in palmarés bullet | Narrative care |
|---|---|---|
| Fútbol (club) | Liga, Copa, Continental (Champions/Europa), Supercopas | Double-role section if coach was also a player at the same club |
| Fútbol (selección) | Mundiales, Eurocopa/Copa América, Nations League | No double-role section. Internacionalidades framing if played for the country |
| Baloncesto (NBA) | Rings, Conference titles, MVP entrenador | Finals counter (W-L); the "casi" weighs in NBA narratives |
| Baloncesto (ACB) | ACB Liga, Copa del Rey, Euroliga | Final Fours counter |
| F1 | Constructors, Drivers (per-driver attribution), GP wins per season | Replace "entrenador" with "team principal" |
| Rugby | Six Nations, Champions Cup, domestic league | Low volume but high-search inside the rugby hub |
| Ciclismo | Grand Tours under technical direction, world championships | Only when the figure has multi-year impact (Brailsford at Sky/Ineos) |

## 5. Integration with the existing pipeline

### Skill changes required (separate task — depends on `INVENTORY-02`)

1. **`prompts/coaches_template.md`** — shipped today (v1.0). Lives next to the other three templates.
2. **`lib/cmd_new.md`** — extend the category switch in the generation flow to handle `Coaches`:
   - Source: WebSearch for current title count + final results + club archives. No rulebook PDF for this category.
   - Fact-check pass against verifiable lists (style guide v1.2 §2 applies; Coaches adds club-trophy archives as canonical sources).
   - WP upload uses the same pattern (`post_excerpt = entradilla`, `post_content` starts at first `<h2>`).
3. **Inventory schema** (`INVENTORY-02`) — add `Coaches` as a valid `category` value. Open question: do `equipo` / `entrenador` / `identidad_era` get their own columns or reuse `nombre` / `sub_hub` / `gancho`? Decision pending under `INVENTORY-02`.
4. **`scripts/imagen/`** — hero image prompt template needs a variant for coach portraits (less action-oriented than the Origins athlete prompts, more institutional — coach on the touchline, in the dugout, lifting the trophy). Defer until `INVENTORY-02` lands and the first Coach pieces are seeded.

### Asana

The category-specific Asana subtask pattern is the same as the other three: one subtask per draft assigned to Joel for Stage 3 review. No changes to the SOP.

## 6. Alternatives considered

- **Skip and let Coach pages live as Origins variants.** Rejected: the structural difference (chronological enumeration + era framing) is too large; the Origins template's H2-by-narrative-anchor pattern fights against the enumerative spine. Trying to force Coach pages through Origins would produce 2.500-word articles when the editorial intent is 800–1.200.
- **Keep the SharePoint Word doc as the source of truth and adapt only in JPW's head.** Rejected: not version-controlled, not picked up by the skill, no way for Claude to follow the structure without it being in `prompts/`.
- **Author the template from scratch instead of mirroring `origins_template.md`.** Rejected: the cross-template consistency in `origins_template.md` (changelog → qué cubre → foco editorial → URL → SEO → estructura → variables → componentes → reglas duras → adaptaciones → sample) is what makes templates legible to both the model and to Joel. Diverging structurally for one category would compound the cognitive load.

## 7. Consequences

### Positive

- 4th content category ready in the prompt layer. Coaches pages can be drafted under the same `/relevo new` invocation pattern.
- Cross-template consistency preserved (changelog, structure, sample resumido).
- Style guide v1.2 binary gates apply automatically — Coaches inherits 25/25 self-check, no new rules invented per category.
- Era-naming convention captured as a hard requirement, not editorial taste.

### Negative / open work

- `lib/cmd_new.md` does not branch on `Coaches` yet. First Coach draft cannot run autonomously until the integration lands.
- `INVENTORY-02` (schema extension) blocks the first Coach row from being seeded.
- Hero image prompt variant for coaches still TBD.

### Reversibility

Easy. The template is a single markdown file. If the editorial team wants to revise the structure (different sample, different word count band, different FAQ defaults), bump to v1.1 with the patch documented in the changelog header. No downstream code depends on the exact internal structure of the file.

## 8. Open items

- `INVENTORY-02`: schema extension to add `Coaches` as a valid category, plus columns for the Coaches-specific variables (or a decision to reuse existing columns).
- First seed: 2-3 high-signal coach pieces to validate the template end-to-end. Candidates raised by the SharePoint base: **Simeone / Atlético**, **Ancelotti / Real Madrid**, **Guardiola / Barça** (or Guardiola / Manchester City). The Simeone piece is also the canonical reference page in the SharePoint doc and is the natural smoke.
- Hero image prompt for coach portraits.

## 9. Version log

- **v1.0 (2026-05-25)** — Initial template + this ADR. Source: SharePoint Coach Era & Trophies base template + adaptation pattern of `origins_template.md` v2.0.
