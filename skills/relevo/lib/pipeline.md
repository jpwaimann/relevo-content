# Lib: Multi-pass article generation pipeline

Detalle de las etapas de generación. Esto es lo que el comando `/relevo new` ejecuta entre lock y publish. Producción completa al 19/5/2026: cada stage tiene implementación real validada en un run end-to-end (post 16100, q=91).

## Pipeline overview

```
research → draft → self-critique → revise (loop max 2x) → fact-check → image → seo-enrich → publish
```

Cada etapa loggea `{stage}_started` y `{stage}_completed`. Cualquier fallo retriable usa el patrón de `retry.md`.

## Stage 1: research

**Goal**: reunir contexto factual + identificar referentes / ejemplos específicos para el artículo.

**Inputs**:
- Row del inventory (deporte, categoria, seo_title, meta_description, notes)
- Reglamento PDF del deporte (cuando esté disponible — leer de SharePoint, en mock omitir)

**Acción**:
1. Si hay reglamento PDF disponible, leerlo y extraer las secciones relevantes a la categoría:
   - Rules: secciones de normativa, dimensiones, sanciones
   - Tactics: secciones de posiciones, formaciones, técnicas (si las cubre el reglamento)
   - Origins: NO lee reglamento (irrelevante para bios)
2. Para Origins: usar WebSearch con queries como `"{nombre}" trayectoria carrera`, `"{nombre}" nacimiento familia origenes`, `"{nombre}" {deporte} debut`.
3. Para Rules/Tactics con tema específico: WebSearch para confirmar terminología actual y ejemplos icónicos del último año.
4. Output: un research_notes block estructurado:
   ```
   {
     "key_facts": [...],
     "iconic_references": [...],  // momentos/jugadores históricos relevantes
     "current_examples": [...],   // referencias del último año/temporada
     "potential_pitfalls": [...]  // áreas donde la voz Relevo puede equivocarse (nombres dudosos, fechas, etc.)
   }
   ```

**Output a memoria del run**: este research_notes va al draft stage como input.

## Stage 2: draft

**Goal**: producir la primera versión completa del artículo, siguiendo template y style guide.

**Inputs**:
- Style guide (`prompts/style_guide.md`)
- Template específico de la categoría (`prompts/rules_template.md` / `tactics_template.md` / `origins_template.md`)
- Row del inventory (todos los metadatos: seo_title, meta, slug, notes, nombre, gancho)
- Research notes del stage 1

**Acción**:
1. Composición del system prompt:
   ```
   [STYLE GUIDE COMPLETO]
   ---
   [TEMPLATE DE LA CATEGORÍA]
   ---
   ## Contexto específico del artículo
   - Deporte: {deporte}
   - Categoría: {categoria} / {tipo}
   - SEO Title objetivo: {seo_title}
   - Meta description objetivo: {meta_description}
   - URL slug: {url_slug}
   - {Notas del editor}
   ---
   ## Research notes
   {research_notes serializado}
   ```
2. User message: "Redacta el artículo completo cumpliendo el template y el style guide. Output: HTML semántico válido del cuerpo del artículo (no incluir `<html>`, `<head>` ni `<body>` — empezar directo en H1)."
3. Generar el draft.

**Output**: HTML string del artículo + word count estimado.

## Stage 3: self-critique

**Goal**: aplicar el 15-point checklist del style guide, dar score 0-100, y emitir lista priorizada de issues.

**Inputs**:
- El draft del stage 2
- El style guide (mismo system prompt context)
- El template de la categoría (para validar estructura)

**Acción**:
1. System prompt:
   ```
   [STYLE GUIDE]
   ---
   Tarea: evalúa el siguiente artículo Relevo contra el 15-point self-check del style guide.
   Para cada punto: APROBADO / FALLO / N-A, con justificación de 1 línea si falla.
   Asigna un score 0-100 según los pesos:
   - 20pts: voz Relevo + léxico peninsular (puntos 1, 3, 4 del checklist)
   - 20pts: HTML semántico (puntos 5, 9, 10)
   - 15pts: ritmo y párrafos (punto 2)
   - 15pts: anti-hallucination (punto 14)
   - 10pts: SEO no crudo (punto 8)
   - 10pts: cierre con personalidad (punto 15)
   - 10pts: ortotipografía (puntos 11, 12, 13)
   Adicionalmente: revisa adherencia al template de la categoría (estructura H2/H3, schema implícito FAQ).
   ```
2. User message: el draft HTML.
3. Output esperado del modelo:
   ```json
   {
     "score": 87,
     "checklist": [
       {"point": 1, "result": "APPROVED", "note": ""},
       {"point": 9, "result": "FAIL", "note": "Encontré 2 instancias de <b> en lugar de <strong>"},
       ...
     ],
     "priority_fixes": [
       "Reemplazar todos los <b> por <strong>",
       "Acortar el segundo párrafo (88 palabras → max 75)"
     ]
   }
   ```

**Output**: score + priority_fixes (lista de cambios a aplicar en revise).

## Stage 4: revise

**Goal**: aplicar los priority_fixes del critique al draft.

**Loop control**:
- Si score >= 80 en el critique anterior → **NO entrar a revise**, ir directo a fact-check.
- Si score < 80 → entrar a revise.
- Max 2 revise passes. Después del 2do, si score sigue < 80 → dead-letter.

**Inputs**:
- Draft actual
- Lista priority_fixes del critique anterior

**Acción**:
1. System prompt: style guide + template (igual que stage 2).
2. User message:
   ```
   Aquí está un draft del artículo. Aplicá EXACTAMENTE los siguientes cambios prioritarios.
   No reescribas todo: cambios quirúrgicos.
   
   Priority fixes:
   {lista de fixes}
   
   Draft:
   {draft HTML}
   ```
