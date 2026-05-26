# Lib: image generation + WordPress upload

## Estado al 19/5/2026

**LIVE end-to-end**:
- Generación con `gpt-image-2` vía LiteLLM iGaming (`litellm.bidwhig.com`). Henry's image-gen skill integrado en `scripts/imagen/`. Smoke test E2E pasó 18/5: 25s/1024×1024, strip de C2PA/JUMBF/openai/gpt-image markers automático.
- Upload a WordPress vía `scripts/wp_upload.py` (helper Python que habla MCP via stdio al `ig-mcp-proxy`). Test E2E 19/5: imagen 1024×1024 PNG (1.3 MB) subió a `relevo-com.sub.plus` como attachment 16094, theme Relevo generó 6 thumbnails automáticamente.

## Cuándo se invoca

Una vez que el draft está finalizado (post-revise pass), el pipeline genera:
- 1 hero image obligatoria (1536x1024, JPEG)
- 0-2 inline images opcionales (1024x1024, según categoría — ver abajo)

## Cuántas imágenes por categoría

| Categoría | Hero | Inline | Notas |
|---|---|---|---|
| Rules | 1 | 0-1 | El inline puede ilustrar una situación de juego clave (e.g., un fuera de juego) |
| Tactics | 1 | 1-2 | Ilustraciones tácticas (diagrama formación, técnica de golpe) |
| Origins | 1 | 0 | Solo hero — un portrait/action shot del atleta. Inline images de personas son riesgosas (likeness issues). |

Default conservador: solo hero. El skill puede agregar inline si el editor lo pide en notas.

## Cómo se invoca el script

### Pre-requisito: env vars en shell

El script lee credenciales LiteLLM desde:
1. Shell environment variables (preferido)
2. `.env` en el cwd (fallback)

Vars necesarias (ya configuradas en `~/.bashrc` del host AWS, sección `# LiteLLM iGaming proxy`):

```
BACKEND=litellm
LITELLM_API_KEY=sk-REDACTED                 # iGaming key, ve gpt-image-2 (solicitar al owner)
LITELLM_BASE_URL=https://litellm.bidwhig.com
```

**Gotcha**: el `.bashrc` tiene early-return para shells no-interactivos (default Ubuntu). Si el skill ejecuta el script desde un shell no-interactivo (cron, remote agent, sub-shell), las vars no se cargan. Workarounds:
- Interactive shells (CC editor session): cargan automáticamente, no hace falta hacer nada.
- Non-interactive: el caller debe `export`-ear las 3 vars inline antes de invocar el script, O pasar `--env` con un archivo que las contenga.

### Flujo de invocación

```bash
# 1. Asegurar dir del run
RUN_DIR=/tmp/relevo-{run_id}
mkdir -p "$RUN_DIR"

# 2. Escribir per-run jobs.json
cat > "$RUN_DIR/imagen-job.json" <<EOF
{
  "vars": {
    "style": "Premium editorial sports photography. High-contrast cinematic color grade with directional rim light. Tight composition on athletic action or portrait. Deep shadows, warm highlights. Subtle film grain. No text overlays, no logos, no scores, no graphic elements. Photographic realism, magazine-cover quality."
  },
  "jobs": [
    {
      "name": "hero.jpg",
      "size": "1536x1024",
      "quality": "high",
      "output_format": "jpeg",
      "output_compression": 85,
      "prompt": "{HERO_PROMPT_BUILT_BY_SKILL} {style}"
    }
  ]
}
EOF

# 3. Ensure env (no-op si shell interactiva)
[ -z "$LITELLM_API_KEY" ] && source ~/.bashrc 2>/dev/null || true
# Si después de eso sigue vacío, abortar con error claro:
[ -z "$LITELLM_API_KEY" ] && { echo "ERROR: LITELLM_API_KEY no está en env. Source ~/.bashrc o exportá inline."; exit 1; }

# 4. Invocar script
python3 /home/pablo/relevo-content/scripts/imagen/generate-images.py \
    --jobs "$RUN_DIR/imagen-job.json" \
    --output-dir "$RUN_DIR/"
```

El script:
- Genera con `gpt-image-2` via LiteLLM iGaming
- Strippea C2PA / JUMBF / OpenAI / gpt-image metadata automáticamente
- Output queda en `/tmp/relevo-{run_id}/hero.jpg`
- Re-encode via Pillow puede convertir el archivo a PNG aunque la extensión sea .jpg (output_format=jpeg honored pero Pillow strip puede sobrescribir). No crítico para el pipeline — el archivo es válido image. Si quisiéramos JPEG estricto, ajustar `strip-ai-metadata.py` para preservar el formato original.

## Upload del hero a WordPress

Tras la generación, el skill llama al helper `scripts/wp_upload.py` para subir la imagen al media library de `relevo-com.sub.plus`. **No pegar base64 inline en una tool call del MCP** — el helper habla directo al `ig-mcp-proxy` por stdio y evita saturar el output del agente.

### Pre-requisito

El proxy debe estar configurado (identity + connection activas en `~/.ig-mcp/config.json`). El setup vive en `[[project_wp_mcp_setup_complete]]` y solo se hace una vez por host. Si `ig_proxy.whoami` desde Claude Code devuelve identity + active connection, el helper funciona.

### Flujo de invocación

