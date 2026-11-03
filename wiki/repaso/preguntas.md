# Preguntas de repaso

No incluyen respuestas para permitir autoevaluación. Pedile al agente `tomame estas preguntas de a una` para recibir corrección y referencias después de cada intento.

## Fundamentos

1. ¿Qué problemas de un sistema de archivos busca resolver un SGBD?
2. Compará los niveles físico, lógico y de vistas.
3. ¿Qué diferencia hay entre DDL y DML?
4. ¿Qué responsabilidades tienen el gestor de transacciones y el procesador de consultas?

## Modelo conceptual

5. ¿Qué tres propiedades debe satisfacer un conjunto de entidades?
6. Clasificá un teléfono celular repetible según presencia, cardinalidad, rol, composición y origen.
7. Explicá la lectura look-across de una relación 1:N con cardinalidades mínimas.
8. ¿Cuándo una entidad debe ser débil?
9. ¿Qué decisiones describen una jerarquía ISA?
10. ¿Por qué el precio efectivamente vendido debería pertenecer al renglón de factura y no solo al producto?

## Derivación y DDL

11. Derivá una relación binaria 1:N al modelo relacional.
12. Derivá una N:N que posee un atributo propio.
13. ¿Cómo se transforma un atributo multivaluado?
14. ¿Cómo se forma la PK de una entidad débil?
15. ¿Por qué importa el orden de creación de tablas con claves extranjeras?

## SQL

16. ¿Qué diferencia hay entre `WHERE` y `HAVING`?
17. ¿Qué cuentan `COUNT(*)` y `COUNT(columna)` cuando existen nulos?
18. ¿Por qué una consulta sin `ORDER BY` no tiene orden garantizado?
19. Compará `IN`, `EXISTS` y una subconsulta escalar.
20. ¿Cómo puede un filtro en `WHERE` convertir de hecho un `LEFT JOIN` en un inner join?
21. ¿Por qué `NOT IN` puede ser peligroso cuando la subconsulta contiene `NULL`?

## Vistas

22. ¿Qué diferencia hay entre una vista virtual y una vista materializada?
23. ¿Por qué la preservación de clave es estructural y no depende de los datos actuales?
24. ¿Qué condiciones caracterizan a una vista σ-π actualizable?
25. En un ensamble N:1 mediante FK → PK, ¿de qué lado se preserva la clave y por qué?
26. ¿Cómo cambia el análisis de actualizabilidad entre `INSERT`, `UPDATE` y `DELETE` en MySQL?
27. Explicá una migración de tupla y cómo la evita `WITH CHECK OPTION`.
28. Compará `LOCAL` y `CASCADED` en una cadena de dos vistas con predicados distintos.

## Planes de ejecución

29. ¿Qué diferencia operativa existe entre `EXPLAIN` y `EXPLAIN ANALYZE`?
30. ¿Cómo se lee un árbol de ejecución y qué función cumplen sus nodos hoja?
31. ¿Qué indica una gran diferencia entre las filas estimadas y las reales?
32. Compará `Seq Scan`, `Index Scan` e `Index Only Scan`.
33. ¿Por qué un índice compuesto por `(apellido, nombre)` puede no servir para filtrar solo por `nombre`?
34. ¿Qué información agrega la opción `BUFFERS` y por qué una única medición temporal puede engañar?

## Integridad y restricciones

35. Compará una RI inherente, implícita y explícita.
36. ¿Qué diferencia una RI de estado de una RI de transición?
37. Explicá el efecto de `RESTRICT`, `CASCADE`, `SET NULL` y `SET DEFAULT` sobre una FK.
38. Para una FK compuesta con un componente nulo, compará `MATCH SIMPLE`, `PARTIAL` y `FULL`.
39. ¿Por qué `CHECK (nota BETWEEN 0 AND 10)` no impide por sí solo una nota nula?
40. Clasificá una regla de atributo, una de tupla, una entre filas y una global, e indicá el recurso declarativo teórico.
41. ¿Por qué un trigger agregado hoy no garantiza que los datos cargados ayer satisfagan la regla?
42. ¿Qué partes de `MATCH`, `CHECK` y `ASSERTION` del SQL estándar no pueden trasladarse literalmente a MySQL?

## SQL procedural

43. Compará trigger, procedimiento almacenado y función por forma de invocación y resultado.
44. Describí un trigger como regla evento–condición–acción.
45. Compará `BEFORE`, `AFTER` e `INSTEAD OF`.
46. ¿Cómo cambia el resultado de un trigger `FOR EACH ROW` frente a uno `FOR EACH STATEMENT` cuando una sentencia afecta tres filas?
47. ¿Qué valores permiten consultar `OLD` y `NEW` en `INSERT`, `UPDATE` y `DELETE`?
48. ¿Cuándo conviene una RI declarativa y cuándo un trigger?
49. Enumerá el ciclo de vida de un cursor y explicá cómo se detecta el fin en MySQL.

## Seguridad

