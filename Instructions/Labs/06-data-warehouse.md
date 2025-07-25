---
lab:
  title: Análisis de los datos en un almacenamiento de datos
  module: Get started with data warehouses in Microsoft Fabric
---

# Análisis de los datos en un almacenamiento de datos

En Microsoft Fabric, un almacenamiento de datos proporciona una base de datos relacional para análisis a gran escala. A diferencia del punto de conexión de SQL de solo lectura predeterminado para las tablas definidas en un almacén de lago, un almacenamiento de datos proporciona semántica de SQL completa; incluida la capacidad de insertar, actualizar y eliminar datos de las tablas.

Este laboratorio se tarda aproximadamente **30** minutos en completarse.

> **Nota**: necesitarás una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, crea un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. En un explorador, ve a la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) en `https://app.fabric.microsoft.com/home?experience=fabric` e inicia sesión con tus credenciales de Fabric.
1. En la barra de menús de la izquierda, selecciona **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
1. Crea una nueva área de trabajo con el nombre que prefieras y selecciona un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Premium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

## Creación del almacenamiento de datos

Ahora que tiene un área de trabajo, es el momento de crear un almacenamiento de datos. La página principal de Synapse Data Warehouse incluye un acceso directo para crear un nuevo almacén:

1. En la barra de menús de la izquierda, selecciona **Crear**. En la página *Nuevo*, en la sección *Almacenamiento de datos*, selecciona **Almacén**. Asígnale un nombre único que elijas.

    >**Nota**: si la opción **Crear** no está anclada a la barra lateral, primero debes seleccionar la opción de puntos suspensivos (**...**).

    Al cabo de un minuto más o menos, se creará un nuevo almacén:

    ![Captura de pantalla de un nuevo almacenamiento.](./Images/new-data-warehouse2.png)

## Creación de tablas e inserción de datos

Un almacenamiento es una base de datos relacional en la que se pueden definir tablas y otros objetos.

1. En el nuevo almacén, selecciona el icono de **T-SQL** y usa la siguiente instrucción CREATE TABLE:

    ```sql
   CREATE TABLE dbo.DimProduct
   (
       ProductKey INTEGER NOT NULL,
       ProductAltKey VARCHAR(25) NULL,
       ProductName VARCHAR(50) NOT NULL,
       Category VARCHAR(50) NULL,
       ListPrice DECIMAL(5,2) NULL
   );
   GO
    ```

2. Use el botón **&#9655; Ejecutar** para ejecutar el script de SQL, que crea una nueva tabla llamada **DimProduct** en el esquema **dbo** del almacenamiento de datos.
3. Use el botón **Actualizar** de la barra de herramientas para actualizar la vista. A continuación, en el panel **Explorador**, expanda **Esquemas** > **dbo** > **Tablas** y compruebe que se ha creado la tabla **DimProduct**.
4. En la pestaña del menú **Inicio**, use el botón **Nueva consulta SQL** para crear una nueva consulta y escriba la siguiente instrucción INSERT:

    ```sql
   INSERT INTO dbo.DimProduct
   VALUES
   (1, 'RING1', 'Bicycle bell', 'Accessories', 5.99),
   (2, 'BRITE1', 'Front light', 'Accessories', 15.49),
   (3, 'BRITE2', 'Rear light', 'Accessories', 15.49);
   GO
    ```

5. Ejecute la nueva consulta para insertar tres filas en la tabla **DimProduct**.
6. Cuando la consulta haya finalizado, en el panel **Explorador**, selecciona la tabla **DimProduct** y comprueba que las tres filas se han agregado a la tabla.
7. En la pestaña del menú **Inicio**, use el botón **Nueva consulta SQL** para crear una nueva consulta. A continuación, copie y pegue el código de Transact-SQL desde `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/create-dw.txt` en el nuevo panel de consulta.
8. Ejecute la consulta, que crea un esquema de almacenamiento de datos simple y carga algunos datos. El script debe tardar unos 30 segundos en ejecutarse.
9. Use el botón **Actualizar** de la barra de herramientas para actualizar la vista. A continuación, en el panel **Explorador**, compruebe que el esquema **dbo** del almacenamiento de datos contiene ahora las cuatro tablas siguientes:
    - **DimCustomer**
    - **DimDate**
    - **DimProduct**
    - **FactSalesOrder**

    > **Sugerencia**: Si el esquema tarda un tiempo en cargarse, actualice la página del explorador.

