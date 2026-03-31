# Clase 09: Docker e Infraestructura para n8n

Esta clase cubre cómo levantar y configurar tu propia instancia de n8n usando Docker Compose, con foco en variables de entorno, persistencia de datos y configuración de producción.

---

## 📚 Conceptos Clave

### ¿Qué es Docker?
Docker permite ejecutar aplicaciones en **contenedores** — entornos aislados que incluyen todo lo necesario para correr la app (código, dependencias, configuración). Es como una "máquina virtual liviana".

### ¿Qué es Docker Compose?
Docker Compose permite definir y ejecutar **múltiples contenedores** con un solo archivo YAML. En vez de escribir comandos largos de Docker, definís todo en `docker-compose.yml` y levantás con un solo comando.

### Anatomía del docker-compose.yml

```yaml
version: '3.8'

services:
  n8n:                                    # Nombre del servicio
    image: docker.n8n.io/n8nio/n8n        # Imagen de Docker a usar
    container_name: n8n_produccion        # Nombre del contenedor
    restart: unless-stopped               # Se reinicia si se cae
    ports:
      - "5678:5678"                       # Puerto externo:interno
    environment:                          # Variables de entorno
      - GENERIC_TIMEZONE=America/Argentina/Buenos_Aires
      - N8N_BLOCK_ENV_ACCESS_IN_NODE=false
    volumes:
      - n8n_data:/home/node/.n8n          # Datos persistentes

volumes:
  n8n_data:                               # Definición del volumen
    name: n8n_datos_persistentes
```

### Conceptos Docker Clave

| Concepto | Descripción |
|:---|:---|
| **Image** | La "receta" del contenedor. Se descarga del registry (Docker Hub). |
| **Container** | Una instancia corriendo de una imagen. Es donde vive tu app. |
| **Volume** | Almacenamiento persistente. Sin volumen, los datos se pierden al reiniciar. |
| **Port Mapping** | Conecta un puerto de tu PC con un puerto dentro del contenedor. |
| **Environment Variables** | Variables que configuran el comportamiento del contenedor sin modificar código. |
| **Restart Policy** | Qué hacer si el contenedor se cae: `always`, `unless-stopped`, `no`. |

### Comandos Esenciales

```bash
# Levantar todos los servicios (en background)
docker-compose up -d

# Ver logs en tiempo real
docker-compose logs -f

# Parar todos los servicios
docker-compose down

# Parar y borrar los volúmenes (¡CUIDADO! Borra datos)
docker-compose down -v

# Ver qué contenedores están corriendo
docker ps

# Reiniciar un servicio específico
docker-compose restart n8n
```

### Variables de Entorno en n8n
Las variables de entorno permiten configurar n8n sin tocar el código. Se definen en el `docker-compose.yml` y se acceden en n8n con `{{ $env.NOMBRE_VARIABLE }}`.

```yaml
environment:
  - MODO_DRY_RUN=true           # Para testear sin ejecutar acciones reales
  - ENTORNO=desarrollo          # Saber en qué ambiente estás
  - API_KEY=tu_key_aca          # Credenciales
  - CLIENT_ID=nombre_cliente    # Identificador del cliente
```

### Ollama + Docker (Modelos Locales)
Para usar modelos de IA localmente (sin enviar datos a la nube), usamos **Ollama** junto con n8n. Desde dentro de Docker, Ollama se accede con `http://host.docker.internal:11434`.

```bash
# Instalar modelos
ollama pull llama3
ollama pull nomic-embed-text

# Verificar que están disponibles
ollama list
```

---

## 🧪 Workflows

### Infraestructura: Docker Compose para n8n
**Archivo**: [`docker-compose.yml`](./docker-compose.yml)

Archivo listo para levantar n8n con Docker, incluyendo timezone de Argentina, variables de entorno y volúmenes persistentes.

```bash
docker-compose up -d
```

### Ejemplo adicional: Health Check del Sistema
**Archivo**: [`ejemplo-health-check.json`](./ejemplo-health-check.json)

Workflow que verifica que todos los componentes del sistema estén funcionando:
1. Chequea que n8n responda
2. Verifica la conexión con Ollama
3. Lista los modelos disponibles
4. Devuelve un reporte de salud del sistema

```bash
curl -X POST http://localhost:5678/webhook-test/health-check
```

---

## �️ Tutorial: De Cero a n8n Funcionando (Guía Completa)

### 🟢 Paso 1: Instalar Docker Desktop

