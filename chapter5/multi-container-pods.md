# Multi-container Pods

Containers in a pod share resources (network/storage). They can interact with one another.

## Shared Resources

**Network:** containers share same network namespace and can communicate on ANY port, even if port is not exposed to the cluster.

**Storage:** can use volumes to share data in a Pod

## Init Containers

Containers that run once to completion, during the startup process. Can have multiple. Each will run once (in order), prior to app container is started.

Use Cases:
 - Pause pod startup until an external service becomes available
 - Perform sensitive startup steps (e.g. fetch secrets)
 - Populate data into a shared volume
 - Communicate with external service (e.g. register the pod with another service)


## Lab

The developers are building an application component designed to run in a Kubernetes pod. This component depends on a Kubernetes service called `shipping-svc`, and they would like their application container to delay startup when this service is not available in the cluster. Once the service becomes available, the main application container should proceed with startup.

Your task is to build a proof-of-concept showing how a pod can be designed that will delay startup of application containers until the service becomes available.

Note: Please use BusyBox 1.27. Newer releases may encounter complications with nslookup.

```
apiVersion: v1
kind: Pod
metadata:
  name: shipping-web
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
  initContainers:
  - name: wait-for-shipping-svc
    image: busybox:1.27
    command: ['sh', '-c', "until nslookup shipping-svc.default.svc.cluster.local; do echo waiting for shipping-svc; sleep 2; done"]
```