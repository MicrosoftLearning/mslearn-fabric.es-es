---
lab:
  title: Implementación de canalizaciones de implementación de Microsoft Fabric
  module: Implement CI/CD in Microsoft Fabric
---

# Implementación de canalizaciones de implementación de Microsoft Fabric

Las canalizaciones de implementación de Microsoft Fabric permiten automatizar el proceso de copiar los cambios realizados en el contenido de los elementos de Fabric entre entornos como desarrollo, pruebas y producción. Puedes usar canalizaciones de implementación para desarrollar y probar contenido antes de que llegue a los usuarios finales. En este ejercicio, crearás una canalización de implementación y asignarás fases a la canalización. A continuación, crearás contenido en un área de trabajo de desarrollo y usarás las canalizaciones de implementación para implementarlo entre las fases de canalización de desarrollo, pruebas y producción.

> **Nota**: para completar este ejercicio, debes ser miembro del rol de administrador del área de trabajo de Fabric. Para asignar roles, consulta [Roles en áreas de trabajo en Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/get-started/roles-workspaces).

Este laboratorio se realiza en **20** minutos aproximadamente.

## Creación de áreas de trabajo

Crea tres áreas de trabajo con la versión de prueba de Fabric habilitada.

1. En la [página principal de Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric) de `https://app.fabric.microsoft.com/home?experience=fabric`, selecciona **Microsoft Fabric** y, después, **Ingeniero de datos** en la barra de menús inferior de la izquierda.
2. En la barra de menús de la izquierda, selecciona **Áreas de trabajo** (el icono tiene un aspecto similar a &#128455;).
3. Crea una nueva área de trabajo denominada Development y selecciona un modo de licencia que incluya la capacidad de Fabric (*Evaluación gratuita*, *Premium* o *Fabric*).
4. Repite los pasos 1 y 2 y crea dos áreas de trabajo más denominadas Pruebas y Producción. Las áreas de trabajo son: Desarrollo, Pruebas y Producción.
5. Selecciona el icono **Áreas de trabajo** de la barra de menús de la izquierda y confirma que hay tres áreas de trabajo denominadas: Desarrollo, Pruebas y Producción.

> **Nota**: si se te pide que escribas un nombre único para las áreas de trabajo, anexa uno o varios números aleatorios a las palabras: Desarrollo, Pruebas y Producción.

## Creación de una canalización de implementación

Luego crea una canalización de implementación.

1. En la barra de menús de la izquierda, selecciona **Áreas de trabajo**.
2. Selecciona **Canalizaciones de implementación** y, luego, **Nueva canalización**.
3. En la ventana **Agregar una nueva canalización de implementación**, asigna un nombre único a la canalización.
4. Acepta los valores predeterminados en la ventana **Personalizar las fases**.  

   ![Captura de pantalla de las fases de canalización.](./Images/customize-stages.png)

5. Selecciona **Crear**.

## Asignación de áreas de trabajo a las fases de una canalización de implementación

Asigna áreas de trabajo a las fases de la canalización de implementación.

1. En la barra de menús de la izquierda, selecciona la canalización que has creado. 
2. En la ventana que aparece, selecciona la palabra **Seleccionar** en cada fase de implementación y selecciona el nombre del área de trabajo que coincida con el nombre de la fase.
3. Selecciona **Asignar un área de trabajo** para cada fase de implementación.

  ![Captura de pantalla de la canalización de implementación.](./Images/deployment-pipeline.png)

## Creación de contenido

Los elementos de Fabric aún no se han creado en las áreas de trabajo. A continuación, crearás un almacén de lago en el área de trabajo de implementación.

1. En la barra de menús de la izquierda, selecciona el **Áreas de trabajo**.
2. Selecciona el área de trabajo **Desarrollo**.
3. Selecciona **Nuevo elemento**.
4. En la ventana que aparece, selecciona **Almacén de lago** y, en la **ventana Nuevo almacén de lago**, denomina al almacén de lago, **LabLakehouse**.
5. Selecciona **Crear**.
6. En la ventana Explorador de almacenes de lago, selecciona **Iniciar con datos de ejemplo** para rellenar el nuevo almacén de lago con datos.

  ![Captura de pantalla del Explorador de almacenes de lago.](./Images/lakehouse-explorer.png)

8. En la barra de menús de la izquierda, selecciona la canalización que has creado.
9. En la fase **Desarrollo**, selecciona **>** hasta que veas **Almacenes de lago**. El almacén de lago se muestra como nuevo contenido en la fase Desarrollo. Entre las fases **Desarrollo** y **Pruebas**, hay una **X** naranja dentro de un círculo. La **X** naranja indica que las fases de desarrollo y pruebas no están sincronizadas.
10. Selecciona la flecha hacia abajo debajo de la **X** naranja para comparar el contenido en los entornos de desarrollo y pruebas. Selecciona **Comparar**. LabLakehouse solo existe en la fase Desarrollo.  

  ![Captura de pantalla de la canalización de implementación que muestra errores de coincidencia de contenido entre las fases.](./Images/lab-pipeline-compare.png)

## Implementación de contenido entre las fases

Implementa el almacén de lago desde la fase **Desarrollo** en las fases **Pruebas** y **Producción**.
1. Selecciona el botón **Implementar** en la fase **Desarrollo** de la canalización para copiar el almacén de lago en su estado actual en la fase de texto. 
2. En la ventana **Implementar en la siguiente fase**, selecciona **Implementar**.
3. Hay una X naranja entre las fases de pruebas y producción. Selecciona la flecha hacia abajo situada debajo de la X naranja. El almacén de lago existe en las fases de desarrollo y pruebas, pero aún no en la fase Producción.
4. En la fase **Pruebas**, selecciona **Implementar**.
5. En la ventana **Implementar en la siguiente fase**, selecciona **Implementar**. La marca de verificación verde entre las fases indica que todas las fases están sincronizadas y contienen el mismo contenido.
6. El uso de canalizaciones de implementación para implementar entre fases también actualiza el contenido de las áreas de trabajo correspondientes a la fase Implementación. Vamos a confirmarlo.
7. En la barra de menús de la izquierda, selecciona el **Áreas de trabajo**.
8. Selecciona el área de trabajo **Pruebas**. El almacén de lago se copió allí.
9. Abre el área de trabajo **Producción** en el icono **Áreas de trabajo** del menú izquierdo. El almacén de lago también se copió en el área de trabajo Producción.

## Limpieza

En este ejercicio, has creado una canalización de implementación y has asignado fases a la canalización. Luego has creado contenido en un área de trabajo de desarrollo y lo has implementado entre fases de canalización mediante canalizaciones de implementación.

- En la barra de navegación izquierda, selecciona el icono de cada área de trabajo para ver todos los elementos que contiene.
- En el menú de la barra de herramientas superior, selecciona Configuración del área de trabajo.
- En la sección General, selecciona Quitar esta área de trabajo.