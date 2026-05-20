# ADR-002: Style guide v1.1 — Incorporación del feedback editorial Joel batch QA #1

**Status**: Approved
**Fecha**: 2026-05-20
**Supersedes**: `prompts/style_guide.md` v1.0 (14/05/2026)
**Drivers**: Joel Sierra (review editorial), Pablo Waimann (project lead)

---

## 1. Contexto

El 15/05/2026 el pipeline `/relevo new` produjo un batch de 10 artículos en paralelo (puntuación media 90.3 sobre 100 según el critic interno). Entre el 18/05 y el 20/05, Joel Sierra, único reviewer editorial del proyecto Relevo.com tras la salida de Tiziana, realizó el análisis sistemático de los 11 artículos del batch (los 10 originales más el smoke test del 14/05) y entregó el feedback en la task de Asana `1214815492685881`, con 12 subtasks y 23 adjuntos (11 `.docx` de análisis individual + 11 `.pdf` con los artículos originales + 1 `.docx` con el análisis global).

El feedback no marca defectos puntuales: identifica **12 defectos sistemáticos** que se repiten en los 11 artículos **independientemente del deporte y del tipo de pieza** (Rules / Tactics / Origins). En palabras textuales de Joel: *"Los problemas ya no están en la calidad técnica básica. Los problemas están en la capa editorial, la profundidad contextual, la actualización, la cultura deportiva, el tono y la capacidad de construir textos visuales y reconocibles para el aficionado real."*

El detalle completo del feedback está documentado en `prompts/joel-feedback-v1.md`.

---

## 2. Decisión

Bumpear `prompts/style_guide.md` a **v1.1** incorporando 11 de los 12 puntos del feedback como reglas de estilo transversales, dejando los 12 puntos para que los absorban los templates de categoría (Rules, Tactics, Origins) en una fase posterior. La parte v1.1 cubre lo que es común a las tres categorías; los 3 templates necesitan una revisión separada (ADR-003 cuando se aborde).

### 2.1 Cambios incorporados al style guide

| # | Regla nueva | Sustituye / amplía |
|---|---|---|
| 1 | Entradilla obligatoria 30-35 palabras entre H1 y primer H2 | Sección "Apertura" — la apertura previa pasa a aplicarse al **cuerpo** post-entradilla |
| 2 | Primer H2 corto y periodístico (no metadescription) | Nueva sub-sección |
| 3 | Ritmo: techo de párrafos baja de 75 a **60 palabras** | Sección "Ritmo y párrafos" |
| 4 | Variedad sintáctica obligatoria — ban de bisagras repetidas (`no solo… sino también`, `más que…`, `eso explica…`) | Nueva sub-sección dentro de "Ritmo" |
| 5 | Profundidad de **3 capas** para cada concepto importante (qué es / cómo funciona / por qué importa hoy) | Nueva sección "Profundidad" |
| 6 | **2-5 ejemplos icónicos por pieza**, desarrollados, no mencionados al pasar; catálogo de referencias canónicas por deporte | Sección "Contexto y visualidad" reescrita |
| 7 | Separación explícita **info evergreen vs info viva**; toda info viva exige WebSearch antes de redactar | Nueva sección |
| 8 | Tecnicismos minoritarios contextualizados la primera vez (fadeaway, eurostep, parc fermé, ERS, etc.) | Sección "Léxico" — tabla nueva |
| 9 | Antipatrones de épica prefabricada con ejemplos concretos del v1.0 (frases que Joel marcó) | Sección "Antipatrones" ampliada |
| 10 | FAQs de **curiosidad real**, no obvias ni del tipo "qué nacionalidad tiene" | Nueva sección dedicada |
| 11 | **Negritas reinterpretadas** — frase/idea de 5-7 palabras consecutivas, no palabras sueltas dispersas; una negrita por párrafo (excepcionalmente dos) | Sección "Negritas" **reescrita completa** |
| 12 | "Valor Relevo" como pregunta de control obligatoria | Promovido de implícito a **regla no negociable nº 6** |

### 2.2 Self-check ampliado de 15 a 20 puntos

El checklist obligatorio que el modelo recorre antes de devolver el artículo pasa de 15 a 20 puntos, incorporando entradilla, primer H2, variedad sintáctica, tres capas, info viva verificada, FAQs no genéricas, valor Relevo, y la nueva interpretación de negritas.

