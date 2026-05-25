# Template: Coaches — v1.0

**Versión 1.0** · 25/05/2026 · Versión inicial. Adapta la base "Coach Era & Trophies" provista por el equipo editorial de Relevo (SharePoint: `seoberlin/ES MARKET/Relevo.com/Content/Relevo - Coach Era & Trophies [Base Template].docx`) al patrón Relevo definido por `origins_template.md` v2.0. Documentado en ADR-011.

## Changelog

- **v1.0 (25/05/2026)** — Versión inicial. 4ta categoría de contenido tras Rules, Tactics y Origins. Hereda style guide v1.2 (25 puntos de self-check, tests pasa/falla, listas verificables) y la disciplina antialucinación. Estructura: 3 párrafos de intro (era setting) → H3 narrativa cronológica de trofeos → H3 palmarés como bullet list → H3 opcional jugador-entrenador → H2 FAQs.

---

## Qué cubre este template

Páginas de **enumeración de títulos de un entrenador en un equipo específico**. "Los títulos que ha ganado {equipo} con {entrenador} al frente" es el brand Relevo para este tipo de contenido — el ángulo es **enumerativo-narrativo**: la lista de trofeos es la columna vertebral, pero **la identidad de la era es la que carga la pieza** (cholismo de Simeone, tiki-taka de Guardiola en el Barça, mourinhismo en Inter, heavy-metal football de Klopp). Aplica a:

| Deporte | Volumen esperado | Adaptación |
|---|---|---|
| **Fútbol (club)** | Alto | Standard. Sección doble rol si el entrenador también fue jugador del mismo club |
| **Fútbol (selección)** | Medio | `{equipo}` = selección nacional. Sin sección doble rol. Añadir framing de internacionalidades si jugó para el país |
| **Baloncesto (NBA/ACB)** | Medio | Sustituir "Ligas" por anillos NBA / títulos ACB. Añadir contador "Finales" (ganadas vs perdidas) — central a las narrativas NBA |
| **F1** | Bajo | Sustituir "entrenador" por "team principal". Trofeos = Constructores + Pilotos. Atribución por piloto importa |
| **Rugby** | Bajo | Seis Naciones / Champions Cup / liga doméstica. Volumen menor pero alta búsqueda dentro del hub rugby |
| **Ciclismo** | Marginal | Solo cuando la figura tiene impacto multi-año (Brailsford en Sky/Ineos). Raro |

**NO aplica** a deportes individuales (tenis, atletismo, MMA, golf): ahí la categoría correcta es Origins o un format distinto.

## Foco editorial vs Rules vs Tactics vs Origins

| Eje | Coaches | Rules | Tactics | Origins |
|---|---|---|---|---|
| Voz dominante | Narrativa enumerativa con era anclaje | Explicación normativa | Análisis con anclaje visual | Narrativa con escenas |
| Pregunta central | ¿Qué títulos ganó X con Y y cómo se llamó la era? | ¿Qué dice la norma y qué consecuencias tiene? | ¿Cuándo se usa, quién lo representa, qué cambió? | ¿De dónde viene esta persona y qué la define? |
| Estructura | Intro 3 párrafos + walk cronológico + palmarés bullet | H2s temáticos | H3s item-por-item dentro de H2s | H2s narrativos por anclaje |
| Tono | Crónica de era — antes/después, identidad, hitos | Explicativo con casos reales | Analítico con ejemplos canónicos | Periodismo de perfil — escenas, conflicto, citas |
| Word count | 800-1.200 | 1.500-2.000 | 1.500-2.500 | 2.500-3.000 |

Nota: este es el formato **más corto** del proyecto. El instinto de inflar a 2.500 palabras es incorrecto — el lector busca la enumeración, la narrativa la enmarca, no la dilata.

---

## URL pattern

`/{hub_slug}/{sub_hub}/titulos-ganados-{equipo_slug}-{entrenador_slug}`

Ejemplos canónicos:

- `/futbol/liga-primera/titulos-ganados-atletico-madrid-simeone`
- `/futbol/liga-primera/titulos-ganados-real-madrid-ancelotti`
- `/futbol/liga-primera/titulos-ganados-barcelona-guardiola`
- `/futbol/premier-league/titulos-ganados-manchester-city-guardiola`
- `/futbol/premier-league/titulos-ganados-chelsea-mourinho`
- `/futbol/liga-italiana/titulos-ganados-inter-mourinho`
- `/baloncesto/nba/titulos-ganados-warriors-kerr`
- `/baloncesto/acb/titulos-ganados-real-madrid-pablo-laso`

