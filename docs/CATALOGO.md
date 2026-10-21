# Catálogo FlowKit

Ficha de cada flujo: qué hace, qué necesitas, qué datos trata y para quién es.

Estados: **✅ listo** (probado, se puede importar ya) · **🚧 en construcción** (publicado en las próximas entregas) · **💎 premium** (incluido en el pack de pago).

---

## ✅ Captura de leads web

- **Archivo:** `workflows/general/captura-leads-web.json`
- **Vertical:** cualquiera (asesoría, clínica, inmobiliaria, agencia, freelance)
- **Disparador:** formulario web alojado en tu n8n (nodo *Form Trigger*)
- **Qué hace:** el visitante rellena el formulario → los datos se normalizan (nombre, email, teléfono, mensaje, fecha) → se guardan como una fila en tu hoja de cálculo → te llega un aviso a Telegram con el resumen del lead.
- **Qué necesitas:** una hoja de Google Sheets, un bot de Telegram. n8n 1.40+.
- **Datos personales que trata:** nombre, email, teléfono y el mensaje que escriba el interesado. Usa el flujo solo si tienes base legal para tratar esos datos (contacto comercial solicitado por el propio interesado) y enlaza tu política de privacidad junto al formulario.
- **Ahorro estimado:** el lead deja de perderse entre el correo y el WhatsApp; el aviso llega en segundos en lugar de cuando alguien mire el buzón.
- **Tiempo de puesta en marcha:** 10–15 minutos.

---

## ✅ Asesoría — alta de cliente y recordatorio de plazos fiscales

- **Archivo:** `workflows/asesoria/alta-cliente-plazos.json`
- **Guía:** [`FLUJO-ASESORIA.md`](FLUJO-ASESORIA.md)
- **Vertical:** asesorías, gestorías, despachos contables
- **Disparador:** formulario web (alta) + programación diaria a las 8:00 (avisos)
- **Qué hace:**
  1. **Alta** — el formulario recoge razón social, NIF, email, teléfono, régimen fiscal, si tiene empleados y notas → normaliza la ficha → la añade a la hoja `Clientes` → avisa al equipo por Telegram.
  2. **Plazos** — cada mañana lee la cartera, calcula qué modelos vencen en los próximos **21 días** según el régimen y los empleados de cada cliente, y envía **un solo mensaje** con la lista ordenada por fecha. Si no hay nada próximo, no envía nada.
- **Modelos cubiertos:** 303, 130, 131, 111, 390, 190, 347 y 100 (calendario general estatal).
- **Qué necesitas:** una hoja de Google Sheets con las cabeceras indicadas en la guía, un bot de Telegram. n8n 1.40+.
- **Datos personales que trata:** identificativos y de contacto de los clientes de la asesoría (nombre, NIF, email, teléfono). Base legal habitual: ejecución del contrato de servicios y obligaciones legales del despacho. El flujo corre en tu propio n8n.
- **Ahorro estimado:** el alta deja de retipearse en dos sitios y los vencimientos dejan de depender de que alguien se acuerde de mirar el calendario. Con 40–60 clientes, el repaso manual diario son 10–15 minutos que desaparecen.
- **Tiempo de puesta en marcha:** 20–30 minutos.
- **Nota legal:** los avisos son **recordatorios internos de gestión**, no asesoramiento fiscal automatizado ni presentación de modelos ante la AEAT. Las fechas son las generales del calendario estatal: verifica siempre el calendario oficial y las particularidades forales (Navarra, País Vasco) antes de presentar.

## ✅ Clínica — recordatorio de cita y recuperación de cancelaciones

