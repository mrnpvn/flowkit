# Cobros vencidos — recordatorios escalonados y conciliación

Guía del flujo **premium** `workflows/asesoria/cobros-vencidos.json` (29 nodos, n8n 2.43.8).
Forma parte del **FlowKit — Pack Asesoría**.

---

## Qué hace

| Parte | Cuándo | Qué hace |
|---|---|---|
| **A · Recordatorios** | Cada día, 09:00 | Lee la hoja `Facturas`, agrupa por cliente y envía **un solo correo por cliente** con sus facturas vencidas del tramo que toque. Después anota en la hoja el nivel y la fecha del recordatorio. |
| **B · Informe de cartera** | Cada lunes, 09:00 | Resumen para el despacho: total pendiente, tramos de antigüedad (al día, 1-30, 31-60, 61-90, +90), cinco mayores deudores y avisos de datos incompletos. |
| **C · Enlace de pago** | Cuando el cliente hace clic | Marca las facturas como `Pagada` sin login, avisa al equipo por Telegram y responde una página HTML (200 válido / 400 no válido). |
| **D · Red de seguridad** | Si algo falla | Error Trigger → Telegram con el nodo y el mensaje exactos, tras 3 reintentos. |

### Escalado (editable en el nodo *Clasificar cartera*)

| Nivel | Días desde el vencimiento | Tono |
|---|---|---|
| 1 | 5 | Aviso amable, «si ya lo has pagado, ignora este mensaje». |
| 2 | 15 | Firme: pide fecha prevista de pago o transferencia. |
| 3 | 30 | Aviso final y oferta de aplazamiento; el equipo lo ve en el resumen. |

**Reglas anti-ruido (importantes):**
- Cada nivel se envía **una sola vez por factura** (`Nivel` solo sube).
- Nunca dos recordatorios el mismo día (`Ultimo_Recordatorio`).
- Solo se escribe a facturas en estado `Pendiente` con email válido.
- Un fallo de SMTP en un cliente **no** marca su fila: se reintenta al día siguiente y el resto del lote sigue.

---

## Requisitos

- n8n **1.40+** (probado en 2.43.8).
- Google Sheets con una pestaña `Facturas`.
- Cuenta SMTP (Gmail con contraseña de aplicación, Brevo, Mailgun, el SMTP del despacho…).
- Bot de Telegram + chat del equipo.
- URL pública de n8n para que los enlaces del correo funcionen (dominio propio o túnel).

---

## La hoja `Facturas`

Crea la pestaña con **estas cabeceras exactas** (fila 1):

| Columna | Ejemplo | Notas |
|---|---|---|
| `ID` | `F-2026-0142-8kd3` | Único. Añade un sufijo aleatorio: el ID es la llave del enlace de pago. |
| `Cliente` | `Clínica Sol SL` | Nombre que verá el cliente en el correo. |
| `Email` | `admin@clinicasol.es` | Sin email no se envía recordatorio (aparece en el informe del lunes). |
| `Concepto` | `Nóminas septiembre` | Texto libre. |
| `Importe` | `1.234,56` | Formato español o `1234.56`. Sin símbolo de moneda. |
| `Fecha_Emision` | `01/09/2026` | Informativa. |
| `Fecha_Vencimiento` | `30/09/2026` | Acepta `dd/mm/aaaa` y `aaaa-mm-dd`. |
| `Estado` | `Pendiente` | `Pendiente`, `Pagada` o `Incobrable`. |
| `Nivel` | `0` | **La rellena el flujo.** No la toques salvo para reiniciar un escalado. |
| `Recordatorios` | `0` | **La rellena el flujo.** |
| `Ultimo_Recordatorio` | `2026-10-06` | **La rellena el flujo.** |
| `Fecha_Pago` | `` | **La rellena el flujo** cuando el cliente confirma. |
| `Notas` | `` | Libre (no la usa el flujo). |

> Si quieres reiniciar los avisos de una factura, pon `Nivel` y `Recordatorios` a `0` y borra `Ultimo_Recordatorio`.

---

## Puesta en marcha (30 minutos)

1. **Importa** el `.json` en n8n (Workflows → Import from file).
2. **Abre el nodo `Clasificar cartera`** y edita las cuatro constantes de arriba:
   ```js
   const BASE_URL = 'https://n8n.tudespacho.es';   // sin barra final
   const EMPRESA  = 'Asesoría Ejemplo SL';
   const IBAN     = 'ES00 0000 0000 0000 0000 0000';
   const DIAS_NIVEL1 = 5, DIAS_NIVEL2 = 15, DIAS_NIVEL3 = 30;
   ```
