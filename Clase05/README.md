# Clase 05: IA Multimodal y Análisis de Imágenes

Esta clase explora las capacidades de los modelos de visión para analizar y extraer datos de imágenes (radiografías, recetas médicas, etc.).

---

## 📚 Conceptos Clave

### ¿Qué es la IA Multimodal?
Un modelo **multimodal** puede procesar más de un tipo de dato. Los LLMs clásicos solo entienden texto. Los modelos multimodales (GPT-4o, Gemini Pro Vision, Llama 3.2 Vision) entienden **texto + imágenes** simultáneamente.

### Cómo funciona la Visión en un LLM / Vision Models
| Paso | Qué pasa | Ejemplo |
|:---|:---|:---|
| 1. **Input** | Se envía la imagen codificada en Base64 o como URL | Una foto de una radiografía |
| 2. **Procesamiento** | El modelo "ve" la imagen y la interpreta | Detecta una fractura en la costilla |
| 3. **Output** | Genera texto describiendo lo que encontró | "Se observa fractura en la 5ta costilla..." |

### Base64: El Idioma de las Imágenes en las APIs
Las APIs no pueden recibir un archivo .jpg directamente. Hay que convertirlo a **Base64** — una representación de la imagen como texto (un string largo de letras y números). Así el modelo puede "leer" la imagen dentro del JSON.

```
Imagen (.jpg) → Base64 (string) → API del LLM → Respuesta en texto
```

### Casos de Uso en el Mundo Real
- **Salud**: Análisis de radiografías, informes de laboratorio, recetas médicas
- **Retail**: Clasificación de productos por foto
- **Documentos**: Leer facturas, remitos, documentos escaneados
- **Seguridad**: Detección de anomalías en imágenes de CCTV

### Limitaciones Importantes
- Los modelos **NO reemplazan** a un profesional médico — son asistentes
- La calidad de la imagen afecta directamente la calidad del análisis
- Algunos modelos tienen límite de tamaño/resolución
- Pueden alucinar detalles que no existen en la imagen

---

## 🧪 Workflows

### Práctica principal: Visión por Computadora con LLMs
**Archivo**: [`AI AUTOMATION-05.json`](./AI%20AUTOMATION-05.json)

### Ejemplo adicional: Análisis de Imagen Multimodal
**Archivo**: [`ejemplo-vision-multimodal.json`](./ejemplo-vision-multimodal.json)

Workflow que recibe una URL de imagen y una pregunta, y usa un modelo de visión para analizarla.

```bash
curl -X POST http://localhost:5678/webhook-test/analizar-imagen \
  -H "Content-Type: application/json" \
  -d '{"imagen_url": "https://ejemplo.com/radiografia.jpg", "pregunta": "¿Qué se observa en esta imagen médica?"}'
```

### Demo: Interfaz de Análisis
- [`index.html`](./index.html) — Interfaz web para cargar imágenes y recibir análisis de la IA
- **Ejemplos de Imágenes**: En `images-examples/` hay muestras para testear

---

## �️ Tutorial: Análisis de Imágenes con IA (Paso a Paso)

### 🟢 Paso 1: Elegir tu Modelo de Visión

No todos los modelos saben "ver" imágenes. Necesitás uno que soporte **multimodal**:

| Modelo | Proveedor | Gratis | Calidad | Cómo configurarlo en n8n |
|:---|:---|:---|:---|:---|
| **GPT-4o** | OpenAI | No (de pago) | ⭐⭐⭐⭐⭐ | Credencial OpenAI → modelo: `gpt-4o` |
| **Gemini Pro Vision** | Google | Sí (free tier) | ⭐⭐⭐⭐ | Credencial Google Gemini → modelo: `gemini-pro-vision` |
| **Llama 3.2 Vision** | Ollama (local) | Sí (local) | ⭐⭐⭐ | Credencial Ollama → modelo: `llama3.2-vision` |
| **Llava** | Ollama (local) | Sí (local) | ⭐⭐⭐ | Credencial Ollama → modelo: `llava` |

### 🔵 Paso 2: Instalar un Modelo de Visión Local (Ollama)

Si querés usar visión sin pagar y sin enviar datos a la nube:

1. Abrí una terminal y corré:
   ```bash
   ollama pull llava
   ```
   (Llava pesa ~4.7 GB — es el modelo de visión más popular para Ollama)

2. Probá que funcione:
   ```bash
   ollama run llava "describe esta imagen" --images ./images-examples/radiografia.jpg
   ```

> 💡 **Alternativa más potente**: Si tu PC tiene buena GPU, probá `ollama pull llama3.2-vision` (11B parámetros).

### 🟣 Paso 3: Configurar Google Gemini Vision (Alternativa Cloud Gratis)

1. Entrá a [https://aistudio.google.com/apikey](https://aistudio.google.com/apikey)
2. Hacé clic en **"Create API Key"** → copiá la key
3. En n8n → Credentials → Add Credential → **"Google Gemini"**
4. Pegá la API key y guardá
5. En el nodo del LLM, seleccioná el modelo: `gemini-pro-vision`

### 🟡 Paso 4: Probar la Interfaz Web (index.html)

La clase incluye una interfaz HTML que te permite cargar una imagen y analizarla con la IA, sin usar curl.

1. Importá [`AI AUTOMATION-05.json`](./AI%20AUTOMATION-05.json) en n8n y activá el workflow (toggle arriba a la derecha)
2. La interfaz está en [`index.html`](./index.html) — abrila en el navegador:
   - **Opción simple**: Hacé doble clic en el archivo `index.html` desde el explorador de archivos
   - **Opción con server**: Si no anda, abrí una terminal en la carpeta `Clase05` y corré:
     ```bash
     python -m http.server 8080
     ```
     Y entrá a `http://localhost:8080` en el navegador
3. Cargá una imagen (podés usar las de `images-examples/`)
4. Escribí una pregunta (ej: "¿Qué se ve en esta imagen?")
5. Hacé clic en enviar — la respuesta de la IA debería aparecer abajo

### 🔧 Errores Posibles

| Problema | Solución |
|:---|:---|
| "This model does not support images" | Estás usando un modelo que no es multimodal. Cambiá a `gpt-4o`, `gemini-pro-vision` o `llava` |
| La imagen no se envía | Verificá que el workflow esté **activo** (toggle verde arriba a la derecha), no solo en modo test |
| "Payload too large" | La imagen es muy grande. Redimensionala a menos de 2MB antes de enviarla |
| El index.html no muestra la respuesta | Revisá que la URL del webhook en el HTML coincida con la de tu n8n (`localhost:5678/webhook/...`) |
| "CORS error" en el navegador | Abrí el `index.html` con un server local (python -m http.server) en vez de doble clic |

> 💡 **Tip**: En `images-examples/` hay un archivo `radiografia - copia.txt` que muestra cómo se ve una imagen convertida a Base64. Si tenés curiosidad, abrilo y vas a ver un texto largo de caracteres — eso es lo que recibe la API.

---

## �📄 Materiales de la Clase
- **Presentación**: `IA Automation-05.pdf`
- **Temas Teóricos**:
  - `Semana 3 - tema 8.pdf`
