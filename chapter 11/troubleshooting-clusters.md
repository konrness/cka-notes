#  Troubleshooting Clusters

### Kube API Server

"connection to the server was refused" - API server maybe down, or kubeconfig isn't setup correctly
 - Check if docker/kubelet services are running

### Checking Node Status

`kubectl describe node [nodeName]`

Each node runs kubelet and Docker.

`systemctl status kubelet`

`systemctl [start, stop] kubelet`

### Checking System Pods

`kubectl get pods -n kube-system`

### Service Logs

Check logs for k8s-related services on each node using `journalctl`

`sudo journalctl -u kubelet`

`sudo journalctl -u docker`

### Cluster Component Logs

K8s components redirect logs to /var/log, unless running via kubeadm...

```
/var/log/kube-apiserver.log
/var/log/kube-scheduler.log
/var/log/kube-controller-manager.log
```

## Lab

Your company, BeeBox, has a new Kubernetes cluster that was just built by an outside contractor. Last night, someone restarted the servers used to run this cluster. Ever since the restart, some of your team members are reporting issues with one of the worker nodes. Unfortunately, the contractor is no longer working with the company, so you will need to find and fix the problem.

Explore the cluster and determine what is causing the issues. Then take steps to fix the problem and ensure that it does not happen again.

### Determine What is Wrong with the Cluster

Explore the cluster and determine which node is having issues. Then see if you can determine what is causing the node to fail.

```
$ kubectl get nodes
NAME          STATUS     ROLES           AGE   VERSION
k8s-control   Ready      control-plane   13m   v1.27.0
k8s-worker1   Ready      <none>          13m   v1.27.0
k8s-worker2   NotReady   <none>          13m   v1.27.0
```

```
cloud_user@k8s-worker2:~$ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; disabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead)
       Docs: https://kubernetes.io/docs/home/
```

### Fix the Problem

Fix the issue with the broken node. Make sure your fix will keep the node working, even if the node is restarted again.


```
$ sudo systemctl start kubelet
$ sudo systemctl enable kubelet

$ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Mon 2023-08-21 21:04:35 UTC; 15s ago
       Docs: https://kubernetes.io/docs/home/
   Main PID: 2027 (kubelet)
      Tasks: 11 (limit: 4604)
     Memory: 29.4M
     CGroup: /system.slice/kubelet.service
             └─2027 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container->
```

```
$ kubectl get nodes
NAME          STATUS   ROLES           AGE   VERSION
k8s-control   Ready    control-plane   21m   v1.27.0
k8s-worker1   Ready    <none>          20m   v1.27.0
k8s-worker2   Ready    <none>          20m   v1.27.0
```
