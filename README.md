# Computación en la Nube - Tarea 1

El proposito de este reporte es el de aplicar las tecnologías y procedimientos aprendidos en la materia de Computación en la Nube, por medio de la ejecución de las siguientes tareas:

1. Construir de una ETL que permita leer los datos del sistema de archivos. **covid19-open-data** y lmacenarlos en un sistema de almacenamiento propio (BlobStorage/s3).
1. Leer los datos extraidos en el sistema de almacenamiento, usando tecnologías como Databriks o Aws Athena.
1. Ejecutar una serie de consutlas SQL sobre los datos leídos.
1. Conectar el aplicativo Power BI al sistema donde se ejecutaron las consultas, para permitir su visualización a través de los diferentes herramientas que provee este sistema de reportes.

## 1. Creación de la ETL

Antes de crear la ETL para consumir los datos de la fuente covid19-open-data, debemos crear un contenedor de archivos para almacenar la información cuando ejecutemos el proceso de extracción. Para esta parte, se optó por utilizar un bucket de S3 en Aws, que permita almacenar la data.

### Creación Bucket S3

Para la creación del bucket vamos ingresaremos a la consola de Aws e ingresaremos en la barra de busqueda (parte superior) el nombre S3. Haremos clic en el primer servicio que se muestra en la lista.

![Opción S3 Aws](./img/img_0.png)

Una vez ingresemos al portal de S3, vamos a hacer clic en el botón **Create Bucket** que aparece en la parte derecha de la página.

![Opción S3 Aws](./img/img_1.png)

Este botón nos llevará a un formulario. En el campo **Bucket name** ingresaremos el nombre que deseemos para nuestro bucket y finalizaremos la creación presionando el botón **Create Bucket** que se muestra al final del formulario.

![Opción S3 Aws](./img/img_2.png)

![Opción S3 Aws](./img/img_3.png)

Una vez creado el bucket, este nos debería aparecer en la lista de buckets existentes.

![Opción S3 Aws](./img/img_4.png)

3. Link del dataset: https://storage.googleapis.com/covid19-open-data/v2/main.csv
