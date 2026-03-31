# Clase 04: Optimización de RAG y LlamaCloud

Se exploran herramientas avanzadas para la limpieza y extracción de datos complejos, mejorando la calidad del contexto entregado a la IA.

---

## 📚 Conceptos Clave

### El Problema del "Garbage In, Garbage Out"
Si le das al LLM un PDF mal parseado (con tablas rotas, texto superpuesto, headers repetidos), la respuesta va a ser mala aunque el modelo sea bueno. La calidad del parsing determina la calidad del RAG.

### Document Intelligence / Parsing Avanzado
| Técnica | Qué hace | Cuándo usarla |
|:---|:---|:---|
| **Text Extraction** | Saca texto plano de un archivo | PDFs simples, sin tablas |
| **OCR (Reconocimiento Óptico)** | Lee texto de imágenes/escaneados | PDFs escaneados, fotos de documentos |
| **Layout Analysis** | Entiende la estructura (tablas, columnas) | PDFs con tablas, formularios |
| **LlamaParse** | Parsing con IA que entiende contexto | Documentos complejos (informes, papers) |

### LlamaCloud
Plataforma de LlamaIndex que ofrece:
- **LlamaParse**: API de parsing de documentos con IA que entiende estructura
- **Managed Indexes**: Índices vectoriales gestionados en la nube
- **Pipeline integration**: Se conecta directo con n8n vía HTTP Request

### Flujo de Parsing Avanzado
```
[Archivo PDF/DOC] → [LlamaParse API] → [Texto Estructurado/Markdown] → [Chunking] → [Vector Store]
```

### Comparación: Parsing Básico vs Avanzado
| Aspecto | Básico (Extract Text) | Avanzado (LlamaParse) |
|:---|:---|:---|
| Tablas | Se rompen | Se mantienen en Markdown |
| Imágenes de texto | No las lee | Las interpreta con OCR |
| Estructura | Todo en un bloque | Respeta headers, listas, secciones |
| Costo | Gratis | API de pago (con free tier) |

---

## 🧪 Workflows

### Práctica principal: Extracción Avanzada
- [`AI AUTOMATION-04.01-extractfile.json`](./AI%20AUTOMATION-04.01-extractfile.json) — Extracción de campos desde archivos
- [`AI AUTOMATION-04.02-llamacloud.json`](./AI%20AUTOMATION-04.02-llamacloud.json) — Integración con LlamaCloud para parsing avanzado
- [`AI AUTOMATION-04.03-llamaCloudRequests.json`](./AI%20AUTOMATION-04.03-llamaCloudRequests.json) — Peticiones a la API de LlamaCloud
- [`AI AUTOMATION-04.03.json`](./AI%20AUTOMATION-04.03.json) — Flujo complementario

### Ejemplo adicional: Parsing de Documento y Resumen
**Archivo**: [`ejemplo-parsing-documento.json`](./ejemplo-parsing-documento.json)

Workflow que recibe un texto (simulando un documento parseado) y genera un resumen estructurado con puntos clave.

```bash
curl -X POST http://localhost:5678/webhook-test/parsear-documento \
  -H "Content-Type: application/json" \
  -d '{"documento": "El paciente Juan Pérez, DNI 30.555.123, ingresó el 15/03/2026 con dolor abdominal agudo. Se realizó ecografía que mostró apendicitis. Se programó cirugía para el mismo día. Evolución favorable post-quirúrgica."}'
```

---

## �️ Tutorial: Configurar LlamaCloud para Parsing de PDFs

LlamaCloud / LlamaParse es un servicio que convierte PDFs complejos (con tablas, imágenes, columnas) en texto limpio y estructurado. Tiene un free tier de 1.000 páginas por día.

### 🟢 Paso 1: Crear tu Cuenta en LlamaCloud

1. Entrá a [https://cloud.llamaindex.ai](https://cloud.llamaindex.ai)
2. Hacé clic en **"Sign Up"** — podés usar tu cuenta de Google o GitHub
3. Confirmá tu email si te lo pide

### 🔵 Paso 2: Conseguir tu API Key

1. Una vez adentro, en el menú de la izquierda hacé clic en **"API Keys"**
2. Hacé clic en **"Generate New Key"**
3. Poné un nombre (ej: `Coderhouse-n8n`) y hacé clic en **"Create"**
4. Copiá la key que aparece — empieza con `llx-...`

> 💡 **Free Tier**: Tenés 1.000 páginas gratis por día. Más que suficiente para el curso.

### 🟣 Paso 3: Usar LlamaParse desde n8n (con HTTP Request)

LlamaParse no tiene un nodo nativo en n8n, así que usamos el nodo **HTTP Request** para llamar a la API.

1. Importá el workflow [`AI AUTOMATION-04.02-llamacloud.json`](./AI%20AUTOMATION-04.02-llamacloud.json) en n8n
2. Buscá el nodo de **HTTP Request** que llama a LlamaCloud
3. En los **headers** del request, vas a ver un campo `Authorization`. Ahí tenés que poner:
   ```
   Bearer llx-TU_API_KEY_ACA
   ```
4. **Alternativa más limpia**: En vez de hardcodear la key en el nodo, podés usar variables de entorno:
   - Agregá en tu `docker-compose.yml`:
     ```yaml
     environment:
       - LLAMA_CLOUD_API_KEY=llx-TU_KEY_ACA
     ```
   - Reiniciá n8n: `docker-compose restart n8n`
   - En el nodo HTTP Request, usá: `Bearer {{ $env.LLAMA_CLOUD_API_KEY }}`

### 🟡 Paso 4: Probar con un PDF Real

1. Tené un PDF a mano (una factura, un informe, lo que sea)
2. El workflow espera que le mandes el PDF como binary data vía webhook
3. Forma fácil de probar: usá la interfaz de n8n, hacé click en el nodo webhook y en "Test" subí el archivo

### 🔧 Errores Posibles

| Problema | Solución |
|:---|:---|
| "401 Unauthorized" | Tu API key está mal. Revisá que empiece con `llx-` y que no tenga espacios extra |
| "429 Rate Limit" | Superaste las 1.000 páginas del free tier. Esperá 24hs o upgradeá el plan |
| El PDF sale como texto basura | Es normal con PDFs escaneados sin OCR. Probá con la opción `parse_mode: "auto"` que activa OCR |
| "File too large" | LlamaParse tiene un límite de tamaño. Dividí el PDF en partes más chicas |
| Tablas se rompen | Asegurate de usar `result_type: "markdown"` en la API — convierte tablas a formato Markdown |

> 💡 **Tip**: Podés probar LlamaParse directo en la web sin n8n entrando a [https://cloud.llamaindex.ai/parse](https://cloud.llamaindex.ai/parse) y subiendo un archivo ahí. Así ves cómo queda el resultado antes de automatizarlo.

---

## �📄 Materiales de la Clase
- **Presentación**: `IA Automation-04.pdf`
- **Temas Teóricos**:
  - `Semana 2 - tema 7.pdf`
