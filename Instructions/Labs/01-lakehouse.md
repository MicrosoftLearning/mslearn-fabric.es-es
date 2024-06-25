---
lab:
  title: "Creación de un almacén de lago de Microsoft\_Fabric"
  module: Get started with lakehouses in Microsoft Fabric
---

# Crear un Lakehouse

Las soluciones de análisis de datos a gran escala se han creado tradicionalmente en torno a un *almacén de datos*, donde los datos se guardan en tablas relacionales y se consultan con el lenguaje SQL. El aumento de los "macrodatos" (caracterizados por los grandes *volúmenes*, la gran *variedad* y la alta *velocidad* de los nuevos recursos de datos), junto con la disponibilidad de tecnologías de proceso distribuido a escala de nube y almacenamiento de bajo costo, ha dado lugar a un enfoque alternativo para el almacenamiento de datos analíticos: el *lago de datos*. En un lago de datos, los datos se almacenan como archivos sin imponer un esquema fijo para el almacenamiento. Cada vez más, los ingenieros y analistas de datos buscan beneficiarse de las mejores características de ambos enfoques combinándolos en un *almacén de lago de datos*, donde los datos se almacenan en archivos en un lago de datos y se les aplica un esquema relacional en forma de capa de metadatos para poder consultarlos con la semántica SQL tradicional.

En Microsoft Fabric, un almacén de lago proporciona un almacenamiento de archivos altamente escalable en un almacén *OneLake* (basado en Azure Data Lake Store Gen2) con un metastore para objetos relacionales, como tablas y vistas, basados en el formato de tabla de código abierto *Delta Lake*. Delta Lake permite definir un esquema de tablas en un almacén de lago que se puede consultar con SQL.

Este laboratorio se realiza en unos **30** minutos.

