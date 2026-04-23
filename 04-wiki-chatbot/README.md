# Flujo 04 — Wiki Chatbot (RAG Agent)

## Descripción

Sistema de chatbot inteligente con **RAG (Retrieval Augmented Generation)** que
responde preguntas sobre el funcionamiento del **Showroom Marketplace** usando la
wiki interna como base de conocimiento. El sistema consta de dos workflows
complementarios:

- **Chatbot (RAG Agent)** — agente conversacional con Claude + vector search.
- **Ingestion Pipeline** — pipeline que indexa los archivos `.md` de la wiki en
  Supabase pgvector.

---

## Arquitectura

```
┌─────────────────────────────────────────────────────────┐
│  INGESTION PIPELINE (ejecución manual / bajo demanda)   │
│                                                         │
│  Google Drive (.md) → Loop → Download → Text Splitter   │
│                        ↑       → OpenAI Embeddings      │
│                        │       → Supabase pgvector      │
│                        └───────────────┘                │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  CHATBOT (RAG AGENT)                                    │
│                                                         │
│  Chat Trigger → AI Agent                                │
│                   ├── LLM: Claude Sonnet 4              │
│                   ├── Memory: Buffer (10 msgs)          │
│                   └── Tool: Supabase Vector Store       │
│                             └── OpenAI Embeddings       │
└─────────────────────────────────────────────────────────┘
```

---

## Diagrama de nodos — Chatbot

| # | Nodo | Tipo | Función |
|---|------|------|---------|
| 1 | Chat Trigger | `@n8n/n8n-nodes-langchain.chatTrigger` | Interfaz de chat pública para recibir preguntas del usuario |
| 2 | AI Agent | `@n8n/n8n-nodes-langchain.agent` | Orquesta la búsqueda RAG y genera respuestas con system prompt especializado |
| 3 | Claude Sonnet 4 | `@n8n/n8n-nodes-langchain.lmChatAnthropic` | LLM para razonamiento y generación de respuestas (temp: 0.3, max: 4096 tokens) |
| 4 | Memory | `@n8n/n8n-nodes-langchain.memoryBufferWindow` | Memoria conversacional de ventana (últimos 10 mensajes) |
| 5 | Wiki Showroom | `@n8n/n8n-nodes-langchain.vectorStoreSupabase` | Vector store como herramienta del agente (top-K: 6 chunks) |
| 6 | OpenAI Embeddings | `@n8n/n8n-nodes-langchain.embeddingsOpenAi` | Genera embeddings de la query para buscar en el vector store |

## Diagrama de nodos — Ingestion Pipeline

| # | Nodo | Tipo | Función |
|---|------|------|---------|
| 1 | Manual Trigger | `n8n-nodes-base.manualTrigger` | Ejecución manual bajo demanda |
| 2 | List Wiki Files | `n8n-nodes-base.googleDrive` | Lista todos los archivos `.md` del folder en Google Drive |
| 3 | Loop Over Items | `n8n-nodes-base.splitInBatches` | Procesa los archivos uno por uno |
| 4 | Download File | `n8n-nodes-base.googleDrive` | Descarga cada archivo `.md` como binario |
| 5 | Supabase Vector Store | `@n8n/n8n-nodes-langchain.vectorStoreSupabase` | Inserta los chunks con embeddings en la tabla `documents` |
| 6 | Document Loader | `@n8n/n8n-nodes-langchain.documentDefaultDataLoader` | Carga el archivo binario y extrae texto con metadata (source) |
| 7 | Text Splitter | `@n8n/n8n-nodes-langchain.textSplitterRecursiveCharacterTextSplitter` | Divide el texto en chunks de ~1000 chars con overlap de 200 |
| 8 | OpenAI Embeddings | `@n8n/n8n-nodes-langchain.embeddingsOpenAi` | Genera embeddings (`text-embedding-3-small`) para cada chunk |

---

## Credenciales necesarias

