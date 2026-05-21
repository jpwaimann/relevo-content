# ADR-006: Style guide v1.1.2 — Incorporación del feedback editorial Joel batch QA #2

**Status**: Approved
**Fecha**: 2026-05-21
**Supersedes**: `prompts/style_guide.md` v1.1.1 (20/05/2026)
**Drivers**: Joel Sierra (review editorial), Pablo Waimann (project lead)
**Relacionado**: ADR-002 (style guide v1.1), ADR-005 (templates v2 rework)

---

## 1. Contexto

El 20/05/2026, tras los reworks de ADR-002 (style guide v1.1) y ADR-005 (templates v2), el pipeline regeneró el batch de 11 artículos bajo `style_guide.md` v1.1.1 + `rules_template.md` / `tactics_template.md` / `origins_template.md` v2.0 (cerrando `EDIT-04`).

El 21/05 Joel Sierra entregó el feedback completo del batch QA #2 en la subtask de Asana **EDIT-08** (`1214981761607196`, "QA - Second batch (11 articles regenerated under v1.1 + templates v2)"). El feedback llegó en dos comentarios separados (mañana y tarde) cubriendo el smoke test y los 11 artículos.

### 1.1 Veredicto general de Joel

Cita literal del comentario del 21/05/2026 11:31 UTC:

> *"El batch ha mejorado mucho respecto al anterior. Hay más cultura deportiva y táctica, un mejor contexto histórico y ejemplos reconocibles. (…) la base ya es claramente publicable con revisión editorial real. El siguiente salto pasa por más escenas, más ejemplos icónicos desarrollados, más capas tácticas/históricas en algunos deportes, más conexión entre reglas/técnica y evolución moderna del deporte en algunos casos y algunos matices en cuanto al relato narrativo o redacción para hacer los textos más fluidos, cohesionados y/o periodísticos."*

