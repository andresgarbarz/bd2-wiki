# Ejercicios tipo parcial — resolución

Fuente: [P14](../../material/catedra/practica/p14-ejercicios-tipo-parcial.pdf).
SQL en [soluciones-sql.sql](soluciones-sql.sql); esquema y datos de prueba de
Investigadores en [investigadores.sql](investigadores.sql).

## Investigadores [P14, p. 1]

Base `parcial` en MySQL. Supuestos: los atributos en negrita cursiva aceptan
nulos; la clave de `Proyecto` es `(IdInvest, fecha_inicio)`, que el enunciado no
marca; "dirigió un proyecto" significa figurar en `Proyecto` con ese `IdInvest`.

Los datos de prueba cubren cada caso borde:

| Investigador | Situación | A | B | C |
|---|---|---|---|---|
| Ana | 3 institutos de Bs. As., uno repetido | sí | no (46 años) | sí |
| Bruno | 3 institutos, uno en Córdoba | no | — | sí |
| Carla | 4 proyectos en solo 2 institutos | no | — | sí |
| Diego | 3 institutos de Bs. As., 25 años | sí | sí | sí |
| Eva | sin proyectos | no | — | sí, con proyecto en `NULL` |
| Fede | cumple 30 mañana | sí | **sí** | sí |
| Gabi | cumple 30 hoy | sí | **no** | sí |

### A — al menos 3 institutos distintos de Buenos Aires

Cuatro tablas encadenadas —investigador, proyecto, instituto, campus— porque la
provincia está en el campus. Se filtra Buenos Aires **antes** de agrupar
(`WHERE`) y se exige la cantidad **después** (`HAVING`). [T05A, pp. 29–35]

**La trampa es `COUNT(*)`.** Cuenta proyectos, no institutos: Carla tiene 4
proyectos en 2 institutos y entraría. `COUNT(DISTINCT p.id_inst)` cuenta
institutos distintos. Resultado: **Ana, Diego, Fede y Gabi**.

### B — los de A con menos de 30 años, usando la vista anterior

Una vista puede consultarse como una tabla, así que B filtra sobre A. [T06, pp. 2–3] "Menos de
30 años" es haber nacido **después** de hoy menos 30 años:
`fec_nac > CURDATE() - INTERVAL 30 YEAR`, equivalente a
`TIMESTAMPDIFF(YEAR, fec_nac, CURDATE()) < 30`. Resultado: **Diego y Fede**.

Dos detalles:

- **Los bordes.** Gabi cumple 30 hoy y queda afuera; Fede cumple mañana y queda
  adentro. Restar años (`YEAR(CURDATE()) - YEAR(fec_nac)`) fallaría para quien
  todavía no cumplió años en el año en curso.
- **La edad no se guarda: se calcula.** `CURDATE()` se evalúa cada vez que se
  consulta la vista, así que la respuesta se mantiene actualizada.

### C — todos los investigadores, con sus proyectos desde 2016

"Todos los investigadores" pide un **`LEFT JOIN`**: Eva no tiene proyectos y
igual tiene que aparecer, con las columnas del proyecto en `NULL`. [T05B, pp. 19–21]

**La trampa es dónde va la fecha.** Con `p.fecha_inicio > '2015-12-31'` en el
`WHERE`, la fila de Eva —que tiene `fecha_inicio` en `NULL`— no cumple la
condición (`NULL > fecha` es desconocido) y se descarta: el `LEFT JOIN` se
vuelve un `JOIN` común. Verificado: en el `WHERE` da 16 filas y 6
investigadores; en el `ON`, **17 filas y los 7**. La condición del `ON` decide
qué proyectos se emparejan, sin eliminar investigadores. [T05B, pp. 19–21, 31; nulos en
[lógica trivaluada](../../wiki/temas/06-nulos-y-logica-trivaluada.md)]

La fecha del enunciado (`31/12/2015`) se escribe `'2015-12-31'`: MySQL compara
fechas en formato ISO.

## Mensajería [P14, p. 1]

Esquema y datos en [mensajeria.sql](mensajeria.sql), en la misma base
`parcial`. El esquema se leyó de la
[figura](../../material/figuras/P14-p1-esquema-mensajes.png).

**Lectura del esquema.** Un mensaje se identifica por `(tipo_mensaje,
cod_mensaje)`. Entre mensaje y adjunto hay **dos** caminos: `CONTIENE` (n:m, qué
adjuntos lleva cada mensaje) y las columnas `tipo_mensaje`/`cod_mensaje` de
`ADJUNTO` (solo el *primer* mensaje que lo incluyó, opcional). "Contienen"
corresponde a `CONTIENE`. `Audio` e `Imagen` son subtipos de `ADJUNTO`, con
`tipo_adj` como discriminador.

