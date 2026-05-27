# ADR-014: Style guide v1.2.3 — Bolds 100% coverage + cursivas baloncesto-táctico

**Status**: Approved
**Fecha**: 2026-05-27
**Supersedes**: `prompts/style_guide.md` v1.2.2
**Drivers**: Juan Pablo Waimann (JPW, format review pre-bulk)
**Relacionado**: ADR-008 (v1.1.3 bolds coverage), ADR-010 (v1.2 binary tests), ADR-012 (v1.2.2 biographical patches), ADR-013 (content strategy + bulk 170)

---

## 1. Contexto

El 27/05/2026, post-smokes pre-bulk, JPW ejecutó **format review** sobre los 4 smokes pilots para validar style guide v1.2.2 + templates antes de lanzar el batch piloto de 50 (subset del sprint 170 evergreen). Verdict:

| Smoke | Asana | Categoría | Verdict |
|---|---|---|---|
| #1 Reglas IFAB | `1215173258122308` | Rules | ✅ Approve |
| #2 Pick and roll | `1215173223193222` | Tactics | ⚠️ Needs change |
| #3 Coaches Mourinho | `1215173234263470` | Coaches | ✅ Approve |
| #4 Origins Yamal | `1215173831689778` | Origins | ✅ Approve |

JPW (sobre alcance): *"tene en cuenta que no tengo expertise deportiva, eso es Joel. Mis comentarios son más de formato."*

### 1.1 Feedback JPW sobre smoke #2

Dos correcciones explícitas:

1. *"Bolding missing: there must be bolding in each paragraph"* — el artículo cerró con 23 de 27 párrafos del cuerpo con bold (85% cobertura). Style guide v1.2.2 §17 lo aceptaba (target ≥80% desde v1.1.3). JPW eleva el umbral: cada párrafo body-proper, largo o corto, debe llevar bold.
2. *"pick and roll should be used in italics as pick and roll"* — el artículo usaba *pick and roll* sin cursiva. Style guide v1.2.2 §357-363 tiene inventario obligado en cursiva (clutch, fadeaway, eurostep, etc. — baloncesto técnico), pero `pick and roll` no estaba listado. Aplicando la regla maestra de Joel (*"si dudas, va en cursiva"*) y por la solicitud JPW, va al inventario.

### 1.2 Compatibilidad con Joel

JPW explicitó: *"fijate que mi solicitud de italics coincida con lo que te dijo Joel en el style guide, no quiero ir en contra de sus reglas."* Análisis cruzado:

- **`pressing`** está en **"Aceptados sin cursiva"** según Joel batch 2 (línea 353 v1.2.2). NO se mueve. JPW NO pidió moverlo.
- **`pick and roll`** + términos del mismo registro (alley-oop, fast break, give-and-go, isolation, post-up, full-court press) **no estaban en ningún inventario** v1.2.2. La regla maestra de Joel los coloca en cursiva por defecto. JPW alinea con esa regla.

Decisión: **agregar al inventario obligado, no recategorizar lo que Joel ya consolidó**.

---

## 2. Decisión

Aplicar dos patches universales al `style_guide.md`:

### 2.1 Bolds — cobertura 100% del cuerpo

| | v1.1.3 / v1.2 / v1.2.2 | v1.2.3 |
|---|---|---|
| Umbral cuantitativo | `bolds_count / paragraphs_count ≥ 0.80` | `body_proper_paragraphs_without_bold = 0` |
| Buffer entre 0.70-0.80 | "sumar más" | (n/a, es binario) |
| Excepción label en listas/esquemas | Vigente (v1.1.1) | Vigente (sin cambio) |
| Test del valor scan | Vigente (v1.2) | Vigente (refuerzo) |

Implicaciones operativas:
- El pipeline `/relevo new` self-critique debe detectar `body_proper_paragraphs_without_bold > 0` y disparar revise.
- "body-proper" = párrafos `<p>` dentro del cuerpo del artículo. Excluye captions de imagen, FAQs (gobernadas por su propia regla), labels de schema-de-lista, footers/atribuciones.
- "Largo o corto" — sin excepción por longitud: párrafos de 1-2 líneas también llevan bold.

### 2.2 Cursivas — inventario amplía baloncesto-táctico

Agregadas a la lista de **"tecnicismos que SIEMPRE van en `<em>` la primera vez"**:

`pick and roll`, `alley-oop`, `fast break`, `give-and-go`, `isolation`, `post-up`, `full-court press`.

NO tocadas:
- `pressing` permanece en "Aceptados sin cursiva" (Joel marker batch 2).
- Inventario v1.2 (`clutch`, `fadeaway`, `eurostep`, `rim protector`, `small ball`, `spacing`) sin cambios.
- F1, MMA/boxeo, tenis: sin cambios.

Reorganización visual en §Extranjerismos:
- "Baloncesto técnico (v1.1.2)" — entradas Joel.
- "Baloncesto táctico (v1.2.3 — JPW smoke #2)" — entradas nuevas.

### 2.3 Self-check actualizado

- **#17**: umbral cuantitativo de `≥ 0.80` a `= 1.00`. Texto actualizado a "100% del cuerpo body-proper".
- **#19**: inventario inline ampliado con los 7 términos nuevos + recordatorio explícito de `pressing` SIN cursiva (Joel marker).

Total self-check sigue en **29 puntos** (no se agregan checks nuevos).

---

## 3. No-decisiones / scope-out

- **Otros deportes**: no se inventariaron anglicismos adicionales en fútbol (ej. `playmaker`, `box-to-box`), MMA, ciclismo, etc. La regla maestra "si dudas, va en cursiva" sigue cubriendo los casos no listados. Si emerge ambigüedad recurrente, ampliar en futuras versiones.
- **`pressing`** se reafirma como **sin cursiva** según Joel. NO se mueve al inventario.
- **No se agrega test cualitativo nuevo** sobre bolds — el test del valor scan (v1.2) y el test de aislamiento (v1.1.2) siguen vigentes. v1.2.3 solo eleva el umbral cuantitativo.
- **No regen de smokes ya aprobados** (#1, #3, #4): el patch aplica al **batch piloto de 50** y siguientes. Smoke #2 (Tactics pick and roll) no se regenera individualmente porque entraría en el batch piloto como redo natural si el inventory row 19 cae en el sample (decisión operativa, no de ADR).

---

## 4. Validación

Lock cualitativo:
- ✅ `pick and roll` ahora en italics (smoke #2 feedback resuelto).
- ✅ Bolds 100% obligatorio (smoke #2 feedback resuelto).
- ✅ `pressing` permanece sin cursiva (Joel batch 2 preservado).
- ✅ Compatibilidad con Joel verificada antes de aplicar — JPW explicitó la restricción.

Lock cuantitativo: el pipeline `/relevo new` debe registrar:
- `body_proper_paragraphs_without_bold` en `run_metadata.json`.
- Si > 0 al final del revise → run abortado o re-revisado, NO se devuelve.

---

## 5. Próximos pasos

1. Commit + push de `style_guide.md` v1.2.3 + este ADR.
2. Lanzar batch piloto de 50 (mix 15 rules + 15 tactics + 15 origins + 5 coaches, todos tier-1 preferentemente).
3. Métricas del batch: si la cobertura bolds 100% cae <100% en >5% de los runs, el pipeline tiene gap de implementación, no de regla. Patch operativo, no patch de style guide.
4. Joel ronda 5 (post-batch piloto): puede ajustar `pressing` u otros si lo considera.

---

**Owner**: JPW. **Reviewer post-batch**: Joel Sierra.
