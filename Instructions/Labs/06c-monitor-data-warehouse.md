---
lab:
  title: Supervisión de almacenamientos de datos en Microsoft Fabric
  module: Monitor a data warehouse in Microsoft Fabric
---

# Supervisión de almacenamientos de datos en Microsoft Fabric

En Microsoft Fabric, un almacenamiento de datos proporciona una base de datos relacional para análisis a gran escala. En Microsoft Fabric, los almacenes de datos incluyen vistas de administración dinámica que puede usar para supervisar la actividad y las consultas.

Este laboratorio se tarda aproximadamente **30** minutos en completarse.

> **Nota**: Necesitará una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. En la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com), seleccione **Synapse Data Warehouse**.
1. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
1. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

## Creación de un almacenamiento de datos de ejemplo

Ahora que tiene un área de trabajo, es el momento de crear un almacenamiento de datos.

1. En la parte inferior izquierda, asegúrese de que está seleccionada la experiencia **Data Warehouse**.
1. En la página **Inicio**, seleccione **Almacenamiento de ejemplo** y cree un almacenamiento de datos denominado **sample-dw**.

    Aproximadamente un minuto después se creará un almacén, que se rellenará con datos de ejemplo para un escenario de análisis de carreras de taxi.

    ![Captura de pantalla de un nuevo almacenamiento.](./Images/sample-data-warehouse.png)

## Exploración de vistas de administración dinámica

Los almacenamientos de datos de Microsoft Fabric incluyen vistas de administración dinámica (DMV), que se pueden usar para identificar la actividad actual en la instancia del almacenamiento de datos.

1. En la página del almacenamiento de datos **sample-dw**, en la lista desplegable **Nueva consulta SQL**, seleccione **Nueva consulta SQL**.
1. En el nuevo panel de consulta en blanco, escriba el siguiente código de Transact-SQL para consultar la DMV de **sys.dm_exec_connections**:

    ```sql
   SELECT * FROM sys.dm_exec_connections;
    ```

1. Use el botón **&#9655; Ejecutar** para ejecutar el script de SQL y ver los resultados, que incluyen detalles de todas las conexiones al almacenamiento de datos.
1. Modifique el código SQL para consultar la vista de administración dinámica **sys.dm_exec_sessions** DMV tal como se indica a continuación:

    ```sql
   SELECT * FROM sys.dm_exec_sessions;
    ```

1. Ejecute la consulta modificada y vea los resultados, que muestran los detalles de todas las sesiones autenticadas.
1. Modifique el código SQL para consultar la vista de administración dinámica **sys.dm_exec_requests** DMV tal como se indica a continuación:

    ```sql
   SELECT * FROM sys.dm_exec_requests;
    ```

1. Ejecute la consulta modificada y vea los resultados, que muestran los detalles de todas las solicitudes que se ejecutan en el almacenamiento de datos.
1. Modifique el código SQL para combinar las vistas de administración dinámica y devolver información sobre las solicitudes que se están ejecutando actualmente en la misma base de datos, tal como se indica a continuación:

    ```sql
   SELECT connections.connection_id,
    sessions.session_id, sessions.login_name, sessions.login_time,
    requests.command, requests.start_time, requests.total_elapsed_time
   FROM sys.dm_exec_connections AS connections
   INNER JOIN sys.dm_exec_sessions AS sessions
       ON connections.session_id=sessions.session_id
   INNER JOIN sys.dm_exec_requests AS requests
       ON requests.session_id = sessions.session_id
   WHERE requests.status = 'running'
       AND requests.database_id = DB_ID()
   ORDER BY requests.total_elapsed_time DESC;
    ```

1. Ejecute la consulta modificada y vea los resultados, que muestran los detalles de todas las consultas en ejecución en la base de datos (incluida esta).
1. En la lista desplegable **Nueva consulta SQL**, seleccione **Nueva consulta SQL** para agregar una segunda pestaña de consulta. Luego, en la nueva pestaña de consulta vacía, ejecute el siguiente código:

    ```sql
   WHILE 1 = 1
       SELECT * FROM Trip;
    ```

1. Deje la consulta en ejecución y vuelva a la pestaña que contiene el código para consultar las vistas de administración dinámica y volver a ejecutarlo. Esta vez, los resultados deben incluir la segunda consulta que se ejecuta en la otra pestaña. Anote el tiempo transcurrido para esa consulta.
1. Espere unos segundos y vuelva a ejecutar el código para consultar las vistas de administración dinámica. El tiempo transcurrido para la consulta en la otra pestaña debería haber aumentado.
1. Vuelva a la segunda pestaña de consulta, donde se sigue ejecutando la consulta, y seleccione **X Cancelar** para cancelarla.
1. Vuelva a la pestaña con el código para consultar las vistas de administración dinámica, vuelva a ejecutar la consulta para confirmar que la segunda consulta ha dejado de ejecutarse.
1. Cierre todas las pestañas de consulta.

> **Información adicional**: Para más información sobre el uso de las vistas de administración dinámica, consulte [Supervisión de conexiones, sesiones y solicitudes mediante DMV](https://learn.microsoft.com/fabric/data-warehouse/monitor-using-dmv) en la documentación de Microsoft Fabric.

## Exploración de información de consultas

Los almacenamientos de datos de Microsoft Fabric proporcionan *información de consulta*: un conjunto especial de vistas que proporcionan detalles sobre las consultas que se ejecutan en el almacenamiento de datos.

1. En la página del almacenamiento de datos **sample-dw**, en la lista desplegable **Nueva consulta SQL**, seleccione **Nueva consulta SQL**.
1. En el nuevo panel de consulta en blanco, escriba el siguiente código de Transact-SQL para consultar la vista **exec_requests_history**:

    ```sql
   SELECT * FROM queryinsights.exec_requests_history;
    ```

1. Use el botón **&#9655; Ejecutar** para ejecutar el script SQL y ver los resultados, que incluyen detalles de las consultas ejecutadas anteriormente.
1. Modifique el código de SQL para consultar la vista **frequently_run_queries** como se indica a continuación:

    ```sql
   SELECT * FROM queryinsights.frequently_run_queries;
    ```

1. Ejecute la consulta modificada y vea los resultados, que muestran los detalles de las consultas que se ejecutan con frecuencia.
1. Modifique el código de SQL para consultar la vista **long_running_queries** como se indica a continuación:

    ```sql
   SELECT * FROM queryinsights.long_running_queries;
    ```

1. Ejecute la consulta modificada y vea los resultados, que muestran los detalles de todas las consultas y sus duraciones.

> **Información adicional**: Para más información sobre el uso de información de consultas, consulte [Información de consultas en el almacenamiento de datos de Fabric](https://learn.microsoft.com/fabric/data-warehouse/query-insights) en la documentación de Microsoft Fabric.


## Limpieza de recursos

En este ejercicio, ha usado vistas de administración dinámica e información de consultas para supervisar la actividad de un almacenamiento de datos de Microsoft Fabric.

Si ha terminado de explorar el almacenamiento de datos, puede eliminar el área de trabajo que creó para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **General**, seleccione **Quitar esta área de trabajo**.
