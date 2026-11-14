# Decisiones de arquitectura — TPO FarmaRed

Fecha: 3 de noviembre de 2026.

Los motores quedan cerrados. El 7 de noviembre se cerró el modelo de las colecciones y quedaron hechos el entorno, la carga y las consultas 13, 14 y 15. El 10 de noviembre se implementaron y verificaron Q01, Q05, Q07, Q08, Q09 y Q11. Para continuar el código, [avances-tpo.md](avances-tpo.md). Para redactar el informe de escrituras, [decisiones-escrituras.md](decisiones-escrituras.md), y para las seis lecturas, [informe-consultas-lectura.md](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/blob/11c9d1b622b2c15a4f2ccc070cf1862ff02d129f/docs/informe-consultas-lectura.md). El 14 de noviembre se integraron las quince consultas y el grafo en `main`, con 25 pruebas de integración correctas. El modelo y las etiquetas del grafo están en [informe-grafo.md](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/blob/main/docs/informe-grafo.md).

## Decisión

El sistema usa dos motores NoSQL, de paradigmas distintos:

- **MongoDB** (documentos) es la fuente de los datos y el dueño de las escrituras.
- **Neo4j** (grafos) guarda el camino entre entidades y responde las consultas que lo recorren.

No se agrega Cassandra, Redis ni un motor SQL. Esas opciones se evalúan más abajo y van al informe como alternativas descartadas. La consigna pide al menos dos paradigmas y justifica cada motor por el caso de uso; no pide usar todos los ejemplos que enumera. Cada motor de más suma integración, despliegue y respaldo. [P18, pp. 5–7; C01, p. 6]

## Qué guarda cada motor

**MongoDB** guarda clientes, sucursales, farmacéuticos, proveedores, productos, lotes, recetas y ventas. Los ítems van dentro del documento de la venta: la cabecera y el detalle se leen juntos, y esa escritura es atómica dentro del documento. El stock vive en esta misma base, al lado de la venta y de la receta. [T14, pp. 3–8; P18, pp. 4–6]

**Neo4j** guarda el camino cliente–receta–venta–farmacéutico–sucursal–producto. No guarda el stock ni confirma la venta. Se actualiza después de que Mongo confirmó la operación. Si esa actualización falla, el grafo se rehace desde Mongo.

| Consultas | Motor |
|---|---|
| 1, 4, 5, 6, 7, 8, 9, 11, 12 | MongoDB. Filtros y agregaciones sobre colecciones. |
| 2, 3, 10 | Neo4j. Recorren farmacéutico, sucursal, cliente, ventas y recetas. |
| 13, 14, 15 | MongoDB. Alta y baja de clientes, registro de la venta y descuento de lotes. |

La consulta 13, al modificar datos de un cliente, también actualiza el nodo correspondiente del grafo. El teléfono y la cobertura no se copian en otras estructuras.

## Por qué este par

La consigna separa cinco necesidades. Tres eligen el motor. [P18, p. 5]

La venta es una cabecera con ítems de longitud variable, y los reportes son totales, rankings y conteos. En la materia eso es un documento con sus ítems y un pipeline de agregación. El catálogo se lee mucho y cambia poco, y entra en el mismo motor. [T14, pp. 3–8; T15, p. 31]

Clientes, recetas y farmacéuticos se consultan recorriendo relaciones. La consulta 10 es ese camino: la sucursal, el farmacéutico que registró la venta y el cliente. La 2 y la 3 son el mismo tipo de paso. Neo4j está presentado como un motor de grafos transaccional, y la consulta toca el nodo y sus relaciones. [P18, pp. 5–6; T17, pp. 1, 6, 15]

El stock pide actualizaciones concurrentes, sin quedar negativo y en orden FEFO. Esa fila, tomada sola, se acerca más a Redis. La dejamos en Mongo por la regla de las consultas 14 y 15, que se explica abajo.

## Consultas 14 y 15

Si el descuento de stock falla, no pueden quedar ni la venta ni la receta modificadas. Venta, receta y lotes comparten ese fracaso, así que comparten motor. [P18, pp. 6–7]

El servicio hace esto, en este orden:

1. Valida en Mongo: cliente existente y activo; farmacéutico existente, activo y de una sucursal de tipo Farmacia; productos existentes y activos; y, si el producto es bajo receta, receta existente, vigente y del mismo cliente.
2. Arma el plan FEFO sobre los lotes de la sucursal del farmacéutico. Saltea los vencidos. El stock no puede quedar negativo.
3. Actualiza cada lote solo si las unidades alcanzan. Si un lote posterior no cumple, repone los que ya había descontado. En ese caso no escribe la venta ni toca la receta.
4. Recién cuando todos los lotes entraron, graba la venta (el total sale de los ítems) y actualiza la receta: incrementa `dispensas_realizadas`, mueve `proxima_dispensa` o completa la receta, según las reglas de tratamiento prolongado, simple y archivada.
5. Después proyecta la venta en Neo4j.