- **Archivo:** `workflows/clinica/recordatorio-citas.json`
- **Guía:** [`FLUJO-CLINICA.md`](FLUJO-CLINICA.md)
- **Vertical:** clínicas dentales, fisioterapia, estética, consultas privadas
- **Disparador:** formulario web (alta de cita) + programación diaria a las 18:00 (recordatorios) + webhook (enlaces del paciente)
- **Qué hace:**
  1. **Alta** — recepción rellena el formulario → la cita se guarda en la hoja `Citas` con estado `Pendiente` → aviso al equipo por Telegram.
  2. **Recordatorio** — cada día a las 18:00 lee la agenda, selecciona las citas de **mañana** (no canceladas ni atendidas) y envía **un email por paciente** con fecha, hora y dos enlaces: confirmar o cancelar. Al equipo le llega **un solo resumen** de la agenda, con las citas sin email marcadas para llamarlas.
  3. **Respuesta** — el paciente pulsa confirmar o cancelar → la hoja se actualiza por `ID` → el equipo recibe el aviso; si es cancelación, incluye el **hueco liberado** para ofrecerlo a otro paciente.
- **Qué necesitas:** una hoja de Google Sheets con las cabeceras de la guía, un bot de Telegram, una cuenta SMTP para los emails y una URL pública de n8n para los enlaces. n8n 1.40+.
- **Datos personales que trata:** identificativos y de contacto (nombre, teléfono, email) y la existencia de una cita, que es **dato de salud** (art. 9 RGPD). El email no incluye el motivo de la cita; el detalle queda en la hoja interna. El flujo corre en tu propio n8n.
- **Ahorro estimado:** con 30–50 citas al día, las llamadas de recordatorio son 1–2 horas diarias de recepción; además, cada cancelación avisada a tiempo permite recuperar el hueco. El flujo sustituye el "llamar uno por uno" por una revisión de la agenda.
- **Tiempo de puesta en marcha:** 30–40 minutos.
- **Nota legal:** requiere consentimiento informado del paciente para los recordatorios, política de privacidad que los mencione y acceso restringido a la hoja y al chat interno. **No diagnostica, no recomienda tratamientos y no toma decisiones clínicas.** Revisa la guía (sección RGPD) antes de usarlo con datos reales.

## ✅ Clinic appointments — reminders with confirm/cancel links (English)

- **Archivo:** `workflows/clinica/appointment-reminders-confirm-cancel.json`
- **Guía (EN):** [`FLOW-APPOINTMENTS-EN.md`](FLOW-APPOINTMENTS-EN.md)
- **Vertical:** clinics, dental practices, physiotherapy, private consultations
- **Disparador:** hosted form (booking) + daily schedule at 18:00 (reminders) + webhook (patient links)
- **Qué hace:** same three parts as the Spanish clinic flow, rebuilt in English for the global template library — 21 nodes, 11-column `Appointments` sheet, one reminder email per patient booked for tomorrow, single team summary with `NO EMAIL` markers, status update by `ID` and a Telegram alert that includes the **freed slot** when a patient cancels.
- **Diferencias con la versión ES:** nombres de columna, estados y textos en inglés (`Pending` / `Confirmed` / `Cancelled` / `Attended`), y los enlaces llevan también fecha y hora para que el aviso de cancelación indique el hueco exacto sin consultar la hoja.
- **Qué necesitas:** Google Sheets, bot de Telegram, cuenta SMTP y URL pública de n8n. n8n 1.40+.
- **Datos personales:** identificativos y de contacto + existencia de una cita (dato de salud, art. 9 RGPD). El email no incluye tratamiento ni motivo clínico.
- **Estado:** importado y reexportado sin errores en n8n 2.43.8; preparado para la librería de plantillas de n8n (envío pendiente de revisión).

---

## ✅ Inmobiliaria — captación de leads y respuesta automática

