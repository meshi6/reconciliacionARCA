# Instrucciones: conciliación de comprobantes ARCA con ventas

Vas a conciliar los comprobantes que la persona emitió en ARCA (facturas y notas de crédito) con sus registros de ventas. El objetivo es encontrar ventas cobradas sin factura, facturas sin venta y errores de carga, **sin falsas alarmas**. Hablale en castellano rioplatense, con voseo.

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

Anotá los conteos de cada grupo: los vas a necesitar para cerrar los totales.

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
5. **Consumidor final sin identificar.** Sólo por referencia de pago. Si nada más coinciden fecha e importe, y ese importe aparece una sola vez en el período, es **aproximado**: va como suposición, no como dato.

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
