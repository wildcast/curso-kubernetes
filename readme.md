# Configurando kubernetes
Para realizar los TPs vas a necesitar un cluster de kubernetes y el archivo de config para poder usar `kubectl` y comunicarte con el cluster. Hay varias alternativas a la hora de levantar un cluster:
* User un servicio manejado: la mayoria de los cloud providers ofrecen un servicio de kubernetes manejado. Manejado quiere decir que ellos se encargan de mantener el cluster de masters (etcd, api server y demas) por vos. Ademas de eso, suelen prohibirte (algunos te lo permiten) hacer `ssh` a los nodos workers. Lo bueno de estos servicios es que son muy faciles de configurar y para lo que vamos a estar haciendo no van a tener mucho costo. El mas barato que hemos visto es [Digital Ocean](https://cloud.digitalocean.com), por ~20usd mensuales pueden tener un cluster con mas que suficientes recursos para los TPs, de todas maneras se paga por uso y no necesitan tenerlo siempre levantado, pueden crear/eliminar para cada TP, si hacen eso les va a salir practicamente nada.
* Cluster local: una alternativa gratis es levantar kubernetes en su maquina. Al estar en su maquina van a haber un par de cosas que no van a poder hacer, pero para el primer TP alcanza. Pueden usar [kind](https://kind.sigs.k8s.io/docs/user/quick-start/), el unico requerimiento es tener Docker instalado. Una vez que lo instalan pueden crear un cluster con `kind create cluster --name k8s` y eso ya les va a dar un config file para usar con `kubectl`. Si no quieren usar kind pueden usar [minikube](https://minikube.sigs.k8s.io/docs/), la diferencia principal es que `minikube` levanta una maquina virtual y corre los componentes del cluster ahi dentro.
* Crear un cluster con [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/): si se sienten aventureros/as pueden crear el cluster de kubernetes desde cero por su cuenta usando `kubeadm`. Esto va a requerir crear un par de maquinas virtuales (en un cloud provider o localmente), inicializar el cluster en el nodo master y unir los workers al cluster.

## TP1
En este TP la idea es familiarizarse con `kubectl` y el ciclo de vida básico de la creación de un pod y deployment. Tambíen se pide
crear un pod multi-container para así aprender cómo los containers dentro del mismo interactúan entre sí.

1. Listar todos los namespaces en un cluster
```
$ kubectl get namespaces
NAME                 STATUS   AGE
default              Active   47s
kube-node-lease      Active   49s
kube-public          Active   49s
kube-system          Active   49s
local-path-storage   Active   42s
```
2. Listar todos los PODS en todos los namespaces
```
$ kubectl get pods --all-namespaces
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
kube-system          coredns-6955765f44-8f22j                     1/1     Running   0          82s
kube-system          coredns-6955765f44-pl8f6                     1/1     Running   0          82s
kube-system          etcd-kind-control-plane                      1/1     Running   0          94s
kube-system          kindnet-s2flp                                1/1     Running   0          82s
kube-system          kube-apiserver-kind-control-plane            1/1     Running   0          94s
kube-system          kube-controller-manager-kind-control-plane   1/1     Running   0          94s
kube-system          kube-proxy-47f9f                             1/1     Running   0          82s
kube-system          kube-scheduler-kind-control-plane            1/1     Running   0          94s
local-path-storage   local-path-provisioner-7745554f7f-cwnzl      1/1     Running   0          82s
```
3. Listar los pods en un namespace en particular
```
$ kubectl get pods -n kube-system
NAME                                         READY   STATUS    RESTARTS   AGE
coredns-6955765f44-8f22j                     1/1     Running   0          2m56s
coredns-6955765f44-pl8f6                     1/1     Running   0          2m56s
etcd-kind-control-plane                      1/1     Running   0          3m8s
kindnet-s2flp                                1/1     Running   0          2m56s
kube-apiserver-kind-control-plane            1/1     Running   0          3m8s
kube-controller-manager-kind-control-plane   1/1     Running   0          3m8s
kube-proxy-47f9f                             1/1     Running   0          2m56s
kube-scheduler-kind-control-plane            1/1     Running   0          3m8s
```
4. Crear un POD con 3 containers busybox que realizen los iguiente:
	- `ls; sleep 3600`
	- `echo "Hello World"; sleep 3600`
	- `echo "Este es el 3er contenedor"; sleep 3600`
```
$ kubectl apply -f custom-busybox.yaml 
pod/busybox-multicontainer created
```
5. Una vez creados los mismos chequear su estado
```
$ kubectl get pods -l app=busybox -o wide
NAME                     READY   STATUS    RESTARTS   AGE    IP           NODE                 NOMINATED NODE   READINESS GATES
busybox-multicontainer   3/3     Running   0          4m1s   10.244.0.9   kind-control-plane   <none>           <none>
```
6. Crear un deployment llamado `webapp` con una imagen de nginx con 2 replicas. 

```
$ kubectl create deploy webapp --image=nginx --dry-run -o yaml | sed -e "s/replicas: 1/replicas: 2/" | kubectl apply -f -
```
Luego:
* Obtener el deployment creado y mostrar tus labels 
```
$ kubectl get deploy webapp -o wide
NAME     READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES   SELECTOR
webapp   2/2     2            2           2m38s   nginx        nginx    app=webapp

$ kubectl get deploy webapp -o json | jq .spec.template.metadata.labels
{
  "app": "webapp"
}
```
* Mostrar los pods corriendo del deployment 
```
kubectl get pod  -l app=webapp
NAME                      READY   STATUS    RESTARTS   AGE
webapp-58867d7bbb-qdv9r   1/1     Running   0          4m24s
webapp-58867d7bbb-xwvls   1/1     Running   0          4m3s
```
* Escalar el deployment de 2 a 5 replicas y verificar que funciona correctamente
```
$ kubectl scale --replicas=5 deployment  webapp
deployment.apps/webapp scaled
```
* Obtener el estado del scale-up del deployment 
```
$ kubectl get pod  -l app=webapp
NAME                      READY   STATUS    RESTARTS   AGE
webapp-58867d7bbb-b5f52   1/1     Running   0          26s
webapp-58867d7bbb-l9pd4   1/1     Running   0          26s
webapp-58867d7bbb-qdv9r   1/1     Running   0          8m42s
webapp-58867d7bbb-slqfw   1/1     Running   0          26s
webapp-58867d7bbb-xwvls   1/1     Running   0          8m21s
```
* Mostrar el replicaset creado por el deployment 
```
 kubectl get replicaset -l app=webapp
NAME                DESIRED   CURRENT   READY   AGE
webapp-58867d7bbb   2         2         2       11m
```