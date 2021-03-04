# TP2
0. Me paro en el cluster de minikube para arrancar con el TP `kubectl config use-context minikube`
1. Crear un `Deployment` con la imagen de `nginx` que escuche en el puerto 8080: `kubectl create deployment nginx --image=nginx --port=8080`
2. Creen un `Service` que solo acepte trafico interno para su deployment de nginx: `kubectl expose deployment nginx` (por defecto le asigna el type ClusterIP)
3. Hagan un `port-forward` al servicio para poder accederlo localmente: `kubectl port-forward svc/nginx 8080` o `kubectl port-forward nginx-d6dcb985-5vzxt 8080:80`
4. Modifiquen el `Service` para que ahora sea `NodePort` y accedan a su servicio sin usar port-forward (obtengan la info del nodo y el puerto),
5. Actualizen el `Deployment` para que tenga una version distinta de nginx. Pueden buscar las versiones disponibles [aca](https://hub.docker.com/_/nginx?tab=tags). Mientras hacen esa actualizacion, chequeen el status de los pods y replicasets,
6. Actualizen ese deployment con una imagen invalida. Luego de ver que los pods no se pueden crear aborten el rollout actual,
7. Escalen su `Deployment` a 6 replicas y:
	* Configuren el rolling update para que reemplaze de a 2 containers a la vez,
	* Actualizen la imagen y analizen como los pods y replicasets van cambiando,
	* Configuren el rolling update para que reemplaze un 50% de containers a la vez,
	* Vuelvan a actualizar la imagen y validen los cambios de pods y replicasets.
8. Editen su `Deployment` y configuren distintos tipos de probes:
	* Agreguen un readiness probe de `http` que se ejecute despues de 1 segundo cada 5 segundos,
	* Agreguen un liveness probe de `tcp` que se ejecute despues de 5 segundos cada 3 segundos y que considere el servicio como no healthy despues de 3 fallas,
