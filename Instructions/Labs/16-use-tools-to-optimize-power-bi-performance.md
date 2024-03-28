---
lab:
  title: "Uso de herramientas para optimizar el rendimiento de Power\_BI"
  module: Optimize enterprise-scale tabular models
---

# Uso de herramientas para optimizar el rendimiento de Power BI

En este laboratorio, aprenderás a usar dos herramientas externas para desarrollar, administrar y optimizar modelos de datos y consultas DAX.

En este ejercicio aprenderá a usar:

- El Analizador de procedimientos recomendados (BPA) en Tabular Editor.
- DAX Studio.

Este laboratorio se realiza en unos **30** minutos.

> **Nota**: Necesitará una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Introducción

Para este laboratorio, instalará y usará Tabular Editor y DAX Studio para optimizar un modelo semántico.

## Usar el Analizador de procedimientos recomendados

En este ejercicio, instalarás Tabular Editor 2 y cargarás reglas del Analizador de procedimientos recomendados (BPA). Revisarás las reglas BPA y después solucionarás los problemas específicos que se encuentran en el modelo de datos.

*BPA es una herramienta gratuita de terceros que te notifica posibles errores de modelado o cambios que puedes realizar para mejorar el diseño y el rendimiento de tu modelo. Incluye recomendaciones de nomenclatura, experiencia de usuario y optimizaciones comunes que puedes aplicar para mejorar el rendimiento. Para obtener más información, consulta [Reglas de procedimientos recomendados para mejorar el rendimiento de tu modelo](https://powerbi.microsoft.com/blog/best-practice-rules-to-improve-your-models-performance/).*

### Descarga e instalación del Tabular Editor 2

Descarga e instalación de Tabular Editor 2 para habilitar la creación de grupos de cálculo.

***Importante**: Si ya ha instalado Tabular Editor 2 en el entorno de máquina virtual, continúe con la siguiente tarea.*

*Tabular Editor es una herramienta alternativa para crear modelos tabulares para Analysis Services y Power BI. Tabular Editor 2 es un proyecto de código abierto que puede editar un archivo BIM sin acceder a ningún dato del modelo.*

1. Asegúrate de que Power BI Desktop está cerrado.

1. En Microsoft Edge, ve a la página Tabular Editor Release.

    ```https://github.com/TabularEditor/TabularEditor/releases```

1. Desplázate hacia abajo hasta la sección **Activos** y selecciona el archivo **TabularEditor.Installer.msi**. Esto iniciará la instalación del archivo.

1. Tras finalizar, selecciona **Abrir archivo** para ejecutar el instalador.

1. En la ventana de instalación de Tabular Editor, selecciona **Siguiente**.

1. En el paso **Contrato de licencia**, si estás de acuerdo, selecciona **Acepto** y después **Siguiente**.

1. En el paso **Seleccionar carpeta de instalación**, selecciona **Siguiente**.

1. En el paso **Métodos abreviados de la aplicación**, selecciona **Siguiente**.

1. En el paso **Confirmar instalación**, selecciona **Siguiente**.

    *Si se te solicita, selecciona **Sí** para permitir que la aplicación realice cambios.*

1. Cuando se haya completado la instalación, seleccione **Cerrar**.

    *Tabular Editor ya está instalado y registrado como herramienta externa de Power BI Desktop.*

### Configurar Power BI Desktop

En esta tarea abrirás una solución de Power BI Desktop desarrollada previamente.

1. Descargue el [archivo de inicio de Análisis de ventas](https://aka.ms/fabric-optimize-starter) de `https://aka.ms/fabric-optimize-starter` y guárdelo en una ubicación que recuerde.

1. Vaya al archivo descargado y ábralo en Power BI Desktop.

1. Selecciona la pestaña de cinta **Herramientas externas**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image8.png)

1. Observa que puedes iniciar Tabular Editor desde esta pestaña de cinta.

    ![](Images/use-tools-to-optimize-power-bi-performance-image9.png)

    *Más adelante en este ejercicio, usarás el Tabular Editor para trabajar con BPA.*

### Revisión del modelo de datos

En esta tarea revisarás el modelo de datos.

1. En Power BI Desktop, en la parte izquierda, cambia a la vista **Modelo**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image10.png)

2. Usa el diagrama de modelos para revisar el diseño del modelo.

    ![](Images/use-tools-to-optimize-power-bi-performance-image11.png)

    *El modelo consta de ocho tablas de dimensiones y una tabla de hechos. La tabla de hechos **Ventas** almacena los detalles de los pedidos de venta. Es un diseño de esquema de estrella clásico que incluye tablas de dimensiones de copo de nieve (**Categoría** > **Subcategoría** > **Producto**) para la dimensión del producto.*

    *En este ejercicio, usarás BPA para detectar problemas del modelo y corregirlos.*

