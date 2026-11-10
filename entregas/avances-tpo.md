# Avances del TPO

Fecha: 7 de noviembre de 2026.

Última revisión: 10 de noviembre de 2026. Base del trabajo: `main` del repositorio del grupo, commit `383168c` (`feat: agregar el entorno, la carga y las consultas 13 a 15`). Las seis lecturas están publicadas en [feat/consultas-lectura-mongodb](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/tree/feat/consultas-lectura-mongodb), con seis commits por consulta y sus pruebas, más un commit de documentación y el fix de Q05. Último commit: `11c9d1b`. La documentación exterior se trabaja en `docs/consultas-lectura-mongodb`.

Esta parte cubre **Q01, Q05, Q07, Q08, Q09 y Q11**, todas implementadas sobre MongoDB y verificadas con el dataset completo. El plan, las decisiones y los controles están en [consultas-asignadas.md](consultas-asignadas.md); el material para el informe está en [informe-consultas-lectura.md](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/blob/11c9d1b622b2c15a4f2ccc070cf1862ff02d129f/docs/informe-consultas-lectura.md).

Se ejecutó el plan: Q09 con `find`; Q01 y Q08 con referencias resueltas por `$lookup`; Q07 con suma de unidades; Q05 conservando farmacéuticos con cero; Q11 mediante la vista `ingresos_mensuales_farmaceutico` y selección del mes actual con `hoy()`. Todas exponen `ejecutar() -> list[dict]`, sin argumentos.

Verificación: MongoDB `8.3.11` real en el contenedor aislado `farmared-consultas-test`, puerto `127.0.0.1:27018`; Python `3.11`, PyMongo `4.15.3`. Pasaron el verificador de los nueve CSV y las 17 pruebas de integración en `tests/test_consultas.py`. Se ejecutaron también el cargador real y las seis consultas con la fecha del sistema (2026-11-10). No se tocó una base preexistente: las pruebas usan bases temporales propias y la ejecución manual usa `farmared` dentro del contenedor nuevo. Las ejecuciones del 7 de noviembre conservan su carácter histórico.

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
- `scripts/cargar.py`: borra las ocho colecciones, las vuelve a llenar, valida extras, crea o actualiza la vista mensual de Q11 y al final llama a `reconstruir()`.
- `src/consultas/q01.py`, `q05.py`, `q07.py`, `q08.py`, `q09.py` y `q11.py`; pruebas con datos y bordes en `tests/test_consultas.py`.
- `src/consultas/q13.py`, `q14.py` y `q15.py`, con la validación compartida en `src/reglas.py`.
- `src/grafo.py`: `reconstruir()`, `upsert_cliente`, `upsert_venta` y `upsert_receta`. Están vacías. Reciben el documento tal como quedó en Mongo.

Las consultas 2, 3, 4, 6, 10 y 12 siguen en `NotImplementedError`.

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

Las otras cinco lecturas se invocan igual, cambiando `q01` por `q05`, `q07`, `q08`, `q09` o `q11`. Las pruebas se corren desde la raíz del repositorio del grupo con `python3 -m unittest discover -s tests -v`. Para la instancia aislada de esta revisión, definir `MONGO_URI=mongodb://127.0.0.1:27018` antes de cargar, consultar o probar.

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

Dos campos no vienen en los CSV:

- `items.lotes`: lo escriben la 14 y la 15. Es `[{id_lote, unidades}]`.
- `fecha_registro`: lo escribe solo la 15. En una venta Completada, `fecha` es el día en que computa ingresos. Si la completó la 15, `fecha` es el día de la entrega y la reserva queda en `fecha_registro`.

Las consultas 3, 5, 11 y 12 usan `fecha`. La 11 suma el mes de esa fecha: una reserva de octubre completada en noviembre entra en noviembre. Los ejemplos de las consultas 1 a 12 se sacan con una carga recién hecha, porque la 13, la 14 y la 15 cambian esos datos.

## Qué espera el grafo

La carga llama a `reconstruir()` al final. La 13 llama a `upsert_cliente`. La 14 y la 15 llaman a `upsert_venta` y, si hubo receta, a `upsert_receta`. Si la proyección falla, Mongo ya quedó confirmado: la consulta responde bien y agrega `aviso_grafo` con el texto `el grafo quedó pendiente`. Hoy todas las que entran muestran ese aviso, porque las cuatro funciones lanzan esa excepción.

Una escritura exitosa devuelve `{ok: true, documento, aviso_grafo?}`. Un rechazo devuelve `{ok: false, error}`.

## Carga y registros extra

`datasets/extra/` usa el mismo nombre y los mismos encabezados que el archivo de la cátedra. Los id siguen la serie (`C009`, `V017`, `L019`). La carga imprime, por colección, cuántos documentos son de la cátedra y cuántos propios. Falla si un id propio repite uno de la cátedra, si una referencia no existe o si el total de una venta no coincide con sus ítems.

Recargar borra lo creado con la 13 y la 14. Una venta que la 15 pasó a Completada vuelve a Pendiente, con la fecha y los ítems del archivo.

Para un caso FEFO que no dependa de la fecha, hace falta un producto con dos lotes de vencimiento lejano en la misma farmacia, y dos lotes que venzan el mismo día. El orden de descuento es `{vencimiento: 1, _id: 1}`.

## Qué sigue abierto

- Implementar las cuatro funciones de `src/grafo.py` y las consultas 2, 3 y 10, que leen ese camino.
- Las consultas 4, 6 y 12 sobre Mongo. La 6 compara `proxima_dispensa` con `hoy()`.
- Al menos diez registros propios por colección, en `datasets/extra/`.
- El diagrama de colecciones y el del grafo.
- Integrar el informe final. El material de entorno, carga y escrituras está en `docs/informe-entorno-y-escrituras.md`, y el de las seis lecturas, en `docs/informe-consultas-lectura.md` del repositorio del grupo.
