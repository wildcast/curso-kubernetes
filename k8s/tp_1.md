 **1 - Listar todos los namespaces en un cluster**
```
$ kubectl get ns
NAME              STATUS   AGE
default           Active   20d
kube-node-lease   Active   20d
kube-public       Active   20d
kube-system       Active   20d
monitoring        Active   10d
```
**2 - Listar todos los PODS en todos los namespaces**
```
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
default       3lo-pod                                 3/3     Running   14         10d
kube-system   coredns-66bff467f8-b6b68                1/1     Running   3          20d
kube-system   coredns-66bff467f8-jqrdx                1/1     Running   3          20d
kube-system   etcd-minikube                           1/1     Running   3          20d
kube-system   kindnet-cwsl4                           1/1     Running   4          20d
kube-system   kube-apiserver-minikube                 1/1     Running   3          20d
kube-system   kube-controller-manager-minikube        1/1     Running   4          20d
kube-system   kube-proxy-tklwc                        1/1     Running   3          20d
kube-system   kube-scheduler-minikube                 1/1     Running   5          20d
kube-system   metrics-server-7bc6d75975-7z2vj         1/1     Running   2          15d
kube-system   storage-provisioner                     1/1     Running   5          20d
monitoring    alertmanager-764695dc67-bg96h           0/2     Pending   0          10d
monitoring    prometheus-deployment-d8dcd6d55-tcw2p   1/1     Running   2          10d
```
**3 – Listar los pods en un namespace en particular**
```
$ kubectl get pods -n monitoring
NAME                                    READY   STATUS    RESTARTS   AGE
alertmanager-764695dc67-bg96h           0/2     Pending   0          10d
prometheus-deployment-d8dcd6d55-tcw2p   1/1     Running   2          10d
```

**4 – Crear un POD con 3 containers busybox que realicen lo siguiente**
```
$ cat 04-pod-with-3-containers.yaml
apiVersion: v1
kind: Pod
metadata:
  name: 3lo-pod
  labels:
    app: 3lo
spec:
  containers:
  - name: myapp-container-1
    image: gcr.io/google-samples/hello-app:1.0
    command: ['sh', '-c', 'ls; sleep 3600']
  - name: myapp-container-2
    image: gcr.io/google-samples/hello-app:1.0
    command: ['sh', '-c', 'echo "Hello World"; sleep 3600']
  - name: myapp-container-3
    image: gcr.io/google-samples/hello-app:1.0
    command: ['sh', '-c', 'echo "Este es el 3er contenedor"; sleep 3600']
```
**5 - Una vez creados los mismos chequear su estado**
```
$ kubectl logs 3lo-pod -c myapp-container-1
bin
dev
etc
hello-app
home
lib
media
mnt
proc
root
run
sbin
srv
sys
tmp
usr
var
patrol@paw:~/git/proyectos/pueble/curso-kubernetes/k8s$ kubectl logs 3lo-pod -c myapp-container-2
Hello World
patrol@paw:~/git/proyectos/pueble/curso-kubernetes/k8s$ kubectl logs 3lo-pod -c myapp-container-3
Este es el 3er contenedor
```

**6 - Crear un deployment llamado `webapp` con una imagen de nginx con 2 replicas. Luego:**
```
$ cat 05-webappp-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

$ kubectl apply -f 05-webappp-nginx.yaml

$ kubectl get deployment.apps/webapp
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
webapp   2/2     2            2           7m37s

$ kubectl rollout status deployment.apps/webapp
deployment "webapp" successfully rolled out

 - Obtener el deployment creado y mostrar tus labels

$ kubectl get deployment.apps/webapp --show-labels
NAME     READY   UP-TO-DATE   AVAILABLE   AGE     LABELS
webapp   2/2     2            2           9m32s   app=nginx

 - Obtener el archivo yaml del deployment creado
 
$ kubectl get deployment -o yaml > webapp.yaml

 - Mostrar los pods corriendo del deployment
 
 $ kubectl get pods --selector app=nginx
NAME                      READY   STATUS    RESTARTS   AGE
webapp-6b474476c4-fjks8   1/1     Running   0          20m
webapp-6b474476c4-vw8k2   1/1     Running   0          20m

 - Escalar el deployment de 2 a 5 replicas y verificar que funciona correctamente
 
 $ kubectl scale deployment webapp --replicas=5
deployment.apps/webapp scaled

 - Obtener el estado del scale-up del deployment

$ kubectl get deployments webapp
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
webapp   5/5     5            5           31m

 $ kubectl get pods --selector app=nginx
NAME                      READY   STATUS    RESTARTS   AGE
webapp-6b474476c4-5c6dj   1/1     Running   0          43s
webapp-6b474476c4-b7pbl   1/1     Running   0          43s
webapp-6b474476c4-fjks8   1/1     Running   0          31m
webapp-6b474476c4-vw8k2   1/1     Running   0          31m
webapp-6b474476c4-wkj5s   1/1     Running   0          43s

 - Mostrar el replicaset creado por el deployment

 $ kubectl get replicaset --selector app=nginx
NAME                DESIRED   CURRENT   READY   AGE
webapp-6b474476c4   5         5         5       41m


```

