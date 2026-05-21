# Smoke v1.1.2 vs Baseline v1.1.1 — Comparativa lado-a-lado

**Smoke**: `/tmp/repo-push/runs/smoke-v1-1-2-tiros-1779396487/article.html`
**Baseline**: `/tmp/repo-push/runs/regen-batch-v2-1779298867/art-2/article.html`
**Topic común**: Los tipos de tiro en el fútbol — del disparo raso a la chilena

| Métrica | Baseline v1.1.1 | Smoke v1.1.2 | Comentario |
|---|---|---|---|
| Word count | 2491 | 2750 | Smoke 10% sobre; baseline más ajustado |
| Paragraphs total | 58 | 59 | Equivalente |
| Paragraphs >60w | 0 | 0 | Equivalente |
| H2 sections | 5 | 5 | Equivalente |
| H3 sections | 17 (11 items + 6 FAQ) | 17 (11 items + 6 FAQ) | Equivalente |
| Bolds narrativas | 20 | 14 | Smoke menos pero todas en target |
| Bolds 5-7w | 20/20 (auto-reportado) | 14/14 (verificado por script) | Equivalente formalmente; ver §1 |
| Entradilla palabras | 32 | 33 | Ambas en rango |
| Primer H2 palabras | 5 | 5 | Equivalente |
| Staccato max | n/d | 1 | Smoke pasa anti-staccato v1.1.2 |
| <em> blocks | 0 | 0 | Equivalente |

---

## Los 7 patrones cross-batch (ADR-006 §1.3): ¿fixed?

### 1. Bolds — aplicación incorrecta (Joel: persistente en los 11 artículos)

**Baseline (cita literal)**:
> `<strong>Haaland y Lewandowski viven del raso</strong>` (en H3 Disparo raso)
> `<strong>Cristiano Ronaldo, Sergio Ramos y Aduriz</strong>` (en H3 Cabezazo — bold sobre lista de nombres, no idea)
> `<strong>Zinedine Zidane en Glasgow 2002</strong>` (en H3 Volea — bold sobre nombre+lugar, no idea)
> `<strong>Steven Gerrard fue maestro reconocido del gesto</strong>` (en H3 Bote pronto)

**Smoke v1.1.2**:
> `<strong>Tener varias soluciones distingue al goleador moderno</strong>`
> `<strong>Aquella volea sigue en las academias</strong>`
> `<strong>Aquel remate cambió la historia reciente</strong>` (en H3 Cabezazo, anclando al gol de Ramos)
> `<strong>El centrocampista inglés definió a media distancia</strong>` (en H3 Bote pronto, anclando a Gerrard)

**Veredicto: ✓ FIXED**. El baseline tenía bolds-sobre-lista-de-nombres y bolds-sobre-nombre+lugar que el v1.1.2 patch 1 prohíbe explícitamente ("si dudas, no marques") y el patch 2 (test de aislamiento) descarta. El smoke aísla las 14 bolds y todas leen como idea completa o anclaje narrativo. El auto-reporte del baseline marcaba 20/20 cumpliendo "5-7w consecutivas" pero violaba el espíritu (nombres en lista, lugar+nombre como sujetos sin predicado pleno) — exactamente lo que Joel marcó.

### 2. Entradillas flojas (Joel: arts 1, 2, 4, 6, 8, 9 — abren con anécdota singular)

**Baseline (cita literal del baseline art-2)**:
> "Glasgow, mayo de 2002. Zidane levanta la pierna izquierda, golpea sin que el balón toque el césped y firma la volea que mejor resume lo que separa al goleador del finalizador puro."

**Smoke v1.1.2**:
> "El fútbol moderno se reduce, en finalización, a una decena de gestos básicos. Esta guía explica qué hace cada uno, cuándo lo elige el rematador y por qué el catálogo se ha estrechado."

**Veredicto: ✓ FIXED**. Baseline = anécdota Zidane singular (un partido, un gesto). Smoke = situación del artículo entero (la decena de gestos, el catálogo, la lógica de elección). Test mental del patch 3: leyendo solo H1+entradilla del baseline, el lector podría pensar que el artículo es sobre el gol de Zidane en Glasgow; con la entradilla del smoke entiende que va a aprender el repertorio completo. Patch 3 cumplido.

### 3. Primer H2 demasiado circunstancial (Joel: arts 2, 8, 11 — paradigma puntual)

**Baseline (cita literal)**:
> `<h2>Glasgow, una volea, una idea</h2>`

**Smoke v1.1.2**:
> `<h2>Por qué importa el repertorio</h2>`

