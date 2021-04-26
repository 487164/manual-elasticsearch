# [elasticsearch](https://elasticsearch-py.readthedocs.io/en/v7.12.0/api.html#elasticsearch)
La librería base de elasticsearch incluye el objeto Elasicsearch (que necesitaremos también para elasticsearch_dsl) que actuará como connector entre nuestro código y Elastic.

Realizar consultas e insertar datos desde la librería base es algo más costoso que mediante elasticsearch_dsl ya que requiere escribir las consultas DSL directamente en JSON.

Como ventaja, incluye abstracciones para realizar tareas de monitorización (get cluster status, startup nodes, etc) 

Podríamos realizar una query con la librería base con este código:
```python
from elasticsearch import Elasticsearch
client = Elasticsearch()

response = client.search(
    index="my-index",
    body={
      "query": {
        "filtered": {
          "query": {
            "bool": {
              "must": [{"match": {"title": "python"}}],
              "must_not": [{"match": {"description": "beta"}}]
            }
          },
          "filter": {"term": {"category": "search"}}
        }
      },
      "aggs" : {
        "per_tag": {
          "terms": {"field": "tags"},
          "aggs": {
            "max_lines": {"max": {"field": "lines"}}
          }
        }
      }
    }
)

for hit in response['hits']['hits']:
    print(hit['_score'], hit['_source']['title'])

for tag in response['aggregations']['per_tag']['buckets']:
    print(tag['key'], tag['max_lines']['value'])
```


En caso de que tengamos problemas al trasladar una query a elasticsearch_dsl, pero sepamos escribirla en DSL JSON podemos utilizar la librería base para lanzar la consulta.