- **Archivo:** `workflows/inmobiliaria/captacion-leads-respuesta.json`
- **Guía:** [`FLUJO-INMOBILIARIA.md`](FLUJO-INMOBILIARIA.md)
- **Vertical:** inmobiliarias, agentes, promotoras
- **Disparador:** formulario web (nuevo lead) + programación diaria a las 10:00 (leads sin atender)
- **Qué hace:**
  1. **Captación** — el formulario recoge nombre, contacto, operación (comprar/alquilar/vender), tipo, zona, presupuesto, habitaciones, mensaje y consentimiento; el flujo normaliza los datos, puntúa el lead (0–100) y lo asigna a un comercial por zona.
  2. **Emparejamiento** — lee la cartera de la hoja `Inmuebles`, descarta los no disponibles, los de otra operación/tipo y los que superan el presupuesto en más de un 10 %, y propone **hasta 3** por encaje.
  3. **Respuesta** — guarda el lead en `Leads`, envía un email informativo con las opciones (solo con consentimiento) y avisa al equipo por Telegram con prioridad, comercial y referencias.
  4. **Seguimiento** — cada día a las 10:00 avisa de los leads que siguen en estado `Nuevo` más de 24 h.
- **Qué necesitas:** una hoja de Google Sheets con las pestañas `Inmuebles` y `Leads` (cabeceras en la guía), un bot de Telegram, una cuenta SMTP y, para el formulario público, una URL pública de n8n. n8n 1.40+.
- **Datos personales que trata:** identificativos y de contacto (nombre, email, teléfono, zona) y preferencias de búsqueda. El flujo corre en tu propio n8n y **no envía email sin consentimiento**. No introduzcas datos del art. 9 RGPD en el mensaje.
- **Ahorro estimado:** con 20–40 leads al mes, la respuesta manual uno por uno son 15–30 minutos diarios y los leads fríos se pierden; el flujo responde en segundos y deja el seguimiento en una sola bandeja.
- **Tiempo de puesta en marcha:** 30–40 minutos.
- **Nota legal:** la respuesta es informativa; no valora inmuebles, no fija precios ni promete disponibilidad. Confirma precio y estado antes de cerrar una operación. Nunca se hace scraping de plataformas que lo prohíben: usa su API oficial o su aviso por correo. Añade tu denominación social, NIF y política de privacidad al email antes de usarlo en producción.

---

## 💎 Pack Asesoría (premium)

Los flujos premium amplían los gratuitos: más pasos, versiones de producción (reintentos, aviso de errores, sin duplicados), panel de KPIs, guía de implantación de 30 días y plantillas de mensajes sectoriales.

**FlowKit — Pack Asesoría:** 29 € de lanzamiento (20 primeras unidades) → 39 €. **Pack Completo:** 79 €. A la venta en [mrnpvn.gumroad.com/l/flowkit-pack-asesoria](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria).

El pack incluye: los 3 flujos premium en `.json`, las guías de cada flujo, el plan de implantación en 30 días, las plantillas RGPD y las plantillas de mensajes editables. **Los `.json` premium no viven en este repositorio público**: aquí quedan las fichas y las guías completas para evaluarlos antes de comprar.

### Cobros vencidos — recordatorios escalonados y conciliación

- **Incluido en:** [FlowKit — Pack Asesoría](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria) · 29 € de lanzamiento. El `.json` no se publica en este repositorio.
- **Guía:** [`FLUJO-COBROS-VENCIDOS.md`](FLUJO-COBROS-VENCIDOS.md)
- **Vertical:** asesorías, gestorías, despachos contables
- **Disparadores:** diario 09:00 (recordatorios) + lunes 09:00 (informe de cartera) + webhook (enlace de pago) + Error Trigger
- **Qué hace:**
  1. **Recordatorios** — lee la hoja `Facturas`, agrupa por cliente y envía **un solo correo por cliente** con sus facturas vencidas. Tres niveles: 5, 15 y 30 días. Cada nivel se envía una sola vez y nunca dos veces el mismo día.
  2. **Informe de cartera** — cada lunes: total pendiente, tramos de antigüedad (al día, 1-30, 31-60, 61-90, +90), cinco mayores deudores y avisos de facturas sin email o con fecha inválida.
  3. **Enlace de pago** — el cliente marca el pago en un clic, sin login; la hoja pasa a `Pagada`, el equipo recibe el aviso por Telegram y la página responde 200/400.
  4. **Red de seguridad** — Error Trigger a Telegram con nodo y mensaje exactos, tras 3 reintentos por nodo.
