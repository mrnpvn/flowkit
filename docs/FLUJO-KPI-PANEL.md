# Panel de KPIs — cobros, expedientes y documentación en una sola hoja

Guía del flujo **premium** `workflows/asesoria/kpi-panel.json` (15 nodos funcionales + 5 notas, n8n 2.43.8).
Forma parte del **FlowKit — Pack Asesoría** y se apoya en los otros dos flujos premium.

---

## Qué hace

| Parte | Cuándo | Qué hace |
|---|---|---|
| **A · Leer y calcular** | Cada lunes, 08:00 | Lee las hojas `Facturas`, `Onboarding` y `Docs`, y calcula **27 KPIs** de cobros, expedientes y documentación. Tolera fechas `dd/mm/aaaa`, `aaaa-mm-dd` y seriales de Sheets, e importes `1.234,56` o `1234.56`. |
| **B · Panel** | Tras el cálculo | Limpia `A2:E500` de la pestaña `KPIs` y escribe una fila por KPI. Si una hoja está vacía, el panel sale a cero: no falla. |
| **C · Histórico** | Tras el cálculo | Añade **una fila por semana** en `KPIs_Historico` para poder hacer gráficos de tendencia. |
| **D · Resumen** | Tras escribir el panel | Envía a Telegram el resumen con las cifras clave y **una acción recomendada** (llamar a bloqueados, revisar vencidos o mantener el ritmo). |
| **E · Red de seguridad** | Si algo falla | Error Trigger → Telegram con el nodo y el mensaje exactos. Los 6 nodos de Sheets y los 2 de Telegram llevan 3 reintentos. |

**Importante:** este flujo **no envía correos ni escribe a clientes**. Solo lee hojas, escribe en tu libro y avisa a tu equipo.

---

## Qué mide (27 KPIs)

### Cobros

| KPI | Qué significa |
|---|---|
| Facturas pendientes | Facturas en estado distinto de `Pagada` e `Incobrable`. |
| Importe pendiente | Suma de esas facturas. |
| Importe vencido | Pendiente con `Fecha_Vencimiento` anterior a hoy. |
| Porcentaje vencido | Vencido / pendiente. Es el termómetro de la caja. |
| Vencido 1-30 / 31-60 / 61-90 / +90 días | Tramos de antigüedad de la deuda. |
| Cobrado últimos 30 días | Facturas `Pagada` con `Fecha_Pago` en los últimos 30 días. |
| Días medios de cobro | Media de días entre emisión y pago (facturas pagadas en 90 días). |
| Mayor deudor | Cliente con más importe vencido y su cifra. |
| Facturas sin email | Pendientes que **no** pueden recibir recordatorio. |
| Recordatorios de cobro (30 d) | Facturas con `Ultimo_Recordatorio` en los últimos 30 días. |
| Salud de cobro | `VERDE` (<25 % vencido), `AMBAR` (25-49 %) o `ROJO` (≥50 %). |

### Clientes y documentación

| KPI | Qué significa |
|---|---|
| Expedientes abiertos / completados | Según `Estado` de la hoja `Onboarding`. |
| Tasa de finalización | Completados / total de expedientes. |
| Antigüedad media (abiertos) | Días desde `Fecha_alta`, solo abiertos. |
| Bloqueados +21 días | Abiertos con más de 21 días: **la lista de llamadas de la semana**. |
| Documentos pendientes / recibidos | Conteo sobre la hoja `Docs`. |
| Documentación recibida | % recibidos sobre el total. |
| Servicio más lento | Servicio con mayor antigüedad media entre los expedientes abiertos. |

### Actividad

| KPI | Qué significa |
|---|---|
| Altas de clientes (30 d) | Expedientes creados en los últimos 30 días. |
| Recordatorios de documentación (30 d) | Documentos con `Ultimo_recordatorio` en los últimos 30 días. |
| Salud general | `ROJO` si la salud de cobro es roja; `AMBAR` si hay cobros en ámbar **o** expedientes bloqueados; si no, `VERDE`. |
| Calculado | Fecha del cálculo (control de frescura del panel). |