50. Diferenciá autenticación, autorización y cifrado.
51. Compará integridad, disponibilidad y confidencialidad como objetivos de seguridad.
52. ¿Qué habilita `WITH GRANT OPTION` y cómo afecta una revocación posterior?
53. ¿Qué diferencia hay entre conceder un privilegio directamente y concederlo mediante un rol?
54. Dibujá el grafo de permisos del ejercicio 1 del TP 8 y justificá cada arista que sobrevive a una revocación.

## Transacciones y concurrencia

55. Explicá cada propiedad ACID con una falla que la pondría en riesgo.
56. Recorré los estados posibles de una transacción desde activa hasta confirmada o abortada.
57. Compará lectura sucia, lectura no repetible y lectura fantasma.
58. ¿Qué gana el sistema con concurrencia y qué responsabilidad agrega al SGBD?
59. Compará bloqueo, control optimista de versiones y ordenamiento por timestamps.
60. Ordená los cuatro niveles de aislamiento y explicá el compromiso entre concurrencia y anomalías.

## Recovery y WAL

61. ¿Qué problema crean las páginas sucias del buffer pool para atomicidad y durabilidad?
62. Enunciá la regla WAL y explicá por qué el log debe forzarse antes de confirmar.
63. ¿Qué información representa un LSN y cómo se relaciona con `PageLSN`?
64. Describí las fases de análisis, redo y undo de ARIES.
65. Compará WAL de PostgreSQL con redo log, undo log, doublewrite buffer y binlog de InnoDB.
66. ¿Por qué redo log y binlog no son intercambiables en MySQL?

## NoSQL, CAP y MongoDB

67. Compará escalabilidad vertical y horizontal usando el esquema de T13, p. 9.
68. Distinguí las cuatro familias NoSQL de la unidad y ubicá MongoDB y Cassandra.
69. Explicá C, A y P; ¿qué resignan CP y AP en el escenario de partición?
70. ¿Cómo clasifica la cátedra MongoDB y qué supuestos hay que explicitar al analizar una instalación real?
71. Desarrollá BASE y distinguí consistencia eventual de consistencia transaccional.
72. ¿Por qué esquema flexible no significa ausencia de diseño del modelo?
73. Compará embebidos y referencias según consultas, duplicación, crecimiento y atomicidad.
74. En el ejemplo editor–libros, ¿por qué puede convenir `publisher_id` en cada libro frente a `books` en el editor?
75. Distinguí documento, colección, BSON y `_id`.
76. Armá un filtro que combine AND implícito, `$or` y un rango; explicá `$exists`.
77. Compará `$all`, `$nin` y `$elemMatch`; ¿qué ocurre con campos ausentes en el ejemplo de `$nin`?
78. Compará `$set`, `$inc` y `$push`; explicá la forma del tercer argumento para upsert.
79. ¿Qué cambia entre `updateOne` y `updateMany`, o `deleteOne` y `deleteMany`?
80. ¿Cómo se excluye `_id` de una proyección y cómo se combinan sort, skip y limit?
81. ¿Qué índice predeterminado describe el material y cómo comprobarías si una consulta usa un índice?
82. Compará `$sum: 1` con `$sum` de un campo y explicá el papel de `_id` en `$group`.
83. Describí `from`, `localField`, `foreignField` y `as` en `$lookup`.
84. ¿Cómo cambia la cantidad de documentos al aplicar `$unwind` sobre un arreglo?
85. ¿Cómo define T13 una vista y para qué pide `bandas_resumen` el TP 9?
86. Explicá map, emit y reduce usando el ejemplo de ventas; distinguí el concepto del estado actual del método MongoDB.
87. Compará replicación y sharding por objetivo, distribución de datos y componentes.
88. ¿Cómo se combinan shards y replica sets? ¿Qué muestra el esquema de elección de primario?
89. Antes de importar P10C y P10D, ¿qué revisarías sobre cabecera, formato, tipos y orden de coordenadas?

Fuentes para corregir el bloque: [T13, pp. 9, 12–30, 41–52; T14, pp. 2–24, 28; T15, pp. 8–9, 21–29, 31–44; C02, pp. 1–2; C03, pp. 1–2; P10A, pp. 2–7; P10B, pp. 1–2; P10C; P10D]. Las precisiones externas necesarias están identificadas en los apuntes y en dudas.

## Cassandra, Bloom filters y DIGEST

