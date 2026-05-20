# Template: Origins — v2.0

**Versión 2.0** · 20/05/2026 · Rework que absorbe el defecto #11 del feedback de Joel (diferenciación real entre tipos de pieza) y los defectos #2/#4/#11 específicos de bios. Ver `prompts/joel-topic-feedback-v1.md`.

## Changelog

- **v2.0 (20/05/2026)** — Reestructurado de "biografía cronológica" a "perfil narrativo con anclajes escena". Sustituye "Dónde nació / Infancia / Trayectoria / Palmarés" rígido por bloques narrativos flexibles. Citas directas verificadas obligatorias cuando existen. Curiosidades movidas a contexto humano. FAQs reformuladas hacia curiosidad real (no edad/nacionalidad). Entradilla cubierta por style_guide v1.1.
- **v1.0 (14/05/2026)** — Versión inicial.

---

## Qué cubre este template

Perfiles biográficos de atletas y figuras del deporte. "Orígenes de" es el brand Relevo para este tipo de contenido — el ángulo es narrativo, no enciclopédico. **Highest-volume content type** del proyecto (100+ páginas previstas).

## Foco editorial vs Rules vs Tactics

| Eje | Origins | Rules | Tactics |
|---|---|---|---|
| Voz dominante | Narrativa con escenas | Explicación normativa | Análisis con anclaje visual |
| Pregunta central | ¿De dónde viene esta persona y qué la define? | ¿Qué dice la norma y qué consecuencias tiene? | ¿Cuándo se usa, quién lo representa, qué cambió? |
| Estructura | H2s narrativos por anclaje | H2s temáticos | H3s item-por-item dentro de H2s |
| Tono | Periodismo de perfil — escenas, conflicto, citas | Explicativo con casos reales | Analítico con ejemplos canónicos |
| Word count | 2.500-3.000 | 1.500-2.000 | 1.500-2.500 |

---

## URL pattern

`/{hub_slug}/{sub_hub}/origenes-{nombre_slug}`

Ejemplos:
- `/mma/ufc/origenes-ilia-topuria`
- `/tenis/atp/origenes-carlos-alcaraz`
- `/futbol/liga-primera/origenes-lamine-yamal`
- `/mas-motor/origenes-alex-palou`

## SEO title (max 60 chars)

**Lead con "Orígenes de"** — patrón de marca Relevo. Tras los dos puntos, un gancho que sitúa al lector:

- `Orígenes de {Nombre}: dónde nació y su familia`
- `Orígenes de {Nombre}: de {ciudad} al {hito definitorio}`
- `Orígenes de {Nombre}: el {gentilicio} que conquistó {liga/torneo}`

## Meta description (120-155 chars)

Patrón: una frase con el gancho identitario + 1-2 hitos + lo que el lector aprenderá.

---

## Estructura editorial obligatoria

```
H1
   ↓
[entradilla 30-35 palabras: el gancho narrativo del personaje]
   ↓
H2 corto y periodístico (la escena anclaje — un momento icónico que define al sujeto)
   ↓
[bloques narrativos en orden flexible — ver inventario abajo]
   ↓
H2: Palmarés
   ↓
H2: Preguntas frecuentes
   ↓
[cierre con personalidad]
```

### Diferencia clave vs Rules y Tactics

Las bios **no** siguen un orden cronológico rígido. Cada H2 es un **anclaje narrativo** — un tema o escena que añade dimensión al personaje. El cronograma surge por dentro de los anclajes, no como esqueleto.

✅ "El KO que lo puso en el mapa" → desde la escena UFC 298, retrocede a la migración familiar Halle-Alicante.

❌ "Dónde nació Topuria → La infancia de Topuria → La trayectoria profesional → Palmarés" (4 H2s en orden cronológico, sin escenas).

### Inventario de bloques narrativos (elige 5-8 según el sujeto)

