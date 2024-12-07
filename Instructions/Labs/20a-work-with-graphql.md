---
lab:
  title: Trabajar con API para GraphQL en Microsoft Fabric
  module: Get started with GraphQL in Microsoft Fabric
---

# Trabajar con API para GraphQL en Microsoft Fabric

La API de Microsoft Fabric para GraphQL es una capa de acceso a datos que permite realizar consultas rápidas y eficaces de varios orígenes de datos con una tecnología de API ampliamente adoptada y conocida. La API permite abstraer los detalles de los orígenes de datos de back-end para que pueda concentrarse en la lógica de la aplicación y proporcionar todos los datos que necesita un cliente en una sola llamada. GraphQL usa un lenguaje de consulta simple y conjuntos de resultados fáciles de manipular, lo que minimiza el tiempo que tardan las aplicaciones en acceder a los datos en Fabric.

Este laboratorio se tarda aproximadamente **30** minutos en completarse.

> **Nota**: necesitarás una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, crea un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. En la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) en `https://app.fabric.microsoft.com/home?experience=fabric`.
1. En la barra de menús de la izquierda, selecciona **Nuevo espacio de trabajo**.
1. Crea una nueva área de trabajo con el nombre que prefieras y selecciona un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Premium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

## Creación de una base de datos con datos de ejemplo

Ahora que tienes un área de trabajo, es el momento de crear una base de datos SQL.

1. En el portal de Fabric, selecciona **+ Nuevo elemento** en el panel izquierdo.
1. Ve a la sección **Almacenar datos** y selecciona **SQL Database**.
1. Escribe **AdventureWorksLT** como nombre de la base de datos y selecciona **Crear**.
1. Una vez que hayas creado la base de datos, puedes cargar datos de ejemplo en la base de datos desde la tarjeta **Datos de ejemplo**.

    Después de un minuto, la base de datos se rellenará con datos de ejemplo para tu escenario.

    ![Captura de pantalla de una nueva base de datos cargada con datos de ejemplo.](./Images/sql-database-sample.png)

## Consulta de una base de datos SQL

El editor de consultas SQL proporciona compatibilidad con IntelliSense, finalización de código, resaltado de sintaxis y análisis, y validación del lado cliente. Puedes ejecutar instrucciones del lenguaje de definición de datos (DDL), el lenguaje de manipulación de datos (DML) y el lenguaje de control de datos (DCL).

1. En la página base de datos **AdventureWorksLT**, ve a **Inicio** y selecciona **Nueva consulta**.
1. En el nuevo panel de consulta en blanco, escribe y ejecuta el siguiente código de T-SQL.

    ```sql
    SELECT 
        p.Name AS ProductName,
        pc.Name AS CategoryName,
        p.ListPrice
    FROM 
        SalesLT.Product p
    INNER JOIN 
        SalesLT.ProductCategory pc ON p.ProductCategoryID = pc.ProductCategoryID
    ORDER BY 
    p.ListPrice DESC;
    ```
    
    Esta consulta combina las tablas `Product` y `ProductCategory` para mostrar los nombres de producto, sus categorías y sus precios de lista, ordenados por precio en orden descendente.

1. Cierra todas las pestañas de consulta.

## Creación de una API para GraphQL

En primer lugar, configurarás un punto de conexión de GraphQL para exponer los datos de pedidos de ventas. Este punto de conexión te permitirá consultar pedidos de ventas en función de varios parámetros, como la fecha, el cliente y el producto.

1. En el portal de Fabric, ve a tu espacio de trabajo y selecciona **+ Nuevo elemento**.
1. Ve a la sección **Desarrollo de datos** y selecciona **API para GraphQL**.
1. Proporciona un nombre y selecciona **Crear**.
1. En la página principal de la API para GraphQL, selecciona **Seleccionar origen de datos**.
1. Si se te pide que elijas una opción de conectividad, selecciona **Conectar a orígenes de datos de Fabric con autenticación de inicio de sesión único (SSO)**.
1. En la página **Elegir los datos que desea conectar**, selecciona la base de datos `AdventureWorksLT` que has creado anteriormente.
1. Seleccione **Conectar**.
1. En la página **Elegir datos**, selecciona la tabla `SalesLT.Product`. 
1. Obtén una vista preliminar de los datos y selecciona **Cargar**.
1. Selecciona **Copiar punto de conexión** y anota el vínculo de dirección URL pública. No lo necesitamos, pero aquí es donde vas a copiar la dirección de tu API.

## Deshabilitar mutaciones

Ahora que se ha creado nuestra API, solo queremos exponer los datos de ventas para las operaciones de lectura de este escenario.

1. En el **Explorador de esquemas** de tu API para GraphQL, expande **Mutaciones**.
1. Selecciona los **...** (puntos suspensivos) junto a cada mutación y selecciona **Deshabilitar**.

Esto impedirá cualquier modificación o actualización de los datos a través de la API. Esto significa que los datos serán de solo lectura y los usuarios solo podrán ver o consultar los datos, pero no realizar ningún cambio en ellos.

## Consulta de datos mediante GraphQL

Ahora, vamos a consultar los datos mediante GraphQL para buscar todos los productos cuyos nombres comienzan por *"HL Road Frame".*

1. En el editor de consultas de GraphQL, escribe y ejecuta la siguiente consulta.

```json
query {
  products(filter: { Name: { startsWith: "HL Road Frame" } }) {
    items {
      ProductModelID
      Name
      ListPrice
      Color
      Size
      ModifiedDate
    }
  }
}
```

En esta consulta, productos es el tipo principal, e incluye campos para `ProductModelID`, `Name`, `ListPrice`, `Color`, `Size` y `ModifiedDate`. Esta consulta devolverá una lista de productos cuyos nombres comienzan por *"HL Road Frame".*

> **Más información**: consulta [¿Qué es la API de Microsoft Fabric para GraphQL?](https://learn.microsoft.com/fabric/data-engineering/api-graphql-overview) en la documentación de Microsoft Fabric para obtener más información sobre otros componentes disponibles en la plataforma.

En este ejercicio, has creado, consultado y expuesto datos de una base de datos SQL mediante GraphQL en Microsoft Fabric.

## Limpieza de recursos

Si has terminado de explorar la base de datos, puedes eliminar el área de trabajo que has creado para este ejercicio.

1. En la barra de la izquierda, selecciona el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, selecciona **Configuración del área de trabajo**.
3. En la sección **General**, selecciona **Quitar esta área de trabajo**.

