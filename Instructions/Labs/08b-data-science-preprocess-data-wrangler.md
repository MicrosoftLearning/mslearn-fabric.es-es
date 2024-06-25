---
lab:
  title: "Procesamiento previo de datos con Data Wrangler en Microsoft\_Fabric"
  module: Preprocess data with Data Wrangler in Microsoft Fabric
---

# Procesamiento previo de datos con Data Wrangler en Microsoft Fabric

En este laboratorio, aprenderá a usar Data Wrangler en Microsoft Fabric para procesar previamente datos y generar código mediante una biblioteca de operaciones comunes de ciencia de datos.

Este laboratorio se tarda aproximadamente **30** minutos en completarse.

> **Nota**: Necesitará una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. En un explorador, vaya a la página principal de Microsoft Fabric en `https://app.fabric.microsoft.com` y, si fuera necesario, inicie sesión con sus credenciales de Fabric.
1. En la página principal de Fabric, seleccione **Ciencia de datos de Synapse**.
1. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
1. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

## Creación de un cuaderno

Para entrenar un modelo, puede crear un *cuaderno*. Los cuadernos proporcionan un entorno interactivo en el que puede escribir y ejecutar código (en varios lenguajes) como *experimentos*.

1. En la página principal de **Ciencia de datos de Synapse**, cree un nuevo **cuaderno**.

    Al cabo de unos segundos, se abrirá un nuevo cuaderno que contiene una sola *celda*. Los cuadernos se componen de una o varias celdas que pueden contener *código* o *Markdown* (texto con formato).

1. Seleccione la primera celda (que actualmente es una celda de *código* ) y, luego, en la barra de herramientas dinámica de su parte superior derecha, use el botón **M&#8595;** para convertir la celda en una celda de *Markdown*.

    Cuando la celda cambie a una celda de Markdown, se representará el texto que contiene.

1. Si fuera necesario, use el botón **&#128393;** (Editar) para cambiar la celda al modo de edición y, después, elimine el contenido y escriba el siguiente texto:

    ```text
   # Perform data exploration for data science

   Use the code in this notebook to perform data exploration for data science.
    ```

## Carga de datos en un objeto DataFrame

