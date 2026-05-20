# ADR-003: Lineamientos de ideación de topics — Joel topic feedback v1

**Status**: Approved
**Fecha**: 2026-05-20
**Drivers**: Joel Sierra (review editorial de ideación), Pablo Waimann (project lead)
**Relacionado**: ADR-002 (style guide v1.1 — review de redacción)

---

## 1. Contexto

El 18/05/2026 el comando `/relevo content-plan` seedeó 148 topics en el Excel inventory (`Relevo - Content - Article production DB.xlsx`), cubriendo los 24 deportes en sus tres categorías (Rules, Tactics, Origins). El batch se generó con dedup interno + dedup contra inventory existente, pero **sin** dedup contra el corpus público de relevo.com y **sin** verificación granular de nombres propios en las bios.

El 20/05/2026 PM, Joel Sierra pasó por Teams (no Asana) un feedback editorial sobre la **ideación** de ese batch. Es la segunda pata de su revisión editorial — la primera (ADR-002) cubrió la redacción de los 11 artículos del batch QA #1; esta cubre los topics antes de redactar.

El feedback tiene dos capas:
- **Tactical** — fixes concretos sobre filas específicas del inventory: 3 topics elementales a reformular, 4 imprecisiones factuales en bios, dedup contra relevo.com, 4 angle topics y ~22 bios nuevos a agregar. Detalle completo en `prompts/joel-topic-feedback-v1.md`.
- **Policy** — principios generales que deben regir toda futura ideación, independientemente del batch.

Esta ADR documenta la **decisión de codificar los principios generales** en el flujo de ideación. Los fixes tácticos se ejecutan como tareas del backlog y no requieren ADR.

---

## 2. Decisión

Crear un nuevo doc editorial `prompts/topic_ideation_lineamientos.md` (paralelo a `style_guide.md`, pero específico para la fase de ideación), e inyectarlo en el system prompt del comando `/relevo content-plan` para que el LLM lo respete en cada generación.

El doc contiene cinco lineamientos:

| # | Lineamiento | Resumen |
|---|---|---|
| **L1** | Evitar topics elementales en deportes de tier-1 | Fútbol/tenis/MMA/F1/baloncesto: el topic obvio ("qué es X", "cuánto dura Y") pierde contra contenido genérico ya posicionado. Atacar desde ángulo no obvio. Excepción: deportes minoritarios |
| **L2** | Dedup obligatorio contra relevo.com | Sitemap scan o `site:relevo.com` antes de seedear. Solapamiento total → blocked. Solapamiento parcial → editar ángulo |
| **L3** | Verificación de nombres propios en bios | Toda bio requiere validación de existencia, apellido, deporte, posición, lugar de nacimiento. Usar segundo apellido en homónimos. No inventar atletas |
| **L4** | Curaduría editorial sobre bios | Atletas con peso real para el lector español. No seedear desde listas algorítmicas. Mejor 5 bios excelentes que 20 mediocres |
| **L5** | Ángulo FIFA WC 2026 cuando aplica | Para topics de fútbol en mayo-julio 2026, incorporar el ángulo del Mundial (VAR saques de esquina, formato 48 equipos, sedes) |

### 2.1 Lugar del doc en la arquitectura editorial

| Doc | Cubre | Se inyecta en |
|---|---|---|
| `style_guide.md` v1.1 | Cómo se redacta (voz, léxico, ritmo, ortotipografía, HTML) | `/relevo new` Y `/relevo content-plan` |
| `topic_ideation_lineamientos.md` v1.0 | Qué se seedea (criterios de selección y descarte de topics) | `/relevo content-plan` únicamente |
| `rules_template.md` / `tactics_template.md` / `origins_template.md` | Estructura específica por categoría | El template correspondiente en cada comando |

### 2.2 Inyección en el system prompt

En `cmd_content_plan.md` Paso 5, el system prompt pasa de:
```
style_guide.md → template categoría → fuentes
```
a:
```
style_guide.md → topic_ideation_lineamientos.md → template categoría → fuentes
```