La atomicidad que enseña la cátedra es la de un documento. La secuencia de varios lotes queda en el servicio. Con el proceso vivo, un lote que no alcanza provoca la reposición de los anteriores. Si el proceso muere entre dos lotes, el primero puede quedar descontado sin una venta que lo explique. Esa limitación va al informe. Dos ventas simultáneas no dejan unidades negativas, porque cada lote solo baja si todavía tiene las unidades pedidas. [T14, pp. 3–8; P18, pp. 4–7]

### El caso que fija la decisión

`PRD003` (Amoxicilina 500 mg, bajo receta) está en Farmacia Palermo (`S001`) en dos lotes: `L005`, 40 unidades, vence el 2026-11-30, y `L015`, 100 unidades, vence el 2027-06-30. Una venta de 50 atendida por `F001` consume 40 de `L005` y 10 de `L015`. Una de 200 encuentra 140 vigentes: no se modifica ningún lote, ni la venta, ni la receta.

## Alternativas evaluadas

### Motor SQL, por `CHECK` y `ASSERTION`

Un `CHECK` de MySQL mira columnas de la misma fila y no admite subconsultas. `ASSERTION` es la construcción del estándar para reglas entre tablas, y la cátedra señala que los DBMS comerciales no la implementan. [T09, pp. 17–25; P06, pp. 4–5]

| Regla de FarmaRed | Dónde queda |
|---|---|
| Unidades no negativas, estado dentro de un conjunto | Validación al escribir, y unicidad de DNI y matrícula por índice |
| El total de la venta es la suma de sus ítems | Los ítems van embebidos; el total se calcula en esa escritura |
| FEFO, lotes vencidos y rechazo de la venta entera | El servicio 15 |
| Bajo receta solo con receta vigente del mismo cliente | El servicio 14, antes de escribir |
| Solo farmacias, y solo con un farmacéutico activo | El servicio 14 |
| Completada descuenta stock; Pendiente y Anulada no | El orden de los servicios 14 y 15 |
| La baja es `activo = false` | El servicio 13 no borra el documento |

Un `CHECK` solo protege la tabla en la que vive el dato. La consigna pide documentar la integridad en la aplicación, con embebido o con otra estrategia, porque los motores NoSQL no garantizan la referencial. [P18, pp. 5–7]

### Cassandra

Serviría para una tabla por consulta: historial de un cliente ordenado por fecha, clientes de una sucursal, ingresos de un farmacéutico en un mes. La clase diseña así porque no hay join, y copia en la fila lo que la consulta muestra. [T16, pp. 16–19]

Esas lecturas ya tienen dueño. El historial y la sucursal se recorren en Neo4j. El total del mes es una agregación de Mongo. Cassandra no contesta una consulta que esos dos dejen mal armada, y una modificación de la consulta 13 tendría que alcanzar cada tabla que haya copiado el dato del cliente. “Últimos 60 días” y “último año” cruzan meses: un bucket mensual obliga a leer varias particiones. La clasificación de la materia es AP, así que tampoco es el lugar del stock. [T13, p. 18; T16, pp. 11, 44–46; P18, p. 6]

### Redis

Un conjunto ordenado puede usar el vencimiento como score, y un script Lua descuenta todos los lotes de una venta mientras ningún otro cliente ejecuta comandos. Si un lote no alcanza, el script no modifica ninguno. En el ejemplo de la amoxicilina, las 40 y las 10 se toman juntas, o no se toma nada. [T19, pp. 13–16; T18, p. 31]

La venta y la receta seguirían en Mongo. El corte se corre a la ventana que hay entre el script y esa escritura: Redis puede haber descontado 50 unidades y Mongo no tener la venta. Una caída puede, además, persistir solo una parte de una transacción, y un snapshot puede perder los minutos de escrituras desde el último guardado. La consulta 8 pasaría a leer las unidades en Redis y los nombres en Mongo. [T18, pp. 12–13, 17–24]

El script mejora el tramo de los lotes y parte el fracaso que la consigna define como uno solo. Por eso el stock se queda en Mongo.

### Los cuatro paradigmas a la vez

Documentos, grafos, clave-valor y columnar cubren las cinco filas de la consigna con el motor más afinado de cada una. La nota de arquitectura no sube por levantarlos. Una venta confirmada atravesaría el stock, la venta, el grafo y las tablas copiadas. El único caso que Redis agrega sobre el par elegido es el script de los lotes, y ese caso es el que decidimos no abrir. [C01, p. 6; P18, pp. 5–7]

## Qué falta cerrar

- Diagrama de colecciones y consolidación del informe PDF. El diagrama del grafo ya está en `docs/informe-grafo.md`.
- Registros adicionales: al menos diez por colección, con casos borde. La carga ya los toma de `datasets/extra/`.
- Las quince consultas ya están implementadas. Q02, Q03 y Q10 recorren el grafo; Q04, Q06 y Q12 se integraron con las otras lecturas MongoDB.

## Fuentes

Consigna del TPO [P18]. Clases de documentos [T14, T15], grafos [T17], clave-valor [T18, T19], Cassandra [T16], CAP [T13], integridad [T09, P06] y persistencia políglota [C01].
