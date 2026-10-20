# Onboarding documental — checklist, recordatorios y portal del cliente

Guía del flujo **premium** `workflows/asesoria/onboarding-documental.json` (51 nodos + 7 notas, n8n 2.43.8).
Forma parte del **FlowKit — Pack Asesoría**.

---

## Qué hace

| Parte | Cuándo | Qué hace |
|---|---|---|
| **A · Alta de cliente** | Al enviar el formulario | Comprueba que el email no esté ya en la hoja, genera la **checklist según el servicio** (fiscal, laboral, constitución, contable o general), crea una fila por documento en `Docs` y una fila por cliente en `Onboarding`, escribe al cliente con la checklist y el enlace privado, y avisa al equipo con el ID de cada documento. |
| **B · Seguimiento** | Cada día, 09:00 | Agrupa los documentos pendientes por cliente y envía **un solo correo por cliente** a los 3, 7, 14 y 21 días. Después anota el nivel y la fecha del aviso. |
| **C · Portal del cliente** | Cuando el cliente abre su enlace | Página HTML con el progreso: qué está recibido, qué falta y qué es obligatorio. Solo lectura: no permite subir ni cambiar nada. |
| **D · Registrar recepción** | Cuando el equipo envía el formulario | Marca los documentos indicados como `Recibido`, recalcula el porcentaje, actualiza la hoja del cliente y, si ya está todo, avisa al equipo y envía el correo de alta completada. |
| **E · Informe semanal** | Cada lunes, 09:00 | Expedientes abiertos, completados, tasa de finalización, antigüedad media, **bloqueados a +21 días** y los tres documentos que más se atascan. |
| **F · Red de seguridad** | Si algo falla | Error Trigger → Telegram con el nodo y el mensaje exactos, tras 3 reintentos. |

### Checklist por servicio (editable en el nodo *Validar y preparar checklist*)

| Servicio | Documentos | Obligatorios |
|---|---|---|
| **Fiscal y contable** | DNI/NIE, escritura, CIF y 036/037, certificado bancario, 4 liquidaciones, modelo 100, libros del año anterior | 4 de 7 |
| **Laboral (nóminas)** | CIF de la empresa, convenio, TC1/TC2, contratos, calendario laboral | 3 de 5 |
| **Constitución de sociedad** | DNI de socios, denominación social, estatutos, escritura, CIF provisional, apertura de cuenta | 5 de 6 |
| **Solo contabilidad** | CIF, poderes, extractos, facturas del ejercicio, inventario | 4 de 5 |
| **Otro / a medida** | Identidad, poderes, certificado bancario, documentación específica | 3 de 4 |

### Escalado de los recordatorios (editable en el nodo *Clasificar pendientes*)

| Nivel | Día | Tono |
|---|---|---|
| 1 | 3 | Recordatorio amable con la lista de lo que falta. |
| 2 | 7 | Firme: «sin esto no podemos avanzar con tu expediente». |
| 3 | 14 | Tono serio, con los días transcurridos. |
| 4 | 21 | Último aviso. El equipo lo ve marcado con ⚠️ para llamar. |

**Reglas anti-ruido (importantes):**
- Cada nivel se envía **una sola vez por cliente** (`Recordatorios` solo sube).
- Nunca dos avisos el mismo día (`Ultimo_recordatorio`).
- Si el email de un cliente falla, **no** se marca nada: se reintenta al día siguiente y el resto del lote sigue.
- Un cliente sin email no recibe avisos; aparece en el informe del lunes.

---

## Requisitos

- n8n **1.40+** (probado en 2.43.8).
- Google Sheets con **dos pestañas**: `Docs` y `Onboarding`.
- Cuenta SMTP (Gmail con contraseña de aplicación, Brevo, Mailgun, el SMTP del despacho…).
- Bot de Telegram + chat del equipo.
- URL pública de n8n para que el enlace del cliente y los formularios funcionen (dominio propio o túnel).

---

## Las hojas

### `Docs` — una fila por documento solicitado

