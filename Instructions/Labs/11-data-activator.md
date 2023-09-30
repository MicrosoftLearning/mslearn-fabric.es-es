---
lab:
  title: Uso de Data Activator en Fabric
  module: Get started with Data Activator in Microsoft Fabric
---

# Uso de Data Activator en Fabric

En Microsoft Fabric, un almacenamiento de datos proporciona una base de datos relacional para análisis a gran escala. A diferencia del punto de conexión de SQL de solo lectura predeterminado para las tablas definidas en un almacén de lago, un almacenamiento de datos proporciona semántica de SQL completa; incluida la capacidad de insertar, actualizar y eliminar datos de las tablas.

Este laboratorio se tarda aproximadamente **30** minutos en completarse.

> **Nota**: Necesitará una licencia de Microsoft Fabric para realizar este ejercicio. Consulte [Introducción a Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para obtener más información sobre cómo habilitar una licencia de evaluación de Fabric gratuita. Para hacerlo, necesitará una cuenta *profesional* o *educativa* de Microsoft. Si no tiene una, puede [registrarse para obtener una evaluación gratuita de Microsoft Office 365 E3 o superior](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Crear un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. Inicie sesión en [Microsoft Fabric](https://app.fabric.microsoft.com) en `https://app.fabric.microsoft.com` y seleccione **Power BI**.
2. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
3. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
4. Cuando se abra la nueva área de trabajo, estará vacía, como se muestra aquí:

    ![Captura de pantalla de un área de trabajo vacía en Power BI.](./Images/new-workspace.png)
