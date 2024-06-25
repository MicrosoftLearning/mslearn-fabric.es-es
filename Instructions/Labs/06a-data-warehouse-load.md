---
lab:
  title: Carga de datos en un almacenamiento mediante T-SQL
  module: Load data into a warehouse in Microsoft Fabric
---

# Carga de datos en un almacenamiento

En Microsoft Fabric, un almacenamiento de datos proporciona una base de datos relacional para análisis a gran escala. A diferencia del punto de conexión de SQL de solo lectura predeterminado para las tablas definidas en un almacén de lago, un almacenamiento de datos proporciona semántica de SQL completa; incluida la capacidad de insertar, actualizar y eliminar datos de las tablas.

Este laboratorio se tarda aproximadamente **30** minutos en completarse.

> **Nota**: Necesitará una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. En la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com), seleccione **Synapse Data Warehouse**.
1. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
1. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

## Creación de un almacén de lago y carga de archivos

En nuestro escenario, dado que no tenemos datos disponibles, debemos ingerirlos para cargar el almacenamiento. Creará una instancia de Data Lakehouse para los archivos de datos que va a usar para cargar el almacenamiento.

1. En la página principal de **Ingeniería de datos de Synapse**, cree un nuevo **almacén de lago** con el nombre que prefiera.

    Al cabo de un minuto más o menos, se creará un nuevo almacén de lago vacío. Debe ingerir algunos datos en el almacén de lago de datos para su análisis. Hay varias maneras de hacerlo, pero en este ejercicio descargará un archivo CSV en el equipo local (o máquina virtual de laboratorio si procede) y a continuación, lo cargará en el almacén de lago.

1. Descargue el archivo de este ejercicio desde `https://github.com/MicrosoftLearning/dp-data/raw/main/sales.csv`.

1. Vuelva a la pestaña del explorador web que contiene el almacén de lago y, en el menú **...** de la carpeta **Archivos** del panel **Explorador**, seleccione **Cargar** y **Cargar archivos** y, luego, cargue el archivo **sales.csv** del equipo local (o la máquina virtual de laboratorio, si procede) en el almacén de lago.

1. Una vez cargados los archivos, seleccione **Archivos**. Compruebe que se ha cargado el archivo CSV, como se muestra aquí:

    ![Captura de pantalla del archivo cargado en una instancia de un almacén de lago.](./Images/sales-file-upload.png)

## Creación de una tabla en el almacén de lago

1. En el menú **...** del archivo **sales.csv** en el panel**Explorer**, seleccione **Cargar en tablas** y, a continuación, **Nueva tabla**.

1. Proporcione la siguiente información en el cuadro de diálogo **Cargar archivo en nueva tabla**.
    - **Nuevo nombre de tabla:** staging_sales
    - **Usar encabezado para los nombres de columnas:** seleccionado
    - **Separador:** ,

1. Seleccione **Cargar**.

## Crear un almacén

Ahora que tiene un área de trabajo, una instancia de un almacén de lago y la tabla de ventas con los datos que necesita, es el momento de crear un almacenamiento de datos. La página principal de Synapse Data Warehouse incluye un acceso directo para crear un nuevo almacén:

1. En la página principal de **Synapse Data Warehouse**, cree un nuevo **almacenamiento** con el nombre que prefiera.

    Al cabo de un minuto más o menos, se creará un nuevo almacenamiento:

    ![Captura de pantalla de un nuevo almacenamiento.](./Images/new-data-warehouse.png)

## Creación de una tabla de hechos, dimensiones y vista

Vamos a crear las tablas de hechos y las dimensiones de los datos de Sales. También creará una vista que apunte a una instancia de un almacén de lago, lo que simplifica el código del procedimiento almacenado que usaremos para cargar.

1. En el área de trabajo, seleccione el almacén que creó.

