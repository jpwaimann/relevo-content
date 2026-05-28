# Incident — Claude Team session-limit during batch-50 launch

**Date**: 2026-05-27 (Wednesday PM, UTC)
**Severity**: Medium — operational throughput interruption, no data loss
**Status**: Resolved (post-incident); preventive strategy in [ADR-015](../adr/015-anti-limit-production-strategy.md)
**Reported by**: JPW (28/5 AM)
**Authored by**: JPW + Claude Code

---

## Summary

While launching the 50-article pilot batch of the 170-evergreen sprint, the main Claude Code session hit a **Claude Team token quota** (5-hour rolling window). The session was suspended around 23:00 local / pre-12:50 UTC, with the message `You've hit your session limit · resets 12:50am (UTC)`. At that point, 15/50 articles had been delivered through three sub-agent waves; a fourth wave of 5 Tenis sub-agents was running in background and got killed mid-pipeline. None of the wave-4 articles wrote `article_final.json`, so none could be uploaded to WordPress before the cut.

No editorial work was lost: the 15 delivered articles are in WordPress as `pending` drafts with Asana review subtasks assigned to Joel. The 5 in-progress Tenis dirs are preserved under `/tmp/relevo-*` and contain partial pipeline artifacts (article HTML, hero JPGs, research notes, critique JSON).

---

## Timeline (UTC)

| Time | Event |
|---|---|
| 19:21 | Session start. Asana audit on the 4 morning smoke subtasks. |
| 19:38–20:50 | Style guide v1.2.3 + ADR-014 drafted, committed (`63449f3`), pushed. Daily entry 27/5 + agenda 28/5 written. One-pager refresh (`6d48544`). |
| 20:50 | Decision to launch pilot batch of 50 (15/15/15/5 split, tier-1 first, 6 parallel agents, snapshot every 5). |
| 21:06 | Wave 1 launched: 5 Football sub-agents (rows 3, 6, 8, 10, 150). |
| 21:06–21:32 | Wave 1 delivered: 5/5, avg score 90.6. Main session uploads each to WP as notifications arrive. |
| 21:37 | Wave 2 launched: 5 Football sub-agents (rows 4, 5, 7, 152, 153). |
| 21:37–21:42 | Wave 2 delivered: 5/5, avg score 90.4. |
| 21:54 | Wave 3 launched: 5 Football coaches (rows 154, 155, 156, 157, 158). |
| 21:54–22:09 | Wave 3 delivered: 5/5, avg score 91.4. **15/50 cumulative.** |
| 22:07 | Wave 4 launched: 5 Tenis sub-agents (rows 11, 12, 13, 15, 16). |
| ~22:30 | Main session attempted to spawn a research dispatch task; transcript shows session-limit message `You've hit your session limit · resets 12:50am (UTC)`. |
| 22:30–23:00 | Background sub-agents kept running but unable to report; some reached the image-generation stage. |
| 12:50 (28/5) | Quota window resets. |
| 28/5 AM | Session resumed; diagnosis of partial Wave 4 artifacts; this report drafted. |

---

## Token-burn accounting

Estimated Opus 4.7 (1M context) tokens consumed in the affected ~5-hour window:

| Component | Calls | Avg tokens | Subtotal |
|---|---|---|---|
| Wave 1 sub-agents | 5 | ~115k | ~575k |
| Wave 2 sub-agents | 5 | ~115k | ~575k |
| Wave 3 sub-agents | 5 | ~115k | ~575k |
| Wave 4 sub-agents (3 active, 2 killed early) | 3 | ~120k | ~360k |
| Main session: file reads, MCP, dispatches, 15 uploads | — | ~350k | ~350k |
| Style guide v1.2.3 edits + ADR-014 + one-pager + 4 commits | — | ~50k | ~50k |
| **Total** | — | — | **~2.5M tokens** |

The `Claude Team` plan does not publish a precise 5-hour Opus cap, but observed behavior across this incident and prior sessions points to a soft ceiling around **~2.0–2.5M Opus tokens per rolling 5h window per user**. Crossing that line triggered the session suspension.

---

## What actually broke (and what did not)

**Worked correctly**
- Sub-agent fan-out architecture: 3 successful waves with consistent quality (avg 90.6/90.4/91.4).
- Anti-VPN-drop memory snapshots after every 5 runs: state was recoverable on resume.
- Inventory locking via SharePoint MSAL: no double-pick, no race conditions.
- WP / Asana / inventory finalize helpers held up.

**Broke under load**
- Total token budget for the day: we underestimated how much main-session orchestration (uploads, dispatching, finalize calls × 15) would cost on top of sub-agent generation. The morning estimate ("38M tokens needed for 170 articles") was correct in magnitude but did not break out the per-window ceiling.
- Wave-4 wall-clock: launching wave 4 at 22:07 with the quota window already 70%+ consumed was a foreseeable mistake. The pilot plan called for "snapshot every 5" but not "reassess token burn before each wave."
- Misread the sidebar token counters of background sub-agents: two of them showed tiny totals (444 and 728 tokens) at the moment of capture, which I interpreted as "killed on launch by rate-limit." Inspection of `/tmp` dirs the next morning showed both had actually produced substantial content. The sidebar counters froze at arbitrary points during background execution and are not a reliable completion signal.

**Did not break, but is fragile**
- The sub-agent prompt does not include a hard stop on token usage. An agent that hits its own internal quota for tools (WebSearch, MCP) may stall silently.
- The pipeline does not write a partial `article_final.json` mid-stream — only at the end. Recovery from a killed sub-agent requires either a full retry or manual inspection of `/tmp` artifacts.

---

