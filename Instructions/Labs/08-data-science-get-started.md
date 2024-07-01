---
lab:
  title: "Introducción a la ciencia de datos en Microsoft\_Fabric"
  module: Get started with data science in Microsoft Fabric
---

# Introducción a la ciencia de datos en Microsoft Fabric

En este laboratorio, ingerirá datos, explorará los datos de un cuaderno, procesará los datos con Data Wrangler y entrenará dos tipos de modelos. Al realizar todos estos pasos, podrá explorar las características de ciencia de datos en Microsoft Fabric.

Al completar este laboratorio, obtendrá experiencia práctica en aprendizaje automático y seguimiento de modelos, y aprenderá a trabajar con *cuadernos*, *Data Wrangler*, *experimentos* y *modelos* en Microsoft Fabric.

Este laboratorio se tarda aproximadamente **20** minutos en completarse.

> **Nota**: Necesitarás una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. Vaya a la página principal de Microsoft Fabric en [https://app.fabric.microsoft.com](https://app.fabric.microsoft.com) en un explorador.
1. Seleccione **Ciencia de datos de Synapse**.
1. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
1. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

## Crear un cuaderno

Para ejecutar código, puede crear un *cuaderno*. Los cuadernos proporcionan un entorno interactivo en el que puede escribir y ejecutar código (en varios lenguajes).

1. En la página principal de **Ciencia de datos de Synapse**, cree un nuevo **cuaderno**.

    Al cabo de unos segundos, se abrirá un nuevo cuaderno que contiene una sola *celda*. Los cuadernos se componen de una o varias celdas que pueden contener *código* o *Markdown* (texto con formato).

1. Seleccione la primera celda (que actualmente es una celda de *código* ) y, luego, en la barra de herramientas dinámica de su parte superior derecha, use el botón **M&#8595;** para convertir la celda en una celda de *Markdown*.

    Cuando la celda cambie a una celda de Markdown, se representará el texto que contiene.

1. Use el botón **&#128393;** (Editar) para cambiar la celda al modo de edición y, luego, elimine el contenido y escriba el texto siguiente:

    ```text
   # Data science in Microsoft Fabric
    ```

## Obtener los datos

Ahora está listo para ejecutar código para obtener los datos y entrenar un modelo. Trabajará con el [conjunto de datos de diabetes](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true) de Azure Open Datasets. Después de cargar los datos, convertirá los datos en un dataframe de Pandas, que es una estructura común para trabajar con datos en filas y columnas.

1. En su cuaderno, use el icono **+ Código** situado debajo de la última salida de celda para agregar una nueva celda de código al cuaderno.

    > **Sugerencia**: Para ver el icono **+ Código**, mueva el ratón hasta justo debajo y a la izquierda de la salida de la celda actual. Como alternativa, en la barra de menús, en la pestaña **Editar**, seleccione **+ Añadir celda de código**.

1. Escriba el código siguiente en la nueva celda de código:

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

1. Use el botón **&#9655; Ejecutar celda** situado a la izquierda de la celda para ejecutarla. Como alternativa, puede presionar `SHIFT` + `ENTER` en el teclado para ejecutar una celda.

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

    La salida muestra las filas y columnas del conjunto de datos de diabetes.

1. Hay dos pestañas en la parte superior de la tabla representada: **Tabla** y **Gráfico**. Seleccionar **Gráfico**.
1. Seleccione **Personalizar gráfico** en la parte superior derecha del gráfico para cambiar la visualización.
1. Cambie el gráfico a la siguiente configuración:
    * **Tipo de gráfico**: `Box plot`
    * **Clave**: *dejar en blanco*
    * **Valores**: `Y`
1. Seleccione **Aplicar** para representar la nueva visualización y explorar la salida.

## Preparación de los datos

Ahora que ha ingerido y explorado los datos, puede transformar los datos. Puede ejecutar código en un cuaderno o usar el Wrangler de datos para generar código automáticamente.

1. Los datos se cargan como un dataframe de Spark. Aunque Data Wrangler acepta DataFrames de Spark o Pandas, actualmente está optimizado para trabajar con Pandas. Por lo tanto, convertirá los datos en un DataFrame de Pandas. Ejecute el código siguiente en su cuaderno:

    ```python
   df = df.toPandas()
   df.head()
    ```

1. Seleccione **Data Wrangler** en la barra de herramientas del cuaderno y, luego, seleccione el conjunto de datos `df`. Cuando se inicia Data Wrangler, se genera una introducción descriptiva del dataframe en el panel **Resumen**.

    Actualmente, la columna de etiqueta es `Y`, que es una variable continua. Para entrenar un modelo de Machine Learning que prediga Y, debe entrenar un modelo de regresión. Los valores (predichos) de Y pueden ser difíciles de interpretar. En su lugar, podríamos explorar la posibilidad de entrenar un modelo de clasificación que prediga si alguien tiene un riesgo bajo o alto de desarrollar diabetes. Para poder entrenar un modelo de clasificación, debe crear una columna de etiqueta binaria basada en los valores de `Y`.

1. Seleccione la columna `Y` en Data Wrangler. Tenga en cuenta que hay una disminución en la frecuencia de la papelera `220-240`. El percentil 75 `211.5` coincide aproximadamente con la transición de las dos regiones en el histograma. Utilicemos este valor como umbral de riesgo bajo y alto.
1. Vaya al panel **Operaciones**, expanda **Fórmulas** y, a continuación, seleccione **Crear columna a partir de la fórmula**.
1. Cree una nueva columna con la siguiente configuración:
    * **Nombre de la columna**: `Risk`
    * **Fórmula de la columna**: `(df['Y'] > 211.5).astype(int)`
1. Revise la nueva columna `Risk` que se agrega a la versión preliminar. Compruebe que el número de filas con valor `1` debe ser aproximadamente el 25 % de todas las filas (ya que es el percentil 75 de `Y`).
1. Seleccione **Aplicar**.
1. Seleccione **Agregar código al cuaderno**.
1. Ejecute la celda con el código que ha generado Data Wrangler.
1. Ejecute el código siguiente en una nueva celda para comprobar que la columna `Risk` tiene la forma esperada:

    ```python
   df_clean.describe()
    ```

## Entrenamiento de modelos de Machine Learning

Ahora que ha preparado los datos, puede utilizarlos para entrenar un modelo de Machine Learning que prediga la diabetes. Podemos entrenar dos tipos diferentes de modelos con nuestro conjunto de datos: un modelo de regresión (predicción `Y`) o un modelo de clasificación (predicción `Risk`). Entrenará los modelos mediante la biblioteca scikit-learn y hará un seguimiento de los modelo con MLflow.

### Entrenamiento de un modelo de regresión

1. Ejecute el código siguiente para dividir los datos en un conjunto de datos de entrenamiento y prueba, y para separar las características de la etiqueta `Y` que desea predecir:

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df_clean[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df_clean['Y'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Agregue otra nueva celda de código al cuaderno, escriba en ella el código siguiente y ejecútela:

    ```python
   import mlflow
   experiment_name = "diabetes-regression"
   mlflow.set_experiment(experiment_name)
    ```

    El código crea un experimento de MLflow llamado `diabetes-regression`. En este experimento se realizará un seguimiento de los modelos.

1. Agregue otra nueva celda de código al cuaderno, escriba en ella el código siguiente y ejecútela:

    ```python
   from sklearn.linear_model import LinearRegression
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = LinearRegression()
      model.fit(X_train, y_train)
    ```

    El código entrena un modelo de regresión mediante la regresión lineal. Los parámetros, las métricas y los artefactos se registran automáticamente con MLflow.

### Entrenamiento de un modelo de clasificación

1. Ejecute el código siguiente para dividir los datos en un conjunto de datos de entrenamiento y prueba, y para separar las características de la etiqueta `Risk` que desea predecir:

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df_clean[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df_clean['Risk'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Agregue otra nueva celda de código al cuaderno, escriba en ella el código siguiente y ejecútela:

    ```python
   import mlflow
   experiment_name = "diabetes-classification"
   mlflow.set_experiment(experiment_name)
    ```

    El código crea un experimento de MLflow llamado `diabetes-classification`. En este experimento se realizará un seguimiento de los modelos.

1. Agregue otra nueva celda de código al cuaderno, escriba en ella el código siguiente y ejecútela:

    ```python
   from sklearn.linear_model import LogisticRegression
    
   with mlflow.start_run():
       mlflow.sklearn.autolog()

       model = LogisticRegression(C=1/0.1, solver="liblinear").fit(X_train, y_train)
    ```

    El código entrena un modelo de clasificación mediante regresión logística. Los parámetros, las métricas y los artefactos se registran automáticamente con MLflow.

## Exploración de los experimentos

Microsoft Fabric realizará un seguimiento de todos los experimentos y le permitirá explorarlos visualmente.

1. Vaya al área de trabajo desde la barra de menús del centro de conectividad de la izquierda.
1. Seleccione el experimento `diabetes-regression` para abrirlo.

    > **Sugerencia:** Si ve que no hay ninguna ejecución de experimentos registrada, actualice la página.

1. Revise las **métricas de ejecución** para explorar la precisión del modelo de regresión.
1. Vuelva a la página principal y seleccione el experimento `diabetes-classification` para abrirlo.
1. Revise las **métricas de ejecución** para explorar la precisión del modelo de clasificación. Tenga en cuenta que el tipo de métrica es diferente, ya que ha entrenado otro tipo de modelo.

## Guardar el modelo

Después de comparar los modelo de Machine Learning que ha entrenado entre experimentos, puede elegir aquel con el mejor rendimiento. Para usar el modelo con el mejor rendimiento, guarde el modelo y úselo para generar predicciones.

1. Seleccione **Guardar como modelo de ML** en la cinta de opciones del experimento.
1. Seleccione **Crear un nuevo modelo de ML** en la ventana emergente recién abierta.
1. Seleccione la carpeta `model`.
1. Asigne al modelo el nombre `model-diabetes` y seleccione **Guardar**.
1. Seleccione **Ver modelo de ML** en la notificación que aparece en la parte superior derecha de la pantalla cuando se crea el modelo. También puede actualizar la ventana. El modelo guardado se vincula en **Versiones del modelo de ML**.

Tenga en cuenta que el modelo, el experimento y la ejecución del experimento están vinculados, lo que le permite revisar cómo se entrena el modelo.

## Guardado del cuaderno y finalización de la sesión con Spark

Ahora que ha terminado de entrenar y evaluar los modelos, puede guardar el cuaderno con un nombre descriptivo y finalizar la sesión con Spark.

1. En la barra de menús del cuaderno, use el icono ⚙️ **Configuración** para ver la configuración del cuaderno.
2. Establezca el **nombre** del cuaderno en **Entrenar y comparar modelos** y, luego, cierre el panel de configuración.
3. En el menú del cuaderno, seleccione **Detener sesión** para finalizar la sesión con Spark.

## Limpieza de recursos

En este ejercicio, ha creado un cuaderno y entrenado un modelo de aprendizaje automático. Ha usado Scikit-Learn para entrenar el modelo y MLflow para realizar un seguimiento de su rendimiento.

Si ha terminado de explorar el modelo y los experimentos, puede eliminar el área de trabajo que ha creado para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **General**, seleccione **Quitar esta área de trabajo**.
