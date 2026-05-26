# Comando: `/relevo new demo` — modo demostración

## Cuándo se invoca

El editor escribe `/relevo new demo` (o `/relevo demo`) en su sesión CC.

**Propósito**: presentar el pipeline ante una audiencia sin hacerla esperar los ~10 minutos de cómputo real. Simula la mecánica del workflow con outputs realistas, pero termina apuntando a un artículo **pre-aprobado por Joel** que ya existe como draft en WordPress con su foto. La subtask Asana y el log a JSONL se hacen reales para que la audiencia vea la integración funcionando — todos marcados como `[DEMO]` / `mode=demo` para no contaminar inventario ni inbox de reviewers.

**No es** una herramienta de QA. No genera artículo nuevo, no llama LLM, no genera imagen, no escribe al Excel inventory. Solo orquesta una experiencia visual que mapea a un draft existente.

## Catálogo de artículos demo

Los 3 artículos están pre-elegidos del batch 3 (v1.1.3, publicados 21/5). Score 91+, featured image en su sitio, contenido aprobado.

| Categoría | Deporte | Topic | WP post_id | Hero attachment | URL slug |
|---|---|---|---|---|---|
| Rules | Fútbol | Posiciones en el fútbol: funciones y características | 21195 | uploaded | `posiciones-futbol-funciones` |
| Tactics | Fórmula 1 | Cómo funciona la estrategia de boxes en Fórmula 1 | 21210 | uploaded | `estrategia-boxes-f1` |
| Origins | Tenis | Orígenes de Carlos Alcaraz: el murciano que conquistó el tenis | 21204 | uploaded | `origenes-carlos-alcaraz-murciano-tenis` |

URLs preview WP (host beta):
- `https://relevo-com.sub.plus/wp-admin/post.php?post=21195&action=edit`
- `https://relevo-com.sub.plus/?p=21210&preview=true`
- `https://relevo-com.sub.plus/?p=21204&preview=true`

## Flow

### Paso 1: Banner de demo

Imprime al inicio (con un toque dramático para que la audiencia entienda que es demo, no producción real):

```
═══════════════════════════════════════════════════════════════════
 🎬 RELEVO PIPELINE · DEMO MODE
═══════════════════════════════════════════════════════════════════
 Simulación de pipeline E2E con tiempos comprimidos.
 El artículo final ya existe como draft pre-aprobado por Joel.
 Asana subtask + logs son reales (marcados [DEMO]).
═══════════════════════════════════════════════════════════════════
```

### Paso 2: AskUserQuestion al editor — categoría a demostrar

```
Pregunta: "¿Qué tipo de artículo querés demostrar?"
Opciones:
  • Rules — Posiciones en el fútbol (Fútbol)
  • Tactics — Estrategia de boxes en F1 (Fórmula 1)
  • Origins — Orígenes de Carlos Alcaraz (Tenis)
```

Header de la pregunta: "Demo article". Una sola pregunta, no multi-step.

Una vez elegida la opción, cargar el mapping correspondiente del catálogo.

### Paso 3: Crear run_id y arrancar logging

```python
import time, uuid
run_id = f"demo-{categoria}-{int(time.time())}"
demo_started_at = time.time()
```

Loggear `run_started` con un payload extendido que deja claro que es demo:

```python
log("run_started", run_id, level="info",
    deporte=deporte, categoria=categoria, row_id="DEMO",
    payload={
        "editor_input": {"seo_title": seo_title, "mode": "demo"},
        "demo": True,
        "mapped_to_wp_post_id": wp_post_id
    })
```

### Paso 4: Simulación de etapas con outputs realistas

**Importante**: NO ejecutar trabajo real (no LLM, no fact-check, no image gen, no WP upload). Solo imprimir el output que vería el editor en una corrida real y dormir entre stages para dar sensación de proceso.

Usar Bash con un solo heredoc por stage (no llamar tools reales). Sleeps deliberadamente cortos — el objetivo es que la audiencia vea la mecánica en ~45-60 segundos totales.

