# Conciliación Facturas ↔ Ventas

Un método para conciliar los comprobantes que emitiste en ARCA 🇦🇷 con tus propios registros de ventas, sea una planilla o un sistema. Sirve para encontrar ventas cobradas sin factura, facturas sin venta y errores de carga, sin falsas alarmas. Sin importar qué vendés, dónde y cómo cobrás. Este ejemplo tiene varios servicios de distintos precios y un punto de venta online.

La conciliación la hace Claude y se trabaja en local: tus archivos, la skill, la memoria y los resultados quedan en una carpeta de tu computadora, sin subir nada a un chat. Claude sí envía a Anthropic lo que lee para poder procesarlo. Vos juntás los archivos, validás lo que entendió y revisás el resultado.

<img width="1056" height="900" alt="arquitectura_conciliacion_arca_es_v2" src="https://github.com/user-attachments/assets/942c27d4-ce76-4937-9054-e85107779925" />

### 1. Juntá los archivos

| Input | Qué es | Datos clave |
|---|---|---|
| **Comprobantes emitidos** | La exportación de *Mis Comprobantes* de ARCA, o la de tu facturador, para el período: facturas *y* notas de crédito | fecha, tipo, número, CUIT/DNI y nombre del receptor, total y, si tu facturador lo guarda, la **referencia del pago** |
| **Tus ventas** | Tu planilla o sistema de ventas, incluidos los períodos anteriores | fecha, cliente, CUIT/DNI o mail, producto o servicio, estado de pago, importe, **referencia del pago** |
| **Lista de precios** | Cuánto salía cada cosa en cada momento, con descuentos | sirve para ver si un importe que nadie explica parece una venta real |

La referencia del pago es el número de operación, pedido o transacción que da el medio de cobro y un dato clave para la conciliación.

### 2. Dáselos a Claude