### 2.3 Clarificación crítica — negritas

El v1.0 decía *"~6-7 palabras destacadas por párrafo"*. Joel marcó este patrón como problema en los 11 artículos: las negritas resaltaban palabras sueltas y dispersas. Pablo clarificó el 20/05 la **interpretación correcta**: las 5-7 palabras deben ir **consecutivas, formando una frase o idea legible al aislarla del resto del texto**. Un lector escaneando el artículo debe poder capturar el mensaje del párrafo solo leyendo las negritas.

- ✅ "El Barça **completó una segunda parte mucho más agresiva** en presión alta."
- ❌ "El **Barça** completó una **segunda parte** mucho más **agresiva** en **presión alta**."

Este cambio es el **delta más grande** entre v1.0 y v1.1, y el más fácil de revertir si genera regresiones en la próxima ronda de QA.

---

## 3. Alternativas consideradas

### 3.1 No tocar el style guide y resolver todo en los templates

**Rechazada.** 11 de los 12 problemas son **transversales** a las tres categorías. Centralizar las reglas en el style guide evita divergencia entre templates y mantiene una sola fuente de verdad editorial.

### 3.2 Hacer un cambio mínimo y esperar a un batch QA #2 antes de v1.1

**Rechazada.** El batch QA #1 ya identifica defectos sistemáticos con muestra de 11 artículos. Esperar otro batch sin aplicar los aprendizajes alargaría el ciclo de mejora y comprometería el deadline del 22/05 (Phase 4 close) y 01/06 (public launch).

### 3.3 Mantener "~6-7 palabras destacadas por párrafo" sin reinterpretar negritas

**Rechazada.** El defecto de negritas-decoración aparece en los 11 artículos. La interpretación de Pablo (5-7 palabras como frase) resuelve el problema sin cambiar la frecuencia ni la cantidad de negritas, solo su **agrupación**. Es un cambio compatible con la métrica cuantitativa.

---

## 4. Impacto y consecuencias

### 4.1 Impacto positivo esperado

- Reducción de los 12 defectos sistemáticos en futuros runs.
- Output más cercano al estándar editorial de Relevo.com (cultura deportiva, escenas reconocibles, profundidad táctica).
- Self-check más estricto → menos artículos pasan el critic interno con problemas que Joel acabará marcando.

### 4.2 Costo

- **Templates de categoría desfasados**: Rules, Tactics y Origins comparten lógica estructural. El feedback #11 ("diferenciar el tipo de pieza") implica que necesitan reescribirse para divergir más. **Pendiente — abordado en ADR-003 cuando se haga.**
- **11 artículos del batch QA #1 quedan obsoletos**: deben regenerarse con prompts v1.1+ antes de cualquier publicación. **Pendiente — capturado en backlog como `EDIT-04`.**
- **Riesgo de regresión en negritas**: la nueva interpretación es más exigente que la anterior. Posible falsos positivos en el self-check del modelo. Se evaluará en el smoke test post-v1.1.

### 4.3 Validación

- **Smoke test multi-categoría** post-v1.1: un artículo de cada tipo (Rules / Tactics / Origins) generado con el style guide nuevo. Comparar contra el batch original (mismo topic) para verificar mejora.
- **Batch QA #2** con Joel después de regenerar los 11 artículos del batch #1 + algunos artículos nuevos. Validación end-to-end del v1.1.

---

## 5. Decisiones derivadas

- **ADR-003** (pendiente): rework de los 3 templates (Rules / Tactics / Origins) para absorber los problemas #2, #3, #7, #8, #11 del feedback con énfasis específico por categoría.
- **Backlog `EDIT-02`** se cierra con este commit.
- **Backlog `EDIT-04`** (nuevo): re-run de los 11 artículos del batch QA #1 con prompts v1.1+.
- **Backlog `EDIT-05`** (nuevo): rework de los 3 templates (Rules / Tactics / Origins) según ADR-003.
- **Backlog `EDIT-06`** (nuevo): batch QA #2 con Joel post re-runs para validar v1.1 end-to-end.

---

## 6. Referencias

- `prompts/style_guide.md` v1.1 — la guía aplicada.
- `prompts/joel-feedback-v1.md` — el feedback consolidado.
- Asana task `1214815492685881` — feedback original con los 23 adjuntos.
- ADR-001 — decisión arquitectónica que estableció `prompts/` como single source of truth editorial.
