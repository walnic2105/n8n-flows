# n8n Flujos Emblemáticos — Instrucciones del Agente

## 🎯 Sobre este proyecto

Repositorio de portafolio profesional con 3 flujos de automatización en **n8n**
integrados con **Claude API (Anthropic)**. El objetivo es demostrar experiencia
validada en automatización de procesos e integración de APIs para una vacante laboral.

**Dueño del proyecto:** Desarrollador X (`soportex715@gmail.com`)
**Estado:** En construcción activa
**Destino:** Repositorio público en GitHub

---

## 🗂️ Flujos del proyecto

| # | Carpeta | Nombre | Descripción corta | Estado |
|---|---------|--------|-------------------|--------|
| 1 | `01-cv-scorer/` | CV-Scorer | Webhook → PDF → Claude API structured output → Google Sheets | ⏳ Pendiente |
| 2 | `02-onboarding-simulado/` | Onboarding Simulado | Formulario → Google Workspace → Slack → Airtable | ⏳ Pendiente |
| 3 | `03-resumidor-operativo/` | Resumidor Operativo | Cron diario → Google Sheets → Claude API → Slack Block Kit | ⏳ Pendiente |

Consulta `memory/progress.md` para el estado detallado y tareas pendientes de cada flujo.

---

## 🛠️ Stack tecnológico

- **n8n** — Motor de automatización (self-hosted, v1.0+)
- **Claude API** — Anthropic (`claude-sonnet-4-6` o `claude-opus-4-6`)
- **Google Sheets** — Almacenamiento de datos tabulares
- **Google Workspace Admin SDK** — Gestión de usuarios corporativos
- **Slack API** — Notificaciones con Block Kit
- **Airtable** — Base de datos de operaciones (Flujo 2)

---

## 📁 Estructura del repositorio

```
n8n-flujos-emblematicos/
├── CLAUDE.md                    ← Este archivo
├── README.md                    ← README público de GitHub
├── .env.example                 ← Variables de entorno sin valores reales
├── .gitignore
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
├── .claude/
│   └── settings.json
│
├── skills/
│   ├── write-readme/SKILL.md
│   ├── validate-workflow/SKILL.md
│   └── export-checklist/SKILL.md
│
└── memory/
    ├── index.md
    ├── progress.md
    └── credentials-map.md
```

---

## ⚙️ Convenciones de nodos en n8n

- **Nombrado:** PascalCase descriptivo → `ExtractPdfText`, `CallClaudeApi`, `AppendToSheets`
- **Siempre incluir** un nodo `ErrorHandler` conectado a los nodos críticos
- **Variables de entorno:** usar siempre `{{ $env.NOMBRE_VARIABLE }}` — nunca valores hardcodeados
- **Credenciales en n8n:** nombrarlas igual que el servicio → `GoogleSheets`, `SlackBot`, `AnthropicApi`
- **Nodo inicial:** el primer nodo siempre es el trigger (`Webhook`, `FormTrigger`, `ScheduleTrigger`)

---

## 📝 Estilo de documentación

- Todos los archivos en **español**
- READMEs con estas secciones obligatorias:
  1. Descripción (2-3 líneas)
  2. Diagrama de nodos (tabla markdown)
  3. Credenciales necesarias (tabla)
  4. Variables de entorno (lista con descripción)
  5. Ejemplo de input y output
  6. Instrucciones de importación en n8n
- Tono: **técnico y conciso**, sin relleno

---

## ✅ Checklist antes de exportar un workflow.json

1. Ningún nodo tiene credenciales o tokens hardcodeados
2. Todas las variables usan `{{ $env.VARIABLE }}`
3. El flujo fue probado end-to-end al menos una vez con datos reales
4. El nodo `ErrorHandler` está conectado
5. Se tomó captura del flujo y se guardó en `assets/diagram.png`
6. El `README.md` del flujo está completo

---

## 🔐 Mapa de credenciales

Consulta `memory/credentials-map.md` para ver qué credencial usa cada flujo.
**Nunca escribir valores reales en ningún archivo del repo.**

---

## 🚀 Frases de acción rápida

Cuando el usuario diga alguna de estas frases, actúa así:

| Frase del usuario | Acción |
|-------------------|--------|
| "construye el flujo N" | Leer el README del flujo, proponer los nodos y generar el workflow.json base |
| "genera el README de flujo N" | Leer workflow.json del flujo y generar README completo |
| "valida el flujo N" | Ejecutar el checklist de exportación sobre el workflow.json |
| "actualiza el progreso" | Editar `memory/progress.md` con el estado actual |
| "prepara el repo para GitHub" | Verificar README principal, .env.example, .gitignore y checklist completo |

---

## 💡 Contexto para la vacante

Este proyecto demuestra:
- Diseño de pipelines de datos con herramientas no-code/low-code
- Integración con LLMs usando structured outputs
- Manejo de APIs REST (Google, Slack, Airtable, Anthropic)
- Documentación técnica profesional
- Buenas prácticas: variables de entorno, control de errores, exportación reproducible

Al describir el proyecto en entrevistas, enfatizar la **lógica de negocio** de cada flujo,
no solo las herramientas usadas.

## Guía del agente para n8n-mcp
@AGENTS.md