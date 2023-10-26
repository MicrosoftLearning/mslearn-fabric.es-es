---
lab:
  title: Generación y guardado de predicciones por lotes
  module: Generate batch predictions using a deployed model in Microsoft Fabric
---

# Generación y guardado de predicciones por lotes

En este laboratorio, usará un modelo de aprendizaje automático para predecir una medida cuantitativa de la diabetes. Usará la función PREDICT en Fabric para generar las predicciones con un modelo registrado.

Al completar este laboratorio, obtendrá experiencia práctica para generar predicciones y visualizar los resultados.

Este laboratorio se tarda aproximadamente **20** minutos en completarse.

> **Nota**: Necesitará una licencia de Microsoft Fabric para realizar este ejercicio. Consulte [Introducción a Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para obtener más información sobre cómo habilitar una licencia de evaluación de Fabric gratuita. Para hacerlo, necesitará una cuenta *profesional* o *educativa* de Microsoft. Si no tiene una, puede [registrarse para obtener una evaluación gratuita de Microsoft Office 365 E3 o superior](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Crear un área de trabajo

Antes de trabajar con modelos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. Inicie sesión en [Microsoft Fabric](https://app.fabric.microsoft.com) en `https://app.fabric.microsoft.com` y seleccione **Power BI**.
2. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
3. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
4. Cuando se abra la nueva área de trabajo, estará vacía, como se muestra aquí:

    ![Captura de pantalla de un área de trabajo vacía en Power BI.](./Images/new-workspace.png)

## Carga del cuaderno

Para ingerir datos, entrenar y registrar un modelo, ejecutará las celdas de un cuaderno. Cargue el cuaderno en el área de trabajo.

1. En una nueva pestaña del navegador, vaya al cuaderno [Generate-Predictions](https://github.com/MicrosoftLearning/mslearn-fabric/blob/main/Allfiles/Labs/08/Generate-Predictions.ipynb) y descárguelo en una carpeta de su elección.
1. Vuelva a la pestaña del explorador de Fabric y, en la parte inferior izquierda del portal de Fabric, seleccione el icono de **Power BI** y cambie a la experiencia **Ciencia de datos**.
1. En la página principal de **Ciencia de datos**, seleccione **Importar cuaderno**.

    Recibirá una notificación cuando el cuaderno se importe correctamente.

1. Vaya al cuaderno importado denominado `Generate-Predictions`.
1. Lea las instrucciones del cuaderno cuidadosamente y ejecute cada celda individualmente.

## Limpieza de recursos

En este ejercicio, usó un modelo para generar predicciones por lotes.

Si terminó de explorar el cuaderno, elimine el área de trabajo que creó para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **Otros**, seleccione **Quitar esta área de trabajo**.
