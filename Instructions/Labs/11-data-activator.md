---
lab:
  title: Uso de Data Activator en Fabric
  module: Get started with Data Activator in Microsoft Fabric
---

# Uso de Data Activator en Fabric

Data Activator de Microsoft Fabric realiza acciones en función de lo que sucede en los datos. Data Activator permite supervisar los datos y crear desencadenadores para reaccionar ante los cambios de datos.

Este laboratorio se realiza en unos **30** minutos.

> **Nota**: Necesitará una [evaluación gratuita de Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) para realizar este ejercicio.

## Creación de un área de trabajo

Antes de trabajar con datos de Fabric, cree un área de trabajo con la evaluación gratuita de Fabric habilitada.

1. En la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com), seleccione **Data Activator**.
1. En la barra de menús de la izquierda, seleccione **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
1. Cree una nueva área de trabajo con el nombre que prefiera y seleccione un modo de licencia que incluya capacidad de Fabric (*Evaluación gratuita*, *Prémium* o *Fabric*).
1. Cuando se abra la nueva área de trabajo, debe estar vacía.

    ![Captura de pantalla de un área de trabajo vacía en Fabric.](./Images/new-workspace.png)

En este laboratorio, usará Data Activator en Fabric para crear un *Reflex*. Data Activator proporciona un conjunto de datos de ejemplo que puede utilizar para explorar las funcionalidades de Data Activator. Usará estos datos de ejemplo para crear un *Reflex* que analice algunos datos en tiempo real y cree un desencadenador para enviar un correo electrónico cuando se cumpla una condición.

> **Nota**: el proceso de ejemplo de Data Activator genera algunos datos aleatorios en segundo plano. Cuanto más complejas sean las condiciones y los filtros, más tiempo se tarda en desencadenarlos. Si no ve ningún dato en el gráfico, espere unos minutos y actualice la página. Dicho esto, no es necesario esperar a que los datos se muestren en los gráficos para continuar con el laboratorio.

## Escenario

En este escenario, es analista de datos de una empresa que vende y distribuye una gama de productos.  Es responsable de los datos de todos los envíos y ventas a la ciudad de Redmond. Desea crear un Reflex que controle los paquetes que están pendientes de entrega. Una categoría de productos que usted envía son las recetas médicas que necesitan estar refrigeradas a una temperatura determinada durante el tránsito. Desea crear un Reflex que envíe un correo electrónico al departamento de envíos si la temperatura de un paquete que contiene una receta es superior o inferior a un determinado umbral. La temperatura ideal debe estar comprendida entre 33 y 41 grados. Dado que los eventos Reflex ya contienen un desencadenador similar, se crea uno específico para los paquetes enviados a la ciudad de Redmond. ¡Empecemos!

## Creación de un Reflex

1. Asegúrese de que está en la pantalla de inicio de Data Activator comprobando que el icono de la parte inferior derecha refleja Data Activator. Vamos a crear un nuevo Reflex seleccionando el botón **Reflex (versión preliminar)** .

    ![Captura de pantalla de la pantalla Inicio de Data Activator.](./Images/data-activator-home-screen.png)

1. En un entorno de producción real, usaría sus propios datos. Sin embargo, para este laboratorio, se utilizan los datos de ejemplo que proporciona Data Activator. Seleccione el botón **Usar datos de ejemplo** para terminar de crear su Reflex.

    ![Captura de pantalla de la pantalla Obtener datos de Data Activator.](./Images/data-activator-get-started.png)

1. De forma predeterminada, Data Activator crea su Reflex con el nombre *Reflex AAAA-MM-DD hh:mm:ss*. Dado que puede tener varios Reflex en el área de trabajo, debe cambiar el nombre predeterminado de Reflex por uno más descriptivo. Seleccione la lista desplegable junto al nombre de Reflex actual en la esquina superior izquierda y cambie el nombre a ***Reflex Envío Contoso*** para nuestro ejemplo.

    ![Captura de pantalla de la pantalla Inicio de Reflex de Data Activator.](./Images/data-activator-reflex-home-screen.png)

Nuestro Reflex ahora se crea y podemos empezar a agregar desencadenadores y acciones a él.

