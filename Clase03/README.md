# Clase 03: RAG (Retrieval-Augmented Generation) y Bases de Datos Vectoriales

Esta clase se enfoca en cómo dotar a la IA de conocimiento específico utilizando técnicas de RAG para consultar documentos propios.

---

## 📚 Conceptos Clave

### ¿Qué es RAG?
**Retrieval-Augmented Generation** es una técnica que combina:
1. **Retrieval** (Búsqueda): Buscar información relevante en tus documentos
2. **Augmented** (Aumento): Agregar esa información al prompt del LLM
3. **Generation** (Generación): El LLM responde basándose en TUS datos, no solo en su entrenamiento

> Sin RAG, el LLM solo sabe lo que aprendió en su entrenamiento. Con RAG, le das "memoria" de tus documentos.

### Embeddings (Vectores de Texto)
Un **embedding** es la representación numérica de un texto. Convierte palabras/frases en una lista de números (vector) que captura su **significado semántico**.

| Texto | Vector (simplificado) | Significado |
|:---|:---|:---|
| "dolor de cabeza" | [0.8, 0.2, 0.1, ...] | Síntoma leve |
| "cefalea intensa" | [0.78, 0.22, 0.12, ...] | ¡Casi idéntico! Son sinónimos |
| "auto deportivo" | [0.1, 0.9, 0.7, ...] | Totalmente diferente |

### Chunking (Fragmentación)
Los documentos largos se dividen en pedazos más chicos (**chunks**) antes de convertirlos en vectores. Esto permite buscar partes específicas sin mandar todo el documento al LLM.

| Estrategia | Descripción | Cuándo usarla |
|:---|:---|:---|
| **Fixed Size** | Corta cada N caracteres | Textos simples |
| **Recursive Character** | Corta respetando párrafos/oraciones | La más usada (recomendada) |
| **Semantic** | Corta por cambio de tema | Documentos largos y complejos |

### Vector Store (Base de Datos Vectorial)
Es donde se guardan los embeddings. Cuando el usuario pregunta algo, se convierte su pregunta en vector y se buscan los chunks más parecidos (vecinos más cercanos).

**Pipeline completo de RAG:**
```
[Documento] → [Chunking] → [Embeddings] → [Vector Store]
                                                  ↑
[Pregunta del usuario] → [Embedding] → [Búsqueda] → [Contexto relevante] → [LLM] → [Respuesta]
```

---

## 🧪 Workflows

### Práctica principal: Implementación de RAG con n8n
Los flujos demuestran el proceso completo:
1. **Carga de Documentos**: Uso de cargadores de datos
2. **Fragmentación (Chunking)**: Recursive Character Text Splitter
3. **Embeddings**: Integración con Google Gemini
4. **Almacenamiento**: Vector Stores simples para recuperación

- **Archivos de flujo**:
  - [`AI AUTOMATION-03.01.json`](./AI%20AUTOMATION-03.01.json)
  - [`AI AUTOMATION-03.02.json`](./AI%20AUTOMATION-03.02.json)
  - [`AI AUTOMATION-03.03.json`](./AI%20AUTOMATION-03.03.json)

### Ejemplo adicional: RAG Básico con Pregunta-Respuesta
**Archivo**: [`ejemplo-rag-basico.json`](./ejemplo-rag-basico.json)

Workflow simplificado que:
1. Recibe una pregunta por webhook
2. Busca en un vector store en memoria
3. Usa el contexto encontrado para responder con el LLM

```bash
curl -X POST http://localhost:5678/webhook-test/rag-pregunta \
  -H "Content-Type: application/json" \
  -d '{"pregunta": "¿Cuál es el protocolo para dolor de pecho?"}'
```

---

## �️ Tutorial: Montar tu Primer RAG (Paso a Paso)

### 🟢 Paso 1: Instalar Ollama y los Modelos Necesarios