- **Qué necesitas:** hoja de Google Sheets con la pestaña `Facturas` (13 columnas, en la guía), SMTP, bot de Telegram y URL pública de n8n. n8n 1.40+.
- **Datos personales que trata:** identificativos y de contacto del cliente y datos de facturación. Base legal: ejecución del contrato y obligaciones contables. Los correos son comunicación contractual, no marketing.
- **Ahorro estimado:** con 40–80 facturas mensuales, el seguimiento manual son 20–30 minutos por semana y se pierden días de caja; el flujo avisa solo, en el tono correcto y sin repetir.
- **Tiempo de puesta en marcha:** 30 minutos.
- **Nota legal:** los textos son informativos y ofrecen aplazamiento; no se automatizan amenazas, publicación de morosidad ni cesión a terceros. Si el impago se reclama judicialmente, revísalo antes con tu asesoría legal.

### Onboarding documental — checklist, recordatorios y portal del cliente

- **Incluido en:** [FlowKit — Pack Asesoría](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria) · 29 € de lanzamiento. El `.json` no se publica en este repositorio.
- **Guía:** [`FLUJO-ONBOARDING-DOCUMENTAL.md`](FLUJO-ONBOARDING-DOCUMENTAL.md)
- **Vertical:** asesorías, gestorías, despachos contables
- **Disparadores:** 2 formularios (alta y recepción) + diario 09:00 (seguimiento) + webhook GET (portal del cliente) + lunes 09:00 (informe) + Error Trigger
- **Qué hace:**
  1. **Alta con checklist automática** — formulario → detección de duplicados por email → checklist según servicio (fiscal, laboral, constitución, contable o a medida) → una fila por documento en `Docs` + una fila por cliente en `Onboarding` → email con la checklist y enlace privado → aviso al equipo con el ID de cada documento.
  2. **Seguimiento en 4 escalones** — a los 3, 7, 14 y 21 días, un solo correo por cliente con lo que falta. Cada nivel se envía una sola vez y nunca dos el mismo día.
  3. **Portal del cliente** — enlace privado con el progreso (recibido / pendiente / obligatorio). Solo lectura: no permite subir ni modificar nada.
  4. **Registrar recepción** — el equipo marca los documentos recibidos con el ID o el código; el flujo recalcula el porcentaje y, al 100 %, avisa y envía el correo de alta completada.
  5. **Informe semanal** — expedientes abiertos, completados, tasa de finalización, antigüedad media, bloqueados a +21 días y los documentos que más se atascan.
  6. **Red de seguridad** — reintentos 3× en todos los nodos externos, salida de error propia en los emails (un fallo no bloquea el lote) y Error Trigger a Telegram.
- **Qué necesitas:** hoja de Google con las pestañas `Docs` y `Onboarding` (cabeceras en la guía), SMTP, bot de Telegram y URL pública de n8n. n8n 1.40+.
- **Datos personales que trata:** identificativos y de contacto del cliente, y **el nombre del documento** recibido (no el archivo: los archivos no pasan por n8n). Base legal: ejecución del contrato y obligaciones legales del despacho.
- **Ahorro estimado:** con 10-20 altas al mes, cada alta manual con persecución de documentos son 20-40 minutos repartidos en varias semanas; el flujo lo deja en una checklist que se persigue sola y un panel que dice a quién hay que llamar.
- **Tiempo de puesta en marcha:** 30 minutos.
- **Nota legal:** el flujo persigue documentación, no evalúa solvencia ni adopta decisiones automatizadas sobre personas. El portal es de solo lectura y usa token aleatorio; no borres filas con enlaces vivos.

