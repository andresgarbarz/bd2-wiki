# Q01, Q05, Q07, Q08, Q09 y Q11 — implementación y verificación

Revisión: 10 de noviembre de 2026. Base del trabajo: `main`, commit `383168c`, del [repositorio del grupo](https://github.com/andresgarbarz/TPO_BD2_FarmaRed). Consigna completa: [P18](../material/catedra/practica/tpo-farmared.pdf), siete páginas. Las seis consultas están implementadas, verificadas y publicadas en [feat/consultas-lectura-mongodb](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/tree/feat/consultas-lectura-mongodb), hasta el commit `7a4fad5`. Cada consulta tiene su commit con pruebas; el [historial resumido](avances-tpo.md#commits-de-las-seis-lecturas) permite retomarlas por separado. El [informe de lecturas](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/blob/7a4fad5777ca2e77a12a866bd91e0b8351bec1c3/docs/informe-consultas-lectura.md) contiene los resultados ejecutados.

Los CSV provistos se encuentran en [datasets/](../datasets/), en la raíz de esta wiki. Sus nueve archivos son idénticos byte a byte a `entregas/tpo-farmared/datasets/`; por eso los controles de esta página sirven para ambas copias. El cargador lee la carpeta interna del proyecto. Las rutas `src/`, `docs/` y `datasets/extra/` de este documento son relativas al repositorio del grupo.

## Contexto para continuar

La arquitectura decidió MongoDB para estas seis consultas. Mongo es la fuente de datos y Neo4j se reserva para Q02, Q03 y Q10. Las seis lecturas funcionan sin el grafo. Se actualizaron sus encabezados y los textos de `src/consultas/__init__.py` y `__main__.py`.

Estado comprobado por lectura del código:

| Área | Estado |
|---|---|
| Entorno y dependencias | Compose, devcontainer Python 3.12, MongoDB `8.3.11`, Neo4j `5.26.32` y drivers fijados |
| Carga | Ocho colecciones; los 22 detalles se embeben en `ventas.items`; valida referencias y totales; prepara la vista mensual de Q11 |
| Q13, Q14 y Q15 | Implementadas; esta revisión no repite sus pruebas de ejecución |
| Q01, Q05, Q07, Q08, Q09, Q11 | Implementadas y verificadas con MongoDB real |
| Q02, Q03, Q04, Q06, Q10, Q12 | Siguen en `NotImplementedError` |
| Grafo | Cuatro funciones de proyección pendientes |
| Datos adicionales | No existe `datasets/extra/`; falta cumplir los diez registros propios por colección/tabla exigidos [P18, pp. 1, 6] |
| Informe | Existen los documentos de entorno/escrituras y de las seis lecturas; faltan diagramas y consolidar el informe final [P18, p. 7] |
| Pruebas | Verificador de CSV y 16 pruebas de integración de las seis lecturas, con resultados y bordes |

## Qué pide cada consulta

Los requisitos de esta tabla están en [P18, p. 6]; los estados de venta y los precios históricos, en [P18, pp. 4–5]. Las relaciones deben explicarse en el informe [P18, p. 7].

| Consulta | Requisito | Datos y cuidado principal |
|---|---|---|
| Q01 | Productos activos con **todos** los datos del proveedor | Partir de `productos`, filtrar `activo: true` y resolver `id_proveedor` contra `proveedores._id`. No se pide filtrar proveedores activos. |
| Q05 | Farmacéuticos activos y cantidad de ventas en los últimos 60 días, excluyendo anuladas; incluir los de cero | Partir de `farmaceuticos`, conservar todos los activos y contar ventas relacionadas por `id_farmaceutico`. Las pendientes sí cuentan. No se pide limitar a sucursales Farmacia: `F007`, de Droguería Central, debe aparecer con cero. |
| Q07 | Cinco productos más vendidos por unidades, solo ventas Completada | Desplegar `ventas.items` y sumar `items.cantidad` por producto. No contar documentos ni sumar pesos. Sin filtro de fecha ni de producto activo. |
| Q08 | Lotes con menos de 50 unidades, indicando producto, sucursal y proveedor | Partir de `stock_lotes`, resolver producto y sucursal; llegar al proveedor a través del producto. El límite es estricto: 50 queda afuera. No excluir lotes vencidos, unidades cero, droguería ni productos inactivos. |
| Q09 | Ventas de tipo Venta libre, total menor a $5.000, excluyendo anuladas | Filtrar `ventas.tipo`, `ventas.total` y `ventas.estado`. Las pendientes sí pueden aparecer. El límite es estricto: $5.000 queda afuera. No confundir con productos de venta libre ni con ventas gratuitas. |
| Q11 | Vista agregada de ingresos por farmacéutico en el mes actual, solo ventas Completada | Agrupar ventas por `id_farmaceutico` y sumar `total`, que conserva el importe histórico. No recalcular desde el precio actual del catálogo ni filtrar por farmacéutico activo. No se exige incluir los que tienen cero. |

## Contratos y decisiones de implementación

- Reutilizar `src/db.py`: `base()` y `hoy()`. Los ids oficiales están tanto en `_id` como en el campo `id_*`; las referencias conservan los nombres del CSV. Fechas BSON `Date`, booleanos reales e importes enteros.
- Q05 y Q11 usan `ventas.fecha`, no `fecha_registro`. Q15 mueve `fecha` al día de la entrega cuando completa una reserva; el informe ya declara ese supuesto en [decisiones-escrituras.md](decisiones-escrituras.md).
- Las fechas de producción deben salir del sistema, nunca del 13/10/2026 de referencia ni de los ejemplos de esta página. [P18, pp. 1, 6]
- La consigna no precisa la inclusión exacta del límite de 60 días. Para los controles de abajo se tomó `hoy() - 60 días` inclusivo hasta hoy inclusivo. Documentar la convención al implementar y probar el día del corte y una fecha futura. Es una decisión de implementación, no una precisión atribuida a la cátedra.
- Para Q11, el plan propone una vista de ingresos agrupados por farmacéutico, año y mes. `ejecutar()` seleccionará el año y mes de `hoy()`, sin guardar límites de fechas fijos en la definición. Es equivalente a tomar desde el primer día del mes inclusivo hasta el primer día del mes siguiente exclusivo. Probar diciembre/enero. La elección de una vista mensual general más el filtro en Q11 es una decisión del proyecto para resolver la «vista agregada» pedida por P18.
- Q07 no especifica desempates. Propuesta de implementación: unidades descendentes e id de producto ascendente para obtener hasta cinco resultados reproducibles.
- Cada `ejecutar()` devuelve `list[dict]`; las seis no requieren argumentos de negocio. Los docstrings documentan propósito, entrada, retorno y motor. Las pruebas controlan el reloj sin fijar fechas en el código productivo. [P18, pp. 6–7]

Los criterios de límites temporales y desempate anteriores son propuestas del proyecto, no reglas adicionales de la fuente oficial.

## Casos de control con los CSV provistos

Estos resultados se calcularon directamente desde los CSV con Python estándar y aserciones; **no son ejecuciones de los módulos de consultas ni de MongoDB**. Corresponden a una carga sin extras ni modificaciones de Q13–Q15. Referencia temporal del control: **2026-11-10**; corte de Q05: **2026-09-11**.

| Consulta | Resultado de control |
|---|---|
| Q01 | 11 productos: `PRD001` a `PRD011`, cada uno con su proveedor completo |
| Q05 | `F001: 1`, `F002: 2`, `F003: 1`, `F005: 3`, `F006: 1`, `F007: 0`; `F004` no aparece por estar inactivo |
| Q07 | `PRD005: 6`, `PRD011: 5`, `PRD001: 4`, `PRD004: 3`, `PRD009: 2` unidades |
| Q08 | 9 lotes: `L002`, `L003`, `L005`, `L007`, `L010`, `L011`, `L012`, `L014`, `L018` |
| Q09 | `V013` ($4.400) y `V014` ($4.600) |
| Q11 | No hay ventas completadas en noviembre de 2026: una agrupación de ventas devuelve cero filas |

Casos que ayudan a detectar filtros equivocados:

- `V003`, del 10 de septiembre, ya queda fuera de Q05 con esa fecha de control. `V011` y `V012`, pendientes, sí cuentan. `V015`, anulada, no cuenta.
- `L012` y `L018` están vencidos al 10 de noviembre, pero pertenecen al resultado de Q08. `L018` es además del producto inactivo `PRD012`.
- `V014` contiene `PRD012`, hoy inactivo: sigue siendo una venta válida para Q07 y Q09.
- Q11 necesita registros de prueba completados del mes actual para mostrar un resultado con ingresos. Un control separado de octubre de 2026, con los CSV intactos, debería sumar `F001: 4400`, `F002: 7800`, `F003: 7100`, `F005: 4600`; no usar ese mes fijo en la implementación.

Al implementar, agregar casos controlados de igualdad con 50 unidades y $5.000, proveedor inactivo con producto activo, empate del ranking, farmacéutico inactivo con ingreso histórico del mes y ventas en los límites de fecha. Los CSV solos no cubren todos esos bordes.

## Alcance del cambio realizado

El código está en `entregas/tpo-farmared/src/consultas/q01.py`, `q05.py`, `q07.py`, `q08.py`, `q09.py` y `q11.py`. Se actualizaron los textos de `src/consultas/__init__.py` y `__main__.py`: `python3 -m consultas` solo enumera, no ejecuta. El README del grupo incluye ejecución y pruebas; `docs/informe-consultas-lectura.md` contiene los ejemplos y decisiones. `scripts/cargar.py` prepara la vista de Q11, y la suite comprueba que la carga y la vista sean repetibles.

Registrar luego el avance real en [avances-tpo.md](avances-tpo.md). Los registros propios del grupo se guardan en `datasets/extra/`, sin editar los CSV provistos. Antes de recargar una base existente, recordar que `scripts/cargar.py` elimina y reconstruye las ocho colecciones: borra altas y ventas creadas durante pruebas y revierte reservas completadas.

En la preparación inicial se verificaron el clon, la exclusión del Git de la wiki y los CSV, y se calcularon los controles sin levantar motores. Después se implementaron las consultas y se verificaron con MongoDB real, según el criterio de terminado al final de esta página.

## Plan de implementación fundamentado

Plan solicitado y luego ejecutado el 10 de noviembre de 2026. Se conserva como fundamento de la implementación; los conceptos se verificaron en T14, T15, T13, P10A y P11A, además de releer P18 completo. Las decisiones propuestas a continuación quedaron implementadas.

### Modelo y contrato común

Mantener MongoDB y la estructura actual. Una venta contiene sus ítems: el detalle se consulta en el contexto de la cabecera, que es el criterio para embeber de T14. Productos, proveedores, sucursales y farmacéuticos siguen referenciados; separar datos compartidos evita repetirlos en cada documento. La elección aplica los conceptos de embebido y referencia, sin cambiar el esquema para cada consulta. [T14, pp. 2–8, 16, 18, 20]

Cada módulo expondrá `ejecutar() -> list[dict]`, reutilizando `base()`; lista vacía cuando no hay resultados. Mantener los identificadores `id_*`, importes enteros y documentos BSON con fechas sin convertirlos a texto dentro de la consulta. Ordenar las salidas para que las pruebas y los ejemplos sean reproducibles. Son decisiones del proyecto; P18 exige documentar propósito, parámetros, retorno y motor, pero no impone ese formato de retorno. [P18, p. 6]

Las relaciones N:1 se resolverán con `$lookup` contra `_id`, seguido del tratamiento del arreglo relacionado. Q01 devolverá el producto con un subdocumento `proveedor` completo; Q08 devolverá el lote con `producto`, `sucursal` y `proveedor`. La carga ya valida referencias: no crear documentos relacionados ficticios para ocultar una referencia inválida. El concepto de ensamble externo izquierdo está presentado en la materia. [T14, pp. 21–24; P18, pp. 4, 6–7]

### 1. Q09: selección simple

Implementar `find` sobre `ventas` con las tres condiciones simultáneas: `tipo = "Venta libre"`, `total < 5000`, `estado != "Anulada"`. Devolver los documentos de las ventas, ordenados por `_id`. No requiere pipeline ni cruces entre colecciones.

**Fundamento:** igualdad y comparadores combinados por AND seleccionan documentos; `$lt` representa el límite estricto y `$ne` la exclusión. Las pendientes permanecen porque el enunciado excluye solamente anuladas. [P10A, pp. 3, 5; P18, p. 6]

**Pruebas:** `V013` y `V014`; ventas con $4.999 y $5.000; pendiente barata incluida, anulada barata excluida, venta Con receta barata excluida.

### 2. Q01: referencia de producto a proveedor

Pipeline propuesto: `productos → $match activo → $lookup proveedores → documento proveedor → $sort por id`.

Relacionar `productos.id_proveedor` con `proveedores._id`; conservar todos los campos del proveedor. No agregar un filtro de proveedor activo: el enunciado solo exige que esté activo el producto. No hace falta copiar proveedores dentro del catálogo para resolver esta lectura.

**Fundamento:** las referencias separan datos compartidos, y `$lookup` permite ensamblarlos durante la consulta. [T14, pp. 7–8, 16, 18, 21–24; P18, p. 6]

**Pruebas:** once productos con los CSV actuales; producto inactivo excluido; dos productos del mismo proveedor sin multiplicación de filas; producto activo de proveedor inactivo incluido; proveedor con todos sus campos.

### 3. Q08: encadenar referencias

Pipeline propuesto: `stock_lotes → $match unidades < 50 → $lookup productos → $lookup sucursales → $lookup proveedores → $sort por id de lote`.

Resolver proveedor desde `producto.id_proveedor`, no desde el lote. El filtro se aplica al lote antes de armar los documentos relacionados. No sumar unidades de distintos lotes ni filtrar por vencimiento, tipo de sucursal o actividad del producto.

**Fundamento:** P18 define las referencias lote–producto, lote–sucursal y producto–proveedor. Los ensambles recuperan entidades relacionadas y la selección conserva el alcance del enunciado. [P18, pp. 3–4, 6; T14, pp. 21–24]

**Pruebas:** nueve lotes actuales; 49 incluido y 50 excluido; lote con cero; lote vencido; producto inactivo; lote de droguería con stock bajo. Cada fila debe identificar correctamente el producto, la sucursal y el proveedor.

### 4. Q07: ranking sobre ítems embebidos

Pipeline propuesto: `ventas → $match Completada → $unwind items → $group por items.id_producto, sumando items.cantidad → $sort unidades descendente e id ascendente → $limit 5 → $lookup productos → $project salida`.

Devolver `id_producto`, `nombre` y `unidades_vendidas`, manteniendo el orden del ranking al terminar el pipeline. Resolver nombres después de reducir a los cinco productos. No desplegar `items.lotes`: son trazabilidad de stock, no nuevas unidades vendidas.

**Fundamento:** la solución de e-commerce ya combina `$unwind`, `$group`, `$sum` de cantidades, `$sort` y `$limit` para el producto más vendido. Adaptamos ese patrón a cinco resultados y al estado exigido por FarmaRed. [P11A, pp. 2–3; P18, p. 6]

**Pruebas:** ranking de control; varias líneas del mismo producto acumuladas; pendiente y anulada excluidas; producto hoy inactivo con venta histórica incluida; empate en el quinto puesto; menos de cinco productos; ningún producto vendido. El desempate por id es una decisión del proyecto, no una regla oficial.

### 5. Q05: conteo con conservación de los ceros

Pipeline propuesto: `farmaceuticos → $match activo → $lookup ventas con filtro temporal y exclusión de anuladas → contar coincidencias → $project datos y cantidad_ventas → $sort por id`.

Partir de los farmacéuticos asegura que el universo de salida incluya a todos los activos. Propuesta técnica: filtrar y contar dentro del ensamble de ventas y convertir la ausencia de coincidencias en cero. No descartar después del ensamble a quienes no tuvieron ventas, ni contar una fila vacía como si fuera una venta.

Calcular una sola vez `hoy()` y usar la convención ya declarada: desde `hoy() - 60 días` inclusivo hasta mañana exclusivo, para incluir todo el día actual y excluir fechas futuras. Devolver `id_farmaceutico`, `nombre`, `apellido`, `matricula` y `cantidad_ventas`. Contar ventas, no ítems; las pendientes cuentan y las anuladas no.

**Fundamento:** un ensamble externo izquierdo conserva el lado inicial; el conteo por grupo es distinto de sumar cantidades. P18 exige explícitamente conservar los ceros. La estrategia concreta de subpipeline y valor cero es una decisión de implementación basada en esos conceptos. [T14, p. 23; T15, p. 33; P18, p. 6]

**Pruebas:** conteos de control con reloj fijado al 10/11/2026; `F007` con cero; `F004` inactivo excluido; pendiente incluida y anulada excluida; fecha exacta del corte, anterior al corte, hoy y mañana; una venta con varios ítems cuenta una sola vez.

### 6. Q11: vista mensual y consulta del mes actual

Crear una vista estándar `ingresos_mensuales_farmaceutico` sobre `ventas`, definida por un pipeline que seleccione Completada, agrupe por `id_farmaceutico`, año y mes de `fecha`, y acumule `total` como `ingresos_totales`. La clave debe incluir año y mes: octubre de dos años distintos no es el mismo período.

`q11.ejecutar()` obtendrá el año y mes actuales con `hoy()`, seleccionará ese período en la vista y resolverá los datos del farmacéutico. Devolverá `id_farmaceutico`, `nombre`, `apellido`, `matricula`, `anio`, `mes` e `ingresos_totales`. Mantener a los farmacéuticos hoy inactivos si tienen ventas completadas en ese período; no inventar filas de ingreso cero.

**Fundamento:** la materia presenta vistas definidas sobre otra colección y consultadas luego; agrupar y sumar es el patrón de agregación. P18 pide una vista agregada y establece que solo Completada computa ingresos. Usar el `total` histórico evita revalorizar ventas con el precio actual del catálogo. [T13, pp. 50–52; T15, p. 33; P18, pp. 4–6]

**Complemento del agente (no consta en el material cargado):** para obtener año y mes usar `$year` y `$month`; una vista estándar calcula su resultado al consultarse. El diseño mensual general más el filtro en `ejecutar()` evita fijar el mes al crearla y permite controlar el reloj desde las pruebas. Verificación técnica en el manual oficial: [`$year`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/year/), [`$month`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/month/) y [vistas](https://www.mongodb.com/docs/manual/core/views/).

Respetar la representación de fechas del proyecto: se guardan componentes de fecha a medianoche sin zona. Extraer año y mes de ese valor tal como está guardado, sin aplicarle otra conversión a Argentina que lo mueva al día anterior. La fecha que determina qué mes consultar sí sale de `hoy()`, que usa Argentina. Esta precaución surge de leer `src/db.py` y la carga; no cambia su contrato.

Integrar la creación o actualización de la definición de la vista al finalizar la carga, antes de la proyección al grafo. Una segunda carga debe funcionar sin conflicto de nombre. Definir la preparación junto a Q11 y llamarla desde `scripts/cargar.py`, sin crear una capa nueva ni modificar Q13–Q15: las escrituras en `ventas` deben reflejarse al volver a consultar la vista.

**Pruebas:** mes vacío; ingresos positivos con datos de prueba; pendiente y anulada excluidas; farmacéutico inactivo con ingresos incluido; venta del mes anterior y del siguiente excluidas; mismo mes de otro año excluido; transición diciembre/enero; cambio de mes del reloj sin recrear la vista; completar una pendiente mediante Q15 y comprobar que sus ingresos entran en el nuevo mes. Verificar también que volver a preparar la vista sea repetible.

### Validación, documentación y alcance

1. Implementar y verificar en este orden: **Q09 → Q01 → Q08 → Q07 → Q05 → Q11**. Avanza de selección a ensambles, agregación, conservación de ceros y vista.
2. Usar pruebas con MongoDB real y una base temporal aislada; controlar `hoy()` en las pruebas. Comprobar resultados y bordes, no limitarse a verificar que el pipeline enviado coincide con una lista escrita en el test. No ejecutar el cargador destructivo contra la base de trabajo para preparar cada prueba.
3. Mantener los CSV originales intactos. Preparar datos de prueba pequeños para los bordes y coordinar con el grupo los diez registros extra por colección exigidos para la entrega. Diferenciar esos extras persistentes de las fixtures temporales de pruebas. [P18, pp. 1, 6]
4. No agregar índices a todos los filtros por anticipado. Si el volumen y `explain()` lo justifican, evaluar primero un índice en `ventas` por `id_farmaceutico` y `fecha` para Q05. Los índices y la inspección de su uso están en la materia; ese índice concreto es una propuesta del proyecto. [P10A, p. 6; T15, pp. 28–29]
5. Actualizar docstrings, el texto desactualizado de `src/consultas/__init__.py`, instrucciones de ejecución en el README del grupo y ejemplos fechados en `docs/informe-consultas-lectura.md`. Conservar el listado actual de consultas: no hace falta construir otro ejecutor.
6. Documentar cada cruce entre colecciones, el desempate del ranking, la convención temporal y el diseño de la vista. Para cada consulta incluir enunciado, pipeline o filtro y al menos un ejemplo de resultado. [P18, pp. 6–7]
7. Actualizar `entregas/avances-tpo.md` y ejecutar `./scripts/verificar-repo.sh` en la wiki. No modificar los motores elegidos, el modelo de las ventas, las escrituras ni el grafo para este alcance; no hacer commit ni push sin pedido explícito.

**Criterio de terminado cumplido:** seis módulos ejecutables y documentados, vista preparada por la carga repetible, 16 pruebas de integración correctas en MongoDB `8.3.11`, ejemplos para el informe y estado actualizado. Se ejecutó además el cargador real y cada consulta con la fecha del sistema; los resultados coincidieron con los controles de esta página. No se añadieron dependencias ni índices. Los extras persistentes y los diagramas siguen siendo pendientes del grupo.
