# Feedback Joel — Batch QA #1 (mayo 2026)

**Origen.** Análisis editorial realizado por Joel Sastre (reviewer único del proyecto Relevo.com) sobre los 11 primeros artículos generados por el pipeline. Entregado en Asana el 18-20/05/2026 (task `1214815492685881`, 12 subtasks con `.docx`). Esta es la base editorial sobre la que se construyen las versiones v1.1+ de los prompts.

**Estado del modelo según Joel:** *"Los problemas ya no están en la calidad técnica básica. Los problemas están en la capa editorial, la profundidad contextual, la actualización, la cultura deportiva, el tono y la capacidad de construir textos visuales y reconocibles para el aficionado real."*

---

## Los 12 problemas sistemáticos

Joel identificó 12 defectos que se repiten en los 11 artículos **independientemente del deporte o el tipo de pieza** (Rules / Tactics / Origins). Todos deben corregirse a nivel prompt + style guide.

### 1. Entradilla ausente + H2 inicial demasiado largo

- **Defecto**: los 11 artículos arrancan directamente con bloques explicativos. No hay entradilla entre H1 y el primer H2.
- **Fix**: entradilla obligatoria de **30-35 palabras** que complemente el H1 (no lo repita) y sitúe el enfoque. Primer H2 corto, periodístico, **no** una metadescription disfrazada.

### 2. Ejemplos icónicos sin desarrollar

- **Defecto**: Zidane, Kareem, Sacchi, Topuria-Volkanovski, Alcaraz en US Open se mencionan **al pasar**, nunca se desarrollan.
- **Fix**: **2-5 ejemplos reconocibles por pieza**, cada uno anclado a una jugada / partido / momento histórico real. El ejemplo sirve para *visualizar* el concepto, no como decoración.

### 3. Información desactualizada o incorrecta

- **Defecto**: palmarés incompletos, categorías de peso viejas, normas cambiadas, tecnología explicada de forma antigua. Crítico en figuras vivas (Alcaraz, Topuria).
- **Fix**: distinguir info **evergreen** (regla histórica, mecánica del juego) vs **viva** (palmarés, ranking, normativa de temporada, plantilla de un equipo). **Toda info viva requiere WebSearch obligatorio antes de redactar.**

### 4. Tono: épica prefabricada vs naturalidad

- **Defecto**: frases artificialmente literarias — *"la cuna era alemana, pero el patio fue alicantino"*, *"los entrenadores seguirán retorciendo la pizarra"*, *"trayectoria untuosa del balón"*.
- **Fix**: tono natural, cercano al aficionado español. Funciona mejor cuando **describe escenas** que cuando **estiliza el lenguaje**. Cero frases escritas "para sonar importantes".

### 5. Plantilla sintáctica repetitiva

- **Defecto**: las mismas estructuras aparecen constantemente: *"no solo… sino también"*, *"más que…"*, *"no era… era…"*, *"eso explica…"*.
- **Fix**: variedad sintáctica obligatoria. Alternar frases cortas y medias. Evitar repetir conectores y bisagras dentro del mismo artículo.

### 6. Negritas sin jerarquía editorial

- **Defecto**: negritas destacan palabras sueltas al azar, sin agrupar ideas.
- **Fix** (interpretación correcta — ver clarificación de Pablo 20/05): **la negrita marca una frase o idea de 5-7 palabras consecutivas**, no palabras dispersas dentro del párrafo. La negrita debe leerse como concepto completo si la aislamos del resto del texto.

### 7. Profundidad táctica/contextual insuficiente

- **Defecto**: los textos explican **qué es algo**, pero no **cuándo se usa**, **por qué importa**, **cómo cambió el deporte**.
- **Fix**: estructura obligatoria de **3 capas** para cada concepto importante:
  1. Qué es.
  2. Cómo funciona.
  3. Por qué importa en el deporte moderno.

### 8. FAQs genéricas

