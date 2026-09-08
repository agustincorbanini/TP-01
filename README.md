# TP-01 — Proyecto Integrador: Agente Autónomo para E-commerce de Suplementos

Repositorio de entregas prácticas del trayecto avanzado. El proyecto es acumulativo: cada
checkpoint parte del workflow de n8n del módulo anterior y le suma una capa nueva de
funcionalidad. Todos los archivos `.json` son workflows exportados de n8n
(**Workflow → Download**) y se importan desde n8n con **Import from File**.

## Autor
Agustín Corbanini

## Contenido del repositorio

| Archivo | Módulo / Checkpoint | Qué agrega este workflow |
|---|---|---|
| `checkpoint1_agustin_corbanini.json` | Módulo 1 | Versión inicial del workflow: recepción de consultas por Webhook y clasificación básica con AI Agent. |
| `checkpoint4_agustin_corbanini.json` | **Módulo 4** — Sincronización del Cerebro Agéntico con Ecosistemas de Negocio | Extiende el workflow del Módulo 3 (memoria + resumen en Airtable) sumando 3 integraciones reales vía OAuth2: **Gmail** (casilla de soporte), **HubSpot** (CRM) y **Slack** (canal de operaciones). Ver detalle abajo. |

> ⚠️ Nota de nomenclatura: los nombres de archivo indican explícitamente el número de
> checkpoint/módulo al que corresponden, para evitar confusión entre entregas de distintas
> unidades del trayecto.

## Detalle: `checkpoint4_agustin_corbanini.json` (Módulo 4)

### Cómo importarlo
1. Descargar el archivo desde este repositorio.
2. En n8n: menú **"..."** → **Import from File** → seleccionar el `.json` descargado.
3. El canvas se abre completo, con el flujo original del Módulo 3 (Webhook → memoria en
   Airtable → Summarization) y el nuevo circuito de soporte por Gmail en paralelo.

### Herramientas conectadas (requieren credenciales OAuth2 propias del usuario que lo importe)
- **Gmail** (Google Workspace) — trigger de entrada y creación de borradores.
- **HubSpot** — CRM, resource `Contact`.
- **Slack** — canal `#general-operaciones`.

> Al importar, cada nodo de estas 3 herramientas va a pedir que se vuelva a autenticar con
> credenciales propias (las credenciales OAuth2 no se exportan por seguridad).

### Evidencia por criterio de la rúbrica

| # | Criterio | Nodo(s) en el workflow | Evidencia |
|---|---|---|---|
| ① | IF anti auto-reply (corta el bucle infinito) | `Gmail Trigger` → `If2` | Condición: `subject` no contiene "Auto-reply" / "Out of office" / "Undeliverable" **AND** `from` no contiene "no-reply". Testeado con un mail real (Mercado Libre) → resultado `true` (mail válido, no bloqueado). |
| ② | Look up antes de Create (evita Error 409) | `Search contacts` → `If3` → `Create or update a contact` | Búsqueda por email en HubSpot antes de decidir crear/actualizar. Testeado: contacto nuevo creado con `vid` confirmado en la respuesta de HubSpot. |
| ③ | Create Draft (Human-in-the-loop) | `Create a draft` (Gmail) | La respuesta generada por IA se guarda como borrador (`labelIds: ["DRAFT"]`), no se envía automáticamente. |
| ④ | Set de limpieza de payload antes de Slack | `Edit Fields4` / `Edit Fields5` | Payload reducido a campos esenciales (nombre, email, asunto, respuesta) antes de las llamadas a HubSpot y Slack, evitando arrastrar el HTML/metadata pesado del mail original. |

### Test realizado (evidencia funcional)
- Mail de prueba: asunto *"Consulta sobre proteína Whey Gold Standard"*.
- Resultado: pasó el filtro anti auto-reply → el AI Agent generó una respuesta →
  se creó el contacto en HubSpot → se guardó el borrador en Gmail → se notificó
  al canal de Slack `#general-operaciones`.
- Capturas de cada paso disponibles en el desarrollo de la entrega (chat de soporte del
  curso / documentación adicional del alumno).

## Cómo se conecta con el resto del proyecto
Este checkpoint no reemplaza el Manager de los módulos anteriores: lo **extiende**. El
Webhook original (consultas de WhatsApp/chat) y el nuevo Gmail Trigger conviven en el mismo
canvas como dos puntos de entrada independientes que confluyen en la misma lógica de negocio
del proyecto "Suplementos Pro".
