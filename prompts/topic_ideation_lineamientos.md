# Lineamientos para ideación de topics — Relevo

**Versión 1.0** · 20/05/2026 · Derivado del feedback editorial de Joel Sierra sobre los 148 topics seedeados el 18/05/2026 (ver `prompts/joel-topic-feedback-v1.md`).

Estos lineamientos rigen **qué** topics seedeamos al Excel inventory. Son distintos del `style_guide.md`, que rige **cómo** se redactan los artículos una vez aprobado el topic. Ambos se inyectan en el system prompt cuando aplica:

- `/relevo content-plan` → inyecta este doc + style_guide.md + el template de categoría.
- `/relevo new` → inyecta solo style_guide.md + template de categoría (el topic ya está fijado).

---

## L1. Evitar topics elementales en deportes de tier-1

En **fútbol, tenis, MMA, F1, baloncesto** (deportes con cobertura masiva ya posicionada en Google), un topic obvio del tipo *"qué es X"*, *"cuánto dura Y"*, *"cuándo se sanciona Z"* pierde contra contenido genérico que ya domina la SERP. Cada topic en estos deportes debe atacarse desde un **ángulo no obvio**.

### Ángulos preferidos para topics evergreen en deportes populares

- **Evolución de la regla** — cuándo cambió, por qué, qué generó.
  - Ejemplo: *"Cómo ha cambiado la norma del penalti para los porteros en la era reciente"* (en lugar de *"Qué es un penalti"*).
- **Dato cuantitativo** — datos comparativos que sostienen un titular.
  - Ejemplo: *"Cuánto duran los partidos de fútbol de media desde el VAR"* (en lugar de *"Cuánto dura un partido de fútbol"*).
- **Decisión histórica** — momentos clave que cambiaron la regla o el deporte.
  - Ejemplo: *"Por qué Wimbledon acabó introduciendo el tie-break en el 5º set"* (en lugar de *"Qué es un tie-break"*).
- **Comparativa entre eras o competiciones** — FIBA vs NBA, hierba vs tierra, F1 turbo vs híbrida.
- **Cambio cultural o táctico** — *"Por qué el revés a una mano se ha vuelto rareza en el tenis"*, *"Por qué el small ball cambió la NBA"*.
- **Debate contemporáneo** — *"Por qué tantas peleas se paran demasiado pronto en MMA"*, *"La regla Wenger y el espíritu del fútbol"*.

### Excepción: deportes de cobertura menor

En **rugby, balonmano, vóley, hockey patines/hierba, vela, wrestling, fútbol sala, hockey hierba/patines** y similares (donde la oferta de contenido es menor), los topics elementales **sí** tienen valor SEO. El criterio "tier-1 vs tier-2" se aplica caso a caso, pero el principio general: si Google ya tiene 50 artículos genéricos en español sobre el topic, no ataques desde el ángulo genérico.

---

## L2. Dedup obligatorio contra relevo.com

**Antes de seedear cualquier topic, verificar que no exista un artículo en relevo.com cubriendo la misma búsqueda.** Joel detectó solapamiento explícito con artículos ya publicados (ej. *ruck-maul-rugby* y *nuevas-reglas-rugby* en julio/octubre 2024).

Métodos válidos:
- **Sitemap scan offline**: descargar `https://www.relevo.com/sitemap*.xml`, grepear por keywords del topic candidato.
- **WebSearch**: `site:relevo.com "<keyword>"` por cada topic candidato.

**Acción al detectar solapamiento:**

| Caso | Acción |
|---|---|
| Solapamiento total (mismo topic, mismo ángulo) | `status=blocked` + nota: *"Cubierto por <URL>"*. No seedear. |
| Solapamiento parcial (mismo tema, ángulo distinto) | Editar el seo_title/url_slug/notes para enfatizar el ángulo nuevo. Seedear como pending_review con nota explicando la diferencia. |
| Sin solapamiento | Seedear normal. |

---

## L3. Verificación de nombres propios en bios

Toda bio requiere **validación antes de seedear**. Los modelos generativos alucinan atletas que no existen o mezclan biografías. Los errores son caros: contaminan el Excel y disparan ejecuciones de `/relevo new` que producen artículos basura.

Checklist obligatorio por cada bio antes de aceptar la fila:

1. **Existencia** — el atleta existe (cruzar con Wikipedia o fuente oficial del deporte).
2. **Apellido correcto** — sin confusiones por homofonía o transliteración (ej. *Halle* la ciudad alemana, no *Halep* la tenista).
3. **Deporte correcto** — el atleta compite en ese deporte hoy (no en un deporte adyacente o pasado).
4. **Posición / categoría correcta** — peso (MMA/boxeo), posición (deportes de equipo), especialidad (atletismo, motociclismo).
5. **Lugar de nacimiento correcto** — crítico para el ángulo de *Orígenes*.

**Homónimos**: cuando dos atletas comparten nombre, usar **segundo apellido** o disambiguación clara en el seo_title:
- ✅ *Hugo González de Oliveira* (nadador) · *Hugo González* (baloncesto).
- ❌ *Hugo González* (ambiguo en ambos casos).

**Cuando hay duda sobre la identidad**: no seedear, pedir confirmación editorial. Mejor un slot vacío que una bio errónea que vaya a producción.

---

## L4. Curaduría editorial sobre bios

Los bios deben cubrir atletas/figuras con **peso real para el lector español**:
- Hitos recientes (campeonato, fichaje, récord, salida).
- Presencia mediática en España.
- Ranking actual relevante a su deporte.
- Relevancia cultural (atleta de cantera, primera figura española en X disciplina).

**No seedear bios al azar desde listas algorítmicas o WebSearch genéricos.** Cuando hay duda → pedir lista editorial al equipo de contenido o cruzar contra una fuente de referencia (ranking oficial, presencia en prensa española reciente). Es preferible seedear 5 bios excelentes que 20 mediocres.

---

## L5. Ángulo FIFA WC 2026 cuando aplica

Para topics de **fútbol** cuya intención conecta con el Mundial 2026 (12/06–19/07), incorporar el ángulo del torneo: cambios reglamentarios anunciados por la FIFA, novedades del VAR (saques de esquina, tiempo añadido), formato 48 equipos, sedes EEUU/Canadá/México.

Esto convierte topics evergreen en **topics calientes** durante la ventana de mayor tráfico de búsqueda del año. Aplica entre **mayo y julio 2026**.

---

## Cómo se aplican estos lineamientos

### En `/relevo content-plan`

Los lineamientos L1-L5 se inyectan en el system prompt del LLM antes de la generación (ver `cmd_content_plan.md` Paso 5). El modelo los recibe junto con `style_guide.md` y el template de categoría.

### En curaduría manual

Cuando Pablo o un editor amplía el inventory manualmente (ej. la lista de 22 bios pasada por Joel el 20/05), aplica las mismas reglas:
- L1 — descartar topics elementales en tier-1.
- L2 — dedup vs relevo.com antes de añadir.
- L3 — verificar cada bio.
- L4 — priorizar atletas con peso real.

### En review editorial post-batch

Joel (o el reviewer del momento) puede usar este doc como **checklist** al revisar batches de topics seedeados. Cada fila debería pasar L1-L5 antes de moverse de `pending_review` a `pending`.

---

## Versionado

- **v1.0 (20/05/2026)** — Versión inicial. Cinco lineamientos derivados del feedback de Joel sobre los 148 topics del 18/05.

Cuando llegue feedback adicional de Joel o de futuros reviewers, se incorpora aquí con bump de versión y entry en el changelog. Igual patrón que `style_guide.md`.
