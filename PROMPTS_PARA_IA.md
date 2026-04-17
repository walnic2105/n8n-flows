# 🤖 Cómo pedirle a una IA que genere estructuras de proyecto

---

## 🧠 Principio clave

Una IA genera mejores estructuras cuando le das **contexto + objetivo + restricciones**.
La fórmula base es:

```
Soy [rol] y quiero [objetivo].
El stack es [tecnologías].
El proyecto debe [restricciones o estándares].
Genera [entregable exacto].
```

---

## 📋 Prompt 1 — Estructura básica de repositorio

```
Soy un automatizador y quiero publicar en GitHub un repositorio
con 3 flujos de n8n como portafolio profesional.

Stack: n8n, Claude API (Anthropic), Google Sheets, Slack, Airtable.

Los flujos son:
1. CV-Scorer: webhook → extrae texto de PDF → Claude API structured output → Google Sheets
2. Onboarding simulado: formulario → Google Workspace → Slack → Airtable
3. Resumidor operativo: cron diario → Google Sheets → Claude API → Slack Block Kit

Genera la estructura de carpetas y archivos del repositorio con:
- Un README.md principal profesional
- Subcarpetas por flujo con su propio README
- Archivos .env.example y .gitignore
- Carpeta de assets para capturas
Incluye el árbol completo con comentarios en cada archivo.
```

---

## 📋 Prompt 2 — Pedir el contenido de un archivo específico

```
Para el flujo "CV-Scorer" de n8n, genera el contenido del README.md
de la subcarpeta 01-cv-scorer/.

Debe incluir:
- Descripción del flujo en 2-3 líneas
- Diagrama de nodos en texto (no imagen)
- Tabla de credenciales necesarias
- Instrucciones para importar el workflow.json en n8n
- Ejemplo del JSON de output de Claude API
- Variables de entorno que usa este flujo
Tono técnico, conciso, en español.
```

---

## 📋 Prompt 3 — Estructura con agentes IA integrados (Claude Code / Cowork)

```
Quiero un repositorio de automatizaciones n8n que también incluya
agentes de Claude (Anthropic) para asistir en el desarrollo.

Genera la estructura completa incluyendo:
1. Carpetas de flujos n8n (01-cv-scorer, 02-onboarding, 03-resumidor)
2. La estructura estándar de Claude Code con:
   - CLAUDE.md en la raíz (instrucciones del agente)
   - Carpeta .claude/ con settings.json y commands/
   - Carpeta skills/ con skills personalizados
   - Carpeta memory/ para contexto persistente del agente
3. README principal que explique cómo usar los agentes junto con n8n
Incluye el contenido de CLAUDE.md con instrucciones reales para este proyecto.
```

---

## 📋 Prompt 4 — Generar el CLAUDE.md del proyecto

```
Crea el archivo CLAUDE.md para un proyecto de automatización con n8n.

Contexto del proyecto:
- 3 flujos: CV-Scorer, Onboarding Simulado, Resumidor Operativo
- Stack: n8n, Claude API, Google Sheets, Slack, Airtable, Google Workspace
- El repo es un portafolio profesional para conseguir trabajo

El CLAUDE.md debe instruir al agente sobre:
- Qué hace cada flujo y dónde están sus archivos
- Convenciones de nombrado de nodos en n8n
- Cómo manejar credenciales y variables de entorno
- Qué validar antes de exportar un workflow.json
- Estilo de documentación de los README individuales
Sé específico y usa secciones con headers markdown.
```

---

---

# 🗂️ Estructura de Claude para Agentes y Skills

Cuando usas Claude Code o Cowork en un proyecto, Claude puede leer
y escribir en una estructura de archivos especial que le da memoria
y habilidades personalizadas persistentes.

---

## Árbol completo

