# ADR-010: Style guide v1.2 — Refactorización de reglas existentes como tests pasa/falla

**Status**: Approved
**Fecha**: 2026-05-25
**Supersedes**: `prompts/style_guide.md` v1.1.3 (8 patches sobre reglas existentes, sin cambios estructurales del paradigma)
**Drivers**: Juan Pablo Waimann (JPW) + Joel Sierra (feedback ronda 3 sobre batch 3)
**Relacionado**: ADR-006 (style guide v1.1.2), ADR-008 (style guide v1.1.3), ADR-009 (gap visual del header)

---

## 1. Contexto

El 25/05/2026 a las 11:15 UTC Joel Sierra publicó su revisión del batch 3 (9 artículos regenerados bajo v1.1.3, publicados como WP drafts el 21–22/5: posts 21195/21198/21201/21204/21207/21210/21213/21216/21219). El verdict en la Asana task `1215038718722890`:

| Resultado | Cantidad |
|---|---|
| ✅ OK (publicables con edición fina) | 2/9 (art-4, art-8) |
| ⚠️ Needs change | 7/9 (art-1, art-3, art-6, art-7, art-9, art-10, art-11) |

Target previo (ADR-008 §4.1): ≥ 7/9 sign-off. **Resultado real: 2/9** — empeoró respecto al ratio absoluto y se mantuvo respecto al ratio relativo del batch 2 (2/11). **v1.1.3 no movió la aguja en los patrones cualitativos.**

### 1.1 Diagnóstico — métrica vs comportamiento

La métrica cuantitativa de v1.1.3 (`bolds_count / paragraphs_count ≥ 0.80`) **sí se cumplió** en el batch 3 (cobertura 0.947–1.00, todos los artículos pasaron el gate). Pero los mismos patrones cross-batch del batch 2 reaparecieron en el feedback de Joel sobre batch 3:

| Patrón | Recurrencia en batch 3 | Estado de la regla en v1.1.3 |
|---|---|---|
| Negritas sin "sentido estratégico" (decorativas) | 8/9 arts | Regla cuantitativa cumplida; cualitativa no operable |
| Tablas sin formato visual | 4/9 arts | Sin regla explícita en el style guide |
| Cierres "topiqueros" / sin negrita / con H2 "Cierre" | 4/9 arts | Regla existe como principio, no como check |
| Frases grandilocuentes / artificiales | 5/9 arts | Lista de antipatrones del batch 1; sin test operable |
| Párrafos > 60 palabras | 2/9 explícitos (uno de 75 en art-10) | Regla existe; self-check no exigía conteo |
| Formato inconsistente intra-lista | 1/9 explícito (art-7) | Sin regla |
| Errores factuales (palmarés ATP, geografía) | 2/9 graves (art-6 Cincinnati 2023, Pekín 2024; art-9 Halle/Renania) | Antialucinación general; sin listas verificables |
| Cursivas faltantes en términos extranjeros | 1/9 (art-8 pit-wall, paddock) | Inventario existe; sin check obligado |

### 1.2 Hipótesis de root cause

Las reglas de v1.0 → v1.1.3 están escritas como **principios** ("evita lo grandilocuente", "verifica datos vivos") en lugar de **tests binarios pasa/falla**. El único caso donde un test binario fue introducido (cobertura ≥80% de negritas, v1.1.3) **sí se cumplió mecánicamente**. Los demás casos siguen siendo subjetivos.

> JPW (25/05 12:30 UTC, sobre el batch 3): *"si son errores constantes entonces hay que re-escribir las instrucciones y prompts para que sean mejor entendibles por ti".*

El insight de v1.2 es **refactorizar** lo existente en checks ejecutables, NO agregar reglas nuevas.

---

## 2. Decisión

Bumpear `prompts/style_guide.md` a **v1.2** con 8 patches sobre reglas existentes. Cada patch traduce un principio en un test binario operable durante self-check.

### 2.1 Patch 1 — Negritas: test cualitativo del valor scan

Complementa la cobertura ≥80% de v1.1.3 (que sigue vigente) con un test cualitativo binario aplicado por negrita:

> Si elimino la negrita del párrafo, ¿el lector aún capta el punto?

- **Si SÍ** → la negrita es decorativa → eliminar o reescribir sobre la pieza clave.
- **Si NO** → la negrita es valor scan → conservar.

Ejemplos correctos/incorrectos viven en `prompts/style_guide.md` §Negritas-Test-cualitativo-del-valor-scan.

### 2.2 Patch 2 — Tablas: patrón HTML obligatorio

Patrón canónico:

