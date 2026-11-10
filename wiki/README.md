# Índice de la wiki

Esta es la puerta de entrada al conocimiento curado. Cada afirmación importante remite a una fuente oficial mediante el [catálogo](../material/catalogo.tsv). El texto extraído sirve para buscar; los PDFs siguen siendo la autoridad cuando una página contiene diagramas.

## Ruta sugerida

1. [Introducción a SGBD](temas/01-introduccion-sgbd.md)
2. [Modelo de entidades y relaciones](temas/02-modelo-entidad-relacion.md)
3. [Derivación al modelo relacional y DDL](temas/03-modelo-relacional-y-ddl.md)
4. [Alteración de tablas y DML](temas/04-alteracion-y-dml.md)
5. [Consultas SQL](temas/05-consultas-sql.md)
6. [Nulos y lógica trivaluada](temas/06-nulos-y-logica-trivaluada.md)
7. [Persistencia políglota, Docker y entorno](temas/07-persistencia-poliglota-y-docker.md)
8. [Dialectos SQL presentes en el material](temas/08-dialectos-sql.md)
9. [Vistas](temas/09-vistas.md)
10. [Planes de ejecución](temas/10-planes-de-ejecucion.md)
11. [Integridad y restricciones](temas/11-integridad-y-restricciones.md)
12. [SQL procedural](temas/12-sql-procedural.md)
13. [Seguridad y transacciones](temas/13-seguridad-y-transacciones.md)
14. [Recovery y WAL](temas/14-recovery-y-wal.md)
15. [NoSQL, escalabilidad y CAP](temas/15-nosql-y-cap.md)
16. [MongoDB: modelado documental](temas/16-mongodb-modelado.md)
17. [MongoDB: consultas, CRUD e índices](temas/17-mongodb-consultas-y-crud.md)
18. [MongoDB: agregaciones, vistas y MapReduce](temas/18-mongodb-agregaciones-y-vistas.md)
19. [MongoDB: replicación y sharding](temas/19-mongodb-replicacion-y-sharding.md)
20. [Cassandra: modelado y CQL](temas/20-cassandra-modelado-y-cql.md)
21. [Cassandra: almacenamiento y Bloom filters](temas/21-cassandra-almacenamiento-y-bloom.md)
22. [Cassandra: replicación, consistencia y DIGEST](temas/22-cassandra-consistencia-y-digest.md)
23. [Neo4j: grafos de propiedad y Cypher](temas/23-neo4j-grafos-y-cypher.md)
24. [Redis y bases de clave-valor](temas/24-redis-clave-valor.md)
25. [Amazon DynamoDB](temas/25-dynamodb.md)

## Referencias rápidas

- [Glosario](glosario.md)
- [Dudas y conflictos detectados](dudas-y-conflictos.md)
- [Preguntas de repaso](repaso/preguntas.md)
- [Guías prácticas](../practica/README.md)
- [Esquema de películas](../material/figuras/P03-p1-esquema-peliculas.png)
- [Esquema de envíos del TP 4](../material/figuras/P04-p1-esquema-envios.png)
- [Esquemas del TP 6](../material/figuras/README.md)
- [Esquemas del TP 8](../material/figuras/README.md)
- [Datos de bandas del TP 9](../material/figuras/P10A-p7-datos-bandas.png)
- [DER del e-commerce del TP 10](../material/figuras/P13-p2-der-ecommerce.png)
- [Incorporación de NoSQL/MongoDB](../material/incorporaciones/2026-10-02-nosql-mongodb.md)
- [Incorporación de Cassandra](../material/incorporaciones/2026-10-02-cassandra.md)
- [Incorporación del TP 10, parte II](../material/incorporaciones/2026-10-13-cassandra-tp10-parte-2.md)
- [Ejercicios tipo parcial](../practica/parcial/README.md) y su [incorporación](../material/incorporaciones/2026-10-13-ejercicios-parcial.md)
- [Claves, consistencia y Bloom filters: figuras](../material/figuras/README.md)
- [Mecanismo DIGEST de cátedra](../material/catedra/teoria/recursos/c05-cassandra-digest.png)
- [TPO FarmaRed](../entregas/tpo-farmared/README.md), las [decisiones de arquitectura](../entregas/decisiones-arquitectura.md), los [avances](../entregas/avances-tpo.md), las [decisiones de escrituras](../entregas/decisiones-escrituras.md) y su [incorporación](../material/incorporaciones/2026-11-03-neo4j-redis-dynamodb-tpo.md)

