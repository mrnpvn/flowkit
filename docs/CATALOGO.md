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

## 🚧 Asesoría — alta de cliente y recordatorio de plazos fiscales

- **Vertical:** asesorías, gestorías, despachos contables
- **Qué hará:** recibir los datos de un cliente nuevo (formulario o texto pegado), crear su ficha en la hoja o CRM, avisar al equipo, y programar los recordatorios de sus obligaciones periódicas (IVA, IRPF, resúmenes) para que no dependan de la memoria de nadie.
- **Qué necesitará:** Google Sheets o Notion, Telegram o email (SMTP).
- **Nota legal:** los avisos son **recordatorios internos de gestión**, no asesoramiento fiscal automatizado ni presentación de modelos ante la AEAT.

## 🚧 Clínica — recordatorio de cita y recuperación de cancelaciones

- **Vertical:** clínicas dentales, fisioterapia, estética, consultas privadas
- **Qué hará:** avisar al paciente de su cita 24 h antes, permitir confirmar o cancelar con un clic, y cuando alguien cancele **avisar al equipo con el hueco liberado** para intentar rellenarlo.
- **Qué necesitará:** Google Calendar o tu agenda en Sheets, Telegram o email.
- **Nota legal:** se manejan **datos de salud** (categoría especial, art. 9 RGPD). Requiere consentimiento informado y una política de privacidad que mencione los recordatorios; el flujo no incluye diagnóstico ni recomendación médica.

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
