---
lab:
  title: Entrenamiento y seguimiento de modelos de Machine Learning con MLflow en Microsoft Fabric
  module: Train and track machine learning models with MLflow in Microsoft Fabric
---

# Entrenamiento y seguimiento de modelos de Machine Learning con MLflow en Microsoft Fabric

En este laboratorio, entrenará un modelo de aprendizaje automático para predecir una medida cuantitativa de la diabetes. Entrenará un modelo de regresión con Scikit-learn, hará un seguimiento y comparará los modelos con MLflow.

Al completar este laboratorio, obtendrá experiencia práctica en aprendizaje automático y seguimiento de modelos, y aprenderá a trabajar con *cuadernos*, *experimentos* y *modelos* en Microsoft Fabric.

Este laboratorio se tarda aproximadamente **25** minutos en completarse.

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

Para entrenar un modelo, puede crear un *cuaderno*. Los cuadernos proporcionan un entorno interactivo en el que puede escribir y ejecutar código (en varios lenguajes).

1. En la página principal de **Ciencia de datos de Synapse**, cree un nuevo **cuaderno**.

    Al cabo de unos segundos, se abrirá un nuevo cuaderno que contiene una sola *celda*. Los cuadernos se componen de una o varias celdas que pueden contener *código* o *Markdown* (texto con formato).

1. Seleccione la primera celda (que actualmente es una celda de *código* ) y, luego, en la barra de herramientas dinámica de su parte superior derecha, use el botón **M&#8595;** para convertir la celda en una celda de *Markdown*.

    Cuando la celda cambie a una celda de Markdown, se representará el texto que contiene.

1. Si fuera necesario, use el botón **&#128393;** (Editar) para cambiar la celda al modo de edición y, después, elimine el contenido y escriba el siguiente texto:

    ```text
   # Train a machine learning model and track with MLflow
    ```

## Carga de datos en un objeto DataFrame

Ahora está listo para ejecutar código para obtener los datos y entrenar un modelo. Trabajará con el [conjunto de datos de diabetes](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true) de Azure Open Datasets. Después de cargar los datos, convertirá los datos en un dataframe de Pandas, que es una estructura común para trabajar con datos en filas y columnas.

1. En su cuaderno, use el icono **+ Código** situado debajo de la última salida de celda para agregar una nueva celda de código al cuaderno.

    > **Sugerencia**: Para ver el icono **+ Código**, mueva el ratón hasta justo debajo y a la izquierda de la salida de la celda actual. Como alternativa, en la barra de menús, en la pestaña **Editar**, seleccione **+ Añadir celda de código**.

1. Escriba el siguiente código en él:

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

    La salida muestra las filas y columnas del conjunto de datos de diabetes.

1. Los datos se cargan como un dataframe de Spark. Scikit-learn esperará que el conjunto de datos de entrada sea un dataframe de Pandas. Ejecute el código siguiente para convertir el conjunto de datos en un dataframe de Pandas:

    ```python
   import pandas as pd
   df = df.toPandas()
   df.head()
    ```

## Entrenar un modelo de Machine Learning

Ahora que ha cargado los datos, puede usarlos para entrenar un modelo de Machine Learning y predecir una medida cuantitativa de la diabetes. Entrenará un modelo de regresión mediante la biblioteca Scikit-Learn y hará un seguimiento del modelo con MLflow.

