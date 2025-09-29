---
lab:
  title: Asegure un almacenamiento de datos de Microsoft Fabric
  module: Secure a Microsoft Fabric data warehouse
---

# Protección de un almacenamiento de datos de Microsoft Fabric

Los permisos de Microsoft Fabric y los permisos de SQL granulares funcionan conjuntamente para controlar el acceso al almacenamiento y los permisos de usuario. En este ejercicio, protegerá los datos mediante permisos granulares, seguridad de nivel de columna, seguridad de nivel de fila y enmascaramiento dinámico de datos.

> **Nota**: Hay pasos opcionales en este ejercicio que requieren una segunda cuenta de usuario para la validación de resultados: a un usuario se le debe asignar el rol Administrador del área de trabajo y el otro debe tener el rol Espectador del área de trabajo. Para asignar roles a las áreas de trabajo, consulta [Concesión de acceso al área de trabajo](https://learn.microsoft.com/fabric/get-started/give-access-workspaces). Si no tiene acceso a una segunda cuenta de la misma organización, puede seguir realizando el ejercicio como Administrador del área de trabajo, consultando los recortes de pantalla del ejercicio para ver a qué tiene acceso una cuenta de Espectador del área de trabajo.

Este laboratorio se realiza en **45** minutos aproximadamente.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, crea un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. En un explorador, ve a la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) en `https://app.fabric.microsoft.com/home?experience=fabric` e inicia sesión con tus credenciales de Fabric.
1. En la barra de menús de la izquierda, selecciona **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
1. Crea una nueva área de trabajo con el nombre que prefieras y selecciona un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Premium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-empty-workspace.png)

> **Nota**: Al crear un área de trabajo, se convierte automáticamente en miembro del rol de administrador del área de trabajo. Puede agregar un segundo usuario de su entorno al rol de espectador de áreas de trabajo para probar la funcionalidad configurada en estos ejercicios. Para ello, seleccione **Administrar acceso** en el área de trabajo y, a continuación, **Agregar personas o grupos**. Esto permitirá al segundo usuario ver el contenido del área de trabajo.

## Creación del almacenamiento de datos

A continuación, crearás un almacenamiento de datos en el área de trabajo que has creado:

1. En la barra de menús de la izquierda, selecciona **Crear**. En la página *Nuevo*, en la sección *Almacenamiento de datos*, selecciona **Almacén**. Asígnale un nombre único que elijas.

    >**Nota**: si la opción **Crear** no está anclada a la barra lateral, primero debes seleccionar la opción de puntos suspensivos (**...**).

    Al cabo de un minuto más o menos, se creará un nuevo almacén:

    ![Captura de pantalla de un nuevo almacenamiento.](./Images/new-empty-data-warehouse.png)

## Aplicación del enmascaramiento dinámico de datos a columnas de una tabla

Las reglas de enmascaramiento dinámico de datos se aplican en columnas individuales en el nivel de tabla para que todas las consultas se vean afectadas por el enmascaramiento. Los usuarios que no tengan permiso explícito para ver datos confidenciales ven los valores enmascarados en los resultados de la consulta, mientras que los usuarios con permiso explícito para ver los datos los ven sin enmascarar. Hay cuatro tipos de máscara: predeterminado, correo electrónico, aleatorio y cadena personalizada. En este ejercicio, aplicará una máscara predeterminada, una máscara de correo electrónico y una máscara de cadena personalizada.

1. En tu almacén, selecciona el icono **T-SQL** y usa las siguientes instrucciones T-SQL para crear una tabla e insertar y visualizar datos.  

    ```T-SQL
   CREATE TABLE dbo.Customers
   (   
       CustomerID INT NOT NULL,   
       FirstName varchar(50) MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)') NULL,     
       LastName varchar(50) NOT NULL,     
       Phone varchar(20) MASKED WITH (FUNCTION = 'default()') NULL,     
       Email varchar(50) MASKED WITH (FUNCTION = 'email()') NULL   
   );
   
   INSERT dbo.Customers (CustomerID, FirstName, LastName, Phone, Email) VALUES
   (29485,'Catherine','Abel','555-555-5555','catherine0@adventure-works.com'),
   (29486,'Kim','Abercrombie','444-444-4444','kim2@adventure-works.com'),
   (29489,'Frances','Adams','333-333-3333','frances0@adventure-works.com');
   
   SELECT * FROM dbo.Customers;
    ```

    Cuando los usuarios que tienen restringida la visualización de datos sin máscara consultan la tabla, la columna **FirstName** mostrará la primera letra de la cadena con XXXXXXX y ninguno de los últimos caracteres. La columna **Phone** mostrará xxxx. La columna **Email** mostrará la primera letra de la dirección de correo electrónico seguida de `XXX@XXX.com`. Este enfoque garantiza que los datos confidenciales permanecen confidenciales, a la vez que permiten a los usuarios restringidos consultar la tabla.

2. Use el botón **&#9655; Ejecutar** para ejecutar el script de SQL, que crea una nueva tabla llamada **Clientes** en el esquema **dbo** del almacenamiento de datos.

