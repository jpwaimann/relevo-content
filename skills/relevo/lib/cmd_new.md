# Comando: `/relevo new`

Genera 1 artículo desde un topic existente del Inventory. Block ~10 min de la sesión CC.

## Pre-requisitos del skill al arrancar este comando

- Leer `data/sports_config.json` (24 deportes, sub_hubs, reglamento_available).
- Leer `prompts/style_guide.md` — siempre va al system prompt.
- Saber qué template cargar según la categoría que elija el editor.

## Flow

### Paso 1: Sweep stuck rows (housekeeping)

Antes de pedir input al editor, hacer el sweep de stuck `in_progress` > 30 min.

```
Pseudocódigo:
  stuck_rows = sweep_stuck_rows(provider)  # ver retry.md
  if stuck_rows:
    Usar AskUserQuestion: "Encontré {N} artículos colgados (>30 min). ¿Qué hacés?"
    Opciones: ["Rescatar (continuar desde última etapa)", "Reset a pending", "Marcar dead-letter", "Ignorar por ahora"]
    Ejecutar la acción elegida.
```

### Paso 2: AskUserQuestion al editor

Pregunta-única o multi-pregunta según UX. **Importante**: usar dropdowns donde aplica (deporte, categoría) — el editor no es técnico.

```
Pregunta 1: "¿Qué deporte?"
  Opciones: los 24 deportes (priorizar los más comunes arriba: Fútbol, Tenis, Baloncesto, MMA, F1, Boxeo)

Pregunta 2: "¿Categoría?"
  Opciones: ["Rules — Reglas y conceptos del deporte", "Tactics — Tácticas, posiciones, técnicas", "Origins — Biografías de jugadores"]

Pregunta 3: "¿Cómo elegís el topic?"
  Opciones: 
    - "Tomar el primer pending del Inventory (auto-pick por priority)"
    - "Elegir un topic específico (te muestro la lista)"
    - "Crear topic nuevo on-the-fly (no se guarda al Inventory, solo se genera)"

Si "elegir específico" → mostrar lista de pending del Inventory para deporte+categoría, AskUserQuestion para elegir.

Pregunta 4 (opcional, solo si on-the-fly): "Dame seo_title + meta_description + url_slug + notas."

Pregunta 5: "¿Alguna nota adicional para este artículo? (e.g., 'enfocate en estilo defensivo', 'incluí ejemplo de la final del Mundial 2024')"
```

### Paso 3: Crear run_id + log run_started

```python
import time, uuid
run_id = f"{categoria}-{int(time.time())}"
log("run_started", run_id, level="info", deporte=deporte, categoria=categoria,
    row_id=row_id, payload={"editor_input": {...}})
```

Crear directorio `/tmp/relevo-{run_id}/`.

### Paso 4: Lock topic (mark in_progress)

Con optimistic locking. Si la row ya está `in_progress` (otro editor la agarró), reintenta con la siguiente.

```python
locked_row = provider.lock_row(row_id)  # mueve status pending → in_progress
log("inventory_locked", run_id, row_id=row_id)
```

### Paso 5-9: Pipeline multi-pass

Ejecutar las etapas descritas en `lib/pipeline.md`:
1. research
2. draft
3. self-critique → score
4. revise (si score < 80, max 2 iteraciones)
5. fact-check
6. image generation (`lib/image.md`)

Cada etapa: `log("{stage}_started")` antes, `log("{stage}_completed")` después.

Si en cualquier etapa después de retries no se puede continuar → marcar `dead_letter`, ir a paso 11.

### Paso 6: Save artifact

```python
import json
artifact = {
    "run_id": run_id,
    "row_id": row_id,
    "html_body": final_html,
    "hero_image_path": "/tmp/relevo-{run_id}/hero.jpg" if generated else None,
    "metadata": {...},
    "quality_score": final_score,
    "stages_completed": [...],
}
with open(f"/tmp/relevo-{run_id}/article_final.json", "w") as f:
    json.dump(artifact, f, indent=2, ensure_ascii=False)

# También guardar el HTML standalone
with open(f"/tmp/relevo-{run_id}/article.html", "w") as f:
    f.write(final_html)

# Y un Markdown para review humana rápida
import html2text  # o conversor manual; usar markdown si no hay lib
with open(f"/tmp/relevo-{run_id}/article.md", "w") as f:
    f.write(html_to_markdown_simple(final_html))
```

### Paso 7: Upload a WordPress

**Mock mode**: ya guardado en `/tmp/relevo-{run_id}/article.html` — no hacer nada más.

