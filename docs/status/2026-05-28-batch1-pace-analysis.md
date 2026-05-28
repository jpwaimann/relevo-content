# Batch 1 pace analysis — 2026-05-28 EOD

**Status**: 27 / 170 articles delivered (16%). Target Monday 2026-06-01: 170. **Gap: 143 articles in 4 days.**

## Where to find the delivered articles

- **WordPress admin** — https://relevo-com.sub.plus/wp-admin/edit.php?post_status=pending&author=77 (all 27 as `pending`, author = JPW)
- **Asana review inbox** — https://app.asana.com/0/1214109786765359/1214527443265898 (one subtask per article, assigned to Joel)
- **Inventory Excel** — `Relevo - Content - Article production DB.xlsx`, filter `status = review` (authoritative ledger)

## Iterations and outcomes

| # | Iter | What | Outcome |
|---|---|---|---|
| 1 | 27/5 PM | Full-Opus pipeline, 6 sub-agents per wave | **15 delivered, then session-limit incident at 2.5M Opus tokens. Wave 4 killed mid-flight.** |
| 2 | 28/5 AM | Opus orchestrator + nested Sonnet sub-agents (research + revise) | **Failed architecturally**. Sub-agents in this harness do not have the Agent tool — nested delegation impossible. All 3 sub-agents ran full-Opus. |
| 3 | 28/5 PM | Proposed: route through LiteLLM Opus 4.6 (Nexern's account) to bypass Claude Team window | **Rejected by JPW** — same total tokens, just shifts billing. Not real reduction. |
| 4 | 28/5 PM | Multi-model: Gemini 2.5 Pro (research) + Opus (draft/critique/revise) + GPT-5 (fact-check) + INFRA-08 autonomous finalize | **WORKING**. 8 articles delivered, avg score 92, all hard quality gates pass. 15 factual errors caught by GPT-5 across the 8 (would have shipped to Joel otherwise). |

Per-article Opus burn dropped from 115k (iter 1) to 70-80k (iter 4) — 35% reduction in the constrained bucket.

## Token consumption today (28/5)

| Bucket | Tokens | Cost basis |
|---|---|---|
| Opus 4.7 (Claude Code Team seat) | ~2.0–2.2M | per-seat allowance, 5h rolling window |
| Gemini 2.5 Pro (Nexern LiteLLM) | ~90k | Nexern's account, ~10× cheaper / token than Opus |
| GPT-5 (Nexern LiteLLM, prompt caching active) | ~85k | Nexern's account, ~3× cheaper / token than Opus |

Stopped at 27 / 50 batch-1 today to avoid a second session-limit incident. Window soft cap empirically ~2.5M Opus / 5h.

## Why we are behind pace

Two days = ~10 hours actual production time. The remainder went to:

- **27/5**: session-limit incident, ~5 articles' work lost mid-flight.
- **28/5 AM**: nested-agent iteration failed → no token savings, no new architecture validated until afternoon.
- **28/5 mid**: built helper scripts (`wp_post.py`, `finalize_article.py`, `llm_call.py`) to enable the working architecture — necessary one-time investment.
- **28/5 PM**: format bug discovered by JPW review of the smoke (duplicate H1 + bold coverage shortfall). Retroactive fix on 18 prior articles required.

Single Claude Code Team seat caps at ~25 articles / 5h window under current architecture. **Two windows / workday = ~50 articles / day theoretical max from one seat.**

## Math for Monday

| Scenario | Articles / day from Friday | Total by Monday EOD | vs 170 target |
|---|---|---|---|
| Optimistic (no friction) | 40 | 27 + 160 = **187** | +10% |
| Realistic (normal friction: Joel review cycle, weekend reduced bandwidth, edge cases) | 30 | 27 + 120 = **147** | −14% |
| Pessimistic (more architectural surprises) | 20 | 27 + 80 = **107** | −37% |

## Options for the team to weigh

| # | Lever | Throughput gain | Cost / risk |
|---|---|---|---|
| A | **Status quo** (current architecture) | ~30 / day | None |
| B | **Second parallel stream** via LiteLLM Opus 4.6 (Nexern account); both streams produce simultaneously | ~2× → 50–60 / day | No token saving; uses Nexern's account budget; needs JPW or Debanjan to authorise |
| C | **Shorten article target** from 1500–2200 to 1000–1500 words | +25–30% / day | Editorial decision; Joel input required |
| D | **Sonnet for non-tier-1 drafts** (request virtual key from Debanjan with Claude Sonnet access) | +40–60% / day | Voice quality risk on Spanish journalism; 1 smoke test needed first |
| E | **Cap revise at 0 loops**, ship below-85 scores to Joel | +15% / day | More Joel rework; quality slips |
| F | **Move deadline** from Monday to Tuesday/Wednesday | Releases pressure | External commitment |

Combinable. **A+B+C+D** stacked could theoretically reach ~100 articles / day. **A alone** = ~140–170 by Monday with normal friction.

## What needs deciding

1. Accept realistic forecast of **140–150 by Monday** and finish remainder Tuesday/Wednesday — or push for 170 hard?
2. If pushing: authorise **B** (parallel stream, Nexern account) and/or **D** (request Sonnet virtual key from Debanjan).
3. **C** (shorter articles) requires Joel sign-off before applying.
4. **E** is the lowest-leverage and lowest-quality option; avoid unless deadline immovable.

JPW picks; I execute Friday morning.

---

*Doc location: `/docs/status/2026-05-28-batch1-pace-analysis.md` — public on GitHub Pages.*
*Prepared by Claude Code session for JPW. Numbers verified against Asana inbox + Inventory Excel + WordPress as of 2026-05-28 18:30 UTC.*