## SEO title (max 60 chars)

Dos fórmulas — usar siempre la **interrogativa o cuantitativa**, evitar el "palmarés completo de…" plano:

- `Títulos de {entrenador} con {equipo}: ¿cuántos lleva?`
- `¿Cuántos títulos ha ganado el {apodo} con {equipo}?`

Ejemplos:

| Entrenador (equipo) | SEO title | Chars |
|---|---|---|
| Simeone (Atlético) | Títulos de Simeone con el Atlético: ¿cuántos lleva? | 52 |
| Ancelotti (R. Madrid) | ¿Cuántos títulos ha ganado Ancelotti con el Real Madrid? | 56 |
| Guardiola (Barça) | Títulos de Guardiola en el Barça: la era del sextete | 53 |
| Mourinho (Inter) | Títulos de Mourinho con el Inter: el histórico Triplete | 56 |
| Guardiola (Man City) | ¿Cuántos títulos lleva Guardiola con el Manchester City? | 56 |
| Kerr (Warriors) | Títulos de Steve Kerr con los Warriors: [Anillos NBA] | 54 |

**Estrategia CTR**: abrir con pregunta de intención directa (`¿cuántos lleva?`, `¿cuántos ha ganado…?`) en vez de etiqueta plana. Corchetes opcionales (`[Año]`, `[Récord]`) si suman foco visual en SERP.

## Meta description (120-155 chars)

Fórmula: pregunta + promesa cuantitativa + lenguaje punchy.

`¿Qué títulos ganó {entrenador} con {equipo}? Descubre el palmarés de la era {identidad_era}: {total_titulos} campeonatos, finales históricas y datos clave.`

Ejemplos:

- `¿Qué títulos ganó Simeone con el Atlético? Descubre el palmarés completo del 'cholismo': 8 campeonatos históricos, finales de Champions y datos clave.` (151 chars)
- `¿Cuántos títulos lleva Ancelotti con el Real Madrid? Desglosamos el palmarés del italiano: Champions, Ligas y el récord histórico en el Bernabéu.` (149 chars)
- `La era dorada: ¿Qué títulos ganó Guardiola con el Barça? Revive el palmarés del 'tiki-taka' con sus 14 trofeos, el sextete y su huella histórica.` (148 chars)

---

## Variables del template

| Variable | Significado | Notas |
|---|---|---|
| `{entrenador}` | Nombre completo (Diego Simeone, Carlo Ancelotti, Pep Guardiola) | Primera mención completo, después apellido solo |
| `{entrenador_slug}` | Slug URL | `simeone`, `ancelotti`, `guardiola` |
| `{entrenador_apodo}` | Apodo público (Cholo, Mou, Pep) | Opcional — solo si es parte de la narrativa pública |
| `{equipo}` | Nombre del club o selección (Atlético de Madrid, Barcelona) | Primera mención completo |
| `{equipo_articulo}` | "el" / "la" / "" (el Atlético, el Barcelona, la Roma) | Crítico para concordancia gramatical |
| `{equipo_slug}` | Slug URL del equipo | `atletico-madrid`, `real-madrid`, `barcelona` |
| `{hub_slug}` | Hub deportivo (futbol, baloncesto, mas-motor) | — |
| `{sub_hub}` | Liga / competición (liga-primera, premier-league, nba, acb) | — |
| `{deporte}` | Nombre del deporte | Para framing narrativo |
| `{era_inicio}` | Temporada de inicio (2011/12) | Formato slash siempre |
| `{era_fin}` | Temporada de fin (2020/21) o `actualidad` si sigue en el cargo | — |
| `{total_titulos}` | Total de trofeos ganados en la era | Numérico |
| `{total_club_historico}` | Total de trofeos en la historia del club | Para framing "X de Y" |
| `{identidad_era}` | Nombre / brand de la era (cholismo, tiki-taka, heavy-metal football) | Crítico — es el ancla narrativo |
| `{predecesor}` | Entrenador anterior despedido (opcional) | Solo si la salida importa narrativamente |
| `{evento_destitucion}` | Trigger de la destitución del predecesor (opcional) | Ej. "el batacazo de copa contra el Albacete" |
| `{rival_clasico}` | Rival principal del club | Real Madrid para Atleti, Barcelona para Real Madrid |
| `{rol_doble}` | "jugador" / "futbolista" | Solo si el entrenador también fue jugador del mismo club |

