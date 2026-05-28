# ADR-015: Anti-limit production strategy — sustainable mass content cadence

**Status**: Approved
**Date**: 2026-05-28
**Driver**: JPW (post-incident review of [2026-05-27 session-limit](../incidents/2026-05-27-session-limit-batch50.md))
**Supersedes**: implicit "launch 6 agents in parallel, snapshot every 5" rule from [ADR-013](013-content-strategy-v1-and-bulk-170-week.md)
**Related**: [ADR-013](013-content-strategy-v1-and-bulk-170-week.md), [SOP 01-content-production](../sop/01-content-production.md), [incident 2026-05-27](../incidents/2026-05-27-session-limit-batch50.md)

---

## Problem

The 27/5 launch of the 50-article pilot batch hit a `Claude Team` session-limit at 15/50 delivered, killing Wave 4 mid-flight. The 170-evergreen sprint needs a production cadence that respects the platform's hidden token caps while still delivering mass output within the FIFA WC window. The current ad-hoc "launch waves until something breaks" approach is not viable.

## Decision

Adopt a quantitative, budget-driven production strategy with the following rules.

### 1. Token-budget model

Per-article cost, measured empirically from yesterday's 15 successful runs:

| Component | Token cost (avg) |
|---|---|
| Sub-agent pipeline (research → draft → critique → revise → fact-check → SEO → image) | **115k** |
| Main-session finalize (hero upload + posts_create + posts_update + Asana subtask + inventory update) | **25k** |
| **Total per article** | **~140k** |

Per-window (5-hour rolling) Opus 4.7 budget on `Claude Team` plan: empirically observed ceiling at **~2.0–2.5M tokens** (the 27/5 incident triggered at ~2.5M). For safety, **plan against a 1.8M effective budget per window** — leaves ~30% headroom for orchestration, retries, ad-hoc work, and metric drift.

**Effective articles per window**: `1.8M / 140k ≈ 12 articles`.

### 2. Daily and sprint cadence

| Horizon | Capacity | Rationale |
|---|---|---|
| Per 5h window | **12 articles** | budget math above |
| Per working day (2 windows × 5h with 30-min gap) | **20–24 articles** | 2nd window is shorter in practice; reviewer cycles |
| 170-article sprint | **7–8 working days** | 170 / 22 avg |

The 170 sprint that was originally framed as "1 day if possible" becomes **7–8 working days** (rough wall-clock 2026-05-28 through 2026-06-06, fitting inside the pre-FIFA-WC window).

### 3. Wave structure

Each "wave" is a single fan-out of N sub-agents that run concurrently. Within one window:

- **Wave size**: 4 sub-agents (not 5 or 6). Rationale: 4 × 115k = 460k for sub-agent burn; plus 4 × 25k = 100k for main-session finalize = 560k. Three waves fit in 1.8M with ~120k of margin.
- **Concurrency inside a wave**: all 4 agents run in parallel — no slow-rolling. Rate-limit risk on WebSearch / SharePoint is acceptable at N=4 (yesterday showed N=5 also worked but pushed contention).
- **Sequential waves**: do not launch the next wave until the previous wave's articles are uploaded and finalized. This prevents overlap and gives a clean token-burn measurement point.
- **Pre-wave gate**: before launching a wave, compute remaining window budget. If `remaining < 600k`, hold and wait for the next window to open.

### 4. Pre-wave checklist (operator-visible)

The bulk dispatcher (whether the `/relevo new` skill or a sub-agent loop) prints this before each wave:

```
Window opened at: <UTC timestamp>
Time remaining in window: <minutes>
Tokens consumed in window: <estimated>
Budget remaining: <1.8M - consumed>
Wave size proposed: 4 articles × 140k = 560k
Decision: PROCEED / HOLD <reason>
```

`HOLD` triggers when remaining < 600k. In a `HOLD`, the bulk pause-state is written to memory; next session resumes from the same wave with the same row IDs.

### 5. Quality gates remain unchanged

Article-level quality floor (per [ADR-013](013-content-strategy-v1-and-bulk-170-week.md)):
- Per-article score ≥ 85 (Tier-1: ≥ 88)
- Batch average score ≥ 89
- If a wave's average drops below 87 → pause batch, escalate to JPW before next wave.

### 6. Future optimization (NOT in scope for the current sprint)

Investigate moving the WP + Asana + inventory finalize step into a CLI script (`scripts/finalize_article.py`) that sub-agents can call directly. The proxy-based `ig-mcp-proxy` is reachable via stdio, so a wrapper script is possible. If shipped, this saves ~25k × N tokens per batch on the main session, raising effective per-window throughput from 12 to ~15 articles. Tracked in backlog as `INFRA-08`. Related: `PIPE-02` (Sonnet smoke test for Option B/C model split, which decides the final pipeline shape before `INFRA-08` ships).

## Consequences

**Positive**
- Predictable cadence: JPW can plan reviewer load (Joel) and milestone dates against a known articles/day rate.
- No more mid-window crashes. Operator sees the budget before each wave and pauses gracefully when needed.
- Wave size of 4 has the same throughput-per-window as 5 (because the bottleneck is the token budget, not concurrency) but with lower contention risk on WebSearch / SharePoint.

**Negative**
- Sprint length extends from "1 day" aspirational to **7–8 working days** realistic. This is a one-time honest reframing — the original 1-day estimate was based on wall-clock, not token-cost reality.
- A wave that hits `HOLD` mid-sprint introduces a context-restore tax at the start of the next session (~50k tokens for catching up on what we were doing). Net effect: the 7–8 day figure includes those resume taxes.
- The 4-articles-per-wave rule means the first wave of any window only produces 4 deliveries before checkpoint — slower visible feedback than 5 or 6 at-a-time.