---

## Requisitos

- n8n **1.40+** (probado en 2.43.8).
- Los flujos premium de **cobros vencidos** y **onboarding documental** ya funcionando (o, al menos, sus hojas con las cabeceras).
- Un libro de Google Sheets con **cinco pestañas**: `Facturas`, `Onboarding`, `Docs`, `KPIs` y `KPIs_Historico`.
- Bot de Telegram + chat del equipo.

---

## Las pestañas `KPIs` y `KPIs_Historico`

Crea las dos pestañas **en el mismo libro** y pega estas cabeceras en la **fila 1** (tal cual, sin cambiar mayúsculas ni acentos):

**`KPIs`** (el flujo limpia desde `A2` y escribe debajo):

| A | B | C | D | E |
|---|---|---|---|---|
| `Seccion` | `KPI` | `Valor` | `Unidad` | `Detalle` |

**`KPIs_Historico`** (el flujo solo añade filas; no la limpia nunca):

| A | B | C | D | E | F | G | H | I | J | K |
|---|---|---|---|---|---|---|---|---|---|---|
| `Fecha` | `Pendiente_eur` | `Vencido_eur` | `Pct_vencido` | `Cobrado_30d_eur` | `Facturas_pendientes` | `Clientes_abiertos` | `Docs_pendientes` | `Tasa_completado` | `Bloqueados_21d` | `Salud` |

`Valor` es numérico (sin símbolos) y `Unidad` dice cómo leerlo (`EUR`, `%`, `dias`, `facturas`, `expedientes`, `documentos` o `-`). Así puedes graficar directamente la columna `Valor`.

---

## Instalación (10 minutos)

1. **Importa** `kpi-panel.json` en n8n: *Workflows → Import from File*.
2. Abre los **3 nodos de lectura** (*Leer facturas*, *Leer onboarding*, *Leer documentos*), elige el libro y la pestaña de cada uno y la credencial de Google Sheets.
3. Abre **Limpiar panel** → mismo libro, pestaña `KPIs`.
4. Abre **Escribir panel** → mismo libro, pestaña `KPIs`.
5. Abre **Añadir al histórico** → mismo libro, pestaña `KPIs_Historico`.
6. Abre los **2 nodos de Telegram** (*Resumen semanal* y *Aviso de error*), elige la credencial y pon el `chatId` del equipo.
7. Pulsa **Prueba manual** y revisa la hoja `KPIs`. Debe haber 27 filas y una nueva en `KPIs_Historico`.
8. Activa el flujo. A partir de ahí corre **cada lunes a las 08:00** (zona `Europe/Madrid`).

> La primera ejecución con hojas vacías es válida: verás todo a cero y `Salud` en `VERDE`.

---

## Cómo leerlo (y sacarle partido)

- **Formato condicional** en `KPIs!E:E`: texto que contenga `ROJO` → fondo rojo; `AMBAR` → ámbar; `VERDE` → verde. Se hace una vez y el panel se colorea solo.
- **Gráfico de tendencia** en `KPIs_Historico`: selecciona `Fecha` + `Vencido_eur` + `Cobrado_30d_eur` → *Insertar → Gráfico → Líneas*. En 4-6 semanas se ve si la caja mejora de verdad.
- **Gráfico de composición** de la deuda: en `KPIs`, filtra `Seccion = Cobros` y usa los cuatro tramos (`Vencido 1-30`, `31-60`, `61-90`, `+90`) → gráfico de columnas.
- **Rutina del lunes (10 min):** abre el panel → mira `Salud` → si hay `Bloqueados +21 días`, llama a esos clientes → si `Porcentaje vencido` sube dos semanas seguidas, revisa el flujo de cobros.

