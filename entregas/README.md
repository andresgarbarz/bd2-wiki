# Entregas

El trabajo obligatorio grupal es el TPO FarmaRed. La consigna oficial es [P18](../material/catedra/practica/tpo-farmared.pdf). El repositorio del grupo es [TPO_BD2_FarmaRed](https://github.com/andresgarbarz/TPO_BD2_FarmaRed) y la copia local está en `tpo-farmared/`. Esa carpeta no se versiona dentro de esta wiki.

Los motores elegidos y las alternativas descartadas están en [decisiones-arquitectura.md](decisiones-arquitectura.md). Los avances para continuar el código están en [avances-tpo.md](avances-tpo.md). Las decisiones de entorno, carga y escrituras, para el informe, están en [decisiones-escrituras.md](decisiones-escrituras.md).

Q01, Q05, Q07, Q08, Q09 y Q11 están implementadas y verificadas con MongoDB real. El código está publicado en la rama [feat/consultas-lectura-mongodb](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/tree/feat/consultas-lectura-mongodb), con un commit por consulta y sus pruebas, más uno de documentación y el fix posterior de Q05. La lectura de la consigna, el plan ejecutado, los casos de control y los pendientes están en [consultas-asignadas.md](consultas-asignadas.md). Los ejemplos para el informe están en [informe-consultas-lectura.md](https://github.com/andresgarbarz/TPO_BD2_FarmaRed/blob/11c9d1b622b2c15a4f2ccc070cf1862ff02d129f/docs/informe-consultas-lectura.md).

La documentación de esta wiki se mantiene en la rama `docs/consultas-lectura-mongodb` del repositorio exterior.

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