1. En el almacén **Explorer**, seleccione **Nueva consulta SQL** y, a continuación, copie y ejecute la siguiente consulta.

    ```sql
    CREATE SCHEMA [Sales]
    GO
        
    IF NOT EXISTS (SELECT * FROM sys.tables WHERE name='Fact_Sales' AND SCHEMA_NAME(schema_id)='Sales')
        CREATE TABLE Sales.Fact_Sales (
            CustomerID VARCHAR(255) NOT NULL,
            ItemID VARCHAR(255) NOT NULL,
            SalesOrderNumber VARCHAR(30),
            SalesOrderLineNumber INT,
            OrderDate DATE,
            Quantity INT,
            TaxAmount FLOAT,
            UnitPrice FLOAT
        );
    
    IF NOT EXISTS (SELECT * FROM sys.tables WHERE name='Dim_Customer' AND SCHEMA_NAME(schema_id)='Sales')
        CREATE TABLE Sales.Dim_Customer (
            CustomerID VARCHAR(255) NOT NULL,
            CustomerName VARCHAR(255) NOT NULL,
            EmailAddress VARCHAR(255) NOT NULL
        );
        
    ALTER TABLE Sales.Dim_Customer add CONSTRAINT PK_Dim_Customer PRIMARY KEY NONCLUSTERED (CustomerID) NOT ENFORCED
    GO
    
    IF NOT EXISTS (SELECT * FROM sys.tables WHERE name='Dim_Item' AND SCHEMA_NAME(schema_id)='Sales')
        CREATE TABLE Sales.Dim_Item (
            ItemID VARCHAR(255) NOT NULL,
            ItemName VARCHAR(255) NOT NULL
        );
        
    ALTER TABLE Sales.Dim_Item add CONSTRAINT PK_Dim_Item PRIMARY KEY NONCLUSTERED (ItemID) NOT ENFORCED
    GO
    ```

    > **Importante:** En un almacenamiento de datos, las restricciones de clave externa no siempre son necesarias en el nivel de tabla. Aunque las restricciones de clave externa pueden ayudar a garantizar la integridad de los datos, también pueden sobrecargar el proceso ETL (extracción, transformación, carga) y ralentizar la carga de datos. La decisión de usar restricciones de clave externa en un almacenamiento de datos debe basarse en una consideración cuidadosa de las ventajas entre la integridad y el rendimiento de los datos.

1. In el **Explorer**, vaya a **Esquemas>> Sales >> Tablas**. Tenga en cuenta las tablas *Fact_Sales*, *Dim_Customer* y *Dim_Item* que acaba de crear.

1. Abra un nuevo **Nuevo editor de consultas SQL** y, a continuación, copie y ejecute la siguiente consulta. Actualice *<your lakehouse name>* con el almacén de lago que creó.

    ```sql
    CREATE VIEW Sales.Staging_Sales
    AS
    SELECT * FROM [<your lakehouse name>].[dbo].[staging_sales];
    ```

1. En el **Explorer**, vaya a **Esquemas>> Sales >> Vistas**. Tenga en cuenta la vista *Staging_Sales* que ha creado.

## Carga de datos en el almacenamiento

Ahora que se crean las tablas de hechos y dimensiones, vamos a crear un procedimiento almacenado para cargar los datos de nuestro almacén de lago en el almacén. Debido al punto de conexión SQL automático creado al crear el almacén de lago, puede acceder directamente a los datos de su instancia de almacén de lago desde el almacén mediante consultas de T-SQL y entre bases de datos.

Por motivos de simplicidad en este caso práctico, usará el nombre del cliente y el nombre del elemento como claves principales.

1. Cree un nuevo **Nuevo editor de consultas SQL** y, a continuación, copie y ejecute la siguiente consulta.

    ```sql
    CREATE OR ALTER PROCEDURE Sales.LoadDataFromStaging (@OrderYear INT)
    AS
    BEGIN
        -- Load data into the Customer dimension table
        INSERT INTO Sales.Dim_Customer (CustomerID, CustomerName, EmailAddress)
        SELECT DISTINCT CustomerName, CustomerName, EmailAddress
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear
        AND NOT EXISTS (
            SELECT 1
            FROM Sales.Dim_Customer
            WHERE Sales.Dim_Customer.CustomerName = Sales.Staging_Sales.CustomerName
            AND Sales.Dim_Customer.EmailAddress = Sales.Staging_Sales.EmailAddress
        );
        
        -- Load data into the Item dimension table
        INSERT INTO Sales.Dim_Item (ItemID, ItemName)
        SELECT DISTINCT Item, Item
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear
        AND NOT EXISTS (
            SELECT 1
            FROM Sales.Dim_Item
            WHERE Sales.Dim_Item.ItemName = Sales.Staging_Sales.Item
        );
        
        -- Load data into the Sales fact table
        INSERT INTO Sales.Fact_Sales (CustomerID, ItemID, SalesOrderNumber, SalesOrderLineNumber, OrderDate, Quantity, TaxAmount, UnitPrice)
        SELECT CustomerName, Item, SalesOrderNumber, CAST(SalesOrderLineNumber AS INT), CAST(OrderDate AS DATE), CAST(Quantity AS INT), CAST(TaxAmount AS FLOAT), CAST(UnitPrice AS FLOAT)
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear;
    END
    ```
