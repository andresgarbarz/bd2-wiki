# Instrucciones para agentes — Bases de Datos II

## Misión

Ayudar a estudiar y trabajar sobre Bases de Datos II usando principalmente el material de la cátedra. Mantener la wiki correcta, trazable y fácil de recuperar. Responder en español salvo que el usuario pida otro idioma.

## Orden obligatorio de fuentes

1. `wiki/`: síntesis navegable; usarla para ubicar conceptos.
2. `material/extraido/` y `material/catedra/`: verificar afirmaciones y páginas.
3. `notas/procesadas/`: aportes personales de clase.
4. Conocimiento general o fuentes externas: solo para completar faltantes; etiquetarlo explícitamente.

La fuente oficial prevalece ante un conflicto. No ocultar divergencias: registrarlas en `wiki/dudas-y-conflictos.md`. No atribuir a la cátedra algo inferido. No modificar archivos dentro de `material/catedra/`.

## Citas y trazabilidad

- Citar afirmaciones académicas como `[ID, p. N]`; los ID están en `material/catalogo.tsv`.
- Citar SQL de apoyo como `[SQL01]` y notas como `[N-AAAA-MM-DD-slug]`.
- Si una respuesta no está respaldada por el repo, decir: `Complemento del agente (no consta en el material cargado)`.
- En respuestas breves, agrupar citas al final del párrafo; en soluciones, citar la regla aplicada.
- No afirmar que una imagen o diagrama dice algo basándose solo en texto extraído. Abrir o renderizar la página del PDF cuando el gráfico sea necesario.

## Cómo buscar antes de responder

1. Leer `wiki/README.md` y la página temática relevante.
2. Buscar términos y sinónimos con `rg -n -i` en `wiki/` y `material/extraido/`.
3. Verificar el pasaje exacto y su número de página.
4. Para consignas, leer completa la fuente práctica y cualquier esquema asociado.
5. Responder con el alcance y dialecto correctos.

## Tutor adaptativo

- Si el usuario pide ayuda sin indicar profundidad, ofrecer tres modos en una sola pregunta: pista, resolución acompañada o solución completa.
- Si pide explícitamente resolver, corregir, dar la respuesta o verificar una solución, no frenar para preguntar el modo.
- No revelar accidentalmente la solución al ofrecer una pista.
- Para correcciones, distinguir errores conceptuales, de dialecto y de estilo.
- Usar MySQL para bases relacionales. Si el material usa sintaxis de PostgreSQL u otro motor, señalar la diferencia y dar la versión MySQL.

## Cambios permitidos por área

- `wiki/`: actualizar al incorporar material o procesar notas.
- `material/catedra/`: solo el flujo `$incorporar-material` puede agregar o reubicar originales; nunca reescribirlos.
- `material/extraido/`: contenido derivado; regenerable con `scripts/extraer-fuentes.sh`.
- `notas/bandeja/`: entrada del usuario; conservar contenido al procesar.
- `notas/procesadas/`: archivo histórico inmutable salvo pedido explícito.
- `practica/`: código y respuestas individuales a guías.
- `entregas/`: trabajo obligatorio grupal. La consigna vigente es el TPO FarmaRed.

## Flujos reconocibles

- Ante `incorporá/organizá/indexá este material`, usar `$incorporar-material`.
- Ante `procesá/integrá/enriquecé con mis notas`, usar `$procesar-notas`.
- Ante preguntas, repaso, simulacros o ejercicios de la materia, usar `$estudiar-bd2`.

Al terminar cambios de conocimiento, ejecutar `./scripts/verificar-repo.sh` y resumir fuentes agregadas, páginas de wiki actualizadas, conflictos y vacíos detectados. No hacer commit ni push salvo pedido explícito.
