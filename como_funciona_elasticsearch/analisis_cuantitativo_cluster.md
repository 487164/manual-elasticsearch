# Análisis cuantitativo para estimar el rendimiento de un cluster
Se describen a continuación una serie de pautas que se pueden seguir para realizar un análisis previo a la creación de un conjunto de índices con unos datos estructurados de una manera similar. Se toman las ideas de esta [charla](https://www.elastic.co/elasticon/conf/2016/sf/quantitative-cluster-sizing). Aunque hay algunas de estas pautas que solo se deben de tener en cuenta si vamos a realizar un cluster multi-nodo, es sano el seguirlas desde un inicio para tener la posibilidad de hacer un escalado horizontal sin demasiados problemas.

## Conocimientos previos al análisis

Para realizar un análisis completo de la viabilidad de un conjunto de índices y los recursos que estos consumirán, necesitamos tener constancia de los siguientes aspectos:
* Estructura de los datos que vamos a indexar:
	* Tipos de datos.
	* Tamaño medio de los documentos a indexar.
	* Tiempo de retención necesario en el cluster.
* Características del entorno donde vamos a desplegar el cluster:
	* Cantidad de memoria, CPU, RAM y sus ratios (ratio RAM/almacenamiento, ratio RAM/nodo, ratio almacenamiento/nodo)
	* Calidad de la misma (SSD, HDD, ...)
* Conocimiento del tipo de queries que vamos a realizar en los dashboards:
	* Periodicidad. Detectar queries que pueden ser frecuentes y cuales más esporádicas.
	* Complejidad. Detectar queries con acceso a diferentes índices.
* Requerimientos de rendimiento:
	* Máximo tiempo de indexado.
	* Máximo tiempo de respuesta de queries.

## Análisis cuantitativo de rendimiento
Con los pasos incrementales que se definen a continuación, lo que se pretende es estimar cómo se comportará el cluster para poder configurarlo correctamente desde un comienzo. Estos pasos se deben realizar en un entorno de pruebas.

### 1. Uso de disco
**Objetivo:** Conocer cuánto almacenamiento en Elastic vamos a utilizar por documento y optimizar esto mediante mappings.

**Pasos:**
1. Usar un cluster de un solo nodo con un solo shard primario para un índice.
2. Indexar una cantidad *decente* de datos (alrededor de 1GB o 10M de documentos).
3. Calcular la diferencia de almacenamiento real y en Elastic. Utilizar *_forcemerge* para visualizar diferencias después de los merges.
4. Probar diferentes configuraciones de mappings (\_all activado o desactivado, estructurar más los documento usango tipos prefijados, probar a indexar solo un porcentaje de los datos, ...).
5. Comparar las configuraciones y seleccionar la más optima.

### 2. Dimensión de los shards
**Objetivo:** Conocer un punto óptimo de tamaño de shards para satisfacer los requerimientos (tiempo de indexado y de respuesta de queries).

**Pasos:**
1. Usar un cluster de un solo nodo con un solo shard primario para un índice.
2. Indexar una cantidad *realista* de datos (algo como 1TB o lo que estimemos que va a tener un único índice de nuestro grupo).
3. Graficar el tiempo de indexado y el tiempo de respuesta de queries frente al tamaño del shard y encontrar el punto donde excede los máximos establecidos.

### 3. Benchmark para un único nodo
**Objetivo:** Conocer el punto de saturación para un único nodo.

**Pasos:**
1. Usar un cluster de un solo nodo con dos shards primarios para un índice.
2. Indexar una cantidad *realista* de datos (algo como 1TB o lo que estimemos que va a tener un único índice de nuestro grupo).
3. Graficar el tiempo de indexado y el tiempo de respuesta de queries frente al tamaño del shard para ver el punto de saturación respecto a los requisitos.
4. Ir aumentando el número de shards para averiguar cuando comienza a ser contraproducente añadir más.

### 4. Benchmark para cluster multi-nodo
**Objetivo:** Conocer el punto de saturación para varios nodos.

**Pasos:**
1. Usar un pequeño cluster representativo, escalado hacia abajo de lo que será nuestro sistema final. Por ejemplo, dos nodos, uno primario y uno de réplica.
2. Añadir una cantidad *realista* de datos. Escalamos hacia abajo del total de datos que estimamos tendrá nuestro conjunto de índices y asumimos que se distribuirán equitativamente.
3. Ejecutar benchmark realistas:
	1. La velocidad de indexado tiene que ser escalada hacia abajo, pues en un entorno con más nodos podremos indexar más. Si Y es la velocidad que necesitamos finalmente, usaremos Y\*nodos_prueba/nodos_total.
	2. Las queries no necesitan ser escaladas, pues se accederá a la misma cantidad de datos por nodo. Es muy útil testear con queries que accedan a varios nodos.
4. Medir el uso de recursos (RAM, almacenamiento, throughput, número de eventos) y sacar conclusiones.

### Más aspectos a tener en cuenta
* Si se van a utilizar [*data tiers*](https://www.elastic.co/guide/en/elasticsearch/reference/current/data-tiers.html) (hot-cold clusters) para gestionar datos más recientes respecto de los más antiguos.
* Si realmente necesitamos réplicas o con un análisis para solo un nodo es suficiente.
* Cuánto de importante es la tolerancia a fallos.
* Continuar la monitorización mediante [Theseus](https://repo1.naudit.es/theseus/theseus-master) tras el despliegue. Ver [Monitorización de Elasticsearch](monitorizacion_de_elasticsearch.html).
