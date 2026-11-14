# Entregas

El trabajo obligatorio grupal es el TPO FarmaRed. La consigna oficial es [P18](../material/catedra/practica/tpo-farmared.pdf). El repositorio del grupo es [TPO_BD2_FarmaRed](https://github.com/andresgarbarz/TPO_BD2_FarmaRed) y la copia local está en `tpo-farmared/`. Esa carpeta no se versiona dentro de esta wiki.

Los motores elegidos y las alternativas descartadas están en [decisiones-arquitectura.md](decisiones-arquitectura.md). Los avances para continuar el código están en [avances-tpo.md](avances-tpo.md). Las decisiones de entorno, carga y escrituras, para el informe, están en [decisiones-escrituras.md](decisiones-escrituras.md).

Las quince consultas están implementadas e integradas en `main` del repositorio del grupo: Q02, Q03 y Q10 usan Neo4j; las demás, MongoDB. La revisión del 14 de noviembre de 2026 ejecutó 25 pruebas de integración contra ambos motores reales. Se resolvió el conflicto del cargador conservando la vista Q11 y la reconstrucción del grafo. El detalle está en [avances-tpo.md](avances-tpo.md).

Los ejemplos para el informe están en [lecturas MongoDB](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/blob/main/docs/informe-consultas-lectura.md), [grafo](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/blob/main/docs/informe-grafo.md) y [Q04/Q06/Q12](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/blob/main/docs/informe-consultas-restantes.md). Faltan registros adicionales, el diagrama de colecciones y consolidar el informe PDF. [P18, pp. 1, 6–7]

Los nueve CSV provistos están en [datasets/](../datasets/), en la raíz de esta wiki. Se verificó que son idénticos a los de `entregas/tpo-farmared/datasets/`, que es la carpeta que lee el cargador del proyecto. Los registros adicionales del grupo van en `entregas/tpo-farmared/datasets/extra/`.

## Clonar y actualizar el código

Desde la raíz de esta wiki, si todavía no existe la copia local:

```bash
git clone https://github.com/andresgarbarz/TPO_BD2_FarmaRed.git entregas/tpo-farmared
```

Es un repositorio Git independiente, ignorado por el `.gitignore` de la wiki; no es un submódulo. El código, los datasets propios y el material del informe se trabajan allí. Esta carpeta `entregas/` conserva el contexto y los avances para retomar el trabajo.

Antes de actualizar una copia existente, revisar sus cambios:

```bash
git -C entregas/tpo-farmared status --short --branch
```

Con la copia limpia, actualizar sin crear un merge:

```bash
git -C entregas/tpo-farmared pull --ff-only
```

Si hay cambios locales o ramas divergentes, conservarlos y resolver esa situación antes de actualizar. Después de cada avance, actualizar el estado en esta wiki y la documentación correspondiente en el repositorio del grupo. No hacer commit ni push sin pedido explícito.

Las guías TP son individuales y no son esta entrega. Sus resoluciones permanecen en `practica/`.