| Columna | Ejemplo | Notas |
|---|---|---|
| `ID` | `k7m2p-DNI` | **La genera el flujo.** Único: token del cliente + código del documento. Es la llave para registrar la entrega. |
| `Cliente` | `Panadería Ruiz SL` | Nombre que verá el cliente. |
| `Email` | `ana@panaderiaruiz.es` | Identifica al cliente en el formulario de recepción. |
| `Servicio` | `fiscal` | Código interno: `fiscal`, `laboral`, `constitucion`, `contable`, `general`. |
| `Documento` | `DNI/NIE del titular` | Texto que ve el cliente. |
| `Codigo` | `DNI` | Atajo para registrar la entrega sin copiar el ID. |
| `Obligatorio` | `SI` | `SI` o `NO`. Solo informativo. |
| `Estado` | `Pendiente` | `Pendiente` o `Recibido`. |
| `Token` | `k7m2p9x1ab` | **La genera el flujo.** Es la clave del enlace privado. |
| `Fecha_solicitud` | `2026-10-20` | **La genera el flujo.** Base del escalado. |
| `Fecha_recibido` | `` | **La rellena el flujo** al registrar la entrega. |
| `Recordatorios` | `0` | **La rellena el flujo.** |
| `Ultimo_recordatorio` | `2026-10-23` | **La rellena el flujo.** |
| `Nota` | `` | Libre (la escribe el equipo desde el formulario). |

### `Onboarding` — una fila por cliente

| Columna | Ejemplo | Notas |
|---|---|---|
| `Cliente` | `Panadería Ruiz SL` | |
| `Email` | `ana@panaderiaruiz.es` | **Columna de coincidencia** al actualizar el progreso. |
| `Telefono` | `600 111 222` | Opcional. |
| `Servicio` | `fiscal` | Código interno. |
| `Estado` | `En curso` | `En curso` o `Completo`. |
| `Docs_total` | `7` | **La rellena el flujo.** |
| `Docs_recibidos` | `0` | **La rellena el flujo.** |
| `Pct` | `0%` | **La rellena el flujo.** |
| `Fecha_alta` | `2026-10-20` | **La genera el flujo.** |
| `Fecha_completo` | `` | **La rellena el flujo** al llegar al 100 %. |
| `Token` | `k7m2p9x1ab` | **La genera el flujo.** |
| `Portal` | `https://n8n…/webhook/flowkit-docs?k=…` | Enlace privado del cliente. |
| `Notas` | `` | Libre. |

> Si quieres **reabrir** un expediente, pon `Estado = En curso`, borra `Fecha_completo` y vuelve a poner el documento afectado en `Pendiente`.

---

## Puesta en marcha (30 minutos)

1. **Importa** el `.json` en n8n (Workflows → Import from file).
2. **Abre `Validar y preparar checklist`** y cambia las dos constantes:
   ```js
   const BASE_URL = 'https://n8n.tudespacho.es';   // sin barra final
   const EMPRESA  = 'Asesoría Ejemplo SL';
   ```
   Haz lo mismo en **`Clasificar pendientes`** (los recordatorios salen con esa firma).
3. **Credenciales** (cada nodo te la pide al abrirlo):
   - Los **11 nodos de Google Sheets** → misma cuenta, y en cada uno el documento + pestaña (`Docs` u `Onboarding`).
   - Los **3 nodos de email** → tu SMTP. Rellena `fromEmail` con la dirección desde la que quieres enviar.
   - Los **7 nodos de Telegram** → tu bot y el chat del equipo.
4. **Activa el flujo** y copia las URLs de producción de los dos formularios y del portal:
   - `https://TU-N8N/form-flowkit-onboarding-alta` (alta)
   - `https://TU-N8N/form-flowkit-onboarding-docs` (recepciones)
   - `https://TU-N8N/webhook/flowkit-docs?k=TOKEN` (portal, lo genera el flujo)
   > Las URLs de los formularios las muestra el propio nodo al abrirlo: usa siempre la de **Production**.
5. **Prueba con un cliente tuyo**: envía el formulario de alta con tu email. Debes recibir la checklist, ver 7 filas en `Docs` y 1 en `Onboarding`, y el aviso en Telegram.
6. **Prueba el portal** con el enlace del correo: debe mostrar 0 de 7 y la lista de documentos.
7. **Prueba una entrega**: abre el formulario de recepción, pon tu email y el `ID` de dos documentos. Deben pasar a `Recibido`, el porcentaje actualizarse y llegar el aviso.
8. **Prueba el escalado sin esperar 3 días**: en `Docs`, pon `Fecha_solicitud` de hace 8 días y `Recordatorios = 0`; ejecuta el flujo a mano. Debe salir el email de nivel 2 y quedar `Recordatorios = 2`.

---

## Cómo probarlo sin molestar a clientes

