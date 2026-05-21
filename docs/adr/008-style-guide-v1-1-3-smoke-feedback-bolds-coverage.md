# ADR-008: Style guide v1.1.3 — Reformulación de la regla de negritas (target de cobertura) + lecciones de integración WordPress

**Status**: Approved
**Fecha**: 2026-05-21 (mismo día que ADR-006 y ADR-007)
**Supersedes**: `prompts/style_guide.md` v1.1.2 (sección "Frecuencia" de negritas + self-check punto 17)
**Drivers**: Pablo Waimann (revisión del smoke v1.1.2 publicado en beta de WordPress)
**Relacionado**: ADR-006 (style guide v1.1.2), ADR-007 (hero image policy)

---

## 1. Contexto

El 21/05/2026, tras cerrar ADR-006 (v1.1.2) y ejecutar el smoke test asociado, el artículo regenerado (Fútbol Tactics, "Los tipos de tiro en el fútbol", `runs/smoke-v1-1-2-tiros-1779396487/`) se publicó como draft en la beta de WordPress (post `21187`, https://relevo-com.sub.plus). Pablo lo revisó renderizado contra el CSS recién aplicado (commit theme-side via WP MCP) y entregó micro-feedback el mismo día con tres puntos:

### 1.1 Hero image — ✅ aprobada sin cambios

Generada por gpt-image-2 (`scripts/imagen/generate-images.py`) y subida vía `scripts/wp_upload.py` como attachment `21189`. Pablo: *"impresionante, hermosa"*. Política gate-by-text de ADR-007 sigue pendiente de implementación; mientras tanto el pipeline genera sin gate y el resultado del primer intento fue aprobado en directo. **No bloquea esta iteración.**

### 1.2 Renderizado WordPress — ❌ duplicación + white space

La entradilla apareció dos veces en la página: una arriba de la línea autor/fecha/share, otra inmediatamente después. Causa: el pipeline subió la entradilla simultáneamente como `post_excerpt` (snippet) y como primer `<p>` de `post_content`. El theme Relevo renderiza `excerpt` arriba del banner como bloque introductorio Y renderiza el `content` con su primer párrafo intacto debajo. Resultado:

```
[H1 desde post_title]
[entradilla desde post_excerpt]
[white space ~vh grande]
[banner autor/fecha/share/whatsapp/X]
[entradilla otra vez como primer <p> de post_content]
[H2 Por qué importa el repertorio]
…
```

Pablo marcó AMBAS molestias (white space + duplicación). El white space es consecuencia secundaria del bloque excerpt: al eliminar el excerpt desaparecen los márgenes que generaban el gap.

### 1.3 Bolds — ❌ aplicación no cumple la regla de cobertura

La regla v1.1.2 (sección "Frecuencia") decía: *"Una sola negrita por párrafo en la mayoría de casos."* El smoke v1.1.2 produjo **14 bolds en 59 párrafos** = **24% de cobertura**. Pablo aportó ejemplo concreto del fragmento problemático:

> *Ejemplo incorrecto (1 bold por sección):*
> P1: "Pegarle al balón parece la acción más sencilla del fútbol. (…)"
> P2: "Una distinción que conviene asentar de entrada. El penalti y la falta directa no son tipos de tiro, (…)"
> P3: "**Tener varias soluciones distingue al goleador moderno.** El delantero contemporáneo elige (…)"
>
> *Ejemplo correcto (1 bold por párrafo):*
> P1: "Pegarle al balón **parece la acción más sencilla del fútbol**. Es probablemente la más matizada. (…)"
> P2: "Una distinción que conviene asentar de entrada. El **penalti y la falta directa no son tipos de tiro**, (…)"
> P3: "**Tener varias soluciones distingue al goleador moderno.** El delantero contemporáneo elige (…)"

El modelo interpretó *"una sola negrita por párrafo en la mayoría de casos"* como un **techo permisivo** ("1 max por párrafo, vale si pongo menos") en lugar de un **target de cobertura** ("la mayoría de los párrafos debe tener una"). La formulación literal admitía ambas lecturas.

---

## 2. Decisión

Bumpear `prompts/style_guide.md` a **v1.1.3** con un único cambio acotado:

### 2.1 Patch a la sección "Frecuencia" de negritas

La reformulación pasa de descripción permisiva a **target cuantitativo con anti-patrón explícito**:

| Antes (v1.1.2) | Después (v1.1.3) |
|---|---|
| "Una sola negrita por párrafo en la mayoría de casos." | "**La mayoría de los párrafos del cuerpo lleva UNA negrita.** El target es **≥ 80%**." |
| (sin ejemplo de fallo) | "❌ **Falla común (smoke v1.1.2): 1 negrita por sección/H2/H3, no por párrafo.** La negrita marca el hilo escaneable del cuerpo párrafo a párrafo, NO el resumen del apartado." |
| (sin verificación operativa) | "Heurística: lee tu draft escaneando SOLO las negritas en orden. Debes poder seguir la línea argumental." |
| (sin métrica) | "`bolds_count / paragraphs_count ≥ 0.80`. Si baja de 0.70 el artículo está subbold y vuelve a draft." |

### 2.2 Patch al self-check punto 17

Refuerzo cuantitativo:

| Antes | Después |
|---|---|
| "¿una negrita por párrafo (excepcionalmente dos)?" | "¿**≥ 80% de los párrafos del cuerpo lleva UNA negrita** (no 1 por sección/H2)? Cálculo obligatorio: `bolds_count / paragraphs_count ≥ 0.80`; si está entre 0.70-0.80 sumas más; si < 0.70 reescribes." |

### 2.3 Lo que NO cambia

- Resto del style guide (entradilla, primer H2, ritmo, léxico, antialucinación) — inalterado.
- Templates v2 (rules/tactics/origins) — inalterados.
- Pipeline `/relevo new` y `/relevo content-plan` — sin cambios estructurales.
- Self-check punto 18 (test de aislamiento de bolds v1.1.2) — sigue vigente y se complementa con el punto 17 reformulado.
- ADR-007 hero image policy — sigue PENDIENTE de implementación; no se toca.

### 2.4 Lecciones de integración WordPress (no van al style guide)

Estas no son reglas de redacción sino patrones de pipeline. Se documentan acá pero la implementación queda en código:

- **`post_excerpt` Y entradilla en `post_content` duplican el bloque introductorio en el theme Relevo.** Decisión: el pipeline de upload deja `post_excerpt` vacío y mantiene la entradilla como **primer `<p>` de `post_content`**. WordPress puede regenerar excerpt automáticamente desde content si lo necesita para meta-description en listing pages, o se setea explícitamente sólo cuando difiera de la entradilla.
- **El white space entre excerpt-block y banner autor desaparece al eliminar el `post_excerpt`** — no requirió fix CSS adicional. Si en algún artículo futuro se quiere reintroducir `post_excerpt` para SEO de listing pages, hay que evaluar si el theme también renderiza el excerpt arriba o sólo lo usa como meta. Para el smoke v1.1.3 y el batch 3 entero: excerpt vacío.
- **wp_upload.py requería `expected_domain`** (wrong-site guard agregado al proxy 0.2.1 después del 19/5). Fix aplicado: nuevo flag `--domain` con default `relevo-com.sub.plus` + propagación a la llamada `media.upload`. Helper ahora levanta `isError: true` del MCP en vez de crashear con `Expecting value` en json parse.

---

## 3. Alternativas consideradas

### 3.1 Mantener v1.1.2 y agregar instrucción extra al prompt de regen

**Rechazada.** La regla pertenece al style guide persistente. Agregar instrucciones one-shot al prompt del agente no resuelve el problema en los siguientes 50, 100, 1000 artículos que se generarán bajo el mismo style guide. La reformulación tiene que estar donde el modelo la lee siempre.

### 3.2 Bumpear a v1.2 (cambio mayor)

**Rechazada.** El cambio es acotado (una sección + un punto del checklist). No introduce reglas nuevas — refuerza una existente con métrica cuantitativa. Bump menor `v1.1.3` mantiene la trazabilidad del linaje v1.1 → v1.1.1 → v1.1.2 → v1.1.3 como refinamientos del mismo paradigma.

### 3.3 Esperar feedback de Joel sobre el smoke v1.1.2 antes de bumpear

**Rechazada.** Pablo identificó el patrón en una lectura rápida y el fix es 10 minutos de patch. La iteración rápida (smoke v1.1.2 → feedback Pablo → v1.1.3 → smoke v1.1.3 → feedback Joel → batch 3) mantiene la cadencia de Phase 4 (deadline 22/05). Esperar a Joel agregaría un día completo.

### 3.4 Cambiar el CSS para reducir el gap excerpt-banner manteniendo el excerpt

**Rechazada.** Tocar el theme CSS para compensar una duplicación de contenido es taparlo, no resolverlo. La causa raíz es que el contenido aparece dos veces; el fix correcto es no subir el excerpt. Si más adelante un caso de uso REAL necesita excerpt distinto (listing SEO), entonces sí se evalúa CSS.

---

## 4. Impacto y consecuencias

### 4.1 Impacto positivo esperado

- Smoke v1.1.3 alcanza ≥80% bold coverage (target medible vs interpretable).
- Batch 3 (9 artículos) se genera con el patrón "1 bold por párrafo" consistente — desbloquea el sign-off de Joel.
- Pipeline de upload limpio: 1 sola entradilla rendereada por artículo, sin gap fantasma.

### 4.2 Costo

- El smoke v1.1.2 (post 21187) queda obsoleto. Se actualiza in-place con el contenido v1.1.3 cuando esté listo, sin crear un post nuevo (la URL `tipos-tiro-futbol` se reusa).
- La hero image (attachment 21189) se preserva — sigue siendo válida para el contenido v1.1.3.

### 4.3 Validación

- Post-regen, verificar `bolds_count / paragraphs_count ≥ 0.80` en `critique_v1_1_3.json`.
- Post-update del post 21187, Pablo abre la URL preview y confirma: (a) no hay duplicación, (b) cada párrafo tiene ~1 bold, (c) la hero sigue arriba.

---

## 5. Decisiones derivadas

- **Smoke v1.1.3**: regenerar el mismo topic (Fútbol Tactics tipos de tiro) bajo v1.1.3, reusando la hero. Output a `runs/smoke-v1-1-3-tiros-{epoch}/`. Cuando esté listo, actualizar in-place el `post_content` del post 21187 (no crear post nuevo) y dejar `post_excerpt` vacío.
- **Batch 3 (post-aprobación del smoke v1.1.3)**: regenerar los 9 artículos ⚠️ del batch 2 bajo style_guide v1.1.3. Cada uno:
  - Generado por subagente con v1.1.3 + tactics/rules/origins template v2.
  - Hero image generada + attacheada al post.
  - Subido a WordPress beta como draft (post_excerpt vacío, entradilla como primer `<p>`).
  - Asana subtask creado para Joel — review en la beta antes de touch-base con Pablo.
- **Backlog**:
  - **EDIT-08** (QA batch 2 con Joel) → cierra con el feedback ronda 2 ya integrado en v1.1.2 + el micro-feedback de Pablo integrado en v1.1.3.
  - **EDIT-10** (regen batch 3): vigente, ahora bajo v1.1.3.
  - **EDIT-11** (QA batch 3 con Joel): se invoca vía Asana subtask por artículo en el upload pattern.
  - **EDIT-12** (nuevo, captured aquí): documentar el upload pattern definitivo en `lib/cmd_new.md` — `post_excerpt` vacío por default + entradilla como primer `<p>` de `post_content`. Hoy es una convención implícita en este ADR; mañana debe ser parte del pipeline doc.
  - **EDIT-13** (ADR-007 hero gate impl): sigue pendiente, sin novedad.
  - **EDIT-14** (CSS gap audit): captured aquí. Si en futuras iteraciones reintroducimos `post_excerpt`, hay que medir el gap excerpt→banner con dev tools y decidir si se necesita override en theme CSS.

---

## 6. Referencias

- `prompts/style_guide.md` v1.1.3 — la guía aplicada (commit en curso).
- WordPress post `21187` (https://relevo-com.sub.plus, status: draft) — smoke v1.1.2 que recibió el feedback.
- WordPress attachment `21189` — hero image aprobada.
- `runs/smoke-v1-1-2-tiros-1779396487/` — artefactos del smoke v1.1.2.
- `runs/smoke-v1-1-3-tiros-{epoch}/` — artefactos del smoke v1.1.3 (en producción al cierre de este ADR).
- ADR-006 (style guide v1.1.2 — la versión que esta v1.1.3 refina).
- ADR-007 (hero image policy — sigue pendiente de implementación).
- `scripts/wp_upload.py` — helper de upload, ahora con `--domain` flag y error handling sobre `isError`.
- Memoria persistente `feedback-regenerate-batch-methodology` — sigue vigente: regen-en-bloque tras cada feedback.
- Memoria persistente `feedback-asana-hitl-exclusive` — sigue vigente: cada artículo del batch 3 = 1 subtask Asana a Joel.
