# ADR-012: Style guide v1.2.2 + Coaches template v1.1 — Feedback Joel ronda 4 sobre smoke Simeone

**Status**: Approved
**Fecha**: 2026-05-27
**Supersedes**: `prompts/style_guide.md` v1.2.1 + `prompts/coaches_template.md` v1.0
**Drivers**: Joel Sierra (feedback ronda 4) · Juan Pablo Waimann (JPW, decisión patch+regen)
**Relacionado**: ADR-008 (style guide v1.1.3), ADR-010 (style guide v1.2 binary tests), ADR-011 (coaches template v1.0)

---

## 1. Contexto

El 27/05/2026 Joel Sierra publicó su revisión del **smoke Coaches Simeone** (TEMPLATE-04 v1.0 + style guide v1.2.1, post WP 30240). El smoke era la primera pieza Coaches productiva — la decisión sobre escalar a batch de 19 entrenadores adicionales colgaba de su verdict.

**Asana task**: `1215146649288358` ([Editorial QA] Coaches smoke — Diego Simeone).
**Verdict**: ⚠️ Needs change.

Métricas self-asignadas por el pipeline: score 92/100, 26/26 self-check pass, cobertura bolds 23/23 (100%), 27/27 facts verificados, 0 párrafos >60 palabras. El pipeline cumplía todos los gates v1.2.1. Joel detectó **patrones cualitativos que el self-check no captura**.

### 1.1 Patrones del feedback Joel agrupados

| Patrón | Naturaleza | Aplica a |
|---|---|---|
| Hitos resueltos solo con marcador + goleador | Falta capa periodística + impacto cultural | Universal (Coaches + Origins) |
| Lexicón cultural ausente (Cholo, colchoneros, rojiblancos, Cerezo, Gil Marín) | Vocabulario del universo del sujeto | Específico Coaches/Origins biográficos |
| Sin conexión con el presente (semis vs Arsenal, Julián Álvarez, salario) | Pieza congelada en el último título | Específico Coaches activos |
| Conflicto público desaprovechado (cholismo defensivo, sueldo, ciclo) | Debate público integrado | Específico Coaches polémicos |
| Categoría histórica imprecisa ("Tercera División" en 1989 = era Segunda B) | Nombre de época | Universal |
| Pasajes técnicos como data dump ("En el San Siro", "El cholismo es el sello de su era") | Frases sin verbo o tautológicas | Universal |
| Negritas siguen decorativas/fragmentadas | Test del valor scan v1.2 no fue suficiente | Universal (refuerzo, no patch) |
| Cierre vuelve al tono-manifiesto | Reglas v1.2 §25 no captaron | Universal (refuerzo, no patch) |

### 1.2 Decisión de scope

- **Patrones universales** (aplican a piezas con sujeto biográfico, no solo coaches) → `style_guide.md` v1.2.2.
- **Patrones específicos coaches** (lexicón del club, conexión presente, debate público) → `coaches_template.md` v1.1.
- **Patrones de bolds y cierre** (Joel los marcó pero ya estaban en v1.2) → NO se introducen patches nuevos. Se aplican con dureza en el self-check existente. La falla fue de ejecución, no de regla.

JPW (27/05 PM): *"actualiza style guide con los comentarios de Joel sobre el artículo de Simeone."* La decisión operativa es **patch + regen** (no per-article fix), consistente con `feedback_regenerate_batch_methodology`.

---

## 2. Decisión

### 2.1 Style guide v1.2.1 → v1.2.2

Tres patches universales:

**Patch 1 — Hitos clave con 3 elementos obligatorios (nueva sección §Hitos clave en piezas biográficas/palmarés)**

En piezas Coaches y Origins, cada hito histórico relevante desarrolla:

1. **Qué pasó** — resultado, score, rival, goleador (capa de datos).
2. **Por qué fue importante en el contexto** — qué cambió, qué dato concreto distingue el hito (capa periodística).
3. **Impacto cultural/emocional reconocible** — cómo se vivió en el universo del club, qué dejó (capa Relevo).