---

## Estructura editorial obligatoria

```
H1: Los títulos que ha ganado {equipo_articulo} {equipo} con {entrenador} al frente

[Intro block — 3 párrafos, sin H2/H3 — instala la era]
   Párrafo 1 (antes/después): un dato duro de contraste (participaciones en Champions antes vs después, finales ligueras, share de títulos sobre la historia del club). Pick the most striking.
   Párrafo 2 (estado del entrenador hoy): franja salarial, reconocimiento de la directiva, quién es la "cara" del proyecto (entrenador o jugadores).
   Párrafo 3 (identidad de la era): nombrar {identidad_era}, describir la filosofía, cómo la afición la adoptó, la cultura de resultados del club. Cita de institución cuando aplica ("Ganar y ganar y volver a ganar" para el Atleti, "Més que un club" para el Barça).

H3: Estos son los títulos de {entrenador} con {equipo_articulo} {equipo}
   Frame line obligatoria: "Con {entrenador}, {equipo_articulo} {equipo} ha conseguido {total_titulos} de los {total_club_historico} títulos que ha levantado a lo largo de su historia."
   Origin story (1-2 frases): cómo llegó el entrenador. ¿Sustitución a mitad de temporada? ¿Reacción tras {evento_destitucion}? ¿Proyecto planificado?
   [Walk cronológico] Cada trofeo recibe: competición + temporada + resultado de la final (rival + score) + goleadores / momentos clave si es narrativamente útil.
   [Finales perdidas] Las finales perdidas que definen la era reciben el mismo tratamiento que las ganadas. (Ejemplo: las dos finales de Champions 2014 y 2016 son centrales a la historia de Simeone — no se omiten.)
   Internal linking generoso (ver reglas abajo).

H3: Palmarés de {entrenador} como entrenador {de/del} {equipo}
   Bullet list. Formato: N {Competicion} ({temporada1} y {temporada2}).
   Ejemplo: 2 Ligas (2013/14 y 2020/21).
   Orden: por prestigio descendente (Liga / Champions / Copa / Europa League / Supercopas).

[OPCIONAL — solo si el entrenador también fue jugador del mismo club]
H3: Los títulos de {entrenador} como {rol_doble} {de/del} {equipo}
   Narrativa breve (1-2 párrafos): de dónde venía, temporada clave, trofeo signature, salida.

H3: Palmarés de {entrenador} como {rol_doble} {de/del} {equipo}
   Bullet list. Mismo formato que arriba.

H2: Preguntas frecuentes sobre {entrenador} en {equipo}
   (Activa el schema FAQPage)
   H3: ¿Cuántos títulos ha ganado {entrenador} con {equipo_articulo} {equipo}?
   H3: ¿Cuál fue el primer título de {entrenador} con {equipo_articulo} {equipo}?
   H3: ¿Cuántas Ligas ha ganado {entrenador} con {equipo_articulo} {equipo}?
   H3: ¿Ha ganado {entrenador} la Champions League con {equipo_articulo} {equipo}? (o la competición continental top que aplique)
   H3: ¿Cuándo llegó {entrenador} a {equipo_articulo} {equipo}?
   Cada FAQ se responde en 2-3 frases.

[Cierre narrativo — opcional, no H2 con "Cierre"/"Conclusión"/"En definitiva"]
   Un párrafo breve que cierre la era (proyección o legado).
   Lleva una negrita que pase el test del valor scan (style guide v1.2 §17).
   Sin conectores prefabricados ("en suma", "como hemos visto", "es solo el comienzo").
```

### Diferencia clave vs Origins

Origins **NO** sigue orden cronológico rígido y arma con anclajes-escena. **Coaches SÍ** sigue orden cronológico para la columna vertebral (la lista de trofeos camina año a año), pero la **narrativa que la enmarca es de era** (no de momento puntual). El equilibrio: cronología en los hitos, identidad de era en la voz.

