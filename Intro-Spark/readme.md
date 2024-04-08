# Udacity - Intro to Spark

1. [Introduccion](#1.-INtroduccion)
2. [big data](#2.-big-data)
3. [Data Wrangling](#3.-data-wrangling)
    - [Wrangling data]()
    - [Functional programming]()
    - [Read and Write out data]()
    - [Spark environment and Spark APIs]()
    - [RDD API]()
3. [Spark cluster on AWS](#3.-spark-cluster-on-aws)
    - [Crear cluster usando el Manager]()
    - [Crear cluster usando CLI]()
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


## 3. Spark cluster on AWS

### 31. Crear cluster usando AWS Manager.

1. From Local to Standalone Model

__MESOS__ y __YARN__ son modos usados para trabajar con varios equipos de personas.
__STANSALONE__ es el otro modo. 

Cuando ejecutamos un JOB de Spark lo que hace es agarrar el File que está en un disco, lo sube a memoria y lo pasa a la CPU apara procesarlo. Si estamos trabajando en modo local eso lo hace es un unico equipo, pero si estamos en un entorno distribuido lo normal seria tener un equipo de storage, otros de procesamiento y otros de memoria.

![](./img/aws-arq-01.png)

En esta arquitectura tenemos el Storage __S3__ y un conjunto de clusters en __EC2__ que será ejecutado cuando desde nuestro entorno corramos un __JOB DE SPARK__.

- Porque necesitamos HDSF?

HDFS es el file system de Hadoop, usa mapReduce como resource manager para distribuir los archivos a travez de los discos de los clusters. Spark no tiene un FileSystem, sino que levanta los archivos a memoria y los procesa.
Una forma de optimizar esto es instalar Spark sobre el HDFS de Hadoop.

- Spark y EMR Cluster.

Debido a que Spark cluster usa varias maquinas, para poder usar Spark en cada una debemos instalar todas sus dependencias. Esto es un trabajo manual. Elastic Map Reduce es un servicio de AWS lo hace automatico.


Esto ultimo es mas trabajoso, vamos por EMR

2. __Creacion de un Cluster EMR en AWS__

EMR es un servicio que nos da EC2 Instancias con tecnologias de big data, nos crea el cluster de forma automatica.

- [Link setting up EMR cluster](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-gs-launch-sample-cluster.html)

2.1 Primero creamos un par de claves __SSH__

En EC2 elegimos la opcion __pares de claves__ y creamos una con un nombre y tio de archivo __.pem__

![](./img/aws-key-pair-01.png)

Cuanod la creamos esto nos descarga un archivo __.pem__ que debemos guardar pero __no compartir__

![](./img/aws-key-pair-02.png)



Otra forma de crear un cluster es hacerlo de forma manual pero es mas trabajoso ya uqe deberiamos instalar spark en cada servidor y ocuparnos de la configuracion.

- [Crear un cluster en AWS](https://blog.insightdatascience.com/create-a-cluster-of-instances-on-aws-899a9dc5e4d0)

2.2 creacion del cluster.

Con las claves creadas y guardadas buscamos el servicio __AWS EMR__ y creamos un cluster.

![](./img/aws-emr-01.png)

![](./img/aws-emr-02.png)

![](./img/aws-emr-03.png)

![](./img/aws-emr-04.png)

Para poder conectarnos desde SSH a los nodos debemos incorporar la regla de entrada.

![](./img/aws-emr-05.png)

__Resolucion de problemas con Reglas de acceso__

[Authorize inbound traffic](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-connect-ssh-prereqs.html)


### 3.2 Crear Cluster EMR usando AWS CLI.

2.3 Para poder conectarnos a la instancia debemos instalar el __Cliente de AWS CLI__

- [Instalacion de AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

- [Comandos importantes de AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/)

Para saber si ya lo tenemos instalado hacemos:

```shell
which aws

aws --version
```

![](./img/aws-cli-01.png)

__para poder conectarnos a nuestra cuenta debemos vincular la clave que creamos (Long Term Access)


[Tutotrial de AWS CLI](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html)

1. En la consola de AWS vamos a __IAM__ y buscamos __Users__ y creamos un usuario nuevo.

![](./img/aws-user-01.png)

![](./img/aws-user-02.png)

Una vez creado este usuario debemos especificar que es un usuario que se conectará por __AWS CLI__ para esto debemos crear una clave de acceso a la cual accedemos ingresando al usuario creado.

![](./img/aws-user-02-01.png)

Esto nos devuelve un __ACCESS KEY__ y un __SECRET ACCESS KEY__ en un archivo que debemos descargar y guardar pero __nunca compartir__.

2. Configuramos AWS para poder conectarnos usando nuestras credenciales.

```shell
# Navigate to the home directory
cd ~
# If you do not use the profile-name, a default profile will be created for you.
aws configure --profile <profile-name>
# View the current configuration
aws configure list --profile <profile-name>
# View all existing profile names
aws configure list-profiles
# In case, you want to change the region in a given profile
# aws configure set <parameter> <value>  --profile <profile-name>
aws configure set region us-east-1  --profile <profile-name>
```

Para setear el directorio donde tenemos el archivo de configuracion de __aws__ hacemos:

```shell
export AWS_CONFIG_FILE=~/.aws/config
export AWS_SHARED_CREDENTIALS_FILE=~/.aws/credentials
```

__./aws__ se crea solo.

```
Si ya tenemos algun profile/credencial creado lo ponemos eliminar.
```

3. Seteamos las claves del archivo .csv que descargamos usando gitbash

```shell
setx AWS_ACCESS_KEY_ID AKIAIOSFODNN7EXAMPLE
setx AWS_SECRET_ACCESS_KEY wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
setx AWS_DEFAULT_REGION us-west-2
```

4. Para ver si todo funcionó ok hacemos:

```shell
# If you've just one profile set locally
aws iam list-users
# If you've multiple profiles set locally
aws iam list-users --profile <profile-name>
```

![](./img/aws-user-04.png)

deberiamos ver algo así.

```json
{
"Users": [
    {
        "Path": "/",
        "UserName": "Admin",
        "UserId": "AIDAZMXYZ3LY2BNC5ZM5E",
        "Arn": "arn:aws:iam::388752792305:user/Admin",
        "CreateDate": "2021-01-28T13:44:15+00:00"
    }
]
}
```

5. Si nos hace falta podemos modificar algo del archivo de configuracion haciendo:

```shell
aws configure set default.region us-east-2
```

__RESOLUCION DE PROBLEMAS__

1. [Configure the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
2. [Configuration and credential file settings
](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
3. [Environment variables to configure the AWS CLI
](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)

- __Habiendo creado y configurado el usuario CLI podemos crear el cluster usando código__

__completar__


3.2.2 Comprobar que el cluster se creó correctamete.

```shell
"ClusterID": "j-2PZ79NHXO7YYX"
"ClusterArn" "arn:aws:elasticmapreduce:us-east-2:027631528606:cluster/j-2PZ79NHXO7YYX": 
```

```shell
aws emr describe-cluster --cluster-id <CLUSTER_ID FROM ABOVE> --profile <profile-name>
```

Deberiamos ver algo así.

![](./img/aws-cli-emr-01.png)

3.2.3 Conectarnos al cluster usando __SSH__

Una vez habilitados los accesos o reglas de acceso usando SSH para nuestra IP o rango de IPs probamos la conexión.

![](./img/aws-emr-06.png)

Vemos la opcion __Conectarse por SSH__

![](./img/aws-emr-07.png)

Nos muestra la dirección y la ingresamos en GitBash que nos puede pedir un segundo factor de autenticación.

![](./img/aws-emr-08.png)

Nos pudimos conectar al cluster de forma exitosa.

[Ver mas documentacion](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-connect-master-node-ssh.html)

3.2.4 __Como hacer __port fortwarding__ para conectarnos al manager de PSark?__

Para ver el Spark UI del entorno EMR hacemos:

```shell
ssh -i ~/spark-cluster.pem -N -D 8157 hadoop@ec2-44-199-210-117.compute-1.amazonaws.com
```

Vamos que al script usado anteriormente se agrega __-N -D 8157__

Paso siguiente copiamos el archivo __.pem__ al master node usando el comando

```shell
scp -i ~/spark-cluster.pem spark-cluster.pem  hadoop@ec2-44-199-210-117.compute-1.amazonaws.com:/home/hadoop/
```

[Crear un puente con ssh](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-ssh-tunnel.html)

Para poder usarlo desde chrome descargamos  __ProxuSwitchOmega__

![](./img/aws-proxy-01.png)

Creamos un nuevo perfil __emr-socks-proxy__ y seleccionamos __PAC profile type__

![](./img/aws-proxy-02.png)

y copiamos y pegamos el siguiente script.

![](./img/aws-proxy-03.png)

```python
function FindProxyForURL(url, host) {
    if (shExpMatch(url, "*ec2*.amazonaws.com*")) return 'SOCKS5 localhost:8157';
    if (shExpMatch(url, "*ec2*.compute*")) return 'SOCKS5 localhost:8157';
    if (shExpMatch(url, "http://10.*")) return 'SOCKS5 localhost:8157';
    if (shExpMatch(url, "*10*.compute*")) return 'SOCKS5 localhost:8157';
    if (shExpMatch(url, "*10*.amazonaws.com*")) return 'SOCKS5 localhost:8157';
    if (shExpMatch(url, "*.compute.internal*")) return 'SOCKS5 localhost:8157';
    if (shExpMatch(url, "*ec2.internal*")) return 'SOCKS5 localhost:8157';
    return 'DIRECT';
}
```

Por ultimo habilitamos el proxy creado.

![](./img/aws-proxy-04.png)

Para poder ingresar al Spark UI hay que habilitar las reglas de __inboud__ para Https y Https

![](./img/aws-proxy-05.png)

Ahora podemos ingresar en:

```shell
http://ec2-44-199-210-117.compute-1.amazonaws.com:18080/
```

[](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-connect-master-node-proxy.html)

3.2.5 Subir un script de python a __EMR__.

En la consola de __AWS EMR__ seleccionamos __Notebboks__ o __cuadrenos__



## 4. Debuging and Optimization

## 5. Machine Learning with pySpark


## 6. Project Overview

