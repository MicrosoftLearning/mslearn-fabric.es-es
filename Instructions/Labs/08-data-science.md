---
lab:
  title: Entrenamiento de un modelo de clasificación para predecir el abandono de clientes
  module: Get started with data science in Microsoft Fabric
---

# Uso de cuadernos para entrenar un modelo en Microsoft Fabric

En este laboratorio, usaremos Microsoft Fabric para crear un cuaderno y entrenar un modelo de aprendizaje automático para predecir el abandono de clientes. Usaremos Scikit-Learn para entrenar el modelo y MLflow para realizar un seguimiento de su rendimiento. El abandono de clientes es un problema empresarial crítico al que se enfrentan muchas empresas y poder predecirlo puede ayudarles a conservar sus clientes y aumentar los ingresos. Al realizar este laboratorio, adquirirá experiencia práctica en el aprendizaje automático y el seguimiento de modelos, y aprenderá a usar Microsoft Fabric para crear un cuaderno para sus proyectos.

Este laboratorio se tarda en completar **45** minutos aproximadamente.

> **Nota**: Necesitará una licencia de Microsoft Fabric para realizar este ejercicio. Consulte [Introducción a Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para más información sobre cómo habilitar una licencia de prueba de Fabric gratuita. Para ello, necesitará una cuenta *profesional* o *educativa* de Microsoft. Si no tiene una, puede [registrarse para una evaluación gratuita de Microsoft Office 365 E3 o superior](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Crear un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. Inicie sesión en [Microsoft Fabric](https://app.fabric.microsoft.com) en `https://app.fabric.microsoft.com` y seleccione **Power BI**.
2. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
3. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Versión de prueba*, *Premium* o *Fabric*).
4. Cuando se abra la nueva área de trabajo, estará vacía, como se muestra aquí:

    ![Captura de pantalla de un área de trabajo vacía en Power BI.](./Images/new-workspace.png)

## Creación de un almacén de lago y carga de archivos

Ahora que tiene un área de trabajo, es el momento de cambiar a la experiencia *Ciencia de datos* en el portal y crear un almacén de lago de datos para los archivos de datos que va a analizar.

1. En la parte inferior izquierda del portal de Power BI, seleccione el icono de **Power BI** y cambie a la experiencia **Ingeniería de datos**.
1. En la página principal de **Ingeniería de datos**, cree un nuevo **almacén de lago** con el nombre que prefiera.

    Al cabo de un minuto más o menos, se creará un nuevo almacén de lago sin **tablas** ni **archivos**. Debe ingerir algunos datos en el almacén de lago de datos para su análisis. Hay varias maneras de hacerlo, pero en este ejercicio simplemente descargará y extraerá una carpeta de archivos de texto del equipo local (o máquina virtual de laboratorio si procede) y, luego, los cargará en el almacén de lago.

1. Descargue y guarde el archivo CSV `churn.csv` para este ejercicio desde [https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/churn.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/churn.csv).


1. Vuelva a la pestaña del explorador web que contiene el almacén de lago y, en el menú **...** del nodo **Archivos** en el panel **Vista de lago**, seleccione **Cargar** y **Cargar archivos** y, luego, cargue el archivo **churn.csv** del equipo local (o la máquina virtual de laboratorio si procede) en el almacén de lago.
6. Una vez cargados los archivos, expanda **Archivos** y compruebe que se ha cargado el archivo CSV.

## Creación de un cuaderno

Para entrenar un modelo, puede crear un *cuaderno*. Los cuadernos proporcionan un entorno interactivo en el que puede escribir y ejecutar código (en varios lenguajes) como *experimentos*.

1. En la parte inferior izquierda del portal de Power BI, seleccione el icono **Ingeniería de datos** y cambie a la experiencia **Ciencia de datos**.

1. En la página principal de **Ciencia de datos**, cree un nuevo **cuaderno**.

    Al cabo de unos segundos, se abrirá un nuevo cuaderno que contiene una sola *celda*. Los cuadernos se componen de una o varias celdas que pueden contener *código* o *Markdown* (texto con formato).

1. Seleccione la primera celda (que actualmente es una celda de *código* ) y, luego, en la barra de herramientas dinámica de su parte superior derecha, use el botón **M&#8595;** para convertir la celda en una celda de *Markdown*.

    Cuando la celda cambie a una celda de Markdown, se representará el texto que contiene.

1. Use el botón **&#128393;** (Editar) para cambiar la celda al modo de edición y, luego, elimine el contenido y escriba el texto siguiente:

    ```text
   # Train a machine learning model and track with MLflow

   Use the code in this notebook to train and track models.
    ``` 

## Carga de datos en un objeto DataFrame

Ahora está listo para ejecutar código para preparar los datos y entrenar un modelo. Para trabajar con datos, usará objetos *DataFrame*. Los objetos DataFrame de Spark son similares a los de Pandas en Python y proporcionan una estructura común para trabajar con datos en filas y columnas.

1. En el panel **Agregar almacén de lago**, seleccione **Agregar** para agregar un almacén de lago.
1. Seleccione **Almacén de lago existente** y elija **Agregar**.
1. Seleccione el almacén de lago que creó en una sección anterior.
1. Expanda la carpeta **Archivos** para que el archivo CSV aparezca junto al editor de cuadernos.
1. En el menú **...** del archivo **churn.csv**, seleccione **Cargar datos** > **Pandas**. Se agregará al cuaderno una nueva celda de código que contiene el código siguiente:

    ```python
   import pandas as pd
   # Load data into pandas DataFrame from "/lakehouse/default/" + "Files/churn.csv"
   df = pd.read_csv("/lakehouse/default/" + "Files/churn.csv")
   display(df)
    ```

    > **Sugerencia**: Puede ocultar el panel que contiene los archivos de la izquierda usando su icono **<<** . De esta forma, podrá centrarse en el cuaderno.

1. Use el botón **&#9655; Ejecutar celda** situado a la izquierda de la celda para ejecutarla.

    > **Nota**: Dado que esta es la primera vez que ha ejecutado código de Spark en esta sesión, se debe iniciar el grupo de Spark. Esto significa que la primera ejecución de la sesión puede tardar un minuto o así en completarse. Las ejecuciones posteriores serán más rápidas.

1. Cuando se haya completado el comando de la celda, revise la salida que aparece debajo de ella, que será algo parecido a esto:

    |Índice|CustomerID|years_with_company|total_day_calls|total_eve_calls|total_night_calls|total_intl_calls|average_call_minutes|total_customer_service_calls|age|churn|
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    |1|1000038|0|117|88|32|607|43.90625678|0.810828179|34|0|
    |2|1000183|1|164|102|22|40|49.82223317|0.294453889|35|0|
    |3|1000326|3|116|43|45|207|29.83377967|1.344657937|57|1|
    |4|1000340|0|92|24|11|37|31.61998183|0.124931779|34|0|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    La salida muestra las filas y columnas de los datos de clientes del archivo churn.csv.

## Entrenar un modelo de Machine Learning

Ahora que se han cargado los datos, puede usarlos para entrenar un modelo de aprendizaje automático y predecir el abandono de clientes. Entrenará un modelo mediante la biblioteca Scikit-Learn y hará un seguimiento del modelo con MLflow. 

1. Use el icono **+Código** debajo de la salida de la celda para agregar una nueva celda de código al cuaderno y escriba en ella el código siguiente:

    ```python
   from sklearn.model_selection import train_test_split

   print("Splitting data...")
   X, y = df[['years_with_company','total_day_calls','total_eve_calls','total_night_calls','total_intl_calls','average_call_minutes','total_customer_service_calls','age']].values, df['churn'].values
   
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Ejecute la celda de código que ha agregado y tenga en cuenta que va a omitir "CustomerID" del conjunto de datos y dividir los datos en un conjunto de datos de entrenamiento y prueba.
1. Agregue otra nueva celda de código al cuaderno, escriba en ella el código siguiente y ejecútela:
    
    ```python
   import mlflow
   experiment_name = "experiment-churn"
   mlflow.set_experiment(experiment_name)
    ```
    
    El código crea un experimento de MLflow llamado `experiment-churn`. En este experimento se realizará un seguimiento de los modelos.

1. Agregue otra nueva celda de código al cuaderno, escriba en ella el código siguiente y ejecútela:

    ```python
   from sklearn.linear_model import LogisticRegression
   
   with mlflow.start_run():
       mlflow.autolog()

       model = LogisticRegression(C=1/0.1, solver="liblinear").fit(X_train, y_train)

       mlflow.log_param("estimator", "LogisticRegression")
    ```
    
    El código entrena un modelo de clasificación mediante regresión logística. Los parámetros, las métricas y los artefactos se registran automáticamente con MLflow. Además, va a registrar un parámetro llamado `estimator`, con el valor `LogisticRegression`.

1. Agregue otra nueva celda de código al cuaderno, escriba en ella el código siguiente y ejecútela:

    ```python
   from sklearn.tree import DecisionTreeClassifier
   
   with mlflow.start_run():
       mlflow.autolog()

       model = DecisionTreeClassifier().fit(X_train, y_train)
   
       mlflow.log_param("estimator", "DecisionTreeClassifier")
    ```

    El código entrena un modelo de clasificación mediante el clasificador de árbol de decisión. Los parámetros, las métricas y los artefactos se registran automáticamente con MLflow. Además, va a registrar un parámetro llamado `estimator`, con el valor `DecisionTreeClassifier`.

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
   experiment_name = "experiment-churn"
   exp = mlflow.get_experiment_by_name(experiment_name)
   print(exp)
    ```

1. Con un nombre de experimento, puede recuperar todos los trabajos de ese experimento:

    ```python
   mlflow.search_runs(exp.experiment_id)
    ```

1. Para comparar más fácilmente las ejecuciones y salidas del trabajo, puede configurar la búsqueda para ordenar los resultados. Por ejemplo, la celda siguiente ordena los resultados por `start_time`, y solo muestra un máximo de `2` resultados: 

    ```python
   mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)
    ```

1. Por último, puede trazar las métricas de evaluación de varios modelos para compararlos fácilmente:

    ```python
   import matplotlib.pyplot as plt
   
   df_results = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)[["metrics.training_accuracy_score", "params.estimator"]]
   
   fig, ax = plt.subplots()
   ax.bar(df_results["params.estimator"], df_results["metrics.training_accuracy_score"])
   ax.set_xlabel("Estimator")
   ax.set_ylabel("Accuracy")
   ax.set_title("Accuracy by Estimator")
   for i, v in enumerate(df_results["metrics.training_accuracy_score"]):
       ax.text(i, v, str(round(v, 2)), ha='center', va='bottom', fontweight='bold')
   plt.show()
    ```

    La salida debe ser similar a la de la imagen siguiente:

    ![Captura de pantalla de las métricas de evaluación trazadas.](./Images/plotted-metrics.png)

## Exploración de los experimentos

Microsoft Fabric realizará un seguimiento de todos los experimentos y le permitirá explorarlos visualmente.

1. Vaya a la página principal de **Ciencia de datos**.
1. Seleccione el experimento `experiment-churn` para abrirlo.

    > **Sugerencia:** Si ve que no hay ninguna ejecución de experimentos registrada, actualice la página.

1. Seleccione la pestaña **Ver**.
1. Seleccione **Ejecutar lista**. 
1. Seleccione las dos ejecuciones más recientes activando su casilla.
    Como resultado, las dos últimas ejecuciones se compararán entre sí en el panel **Comparación de métricas**. De forma predeterminada, las métricas se trazan por nombre de ejecución. 
1. Seleccione el botón **&#128393;** (Editar) del gráfico que visualiza la precisión de cada ejecución. 
1. Cambie el **tipo de visualización** a `bar`. 
1. Cambie el **eje X** a `estimator`. 
1. Seleccione **Reemplazar** y explore el nuevo gráfico.

Al trazar la precisión por estimador registrado, puede revisar qué algoritmo dio lugar a un mejor modelo.

## Guardar el modelo

Después de comparar los modelos de aprendizaje automático que ha entrenado entre ejecuciones de experimentos, puede elegir aquel con el mejor rendimiento. Para usar el modelo con el mejor rendimiento, guarde el modelo y úselo para generar predicciones.

1. En la información general del experimento, asegúrese de que la pestaña **Ver** está seleccionada.
1. Seleccione **Detalles de ejecución**.
1. Seleccione la ejecución con la precisión más alta. 
1. Seleccione **Guardar** en el cuadro **Guardar como modelo**.
1. Seleccione **Crear un nuevo modelo** en la ventana emergente recién abierta.
1. Asigne al modelo el nombre `model-churn` y seleccione **Crear**. 
1. Seleccione **Ver modelo** en la notificación que aparece en la parte superior derecha de la pantalla cuando se crea el modelo. También puede actualizar la ventana. El modelo guardado se vincula en **Versión registrada**. 

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
3. En la sección **Otros**, seleccione **Quitar esta área de trabajo**.