Test pasa/falla: si más de un hito clave tiene solo capa 1, el artículo vuelve a draft.

Ejemplo canónico: Lisboa 2014 no puede ser "Atleti perdió 1-4 con gol de Ramos". Debe incluir el "a 90 segundos del título", el "primer Champions a un paso", y el "trauma fundacional del cholismo".

**Patch 2 — Categorías históricas con nombre de época (extensión §Listas verificables)**

Nueva tabla de categorías españolas históricas:

- 1929–1977: 1ª, 2ª, 3ª División, Regional.
- 1977–2021: 1ª, 2ª, **Segunda División B** (3er nivel), Tercera División (4to nivel).
- 2021–presente: LaLiga, LaLiga Hypermotion, **Primera Federación** (3er nivel), Segunda Federación, Tercera Federación.

Política: usar el nombre vigente en el año del hecho. WebSearch obligatorio ante duda. Ejemplo: el Albacete de 1989 = Segunda B, no Tercera División (Joel marcó esto en Simeone).

**Patch 3 — Pasajes técnicos sin data dump (extensión §Antipatrones)**

Nueva subsección. Frases técnicas/transición no pueden quedar como fragmentos pegados:

- ❌ "En el San Siro." (localización sin verbo)
- ❌ "El cholismo es el sello de su era." (oración tautológica)
- ❌ "El Atleti ganó 3-0 con doblete de Radamel Falcao y tanto de Diego Ribas." (data dump con sintaxis pegada + segunda mención nombre completo + "tanto de" sin artículo)
- ✅ "El Atleti se impuso 3-0 en San Siro con doblete de Falcao y un tanto de Diego Ribas." (oración periodística completa)

Test: aislar la frase del párrafo. Si suena a pie de foto o bullet de data en lugar de a crónica deportiva → reescribir.

**Self-check**: 26 → 29 puntos. Nuevos puntos 27, 28, 29 con los patches.

### 2.2 Coaches template v1.0 → v1.1

Tres componentes obligatorios nuevos en §Componentes obligatorios (puntos 8, 9, 10):

**Componente 8 — Lexicón cultural del club obligatorio**

Antes de redactar, el modelo inventaría:

- **Mote del entrenador** (Cholo, Pep, Mou, Carletto…) con primera mención por nombre + apellido y al menos una aparición del mote como forma natural en el cuerpo.
- **Apodo del aficionado** (colchoneros, culés, blaugranas, merengues…) con 1-2 apariciones donde encaja.
- **Dirigentes históricos clave** cuando aplica al periodo (Cerezo + Gil Marín en Atleti, Florentino, Laporta…) con 1 mención si protagonizaron el ciclo.
- **Símbolos de la institución** (estadio, grada organizada, lema del club…) con 1-2 apariciones naturales.

Test pasa/falla: listar las menciones. Si falta el mínimo (mote + apodo + 1 dirigente cuando aplica + 1 símbolo), draft.

**Componente 9 — Conexión con el presente obligatoria (entrenadores en activo)**

Al menos un párrafo del cuerpo conecta con los últimos 12 meses:

- Última temporada deportiva (posición, eliminación, racha).
- Plantilla actual relevante (fichajes/ventas, capitanes, lesionados).
- Renovación/rumor de salida.
- Crítica/debate reciente sobre el proyecto.

Sin esto, la pieza se lee como obituario en vida. Test: ¿hay datos del último año en al menos un párrafo? Si NO, draft.

**Componente 10 — Debate público integrado (figuras polémicas)**

Si la figura tiene debate público vivo y documentado (cholismo defensivo, anti-fútbol, petrodólares, ciclo cuestionado), al menos un párrafo lo integra con framing periodístico ("X parte del debate público sobre Y es Z") **sin editorializar** ("Simeone es defensivo y eso está mal" → fuera).