## Familiarizarse con la pantalla Inicio de Reflex

La pantalla Inicio de Reflex se divide en dos secciones, el modo *Diseño* del modo *Datos*. Para seleccionar el modo, seleccione la pestaña correspondiente en la parte inferior izquierda de la pantalla.  La pestaña del modo *Diseño* es donde se definen los objetos con los desencadenadores, las propiedades y los eventos. La pestaña del modo *Datos* es donde puede agregar los orígenes de datos y ver los datos que procesa Reflex. Echemos un vistazo a la pestaña del modo *Diseño*, que debe abrirse de forma predeterminada al crear Reflex.

### Modo de diseño

Si no está actualmente en modo *Diseño*, seleccione la pestaña **Diseño** en la parte inferior izquierda de la pantalla.

![Captura de pantalla del modo Diseño de Reflex de Data Activator.](./Images/data-activator-design-tab.png)

Para familiarizarse con el modo *Diseño*, seleccione las distintas secciones de la pantalla, desencadenadores, propiedades y eventos. En las secciones siguientes se explica cada sección con más detalle.

### Modo de datos

Si no está actualmente en modo *Datos*, seleccione la pestaña **Datos** en la parte inferior izquierda de la pantalla. En un ejemplo real, agregaría sus propios orígenes de datos desde los objetos visuales de EventStreams y Power BI aquí. Para este laboratorio, utiliza los datos de ejemplo que proporciona Data Activator. Este ejemplo ya está configurado con tres EventStreams que supervisan el estado de entrega del paquete.

![Captura de pantalla del modo Datos de Reflex de Data Activator.](./Images/data-activator-data-tab.png)

Seleccione cada uno de los distintos eventos y observe los datos que se usan en la secuencia.

![Captura de pantalla de los eventos del modo Datos de Reflex de Data Activator.](./Images/data-activator-get-data-tab-event-2.png)

Es el momento de agregar un desencadenador a nuestro Reflex, pero primero, vamos a crear un nuevo objeto.

## Creación de un objeto 

En un escenario real, es posible que no sea necesario crear un nuevo objeto para este Reflex, ya que el ejemplo de Data Activator ya incluye un objeto denominado *Package*. Pero para este laboratorio, creamos un nuevo objeto para demostrar cómo crear uno. Vamos a crear un nuevo objeto denominado *Redmond Packages*.

1. Si no está actualmente en modo *Datos*, seleccione la pestaña **Datos** en la parte inferior izquierda de la pantalla.

1. Seleccione el evento ***Paquete en tránsito***. Preste mucha atención a los valores de las columnas *PackageId*, *Temperatura*, *ColdChainType*, *Ciudad* y *SpecialCare*. Estas columnas se usarán para crear el desencadenador.

1. Si el cuadro de diálogo *Asignar datos* aún no está abierto en el lado derecho, seleccione el botón **Asignar datos** a la derecha de la pantalla.

    ![Captura de pantalla del botón Asignar datos del modo Datos de Reflex de Data Activator.](./Images/data-activator-data-tab-assign-data-button.png)

1. En el cuadro de diálogo *Asignar datos*, seleccione la pestaña ***Asignar a un nuevo objeto*** y escriba los valores siguientes:

    - **Nombre de objeto**: *Paquetes de Redmond*
    - **Asignar columna de clave**: *PackageId*
    - **Asignar propiedades**: *City, ColdChainType, SpecialCare, Temperature*

    ![Captura de pantalla del cuadro de diálogo Asignar datos del modo Datos de Reflex de Data Activator.](./Images/data-activator-data-tab-assign-data.png)

1. Seleccione **Guardar** y, a continuación, seleccione **Guardar y vaya al modo de diseño**.

1. Ahora debería volver al modo *Diseño*. Se ha agregado un nuevo objeto denominado ***Redmond Packages***. Seleccione este nuevo objeto, expanda sus *Eventos* y seleccione el evento **Paquete en tránsito**.

    ![Captura de pantalla del modo Diseño de Reflex de Data Activator con el nuevo objeto.](./Images/data-activator-design-tab-new-object.png)

