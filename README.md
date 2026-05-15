# Relevo Content — Preview & batches

Outputs públicos del workflow Claude-Code-native para generación de artículos SEO de [Relevo.com](https://relevo.com). Cada subcarpeta de `runs/` es un artículo generado por el pipeline multi-pass (research → draft → critique → revise → critique → fact-check).

## 🌐 Sitio renderizado

**[https://jpwaimann.github.io/relevo-content/](https://jpwaimann.github.io/relevo-content/)**

Servido por GitHub Pages desde la rama `main`.

## 📂 Estructura

```
.
├── index.html                      # landing del preview
├── docs/
│   ├── EXEC_SUMMARY_call.html      # resumen ejecutivo (HTML)
│   ├── EXEC_SUMMARY_call.md        # versión markdown
│   └── README.md                   # arquitectura técnica del skill /relevo
├── _meta/
│   ├── topics.json                 # topics del batch
│   └── AGENT_CONTRACT.md           # contrato que siguen los subagentes generadores
└── runs/                           # un directorio por artículo generado
    ├── relevo-rules-smoke-<id>/    # smoke test inicial (14/5)
    └── art-N-<batch-ts>/           # artículos del batch (15/5)
        ├── article.html            # versión final (la que se publica)
        ├── article_draft.html      # draft pre-revise (para trazabilidad)
        ├── article_final.json      # metadata + quality_journey
        ├── critique_pass1.json     # auto-crítica primera pasada
        └── critique_pass2.json     # auto-crítica post-revise (si aplica)
```

## 🧪 Batch del 15/5/2026 — 10 artículos

| # | Deporte | Categoría | Tema | Score | Words |
|---|---|---|---|---|---|
| 1 | Fútbol | Tactics | Posiciones | 86 | 1791 |
| 2 | Fútbol | Tactics | Tipos de tiro | 93 | 1836 |
| 3 | Baloncesto | Rules | Reglas FIBA/NBA | 92 | 1995 |
| 4 | Baloncesto | Tactics | Tipos de tiro | 91 | 1683 |
| 5 | Tenis | Rules | Reglas y puntuación | 88 | 1998 |
| 6 | Tenis | Origins | Orígenes de Carlos Alcaraz | 88 | 2604 |
| 7 | Fórmula 1 | Rules | Reglamento FIA | 91 | 2000 |
| 8 | Fórmula 1 | Tactics | Estrategia de boxes | 93 | 1986 |
| 9 | MMA | Origins | Orígenes de Ilia Topuria | 89 | 2567 |
| 10 | MMA | Rules | Reglas UFC | 92 | 1998 |

**Promedio**: 90.3/100. Todos pasaron el gate de calidad (score ≥ 80, sin fails de anti-hallucination).

## 🛠️ Skill `/relevo`

El código del skill (prompts, templates, scripts, sports_config) vive por separado en `/home/pablo/relevo-content/` en la VM. Se publicará en un repo aparte cuando esté estabilizado para colaboración del equipo de content.

## 📬 Contacto

Pablo Waimann · `pablo.waimann@igaming.com`
