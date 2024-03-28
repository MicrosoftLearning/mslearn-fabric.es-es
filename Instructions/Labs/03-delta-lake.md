---
lab:
  title: Uso de tablas Delta en Apache Spark
  module: Work with Delta Lake tables in Microsoft Fabric
---

# Uso de tablas Delta en Apache Spark

Las tablas de un almacén de lago de Microsoft Fabric se basan en el formato *Delta Lake* de código abierto de Apache Spark. Delta Lake agrega compatibilidad con la semántica relacional para las operaciones de datos por lotes y de streaming, y permite la creación de una arquitectura de almacén de lago en la que se puede usar Apache Spark para procesar y consultar datos en tablas basadas en archivos subyacentes de un lago de datos.

Este ejercicio debería tardar en completarse **40** minutos aproximadamente.

> **Nota**: Necesitarás una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. En la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com) en `https://app.fabric.microsoft.com`, seleccione **Ingeniería de datos de Synapse**.
2. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
3. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
4. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

## Creación de un almacén de lago y carga de datos

Ahora que tiene un área de trabajo, es el momento de crear un almacén de lago de datos para los datos que va analizar.

1. En la página principal de **Ingeniería de datos de Synapse**, cree un nuevo **almacén de lago** con el nombre que prefiera.

    Al cabo de un minuto más o menos, se creará un nuevo almacén de lago vacío. Debe ingerir algunos datos en el almacén de lago de datos para su análisis. Hay varias maneras de hacerlo, pero en este ejercicio simplemente descargará un archivo de texto en el equipo local (o máquina virtual de laboratorio, si procede) y, luego, lo cargará en el almacén de lago.

1. Descargue el [archivo de datos](https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv) de este ejercicio desde `https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv` y guárdelo como **products.csv** en el equipo local (o máquina virtual de laboratorio, si procede).

1. Vuelva a la pestaña del explorador web que contiene el almacén de lago y, en el menú **...** de la carpeta **Archivos** del panel **Explorador**, seleccione **Nueva subcarpeta** y cree una carpeta llamada **products**.

1. En el menú **...** de la carpeta **products**, seleccione **Cargar** y **Cargar archivos** y, luego, cargue el archivo **products.csv** del equipo local (o la máquina virtual de laboratorio, si procede) en el almacén de lago.
1. Una vez cargado el archivo, seleccione la carpeta **products** y compruebe que se ha cargado el archivo **products.csv**, como se muestra aquí:

    ![Captura de pantalla del archivo products.csv cargado en un almacén de lago.](./Images/products-file.png)

## Exploración de datos en un objeto DataFrame

1. En la página **Inicio**, mientras ve el contenido de la carpeta **products** en el lago de datos, vaya al menú **Abrir cuaderno** y seleccione **Nuevo cuaderno**.

    Al cabo de unos segundos, se abrirá un nuevo cuaderno que contiene una sola *celda*. Los cuadernos se componen de una o varias celdas que pueden contener *código* o *Markdown* (texto con formato).

2. Seleccione la celda existente en el cuaderno, que contiene código sencillo y, luego, use su icono **&#128465;** (*Eliminar*) en su parte superior derecha para quitarla, ya que no la necesitará.
3. En el panel **Explorador de almacenes de lago** de la izquierda, expanda **Archivos** y seleccione **products**; aparece un nuevo panel que muestra el archivo **products.csv** que cargó anteriormente:

    ![Captura de pantalla de un cuaderno con un panel de archivos.](./Images/notebook-products.png)

4. En el menú **...** de **products.csv**, seleccione **Cargar datos** > **Spark**. Se agregará al cuaderno una nueva celda de código que contiene el código siguiente:

    ```python
   df = spark.read.format("csv").option("header","true").load("Files/products/products.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/products/products.csv".
   display(df)
    ```

    > **Sugerencia**: Puede ocultar el panel que contiene los archivos de la izquierda usando su icono **<<** . De esta forma, podrá centrarse en el cuaderno.

5. Use el botón **&#9655;** (*Ejecutar celda*) a la izquierda de la celda para ejecutarla.

    > **Nota**: Dado que esta es la primera vez que se ejecuta código de Spark en este cuaderno, se debe iniciar una sesión con Spark. Esto significa que la primera ejecución puede tardar un minuto en completarse. Las ejecuciones posteriores serán más rápidas.

6. Cuando se haya completado el comando de la celda, revise la salida que aparece debajo de ella, que será algo parecido a esto:

    | Índice | ProductID | ProductName | Category | ListPrice |
    | -- | -- | -- | -- | -- |
    | 1 | 771 | Mountain-100 Silver, 38 | Bicicletas de montaña | 3399.9900 |
    | 2 | 772 | Mountain-100 Silver, 42 | Bicicletas de montaña | 3399.9900 |
    | 3 | 773 | Mountain-100 Silver, 44 | Bicicletas de montaña | 3399.9900 |
    | ... | ... | ... | ... | ... |

