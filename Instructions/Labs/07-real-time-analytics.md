---
lab:
  title: "Introducción al análisis en tiempo real en Microsoft\_Fabric"
  module: Get started with real-time analytics in Microsoft Fabric
---

# Introducción al análisis en tiempo real en Microsoft Fabric

Microsoft Fabric proporciona un entorno de ejecución que puede usar para almacenar y consultar datos con el Lenguaje de consulta Kusto (KQL). Kusto está optimizado para datos que incluyen un componente de serie temporal, como los datos en tiempo real de archivos de registro o dispositivos IoT.

Este laboratorio se realiza en unos **30** minutos.

> **Nota:** Necesitará una licencia de Microsoft Fabric para realizar este ejercicio. Consulte [Introducción a Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para obtener más información sobre cómo habilitar una licencia de evaluación de Fabric gratuita. Para hacerlo, necesitará una cuenta *profesional* o *educativa* de Microsoft. Si no tiene una, puede [registrarse para obtener una evaluación gratuita de Microsoft Office 365 E3 o superior](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Crear un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. Inicie sesión en [Microsoft Fabric](https://app.fabric.microsoft.com) en `https://app.fabric.microsoft.com` y seleccione **Power BI**.
2. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
3. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
4. Cuando se abra la nueva área de trabajo, debe estar vacía, como se muestra aquí:

    ![Captura de pantalla de un área de trabajo vacía en Power BI.](./Images/new-workspace.png)

5. En la parte inferior izquierda del portal de Power BI, seleccione el icono de **Power BI** y cambie a la experiencia **Microsoft Fabric**.

## Descarga de un archivo para la base de datos KQL

Ahora que tiene un área de trabajo, es el momento de cambiar a la experiencia *Análisis en tiempo real de Synapse* en el portal y descargar el archivo de datos que va a analizar.

1. Descargue el archivo de datos para este ejercicio desde [https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv) y guárdelo como **sales.csv** en su PC local (o su máquina virtual del laboratorio si procede).
2. Vuelva a la ventana del explorador con la experiencia **Microsoft Fabric**.

## Creación de una base de datos KQL

El Lenguaje de consulta Kusto (KQL) se usa para consultar datos estáticos o de streaming en una tabla que se define en una base de datos KQL. Para analizar los datos de ventas, debe crear una tabla en una base de datos KQL e ingerir los datos del archivo.

1. En el portal de la experiencia **Microsoft Fabric**, seleccione la imagen de la experiencia **Análisis en tiempo real de Synapse**, como se muestra aquí:

    ![Captura de pantalla de la página Inicio de La experiencia Microsoft Fabric con RTA seleccionado](./Images/fabric-experience-home.png)

2. En la página **Inicio** de la experiencia **Análisis en tiempo real**, seleccione **Base de datos KQL** y cree una nueva base de datos con el nombre que prefiera.
3. Cuando se haya creado la nueva base de datos, seleccione la opción para obtener datos de un **Archivo local**. A continuación, use el asistente para importar los datos en una nueva tabla. Seleccione las siguientes opciones:
    - **Destino**:
        - **Base de datos**: *La base de datos que ha creado ya está seleccionada*.
        - **Tabla**: *Cree una tabla denominada* **sales**.
    - **Origen**:
        - **Tipo de origen**: archivo
        - **Cargar archivos**: *Arrastre o busque el archivo que descargó antes*.
    - **Esquema**:
        - **Tipo de compresión**: sin comprimir
        - **Formato de datos**: CSV
        - **Omitir el primer registro**: *Seleccionado*.
        - **Nombre de asignación**: sales_mapping.
    - **Resumen**:
        - *Revise la vista previa de la tabla y cierre el asistente.*

> **Nota:** En este ejemplo, ha importado una cantidad muy pequeña de datos estáticos de un archivo, que está bien para los fines de este ejercicio. En realidad, Kusto se puede usar para analizar volúmenes de datos más grandes, incluidos datos en tiempo real de un origen de streaming como Azure Event Hubs.

## Uso de KQL para consultar la tabla "sales"

Ahora que tiene una tabla de datos en la base de datos, puede usar código KQL para consultarla.

1. Asegúrese de que tiene resaltada la tabla **sales**. En la barra de menús, seleccione la lista desplegable **Tabla de consultas** y seleccione **Mostrar 100 registros cualesquiera**.

2. Se abre un nuevo panel con la consulta y el resultado. 

3. Modifique la consulta del siguiente modo:

    ```kusto
   sales
   | where Item == 'Road-250 Black, 48'
    ```

4. Ejecuta la consulta. A continuación, revise los resultados, que deben contener solo las filas de los pedidos de ventas del producto *Road-250 Black, 48*.

5. Modifique la consulta del siguiente modo:

    ```kusto
   sales
   | where Item == 'Road-250 Black, 48'
   | where datetime_part('year', OrderDate) > 2020
    ```

6. Ejecute la consulta y revise los resultados, que solo deben contener los pedidos de ventas de *Road-250 Black, 48* realizados después de 2020.

7. Modifique la consulta del siguiente modo:

    ```kusto
   sales
   | where OrderDate between (datetime(2020-01-01 00:00:00) .. datetime(2020-12-31 23:59:59))
   | summarize TotalNetRevenue = sum(UnitPrice) by Item
   | sort by Item asc
    ```

8. Ejecute la consulta y revise los resultados, que deben contener los ingresos netos totales de cada producto entre el 1 de enero y el 31 de diciembre de 2020 por nombre de producto en orden ascendente.
9. Seleccione **Guardar como conjunto de consultas KQL** y guarde la consulta como **Ingresos por producto**.

## Creación de un informe de Power BI a partir de un conjunto de consultas KQL

Puede usar el conjunto de consultas KQL como base para un informe de Power BI.

1. En el editor del workbench del conjunto de consultas, ejecute la consulta y espere los resultados.
2. Seleccione **Crear informe de Power BI** y espere a que se abra el editor de informes.
3. En el editor de informes, en el panel **Datos**, expanda **Resultado de la consulta de Kusto** y seleccione los campos **Item** y **TotalRevenue**.
4. En el lienzo de diseño del informe, seleccione la visualización de tabla que se ha agregado y, en el panel **Visualizaciones**, seleccione **Gráfico de barras agrupadas**.

    ![Captura de pantalla de un informe de una consulta KQL.](./Images/kql-report.png)

5. En la ventana de **Power BI**, en el menú **Archivo**, seleccione **Guardar**. A continuación, guarde el informe como **Ingresos por artículo.pbix** en el área de trabajo donde se han definido el almacén de lago y la base de datos KQL con una etiqueta de confidencialidad **No empresarial**.
6. Cierre la ventana de **Power BI** y, en la barra de la izquierda, seleccione el icono del área de trabajo.

    Actualice la página Área de trabajo si es necesario para ver todos los elementos que contiene.

7. En la lista de elementos del área de trabajo, observe que aparece el informe **Ingresos por artículo**.

## Limpieza de recursos

En este ejercicio, ha creado un almacén de lago, una base de datos KQL para analizar los datos cargados en el almacén de lago. Ha usado KQL para consultar los datos y crear un conjunto de consultas, que luego ha usado para crear un informe de Power BI.

Si ha terminado de explorar la base de datos KQL, puede eliminar el área de trabajo que ha creado para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **Otros**, seleccione **Quitar esta área de trabajo**.
