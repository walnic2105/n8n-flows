# Flujo 02 — Onboarding Simulado

## Descripción

Flujo n8n que automatiza el alta de un nuevo colaborador: recibe los datos vía
formulario, consulta en **Airtable** el grupo de email del departamento,
**simula** la creación de la cuenta en **Google Workspace** (Admin SDK) y su
asignación al grupo, notifica al canal de Slack con Block Kit y registra el alta
en Airtable. Todas las fallas se capturan, se publican en un canal de errores y
quedan registradas con `estado=error` en la misma tabla.

> 🧪 **Modo demo**: los 2 nodos que llaman al Admin SDK están en modo **mock**
> (nodos `Set` que devuelven una respuesta estructuralmente idéntica a la del
> API real). Ver la sección [Modo demo vs producción](#modo-demo-vs-producción).

---

## Diagrama de nodos

| # | Nodo | Tipo | Función |
|---|------|------|---------|
| 1 | FormTrigger | `n8n-nodes-base.formTrigger` | Formulario con campos: nombre, apellido, email personal, departamento (dropdown), rol, fecha inicio, email del manager |
| 2 | LookupDepartmentGroup | `n8n-nodes-base.airtable` | Busca en tabla `Departamentos` el `groupEmail` y `slackChannel` correspondientes |
| 3 | GenerateCredentials | `n8n-nodes-base.code` | Normaliza nombres, genera `email corporativo` (`nombre.apellido@dominio`) y password temporal aleatorio |
| 4 | CreateWorkspaceUser 🧪 | `n8n-nodes-base.set` | **MOCK** — devuelve un payload con la misma forma que `POST /admin/directory/v1/users` (`kind`, `id`, `primaryEmail`, `name`, `isMailboxSetup`, `creationTime`) |
| 5 | AddUserToGroup 🧪 | `n8n-nodes-base.set` | **MOCK** — devuelve un payload con la misma forma que `POST /admin/directory/v1/groups/{groupEmail}/members` (`kind`, `id`, `email`, `role: MEMBER`, `status: ACTIVE`) |
| 6 | NotifyOnboardingChannel | `n8n-nodes-base.slack` | Publica mensaje con **Block Kit** en `#onboarding` (header + fields + credenciales) |
| 7 | LogToAirtable | `n8n-nodes-base.airtable` | Inserta fila en tabla `Colaboradores` con `estado=activo` |
| 8 | BuildResponse | `n8n-nodes-base.set` | Construye la respuesta final del formulario (`ok: true`, email generado, grupo) |
| 9 | ErrorHandler | `n8n-nodes-base.code` | Normaliza el payload de error y lo combina con los datos del formulario/credenciales |
| 10 | NotifyErrorChannel | `n8n-nodes-base.slack` | Publica el fallo en `#errores` con contexto (nombre, departamento, error) |
| 11 | LogErrorToAirtable | `n8n-nodes-base.airtable` | Registra el intento fallido con `estado=error` en `Colaboradores` |

Los nodos 2–7 enrutan su salida de error al `ErrorHandler`, que delega en
`NotifyErrorChannel` y `LogErrorToAirtable`. No existe un `Respond to Webhook`
porque el `FormTrigger` usa `responseMode: lastNode` y cierra con `BuildResponse`.

![Diagrama](assets/diagram.png)

---

## Credenciales necesarias

| Credencial en n8n | Servicio | Alcance / Scopes |
|-------------------|----------|------------------|
| `AirtableToken` | Airtable Personal Access Token | Scopes `data.records:read`, `data.records:write` sobre la base del portafolio |
| `SlackBot` | Slack OAuth2 / Bot Token | `chat:write`, `chat:write.public` (para escribir sin estar en el canal) |

> En modo demo **no se requiere credencial de Google**. Los nodos
> `CreateWorkspaceUser` y `AddUserToGroup` son mocks. Para el modo producción,
> ver la sección [Modo demo vs producción](#modo-demo-vs-producción).

---

## Variables de entorno

| Variable | Descripción |
|----------|-------------|
| `GOOGLE_WORKSPACE_DOMAIN` | Dominio usado para construir los emails corporativos (`nombre.apellido@<dominio>`). En modo demo puede ser cualquier string (ej. `demo.tudominio.com`); en producción debe ser un dominio verificado en Cloud Identity/Workspace. |
| `AIRTABLE_BASE_ID` | ID de la base Airtable (empieza con `app...`) |
| `AIRTABLE_TABLE_DEPARTAMENTOS` | ID o nombre de la tabla `Departamentos` |
| `AIRTABLE_TABLE_COLABORADORES` | ID o nombre de la tabla `Colaboradores` |
| `SLACK_CHANNEL_ONBOARDING` | ID del canal Slack de bienvenida (ej. `C0123456789`) |
| `SLACK_CHANNEL_ERRORS` | ID del canal Slack donde se publican fallos |

---

## Estructura esperada de Airtable

### Tabla `Departamentos`

| Campo | Tipo | Ejemplo |
|-------|------|---------|
| `nombre` | Single line text | `engineering` |
| `groupEmail` | Email | `engineering@demo.tudominio.com` |
| `slackChannel` | Single line text | `C01ABC23456` |

### Tabla `Colaboradores`

| Campo | Tipo |
|-------|------|
| `nombre`, `apellido` | Single line text |
| `emailCorporativo`, `emailPersonal`, `managerEmail` | Email |
| `departamento`, `rol` | Single line text |
| `fechaInicio`, `createdAt` | Date / Date with time |
| `estado` | Single select: `activo`, `error` |

---

## Ejemplo de input

El formulario se rellena en `https://<tu-n8n>/form/<form-path>` con los campos:

```
Nombre:            Laura
Apellido:          Gómez
Email personal:    laura.gomez.personal@gmail.com
Departamento:      engineering
Rol:               Software Engineer
Fecha de inicio:   2026-05-05
Email del manager: carlos.ruiz@demo.tudominio.com
```

---

## Ejemplo de output

Respuesta final en el formulario:

```json
{
  "ok": true,
  "emailCorporativo": "laura.gomez@demo.tudominio.com",
  "departamento": "engineering",
  "grupoAsignado": "engineering@demo.tudominio.com",
  "mensaje": "Colaborador registrado correctamente. Revisa Slack para la bienvenida."
}
```

Mensaje en `#onboarding` (Block Kit):

```
🎉 Nuevo colaborador: Laura Gómez
Rol: Software Engineer     Departamento: engineering
Fecha de inicio: 2026-05-05     Manager: carlos.ruiz@demo.tudominio.com
Email corporativo: laura.gomez@demo.tudominio.com
Grupo asignado:    engineering@demo.tudominio.com
```

### Output de error

Mensaje en `#errores`:

```
⚠️ Fallo en onboarding — Laura Gómez (engineering)
Error: `Request failed with status code 409: Entity already exists.`
Timestamp: 2026-04-17T14:20:33.112Z
```

Fila en Airtable con `estado=error`.

---

## Instrucciones de importación en n8n

1. **Workflows → Import from file** → seleccionar [workflow.json](workflow.json).
2. Crear dos credenciales en n8n con los nombres exactos:
   `AirtableToken` y `SlackBot`.
3. Poblar la tabla `Departamentos` en Airtable con los grupos de cada área.
4. Crear los canales Slack y obtener sus IDs (click derecho sobre el canal →
   Copy link → el último segmento).
5. Definir las variables de entorno en Dokploy (o en el `.env` del
   `docker-compose`).
6. Activar el workflow y probar enviando el formulario.

---

## Modo demo vs producción

El flujo está diseñado para **demostrar el pipeline completo** sin requerir una
cuenta real de Google Workspace / Cloud Identity. Los nodos que llaman al Admin
SDK se implementan como mocks usando nodos `Set`, que devuelven un payload con
**exactamente la misma forma** que la respuesta real del API. Esto permite que
el resto del flujo (Airtable, Slack, Block Kit, error handling) funcione de
forma idéntica tanto en demo como en producción.

### Cómo reemplazar los mocks por llamadas reales

Para pasar a producción, sustituir los nodos 4 y 5 por nodos `HTTP Request`
(`n8n-nodes-base.httpRequest`, `typeVersion: 4.4`) con esta configuración:

**`CreateWorkspaceUser`** (reemplaza el `Set` actual):

```
Method:         POST
URL:            https://admin.googleapis.com/admin/directory/v1/users
Auth:           Predefined Credential Type → Google API (credencial GoogleAdminSDK)
Body (JSON):    {
                  "primaryEmail": "{{ $json.emailCorporativo }}",
                  "name": { "givenName": "{{ $json.nombre }}", "familyName": "{{ $json.apellido }}" },
                  "password": "{{ $json.passwordTemporal }}",
                  "changePasswordAtNextLogin": true,
                  "organizations": [{ "department": "{{ $json.departamento }}", "title": "{{ $json.rol }}", "primary": true }]
                }
Retry:          retryOnFail=true, maxTries=3, waitBetweenTries=2000
```

**`AddUserToGroup`** (reemplaza el `Set` actual):

```
Method:         POST
URL:            https://admin.googleapis.com/admin/directory/v1/groups/{{ encodeURIComponent($('GenerateCredentials').item.json.groupEmail) }}/members
Auth:           Predefined Credential Type → Google API (credencial GoogleAdminSDK)
Body (JSON):    {
                  "email": "{{ $('GenerateCredentials').item.json.emailCorporativo }}",
                  "role": "MEMBER"
                }
Retry:          retryOnFail=true, maxTries=3, waitBetweenTries=2000
```

### Credencial y scopes que se añaden en producción

| Credencial en n8n | Servicio | Scopes OAuth2 |
|-------------------|----------|---------------|
| `GoogleAdminSDK` | Google API (OAuth2) | `https://www.googleapis.com/auth/admin.directory.user` y `https://www.googleapis.com/auth/admin.directory.group.member` |

Autenticar con un usuario **admin** del dominio verificado en Cloud Identity /
Workspace. Para un entorno productivo robusto se recomienda **Service Account
con Domain-Wide Delegation**, pero el nodo HTTP nativo de n8n no firma JWTs de
service account — en ese caso se usa el nodo `Google Workspace Admin` (beta) o
un sub-workflow que firme el JWT.

### Qué NO cambia al pasar a producción

- `FormTrigger`, `LookupDepartmentGroup`, `GenerateCredentials`, `NotifyOnboardingChannel`, `LogToAirtable`, `BuildResponse`
- Todos los nodos de la rama de error (`ErrorHandler`, `NotifyErrorChannel`, `LogErrorToAirtable`)
- Las conexiones, el manejo de errores centralizado y las expresiones

---

## Notas de diseño

- **Fuente de verdad externa**: la tabla `Departamentos` permite que RRHH agregue
  áreas nuevas sin tocar el flujo. Es el punto de configuración.
- **Idempotencia parcial (en producción)**: si `CreateWorkspaceUser` falla con
  `409` (usuario ya existe), el error se captura limpiamente. Una mejora futura
  es hacer un `GET` previo y ramificar con un `IF`.
- **Password temporal**: se genera en el nodo `GenerateCredentials` con
  `crypto.getRandomValues` (no `Math.random`) y se fuerza el cambio al primer
  login con `changePasswordAtNextLogin: true`. En este portafolio **no** se
  envía por email — en producción se enviaría al email personal con un enlace de
  primer inicio de sesión.
- **Block Kit**: el mensaje de `#onboarding` usa `header`, `section` con
  `fields`, y `context` para separar la bienvenida del detalle técnico.
- **Error handling centralizado**: todos los nodos críticos (2–7) convergen en
  `ErrorHandler`, que compone un payload normalizado incluso cuando el fallo
  ocurre antes de generar credenciales.
- **Retry en HTTP (en producción)**: los dos nodos que llaman al Admin SDK
  deben usar `retryOnFail` x3 con 2s entre intentos para absorber fallos
  transitorios de la API de Google. En modo demo no aplica.
