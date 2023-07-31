# Inspecting Pod Resource Usage


## Kubernetes Metrics Server

### Install

`kubectl apply -f https://raw.githubusercontent.com/linuxacademy/content-cka-resources/master/metrics-server-components.yaml`

> This is customized. Default metrics server that comes OOTB doesn't work with clusters created with `kubeadm`.

### `kubectl top`

View data about resource usage in pods/nodes.

`kubectl top pod --sort-by <JSONPATH> selector <selector>`

Test if accessible:

`kubectl get --raw /apis/metrics.k8s.io/`

CPU and Memory for all pods:

`kubectl top pod`

Sort by CPU:

`kubectl top pod --sort-by cpu`

Query based on label value:

`kubectl top pod --selector app=metrics-test`

CPU and Memory for cluster nodes:

`kubectl top node`

## Lab

### Scenario

It looks like a pod in the cluster may be using more CPU than expected. Search the beebox-mobile namespace for pods with the label app=auth, and determine which of these pods is using the most CPU. Save the name of that pod to a file for future reference.

The cluster does not have Metrics Server installed, so you will need to install it in order to perform this task.

### Install Kubernetes Metrics Server

Install the Kubernetes Metrics Server in the cluster. Use this file to install Metrics Server: https://raw.githubusercontent.com/ACloudGuru-Resources/content-cka-resources/master/metrics-server-components.yaml

`kubectl apply -f https://raw.githubusercontent.com/ACloudGuru-Resources/content-cka-resources/master/metrics-server-components.yaml`

### Locate the CPU-Using Pod and Write Its Name to a File

Locate the pod using the most CPU that also meets the following criteria:

 - Located in the `beebox-mobile` namespace
 - Has the label `app=auth`

Ignore any pods that do not meet those criteria, even if they are using more CPU.

Write the name of the pod to the file `/home/cloud_user/cpu-pod-name.txt`.

`kubectl top pod --sort-by cpu --selector app=auth -n beebox-mobile`

```
NAME           CPU(cores)   MEMORY(bytes)
auth-proc      0m           1Mi
beebox-auth1   0m           0Mi
beebox-auth2   0m           0Mi
```

`echo "auth-proc" > /home/cloud_user/cpu-pod-name.txt`

