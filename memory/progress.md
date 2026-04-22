# Estado del Proyecto

Última actualización: 2026-04-21 — Flujo 03 (Resumidor Operativo) construido y documentado (workflow.json + README). Pendiente desplegar en n8n, crear tabla `Runs` en Airtable, setear env vars y validar end-to-end.

## Flujos

### 01 — CV-Scorer ✅
- [x] Crear carpeta y estructura
- [x] Construir flujo en n8n (workflow.json con 10 nodos + error handling)
- [x] Validar con n8n-mcp (ErrorHandler conectado, onError configurado, HTTP con retry x3)
- [x] Exportar workflow.json
- [x] Escribir README.md
- [x] Definir schema-output.json
- [x] Desplegado en n8n (id=`odsrEUmL0ujLHqFg`)
- [x] Crear credencial Google Sheets OAuth2 y vincularla a AppendToSheets
- [x] Setear env vars `ANTHROPIC_API_KEY`, `GOOGLE_SHEETS_CV_SCORER_ID`, `GOOGLE_OAUTH_CLIENT_ID`, `GOOGLE_OAUTH_CLIENT_SECRET` en Dokploy + compose
- [x] Probar end-to-end con PDFs reales
- [x] Soporte multi-archivo: nodo `SplitBinaries` (Code) entre Webhook y ExtractPdfText; ParseJsonResponse en modo `runOnceForEachItem`
- [x] Publicar versión CV-Scorer:V1.0.0 (draft/publish de n8n 2.0)
- [x] Tomar captura → assets/diagram.png

### 02 — Onboarding Simulado ✅
- [x] Crear carpeta y estructura
- [x] Construir flujo en n8n (workflow.json con 11 nodos + error handling)
- [x] Validar con n8n-mcp (0 errores, 0 warnings bloqueantes)
- [x] Exportar workflow.json
- [x] Escribir README.md
- [x] Pivote a **modo demo**: nodos `CreateWorkspaceUser` y `AddUserToGroup` convertidos a `Set` con payload estructuralmente idéntico al Admin SDK; README actualizado con sección "Modo demo vs producción"
- [x] Crear credenciales en n8n: `AirtableToken`, `SlackBot` (Google ya no aplica en modo demo)
- [x] Poblar tabla `Departamentos` en Airtable con grupos por área
- [x] Setear env vars `GOOGLE_WORKSPACE_DOMAIN` (cualquier string para emails), `AIRTABLE_BASE_ID`, `AIRTABLE_TABLE_*`, `SLACK_CHANNEL_ONBOARDING`, `SLACK_CHANNEL_ERRORS`
- [x] Probar end-to-end con un colaborador de prueba
- [x] Tomar captura → assets/diagram.png

### 03 — Resumidor Operativo
- [x] Crear carpeta y estructura
- [x] Construir flujo en n8n (workflow.json con 13 nodos + error handling + guardia silenciosa)
- [x] Exportar workflow.json
- [x] Escribir README.md
- [x] Importar workflow en n8n (id=`rBTTePnDgB6JVFXp`) — pendiente vincular credenciales (`SlackBot`, `AirtableToken`)
- [ ] Crear tabla `Runs` en Airtable con los campos documentados
- [ ] Crear canal `#resumen-diario` en Slack y setear `SLACK_CHANNEL_RESUMEN` + `AIRTABLE_TABLE_RUNS`
- [ ] Probar cron con datos reales en Google Sheets (Execute Workflow manual)
- [ ] Tomar captura → assets/diagram.png

## Tareas globales
- [ ] Completar README.md principal
- [ ] Verificar .env.example con todas las variables
- [ ] Revisar .gitignore
- [ ] Hacer repo público en GitHub
- [ ] Añadir link del repo al CV y LinkedIn
