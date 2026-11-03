# Neo4j: grafos de propiedad y Cypher

## Qué representa

Neo4j guarda datos estructurados como un grafo, no como tablas. La clase lo presenta como motor transaccional, con almacenamiento y procesamiento de grafos nativos, escrito en Java. El modelo que usa es el **grafo de propiedad**: nodos y relaciones pueden llevar etiquetas y propiedades, y las relaciones pueden además tener un peso. [T17, pp. 1, 4, 15]

| Pieza | Papel en el material |
|---|---|
| Nodo | Objeto del dominio. |
| Relación | Vínculo dirigido entre nodos; puede tener tipo y propiedades. |
| Propiedad | Par clave-valor de un nodo o de una relación. |
| Label | Rol o tipo asignado a un nodo o a una relación. |

La [figura de conceptos](../../material/figuras/T17-p17-grafo-propiedades.png) separa esas cuatro piezas en un ejemplo de personas, empresa y ciudad. Un grafo no dirigido permite leer el vínculo en ambos sentidos; uno dirigido no. [T17, pp. 3–4, 17–18]

La misma clase afirma que una consulta toca el nodo y sus relaciones, y no el grafo entero, y da como techo ilustrativo 34.000 millones de nodos, otras tantas relaciones, 68.000 millones de propiedades y 32.000 tipos de relación. Más adelante el texto del libro citado habla de 34.4 mil millones. Son órdenes de magnitud de la fuente, no una medición de este repositorio. [T17, pp. 6–7, 39]

## Cypher

Cypher describe un patrón de nodos y relaciones. La notación de una arista dirigida es `(a)-[:LIKES]->(b)`, como muestra el [patrón LIKES](../../material/figuras/T17-p9-patron-likes.png). [T17, p. 9]

El recorrido de la clase arma un grafo de vinos, una publicación, una bodega y personas. `CREATE` agrega nodos o relaciones; `MATCH` los busca; `SET` modifica una propiedad, como el `rating` de `reported_on`. Para vaciar el grafo, el material muestra `MATCH (n) DETACH DELETE n`. [T17, pp. 19–25]

El [ejemplo de la red](../../material/figuras/T17-p12-red-amigos.png) pide un camino de amistad hasta alguien que trabajó con Neo4j. La consulta de la diapositiva siguiente usa `shortestPath` con un tramo de longitud variable `[:FRIEND*..5]`. [T17, pp. 12–13]

```cypher
MATCH (you {name:"You"})
MATCH (expert)-[:WORKED_WITH]->(db:Database {name:"Neo4j"})
MATCH path = shortestPath( (you)-[:FRIEND*..5]-(expert) )
RETURN db, expert, path
```

[T17, p. 13]

Otras operaciones que la clase deja escritas:

- Índice por etiqueta y propiedad: `CREATE INDEX FOR (w:Wine) ON (w.name)`.
- Unicidad: `CREATE CONSTRAINT FOR (p:Publication) REQUIRE p.name IS UNIQUE`. El ejemplo siguiente crea un nodo `Person` y el mensaje de error habla de la etiqueta `Person`: el texto de la restricción y el del error no coinciden. Conservar el original y no tratar esa etiqueta como la de la sentencia. [T17, p. 28]
- Distancia entre dos puntos, con un resultado de 87.34 m para el par de coordenadas del ejemplo. [T17, p. 29]
- Metadatos: `CALL db.schema.visualization()` y `CALL db.schema()`, este último marcado como deprecado en 4.x; `CALL db.labels()` y `CALL db.relationshipTypes()` listan etiquetas y tipos. [T17, pp. 30–31]
- La interfaz REST de los ejemplos responde `neo4j_version` 3.4.9 y exige autenticación, salvo desactivarla en `neo4j.conf`. [T17, pp. 31–33]

El anexo precisa tres verbos que la clase de vinos usa sin contrastarlos:

| Verbo | Efecto |
|---|---|
| `CREATE` | Siempre crea un patrón nuevo. |
| `MATCH` | Busca un patrón existente. |
| `MERGE` | Busca el patrón completo y lo crea si no existe. |

