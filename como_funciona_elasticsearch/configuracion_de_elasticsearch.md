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
