# Configuración de Elasticsearch

Bajo una instalación por defecto basada en los paquetes de Naudit, la configuración por defecto la establece el paquete [elasticsearch-conf](https://repo1.naudit.es/deb-repo/elasticsearch-conf).

El fichero de configuración principal de Elasticsearch se encuentra en 
`/etc/elasticsearch/elasticsearch.yml` y deberá contener por norma general (para ES7) la siguiente configuración:

```yml
path.data: /disco_flowlytics/elasticsearch/data
path.logs: /disco_flowlytics/elasticsearch/logs

node.name: node1
network.host: 0.0.0.0
http.port: 27015

bootstrap.memory_lock: true
cluster.initial_master_nodes: ['127.0.0.1']
```

En caso de encontrarse instalado el paquete [elasticsearch-conf](https://repo1.naudit.es/deb-repo/elasticsearch-conf), estos valores serán controlados por el fichero `/etc/naudit.d/elasticsearch.ini`.
El formato y los valores por defecto de susodicho fichero son los siguientes:

```ini
[Elastic.elasticsearch]
# Path for data (index)
path_data=/disco_flowlytics/elasticsearch/data
# Path for logs (mejor usar el raid para no quemar los SSDs del S.O.)
path_logs=/disco_flowlytics/elasticsearch/logs
# Set this to true if you don't want elastic to be swapped
bootstrap_memory_lock=true

## LISTENING
# Ip where elasticsearch should listen for. 0.0.0.0 means any available IP. Add as many IPs as you want without spaces and comma separated.
network_host=0.0.0.0
# HTTP port where requests should be served
http_port=27015
# Cluster master nodes. Should not be empty on ES7. Add as many IPs as you want without spaces and comma separated.
cluster_initial_master_nodes=127.0.0.1
```


**Notas**:
- Se establece el network.host a `0.0.0.0` para que Elasticsearch escuche tanto a peticiones externas como internas.
Si se desease restringir el acceso del exterior bastaría con cambiar este valor a `127.0.0.1` o en su defecto a alguna de las IPs privadas.
También es posible proveer un listado de IPs con la siguiente sintaxis: `["10.252.3.142", "127.0.0.1"]`

- Elasticsearch 7 requiere que se establezca alguno de los siguientes valores: `discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes` o en su defecto `discovery.type: single-node`.
La configuración de arriba es equivalente a `discovery.type: single-node` ya que, aunque Elasticsearch sigue funcionando en modo cluster, a efectos prácticos es el único nodo y actúa siempre como Máster.
La decisión de establecer un cluster de un único nodo frente a una configuración estricta mono-nodo se ha hecho por motivos de gestionabilidad de los scripts de generación de configuración automática.

## Definir la memoria heap para Elastic:

¿Cuánto heap? Por lo general, 50% de la RAM disponible. Nunca más de 32GB (es lo máximo que puede usar Elastic).

```yml
# si los valores de memoria máx. y mín. son iguales,
# elastic no necesita asignar memoria adicional en ejecución:
/bin/elasticsearch -Xmx16g -Xms16g
#               máximo^       ^mínimo

```

Se puede consultar el heap máximo asignado en el nodo actual con:

```
GET /_cat/nodes?h=heap.max
```

En instalaciones estándar el paquete [elasticsearch-conf](https://repo1.naudit.es/deb-repo/elasticsearch-conf) establece esta configuración a 16g por defecto.

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
