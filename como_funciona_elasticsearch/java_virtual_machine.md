# Java Virtual Machine
## ¿Qué es una Java Virtual Machine?
Una *Java Virtual Machine* no es más que una máquina virtual capaz de ejecutar [bytecode de Java](https://es.wikipedia.org/wiki/Bytecode_Java).
En general, se utiliza como puente entre el entorno de ejecución y el sistema físico que ejecutará realmente la aplicación Java.

El motivo principal por el que se utilizan las JVMs para aplicaciones en Java es por la portabilidad, permitiendo a estas ejecutarse en cualquier sistema que posea una JVM.

## ¿Por qué Elasticsearch utiliza JVMs?
Elasticsearch utiliza JVMs porque está escrito en Java. [Lucene](../como_funciona_elasticsearch/lucene.html) es una librería de búsqueda en Java y es la base de Elastic.
Es por esto que es necesario conocer las características y necesidades de nuestro proyecto para definir las especificaciones de la JVM donde se ejecutará.

## Conceptos importantes acerca de las JVMs
Algunos de los aspectos a tener en cuenta acerca de las JVMs, que nos servirán durante el manejo de Elastic, están relacionados con cómo gestiona la memoria:
* Heap de memoria: Una JVM tiene asignada una memoria máxima que toma del sistema que la contiene. La JVM manejará esta memoria internamente y se la irá asignando a los diferentes procesos. Es importante definirla acorde a las necesidades del proyecto. Puedes obtener más información en la sección de [configuración de Elasticsearch](../como_funciona_elasticsearch/configuracion_de_elasticsearch.html).
* Garbage Collection: Es una tarea periódica que ejecuta automáticamente la JVM para liberar memoria que ya no está en uso, de forma que pueda ser usada por otros procesos. Se debe controlar que esta tarea se haga correctamente y que no se llegue al límite de memoria. Hay más información en la sección de [monitorización de Elasticsearch](../como_funciona_elasticsearch/monitorizacion_de_elasticsearch.html).
