# Flujos

Cada carpeta es una vertical. Cada archivo `.json` es un flujo exportado de n8n, listo para importar.

```
workflows/
├── general/        Flujos que sirven a cualquier negocio
├── asesoria/       Asesorías, gestorías y despachos contables
├── clinica/        Clínicas dentales, fisioterapia, consultas privadas
└── inmobiliaria/   Inmobiliarias, agentes y promotoras
```

## Cómo importar

**n8n → Workflows → ⋯ → Import from File…** y selecciona el archivo.
Pasos detallados y credenciales: [`../docs/INSTALACION-N8N.md`](../docs/INSTALACION-N8N.md).

## Índice

| Flujo | Carpeta | Estado |
|---|---|---|
| Captura de leads web | `general/` | ✅ listo |
| Captura de leads web (EN, para la librería n8n) | `general/` | ✅ listo |
| Alta de cliente + plazos fiscales | `asesoria/` | ✅ listo |
| Recordatorio de cita + recuperación de cancelaciones | `clinica/` | ✅ listo |
| Appointment reminders with confirm/cancel links (EN, para la librería n8n) | `clinica/` | ✅ listo |
| Captación de leads + respuesta automática | `inmobiliaria/` | ✅ listo |
| 💎 Cobros vencidos (recordatorios + informe + enlace de pago) | `asesoria/` | 💎 premium |
| 💎 Onboarding documental (checklist + seguimiento + portal) | `asesoria/` | 💎 premium |
| 💎 Panel de KPIs (cobros, expedientes y documentación) | `asesoria/` | 💎 premium |

Las fichas completas (qué hace, qué datos trata, tiempo de puesta en marcha) están en [`../docs/CATALOGO.md`](../docs/CATALOGO.md).

> Los flujos marcados como premium **no están en este repositorio**: se entregan dentro del [Pack Asesoría](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria) (29 € de lanzamiento). Su documentación sí es pública en `docs/`.
