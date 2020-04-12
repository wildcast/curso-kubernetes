## Tp1

En este TP la idea es familiarizarse con `kubectl` y el ciclo de vida básico de la creación de un pod y deployment. Tambíen se pide
crear un pod multi-container para así aprender cómo los containers dentro del mismo interactúan entre sí.


1. Listar todos los namespaces en un cluster

2. Listar todos los PODS en todos los namespaces

3. Listar los pods en un namespace en particular

4. Crear un POD con 3 containers busybox que realizen los iguiente:

    ls; sleep 3600 

    echo "Hello World"; sleep 3600 

    echo "Este es el 3er contenedor"; sleep 3600 


5. Una vez creados los mismos chequear su estado


6. Crear un deployment llamado "webapp" con una imagen de nginx con 2 replicas. Luego:

    Obtener el deployment creado y mostrar tus labels 

    Obtener el archivo yaml del deployment creado 
  
    Mostrar los pods corriendo del deployment 
  
    Escalar el deployment de 2 a 5 replicas y verificar que funciona correctamente 
  
    Obtener el estado del scale-up del deployment 
  
    Mostrar el replicaset creado por el deployment 
