# Registro de cambios: Generador de Presupuestos

Archivo: `generadordepresupuestos/index.html` (repositorio `leandrocuadro/SGF`)

Formato de cada entrada: versión, fecha, detalle de cambios (qué se modificó, en qué parte de la app y por qué). Las entradas más nuevas van arriba.

---

## v3.5 (copia de trabajo, pendiente de subir) · 2026-11-17

Dividir un pliego entre varios usuarios y unificarlo después.

- **Campo "Usuario"** (última columna; uso interno, no sale en PDF ni CSV): el coordinador asigna a cada ítem un número de usuario a mano. El **0** significa que no se presenta oferta: esos ítems no van a ninguna parte y quedan en el principal.
- **Importación Excel/CSV con Usuario:** reconoce la columna "Usuario" (última). Acepta 0 a 99; lo inválido queda vacío y se avisa. La plantilla de ejemplo la incluye.
- **Un solo zip:** "Dividir por usuario" descarga `pliego_<id>_partes.zip` con el principal (`pliego_<id>_principal.json`) y una parte por usuario (`pliego_<id>_usuario_<n>.json`). Ya no se descargan archivos sueltos.
- **Pliego principal:** el principal va dentro del mismo zip. El coordinador lo carga primero y después usa "Unir partes". Desde una plantilla vacía, la unión pide cargar ese archivo.
- **Nombre de archivo:** cada JSON de parte lleva el usuario (`pliego_<id>_usuario_<n>.json`), tanto al dividir como al guardar desde una parte.
- **Dividir por usuario:** genera un JSON por usuario con sus ítems, con el mismo identificador de pliego y los números originales.
- **Modo parte:** al abrir un JSON de parte, el usuario solo completa sus ítems. No puede agregar ni borrar ítems, ni cambiar su número ni el usuario asignado. Oculta Dividir, Unir e Importar Excel.
- **Unir partes:** importa los JSON de los usuarios en el presupuesto principal. Verifica pliego, usuarios repetidos, ítems inexistentes o mal asignados. Si falta una parte, pide confirmación. Al final ordena por número de ítem del pliego, con orden numérico (1, 1.9, 1.10, 2…).
- **Identificador interno por renglón (uid):** cada ítem guarda un identificador propio en el JSON. La unión empareja por ese identificador y no por número, así que los ítems sin número también se unen.
- **Números repetidos:** al dividir, si un mismo número de ítem aparece en más de un renglón, la división se bloquea y pide corregir el pliego antes.
- **Ítems sin número:** al dividir, un cartel avisa cuántos ítems no tienen número y pregunta si se continúa igual o se cancela.
- **Cotización del usuario prioritaria:** al unir, se respetan la moneda, el costo y el margen que cargó cada usuario. Si difieren de los del principal, pide confirmación mostrando el cambio (por ejemplo, "Dólar BNA: 1000 → 1250"); cancelar mantiene la cotización del principal. Si dos partes cotizaron distinto entre sí, la unión se bloquea.
- **Alertas no bloqueantes tras unir:** "Sin costo" (precio en 0 o menos) y "Con costo y sin margen". Son avisos para el coordinador; no impiden la unión.
- Pruebas: 27 casos de división, edición de parte, unión y errores, y 9 casos de bordes (X1–X6), todos OK.

## v3.4.1 · 2026-11-17 (subida al repositorio, commit 0dc53d1)

- **Campo de cliente 1 libre:** con el Nro. Cliente 1 el campo Cliente / Empresa es texto libre. No muestra sugerencias y Enter o Tab no reemplazan lo escrito (antes, un nombre que coincidía con la base se reemplazaba por el cliente de la base).
- **Número de versión visible** en la barra superior de la app.

## v3.4 · 2026-11-17 (subida al repositorio, commit 01a527d)

Sobre v3.3.

- **Cliente 1 libre:** con el Nro. Cliente 1 (o 001) el nombre se escribe libremente y no se valida ni se completa contra la base. El resto de los clientes no cambia.
- **Advertencia de cotización:** si una fila usa una moneda sin cotización cargada, el campo Precio Lista se marca en rojo con un mensaje al pasar el mouse, y el aviso general de cotización aparece también al cambiar la moneda de una sola fila.
- **Falta de cotización:** la fila ya no toma el precio sin convertir; queda en 0 hasta que se cargue la cotización.
- **Redondeo comercial:** los precios unitarios de costo y de venta se redondean a 2 decimales (si el tercer decimal es 5 o más, sube). El total de cada fila sale de ese unitario redondeado, así que "unitario × cantidad" coincide con el total impreso y con el TOTAL del PDF. Verificado: 30 filas sin diferencias, 200 filas con diferencia 0,00.

