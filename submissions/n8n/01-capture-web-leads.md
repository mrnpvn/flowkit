# Envío 1 — n8n Creator Hub

**Plantilla:** Capture web leads to Google Sheets and alert on Telegram
**Archivo para enviar:** [`workflows/general/capture-web-leads-google-sheets-telegram.json`](../../workflows/general/capture-web-leads-google-sheets-telegram.json)
**Estado:** listo para envío (probado en n8n 2.43.8). Pendiente: 3 clics del titular.

## Qué hace

Formulario web alojado en el propio n8n → normaliza nombre, email, teléfono, mensaje y fecha → añade una fila a Google Sheets → avisa por Telegram. Sin servicios de pago y sin credenciales incluidas: cada usuario conecta las suyas.

## Por qué esta plantilla y no otra

- Es la más buscada y la más reutilizable (sirve a cualquier vertical).
- Tiene 5 nodos y se entiende en 30 segundos → buena puerta de entrada al perfil de creador.
- No trata datos de salud ni financieros: menos fricción en la revisión.

## Envío (3 clics, ~5 minutos)

1. Entrar en https://creators.n8n.io con la cuenta ya creada.
2. **Submit a template** → arrastrar el archivo JSON → marcar **Free** → aceptar los términos.
3. Pulsar **Submit for AI review** y esperar (primero pasa un control automático y después la revisión humana de n8n; suelen tardar menos de una semana).

Regla de n8n: **una plantilla en revisión a la vez** hasta tener 3 aprobadas; después se desbloquea el perfil de creador verificado.

## Si el portal pide texto de ficha

- **Título:** Capture web leads to Google Sheets and alert on Telegram
- **Descripción corta:** Turn a hosted web form into a lead pipeline: normalize the submission, append it to Google Sheets and get a Telegram alert in seconds.
- **Categorías sugeridas:** Lead generation · Google Sheets · Telegram · Forms
- **Nivel:** Beginner
- **Requisitos:** Google account (Sheets) + Telegram bot. No paid services.

## Comprobaciones hechas antes del envío

- Importado y reexportado sin errores en n8n **2.43.8** (instancia local aislada, `npx n8n`).
- `typeVersion` de cada nodo soportada por la versión actual: formTrigger 2.2 ✔ · set 3.4 ✔ · googleSheets 4.5 ✔ · telegram 1.2 ✔ · stickyNote 1 ✔.
- JSON válido, conexiones completas, sin bloques `credentials` ni secretos.
- Nombres de nodo en inglés y notas de configuración dentro del propio workflow.
- Datos esperados en la hoja: `name`, `email`, `phone`, `message`, `received_at`.

## Después del envío

- Anotar la fecha en STATE.md y avisar por Telegram.
- Si n8n pide cambios, se corrigen aquí y se reenvía.
- Cuando esté aprobada: enlazarla desde el README y el hub (y añadir el `?ref=` de afiliado cuando la cuenta esté aprobada).
