---
lab:
  title: "Creación y uso de flujos de datos (Gen2) en Microsoft\_Fabric"
  module: Ingest Data with Dataflows Gen2 in Microsoft Fabric
---

# Creación y uso de un flujo de datos (Gen2) en Microsoft Fabric

En Microsoft Fabric, los flujos de datos (Gen2) se conectan a varios orígenes de datos y realizan transformaciones en Power Query Online. Luego, se pueden usar en canalizaciones de datos para ingerir datos en un almacén de lago u otro almacén analítico, o para definir un conjunto de datos para un informe de Power BI.

Este laboratorio está diseñado para introducir los distintos elementos de los flujos de datos (Gen2) y no para crear una solución compleja que pueda existir en una empresa. Este laboratorio se realiza en **30 minutos aproximadamente**.

> **Nota**: Necesitará una licencia de Microsoft Fabric para realizar este ejercicio. Consulte [Introducción a Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para más información sobre cómo habilitar una licencia de prueba de Fabric gratuita. Para ello, necesitará una cuenta *profesional* o *educativa* de Microsoft. Si no tiene una, puede [registrarse para una evaluación gratuita de Microsoft Office 365 E3 o superior](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Crear un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. Inicie sesión en [Microsoft Fabric](https://app.fabric.microsoft.com) en `https://app.fabric.microsoft.com` y seleccione **Power BI**.
2. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
3. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Versión de prueba*, *Premium* o *Fabric*).
4. Cuando se abra la nueva área de trabajo, estará vacía, como se muestra aquí:

    ![Captura de pantalla de un área de trabajo vacía en Power BI.](./Images/new-workspace.png)

## Creación de un almacén de lago

Ahora que tiene un área de trabajo, es el momento de cambiar a la experiencia **Ingeniería de datos** en el portal y crear un almacén de lago de datos en el que ingerir datos.

1. En la parte inferior izquierda del portal de Power BI, seleccione el icono de **Power BI** y cambie a la experiencia **Ingeniería de datos**.

2. En la página principal de **Ingeniería de datos**, cree un nuevo **almacén de lago** con el nombre que prefiera.

    Al cabo de un minuto más o menos, se creará un nuevo almacén de lago vacío.

 ![Nuevo almacén de lago.](./Images/new-lakehouse.png)

## Creación de un flujo de datos (Gen2) para ingerir datos

Ahora que tiene un almacén de lago, debe ingerir en él algunos datos. Una manera de hacerlo es definir un flujo de datos que encapsula un proceso de *extracción, transformación y carga* (ETL).

1. En la página principal del área de trabajo, seleccione **Nuevo flujo de datos Gen2**. Al cabo de unos segundos, se abre el Editor de Power Query para el nuevo flujo de datos, como se muestra aquí.

 ![Nuevo flujo de datos.](./Images/new-dataflow.png)

2. Seleccione **Importar desde un archivo de texto o CSV** y cree un nuevo origen de datos con la siguiente configuración:
 - **Vínculo al archivo**: *Seleccionado*.
 - **URL o ruta del archivo**: `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/orders.csv`.
 - **Conexión**: Crear nueva conexión.
 - **Puerta de enlace de datos**: (ninguna).
 - **Tipo de autenticación**: Anónima.

3. Seleccione **Siguiente** para obtener una vista previa de los datos del archivo y, luego **Crear** para crear el origen de datos. El Editor de Power Query muestra el origen de datos y un conjunto inicial de pasos de consulta para dar formato a los datos, como se muestra aquí:

 ![Consulta en el Editor de Power Query.](./Images/power-query.png)

4. En la cinta de opciones de la barra de herramientas, seleccione la pestaña **Agregar columna**. A continuación, seleccione **Columna personalizada** y cree una columna llamada **MonthNo** que contenga un número basado en la fórmula `Date.Month([OrderDate])`, como se muestra aquí:

 ![Columna personalizada en el Editor de Power Query.](./Images/custom-column.png)

 El paso para agregar la columna personalizada se agrega a la consulta y la columna resultante se muestra en el panel de datos:

 ![Consulta con un paso de columna personalizado.](./Images/custom-column-added.png)

> **Sugerencia:** En el panel "Configuración de la consulta" del lado derecho, observe que los **pasos aplicados** incluyen cada paso de transformación. En la parte inferior, también puede alternar el botón **Flujo de diagrama** para activar el diagrama visual de los pasos.
>
> Los pasos se pueden mover hacia arriba o hacia abajo, se pueden editar seleccionando el icono de engranaje y puede seleccionar cada paso para ver las transformaciones que se aplican en el panel de vista previa.

## Adición de un destino de datos al flujo de datos

1. En la cinta de opciones de la barra de herramientas, seleccione la pestaña **Inicio**. A continuación, en el menú desplegable **Agregar destino de datos**, seleccione **Almacén de lago**.

   > **Nota:** Si esta opción está atenuada, es posible que ya tenga un conjunto de destino de datos. Compruebe el destino de los datos en la parte inferior del panel "Configuración de la consulta" en el lado derecho del Editor de Power Query. Si ya se ha establecido un destino, puede cambiarlo con el engranaje.

2. En el cuadro de diálogo **Conectarse al destino de datos**, edite la conexión e inicie sesión con su cuenta organizativa de Power BI para establecer la identidad que usa el flujo de datos para acceder al almacén de lago.

 ![Página de configuración del destino de datos.](./Images/dataflow-connection.png)

3. Seleccione **Siguiente** y, en la lista de áreas de trabajo disponibles, busque el área de trabajo y seleccione el almacén de lago que creó al principio de este ejercicio. A continuación, especifique una nueva tabla llamada **orders**:

   ![Página de configuración del destino de datos.](./Images/data-destination-target.png)

   > **Nota:** En la página **Configuración de destino**, observe cómo OrderDate y MonthNo no están seleccionadas en la asignación de columnas y hay un mensaje informativo: *Cambiar a fecha y hora*.

   ![Página de configuración del destino de datos.](./Images/destination-settings.png)

1. Cancele esta acción y vuelva a las columnas OrderDate y MonthNo en Power Query en línea. Haga clic con el botón derecho en el encabezado de columna y en **Cambiar tipo**.

    - OrderDate = Fecha y hora
    - MonthNo = Número entero

1. Ahora repita el proceso descrito anteriormente para agregar un destino de almacén de lago.

8. En la página **Configuración de destino**, seleccione **Anexar** y, luego, guarde la configuración.  El destino **Almacén de lago** se indica con un icono en la consulta en el Editor de Power Query.

   ![Consulta con un destino de almacén de lago.](./Images/lakehouse-destination.png)

9. Seleccione **Publicar** para publicar el flujo de datos. A continuación, espere a que se cree el flujo de datos **Dataflow 1** en el área de trabajo.

1. Una vez publicado, puede hacer clic con el botón derecho en el flujo de datos del área de trabajo, seleccionar **Propiedades** y cambiar el nombre del flujo de datos.

## Adición de un flujo de datos a una canalización

Puede incluir un flujo de datos como actividad en una canalización. Las canalizaciones se usan para orquestar las actividades de ingesta y procesamiento de datos, lo que permite combinar flujos de datos con otros tipos de operaciones en un único proceso programado. Se pueden crear canalizaciones en unas cuantas experiencias diferentes, incluida la experiencia Data Factory.

1. En el área de trabajo habilitada para Fabric, asegúrese de que todavía está en la experiencia **Ingeniería de datos**. Seleccione **Nueva**, **Canalización de datos** y, cuando se le solicite, cree una canalización llamada **Cargar datos**.

   Se abre el editor de canalizaciones.

   ![Canalización de datos vacía.](./Images/new-pipeline.png)

   > **Sugerencia**: Si el Asistente para copiar datos se abre automáticamente, ciérralo.

2. Seleccione **Agregar actividad de canalización** y agregue una actividad **Flujo de datos** a la canalización.

3. Con la nueva actividad **Dataflow1** seleccionada, en la pestaña **Configuración**, en la lista desplegable **Flujo de datos**, seleccione **Dataflow 1** (el flujo de datos que creó anteriormente).

   ![Canalización con una actividad de flujo de datos.](./Images/dataflow-activity.png)

4. En la pestaña **Inicio**, guarde la canalización con el icono **&#128427;** (*Guardar*).
5. Use el botón **&#9655; Ejecutar** para ejecutar la canalización y espere a que se complete. Esto puede tardar unos minutos.

   ![Canalización con un flujo de datos que se ha completado correctamente.](./Images/dataflow-pipeline-succeeded.png)

6. En la barra de menús del borde izquierdo, seleccione su almacén de lago.
7. En el menú **...** de **Tablas**, seleccione **Actualizar**. A continuación, expanda **Tablas** y seleccione la tabla **orders**, que ha creado el flujo de datos.

   ![Tabla cargada por un flujo de datos.](./Images/loaded-table.png)

> **Sugerencia**: Use el *conector de flujos de datos* de Power BI Desktop para conectarse directamente a las transformaciones de datos realizadas con el flujo de datos.
>
> También puede realizar transformaciones adicionales, publicarlas como un nuevo conjunto de datos y distribuirlas con la audiencia prevista en el caso de conjuntos de datos especializados.
>
>![Conectores de orígenes de datos de Power BI](Images/pbid-dataflow-connectors.png)

## Limpieza de recursos

Si ha terminado de explorar flujos de datos en Microsoft Fabric, puede eliminar el área de trabajo que creó para este ejercicio.

1. Vaya a Microsoft Fabric en el explorador.
1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
1. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
1. En la sección **Otros**, seleccione **Quitar esta área de trabajo**.
1. No guarde los cambios en Power BI Desktop ni elimine el archivo .pbix si ya está guardado.
