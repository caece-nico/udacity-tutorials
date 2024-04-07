# Udacity - Intro to Spark

1. [Introduccion](#1.-INtroduccion)
2. [big data](#2.-big-data)
3. [Data Wrangling](#3.-data-wrangling)
    - [Wrangling data]()
    - [Functional programming]()
    - [Read and Write out data]()
    - [Spark environment and Spark APIs]()
    - [RDD API]()
4. [Debuging and Optimization](#4.-debuging-and-optimization)
5. [Machine Learning with PySpark](#5.-machine-learning-with-pyspark)
6. [Project overview](#.6-project-overview)


## 1. Introduccion

Analizar datos a gran escala es muy dificil. Para solucionar esto se usa Spark con capacidad de computación distribuida.

Aparte de Spark hay otras tecnologias que se usan para el proecsamiento de grandes cantidades de datos.

1. [Cassandra Link](https://prestodb.io/)
2. [Presto Link](https://cassandra.apache.org/_/index.html)

### 1.1 Que es Big Data?

```
No hay una única definicion.
Hablamos de Big Data cuando para procesar datos necesitamos procesamiento distribuido entre varias computadoras. 
```

__Conceptos de Hardware: Numbers Everyone should Know__

|Componente|tiempos|
|----------|-------|
|CPU|0.4 ns|
|Storage|16 ns|
|Memory|100 ns|
|Network|150 ms|

# __Completar los ejercicios__

¿QUe pasa sin necesitamos procesar un archivo que no entra en la memoria?

__Pandas puede procesar los datos en _chunks_ haciendo que se procesen pequeñas cantidades del archivo por vez__

[Ejemplo de pandas chunks](https://pandas.pydata.org/pandas-docs/stable/user_guide/io.html#io-chunking)

## 1.2 Ecosistema Hadoop

1. HDFS.
2. Hadoop MapReduce
3. YARN - REsource Manager
4. Hadoop Utilities

Una forma más fácil de ejecutar __mapReduce__ es usando otras tecnologias como __Pig__ y __hive__

A diferencia de Hadoop, Spark no tiene que escribir a disco resultados intermedios haciendo que el procesamiento en memoria sea mas rápido.
Otra diferencia es que Hadoop obtiene los datos de un HDFS pero Spark puede leerlos de un S3 u otros.

Spark tambien tiene una libreria para procesar datos en Streamining menos popular que __Apache Storm__ o __Apache Flink__

## 1.3 ¿Que es mapReduce?

En mapREduce, una texnica para dividir DataSets y distribuirlos antre varios clusters, tenemos tres pasos bien definidos.

Cuando tenemos un File en un HDFS Hadoop lo particiona en __chunks__ o __partitions__

__map__ Cada particion se procesa en cada map. Podemos tener mas de un map. Y los rsultados posteriores de este procesamiento se almacena en el formato (Key, Val).

__shuffle__ Los resultados del map son intercambiados entre varios nodos para ejecutar las agregaciones y obtener los resultados correctos.

__reduce__ Es el último donde se agregan los resultados de los pasos anteriores.

## 1.4 Spark Cluster

Spark está compuesto por un Cluster Master que controla a los nodos workers.

Spark tiene el modo __local__ que se da cuando tenemos una unica PC para hacer pruebas.
Otro modo es __Distributed__ o __cluster manager__.

## 1.5 Casos de uso de Spark

Unos de los casos de usos mas comunes es ETL, tambien se usa para entrenal modelos de ML, este ultimo caso es particularmente importante porque Spark mantiene los datos en memoria haciendo que el procesamiento y entrenamiento sea mas rápido.

- [Data analytics](https://spark.apache.org/sql/)
- [Machile Learning](https://spark.apache.org/mllib/)
- [Streaming](https://spark.apache.org/streaming/)
- [Graph Analytics](https://spark.apache.org/graphx/)

## 1.6 Lkmitaciones de Spark

__Streaming__ Spark tiene una latencia de 500 milisegundos ya uqe opera por micro-batches. Hay otras soluciones de Streaming nativas como Storm y Flink para desarrollar aplicaciones de baja latencia. 

__ML__ otra limitacion son los algoritmos de ML ya que solo soporta algoritmos que escalan de forma lineal. En general no acepta algoritmos de DL. 

__Modelos de SQL__ SI la data que queremos extraer ya está en un motor de SQL podemos usar directamente SQL contra el motor y evitar usar SPARK.
Aveces es mejor usar la simpleza de SQL antes que un modelo de Big Data de Spark. Por ejemplo Cassandra o Presto, ya que Spark no es un sistema de almacenamiento.

__Hadoop vs Spark__ SI bien es cierto que Spark puede ser mas rápido que Hadoop, hay muchos sistemas Legacy que se construyeron alrededor de Hadoop haciendo que su migración sea costosa. Otro punto a tener en cuenta es la cantidad de datos con la que se trabaja.

## 2. Big Data

## 3. Data Wrangling

Usamos Python API o PySpark.

Lo mas dificil de aprender Saprk es dominar la programación funcional que es bastante diferente de la programacion procedural.

En la __programacion funcional__ seguimos el esquema map -> shuffle -> reduce

Este tipo de programacion se ajusta bien a las necesidades de los sistemas distribuidos. 

por ejemplo: tenemos una lista de canciones reproducidads y queres saber la cantidad de veces que se escucharon.
Debemos convertir esa lista en tuplas (key,val) -> luego agrupar por key y finalmente optener la cantidad.


|cancion|
|-------|
|c1|
|c2|
|c1|
|c1|

- quedaria

|tupla|
|-----|
|(c1,1)|
|(c2,1)|
|(c1,1)|
|(c1,1)|

- shuffle

|tupla|
|-----|
|(c1,[1,1,1])|
|(c2, [1])|

- reduce 

|key|valor|
|---|-----|
|c1|3|
|c2|1|

__Mientras que en la programacion procedural hariamos__

```python
cta = {}
for i in range(1, lista_canciones):
    if lista_canciones[i] in cta:
        cta[lista_canciones[i]] += 1
    else:
        cta[lista_canciones[i]] = 0
```

UN posible problema con la programación procedural en entornos distrbuidos es cuando usamos variables globales. Si un nodo se cae y volvemos a ejecutar este codigo, el resultado va a cambiar. __Por este motivo la programacion funcional es la indicada para la computación en paralelo__

```python
cta = {}
for i in range(1, lista_canciones):
    global cta
    if lista_canciones[i] in cta:
        cta[lista_canciones[i]] += 1
    else:
        cta[lista_canciones[i]] = 0
```

__DAGs__ Es la base del modelo __lazzy avaluation__ de Spark. Lo que hace es no transformar los datos o aplicar acciones sobre el DF hasta que sea necesario.
LOs DataFRames de Spark son Inmutables y la forma en que Spark trabaja es uniendo funciones.

## Maps and Lambda functions.

Usamos map para generar un salida a un DataFrame aplicando una transformacion sobre los datos. 
Hay dos formas de hacerlo.
1. Podemos definir y usar funciones.
2. O podemos usar funciones anonimas __lambda__.

```python
from pyspark import SparkContext

sc = SparkContext(appName = "EjemploMap")

lista_nombres = ['nicoLas', 'silVina', 'Martin', 'patricio', 'alEjandro']

rdd = sc.parallelize(lista_nombres)

def mayusculas(nombre):
    return nombre.capitalize()

rdd.map(mayusculas).collect()

rdd.map(lambda nombre: nombre.capitalize()).collect()
```

[Para leer sobre funciones Lambda](https://palmstroem.blogspot.com/2012/05/lambda-calculus-for-absolute-dummies.html)

Ambos ejemplos son identicos.

### Read and Write out data

Antes de poder empezar con __Data Wrangling__ hay que importar datos.
Los formatos mas importantes son :
- .csv
- .json
- .parquet

1. __distributed Data Stores__

Para poder procesar grandes cantidades de datos es necesario tambien almacenar os datos en medios __distribuidos__ 
Hadoop usa __HDFS__ la data se divide en folds de 128MB cada uno. Otro almacenamiento puede ser __AWS S3__

- [Leer mas HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html)
- [Leer mas S3](https://aws.amazon.com/s3/)

2. __SparkSession__

El __sparkContext__ es el punto de entrada de cualquier programa Spark. 
Conecta el Cluster con la Aplicacion.

```python
from pyspark import SparkConf, SparkContext

conf = SparkConf().setAppName("name").setMaster("local o IP Address")
sc = SparkContext(conf = conf)

```

__otra forma de iniciar Spark__

```python
from pyspark.sql import SparkSession

sc = SparkSession.builder.\
    appName("app name").\
        config("param", "value").\
            getOrCreate()
```

Este último método se usa para trabajar con DataFrames que ofrece un nivel de abstracción mayor a los __RDD__

3. __Leer un archivo__

```python
spark.sparkContext.getCOnf().getAll()

path = "hdfs://"

df = sc.read.json(path)

df.printSchema()

df.describe()

df.show(n=1)

df.take(5)

df.write.csv(path)

```

### Spark environment and Spark APIs

Hay dos formas de hacer __wrangling__ con Spark. Una es usando __programacion imperativa__ (DF) y la otra __programacion declarativa__ (SQL).


 
## 4. Debuging and Optimization

## 5. Machine Learning with pySpark


## 6. Project Overview