3. A continuación, en el panel **Explorador**, expanda **Esquemas** > **dbo** > **Tablas** y compruebe que se ha creado la tabla **Clientes**. La instrucción `SELECT` devuelve datos sin máscara porque, como creador del área de trabajo, es miembro del rol de administrador del área de trabajo que puede ver datos sin máscara.

    >**Nota**: Si se conecta como un usuario de prueba que es miembro del rol de área de trabajo **Espectador** y ejecuta una instrucción `SELECT` en la tabla **Customers**, verá los siguientes resultados para los datos enmascarados.
   
    ![Recorte de pantalla de la tabla Customers con datos enmascarados.](./Images/masked-table.png)
 
    Al usuario de prueba no se le ha concedido permiso UNMASK, por lo que los datos devueltos para las columnas FirstName, Phone y Email se enmascaran porque esas columnas se han definido con una máscara en la instrucción `CREATE TABLE`.

## Aplicación de seguridad a nivel de fila

La seguridad de nivel de fila (RLS) se puede usar para limitar el acceso a las filas en función de la identidad o el rol del usuario que ejecuta una consulta. En este ejercicio, va a restrigir el acceso a las filas mediante la creación de una directiva de seguridad y un predicado de seguridad definido como una función con valores de tabla insertados.

1. En el almacén que has creado en el último ejercicio, selecciona el desplegable **Nueva consulta SQL** y selecciona **Nueva consulta SQL**.

2. Cree una tabla e inserte datos. Para implementar la seguridad de nivel de fila en un paso posterior, reemplace `<username1>@<your_domain>.com` por un nombre de usuario ficticio o uno real de su entorno (rol de **Espectador** ) y reemplace `<username2>@<your_domain>.com` por el nombre de usuario (rol de**Administrador**).

    ```T-SQL
   CREATE TABLE dbo.Sales  
   (  
       OrderID INT,  
       SalesRep VARCHAR(60),  
       Product VARCHAR(10),  
       Quantity INT  
   );
    
   --Populate the table with 6 rows of data, showing 3 orders for each test user. 
   INSERT dbo.Sales (OrderID, SalesRep, Product, Quantity) VALUES
   (1, '<username1>@<your_domain>.com', 'Valve', 5),   
   (2, '<username1>@<your_domain>.com', 'Wheel', 2),   
   (3, '<username1>@<your_domain>.com', 'Valve', 4),  
   (4, '<username2>@<your_domain>.com', 'Bracket', 2),   
   (5, '<username2>@<your_domain>.com', 'Wheel', 5),   
   (6, '<username2>@<your_domain>.com', 'Seat', 5);  
    
   SELECT * FROM dbo.Sales;  
    ```

3. Use el botón **&#9655; Ejecutar** para ejecutar el script de SQL, que crea una nueva tabla llamada **Sales** en el esquema **dbo** del almacenamiento de datos.

4. A continuación, en el panel **Explorador**, expanda **Esquemas** > **dbo** > **Tablas** y compruebe que se ha creado la tabla **Sales**.
5. Cree un nuevo esquema, un predicado de seguridad definido como una función y una directiva de seguridad.  

    ```T-SQL
   --Create a separate schema to hold the row-level security objects (the predicate function and the security policy)
   CREATE SCHEMA rls;
   GO
   
   /*Create the security predicate defined as an inline table-valued function.
   A predicate evaluates to true (1) or false (0). This security predicate returns 1,
   meaning a row is accessible, when a row in the SalesRep column is the same as the user
   executing the query.*/   
   --Create a function to evaluate who is querying the table
   CREATE FUNCTION rls.fn_securitypredicate(@SalesRep AS VARCHAR(60)) 
       RETURNS TABLE  
   WITH SCHEMABINDING  
   AS  
       RETURN SELECT 1 AS fn_securitypredicate_result   
   WHERE @SalesRep = USER_NAME();
   GO   
   /*Create a security policy to invoke and enforce the function each time a query is run on the Sales table.
   The security policy has a filter predicate that silently filters the rows available to 
   read operations (SELECT, UPDATE, and DELETE). */
   CREATE SECURITY POLICY SalesFilter  
   ADD FILTER PREDICATE rls.fn_securitypredicate(SalesRep)   
   ON dbo.Sales  
   WITH (STATE = ON);
   GO
    ```

6. Utilice el botón **&#9655; Ejecutar** para ejecutar el script SQL
7. Después, en el panel **Explorador**, expanda **Esquemas** > **rls** > **Funciones** > **Funciones con valores de tabla** y compruebe que se ha creado la función.

    > **Nota**: Si se conecta con el usuario por el que ha reemplazado a `<username1>@<your_domain>.com` y ejecuta una instrucción `SELECT` en la tabla **Sales**, verá los siguientes resultados para la seguridad de nivel de fila.

    ![Recorte de pantalla de la tabla Sales con RLS.](./Images/rls-table.png)

## Implementación de la seguridad de nivel de columna