## Wave-4 forensics

Five rows locked as `in_progress`; none reached `article_final.json`. Recovery state in `/tmp`:

| Row | Topic | Stage reached | Files present |
|---|---|---|---|
| 11 | Tenis · rules · ITF main rules | research + draft + critique probable | `article.html` 14k, `draft.html`, `research.md`, IFAB PDF |
| 12 | Tenis · rules · Wimbledon 5th-set tie-break | research + draft + **image-gen done** | `article.html` 9k, `hero.jpg` 252k, `jobs.json` |
| 13 | Tenis · tactics · Serve-and-volley | research + draft initial | `article_draft.html`, `research.md`, `RUN_ID` |
| 15 | Tenis · origins · Alcaraz | research + draft | `article.html` 20k, `research.md` |
| 16 | Tenis · origins · Landaluce | research + draft + critique | `article.html` 17k, `critique.json`, `research.md` |

Row 12 in particular got within one step of completion — only `article_final.json` write + WP upload remained.

---

## Root cause

Two compounding factors:

1. **Capacity miscalculation.** The per-window Opus quota on `Claude Team` is roughly half of what an unbounded "lanzar de a 50 con 6 agentes paralelos" plan implies. We measured throughput in articles/hour but not in tokens/window.
2. **Wave-4 timing.** Wave 4 was launched at a point where ~80% of the window quota had already been consumed by Waves 1–3 + main-session work. No checkpoint forced reassessment.

---

## Remediation

Implemented today (28/5):
- This incident report committed to `/docs/incidents/` for institutional memory.
- New strategy ADR-015 — "Anti-limit production strategy" — quantifies the budget and prescribes a sustainable cadence.
- Memory entry on the sidebar-token-counter misread (forensic lesson, not a code change).

To be implemented as part of ADR-015 rollout:
- Pre-wave token-burn checkpoint in the bulk pipeline.
- Wave-recovery script that can resume a killed sub-agent from `/tmp` artifacts (Row-12 case: skip to image-attach + WP upload; Rows 11/15/16: resume from post-critique).
- Conservative defaults for wave size and parallelism (see ADR-015).

---

## Open question for JPW

The 5 Tenis rows are recoverable (forensics above). Two options:

- **Recover** — finalize each from its `/tmp` state with targeted sub-agent runs. Cheaper (~50% of full-pipeline tokens) but operationally messy: each dir is at a different stage.
- **Retry** — release the locks, wipe `/tmp` dirs, re-launch as part of the next wave under the ADR-015 cadence. Operationally clean.

Recommendation: **retry** under the new cadence. The total token saving from recovery (~300k) is small versus the engineering cost of a one-off recovery script that will not be reused. Operational simplicity wins.

---

## Addendum 2026-05-28 PM — failed nested-Agent architecture + revised model split

JPW approved Option A (Opus orchestrator + Sonnet delegated research+revise) earlier in the day. Wave 1 of the retry batch (rows 11, 12, 13) dispatched 4 sub-agents using the Agent tool with nested delegation to `model: sonnet`.

**The architecture failed**: 3/3 sub-agents reported that the `Agent` tool was **not present in their deferred-tool list** and ran the full pipeline as plain Opus. No Sonnet delegation happened. Operationally the runs produced strong articles (scores 90, 95, 95) — just at the cost profile of yesterday, not the projected Option A profile.

The root cause is a runtime constraint of this Claude Code harness: sub-agents spawned via the Agent tool cannot themselves spawn nested sub-agents with a different model. Nested delegation is not available in this environment.

### Pivot: real token-reduction levers (Plan A revised)

JPW also rejected a proposed bucket-shift to LiteLLM Opus 4.6 (claude-opus-4-6 via Nexern's LiteLLM proxy) on the grounds that total consumption stays the same — only the billing account changes. The decision instead is to pursue **levers that actually reduce token consumption**:

| # | Lever | Mechanism | Picked |
|---|---|---|---|
| 1 | **Gemini 2.5 Pro for research** | Gemini ~10x cheaper per token than Opus. Research is fact-gathering — Gemini handles it well. Output feeds the Opus draft (voice stays Opus). | ✓ |
| 2 | **Prompt caching for `style_guide.md` + templates** | ~90% input-token discount on repeated reads across waves. | ✓ |
| 3 | Shorter word-count target (1500-2200 → 1200-1500) | Editorial decision pending Joel's input. | Deferred |
| 4 | Cap revise to 0 iterations | Lower risk of multi-pass loops. | Not picked |
| 5 | **GPT-5 for fact-check** | JPW: "ChatGPT anda bastante bien para búsqueda." Lower per-token cost than Opus. | ✓ |
| 6 | GPT-5 for draft | Quality risk on Spanish journalism voice. | Rejected |

The selected combination (1+2+5) plus **INFRA-08** (finalize-as-CLI; see backlog) is the path executing today.

### Per-article token reporting (JPW explicit requirement)

Every `article_final.json` now carries a `token_usage` block with the breakdown:
```
{
  "opus_session":         <tokens consumed on the Claude Team window>,
  "gemini_research":      <tokens consumed via LiteLLM Gemini>,
  "gpt5_factcheck":       <tokens consumed via LiteLLM GPT-5>,
  "cached_input_tokens":  <input tokens read from prompt cache>,
  "uncached_input_tokens":<input tokens not in cache>
}
```
Aggregated to a batch CSV at the end of the run.

### Internal finance alignment

A standalone Asana task (`1215215371482362`) tracks JPW's periodic check-in with the Nexern team on LiteLLM usage from a financial perspective. Production is not blocked on the alignment — it runs today under the revised plan regardless.
