# ADR-004: Umbrella hubs — Más motor + Más deporte (sport list 24 → 26)

**Status**: Approved
**Fecha**: 2026-05-20
**Drivers**: Joel Sierra (lista de bios pasada por Teams 20/05), Pablo Waimann (project lead)
**Relacionado**: ADR-003 (lineamientos de ideación), ADR-001 (arquitectura)

---

## 1. Contexto

El 20/05/2026 Joel pasó por Teams una lista de ~22 bios "obligatorios" para el lector español. La mayoría mapeaban limpiamente a los 24 deportes existentes en `data/sports_config.json`. Uno no: **Álex Palou** (3x campeón de IndyCar — 2021, 2023, 2024 — y mayor referente español en monoplazas USA).

IndyCar no está en los 24 deportes actuales. Las categorías de motor son:
- **Fórmula 1** (vertical específico)
- **Rallies** (vertical específico)
- **Motociclismo** (vertical específico)

Al seedearlo en EDIT-07 lo marqué `status=blocked` con una nota pidiendo decisión sobre el deporte. Al consultar a Pablo, indicó que **relevo.com ya tiene en su web actual un hub llamado "Más motor"** — el umbrella para todo lo que no es F1 ni rallies (IndyCar, NASCAR, Fórmula E, endurance). Además existe un hub paralelo **"Más deporte"** para deportes minoritarios sin vertical propio.

Esto encaja perfectamente con la decisión de seedear Palou y abre la puerta a futuros bios (Verstappen jr. en F1 → ya cubierto; Jenson Button en endurance, NASCAR español, Da Costa en Formula E, surf, skate, esports, equitación, etc.) sin reabrir esta discusión.

---

## 2. Decisión

Ampliar `data/sports_config.json` de **24 → 26 deportes** agregando dos hubs umbrella:

### 2.1 Más motor (slug `mas-motor`)

```json
{
  "name": "Más motor",
  "slug": "mas-motor",
  "sub_hubs": ["indycar", "nascar", "formula-e", "endurance"],
  "reglamento_available": false,
  "_pdf_name": null
}
```

**Sub_hubs poblados desde el primer commit** porque conocemos las disciplinas que cubre: IndyCar, NASCAR, Fórmula E, endurance (24h Le Mans, WEC). Pueden crecer con el tiempo (Super Formula, IMSA, DTM, rallycross).

### 2.2 Más deporte (slug `mas-deporte`)

```json
{
  "name": "Más deporte",
  "slug": "mas-deporte",
  "sub_hubs": [],
  "reglamento_available": false,
  "_pdf_name": null
}
```

**Sub_hubs vacíos por defecto** — se pueblan cuando aparezca el primer topic en una sub-disciplina nueva (surf, skate, snowboard, equitación, esports, ajedrez, etc.).

### 2.3 Sin reglamento PDF

Ninguno de los dos hubs tiene un único reglamento canónico (IndyCar Rule Book, NASCAR Rule Book, FIA WEC, FIA Formula E son todos distintos). Para topics Rules/Tactics en estos hubs, `/relevo content-plan` debe fetchar la normativa correspondiente vía **WebSearch dinámico** en lugar de cargar un PDF local. Este flujo se implementa cuando aparezca el primer topic Rules/Tactics en un hub umbrella (Origins no necesita reglamento — usa investigación de jugadores).

### 2.4 Aplicación inmediata

- **Álex Palou** (Excel row id ~175): `sport=Más motor`, `hub_slug=mas-motor`, `sub_hub=indycar`, `status=pending_review` (desbloqueado).
- **Sin retrocompatibilidad rota**: las 24 entradas previas no cambian.

---

## 3. Alternativas consideradas

### 3.1 Meter Palou bajo Fórmula 1 con nota

**Rechazada.** Editorialmente incorrecto — IndyCar no es F1, son series rivales con reglamentos, equipos y filosofía distintos. Diluiría el hub F1 y confundiría futuras búsquedas SEO.

### 3.2 Renombrar Fórmula 1 a "Automovilismo" englobando F1+IndyCar+Rallies

**Rechazada.** Pierde precisión del slug F1 (que ya tiene tráfico y URLs en producción imaginables). Rallies tiene su propia identidad. Más motor como hub paralelo respeta la jerarquía editorial real de relevo.com.

### 3.3 Crear sports individuales por sub-disciplina (IndyCar, NASCAR, Formula E…)

**Rechazada.** Aumenta el management overhead de la config (cada uno con reglamento, sub_hubs, posibles WP categories) sin ROI hoy — Joel pidió 1 bio (Palou). Si IndyCar genera 20 topics más adelante puede graduarse a deporte propio sin romper nada (el hub_slug se mueve de `mas-motor` a `indycar`).

### 3.4 Solo agregar Más motor, no Más deporte

**Rechazada.** Pablo confirmó que relevo.com también tiene Más deporte como hub real. Agregar las dos juntas evita una segunda decisión idéntica cuando aparezca el primer topic minoritario.

---

## 4. Impacto y consecuencias

### 4.1 Impacto positivo

- Palou queda seedeado correctamente.
- Espacio editorial reservado para futuros bios de motor (NASCAR, Formula E, endurance) y de deportes minoritarios (surf, skate, esports) sin nueva decisión de arquitectura.
- Alineamiento 1:1 con la estructura de hubs de relevo.com → mejor mapping a WP categories cuando se publique.

### 4.2 Costo

- **Reglamento dinámico pendiente** en cmd_content_plan.md: cuando llegue el primer topic Rules/Tactics en Más motor o Más deporte, hay que implementar el fetch del reglamento via WebSearch. **Captura: `INTEG-05`** en el backlog.
- **WP category mapping pendiente**: confirmar que el slug `mas-motor` (y `mas-deporte`) renderiza bajo la sección correcta en relevo-com.sub.plus. **Captura: `INTEG-06`** en el backlog (validar en el próximo publish).

### 4.3 Validación

- Smoke test post-cambio: `/relevo content-plan` con `sport=Más motor`, `categoria=origins`, N=2 (con WebSearch identificando candidatos IndyCar/NASCAR). Verificar que el system prompt acepta el hub y que la dedup interna no rompe.

---

## 5. Decisiones derivadas

- **EDIT-10** (Palou sport-fit) se cierra con este commit.
- **`INTEG-05`** (nuevo): implementar fetch dinámico de reglamento via WebSearch para topics Rules/Tactics en hubs umbrella. P1, no bloquea launch porque hoy no hay topics Rules/Tactics en Más motor / Más deporte.
- **`INTEG-06`** (nuevo): confirmar mapping WP de los slugs `mas-motor` y `mas-deporte` a las categorías existentes en relevo-com.sub.plus.

---

## 6. Referencias

- `data/sports_config.json` — sport list canonical (26 sports tras este commit).
- `prompts/joel-topic-feedback-v1.md` — feedback original que disparó la pregunta.
- `prompts/topic_ideation_lineamientos.md` — L1 aplica caso a caso por sub-disciplina dentro de los hubs umbrella.
- ADR-001 — arquitectura general (sports_config como single source of truth).
- ADR-003 — lineamientos de ideación.
- relevo.com web actual — Pablo confirmó las secciones Más motor y Más deporte como hubs existentes (20/05/2026).
