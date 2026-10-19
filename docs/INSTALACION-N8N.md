# Instalación de un flujo FlowKit en n8n

Guía completa, en español, para importar y dejar funcionando cualquiera de los flujos de este repositorio.

---

## 1. Requisitos

| Requisito | Detalle |
|---|---|
| n8n | Versión **1.40 o superior**. Vale tanto n8n Cloud como self-hosted (Docker, VPS, servidor propio). |
| Cuentas de las herramientas | Cada flujo usa una o varias. Los incluidos en la carpeta `workflows/` usan **Google Sheets** y **Telegram**. |
| Navegador | Para importar y editar en el editor de n8n. |

> ¿No tienes n8n todavía? La forma más rápida de probarlo es n8n Cloud (plan gratuito) o levantarlo con Docker:
> `docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n`
> y abrir `http://localhost:5678`.

---

## 2. Importar el flujo

1. Entra en tu n8n y ve a **Workflows**.
2. Arriba a la derecha: menú **⋯ → Import from File…**.
3. Selecciona el archivo `.json` descargado (por ejemplo `captura-leads-web.json`).
4. El flujo se abre en el editor. **Todavía no lo actives**: primero hay que configurarlo.

> También puedes copiar el contenido del `.json` y pegarlo con **Import from Clipboard**. Útil si estás en el móvil.

---

## 3. Configurar credenciales y parámetros

Al importar, verás nodos con un **triángulo rojo**. Es normal: son los que dependen de tus cuentas.

1. **Haz doble clic en el nodo rojo.** En el desplegable *Credential* elige una credencial existente o pulsa **Create new**.
2. Los nodos que no llevan credencial pero sí parámetros (por ejemplo, la hoja de cálculo de destino) hay que rellenarlos a mano:
   - **Google Sheets** → elige el *Spreadsheet* y la *Sheet* donde se guardarán los datos.
   - **Telegram** → elige el chat o canal de destino.
3. Repite con cada nodo hasta que **no quede ningún triángulo rojo**.
4. Pulsa **Save**.

### Credenciales más habituales

**Google Sheets**
1. En n8n: **Credentials → New → Google Sheets OAuth2 API**.
2. Pega tu *Client ID* y *Client Secret* (se crean en Google Cloud Console, APIs y servicios → Credenciales → ID de cliente OAuth).
3. Autoriza el acceso con tu cuenta de Google.
4. *Alternativa sin OAuth*: en n8n existe el tipo de credencial **Google Service Account**; hay que compartir la hoja con el email de la cuenta de servicio.

**Telegram**
1. En Telegram, habla con **@BotFather** → `/newbot` → copia el token.
2. En n8n: **Credentials → New → Telegram API** y pega el token.
3. Escribe un mensaje a tu bot (o añádelo a tu grupo/canal) para que exista el chat.
4. En el nodo Telegram, selecciona ese chat en *Chat ID*.

**Avisos de privacidad**: las credenciales se guardan cifradas en tu instancia de n8n. Revísalas de vez en cuando y revoca las que ya no uses.

---

## 4. Probar antes de activar

1. Con todos los nodos configurados, pulsa **Execute Workflow** (abajo, en el centro).
2. En el nodo de disparo verás la URL de prueba o el formulario. Envíale un dato de mentira (tú mismo, con tu email personal si hace falta).
3. Comprueba que el dato aparece donde toca:
   - en la hoja de cálculo: una fila nueva;
   - en Telegram: el mensaje de aviso.
4. Repite la prueba 2–3 veces y borra las filas de prueba.
5. Cuando funcione, activa el flujo con el interruptor **Active** (arriba a la derecha).

> Los datos usados en las pruebas se quedan en tu Google Sheets: bórralos. No dejes datos de prueba mezclados con datos reales.

---

## 5. Problemas frecuentes

| Síntoma | Causa habitual | Solución |
|---|---|---|
| El nodo de disparo no recibe nada | El flujo está guardado pero **no activo**, o estás usando la URL de prueba en lugar de la de producción | Activa el flujo y usa la URL de producción que muestra el nodo |
| "Credentials not set" | Falta elegir la credencial en ese nodo | Doble clic en el nodo → *Credential* → selecciona o crea |
| Error 403 en Google Sheets | La hoja no está compartida con la cuenta de servicio, o el permiso OAuth es demasiado limitado | Comparte la hoja con el email de la cuenta de servicio, o vuelve a autorizar pidiendo permiso de escritura |
| Telegram: "chat not found" | Nunca has escrito al bot, o el *Chat ID* es incorrecto | Envía `/start` al bot y vuelve a elegir el chat; en grupos, añade el bot y usa el ID del grupo (negativo) |
| Los datos salen vacíos o con el nombre de campo raro | Los nombres de los campos del formulario no coinciden con los del nodo de normalización | Abre el nodo *Normalizar* y ajusta los nombres a la izquierda a los de tu formulario |
| El correo o el mensaje llega con `{{ ... }}` literal | Falta el `=` al principio del campo, o ese campo no admite expresiones | En n8n, las expresiones empiezan por `=`; usa el selector *Expression* del campo |

---

## 6. Pasar de prueba a producción

- **Copia la URL/formulario de producción** del nodo de disparo (no la de test) y sustitúyela en tu web.
- Si usas el disparador de formulario, personaliza los campos y el texto de agradecimiento directamente en el nodo.
- Anota en tu calendario una revisión mensual: que el flujo siga activo y que las filas se estén guardando.

---

## 7. Actualizar un flujo

Este repositorio publica **versiones nuevas** de cada flujo. Para actualizar:

1. Descarga el `.json` nuevo.
2. En n8n, si has personalizado tu versión, **duplica antes tu flujo** (menú ⋯ → Duplicate): así puedes comparar y no pierdes tus cambios.
3. Importa el nuevo y vuelve a configurar credenciales.

Los números de versión y los cambios se anotan en [`CATALOGO.md`](CATALOGO.md).
