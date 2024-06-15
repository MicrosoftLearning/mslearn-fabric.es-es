---
lab:
  title: Protección de los datos en un almacenamiento de datos
  module: Get started with data warehouses in Microsoft Fabric
---

# Protección de los datos en un almacenamiento de datos

Los permisos de Microsoft Fabric y los permisos de SQL granulares funcionan conjuntamente para controlar el acceso al almacenamiento y los permisos de usuario. En este ejercicio, protegerá los datos mediante permisos granulares, seguridad de nivel de columna, seguridad de nivel de fila y enmascaramiento dinámico de datos.

Este laboratorio tardará aproximadamente **45** minutos en completarse.

> **Nota**: Necesitarás una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. En la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com), seleccione **Synapse Data Warehouse**.
1. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
1. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-empty-workspace.png)

## Creación del almacenamiento de datos

A continuación, creará un almacenamiento de datos en el área de trabajo que acaba de crear. La página principal de Synapse Data Warehouse incluye un acceso directo para crear un nuevo almacén:

1. En la página principal de **Synapse Data Warehouse**, cree un nuevo **almacenamiento** con el nombre que prefiera.

    Al cabo de un minuto más o menos, se creará un nuevo almacenamiento:

    ![Captura de pantalla de un nuevo almacenamiento.](./Images/new-empty-data-warehouse.png)

## Aplicación del enmascaramiento dinámico de datos a columnas de una tabla

Las reglas de enmascaramiento dinámico de datos se aplican en columnas individuales en el nivel de tabla para que todas las consultas se vean afectadas por el enmascaramiento. Los usuarios que no tengan permiso explícito para ver datos confidenciales verán los valores enmascarados en los resultados de la consulta, mientras que los usuarios con permiso explícito para ver los datos los verán sin enmascarar. Hay cuatro tipos de máscara: predeterminado, correo electrónico, aleatorio y cadena personalizada. En este ejercicio, aplicará una máscara predeterminada, una máscara de correo electrónico y una máscara de cadena personalizada.

1. En su almacén, seleccione el icono **T-SQL** y sustituya el código SQL predeterminado por las siguientes instrucciones T-SQL para crear una tabla e insertar y visualizar datos.  Las máscaras aplicadas en la instrucción `CREATE TABLE` hacen lo siguiente:

    ```sql
    CREATE TABLE dbo.Customer
    (   
        CustomerID INT NOT NULL,   
        FirstName varchar(50) MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)') NULL,     
        LastName varchar(50) NOT NULL,     
        Phone varchar(20) MASKED WITH (FUNCTION = 'default()') NULL,     
        Email varchar(50) MASKED WITH (FUNCTION = 'email()') NULL   
    );
    GO
    --Users restricted from seeing masked data will see the following when they query the table
    --The FirstName column shows the first letter of the string with XXXXXXX and none of the last characters.
    --The Phone column shows xxxx
    --The Email column shows the first letter of the email address followed by XXX@XXX.com.
    
    INSERT dbo.Customer (CustomerID, FirstName, LastName, Phone, Email) VALUES
    (29485,'Catherine','Abel','555-555-5555','catherine0@adventure-works.com'),
    (29486,'Kim','Abercrombie','444-444-4444','kim2@adventure-works.com'),
    (29489,'Frances','Adams','333-333-3333','frances0@adventure-works.com');
    GO

    SELECT * FROM dbo.Customer;
    GO
    ```

2. Use el botón **&#9655; Ejecutar** para ejecutar el script de SQL, que crea una nueva tabla llamada **Customer** en el esquema **dbo** del almacenamiento de datos.

3. A continuación, en el panel **Explorador**, expanda **Esquemas** > **dbo** > **Tablas** y compruebe que se ha creado la tabla **Customer**. La instrucción SELECT devuelve datos sin enmascarar porque está conectado como administrador del área de trabajo que puede ver datos sin enmascarar.

4. Conéctese como usuario de prueba que sea miembro del rol de área de trabajo del **visualizador** y ejecute la siguiente instrucción T-SQL.

    ```sql
    SELECT * FROM dbo.Customer;
    GO
    ```

    A este usuario no se le ha concedido permiso UNMASK, por lo que los datos devueltos para las columnas FirstName, Phone y Email se enmascaran porque esas columnas se definieron con una máscara en la instrucción `CREATE TABLE`.

5. Vuelva a conectarse como administrador del área de trabajo y ejecute la siguiente instrucción T-SQL para desenmascarar los datos del usuario de prueba.

    ```sql
    GRANT UNMASK ON dbo.Customer TO [testUser@testdomain.com];
    GO
    ```

