# Incorporación 2026-11-03 — Neo4j, Redis, DynamoDB y TPO

## Alcance y procedencia

`material/entrada/` tenía diez archivos nuevos. Ningún SHA-256 coincidía con el catálogo. El usuario indicó que el TPO es el trabajo de entrega, que los TP numerados son guías y que el resto es teoría o resoluciones.

| ID | Fuente | Páginas | Nombre recibido |
|---|---|---|---|
| T17 | Clase 16 — Neo4j | 41 | `BD2_Clase 16 - Neo4J.pdf` |
| T18 | Clase 17 — clave-valor, generalidades | 31 | `BD2_Clase 17 - Bases de Clave-Valor (generalidades).pdf` |
| T19 | Clase 18 — Redis, hands-on | 36 | `BD2_Clase 18 - Bases de Clave-Valor (Hands-on).pdf` |
| T20 | Clase 20 — Amazon DynamoDB | 34 | `BD2_Clase 20 - Amazon DynamoDB.pdf` |
| C06 | Anexo Neo4j: MERGE, Causal Clustering y Raft | 13 | `Neo4j_anexo.pdf` |
| P15 | TP 11 — Neo4j | 2 | `ITBA TP 11 - Neo4j(1).pdf` |
| P16 | TP 12 — Redis | 2 | `ITBA TP 12 - Redis.pdf` |
| P16A | Solución de un ejercicio de Redis | texto | `Solución ejercicio REDIS.txt` |
| P17 | TP 13 — DynamoDB | 2 | `ITBA TP 13 - DynamoDB.pdf` |
| P18 | TPO FarmaRed | 7 | `TPO_FarmaRed.pdf` |

Los ID siguen el primer número libre. T17 no es la clase 17: la clase 17 es T18. No vino una clase 19. P16A no resuelve el TP 12: practica otras estructuras y otros datos.

T18 y T19 tienen fecha de creación del PDF el 2023-10-27. P18 fue creado el 2026-11-01. El ciclo del catálogo queda 2026, igual que el resto de la cursada.

## Lectura y productos

- Texto por página con `./scripts/extraer-fuentes.sh`.
- Ocho figuras de páginas que el texto no recupera: patrón Cypher, red de amigos, `shortestPath`, grafo de propiedades, ranking clave-valor, familias, guía visual CAP y tabla IMDB del TP 12.
- Apuntes nuevos: [Neo4j](../../wiki/temas/23-neo4j-grafos-y-cypher.md), [Redis](../../wiki/temas/24-redis-clave-valor.md) y [DynamoDB](../../wiki/temas/25-dynamodb.md).
- Espacios [TP 11](../../practica/tp-11/README.md), [TP 12](../../practica/tp-12/README.md) y [TP 13](../../practica/tp-13/README.md), sin resolución.
- Consigna del TPO en [entregas/tpo-farmared](../../entregas/tpo-farmared/README.md). No hay diseño ni código.
- Preguntas de repaso 113–128. La nota de P14 que decía que no había material de Cypher quedó reemplazada por la lectura con T17.

## Conflictos y vacíos

En [dudas y conflictos](../../wiki/dudas-y-conflictos.md):

- T17 clasifica Neo4j HA como AP; T19 clasifica Neo4j y Redis como CA; la lámina de T20 ubica Redis en CP y al paper Dynamo en AP, sin rotular DynamoDB.
- HA master-slave (T17, Enterprise 3.4.9) y Causal Clustering con Raft (C06) no se tratan como la misma arquitectura.
- T18 afirma atomicidad de la transacción y, a la vez, un registro parcial si el servidor cae.
- El TPO dice grupos de 4 y, más adelante, “tres integrantes”.
- No están los nueve CSV que la consigna dice entregar. Tampoco llegó la clase 19.
- Las imágenes Docker de P15, P16 y P17 no traen etiqueta de versión y no se ejecutaron.

Los rankings de T17, T18 y T19 son fotos de 2020–2023. No se usaron como información actual.
