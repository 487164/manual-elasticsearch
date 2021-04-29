# Templates

Una [plantilla](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html)
sirve para definir algunas configuraciones a la hora de crear ciertos índices (los que coincidan con los *index_patterns*).
Se les puede dar cualquier nombre y se aplican según el orden establecido en el campo *order*:

```
naudit@sonda:~$ curl localhost:27015/_cat/templates?v
name                              index_patterns               order
default_shards                    [*]                          1
importer                          [importer-*]                 3
```

Las configuraciones que establece una plantilla sobre un índice son configuraciones por defecto. Es decir, que se pueden modificar sobre
el propio índice a posteriori.

Por ejemplo, así se crearía una plantilla "default shards" que se aplica en primer lugar y fija en 1 el número de shards de todos los índices:

```
naudit@sonda:~$ curl -XPUT localhost:27015/_template/default_shards?pretty -H 'Content-Type: application/json' -d'
{
"index_patterns" : ["*"],
"order" : 1,
"settings" : {"number_of_shards": 1}
}'
# respuesta:
{
  "acknowledged" : true
}
naudit@sonda:~$ curl localhost:27015/_template/default_shards?pretty # (comprobación)
{
  "default_shards" : {
    "order" : 1,
    "index_patterns" : [
      "*"
    ],
    "settings" : {
      "index" : {
        "number_of_shards" : "1"
      }
    },
    "mappings" : { },
    "aliases" : { }
  }
}

```

Sin embargo, la siguiente plantilla "importer" sobreescribirá con 2 shards en la configuración de los índices que empiecen por "importer-*".
Además, establecerá number_of_replicas=0 (lo que tiene sentido si trabajamos con un único nodo).

Usamos la plantilla "importer" para definir también el tipo de los campos que importamos a Elastic, de forma que un campo llamado "occurrences_int"
tenga datos de tipo entero, "src_ip" sea de [tipo IP](https://www.elastic.co/guide/en/elasticsearch/reference/current/ip.html), etc.

```
naudit@sonda:~$ curl localhost:27015/_template/importer?pretty
{
  "importer" : {
    "order" : 3,
    "index_patterns" : [
      "importer-*"
    ],
    "settings" : {
      "index" : {
        "number_of_shards" : "2",
        "number_of_replicas" : "0"
      }
    },
    "mappings" : {
      "dynamic_templates" : [
        {
          "ips" : {
            "mapping" : {
              "type" : "ip"
            },
            "match" : "*_ip"
          }
        },
        {
          "ints" : {
            "mapping" : {
              "type" : "integer"
            },
            "match" : "*_int"
          }
        },
        {
          "longs" : {
            "mapping" : {
              "type" : "long"
            },
            "match" : "*long"
          }
        },
        {
          "floats" : {
            "mapping" : {
              "type" : "float"
            },
            "match" : "*_float"
          }
        },
        {
          "texts" : {
            "mapping" : {
              "type" : "text"
            },
            "match" : "*_text"
          }
        },
        {
          "keywords" : {
            "unmatch" : "*_text",
            "mapping" : {
              "type" : "keyword"
            },
            "match_mapping_type" : "string"
          }
        }
      ],
      "properties" : {
        "count" : {
          "type" : "long"
        }
      }
    },
    "aliases" : { }
  }
}
```

En cualquier caso, las plantillas son solo ayudas para la configuración por defecto de los índices.
No son obligatorias y los valores de configuración de cada índice pueden modificarse según sea necesario.
