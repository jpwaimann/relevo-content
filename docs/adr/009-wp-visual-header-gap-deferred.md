# ADR-009: WordPress visual header gap — root cause + Phase 4 plan

**Status:** Decided 2026-05-22 EOD — fix deferred to Phase 4 (Tue 26/5 onwards).
**Context:** WP-VISUAL-GAP backlog item, escalated from P0 (closes Phase 3) to P2 (Phase 4 work).
**Authors:** JPW + Claude.

## Context

Since the first batch of articles was published as drafts on the beta theme (`relevo-com.sub.plus`), the `single.php` article header has shown a visible white space between the lower third of the hero image and the byline banner. The gap is purely visual — content renders correctly, schema is intact, the article body is unaffected.

Three CSS iterations on 2026-05-21 (align-items center; align-items stretch + object-fit cover; content-block.css v1.1.2) did not resolve the issue. A focused diagnostic + fix attempt on Friday 2026-05-22 produced enough data to identify the root cause definitively, but also revealed that a clean fix is not achievable inside Phase 3's remaining time without introducing worse trade-offs.

## Diagnosis (Friday 22/5 session)

Live measurement against draft post 21195 ("Posiciones en el fútbol") via DevTools `getComputedStyle` + `getBoundingClientRect`, executed in viewport 1280px (the closest case to the production desktop view):

| Element | Measurement (initial) | After fix attempt |
|---|---|---|
| `.layout-article-header` | grid 1fr 1fr, `align-items: start` | grid 1fr 1fr, `align-items: stretch` |
| Column 1 (text: badge + h1 + subtitle) | 200px natural height | 368.66px (stretched) |
| Column 2 (hero image wrapper) | 368.66px (3:2 aspect of 1024×683 image) | 368.66px |
| `.article-hero-image` (inside col 2) | 368.66px | 368.66px |
| Visible "gap" | Below text, inside col 1 | Same — only the layout mechanism that produces it changed |

### Root cause

The hero image is generated at **3:2 aspect ratio** (gpt-image-2 default for the current `lib/image.md` config: 1024×683 in the published version). In the desktop grid 1fr 1fr layout (column width ~553px at viewport 1280), a 3:2 image scales naturally to **~369px tall** to maintain its aspect ratio.

The text block (badge + 2-line H1 + 3-line subtitle for the test article) has a natural height of **~200px**.

Because the grid track is sized by `max(col_natural_heights)`, the row becomes 369px tall — decided by the image. The text either anchors to the top (with `align-items: start`, leaving 169px of white space below the subtitle) or stretches its column (with `align-items: stretch`, still leaving the contents visually anchored at the top because the H1/subtitle elements have no inherent vertical distribution rule).

**It is structurally impossible to eliminate the gap in CSS alone while preserving (a) the current 2-column layout, (b) the full uncropped image, and (c) the current 3:2 image aspect.** Any two of those three can coexist; all three together cannot.

### Verification attempts and outcomes

