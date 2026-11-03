# TP 13 — DynamoDB

- Fuente: [P17](../../material/catedra/practica/tp-13-dynamodb.pdf), dos páginas, ITBA 2026.
- Apunte: [DynamoDB](../../wiki/temas/25-dynamodb.md).
- Libro que cita la guía: [B01](../../material/catedra/bibliografia/b01-seven-databases-in-seven-weeks-2e.pdf), capítulo 7. La copia local de B01 no está versionada.
- Estado: consigna incorporada, sin resolución ni ejecución.

## Preparación indicada por la guía

La guía dice que el servicio de AWS no se instala: es cloud. Para la práctica usa DynamoDB Local:

```bash
docker pull amazon/dynamodb-local
docker run --name Mydynamodb -p 8000:8000 -d amazon/dynamodb-local
```

Hace falta AWS CLI, en el host o dentro del contenedor (`docker exec -u 0 -it Mydynamodb bash` si faltan paquetes). `aws configure` con access key `1234`, secret `567890`, región `us-east-1` y salida `json`. Cada comando lleva `--endpoint-url http://localhost:8000`. No hay etiqueta de versión de la imagen. El laboratorio de este repositorio todavía no incluye el servicio. [P17, p. 1]

## Ejercicios

| Ejercicio | Alcance | Página |
|---|---|---|
| 1 | Capítulo 7, pp. 218–227: CRUD de `ShoppingCart` y `Books`. | 2 |
| 2 | Diferencia entre clave simple y clave compuesta, pp. 223–224. | 2 |
| 3 | Diferencia entre índice secundario local y global, pp. 230–232. | 2 |
| 4 | Find.1, p. 232, sobre particiones. | 2 |
| 5 | Do.1 y Do.3, p. 233. | 2 |
| 6 | Fortalezas, debilidades y cuándo usarlo. | 2 |
| 7 | Clasificación según CAP. | 2 |

[P17, p. 2]

T20 no etiqueta el servicio DynamoDB como CP o AP. El paper de 2007, en el resumen de la clase, prioriza disponibilidad sobre consistencia, y la guía visual ubica a Dynamo —no el rótulo DynamoDB— del lado AP. Al responder el ejercicio 7 hay que decir qué sistema se está clasificando. [T20, pp. 5, 31]
