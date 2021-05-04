# Conceptos básicos

[Elasticsearch](https://www.elastic.co/es/what-is/elasticsearch) es un motor de búsqueda y análisis basado en [Apache Lucene](https://lucene.apache.org/) y escrito en Java. Nosotros lo utilizamos como base de datos analítica. Principalmente, con Elastic almacenamos logs y usamos sus capacidades de filtrado y visualización (con [Kibana](https://www.elastic.co/es/what-is/kibana) o [Grafana](https://grafana.com/grafana/)) para identificar problemas o extraer información de interés.

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
    "system": "Ambiente Producción",
    "title": "Número elevado de eventos windows",
    "term": "SERVIDOR1"
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

- El *mapping* es una colección de *campos* que los documentos de un índice pueden tener (Elastic es NoSQL, los documentos no tienen por qué tener los mismos campos todos). En el ejemplo anterior, según el mapping del índice "alarms", sus documentos tendrán los campos "@timestamp" (el @ por convención indica que es el campo temporal, a partir del cual podrán dibujarse series temporales en Grafana/Kibana y filtrar por rango temporal), "severity", "value", "system", "title" y "term".
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

## Shards

El concepto de [shard](https://en.wikipedia.org/wiki/Shard_(database_architecture)) es bastante común en las bases de datos, y sería algo así como una partición,
un fragmento de los datos que están contenidos en un índice.
En Elasticsearch, un shard es una instancia de un índice de Lucene, aunque a nivel práctico no es lo verdaderamente relevante.

Elasticsearch utiliza el shard como la unidad con la que distribuye los datos en un cluster.
Existen **shards primarios**, que son los que realmente contienen los datos, y **shards réplica**, que ofrecen redundancia sobre esos datos.
La distribución de shards por los diferentes nodos la gestiona Elasticsearch automáticamente, de manera que, cuando un nodo falla, balancea los datos moviendo los shards necesarios.
Lógicamente, esta separación de clases de shards en primarios y réplicas solo tiene sentido si trabajamos con *clusters multi-nodo*.

Dividir los índices mediante shards facilita la escalabilidad horizontal e, independientemente del número de nodos, nos permite paralelizar consultas.

Al final de lo que nos tenemos que preocupar nosotros sobre todo es de tener **un número adecuado de shards para nuestro caso**,
tema que se aborda en la sección de [notas sobre asignación de shards](../como_funciona_elasticsearch/configuracion_de_elasticsearch.html#notas-sobre-asignación-de-shards)