**Supuestos:** `anio_creacion` es `date` pese a su nombre, así que se compara su
año con `YEAR(CURDATE())`; `tamanio` está en MB. Registrados en
[dudas y conflictos](../../wiki/dudas-y-conflictos.md#ejercicios-tipo-parcial-p14).

| Mensaje | Adjuntos que cumplen | Lectura literal | Lectura 1–3 |
|---|---|---|---|
| (1,1) | 2 (y uno de 80 MB que no) | sí | sí |
| (1,2) | 4 | no | no |
| (1,3) | 3 (y uno de **exactamente 50**, que no cumple `< 50`) | sí | sí |
| (2,1) | 0 (solo uno del año pasado) | **sí** | no |
| (2,2) | 0 (sin adjuntos) | **sí** | no |

### A — a lo sumo 3 adjuntos del año y de menos de 50 MB

**La ambigüedad.** "A lo sumo 3" incluye el 0. La lectura literal devuelve
también los mensajes sin adjuntos que cumplan; la otra entiende que "si es que
contienen" exige al menos uno. En un parcial conviene **declarar el supuesto** y
resolver la literal, que es la que el enunciado dice.

**La literal requiere dos cuidados**, ambos verificados:

1. **`LEFT JOIN` con las condiciones del adjunto en el `ON`.** Si van en el
   `WHERE`, las filas sin adjunto válido tienen `NULL` en esas columnas, no
   cumplen y desaparecen: el resultado se reduce a (1,1) y (1,3). Es la misma
   trampa del inciso C de Investigadores.
2. **`COUNT(a.id_adjunto)` y no `COUNT(*)`.** Un mensaje sin adjuntos válidos
   llega al `GROUP BY` con una fila de `NULL`: `COUNT(*)` la cuenta como 1 y
   `COUNT(columna)` ignora los nulos y da 0. [T05B, p. 29]

Alternativa equivalente: una subconsulta correlacionada que cuente los adjuntos
válidos de cada mensaje, `WHERE (SELECT COUNT(*) …) <= 3`, que devuelve 0
naturalmente cuando no hay ninguno.

`YEAR(a.anio_creacion) = YEAR(CURDATE())` aplica una función a la columna e
impide usar un índice sobre ella (TP 5). La forma indexable es un rango:
`a.anio_creacion >= MAKEDATE(YEAR(CURDATE()), 1) AND a.anio_creacion <
MAKEDATE(YEAR(CURDATE()) + 1, 1)`. *Complemento del agente.*

### B — datos comunes más duración o resolución

Dos `LEFT JOIN`, uno por subtipo, y un `CASE` sobre el discriminador `tipo_adj`
que elige de qué tabla tomar el valor. `COALESCE(au.duracion, im.resolucion)`
da lo mismo si la jerarquía es exclusiva.

- **Por qué `LEFT JOIN` y no `UNION ALL` de dos `JOIN`.** El enunciado pide los
  datos de **todos** los adjuntos. Si la jerarquía no es total —el esquema no lo
  garantiza—, un adjunto que no es ni audio ni imagen desaparecería con la unión.
  Verificado: el adjunto 104 (documento) sale con `NULL` en la versión con
  `LEFT JOIN` y no aparece en la versión con `UNION ALL`.
- **La columna mezcla magnitudes** (segundos y píxeles): por eso conviene
  devolver también `tipo_adj`, para que quien lee sepa cuál es cuál.

## Nulos en agregados — Voluntarios [P14, p. 2]

`tarea` tiene 20 filas y `min_horas` es `NULL` en todas. Respuesta: **D,
`(null, 20, 0)`**. Verificado en MySQL con una tabla `tarea` de 20 filas.

| Expresión | Valor | Por qué |
|---|---|---|
| `SUM(min_horas)` | `NULL` | los agregados descartan los nulos; sin ningún valor que sumar, el resultado es `NULL`, no 0 |
| `COUNT(*)` | 20 | cuenta **filas**, tengan o no nulos |
| `COUNT(min_horas)` | 0 | cuenta **valores no nulos** de esa columna |

[T05B, p. 29]

Por qué fallan las otras:

- **A `(null, null, null)`**: `COUNT` nunca devuelve `NULL`; en el peor caso, 0.
- **B `(0, 20, 0)`**: confunde "no hay nada que sumar" con "la suma es cero". Si
  se quiere un 0, hay que pedirlo: `COALESCE(SUM(min_horas), 0)`.
- **C `(0, 20, 20)`**: además, `COUNT(min_horas)` no cuenta los nulos.

El mismo `NULL` aparece en `AVG`, `MIN` y `MAX` sobre una columna sin valores, y
en `SUM` sobre una tabla vacía. Es la misma regla del ejercicio de mensajería:
`COUNT(*)` cuenta filas y `COUNT(columna)`, valores.

La misma diapositiva dice que en `ORDER BY` el `NULL` es "el mayor". Eso vale
para PostgreSQL; en MySQL los nulos salen **primero** en orden ascendente.
Registrado en [dudas y conflictos](../../wiki/dudas-y-conflictos.md#posición-de-null-al-ordenar).

## Películas — corrección de una consulta [P14, pp. 2–3]

Se ejecutó sobre `mydb`, el esquema de películas del repositorio [SQL01]; la
consigna usa la notación PostgreSQL `unc_esq_peliculas.<tabla>`.

### a) ¿Responde a lo pedido?

**El filtro es correcto, el resultado no.** La cadena película → renglón →
entrega → nacional con `id_distrib_mayorista IS NULL` selecciona exactamente las
películas pedidas. Pero falla en tres puntos:

1. **`SELECT *` devuelve las columnas de las cinco tablas**: 22 columnas, con
   `codigo_pelicula`, `nro_entrega` e `id_distribuidor` repetidas. Se pidieron
   los datos de las películas: `p.*`.
2. **Repite películas.** Cada película aparece una vez por cada renglón de
   entrega que cumple. Los datos de ejemplo no tienen ningún nacional sin
   mayorista, así que la consulta devuelve 0 filas; dentro de una transacción
   revertida, sacándoles el mayorista a los distribuidores 3 y 5, devuelve
   **4 filas para 2 películas**.
3. **`LIKE 'Drama%'` es más amplio que "género Drama"**: también acepta
   cualquier género que empiece así, como "Dramedia". Lo correcto es
   `genero = 'Drama'`.

Además, el `JOIN` con `distribuidor` no aporta nada: `nacional` ya se une por
`e.id_distribuidor`. No cambia el resultado, pero sobra.

### b) Otra forma

Lo pedido es un **semi-ensamble**: películas para las que *existe* una entrega
de un nacional sin mayorista. `EXISTS` lo expresa directamente y devuelve cada
película una sola vez, con solo sus columnas. [T05B, p. 16]

```sql
SELECT p.*
FROM pelicula p
WHERE p.genero = 'Drama'
  AND EXISTS (SELECT 1
              FROM renglon_entrega re
              JOIN entrega e  ON e.nro_entrega = re.nro_entrega
              JOIN nacional n ON n.id_distribuidor = e.id_distribuidor
              WHERE re.codigo_pelicula = p.codigo_pelicula
                AND n.id_distrib_mayorista IS NULL);
```

Alternativas válidas: `p.codigo_pelicula IN (subconsulta)`, o los mismos joins
con `SELECT DISTINCT p.*`. Verificado: las tres devuelven Shawshank Redemption y
Forrest Gump, una vez cada una.

## Teoría [P14, p. 3]

### A — ¿Qué es un SGBD?

Un **sistema gestor de bases de datos** es una colección de datos
interrelacionados —la base de datos— más un conjunto de programas para
definirlos, almacenarlos, consultarlos y modificarlos. Su objetivo es guardar y
recuperar grandes volúmenes de información de forma práctica y eficiente.
[T01, p. 3]

Para un parcial conviene agregar **por qué existe**: frente a archivos manejados
por programas sueltos, resuelve redundancia e inconsistencia, dificultad de
acceso, aislamiento de datos, integridad, atomicidad, concurrencia y seguridad.
[T01, p. 5] Separa además tres niveles —físico, lógico y de vistas— y ofrece un
lenguaje de definición (DDL) y uno de manipulación (DML). [T01, pp. 6, 8]

### B — Modelo conceptual y elementos de un DER

Un **modelo conceptual de datos** es un conjunto formal y consistente de
enunciados que describe la información relevante de una parte del mundo real,
independientemente de cómo se va a almacenar. Es la primera etapa del diseño:
conceptual → lógico → físico. El más usado es el modelo de entidades y
relaciones de Chen, y su versión extendida. [T02, pp. 5–7]

Elementos de un DER:

| Elemento | Qué es |
|---|---|
| **Entidad** (fuerte o débil) | objeto real o abstracto sobre el que interesa guardar información |
| **Atributo** | propiedad de una entidad o relación, con un dominio; puede ser obligatorio u opcional, univaluado o multivaluado, simple o compuesto, nativo o derivado |
| **Identificador** | atributo o conjunto de atributos que distingue cada instancia |
| **Relación** | asociación entre entidades, con grado (unaria, binaria, ternaria) y cardinalidades mínima y máxima de cada lado; puede tener atributos |
| **Jerarquía (ES-UN)** | supertipo con subtipos |

[T02, pp. 8–35]

### C — Jerarquía exclusiva frente a compartida

- **Exclusiva:** cada instancia del supertipo pertenece **a lo sumo a un**
  subtipo. Se marca con un atributo discriminante (`<Tipo>`). En el ejemplo de
  la cátedra, un producto es sólido **o** líquido.
- **Compartida (solapada):** una instancia puede pertenecer **a varios** subtipos
  a la vez. En el ejemplo, un líquido puede ser envasado **y** a granel.

Es independiente de la otra pregunta de toda jerarquía: si es **total** (toda
instancia del supertipo está en algún subtipo) o **parcial**. [T02, p. 35]

Al derivar al modelo lógico: una tabla para el supertipo y una por subtipo, con
la misma clave; en las exclusivas, el discriminante va en la tabla del
supertipo. [T03, p. 19] El adjunto de mensajería de este mismo parcial es un
ejemplo: `tipo_adj` es el discriminante.

### D — Entidad débil y su derivación

Una **entidad débil** solo puede identificarse en el contexto de otra entidad,
la fuerte o propietaria. Tiene **dependencia de existencia** (no existe sin su
fuerte) y **de identificación** (su clave parcial solo distingue dentro de su
fuerte). Se vincula con ella por una relación identificatoria con cardinalidad
(1,1) del lado de la fuerte: cada instancia débil tiene exactamente una dueña.
[T02, pp. 12, 34]

**Derivación al modelo lógico:** la tabla de la entidad débil lleva como clave
primaria su **clave parcial más la clave de la fuerte**, y esa parte heredada es
también **clave foránea** hacia la fuerte. [T03, p. 22] En el ejemplo de la
cátedra, `PARCELA(NroParcela, IdCampo, …)` con PK `(NroParcela, IdCampo)` e
`IdCampo` FK a `CAMPO`: la parcela 3 solo tiene sentido dentro de un campo.

La dependencia de existencia suele reforzarse con `ON DELETE CASCADE` en esa
clave foránea, como en el TP 6: si se borra el campo, se borran sus parcelas.
*Complemento del agente.*

## Roles y privilegios [P14, p. 3]

Preparación y resolución en [privilegios.sql](privilegios.sql): base `tienda`,
cuentas `p14_admin`, `p14_u1`–`p14_u3` (el prefijo evita chocar con las del
TP 8). ADMIN tiene `ALL` sobre la base con `WITH GRANT OPTION`. Cada sentencia se
ejecutó con la cuenta que corresponde.

| Inciso | Quién | Sentencia |
|---|---|---|
| A | ADMIN | `GRANT SELECT ON remera TO U1 WITH GRANT OPTION;` |
| B | ADMIN | `GRANT INSERT, UPDATE ON cliente TO U2;` y `GRANT INSERT, UPDATE ON ventas TO U2;` |
| C | ADMIN | `GRANT INSERT ON vendedor TO U2 WITH GRANT OPTION;` |
| C | U2 | `GRANT INSERT ON vendedor TO U3;` |
| D | ADMIN | `REVOKE UPDATE ON ventas FROM U2;` |
| E | ADMIN | `CREATE VIEW remeras_deportivas AS …;` y `GRANT SELECT ON remeras_deportivas TO U3;` |

**"Asegúrese que todas las operaciones sean permitidas"** apunta a tres cosas:

1. **C esconde un paso.** Para que U2 autorice a U3, U2 tiene que **tener**
   `INSERT` sobre `VENDEDOR` **con `WITH GRANT OPTION`**, y nadie se lo dio en
   A ni en B. Primero lo concede ADMIN; recién después U2 lo delega. Es el ítem
   2.c del TP 8. Verificado: sin el privilegio, el `GRANT` de U2 falla con
   `INSERT, GRANT command denied`; con `INSERT` pero sin WGO, con `GRANT command
   denied`; con WGO, funciona. Como efecto, U2 también puede insertar en
   `VENDEDOR`: la capacidad de delegar no viaja sin el privilegio.
2. **B son dos sentencias.** `GRANT` admite **un solo objeto**:
   `ON cliente, ventas` da error de sintaxis (`ERROR 1064`). Los dos privilegios
   sí pueden ir juntos: `INSERT, UPDATE`.
3. **E necesita que la vista exista** y las cuentas también: desde MySQL 8,
   `GRANT` no crea usuarios (TP 8). La definición de `RemerasDeportivas` no
   está en el enunciado; se supuso un filtro por categoría.

**D** quita solo `UPDATE` sobre `VENTAS`: U2 conserva `INSERT` sobre `VENTAS` y
todo lo de `CLIENTE` y `VENDEDOR`. Como U2 no había delegado ese `UPDATE`, no
hay cascada que analizar. [T11, pp. 10, 12]

**E muestra para qué sirve una vista en seguridad.** U3 puede consultar
`remeras_deportivas` y recibe solo las dos remeras deportivas y tres columnas,
pero un `SELECT` directo sobre `remera` falla (`ERROR 1142`): la vista se
ejecuta con los permisos de quien la definió. Es la forma de dar acceso a
**algunas filas y columnas** de una tabla sin dar la tabla. [T06, pp. 2–3]
*La ejecución con los permisos del definidor (`SQL SECURITY DEFINER`, el valor
por defecto de MySQL) es complemento del agente.*

El título dice "roles", pero ninguna acción los pide. Si hubiera varios usuarios
con los mismos permisos, convendría agruparlos en un rol (TP 8, ítems g–j).

## NoSQL [P14, p. 4]

### MongoDB — traducir `GROUP BY` con `ORDER BY COUNT(*) DESC`

```sql
SELECT tipo, COUNT(*) FROM hospitales GROUP BY tipo ORDER BY COUNT(*) DESC
```

```javascript
db.hospitales.aggregate([
  {$group: {_id: '$tipo', cantidad: {$sum: 1}}},
  {$sort: {cantidad: -1}}
])
```

Cada cláusula es una etapa: `GROUP BY tipo` → `$group` con `_id: '$tipo'` (la
clave de agrupación, con `$` porque es el valor del campo); `COUNT(*)` →
`$sum: 1`, que suma 1 por documento; `ORDER BY … DESC` → `$sort` con `-1`. Es
casi literalmente el ejemplo de la cátedra, que agrupa por
`'$properties.TIPO'` porque allí los hospitales están guardados como GeoJSON.
[T15, p. 33]

Para que la salida diga `tipo` en lugar de `_id`, se agrega
`{$project: {_id: 0, tipo: '$_id', cantidad: 1}}`; y conviene un desempate,
`{$sort: {cantidad: -1, _id: 1}}`, porque `ORDER BY COUNT(*)` tampoco ordena
los empates. Verificado sobre una colección de prueba (`parcial.hospitales`):
General 3, Pediátrico 2, Maternidad 1.

### Neo4j — la pregunta detrás de la consulta

```
MATCH (:User {name: 'Mary'})-[:FRIENDS_WITH*2..3]-(friend_of_friend:User)
WHERE NOT (:User {name: 'Mary'})-[:FRIENDS_WITH]-(friend_of_friend)
RETURN DISTINCT friend_of_friend.name
```

La lectura usa los patrones de camino de T17: una relación de longitud variable y el filtro que deja afuera el vínculo directo. No se ejecutó en un motor Neo4j. [T17, pp. 13, 25–26; P14, p. 4]

**En lenguaje natural:** "¿Qué usuarios están a dos o tres pasos de amistad de
Mary —amigos de sus amigos, o amigos de esos— pero todavía no son amigos
directos suyos?" Es una **recomendación de amistades**: "personas que quizás
conozcas".

Cómo se lee cada parte:

| Fragmento | Significa |
|---|---|
| `(:User {name: 'Mary'})` | un nodo de tipo `User` con nombre Mary |
| `-[:FRIENDS_WITH*2..3]-` | un camino de **2 o 3** relaciones `FRIENDS_WITH`; sin flecha, en cualquier dirección |
| `(friend_of_friend:User)` | el usuario al final de ese camino |
| `WHERE NOT (…)-[:FRIENDS_WITH]-(…)` | que **no** haya una amistad directa entre Mary y él |
| `RETURN DISTINCT` | cada nombre una vez: a la misma persona se puede llegar por varios caminos |

**Un caso borde:** Mary misma puede aparecer. Si dos amigas de Mary son amigas
entre sí, el camino Mary → A → B → Mary tiene 3 relaciones distintas y termina
en Mary, que no es "amiga directa" de sí misma. Se corrige ligando a Mary a una
variable y excluyéndola: `MATCH (mary:User {name: 'Mary'})-[…]-(f:User) WHERE f
<> mary AND NOT (mary)-[:FRIENDS_WITH]-(f)`.

### Cassandra — crear un keyspace

```sql
CREATE KEYSPACE tienda
  WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};
```

Un keyspace **exige** la opción `replication`: la estrategia y la cantidad de
copias de cada dato. [P12, p. 1; T16, pp. 59–61] Con varios data centers se usa
`NetworkTopologyStrategy` y un factor por data center:
`{'class': 'NetworkTopologyStrategy', 'dc1': 3, 'dc2': 2}`. Opcionalmente,
`IF NOT EXISTS` evita el error si ya existe.

Verificado en el laboratorio de un nodo: el keyspace con factor 3 se crea, con
la advertencia "Your replication factor 3 … is higher than the number of nodes
1". Con un solo nodo, una escritura con `CONSISTENCY QUORUM` falla
(`Cannot achieve consistency level QUORUM`: necesita 2 réplicas y hay 1) y con
`ONE` funciona. Es la base del ejercicio adicional de replicación y
consistencia.

## Adicionales MongoDB — colección `productos` [P14, p. 5]

Datos de prueba en [productos.js](productos.js): el documento del enunciado
(101) más seis que cubren casos borde —marcas en otro orden, un nombre en
minúscula, stock 0, un proveedor con un campo extra, un producto sin `marcas`—.

```javascript
// A) nombre y precio de Informática con precio < 1.000.000, sin _id
db.productos.find({categoria: 'Informática', precio: {$lt: 1000000}},
                  {_id: 0, nombre: 1, precio: 1})
// B) nombre que empieza con "Note" y stock > 0
db.productos.find({nombre: /^Note/, stock: {$gt: 0}})
// C) "Lenovo" e "Intel" simultáneamente en marcas
db.productos.find({marcas: {$all: ['Lenovo', 'Intel']}})
// D) proveedor de Buenos Aires
db.productos.find({'proveedor.provincia': 'Buenos Aires'})
// E) precio +10 % del 101
db.productos.updateOne({_id: 101}, {$mul: {precio: 1.1}})
// F) eliminar los de stock 0
db.productos.deleteMany({stock: 0})
// G) índice ascendente sobre categoria
db.productos.createIndex({categoria: 1})
// H) por categoría: cantidad y precio promedio
db.productos.aggregate([
  {$group: {_id: '$categoria', cantidad: {$sum: 1}, precio_promedio: {$avg: '$precio'}}}
])
```

Las trampas, todas verificadas:

- **B — `^` y mayúsculas.** `/^Note/` exige que el nombre **empiece** con
  "Note", con esa mayúscula: devuelve 101 y 102. "notepad de papel" no entra; con
  `/^note/i` sí. Sin el `^`, `/Note/` aceptaría "Note" en cualquier posición.
  [P10A, p. 4; T15, p. 21]
- **C — `$all` y no igualdad de arreglo.** `{marcas: ['Lenovo', 'Intel']}`
  pide el arreglo **exacto, en ese orden y sin más elementos**: devuelve solo el
  101, y con el orden invertido, nada. `$all` pide que **contenga** ambos, en
  cualquier orden y aunque haya otros: devuelve 101 y 102 (`['Intel', 'Nvidia',
  'Lenovo']`). [T15, p. 22]
- **D — notación de punto y no subdocumento.** `{proveedor: {provincia: 'Buenos
  Aires'}}` compara el subdocumento **entero**: como todos los proveedores
  tienen también `nombre`, no devuelve **ninguno**. `'proveedor.provincia'`
  compara solo ese campo: 101, 103, 104 y 107. [P10A, p. 6]
- **E — `$mul` y no `$inc`.** `$inc` suma una cantidad fija; un porcentaje es
  una multiplicación. 950000 × 1.1 queda guardado como `1045000.0000000001`,
  porque 1.1 no tiene representación binaria exacta. Para dinero conviene
  redondear —`updateOne({_id: 101}, [{$set: {precio: {$round: [{$multiply:
  ['$precio', 1.1]}, 2]}}}])` da 1045000— o usar `Decimal128`. *`$mul`,
  `$round` y `Decimal128` son complemento del agente.*
- **F — `deleteMany`, no `deleteOne`.** Borra todos los que cumplen (103 y 107);
  `deleteOne` borraría solo el primero. Un producto **sin** campo `stock` no se
  borra: `{stock: 0}` no coincide con un campo ausente.
- **G — efecto del índice.** Las búsquedas por igualdad de categoría pasan de
  `COLLSCAN` (recorrer todos los documentos) a `FETCH` sobre `IXSCAN
  categoria_1`: se recorre el B-tree hasta las entradas de esa categoría y solo
  se leen esos documentos. Sirve también para ordenar por categoría y para
  rangos. El costo: cada inserción, borrado o cambio de categoría tiene que
  actualizar el índice. [P10A, p. 6; B01, pp. 110–112]
- **H.** Después de E y F: Informática 2 productos (promedio 1.422.500),
  Librería 1, Telefonía 2.

## Adicionales MongoDB — `$nin` [P14, p. 6]

```javascript
db.productos.find({marcas: {$nin: ['Apple', 'Samsung']}}, {_id: 0, nombre: 1})
```

**A.** Devuelve el **nombre** de los productos que no tienen **ni** "Apple"
**ni** "Samsung" entre sus marcas. Sobre un arreglo, `$nin` exige que
**ningún** elemento esté en la lista: el iPhone (`['Apple', 'Samsung']`) y el
Galaxy (`['Samsung']`) quedan afuera.

**B.** Un documento **sin** el atributo `marcas` **también se devuelve**: `$nin`
selecciona los documentos donde el valor no está en la lista **o el campo no
existe**. [T15, p. 22] Verificado: el "Cable USB", sin `marcas`, aparece. Por
eso la diapositiva advierte no escribir mal el nombre del campo: con un error de
tipeo, `$nin` devuelve todos los documentos.

**C.** Sí, cambia por completo. `$all: ['Apple', 'Samsung']` devuelve los que
tienen **las dos** marcas: solo el iPhone. No es lo opuesto de `$nin`: el
complemento de `$nin` es `$in` ("alguna de las dos": iPhone y Galaxy). Y `$all`
no devuelve documentos sin el campo. Verificado sobre la misma colección:
`$nin` → 101, 102, 103, 104, 107; `$all` → 106; `$in` → 105, 106.

## Adicionales MongoDB — `exports.foods` y `$elemMatch` [P14, p. 6]

Colección de prueba en [paises.js](paises.js), con el dominio del ejemplo de
la cátedra más un caso trampa: Argentina exporta bacon **sabroso** y otro
alimento **no** sabroso. [T15, pp. 23–27]

```javascript
// A) países que exportan "bacon" con tasty false
db.paises.find({'exports.foods': {$elemMatch: {name: 'bacon', tasty: false}}})
// C) eliminar uno solo que cumpla
db.paises.deleteOne({'exports.foods': {$elemMatch: {name: 'bacon', tasty: false}}})
```

**B.** `$elemMatch`. Sin él, `{'exports.foods.name': 'bacon',
'exports.foods.tasty': false}` evalúa cada condición por separado sobre el
arreglo: basta con que **algún** alimento se llame bacon y **algún** alimento
—quizás otro— no sea sabroso. Verificado: sin `$elemMatch` devuelve Canadá
**y Argentina**; con `$elemMatch`, solo Canadá, porque exige que **un mismo
elemento** cumpla las dos condiciones. [T15, pp. 23, 27]

**C.** `deleteOne` borra el primer documento que cumple, aunque haya varios;
`deleteMany` los borraría todos. Es el ejemplo `badBacon` de la diapositiva.
[T15, p. 27]

## Adicionales Cassandra — mediciones de sensores [P14, p. 6]

Registro ejecutable en [sensores.cql](sensores.cql), verificado.

**A–B.** Una tabla dedicada a la consulta: `mediciones_por_sensor_y_dia`.

- **Partition key: `(id_sensor, dia)`.** La consulta da un sensor y un día:
  con los dos, Cassandra va directo a **una** partición.
- **Clustering key: `fecha_hora`, ascendente.** Las mediciones se guardan ya
  ordenadas y la respuesta sale cronológica sin `ORDER BY`.

¿Por qué no solo `id_sensor`? Un sensor mide durante años: su partición
crecería sin límite. Agregar el día la acota y reparte la carga entre nodos. Es
el mismo *bucket* por día de los comentarios del TP 10. [P13, p. 1; T16,
pp. 63–69]

**C–E.** En [sensores.cql](sensores.cql):

```sql
CREATE TABLE mediciones_por_sensor_y_dia (
  id_sensor text, dia date, fecha_hora timestamp,
  tipo text, valor double, ciudad text,
  PRIMARY KEY ((id_sensor, dia), fecha_hora)
) WITH CLUSTERING ORDER BY (fecha_hora ASC);

INSERT INTO mediciones_por_sensor_y_dia (id_sensor, dia, fecha_hora, tipo, valor, ciudad)
  VALUES ('S01', '2026-10-01', '2026-10-01 14:00:00+0000', 'temperatura', 23.1, 'Buenos Aires');

SELECT fecha_hora, tipo, valor, ciudad FROM mediciones_por_sensor_y_dia
  WHERE id_sensor = 'S01' AND dia = '2026-10-01';
```

Verificado: tres mediciones insertadas fuera de orden salen 08:00, 14:00 y
20:00. Un rango dentro del día (`AND fecha_hora >= …`) también funciona; sin
el día, la partition key queda incompleta y la consulta se rechaza.

**F. ¿Por qué no un modelo normalizado con JOIN?** Cassandra **no tiene
joins**: un modelo con `sensor` y `medicion` separados obligaría a la aplicación
a hacer varias consultas y combinarlas. Además, con los datos repartidos por
hash entre nodos, un join tendría que reunir filas de muchos nodos, justo lo que
el diseño busca evitar. Se desnormaliza: `ciudad` y `tipo` viajan en cada
medición y la consulta frecuente lee una sola partición. [T16, pp. 13, 16–19]

## Adicionales Cassandra — keyspace, replicación y consistencia [P14, p. 6]

**A.** `CREATE KEYSPACE sensores WITH replication = {'class': 'SimpleStrategy',
'replication_factor': 3};` Con varios data centers:
`{'class': 'NetworkTopologyStrategy', 'dc1': 3}`.

**B.** Cada dato —cada partición— se guarda en **3 nodos distintos**. Si uno
falla, quedan dos copias. [T16, pp. 59–61]

**C.** **2.** `QUORUM` es la mayoría de las réplicas: ⌊3/2⌋ + 1 = 2. [T16,
p. 45] El coordinador envía la escritura a las 3, pero responde éxito cuando
confirman 2. Verificado al revés en el laboratorio de un nodo: con factor 3,
`QUORUM` falla porque pide 2 réplicas y hay 1 viva.

**D.**

| | `ONE` | `QUORUM` |
|---|---|---|
| Ventaja | menor latencia: alcanza la respuesta más rápida; sigue funcionando con 2 de 3 réplicas caídas | consistencia más fuerte: si lectura y escritura usan `QUORUM`, siempre se solapan en al menos una réplica, y la lectura ve la última escritura confirmada |
| Costo | una lectura puede devolver un dato viejo | más latencia; falla si no hay mayoría disponible |

[T16, pp. 44–46] *El argumento del solapamiento (R + W > N) es complemento del
agente.*

**E.** **No necesariamente.** Con factor 3 y un nodo caído, cada partición
conserva 2 réplicas vivas: `ONE` y `QUORUM` siguen funcionando; solo `ALL`
falla. No hay maestro que perder: cualquier nodo vivo coordina. Distinto sería
con factor 1, donde los datos del nodo caído quedan inaccesibles, o si la caída
deja sin mayoría a alguna partición para el nivel pedido. [T16, pp. 11, 36,
41, 44–46]

## Adicionales Cassandra — arquitectura [P14, p. 6]

**A. Peer-to-peer.** Todos los nodos cumplen el mismo papel: no hay un punto
único de falla, se escala agregando nodos y el sistema sigue disponible aunque
caigan algunos. [T16, pp. 12, 25–29]

**B. No hay maestro permanente.** Cualquier nodo puede atender una petición.
"Coordinador" es un rol de cada operación, no un nodo especial. [T16, pp. 12,
25–29]

**C. Coordinador.** Es el nodo que recibe la petición del cliente: determina
con la partition key qué réplicas tienen el dato, les reenvía la operación,
espera tantas respuestas como exija el nivel de consistencia y devuelve el
resultado. No necesita guardar el dato. [T16, pp. 35–37]

**D. Gossip.** Protocolo por el que los nodos intercambian periódicamente
información de estado entre pares. Así cada nodo conoce la composición del
clúster y detecta qué nodos cayeron, sin un servidor central. [T16, pp. 25–29]

**E. Varios data centers.** Réplicas en distintos lugares físicos: tolerancia a
la caída de un centro entero y lecturas más cercanas a los usuarios de cada
región, con niveles como `LOCAL_QUORUM` que se resuelven dentro de un centro.
[T16, pp. 4, 45–46]

## Adicionales Cassandra — escritura y almacenamiento [P14, p. 7]

**A.** **Commit log → MemTable → SSTable.** La escritura se registra en el
commit log, se aplica en la MemTable y, cuando esta se llena, un *flush* la
vuelca a una SSTable en disco. [T16, pp. 30–33]

**B.** La **MemTable**.

**C.** El **commit log**: si el nodo se cae con datos en la MemTable, al
reiniciar se reaplica el log. Cumple el papel del WAL que viste en recovery.
[T16, pp. 30–33] Ver [recovery y WAL](../../wiki/temas/14-recovery-y-wal.md).

**D.** Porque se escribe una sola vez, completa y en forma secuencial, en el
flush, y nunca se modifica después. Una actualización o un borrado no la
reescribe: se escribe en la MemTable y termina en una SSTable nueva.
[T16, pp. 33–34]

**E.** Escribir es barato porque es secuencial y en memoria: agregar al final
del commit log y actualizar la MemTable, sin leer antes ni buscar dónde estaba
el dato. Leer puede costar más porque una misma partición puede estar repartida
entre la MemTable y **varias** SSTables: hay que consultarlas y combinar las
versiones, quedándose con la de timestamp más reciente. [T16, pp. 33–34, 40]

**F. Compactación.** Fusiona varias SSTables en una: conserva la versión más
reciente de cada dato, descarta las viejas y, pasado el período de gracia, las
tombstones. Así hay menos archivos que consultar en cada lectura y se recupera
espacio. [T16, pp. 33–34, 40]

## Adicionales Cassandra — borrado [P14, p. 7]

**A.** Se escribe una **tombstone**: una marca que dice que el dato está
borrado desde ese momento. El `SELECT` no la muestra y la marca oculta los
valores anteriores. [T16, pp. 38–39]

**B.** Porque las SSTables son **inmutables**: no se puede editar el archivo
para sacar el dato. [T16, pp. 33–34, 38]

**C.** En una **compactación** posterior al período de gracia (10 días por
defecto). [T16, pp. 38–39, 47]

**D.** Si un nodo estuvo caído durante el borrado, no recibió la tombstone y
sigue teniendo el dato. Si vuelve después de que las demás réplicas ya
compactaron y eliminaron la marca, ese dato **reaparece** ("resucita") y se
propaga como si fuera válido. Por eso existe el período de gracia y se
recomiendan *repairs* periódicos. [T16, pp. 38–39, 47] Ya lo viste en vivo en
la parte I del TP 10.

## Adicionales Cassandra — Bloom filters [P14, p. 7]

**A. Verdadero.** Un Bloom filter **no da falsos negativos**: si dice que la
clave no está, seguro no está, y la SSTable se descarta sin leerla.
[C04, pp. 5–7]

**B. Falso.** "Puede estar" no es certeza: los bits consultados pudieron
haberlos encendido otras claves (**falso positivo**). Hay que leer la SSTable
para confirmar. [C04, pp. 5–6]

**C. Verdadero.** Es su propósito: evitar abrir SSTables que no tienen la
clave. Un falso positivo solo agrega una lectura innecesaria; nunca se pierde un
dato. [C04, pp. 1–2, 7]
