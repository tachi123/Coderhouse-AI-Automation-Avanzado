# 🤖 Curso de IA Automation Avanzado — Coderhouse

Repositorio oficial del curso **IA Automation Avanzado** de Coderhouse. Acá vas a encontrar la teoría, los workflows de n8n y los proyectos prácticos de cada clase, desde los fundamentos de automatización con IA hasta pipelines de evaluación en producción.

## 🎯 ¿De qué trata el curso?

El curso recorre el camino completo para construir **automatizaciones inteligentes con IA**, usando **n8n** como motor de orquestación y modelos de lenguaje (LLMs) como cerebro. Arrancamos desde cero y terminamos con pipelines de evaluación que miden la calidad de las respuestas de la IA con métricas reales.

### Tecnologías y herramientas que se usan

| Categoría | Herramientas |
|:---|:---|
| **Orquestación** | n8n (self-hosted con Docker) |
| **LLMs** | OpenAI, Groq (Llama 3), Ollama (local), Google Gemini |
| **Embeddings** | Nomic Embed Text, Google Gemini Embeddings |
| **Vector Stores** | In-memory, Pinecone, Qdrant |
| **Parsing** | LlamaCloud, LlamaParse |
| **Infraestructura** | Docker, Docker Compose |
| **Integraciones** | Telegram, Google Sheets, Webhooks, MySQL |
| **Frontend** | HTML/JS para demos interactivas |

---

## 🚀 Índice de Clases

| # | Clase | Tema Principal | Conceptos Clave | Workflows |
|:---:|:---|:---|:---|:---|
| 01 | **[Clase 01](./Clase01)** | Introducción y Fundamentos | n8n, LLMs, APIs, arquitectura de automatización | [ejemplo-hola-mundo.json](./Clase01/ejemplo-hola-mundo.json) |
| 02 | **[Clase 02](./Clase02)** | Extracción de Datos y Triage | Prompts, structured output, clasificación, webhooks | [Clase-02.json](./Clase02/Clase-02.json), [ejemplo-extraccion-datos.json](./Clase02/ejemplo-extraccion-datos.json) |
| 03 | **[Clase 03](./Clase03)** | RAG y Bases Vectoriales | Embeddings, chunking, vector stores, retrieval | [03.01](./Clase03/AI%20AUTOMATION-03.01.json), [03.02](./Clase03/AI%20AUTOMATION-03.02.json), [03.03](./Clase03/AI%20AUTOMATION-03.03.json), [ejemplo-rag-basico.json](./Clase03/ejemplo-rag-basico.json) |
| 04 | **[Clase 04](./Clase04)** | Parsing Avanzado y LlamaCloud | Document intelligence, OCR, parsing de PDFs | [04.01](./Clase04/AI%20AUTOMATION-04.01-extractfile.json), [04.02](./Clase04/AI%20AUTOMATION-04.02-llamacloud.json), [04.03](./Clase04/AI%20AUTOMATION-04.03-llamaCloudRequests.json), [ejemplo-parsing-documento.json](./Clase04/ejemplo-parsing-documento.json) |
| 05 | **[Clase 05](./Clase05)** | IA Multimodal y Visión | Modelos de visión, análisis de imágenes, base64 | [05.json](./Clase05/AI%20AUTOMATION-05.json), [ejemplo-vision-multimodal.json](./Clase05/ejemplo-vision-multimodal.json) |
| 06 | **[Clase 06](./Clase06)** | Agentes de IA | Tool-use, ReAct, orquestación de agentes | [06.json](./Clase06/AI%20AUTOMATION-06.json), [06.01.json](./Clase06/AI%20AUTOMATION-06.01.json), [ejemplo-agente-herramientas.json](./Clase06/ejemplo-agente-herramientas.json) |
| 07 | **[Clase 07](./Clase07)** | Human-in-the-Loop (HITL) | Supervisión humana, aprobación, chat en tiempo real | [HITL Triage](./Clase07/hitl-n8n-demo/workflow_hitl_triage_n8n.json), [ejemplo-aprobacion-humana.json](./Clase07/ejemplo-aprobacion-humana.json) |
| 08 | **[Clase 08](./Clase08)** | Escalabilidad y Buenas Prácticas | Error handling, costos, seguridad, retry patterns | [ejemplo-error-handling.json](./Clase08/ejemplo-error-handling.json) |
| 09 | **[Clase 09](./Clase09)** | Docker e Infraestructura | Docker Compose, n8n self-hosted, variables de entorno | [docker-compose.yml](./Clase09/docker-compose.yml), [ejemplo-health-check.json](./Clase09/ejemplo-health-check.json) |
| 10 | **[Clase 10](./Clase10)** | Evaluación de IA y Evals | Embeddings, similitud de coseno, batch testing, A/B prompts | [Pipeline Evaluación](./Clase10/pipelineevaluacion.json), [Lab Evals](./Clase10/laboratorio-evals.json) |

---

## ⚡ Instalación Rápida (Tener Todo Listo en 5 Minutos)

Seguí estos 3 pasos y vas a tener el entorno completo para arrancar el curso.

### 🟢 Paso 1: Instalar Docker Desktop

Docker es lo que va a correr n8n y todas las herramientas del curso en tu máquina.

