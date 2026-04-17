---
name: n8n instance
description: URL, API endpoint y workflows desplegados en la instancia n8n del usuario
type: reference
---

## Instancia n8n del usuario

- **Base URL**: `https://n8ntest.niorahub.tech`
- **API endpoint**: `https://n8ntest.niorahub.tech/api/v1` (usar este, NO `/home/workflows`)
- **Auth**: header `X-N8N-API-KEY` con el JWT guardado en `.env` del repo como `N8N_API_KEY`

## Comportamiento del `.env`

Claude Code y el MCP oficial de n8n **no** leen el `.env` del repo automáticamente.
El `.mcp.json` referencia `${N8N_API_URL}` etc. pero esas vars se expanden desde el
entorno del proceso padre (la terminal que lanza Claude Code), no del archivo.

**Workarounds:**
- Para desplegar ad-hoc desde Claude Code: `set -a; source .env; set +a;` antes del curl.
- Para que el MCP de n8n funcione nativamente: setear las vars en Windows con
  `setx N8N_API_KEY "..."` y reiniciar la terminal antes de Claude Code.

## Workflows desplegados

| Flujo | ID | Estado |
|-------|------|--------|
| 01 — CV-Scorer | `odsrEUmL0ujLHqFg` | inactivo (creado 2026-04-16) |

Editor: `https://n8ntest.niorahub.tech/workflow/<ID>`