### Carga de reglas BPA

En esta tarea cargarás reglas BPA.

*Las reglas BPA no se agregan durante la instalación de Tabular Editor. Debes descargarlas e instalarlas.*

1. En la cinta **Herramientas externas**, seleccione **Tabular Editor**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image12.png)

    *Tabular Editor se abre en una nueva ventana y se conecta dinámicamente al modelo de datos hospedado en Power BI Desktop. Los cambios realizados en el modelo en Tabular Editor no se propagan a Power BI Desktop hasta que los guardes.*

2. Para cargar las reglas BPA, selecciona la pestaña **C# Script**.

    *Nota: A esto se le puede denominar la pestaña Advanced Scripting en versiones anteriores de Tabular Editor.*

    ![](Images/use-tools-to-optimize-power-bi-performance-image13.png)

3. Copia y pega el siguiente script:

    ```csharp
    System.Net.WebClient w = new System.Net.WebClient(); 

    string path = System.Environment.GetFolderPath(System.Environment.SpecialFolder.LocalApplicationData);
    string url = "https://raw.githubusercontent.com/microsoft/Analysis-Services/master/BestPracticeRules/BPARules.json";
    string downloadLoc = path+@"\TabularEditor\BPARules.json";
    w.DownloadFile(url, downloadLoc);
    ```

4. Para ejecutar el script, en la barra de herramientas selecciona el comando **Run script**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image14.png)

    *Para usar las reglas BPA, debes cerrar y volver a abrir Tabular Editor.*

5. Cierra Tabular Editor.

6. Para volver a abrir Tabular Editor, en Power BI Desktop, en la cinta **Herramientas externas** selecciona **Tabular Editor**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image15.png)

### Revisión de las reglas BPA

En esta tarea revisarás las reglas BPA que cargaste en la tarea anterior.

1. En Tabular Editor, en el menú, selecciona **Tools** > **Manage BPA Rules**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image16.png)

2. En la ventana **Manage Best Practice Rules**, en la lista **Rule collections**, selecciona **Rules for the local user**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image17.png)

3. En la lista **Rules for the local user**, desplázate hacia abajo en la lista de reglas.

    *Sugerencia: puedes arrastrar la esquina inferior derecha para ampliar la ventana.*

    *En segundos, Tabular Editor puede examinar todo el modelo en cada una de las reglas y proporciona un informe de todos los objetos de modelo que cumplen la condición en cada regla.*

4. Observa que BPA agrupa las reglas en categorías.

    *Algunas reglas, como las expresiones DAX, se centran en la optimización del rendimiento, mientras que otras, como las reglas de formato, están orientadas a la estética.*

5. Observa la columna **Severity**.

    *Cuanto mayor sea el número, más importante será la regla*.

6. Desplázate hasta la parte inferior de la lista y después desactiva la regla **Set IsAvailableInMdx to false on non-attribute columns**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image18.png)

    *Puedes deshabilitar reglas individuales o categorías completas de reglas. BPA no comprobará las reglas deshabilitadas en el modelo. La eliminación de esta regla específica es mostrar cómo deshabilitar una regla.*

7. Seleccione **Aceptar**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image19.png)

### Solución de problemas de BPA

En esta tarea abrirás BPA y revisarás los resultados de las comprobaciones.

1. En el menú, selecciona **Tools** > **Best Practice Analyzer** (o presiona **F10).**

    ![](Images/use-tools-to-optimize-power-bi-performance-image20.png)

2. Si fuera necesario, maximiza la ventana **Best Practice Analyzer**.

3. Observa la lista de problemas (posibles), agrupados por categoría.

4. En la primera categoría, haz clic con el botón derecho en la tabla **"Product"** y luego selecciona **Ignore item**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image21.png)

    *Cuando un problema no es realmente un problema, puedes omitir ese elemento. Siempre puedes mostrar elementos omitidos mediante el comando **Show ignored** en la barra de herramientas.*

5. Más abajo en la lista, en la categoría **Use the DIVIDE function for division**, haz clic con el botón derecho en **[Profit Margin]** y después selecciona **Go to object**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image22.png)

    *Este comando cambia a Tabular Editor y se centra en el objeto. Facilita la aplicación de una corrección al problema.*

