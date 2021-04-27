# Lucene
## ¿Qué es Lucene?
[Apache Lucene](http://lucene.apache.org/) es un motor de búsqueda de texto completo de alto rendimiento. Está escrito en Java (aunque también tiene una extensión en Python con [PyLucene](https://lucene.apache.org/pylucene/)) y es open-source.

Lucene trabaja con **índices** y **documentos**. Los índices están formados por conjuntos de documentos que pueden estar más o menos relacionados. Los documentos son objetos que contienen una serie de **campos** (*field*) de diferentes tipos según almacenen texto, fechas o números.

El principal objetivo de esta organización de los datos es el de, tras invertir tiempo en su indexado estructurado, agilizar enormemente las búsquedas. De esta manera se permite un acceso rápido y dinámico a una gran cantidad de datos.

## ¿Para qué usa ElasticSearch Lucene?

Elastic está basado en Lucene y amplía su alcance y funcionalidades para crear un sistema completo de almacenamiento y búsqueda de datos. Por este motivo, todas las características sobre Lucene que se explican en este apartado son aplicables a los datos gestionados con Elasticsearch.


## Indexado de datos
El aspecto más característico de Lucene y donde alberga su mayor potencial, es durante el indexado de datos, en especial cuando se trata de texto. Hay dos aspectos que merece la pena comprender: los **índices invertidos** y los **analizadores**.

### Índices invertidos
La manera de almacenar el texto en Lucene es mediante índices invertidos. Esto es, se almacena cada palabra *tokenizada* por separado junto con una lista de los documentos en los que aparece. Esto es muy sencillo de visualizar con un ejemplo:

Si quisieramos almacenar estos documentos

| Doc. ID | Texto |
|------------|---------|
| 0 | hola mundo |
| 1 | elastic mola |
| 2 | naudit mola |
| 3 | en naudit usamos elastic |
| 4 | todo el mundo usa elastic  |

En Lucene quedarían organizados de la siguiente manera:

| Token | Frecuencia | Doc. Id |
|-------|------------|---------|
| hola | 1 | 0 |
| mundo | 2 | 0, 4 |
| elastic | 3 | 1, 3, 4 |
| mola | 2 | 1, 2 |
| naudit | 2 | 2, 3 |
| en | 1 | 3 |
| usamos | 1 | 3 |
| todo | 1 | 4 |
| el | 1 | 4 |
| usa | 1 | 4 |


### Analizadores

En el ejemplo anterior podemos ver cómo hay claramente algunas parabras que son menos significativas o que apenas aportan información al texto que estamos almacenando (en, el) y palabras similares almacenadas como diferentes (usa, usamos). Es en este punto donde entran en juego los analizadores, encargados de extraer la información mas valiosa del texto para facilitar su posterior búsqueda.

Hay una gran variedad de analizadores que se pueden concatenar unos detrás de otros dependiendo de las necesidades específicas, e incluso se pueden definir personalizados. Los más usuales son los siguientes:
* StopWords. Elimina las palabras no significativas como: a, o , en, el, la, etc...
* Lowercase. Convierte todas las palabras a minúscula.
* RootWord. Útil para almacenar palabras con diferentes terminaciones como la misma. Por ejemplo, usamos, usa, usaré, etc... como us.

Cabe destacar que algunos de los analizadores son dependientes del idioma.

## Tipos de datos
Lucene está muy orientado al análisis y búsqueda de texto, pero también admite otros tipos de datos. Aunque para estos no se pase por el proceso de tokenización y normalización, la búsqueda y agregación siguen siendo de gran velocidad. Es de esta mezcla de búsqueda completa de texto y distintos tipos de datos de lo que más se beneficia Elasticsearch.

Los principales tipos de datos usados por Lucene son:
* Integer.
* Long. Es usado para fechas en Elasticsearch
* Double.
* Binary.
* Text. Hay que diferenciar dos tipos:
	* Keywords (las String normales). No se analiza el texto y, a la hora de realizar búsquedas, hay que hacer un *match* exacto del contenido del campo.
	* Text. Se analiza el texto al indexar y se pueden realizar búsquedas de texto completas en estos campos.

## Búsqueda de datos (queries)
La sintaxis de *queries* de Lucene está basada en JSON y permite una gran variedad de búsquedas y agregaciones por campos exactos o rangos de datos y con la posibilidad de utilizar expresiones regulares.

Dado que Elasticsearch está basado en Lucene, hereda toda su sintaxis. Para realizar queries de una manera efectiva se requiere un análisis en mayor profundidad que se puede ver en la [sección correspondiente](../conceptos_basicos/queries.html).
