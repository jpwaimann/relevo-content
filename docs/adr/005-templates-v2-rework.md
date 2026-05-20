# ADR-005: Templates rework v2 (Rules / Tactics / Origins) — divergence editorial

**Status**: Approved
**Fecha**: 2026-05-20
**Drivers**: Joel Sierra (defecto #11 del feedback v1), Pablo Waimann
**Relacionado**: ADR-002 (style guide v1.1), ADR-003 (topic ideation lineamientos)

---

## 1. Contexto

El feedback de Joel sobre el batch QA #1 incluía 12 defectos sistemáticos. ADR-002 absorbió 11 de los 12 en el style guide v1.1 (reglas transversales que aplican a las tres categorías). El defecto #12 quedó deferido:

> **#11 — No diferencia el tipo de pieza.** Bio, regla y guía táctica se escriben con la misma lógica estructural cuando deberían tener prompts distintos.

Este ADR documenta la decisión de rework de los tres templates de categoría (`prompts/rules_template.md`, `prompts/tactics_template.md`, `prompts/origins_template.md`) para que diverjan editorialmente en serio y absorban las recomendaciones específicas por tipo de pieza que Joel marcó en el batch.

---

## 2. Decisión

Reescribir los tres templates a **v2.0** con divergencia explícita en cuatro ejes:

| Eje | Rules v2 | Tactics v2 | Origins v2 |
|---|---|---|---|
| **Voz dominante** | Explicación normativa anclada en aplicación | Análisis táctico con anclaje visual | Narrativa con escenas y citas directas |
| **Pregunta central** | ¿Qué dice la norma y qué consecuencias tiene? | ¿Cuándo se usa, quién lo representa, qué cambió? | ¿De dónde viene esta persona y qué la define? |
| **Estructura** | H2s temáticos (cuerpo, espacio, sanciones, polémicas) | H3s item-por-item con esquema fijo dentro de pocos H2s | H2s narrativos en orden flexible (anclajes escena, no cronograma rígido) |
| **3ª capa obligatoria** | Por qué importa hoy + polémicas y debates | Cómo ha evolucionado y qué la cambió | Personalidad pública + lo que dice del deporte |

### 2.1 Cambios clave por template

#### Rules v2

- Diferencia explícita **pillar** vs **sub-rule**, con esqueletos distintos.
- Sub-rule de evolución gana un H2 "**Escena o caso anclaje**" antes del cuerpo normativo (ej. Isner-Mahut 2010 antes de explicar el tie-break).
- Sección opcional "**Polémicas y debates**" cuando la regla genera discusión viva (VAR, fuera de juego semiautomático, paradas tempranas en MMA, ley Wenger).
- Word count tighter: 1500-2000 pillar / 1300-1700 sub-rule.

#### Tactics v2

- **Esquema fijo por item** (H3): qué es / cuándo se usa / quién lo representa / qué cambió. Antes los H3s eran libres y caían en mera enumeración.
- Sub-tipos diferenciados: tipologías, conceptos, comparativas.
- H2 obligatorio "**Cómo ha evolucionado**" — la 3ª capa que Joel marcó como faltante.
- Sección "**¿Cuál es el {item} más {importante}?**" para forzar toma de postura editorial razonada.

#### Origins v2

- **Sustituye el cronograma rígido por bloques narrativos flexibles**. Antes: "Dónde nació / Infancia / Trayectoria / Palmarés". Ahora: inventario de bloques (escena que lo definió, momento bisagra, su estilo, personalidad pública, rivalidades, polémicas, palmarés) que se eligen según el sujeto.
- **2-5 citas directas verificadas obligatorias** cuando existen. WebSearch obligatorio para encontrarlas. Sin invento.
- FAQs reformuladas: prohibido `¿qué nacionalidad tiene`, `¿cuántos años tiene`, `¿dónde nació` (defectos v1.0 marcados por Joel). Preferir preguntas de curiosidad real ("¿por qué representa a X país si nació en Y?", "¿qué decía sobre Z tema?", "¿cómo se conocieron X y entrenador?").
- **Disambiguación obligatoria con segundo apellido** cuando hay homónimos.

### 2.2 Lo que NO cambia

- URL patterns, SEO title patterns, meta description patterns — están alineados con la estructura de relevo.com y funcionan.
- Word count targets se mantienen (con tightening menor en Rules).
- Internal linking rules.
- Sport-specific adaptations (palmarés tables per sport, FAQ variations).
- Relación con `style_guide.md` v1.1 — los templates **no** repiten reglas transversales; las hereda el system prompt cuando ambos se inyectan juntos.

---

## 3. Alternativas consideradas

### 3.1 Mergear todo en `style_guide.md`

**Rechazada.** El style guide cubre **cómo se redacta** (voz, léxico, ritmo, ortotipografía). Los templates cubren **qué estructura tiene cada tipo de pieza** (H2s, H3s, secciones, esqueleto). Mezclarlos diluye ambos docs.

### 3.2 Crear más templates (sub-rule separado de pillar, comparativas separadas de tipologías, etc.)

**Rechazada para v2.** Hoy el editor elige una categoría (rules / tactics / origins) al lanzar `/relevo content-plan`. Multiplicar sub-templates multiplica decisiones para el editor sin ROI claro. Los sub-tipos quedan como secciones dentro de cada template — si más adelante hay volumen suficiente para sub-templates dedicados, se reabre.

### 3.3 Mantener templates v1 y solo añadir notas al margen

**Rechazada.** Joel marcó el defecto #11 como sistémico ("bio, regla y guía táctica se escriben con la misma lógica estructural"). Notas al margen no fuerzan al modelo a divergir; cambiar el esqueleto sí.

### 3.4 Reescribir templates pero sin samples concretos para no inflar el system prompt

**Considerada parcialmente.** Los samples completos del v1 eran largos. v2 los reduce a samples resumidos (esqueleto de H2s + comentarios cortos) en lugar de HTML completo. Mantiene la utilidad de orientación sin balloonar el contexto.

---

## 4. Impacto y consecuencias

### 4.1 Impacto positivo esperado

- Cada categoría produce ahora artículos estructuralmente distintos. Una bio se lee como bio (escenas), una regla como regla (aplicación + polémicas), una guía táctica como guía táctica (cuándo/quién/qué cambió).
- Defecto #11 cerrado.
- Los defectos #2 (ejemplos sin desarrollar), #4 (épica prefabricada) y #11 atacados de forma directa en Origins, que era donde más se notaban.
- Templates más cortos en el prompt (samples resumidos vs HTML completos del v1).

### 4.2 Costo

- **11 artículos del batch QA #1 quedan obsoletos** bajo v2 también — la regeneración (`EDIT-04`) ahora usa style_guide v1.1 + templates v2.
- **Curva de calibración**: la primera tanda de artículos bajo v2 puede sobre-aplicar las reglas nuevas (demasiado escena, demasiado tres-capas) hasta que el modelo encuentre el equilibrio. Mitigación: el smoke test del 20/05 sobre tie-break-quinto-set (rules sub-rule de evolución) validó que el equilibrio se encuentra en un solo paso.
- **Riesgo de inconsistencia con artículos publicados en relevo.com hoy**. Algunos artículos viejos del site siguen patrones v1.0. La inconsistencia se irá resolviendo a medida que se publiquen los nuevos; no se reescribirán los viejos de forma masiva.

### 4.3 Validación

- **Smoke test post-rework**: el artículo `smoke-v1-1-tiebreak-1779298120` se generó bajo style guide v1.1 + rules_template v2 (sub-rule de evolución). Pasa el self-check automático de v1.1 (entradilla 35w, primer H2 6w, bolds de 5-7 palabras, ningún párrafo > 60 palabras, FAQs no obvias). Ver `runs/smoke-v1-1-tiebreak-1779298120/critique_v1_1.json`.
- **Batch QA #2 con Joel** (EDIT-08) sobre los artículos regenerados bajo v1.1 + v2 — la validación end-to-end editorial.

---

## 5. Decisiones derivadas

- **Backlog `EDIT-05`** se cierra con este commit.
- **Backlog `EDIT-04`** (re-run batch) usa style_guide v1.1 + templates v2 a partir de ahora.
- Cuando llegue feedback adicional de Joel sobre uno de los tres templates, se incorpora con bump de versión (v2.1, v3.0…) del template afectado, sin necesidad de nueva ADR salvo cambio estructural.

---

## 6. Referencias

- `prompts/rules_template.md` v2.0
- `prompts/tactics_template.md` v2.0
- `prompts/origins_template.md` v2.0
- `prompts/style_guide.md` v1.1 — voz y reglas transversales que los templates heredan
- `prompts/joel-feedback-v1.md` — los 12 defectos sistemáticos, incluyendo #11
- `runs/smoke-v1-1-tiebreak-1779298120/article.html` + `critique_v1_1.json` — primer artículo bajo v1.1 + v2
- ADR-002 (style guide v1.1), ADR-003 (topic ideation lineamientos), ADR-004 (umbrella hubs)
