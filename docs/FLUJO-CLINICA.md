# Flujo: Clínica — recordatorio de cita + recuperación de cancelaciones

Guía de puesta en marcha del flujo `workflows/clinica/recordatorio-citas.json`.
Tiempo estimado: **30–40 minutos**. No hace falta saber programar.

> **Qué es y qué no es.** Automatiza tres tareas **administrativas** de una clínica: registrar una cita sin retipear datos, recordar al paciente su cita por email 24 h antes con enlaces para confirmar o cancelar, y avisar al equipo cuando se libera un hueco. **No diagnostica, no recomienda tratamientos y no sustituye al personal sanitario.** Los emails solo indican fecha y hora: el motivo de la cita queda en la hoja interna.

---

## 1. Antes de empezar

| Necesitas | Detalle |
|---|---|
| n8n | 1.40 o superior, **con URL pública** (self-hosted con dominio o n8n Cloud) |
| Google Sheets | Una hoja de cálculo tuya |
| Telegram | Un bot (`@BotFather` → `/newbot`) y un chat interno del equipo |
| Email (SMTP) | Una cuenta desde la que enviar los recordatorios (Gmail con contraseña de aplicación, Brevo, Mailgun, tu proveedor de correo…) |

> Los enlaces de confirmación/cancelación son webhooks: si tu n8n solo es accesible en `localhost`, los pacientes no podrán abrirlos desde su móvil. Para probar en local puedes usar un túnel (Cloudflare Tunnel, ngrok), pero para producción necesitas una URL pública estable.

---

## 2. Prepara la hoja de cálculo

Crea una hoja nueva llamada, por ejemplo, **Citas clínica** y en la **primera fila** escribe exactamente estas cabeceras, una por columna y en este orden:

```
ID | Paciente | Telefono | Email | Tratamiento | Fecha | Hora | Estado | Notas | Alta | Actualizado
```

Notas importantes:

- Los nombres **deben coincidir** con los de arriba: el flujo escribe por nombre de columna.
- `Estado` lo gestiona el flujo: `Pendiente` al crear la cita, `Confirmada` o `Cancelada` cuando el paciente pulsa el enlace. También puedes escribir `Atendida` a mano al terminar la consulta.
- `Fecha` admite `2026-10-20` (lo que envía el formulario) o `20/10/2026` (si la escribes a mano). `Hora` en formato 24 h: `09:30`.
- No borres la columna `ID`: es la que permite actualizar la cita correcta cuando el paciente confirma o cancela.
- Si das citas de alta a mano, rellena también un `ID` único y difícil de adivinar (por ejemplo `CITA-20261020-0930-ANA-7f3k9x`). El `ID` viaja dentro del enlace de confirmar/cancelar y actúa como llave: no lo publiques ni lo reutilices entre pacientes.

---

## 3. Importa el flujo

**n8n → Workflows → ⋯ → Import from File…** y elige `recordatorio-citas.json`.

Verás **tres partes** en el lienzo:

- **A) Nueva cita** — formulario → normalizar → guardar → aviso al equipo.
- **B) Recordatorios** — cada día a las 18:00 → leer citas → email a pacientes + resumen al equipo.
- **C) Enlaces del paciente** — webhook → actualizar estado → aviso al equipo → página de respuesta.

Las tres partes viven en el mismo flujo pero **se ejecutan de forma independiente**.

---

## 4. Configura los nodos que faltan

### 4.1 Google Sheets — los tres nodos

En **Guardar cita**, **Leer citas** y **Actualizar estado de la cita**:

1. **Credential**: crea o elige tu credencial de Google Sheets.
2. **Spreadsheet**: elige la hoja del paso 2.
3. **Sheet**: elige la pestaña concreta (por ejemplo `Hoja 1`).
4. En *Guardar cita* deja **Mapping Column Mode** en `Map Automatically`.
5. En *Actualizar estado de la cita* deja **Column to match on** = `ID`.

### 4.2 Telegram — los tres nodos

En **Aviso nueva cita**, **Resumen de mañana** y **Aviso cambio de cita**:

1. **Credential**: crea una credencial *Telegram API* con el token de tu bot.
2. **Chat ID**: elige el chat **interno del equipo** (no el del paciente). Si no aparece, escribe primero algo al bot y recarga la lista.

### 4.3 Email — nodo *Email recordatorio*

1. **Credential**: crea una credencial *SMTP* con los datos de tu proveedor.
   - Gmail: servidor `smtp.gmail.com`, puerto `465` (SSL) y una **contraseña de aplicación** (no tu contraseña normal). Requiere verificación en dos pasos.
   - Brevo, Mailgun, etc.: usa el host, puerto y usuario que indique el proveedor.
