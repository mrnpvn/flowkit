# FlowKit — plantillas de automatización n8n para pymes y despachos

[![Licencia: MIT](https://img.shields.io/badge/licencia-MIT-blue.svg)](LICENSE)
[![Compatible con n8n](https://img.shields.io/badge/n8n-compatible-EA4B71.svg)](https://n8n.io)

**Flujos n8n listos para importar y usar**, pensados para negocios pequeños que no tienen equipo técnico: asesorías y gestorías, clínicas, inmobiliarias, agencias y despachos profesionales.

No son capturas de pantalla ni tutoriales genéricos: cada flujo es un **archivo `.json` que importas en tu n8n en 30 segundos**, con credenciales, plantillas de mensajes y documentación paso a paso en español.

---

## ¿Qué resuelve FlowKit?

Una pyme pierde horas cada semana en tareas repetitivas que ya están resueltas:

- un formulario de contacto **no se guarda en ningún sitio** y se pierde un lead;
- los **recordatorios de cita** se hacen a mano, una llamada por cliente;
- los **plazos fiscales** se persiguen en una hoja de cálculo;
- nadie contesta al cliente hasta el día siguiente (y se va a la competencia).

FlowKit empaqueta ese trabajo en flujos probados: los importas, conectas tus cuentas y funcionan. **Sin código, sin consultora, sin cuota mensual de SaaS.**

## Índice de flujos

| Flujo | Vertical | Estado | Archivo |
|---|---|---|---|
| Captura de leads web (formulario → hoja de cálculo → aviso Telegram) | General | ✅ Listo | [`workflows/general/captura-leads-web.json`](workflows/general/captura-leads-web.json) |
| Asesoría: alta de cliente + recordatorio de plazos fiscales | Asesoría / gestoría | ✅ Listo | [`workflows/asesoria/alta-cliente-plazos.json`](workflows/asesoria/alta-cliente-plazos.json) |
| Clínica: recordatorio de cita + recuperación de cancelaciones | Clínica / dental | ✅ Listo | [`workflows/clinica/recordatorio-citas.json`](workflows/clinica/recordatorio-citas.json) |
| Inmobiliaria: captación de leads + respuesta automática | Inmobiliaria | ✅ Listo | [`workflows/inmobiliaria/captacion-leads-respuesta.json`](workflows/inmobiliaria/captacion-leads-respuesta.json) |
| Clinic: appointment reminders with confirm/cancel links (EN) | Clinic | ✅ Ready | [`workflows/clinica/appointment-reminders-confirm-cancel.json`](workflows/clinica/appointment-reminders-confirm-cancel.json) |
| 💎 Asesoría: cobros vencidos (recordatorios escalonados + informe + enlace de pago) | Asesoría / gestoría | 💎 Premium | [`workflows/asesoria/cobros-vencidos.json`](workflows/asesoria/cobros-vencidos.json) |

Cada flujo incluye su propia ficha en [`docs/CATALOGO.md`](docs/CATALOGO.md): qué hace, qué necesitas configurar, qué datos toca y cuánto tiempo ahorra. Las guías de puesta en marcha están en [`docs/`](docs/): [`INSTALACION-N8N.md`](docs/INSTALACION-N8N.md) (general), [`FLUJO-ASESORIA.md`](docs/FLUJO-ASESORIA.md) (alta de cliente + plazos fiscales), [`FLUJO-CLINICA.md`](docs/FLUJO-CLINICA.md) (recordatorios de cita + cancelaciones) y [`FLUJO-INMOBILIARIA.md`](docs/FLUJO-INMOBILIARIA.md) (captación de leads + respuesta automática). El flujo de clínica existe también en inglés, documentado en [`docs/FLOW-APPOINTMENTS-EN.md`](docs/FLOW-APPOINTMENTS-EN.md). El flujo premium de cobros vencidos se documenta en [`docs/FLUJO-COBROS-VENCIDOS.md`](docs/FLUJO-COBROS-VENCIDOS.md).

## Instalación (2 minutos)

1. Descarga el `.json` del flujo que quieras.
2. En n8n: **Workflows → Import from File** y selecciona el archivo.
3. Abre cada nodo marcado en rojo, elige tus credenciales (Google Sheets, Telegram, etc.) y guarda.

Guía detallada, requisitos y solución de problemas: [`docs/INSTALACION-N8N.md`](docs/INSTALACION-N8N.md).

**Requisitos:** n8n self-hosted o n8n Cloud (versión 1.40 o superior) y las cuentas de las herramientas que use cada flujo (Google Sheets y Telegram en los ejemplos incluidos).

## Casos de uso por vertical

- **Asesorías y gestorías** — alta de cliente sin retipear datos, avisos automáticos de plazos (IVA, IRPF, modelos), registro de documentos recibidos.
- **Clínicas y dentales** — recordatorio de cita 24 h antes, recuperación de huecos por cancelación, confirmación por el propio paciente.
- **Inmobiliarias** — respuesta inmediata al lead de la web, emparejamiento con la cartera, reparto entre comerciales y aviso de leads sin atender.
- **Agencias y despachos** — captura de formularios, seguimiento de presupuestos, informes periódicos al cliente sin exportar CSV a mano.

Si tu caso no está, [abre una petición](https://github.com/mrnpvn/flowkit/issues): los flujos que se piden se construyen.

## Personalización y soporte

- Cada flujo lleva **notas dentro del propio n8n** (nodo *sticky note*) explicando qué hace cada paso.
- Las plantillas de mensajes están en español y se editan sin tocar la lógica.
- ¿Se te atasca algo? Abre un *issue* con el nombre del flujo y el error; se responde en el repositorio.

## Privacidad y RGPD

Estos flujos se ejecutan **en tu propia instancia de n8n**, con tus credenciales. Los datos de tus clientes no pasan por servidores nuestros.

Aun así, tenlo en cuenta al usarlos con datos personales (RGPD, y con especial cuidado con datos de salud en el caso de clínicas):

- conecta solo las cuentas necesarias y con permisos mínimos;
- informa a tus clientes de para qué usas sus datos (recordatorios, gestión) en tu política de privacidad;
- guarda los datos en herramientas con contrato de encargado de tratamiento (Google Workspace, etc.);
- borra los datos cuando dejen de ser necesarios.

## Licencia

El código y la documentación de este repositorio se publican bajo licencia **MIT** (ver [`LICENSE`](LICENSE)): úsalos, modifícalos y úsalos comercialmente en tu negocio, incluida tu actividad con clientes.

Los **packs premium** (versiones ampliadas, flujos sectoriales completos y soporte) se distribuyen aparte y no están cubiertos por la MIT.

## Contribuir

¿Un flujo que funciona en tu negocio? Un *pull request* con el `.json` exportado y una ficha en `docs/CATALOGO.md` es bienvenido. Normas en [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

FlowKit es un producto de **AIVANTA**.
