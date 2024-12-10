---
lab:
  title: Introducción a la consulta de una base de datos KQL en Microsoft Fabric
  module: Query data from a KQL database in Microsoft Fabric
---

# Introducción a la consulta de una base de datos KQL en Microsoft Fabric

Un conjunto de consultas KQL es una herramienta que permite ejecutar consultas, modificar y mostrar los resultados de la consulta desde una base de datos KQL. Puede vincular cada pestaña del conjunto de consultas KQL a una base de datos KQL diferente y guardar las consultas para su uso futuro o compartirlas con otras personas para el análisis de datos. También puede cambiar la base de datos KQL para cualquier pestaña, de forma que pueda comparar los resultados de la consulta de diferentes orígenes de datos.

En este ejercicio, realizarás el rol de un analista encargado de consultar un conjunto de datos de los datos de carreras de taxi de Nueva York. Use KQL para consultar estos datos y recopilar información con el fin de obtener conclusiones informativas sobre los datos.

> **Sugerencia**: el conjunto de consultas KQL usa el lenguaje de consulta Kusto, que es compatible con muchas funciones SQL, para crear consultas. Para obtener más información sobre KQL, consulta [Información general sobre el Lenguaje de consulta Kusto (KQL)](https://learn.microsoft.com/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext).

Este laboratorio se realiza en **25** minutos aproximadamente.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la capacidad gratuita de Fabric habilitada.

1. En la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) en `https://app.fabric.microsoft.com/home?experience=fabric`, selecciona **Real-Time Intelligence**.
1. En la barra de menús de la izquierda, selecciona **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
1. Crea una nueva área de trabajo con el nombre que prefieras y selecciona un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Premium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

## Creación de instancia de Eventhouse

1. En la página principal de **Inteligencia en tiempo real**, crea un nuevo **Centro de eventos** con el nombre que prefieras. Cuando se haya creado el centro de eventos, cierra las indicaciones o sugerencias que se muestran hasta que veas la página del centro de eventos:

   ![Captura de pantalla de un nuevo centro de eventos.](./Images/create-eventhouse.png)
   
1. En el menú **...** de la base de datos KQL que se ha creado en el centro de eventos, selecciona **Obtener datos** > **Muestra**. A continuación, elige los datos de muestra de **Análisis de operaciones de automoción**.

1. Una vez finalizada la carga de datos, comprueba que se ha creado la tabla **Automoción**.

   ![Captura de pantalla de la tabla Automoción en una base de datos de centro de eventos.](./Images/choose-automotive-operations-analytics.png)

## Consulta de datos mediante KQL

Lenguaje de consulta Kusto (KQL) es un lenguaje intuitivo y completo que puedes usar para consultar una base de datos KQL.

### Recuperación de datos de una tabla con KQL

1. En el panel izquierdo de la ventana del centro de eventos, en tu base de datos KQL, selecciona el archivo **queryset** predeterminado. Este archivo contiene algunas consultas KQL de muestra para empezar.
1. Modifica la primera consulta de ejemplo de la siguiente manera.

    ```kql
    Automotive
    | take 100
    ```

    > **NOTA:** El carácter de barra vertical ( | ) se usa para dos propósitos en KQL, entre ellos para separar operadores de consulta en una instrucción de expresión tabular. También se usa como operador OR lógico entre corchetes o paréntesis para indicar que se puede especificar uno de los elementos separados por la barra vertical.

1. Selecciona el código de consulta y ejecútalo para devolver 100 filas de la tabla.

   ![Captura de pantalla del editor de consultas KQL.](./Images/kql-take-100-query.png)

    Puedes ser más preciso al agregar atributos específicos que deseas consultar mediante la palabra clave `project` y después usar la palabra clave `take` para indicarle al motor cuántos registros debe devolver.

1. Escribe, selecciona y ejecuta la siguiente consulta:

    ```kql
    // Use 'project' and 'take' to view a sample number of records in the table and check the data.
    Automotive 
    | project vendor_id, trip_distance
    | take 10
    ```

    > **NOTA:** El uso de // denota un comentario.

    Otra práctica habitual en el análisis consiste en cambiar el nombre de las columnas del conjunto de consultas para que sean más fáciles de usar.

1. Prueba la siguiente consulta:

    ```kql
    Automotive 
    | project vendor_id, ["Trip Distance"] = trip_distance
    | take 10
    ```

### Resumir datos con KQL

Puedes usar la palabra clave *summarize* con una función para agregar y manipular datos de otro modo.

1. Prueba la siguiente consulta, que usa la función **sum** para resumir los datos del viaje y ver cuántas millas se recorrieron en total:

    ```kql

    Automotive
    | summarize ["Total Trip Distance"] = sum(trip_distance)
    ```

    Puedes agrupar los datos resumidos por una columna o expresión especificadas.

1. Ejecuta la siguiente consulta para agrupar las distancias de viaje por distrito dentro del sistema de taxis de NY para determinar la distancia total viajada desde cada distrito.

    ```kql
    Automotive
    | summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
    | project Borough = pickup_boroname, ["Total Trip Distance"]
    ```

    Los resultados incluyen un valor en blanco, que nunca es bueno para el análisis.

