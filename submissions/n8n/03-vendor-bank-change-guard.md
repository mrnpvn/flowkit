# Envío 3 — n8n Creator Hub

> **⛔ NO subir este archivo (.md).** Este documento son solo las instrucciones.
> **El archivo que se sube es el JSON:** [`workflows/finance/vendor-bank-change-guard.json`](../../workflows/finance/vendor-bank-change-guard.json)
> URL directa: https://raw.githubusercontent.com/mrnpvn/flowkit/main/workflows/finance/vendor-bank-change-guard.json

**Plantilla:** Verify vendor bank detail changes with Google Sheets, email and Telegram
**Archivo para enviar:** [`workflows/finance/vendor-bank-change-guard.json`](../../workflows/finance/vendor-bank-change-guard.json)
**Estado:** listo para envío (import/export limpio en n8n 2.43.8 + 45 pruebas de lógica).
**Sustituye a:** envíos 1 y 2, rechazados por «too basic». Esta vez no es una ruta: es un mecanismo.

## Qué hace

Un cambio de datos bancarios de un proveedor es la petición más valiosa que puede enviar un atacante a un equipo de finanzas (fraude del CEO / BEC). Este workflow hace que **no se pueda aplicar por accidente**:

1. **Alta (formulario → retención).** El proveedor envía el cambio por un formulario. El IBAN se normaliza y se valida de verdad: formato, **checksum mod-97** y país del IBAN contra país del banco. El proveedor se cruza con la hoja `Vendors` y la petición se puntúa por riesgo (proveedor desconocido 100, proveedor inactivo +40, país del IBAN distinto +30, dominio del solicitante distinto +25, buzón gratuito +15, segundo cambio en 90 días +15, dos o más +30). **Riesgo ≥ 70 va a revisión manual y no recibe enlace de aprobación**; riesgo ≥ 40 usa el periodo de enfriamiento largo (72 h; 48 h por defecto). Los duplicados se detectan por hash del IBAN y se ignoran sin mandar un segundo correo.
2. **Decisión (webhook).** El enlace de un solo uso va **solo a la dirección de finanzas que ya estaba guardada**, nunca a la que pide el cambio. Aprobar antes de que termine el enfriamiento se rechaza sin tocar el estado. Un segundo clic devuelve «Already handled» y no cambia nada (idempotencia). Tokens inválidos, desconocidos o caducados → HTTP 400 y fila `LINK_DENIED` en `Audit`.
3. **Vigilancia (diario 07:30).** 24 h → recordatorio a finanzas; 48 h → escalado al responsable; 72 h → se cierra como `EXPIRED` y se avisa al solicitante de que no se aplicó nada. `last_stage` garantiza que cada etapa se manda una sola vez. Un único digest diario sustituye al ruido por proveedor.

Integraciones: **Google Sheets + SMTP + Telegram + webhook + formulario**. Sin IA y sin servicios de pago.

## Por qué NO es «too basic» (argumento para revisión)

La lección de los dos rechazos: la librería no mide tamaño, mide **mecanismo**. Un flujo que se describe como «lee una tabla y avisa» es un snippet aunque tenga 21 nodos. Este se describe así:

> *Retiene cada cambio de datos de pago hasta que un aprobador externo supera un periodo de enfriamiento, y rechaza repeticiones y enlaces caducados dejando rastro de auditoría.*

Tiene las tres cosas que la librería pide:

- **Estado que sobrevive entre ejecuciones** (la hoja es la máquina de estados: `PENDING_VERIFICATION` → `APPROVED` / `REJECTED` / `EXPIRED` / `MANUAL_REVIEW` / `DUPLICATE_IGNORED`).
- **Una decisión que cambia el comportamiento** (validez del IBAN, proveedor, duplicado, velocidad de cambios y riesgo).
- **Un camino de error que hace más que parar** (fila en `Audit`, HTTP coherente, el estado no avanza si el correo falla).

Y el caso de uso **no existe en la librería**: búsquedas en el catálogo (21-oct) dan 1 plantilla para *dunning*, 1 para *magic link*, 1 para *approval gate*, 1 para *audit trail* y **ninguna** para verificación de cambios de datos bancarios de proveedores. No es un re-skin de otro flujo.