Ahora está listo para ejecutar código para obtener datos. Trabajará con el [**conjunto de datos de OJ Sales**](https://learn.microsoft.com/en-us/azure/open-datasets/dataset-oj-sales-simulated?tabs=azureml-opendatasets?azure-portal=true) de Azure Open Datasets. Después de cargar los datos, convertirá los datos en un dataframe de Pandas, que es la estructura compatible con Data Wrangler.

1. En el cuaderno, use el icono **+ Código** situado debajo de la celda más reciente para agregar una nueva celda de código al cuaderno.

    > **Sugerencia**: Para ver el icono **+ Código**, mueva el ratón hasta justo debajo y a la izquierda de la salida de la celda actual. Como alternativa, en la barra de menús, en la pestaña **Editar**, seleccione **+ Añadir celda de código**.

1. Escriba el código siguiente para cargar el conjunto de datos en una trama de datos.

    ```python
   # Azure storage access info for open dataset diabetes
   blob_account_name = "azureopendatastorage"
   blob_container_name = "ojsales-simulatedcontainer"
   blob_relative_path = "oj_sales_data"
   blob_sas_token = r"" # Blank since container is Anonymous access
    
   # Set Spark config to access  blob storage
   wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
   spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
   print("Remote blob path: " + wasbs_path)
    
   # Spark reads csv
   df = spark.read.csv(wasbs_path, header=True)
    ```

1. Use el botón **&#9655; Ejecutar celda** situado a la izquierda de la celda para ejecutarla. Como alternativa, puede presionar `SHIFT` + `ENTER` en el teclado para ejecutar una celda.

    > **Nota**: Dado que esta es la primera vez que ha ejecutado código de Spark en esta sesión, se debe iniciar el grupo de Spark. Esto significa que la primera ejecución de la sesión puede tardar un minuto o así en completarse. Las ejecuciones posteriores serán más rápidas.

1. Use el icono **+Código** debajo de la salida de la celda para agregar una nueva celda de código al cuaderno y escriba en ella el código siguiente:

    ```python
   import pandas as pd

   df = df.toPandas()
   df = df.sample(n=500, random_state=1)
    
   df['WeekStarting'] = pd.to_datetime(df['WeekStarting'])
   df['Quantity'] = df['Quantity'].astype('int')
   df['Advert'] = df['Advert'].astype('int')
   df['Price'] = df['Price'].astype('float')
   df['Revenue'] = df['Revenue'].astype('float')
    
   df = df.reset_index(drop=True)
   df.head(4)
    ```

1. Cuando se haya completado el comando de la celda, revise la salida que aparece debajo de ella, que será algo parecido a esto:

    |   |WeekStarting|Tienda|Marca|Quantity|Anuncio|Price|Ingresos|
    |---|---|---|---|---|---|---|---|
    |0|1991-10-17|947|minute.maid|13306|1|2,42|32200.52|
    |1|1992-03-26|1293|dominicks|18596|1|1,94|36076.24|
    |2|1991-08-15|2278|dominicks|17457|1|2.14|37357.98|
    |3|1992-09-03|2175|tropicana|9652|1|2,07|19979.64|
    |...|...|...|...|...|...|...|...|

    La salida muestra las cuatro primeras filas del conjunto de datos de OJ Sales.

## Visualización de estadísticas de resumen

Ahora que hemos cargado los datos, el siguiente paso consiste en preprocesarlos mediante Data Wrangler. El preprocesamiento es un paso fundamental en cualquier flujo de trabajo de aprendizaje automático. Implica limpiar los datos y transformarlos en un formato que se pueda introducir en un modelo de Machine Learning.

1. Seleccione **Datos** en la cinta de opciones del cuaderno y, a continuación, seleccione **Iniciar Data Wrangler**.

1. Seleccione el conjunto de datos `df`. Cuando se inicia Data Wrangler, se genera una introducción descriptiva del dataframe en el panel **Resumen**.

1. Seleccione la función **Ingresos** y observe la distribución de datos de esta función.

1. Revise los detalles del panel lateral **Resumen** y observe los valores de las estadísticas.

    ![Captura de pantalla de la página Data Wrangler que muestra los detalles del panel de resumen.](./Images/data-wrangler-summary.png)

    ¿Cuáles son algunas de las conclusiones que se pueden extraer? Los ingresos promedio son de, aproximadamente, **33 459,54 $**, con una desviación estándar de **8032,23 $**. Esto sugiere que los valores de los ingresos se reparten en un intervalo de, aproximadamente, **8032,23 USD** con respecto a la media.

## Aplicación de formato a los datos de texto

Ahora vamos a aplicar algunas transformaciones a la característica **Marca**.

1. En el panel **Data Wrangler**, seleccione la característica `Brand` en la cuadrícula.

1. Vaya al panel **Operaciones**, expanda **Buscar y reemplazar** y, a continuación, seleccione **Buscar y reemplazar**.

1. En el panel **Buscar y reemplazar**, cambie las propiedades siguientes:

    - **Valor anterior:** "`.`"
    - **Nuevo valor:** "` `" (carácter de espacio)

    Puede ver los resultados de la operación en vista previa automática en la cuadrícula de presentación.

1. Seleccione **Aplicar**.

1. Vuelva al panel **Operaciones** y expanda **Formato**.

1. Seleccione **Poner en mayúsculas el primer carácter**. Active el botón de alternancia **Poner en mayúscula todas las palabras** y, a continuación, seleccione **Aplicar**.

1. Seleccione **Agregar código al cuaderno**. Además, también puede copiar el código y guardar el conjunto de datos transformado como un archivo CSV.

    >**Nota:** El código se copia automáticamente en la celda del cuaderno y está listo para su uso.

1. Reemplace las líneas 10 y 11 por el código `df = clean_data(df)`, ya que el código generado en Data Wrangler no sobrescribe el dataframe original. El bloque de código final debería ser similar al siguiente:

    ```python
   def clean_data(df):
       # Replace all instances of "." with " " in column: 'Brand'
       df['Brand'] = df['Brand'].str.replace(".", " ", case=False, regex=False)
       # Capitalize the first character in column: 'Brand'
       df['Brand'] = df['Brand'].str.title()
       return df
    
   df = clean_data(df)
    ```

1. Ejecute la celda de código y compruebe la variable `Brand`.

    ```python
   df['Brand'].unique()
    ```

    El resultado debe mostrar los valores *Minute Maid*, *Dominicks* y *Tropicana*.

Ha aprendido a manipular gráficamente los datos de texto y a generar fácilmente código mediante Data Wrangler.

## Aplicación de transformación de codificación one-hot

Ahora, generaremos el código para aplicar la transformación de codificación one-hot a nuestros datos como parte de nuestros pasos de preprocesamiento. Para que nuestro escenario sea más práctico, empezamos generando algunos datos de ejemplo. Esto nos permite simular una situación real y nos proporciona una característica que se puede trabajar.

1. Inicie Data Wrangler en el menú superior del dataframe `df`.

1. Seleccione la característica `Brand` en la cuadrícula.

1. En el panel **Operaciones**, expanda **Fórmulas** y, a continuación, seleccione **Codificación one-hot**.

1. En el panel **Codificación de acceso único**, seleccione **Aplicar**.

    Navegue hasta el final de la cuadrícula de visualización de Data Wrangler. Observe que se agregaron tres nuevas características (`Brand_Dominicks`, `Brand_Minute Maid` y `Brand_Tropicana`) y se quitó la característica `Brand`.

1. Cierre Data Wrangler sin generar el código.

## Operaciones de ordenación y filtrado

Imagine que necesitamos revisar los datos de ingresos de una tienda específica y, a continuación, ordenar los precios de los productos. En los pasos siguientes, usamos Data Wrangler para filtrar y analizar el dataframe `df`.

1. Inicie Data Wrangler para el dataframe `df`.

1. En el panel **Operaciones**, expanda **Ordenar y filtrar**.

1. Seleccione la opción **Filtro**.

1. En el panel **Filtrar**, agregue la siguiente condición:

    - **Columna de destino**: `Store`
    - **Operación**: `Equal to`
    - **Valor**: `1227`
    - **Acción**: `Keep matching rows`

1. Seleccione **Aplicar** y observe los cambios en la cuadrícula de visualización de Data Wrangler.

1. Seleccione la característica **Ingresos** y, a continuación, revise los detalles del panel lateral **Resumen**.

    ¿Cuáles son algunas de las conclusiones que se pueden extraer? La asimetría es **-0,751**, lo que indica un ligero sesgo a la izquierda (sesgo negativo). Esto significa que la cola izquierda de la distribución es ligeramente más larga que la cola derecha. En otras palabras, hay un número de períodos con ingresos significativamente por debajo de la media.

1. Vuelva al panel **Operaciones** y expanda **Ordenar y filtrar**.

1. Seleccione **Ordenar valores**.

1. En el panel **Ordenar valores**, seleccione las siguientes propiedades:

    - **Nombre de la columna**: `Price`
    - **Criterio de ordenación**: `Descending`

1. Seleccione **Aplicar**.

    El precio de producto más alto para la tienda **1227** es **2,68 $**. Con solo unos pocos registros es más fácil identificar el precio de producto más alto, pero tenga en cuenta la complejidad al tratar con miles de resultados.

## Examen y eliminación de pasos

Supongamos que cometió un error y necesitase quitar el orden que creó en el paso anterior. Siga estos pasos para quitarla:

1. Vaya al panel **Pasos de limpieza**.

1. Seleccione el paso **Ordenar valores**.

1. Seleccione el icono de eliminación para quitarla.

    ![Captura de pantalla de la página Data Wrangler que muestra el panel buscar y reemplazar.](./Images/data-wrangler-delete.png)

    > **Importante:** La vista de cuadrícula y el resumen se limitan al paso actual.

    Observe que los cambios se revierten al paso anterior, que es el paso **Filtrar**.

1. Cierre Data Wrangler sin generar el código.

## Agregar datos

Supongamos que necesitamos comprender el promedio de ingresos generados por cada marca. En los pasos siguientes, se usa Data Wrangler para realizar un grupo por operación en el dataframe `df`.

1. Inicie Data Wrangler para el dataframe `df`.

1. De vuelta en el panel **Operaciones**, seleccione **Agrupar por y agregar**.

1. En el panel **Columnas para agrupar por**, seleccione la característica `Brand`.

1. Seleccione **Agregar agregación**.

1. En la propiedad **Columna para agregar**, seleccione la característica `Revenue`.

1. Seleccione `Mean` en la propiedad **Tipo de agregación**.

1. Seleccione **Aplicar**.

1. Seleccione **Copiar código en el Portapapeles**.

1. Cierre Data Wrangler sin generar el código.

1. Combine el código de la transformación de la variable `Brand` con el código generado por el paso de agregación de la función `clean_data(df)`. El bloque de código final debería ser similar al siguiente:

    ```python
   def clean_data(df):    
       # Replace all instances of "." with " " in column: 'Brand'    
       df['Brand'] = df['Brand'].str.replace(".", " ", case=False, regex=False)    
       # Capitalize the first character in column: 'Brand'    
       df['Brand'] = df['Brand'].str.title()
        
       # Performed 1 aggregation grouped on column: 'Brand'    
       df = df.groupby(['Brand']).agg(Revenue_mean=('Revenue', 'mean')).reset_index()    
        
       return df    
        
   df = clean_data(df)
    ```

1. Ejecute el código de la celda.

1. Compruebe los datos del dataframe.

    ```python
   print(df)
    ```

    Resultados:

    |   |Marca|Revenue_mean|
    |---|---|---|
    |0|Dominicks|33206.330958|
    |1|Minute Maid|33532.999632|
    |2|Tropicana|33637.863412|

Ha generado el código para algunas de las operaciones de procesamiento previo y lo ha copiado en el cuaderno como una función, que luego puede ejecutar, reutilizar o modificar según sea necesario.

## Guardado del cuaderno y finalización de la sesión con Spark

Ahora que ha terminado el procesamiento previo de los datos para el modelado, puede guardar el cuaderno con un nombre descriptivo y finalizar la sesión con Spark.

1. En la barra de menús del cuaderno, use el icono ⚙️ **Configuración** para ver la configuración del cuaderno.
2. Establezca el **nombre** del cuaderno en **Procesamiento previo de datos con Data Wrangler** y, luego, cierre el panel de configuración.
3. En el menú del cuaderno, seleccione **Detener sesión** para finalizar la sesión con Spark.

## Limpieza de recursos

En este ejercicio, ha creado un cuaderno y ha usado Data Wrangler para explorar y procesar previamente los datos de un modelo de Machine Learning.

Si ha terminado de explorar los pasos de procesamiento previo, puede eliminar el área de trabajo que creó para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **General**, seleccione **Quitar esta área de trabajo**.
