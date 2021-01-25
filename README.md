# Computación en la Nube - Tarea 1

El proposito de este reporte es el de aplicar las tecnologías y procedimientos aprendidos en la materia de Computación en la Nube, por medio de la ejecución de las siguientes tareas:

1. Construir de una ETL que permita leer los datos del sistema de archivos. **covid19-open-data** y lmacenarlos en un sistema de almacenamiento propio (BlobStorage/s3).
1. Leer los datos extraidos en el sistema de almacenamiento, usando tecnologías como Databricks o Aws Athena.
1. Ejecutar una serie de consutlas SQL sobre los datos leídos.
1. Conectar el aplicativo Power BI al sistema donde se ejecutaron las consultas, para permitir su visualización a través de los diferentes herramientas que provee este sistema de reportes.

## 1. Creación de la ETL

Antes de crear la ETL para consumir los datos de la fuente covid19-open-data, debemos crear un contenedor de archivos para almacenar la información cuando ejecutemos el proceso de extracción. Para esta parte, se optó por utilizar un bucket de S3 en Aws, que permita almacenar la data.

### Creación del Bucket en Aws S3

Ingresaremos a la consola de Aws y buscaremos S3 en la barra de busqueda de la página (parte superior). Luego haremos clic en el primer servicio que se muestra en la lista.

![Opción S3 Aws](./img/img_0.png)

Una vez ingresemos al portal de S3, vamos a hacer clic en el botón **Create Bucket** que aparece en la parte derecha de la página.

![Opción S3 Aws](./img/img_1.png)

Este botón nos llevará a un formulario. En el campo **Bucket name** ingresaremos el nombre que deseemos para nuestro bucket y finalizaremos la creación presionando el botón **Create Bucket** que se muestra al final del formulario.

![Opción S3 Aws](./img/img_2.png)

![Opción S3 Aws](./img/img_3.png)

Una vez creado el bucket, este nos debería aparecer en la lista de buckets existentes.

![Opción S3 Aws](./img/img_4.png)

### Creación de la ETL con Databricks

Nos logearemos en la versión community de Databricks e iremos a la opción **Clusters** que se encuentra en la parte izquierda. Allí haremos clic en el botón **Create Cluster**.

![Opción S3 Aws](./img/img_5.png)

Se nos mostrará un formulario para la creación del cluster. En el campo **Cluster Name** ingresaremos el nombre que deseemos para nuestro cluster. Las demás opciones no las modificaremos. Finalizaremos este proceso presionando el botón **Create Cluster**.

![Opción S3 Aws](./img/img_6.png)

Una vez finalizada la creación, el cluster nos debería aparecer en la lista mostrada a continuación.

![Opción S3 Aws](./img/img_6_1.png)

El siguiente paso es crear un Notebook para trabajar nuestra ETL. Para ello, presionaremos en el logo de Databricks para acceder a la página inicial y presionaremos la opción **New Notebook**.

![Opción S3 Aws](./img/img_7.png)

En el formulario ingresaremos el nombre del Notebook, el lenguaje de programación y el cluster que hemos creado en los pasos anteriores. Finalizamos la creación haciendo clic en el botón **Create**.

Una vez creado el Notebook, vamos a ejecutar los siguientes comandos para crear la ETL.

Para leer el dataset **covid19-open-data**, ejecutaremos el siguiente código para descargar el archivo del siguiente [link](https://storage.googleapis.com/covid19-open-data/v2/main.csv):

```
url = "https://storage.googleapis.com/covid19-open-data/v2/main.csv"

from pyspark import SparkFiles
spark.sparkContext.addFile(url)

df = spark.read.csv("file://"+SparkFiles.get("main.csv"), header=True, inferSchema= True)
```

Una vez leído el archivo y cargado en un dataframe de Spark, crearemos una unidad de montaje a nuestro bucket de S3, que no es más que una referencia a nuestro sistema de almacenamiento:

```
access_key = "XXXXXXXXXXXXXXX"
secret_key = "XXXXXXXXXXXXX"
encoded_secret_key = secret_key.replace("/", "%2F")
aws_bucket_name = "<bucket_name>"
mount_name = "<folder_name>"

dbutils.fs.mount(source = "s3a://%s:%s@%s" % (access_key, encoded_secret_key, aws_bucket_name), mount_point = "/mnt/%s" % mount_name)
```

Ahora ejecutaremos el siguiente código para guardar el dataframe en nuestro bucket:

```
out = "/mnt/<folder_name>/<s3_folder>"
df.write.option("header","true").csv(out)
```

Una vez se ha ejecutado todo el código del Notebook, nuestro bucket de S3 debería contener una carpeta con todos los archivos que Spark ha creado para el dataset.

![Opción S3 Aws](./img/img_13.png)

El Notebook creado puede accederse a través del siguiente [link](https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/1368387989707308/2228990396878086/6696879192669646/latest.html).

#### Obtener keys de acceso a Aws

La creación de la unidad de montaje requiere de un **access key** y **secret key** para acceder al bucket de S3. Estos valores se pueden obtener ingresando a la consola de Aws, luego vamos a nuestro usuario en la parte superior derecha y seleccionamos la opción **My Security Credentiales**.

![Opción S3 Aws](./img/img_9.png)

Una vez en la página vamos a la sección **Access Key** y generamos un par de claves.

![Opción S3 Aws](./img/img_10.png)

### Reproducir consultas SQL con Databriks

Ahora vamos a crear un nuevo Notebook en Databricks.

Nuevamente vamos a crear una unidad de montaje para crear una referencia a nuestro bucket de S3. Esto lo podemos hacer con el siguiente código:

```
access_key = "XXXXXXXXXXXXXXX"
secret_key = "XXXXXXXXXXXXX"
encoded_secret_key = secret_key.replace("/", "%2F")
aws_bucket_name = "<bucket_name>"
mount_name = "<folder_name>"

dbutils.fs.mount(source = "s3a://%s:%s@%s" % (access_key, encoded_secret_key, aws_bucket_name), mount_point = "/mnt/%s" % mount_name)
```

Una vez creada la unidad de montaje, vamos a leer los datos particionados que se almacenaron en el bucket de S3, con los siguientes comandos:

```
path = "/mnt/<folder_name>/<s3_folder>/"
df = spark.read.format("csv").option("header", "true").load("%s/*" % path)
```

Ahora vamos a crear una vista temporal de nuestro dataframe usando el siguiente comando:

```
df.createOrReplaceTempView("<view_name>")
```
