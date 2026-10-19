# Contribuir a FlowKit

Gracias por querer aportar. Lo más valioso que puedes enviar es **un flujo que ya usas de verdad** en tu negocio.

## Qué buscamos

- Flujos que resuelvan una tarea repetitiva concreta de una pyme o despacho.
- Que funcionen con herramientas comunes (Google Sheets, Notion, Telegram, email, Stripe, CRMs con API oficial).
- Documentados en español.

## Qué no aceptamos

- Flujos que hagan *scraping* de plataformas que lo prohíben en sus términos, o que se salten medidas técnicas.
- Flujos que traten datos personales sin una finalidad clara y sin aviso en su ficha (especialmente **datos de salud**).
- Flujos de captación masiva no solicitada (spam).

## Cómo enviar un flujo

1. En tu n8n, exporta el flujo: menú **⋯ → Download**.
2. Crea la carpeta de la vertical si no existe: `workflows/<vertical>/`.
3. Nombra el archivo en minúsculas con guiones: `recordatorio-citas.json`.
4. Añade su ficha al final del apartado correspondiente en `docs/CATALOGO.md` con: qué hace, qué necesita, qué datos personales trata y tiempo de puesta en marcha.
5. Abre un *pull request* explicando en dos líneas para quién lo construiste y qué problema resolvía.

## Requisitos del `.json`

- Nombres de nodos **descriptivos** (no `Set1`, `HTTP Request2`).
- Al menos una **nota (sticky note)** al principio del flujo explicando el propósito y qué hay que configurar.
- **Sin credenciales ni datos reales** dentro del archivo: borra tokens, IDs de hoja, emails y nombres de clientes antes de enviar.
- Nada de datos de prueba de personas reales.

## Revisión

Se comprueba que el flujo importe sin errores, que la documentación coincida con lo que hace y que no haya nada prohibido arriba. Si algo no encaja, se comenta en el mismo *pull request*.
