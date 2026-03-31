# Clase 10: Evaluación de IA — Pipeline de Evaluación y Laboratorio de Evals

Esta clase cierra el curso con el tema más avanzado: cómo **medir la calidad** de las respuestas de una IA usando métricas reales, embeddings y similitud de coseno. Además, se incluye un tutorial para conectar un bot de Telegram con IA usando OpenClaw.

---

## 📚 Conceptos Clave

### ¿Por qué evaluar a la IA?
A diferencia del software tradicional (donde el test es binario: pasa o no pasa), la IA es **probabilística** — cada vez puede responder distinto. La pregunta no es "¿funciona?" sino **"¿qué tan bien funciona?"**.

### Embeddings y Similitud de Coseno
La forma de medir si una respuesta de IA es "correcta" es compararla numéricamente contra un **protocolo de referencia** (la respuesta ideal).

| Paso | Qué hace | Ejemplo |
|:---|:---|:---|
| 1. **Embedding del protocolo** | Convierte el texto de referencia en un vector de 768 números | "Derivar a urgencias" → [0.8, 0.1, ...] |
| 2. **Embedding de la respuesta** | Convierte lo que dijo la IA en otro vector | "Tomese un té" → [0.2, 0.7, ...] |
| 3. **Similitud de Coseno** | Calcula qué tan "alineados" están los vectores | 0.3 (30%) → MUY diferente |

> Pensalo como una brújula: el protocolo es el Norte. Si la IA apunta cerca del Norte (>75%), está bien. Si se va al Sur (<50%), está mal.

### IA como Jurado (LLM-as-a-Judge)
Usar un **segundo LLM** para evaluar la respuesta del primero:
- **Residente** (LLM 1): Genera la respuesta al paciente
- **Jefe de Guardia** (LLM 2): Lee la respuesta y dice APROBADO o RECHAZADO

### Doble Check (IA + Matemática)
El pipeline usa dos capas de seguridad simultáneas:

```
Respuesta del Residente
        ↓
   ┌────┴────┐
   ↓         ↓
[Jefe de    [Similitud de
 Guardia]    Coseno ≥ 75%]
   ↓         ↓
   └────┬────┘
        ↓
 ¿AMBOS aprueban?
   ↓ SÍ      ↓ NO
[Enviar]   [Bloquear + Alertar]
```

### Evaluación en Batch (Laboratorio de Evals)
Probar con un solo caso no alcanza. El laboratorio ejecuta **20 casos de prueba** en 4 categorías:

| Categoría | Casos | Qué testea |
|:---|:---|:---|
| **Fáciles** (1-5) | Dolor de cabeza, resfriado | La IA debería responder bien siempre |
| **Urgentes** (6-10) | Dolor de pecho, desmayo | La IA TIENE que derivar a urgencias |
| **Trampas** (11-15) | "Un poco de dolor de pecho, seguro no es nada" | Detectar si la IA se confunde cuando el paciente minimiza |
| **Ambiguos** (16-20) | Estrés, insomnio, mareos | Zona gris — mide el criterio de la IA |

### Scorecard (Tabla de Resultados)
Al final del batch, se genera un reporte con:
- Nota general sobre 10
- Desglose por categoría
- Lista de casos que falló
- Promedio de similitud de coseno

### A/B Testing de Prompts
Si el prompt A saca 7/10 y le agregás una línea ("NUNCA ignores síntomas de pecho"), el prompt B saca 9/10 → **optimizaste con datos, no con intuición**.

### Monitoreo en Producción
Guardar las notas de cada evaluación en una base de datos. Si la nota promedio cae de 9.5 a 7.0 entre lunes y viernes, **algo cambió** y lo detectaste automáticamente con métricas.

---

## 🧪 Workflows

### Pipeline Principal: Evaluación con Doble Check
**Archivo**: [`pipelineevaluacion.json`](./pipelineevaluacion.json)

Flujo completo con:
1. Webhook que recibe la consulta
2. AI Agent Residente (genera respuesta)
3. AI Agent Jefe de Guardia (audita la respuesta)
4. Nodo de Similitud de Coseno (compara con protocolo)
5. Doble check (IF: aprobado + similitud ≥ 75%)
6. Log de evaluación

```bash
curl -X POST http://localhost:5678/webhook-test/triage-eval \
  -H "Content-Type: application/json" \
  -d '{"message": "Me duele mucho el pecho y me falta el aire"}'
```

### Laboratorio de Evals (Batch Testing)
**Archivo**: [`laboratorio-evals.json`](./laboratorio-evals.json)

Ejecuta 20 casos de prueba automáticamente y genera un scorecard.

```bash
curl -X POST http://localhost:5678/webhook-test/lab-evals \
  -H "Content-Type: application/json" \
  -d '{}'
```

---

## �️ Tutorial: Montar el Sistema de Evaluación Completo (Paso a Paso)

### 🟢 Paso 1: Preparar los Modelos de Ollama

El pipeline de evaluación necesita dos modelos: uno para generar respuestas y otro para calcular embeddings.

1. Verificá que Ollama esté corriendo (buscá el ícono en la barra de notificaciones)
2. Abrí una terminal y descargá los modelos:
   ```bash
   ollama pull llama3
   ollama pull nomic-embed-text
   ```
3. Verificá que estén:
   ```bash
   ollama list
   ```
4. Probá que los embeddings funcionen (esto es lo que usa el nodo de similitud de coseno):
   ```bash
   curl http://localhost:11434/api/embeddings -d "{\"model\":\"nomic-embed-text\",\"prompt\":\"hola mundo\"}"
   ```
   Deberías recibir un JSON con un array de 768 números — esos son los embeddings.

> ⚠️ Si el curl da error, Ollama no está corriendo. Corré `ollama serve` en una terminal.

