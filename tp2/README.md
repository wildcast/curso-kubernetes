### TP2

- Create deployment nginx: `kubectl create deployment nginx --image nginx`

- Create service for internal traffic:

    `kubectl expose deployment nginx --port 8080 --target-port 80`
    
- Port-forward to access locally:

    `kubectl port-forward svc/nginx 8080`

- Edit service:

    `kubectl edit svc/nginx` and change `spec.type` to `NodePort`. Will result:
    ```
    NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
    nginx        NodePort    10.37.54.90   <none>        8080:31848/TCP   149m
    ```
    Nginx service will be accesible with node's external IPs.
    
- **Rolling updates:**

    In deployment's yaml (default):
    
    ```yaml
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    ```
    
    To update deployment image: `kubectl set image deploy nginx nginx=nginx:stable-perl`
    
    New pods will go through states: `Pending` -> `ContainerCreating` -> `Running`.
    Old pods will pass to `Terminating` state before being deleted.
    
    If image update fails. Ex: set invalid image `kubectl set image deploy nginx nginx=nginx:stable-perl2`.
    New pods will go through states: `Pending` -> `ContainerCreating` -> `ErrImagePull` -> `ImagePullBackOff`.
    
    To roll-back a failed update: `kubectl rollout undo deploy nginx`, and failed pods will be terminated.

- With 6 replicas `kubectl scale deployment nginx --replicas=6`, configure rolling update by two containers at a time:
    
    ```yaml
    strategy:
      rollingUpdate:
        maxSurge: 2
        maxUnavailable: 2
      type: RollingUpdate
    ```

    Editing the rolling update to 50%:
    ```yaml
    strategy:
      rollingUpdate:
        maxSurge: 50%
        maxUnavailable: 50%
      type: RollingUpdate
    ```
  Which means:
    - At most 9 pods (6 desired pods + 3 maxSurge pods) will be Ready during the update.
    - At least 3 pods (6 desired pods - 3 maxUnavailable pods) will always be Ready during the update.

- **Readiness and Liveness:**
    
    Readiness probe: `http` after one second, each five seconds. If not ready, pod's service will not be exposed (it won't receive requests).
    ```yaml
    readinessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 1
      periodSeconds: 5
    ```
    Liveness probe: `tcp` after five seconds, each three seconds. Consider unhealthy after three failures. If not alive, pod will be restarted.
    ```yaml
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 3
      failureThreshold: 3
    ```  