## Creación de tablas delta

Puede guardar el objeto DataFrame como una tabla Delta mediante el método `saveAsTable`. Delta Lake admite la creación de tablas *administradas* y *externas*.

### Creación de una tabla *administrada*

Las tablas *administradas* son tablas cuyos metadatos de esquema y archivos de datos administra Fabric. Los archivos de datos de la tabla se crean en la carpeta **Tablas**.

1. En los resultados devueltos por la primera celda de código, use el icono **+Código** para agregar una nueva celda de código si aún no existe.

    > **Sugerencia**: Para ver el icono **+ Código**, mueva el ratón hasta justo debajo y a la izquierda de la salida de la celda actual. Como alternativa, en la barra de menús, en la pestaña **Editar**, seleccione **+ Añadir celda de código**.

2. Escriba el código siguiente en la nueva celda y ejecútela:

    ```python
   df.write.format("delta").saveAsTable("managed_products")
    ```

3. En el panel **Explorador de almacenes de lagos**, en el menú **...** de la carpeta **Tablas**, seleccione **Actualizar**. A continuación, expanda el nodo **Tablas** y compruebe que se ha creado la tabla **managed_products**.

### Creación de una tabla *externa*

También puede crear tablas *externas* en las que los metadatos del esquema se definen en el metastore del almacén de lago, pero los archivos de datos se almacenan en una ubicación externa.

1. Agregue otra nueva celda de código y escriba en ella el código siguiente:

    ```python
   df.write.format("delta").saveAsTable("external_products", path="abfs_path/external_products")
    ```

2. En el panel **Explorador de almacenes de lago**, en el menú **...** de la carpeta **Archivos**, seleccione **Copiar ruta de acceso de ABFS**.

    La ruta de acceso de ABFS es la ruta de acceso completa a la carpeta **Archivos** del almacenamiento de OneLake de su almacén de lago, parecida a esta:

    *abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files*

3. En el código que escribió en la celda de código, reemplace **abfs_path** por la ruta de acceso que copió en el Portapapeles para que el código guarde el objeto DataFrame como una tabla externa con archivos de datos en una carpeta llamada **external_products** en la ubicación de la carpeta **Archivos**. La ruta de acceso completa debe ser similar a la siguiente:

    *abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files/external_products*

4. En el panel **Explorador de almacenes de lagos**, en el menú **...** de la carpeta **Tablas**, seleccione **Actualizar**. A continuación, expanda el nodo **Tablas** y compruebe que se ha creado la tabla **external_products**.

5. En el panel **Explorador de almacenes de lago**, en el menú **...** de la carpeta **Archivos**, seleccione **Actualizar**. A continuación, expanda el nodo **Archivos** y compruebe que se ha creado la carpeta **external_products** para los archivos de datos de la tabla.

### Comparación de tablas *administradas* y *externas*

Vamos a explorar las diferencias entre tablas administradas y externas.

1. Agregue otra celda y escriba el código siguiente:

    ```sql
   %%sql

   DESCRIBE FORMATTED managed_products;
    ```

    En los resultados, examine la propiedad **Location** de la tabla, que debe ser una ruta de acceso al almacenamiento de OneLake del almacén de lago que termina con **/Tablas/managed_products** (es posible que tenga que ensanchar la columna **Tipo de datos** para ver la ruta de acceso completa).

2. Modifique el comando `DESCRIBE` para mostrar los detalles de la tabla **external_products** como se muestra aquí:

    ```sql
   %%sql

   DESCRIBE FORMATTED external_products;
    ```

    En los resultados, examine la propiedad **Location** de la tabla, que debe ser una ruta de acceso al almacenamiento de OneLake del almacén de lago que finaliza con **/Archivos/managed_products** (es posible que tenga que ensanchar la columna **Tipo de datos** para ver la ruta de acceso completa).

    Los archivos de la tabla administrada se almacenan en la carpeta **Tablas** del almacenamiento de OneLake del almacén de lago. En este caso, se ha creado una carpeta llamada **managed_products** para almacenar los archivos Parquet y la carpeta **delta_log** para la tabla que ha creado.

3. Agregue otra celda y escriba el código siguiente:

    ```sql
   %%sql

   DROP TABLE managed_products;
   DROP TABLE external_products;
    ```

4. En el panel **Explorador de almacenes de lagos**, en el menú **...** de la carpeta **Tablas**, seleccione **Actualizar**. A continuación, expanda el nodo **Tablas** y compruebe que no se muestran tablas.

5. En el panel **Explorador de almacenes de lago**, expanda la carpeta **Archivos** y compruebe que la tabla **external_products** no se ha eliminado. Seleccione esta carpeta para ver los archivos de datos de Parquet y la carpeta **_delta_log** para los datos que estaban anteriormente en la tabla **external_products**. Los metadatos de la tabla externa se eliminaron, pero los archivos no se vieron afectados.

### Uso de SQL para crear una tabla

