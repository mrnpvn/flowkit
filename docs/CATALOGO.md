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

## 🚧 Inmobiliaria — captación de leads y respuesta automática

- **Vertical:** inmobiliarias, agentes, promotoras
- **Qué hará:** recoger el lead (portal, web o formulario), clasificarlo por interés y zona, repartirlo entre comerciales y disparar una **primera respuesta inmediata** con la ficha del inmueble y una propuesta de visita.
- **Qué necesitará:** Google Sheets, Telegram, email. Compatible con los portales que ofrecen aviso por correo o API oficial.
- **Nota:** nunca se hace scraping de plataformas que lo prohíben; solo entradas propias (formulario, correo, API oficial).

---

## 💎 Pack premium (en preparación)

Los flujos premium amplían los gratuitos: más pasos, más integraciones (Notion, Stripe, WhatsApp vía API oficial, CRM), plantillas de mensajes sectoriales, guía de implantación y soporte por correo.

Precio previsto: **29–79 €** según el pack. Se anunciará en este repositorio y en el perfil de Gumroad de AIVANTA.

---

## Historial de versiones

| Fecha | Cambio |
|---|---|
| 2026-10-19 | Publicación del repositorio: estructura, documentación y flujo gratuito `captura-leads-web` v1. |
| 2026-10-19 | Nuevo flujo `asesoria/alta-cliente-plazos` v1.0.0 (alta de cliente + recordatorio de 8 modelos a 21 días) + guía `FLUJO-ASESORIA.md`. |
| 2026-10-19 | Nuevo flujo `clinica/recordatorio-citas` v1.0.0 (alta de cita + email 24 h antes + confirmar/cancelar en un clic + aviso de hueco liberado) + guía `FLUJO-CLINICA.md`. |
