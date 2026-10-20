# Flujo: Inmobiliaria — captación de leads y respuesta automática

Guía de puesta en marcha del flujo `workflows/inmobiliaria/captacion-leads-respuesta.json`.
Tiempo estimado: **30–40 minutos**. No hace falta saber programar.

> **Qué es y qué no es.** Automatiza tareas **administrativas** de una inmobiliaria: registrar un lead sin retipear datos, ordenarlo por prioridad, asignarlo a un comercial por zona, proponerle hasta tres inmuebles de la cartera y avisar al equipo. **No valora inmuebles, no fija precios, no firma contratos y no sustituye el criterio del comercial.** La respuesta automática es informativa: el precio y la disponibilidad de cada inmueble deben confirmarse antes de cerrar cualquier operación.

---

## 1. Antes de empezar

| Necesitas | Detalle |
|---|---|
| n8n | 1.40 o superior (Cloud o self-hosted) |
| Google Sheets | Una hoja de cálculo con dos pestañas: `Inmuebles` y `Leads` |
| Telegram | Un bot (`@BotFather` → `/newbot`) y un chat interno del equipo |
| Email (SMTP) | Una cuenta desde la que responder al lead (Gmail con contraseña de aplicación, Brevo, Mailgun, tu proveedor…) |
| URL pública | Solo si quieres que el formulario sea accesible desde fuera de tu red |

> El flujo **no usa IA ni APIs de pago**. La clasificación y el emparejamiento son reglas deterministas que puedes leer y ajustar en el nodo `Clasificar y emparejar`.

---

## 2. Prepara la hoja de cálculo

Crea una hoja nueva (por ejemplo **Leads inmobiliaria**) con **dos pestañas**.

### 2.1 Pestaña `Inmuebles`

En la **primera fila** escribe exactamente estas cabeceras, una por columna y en este orden:

```
ID | Titulo | Operacion | Tipo | Zona | Precio | Habitaciones | Superficie | Estado | Enlace | Notas
```

- `ID`: referencia única del inmueble (por ejemplo `INM-001`). No la repitas.
- `Operacion`: `Venta` o `Alquiler`.
- `Tipo`: `Piso`, `Casa`, `Chalet`, `Local`, `Oficina` o `Terreno` (los mismos valores del formulario).
- `Precio`: número o texto con separadores (`280000`, `280.000` o `280.000 €`). El flujo lo interpreta.
- `Habitaciones` y `Superficie`: números.
- `Estado`: `Disponible`, `Reservado` o `Vendido`. Solo se proponen inmuebles `Disponible`.
- `Enlace` y `Notas`: opcionales; no se envían al lead, quedan para el equipo.

### 2.2 Pestaña `Leads`

En la **primera fila** escribe exactamente estas cabeceras:

```
ID | Fecha | Nombre | Email | Telefono | Operacion | Tipo | Zona | Presupuesto | Habitaciones | Mensaje | Consentimiento | Puntuacion | Prioridad | Comercial | Inmuebles | Estado
```

- `ID` lo genera el flujo (`LEAD-AAAAMMDD-HHMM-XXXX`). No lo cambies: es la referencia del lead.
- `Estado`: el flujo escribe `Nuevo`. Después lo actualizas a mano: `Contactado`, `Visita`, `Cerrado` o `Descartado`.
- `Inmuebles`: referencias propuestas y su puntuación de encaje (por ejemplo `INM-001 (90) | INM-002 (53)`).
- `Puntuacion` y `Prioridad` se calculan automáticamente (ver sección 5).
- `Consentimiento`: `Sí` o `No`. Solo se envía email automático cuando es `Sí`.

> Los nombres deben coincidir **exactamente** con los de arriba: el flujo escribe por nombre de columna. Si cambias una cabecera, cambia también la asignación en el nodo `Preparar fila del lead`.

---

## 3. Importa el flujo

**n8n → Workflows → ⋯ → Import from File…** y elige `captacion-leads-respuesta.json`.

Verás **dos partes** en el lienzo:

- **A) Nuevo lead y respuesta** — formulario → normalizar → leer cartera → clasificar y emparejar → guardar en `Leads` → email al lead + aviso al equipo.
- **B) Leads sin atender** — cada día a las 10:00 → leer `Leads` → avisar de los que siguen en estado `Nuevo` más de 24 h.

Las dos partes viven en el mismo flujo pero **se ejecutan de forma independiente**.

---

## 4. Configura los nodos que faltan

### 4.1 Google Sheets — los tres nodos

En **Leer cartera de inmuebles**, **Guardar lead en Leads** y **Leer leads**:

1. **Credential**: crea o elige tu credencial de Google Sheets.
2. **Spreadsheet**: elige la hoja del paso 2.
3. **Sheet**: elige la pestaña correspondiente (`Inmuebles` o `Leads`).
4. En *Guardar lead en Leads* deja **Mapping Column Mode** en `Map Automatically`.

