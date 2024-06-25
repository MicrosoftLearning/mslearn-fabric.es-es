---
lab:
  title: Explorar datos sobre la ciencia de datos con cuadernos en Microsoft Fabric
  module: Explore data for data science with notebooks in Microsoft Fabric
---

# Explorar datos sobre la ciencia de datos con cuadernos en Microsoft Fabric

En este laboratorio, usaremos cuadernos para la exploración de datos. Los cuadernos son una herramienta eficaz para explorar y analizar datos de forma interactiva. Durante este ejercicio, aprenderemos a crear y usar cuadernos para explorar un conjunto de datos, generar estadísticas de resumen y crear visualizaciones para comprender mejor los datos. Al final de este laboratorio, comprenderá de forma sólida cómo usar cuadernos para la exploración y el análisis de datos.

Este laboratorio se realiza en unos **30** minutos.

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

Ahora está listo para ejecutar código para obtener datos. Trabajará con el [**conjunto de datos de diabetes**](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true) de Azure Open Datasets. Después de cargar los datos, convertirá los datos en un dataframe de Pandas, que es una estructura común para trabajar con datos en filas y columnas.

1. En el cuaderno, use el icono **+ Código** situado debajo de la celda más reciente para agregar una nueva celda de código al cuaderno.

    > **Sugerencia**: Para ver el icono **+ Código**, mueva el ratón hasta justo debajo y a la izquierda de la salida de la celda actual. Como alternativa, en la barra de menús, en la pestaña **Editar**, seleccione **+ Añadir celda de código**.

1. Escriba el código siguiente para cargar el conjunto de datos en una trama de datos.

    ```python
   # Azure storage access info for open dataset diabetes
   blob_account_name = "azureopendatastorage"
   blob_container_name = "mlsamples"
   blob_relative_path = "diabetes"
   blob_sas_token = r"" # Blank since container is Anonymous access
    
   # Set Spark config to access  blob storage
   wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
   spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
   print("Remote blob path: " + wasbs_path)
    
   # Spark read parquet, note that it won't load any data yet by now
   df = spark.read.parquet(wasbs_path)
    ```

1. Use el botón **&#9655; Ejecutar celda** situado a la izquierda de la celda para ejecutarla. Como alternativa, presione **MAYÚS** + **ENTRAR** en el teclado para ejecutar una celda.

    > **Nota**: Dado que esta es la primera vez que ha ejecutado código de Spark en esta sesión, se debe iniciar el grupo de Spark. Esto significa que la primera ejecución de la sesión puede tardar un minuto o así en completarse. Las ejecuciones posteriores serán más rápidas.

1. Use el icono **+Código** debajo de la salida de la celda para agregar una nueva celda de código al cuaderno y escriba en ella el código siguiente:

    ```python
   display(df)
    ```

1. Cuando se haya completado el comando de la celda, revise la salida que aparece debajo de ella, que será algo parecido a esto:

    |AGE|SEX|BMI|BP|S1|S2|S3|S4|S5|S6|Y|
    |---|---|---|--|--|--|--|--|--|--|--|
    |59|2|32,1|101.0|157|93.2|38.0|4.0|4.8598|87|151|
    |48|1|21.6|87,0|183|103.2|70.0|3.0|3.8918|69|75|
    |72|2|30,5|93.0|156|93.6|41,0|4.0|4.6728|85|141|
    |24|1|25,3|84.0|198|131.4|40,0|5.0|4.8903|89|206|
    |50|1|23,0|101.0|192|125,4|52,0|4.0|4.2905|80|135|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    La salida muestra las filas y columnas del conjunto de datos de diabetes. Los datos constan de diez variables de línea base, edad, sexo, índice de masa corporal, presión arterial media y seis mediciones de suero sanguíneo de pacientes con diabetes, así como la respuesta de interés (una medida cuantitativa de la progresión de la enfermedad un año después de la línea base), que se etiqueta como **Y**.

1. Los datos se cargan como un dataframe de Spark. Scikit-learn esperará que el conjunto de datos de entrada sea un dataframe de Pandas. Ejecute el código siguiente para convertir el conjunto de datos en un dataframe de Pandas:

    ```python
   df = df.toPandas()
   df.head()
    ```

## Comprobar la forma de los datos