1. Agregue otra celda y escriba el código siguiente:

    ```sql
   %%sql

   CREATE TABLE products
   USING DELTA
   LOCATION 'Files/external_products';
    ```

2. En el panel **Explorador de almacenes de lagos**, en el menú **...** de la carpeta **Tablas**, seleccione **Actualizar**. A continuación, expanda el nodo **Tablas** y compruebe que aparece una nueva tabla llamada **products**. A continuación, expanda la tabla para comprobar que el esquema coincide con el objeto DataFrame original que se guardó en la carpeta **external_products**.

3. Agregue otra celda y escriba el código siguiente:

    ```sql
   %%sql

   SELECT * FROM products;
   ```

## Exploración del control de versiones de tabla

El historial de transacciones de las tablas Delta se almacena en archivos JSON en la carpeta **delta_log**. Puede usar este registro de transacciones para administrar el control de versiones de los datos.

1. Agregue una nueva celda de código al cuaderno y ejecute el código siguiente:

    ```sql
   %%sql

   UPDATE products
   SET ListPrice = ListPrice * 0.9
   WHERE Category = 'Mountain Bikes';
    ```

    Este código aplica una reducción del 10 % en el precio de las bicicletas de montaña.

2. Agregue otra celda y escriba el código siguiente:

    ```sql
   %%sql

   DESCRIBE HISTORY products;
    ```

    Los resultados muestran el historial de transacciones registradas para la tabla.

3. Agregue otra celda y escriba el código siguiente:

    ```python
   delta_table_path = 'Files/external_products'

   # Get the current data
   current_data = spark.read.format("delta").load(delta_table_path)
   display(current_data)

   # Get the version 0 data
   original_data = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
   display(original_data)
    ```

    Los resultados muestran dos objetos DataFrame: uno que contiene los datos después de la reducción de precios y el otro que muestra la versión original de los datos.

## Uso de tablas Delta para transmitir datos

Delta Lake admite datos de streaming. Las tablas Delta pueden ser un *receptor* o un *origen* para flujos de datos creados mediante Spark Structured Streaming API. En este ejemplo, usará una tabla Delta como receptor para algunos datos de streaming en un escenario simulado de Internet de las cosas (IoT).

1. Agregue una nueva celda de código al cuaderno. A continuación, en la nueva celda, agregue el código siguiente y ejecútelo:

    ```python
   from notebookutils import mssparkutils
   from pyspark.sql.types import *
   from pyspark.sql.functions import *

   # Create a folder
   inputPath = 'Files/data/'
   mssparkutils.fs.mkdirs(inputPath)

   # Create a stream that reads data from the folder, using a JSON schema
   jsonSchema = StructType([
   StructField("device", StringType(), False),
   StructField("status", StringType(), False)
   ])
   iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

   # Write some event data to the folder
   device_data = '''{"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"error"}
   {"device":"Dev2","status":"ok"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}'''
   mssparkutils.fs.put(inputPath + "data.txt", device_data, True)
   print("Source stream created...")
    ```

    Asegúrese de que se muestra el mensaje *Flujo de origen creado...* El código que acaba de ejecutar ha creado un origen de datos de streaming basado en una carpeta en la que se han guardado algunos datos, que representan lecturas de dispositivos IoT hipotéticos.

2. En la nueva celda de código, agregue el código siguiente:

    ```python
   # Write the stream to a delta table
   delta_stream_table_path = 'Tables/iotdevicedata'
   checkpointpath = 'Files/delta/checkpoint'
   deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
   print("Streaming to delta sink...")
    ```

    Este código escribe los datos del dispositivo de streaming en formato Delta en una carpeta llamada **iotdevicedata**. Como la ruta de acceso de la ubicación de carpeta es la carpeta **Tablas**, se creará automáticamente una tabla para ella.

3. En la nueva celda de código, agregue el código siguiente:

    ```sql
   %%sql

   SELECT * FROM IotDeviceData;
    ```

    Este código consulta la tabla **IotDeviceData**, que contiene los datos del dispositivo del origen de streaming.

4. En la nueva celda de código, agregue el código siguiente:

    ```python
   # Add more data to the source stream
   more_data = '''{"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"error"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}'''

   mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
    ```

    Este código escribe datos de dispositivo más hipotéticos en el origen de streaming.

5. Vuelva a ejecutar la celda que contiene el código siguiente:

    ```sql
   %%sql

   SELECT * FROM IotDeviceData;
    ```

    Este código consulta de nuevo la tabla **IotDeviceData**, que ahora debe incluir los datos adicionales que se agregaron al origen de streaming.

6. En la nueva celda de código, agregue el código siguiente:

    ```python
   deltastream.stop()
    ```

    Este código detiene la transmisión.

## Limpieza de recursos

En este ejercicio, ha aprendido a trabajar con tablas Delta en Microsoft Fabric.

Si ha terminado de explorar el almacén de lago, puede eliminar el área de trabajo que ha creado para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **Otros**, seleccione **Quitar esta área de trabajo**.
