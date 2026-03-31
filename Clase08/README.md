# Clase 08: Escalabilidad y Mejores Prácticas

Enfoque en cómo llevar las automatizaciones con IA a producción de manera segura, eficiente y mantenible.

---

## 📚 Conceptos Clave

### Error Handling en Workflows con IA
A diferencia del software tradicional, las APIs de IA pueden fallar por motivos impredecibles: rate limits, timeouts, respuestas mal formateadas, alucinaciones.

| Tipo de Error | Causa | Estrategia |
|:---|:---|:---|
| **Rate Limit (429)** | Demasiadas llamadas a la API | Retry con backoff exponencial |
| **Timeout** | El modelo tarda mucho (textos largos) | Aumentar timeout, dividir en chunks |
| **JSON Inválido** | El LLM no respetó el formato pedido | Try/Catch + re-prompt |
| **Alucinación** | El modelo inventa datos | Validación post-respuesta |
| **API Down** | El proveedor tiene problemas | Fallback a modelo alternativo |

### Patrón de Retry con Backoff Exponencial
```
Intento 1: Falla → Esperar 1 segundo
Intento 2: Falla → Esperar 2 segundos
Intento 3: Falla → Esperar 4 segundos
Intento 4: Falla → Notificar al humano
```

### Optimización de Costos de Tokens
| Técnica | Ahorro Estimado | Cómo |
|:---|:---|:---|
| **Prompts cortos** | 30-50% | Eliminar instrucciones redundantes |
| **Modelo más chico** | 70-90% | Usar GPT-3.5 o Llama 3 8B para tareas simples |
| **Cacheo de respuestas** | Variable | Si la misma pregunta se repite, devolver cache |
| **Batching** | 20-40% | Agrupar múltiples inputs en una sola llamada |
| **Filtrar antes del LLM** | 100% en descartados | Validar input antes de mandar al modelo |

### Seguridad y Privacidad
- **Nunca** enviar datos sensibles (DNI, tarjetas de crédito) a APIs externas sin encriptar
- Usar **modelos locales** (Ollama) para datos confidenciales
- Implementar **sanitización de inputs**: el usuario no debe poder inyectar prompts maliciosos
- Guardar **logs** de todas las interacciones para auditoría

### Variables de Entorno
Nunca hardcodear API keys en los workflows. Usar variables de entorno en Docker Compose:
```yaml
environment:
  - OPENAI_API_KEY=tu_key_aca
```
Y accederlas en n8n con `{{ $env.OPENAI_API_KEY }}`.

### Checklist de Producción
- [ ] Error handling en todos los nodos de API
- [ ] Retry automático con límite de intentos
- [ ] Logs de todas las ejecuciones
- [ ] Variables de entorno (no keys hardcodeadas)
- [ ] Alertas cuando un workflow falla
- [ ] Monitoreo de costos (tokens consumidos)
- [ ] Rate limiting en webhooks públicos
- [ ] Backup de workflows (exportar JSONs)

---

## 🧪 Workflow de Ejemplo: Error Handling y Retry
**Archivo**: [`ejemplo-error-handling.json`](./ejemplo-error-handling.json)

Workflow que demuestra:
1. Recibe una consulta por webhook
2. Intenta llamar al LLM
3. Si falla, reintenta con un modelo alternativo (fallback)
4. Si todo falla, devuelve un mensaje de error controlado

```bash
curl -X POST http://localhost:5678/webhook-test/error-handling \
  -H "Content-Type: application/json" \
  -d '{"pregunta": "¿Cuáles son los efectos secundarios del ibuprofeno?"}'
```

---

## �️ Tutorial: Preparar tu Workflow para Producción (Paso a Paso)

### 🟢 Paso 1: Configurar Variables de Entorno (No Hardcodear Keys)

El error más común es dejar las API keys directamente en los nodos de n8n. Si compartís el workflow o lo subís a GitHub, **cualquiera ve tus keys**. La solución: variables de entorno.

1. Abrí tu archivo `docker-compose.yml` (el de Clase 09, o el que uses para levantar n8n)
2. En la sección `environment`, agregá tus variables:
   ```yaml
   environment:
     - GENERIC_TIMEZONE=America/Argentina/Buenos_Aires
     - N8N_BLOCK_ENV_ACCESS_IN_NODE=false   # ← IMPORTANTE: esto permite usar $env
     - OPENAI_API_KEY=sk-TU_KEY_ACA
     - GROQ_API_KEY=gsk_TU_KEY_ACA
     - LLAMA_CLOUD_API_KEY=llx-TU_KEY_ACA
     - ENTORNO=desarrollo
   ```