Ahora que ha cargado los datos, puede comprobar la estructura del conjunto de datos, como el número de filas y columnas, los tipos de datos y los valores que faltan.

1. Use el icono **+Código** debajo de la salida de la celda para agregar una nueva celda de código al cuaderno y escriba en ella el código siguiente:

    ```python
   # Display the number of rows and columns in the dataset
   print("Number of rows:", df.shape[0])
   print("Number of columns:", df.shape[1])

   # Display the data types of each column
   print("\nData types of columns:")
   print(df.dtypes)
    ```

    El conjunto de datos contiene **442 filas** y **11 columnas**. Esto significa que tiene 442 ejemplos y 11 características o variables en el conjunto de datos. Es probable que la variable **SEX** contenga datos de categorías o cadenas.

## Comprobar los datos que faltan

1. Use el icono **+Código** debajo de la salida de la celda para agregar una nueva celda de código al cuaderno y escriba en ella el código siguiente:

    ```python
   missing_values = df.isnull().sum()
   print("\nMissing values per column:")
   print(missing_values)
    ```

    El código comprueba si faltan valores. Observar que no faltan datos en el conjunto de datos.

## Generar estadísticas descriptivas para variables numéricas

Ahora, vamos a generar estadísticas descriptivas para comprender la distribución de variables numéricas.