1. Creá una carpeta en tu computadora, por ejemplo `conciliacion`, y poné adentro tus archivos. Copiá también la carpeta [`ejemplo/`](ejemplo) de este repositorio: Claude la usa para probar el método cada vez que lo cambia.
2. Guardá el [SKILL.md](#skillmd) que está al final en `conciliacion/.claude/skills/conciliacion-arca/SKILL.md`. La carpeta `.claude` queda oculta; en la Mac, Cmd+Shift+. la muestra en el Finder.
3. Abrí la app de escritorio de Claude, entrá en **Code** y elegí la carpeta `conciliacion` como proyecto.
4. Escribí: «Conciliá mis comprobantes con mis ventas».

Claude trabaja sobre los archivos de tu computadora y deja el resultado en la misma carpeta. 🧉 🥐

### 3. Validá lo que entendió

Antes de conciliar, Claude te va a mostrar qué entendió de tus archivos: qué columna es cuál, la fecha de corte, si hay ventas sin costo o comprobantes de otra persona. ⚠️ Corregilo si algo está mal. De eso depende todo lo demás.

### 4. Revisá el resultado

Recibís un Excel nuevo. La pestaña **Discrepancias** tiene un hallazgo por fila, ordenado por gravedad (🔴 alta, 🟡 media, 🟢 baja), y cada uno dice **qué hacer**: qué registro, qué campo, qué valor. Las coincidencias **aproximadas** vienen resaltadas para que decidas vos.

Revisá también algunas de la pestaña **Coincidencias**, que viene ordenada de la más débil a la más firme. Las encontradas por CUIT/DNI + importe o por nombre + importe pueden juntar a dos personas distintas, y un error ahí esconde una discrepancia real. Las encontradas por referencia no hace falta revisarlas.

### 5. Corregí, volvé a ejecutar y enseñale

1. **Corregí tus registros** siguiendo la columna **qué hacer**: cargá las referencias que faltan, emití las facturas pendientes.
2. **Contale a Claude lo que decidiste** sobre las aproximadas y sobre lo que no era un error. Por ejemplo: «La factura 6 es de Julián Castro, confirmado» o «Marta Benítez paga las ventas de Nicolás Benítez».
3. **Volvé a ejecutar la conciliación** con los archivos actualizados. Ejecutá la conciliación completa, no sólo lo que corregiste: un cambio puede mover otras coincidencias y los totales tienen que cerrar sobre todo el período. Claude la compara con la anterior y te dice qué se resolvió, qué sigue y qué es nuevo.

Cada vez que le enseñás algo, Claude te propone dónde guardarlo y lo guarda sólo cuando lo validás:

- **Lo propio de tu negocio** (qué columna es la referencia, un cliente que paga por otro, un caso que no es error) va al `CLAUDE.md` de la carpeta, que Claude lee cada vez que abrís el proyecto.
- **Lo que cambia el método** (un criterio nuevo, un tipo de falsa alarma) va al `SKILL.md`. Antes de guardarlo, Claude vuelve a ejecutar el ejemplo y lo compara con `resultado-esperado.xlsx`. Si el resultado cambia sin que debiera, el cambio no se guarda. Para guardarlo, Claude te va a pedir permiso para editar el `SKILL.md`: aceptalo.

Así, cada ejecución marca menos falsas alarmas que la anterior.

### Probalo con el ejemplo

La carpeta [`ejemplo/`](ejemplo) tiene un negocio inventado, con tres servicios de distintos precios y cobro online. Todos los datos son ficticios.

| Archivo | Qué tiene |
|---|---|
| [`comprobantes.xlsx`](ejemplo/comprobantes.xlsx) | 15 comprobantes de marzo y abril de 2026: 14 facturas C y una nota de crédito. Columnas parecidas a las de *Mis Comprobantes*, más la referencia del pago |
| [`ventas.xlsx`](ejemplo/ventas.xlsx) | 17 ventas con los problemas de siempre: referencias guardadas como número, importes como texto, una referencia repetida, otra mal copiada, ventas sin costo, pendientes y anteriores al corte |
| [`precios.xlsx`](ejemplo/precios.xlsx) | Lista de precios, con un aumento en abril, y un código de descuento |
| [`resultado-esperado.xlsx`](ejemplo/resultado-esperado.xlsx) | Lo que Claude tendría que encontrar: 9 hallazgos (3 🔴, 3 🟡, 3 🟢), las coincidencias con su criterio y los totales |

Poné los tres primeros en la carpeta del paso 2 y compará lo que te devuelve con `resultado-esperado.xlsx`.

### SKILL.md

```markdown
---
name: conciliacion-arca
description: Concilia los comprobantes emitidos en ARCA (facturas y notas de
  crédito) con los registros de ventas y arma un Excel de discrepancias. Usar
  cuando la persona pida conciliar facturas o comprobantes con ventas.
---

# Instrucciones: conciliación de comprobantes ARCA con ventas

Vas a conciliar los comprobantes que la persona emitió en ARCA (facturas y notas
de crédito) con sus registros de ventas. El objetivo es encontrar ventas
cobradas sin factura, facturas sin venta y errores de carga, **sin falsas
alarmas**.

## Criterios generales

- Leé y conciliá con código, nunca a ojo.
- No edites los archivos originales. El resultado va en un `.xlsx` **nuevo**.
- No inventes datos. Si una coincidencia es una suposición, marcala como tal.
- No muestres en el chat más datos personales (nombres, CUIT, mails) de los
  necesarios.
- Los importes vienen en formato argentino: punto de miles, coma decimal.

## 1. Antes de conciliar, validá con la persona

Si el `CLAUDE.md` del proyecto ya tiene definiciones de ejecuciones anteriores
(columnas, casos conocidos), partí de esas, pero mostralas igual.

Leé los archivos y mostrale a la persona, en una lista corta, lo que entendiste.
Esperá su validación antes de seguir:

- Qué archivo son los comprobantes y cuál las ventas, y qué columna es cuál:
  fecha, cliente, CUIT/DNI, importe, estado de pago, **referencia del pago**
  (número de operación, pedido o transacción).
- **Fecha de corte:** la del primer comprobante emitido. Las ventas anteriores
  quedan afuera y *no* cuentan como «factura faltante».
- Qué valor indica que una venta está **cobrada**.
- Si hay **ventas sin costo** (bonificadas, cortesías): no llevan factura; se
  listan, pero no se marcan.
- Si hay **comprobantes de otro contribuyente** mezclados: van aparte y no se
  concilian contra estas ventas.

## 2. Normalizá

Casi todos los errores salen de este paso.

- **Referencias como texto de dígitos.** Una planilla guarda `172604918356.0` y
  la exportación `"172604918356"`: comparadas así no coincide **ninguna**. Pasá
  todo a texto, sacá el `.0` y todo lo que no sea dígito.
- **CUIT/DNI** sólo con dígitos.
- **Importes** como número.
- **Nombres:** minúscula, sin tildes, sin espacios dobles, y «APELLIDO, NOMBRE»
  como conjunto de palabras.
- **Mails** en minúscula y **fechas** como fecha.

## 3. Conciliá, en este orden

Una factura que ya matcheó sale de la lista y no puede matchear con otra venta.
En cada coincidencia anotá con qué criterio se encontró.

1. **Misma referencia de pago.** Es prueba.
2. **Referencia con un dígito de menos** (`73958204617` contra `173958204617`).
   Es coincidencia **y además** error de carga a corregir.
3. **Mismo CUIT/DNI + importe exacto.**
4. **Nombre + importe exacto.** Al menos **dos palabras** del nombre de la venta
   tienen que estar en el de la factura. No exijas el nombre entero (los
   segundos nombres lo rompen) ni concilies sólo por nombre (un apellido común
   más un precio común junta a dos personas distintas).
5. **Consumidor final sin identificar.** Sólo por referencia de pago. Si nada
   más coinciden fecha e importe, y ese importe aparece una sola vez en el
   período, es **aproximado**: va como suposición, no como dato, y resaltado con
   otro color para que la persona decida.

## 4. Clasificá las discrepancias

- 🔴 **Alta**: una misma referencia de pago en ventas de **dos clientes
  distintos**. Casi siempre a uno le pisaron el dato y aparece también como
  cobrado sin factura.
- 🔴 **Alta**: venta cobrada sin factura.
- 🟡 **Media**: factura sin venta, por un importe que no está en la lista de
  precios. Que busque la referencia en su medio de cobro.
- 🟡 **Media**: factura a un nombre que no está en las ventas. Suele ser alguien
  que pagó por otra persona. Si una venta sin factura, del mismo importe y
  fecha, tiene una nota que lo indica, informá las dos como un solo hallazgo y
  marcalo como suposición.
- 🟡 **Media**: consumidor final conciliado sólo por fecha e importe.
- 🟢 **Baja**: referencia vacía o mal copiada en las ventas. Indicá el valor
  correcto.
- 🟢 **Baja**: nota de crédito seguida de una factura reemitida. Anotala para que
  nadie la vuelva a levantar.
- 🟢 **Baja**: factura emitida tarde. La venta cobrada tiene factura, pero se
  emitió más de 30 días después de la fecha de la venta. Indicá las dos fechas
  y los días de diferencia. La coincidencia sigue valiendo: la venta cuenta
  como conciliada.

**No marques como error:** dos facturas a la misma persona (suelen ser dos
compras: mirá productos y fechas), una nota de crédito más una factura nueva (es
una corrección bien hecha), ni un mail de pago distinto al de la venta (es la
misma persona con otra cuenta).

## 5. Cerrá los totales

Cada comprobante tiene que caer en **un solo grupo**: conciliado, de otro
contribuyente, sin identificar o nota de crédito. La **cantidad** y el
**importe** tienen que dar igual que la exportación; si no, hay uno contado dos
veces o uno perdido. Encontralo antes de entregar. Hacé lo mismo al revés: cada
venta cobrada dentro del alcance está conciliada, es sin costo o está marcada.

## 6. Entregá

- Un `.xlsx` nuevo, llamado `conciliacion-AAAA-MM-DD.xlsx`. Si ya existe,
  agregá `-v2`, `-v3`…: nunca reemplaces una conciliación anterior. Con:
  - **Discrepancias:** un hallazgo por fila (gravedad, qué pasa, cliente,
    importe, detalle con comprobante, referencia y fecha, y **qué hacer**),
    ordenado por gravedad. Cada «qué hacer» es concreto: qué registro, qué
    campo, qué valor.
  - **Coincidencias:** cada una con su criterio, ordenadas de la más débil a la
    más firme.
  - Las pestañas de apoyo que hagan falta: por ejemplo, los comprobantes de
    otro contribuyente.
  - **Log:** qué encontraste y por qué el resultado da lo que da.
- En el chat, un resumen corto: total por grupo, si los totales cierran y
  cuántos hallazgos hay de cada gravedad. Proponé 3 a 5 coincidencias por
  CUIT/DNI + importe o por nombre + importe para que la persona las revise.
- Si no podés generar archivos, entregá la tabla de Discrepancias en el chat.

## 7. Ejecuciones siguientes y aprendizajes

- Conciliá siempre todo el período, no sólo lo que la persona corrigió: un
  cambio puede mover otras coincidencias y los totales tienen que cerrar.
- Si en la carpeta hay una conciliación anterior, leela junto con su Log. En
  Discrepancias agregá una columna **Estado** (nueva o sigue) y en el resumen
  del chat listá también las que se resolvieron.
- No vuelvas a marcar lo que la persona ya explicó o decidió. Si aplicás una
  decisión anterior, anotalo en el Log.
- Cuando la persona te corrija o decida algo, proponé dónde guardarlo:
  - si es propio de su negocio (qué columna es cuál, un cliente que paga por
    otro, un caso que no es error), en el `CLAUDE.md` del proyecto, bajo
    «Conciliación»;
  - si cambia el método (un criterio nuevo, un tipo de falsa alarma), en este
    `SKILL.md`.
- Mostrá el texto exacto que vas a agregar y escribilo sólo después de que la
  persona lo valide.
- Antes de guardar un cambio en este `SKILL.md`, probalo: ejecutá la
  conciliación con los archivos de `ejemplo/`, usando sólo este método y no las
  definiciones del `CLAUDE.md`, y comparala con
  `ejemplo/resultado-esperado.xlsx`. Si da distinto y no debía, no guardes el
  cambio. Si el cambio tiene que cambiar el resultado, mostrá la diferencia y
  actualizá `resultado-esperado.xlsx` sólo con la validación de la persona.
- Nunca mezcles los archivos de `ejemplo/` con los de la persona.

## 📋 Validación antes de la entrega

- [ ] Originales sin tocar
- [ ] Columnas, fecha de corte, ventas sin costo y otros contribuyentes
      validados con la persona
- [ ] Referencias y CUIT como dígitos, importes como número, nombres sin tildes
- [ ] Conciliación en orden: referencia → referencia sin un dígito → CUIT +
      importe → nombre + importe → consumidor final
- [ ] Cada coincidencia con su criterio; las suposiciones marcadas como tales
- [ ] Cada comprobante en un solo grupo; cantidad y total iguales a la
      exportación
- [ ] Cada hallazgo con un próximo paso concreto
- [ ] Comparada con la ejecución anterior, si la hay
- [ ] Lo aprendido, guardado sólo después de la validación de la persona
- [ ] Si cambió este `SKILL.md`, el ejemplo da el resultado esperado
```