90. ¿Por qué el diseño de tablas Cassandra parte de las consultas y qué implica que no existan joins?
91. Distinguí cluster, data center, keyspace y familia de columnas.
92. Compará `PRIMARY KEY (a, b, c)` con `PRIMARY KEY ((a, b), c)` por partición, identificación y orden.
93. En la playlist del TP 10, ¿por qué se repite `id` y qué aporta `nro_cancion`?
94. Explicá el rechazo de una consulta que solo restringe `mes` cuando la partition key es `(idusuario, mes)`.
95. Compará acceso por partition key, índice secundario y `ALLOW FILTERING`; ¿qué costo puede ocultar el último?
96. Recorré una escritura local desde commit log hasta SSTable. ¿Cómo se recupera la memtable tras una caída?
97. ¿Por qué pueden existir datos de una partición en varias SSTables y qué mejora la compactación?
98. Explicá tombstone y período de gracia; ¿qué problema crea una réplica que no recibe un borrado?
99. Reproducí el vector de Juan y explicá el rechazo de Pedro en C04. ¿Dónde aparece el bit decisivo?
100. ¿Por qué un Bloom filter admite falsos positivos pero no falsos negativos? ¿Devuelve el dato?
101. Separá el hash del partitioner, los hashes del Bloom filter y el digest entre réplicas por propósito.
102. Distinguí RF y CL: con RF=3 y escritura ONE, ¿se envía la escritura a una sola réplica?
103. Explicá ONE, QUORUM y ALL; ¿qué cantidad representa una mayoría con RF=3 y RF=4?
104. Recorré C05: dato completo, digest, comparación y timestamp. ¿Qué paso adicional necesita un digest diferente?
105. ¿Qué salvedad hay que hacer sobre background read repair al usar Cassandra 4.0 o posterior?
106. ¿Por qué read repair no equivale al mantenimiento periódico `repair` ni garantiza consultar siempre todas las réplicas?
107. En el TP 10, distinguí borrar una canción de la playlist, toda la playlist, la tabla y el keyspace.
108. Identificá la contradicción sobre agregaciones de T16 y explicá por qué sus ejemplos no deben copiarse sin revisar tipos y versión.
109. ¿Qué dos objetivos fija el TP 10, parte II, antes de diseñar las tablas y por qué pueden entrar en tensión al elegir la partition key?
110. En el blog de noticias del TP 10, ¿qué consultas combinan datos de dos entidades y cómo se resuelven sin joins?
111. ¿Qué supuestos del DER de e-commerce conviene declarar antes de modelarlo en Cassandra?
112. ¿Cómo clasifica la cátedra a Cassandra según CAP y qué matiz agrega el nivel de consistencia configurable?

Fuentes para corregir el bloque: [T13, p. 18; T16, pp. 11–19, 25–47, 51, 56, 59–74; C04, pp. 3–8; C05; P12, pp. 1–3; P13, pp. 1–2]. Las precisiones de versión, tipos y reparación que no constan en las fuentes están etiquetadas como complemento en los [apuntes](../temas/22-cassandra-consistencia-y-digest.md) y en [dudas](../dudas-y-conflictos.md).

## Neo4j, Redis y DynamoDB

113. ¿Qué piezas tiene un grafo de propiedad y qué guarda cada una?
114. Compará `CREATE`, `MATCH` y `MERGE`. ¿Cuándo corren `ON CREATE SET` y `ON MATCH SET`?
115. ¿Por qué una restricción de unicidad sigue haciendo falta aunque se use `MERGE`?
116. En Causal Clustering, ¿quién vota en Raft y qué hacen los secundarios? Con 3 primarios, ¿cuántas fallas tolera el quórum?
117. ¿Qué evita un bookmark en una lectura hecha sobre un secundario?
118. ¿Cómo clasifica T17 a Neo4j HA y cómo clasifica T19 a Neo4j? ¿Qué arquitectura describe cada frase?
119. En el modelo de T18, ¿qué conoce el motor del valor y cuáles son las tres operaciones básicas?
120. Compará RDB y AOF: qué puede perderse en una caída y cuál usa Redis al reiniciar si ambos están activos.
121. ¿Qué hacen `MULTI`, `EXEC`, `DISCARD` y `WATCH`? ¿En qué se distingue `DISCARD` de un `ROLLBACK`?
122. ¿Qué vigila Sentinel y por qué la clase pide al menos tres instancias?
123. ¿Qué estructura de Redis guarda miembros únicos con un score, admite scores repetidos y permite consultar un rango de puntajes, como en el TP 12?
124. ¿Dónde clasifican a Redis T19 y la guía visual de T20?
125. ¿En qué separa la clase al paper Dynamo del servicio DynamoDB, y qué tope de tamaño tiene un ítem?
126. Compará clave solo hash con clave hash más rango, y un LSI con un GSI. ¿Cuál de los índices no se agrega después de crear la tabla?
127. ¿Qué miden un RCU y un WCU en el material, y por qué una lectura fuertemente consistente cuesta más?
128. En el TPO, ¿qué hacen con el stock y con los ingresos los estados Completada, Pendiente y Anulada, y en qué orden se consumen los lotes?

Fuentes para corregir el bloque: [T17, pp. 3–4, 9, 13, 17–20, 28, 36–40; C06, pp. 2–13; T18, pp. 2–3, 9, 12–18, 24–27; T19, pp. 7, 13–16, 36; T20, pp. 4–6, 10, 17–18, 23, 31; P16, p. 2; P18, pp. 4–5]. Las clasificaciones CAP que se contradicen están en [dudas](../dudas-y-conflictos.md).