1. Entrá a [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Hacé clic en **"Download for Windows"** (o Mac/Linux)
3. Ejecutá el instalador — siguiente, siguiente, finalizar
4. Reiniciá la PC si te lo pide
5. Abrí **Docker Desktop** — la primera vez puede tardar 1-2 minutos
6. Verificá que funcione abriendo PowerShell:
   ```bash
   docker --version
   docker-compose --version
   ```
   Deberías ver las versiones de ambos.

> ⚠️ **Windows: "WSL 2 not installed"**: Docker necesita WSL 2. Si te sale este error:
> 1. Abrí PowerShell como Administrador
> 2. Corré: `wsl --install`
> 3. Reiniciá la PC
> 4. Volvé a abrir Docker Desktop

> ⚠️ **Windows: "Hyper-V not enabled"**: Abrí PowerShell como Admin y corré:
> ```bash
> Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
> ```
> Reiniciá la PC.

### 🔵 Paso 2: Levantar n8n con Docker Compose

1. Abrí una terminal y navegá a la carpeta Clase09:
   ```bash
   cd Clase09
   ```
2. Mirá el archivo [`docker-compose.yml`](./docker-compose.yml) — ahí está toda la configuración de n8n
3. Levantá n8n con:
   ```bash
   docker-compose up -d
   ```
4. Esperá 10-15 segundos y abrí el navegador en: **http://localhost:5678**
5. La primera vez te va a pedir:
   - **Email**: Poné cualquiera (ej: `admin@local.com`)
   - **Nombre y apellido**: Lo que quieras
   - **Contraseña**: Elegí una (mínimo 8 caracteres)
6. Hacé clic en **"Next"** y listo — estás adentro de n8n 🎉

> 💡 **Tip**: n8n guarda tus datos en un **volumen Docker** (`n8n_datos_persistentes`). Si parás y volvés a levantar el contenedor, tus workflows siguen ahí.

### 🟣 Paso 3: Instalar Ollama (IA Local)

1. Entrá a [https://ollama.com/download](https://ollama.com/download) y descargá el instalador
2. Instalá Ollama — va a aparecer un ícono en tu barra de notificaciones
3. Abrí una terminal y descargá los modelos que usamos en el curso:

```bash
# Modelo de chat (el principal)
ollama pull llama3

# Modelo de embeddings (para RAG y evaluaciones)
ollama pull nomic-embed-text

# Modelo de visión (opcional, para Clase 05)
ollama pull llava
```

4. Verificá que se instalaron:
```bash
ollama list
```

5. Probá que funcione:
```bash
ollama run llama3 "Decime hola en español"
```

> ⚠️ **"Error: could not connect"**: Ollama no está corriendo. Buscá el ícono en la barra de notificaciones y hacé clic derecho → "Start".

### 🟡 Paso 4: Conectar Ollama con n8n

Acá está el truco: n8n corre **dentro de Docker**, y Ollama corre **fuera de Docker** (en tu PC). No pueden hablar por `localhost` — hay que usar `host.docker.internal`.

1. Abrí n8n → **http://localhost:5678**
2. Menú izquierdo → **"Credentials"** (🔑) → **"Add Credential"**
3. Buscá **"Ollama"** → hacé clic
4. En **"Base URL"**, poné:
   ```
   http://host.docker.internal:11434
   ```
5. Hacé clic en **"Save"**

Ahora cuando uses un nodo de Ollama en un workflow, seleccionás esta credencial y elegís el modelo (`llama3`, `nomic-embed-text`, `llava`, etc.).

### 🔴 Paso 5: Verificar que Todo Funciona

Importá el workflow [`ejemplo-health-check.json`](./ejemplo-health-check.json) y ejecutalo:

```bash
curl -X POST http://localhost:5678/webhook-test/health-check
```

Debería devolverte un reporte diciendo:
- ✅ n8n está corriendo
- ✅ Ollama está disponible
- ✅ Modelos descargados: llama3, nomic-embed-text

Si algo falla, el reporte te dice qué componente tiene problemas.

### 🔧 Errores Posibles

| Problema | Solución |
|:---|:---|
| `docker-compose` no reconocido | Probá con `docker compose` (sin guión). Las versiones nuevas de Docker cambiaron el comando |
| Puerto 5678 ocupado | Cambiá `"5678:5678"` por `"5679:5678"` en docker-compose.yml |
| n8n no arranca | Corré `docker-compose logs n8n` para ver el error |
| Ollama no se conecta desde n8n | Usá `http://host.docker.internal:11434`, NO `http://localhost:11434` |
| Los datos se borraron al reiniciar | Verificá que el volumen `n8n_data` esté definido en docker-compose.yml |
| "Permission denied" en Linux/Mac | Agregá tu usuario al grupo docker: `sudo usermod -aG docker $USER` y relogueá |
| Docker Desktop no abre | Asegurate de que la virtualización esté habilitada en la BIOS (VT-x / AMD-V) |

> 💡 **Pro tip**: Si querés ver qué pasa adentro del contenedor de n8n:
> ```bash
> docker exec -it n8n_produccion sh
> ```
> Esto te abre una terminal dentro del contenedor.

---

## �📄 Materiales de la Clase
- **Docker Compose**: `docker-compose.yml` listo para usar