3. Output: nuevo HTML revisado.

**Después de revise**: vuelve a Stage 3 (self-critique) sobre el nuevo draft. Si pasa (≥80), avanza. Si no, decrement attempts y otro revise (max 2).

## Stage 5: fact-check

**Goal**: catch anti-hallucination issues — nombres propios, fechas, datos numéricos.

**Inputs**: draft post-revise.

**Acción**:
1. Extraer del HTML toda mención de:
   - Nombres propios (jugadores, equipos, lugares, competiciones)
   - Fechas y números (años, marcas, scores, dimensiones)
2. Para cada uno, WebSearch para verificar existencia + atribución correcta.
3. Si encuentra discrepancias serias (e.g., "Vinicius Jr es español" cuando es brasileño), generar fixes y aplicar un mini-revise.
4. Si no hay issues, avanza.

**Output**: draft final + fact_check_report con lista de claims verificadas/corregidas.

**Eficiencia**: el fact-check completo puede ser costoso si hay muchos nombres. Para MVP, limitar a max 10 verifications. Issues serios encontrados se loggean para review humano.

## Stage 6: image

Ver `image.md`. Resumen: `scripts/imagen/generate-images.py` invoca `gpt-image-2` via LiteLLM iGaming, produce `hero.jpg` (1536×1024 para Rules/Tactics, 1024×1024 para Origins). El strip de metadata C2PA / JUMBF / OpenAI corre automáticamente vía `scripts/imagen/strip-ai-metadata.py` (fixed 19/5: el script ahora respeta la extensión del archivo y normaliza format mismatches incluso cuando no hay markers).

## Stage 7: SEO enrichment (FAQPage JSON-LD)

**Goal**: cuando el artículo cierra con una sección "Preguntas frecuentes", emitir un `FAQPage` JSON-LD para Google's FAQ rich results.

**Why** después del fact-check y no antes: el contenido de las preguntas tiene que estar verificado. El helper es deterministico y no llama al modelo, así que no introduce variación.

**Acción**:

```python
from scripts.seo_jsonld import inject_faq_jsonld
final_html = inject_faq_jsonld(final_html)   # idempotente
```

El helper:
- Busca `<h2>` con texto "preguntas frecuentes" / "faq" / "preguntas y respuestas" (case-insensitive).
- Extrae los pares `<h3>pregunta</h3><p>respuesta</p>` del bloque siguiente, hasta el próximo `<h2>` o el final del body.
- Strip de tags HTML en el text antes de emitir el JSON-LD (Google parsea plain text mejor).
- Anexa `<script type="application/ld+json">` con el `FAQPage` al final del body.
- Si no hay FAQ section o ya hay un `FAQPage` previo, no toca el HTML.

Note: el theme Relevo ya emite `NewsArticle` + `BreadcrumbList` automáticamente para cada post. El `FAQPage` es el tercer schema, opt-in, solo si hay material.

## Stage 8: publish (WordPress + Asana + inventory advance)

**Goal**: subir el hero al media library, crear el post como `pending`, attach featured image, normalizar el author, notificar al reviewer en Asana, advance la row del inventory.

Está descrito paso a paso en `cmd_new.md` (sección "Paso 7"). Resumen de los helpers usados:

| Sub-paso | Helper | Lo que hace |
|---|---|---|
| 7.a — upload hero | `scripts/wp_upload.py` | Habla MCP por stdio al `ig-mcp-proxy`. Sube el archivo via `media.upload` y devuelve el `attachment_id`. Resuelve el cuello de botella del base64 inline en context. |
| 7.b — SEO enrich | `scripts/seo_jsonld.py` | Inyecta FAQPage JSON-LD (ver Stage 7). |
| 7.c — create post | `mcp__igaming-wp__posts_create` | post como `pending`, featured image attached, taxonomies.category, post_excerpt = meta description (el theme lo lee para og:description). |
| 7.d — fix author | `mcp__igaming-wp__posts_update` | El adapter ignora `post_author` en create y guarda admin (user 1). `posts_update` post-create SÍ lo persiste. Llamada obligatoria. |
| 7.e — verify | `mcp__igaming-wp__posts_get` | Read-back opcional. |
| 7.f — Asana notify | `scripts/asana_notify.py` | Crea subtask en Asana con el quality journey, link al preview, run_id. Asignee = próximo reviewer del pool (Joel hoy, round-robin cuando crezca). |
| 7.g — advance inventory | `sharepoint_msal.update_table_row` | `status` → `review`, `quality_score`, `article_path`, `wp_url`, `asana_subtask_gid`, `completed_at`. |

## Composición final

Una vez completadas todas las etapas:

```python
final_artifact = {
    "run_id": run_id,
    "row_id": row_id,
    "html_body": final_html,
    "hero_image_path": "/tmp/relevo-{run_id}/hero.jpg",
    "inline_images": [...],  # paths si hay
    "metadata": {
        "seo_title": row["seo_title"],
        "meta_description": row["meta_description"],
        "url_slug": row["url_slug"],
        "hub_slug": row["hub_slug"],
        "categoria": row["categoria"],
    },
    "quality_score": final_score,
    "stages_completed": [...],
    "duration_total_seconds": ...,
}
```

Este artifact se guarda en `/tmp/relevo-{run_id}/article_final.json` antes de la subida a WP.

## Defensiveness

- Si algún stage falla irreversiblemente (después de retries), el row se marca `dead_letter` y se crea Asana subtask con el error.
- El artifact parcial siempre se guarda en `/tmp/relevo-{run_id}/` para forensics.
- Logs JSONL conservan toda la secuencia de stages_started / stages_completed con duraciones.
