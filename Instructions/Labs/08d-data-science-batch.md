---
lab:
  title: "Generación de predicciones por lotes mediante un modelo implementado en Microsoft\_Fabric"
  module: Generate batch predictions using a deployed model in Microsoft Fabric
---

# Generación de predicciones por lotes mediante un modelo implementado en Microsoft Fabric

En este laboratorio, usará un modelo de aprendizaje automático para predecir una medida cuantitativa de la diabetes.

Al completar este laboratorio, obtendrá experiencia práctica para generar predicciones y visualizar los resultados.

Este laboratorio se tarda aproximadamente **20** minutos en completarse.

> **Nota**: Necesitarás una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. Vaya a la página principal de Microsoft Fabric en `https://app.fabric.microsoft.com` en un explorador.
1. En la página principal de Microsoft Fabric, seleccione **Ciencia de datos de Synapse**
1. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
1. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

## Crear un cuaderno

Usará un *cuaderno* para entrenar y usar un modelo en este ejercicio.

1. En la página principal de **Ciencia de datos de Synapse**, cree un nuevo **cuaderno**.

    Al cabo de unos segundos, se abrirá un nuevo cuaderno que contiene una sola *celda*. Los cuadernos se componen de una o varias celdas que pueden contener *código* o *Markdown* (texto con formato).

1. Seleccione la primera celda (que actualmente es una celda de *código* ) y, luego, en la barra de herramientas dinámica de su parte superior derecha, use el botón **M&#8595;** para convertir la celda en una celda de *Markdown*.

    Cuando la celda cambie a una celda de Markdown, se representará el texto que contiene.

1. Si fuera necesario, use el botón **&#128393;** (Editar) para cambiar la celda al modo de edición y, luego, elimine el contenido y escriba el texto siguiente:

    ```text
   # Train and use a machine learning model
    ```

## Entrenar un modelo de Machine Learning

En primer lugar, entrenemos un modelo de Machine Learning que use un algoritmo de *regresión* para predecir la respuesta de interés de pacientes con diabetes (una medida cuantitativa del progreso de la enfermedad un año después de la línea de base)

1. En el cuaderno, use el icono **+ Código** situado debajo de la celda más reciente para agregar una nueva celda de código al cuaderno.

    > **Sugerencia**: Para ver el icono **+ Código**, mueva el ratón hasta justo debajo y a la izquierda de la salida de la celda actual. Como alternativa, en la barra de menús, en la pestaña **Editar**, seleccione **+ Añadir celda de código**.

1. Escriba el código siguiente para cargar y preparar los datos y usarlos para entrenar un modelo.

    ```python
   import pandas as pd
   import mlflow
   from sklearn.model_selection import train_test_split
   from sklearn.tree import DecisionTreeRegressor
   from mlflow.models.signature import ModelSignature
   from mlflow.types.schema import Schema, ColSpec

   # Get the data
   blob_account_name = "azureopendatastorage"
   blob_container_name = "mlsamples"
   blob_relative_path = "diabetes"
   blob_sas_token = r""
   wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
   spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
   df = spark.read.parquet(wasbs_path).toPandas()

   # Split the features and label for training
   X, y = df[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df['Y'].values
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)

   # Train the model in an MLflow experiment
   experiment_name = "experiment-diabetes"
   mlflow.set_experiment(experiment_name)
   with mlflow.start_run():
       mlflow.autolog(log_models=False)
       model = DecisionTreeRegressor(max_depth=5)
       model.fit(X_train, y_train)
       
       # Define the model signature
       input_schema = Schema([
           ColSpec("integer", "AGE"),
           ColSpec("integer", "SEX"),\
           ColSpec("double", "BMI"),
           ColSpec("double", "BP"),
           ColSpec("integer", "S1"),
           ColSpec("double", "S2"),
           ColSpec("double", "S3"),
           ColSpec("double", "S4"),
           ColSpec("double", "S5"),
           ColSpec("integer", "S6"),
        ])
       output_schema = Schema([ColSpec("integer")])
       signature = ModelSignature(inputs=input_schema, outputs=output_schema)
   
       # Log the model
       mlflow.sklearn.log_model(model, "model", signature=signature)
    ```

1. Use el botón **&#9655; Ejecutar celda** situado a la izquierda de la celda para ejecutarla. Como alternativa, presione **MAYÚS** + **ENTRAR** en el teclado para ejecutar una celda.

    > **Nota**: Dado que esta es la primera vez que ha ejecutado código de Spark en esta sesión, se debe iniciar el grupo de Spark. Esto significa que la primera ejecución de la sesión puede tardar un minuto o así en completarse. Las ejecuciones posteriores serán más rápidas.

1. Use el icono **+ Código** situado debajo de la salida de la celda para agregar una nueva celda de código al cuaderno y escriba el código siguiente para registrar el modelo entrenado por el experimento en la celda anterior:

    ```python
   # Get the most recent experiement run
   exp = mlflow.get_experiment_by_name(experiment_name)
   last_run = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=1)
   last_run_id = last_run.iloc[0]["run_id"]

   # Register the model that was trained in that run
   print("Registering the model from run :", last_run_id)
   model_uri = "runs:/{}/model".format(last_run_id)
   mv = mlflow.register_model(model_uri, "diabetes-model")
   print("Name: {}".format(mv.name))
   print("Version: {}".format(mv.version))
    ```

    El modelo ahora se guarda en el área de trabajo como **diabetes-model**. Opcionalmente, se puede usar la característica examinar del área de trabajo para buscar el modelo en el área de trabajo y explorarlo mediante la interfaz de usuario.