**Production mode** — dos sub-pasos: subir hero, después crear el post.

**7.a — Subir la hero image** (via `scripts/wp_upload.py`, no inline desde el agente):

```bash
log("wp_upload_started", run_id)

ATTACHMENT_JSON=$(python3 /home/pablo/relevo-content/scripts/wp_upload.py \
    "/tmp/relevo-${run_id}/hero.jpg" \
    --alt "${seo_title} — imagen principal" \
    --filename "${url_slug}-hero.jpg" \
    --quiet)

ATTACHMENT_ID=$(echo "$ATTACHMENT_JSON" | python3 -c "import json,sys; print(json.load(sys.stdin)['attachment_id'])")
ATTACHMENT_URL=$(echo "$ATTACHMENT_JSON" | python3 -c "import json,sys; print(json.load(sys.stdin)['url'])")

log("media_uploaded", run_id, payload={"attachment_id": ATTACHMENT_ID, "url": ATTACHMENT_URL})
```

Si el upload falla, marcar `image_upload_failed` en el log y continuar **sin** featured image — el reviewer subsana en el WP admin. No abortar el publish del artículo por una falla de imagen.

**7.b — SEO enrichment (FAQPage JSON-LD) antes del create**:

Si el artículo cierra con una sección "Preguntas frecuentes" (lo hacen casi todos los Rules y los Tactics que terminan en FAQ), enriquecer el HTML con un JSON-LD `FAQPage` para que Google genere FAQ rich results. El theme Relevo ya emite `NewsArticle` + `BreadcrumbList` automáticamente; el `FAQPage` es el tercer schema y solo se inyecta cuando hay material.

```python
from scripts.seo_jsonld import inject_faq_jsonld
final_html = inject_faq_jsonld(final_html)   # idempotente: detecta y skipea si ya existe
```

El helper es deterministico: detecta el `<h2>` con texto "preguntas frecuentes"/"faq"/etc., extrae los pares `<h3>`/`<p>` y anexa el `<script type="application/ld+json">` al final del body. Si no hay FAQ, retorna el HTML sin cambios.

**7.c — Crear el post como pending_review**:

```
- Llamar mcp__igaming-wp__posts_create con:
  - post_type: "post"
  - post_status: "pending"           # NUNCA "publish" desde el skill
  - post_title: seo_title
  - post_content: final_html         # HTML post-7.b con FAQPage si aplica
  - post_excerpt: meta_description   # cuerpo corto para listados; el theme lo usa como og:description
  - post_name: url_slug              # WP usa esto como slug
  - featured_image_id: ATTACHMENT_ID # del paso 7.a (omitir si falló)
  - taxonomies: { "category": [hub_slug] }   # hub_slug del Inventory (e.g. "futbol")

- Capturar wp_post_id del response (campo `id`)
log("wp_upload_completed", run_id, payload={"wp_post_id": wp_post_id, "attachment_id": ATTACHMENT_ID})
```

**7.d — Fix obligatorio del `post_author`**:

```
- Llamar mcp__igaming-wp__posts_update con:
  - post_type: "post"
  - id: wp_post_id
  - post_author: <author_user_id>   # WP user ID del editor (Pablo=77)
```

`posts_create` ignora silenciosamente el campo `post_author` y siempre guarda el admin default (user 1). `posts_update` SÍ lo persiste. Confirmado por test del 19/5 (post 16096): create → `post_author=1`, update inmediato → `post_author=77`.

**7.e — (Opcional) Read-back de verificación**:

```
- mcp__igaming-wp__posts_get(post_type="post", id=wp_post_id)
- Verificar: post_status, _thumbnail_id, taxonomies.category, post_author
```

**Otros gotchas conocidos**:
- Si la category no existe, crearla con `wp-terms_create` antes de attach. Las 24 sports de Relevo ya están en el site como categories (Futbol term_id 3329 verified May 19).
- **El theme Relevo no genera `<meta name="description">` tradicional**; el SEO meta se renderiza como `og:description` y lo popula automáticamente del `post_excerpt`. Setear `post_excerpt = meta_description` en step 7.b es suficiente (no hay que setear meta keys custom de Yoast / Rank Math / AIO SEO — ninguno está instalado).
- **El theme YA genera JSON-LD automático** para cada post: `BreadcrumbList` + `NewsArticle` (con headline, description del excerpt, dates, author, publisher, image, articleSection). No hace falta inyectar Article schema manual. Para Rules/Tactics el `@type` queda como `NewsArticle` — si SEO team quiere `Article` o `HowTo`, eso es un cambio en el theme, no en el skill.
- El plugin `ig-schema` está absent en el site; no usar las tools `ig-schema_*` (van a fallar). Si se necesita schema custom (e.g., `FAQPage` para los artículos con sección FAQ), inyectarlo como `<script type="application/ld+json">` al final del HTML en `post_content`.