```
tu-proyecto/
│
├── CLAUDE.md                    ← 🧠 Instrucciones principales del agente
│                                   Claude lee esto al inicio de cada sesión
│
├── .claude/
│   ├── settings.json            ← ⚙️ Configuración de Claude para este proyecto
│   │                               (permisos, modelo, herramientas habilitadas)
│   │
│   ├── settings.local.json      ← ⚙️ Config local (NO subir a git — datos personales)
│   │
│   ├── commands/                ← 🔧 Slash commands personalizados
│   │   ├── deploy.md            ← Ejemplo: /deploy → instrucciones de despliegue
│   │   ├── test-flow.md         ← Ejemplo: /test-flow → cómo probar un flujo
│   │   └── export.md            ← Ejemplo: /export → exportar workflow.json
│   │
│   └── hooks/                   ← 🪝 Scripts que corren antes/después de acciones
│       ├── pre-tool-use.sh      ← Antes de que Claude use una herramienta
│       └── post-tool-use.sh     ← Después de que Claude use una herramienta
│
├── skills/                      ← 🛠️ Habilidades reutilizables del agente
│   ├── n8n-export/
│   │   └── SKILL.md             ← Instrucciones para exportar flujos n8n
│   ├── validate-workflow/
│   │   └── SKILL.md             ← Instrucciones para validar workflow.json
│   └── write-readme/
│       └── SKILL.md             ← Instrucciones para escribir READMEs de flujos
│
└── memory/                      ← 💾 Memoria persistente entre sesiones
    ├── index.md                 ← Índice de todos los archivos de memoria
    ├── project-context.md       ← Contexto general del proyecto
    ├── credentials-map.md       ← Mapa de qué credencial usa cada flujo (sin valores)
    └── progress.md              ← Estado actual: qué flujos están terminados
```

---

## 📄 Ejemplo de CLAUDE.md para este proyecto

```markdown
# n8n Flujos Emblemáticos — Instrucciones del Agente

## Sobre este proyecto
Repositorio de portafolio con 3 flujos n8n + Claude API.
Objetivo: demostrar experiencia en automatización para vacante laboral.

## Flujos del proyecto
| Carpeta | Flujo | Estado |
|---------|-------|--------|
| 01-cv-scorer/ | CV-Scorer | En construcción |
| 02-onboarding-simulado/ | Onboarding Simulado | Pendiente |
| 03-resumidor-operativo/ | Resumidor Operativo | Pendiente |

## Convenciones de nodos en n8n
- Los nombres de nodos usan PascalCase: `ExtractPdfText`, `CallClaudeApi`
- Siempre incluir nodo `ErrorHandler` al final de cada flujo
- Las credenciales se llaman igual que el servicio: `GoogleSheets`, `SlackBot`

## Antes de exportar un workflow.json
1. Verificar que no haya credenciales hardcodeadas en ningún nodo
2. Usar variables de entorno con la sintaxis `{{ $env.VARIABLE_NAME }}`
3. Probar el flujo completo al menos una vez con datos reales
4. Tomar captura y guardar en assets/diagram.png

## Estilo de documentación
- READMEs en español
- Tablas para listar nodos y credenciales
- Incluir siempre un ejemplo de input y output del flujo
```

---

## 📄 Ejemplo de .claude/settings.json

```json
{
  "model": "claude-sonnet-4-6",
  "permissions": {
    "allow": [
      "Bash(npm:*)",
      "Bash(node:*)",
      "Read(**)",
      "Write(**.md)",
      "Write(**.json)"
    ],
    "deny": [
      "Bash(rm -rf:*)"
    ]
  },
  "env": {
    "PROJECT_NAME": "n8n-flujos-emblematicos"
  }
}
```

---

## 📄 Ejemplo de skill: skills/write-readme/SKILL.md

```markdown
# Skill: write-readme

Úsame cuando necesites generar el README.md de un flujo n8n.

## Instrucciones
1. Lee el workflow.json del flujo en cuestión
2. Identifica: trigger, nodos principales, integraciones y output final
3. Genera el README con estas secciones obligatorias:
   - Descripción (2-3 líneas)
   - Diagrama de nodos (tabla markdown)
   - Credenciales necesarias (tabla)
   - Variables de entorno (lista)
   - Ejemplo de input/output
   - Instrucciones de importación en n8n
4. Tono: técnico, conciso, en español
5. Guardar en la carpeta del flujo correspondiente
```

---

## 📄 Ejemplo de memory/progress.md

```markdown
# Estado del Proyecto

Última actualización: 2024-01-15

## Flujos
- [x] 01-cv-scorer → workflow.json exportado, README completo
- [ ] 02-onboarding-simulado → En construcción, falta nodo Airtable
- [ ] 03-resumidor-operativo → Pendiente

## Pendientes globales
- Crear .env.example con todas las variables
- Tomar capturas de los 3 flujos para assets/
- Revisar README principal antes del push
```

---

## 🔑 Regla de oro para los prompts

Cuando le pidas a cualquier IA (Claude, ChatGPT, Gemini) que genere
una estructura, siempre incluye:

1. **Rol:** quién eres y para qué usarás el proyecto
2. **Stack exacto:** tecnologías y versiones si las sabes
3. **Objetivo final:** portafolio, producción, prueba de concepto
4. **Entregable concreto:** árbol de carpetas, archivo específico, JSON, etc.
5. **Restricciones:** idioma, estilo, qué NO incluir

Cuanto más específico, mejor resultado.
```

---