Test: ¿la pieza menciona al menos uno de los debates consolidados de la figura, con framing presentativo? Si la figura es polémica y la pieza no lo integra → draft.

### 2.3 Reglas duras (sumas)

A `coaches_template.md` §Reglas duras se agregan:

- **NO** publicar sin lexicón cultural mínimo (v1.1).
- **NO** publicar sin párrafo de conexión con el presente cuando el entrenador sigue activo (v1.1).
- **NO** publicar sin integrar el debate público vivo cuando es polémica (v1.1).
- **NO** resolver hitos clave solo con marcador y goleador (v1.2.2 §27).
- **SÍ** inventariar el lexicón cultural antes de redactar (motes, apodos, dirigentes, símbolos).

---

## 3. Consecuencias

### 3.1 Lo que cambia

- **Smoke Simeone re-genera** bajo v1.2.2 + coaches v1.1. Mismo sujeto, mismos hechos, redacción que incorpora los 3 patches universales + 3 patches específicos de coaches. Asana subtask `1215146649288358` retomable con la nueva regen.
- **Batch coaches 19 NO se lanza** hasta que el re-smoke pase Joel.
- **Patches universales (style guide v1.2.2)** se aplican a futuros artículos de Origins (también biográficos) sin patch separado al `origins_template.md`. El self-check ya cubre los 29 puntos para cualquier categoría.
- **Self-check sube a 29 puntos** (era 26). Pipeline `relevo_critique` debe actualizarse para reportar los 3 nuevos puntos.

### 3.2 Riesgos / lo que NO se patchea

- **Negritas decorativas** siguieron apareciendo en el smoke Simeone. v1.2.2 NO introduce un patch nuevo de bolds — el test del valor scan v1.2 sigue vigente y debe ejecutarse con dureza durante la regen. Si el re-smoke vuelve a fallar en bolds, se reabre la regla en v1.2.3.
- **Cierre tono-manifiesto** también reapareció. Mismo razonamiento: regla existe (v1.2 §3), el self-check obliga, la ejecución fue laxa. Se aplica con dureza.
- **Imprecisiones factuales menores** (Albacete = Segunda B, no 3ra División) están cubiertas por patch 2 + WebSearch obligatorio en categorías históricas. Si reaparecen tras v1.2.2, ampliar tabla histórica.

### 3.3 Métricas de éxito

- Re-smoke Simeone bajo v1.2.2 + coaches v1.1 → Joel verdict ✅ sign-off (o ⚠️ with notes menor de ajuste fino, no estructural).
- Si pasa: lanzar batch 19 coaches bajo mismo setup.
- Si no pasa por los patrones nuevos: patch v1.2.3 con la falla específica. Si no pasa por los viejos (bolds + cierre): NO se patchea regla; se ajusta ejecución del subagente (revisar system prompt del paso de redacción).

---

## 4. Plan de ejecución

1. Commit `style_guide.md` v1.2.2 + `coaches_template.md` v1.1 + este ADR a GitHub (source of truth operativa al 27/05).
2. Re-generar smoke Simeone bajo el setup nuevo (mismo sujeto, mismos hechos, redacción nueva).
3. Self-check 29 puntos antes de subir a WP.
4. Actualizar Asana `1215146649288358` con el nuevo draft + handoff a Joel.
5. Esperar verdict ronda 5. Si ✅ → batch 19 coaches. Si ⚠️ → ver §3.2.

---

## 5. Anexos

- **Feedback completo de Joel**: comentario en Asana task `1215146649288358` (27/05/2026, transcripción en `runs/smoke-coaches-simeone-1779825812/joel_feedback_round4.md` si se preserva).
- **Style guide v1.2.2**: `prompts/style_guide.md` (29 puntos self-check).
- **Coaches template v1.1**: `prompts/coaches_template.md` (10 componentes obligatorios).
- **Memoria operativa**: ver `project_relevo_v1_2_2_patch_27may` (snapshot del patch).