1. Entrá a [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Hacé clic en **"Download for Windows"** (o Mac/Linux según tu SO)
3. Instalá el `.exe` que se descarga — siguiente, siguiente, finalizar
4. Abrí **Docker Desktop** desde el menú de inicio
5. Esperá a que el ícono de la ballena en la barra quede **verde** (puede tardar 1-2 minutos la primera vez)

**¿Cómo sé que funciona?** Abrí una terminal (PowerShell o CMD) y escribí:
```bash
docker --version
```
Si te muestra algo como `Docker version 24.x.x`, estás listo.

> ⚠️ **Error común en Windows**: Si dice "WSL 2 is not installed", seguí las instrucciones que aparecen en Docker Desktop para instalar WSL 2 y reiniciá la máquina.

### 🔵 Paso 2: Levantar n8n con Docker Compose

1. Abrí una terminal y navegá a la carpeta de la Clase 09:
   ```bash
   cd Clase09
   ```
2. Levantá n8n con un solo comando:
   ```bash
   docker-compose up -d
   ```
3. Esperá unos segundos y abrí el navegador en: **http://localhost:5678**
4. La primera vez te va a pedir crear un usuario y contraseña — elegí lo que quieras, es local

**¿Cómo sé que funciona?** En el navegador tenés que ver el editor visual de n8n con el lienzo para crear workflows.

> ⚠️ **Error "port already in use"**: Si el puerto 5678 está ocupado, cambiá en el `docker-compose.yml` la línea `"5678:5678"` por `"5679:5678"` y entrá a `http://localhost:5679`.

### 🟣 Paso 3: Instalar Ollama (Modelos de IA Locales)

Ollama te permite correr modelos de IA en tu propia máquina, sin enviar datos a la nube.

1. Entrá a [https://ollama.com/download](https://ollama.com/download)
2. Descargá e instalá la versión para tu sistema operativo
3. Abrí una terminal y descargá los modelos que vamos a usar:
   ```bash
   ollama pull llama3
   ollama pull nomic-embed-text
   ```
4. Verificá que se descargaron bien:
   ```bash
   ollama list
   ```
   Deberías ver `llama3` y `nomic-embed-text` en la lista.

**¿Cómo sé que funciona?** Probá escribir en la terminal:
```bash
ollama run llama3 "Hola, decime algo en español"
```
Si te responde en español, Ollama está funcionando perfecto.

> ⚠️ **Error "connection refused" desde n8n**: Dentro de Docker, Ollama no está en `localhost`. En n8n, cuando configures las credenciales de Ollama, usá la URL: `http://host.docker.internal:11434`

### 🔑 API Keys que vas a necesitar (según la clase)

| Servicio | Para qué | Dónde conseguirla | Clase |
|:---|:---|:---|:---|
| **OpenAI** | GPT-4, GPT-3.5 | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) | 01, 02 |
| **Groq** | Llama 3 rápido y gratis | [console.groq.com](https://console.groq.com) → API Keys | 02, 10 |
| **Google Gemini** | Gemini Pro, embeddings | [aistudio.google.com/apikey](https://aistudio.google.com/apikey) | 03, 05 |
| **LlamaCloud** | Parsing de PDFs | [cloud.llamaindex.ai](https://cloud.llamaindex.ai) → API Keys | 04 |
| **Telegram** | Bot para chatear con la IA | @BotFather en Telegram | 07, 10 |

---

## 📋 Requisitos Previos (Resumen)

- **Docker Desktop** instalado y corriendo (ver Paso 1 arriba ☝️)
- **n8n** levantado con Docker Compose (ver Paso 2)
- **Ollama** con modelos (ver Paso 3): `llama3` y `nomic-embed-text`
- Cuentas/API Keys según la clase (ver tabla arriba ☝️)

---

## 🛠️ Cómo Importar los Workflows (.json) en n8n

1. Abrí tu instancia de n8n en el navegador → **http://localhost:5678**
2. Hacé clic en **"Add workflow"** (arriba a la derecha) para crear uno nuevo
3. Para importar un JSON, tenés 3 opciones:
   - **Opción A**: Arrastrá el archivo `.json` directo al lienzo de n8n
   - **Opción B**: Click en los tres puntitos (⋮) arriba a la derecha → **"Import from File"** → seleccioná el `.json`
   - **Opción C**: Abrí el `.json` con el Bloc de Notas, copiá todo el contenido, y en n8n pegalo con **Ctrl+V**
4. Los nodos van a aparecer en el lienzo. **Importante**: hacé clic en los nodos que tengan un ⚠️ y configurá las credenciales (API keys)

### ¿Cómo configuro las credenciales en n8n?
1. Hacé clic en el nodo que tiene el ⚠️ (por ejemplo, un nodo de OpenAI)
2. Donde dice **"Credential"**, hacé clic en **"Create New Credential"**
3. Pegá tu API Key en el campo correspondiente
4. Hacé clic en **"Save"**
5. Repetí para cada nodo que lo necesite

> 💡 **Tip**: Una vez que creaste una credencial (ej: OpenAI), la podés reutilizar en todos los workflows — no hace falta crearla de nuevo cada vez.

---

## 📂 Estructura del Repositorio

```
📁 Coderhouse-AI-Automation-Avanzado/
├── 📄 README.md              ← Estás acá
├── 📁 Clase01/                ← Introducción y Fundamentos
├── 📁 Clase02/                ← Extracción de Datos y Triage
├── 📁 Clase03/                ← RAG y Bases Vectoriales
├── 📁 Clase04/                ← Parsing Avanzado y LlamaCloud
├── 📁 Clase05/                ← IA Multimodal y Visión
├── 📁 Clase06/                ← Agentes de IA
├── 📁 Clase07/                ← Human-in-the-Loop
├── 📁 Clase08/                ← Escalabilidad y Buenas Prácticas
├── 📁 Clase09/                ← Docker e Infraestructura
└── 📁 Clase10/                ← Evaluación de IA y Evals
```

Cada carpeta contiene:
- 📄 **PDFs** — Material teórico de la clase
- ⚙️ **JSONs** — Workflows de n8n listos para importar
- 📝 **README** — Explicación de conceptos y guía de la clase
- 🌐 **HTML** — Demos interactivas (cuando aplica)
