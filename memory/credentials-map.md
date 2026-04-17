# Mapa de Credenciales por Flujo

> ⚠️ Este archivo documenta QUÉ credenciales se usan.
> Los valores reales van en el archivo .env (no subir a git).

## Flujo 1 — CV-Scorer

| Credencial en n8n | Servicio | Variable de entorno |
|-------------------|----------|---------------------|
| `AnthropicApi` | Claude API | `ANTHROPIC_API_KEY` |
| `GoogleSheets` | Google Sheets OAuth2 | `GOOGLE_SERVICE_ACCOUNT_EMAIL`, `GOOGLE_PRIVATE_KEY` |

## Flujo 2 — Onboarding Simulado

| Credencial en n8n | Servicio | Variable de entorno |
|-------------------|----------|---------------------|
| `GoogleAdminSdk` | Google Workspace Admin SDK | `GOOGLE_ADMIN_EMAIL`, `GOOGLE_DOMAIN` |
| `SlackBot` | Slack API | `SLACK_BOT_TOKEN` |
| `AirtableApi` | Airtable | `AIRTABLE_API_KEY`, `AIRTABLE_BASE_ID` |

## Flujo 3 — Resumidor Operativo

| Credencial en n8n | Servicio | Variable de entorno |
|-------------------|----------|---------------------|
| `GoogleSheets` | Google Sheets OAuth2 | (misma del Flujo 1) |
| `AnthropicApi` | Claude API | (misma del Flujo 1) |
| `SlackWebhook` | Slack Incoming Webhook | `SLACK_WEBHOOK_URL` |
