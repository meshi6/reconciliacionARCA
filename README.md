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

Abrí una conversación con una IA que pueda leer archivos y ejecutar código (por ejemplo, Claude). Subí tus archivos y pegá las [instrucciones para la IA](#instrucciones-para-la-ia) que están al final.

### 3. Validá lo que entendió

Antes de conciliar, la IA te va a mostrar qué entendió de tus archivos: qué columna es cuál, la fecha de corte, si hay ventas sin costo o comprobantes de otra persona. Corregila si algo está mal. De eso depende todo lo demás.

### 4. Revisá el resultado

Recibís un Excel nuevo; tus archivos no se tocan. La pestaña **Discrepancias** tiene un hallazgo por fila, ordenado por gravedad (🔴 alta, 🟡 media, 🟢 baja), y cada uno dice **qué hacer**: qué registro, qué campo, qué valor. Las coincidencias **aproximadas** vienen resaltadas para que decidas vos.

### Probalo con el ejemplo

La carpeta [`ejemplo/`](ejemplo) tiene un negocio inventado, con tres servicios de distintos precios y cobro online. Todos los datos son ficticios.

| Archivo | Qué tiene |
|---|---|
| [`comprobantes.xlsx`](ejemplo/comprobantes.xlsx) | 15 comprobantes de marzo y abril de 2026: 14 facturas C y una nota de crédito. Columnas parecidas a las de *Mis Comprobantes*, más la referencia del pago |
| [`ventas.xlsx`](ejemplo/ventas.xlsx) | 17 ventas con los problemas de siempre: referencias guardadas como número, importes como texto, una referencia repetida, otra mal copiada, ventas sin costo, pendientes y anteriores al corte |
| [`precios.xlsx`](ejemplo/precios.xlsx) | Lista de precios, con un aumento en abril, y un código de descuento |
| [`resultado-esperado.xlsx`](ejemplo/resultado-esperado.xlsx) | Lo que la IA tendría que encontrar: 9 hallazgos (3 🔴, 3 🟡, 3 🟢), las coincidencias con su criterio y los totales |

Subí los tres primeros con las instrucciones y compará lo que te devuelve con `resultado-esperado.xlsx`.

### 🔒 Privacidad

Tus archivos tienen nombres, CUIT, domicilios y mails de tus clientes. Antes de subirlos a una IA, fijate qué hace esa herramienta con los datos. No los subas a un repositorio y, si compartís el método, usá ejemplos inventados.

### Instrucciones para la IA

Copialas con el botón de copiar del bloque y pegalas en la conversación.

```markdown
# Instrucciones: conciliación de comprobantes ARCA con ventas

Vas a conciliar los comprobantes que la persona emitió en ARCA (facturas y notas de crédito) con sus registros de ventas. El objetivo es encontrar ventas cobradas sin factura, facturas sin venta y errores de carga, **sin falsas alarmas**.

## Criterios generales

- Leé y conciliá con código, nunca a ojo.
- No edites los archivos originales. El resultado va en un `.xlsx` **nuevo**.
- No inventes datos. Si una coincidencia es una suposición, marcala como tal.
- No muestres en el chat más datos personales (nombres, CUIT, mails) de los necesarios.
- Los importes vienen en formato argentino: punto de miles, coma decimal.

## 1. Antes de conciliar, validá con la persona

Leé los archivos y mostrale a la persona, en una lista corta, lo que entendiste. Esperá su validación antes de seguir:

- Qué archivo son los comprobantes y cuál las ventas, y qué columna es cuál: fecha, cliente, CUIT/DNI, importe, estado de pago, **referencia del pago** (número de operación, pedido o transacción).
- **Fecha de corte:** la del primer comprobante emitido. Las ventas anteriores quedan afuera y *no* cuentan como «factura faltante».
- Qué valor indica que una venta está **cobrada**.
- Si hay **ventas sin costo** (bonificadas, cortesías): no llevan factura; se listan, pero no se marcan.
- Si hay **comprobantes de otro contribuyente** mezclados: van aparte y no se concilian contra estas ventas.

Anotá el total de cada grupo: los vas a necesitar para cerrar los totales.

## 2. Normalizá

Casi todos los errores salen de este paso.

- **Referencias como texto de dígitos.** Una planilla guarda `172604918356.0` y la exportación `"172604918356"`: comparadas así no coincide **ninguna**. Pasá todo a texto, sacá el `.0` y todo lo que no sea dígito.
- **CUIT/DNI** sólo con dígitos.
- **Importes** como número.
- **Nombres:** minúscula, sin tildes, sin espacios dobles, y «APELLIDO, NOMBRE» como conjunto de palabras.
- **Mails** en minúscula y **fechas** como fecha.

## 3. Conciliá, en este orden

Una factura que ya matcheó sale de la lista y no puede matchear con otra venta. En cada coincidencia anotá con qué criterio se encontró.

1. **Misma referencia de pago.** Es prueba.
2. **Referencia con un dígito de menos** (`73958204617` contra `173958204617`). Es coincidencia **y además** error de carga a corregir.
3. **Mismo CUIT/DNI + importe exacto.**
4. **Nombre + importe exacto.** Al menos **dos palabras** del nombre de la venta tienen que estar en el de la factura. No exijas el nombre entero (los segundos nombres lo rompen) ni concilies sólo por nombre (un apellido común más un precio común junta a dos personas distintas).
5. **Consumidor final sin identificar.** Sólo por referencia de pago. Si nada más coinciden fecha e importe, y ese importe aparece una sola vez en el período, es **aproximado**: va como suposición, no como dato, y resaltado con otro color para que la persona decida.

## 4. 🗂️ Clasificá lo que sobra

| Qué sobra | Gravedad |
|---|---|
| Una misma referencia de pago en ventas de **dos clientes distintos** | 🔴 **Alta**: casi siempre a uno le pisaron el dato y aparece también como cobrado sin factura |
| Venta cobrada sin factura | 🔴 **Alta** |
| Factura sin venta, por un importe que no está en la lista de precios | 🟡 **Media**: que busque la referencia en su medio de cobro |
| Factura a un nombre que no está en las ventas | 🟡 **Media**: suele ser alguien que pagó por otra persona |
| Consumidor final conciliado sólo por fecha e importe | 🟡 **Media** |
| Referencia vacía o mal copiada en las ventas | 🟢 **Baja**: indicá el valor correcto |
| Nota de crédito seguida de una factura reemitida | 🟢 **Baja**: anotala para que nadie la vuelva a levantar |

**No marques como error:** dos facturas a la misma persona (suelen ser dos compras: mirá productos y fechas), una nota de crédito más una factura nueva (es una corrección bien hecha), ni un mail de pago distinto al de la venta (es la misma persona con otra cuenta).

## 5. Cerrá los totales

Cada comprobante tiene que caer en **un solo grupo**: conciliado, de otro contribuyente, sin identificar o nota de crédito. La **cantidad** y el **importe** tienen que dar igual que la exportación; si no, hay uno contado dos veces o uno perdido. Encontralo antes de entregar. Hacé lo mismo al revés: cada venta cobrada dentro del alcance está conciliada, es sin costo o está marcada.

## 6. Entregá

- Un `.xlsx` nuevo con:
  - **Discrepancias:** un hallazgo por fila (gravedad, qué pasa, cliente, importe, detalle con comprobante, referencia y fecha, y **qué hacer**), ordenado por gravedad. Cada «qué hacer» es concreto: qué registro, qué campo, qué valor.
  - Las pestañas de apoyo que hagan falta: por ejemplo, las coincidencias con el criterio de cada una, o los comprobantes de otro contribuyente.
  - **Log:** qué encontraste y por qué el resultado da lo que da.
- En el chat, un resumen corto: conteos por grupo, si los totales cierran y cuántos hallazgos hay de cada gravedad.
- Si no podés generar archivos, entregá la tabla de Discrepancias en el chat.

## 📋 Validación antes de entregar

- [ ] Originales sin tocar
- [ ] Columnas, fecha de corte, ventas sin costo y otros contribuyentes validados con la persona
- [ ] Referencias y CUIT como dígitos, importes como número, nombres sin tildes
- [ ] Conciliación en orden: referencia → referencia sin un dígito → CUIT + importe → nombre + importe → consumidor final
- [ ] Cada coincidencia con su criterio; las suposiciones marcadas como tales
- [ ] Cada comprobante en un solo grupo; cantidad y total iguales a la exportación
- [ ] Cada hallazgo con un próximo paso concreto
```
