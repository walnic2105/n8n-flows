# n8n — Flujos Emblemáticos

Portafolio técnico con **4 flujos de automatización en n8n** integrados con la
**Claude API (Anthropic)**, Google Sheets, Slack, Airtable y **Supabase pgvector**.
El objetivo es demostrar experiencia validada en diseño de pipelines de datos,
integración con LLMs usando structured outputs, RAG y documentación técnica
reproducible.

---

## 📋 Flujos

| # | Carpeta | Nombre | Descripción corta | Estado |
|---|---------|--------|-------------------|--------|
| 1 | [01-cv-scorer/](01-cv-scorer/) | CV-Scorer | Webhook → PDF → Claude API (structured output) → Google Sheets | ✅ Funcional |
| 2 | [02-onboarding-simulado/](02-onboarding-simulado/) | Onboarding Simulado | Formulario → Google Workspace (modo demo) → Slack → Airtable | ✅ Funcional |
| 3 | [03-resumidor-operativo/](03-resumidor-operativo/) | Resumidor Operativo | Cron diario → Google Sheets → Claude API → Slack Block Kit + Airtable log | ✅ Funcional |
| 4 | [04-wiki-chatbot/](04-wiki-chatbot/) | Wiki Chatbot (RAG) | Chat → Claude + Supabase pgvector (RAG) — wiki del Showroom Marketplace | ✅ Funcional |

Cada carpeta contiene su propio `README.md` con diagrama de nodos, credenciales
requeridas, variables de entorno e instrucciones de importación.

---

## 🛠️ Stack tecnológico

- **[n8n](https://n8n.io)** — motor de automatización (self-hosted, v1.0+)
- **[Claude API](https://docs.anthropic.com)** — `claude-sonnet-4-6` / `claude-opus-4-6` con structured outputs
- **Google Sheets** — almacenamiento tabular (Flujos 01 y 03)
- **Slack API** — notificaciones con Block Kit (Flujos 02 y 03)
- **Airtable** — base de operaciones y log de ejecuciones (Flujos 02 y 03)
- **[Supabase](https://supabase.com)** — pgvector para RAG (Flujo 04)
- **[OpenAI](https://openai.com)** — embeddings `text-embedding-3-small` (Flujo 04)
- **Google Drive** — fuente de archivos wiki (Flujo 04)

---

## 📁 Estructura del repositorio

```
n8n-flows/
├── README.md                    ← Este archivo
├── CLAUDE.md                    ← Instrucciones para el agente
├── AGENTS.md                    ← Guía de uso de n8n-mcp
├── .env.example                 ← Variables de entorno (sin valores reales)
├── .gitignore
├── LICENSE
│
├── 01-cv-scorer/
│   ├── README.md
│   ├── workflow.json
│   ├── schema-output.json
│   └── assets/diagram.png
│
├── 02-onboarding-simulado/
│   ├── README.md
│   ├── workflow.json
│   └── assets/diagram.png
│
├── 03-resumidor-operativo/
│   ├── README.md
│   ├── workflow.json
│   └── assets/diagram.png
│
├── 04-wiki-chatbot/
│   ├── README.md
│   ├── workflow-chatbot.json    ← Agente RAG conversacional
│   └── workflow-ingestion.json  ← Pipeline de ingesta de wiki
│
└── memory/
    ├── progress.md              ← Estado detallado por flujo
    └── credentials-map.md       ← Mapa de credenciales por flujo
```

---

## ✅ Requisitos

- **n8n v1.0+** (self-hosted o cloud)
- Cuenta de **Anthropic** con API key (`ANTHROPIC_API_KEY`)
- **Google Cloud** con OAuth2 habilitado para Sheets y Drive (Flujos 01, 03 y 04)
- **Workspace Slack** con bot token y canales creados (Flujos 02 y 03)
- **Airtable** con base configurada (Flujos 02 y 03)
- **Supabase** con pgvector habilitado y Data API activo (Flujo 04)
- **OpenAI** con API key para embeddings (Flujo 04)

El detalle de credenciales por flujo está en
[memory/credentials-map.md](memory/credentials-map.md).

---

## 🚀 Cómo importar un flujo

1. Copiar `.env.example` a `.env` y rellenar los valores reales.
2. En n8n → **Workflows** → **Import from File** → seleccionar el `workflow.json`
   del flujo deseado (por ejemplo [01-cv-scorer/workflow.json](01-cv-scorer/workflow.json)).
3. Crear las credenciales en n8n con los nombres indicados en el README del flujo
   (`AnthropicApi`, `GoogleSheets`, `SlackBot`, `AirtableToken`, etc.).
4. Vincular cada credencial a los nodos correspondientes.
5. Ajustar las variables de entorno en n8n (`Settings → Variables` o archivo
   de compose según tu despliegue).
6. Activar el workflow.

Cada flujo incluye instrucciones específicas en su propio README.

---

## 🔐 Variables de entorno

Ver [.env.example](.env.example) para la lista completa. Principales grupos:

- **n8n API** — `N8N_API_URL`, `N8N_BASE_URL`, `N8N_API_KEY`
- **Anthropic** — `ANTHROPIC_API_KEY`
- **Google** — `GOOGLE_SHEETS_CV_SCORER_ID`, `GOOGLE_OAUTH_CLIENT_ID`, `GOOGLE_OAUTH_CLIENT_SECRET`, `GOOGLE_WORKSPACE_DOMAIN`
- **Slack** — `SLACK_BOT_TOKEN`, `SLACK_CHANNEL_ONBOARDING`, `SLACK_CHANNEL_ERRORS`, `SLACK_CHANNEL_RESUMEN`
- **Airtable** — `AIRTABLE_API_KEY`, `AIRTABLE_BASE_ID`, `AIRTABLE_TABLE_DEPARTAMENTOS`, `AIRTABLE_TABLE_COLABORADORES`, `AIRTABLE_TABLE_RUNS`
- **Supabase** — `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`
- **OpenAI** — `OPENAI_API_KEY`

> ⚠️ **Nunca** subir el archivo `.env` real al repositorio. Solo `.env.example`
> con placeholders.

---

## 📄 Licencia

[MIT](LICENSE)