### 🔵 Paso 2: Configurar Ollama en n8n

1. Abrí n8n → **http://localhost:5678**
2. Menú izquierdo → **"Credentials"** (🔑) → **"Add Credential"** → buscá **"Ollama"**
3. En **"Base URL"**: `http://host.docker.internal:11434`
4. Hacé clic en **"Save"**

> 💡 Recordá: `host.docker.internal` es la forma de que n8n (que corre dentro de Docker) hable con Ollama (que corre fuera de Docker).

### 🟣 Paso 3: Importar y Configurar el Pipeline de Evaluación

1. Abrí n8n y creá un workflow nuevo
2. Abrí [`pipelineevaluacion.json`](./pipelineevaluacion.json) con el Bloc de Notas → copiá todo → pegá en n8n con Ctrl+V
3. Configurá las credenciales en los nodos marcados con ⚠️:
   - **AI Agent Residente**: Credencial de Ollama → modelo: `llama3`
   - **AI Agent Jefe de Guardia**: Misma credencial de Ollama → modelo: `llama3`
   - **Nodo de Embeddings**: Credencial de Ollama → modelo: `nomic-embed-text`
4. Hacé clic en **"Test Workflow"**
5. Probá con un caso fácil:
   ```bash
   curl -X POST http://localhost:5678/webhook-test/triage-eval -H "Content-Type: application/json" -d "{\"message\": \"Me duele un poco la cabeza desde ayer\"}"
   ```
6. Mirá los resultados en cada nodo — especialmente el nodo **📐 Similitud de Coseno** que te muestra el porcentaje

### 🟡 Paso 4: Correr el Laboratorio de Evals (20 Casos)

1. Importá [`laboratorio-evals.json`](./laboratorio-evals.json) en otro workflow
2. Configurá las mismas credenciales de Ollama
3. Ejecutá:
   ```bash
   curl -X POST http://localhost:5678/webhook-test/lab-evals -H "Content-Type: application/json" -d "{}"
   ```
4. **Paciencia**: Son 20 casos × 2 agentes × embeddings = puede tardar 2-5 minutos con Ollama local
5. Al terminar, el nodo **Scorecard** te muestra la nota sobre 10

### 🔴 Paso 5: Crear un Bot de Telegram con OpenClaw (Bonus)

Si querés conectar tu IA a Telegram para chatear desde el celular, usá **OpenClaw**. Toda la guía está en [`tutorial_openclaw.md`](./tutorial_openclaw.md), pero el resumen rápido:

1. **Conseguir key de Groq**: Entrá a [console.groq.com](https://console.groq.com) → API Keys → Create → copiá la `gsk_...`
2. **Crear bot en Telegram**: Abrí Telegram → buscá `@BotFather` → escribí `/newbot` → seguí los pasos → copiá el token
3. **Clonar e instalar OpenClaw**:
   ```bash
   git clone https://github.com/openclaw/openclaw.git
   cd openclaw
   cp .env.example .env
   ```
4. **Editar el .env** con el Bloc de Notas o VS Code:
   ```
   OPENAI_API_KEY=TU_KEY_DE_GROQ_ACA
   OPENAI_BASE_URL=https://api.groq.com/openai/v1
   MODEL=llama3-70b-8192
   TELEGRAM_TOKEN=TU_TOKEN_DE_BOTFATHER_ACA
   ```
5. **Levantar con Docker**:
   ```bash
   docker-compose up -d
   ```
6. ¡Abrí Telegram, buscá tu bot y escribile!

> ⚠️ **Error "image not found"**: Cambiá `image: openclaw` por `build: .` en el `docker-compose.yml` de OpenClaw (ver detalles en [`tutorial_openclaw.md`](./tutorial_openclaw.md))

---

## 📝 Resumen de Conceptos vs Workflows

| Concepto | Dónde se ve | Workflow |
|:---|:---|:---|
| **IA como Jurado** | El Jefe de Guardia audita al Residente | `pipelineevaluacion.json` |
| **Flagging** | El sistema marca y bloquea respuestas peligrosas | `pipelineevaluacion.json` |
| **Embeddings / Similitud de Coseno** | Comparación numérica respuesta vs protocolo | `pipelineevaluacion.json` |
| **Reglas Heurísticas** | Prompt del Juez con reglas explícitas | Ambos |
| **Métricas Semánticas** | Vectores + distancia angular | Ambos |
| **Validación Batch** | 20 casos de prueba con scorecard | `laboratorio-evals.json` |
| **A/B Testing de Prompts** | Cambiar prompt y comparar notas | `laboratorio-evals.json` |
| **Monitoreo** | Log de evaluación con timestamp | `pipelineevaluacion.json` |

---

## 🔧 Troubleshooting

| Problema | Solución |
|:---|:---|
| `nomic-embed-text` no encontrado | `ollama pull nomic-embed-text` |
| Error de conexión en embeddings | Verificar Ollama en `localhost:11434` |
| Los agentes no responden | `ollama list` para verificar modelos |
| Ollama pide credenciales en n8n | Crear credencial con URL `http://host.docker.internal:11434` |
| Similitud siempre da 0 | Probar: `curl http://localhost:11434/api/embeddings -d '{"model":"nomic-embed-text","prompt":"hola"}'` |
| El lab de evals tarda mucho | Es normal con Ollama local (2-5 min). Con Groq es más rápido |
| OpenClaw no se conecta a Telegram | Verificá el token del bot y que Docker esté corriendo |

---

## 📄 Materiales Adicionales
- **Tutorial OpenClaw + Telegram**: [`tutorial_openclaw.md`](./tutorial_openclaw.md) — Guía paso a paso para conectar un bot de Telegram con Groq