### Panel de KPIs — cobros, expedientes y documentación en una hoja

- **Incluido en:** [FlowKit — Pack Asesoría](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria) · 29 € de lanzamiento. El `.json` no se publica en este repositorio.
- **Guía:** [`FLUJO-KPI-PANEL.md`](FLUJO-KPI-PANEL.md)
- **Vertical:** asesorías, gestorías, despachos contables
- **Disparadores:** lunes 08:00 (semanal) + ejecución manual de prueba + Error Trigger
- **Qué hace:**
  1. **Lee y calcula** — cruza las hojas `Facturas`, `Onboarding` y `Docs` y calcula 27 KPIs: cartera pendiente y vencida por tramos, cobrado en 30 días, días medios de cobro, mayor deudor, expedientes abiertos/completados, tasa de finalización, antigüedad media, bloqueados +21 días y estado de la documentación.
  2. **Panel** — limpia y reescribe la pestaña `KPIs` (27 filas, valores numéricos + unidad + detalle) con semáforo de salud de cobro y salud general.
  3. **Histórico** — añade una fila por semana en `KPIs_Historico` para gráficos de tendencia.
  4. **Resumen** — manda a Telegram las cifras clave y **una acción recomendada** (llamar a bloqueados, revisar vencidos o mantener el ritmo).
  5. **Red de seguridad** — reintentos 3× en los 6 nodos de Sheets y los 2 de Telegram; Error Trigger con nodo y mensaje exactos.
- **Qué necesitas:** libro de Google Sheets con las pestañas `Facturas`, `Onboarding`, `Docs`, `KPIs` y `KPIs_Historico` (cabeceras en la guía), bot de Telegram. n8n 1.40+.
- **Datos personales que trata:** los mismos que los flujos de cobros y onboarding (identificativos, contacto, importes y estado de documentos). El panel se queda en tu libro y tu chat privado; no se envía nada a clientes. Base legal: interés legítimo en el control de la operación (art. 6.1.f RGPD).
- **Ahorro estimado:** sustituye el repaso manual de hojas y el informe mensual del despacho; 30-60 minutos al mes y, sobre todo, detecta antes la deuda vieja y los expedientes parados.
- **Tiempo de puesta en marcha:** 10 minutos.
- **Nota legal:** el panel informa, no decide: no hay decisiones automatizadas sobre personas. Los umbrales (tramos de deuda y semáforo) son editables y responsabilidad del despacho.

---

## Finance — vendor bank-change guard (English, for the n8n library)

- **Archivo:** [`workflows/finance/vendor-bank-change-guard.json`](../workflows/finance/vendor-bank-change-guard.json)
- **Guía (EN):** [`FLOW-VENDOR-BANK-CHANGE-GUARD.md`](FLOW-VENDOR-BANK-CHANGE-GUARD.md)
- **Vertical:** cualquier empresa con proveedores y pagos por transferencia (finanzas, operaciones, administración)
- **Disparador:** formulario de solicitud + webhook de decisión + vigilancia diaria 07:30 + Error Trigger
- **Qué hace:**
  1. **Retiene el cambio** — el proveedor envía los datos nuevos por formulario; el IBAN se valida (formato, **checksum mod-97** y país) y el proveedor se cruza con la hoja `Vendors`.
  2. **Puntúa el riesgo y decide** — proveedor desconocido, proveedor inactivo, país del IBAN distinto, dominio del solicitante distinto, buzón gratuito y cambios recientes suman puntos. Riesgo ≥ 70 va a revisión manual **sin enlace de aprobación**; riesgo ≥ 40 usa enfriamiento de 72 h (48 h por defecto).
  3. **Verifica fuera de banda** — el enlace de un solo uso va **solo a la dirección de finanzas ya guardada**, nunca a la que pide el cambio. Solo se guarda el hash SHA-256 del token.
  4. **Decide en un clic** — aprobar antes de que acabe el enfriamiento se rechaza sin tocar el estado; un segundo clic devuelve «Already handled»; tokens inválidos o caducados → HTTP 400 y fila en `Audit`.
  5. **Vigila y escala** — recordatorio a las 24 h, escalado al responsable a las 48 h, cierre como `EXPIRED` a las 72 h (con aviso al solicitante) y un digest diario para el responsable.
