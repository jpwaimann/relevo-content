# Subagent contract — `/relevo new` pipeline

Each agent generates ONE SEO article (Spanish, peninsular) for Relevo.com by running the full multi-pass pipeline.

## Inputs

- `deporte`, `categoria` (rules | tactics | origins), `tipo`, `seo_title`, `meta_description`, `url_slug`, `hub_slug`. For origins also: `nombre`, `gancho`, `sub_hub`.
- `run_id` and `output_dir` = `/var/www/relevo/runs/<run_id>/`.

## Canonical files to read BEFORE generating

1. `/home/pablo/relevo-content/prompts/style_guide.md` — Relevo voice manual (mandatory 15-point checklist).
2. Category template:
   - rules → `/home/pablo/relevo-content/prompts/rules_template.md`
   - tactics → `/home/pablo/relevo-content/prompts/tactics_template.md`
   - origins → `/home/pablo/relevo-content/prompts/origins_template.md`
3. `/home/pablo/relevo-content/data/sports_config.json` — sub-hubs and rulebook availability per sport.
4. (Reference for expected quality) `/var/www/relevo/runs/relevo-rules-smoke-1778792284/article.html` — the smoke test that reached score 95.

## Pipeline — stages in order

### 1. draft
Generate the article HTML using the category template. Length targets:
- rules / tactics: 1,500–2,000 words
- origins: 2,500–3,000 words

Hard requirements:
- Semantic HTML (`<h2>`, `<h3>`, `<p>`, `<ul>`, `<strong>`, `<em>`, `<table>` only where it adds value). **Never** `<b>` or `<i>`.
- Peninsular Spanish journalistic voice per `style_guide.md`.
- Each H2 must reflect a real search intent. Zero generic H2s like "Introduction".
- No clichés or unjustified grandiose adjectives.
- Anti-hallucination: zero specific unverifiable data (dates, scores, specific matches you cannot confirm). If you need to give an example, use generic or universally known cases.

Write the draft to `<output_dir>/article_draft.html`.

### 2. critique_pass_1
Evaluate the draft against the 15 points of the style guide. Return JSON in this shape:
```json
{
  "run_id": "...",
  "pass": 1,
  "score": <0-100>,
  "score_breakdown": {
    "voz_periodistica": {"weight": 20, "obtained": <n>, "note": "..."},
    "html_semantic": {"weight": 20, "obtained": <n>, "note": "..."},
    "ritmo_parrafos": {"weight": 15, "obtained": <n>, "note": "..."},
    "antihallucination": {"weight": 15, "obtained": <n>, "note": "..."},
    "seo_no_crudo": {"weight": 10, "obtained": <n>, "note": "..."},
    "cierre_personalidad": {"weight": 10, "obtained": <n>, "note": "..."},
    "ortotipografia": {"weight": 10, "obtained": <n>, "note": "..."}
  },
  "checklist_15_points": [<15 entries: point, name, result (APPROVED|PARTIAL|FAIL), note>],
  "priority_fixes": [<list of actionable fixes>],
  "gate_decision": "PASS" | "REVISE_REQUIRED",
  "reason": "..."
}
```
Gate: PASS if score >= 80 AND no anti-hallucination FAIL. Otherwise REVISE_REQUIRED.

Write to `<output_dir>/critique_pass1.json`.

### 3. revise (only if REVISE_REQUIRED)
Apply the priority_fixes to the draft. Produce a new version.

### 4. critique_pass_2 (only if you revised)
Re-evaluate. Same JSON shape. Write to `<output_dir>/critique_pass2.json`.

### 5. fact-check
Identify the 8–12 strongest claims in the article and verify they are reasonably checkable (you do not need to run web search; ensure you are not inventing specific attributions). Record the result in the final JSON.

### 6. final output

Write the final version to `<output_dir>/article.html`.

Write `<output_dir>/article_final.json` with:
```json
{
  "run_id": "...",
  "deporte": "...",
  "categoria": "...",
  "tipo": "...",
  "metadata": {
    "seo_title": "...",
    "meta_description": "...",
    "url_slug": "...",
    "hub_slug": "...",
    "sub_hub": "...",
    "word_count": <int>,
    "html_size_bytes": <int>
  },
  "files": {
    "html": "<output_dir>/article.html",
    "critique_pass1": "<output_dir>/critique_pass1.json",
    "critique_pass2": "<output_dir>/critique_pass2.json or null if passed in 1"
  },
  "quality_score": <final score>,
  "quality_journey": {
    "draft_pass": "Generated, <n> words",
    "critique_pass_1": "Score <n> — <PASS|REVISE_REQUIRED> — <short reason>",
    "revise_pass_1": "<applied X fixes | skipped>",
    "critique_pass_2": "<Score <n> — PASS | skipped>",
    "fact_check": "<n claims verified, <m> adjusted>"
  },
  "stages_completed": [...],
  "duration_total_seconds": <approx>,
  "mode": "full_qa_batch_call_2026-05-15"
}
```

## Output report to the supervisor

One line: `<run_id> | <deporte> | <categoria> | score=<n> | words=<n> | pass=<1|2>`.

## DO NOT

- Upload to WordPress (mock mode).
- Create Asana subtasks (mock mode).
- Generate the hero image (LiteLLM proxy lacks image models).
- Edit files outside `<output_dir>`.
- Read or write the system memory.