Tiempo para crear el desencadenador.

## Crear un desencadenador

Vamos a revisar lo que desea que haga el desencadenador: *Quiere crear un Reflex que envíe un correo electrónico al departamento de envío si la temperatura de un paquete que contiene una receta es superior o inferior a un umbral determinado. La temperatura ideal debe estar comprendida entre 33 y 41 grados. Dado que los eventos Reflex ya contienen un desencadenador similar, creará uno específicamente para los paquetes enviados a la ciudad de Redmond*.

1. En el evento *Paquete en tránsito* del objeto **Redmond Packages**, seleccione el botón **Nuevo desencadenador** en el menú superior. Se crea un nuevo desencadenador con el nombre predeterminado *Sin título*, cambie el nombre a ***Temperatura de los medicamentos fuera del intervalo*** para definir mejor el desencadenador.

    ![Captura de pantalla de la creación de un nuevo desencadenador de Diseño de Reflex de Data Activator.](./Images/data-activator-trigger-new.png)

1. Tiempo para seleccionar la propiedad o la columna de evento que desencadena su Reflex. Puesto que creó varias propiedades al crear el objeto, seleccione el botón **Propiedad existente** y seleccione la propiedad ***Temperature***. 

    ![Captura de pantalla de la selección de una propiedad de Diseño de Reflex de Data Activator.](./Images/data-activator-trigger-select-property.png)

    Al seleccionar esta propiedad, se debe devolver un gráfico con valores de temperatura históricos de ejemplo.

    ![Captura de pantalla del gráfico de propiedades de valores históricos de Data Activator.](./Images/data-activator-trigger-property-sample-graph.png)

1. Ahora debe decidir qué tipo de condición desea desencadenar desde esta propiedad. En este caso, quiere desencadenar su Reflex cuando la temperatura sea superior a 41 grados o inferior a 33 grados. Puesto que buscamos un intervalo numérico, seleccione el botón **Numérico** y seleccione la condición **Fuera del intervalo**.

    ![Captura de pantalla de la elección del tipo de condición de Diseño de Reflex de Data Activator.](./Images/data-activator-trigger-select-condition-type.png)

1. Ahora debe especificar los valores de la condición. Escriba ***33*** y ***41*** como valores de intervalo. Como se elige la condición de *intervalo numérico de salida*, el desencadenador se debe activar cuando la temperatura es inferior a *33* o superior a *41* grados.

    ![Captura de pantalla de la introducción de valores de condición de Diseño de Reflex de Data Activator.](./Images/data-activator-trigger-select-condition-define.png)

1. Hasta ahora se ha definido la propiedad y la condición en la que desea que se active el desencadenador, pero eso todavía no incluye todos los parámetros necesarios. Todavía tiene que asegurarse de que el desencadenador solo se activa para la *ciudad* de **Redmond** y para el tipo de *cuidado especial* de **Medicamento**. Vamos a añadir un par de filtros para esas condiciones.  Seleccione el botón **Agregar filtro**, establezca la propiedad en ***City***, establezca la relación en ***Igual*** y escriba ***Redmond*** como valor. A continuación, agregue un nuevo filtro con la propiedad ***SpecialCare***, establézcalo en ***Igual*** y escriba ***Medicine*** como valor.

    ![Captura de pantalla de la adición de filtro de Diseño de Reflex de Data Activator.](./Images/data-activator-trigger-select-condition-add-filter.png)

1. Añadamos un filtro más para asegurarnos de que el medicamento está refrigerado. Seleccione el botón **Agregar filtro**, establezca la propiedad ***ColdChainType***, establézcala en ***Igual*** y escriba ***Refrigerated*** como valor.

    ![Captura de pantalla de la adición de filtro de Diseño de Reflex de Data Activator.](./Images/data-activator-trigger-select-condition-add-filter-additional.png)

1. Casi ha terminado. Solo tiene que definir qué acción desea realizar cuando se activa el desencadenador. En este caso, desea enviar un correo electrónico al departamento de envío. Seleccione el botón **Correo electrónico**.

    ![Captura de pantalla de la acción de adición de Data Activator.](./Images/data-activator-trigger-select-action.png)