6. En el Editor de expresiones, modifica la fórmula DAX para usar la función [DIVIDE](https://docs.microsoft.com/dax/divide-function-dax) más eficaz (y segura), como se indica a continuación.

    *Sugerencia: Todas las fórmulas están disponibles para copiar y pegar desde **D:\fabric\Allfiles\Labs\16\Snippets.txt**.*

    ```dax
    DIVIDE ( [Profit], SUM ( 'Sales'[Sales Amount] ) )
    ```

7. Para guardar los cambios del modelo, en la barra de herramientas, selecciona el comando **Guardar cambios en la base de datos conectada** (o presiona **Ctrl+S**).

    ![](Images/use-tools-to-optimize-power-bi-performance-image23.png)

    *Al guardar los cambios, se insertan modificaciones en el modelo de datos de Power BI Desktop.*

8. Vuelve a la ventana **Best Practice Analyzer** (fuera del foco).

9. Observa que BPA ya no enumera el problema.

10. Desplázate hacia abajo en la lista de problemas para buscar la categoría **Provide format string for "Date" columns**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image24.png)

11. Haz clic con el botón derecho en el problema **"Date"[Date]** y selecciona **Generate fix script**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image25.png)

    *Este comando genera un script de C# y lo copia en el Portapapeles. También puedes usar el comando **Generate fix script** para generar y ejecutar el script. Sin embargo, podría ser más seguro revisar (y modificar) el script antes de ejecutarlo.*

12. Cuando se notifique que BPA ha copiado el script de corrección en el Portapapeles, selecciona **OK**.

13. Cambia a Tabular Editor y selecciona la pestaña **C# Script**.

    *Nota: A esto se le puede denominar la pestaña Scripting avanzada en versiones anteriores de Tabular editor.*

    ![](Images/use-tools-to-optimize-power-bi-performance-image13.png)

14. Para pegar el script de corrección, haz clic con el botón derecho en el panel y presiona **Ctrl+V**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image27.png)

    *Puedes optar por realizar un cambio en la cadena de formato.*

15. Para ejecutar el script, en la barra de herramientas selecciona el comando **Run script**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image14.png)

16. Guarda los cambios de modelo.

17. Para cerrar Tabular Editor, en el menú, selecciona **Archivo** > **Salir**.

18. Guarde el archivo de Power BI Desktop.

    ![](Images/use-tools-to-optimize-power-bi-performance-image29.png)

    *También debes guardar el archivo de Power BI Desktop para asegurarte de que se guardan los cambios de Tabular Editor.*

    *En el mensaje sobre los cambios pendientes, seleccione **Aplicar más adelante**.*

## Uso de DAX Studio

En este ejercicio, usará DAX Studio para optimizar las consultas DAX en el archivo de informe de Power BI.

*Según su sitio web, DAX Studio es “la herramienta definitiva para ejecutar y analizar consultas DAX en modelos tabulares de Microsoft”. Es una herramienta enriquecida para la creación, el diagnóstico, el ajuste del rendimiento y el análisis de DAX. Entre las características se incluyen la exploración de objetos, el seguimiento integrado, los desgloses de ejecución de consultas con estadísticas detalladas, resaltado de sintaxis DAX y formato.*

### Descargar DAX Studio

En esta tarea descargarás DAX Studio.

1. En Microsoft Edge, ve a la página de descargas de DAX Studio.

    ```https://daxstudio.org/downloads/```

1. Seleccione **DaxStudio_3_X_XX_setup.exe (instalador)**; esto iniciará la instalación del archivo.
    *Nota: La versión de DAX Studio cambiará ligeramente con el tiempo. Descarga la versión más reciente.*

1. Tras finalizar, selecciona **Abrir archivo** para ejecutar el instalador.

    ![Interfaz gráfica de usuario/Descripción generada automáticamente de la aplicación.](Images/use-tools-to-optimize-power-bi-performance-image31b.png)

1. En la ventana del instalador de DAX Studio, selecciona **Instalar para todos los usuarios (recomendado).**

1. En la ventana Control de cuentas de usuario, selecciona Sí para permitir que la aplicación realice cambios en el dispositivo.

    ![Interfaz gráfica de usuario/Descripción generada automáticamente de la aplicación.](Images/use-tools-to-optimize-power-bi-performance-image31c.png)

1. En el paso **Contrato de licencia**, si aceptas los términos de la licencia, selecciona **Acepto el contrato** y selecciona **Siguiente**.

    ![Interfaz gráfica de usuario/Descripción generada automáticamente de la aplicación.](Images/use-tools-to-optimize-power-bi-performance-image31d.png)

1. Selecciona **Siguiente** para usar la ubicación de destino predeterminada.
1. Elige **Siguiente** para seleccionar los componentes predeterminados que deseas instalar.
1. Selecciona **Siguiente** para colocar el acceso directo en la carpeta de menú inicio predeterminada.
1. Selecciona **Crear un acceso directo de escritorio** y selecciona siguiente.

    ![Interfaz gráfica de usuario/Descripción generada automáticamente de la aplicación.](Images/use-tools-to-optimize-power-bi-performance-image31e.png)
1. Seleccione **Instalar**.

