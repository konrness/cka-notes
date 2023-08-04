# Container Health

K8s allows to automatically restart unhealthy containers. To determine status of apps, use probes.

By default, without any probes, k8s will only consider a container to be "down" if the container process stops.

## Probes

### Startup Probes

Only runs at container startup and stops running once they succeed. 

Liveness and Readiness probes will not run until the startup probe succeeds. Good for apps with long startup times to prevent liveness/readiness from failing too early.

```
startupProbe:
  httpGet:
    path: /
    port: 80
  failureThreshold: 30
  periodSeconds: 10
```

> Checks every 10 seconds, for up to 30 times (300 seconds = 5 mins)

### Liveness Probes

Customize the detection mechanism for liveness. Runs constantly on a schedule. Determines when k8s should restart a container.

### Readiness Probes

Prevent user traffic from being sent to pods if the process is not ready. When not ready the pod is removed from the service load balancers. Runs constantly on a schedule.

## Probe Types

### Exec Probe

Runs a command in the container.

```
livenessProbe:
  exec:
    command: ["echo", "Hello, world!"]
  initialDelaySeconds: 5
  periodSeconds: 5
```

### HTTP Probe

```
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Restart Policies

K8s automatically restarts containers when they fail. Restart Policies defines what should happen when container is unhealthy.

Types:
 - Always - (default) - regardless of exit code (success or error)
 - OnFailure - only if error exit code, best for software that should run once to a successful completion
 - Never - never restart. Used for apps that should only run once, regardless of success.

For OnFailure and Never, pods have Status: Completed or Error.

## Lab

One of these application components is a pod called `beebox-shipping-data` located in the `default` namespace. Unfortunately, the application running in this pod has been crashing repeatedly. While the developers are looking into why this application is crashing, you have been asked to implement a self-healing solution in Kubernetes to quickly recover whenever the application crashes.

Luckily, the application can be fixed when it crashes simply by restarting the container. Modify the pod configuration so the application will automatically restart when it crashes. You can detect an application crash when requests to port `8080` on the container return an `HTTP 500` status code.

Note: The kubeclt `--export` no longer works in the lab environment. An alternative would be:

`kubectl get pod beebox-shipping-data -o yaml > beebox-shipping-data.yml`

### Set a Restart Policy to Restart the Container when it is down

Find the `beebox-shipping-data` pod located in the `default` namespace. Modify this pod so its `restartPolicy` will restart the container whenever it fails.

`restartPolicy: Always`

### Create a Liveness Probe to Detect When the Application Has Crashed

Add a liveness probe to the container that checks the container status by making an HTTP request to the container every 5 seconds. The request should check the / (root) path on port 8080.

```
spec:
  containers:
  - image: linuxacademycontent/random-crashing-web-server:1
    imagePullPolicy: IfNotPresent
    name: shipping-data
    ports:
    - name: liveness-port
      containerPort: 8080
      hostPort: 8080
    livenessProbe:
      httpGet:
        path: /
        port: liveness-port
      failureThreshold: 2
      periodSeconds: 5
```

```
Events:
  Type     Reason     Age                  From     Message
  ----     ------     ----                 ----     -------
  Normal   Pulled     68s (x4 over 4m23s)  kubelet  Container image "linuxacademycontent/random-crashing-web-server:1" already present on machine
  Normal   Created    68s (x4 over 4m23s)  kubelet  Created container shipping-data
  Normal   Started    68s (x4 over 4m23s)  kubelet  Started container shipping-data
  Warning  Unhealthy  23s (x8 over 3m43s)  kubelet  Liveness probe failed: HTTP probe failed with statuscode: 500
  Normal   Killing    23s (x4 over 3m38s)  kubelet  Container shipping-data failed liveness probe, will be restarted


cloud_user@acgk8s-control:~$ kubectl get pods
NAME                   READY   STATUS    RESTARTS     AGE
beebox-shipping-data   1/1     Running   4 (9s ago)   4m40s
busybox                1/1     Running   0            112m
cloud_user@acgk8s-control:~$ 

```