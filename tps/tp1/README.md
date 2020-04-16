Resolución TP1

1. Listar todos los namespaces en un cluster

	λ kubectl get ns
	NAME              STATUS   AGE
	default           Active   10h
	docker            Active   10h
	kube-node-lease   Active   10h
	kube-public       Active   10h
	kube-system       Active   10h

2. Listar todos los PODS en todos los namespaces

	λ kubectl get pods -A
	NAMESPACE     NAME                                     READY   STATUS    RESTARTS   AGE
	docker        compose-7b7c5cbbcc-q9gdj                 1/1     Running   0          10h
	docker        compose-api-dbbf7c5db-5qjpr              1/1     Running   0          10h
	kube-system   coredns-5c98db65d4-22fgg                 1/1     Running   0          10h
	kube-system   coredns-5c98db65d4-h8pql                 1/1     Running   0          10h
	kube-system   etcd-docker-desktop                      1/1     Running   0          10h
	kube-system   kube-apiserver-docker-desktop            1/1     Running   0          10h
	kube-system   kube-controller-manager-docker-desktop   1/1     Running   0          10h
	kube-system   kube-proxy-s786h                         1/1     Running   0          10h
	kube-system   kube-scheduler-docker-desktop            1/1     Running   0          10h
	kube-system   storage-provisioner                      1/1     Running   0          10h

3. Listar los pods en un namespace en particular

	λ kubectl get pods -n kube-system
	NAME                                     READY   STATUS    RESTARTS   AGE
	coredns-5c98db65d4-22fgg                 1/1     Running   0          10h
	coredns-5c98db65d4-h8pql                 1/1     Running   0          10h
	etcd-docker-desktop                      1/1     Running   0          10h
	kube-apiserver-docker-desktop            1/1     Running   0          10h
	kube-controller-manager-docker-desktop   1/1     Running   0          10h
	kube-proxy-s786h                         1/1     Running   0          10h
	kube-scheduler-docker-desktop            1/1     Running   0          10h
	storage-provisioner                      1/1     Running   0          10h

4. Crear un POD con 3 containers busybox que realizen los iguiente 
5. Una vez creados los mismos chequear su estado

Probé y busqué comandos la forma de cumplir la consigna utilizando unicamente kubectl pero no tuve éxito.
Está generado en 3 PODs diferentes, asi que no cumple la consigna. 

	λ kubectl run punto4a --image=busybox --restart=Never -- ls && sleep 5 && date
	pod/punto4a created
	Wed, Apr 15, 2020 06:28:25 AM

	λ kubectl run punto4b --image=busybox --restart=Never -- echo "Hello World!" && sleep 5 && date
	pod/punto4b created
	Wed, Apr 15, 2020 06:28:55 AM
	λ kubectl logs punto4b
	Hello World!

	λ kubectl run punto4c --image=busybox --restart=Never -- echo "Este es el 3er contenedor" && sleep 5 & & date
	pod/punto4c created
	Wed, Apr 15, 2020 06:31:57 AM
	λ k logs punto4c
	Este es el 3er contenedor

Estoy más acostumbrado a usar los manifest yaml, no me había planteado nunca generar un POD multicontainer solo utilizando kubectl. 
Lo armé como yaml en el file punto4.yml (En ambos casos baje el sleep solo para acortar la prueba)

	λ kubectl apply -f punto4.yml
	pod/app created

	λ kubectl get pods
	NAME   READY   STATUS      RESTARTS   AGE
	app    0/3     Completed   0          18s

	λ kubectl logs app c1
	bin
	dev
	etc
	home
	proc
	root
	sys
	tmp
	usr
	var
	Wed Apr 15 09:54:49 UTC 2020
	Wed Apr 15 09:54:54 UTC 2020

	λ kubectl logs app c2
	Hello World!
	Wed Apr 15 09:54:53 UTC 2020
	Wed Apr 15 09:54:58 UTC 2020

	λ kubectl logs app c3
	Este es el 3er contendor
	Wed Apr 15 09:54:57 UTC 2020
	Wed Apr 15 09:55:02 UTC 2020
	Wed Apr 15 09:54:54 UTC 2020

