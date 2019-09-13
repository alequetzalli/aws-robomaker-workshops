---
title: "Activity #2: Encuentra a Fido, Dog Finder Robot"
chapter: true
weight: 8
---


# Integración de servicios en la nube para hacer detección de objetos


Nuestro objetivo: ¡Encontrar a Fido! 🐶 

Esta actividad cubre el trabajo con una aplicación de robot que se integra con otros servicios de AWS. El robot funcionará en un mundo virtual y activará/detectará imágenes, buscando una que incluya un perro.

Cuando termine, habrá aprendido a:

* Usar comandos en la terminal para construir y agrupar nuestras aplicaciones
* Enviar un *trabajo de simulación* mediante programación desde la línea de comandos
* Revisión de la salida de Gazebo contra CloudWatch registrando las publicaciones de temas directamente desde el robot
* Muestra cómo se puede enviar la salida de la cámara del robot a Kinesis Video Streams para su uso posterior
* Utilice los comandos `rostopic` para enviar un mensaje al robot para reiniciar su búsqueda de objetivos una vez que se haya encontrado la imagen del perro

## Tareas de actividad


1. Para esta actividad, utilizará **tres pestañas de terminal** para trabajar lado a lado en los directorios de aplicaciones de simulación y robot, mientras usa la **tercera pestaña** para trabajar con el OS (sistema operativo).

    Cierre todas las ventanas de terminal (*bash, Inmediato, etc.*) y luego use el signo más verde para abrir tres pestañas más:

    Cuando una tarea en los próximos pasos diga "Desde el **SIM TAB** ejecute XXX", use la *segunda pestaña* del medio llamada "sim".

2. El proyecto con el que trabajaremos se encuentra en GitHub. Debe clonarlo en el entorno Cloud9 para poder trabajar con él. Desde **TAB OS**, ejecute los siguientes comandos para clonar el repositorio:

    ```bash
    cd ~/environment

    # clonar el repositorio DogFinder
    git clone https://github.com/jerwallace/aws-robomaker-sample-application-dogfinder.git
    ```

3. Para compilar la aplicación de robot, emita los siguientes comandos desde **ROBOT TAB**:
    
    ```bash
    cd aws-robomaker-sample-application-dogfinder/robot_ws/

    # Baje los últimos paquetes
    sudo apt-get update

    # Tire de los paquetes ROS (los errores vistos desde el principio se pueden ignorar)
    # Este paso toma unos 5-10 minutos para completar
    rosdep install --from-paths src --ignore-src -r -y

    # Construye la aplicación del robot
    colcon build
    ```
    

4. Una vez que se haya completado, cree la aplicación de simulación a partir de **SIM TAB**:
    
    ```bash
    cd aws-robomaker-sample-application-dogfinder/simulation_ws/

    # rosdep nuevamente - se completará rápidamente 
    rosdep install --from-paths src --ignore-src -r -y

    # Cree la aplicación de simulación: se completará rápidamente
    colcon build
    ```

    El proceso de creación y dependencia de ROS inicial lleva más tiempo debido a todos los paquetes externos que deben descargarse, compilarse o instalarse. A medida que realiza pequeños cambios en el código e itera, el proceso de compilación se vuelve mucho más rápido. 
    
    Ahora, tanto el robot como la aplicación de simulación están listos. La *aplicación de simulación* tendrá el mundo del hexágono listo con el TurtleBot3 centrado, y la aplicación de robot se ha creado con integración nativa a CloudWatch Logs, Metric y Kinesis Video Streams; y soporte de **boto3** (SDK de Python) para enviar imágenes a **Amazon Rekognition** para la detección de objetos.

    Sin embargo, dado que no podemos simular desde el IDE de Cloud9, continúe agrupando (bundling/building) ambas aplicaciones.

5. Para agrupar la aplicación del robot, desde **ROBOT TAB** ejecute lo siguiente:
 
    ```bash
    #  asegúrese de que colcon bundle sea la última versión. Esto solo debe ejecutarse una vez en el entorno Cloud9
    sudo pip3 install -U colcon-bundle colcon-ros-bundle

    # crear el paquete para la aplicación del robot
    colcon bundle
    ```

    Una vez completado con éxito, haga lo mismo en **SIM TAB**:

    ```bash
    # crear el paquete para la aplicación de simulación
    colcon bundle
    ```

    Estas dos operaciones crearán archivos tar completos para su uso y los escribirán en cada directorio de paquete respectivo de espacios de trabajo.

    *¿Por qué tengo que seguir todos estos pasos cuando en la actividad anterior hice clic en un comando de menú y ocurrió la magia?!?!* 

Ese es uno de los beneficios de AWS RoboMaker, la capacidad de incluir la complejidad de ROS en unos pocos comandos.😁  
    
En el fondo, se estaban siguiendo los mismos pasos que acaba de completar. Al hacer esto paso a paso, puede entender mejor el proceso completo de construir e implementar una aplicación de robot. En muchas situaciones, tendrá que pasar por configuraciones similares para sus aplicaciones, por lo que es útil familiarizarse con ellas.

6. Con ambas aplicaciones integradas, ahora las copiará a S3 para que puedan ser utilizadas por el servicio de simulación. Para ambas aplicaciones, copie a S3:

    De la **PESTAÑA ROBOT**:
    
    ```bash
    
    # Reemplace <YOUR_BUCKET_NAME> con su bucket 
    aws s3 cp bundle/output.tar s3://<YOUR_BUCKET_NAME>/dogfinder/output-robot.tar
    ```

   ... y del **SIM TAB**:
    
    ```bash
     
    # Reemplace <YOUR_BUCKET_NAME> con..
    aws s3 cp bundle/output.tar s3://<YOUR_BUCKET_NAME>/dogfinder/output-sim.tar
    ```

