# TP 12 — Redis

- Fuente: [P16](../../material/catedra/practica/tp-12-redis.pdf), dos páginas, ITBA 2026.
- Apuntes: [clave-valor y Redis](../../wiki/temas/24-redis-clave-valor.md), [CAP](../../wiki/temas/15-nosql-y-cap.md).
- Libro que cita la guía: [B01](../../material/catedra/bibliografia/b01-seven-databases-in-seven-weeks-2e.pdf), capítulo 8. La copia local de B01 no está versionada.
- Ejercicio resuelto aparte: [P16A](../../material/catedra/practica/recursos/p16a-solucion-ejercicio-redis.txt). Practica cadenas, listas, hashes, conjuntos y conjuntos ordenados. No responde los ejercicios 2 a 8 de esta guía ni carga la tabla IMDB.
- Estado: consigna incorporada, sin resolución ni ejecución de P16.

## Preparación indicada por la guía

P16 propone `docker pull redis` y `docker run --name Myredis -p 6379:6379 -d redis`, sin etiqueta de versión. Dentro del contenedor, `redis-cli`. El laboratorio de este repositorio todavía no incluye el servicio. [P16, p. 1]

## Ejercicios

| Ejercicio | Alcance | Página |
|---|---|---|
| 1 | Completar la tabla de comandos de lectura, inserción, modificación y borrado para cadenas, hashes, listas, listas bloqueantes, conjuntos y conjuntos ordenados. La guía ya anota `GET`/`MGET` y `SET`/`MSET`, y marca “no aplica” en una celda de conjuntos. | 1 |
| 2 | Transacciones de las pp. 261–262 del libro. Diferencia entre `ROLLBACK` de PostgreSQL y `DISCARD` de Redis. | 1 |
| 3 | Qué función permite usar Redis como caché, pp. 271–272, y probarla. | 2 |
| 4 | Bloom filter con `SETBIT` y `GETBIT`, pp. 285–288. | 2 |
| 5 | Ejercicio Find.2, p. 289, con la documentación de Redis Sentinel. | 2 |
| 6 | Cargar las películas y su score IMDB en la estructura que responda los cinco incisos. | 2 |
| 7 | Fortalezas, debilidades y cuándo usarlo. | 2 |
| 8 | Clasificación según CAP. | 2 |

[P16, pp. 1–2]

## Películas del ejercicio 6

`pdftotext` no recupera la tabla. Los valores salen de la [página renderizada](../../material/figuras/P16-p2-peliculas-imdb.png):

| Película | Score |
|---|---|
| The theory of everything | 7.7 |
| The Imitation Game | 8.0 |
| Amelie | 8.3 |
| The Shawshank Redemption | 9.3 |
| The Godfather | 9.2 |
| Fight Club | 8.8 |
| Matrix | 8.7 |
| Titanic | 7.8 |
| Jurassic Park | 8.1 |
| ET | 7.9 |

[P16, p. 2]

T19 muestra conjuntos ordenados (`ZADD`, `ZRANGE`, `ZRANGEBYSCORE`, `ZREVRANGE`) como la estructura asociada a un score. Elegirla al resolver es una lectura de esa clase, no una instrucción explícita de la guía. [T19, pp. 13–16]

Para el ejercicio 8, T19 clasifica Redis como CA y la lámina de T20 lo ubica en CP. Ver [dudas](../../wiki/dudas-y-conflictos.md). [T19, p. 36; T20, p. 31]

## Qué cubre P16A

Cinco bloques: cinco títulos con `SET`/`MSET`/`SETNX`; una lista de cantantes; cinco hashes de libros; conjuntos `BD_NoSQL` y `BD_Relacionales`; un conjunto ordenado `productosNoSQL` con scores 100, 75, 50 y 40. El comentario que sigue a `EXISTS pelicula3` dice “Si no existe se borrará”, después de un `DEL`: el comentario no describe el comando que tiene debajo. El original se conservó sin corregir. [P16A]
