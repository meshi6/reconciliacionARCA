# Conciliación Facturas ↔ Ventas

Un método para conciliar los comprobantes que emitiste en ARCA con tus propios registros de ventas, sea una planilla o un sistema. Sirve para encontrar ventas cobradas sin factura, facturas sin venta y errores de carga, sin falsas alarmas. Sin importar qué vendés, dónde y cómo cobrás. Este ejemplo tiene varios servicios de distintos precios y un punto de venta online.

---

## 1. Qué necesitás

| Input | Qué es | Datos clave |
|---|---|---|
| **Comprobantes emitidos** | La exportación de *Mis Comprobantes* de ARCA, o la de tu facturador, para el período: facturas *y* notas de crédito | fecha, tipo, número, CUIT/DNI y nombre del receptor, total y, si tu facturador lo guarda, la **referencia del pago** |
| **Tus ventas** | Tu planilla o sistema de ventas, incluidos los períodos anteriores | fecha, cliente, CUIT/DNI o mail, producto o servicio, estado de pago, importe, **referencia del pago** |
| **Lista de precios** | Cuánto salía cada cosa en cada momento, con descuentos | sirve para ver si un importe que nadie explica parece una venta real |

La referencia del pago es el número de operación, pedido o transacción que da el medio de cobro y un dato clave para la conciliación.

No edites los originales. Trabajá sobre copias y dejá el resultado en un archivo **nuevo**.

---

## 2. Definí el alcance

1. **Fecha de corte:** la del primer comprobante emitido. Una venta anterior no puede tener factura en este período; queda afuera y *no* cuenta como «factura faltante».
2. **Sólo ventas cobradas.**
3. **Separá las ventas sin costo** (bonificadas, cortesías): no llevan factura. Listalas, pero no las marques.
4. **Separá los comprobantes de otro contribuyente**, si la exportación los mezcla. Se concilian contra los registros de esa persona.

Anotá los conteos: los vas a necesitar para cerrar los totales.

---

## 3. Normalizá antes de conciliar

Casi todos los errores salen de este paso.

- **Referencias como texto de dígitos.** Una planilla guarda `172604918356.0`, la exportación `"172604918356"`: comparadas así no coincide **ninguna**. Pasá todo a texto, sacá el `.0` y lo que no sea dígito.
- **CUIT/DNI sólo con dígitos**, sin guiones ni puntos.
- **Importes como número:** sin `$`, sin puntos de miles; ojo con la coma decimal.
- **Nombres:** minúscula, sin tildes, sin espacios dobles, y «APELLIDO, NOMBRE» como conjunto de palabras.
- **Mails** en minúscula y **fechas** como fecha, nunca texto.

---

## 4. Conciliá, en este orden

Una factura que ya matcheó sale de la lista y no puede matchear con otra venta.

1. **Misma referencia de pago.** Es prueba.
2. **Referencia con un dígito de menos** (`73958204617` contra `173958204617`). Es coincidencia **y además** error de carga a corregir.
3. **Mismo CUIT/DNI + importe exacto.**
4. **Nombre + importe exacto.** Al menos **dos palabras** del nombre de tu registro tienen que estar en el de la factura. No pidas el nombre entero (los segundos nombres lo rompen) ni concilies sólo por nombre (un apellido común más un precio común junta a dos personas distintas).
5. **Consumidor final sin identificar.** Sólo por referencia de pago. Si nada más coinciden fecha e importe, y ese importe aparece una sola vez en el período, marcalo **aproximado** y mostralo como suposición, no como dato.

Anotá en cada coincidencia qué regla la encontró, para que se vea qué tan firme es.

---

## 5. Clasificá lo que sobra

| Qué sobra | Gravedad |
|---|---|
| Una misma referencia de pago en ventas de **dos clientes distintos** | **Alta**: casi siempre a uno le pisaron el dato y aparece también como cobrado sin factura |
| Venta cobrada sin factura | **Alta** |
| Factura sin venta, por un importe que no está en la lista de precios | **Media**: buscá la referencia en tu medio de cobro |
| Factura a un nombre que no está en tus registros | **Media**: suele ser alguien que pagó por otra persona |
| Consumidor final conciliado sólo por fecha e importe | **Media** |
| Referencia vacía o mal copiada en tus registros | **Baja**: indicá el valor correcto |
| Nota de crédito seguida de una factura reemitida | **Baja**: anotala para que nadie la vuelva a levantar |

**Antes de marcar, fijate:** dos facturas a la misma persona suelen ser dos compras; una nota de crédito más una factura nueva es una corrección bien hecha; y un mail de pago distinto al de tu registro es la misma persona con otra cuenta.

---

## 6. Cerrá los totales

Cada comprobante tiene que caer en **un solo grupo**: conciliado, de otro contribuyente, sin identificar o nota de crédito. La **cantidad** y el **importe** tienen que dar igual que la exportación; si no, hay uno contado dos veces o uno perdido. Hacé lo mismo al revés con las ventas cobradas: cada una conciliada, sin costo o marcada.

---

## 7. El resultado

Un `.xlsx` nuevo con una pestaña **Discrepancias** (una fila por hallazgo: gravedad, qué pasa, cliente, importe, detalle y **qué hacer**, ordenada por gravedad), las pestañas de apoyo que hagan falta, y un **Log** con qué se encontró y por qué el resultado da lo que da. Cada «qué hacer» tiene que ser concreto: qué registro, qué campo, qué valor.

---

## 8. Privacidad

Los originales y el resultado tienen nombres, CUIT, domicilios y mails. Dejalos **fuera del control de versiones** y, si compartís el método, usá ejemplos inventados.

---

## Checklist

- [ ] Originales copiados, nunca editados
- [ ] Corte en el primer comprobante; ventas sin costo y otros contribuyentes separados
- [ ] Referencias y CUIT como dígitos, importes como número, nombres sin tildes
- [ ] Conciliación en orden: referencia → referencia sin un dígito → CUIT + importe → nombre + importe → consumidor final
- [ ] Cada coincidencia con su regla; las suposiciones marcadas como tales
- [ ] Cada comprobante en un solo grupo; cantidad y total iguales a la exportación
- [ ] Cada hallazgo con un próximo paso concreto
