# Clase 07: Human-in-the-Loop (HITL)

Esta clase cubre la integración de la supervisión humana en flujos de trabajo automatizados para garantizar la calidad en procesos críticos.

---

## 📚 Conceptos Clave

### ¿Qué es Human-in-the-Loop?
**HITL** es un patrón donde la IA propone una acción pero un **humano tiene que aprobarla** antes de que se ejecute. Es fundamental en procesos donde un error puede ser costoso o peligroso (salud, finanzas, legal).

### ¿Cuándo usar HITL?
| Escenario | Sin HITL | Con HITL |
|:---|:---|:---|
| **Clasificar emails** | IA mueve automáticamente a carpetas | OK — error menor, reversible |
| **Diagnóstico médico** | IA responde directo al paciente | PELIGROSO — necesita revisión humana |
| **Enviar factura** | IA genera y envía automáticamente | RIESGOSO — un error puede ser legal |
| **Responder FAQs** | IA responde preguntas comunes | OK si son respuestas verificadas |

### Patrón de Aprobación
```
[Input] → [IA Procesa] → [¿Confianza alta?]
                              ↓ SÍ          ↓ NO
                         [Ejecutar]    [Cola de Revisión Humana]
                                              ↓
                                       [Humano Revisa]
                                         ↓ APRUEBA      ↓ RECHAZA
                                       [Ejecutar]    [Descartar/Corregir]
```

### Niveles de Supervisión Humana
| Nivel | Descripción | Ejemplo |
|:---|:---|:---|
| **Full Auto** | La IA decide y ejecuta sin humano | Chatbot de clima |
| **Human-on-the-loop** | El humano monitorea pero no frena | Dashboard de alertas |
| **Human-in-the-loop** | El humano aprueba/rechaza antes de ejecutar | Triage médico con aprobación |
| **Human-in-command** | El humano decide qué hace la IA | Copilot (sugiere, vos decidís) |

### Chat en Tiempo Real para HITL
En el proyecto de esta clase usamos un **simulador de chat** que permite al humano ver la propuesta de la IA y decidir en tiempo real si la aprueba o la rechaza.

---

## 🧪 Workflows

### Proyecto: HITL Triage Demo
Ubicado en la carpeta `hitl-n8n-demo/`:
- **Workflow n8n**: [`workflow_hitl_triage_n8n.json`](./hitl-n8n-demo/workflow_hitl_triage_n8n.json)
- **Simulador de chat**: [`chat-simulador/index.html`](./hitl-n8n-demo/chat-simulador/index.html)
- **SQL para logs**: [`mysql_init_logs_hitl.sql`](./hitl-n8n-demo/mysql_init_logs_hitl.sql)
- **Guía específica**: [`hitl-n8n-demo/README.md`](./hitl-n8n-demo/README.md)

### Ejemplo adicional: Aprobación Humana Simple
**Archivo**: [`ejemplo-aprobacion-humana.json`](./ejemplo-aprobacion-humana.json)

Workflow que recibe una solicitud, la IA genera una respuesta, y se pausa hasta que un humano la apruebe o rechace vía un segundo webhook.

```bash
# Paso 1: Enviar solicitud (la IA genera respuesta y queda en espera)
curl -X POST http://localhost:5678/webhook-test/solicitud-hitl \
  -H "Content-Type: application/json" \
  -d '{"consulta": "Necesito autorizar una devolución de $50.000", "usuario": "juan.perez"}'

# Paso 2: Aprobar o rechazar (el humano decide)
curl -X POST http://localhost:5678/webhook-test/decision-hitl \
  -H "Content-Type: application/json" \
  -d '{"decision": "aprobado", "revisor": "supervisor.garcia"}'
```

---

## �️ Tutorial: Levantar el Sistema HITL Completo (Paso a Paso)

### 🟢 Paso 1: Levantar MySQL con Docker

El proyecto HITL guarda los logs de aprobaciones en MySQL. Lo levantamos con Docker:

1. Abrí una terminal en la carpeta `hitl-n8n-demo`:
   ```bash
   cd Clase07/hitl-n8n-demo
   ```

2. Si no tenés un `docker-compose.yml` con MySQL, podés levantar uno rápido con este comando:
   ```bash
   docker run -d --name mysql-hitl -e MYSQL_ROOT_PASSWORD=hitl123 -e MYSQL_DATABASE=hitl_logs -p 3306:3306 mysql:8
   ```

