---
title: "Prep: Workshop Setup"
chapter: true
weight: 3
---

# Configuración del taller

### Inicie sesión en la consola de AWS

Para completar este taller, necesita una **cuenta de AWS con permisos administrativos**. Esto es necesario para crear o modificar recursos y permitir que AWS RoboMaker interactúe con los servicios en su nombre.

**Nota: Utilizaremos us-west-2 (Oregon) para este taller.**

Si está en un salón de clases, le proporcionaremos un código de crédito que debe aplicar ahora. Para aplicar el código de crédito, seleccione su nombre de usuario en la esquina superior derecha de la consola de AWS y haga clic en ** Mi cuenta **. Luego, haga clic en **Créditos**.
      
   **Importante:** *Los códigos de crédito proporcionados cubrirán el costo de este taller. Sin embargo, debe limpiar los recursos una vez que el taller haya finalizado.*

### Lanzar pila de información en la nube

Una vez que haya iniciado sesión con éxito en la consola de AWS, inicie el siguiente "stack" de información en la nube para crear los recursos necesarios:

[Launch Stack](https://console.aws.amazon.com/cloudformation/home#/stacks/new?templateURL=https://s3.amazonaws.com/assets.robomakerworkshops.com/cfn/bootstrap.cfn.yaml&region=us-west-2)

Esto creará:

   - un **VPC** y un par de **subnets** y un **grupo de seguridad default** para ejecutar instancias de AWS RoboMaker.
   - un **S3 bucket** para almacenar tus assets de Robomaker (como paquetes de aplicaciones).
   - **Dos roles de IAM** que usarás para el taller.

Una vez que se haya lanzado el nuevo "stack", tome nota de los **outputs**. Utilizaremos estos valores durante todo el taller.

### Crear video de Kinesis

Finalmente, abra la consola para [Kinesis Video Streams](https://console.aws.amazon.com/kinesisvideo/home) y cree un nuevo stream con la siguiente configuración. Al crear el stream, **desmarque "Usar la configuración predeterminada"**:

   - **Nombre de la transmisión**: *roboMaker_TurtleBot3*
   - **Período de retención de datos**: *1 hora*


*Anota el nombre del stream (`roboMaker_TurtleBot3`) para su uso futuro.*

**¡Felicidades! Has completado el proceso de configuración del taller.**










