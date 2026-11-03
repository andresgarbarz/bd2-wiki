# Material y trazabilidad

`catedra/` contiene los originales oficiales y no se edita. `catedra/bibliografia/` guarda bibliografía provista por la cátedra con licencia institucional: tiene estado `local` en el catálogo, se ignora en git y no se publica. Sus hashes están en `checksums.sha256` para detectar alteraciones accidentales. `externo/` guarda fuentes complementarias claramente separadas. `extraido/` contiene una representación textual, separada por páginas, para que los agentes puedan buscar y citar rápido. Ante una diferencia, manda el original.

El [catálogo](catalogo.tsv) asigna un ID estable a cada fuente. Las citas usan `[T02, p. 7]`, `[P03, p. 2]` o `[SQL01]`. El ciclo indica la cursada en la que fue entregado el archivo, no necesariamente el año de creación interna de todas sus diapositivas.

## Incorporar material

1. Dejar los archivos nuevos en `material/entrada/` sin renombrarlos.
2. Desde la raíz, pedir `incorporá el material nuevo` o invocar `$incorporar-material`.
3. El agente clasifica, asigna ID, preserva el original, extrae texto, actualiza la wiki y valida referencias.

No copiar manualmente una fuente nueva a `extraido/`: ese contenido se regenera con `./scripts/extraer-fuentes.sh` a partir del catálogo.

## Incorporaciones y continuidad

- [2026-10-02 — NoSQL y MongoDB](incorporaciones/2026-10-02-nosql-mongodb.md): 11 fuentes incorporadas; registro de la primera unidad y su continuidad.
- [2026-10-02 — Cassandra](incorporaciones/2026-10-02-cassandra.md): cuatro fuentes oficiales incorporadas (85 páginas PDF y una imagen); se completó lo pendiente en entrada.
- [2026-10-13 — TP 10, parte II](incorporaciones/2026-10-13-cassandra-tp10-parte-2.md): P13 incorporada; dos reenvíos duplicados de C04 y C05 descartados.
- [2026-10-13 — Seven Databases in Seven Weeks](incorporaciones/2026-10-13-seven-databases.md): bibliografía B01, uso local y acotado al TP 9.
- [2026-10-13 — Ejercicios tipo parcial](incorporaciones/2026-10-13-ejercicios-parcial.md): P14, siete páginas y cuatro figuras.
- [2026-11-03 — Neo4j, Redis, DynamoDB y TPO](incorporaciones/2026-11-03-neo4j-redis-dynamodb-tpo.md): diez fuentes; la clase 19 y los CSV del TPO no vinieron en el lote.

Los ID no tienen que coincidir con el número de clase o de TP. Se asignan sin reutilizar los existentes; las series con sufijos agrupan partes y recursos relacionados.
