# Conciliación Facturas ↔ Ventas

Un método para conciliar los comprobantes que emitiste en ARCA con tus propios registros de ventas, sea una planilla o un sistema. Sirve para encontrar ventas cobradas sin factura, facturas sin venta y errores de carga, sin falsas alarmas. Sin importar qué vendés, dónde y cómo cobrás. Este ejemplo tiene varios servicios de distintos precios y un punto de venta online.

La conciliación la hace una IA. Vos juntás los archivos, validás lo que entendió la IA y revisás el resultado.

### 1. Juntá los archivos

| Input | Qué es | Datos clave |
|---|---|---|
| **Comprobantes emitidos** | La exportación de *Mis Comprobantes* de ARCA, o la de tu facturador, para el período: facturas *y* notas de crédito | fecha, tipo, número, CUIT/DNI y nombre del receptor, total y, si tu facturador lo guarda, la **referencia del pago** |
| **Tus ventas** | Tu planilla o sistema de ventas, incluidos los períodos anteriores | fecha, cliente, CUIT/DNI o mail, producto o servicio, estado de pago, importe, **referencia del pago** |
| **Lista de precios** | Cuánto salía cada cosa en cada momento, con descuentos | sirve para ver si un importe que nadie explica parece una venta real |

La referencia del pago es el número de operación, pedido o transacción que da el medio de cobro y un dato clave para la conciliación.

### 2. Dáselos a la IA

Abrí una conversación con una IA que pueda leer archivos y ejecutar código (por ejemplo, Claude). Subí tus archivos y [`instrucciones.md`](instrucciones.md), y escribí:

> Conciliá estos comprobantes con mis ventas siguiendo instrucciones.md.

### 3. Validá lo que entendió

Antes de conciliar, la IA te va a mostrar qué entendió de tus archivos: qué columna es cuál, la fecha de corte, si hay ventas sin costo o comprobantes de otra persona. Corregila si algo está mal. De eso depende todo lo demás.

### 4. Revisá el resultado

Recibís un Excel nuevo; tus archivos no se tocan. La pestaña **Discrepancias** tiene un hallazgo por fila, ordenado por gravedad (🔴 alta, 🟡 media, 🟢 baja), y cada uno dice **qué hacer**: qué registro, qué campo, qué valor. Las coincidencias **aproximadas** vienen marcadas como tales: esas decidilas vos.

### 🔒 Privacidad

Tus archivos tienen nombres, CUIT, domicilios y mails de tus clientes. Antes de subirlos a una IA, fijate qué hace esa herramienta con los datos. No los subas a un repositorio y, si compartís el método, usá ejemplos inventados.