```bash
echo "🔍 [1/9] Investigación · leyendo reglamento oficial + búsqueda web…"
sleep 4
echo "   ↳ Read: IFAB Laws of the Game 2025/26 (Ley 4 · La equipación · Ley 11 · Fuera de juego)"
echo "   ↳ Search: '4-3-3 vs 4-4-2 transition' · '11 player positions soccer'"
echo "   ✓ Investigación · 32 s · contexto + 6 facts ancla"
echo ""

sleep 1
echo "✍️  [2/9] Redacción · style guide v1.1.3…"
sleep 5
echo "   ↳ ~2 000 palabras · HTML semántico · 8 H2 · FAQ con 6 H3"
echo "   ↳ Bolds coverage target ≥0.80 · entradilla 30-35 palabras"
echo "   ✓ Draft completado · 124 s · 1 989 palabras"
echo ""

sleep 1
echo "🎯 [3/9] Auto-crítica contra self-check de 15 puntos…"
sleep 3
echo "   ↳ Score parcial: 76 / 100 — REVISE_REQUIRED"
echo "   ↳ Issues detectados: 3 párrafos >68 palabras · bolds coverage 0.72 (target 0.80)"
echo ""

sleep 1
echo "📝 [4/9] Revisión auto-correctiva…"
sleep 3
echo "   ↳ Aplicando fixes: shortening párrafos · agregando 5 bolds editorial"
echo "   ↳ Re-evaluación post-revise"
echo "   ✓ Score 91 / 100 — PASS · 15/15 checklist puntos verdes"
echo ""

sleep 1
echo "🔎 [5/9] Fact-check · cruzando claims contra reglamento…"
sleep 4
echo "   ↳ 13 claims concretos verificados · 0 corregidos · 0 ablandados"
echo "   ↳ Anti-hallucination: PASS"
echo "   ✓ Fact-check OK · 56 s"
echo ""

sleep 1
echo "🎨 [6/9] Generación de imagen hero · gpt-image-2 vía LiteLLM…"
sleep 4
echo "   ↳ Prompt: editorial photo · sport-specific · landscape composition"
echo "   ↳ Strip metadata C2PA/JUMBF · downscale para WP"
echo "   ✓ Imagen generada · 27 s · 1 024×683 listo para subir"
echo ""

sleep 1
echo "☁️  [7/9] Subida a WordPress…"
sleep 3
echo "   ↳ Hero image → media library"
echo "   ↳ Schema FAQPage JSON-LD inyectado"
echo "   ↳ Post creado · status=pending · category attached · author=editor"
echo "   ✓ WP post ${WP_POST_ID} · 10 s"
echo ""

sleep 1
echo "📋 [8/9] Inventory update · Excel…"
sleep 2
echo "   ↳ Row → status=review · wp_post_id linked · quality_score=91"
echo "   ✓ Inventory sincronizado · 3 s"
echo ""

sleep 1
echo "🔔 [9/9] Notificación al reviewer · Asana…"
sleep 2
echo "   ↳ Subtask creada · asignada al reviewer pool"
echo "   ↳ Quality journey incluido (research → draft → critique 76 → revise → 91)"
echo "   ↳ Link al WP draft incluido"
echo ""
```

Loggear cada stage al JSONL con `mode=demo`:

```python
# Después de cada bloque de echo + sleep
for stage in ["research", "draft", "critique", "revise", "factcheck", "image"]:
    log(f"{stage}_started", run_id, level="info", deporte=deporte, categoria=categoria, row_id="DEMO", payload={"demo": True})
    log(f"{stage}_completed", run_id, level="info", deporte=deporte, categoria=categoria, row_id="DEMO", payload={"demo": True, "duration_s": stage_durations[stage]})

log("wp_upload_completed", run_id, payload={"demo": True, "wp_post_id": wp_post_id})
log("asana_subtask_created", run_id, payload={"demo": True, "asana_gid": gid})
```

### Paso 5: Crear Asana subtask real con prefijo [DEMO]

Esta llamada es **real**. Crea una subtask con:
- Name: `[DEMO] {categoria} · {deporte}: {seo_title}`
- Parent: la parent task del inbox de Relevo (gid `1214527443265898`)
- Notes: incluye texto explícito "**Este es un artículo de demostración**" + link al WP draft + run_id + quality journey simulado
- Assignee: el mismo editor (Pablo) — NO Joel, para no contaminar su inbox

Ejecutar via Bash con curl directo:

```bash
ASANA_PAT="${ASANA_PAT:?ASANA_PAT no seteado — el demo necesita el token para crear la subtask}"

ASANA_RESPONSE=$(curl -sS -X POST "https://app.asana.com/api/1.0/tasks" \
  -H "Authorization: Bearer ${ASANA_PAT}" \
  -H "Content-Type: application/json" \
  -d @- <<EOF
{
  "data": {
    "name": "[DEMO] ${CATEGORIA_HUMAN} · ${DEPORTE}: ${SEO_TITLE}",
    "parent": "1214527443265898",
    "assignee": "me",
    "notes": "🎬 ARTÍCULO DE DEMOSTRACIÓN — generado por /relevo new demo durante la presentación al equipo.\n\nEste no es un artículo nuevo. Mapea a un draft pre-aprobado por Joel del batch 3 (v1.1.3) para mostrar el flujo end-to-end sin esperar los ~10 minutos de cómputo real.\n\n— Topic: ${SEO_TITLE}\n— Deporte: ${DEPORTE}\n— Categoría: ${CATEGORIA_HUMAN}\n— Score simulado: 91/100\n— WP draft: ${WP_PREVIEW_URL}\n— Run id: ${RUN_ID}\n\nQuality journey (simulado):\n  research → draft (1989 palabras) → critique (76/100) → revise → critique (91/100) → fact-check (13 claims OK) → image (gpt-image-2, 27s) → WP upload\n\nSafe to mark completed inmediatamente — no requiere review real."
  }
}
EOF
)

ASANA_GID=$(echo "$ASANA_RESPONSE" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('data',{}).get('gid',''))")
ASANA_URL="https://app.asana.com/0/0/${ASANA_GID}/f"

echo "   ✓ Asana subtask creada · gid=${ASANA_GID}"
echo "     URL: ${ASANA_URL}"
echo ""
```

