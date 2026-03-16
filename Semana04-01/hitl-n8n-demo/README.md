# HITL Triage Demo (n8n + Ollama + Chat + Webhook)

Archivos incluidos:
- `workflow_hitl_triage_n8n.json`: workflow importable en n8n.
- `chat-simulador/index.html`: mini pagina para simular la decision del medico (aprobar/rechazar).

## 1) Importar workflow en n8n
1. Abri n8n.
2. `Import from File` y elegi `workflow_hitl_triage_n8n.json`.
3. Revisa nodos con credenciales: Chatwoot, MySQL, Telegram.

## 2) Variables de entorno sugeridas
Configuralas en n8n (Docker env o `.env`):

- `CHATWOOT_URL`
- `CHATWOOT_ACCOUNT_ID`
- `CHATWOOT_CONVERSATION_ID`
- `CHATWOOT_TOKEN`
- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_CHAT_ID`
- `APPROVAL_WEBHOOK_URL` (ej: `http://localhost:5678/webhook/aprobacion-medica`)
- `SIMULADOR_CHAT_URL` (ej: `http://localhost:8080`)
- `OLLAMA_BASE_URL` (ej: `http://host.docker.internal:11434` si n8n corre en Docker y Ollama en Windows)

## 3) Base de datos logs
Tabla minima para MySQL:

```sql
CREATE TABLE logs_hitl (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  paciente_id VARCHAR(128) NOT NULL,
  tiempo_ia DATETIME NULL,
  tiempo_humano DATETIME NULL,
  decision VARCHAR(32) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Importante:
- n8n no crea esta tabla automaticamente.
- tenes que crearla vos en tu MySQL local (por ejemplo en la DB `hitl_demo`).
- deje un script listo en `mysql_init_logs_hitl.sql`.

Chequeo rapido en MySQL:

```sql
USE hitl_demo;
SHOW TABLES;
SELECT * FROM logs_hitl ORDER BY id DESC LIMIT 20;
```

Fallback si falla MySQL:
- los 3 nodos (`MySQL - Log Aprobado`, `MySQL - Log Rechazado`, `MySQL - Log Timeout`) tienen manejo de error activado.
- si falla un `INSERT`, el flujo no queda silencioso: pasa por `IF - Error en Log MySQL?` y dispara `HTTP - Fallback Error MySQL`.
- ese fallback manda alerta por Telegram con el detalle del error SQL.

## 4) Ejemplo de payload al webhook inicial
`POST /webhook/recepcion-paciente`

```json
{
  "id": "P-1001",
  "nombre": "Juan Perez",
  "sintoma": "Dolor abdominal y fiebre",
  "imagen_url": "https://picsum.photos/seed/medica/512/512"
}
```

Importante con `imagen_url`:
- tiene que ser una URL directa a la imagen (`.jpg`, `.png`, etc.), no una pagina HTML.
- si pegas una URL de Google Images tipo `google.com/imgres?...&imgurl=...`, el workflow ahora intenta extraer `imgurl` automaticamente.
- si el servidor remoto devuelve HTML en vez de imagen, el nodo `Code - Preparar Prompt Vision` va a cortar con un error mas claro.

## 5) Levantar el simulador de chat
Desde `chat-simulador/`:

```powershell
python -m http.server 8080
```

Luego abrir `http://localhost:8080`.

Si queres abrirlo ya con datos, usa algo asi:
`http://localhost:8080/?webhook=http%3A%2F%2Flocalhost%3A5678%2Fwebhook%2Faprobacion-medica&paciente_id=P-1001`

## 5) Modo simulador vs Chatwoot
Hoy el workflow esta adaptado para usar primero el simulador, no Chatwoot.

Eso significa:
- `Set - Armar Links Aprobacion1` ahora genera `aprobacion_webhook_url`, `simulador_url` y `mensaje_medico`
- desde ese paso el flujo sigue directo a `Wait for Webhook - Aprobacion1`
- `HTTP - Notificar Medico (Chatwoot)1` queda como nodo opcional por si despues queres mandar el mismo mensaje a Chatwoot

Para probarlo simple:
- ejecuta hasta `Set - Armar Links Aprobacion1`
- copia `simulador_url` del output
- abri esa URL en el navegador
- desde la pagina aproba o rechaza y eso reanuda el `Wait`

De donde sale el webhook de aprobacion:
- no lo tenes que escribir a mano
- `Wait for Webhook - Aprobacion1` lo genera en runtime como `{{$execution.resumeUrl}}`
- `Set - Armar Links Aprobacion1` ya usa ese valor para construir `simulador_url`

Si en `simulador_url` no aparece nada:
- ejecuta el flujo desde el webhook inicial, no el nodo suelto
- revisa el output del nodo `Set - Armar Links Aprobacion1` dentro de esa ejecucion

Si queres usar Chatwoot despues, reconecta `Set - Armar Links Aprobacion1` hacia `HTTP - Notificar Medico (Chatwoot)1` y desde ahi al `Wait`.

## 5.1) Ollama y el error de JSON/conexion
Si el nodo `Ollama Vision - LLaVA` muestra `JSON parameter needs to be valid JSON`, no pegues expresiones sueltas dentro del JSON visual. En este workflow el body ya queda resuelto como una expresion completa que devuelve un objeto.

Si despues aparece `The service refused the connection`, entonces ya no es JSON: es conectividad.

Usa segun tu caso:
- n8n en Docker + Ollama en Windows host: `OLLAMA_BASE_URL=http://host.docker.internal:11434`
- n8n y Ollama en el mismo docker-compose: `OLLAMA_BASE_URL=http://ollama:11434`
- n8n fuera de Docker: `OLLAMA_BASE_URL=http://localhost:11434`

## 6) Detalle importante sobre el timeout paralelo
Este workflow respeta la arquitectura didactica (wait de aprobacion + wait de timeout en paralelo).
En ejecuciones reales, este patron puede disparar timeout incluso si hubo aprobacion antes.

Para produccion, se recomienda:
- usar un solo `Wait` con control de tiempo maximo, o
- persistir estado en DB/Redis y validar estado antes de mandar alerta de timeout.
