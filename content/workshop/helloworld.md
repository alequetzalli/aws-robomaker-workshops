---
title: "Activity #1: Hello Robot Simulation"
chapter: true
weight: 4
---




# Entorno de desarrollo y HelloWorld app

En esta actividad, configurará un entorno de desarrollo de AWS RoboMaker para compilar y ejecutar rápidamente una aplicación ROS "Hello World". Una vez completado, habrás aprendido:

* Cómo navegar por la consola de AWS RoboMaker y acceder al entorno de desarrollo y a los trabajos de simulación
* Diseño básico del espacio de trabajo ROS y tareas de compilación / paquete
* Cómo enviar e interactuar con un trabajo de simulación

## Tareas de actividad

1. Abra una pestaña nueva en la consola de AWS RoboMaker (*Servicios-> RoboMaker-> clic derecho-> pestaña nueva*)

2. Cree un entorno de desarrollo (*Desarrollo-> Entornos de desarrollo-> Crear entorno*) y complete lo siguiente:

    * Nombre: `taller` o algo descriptivo
    * Tipo de instancia: `m4.large`
    * Elija el default VPC y una subred para su entorno de desarrollo
    * Haga clic en **Crear** 

3. Esto abre la página de detalles del entorno, haga clic en *Abrir entorno*, que abrirá una nueva pestaña del navegador con el IDE de Cloud9.

*Esto puede tardar unos minutos en completarse...*
  