## Cifras del workflow

- **60 nodos** (54 funcionales + 6 sticky notes), **44 grupos de conexión**.
- **5 integraciones** (Sheets, SMTP, Telegram, webhook, form).
- **4 puntos de entrada** (formulario, webhook de decisión, watchdog diario, Error Trigger).
- **7 nodos Code** con lógica pura (validación IBAN, scoring, token, decisión, watchdog, digest, error).
- **4 nodos Crypto**: generación del token y hash SHA-256 (el token en claro no se guarda nunca).
- 11 nodos de Google Sheets, 6 de email, 6 de Telegram, 8 IF, 5 respuestas HTML.

## Verificación hecha antes de entregar

- Import + export **sin errores en n8n 2.43.8**: 60/60 nodos y 44 grupos de conexión preservados.
- **45 aserciones** sobre los 7 Code: checksum IBAN, país, proveedor desconocido, duplicado, velocidad, token caducado, enfriamiento, replay, token malformado/desconocido, decisión inválida, etapas 24/48/72 h, no repetición de etapa, destinatario del aviso de expiración, conteos del digest.
- Todas las referencias `$('Nodo')` resuelven a un nodo existente (comprobado por script).
- Los 3 nodos Config dejan pasar los campos de entrada (comprobado por script).
- Sin credenciales, sin secretos, sin APIs de pago.

## Envío (3 clics, ~5 minutos)

1. Entrar en https://creators.n8n.io con la cuenta ya creada.
2. **Submit a template** → arrastrar el JSON → marcar **Free** → aceptar los términos.
3. Pulsar **Submit for AI review** y esperar.

Regla de n8n: **una plantilla en revisión a la vez** hasta tener 3 aprobadas. No enviar nada más hasta que respondan a esta.

## Texto de ficha (inglés, para pegar en el portal)

- **Title:** Verify vendor bank detail changes with Google Sheets, email and Telegram
- **Short description:** Hold every vendor bank-detail change until the finance contact already on file clears a cooling period. Validates the IBAN, scores risk, refuses replays and expired links, escalates at 24/48/72 h and keeps an append-only audit trail. No paid services.

### Description for the portal (Markdown, ~200 words, paste as is)

```markdown
## Who's it for
Finance and accounts-payable teams that receive vendor payment-detail changes by email and cannot afford a fraudulent one to be applied.

## How it works
Every request is held in a pending state. The IBAN is validated (format, mod-97 checksum and country match), the vendor is matched against your sheet and the request is scored for risk. A single-use approval link goes only to the finance address already on file, never to the sender. Approval is refused until the cooling period ends (48 h, or 72 h for higher risk); replays, unknown and expired links are denied and logged. A daily watchdog reminds at 24 h, escalates at 48 h and closes the request as expired at 72 h, with one digest instead of per-vendor noise. Every attempt lands in an append-only audit tab.

## How to set up
Create the Vendors, BankChanges and Audit tabs (headers in the Setup note), fill Vendors with the trusted finance address per vendor, set `base_url` and `finance_lead` in the three Config nodes, connect the Google Sheets, SMTP and Telegram credentials, then activate.

## Requirements
Google Sheets, an SMTP account, a Telegram bot and a public n8n URL for the approval links. No paid services.

## How to customize
Change thresholds, cooling periods and token TTL in the Config nodes; add or remove form fields in the trigger.
```

> **Regla de n8n (guía oficial, 21-oct)**: la sticky amarilla debe contener **la descripción completa**, no un resumen. Ya está aplicada en el `.json`.
- **Categories:** Finance & Accounting · Google Sheets · Telegram · Email · Security
- **Level:** Advanced
- **Requirements:** Google account (Sheets) · SMTP account · Telegram bot + chat ID · public n8n URL for the approval links. No paid services.
- **Setup:** create the three tabs (`Vendors`, `BankChanges`, `Audit`) with the headers in the sticky note, fill `Vendors` with the trusted finance address per vendor, set `base_url` and `finance_lead` in the three Config nodes, connect the three credentials, activate.

## Qué necesita del titular

Nada nuevo: cuenta de creators ya creada. Solo el clic final de subida (identidad) cuando el CEO avise. El `.json` se entrega por Telegram como documento.
