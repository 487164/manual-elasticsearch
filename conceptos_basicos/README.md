# Conceptos básicos

[Elasticsearch](https://www.elastic.co/es/what-is/elasticsearch) es un motor de búsqueda y análisis basado en [Apache Lucene](https://lucene.apache.org/). Nosotros lo utilizamos como base de datos analítica. Principalmente, con Elastic almacenamos logs y usamos sus capacidades de filtrado y visualización (con [Kibana](https://www.elastic.co/es/what-is/kibana) o [Grafana](https://grafana.com/grafana/)) para identificar problemas o extraer información de interés.

## Índices y documentos

Elasticsearch organiza los datos en *índices*. Cada índice es una colección de *documentos* JSON. Cuando se inserta un documento en un índice, Elasticsearch añade algunos metadatos, como el "_id" único que identifica al documento.

```
{
"_index": "alarms",
"_type": "_doc",
"_id": "wjeS_HcB6pXqTmlO1Dfg",
"_score": 1.0,
"_source": {
    "@timestamp": 1614850217137,
    "severity": "4",
    "value": 9990.0,
    "system": "Portal Financiero",
    "title": "Número elevado de eventos windows",
    "term": "BOCAY"
}
```

> Observación importante:

Si te fijas en el valor del campo `@timestamp`, verás que parece un timestamp en formato [epoch](https://en.wikipedia.org/wiki/Unix_time), pero al convertirlo:

```
$ date -d @1614850217137
Sun 26 Jul 53142 06:45:37 PM CST
```

¡Da una fecha de un futuro muy lejano! Esto es porque **Elasticsearch trabaja en milisegundos**. En realidad ese timestamp es:

```
$ date -d @1614850217.137
Thu 04 Mar 2021 03:30:17 AM CST
```

Los índices también tienen *mappings* y configuración:

- El *mapping* es una colección de *campos* que los documentos de un índice deben tener. En el ejemplo anterior, según el mapping del índice "alarms", sus documentos tendrán los campos "@timestamp" (el @ por convención indica que es el campo temporal, a partir del cual podrán dibujarse series temporales en Grafana/Kibana y filtrar por rango temporal), "severity", "value", "system", "title" y "term".
- La configuración incluye el nombre del índice, su fecha de creación y el número de shards.

```
$ curl localhost:27015/alarms 2>/dev/null | python -m json.tool
{
    "alarms": {
        "aliases": {},
        "mappings": {
            "properties": {
                "@timestamp": {
                    "type": "date"
                },
                "severity": {
                    "type": "float"
                },
 ...
        },
        "settings": {
            "index": {
                "number_of_shards": "1",
                "provided_name": "alarms",
                "creation_date": "1613657679969",
                "number_of_replicas": "0",
                "uuid": "QiBdWS0PTxyosbAhrAcC7w",
 ...
            }
        }
    }
}
```

En versiones antiguas de Elasticsearch, se permitía el uso arbitrario de "tipos de documento" (y mappings distintos) en un mismo índice, con el campo "_type". En las últimas versiones, este campo siempre tiene el valor "_doc" y se mantiene el mismo mapping para todo el índice.

## Queries

## Shards

[Notas sobre asignación de shards](../como_funciona_elasticsearch/configuracion_de_elasticsearch.html#notas-sobre-asignación-de-shards)

## API de Elasticsearch
