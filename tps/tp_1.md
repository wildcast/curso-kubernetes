# TP1
0. Me paro en el cluster de minikube para arrancar con el TP `kubectl config use-context minikube`
1. Listar todos los namespaces en un cluster: `kubectl get namespaces` o `kubectl get ns`
2. Listar todos los PODS en todos los namespaces: `kubectl get pods --all-namespaces` o `kubectl get pods -A`
3. Listar los pods en un namespace en particular: `kubectl get pods --namespace kube-system` o `kubectl get pods -n kube-system`
4. Crear un POD con 3 containers busybox que realizen los iguiente:
	- `ls; sleep 3600`
	- `echo "Hello World"; sleep 3600`
	- `echo "Este es el 3er contenedor"; sleep 3600`
5. Una vez creados los mismos chequear su estado
6. Crear un deployment llamado `webapp` con una imagen de nginx con 2 replicas. Luego:
	* Obtener el deployment creado y mostrar tus labels 
	* Obtener el archivo yaml del deployment creado 
	* Mostrar los pods corriendo del deployment 
	* Escalar el deployment de 2 a 5 replicas y verificar que funciona correctamente 
	* Obtener el estado del scale-up del deployment 
	* Mostrar el replicaset creado por el deployment