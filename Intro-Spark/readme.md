# Udacity - Intro to Spark

1. [Introduccion](#1.-INtroduccion)
2. [big data](#2.-big-data)
3. [Data Wrangling](#3.-data-wrangling)
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

[Data analytics](https://spark.apache.org/sql/)
[Machile Learning](https://spark.apache.org/mllib/)
[Streaming](https://spark.apache.org/streaming/)
[Graph Analytics](https://spark.apache.org/graphx/)

## 1.6 Lkmitaciones de Spark

__Streaming__ Spark tiene una latencia de 500 milisegundos ya uqe opera por micro-batches. Hay otras soluciones de Streaming nativas como Storm y Flink para desarrollar aplicaciones de baja latencia. 

__ML__ otra limitacion son los algoritmos de ML ya que solo soporta algoritmos que escalan de forma lineal. En general no acepta algoritmos de DL. 

__Modelos de SQL__ SI la data que queremos extraer ya está en un motor de SQL podemos usar directamente SQL contra el motor y evitar usar SPARK.
Aveces es mejor usar la simpleza de SQL antes que un modelo de Big Data de Spark. Por ejemplo Cassandra o Presto, ya que Spark no es un sistema de almacenamiento.

__Hadoop vs Spark__ SI bien es cierto que Spark puede ser mas rápido que Hadoop, hay muchos sistemas Legacy que se construyeron alrededor de Hadoop haciendo que su migración sea costosa. Otro punto a tener en cuenta es la cantidad de datos con la que se trabaja.

## 2. Big Data

## 3. Data Wrangling

## 4. Debuging and Optimization

## 5. Machine Learning with pySpark


## 6. Project Overview