> El nodo *Leer cartera de inmuebles* tiene activada la opción **Always Output Data**. Así el flujo sigue funcionando aunque la pestaña `Inmuebles` esté vacía: el lead se guarda igualmente y la respuesta es genérica.

### 4.2 Telegram — los dos nodos

En **Aviso lead al equipo** y **Recordatorio de leads sin atender**:

1. **Credential**: crea una credencial *Telegram API* con el token de tu bot.
2. **Chat ID**: elige el chat **interno del equipo** (no el del cliente). Si no aparece, escribe primero algo al bot y recarga la lista.

### 4.3 Email — nodo *Email de respuesta al lead*

1. **Credential**: crea una credencial *SMTP* con los datos de tu proveedor.
   - Gmail: `smtp.gmail.com`, puerto `465` (SSL) y una **contraseña de aplicación** (no tu contraseña normal). Requiere verificación en dos pasos.
2. **From Email**: la dirección desde la que quieres responder (por ejemplo `inmobiliaria@tudominio.es`).
3. No toques *To Email*, *Subject* ni *Text*: usan variables del flujo.

> El nodo tiene **Continue on Fail** activado por diseño del flujo: si el SMTP falla, el lead ya está guardado y el aviso a Telegram se envía igualmente. El error queda en *Executions*.

### 4.4 Nombre de la agencia, zonas y comerciales

Abre el nodo **Clasificar y emparejar**. Las primeras líneas son configuración:

```js
const AGENCIA = 'Tu inmobiliaria';
const AGENTE_DEFECTO = 'Equipo comercial';
const AGENTES = {
  centro: 'Ana',
  norte: 'Luis',
  sur: 'Marta',
  este: 'Jorge',
  oeste: 'Elena'
};
```

- `AGENCIA`: el nombre que verá el lead en el email.
- `AGENTES`: asigna un comercial a cada zona. La comparación ignora mayúsculas y acentos y busca la palabra dentro de la zona (`Centro histórico` → `Ana`). Si no hay coincidencia, se usa `AGENTE_DEFECTO`.
- Si tu equipo no trabaja por zonas, deja solo `AGENTE_DEFECTO` y vacía el mapa.

### 4.5 Zona horaria y hora del recordatorio

- En **Settings → Timezone** del flujo (o en la configuración general de n8n) elige `Europe/Madrid`. El flujo calcula la fecha de alta y la antigüedad de los leads con la hora del servidor.
- El recordatorio de leads sin atender sale **cada día a las 10:00**. Para cambiarlo, abre el nodo **Cada día a las 10:00** y ajusta la hora.

### 4.6 URL del formulario

- El formulario se publica en `https://TU-N8N/form/flowkit-inmobiliaria` (o la ruta que elijas en el nodo **Formulario nuevo lead**).
- Si tu n8n solo es accesible en `localhost`, el formulario solo funcionará en tu red. Para producción, usa una URL pública estable (dominio propio, Cloudflare Tunnel, etc.).
- Enlaza el formulario desde tu web y añade junto a él tu **política de privacidad**.

---

## 5. Cómo puntúa y empareja (sin caja negra)

### 5.1 Puntuación del lead (0–100)

| Señal | Puntos |
|---|---|
| Datos mínimos del formulario | 30 |
| Email válido | +15 |
| Teléfono | +15 |
| Presupuesto indicado | +10 |
| Zona indicada | +10 |
| Mensaje de 20 caracteres o más | +10 |
| Consentimiento de privacidad | +10 |

- **Alta**: 70–100 → aviso destacado al equipo.
- **Media**: 45–69.
- **Baja**: menos de 45.

La puntuación ordena el trabajo interno; **no rechaza ni bloquea** a ningún lead. Todos se guardan.

### 5.2 Emparejamiento con la cartera

1. Descarta inmuebles que no estén en estado `Disponible`.
2. Exige que la operación coincida (`Comprar` → `Venta`; `Alquilar` → `Alquiler`). Si el lead quiere `Vender`, no se empareja: la respuesta es una propuesta de valoración.
3. Exige que el tipo de inmueble coincida cuando el lead lo indica.
4. Descarta inmuebles cuyo precio supere el presupuesto en más de un **10 %**.
5. Puntúa los restantes: zona (+30), tipo (+20), habitaciones exactas (+15) o ±1 (+8), precio dentro de presupuesto (+25).
6. Ordena de mayor a menor y envía **hasta 3**.

Puedes cambiar los pesos, el umbral del 10 % o el número de inmuebles editando el nodo. Está escrito en JavaScript legible, sin dependencias.

---

## 6. Prueba antes de activar

1. **Cartera**: añade 2–3 inmuebles de prueba en `Inmuebles` (uno `Disponible`, otro `Reservado`, uno por encima del presupuesto).
2. **Alta**: pulsa *Execute Workflow*, abre el formulario y envía un lead de prueba **con tu propio email** y consentimiento marcado. Comprueba que:
   - aparece una fila en `Leads` con estado `Nuevo`, puntuación y comercial;
   - recibes el email con las referencias propuestas;
   - llega el aviso a Telegram.
