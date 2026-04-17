# 🗂️ Estructura del Proyecto — n8n Flujos Emblemáticos

> Guía completa para construir 3 flujos de automatización con n8n,
> publicarlos en GitHub con README profesional y demostrar experiencia
> validada para una vacante de trabajo.

---

## 📁 Estructura del Repositorio GitHub

```
n8n-flujos-emblematicos/
│
├── README.md                          ← README principal del repo
│
├── 01-cv-scorer/
│   ├── README.md                      ← Explicación del flujo
│   ├── workflow.json                  ← Export del flujo n8n
│   ├── schema-output.json             ← Schema JSON del structured output
│   └── assets/
│       └── diagram.png                ← Captura del flujo en n8n
│
├── 02-onboarding-simulado/
│   ├── README.md
│   ├── workflow.json
│   └── assets/
│       └── diagram.png
│
├── 03-resumidor-operativo/
│   ├── README.md
│   ├── workflow.json
│   └── assets/
│       └── diagram.png
│
├── .env.example                       ← Variables de entorno (sin valores reales)
└── .gitignore
```

---

## 📄 .gitignore (raíz del repo)

```gitignore
.env
*.log
node_modules/
.n8n/
```

---

## 📄 .env.example (raíz del repo)

```env
# Claude API
ANTHROPIC_API_KEY=sk-ant-...

# Google
GOOGLE_SERVICE_ACCOUNT_EMAIL=
GOOGLE_PRIVATE_KEY=
GOOGLE_SPREADSHEET_ID=

# Slack
SLACK_BOT_TOKEN=xoxb-...
SLACK_CHANNEL_ONBOARDING=#onboarding
SLACK_CHANNEL_RESUMEN=#resumen-diario

# Airtable
AIRTABLE_API_KEY=
AIRTABLE_BASE_ID=
AIRTABLE_TABLE_NAME=

# Google Workspace (Admin SDK)
GOOGLE_ADMIN_EMAIL=admin@tudominio.com
GOOGLE_DOMAIN=tudominio.com
```

---

---

# 🔴 FLUJO 1 — CV-Scorer

**Descripción:** Un webhook recibe un PDF de CV, extrae el texto, lo
envía a Claude API con un schema JSON estructurado, y registra el score,
fortalezas y red flags en Google Sheets.

---

## Nodos del flujo (en orden)

| # | Nodo | Tipo | Descripción |
|---|------|------|-------------|
| 1 | `Webhook Trigger` | Webhook | Recibe el PDF por POST multipart/form-data |
| 2 | `Extract PDF Text` | HTTP Request / Code | Extrae texto plano del PDF |
| 3 | `Build Prompt` | Set | Construye el prompt para Claude con el texto del CV |
| 4 | `Claude API` | HTTP Request | Llama a Anthropic con structured output |
| 5 | `Parse JSON Response` | Code (JS) | Parsea el JSON devuelto por Claude |
| 6 | `Google Sheets — Append` | Google Sheets | Registra score, fortalezas y red flags |
| 7 | `Respond to Webhook` | Respond to Webhook | Devuelve el JSON de resultado al cliente |

---

## Schema JSON esperado (structured output de Claude)

Archivo: `01-cv-scorer/schema-output.json`

```json
{
  "score": 85,
  "nivel": "Senior",
  "fortalezas": [
    "5 años de experiencia en automatización",
    "Dominio de Python y n8n",
    "Proyectos publicados en GitHub"
  ],
  "red_flags": [
    "Sin experiencia en entornos enterprise",
    "No menciona trabajo en equipo"
  ],
  "recomendacion": "Avanzar a entrevista técnica"
}
```

---

## Prompt para Claude (nodo Build Prompt)

```
Eres un evaluador de CVs técnicos especializado en perfiles de automatización e integración.

Analiza el siguiente CV y responde ÚNICAMENTE con un JSON válido con esta estructura:
{
  "score": (número del 0 al 100),
  "nivel": ("Junior" | "Mid" | "Senior"),
  "fortalezas": [lista de strings],
  "red_flags": [lista de strings],
  "recomendacion": (string corto)
}

CV:
{{ $json["cv_text"] }}
```

---

## Columnas en Google Sheets

| Columna | Valor |
|---------|-------|
| Timestamp | `={{ $now.toISO() }}` |
| Nombre archivo | `={{ $json["filename"] }}` |
| Score | `={{ $json["score"] }}` |
| Nivel | `={{ $json["nivel"] }}` |
| Fortalezas | `={{ $json["fortalezas"].join(", ") }}` |
| Red Flags | `={{ $json["red_flags"].join(", ") }}` |
| Recomendación | `={{ $json["recomendacion"] }}` |

