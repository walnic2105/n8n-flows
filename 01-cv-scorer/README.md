# Flujo 01 — CV-Scorer

## Descripción

Flujo n8n que recibe un CV en PDF por webhook, lo convierte a texto, lo envía a
**Claude API** con un prompt estructurado y guarda el resultado (score, nivel,
fortalezas, red flags y recomendación) en Google Sheets. Responde al webhook con
el JSON evaluado.

---

## Diagrama de nodos

| # | Nodo | Tipo | Función |
|---|------|------|---------|
| 1 | Webhook Trigger | `n8n-nodes-base.webhook` | Recibe `POST /webhook/cv-scorer` con el PDF adjunto en `cv` |
| 2 | ExtractPdfText | `n8n-nodes-base.extractFromFile` | Convierte el binario PDF en texto plano |
| 3 | BuildPrompt | `n8n-nodes-base.set` | Construye `filename`, `cv_text` y el `prompt` final para Claude |
| 4 | CallClaudeApi | `n8n-nodes-base.httpRequest` | `POST https://api.anthropic.com/v1/messages` con `claude-sonnet-4-6`, retry x3 |
| 5 | ParseJsonResponse | `n8n-nodes-base.code` | Parsea el texto devuelto por Claude a objeto estructurado |
| 6 | AppendToSheets | `n8n-nodes-base.googleSheets` | Inserta una fila en la hoja de resultados |
| 7 | RespondToWebhook | `n8n-nodes-base.respondToWebhook` | Devuelve `{ ok: true, result }` al cliente |
| 8 | ErrorHandler | `n8n-nodes-base.code` | Captura errores de nodos 2-6 y normaliza el payload de error |
| 9 | RespondError | `n8n-nodes-base.respondToWebhook` | Devuelve HTTP 500 con `{ ok: false, error }` |

Conexiones de error: `ExtractPdfText`, `CallClaudeApi`, `ParseJsonResponse` y
`AppendToSheets` enrutan su salida de error al `ErrorHandler`.

![Diagrama](assets/diagram.png)

---

## Credenciales necesarias

| Credencial en n8n | Servicio | Alcance / Scopes |
|-------------------|----------|------------------|
| `GoogleSheets` | Google Sheets OAuth2 | `https://www.googleapis.com/auth/spreadsheets` |
| `AnthropicApi` | Anthropic (vía variable de entorno, no credencial de n8n) | API key con permisos de `messages` |

> La llamada a Claude usa `HTTP Request` con header `x-api-key` leído desde
> `{{ $env.ANTHROPIC_API_KEY }}`, así que no requiere credencial de n8n.

---

## Variables de entorno

| Variable | Descripción |
|----------|-------------|
| `ANTHROPIC_API_KEY` | API key de Anthropic con acceso al endpoint `/v1/messages` |
| `GOOGLE_SHEETS_CV_SCORER_ID` | ID del Google Sheet donde se guardan los resultados (la parte entre `/d/` y `/edit` en la URL) |

---

## Ejemplo de input

```bash
curl -X POST "https://<tu-n8n>/webhook/cv-scorer" \
  -F "cv=@./candidato.pdf"
```

Requisitos del request:
- `Content-Type: multipart/form-data`
- Campo binario llamado **`cv`** con el PDF.

---

## Ejemplo de output

```json
{
  "ok": true,
  "result": {
    "timestamp": "2026-04-16T18:45:12.001Z",
    "filename": "candidato.pdf",
    "score": 78,
    "nivel": "Mid",
    "fortalezas": [
      "Experiencia con integraciones REST",
      "Uso de Google Workspace APIs",
      "Documentación técnica clara"
    ],
    "red_flags": [
      "Sin evidencia de pruebas automatizadas"
    ],
    "recomendacion": "Avanzar a entrevista técnica enfocada en testing y diseño de pipelines."
  }
}
```

El esquema completo está en [schema-output.json](schema-output.json).

### Output de error

```json
{
  "ok": false,
  "error": "No se pudo parsear el JSON de Claude: Unexpected token ...",
  "timestamp": "2026-04-16T18:45:12.001Z"
}
```

---

## Estructura esperada de Google Sheets

El flujo hace `append` sobre la hoja `Hoja 1` con estas columnas (en este orden):

| Timestamp | Nombre archivo | Score | Nivel | Fortalezas | Red Flags | Recomendación |
|-----------|----------------|-------|-------|------------|-----------|---------------|

---

## Instrucciones de importación en n8n

1. En n8n: **Workflows → Import from file** → seleccionar [workflow.json](workflow.json).
2. Crear la credencial `GoogleSheets` (OAuth2) y vincularla al nodo `AppendToSheets`.
3. Definir las variables de entorno `ANTHROPIC_API_KEY` y
   `GOOGLE_SHEETS_CV_SCORER_ID` en el host donde corre n8n
   (o en `.env` si usas docker-compose).
4. Activar el workflow y copiar la URL del webhook.
5. Probar con el `curl` de ejemplo y revisar la nueva fila en la hoja.

---

## Notas de diseño

- **Modelo**: `claude-sonnet-4-6`, 1024 tokens de salida (suficiente para el JSON).
- **Reintentos**: `CallClaudeApi` reintenta 3 veces con 2s entre intentos para
  tolerar fallos transitorios de red.
- **Parsing defensivo**: `ParseJsonResponse` tolera que Claude envuelva la
  respuesta con \`\`\`json fences y lanza un error explícito con el raw text si
  el parseo falla.
- **Error handling**: todas las salidas de error convergen en `ErrorHandler` →
  `RespondError` (HTTP 500), garantizando que el cliente siempre reciba respuesta.