La seguridad de nivel de columna permite designar qué usuarios pueden acceder a columnas específicas de una tabla. Se implementa mediante la emisión de una instrucción `GRANT` o `DENY` en una tabla que especifica una lista de columnas y el usuario o rol que pueden o no pueden leerlas. Para simplificar la administración de acceso, asigne permisos a los roles en lugar de a usuarios individuales. En este ejercicio, creará una tabla, concederá acceso a un subconjunto de columnas de la tabla y comprobará que las columnas restringidas no las pueden ver otro usuario que no sea usted.

1. En el almacén que has creado en el ejercicio anterior, selecciona el desplegable **Nueva consulta SQL** y después **Nueva consulta SQL**.  

2. Cree una tabla e inserte datos en ella.

    ```T-SQL
   CREATE TABLE dbo.Orders
   (   
       OrderID INT,   
       CustomerID INT,  
       CreditCard VARCHAR(20)      
   );   
   INSERT dbo.Orders (OrderID, CustomerID, CreditCard) VALUES
   (1234, 5678, '111111111111111'),
   (2341, 6785, '222222222222222'),
   (3412, 7856, '333333333333333');   
   SELECT * FROM dbo.Orders;
    ```

3. Deniegue el permiso para ver una columna en la tabla. La instrucción T-SQL impide a `<username1>@<your_domain>.com` ver la columna CreditCard en la tabla Orders. En la instrucción `DENY`, reemplace `<username1>@<your_domain>.com` por el nombre de usuario de un usuario que tenga permisos de Espectador en el área de trabajo.

    ```T-SQL
   DENY SELECT ON dbo.Orders (CreditCard) TO [<username1>@<your_domain>.com];
    ```

    > **Nota**: Si se conecta con el usuario con el que ha reemplazado a `<username1>@<your_domain>.com` y ejecuta una instrucción `SELECT` en la tabla **Orders**, verá los siguientes resultados para la seguridad de nivel de columna.

    ![Recorte de pantalla de la consulta de la tabla Orders con errores.](./Images/cls-table.png)

    El error que se muestra en el recorte de pantalla se produce porque se ha restringido el acceso a la columna `CreditCard`. Si selecciona solo las columnas `OrderID` y `CustomerID`, la consulta se ejecuta correctamente.

## Configuración de permisos pormenorizados de SQL mediante T-SQL

Fabric tiene un modelo de permisos que permite controlar el acceso a los datos en el nivel de área de trabajo y en el nivel de elemento. Cuando necesite un control más granular de lo que los usuarios pueden hacer con los elementos protegibles en un almacén de Fabric, utilice los comandos estándar del lenguaje de control de datos SQL (DCL) `GRANT`, `DENY` y `REVOKE`. En este ejercicio, creará objetos, los protegerá mediante `GRANT` y `DENY` y, a continuación, ejecutará consultas para ver el efecto de aplicar permisos detallados.

1. En el almacén que creó en el ejercicio anterior, seleccione la lista desplegable **Nueva consulta SQL**. Seleccione **Nueva consulta SQL**.  

2. Creación de un procedimiento almacenado y una tabla. A continuación, ejecute el procedimiento y consulte la tabla.

    ```T-SQL
   CREATE PROCEDURE dbo.sp_PrintMessage
   AS
   PRINT 'Hello World.';
   GO   
   CREATE TABLE dbo.Parts
   (
       PartID INT,
       PartName VARCHAR(25)
   );
   
   INSERT dbo.Parts (PartID, PartName) VALUES
   (1234, 'Wheel'),
   (5678, 'Seat');
    GO
   
   /*Execute the stored procedure and select from the table and note the results you get
   as a member of the Workspace Admin role. Look for output from the stored procedure on 
   the 'Messages' tab.*/
   EXEC dbo.sp_PrintMessage;
   GO   
   SELECT * FROM dbo.Parts
    ```

3. A continuación, conceda los permisos `DENY SELECT` para la tabla a un usuario que sea miembro del rol **Espectador del área de trabajo** y los permisos `GRANT EXECUTE` para el procedimiento al mismo usuario. Reemplace `<username1>@<your_domain>.com` por el nombre de usuario de un usuario que tenga permisos de **Espectador** para el área de trabajo.

    ```T-SQL
   DENY SELECT on dbo.Parts to [<username1>@<your_domain>.com];

   GRANT EXECUTE on dbo.sp_PrintMessage to [<username1>@<your_domain>.com];
    ```

    > **Nota**: Si se conecta con el usuario con el que ha reemplazado a `<username1>@<your_domain>.com`, ejecute el procedimiento almacenado y ejecute una instrucción `SELECT` en la tabla **Parts**. Verá los siguientes resultados para permisos específicos.

    ![Recorte de pantalla de la consulta de la tabla Parts con errores.](./Images/grant-deny-table.png)

## Limpieza de recursos

En este ejercicio, ha aplicado reglas de enmascaramiento dinámico de datos a las columnas de una tabla, ha aplicado la seguridad de nivel de fila, ha implementado la seguridad de nivel de columna y ha configurado permisos detallados de SQL mediante T-SQL.

1. En la barra de navegación izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú de la barra de herramientas superior, selecciona **Configuración del área de trabajo**.
3. En la sección **General**, selecciona **Quitar esta área de trabajo**.