---

## Credenciales necesarias

- ✅ **Claude API** → HTTP Request con `x-api-key` header (Anthropic)
- ✅ **Google Sheets** → OAuth2 con cuenta de servicio

---

## Cómo exportar el flujo

En n8n: `Flujo → ··· (menú) → Download → Guardar como workflow.json`

---

---

# 🟡 FLUJO 2 — Onboarding Simulado

**Descripción:** Un formulario dispara la creación de un usuario en
Google Workspace, lo invita a Slack, notifica en un canal y registra
en Airtable.

---

## Nodos del flujo (en orden)

| # | Nodo | Tipo | Descripción |
|---|------|------|-------------|
| 1 | `n8n Form Trigger` | Form Trigger | Formulario: nombre, email, rol, área |
| 2 | `Create Google Workspace User` | HTTP Request | Admin SDK → crea cuenta corporativa |
| 3 | `Invite to Slack` | HTTP Request | Usa Slack SCIM API para invitar al email |
| 4 | `Notify Slack Channel` | Slack | Mensaje en #onboarding con datos del nuevo usuario |
| 5 | `Register in Airtable` | HTTP Request | Crea registro en tabla de empleados |
| 6 | `Send Welcome Email` | Gmail / SMTP | (Opcional) Email de bienvenida |

---

## Campos del formulario (nodo Form Trigger)

```
- Nombre completo      (text, requerido)
- Email corporativo    (email, requerido)
- Área / Departamento  (select: Ingeniería, Operaciones, Marketing, Ventas)
- Rol                  (text, requerido)
- Fecha de inicio      (date, requerido)
```

---

## HTTP Request — Crear usuario Google Workspace

```
Method: POST
URL: https://admin.googleapis.com/admin/directory/v1/users
Headers:
  Authorization: Bearer {{ $json["google_token"] }}
  Content-Type: application/json

Body:
{
  "primaryEmail": "{{ $json["email"] }}",
  "name": {
    "fullName": "{{ $json["nombre"] }}",
    "givenName": "{{ $json["nombre"].split(' ')[0] }}",
    "familyName": "{{ $json["nombre"].split(' ')[1] }}"
  },
  "password": "TempPass2024!",
  "changePasswordAtNextLogin": true
}
```

---

## Mensaje Slack — Canal #onboarding

```
🎉 *Nuevo miembro en el equipo*

👤 *Nombre:* {{ $json["nombre"] }}
📧 *Email:* {{ $json["email"] }}
🏢 *Área:* {{ $json["area"] }}
💼 *Rol:* {{ $json["rol"] }}
📅 *Inicio:* {{ $json["fecha_inicio"] }}

_Cuenta creada y accesos configurados automáticamente._
```

---

## Campos en Airtable

| Campo | Valor |
|-------|-------|
| Nombre | `{{ $json["nombre"] }}` |
| Email | `={{ $json["email"] }}` |
| Área | `={{ $json["area"] }}` |
| Rol | `={{ $json["rol"] }}` |
| Fecha Inicio | `={{ $json["fecha_inicio"] }}` |
| Estado | `Activo` |
| Creado en | `={{ $now.toISO() }}` |

---

## Credenciales necesarias

- ✅ **Google Admin SDK** → OAuth2 con cuenta de servicio (con permisos de dominio)
- ✅ **Slack Bot Token** → OAuth con scopes: `users:write`, `chat:write`
- ✅ **Airtable API Key** → Personal Access Token

---

---

# 🟢 FLUJO 3 — Resumidor Operativo

**Descripción:** Un cron diario lee datos de Google Sheets, genera un
resumen con Claude API y lo envía a Slack usando el formato Block Kit
de Slack.

---

## Nodos del flujo (en orden)

| # | Nodo | Tipo | Descripción |
|---|------|------|-------------|
| 1 | `Schedule Trigger` | Schedule | Cron diario a las 8:00 AM |
| 2 | `Read Google Sheets` | Google Sheets | Lee las últimas N filas del día anterior |
| 3 | `Format Data` | Code (JS) | Prepara los datos como lista para el prompt |
| 4 | `Claude API — Resumen` | HTTP Request | Genera el resumen operativo |
| 5 | `Build Block Kit` | Code (JS) | Arma el JSON de Slack Block Kit |
| 6 | `Send to Slack` | HTTP Request (Webhook) | Envía el mensaje formateado al canal |

---

## Cron Schedule

```
Campo: Cron Expression
Valor: 0 8 * * 1-5   ← 8:00 AM, lunes a viernes
```

---

## Prompt para Claude (nodo Claude API)

