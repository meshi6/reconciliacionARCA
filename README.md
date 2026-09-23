# Conciliación Facturas ↔ Ventas

Un método paso a paso para conciliar lo que exporta un sistema de facturación contra las planillas donde se registran las ventas o inscripciones. La idea es encontrar toda venta cobrada sin factura, toda factura sin venta y todo error de carga entre las dos, sin que se cuelen falsas alarmas.

Se armó para un negocio de cursos que factura con **Facturante**, cobra por **Mercado Pago** y lleva las inscripciones en Excel. Los pasos sirven para cualquier facturador, medio de cobro o planilla de ventas. Lo único que cambia son los nombres de las columnas.

---

## 1. Qué necesitás

| Insumo | Qué es | Columnas clave |
|---|---|---|
| **Exportación de comprobantes** | Todos los comprobantes emitidos en el período (facturas *y* notas de crédito) | fecha, razón social, CUIT/DNI, tipo (`FC` factura / `NCC` nota de crédito), número, total, **número de operación del pago**, observaciones |
| **Planilla de ventas actual** | Inscripciones de los cursos o cohortes en curso | fecha, nombre, apellido, mail, producto/nivel, estado de pago, importe, **operación**, mail de pago |
| **Planilla(s) histórica(s)** | Inscripciones de cohortes anteriores, muchas veces una pestaña por cohorte | lo mismo, quizá con otros nombres de columna en cada pestaña |
| **PDF de las facturas** *(opcional)* | Los comprobantes uno por uno | suelen traer el mail de la clienta, que la exportación a veces no trae |
| **Lista de precios** | Cuánto salía cada producto en cada momento, con los códigos de descuento | sirve para ver si un importe que nadie explica parece una venta real |

> **Particularidades de Facturante:** el «Informe» se descarga como `.xls`, pero en realidad es XML Spreadsheet 2003. Leé el XML, o abrilo y volvelo a guardar en Excel o LibreOffice. El número de operación de Mercado Pago está en `NroOrdenCompra` y también dentro de `Observaciones` («…operación de MercadoPago con ID 168420135790…»). En una nota de crédito, `Observaciones` dice qué factura anula.

No toques nunca los originales. Trabajá sobre copias y generá el resultado en un archivo **nuevo**.

---

## 2. Definí el alcance antes de comparar

1. **Buscá la fecha de corte**, que es la del primer comprobante del sistema de facturación. Un pago anterior no puede tener factura porque el sistema todavía no existía. Esos pagos quedan afuera del todo: *no* son «facturas faltantes». (En la conciliación original, contarlos habría inventado un problema de millones en 200 filas.)
2. **Quedate sólo con las filas pagas** de las planillas (estado ✅ / «Pagado» / tiene un importe cobrado).
3. **Separá las inscripciones sin costo.** Las filas en $0 (becas, equipo, invitadas) no llevan factura. Listalas, pero no las marques como problema.
4. **Separá a los otros contribuyentes.** Si en la misma cuenta de facturación, o en la misma exportación, hay comprobantes de otra persona (por ejemplo, una socia que factura por su cuenta), mandalos a una lista aparte. Se concilian contra los registros de esa persona, no contra las inscripciones.

Anotá los conteos en este punto. Los vas a necesitar para cerrar los totales al final.

---

## 3. Normalizá antes de conciliar

Casi todos los resultados equivocados salen de este paso. Hacelo completo:

- **Operaciones como texto, sólo con dígitos.** Las planillas suelen guardarlas como número (`172604918356.0`) y la exportación como texto (`"172604918356"`). Si las comparás así, no coincide **ninguna**. Pasá las dos a texto, sacá el `.0` final y quitá los espacios y todo lo que no sea dígito.
- **Importes como número.** Sacá el signo pesos, los puntos de miles y el «ARS». Ojo con el punto y la coma como separador decimal.
- **Nombres:** pasalos a minúscula, sacá las tildes (`á→a`, `ñ→n`), juntá los espacios dobles y convertí «APELLIDO, NOMBRE SEGUNDO» en un conjunto de palabras. En la factura suele ir el nombre legal completo. En la planilla va lo que la persona escribió («Ana Pérez» contra «PÉREZ, ANA MARÍA»).
- **Mails:** en minúscula y sin espacios.
- **Fechas:** fechas de verdad, nunca texto.