- **Defecto**: *"¿Qué nacionalidad tiene Alcaraz?"*, *"¿Cuántos años tiene?"*, *"¿Quién decide el fuera de juego?"* — preguntas obvias o sin sentido evergreen.
- **Fix**: FAQs centradas en **curiosidad real** del aficionado. Ejemplos del propio Joel:
  - *"¿Por qué el tenis cuenta 15-30-40?"*
  - *"¿Qué significa tirar la línea?"*
  - *"¿Por qué el triple cambió el baloncesto?"*
  - *"¿Qué diferencia hay entre UFC y MMA?"*
  - *"¿Qué cambiaría con la ley Wenger?"*

### 9. Párrafos demasiado largos

- **Defecto**: bloques de 70, 80, 100+ palabras siguen apareciendo.
- **Fix**: **máximo 60 palabras por párrafo**. Excepciones solo en perfiles narrativos o análisis tácticos especialmente complejos, justificadas.

### 10. Anglicismos sin contexto

- **Defecto**: tie-break, fadeaway, eurostep, undercut, grappling se usan bien técnicamente, pero sin explicación para el lector generalista.
- **Fix**: distinguir
  - **Anglicismos integrados** (VAR, hat-trick, MVP, pole position): uso natural sin explicar.
  - **Tecnicismos minoritarios** (fadeaway, eurostep, weight cutting, ERS, parc fermé): breve explicación contextual la primera vez.

### 11. No diferencia el tipo de pieza

- **Defecto**: bio, regla y guía táctica se escriben con la misma lógica estructural cuando deberían tener prompts distintos.
- **Fix**: cada categoría (Rules / Tactics / Origins) requiere prompt diferenciado en lo estructural:
  - **Biografías (Origins)** → escenas, conflicto, relato, citas directas, personalidad, evolución narrativa, contexto humano.
  - **Reglas (Rules)** → claridad, ejemplos prácticos, actualización, polémicas, aplicación real, consecuencias tácticas.
  - **Guías tácticas (Tactics)** → cuándo se usa, quién lo representa, qué cambió, cómo evolucionó.

### 12. Falta la capa "valor Relevo"

- **Defecto**: la mayoría de textos explican correctamente el tema, pero pocos aportan algo diferencial respecto a una entrada genérica de internet.
- **Fix**: el prompt debe forzar internamente la pregunta: ***"¿Qué aprende aquí el lector que no encontraría en una explicación básica de Wikipedia o de la primera página de Google?"*** Si la respuesta es "nada", reescribir.

---

## Issues factuales por artículo (corregir al re-run)

Estos son errores de hecho específicos, no defectos de prompt. Al regenerar cada artículo hay que verificar via WebSearch.

| Artículo | Error / falta |
|---|---|
| **Orígenes de Alcaraz** | Palmarés sin Australian Open 2026, US Open 2025, Roland Garros 2025; sin ruptura con Juan Carlos Ferrero (dic 2025); sin declaraciones sobre presión/disfrute; preguntar si añadir apodo "Carlitos" |
| **Orígenes de Topuria** | Sigue dicho peso pluma — ya es **ligero**, doble campeón tras KO a Charles Oliveira (29/06/2025); falta combate Casa Blanca/Trump, conexión Sergio Ramos, deseo de pelear en Bernabéu, polémicas (expareja, exceso de violencia con rival noqueado) |
| **Fuera de juego** | Afirma que la ley Wenger "no se ha probado" — **sí se probó en la liga canadiense** (CPL). Falta línea adelantadísima del Barça de Flick, fueras de juego de Morata, ejemplos semiautomático |
| **F1 boxes** | "Cinco familias de compuestos" mal contextualizado; falta normativa 2026 (Pirelli C1-C5, neumáticos más estrechos); falta confirmar Pirelli como proveedor hasta 2027 + posible neumático único de lluvia |
| **Reglas F1** | Punto por vuelta rápida queda ambiguo (decir cuándo existió/se eliminó); FAQ "descalificación tras meta" mal formulada; faltan ejemplos: Abu Dabi 2021, límites de pista en Austria, descalificaciones recientes, porpoising 2022 |
| **Tipos de tiro fútbol** | Penalti y falta directa **no son tipos de tiro** (son situaciones de juego). Faltan: taconazo, rabona, trivela, punterazo, tiro con exterior. "Golpe franco al gol" no es natural en España — usar "falta directa" / "tiro libre". "Picadito" → "balón picado" |
| **Tipos de tiro básquet** | Eurostep **no es equivalente** a doble paso (es variante). Afirmar que el mate es "el más sencillo técnicamente" es discutible. Faltan: floater/bomba (Navarro), step-back (Harden), referencias a Curry, Kareem, Ginóbili, Jordan/Kobe con fadeaway |
| **Reglas MMA** | "4 onzas" → traducir a **110 gramos**. "Grapper" suena raro → "especialista en lucha/suelo". "10-point must system" → "sistema de 10 puntos". Faltan: Jon Jones / dedos en ojos, Petr Yan vs Sterling, Herb Dean, weight cutting, diferencias UFC/ONE/RIZIN |
| **Reglas básquet** | Faltan: goaltending/interposición FIBA vs NBA, desafíos arbitrales, debate sobre pasos, flopping (Harden), small ball. "Positionless" metido con calzador, contextualizar antes |
| **Reglas tenis** | Falta explicar el **origen histórico** del sistema 15-30-40 (reloj medieval, simplificación del 45 a 40, deuce); falta electronic line calling (sustitución de jueces de línea); falta Isner vs Mahut, super tie-break, shot clock |
| **Posiciones fútbol** | Box-to-box ≠ interior (es un perfil entre varios: organizador, llegador, asociativo, físico). Faltan: carrilero, central abierto en línea de tres, mediocentro organizador, mención al líbero en desuso. Faltan jugadores como ejemplos vivos (Rodri, Bellingham, Haaland, De Bruyne, Iniesta) |

