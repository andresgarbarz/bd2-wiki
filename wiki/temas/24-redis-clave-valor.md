# Redis y bases de clave-valor

## El modelo de clave-valor

Una base de clave-valor, en esta unidad, es una tabla hash: el acceso es por clave primaria. Las operaciones básicas que presenta la clase son **PUT**, **GET** y **DEL**. El valor se guarda como un BLOB opaco; el motor no interpreta su interior y la aplicación define ese formato. Ese acceso por clave es el argumento de rendimiento y de escalabilidad de la clase. [T18, pp. 2–3]

Casos que la clase considera adecuados: la sesión web entera en un solo PUT/GET, el perfil de un usuario recuperado por `userId`, y el carrito asociado a un usuario. Casos que desaconseja: relaciones entre datos, transacciones de varias operaciones, consultar por el interior del valor y operaciones que cruzan conjuntos de datos distintos. [T18, pp. 6–7]

La lámina de productos nombra Redis, Aerospike, Oracle NoSQL, Amazon DynamoDB, Memcached y Riak. El [ranking](../../material/figuras/T18-p5-ranking-clave-valor.png) es una captura DB-Engines de agosto/septiembre de 2020 y 2021: Redis figura primero y Amazon DynamoDB segundo entre las clave-valor de esa tabla. No es un ranking actual. T19 trae otra foto: release estable 7.0.11 del 17 de abril de 2023, puesto 6 global y 1 entre clave-valor. [T18, pp. 4–5; T19, pp. 1–3]

## Redis

Redis es un almacén de estructuras en memoria, con persistencia opcional en disco. Sirve como base, caché o broker de mensajes. Las estructuras que enumera T18 son cadenas, hashes, listas, conjuntos, conjuntos ordenados, bitmaps, HyperLogLog e índices geoespaciales; el hands-on agrega streams. El puerto de los ejemplos es 6379. Un benchmark citado en la clase habla de más de 100.000 `SET` por segundo. [T18, p. 9; T19, pp. 1, 4]

La [figura de familias](../../material/figuras/T19-p2-familias.png) ubica Redis como almacén clave-valor, al lado de PostgreSQL, MongoDB, CouchDB, HBase, Amazon DynamoDB y Neo4j. [T19, p. 2]

Los prefijos de comando del hands-on: `S` para conjuntos, `H` para hashes, `Z` para conjuntos ordenados, `L` o `R` según el extremo de una lista. Una colección puede guardar hasta 2^32 elementos por clave. [T19, p. 8]

| Estructura | Operaciones que el material muestra | Uso que ilustra |
|---|---|---|
| Cadena | `SET`, `GET`, `MSET`, `MGET`, `INCR`, `SETNX`, `EXPIRE`, `TTL` | URL corta, contador, clave con vencimiento. |
| Hash | `HMSET`, `HSET`, `HGET`, `HGETALL`, `HKEYS`, `HVALS` | Varios campos bajo una sola clave, en lugar de una clave por campo. |
| Lista | `LPUSH`, `RPUSH`, `LRANGE`, `LPOP`, `RPOP`, `LREM`, `RPOPLPUSH` | Cola (entrar por un extremo y salir por el otro) o pila (mismo extremo). |
| Lista bloqueante | `BRPOP` con timeout | Esperar un elemento hasta que otro cliente haga `LPUSH`. |
| Conjunto | `SADD`, `SMEMBERS`, `SINTER`, `SUNION`, `SDIFF`, y las variantes `*STORE`, `SMOVE`, `SCARD`, `SPOP`, `SREM` | Pertenencia y álgebra de conjuntos, sin orden. |
| Conjunto ordenado | `ZADD`, `ZRANGE`, `ZREVRANGE`, `ZCOUNT`, `ZRANGEBYSCORE`, `ZREM` | Ranking: el miembro es único y el score puede repetirse. |

[T18, p. 11; T19, pp. 5–16; P16A]

`KEYS *` lista las claves existentes. El hands-on corrige al libro: para ver los campos de un hash corresponde `HKEYS`; `HVALS` devuelve los valores. [T19, pp. 8–9]

El ejercicio resuelto en P16A practica esas estructuras con películas, cantantes, libros y productos de bases de datos. No resuelve la tabla de puntajes IMDB del TP 12 ni sus preguntas de transacciones, caché, Bloom filter, Sentinel o CAP. [P16A; P16, p. 2]

## Transacciones

`MULTI` encola comandos y responde `OK`. `EXEC` dispara la cola. `DISCARD` la vacía y termina la transacción. `WATCH` observa claves: si alguna cambia antes del `EXEC`, la transacción se aborta. La clase lo presenta como bloqueo optimista desde Redis 2.2. [T18, pp. 12–14]

