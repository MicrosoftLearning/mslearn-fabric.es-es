---
lab:
  title: Chatea con tus datos mediante agentes de datos de Microsoft Fabric
  module: Implement Fabric Data Agents
---

# Chatea con tus datos mediante agentes de datos de Microsoft Fabric

Un agente de datos de Microsoft Fabric permite la interacción natural con los datos, ya que le permite formular preguntas en inglés sin formato y recibir respuestas estructuradas y legibles para el usuario. Al eliminar la necesidad de comprender los lenguajes de consulta como SQL (Lenguaje de consulta estructurado), DAX (Expresiones de análisis de datos) o KQL (Lenguaje de consulta Kusto), el agente de datos hace que la información de los datos sea accesible en toda la organización, independientemente del nivel de aptitud técnica.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

## Temas que se abordarán

Al completar este laboratorio, aprenderá a:

- Comprender el propósito y las ventajas de los agentes de datos de Microsoft Fabric para el análisis de datos de lenguaje natural.
- Crear y configurar un área de trabajo de Fabric y un almacenamiento de datos.
- Obtener experiencia práctica mediante la carga y exploración un conjunto de datos de ventas de esquema de estrella.
- Ver cómo los agentes de datos traducen preguntas en inglés sin formato en consultas SQL.
- Desarrollar aptitudes para formular preguntas analíticas eficaces e interpretar los resultados generados por la inteligencia artificial.
- Crear confianza en el aprovechamiento de las herramientas de inteligencia artificial para democratizar el acceso a los datos y la información.

## Antes de comenzar

Necesita una [Capacidad de Microsoft Fabric (F2 o superior)](https://learn.microsoft.com/fabric/fundamentals/copilot-enable-fabric) con Copilot habilitado para completar este ejercicio.

## Escenario del ejercicio

En este ejercicio, se creará un almacenamiento de datos de ventas, se le cargarán algunos datos y, después, se creará un agente de datos de Fabric. Luego, le formulará varias preguntas y explorará cómo el agente de datos traduce el lenguaje natural en consultas SQL para proporcionar información. Este enfoque práctico demostrará la eficacia del análisis de datos asistido por IA sin necesidad de conocimientos profundos de SQL. En primer lugar,

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con Fabric habilitado. Un área de trabajo de Microsoft Fabric actúa como un entorno de colaboración donde puede organizar y administrar todos los artefactos de ingeniería de datos, incluidos almacenes de lago de datos, cuadernos y conjuntos de datos. Imagine que es como una carpeta de proyecto que contiene todos los recursos necesarios para el análisis de datos.

1. En un explorador, ve a la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) en `https://app.fabric.microsoft.com/home?experience=fabric` e inicia sesión con tus credenciales de Fabric.

1. En la barra de menús de la izquierda, selecciona **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).

1. Cree una área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Premium* o *Fabric*). Tenga en cuenta que *Versión de evaluación* no se admite.
   
    > **Por qué esto importa**: Copilot necesita una capacidad de Fabric de pago para funcionar. Esto garantiza que tiene acceso a las características con tecnología de inteligencia artificial que ayudarán a generar código a lo largo de este laboratorio.

1. Cuando se abra la nueva área de trabajo, debe estar vacía.

