# Wiki NOC — Servidor IA

Backend pequeño en Node.js/Express que expone un endpoint `/api/chat` consumido por el panel de chat IA embebido en `index.html`. Usa **Groq SDK** (modelo `llama-3.3-70b-versatile` por defecto) para responder consultas de los operadores con **TODO el contenido de la wiki** como contexto.

## Por qué un backend aparte

La wiki es un único HTML estático (`index.html`) que se puede abrir desde `file://`, servir con cualquier HTTP server, o publicar en GitHub Pages. Pero el SDK `groq-sdk` **sólo funciona en backend** (necesita la API key de Groq que no puede exponerse en el navegador). Por eso el chat IA hace fetch a `http://127.0.0.1:8787/api/chat`, y ese endpoint es el que habla con el LLM.

## Instalación

```bash
cd server
npm install
```

Requiere Node.js >= 18.

## Cómo correrlo

1. Obtené tu API key de Groq en https://console.groq.com/keys (es gratis, sólo requiere login con cuenta Google/GitHub).
2. Exportala como variable de entorno:

```bash
export GROQ_API_KEY="gsk_tu_api_key_aqui"
```

3. Arrancá el server:

```bash
npm start
```

Por defecto escucha en `http://127.0.0.1:8787` y usa el modelo `llama-3.3-70b-versatile`. Para cambiar puerto, host o modelo:

```bash
GROQ_API_KEY="gsk_..." \
GROQ_MODEL="llama-3.1-8b-instant" \
WIKINOC_AI_PORT=9000 \
WIKINOC_AI_HOST=0.0.0.0 \
npm start
```

Modelos disponibles en Groq (a jul/2026):
- `llama-3.3-70b-versatile` (default, mejor calidad)
- `llama-3.1-8b-instant` (más rápido, peor razonamiento)
- `mixtral-8x7b-32768` (contexto largo de 32k)
- `gemma2-9b-it`

Mientras el servidor esté corriendo, el botón flotante de IA en la wiki va a mostrar un punto verde y las consultas van a funcionar. Si el servidor no está levantado, la wiki sigue funcionando normalmente (sólo se ve un punto rojo en el botón y aparece un mensaje de error al intentar consultar).

## Endpoints

### `GET /api/health`

Health check. Devuelve `{ok: true, sdk: 'groq', model: 'llama-3.3-70b-versatile', port: 8787}` si el SDK está inicializado.

### `POST /api/chat`

Body:
```json
{
  "messages": [{"role": "user", "content": "..."}, ...],
  "wikiContext": "=== WIKI NOC — 16 entradas ===\n--- ENTRADA: RNX1221 ---\n...",
  "question": "El job BCHAGENTES está en MSGW, ¿qué hago?"
}
```

Respuesta:
```json
{
  "response": "Según la entrada BCHAGENTES-MSGW…",
  "usage": {"prompt_tokens": 837, "completion_tokens": 171, "total_tokens": 1008},
  "model": "llama-3.3-70b-versatile",
  "conversationId": "20260716…",
  "timestamp": "2026-07-15T16:07:40.415Z"
}
```

### `POST /api/export`

Recibe `{conversation, format, ticket, operator}` y devuelve un `.txt` o `.json` formateado para descargar. (Aunque el frontend también puede generar el export localmente, este endpoint queda disponible por si se quiere integrar con otro sistema.)

## Cómo construye el contexto el frontend

Cuando el operador manda una pregunta, el frontend (`index.html`) serializa **todas** las entradas visibles — `BASE_ENTRIES` más las entradas custom del `localStorage` — en un texto plano con este formato:

```
=== WIKI NOC — 16 entradas ===

--- ENTRADA: RNX1221 ---
Título: RNX1221 — Actualización o supresión en archivo sin…
Categoría: Errores de Archivo / Operación
Sistemas: Pinot, Semillón
Severidad: warning
Mensaje: …
Causa: …
Programas: P57608, PTRFSTY@J
Archivos: F57607LE, F550040B
WORKAROUND ACTIVO hasta 21/07/2026: Responder al mensaje con G…
  Fuente: Ricardo Caldeiro — mail 15/07/2026 (INC 0041043)
Resolución:
  1. Responder con G (continuar)…
  2. En algunos casos puntuales puede requerirse C…
Casos documentados:
  • 2026-07-15 | Ticket: INC 0041043 | Job: BCHAGENTES / 631095
    Acción: …

--- ENTRADA: BCHAGENTES-MSGW ---
…
```