```html
<table>
  <thead><tr><th>...</th></tr></thead>
  <tbody><tr><td>...</td></tr></tbody>
</table>
```

El theme Relevo aplica estilos solo si la tabla tiene `<thead>` + `<th>`. Tablas con `<table><tr><td>...</td></tr></table>` (sin `<thead>`) renderizan como bloque plano sin estilo. Test pasa/falla: ¿la tabla tiene `<thead>` con `<th>` y `<tbody>` con `<td>`? Si NO, falla.

### 2.3 Patch 3 — Cierre: reglas accionables

- El último H2 NO se titula "Cierre", "Conclusión", "Para terminar", "En definitiva", "El final".
- El último párrafo del cuerpo lleva UNA negrita (que pase el test del valor scan).
- El último párrafo NO empieza con conector de cierre prefabricado.

### 2.4 Patch 4 — Grandilocuencia: test del valor cero

> Si quito la metáfora del párrafo, ¿se pierde información o solo se pierde decoración?

- **Si pierde decoración** → eliminar la metáfora.
- **Si pierde información** (era el ejemplo concreto que anclaba el concepto) → conservar.

Lista de antipatrones expandida con los 9 ejemplos específicos del batch 3 (*"el partido se mira a oscuras"*, *"el feudo de hierba"*, *"reescribió el calendario del relevo"*, etc.).

### 2.5 Patch 5 — Párrafos > 60 palabras: conteo binario

Self-check punto 4 ahora exige **contar** las palabras de cada párrafo (no estimar). 60 es techo binario, no guía aproximada. Si alguno excede 60 → dividir antes de devolver.

### 2.6 Patch 6 — Consistencia intra-lista

Regla nueva: todos los items de una misma `<ul>` o `<ol>` deben usar el **mismo patrón sintáctico**. Si el primer item es `<strong>Label</strong>: descripción`, todos los items restantes siguen el mismo formato.

### 2.7 Patch 7 — Antialucinación: listas verificables

Dos tablas explícitas en `prompts/style_guide.md` §Antialucinación-Listas-verificables:

1. **Masters 1000 ATP vigentes** (Indian Wells, Miami, Monte Carlo, Madrid, Roma, Canadá, Cincinnati, Shanghai, Paris). Pekín NO es M1000 hombres.
2. **Geografía de torneos clave** (Halle = Renania del Norte-Westfalia, etc.).

Antes de afirmar un dato verificable, consultar la tabla. Si el dato no está en la tabla, WebSearch obligatorio.

### 2.8 Patch 8 — Cursivas: scan obligado del inventario

Self-check punto 19 ahora exige **abrir el inventario obligado** y escanear el artículo por cada término antes de devolver. Inventario actualizado con `paddock`.

### 2.9 Self-check: 22 → 25 puntos

Tres puntos nuevos:
- **23** — Tablas formato HTML.
- **24** — Consistencia intra-lista.
- **25** — Cierre accionable (no H2 "Cierre", último párrafo con negrita, sin conector prefabricado).

Puntos refrescados con tests binarios explícitos: 4 (conteo de palabras), 13 (test del valor cero), 17 (test del valor scan), 19 (inventario obligado), 21 (listas verificables).

### 2.10 Lo que NO cambia

- El paradigma de voz, léxico, ortotipografía, equivalencias, antialucinación general — todos vigentes.
- Templates v2 (rules/tactics/origins) — inalterados.
- Pipeline `/relevo new` y `/relevo content-plan` — sin cambios estructurales.
- Reglas no negociables (claridad, español peninsular, precisión, sugerir no pontificar, HTML semántico, valor Relevo) — vigentes.
- Cobertura cuantitativa de negritas ≥80% — vigente, complementada por el test cualitativo nuevo.

---

## 3. Alternativas consideradas

### 3.1 Sumar reglas nuevas en lugar de refactorizar las existentes

**Rechazada.** El problema diagnosticado es que las reglas existentes no son accionables. Agregar más reglas (en formato principio) habría ampliado el style guide sin mejorar la operabilidad. v1.2 trabaja sobre lo que ya está.

### 3.2 Esperar feedback de Joel sobre v1.1.3 antes de refactorizar

**Rechazada.** Ya tenemos el feedback (ronda 3, batch 3, 11:15 UTC del 25/5). Esperar otra ronda dilataría el cierre de Phase 3 sin nueva información — los patrones son los mismos del batch 2.

### 3.3 Bumpear a v2.0 (cambio mayor)

**Rechazada.** El cambio es semánticamente acotado: refactor de presentación de reglas existentes, no nuevo paradigma. v1.2 mantiene la trazabilidad del linaje v1.1 → v1.1.1 → v1.1.2 → v1.1.3 → v1.2. Un v2.0 estaría justificado solo si re-pensáramos el paradigma editorial entero (voz, léxico, estructura) — no es el caso.

