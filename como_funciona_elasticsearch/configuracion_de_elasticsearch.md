# Configuración de Elasticsearch

`/etc/elasticsearch/elasticsearch.yml`:

```
path.data: /disco_flowlytics/elasticsearch/data
path.logs: /var/log/elasticsearch

node.name: GyT_node1
network.host: ["10.252.3.142", "127.0.0.1"]
http.port: 27015

bootstrap.memory_lock: true
discovery.type: single-node
```

## Notas sobre asignación de shards

[[doc] Size your shards](https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html)

### CPU/Threads

Ten en cuenta que, aunque una query afecte a múltiples shards, cada shard ejecuta la query en un solo hilo.

### Cada shard tiene overhead

Cada shard asignado consume recursos de CPU y memoria, pero el verdadero peligro viene de los metadatos de los segmentos almacenados en el heap. La mayoría de los shards tienen varios [segmentos de Lucene](https://lucene.apache.org/core/8_8_2/core/org/apache/lucene/codecs/lucene87/package-summary.html#Segments), que contienen los datos de un índice y se van mergeando, cada vez en menos segmentos más grandes. Cada shard abierto consume una cantidad de heap fija en los metadatos de los segmentos, memoria que el GC no puede liberar.

### Eliminar índices en vez de documentos

Los documentos borrados no se eliminan realmente hasta que no se *fusionan* (mergean), así que eliminar documentos no va a liberar ningún recurso hasta que no se dispare el siguiente merge.

### El tamaño de los shards debería estar entre 10GB y 50GB

Esta recomendación está basada en el tiempo necesario para que un único shard de 50GB se recupere de un fallo.

### Intentar tener como mucho 20 shards por GB de heap

Con esta recomendación se pretende evitar llenar el heap con metadatos de segmentos.

Se puede comprobar el número de shards que tiene un nodo con:
```
GET _cat/shards?pretty
```

### Evita nodos "hotspot"

Esto solo es relevante para configuraciones de cluster.

Como Elastic asigna shards automáticamente a los nodos, alguno de estos puede convertirse en un "punto peligroso", en caso de que tenga muchos shards para un índice con un alto volumen de indexado.

Para evitar hotspots, se puede configurar el siguiente valor por índice:
```
PUT /my-index-000001/_settings
{
  "index": {
    "routing.allocation.total_shards_per_node" : 5
  }
}
```