1. Tras finalizar, con **Iniciar DAX Studio** seleccionado, selecciona **Finalizar**. Se abrirá DAX Studio.
    ![Interfaz gráfica de usuario/Descripción generada automáticamente de la aplicación.](Images/use-tools-to-optimize-power-bi-performance-image31f.png)

1. En la ventana **Conectar**, seleccione la opción **Modelo de Power BI/SSDT**.

1. En la lista desplegable correspondiente, asegúrate de que está seleccionado el modelo **Análisis de ventas: uso de herramientas para optimizar el rendimiento de Power BI**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image30.png)

    *Si no tienes abierto el archivo de inicio **Sales Analysis - Use tools to optimize Power BI performance**, no podrás conectarte. Asegúrate de que el archivo está abierto.*

1. Seleccione **Conectar**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image31.png)

1. Si es necesario, maximiza la ventana de DAX Studio.

### Uso de DAX Studio para optimizar una consulta

En esta tarea, optimizarás una consulta mediante una fórmula de medida mejorada.

*Ten en cuenta que es difícil optimizar una consulta cuando los volúmenes del modelo de datos son pequeños. Este ejercicio se centra en el uso de DAX Studio en lugar de optimizar las consultas DAX.*

1. En una ventana del explorador, descargue el archivo [Monthly Profit Growth.dax](https://aka.ms/fabric-optimize-dax) de `https://aka.ms/fabric-optimize-dax` y guárdelo en el equipo local (en cualquier carpeta).

   ![](https://github.com/MicrosoftLearning/mslearn-fabric/assets/34583336/58254cce-753e-4322-9060-536e12554aa7)

3. Cambie a la ventana de DAX Studio y, en el menú **Archivo**, seleccione **Examinar** para ir al archivo **Monthly Profit Growth.dax** y seleccione **Abrir**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image33.png)

6. Lee los comentarios en la parte superior del archivo y después revisa la consulta siguiente.

    *No es importante comprender la consulta en su totalidad.*

    *La consulta define dos medidas que determinan el crecimiento de beneficios mensuales. Actualmente, la consulta solo usa la primera medida (en la línea 72). Cuando no se usa una medida, no afecta a la ejecución de la consulta.*

7. Para ejecutar un seguimiento del servidor y registrar información detallada de tiempo para la generación de perfiles de rendimiento, en la pestaña **Inicio** de la cinta de opciones, en el grupo **Seguimientos**, selecciona **Intervalos de servidor**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image34.png)

8. Para ejecutar el script, en la pestaña **Inicio** de la cinta de opciones, en el grupo **Consulta**, selecciona el icono **Ejecutar**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image35.png)

9. Revisa los resultados de la consulta en el panel inferior.

    *La última columna muestra los resultados de la medida.*

10. En el panel inferior, selecciona la pestaña **Intervalos de servidor**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image36.png)

11. Revisa las estadísticas disponibles en el lado izquierdo.

    ![](Images/use-tools-to-optimize-power-bi-performance-image37.png)

    *Desde la parte superior izquierda hasta la parte inferior derecha, las estadísticas indican cuántos milisegundos tardó en ejecutar la consulta y la duración que tardó la CPU del motor de almacenamiento (SE). En este caso (los resultados variarán), el motor de fórmulas (FE) tardó el 73,5 % del tiempo, mientras que el SE tardó el 26,5 % restante del tiempo. Hubo 34 consultas SE individuales y 21 aciertos de caché.*

12. Vuelve a ejecutar la consulta y observa que todas las consultas SE proceden de la memoria caché de SE.

    *Esto se debe a que los resultados se almacenaron en caché para su reutilización. A veces, en las pruebas, es posible que desees borrar la memoria caché. En ese caso, en la pestaña **Inicio** de la cinta de opciones, selecciona la flecha hacia abajo para el comando **Ejecutar**.*

    ![](Images/use-tools-to-optimize-power-bi-performance-image38.png)

    *La segunda definición de medida proporciona un resultado más eficaz. Ahora actualizarás la consulta para usar la segunda medida.*

13. En la línea 72, reemplaza la palabra **Bad** por **Better**.

    ![](Images/use-tools-to-optimize-power-bi-performance-image39.png)

14. Ejecuta la consulta y revisa las estadísticas de tiempo del servidor.

    ![](Images/use-tools-to-optimize-power-bi-performance-image40.png)

15. Ejecútalo una segunda vez para dar lugar a aciertos de caché completa.

    ![](Images/use-tools-to-optimize-power-bi-performance-image41.png)

    *En este caso, puedes determinar que la consulta "óptima", que usa variables y una función de inteligencia de tiempo, tiene un mejor rendimiento: una reducción de casi el 50 % en el tiempo de ejecución de consultas.*

### Finalización

Para concluir este ejercicio, cierre todas las aplicaciones; no es necesario guardar los archivos.