1. Escriba los valores siguientes para la acción de correo electrónico:

    - **Enviar a**: la cuenta de usuario actual debería estar seleccionada de forma predeterminada. Con eso es suficiente para este laboratorio.
    - **Asunto**: *el paquete de medicamentos de Redmond se encuentra fuera del intervalo de temperatura aceptable*
    - **Título**: *temperatura demasiado alta o demasiado baja*
    - **Información adicional**: seleccione la propiedad *Temperature* en la lista de casillas.

    ![Captura de pantalla de la acción Definir del activador de datos.](./Images/data-activator-trigger-define-action.png)

1. Seleccione **Guardar** y, a continuación, **Iniciar** en el menú superior.

Ahora ha creado e iniciado un desencadenador en Data Activator.

## Actualización y detención de un desencadenador

El único problema con este desencadenador es que mientras el desencadenador envió un correo electrónico con la temperatura, el desencadenador no envió el *PackageId* del paquete. Vamos a seguir y actualizar el desencadenador para incluir el *PackageId*.

1. Seleccione el evento **Paquetes en tránsito** en el objeto **Redmond Packages** y seleccione **Nueva propiedad** en el menú superior.

    ![Captura de pantalla de la selección de un evento de Data Activator del objeto.](./Images/data-activator-trigger-select-event.png)

1. Vamos a agregar la propiedad **PackageId** seleccionando la columna en el evento *Paquetes en tránsito*. No olvide cambiar el nombre de la propiedad de *Sin título* a *PackageId*.

    ![Captura de pantalla de la creación de propiedad de Data Activator.](./Images/data-activator-trigger-create-new-property.png)

1. Actualicemos la acción del desencadenador. Seleccione el desencadenador **Temperatura de medicamento fuera del intervalo**, desplácese hasta la sección **Acción** de la parte inferior, seleccione **Información adicional** y añada la propiedad **PackageId**. NO seleccione el botón **Guardar** todavía.

    ![Captura de pantalla de la adición de propiedad al desencadenador de Data Activator.](./Images/data-activator-trigger-add-property-existing-trigger.png)

1. Puesto que ha actualizado el desencadenador, la acción correcta debe ser actualizar y no guardar el desencadenador, pero para este laboratorio hacemos lo contrario y seleccionamos el botón **Guardar** en lugar del botón **Actualizar** para ver también qué sucede. La razón por la que debería haber seleccionado el botón *Actualizar* es porque cuando selecciona *actualizar* el desencadenador, este guarda el desencadenador y actualiza el desencadenador que se está ejecutando actualmente con las nuevas condiciones. Si simplemente selecciona el botón *Guardar*, el desencadenador que actualmente en ejecución no aplica las nuevas condiciones hasta que seleccione actualizar el desencadenador. Vamos a seguir y seleccionar el botón **Guardar**.

1. Al haber seleccionado *Guardar *, en lugar de *Actualizar*, ha observado que el mensaje *Hay una actualización de la propiedad disponible. Realícela ahora para asegurarse de que el desencadenador tiene los cambios más recientes* aparece en la parte superior de la pantalla. El mensaje también tiene un botón *Actualizar*. Vamos a seguir y seleccionar el botón **Actualizar**.

    ![Captura de pantalla de la actualización del desencadenador de Data Activator.](./Images/data-activator-trigger-updated.png)

1. Detenga el desencadenador seleccionando el botón **Detener** en el menú superior.

## Limpieza de recursos

En este ejercicio, ha creado un Reflex con un desencadenador en Data Activator. Ahora debería estar familiarizado con la interfaz de Data Activator y cómo crear un Reflex y sus objetos, desencadenadores y propiedades.

Si ha terminado de explorar el Reflex de Data Activator, puede eliminar el área de trabajo que creó para este ejercicio.

1. En la barra de la izquierda, seleccione el icono del área de trabajo para ver todos los elementos que contiene.
2. En el menú **...** de la barra de herramientas, seleccione **Configuración del área de trabajo**.
3. En la sección **General**, seleccione **Quitar esta área de trabajo**.
