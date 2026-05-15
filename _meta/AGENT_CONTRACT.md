# Contrato para subagentes — pipeline /relevo new

Cada agente genera UN artículo SEO en castellano para Relevo.com siguiendo el pipeline multi-pass.

## Inputs

- `deporte`, `categoria` (rules|tactics|origins), `tipo`, `seo_title`, `meta_description`, `url_slug`, `hub_slug`. Para origins: `nombre` + `gancho` + `sub_hub`.
- `run_id` y `output_dir` = `/var/www/relevo/runs/<run_id>/`.

## Archivos canónicos a leer ANTES de generar

1. `/home/pablo/relevo-content/prompts/style_guide.md` — manual de voz Relevo (15-point checklist).
2. Template:
   - rules → `/home/pablo/relevo-content/prompts/rules_template.md`
   - tactics → `/home/pablo/relevo-content/prompts/tactics_template.md`
   - origins → `/home/pablo/relevo-content/prompts/origins_template.md`
3. `/home/pablo/relevo-content/data/sports_config.json`.
4. Referencia de calidad esperada: `/var/www/relevo/runs/relevo-rules-smoke-1778792284/article.html`.

## Pipeline

### 1. draft
Generá artículo HTML. Targets:
- rules/tactics: 1500-2000 palabras
- origins: 2500-3000 palabras

Reglas duras:
- HTML semántico: `<h2>`, `<h3>`, `<p>`, `<ul>`, `<strong>`, `<em>`, `<table>` solo donde aporte. **NUNCA** `<b>` ni `<i>`.
- Voz periodística española peninsular según style_guide.
- Cada H2 = intención de búsqueda real. Cero "Introducción".
- Anti-hallucination: cero datos específicos no verificables (fechas/scores/partidos concretos no confirmables). Si hay que ejemplificar, usá casos genéricos o muy conocidos.
- Sin clichés ni adjetivos grandilocuentes injustificados.

Salida intermedia → `<output_dir>/article_draft.html`.

### 2. critique_pass_1
Evaluá contra los 15 puntos. JSON:
```json
{
  "run_id": "...", "pass": 1, "score": <0-100>,
  "score_breakdown": {
    "voz_periodistica": {"weight": 20, "obtained": <n>, "note": "..."},
    "html_semantic": {"weight": 20, "obtained": <n>, "note": "..."},
    "ritmo_parrafos": {"weight": 15, "obtained": <n>, "note": "..."},
    "antihallucination": {"weight": 15, "obtained": <n>, "note": "..."},
    "seo_no_crudo": {"weight": 10, "obtained": <n>, "note": "..."},
    "cierre_personalidad": {"weight": 10, "obtained": <n>, "note": "..."},
    "ortotipografia": {"weight": 10, "obtained": <n>, "note": "..."}
  },
  "checklist_15_points": [<15 entries: point, name, result(APPROVED|PARTIAL|FAIL), note>],
  "priority_fixes": [...],
  "gate_decision": "PASS" | "REVISE_REQUIRED",
  "reason": "..."
}
```
Gate: PASS si score ≥ 80 Y ningún anti-hallucination FAIL.

Salida → `<output_dir>/critique_pass1.json`.

### 3. revise (solo si REVISE_REQUIRED)
Aplicá priority_fixes.

### 4. critique_pass_2 (solo si revisaste)
Re-evaluá. → `<output_dir>/critique_pass2.json`.

### 5. fact-check
Verificá 8-12 claims más fuertes. Remové cualquier atribución específica no comprobable.

### 6. final output

- `<output_dir>/article.html` — versión final.
- `<output_dir>/article_final.json` con:
```json
{
  "run_id": "...", "deporte": "...", "categoria": "...", "tipo": "...",
  "metadata": {"seo_title":"...","meta_description":"...","url_slug":"...","hub_slug":"...","sub_hub":"...","word_count":<int>,"html_size_bytes":<int>},
  "files": {"html":"...","critique_pass1":"...","critique_pass2":"... o null"},
  "quality_score": <final>,
  "quality_journey": {"draft_pass":"...","critique_pass_1":"...","revise_pass_1":"...","critique_pass_2":"...","fact_check":"..."},
  "stages_completed": [...],
  "duration_total_seconds": <approx>,
  "mode": "full_qa_batch_call_2026-05-15"
}
```

## Output al supervisor

Una línea: `<run_id> | <deporte> | <categoria> | score=<n> | words=<n> | pass=<1|2>`.

## NO hagas

- No WP upload, no Asana, no imagen.
- No edites fuera de `<output_dir>`.
- No leas/escribas memoria.