**Risks accepted**
- The 1.8M budget is empirical, not documented by Anthropic. If `Claude Team` changes its cap, the strategy needs re-baselining within 1–2 days.
- Sub-agent token cost varies (Tier-1 + fact-check heavy = 130k; simple Rules = 100k). The 115k average smooths this but extreme outliers (rich biographies with 15+ WebSearch calls) can push a wave over budget.

## How to apply (operator quick-reference)

1. Start of session: check the time and estimate which window we are in.
2. Before any wave: run the **pre-wave checklist** (above). If `HOLD`, snapshot to memory and stop.
3. Wave size: **4** sub-agents.
4. Wait for all 4 to finalize (WP upload + Asana subtask + inventory advance) before launching the next.
5. After each wave: brief commit + memory snapshot. Anti-VPN-drop policy from [feedback_vpn_session_loss](../sop/01-content-production.md#vpn-stability-procedures) stays in force.
6. End of window: write a single-line accounting note to memory (`window 2026-05-28-AM: 12 articles, 1.65M tokens, avg score 90.3`).

## Reviewer

JPW (sign-off pending). Joel does not gate this ADR — it is operational, not editorial.

---

## Update 2026-05-28 PM — Option A original architecture failed; revised plan

The original Option A required Opus orchestrator sub-agents to spawn nested Sonnet sub-sub-agents via the Agent tool for the research and revise stages. Confirmed in 3/3 Wave-1 runs (rows 11, 12, 13) that **nested Agent invocation is not available in this Claude Code harness** — the Agent tool is absent from the sub-agent's deferred-tool list. All Wave-1 articles ran full Opus despite the prompt instructing delegation.

JPW then rejected a fallback to LiteLLM Opus 4.6 on the grounds that bucket-shifting Nexern's account budget is not real consumption reduction. The decision is to pursue **real token-reduction levers** instead:

### Revised pipeline (effective 2026-05-28 PM)

| Stage | Model | Why |
|---|---|---|
| Research | **Gemini 2.5 Pro** via LiteLLM | ~10x cheaper per token than Opus; fact-gathering quality is high; output feeds Opus draft so voice risk is contained. |
| Draft | Opus 4.7 (Claude Code session) | Editorial voice quality stays on the strongest model. |
| Self-critique | Opus 4.7 | Quality gate decisions stay on Opus. |
| Revise (when needed) | Opus 4.7 | Surgical fixes on the draft Opus produced — same model for consistency. |
| Fact-check | **GPT-5** via LiteLLM | JPW: "ChatGPT anda bastante bien para búsqueda." Cheaper per token than Opus for the WebSearch-heavy stage. |
| SEO + image + save | Opus 4.7 (deterministic + image gen call) | No change. |
| Finalize (WP+Asana+inventory) | None — pure Python via `scripts/finalize_article.py` | INFRA-08 lands today: zero LLM cost. |

Plus **prompt caching** on the `style_guide.md` + per-category templates (~30k tokens), which are loaded once per sub-agent run. Anthropic's prompt-cache discount (~90% on cached input tokens) applies to all repeated reads across waves.

### New per-article token reporting

Each `article_final.json` now includes a `token_usage` block:

```json
{
  "opus_session":          145000,
  "gemini_research":         32000,
  "gpt5_factcheck":          18000,
  "cached_input_tokens":     27000,
  "uncached_input_tokens":   85000
}
```

Aggregated batch CSV at end of run for finance reporting.

### Token-budget update (Plan A revised)

Per article, post-revision:

| Component | Tokens | Bucket |
|---|---|---|
| Opus session (draft + critique + revise + SEO + image) | ~60-65k | Claude Code Team window |
| Gemini 2.5 Pro (research) | ~30-40k Gemini | LiteLLM (Nexern, separate cost) |
| GPT-5 (fact-check) | ~15-25k GPT-5 | LiteLLM (Nexern, separate cost) |
| Cache hits on style_guide+templates | ~25-30k input @ 10% effective | Claude Code Team window |
| Main session (post-INFRA-08) | ~5k | Claude Code Team window |
| **Total Opus on the constrained window** | **~65-70k per article** | — |

Effective per-window throughput: **1.8M / 70k ≈ 25-26 articles per window** under the revised plan. Combined with the eliminated main-session finalize cost (INFRA-08), the binding constraint shifts from the per-window Opus budget to wall-clock and reviewer bandwidth.

The 32 remaining articles of the Tier-1 batch fit in ~1.3 windows — well within a single day under the revised architecture.

### Sub-agent ↔ LiteLLM interface

The Opus sub-agent invokes `scripts/llm_call.py` via Bash for the Gemini and GPT-5 stages. The helper:
- Reads the prompt from a file (avoids inlining 30k+ token style_guide into the Bash arg).
- Calls LiteLLM with the appropriate model name and the `cache_control` parameter for the cacheable style-guide prefix.
- Returns the generated text plus a `usage` block (input_tokens, output_tokens, cached_input_tokens) which the sub-agent merges into its running `token_usage` block.

The sub-agent owns the orchestration; main session is a thin dispatcher post-INFRA-08.

### LiteLLM internal alignment

Tracked in Asana task `1215215371482362` (JPW pings the Nexern team periodically on the financial impact of LiteLLM usage). Production runs under the revised plan regardless of when that alignment lands.