`ON CREATE SET` corre solo al crear; `ON MATCH SET`, solo si el patrón ya estaba; un `SET` fuera de esas cláusulas corre en ambos casos. Para una relación, el anexo recomienda resolver primero los dos nodos y después el `MERGE` de la arista, porque `MERGE` opera sobre el patrón completo. `MERGE` no borra duplicados ya existentes: la identidad se protege con una restricción de unicidad y conviene no usar como identidad una propiedad que cambia. [C06, pp. 2–5]

## Dos relatos de distribución

T17 describe el modo HA de la edición Enterprise: un master y slaves, coordinación con ZooKeeper, y slaves que aceptan escrituras y las sincronizan con el master. Si el master cae, un slave asume ese rol. El ejemplo de configuración usa Neo4j Enterprise 3.4.9, tres procesos locales y `dbms.mode=HA`. En CAP, esa arquitectura se clasifica **AP**: cada slave responde con lo que tiene, puede ir atrasado respecto del master y el conjunto es eventualmente consistente. Por eso el texto recomienda HA para cargas de lectura mayoritaria. La debilidad explícita es que HA replica el grafo completo y no parte subgrafos. [T17, pp. 19, 36–40]

El anexo describe otra arquitectura, Causal Clustering. Los **primarios** participan de Raft: uno es líder y los otros seguidores; confirman escrituras por mayoría. Los **secundarios** son réplicas de lectura, no votan y pueden replicar de forma asíncrona. Confirmar una escritura recorre cinco pasos: el cliente la envía, el líder la agrega al log, la replica a los seguidores, la mayoría confirma y entonces se compromete y se aplica. Para tolerar F fallas de primarios hace falta `N ≥ 2F + 1`. Con 3 primarios el quórum es 2 y se tolera 1 falla; si queda un solo primario, no hay mayoría para escrituras nuevas. Raft ordena transacciones y no coordina las lecturas de los secundarios. [C06, pp. 6–9, 13]

Un **bookmark** guarda el punto de progreso de una transacción. La lectura siguiente puede esperar una réplica que ya alcanzó ese punto y así obtener *read your own writes*, aunque el secundario vaya atrasado. [C06, p. 10]

La comparación del anexo: Neo4j en clúster usa primarios con Raft y secundarios de lectura; Cassandra, en la operación habitual del material, no tiene un líder único de escritura y configura el nivel de consistencia por operación. El anexo agrega que Cassandra usa consenso adicional en operaciones puntuales, y da las LWT como ejemplo. [C06, p. 11]

T19, en cambio, dice que Neo4j es CA porque no reparte datos con sharding. Esa frase y la clasificación AP de T17 no dicen lo mismo. El registro está en [dudas y conflictos](../dudas-y-conflictos.md). [T19, p. 36; T17, p. 40]

Las fechas y versiones de T17 tampoco forman una sola línea de tiempo: desarrollo inicial en 2003, disponibilidad pública en 2007, versión 1.0 en febrero de 2010, release estable 5.6 en marzo de 2023, y el ranking citado es el puesto 21 global (19 en abril de 2022) y 1 entre grafos. Java 11 es el requisito que la clase fija para la serie 4.x. [T17, pp. 1, 15, 20]

## TP 11

La guía pide Docker con `neo4j` sin etiqueta de versión, el contenedor `Myneo4j`, puertos 7474 y 7687 y `NEO4J_AUTH=none`. Aclara que el libro *Seven Databases in Seven Weeks* tiene métodos deprecados. Los ejercicios 1 y 2 siguen el capítulo 6 (pp. 181–189 del libro) y el tutorial `:play movies`. El 3 crea dos localidades con latitud y longitud y guarda en la relación `hay_ruta` la distancia calculada. El 4 usa el sandbox *Paradise Papers* y pregunta por esquema, etiquetas, tipos de relación y vecindades de Hutomo Mandala y Saadi Yacef. El 5 pide fortalezas, debilidades y cuándo usarlo; el 6, la clasificación CAP. [P15, pp. 1–2; B01]

La consulta Cypher del ejercicio tipo parcial se lee con estos patrones de camino: amigos a distancia 2 o 3 que no sean amigos directos. [P14, p. 4; T17, pp. 13, 25–26]

## Continuar

- [Redis y clave-valor](24-redis-clave-valor.md).
- [DynamoDB](25-dynamodb.md).
- [CAP](15-nosql-y-cap.md), con las tres clasificaciones de Neo4j y Redis sin unificarlas.
- [Guía TP 11](../../practica/tp-11/README.md).
