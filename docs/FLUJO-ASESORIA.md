# Flujo: Asesoría — alta de cliente + recordatorio de plazos fiscales

Guía de puesta en marcha del flujo `workflows/asesoria/alta-cliente-plazos.json`.
Tiempo estimado: **20–30 minutos**. No hace falta saber programar.

> **Qué es y qué no es.** El flujo automatiza dos tareas **internas** de una asesoría: dar de alta un cliente sin retipear datos y avisar al equipo de los modelos que se acercan. **No presenta modelos ante la AEAT, no calcula cuotas y no sustituye el criterio de un asesor.** Las fechas son las generales del calendario estatal: confirma siempre el calendario oficial y las particularidades forales antes de presentar.

---

## 1. Antes de empezar

| Necesitas | Detalle |
|---|---|
| n8n | 1.40 o superior (Cloud o self-hosted) |
| Google Sheets | Una hoja de cálculo tuya |
| Telegram | Un bot (`@BotFather` → `/newbot`) y un chat de equipo |

---

## 2. Prepara la hoja de cálculo

Crea una hoja nueva llamada, por ejemplo, **Clientes asesoría** y en la **primera fila** escribe exactamente estas cabeceras, una por columna y en este orden:

```
Nombre | NIF | Email | Telefono | Regimen | Empleados | Notas | Fecha alta | Activo
```

Notas importantes:

- Los nombres **deben coincidir** con los de arriba: el flujo escribe por nombre de columna.
- `Regimen` admite estos valores (el formulario los ofrece como desplegable):
  - `Autónomo — estimación directa`
  - `Autónomo — módulos`
  - `Sociedad limitada`
  - `Particular (sin actividad)`
- `Empleados` admite `Sí` o `No`.
- `Activo` se rellena solo con `Sí` al dar de alta. Pon `No` cuando un cliente cause baja: el flujo deja de avisar de sus plazos.
- Si prefieres dar de alta a mano, respeta las mismas cabeceras y rellena `Activo` con `Sí`.

---

## 3. Importa el flujo

**n8n → Workflows → ⋯ → Import from File…** y elige `alta-cliente-plazos.json`.

Verás **dos partes** en el lienzo:

- **A) Alta de cliente** — formulario → normalizar → guardar → aviso.
- **B) Recordatorios** — cada día a las 8:00 → leer clientes → calcular plazos → aviso.

Las dos partes viven en el mismo flujo pero **se ejecutan de forma independiente**.

---

## 4. Configura los cuatro nodos que faltan

### 4.1 Google Sheets — nodo *Guardar ficha en Clientes*

1. Doble clic → **Credential** → crea o elige tu credencial de Google Sheets.
2. **Spreadsheet**: elige la hoja del paso 2.
3. **Sheet**: elige la pestaña concreta (por ejemplo `Hoja 1`).
4. **Mapping Column Mode**: deja `Map Automatically`.

### 4.2 Google Sheets — nodo *Leer clientes*

Misma credencial, **misma hoja y misma pestaña** que el nodo anterior.
Deja la operación en **Get Row(s)** sin filtros: el flujo se encarga de descartar los inactivos.

### 4.3 Telegram — nodo *Aviso alta (Telegram)*

1. **Credential**: crea una credencial *Telegram API* con el token de tu bot.
2. **Chat ID**: elige el chat del equipo. Si no aparece, escribe primero algo al bot y recarga la lista.
3. No toques el texto: ya trae las variables rellenas.

### 4.4 Telegram — nodo *Aviso plazos (Telegram)*

Misma credencial y mismo chat. Este nodo solo se ejecuta **si hay plazos** en los próximos 21 días; si no hay ninguno, el flujo termina antes y no manda nada (así no genera ruido).

---

## 5. Prueba antes de activar

1. **Parte A**: pulsa *Execute Workflow*. Abre el formulario que muestra el nodo, rellena un cliente de prueba y envíalo. Comprueba que:
   - aparece una fila nueva en la hoja;
   - llega el aviso a Telegram.
2. **Parte B**: con ese cliente de prueba en la hoja, pulsa *Execute Workflow* otra vez. Si hoy cae dentro de los 21 días previos a un vencimiento, recibirás la lista. Si no, verás que el nodo *Calcular plazos próximos* devuelve **0 items**: es el comportamiento correcto.
3. Borra la fila de prueba.
4. Activa el flujo con el interruptor **Active**.

