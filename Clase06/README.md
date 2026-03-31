# Clase 06: Agentes de IA y Orquestación

Introducción a los agentes autónomos capaces de utilizar herramientas para resolver tareas complejas.

---

## 📚 Conceptos Clave

### ¿Qué es un Agente de IA?
Un **agente** es un LLM que puede **tomar decisiones y ejecutar acciones** por sí solo. A diferencia de un chain (cadena lineal), el agente decide en cada paso qué herramienta usar, la ejecuta, lee el resultado, y decide si necesita hacer algo más.

### Chain vs Agent
| Aspecto | Chain (Cadena) | Agent (Agente) |
|:---|:---|:---|
| **Flujo** | Lineal: A → B → C | Dinámico: decide en cada paso |
| **Decisiones** | No toma decisiones | Elige qué herramienta usar |
| **Iteraciones** | Una sola pasada | Puede hacer varias vueltas (loops) |
| **Ejemplo** | "Tomá este texto y resumilo" | "Investigá sobre X, buscá en Google, leé el PDF, y armá un informe" |

### Patrón ReAct (Reason + Act)
La mayoría de los agentes usan el patrón **ReAct**:
1. **Reason** (Razonar): "El usuario quiere saber el clima. Necesito usar la herramienta de clima."
2. **Act** (Actuar): Ejecuta la herramienta de clima con los parámetros correctos
3. **Observe** (Observar): Lee el resultado de la herramienta
4. **Repeat o Respond**: Si necesita más info, vuelve al paso 1. Si ya tiene todo, responde.

```
[Pregunta] → [Razonar] → [Elegir Herramienta] → [Ejecutar] → [Observar Resultado] → [¿Suficiente?]
                                                                                          ↓         ↓
                                                                                         NO → [Volver a Razonar]
                                                                                         SÍ → [Responder al usuario]
```

### Tool-Use (Uso de Herramientas)
Las herramientas son funciones que el agente puede llamar. En n8n, cada herramienta es un sub-workflow o un nodo conectado al agente.

| Herramienta | Qué hace | Ejemplo |
|:---|:---|:---|
| **Calculator** | Hace cálculos matemáticos | "¿Cuánto es el 21% de IVA de $5000?" |
| **Web Search** | Busca en internet | "¿Cuál es la cotización del dólar hoy?" |
| **Code Executor** | Ejecuta código | "Generá un CSV con estos datos" |
| **HTTP Request** | Llama a cualquier API | "Consultá el stock del producto X" |
| **Workflow Tool** | Ejecuta otro workflow de n8n | "Procesá estos datos con el flujo de triage" |

### Orquestación de Agentes
Cuando un solo agente no alcanza, podés usar **múltiples agentes** que se coordinan:
- **Agente Planificador**: Decide qué pasos seguir
- **Agente Ejecutor**: Ejecuta cada paso
- **Agente Revisor**: Verifica la calidad del resultado

---

## 🧪 Workflows

### Práctica principal: Agentes Inteligentes
- [`AI AUTOMATION-06.json`](./AI%20AUTOMATION-06.json) — Agente con herramientas
- [`AI AUTOMATION-06.01.json`](./AI%20AUTOMATION-06.01.json) — Variante de agente

### Ejemplo adicional: Agente con Herramientas
**Archivo**: [`ejemplo-agente-herramientas.json`](./ejemplo-agente-herramientas.json)

Agente que recibe una pregunta y decide si necesita calcular, buscar información o responder directamente.

```bash
curl -X POST http://localhost:5678/webhook-test/agente \
  -H "Content-Type: application/json" \
  -d '{"pregunta": "Si tengo 3 productos a $1500 cada uno, ¿cuánto es el total con 21% de IVA?"}'
```

---

## �️ Tutorial: Armar un Agente con Herramientas en n8n

### 🟢 Paso 1: Entender qué es el Nodo "AI Agent" en n8n

El nodo **AI Agent** es diferente al nodo **LLM Chain**. La diferencia clave:
- **LLM Chain**: Le mandás texto → te devuelve texto. Fin. No hace nada más.
- **AI Agent**: Le mandás texto → **piensa** qué herramienta necesita → la usa → lee el resultado → decide si necesita más → responde.