| Credencial en n8n | Servicio | Uso |
|-------------------|----------|-----|
| `Anthropic account` | Anthropic API | LLM del agente (Claude Sonnet 4) |
| `Supabase account` | Supabase | Vector store pgvector (host + service_role key) |
| `OpenAi account` | OpenAI | Embeddings `text-embedding-3-small` |
| `Google Drive account` | Google Drive OAuth2 | Lectura de archivos de la wiki (solo ingestion) |

---

## Variables de entorno

| Variable | Descripción |
|----------|-------------|
| `SUPABASE_URL` | URL del proyecto Supabase (`https://{project-id}.supabase.co`) |
| `SUPABASE_SERVICE_ROLE_KEY` | Service role key de Supabase (acceso completo, bypassea RLS) |
| `OPENAI_API_KEY` | API key de OpenAI para embeddings |

---

## Requisitos previos — Supabase

Ejecutar en **SQL Editor** de Supabase:

```sql
-- 1. Extensión pgvector
CREATE EXTENSION IF NOT EXISTS vector;

-- 2. Tabla de documentos
CREATE TABLE IF NOT EXISTS documents (
  id bigserial PRIMARY KEY,
  content text,
  metadata jsonb,
  embedding vector(1536)
);

-- 3. Índice para búsquedas eficientes
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops)
  WITH (lists = 100);

-- 4. Función de búsqueda por similitud
CREATE OR REPLACE FUNCTION match_documents (
  query_embedding vector(1536),
  match_count int DEFAULT 5,
  filter jsonb DEFAULT '{}'
) RETURNS TABLE (
  id bigint,
  content text,
  metadata jsonb,
  similarity float
) LANGUAGE plpgsql AS $$
BEGIN
  RETURN QUERY
  SELECT
    documents.id,
    documents.content,
    documents.metadata,
    1 - (documents.embedding <=> query_embedding) AS similarity
  FROM documents
  WHERE documents.metadata @> filter
  ORDER BY documents.embedding <=> query_embedding
  LIMIT match_count;
END;
$$;
```

> ⚠️ **Importante**: El **Data API** de Supabase debe estar **habilitado**
> (Project Settings → API). n8n se conecta vía PostgREST.

---

## Instrucciones de importación en n8n

### Paso 1 — Ingestion Pipeline
1. En n8n: **Workflows → Import from file** → seleccionar
   [workflow-ingestion.json](workflow-ingestion.json).
2. Crear credenciales: `Google Drive account` (OAuth2), `Supabase account`,
   `OpenAi account`.
3. Editar el nodo **"List Wiki Files"** y reemplazar `FOLDER_ID_AQUI` por el ID
   del folder de Google Drive con los archivos `.md`.
4. Ejecutar manualmente. Procesará todos los archivos y los indexará.

### Paso 2 — Chatbot
1. En n8n: **Workflows → Import from file** → seleccionar
   [workflow-chatbot.json](workflow-chatbot.json).
2. Vincular credenciales: `Anthropic account`, `Supabase account`,
   `OpenAi account`.
3. Activar el workflow.
4. Abrir el Chat Trigger (URL pública) y probar con una pregunta como:
   *"¿Cuál es el proceso de incorporación de una marca?"*

---

## Notas de diseño

- **Modelo LLM**: Claude Sonnet 4, temperatura 0.3 para respuestas consistentes
  y factuales.
- **Embeddings**: OpenAI `text-embedding-3-small` (1536 dimensiones) — balance
  óptimo entre calidad y costo.
- **Chunking**: 1000 caracteres con overlap de 200 para preservar contexto entre
  fragmentos.
- **Top-K**: 6 chunks recuperados por query — suficiente para cubrir contexto sin
  exceder la ventana del LLM.
- **Memory**: Buffer de 10 mensajes para mantener coherencia conversacional sin
  consumir tokens excesivos.
- **System Prompt**: Incluye reglas de respuesta en español, citación de fuentes
  por nombre de archivo, y fallback explícito cuando no encuentra información.
- **Metadata**: Cada chunk almacena el nombre del archivo fuente para trazabilidad.