3. **Sin consentimiento**: envía otro lead de prueba sin marcar la casilla. No debe salir email; en Telegram debe aparecer el aviso `sin consentimiento`.
4. **Sin coincidencias**: envía un lead de una zona/tipo que no exista en la cartera. El email debe decir que un comercial enviará una selección personalizada.
5. **Leads sin atender**: pon en una fila de prueba `Estado = Nuevo` y una `Fecha` de hace más de 24 h (formato `AAAA-MM-DD HH:MM` o `DD/MM/AAAA`). Ejecuta la parte B: debe llegar un único resumen con ese lead. Si no hay ninguno, no se envía nada.
6. Borra las filas de prueba y activa el flujo con el interruptor **Active**.

> **Importante:** el formulario de producción solo responde con el flujo **activo**. Para probar antes de activar, usa *Execute Workflow* y el formulario de test que abre n8n; o activa, prueba y vuelve a desactivar.

---

## 7. RGPD y comunicación comercial

- El formulario exige una **casilla de consentimiento** para gestionar la solicitud. No la marques por defecto ni la elimines sin revisar la base legal con tu asesor.
- El email automático **solo se envía si `Consentimiento = Sí`**. Sin consentimiento, el lead se guarda para que un comercial lo atienda por teléfono, pero no se le escribe.
- El flujo trata nombre, email, teléfono, zona y preferencias de búsqueda. Son datos identificativos y de contacto; **no trates datos de salud, ideología, religión ni otros datos del art. 9 RGPD** en el campo `Mensaje`.
- La hoja `Leads` y el chat de Telegram son accesos internos: limita quién puede verlos y no reenvíes leads a grupos ajenos al equipo.
- El email incluye una cláusula genérica de información. **Antes de usarlo en producción**, añade tu denominación social, NIF y enlace a tu política de privacidad, y revisa el texto con tu asesor legal.
- La puntuación es una priorización interna. No tomes decisiones automatizadas con efectos jurídicos o significativos sobre el interesado basándote solo en ella (art. 22 RGPD).
- Guarda los datos solo el tiempo necesario y borra los leads descartados según tu política de conservación.

**Límites del flujo:** no hace scraping de portales ni envía mensajes a plataformas que lo prohíben. Para recibir leads de portales, usa su **API oficial** o su aviso por correo y adapta la entrada del flujo (un nodo *IMAP* o *Webhook* delante de `Normalizar lead`).

**Comunicación:** el email es informativo y no debe prometer disponibilidad, financiación ni precios cerrados. Revisa que los datos de la cartera estén actualizados antes de activar la respuesta automática.

---

## 8. Problemas frecuentes

| Problema | Causa y solución |
|---|---|
| El lead no se guarda | Revisa las cabeceras de `Leads` y la credencial de Google Sheets. Deben coincidir exactamente. |
| No se propone ningún inmueble | La cartera está vacía, los inmuebles no están `Disponible`, la operación no coincide o el precio supera el presupuesto +10 %. |
| Se propone un inmueble de otra zona | La zona solo suma puntos; no descarta. Si quieres exigir coincidencia exacta de zona, añade un filtro en el nodo `Clasificar y emparejar`. |
| No sale el email | `Consentimiento` no es `Sí`, el email está vacío o el SMTP falla. Revisa *Executions* y la credencial. |
| No llega el aviso de Telegram | Chat ID incorrecto o el bot no está en el chat. Escribe al bot y recarga la lista. |
| El recordatorio de las 10:00 no llega | No hay leads `Nuevo` con más de 24 h, la fecha no se puede interpretar o el flujo está inactivo. |
| La fecha del lead no es la esperada | Configura `Europe/Madrid` en Settings → Timezone. |
| El formulario no abre desde fuera | n8n no tiene URL pública. Usa un dominio o un túnel estable. |
| Se duplican leads | El flujo no deduplica por email/teléfono. Añade un nodo de búsqueda en `Leads` antes de guardar si tu volumen lo requiere. |

---

## 9. Personalización habitual

- **Más zonas o comerciales**: amplía el objeto `AGENTES`.
- **Otro reparto**: cambia el mapa por un round-robin con una pestaña `Comerciales` (nombre + último lead asignado).
- **Más campos**: añade columnas a `Leads` y al nodo `Preparar fila del lead`.
- **Otro idioma**: edita los textos del email, del formulario y del aviso de Telegram.
- **Entrada desde portal**: añade un nodo *IMAP* (aviso por correo) o un *Webhook* (API oficial) que alimente `Normalizar lead` con los mismos nombres de campo.

---

## 10. Qué queda fuera (y por qué)

- **Valoración automática de inmuebles**: requiere datos de mercado y criterio profesional; no se automatiza aquí.
- **Firma de contratos o reservas**: fuera del alcance administrativo del flujo.
- **Scraping de portales**: prohibido por sus términos; usa API oficial o avisos por correo.
- **IA generativa**: evita coste por token y respuestas inventadas. Las reglas son auditables y gratuitas.