2. **From Email**: la dirección desde la que quieres enviar (por ejemplo `citas@tuclinica.es`).
3. No toques *To Email*, *Subject* ni *Text*: usan variables del flujo.

> El nodo está configurado con **Continue on Fail**: si el email falla (SMTP caído, dirección inválida), el resumen al equipo se envía igualmente y el error queda en *Executions*.

### 4.4 URL pública y nombre del centro

Abre el nodo **Recordatorios a pacientes** y cambia las dos primeras líneas:

```js
const BASE_URL = 'https://TU-N8N.ejemplo.com'; // tu URL pública, sin barra final
const CENTRO = 'tu centro';                    // nombre que verá el paciente
```

- `BASE_URL` debe ser la URL con la que tus pacientes llegan a n8n (por ejemplo `https://n8n.tuclinica.es`). El flujo añade `/webhook/flowkit-cita`.
- Si usas un túnel de pruebas, cámbiala también mientras dure la prueba.
- `CENTRO`: por ejemplo `Clínica Dental Sonrisa`.

### 4.5 Zona horaria y hora del recordatorio

- En **Settings → Timezone** del flujo (o en la configuración general de n8n) elige `Europe/Madrid`. El flujo calcula "mañana" con la hora del servidor.
- El recordatorio sale **cada día a las 18:00**. Para cambiarlo, abre el nodo **Cada día a las 18:00** y ajusta la hora.

---

## 5. Prueba antes de activar

1. **Alta**: pulsa *Execute Workflow*, abre el formulario, crea una cita de prueba **para mañana** con **tu propio email** y envíala. Comprueba que:
   - aparece una fila nueva en la hoja con estado `Pendiente`;
   - llega el aviso a Telegram.
2. **Recordatorio**: pulsa *Execute Workflow* otra vez (o ejecuta solo la rama B). Deberías recibir:
   - un email con la fecha, la hora y dos enlaces;
   - el resumen de la agenda de mañana en Telegram.
3. **Enlaces**: abre el email y pulsa **Confirmar**. Debe abrirse una página verde y la hoja debe pasar a `Confirmada`. Repite con **Cancelar** en otra cita de prueba: la hoja pasa a `Cancelada` y Telegram avisa del **hueco libre**.
4. **Enlace roto**: abre `https://TU-N8N/webhook/flowkit-cita?id=NOEXISTE&accion=confirmar`. Debe mostrarse la página "Enlace no válido" (no un error técnico).
5. Borra las filas de prueba.
6. Activa el flujo con el interruptor **Active**.

> **Importante:** los enlaces usan la URL de **producción** (`/webhook/`). Mientras el flujo está inactivo, esa URL no responde. Para probar los clics antes de activar, activa el flujo, haz la prueba y vuelve a desactivarlo si lo necesitas; o usa la URL de test (`/webhook-test/flowkit-cita`) cambiando `BASE_URL` temporalmente.

> Truco para probar el recordatorio sin esperar a mañana: pon en la hoja una cita con la fecha de mañana y ejecuta el flujo manualmente. Si no hay citas para mañana, el nodo *Resumen para el equipo* devuelve **0 items** y no se envía nada: es el comportamiento correcto.

---

## 6. Qué hace exactamente

### Alta de cita

El formulario pide nombre, teléfono, email, tratamiento o motivo, fecha, hora y notas. El flujo:

- genera un `ID` único (`CITA-AAAAMMDDhhmmss`);
- normaliza los campos a las columnas de la hoja;
- guarda la cita con estado `Pendiente`;
- avisa al equipo por Telegram.

### Recordatorio diario (18:00)

- Lee todas las filas de la hoja.
- Selecciona las citas de **mañana** cuyo estado **no** sea `Cancelada` ni `Atendida`.
- Envía **un email por paciente** con email rellenado. El email solo contiene fecha y hora (sin motivo clínico) y dos enlaces:
  - **Confirmar** → estado `Confirmada`.
  - **Cancelar** → estado `Cancelada`.
- Envía **un solo mensaje** al equipo con la agenda de mañana ordenada por hora. Las citas sin email aparecen marcadas `SIN EMAIL` para que el equipo las llame.
- Si no hay citas para mañana, **no envía nada**.

### Confirmar o cancelar

- El enlace abre una página web con el resultado (verde al confirmar, roja al cancelar) y actualiza la hoja por `ID`.
- El equipo recibe un aviso por Telegram en ambos casos.
- En una cancelación, el aviso incluye **"Hueco libre"** para que recepción pueda ofrecerlo a otro paciente.