La *Página de bienvenida* proporciona información útil para comenzar, pero por ahora no la vamos a usar, así que haga clic en la *X* en la pestaña para cerrar. El IDE se divide en cuatro secciones:

    - (1) El menú de AWS RoboMaker proporciona acceso rápido a acciones comunes. Se actualiza cuando el `roboMakerSettings.json` se modifica más adelante en esta tarea.
    - (2) Todos los archivos y folders residirán aquí, y se pueden seleccionar y hacer doble clic para abrirlos en el panel del editor (# 4).
    - (3) La sección inferior es un panel ajustable para crear o monitorear operaciones de línea de comando. Los desarrolladores de ROS trabajan en esta área para construir, probar e interactuar con el código local.
    - (4) Este es el panel principal del editor.

4. Elimine el archivo `roboMakerSettings.json` haciendo clic derecho sobre él y seleccionando *Eliminar* -> Sí. Usaremos el archivo de aplicaciones de ejemplo para completar. 

5. Luego, use el menú para descargar y crear la aplicación *HelloWorld* haciendo clic en *Recursos-> Descargar muestras-> 1. Hello World*. Esto descargará, descomprimirá y cargará el archivo README para la aplicación Hello World.

6. Mientras se descargan los archivos de la aplicación, en otra ventana, haga clic en **AWS CloudFormation** y seleccione la pila que inició en los pasos de configuración. En **Outputs** encontrará dos ARN roles para usar con RoboMaker. Para este ejercicio, utilizaremos el *rol de simulación* que se puede encontrar en el par de valores clave titulado 'SimulationRole' y el depósito S3 que se creó. Deberían ser similares a esto:

    ```text
    arn:aws:iam::123456789012:role/robomaker-simulation-role
    <su-stack-name>-assets
    ```
  
    Copie el rol ARN y el nombre del depósito S3 y regrese a su editor *RoboMaker Cloud9*. 

7. Para este proyecto, vamos a utilizar la opción de menú para compilar, agrupar y simular. Cierre el archivo `README.md` en el panel del editor, luego abra la carpeta *HelloWorld* (haga doble clic) y haga doble clic en el archivo `roboMakerSettings.json` para editar. 

    Este archivo contiene todas las configuraciones para construir el menú de arriba. Utilizará estas configuraciones predeterminadas, pero debe completar las secciones de cubo S3 y de ARN de rol de IAM para su cuenta.

9. Desplácese hacia abajo hasta la sección `simulación` y reemplace la `ubicación de salida` con su nombre de depósito S3. Debajo de eso, reemplace el <<your ... role ARN>> con el ARN completo guardado del paso anterior. Guarda el archivo. Esto actualizará las opciones del menú para usar los nuevos valores.

Justo encima del atributo de simulación, también reemplace las entradas (outputs) de s3Bucket para `robotApp` y` simulationApp` para que coincida con lo que se creó en CloudFormation.

*Hay tres ubicaciones para ingresar los detalles del depósito S3. Si recibe un error durante la ejecución, verifique que los tres estén completos y que se haya ingresado el rol ARN.*

Cuando termine, las secciones modificadas deberían ser similares a esta:

    ```json
       }, {
         "id": "HelloWorld_SimulationJob1",
         "name": "HelloWorld",
         "type": "simulation",
         "cfg": {
           "robotApp": {
             "name": "RoboMakerHelloWorldRobot",
             "s3Bucket": "<your-stack-name>-assets",
             "sourceBundleFile": "./HelloWorld/robot_ws/bundle/output.tar.gz",
             "architecture": "X86_64",
             "robotSoftwareSuite": {
               "version": "Kinetic",
               "name": "ROS"
             },
             "launchConfig": {
               "packageName": "hello_world_robot",
               "launchFile": "rotate.launch"
             }
           },
           "simulationApp": {
             "name": "RoboMakerHelloWorldSimulation",
             "s3Bucket": "<your-stack-name>-assets",
             "sourceBundleFile": "./HelloWorld/simulation_ws/bundle/output.tar.gz",
             "architecture": "X86_64",
             "launchConfig": {
               "packageName": "hello_world_simulation",
               "launchFile": "empty_world.launch"
             },
             "simulationSoftwareSuite": {
               "name": "Gazebo",
               "version": "7"
             },
             "renderingEngine": {
               "name": "OGRE",
               "version": "1.x"
             },
             "robotSoftwareSuite": {
               "version": "Kinetic",
               "name": "ROS"
             }
           },
           "simulation": {
             "outputLocation": "df-workshop",
             "failureBehavior": "Fail",
             "maxJobDurationInSeconds": 28800,
             "iamRole": "arn:aws:iam::YOUR_ACCOUNT:role/YOUR_ROLE"
           }
         }
       },
    ```


10. A continuación, actualice la aplicación de simulación *hello world* para usar TurtleBot3 'waffle_pi' en lugar de 'burger'. TurtleBot3 Waffle Pi tiene un módulo de cámara, que utilizaremos como parte de esta actividad. Para hacer esto, abra el siguiente archivo:

    ```text
     HelloWorld/simulation_ws/src/hello_world_simulation/launch/empty_world.launch
    ```
    
Luego, agregue esta línea en el bloque de inclusión TurtleBot3: `<arg name =" model "value =" waffle_pi "/>`. Después de completar, el archivo debería verse así:

    ```xml
      <launch>
        <!-- Always set GUI to false for AWS RoboMaker Simulation
            Use gui:=true on roslaunch command-line to run with a gui.
        -->
        <arg name="gui" default="false"/>

        <include file="$(find gazebo_ros)/launch/empty_world.launch">
          <arg name="world_name" value="$(find hello_world_simulation)/worlds/empty.world"/>
          <arg name="paused" value="false"/>
          <arg name="use_sim_time" value="true"/>
          <arg name="gui" value="$(arg gui)"/>
          <arg name="headless" value="false"/>
          <arg name="debug" value="false"/>
          <arg name="verbose" value="true"/>
        </include>

        <!-- Spawn Robot -->
        <include file="$(find turtlebot3_description_reduced_mesh)/launch/spawn_turtlebot.launch">
          <arg name="model" value="waffle_pi" />
        </include>
      </launch>
    ```

11. Ahora, use el menú para construir y agrupar tanto el robot como la aplicación de simulación. Haga clic en *Ejecutar-> Construir-> HelloWorld Robot* para iniciar la compilación de la aplicación del robot. Esto tomará aproximadamente 1-2 minutos ya que necesita descargar y compilar el código. Cuando vea el mensaje de proceso completo exitoso `El proceso salió con el código: 0`, use el mismo comando para construir la *Simulación HelloWorld*. 

12. En este punto, ambas aplicaciones se han compilado localmente. Para ejecutarse como un trabajo de simulación de AWS RoboMaker, también debe agruparlos. En este paso, la aplicación junto con todas las dependencias del sistema operativo se empaquetan en un paquete que es como un contenedor. Una vez que se completa el proceso, se generará un archivo de salida comprimido localmente. Similar a los pasos de compilación anteriores, debe agrupar tanto el robot como la aplicación de simulación. Para hacer esto, seleccione *Ejecutar (Run)-> Paquete (Bundle)-> HelloWorld Robot*. **Esto tomará entre 10 y 15 minutos más o menos para completar ambos, y es posible que vea advertencias de Cloud9 sobre poca memoria, que puede ignorar.**

Mientras se están creando, tómese un momento para revisar el archivo JSON para el área de simulación. Aquí, puede ver cómo las configuraciones de lanzamiento hacen referencia al nombre del paquete (hello_world_robot o hello_world_simulation) y al archivo de lanzamiento específico a usar (rotate.launch y empty_world.launch respectivamente).

13. Cuando se completen ambas operaciones de paquete, inicie un trabajo de simulación (*Ejecutar-> Iniciar simulación-> HelloWorld*). Esto hará lo siguiente:

    * Cargar el robot y los paquetes de aplicaciones de simulación (aproximadamente 1.2GiB) en el cubo S3
    * Crear una aplicación de robot y una aplicación de simulación que hagan referencia a los paquetes cargados
    * Iniciar el trabajo de simulación en su VPC definida

    *Nota: Si se encuentra con un error en este paso, verifique su archivo `roboMakerSettings.json` y asegúrese de que todas las referencias de S3 y el rol de IAM hayan cambiado a los valores de sus salidas de CloudFormation.*

14. Abra la consola de AWS RoboMaker y haga clic en trabajos de simulación. Debería ver su trabajo en un estado *En ejecución*. Haga clic en la identificación del trabajo para ver los valores que se pasaron como parte del trabajo. Esta vista proporciona todos los detalles del trabajo y el acceso a las herramientas que usará en un momento.

     Si el estado muestra Fallido, lo más probable es que se deba resolver un error tipográfico o de configuración. En la sección *Detalles*, busque el *Motivo de falla* para determinar qué ocurrió para poder corregirlo.
     
     
15. A partir de los detalles del trabajo de simulación, lanzaremos un par de herramientas para interactuar con el robot. Primero, haga clic en Gazebo, que abrirá una ventana emergente para la aplicación. Este es un cliente que proporciona un visión del mundo virtual.


     Usando su mouse o trackpad, haga clic en la ventana principal. Consulte [esta página](http://gazebosim.org/tutorials?tut=guided_b2&cat=) para obtener más detalles y cómo navegar (cerca de la parte inferior de la página).

     Cuando se acerca, verá que el robot gira lentamente en sentido contrario a las agujas del reloj en la misma posición. Esto se debe al archivo `rotate.launch` enviado como parte de la aplicación del robot.

     Deje esta ventana abierta por ahora y regrese a la página de trabajo de simulación. Aquí, haga clic en **Rviz**, que abrirá una nueva ventana. :bulb: si es posible, agrande la ventana de su computadora portátil para una mejor visibilidad.

     [Rviz](http://wiki.ros.org/rviz) es una herramienta de visualización 3D para ROS. Proporciona información sobre el estado del robot y el mundo que lo rodea (virtual o real). Para que su robot virtual funcione correctamente, debemos apuntar a un componente de robot. Haga clic en la ventana y luego haga clic en el "mapa" junto a *Fixed Frame* (Marco Fijo).

     En el menú desplegable, seleccione `base_footprint` y luego haga clic en el espacio en blanco a continuación. Esto debería corregir el mensaje *Estado global: error*. A continuación, seleccione el botón *Agregar* en la esquina inferior izquierda y desde *By display type*, seleccione *Cámara* y haga clic en *OK*. Observe el patrón de cuadrícula en la ventana de la cámara. Para ver lo que "ve" el robot virtual, haga clic en el triángulo al lado de Cámara para girarlo hacia abajo, luego haga clic en el campo *Image Topic* (Tema de Imagen). Debe haber un solo nombre de *name/camera/rgb/image_raw* para que seleccione.

     Como en el caso anterior, una vez que seleccione, haga clic fuera de esta área (o haga clic en *Estado global: OK*) para que surta efecto. La ventana de la cámara debería cambiar a una vista gris de dos tonos. Esto es correcto ya que el mundo en el que se lanzó está realmente vacío. Es hora de agregar algo.

15. Vuelva a la ventana de **Gazebo** y seleccione el menú de objeto (cubo, bola o cilindro) y luego, dentro de la vista del mundo, arrastre su objeto cerca del robot y haga clic para colocarlo. Haga esto un par de veces para agregar objetos a su alrededor.

     Y ahora vuelve a **Rviz** y mira la ventana de la cámara. ¡Verás pasar los objetos por delante mientras el robot gira! Si puede, coloque ambas ventanas para que pueda ver el robot en Gazebo y la vista de la cámara desde Rviz:

16. Para el ejemplo de Hello World, estas son las operaciones básicas del robot y el mundo. Antes de pasar a la siguiente actividad, echemos un vistazo a algunas de las otras cosas que hizo esta simulación.

17. Primero, cierre las ventanas **Gazebo** y **Rviz** y vuelva a la página de *Simulación de Trabajos*. Desde la simulación, en acciones, seleccione *Cancelar-> Sí, Cancelar* para detener la simulación. Esto finaliza con gracia la simulación y detiene las cargas. Regrese a la lista de trabajos de simulación y debería ver el trabajo en estado *Cancelado*. Vuelva a hacer clic en el trabajo y bajo *Detalles* haga clic en el enlace *Destino de salida de trabajo de simulación*. Esto abre una nueva pestaña para S3 donde esta es la carpeta correspondiente a la ID de trabajo de simulación.

     Haga clic en la carpeta y luego en la carpeta con fecha / hora estampada. Aquí tendrá tres conjuntos de salidas. La carpeta `gazebo-logs` contiene los archivos de registro de la salida de las aplicaciones de simulación, mientras que` ros-logs` contiene la salida de las aplicaciones del robot. Los 'ros-bags' contendrán grabaciones de todos los eventos que pueden usarse como entrada de simulación para otras tareas. Para la aplicación de simulación o robot, descargue uno de los archivos y ábralo localmente. Esto le dará una idea de qué tipos de eventos se guardan para su revisión o uso futuro.

18. S3 contiene los archivos de registro de la salida de las aplicaciones, pero también puede obtenerlos de CloudWatch Logs(registros). Navegue a la consola de CloudWatch y seleccione *Registro*s desde la izquierda. Aquí verá los nombres de un grupo de registro `/aws/robomaker/SimulationJobs`. Haga clic en eso y luego haga clic en su ID de simulación para *RobotApplicationLogs* o *SimulationApplicationLogs* y vea las entradas. 



## Resumen de actividad

En esta actividad, ha trabajado con 2 de los principales productos de AWS RoboMaker: **entorno de desarrollo** y **trabajos de simulación**. El entorno de desarrollo proporciona un IDE en la nube basado en la web con todos los principales paquetes y dependencias ROS integrados. Puede crear tantos como sea necesario y, cuando no esté en uso, el IDE se suspenderá automáticamente para reducir los costos. Aquí puede trabajar para crear, probar y construir/agrupar (build/bundle) su código.

Cuando está listo para la prueba, la creación del *trabajo de Simulación* automatiza el proceso de peinar las dos aplicaciones, proporciona una interacción gráfica basada en la web con su robot y el mundo simulado, y registra la salida de todos los componentes de múltiples maneras.

Algunos de los beneficios clave del uso de AWS RoboMaker incluyen:

* No se limita a los recursos locales: al usar la nube, AWS RoboMaker proporciona recursos según sea necesario y solo cuando los necesite.
* Desarrolle en cualquier lugar en cualquier dispositivo basado en la web: al transmitir la salida de aplicaciones gráficamente intensivas como Gazebo en lugar de procesarla localmente, puede usar una computadora de escritorio, una computadora portátil o incluso una tableta para desarrollar e interactuar con simulaciones.
* Escala: esta actividad mostró una sola simulación de una aplicación. Si se necesita probar un robot en diferentes entornos (simulaciones), o probar diferentes iteraciones de aplicaciones de robots en un solo entorno simulado, ambos se pueden escalar a múltiples trabajos de simulación.

### Recordatorio de limpieza

En esta actividad, creó un *entorno de Desarrollo*, *registros de CloudWatch* y *S3 buckets* que inciden en el costo. Si **no continúa** con las siguientes secciones del taller, recuerde ir a los [pasos de limpieza](https://www.robomakerworkshops.com/workshop/cleanup/) y elimine estos recursos para detener cualquier costo potencial por ocurrir.