| Bloque | Cuándo usarlo | Foco |
|---|---|---|
| **La escena que lo definió** | Casi siempre el primer H2 tras la entradilla | Un momento concreto: un KO, un gol, una pole, una llegada al podio. Desarrollado en imagen, no en datos |
| **Dónde nació y su familia** | Cuando el origen geográfico/familiar es clave (migración, saga deportiva, cantera) | Lugar + contexto cultural + familia. Sin cronología seca |
| **Cómo llegó al deporte** | Casi siempre | Primer contacto, mentores, club/academia formadora. Anécdotas, no fechas |
| **El momento bisagra** | Cuando hay un punto de inflexión claro (fichaje, lesión superada, salto a la élite) | La decisión o evento que cambió la trayectoria |
| **Su estilo: cómo juega/pelea/corre** | Casi siempre | Caracterización técnica del sujeto, qué lo hace reconocible en su deporte |
| **Personalidad pública y citas** | Cuando hay material verificable | Citas directas verificadas + lectura de carácter. Cero invento |
| **Las rivalidades / Los grandes duelos** | Cuando aplica (deportes de oposición) | Rivales recurrentes, partidos icónicos contra ellos |
| **Polémicas o sombras** | Cuando existen y son verificables | Lesiones, sanciones, declaraciones controvertidas, polémicas mediáticas. **Nunca** territorio tabloide |
| **Lo que dice de él el deporte** | Casi siempre, hacia el final | Su lugar en la historia del deporte / del país / de la disciplina |
| **Palmarés** | Siempre, como sección estructurada | Tabla con títulos, récords, premios. Verificada via WebSearch para info viva |
| **Curiosidades** | Opcional | 3-4 datos no obvios que aportan dimensión. NO obvios (idiomas que habla, mascota, hobby) |
| **Preguntas frecuentes** | Siempre | Curiosidad real, no "qué edad tiene" |

### Reglas para FAQs en Origins (las antiguas eran obvias)

**Evitar** (defecto v1.0 marcado por Joel):
- ¿Qué nacionalidad tiene X?
- ¿Cuántos años tiene X?
- ¿Dónde nació X?
- ¿Cuántos Grand Slams tiene X? (cuando el palmarés ya lo cubre)

**Preferir** preguntas que el aficionado realmente busca y que no se responden trivialmente del cuerpo:
- ¿Por qué X representa a {país} si nació en {otro país}?
- ¿Qué decía X sobre {tema relevante}?
- ¿Cómo se conocieron X y {entrenador/pareja}?
- ¿Por qué dejó X a {club/equipo anterior}?
- ¿Qué relación tiene X con {figura mediática}?
- ¿Cuál es la rivalidad entre X y {rival}?

---

## Componentes obligatorios

1. **Escenas, no fichas.** Cada bloque narrativo debe contener al menos una escena concreta (lugar, momento, gesto). Una bio sin escenas es una entrada de Wikipedia.

2. **2-5 citas directas verificadas** cuando existen. La voz del sujeto humaniza el perfil. WebSearch obligatorio para encontrarlas — **nunca inventar**.

3. **Información viva verificada antes de redactar** (palmarés, equipo actual, ranking, peso/categoría, número de hijos, pareja conocida). Sin verificación → no incluir.

4. **Disambiguación clara cuando hay homónimos.** Aplicar segundo apellido (*Hugo González de Oliveira* nadador vs *Hugo González* baloncesto) ya en H1 y SEO title.

5. **Cero territorio tabloide.** Polémicas y vida personal solo cuando son públicas, verificables y aportan al perfil. Sin especulación.

---

## Sample resumido: Carlos Alcaraz

