# Decisiones de entorno, carga y escrituras

Fecha: 7 de noviembre de 2026. Autor: Andrés Garbarz.

Sirve para redactar la parte del informe que cubre el entorno, la carga y las consultas 13, 14 y 15. La elección de MongoDB y Neo4j, y el descarte de SQL, Cassandra y Redis, ya están en [decisiones-arquitectura.md](decisiones-arquitectura.md). Los ejemplos de esta página se ejecutaron el 2026-11-07.

## Entorno

MongoDB es `8.3.11`, la misma versión que el contenedor de las guías. Neo4j es `5.26.32`, el último parche de la serie 5.26 publicado en Docker Hub ese día: `5.26.33` no estaba. Los dos van sin autenticación y publican los puertos solo en `127.0.0.1`. `pymongo` queda en `4.15.3` y el driver `neo4j` en `5.28.1`.

El compose fija `GLIBC_TUNABLES=glibc.pthread.rseq=1`. La imagen de Mongo trae `rseq=0`. En kernels 6.19 a 7.0.13, como el de Docker Desktop para Mac, el proceso se corta al iniciar si glibc también usa rseq. Con `rseq=1`, glibc se queda el registro, TCMalloc no lo usa y Mongo arranca. El log muestra un aviso de rendimiento: es el efecto esperado y con estos datos no importa. En Codespaces, con kernel 6.8, arranca igual sin la variable. Queda fija para que el mismo archivo sirva en todas las máquinas. Esta variable va entre las limitaciones del informe.

Las URIs salen de `MONGO_URI` y `NEO4J_URI`. El devcontainer las define para el Codespace. Fuera de él, el defecto es `localhost`.

## Fecha del sistema

`hoy()` devuelve la fecha de Argentina a medianoche, armada con año, mes y día. Las fechas de la carga se construyen igual. No se usa un parseo del ISO completo, porque en Argentina corre el día. El Codespace corre en UTC: entre las 21 y las 24, UTC ya es el día siguiente. La 14 y la 15 no reciben la fecha como parámetro. Un lote está vencido si `vencimiento` es anterior a `hoy()`: el día que vence todavía se vende. Las consultas que dicen «hoy», «mes actual», «últimos 60 días» o «último año» tienen que usar esta función, no una fecha fija. [P18, p. 6]

## Modelo que quedó cerrado

MongoDB es la fuente. Neo4j se actualiza después y, si esa proyección falla, se rehace desde Mongo. Los ítems van dentro del documento de la venta: la cabecera y el detalle se leen juntos, y esa escritura es atómica dentro del documento. [T14, pp. 3–8; P18, pp. 4–6]

El proveedor queda referenciado con `id_proveedor`, como en el CSV. La consulta 1 hace ese encuentro. `_id` es el id de la cátedra. Los nombres de campo son los del CSV. Los booleanos son booleanos y los importes, enteros. Los campos opcionales vacíos (`nro_afiliado`, `id_receta`, `proxima_dispensa`) quedan en nulo.

Cada ítem que descuentan la 14 y la 15 guarda de qué lotes salió: `items.lotes`, una lista de `{id_lote, unidades}`. Las ventas de los CSV no lo traen. `fecha_registro` lo escribe solo la 15.

En una venta Completada, `fecha` es el día en que computa ingresos. [P18, p. 5] Si la completó la 15, `fecha` es el día de la entrega y la reserva queda en `fecha_registro`. Las consultas 3, 5, 11 y 12 ordenan o filtran por `fecha`. La 11 suma los ingresos del mes por esa fecha, así que una reserva de octubre completada en noviembre entra en noviembre. `fecha_registro` solo conserva el día de la reserva.

## Carga

La carga borra las ocho colecciones y las vuelve a llenar desde los archivos. Corrida dos veces, deja las dos el mismo contenido: 8 clientes, 16 ventas y 22 ítems embebidos. Es reproducible e idempotente. [P18, p. 6]