✅ "En 2013/14 el Atlético levantó la Liga tras 18 años sin ganarla. Aquel cholismo todavía sin nombre se construía sobre el bloque medio y la presión coordinada." → cronología (2013/14) + identidad de era (cholismo).

❌ "Simeone es uno de los entrenadores más exitosos del Atlético." → genérico, sin escena ni dato.

---

## Componentes obligatorios

1. **Antes/después con dato duro** en el primer párrafo. No vale "X transformó al equipo". Vale: "Antes de Simeone, el Atlético acumulaba 4 participaciones en Champions; bajo su dirección ha disputado 12 en 14 temporadas."

2. **Nombrar la era**. Si la era no tiene nombre público consolidado (`cholismo`, `tiki-taka`, `mourinhismo`), nombrarla descriptivamente en el párrafo 3 ("el bloque-presión coordinado de Simeone", "el ciclismo de posesión del Guardiola del Barça"). Sin nombre de era, el artículo pierde su anclaje.

3. **Trofeos en orden cronológico, no por competición**. La narrativa va año a año, no compitiendo Ligas con Champions agrupadas. La lista bullet del palmarés sí va por prestigio descendente, pero la narrativa camina cronológica.

4. **Finales perdidas que definen la era**. Las dos finales de Champions de Simeone (2014, 2016), la final perdida de Guardiola en 2024, etc. Se cuentan con el mismo nivel de detalle que las ganadas cuando son centrales a la era.

5. **Información viva verificada**. WebSearch obligatorio para: total de títulos a la fecha actual (sobre todo si el entrenador sigue en el cargo), última temporada disputada, rival de la final con su score exacto, total de títulos históricos del club. Si el dato no está confirmado, **omitir** antes que afirmar incorrecto.

6. **Cita de institución del club cuando aplica**. "Ganar y ganar y volver a ganar" (Atleti), "Més que un club" (Barça), "Hala Madrid y nada más" (Real Madrid). Refuerza el framing de era sin invento.

7. **Schema markup**: Article + Person (el entrenador) + SportsTeam (el equipo) + FAQPage. El plugin del theme lo emite por default si la estructura del artículo es correcta — verificar con el helper de FAQ JSON-LD para el bloque H2/H3 de preguntas.

---

## Reglas para FAQs en Coaches

5 FAQs estándar, todas activan FAQPage schema:

1. **¿Cuántos títulos ha ganado {entrenador} con {equipo_articulo} {equipo}?**
2. **¿Cuál fue el primer título de {entrenador} con {equipo_articulo} {equipo}?**
3. **¿Cuántas Ligas ha ganado {entrenador} con {equipo_articulo} {equipo}?** (sustituir "Ligas" por la competición doméstica top del deporte/país)
4. **¿Ha ganado {entrenador} la Champions League con {equipo_articulo} {equipo}?** (o la continental top que aplique — Euroleague para baloncesto, Constructors para F1, etc.)
5. **¿Cuándo llegó {entrenador} a {equipo_articulo} {equipo}?**

Cada respuesta: 2-3 frases. Si el cuerpo del artículo ya responde la FAQ con detalle, la FAQ resume con el dato puntual (no repite el desarrollo).

---

## Internal linking — obligatorio

Cada página coach-trophies enlaza a:

| Destino | En qué parte del artículo |
|---|---|
| Hub del club (`/futbol/equipos/{equipo_slug}/`) | Párrafo 1 del intro |
| Página Origins del entrenador (`/{hub_slug}/{sub_hub}/origenes-{entrenador_slug}`) | Párrafo 1 del intro (cuando exista) |
| Palmarés general del club (`/futbol/palmares-{equipo_slug}`) | En el frame line de total trophies |
| Hub de cada competición mayor (`/futbol/champions-league-masculina/`, etc.) | Primera mención de cada competición |
| Páginas per-competition trophy-count (`/futbol/{sub_hub}/cuantas-{competicion}-{equipo_slug}`) | Al menos 2-3 a lo largo del artículo |
| Fan-identity / cultural (`/futbol/{sub_hub}/aficionados-{equipo_slug}-llama-{nombre_aficion}`) | Cuando aplica (ej. "los colchoneros") |
| Rival storyline (club rival o entrenador rival) | Primera mención del clásico/derbi |