Y la sección "Restricciones (NO romper)" del prompt gana una regla #6 explícita referenciando los lineamientos.

---

## 3. Alternativas consideradas

### 3.1 Mergear los lineamientos dentro de style_guide.md

**Rechazada.** `style_guide.md` cubre redacción; los lineamientos cubren ideación. Mezclarlos diluye ambos docs y complica el versionado independiente. Además, `/relevo new` no necesita los lineamientos de ideación (el topic ya está fijado), así que cargarlos en el system prompt de ese comando sería ruido.

### 3.2 Codificar los lineamientos directamente en `cmd_content_plan.md` sin doc externo

**Rechazada.** `cmd_content_plan.md` es **spec del comando** (qué hace el skill, paso a paso). Los lineamientos son **política editorial** del equipo de contenido — patrimonio de Joel y Pablo, no del skill. Separarlos permite que el equipo de contenido los edite vía PR sin tocar la lógica del skill.

### 3.3 Sólo aplicar los fixes tácticos y dejar la política implícita

**Rechazada.** Sin codificar los lineamientos, el siguiente `/relevo content-plan` repetirá los mismos defectos (topics elementales en deportes tier-1, falta de dedup vs relevo.com). El feedback de Joel apunta a problemas sistémicos, no a un batch malo.

---

## 4. Impacto y consecuencias

### 4.1 Impacto positivo

- Cada futuro batch de `/relevo content-plan` aplicará L1–L5 automáticamente vía system prompt.
- El editor humano tiene un checklist explícito (L1–L5) para revisar batches manualmente.
- Trazabilidad: la política editorial vive en GitHub, versionada, separada del código.
- Patrón replicable: cuando llegue feedback adicional, se incorpora con bump de versión sin cambios estructurales.

### 4.2 Costo

- **Inventory actual (148 topics) inconsistente con la política nueva.** Los fixes tácticos (3 topics reformulados, 4 imprecisiones, dedup vs relevo.com, expansión con 4 angle topics + 22 bios) se ejecutan como tareas independientes del backlog (`EDIT-07` y sub-items).
- **System prompt más largo en `/relevo content-plan`.** Marginal (~3 KB adicionales). Sin impacto medible en latencia o costo.
- **Curva de aprendizaje para nuevos editores.** Mitigada con el `Cómo se aplican estos lineamientos` del propio doc.

### 4.3 Validación

- **Smoke test post-lineamientos**: generar un batch nuevo de 5 topics en un deporte tier-1 (ej. Tenis / Rules) con los lineamientos aplicados. Verificar que ningún topic generado sea elemental y que dedup vs relevo.com esté incorporado.
- **Joel batch QA #2 de ideación**: cuando se reseedee después de los fixes, pedir a Joel una segunda revisión sobre el inventory actualizado.

---

## 5. Decisiones derivadas

- **Backlog `EDIT-07`** (nuevo): aplicar al inventory los fixes tácticos del feedback (3 topic reformulations, 4 factual fixes, dedup vs relevo.com, seed expansion 4 + 22).
- **Backlog `EDIT-08`** (nuevo): batch QA #2 de ideación con Joel sobre el inventory post-fixes.
- Si Joel pasa feedback futuro sobre ideación → bump `topic_ideation_lineamientos.md` (v1.1, v1.2…) con changelog visible, sin necesidad de nueva ADR salvo cambio estructural.

---

## 6. Referencias

- `prompts/topic_ideation_lineamientos.md` v1.0 — la política aplicada.
- `prompts/joel-topic-feedback-v1.md` — feedback consolidado (tactical + policy).
- `.claude/skills/relevo/lib/cmd_content_plan.md` — comando que inyecta los lineamientos.
- ADR-002 — style guide v1.1, decisión hermana para el lado de redacción.
- ADR-001 — arquitectura del workflow.
