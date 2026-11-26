# VentaFácil VZ

Asistente de cotizaciones por WhatsApp para comercios pequeños en Venezuela.
Agrega tus productos una vez, elige cantidades y genera un mensaje listo para
enviar con precios en **USD** y el total en **bolívares** a la tasa del BCV.

- Sin backend, sin base de datos, sin login y sin claves de API.
- Interfaz en español, pensada primero para teléfono.
- Los datos viven en `localStorage`, en el navegador de cada teléfono.

## Archivos

| Archivo | Para qué sirve |
| --- | --- |
| `index.html` | Toda la app: HTML, CSS y JavaScript en un solo archivo. |
| `manifest.json` | Nombre, colores e íconos para instalarla en la pantalla de inicio. |
| `sw.js` | Service worker que guarda la app para abrirla sin conexión. |
| `icon-*.png`, `icon.svg` | Íconos de la app. |
| `README.md` | Este documento. |

## Cómo abrirla

### En el teléfono, sin instalar nada

Copia la carpeta al teléfono (o súbela a cualquier hosting estático) y abre
`index.html` en Chrome o Safari. La app funciona, guarda los datos y consulta la
tasa. Ten en cuenta que al abrirla como archivo (`file://`) el navegador no
permite instalar el service worker, así que la app no quedará disponible offline
ni aparecerá como aplicación instalable.

### Recomendado: servirla por HTTPS o localhost

Para que funcione la instalación PWA y el modo sin conexión, la app debe
servirse por **HTTPS** (o en `localhost`). Cualquier hosting estático sirve:
GitHub Pages, Netlify, Cloudflare Pages, Vercel o un servidor propio.

Prueba local:

```sh
cd whatsappweb
python3 -m http.server 8080
```

Luego abre `http://localhost:8080` en el navegador. Para probarla desde el
teléfono en la misma red, usa la IP de tu computadora (`http://192.168.x.x:8080`);
la instalación y el service worker solo se activan en `localhost` o HTTPS.

## Instalar en la pantalla de inicio

- **Android (Chrome):** menú ⋮ → **Instalar app** (o "Añadir a pantalla de inicio").
  También aparece el botón **Instalar VentaFácil** en Ajustes cuando el navegador
  lo permite.
- **iPhone (Safari):** botón Compartir → **Añadir a pantalla de inicio**.
- En escritorio, Chrome y Edge muestran un ícono de instalación en la barra de
  direcciones.

Una vez instalada, la app abre sin barra del navegador. La primera visita debe
hacerse con conexión para que el service worker guarde la interfaz; después se
abre sin internet y usa la última tasa guardada.

## De dónde viene la tasa

Al abrir la app se consulta la tasa **oficial del BCV** en:

1. `https://ve.dolarapi.com/v1/dolares/oficial` (DolarAPI Venezuela).
2. Si esa consulta falla, se intenta `https://ve.dolarapi.com/v1/dolares` y se
   toma el registro con `fuente: "oficial"`.

Ambos endpoints son públicos, no requieren clave y permiten CORS. La app muestra
siempre:

- el valor de la tasa (Bs por dólar),
- la fecha de actualización que reporta la fuente,
- la fuente usada,
- y un aviso si la tasa tiene **más de 24 horas** o si no se pudo consultar.

Si no hay conexión o la consulta falla, se conserva la última tasa guardada. Si
nunca hubo una, ve a **Ajustes → Tasa BCV** e ingresa la tasa manualmente. La
tasa manual se guarda con la fecha del momento y se identifica como
"Tasa manual" en el mensaje.

> La tasa es una referencia. Confirma siempre el valor oficial del BCV antes de
> cerrar una venta.

## Uso rápido

1. **Catálogo:** agrega nombre, precio en USD, stock y categoría. El stock vacío
   significa "sin límite".
2. **Cotizar:** escribe el nombre del cliente, busca productos y usa **+ / −**
   para ajustar cantidades. Los totales en USD y Bs se actualizan al instante.
3. **Ajustes → Datos de pago:** activa Pago Móvil, Transferencia, Zelle o USDT y
   completa banco, teléfono/correo, cuenta y titular.
4. **Enviar por WhatsApp:** abre WhatsApp con el mensaje armado para que elijas
   el chat del cliente. Si WhatsApp no está instalado, usa **Copiar mensaje**.

El mensaje incluye cada línea con cantidad y precio, subtotal y total en USD,
total en Bs, la tasa usada con su fecha y fuente, y las formas de pago activas.

## Recibo PDF (factura)

En el detalle de la cotización, junto a **Enviar por WhatsApp**, está el botón
**Recibo PDF**. Abre un formulario para elegir cliente, método de pago (Pago Móvil,
Transferencia, Zelle, USDT, Efectivo $ o Efectivo Bs), referencia, banco (solo para
Pago Móvil y Transferencia) y fecha. Al generar, descarga un PDF A4 con el
encabezado, la tasa BCV aplicada con su fecha y fuente, la tabla de productos
(cantidad, precio unitario en USD, total en USD y total en Bs), los totales y los
datos de pago.

- Nombre del archivo: `Recibo-VentaFacil-DDMMYY.pdf`.
- El método de pago elegido se recuerda para la próxima vez.
- La opción **Limpiar carrito después de generar** vacía la cotización al terminar.
- Al terminar se abre el diálogo **Recibo generado** con estas acciones:
  - **📤 Compartir con…** (solo si el navegador permite adjuntar archivos, p. ej.
    Chrome en Android): abre la hoja de compartir del sistema con el PDF adjunto;
    al elegir WhatsApp el archivo se envía listo.
  - **📥 Descargar PDF**: vuelve a descargar el recibo (por si se perdió).
  - **📋 Copiar mensaje**: copia un texto corto para pegar en el chat junto al PDF.
  - **💬 Abrir WhatsApp**: abre `wa.me` con el resumen escrito. WhatsApp no permite
    adjuntar archivos por enlace, así que el PDF se adjunta a mano.
- En iOS Safari no existe compartir archivos: la app muestra el aviso
  “En iOS: descarga el PDF, luego adjúntalo en WhatsApp manualmente”.
- El PDF se arma con jsPDF (CDN). Sin conexión, la app avisa
  “jsPDF no disponible sin conexión” y no genera el archivo.

## Catálogo CSV

Columnas: `nombre,precio_usd,stock,categoria`.

- Se aceptan comas o punto y coma como separador.
- La coma decimal también se acepta (`1,20`).
- La primera fila puede ser encabezado o no.
- Al importar, los productos se combinan por nombre: si existe se actualiza, si
  no se agrega.
- La exportación incluye BOM para que Excel abra bien los acentos.

Ejemplo:

```csv
nombre,precio_usd,stock,categoria
Harina PAN 1 kg,1.20,24,Alimentos
Café Fama de América 500 g,3.80,9,Alimentos
```

## Datos y respaldo

- Todo se guarda en `localStorage`: productos, última tasa, cliente y datos de
  pago. No viaja a ningún servidor.
- **Restablecer demo** reemplaza el catálogo con 8 productos de ejemplo y vacía
  la cotización; no borra la tasa ni los datos de pago.
- Exporta el catálogo a CSV antes de cambiar de teléfono o borrar datos del
  navegador. Los datos de pago no se exportan.

## Límites de esta versión

- No hay sincronización entre dispositivos ni respaldo en la nube.
- No descuenta stock automáticamente al enviar una cotización.
- No envía mensajes por sí sola: abre WhatsApp para que confirmes el chat y el
  envío.
- No hay cuentas de usuario ni historial de ventas.