```
Eres un analista de operaciones. Se te entrega una tabla de registros del día anterior.

Genera un resumen ejecutivo en español con:
1. Total de registros procesados
2. Observaciones clave (máximo 3 puntos)
3. Alertas o anomalías detectadas
4. Recomendación de acción para hoy

Sé conciso. Usa bullet points. Máximo 150 palabras.

Datos:
{{ $json["datos_formateados"] }}
```

---

## Block Kit para Slack

```json
{
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "📊 Resumen Operativo — {{ $today }}"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "{{ $json['resumen_claude'] }}"
      }
    },
    {
      "type": "divider"
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "mrkdwn",
          "text": "_Generado automáticamente por n8n + Claude API_"
        }
      ]
    }
  ]
}
```

---

## Credenciales necesarias

- ✅ **Google Sheets** → OAuth2 (misma credencial del Flujo 1)
- ✅ **Claude API** → HTTP Request con `x-api-key`
- ✅ **Slack Incoming Webhook URL** → Webhook de app Slack

---

---

# 📄 README Principal del Repositorio

> Copia este contenido en el archivo `README.md` de la raíz del repo.

---

```markdown
# ⚡ n8n Flujos Emblemáticos

Colección de 3 flujos de automatización construidos con **n8n**,
**Claude API (Anthropic)** e integraciones con Google Workspace, Slack
y Airtable.

Proyecto desarrollado para demostrar habilidades en automatización de
procesos, integración de APIs y uso de IA en flujos operativos.

---

## 🗂️ Flujos incluidos

| # | Flujo | Descripción |
|---|-------|-------------|
| 1 | [CV-Scorer](./01-cv-scorer/) | Evalúa CVs con IA y registra resultados en Google Sheets |
| 2 | [Onboarding Simulado](./02-onboarding-simulado/) | Automatiza el alta de nuevos empleados en múltiples sistemas |
| 3 | [Resumidor Operativo](./03-resumidor-operativo/) | Genera resúmenes diarios con IA y los envía a Slack |

---

## 🛠️ Stack tecnológico

- **n8n** — Motor de automatización (self-hosted)
- **Claude API** — Anthropic, modelos claude-3-5-sonnet / claude-opus-4
- **Google Sheets** — Almacenamiento de datos tabulares
- **Google Workspace Admin SDK** — Gestión de usuarios corporativos
- **Slack API** — Notificaciones y Block Kit
- **Airtable** — Base de datos de operaciones

---

## 🚀 Cómo usar este repositorio

### 1. Clonar el repo

git clone https://github.com/tu-usuario/n8n-flujos-emblematicos.git
cd n8n-flujos-emblematicos

### 2. Configurar variables de entorno

cp .env.example .env
# Edita .env con tus claves reales

### 3. Importar un flujo en n8n

1. Abre tu instancia de n8n
2. Ve a **Workflows → Import from File**
3. Selecciona el archivo `workflow.json` del flujo deseado
4. Configura las credenciales según la documentación de cada flujo

---

## 📋 Requisitos previos

- n8n instalado y corriendo (v1.0+)
- Cuenta en Anthropic con API Key activa
- Credenciales de Google Cloud (OAuth2 / Service Account)
- Slack App con los scopes necesarios
- Cuenta de Airtable (para el flujo 2)

---

## 👤 Autor

**Tu Nombre**
[LinkedIn](https://linkedin.com/in/tu-perfil) · [GitHub](https://github.com/tu-usuario)

---

## 📝 Licencia

MIT
```

---

---

# ✅ Checklist de publicación en GitHub

Antes de hacer push, verifica:

- [ ] Cada carpeta de flujo tiene su propio `README.md`
- [ ] Los archivos `workflow.json` están exportados desde n8n
- [ ] Las capturas `diagram.png` están en la carpeta `assets/`
- [ ] El `.env.example` tiene todas las variables (sin valores reales)
- [ ] El `.gitignore` excluye el `.env` real
- [ ] El README principal tiene los links a cada subcarpeta funcionando
- [ ] Los flujos han sido probados end-to-end al menos una vez
- [ ] El repo es público en GitHub

---

# 🎯 Tip para la vacante

En tu CV o entrevista puedes describir cada flujo así:

> *"Implementé un pipeline de evaluación de CVs en n8n que integra
> webhooks, extracción de texto de PDFs, structured outputs con Claude
> API (Anthropic) y escritura automática en Google Sheets. El sistema
> procesa solicitudes en tiempo real y devuelve un score JSON con
> fortalezas y red flags del candidato."*

Esto demuestra: **manejo de APIs REST, diseño de flujos de datos,
uso práctico de LLMs y experiencia con herramientas de productividad.**