## Cobertura actual

| Tema | Fuentes principales | Estado |
|---|---|---|
| Fundamentos de SGBD | T01 | Curado |
| MER/DERE | T02, P01 | Curado; los diagramas deben verse en PDF |
| Transformación y creación de tablas | T03, P02 | Curado; figuras de P02 extraídas |
| ALTER, INSERT, UPDATE, DELETE | T04 | Curado con equivalencias MySQL |
| SELECT, filtros, agregación | T05A, T05C, P03 | Curado |
| Joins, subconsultas y nulos | T05B | Curado |
| Persistencia políglota y Docker | C01 | Curado |
| Vistas y actualizabilidad | T06, T07, P04 | Curado; capturas visuales revisadas y esquema de P04 extraído |
| Planes de ejecución | T08 | Curado; ejemplos específicos de PostgreSQL |
| Integridad referencial y restricciones declarativas | T09, P06, P06A, P06B | Curado; `MATCH`, `CHECK` y `ASSERTION` distinguidos de MySQL |
| Triggers, procedimientos, funciones y cursores | T09, T10, P07, P09, SQL02, SQL03 | Curado; sintaxis PostgreSQL separada de MySQL |
| Seguridad, transacciones y concurrencia | T11, P08, P08A | Curado; ejemplos de privilegios y dialectos identificados |
| Recovery, WAL, PostgreSQL e InnoDB | T12 | Curado; comparación conceptual por motor |
| NoSQL, familias, escalabilidad, CAP y BASE | T13 | Curado; figuras revisadas y simplificaciones registradas |
| MongoDB: BSON, embebidos y referencias | T13, T14, T15 | Curado; crecimiento y atomicidad del documento |
| MongoDB: CRUD, filtros, arreglos, índices y explain | T13, T14, T15, P10A | Curado; métodos históricos diferenciados de mongosh |
| MongoDB: pipelines, lookup, vistas y MapReduce | T13, T14, T15, C03, P11, P11A | Curado; omisiones de la solución oficial registradas |
| MongoDB: replicación y sharding | T13, T15, C02 | Curado; propósitos y componentes diferenciados |
| TP 9 MongoDB y datos de egresados/ciudades | P10A, P10B, P10C, P10D | Indexado; falta el libro remitido y revisar coordenadas del JSON |
| Cassandra: modelado, claves y CQL | T16, P12, P13 | Curado; columnas históricas separadas del esquema CQL |
| Cassandra: commit log, memtable, SSTable, tombstones y Bloom filters | T16, C04 | Curado; diagramas recuperados y salvedades de durabilidad/compactación |
| Cassandra: replicación, consistencia y DIGEST | T16, C05 | Curado; background read repair histórico y diferencias de versión registradas |
| TP 10 Cassandra, partes I y II | P12, P13 | Indexado; sin resolución ni ejecución; supuestos del modelado de la parte II registrados; versión del entorno no fijada |
| Neo4j: grafos de propiedad, Cypher, HA y Causal Clustering | T17, C06, P15 | Curado; HA y Raft conviven como arquitecturas distintas; CAP de T17 y T19 sin unificar |
| Redis y clave-valor | T18, T19, P16, P16A | Curado; P16A no resuelve la guía; tabla IMDB recuperada de la página |
| DynamoDB | T20, P17 | Curado; el servicio no recibe etiqueta CAP en T20; la guía visual clasifica al paper Dynamo |
| TPO FarmaRed | P18 | Entorno, carga y Q13–Q15 hechos. Q01, Q05, Q07, Q08, Q09 y Q11 implementadas y verificadas en MongoDB real; ver [avances](../entregas/avances-tpo.md). Faltan grafo, Q02, Q03, Q04, Q06, Q10, Q12, extras e informe final. Grupos de 4 y “tres integrantes” sin resolver |