Cada link en su contexto natural, no apilados al final.

---

## Sample resumido: Simeone / Atlético de Madrid

```
H1: Los títulos que ha ganado el Atlético de Madrid con Simeone al frente del equipo

[Párrafo 1 — antes/después]
   Antes de Simeone, el Atlético acumulaba apenas 4 participaciones
   en Champions League y ningún título de Liga desde 1996. Bajo su
   dirección, el club ha disputado 12 Champions en 14 temporadas
   y ha ganado 8 grandes títulos, incluida la Liga 2013/14 que
   rompió 18 años de espera.

[Párrafo 2 — estado hoy]
   Pasada una década en el banquillo, Simeone sigue siendo la
   cara visible del proyecto. Su salario lo posiciona entre los
   tres entrenadores mejor pagados del mundo y la directiva ha
   renovado su contrato tres veces sin presiones reales de
   sustitución.

[Párrafo 3 — identidad de era]
   El cholismo es el brand de su era: bloque medio compacto,
   transiciones rápidas, presión coordinada en pérdida y un
   discurso de partido a partido que la afición rojiblanca
   adoptó por completo. "Ganar, y ganar, y volver a ganar"
   resume su filosofía.

H3: Estos son los títulos de Simeone con el Atlético de Madrid
   Con Simeone, el Atlético de Madrid ha conseguido 8 de los
   30 títulos que ha levantado a lo largo de su historia.

   Simeone llegó al banquillo del Atleti a mitad de temporada
   2011/12, sustituyendo a Gregorio Manzano tras una racha
   negativa que dejó al equipo cerca del descenso.

   [Walk cronológico de los 8 títulos:
    Europa League 2011/12 (final ante Athletic 3-0),
    Supercopa de Europa 2012 (Chelsea 4-1),
    Copa del Rey 2012/13 (Real Madrid 2-1 en el Bernabéu),
    Liga 2013/14 (cerrada en el Camp Nou con empate 1-1),
    final perdida Champions 2014 (Real Madrid 1-4 en Lisboa),
    Liga 2020/21 (Valladolid 2-1),
    final perdida Champions 2016 (Real Madrid penaltis Milán),
    Supercopa de España 2014, etc.]

H3: Palmarés de Simeone como entrenador del Atlético de Madrid
   - 2 Ligas (2013/14, 2020/21)
   - 1 Copa del Rey (2012/13)
   - 1 Europa League (2011/12)
   - 1 Supercopa de Europa (2012)
   - 1 Supercopa de España (2014)
   - [resto]

H3: Los títulos de Simeone como jugador del Atlético de Madrid
   [Narrativa: llegada del Inter, el doblete histórico 1995/96
    con Radomir Antić, salida a Lazio]

H3: Palmarés de Simeone como jugador del Atlético de Madrid
   - 1 Liga (1995/96)
   - 1 Copa del Rey (1995/96)

H2: Preguntas frecuentes sobre Simeone en el Atlético de Madrid
   H3: ¿Cuántos títulos ha ganado Simeone con el Atlético de Madrid?
   H3: ¿Cuál fue el primer título de Simeone con el Atlético?
   H3: ¿Cuántas Ligas ha ganado Simeone con el Atlético?
   H3: ¿Ha ganado Simeone la Champions League con el Atlético?
   H3: ¿Cuándo llegó Simeone al Atlético?

[Cierre — sin H2 "Cierre"]
   El cholismo ya tiene años de historia y temporadas en las
   que la épica se ha visto contestada. **Lo que sostiene la
   era es la disciplina del bloque**, pieza de identidad que
   sobreviviría a varias generaciones de plantilla.
```

---

## Reglas duras

