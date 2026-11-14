# Avances del TPO

Fecha: 7 de noviembre de 2026.

Última revisión: 14 de noviembre de 2026. Se integraron en `main` los tres PR del [repositorio del grupo](https://github.com/andresgarbarz/TPO_BD2_FarmaRed): [#1, seis lecturas MongoDB](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/pull/1), [#2, grafo y Q02/Q03/Q10](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/pull/2) y [#3, Q04/Q06/Q12](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/pull/3), preservando sus commits. La revisión se contrastó con la consigna completa [P18, pp. 1–7] y las decisiones de esta wiki.

Las quince consultas están implementadas. Q02, Q03 y Q10 usan Neo4j; las demás usan MongoDB. Se resolvió el conflicto en `scripts/cargar.py` conservando tanto `crear_vista()` de Q11 como `proyectar_todo()` del grafo. Se corrigieron los estados pendientes del README y se documentaron Q04/Q06/Q12. El material para el informe está en [lecturas MongoDB](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/blob/main/docs/informe-consultas-lectura.md), [grafo](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/blob/main/docs/informe-grafo.md), [Q04/Q06/Q12](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/blob/main/docs/informe-consultas-restantes.md) y [escrituras](decisiones-escrituras.md).

**Verificación del 14 de noviembre:** 25 pruebas correctas (22 MongoDB, tres de grafo), verificador de los nueve CSV y dos ejecuciones del cargador real. MongoDB `8.3.11` y Neo4j `5.26.32` en contenedores exclusivos `farmared-pr-review-mongo` y `farmared-pr-review-neo4j`, puertos `127.0.0.1:27019` y `127.0.0.1:7689`; Python `3.11`, drivers fijados por el proyecto. Las pruebas usan bases MongoDB temporales y el grafo exclusivo de revisión. No se recargaron bases preexistentes. Se comprobó idempotencia (58 nodos, 96 relaciones), Q02/Q03/Q10, sucursal histórica tras reasignar un farmacéutico inactivo, precios de ítems repetidos, referencias inválidas, proyección de Q13/Q14/Q15, FEFO y rechazo sin cambios por falta de stock. Las pruebas MongoDB cubren resultados, estados, nulos y límites temporales; la prueba MongoDB de Q15 ya no intenta escribir en Neo4j.

**Alcance:** las pruebas de grafo se habilitan con `FARMARED_TEST_NEO4J_URI`, que debe apuntar a una instancia descartable: la reconstrucción vacía su contenido. No se hizo una prueba de carga concurrente ni de caída del proceso. Se conservan las limitaciones de compensación de [decisiones-arquitectura.md](decisiones-arquitectura.md). Q12 adopta 365 días incluyendo hoy, convención del proyecto registrada en [dudas y conflictos](../wiki/dudas-y-conflictos.md#tpo-farmared).

La implementación y verificación del 10 de noviembre de Q01/Q05/Q07/Q08/Q09/Q11 se conservan a continuación como historial, incluidos los commits por consulta.

Los datasets provistos están ahora en `datasets/`, en la raíz de esta wiki. Los nueve archivos se compararon byte a byte con `entregas/tpo-farmared/datasets/` y son idénticos. El cargador sigue leyendo la copia interna del proyecto. En los apartados siguientes, `datasets/extra/` es relativo al repositorio del grupo.

El código está en el repositorio del grupo, [TPO_BD2_FarmaRed](https://github.com/andresgarbarz/TPO_BD2_FarmaRed). La carpeta `entregas/tpo-farmared/` de esta wiki no se versiona. La justificación para el informe está en [decisiones-escrituras.md](decisiones-escrituras.md). Los motores, en [decisiones-arquitectura.md](decisiones-arquitectura.md).

## Commits de las seis lecturas

Se verificó cada versión exacta antes de crear su commit, contra MongoDB real. La suite crece con cada consulta: 2, 4, 6, 9, 11 y 16 pruebas correctas, respectivamente. No se reescribió el historial anterior.

| Commit | Alcance |
|---|---|
| [d18ec93](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/commit/d18ec93f1641b3baee1dbf9142d669b39b15aae4) | Q09 y sus pruebas; base compartida de integración |
| [510f613](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/commit/510f6135f59a051b8db34851ea6897be342bdc6e) | Q01 y sus pruebas |
| [55672a5](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/commit/55672a5b8511153bb9b69e6b20913bed1f5275a9) | Q08 y sus pruebas |
| [aa852fb](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/commit/aa852fb42787b4bf4afd4e3e38943478d01dc266) | Q07 y sus pruebas |
| [1af1131](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/commit/1af11315a76900e289b7df5f9a301223c8c88ad4) | Q05 y sus pruebas |
| [278db95](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/commit/278db95dad211e0fc085a03dd6cee859ce7c9dea) | Q11, preparación de su vista en la carga y sus pruebas |
| [7a4fad5](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/commit/7a4fad5777ca2e77a12a866bd91e0b8351bec1c3) | Instrucciones, ejemplos para el informe y textos del listado de consultas |
| [11c9d1b](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/commit/11c9d1b622b2c15a4f2ccc070cf1862ff02d129f) | Fix de Q05: 60 fechas incluyendo hoy; regresión con una venta por día durante 61 fechas; 17 pruebas correctas |

La revisión del enunciado detectó que el rango original de Q05 incluía 61 fechas. Se corrigió a `hoy() - 59 días` inclusive hasta mañana exclusivo. El 2026-11-10 el corte es 2026-09-12; los conteos del dataset provisto no cambian. La corrección, su prueba y la documentación están en el mismo commit.

## Qué quedó hecho

- `compose.yaml` con MongoDB `8.3.11` y Neo4j `5.26.32`, sin autenticación y con los puertos solo en `127.0.0.1`. El devcontainer levanta los dos al crearse.
- `src/db.py`: `MONGO_URI`, `NEO4J_URI` y `hoy()`, la fecha de Argentina a medianoche.
- `scripts/cargar.py`: borra las ocho colecciones, las vuelve a llenar, valida extras, crea o actualiza la vista mensual de Q11 y al final llama a `proyectar_todo()`.
- `src/consultas/q01.py`, `q05.py`, `q07.py`, `q08.py`, `q09.py` y `q11.py`; pruebas con datos y bordes en `tests/test_consultas.py`.
- `src/consultas/q13.py`, `q14.py` y `q15.py`, con la validación compartida en `src/reglas.py`.
- `src/grafo.py`: proyección completa e incremental, restricciones de unicidad y consultas Cypher. Q14/Q15 proyectan venta y receta en una transacción de Neo4j.

Q02/Q03/Q10 y Q04/Q06/Q12 también están implementadas y verificadas. No quedan módulos en `NotImplementedError`.

## Cómo levantarlo

En Codespaces el devcontainer instala dependencias y ejecuta `docker compose up -d`. En una máquina propia, con el puerto 27017 libre:

```bash
docker compose up -d
python3 -m pip install -r requirements.txt
python3 scripts/verificar_datasets.py
python3 scripts/cargar.py
```

Si el Python del sistema rechaza `pip`, lo mismo va dentro de `python3 -m venv .venv`. Las consultas se ejecutan con el directorio actual en `src/`:

```bash
python3 -m consultas
python3 -c "from consultas.q01 import ejecutar; print(ejecutar())"
python3 -c "from consultas.q13 import ejecutar; print(ejecutar('baja', id_cliente='C009'))"
python3 -c "from consultas.q14 import ejecutar; print(ejecutar('C006', 'F001', [{'id_producto': 'PRD003', 'cantidad': 50}], 'Obra social', 'RC003'))"
python3 -c "from consultas.q15 import ejecutar; print(ejecutar('V011'))"
```

Fuera del Codespace, si `MONGO_URI` y `NEO4J_URI` no están definidas, el código usa `127.0.0.1`.

Las otras lecturas MongoDB se invocan igual, cambiando `q01` por `q04`, `q05`, `q06`, `q07`, `q08`, `q09`, `q11` o `q12`. Q02 no recibe argumentos; Q03 recibe `id_cliente` y Q10, `id_sucursal`. Las pruebas se corren desde la raíz del repositorio del grupo con `python3 -m unittest discover -s tests -v`. Para repetir la revisión completa, levantar instancias descartables, definir sus URIs y habilitar el grafo con `FARMARED_TEST_NEO4J_URI`; ver el README del proyecto.

## Contrato de datos

`_id` es el id de la cátedra (`C001`, `V001`). Los demás campos conservan el nombre del CSV. Booleanos reales, importes enteros, fechas `Date` armadas con año, mes y día. La base se llama `farmared`. Hay un índice único `dni_unico` sobre `clientes.dni`.

| Colección | Origen |
|---|---|
| `clientes` | `clientes.csv` |
| `sucursales` | `sucursales.csv` |
| `farmaceuticos` | `farmaceuticos.csv` |
| `proveedores` | `proveedores.csv` |
| `productos` | `productos.csv` |
| `stock_lotes` | `stock_lotes.csv` |
| `recetas` | `recetas.csv` |
| `ventas` | `ventas.csv` más sus ítems |

No hay colección de detalle. Cada venta tiene `items`, con `id_detalle`, `id_producto`, `cantidad` y `precio_unit`. El proveedor del producto queda referenciado con `id_proveedor`.

Tres campos no vienen en los CSV:

- `id_sucursal`: se fija en la venta al cargar, registrar o completar; conserva la sucursal histórica.

- `items.lotes`: lo escriben la 14 y la 15. Es `[{id_lote, unidades}]`.
- `fecha_registro`: lo escribe solo la 15. En una venta Completada, `fecha` es el día en que computa ingresos. Si la completó la 15, `fecha` es el día de la entrega y la reserva queda en `fecha_registro`.

Las consultas 3, 5, 11 y 12 usan `fecha`. La 11 suma el mes de esa fecha: una reserva de octubre completada en noviembre entra en noviembre. Los ejemplos de las consultas 1 a 12 se sacan con una carga recién hecha, porque la 13, la 14 y la 15 cambian esos datos.

## Proyección del grafo

La carga llama a `proyectar_todo()` al final. Q13 llama a `upsert_cliente`; Q14/Q15 llaman a `upsert_venta_y_receta`. Las relaciones se reemplazan dentro de la transacción de Neo4j para evitar duplicados o vínculos obsoletos. Si falla la proyección, Mongo ya quedó confirmado y la respuesta incluye `aviso_grafo`; `reconstruir()` rehace el grafo desde Mongo, sin escrituras concurrentes durante esa lectura.

Una escritura exitosa devuelve `{ok: true, documento, aviso_grafo?}`. Un rechazo devuelve `{ok: false, error}`.

## Carga y registros extra

`datasets/extra/` usa el mismo nombre y los mismos encabezados que el archivo de la cátedra. Los id siguen la serie (`C009`, `V017`, `L019`). La carga imprime, por colección, cuántos documentos son de la cátedra y cuántos propios. Falla si un id propio repite uno de la cátedra, si una referencia no existe o si el total de una venta no coincide con sus ítems.

Recargar borra lo creado con la 13 y la 14. Una venta que la 15 pasó a Completada vuelve a Pendiente, con la fecha y los ítems del archivo.

Para un caso FEFO que no dependa de la fecha, hace falta un producto con dos lotes de vencimiento lejano en la misma farmacia, y dos lotes que venzan el mismo día. El orden de descuento es `{vencimiento: 1, _id: 1}`.

## Qué sigue abierto

- Al menos diez registros propios por colección, en `datasets/extra/`.
- El diagrama de colecciones. El del grafo está en `docs/informe-grafo.md`.
- Integrar el informe PDF final con los documentos de `docs/`, sus consultas, ejemplos, diagramas y limitaciones. [P18, p. 7]
