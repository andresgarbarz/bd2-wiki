# Amazon DynamoDB

## El servicio y el paper

DynamoDB es un servicio NoSQL administrado de AWS. El material lo describe como almacén clave-valor y también como almacén de documentos, con los datos en SSD y replicados entre zonas de disponibilidad de una región. El anuncio que cita la clase es el de Werner Vogels el 18 de enero de 2012. No es el sistema del paper *Dynamo* (2007): la clase hace esa pregunta de forma explícita y resume el paper como modelo no relacional, disponibilidad por encima de consistencia y escalabilidad. [T20, pp. 2–5]

Un **ítem** equivale, en la analogía de la clase, a una fila. Tiene atributos y un tope de 400 KB. Al crear la tabla se declaran los atributos de la clave; al insertar se pueden agregar otros, y la clase invita a discutir si eso es esquema libre. El JSON de la API no es el de MongoDB: cada valor lleva un tipo (`S`, `N`, …). La clase advierte que no se usan cadenas vacías. [T20, pp. 4, 10–16]

La consistencia que describe es por ítem, análoga a una fila. No hay operaciones atómicas entre ítems, y una consulta sobre un índice o sobre la tabla entera puede no ver el estado más nuevo. Una lectura fuertemente consistente cuesta alrededor del doble que una eventualmente consistente. [T20, pp. 6, 23]

## Claves e índices

La clave de partición es la **hash key**. La clase anota que “PK” ahí no significa primary key. La **sort key** es la range key y ordena búsquedas dentro de la partición. Una tabla como `ShoppingCart` puede tener solo hash (`ItemName`). Una tabla como `Books` puede combinar hash (`Title`) y rango (`PublishYear`): el mismo título admite varias ediciones y un `query` puede pedir `PublishYear > :year` junto con la igualdad del título. Proyectar un atributo ausente devuelve un ítem vacío para esa edición, no omite la fila. [T20, pp. 10–16]

| Índice | Alcance | Después de crear la tabla |
|---|---|---|
| Local (LSI) | Otra sort key dentro de la misma partition key. | No se modifica. Hay que planificarlo al crear la tabla. |
| Global (GSI) | No está atado a la misma partition key; la clase lo resume como “cualquier atributo”. | Sí se puede agregar después. |

El ejemplo de LSI de la clase es `TemperatureIndex` sobre `SensorData`, con `SensorId` como hash y `Temperature` como rango. [T20, pp. 17–22]

## Capacidad y acceso local

En el modo provisionado, la clase mide un RCU como 4 KB por segundo y un WCU como 1 KB por segundo. Esos valores se pueden cambiar después. Los ejemplos crean tablas con 1 y 1. `scan` recorre la tabla; `get-item` busca por clave; `query` usa una condición de clave. [T20, pp. 10, 13–16, 22–23]

La guía y la clase usan DynamoDB Local porque el servicio de AWS es cloud. El contenedor de la guía es `amazon/dynamodb-local`, nombre `Mydynamodb`, puerto 8000. Cada comando de AWS CLI lleva `--endpoint-url http://localhost:8000`. `aws configure` acepta credenciales de relleno; la guía propone `1234` / `567890`, región `us-east-1` y salida `json`. [P17, p. 1; T20, pp. 7–9]

El resto de T20 sigue el libro hacia un pipeline con Kinesis y Lambda, exportación a S3 y consulta con Athena, clientes como boto3, y un enlace a diseño de tabla única. Esos tramos nombran servicios de AWS que el TP 13 no pide implementar. [T20, pp. 20–30, 34]

## CAP

T20 no asigna una etiqueta CAP al servicio DynamoDB. El paper, en el resumen de la clase, prioriza disponibilidad sobre consistencia. La [guía visual](../../material/figuras/T20-p31-guia-visual-cap.png) coloca **Dynamo** —el sistema del paper, no el nombre DynamoDB— del lado AP, junto con Cassandra, Riak, CouchDB, SimpleDB, Voldemort, Tokyo Cabinet y KAI. En esa misma lámina Redis queda del lado CP. La clasificación CA de Redis está en la unidad anterior y no se copia desde esta figura. [T20, pp. 5, 31; T19, p. 36]

## Fortalezas y debilidades que copia la clase

La clase destaca no instalar un servidor, el crecimiento del servicio si el presupuesto alcanza, índices y consultas por rango. Las dificultades que enumera son el aprovisionamiento, el diseño de índices, encajar el modelo con el particionamiento y el costo. Sugiere una base relacional cuando el problema no tiene esa forma. [T20, pp. 32–33]

## TP 13

La guía se apoya en el capítulo 7 del libro (pp. 218–233): CRUD de `ShoppingCart` y `Books`, diferencia entre clave simple y compuesta, diferencia entre LSI y GSI, particiones, y dos ejercicios Do. Cierra con fortalezas, debilidades y la clasificación CAP. No fija la versión de la imagen Docker. [P17, pp. 1–2; B01]

## TPO

FarmaRed pide al menos dos motores NoSQL de paradigmas distintos. DynamoDB es uno de los candidatos que el material ya caracterizó; la consigna no lo impone. El detalle de entrega está en [el espacio del TPO](../../entregas/tpo-farmared/README.md). [P18, p. 5]

## Continuar

- [Clave-valor y Redis](24-redis-clave-valor.md): el otro motor de la familia y la otra etiqueta CAP.
- [CAP](15-nosql-y-cap.md).
- [Guía TP 13](../../practica/tp-13/README.md).