- **NO** "X es uno de los entrenadores más exitosos" como apertura. Siempre arrancar con antes/después + dato duro.
- **NO** fechas y datos secos sin contexto. Cada trofeo lleva al menos rival + score + 1 dato narrativo (goleador, momento clave, dato físico de la final).
- **NO** drift a territorio tabloide sobre vida personal del entrenador. Las polémicas profesionales sí (decisiones tácticas controvertidas, salida turbulenta del predecesor), las personales no.
- **NO** publicar sin verificar el conteo actual de títulos si el entrenador sigue en activo. WebSearch obligatorio el día del draft.
- **NO** mezclar palmarés y narrativa en el bullet list. La lista es seca, la narrativa va en la sección H3 "Estos son los títulos…".
- **NO** invento de citas de la institución del club ni del entrenador. WebSearch + verificación textual.
- **NO** H2 final titulado "Cierre" / "Conclusión" / "En definitiva" / "Para terminar" (style guide v1.2 §6). Último párrafo lleva 1 negrita valor-scan y sin conectores prefabricados.
- **SÍ** verificar via WebSearch info viva (total de títulos a fecha, última temporada, rival + score de cada final).
- **SÍ** desarrollar finales perdidas que definen la era con el mismo cuidado que las ganadas.
- **SÍ** nombrar la era — sin nombre, el artículo pierde el ancla narrativo.

---

## Adaptaciones por deporte (especificidades en la tabla de palmarés y narrativa)

| Deporte | Datos críticos en bullet palmarés | Cuidado narrativo extra |
|---|---|---|
| Fútbol (club) | Liga doméstica, Copa, Continental (Champions/Europa), Supercopas | Sección doble rol si fue jugador del mismo club |
| Fútbol (selección) | Mundiales, Eurocopas/Copas América, Nations League | No sección doble rol. Internacionalidades si jugó para el país |
| Baloncesto (NBA) | Anillos NBA, Conference titles, MVPs entrenador | Contador Finales (W-L); el "casi" pesa en NBA |
| Baloncesto (ACB) | Liga ACB, Copa del Rey, Euroliga | Conteo de Final Fours |
| F1 | Constructores, Pilotos (con piloto atribuido), GP wins por temporada | Sustituir "entrenador" por "team principal" |
| Rugby | Six Nations, Champions Cup, liga doméstica | Volumen bajo pero high-search en hub rugby |
| Ciclismo | Grand Tours (general/etapas) bajo dirección técnica, mundiales | Solo cuando la figura tiene impacto multi-año |

---

## Integración con el style guide v1.2

Este template hereda los 25 puntos del self-check de `style_guide.md` v1.2. Específicamente, en Coaches se aplican con dureza:

- **Negritas + test del valor scan** (§17): cobertura ≥80% del párrafos del cuerpo, cada negrita pasa el test "si la elimino, ¿el lector aún capta el punto?". Las negritas naturales: nombre de competición, año, score, rival, dato pivote.
- **Listas — consistencia intra-lista** (§24): el bullet del palmarés usa formato uniforme `N {Competicion} ({temporada1} y {temporada2})`. Sin mezcla.
- **Cierre accionable** (§25): sin H2 "Cierre", último párrafo con negrita valor-scan, sin prefabs.
- **Antialucinación con listas verificables** (§21): toda atribución de título a un entrenador verificada contra fuente oficial (web del club, ATP/ACB/NBA según deporte). Cero "X ganó Y" sin confirmación.

## Reference

Página de referencia en SharePoint: `/futbol/liga-primera/titulos-ganados-atletico-madrid-simeone-20230605161108-nt.html`. Los artículos nuevos en este formato deberían replicar su ritmo:

1. Intro era-setting de 3 párrafos.
2. 1 sección H3 narrativa cronológica de trofeos (incluyendo finales perdidas que definen la era).
3. 1 H3 con bullet palmarés.
4. (Opcional) Sección doble rol + palmarés.
5. H2 FAQs (5 preguntas).
6. Cierre breve con negrita valor-scan.

## Editorial Do's and Don'ts (compactos)

### Do
- Negritas sobre fechas clave, números, rivales y scores (dentro del rango 5-7 palabras consecutivas siempre que es posible — la regla de cobertura es por párrafo, no por sección).
- Caminar los trofeos en orden cronológico, no agrupados por competición.
- Tratar las finales perdidas que definen la era como **load-bearing**, no como notas a pie.
- Nombrar la era (cholismo, tiki-taka, heavy-metal…) — esa es la historia.

### Don't
- Abrir con framing genérico tipo "X es uno de los entrenadores más exitosos".
- Drift a territorio tabloide en vida personal.
- Publicar sin verificar el conteo de títulos de la temporada en curso si el entrenador sigue activo.
- Inflar el word count hacia las 2.500 palabras — el formato es 800-1.200 por diseño.
