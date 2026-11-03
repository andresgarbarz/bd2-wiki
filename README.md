# Bases de Datos II — wiki para agentes

Este repositorio convierte el material de la cátedra en una base de conocimiento trazable que puede usar una persona o un agente iniciado desde la raíz. Está preparado para estudiar, resolver guías prácticas, tomar notas e incorporar clases o unidades nuevas sin mezclar fuentes oficiales con interpretaciones.

## Empezar

1. Abrí Codex o Claude Code en la raíz del repositorio.
2. Preguntá en lenguaje natural, por ejemplo:
   - `Explicame la diferencia entre clave y superclave.`
   - `Ayudame con el ejercicio 1 del TP 1.`
   - `Tomame un parcial corto sobre joins.`
3. El agente consulta primero la [wiki](wiki/README.md), verifica el contenido contra las fuentes y cita con identificadores como `[T02, p. 14]`.

El modo de resolución es adaptativo: ante una consigna ambigua, el agente ofrece pista, resolución acompañada o solución completa. Si el pedido ya dice `resolvelo`, `dame la solución` o equivalente, responde directamente.

## Operaciones habituales

| Quiero… | Pedido al agente | Alternativa |
|---|---|---|
| Estudiar o resolver | `Ayudame a estudiar…` | `$estudiar-bd2` |
| Incorporar un PDF o unidad | `Incorporá el material nuevo` | `$incorporar-material` o `/incorporar-material` en Claude |
| Integrar notas de clase | `Procesá mis notas pendientes` | `$procesar-notas` o `/procesar-notas` en Claude |
| Crear una nota | `./scripts/nueva-nota.sh "tema"` | Copiar [la plantilla](notas/PLANTILLA.md) a `notas/bandeja/` |
| Buscar en todo el conocimiento | `./scripts/buscar.sh "texto"` | `rg -n -i "texto" wiki material/extraido notas` |
| Validar la organización | `./scripts/verificar-repo.sh` | — |

Para incorporar archivos, colocalos sin reorganizar en `material/entrada/` y pedile al agente que los incorpore. Para notas, usá `notas/bandeja/`. Las notas originales se conservan y la wiki se actualiza automáticamente.

## Mapa del repositorio

```text
wiki/                 apuntes curados y navegables
material/
  catedra/            originales oficiales, inmutables
  extraido/           texto por página para búsquedas y citas
  entrada/            bandeja para material nuevo
notas/
  bandeja/            notas todavía no integradas
  procesadas/         originales ya integrados, sin reescritura
practica/             espacio individual para resolver las guías
laboratorio/          MySQL local reproducible
entregas/             trabajo obligatorio grupal (TPO FarmaRed)
.agents/skills/       flujos reutilizables de Codex
.claude/commands/     comandos equivalentes de Claude Code
```

La política completa para agentes está en [AGENTS.md](AGENTS.md). El catálogo de fuentes y la convención de citas están en [material/README.md](material/README.md).

## MySQL local

El laboratorio usa MySQL 9.7.2, la versión indicada por la Clase I, y credenciales exclusivamente locales. Requiere Docker con Compose.

```bash
cp .env.example .env
make db-up
make db-shell
```

Consultá [laboratorio/README.md](laboratorio/README.md) antes de reiniciar la base: `make db-reset` elimina el volumen local y vuelve a ejecutar los scripts de inicialización.