6. Vuelva a conectarse como usuario de prueba y ejecute la siguiente instrucción T-SQL.

    ```sql
    SELECT * FROM dbo.Customer;
    GO
    ```

    Los datos se devuelven sin máscara porque al usuario de prueba se le ha concedido el permiso `UNMASK`.

## Aplicación de seguridad a nivel de fila

La seguridad de nivel de fila (RLS) se puede usar para limitar el acceso a las filas en función de la identidad o el rol del usuario que ejecuta una consulta.  En este ejercicio, restringirá el acceso a las filas mediante la creación de una directiva de seguridad y un predicado de seguridad definido como una función con valores de tabla insertados.

1. En el almacén que creó en el último ejercicio, seleccione la lista desplegable **Nueva consulta SQL**.  En el menú desplegable bajo el encabezado **En blanco**, seleccione **Nueva consulta SQL**.

2. Cree una tabla e inserte datos. Para que pueda probar la seguridad a nivel de filas en un paso posterior, sustituya "testuser1@mydomain.com" por un nombre de usuario de su entorno y sustituya "testuser2@mydomain.com" por su nombre de usuario.
    ```sql
    CREATE TABLE dbo.Sales  
    (  
        OrderID INT,  
        SalesRep VARCHAR(60),  
        Product VARCHAR(10),  
        Quantity INT  
    );
    GO
     
    --Populate the table with 6 rows of data, showing 3 orders for each test user. 
    INSERT dbo.Sales (OrderID, SalesRep, Product, Quantity) VALUES
    (1, 'testuser1@mydomain.com', 'Valve', 5),   
    (2, 'testuser1@mydomain.com', 'Wheel', 2),   
    (3, 'testuser1@mydomain.com', 'Valve', 4),  
    (4, 'testuser2@mydomain.com', 'Bracket', 2),   
    (5, 'testuser2@mydomain.com', 'Wheel', 5),   
    (6, 'testuser2@mydomain.com', 'Seat', 5);  
    GO
   
    SELECT * FROM dbo.Sales;  
    GO
    ```

3. Use el botón **&#9655; Ejecutar** para ejecutar el script de SQL, que crea una nueva tabla llamada **Sales** en el esquema **dbo** del almacenamiento de datos.

4. A continuación, en el panel **Explorador**, expanda **Esquemas** > **dbo** > **Tablas** y compruebe que se ha creado la tabla **Sales**.
5. Cree un nuevo esquema, un predicado de seguridad definido como una función y una directiva de seguridad.  

    ```sql
    --Create a separate schema to hold the row-level security objects (the predicate function and the security policy)
    CREATE SCHEMA rls;
    GO
    
    --Create the security predicate defined as an inline table-valued function. A predicate evalutes to true (1) or false (0). This security predicate returns 1, meaning a row is accessible, when a row in the SalesRep column is the same as the user executing the query.

    --Create a function to evaluate which SalesRep is querying the table
    CREATE FUNCTION rls.fn_securitypredicate(@SalesRep AS VARCHAR(60)) 
        RETURNS TABLE  
    WITH SCHEMABINDING  
    AS  
        RETURN SELECT 1 AS fn_securitypredicate_result   
    WHERE @SalesRep = USER_NAME();
    GO 
    
    --Create a security policy to invoke and enforce the function each time a query is run on the Sales table. The security policy has a Filter predicate that silently filters the rows available to read operations (SELECT, UPDATE, and DELETE). 
    CREATE SECURITY POLICY SalesFilter  
    ADD FILTER PREDICATE rls.fn_securitypredicate(SalesRep)   
    ON dbo.Sales  
    WITH (STATE = ON);
    GO
6. Use the **&#9655; Run** button to run the SQL script
7. Then, in the **Explorer** pane, expand **Schemas** > **rls** > **Functions**, and verify that the function has been created.
7. Confirm that you're logged as another user by running the following T-SQL.

    ```sql
    SELECT USER_NAME();
    GO