Lo creado solo con las consultas 13 y 14 se pierde al recargar. Lo que la 15 pasó de Pendiente a Completada vuelve a Pendiente. Eso se dice en el README. Los ejemplos de las consultas 1 a 12 se sacan con una carga recién hecha.

Cada archivo de `datasets/extra/` se carga si existe, con el mismo nombre, los mismos encabezados e ids que siguen la serie. La carga informa, por colección, cuántos documentos son de la cátedra y cuántos propios, y avisa si no hay propios. Falla si un id propio repite uno de la cátedra, si una referencia no existe o si el total de una venta no coincide con sus ítems. El 2026-11-07 se probó con un cliente `C001` repetido y con una venta `V017` apuntando a `C999`: las dos frenaron la carga. Los motores NoSQL no garantizan la referencial; estas validaciones, el índice único de DNI y los ítems embebidos son la estrategia. [P18, pp. 5–6]

Al final llama a `reconstruir()`.

## Consulta 13

`ejecutar(operacion, **datos)` llama a `alta`, `modificar(id_cliente, ...)` o `baja(id_cliente)`. El alta y la venta generan el siguiente correlativo con el mismo prefijo y tres dígitos, y reintentan si chocan con el índice de `_id`. Lo mismo para el `id_detalle` de cada ítem nuevo.

El alta rechaza un DNI repetido, también si el cliente anterior está de baja. El índice único se llama `dni_unico`. Modificar solo cambia email, teléfono, ciudad, provincia, obra social y número de afiliado. La consigna llama domicilio a ciudad y provincia; tratarlas como contacto es un supuesto del informe. La cobertura tiene que ser OSDE, Swiss Medical, PAMI, IOMA, Galeno o Sin cobertura, y el número de afiliado va vacío solo con Sin cobertura. La baja pone `activo` en false y no borra el documento. [P18, pp. 5–6]

Después se llama a `upsert_cliente`.

El 2026-11-07, sobre 8 clientes:

| Caso | Resultado |
|---|---|
| Alta con el DNI `25678901` de `C008`, que está de baja | Rechazo: ya existe un cliente con ese DNI. Siguen 8 |
| Alta en OSDE sin número de afiliado | Rechazo: la obra social exige número de afiliado. Siguen 8 |
| Alta de Ana Prueba, DNI `30111222`, OSDE, afiliado `99` | Queda `C009`, activo |
| Modificar el email de `C009` | Queda `ana2@x.com` |
| Baja de `C009` | `activo` pasa a false. El documento sigue y el conteo queda en 9 |

## Precio y consulta 14

La 14 registra una venta nueva. El llamador manda `id_producto` y `cantidad`. El servicio lee `precio_unit` del producto en ese momento, lo copia en el ítem y el total es la suma. [P18, pp. 5–6] El medio de pago es Efectivo, Débito, Crédito, Transferencia u Obra social. El tipo es Con receta si algún ítem es Bajo receta o Bajo receta archivada; si no, Venta libre. Hay una receta por venta. Se exige solo cuando un producto la pide. Si viene, tiene que existir, estar Vigente, ser del mismo cliente y tener dispensas disponibles.

Valida, en este orden, cliente, farmacéutico, sucursal, productos y receta. Descuenta antes de insertar. Solo si todos los lotes alcanzaron y la receta se pudo actualizar, inserta la venta Completada. Si falta stock, no deja venta. Si no puede asignar el id, repone lotes y receta.

El 2026-11-07, con carga recién hecha: `L005` en `S001` tiene 40 unidades y vence el 2026-11-30; `L015` tiene 100 y vence el 2027-06-30. `PRD003` sale 9800. `RC003` está vigente, 2 de 3. `RC004` está vigente, 1 de 3.