Para RAG necesitás dos cosas: un modelo que **genere texto** (Llama 3) y un modelo que **genere embeddings** (Nomic Embed Text).

1. Si todavía no tenés Ollama, descargalo de [https://ollama.com/download](https://ollama.com/download) e instalalo
2. Abrí una terminal y descargá los modelos:
   ```bash
   ollama pull llama3
   ollama pull nomic-embed-text
   ```
3. Verificá que estén instalados:
   ```bash
   ollama list
   ```
   Deberías ver ambos modelos en la lista.

> ⚠️ **¿Cuánto espacio ocupan?** Llama 3 pesa ~4.7 GB y Nomic Embed Text ~274 MB. Asegurate de tener espacio en disco.

### 🔵 Paso 2: Configurar las Credenciales de Ollama en n8n

1. Abrí n8n → **http://localhost:5678**
2. Menú izquierdo → **"Credentials"** (🔑)
3. Click en **"Add Credential"** → buscá **"Ollama"**
4. En el campo **"Base URL"**, poné:
   ```
   http://host.docker.internal:11434
   ```
   (NO uses `localhost` — n8n corre dentro de Docker y `localhost` apunta al contenedor, no a tu PC)
5. Hacé clic en **"Save"**

> ⚠️ **Error "ECONNREFUSED"**: Significa que Ollama no está corriendo. Verificá que el ícono de Ollama esté en tu barra de notificaciones, o corré `ollama serve` en una terminal.

### 🟣 Paso 3: Configurar Google Gemini Embeddings (Alternativa Cloud)

Si preferís usar embeddings en la nube en vez de Ollama:

1. Entrá a [https://aistudio.google.com/apikey](https://aistudio.google.com/apikey)
2. Hacé clic en **"Create API Key"**
3. Elegí un proyecto (o creá uno nuevo) y copiá la key
4. En n8n → Credentials → Add Credential → buscá **"Google Gemini"** (o **"Google PaLM"**)
5. Pegá tu API key y guardá

### 🟡 Paso 4: Armar el RAG en n8n

1. Importá [`ejemplo-rag-basico.json`](./ejemplo-rag-basico.json) en n8n
2. Hacé clic en cada nodo con ⚠️ y asigná las credenciales:
   - **Nodo LLM**: Seleccioná tu credencial de Ollama (o OpenAI/Groq)
   - **Nodo Embeddings**: Seleccioná Ollama (modelo: `nomic-embed-text`) o Google Gemini
   - **Nodo Vector Store**: Usá "In Memory" — no necesita credenciales
3. Hacé clic en **"Test Workflow"**
4. Mandá una pregunta:

```bash
curl -X POST http://localhost:5678/webhook-test/rag-pregunta -H "Content-Type: application/json" -d "{\"pregunta\": \"Que protocolo sigo para dolor de pecho?\"}"
```

### 🔧 Errores Posibles

| Problema | Solución |
|:---|:---|
| "Model not found: nomic-embed-text" | Corré `ollama pull nomic-embed-text` en la terminal |
| "ECONNREFUSED 11434" | Ollama no está corriendo. Buscá el ícono en la barra o corré `ollama serve` |
| El vector store queda vacío | Necesitás cargar documentos primero. Usá el workflow 03.01 para ingestar docs |
| "Embedding dimension mismatch" | Estás mezclando modelos de embeddings distintos. Usá siempre el mismo modelo para cargar y buscar |
| Respuesta genérica sin contexto | El retriever no encontró chunks relevantes. Revisá que los documentos estén cargados en el vector store |

> 💡 **Tip para probar rápido**: El vector store "In Memory" se borra cada vez que reiniciás n8n. Para datos persistentes, usá Pinecone o Qdrant (ver workflows 03.02 y 03.03).

---

## �📄 Materiales de la Clase
- **Presentación**: `IA Automation-03.pdf`
- **Temas Teóricos**:
  - `Semana 2 - tema 5.pdf`
  - `Semana 2 - tema 6.pdf`