5. Query the sales table to confirm that row-level security works as expected. You should only see data that meets the conditions in the security predicate defined for the user you're logged in as.

    ```sql
    SELECT * FROM dbo.Sales;
    GO

## Implement column-level security

Column-level security allows you to designate which users can access specific columns in a table. It is implemented by issuing a GRANT statement on a table specifying a list of columns and the user or role that can read them. To streamline access management, assign permissions to roles in lieu of individual users. In this exercise, you will create a table, grant access to a subset of columns on the table, and test that restricted columns are not viewable by a user other than yourself.

1. In the warehouse you created in the earlier exercise, select the **New SQL Query** dropdown.  Under the dropdown under the header **Blank**, select **New SQL Query**.  

2. Create a table and insert data into the table.

 ```sql
    CREATE TABLE dbo.Orders
    (   
        OrderID INT,   
        CustomerID INT,  
        CreditCard VARCHAR(20)      
        );
    GO

    INSERT dbo.Orders (OrderID, CustomerID, CreditCard) VALUES
    (1234, 5678, '111111111111111'),
    (2341, 6785, '222222222222222'),
    (3412, 7856, '333333333333333');
    GO

    SELECT * FROM dbo.Orders;
    GO
 ```

3. Deniegue el permiso para ver una columna en la tabla. La siguiente transacción SQL impedirá que "<testuser@mydomain.com>" vea la columna CreditCard en la tabla Orders. En la instrucción `DENY` siguiente, reemplace testuser@mydomain.com por un nombre de usuario en el sistema que tenga permisos de visualizador en el área de trabajo.

 ```sql
DENY SELECT ON dbo.Orders (CreditCard) TO [testuser@mydomain.com];
 ```

4. Pruebe la seguridad de nivel de columna iniciando sesión en Fabric como el usuario al que denegó seleccionar permisos.

5. Consulte la tabla Orders para confirmar que la seguridad de nivel de columna funciona según lo previsto. La consulta siguiente devolverá solo las columnas OrderID y CustomerID, no la columna CreditCard.  

    ```sql
    SELECT * FROM dbo.Orders;
    GO

    --You'll receive an error because access to the CreditCard column has been restricted.  Try selecting only the OrderID and CustomerID fields and the query will succeed.

    SELECT OrderID, CustomerID from dbo.Orders
    ```

## Configuración de permisos pormenorizados de SQL mediante T-SQL

El almacenamiento de Fabric tiene un modelo de permisos que permite controlar el acceso a los datos en el nivel de área de trabajo y en el nivel de elemento. Cuando necesite un control más granular de lo que los usuarios pueden hacer con los elementos protegibles en un almacén de Fabric, puede utilizar los comandos estándar del lenguaje de control de datos SQL (DCL) `GRANT`, `DENY` y `REVOKE`. En este ejercicio, creará objetos, los protegerá mediante `GRANT` y `DENY` y, a continuación, ejecutará consultas para ver el efecto de aplicar permisos detallados.

1. En el almacén que creó en el ejercicio anterior, seleccione la lista desplegable **Nueva consulta SQL**.  En el encabezado **En blanco**, seleccione **Nueva consulta SQL**.  

2. Creación de un procedimiento almacenado y una tabla.

 ```
    CREATE PROCEDURE dbo.sp_PrintMessage
    AS
    PRINT 'Hello World.';
    GO
  
    CREATE TABLE dbo.Parts
    (
        PartID INT,
        PartName VARCHAR(25)
    );
    GO
    
    INSERT dbo.Parts (PartID, PartName) VALUES
    (1234, 'Wheel'),
    (5678, 'Seat');
    GO  
    
    --Execute the stored procedure and select from the table and note the results you get because you're a member of the Workspace Admin. Look for output from the stored procedure on the 'Messages' tab.
      EXEC dbo.sp_PrintMessage;
    GO
    
    SELECT * FROM dbo.Parts
    GO
  ```

3. A continuación, `DENY SELECT` permisos de la tabla a un usuario que sea miembro del rol Visualizador del área de trabajo y `GRANT EXECUTE` en el procedimiento al mismo usuario.

 ```sql
    DENY SELECT on dbo.Parts to [testuser@mydomain.com];
    GO

    GRANT EXECUTE on dbo.sp_PrintMessage to [testuser@mydomain.com];
    GO

 ```

4. Inicie sesión en Fabric como usuario que especificó en las instrucciones DENY y GRANT anteriores en lugar de [testuser@mydomain.com]. A continuación, pruebe los permisos detallados que acaba de aplicar ejecutando el procedimiento almacenado y consultando la tabla.  

 ```sql
    EXEC dbo.sp_PrintMessage;
    GO
    
    SELECT * FROM dbo.Parts
 ```

## Limpieza de recursos

En este ejercicio, ha aplicado el enmascaramiento dinámico de datos a las columnas de una tabla, ha aplicado la seguridad de nivel de fila, ha implementado la seguridad de nivel de columna y ha configurado permisos detallados de SQL mediante T-SQL.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **General**, seleccione **Quitar esta área de trabajo**.
