# Template: Tactics — v2.0

**Versión 2.0** · 20/05/2026 · Rework que absorbe el defecto #11 del feedback de Joel (diferenciación real entre tipos de pieza). Ver `prompts/joel-topic-feedback-v1.md`.

## Changelog

- **v2.0 (20/05/2026)** — Reestructurado para enfatizar el "cuándo se usa / quién lo representa / qué cambió / cómo evolucionó" que diferencia táctica de regla. H3s ahora siguen un esquema fijo por ítem. Eliminado el H2 deck (cubierto por entradilla v1.1). Adaptado al ángulo no obvio para deportes tier-1.
- **v1.0 (14/05/2026)** — Versión inicial.

---

## Qué cubre este template

Artículos sobre **conceptos tácticos, técnicas, posiciones, tipologías y categorizaciones**. Tres sub-tipos:

- **Tipologías** — listado y análisis de un grupo (*tipos de tiro en fútbol*, *posiciones en rugby*, *tipos de defensa en baloncesto*).
- **Conceptos** — un concepto táctico desarrollado (*el tercer hombre en fútbol*, *el undercut en F1*, *pick and roll en baloncesto*).
- **Comparativas** — diferencias entre formatos o disciplinas (*FIBA vs NBA*, *MotoGP vs Moto2*).

## Foco editorial vs Rules vs Origins

| Eje | Tactics | Rules | Origins |
|---|---|---|---|
| Voz dominante | Análisis con anclaje visual | Explicación normativa | Narrativa con escenas |
| Pregunta central | ¿Cuándo se usa, quién lo representa, qué cambió? | ¿Qué dice la norma y qué consecuencias tiene? | ¿De dónde viene esta persona y qué la define? |
| H3s | Item-por-item con esquema fijo (ver abajo) | No habituales (excepto FAQs) | Etapas narrativas dentro de H2 |
| Ejemplos | Jugadores/equipos icónicamente asociados a la técnica | Casos reales que motivaron cambios de regla | Escenas del propio sujeto |
| Word count | 1.500-2.500 | 1.500-2.000 (pillar) · 1.300-1.700 (sub-rule) | 2.500-3.000 |

---

## URL pattern

- **Tipologías**: `/{hub_slug}/tipos-{tema}-{deporte_slug}` o `/{hub_slug}/posiciones-{deporte_slug}`
- **Conceptos**: `/{hub_slug}/{concepto}-{deporte_slug}`
- **Comparativas**: `/{hub_slug}/diferencias-{a}-vs-{b}`

Ejemplos: `/futbol/tipos-tiro-futbol`, `/futbol/tercer-hombre-futbol`, `/baloncesto/pick-and-roll-baloncesto`, `/baloncesto/diferencias-fiba-nba`, `/formula1/undercut-overcut-f1`.

## SEO title (max 60 chars)

- **Tipologías**: `Los tipos de {cosa} en {deporte}: {2 ejemplos canónicos}`
- **Conceptos**: `Qué es {concepto} en {deporte} y por qué decide partidos` o `Cómo funciona {concepto} en {deporte}`
- **Comparativas**: `{A} vs {B}: las diferencias en {deporte}`

**⚠ Para deportes tier-1**: evitar el "qué es" plano. Buscar ángulo no obvio según lineamiento L1 (evolución, dato, decisión histórica, comparativa, cambio cultural).

## Meta description (120-155 chars)

Patrón: descripción del tema + 2-3 items clave + lo que diferencia esta pieza de una explicación genérica.

---

## Estructura editorial obligatoria

```
H1
   ↓
[entradilla 30-35 palabras]
   ↓
H2 corto y periodístico (el anclaje, no la definición)
   ↓
H2: [introducción al concepto / tipología]
   ↓
H3s o H2s item-por-item (depende del sub-tipo, ver abajo)
   ↓
H2: Cómo ha evolucionado / Cómo cambió el juego
   ↓
H2: Preguntas frecuentes
   ↓
[cierre con personalidad]
```

### Esquema fijo por ítem (tipologías y comparativas)

Cada item (H3) debe contener obligatoriamente:

1. **Qué es** — 1-2 líneas de definición técnica.
2. **Cuándo se usa** — en qué contexto del partido/competición aparece.
3. **Quién lo representa** — jugador/equipo canónicamente asociado, desarrollado en una frase (no mencionado al pasar).
4. **Qué cambió** (cuando aplica) — si el item ha evolucionado en la era moderna.