1. Modificar la consulta como se muestra aquí para usar la función *case* junto con las funciones *isempty* y *isnull* para agrupar todos los viajes de los que se desconoce el municipio en la categoría ***No identificado*** para su seguimiento.

    ```kql
    Automotive
    | summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
    | project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
    ```

### Ordenar datos mediante KQL

Para darle más sentido a nuestros datos, normalmente los ordenamos por columna, y este proceso se realiza en KQL con un operador *ordenar por***.

1. Prueba la siguiente consulta:

    ```kql
    Automotive
    | summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
    | project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
    | sort by Borough asc
    ```

1. Modifica la consulta de la siguiente manera y ejecútala de nuevo y observa cómo funciona el operador *ordenar por***:

    ```kql
    Automotive
    | summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
    | project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
    | order by Borough asc 
    ```

### Filtrado de datos mediante KQL

En KQL, la cláusula *where* se usa para filtrar los datos. Puedes combinar condiciones en una cláusula *where* mediante los operadores lógicos *y* y *o*.

1. Ejecuta la siguiente consulta para filtrar los datos del viaje para incluir solo los viajes que se originaron en Manhattan:

    ```kql
    Automotive
    | where pickup_boroname == "Manhattan"
    | summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
    | project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
    | sort by Borough asc
    ```

## Consulta de datos mediante Transact-SQL

La base de datos KQL no admite Transact-SQL de forma nativa, pero proporciona un punto de conexión de T-SQL que emula a Microsoft SQL Server y permite ejecutar consultas de T-SQL en los datos. El punto de conexión de T-SQL tiene algunas limitaciones y diferencias con respecto al SQL Server nativo. Por ejemplo, no admite la creación, modificación o eliminación de tablas, ni la inserción, actualización o eliminación de datos. Tampoco admite algunas funciones y sintaxis de T-SQL que no son compatibles con KQL. Se creó para permitir que los sistemas que no admitan KQL usen T-SQL para consultar los datos dentro de una base de datos KQL. Por lo tanto, se recomienda usar KQL como lenguaje de consulta principal para bases de datos KQL, ya que ofrece más funcionalidades y rendimiento que T-SQL. También se pueden usar algunas funciones de SQL compatibles con KQL, como count, sum, avg, min, max, etc.

### Recuperación de datos de una tabla mediante Transact-SQL

1. En el conjunto de consultas, agrega y ejecuta la siguiente consulta de Transact-SQL: 

    ```sql  
    SELECT TOP 100 * from Automotive
    ```

1. Modifica la consulta como se indica a continuación para recuperar columnas específicas

    ```sql
    SELECT TOP 10 vendor_id, trip_distance
    FROM Automotive
    ```

1. Modifica la consulta para asignar un alias que cambie el nombre de **trip_distance** a un nombre más descriptivo.

    ```sql
    SELECT TOP 10 vendor_id, trip_distance as [Trip Distance]
    from Automotive
    ```

### Resumen de datos mediante Transact-SQL

1. Ejecuta la consulta siguiente para encontrar la distancia total que se ha recorrido:

    ```sql
    SELECT sum(trip_distance) AS [Total Trip Distance]
    FROM Automotive
    ```

1. Modifica la consulta para agrupar la distancia total por distrito de recogida:

    ```sql
    SELECT pickup_boroname AS Borough, Sum(trip_distance) AS [Total Trip Distance]
    FROM Automotive
    GROUP BY pickup_boroname
    ```

1. Modifica aún más la consulta para usar una instrucción *CASE* para agrupar viajes con un origen desconocido en una categoría ***No identificada*** para su seguimiento. 

    ```sql
    SELECT CASE
             WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'Unidentified'
             ELSE pickup_boroname
           END AS Borough,
           SUM(trip_distance) AS [Total Trip Distance]
    FROM Automotive
    GROUP BY CASE
               WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'Unidentified'
               ELSE pickup_boroname
             END;
    ```

### Ordenar datos mediante Transact-SQL

1. Ejecuta la siguiente consulta para ordenar los resultados agrupados por distrito.
 
    ```sql
    SELECT CASE
             WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
             ELSE pickup_boroname
           END AS Borough,
           SUM(trip_distance) AS [Total Trip Distance]
    FROM Automotive
    GROUP BY CASE
               WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
               ELSE pickup_boroname
             END
    ORDER BY Borough ASC;
    ```

### Filtrado de datos mediante Transact-SQL
    
1. Ejecuta la siguiente consulta para filtrar los datos agrupados para que solo se incluyan en los resultados filas que tengan en distrito de "Manhattan".

    ```sql
    SELECT CASE
             WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
             ELSE pickup_boroname
           END AS Borough,
           SUM(trip_distance) AS [Total Trip Distance]
    FROM Automotive
    GROUP BY CASE
               WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
               ELSE pickup_boroname
             END
    HAVING Borough = 'Manhattan'
    ORDER BY Borough ASC;
    ```

## Limpieza de recursos

En este ejercicio, has creado un centro de eventos y has consultado datos mediante KQL y SQL.

Si ha terminado de explorar la base de datos KQL, puede eliminar el área de trabajo que ha creado para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo.
2. En la barra de herramientas, selecciona **Configuración del área de trabajo**.
3. En la sección **General**, selecciona **Quitar esta área de trabajo**.