---

## Lo que SÍ funciona (no romper en v1.1+)

- **Foco temático mantenido** — los artículos no se van de tema.
- **Cero relleno SEO artificial** — no hay H2 metidos para alargar.
- **Terminología deportiva española** — generalmente bien usada para retransmisiones (rosca, escuadra, bloque bajo, prórroga, fadeaway, parc fermé).
- **Estructura interna razonable** — los H2 internos son coherentes (el problema es el primero).
- **Cero alucinación de keywords** — las keywords aparecen integradas con naturalidad.
- **Conexión reglamento ↔ táctica** — cuando aparece, funciona (mejor ejemplo: artículo de fuera de juego).
- **Tono adecuado en F1 boxes** y **fútbol tipos de tiro** — son los más cercanos al estándar Relevo.

---

## Impacto en prompts

Esta v1 del feedback impacta a tres lugares:

1. **`prompts/style_guide.md` v1.1** — incorpora reglas 1, 4, 5, 6, 9, 10, 12 (las transversales). **Ya aplicado** en mismo commit que este doc.
2. **`prompts/rules_template.md`** — debe absorber reglas 3, 7, 8 (verificación de info viva, estructura de 3 capas, FAQs de curiosidad). **Pendiente.**
3. **`prompts/tactics_template.md`** — mismas que Rules más énfasis en ejemplos icónicos desarrollados (regla 2). **Pendiente.**
4. **`prompts/origins_template.md`** — escenas reconocibles, citas directas, ruptura clara con tono enciclopédico (reglas 2, 4, 11). **Pendiente.**

La regla 11 ("diferenciar el tipo de pieza") implica que los 3 templates deben **divergir más** entre sí. Hoy comparten demasiada lógica estructural.

---

## Próximos pasos sugeridos

1. **Aplicar v1.1** del style guide (este commit). ✅
2. **Reescribir los 3 templates** absorbiendo lo que les corresponde de la tabla anterior.
3. **Smoke test multi-categoría** — un artículo de cada tipo con prompts nuevos, comparar contra el batch original.
4. **Re-run de los 11 artículos** del batch QA #1 con prompts v1.1+ y fact-checks por WebSearch.
5. **Joel batch QA #2** — segunda ronda de review para validar la mejora.

---

## Anchor

> *"El modelo ya produce estructuras razonables, buena terminología y textos limpios de SEO artificial. Es exactamente en la capa editorial, la profundidad contextual, la actualización, la cultura deportiva y el tono donde Relevo marca diferencias respecto al contenido deportivo genérico. Y es precisamente ahí donde los prompts necesitan volverse muchísimo más específicos."* — Joel Sastre, análisis global batch #1, 20/05/2026.