## Creación de un conjunto de datos de prueba en un almacén de lago

Para usar el modelo, será necesario un conjunto de datos de detalles de pacientes para los que se necesita predecir un diagnóstico de diabetes. Creará este conjunto de datos como una tabla en un almacén de lago de Microsoft Fabric.

1. En el editor de Notebook, en el panel **Explorador** de la izquierda, seleccione **+ Orígenes de datos** para agregar un almacén de lago.
1. Seleccione **Nuevo almacén de lago**, seleccione **Agregar** y cree un nuevo **Almacén de lago** con un nombre válido de su elección.
1. Cuando se le pida que detenga la sesión actual, seleccione **Detener ahora** para reiniciar el cuaderno.
1. Cuando se cree y adjunte el almacén de lago al cuaderno, agregue una nueva celda de código que ejecute el código siguiente para crear un conjunto de datos y guardarlo en una tabla del almacén de lago:

    ```python
   from pyspark.sql.types import IntegerType, DoubleType

   # Create a new dataframe with patient data
   data = [
       (62, 2, 33.7, 101.0, 157, 93.2, 38.0, 4.0, 4.8598, 87),
       (50, 1, 22.7, 87.0, 183, 103.2, 70.0, 3.0, 3.8918, 69),
       (76, 2, 32.0, 93.0, 156, 93.6, 41.0, 4.0, 4.6728, 85),
       (25, 1, 26.6, 84.0, 198, 131.4, 40.0, 5.0, 4.8903, 89),
       (53, 1, 23.0, 101.0, 192, 125.4, 52.0, 4.0, 4.2905, 80),
       (24, 1, 23.7, 89.0, 139, 64.8, 61.0, 2.0, 4.1897, 68),
       (38, 2, 22.0, 90.0, 160, 99.6, 50.0, 3.0, 3.9512, 82),
       (69, 2, 27.5, 114.0, 255, 185.0, 56.0, 5.0, 4.2485, 92),
       (63, 2, 33.7, 83.0, 179, 119.4, 42.0, 4.0, 4.4773, 94),
       (30, 1, 30.0, 85.0, 180, 93.4, 43.0, 4.0, 5.3845, 88)
   ]
   columns = ['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']
   df = spark.createDataFrame(data, schema=columns)

   # Convert data types to match the model input schema
   df = df.withColumn("AGE", df["AGE"].cast(IntegerType()))
   df = df.withColumn("SEX", df["SEX"].cast(IntegerType()))
   df = df.withColumn("BMI", df["BMI"].cast(DoubleType()))
   df = df.withColumn("BP", df["BP"].cast(DoubleType()))
   df = df.withColumn("S1", df["S1"].cast(IntegerType()))
   df = df.withColumn("S2", df["S2"].cast(DoubleType()))
   df = df.withColumn("S3", df["S3"].cast(DoubleType()))
   df = df.withColumn("S4", df["S4"].cast(DoubleType()))
   df = df.withColumn("S5", df["S5"].cast(DoubleType()))
   df = df.withColumn("S6", df["S6"].cast(IntegerType()))

   # Save the data in a delta table
   table_name = "diabetes_test"
   df.write.format("delta").mode("overwrite").saveAsTable(table_name)
   print(f"Spark dataframe saved to delta table: {table_name}")
    ```

1. Cuando se haya completado el código, seleccione el **...** situado junto a las **Tablas** del panel **Explorador de almacenes de lago** y seleccione **Actualizar**. Debería aparecer la tabla **diabetes_test**.
1. Expanda la tabla **diabetes_test** del panel izquierdo para ver todos los campos que se incluyen.

## Aplicación del modelo para generar predicciones

Ahora se puede usar el modelo que se entrenó anteriormente para generar predicciones del progreso de la diabetes para las filas de datos de pacientes en la tabla.

1. Agregue una nueva celda de código y ejecute el código siguiente:

    ```python
   import mlflow
   from synapse.ml.predict import MLFlowTransformer

   ## Read the patient features data 
   df_test = spark.read.format("delta").load(f"Tables/{table_name}")

   # Use the model to generate diabetes predictions for each row
   model = MLFlowTransformer(
       inputCols=["AGE","SEX","BMI","BP","S1","S2","S3","S4","S5","S6"],
       outputCol="predictions",
       modelName="diabetes-model",
       modelVersion=1)
   df_test = model.transform(df)

   # Save the results (the original features PLUS the prediction)
   df_test.write.format('delta').mode("overwrite").option("mergeSchema", "true").saveAsTable(table_name)
    ```

1. Cuando se finalice el código, seleccione el **...** situado junto a la tabla **diabetes_test** del panel **Explorador de almacenes de lago** y seleccione **Actualizar**. Se agregó un nuevo campo **predicciones**.
1. Agregue una nueva celda de código al cuaderno y arrástrele la tabla **diabetes_test**. Aparecerá el código necesario para ver el contenido de la tabla. Ejecute la celda para mostrar los datos.

## Limpieza de recursos

En este ejercicio, usó un modelo para generar predicciones por lotes.

Si terminó de explorar el cuaderno, elimine el área de trabajo que creó para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **General**, seleccione **Quitar esta área de trabajo**.