---

## 7. Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| No llega el email | SMTP mal configurado, credencial caducada o dirección inválida | Revisa *Executions* → nodo *Email recordatorio*. El resumen al equipo sí llega porque el nodo continúa en caso de fallo |
| El email llega pero los enlaces dan 404 | `BASE_URL` incorrecta, flujo inactivo o n8n sin URL pública | Corrige la constante, activa el flujo y comprueba que `/webhook/flowkit-cita` responde |
| El enlace abre JSON o error técnico | Falta el nodo *Respond to Webhook* o se borró su conexión | Vuelve a importar el flujo o revisa las conexiones de la parte C |
| No se actualiza la hoja al pulsar | La columna `ID` no existe, está vacía o tiene valores repetidos | Revisa el paso 2; cada cita necesita un `ID` único |
| No se envía el resumen | No hay citas para mañana, o todas están canceladas/atendidas | Es lo esperado; crea una cita de prueba con fecha de mañana |
| El recordatorio sale a una hora rara | Zona horaria del servidor distinta de la clínica | Configura `Europe/Madrid` en Settings → Timezone |
| Se envía dos veces el recordatorio | Ejecutaste el flujo manualmente y además se ejecutó el cron | Normal: el cron solo dispara una vez al día a las 18:00 |
| Llega `{{ ... }}` literal | Se borró el `=` inicial de un campo de texto | Vuelve a poner `=` al principio |
| El email cae en spam | El dominio de envío no está autenticado | Usa un proveedor con SPF/DKIM (Brevo, Mailgun, Google Workspace) y una dirección del dominio de la clínica |
| Un enlace antiguo devuelve error técnico | La fila de esa cita se borró de la hoja | No borres filas mientras el enlace pueda estar vivo; archiva las citas antiguas en otra pestaña |

---

## 8. Legal y RGPD (leer una vez)

Este flujo trata **datos de salud** (categoría especial, **art. 9 RGPD**): el hecho de tener una cita en una clínica ya revela información de salud, aunque el email no mencione el tratamiento.

Antes de usarlo en producción:

- **Base legal**: consentimiento explícito e informado del paciente para los recordatorios, o asistencia sanitaria prestada por un profesional sujeto al secreto profesional (art. 9.2.h). Consúltalo con tu asesoría; no es asesoramiento jurídico.
- **Información**: la política de privacidad y el formulario deben explicar que se envían recordatorios por email, quién los envía y cómo ejercer los derechos.
- **Minimización**: el flujo **no incluye el motivo de la cita en el email**; el detalle queda en la hoja interna. No añadas diagnósticos ni información clínica a los mensajes.
- **Seguridad**: limita el acceso a la hoja de cálculo y al chat de Telegram al personal autorizado; usa cuentas con verificación en dos pasos; no compartas el bot.
- **Enlaces**: el `ID` de la cita (con sufijo aleatorio) funciona como llave del enlace de confirmar/cancelar. No lo publiques, no lo reutilices y no lo incluyas en listados abiertos. Si un enlace se filtra, cambia el `ID` de esa fila para invalidarlo.
- **Encargados de tratamiento**: Google (Sheets), tu proveedor SMTP y el proveedor de n8n deben tener contrato de encargado (DPA) y, si aplica, garantías para transferencias internacionales.
- **Retención**: define cuánto tiempo conservas las citas y bórralas cuando dejen de ser necesarias (por ejemplo, al cerrar el episodio asistencial y cumplir los plazos legales).
- **Evaluación de impacto (EIPD/DPIA)**: si la clínica trata datos de salud a gran escala, revisa si necesitas una evaluación de impacto. Este flujo no la sustituye.
- **No decisiones automatizadas**: el flujo no diagnostica, no prioriza pacientes ni toma decisiones clínicas. Solo envía recordatorios administrativos.

---

## 9. Mantenimiento

- Revisa una vez al mes que el flujo sigue **Active**, que la hoja recibe filas y que los emails salen (Executions).
- Si cambias de proveedor SMTP, actualiza solo el nodo *Email recordatorio*.
- Si cambias la URL pública de n8n, actualiza `BASE_URL` en *Recordatorios a pacientes* y vuelve a probar un enlace.
- Si añades columnas a la hoja, no rompes nada: el flujo escribe por nombre de columna y las nuevas quedan vacías.
- Si quieres recordar a **48 h** en lugar de 24 h, cambia `const manana = new Date(hoy.getTime() + 86400000);` por `+ 2 * 86400000` en los dos nodos de código de la parte B.