1. **`align-items: stretch` + `object-fit: cover` + image `height: 100%`** (CSS-only attempt v1). Outcome: text column stretches to 369px but contents stay anchored top → gap is now "inside" the text column instead of "between" the columns. Visually identical.
2. **`position: absolute; inset: 0` on `.article-hero-image`** (CSS-only attempt v2 — removes image from layout flow so row is sized by text). Outcome: gap eliminated. **But** the image is now constrained to a 553×200 box (aspect 2.76) while its natural aspect is 1.5, forcing `object-fit: cover` to scale and crop heavily — the player loses both head and feet. Visually unacceptable (JPW's call).

Both attempts were applied to production CSS during the session, then reverted to the original state once it was confirmed neither produced an acceptable visual outcome. The original gap is back in place — known, stable, ugly but not broken.

## Decision

**Defer the fix to Phase 4** (starting Tuesday 26/5, after Monday demo). The path forward is a three-step coordinated change, not a CSS tweak.

### Phase 4 plan (3 steps, ~45 min of execution)

1. **Define a fixed header height with vertical text alignment.** `.layout-article-header` gets a `min-height` per breakpoint (e.g., 350px at ≥1024px) and the text column gets `display: flex; flex-direction: column; justify-content: center;` so short titles centre nicely instead of bunching at the top. CSS-only, ~10 min.

2. **Change the image generator to produce 16:9 hero images.** Edit `scripts/imagen/` (or the prompt size argument in `lib/image.md`) from `1024×683` (3:2) to `1792×1024` (16:9). In a 553-px-wide column, 16:9 renders to ~311px tall — a near-perfect match for the 350px fixed header above (only 12% recrop). ~15 min including smoke test.

3. **Regenerate the batch 3 hero images.** 10 images via the updated image gen (~4 min generation + ~10 min validate-and-upload via WP MCP). All 9 existing batch 3 drafts (posts 21195–21219) get their featured image replaced. ~20 min.

Result: image-complete + zero gap + scales cleanly to production. No CSS overrides per article, no fragile selectors.

## Why we are not doing this on Friday EOD

1. **Phase 3 closes the Monday demo with the current draft set.** Joel must see batch 3 with stable, known visuals when he gives editorial feedback on Monday — otherwise his QA gets contaminated with a fresh visual regression.
2. **No QA window**. Friday evening → Monday morning is the weekend; if a regression appears in the regeneration step, there is no time to recover before the demo.
3. **Scope creep**. Items 1+2 above touch the image generator and require a smoke test before bulk regen. Doing this in a 5pm window is risky.
4. **Reversibility**. The gap is a stable known state. The regeneration changes 10 published drafts; rolling back to the previous heroes is doable but a real cost.

## Negative consequences considered (Phase 4 execution)

The decision to defer is itself not a trade-off — it is the only safe path Friday EOD. But the Phase 4 execution does carry real considerations, documented here so they are not surfaced as surprises on Tuesday:

- **Title-length sensitivity.** A fixed 350px header leaves ample air for short H1s (1-line title, no subtitle) and may be tight for 4-line H1 + long subtitle. The image gen and the editorial style guide must keep title length in a predictable range — already implicit in the style guide v1.1.3 but should be re-validated.
- **Aesthetic shift across the entire feed.** 16:9 is more "cinema/landscape", 3:2 is more "magazine/portrait". The whole site will read slightly more "broadcast TV" after the regen. Worth confirming with the brand-look intent.
- **Cost.** ~$0.40 to regenerate batch 3 (10 images × $0.04 via LiteLLM gpt-image-2). Negligible.
- **WP srcset side effects.** Uploading new images at a different aspect ratio will cause WordPress to generate new thumbnail sizes and may invalidate srcset entries cached by Joel's draft preview URLs. Joel needs to hard-refresh his preview links after the swap.
- **Social previews.** Twitter/Facebook prefer ~1.91:1 (close to 16:9 = 1.78:1) over 3:2 (= 1.5:1). The open-graph image will look **better** after the change, not worse.
- **The image gen pipeline must be smoke-tested with 1 article before bulk regen** to confirm `lib/image.md` and downstream stages accept the new size without breakage.

## Out of scope for Phase 4

- Refactoring `single.php` to a vertical hero-on-top layout (NYT/Atlantic style). Considered during the Friday session and rejected for now: bigger PHP refactor, more rendering risk, and the 3-step plan above achieves the same visual goal without touching templates.
- Tightening title/subtitle length constraints at the style guide level. The current style guide v1.1.3 already produces titles in the predictable range. If Phase 4 implementation reveals edge cases, that's a follow-up ADR.
- Hero image policy (gate-by-text Asana 2-step approval) is already covered by [ADR-007](007-hero-image-policy-gate-by-text.md) and remains independent of the aspect-ratio decision documented here.

## Backlog impact

| Backlog item | Before | After |
|---|---|---|
| `WP-VISUAL-GAP` | P0 (Phase 3 closing blocker) | **P2** (Phase 4) with renamed scope as `HEADER-REDESIGN` covering all 3 steps |
| `EDIT-14` (ADR-007 hero gate impl.) | P2 | unchanged — independent track |

The Friday session deliverables (the original-original CSS now back in place; the local + sidecar backups of every intermediate state still preserved under `backups/theme_css/2026-05-22T15-14Z/` and `*.bak-*` on the server) leave a clean rollback path if Tuesday's Phase 4 work needs to revert.

## Related

- [ADR-007](007-hero-image-policy-gate-by-text.md) — hero image gate-by-text approval flow.
- Backlog: `WP-VISUAL-GAP` (now `HEADER-REDESIGN`), Phase 4 priority.
- Session memory: `project_relevo_agenda_22may` updated to reflect the deferral and the Monday demo planning pivot.
