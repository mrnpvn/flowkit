# Envío 2 — n8n Creator Hub

> **⚠️ NO subir este archivo (.md).** Este documento son solo las instrucciones.
> **El archivo que se sube es el JSON:** [`workflows/clinica/appointment-reminders-confirm-cancel.json`](../../workflows/clinica/appointment-reminders-confirm-cancel.json)
> URL directa: https://raw.githubusercontent.com/mrnpvn/flowkit/main/workflows/clinica/appointment-reminders-confirm-cancel.json

**Plantilla:** Appointment reminders with confirm and cancel links (clinic)
**Archivo para enviar:** [`workflows/clinica/appointment-reminders-confirm-cancel.json`](../../workflows/clinica/appointment-reminders-confirm-cancel.json)
**Estado:** listo para envío (probado en n8n 2.43.8). Pendiente: los 3 clics del titular.
**Sustituye a:** envío 1 (*Capture web leads*), rechazado el 20-oct por «too basic».

## Qué hace (para la ficha)

Tres procesos en un solo workflow de 21 nodos:

1. **Reservar cita** — formulario alojado en n8n → normaliza 11 campos → añade la fila a la hoja `Appointments` → aviso al equipo por Telegram.
2. **Recordatorio 24 h antes** — cada día a las 18:00 lee la agenda, selecciona las citas de mañana (no canceladas ni atendidas) y envía **un email por paciente** con dos enlaces: confirmar y cancelar. Al equipo le llega **un único resumen**, con las citas sin email marcadas para llamarlas. Si no hay citas, no envía nada.
3. **Enlaces del paciente** — el paciente pulsa confirmar o cancelar → la fila se actualiza por `ID` → el equipo recibe el aviso; si es cancelación, incluye el **hueco liberado** (fecha y hora) para ofrecerlo a otro paciente → el paciente ve una página HTML (200) o una página de enlace inválido (400).

Integraciones: **Google Sheets + Telegram + SMTP**, sin IA y sin servicios de pago.

## Por qué cumple el listón (tras el rechazo del envío 1)

- **21 nodos**, 3 partes independientes y un caso de uso completo, no un tutorial.
- **3 integraciones reales** y 3 disparadores distintos (form, schedule, webhook).
- Manejo de errores: el envío de email no bloquea el resto del lote; el webhook responde HTTP coherente.
- Sin credenciales ni secretos en el JSON; sin datos de salud en el email (privacidad por diseño).
- Nodos, sticky notes, emails, HTML y mensajes **íntegramente en inglés**.

## Envío (3 clics, ~5 minutos)

1. Entrar en https://creators.n8n.io con la cuenta ya creada.
2. **Submit a template** → arrastrar el archivo JSON → marcar **Free** → aceptar los términos.
3. Pulsar **Submit for AI review** y esperar (control automático + revisión humana, <1 semana).

Regla de n8n: **una plantilla en revisión a la vez** hasta tener 3 aprobadas.

## Si el portal pide texto de ficha

- **Título:** Appointment reminders with confirm and cancel links (clinic)
- **Descripción corta:** Book appointments from a form, email every patient a reminder 24 h ahead with one-click confirm/cancel links, keep the Google Sheets status in sync and alert the team on Telegram — including the freed slot when someone cancels.
- **Categorías sugeridas:** Scheduling · Google Sheets · Telegram · Email · Healthcare
- **Nivel:** Intermediate
- **Requisitos:** Google account (Sheets) + Telegram bot + SMTP account + public n8n URL for the links. No paid services.

## Comprobaciones hechas antes del envío (20-oct)

- **Import y export sin errores en n8n real 2.43.8** (instancia local aislada).
- `typeVersion` soportadas: formTrigger 2.2 · set 3.4 · googleSheets 4.5 · telegram 1.2 · scheduleTrigger 1.2 · code 2 · emailSend 2.1 · webhook 2 · if 2.2 · respondToWebhook 1.1 · stickyNote 1.
- JSON válido, nombres de nodo únicos, todas las conexiones apuntan a nodos existentes y todas las ramas son alcanzables.
- **Lógica de los tres nodos Code probada en Node con 7 filas simuladas**: ISO, `dd/mm/yyyy`, ISO de otro día, cancelada, atendida, sin email y fecha inválida → 2 emails correctos (los 2 de mañana con email), resumen con 3 citas y marca `NO EMAIL`, 0 mensajes cuando no hay citas.
- Enlaces probados: confirmar → `Confirmed`; cancelar → `Cancelled` + hueco `dd/mm/yyyy at hh:mm`; parámetros inválidos → HTTP 400 y página gris.
- Sin texto en español y sin caracteres corruptos (comprobado por script).
- `Set` con `includeOtherFields: false` → la hoja queda exactamente en 11 columnas.

## Después del envío

- Anotar la fecha en STATE.md y avisar por Telegram.
- Si n8n pide cambios, se corrigen aquí y se reenvía (no se reenvía igual).
- Cuando esté aprobada: enlazarla desde README, CATALOGO y el hub; añadir el `?ref=` de afiliado cuando la cuenta esté aprobada.
