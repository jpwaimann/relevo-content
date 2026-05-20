# Template: Rules — v2.0

**Versión 2.0** · 20/05/2026 · Rework que absorbe el defecto #11 del feedback de Joel (diferenciación real entre tipos de pieza). Ver `prompts/joel-topic-feedback-v1.md` para contexto.

## Changelog

- **v2.0 (20/05/2026)** — Estructura reworked: entradilla periodística separada de H2 deck (ya cubierta por style_guide v1.1); ejemplos reales obligatorios; sección "Por qué importa hoy" (3ª capa); "Polémicas y debates" opcional cuando aplica; FAQs de curiosidad. Diverge de Tactics y Origins en foco editorial.
- **v1.0 (14/05/2026)** — Versión inicial.

---

## Qué cubre este template

Artículos sobre **reglamento y conceptos normativos** de un deporte. Dos sub-tipos:

- **Pillar Rules** — el reglamento global del deporte (ej. *"Reglas del fútbol"*).
- **Sub-rule** — una regla específica desarrollada (ej. *"El fuera de juego"*, *"El VAR en saques de esquina"*).

## Foco editorial vs Tactics vs Origins

| Eje | Rules | Tactics | Origins |
|---|---|---|---|
| Voz dominante | Explicación normativa con anclaje en aplicación | Análisis táctico con anclaje en ejemplos visuales | Narrativa con escenas y citas directas |
| Estructura | H2s temáticos (cuerpo, espacio, sanciones, polémicas) | H3s item-by-item dentro de pocos H2s | H2s narrativos (anclajes biográficos, no cronológicos rígidos) |
| Ejemplos | Reales recientes + casos icónicos que motivaron cambios de regla | Jugadores/equipos canónicos asociados a cada técnica | Escenas concretas del propio sujeto + citas verificadas |
| 3ª capa obligatoria | Por qué importa hoy (consecuencias tácticas o polémicas vivas) | Cómo ha evolucionado y qué la cambió | Qué dice del deporte / personalidad pública |
| Word count | 1.500-2.000 (pillar) · 1.300-1.700 (sub-rule) | 1.500-2.500 | 2.500-3.000 |

---

## URL pattern

- **Pillar**: `/{hub_slug}/reglas-{deporte_slug}`
- **Sub-rule**: `/{hub_slug}/{concepto-especifico}-{deporte_slug}`

Ejemplos: `/futbol/reglas-futbol`, `/futbol/fuera-de-juego-futbol`, `/tenis/wimbledon-tie-break-quinto-set`, `/mma/categorias-peso-ufc`.

## SEO title (max 60 chars)

- **Pillar**: `Reglas de {deporte}: normativa completa y claves del juego`
- **Sub-rule (evolución de regla)**: `Por qué {decisión} en {deporte}` o `Cómo ha cambiado {regla} en {deporte}`
- **Sub-rule (dato cuantitativo)**: `Cuánto dura un partido de {deporte} de media desde {evento}`
- **Sub-rule (qué es — solo en deportes minoritarios)**: `Qué es {concepto} en {deporte} y cómo funciona`

**⚠ Para deportes tier-1 (fútbol, tenis, baloncesto, MMA, F1)**: NO usar el patrón "qué es" salvo que el concepto no esté ya cubierto en relevo.com. Aplicar lineamiento L1 de `topic_ideation_lineamientos.md`.

## Meta description (120-155 chars)

Patrón: contexto del topic + 2-3 ganchos concretos + lo que el lector aprenderá.

---

## Estructura editorial obligatoria

```
H1
   ↓
[entradilla 30-35 palabras — cubierta por style_guide v1.1]
   ↓
H2 corto y periodístico
   ↓
[cuerpo del artículo]
   ↓
H2: Preguntas frecuentes
   ↓
[cierre con personalidad]
```

### Componentes obligatorios del cuerpo

1. **Anclaje en lo concreto al inicio.** El primer H2 abre con escena, dato o decisión específica — no con una definición general del deporte. Para sub-rules de evolución: el partido o caso que motivó el cambio. Para pillar: la imagen más reconocible del reglamento (un saque de Wimbledon, un saque de banda controvertido, etc.).

2. **Tres capas para cada regla principal del artículo:**
   - **Qué es** — definición clara y compacta.
   - **Cómo funciona** — mecánica real, casos de aplicación, excepciones.
   - **Por qué importa hoy** — qué cambió en el deporte, qué debate genera, qué consecuencias tácticas tiene. **Esta capa es la que diferencia un artículo Relevo de una entrada de Wikipedia.**

3. **2-5 ejemplos icónicos desarrollados.** No mencionar al pasar. Cada ejemplo necesita 1-2 líneas que sitúen el caso (cuándo, dónde, quién). Ver style_guide v1.1 sección "Ejemplos icónicos".