### Paso 8: Update inventory row → status=review

```python
provider.update_row(row_id, {
    "status": "review",
    "wp_post_id": wp_post_id,  # None en mock
    "quality_score": final_score,
})
```

### Paso 9: Asana subtask para reviewer

**Mock mode**: escribir `/tmp/relevo-{run_id}/asana_subtask.json` con lo que se crearía.

**Production mode**: invocar `scripts/asana_notify.py` con el directorio del run como argumento. El helper lee `article_final.json`, arma el payload (sport · category: seo_title, quality journey, link al preview público, run_id, score) y crea la subtask bajo `Main task - Solicitar artículo Relevo.com` (parent gid `1214527443265898`) asignada al próximo reviewer del pool (MVP: Joel; round-robin cuando crezca).

```bash
ASANA_PAT="<token>" python scripts/asana_notify.py /tmp/relevo-{run_id}/
# stdout: ✓ Created subtask gid=<gid>
#         URL: https://app.asana.com/0/0/<gid>/f/
```

Para dry-run (validar el payload sin tocar Asana): añadir `--dry-run`. Para tests sin contaminar la inbox: `--name-prefix "[TEST] "` y marcar `completed=true` inmediatamente vía API.

```python
log("asana_subtask_created", run_id, payload={"asana_gid": gid, "reviewer": user_id})
```

### Paso 10: log run_completed + show summary al editor

```python
duration = time.time() - start_time
log("run_completed", run_id, level="info", payload={
    "duration_seconds": duration,
    "final_score": final_score,
    "word_count": word_count,
})
```

Mostrar al editor un summary breve:
```
✅ Artículo generado en {duration}s
   Topic: {seo_title}
   Score final: {score}/100
   Files: /tmp/relevo-{run_id}/
     - article.html (HTML semántico para WP)
     - article.md   (Markdown para review rápida)
     - hero.jpg     (imagen 1536x1024)
     - article_final.json (artifact con metadata)
   {Asana subtask creada — link en mock mode: ver asana_subtask.json}
```

### Paso 11: Dead-letter handling

Si el pipeline falla irreversiblemente:

```python
provider.update_row(row_id, {
    "status": "dead_letter",
    "attempts": current_attempts + 1,
})
log("dead_letter", run_id, level="error", payload={
    "stage_failed": stage_name,
    "error": error_message,
    "attempts": current_attempts + 1,
})

# Crear Asana subtask "needs manual attention" con detalles del error
```

Mostrar al editor:
```
❌ Pipeline falló después de retries.
   Stage que falló: {stage}
   Error: {error}
   Artículo marcado como dead_letter, ticket Asana creado.
   Artifact parcial en: /tmp/relevo-{run_id}/ (para forensics)
```

## Reglas de invocación importantes

- **NUNCA** seguir adelante si la lock falla (otra sesión tiene la row): retry con next row o abort.
- **NUNCA** marcar `status=published` desde el skill — eso lo hace el reviewer humano.
- **SIEMPRE** loggear timestamp UTC y mantener idempotencia: re-ejecutar un run_id desde un crash no debe duplicar Asana subtasks ni WP drafts (chequear si ya existe el wp_post_id en la row).
- **NO** llamar fact-check con el reglamento PDF entero como input (cost prohibitivo) — solo extracts relevantes.

## Output esperado al user (formato final)

Después del éxito, mostrar:

```markdown
## ✅ Artículo Relevo generado

**Topic**: {seo_title}
**Categoría**: {categoria} / {tipo}
**Score**: {score}/100
**Duración**: {duration} segundos
**Word count**: {wc}

**Archivos** (en `/tmp/relevo-{run_id}/`):
- `article.html` — HTML semántico listo para WP
- `article.md` — Markdown para review rápida
- `hero.jpg` — Imagen hero 1536x1024 (C2PA stripped)
- `article_final.json` — Metadata estructurada

**Acción siguiente**: el reviewer asignado ({reviewer}) recibe Asana subtask. Si todo OK, publica desde WP.

¿Querés que muestre el HTML aquí en chat o lo dejamos solo en archivo?
```