1. Cree un nuevo **Nuevo editor de consultas SQL** y, a continuación, copie y ejecute la siguiente consulta.

    ```sql
    EXEC Sales.LoadDataFromStaging 2021
    ```

    > **Nota:** En este caso, solo estamos cargando datos del año 2021. Sin embargo, tiene la opción de modificarlo para cargar datos de años anteriores.

## Ejecución de consultas de análisis

Vamos a ejecutar algunas consultas analíticas para validar los datos en el almacenamiento.

1. En el menú superior, seleccione **Nueva consulta SQL**y, a continuación, copie y ejecute la siguiente consulta.

    ```sql
    SELECT c.CustomerName, SUM(s.UnitPrice * s.Quantity) AS TotalSales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Customer c
    ON s.CustomerID = c.CustomerID
    WHERE YEAR(s.OrderDate) = 2021
    GROUP BY c.CustomerName
    ORDER BY TotalSales DESC;
    ```

    > **Nota:** Esta consulta muestra los clientes por ventas totales para el año 2021. El cliente con las ventas totales más altas del año especificado es **Jordan Turner**, con unas ventas totales de **14686.69**. 

1. En el menú superior, seleccione **Nueva consulta SQL**o reutilice el mismo editor y, a continuación, copie y ejecute la siguiente consulta.

    ```sql
    SELECT i.ItemName, SUM(s.UnitPrice * s.Quantity) AS TotalSales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Item i
    ON s.ItemID = i.ItemID
    WHERE YEAR(s.OrderDate) = 2021
    GROUP BY i.ItemName
    ORDER BY TotalSales DESC;

    ```

    > **Nota:** En esta consulta se muestran los elementos iniciales por ventas totales para el año 2021. Estos resultados sugieren que el modelo *Mountain-200 bike*, en los colores negro y plateado, fue el elemento más popular entre los clientes en 2021.

1. En el menú superior, seleccione **Nueva consulta SQL**o reutilice el mismo editor y, a continuación, copie y ejecute la siguiente consulta.

    ```sql
    WITH CategorizedSales AS (
    SELECT
        CASE
            WHEN i.ItemName LIKE '%Helmet%' THEN 'Helmet'
            WHEN i.ItemName LIKE '%Bike%' THEN 'Bike'
            WHEN i.ItemName LIKE '%Gloves%' THEN 'Gloves'
            ELSE 'Other'
        END AS Category,
        c.CustomerName,
        s.UnitPrice * s.Quantity AS Sales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Customer c
    ON s.CustomerID = c.CustomerID
    JOIN Sales.Dim_Item i
    ON s.ItemID = i.ItemID
    WHERE YEAR(s.OrderDate) = 2021
    ),
    RankedSales AS (
        SELECT
            Category,
            CustomerName,
            SUM(Sales) AS TotalSales,
            ROW_NUMBER() OVER (PARTITION BY Category ORDER BY SUM(Sales) DESC) AS SalesRank
        FROM CategorizedSales
        WHERE Category IN ('Helmet', 'Bike', 'Gloves')
        GROUP BY Category, CustomerName
    )
    SELECT Category, CustomerName, TotalSales
    FROM RankedSales
    WHERE SalesRank = 1
    ORDER BY TotalSales DESC;
    ```

    > **Nota:** Los resultados de esta consulta muestran el cliente superior para cada una de las categorías: Bicicleta, casco y guantes, en función de sus ventas totales. Por ejemplo, **Joan Coleman** es el cliente principal de la categoría **Guantes**.
    >
    > La información de categoría se extrajo de la columna `ItemName` mediante la manipulación de cadenas, ya que no hay ninguna columna de categoría independiente en la tabla de dimensiones. Este enfoque supone que los nombres de elemento siguen una convención de nomenclatura coherente. Si los nombres de elemento no siguen una convención de nomenclatura coherente, es posible que los resultados no reflejen con precisión la categoría verdadera de cada elemento.

En este ejercicio, ha creado una instancia de almacén de lago y un almacenamiento de datos con varias tablas. Ha ingerido datos y ha usado consultas entre bases de datos para cargar datos desde un almacén de lago al almacén. Además, ha usado la herramienta de consulta para realizar consultas analíticas.

## Limpieza de recursos

Si ha terminado de explorar el almacenamiento de datos, puede eliminar el área de trabajo que creó para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **General**, seleccione **Quitar esta área de trabajo**.
