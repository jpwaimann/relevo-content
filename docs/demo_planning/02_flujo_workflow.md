# Flujo gráfico del workflow `/relevo` (borrador)

> Estado: borrador armado por Claude el 22/5 EOD. Versión ASCII first — si querés versión visual (Figma / Excalidraw / Mermaid) la armamos el lunes.

## Vista 1: Pipeline `/relevo new` end-to-end con HITL gates

```
                          ┌────────────────────────────────────┐
                          │  👤 EDITOR (content writer)        │
                          │  Abre Claude Code · invoca         │
                          │     /relevo new                    │
                          └─────────────────┬──────────────────┘
                                            │
                                            ▼
              ┌──────────────────────────────────────────────────────┐
              │  🤚 HITL #1 — Editor define el job (AskUserQuestion) │
              │     ¿Qué deporte? ¿Categoría? ¿Topic? ¿Notas?         │
              └─────────────────┬────────────────────────────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────────────────────┐
              │  🤖 AUTO — Sweep stuck rows + lock topic + run_id   │
              │     (housekeeping, ~5 seg)                          │
              └─────────────────┬───────────────────────────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────────────────────┐
              │  🤖 STAGE 1: RESEARCH                               │
              │     LLM lee reglamento PDF + WebSearch              │
              │     ~30 seg · output: contexto + facts ancla        │
              └─────────────────┬───────────────────────────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────────────────────┐
              │  🤖 STAGE 2: DRAFT                                  │
              │     LLM redacta artículo bajo style guide v1.1.3    │
              │     ~2 min · output: HTML semántico ~2000 palabras  │
              └─────────────────┬───────────────────────────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────────────────────┐
              │  🤖 STAGE 3: SELF-CRITIQUE → SCORE                  │
              │     LLM se evalúa contra el self-check de 15 puntos │
              │     ~30 seg · output: score 0–100 + lista issues    │
              └─────────────────┬───────────────────────────────────┘
                                │
                                ▼
                       ┌────────────────────┐
                       │ score < 80?        │
                       └────┬───────────┬───┘
                            │ sí         │ no
                            ▼            │
              ┌─────────────────────┐   │
              │ 🤖 STAGE 4: REVISE  │   │
              │   max 2 iters       │   │
              │   ~1 min/iter       │   │
              └─────┬───────────────┘   │
                    │                   │
                    └──────────┬────────┘
                               │
                               ▼
              ┌─────────────────────────────────────────────────────┐
              │  🤖 STAGE 5: FACT-CHECK                             │
              │     LLM revisa contra reglamento extracts + 3       │
              │     búsquedas WebSearch · catch erratas factuales   │
              │     ~1 min · output: HTML corregido                 │
              └─────────────────┬───────────────────────────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────────────────────┐
              │  🤖 STAGE 6: IMAGE GENERATION                       │
              │     gpt-image-2 vía LiteLLM · ~25 seg               │
              │     hero 1024×683 (3:2) — pasará a 16:9 en Phase 4  │
              └─────────────────┬───────────────────────────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────────────────────┐
              │  🤚 HITL #2 (PENDIENTE — ADR-007 / EDIT-14)         │
              │     Gate-by-text Asana 2-step para aprobar hero     │
              │     NO implementado todavía · hoy: pipeline avanza  │
              │     sin pausa (JPW aprueba in-line por ahora)       │
              └─────────────────┬───────────────────────────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────────────────────┐
              │  🤖 STAGE 7: WP UPLOAD                              │
              │     a) sube hero como media · b) inyecta FAQPage    │
              │     JSON-LD · c) crea post (status=pending) ·       │
              │     d) fix post_author · e) read-back               │
              │     ~10 seg · output: post_id WP                    │
              └─────────────────┬───────────────────────────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────────────────────┐
              │  🤖 STAGE 8: INVENTORY UPDATE                       │
              │     Excel row → status=review · wp_post_id linked    │
              │     ~3 seg                                          │
              └─────────────────┬───────────────────────────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────────────────────┐
              │  🤖 STAGE 9: ASANA SUBTASK CREATED                  │
              │     asana_notify.py crea subtask con quality        │
              │     journey + link al WP draft · asigna al pool     │
              │     ~5 seg                                          │
              └─────────────────┬───────────────────────────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────────────────────┐
              │  🤚 HITL #3 — REVIEWER EDITORIAL (gate de calidad)  │
              │     Joel (o pool) recibe subtask Asana              │
              │     Lee el draft en WP · approve / request changes  │
              │     SLA: ~30 min por artículo                       │
              └─────────────────┬───────────────────────────────────┘
                                │
                                ▼
                       ┌────────────────────┐
                       │ ¿aprobado?         │
                       └────┬───────────┬───┘
                            │ sí         │ no
                            ▼            ▼
              ┌─────────────────┐  ┌──────────────────────────────┐
              │ 👤 HITL #4      │  │ 🤚 Reviewer escribe feedback │
              │   PUBLISH       │  │   en Asana · JPW decide:     │
              │   Reviewer le   │  │   re-run con notas / drop /  │
              │   pone Publish  │  │   ediciones inline en WP     │
              │   en WP         │  └──────────────────────────────┘
              └─────────────────┘
```