## Definición de un modelo de datos

Normalmente, un almacenamiento de datos relacional consta de tablas de *hechos* y *dimensiones*. Las tablas de hechos contienen medidas numéricas que se pueden agregar para analizar el rendimiento empresarial (por ejemplo, ingresos de ventas) y las tablas de dimensiones contienen atributos de las entidades por las que puede agregar los datos (por ejemplo, producto, cliente o tiempo). En un almacenamiento de datos de Microsoft Fabric, puede usar estas claves para definir un modelo de datos que encapsula las relaciones entre las tablas.

1. En la barra de herramientas, selecciona el botón **Diseños de modelo**.
2. En el panel del modelo, reorganice las tablas del almacenamiento de datos para que la tabla **FactSalesOrder** esté en el medio, de la siguiente manera:

    ![Captura de pantalla de la página del modelo de almacenamiento de datos.](./Images/model-dw.png)

> **Nota**: las vistas **frequently_run_queries**, **long_running_queries**, **exec_sessions_history** y **exec_requests_history** forman parte del esquema **queryinsights** creado automáticamente por Fabric. Es una característica que proporciona una vista holística de la actividad de consulta histórica en el punto de conexión de análisis SQL. Dado que esta característica está fuera del ámbito de este ejercicio, esas vistas deben omitirse por ahora.

3. Arrastre el campo **ProductKey** de la tabla **FactSalesOrder** y colóquelo en el campo **ProductKey** de la tabla **DimProduct**. A continuación, confirme los siguientes detalles de relación:
    - **De la tabla**: FactSalesOrder
    - **Columna**: ProductKey
    - **A la tabla**: DimProduct
    - **Columna**: ProductKey
    - **Cardinalidad**: Varios a uno (*:1)
    - **Dirección del filtro cruzado**: Único
    - **Activar esta relación**: Seleccionado
    - **Asumir integridad referencial**: No seleccionado

4. Repita el proceso para crear relaciones de varios a uno entre las tablas siguientes:
    - **FactSalesOrder.CustomerKey** &#8594; **DimCustomer.CustomerKey**
    - **FactSalesOrder.SalesOrderDateKey** &#8594; **DimDate.DateKey**

    Cuando se hayan definido todas las relaciones, el modelo debe tener este aspecto:

    ![Captura de pantalla del modelo con las relaciones.](./Images/dw-relationships.png)

## Consulta de las tablas de almacenamiento de datos

Dado que el almacenamiento de datos es una base de datos relacional, puede usar SQL para consultar sus tablas.

### Tablas de hechos y dimensiones

La mayoría de las consultas de un almacenamiento de datos relacional implican agregar y agrupar datos (mediante funciones de agregado y cláusulas GROUP BY) en tablas relacionadas (mediante cláusulas JOIN).

1. Cree una nueva consulta SQL y ejecute el código siguiente:

    ```sql
   SELECT  d.[Year] AS CalendarYear,
            d.[Month] AS MonthOfYear,
            d.MonthName AS MonthName,
           SUM(so.SalesTotal) AS SalesRevenue
   FROM FactSalesOrder AS so
   JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
   GROUP BY d.[Year], d.[Month], d.MonthName
   ORDER BY CalendarYear, MonthOfYear;
    ```

    Ten en cuenta que los atributos de la dimensión de fecha te permiten agregar las medidas de la tabla de hechos en varios niveles jerárquicos, en este caso, año y mes. Se trata de un patrón común en los almacenamientos de datos.

2. Modifique la consulta de la siguiente manera para agregar una segunda dimensión a la agregación.

    ```sql
   SELECT  d.[Year] AS CalendarYear,
           d.[Month] AS MonthOfYear,
           d.MonthName AS MonthName,
           c.CountryRegion AS SalesRegion,
          SUM(so.SalesTotal) AS SalesRevenue
   FROM FactSalesOrder AS so
   JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
   JOIN DimCustomer AS c ON so.CustomerKey = c.CustomerKey
   GROUP BY d.[Year], d.[Month], d.MonthName, c.CountryRegion
   ORDER BY CalendarYear, MonthOfYear, SalesRegion;
    ```