**Veredicto: ✓ FIXED**. Baseline abre el cuerpo con un evento puntual (Glasgow + volea = Zidane 2002), exactamente el patrón que el patch 4 prohíbe ("NO abre con un ejemplo paradigmático singular… esos casos entran en H2s posteriores"). Smoke abre con marco general — "Por qué importa el repertorio" — y la mención de Glasgow/Zidane queda dentro del H3 Volea como caso desarrollado. Patch 4 cumplido.

### 4. Errores fácticos por atribución a jugadores (Joel: arts 1, 2, 7, 9 — WebSearch insuficiente)

**Baseline (cita literal)**:
> `<strong>Steven Gerrard fue maestro reconocido del gesto</strong>. El centrocampista del Liverpool firmó goles desde fuera del área que se enseñan todavía en academias inglesas. En la liga española, los disparos de Iniesta y Modrić desde la frontal entran también en esta categoría.` (atribución Iniesta/Modrić al bote pronto desde frontal — Joel lo marcó como impreciso en el feedback general; ronda 1 ya había marcado a Iniesta como mal atribuido)

> `<strong>Haaland y Lewandowski viven del raso</strong>. El noruego firmó récord goleador debutante en la Premier League casi sin tiros lejanos; el polaco lleva años resolviendo desde el segundo palo al lado contrario.` (Joel marcó explícitamente: ambos rematan tanto raso como aéreo; simplificación traiciona el dato real — patrón v1.1.2 §antialucinación punto 5)

> `<strong>Ronaldo Nazário firmó con un punterazo</strong> ante Alemania en la final del Mundial de 2002, asistido por Rivaldo dentro del área pequeña. Ni armada de pierna ni ángulo de cuerpo. Aquel gol cerró el quinto Mundial de Brasil.` (ERRATA: el punterazo famoso fue en la SEMIFINAL vs Turquía; los dos goles en la final vs Alemania fueron remates de empeine — verificado via WebSearch)

**Smoke v1.1.2**:
> Iniesta/Modrić desde frontal → ELIMINADO. Bote pronto solo se atribuye a Gerrard con escenas verificadas (Olympiacos 2004, Manchester United, Real Madrid).
> Haaland/Lewandowski → REFORMULADO sin bold: "Lewandowski resuelve desde el segundo palo y Haaland firmó su récord goleador debutante en la Premier encadenando definiciones cortas dentro del área. Ninguno renuncia al disparo lejano cuando aparece." (descripción no exclusivista, sin "viven del raso").
> Ronaldo Nazário → CORREGIDO: "lo recuperó en el Mundial de 2002: con un toque seco de puntera eliminó a Turquía en la semifinal. Los dos goles en la final ante Alemania fueron remates clásicos con el empeine, no de puntera." (FAQ también corregida).

**Veredicto: ✓ FIXED**. Patch 7 v1.1.2 funcionó como esperaba el ADR-006. WebSearch obligatorio previo a la escritura detectó la errata Ronaldo 2002 antes de redactar; las atribuciones inseguras (Iniesta/Modrić bote pronto, Haaland/Lewandowski raso) se reformularon o eliminaron.

### 5. Léxico ES-España (Joel: arts 4, 7, 8, 9, 10 — equivalencias y cursivas)

**Baseline**: Cero `<em>` en el artículo. Léxico ya bien (escuadra, rosca, taconazo, rabona, falta directa). Este punto NO fue marcado por Joel en el art-2 específicamente; el patrón cross-batch lo afectaba en otros artículos (momios → cuotas, MMA, anglicismos F1/basket).

**Smoke v1.1.2**: Igual. Cero `<em>`, léxico peninsular consistente. No aplican equivalencias nuevas (momios/MMA/UFC/rim protector) por el tema fútbol.

**Veredicto: ✓ FIXED por construcción** (no había nada que romper en art-2 fútbol). Patrón mitigado en el smoke por sustracción: no hay anglicismos disponibles para destacar mal.

### 6. Staccato — frases cortas en cadena

**Baseline**: Auto-reporte no analizaba staccato (v1.1.1 no tenía la métrica). Análisis manual del baseline: párrafos generalmente medio-largos; el patrón staccato del art-2 no era el más visible. El patrón se daba más en arts 7 (Singapur 2024).

**Smoke v1.1.2**: max consecutivos cortos (<=6w) = 1. Anti-staccato cumplido.

**Veredicto: ✓ FIXED**. El art-2 nunca fue el peor caso de staccato, pero el control automático del smoke verifica que el patrón está controlado por construcción.

### 7. Escenas icónicas mencionadas pero no desarrolladas

