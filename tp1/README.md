### TP1

- Set up config to cluster:

    `export KUBECONFIG=/path/to/my/kubeconfig.yaml`

- List current namespaces in the cluster:

    `kubectl get namespace`
    
- List all pods in all namespaces:

    `kubectl get pods --all-namespaces`
    
- List pods in one namespace: Flag `--namespace` or `-n`

    `kubectl get pods --namespace=<insert-namespace-name-here>`

- Run a pod that won't be restated after exited:

    `kubectl run --restart=Never <name> --image=<image> <command>`
    
    Ex: `kubectl run --restart=Never sleep --image=busybox sleep 20;`

- Create a pod with 3 containers busybox with the following commands:
    - `ls; sleep 3600`
    - `echo "Hello World"; sleep 3600`
    - `echo "Este es el 3er contenedor"; sleep 3600`
    
    `kubectl create -f three-containers-pod.yaml`
    
- Get pod status:

    `kubectl get pod three-containers -o wide`
    `kubectl get pod three-containers -o yaml > three-containers-pod-created.yaml`
    
- Generate deployment (named "webapp") yaml:

    `kubectl create deployment webapp --dry-run=client --image nginx -o yaml > webapp-deployment.yaml`

- Apply deployment from file:

    `kubectl apply -f webapp-deployment.yaml`

- Get deployment with labels:

   `kubectl get deployments --show-labels`
   
   ```
   NAME     READY   UP-TO-DATE   AVAILABLE   AGE     LABELS
   webapp   2/2     2            2           3m29s   app=webapp
   ```

- Get deployment into yaml file:

    `kubectl get deployment webapp -o yaml > webapp-deployment.yaml`

- Get running pods from the webapp deployment:

    `kubectl get pods --selector=app=webapp --field-selector=status.phase=Running`

- Scale webapp deployment to 5 replicas:

    `kubectl scale deployment webapp --replicas=5`

- Get pods from webapp deployment:

    `kubectl get pods -l app=webapp`
    
    ```
  NAME                      READY   STATUS    RESTARTS   AGE
  webapp-59d9889648-4swdj   1/1     Running   0          14s
  webapp-59d9889648-58h9m   1/1     Running   0          14s
  webapp-59d9889648-hrtrl   1/1     Running   0          14s
  webapp-59d9889648-tk6tv   1/1     Running   0          19m
  webapp-59d9889648-zf9gg   1/1     Running   0          19m
  ```

- Describe deployment (get status, events, details)

    `kubectl describe deployment webapp`

- Get deployment's replicaset into yaml file:

    `kubectl get replicaset -l app=webapp -o yaml > webapp-replicaset.yaml`