Para crearlo en n8n:
1. En el lienzo, hacé clic en **"+"** para agregar un nodo
2. Buscá **"AI Agent"** y arrastralo
3. Vas a ver que tiene **3 entradas**:
   - 🔌 **Input**: De dónde viene la pregunta (webhook, chat, etc.)
   - 🧠 **Model**: Qué LLM usa para pensar (OpenAI, Ollama, Groq)
   - 🔧 **Tools**: Las herramientas que puede usar

### 🔵 Paso 2: Conectarle un Modelo de IA

1. Hacé clic en el puerto **"Model"** del nodo AI Agent
2. Elegí el modelo. Opciones recomendadas:
   - **OpenAI Chat Model** → `gpt-4o-mini` (barato y capaz)
   - **Ollama Chat Model** → `llama3` (gratis, local)
   - **Groq** → Usá credencial OpenAI con Base URL de Groq (ver tutorial Clase 02)
3. Configurá las credenciales del modelo

### 🟣 Paso 3: Agregarle Herramientas (Tools)

Acá es donde el agente se vuelve poderoso. Para cada herramienta:

1. Hacé clic en el puerto **"Tools"** del nodo AI Agent
2. Elegí qué herramienta agregar:

**Calculator** (Calculadora):
- Buscá **"Calculator"** → agregalo → listo, no necesita configuración
- El agente la usa cuando detecta cálculos matemáticos

**HTTP Request Tool** (Llamar APIs externas):
- Buscá **"HTTP Request Tool"** → configurá la URL, método y headers
- Ejemplo: consultar cotización del dólar, buscar productos, etc.

**Code Tool** (Ejecutar código):
- Buscá **"Code Tool"** → escribí una función JavaScript
- El agente puede ejecutar código cuando lo necesita

**Workflow Tool** (Llamar otro workflow):
- Buscá **"Call n8n Workflow Tool"** → seleccioná otro workflow
- Sirve para conectar agentes entre sí o reutilizar flujos

3. Podés agregar **varias herramientas al mismo agente** — él decide cuándo usar cada una

### 🟡 Paso 4: Probar el Agente

1. Importá [`ejemplo-agente-herramientas.json`](./ejemplo-agente-herramientas.json) en n8n
2. Configurá las credenciales del modelo (apuntá a tu OpenAI, Groq u Ollama)
3. Hacé clic en **"Test Workflow"**
4. Mandá distintos tipos de preguntas para ver cómo elige la herramienta:

```bash
# Pregunta que requiere cálculo → debería usar Calculator
curl -X POST http://localhost:5678/webhook-test/agente -H "Content-Type: application/json" -d "{\"pregunta\": \"Cuanto es 1500 por 3 mas 21 por ciento de IVA?\"}"

# Pregunta general → debería responder directamente sin herramientas
curl -X POST http://localhost:5678/webhook-test/agente -H "Content-Type: application/json" -d "{\"pregunta\": \"Que es la inflacion?\"}"
```

5. Mirá la ejecución en n8n — podés ver el "razonamiento" del agente en cada paso (qué herramienta eligió y por qué)

### 🔧 Errores Posibles

| Problema | Solución |
|:---|:---|
| "No tools available" | El agente no tiene herramientas conectadas. Verificá que el puerto "Tools" tenga al menos un nodo |
| El agente no usa la herramienta | Algunos modelos chicos no son buenos para tool-use. Probá con `gpt-4o-mini` o `llama3-70b-8192` (Groq) |
| "Maximum iterations reached" | El agente entró en loop. Bajá el "Max Iterations" a 5 en las opciones del nodo AI Agent |
| "Tool execution failed" | Revisá la configuración de la herramienta específica (URL, autenticación, etc.) |
| Respuesta lenta | Los agentes hacen múltiples llamadas al LLM. Es normal que tarden más que un chain simple |

> 💡 **Tip**: Para debuggear, hacé clic en la ejecución y mirá cada paso del agente. n8n te muestra el "pensamiento" interno: "Voy a usar Calculator para resolver esto..." — muy útil para entender el ReAct.

---

## �📄 Materiales de la Clase
- **Presentación**: `IA Automation-06.pdf`
- **Temas Teóricos**:
  - `Semana 3 - tema 9.pdf`