3. Reiniciá n8n para que tome las nuevas variables:
   ```bash
   docker-compose restart n8n
   ```
4. Ahora en cualquier nodo de n8n, podés acceder a esas variables con:
   ```
   {{ $env.OPENAI_API_KEY }}
   {{ $env.ENTORNO }}
   ```

> ⚠️ **"$env is not defined"**: Significa que `N8N_BLOCK_ENV_ACCESS_IN_NODE` está en `true` o no existe. Agregá la línea `- N8N_BLOCK_ENV_ACCESS_IN_NODE=false` en tu docker-compose y reiniciá.

### 🔵 Paso 2: Agregar Error Handling a un Nodo de LLM

1. En n8n, hacé clic en el nodo que llama a la API del LLM
2. Hacé clic en el icono de **engranaje** (⚙️) → **Settings**
3. En **"On Error"**, cambiá de "Stop Workflow" a **"Continue (using error output)"**
4. Vas a ver que aparece una **segunda salida** (roja) en el nodo
5. Conectá esa salida roja a un nodo alternativo:
   - Otro LLM como fallback (ej: si falla OpenAI, intentar con Groq)
   - Un nodo de código que prepare un mensaje de error amigable
   - Un nodo de notificación (email, Telegram, Slack)

### 🟣 Paso 3: Implementar Retry para Rate Limits

Cuando la API de OpenAI o Groq te tira un **429 (Rate Limit)**, no sirve reintentar inmediatamente. Necesitás esperar.

1. Hacé clic en el nodo que llama al LLM → ⚙️ Settings
2. En **"Retry On Fail"**, activalo en **"On"**
3. Configurá:
   - **Max Tries**: `3` (intenta hasta 3 veces)
   - **Wait Between Tries (ms)**: `2000` (espera 2 segundos entre intentos)
4. Opcionalmente, podés ir aumentando el tiempo con cada intento (backoff exponencial) — para eso usá un nodo Code que maneje la lógica

### 🟡 Paso 4: Checklist Antes de Activar en Producción

Recorré esta lista antes de poner tu workflow en modo "activo":

| # | Check | Cómo verificarlo |
|:---:|:---|:---|
| 1 | ✅ Todas las API keys están en variables de entorno | Buscá `sk-`, `gsk_`, `llx-` en los nodos — no deberían estar |
| 2 | ✅ Todos los nodos de API tienen error handling | Revisá que digan "Continue" en vez de "Stop Workflow" |
| 3 | ✅ Hay retry en nodos que llaman APIs externas | Click en cada nodo → Settings → "Retry On Fail": On |
| 4 | ✅ Hay un nodo de fallback si todo falla | Conectá la salida de error del último retry a un mensaje amigable |
| 5 | ✅ Los webhooks tienen autenticación | En el nodo Webhook → Header Auth o Basic Auth (no dejar abiertos) |
| 6 | ✅ Hay logs de las ejecuciones | n8n guarda ejecuciones por defecto. Verificá en Settings → Execution Data |
| 7 | ✅ Testeaste con casos buenos Y malos | Mandá un input que haga fallar al LLM a propósito |
| 8 | ✅ El workflow tiene nombre descriptivo | Renombrá de "My Workflow" a algo como "Triage Médico v2 - Producción" |

### 🔧 Errores Posibles en Producción

| Problema | Causa Probable | Solución |
|:---|:---|:---|
| El workflow dejó de funcionar de repente | Se venció o rotó la API key | Regenerá la key y actualizá la variable de entorno |
| Las respuestas son más malas que antes | Actualizaron el modelo (ej: GPT-4 → 4o) | Fijá la versión del modelo en el nodo (ej: `gpt-4o-2024-08-06`) |
| Costos altísimos inesperados | Prompts muy largos o un loop infinito | Poné un límite de tokens en el nodo LLM + Max Iterations en agentes |
| "Execution timed out" | El modelo tarda mucho con inputs grandes | Aumentá el timeout en n8n Settings o dividí el input en partes |

> 💡 **Tip Profe**: Exportá siempre una versión "limpia" de tu workflow (sin credenciales) antes de compartirlo. En n8n → tres puntitos (⋮) → Export → se genera un JSON sin las keys adentro.

---

## �📄 Materiales de la Clase
- **Temas Teóricos**:
  - `Semana 4 - tema 11.pdf`