3. Ejecute la consulta modificada y revise los resultados, que ahora incluyen los ingresos de ventas agregados por año, mes y región de ventas.

## Creación de una vista

Un almacenamiento de datos en Microsoft Fabric tiene muchas de las mismas funcionalidades con las que puede estar familiarizado en las bases de datos relacionales. Por ejemplo, puede crear objetos de base de datos como *vistas* y *procedimientos almacenados* para encapsular lógica SQL.

1. Modifique la consulta que creó anteriormente como se indica a continuación para crear una vista (tenga en cuenta que para ello debe quitar la cláusula ORDER BY).

    ```sql
   CREATE VIEW vSalesByRegion
   AS
   SELECT  d.[Year] AS CalendarYear,
           d.[Month] AS MonthOfYear,
           d.MonthName AS MonthName,
           c.CountryRegion AS SalesRegion,
          SUM(so.SalesTotal) AS SalesRevenue
   FROM FactSalesOrder AS so
   JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
   JOIN DimCustomer AS c ON so.CustomerKey = c.CustomerKey
   GROUP BY d.[Year], d.[Month], d.MonthName, c.CountryRegion;
    ```

2. Ejecute la consulta para crear la vista. A continuación, actualice el esquema de almacenamiento de datos y compruebe que la nueva vista aparece en el panel **Explorador**.
3. Cree una nueva consulta SQL y ejecute la siguiente instrucción SELECT:

    ```SQL
   SELECT CalendarYear, MonthName, SalesRegion, SalesRevenue
   FROM vSalesByRegion
   ORDER BY CalendarYear, MonthOfYear, SalesRegion;
    ```

## Creación de una consulta visual

En lugar de escribir código SQL, puede usar el diseñador gráfico de consultas para consultar las tablas en el almacenamiento de datos. Esta experiencia es similar a Power Query en línea, donde puede crear pasos de transformación de datos sin código. En el caso de tareas más complejas, puede usar el lenguaje M (Mashup) de Power Query.

1. En el menú **Inicio**, expande las opciones en **Nueva consulta SQL** y selecciona **Nueva consulta visual**.

1. Arrastre **FactSalesOrder** al **lienzo**. Observe que se muestra una vista previa de la tabla en el panel **Vista previa** siguiente.

1. Arrastre **DimProduct** al **lienzo**. Ahora tenemos dos tablas en nuestra consulta.

2. Use el botón **(+)** de la tabla **FactSalesOrder** en el lienzo para **combinar consultas**.
![Captura de pantalla del lienzo con la tabla FactSalesOrder seleccionada.](./Images/visual-query-merge.png)

1. En la ventana **Combinar consultas**, seleccione **DimProduct** (la tabla correcta para combinar). Seleccione **ProductKey** en ambas consultas, deje el tipo de combinación predeterminado **Externa izquierda** y haga clic en **Aceptar**.

2. En **Vista previa**, observe que la nueva columna **DimProduct** se ha agregado a la tabla FactSalesOrder. Expanda la columna haciendo clic en la flecha situada a la derecha del nombre de la columna. Seleccione **ProductName** y haga clic en **Aceptar**.

    ![Captura de pantalla del panel de vista previa con la columna DimProduct expandida, con ProductName seleccionado.](./Images/visual-query-preview.png)

1. Si está interesado en examinar los datos de un solo producto, de acuerdo con una solicitud de administrador, ahora puede usar la columna **ProductName** para filtrar los datos de la consulta. Filtre la columna **ProductName** para ver solo los datos de **Cable Lock**.

1. Desde aquí, puedes analizar los resultados de esta única consulta con la selección de **Visualizar resultados** o **Descargar archivo de Excel**. Ahora puede ver exactamente lo que solicitó el administrador, por lo que no es necesario seguir analizando los resultados.

## Limpieza de recursos

En este ejercicio, ha creado un almacenamiento de datos que contiene varias tablas. Ha usado SQL para insertar datos en las tablas y tablas consultadas mediante T-SQL y la herramienta de consulta visual. Por último, has mejorado el modelo de datos para el conjunto de datos predeterminado del almacén de datos para análisis e informes de bajada.

Si ha terminado de explorar el almacenamiento de datos, puede eliminar el área de trabajo que creó para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, selecciona **Configuración del área de trabajo**.
3. En la sección **General**, selecciona **Quitar esta área de trabajo**.