Hay dos matices dentro del propio material. T18 dice que la transacción es atómica, todos los comandos o ninguno, y en la misma lámina admite que, si el servidor cae, puede persistir solo una parte de las operaciones. También dice que no hay rollback ante un error de sintaxis descubierto tarde o un tipo de dato incorrecto. T19 describe `DISCARD` como cancelar la transacción todavía no ejecutada, y lo distingue de un `ROLLBACK` que revierte un estado ya aplicado. [T18, pp. 12–13; T19, p. 7]

## Durabilidad, réplicas y Sentinel

Se puede apagar la persistencia y dejar los datos solo en el proceso. **RDB** guarda snapshots del conjunto en intervalos; el proceso de grabación es un hijo, así que el servidor no hace esa E/S en el hilo principal. Sirve para copias y se reinicia más rápido que AOF en volúmenes grandes; ante una caída se pierden los minutos desde el último snapshot. **AOF** registra cada escritura y reconstruye el estado al arrancar. La política de `fsync` puede ser ninguna, cada segundo o en cada consulta; con la política por segundo, la clase dice que se perdería como máximo un segundo. El AOF suele ocupar más que el RDB. Si ambos están activos, al reiniciar se usa el AOF por ser el registro más completo. La recomendación de la clase es AOF más un snapshot RDB periódico. [T18, pp. 17–24]

La replicación que describe T18 es maestro-esclavo: la réplica intenta ser una copia del maestro y, tras una desconexión, pide una resincronización parcial o, si no alcanza, una completa mediante un snapshot. [T18, pp. 15–16]

**Sentinel** vigila maestros y réplicas y promociona una réplica si el maestro cae. La clase separa tres procesos —servidor Redis, Sentinel y cliente— y dice que pueden convivir en una máquina, aunque no conviene. El quórum es cuántos Sentinel deben acordar que un nodo cayó. Pide al menos 3 Sentinel para un quórum de 2, y una sola instancia maestra. [T18, pp. 25–27]

El hands-on muestra además un `Redis::Distributed` de Ruby contra dos puertos y aclara que el módulo de cluster que comenta el libro seguía, en ese texto, en desarrollo para superar la RAM de una sola máquina. La debilidad que copia del libro es esa: los datos viven en memoria y un cierre antes del snapshot puede perder escrituras; el tamaño práctico es la RAM disponible. [T19, pp. 30–35]

## Seguridad, Lua y Bloom filters

Redis está pensado para clientes de confianza en una red de confianza. No hay que exponer el puerto a Internet. `AUTH` rechaza clientes no autenticados cuando la autorización está activa, y la clase aclara que no protege el transporte. La aplicación debe mediar el acceso no confiable con ACL y validación. [T18, pp. 28–30]

Un script Lua se ejecuta de forma atómica: mientras corre, ningún otro cliente ejecuta comandos. [T18, p. 31]

Un Bloom filter responde si un elemento puede pertenecer a un conjunto, con falsos positivos posibles. T19 dice que no venía por defecto y que a partir de la versión 4 puede agregarse como módulo (RedisBloom / ReBloom). Antes de eso, el material lo construye con `SETBIT` y `GETBIT` o con una gema de Ruby. El TP 12 pide justamente esa prueba con bits, después de leer la teoría de las páginas 285–286 del libro. [T19, pp. 32–33; P16, p. 2; C04]

## CAP en esta unidad

T19 clasifica Redis y Neo4j como **CA**: no hacen sharding, así que la partición no sería el problema, y la propia lámina duda de aplicar CAP a un sistema no distribuido. La [guía visual](../../material/figuras/T20-p31-guia-visual-cap.png) de T20 ubica Redis del lado **CP**, junto con MongoDB, y no del lado CA. No unificar las dos etiquetas; el detalle está en [dudas y conflictos](../dudas-y-conflictos.md). [T19, p. 36; T20, p. 31]

## TP 12

La guía usa `docker pull redis` y el contenedor `Myredis` en el puerto 6379, sin fijar la etiqueta de la imagen. Pide completar la tabla de comandos por tipo, probar `MULTI`/`DISCARD` contra el `ROLLBACK` de PostgreSQL, la expiración como caché, un Bloom filter con bits, Sentinel, y cargar las películas de la [tabla IMDB](../../material/figuras/P16-p2-peliculas-imdb.png) en la estructura que permita ordenar y filtrar por score. El cierre pide fortalezas, debilidades y la clasificación CAP. [P16, pp. 1–2]

Los puntajes de esa tabla, leídos en la página renderizada, van de *The theory of everything* (7.7) a *The Shawshank Redemption* (9.3). La autoridad de los nombres y los números es la figura. [P16, p. 2]

## Continuar

- [Neo4j](23-neo4j-grafos-y-cypher.md): la otra mitad de la frase CA de T19, y el anexo Raft.
- [DynamoDB](25-dynamodb.md): la otra clave-valor del ranking, con modelo de ítem y claves de partición.
- [Guía TP 12](../../practica/tp-12/README.md).
- [Bloom filters de Cassandra](21-cassandra-almacenamiento-y-bloom.md): el mismo concepto, otro uso.