1. Use el icono **+Código** situado debajo de la salida de la celda para agregar una nueva celda de código al cuaderno y escriba el código siguiente:

    ```python
   df.describe()
    ```

    La edad promedio es de aproximadamente 48.5 años, con una desviación estándar de 13.1 años. La persona más joven tiene 19 años y la más mayor tiene 79 años. El promedio del IMC es de aproximadamente 26,4, que entra en la categoría de **sobrepeso** según las [normas de la OMS](https://www.who.int/health-topics/obesity#tab=tab_1). El IMC mínimo es 18 y el máximo es 42.2.

## Trazar la distribución de datos

Vamos a comprobar la característica del IMC y trazar su distribución para comprender mejor sus características.

1. Agregue otra celda de código al cuaderno. A continuación, escriba el código siguiente en esta celda y ejecútelo.

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
   import numpy as np
    
   # Calculate the mean, median of the BMI variable
   mean = df['BMI'].mean()
   median = df['BMI'].median()
   
   # Histogram of the BMI variable
   plt.figure(figsize=(8, 6))
   plt.hist(df['BMI'], bins=20, color='skyblue', edgecolor='black')
   plt.title('BMI Distribution')
   plt.xlabel('BMI')
   plt.ylabel('Frequency')
    
   # Add lines for the mean and median
   plt.axvline(mean, color='red', linestyle='dashed', linewidth=2, label='Mean')
   plt.axvline(median, color='green', linestyle='dashed', linewidth=2, label='Median')
    
   # Add a legend
   plt.legend()
   plt.show()
    ```

    A partir de este gráfico, puede observar el intervalo y la distribución del IMC en el conjunto de datos. Por ejemplo, la mayor parte del IMC está entre 23.2 y 29.2, y los datos están sesgados a la derecha.

## Realizar un análisis multivariante

Vamos a generar visualizaciones como gráficos de dispersión y diagramas de caja para descubrir patrones y relaciones dentro de los datos.

1. Use el icono **+Código** situado debajo de la salida de la celda para agregar una nueva celda de código al cuaderno y escriba el código siguiente:

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns

   # Scatter plot of BMI vs. Target variable
   plt.figure(figsize=(8, 6))
   sns.scatterplot(x='BMI', y='Y', data=df)
   plt.title('BMI vs. Target variable')
   plt.xlabel('BMI')
   plt.ylabel('Target')
   plt.show()
    ```

    Podemos ver que a medida que el IMC aumenta, la variable de destino también aumenta, lo que indica una relación lineal positiva entre estas dos variables.

1. Agregue otra celda de código al cuaderno. A continuación, escriba el código siguiente en esta celda y ejecútelo.

    ```python
   import seaborn as sns
   import matplotlib.pyplot as plt
    
   fig, ax = plt.subplots(figsize=(7, 5))
    
   # Replace numeric values with labels
   df['SEX'] = df['SEX'].replace({1: 'Male', 2: 'Female'})
    
   sns.boxplot(x='SEX', y='BP', data=df, ax=ax)
   ax.set_title('Blood pressure across Gender')
   plt.tight_layout()
   plt.show()
    ```

    Estas observaciones sugieren que existen diferencias en los perfiles de presión arterial de los pacientes masculinos y femeninos. En promedio, los pacientes femeninos tienen una presión arterial más alta que los pacientes masculinos.

1. La agregación de los datos puede hacerlos más manejables para su visualización y análisis. Agregue otra celda de código al cuaderno. A continuación, escriba el código siguiente en esta celda y ejecútelo.

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
    
   # Calculate average BP and BMI by SEX
   avg_values = df.groupby('SEX')[['BP', 'BMI']].mean()
    
   # Bar chart of the average BP and BMI by SEX
   ax = avg_values.plot(kind='bar', figsize=(15, 6), edgecolor='black')
    
   # Add title and labels
   plt.title('Avg. Blood Pressure and BMI by Gender')
   plt.xlabel('Gender')
   plt.ylabel('Average')
    
   # Display actual numbers on the bar chart
   for p in ax.patches:
       ax.annotate(format(p.get_height(), '.2f'), 
                   (p.get_x() + p.get_width() / 2., p.get_height()), 
                   ha = 'center', va = 'center', 
                   xytext = (0, 10), 
                   textcoords = 'offset points')
    
   plt.show()
    ```

    Este gráfico muestra que la presión arterial media es mayor en pacientes femeninos en comparación con los pacientes masculinos. Además, muestra que el Índice de Masa Corporal (IMC) medio es ligeramente superior en las mujeres que en los hombres.

1. Agregue otra celda de código al cuaderno. A continuación, escriba el código siguiente en esta celda y ejecútelo.

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
    
   plt.figure(figsize=(10, 6))
   sns.lineplot(x='AGE', y='BMI', data=df, errorbar=None)
   plt.title('BMI over Age')
   plt.xlabel('Age')
   plt.ylabel('BMI')
   plt.show()
    ```

    El grupo de edad de 19 a 30 años tiene los valores medios de IMC más bajos, mientras que el IMC medio más alto se encuentra en el grupo de edad de 65 a 79 años. Además, observe que el IMC medio de la mayoría de los grupos de edad se sitúa en el intervalo del sobrepeso.

## Análisis de correlación

Vamos a calcular las correlaciones entre diferentes características para comprender sus relaciones y dependencias.

1. Use el icono **+Código** situado debajo de la salida de la celda para agregar una nueva celda de código al cuaderno y escriba el código siguiente:

    ```python
   df.corr(numeric_only=True)
    ```

1. Un mapa térmico es una herramienta útil para visualizar rápidamente la fuerza y la dirección de las relaciones entre pares variables. Puede resaltar correlaciones positivas o negativas fuertes e identificar pares que carezcan de correlación. Para crear un mapa térmico, agregue otra celda de código al cuaderno y escriba el código siguiente.

    ```python
   plt.figure(figsize=(15, 7))
   sns.heatmap(df.corr(numeric_only=True), annot=True, vmin=-1, vmax=1, cmap="Blues")
    ```

    Las variables S1 y S2 tienen una correlación positiva alta de **0,89**, lo que indica que se mueven en la misma dirección. Cuando S1 aumenta, S2 también tiende a aumentar y viceversa. Además, S3 y S4 tienen una fuerte correlación negativa de **-0,73**. Esto significa que a medida que S3 aumenta, S4 tiende a disminuir.

## Guardado del cuaderno y finalización de la sesión con Spark

Ahora que ha terminado de trabajar con los datos, puede guardar el cuaderno con un nombre descriptivo y finalizar la sesión con Spark.

1. En la barra de menús del cuaderno, use el icono ⚙️ **Configuración** para ver la configuración del cuaderno.
2. Establezca el **nombre** del cuaderno en **Explorar datos para la ciencia de datos** y, luego, cierre el panel de configuración.
3. En el menú del cuaderno, seleccione **Detener sesión** para finalizar la sesión con Spark.

## Limpieza de recursos

En este ejercicio, ha creado y usado cuadernos para la exploración de datos. También ha ejecutado código para calcular estadísticas de resumen y crear visualizaciones para comprender mejor los patrones y las relaciones de los datos.

Si ha terminado de explorar el modelo y los experimentos, puede eliminar el área de trabajo que ha creado para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **General**, seleccione **Quitar esta área de trabajo**.