## Vista 2: Quién hace qué (humano vs AI)

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  HUMANO (👤 + 🤚)                  AI (🤖)                      │
│  ────────────────────              ───────────────────────       │
│                                                                  │
│  Define el job                    Lee reglamento                 │
│  (sport, cat, topic)              Investiga (WebSearch)          │
│                                   Redacta el draft               │
│  Aprueba imagen hero              Se autoevalúa                  │
│  (Phase 4 — gate-by-text)         Revisa si score < 80           │
│                                   Hace fact-check                │
│  REVIEW EDITORIAL                 Genera imagen                  │
│  (Joel / pool)                    Sube a WP                      │
│  → approve / request              Crea subtask Asana             │
│                                                                  │
│  Publica desde WP admin                                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘

   🤚 = HITL (gate humano del pipeline)
   👤 = humano operando fuera del pipeline (editor define / reviewer publica)
   🤖 = stage automático (LLM + tools)
```

## Vista 3: HITL gates resumidos (la "vida del artículo")

```
   ┌────────────────────────────────────────────────────────────┐
   │ Gate │ Quién │ Cuándo │ Decisión │ Tool                    │
   ├──────┼───────┼────────┼──────────┼─────────────────────────┤
   │ #1   │ Editor│ Inicio │ ¿Qué tema?│ AskUserQuestion en CC  │
   │ #2 ⏳│ Editor│ Stage 6│ ¿Imagen OK?│ Asana subtask (Phase 4)│
   │ #3   │ Joel  │ Stage 9│ ¿Calidad? │ Asana subtask + WP draft│
   │ #4   │ Joel  │ Final  │ ¿Publicar?│ WP admin (botón Publish)│
   └──────┴───────┴────────┴──────────┴─────────────────────────┘

   ⏳ = no implementado todavía (Phase 4)
```

## Lo que falta para la versión visual

Si para el lunes querés esto en Figma / Excalidraw / Mermaid:
- El ASCII de arriba se traduce 1:1 a un diagrama horizontal con bloques verdes (AI) y bloques amarillos (humano)
- Recomendaría **Excalidraw** porque es gratis, exporta a PNG/SVG, y a la audiencia editorial no le importa que sea polished — necesita ser claro
- Si JPW no tiene tiempo, ASCII funciona en una slide y se entiende al primer vistazo

**Mockup visual sugerido (textual, no quiero anticiparme a Figma):**
- Top de la slide: línea de tiempo horizontal de izquierda a derecha (Editor → Pipeline → Reviewer → Publish)
- Bloques de stages en cajas, color-coded
- Tres rombos amarillos (👤 humano) intercalados en posiciones #1, #3, #4
- Un rombo ⏳ amarillo en posición #2 con leyenda "Phase 4"
- Stats en el footer: "tiempo total: ~10 min cómputo + ~30 min QA humana"