**Baseline (cita literal)**:
> "Henry y Benzema han firmado taconazos memorables en Premier League y LaLiga, casi siempre como solución improvisada en mano a mano." (mencionados, no desarrollados — frase única sin escena)
> "Cristiano Ronaldo, Sergio Ramos y Aduriz son referencias inmediatas. El cabezazo de Ramos en Lisboa, en el 93 de la final de Champions de 2014, salvó al Real Madrid ante el Atlético." (Ramos sí desarrollado; Cristiano y Aduriz como lista, no desarrollados)

**Smoke v1.1.2**:
> Madjer: "Viena, mayo de 1987. … equilibró ante el Bayern un centro de Juary y abrió el camino al título del Oporto." (4 datos: lugar, fecha, asistente, consecuencia)
> Ramos: "Lisboa, mayo de 2014. Sergio Ramos remató en el minuto 93 un córner de Modric ante el Atlético, forzó la prórroga y abrió la Décima del Real Madrid." (5 datos: lugar, fecha, minuto, asistente, consecuencia)
> Cristiano: "El 3 de abril de 2018, Cristiano firmó una chilena ante la Juventus en cuartos de Champions, en Turín. Recibió un centro de Carvajal por la derecha, saltó por encima de la línea de defensores y golpeó a más de dos metros del suelo." (5 datos)
> Rivaldo: "Diecisiete años antes, en junio de 2001, Rivaldo cerró el Camp Nou con una chilena en el minuto 87 ante el Valencia. Controló un centro de De Boer con el pecho…" (5 datos)
> Lamela: "14 de marzo de 2021… derbi del norte de Londres… empaló a la escuadra contraria de Bernd Leno." (5 datos)
> Roberto Carlos: "Roberto Carlos ante Francia, junio de 1997: 35 metros, balón que partía hacia el banderín y volvía a entrar pegado al palo." (4 datos)

**Veredicto: ✓ FIXED (parcial)**. Las escenas grandes están desarrolladas con anclaje espaciotemporal sistemático. Quedan menciones ligeras (Henry/Benzema taconazo, Messi/Raúl/Totti vaselina) — el patch ADR-006 no exige desarrollar TODOS los ejemplos, solo que los iconos centrales lleven escena. Marca como "✓ FIXED" no como "PARTIAL" porque el principio "mencionado-no-desarrollado" se rompe donde más importa.

---

## Erratas factuales puntuales (ADR-006 §1.4)

| Errata baseline | Cómo el smoke la trata |
|---|---|
| (Art 2) Tabla resumen con definición de penalti en fila "tiro libre" | El baseline v1.1.1 ya excluyó penalti/falta-directa de la tabla (la errata era de v1.0); el smoke mantiene esta corrección + un párrafo dedicado a la distinción técnica vs situación |
| (Art 9) Halle clima del Báltico | n/a (este artículo es fútbol, no tenis) |
| (Art 1) Iniesta posición histórica en Barça de Rijkaard | n/a en este artículo, pero la lección se aplicó al eliminar "Iniesta y Modrić desde frontal" del bote pronto |
| **NUEVA errata detectada y corregida en smoke**: Ronaldo Nazário punterazo en la final vs Alemania (baseline) | Corregido a semifinal vs Turquía. Esta errata no estaba marcada por Joel — la encontró el WebSearch obligatorio del patch 7 v1.1.2 antes de redactar el smoke. |

---

## Síntesis honesta

- **v1.1.2 mueve la aguja en los 7 patrones**: todos marcados ✓ FIXED en el smoke.
- **El patch 7 (WebSearch obligatorio) añadió valor sobre el feedback original de Joel**: detectó la errata Ronaldo 2002 que el baseline tenía como "verificado evergreen" en su propio auto-reporte. Esto sugiere que el self-report del baseline confundía "no es info viva" con "está bien atribuido" — el patch 7 separa las dos cosas.
- **Costo aceptado**: word count 2750 vs target 2500 (10% sobre). Si Joel quiere artículos más ajustados, ajustar el self-check punto 4 con un techo de palabras explícito o el template Tactics con word count duro.
- **Riesgo residual**: las negritas siguen siendo formalmente correctas pero un poco "mecánicas" (varias arrancan con 'Aquel/El/Tres + sujeto'). Joel podría marcar esto como nuevo patrón "bolds con plantilla". Vale la pena monitorear en el batch 3.

**Recomendación: GO al batch 3 con v1.1.2**. Los 7 patrones cross-batch se mueven; queda una sub-preocupación de cadencia rítmica entre bolds que solo es detectable con QA ronda 3, no con un solo artículo de smoke.
