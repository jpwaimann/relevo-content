# Feedback Joel — Topic ideation v1 (mayo 2026)

**Origen.** Segunda ronda de feedback editorial de Joel Sierra, esta vez sobre la **ideación** (los 148 topics seedeados el 18/05/2026 en el Excel `Relevo - Content - Article production DB.xlsx`). Complementa el [feedback v1 sobre redacción](joel-feedback-v1.md) que cubrió los 11 artículos del batch QA #1.

**Canal.** Recibido por Pablo via Teams el 20/05/2026 PM (no Asana). Se archiva en Asana como subtask de Phase 3 para trazabilidad.

**Audiencia.** Este doc tiene dos lecturas:
- **Tactical items** (sección "Fixes específicos") — acciones puntuales sobre filas concretas del inventory. Se aplican una vez y desaparecen del flujo.
- **Policy items** (sección "Lineamientos durables") — principios que afectan toda futura ideación. Quedan codificados en `.claude/skills/relevo/lib/cmd_content_plan.md` para que cualquier ejecución de `/relevo content-plan` los respete.

---

## Lineamientos durables — codificados en `cmd_content_plan.md`

Estos son los principios generales que extraemos del feedback y que aplican a toda futura ideación, independientemente del deporte o del batch.

### L1. Evitar topics elementales en deportes de tier-1

En **fútbol, tenis, MMA, F1, baloncesto** (deportes con cobertura masiva ya posicionada en Google), un topic obvio del tipo *"qué es X"*, *"cuánto dura Y"*, *"cuándo se sanciona Z"* pierde contra contenido genérico que ya domina la SERP. Cada topic en estos deportes debe atacarse desde un ángulo no obvio.

**Ángulos preferidos para topics evergreen en deportes populares:**
- Evolución de la regla — cuándo cambió, por qué, qué generó.
- Dato cuantitativo — cuánto duran los partidos de media post-VAR, % de penaltis transformados.
- Decisión histórica — por qué Wimbledon introdujo el tie-break en el 5º set.
- Comparativa entre eras o competiciones.
- Cambio cultural o táctico — por qué desapareció el revés a una mano.

**Excepción**: en deportes con menos competencia SEO (rugby, balonmano, vóley, hockey patines/hierba, vela, wrestling) los topics elementales **sí** tienen valor porque la oferta de contenido es menor. La regla se aplica solo a tier-1.

### L2. Dedup obligatorio contra relevo.com

Antes de seedear cualquier topic nuevo, verificar que **no exista ya un artículo en relevo.com** cubriendo la misma búsqueda. Métodos válidos:

- Sitemap scan offline (`https://www.relevo.com/sitemap*.xml`) → grep por keyword.
- `site:relevo.com "<keyword>"` vía WebSearch.

Si hay solapamiento claro → status `blocked` + nota: *"Cubierto por [URL]"*. Si el ángulo es distinto pero el tema se solapa parcialmente → editar el topic para enfatizar el ángulo nuevo.

### L3. Verificación de nombres propios en bios

Toda bio requiere **validación antes de seedear**:
- Existencia del atleta (no inventar).
- Apellido correcto.
- Deporte correcto.
- Posición / categoría correcta.
- Lugar de nacimiento correcto.

**Crítico en homónimos** — usar segundo apellido o disambiguación clara (ej: *Hugo González de Oliveira* nadador vs *Hugo González* baloncesto). Cuando hay duda sobre identidad → no seedear; pedir confirmación editorial.

### L4. Curaduría editorial sobre bios

Los bios deben cubrir atletas/figuras con **peso real para el lector español**: hitos recientes, presencia mediática, ranking actual, relevancia en su deporte. No seedear bios al azar desde listas algorítmicas. Cuando hay duda → pedir lista editorial al equipo de contenido o cruzar contra una fuente de referencia (ranking, presencia en prensa española reciente).

### L5. Ángulo de FIFA WC 2026 cuando aplica

Para topics de fútbol cuya intención puede conectarse con el Mundial 2026 (12/06–19/07), incorporar el ángulo: cambios reglamentarios anunciados por la FIFA para el torneo, novedades del VAR, formato 48 equipos, sedes EEUU/Canadá/México. Convierte topics evergreen en topics calientes durante la ventana del Mundial.

---

## Fixes específicos — aplicar al inventory actual

### F1. Topics elementales a reformular (3 filas)

Joel identificó 3 topics flojos. En lugar de borrar y reseedear, editar el seo_title, url_slug y notes in-place.

| Topic actual | Reformulación |
|---|---|
| Cuánto dura un partido de fútbol: tiempos y prórrogas | **Cuánto duran los partidos de fútbol de media desde la llegada del VAR** (data-driven, descontento creciente con tiempos añadidos infinitos) |
| Penalti en fútbol: cuándo se pita y cómo se ejecuta | **Cómo ha cambiado la norma del penalti para los porteros en la era reciente** (evolución de la regla, polémica del pie en la línea, casos icónicos) |
| Qué es un tie-break y cómo se juega en tenis | **Por qué Wimbledon acabó introduciendo el tie-break en el quinto set** (decisión histórica 2019, Isner vs Mahut 2010 como detonante; ref: rtve.es/deportes/20181019/wimbledon-introduce-tie-break-quinto-set/1822681.shtml) |

