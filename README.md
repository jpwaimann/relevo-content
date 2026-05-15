# Relevo Content Workflow

Automated SEO content generation for [relevo.com](https://relevo.com). 24 sports, 3 content categories (Rules & Basics, Tactics Explainers, Origins), multi-pass quality gate. Built as a Claude Code skill replacing the legacy n8n workflow retired on 2026-05-14.

**Current phase:** Editorial validation (Phase 4 of 7). Style guide v1 deployed, 10-article QA batch generated, awaiting writer review to lock v1.1.

**Live site:** [jpwaimann.github.io/relevo-content](https://jpwaimann.github.io/relevo-content/) (GitHub Pages from `main`).

---

## Roadmap

The project runs from May 1 through expected scale-out in late June. Phases 1–3 are complete; Phase 4 is the current focus. The Gantt below uses weekly granularity.

```mermaid
gantt
    title Relevo Content Workflow — Roadmap (weekly)
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section Phase 1 · n8n prototype
    Personal n8n workflows + prompts   :done, p1a, 2026-05-01, 7d
    Corporate migration + v17 stable   :done, p1b, 2026-05-08, 6d

    section Phase 2 · CC-native pivot
    Architecture decision              :done, p2a, 2026-05-14, 1d
    Skill scaffold + style guide v1    :done, p2b, 2026-05-14, 2d
    Templates (Rules/Tactics/Origins)  :done, p2c, 2026-05-14, 1d
    Smoke test (score 95)              :milestone, m1, 2026-05-14, 0d

    section Phase 3 · Testing & batch
    10-article QA batch                :done, p3a, 2026-05-15, 1d
    Public hosting (GitHub Pages)      :done, p3b, 2026-05-15, 1d

    section Phase 4 · Editorial validation
    Writer review of batch             :active, p4a, 2026-05-18, 5d
    Style guide v1.1                   :p4b, 2026-05-20, 5d

    section Phase 5 · Production integrations
    SharePoint inventory API           :p5a, 2026-05-25, 5d
    WordPress + Asana MCP              :p5b, 2026-05-25, 5d
    LiteLLM image models               :p5c, 2026-05-27, 4d

    section Phase 6 · First prod batch
    Reviewer workflow live             :p6a, 2026-06-01, 5d
    First production batch             :milestone, m2, 2026-06-05, 0d

    section Phase 7 · Scale
    Onboard 2nd editor                 :p7a, 2026-06-08, 7d
    Scale to 50 articles per day       :p7b, 2026-06-15, 14d
```

### Phase summary

| Phase | Dates | Status | What |
|---|---|---|---|
| 1 — n8n prototype | May 1 – May 13 | ✅ done | Personal n8n workflows; corporate migration to `n8n.bidwhig.com`; v17 stable |
| 2 — CC-native pivot | May 14 | ✅ done | Skill `/relevo` scaffolded; style guide v1; templates for 3 categories; sports 15 → 24; smoke test score 95 |
| 3 — Testing & batch | May 15 | ✅ done | 10-article QA batch (mean 90.3/100); GitHub Pages site |
| 4 — Editorial validation | May 18 – 22 | 🔴 current | Writer reviews batch; style guide v1.1; template refinements |
| 5 — Production integrations | May 25 – 29 | ⏳ next | SharePoint API; WordPress MCP; Asana MCP; LiteLLM image models |
| 6 — First prod batch | Week of June 1 | ⏳ later | Full reviewer workflow live; first production articles |
| 7 — Scale | June 8 onward | ⏳ later | Second editor onboarded; throughput ramps to 50/day |

---

## Repository structure

```
.
├── index.html                              # landing page (this content rendered)
├── README.md                               # repo README, mirrors the index
├── docs/
│   ├── project-overview.html               # project brief — read first
│   └── README.md                           # technical architecture of the /relevo skill
├── _meta/
│   ├── topics.json                         # topic list for the May 15 batch
│   └── AGENT_CONTRACT.md                   # the contract every generation subagent follows
└── runs/                                   # one directory per generated article
    ├── relevo-rules-smoke-1778792284/      # first smoke test
    └── art-N-1778849077/                   # May 15 batch articles (N = 1..10)
        ├── article.html                    # final version (the publishable output)
        ├── article_draft.html              # pre-revise draft (for traceability)
        ├── article_final.json              # metadata + quality journey
        ├── critique_pass1.json             # first auto-critique pass
        └── critique_pass2.json             # post-revise auto-critique (when applicable)
```

Each generation run produces a self-contained directory. To understand any single article: open `article.html` for the final version, then drill into `critique_pass1.json` to see what the automated critic flagged in the first draft, and into `critique_pass2.json` to confirm the issues were resolved.

---

## Project overview

**Relevo Content** is an automated SEO content generation system for relevo.com. It produces editorial-quality Spanish-language articles across 24 sports and three content categories using a multi-pass pipeline that includes automated critique and fact-checking before any human review.

### Why we rewrote it

The previous n8n implementation worked for a limited topic set but had three structural limitations:

1. **Prompt rigidity.** Editorial voice was encoded inside individual workflow nodes; style changes required a coordinated PR + redeploy across multiple nodes.
2. **No quality gate.** Drafts reached human reviewers without any automated critique stage. Inventions, length misses, and format inconsistencies consumed editorial time.
3. **Limited coverage.** Only 15 sports and 2 categories. Expansion required hand-editing workflow branches.

The Claude Code skill addresses all three. Prompts and templates are versioned markdown that anyone can read and propose changes to. Every draft passes through automated critique against a 15-point editorial checklist before reaching humans. Coverage expands to 24 sports and adds Origins (athlete biographies) as a third category.

### Pipeline

Each article is produced by a deterministic seven-stage pipeline:

1. **Research** — reads the official rulebook PDF (SharePoint) plus authoritative sources.
2. **Draft** — first version against the category template and the Relevo style guide.
3. **Critique pass 1** — internal critic evaluates against the 15-point checklist; returns score 0–100 plus priority fixes.
4. **Revise** — if score < 80 or anti-hallucination check fails, applies fixes. Skipped if the draft passes the gate.
5. **Critique pass 2** — re-scores the revised version. Loop repeats until gate passes or retries exhausted.
6. **Fact-check** — verifies claims; unverifiable specific attributions are softened or removed.
7. **Publish** — uploads to WordPress as `pending_review` and creates an Asana subtask for the reviewer.

The full intermediate state of every article is preserved on disk, giving editors complete traceability into how each piece was produced.

### Cost structure

| Item | Monthly estimate | Notes |
|---|---|---|
| Text generation | $0 marginal | Covered by editor's Claude Code subscription |
| Image generation (gpt-image-2) | $60 – $240 | 50 articles/day × 1–2 images each |
| WordPress / Asana / Graph APIs | $0 | Within standard quotas |
| Infrastructure | $0 | No servers to operate |

For the full project brief — written for someone joining the project — see [`docs/project-overview.html`](docs/project-overview.html).

---

## QA & Testing

Two artifacts demonstrate the pipeline working end-to-end: a single-article smoke test on May 14, and a 10-article quality batch on May 15.

### Smoke test · May 14

The first end-to-end run produced **"Qué es el fuera de juego en fútbol y cómo funciona"**. The pipeline caught and corrected a critical issue automatically.

| Stage | Result |
|---|---|
| Draft | 850 words. Critic returned score 72. |
| Critique pass 1 | Flagged an unverifiable specific attribution and one 78-word paragraph. Length below target. |
| Revise pass 1 | Three priority fixes: removed the attribution, split the long paragraph, extended to 1,592 words with two added H2 sections. |
| Critique pass 2 | **Score 95.** Gate passed. Anti-hallucination resolved. |
| Fact-check | 8 claims verified, 0 corrected. |
| Total elapsed | **132 seconds** |

Artifacts: [final article](runs/relevo-rules-smoke-1778792284/article.html) · [metadata](runs/relevo-rules-smoke-1778792284/article_final.json) · [critique pass 1 (72)](runs/relevo-rules-smoke-1778792284/critique_pass1.json) · [critique pass 2 (95)](runs/relevo-rules-smoke-1778792284/critique_pass2.json)

The automated critic caught a fabricated incident before it reached a human reviewer. This is exactly the failure mode the multi-pass design exists to prevent.

### May 15 batch · 10 articles for editorial review

A balanced sample across the three content categories and five representative sports, generated in approximately 17 minutes of wall clock time.

**Summary:** 10 articles · mean score 90.3 · 19,500 words · 0 gate failures.

| # | Sport | Category | Topic | Score | Words | Final |
|---|---|---|---|---|---|---|
| 1 | Football | tactics | Player positions | 86 | 1,791 | [open](runs/art-1-1778849077/article.html) |
| 2 | Football | tactics | Types of shots | 93 | 1,836 | [open](runs/art-2-1778849077/article.html) |
| 3 | Basketball | rules | FIBA & NBA rules | 92 | 1,995 | [open](runs/art-3-1778849077/article.html) |
| 4 | Basketball | tactics | Types of shots | 91 | 1,683 | [open](runs/art-4-1778849077/article.html) |
| 5 | Tennis | rules | Scoring & tie-break | 88 | 1,998 | [open](runs/art-5-1778849077/article.html) |
| 6 | Tennis | origins | Origins of Carlos Alcaraz | 88 | 2,604 | [open](runs/art-6-1778849077/article.html) |
| 7 | Formula 1 | rules | FIA regulations | 91 | 2,000 | [open](runs/art-7-1778849077/article.html) |
| 8 | Formula 1 | tactics | Pit-stop strategy | 93 | 1,986 | [open](runs/art-8-1778849077/article.html) |
| 9 | MMA | origins | Origins of Ilia Topuria | 89 | 2,567 | [open](runs/art-9-1778849077/article.html) |
| 10 | MMA | rules | UFC rules | 92 | 1,998 | [open](runs/art-10-1778849077/article.html) |

Each entry links to the final published HTML. The full bundle of artifacts is available in each `runs/art-N-1778849077/` directory.

### What reviewers are asked to evaluate

1. **Voice and tone.** Do these read like Relevo articles? Are there expressions or rhythms in the style guide that should change?
2. **Category templates.** Do Rules, Tactics, and Origins each have the structure the team expects?
3. **Anti-hallucination behavior.** Calibrated correctly, or too conservative?
4. **Asana review workflow.** Round-robin assignment, SLA, criteria for return-vs-approve — open for definition by editorial.

---

## Ownership

- **Pablo Waimann** — technical owner · [pablo.waimann@igaming.com](mailto:pablo.waimann@igaming.com)
- **Main content writer** — owner of the style guide; validates voice and approves structural changes
- **Reviewers** (round-robin) — approve or return articles before publication
- **BI team (Debanjan)** — Azure, SharePoint, and credentials
