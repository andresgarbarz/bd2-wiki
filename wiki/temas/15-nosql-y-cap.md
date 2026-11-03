# NoSQL, escalabilidad y CAP

## Motivación y alcance

La unidad relaciona el surgimiento de NoSQL con el crecimiento de la web, el volumen de datos y el aumento de lecturas y escrituras. Presenta modelos no relacionales y estructuras distribuidas como alternativas para cargas con dificultades de escalabilidad. La elección de tecnología debe considerar las necesidades de la aplicación y puede combinar varios motores mediante persistencia políglota. [T13, pp. 2–4, 8–10; C01, pp. 5–6]

El modelo documental permite que los documentos de una colección tengan campos diferentes. Eso expresa flexibilidad de estructura; las afirmaciones generales de las diapositivas sobre ausencia de esquema, rendimiento o imposibilidad de ACID requieren el alcance registrado en [dudas y conflictos](../dudas-y-conflictos.md). [T13, pp. 4, 6–11, 26; T15, p. 23]

## Escalabilidad vertical y horizontal

La [figura de la unidad](../../material/figuras/T13-p9-escalabilidad-vertical-horizontal.png) contrapone aumentar recursos de un servidor con agregar servidores. La distribución horizontal se vincula con el manejo de grandes volúmenes en clusters. En MongoDB, el material distingue sharding para escalabilidad y replicación para disponibilidad. [T13, pp. 9–10, 30; C02, pp. 1–2]

## Cuatro familias

| Familia | Representación presentada | Ejemplos de la taxonomía |
|---|---|---|
| Clave–valor | Duplas de clave y valor, con estructura variable del valor. | Redis, Riak |
| Documentos | Documentos con campos, valores y datos anidados; estructura flexible dentro de la colección. | MongoDB, Couchbase |
| Familias de columnas | Organización por familias/columnas, presentada mediante un contraste visual con filas. | Cassandra, HBase |
| Grafos | Nodos y relaciones entre ellos; el ejemplo representa personas, mensajes y vínculos. | Neo4j |

La clasificación y los ejemplos provienen de la [taxonomía visual](../../material/figuras/T13-p21-taxonomia-nosql.png). Cassandra se desarrolla en [modelado y CQL](20-cassandra-modelado-y-cql.md) y [replicación y consistencia](22-cassandra-consistencia-y-digest.md). Neo4j, Redis y DynamoDB tienen apuntes propios: [grafos y Cypher](23-neo4j-grafos-y-cypher.md), [clave-valor](24-redis-clave-valor.md) y [DynamoDB](25-dynamodb.md). [T13, pp. 21–27; T16, pp. 16, 44–46, 63–69; T17; T18; T20]

## CAP: lectura de cátedra

La presentación introduce tres propiedades: **consistencia** de los datos observados, **disponibilidad** del servicio y **tolerancia a particiones** de red. Expone el compromiso mediante las combinaciones CP, AP y CA: CP puede resignar disponibilidad; AP puede responder con datos inconsistentes; CA se describe bajo el supuesto de no permitir particiones. [T13, pp. 12–17]

El [gráfico de clasificación](../../material/figuras/T13-p18-clasificacion-cap.png), revisado visualmente, ubica MongoDB en CP, Cassandra en AP y MySQL/PostgreSQL en CA. Esta es la clasificación didáctica de esa diapositiva y la referencia para la pregunta 8 del TP 9, parte II. [T13, p. 18; P10B, p. 2]

El ejercicio 3 del TP 10, parte II, pide clasificar Cassandra. Además del gráfico de T13, la clase de Cassandra la describe como tolerante a particiones y disponible a cambio de consistencia eventual, y aclara que el nivel de consistencia se configura incluso por consulta. Ese ajuste se desarrolla en [replicación y consistencia](22-cassandra-consistencia-y-digest.md). [T13, p. 18; T16, p. 11; P13, p. 2]

Las unidades de Neo4j, Redis y DynamoDB no repiten una sola etiqueta. T17 clasifica Neo4j HA como AP; T19 clasifica Redis y Neo4j como CA; la [guía visual](../../material/figuras/T20-p31-guia-visual-cap.png) de T20 ubica Redis en CP, Cassandra en AP y el sistema Dynamo del paper de 2007 en AP. Esas diferencias están en [dudas y conflictos](../dudas-y-conflictos.md). [T17, p. 40; T19, p. 36; T20, pp. 5, 31]

**Complemento del agente (no consta en el material cargado):** el compromiso de CAP se plantea ante una partición: no se pueden garantizar simultáneamente consistencia fuerte y disponibilidad para todas las peticiones. Fuera de esa situación, la regla “elegir dos” oculta posibilidades del diseño. CAP usa consistencia de una copia única, distinta de la preservación de restricciones de la C de ACID; su disponibilidad tampoco es simplemente un porcentaje de uptime. Véanse [Brewer, CAP Twelve Years Later](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/) y [Gilbert y Lynch, Perspectives on the CAP Theorem, sección 2](https://groups.csail.mit.edu/tds/papers/Gilbert/Brewer2.pdf).

Para estudiar o corregir, enunciar primero el criterio de T13 y después explicitar la configuración y las operaciones consideradas cuando se razone sobre un sistema real. No deducir garantías de toda instalación solo por el nombre del motor.

## BASE y ACID

La [figura BASE](../../material/figuras/T13-p19-base.png) desarrolla **Basically Available**, **Soft State** y **Eventual Consistency**. La explicación contrapone priorizar disponibilidad y aceptar estados transitorios con exigir consistencia transaccional. La consistencia eventual admite que distintos usuarios no observen inmediatamente el mismo estado. [T13, pp. 19–20]

**Complemento del agente (no consta en el material cargado):** esa contraposición no permite concluir que MongoDB carezca de transacciones ACID. MongoDB admite transacciones sobre múltiples documentos, incluso en replica sets y clusters fragmentados; sus garantías dependen de los parámetros de lectura y escritura. [Manual oficial: transacciones](https://www.mongodb.com/docs/manual/core/transactions/).

## Continuar

- [Modelado documental](16-mongodb-modelado.md): cuándo embeber y cuándo referenciar.
- [Consultas y CRUD](17-mongodb-consultas-y-crud.md).
- [Agregaciones y vistas](18-mongodb-agregaciones-y-vistas.md).
- [Replicación y sharding](19-mongodb-replicacion-y-sharding.md).
- [Neo4j: grafos y Cypher](23-neo4j-grafos-y-cypher.md).
- [Redis y clave-valor](24-redis-clave-valor.md).
- [DynamoDB](25-dynamodb.md).
