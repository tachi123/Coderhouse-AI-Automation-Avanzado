# Clase 01: Introducción a la Automatización con IA

Esta clase cubre los fundamentos de la automatización utilizando Inteligencia Artificial, estableciendo las bases para el resto del curso.

---

## 📚 Conceptos Clave

### ¿Qué es la Automatización con IA?
La automatización con IA combina flujos de trabajo automatizados (workflows) con modelos de lenguaje (LLMs) para procesar, analizar y generar información sin intervención humana constante.

### Componentes fundamentales

| Concepto | Descripción |
|:---|:---|
| **LLM (Large Language Model)** | Modelos de IA entrenados con grandes cantidades de texto. Ejemplos: GPT-4, Llama 3, Gemini. Reciben texto y generan texto. |
| **API (Application Programming Interface)** | El "puente" que conecta tu aplicación con el modelo de IA. Le mandás un request, te devuelve una respuesta. |
| **n8n** | Plataforma open-source de automatización visual. Permite crear workflows arrastrando nodos y conectándolos. Se hostea con Docker. |
| **Webhook** | Un endpoint HTTP que recibe datos. Es el "oído" de tu workflow: cuando alguien le manda un mensaje, se dispara la automatización. |
| **Nodo** | Cada bloque dentro de un workflow de n8n. Puede ser un trigger (inicio), una acción (llamar API), o una transformación (formatear datos). |
| **Workflow** | La secuencia completa de nodos conectados. De principio a fin: entra un dato, se procesa, sale un resultado. |

### Arquitectura básica de una automatización con IA

```
[Trigger/Webhook] → [Procesar Input] → [Llamar al LLM] → [Formatear Respuesta] → [Output]
```

### ¿Por qué n8n y no otro?
- **Self-hosted**: Tus datos no salen de tu servidor
- **Visual**: No necesitás ser programador para crear flujos
- **Extensible**: +400 integraciones nativas + nodos de código custom
- **Gratis**: Open source, sin límites de ejecuciones

---

## 🧪 Workflow de Ejemplo: Hola Mundo con IA

**Archivo**: [`ejemplo-hola-mundo.json`](./ejemplo-hola-mundo.json)

Un workflow introductorio que demuestra la estructura básica:
1. **Webhook** — Recibe una pregunta vía HTTP POST
2. **LLM (OpenAI/Groq)** — Procesa la pregunta y genera una respuesta
3. **Respond to Webhook** — Devuelve la respuesta al usuario

### Cómo probarlo
```bash
curl -X POST http://localhost:5678/webhook-test/hola-mundo \
  -H "Content-Type: application/json" \
  -d '{"pregunta": "¿Qué es la inteligencia artificial?"}'
```

---

## �️ Tutorial: Tu Primer Workflow con IA (Paso a Paso)

### 🟢 Paso 1: Conseguir tu API Key de OpenAI

1. Entrá a [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Si no tenés cuenta, creá una (podés usar tu Google). Te van a pedir verificar con teléfono.
3. Una vez adentro, hacé clic en **"+ Create new secret key"**
4. Poné un nombre (ej: `Clase-Coderhouse`) y hacé clic en **"Create secret key"**
5. **COPIÁ LA KEY** que empieza con `sk-...` — ¡Guardala bien! Solo la ves una vez.

> 💡 **Alternativa gratuita**: Si no querés gastar en OpenAI, podés usar **Groq** (gratis):
> 1. Entrá a [https://console.groq.com](https://console.groq.com)
> 2. Logueate con Google
> 3. Menú izquierdo → **"API Keys"** → **"Create API Key"**
> 4. Copiá la key que empieza con `gsk_...`

### 🔵 Paso 2: Configurar la Credencial en n8n

1. Abrí n8n en el navegador → **http://localhost:5678**
2. En el menú de la izquierda, hacé clic en **"Credentials"** (el ícono de la llave 🔑)
3. Hacé clic en **"Add Credential"** (arriba a la derecha)
4. Buscá **"OpenAI"** (o **"Groq"** si usás la alternativa gratis)
5. Pegá tu API Key en el campo **"API Key"**
6. Hacé clic en **"Save"**

> ⚠️ **Si usás Groq en lugar de OpenAI**: en n8n buscá la credencial **"OpenAI"** igual, pero en el campo **"Base URL"** poné: `https://api.groq.com/openai/v1` — Groq es compatible con el formato de OpenAI.

### 🟣 Paso 3: Importar y Ejecutar tu Primer Workflow

1. En n8n, hacé clic en **"Add workflow"** para crear uno nuevo
2. Abrí el archivo [`ejemplo-hola-mundo.json`](./ejemplo-hola-mundo.json) con el Bloc de Notas
3. Copiá **todo** el contenido (Ctrl+A → Ctrl+C)
4. Volvé a n8n y pegalo en el lienzo con **Ctrl+V** — van a aparecer los nodos
5. Hacé clic en el nodo del LLM (tiene un ⚠️) → seleccioná la credencial que creaste en el Paso 2
6. Hacé clic en **"Test Workflow"** (el botón de play arriba)
7. Abrí otra terminal y mandá este comando:

```bash
curl -X POST http://localhost:5678/webhook-test/hola-mundo -H "Content-Type: application/json" -d "{\"pregunta\": \"Hola, que es la IA?\"}"
```

8. ¡Deberías ver la respuesta de la IA en la terminal! 🎉

### 🔧 Errores Posibles

| Problema | Solución |
|:---|:---|
| "Unauthorized" o "Invalid API key" | Revisá que la key esté bien pegada en Credentials, sin espacios |
| "Connection refused" en el curl | Verificá que n8n esté corriendo (`docker ps`) y que el webhook diga "Listening" |
| "Workflow not active" | No te preocupes — en modo test funciona igual. El "Test Workflow" activa el webhook temporalmente |
| No devuelve nada | Fijate que el nodo "Respond to Webhook" esté conectado al final del flujo |

---

## �📄 Materiales de la Clase
- **Presentación**: `IA Automation-01.pdf`
- **Temas Teóricos**:
  - `Semana 1 - tema 1.pdf`
  - `Semana 1 - tema 2.pdf`