4. **Sección opcional "Polémicas y debates"** cuando la regla genera discusión viva (VAR, fuera de juego semiautomático, paradas tempranas en MMA, ley Wenger, penalti adelantado del portero, etc.).

5. **FAQs de curiosidad real, no obvias.** Ver style_guide v1.1 sección "FAQs".

### Adaptaciones por sub-tipo

#### Pillar Rules (reglamento global)

H2s típicos en este orden, adaptados al deporte:
1. Reglas básicas (qué se gana, cuántos jugadores, estructura)
2. Las reglas más importantes (lista de 5-8 reglas en una sola línea cada una; targets featured snippet)
3. {Espacio de juego} y sus dimensiones (campo / pista / ring / octógono / circuito)
4. Cuánto dura un partido (formato, tiempos, prórroga)
5. {Sistema de puntuación / Cómo se gana}
6. {Faltas / Sanciones / Penalizaciones}
7. {Posiciones / Categorías} (solo si aplica)
8. **Polémicas y debates actuales** (cuando hay)
9. Preguntas frecuentes

#### Sub-rule (regla específica desarrollada)

H2s típicos en este orden:
1. **Escena o caso anclaje** (el partido / decisión / momento que motivó el cambio o que mejor ilustra la regla)
2. **Cómo se contaba antes / cómo era el problema** (contexto histórico)
3. **El cambio, decisión o mecanismo actual** (la regla en sí, con sus matices)
4. **Cómo se aplica hoy** (2-3 ejemplos contemporáneos)
5. **Polémicas y debates** (cuando hay)
6. **Qué cambió en el juego** (consecuencias tácticas o culturales)
7. **Preguntas frecuentes**

---

## Sample (sub-rule de evolución — abreviado)

Título: *"Por qué Wimbledon introdujo el tie-break en el quinto set"*

```
H1: Por qué Wimbledon introdujo el tie-break en el quinto set

[Entradilla 35 palabras situando la decisión.]

H2: La final que duró tres días
  [Escena Isner-Mahut 2010 desarrollada en 2-3 párrafos.
   La grieta en el reglamento. La pista 18 saturada.]

H2: Cómo se contaba el quinto set hasta 2018
  [El sistema histórico. Argumento conservador vs argumento de cambio.
   Caso adicional: las semifinales Wimbledon 2018 acumuladas.]

H2: Por qué Wimbledon eligió el 12-12 y no el 6-6
  [La decisión de octubre 2018. Razonamiento del club.
   Comparativa con AO (super tie-break) y US Open (tie-break al 6-6 desde 1970).]

H2: El día que los cuatro grandes se unificaron
  [Marzo 2022. Formato común. Presión de televisión + calendario.]

H2: Qué cambió en el juego cuando llegó el cambio
  [Tres efectos tácticos concretos: sacadores, mental, aficionados.
   Carlos Alcaraz, Jannik Sinner ganando super tie-breaks recientes.]

H2: Preguntas frecuentes sobre el tie-break en Wimbledon
  H3: ¿Cuándo aplicó Wimbledon por primera vez tie-break en el quinto set?
  H3: ¿Por qué Wimbledon no usó el tie-break antes?
  H3: ¿En qué se diferencia el tie-break del super tie-break?
  H3: ¿Qué pasó con el récord de Isner y Mahut tras el cambio?
  H3: ¿Aplican los cuatro Grand Slams el mismo formato?

[Cierre con personalidad: la decisión que el partido más largo llevaba pidiendo desde 2010.]
```

Sample completo trabajado en `runs/smoke-v1-1-tiebreak-1779298120/article.html` (smoke test v1.1, mayo 2026).

---

## Reglas duras

- **NO** abrir el artículo con "El X es uno de los deportes más populares del mundo".
- **NO** repetir la keyword principal más de 4-5 veces en el cuerpo.
- **NO** caer en pillar-template para sub-rules. Un sub-rule sobre el VAR no necesita las dimensiones del campo ni la duración del partido.
- **NO** poner "qué es" como primer H2 en deportes tier-1 (regla L1).
- **NO** terminar con "el {deporte} es un deporte fascinante" o equivalente. El cierre tiene que aportar.
- **SÍ** verificar via WebSearch toda info viva antes de redactar (rosters, marcadores, fechas de cambios reglamentarios, palmarés citados).
- **SÍ** desarrollar entre 2 y 5 ejemplos icónicos por artículo, no solo mencionarlos.

## Internal linking

- Link al hub padre.
- Link a sub-rules relacionados (desde un pillar, linkear a todos los sub-rules del mismo deporte).
- Link a bios de jugadores famosos por momentos asociados a esa regla.
- Link a tactics que dependen de esa regla.
- Cross-sport sólo cuando la mecánica realmente se transfiere.
