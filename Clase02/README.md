# Clase 02: Extracción de Datos y Triage con LLMs

En esta clase se profundiza en el uso de modelos de lenguaje (LLMs) para procesar información no estructurada y convertirla en datos útiles para flujos de trabajo.

---

## 📚 Conceptos Clave

### Structured Output (Salida Estructurada)
Cuando le pedís a un LLM que responda en un formato específico (JSON, categorías fijas, campos definidos), estás haciendo **structured output**. La clave está en el prompt: si le decís "respondé SOLO con un JSON con los campos X, Y, Z", el modelo se adapta.

### Prompt Engineering para Extracción
| Técnica | Descripción | Ejemplo |
|:---|:---|:---|
| **Role prompting** | Darle un rol al modelo | "Sos un médico de guardia..." |
| **Few-shot** | Dar ejemplos de input/output esperado | "Ejemplo: 'me duele la cabeza' → LEVE" |
| **Output format** | Definir el formato exacto de respuesta | "Respondé SOLO con un JSON: {urgencia, motivo}" |
| **Constraints** | Limitar las opciones | "Solo podés responder: LEVE, GRAVE o URGENTE" |

### Triage con IA
El triage es la clasificación por urgencia. En el contexto del curso, usamos un LLM para:
1. **Recibir** síntomas en lenguaje natural
2. **Clasificar** la urgencia (LEVE / GRAVE / URGENTE)
3. **Extraer** datos estructurados (síntomas, área del cuerpo, recomendación)
4. **Registrar** el resultado en Google Sheets o base de datos

### Webhooks como Entrada de Datos
Un webhook es una URL que "escucha". Cuando alguien le manda un POST con datos, se dispara el workflow. Es la forma más común de conectar un chatbot, un formulario web o una app externa con n8n.

---

## 🧪 Workflows

### Práctica principal: Asistente de Triage Médico
**Archivo**: [`Clase-02.json`](./Clase-02.json)

Flujo que recibe descripciones de síntomas vía Webhook, usa Groq (Llama 3) para categorizar la urgencia (LEVE, GRAVE, URGENTE) y guarda el resultado en Google Sheets.

### Ejemplo adicional: Extracción de Datos de Texto
**Archivo**: [`ejemplo-extraccion-datos.json`](./ejemplo-extraccion-datos.json)

Workflow que recibe un texto libre (email, mensaje, nota) y extrae campos estructurados (nombre, fecha, asunto, sentimiento, prioridad) en formato JSON usando un LLM.

```bash
curl -X POST http://localhost:5678/webhook-test/extraer-datos \
  -H "Content-Type: application/json" \
  -d '{"texto": "Hola, soy Juan Pérez. Necesito cancelar mi turno del martes 15 de abril. Gracias."}'
```

---

## �️ Tutorial: Configurar Groq (LLM Gratis y Rápido)

Groq te da acceso gratuito a Llama 3 con una velocidad impresionante. Es la mejor opción para practicar sin gastar un peso.

### 🟢 Paso 1: Crear tu cuenta y API Key en Groq

1. Entrá a [https://console.groq.com](https://console.groq.com)
2. Hacé clic en **"Sign Up"** o logueate con tu cuenta de Google
3. Una vez adentro, en el menú de la izquierda hacé clic en **"API Keys"**
4. Hacé clic en el botón naranja **"Create API Key"**
5. Poné un nombre (ej: `Clase-Coderhouse`) y hacé clic en **"Submit"**
6. Copiá la key que empieza con `gsk_...` — **guardala en un lugar seguro**

### 🔵 Paso 2: Configurar Groq en n8n

Groq usa el mismo formato que OpenAI, así que n8n lo reconoce como si fuera OpenAI.

1. Abrí n8n → **http://localhost:5678**
2. Menú izquierdo → **"Credentials"** (🔑)
3. Click en **"Add Credential"** → buscá **"OpenAI"**
4. En el campo **"API Key"**: pegá tu key de Groq (`gsk_...`)
5. **IMPORTANTE**: Hacé clic en **"Base URL"** (o "Override Base URL") y poné:
   ```
   https://api.groq.com/openai/v1
   ```
6. Hacé clic en **"Save"**
7. Cuando uses un nodo de OpenAI en un workflow, en el campo **"Model"** elegí: `llama3-70b-8192`

> 💡 **Tip**: Podés tener varias credenciales — una de OpenAI "real" y otra de Groq. Así elegís cuál usar en cada workflow.

### 🟣 Paso 3: Conectar Google Sheets a n8n

En el workflow de triage, los resultados se guardan en una planilla de Google Sheets.

1. En n8n, cuando encuentres el nodo de **Google Sheets** con un ⚠️, hacé clic y luego en **"Create New Credential"**
2. Seleccioná **"Google Sheets OAuth2 API"**
3. n8n te va a pedir que autorices con tu cuenta de Google — seguí los pasos del navegador
4. Aceptá los permisos y volvé a n8n
5. En la configuración del nodo, pegá la **URL de tu Google Sheet** (ej: `https://docs.google.com/spreadsheets/d/TU_ID_ACA/edit`)
6. Seleccioná la **hoja** (Sheet) y hacé clic en **"Save"**

> ⚠️ **Error "Could not connect"**: Si n8n no puede conectarse a Google, es probable que tengas que configurar OAuth2 con un proyecto en Google Cloud Console. Una alternativa más simple: usá la credencial **"Google Sheets API (Service Account)"** con un archivo JSON de servicio.

> ⚠️ **Error "Spreadsheet not found"**: Verificá que la URL del sheet esté correcta y que la cuenta de Google que autorizaste tenga acceso a esa planilla.

### 🔧 Probá el Workflow de Triage

1. Importá [`Clase-02.json`](./Clase-02.json) en n8n (Ctrl+V)
2. Configurá las credenciales de Groq/OpenAI y Google Sheets en los nodos marcados con ⚠️
3. Hacé clic en **"Test Workflow"**
4. Mandá esta consulta de prueba:

```bash
curl -X POST http://localhost:5678/webhook-test/triage -H "Content-Type: application/json" -d "{\"sintomas\": \"Me duele mucho el pecho y me cuesta respirar\"}"
```

5. Revisá tu Google Sheet — debería aparecer una nueva fila con la urgencia clasificada

---

## �📄 Materiales de la Clase
- **Presentación**: `IA Automation-02.pdf`
- **Temas Teóricos**:
  - `semana 1 - tema 3.pdf`
  - `Semana 1 - tema 4.pdf`