---

## 4. Conciliá, en este orden

Aplicá las reglas en orden. Una factura que ya matcheó sale de la lista y no puede matchear con una segunda venta.

1. **Misma operación.** Es la señal más fuerte. Si el número de operación coincide, es un dato.
2. **Operación a la que le falta un dígito.** La planilla tiene el número sin el primer dígito (`73958204617` contra `173958204617`) o algo muy parecido. Tomalo como coincidencia **y además** anotalo como error de carga para corregir en la planilla.
3. **Nombre + importe exacto.** Por lo menos **dos palabras** del nombre de la planilla tienen que aparecer en el de la factura, **y** los importes tienen que ser iguales. Los apodos se aceptan si son el comienzo del nombre (`san` coincide con `sandra`).
   - No pidas que el nombre coincida entero. Los segundos nombres lo rompen (en la conciliación original aparecieron 12 falsos «sin factura»).
   - No concilies sólo por nombre. Un apellido compartido más un precio común junta a dos personas distintas. El importe es lo que lo evita.
4. **Cliente genérico o anónimo.** Algunas facturas no tienen el nombre de quien compró. Facturante hace esto con los pagos del bot de Mercado Pago («Cliente Genérico BOT de MercadoPago», DNI 1). Esas conciliálas **sólo por operación**, contra todas las planillas, incluidas las históricas.
   - Si la operación coincide: **confirmado**.
   - Si sólo coinciden la fecha y el importe, y ese importe aparece una sola vez en todo el período: **aproximado**. Va a la lista, pero **dejá claro que es una suposición** (una columna aparte, otro color). No lo informes como dato.
   - Si no coincide nada: **sin identificar**.

En cada coincidencia anotá qué regla la encontró (`operación`, `operación sin un dígito`, `nombre+importe`, `genérico por operación`, `aproximado`). Quien lea el resultado tiene que poder ver qué tan firme es cada una.

---

## 5. Clasificá lo que sobra

| Qué sobra | Etiqueta | Gravedad |
|---|---|---|
| La misma operación en las filas de **dos personas distintas** | Operación repetida en la planilla | **Alta**: un pago no puede ser de dos personas. Casi siempre a una le pisaron la operación, y esa persona aparece además como «cobrada sin factura». |
| Una venta paga sin factura después de aplicar todas las reglas | Cobrada sin factura | **Alta** |
| Una factura sin venta en ningún lado, por un importe que no está en la lista de precios | Factura sin dueño | **Media**. Buscá la operación en Mercado Pago. |
| Una factura con un nombre que no está en ninguna planilla | Factura sin inscripción | **Media**. Muchas veces es alguien que pagó por otra persona. Buscá el nombre en las notas de la planilla. |
| Una factura a cliente genérico que sólo coincide por fecha e importe | Probablemente facturada como genérico | **Media** |
| Factura conciliada, pero la celda de operación de la planilla está vacía | Operación vacía en la planilla | **Baja**. Poné el valor exacto a cargar. |
| Operación a la que le falta un dígito | Operación mal copiada | **Baja**. Poné el valor corregido. |
| Nota de crédito seguida de una factura reemitida sobre la misma operación | Nota de crédito ya explicada | **Baja**. Dejala anotada para que nadie la vuelva a levantar. |

### Antes de marcar, fijate

En la conciliación original, estos casos parecían errores y no lo eran:

- **Dos facturas a la misma persona** suelen ser dos compras (por ejemplo, el nivel 1 en agosto y el nivel 2 en septiembre). Mirá productos y fechas antes de hablar de doble facturación.
- **Nota de crédito + dos facturas sobre una misma operación:** la primera factura estaba mal (por ejemplo, no se aplicó un código de descuento), se anuló y se volvió a emitir. Es una corrección bien hecha.
- **El mail de la factura no coincide con el de la inscripción:** la gente paga con el mail que tiene en su cuenta de Mercado Pago. Es la misma persona. No es un error, pero complica cualquier búsqueda por mail (listas de envío, listas de descuento), así que conviene listar esos casos aparte.