| Caso | Resultado |
|---|---|
| 200 unidades de `PRD003` con `F001` y `RC004` | Rechazo por stock. No toca lotes, venta ni receta. Siguen 16 ventas |
| Una unidad con `F007` | Rechazo: el farmacéutico no pertenece a una sucursal de tipo Farmacia. `F007` es de la droguería |
| 50 unidades, `F001`, `C006`, `RC003`, obra social | `V017`, total 490000, Completada, fecha 2026-11-07, ítem `D023`. Salen 40 de `L005` y 10 de `L015`. `RC003` queda Completada, 3 de 3, sin próxima dispensa |
| 50 unidades, `F001`, `C006`, `RC004`, obra social | `V018`, ítem `D024`, las 50 de `L015`. `L015` queda en 40. `RC004` sigue Vigente, 2 de 3, y la próxima dispensa pasa al 2026-12-07 |

Desde el 2026-12-01 `L005` está vencido y las 50 unidades salen de `L015`. El informe tiene que dejar anotada la fecha de ejecución.

## Consulta 15

La consigna pide, dada una venta registrada, decrementar los lotes de la sucursal del farmacéutico. [P18, p. 6] En esta parte esa venta tiene que estar Pendiente. La 15 la revalida, descuenta por FEFO y, solo si todo alcanzó, la pasa a Completada y aplica la receta. Una Completada o una Anulada se rechaza sin tocar lotes. Los precios quedan los del registro. Al completar, `fecha` pasa a ser la de la entrega; la de la reserva queda en `fecha_registro`. Leer la 15 como ese cambio de estado, y mover `fecha`, es un supuesto del informe.

El descuento es `descontar(id_sucursal, items)`, en el módulo de la 15. La 14 lo llama antes de insertar.

El 2026-11-07, con carga recién hecha:

| Caso | Resultado |
|---|---|
| `V003`, ya Completada | Rechazo. Ningún lote cambia |
| `V012`, Pendiente en `S003`, que no tiene lotes de ibuprofeno ni de loratadina | Rechazo: no hay stock vigente de `PRD002` en `S003`. Sigue Pendiente |
| `V011`, una unidad de `PRD007`, receta `RC006` | Completada, fecha 2026-11-07, `fecha_registro` 2026-10-12. Sale 1 de `L014`, que pasa de 45 a 44. `RC006` queda Completada, 1 de 1 |

## Consistencia

Si el descuento de stock falla, no pueden quedar una venta ni una receta modificadas. [P18, p. 7]

El plan FEFO lee lotes con vencimiento mayor o igual a `hoy()`, ordenados por `{vencimiento: 1, _id: 1}`. El `sort` de MongoDB no es estable, y `_id` es el id del lote, así que dos lotes que vencen el mismo día salen siempre en el mismo orden. Si el plan no alcanza, se rechaza antes de escribir. Si alcanza, cada lote baja con la condición de que `unidades` alcance. Si un lote ya no alcanza, se reponen los anteriores y se rechaza. Dos ventas simultáneas no dejan unidades negativas por esa condición.

La receta se actualiza si `dispensas_realizadas` sigue valiendo lo que se leyó. En un tratamiento prolongado, mientras queden dispensas, la próxima es la fecha de la dispensa más 30 días. Al llegar a las autorizadas pasa a Completada y la próxima queda vacía. Una receta simple o archivada se completa en esa dispensa. [P18, p. 5] Si la condición no coincide, se reponen los lotes y no se graba ni se completa la venta.

La 14 inserta después del descuento y de la receta. La 15 pasa a Completada con la condición `estado: "Pendiente"`. Una segunda llamada no encuentra la venta, así que no vuelve a descontar. No hay un campo `stock_descontado`: el estado cumple esa función. Si la condición no coincide, se reponen los lotes y la receta.

No se usan transacciones de varios documentos. En MongoDB esas transacciones piden un replica set, y este entorno es una sola instancia. La atomicidad que enseña la cátedra es la de un documento. [T14, pp. 3–8] La estrategia es el orden de las operaciones, la actualización condicional y la reposición. Si el proceso muere entre dos lotes, el primero puede quedar descontado sin una venta que lo explique. Esa ventana va al informe.

Si el grafo falla, Mongo ya confirmó. La consulta no devuelve error: avisa que el grafo quedó pendiente y `reconstruir()` lo rehace. Las cuatro funciones están vacías, así que los casos que entran muestran ese aviso.
