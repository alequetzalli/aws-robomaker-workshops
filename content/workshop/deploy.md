---
title: "Activity #3: Deploy Hello World Robot"
chapter: true
weight: 10
---


# Implemente una aplicación ROS en su robot

En los ejercicios anteriores, usó RoboMaker con Cloud9 para construir, agrupar y simular dos aplicaciones de robot diferentes. En la actividad final de este taller, implementaremos la sencilla aplicación 'Hello World' que creamos en la primera actividad.

En RoboMaker, las simulaciones usan Gazebo, que se ejecuta en AWS en una flota de servidores con arquitectura de CPU x86. Sin embargo, muchos robots físicos usan diferentes arquitecturas de CPU, como ARM. Antes de que una aplicación de robot pueda implementarse e invocarse en un robot físico, es posible que deba reconstruirse y volver a agruparse para la arquitectura de CPU de destino del robot.

En este taller, implementará una aplicación en un robot 🤖 *TurtleBot 3 Burger*. Este robot utiliza un Raspberry Pi, que se basa en la arquitectura ARMHF. Dentro del entorno de RoboMaker Cloud9, hemos preinstalado un contenedor Docker que simplifica el proceso de compilación para arquitecturas alternativas. Si tiene tiempo después de completar este ejercicio, puede revisar los pasos detallados para construir arquitecturas alternativas en la [documentación de RoboMaker](https://docs.aws.amazon.com/robomaker/latest/dg/gs-deploy.html ). Para este ejercicio, hemos creado previamente un paquete para el robot TurtleBot3 Burger. Usaremos ese paquete para implementar la aplicación en su robot.

Esta actividad cubre los pasos necesarios para preparar un robot físico para recibir una aplicación ROS utilizando RoboMaker. Cuando termine, habrá aprendido:

* Cómo registrar su robot en RoboMaker.
* Cómo implementar certificados de autenticación en sus robots.
* Cómo crear versiones de aplicaciones de robots y flotas de robots.
* Cómo implementar una aplicación ROS incluida en su robot.

## Tareas de la actividad

1. Abra el *entorno de desarrollo* Cloud9 que utilizó en el ejercicio 2 de este taller.

2. Para esta actividad, necesita el ARN para el **rol de implementación** (deployment role) que se creó en el template de CloudFormation. Mire en la pestaña **Outputs** y copie el valor del ARN, que debería ser similar a esto:

 ```text
    arn:aws:iam::123456789012:role/robomaker-deployment-role
    ```

3. Usando el ARN que encontró en el paso anterior, ejecute el siguiente comando para permitir que AWS Greengrass lo use para la implementación:

    ```bash
    # reemplaza DEPLOYMENT_ROLE_ARN con tu ARN
    aws greengrass associate-service-role-to-account --role-arn $DEPLOYMENT_ROLE_ARN
    ```

4. El paquete que usaremos para este paso ha sido creado previamente, simplemente debe decirle a AWS RoboMaker dónde encontrarlo. Abra la consola de AWS RoboMaker y revise las aplicaciones de robot *(Desarrollo-> Aplicaciones de robot)*. Haga clic en el nombre de la aplicación del robot, *RoboMakerHelloWorldRobot*, para revisar sus detalles.

5. Para ver la ubicación de los archivos de paquete para la aplicación, haga clic en el enlace **$ LATEST**, en la versión más reciente.

6. Ahora verá los detalles de la aplicación *RoboMakerHelloWorldRobot*, incluyendo los "sources". Tenga en cuenta que actualmente solo tiene una fuente: la versión X86_64 (esta es la versión que acaba de usar para la simulación). Para contarle a RoboMaker sobre la versión ARMHF, haga clic en el botón **Actualizar** (Update).

7. Copie el siguiente archivo *tar* del robot en su S3 bucket.

    ```text
    aws s3 cp s3://robomakerbundles/turtlebot3-burger/hello-world/robot-armhf.tar s3://<YOUR_BUCKET_NAME>/hello-world/robot-armhf.tar
    ```

8. En el cuadro de texto del archivo de fuente ARMHF, pegue la nueva ubicación S3 para el paquete ARMHF:


    ```text
    s3://<YOUR_BUCKET_NAME>/hello-world/robot-armhf.tar
    ```
    
    Haga clic en **Crear**.

9. Antes de que AWS RoboMaker pueda desplegarse en un robot físico, debe configurar su robot. Debe crear certificados de autenticación que permitan que el dispositivo se comunique de forma segura con AWS. También debe registrar su robot en AWS RoboMaker. Para comenzar con esta tarea, haga clic en el enlace de **robots** bajo la seccion de *Administración de flotas*.

10. Haga clic en el botón **Crear robot**.

11. Déle a su robot un nombre y configure la Arquitectura en ARMHF. Al establecer este valor, le está diciendo a AWS RoboMaker que use el paquete ARMHF cuando realice la implementación en este robot.

12. AWS RoboMaker utiliza AWS GreenGrass para implementar sus paquetes de robots en su dispositivo. Ahora debe configurar los ajustes de AWS GreenGrass de RoboMaker. Puede dejar la configuración de *Grupo de AWS Greengrass* y *Prefijo de AWS Greengrass* como sus valores predeterminados.

13. Para *rol del IAM *, elija "rol-deployment-robomaker". Este rol fue creado en el ejercicio 1 de este trabajo. Su aplicación de robot asume esta función cuando se ejecuta en su dispositivo y le da permiso para acceder a los servicios de AWS en su nombre.

14. Haga clic en **Crear**.

15. Ahora debe descargar los certificados que deben instalarse en su robot. Cuando se instalen en su robot, le dará acceso a su robot para llamar a los servicios de AWS. Haga clic en el botón naranja **Descargar** (download). No es necesario descargar el software *Greengrass Core*. Su dispositivo ha sido preconfigurado con los binarios de Greengrass. Esto descargará un archivo zip llamado *HelloRobot-setup.zip* (o similar, dependiendo del nombre que proporcionó para su robot en el Paso 9 anterior).

16. Los certificados que acaba de descargar deben copiarse en el robot físico y extraerse en un directorio del dispositivo. Estas instrucciones usan *scp* para copiar archivos al dispositivo y *ssh* para conectarse al dispositivo. Ambos comandos están disponibles en Terminal en macOS y en Windows PowerShell. Sin embargo, la disponibilidad de estas herramientas puede variar, dependiendo de su configuración (particularmente en Windows).

17. Si está utilizando una computadora con Windows, salte al siguiente paso. En macOS, abra la Terminal presionando la barra espaciadora de Comando para iniciar Spotlight y escriba "Terminal", luego haga doble clic en el resultado de la búsqueda. Omita el siguiente con respecto a Windows y continúe con el *paso 17*.

18. Si está utilizando Windows, presione la tecla de Windows o haga clic en el botón de Windows (Inicio), escriba "PowerShell" y presione Entrar.

19. En su computadora portátil, navegue hasta el directorio donde descargó los certificados en el *Paso 13* anterior. ($ cd Descargas)

20. Copie el archivo zip a su robot. Reemplace el nombre del archivo con el nombre del archivo que descargó. La dirección IP de su robot se proporcionó con el robot. Se te solicitará una contraseña. La contraseña para el usuario pi es: **robomaker**.

    ```bash
    # reemplace FILE_NAME con el valor de su archivo zip
    $ scp FILE_NAME.zip pi@<ROBOT_IP_ADDRESS>:/home/pi
    ```

21. Conéctese al robot, muestre la placa OpenCR, configure los certificados e inicie el servicio AWS Greengrass. En este paso, utilizará *ssh* para conectarse al robot y luego descomprimirá el archivo de certificados en la ubicación utilizada por AWS Greengrass. Finalmente, inicia el servicio AWS Greengrass. Esto permite que el dispositivo recupere su paquete (packege) de robots y lo implemente en el robot. 

Como recordatorio, el usuario es **pi** y la contraseña es **robomaker**.
    
    ```bash
    # usa SSH y conéctate al robot. Reemplace ROBOT_IP_ADDRESS con la dirección IP de su dispositivo
    $ ssh pi@<ROBOT_IP_ADDRESS>
    # enter password: robomaker.
    $ sudo su
    # enter password: robomaker.

    # descomprima los certificados en el directorio /greengrass. Reemplace FILE_NAME con el archivo que copió anteriormente.
    $ unzip FILE_NAME.zip -d /greengrass

    # actualizar el certificado de CA utilizado por RoboMaker 
    $ cd /greengrass/certs/
    $ wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem

    # comience el servicio Greengrass
    $ /greengrass/ggc/core/greengrassd start
    ```
    

22. **Crea una flota** y agrega tu robot a la flota. Las flotas le permiten administrar un grupo de robots. Por ejemplo, puede implementar la misma aplicación de robot en todos los robots de una flota. Luego se asegura de que todos sus robots estén ejecutando el mismo software. Si necesita diferentes robots para ejecutar un software diferente, puede crear varias flotas. En este taller, creará una flota única que contiene un solo robot. En la consola de AWS RoboMaker, elija *Flotas* en *Administración de Flotas*. Haga clic en el botón **Crear una flota**.

23. Dé a su flota un nombre apropiado y haga clic en **Crear**.

24. Ahora debe agregar su robot a la flota recién creada. En la página de la flota de su nueva flota, haga clic en el botón **Registrar nuevo** en la sección *Robots registrados*.

25. Seleccione su robot y elija **Registrar robot**. ¡Felicidades, su robot ahora es miembro de su flota! 

26. El último paso es implementar nuestra aplicación en nuestra flota. Para comenzar, haga clic en *Implementaciones* en *Administración de flotas* y elija **Crear implementación**.

27. Elija la aplicación 'Fleet and Robot' (RoboMakerHelloWorldRobot) que creó anteriormente.

28. Para *Versión de la aplicación de robot*, elija *Crear nuevo* y haga clic en **Crear** en la confirmación. Esto creará una nueva versión para la aplicación de robot HelloRobot_application.

29. Para *Nombre del paquete*, ingrese `hello_world_robot`. Este es el paquete ROS que contiene el archivo de inicio de nuestro robot.

30. Para *Archivo de lanzamiento*, ingrese `deploy_rotate.launch`. Este es el archivo de inicio dentro del paquete anterior. Contiene información sobre los nodos ROS que se iniciarán en el dispositivo.

31. Todos los demás campos pueden usar sus valores predeterminados. Desplácese hasta la parte inferior y elija **Crear**.

En unos segundos, su robot comenzará a descargar el paquete de robots. Debido a que esta es la primera vez que esta aplicación se implementa en su robot, el tamaño del paquete es grande (~ 600 MB). El robot tardará unos 15 minutos en descargar y extraer el contenido del robot. Puede seguir el progreso de la implementación en la sección *Estado de los robots*:



¡Cuando el paquete se haya implementado con éxito, la aplicación de robot se iniciará automáticamente en su robot! 