**Umbrales editables** (nodo *Calcular KPIs*, al principio del código): los tramos están escritos como 30/60/90 días y la salud como 25/50 %. Cámbialos si tu despacho trabaja con otros plazos. El histórico guarda los valores ya calculados: si cambias umbrales, la serie antigua no se recalcula (es lo esperable).

---

## Qué datos trata

- **Origen:** las mismas hojas que ya usan los flujos de cobros y onboarding (nombre de cliente, email, importes, fechas, estado de documentos).
- **Destino:** tu propio libro de Google Sheets y tu chat de Telegram. **No sale nada a terceros** y no se envía nada a los clientes.
- **Base legal:** interés legítimo del despacho en controlar su cobro y su operación (art. 6.1.f RGPD). Los datos no se usan para decisiones automatizadas sobre personas: el panel informa, la decisión es humana.
- **Buena práctica:** el chat de Telegram del equipo debe ser privado; el panel muestra nombres de clientes y cifras.

---

## Problemas frecuentes

| Síntoma | Causa habitual | Solución |
|---|---|---|
| «No se encontró la hoja» | Documento o pestaña sin seleccionar | Abre los 6 nodos de Sheets y elige libro + pestaña en cada uno. |
| El panel sale a cero | Hojas de origen vacías o pestaña equivocada | Comprueba que los 3 nodos de lectura apuntan a `Facturas`, `Onboarding` y `Docs`. |
| «Column mismatch» al escribir | Cabeceras de `KPIs` distintas a `Seccion/KPI/Valor/Unidad/Detalle` | Copia las cabeceras exactas de esta guía. |
| El histórico se llena de ceros | Ejecución de prueba con hojas vacías | Es normal en la prueba; puedes borrar esa fila a mano. |
| Un importe sale raro | Formato `1,234.56` (anglosajón) | Usa `1.234,56` o `1234.56` en la hoja `Facturas`. |
| Una fecha no vence nunca | Fecha imposible (`31/02`) o texto libre | El flujo la ignora como vencida; corrígela en la hoja. |
| Llega aviso de error | Mira el texto de Telegram: nodo + mensaje | Corrige la credencial o el dato y vuelve a ejecutar. |
| Se duplican filas en el histórico | Varias ejecuciones el mismo día | Es intencionado (una fila por ejecución); borra las sobrantes. |

---

## Personalización

- **Añadir un KPI:** en *Calcular KPIs*, añade una línea `add('Seccion', 'Nombre', valor, 'unidad', 'detalle')`. Aparecerá en el panel sin tocar los nodos de Sheets.
- **Cambiar el día:** en el trigger, la expresión cron `0 8 * * 1` es lunes a las 08:00. `0 8 * * 5` sería viernes.
- **Quitar el histórico:** desconecta *Preparar histórico* → *Añadir al histórico*. El panel sigue funcionando.
- **Cambiar el resumen de Telegram:** el texto se monta al final de *Calcular KPIs* (`const resumen = [...]`). No toques la estructura del array si no quieres romper el formato.

---

## Verificación realizada

- **Import + export sin errores en n8n real 2.43.8**: 20/20 nodos y 13 grupos de conexiones preservados.
- **Validación estructural:** IDs y nombres únicos, 14 enlaces válidos, 0 nodos inalcanzables, 0 mojibake, 6 nodos Code compilando.
- **Lógica probada en Node** con datos simulados:
  - cartera realista (8 facturas, 4 expedientes, 6 documentos): pendiente, vencido, cuatro tramos, cobrado 30 d, días medios de cobro, mayor deudor, sin email, bloqueados, tasa, % de documentación y servicio más lento;
  - hojas vacías y filas `{}`: panel de 27 filas a cero, salud `VERDE`, sin errores;
  - 10 formatos de importe (`1.234,56`, `1234.56`, `1.234`, `12,5`, `1,234.56`, `-50,25`, `10,50 €`…);
  - fecha imposible `31/02/2026`: no se cuenta como vencida y no rompe el cálculo.
