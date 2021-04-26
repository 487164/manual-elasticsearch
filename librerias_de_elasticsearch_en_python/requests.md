# requests
Podemos interactuar con ES directamente por las APIs utilizando la librería requests (a base de POSTs, GETs...).

Deberemos elegir el método adecuado para cada API y formatear los datos en JSON.

requests es recomendable para automatizar tareas relacionadas con las APIs de gestión (por ejemplo, borrar un index o cargar un template) 

Por ejemplo, la siguiente function llama a la API de reindex con requests

```python
def reindex(source, target):
    reindex_template = '''
        {
            "source": {
                "index": "",
                "size": "50000"
            },
            "dest": {
                "index": ""
            }
        }
    '''
    reindex_url = "http://localhost:27015/_reindex?pretty"

    headers = {"Content-Type": "application/json"}



    reindex_template = json.loads(reindex_template)

    reindex_template['source']['index'] = source
    reindex_template['dest']['index'] = target

    r = requests.post(reindex_url, data=json.dumps(reindex_template), headers=headers)

    print(json.dumps(r.status_code))
    print(json.dumps(r.text))
```