Nota: el topic de penalti ya tiene un artículo publicado en WP (post 16100). Al reformular el topic en el inventory, el artículo viejo queda obsoleto y entra en el flujo de regeneración (EDIT-04 del backlog).

### F2. Imprecisiones factuales en bios (4 filas)

| Bio | Error | Fix |
|---|---|---|
| Ilia Topuria | "Halep a Madrid" | "Halle a Madrid" (Halle, Alemania, ciudad natal — Halep es la tenista, error obvio) |
| Lin (portero chino-español campeón del mundo) | El personaje **no existe** | Reescribir como **Lin (Adolfo Fernández Olivera)**, ala de fútbol sala, segoviano, ex-Barça, internacional con España. Mover de Fútbol → Fútbol Sala. https://es.wikipedia.org/wiki/Lin |
| Marc Márquez | "se inicia en Moto2" | "se inicia en 125cc antes de Moto2" |
| Hugo González (nadador) | Apellido único ambiguo | Renombrar a **Hugo González de Oliveira** para diferenciar del jugador de baloncesto Hugo González también seedeado |

### F3. Dedup check pendiente

Cruzar el inventory completo (148 filas) contra relevo.com. Joel citó 2 ejemplos de rugby ya publicados:
- https://www.relevo.com/rugby/ruck-maul-rugby-20241030224832-nt.html
- https://www.relevo.com/rugby/nuevas-reglas-rugby-agitan-verano-20240811225404-nt.html

Todo topic del inventory que solape con uno publicado → status `blocked` + nota.

### F4. Topics nuevos a seedear (4 angle topics)

| Topic | Deporte | Categoría | Ángulo |
|---|---|---|---|
| El concepto del tercer hombre en fútbol | Fútbol | tactics | Mecanismo táctico para romper líneas; referentes Cruyff/Guardiola |
| Por qué España y Argentina dominan el pádel a nivel mundial | Pádel | tactics / análisis | Factores: club system, escuelas, dimensión cultural |
| Por qué el revés a una mano se ha vuelto rareza en el tenis | Tenis | tactics | Cambio cultural-técnico; comparativa Federer/Wawrinka vs era moderna |
| Introducción del VAR en los saques de esquina en el Mundial 2026 | Fútbol | rules | Cambio reglamentario FIFA WC 2026 |

### F5. Bios nuevos a seedear (~22 atletas)

Lista completa de Joel, ya mapeada a los 24 deportes del sistema.

| Atleta | Deporte | Notas |
|---|---|---|
| Rafa Jódar | Tenis | |
| Paula Badosa | Tenis | |
| Jessica Bouzas | Tenis | |
| Kaitlin Quevedo | Tenis | |
| Juan Lebrón | Pádel | |
| Paquito Navarro | Pádel | |
| Gemma Triay | Pádel | |
| Bea González | Pádel | |
| Ana Peleteiro | Atletismo | |
| Izan Almansa | Baloncesto | |
| Usman Garuba | Baloncesto | |
| Santi Aldama | Baloncesto | |
| Joel Álvarez | MMA | |
| Jorge Martín | Motociclismo | |
| Máximo Quiles | Motociclismo | |
| Manu González | Motociclismo | |
| José Antonio Rueda | Motociclismo | |
| David Alonso | Motociclismo | |
| Enric Mas | Ciclismo | |
| Markel Beloki | Ciclismo | |
| Colin Kaepernick | NFL | |
| Tom Brady | NFL | |
| Álex Palou | **(blocked)** | IndyCar — no hay deporte exacto en los 24. Pedir decisión a Pablo: (a) clasificar bajo Fórmula 1 con nota, (b) ampliar lista de deportes con IndyCar/automovilismo, (c) descartar |

---

## Validación post-fix

Después de aplicar F1–F5:
1. **Auditar el inventory**: confirmar 148 + 4 angle + 22 bios = 174 filas (menos los marcados `blocked` por dedup o por Palou).
2. **Revisar reformulaciones (F1)**: que los 3 ángulos nuevos cuajen con la voz Relevo (sin caer en otro topic obvio).
3. **Pedir aprobación de Joel**: una nueva ronda de QA sobre el inventory actualizado antes de cualquier `/relevo new` a escala.

---

## Conecta con

- [`joel-feedback-v1.md`](joel-feedback-v1.md) — feedback v1 sobre redacción (el otro eje editorial).
- [`style_guide.md`](style_guide.md) — v1.1, que ya recoge varios principios alineados con este feedback.
- `.claude/skills/relevo/lib/cmd_content_plan.md` — donde viven los lineamientos L1–L5 para futuras ideaciones.
- `docs/adr/002-style-guide-v1-1.md` — ADR anterior, mismo patrón decisorio.