6. Crear un deployment llamado `webapp` con una imagen de nginx con 2 replicas. Luego:

	λ kubectl create deployment --dry-run -o yaml webapp --image nginx | kubectl apply -f -
	deployment.apps/webapp created

	λ kubectl get deployment
	NAME     READY   UP-TO-DATE   AVAILABLE   AGE
	webapp   1/1     1            1           6s

	λ kubectl scale deployment webapp --replicas=2
	deployment.extensions/webapp scaled

	λ kubectl get deployment
	NAME     READY   UP-TO-DATE   AVAILABLE   AGE
	webapp   2/2     2            2           23s

	λ kubectl get pods
	NAME                      READY   STATUS      RESTARTS   AGE
	webapp-7fc499b78b-45pz4   1/1     Running     0          26s
	webapp-7fc499b78b-9jt54   1/1     Running     0          8s

	* Obtener el deployment creado y mostrar tus labels 
	
		λ kubectl get deployment --show-labels
		NAME     READY   UP-TO-DATE   AVAILABLE   AGE     LABELS
		webapp   2/2     2            2           4m55s   app=webapp

	* Obtener el archivo yaml del deployment creado 

		//deployment-webapp.yaml
		λ kubectl get deployment webapp -o yaml > deployment-webapp.yaml

	* Mostrar los pods corriendo del deployment 

		λ kubectl get pods --selector=app=webapp
		NAME                      READY   STATUS    RESTARTS   AGE
		webapp-7fc499b78b-45pz4   1/1     Running   0          9m12s
		webapp-7fc499b78b-9jt54   1/1     Running   0          8m54s	

	* Escalar el deployment de 2 a 5 replicas y verificar que funciona correctamente 
		λ kubectl get pods --selector=app=webapp
		NAME                      READY   STATUS    RESTARTS   AGE
		webapp-7fc499b78b-45pz4   1/1     Running   0          10m
		webapp-7fc499b78b-9jt54   1/1     Running   0          10m
		webapp-7fc499b78b-h65b9   1/1     Running   0          13s
		webapp-7fc499b78b-hwjzc   1/1     Running   0          13s
		webapp-7fc499b78b-wwjh7   1/1     Running   0          13s
	
	* Obtener el estado del scale-up del deployment 
		λ kubectl describe deployment webapp
		Name:                   webapp
		Namespace:              default
		CreationTimestamp:      Thu, 16 Apr 2020 00:21:18 -0300
		Labels:                 app=webapp
		Annotations:            deployment.kubernetes.io/revision: 1
								kubectl.kubernetes.io/last-applied-configuration:
								{"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"creationTimestamp":null,"labels":{"app":"webapp"},"name":"webapp...
		Selector:               app=webapp
		Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
		StrategyType:           RollingUpdate
		MinReadySeconds:        0
		RollingUpdateStrategy:  25% max unavailable, 25% max surge
		Pod Template:
		Labels:  app=webapp
		Containers:
		nginx:
			Image:        nginx
			Port:         <none>
			Host Port:    <none>
			Environment:  <none>
			Mounts:       <none>
		Volumes:        <none>
		Conditions:
		Type           Status  Reason
		----           ------  ------
		Progressing    True    NewReplicaSetAvailable
		Available      True    MinimumReplicasAvailable
		OldReplicaSets:  <none>
		NewReplicaSet:   webapp-7fc499b78b (5/5 replicas created)
		Events:
		Type    Reason             Age   From                   Message
		----    ------             ----  ----                   -------
		Normal  ScalingReplicaSet  11m   deployment-controller  Scaled up replica set webapp-7fc499b78b to 1
		Normal  ScalingReplicaSet  11m   deployment-controller  Scaled up replica set webapp-7fc499b78b to 2
		Normal  ScalingReplicaSet  114s  deployment-controller  Scaled up replica set webapp-7fc499b78b to 5

	* Mostrar el replicaset creado por el deployment 
		λ kubectl get replicaset -o wide
		NAME                DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES   SELECTOR
		webapp-7fc499b78b   5         5         5       13m   nginx        nginx    app=webapp,pod-template-hash=7fc499b78b

		//replicaset-webapp.yaml
		λ kubectl get replicaset -o yaml > replicaset-webapp.yaml

		λ kubectl describe replicaset --selector=app=webapp
		Name:           webapp-7fc499b78b
		Namespace:      default
		Selector:       app=webapp,pod-template-hash=7fc499b78b
		Labels:         app=webapp
						pod-template-hash=7fc499b78b
		Annotations:    deployment.kubernetes.io/desired-replicas: 5
						deployment.kubernetes.io/max-replicas: 7
						deployment.kubernetes.io/revision: 1
		Controlled By:  Deployment/webapp
		Replicas:       5 current / 5 desired
		Pods Status:    5 Running / 0 Waiting / 0 Succeeded / 0 Failed
		Pod Template:
		Labels:  app=webapp
				pod-template-hash=7fc499b78b
		Containers:
		nginx:
			Image:        nginx
			Port:         <none>
			Host Port:    <none>
			Environment:  <none>
			Mounts:       <none>
		Volumes:        <none>
		Events:
		Type    Reason            Age    From                   Message
		----    ------            ----   ----                   -------
		Normal  SuccessfulCreate  14m    replicaset-controller  Created pod: webapp-7fc499b78b-45pz4
		Normal  SuccessfulCreate  14m    replicaset-controller  Created pod: webapp-7fc499b78b-9jt54
		Normal  SuccessfulCreate  4m47s  replicaset-controller  Created pod: webapp-7fc499b78b-h65b9
		Normal  SuccessfulCreate  4m47s  replicaset-controller  Created pod: webapp-7fc499b78b-hwjzc
		Normal  SuccessfulCreate  4m46s  replicaset-controller  Created pod: webapp-7fc499b78b-wwjh7