# K8s Scheduling

## Scheduling Process

1. Kubernetes scheduler selects a suitable Node for each Pod
1. Eliminates nodes that do not satisfy resource requests
1. nodeSelector - limit which nodes the pod can be scheduled on, using labels (only schedule on pods where label exists with same value)
1. nodeName - bypass scheduling and assign a pod to a node by nodeName.

## Lab

You are working for BeeBox, a company that provides regular shipments of bees to customers. The company has a few pods running in their Kubernetes cluster that depend on special services that exist outside the cluster. These services are highly sensitive, and the security team has asked that they be exposed only to certain network segments.

Unfortunately, only the `k8s-worker2` node exists in the network segment shared by these services. This means only pods on the `k8s-worker2` node will be able to access these sensitive external services, and pods on the `k8s-worker1` or `k8s-control` nodes cannot access them.

Your task is to reconfigure the `auth-gateway` pod and the `auth-data` deployment's replica pods so they will always run on the `k8s-worker2` node.

### Configure the `auth-gateway` Pod to Only Run on `k8s-worker2`

Locate the `auth-gateway` pod in the `beebox-auth` namespace. Modify the pod, using a label and a `nodeSelector` constraint, so it will always be scheduled on `k8s-worker2`. You will need to delete and re-create the pod in order for these changes to take effect.

You can find a YAML descriptor for this pod at `/home/cloud_user/auth-gateway.yml`.

```
$ kubectl label node k8s-worker2 secure=true

apiVersion: v1
kind: Pod
metadata:
  name: auth-gateway
  namespace: beebox-auth
spec:
  nodeSelector:
    secure: "true"
  containers:
  - name: nginx
    image: nginx:1.19.1
    ports:
    - containerPort: 80

$ kubectl get pods -n beebox-auth -o wide
NAME                         READY   STATUS    RESTARTS   AGE     IP              NODE          NOMINATED NODE   READINESS GATES
auth-data-65b88b9d94-j5grb   1/1     Running   0          4m25s   192.168.126.4   k8s-worker2   <none>           <none>
auth-data-65b88b9d94-sbg5n   1/1     Running   0          4m24s   192.168.126.1   k8s-worker2   <none>           <none>
auth-data-65b88b9d94-wcvh5   1/1     Running   0          4m24s   192.168.126.2   k8s-worker2   <none>           <none>
auth-gateway                 1/1     Running   0          4m25s   192.168.126.3   k8s-worker2   <none>           <none>
```

### Configure the `auth-data` Deployment's Replica Pods to Only Run on `k8s-worker2`

You will find the `auth-data` deployment in the `beebox-auth` namespace. Modify the deployment, using a `nodeSelector` constraint, so its replica pods will always run on `k8s-worker2`. These changes should take effect once you make this change via a rolling deployment.

You can find a YAML descriptor for this pod at `/home/cloud_user/auth-data.yml`.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-data
  namespace: beebox-auth
spec:
  replicas: 3
  selector:
    matchLabels:
      app: auth-data
  template:
    metadata:
      labels:
        app: auth-data
    spec:
      nodeSelector:
        secure: "true"
      containers:
      - name: nginx
        image: nginx:1.19.1
        ports:
        - containerPort: 80
```

