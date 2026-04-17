# Estado del Proyecto

Última actualización: 2026-04-16 — Flujo 01 validado y documentado.

## Flujos

### 01 — CV-Scorer
- [x] Crear carpeta y estructura
- [x] Construir flujo en n8n (workflow.json con 9 nodos + error handling)
- [x] Validar con n8n-mcp (ErrorHandler conectado, onError configurado, HTTP con retry x3)
- [x] Exportar workflow.json
- [x] Escribir README.md
- [x] Definir schema-output.json
- [x] Desplegado en n8n (id=`odsrEUmL0ujLHqFg`, inactivo, pendiente configurar credenciales en la UI)
- [ ] Crear credencial `GoogleSheets` en n8n y vincularla a AppendToSheets
- [ ] Setear env vars `ANTHROPIC_API_KEY` y `GOOGLE_SHEETS_CV_SCORER_ID` en el host n8n
- [ ] Probar end-to-end con PDF real en instancia n8n
- [ ] Activar workflow
- [ ] Tomar captura → assets/diagram.png

### 02 — Onboarding Simulado
- [ ] Crear carpeta y estructura
- [ ] Construir flujo en n8n
- [ ] Simular con cuenta de prueba en Google Workspace
- [ ] Exportar workflow.json
- [ ] Tomar captura → assets/diagram.png
- [ ] Escribir README.md

### 03 — Resumidor Operativo
- [ ] Crear carpeta y estructura
- [ ] Construir flujo en n8n
- [ ] Probar cron con datos reales en Google Sheets
- [ ] Exportar workflow.json
- [ ] Tomar captura → assets/diagram.png
- [ ] Escribir README.md

## Tareas globales
- [ ] Completar README.md principal
- [ ] Verificar .env.example con todas las variables
- [ ] Revisar .gitignore
- [ ] Hacer repo público en GitHub
- [ ] Añadir link del repo al CV y LinkedIn
