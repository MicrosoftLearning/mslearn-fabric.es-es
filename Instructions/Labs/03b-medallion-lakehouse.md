---
lab:
  title: "Crear una arquitectura de medallas en un almacén de lago de Microsoft\_Fabric"
  module: Organize a Fabric lakehouse using medallion architecture design
---

# Crear una arquitectura de medallas en un almacén de lago de Microsoft Fabric

En este ejercicio, creará una arquitectura de medallas en un almacén de lago de Fabric mediante cuadernos. Creará un área de trabajo, creará un almacén de lago, cargará datos en la capa de bronce, transformará los datos y los cargará en la tabla Delta de plata, después transformará aún más los datos y los cargará en las tablas Delta de oro y, entonces, explorará el conjunto de datos y creará relaciones.

Este ejercicio debería tardar en completarse **40** minutos aproximadamente.

> **Nota**: Necesitará una licencia de Microsoft Fabric para realizar este ejercicio. Consulte [Introducción a Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para obtener más información sobre cómo habilitar una licencia de evaluación de Fabric gratuita. Para hacerlo, necesitará una cuenta *profesional* o *educativa* de Microsoft. Si no tiene una, puede [registrarse para obtener una evaluación gratuita de Microsoft Office 365 E3 o superior](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Crear un área de trabajo y habilitar la edición del modelo de datos

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. Inicie sesión en [Microsoft Fabric](https://app.fabric.microsoft.com) en `https://app.fabric.microsoft.com` y seleccione **Power BI**.
2. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
3. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
4. Cuando se abra la nueva área de trabajo, estará vacía, como se muestra aquí:

    ![Captura de pantalla de un área de trabajo vacía en Power BI.](./Images/new-workspace-medallion.png)
5. Vaya a la configuración del área de trabajo y habilite la característica en vista previa (GB) **Edición del modelo de datos**. Esto le permitirá crear relaciones entre tablas en el almacén de lago.

    ![Captura de pantalla de la página de configuración del área de trabajo en Power BI.](./Images/workspace-settings.png)

    > **Nota**: Es posible que tenga que actualizar la pestaña del explorador después de habilitar la característica en vista previa (GB).
## Crear un almacén de lago y cargar datos a la capa de bronce

Ahora que tiene un área de trabajo, es el momento de cambiar a la experiencia *Ingeniería de datos* en el portal de Fabric y crear un almacén de lago de datos para los datos que va a analizar.

1. En la parte inferior izquierda del portal de Power BI, seleccione el icono de **Power BI** y cambie a la experiencia **Ingeniería de datos**.

2. En la página principal de **Ingeniería de datos de Synapse**, cree un nuevo **almacén de lago** con el nombre que prefiera.

    Al cabo de un minuto más o menos, se creará un nuevo almacén de lago vacío. Debe ingerir algunos datos en el almacén de lago de datos para su análisis. Hay varias maneras de hacerlo, pero en este ejercicio simplemente descargará un archivo de texto en el equipo local (o máquina virtual de laboratorio, si procede) y, luego, lo cargará en el almacén de lago.

3. Descargue el archivo de datos de este ejercicio desde `https://github.com/MicrosoftLearning/dp-data/blob/main/orders.zip`. Extraiga los archivos y guárdelos con sus nombres originales en el equipo local (o máquina virtual de laboratorio, si procede). Debe haber 3 archivos con datos de ventas de 3 años: 2019.csv, 2020.csv y 2021.csv.

4. Vuelva a la pestaña del explorador web que contiene el almacén de lago y, en el menú **...** de la carpeta **Archivos** del panel **Explorador**, seleccione **Nueva subcarpeta** y cree una carpeta llamada **bronze**.

5. En el menú **...** de la carpeta **bronze**, seleccione **Cargar** y **Cargar archivos**, entonces cargue los 3 archivos (2019.csv, 2020.csv, y 2021.csv) desde el equipo local (o máquina virtual de laboratorio, si procede) al almacén de lago. Use la tecla Mayús para cargar los 3 archivos a la vez.
   
6. Una vez caragados los archivos, seleccione la carpeta **bronze** y compruebe que los archivos han sido cargados, como se muestra aquí:

    ![Captura de pantalla del archivo products.csv cargado en un almacén de lago.](./Images/bronze-files.png)

## Transformar datos y cargar a la tabla Delta de plata

Ahora que tiene datos en la capa de bronce del almacén de datos, puede usar un cuaderno para transformar los datos y cargarlos a una tabla Delta en la capa de plata. 

1. En la página **Inicio**, viendo el contenido de la carpeta **bronze** en el lago de datos, vaya al menú **Abrir cuaderno** y seleccione **Nuevo cuaderno**.

    Al cabo de unos segundos, se abrirá un nuevo cuaderno que contiene una sola *celda*. Los cuadernos se componen de una o varias celdas que pueden contener *código* o *Markdown* (texto con formato).

2. Cuando se abra el cuaderno, cámbiele el nombre a **Transformar datos para plata** seleccionando el texto **Cuaderno xxxx** en la parte superior izquierda del cuaderno e introduciendo el nuevo nombre.

    ![Captura de pantalla de un nuevo cuaderno llamado Sales.](./Images/sales-notebook-rename.png)

2. Seleccione la celda existente del cuaderno, que contiene un sencillo código comentado. Resalte y elimine estas dos líneas: no necesitará este código.
   
   > **Nota**: Los cuadernos permiten ejecutar código en varios lenguajes, como Python, Scala y SQL. En este ejercicio, usará PySpark y SQL. También puede agregar celdas Markdown para proporcionar texto con formato e imágenes para documentar el código.

3. Pegue el siguiente código en la celda:

    ```python
    from pyspark.sql.types import *
    
    # Create the schema for the table
    orderSchema = StructType([
        StructField("SalesOrderNumber", StringType()),
        StructField("SalesOrderLineNumber", IntegerType()),
        StructField("OrderDate", DateType()),
        StructField("CustomerName", StringType()),
        StructField("Email", StringType()),
        StructField("Item", StringType()),
        StructField("Quantity", IntegerType()),
        StructField("UnitPrice", FloatType()),
        StructField("Tax", FloatType())
        ])

    # Import all files from bronze folder of lakehouse
    df = spark.read.format("csv").option("header", "true").schema(orderSchema).load("Files/bronze/*.csv")
    
    # Display the first 10 rows of the dataframe to preview your data
    display(df.head(10))
    ```

4. Use el botón **&#9655;** (*Ejecutar celda*) a la izquierda de la celda para ejecutar el código.

    > **Nota**: Dado que esta es la primera vez que se ejecuta código de Spark en este cuaderno, se debe iniciar una sesión con Spark. Esto significa que la primera ejecución puede tardar un minuto en completarse. Las ejecuciones posteriores serán más rápidas.

5. Cuando se haya completado el comando de la celda, revise la salida que aparece debajo de ella, que será algo parecido a esto:

    | Índice | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | Correo electrónico | Elemento | Quantity | UnitPrice | Impuesto |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO49172 | 1 | 01-01-2021 | Brian Howard | brian23@adventure-works.com | Road-250 Red, 52 | 1 | 2443.35 | 195.468 |
    | 2 |  SO49173 | 1 | 01-01-2021 | Linda Álvarez | Mountain-200 Silver, 38 | 1 | 2071.4197 | 165.7136 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    El código ejecutado ha cargado los datos de los archivos CSV de la carpeta **bronze** en un dataframe de Spark y, a continuación, ha mostrado las primeras filas del dataframe.

    > **Nota**: Puede borrar, ocultar y cambiar automáticamente el tamaño del contenido de la salida de la celda seleccionando el menú **...** situado en la parte superior izquierda del panel de salida.

6. Ahora agregará columnas para la validación y limpieza de datos, mediante un dataframe de PySpark para agregar columnas y actualizar los valores de algunas de las columnas existentes. Use el botón + para agregar un nuevo bloque de código y agregue el código siguiente a la celda:

    ```python
    from pyspark.sql.functions import when, lit, col, current_timestamp, input_file_name
    
    # Add columns FileName, IsFlagged, CreatedTS and ModifiedTS for data validation and tracking
    df = df.withColumn("FileName", input_file_name())
    df = df.withColumn("IsFlagged", when(col("OrderDate") < '2019-08-01',True).otherwise(False))
    df = df.withColumn("CreatedTS", current_timestamp()).withColumn("ModifiedTS", current_timestamp())
    df = df.withColumn("CustomerID", lit(None).cast("BigInt"))
    df = df.withColumn("ItemID", lit(None).cast("BigInt"))
    
    # Update CustomerName to "Unknown" if CustomerName null or empty
    df = df.withColumn("CustomerName", when((col("CustomerName").isNull() | (col("CustomerName")=="")),lit("Unknown")).otherwise(col("CustomerName")))
    ```

    La primera línea del código que ejecutó importa las funciones necesarias de PySpark. Después, va a agregar nuevas columnas al dataframe para poder realizar un seguimiento del nombre del archivo de origen, si el pedido se marcó como anterior al año fiscal de interés y cuándo se creó y modificó la fila. 
    
    También va a agregar columnas para CustomerID e ItemID, que se rellenarán más adelante.
    
    Por último, va a actualizar la columna CustomerName a “Unknown” si es null o está vacía.

7. Ejecute la celda para ejecutar el código mediante el botón **&#9655;** (*Ejecutar celda*).

8. A continuación, usará SQL Magic para crear el dataframe limpio como una tabla nueva llamada sales_silver en la base de datos de ventas mediante el formato Delta Lake. Cree un nuevo bloque de código y agregue el siguiente código a la celda:

    ```python
     %%sql
    
    -- Create sales_silver table 
    CREATE TABLE sales.sales_silver (
        SalesOrderNumber string
        , SalesOrderLineNumber int
        , OrderDate date
        , CustomerName string
        , Email string
        , Item string
        , Quantity int
        , UnitPrice float
        , Tax float
        , FileName string
        , IsFlagged boolean
        , CustomerID bigint
        , ItemID bigint
        , CreatedTS date
        , ModifiedTS date
    ) USING delta;
    ```

    Este código usa el comando `%sql` magic para ejecutar instrucciones SQL. La primera instrucción crea una nueva base de datos denominada **sales**. La segunda instrucción crea una nueva tabla llamada **sales_silver** en la base de datos **sales**, usando el formato Delta Lake y el dataframe que creó en el anterior bloque de código.

9. Ejecute la celda para ejecutar el código mediante el botón **&#9655;** (*Ejecutar celda*).

10. Seleccione **...** en la sección Tablas en el panel del explorador del almacén de lago y seleccione **Actualizar**. Ahora debería ver la nueva tabla **sales_silver** en la lista. El icono de triángulo indica que es una tabla Delta.

    ![Captura de pantalla de la tabla sales_silver en un almacén de lago.](./Images/sales-silver-table.png)

    > **Nota**: Si no ve la nueva tabla, espere unos segundos y vuelva a seleccionar **Actualizar**, o actualice la pestaña del explorador

11. Ahora va a realizar una operación upsert en una tabla Delta, actualizando los registros existentes según unas condiciones específicas e insertando nuevos registros cuando no se encuentren coincidencias. Agregue un nuevo bloque de código y pegue el siguiente código:

    ```python
    # Update existing records and insert new ones based on a condition defined by the columns SalesOrderNumber, OrderDate, CustomerName, and Item.

    from delta.tables import *
    
    deltaTable = DeltaTable.forPath(spark, 'abfss://1daff8bf-a15d-4063-97c2-fd6381bd00b4@onelake.dfs.fabric.microsoft.com/065c411a-27de-4dec-b4fb-e1df9737f0a0/Tables/sales_silver')
    
    dfUpdates = df
    
    deltaTable.alias('silver') \
      .merge(
        dfUpdates.alias('updates'),
        'silver.SalesOrderNumber = updates.SalesOrderNumber and silver.OrderDate = updates.OrderDate and silver.CustomerName = updates.CustomerName and silver.Item = updates.Item'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "SalesOrderNumber": "updates.SalesOrderNumber",
          "SalesOrderLineNumber": "updates.SalesOrderLineNumber",
          "OrderDate": "updates.OrderDate",
          "CustomerName": "updates.CustomerName",
          "Email": "updates.Email",
          "Item": "updates.Item",
          "Quantity": "updates.Quantity",
          "UnitPrice": "updates.UnitPrice",
          "Tax": "updates.Tax",
          "FileName": "updates.FileName",
          "IsFlagged": "updates.IsFlagged",
          "CustomerID": "updates.CustomerID",
          "ItemID": "updates.ItemID",
          "CreatedTS": "updates.CreatedTS",
          "ModifiedTS": "updates.ModifiedTS"
        }
      ) \
      .execute()
    ```
    Esta operación es importante porque permite actualizar los registros existentes en la tabla según los valores de columnas específicas e insertar nuevos registros cuando no se encuentren coincidencias. Este es un requisito común cuando se cargan datos desde un sistema de origen que pueden contener actualizaciones de registros existentes y registros nuevos.

Ahora tiene datos en la tabla Delta de plata que están listos para ser aún más transformados y modelados.
    

## Transformar datos para la capa de oro

Ha tomado correctamente los datos de la capa de bronce, los ha transformado y cargado en una tabla Delta de plata. Ahora usará un nuevo cuaderno para transformar los datos aún más, modelarlos en un esquema de estrella y cargarlos en tablas Delta de oro.

Tenga en cuenta que podría haber hecho todo esto en un solo cuaderno, pero en este ejercicio se usan cuadernos independientes para demostrar el proceso de transformación de datos de bronce a plata y, a continuación, de plata a oro. Esto puede ayudar con la depuración, la solución de problemas y la reutilización.

1. Vuelva a la página principal de **Ingeniería de datos** y cree un cuaderno llamado **Transformar datos para oro**.

2. En el panel de exploración de almacenes de lago, agregue su almacén de lago **Sales** seleccionando **Agregar** y seleccionando entonces el almacén de lago **Sales** que creó anteriormente. Debería ver la tabla **sales_silver** en la sección **Tablas** del panel de exploración.

3. En el bloque de código existente, quite el texto reutilizable y agregue el código siguiente para cargar datos al dataframe y comenzar a crear el esquema de estrella:

    ```python
    # Load data to the dataframe as a starting point to create the gold layer
    df = spark.read.table("Sales.sales_silver")
    ```

4. Agregue un nuevo bloque de código y pegue el código siguiente para crear la tabla de dimensiones de fecha:

    ```python
        %%sql
    -- Create Date_gold dimension table
    CREATE TABLE IF NOT EXISTS sales.dimdate_gold (
        OrderDate date
        , Day int
        , Month int
        , Year int
        , `mmmyyyy` string
        , yyyymm string
    ) USING DELTA;
    
    ```
    > **Nota**: Puede ejecutar el comando `display(df)` en cualquier momento para comprobar el progreso del trabajo. En este caso, ejecutaría “display(dfdimDate_gold)” para ver el contenido del dataframe dimDate_gold.

5. En un nuevo bloque de código, agregue el código siguiente para actualizar la dimensión de fecha a medida que entran nuevos datos:

    ```python
    from delta.tables import *

    deltaTable = DeltaTable.forPath(spark, 'abfss://1daff8bf-a15d-4063-97c2-fd6381bd00b4@onelake.dfs.fabric.microsoft.com/065c411a-27de-4dec-b4fb-e1df9737f0a0/Tables/dimdate_gold')
    
    dfUpdates = dfdimDate_gold
    
    deltaTable.alias('silver') \
      .merge(
        dfUpdates.alias('updates'),
        'silver.OrderDate = updates.OrderDate'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "OrderDate": "updates.OrderDate",
          "Day": "updates.Day",
          "Month": "updates.Month",
          "Year": "updates.Year",
          "mmmyyyy": "updates.mmmyyyy",
          "yyyymm": "yyyymm"
        }
      ) \
      .execute()
    ```
5. Ahora crearemos nuestra tabla de dimensiones Customer. Agregue un nuevo bloque de código y pegue el siguiente código:

    ```python
   %%sql
    -- Create Customer dimension table
    CREATE TABLE sales.dimCustomer_gold (
        CustomerName string
        , Email string
        , First string
        , Last string
        , CustomerID BIGINT
    ) USING DELTA;
    ```
    
6. En un nuevo bloque de código, agregue el código siguiente para actualizar la dimensión de cliente a medida que entran nuevos datos:

    ```python
    from pyspark.sql.functions import col, split

    # Create Customer_gold dataframe

    dfdimCustomer_silver = df.dropDuplicates(["CustomerName","Email"]).select(col("CustomerName"),col("Email")) \
        .withColumn("First",split(col("CustomerName"), " ").getItem(0)) \
        .withColumn("Last",split(col("CustomerName"), " ").getItem(1)) \
    ```

     Aquí ha creado un nuevo DataFrame “dfdimCustomer_silver” realizando varias transformaciones, como anular duplicados, seleccionar columnas específicas y separar la columna “CustomerName” para crear las columnas “First” y “Last” name. El resultado es un DataFrame con datos de cliente estructurados limpios, incluyendo las columnas “First” y “Last” name extraídas de la columna “CustomerName”.

7. A continuación crearemos la columna ID para nuestros clientes. En un nuevo bloque de código, pegue lo siguiente:

    ```python
    from pyspark.sql.functions import monotonically_increasing_id, col, when

    dfdimCustomer_temp = spark.sql("SELECT * FROM dimCustomer_gold")
    CustomerIDCounters = spark.sql("SELECT COUNT(*) AS ROWCOUNT, MAX(CustomerID) AS MAXCustomerID FROM dimCustomer_gold")
    MAXCustomerID = CustomerIDCounters.select((when(col("ROWCOUNT")>0,col("MAXCustomerID"))).otherwise(0)).first()[0]
    
    dfdimCustomer_gold = dfdimCustomer_silver.join(dfdimCustomer_temp,(dfdimCustomer_silver.CustomerName == dfdimCustomer_temp.CustomerName) & (dfdimCustomer_silver.Email == dfdimCustomer_temp.Email), "left_anti")
    
    dfdimCustomer_gold = dfdimCustomer_gold.withColumn("CustomerID",monotonically_increasing_id() + MAXCustomerID)
    
    ```
    Aquí va a limpiar y transformar los datos del cliente (dfdimCustomer_silver) mediante la realización de una anticombinación izquierda para excluir duplicados que ya existen en la tabla dimCustomer_gold y, a continuación, generar valores CustomerID únicos mediante la función monotonically_increasing_id().

8. Ahora se asegurará de que su tabla de clientes se mantenga actualizada conforme vayan entrando nuevos datos. En un nuevo bloque de código, pegue lo siguiente:

    ```python
    from delta.tables import *

    deltaTable = DeltaTable.forPath(spark, 'abfss://1daff8bf-a15d-4063-97c2-fd6381bd00b4@onelake.dfs.fabric.microsoft.com/065c411a-27de-4dec-b4fb-e1df9737f0a0/Tables/dimcustomer_gold')
    
    dfUpdates = dfdimCustomer_gold
    
    deltaTable.alias('silver') \
      .merge(
        dfUpdates.alias('updates'),
        'silver.CustomerName = updates.CustomerName AND silver.Email = updates.Email'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "CustomerName": "updates.CustomerName",
          "Email": "updates.Email",
          "First": "updates.First",
          "Last": "updates.Last",
          "CustomerID": "updates.CustomerID"
        }
      ) \
      .execute()
    ```
9. Ahora repetirá esos pasos para crear la dimensión del producto. En un nuevo bloque de código, pegue lo siguiente:

    ```python
    %%sql
    -- Create Product dimension table
    CREATE TABLE sales.dimProduct_gold (
        Item string
        , ItemID BIGINT
    ) USING DELTA;
    ```    
10. Agregue otro bloque de código para crear el dataframe customer_gold. Usará esto más adelante en el join de Sales.
    
    ```python
    from pyspark.sql.functions import col, split, lit

    # Create Customer_gold dataframe, this dataframe will be used later on on the Sales join
    
    dfdimProduct_silver = df.dropDuplicates(["Item"]).select(col("Item")) \
        .withColumn("ItemName",split(col("Item"), ", ").getItem(0)) \
        .withColumn("ItemInfo",when((split(col("Item"), ", ").getItem(1).isNull() | (split(col("Item"), ", ").getItem(1)=="")),lit("")).otherwise(split(col("Item"), ", ").getItem(1))) \
    
    # display(dfdimProduct_gold)
            ```

11. Now you'll prepare to add new products to the dimProduct_gold table. Add the following syntax to a new code block:

    ```python
    from pyspark.sql.functions import monotonically_increasing_id, col

    dfdimProduct_temp = spark.sql("SELECT * FROM dimProduct_gold")
    Product_IDCounters = spark.sql("SELECT COUNT(*) AS ROWCOUNT, MAX(ItemID) AS MAXProductID FROM dimProduct_gold")
    MAXProduct_ID = Product_IDCounters.select((when(col("ROWCOUNT")>0,col("MAXProductID"))).otherwise(0)).first()[0]
    
    
    dfdimProduct_gold = dfdimProduct_gold.withColumn("ItemID",monotonically_increasing_id() + MAXProduct_ID)
    
    #display(dfdimProduct_gold)

12.  Similar to what you've done with your other dimensions, you need to ensure that your product table remains up-to-date as new data comes in. In a new code block, paste the following:
    
    ```python
    from delta.tables import *
    
    deltaTable = DeltaTable.forPath(spark, 'abfss://Learn@onelake.dfs.fabric.microsoft.com/Sales.Lakehouse/Tables/dimproduct_gold')
    
    dfUpdates = dfdimProduct_gold
    
    deltaTable.alias('silver') \
      .merge(
        dfUpdates.alias('updates'),
        'silver.ItemName = updates.ItemName AND silver.ItemInfo = updates.ItemInfo'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "ItemName": "updates.ItemName",
          "ItemInfo": "updates.ItemInfo",
          "ItemID": "updates.ItemID"
        }
      ) \
      .execute()
    ```

    Esto calcula el siguiente id. de producto disponible según los datos actuales de la tabla, asignando estos nuevos identificadores a los productos, y mostrando entonces la información de producto actualizada (en el caso de que el comando display no esté comentado).

Ahora que ha creado las dimensiones, el último paso es crear la tabla de hechos.

13. En un nuevo bloque de código, pegue el código siguiente para crear la tabla de hechos:

    ```python
       %%sql
    -- Create Date_gold dimension table if not exist
    CREATE TABLE IF NOT EXISTS sales.factsales_gold (
        CustomerID BIGINT
        , ItemID BIGINT
        , OrderDate date
        , Quantity INT
        , UnitPrice float
        , Tax float
    ) USING DELTA;
    ```
14. En un nuevo bloque de código, pegue el código siguiente para crear un nuevo dataframe para combinar los datos de ventas con la información del cliente y del producto, como el identificador de cliente, el identificador del artículo, la fecha de pedido, la cantidad, el precio unitario y los impuestos:

    ```python
    from pyspark.sql.functions import col

    dfdimCustomer_temp = spark.sql("SELECT * FROM dimCustomer_gold")
    dfdimProduct_temp = spark.sql("SELECT * FROM dimProduct_gold")
    
    df = df.withColumn("ItemName",split(col("Item"), ", ").getItem(0)) \
        .withColumn("ItemInfo",when((split(col("Item"), ", ").getItem(1).isNull() | (split(col("Item"), ", ").getItem(1)=="")),lit("")).otherwise(split(col("Item"), ", ").getItem(1))) \
    
    # Create Sales_gold dataframe
    
    dffactSales_gold = df.alias("df1").join(dfdimCustomer_temp.alias("df2"),(df.CustomerName == dfdimCustomer_temp.CustomerName) & (df.Email == dfdimCustomer_temp.Email), "left") \
            .join(dfdimProduct_temp.alias("df3"),(df.ItemName == dfdimProduct_temp.ItemName) & (df.ItemInfo == dfdimProduct_temp.ItemInfo), "left") \
        .select(col("df2.CustomerID") \
            , col("df3.ItemID") \
            , col("df1.OrderDate") \
            , col("df1.Quantity") \
            , col("df1.UnitPrice") \
            , col("df1.Tax") \
        ).orderBy(col("df1.OrderDate"), col("df2.CustomerID"), col("df3.ItemID"))
    
    
    display(dffactSales_gold)
    ```

15. Ahora se asegurará de que los datos de ventas permanecen actualizados mediante la ejecución del código siguiente en un nuevo bloque de código:
    ```python
    from delta.tables import *

    deltaTable = DeltaTable.forPath(spark, 'abfss://Learn@onelake.dfs.fabric.microsoft.com/Sales.Lakehouse/Tables/factsales_gold')
    
    dfUpdates = dffactSales_gold
    
    deltaTable.alias('silver') \
      .merge(
        dfUpdates.alias('updates'),
        'silver.OrderDate = updates.OrderDate AND silver.CustomerID = updates.CustomerID AND silver.ItemID = updates.ItemID'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "CustomerID": "updates.CustomerID",
          "ItemID": "updates.ItemID",
          "OrderDate": "updates.OrderDate",
          "Quantity": "updates.Quantity",
          "UnitPrice": "updates.UnitPrice",
          "Tax": "updates.Tax"
        }
      ) \
      .execute()
    ```
     Aquí va a usar la operación merge de Delta Lake para sincronizar y actualizar la tabla de factsales_gold con nuevos datos de ventas (dffactSales_gold). La operación compara la fecha de pedido, el identificador de cliente y el identificador de producto entre los datos existentes (tabla de plata) y los nuevos datos (actualiza DataFrame), actualizando los registros coincidentes e insertando nuevos registros según sea necesario.

Ahora tiene una capa de oro mantenida y modelada que puede usarse para informes y análisis.

## Crear un conjunto de datos

En el área de trabajo, ahora puede usar la capa de oro para crear un informe y analizar los datos. Puede acceder al conjunto de datos directamente en el área de trabajo para crear relaciones y medidas para los informes.

Tenga en cuenta que no puede usar el conjunto de datos predeterminado que se crea automáticamente al crear un almacén de lago. Debe crear un nuevo conjunto de datos que incluya las tablas de oro que creó en este ejercicio, desde el explorador de almacenes de lago.

1. En el área de trabajo, vaya a su almacén de lago **Sales**.
2. Seleccione **Nuevo conjunto de datos de Power BI** en la cinta de la vista de exploración del almacén de lago.
3. Seleccione las tablas de oro transformadas para incluirlas en el conjunto de datos y seleccione **Confirmar**.
   - dimdate_gold
   - dimcustomer_gold
   - dimproduct_gold
   - factsales_gold

    Esto abrirá el conjunto de datos en Fabric, donde podrá crear relaciones y medidas.

4. Cambie el nombre del conjunto de datos para que sea más fácil de identificar. Seleccione el nombre del conjunto de datos en la esquina superior izquierda de la ventana. Cambie el nombre del conjunto de datos a **Sales_Gold**.

Desde aquí, usted u otros miembros del equipo de datos pueden crear informes y paneles basados en los datos del almacén de lago. Estos informes se conectarán directamente a la capa de oro de su lago, por lo que siempre reflejarán los datos más recientes.

## Limpieza de recursos

En este ejercicio, ha aprendido a crear una arquitectura de medallas en un almacén de lago de Microsoft Fabric.

Si ha terminado de explorar el almacén de lago, puede eliminar el área de trabajo que ha creado para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **Otros**, seleccione **Quitar esta área de trabajo**.
