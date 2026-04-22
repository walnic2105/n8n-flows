# Flujo 03 — Resumidor Operativo

## Descripción

Flujo n8n disparado por cron diario que **cierra el loop** sobre la salida del
[Flujo 01 CV-Scorer](../01-cv-scorer/): lee los CVs procesados el día anterior
desde Google Sheets, agrega KPIs (volumen, promedio de score, distribución por
nivel, patrones recurrentes de fortalezas y red flags), pide a **Claude API**
un resumen ejecutivo con observaciones y alertas, y publica el mensaje
formateado con **Block Kit** en un canal de Slack. Cada ejecución queda
auditada en Airtable.

Si no hay registros del día anterior, el flujo termina silenciosamente (no se
envía mensaje vacío a Slack).

---

## Diagrama de nodos

| # | Nodo | Tipo | Función |
|---|------|------|---------|
| 1 | ScheduleTrigger | `n8n-nodes-base.scheduleTrigger` | Cron `0 8 * * 1-5` — 8:00 AM lunes a viernes |
| 2 | ReadSheetData | `n8n-nodes-base.googleSheets` | Lee todas las filas de la hoja del CV-Scorer |
| 3 | AggregateAndFilter | `n8n-nodes-base.code` | Filtra filas del día anterior, calcula KPIs y top patrones |
| 4 | IfHasData | `n8n-nodes-base.if` | Si `hasData=false`, termina sin notificar |
| 5 | BuildPrompt | `n8n-nodes-base.set` | Arma el prompt con los datos agregados |
| 6 | CallClaudeApi | `n8n-nodes-base.httpRequest` | POST a `/v1/messages` con structured output + retry x3 |
| 7 | ParseClaudeResponse | `n8n-nodes-base.code` | Extrae JSON del `content[0].text`, descarta el campo `pensamiento` del CoT |
| 8 | BuildBlockKit | `n8n-nodes-base.code` | Construye los `blocks` de Slack (header + KPIs + resumen + observaciones + alertas) |
| 9 | SendToSlack | `n8n-nodes-base.slack` | Publica el mensaje en `#resumen-diario` |
| 10 | LogRunToAirtable | `n8n-nodes-base.airtable` | Registra la corrida en tabla `Runs` con `estado=ok` |
| 11 | ErrorHandler | `n8n-nodes-base.code` | Normaliza el payload de error y recupera `fecha` parcial del agregador |
| 12 | NotifyErrorChannel | `n8n-nodes-base.slack` | Publica el fallo en `#errores` |
| 13 | LogErrorToAirtable | `n8n-nodes-base.airtable` | Registra la corrida con `estado=error` |

Los nodos 2, 3, 6-10 enrutan su salida de error al `ErrorHandler`.

![Diagrama](assets/diagram.png)

---

## Credenciales necesarias

| Credencial en n8n | Servicio | Alcance / Scopes |
|-------------------|----------|------------------|
| `Google Sheets account` | Google Sheets OAuth2 | Reutiliza la credencial del Flujo 01 (scope `spreadsheets.readonly` es suficiente aquí) |
| `AnthropicApi` | HTTP Request (header `x-api-key`) | Leído de `$env.ANTHROPIC_API_KEY` |
| `SlackBot` | Slack OAuth2 / Bot Token | `chat:write`, `chat:write.public` |
| `AirtableToken` | Airtable Personal Access Token | `data.records:write` sobre la base del portafolio |

---

## Variables de entorno

| Variable | Descripción |
|----------|-------------|
| `ANTHROPIC_API_KEY` | API key de Anthropic (`sk-ant-...`) |
| `GOOGLE_SHEETS_CV_SCORER_ID` | ID del Google Sheet con los resultados del CV-Scorer (reutiliza la misma hoja) |
| `SLACK_CHANNEL_RESUMEN` | ID del canal Slack donde se publica el resumen diario (ej. `C0123456789`) |
| `SLACK_CHANNEL_ERRORS` | ID del canal Slack donde se publican fallos |
| `AIRTABLE_BASE_ID` | ID de la base Airtable (empieza con `app...`) |
| `AIRTABLE_TABLE_RUNS` | ID o nombre de la tabla `Runs` donde se loguea cada ejecución |

---

## Estructura esperada de Airtable

### Tabla `Runs`

