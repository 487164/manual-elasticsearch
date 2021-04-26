# [elasticsearch_dsl](https://elasticsearch-dsl.readthedocs.io/en/latest/search_dsl.html)
elasticsearch_dsl está construida directamente sobre la librería base y su objetivo es hacer consultas a ES más rápido y cómodo.

Para ello trata de replicar la estructura del JSON DSL, pero dentro de la sintaxis de Python (sin necesidad de declarar un JSON string con la consulta).

La consulta de ejemplo presentada en la sección anterior, podría escribirse de la siguiente forma:

```python
from elasticsearch import Elasticsearch
from elasticsearch_dsl import Search

client = Elasticsearch()

s = Search(using=client, index="my-index") \
    .filter("term", category="search") \
    .query("match", title="python")   \
    .exclude("match", description="beta")

s.aggs.bucket('per_tag', 'terms', field='tags') \
    .metric('max_lines', 'max', field='lines')

response = s.execute()

for hit in response:
    print(hit.meta.score, hit.title)

for tag in response.aggregations.per_tag.buckets:
    print(tag.key, tag.max_lines.value)
```

En nuestro ejemplo hemos visto como la librería se ha encargado de:

- Crear los objetos Query apropiados basándose en el nombre utilizado (match en este caso)
- Unir las distintas Querys sin necesidad de especificar los bloques bool
- Convertir la query en un filter ya que hemos especificado .filter() en lugar de .query()
- Organizar los datos de forma lógica haciendo más fácil actuar sobre ellos.

En la documentación podemos encontrar ejemplos de persistencia de datos usando clases.

### Tips y consejos

La [documentación de elasticsearch_dsl](https://elasticsearch-dsl.readthedocs.io/) va completándose con el tiempo y cada vez es menos necesario mirar el código para entender funcionalidades, pero es posible que quieras hacer algo que no aparece claramente en la documentación. Para estos casos, el proceso ha seguir debería ser el siguiente:

- Preguntar en el canal de off-topic de Naudit (por ejemplo) para ver si alguien ya ha resuelto el problema (en caso de ser así, igual es interesante actualizar esta documentación con esa información)
- Búsqueda en internet (los foros de elastic, stackoverflow, github..)
- Tratar de trasladar la query a DSL JSON y emplear la librería base
- Búsqueda dentro del [github de elasticsearch_dsl](https://github.com/elastic/elasticsearch-dsl-py)
- Mirar el código fuente de la librería

#### Referencias a *dotted fields* (o subcampos, por ejemplo agent.hostname)

Para referirnos a un campo anidado, por ejemplo `agent.hostname` debemos emplear una sintaxis especial, ya que `agent.hostname` no es python válido.

La librería soluciona este problema reemplazando el carácter `.` con `__`, por ejemplo:

```python
s = Search()
s = s.filter('term', category__keyword='Python')
s = s.query('match', address__city='prague')
```

Otra opción (que además es útil al referirnos a otros campos que también dan problemas, como `@timestamp`) es utilizar kwarg unpacking:
```python
s = Search()
s = s.filter('term', **{'category.keyword': 'Python'})
s = s.query('match', **{'address.city': 'prague'})
```
Recientemente la documentación se ha actualizado para incluir esta información: [dotted fields](https://elasticsearch-dsl.readthedocs.io/en/latest/search_dsl.html#dotted-fields)