**Pendiente (no tocado):** el CSV exporta el Precio Lista con 2 decimales, así que un precio de 12,3456 vuelve como 12,35 al reimportar.

---

## Revisión general de cálculos y funciones críticas · 2026-11-17

Revisión sobre v3.3 (sin cambios de código). Se probó en un navegador real con casos de resultado conocido.

**Verificado y correcto**
- Cálculo de fila: cascada de 5 descuentos, recargo con Desc 4 y 5 negativos, tope de 100 %, margen sobre costo, total por cantidad (19 de 19 casos coinciden con el cálculo a mano).
- Conversión entre $, USD BNA, Dólar Divisa y EUR en todas las direcciones.
- Totales generales y KPIs (costo, venta, ganancia, unidades).
- Respaldo JSON: guardar y restaurar conserva todos los campos.
- PDF: archivo válido, acentos y ñ correctos, el TOTAL impreso coincide con el calculado.
- Cliente 1 libre (copia local, no subida): el nombre se escribe libre solo con el código 1 (también "001"); el resto sigue validado contra la base.

**Hallazgos (sin corregir)**
1. Al cambiar a USD/DIVISA/EUR la moneda de una sola fila con la cotización en 0, no aparece el aviso de cotización y el importe de esa fila queda en 0,00.
2. Redondeo: el PDF imprime el precio unitario redondeado y el total de cada fila sin redondear antes. En una prueba de 30 ítems, 23 filas no coinciden con unitario × cantidad (hasta 0,08) y la suma de los totales impresos difiere en 0,01 del TOTAL.
3. CSV: el Precio Lista sale con 2 decimales; un precio de 12,3456 vuelve a importarse como 12,35. El respaldo JSON no tiene este problema.
4. Emojis y algunos símbolos fuera de la codificación del PDF se imprimen como "?".
5. Ya conocidos: con filtros activos los totales en pantalla no coinciden con el PDF/CSV; mensajes de cotización que se pisan; "1.500" se lee como 1500 al importar; el autoguardado falla sin aviso al superar la cuota del navegador; la carga de muchos ítems crece más que en forma lineal.

---

## v3.3 (línea base) · 2026-11-04

Estado de la app al momento de iniciar este registro. Es la versión de referencia para los cambios siguientes. Esta entrada no describe cambios nuevos: documenta lo que ya existía.

**Funciones principales**
- Tabla de ítems con 22 columnas: cantidad, código, detalle, marca, solicitud del cliente, observaciones, precio de lista, moneda del costo, 5 descuentos en cascada, margen, precio de venta y totales.
- Autocompletado de Detalle y Marca a partir del código, con datos de `productos.parquet`.
- Autocompletado y validación de Nro. y nombre de cliente contra `clientes.parquet`.
- Conversión de precios entre $, USD BNA, Dólar Divisa y EUR, con cotizaciones que se pueden actualizar desde servicios externos.
- Notas internas (Memo) por ítem.
- Filtros por columna, reemplazo global sobre filas visibles y deshacer.
- Exportación: PDF propio (sin librerías externas), vista previa, CSV y respaldo JSON.
- Importación: Excel/CSV y respaldo JSON, con plantilla de ejemplo.
- Autoguardado en el navegador (localStorage).

**Problemas conocidos (sin corregir todavía)**
- El repositorio es público y `clientes.parquet` puede descargarse sin autenticación.
- Con filtros activos, los totales en pantalla no coinciden con los exportados en PDF y CSV (estos exportan todas las filas).
- La línea de cotizaciones (`infoCotizacionBNA`) muestra dos mensajes distintos que se sobrescriben entre sí.
- Si falta una cotización, las filas en esa moneda se calculan en 0 sin aviso en la fila.
- Al importar, `normalizarNumero` puede interpretar "1.500" como 1500 (separador de miles).

**Observaciones del repositorio**
- El último commit (`c29d69d`, 2026-11-02) se titula "v1.3.0: encabezado fijo en la tabla de resultados", pero en el código no hay ningún encabezado fijo en la tabla. La numeración del commit no coincide con la de la app (v3.3).
- Existe un archivo basura en la raíz del repositorio llamado `generadordeprespuestos` (contiene solo un salto de línea).