> Truco para probar la parte B en cualquier fecha: cambia temporalmente `DIAS_AVISO` (primera línea del nodo *Calcular plazos próximos*) a `120`, ejecuta, comprueba el mensaje y vuelve a dejarlo en `21`.

---

## 6. Qué avisa exactamente

Vencimientos generales que incluye la versión 1.0.0 (día y mes, se repiten cada año):

| Modelo | Qué es | Vence | A quién avisa |
|---|---|---|---|
| 303 | IVA trimestral | 20 de ene / abr / jul / oct | regímenes con IVA (directa, módulos, SL) |
| 130 | IRPF pago fraccionado, estimación directa | 20 de ene / abr / jul / oct | autónomos en directa |
| 131 | IRPF pago fraccionado, módulos | 20 de ene / abr / jul / oct | autónomos en módulos |
| 111 | Retenciones del trabajo, trimestral | 20 de ene / abr / jul / oct | clientes con empleados |
| 390 | Resumen anual de IVA | 30 de enero | regímenes con IVA |
| 190 | Resumen anual de retenciones | 31 de enero | clientes con empleados |
| 347 | Operaciones con terceros | 28 de febrero | regímenes con IVA |
| 100 | Declaración de la renta | 30 de junio | autónomos en directa |

**Aviso de 21 días** de antelación: el flujo agrupa todos los vencimientos próximos en **un único mensaje** ordenado por fecha. Un mensaje al día como máximo, no uno por cliente.

### Personalizar

Todo el calendario está en el nodo **Calcular plazos próximos**, en dos bloques legibles:

- `const T = [[4, 20, '1T'], ...]` — los cuatro trimestres (`mes, día`).
- Las líneas `add('303', ...)` — cada modelo con su condición (`tieneIVA`, `emp`, ...).

Para añadir un modelo nuevo, copia una línea `add(...)` y ajusta código, nombre, mes, día y condición. Para cambiar la antelación, edita `DIAS_AVISO`.

---

## 7. Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| El formulario no aparece al ejecutar | El flujo está en modo prueba y el nodo de formulario usa la URL de test | Abre el nodo y usa la URL de **producción** (tras activar) en tu web/intranet |
| Fila nueva con columnas vacías o desordenadas | Las cabeceras de la hoja no coinciden exactamente | Revisa el paso 2: `Telefono` sin tilde, `Fecha alta` con espacio, `Regimen` sin tilde |
| `403` en Google Sheets | Falta permiso de escritura o la cuenta de servicio no ve la hoja | Vuelve a autorizar pidiendo escritura, o comparte la hoja con el email de la cuenta de servicio |
| No llega el aviso de plazos | No hay vencimientos en 21 días, o todos los clientes están en `Activo = No` | Es lo esperado; prueba con `DIAS_AVISO = 120` |
| Llega `{{ ... }}` literal | Se borró el `=` inicial del campo de texto | Vuelve a poner `=` al principio del campo |
| El mensaje se corta | Telegram limita a 4096 caracteres; el flujo recorta a 3500 | Con carteras muy grandes, divide el aviso por comercial o filtra por régimen |

---

## 8. Legal y RGPD (leer una vez)

- Los datos tratados son identificativos y de contacto de **clientes de la asesoría** (nombre, NIF, email, teléfono). La base legal habitual es la **ejecución de un contrato de prestación de servicios** y/o el cumplimiento de obligaciones legales del despacho.
- El flujo corre **en tu n8n**, con tus credenciales: los datos no pasan por servidores de FlowKit.
- Si usas Google Sheets, necesitas el **contrato de encargado de tratamiento** con Google (incluido en Google Workspace).
- Define una **retención** razonable (por ejemplo, mientras dure la relación y los años de prescripción fiscal) y borra lo que ya no necesites.
- Los avisos internos no son comunicaciones comerciales: no requieren consentimiento de marketing, pero sí que el personal autorizado sea el único con acceso al chat de Telegram.
- Este flujo **no** realiza perfilado ni decisiones automatizadas con efectos jurídicos sobre el cliente.

---

## 9. Mantenimiento

- Revisa una vez al mes que el flujo sigue **Active** y que las filas se guardan.
- Cuando cambie el calendario fiscal, actualiza el nodo *Calcular plazos próximos* y anota el cambio en `docs/CATALOGO.md` (historial de versiones).
- Si añades columnas a la hoja, no rompes nada: el flujo escribe por nombre de columna y las nuevas quedan vacías.
