# DaemonSet

Run copy of Pod on each node. New pods created on new nodes added to clusters.

Still respect scheduling rules (node labels, taints/tolerations)

## Lab

You are working for BeeBox, a company that provides regular shipments of bees to customers.

The company is using some software on their K8s worker nodes that periodically leaves unnecessary data in a particular location on the worker node. Since this software could run on any node at any time, this trash data periodically appears on various worker nodes and should be cleaned up.

Configure the cluster to create a pod on each worker node that periodically deleted the contents of the `/etc/beebox/tmp` on the worker node. Make sure a copy of this pod is automatically created on each worker node, even as new nodes are added to the cluster. Note that you should not run a copy of the pod on the control plane node.

To accomplish this, you can use the following pod specification as a template:

```
spec:
  containers:
  - name: busybox
    image: busybox:1.27
    command: ['sh', '-c', 'while true; do rm -rf /beebox-temp/*; sleep 60; done']
    volumeMounts:
    - name: beebox-tmp
      mountPath: /beebox-temp
  volumes:
  - name: beebox-tmp
    hostPath:
      path: /etc/beebox/tmp
```

### Create a DaemonSet Specification YAML File

Create a file called daemonset.yml with a DaemonSet YAML specification. Use the provided pod specification in the DaemonSet template.


```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: logs-cleanup
spec:
  selector:
    matchLabels:
      name: logs-cleanup
  template:
    metadata:
      labels:
        name: logs-cleanup
    spec:
      containers:
      - name: busybox
        image: busybox:1.27
        command: ['sh', '-c', 'while true; do rm -rf /beebox-temp/*; sleep 60; done']
        volumeMounts:
        - name: beebox-tmp
          mountPath: /beebox-temp
      volumes:
      - name: beebox-tmp
        hostPath:
          path: /etc/beebox/tmp
```

### Create the DaemonSet in the Cluster

Use your specification YAML file to create the DaemonSet in the cluster. Verify each worker node is running a copy of your pod and the DaemonSet pods are all running.

```
$ kubectl get pods -o wide
NAME                 READY   STATUS    RESTARTS   AGE   IP               NODE          NOMINATED NODE   READINESS GATES
logs-cleanup-dt4fv   1/1     Running   0          7s    192.168.126.4    k8s-worker2   <none>           <none>
logs-cleanup-qg6v5   1/1     Running   0          7s    192.168.194.65   k8s-worker1   <none>           <none>
```