> **Nota**: Necesitará una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. En la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com) en `https://app.fabric.microsoft.com`, seleccione **Ingeniería de datos de Synapse**.
1. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
1. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia en la sección **Avanzado** que incluya la capacidad de Fabric (*Prueba*, *Premium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

## Crear un almacén de lago

Ahora que tiene un área de trabajo, es el momento de crear un almacén de lago de datos para los archivos de datos.

1. En la página principal de **Ingeniería de datos de Synapse**, cree un nuevo **almacén de lago** con el nombre que prefiera.

    Después de un minuto o así, se habrá creado un nuevo almacén de lago:

    ![Captura de pantalla de un nuevo almacén de lago.](./Images/new-lakehouse.png)

1. Vea el nuevo almacén de lago y tenga en cuenta que el panel **Explorador del almacén de lago** de la izquierda le permite examinar las tablas y los archivos del almacén de lago:
    - La carpeta **Tablas** contiene tablas que puede consultar usando la semántica SQL. Las tablas de un almacén de lago de Microsoft Fabric se basan en el formato de archivo de *Delta Lake* de código abierto, que se usa habitualmente en Apache Spark.
    - La carpeta **Archivos** contiene archivos de datos del almacenamiento OneLake para el almacén de lago que no están asociados a tablas Delta administradas. También puede crear *accesos directos* en esta carpeta para hacer referencia a datos almacenados externamente.

    Actualmente, no hay tablas ni archivos en el almacén de lago.

## Cargar un archivo

Fabric proporciona varias formas de cargar datos en el almacén de lago, incluida la compatibilidad integrada con canalizaciones que copian datos de orígenes externos y flujos de datos (Gen 2) que puede definir con herramientas visuales basadas en Power Query. Sin embargo, una de las formas más sencillas de ingerir pequeñas cantidades de datos es cargar archivos o carpetas desde su PC local (o la máquina virtual del laboratorio si procede).

1. Descargue el archivo [sales.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv) de `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv`, guárdelo como **sales.csv** en su PC local (o máquina virtual del laboratorio si procede).

   > **Nota:** Para descargar el archivo, abra una nueva pestaña en el explorador y pegue la dirección URL. Haga clic con el botón derecho en cualquier parte de la página que contenga los datos y seleccione **Guardar como** para guardar la página como un archivo CSV.

2. Vuelva a la pestaña del explorador web que contiene el almacén de lago y, en el menú **...** de la carpeta **Archivos** del panel **Explorador del almacén de lago**, seleccione **Nueva subcarpeta** y cree una subcarpeta denominada **datos**.
3. En el menú **...** de la nueva carpeta **datos**, seleccione **Cargar**, **Cargar archivo** y cargue el archivo **sales.csv** desde su PC local (o máquina virtual del laboratorio si procede).
4. Una vez cargado el archivo, seleccione la carpeta **Archivos/datos** y compruebe que se ha cargado el archivo **sales.csv**, como se muestra aquí:

    ![Captura de pantalla del archivo sales.csv cargado en un almacén de lago.](./Images/uploaded-sales-file.png)

5. Seleccione el archivo **sales.csv** para ver una vista previa de su contenido.

## Exploración de accesos directos

En muchos casos, los datos con los que necesita trabajar en el almacén de lago pueden estar guardados en otra ubicación. Aunque hay muchas formas de ingerir datos en el almacenamiento OneLake del almacén de lago, otra opción es crear un *acceso directo*. Los accesos directos permiten incluir datos de orígenes externos en la solución de análisis sin la sobrecarga y el riesgo de incoherencia de los datos que supone copiarlos.

1. En el menú **...** de la carpeta **Archivos**, seleccione **Nuevo acceso directo**.
2. Vea los tipos de orígenes de datos disponibles para accesos directos. A continuación, cierre el cuadro de diálogo **Nuevo acceso directo** sin crear ninguno.

## Carga de datos de un archivo en una tabla

Los datos de ventas que cargó están en un archivo, con el que los analistas e ingenieros de datos pueden trabajar directamente usando código de Apache Spark. Sin embargo, en muchos casos es posible que desee cargar los datos del archivo en una tabla para poder consultarlos con SQL.

1. En la página **Inicio**, seleccione la carpeta **Archivos/datos** para ver el archivo **sales.csv** que contiene.
2. En el menú **...** del archivo **sales.csv**, seleccione **Cargar en tablas**.
3. En el cuadro de diálogo **Cargar en tabla**, establezca el nombre de la tabla en **sales** y confirme la operación de carga. Espere a que se cree y se cargue la tabla.

    > **Sugerencia:** Si la tabla **sales** no aparece automáticamente, en el menú **...** de la carpeta **Tablas**, seleccione **Actualizar**.

3. En el panel **Explorador del almacén de lago**, seleccione la tabla **sales** que se ha creado para ver los datos.

    ![Captura de pantalla de la vista previa de una tabla.](./Images/table-preview.png)

4. En el menú **...** de la tabla **sales**, seleccione **Ver archivos** para ver los archivos subyacentes de esta tabla.

    ![Captura de pantalla de la vista previa de una tabla.](./Images/delta-table-files.png)

    Los archivos de una tabla Delta se almacenan con el formato *Parquet* e incluyen una subcarpeta denominada **_delta_log**, en la que se registran los detalles de las transacciones aplicadas a la tabla.

## Uso de SQL para consultar tablas

Cuando se crea un almacén de lago y se definen tablas en él, se crea automáticamente un punto de conexión de SQL con el que se pueden consultar las tablas usando instrucciones SQL `SELECT`.

1. En la parte superior derecha de la página del almacén de lago, cambie de **Almacén de lago** a **Punto de conexión de análisis SQL**. Espere un momento hasta que se abra el punto de conexión de análisis SQL para el almacén de lago en una interfaz visual desde la que puede consultar las tablas.

2. Use el botón **Nueva consulta SQL** para abrir un nuevo editor de consultas y escriba la siguiente consulta SQL:

    ```sql
   SELECT Item, SUM(Quantity * UnitPrice) AS Revenue
   FROM sales
   GROUP BY Item
   ORDER BY Revenue DESC;
    ```
> **Nota**: Si está en una máquina virtual de laboratorio y tiene problemas para escribir la consulta SQL, puede descargar el archivo [01-Snippets.txt](https://github.com/MicrosoftLearning/mslearn-fabric/raw/main/Allfiles/Labs/01/Assets/01-Snippets.txt) de `https://github.com/MicrosoftLearning/mslearn-fabric/raw/main/Allfiles/Labs/01/Assets/01-Snippets.txt`, guardándolo en la máquina virtual. A continuación, puede copiar la consulta desde el archivo de texto.

3. Use el botón **&#9655; Ejecutar** para ejecutar la consulta y ver los resultados, que deben mostrar los ingresos totales por cada producto.

    ![Captura de pantalla de una consulta SQL con resultados.](./Images/sql-query.png)

## Creación de una consulta visual

Aunque muchos profesionales de los datos están familiarizados con SQL, los analistas de datos con experiencia en el uso de Power BI pueden aplicar sus conocimientos de Power Query para crear consultas visuales.

1. En la barra de herramientas, seleccione **Nueva consulta visual**.
2. Arrastre la tabla **sales** al nuevo panel de editor de consultas visuales que se abre para crear una consulta con Power Query como se muestra aquí: 

    ![Captura de pantalla de una consulta visual.](./Images/visual-query.png)

3. En el menú **Administrar columnas**, seleccione **Elegir columnas**. Seleccione únicamente las columnas **SalesOrderNumber** y **SalesOrderLineNumber**.

    ![Captura de pantalla del cuadro de diálogo Elegir columnas.](./Images/choose-columns.png)

4. En el menú **Transformar**, seleccione **Agrupar por**. Agrupe los datos usando la siguiente configuración **básica**:

    - **Agrupar por**: SalesOrderNumber
    - **Nuevo nombre de columna**: LineItems
    - **Operación**: Recuento de valores distintos
    - **Columna**: SalesOrderLineNumber

    Cuando haya terminado, el panel de resultados de la consulta visual mostrará el número de elementos de línea para cada pedido de ventas.

    ![Captura de pantalla de una consulta visual con resultados.](./Images/visual-query-results.png)

## Creación de un informe

Las tablas del almacén de lago se agregan automáticamente a un modelo semántico predeterminado para la generación de informes con Power BI.


1. En la parte inferior de la página Punto de conexión de SQL, seleccione la pestaña **Modelo**. Se muestra el esquema del modelo de datos del modelo semántico.

    ![Captura de pantalla 2024-04-29 155248](https://github.com/afelix-95/mslearn-fabric/assets/148110824/ba9bd67d-8968-4c46-ac7a-f96a9f697f4c)

    > **Nota 1**: En este ejercicio, el modelo semántico consta de una sola tabla. En un escenario real, es probable que cree varias tablas en el almacén de lago, cada una de las cuales se incluiría en el modelo. Después, podría definir relaciones entre estas tablas en el modelo.
    
    > **Nota 2**: Las vistas `frequently_run_queries`, `long_running_queries`, `exec_sessions_history` y `exec_requests_history` forman parte del esquema de `queryinsights` creado automáticamente por Fabric. Es una característica que proporciona una vista holística de la actividad de consulta histórica en el punto de conexión de análisis SQL. Dado que esta característica está fuera del ámbito de este ejercicio, esas vistas deben omitirse por ahora.

2. En la cinta de menús, seleccione la pestaña **Informes** y elija **Nuevo informe**. Se abre una nueva pestaña del explorador en la que puede diseñar el informe.

    ![Captura de pantalla del diseñador de informes.](./Images/report-designer.png)

3. En el panel **Datos** de la derecha, expanda la tabla **sales**. Seleccione los siguientes campos:
    - **Elemento**
    - **Cantidad**

    Se agrega una visualización de tabla al informe:

    ![Captura de pantalla de un informe que contiene una tabla.](./Images/table-visualization.png)

4. Oculte los paneles **Datos** y **Filtros** para crear más espacio. Asegúrese de que está seleccionada la visualización de tabla y, en el panel **Visualizaciones**, cambie la visualización a un **Gráfico de barras agrupadas** y ajústele el tamaño como se muestra aquí.

    ![Captura de pantalla de un informe que contiene un gráfico de barras agrupadas.](./Images/clustered-bar-chart.png)

5. En el menú **Archivo**, seleccione **Guardar**. A continuación, guarde el informe como **Informe de ventas de artículos** en el área de trabajo que creó antes.
6. Cierre la pestaña del explorador que contiene el informe para volver al punto de conexión SQL del almacén de lago. Ahora, en la barra de menús de la izquierda, seleccione el área de trabajo para comprobar que contiene los siguientes elementos:
    - Almacén de lago.
    - Punto de conexión de análisis SQL del almacén de lago.
    - Modelo semántico predeterminado para las tablas del almacén de lago.
    - **Informe de ventas de artículos**.

## Limpieza de recursos

En este ejercicio, ha creado un almacén de lago y ha importado datos en él. Ha visto que un almacén de lago consta de tablas y archivos guardados en un almacén de datos OneLake. Las tablas administradas se pueden consultar mediante SQL y se incluyen en un modelo semántico predeterminado para admitir visualizaciones de datos.

Si ha terminado de explorar el almacén de lago, puede eliminar el área de trabajo que ha creado para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **General**, seleccione **Quitar esta área de trabajo**.