---

## 6. Cerrá los totales

Antes de confiar en el resultado, **cada comprobante de la exportación tiene que estar en un solo grupo**:

```
comprobantes en la exportación =
    conciliados con una venta actual
  + conciliados con una venta histórica
  + cliente genérico identificado por operación
  + comprobantes de otro contribuyente
  + sin identificar / sin dueño
  + notas de crédito
```

La **cantidad** y el **importe** tienen que dar lo mismo que informa el sistema de facturación. Si no dan, hay un comprobante contado dos veces o uno que se perdió. Encontralo antes de informar nada.

Hacé lo mismo al revés con las ventas pagas dentro del alcance: cada una tiene que estar conciliada, ser sin costo o estar marcada.

---

## 7. El archivo de salida

Un `.xlsx` nuevo, con una pestaña por tema:

| Pestaña | Contenido |
|---|---|
| **Discrepancias** | Una fila por hallazgo: Gravedad · Qué pasa · Quién · Producto · Importe · Detalle (comprobante, operación, fecha) · **Qué hacer**. Ordenada por gravedad. Abajo, un resumen con `CONTAR.SI` por gravedad. |
| **Cohortes anteriores** | Las ventas históricas dentro del alcance, cada una con su número de comprobante (o `SIN FACTURA` / `sin costo`) y la regla con que se concilió. |
| **Cliente genérico** | Cada factura sin nombre: comprobante, importe, operación, de quién es, cómo se supo y **certeza** (Confirmado / Aproximado / Sin datos). |
| **Mail de pago distinto** | Comprobante · razón social · mail con el que pagó · mail en la planilla · importe. |
| **Otro contribuyente** | Los comprobantes que son de otra persona, listados para que los totales cierren. |
| **Cómo se comparó** | Fecha de corte, conteos por grupo, reglas de conciliación en orden, excepciones conocidas. |
| **Log** | Notas con fecha de lo que se fue encontrando y **por qué el resultado da lo que da**, incluidos los errores del propio método. Así quien venga después no rehace el análisis. |

Pautas:
- Cada «Qué hacer» tiene que ser concreto: qué planilla, qué celda, qué valor, o en qué sistema buscar.
- Los importes van como número, no como texto, y los negativos entre paréntesis.
- Un **«Detalle»** que cualquiera pueda chequear en 30 segundos sin tener que preguntarte.
- Recalculá el archivo (LibreOffice headless o Excel) y confirmá que no hay errores de fórmula.

---

## 8. Privacidad

Los originales y el resultado tienen nombres, CUIT, domicilios y mails. Dejalos **fuera del control de versiones** (`.gitignore` para las carpetas de originales, el archivo de salida y `*.pdf`). Versioná sólo el método, o sea esta guía y los scripts que haya. Si compartís el método, usá ejemplos inventados.

---

## Checklist

- [ ] Originales copiados, nunca editados
- [ ] Corte = fecha del primer comprobante; pagos anteriores afuera
- [ ] Inscripciones sin costo y otros contribuyentes separados
- [ ] Operaciones como texto de dígitos (sin `.0`), importes como número, nombres sin tildes
- [ ] Conciliación en orden: operación → operación sin un dígito → dos palabras del nombre + importe exacto → cliente genérico por operación
- [ ] Cada coincidencia con su regla; las suposiciones marcadas como suposiciones
- [ ] Revisadas las operaciones repetidas entre personas distintas
- [ ] «Dobles facturas» y notas de crédito explicadas antes de marcarlas
- [ ] Cada comprobante en un solo grupo; cantidad y total iguales a la exportación
- [ ] Cada hallazgo con un próximo paso concreto
- [ ] Archivo recalculado sin errores; datos personales fuera de git