### 3.4 Aplicar solo los 3 patches más recurrentes (negritas, cierre, factuales)

**Rechazada (por JPW, 25/05 12:35 UTC).** Aplicar los 8 patches en una sola tanda mantiene la coherencia del refactor y evita un v1.2.1 inmediato. El costo marginal de los 5 patches "menos recurrentes" es bajo y cubren patrones que reaparecerían en batch 4.

---

## 4. Impacto y consecuencias

### 4.1 Impacto positivo esperado

- Batch 4 (regen de los 7 ⚠️ del batch 3) bajo v1.2 debe reducir significativamente la recurrencia de los 8 patrones identificados.
- El self-check se vuelve mecánicamente ejecutable: cada punto tiene un check binario claro (cuenta, abre, verifica, elimina).
- Las dos listas verificables (Masters 1000 + geografía de torneos) reducen errores factuales en torneos ATP y ciudades sin requerir WebSearch en el 100% de los casos.

### 4.2 Costo

- 7 artículos del batch 3 (los ⚠️) deben regenerarse bajo v1.2. Decidido por JPW (25/05 12:35 UTC): regenerar-en-bloque siguiendo `[[feedback_regenerate_batch_methodology]]`, NO fixes per-article. Los 2 ✅ del batch 3 (art-4, art-8) se publican tal cual con edición fina manual.
- El self-check de 25 puntos es más largo que el de 22 — esperable que el tiempo de producción por artículo aumente ~10% en el corto plazo. Compensable a escala porque reduce vueltas de revisión.

### 4.3 Validación

- **Smoke v1.2**: regenerar 1 artículo del batch 3 ⚠️ (sugerencia: art-6 Alcaraz, porque contiene 2 errores factuales graves + frases grandilocuentes + negritas mejorables — cubre 4 patches en 1 caso). Verificar que pase los 25 puntos del self-check antes de continuar.
- **Batch 4**: regenerar los 7 ⚠️ restantes (arts 1, 3, 6, 7, 9, 10, 11) bajo v1.2. Asana subtask consolidada a Joel para QA ronda 4. Target: ≥ 6/7 sign-off.
- **Métrica**: si batch 4 logra ≥ 6/7 sign-off → v1.2 es la línea de producción del launch 01/06. Si < 6/7 → v1.3 antes del launch.

---

## 5. Decisiones derivadas

- **Smoke v1.2**: arrancar tras el commit + push de este ADR. Topic candidato: art-6 (Alcaraz Origins) bajo Tactics/Rules según corresponda a la inventory row original. JPW define el topic al retomar post-demo.
- **Batch 4 (post-aprobación del smoke v1.2)**: regenerar los 7 ⚠️ del batch 3 bajo v1.2. Patrón por artículo: subagente genera → genera hero image → sube a WP draft → Asana subtask a Joel.
- **Backlog**:
  - **EDIT-12** (batch QA #3 con Joel) → cierra con el verdict del 25/5 + decisión de v1.2 + plan de batch 4.
  - **EDIT-16 nuevo (a crear)**: regen batch 4 — 7 artículos del batch 3 bajo v1.2. P0, gate del launch 01/06.
  - **EDIT-17 nuevo (a crear)**: batch QA #4 con Joel bajo v1.2 — target ≥ 6/7 sign-off para confirmar línea de producción.
  - **HEADER-REDESIGN**: sigue P2 sin cambios. Será atacado en Phase 4 conjuntamente con el regen de imágenes 3:2 → 16:9 (ADR-009).

---

## 6. Referencias

- `prompts/style_guide.md` v1.2 — la guía aplicada (commit en curso).
- Asana task `1215038718722890` — Joel batch 3 verdict (25/05 11:15 UTC), 9 comentarios per artículo.
- WordPress drafts batch 3: posts 21195/21198/21201/21204/21207/21210/21213/21216/21219 (https://relevo-com.sub.plus).
- ADR-006 (style guide v1.1.2 — ronda 2, 7 patrones cross-batch).
- ADR-008 (style guide v1.1.3 — ronda 3, cobertura ≥80% de negritas + lecciones WP).
- ADR-009 (gap visual del header — deferred a Phase 4).
- `[[feedback_regenerate_batch_methodology]]` — política de regen-en-bloque.
- `[[feedback_asana_hitl_exclusive]]` — Joel verdict y nuevas QA gates vía Asana.
- `[[feedback_jpw_name]]` — JPW como "Juan Pablo Waimann" en docs públicos/ejecutivos.