![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

## Creación del almacenamiento de datos

Ahora que tiene un área de trabajo, es el momento de crear un almacenamiento de datos. Un almacenamiento de datos es un repositorio centralizado que almacena datos estructurados de varios orígenes, optimizados para consultas analíticas e informes. En este caso, se creará un almacenamiento de datos de ventas simple que servirá como base para las interacciones del agente de datos. Busque el acceso directo para crear un almacén:

1. En la barra de menús de la izquierda, selecciona **Crear**. En la página *Nuevo*, en la sección *Almacenamiento de datos*, selecciona **Almacén**. Asígnale un nombre único que elijas.

    >**Nota**: si la opción **Crear** no está anclada a la barra lateral, primero debes seleccionar la opción de puntos suspensivos (**...**).

    Al cabo de un minuto más o menos, se creará un nuevo almacén:

    ![Captura de pantalla de un nuevo almacenamiento.](./Images/new-data-warehouse2.png)

## Creación de tablas e inserción de datos

Un almacenamiento es una base de datos relacional en la que se pueden definir tablas y otros objetos. Para que el agente de datos sea útil, es necesario rellenarlo con datos de ventas de ejemplo. El script que ejecutará crea un esquema de almacenamiento de datos típico con tablas de dimensiones (que contienen atributos descriptivos) y una tabla de hechos (que contiene eventos empresariales medibles). Este diseño de esquema de estrella está optimizado para consultas analíticas que generará el agente de datos.

1. En la pestaña del menú **Inicio**, use el botón **Nueva consulta SQL** para crear una nueva consulta. A continuación, copie y pegue el código de Transact-SQL desde `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-fabric/refs/heads/main/Allfiles/Labs/22d/create-dw.txt` en el nuevo panel de consulta.

    > **Funcionamiento del script**: El script crea un almacenamiento de datos de ventas completo con información del cliente, detalles del producto, dimensiones de fecha y transacciones de ventas. Este conjunto de datos realista le permitirá formular preguntas empresariales significativas al agente de datos.

1. Ejecute la consulta, que crea un esquema de almacenamiento de datos simple y carga algunos datos. El script debe tardar unos 30 segundos en ejecutarse.

1. Use el botón **Actualizar** de la barra de herramientas para actualizar la vista. A continuación, en el panel **Explorador**, compruebe que el esquema **dbo** del almacenamiento de datos contiene ahora las cuatro tablas siguientes:
   
    - **DimCustomer**: contiene información del cliente, incluidos los nombres, las ubicaciones y los detalles de contacto
    - **DimDate**: contiene atributos relacionados con la fecha, como años fiscales, trimestres y meses para el análisis basado en el tiempo
    - **DimProduct**: contiene información del producto, incluidos nombres, categorías y precios
    - **FactSalesOrder**: contiene las transacciones de ventas reales que vinculan clientes, productos y fechas

    > **Sugerencia**: Si el esquema tarda un tiempo en cargarse, actualice la página del explorador.

## Creación de un agente de datos de Fabric

Un agente de datos de Fabric es un asistente con tecnología de inteligencia artificial que puede comprender preguntas en lenguaje natural sobre los datos y generar automáticamente las consultas adecuadas para responderlas. Esto elimina la necesidad de que los usuarios conozcan la sintaxis SQL, KQL o DAX, a la vez que proporcionan información precisa y controlada por datos. Ahora se creará y configurará el agente de datos:

1. Cree un agente de datos.
   
    ![Recorte de pantalla de la creación de un agente de datos](./Images/copilot-fabric-data-agent-new.png)

1. Asígnele un nombre como **`sales-data-agent`**.

    > **Por qué es importante la nomenclatura**: Un nombre descriptivo le ayuda a usted y al equipo a comprender el propósito y el ámbito de este agente de datos, especialmente cuando se administran varios agentes para distintos dominios de datos.
    
    ![Recorte de pantalla de la creación de nuevo agente de datos y la asignación de un nombre.](./Images/copilot-fabric-data-agent-create.png)

1. Seleccione **Agregar un origen de datos**. 

    ![Recorte de pantalla del agente de datos creado.](./Images/copilot-fabric-data-agent-created.png)

1. Elija el almacenamiento de datos que ha creado antes.

    > **Conexión a los datos**: El agente de datos necesita acceso a las tablas para comprender el esquema y las relaciones. Esto le permite generar consultas SQL precisas en función de las preguntas.

1. Expanda el almacenamiento de datos y seleccione **DimCustomer**, **DimDate**, **DimProduct** y **FactSalesOrder**.

    > **Estrategia de selección de tablas**: Al seleccionar las cuatro tablas, se proporciona al agente de datos acceso al modelo de datos completo. Esto le permite responder a preguntas complejas que abarcan varias tablas, como tendencias de ventas por ubicación del cliente o rendimiento del producto en el tiempo.

    ![Recorte de pantalla de las tablas de almacenamiento del agente de datos seleccionadas.](./Images/copilot-fabric-data-agent-select-tables.png)

## Realizar preguntas

Es el momento de empezar a experimentar con el agente de datos y formularle preguntas. En esta sección se muestra cómo se puede transformar el lenguaje natural en consultas SQL, lo que hace que el análisis de datos sea accesible para los usuarios sin conocimientos técnicos de SQL. Cada pregunta le mostrará tanto la respuesta como la consulta subyacente que se genera.

1. Para formular una pregunta, escriba el siguiente mensaje: 

    ```copilot-prompt
    How many products did we sell by fiscal year?
    ```

    Anote la respuesta resultante: Se han vendido un total de 12 630 productos en el año fiscal 2021 y de 13 336 productos en el año fiscal 2022.

1. Expanda el paso completado y su subpaso. Esto revela la consulta SQL generada por el agente de datos para responder a la pregunta.

    > **Oportunidad de aprendizaje**: Mediante el examen del código SQL generado, puede comprender cómo ha interpretado el agente de datos la pregunta y obtener información sobre las relaciones de datos subyacentes. Esta transparencia crea confianza en los resultados generados por la inteligencia artificial.
    
    ![Recorte de pantalla de los pasos de consulta del agente de datos explicados](./Images/copilot-fabric-data-agent-query-1-explanation.png)
    
    Copilot ha generado el siguiente código SQL, que puede diferir ligeramente en función del entorno y de las actualizaciones más recientes de Copilot.
    
    ```sql
    SELECT d.Year, SUM(f.Quantity) AS TotalProductsSold
    FROM dbo.FactSalesOrder f
    JOIN dbo.DimDate d ON f.SalesOrderDateKey = d.DateKey
    GROUP BY d.Year
    ORDER BY d.Year;
    ```

    > **Explicación de SQL**: Esta consulta combina la tabla de hechos (FactSalesOrder) con la dimensión de fecha (DimDate) para agrupar las ventas por año y sumar las cantidades. Observe cómo el agente de datos ha entendido automáticamente que "productos vendidos" hace referencia al campo Quantity y "año fiscal" se asigna al campo Year en la dimensión de fecha.

1. Continúe con la pregunta siguiente: 

    ```copilot-prompt
    What are the top 10 most popular products all time?
    ```

    > **Qué puede esperar**: Esta pregunta demostrará cómo el agente de datos puede realizar operaciones de clasificación, unir información de productos con datos de ventas para identificar los más vendidos.

1. Siga con esta pregunta: 

    ```copilot-prompt
    What are the historical trends across all my data?
    ```

    > **Análisis avanzado**: Esta pregunta más amplia mostrará cómo el agente de datos puede proporcionar análisis de tendencias en varias dimensiones, lo que podría incluir patrones basados en el tiempo en las ventas, el comportamiento del cliente y el rendimiento del producto.

1. Pruebe otras preguntas para explorar diferentes aspectos de los datos:

    ```copilot-prompt
    In which countries are our customers located?
    ```
    
    ```copilot-prompt
    How many products did we sell in the United States?
    ```
    
    ```copilot-prompt
    How much revenue did we make in FY 2022?
    ```
    
    ```copilot-prompt
    How much was our total sales revenue, by fiscal year, fiscal quarter and month name?
    ```

    > **Sugerencia profesional**: Cada una de estas preguntas tiene como destino diferentes escenarios analíticos: análisis geográfico, agregaciones filtradas, cálculos de ingresos y análisis de tiempo jerárquico. Experimente con variaciones para ver cómo se adapta el agente de datos a diferentes estilos de pregunta.

## Comprensión de la estructura de los datos

A medida que experimente con las preguntas, tenga en cuenta estas características de datos para formular preguntas más específicas:

- **Cronología del año fiscal**: El año fiscal comienza en julio (el mes 7). Por tanto, Q1 va de julio-a septiembre, Q2 de octubre-a diciembre, Q3 de enero a marzo y Q4 de abril a junio.

- **Identificación del cliente**: El campo CustomerAltKey contiene direcciones de correo electrónico del cliente, que pueden ser útiles para consultas específicas del cliente.

- **Moneda**: Todos los precios de lista y los totales de ventas se expresan en GBP (libras esterlina).

- **Relaciones de datos**: La tabla FactSalesOrder conecta clientes, productos y fechas mediante claves externas, lo que permite un análisis multidimensional complejo.

> **Experimentos adicionales**: Pruebe a formular preguntas que combinen estos elementos, como "¿Cuáles fueron los ingresos en el primer trimestre de 2022?" o bien, "¿Qué clientes del Reino Unido compraron los productos más caros?" El agente de datos controlará automáticamente las combinaciones complejas y los cálculos necesarios para responder a estas preguntas.

## Resumen

¡Enhorabuena! Ha realizado correctamente las acciones siguientes:

- **Creado un área de trabajo de Fabric** y un almacenamiento de datos con un conjunto de datos de ventas realista
- **Creado y configurado un agente de datos** que puede comprender preguntas en lenguaje natural sobre los datos
- **Experimentado el análisis de datos con tecnología de inteligencia artificial** mediante la formulación de preguntas en inglés sin formato y comprobando cómo se traducen en consultas SQL
- **Ha explorado varios tipos de preguntas analíticas** de agregaciones simples a análisis de tendencias complejos

### Puntos clave

- **Acceso a datos democratizado**: Los agentes de datos hacen que los análisis sean accesibles para los usuarios independientemente de sus conocimientos de SQL
- **Transparencia y confianza**: Siempre puede inspeccionar el código SQL generado para comprender cómo se responden las preguntas
- **Flexibilidad del lenguaje natural**: La inteligencia artificial puede controlar variaciones en las expresiones e incluso errores tipográficos menores en las preguntas
- **Generación de consultas complejas**: El agente controla automáticamente combinaciones, agregaciones y filtros en función de la entrada de lenguaje natural

### Pasos siguientes

Considere la posibilidad de explorar lo siguiente:

- **Instrucciones personalizadas**: Agregue contexto específico de la empresa para mejorar las respuestas del agente de datos
- **Orígenes de datos adicionales**: Conecte más tablas o conjuntos de datos para ampliar el conocimiento del agente
- **Preguntas avanzadas**: Pruebe escenarios analíticos más complejos que impliquen varios períodos de tiempo, segmentos de clientes o categorías de productos
- **Integración**: Inserte información del agente de datos en informes, paneles o aplicaciones empresariales

El agente de datos de Fabric representa un paso importante para hacer que la información de los datos sea realmente accesible en toda la organización, lo que reduce la brecha entre los datos y la toma de decisiones.