- **Qué necesitas:** un libro de Google Sheets con las pestañas `Vendors`, `BankChanges` y `Audit` (cabeceras en la guía), cuenta SMTP, bot de Telegram y la URL pública de tu n8n. n8n 1.40+.
- **Datos personales que trata:** datos de contacto del solicitante y del proveedor, y datos bancarios. El IBAN completo queda en la hoja porque finanzas lo necesita para aplicar el cambio: acceso restringido, verificación en dos pasos y plazo de conservación definido. Base legal habitual: ejecución de contrato e interés legítimo en prevención de fraude.
- **Ahorro estimado:** sustituye la verificación informal por correo (el vector habitual del fraude BEC) por un circuito con enfriamiento, aprobador conocido y rastro de auditoría. No elimina la llamada de confirmación: la hace obligatoria y demostrable.
- **Tiempo de puesta en marcha:** 10–15 minutos.
- **Nota legal:** no verifica la titularidad del IBAN ni aplica el cambio en el banco; eso lo hace una persona después de la aprobación. No sustituye controles de prevención de blanqueo ni el juicio del equipo de finanzas.
- **Nivel técnico:** avanzado (60 nodos, 7 Code, 4 Crypto, 11 Sheets).

---

## Historial de versiones

| Fecha | Cambio |
|---|---|
| 2026-10-19 | Publicación del repositorio: estructura, documentación y flujo gratuito `captura-leads-web` v1. |
| 2026-10-19 | Nuevo flujo `asesoria/alta-cliente-plazos` v1.0.0 (alta de cliente + recordatorio de 8 modelos a 21 días) + guía `FLUJO-ASESORIA.md`. |
| 2026-10-19 | Nuevo flujo `clinica/recordatorio-citas` v1.0.0 (alta de cita + email 24 h antes + confirmar/cancelar en un clic + aviso de hueco liberado) + guía `FLUJO-CLINICA.md`. |
| 2026-10-19 | Nuevo flujo `inmobiliaria/captacion-leads-respuesta` v1.0.0 (formulario + puntuación + emparejamiento con cartera + email al lead + reparto por zona + aviso de leads sin atender) + guía `FLUJO-INMOBILIARIA.md`. |
| 2026-10-20 | Flujo **premium** `asesoria/cobros-vencidos` v1.0.0 (recordatorios escalonados 5/15/30 días + informe semanal de cartera + enlace de pago + Error Trigger) + guía `FLUJO-COBROS-VENCIDOS.md`. |
| 2026-10-20 | Flujo **premium** `asesoria/onboarding-documental` v1.0.0 (alta con checklist por servicio + seguimiento 3/7/14/21 días + portal privado del cliente + registro de recepción + informe semanal + Error Trigger) + guía `FLUJO-ONBOARDING-DOCUMENTAL.md`. |
| 2026-10-20 | Flujo **premium** `asesoria/kpi-panel` v1.0.0 (27 KPIs de cobros, expedientes y documentación + panel en Sheets + histórico semanal + resumen y acción por Telegram + Error Trigger) + guía `FLUJO-KPI-PANEL.md`. |
| 2026-10-21 | Flujo gratuito `finance/vendor-bank-change-guard` v1.0.0 (validación IBAN mod-97 + scoring de riesgo + token de un solo uso con hash + enfriamiento + watchdog 24/48/72 h + auditoría) + guía `FLOW-VENDOR-BANK-CHANGE-GUARD.md`. |