7. Con los archivos de paquete listos, cree un trabajo de simulación desde la pestaña del sistema operativo. En la raíz del directorio /DogFinder hay un nombre de archivo.

7. Con los archivos de paquete listos, cree un trabajo de simulación desde la pestaña del sistema operativo. En la raíz del directorio /DogFinder hay un archivo llamado `submit_job.sh`. Haga doble clic en él y reemplace los "outputs" en la parte superior del archivo con las específicas (depósito S3, detalles de VPC, etc.), **y luego guardelos** (save). Hay uno completo en su **CloudFormation> Outputs**. Debería ser similar a esto:

    ```bash
  
     #!/bin/bash
     # Ejemplo: reemplace con el suyo
     export BUCKET_NAME="<YOUR_BUCKET_NAME>"
     export SUBNETS="subnet-e2xxx795,subnet-e2xxx123"
     export SECURITY_GROUP="sg-fe2xxx9a"
     export ROLE_ARN="arn:aws:iam::1234565789012:role/robomaker_role"
    ```


8. En el **OS TAB**, ejecute el script que creará el robot y las aplicaciones de simulación, luego cree e inicie el trabajo de simulación:
    
    ```bash
    # script en el nivel superior del directorio /DogFinder, ajuste según sea necesario
    aws-robomaker-sample-application-dogfinder/submit_job.sh
    ```


Un lanzamiento exitoso devolverá un documento JSON con todos los detalles, incluido un *ARN* con el valor del trabajo de simulación:

    ```json
    "arn": "arn:aws:robomaker:us-west-2:123456789012:simulation-job/sim-8rcvbm7p023f",
    
    ```

9. En este punto, puede abrir una consola de AWS RoboMaker y verificar el estado del trabajo de simulación. Tardará unos minutos en pasar de *Pendiente* a *En ejecución*, pero en ese punto puede abrir las aplicaciones Gazebo y Terminal.

Observe en Gazebo mientras ve como el robot está mirando hacia el norte en la imagen del puente. En este momento, el robot está esperando un mensaje para comenzar a buscar objetivos y encontrar la imagen del perro. Antes de emitir el comando desde el terminal de simulación, veamos las siguientes ventanas y redimensionemos para que podamos verlas todas (puede tomar un poco de ajuste):

* **Kinesis Video Streams consola**, luego haga clic en su transmisión
* **Gazebo**, acerca el hexágono y el robot
* **Terminal de trabajo de simulación**, donde emitirá el comando de inicio


No necesita ver demasiado de la ventana de transmisión de video en segundo plano, solo lo suficiente para ver el video humeante.

10. En este punto, en Gazebo, el robot debe mirar hacia arriba (hacia el norte); la transmisión de video debe mostrar la foto del puente; y los registros (logs) de CloudWatch deben mostrar un mensaje "Esperando para comenzar a encontrar a Fido". Ahora desde la terminal, enviará un mensaje a un tema que el robot está escuchando para comenzar la acción de búsqueda de objetivos:

     ```bash
     rostopic pub --once /df_action std_msgs/String 'start'
     ```

Lo que esto hará es publicar (`pub`) un solo mensaje (`--once`) sobre el tema que escucha su robot (`/ df_action`), y enviará un tipo de string (`std_msgs / String`) con el comando para procesar (`start`). El robot recibirá este comando y comenzará la tarea (girar y procesar imágenes), buscando nuestro objetivo, una imagen de un perro. 🐶 

**Cuando vea que el robot comienza a girar en Gazebo, si la transmisión de video no se actualiza, haga clic en el botón "avance rápido" para avanzar al tiempo real.**

Una vez que se ha encontrado un perro, el robot se detendrá e iniciará sesión en **CloudWatch Registros-> Groupos de Registro-> dogfinder_workshop-> TurtleBot3** ¡mensajes informativos sobre cómo encontrar al perro!


11. Una vez que se encuentra la imagen del perro, el robot espera el siguiente comando para comenzar el proceso nuevamente. Puede emitir el comando `rostopic pub` nuevamente en la terminal para iniciar el proceso nuevamente.

12. En este punto, si hay tiempo, siéntase libre de investigar las otras aplicaciones y ver cómo funciona el código. Por ejemplo, si desea ver la vista del mundo del robot a través de **rqt**, abra la aplicación *rqt* y en el menú *rqt* seleccione **Plugins-> Visualizacion-> Image View** y luego en ese drop down menú, seleccione */camera/rgb/image_raw.*

## Resumen de actividad

En esta actividad, creó y simuló una aplicación de robot 🤖  que no solo interactúa (gira) en Gazebo, sino que también utiliza otros servicios de AWS. Esto incluye la integración nativa de ROS, como los registros (logs) de CloudWatch y Kinesis Video Streams, o mediante el uso flexible de AWS SDK, como *boto3*, para interactuar con otros servicios de AWS como Amazon Rekognition.

Lo que estaba cubierto:

* Trabajar con aplicaciones ROS dentro del entorno de desarrollo a nivel de línea de comando
* Comprender los recursos necesarios para compilar (compilar) y empaquetar (agrupar) una aplicación ROS
* Uso de la CLI de AWS para interactuar con AWS RoboMaker para crear aplicaciones, cargar paquetes e iniciar trabajos de simulación
* Uso de servicios de AWS como Kinesis Video Streams y CloudWatch logs (registros) directamente desde ROS para interactuar con el robot (virtual o real)
* Use SDK estándar para interactuar con otros servicios de AWS

### Limpiar

En esta actividad, creó un entorno de Desarrollo, registros de CloudWatch y objetos S3 que inciden en el costo. Siga los pasos de limpieza en la página principal. Lea el documento README sobre cómo eliminarlos y detener cualquier costo potencial por ocurrir.