1. Ejecute el código siguiente para dividir los datos en un conjunto de datos de entrenamiento y prueba, y para separar las características de la etiqueta que desea predecir:

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df['Y'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Agregue otra nueva celda de código al cuaderno, escriba en ella el código siguiente y ejecútela:

    ```python
   import mlflow
   experiment_name = "experiment-diabetes"
   mlflow.set_experiment(experiment_name)
    ```

    El código crea un experimento de MLflow llamado **experiment-diabetes**. En este experimento se realizará un seguimiento de los modelos.

1. Agregue otra nueva celda de código al cuaderno, escriba en ella el código siguiente y ejecútela:

    ```python
   from sklearn.linear_model import LinearRegression
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = LinearRegression()
      model.fit(X_train, y_train)
    
      mlflow.log_param("estimator", "LinearRegression")
    ```

    El código entrena un modelo de regresión mediante la regresión lineal. Los parámetros, las métricas y los artefactos se registran automáticamente con MLflow. Además, va a registrar un parámetro denominado **estimator** con el valor *LinearRegression*.

1. Agregue otra nueva celda de código al cuaderno, escriba en ella el código siguiente y ejecútela:

    ```python
   from sklearn.tree import DecisionTreeRegressor
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = DecisionTreeRegressor(max_depth=5) 
      model.fit(X_train, y_train)
    
      mlflow.log_param("estimator", "DecisionTreeRegressor")
    ```

    El código entrena un modelo de regresión mediante regresión del árbol de decisión. Los parámetros, las métricas y los artefactos se registran automáticamente con MLflow. Además, va a registrar un parámetro denominado **estimator** con el valor *DecisionTreeRegressor*.

## Uso de MLflow para buscar y ver los experimentos

Cuando haya entrenado y realizado un seguimiento de los modelos con MLflow, puede usar la biblioteca de MLflow para recuperar los experimentos y sus detalles.

1. Para enumerar todos los experimentos, use el código siguiente:

    ```python
   import mlflow
   experiments = mlflow.search_experiments()
   for exp in experiments:
       print(exp.name)
    ```

1. Para recuperar un experimento específico, puede obtenerlo por su nombre:

    ```python
   experiment_name = "experiment-diabetes"
   exp = mlflow.get_experiment_by_name(experiment_name)
   print(exp)
    ```

1. Con un nombre de experimento, puede recuperar todos los trabajos de ese experimento:

    ```python
   mlflow.search_runs(exp.experiment_id)
    ```

1. Para comparar más fácilmente las ejecuciones y salidas del trabajo, puede configurar la búsqueda para ordenar los resultados. Por ejemplo, la celda siguiente ordena los resultados por *start_time*, y muestra un máximo de dos resultados:

    ```python
   mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)
    ```

1. Por último, puede trazar las métricas de evaluación de varios modelos para compararlos fácilmente:

    ```python
   import matplotlib.pyplot as plt
   
   df_results = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)[["metrics.training_r2_score", "params.estimator"]]
   
   fig, ax = plt.subplots()
   ax.bar(df_results["params.estimator"], df_results["metrics.training_r2_score"])
   ax.set_xlabel("Estimator")
   ax.set_ylabel("R2 score")
   ax.set_title("R2 score by Estimator")
   for i, v in enumerate(df_results["metrics.training_r2_score"]):
       ax.text(i, v, str(round(v, 2)), ha='center', va='bottom', fontweight='bold')
   plt.show()
    ```

    La salida debe ser similar a la de la imagen siguiente:

    ![Captura de pantalla de las métricas de evaluación trazadas.](./Images/data-science-metrics.png)

## Exploración de los experimentos

Microsoft Fabric realizará un seguimiento de todos los experimentos y le permitirá explorarlos visualmente.

1. Vaya al área de trabajo desde la barra de menús de conectividad de la izquierda.
1. Seleccione el experimento **experiment-diabetes** para abrirlo.

    > **Sugerencia:** Si ve que no hay ninguna ejecución de experimentos registrada, actualice la página.

1. Seleccione la pestaña **Ver**.
1. Seleccione **Ejecutar lista**.
1. Seleccione las dos ejecuciones más recientes activando su casilla.

    Como resultado, las dos últimas ejecuciones se compararán entre sí en el panel **Comparación de métricas**. De forma predeterminada, las métricas se trazan por nombre de ejecución.

1. Seleccione el botón **&#128393;** (Editar) del gráfico que visualiza el error medio absoluto de cada ejecución.
1. Cambie el **tipo de visualización** a ** barra**.
1. Cambie el valor de **eje X** a **estimador**.
1. Seleccione **Reemplazar** y explore el nuevo gráfico.
1. Opcionalmente, puede repetir estos pasos para los demás gráficos en el panel **Comparación de métricas**.

Al trazar las métricas de rendimiento por estimador registrado, puede revisar qué algoritmo dio lugar a un mejor modelo.

## Guardar el modelo

Después de comparar los modelos de aprendizaje automático que ha entrenado entre ejecuciones de experimentos, puede elegir aquel con el mejor rendimiento. Para usar el modelo con el mejor rendimiento, guarde el modelo y úselo para generar predicciones.

1. En la información general del experimento, asegúrese de que la pestaña **Ver** está seleccionada.
1. Seleccione **Detalles de ejecución**.
1. Seleccione la ejecución con la puntuación de Entrenamiento de R2 más alta.
1. Seleccione **Guardar** en el cuadro **Guardar ejecución como modelo** (es posible que tenga que desplazarse a la derecha para verlo).
1. Seleccione **Crear un nuevo modelo** en la ventana emergente recién abierta.
1. Seleccione la carpeta **model**.
1. Asigne al modelo el nombre `model-diabetes` y seleccione **Guardar**.
1. Seleccione **Ver modelo de ML** en la notificación que aparece en la parte superior derecha de la pantalla cuando se crea el modelo. También puede actualizar la ventana. El modelo guardado se vincula en **Versiones del modelo**.

Tenga en cuenta que el modelo, el experimento y la ejecución del experimento están vinculados, lo que le permite revisar cómo se entrena el modelo.

## Guardado del cuaderno y finalización de la sesión con Spark

Ahora que ha terminado de entrenar y evaluar los modelos, puede guardar el cuaderno con un nombre descriptivo y finalizar la sesión con Spark.

1. Vuelva a su cuaderno y, en la barra de menús del cuaderno, use el icono ⚙️ **Configuración** para ver la configuración del cuaderno.
2. Establezca el **nombre** del cuaderno en **Entrenar y comparar modelos** y, luego, cierre el panel de configuración.
3. En el menú del cuaderno, seleccione **Detener sesión** para finalizar la sesión con Spark.

## Limpieza de recursos

En este ejercicio, ha creado un cuaderno y entrenado un modelo de aprendizaje automático. Ha usado Scikit-Learn para entrenar el modelo y MLflow para realizar un seguimiento de su rendimiento.

Si ha terminado de explorar el modelo y los experimentos, puede eliminar el área de trabajo que ha creado para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **General**, seleccione **Quitar esta área de trabajo**.