Si el POST falla, loggear `asana_subtask_failed` como warn pero **no abortar la demo** — la audiencia debe ver el flow completo. En ese caso mostrar `(simulado — Asana token no disponible)` en el output.

### Paso 6: Marcar run_completed con flag demo

```python
duration_total = time.time() - demo_started_at
log("run_completed", run_id, level="info",
    payload={
        "demo": True,
        "duration_seconds": round(duration_total, 1),
        "final_score": 91,
        "word_count": 1989,
        "mapped_to_wp_post_id": wp_post_id,
        "asana_gid": asana_gid
    })
```

### Paso 7: Summary final al editor

Markdown limpio para mostrar al final, con visual de éxito:

```markdown
═══════════════════════════════════════════════════════════════════
 ✅ DEMO PIPELINE COMPLETADO
═══════════════════════════════════════════════════════════════════

  📰 **Artículo**: {seo_title}
  🏷️  **Categoría**: {categoria_human} · {deporte}
  📊 **Score simulado**: 91 / 100
  ⏱️  **Duración demo**: ~45 segundos (real ~10 min)
  📝 **Word count**: 1 989

  🔗 **WP draft (preview público)**:
     {WP_PREVIEW_URL}

  🔔 **Asana subtask creada**:
     {ASANA_URL}

  📂 **Logs reales en**:
     /home/pablo/relevo-content/logs/runs.jsonl
     (todos los eventos con flag `"demo": true`)

═══════════════════════════════════════════════════════════════════
 Próximo paso para la audiencia: abrir el link de WP y ver el
 artículo final con foto y formato editorial completo.
═══════════════════════════════════════════════════════════════════
```

## Reglas duras

- **Nunca** llamar a LLM real, fact-check real, image gen real, WP upload real, ni Excel inventory en demo mode. Es 100% scripted.
- **Sí** llamar Asana real (subtask `[DEMO]` asignada al editor) y JSONL real (entries con `"demo": true`).
- **Nunca** crear posts WP nuevos. Solo apuntar a los 3 post_id pre-mapeados (21195, 21210, 21204).
- **Nunca** tocar el Excel inventory en demo mode — no marcar topics como in_progress ni review.
- **Siempre** prefijar la subtask Asana con `[DEMO]`.
- **Siempre** asignar la subtask al editor que corre la demo, no al reviewer pool.
- Si falta `ASANA_PAT`, hacer fallback al output simulado pero seguir adelante.

## Mapping de categoría → artículo

Si la AskUserQuestion devuelve:

| Selección del editor | Variables a setear |
|---|---|
| `Rules — Posiciones en el fútbol (Fútbol)` | `categoria=rules`, `categoria_human="Rules"`, `deporte="Fútbol"`, `seo_title="Posiciones en el fútbol: funciones y características"`, `wp_post_id=21195`, `wp_preview_url="https://relevo-com.sub.plus/?p=21195&preview=true"` |
| `Tactics — Estrategia de boxes en F1 (Fórmula 1)` | `categoria=tactics`, `categoria_human="Tactics"`, `deporte="Fórmula 1"`, `seo_title="Cómo funciona la estrategia de boxes en Fórmula 1"`, `wp_post_id=21210`, `wp_preview_url="https://relevo-com.sub.plus/?p=21210&preview=true"` |
| `Origins — Orígenes de Carlos Alcaraz (Tenis)` | `categoria=origins`, `categoria_human="Origins"`, `deporte="Tenis"`, `seo_title="Orígenes de Carlos Alcaraz: el murciano que conquistó el tenis"`, `wp_post_id=21204`, `wp_preview_url="https://relevo-com.sub.plus/?p=21204&preview=true"` |

## Output esperado al user

Tras una corrida exitosa, ya viste arriba el summary final en Paso 7. JPW lo proyecta en pantalla durante la demo, después abre el link de WP para mostrar el artículo real con foto. La audiencia entiende:
1. Que se invocó un comando único (`/relevo new demo`)
2. Que el pipeline pasó por 9 etapas estructuradas
3. Que en menos de 1 minuto el equipo tiene un draft listo para review
4. Que la integración con Asana y los logs son automáticos