| Campo | Tipo | Ejemplo |
|-------|------|---------|
| `fecha` | Date | `2026-04-20` |
| `totalCVs` | Number | `7` |
| `avgScore` | Number | `72` |
| `resumen` | Long text | "Pool técnicamente sólido..." |
| `observaciones` | Long text | "3 Senior · alta rotación en menciones..." |
| `alertas` | Long text | "2 CVs sin experiencia real en IA..." |
| `recomendacion` | Long text | "Avanzar a entrevista los 3 Senior." |
| `estado` | Single select | `ok` \| `error` |
| `createdAt` | Date with time | `2026-04-21T08:00:12.443Z` |

---

## Schema JSON esperado (structured output de Claude)

```json
{
  "resumen": "El pool del día fue mayoritariamente Mid con un score promedio sólido.",
  "observaciones": [
    "3 de 7 candidatos son Senior con score ≥ 85",
    "Domina la mención de n8n y Python entre fortalezas",
    "Sin menciones de trabajo en equipo en 4 CVs"
  ],
  "alertas": [
    "2 CVs mencionan experiencia en IA sin evidencia de proyectos"
  ],
  "recomendacion": "Avanzar a entrevista a los 3 Senior y pedir referencias a los 2 CVs con IA sin evidencia."
}
```

---

## Ejemplo de output en Slack (Block Kit)

```
📊 Resumen Operativo — 2026-04-20

CVs evaluados:      7              Score promedio:     72 (min 45 · max 92)
Distribución:       Senior: 3 · Mid: 3 · Junior: 1
Recomendación:      Avanzar a entrevista a los 3 Senior.

───────────────────────────────────

Resumen:
El pool del día fue mayoritariamente Mid con un score promedio sólido.

Observaciones
• 3 de 7 candidatos son Senior con score ≥ 85
• Domina la mención de n8n y Python entre fortalezas
• Sin menciones de trabajo en equipo en 4 CVs

Alertas
⚠️ 2 CVs mencionan experiencia en IA sin evidencia de proyectos

Generado automáticamente por n8n + Claude API · 2026-04-21T08:00:12.443Z
```

### Output de error

```
⚠️ Fallo en Resumidor Operativo — fecha 2026-04-20
Error: `Request failed with status code 429: rate_limit_exceeded`
Timestamp: 2026-04-21T08:00:14.991Z
```

Fila en Airtable con `estado=error`.

---

## Instrucciones de importación en n8n

1. **Workflows → Import from file** → seleccionar [workflow.json](workflow.json).
2. En el nodo `ReadSheetData`, abrir el selector de `Document` y reasignar la
   credencial `Google Sheets account` (reutiliza la del Flujo 01).
3. Verificar que las credenciales `SlackBot` y `AirtableToken` estén creadas
   (reutiliza las del Flujo 02).
4. Crear la tabla `Runs` en la misma base Airtable del Flujo 02 con los campos
   de la sección anterior.
5. Obtener el ID del canal Slack de resumen (`C0...`) y las variables de entorno
   faltantes en Dokploy / `docker-compose`.
6. Activar el workflow. Para validarlo end-to-end sin esperar al cron, usar
   **Execute Workflow** manualmente — el Schedule Trigger no dispara en modo
   manual pero el resto del pipeline sí corre.

---

## Notas de diseño

- **Fuente de verdad reutilizada**: el Flujo 03 consume la salida del Flujo 01,
  lo que demuestra el pipeline completo de punta a punta sin datos inventados.
- **Guardia silenciosa (IfHasData)**: un día sin CVs no produce ruido en Slack.
  Es una decisión explícita para evitar la fatiga de notificaciones vacías.
- **Agregación antes del LLM**: el Code node calcula KPIs y patrones recurrentes
  *antes* de llamar a Claude. Esto reduce tokens, mejora la señal del resumen y
  deja los números auditables aunque la respuesta del LLM falle.
- **CoT descartable**: el system prompt fuerza a Claude a razonar en una
  propiedad `pensamiento` que se elimina antes de publicar — mejora la calidad
  del JSON sin exponer el scratchpad.
- **Zona horaria**: el filtro por "ayer" usa `America/Mexico_City` vía
  `Intl.DateTimeFormat` para que el cron local (8:00) siempre resuma el día
  operativo correcto, independiente de la TZ del server de n8n.
- **Retry en Claude**: `retryOnFail=true`, `maxTries=3`, `waitBetweenTries=2000`
  absorbe fallos transitorios de 429/503 sin falsos positivos en `#errores`.
- **Auditoría en Airtable**: cada corrida (ok o error) deja una fila. Útil para
  monitoreo en dashboards y para detectar silencios del cron (días sin fila).