```bash
# Después del paso de image-gen, hero.jpg ya existe en $RUN_DIR/hero.jpg

# Subir y capturar el attachment_id en una sola línea con --quiet
ATTACHMENT_JSON=$(python3 /home/pablo/relevo-content/scripts/wp_upload.py \
    "$RUN_DIR/hero.jpg" \
    --alt "$ARTICLE_TITLE — imagen principal" \
    --filename "${ARTICLE_SLUG}-hero.jpg" \
    --quiet)

ATTACHMENT_ID=$(echo "$ATTACHMENT_JSON" | python3 -c "import json,sys; print(json.load(sys.stdin)['attachment_id'])")
ATTACHMENT_URL=$(echo "$ATTACHMENT_JSON" | python3 -c "import json,sys; print(json.load(sys.stdin)['url'])")

# El attachment_id es el que se pasa después a posts.create como featured_image_id.
```

### Como módulo Python (desde otros helpers)

```python
from scripts.wp_upload import upload_image

result = upload_image(
    "/tmp/relevo-{run_id}/hero.jpg",
    alt=f"{article_title} — imagen principal",
    filename=f"{article_slug}-hero.jpg",
)
attachment_id = result["attachment_id"]   # int — pasar a posts.create
attachment_url = result["url"]            # str — para el log/journal
```

### Casos cubiertos

- Tamaño completo 1024×1024 o 1536×1024 PNG/JPEG (>1 MB): OK, sin recompresión. Probado 19/5 con 1024×1024 PNG 1.3 MB.
- WP genera automáticamente los thumbnails que define el theme Relevo: `thumbnail` (150), `card-feed` (300×280), `card-photo-top` (570×320), `card-small` (285×178), `card-video` (400×500), `author-avatar` (64), `team-logo` (80), `player-photo` (120). El editorial no tiene que producir esos crops manualmente.
- Alt text se persiste en `_wp_attachment_image_alt`.
- Si la imagen es muy grande para el upload límite del MCP (25 MB media-library, intersectado con `wp_max_upload_size` del site), el helper retorna el error del server tal cual. No silenciar — loggear como `image_upload_failed` y abortar el publish del artículo (el artículo queda como `pending_review` sin hero, el reviewer lo subsana manualmente).

### Fallback del upload

Si el helper falla por proxy caído, identity perdida, o classifier de Anthropic indisponible (transient, ~5 min típicos):

| Modo | Respuesta |
|---|---|
| `error: tool_not_found` | Connection del proxy se perdió. Re-correr `ig_proxy.add_connection` con `https://relevo-com.sub.plus/wp-json/mcp/v1/rpc` |
| `error: source_truncated` | Sha256 mismatch — la imagen se truncó en transit. Reintentar 1 vez; si persiste, regenerar la imagen |
| HTTP 5xx del server WP | Retry con backoff (2, 4, 8s). Si persiste, marcar artículo `image_upload_failed` y continuar a Asana sin imagen |
| `claude-opus-4-7[1m] is temporarily unavailable` | Es Anthropic upstream, no WP. Esperar 5–10 min y reintentar. NO es problema local |

## Cómo construir el hero prompt

El skill genera el `HERO_PROMPT` basado en el artículo. Pattern:

```
Para Rules:
"Wide cinematic action shot of {sport} — {key situation from article}. 
{Setting: stadium / arena / track}. 
Professional sports broadcast aesthetic."

Para Tactics:
"Tactical aerial / wide-angle view of {sport} showing {concept}.
{Players/equipment positioned to illustrate {concept}}.
Diagrammatic clarity meets photographic realism."

Para Origins:
"Portrait of a professional {sport} athlete in action — {context from article: training, victory celebration, defining moment}.
{Setting matching their story arc}.
Editorial magazine cover composition. No identifiable individual likeness."
```

**IMPORTANTE — riesgo likeness en Origins**: `gpt-image-2` NO debe generar likenesses identificables de personas reales (problemas legales). Para Origins, el prompt debe describir "a professional athlete" genérico, no "Lamine Yamal specifically". El reviewer humano luego decide si esa imagen genérica funciona o si necesita ser reemplazada por una foto licenciada.

## Brand visual

El `vars.style` actual es una hipótesis conservadora. Cuando Pablo pase screenshots de Relevo y definamos el brand visual real, ese campo se actualiza en este file. Cero código a cambiar.

## Fallback si la generación falla

Modos de fallo conocidos y respuesta:

| Modo | Respuesta MVP |
|---|---|
| HTTP 503/504 (transient) | Retry con backoff exponencial (2, 4, 8s, 3 reintentos). Ya implementado en patterns sharepoint_msal.py — replicar acá |
| `model not found` (LiteLLM cambió config) | Loggear `image_failed`. Continuar sin hero, marcar artículo con flag `needs_image` en notes |
| Timeout (>4min) | Loggear `image_failed`. Continuar sin hero. Reviewer pega imagen manual |
| API key invalid | Hard fail, abortar pipeline. Pablo debe revisar `~/.bashrc` o creds en Passbolt |

Para MVP no bloquear el pipeline por fallo de imagen — el artículo sube a WP sin hero + flag claro al reviewer.

## Logs relacionados

```
image_started:   payload {hero_prompt_first_80_chars, model: "gpt-image-2", backend: "litellm"}
image_completed: payload {file_path, size_bytes, duration_seconds, format}
image_failed:    payload {error, http_status, fallback_action: "no_hero"|"abort"}
```

## Conecta con

- [[reference_litellm_proxy]] — proxy info, modelos disponibles
- [[project_image_gen_working]] — smoke test que confirmó el flujo end-to-end
- `scripts/imagen/README.md` — doc del script standalone (Henry's original)
- `scripts/imagen/jobs.json` — template multi-job (no usado en /relevo new — generamos uno per-run)