```
H1: Orígenes de Carlos Alcaraz: el murciano que conquistó el tenis

[Entradilla 33 palabras: una escena de Wimbledon 2023 + identidad murciana.]

H2: La pista en la que aprendió a jugar
   [El club El Palmar de su padre. Carlos con cuatro años golpeando pelotas
    con una raqueta más grande que su brazo. Escena, no ficha.]

H2: De El Palmar a Villena: el camino a la academia de Ferrero
   [Llegada a la Equelite. Decisión familiar de irse a Villena. Cita
    verificada de Juan Carlos Ferrero sobre su primera impresión.]

H2: El año que cambió todo: Madrid 2022
   [Las dos victorias contra Nadal y Djokovic en una semana. La primera
    final de Masters 1000 con 19 años. Escena del Mutua Madrid Open.]

H2: Su tenis: la mezcla que nadie esperaba
   [Estilo: derecha pesada + dejada + revés a dos manos + saque mejorado.
    Comparativas con jugadores que combinan polivalencia similar.]

H2: La relación con Ferrero y la ruptura de 2025
   [Ocho años juntos. La separación profesional de diciembre 2025.
    Citas verificadas, sin especulación.]

H2: La filosofía Alcaraz: disfrutar antes que dominar
   [Su discurso público sobre la presión y el número uno. Cita verificada
    sobre los "12-15 años de carrera que le quedan". Críticas que recibió
    por esa actitud.]

H2: Palmarés
   [TABLA: Grand Slams (incluyendo Australia 2026, Roland Garros 2025,
    US Open 2025, Wimbledon 2023, US Open 2022), Masters 1000, número 1.]

H2: Curiosidades
   - Su segundo apellido es Garfia
   - Empezó jugando con una raqueta cortada para que pesara menos
   - Habla un inglés que perfeccionó en los torneos americanos
   - [...]

H2: Preguntas frecuentes
   H3: ¿Por qué Carlos Alcaraz cambió de entrenador en diciembre de 2025?
   H3: ¿Cuál es la relación entre Alcaraz y la academia Equelite?
   H3: ¿Qué decía Alcaraz sobre la presión del número uno?
   H3: ¿Por qué tantos comparan a Alcaraz con Nadal y no con Djokovic?
   H3: ¿Cómo es la rivalidad entre Alcaraz y Sinner?

[Cierre con personalidad: lo que Alcaraz representa para el tenis español post-Nadal.]
```

---

## Reglas duras

- **NO** "X es uno de los deportistas más populares del mundo" como apertura.
- **NO** fechas y datos secos sin escena. Cada bloque necesita imagen.
- **NO** inventar citas, anécdotas, lugares, fechas o relaciones. Verificación previa obligatoria.
- **NO** FAQs sobre edad, nacionalidad o lugar de nacimiento ya cubiertos en el cuerpo.
- **NO** caer en grandilocuencia ("la cuna era alemana, pero el patio fue alicantino"). Joel marcó esto explícitamente. Describir, no estilizar.
- **NO** "los entrenadores seguirán retorciendo la pizarra" y equivalentes — frases construidas para sonar literarias sin contar nada.
- **SÍ** verificar via WebSearch info viva. Si un palmarés está desactualizado, la bio pierde credibilidad de un golpe.
- **SÍ** desarrollar 2-5 escenas reconocibles por bio. Las grandes biografías deportivas se sostienen en momentos visuales que el lector recuerda.

## Internal linking

- Link al hub padre.
- Link al pillar de Rules del deporte (cuando aplica).
- Link a tactics que el sujeto domina.
- Link a 2-3 bios de otros atletas del mismo hub ("como sucedió con {otro}…").
- Link a las competiciones donde compite (subhub o liga).
- Cross-hub solo cuando el sujeto realmente cruza disciplinas (boxeador que pasó al MMA, etc.).

## Adaptaciones por deporte (información estructurada en el palmarés)

| Deporte | Datos críticos en tabla palmarés |
|---|---|
| Fútbol | Trofeos club, copas selección, goles totales, premios individuales |
| Tenis | Grand Slams, Masters 1000, semanas como nº 1, Copa Davis |
| MMA / Boxeo | Récord profesional (W-L-D), categoría, títulos por organización |
| Fórmula 1 / Motociclismo / Más motor | Campeonatos, victorias, podios, poles, vueltas rápidas, equipos por temporada |
| Atletismo | Marcas personales, medallas olímpicas / mundiales, récords nacionales |
| Baloncesto | Estadísticas NBA/ACB, anillos, MVPs, all-star selections |
| Pádel | Títulos WPT/Premier, finales, semanas nº1, partners principales |
| NFL | Super Bowls, MVPs, Pro Bowls, estadísticas QB/RB/WR según posición |
| Rugby | Internacionalidades, mundiales, posición fija |
| Ciclismo | Grand Tours (general/etapas), Monumentos, mundiales en línea/crono |
