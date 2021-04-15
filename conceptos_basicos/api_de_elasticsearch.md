# API de Elasticsearch

Todo en Elasticsearch se expone mediante APIs REST, tanto los datos como la gestión de la herramienta. Por eso, se puede hacer todo a través de peticiones con `curl` o lo que sea.

Algunas opciones, como la flag `?pretty` al final de la URL o usar wildcards `*` dentro de los nombres, son comunes para cualquier endpoint.

[[doc] API conventions](https://www.elastic.co/guide/en/elasticsearch/reference/current/api-conventions.html)

Las APIs de gestión suelen ser las que empiezan por barra baja `_` (`_cat/indices`, `_templates`, `_nodes`...). Como hay muchas, lo mejor para saber cómo se pide algo concreto es consultar la documentación:

[[doc] REST APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html)