3. **Credenciales** (cada nodo te la pide al abrirlo):
   - `Leer facturas`, `Leer facturas (informe)`, `Leer facturas (pago)` y los dos nodos de actualización → la misma cuenta de Google Sheets y el mismo documento/pestaña.
   - `Email de recordatorio` → tu SMTP. Rellena `fromEmail` con la dirección desde la que quieres enviar.
   - Los cuatro nodos de Telegram → tu bot y el chat del equipo.
4. **Activa el flujo** y copia la URL de producción del webhook (`Webhook pago`): debe ser `https://TU-N8N/webhook/flowkit-cobro`.
5. **Prueba con una factura tuya**: crea una fila con tu email, `Estado = Pendiente` y una `Fecha_Vencimiento` de hace 6 días. Ejecuta el flujo a mano (*Execute workflow*) y comprueba el correo, la hoja y el aviso de Telegram.
6. **Comprueba el enlace**: abre el enlace del correo, confirma que la fila pasa a `Pagada`, que llega el aviso y que la página responde en verde. Prueba también un enlace manipulado: debe dar página roja y 400.

---

## Cómo probarlo sin molestar a clientes

- Usa una copia de la hoja (`Facturas_pruebas`) mientras haces las pruebas y cambia el documento en los cinco nodos de Sheets.
- O pon tu propio email en las filas de prueba y `Estado = Pendiente`.
- El informe del lunes se puede lanzar a mano sin esperar al calendario (*Execute workflow* ejecuta todas las ramas con disparador).

---

## Ajustes habituales

- **Otro calendario de avisos:** cambia `DIAS_NIVEL1/2/3` y, si quieres, los textos de la función `parrafo()`.
- **Otro tono o firma:** el cuerpo del correo se construye en el mismo nodo; el asunto también.
- **Sin enlace de pago:** borra el bloque del enlace en `Clasificar cartera` y la Parte C completa.
- **Más de un recordatorio en el mismo nivel:** hoy no se repite por diseño; si lo necesitas, cambia la condición `nivel <= nivelActual` por una regla de días desde `Ultimo_Recordatorio`.
- **Moneda distinta del euro:** cambia `currency` en las dos funciones `money()`.

---

## RGPD y límites legales

- **Datos tratados:** identificativos y de contacto del cliente (razón social, email) y datos de facturación. Base legal habitual: ejecución del contrato y obligaciones contables/fiscales.
- **Correo B2B de cobro:** es comunicación contractual, no marketing. No incluyas promociones en estos emails.
- **Conservación:** las facturas y su historial de cobro deben conservarse según la normativa contable/fiscal aplicable (habitualmente 4-6 años). No borres filas con enlaces vivos: un enlace de una fila borrada devolverá «no encontramos la factura».
- **Sin amenazas ni presión:** los textos incluidos son informativos y ofrecen aplazamiento. No automatices amenazas de acciones legales, publicación de morosidad ni comunicación a terceros: eso requiere revisión legal y, según el caso, base jurídica específica.
- **Enlaces:** usa IDs con sufijo aleatorio. El parámetro `k` es la clave del enlace; si lo quitas, cualquiera que adivine un ID podría marcar una factura como pagada.
- **Si el impago se cede a un tercero o se reclama judicialmente**, revisa antes con tu asesoría legal el tratamiento de datos y la comunicación al deudor.

---

## Problemas frecuentes

| Síntoma | Causa habitual | Solución |
|---|---|---|
| No se envía nada | Ninguna factura supera el nivel 1, o `Nivel` ya está en ese tramo | Revisa fechas y la columna `Nivel`. |
| «No se encontró la hoja» | Documento o pestaña sin seleccionar en algún nodo | Abre los cinco nodos de Sheets y elige documento + pestaña. |
| El correo no llega | SMTP sin credenciales o `fromEmail` vacío | Prueba el nodo *Email de recordatorio* con «Execute step». |
| El enlace da 400 | `k` no coincide o el ID no existe | Usa el enlace original del correo; comprueba que la fila no se borró. |
| Se actualiza la fila equivocada | `ID` duplicado en la hoja | Los IDs deben ser únicos. |
| Importe mal interpretado | Formato raro (`1,234.56`) | Normaliza la hoja a formato español o `1234.56`. |
| Llega aviso de error | Mira el texto de Telegram: nodo + mensaje | Corrige la credencial o el dato y vuelve a ejecutar. |

---

## Ficha comercial (para la página del pack)

- **Para quién:** asesorías, gestorías y despachos contables con 20-200 clientes y facturación recurrente.
- **Problema:** el seguimiento de impagos vive en la cabeza de una persona; se avisa tarde, se pierden días de caja y se escribe a mano.
- **Resultado:** tres avisos automáticos bien escalonados, conciliación en un clic y un informe semanal que se lee en 20 segundos.
- **Tiempo de puesta en marcha:** 30 minutos.
- **Incluido en:** FlowKit — Pack Asesoría (29 € lanzamiento / 39 €).
