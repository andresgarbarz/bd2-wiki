# TP 11 — Neo4j

- Fuente: [P15](../../material/catedra/practica/tp-11-neo4j.pdf), dos páginas, ITBA 2026.
- Apuntes: [grafos y Cypher](../../wiki/temas/23-neo4j-grafos-y-cypher.md), [anexo MERGE y Raft](../../material/catedra/teoria/c06-neo4j-merge-causal-clustering.pdf), [CAP](../../wiki/temas/15-nosql-y-cap.md).
- Libro que cita la guía: [B01](../../material/catedra/bibliografia/b01-seven-databases-in-seven-weeks-2e.pdf), capítulo 6. La propia guía advierte que algunos métodos quedaron deprecados. La copia local de B01 no está versionada.
- Estado: consigna incorporada, sin resolución ni ejecución.

## Preparación indicada por la guía

P15 propone `docker pull neo4j` y este contenedor, sin etiqueta de versión:

```bash
docker run --name Myneo4j -p 7474:7474 -p 7687:7687 --env=NEO4J_AUTH=none -d neo4j
```

La interfaz es `http://localhost:7474/browser`, con el campo de contraseña vacío. `docker exec -it Myneo4j bash` entra al contenedor. También menciona una instalación local y el manual de operaciones. El laboratorio de este repositorio todavía no incluye el servicio. [P15, p. 1]

## Ejercicios

| Ejercicio | Alcance | Página |
|---|---|---|
| 1 | Capítulo 6, pp. 181–188: nodos de vinos, el grafo de la p. 185, personas y gustos de la p. 186, consultas de las pp. 185–187, índice y unicidad de las pp. 187–188. | 1 |
| 2 | `:play movies` hasta el paso 7, después de borrar el grafo anterior. Antes del paso 8: cantidad de nodos, etiquetas y cantidades, tipos de relación. Luego el ejercicio Do.1. | 1 |
| 3 | Dos localidades con longitud y latitud, relación `hay_ruta` y la distancia guardada en esa relación. | 2 |
| 4 | Sandbox *Paradise Papers by ICIJ*: esquema, cantidades, etiquetas, tipos de relación, vecindades de Hutomo Mandala y de Saadi Yacef. | 2 |
| 5 | Fortalezas, debilidades y cuándo usar Neo4j. | 2 |
| 6 | Clasificación según CAP. | 2 |

[P15, pp. 1–2]

La guía escribe “Sanbox Login” al describir el acceso al sandbox. El servicio y el tutorial `:play` dependen del sitio actual de Neo4j; no se comprobó que sigan disponibles con esos nombres.

Para el ejercicio 6, T17 clasifica Neo4j HA como AP y T19 clasifica Neo4j como CA. Hay que explicitar cuál arquitectura se está clasificando. Ver [dudas](../../wiki/dudas-y-conflictos.md). [T17, p. 40; T19, p. 36]
