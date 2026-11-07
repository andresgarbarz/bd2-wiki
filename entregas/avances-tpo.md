# Avances del TPO — Andrés Garbarz

Fecha: 7 de noviembre de 2026.

El código está en el repositorio del grupo, [TPO_BD2_FarmaRed](https://github.com/andresgarbarz/TPO_BD2_FarmaRed). La carpeta `entregas/tpo-farmared/` de esta wiki no se versiona. La justificación para el informe está en [decisiones-escrituras.md](decisiones-escrituras.md). Los motores, en [decisiones-arquitectura.md](decisiones-arquitectura.md).

## Qué quedó hecho

- `compose.yaml` con MongoDB `8.3.11` y Neo4j `5.26.32`, sin autenticación y con los puertos solo en `127.0.0.1`. El devcontainer levanta los dos al crearse.
- `src/db.py`: `MONGO_URI`, `NEO4J_URI` y `hoy()`, la fecha de Argentina a medianoche.
- `scripts/cargar.py`: borra las ocho colecciones, las vuelve a llenar, valida extras y al final llama a `reconstruir()`.
- `src/consultas/q13.py`, `q14.py` y `q15.py`, con la validación compartida en `src/reglas.py`.
- `src/grafo.py`: `reconstruir()`, `upsert_cliente`, `upsert_venta` y `upsert_receta`. Están vacías. Reciben el documento tal como quedó en Mongo.

Las consultas 1 a 12 siguen en `NotImplementedError`.

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
python3 -c "from consultas.q13 import ejecutar; print(ejecutar('baja', id_cliente='C009'))"
python3 -c "from consultas.q14 import ejecutar; print(ejecutar('C006', 'F001', [{'id_producto': 'PRD003', 'cantidad': 50}], 'Obra social', 'RC003'))"
python3 -c "from consultas.q15 import ejecutar; print(ejecutar('V011'))"
```

Fuera del Codespace, si `MONGO_URI` y `NEO4J_URI` no están definidas, el código usa `127.0.0.1`.

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
- Las consultas 1, 4, 5, 6, 7, 8, 9, 11 y 12 sobre Mongo. La 1 resuelve `id_proveedor` contra `proveedores`. La 6 compara `proxima_dispensa` con `hoy()`.
- Al menos diez registros propios por colección, en `datasets/extra/`.
- El diagrama de colecciones y el del grafo.
- Cada quien redacta en el informe la parte que implementa. El material de entorno, carga y escrituras ya está en `docs/informe-entorno-y-escrituras.md` del repositorio del grupo.
