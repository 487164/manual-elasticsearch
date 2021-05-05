# Java Virtual Machine
## ¿Qué es una Java Virtual Machine?
Una *Java Virtual Machine* no es más que una máquina virtual que es capaz de ejecutar código binario especial de Java. En general, se utiliza como puente entre el entrono de ejecución y el sistema físico que realizará la ejecución.

El mayor motivo por el que se utilizan las JVMs para aplicaciones en Java es por la portabilidad, permitiendo a estas ejecutarse en cualquier sistema que posea una JVM.

## ¿Por qué ElasticSearch utiliza JVMs?
ElasticSearch utiliza JVMs puesto que sus aplicaciones están escritas en Java. [Lucene](como_funciona_elasticsearch/lucene.md) es una librería de búsqueda en Java y es la base de Elastic. Es por esto que es necesario conocer las carácteristicas y necesidades de nuestro proyecto para definir las especificaciones de la JVM donde se ejecutará.

## Conceptos importantes acerca de las JVMs
Algunos de los aspectos a tener en cuenta acerca de las JVMs y que nos servirán durante el manejo de Elastic son como maneja la memoria:
* Heap de memoria: Una JVM tiene asignada una memoria máxima que toma del sistema que la contiene. Esta manejara esta memoria interiormente y se la ira asignando a los diferentes procesos y datos que ocurran. Es importante definirla acorde a las necesidades del proyecto. Puedes obtener más información en la sección de [configuracion de ElasticSearch](como_funciona_elasticsearch/configuracion_de_elasticsearch.md)
* Garbage Collection: Es un proceso periódico que ejecuta automáticamente la JVM para liberar memoria que ya no está en uso y que puede ser usada por nuevos procesos que se ejecuten. Se debe de controlar que este proceso se haga correctamente y que no se llegue al límite de memoria. Hay más información en la sección de [monitorización de ElasticSearch](como_funciona_elasticsearch/monitorizacion_de_elasticsearch.md)