3. Esperá 15-20 segundos a que MySQL arranque, y después ejecutá el script SQL para crear las tablas:
   ```bash
   docker exec -i mysql-hitl mysql -uroot -phitl123 hitl_logs < mysql_init_logs_hitl.sql
   ```

4. Verificá que la tabla se creó:
   ```bash
   docker exec -it mysql-hitl mysql -uroot -phitl123 -e "USE hitl_logs; SHOW TABLES;"
   ```
   Deberías ver la tabla de logs listada.

> ⚠️ **Error "port 3306 already in use"**: Ya tenés otro MySQL corriendo. Cambiá el puerto: `-p 3307:3306` y usá `3307` en n8n.

### 🔵 Paso 2: Configurar MySQL en n8n

1. Abrí n8n → **http://localhost:5678**
2. Menú izquierdo → **"Credentials"** (🔑) → **"Add Credential"** → buscá **"MySQL"**
3. Completá los campos:
   - **Host**: `host.docker.internal` (porque n8n corre en Docker)
   - **Port**: `3306`
   - **Database**: `hitl_logs`
   - **User**: `root`
   - **Password**: `hitl123`
4. Hacé clic en **"Test Connection"** — si dice "Connection successful", guardá

> ⚠️ **Error "ECONNREFUSED"**: Si usaste Docker para MySQL, y n8n también está en Docker, probá con `host.docker.internal` como host. Si no anda, probá con la IP de tu máquina (`ipconfig` en Windows).

### 🟣 Paso 3: Importar el Workflow HITL

1. Importá [`workflow_hitl_triage_n8n.json`](./hitl-n8n-demo/workflow_hitl_triage_n8n.json) en n8n
2. Configurá las credenciales en cada nodo:
   - Nodos de **LLM**: Tu credencial de OpenAI/Groq/Ollama
   - Nodos de **MySQL**: La credencial que creaste en el Paso 2
3. **Activá el workflow** (toggle verde arriba a la derecha) — necesario para que los webhooks funcionen en el chat simulador

### 🟡 Paso 4: Usar el Chat Simulador

El simulador de chat te permite ver en tiempo real cómo un "humano" aprueba o rechaza las decisiones de la IA.

1. Abrí el archivo [`chat-simulador/index.html`](./hitl-n8n-demo/chat-simulador/index.html) en el navegador:
   - **Opción simple**: Doble clic en el archivo desde el explorador de archivos
   - **Si no funciona**: Abrí una terminal en `Clase07/hitl-n8n-demo/chat-simulador` y corré:
     ```bash
     python -m http.server 8081
     ```
     Entrá a `http://localhost:8081` en el navegador

2. Escribí un síntoma en el chat (ej: "Me duele mucho el pecho")
3. La IA va a generar una respuesta y te la va a mostrar para que la revises
4. Hacé clic en **"Aprobar"** o **"Rechazar"**
5. El resultado se guarda en MySQL y se devuelve la respuesta

> ⚠️ **El chat no conecta con n8n**: Revisá que:
> - El workflow esté **activado** (no solo en modo test)
> - Las URLs de los webhooks en el `index.html` coincidan con las de tu n8n
> - No tengas bloqueadores de CORS (usá el server local de Python)

### 🔧 Errores Posibles

| Problema | Solución |
|:---|:---|
| MySQL no arranca | Esperá 20 segundos más — la primera vez tarda en inicializar |
| "Access denied for user root" | Verificá que la contraseña sea `hitl123` (o la que pusiste) |
| El chat muestra "Error de conexión" | El workflow de n8n no está activado o la URL del webhook es incorrecta |
| No se guardan los logs en MySQL | Revisá la credencial de MySQL en n8n — hacé "Test Connection" |
| La tabla no existe | Corré de nuevo el script SQL del Paso 1 |

### 🧹 Limpieza (cuando termines)

Para parar y borrar el contenedor de MySQL:
```bash
docker stop mysql-hitl
docker rm mysql-hitl
```

---

## �📄 Materiales de la Clase
- **Presentación**: `IA Automation-07.pdf`
- **Temas Teóricos**:
  - `Semana 4 - tema 10.pdf`