**Conclusión clave:** los cambios estructurales de ADR-002 + ADR-005 funcionaron — los problemas estructurales (defectos #1-#12 de la ronda 1) están cerrados. Los problemas residuales son **transversales y de superficie editorial**, no de arquitectura de prompt.

### 1.2 Métrica antes/después

| Métrica | Batch #1 (15/5, pre-v1.1) | Batch #2 (20/5, post-v1.1.1 + v2) |
|---|---|---|
| Artículos con sign-off Joel | 1 / 11 (smoke test) | 2 / 11 (smoke + art 5 tenis) |
| Veredicto general Joel | "muchos defectos sistemáticos" | "base ya es claramente publicable" |
| Defectos estructurales transversales | 12 sistémicos | 0 estructurales / 7 transversales editoriales |
| Defectos de plantilla por categoría | sí (defecto #11) | resueltos |

### 1.3 Los 7 patrones transversales identificados

Joel marcó 9 de los 11 artículos como `⚠️ Needs change` (arts 1, 2, 3, 4, 6, 7, 8, 9, 10, 11). El análisis cruzado del feedback agrupa los problemas en **7 patrones recurrentes**, no en defectos per-artículo:

1. **Bolds**: aplicación sigue incorrecta en los 11 artículos. La v1.1.1 ya tenía la regla "frase de 5-7 palabras consecutivas" + la excepción de bolds-como-label. Pero el modelo sigue cayendo en el patrón v1.0 — destacar tecnicismos sueltos (porpoising, pole, FIA, pit-wall) en lugar de ideas legibles. Joel: *"Como ya hemos hablado, la aplicación de las negritas sigue sin ser correcta en todos los artículos."*
2. **Entradillas flojas** (arts 1, 2, 4, 6, 8, 9): tienden a abrir con una anécdota singular o un caso paradigmático único en lugar de situar el artículo entero.
3. **Primer H2 demasiado circunstancial** (arts 2, 8, 11): empieza con un ejemplo paradigmático puntual (Barça de Flick, Singapur 2024, un gol icónico) en lugar de un marco general o pregunta funcional.
4. **Errores fácticos por atribución a jugadores** (arts 1, 2, 7, 9): WebSearch insuficiente para verificar atribuciones específicas (Iniesta extremo Rijkaard, Haaland/Lewandowski viven del raso, Gerrard/Iniesta/Modrić/Quaresma/Lamela mal atribuidos).
5. **Léxico ES-España** (arts 4, 7, 8, 9, 10): falta de equivalencias específicas (momios → cuotas), género incorrecto de organizaciones (el MMA → las MMA, el UFC → la UFC), anglicismos técnicos sin cursiva (pit-wall, budget cap, porpoising, clutch), cursivas mal puestas sobre palabras españolas (amago en cursiva como error en art 3).
6. **Staccato — cadenas de frases muy cortas**: Joel marcó *"frases excesivamente cortas en muchos puntos que no favorecen la continuidad lectora"*. Ejemplo art 7: *"El Singapur 2024 lo enterró."* en cadena con otras frases muy cortas.
7. **Escenas icónicas mencionadas pero no desarrolladas**: la regla v1.1 de "2-5 ejemplos icónicos desarrollados" se cumple en cantidad pero falla en profundidad — los ejemplos se citan sin desarrollar la escena visual.

### 1.4 Erratas factuales puntuales (no patrones)

Además de los 7 patrones, Joel marcó **erratas específicas** que no son cross-batch sino fallos individuales:

- Art 2 fila "Tiro libre" de la tabla resumen contenía la definición de penalti (*"desde otro metros, mano a mano con el portero"*).
- Art 9 *"Halle, clima del Báltico"* — Halle no está en el Báltico.
- Art 1 atribución imprecisa de la posición histórica de Iniesta en el Barça de Rijkaard.

Estas no entran al style guide como reglas globales — entran al fact-check pass como **patrones de verificación obligatoria** (sección 2.3 de este ADR).

---

## 2. Decisión

Bumpear `prompts/style_guide.md` a **v1.1.2** incorporando **8 patches** que atacan los 7 patrones cross-batch y formalizan reglas explícitas de fact-check. Mantener intactos los templates v2 (ADR-005) y la arquitectura del pipeline — los problemas residuales son de superficie editorial, no estructurales.

### 2.1 Los 8 patches de v1.1.2

| # | Sección afectada | Cambio | Patrón atacado |
|---|---|---|---|
| 1 | Negritas — regla de oro | "Si dudas, no marques" + ejemplo negativo explícito de tecnicismos sueltos (porpoising, pole, FIA, pit-wall) | #1 Bolds |
| 2 | Negritas — verificación post-escritura | Test obligatorio: extraer bolds del artículo, leerlos en orden, verificar que cada uno se lee como idea completa o label de ítem | #1 Bolds |
| 3 | Entradilla — sitúa el ARTÍCULO ENTERO | Clarificación + ejemplo negativo: arrancar con anécdota singular falla; la entradilla habla del concepto, no de un caso paradigmático único | #2 Entradillas |
| 4 | Primer H2 — marco general | Prohibición explícita de abrir con ejemplo paradigmático singular, partido específico o equipo concreto; H2 abre con marco general o pregunta funcional | #3 Primer H2 |
| 5 | Ritmo — anti-staccato | Ban explícito de 3+ frases muy cortas consecutivas; ejemplo de Joel del art 7 (Singapur 2024) | #6 Staccato |
| 6 | Léxico ES — equivalencias nuevas + inventario tecnicismos | Nuevas equivalencias (momios→cuotas, el MMA→las MMA, el UFC→la UFC, pívot rim protector→pívot defensor del aro); inventario explícito de tecnicismos siempre en `<em>` (pit-wall, budget cap, porpoising, clutch, fadeaway…); inventario explícito de palabras españolas que NO van en cursiva (amago, escuadra, taconazo, rosca…) | #5 Léxico |
| 7 | Antialucinación — 5 patrones de fact-check obligatorio | Atribución de gesto técnico a jugador, datos geográficos/climáticos, datos numéricos vivos, posiciones/roles tácticos atribuidos, atribuciones técnicas discutibles | #4 Errores fácticos |
| 8 | Self-check — 20 → 22 puntos | Nuevo punto 18 (test de aislamiento de bolds), nuevo punto 19 (cursivas con inventario obligado + falsa cursiva), punto 21 (fact-check específico) más exigente, punto 22 (valor Relevo) desdoblado del antiguo punto 20 | Transversal |

### 2.2 Lo que NO cambia

- Templates v2 — no se tocan. Los defectos de ronda 2 son superficie editorial, no estructura.
- Pipeline `/relevo new` y `/relevo content-plan` — sin cambios arquitectónicos.
- Arquitectura ADR-001 — el style guide se sigue inyectando antes del template; el orden y composición del system prompt se mantienen.
- URL patterns, SEO patterns, internal linking — sin cambios.
- Word counts — sin cambios.
- Excepción bolds-como-label (v1.1.1) — se mantiene intacta.

### 2.3 Metodología "regenerate-en-bloque" como decisión durable

Una decisión metodológica importante implícita en este ADR — y validada por Pablo el 21/05 — es:

> **Tras cada ronda de feedback editorial, NO fixear artículos uno-por-uno a mano. Aplicar patches transversales al style guide / templates / pipeline y regenerar el batch completo bajo la nueva versión.**

Esto aplica a este ADR (regen batch 3 cierra `EDIT-10` — ver §5) y a todas las rondas futuras de QA con Joel hasta que la metodología se rediscuta. Las erratas puntuales (§1.4) son la única excepción: van a reglas reusables del fact-check pass, no a fixes manuales.

Esta decisión está documentada también en la memoria persistente del agente como `feedback-regenerate-batch-methodology`.

### 2.4 Hero image policy → ADR-007

Joel también marcó en una tarea paralela (`1214965670270081`, "WP visual check") que las hero images no ilustran adecuadamente arts 1 y 2 del batch. La solución acordada — un **gate-by-text** donde el pipeline propone el prompt textual de la imagen, un reviewer humano lo aprueba/rechaza en Asana antes de invocar gpt-image-2, y luego una segunda subtask Asana valida la imagen final — se documenta en **ADR-007 (en curso)**, no aquí. Este ADR se limita al style guide.

---

## 3. Alternativas consideradas

### 3.1 Reescribir los templates v2 también

**Rechazada.** Joel fue explícito: *"la base ya es claramente publicable con revisión editorial real"*. Los templates v2 funcionaron — los defectos residuales son transversales y de superficie. Reescribir templates introduce churn sin ROI claro; se reabriría solo si una ronda 3 muestra que los patrones siguen sin moverse pese a los patches del style guide.

### 3.2 Patches per-artículo en lugar de v1.1.2 global

**Rechazada.** Los 7 patrones son cross-batch — aparecen en 9 de los 11 artículos. Fixearlos uno-por-uno a mano (i) introduciría 9 estilos distintos de "fix", (ii) no resuelve el problema en futuros artículos (los siguientes 50, 100, 1000 también tendrán los mismos defectos sin cambio en el style guide), (iii) confunde la señal del pipeline: lo que estamos midiendo es la calidad del prompt, no la calidad del fixeo humano.

Pablo confirmó la metodología regenerate-en-bloque el 21/05 (§2.3).

### 3.3 Esperar a una ronda 3 antes de bumpear

**Rechazada.** Deadline 22/05 (Phase 4 close) + 01/06 (public launch) no dan margen. Joel marcó los 7 patrones con suficiente claridad y cantidad de muestra (9 artículos). Bumpear hoy + regenerar batch 3 + QA ronda 3 entrega un ciclo más antes del launch.

### 3.4 Bump menor (v1.1.2 patch) vs bump mayor (v1.2)

**Decisión: bump menor.** Los cambios son refinamientos sobre las reglas de v1.1 — no introducen nuevas reglas transversales fundamentales. El changelog explica las 7 mejoras pero no rompe compatibilidad con v1.1 / v1.1.1. La excepción bolds-como-label de v1.1.1 sigue intacta.

---

## 4. Impacto y consecuencias

### 4.1 Impacto positivo esperado

- Cierre de los 7 patrones cross-batch que Joel marcó: bolds, entradillas, primer H2, errores fácticos, léxico ES, staccato, escenas no desarrolladas.
- Style guide más explícito en sus zonas de fricción frecuente: trampa de tecnicismos sueltos como bolds, falsa cursiva sobre palabras españolas, atribuciones de gesto sin WebSearch.
- Self-check más estricto (22 puntos vs 20) con un test específico de bolds-en-aislamiento que ataca el patrón más persistente del modelo.
- Documentación del fact-check pass mejora: 5 patrones explícitos que requieren WebSearch obligatorio, con ejemplos de batch 2.

### 4.2 Costo

- **Los 9 artículos marcados ⚠️ por Joel quedan obsoletos** bajo v1.1.2. Regenerarse como batch 3 — capturado en backlog como `EDIT-10` (sustituye al cerrado `EDIT-04`).
- **El smoke test y el art 5 tenis** que ya tenían sign-off Joel se mantienen como están — no se regeneran, aunque la opción de re-correr el self-check v1.1.2 sobre ellos queda abierta como sanity check si se considera barato.
- **Riesgo de overcorrección**: la regla "si dudas, no marques" para bolds podría llevar al modelo a producir artículos con cero negritas. Mitigación: el test de aislamiento (patch 2) verifica que las negritas presentes sean buenas, no que haya suficientes — pero el self-check punto 17 sigue pidiendo "una negrita por párrafo (excepcionalmente dos)". El balance se monitorea en batch 3.
- **Riesgo de explosión de WebSearch**: 5 patrones obligatorios pueden alargar el tiempo de generación por artículo. Aceptable dado que los fact-check fallidos del batch 2 (Halle/Báltico, Iniesta/Rijkaard) son justamente lo que más daña la credibilidad.

### 4.3 Validación

- **Smoke test post-v1.1.2 sobre un sub-rule** (similar a `smoke-v1-1-tiebreak-1779298120` para v1.1) — opcional, pero útil para detectar overcorrección antes del batch.
- **Batch QA #3 con Joel** (capturado como `EDIT-11` en backlog) sobre los 9 artículos regenerados de batch 3. Métrica esperada: pasar de 2/11 sign-off (batch 2) a ≥ 7/11 sign-off (batch 3), con los 7 patrones cross-batch reducidos a defectos puntuales — no transversales.

---

## 5. Decisiones derivadas

- **Backlog `EDIT-08` se cierra** con el feedback completo de Joel del 21/05.
- **Backlog `EDIT-10` (nuevo)**: regenerar los 9 artículos ⚠️ del batch 2 bajo style_guide v1.1.2. Excluye smoke + art 5 (sign-off ya obtenido). Sustituye conceptualmente al `EDIT-04` cerrado el 20/5.
- **Backlog `EDIT-11` (nuevo)**: batch QA #3 con Joel post-regen para validar v1.1.2 end-to-end.
- **ADR-007 (en curso)**: hero image policy — gate-by-text con aprobación humana vía Asana subtask antes de invocar gpt-image-2. Triggered por feedback paralelo de Joel (`1214965670270081`).
- **CSS theme** (no requiere ADR — fix puntual): ajuste del white space desktop entre entradilla y primer H2 + estilo de tablas `.entry-content table` (problema visual marcado por Joel en `1214965670270081`). Backup previo del CSS obligatorio.
- **Erratas factuales puntuales** (§1.4) — se documentan como reglas reusables en el patch 7 de v1.1.2; no requieren fix manual a los artículos.

---

## 6. Referencias

- `prompts/style_guide.md` v1.1.2 — la guía aplicada (commit en curso).
- Asana subtask **EDIT-08** (`1214981761607196`) — feedback original de Joel del 21/05 con los dos comentarios mañana/tarde.
- Asana task **WP visual check** (`1214965670270081`) — feedback visual paralelo con screenshots (white space + tabla art 2).
- `runs/regen-batch-v2-1779298867/` — artefactos del batch 2 que Joel revisó.
- ADR-002 (style guide v1.1), ADR-005 (templates v2 rework).
- Memoria persistente `feedback-regenerate-batch-methodology` — metodología confirmada por Pablo.
- Memoria persistente `feedback-asana-hitl-exclusive` — Asana como canal único de HITL approvals (aplica a ADR-007 hero image).
