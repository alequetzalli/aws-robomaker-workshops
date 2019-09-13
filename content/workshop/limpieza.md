---
title: "Taller de limpieza"
chapter: true
weight: 12
---



#Taller de limpieza

AWS solo cobra por los recursos consumidos. Siga los siguientes pasos para cerrar/eliminar recursos para que no le cobren.

1. Elimine el **S3 bucket** seleccionando el depósito y luego haciendo clic en *Delete* desde arriba de la lista de depósitos.

2. Desde la **consola de AWS RoboMaker**, asegúrese de que no haya *trabajos de simulación* en progreso. Si los hay, selecciónelos y haga clic en *Acciones-> Cancelar*.

3. También desde **AWS RoboMaker Console**, desde *Desarrollo-> Entornos de desarrollo*, haga clic en el nombre del entorno, *Editar*, y luego* Eliminar* de la consola de AWS Cloud9.

4. En **CloudWatch Console**, en *Registros*, seleccione cada *Grupos de Registros* (`/ aws / robomaker / SimulationJobs` y` dogfinder_workshop`) y haga clic en *Acciones-> Elimine el grupo de registros*.

5. En **Kinesis Video Streams**, elimine su transmisión que liberará el contenido de video almacenado.

6. En **CloudFormation**, seleccione la pila que creó y haga clic en **Delete**.
