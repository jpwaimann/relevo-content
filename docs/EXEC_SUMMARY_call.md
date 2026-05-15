# Relevo Content Workflow — Resumen ejecutivo
**Para call con main content writer · 2026-05-15**

## Qué cambia

Pasamos de un workflow n8n (v17) a un **skill de Claude Code que corre dentro de la sesión del editor**. Cero servidor propio, cero infra que operar. El editor escribe `/relevo new` en su CLI y obtiene un artículo listo para review.

## Cómo funciona en 30 segundos

1. **El editor abre Claude Code** en cualquier directorio del host AWS compartido y escribe `/relevo new`.
2. El skill lee el **Inventory** (SharePoint Excel `Inventory final version.xlsx`; mock CSV local mientras la API esté en setup), elige el siguiente topic con `status=pending`, y arranca el **pipeline multi-pass**:
   - `research` → recolecta fuentes verificables (reglamentos PDF en SharePoint + dominios autoritativos).
   - `draft` → genera el artículo aplicando el template de la categoría (Rules / Tactics / Origins) + el **style guide Relevo** (voz periodística española, equivalencias léxicas, HTML semántico).
   - `critique pass 1` → un crítico interno evalúa el artículo contra el **checklist de 15 puntos del style guide** y devuelve un score 0-100 + priority fixes.
   - `revise` → aplica los fixes si score < 80 (umbral de gate).
   - `critique pass 2` → re-evalúa. Loop hasta pasar gate o agotar reintentos.
   - `fact-check` → verifica claims contra fuentes.
   - `image` → genera hero image vía LiteLLM (`gpt-image-2`) con C2PA stripping.
   - `publish` → sube el artículo a WordPress como `pending_review` y crea una **subtask en Asana** asignada round-robin al reviewer.
3. **El reviewer humano** abre Asana, ve la subtask con el link al WP draft y el score, revisa, y si aprueba, publica.

## Costos

- **Texto**: $0 marginal — el editor usa su subscripción Claude Code existente.
- **Imágenes**: $0.04-0.16 por imagen × ~1.5 imágenes × 50 art/día = **$60-240 USD/mes**.
- **Infra**: $0 — no hay servidor propio, no hay n8n a operar.
- **APIs (WP/Asana/Graph)**: $0 dentro de cuotas estándar.

Comparado con n8n v17: eliminamos el costo operativo de mantener workflows, debugging de nodos rotos, y la rigidez de prompts hard-coded en un nodo.

## Qué gana el content team

| Antes (n8n v17) | Ahora (CC-native) |
|---|---|
| 1 prompt hard-coded por nodo, cambios = PR + redeploy | Prompts en **markdown versionado** (`prompts/*.md`). Cambio de voz = edit un `.md` + PR |
| 15 deportes cubiertos | **24 deportes** |
| 2 categorías (Rules + Tactics) | **3 categorías** (Rules + Tactics + **Origins** = bios deportistas) |
| Generación lineal sin auto-crítica | **Multi-pass con gate de calidad** (score ≥ 80 antes de pasar a humano) |
| Output: Google Doc draft | Output: WordPress `pending_review` + Asana notif al reviewer |
| Score promedio observado: variable | Smoke test 14/5: **score 95** post-revise pass |

## Cómo el writer va a revisar el output

Cada artículo generado deja en disco un bundle completo:

- `article.html` — el HTML semántico final (lo que va a WP)
- `article_final.json` — metadata: word_count, quality_score, fact-check status, etapas completadas
- `critique_pass1.json` — el primer score 0-100 con breakdown por dimensión (voz, ritmo, anti-hallucination, SEO, etc.) y los priority_fixes que se aplicaron
- `critique_pass2.json` — el score final ya post-revise
- `asana_subtask.json` — la notificación que se mandó al reviewer

El writer puede abrir cada uno y entender exactamente qué hizo el pipeline, qué problemas detectó la auto-crítica, y qué se corrigió. Esto da **trazabilidad total** — algo que n8n v17 no daba.

## Ejemplo real (smoke test 14/5)

**Topic**: "Qué es el fuera de juego en fútbol y cómo funciona"

| Stage | Resultado |
|---|---|
| draft | Generado, 850 palabras |
| critique pass 1 | Score 72 — FAIL en anti-hallucination (caso "Iker Bravo en Bernabéu marzo 2024" no verificable) + longitud insuficiente |
| revise pass 1 | 3 priority fixes aplicados |
| critique pass 2 | Score 95 — PASS. 1592 palabras dentro de target 1500-2000 |
| fact-check | 8 claims verificados, 0 corregidos |
| **Duración total** | **132 segundos** |

URL del bundle público: *(se pega cuando esté listo el hosting)*

## Qué necesita el writer hacer / aprobar

1. **Validar la voz** en el sample del fuera de juego. ¿Suena a Relevo o todavía hay correcciones por hacer en el style guide?
2. **Decidir si los templates por categoría** (`prompts/rules_template.md`, `prompts/tactics_template.md`, `prompts/origins_template.md`) reflejan la estructura editorial deseada.
3. **Definir el proceso de review** en Asana: round-robin, SLA, criterios para devolver vs aprobar.
4. **Aportar los reglamentos faltantes** (Wrestling, Juegos Olímpicos, Juegos Paralímpicos — 3 deportes sin PDF en SharePoint).

## Roadmap próximas 2 semanas

- Hoy: validar voz con writer + generar batch de muestra (5 fútbol + 5 baloncesto + 5 tenis + 5 boxeo + 5 F1 + 1 de cada resto).
- Semana 1: cerrar style guide v1.1 con feedback, conectar Inventory real (SharePoint), validar Graph API auth.
- Semana 2: conectar WP MCP + Asana MCP, primer batch en producción con reviewer humano real.

## Quién mantiene qué

- **Pablo Waimann** — owner técnico, mantiene skill + templates + inventory.
- **Main content writer (la persona de esta call)** — owner del style guide, validador final de voz.
- **Reviewers** (round-robin) — aprobación pre-publicación en Asana.
- **Debanjan (BI)** — Azure/SharePoint/Passbolt setup.

## Riesgos abiertos

- Hosting de previews para review: en curso (puerto 80 vs cloudflared vs alternativas).
- LiteLLM proxy todavía no tiene modelos de imagen — pendiente con Debanjan.
- 3 deportes sin reglamento PDF.