Ejemplo (tipo de tiro: vaselina):
```
H3: La vaselina

[Qué es] La vaselina es un tiro corto y elevado que pasa por encima
del portero cuando este está adelantado. Se ejecuta con el empeine,
golpeando bajo el balón para imprimirle altura.

[Cuándo se usa] El delantero la elige cuando el portero sale a achicar
ángulos y queda fuera de los palos. Es un golpeo de precisión, no de
potencia: la lectura del momento manda.

[Quién lo representa] Messi ante el Betis 2019, Raúl frente al Valencia,
Totti contra el Inter en el derbi capitolino. Tres vaselinas que el
público asocia con un tipo de delantero que ve la jugada antes que el rival.

[Qué cambió] El crecimiento del juego con porteros adelantados —
herencia de Cruyff y Guardiola — multiplicó las oportunidades de
vaselina en el fútbol contemporáneo.
```

### Adaptaciones por sub-tipo

#### Tipologías (listado de items)

```
H1
[entradilla]
H2: [anclaje — el ejemplo más reconocible de la categoría]
H2: Las {N} categorías de {tema}: resumen
   [Tabla o lista compacta de los N items con descripción de 1 línea cada uno.
    Target featured snippet.]

H3: {Item 1}    [esquema fijo: qué es / cuándo / quién / qué cambió]
H3: {Item 2}    [...]
H3: {Item 3}    [...]
... [típicamente 5-10 items, no menos de 3]

H2: ¿Cuál es {el / la} {tema} más {importante / efectivo / icónico} en {deporte}?
   [Sección editorial con voz Relevo. Toma de postura razonada.]

H2: Cómo ha evolucionado {tema} en {deporte}
   [Cambio histórico — qué se hacía antes, qué cambió, por qué.]

H2: Preguntas frecuentes sobre {tema}
   [Curiosidad real, no obvia.]
```

#### Conceptos (un concepto desarrollado)

```
H1
[entradilla]
H2: [escena anclaje — un partido/jugada que ilustre el concepto]
H2: Qué es {concepto} y cómo se ejecuta
H2: Cuándo lo usan los equipos
H2: Quién lo representa: jugadores/entrenadores canónicos
   [2-4 perfiles desarrollados de quienes lo dominan]
H2: Cómo ha evolucionado {concepto} en {deporte}
   [De dónde viene, qué lo cambió, dónde está hoy]
H2: Por qué importa en el juego moderno
   [Consecuencias tácticas concretas en la era actual]
H2: Preguntas frecuentes
[Cierre]
```

#### Comparativas

```
H1
[entradilla]
H2: [anclaje — un caso reciente donde la diferencia se notó]
H2: Las diferencias principales de un vistazo
   [Tabla con 5-7 filas comparando A vs B en aspectos clave]

H2: Espacio de juego: {A} vs {B}
H2: Reglamento: {A} vs {B}
H2: Estilo de juego: {A} vs {B}
H2: Jugadores que cruzan los dos formatos
   [Casos de jugadores que han brillado/sufrido en ambos]
H2: Cómo ha cambiado la comparación
   [Convergencia o divergencia histórica reciente]
H2: Preguntas frecuentes
[Cierre]
```

---

## Sample resumido (concepto): "El tercer hombre en fútbol"

```
H1: Qué es el tercer hombre en fútbol y por qué decide partidos
[entradilla 32 palabras]
H2: La jugada que Cruyff convirtió en doctrina
   [Anclaje: una jugada concreta del Barça de Pep o la selección 2010.]
H2: Qué es el tercer hombre y cómo se ejecuta
   [Mecánica: A pasa a B, B descarga a C que ha hecho el desmarque de ruptura.]
H2: Cuándo los equipos lo buscan
   [Salida bajo presión, ataque posicional contra bloque medio, finalizaciones.]
H2: Los entrenadores y jugadores que lo institucionalizaron
   H3: Cruyff y el Dream Team
   H3: Guardiola en Manchester
   H3: Luis Enrique y la selección
   H3: Xavi y la última versión del Barça
H2: Cómo ha evolucionado el concepto
   [Del fútbol total al juego posicional contemporáneo.]
H2: Por qué importa en el fútbol moderno
   [Romper presión alta, generar superioridades, conectar con extremos.]
H2: Preguntas frecuentes
[Cierre]
```

---

## Reglas duras

- **NO** abrir con definición enciclopédica del deporte.
- **NO** listar items sin desarrollar — cada uno necesita el esquema completo "qué es / cuándo / quién / qué cambió".
- **NO** repetir las mismas estructuras sintácticas dentro de los H3s. Variar el arranque de cada ítem.
- **NO** caer en "el más importante" sin justificar. La toma de postura va respaldada por ejemplos.
- **NO** poner FAQs del tipo "qué es X" cuando X ya está explicado en el cuerpo.
- **SÍ** desarrollar 2-5 ejemplos icónicos por concepto. Las cabezas-de-cartel del deporte tienen que aparecer asociadas a la técnica que dominan.
- **SÍ** explicar la evolución histórica — es la 3ª capa que diferencia táctica de regla.

## Internal linking

- Link al hub padre.
- Link al pillar de Rules del deporte.
- Link a bios de los jugadores citados en los H3s.
- Cross-link a otros tactics del mismo deporte (típicamente 1-2).
