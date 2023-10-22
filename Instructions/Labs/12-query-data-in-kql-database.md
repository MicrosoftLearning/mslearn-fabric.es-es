---
lab:
  title: Consulta de datos en una base de datos KQL
  module: Query data from a Kusto Query database in Microsoft Fabric
---
# Introducción a la consulta de una base de datos de Kusto en Microsoft Fabric
Un conjunto de consultas KQL es una herramienta que permite ejecutar consultas, modificar y mostrar los resultados de la consulta desde una base de datos KQL. Puede vincular cada pestaña del conjunto de consultas KQL a una base de datos KQL diferente y guardar las consultas para su uso futuro o compartirlas con otras personas para el análisis de datos. También puede cambiar la base de datos KQL para cualquier pestaña, de forma que pueda comparar los resultados de la consulta de diferentes orígenes de datos.

El conjunto de consultas KQL usa el lenguaje de consulta Kusto, que es compatible con muchas funciones SQL, para crear consultas. Para más información sobre el [lenguaje de consulta kusto (KQL)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext), 

Este laboratorio se tarda aproximadamente **25** minutos en completarse.

## Crear un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. Inicie sesión en [Microsoft Fabric](https://app.fabric.microsoft.com) en `https://app.fabric.microsoft.com` y seleccione **Power BI**.
2. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
3. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
4. Cuando se abra la nueva área de trabajo, estará vacía, como se muestra aquí:

    ![Captura de pantalla de un área de trabajo vacía en Power BI.](./Images/new-workspace.png)

En este laboratorio, usará Real-Time Analytics (RTA) en Fabric para crear una base de datos KQL a partir de una secuencia de eventos de muestra. Real-Time Analytics proporciona un conjunto de datos de ejemplo que puede utilizar para explorar las funcionalidades de RTA. Usará estos datos de muestra para crear consultas KQL | SQL y conjuntos de consultas que analicen algunos datos en tiempo real y permitan un uso adicional en procesos posteriores.

## Creación de una base de datos KQL

1. En **Análisis en tiempo real**, seleccione la casilla **Base de datos KQL**.

   ![Imagen de la elección de kqldatabase](./Images/select-kqldatabase.png)

2. Se le pedirá que asigne un **Nombre** a la base de datos KQL.

   ![Imagen de nombrar kqldatabase](./Images/name-kqldatabase.png)

3. Dele un nombre a la base de datos KQL que sea fácil de recordar, como **MyStockData**, y presione **Crear**.

4. En el panel **Detalles de la base de datos**, seleccione el icono de lápiz para activar la disponibilidad en OneLake.

   ![Imagen de la habilitación de onlake](./Images/enable-onelake-availability.png)

5. Seleccione el cuadro de **datos de ejemplo** en las opciones de ***Inicio obteniendo datos***.
 
   ![Imagen de opciones de selección con datos de ejemplo resaltados](./Images/load-sample-data.png)

6. elija el cuadro **Análisis de métricas** en las opciones de los datos de ejemplo.

   ![Imagen de la elección de datos de análisis para el laboratorio](./Images/create-sample-data.png)

7. Una vez cargados los datos, verifíquelos en la base de datos KQL. Para ello, seleccione los puntos suspensivos situados a la derecha de la tabla, vaya a **Consultar tabla** y seleccione **Mostrar 100 registros cualesquiera**.

    <div><video controls src="./Images/check-kql-sample-dataset.mp4" muted="false" autoplay loop></video></div>

> **NOTA**: La primera vez que ejecute esto, puede tardar varios segundos en asignar recursos de proceso.

## Escenario
En este escenario, es un analista que se encarga de consultar un conjunto de datos de ejemplo que implementará desde el entorno de Fabric.



Una consulta de Kusto es una manera de leer datos, procesarlos y mostrar los resultados. La consulta se escribe en texto sin formato con el que es fácil trabajar. Una consulta de Kusto puede tener una o varias instrucciones que muestren datos como una tabla o un grafo.

Una instrucción de tabla tiene algunos operadores que funcionan en datos de tabla. Cada operador toma una tabla como entrada y proporciona una tabla como salida. Los operadores se unen mediante una pleca (|). Los datos se mueven de un operador a otro. Cada operador cambia los datos de alguna manera y los pasa.

Puede imaginárselo como un embudo, en el que se empieza con una tabla entera de datos. Cada operador filtra, ordena o resume los datos. El orden de los operadores es importante porque funcionan uno después de otro. Al final del embudo, obtendrá una salida final.

Estos operadores son específicos de KQL, pero pueden ser similares a SQL u otros lenguajes.