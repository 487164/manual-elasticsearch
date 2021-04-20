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

Ten en cuenta que, aunque un shard puede ejecutar múltiples queries concurrentemente, **las queries que afecten a un gran número de shards pueden agotar la pool de threads**, ya que cada shard ejecuta la query en un solo hilo. Esto implicará throughput bajo y búsquedas lentas.

### Cada shard tiene overhead

Cada shard asignado consume recursos de CPU y memoria, pero el verdadero peligro viene de los metadatos de los segmentos almacenados en el heap. La mayoría de los shards tienen varios [segmentos de Lucene](https://lucene.apache.org/core/8_8_2/core/org/apache/lucene/codecs/lucene87/package-summary.html#Segments), que contienen los datos de un índice y se van mergeando, cada vez en menos segmentos más grandes. Cada shard abierto consume una cantidad de heap fija en los metadatos de los segmentos, memoria que el GC no puede liberar. En la mayoría de los casos, **un conjunto pequeño de shards grandes utiliza menos recursos que muchos shards pequeños.**

### Eliminar índices en vez de documentos

Los documentos borrados no se eliminan realmente hasta que no se *fusionan* (mergean) los segmentos correspondientes, así que eliminar documentos no va a liberar ningún recurso hasta que no se dispare el siguiente merge. En caso de necesitar espacio, es mejor **eliminar los índices completamente en vez de sus documentos individualmente**, para que Elastic los elimine de inmediato.

### El tamaño de los shards debería estar entre 10GB y 65GB

Esta recomendación está basada en el tiempo necesario para que un único shard de 65GB se recupere de un fallo.

### Intentar tener como mucho 20 shards por GB de heap

Con esta recomendación se pretende evitar llenar el heap con metadatos de segmentos.

Se puede comprobar el número de shards que tiene un nodo con:
```
GET _cat/shards?pretty
```

### Evitar nodos "hotspot"

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

### Manejar nodos con demasiados shards
Si en un nodo (individual o perteneciente a un cluster) se están produciendo problemas de estabilidad debido a un número excesivo de shards, podemos utilizar alguno de los siguientes métodos:
* Eliminar índices vacios o innecesarios.
* Cambiar los índices más antiguos para que abarquen rangos de tiempo más grandes. De diario a mensual, de mensual a semestral, etc. Para ello podemos reindexar:
```
POST _reindex { "source": { "index": "my-index-2099.10.*" }, "dest": { "index": "my-index-2099.10" } }
```
* En índices en los que ya no se escribe, podemos forzar los merges en horas de poca carga utilizando:
```
POST my-index-000001/_forcemerge
```
* En índices en los que ya no se escribe, podemos reducir (*shrink*) el número de shards con:
```
POST my-index-000001/_shrink/my-shrunken-index-000001
```