- Usa una copia de la hoja (`Docs_pruebas` / `Onboarding_pruebas`) mientras haces las pruebas y cambia el documento en los 11 nodos de Sheets.
- Pon tu propio email en las altas de prueba.
- El informe del lunes y el seguimiento diario se pueden lanzar a mano (*Execute workflow* ejecuta todas las ramas con disparador).

---

## Ajustes habituales

- **Otro servicio o checklist distinta:** añade una entrada al objeto `CHECKLIST` con pares `['CODIGO', 'Nombre del documento', 'SI'|'NO']` y su opción en el desplegable del formulario de alta (`SERVICIOS`).
- **Otros días de aviso:** cambia `const DIAS = [3, 7, 14, 21]`. El nivel se calcula solo a partir de esos días.
- **Otro tono o firma:** el cuerpo del correo se construye en `Clasificar pendientes`; el asunto también.
- **Sin portal del cliente:** borra la Parte C y el enlace `BASE_URL + PORTAL_PATH` de los correos. El resto sigue funcionando.
- **Registrar varios documentos de golpe:** en el formulario de recepción, un ID por línea (o el código: `DNI`, `CIF`…). También admite separarlos por comas o espacios.
- **Portal con tu marca:** el HTML está en `Validar y renderizar portal`; cambia colores en el bloque `<style>` y el texto del pie.

---

## RGPD y límites legales

- **Datos tratados:** identificativos y de contacto del cliente y **documentación que puede contener datos sensibles** (por ejemplo, un DNI o una nómina). Base legal habitual: ejecución del contrato y obligaciones legales del despacho.
- **Los archivos no pasan por n8n:** este flujo solo registra *qué* documento se ha recibido y *cuándo*. Los archivos llegan por correo y se archivan donde ya archiváis (Drive, gestor documental, etc.). Eso reduce el riesgo: no hay una copia extra en la base de datos de n8n.
- **Enlaces privados:** el portal usa un token aleatorio de 10 caracteres y es **solo lectura**. No permite subir, borrar ni cambiar nada; si se reenvía por error, no se puede manipular el expediente.
- **Plazos de conservación:** aplica tu política habitual de conservación documental. El flujo no borra nada por su cuenta.
- **Información al cliente:** incluye en tu política de privacidad que usas sus datos y su documentación para gestionar el servicio contratado y que puedes enviarle recordatorios por correo.
- **No automatices decisiones sensibles:** el flujo persigue documentación, no evalúa solvencia ni adopta decisiones automatizadas sobre personas.

---

## Problemas frecuentes

| Síntoma | Causa habitual | Solución |
|---|---|---|
| El alta no se crea | Email mal escrito o ya existe un onboarding con ese email | Revisa el aviso de Telegram: dice el motivo exacto. |
| «No se encontró la hoja» | Documento o pestaña sin seleccionar en algún nodo | Abre los 11 nodos de Sheets y elige documento + pestaña. |
| No salen recordatorios | Menos de 3 días desde el alta, nivel ya enviado o sin email | Revisa `Fecha_solicitud`, `Recordatorios` y `Ultimo_recordatorio`. |
| El correo no llega | SMTP sin credenciales o `fromEmail` vacío | Prueba el nodo *Email de recordatorio* con «Execute step». |
| El portal da 400 | Token incompleto o fila borrada | Usa el enlace original del correo; no borres filas con enlaces vivos. |
| La recepción no encuentra al cliente | Email distinto al de la hoja | Copia el email de la hoja `Onboarding`. |
| «Ninguno de los documentos está pendiente» | Ya estaban marcados como `Recibido` | El aviso de Telegram lista los que ya constaban. |
| El porcentaje no cuadra | Filas duplicadas del mismo cliente | Un cliente = un token. Borra duplicados y vuelve a registrar. |
| Llega aviso de error | Mira el texto de Telegram: nodo + mensaje | Corrige la credencial o el dato y vuelve a ejecutar. |

---

## Ficha comercial (para la página del pack)

- **Para quién:** asesorías, gestorías y despachos contables que dan de alta clientes nuevos cada mes.
- **Problema:** el alta se eterniza. Se piden documentos de palabra, se pierden correos, nadie sabe qué falta y el trabajo empieza tarde. Cada expediente parado son semanas de facturación que no arrancan.
- **Resultado:** checklist automática por servicio, recordatorios en 4 escalones, portal de estado para el cliente y un panel que dice exactamente qué expedientes están bloqueados y por qué documento.
- **Tiempo de puesta en marcha:** 30 minutos.
- **Incluido en:** FlowKit — Pack Asesoría (29 € lanzamiento / 39 €).