Ese texto viaja en el campo `wikiContext` del body. El backend lo inyecta en el system prompt del LLM, junto con reglas de comportamiento (responder en español rioplatense, citar IDs de entradas, respetar workarounds activos, etc.).

## System prompt

El system prompt completo está en `server.js` → `buildSystemPrompt(wikiContext)`. Las reglas clave son:

- Respondé en español rioplatense (vos).
- Citar entradas por ID (ej: "Según la entrada BCHAGENTES-MSGW…").
- Respetar workarounds activos y mencionar fecha de vencimiento.
- Para errores críticos, recordar SIEMPRE llamar a Ricardo Caldeiro antes de actuar.
- No inventar pasos: si la wiki no lo cubre, decirlo y sugerir escalar.
- Estructura: Diagnóstico → Acción inmediata → Verificación → Escalado.
- Comandos AS400 en mayúsculas y monoespaciados (`WRKJOB`, `ENDJOB`).
- Mencionar tickets documentados (INC 0041043, SR0177063…) para trazabilidad.

## Salidas hacia el exterior

El panel de chat tiene 4 acciones de salida:

1. **Copiar** — copia la última respuesta al portapapeles (para pegar en mail o ticket).
2. **Exportar .txt** — descarga la conversación completa como `.txt` formateado con separadores.
3. **Exportar .json** — descarga un JSON con metadata (`source`, `version`, `exportedAt`, `entryCount`, `messages`) pensado para integraciones externas (POST a un sistema de tickets, webhook, etc.).
4. **Generar reporte** — descarga un `.txt` con formato de **reporte de incidencia**: consulta del operador, respuesta de la IA, conversación completa, disclaimer. Listo para adjuntar al ticket.

## Debugging

Desde la consola del navegador (con la wiki abierta):

```javascript
// Ver conversación actual
__wikinocAI.getConversation();

// Ver estado del backend
__wikinocAI.getBackendStatus();

// Ver el contexto que se le manda al LLM
__wikinocAI.buildWikiContext();

// Mandar una consulta programáticamente
__wikinocAI.send('¿Qué hago con RNX1221?');
```

Para probar el backend desde la terminal:

```bash
# Health
curl http://127.0.0.1:8787/api/health

# Chat
curl -X POST http://127.0.0.1:8787/api/chat \
  -H 'Content-Type: application/json' \
  -d '{
    "wikiContext": "Entrada RNX1221: …",
    "messages": [],
    "question": "¿Qué hago con RNX1221?"
  }'
```

## Arquitectura

```
┌─────────────────────────────┐
│  Navegador (index.html)     │
│                             │
│  Wiki HTML estática         │
│  + Panel de chat IA         │
│  + Serialización de contexto│
│                             │
└──────────┬──────────────────┘
           │ POST /api/chat
           │ {wikiContext, messages, question}
           ▼
┌─────────────────────────────┐
│  Backend Node.js (server.js)│
│  http://127.0.0.1:8787      │
│                             │
│  System prompt + contexto   │
│  → groq-sdk                 │
│  → llama-3.3-70b-versatile  │
│                             │
└─────────────────────────────┘
```

## Seguridad

- El backend escucha en `127.0.0.1` por defecto (sólo accesible desde la misma máquina). Si lo exponés con `WIKINOC_AI_HOST=0.0.0.0`, aseguralo con un proxy reverso con auth.
- El system prompt no expone la API key de Groq.
- La API key se lee de la variable de entorno `GROQ_API_KEY` (nunca se hardcodea ni se loguea).
- No hay persistencia de conversaciones del lado del backend — el historial vive en el `localStorage` del navegador (a través del frontend) y sólo se manda al backend cuando se hace una nueva consulta.
